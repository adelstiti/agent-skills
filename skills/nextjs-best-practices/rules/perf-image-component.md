---
title: Use next/image for All Images
impact: HIGH
impactDescription: Automatic optimization, lazy loading, and layout shift prevention
tags: performance, image, optimization, core-web-vitals
---

## Use next/image for All Images

**Impact: HIGH (Automatic optimization, lazy loading, and layout shift prevention)**

Always use the `next/image` component instead of `<img>`. It automatically optimizes images (format conversion, resizing), provides lazy loading, prevents Cumulative Layout Shift (CLS), and serves images in modern formats (WebP, AVIF).

## Bad Example

```tsx
// ❌ Raw <img> tag — no optimization
export default function ProductCard({ product }) {
  return (
    <div>
      {/* ❌ Full-size image loaded regardless of viewport */}
      {/* ❌ No lazy loading, no format optimization */}
      {/* ❌ Layout shift when image loads */}
      <img src={product.imageUrl} alt={product.name} />

      {/* ❌ Hardcoded size but still no optimization */}
      <img
        src="/hero-banner.jpg"
        width="1920"
        height="1080"
        alt="Hero"
      />
    </div>
  )
}
```

## Good Example

```tsx
// ✅ next/image — automatic optimization
import Image from 'next/image'

export default function ProductCard({ product }) {
  return (
    <div>
      {/* ✅ Fixed dimensions — prevents layout shift */}
      <Image
        src={product.imageUrl}
        alt={product.name}
        width={400}
        height={300}
        className="rounded-lg object-cover"
      />
    </div>
  )
}

// ✅ Fill mode for responsive images
export function HeroBanner() {
  return (
    <div className="relative h-[400px] w-full">
      <Image
        src="/hero-banner.jpg"
        alt="Welcome to our store"
        fill
        className="object-cover"
        priority  // ✅ Above-the-fold images should use priority
        sizes="100vw"
      />
    </div>
  )
}

// ✅ Responsive sizes for optimal loading
export function ProductGrid({ products }) {
  return (
    <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-4">
      {products.map(product => (
        <div key={product.id} className="relative aspect-square">
          <Image
            src={product.imageUrl}
            alt={product.name}
            fill
            className="rounded-lg object-cover"
            sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 25vw"
          />
        </div>
      ))}
    </div>
  )
}

// ✅ External images — configure in next.config.ts
// next.config.ts
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
        pathname: '/images/**',
      },
    ],
  },
}
```

## Why It Matters

- **Automatic optimization**: Converts to WebP/AVIF, resizes per viewport
- **CLS prevention**: Width/height or fill mode reserves space before image loads
- **Lazy loading**: Images below the fold load on scroll (default behavior)
- **Core Web Vitals**: Directly improves LCP and CLS scores

Reference: [Next.js Image Optimization](https://nextjs.org/docs/app/building-your-application/optimizing/images)
