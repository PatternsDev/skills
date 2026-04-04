---
name: vite-bundle-optimization
description: Vite-specific bundle optimization patterns. Use when configuring builds, code splitting, managing dependencies, or troubleshooting slow Vite builds.
paths:
  - "**/*.js"
  - "**/*.ts"
license: MIT
metadata:
  author: patterns.dev
  version: "1.1"
related_skills:
  - "module-pattern"
  - "singleton-pattern"
---

# Vite Bundle Optimization

Production-ready patterns for optimizing bundle size and build performance in Vite + React applications. These patterns leverage Vite's architecture (native ESM in dev, Rollup in production) to deliver smaller, faster bundles.

## When to Use

Reference these patterns when:
- Setting up a new Vite + React project for production
- Analyzing bundle size with `npx vite-bundle-visualizer`
- Build times are slow or bundles are unexpectedly large
- Migrating from webpack/CRA to Vite
- Optimizing Core Web Vitals (LCP, FID/INP, CLS)

## Instructions

- Apply these patterns during project setup, build configuration, and bundle size reviews. When you see large bundles or slow builds, diagnose with `npx vite-bundle-visualizer` and apply the relevant pattern.

## Details

### Overview

Vite uses esbuild for dependency pre-bundling and development transforms, and Rollup for production builds. Understanding this dual architecture is key to optimizing effectively. The patterns below are ordered by impact.

---

### 1. Avoid Barrel File Imports

**Impact: CRITICAL** — Can add 200-800ms to startup and 2-4s to dev server boot.

Barrel files (`index.ts` that re-export from many modules) force bundlers to load the entire module graph even when you only use one export. This is the #1 bundle size issue in React apps.

**Avoid — imports entire library through barrel:**

```tsx
import { Button, TextField } from '@/components'
// Loads ALL components in the barrel, even unused ones

import { Check, X, Menu } from 'lucide-react'
// Loads all 1,500+ icons (~2.8s in dev)
```

**Prefer — direct imports:**

```tsx
import { Button } from '@/components/Button'
import { TextField } from '@/components/TextField'

import Check from 'lucide-react/dist/esm/icons/check'
import X from 'lucide-react/dist/esm/icons/x'
import Menu from 'lucide-react/dist/esm/icons/menu'
```

**Auto-fix with `vite-plugin-barrel`:**

```typescript
// vite.config.ts
import barrel from 'vite-plugin-barrel'

export default defineConfig({
  plugins: [
    react(),
    barrel({
      packages: ['lucide-react', '@mui/material', '@mui/icons-material'],
    }),
  ],
})
```

This transforms barrel imports into direct imports at build time, giving you ergonomic syntax with direct-import performance.

**Commonly affected libraries:** `lucide-react`, `@mui/material`, `@mui/icons-material`, `@tabler/icons-react`, `react-icons`, `@radix-ui/react-*`, `lodash`, `date-fns`, `rxjs`.

---

### 2. Configure Manual Chunk Splitting

**Impact: HIGH** — Better caching, parallel loading, smaller initial bundle.

Vite's default chunking puts all vendor code into one file. Split it so that frequently-changing app code doesn't invalidate the vendor cache.

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // Core React — rarely changes
          'vendor-react': ['react', 'react-dom'],
          // Router — changes infrequently
          'vendor-router': ['react-router-dom'],
          // Data layer — changes occasionally
          'vendor-query': ['@tanstack/react-query'],
          // UI framework — changes with design updates
          'vendor-ui': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
        },
      },
    },
  },
})
```

For more dynamic splitting based on module paths:

```typescript
manualChunks(id) {
  if (id.includes('node_modules')) {
    if (id.includes('react-dom')) return 'vendor-react'
    if (id.includes('react-router')) return 'vendor-router'
    if (id.includes('@tanstack')) return 'vendor-query'
    return 'vendor' // everything else
  }
},
```

---

### 3. Dynamic Imports for Route-Level Code Splitting

**Impact: HIGH** — Load only the code needed for the current page.

Use `React.lazy()` with dynamic imports to split each route into its own chunk.

```tsx
import { lazy, Suspense } from 'react'
import { BrowserRouter, Routes, Route } from 'react-router-dom'

const Home = lazy(() => import('./pages/Home'))
const Dashboard = lazy(() => import('./pages/Dashboard'))
const Settings = lazy(() => import('./pages/Settings'))

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageSkeleton />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  )
}
```

Vite automatically creates separate chunks for each lazy import. Name them for easier debugging:

```tsx
const Dashboard = lazy(() =>
  import(/* webpackChunkName: "dashboard" */ './pages/Dashboard')
)
```

---

### 4. Lazy-Load Heavy Components Below the Fold

**Impact: HIGH** — Reduces initial bundle for faster LCP.

Components that aren't visible on initial load (modals, charts, editors, maps) should be lazy-loaded.

```tsx
import { lazy, Suspense, useState } from 'react'

const RichTextEditor = lazy(() => import('./components/RichTextEditor'))
const ChartPanel = lazy(() => import('./components/ChartPanel'))

