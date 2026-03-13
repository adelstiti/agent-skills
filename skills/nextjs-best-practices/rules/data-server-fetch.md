---
title: Fetch Data in Server Components
impact: CRITICAL
impactDescription: Eliminates client-server waterfalls and enables direct data access
tags: data-fetching, server-components, async, caching
---

## Fetch Data in Server Components

**Impact: CRITICAL (Eliminates client-server waterfalls and enables direct data access)**

Fetch data directly in Server Components using async/await. This avoids the client → API → database waterfall, enables automatic request deduplication, and integrates with Next.js caching. Never use useEffect for initial data fetching in App Router.

## Bad Example

```tsx
// ❌ Client-side fetching with useEffect — waterfall pattern
'use client'

import { useState, useEffect } from 'react'

export default function OrdersPage() {
  const [orders, setOrders] = useState([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  // ❌ Waterfall: HTML loads → JS loads → React hydrates → fetch starts
  useEffect(() => {
    fetch('/api/orders')
      .then(res => res.json())
      .then(data => {
        setOrders(data)
        setLoading(false)
      })
      .catch(err => {
        setError(err.message)
        setLoading(false)
      })
  }, [])

  if (loading) return <Spinner />           // User sees spinner
  if (error) return <Error message={error} />

  return <OrderList orders={orders} />
}

// ❌ API route that's only used for this page
// app/api/orders/route.ts
export async function GET() {
  const orders = await db.order.findMany()  // Could be done in Server Component
  return Response.json(orders)
}
```

## Good Example

```tsx
// ✅ Server Component — data available at render time
// app/(app)/orders/page.tsx
import { db } from '@/lib/db'
import { OrderList } from '@/components/orders/order-list'
import { Suspense } from 'react'

export default async function OrdersPage() {
  // ✅ Direct database access — no API route needed
  // ✅ Data available at render time — no loading spinner
  const orders = await db.order.findMany({
    orderBy: { createdAt: 'desc' },
    include: { items: true },
  })

  return (
    <div>
      <h1 className="text-2xl font-bold">Orders</h1>
      <OrderList orders={orders} />
    </div>
  )
}

// ✅ With external API and caching
export default async function OrdersPage() {
  const orders = await fetch('https://api.example.com/orders', {
    next: { revalidate: 60 },  // Revalidate every 60 seconds
  }).then(res => res.json())

  return <OrderList orders={orders} />
}

// ✅ Parallel data fetching — no waterfall
export default async function DashboardPage() {
  // ✅ Both requests start simultaneously
  const [orders, stats, notifications] = await Promise.all([
    db.order.findMany({ take: 10, orderBy: { createdAt: 'desc' } }),
    db.order.aggregate({
      _sum: { total: true },
      _count: { id: true },
    }),
    db.notification.findMany({ where: { read: false }, take: 5 }),
  ])

  return (
    <div>
      <StatsCards stats={stats} />
      <RecentOrders orders={orders} />
      <Notifications items={notifications} />
    </div>
  )
}

// ✅ Streaming with Suspense — show partial UI immediately
export default async function DashboardPage() {
  return (
    <div>
      {/* Fast content shows immediately */}
      <h1>Dashboard</h1>

      {/* Streamed in when ready */}
      <Suspense fallback={<StatsSkeleton />}>
        <DashboardStats />
      </Suspense>

      <Suspense fallback={<OrdersSkeleton />}>
        <RecentOrders />
      </Suspense>
    </div>
  )
}

// Each component fetches its own data
async function DashboardStats() {
  const stats = await db.order.aggregate({...})
  return <StatsCards stats={stats} />
}

async function RecentOrders() {
  const orders = await db.order.findMany({...})
  return <OrderList orders={orders} />
}
```

## Why It Matters

- **No waterfalls**: Data fetches start at render time, not after hydration
- **No API boilerplate**: Skip creating API routes just to serve page data
- **Automatic caching**: Next.js caches fetch results and deduplicates requests
- **Streaming**: Suspense boundaries show partial UI while slow data loads

Reference: [Next.js Data Fetching](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching)
