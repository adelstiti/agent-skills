---
title: Validate Input in Features
impact: CRITICAL
impactDescription: Ensures data integrity at the entry point of the Lucid pipeline
tags: feature, validation, form-request, input
---

## Validate Input in Features

**Impact: CRITICAL (Ensures data integrity at the entry point of the Lucid pipeline)**

Features are the first Lucid unit that processes a request. Input validation should happen here — either via Laravel Form Request type-hints or explicit validation within the feature. Never pass unvalidated data to jobs or operations.

## Bad Example

```php
<?php

// ❌ No validation — raw request data passed to jobs
namespace App\Services\Web\Features;

use Lucid\Units\Feature;

class CreateOrderFeature extends Feature
{
    public function handle(Request $request)
    {
        // ❌ Passing raw, unvalidated input to a domain job
        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->input('user_id'),  // Could be spoofed
            'items' => $request->input('items'),      // No structure guarantee
        ]);

        return redirect()->route('orders.show', $order);
    }
}

// ❌ Validation buried inside a job
namespace App\Domains\Order\Jobs;

class CreateOrderJob extends Job
{
    public function handle(): Order
    {
        // ❌ Jobs shouldn't validate HTTP input — that's a feature concern
        $validator = Validator::make($this->data, [
            'items' => 'required|array',
        ]);

        if ($validator->fails()) {
            throw new ValidationException($validator);
        }

        return Order::create($this->data);
    }
}
```

## Good Example

```php
<?php

// ✅ Form Request for validation — injected into the feature
namespace App\Services\Web\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class CreateOrderRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user() !== null;
    }

    public function rules(): array
    {
        return [
            'items' => ['required', 'array', 'min:1'],
            'items.*.product_id' => ['required', 'integer', 'exists:products,id'],
            'items.*.quantity' => ['required', 'integer', 'min:1', 'max:100'],
            'discount_code' => ['nullable', 'string', 'exists:discount_codes,code'],
            'payment_method_id' => ['required', 'string'],
        ];
    }

    public function messages(): array
    {
        return [
            'items.min' => 'At least one item is required.',
            'items.*.quantity.max' => 'Maximum 100 units per item.',
        ];
    }
}

// ✅ Feature type-hints the Form Request — validation happens automatically
namespace App\Services\Web\Features;

use App\Services\Web\Http\Requests\CreateOrderRequest;
use App\Domains\Order\Jobs\CreateOrderJob;
use App\Domains\Order\Jobs\ValidateStockJob;
use Lucid\Units\Feature;

class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        // Input is already validated when we get here
        $validated = $request->validated();

        $this->run(ValidateStockJob::class, [
            'items' => $validated['items'],
        ]);

        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->user()->id,  // From authenticated user, not input
            'items' => $validated['items'],
        ]);

        return redirect()->route('orders.show', $order)
            ->with('success', 'Order placed!');
    }
}
```

## Why It Matters

- **Security**: Validated data prevents injection, spoofing, and malformed input
- **Separation**: Features handle HTTP concerns (validation), jobs handle business logic
- **Early failure**: Invalid requests are rejected before any jobs execute
- **Reusability**: Form Requests can be shared across features in different services

Reference: [Laravel Form Requests](https://laravel.com/docs/12.x/validation#form-request-validation)
