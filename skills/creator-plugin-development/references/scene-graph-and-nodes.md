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

All layers share common properties from `LayerMixin`:

```typescript
// Common layer properties
layer.id;              // readonly string
layer.name;            // string (read/write)
layer.type;            // layer type constant
layer.visible;         // boolean
layer.locked;          // boolean
layer.focused;         // boolean
layer.startFrame;      // number
layer.endFrame;        // number
layer.timelineOffset;  // number
layer.blendMode;       // BlendMode

// Common animatable properties (TransformMixin)
layer.position;        // Animatable<Vector>
layer.rotation;        // Animatable<number>
layer.scale;           // Animatable<Vector>
layer.opacity;         // Animatable<number> (0-100)
layer.skew;            // Animatable<number>
layer.skewAxis;        // Animatable<number>

// Common methods
layer.clone();         // Duplicate the layer
layer.remove();        // Remove from scene
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

// TextLayer — has text content
if (layer.type === 'TEXT_LAYER') {
  layer.text;        // string (read/write)
}

// SceneLayer — references another scene
if (layer.type === 'SCENE_LAYER') {
  layer.scene;       // Scene
  layer.break();     // Break connection to source scene
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

### Layer Type Guard

```typescript
function isLayer(node: Layer | Shape): node is Layer {
  return (
    node.type === 'SHAPE_LAYER' ||
    node.type === 'SCENE_LAYER' ||
    node.type === 'IMAGE_LAYER' ||
    node.type === 'TEXT_LAYER'
  );
}
```
