---
title: Feature Tests for End-to-End Flows
impact: MEDIUM
impactDescription: Validates the complete request-to-response lifecycle
tags: testing, feature-test, end-to-end, integration
---

## Feature Tests for End-to-End Flows

**Impact: MEDIUM (Validates the complete request-to-response lifecycle)**

Feature tests in Lucid verify the full user interaction: HTTP request → controller → feature → jobs → response. They test that Lucid units are wired together correctly and produce the expected outcome.

## Bad Example

```php
<?php

// ❌ Testing implementation details instead of behavior
namespace Tests\Feature;

class CreateOrderTest extends TestCase
{
    public function test_create_order_calls_correct_jobs(): void
    {
        // ❌ Testing that specific jobs are called — brittle
        $this->mock(CreateOrderJob::class)
            ->shouldReceive('handle')
            ->once();

        $this->mock(CalculateOrderTotalJob::class)
            ->shouldReceive('handle')
            ->once();

        $this->post('/orders', [...]);
    }

    public function test_order_creation(): void
    {
        // ❌ No authentication, no factory, raw data
        $response = $this->post('/orders', [
            'items' => [
                ['product_id' => 1, 'quantity' => 2],
            ],
        ]);

        // ❌ Only checking status code — not enough
        $response->assertStatus(302);
    }
}
```

## Good Example

```php
<?php

// ✅ Feature test verifying behavior end-to-end
namespace Tests\Feature;

use App\Domains\Order\Models\Order;
use App\Domains\Product\Models\Product;
use App\Domains\User\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class CreateOrderTest extends TestCase
{
    use RefreshDatabase;

    public function test_authenticated_user_can_create_order(): void
    {
        $user = User::factory()->create();
        $product = Product::factory()->create([
            'price' => 29.99,
            'stock' => 10,
        ]);

        $response = $this->actingAs($user)->post('/orders', [
            'items' => [
                [
                    'product_id' => $product->id,
                    'quantity' => 2,
                    'price' => $product->price,
                ],
            ],
        ]);

        $response->assertRedirect();
        $response->assertSessionHas('success');

        // Assert the order was created correctly
        $this->assertDatabaseHas('orders', [
            'user_id' => $user->id,
            'status' => 'pending',
        ]);

        $this->assertDatabaseHas('order_items', [
            'product_id' => $product->id,
            'quantity' => 2,
            'price' => 29.99,
        ]);

        // Assert stock was decremented
        $this->assertEquals(8, $product->fresh()->stock);
    }

    public function test_order_fails_with_insufficient_stock(): void
    {
        $user = User::factory()->create();
        $product = Product::factory()->create([
            'stock' => 1,
        ]);

        $response = $this->actingAs($user)->post('/orders', [
            'items' => [
                [
                    'product_id' => $product->id,
                    'quantity' => 5,
                    'price' => $product->price,
                ],
            ],
        ]);

        $response->assertStatus(422);
        $this->assertDatabaseCount('orders', 0);
    }

    public function test_unauthenticated_user_cannot_create_order(): void
    {
        $response = $this->post('/orders', [
            'items' => [],
        ]);

        $response->assertRedirect('/login');
    }
}

// ✅ Unit test for an individual job
namespace Tests\Unit\Domains\Order\Jobs;

use App\Domains\Order\Data\OrderTotals;
use App\Domains\Order\Jobs\CalculateOrderTotalJob;
use Tests\TestCase;

class CalculateOrderTotalJobTest extends TestCase
{
    public function test_calculates_totals_correctly(): void
    {
        $job = new CalculateOrderTotalJob(items: [
            ['product_id' => 1, 'quantity' => 2, 'price' => 10.00],
            ['product_id' => 2, 'quantity' => 1, 'price' => 25.00],
        ]);

        $totals = $job->handle();

        $this->assertInstanceOf(OrderTotals::class, $totals);
        $this->assertEquals(45.00, $totals->subtotal);
        $this->assertEquals(4.50, $totals->tax);
        $this->assertEquals(49.50, $totals->total);
    }

    public function test_handles_empty_items(): void
    {
        $job = new CalculateOrderTotalJob(items: []);

        $totals = $job->handle();

        $this->assertEquals(0.00, $totals->total);
    }
}
```

## Why It Matters

- **Confidence**: Feature tests verify the entire Lucid pipeline works together
- **Behavior-focused**: Test what the user sees, not which jobs run internally
- **Refactoring-safe**: Change internal job structure without breaking tests
- **Regression prevention**: Catch integration bugs that unit tests miss

Reference: [Laravel Testing](https://laravel.com/docs/12.x/testing)
