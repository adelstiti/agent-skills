---
title: Multi-Service Monolith Pattern
impact: LOW-MEDIUM
impactDescription: Enables scaling with shared domains across application boundaries
tags: advanced, multi-service, monolith, scaling
---

## Multi-Service Monolith Pattern

**Impact: LOW-MEDIUM (Enables scaling with shared domains across application boundaries)**

Lucid Architecture's multi-service monolith allows a single Laravel application to serve multiple interfaces (web, API, admin, console) while sharing domain logic. Each service has its own features, operations, controllers, and routes, but all services share the same domains.

## Bad Example

```php
<?php

// ❌ One "App" service trying to handle everything
// app/Services/App/Http/Controllers/
//   ├── OrderController.php        (handles web + API)
//   ├── AdminOrderController.php   (admin mixed in)
//   └── ...

// ❌ Controller with conditional logic based on request type
namespace App\Services\App\Http\Controllers;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        $order = Order::create($request->validated());

        // ❌ Conditional response format — messy
        if ($request->expectsJson()) {
            return response()->json(['data' => $order], 201);
        }

        return redirect()->route('orders.show', $order);
    }

    public function index(Request $request)
    {
        // ❌ Admin vs user logic mixed
        if ($request->user()->isAdmin()) {
            $orders = Order::with('user')->paginate(50);
        } else {
            $orders = $request->user()->orders()->paginate(15);
        }

        if ($request->expectsJson()) {
            return response()->json($orders);
        }

        return view('orders.index', compact('orders'));
    }
}
```

## Good Example

```php
<?php

// ✅ Separate services with shared domains
// app/
// ├── Domains/              ← Shared business logic
// │   ├── Order/
// │   ├── User/
// │   └── Payment/
// ├── Services/
// │   ├── Web/              ← Browser interface
// │   │   ├── Features/
// │   │   │   └── CreateOrderFeature.php
// │   │   ├── Http/Controllers/
// │   │   │   └── OrderController.php
// │   │   └── routes/web.php
// │   ├── Api/              ← REST API for mobile/SPA
// │   │   ├── Features/
// │   │   │   └── CreateOrderFeature.php
// │   │   ├── Http/Controllers/
// │   │   │   └── OrderController.php
// │   │   └── routes/api.php
// │   ├── Admin/            ← Admin panel
// │   │   ├── Features/
// │   │   │   └── ListAllOrdersFeature.php
// │   │   ├── Http/Controllers/
// │   │   │   └── OrderController.php
// │   │   └── routes/admin.php
// │   └── Console/          ← Artisan commands
// │       └── Features/
// │           └── CleanExpiredOrdersFeature.php

// ✅ Web service — returns views
namespace App\Services\Web\Features;

use App\Domains\Order\Jobs\CreateOrderJob;
use App\Domains\Order\Jobs\ValidateStockJob;
use Lucid\Units\Feature;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        $this->run(ValidateStockJob::class, [
            'items' => $request->input('items'),
        ]);

        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->user()->id,
            'items' => $request->validated(),
        ]);

        return redirect()->route('orders.show', $order)
            ->with('success', 'Order created!');
    }
}

// ✅ API service — returns JSON (same domain jobs!)
namespace App\Services\Api\Features;

use App\Domains\Order\Jobs\CreateOrderJob;
use App\Domains\Order\Jobs\ValidateStockJob;
use App\Http\Resources\OrderResource;
use Lucid\Units\Feature;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        $this->run(ValidateStockJob::class, [
            'items' => $request->input('items'),
        ]);

        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->user()->id,
            'items' => $request->validated(),
        ]);

        return new OrderResource($order);
    }
}

// ✅ Admin service — different authorization, different data
namespace App\Services\Admin\Features;

use App\Domains\Order\Jobs\GetAllOrdersJob;
use Lucid\Units\Feature;

class ListAllOrdersFeature extends Feature
{
    public function handle(Request $request)
    {
        $orders = $this->run(GetAllOrdersJob::class, [
            'perPage' => 50,
            'filters' => $request->only(['status', 'date_from', 'date_to']),
        ]);

        return view('admin.orders.index', compact('orders'));
    }
}
```

## Why It Matters

- **Shared logic**: All services reuse the same domain jobs — no duplication
- **Clean separation**: Each service has appropriate middleware, auth, and response format
- **Independent deployment**: Services can evolve independently (add mobile API without touching web)
- **Team scalability**: Teams can own services while sharing domain knowledge

Reference: [Lucid Architecture - Multi-Service](https://docs.lucidarch.dev/services)
