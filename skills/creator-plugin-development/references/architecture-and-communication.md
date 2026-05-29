# Architecture and Communication

## Plugin Architecture

Creator plugins are separated into two isolated environments for security:

```text
┌─────────────────────────┐     ┌─────────────────────────┐
│   Plugin Sandbox        │     │   Plugin UI (iframe)     │
│                         │     │                          │
│ - creator API access    │◄───►│ - React/HTML/CSS         │
│ - Scene manipulation    │     │ - Browser APIs           │
│ - NO network access     │     │ - fetch() for network    │
│ - NO DOM access         │     │ - NO creator API access  │
│                         │     │                          │
│ File: plugin/plugin.ts  │     │ Files: src/*.tsx          │
└─────────────────────────┘     └─────────────────────────┘
        ▲
        │ creator API
        ▼
┌─────────────────────────┐
│   LottieFiles Creator   │
│   (Scene graph)         │
└─────────────────────────┘
```

| | Plugin Sandbox | UI iframe |
|---|---|---|
| **Runs** | Plugin code (`plugin.ts`) | Interface (`App.tsx` or HTML) |
| **Has access to** | Creator Plugin APIs | Browser APIs |
| **Can** | Read/modify layers, shapes, keyframes; control timeline; access selection; store plugin data | Render UI with React/Vue/HTML; make network requests; handle user input |
| **Cannot** | Update the DOM; access browser APIs; make network requests | Access plugin APIs; access the animation scene |

## Communication: UI to Plugin

UI sends messages using `parent.postMessage()`. Messages **must** be wrapped in a `pluginMessage` object:

```typescript
// In plugin UI (src/app.tsx)
function App() {
  const createRectangle = () => {
    parent.postMessage(
      { pluginMessage: { type: 'create-rectangle', color: '#00ff00' } },
      '*'
    );
  };

  return <button onClick={createRectangle}>Create Rectangle</button>;
}
```

## Communication: Plugin Receives and Responds

The plugin sandbox listens with `creator.ui.onMessage()`. Messages arrive **unwrapped** (just the inner object):

```typescript
// In plugin sandbox (plugin/plugin.ts)
creator.ui.onMessage((message) => {
  if (message.type === 'create-rectangle') {
    const layer = creator.activeScene.createShapeLayer({
      position: { x: 100, y: 100 }
    });
    layer.createRectangle({ size: { width: 200, height: 200 } });
    layer.createFill({
      type: 'SOLID',
      color: { r: 0, g: 255, b: 0 }
    });

    // Send response back to UI (no wrapping needed)
    creator.ui.postMessage({ type: 'success' });
  }
});
```

## Communication: UI Receives Response

The UI listens with `window.addEventListener('message', ...)`. Messages from the plugin arrive **wrapped** in `pluginMessage`:

```typescript
// In plugin UI (src/app.tsx)
useEffect(() => {
  const handler = (event: MessageEvent) => {
    const message = event.data.pluginMessage;
    if (message?.type === 'success') {
      console.log('Rectangle created!');
    }
  };

  window.addEventListener('message', handler);
  return () => window.removeEventListener('message', handler);
}, []);
```

## Request/Response Pattern with messageId

For correlating responses to specific requests, include a `messageId`:

```typescript
// UI sends with messageId
const messageId = crypto.randomUUID();
parent.postMessage({
  pluginMessage: { type: 'import-svg', content: svgString, messageId }
}, '*');

// Plugin responds with same messageId
creator.ui.onMessage(async (msg) => {
  if (msg.type === 'import-svg') {
    try {
      const layer = await creator.activeScene.import({
        type: 'SVG',
        content: msg.content,
      });
      creator.ui.postMessage({
        type: 'import-success',
        data: { layerId: layer.id },
        messageId: msg.messageId,
      });
    } catch (error) {
      creator.ui.postMessage({
        type: 'import-error',
        data: { error: error instanceof Error ? error.message : 'Unknown error' },
        messageId: msg.messageId,
      });
    }
  }
});
```

## Type-Safe Message Patterns

Define shared message types to prevent mismatches:

```typescript
// shared/types.ts
export type PluginMessage =
  | { type: 'create-shape'; color: string }
  | { type: 'import-svg'; content: string }
  | { type: 'delete-selection' };

// In plugin UI
import type { PluginMessage } from '../shared/types';

function handleCreateShape() {
  const message: PluginMessage = { type: 'create-shape', color: '#ff0000' };
  parent.postMessage({ pluginMessage: message }, '*');
}

// In plugin sandbox
import type { PluginMessage } from '../shared/types';

creator.ui.onMessage((msg: PluginMessage) => {
  if (msg.type === 'create-shape') {
    console.log(msg.color); // TypeScript knows this has .color
  }
});
```

## UI API Reference

### `creator.ui.show(opts?)`

Display the plugin UI window:

```typescript
creator.ui.show({ width: 300, height: 400 });
```

### `creator.ui.resize(opts)`

Resize the plugin UI window (must specify at least width or height):

```typescript
creator.ui.resize({ width: 400, height: 600 });
```

### `creator.ui.postMessage(message)`

Send data from plugin to UI:

```typescript
creator.ui.postMessage({
  type: 'selection-changed',
  count: creator.selection.nodes.length
});
```

### `creator.ui.onMessage(callback)`

Receive messages from UI in plugin code:

```typescript
creator.ui.onMessage((msg) => {
  if (msg.type === 'create-rectangle') {
    // Handle message
  }
});
```

## UI Styling

The UI runs in a standard iframe. Any CSS approach works:

- Inline styles
- CSS files
- Tailwind CSS
- CSS modules
- Styled-components / CSS-in-JS
