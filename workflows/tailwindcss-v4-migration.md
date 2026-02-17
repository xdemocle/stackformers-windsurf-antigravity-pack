---
description: Upgrade Tailwind CSS from v3 to v4 with complete config porting
auto_execution_mode: 1
---

# Tailwind CSS v3 to v4 Upgrade Workflow

This workflow documents the complete process for upgrading Tailwind CSS from v3 to v4 across any framework. Based on the rocco-s-realm implementation.

## Prerequisites

- Existing Tailwind CSS v3 project with PostCSS configuration
- Bun, npm, or yarn package manager
- Git for version control
- Understanding of Tailwind configuration structure

## Steps

### 1. Backup Current Configuration

Create a backup of your current Tailwind setup:

```bash
git add .
git commit -m "chore: backup before tailwind v3 to v4 upgrade"
git branch backup/tailwind-v3
```

### 2. Update Package Dependencies

// turbo

```bash
# Remove old versions
bun remove tailwindcss postcss autoprefixer

# Install v4 versions
bun add -D tailwindcss@latest postcss autoprefixer
```

Verify in `package.json`:

- `tailwindcss`: ^4.0.0 or higher
- `postcss`: ^8.4.0 or higher
- `autoprefixer`: Latest

### 3. Port tailwind.config.ts to v4 Syntax

#### 3.1 Remove Content Array

**v3 (before):**

```typescript
import type { Config } from 'tailwindcss';

export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      // ...
    },
  },
  plugins: [],
} satisfies Config;
```

**v4 (after):**

```typescript
import type { Config } from 'tailwindcss';

export default {
  // Content auto-detected in v4, no need to specify
  theme: {
    extend: {
      // ...
    },
  },
  plugins: [],
} satisfies Config;
```

#### 3.2 Update Color Definitions to Use CSS Variables

**v3 approach (static colors):**

```typescript
colors: {
  primary: '#6B5CE7',
  accent: '#FF3366',
  background: '#0F0F1A',
}
```

**v4 approach (CSS variables with hsl()):**

```typescript
colors: {
  primary: 'hsl(var(--color-primary) / <alpha-value>)',
  accent: 'hsl(var(--color-accent) / <alpha-value>)',
  background: 'hsl(var(--color-background) / <alpha-value>)',
}
```

#### 3.3 Update Theme Extensions

**v3:**

```typescript
theme: {
  extend: {
    colors: { /* ... */ },
    spacing: { /* ... */ },
    fontSize: { /* ... */ },
    fontFamily: {
      heading: ['Space Grotesk', 'sans-serif'],
      body: ['Inter', 'sans-serif'],
      code: ['Fira Code', 'monospace'],
    },
  },
}
```

**v4 (same structure, but with CSS variable colors):**

```typescript
theme: {
  extend: {
    colors: {
      primary: 'hsl(var(--color-primary) / <alpha-value>)',
      accent: 'hsl(var(--color-accent) / <alpha-value>)',
    },
    spacing: { /* ... */ },
    fontSize: { /* ... */ },
    fontFamily: {
      heading: ['Space Grotesk', 'sans-serif'],
      body: ['Inter', 'sans-serif'],
      code: ['Fira Code', 'monospace'],
    },
  },
}
```

#### 3.4 Remove Deprecated Plugins

Check for deprecated plugins and remove them:

**v3 plugins to remove:**

- `@tailwindcss/forms` (use native form styling in v4)
- `@tailwindcss/typography` (use prose classes directly)
- Custom plugins with v3-specific syntax

**v4 compatible plugins:**

- Keep plugins that have v4 support
- Update plugin versions if needed

### 4. Update PostCSS Configuration

Update `postcss.config.js` or `postcss.config.mjs`:

**v3:**

