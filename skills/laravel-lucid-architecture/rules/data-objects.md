---
title: Data Objects for Structured Data
impact: MEDIUM
impactDescription: Improves type safety and prevents primitive obsession
tags: data, dto, value-object, type-safety
---

## Data Objects for Structured Data

**Impact: MEDIUM (Improves type safety and prevents primitive obsession)**

Use data objects (DTOs) to pass structured data between Lucid units instead of raw arrays. Data objects provide type safety, autocompletion, and self-documenting code. They prevent bugs from typos in array keys and make refactoring safe.

## Bad Example

```php
<?php

// ❌ Passing raw arrays between jobs — no type safety
namespace App\Domains\Order\Jobs;

use Lucid\Units\Job;

class CreateOrderJob extends Job
{
    public function __construct(
        private readonly int $userId,
        private readonly array $items,  // What keys does this have?
        private readonly array $totals, // 'subtotal'? 'sub_total'? 'subTotal'?
    ) {}

    public function handle(): Order
    {
        return Order::create([
            'user_id' => $this->userId,
            'subtotal' => $this->totals['subtotal'],  // Typo risk
            'tax' => $this->totals['tax'],
            'total' => $this->totals['totl'],          // Bug! Typo goes unnoticed
        ]);
    }
}

// ❌ Feature building arrays with unclear structure
$totals = $this->run(CalculateOrderTotalJob::class, [
    'items' => $request->input('items'),
]);
// What is $totals? ['subtotal' => ?, 'tax' => ?, 'total' => ?]
// No autocompletion, no type checking
```

## Good Example

```php
<?php

// ✅ Data object with clear types
namespace App\Domains\Order\Data;

final readonly class OrderTotals
{
    public function __construct(
        public float $subtotal,
        public float $tax,
        public float $total,
    ) {}

    public static function fromCalculation(float $subtotal, float $taxRate = 0.1): self
    {
        $tax = round($subtotal * $taxRate, 2);

        return new self(
            subtotal: $subtotal,
            tax: $tax,
            total: round($subtotal + $tax, 2),
        );
    }
}

// ✅ Data object for order items
namespace App\Domains\Order\Data;

final readonly class OrderItemData
{
    public function __construct(
        public int $productId,
        public int $quantity,
        public float $price,
    ) {}

    public static function fromArray(array $data): self
    {
        return new self(
            productId: $data['product_id'],
            quantity: $data['quantity'],
            price: $data['price'],
        );
    }

    public function lineTotal(): float
    {
        return round($this->price * $this->quantity, 2);
    }
}

// ✅ Job returning a typed data object
namespace App\Domains\Order\Jobs;

use App\Domains\Order\Data\OrderTotals;
use Lucid\Units\Job;

class CalculateOrderTotalJob extends Job
{
    /** @param OrderItemData[] $items */
    public function __construct(
        private readonly array $items,
    ) {}

    public function handle(): OrderTotals
    {
        $subtotal = collect($this->items)->sum(
            fn (OrderItemData $item) => $item->lineTotal()
        );

        return OrderTotals::fromCalculation($subtotal);
    }
}

// ✅ Job consuming a typed data object — no typo risk
namespace App\Domains\Order\Jobs;

use App\Domains\Order\Data\OrderTotals;
use App\Domains\Order\Models\Order;
use Lucid\Units\Job;

class CreateOrderJob extends Job
{
    public function __construct(
        private readonly int $userId,
        private readonly array $items,
        private readonly OrderTotals $totals,  // Typed!
    ) {}

    public function handle(): Order
    {
        return Order::create([
            'user_id' => $this->userId,
            'subtotal' => $this->totals->subtotal, // Autocompletion, type-safe
            'tax' => $this->totals->tax,
            'total' => $this->totals->total,
        ]);
    }
}
```

## Why It Matters

- **Type safety**: Compiler catches typos and wrong types at analysis time
- **Autocompletion**: IDE knows the shape of data flowing between units
- **Self-documenting**: Data objects describe what data is expected
- **Refactoring**: Rename a property and the IDE updates all usages

Reference: [PHP Readonly Classes](https://www.php.net/manual/en/language.oop5.basic.php#language.oop5.basic.class.readonly)
