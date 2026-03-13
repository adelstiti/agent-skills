---
title: Data Revalidation with ISR
impact: CRITICAL
impactDescription: Balances data freshness with caching performance
tags: data-fetching, revalidation, isr, caching
---

## Data Revalidation with ISR

**Impact: CRITICAL (Balances data freshness with caching performance)**

Configure revalidation strategies to balance data freshness with caching performance. Use time-based revalidation (ISR) for regularly updated data, on-demand revalidation for immediate updates after mutations, and cache tags for granular invalidation.

## Bad Example

```tsx
// ❌ No caching strategy — either stale forever or never cached
export default async function ProductsPage() {
  // ❌ No cache config — behavior is unclear
  const products = await fetch('https://api.example.com/products')
    .then(res => res.json())

  return <ProductList products={products} />
}

// ❌ force-dynamic on everything — no caching benefit
export const dynamic = 'force-dynamic'  // ❌ Every request hits the server

export default async function ProductsPage() {
  const products = await db.product.findMany()
  return <ProductList products={products} />
}

// ❌ Manual cache busting with random query params
const res = await fetch(`/api/products?t=${Date.now()}`)  // ❌ Defeats caching
```

## Good Example

```tsx
// ✅ Time-based revalidation — ISR
export default async function ProductsPage() {
  // ✅ Revalidate every 60 seconds
  const products = await fetch('https://api.example.com/products', {
    next: { revalidate: 60 },
  }).then(res => res.json())

  return <ProductList products={products} />
}

// ✅ Route-level revalidation
export const revalidate = 60  // Revalidate this page every 60 seconds

export default async function ProductsPage() {
  const products = await db.product.findMany({
    orderBy: { createdAt: 'desc' },
  })

  return <ProductList products={products} />
}

// ✅ Cache tags for granular invalidation
export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
  const product = await fetch(`https://api.example.com/products/${id}`, {
    next: { tags: [`product-${id}`] },
  }).then(res => res.json())

  return <ProductDetail product={product} />
}

// ✅ On-demand revalidation in Server Actions
// lib/actions/products.ts
'use server'

import { revalidatePath, revalidateTag } from 'next/cache'

export async function updateProduct(id: string, formData: FormData) {
  await db.product.update({
    where: { id },
    data: { name: formData.get('name') as string },
  })

  // ✅ Revalidate the specific product page
  revalidateTag(`product-${id}`)

  // ✅ Revalidate the products list page
  revalidatePath('/products')
}

export async function deleteProduct(id: string) {
  await db.product.delete({ where: { id } })

  // ✅ Revalidate both the list and the specific product
  revalidatePath('/products')
  revalidateTag(`product-${id}`)
}

// ✅ Webhook-triggered revalidation (API route)
// app/api/webhooks/products/route.ts
import { revalidateTag } from 'next/cache'
import { NextRequest } from 'next/server'

export async function POST(request: NextRequest) {
  const secret = request.headers.get('x-webhook-secret')
  if (secret !== process.env.WEBHOOK_SECRET) {
    return new Response('Unauthorized', { status: 401 })
  }

  const { productId } = await request.json()
  revalidateTag(`product-${productId}`)

  return Response.json({ revalidated: true })
}
```

## Why It Matters

- **Performance**: Cached pages serve instantly, revalidation happens in the background
- **Freshness**: On-demand revalidation ensures data is current after mutations
- **Granularity**: Cache tags allow invalidating specific data without full rebuilds
- **Scalability**: ISR serves pages from cache globally, reducing server load

Reference: [Next.js Caching](https://nextjs.org/docs/app/building-your-application/caching)
