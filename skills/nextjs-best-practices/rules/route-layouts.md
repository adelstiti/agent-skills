---
title: Shared Layouts for Nested Routes
impact: CRITICAL
impactDescription: Prevents unnecessary re-renders and enables consistent UI structure
tags: routing, layout, nesting, composition
---

## Shared Layouts for Nested Routes

**Impact: CRITICAL (Prevents unnecessary re-renders and enables consistent UI structure)**

Layouts wrap child routes and are preserved across navigations within the same segment. They don't re-render when navigating between child routes, making them ideal for navigation, sidebars, and persistent state.

## Bad Example

```tsx
// ❌ Repeating layout UI in every page
// app/(app)/dashboard/page.tsx
export default function DashboardPage() {
  return (
    <div className="flex">
      <Sidebar />                    {/* Repeated in every page */}
      <main className="flex-1 p-6">
        <Header />                   {/* Repeated in every page */}
        <h1>Dashboard</h1>
        <DashboardContent />
      </main>
    </div>
  )
}

// app/(app)/settings/page.tsx
export default function SettingsPage() {
  return (
    <div className="flex">
      <Sidebar />                    {/* Copy-pasted again */}
      <main className="flex-1 p-6">
        <Header />                   {/* Copy-pasted again */}
        <h1>Settings</h1>
        <SettingsContent />
      </main>
    </div>
  )
}
```

## Good Example

```tsx
// ✅ Shared layout — rendered once, preserved across navigations
// app/(app)/layout.tsx
import { auth } from '@/lib/auth'
import { redirect } from 'next/navigation'
import { Sidebar } from '@/components/layout/sidebar'
import { Header } from '@/components/layout/header'

export default async function AppLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const session = await auth()
  if (!session) redirect('/login')

  return (
    <div className="flex min-h-screen">
      <Sidebar user={session.user} />
      <div className="flex flex-1 flex-col">
        <Header user={session.user} />
        <main className="flex-1 p-6">{children}</main>
      </div>
    </div>
  )
}

// ✅ Pages only render their unique content
// app/(app)/dashboard/page.tsx
export default async function DashboardPage() {
  const stats = await getDashboardStats()
  return (
    <div>
      <h1 className="text-2xl font-bold">Dashboard</h1>
      <DashboardStats stats={stats} />
    </div>
  )
}

// app/(app)/settings/page.tsx
export default async function SettingsPage() {
  const settings = await getUserSettings()
  return (
    <div>
      <h1 className="text-2xl font-bold">Settings</h1>
      <SettingsForm settings={settings} />
    </div>
  )
}

// ✅ Nested layouts for deeper structure
// app/(app)/settings/layout.tsx
import { SettingsTabs } from '@/components/settings/tabs'

export default function SettingsLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div>
      <h1 className="text-2xl font-bold">Settings</h1>
      <SettingsTabs />
      <div className="mt-4">{children}</div>
    </div>
  )
}

// app/(app)/settings/profile/page.tsx → /settings/profile
// app/(app)/settings/billing/page.tsx → /settings/billing
```

## Why It Matters

- **No re-renders**: Layout stays mounted when navigating between child pages
- **DRY**: Sidebar, header, and auth check defined once
- **Performance**: Only the page content re-renders on navigation
- **Composition**: Nested layouts create natural UI hierarchies

Reference: [Next.js Layouts](https://nextjs.org/docs/app/building-your-application/routing/layouts-and-templates)
