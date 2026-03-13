# Laravel 12 Lucid Architecture

Comprehensive Lucid Architecture patterns for Laravel 12 applications.

## Overview

This skill provides guidance for:
- Lucid Architecture foundation and directory structure
- Features, operations, and jobs
- Domain boundaries and service isolation
- Cross-domain communication
- Data objects and validation
- Testing Lucid units
- Multi-service monolith patterns

## Categories

### 1. Foundation & Structure (Critical)
Directory layout, services, domains, features, operations, and jobs.

### 2. Features & Operations (Critical)
Feature classes, operation orchestration, request validation, and response formatting.

### 3. Domains & Services (High)
Domain boundaries, models, cross-domain communication, and service isolation.

### 4. Jobs & Units (High)
Single-responsibility jobs, constructor injection, return values, and reusability.

### 5. Data & Validation (Medium)
Data objects, transformers, validation rules, and attribute casting.

### 6. Testing (Medium)
Feature tests, operation tests, job unit tests, and domain tests.

### 7. Advanced Patterns (Low-Medium)
Multi-service monolith, event-driven architecture, and API versioning.

## Quick Start

```php
// Controller — thin, just serves features
class OrderController extends Controller
{
    public function store()
    {
        return $this->serve(CreateOrderFeature::class);
    }
}

// Feature — single user interaction
class CreateOrderFeature extends Feature
{
    public function handle(CreateOrderRequest $request)
    {
        $order = $this->run(CreateOrderJob::class, [
            'userId' => $request->user()->id,
            'items' => $request->validated(),
        ]);

        return $this->run(RespondWithJsonJob::class, [
            'data' => ['order' => $order],
            'status' => 201,
        ]);
    }
}

// Job — atomic unit of work
class CreateOrderJob extends Job
{
    public function __construct(
        private readonly int $userId,
        private readonly array $items,
    ) {}

    public function handle(): Order
    {
        return DB::transaction(function () {
            return Order::create([
                'user_id' => $this->userId,
                'status' => 'pending',
            ]);
        });
    }
}
```

## Usage

This skill triggers automatically when:
- Structuring a Laravel app with Lucid Architecture
- Creating features, operations, or jobs
- Organizing domains and services
- Implementing cross-domain communication

## References

- [Lucid Architecture Docs](https://docs.lucidarch.dev)
- [Lucid GitHub](https://github.com/lucidarch/lucid)
- [Laravel Documentation](https://laravel.com/docs)
