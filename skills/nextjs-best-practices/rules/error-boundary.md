---
title: Use error.tsx for Error Boundaries
impact: MEDIUM
impactDescription: Automatic error isolation per route segment
tags: error-handling, error-boundary, recovery, resilience
---

## Use error.tsx for Error Boundaries

**Impact: MEDIUM (Automatic error isolation per route segment)**

Place `error.tsx` files in route segments to create automatic React error boundaries. These catch rendering errors, data fetching errors, and errors thrown in Server Components. They isolate failures to the smallest route segment and provide recovery mechanisms.

## Bad Example

```tsx
// ❌ No error handling — app crashes on error
// app/(app)/orders/page.tsx
export default async function OrdersPage() {
  // If this throws, the entire app crashes with a white screen
  const orders = await db.order.findMany()
  return <OrderList orders={orders} />
}

// ❌ Try-catch that swallows errors silently
export default async function OrdersPage() {
  try {
    const orders = await db.order.findMany()
    return <OrderList orders={orders} />
  } catch {
    // ❌ Silent failure — user sees nothing useful
    return <div>Error</div>
  }
}
```

## Good Example

```tsx
// ✅ error.tsx per route segment — automatic error boundary
// app/(app)/orders/error.tsx
'use client'  // Error components must be Client Components

export default function OrdersError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex flex-col items-center justify-center gap-4 py-16">
      <div className="rounded-lg bg-red-50 p-6 text-center">
        <h2 className="text-lg font-semibold text-red-800">
          Failed to load orders
        </h2>
        <p className="mt-2 text-sm text-red-600">
          {error.message || 'An unexpected error occurred'}
        </p>
        <button
          onClick={reset}
          className="mt-4 rounded bg-red-600 px-4 py-2 text-white hover:bg-red-700"
        >
          Try again
        </button>
      </div>
    </div>
  )
}

// ✅ Not found handling with notFound()
// app/(app)/orders/[id]/page.tsx
import { notFound } from 'next/navigation'

export default async function OrderPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const order = await db.order.findUnique({ where: { id } })

  if (!order) {
    notFound()  // ✅ Triggers the nearest not-found.tsx
  }

  return <OrderDetail order={order} />
}

// app/(app)/orders/[id]/not-found.tsx
import Link from 'next/link'

export default function OrderNotFound() {
  return (
    <div className="py-16 text-center">
      <h2 className="text-2xl font-bold">Order not found</h2>
      <p className="mt-2 text-gray-500">
        The order you're looking for doesn't exist.
      </p>
      <Link
        href="/orders"
        className="mt-4 inline-block text-blue-600 hover:underline"
      >
        Back to orders
      </Link>
    </div>
  )
}

// ✅ Global error handler for root layout errors
// app/global-error.tsx
'use client'

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <html>
      <body>
        <div className="flex min-h-screen items-center justify-center">
          <div className="text-center">
            <h2 className="text-2xl font-bold">Something went wrong</h2>
            <button onClick={reset} className="mt-4 rounded bg-black px-4 py-2 text-white">
              Try again
            </button>
          </div>
        </div>
      </body>
    </html>
  )
}
```

## Why It Matters

- **Error isolation**: A failing orders page doesn't crash the sidebar or dashboard
- **Recovery**: The reset function re-renders the segment without a full page reload
- **User experience**: Users see helpful error messages, not white screens
- **Graceful degradation**: The rest of the app continues working

Reference: [Next.js Error Handling](https://nextjs.org/docs/app/building-your-application/routing/error-handling)
