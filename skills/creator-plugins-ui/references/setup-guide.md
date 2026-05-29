# Setup Guide

Instructions for adding `@lottiefiles/creator-plugins-ui` to a Creator Plugin.

## Prerequisites

- A Creator Plugin project (see the `creator-plugin-development` skill)
- Node.js 18+
- pnpm
- React 18+ (already present in all plugins)

## 1. Install the package

From your plugin directory:

```bash
pnpm add @lottiefiles/creator-plugins-ui
```

## 2. Import the stylesheet

In your plugin's entry file (typically `src/main.tsx`), import the stylesheet **before** local styles:

```typescript
import "@lottiefiles/creator-plugins-ui/styles.css";
import "./styles/index.css"; // your local styles (if any)
```

This provides everything needed: light/dark theme CSS variables, DM Sans font, base resets, pre-built Tailwind utility classes, and typography utilities.

Theme color utility classes like `bg-primary`, `text-foreground`, `border-border`, `bg-card`, `text-muted-foreground`, etc. **already work** from just this import — no Tailwind configuration needed.

## 3. Start using components

```typescript
import { Button, Input, Select, cn } from "@lottiefiles/creator-plugins-ui";

function App() {
  return (
    <div className="flex flex-col gap-4 p-4">
      <Input placeholder="Layer name" />
      <Button>Apply</Button>
    </div>
  );
}
```

Components work with just the `styles.css` import — no Tailwind configuration required.

## Custom Tailwind CSS (Optional)

The library's `styles.css` already includes pre-built Tailwind utility classes for all theme colors (`bg-primary`, `text-foreground`, `border-border`, etc.), common spacing, layout, and typography. 

Only set up Tailwind if you need additional custom utility classes beyond what the library provides (e.g., plugin-specific colors, custom spacing values).

### Setup with Tailwind v4

> **Important:** `pnpm add -D tailwindcss` installs Tailwind v4, which uses CSS-based configuration. It does not need `tailwind.config.js` or `postcss.config.js` to work.

1. Install dependencies:

```bash
pnpm add -D tailwindcss @tailwindcss/vite
```

2. Add the Vite plugin to `vite.config.ts`:

```typescript
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [tailwindcss(), react(), /* ...other plugins */],
});
```

3. Create `src/styles.css`:

```css
@import "tailwindcss";
```

4. Import in `src/main.tsx` (after the library stylesheet):

```typescript
import "@lottiefiles/creator-plugins-ui/styles.css";
import "./styles.css";
```

For custom colors, dark mode, runtime theme overrides, and CSS variable details, see `references/theming-guide.md`.

## Troubleshooting

### Components render without styles
- Verify `import "@lottiefiles/creator-plugins-ui/styles.css"` is in your entry file

### Custom Tailwind classes not working
- Ensure `@tailwindcss/vite` is in your Vite plugins and `tailwindcss` is installed
- Verify `src/styles.css` has `@import "tailwindcss"` and is imported in `main.tsx`
- There's no need to create `tailwind.config.js` or `postcss.config.js` — Tailwind v4 uses CSS-based config
- There's no need to install `postcss` or `autoprefixer` separately — `@tailwindcss/vite` handles everything

### Dark mode or theming issues
- See `references/theming-guide.md` for troubleshooting
