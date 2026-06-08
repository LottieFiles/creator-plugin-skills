---
name: creator-plugins-ui
description: |
  Use when building or modifying LottieFiles Creator Plugin UIs with
  @lottiefiles/creator-plugins-ui. Covers React components, Tailwind CSS setup,
  theming, dark mode, cn(), Base UI patterns, and common migration pitfalls.
---

# Creator Plugins UI Library

`@lottiefiles/creator-plugins-ui` — 25 accessible components built on Base UI + Tailwind CSS + CVA. Used in plugin UI code (`src/`) only, not the plugin sandbox.

## Quick Setup

Full guide: `references/setup-guide.md`

```bash
npm install @lottiefiles/creator-plugins-ui
```

```typescript
// src/main.tsx — import before local styles
import "@lottiefiles/creator-plugins-ui/styles.css";
```

Components and theme color utility classes (`bg-primary`, `text-foreground`, `border-border`, etc.) work with just `styles.css` — no Tailwind configuration needed.

If you need **additional** custom Tailwind classes beyond what the library provides, set up Tailwind v4:

```bash
npm install --save-dev tailwindcss @tailwindcss/vite
```

Add `tailwindcss()` to `vite.config.ts` plugins, create `src/styles.css` with `@import "tailwindcss"`, and use `@theme {}` for custom colors. Do **not** create `tailwind.config.js` or `postcss.config.js` (v3 patterns). See `references/setup-guide.md` for full instructions.

## Import Pattern

```typescript
import { Button, Input, Select, Slider, cn } from "@lottiefiles/creator-plugins-ui";
```

## Available Components

For props, variants, and API details, read the TypeScript types from the installed package — they include JSDoc comments, explicit union types, and default values:

```
node_modules/@lottiefiles/creator-plugins-ui/dist/components/*/
```

**Components:** Button, Input, Label, Select, Slider, NumberInput, DualNumberInput, Checkbox, Toggle, RadioGroup, Tabs, Dialog, Dropdown, Tooltip, SegmentedControl, SearchBar, Toast, Spinner, Separator, Card, Grid, EmptyState, Badge, AIPromptInput, ThemeProvider

**Compound components** (use sub-component composition): Dialog, Select, Dropdown, Tabs, Card, Tooltip, Toast, RadioGroup

## Compound Component Pattern

Compound components (Dialog, Select, Dropdown, Tabs) use sub-component composition. Use `render` prop for polymorphic rendering (**not** `asChild` — this library uses Base UI, not Radix):

```tsx
<DialogRoot>
  <DialogTrigger render={<Button variant="outline" />}>Delete</DialogTrigger>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>Delete Layer?</DialogTitle>
      <DialogDescription>This cannot be undone.</DialogDescription>
    </DialogHeader>
    <DialogFooter>
      <DialogClose render={<Button variant="outline" />}>Cancel</DialogClose>
      <Button variant="destructive" onClick={handleDelete}>Delete</Button>
    </DialogFooter>
  </DialogContent>
</DialogRoot>
```

## Theming

Full guide: `references/theming-guide.md`

Dark mode: add `class="dark"` to `<html>` — `document.documentElement.classList.add("dark")`.

Runtime overrides with ThemeProvider:

```tsx
<ThemeProvider tokens={{ "--primary": "hsl(260 80% 55%)" }} themeName="custom">
  <App />
</ThemeProvider>
```

Key CSS variables: `--background`, `--foreground`, `--primary`, `--secondary`, `--destructive`, `--muted-foreground`, `--border`, `--ring`.

## cn() Utility

Use `cn()` from the library (not a local `clsx`) — it includes custom Tailwind merge config for theme colors and font sizes:

```tsx
import { cn } from "@lottiefiles/creator-plugins-ui";
<div className={cn("p-4 text-foreground", isActive && "bg-primary")} />
```

## Common Pitfalls

1. **Missing `styles.css` import** — Components render unstyled. Import `@lottiefiles/creator-plugins-ui/styles.css` once in entry file.
2. **Using Tailwind v3 config with v4** — Do not create `tailwind.config.js` or `postcss.config.js`. Tailwind v4 uses `@import "tailwindcss"` + `@theme {}` in CSS and `@tailwindcss/vite` for Vite integration.
3. **Using `asChild` instead of `render`** — Base UI uses `render={<a href="/" />}`, not Radix's `asChild` + `Slot`.
4. **Select empty string values** — `""` is reserved for "no selection". Use `"none"` or similar.
5. **NumberInput is controlled only** — Requires `value` + `onChange`. No uncontrolled mode.
6. **Slider value is always an array** — Pass `[50]` not `50`. Callback returns array too.
7. **Toast requires provider** — `useToast()` throws outside `<ToastProvider>`. Wrap app root.
8. **Dark mode class placement** — `dark` class goes on `<html>` or parent, not on the component.
9. **Wrong `cn` import** — Use `cn` from `@lottiefiles/creator-plugins-ui`, not a local `clsx`.

## Migration from Custom Components

Full before/after examples: `references/setup-guide.md`

1. Replace `@stitches/react` styled components with library components + Tailwind classes
2. Replace `@radix-ui/*` imports with library components
3. Replace `asChild` with `render` prop
4. Replace `data-[state=checked]` → `data-[checked]`, `data-[state=open]` → `data-[open]`, etc.
5. Replace inline colors with CSS variable classes (`text-foreground`, `bg-primary`)
6. Replace custom `cn()`/`clsx()` with library's `cn()`

## Reference Guide

| Reference | Contents |
|---|---|
| `references/setup-guide.md` | Installation, Tailwind config, migration from Stitches/Radix, troubleshooting |
| `references/theming-guide.md` | CSS variables, dark mode, ThemeProvider, custom colors |
