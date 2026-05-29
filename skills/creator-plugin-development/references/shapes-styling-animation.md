# Shapes, Styling, and Animation

## Creating Shapes

First create a **shape layer**, then add shapes to it:

```typescript
const layer = creator.activeScene.createShapeLayer();
const rect = layer.createRectangle({ size: { width: 200, height: 150 } });
```

Shapes need a fill or stroke to be visible.

### Shape Creation Methods

All available on `ShapeLayer` and `Group`:

```typescript
// Rectangle
layer.createRectangle({ position?: Vector, size?: Size, roundness?: number });

// Ellipse
layer.createEllipse({ position?: Vector, size?: Size });

// Polygon
layer.createPolygon({
  position?: Vector, rotation?: number,
  points?: number, outerRadius?: number, outerRoundness?: number
});

// Star
layer.createStar({
  position?: Vector, rotation?: number,
  points?: number, innerRadius?: number, outerRadius?: number,
  innerRoundness?: number, outerRoundness?: number
});

// Path (custom bezier)
layer.createPath({
  points?: PathPoint[], closed?: boolean
});

// Group (combine shapes)
layer.createGroup({ shapes: [rect, ellipse] });
```

### PathPoint Format

```typescript
const point: PathPoint = {
  vertex: { x: 0, y: 0 },    // Point position
  inTan: { x: 0, y: 0 },     // Incoming tangent
  outTan: { x: 0, y: 0 },    // Outgoing tangent
};
```

## Styling

### Fills

Fills and strokes are created on `ShapeLayer` or `Group` and apply to all shapes within.

```typescript
// Solid fill
layer.createFill({
  type: 'SOLID',
  color: { r: 66, g: 133, b: 244 }
});

// Linear gradient
layer.createFill({
  type: 'GRADIENT_LINEAR',
  start: { x: 0, y: 100 },      // Optional (defaults to left edge)
  end: { x: 200, y: 100 },      // Optional (defaults to right edge)
  stops: [
    { color: { r: 255, g: 0, b: 0 }, offset: 0, opacity: 1 },
    { color: { r: 0, g: 0, b: 255 }, offset: 1, opacity: 1 }
  ]
});

// Radial gradient
layer.createFill({
  type: 'GRADIENT_RADIAL',
  start: { x: 100, y: 100 },    // Optional (defaults to center)
  end: { x: 200, y: 100 },      // Optional (defaults to right edge)
  highlightAngle: 45,            // Optional
  highlightLength: 50,           // Optional (-100 to 100)
  stops: [
    { color: { r: 255, g: 255, b: 255 }, offset: 0, opacity: 1 },
    { color: { r: 0, g: 0, b: 0 }, offset: 1, opacity: 1 }
  ]
});
```

### Strokes

```typescript
// Solid stroke
layer.createStroke({
  fill: { type: 'SOLID', color: { r: 0, g: 0, b: 0 } },
  width: 2
});

// Gradient stroke
layer.createStroke({
  fill: {
    type: 'GRADIENT_LINEAR',
    stops: [
      { color: { r: 255, g: 0, b: 0 }, offset: 0, opacity: 1 },
      { color: { r: 0, g: 0, b: 255 }, offset: 1, opacity: 1 }
    ]
  },
  width: 5
});
```

### Modifying Existing Styles

```typescript
// Change fill color
layer.fills[0].color.staticValue = { r: 255, g: 0, b: 0 };

// Remove a fill/stroke
layer.fills[0].remove();
layer.strokes[0].remove();
```

### Trim Paths

Trim paths control how much of a stroke path is drawn:

```typescript
const trimPath = layer.createTrimPath({
  start: 0,     // Percentage (0-100)
  end: 100,     // Percentage (0-100)
  offset: 0     // Degrees (0-360)
});

// Animate trim path for draw-on effect
trimPath.end.addKeyframes([
  { frame: 0, value: 0 },
  { frame: 60, value: 100 }
]);
```

