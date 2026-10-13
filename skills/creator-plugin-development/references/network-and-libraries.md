# Network Requests and External Libraries

## Network Requests

Plugin sandbox code **cannot** make network requests directly. To fetch data from external APIs, make the request from UI code and send the result to the plugin sandbox.

Every host the UI contacts must be declared in the manifest first. See [Declaring network access](#declaring-network-access).

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

## Declaring Network Access

List every host the plugin UI contacts in `manifest.json`:

```json
{
  "id": "unique-uuid-v4",
  "name": "My Plugin",
  "apiVersion": "1",
  "entry": "plugin.js",
  "ui": "ui.html",
  "networkAccess": {
    "allowedDomains": ["api.example.com", "*.cdn.example.com"],
    "reasoning": "Loads icon sets from the publisher's API."
  }
}
```

- `allowedDomains` is required. Use `[]` for a plugin that makes no network requests.
- `reasoning` explains why the plugin contacts these hosts. It is required when `allowedDomains` has a wildcard such as `*.example.com`.
- Plugins submitted to Creator Extensions must include `networkAccess`.

### Entry format

| Entry | Valid? |
| --- | --- |
| `api.example.com` | Yes: exact host |
| `*.example.com` | Yes: any subdomain at any depth, but **not** `example.com` itself |
| `live.example.com:8443` | Yes: that port only |
| `https://api.example.com` | No: no scheme |
| `api.example.com/v1` | No: no path or query |
| `*` or `*.com` | No: too broad |
| `192.168.0.1`, `localhost` | No: no IP addresses or local hosts |

- If the plugin needs both `example.com` and its subdomains, declare both.
- Use `https` and `wss`. Plain `http` and `ws` are not supported.
- If a request redirects, declare the host it redirects to as well.

### What to declare

Declare the host of every request the UI makes:

- `fetch`, `XMLHttpRequest`, `WebSocket` and `navigator.sendBeacon`
- images, video, audio, fonts and stylesheets

URLs passed to `scene.import({ url })` from the sandbox need a declared host too.

### What to bundle instead

Do not load these from a URL, even from a declared host:

- **Scripts:** `<script src="https://...">`, `import("https://...")`, and workers started from a URL. Bundle every script into `ui.html`.
- **WebAssembly:** bundle the `.wasm` file (see [WebAssembly](#webassembly-dotlottie-and-thorvg-players)).
- **Frames and embeds:** do not use `<iframe>`, `<object>` or `<embed>`.
- **Form posts:** use `fetch` instead of `<form action="...">`.
- **Scripts added at runtime:** do not create `<script>` elements while the plugin runs. Put every script in `ui.html`.
- **`javascript:` URLs:** use `addEventListener` instead of `<a href="javascript:...">`.
- **Navigating the plugin page:** do not set `location.href`, add a `<meta http-equiv="refresh">`, or link to another page. To open a web page, send a message to the sandbox and call `creator.openLink(url)`, which opens it in the user's browser.

### Development

- Dev plugins in the Develop tab follow the same rules.
- Do not declare the dev server or local servers on `localhost` or `127.0.0.1`. They need no declaration during development.
- If the browser console shows `Creator blocked <host>`, add the host to `allowedDomains`, or bundle the resource.

## Never Run Fetched Text as Code

Fetch data, not code. JSON, SVG markup and Lottie JSON are fine. Never pass fetched text to `eval`, `new Function`, or a new `<script>` element.

## Capabilities

Declare these in `manifest.json` when the plugin needs them:

```json
{
  "capabilities": ["codeGeneration"]
}
```

| Capability | Declare it when the plugin uses |
| --- | --- |
| `codeGeneration` | `eval`, `new Function` or the other function constructors, in the UI or the plugin sandbox |

- Some libraries use `new Function` internally. For example, the full `lottie-web` build uses it for expressions. Use the library's build without it, such as `lottie-web/build/player/lottie_light`, or declare `codeGeneration`.
- Declare only what the plugin uses. Creator Extensions reviews each declared capability.

## External Libraries

Bundle every library into the plugin. Do not load a library from a CDN, even from a declared host. Bundle the library's stylesheet too, so the CDN host does not need declaring.

### Using Libraries with Bundlers (React Template)

The React template uses Vite for bundling. Install npm packages normally:

```bash
npm install react-colorful
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

`@lottiefiles/vite-plugin-creator`, which the React template uses, inlines every imported script, stylesheet and asset into `ui.html`, so the built plugin makes no request for them. Two things are not inlined:

- **Files in `public/`.** Vite copies them as they are, so the built plugin still requests them by URL, and Creator blocks the request. Put the file in `src/` and import it: `import logo from './logo.png'`.
- **Full URLs in your code or its libraries**, such as `fetch("https://api.example.com")` or a player's built-in CDN URL for its `.wasm` file. Declare the host for data. For code or a `.wasm` file, bundle the file and pass it to the library (see [WebAssembly](#webassembly-dotlottie-and-thorvg-players)).

### Using Libraries with Plain HTML/JS

Without a bundler, copy the library's built file into `ui.html`:

1. Download the library's minified file, for example `pickr.min.js`, from npm or the project's releases.
2. Paste its contents into an inline `<script>` in `ui.html`, before the script that uses it. Keep the license comment at the top of the file.
3. Paste any stylesheet the library needs into a `<style>` element.

```html
<style>
  /* Pickr nano theme, pasted from @simonwep/pickr/dist/themes/nano.min.css */
</style>
<!-- ... -->
<script>
  /*! Pickr 1.10.2 MIT | https://github.com/simonwep/pickr */
  /* contents of @simonwep/pickr/dist/pickr.min.js */
</script>
<script>
  const pickr = Pickr.create({ el: '#color-picker', theme: 'nano' });
</script>
```

For more than one or two libraries, switch to the React template so Vite bundles them.

### Fonts

- Put font files in `src/` and reference them with a relative path in `@font-face`, in a stylesheet you import, for example `src: url('./fonts/inter.woff2')`. Vite inlines them into `ui.html`. A font in `public/` is not inlined.
- To load fonts from Google Fonts instead, declare both `fonts.googleapis.com` and `fonts.gstatic.com`.

## WebAssembly (dotlottie and ThorVG players)

The dotlottie and ThorVG Lottie players render with a `.wasm` file that is not part of their JavaScript. Installing them from npm does not change this. Unless you give the player another address, it downloads the `.wasm` from a CDN when it starts:

| Package | Downloads its `.wasm` from |
| --- | --- |
| `@lottiefiles/dotlottie-web`, and `@lottiefiles/dotlottie-react`, which uses it | `cdn.jsdelivr.net` or `unpkg.com` |
| `@thorvg/lottie-player` | `unpkg.com` |

A plugin must not load code from another server, so for either player:

1. Bundle the `.wasm` file: import it with Vite's `?url` suffix. `@lottiefiles/vite-plugin-creator` turns an imported `.wasm` into a `data:` URL inside `ui.html`, whether it is imported with `?url`, `?init`, `new URL(…, import.meta.url)` or from an inline worker.
2. Give the player that import as its `.wasm` address: `DotLottie.setWasmUrl(wasmUrl)` for dotlottie, or the `wasmUrl` property for ThorVG. Bundling the file without this step does nothing, because the player still requests the CDN address.

dotlottie player:

```typescript
import { DotLottie } from '@lottiefiles/dotlottie-web';
import wasmUrl from '@lottiefiles/dotlottie-web/dotlottie-player.wasm?url';

DotLottie.setWasmUrl(wasmUrl);
```

`@lottiefiles/dotlottie-react` exports `setWasmUrl` too. The file still comes from `@lottiefiles/dotlottie-web`.

ThorVG player: its package does not export the `.wasm` file, so copy `node_modules/@thorvg/lottie-player/dist/thorvg.wasm` into `src/` and import the copy:

```typescript
import '@thorvg/lottie-player';
import wasmUrl from './thorvg.wasm?url';

const player = document.querySelector('lottie-player');
player.wasmUrl = wasmUrl;
```

## Key Constraint

External libraries can only be used in UI code (`src/`). The plugin sandbox (`plugin/`) runs in isolation and cannot import external packages at runtime.
