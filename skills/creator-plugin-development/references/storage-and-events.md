# Storage, Events, and Timeline

## Storage Options

| Storage | Persisted Where | Saved with File | Limit | Value Types |
|---|---|---|---|---|
| `creator.clientStorage` | User's browser | No | 5 MB per plugin | Boolean, number, string, object, array |
| `node.data` | Animation file | Yes (not exported) | 5 KB per node | Strings only |

## Client Storage

Persists across sessions but is local to the user's browser. Ideal for user preferences, plugin settings, and cached data.

```typescript
// Save data
await creator.clientStorage.set('lastUsedColor', '#FF5733');
await creator.clientStorage.set('preferences', { theme: 'dark', grid: true });

// Retrieve data
const color = await creator.clientStorage.get('lastUsedColor');
const prefs = await creator.clientStorage.get('preferences');

// List all keys
const keys = await creator.clientStorage.keys();

// Check usage
const usedBytes = await creator.clientStorage.usedQuota();

// Delete specific key
await creator.clientStorage.delete('lastUsedColor');

// Clear all plugin data
await creator.clientStorage.clear();
```

Important notes:
- Data is specific to the plugin ID. Changing the ID loses access.
- Data can be inspected via browser DevTools. Avoid storing sensitive data.
- Data may be cleared if the user clears browser data.
- Accessible only from plugin sandbox code (not directly from UI).

## Node Data

Stored on individual nodes within the animation file. Ideal for layer metadata and plugin state specific to the current project.

```typescript
const layer = creator.activeScene.layers[0];

// Store string values (5 KB limit per node)
layer.data.set('customId', 'my-special-layer');

// For complex data, stringify first (values must be strings)
layer.data.set('metadata', JSON.stringify({
  created: Date.now(),
  author: 'Plugin User'
}));

// Retrieve data
const customId = layer.data.get('customId');
const metadata = JSON.parse(layer.data.get('metadata') ?? '{}');

// List keys
const keys = layer.data.keys;  // Note: property, not method

// Check usage
const usedBytes = layer.data.usedQuota;  // Note: property, not method

// Delete
layer.data.delete('customId');

// Clear all plugin data from this node
layer.data.clear();
```

Important notes:
- Saved within Creator's animation file but NOT retained when exported to Lottie.
- Data is specific to the plugin ID.
- Values must be strings. Use `JSON.stringify()`/`JSON.parse()` for complex data.

## Events

### Selection Events

```typescript
// Listen for node selection changes
creator.on('selection:nodes', (nodes) => {
  console.log('Selected nodes:', nodes.length);
  creator.ui.postMessage({
    type: 'selection-changed',
    data: nodes.map(n => ({ id: n.id, name: n.name, type: n.type }))
  });
});

// Listen for keyframe selection changes
creator.on('selection:keyframes', (keyframes) => {
  console.log('Selected keyframes:', keyframes.length);
});

// Remove listener
const handler = (nodes) => { /* ... */ };
creator.on('selection:nodes', handler);
creator.off('selection:nodes', handler);
```

### Selection API

```typescript
// Read current selection
const selectedNodes = creator.selection.nodes;       // ReadonlyArray<Layer | Shape>
const selectedKeyframes = creator.selection.keyframes; // ReadonlyArray<Keyframe<unknown>>
```

## Timeline API

```typescript
// Read state
const currentFrame = creator.timeline.currentFrame;
const isPlaying = creator.timeline.isPlaying;

// Control playback
creator.timeline.play();
creator.timeline.pause();
creator.timeline.goToFrame(60);
```

## Other Global APIs

```typescript
// Open external link
creator.openLink('https://lottiefiles.com');

// Close the plugin
creator.closePlugin();

// User info (requires "user" permission in manifest)
const userId = creator.user?.id;
const userName = creator.user?.name;
const userToken = creator.user?.token;

// Workspace info (requires "workspaces" permission in manifest)
const currentWorkspace = creator.currentWorkspace;  // { id, name }
const allWorkspaces = creator.workspaces;           // Array<{ id, name }>
```
