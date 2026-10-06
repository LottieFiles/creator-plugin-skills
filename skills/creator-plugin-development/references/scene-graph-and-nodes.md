# Scene Graph and Nodes

## Scene Hierarchy

Creator organizes animation content in a hierarchy:

```text
File
├── Scene (main)
│   ├── ShapeLayer
│   │   ├── Group
│   │   │   ├── Rectangle
│   │   │   └── Ellipse
│   │   └── Star
│   ├── ImageLayer
│   ├── TextLayer
│   ├── NullLayer
│   ├── AudioLayer
│   └── SceneLayer ──references──► Nestable Scene
└── Nestable Scene
    └── ShapeLayer
        └── ...
```

## Scenes

Every Creator file has one or more scenes. A scene defines the canvas, framerate, duration, and contains layers.

```typescript
const scene = creator.activeScene;

// Scene properties
scene.size;              // { width: number, height: number }
scene.backgroundColor;   // { r, g, b } (preview only, not exported)
scene.framerate;         // number (FPS)
scene.duration;          // number (seconds)
scene.isNestableScene;   // boolean
scene.layers;            // ReadonlyArray<Layer>

// Modify scene
scene.size = { width: 1920, height: 1080 };
scene.framerate = 60;
scene.duration = 5;
scene.backgroundColor = { r: 255, g: 0, b: 0 };

// Access all scenes
const allScenes = creator.scenes;

// Create a new scene
const newScene = creator.createScene({
  name: 'New Scene',
  size: { width: 1920, height: 1080 },
  framerate: 60,
  duration: 5
});

// Switch active scene
creator.switchToScene(newScene);
```

## Nested Scenes and Scene Layers

A scene can be nestable (child of another scene via a scene layer):

- **Scene layer** — A layer that references a nestable scene. It behaves like a regular layer but its content comes from the referenced scene.
- Changes to the source scene automatically reflect in all instances.

```typescript
// Access the referenced scene
const sourceScene = sceneLayer.scene;

// Break the connection (converts to regular layer)
sceneLayer.break();
```

## Layer Types

Layers are top-level elements of a scene:

| Type Constant | Interface | Description |
|---|---|---|
| `'SHAPE_LAYER'` | `ShapeLayer` | Contains shapes, fills, strokes, trim paths |
| `'IMAGE_LAYER'` | `ImageLayer` | Contains an image asset |
| `'TEXT_LAYER'` | `TextLayer` | Contains text content |
| `'SCENE_LAYER'` | `SceneLayer` | References another scene |
| `'NULL_LAYER'` | `NullLayer` | Transform-only controller other layers can be parented to |
| `'AUDIO_LAYER'` | `AudioLayer` | Plays an audio asset; has no transform |

