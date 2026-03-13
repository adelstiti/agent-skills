# Next.js 15 Best Practices

Modern Next.js 15 App Router patterns and performance optimization.

## Overview

This skill provides guidance for:
- App Router routing and layout conventions
- Data fetching, caching, and revalidation
- Server Components and Client Components
- Server Actions for mutations
- Performance optimization (images, fonts, prefetching)
- Metadata and SEO
- Error handling and loading states
- Security and middleware

## Categories

### 1. Routing & Layout (Critical)
App Router file conventions, layouts, route groups, and dynamic segments.

### 2. Data Fetching & Caching (Critical)
Server-side fetching, cache strategies, revalidation, and streaming.

### 3. Server & Client Components (Critical)
RSC by default, client boundaries, composition, and serialization.

### 4. Server Actions (High)
Mutations, form integration, validation, and revalidation.

### 5. Performance & Optimization (High)
next/image, next/font, next/link, dynamic imports, bundle analysis.

### 6. Metadata & SEO (Medium)
Static/dynamic metadata, Open Graph, sitemap, robots.txt.

### 7. Error Handling & Loading (Medium)
error.tsx, not-found.tsx, loading.tsx, Suspense boundaries.

### 8. Security & Middleware (Medium)
Auth middleware, server-only, env vars, security headers.

## Quick Start

```tsx
// Server Component (default) — async data fetching
export default async function OrdersPage() {
  const orders = await getOrders()
  return <OrderList orders={orders} />
}

// Client Component — interactive UI
'use client'
export function OrderList({ orders }: { orders: Order[] }) {
  const [filter, setFilter] = useState('')
  return (
    <div>
      <input value={filter} onChange={e => setFilter(e.target.value)} />
      {orders.filter(o => o.name.includes(filter)).map(o => (
        <div key={o.id}>{o.name}</div>
      ))}
    </div>
  )
}

// Server Action — mutation with revalidation
'use server'
export async function createOrder(formData: FormData) {
  await db.order.create({ data: parse(formData) })
  revalidatePath('/orders')
}
```

## Usage

This skill triggers automatically when:
- Creating Next.js pages, layouts, or routes
- Implementing data fetching or Server Actions
- Optimizing Next.js performance
- Configuring middleware or metadata

## References

- [Next.js Documentation](https://nextjs.org/docs)
- [React Server Components](https://react.dev/reference/rsc/server-components)
