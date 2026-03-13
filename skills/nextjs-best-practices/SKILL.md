---
name: nextjs-best-practices
description: Next.js 15 App Router best practices and patterns. Use when building, reviewing, or optimizing Next.js applications. Triggers on tasks involving App Router, React Server Components, data fetching, caching, routing, middleware, server actions, metadata, or Next.js performance optimization.
license: MIT
metadata:
  author: Next.js Community
  version: "1.0.0"
  nextjsVersion: "15.x"
  reactVersion: "19.x"
---

# Next.js 15 Best Practices

Comprehensive guide for building Next.js 15 applications with App Router. Contains 11 rules across 8 categories for building fast, scalable, production-ready Next.js applications.

## When to Apply

Reference these guidelines when:
- Setting up or structuring a Next.js App Router project
- Creating pages, layouts, and route segments
- Implementing data fetching and caching strategies
- Building Server Components and Client Components
- Using Server Actions for mutations
- Optimizing performance and Core Web Vitals
- Implementing authentication and middleware

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Routing & Layout | CRITICAL | `route-` |
| 2 | Data Fetching & Caching | CRITICAL | `data-` |
| 3 | Server & Client Components | CRITICAL | `comp-` |
| 4 | Server Actions | HIGH | `action-` |
| 5 | Performance & Optimization | HIGH | `perf-` |
| 6 | Metadata & SEO | MEDIUM | `meta-` |
| 7 | Error Handling & Loading | MEDIUM | `error-` |
| 8 | Security & Middleware | MEDIUM | `sec-` |

## Quick Reference

### 1. Routing & Layout (CRITICAL)

- `route-app-directory` - Use App Router directory conventions
- `route-layouts` - Shared layouts for nested routes

### 2. Data Fetching & Caching (CRITICAL)

- `data-server-fetch` - Fetch data in Server Components
- `data-revalidation` - ISR with revalidatePath/revalidateTag

### 3. Server & Client Components (CRITICAL)

- `comp-server-by-default` - Server Components by default
- `comp-client-boundary` - Push 'use client' to leaf components

### 4. Server Actions (HIGH)

- `action-mutations` - Use Server Actions for data mutations

### 5. Performance & Optimization (HIGH)

- `perf-image-component` - Use next/image for all images

### 6. Metadata & SEO (MEDIUM)

- `meta-static-metadata` - Export metadata object for static pages

### 7. Error Handling & Loading (MEDIUM)

- `error-boundary` - Use error.tsx for error boundaries

### 8. Security & Middleware (MEDIUM)

- `sec-middleware` - Use middleware.ts for auth and redirects

## Project Structure

```
app/
├── layout.tsx              # Root layout (required)
├── page.tsx                # Home page
├── loading.tsx             # Global loading UI
├── error.tsx               # Global error boundary
├── not-found.tsx           # Global 404
├── globals.css             # Global styles
├── (marketing)/            # Route group — no URL segment
│   ├── layout.tsx
│   ├── page.tsx            # /
│   ├── about/
│   │   └── page.tsx        # /about
│   └── pricing/
│       └── page.tsx        # /pricing
├── (app)/                  # Route group — authenticated
│   ├── layout.tsx          # Shared dashboard layout
│   ├── dashboard/
│   │   ├── page.tsx        # /dashboard
│   │   └── loading.tsx
│   ├── settings/
│   │   └── page.tsx        # /settings
│   └── orders/
│       ├── page.tsx         # /orders
│       ├── [id]/
│       │   └── page.tsx     # /orders/:id
│       └── new/
│           └── page.tsx     # /orders/new
├── api/
│   └── webhooks/
│       └── route.ts         # API route handler
components/
├── ui/                      # Shared UI components (Client)
├── forms/                   # Form components (Client)
└── layout/                  # Layout components (Server)
lib/
├── actions/                 # Server Actions
├── db.ts                    # Database client
├── auth.ts                  # Auth utilities
└── utils.ts                 # Shared utilities
```

## Essential Patterns

### Server Component with Data Fetching

```tsx
// app/(app)/orders/page.tsx — Server Component (default)
import { getOrders } from '@/lib/actions/orders'
import { OrderList } from '@/components/orders/order-list'

export default async function OrdersPage() {
  const orders = await getOrders()

  return (
    <main>
      <h1 className="text-2xl font-bold">Orders</h1>
      <OrderList orders={orders} />
    </main>
  )
}
```

### Client Component (Pushed to Leaf)

```tsx
// components/orders/order-list.tsx
'use client'

import { useState } from 'react'
import type { Order } from '@/lib/types'

interface OrderListProps {
  orders: Order[]
}

export function OrderList({ orders }: OrderListProps) {
  const [filter, setFilter] = useState('')

  const filtered = orders.filter(o =>
    o.name.toLowerCase().includes(filter.toLowerCase())
  )

  return (
    <div>
      <input
        type="text"
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="Filter orders..."
        className="rounded border px-3 py-2"
      />
      <ul>
        {filtered.map(order => (
          <li key={order.id}>{order.name} — ${order.total}</li>
        ))}
      </ul>
    </div>
  )
}
```

### Server Action with Form

```tsx
// lib/actions/orders.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'
import { z } from 'zod'

const CreateOrderSchema = z.object({
  name: z.string().min(1).max(255),
  items: z.array(z.object({
    productId: z.number().positive(),
    quantity: z.number().int().positive().max(100),
  })).min(1),
})

export async function createOrder(prevState: unknown, formData: FormData) {
  const parsed = CreateOrderSchema.safeParse({
    name: formData.get('name'),
    items: JSON.parse(formData.get('items') as string),
  })

  if (!parsed.success) {
    return { errors: parsed.error.flatten().fieldErrors }
  }

  await db.order.create({ data: parsed.data })

  revalidatePath('/orders')
  redirect('/orders')
}
```

### Layout with Authentication

```tsx
// app/(app)/layout.tsx
import { auth } from '@/lib/auth'
import { redirect } from 'next/navigation'
import { Sidebar } from '@/components/layout/sidebar'

export default async function AppLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const session = await auth()

  if (!session) {
    redirect('/login')
  }

  return (
    <div className="flex min-h-screen">
      <Sidebar user={session.user} />
      <main className="flex-1 p-6">{children}</main>
    </div>
  )
}
```
