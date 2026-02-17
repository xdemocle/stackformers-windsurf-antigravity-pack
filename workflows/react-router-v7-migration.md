---
description: Migrate from Create React App to React Router v7 with Vite, including root.tsx, routes.ts, and routes/ folder structure
auto_execution_mode: 1
---

# React Router v7 Migration Workflow

This workflow documents the complete process for migrating from Create React App (CRA) to React Router v7 with Vite. Based on the rocco-s-realm implementation.

## Prerequisites

- Existing Create React App project
- Node.js/Bun installed
- Git for version control
- Understanding of React Router concepts
- Existing React components and pages

## Steps

### 1. Backup Current Project

Create a backup before starting:

```bash
git add .
git commit -m "chore: backup before react-router v7 migration"
git branch backup/cra-to-router-v7
```

### 2. Install Dependencies

// turbo

```bash
# Install React Router v7 and Vite
bun add react-router
bun add -D vite @vitejs/plugin-react

# Remove CRA dependencies (optional, can be gradual)
bun remove react-scripts
```

Verify in `package.json`:

- `react-router`: ^7.0.0 or higher
- `vite`: ^5.0.0 or higher
- `@vitejs/plugin-react`: Latest

### 3. Create Vite Configuration

Create `vite.config.ts`:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 5173,
    open: true,
  },
  build: {
    outDir: 'dist',
    sourcemap: false,
  },
});
```

### 4. Create Root Entry Point (index.html)

Create `index.html` in project root:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Your App Title</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### 5. Create Main Entry Point (main.tsx)

Create `src/main.tsx`:

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { RouterProvider } from 'react-router';
import router from './router';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

### 6. Create Root Layout Component (root.tsx)

Create `src/root.tsx`:

```typescript
/* eslint-disable react-refresh/only-export-components */
import { Footer } from '@/components/layout/Footer';
import { Navigation } from '@/components/layout/Navigation';
import { Toaster as SonnerToaster } from '@/components/ui/sonner';
import '@/index.css';
import { TooltipProvider } from '@radix-ui/react-tooltip';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import type { LinksFunction } from 'react-router';
import {
  isRouteErrorResponse,
  Links,
  Meta,
  Outlet,
  Scripts,
  ScrollRestoration,
} from 'react-router';
import { Toaster } from 'sonner';
import { ThemeProvider } from './contexts/theme-context';
import { useApp } from './hooks/use-app';

const queryClient = new QueryClient();

export const links: LinksFunction = () => [
  { rel: 'preconnect', href: 'https://fonts.googleapis.com' },
  {
    rel: 'preconnect',
    href: 'https://fonts.gstatic.com',
    crossOrigin: 'anonymous',
  },
  {
    rel: 'stylesheet',
    href: 'https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=Inter:wght@300;400;500;600;700&family=Fira+Code:wght@400;500&display=swap',
  },
  { rel: 'icon', type: 'image/svg+xml', href: '/favicon.svg' },
];

export const meta = () => [
  { charset: 'utf-8' },
  { name: 'viewport', content: 'width=device-width, initial-scale=1' },
  { title: 'Your App Title' },
  { name: 'description', content: 'Your app description' },
];

function Layout({ children }: { children: React.ReactNode }) {
  const { isNavbarHidden } = useApp();

  return (
    <html lang='en'>
      <head>
        <Meta />
        <Links />
        <script
          dangerouslySetInnerHTML={{
            __html: `(function () {
              var theme = localStorage.getItem('theme');
              var isDark = theme === 'dark' || (theme === 'system' || !theme) && window.matchMedia('(prefers-color-scheme: dark)').matches;
              document.documentElement.classList.remove('light', 'dark');
              document.documentElement.classList.add(isDark ? 'dark' : 'light');
            })();`,
          }}
        />
      </head>
      <body>
        <ThemeProvider>
          <QueryClientProvider client={queryClient}>
            <TooltipProvider>
              <div className='min-h-screen flex flex-col'>
                <Navigation isHidden={isNavbarHidden} />
                <main className='flex-1'>{children}</main>
                <Footer />
              </div>
              <Toaster />
              <SonnerToaster />
            </TooltipProvider>
          </QueryClientProvider>
        </ThemeProvider>
        <ScrollRestoration />
        <Scripts />
      </body>
    </html>
  );
}

export default function App() {
  return (
    <Layout>
      <Outlet />
    </Layout>
  );
}

