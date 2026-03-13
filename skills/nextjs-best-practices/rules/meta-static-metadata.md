---
title: Static and Dynamic Metadata
impact: MEDIUM
impactDescription: Type-safe SEO with automatic head management
tags: metadata, seo, opengraph, head
---

## Static and Dynamic Metadata

**Impact: MEDIUM (Type-safe SEO with automatic head management)**

Use the Metadata API for SEO instead of manually managing `<head>` tags. Export a `metadata` object for static pages or a `generateMetadata` function for dynamic pages. Next.js automatically manages `<title>`, `<meta>`, and Open Graph tags.

## Bad Example

```tsx
// ❌ Manual head tags — no type safety, easy to forget
'use client'

import Head from 'next/head'  // ❌ Pages Router pattern

export default function ProductPage({ product }) {
  return (
    <>
      {/* ❌ Not type-safe, no autocompletion */}
      <Head>
        <title>{product.name} | My Store</title>
        <meta name="description" content={product.description} />
        <meta property="og:title" content={product.name} />
        <meta property="og:image" content={product.imageUrl} />
      </Head>
      <div>{/* ... */}</div>
    </>
  )
}

// ❌ Hardcoded metadata scattered across components
export default function AboutPage() {
  return (
    <div>
      <title>About Us</title>  {/* ❌ Not the right way in App Router */}
      <h1>About Us</h1>
    </div>
  )
}
```

## Good Example

```tsx
// ✅ Static metadata — for pages with known content
// app/(marketing)/about/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'About Us',
  description: 'Learn more about our company and mission.',
  openGraph: {
    title: 'About Us | My Store',
    description: 'Learn more about our company and mission.',
    images: ['/images/about-og.jpg'],
  },
}

export default function AboutPage() {
  return <h1>About Us</h1>
}

// ✅ Dynamic metadata — for pages with dynamic content
// app/(app)/products/[id]/page.tsx
import type { Metadata } from 'next'
import { db } from '@/lib/db'
import { notFound } from 'next/navigation'

type Props = {
  params: Promise<{ id: string }>
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { id } = await params
  const product = await db.product.findUnique({ where: { id } })

  if (!product) return { title: 'Product Not Found' }

  return {
    title: product.name,
    description: product.description.slice(0, 160),
    openGraph: {
      title: `${product.name} | My Store`,
      description: product.description.slice(0, 160),
      images: [product.imageUrl],
      type: 'website',
    },
    twitter: {
      card: 'summary_large_image',
      title: product.name,
      description: product.description.slice(0, 160),
      images: [product.imageUrl],
    },
  }
}

export default async function ProductPage({ params }: Props) {
  const { id } = await params
  const product = await db.product.findUnique({ where: { id } })
  if (!product) notFound()

  return <ProductDetail product={product} />
}

// ✅ Root layout with default metadata
// app/layout.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: {
    default: 'My Store',
    template: '%s | My Store',  // Child pages: "About Us | My Store"
  },
  description: 'The best online store for quality products.',
  metadataBase: new URL('https://mystore.com'),
  robots: {
    index: true,
    follow: true,
  },
}
```

## Why It Matters

- **Type safety**: TypeScript catches typos and missing fields in metadata
- **Automatic deduplication**: Next.js merges metadata from layouts and pages
- **Template system**: Title templates avoid repetitive brand names
- **SEO consistency**: All pages follow the same metadata pattern

Reference: [Next.js Metadata](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)
