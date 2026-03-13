---
title: Cross-Domain Communication Patterns
impact: HIGH
impactDescription: Prevents tight coupling while enabling domain collaboration
tags: domain, cross-domain, events, communication
---

## Cross-Domain Communication Patterns

**Impact: HIGH (Prevents tight coupling while enabling domain collaboration)**

When domains need to communicate, use the feature/operation layer for orchestration or Laravel events for asynchronous decoupling. Never have domain jobs directly reference other domain's jobs or models.

## Bad Example

```php
<?php

// ❌ Order domain job directly using Payment domain
namespace App\Domains\Order\Jobs;

use App\Domains\Payment\Models\Payment;
use App\Domains\Payment\Jobs\ProcessRefundJob;
use Lucid\Units\Job;

class CancelOrderJob extends Job
{
    public function handle(): void
    {
        $order = Order::findOrFail($this->orderId);
        $order->update(['status' => 'cancelled']);

        // ❌ Direct dependency on Payment domain
        $payment = Payment::where('order_id', $order->id)->first();
        (new ProcessRefundJob(payment: $payment))->handle();

        // ❌ Direct dependency on Inventory domain
        foreach ($order->items as $item) {
            $item->product->increment('stock', $item->quantity);
        }
    }
}
```

## Good Example

```php
<?php

// ✅ Option 1: Feature-level orchestration — explicit and visible
namespace App\Services\Web\Features;

use App\Domains\Order\Jobs\CancelOrderJob;
use App\Domains\Payment\Jobs\RefundPaymentJob;
use App\Domains\Inventory\Jobs\RestoreStockJob;
use App\Domains\Notification\Jobs\SendCancellationEmailJob;
use Lucid\Units\Feature;

class CancelOrderFeature extends Feature
{
    public function handle(Request $request, int $id)
    {
        // Cancel order (Order domain)
        $order = $this->run(CancelOrderJob::class, [
            'orderId' => $id,
        ]);

        // Refund payment (Payment domain)
        $this->run(RefundPaymentJob::class, [
            'orderId' => $order->id,
        ]);

        // Restore stock (Inventory domain)
        $this->run(RestoreStockJob::class, [
            'items' => $order->items->toArray(),
        ]);

        // Notify user (Notification domain)
        $this->run(SendCancellationEmailJob::class, [
            'order' => $order,
        ]);

        return redirect()->route('orders.index')
            ->with('success', 'Order cancelled and refunded.');
    }
}

// ✅ Option 2: Event-driven — decoupled and async-capable
namespace App\Domains\Order\Jobs;

use App\Domains\Order\Events\OrderCancelled;
use App\Domains\Order\Models\Order;
use Lucid\Units\Job;

class CancelOrderJob extends Job
{
    public function __construct(
        private readonly int $orderId,
    ) {}

    public function handle(): Order
    {
        $order = Order::findOrFail($this->orderId);
        $order->update(['status' => 'cancelled']);

        // ✅ Fire domain event — listeners handle cross-domain effects
        event(new OrderCancelled($order));

        return $order;
    }
}

// ✅ Event class in Order domain
namespace App\Domains\Order\Events;

use App\Domains\Order\Models\Order;
use Illuminate\Foundation\Events\Dispatchable;

class OrderCancelled
{
    use Dispatchable;

    public function __construct(
        public readonly Order $order,
    ) {}
}

// ✅ Listener in Payment domain — reacts to Order event
namespace App\Domains\Payment\Listeners;

use App\Domains\Order\Events\OrderCancelled;
use App\Domains\Payment\Models\Payment;

class RefundPaymentOnOrderCancellation
{
    public function handle(OrderCancelled $event): void
    {
        $payment = Payment::where('order_id', $event->order->id)
            ->where('status', 'completed')
            ->first();

        if ($payment) {
            $payment->refund();
        }
    }
}

// ✅ Listener in Inventory domain
namespace App\Domains\Inventory\Listeners;

use App\Domains\Order\Events\OrderCancelled;

class RestoreStockOnOrderCancellation
{
    public function handle(OrderCancelled $event): void
    {
        foreach ($event->order->items as $item) {
            $item->product->increment('stock', $item->quantity);
        }
    }
}
```

## Why It Matters

- **Loose coupling**: Domains don't depend on each other's internals
- **Flexibility**: Choose explicit orchestration (features) or implicit (events) per use case
- **Async-ready**: Event listeners can be queued for background processing
- **Scalability**: New domains react to events without modifying the source domain

Reference: [Laravel Events](https://laravel.com/docs/12.x/events)
