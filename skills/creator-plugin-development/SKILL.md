---
name: creator-plugin-development
description: |
  Use when creating or modifying LottieFiles Creator Plugins with the Creator Plugin API.
  Covers plugin sandbox/UI architecture, message passing, scene graph manipulation, assets,
  storage, animation, and plugin development workflows.
---

# Creator Plugin Development

Creator Plugins extend the LottieFiles Creator animation application. They have a **two-part sandboxed architecture**:

1. **Plugin Sandbox** (`plugin/plugin.ts`) — Runs in isolation with access to the `creator` global API. Can manipulate scenes, layers, shapes, keyframes. **Cannot make network requests.**
2. **UI** (`src/`) — Standard React application rendered in an iframe. Can make network requests via `fetch`. **Cannot access the `creator` API.**

The two parts communicate exclusively via message passing.

## Project Structure

```text
my-plugin/
├── plugin/
│   ├── manifest.json          # Plugin metadata (id, name, apiVersion, entry, ui)
│   ├── plugin.ts              # Sandbox code — has `creator` API access
│   └── [helpers].ts           # Optional helper modules
├── src/
│   ├── main.tsx               # React DOM entry point
│   ├── app.tsx                # Main UI component
│   └── components/            # React components
├── vite.config.ts             # Uses @lottiefiles/vite-plugin-creator
├── tsconfig.json              # Root config with references
├── tsconfig.plugin.json       # Plugin sandbox TypeScript config (no DOM)
├── tsconfig.app.json          # UI TypeScript config (DOM + JSX)
├── index.html                 # Vite app template
└── package.json
```

`tsconfig.plugin.json` compiles sandbox code (no DOM libs). `tsconfig.app.json` compiles UI code (DOM + JSX).

The plugin manifest (`plugin/manifest.json`) defines the plugin's identity and entry points:

```json
{
  "id": "unique-uuid-v4",
  "name": "My Plugin",
  "apiVersion": "1",
  "entry": "plugin.js",
  "ui": "ui.html"
}
```

## Development Commands

```bash
npm create @lottiefiles/creator-plugin@latest my-plugin  # Scaffold a new plugin

# From the plugin directory (e.g., plugins/my-plugin/):
npm run dev                                      # Start dev server with HTTPS hot-reload
npm run build                                    # TypeScript check + Vite production build
npm exec tsc -- -b                               # Type check (run before completing any task)
```

To load in Creator: **Plugins > Develop > New plugin** > enter the localhost URL from `npm run dev`.

## Communication Pattern (Critical)

This is the most common source of bugs. The message wrapping is asymmetric:

### UI to Plugin

```typescript
// In UI code (src/app.tsx) — MUST wrap in pluginMessage object
parent.postMessage(
  { pluginMessage: { type: 'create-shape', color: '#ff0000' } },
  '*'
);
```

### Plugin Receives Message

```typescript
// In plugin sandbox (plugin/plugin.ts) — messages arrive unwrapped
creator.ui.onMessage((msg) => {
  if (msg.type === 'create-shape') {
    // Use creator API here
  }
});
```

### Plugin to UI

```typescript
// In plugin sandbox — no wrapping needed
creator.ui.postMessage({ type: 'shape-created', layerId: layer.id });
```

### UI Receives Message

```typescript
// In UI code — messages arrive wrapped in pluginMessage
window.addEventListener('message', (event) => {
  const message = event.data.pluginMessage;
  if (message?.type === 'shape-created') {
    // Handle response
  }
});
```

### Type-Safe Messages

Define shared message types to catch mismatches at compile time:

```typescript
// shared/types.ts
export type PluginMessage =
  | { type: 'create-shape'; color: string }
  | { type: 'import-svg'; content: string }
  | { type: 'delete-selection' };
```

For request/response tracking, include a `messageId` field.

## Key API Patterns

### Initialize Plugin

```typescript
creator.ui.show({ width: 300, height: 500 });
```

### Scene Access

```typescript
const scene = creator.activeScene;
scene.size;        // { width, height }
scene.duration;    // seconds
scene.framerate;   // FPS
scene.layers;      // ReadonlyArray<Layer>
```

### Create Shapes

```typescript
const layer = creator.activeScene.createShapeLayer();
const rect = layer.createRectangle({ size: { width: 200, height: 150 } });
layer.createFill({ type: 'SOLID', color: { r: 66, g: 133, b: 244 } });
```

### Import Assets

```typescript
// From URL
const anim = await scene.import({ type: 'LOTTIE', url: 'https://...' });
const img = await scene.import({ type: 'IMAGE', url: 'https://...' });
const svg = await scene.import({ type: 'SVG', url: 'https://...' });

// From content string
const svgLayer = await scene.import({ type: 'SVG', content: svgString });
```

