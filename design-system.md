# HelloLeo Design System

## Purpose

Centralized UI components ensure design consistency. When THE USER asks to change button colors or card styles, you modify ONE file (`src/index.css`) instead of hunting through the entire codebase.

## Rule: Check Before Creating

BEFORE writing any styled UI element (button, card, input, modal, table, badge, alert, dropdown, tabs, tooltip, spinner, skeleton) directly in a page:

1. Check if `src/components/ui/` exists and contains the component
2. If YES → import and use it
3. If NO → create the component first, then use it

**NEVER write inline styled UI elements directly in pages.**

## Setup

All design system files (CSS variables, tailwind config, cn helper, Google Fonts) are created by the `scaffold_react_project` tool. Do NOT create these files manually.

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
