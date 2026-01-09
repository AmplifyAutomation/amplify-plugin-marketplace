---
name: webapp-dev:nextjs-patterns
description: Use this skill when building Next.js applications. Provides best practices for App Router, server components, data fetching, API routes, and Next.js-specific patterns.
---

# Next.js App Router Patterns

## Project Structure

```
src/
├── app/
│   ├── (auth)/              # Route group for auth pages
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   ├── (dashboard)/         # Route group for authenticated pages
│   │   ├── layout.tsx       # Shared dashboard layout
│   │   ├── page.tsx         # Dashboard home
│   │   └── settings/
│   │       └── page.tsx
│   ├── api/                 # API routes
│   │   ├── auth/
│   │   │   └── [...nextauth]/
│   │   └── health/
│   │       └── route.ts
│   ├── layout.tsx           # Root layout
│   ├── page.tsx             # Home page
│   └── globals.css
├── components/
│   ├── ui/                  # Reusable UI components
│   ├── forms/               # Form components
│   └── layouts/             # Layout components
├── lib/
│   ├── db.ts               # Database client
│   ├── auth.ts             # Auth utilities
│   └── utils.ts            # Helper functions
├── hooks/                   # Custom React hooks
├── types/                   # TypeScript types
└── middleware.ts           # Next.js middleware
```

## Server Components (Default)

Server Components are the default in App Router. Use them for:
- Data fetching
- Accessing backend resources
- Keeping sensitive logic server-side

```tsx
// app/users/page.tsx - Server Component (default)
import { db } from '@/lib/db'

export default async function UsersPage() {
  // Direct database access - runs on server only
  const users = await db.user.findMany()

  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  )
}
```

## Client Components

Use `'use client'` directive when you need:
- Event handlers (onClick, onChange)
- React hooks (useState, useEffect)
- Browser-only APIs

```tsx
// components/Counter.tsx
'use client'

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  )
}
```

## Data Fetching Patterns

### Server-Side Data Fetching

```tsx
// app/posts/page.tsx
async function getPosts() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 } // Cache for 1 hour
  })

  if (!res.ok) throw new Error('Failed to fetch posts')
  return res.json()
}

export default async function PostsPage() {
  const posts = await getPosts()
  return <PostList posts={posts} />
}
```

### Parallel Data Fetching

```tsx
// app/dashboard/page.tsx
async function getUser() { /* ... */ }
async function getStats() { /* ... */ }
async function getNotifications() { /* ... */ }

export default async function DashboardPage() {
  // Fetch in parallel for better performance
  const [user, stats, notifications] = await Promise.all([
    getUser(),
    getStats(),
    getNotifications()
  ])

  return (
    <Dashboard
      user={user}
      stats={stats}
      notifications={notifications}
    />
  )
}
```

### With Loading States

```tsx
// app/posts/loading.tsx
export default function Loading() {
  return <PostsSkeleton />
}

// app/posts/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

## API Routes

```tsx
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { db } from '@/lib/db'

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  const page = parseInt(searchParams.get('page') || '1')

  const users = await db.user.findMany({
    skip: (page - 1) * 10,
    take: 10,
  })

  return NextResponse.json(users)
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json()

    // Validate input
    if (!body.email || !body.name) {
      return NextResponse.json(
        { error: 'Email and name are required' },
        { status: 400 }
      )
    }

    const user = await db.user.create({
      data: { email: body.email, name: body.name }
    })

    return NextResponse.json(user, { status: 201 })
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
```

### Dynamic Route Handlers

```tsx
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await db.user.findUnique({
    where: { id: params.id }
  })

  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    )
  }

  return NextResponse.json(user)
}
```

## Middleware

```tsx
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // Check authentication
  const token = request.cookies.get('token')

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*', '/api/:path*']
}
```

## Server Actions

```tsx
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'
import { db } from '@/lib/db'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const content = formData.get('content') as string

  await db.post.create({
    data: { title, content }
  })

  revalidatePath('/posts')
}

// Usage in component
// app/posts/new/page.tsx
import { createPost } from '../actions'

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <textarea name="content" required />
      <button type="submit">Create Post</button>
    </form>
  )
}
```

## Metadata and SEO

```tsx
// app/layout.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: {
    default: 'My App',
    template: '%s | My App',
  },
  description: 'My awesome application',
  openGraph: {
    title: 'My App',
    description: 'My awesome application',
    images: ['/og-image.png'],
  },
}

// app/posts/[slug]/page.tsx
export async function generateMetadata({
  params
}: {
  params: { slug: string }
}): Promise<Metadata> {
  const post = await getPost(params.slug)

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.image],
    },
  }
}
```

## Environment Variables

```tsx
// Server-only (no NEXT_PUBLIC_ prefix)
const dbUrl = process.env.DATABASE_URL

// Client-accessible (NEXT_PUBLIC_ prefix)
const apiUrl = process.env.NEXT_PUBLIC_API_URL

// Type-safe env (create env.ts)
// lib/env.ts
import { z } from 'zod'

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXT_PUBLIC_APP_URL: z.string().url(),
})

export const env = envSchema.parse(process.env)
```

## Common Patterns

### Protected Routes

```tsx
// app/(dashboard)/layout.tsx
import { redirect } from 'next/navigation'
import { getSession } from '@/lib/auth'

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const session = await getSession()

  if (!session) {
    redirect('/login')
  }

  return (
    <div className="dashboard-layout">
      <Sidebar user={session.user} />
      <main>{children}</main>
    </div>
  )
}
```

### Streaming with Suspense

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react'

export default function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<StatsSkeleton />}>
        <Stats />
      </Suspense>
      <Suspense fallback={<ActivitySkeleton />}>
        <RecentActivity />
      </Suspense>
    </div>
  )
}
```

### next.config.js for Production

```js
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'standalone', // Required for Docker
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.supabase.co',
      },
    ],
  },
  experimental: {
    serverActions: {
      bodySizeLimit: '2mb',
    },
  },
}

module.exports = nextConfig
```
