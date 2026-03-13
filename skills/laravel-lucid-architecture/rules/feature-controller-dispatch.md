---
title: Dispatch Features from Controllers
impact: CRITICAL
impactDescription: Keeps controllers thin and delegates to the Lucid pipeline
tags: feature, controller, dispatch, serve
---

## Dispatch Features from Controllers

**Impact: CRITICAL (Keeps controllers thin and delegates to the Lucid pipeline)**

Controllers in Lucid Architecture are extremely thin — they only serve features. No business logic, no validation, no data manipulation. The controller's `serve()` method dispatches a feature class that handles the entire request lifecycle.

## Bad Example

```php
<?php

// ❌ Fat controller with logic — bypasses Lucid pipeline
namespace App\Services\Web\Http\Controllers;

use App\Domains\Order\Models\Order;
use Illuminate\Http\Request;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        // ❌ Validation in controller
        $validated = $request->validate([
            'items' => 'required|array',
            'items.*.product_id' => 'required|exists:products,id',
            'items.*.quantity' => 'required|integer|min:1',
        ]);

        // ❌ Business logic in controller
        $total = 0;
        foreach ($validated['items'] as $item) {
            $product = Product::find($item['product_id']);
            $total += $product->price * $item['quantity'];
        }

        // ❌ Data creation in controller
        $order = Order::create([
            'user_id' => auth()->id(),
            'total' => $total,
        ]);

        // ❌ Side effects in controller
        event(new OrderCreated($order));

        return redirect()->route('orders.show', $order);
    }
}
```

## Good Example

```php
<?php

// ✅ Thin controller — only serves features
namespace App\Services\Web\Http\Controllers;

use App\Services\Web\Features\CreateOrderFeature;
use App\Services\Web\Features\ListOrdersFeature;
use App\Services\Web\Features\ShowOrderFeature;
use App\Services\Web\Features\UpdateOrderFeature;
use App\Services\Web\Features\DeleteOrderFeature;
use Lucid\Units\Controller;

class OrderController extends Controller
{
    public function index()
    {
        return $this->serve(ListOrdersFeature::class);
    }

    public function store()
    {
        return $this->serve(CreateOrderFeature::class);
    }

    public function show(int $id)
    {
        return $this->serve(ShowOrderFeature::class, [
            'id' => $id,
        ]);
    }

    public function update(int $id)
    {
        return $this->serve(UpdateOrderFeature::class, [
            'id' => $id,
        ]);
    }

    public function destroy(int $id)
    {
        return $this->serve(DeleteOrderFeature::class, [
            'id' => $id,
        ]);
    }
}
```

## Why It Matters

- **Consistency**: Every controller method follows the same `serve()` pattern
- **Testability**: Features can be tested independently of HTTP
- **Traceable**: The request → controller → feature → jobs flow is always clear
- **No surprises**: No hidden business logic in controllers

Reference: [Lucid Architecture - Controllers](https://docs.lucidarch.dev/controllers)
