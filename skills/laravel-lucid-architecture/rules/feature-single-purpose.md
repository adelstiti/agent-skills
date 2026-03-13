---
title: Feature as Single User Interaction
impact: CRITICAL
impactDescription: Ensures clean request-to-response lifecycle
tags: feature, single-purpose, user-interaction, entry-point
---

## Feature as Single User Interaction

**Impact: CRITICAL (Ensures clean request-to-response lifecycle)**

A Feature in Lucid Architecture represents exactly one user interaction. It is dispatched from a controller and orchestrates domain jobs and operations to fulfill the request. Features handle the request lifecycle: validate input, run jobs, and return a response.

## Bad Example

```php
<?php

// ❌ Feature doing too many unrelated things
namespace App\Services\Web\Features;

use Lucid\Units\Feature;

class ManageOrderFeature extends Feature
{
    public function handle(Request $request)
    {
        // ❌ Multiple responsibilities in one feature
        if ($request->isMethod('post')) {
            $order = Order::create($request->all());
            return redirect()->route('orders.show', $order);
        }

        if ($request->isMethod('put')) {
            $order = Order::findOrFail($request->route('id'));
            $order->update($request->all());
            return redirect()->route('orders.show', $order);
        }

        if ($request->isMethod('delete')) {
            Order::findOrFail($request->route('id'))->delete();
            return redirect()->route('orders.index');
        }

        // Also listing orders
        return view('orders.index', ['orders' => Order::all()]);
    }
}
```

## Good Example

```php
<?php

// ✅ One feature per user interaction
namespace App\Services\Web\Features;

use App\Domains\Order\Jobs\CreateOrderJob;
use App\Domains\Order\Jobs\ValidateOrderItemsJob;
use App\Domains\Notification\Jobs\SendOrderConfirmationJob;
use Lucid\Units\Feature;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        $this->run(ValidateOrderItemsJob::class, [
            'items' => $request->input('items'),
        ]);

        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->user()->id,
            'items' => $request->validated(),
        ]);

        $this->run(SendOrderConfirmationJob::class, [
            'order' => $order,
        ]);

        return redirect()->route('orders.show', $order)
            ->with('success', 'Order created successfully.');
    }
}

// ✅ Separate feature for listing
class ListOrdersFeature extends Feature
{
    public function handle(Request $request)
    {
        $orders = $this->run(GetUserOrdersJob::class, [
            'userId' => $request->user()->id,
        ]);

        return view('orders.index', compact('orders'));
    }
}

// ✅ Separate feature for updating
class UpdateOrderFeature extends Feature
{
    public function handle(UpdateOrderRequest $request, int $id)
    {
        $order = $this->run(UpdateOrderJob::class, [
            'orderId' => $id,
            'data' => $request->validated(),
        ]);

        return redirect()->route('orders.show', $order)
            ->with('success', 'Order updated successfully.');
    }
}
```

## Why It Matters

- **Clarity**: Each feature has one clear purpose — easy to name, find, and understand
- **Testability**: Test one user interaction per test class
- **Debugging**: Stack traces point to a specific feature, not a multi-purpose handler
- **Code review**: Features are small, focused, and easy to review

Reference: [Lucid Architecture - Features](https://docs.lucidarch.dev/features)