function ArticlePage() {
  const [editing, setEditing] = useState(false)

  return (
    <article>
      <h1>Article Title</h1>
      <p>Content visible immediately...</p>

      {editing && (
        <Suspense fallback={<EditorSkeleton />}>
          <RichTextEditor />
        </Suspense>
      )}

      <Suspense fallback={<ChartSkeleton />}>
        <ChartPanel />
      </Suspense>
    </article>
  )
}
```

---

### 5. Defer Third-Party Scripts

**Impact: HIGH** — Analytics, tracking, and widgets shouldn't block rendering.

Load non-critical third-party scripts after the page is interactive.

**Avoid — blocks initial render:**

```tsx
// main.tsx
import * as Sentry from '@sentry/react'
import posthog from 'posthog-js'

Sentry.init({ dsn: '...' })
posthog.init('...')
```

**Prefer — load after hydration/mount:**

```tsx
// main.tsx — defer to idle time
function initThirdParty() {
  import('@sentry/react').then(Sentry => {
    Sentry.init({ dsn: import.meta.env.VITE_SENTRY_DSN })
  })
  import('posthog-js').then(({ default: posthog }) => {
    posthog.init(import.meta.env.VITE_POSTHOG_KEY)
  })
}

if ('requestIdleCallback' in window) {
  requestIdleCallback(initThirdParty)
} else {
  setTimeout(initThirdParty, 2000)
}
```

For external script tags, use `defer` or dynamically inject them:

```typescript
function loadScript(src: string) {
  const script = document.createElement('script')
  script.src = src
  script.async = true
  document.body.appendChild(script)
}
```

---

### 6. Preload Critical Assets on User Intent

**Impact: MEDIUM** — Eliminates perceived latency on navigation.

Start loading a route's code when the user signals intent (hover, focus) rather than waiting for the click.

```tsx
function NavLink({ to, children }: { to: string; children: React.ReactNode }) {
  const preload = () => {
    // Vite creates a module preload for dynamic imports
    switch (to) {
      case '/dashboard':
        import('./pages/Dashboard')
        break
      case '/settings':
        import('./pages/Settings')
        break
    }
  }

  return (
    <Link to={to} onMouseEnter={preload} onFocus={preload}>
      {children}
    </Link>
  )
}
```

For `<link rel="modulepreload">` in the HTML head:

```html
<!-- Preload critical route chunks -->
<link rel="modulepreload" href="/assets/Home-abc123.js" />
```

Vite automatically adds `<link rel="modulepreload">` for entry chunks. Add manual preloads for routes you know users will visit next.

---

### 7. Configure Dependency Pre-Bundling

**Impact: MEDIUM** — Faster dev server startup and page loads.

Vite pre-bundles `node_modules` dependencies using esbuild. Configure it to handle edge cases.

```typescript
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    // Force pre-bundle these (useful for CJS deps or deep imports)
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      '@tanstack/react-query',
      'date-fns/format',
      'date-fns/parseISO',
    ],
    // Skip pre-bundling for these (already ESM, or causes issues)
    exclude: ['@vite-pwa/assets-generator'],
  },
})
```

If you see slow page loads in dev with many small requests, it's usually because a dependency isn't pre-bundled. Add it to `include`.

---

### 8. Enable Compression

**Impact: MEDIUM** — 60-80% smaller transfer sizes.

Vite doesn't compress by default. Add the compression plugin for production.

```typescript
// vite.config.ts
import viteCompression from 'vite-plugin-compression'

export default defineConfig({
  plugins: [
    react(),
    viteCompression({ algorithm: 'gzip' }),
    viteCompression({ algorithm: 'brotliCompress' }),
  ],
})
```

This generates `.gz` and `.br` files alongside your assets. Configure your server (Nginx, Cloudflare, Vercel) to serve them.

---

### 9. Analyze Your Bundle Regularly

**Impact: INFORMATIONAL** — Catch size regressions before they ship.

Run the bundle visualizer after every significant dependency change.

```bash
npx vite-bundle-visualizer
```

Or add it to your build script:

```json
{
  "scripts": {
    "build": "vite build",
    "analyze": "vite build && npx vite-bundle-visualizer"
  }
}
```

**What to look for:**
- Any single chunk > 200KB gzipped — consider splitting
- Duplicate libraries loaded in multiple chunks
- Full library loaded when only a few functions are used
- `node_modules` code that could be dynamically imported

---

### 10. Use `import.meta.env` for Dead Code Elimination

**Impact: LOW-MEDIUM** — Removes unused code paths in production.

Vite replaces `import.meta.env.*` at build time, allowing Rollup to tree-shake dead branches.

```typescript
// This code is completely removed in production
if (import.meta.env.DEV) {
  console.log('Debug info:', data)
  window.__DEBUG_DATA__ = data
}

