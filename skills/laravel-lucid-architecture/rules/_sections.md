# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Foundation & Structure (foundation)

**Impact:** CRITICAL
**Description:** Core Lucid Architecture concepts — the directory layout, services as high-level boundaries, domains as business logic containers, features as entry points, operations as orchestrators, and jobs as atomic work units. These fundamentals determine whether the application benefits from Lucid's separation of concerns or devolves into a misstructured monolith.

## 2. Features & Operations (feature)

**Impact:** CRITICAL
**Description:** Features represent single user interactions and are dispatched from controllers. Operations group related jobs into reusable sequences. Proper feature and operation design ensures clean request handling, consistent responses, and business logic that's easy to test and maintain. Misusing features leads to fat controllers or scattered logic.

## 3. Domains & Services (domain)

**Impact:** HIGH
**Description:** Domains encapsulate business logic — models, jobs, policies, events, and exceptions belong to their domain. Services represent application boundaries (web, API, admin). Clear boundaries prevent cross-domain coupling and make the codebase navigable at scale.

## 4. Jobs & Units (job)

**Impact:** HIGH
**Description:** Jobs are the atomic units of Lucid Architecture. Each job performs exactly one task, accepts dependencies through its constructor, and returns a meaningful value. Well-designed jobs are reusable across multiple features and operations, forming the building blocks of all business logic.

## 5. Data & Validation (data)

**Impact:** MEDIUM
**Description:** Data objects provide structure for data flowing between layers. Validation rules are centralized in request classes or dedicated validators. Transformers convert between internal models and external representations. Proper data handling prevents leaking internal state.

## 6. Testing (test)

**Impact:** MEDIUM
**Description:** Lucid's layered architecture enables testing at the right abstraction level — feature tests for end-to-end flows, operation tests for orchestration logic, and unit tests for individual jobs. Each layer has appropriate isolation and mocking strategies.

## 7. Advanced Patterns (advanced)

**Impact:** LOW-MEDIUM
**Description:** Patterns for mature Lucid applications: multi-service monoliths sharing domains, event-driven cross-domain communication, authorization with policies, and API versioning strategies. These patterns emerge as applications grow in complexity.
