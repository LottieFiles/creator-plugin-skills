# Importing Assets

Plugins can import external assets into the Creator scene using `scene.import()`.

## Supported Formats

| Type | Formats | Returns |
|---|---|---|
| `'LOTTIE'` | Lottie JSON (`.json`), dotLottie (`.lottie`) | `SceneLayer` |
| `'SVG'` | SVG (`.svg`) | `SceneLayer` |
| `'IMAGE'` | PNG, JPEG, WebP | `ImageLayer` |
| `'AUDIO'` | MP3, WAV, OGG, M4A, FLAC | `AudioLayer` |

A failed import (unreachable URL, unsupported format, malformed content) rejects. Wrap `scene.import()` in `try`/`catch` and report the error to the UI.

## Importing from URL

```typescript
// Lottie animation
const animation = await creator.activeScene.import({
  type: 'LOTTIE',
  url: 'https://example.com/animation.json'
});

// Image
const image = await creator.activeScene.import({
  type: 'IMAGE',
  url: 'https://example.com/image.png'
});

// SVG
const svg = await creator.activeScene.import({
  type: 'SVG',
  url: 'https://example.com/graphic.svg'
});

// Audio
const audio = await creator.activeScene.import({
  type: 'AUDIO',
  url: 'https://example.com/sound.mp3'
});
```

## Importing from Content String

```typescript
// Lottie JSON string
const animation = await creator.activeScene.import({
  type: 'LOTTIE',
  content: lottieJsonString
});

// SVG markup string
const svg = await creator.activeScene.import({
  type: 'SVG',
  content: '<svg width="100" height="100">...</svg>'
});
```

```typescript
// Audio, as standard base64 or a base64 data: URI
const audio = await creator.activeScene.import({
  type: 'AUDIO',
  content: 'data:audio/mpeg;base64,SUQzBAAAAAAA...'
});
```

Audio content must be standard base64 (`+` and `/`), not URL-safe base64. The plugin sandbox has no `FileReader` or `btoa` for binary data, so read and encode a user-picked audio file in the UI, then post the string to the plugin.

Note: IMAGE type does not support `content` — only `url`.

## Working with Imported Layers

Imported LOTTIE, SVG, and IMAGE layers behave like any other layer. Position, scale, and animate them:

```typescript
const animation = await scene.import({
  type: 'LOTTIE',
  url: 'https://example.com/animation.json'
});

animation.name = 'My Animation';
animation.position.staticValue = { x: 100, y: 100 };
animation.scale.staticValue = { x: 50, y: 50 };  // 50% scale
```

An imported `AudioLayer` has no transform. Set its volume, mute state, and timing instead:

```typescript
const audio = await scene.import({ type: 'AUDIO', url: 'https://example.com/sound.mp3' });

audio.name = 'Background music';
audio.volume.addKeyframes([
  { frame: 0, value: 0 },    // fade in over the first second at 30 fps
  { frame: 30, value: 100 },
]);
audio.shiftTo(15);           // start at frame 15

const seconds = await audio.audio.getDuration();  // number | null
```

## Centering and Scaling Pattern

Common pattern for importing and centering content in the scene:

```typescript
const scene = creator.activeScene;
const sceneSize = scene.size;
const sceneCenter = { x: sceneSize.width / 2, y: sceneSize.height / 2 };

const layer = await scene.import({ type: 'SVG', content: svgString });
layer.name = 'Imported SVG';

// Get the imported content's dimensions
const importedSize = layer.type === 'SCENE_LAYER'
  ? layer.scene.size
  : { width: layer.image.width, height: layer.image.height };

// Scale to fit within target size
const targetSize = { width: 200, height: 200 };
const scale = Math.min(
  targetSize.width / importedSize.width,
  targetSize.height / importedSize.height
);
layer.scale.staticValue = { x: scale * 100, y: scale * 100 };

// Center in scene
const scaledWidth = importedSize.width * scale;
const scaledHeight = importedSize.height * scale;
layer.position.staticValue = {
  x: sceneCenter.x,
  y: sceneCenter.y,
};
```
