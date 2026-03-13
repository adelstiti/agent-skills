---
title: Push Client Boundary to Leaf Components
impact: CRITICAL
impactDescription: Minimizes client-side JavaScript by keeping most of the tree on the server
tags: components, client-boundary, composition, bundle-size
---

## Push Client Boundary to Leaf Components

**Impact: CRITICAL (Minimizes client-side JavaScript by keeping most of the tree on the server)**

The 'use client' directive marks the boundary where components become Client Components. Everything below that boundary (including imports) becomes part of the client bundle. Push 'use client' as deep as possible — to the smallest interactive leaf — to keep the majority of the component tree as Server Components.

## Bad Example

```tsx
// ❌ 'use client' at the page level — entire tree ships to client
// app/(app)/orders/page.tsx
'use client'  // ❌ Everything below is now a Client Component

import { useState } from 'react'
import { OrderTable } from '@/components/orders/order-table'
import { OrderStats } from '@/components/orders/order-stats'
import { OrderFilters } from '@/components/orders/order-filters'

export default function OrdersPage() {
  const [orders, setOrders] = useState([])

  // ❌ All these components are now Client Components
  // ❌ OrderTable and OrderStats don't need interactivity
  // ❌ Heavy chart libraries in OrderStats ship to client
  return (
    <div>
      <OrderStats orders={orders} />        {/* Only displays data */}
      <OrderFilters onFilter={setOrders} /> {/* Actually interactive */}
      <OrderTable orders={orders} />        {/* Only displays data */}
    </div>
  )
}
```

## Good Example

```tsx
// ✅ Server Component page — only the interactive filter is 'use client'
// app/(app)/orders/page.tsx (Server Component)
import { db } from '@/lib/db'
import { OrderStats } from '@/components/orders/order-stats'
import { OrderTable } from '@/components/orders/order-table'
import { OrderFilters } from '@/components/orders/order-filters'

export default async function OrdersPage() {
  const orders = await db.order.findMany({
    orderBy: { createdAt: 'desc' },
  })

  // ✅ OrderStats renders on server — zero JS for charts
  // ✅ OrderTable renders on server — zero JS for the table
  // ✅ OrderFilters is 'use client' — only interactive part
  return (
    <div>
      <OrderStats orders={orders} />
      <OrderFilters />
      <OrderTable orders={orders} />
    </div>
  )
}

// ✅ Server Component — no 'use client' needed
// components/orders/order-stats.tsx
export function OrderStats({ orders }: { orders: Order[] }) {
  const total = orders.reduce((sum, o) => sum + o.total, 0)
  const pending = orders.filter(o => o.status === 'pending').length

  return (
    <div className="grid grid-cols-3 gap-4">
      <div className="rounded bg-white p-4 shadow">
        <p className="text-sm text-gray-500">Total Orders</p>
        <p className="text-2xl font-bold">{orders.length}</p>
      </div>
      <div className="rounded bg-white p-4 shadow">
        <p className="text-sm text-gray-500">Revenue</p>
        <p className="text-2xl font-bold">${total.toFixed(2)}</p>
      </div>
      <div className="rounded bg-white p-4 shadow">
        <p className="text-sm text-gray-500">Pending</p>
        <p className="text-2xl font-bold">{pending}</p>
      </div>
    </div>
  )
}

// ✅ Only the actually interactive part is a Client Component
// components/orders/order-filters.tsx
'use client'

import { useRouter, useSearchParams } from 'next/navigation'

export function OrderFilters() {
  const router = useRouter()
  const searchParams = useSearchParams()

  function handleFilter(status: string) {
    const params = new URLSearchParams(searchParams.toString())
    params.set('status', status)
    router.push(`/orders?${params.toString()}`)
  }

  return (
    <div className="flex gap-2">
      <button onClick={() => handleFilter('all')}>All</button>
      <button onClick={() => handleFilter('pending')}>Pending</button>
      <button onClick={() => handleFilter('completed')}>Completed</button>
    </div>
  )
}
```

## Why It Matters

- **Bundle size**: Only interactive components ship JS to the client
- **Performance**: Server Components are pre-rendered as HTML — instant display
- **Data security**: Server Components can access secrets, databases, and APIs directly
- **Scalability**: As the app grows, most new components don't add to client bundle

Reference: [Next.js - Client Components](https://nextjs.org/docs/app/building-your-application/rendering/client-components)