export function ErrorBoundary() {
  const error = useRouteError();

  if (isRouteErrorResponse(error)) {
    return (
      <Layout>
        <div className='flex items-center justify-center min-h-screen'>
          <div className='text-center'>
            <h1 className='text-4xl font-bold'>{error.status}</h1>
            <p className='text-xl text-muted-foreground'>{error.statusText}</p>
          </div>
        </div>
      </Layout>
    );
  }

  return (
    <Layout>
      <div className='flex items-center justify-center min-h-screen'>
        <div className='text-center'>
          <h1 className='text-4xl font-bold'>Error</h1>
          <p className='text-xl text-muted-foreground'>Something went wrong</p>
        </div>
      </div>
    </Layout>
  );
}
```

### 7. Create Routes Configuration (routes.ts)

Create `src/routes.ts`:

```typescript
import type { RouteConfig } from 'react-router';
import { index, layout, route } from 'react-router';
import Root, { ErrorBoundary } from './root';

// Import page components
import Index from './routes/Index';
import About from './routes/About';
import Services from './routes/Services';
import Portfolio from './routes/Portfolio';
import Blog from './routes/Blog';
import Contact from './routes/Contact';

export const routes: RouteConfig[] = [
  layout(Root, [
    index({
      Component: Index,
      ErrorBoundary,
    }),
    route('about', {
      Component: About,
      ErrorBoundary,
    }),
    route('services', {
      Component: Services,
      ErrorBoundary,
    }),
    route('portfolio', {
      Component: Portfolio,
      ErrorBoundary,
    }),
    route('blog', {
      Component: Blog,
      ErrorBoundary,
    }),
    route('contact', {
      Component: Contact,
      ErrorBoundary,
    }),
  ]),
];
```

### 8. Create Router Instance

Create `src/router.ts`:

```typescript
import { createBrowserRouter } from 'react-router';
import { routes } from './routes';

const router = createBrowserRouter(routes);

export default router;
```

### 9. Create Routes Folder Structure

Create the following folder structure:

```markdown
src/routes/
├── Index.tsx          # Home page
├── About.tsx          # About page
├── Services.tsx       # Services page
├── Portfolio.tsx      # Portfolio page
├── Blog.tsx           # Blog page
└── Contact.tsx        # Contact page
```

Each route file should export a default component:

```typescript
// src/routes/About.tsx
export default function About() {
  return <>{/* About page content */}</>;
}
```

### 10. Update Package.json Scripts

Update `package.json` scripts:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint src --ext ts,tsx",
    "format": "prettier --write src"
  }
}
```

### 11. Create TypeScript Configuration

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "resolveJsonModule": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

Create `tsconfig.node.json`:

```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

### 12. Migrate Components

Move all components to `src/components/`:

```markdown
src/components/
├── layout/
│   ├── Navigation.tsx
│   ├── Footer.tsx
│   └── Layout.tsx
├── home/
│   ├── HeroSection.tsx
│   ├── StatsSection.tsx
│   └── ...
├── ui/
│   ├── button.tsx
│   ├── card.tsx
│   └── ...
└── ...
```

### 13. Migrate Styles

Move global styles to `src/index.css`:

```css
@import 'tailwindcss';

@layer base {
  :root {
    --color-primary: 267 73% 67%;
    --color-accent: 347 100% 60%;
    --color-background: 240 13% 8%;
  }
}

@layer components {
  .glass-card {
    @apply bg-white/10 backdrop-blur-xl border border-white/20 rounded-lg;
  }
}
```

### 14. Migrate Context and Hooks

Move all contexts and hooks to `src/contexts/` and `src/hooks/`:

```markdown
src/
├── contexts/
│   ├── theme-context.tsx
│   └── ...
├── hooks/
│   ├── use-app.tsx
│   └── ...
└── ...
```

### 15. Update Imports

Update all imports to use the `@` alias:

**Before:**

```typescript
import { Button } from '../components/ui/button';
import { useApp } from '../hooks/use-app';
```

**After:**

```typescript
import { Button } from '@/components/ui/button';
import { useApp } from '@/hooks/use-app';
```

### 16. Test Development Server

```bash
bun dev
```

Verify:

- App starts without errors
- All routes are accessible
- Navigation works correctly
- Components render properly

### 17. Build for Production

```bash
bun build
```

Verify:

- Build completes without errors
- Output is in `dist/` folder
- No console errors or warnings

### 18. Test Production Build

```bash
bun preview
```

Verify:

- Production build runs correctly
- All routes work
- No missing assets or styles

### 19. Run Linting and Formatting

```bash
bun lint
bun format
```

Address any linting issues.

### 20. Update Documentation

Update project documentation:

```markdown
## Project Structure
```

src/
├── components/ # React components
│ ├── layout/ # Layout components
│ ├── home/ # Homepage components
│ └── ui/ # Reusable UI components
├── contexts/ # React contexts
├── hooks/ # Custom hooks
├── routes/ # Page components
├── types/ # TypeScript types
├── lib/ # Utility functions
├── index.css # Global styles
├── main.tsx # Entry point
├── root.tsx # Root layout
├── routes.ts # Route configuration
└── router.ts # Router instance

````markdown

## Development

```bash
bun dev      # Start dev server
bun build    # Build for production
bun preview  # Preview production build
bun lint     # Run linter
bun format   # Format code
````

