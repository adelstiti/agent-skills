# Sections

This file defines all sections, their ordering, impact levels, and descriptions.
The section ID (in parentheses) is the filename prefix used to group rules.

---

## 1. Routing & Layout (route)

**Impact:** CRITICAL
**Description:** The App Router uses file-system based routing with special files (page.tsx, layout.tsx, loading.tsx, error.tsx). Route groups, dynamic segments, parallel routes, and intercepting routes enable complex UI patterns. Proper layout composition avoids unnecessary re-renders and enables streaming.

## 2. Data Fetching & Caching (data)

**Impact:** CRITICAL
**Description:** Data fetching happens in Server Components with async/await. Understanding the caching layers (Request Memoization, Data Cache, Full Route Cache) and configuring revalidation correctly is essential for both performance and data freshness. Parallel fetching and streaming prevent waterfall requests.

## 3. Server & Client Components (comp)

**Impact:** CRITICAL
**Description:** Server Components render on the server and ship zero JS to the client. Client Components ('use client') enable interactivity. The composition model — pushing client boundaries to leaves, passing Server Components as children — determines bundle size and performance.

## 4. Server Actions (action)

**Impact:** HIGH
**Description:** Server Actions are async functions that run on the server, callable from forms and Client Components. They replace API routes for mutations, integrate with useActionState for progressive enhancement, and trigger revalidation. Proper input validation is mandatory.

## 5. Performance & Optimization (perf)

**Impact:** HIGH
**Description:** Next.js built-in components (next/image, next/font, next/link) provide automatic optimization. Dynamic imports reduce initial bundle size. Partial Prerendering combines static shells with dynamic content. Bundle analysis identifies optimization opportunities.

## 6. Metadata & SEO (meta)

**Impact:** MEDIUM
**Description:** The Metadata API provides type-safe metadata definition. Static metadata exports objects, dynamic metadata uses generateMetadata. Open Graph, Twitter cards, structured data, sitemaps, and robots.txt configuration complete SEO.

## 7. Error Handling & Loading (error)

**Impact:** MEDIUM
**Description:** Special files create automatic error boundaries (error.tsx), 404 pages (not-found.tsx), and loading states (loading.tsx). Suspense boundaries enable granular streaming. Global error handling catches root layout errors.

## 8. Security & Middleware (sec)

**Impact:** MEDIUM
**Description:** Middleware (middleware.ts) runs at the edge before every request for auth, redirects, and headers. The server-only package prevents server code from leaking to the client. Environment variables must follow NEXT_PUBLIC_ naming. Security headers protect against common attacks.
