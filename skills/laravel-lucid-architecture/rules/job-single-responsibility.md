---
title: Job as Single Responsibility Unit
impact: HIGH
impactDescription: Ensures atomic, reusable, testable business logic
tags: job, single-responsibility, atomic, reusable
---

## Job as Single Responsibility Unit

**Impact: HIGH (Ensures atomic, reusable, testable business logic)**

A Job in Lucid Architecture is the smallest unit of work. Each job does exactly one thing, accepts its dependencies through the constructor, and returns a meaningful value. Jobs are the building blocks that features and operations compose.

## Bad Example

```php
<?php

// ❌ Job doing multiple things
namespace App\Domains\Order\Jobs;

use Lucid\Units\Job;

class ProcessOrderJob extends Job
{
    public function __construct(
        private readonly array $data,
    ) {}

    public function handle()
    {
        // ❌ Validates stock
        foreach ($this->data['items'] as $item) {
            $product = Product::findOrFail($item['product_id']);
            if ($product->stock < $item['quantity']) {
                throw new \Exception('Out of stock');
            }
        }

        // ❌ Calculates total
        $total = 0;
        foreach ($this->data['items'] as $item) {
            $product = Product::find($item['product_id']);
            $total += $product->price * $item['quantity'];
        }

        // ❌ Creates order
        $order = Order::create([
            'user_id' => $this->data['user_id'],
            'total' => $total,
        ]);

        // ❌ Sends email
        Mail::to($order->user)->send(new OrderConfirmation($order));

        // ❌ Fires event
        event(new OrderCreated($order));

        return $order;
    }
}
```

## Good Example

```php
<?php

// ✅ Each job does exactly one thing

namespace App\Domains\Order\Jobs;

use App\Domains\Order\Exceptions\InsufficientStockException;
use App\Domains\Product\Models\Product;
use Lucid\Units\Job;

class ValidateStockJob extends Job
{
    public function __construct(
        private readonly array $items,
    ) {}

    public function handle(): void
    {
        foreach ($this->items as $item) {
            $product = Product::findOrFail($item['product_id']);

            if ($product->stock < $item['quantity']) {
                throw new InsufficientStockException(
                    "Insufficient stock for {$product->name}. Available: {$product->stock}"
                );
            }
        }
    }
}

// ✅ Separate job for calculation
namespace App\Domains\Order\Jobs;

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
            'total' => round($subtotal + $tax, 2),
        ];
    }
}

// ✅ Separate job for creation
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

// ✅ Separate job for notification
namespace App\Domains\Notification\Jobs;

use App\Domains\Order\Models\Order;
use App\Mail\OrderConfirmation;
use Illuminate\Support\Facades\Mail;
use Lucid\Units\Job;

class SendOrderConfirmationJob extends Job
{
    public function __construct(
        private readonly Order $order,
    ) {}

    public function handle(): void
    {
        Mail::to($this->order->user)->send(
            new OrderConfirmation($this->order)
        );
    }
}
```

## Why It Matters

- **Reusability**: `CalculateOrderTotalJob` can be used in previews, invoices, reports
- **Testability**: Each job can be unit tested with clear inputs and outputs
- **Composability**: Features arrange jobs like building blocks
- **Debugging**: When something fails, you know exactly which job broke

Reference: [Lucid Architecture - Jobs](https://docs.lucidarch.dev/jobs)