New layer types can be added to Creator over time. Handle the types your plugin knows and skip the rest; see [Unknown Node Types](#unknown-node-types).

Every layer has the members of `BaseLayerMixin`:

```typescript
layer.id;              // readonly string
layer.name;            // string (read/write)
layer.type;            // layer type constant
layer.locked;          // boolean
layer.startFrame;      // number
layer.endFrame;        // number
layer.timelineOffset;  // number

layer.clone();         // Duplicate the layer
layer.remove();        // Remove from scene
layer.shiftTo(30);     // Move the layer's timeline to a frame
layer.bringToFront();  // Render order
```

Layers with a transform (every type except `AUDIO_LAYER`) also have the members of `LayerMixin`. `creator.utils.isTransformableLayer(layer)` narrows a layer to these types:

```typescript
layer.visible;         // boolean
layer.focused;         // boolean
layer.blendMode;       // BlendMode

// Animatable transform properties (TransformMixin)
layer.position;        // Animatable<Vector>
layer.rotation;        // Animatable<number>
layer.scale;           // Animatable<Vector>
layer.opacity;         // Animatable<number> (0-100)
layer.skew;            // Animatable<number>
layer.skewAxis;        // Animatable<number>

layer.align('left');   // Align within parent
layer.flip('horizontal'); // Flip direction

// Masks
layer.masks;           // ReadonlyArray<Mask>
layer.createMask({ mode: 'add', pathData, opacity: 100 });

// Mattes
layer.isMatte;         // boolean
layer.matte;           // Matte | undefined
```

### Type-Specific Properties

```typescript
// ShapeLayer — has shapes and styling
if (layer.type === 'SHAPE_LAYER') {
  layer.shapes;      // ReadonlyArray<Shape>
  layer.fills;       // ReadonlyArray<Paint>
  layer.strokes;     // ReadonlyArray<Stroke>
  layer.trimPaths;   // ReadonlyArray<TrimPath>
}

// ImageLayer — has image asset
if (layer.type === 'IMAGE_LAYER') {
  layer.image;       // Image { type, width, height }
}

// TextLayer — has text content and styling
if (layer.type === 'TEXT_LAYER') {
  layer.text;        // string (read/write)
  layer.fontFamily;  // string, e.g. "Roboto"
  layer.fontStyle;   // string, e.g. "Regular", "Bold"
  layer.fontSize;    // number
  layer.alignment;   // 'left' | 'center' | 'right'
  layer.fill;        // SolidPaint | undefined
  layer.stroke;      // TextStroke | undefined
}

// SceneLayer — references another scene
if (layer.type === 'SCENE_LAYER') {
  layer.scene;       // Scene
  layer.break();     // Break connection to source scene
}

// NullLayer — transform-only controller
if (layer.type === 'NULL_LAYER') {
  layer.layers;      // ReadonlyArray<TransformableLayer> parented to this null layer
}

// AudioLayer — plays sound, no transform
if (layer.type === 'AUDIO_LAYER') {
  layer.audio;       // AudioAsset
  layer.muted;       // boolean
  layer.volume;      // Animatable<number> (0-100)
}

// AudioAsset — the sound an audio layer plays (also in creator.assets)
const sound = audioLayer.audio;
sound.id;               // readonly string
sound.name;             // string (read/write)
sound.uri;              // string | null: a base64 `data:` URI or a URL
await sound.getDuration(); // number | null, in seconds
sound.remove();         // removes the asset and every audio layer that plays it
```

Audio layers cannot be a transform parent, a matte source, or a null layer child. `transformParent`, `matte` and `createNullLayer({ layers })` accept only layers with a transform (`TransformableLayer`); passing an audio layer, or a value typed as a plain `Layer`, fails to compile and throws at runtime:

```typescript
shapeLayer.transformParent = nullLayer;          // OK
shapeLayer.transformParent = audioLayer;         // compile error, throws
scene.createNullLayer({ layers: [audioLayer] }); // compile error, throws

for (const layer of scene.layers) {
  if (creator.utils.isTransformableLayer(layer)) {
    layer.transformParent = nullLayer;           // OK: narrowed to TransformableLayer
  }
}
```

## Shape Types

Shapes are children of `ShapeLayer` or `Group`:

| Type Constant | Interface | Specific Properties |
|---|---|---|
| `'RECTANGLE'` | `Rectangle` | `size`, `position`, `roundness` |
| `'ELLIPSE'` | `Ellipse` | `size`, `position` |
| `'POLYGON'` | `Polygon` | `points`, `position`, `rotation`, `outerRadius`, `outerRoundness` |
| `'STAR'` | `Star` | `points`, `position`, `rotation`, `innerRadius`, `outerRadius`, `innerRoundness`, `outerRoundness` |
| `'PATH'` | `Path` | `pathData` |
| `'GROUP'` | `Group` | Contains other shapes; has `shapes`, `fills`, `strokes`, `trimPaths`, `opacity`, `blendMode` |

## Accessing Scene Content

### Via Scene

```typescript
const scene = creator.activeScene;
const layers = scene.layers;

for (const layer of layers) {
  console.log(layer.name, layer.type);
}
```

### Via Selection

```typescript
const selectedNodes = creator.selection.nodes;     // Layers and shapes
const selectedKeyframes = creator.selection.keyframes;
```

## Common Traversal Patterns

### Filter by Type

```typescript
// Get only image layers from selection
const imageLayers = creator.selection.nodes.filter(
  (node): node is ImageLayer => node.type === 'IMAGE_LAYER'
);
```

### Traverse Down (Recursive Shape Visitor)

```typescript
function visitShapes(parent: ShapeLayer | Group, callback: (shape: Shape) => void) {
  for (const shape of parent.shapes) {
    callback(shape);
    if (shape.type === 'GROUP') {
      visitShapes(shape, callback);
    }
  }
}

// Find all shapes in selected layers
const selection = creator.selection.nodes;
const shapes: Shape[] = [];

selection.forEach(node => {
  if (node.type === 'SHAPE_LAYER') {
    visitShapes(node, (shape) => shapes.push(shape));
  }
});
```

### Unknown Node Types

To avoid errors and plugin crashes, your plugin should plan for unexpected node types.

Check `type` before reading type-specific members. For example, audio layers have no `opacity`, so `layer.opacity.keyframes` throws on one.

#### Checking a node's type

Every layer, shape and asset has a `type` string, such as `'SHAPE_LAYER'`, `'RECTANGLE'` or `'AUDIO'`. The [Layer Types](#layer-types) and [Shape Types](#shape-types) tables list the values.

Compare `type` with `if` or `switch`. In TypeScript, this also narrows the node, so the members of that type become available:

```typescript
for (const node of creator.selection.nodes) {
  switch (node.type) {
    case 'SHAPE_LAYER':
      node.fills;        // ShapeLayer members
      break;
    case 'TEXT_LAYER':
      node.text;         // TextLayer members
      break;
    case 'RECTANGLE':
      node.size;         // Rectangle members
      break;
    default:
      // Unsupported or unknown type: provide fallback handling
      break;
  }
}
```

`creator.utils` has type guards for broader groups:

| Guard | Returns `true` for |
|---|---|
| `creator.utils.isLayer(node)` | Any layer |
| `creator.utils.isShape(node)` | Any shape |
| `creator.utils.isTransformableLayer(node)` | Layers with a transform (every layer except audio) |

Assets work the same way: check `asset.type` (`'SCENE'`, `'IMAGE'`, `'FONT'`, `'AUDIO'`) before reading asset-specific members.

Keep a list of the types your plugin supports and skip the rest:

```typescript
const SUPPORTED_TYPES: ReadonlySet<string> = new Set([
  'SHAPE_LAYER',
  'TEXT_LAYER',
  'RECTANGLE',
  'ELLIPSE',
]);

const supported = creator.selection.nodes.filter((node) => SUPPORTED_TYPES.has(node.type));
```

Decide what your plugin does with the nodes it skips:

- **Skip them and notify the user** when the plugin works on many nodes at once (e.g. batch-renaming), then tell the users that some layers could not be updated.
- **Notify the user upfront** when they selected only unsupported nodes. Show a message in your plugin UI, such as "Select a shape or text layer".

```typescript
if (supported.length === 0) {
  creator.ui.postMessage({ type: 'unsupported-selection' });
  return;
}
```

#### Checking for a transform

`creator.utils.isTransformableLayer(node)` returns `true` for layers that have `position`, `opacity` and the other transform members. It returns `false` for audio layers and for all shapes. It only exists in Creator builds that support audio layers, so a plugin that must also run on older builds should use its own type list instead.

`creator.utils.isLayer(node)` returns `true` for every layer, including audio layers. Use it to tell layers from shapes, not to check for a transform.
