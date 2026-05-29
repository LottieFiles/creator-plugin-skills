# Importing Assets

Plugins can import external assets into the Creator scene using `scene.import()`.

## Supported Formats

| Type | Formats | Returns |
|---|---|---|
| `'LOTTIE'` | Lottie JSON (`.json`), dotLottie (`.lottie`) | `SceneLayer` |
| `'SVG'` | SVG (`.svg`) | `SceneLayer` |
| `'IMAGE'` | PNG, JPEG, WebP | `ImageLayer` |

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

Note: IMAGE type does not support `content` — only `url`.

## Working with Imported Layers

Imported layers behave like any other layer. Position, scale, and animate them:

```typescript
const animation = await scene.import({
  type: 'LOTTIE',
  url: 'https://example.com/animation.json'
});

animation.name = 'My Animation';
animation.position.staticValue = { x: 100, y: 100 };
animation.scale.staticValue = { x: 50, y: 50 };  // 50% scale
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
