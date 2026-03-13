---
title: Lucid Directory Structure
impact: CRITICAL
impactDescription: Determines long-term maintainability and team navigation
tags: foundation, structure, directory, organization
---

## Lucid Directory Structure

**Impact: CRITICAL (Determines long-term maintainability and team navigation)**

Lucid Architecture prescribes a specific directory structure that separates concerns into three pillars: Domains (business logic), Services (application boundaries), and Data (shared layer). Following this structure is non-negotiable — deviating breaks tooling and team expectations.

## Bad Example

```php
<?php

// ❌ Standard Laravel structure — business logic scattered everywhere
// app/
// ├── Http/Controllers/OrderController.php    ← mixed concerns
// ├── Models/Order.php
// ├── Services/OrderService.php               ← ad-hoc service layer
// ├── Jobs/ProcessOrder.php                   ← queue job, not Lucid job
// └── Mail/OrderConfirmation.php

// Fat controller with business logic
namespace App\Http\Controllers;

class OrderController extends Controller
{
    public function store(Request $request)
    {
        // Validation, business logic, notification, response
        // all tangled in one method
        $validated = $request->validate([...]);
        $order = Order::create($validated);
        Mail::to($request->user())->send(new OrderConfirmation($order));
        return redirect()->route('orders.show', $order);
    }
}
```

## Good Example

```php
<?php

// ✅ Lucid Architecture directory structure
// app/
// ├── Domains/                    ← Business logic boundaries
// │   ├── Order/
// │   │   ├── Jobs/
// │   │   │   ├── CreateOrderJob.php
// │   │   │   └── CalculateOrderTotalJob.php
// │   │   ├── Models/
// │   │   │   ├── Order.php
// │   │   │   └── OrderItem.php
// │   │   ├── Policies/
// │   │   │   └── OrderPolicy.php
// │   │   ├── Events/
// │   │   │   └── OrderCreated.php
// │   │   └── Exceptions/
// │   │       └── InsufficientStockException.php
// │   ├── User/
// │   │   ├── Jobs/
// │   │   ├── Models/
// │   │   └── ...
// │   └── Payment/
// │       ├── Jobs/
// │       └── ...
// ├── Services/                   ← Application boundaries
// │   ├── Web/
// │   │   ├── Features/
// │   │   │   ├── CreateOrderFeature.php
// │   │   │   └── ListOrdersFeature.php
// │   │   ├── Operations/
// │   │   │   └── ProcessOrderOperation.php
// │   │   ├── Http/
// │   │   │   ├── Controllers/
// │   │   │   │   └── OrderController.php
// │   │   │   └── Middleware/
// │   │   ├── routes/
// │   │   │   └── web.php
// │   │   └── resources/
// │   │       └── views/
// │   └── Api/
// │       ├── Features/
// │       ├── Operations/
// │       ├── Http/Controllers/
// │       └── routes/api.php
// └── Data/                       ← Shared data layer
//     └── Repositories/

// Thin controller dispatching a feature
namespace App\Services\Web\Http\Controllers;

use App\Services\Web\Features\CreateOrderFeature;
use Lucid\Units\Controller;

class OrderController extends Controller
{
    public function store()
    {
        return $this->serve(CreateOrderFeature::class);
    }
}
```

## Why It Matters

- **Navigability**: Developers find code by domain (Order, User, Payment), not by type
- **Separation of concerns**: Business logic in Domains, HTTP in Services, shared data in Data
- **Scalability**: New domains and services can be added without affecting existing ones
- **Tooling support**: Lucid CLI generates files in the correct locations automatically

Reference: [Lucid Architecture - Getting Started](https://docs.lucidarch.dev/getting-started)
