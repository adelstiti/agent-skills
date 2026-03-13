# Laravel 12 Lucid Architecture - Complete Guide

**Version:** 1.0.0
**Laravel Version:** 12.x
**PHP Version:** 8.3+
**Organization:** Lucid Community
**Date:** March 2026

## Overview

Comprehensive guide for building Laravel 12 applications with Lucid Architecture, designed for AI agents and LLMs. Contains 30+ rules across 7 categories, prioritized by impact from critical (foundation and feature patterns) to incremental (advanced patterns). Each rule includes detailed explanations, real-world examples comparing incorrect vs. correct implementations using PHP 8.3 and Laravel 12 features, and specific impact metrics to guide automated refactoring and code generation.

### Key Features

- Domains as business logic boundaries
- Features as user-facing entry points
- Operations as orchestrators of jobs
- Jobs as atomic, reusable units of work
- Service-based organization (web, api, admin)
- Cross-domain communication patterns
- Modern PHP 8.3 syntax (readonly, constructor promotion, enums)
- Laravel 12 patterns and conventions

## Categories

This guide is organized into 7 categories, prioritized by their impact on application quality:

1. **Foundation & Structure (CRITICAL)** - Core Lucid concepts and directory layout
2. **Features & Operations (CRITICAL)** - Feature classes and operation orchestration
3. **Domains & Services (HIGH)** - Domain boundaries and service isolation
4. **Jobs & Units (HIGH)** - Atomic job design and reusability
5. **Data & Validation (MEDIUM)** - Data objects, transformers, and validation
6. **Testing (MEDIUM)** - Testing Lucid units at correct abstraction levels
7. **Advanced Patterns (LOW-MEDIUM)** - Multi-service, events, API versioning

### References

- [Lucid Architecture Documentation](https://docs.lucidarch.dev)
- [Lucid GitHub Repository](https://github.com/lucidarch/lucid)
- [Laravel 12 Documentation](https://laravel.com/docs/12.x)
- [PHP Type Declarations](https://php.net/manual/en/language.types.declarations.php)

---

## 1. Foundation & Structure (CRITICAL)

**Impact:** CRITICAL
**Description:** Core Lucid Architecture concepts — the directory layout, services as high-level boundaries, domains as business logic containers, features as entry points, operations as orchestrators, and jobs as atomic work units. Understanding these fundamentals is essential for correctly structuring any Lucid application.

**Rules in this category:** 6

---

## 2. Features & Operations (CRITICAL)

**Impact:** CRITICAL
**Description:** Features represent single user interactions and are dispatched from controllers. Operations group related jobs into reusable sequences. Proper feature and operation design ensures clean request handling, consistent responses, and business logic that's easy to test and maintain.

**Rules in this category:** 5

---

## 3. Domains & Services (HIGH)

**Impact:** HIGH
**Description:** Domains encapsulate business logic — models, jobs, policies, events, and exceptions. Services represent application boundaries (web, API, admin). Clear domain boundaries prevent coupling and make the codebase navigable. Cross-domain communication should follow explicit patterns.

**Rules in this category:** 5

---

## 4. Jobs & Units (HIGH)

**Impact:** HIGH
**Description:** Jobs are the atomic units of Lucid Architecture. Each job performs exactly one task, accepts dependencies through its constructor, and returns a meaningful value. Well-designed jobs are reusable across multiple features and operations.

**Rules in this category:** 5

---

## 5. Data & Validation (MEDIUM)

**Impact:** MEDIUM
**Description:** Data objects provide structure for data flowing between layers. Validation rules are centralized in request classes or dedicated validators. Transformers convert between internal models and external representations.

**Rules in this category:** 4

---

## 6. Testing (MEDIUM)

**Impact:** MEDIUM
**Description:** Lucid's layered architecture enables testing at the right abstraction level — feature tests for end-to-end flows, operation tests for orchestration, and unit tests for individual jobs. Each layer has its own testing strategy.

**Rules in this category:** 4

---

## 7. Advanced Patterns (LOW-MEDIUM)

**Impact:** LOW-MEDIUM
**Description:** Advanced patterns for mature Lucid applications: multi-service monoliths sharing domains, event-driven cross-domain communication, policy integration for authorization, and API versioning strategies.

**Rules in this category:** 4
