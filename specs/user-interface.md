# User Interface Specification - Canvas-Based Architecture

## Overview
The Entity Relationship Builder features a split-panel interface optimized for Canvas-based visualization and control operations. The interface prioritizes performance with hardware-accelerated Canvas rendering and glassmorphic control panels.

## Layout Architecture

### Primary Layout Structure
```
┌─────────────────────────────────────────────────────────┐
│                    Main Container                        │
│  (display: flex)                                        │
│                                                         │
│  ┌─────────────────────┐  ┌─────────────────────────┐   │
│  │                     │  │                         │   │
│  │  Canvas Container   │  │    Controls Panel       │   │
│  │    (flex: 2)        │  │    (flex: 1)            │   │
│  │                     │  │                         │   │
│  │  [Canvas Element]   │  │ [Glassmorphic Cards]    │   │
│  │                     │  │                         │   │
│  └─────────────────────┘  └─────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Canvas Container (`#canvas-container`)
- **Purpose**: High-performance Canvas wrapper for all visual content
- **Dimensions**: Takes 2/3 of screen width, full viewport height minus margins
- **Styling**: Glassmorphic background with rounded corners
- **Contents**: Single HTML5 Canvas element for all rendering
- **Overflow**: Hidden to contain Canvas interactions

**Canvas Container CSS:**
```css
#canvas-container {
  flex: 2;
  background: rgba(255, 255, 255, 0.08);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 20px;
  position: relative;
  margin: 20px 10px 20px 20px;
  height: calc(100vh - 40px);
  overflow: hidden;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
}
```

### Main Canvas Element (`#main-canvas`)
- **Purpose**: All visual rendering (entities, relationships, grid)
- **Dimensions**: 100% of container with high-DPI scaling
- **Rendering**: Hardware-accelerated 2D graphics context
- **Interactions**: Direct mouse event handling for precision
- **Performance**: 60fps render loop with optimized updates

**Canvas Element CSS:**
```css
#main-canvas {
  width: 100%;
  height: 100%;
  display: block;
  border-radius: 20px;
  cursor: default;
}
```

### Controls Panel (`#controls`)
- **Purpose**: Glassmorphic control interface for all application functions
- **Dimensions**: Takes 1/3 of screen width, full viewport height minus margins
- **Styling**: Performance-optimized glassmorphic design
- **Scrolling**: Vertical scroll for overflow content (overflow-y: auto)
- **Performance**: Reduced backdrop filters for optimal rendering

**Controls Panel CSS:**
```css
#controls {
  flex: 1;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  padding: 20px;
  margin: 20px 20px 20px 10px;
  height: calc(100vh - 40px);
  overflow-y: auto;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
}
```

## Canvas Rendering System

### Entity Visualization
Entities are rendered directly on Canvas with mathematical precision:

**Canvas Entity Rendering:**
- **Shape**: Clean rounded rectangles with gradient fills
- **Colors**: Semantic color generation with gradient effects
- **Text**: Clean typography positioned at top-left (no background)
- **Content**: Entity name only (attributes managed via control panel)
- **Interactions**: Hover effects with dynamic shadows
- **Performance**: 60fps smooth animations and transitions

### Relationship Visualization
Relationships use mathematical algorithms for perfect connections:

**Canvas Relationship Rendering:**
- **Lines**: Edge-to-edge connections with gradient strokes
- **Arrows**: Mathematically precise directional indicators
- **Labels**: Rounded background labels with optimal positioning
- **Layering**: Rendered above entities for maximum visibility
- **Performance**: Hardware-accelerated drawing operations

### Visual Effects
- **Dynamic Grid**: Subtle alignment guides drawn on Canvas
- **Hover States**: Real-time shadow and scaling effects
- **Drag Feedback**: Smooth position updates with constraints
- **High-DPI**: Automatic scaling for crisp visuals on all displays

## Glassmorphic Control Card System

### Performance-Optimized Card Structure
Each functional area uses optimized glassmorphic styling:

```css
.control-card {
  border-radius: 16px;
  padding: 20px;
  margin-bottom: 16px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px) saturate(120%);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 
    0 4px 16px rgba(0, 0, 0, 0.08),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
}
```

### Card Types

#### 1. Entity Creation Card
- **Title**: "Create Entity"
- **Functionality**: Canvas entity creation with semantic colors
- **Components**:
  - Text input for entity name (with validation feedback)
  - Textarea for attributes (multi-line with filtering)
  - Submit button with hover animations

#### 2. Relationship Creation Card
- **Title**: "Create Relationship"
- **Functionality**: Mathematical relationship connections
- **Components**:
  - Source entity dropdown (auto-populated)
  - Target entity dropdown (auto-populated)
  - Relationship label input (with validation)
  - Submit button with success feedback
  - Dynamic relationship list with delete buttons

#### 3. Entity Management Card
- **Title**: "Entities"
- **Functionality**: Canvas entity data management
- **Components**:
  - Dynamic list of entity summaries
  - Real-time editing interfaces
  - Color-coded entity cards

#### 4. Scene Graph Export Card
- **Title**: "Scene Graph"
- **Functionality**: Canvas data serialization and export
- **Components**:
  - Export button (JSON generation)
  - Copy to clipboard button (with fallback support)
  - Formatted output preview area

## Advanced Form Element Styling

### Glassmorphic Form Design
```css
input, textarea, select, button {
  margin-top: 8px;
  display: block;
  width: 100%;
  padding: 12px 16px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  color: rgba(255, 255, 255, 0.9);
  font-size: 14px;
  transition: all 0.2s ease;
}
```

### Interactive States
- **Focus States**: Color-changing borders with glow effects
- **Hover States**: Subtle background lightening
- **Validation**: Visual feedback for errors and success
- **Animations**: Smooth transitions for all state changes

