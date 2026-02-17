---
description: Implement app-wide dark mode with ThemeContext, root theme script, and navbar theme-toggle
---

# Dark Mode + Theme Toggle Workflow

Global workflow for implementing a robust dark mode system similar to **rocco-s-realm**, including:

- `ThemeContext` + `ThemeProvider`
- Pre-hydration theme script in `root.tsx`
- `useTheme` hook
- `ThemeToggle` button in the navbar

This is framework-agnostic but examples use **React + React Router + Vite + Tailwind v4**.

---

## 1. Define Theme Types

Create `src/types/theme.ts` (or similar):

```ts
export type Theme = 'light' | 'dark' | 'system';
export type ActualTheme = 'light' | 'dark';
```

---

## 2. Implement ThemeContext + Provider

Create `src/contexts/theme-context.tsx`:

```tsx
import React, { createContext, useCallback, useEffect, useState } from 'react';
import type { Theme, ActualTheme } from '@/types/theme';

interface ThemeState {
  theme: Theme; // user preference: 'light' | 'dark' | 'system'
  actualTheme: ActualTheme; // resolved: 'light' | 'dark'
  mounted: boolean; // true after client has initialized from DOM/localStorage
}

export const ThemeContext = createContext<
  | {
      theme: Theme;
      setTheme: (theme: Theme) => void;
      actualTheme: ActualTheme;
      mounted: boolean;
    }
  | undefined
>(undefined);

function resolveTheme(theme: Theme): ActualTheme {
  if (typeof window === 'undefined') return 'light';

  if (theme === 'system') {
    return window.matchMedia('(prefers-color-scheme: dark)').matches
      ? 'dark'
      : 'light';
  }

  return theme;
}

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [state, setState] = useState<ThemeState>({
    theme: 'system',
    actualTheme: 'light',
    mounted: false,
  });

  // 1) Initialize from localStorage + DOM after mount
  useEffect(() => {
    const storedTheme = (localStorage.getItem('theme') as Theme) || 'system';
    const isDark = document.documentElement.classList.contains('dark');

    // eslint-disable-next-line
    setState({
      theme: storedTheme,
      actualTheme: isDark ? 'dark' : 'light',
      mounted: true,
    });
  }, []);

  // 2) Sync DOM class + localStorage when theme changes (after mounted)
  useEffect(() => {
    if (!state.mounted) return;

    const resolved = resolveTheme(state.theme);
    const root = document.documentElement;

    root.classList.remove('light', 'dark');
    root.classList.add(resolved);

    if (resolved !== state.actualTheme) {
      // eslint-disable-next-line
      setState((prev) => ({ ...prev, actualTheme: resolved }));
    }

    localStorage.setItem('theme', state.theme);
  }, [state.theme, state.mounted, state.actualTheme]);

  const setTheme = useCallback((theme: Theme) => {
    setState((prev) => ({ ...prev, theme }));
  }, []);

  return (
    <ThemeContext.Provider
      value={{
        theme: state.theme,
        setTheme,
        actualTheme: state.actualTheme,
        mounted: state.mounted,
      }}
    >
      {children}
    </ThemeContext.Provider>
  );
}
```

---

## 3. Create `useTheme` Hook

Create `src/hooks/use-theme.ts`:

```tsx
import { useContext } from 'react';
import { ThemeContext } from '@/contexts/theme-context';

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return ctx;
}
```

---

## 4. Add Pre-Hydration Theme Script in Root Layout

In React Router apps, ensure theme is set **before** CSS/React hydrate to avoid a flash and hydration mismatch.

In `src/root.tsx` (or your root layout component):

```tsx
<script
  dangerouslySetInnerHTML={{
    __html: `(function () {
      try {
        var theme = localStorage.getItem('theme');
        var isDark =
          theme === 'dark' ||
          ((theme === 'system' || !theme) && window.matchMedia('(prefers-color-scheme: dark)').matches);

        document.documentElement.classList.remove('light', 'dark');
        document.documentElement.classList.add(isDark ? 'dark' : 'light');
      } catch (e) {
        // fail-safe: default to light
        document.documentElement.classList.add('light');
      }
    })();`,
  }}
/>
```

Place this `<script>` in `<head>` of the root HTML/JSX, **before** CSS is loaded.

---

## 5. Wire ThemeProvider Around the App

Wrap your app with `ThemeProvider` high in the tree.

Example (React Router root in `root.tsx`):

```tsx
export default function App() {
  return (
    <html lang='en'>
      <head>
        <Meta />
        <Links />
        {/* theme script here */}
      </head>
      <body>
        <ThemeProvider>
          {/* other providers, e.g. QueryClientProvider, TooltipProvider */}
          <Layout>
            <Outlet />
          </Layout>
        </ThemeProvider>
        <ScrollRestoration />
        <Scripts />
      </body>
    </html>
  );
}
```

For other frameworks (Next.js, Remix, etc.), apply the same idea in their respective root layout file.

---

## 6. Implement `ThemeToggle` Component

Create `src/components/theme-toggle.tsx`:

