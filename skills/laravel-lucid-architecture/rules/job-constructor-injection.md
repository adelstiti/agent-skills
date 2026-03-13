---
title: Job Constructor Injection
impact: HIGH
impactDescription: Makes job dependencies explicit and testable
tags: job, constructor, injection, dependency
---

## Job Constructor Injection

**Impact: HIGH (Makes job dependencies explicit and testable)**

Jobs receive their dependencies through the constructor. This makes the job's requirements explicit, enables IDE autocompletion, and simplifies testing. Constructor-promoted readonly properties are the recommended pattern in PHP 8.3+.

## Bad Example

```php
<?php

// ❌ Job using request or global state
namespace App\Domains\Order\Jobs;

use Illuminate\Http\Request;
use Lucid\Units\Job;

class CreateOrderJob extends Job
{
    public function handle(Request $request): Order
    {
        // ❌ Accessing request directly — couples job to HTTP
        return Order::create([
            'user_id' => $request->user()->id,
            'items' => $request->input('items'),
        ]);
    }
}

// ❌ Job using auth() helper — implicit dependency
class GetUserOrdersJob extends Job
{
    public function handle()
    {
        // ❌ Implicit dependency on authentication state
        return auth()->user()->orders()->paginate(15);
    }
}

// ❌ Job with untyped properties
class UpdateOrderJob extends Job
{
    public $orderId;
    public $data;

    public function handle()
    {
        // ❌ No type safety, no readonly protection
        $order = Order::find($this->orderId);
        $order->update($this->data);
        return $order;
    }
}
```

## Good Example

```php
<?php

// ✅ Constructor-promoted readonly properties
namespace App\Domains\Order\Jobs;

use App\Domains\Order\Models\Order;
use Lucid\Units\Job;

class CreateOrderJob extends Job
{
    public function __construct(
        private readonly int $userId,
        private readonly array $items,
        private readonly array $totals,
    ) {}

    public function handle(): Order
    {
        return Order::create([
            'user_id' => $this->userId,
            'subtotal' => $this->totals['subtotal'],
            'tax' => $this->totals['tax'],
            'total' => $this->totals['total'],
        ]);
    }
}

// ✅ Explicit user ID — no auth() coupling
namespace App\Domains\Order\Jobs;

use Lucid\Units\Job;

class GetUserOrdersJob extends Job
{
    public function __construct(
        private readonly int $userId,
        private readonly int $perPage = 15,
    ) {}

    public function handle()
    {
        return Order::where('user_id', $this->userId)
            ->latest()
            ->paginate($this->perPage);
    }
}

// ✅ Typed, immutable properties
namespace App\Domains\Order\Jobs;

use App\Domains\Order\Models\Order;
use Lucid\Units\Job;

class UpdateOrderJob extends Job
{
    public function __construct(
        private readonly int $orderId,
        private readonly array $data,
    ) {}

    public function handle(): Order
    {
        $order = Order::findOrFail($this->orderId);
        $order->update($this->data);

        return $order->fresh();
    }
}

// Feature dispatching with explicit parameters
class UpdateOrderFeature extends Feature
{
    public function handle(UpdateOrderRequest $request, int $id)
    {
        $order = $this->run(UpdateOrderJob::class, [
            'orderId' => $id,
            'data' => $request->validated(),
        ]);

        return redirect()->route('orders.show', $order);
    }
}
```

## Why It Matters

- **Explicit dependencies**: Constructor shows exactly what the job needs
- **Testability**: Pass test data directly — no mocking request or auth
- **Immutability**: Readonly properties prevent accidental mutation
- **Reusability**: Job works from any context — HTTP, console, queue

Reference: [PHP Constructor Promotion](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion)
