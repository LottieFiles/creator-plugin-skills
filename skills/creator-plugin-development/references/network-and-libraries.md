# Network Requests and External Libraries

## Network Requests

Plugin sandbox code **cannot** make network requests directly. To fetch data from external APIs, make the request from UI code and send the result to the plugin sandbox.

### Step 1: UI Fetches Data

```typescript
// In plugin UI (src/app.tsx)
const response = await fetch(`https://api.iconlibrary.com/icons/${iconId}.svg`);
const svgContent = await response.text();
```

### Step 2: UI Sends Data to Plugin

```typescript
// In plugin UI (src/app.tsx)
parent.postMessage({
  pluginMessage: {
    type: 'import-svg',
    content: svgContent
  }
}, '*');
```

### Step 3: Plugin Receives and Processes

```typescript
// In plugin sandbox (plugin/plugin.ts)
creator.ui.onMessage(async (msg) => {
  if (msg.type === 'import-svg') {
    const svgLayer = await creator.activeScene.import({
      type: 'SVG',
      content: msg.content
    });
  }
});
```

### Step 4: UI Listens for Response

```typescript
// In plugin UI (src/app.tsx)
window.addEventListener('message', (event) => {
  if (
    event.data.pluginMessage &&
    event.data.pluginMessage.type === 'import-success'
  ) {
    setGenerating(false);
  }
});
```

## External Libraries

### Using Libraries with Bundlers (React Template)

The React template uses Vite for bundling. Install npm packages normally:

```bash
pnpm install react-colorful
```

Then import and use in UI components:

```typescript
import { HexColorPicker } from 'react-colorful';

function ColorSelector() {
  const [color, setColor] = useState('#000000');

  const handleChange = (newColor: string) => {
    setColor(newColor);
    parent.postMessage({
      pluginMessage: { type: 'set-color', color: newColor }
    }, '*');
  };

  return <HexColorPicker color={color} onChange={handleChange} />;
}
```

### Using Libraries with Plain HTML/JS

Include libraries via CDN script tags:

```html
<script src="https://cdn.jsdelivr.net/npm/@simonwep/pickr/dist/pickr.min.js"></script>
```

## Key Constraint

External libraries can only be used in UI code (`src/`). The plugin sandbox (`plugin/`) runs in isolation and cannot import external packages at runtime.