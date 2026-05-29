# Theming Guide

How theming works in `@lottiefiles/creator-plugins-ui` — CSS variables, dark mode, runtime overrides, and custom colors.

## Theme CSS Variables

The library's `styles.css` defines CSS variables on `:root` (light) and `.dark` (dark). These are used by all components and available as pre-built Tailwind utility classes (`bg-primary`, `text-foreground`, `border-border`, etc.) — no configuration needed.

## Dark Mode

The library's `styles.css` includes both `:root` (light) and `.dark` (dark) theme variable definitions. Add the `dark` class to `<html>` to activate dark mode:

```typescript
document.documentElement.classList.add("dark");
```

## Matching Creator's interface theme

To automatically match Creator's current theme, read Creator's current theme tokens and pass them to ThemeProvider.

Plugin sandbox:

```typescript
creator.ui.show({ width: 360, height: 500 });

// set the theme on plugin load
creator.ui.onMessage((msg) => {
  if (msg.type === "ui-ready") {
    const { tokens, themeName } = creator.ui.theme;
    creator.ui.postMessage({ type: "change:theme", tokens, themeName });
  }
});

// update the theme when Creator's theme changes
creator.on("change:theme", ({ tokens, themeName }) => {
  creator.ui.postMessage({ type: "change:theme", tokens, themeName });
});
```

Plugin UI:
```typescript
import { useState, useEffect } from "react";
import { ThemeProvider, Button } from "@lottiefiles/creator-plugins-ui";

function App() {
  const [tokens, setTokens] = useState<Record<string, string>>();
  const [themeName, setThemeName] = useState<string>();

  useEffect(() => {
    window.addEventListener("message", (e) => {
      const msg = e.data?.pluginMessage;
      if (msg?.type === "change:theme") {
        setTokens(msg.tokens);
        setThemeName(msg.themeName);
      }
    });
    parent.postMessage({ pluginMessage: { type: "ui-ready" } }, "*");
  }, []);

  return (
    <ThemeProvider tokens={tokens} themeName={themeName}>
      <Button>Themed Button</Button>
    </ThemeProvider>
  );
}
```

## Runtime Theme Overrides

Wrap your app in `ThemeProvider` to override CSS variables at runtime:

```tsx
import { ThemeProvider } from "@lottiefiles/creator-plugins-ui";

<ThemeProvider
  tokens={{
    "--primary": "hsl(260 80% 55%)",
    "--primary-hover": "hsl(260 80% 45%)",
    "--primary-foreground": "hsl(0 0% 100%)",
  }}
>
  <App />
</ThemeProvider>
```

Use `useTheme()` hook to access the current theme name and tokens.