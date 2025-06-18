# Visualization System Specification

## Overview
The Visualization System manages the graphical representation and interactive elements within the Entity Relationship Builder's canvas. It handles entity positioning, relationship rendering, drag-and-drop interactions, and real-time visual updates.

## Canvas Architecture

### Canvas Container Structure
```html
<div id="canvas">
  <svg id="lines"></svg>
  <!-- Dynamic entity elements -->
</div>
```

### Canvas Properties
- **Dimensions**: 2/3 viewport width × full viewport height
- **Positioning**: Relative container for absolute child elements
- **Overflow**: Hidden to contain draggable elements
- **Border**: 2px solid #ccc for visual definition
- **Background**: Default white background

### Layout Positioning System
- **Entity Positioning**: Absolute positioning within canvas bounds
- **Coordinate System**: Top-left origin (0,0)
- **Default Entity Position**: (50px, 50px) from top-left
- **Position Units**: Pixels for precise control

## Entity Visualization

### Visual Representation
Entities are rendered as interactive DOM elements with the following properties:

**CSS Styling:**
```css
.entity {
  position: absolute;
  padding: 10px;
  border-radius: 10px;
  cursor: move;
  color: #000;
  resize: both;
  overflow: auto;
  min-width: 50px;
  min-height: 30px;
}
```

### Entity Visual Properties

