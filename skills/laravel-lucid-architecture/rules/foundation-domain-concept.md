---
title: Domain as Business Logic Container
impact: CRITICAL
impactDescription: Encapsulates all business logic within clear boundaries
tags: foundation, domain, business-logic, encapsulation
---

## Domain as Business Logic Container

**Impact: CRITICAL (Encapsulates all business logic within clear boundaries)**

A Domain in Lucid Architecture encapsulates all business logic for a specific area of the application. Each domain contains its models, jobs, policies, events, exceptions, and anything else related to that business concept. Domains are the source of truth for business rules.

## Bad Example

```php
<?php

// ❌ Models in generic App\Models — no domain boundaries
namespace App\Models;

class Order extends Model
{
    // Business logic mixed into the model
    public function calculateTotal(): float
    {
        $subtotal = $this->items->sum(fn ($i) => $i->price * $i->quantity);
        $tax = $subtotal * 0.1;
        $discount = $this->coupon ? $this->coupon->apply($subtotal) : 0;
        return $subtotal + $tax - $discount;
    }

    public function charge(): void
    {
        // Payment logic in the Order model
        Stripe::charge($this->calculateTotal(), $this->user->stripe_id);
    }

    public function sendConfirmation(): void
    {
        // Notification logic in the Order model
        Mail::to($this->user)->send(new OrderConfirmation($this));
    }
}

// ❌ Jobs in generic App\Jobs — not organized by domain
namespace App\Jobs;

class ProcessOrder implements ShouldQueue
{
    // Queue job that mixes concerns from multiple domains
}
```

## Good Example

```php
<?php

// ✅ Domain: Order — encapsulates order business logic
// app/Domains/Order/
// ├── Jobs/
// │   ├── CreateOrderJob.php
// │   ├── CalculateOrderTotalJob.php
// │   └── CancelOrderJob.php
// ├── Models/
// │   ├── Order.php
// │   └── OrderItem.php
// ├── Policies/
// │   └── OrderPolicy.php
// ├── Events/
// │   └── OrderCreated.php
// └── Exceptions/
//     └── InsufficientStockException.php

namespace App\Domains\Order\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Order extends Model
{
    protected $fillable = ['user_id', 'status', 'total', 'tax'];

    protected $casts = [
        'total' => 'decimal:2',
        'tax' => 'decimal:2',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(\App\Domains\User\Models\User::class);
    }

    public function items(): HasMany
    {
        return $this->hasMany(OrderItem::class);
    }
}

// ✅ Domain job — atomic unit of business logic
namespace App\Domains\Order\Jobs;

use App\Domains\Order\Models\Order;
use Lucid\Units\Job;

class CalculateOrderTotalJob extends Job
{
    public function __construct(
        private readonly array $items,
    ) {}

    public function handle(): array
    {
        $subtotal = collect($this->items)->sum(
            fn (array $item) => $item['price'] * $item['quantity']
        );

        $tax = round($subtotal * 0.1, 2);

        return [
            'subtotal' => $subtotal,
            'tax' => $tax,
            'total' => $subtotal + $tax,
        ];
    }
}

// ✅ Domain: Payment — separate domain for payment concerns
namespace App\Domains\Payment\Jobs;

use Lucid\Units\Job;

class ChargePaymentJob extends Job
{
    public function __construct(
        private readonly float $amount,
        private readonly string $paymentMethodId,
    ) {}

    public function handle(): PaymentResult
    {
        // Payment logic isolated in its own domain
        return PaymentGateway::charge($this->amount, $this->paymentMethodId);
    }
}
```

## Why It Matters

- **Encapsulation**: All business rules for a concept live in one place
- **Independence**: Domains don't depend on services or HTTP layer
- **Reusability**: Domain jobs can be used from any service (web, API, console)
- **Team ownership**: Teams can own entire domains without stepping on each other

Reference: [Lucid Architecture - Domains](https://docs.lucidarch.dev/domains)
