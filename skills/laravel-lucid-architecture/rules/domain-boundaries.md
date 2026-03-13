---
title: Domain Boundaries and Isolation
impact: HIGH
impactDescription: Prevents coupling between business domains
tags: domain, boundaries, isolation, coupling
---

## Domain Boundaries and Isolation

**Impact: HIGH (Prevents coupling between business domains)**

Each domain in Lucid Architecture must have clear boundaries. Domains should not directly instantiate or call jobs from other domains within their own jobs. Cross-domain communication happens at the feature or operation level, where jobs from multiple domains are orchestrated together.

## Bad Example

```php
<?php

// ❌ Domain job calling another domain's job directly
namespace App\Domains\Order\Jobs;

use App\Domains\Payment\Jobs\ChargePaymentJob;
use App\Domains\Notification\Jobs\SendOrderConfirmationJob;
use Lucid\Units\Job;

class CreateOrderJob extends Job
{
    public function handle(): Order
    {
        $order = Order::create([...]);

        // ❌ Calling Payment domain directly from Order domain
        $payment = (new ChargePaymentJob(
            amount: $order->total,
            userId: $order->user_id,
        ))->handle();

        // ❌ Calling Notification domain directly from Order domain
        (new SendOrderConfirmationJob(order: $order))->handle();

        return $order;
    }
}

// ❌ Domain model depending on another domain's model directly
namespace App\Domains\Order\Models;

class Order extends Model
{
    public function charge(): void
    {
        // ❌ Order domain knows about Payment internals
        $this->user->stripeCustomer->charge($this->total);
    }
}
```

## Good Example

```php
<?php

// ✅ Each domain job only knows about its own domain
namespace App\Domains\Order\Jobs;

use App\Domains\Order\Models\Order;
use App\Domains\Order\Models\OrderItem;
use Illuminate\Support\Facades\DB;
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
        return DB::transaction(function () {
            $order = Order::create([
                'user_id' => $this->userId,
                'subtotal' => $this->totals['subtotal'],
                'tax' => $this->totals['tax'],
                'total' => $this->totals['total'],
                'status' => 'pending',
            ]);

            foreach ($this->items as $item) {
                OrderItem::create([
                    'order_id' => $order->id,
                    'product_id' => $item['product_id'],
                    'quantity' => $item['quantity'],
                    'price' => $item['price'],
                ]);
            }

            return $order;
        });
    }
}

// ✅ Cross-domain orchestration happens in features/operations
namespace App\Services\Web\Features;

use App\Domains\Order\Jobs\CreateOrderJob;
use App\Domains\Order\Jobs\ValidateStockJob;
use App\Domains\Order\Jobs\CalculateOrderTotalJob;
use App\Domains\Payment\Jobs\ChargePaymentJob;
use App\Domains\Notification\Jobs\SendOrderConfirmationJob;
use Lucid\Units\Feature;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        // Validate (Order domain)
        $this->run(ValidateStockJob::class, [
            'items' => $request->input('items'),
        ]);

        // Calculate (Order domain)
        $totals = $this->run(CalculateOrderTotalJob::class, [
            'items' => $request->validated()['items'],
        ]);

        // Create (Order domain)
        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->user()->id,
            'items' => $request->validated()['items'],
            'totals' => $totals,
        ]);

        // Charge (Payment domain) — cross-domain at feature level
        $this->run(ChargePaymentJob::class, [
            'amount' => $order->total,
            'paymentMethodId' => $request->input('payment_method_id'),
        ]);

        // Notify (Notification domain) — cross-domain at feature level
        $this->run(SendOrderConfirmationJob::class, [
            'order' => $order,
        ]);

        return redirect()->route('orders.show', $order);
    }
}
```

## Why It Matters

- **Loose coupling**: Domains evolve independently — changing Payment doesn't affect Order jobs
- **Testability**: Domain jobs can be tested without mocking other domains
- **Replaceability**: Swap payment providers by changing only Payment domain
- **Clarity**: The feature shows the full cross-domain workflow in one place

Reference: [Lucid Architecture - Domains](https://docs.lucidarch.dev/domains)
