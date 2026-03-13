---
title: Use Server Actions for Mutations
impact: HIGH
impactDescription: Type-safe mutations with built-in form integration and revalidation
tags: server-actions, mutations, forms, revalidation
---

## Use Server Actions for Mutations

**Impact: HIGH (Type-safe mutations with built-in form integration and revalidation)**

Server Actions are async functions that run on the server, invoked from Client Components or form actions. They replace API routes for data mutations, integrate with useActionState for progressive enhancement, and trigger cache revalidation.

## Bad Example

```tsx
// ❌ API route for every mutation
// app/api/orders/route.ts
export async function POST(request: Request) {
  const body = await request.json()
  // No type safety, manual error handling
  const order = await db.order.create({ data: body })
  return Response.json(order, { status: 201 })
}

// ❌ Client-side fetch for mutation
'use client'
export function CreateOrderForm() {
  const [loading, setLoading] = useState(false)

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    setLoading(true)
    try {
      const res = await fetch('/api/orders', {
        method: 'POST',
        body: JSON.stringify(formData),
      })
      if (!res.ok) throw new Error('Failed')
      // ❌ Manual cache invalidation — error prone
      router.refresh()
    } catch (err) {
      // ❌ Manual error handling
    }
    setLoading(false)
  }

  return <form onSubmit={handleSubmit}>...</form>
}
```

## Good Example

```tsx
// ✅ Server Action with validation and revalidation
// lib/actions/orders.ts
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'
import { z } from 'zod'
import { db } from '@/lib/db'
import { auth } from '@/lib/auth'

const CreateOrderSchema = z.object({
  name: z.string().min(1, 'Name is required').max(255),
  items: z.string().transform((val, ctx) => {
    try {
      const parsed = JSON.parse(val)
      if (!Array.isArray(parsed) || parsed.length === 0) {
        ctx.addIssue({ code: 'custom', message: 'At least one item required' })
        return z.NEVER
      }
      return parsed
    } catch {
      ctx.addIssue({ code: 'custom', message: 'Invalid items format' })
      return z.NEVER
    }
  }),
})

export type CreateOrderState = {
  errors?: { name?: string[]; items?: string[] }
  message?: string
}

export async function createOrder(
  prevState: CreateOrderState,
  formData: FormData,
): Promise<CreateOrderState> {
  // ✅ Authentication check
  const session = await auth()
  if (!session) {
    return { message: 'Unauthorized' }
  }

  // ✅ Input validation with Zod
  const parsed = CreateOrderSchema.safeParse({
    name: formData.get('name'),
    items: formData.get('items'),
  })

  if (!parsed.success) {
    return { errors: parsed.error.flatten().fieldErrors }
  }

  // ✅ Create order
  await db.order.create({
    data: {
      name: parsed.data.name,
      userId: session.user.id,
      items: { create: parsed.data.items },
    },
  })

  // ✅ Revalidate cached data and redirect
  revalidatePath('/orders')
  redirect('/orders')
}

// ✅ Form component using useActionState for progressive enhancement
// components/orders/create-order-form.tsx
'use client'

import { useActionState } from 'react'
import { createOrder, type CreateOrderState } from '@/lib/actions/orders'

export function CreateOrderForm() {
  const [state, formAction, isPending] = useActionState<CreateOrderState, FormData>(
    createOrder,
    { errors: {} },
  )

  return (
    <form action={formAction} className="space-y-4">
      <div>
        <label htmlFor="name" className="block text-sm font-medium">
          Order Name
        </label>
        <input
          id="name"
          name="name"
          type="text"
          className="mt-1 block w-full rounded border px-3 py-2"
        />
        {state.errors?.name && (
          <p className="mt-1 text-sm text-red-600">{state.errors.name[0]}</p>
        )}
      </div>

      <button
        type="submit"
        disabled={isPending}
        className="rounded bg-blue-600 px-4 py-2 text-white disabled:opacity-50"
      >
        {isPending ? 'Creating...' : 'Create Order'}
      </button>

      {state.message && (
        <p className="text-sm text-red-600">{state.message}</p>
      )}
    </form>
  )
}
```

## Why It Matters

- **Type safety**: Server Actions are typed functions, not stringly-typed fetch calls
- **Progressive enhancement**: Forms work without JavaScript (useActionState)
- **Automatic revalidation**: revalidatePath/revalidateTag update cached data
- **Less boilerplate**: No API route, no fetch, no manual error handling

Reference: [Next.js Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