### Button Styling
```css
button {
  background: linear-gradient(145deg, 
    rgba(0, 122, 255, 0.8) 0%,
    rgba(0, 122, 255, 0.6) 100%);
  color: white;
  font-weight: 600;
  cursor: pointer;
  border: 1px solid rgba(0, 122, 255, 0.3);
  transition: all 0.2s ease;
}

button:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0, 122, 255, 0.3);
  background: linear-gradient(145deg, 
    rgba(0, 122, 255, 0.9) 0%,
    rgba(0, 122, 255, 0.7) 100%);
}
```

## Visual Feedback Elements

### Canvas Interaction Feedback
- **Hover Cursors**: Context-aware cursor changes
- **Drag States**: Visual feedback during entity movement
- **Selection**: Clear visual indication of selected elements
- **Boundaries**: Constraint visualization during interactions

### Entity Summary Cards
```css
.entity-summary {
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  padding: 12px;
  margin-bottom: 12px;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
}
```

### Relationship Summary Items
```css
.relationship-summary {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  padding: 10px 16px;
  margin-top: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 13px;
}
```

## Color System and Theming

### Intelligent Color Generation
- **Semantic Colors**: Hash-based color assignment
- **Gradient Systems**: Mathematical color harmonies
- **Contrast Optimization**: Readable text on all backgrounds
- **Color Consistency**: Matching between Canvas and UI elements

### Theme Properties
- **Background**: Dynamic gradient (135deg, #667eea to #764ba2)
- **Glass Effects**: Subtle transparency with backdrop filters
- **Shadows**: Layered shadow systems for depth
- **Typography**: Professional font stack with optimal weights

## Performance Optimizations

### Canvas-Specific Optimizations
- **High-DPI Support**: Automatic device pixel ratio scaling
- **Render Batching**: Single render per frame via requestAnimationFrame
- **Interaction Efficiency**: Direct Canvas event handling
- **Memory Management**: Optimized object creation and cleanup

### UI Performance
- **Reduced Backdrop Filters**: Optimized for performance while maintaining aesthetics
- **Transition Targeting**: Specific property transitions vs. 'all'
- **Shadow Optimization**: Single shadows instead of complex multi-layer effects
- **Animation Constraints**: Performance-conscious animation choices

## Responsive Design Considerations

### Adaptive Canvas Sizing
- **Dynamic Scaling**: Canvas adapts to viewport changes
- **High-DPI Handling**: Automatic scaling for retina displays
- **Boundary Management**: Entity constraints within Canvas bounds
- **Zoom Considerations**: Potential for future zoom functionality

### Control Panel Adaptability
- **Flexible Layout**: Flexbox-based proportional sizing
- **Scrollable Content**: Overflow handling for varying content
- **Form Responsiveness**: Input elements adapt to container width

## Accessibility Features

### Canvas Accessibility
- **Keyboard Navigation**: Potential for future keyboard controls
- **Screen Reader**: Alternative text descriptions for Canvas content
- **High Contrast**: Support for high contrast preferences
- **Reduced Motion**: Respect for motion sensitivity preferences

### Control Panel Accessibility
- **Focus Management**: Clear focus indicators
- **Color Contrast**: High contrast text on glassmorphic backgrounds
- **Label Association**: Proper form label relationships
- **Error Feedback**: Clear validation messages

### Accessibility CSS
```css
@media (prefers-contrast: high) {
  .entity {
    border: 2px solid #000;
    background: #fff;
    color: #000;
  }
}

@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Advanced Interaction Patterns

### Canvas Interactions
- **Precise Dragging**: Pixel-perfect entity positioning
- **Double-Click Creation**: Quick entity creation at cursor position
- **Hover Detection**: Real-time entity highlighting
- **Boundary Constraints**: Smooth constraint handling during drag

### Form Interactions
- **Real-Time Validation**: Immediate feedback on input errors
- **Success Animations**: Visual confirmation of successful actions
- **Loading States**: Clear indication during processing
- **Auto-Population**: Dynamic dropdown updates

#### Entity Resize Interactions
The Canvas system provides intuitive entity resizing with comprehensive visual feedback:

**Resize Handle Discovery**
- Resize handles appear as blue squares when hovering over entities
- Positioned precisely at the bottom-right corner of each entity
- 12x12 pixel interactive area with white border for visibility
- Only visible on hover to maintain clean interface design

**Cursor Feedback System**
- **Default State**: Standard arrow cursor over empty canvas
- **Entity Hover**: "grab" cursor when hovering over entity body
- **Resize Handle Hover**: "se-resize" cursor when over resize handle
- **Active Dragging**: "grabbing" cursor during drag operations
- **Active Resizing**: "se-resize" cursor maintained during resize operations

**Resize Interaction Flow**
1. User hovers over entity → resize handle appears
2. User hovers over resize handle → cursor changes to "se-resize"
3. User clicks and drags → entity resizes in real-time
4. User releases mouse → resize operation completes
5. Cursor updates based on current hover state

**Visual Feedback During Resizing**
- Real-time entity dimension updates at 60fps
- Entity maintains hover highlighting during resize
- Resize handle remains visible and interactive
- Smooth transitions between interaction states

**Accessibility Considerations**
- High contrast resize handles (#4A90E2 blue with white border)
- Clear cursor feedback for different interaction modes
- Visual boundaries prevent accidental resizing beyond canvas
- Minimum size constraints ensure entities remain usable

### Keyboard Interactions
// ... existing code ...

## Responsive Design
// ... existing code ...

This Canvas-based user interface provides a revolutionary combination of high-performance graphics rendering with beautiful, functional controls, creating an optimal environment for entity-relationship diagram creation and management. 