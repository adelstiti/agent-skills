---
title: Server Components by Default
impact: CRITICAL
impactDescription: Reduces JavaScript bundle size and enables server-side data access
tags: components, server-components, rsc, rendering
---

## Server Components by Default

**Impact: CRITICAL (Reduces JavaScript bundle size and enables server-side data access)**

In the App Router, all components are React Server Components (RSC) by default. They render on the server, ship zero JavaScript to the client, and can directly access databases, file systems, and server-only APIs. Only add 'use client' when you need interactivity, browser APIs, or React hooks.

## Bad Example

```tsx
// ❌ Adding 'use client' unnecessarily — ships extra JS to the client
'use client'  // ❌ Not needed! No hooks, no interactivity

import { formatDate } from '@/lib/utils'

export default function OrdersPage() {
  // ❌ Fetching on client with useEffect — shows loading spinner
  const [orders, setOrders] = useState([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch('/api/orders')
      .then(res => res.json())
      .then(data => {
        setOrders(data)
        setLoading(false)
      })
  }, [])

  if (loading) return <Spinner />

  return (
    <ul>
      {orders.map(order => (
        <li key={order.id}>
          {order.name} — {formatDate(order.createdAt)}
        </li>
      ))}
    </ul>
  )
}
```

## Good Example

```tsx
// ✅ Server Component — no 'use client', no useState, no useEffect
// app/(app)/orders/page.tsx
import { db } from '@/lib/db'
import { formatDate } from '@/lib/utils'
import { OrderActions } from '@/components/orders/order-actions'

export default async function OrdersPage() {
  // ✅ Direct database access — no API route needed
  const orders = await db.order.findMany({
    orderBy: { createdAt: 'desc' },
    include: { user: true },
  })

  return (
    <div>
      <h1 className="text-2xl font-bold">Orders</h1>
      <ul className="space-y-4">
        {orders.map(order => (
          <li key={order.id} className="rounded border p-4">
            <div className="flex items-center justify-between">
              <div>
                <p className="font-medium">{order.name}</p>
                <p className="text-sm text-gray-500">
                  {formatDate(order.createdAt)}
                </p>
              </div>
              {/* ✅ Only the interactive part is a Client Component */}
              <OrderActions orderId={order.id} />
            </div>
          </li>
        ))}
      </ul>
    </div>
  )
}

// ✅ Client Component — only for interactive elements
// components/orders/order-actions.tsx
'use client'

import { useState } from 'react'
import { deleteOrder } from '@/lib/actions/orders'

export function OrderActions({ orderId }: { orderId: string }) {
  const [isDeleting, setIsDeleting] = useState(false)

  async function handleDelete() {
    setIsDeleting(true)
    await deleteOrder(orderId)
  }

  return (
    <button
      onClick={handleDelete}
      disabled={isDeleting}
      className="text-red-600 hover:text-red-800"
    >
      {isDeleting ? 'Deleting...' : 'Delete'}
    </button>
  )
}
```

## Why It Matters

- **Smaller bundles**: Server Components ship zero JS to the client
- **Direct data access**: Query databases, read files, call services without API routes
- **SEO**: Content is rendered on the server and sent as HTML
- **Security**: Sensitive logic (tokens, queries) never reaches the client

Reference: [React Server Components](https://react.dev/reference/rsc/server-components)
