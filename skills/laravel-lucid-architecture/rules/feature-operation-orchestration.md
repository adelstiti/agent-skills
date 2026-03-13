---
title: Operation as Job Orchestrator
impact: CRITICAL
impactDescription: Groups related jobs into reusable sequences
tags: feature, operation, orchestration, composition
---

## Operation as Job Orchestrator

**Impact: CRITICAL (Groups related jobs into reusable sequences)**

Operations are mid-level units that orchestrate multiple jobs into a reusable sequence. When a sequence of jobs is needed by more than one feature, extract it into an operation. Operations run jobs, pass data between them, and can contain conditional logic.

## Bad Example

```php
<?php

// ❌ Duplicated job sequences across features
namespace App\Services\Web\Features;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        // This sequence of validate + calculate + create
        // is duplicated in the API feature too
        $this->run(ValidateStockJob::class, ['items' => $request->input('items')]);
        $totals = $this->run(CalculateOrderTotalJob::class, ['items' => $request->input('items')]);
        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->user()->id,
            'items' => $request->input('items'),
            'totals' => $totals,
        ]);

        return redirect()->route('orders.show', $order);
    }
}

// ❌ Same sequence duplicated in API service
namespace App\Services\Api\Features;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        // Copy-pasted from web feature
        $this->run(ValidateStockJob::class, ['items' => $request->input('items')]);
        $totals = $this->run(CalculateOrderTotalJob::class, ['items' => $request->input('items')]);
        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->user()->id,
            'items' => $request->input('items'),
            'totals' => $totals,
        ]);

        return response()->json(['data' => $order], 201);
    }
}
```

## Good Example

```php
<?php

// ✅ Extract shared job sequence into an operation
namespace App\Services\Web\Operations;

use App\Domains\Order\Jobs\ValidateStockJob;
use App\Domains\Order\Jobs\CalculateOrderTotalJob;
use App\Domains\Order\Jobs\CreateOrderJob;
use App\Domains\Order\Models\Order;
use Lucid\Units\Operation;

class PlaceOrderOperation extends Operation
{
    public function __construct(
        private readonly int $userId,
        private readonly array $items,
        private readonly ?string $discountCode = null,
    ) {}

    public function handle(): Order
    {
        $this->run(ValidateStockJob::class, [
            'items' => $this->items,
        ]);

        $totals = $this->run(CalculateOrderTotalJob::class, [
            'items' => $this->items,
        ]);

        if ($this->discountCode) {
            $totals = $this->run(ApplyDiscountJob::class, [
                'totals' => $totals,
                'code' => $this->discountCode,
            ]);
        }

        return $this->run(CreateOrderJob::class, [
            'userId' => $this->userId,
            'items' => $this->items,
            'totals' => $totals,
        ]);
    }
}

// ✅ Web feature uses the operation
namespace App\Services\Web\Features;

use App\Services\Web\Operations\PlaceOrderOperation;
use Lucid\Units\Feature;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        $order = $this->run(PlaceOrderOperation::class, [
            'userId' => $request->user()->id,
            'items' => $request->validated()['items'],
            'discountCode' => $request->input('discount_code'),
        ]);

        return redirect()->route('orders.show', $order)
            ->with('success', 'Order placed!');
    }
}

// ✅ API feature reuses the same operation
namespace App\Services\Api\Features;

use App\Services\Web\Operations\PlaceOrderOperation;
use Lucid\Units\Feature;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        $order = $this->run(PlaceOrderOperation::class, [
            'userId' => $request->user()->id,
            'items' => $request->validated()['items'],
            'discountCode' => $request->input('discount_code'),
        ]);

        return response()->json(['data' => $order], 201);
    }
}
```

## Why It Matters

- **DRY**: Shared job sequences are defined once in an operation
- **Composability**: Features compose operations and jobs like building blocks
- **Testability**: Operations can be tested independently with mocked jobs
- **Readability**: Features read like a high-level description of the workflow

Reference: [Lucid Architecture - Operations](https://docs.lucidarch.dev/operations)