LOTTIE and SVG imports return `SceneLayer`. IMAGE imports return `ImageLayer`.

### Animate Properties

```typescript
layer.position.addKeyframes([
  { frame: 0, value: { x: 100, y: 100 } },
  { frame: 60, value: { x: 400, y: 100 } },
]);

// With easing
const easeInOut = { type: 'CUBIC_BEZIER', x1: 0.42, y1: 0, x2: 0.58, y2: 1 };
layer.position.addKeyframes([
  { frame: 0, value: { x: 50, y: 100 }, easing: easeInOut },
  { frame: 60, value: { x: 350, y: 100 } },
]);
```

### Selection

```typescript
const selectedNodes = creator.selection.nodes;

creator.on('selection:nodes', (nodes) => {
  creator.ui.postMessage({ type: 'selection-changed', count: nodes.length });
});
```

### Node Type Checking

Always verify node types before operations:

```typescript
const layers = creator.selection.nodes;
layers.forEach((node) => {
  if (node.type === 'SHAPE_LAYER') {
    // Shape layer operations (has .shapes, .fills, .strokes, .trimPaths)
  } else if (node.type === 'IMAGE_LAYER') {
    // Image layer operations (has .image)
  } else if (node.type === 'SCENE_LAYER') {
    // Scene layer operations (has .scene, .break())
  } else if (node.type === 'TEXT_LAYER') {
    // Text layer operations (has .text, .fontFamily, .fontStyle, .fontSize,
    // .alignment, .fill/.createFill, .stroke/.createStroke)
  }
});
```

## Network Requests

The plugin sandbox **cannot** make `fetch` requests. Use this pattern:

1. **UI fetches** data from external API
2. **UI sends** data to plugin via `parent.postMessage({ pluginMessage: ... }, '*')`
3. **Plugin processes** data and applies to scene

For complete examples, see `references/network-and-libraries.md`.

## Common Pitfalls

1. **Missing `pluginMessage` wrapper** — UI-to-plugin messages MUST be wrapped: `{ pluginMessage: { ... } }`. Plugin-to-UI messages do NOT need wrapping.
2. **Fetching from plugin sandbox** — Network requests only work in UI code. Move `fetch` calls to `src/`.
3. **Using `localStorage`/`sessionStorage`** — The sandboxed iframe blocks browser storage APIs. Use `creator.clientStorage` from plugin code instead.
4. **Not checking node types** — Always verify `node.type` before accessing type-specific properties.
5. **Setting `staticValue` on animated properties** — Setting `staticValue` when keyframes exist will not affect the animation. Clear keyframes first or modify keyframe values directly.
6. **Invisible shapes** — Shapes need a fill or stroke to be visible. After `createRectangle()`, call `createFill()`.
7. **Scale values are percentages** — `100` = 100% scale (not `1.0`). Use `{ x: 100, y: 100 }` for normal size.
8. **Opacity is 0-100** — Not 0-1. Use `100` for fully opaque.
9. **Color values are 0-255** — RGB channels use the range `{ r: 0-255, g: 0-255, b: 0-255 }`.
10. **Not calling `creator.ui.show()` early** — Call it at the top of `plugin.ts`, before setting up message handlers.
11. **Sending messages before UI is ready** — `creator.ui.postMessage()` right after `creator.ui.show()` will be dropped because the iframe hasn't loaded. Use a "ui-ready" handshake: have the UI send `{ type: 'ui-ready' }` on mount, then send data from the plugin only after receiving that message.

## Verification Checklist

Before considering a task complete:

- [ ] Run `npm exec tsc -- -b` — fix all type errors
- [ ] Test message flow: UI sends > plugin receives > plugin responds > UI receives
- [ ] Confirm network requests are made from UI code, not plugin sandbox
- [ ] Verify `pluginMessage` wrapping is correct in both directions

## Reference Guide

For deeper information, consult these reference files as needed:

| Reference | When to Consult |
| --- | --- |
| `references/architecture-and-communication.md` | Detailed architecture, complete message passing examples, UI API |
| `references/scene-graph-and-nodes.md` | Scene hierarchy, node types, traversal patterns |
| `references/shapes-styling-animation.md` | Creating shapes, fills/strokes/gradients, keyframes, easing |
| `references/importing-assets.md` | LOTTIE/SVG/IMAGE import formats and patterns |
| `references/storage-and-events.md` | clientStorage, node data, selection events, timeline API |
| `references/network-and-libraries.md` | Fetch-from-UI pattern, using npm packages |