```tsx
import { Button } from '@/components/ui/button';
import { Moon, Sun } from 'lucide-react';
import { useTheme } from '@/hooks/use-theme';

export function ThemeToggle() {
  const { theme, setTheme, actualTheme, mounted } = useTheme();

  const handleToggle = () => {
    if (theme === 'system') {
      setTheme(actualTheme === 'light' ? 'dark' : 'light');
    } else if (theme === 'light') {
      setTheme('dark');
    } else {
      setTheme('light');
    }
  };

  // Avoid hydration mismatch: do not render icons until theme is fully initialized
  if (!mounted) {
    return (
      <Button variant='ghost' size='icon' className='relative' disabled>
        <span className='size-5.5' />
      </Button>
    );
  }

  const isDark = actualTheme === 'dark';

  return (
    <Button
      variant='ghost'
      size='icon'
      className='relative'
      aria-label={isDark ? 'Switch to light mode' : 'Switch to dark mode'}
      onClick={handleToggle}
    >
      <Sun
        className={`absolute h-5 w-5 transition-all ${
          isDark
            ? 'rotate-90 scale-0 opacity-0'
            : 'rotate-0 scale-100 opacity-100'
        }`}
      />
      <Moon
        className={`absolute h-5 w-5 transition-all ${
          isDark
            ? 'rotate-0 scale-100 opacity-100'
            : '-rotate-90 scale-0 opacity-0'
        }`}
      />
      <span className='sr-only'>Toggle theme</span>
    </Button>
  );
}
```

Key behaviors:

- Uses `theme` (preference) and `actualTheme` (resolved) for correct icon
- Uses `mounted` flag to avoid SSR/CSR mismatch
- Updates preference via `setTheme`

---

## 7. Integrate ThemeToggle into Navbar

In your navigation component (e.g., `src/components/layout/Navigation.tsx`):

```tsx
import { ThemeToggle } from '@/components/theme-toggle';

export function Navigation(/* props */) {
  // ... existing nav logic
  return (
    <header>
      <nav className='...'>
        <div className='flex items-center justify-between h-16 md:h-20'>
          {/* left: logo */}
          {/* center: nav links */}
          <div className='flex items-center gap-2'>
            {/* other actions (e.g., contact button) */}
            <ThemeToggle />
          </div>
        </div>
      </nav>
    </header>
  );
}
```

This keeps the toggle always visible and consistent across routes.

---

## 8. Tailwind + CSS Setup for Themes

In `src/index.css` (Tailwind v4 style):

```css
@import 'tailwindcss';

@layer base {
  :root {
    --color-background: 240 13% 8%;
    --color-foreground: 0 0% 90%;
    --color-primary: 267 73% 67%;
    --color-accent: 347 100% 60%;
    --color-muted-foreground: 240 5% 63%;
  }

  .light {
    /* optional overrides for light mode if different from default */
  }

  .dark {
    --color-background: 240 13% 3%;
    --color-foreground: 0 0% 98%;
  }
}
```

In `tailwind.config.ts`, map colors to CSS variables:

```ts
import type { Config } from 'tailwindcss';

export default {
  theme: {
    extend: {
      colors: {
        background: 'hsl(var(--color-background) / <alpha-value>)',
        foreground: 'hsl(var(--color-foreground) / <alpha-value>)',
        primary: 'hsl(var(--color-primary) / <alpha-value>)',
        accent: 'hsl(var(--color-accent) / <alpha-value>)',
        'muted-foreground':
          'hsl(var(--color-muted-foreground) / <alpha-value>)',
      },
    },
  },
  plugins: [],
} satisfies Config;
```

---

## 9. Behavioural Expectations

After implementing this workflow, any app should:

- Remember user theme preference in `localStorage.theme` (`'light' | 'dark' | 'system'`)
- Resolve `actualTheme` from preference + system setting
- Apply `light` / `dark` class on `<html>` **before** CSS/React load
- Avoid hydration mismatches using `mounted` flag
- Expose an ergonomic `useTheme` hook and `ThemeToggle` component reusable across layouts

---

## 10. Verification Checklist

- [ ] `ThemeContext` and `ThemeProvider` created with `theme`, `actualTheme`, `mounted`
- [ ] `useTheme` hook throws if used outside provider
- [ ] Theme pre-hydration script in root layout/head
- [ ] `ThemeProvider` wraps the entire app
- [ ] `ThemeToggle` uses `useTheme` and respects `mounted`
- [ ] Navbar renders `ThemeToggle` in the right area
- [ ] `localStorage.theme` updates on toggle
- [ ] System theme respected when `theme === 'system'`
- [ ] No hydration warnings in console
- [ ] Dark/light mode visually correct across all pages

---

## Adapting to Other Frameworks

- **Next.js**: put theme script + provider in `app/layout.tsx` or `_document.tsx` (for the script) and `ThemeProvider` in `app/layout.tsx`.
- **Remix**: use a root route with `<html>`/`<head>`/`<body>` similar to this React Router setup.
- **Plain Vite React**: place the pre-hydration script in `index.html`, and keep `ThemeProvider` at the top of `main.tsx` tree.

The core pattern (context + pre-hydration script + toggle + CSS variables) remains the same everywhere.
