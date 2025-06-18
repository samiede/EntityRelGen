# Application Architecture Specification - Canvas-Based Rendering

## Overview
The Entity Relationship Builder follows a high-performance client-side architecture with Canvas-based rendering for optimal performance and smooth 60fps interactions. The system uses a single HTML5 Canvas element for all visual rendering, eliminating DOM overhead and enabling hardware-accelerated graphics.

## Technical Architecture

### Core Technologies
- **HTML5**: Semantic markup and Canvas element
- **CSS3**: Control panel styling and glassmorphic design
- **Vanilla JavaScript**: Business logic, Canvas rendering, and event handling
- **HTML5 Canvas**: High-performance 2D graphics rendering
- **RequestAnimationFrame**: Smooth 60fps render loop

### Application Structure

#### HTML Structure
```html
<body>
  <div id="canvas-container">     <!-- Canvas wrapper -->
    <canvas id="main-canvas"></canvas>  <!-- High-performance rendering -->
  </div>
  <div id="controls">             <!-- Control panels -->
    <!-- Control cards for different functionalities -->
  </div>
</body>
```

#### CSS Architecture
- **Layout System**: Flexbox-based split-panel design
- **Canvas Container**: Full-screen Canvas with glassmorphic styling
- **Control Styling**: Performance-optimized glassmorphic panels
- **Responsive Design**: Dynamic Canvas sizing with high-DPI support

#### JavaScript Architecture

##### Canvas State Management
```javascript
// Canvas state
let canvasWidth = 0;
let canvasHeight = 0;
let devicePixelRatio = window.devicePixelRatio || 1;

// Interaction state
let isDragging = false;
let dragTarget = null;
let dragOffset = { x: 0, y: 0 };
let mousePos = { x: 0, y: 0 };
let hoveredEntity = null;
```

##### Data Models
```javascript
const entities = [];        // Array of Canvas entity objects
const relationships = [];   // Array of relationship objects
```

**Canvas Entity Object Structure:**
```javascript
{
  name: String,            // Entity identifier
  attributes: Array,       // List of attributes
  color: String,          // Visual identifier
  x: Number,              // Canvas X position
  y: Number,              // Canvas Y position
  width: Number,          // Entity width
  height: Number          // Entity height
}
```

**Relationship Object Structure:**
```javascript
{
  source: String,         // Source entity name
  target: String,         // Target entity name
  label: String          // Relationship description
}
```

##### Core Rendering Functions

**Canvas Management:**
- `initCanvas()` - Initialize Canvas with high-DPI support
- `render()` - Main rendering function (60fps)
- `requestRender()` - Optimized render request batching

**Drawing Functions:**
- `drawEntities()` - Render clean entities with top-left names (no attributes)
- `drawRelationships()` - Render connections above entities for visibility
- `drawGrid()` - Subtle alignment grid system

**Interaction Handling:**
- `setupCanvasEvents()` - Mouse event listeners
- `hitTest()` - Efficient entity collision detection
- `handleMouseDown/Move/Up()` - Drag and drop system

**Entity Management:**
- `createEntityAt()` - Canvas-based entity creation
- `addCustomEntity()` - Form-based entity creation
- `updateEntityList()` - Synchronize control panel

## Design Patterns

### Canvas Rendering Pattern
- **Single Canvas Element**: All visual content rendered on one Canvas
- **Render Loop**: RequestAnimationFrame-based smooth animations
- **Hit Testing**: Mathematical collision detection for interactions
- **Dirty Rect Optimization**: Only render when changes occur

### Factory Pattern
- **Entity Creation**: Consistent Canvas entity instantiation
- **Render Functions**: Modular drawing component system

### Event-Driven Architecture
- **Canvas Events**: Direct Canvas mouse interaction handling
- **Render Events**: Optimized render request batching
- **Data Events**: Model-view synchronization

## Performance Architecture

### Canvas Optimization
```javascript
// High-DPI Canvas setup
canvas.width = canvasWidth * devicePixelRatio;
canvas.height = canvasHeight * devicePixelRatio;
ctx.scale(devicePixelRatio, devicePixelRatio);

// Optimized render loop
let renderRequested = false;
function requestRender() {
  if (!renderRequested) {
    renderRequested = true;
    requestAnimationFrame(() => {
      render();
      renderRequested = false;
    });
  }
}
```

### Mathematical Precision
- **Edge Intersection**: Geometric algorithms for perfect arrow positioning
- **Hit Detection**: Efficient rectangular collision detection
- **Smooth Curves**: Mathematical curve generation for relationships

## Data Flow

1. **User Input** → Canvas events or form controls
2. **Event Processing** → Canvas coordinate calculation and validation
3. **Model Updates** → Pure JavaScript data arrays
4. **Render Request** → Batched rendering via requestAnimationFrame
5. **Canvas Drawing** → High-performance 2D graphics rendering

## Performance Benefits

### Rendering Performance
- **60fps Animations**: Hardware-accelerated Canvas rendering
- **Single Element**: No DOM overhead for visual elements
- **Batched Updates**: Efficient render request batching
- **Mathematical Precision**: Perfect positioning and connections

### Memory Efficiency
- **Low Memory Footprint**: Canvas vs. hundreds of DOM elements
- **Garbage Collection**: Minimal object creation during rendering
- **Event Optimization**: Single Canvas event handling

### Scalability Factors
- **Entity Capacity**: Hundreds of entities at 60fps
- **Relationship Complexity**: Efficient mathematical connection rendering
- **Interactive Performance**: Instant responsiveness for all operations

## Browser Compatibility
- **Modern Canvas Support**: HTML5 Canvas 2D context
- **High-DPI Support**: Device pixel ratio scaling
- **RequestAnimationFrame**: Smooth animation support
- **Modern JavaScript**: ES6+ features for optimal performance

## Security Considerations
- **Canvas Isolation**: No direct DOM manipulation for visual elements
- **Input Validation**: Form-based data validation
- **Client-Side Only**: No server communication reduces attack surface 