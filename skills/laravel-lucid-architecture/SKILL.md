---
name: laravel-lucid-architecture
description: Laravel 12 Lucid Architecture patterns and conventions. Use when structuring Laravel applications with Lucid domains, features, operations, jobs, and data objects. Triggers on tasks involving Lucid Architecture, domain-driven design in Laravel, feature classes, operation classes, job classes, or organizing business logic in Laravel.
license: MIT
metadata:
  author: Lucid Community
  version: "1.0.0"
  laravelVersion: "12.x"
  phpVersion: "8.3+"
---

# Laravel 12 Lucid Architecture

Comprehensive guide for building Laravel 12 applications with Lucid Architecture. Contains 14 rules across 7 categories for building scalable, domain-driven Laravel applications.

## When to Apply

Reference these guidelines when:
- Setting up a new Laravel project with Lucid Architecture
- Creating domains, features, operations, and jobs
- Organizing business logic across service boundaries
- Building multi-service monolith applications
- Structuring data objects and validators
- Implementing cross-domain communication

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Foundation & Structure | CRITICAL | `foundation-` |
| 2 | Features & Operations | CRITICAL | `feature-` |
| 3 | Domains & Services | HIGH | `domain-` |
| 4 | Jobs & Units | HIGH | `job-` |
| 5 | Data & Validation | MEDIUM | `data-` |
| 6 | Testing | MEDIUM | `test-` |
| 7 | Advanced Patterns | LOW-MEDIUM | `advanced-` |

## Quick Reference

### 1. Foundation & Structure (CRITICAL)

- `foundation-directory-structure` - Lucid's directory layout and conventions
- `foundation-service-concept` - Service as a high-level boundary
- `foundation-domain-concept` - Domains encapsulate business logic

### 2. Features & Operations (CRITICAL)

- `feature-single-purpose` - One feature per user interaction
- `feature-controller-dispatch` - Dispatch features from controllers
- `feature-request-validation` - Validate input in features
- `feature-operation-orchestration` - Operations coordinate jobs

### 3. Domains & Services (HIGH)

- `domain-boundaries` - Clear domain boundaries
- `domain-cross-domain` - Cross-domain communication patterns

### 4. Jobs & Units (HIGH)

- `job-single-responsibility` - One task per job
- `job-constructor-injection` - Inject dependencies via constructor

### 5. Data & Validation (MEDIUM)

- `data-objects` - Use data objects for structured data

### 6. Testing (MEDIUM)

- `test-feature-tests` - Test features end-to-end

### 7. Advanced Patterns (LOW-MEDIUM)

- `advanced-multi-service` - Multi-service monolith patterns

## Lucid Architecture Overview

Lucid Architecture organizes Laravel applications into three fundamental pillars:

```
app/
├── Domains/              # Business logic domains
│   ├── User/
│   │   ├── Jobs/
│   │   ├── Models/
│   │   ├── Policies/
│   │   ├── Requests/
│   │   ├── Events/
│   │   └── Exceptions/
│   ├── Order/
│   │   ├── Jobs/
│   │   ├── Models/
│   │   └── ...
│   └── Payment/
│       ├── Jobs/
│       ├── Models/
│       └── ...
├── Services/             # Service boundaries (web, api, admin...)
│   ├── Web/
│   │   ├── Features/
│   │   ├── Operations/
│   │   ├── Http/
│   │   │   ├── Controllers/
│   │   │   └── Middleware/
│   │   ├── routes/
│   │   └── resources/
│   └── Api/
│       ├── Features/
│       ├── Operations/
│       ├── Http/
│       │   └── Controllers/
│       └── routes/
└── Data/                 # Shared data layer
    ├── Models/           # (if not domain-specific)
    └── Repositories/
```

### The Request Lifecycle in Lucid

```
Request → Controller → Feature → Operation → Job(s)
                                              ↓
                                          Domain Logic
                                              ↓
                                          Response
```

## Essential Patterns

### Feature Class

```php
<?php

namespace App\Services\Web\Features;

use App\Domains\Order\Jobs\CreateOrderJob;
use App\Domains\Order\Jobs\ValidateOrderItemsJob;
use App\Domains\Payment\Jobs\ChargePaymentJob;
use App\Domains\User\Jobs\NotifyUserJob;
use Lucid\Units\Feature;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        // Validate items availability
        $this->run(ValidateOrderItemsJob::class, [
            'items' => $request->input('items'),
        ]);

        // Create the order
        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->user()->id,
            'items' => $request->validated(),
        ]);

        // Charge payment
        $this->run(ChargePaymentJob::class, [
            'order' => $order,
            'paymentMethod' => $request->input('payment_method'),
        ]);

        // Notify user
        $this->run(NotifyUserJob::class, [
            'user' => $request->user(),
            'notification' => 'order_created',
            'data' => ['order_id' => $order->id],
        ]);

        return $this->run(RespondWithJsonJob::class, [
            'data' => ['order' => $order],
            'status' => 201,
        ]);
    }
}
```

### Operation Class

```php
<?php

namespace App\Services\Web\Operations;

use App\Domains\Order\Jobs\CalculateOrderTotalJob;
use App\Domains\Order\Jobs\ApplyDiscountJob;
use App\Domains\Order\Jobs\ValidateStockJob;
use Lucid\Units\Operation;

class ProcessOrderOperation extends Operation
{
    public function handle(array $items, ?string $discountCode = null)
    {
        // Validate stock for all items
        $this->run(ValidateStockJob::class, [
            'items' => $items,
        ]);

        // Calculate base total
        $total = $this->run(CalculateOrderTotalJob::class, [
            'items' => $items,
        ]);

        // Apply discount if provided
        if ($discountCode) {
            $total = $this->run(ApplyDiscountJob::class, [
                'total' => $total,
                'code' => $discountCode,
            ]);
        }

        return $total;
    }
}
```

### Job Class (Domain)

```php
<?php

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
    ) {}

    public function handle(): Order
    {
        return DB::transaction(function () {
            $order = Order::create([
                'user_id' => $this->userId,
                'status' => 'pending',
                'total' => collect($this->items)->sum(
                    fn ($item) => $item['price'] * $item['quantity']
                ),
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
```

### Controller (Thin)

```php
<?php

namespace App\Services\Web\Http\Controllers;

use App\Services\Web\Features\CreateOrderFeature;
use App\Services\Web\Features\ListOrdersFeature;
use App\Services\Web\Features\ShowOrderFeature;
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
}
```
