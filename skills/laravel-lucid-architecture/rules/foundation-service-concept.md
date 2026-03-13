---
title: Service as High-Level Boundary
impact: CRITICAL
impactDescription: Defines application entry points and prevents cross-cutting concerns
tags: foundation, service, boundary, isolation
---

## Service as High-Level Boundary

**Impact: CRITICAL (Defines application entry points and prevents cross-cutting concerns)**

In Lucid Architecture, a Service is the highest-level boundary. Each service represents a distinct application interface (web, API, admin, console). Services contain features, operations, controllers, routes, and views — but never business logic directly. Services consume domain jobs through features and operations.

## Bad Example

```php
<?php

// ❌ Single monolithic service for everything
// app/Services/App/Http/Controllers/
//   ├── WebOrderController.php
//   ├── ApiOrderController.php
//   ├── AdminOrderController.php
//   └── ... everything in one service

// ❌ Business logic inside a service
namespace App\Services\Web\Features;

class CreateOrderFeature extends Feature
{
    public function handle(Request $request)
    {
        // ❌ Business logic directly in the feature
        $subtotal = 0;
        foreach ($request->input('items') as $item) {
            $product = Product::find($item['product_id']);
            $subtotal += $product->price * $item['quantity'];
            $product->decrement('stock', $item['quantity']);
        }

        $order = Order::create([
            'user_id' => $request->user()->id,
            'total' => $subtotal * 1.1,
        ]);

        return response()->json($order, 201);
    }
}
```

## Good Example

```php
<?php

// ✅ Separate services for each application boundary
// app/Services/
// ├── Web/          ← Browser-facing application
// │   ├── Features/
// │   ├── Operations/
// │   ├── Http/Controllers/
// │   ├── routes/web.php
// │   └── resources/views/
// ├── Api/          ← REST API for mobile/SPA
// │   ├── Features/
// │   ├── Operations/
// │   ├── Http/Controllers/
// │   └── routes/api.php
// └── Admin/        ← Admin panel
//     ├── Features/
//     ├── Http/Controllers/
//     └── routes/admin.php

// ✅ Web service feature — delegates to domain jobs
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
            ->with('success', 'Order placed successfully.');
    }
}

// ✅ API service feature — same domain jobs, different response
namespace App\Services\Api\Features;

use App\Domains\Order\Jobs\CreateOrderJob;
use App\Domains\Order\Jobs\ValidateOrderItemsJob;
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

        return response()->json(['data' => $order], 201);
    }
}
```

## Why It Matters

- **Reuse**: Domain jobs are shared across services — web and API use the same `CreateOrderJob`
- **Isolation**: Each service has its own routes, middleware, and response format
- **Clarity**: A new developer knows exactly where to find the API feature vs. the web feature
- **Scalability**: New services (mobile API, webhooks) can be added without modifying existing ones

Reference: [Lucid Architecture - Services](https://docs.lucidarch.dev/services)
