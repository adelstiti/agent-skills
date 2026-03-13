---
title: Use App Router Directory Conventions
impact: CRITICAL
impactDescription: Foundation of all routing, layouts, and rendering in Next.js 15
tags: routing, app-router, file-conventions, layout
---

## Use App Router Directory Conventions

**Impact: CRITICAL (Foundation of all routing, layouts, and rendering in Next.js 15)**

The App Router uses file-system conventions to define routes, layouts, loading states, and error boundaries. Understanding and following these conventions is the foundation of every Next.js 15 application.

## Bad Example

```tsx
// ❌ Pages Router patterns in an App Router project
// pages/index.tsx — wrong directory
export default function Home() {
  return <div>Home</div>
}

// ❌ Using getServerSideProps — Pages Router pattern
export async function getServerSideProps() {
  const data = await fetchData()
  return { props: { data } }
}

// ❌ Custom _app.tsx for global layout — Pages Router
export default function MyApp({ Component, pageProps }) {
  return (
    <Layout>
      <Component {...pageProps} />
    </Layout>
  )
}

// ❌ Wrong file names in app/ directory
// app/orders/Orders.tsx  — not recognized by Next.js
// app/orders/index.tsx   — Pages Router convention
```

## Good Example

```tsx
// ✅ App Router file conventions

// app/layout.tsx — Root layout (required)
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}

// app/page.tsx — Home page (/)
export default function HomePage() {
  return <h1>Welcome</h1>
}

// app/orders/page.tsx — Orders page (/orders)
export default async function OrdersPage() {
  const orders = await getOrders()
  return <OrderList orders={orders} />
}

// app/orders/[id]/page.tsx — Dynamic route (/orders/123)
export default async function OrderPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const order = await getOrder(id)
  return <OrderDetail order={order} />
}

// app/orders/loading.tsx — Loading state for /orders
export default function OrdersLoading() {
  return <OrdersSkeleton />
}

// app/orders/error.tsx — Error boundary for /orders
'use client'
export default function OrdersError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}

// app/orders/not-found.tsx — 404 for /orders
export default function OrderNotFound() {
  return <h2>Order not found</h2>
}
```

**Special files recognized by App Router:**

```
page.tsx        — Route UI (makes segment publicly accessible)
layout.tsx      — Shared layout (wraps children, preserved on navigation)
loading.tsx     — Loading UI (Suspense boundary)
error.tsx       — Error boundary (must be 'use client')
not-found.tsx   — Not found UI
template.tsx    — Re-rendered layout (new instance on navigation)
default.tsx     — Fallback for parallel routes
route.ts        — API endpoint (GET, POST, etc.)
```

## Why It Matters

- **Automatic optimization**: Next.js optimizes based on conventions (code splitting, prefetching)
- **Streaming**: loading.tsx enables streaming from the server automatically
- **Error isolation**: error.tsx creates React error boundaries per route segment
- **Team conventions**: Everyone knows where to find the page, layout, and error handler

Reference: [Next.js App Router - File Conventions](https://nextjs.org/docs/app/api-reference/file-conventions)