#### Shape and Appearance
- **Shape**: Rounded rectangles (10px border-radius)
- **Padding**: 10px internal spacing
- **Text Color**: Black (#000) for high contrast
- **Background**: Randomly generated colors
- **Default Size**: 100px width × 60px height

#### Interactive Features
- **Cursor**: Move cursor indicating draggable state
- **Resize Handles**: Native browser resize controls
- **Size Constraints**: Minimum 50px width, 30px height
- **Overflow**: Auto scrolling for content overflow

### Color Generation System
```javascript
function getRandomColor() {
  const letters = '0123456789ABCDEF';
  let color = '#';
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
}
```

**Color Properties:**
- **Format**: 6-digit hexadecimal (#RRGGBB)
- **Distribution**: Uniform random across color spectrum
- **Uniqueness**: No collision prevention (potential duplicates)
- **Application**: Background color for visual entity distinction

## SVG Relationship Rendering

### SVG Layer Architecture
The SVG system provides a vector graphics overlay for relationship visualization:

**SVG Container Setup:**
```css
svg {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  pointer-events: none;
  z-index: 0;
}
```

### SVG Properties
- **Positioning**: Absolute overlay covering entire canvas
- **Dimensions**: 100% width and height of canvas
- **Pointer Events**: Disabled to allow entity interaction
- **Z-Index**: Behind entities (0) for proper layering

### Arrow Marker Definition
```svg
<defs>
  <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" 
          markerWidth="6" markerHeight="6" orient="auto">
    <polygon points="0 0, 10 5, 0 10" fill="#555" />
  </marker>
</defs>
```

**Arrow Properties:**
- **Shape**: Triangular polygon
- **Color**: Medium gray (#555)
- **Size**: 6×6 pixel marker
- **Orientation**: Auto-aligned with line direction
- **Position**: End of relationship lines

### Line Rendering Algorithm

#### Position Calculation
For each relationship, the system calculates connection points:

1. **Entity Center Calculation**
   ```javascript
   const x1 = from.offsetLeft + from.offsetWidth / 2;
   const y1 = from.offsetTop + from.offsetHeight / 2;
   const x2 = to.offsetLeft + to.offsetWidth / 2;
   const y2 = to.offsetTop + to.offsetHeight / 2;
   ```

2. **Line Element Creation**
   ```javascript
   const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
   line.setAttribute('x1', x1);
   line.setAttribute('y1', y1);
   line.setAttribute('x2', x2);
   line.setAttribute('y2', y2);
   line.setAttribute('stroke', '#555');
   line.setAttribute('stroke-width', '2');
   line.setAttribute('marker-end', 'url(#arrow)');
   ```

#### Label Positioning
Relationship labels are positioned at the midpoint of connection lines:

```javascript
const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
text.setAttribute('x', (x1 + x2) / 2);
text.setAttribute('y', (y1 + y2) / 2 - 5);
text.textContent = rel.label;
```

**Label Properties:**
- **Position**: Centered between entities
- **Vertical Offset**: 5px above line for readability
- **Text Content**: Relationship label string
- **Styling**: Default SVG text rendering

## Interactive Drag System

### Drag Implementation Architecture
The drag system enables intuitive entity positioning through mouse interactions:

```javascript
function makeDraggable(el) {
  let isDragging = false;
  let offsetX = 0, offsetY = 0;
  
  // Event listener implementation
}
```

### Drag Interaction Flow

#### 1. Mouse Down Event
```javascript
el.addEventListener('mousedown', (e) => {
  if (e.target !== el) return; // Prevent child interference
  isDragging = true;
  offsetX = e.offsetX;
  offsetY = e.offsetY;
  document.body.style.userSelect = 'none'; // Prevent text selection
});
```

**Mouse Down Actions:**
- **Target Validation**: Ensure click on entity element
- **State Initialization**: Set dragging flag to true
- **Offset Capture**: Record initial click position within element
- **Text Selection Prevention**: Disable user selection during drag

#### 2. Mouse Move Event
```javascript
document.addEventListener('mousemove', (e) => {
  if (isDragging) {
    el.style.left = `${e.pageX - canvas.offsetLeft - offsetX}px`;
    el.style.top = `${e.pageY - canvas.offsetTop - offsetY}px`;
    updateLines();
  }
});
```

**Mouse Move Actions:**
- **Position Update**: Calculate new element position
- **Canvas Coordinate Translation**: Convert page coordinates to canvas coordinates
- **Offset Compensation**: Maintain consistent grab point
- **Visual Update**: Trigger relationship line re-rendering

#### 3. Mouse Up Event
```javascript
document.addEventListener('mouseup', () => {
  isDragging = false;
  document.body.style.userSelect = ''; // Restore text selection
});
```

**Mouse Up Actions:**
- **State Reset**: Clear dragging flag
- **Text Selection Restoration**: Re-enable user selection
- **Event Cleanup**: Implicit through event listener scope

### Drag Interaction Features

#### Coordinate System
- **Canvas Relative**: Positions calculated relative to canvas container
- **Offset Preservation**: Maintains grab point during drag
- **Boundary Management**: No explicit boundary constraints (can drag outside)

#### Visual Feedback
- **Cursor Indication**: Move cursor shows draggable state
- **Real-time Updates**: Relationship lines update during drag
- **Smooth Movement**: Direct position updates for responsive feel

## Real-time Update System

### Update Triggers
Visual updates occur automatically on:
- **Entity Creation**: New entity appears on canvas
- **Entity Deletion**: Visual element removed from canvas
- **Entity Movement**: Relationship lines recalculated
- **Relationship Changes**: SVG elements added/removed
- **Entity Editing**: Name changes update display

### Update Functions

#### `updateLines()` - Primary Update Function
Complete re-rendering of all relationship visualizations:

1. **SVG Reset**: Clear existing lines and regenerate definitions
2. **Relationship Processing**: Iterate through relationships array
3. **Entity Resolution**: Find current DOM elements
4. **Position Calculation**: Get current entity center points
5. **Line Generation**: Create SVG line and text elements
6. **DOM Integration**: Append elements to SVG container

#### Performance Characteristics
- **Update Strategy**: Complete regeneration vs. incremental updates
- **Frequency**: Triggered by user interactions and data changes
- **Efficiency**: Linear time complexity with relationship count

## Visual Hierarchy and Layering

### Z-Index Management
```
Layer Stack (bottom to top):
1. SVG Lines (z-index: 0)
2. Entities (default positioning)
3. Control panels (separate container)
```

### Element Stacking
- **Background**: Canvas container with border
- **SVG Layer**: Relationship lines and arrows
- **Entity Layer**: Interactive draggable elements
- **Controls**: Separate panel outside canvas

## Responsive Visual Behavior

### Entity Resize Functionality
- **CSS Property**: `resize: both` enables native resize handles
- **Constraints**: Minimum size enforcement
- **Behavior**: Independent width and height adjustment
- **Side Effects**: Automatic relationship line repositioning

### Canvas Overflow Management
- **Container**: `overflow: hidden` contains draggable elements
- **Scrolling**: No scrolling mechanisms within canvas
- **Boundaries**: Entities can be positioned outside visible area

## Visual Performance Considerations

### Rendering Optimization
- **SVG Efficiency**: Vector graphics scale well for moderate complexity
- **DOM Manipulation**: Minimal direct DOM access patterns
- **Event Handling**: Efficient event delegation for interactions
- **Memory Management**: Proper cleanup on element removal

### Scalability Factors
- **Entity Count**: Visual performance decreases with high entity counts
- **Relationship Density**: Complex relationship networks impact rendering
- **Update Frequency**: Frequent updates during drag operations
- **Browser Performance**: Dependent on browser SVG implementation

## Accessibility Considerations

### Visual Accessibility
- **Color Contrast**: Black text on colored backgrounds
- **Visual Indicators**: Clear cursor changes for interactivity
- **Size Flexibility**: User-controlled entity sizing
- **Clear Boundaries**: Defined canvas borders

### Interaction Accessibility
- **Mouse Dependencies**: Primary interaction through mouse/pointer
- **Visual Feedback**: Clear indication of interactive elements
- **Drag Affordances**: Move cursor indicates draggable state

## Future Enhancement Opportunities

### Advanced Visualization
- **Grid Snapping**: Align entities to grid positions
- **Zoom and Pan**: Canvas navigation for large diagrams
- **Visual Themes**: Color schemes and styling options
- **Animation**: Smooth transitions for interactions

### Enhanced Interactions
- **Multi-select**: Select and move multiple entities
- **Connection Points**: Specific attachment points for relationships
- **Visual Effects**: Hover states and selection highlighting
- **Touch Support**: Mobile and tablet interaction patterns 