```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

**v4:**

```javascript
export default {
  plugins: {
    'tailwindcss/nesting': {},
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### 5. Update Global CSS Entry Point

Update your main CSS file (e.g., `src/index.css`, `src/styles/globals.css`):

**v3 approach:**

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --color-primary: 107 92 231;
  --color-accent: 255 51 102;
  --color-background: 15 15 26;
}

.glass-card {
  @apply bg-white/10 backdrop-blur-xl border border-white/20 rounded-lg;
}
```

**v4 approach:**

```css
@import 'tailwindcss';

@layer base {
  :root {
    --color-primary: 107 92 231;
    --color-accent: 255 51 102;
    --color-background: 15 15 26;
  }
}

@layer components {
  .glass-card {
    @apply bg-white/10 backdrop-blur-xl border border-white/20 rounded-lg;
  }
}
```

### 6. Define CSS Variables in HSL Format

Convert your color values to HSL (Hue, Saturation, Lightness) format:

**RGB to HSL conversion:**

- Primary: `#6B5CE7` → `267 73% 67%` (HSL)
- Accent: `#FF3366` → `347 100% 60%` (HSL)
- Background: `#0F0F1A` → `240 13% 8%` (HSL)

Update CSS variables:

```css
@layer base {
  :root {
    /* Light theme */
    --color-primary: 267 73% 67%;
    --color-accent: 347 100% 60%;
    --color-background: 240 13% 8%;
    --color-text: 0 0% 90%;
    --color-text-secondary: 240 5% 63%;
  }

  .dark {
    /* Dark theme (if different) */
    --color-primary: 267 73% 67%;
    --color-accent: 347 100% 60%;
  }
}
```

### 7. Update All CSS Variable References

Search for and update all CSS variable usages:

**Find all references:**

```bash
grep -r "rgb(var(" src/ --include="*.css" --include="*.tsx" --include="*.ts"
grep -r "var(--" src/ --include="*.css" --include="*.tsx" --include="*.ts"
```

**Replace patterns:**

- `rgb(var(--color-primary))` → `hsl(var(--color-primary))`
- `rgb(var(--color-accent) / 0.5)` → `hsl(var(--color-accent) / 0.5)`

**In inline styles:**

```typescript
// Before
style={{ backgroundColor: `rgb(var(--color-primary))` }}

// After
style={{ backgroundColor: `hsl(var(--color-primary))` }}
```

### 8. Update Component Styling

Review and update component classes for v4 compatibility:

**Gradient syntax (unchanged, but verify colors):**

```typescript
className = 'bg-gradient-to-r from-primary to-accent';
```

**Glass-morphism effects:**

```typescript
className = 'glass-card';
// Uses @layer components definition from CSS
```

**Dark mode classes (if using):**

```typescript
// v3 and v4 syntax is the same
className = 'bg-white dark:bg-slate-900';
```

**New v4 syntax for descendant selectors:**

```typescript
// v3
className = '[&_[cmdk-group-heading]]:px-2';

// v4 (new syntax, optional)
className = '**:[[cmdk-group-heading]]:px-2';
```

### 9. Update Theme Context/Provider (if applicable)

If using a theme context for dark mode:

```typescript
// Ensure theme script in root.tsx/index.html matches v4 logic
const theme = localStorage.getItem('theme') || 'system';
const isDark =
  theme === 'dark' ||
  (theme === 'system' &&
    window.matchMedia('(prefers-color-scheme: dark)').matches);
document.documentElement.classList.add(isDark ? 'dark' : 'light');
```

### 10. Test Responsive Design

Test across all breakpoints:

```bash
# Mobile: < 640px
# Tablet: 640px - 1024px
# Desktop: > 1024px
```

Verify:

- All responsive classes work (`sm:`, `md:`, `lg:`, `xl:`, `2xl:`)
- Dark mode toggling works correctly
- Animations and transitions render properly
- Hover states and interactive elements function

### 11. Run Linting and Formatting

```bash
bun lint
bun format
```

Address Tailwind-specific warnings:

- Deprecated class syntax
- Invalid color references
- Unused or conflicting styles

### 12. Build and Test Production

```bash
bun build
```

Verify:

- Build completes without errors
- CSS file size is reasonable
- No missing styles in production
- All pages render correctly
- No console errors

### 13. Visual Regression Testing

Compare before/after appearance:

- Screenshot all pages before and after
- Test all components in isolation
- Verify dark mode appearance
- Check hover states, animations, transitions
- Test on multiple devices/browsers

### 14. Update Documentation

Update project docs:

```markdown
## Tailwind CSS v4

This project uses Tailwind CSS v4 with CSS variables for theming.

### Color System

Colors are defined as CSS variables in HSL format:

- `--color-primary`: Primary brand color
- `--color-accent`: Accent color
- `--color-background`: Background color

### Configuration

See `tailwind.config.ts` for theme extensions and customizations.

### CSS Variables

Global CSS variables are defined in `src/index.css` using `@layer base`.
```

## Configuration Examples

### Complete v4 tailwind.config.ts

```typescript
import type { Config } from 'tailwindcss';

export default {
  theme: {
    extend: {
      colors: {
        primary: 'hsl(var(--color-primary) / <alpha-value>)',
        accent: 'hsl(var(--color-accent) / <alpha-value>)',
        background: 'hsl(var(--color-background) / <alpha-value>)',
        foreground: 'hsl(var(--color-foreground) / <alpha-value>)',
        'muted-foreground':
          'hsl(var(--color-muted-foreground) / <alpha-value>)',
      },
      fontFamily: {
        heading: ['Space Grotesk', 'sans-serif'],
        body: ['Inter', 'sans-serif'],
        code: ['Fira Code', 'monospace'],
      },
      backgroundImage: {
        'hero-gradient':
          'linear-gradient(135deg, hsl(var(--color-primary)) 0%, hsl(var(--color-accent)) 100%)',
      },
      animation: {
        float: 'float 3s ease-in-out infinite',
        'pulse-slow': 'pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite',
      },
      keyframes: {
        float: {
          '0%, 100%': { transform: 'translateY(0px)' },
          '50%': { transform: 'translateY(-20px)' },
        },
      },
    },
  },
  plugins: [],
} satisfies Config;
```

### Complete v4 index.css

```css
@import 'tailwindcss';

@layer base {
  :root {
    --color-primary: 267 73% 67%;
    --color-accent: 347 100% 60%;
    --color-background: 240 13% 8%;
    --color-foreground: 0 0% 90%;
    --color-muted-foreground: 240 5% 63%;
  }

  .dark {
    --color-primary: 267 73% 67%;
    --color-accent: 347 100% 60%;
  }
}

@layer components {
  .glass-card {
    @apply bg-white/10 backdrop-blur-xl border border-white/20 rounded-lg;
  }

  .gradient-text {
    @apply bg-gradient-to-r from-primary to-accent bg-clip-text text-transparent;
  }
}
```

## Common Issues and Solutions

### Issue: Colors not applying

**Solution:** Verify CSS variables are wrapped with `hsl()` in both config and CSS. Check variable names match exactly.

### Issue: Dark mode not working

**Solution:** Ensure `dark` class is applied to root element. Verify dark theme CSS variables are defined. Check theme context logic.

### Issue: Build size increased

**Solution:** Check for unused styles. Verify `@layer` directives are correct. Remove unused plugins.

### Issue: Linting warnings about class syntax

**Solution:** Update to v4 syntax. Use `**:` instead of `[&_:]` for descendant selectors (optional).

### Issue: Animations not working

**Solution:** Verify keyframes are defined in `tailwind.config.ts` under `theme.extend.keyframes`. Check animation names match.

### Issue: Gradient backgrounds not rendering

**Solution:** Use CSS variables in `backgroundImage` config. Ensure variables are defined in `@layer base`.

## Verification Checklist

- [ ] Dependencies updated to v4 (tailwindcss, postcss, autoprefixer)
- [ ] `tailwind.config.ts` updated with v4 syntax
- [ ] `content` array removed from config
- [ ] All colors converted to CSS variables with `hsl()`
- [ ] `postcss.config.js` updated with `tailwindcss/nesting`
- [ ] Global CSS uses `@import "tailwindcss"`
- [ ] CSS variables defined in `@layer base`
- [ ] Component classes defined in `@layer components`
- [ ] All `rgb(var(--` patterns replaced with `hsl(var(--`
- [ ] No deprecated plugins remain
- [ ] Linting passes (`bun lint`)
- [ ] Build succeeds (`bun build`)
- [ ] All pages render correctly
- [ ] Dark mode works (if applicable)
- [ ] Responsive design tested across breakpoints
- [ ] No visual regressions
- [ ] Production build tested

## Rollback Plan

If critical issues arise:

```bash
# Revert to v3
bun remove tailwindcss postcss autoprefixer
bun add -D tailwindcss@3 postcss@8 autoprefixer

# Revert file changes
git checkout src/index.css tailwind.config.ts postcss.config.js

# Verify
bun build
```

## References

- [Tailwind CSS v4 Migration Guide](https://tailwindcss.com/docs/upgrade-guide)
- [Tailwind CSS v4 Features](https://tailwindcss.com/blog/tailwindcss-v4)
- [CSS Variables in Tailwind](https://tailwindcss.com/docs/customizing-colors#using-css-variables)
- [Tailwind Configuration Reference](https://tailwindcss.com/docs/configuration)