## Routing

Routes are defined in `src/routes.ts` and page components are in `src/routes/`.

Add new routes by:

1. Creating a new file in `src/routes/`
2. Adding a route entry in `src/routes.ts`

````markdown

## Complete File Examples

### Complete routes.ts

```typescript
import type { RouteConfig } from 'react-router';
import { index, layout, route } from 'react-router';
import Root, { ErrorBoundary } from './root';
import Index from './routes/Index';
import About from './routes/About';
import Services from './routes/Services';
import Portfolio from './routes/Portfolio';
import Blog from './routes/Blog';
import Contact from './routes/Contact';

export const routes: RouteConfig[] = [
  layout(Root, [
    index({
      Component: Index,
      ErrorBoundary,
    }),
    route('about', {
      Component: About,
      ErrorBoundary,
    }),
    route('services', {
      Component: Services,
      ErrorBoundary,
    }),
    route('portfolio', {
      Component: Portfolio,
      ErrorBoundary,
    }),
    route('blog', {
      Component: Blog,
      ErrorBoundary,
    }),
    route('contact', {
      Component: Contact,
      ErrorBoundary,
    }),
  ]),
];
````

### Complete main.tsx

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { RouterProvider } from 'react-router';
import router from './router';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
);
```

## Common Issues and Solutions

### Issue: Module not found errors

**Solution:** Verify `@` alias is configured in `vite.config.ts` and `tsconfig.json`. Check import paths match folder structure.

### Issue: Styles not loading

**Solution:** Ensure `index.css` is imported in `main.tsx`. Verify Tailwind configuration is correct.

### Issue: Routes not working

**Solution:** Check `routes.ts` configuration. Verify route paths match component file names. Test with `bun dev`.

### Issue: Build fails

**Solution:** Run `bun lint` to check for errors. Verify all imports are correct. Check TypeScript compilation.

### Issue: HMR not working

**Solution:** Verify Vite dev server is running on correct port. Check browser console for errors.

## Migration Checklist

- [ ] Vite and React Router v7 installed
- [ ] `vite.config.ts` created and configured
- [ ] `index.html` created in project root
- [ ] `src/main.tsx` created as entry point
- [ ] `src/root.tsx` created as root layout
- [ ] `src/routes.ts` created with route configuration
- [ ] `src/router.ts` created with router instance
- [ ] `src/routes/` folder created with page components
- [ ] All components migrated to `src/components/`
- [ ] All styles migrated to `src/index.css`
- [ ] All contexts migrated to `src/contexts/`
- [ ] All hooks migrated to `src/hooks/`
- [ ] All imports updated to use `@` alias
- [ ] `tsconfig.json` and `tsconfig.node.json` created
- [ ] `package.json` scripts updated
- [ ] Dev server runs without errors (`bun dev`)
- [ ] Build succeeds (`bun build`)
- [ ] Production preview works (`bun preview`)
- [ ] All routes accessible and working
- [ ] Linting passes (`bun lint`)
- [ ] No console errors or warnings
- [ ] Documentation updated

## Rollback Plan

If critical issues arise:

```bash
# Revert to CRA
git checkout backup/cra-to-router-v7
bun install
bun start
```

## References

- [React Router v7 Documentation](https://reactrouter.com/)
- [Vite Documentation](https://vitejs.dev/)
- [React Router Migration Guide](https://reactrouter.com/upgrading/v6-to-v7)
- [Vite React Plugin](https://github.com/vitejs/vite-plugin-react)