// Feature flags eliminated at build time
if (import.meta.env.VITE_FEATURE_NEW_DASHBOARD === 'true') {
  // Only included when flag is set
  initNewDashboard()
}
```

Define custom env variables in `.env` files:

```env
# .env.production
VITE_FEATURE_NEW_DASHBOARD=true
VITE_API_URL=https://api.example.com
```

---

### 11. Optimize Images and Static Assets

**Impact: MEDIUM** — Images are typically the largest assets.

Configure asset handling in Vite:

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    assetsInlineLimit: 4096, // Inline assets < 4KB as base64
    rollupOptions: {
      output: {
        assetFileNames: (assetInfo) => {
          // Organize assets by type
          if (/\.(png|jpe?g|gif|svg|webp|avif)$/.test(assetInfo.name ?? '')) {
            return 'images/[name]-[hash][extname]'
          }
          if (/\.(woff2?|ttf|eot)$/.test(assetInfo.name ?? '')) {
            return 'fonts/[name]-[hash][extname]'
          }
          return 'assets/[name]-[hash][extname]'
        },
      },
    },
  },
})
```

Use `vite-plugin-image-optimizer` for automatic image compression:

```typescript
import { ViteImageOptimizer } from 'vite-plugin-image-optimizer'

export default defineConfig({
  plugins: [
    react(),
    ViteImageOptimizer({
      png: { quality: 80 },
      jpeg: { quality: 80 },
      webp: { quality: 80 },
    }),
  ],
})
```

---

### 12. Configure Dev Server Proxy for API Development

**Impact: MEDIUM** — Eliminates CORS issues and simplifies local development.

Vite SPAs typically talk to a separate backend API. Configure `server.proxy` to forward API requests during development, avoiding CORS and matching production URL patterns.

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
      },
      '/auth': {
        target: 'http://localhost:3001',
        changeOrigin: true,
      },
      // WebSocket support for real-time features
      '/ws': {
        target: 'ws://localhost:3001',
        ws: true,
      },
    },
  },
})
```

In your app code, use relative paths (`fetch('/api/users')`) — they hit Vite's dev server which proxies to your backend. In production, configure your reverse proxy (Nginx, Caddy) to do the same routing.

---

### 13. Add PWA Support with `vite-plugin-pwa`

**Impact: MEDIUM** — Offline capability, installability, and cached assets for Vite SPAs.

For SPAs that need offline support or installability, `vite-plugin-pwa` handles service worker generation, precaching, and manifest creation.

```typescript
// vite.config.ts
import { VitePWA } from 'vite-plugin-pwa'

export default defineConfig({
  plugins: [
    react(),
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.svg', 'robots.txt'],
      manifest: {
        name: 'My App',
        short_name: 'App',
        theme_color: '#ffffff',
        icons: [
          { src: '/icon-192.png', sizes: '192x192', type: 'image/png' },
          { src: '/icon-512.png', sizes: '512x512', type: 'image/png' },
        ],
      },
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/api\./,
            handler: 'NetworkFirst',
            options: { cacheName: 'api-cache', expiration: { maxEntries: 50 } },
          },
        ],
      },
    }),
  ],
})
```

Use `registerType: 'autoUpdate'` for apps that should silently update. Use `registerType: 'prompt'` to show users an update notification.

---

### 14. Choose a CSS Strategy

**Impact: MEDIUM** — Vite supports multiple CSS approaches with zero config.

Vite handles CSS Modules, PostCSS, and preprocessors out of the box. Choose based on your needs:

**CSS Modules** — scoped styles, no runtime cost, built into Vite:

```tsx
// Button.module.css → automatically scoped
import styles from './Button.module.css'

function Button({ children }: { children: React.ReactNode }) {
  return <button className={styles.primary}>{children}</button>
}
```

**Tailwind CSS** — utility-first, works with Vite's PostCSS support:

```typescript
// vite.config.ts — no plugin needed, Tailwind uses PostCSS
// Just install tailwindcss and add postcss.config.js

// postcss.config.js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
}
```

**CSS-in-JS considerations:** Libraries like styled-components and Emotion add runtime overhead. For Vite SPAs prioritizing performance, prefer CSS Modules or Tailwind. If you need CSS-in-JS, consider zero-runtime options like Vanilla Extract or Panda CSS.

---

### 15. Set Up the React Compiler as a Vite Plugin

**Impact: HIGH** — Automatic memoization eliminates manual `useMemo`, `useCallback`, and `React.memo`.

The React Compiler analyzes your components and auto-inserts memoization. In a Vite project, add it as a Babel plugin:

```bash
npm install -D babel-plugin-react-compiler
```

```typescript
// vite.config.ts
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: ['babel-plugin-react-compiler'],
      },
    }),
  ],
})
```

Once enabled, you can gradually remove manual `useMemo`, `useCallback`, and `React.memo` calls — the compiler handles them automatically. Verify behavior is preserved by running your test suite after enabling.

The compiler requires React 19. It's opt-in and can be enabled per-file with a `'use memo'` directive if you prefer incremental adoption.

---

## Source

Patterns from [patterns.dev](https://www.patterns.dev/) — Vite-specific optimization guidance for the broader React web engineering community.
