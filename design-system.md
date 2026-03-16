# HelloLeo Design System

## Purpose

Centralized UI components ensure design consistency. When THE USER asks to change button colors or card styles, you modify ONE file (`src/index.css`) instead of hunting through the entire codebase.

## Rule: Check Before Creating

BEFORE writing any styled UI element (button, card, input, modal, table, badge, alert, dropdown, tabs, tooltip, spinner, skeleton) directly in a page:

1. Check if `src/components/ui/` exists and contains the component
2. If YES → import and use it
3. If NO → create the component first, then use it

**NEVER write inline styled UI elements directly in pages.**

## First-Time Setup

When creating the first UI component in a project, ensure:

1. **Dependencies** in package.json: `clsx`, `tailwind-merge`
2. **Helper** `src/lib/utils.ts`:

```tsx
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

1. **CSS Variables** in `src/index.css` (THE SOURCE OF TRUTH for design tokens):

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* Colors */
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --success: 142 76% 36%;
    --success-foreground: 210 40% 98%;
    --warning: 38 92% 50%;
    --warning-foreground: 222.2 47.4% 11.2%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;

    /* Effects */
    --radius: 0.5rem;
    --border-width: 1px;
    --shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
    --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);

    /* Fonts */
    --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
    --font-heading: 'Inter', ui-sans-serif, system-ui, sans-serif;
  }

  .dark {
    /* Colors */
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --success: 142 76% 36%;
    --success-foreground: 210 40% 98%;
    --warning: 38 92% 50%;
    --warning-foreground: 222.2 47.4% 11.2%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 48%;

    /* Effects */
    --border-width: 1px;
    --shadow: 0 1px 3px 0 rgb(0 0 0 / 0.3), 0 1px 2px -1px rgb(0 0 0 / 0.3);
    --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.3), 0 4px 6px -4px rgb(0 0 0 / 0.3);

    /* Fonts (same as light, but can be overridden if needed) */
    --font-sans: 'Inter', ui-sans-serif, system-ui, sans-serif;
    --font-heading: 'Inter', ui-sans-serif, system-ui, sans-serif;
  }
}

body {
  @apply bg-background text-foreground font-sans;
}
```

1. **Tailwind config** `tailwind.config.js` references CSS variables:

```jsx
export default {
  darkMode: ["class"],
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {
      colors: {
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        card: { DEFAULT: "hsl(var(--card))", foreground: "hsl(var(--card-foreground))" },
        primary: { DEFAULT: "hsl(var(--primary))", foreground: "hsl(var(--primary-foreground))" },
        secondary: { DEFAULT: "hsl(var(--secondary))", foreground: "hsl(var(--secondary-foreground))" },
        muted: { DEFAULT: "hsl(var(--muted))", foreground: "hsl(var(--muted-foreground))" },
        accent: { DEFAULT: "hsl(var(--accent))", foreground: "hsl(var(--accent-foreground))" },
        destructive: { DEFAULT: "hsl(var(--destructive))", foreground: "hsl(var(--destructive-foreground))" },
        success: { DEFAULT: "hsl(var(--success))", foreground: "hsl(var(--success-foreground))" },
        warning: { DEFAULT: "hsl(var(--warning))", foreground: "hsl(var(--warning-foreground))" },
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
      },
      fontFamily: {
        sans: "var(--font-sans)",
        heading: "var(--font-heading)",
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      borderWidth: {
        DEFAULT: "var(--border-width)",
      },
      boxShadow: {
        DEFAULT: "var(--shadow)",
        lg: "var(--shadow-lg)",
      },
    },
  },
  plugins: [],
}
```

1. **Google Fonts import** in `index.html` `<head>` section (required for fonts to load):

```html
<link rel="preconnect" href="<https://fonts.googleapis.com>">
<link rel="preconnect" href="<https://fonts.gstatic.com>" crossorigin>
<link href="<https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap>" rel="stylesheet">
```

**IMPORTANT:** The font family name in the Google Fonts URL must match the font name used in `--font-sans` and `--font-heading` CSS variables. If using a different font (e.g., Poppins, Roboto), update both the Google Fonts import URL and the CSS variables accordingly.

## Design Change Rule

**To change colors/design: edit `src/index.css` ONLY** (not tailwind.config.js, not component files).

- Changes to CSS variables hot reload instantly
- If editing 3+ files for a design change → STOP, you're doing it wrong

## Strict Rules

- ❌ NEVER hardcode colors: `bg-blue-500`, `text-red-600`
- ❌ NEVER hardcode shadows: `shadow-md`, `shadow-xl`
- ❌ NEVER hardcode border-width: `border-2`, `border-4`
- ✅ ALWAYS use tokens: `bg-primary`, `text-destructive`, `shadow`, `shadow-lg`, `border`
- ❌ NEVER duplicate UI code across pages
- ✅ ALWAYS reuse components from `src/components/ui/`

## Component Standards

All UI components must:

- Use `cn()` for className merging
- Accept `className` prop for customization
- Use design tokens only (hsl variables)
- Support variants via props (e.g., `variant="primary"`, `size="md"`)

## Available Components

Create in `src/components/ui/` as needed:

| Component | Variants |
| --- | --- |
| Button | primary, secondary, outline, ghost, destructive + sizes (sm, md, lg) |
| Card | default, bordered, elevated |
| Input, Select, Textarea | error state + sizes |
| Modal | sizes (sm, md, lg, full) |
| Table | striped, bordered, hoverable |
| Badge | primary, secondary, success, warning, destructive |
| Alert | info, success, warning, error |
| Tabs | default, pills |
| Dropdown | - |
| Tooltip | positions (top, bottom, left, right) |
| Spinner | sizes (sm, md, lg) |
| Skeleton | text, circle, rectangle |
| EmptyState | icon, title, description, action |
