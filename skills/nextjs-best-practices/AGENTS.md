# Next.js 15 Best Practices - Complete Guide

**Version:** 1.0.0
**Next.js Version:** 15.x
**React Version:** 19.x
**Organization:** Next.js Community
**Date:** March 2026

## Overview

Comprehensive guide for building Next.js 15 applications with the App Router, designed for AI agents and LLMs. Contains 35+ rules across 8 categories, prioritized by impact from critical (routing, data fetching, component model) to incremental (security and middleware). Each rule includes detailed explanations, real-world examples comparing incorrect vs. correct implementations, and specific impact metrics.

### Key Features

- App Router with file-system based routing
- React Server Components by default
- Server Actions for data mutations
- Streaming with Suspense boundaries
- Parallel and intercepting routes
- Partial Prerendering (PPR)
- next/image, next/font, next/link optimizations
- Middleware for authentication and redirects
- Static and dynamic metadata API

## Categories

This guide is organized into 8 categories, prioritized by their impact on application quality:

1. **Routing & Layout (CRITICAL)** - App Router directory conventions and route organization
2. **Data Fetching & Caching (CRITICAL)** - Server-side fetching, caching, revalidation, streaming
3. **Server & Client Components (CRITICAL)** - RSC by default, Client Component boundaries
4. **Server Actions (HIGH)** - Mutations, form integration, validation
5. **Performance & Optimization (HIGH)** - Images, fonts, prefetching, bundle size
6. **Metadata & SEO (MEDIUM)** - Metadata API, Open Graph, sitemap
7. **Error Handling & Loading (MEDIUM)** - Error boundaries, loading states, Suspense
8. **Security & Middleware (MEDIUM)** - Auth middleware, server-only, env vars, headers

### References

- [Next.js Documentation](https://nextjs.org/docs)
- [Next.js App Router](https://nextjs.org/docs/app)
- [React Server Components](https://react.dev/reference/rsc/server-components)
- [React Server Actions](https://react.dev/reference/rsc/server-actions)

---

## 1. Routing & Layout (CRITICAL)

**Impact:** CRITICAL
**Description:** The App Router uses file-system based routing with special files (page.tsx, layout.tsx, loading.tsx, error.tsx). Proper route organization with groups, dynamic segments, and layouts creates a maintainable, performant application structure.

**Rules in this category:** 7

---

## 2. Data Fetching & Caching (CRITICAL)

**Impact:** CRITICAL
**Description:** Data fetching in the App Router happens in Server Components using async/await. The caching layer (fetch cache, Data Cache, Full Route Cache) must be understood and configured correctly. Parallel fetching, streaming, and revalidation are essential for performance.

**Rules in this category:** 6

---

## 3. Server & Client Components (CRITICAL)

**Impact:** CRITICAL
**Description:** The mental model of Server Components (default, no JS shipped) vs Client Components ('use client', interactive) is the foundation of App Router. Pushing the client boundary to leaves, composing server/client correctly, and understanding serialization rules are critical.

**Rules in this category:** 5

---

## 4. Server Actions (HIGH)

**Impact:** HIGH
**Description:** Server Actions replace API routes for mutations. They integrate with forms, useActionState, and the revalidation system. Proper validation, error handling, and revalidation after mutations are essential for data integrity.

**Rules in this category:** 5

---

## 5. Performance & Optimization (HIGH)

**Impact:** HIGH
**Description:** Next.js provides built-in optimizations for images (next/image), fonts (next/font), and navigation (next/link). Dynamic imports, bundle analysis, and Partial Prerendering further improve performance and Core Web Vitals.

**Rules in this category:** 6

---

## 6. Metadata & SEO (MEDIUM)

**Impact:** MEDIUM
**Description:** The Metadata API provides a type-safe way to define page metadata. Static metadata uses exported objects, dynamic metadata uses generateMetadata. Open Graph, Twitter cards, sitemaps, and robots.txt complete the SEO picture.

**Rules in this category:** 5

---

## 7. Error Handling & Loading (MEDIUM)

**Impact:** MEDIUM
**Description:** Special files (error.tsx, not-found.tsx, loading.tsx) create automatic error boundaries and loading states. Suspense boundaries enable streaming for granular loading UIs.

**Rules in this category:** 5

---

## 8. Security & Middleware (MEDIUM)

**Impact:** MEDIUM
**Description:** Middleware runs before every request for auth checks, redirects, and header manipulation. The server-only package prevents accidental client-side imports of server code. Security headers and CSRF protection complete the security picture.

**Rules in this category:** 5
