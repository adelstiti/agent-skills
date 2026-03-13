---
title: Use Middleware for Auth and Redirects
impact: MEDIUM
impactDescription: Centralized request-level authentication and routing logic
tags: security, middleware, authentication, redirects
---

## Use Middleware for Auth and Redirects

**Impact: MEDIUM (Centralized request-level authentication and routing logic)**

Use `middleware.ts` at the project root for authentication checks, redirects, and request-level logic. Middleware runs at the edge before any rendering happens, making it the ideal place for route protection and request manipulation.

## Bad Example

```tsx
// ❌ Auth checks duplicated in every page
// app/(app)/dashboard/page.tsx
import { auth } from '@/lib/auth'
import { redirect } from 'next/navigation'

export default async function DashboardPage() {
  const session = await auth()
  if (!session) redirect('/login')  // ❌ Duplicated in every protected page
  // ...
}

// app/(app)/settings/page.tsx
export default async function SettingsPage() {
  const session = await auth()
  if (!session) redirect('/login')  // ❌ Copy-pasted everywhere
  // ...
}

// app/(app)/orders/page.tsx
export default async function OrdersPage() {
  const session = await auth()
  if (!session) redirect('/login')  // ❌ Easy to forget in new pages
  // ...
}
```

## Good Example

```tsx
// ✅ Centralized middleware for auth
// middleware.ts (project root)
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import { auth } from '@/lib/auth'

export async function middleware(request: NextRequest) {
  const session = await auth()
  const { pathname } = request.nextUrl

  // Public routes — no auth required
  const publicRoutes = ['/login', '/register', '/forgot-password']
  if (publicRoutes.some(route => pathname.startsWith(route))) {
    // Redirect to dashboard if already authenticated
    if (session) {
      return NextResponse.redirect(new URL('/dashboard', request.url))
    }
    return NextResponse.next()
  }

  // Protected routes — require auth
  if (!session) {
    const loginUrl = new URL('/login', request.url)
    loginUrl.searchParams.set('callbackUrl', pathname)
    return NextResponse.redirect(loginUrl)
  }

  // Add security headers
  const response = NextResponse.next()
  response.headers.set('X-Frame-Options', 'DENY')
  response.headers.set('X-Content-Type-Options', 'nosniff')

  return response
}

// ✅ Configure which routes middleware runs on
export const config = {
  matcher: [
    /*
     * Match all request paths except:
     * - _next/static (static files)
     * - _next/image (image optimization)
     * - favicon.ico (favicon)
     * - public assets
     */
    '/((?!_next/static|_next/image|favicon.ico|public/).*)',
  ],
}

// ✅ Pages no longer need individual auth checks
// app/(app)/dashboard/page.tsx
export default async function DashboardPage() {
  // ✅ Middleware already verified auth — session is guaranteed
  const session = await auth()
  const stats = await getDashboardStats(session!.user.id)

  return <Dashboard stats={stats} />
}

// ✅ Layout-level auth for additional user data
// app/(app)/layout.tsx
import { auth } from '@/lib/auth'

export default async function AppLayout({
  children,
}: {
  children: React.ReactNode
}) {
  // ✅ Session guaranteed by middleware
  const session = await auth()

  return (
    <div className="flex min-h-screen">
      <Sidebar user={session!.user} />
      <main className="flex-1 p-6">{children}</main>
    </div>
  )
}
```

## Why It Matters

- **Single source of truth**: Auth logic defined once, applied to all routes
- **No forgotten checks**: New pages are automatically protected by middleware
- **Performance**: Middleware runs at the edge, before rendering starts
- **Clean pages**: Page components focus on content, not access control

Reference: [Next.js Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)