## Animation

### Animatable Properties

Any property implementing `Animatable<T>` can be animated:

```typescript
interface Animatable<T> {
  staticValue: T;                                   // Value when not animated
  readonly isAnimated: boolean;                     // Has keyframes?
  readonly keyframes: ReadonlyArray<Keyframe<T>>;   // All keyframes
  getKeyframeAt(frame: number): Keyframe<T> | undefined;
  addKeyframes(keyframes: Array<KeyframeAdd<T>>): void;
  clearKeyframes(): void;
}
```

### Transform Properties (Common to Layers and Groups)

| Property | Type | Notes |
|---|---|---|
| `position` | `Animatable<Vector>` | X/Y position |
| `rotation` | `Animatable<number>` | Degrees |
| `scale` | `Animatable<Vector>` | **Percentage** (100 = 100%) |
| `opacity` | `Animatable<number>` | **0 to 100** |
| `skew` | `Animatable<number>` | Degrees |
| `skewAxis` | `Animatable<number>` | Degrees |

### Shape-Specific Animatable Properties

| Shape | Properties |
|---|---|
| Rectangle | `size: Animatable<Size>`, `roundness: Animatable<number>` |
| Ellipse | `size: Animatable<Size>` |
| Polygon | `points`, `outerRadius`, `outerRoundness` |
| Star | `points`, `innerRadius`, `outerRadius`, `innerRoundness`, `outerRoundness` |
| Path | `pathData: Animatable<PathData>` |

### Style Animatable Properties

| Style | Properties |
|---|---|
| Solid fill | `color: Animatable<Color>` |
| Gradient fill | `start`, `end: Animatable<Vector>`, `stops: Animatable<ColorStops>` |
| Radial gradient | Above + `highlightAngle`, `highlightLength: Animatable<number>` |
| Stroke | `width: Animatable<number>` |
| Trim path | `start`, `end: Animatable<number>`, `offset: Animatable<number>` |

### Adding Keyframes

```typescript
// Simple animation
layer.position.addKeyframes([
  { frame: 0, value: { x: 100, y: 100 } },
  { frame: 60, value: { x: 400, y: 100 } }
]);

// With easing
layer.position.addKeyframes([
  { frame: 0, value: { x: 50, y: 100 }, easing: { type: 'CUBIC_BEZIER', x1: 0.42, y1: 0, x2: 0.58, y2: 1 } },
  { frame: 60, value: { x: 350, y: 100 } }
]);
```

### Easing Types

```typescript
// Linear (constant speed)
{ type: 'LINEAR' }

// Cubic bezier (customizable curve)
{ type: 'CUBIC_BEZIER', x1: number, y1: number, x2: number, y2: number }
```

Common cubic bezier presets:
- **Ease in**: `x1: 0.42, y1: 0, x2: 1, y2: 1`
- **Ease out**: `x1: 0, y1: 0, x2: 0.58, y2: 1`
- **Ease in-out**: `x1: 0.42, y1: 0, x2: 0.58, y2: 1`

### Updating Existing Keyframes

```typescript
// Check if animated
const isAnimated = layer.position.isAnimated;

// Read keyframes
const keyframes = layer.position.keyframes;

// Update a keyframe value
const kf = layer.position.getKeyframeAt(30);
if (kf) {
  kf.value = { x: kf.value.x + 20, y: kf.value.y + 20 };
  kf.easing = { type: 'CUBIC_BEZIER', x1: 0.42, y1: 0, x2: 0.58, y2: 1 };
}

// Remove a specific keyframe
kf.remove();

// Remove all keyframes
layer.position.clearKeyframes();
```

### Grouping and Animating Together

```typescript
const group = layer.createGroup({ shapes: [rect, ellipse] });

const endFrame = scene.duration * scene.framerate;

// Rotate the entire group
group.rotation.addKeyframes([
  { frame: 0, value: 0 },
  { frame: endFrame, value: 360 }
]);
```
