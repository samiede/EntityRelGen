# Relationship Management Specification - Canvas-Based Rendering

## Overview
The Relationship Management system handles the creation, visualization, and management of connections between entities using high-performance Canvas rendering. Relationships are directional connections with labels that are drawn with mathematical precision using Canvas 2D graphics.

## Relationship Data Model

### Core Relationship Structure
```javascript
{
  source: String,         // Source entity name
  target: String,         // Target entity name  
  label: String          // Relationship description
}
```

### Relationship Properties

#### Source Entity
- **Type**: String (entity name reference)
- **Requirements**: Must reference existing entity
- **Validation**: Selected from dropdown of available entities
- **Purpose**: Starting point of relationship connection

#### Target Entity
- **Type**: String (entity name reference)
- **Requirements**: Must reference existing entity, cannot equal source
- **Validation**: Selected from dropdown of available entities
- **Purpose**: End point of relationship connection

#### Label
- **Type**: String
- **Requirements**: Non-empty descriptive text
- **Purpose**: Describes the nature of the relationship
- **Display**: Rendered on Canvas with rounded background

## Relationship Creation Process

### User Interface
Location: Relationship Creation Card in controls panel

**Input Fields:**
- `#source-entity`: Dropdown selection of available entities
- `#target-entity`: Dropdown selection of available entities
- `#relation-label`: Text input for relationship description
- Submit button: "Add Relationship"

### Creation Workflow
1. **Input Validation**
   ```javascript
   const source = sourceSelect.value;
   const target = targetSelect.value;
   const label = document.getElementById('relation-label').value.trim();
   if (!source || !target || !label || source === target) return;
   ```

2. **Validation Rules**
   - Source entity must be selected
   - Target entity must be selected
   - Label must be non-empty
   - Source and target must be different entities

3. **Relationship Creation**
   ```javascript
   relationships.push({ source, target, label });
   ```

4. **System Updates**
   - Add to relationships array
   - Update relationship display list
   - Trigger Canvas re-render

## Canvas Relationship Visualization

### Mathematical Edge-to-Edge Connection
Relationships are visualized using precise mathematical algorithms for edge-to-edge connections:

### Edge Intersection Algorithm
```javascript
function calculateEdgeIntersection(center, targetCenter, bounds) {
  const dx = targetCenter.x - center.x;
  const dy = targetCenter.y - center.y;
  
  // Handle edge cases
  if (dx === 0 && dy === 0) return center;
  
  if (dx === 0) {
    // Vertical line
    return {
      x: center.x,
      y: dy > 0 ? bounds.bottom : bounds.top
    };
  }
  
  if (dy === 0) {
    // Horizontal line
    return {
      x: dx > 0 ? bounds.right : bounds.left,
      y: center.y
    };
  }
  
  // Calculate intersection with rectangle edges
  const slope = dy / dx;
  let intersectionX, intersectionY;
  
  if (Math.abs(dx) > Math.abs(dy)) {
    // Intersect with left or right edge
    if (dx > 0) {
      intersectionX = bounds.right;
      intersectionY = center.y + slope * (bounds.right - center.x);
    } else {
      intersectionX = bounds.left;
      intersectionY = center.y + slope * (bounds.left - center.x);
    }
  } else {
    // Intersect with top or bottom edge
    if (dy > 0) {
      intersectionY = bounds.bottom;
      intersectionX = center.x + (bounds.bottom - center.y) / slope;
    } else {
      intersectionY = bounds.top;
      intersectionX = center.x + (bounds.top - center.y) / slope;
    }
  }
  
  return { x: intersectionX, y: intersectionY };
}
```

### Canvas Relationship Rendering
```javascript
function drawRelationship(rel) {
  const fromEntity = entities.find(e => e.name === rel.source);
  const toEntity = entities.find(e => e.name === rel.target);
  
  if (!fromEntity || !toEntity) return;
  
  // Calculate entity centers
  const fromCenter = {
    x: fromEntity.x + fromEntity.width / 2,
    y: fromEntity.y + fromEntity.height / 2
  };
  const toCenter = {
    x: toEntity.x + toEntity.width / 2,
    y: toEntity.y + toEntity.height / 2
  };
  
  // Calculate precise edge intersection points
  const startPoint = calculateEdgeIntersection(fromCenter, toCenter, {
    left: fromEntity.x,
    right: fromEntity.x + fromEntity.width,
    top: fromEntity.y,
    bottom: fromEntity.y + fromEntity.height
  });
  
  const endPoint = calculateEdgeIntersection(toCenter, fromCenter, {
    left: toEntity.x,
    right: toEntity.x + toEntity.width,
    top: toEntity.y,
    bottom: toEntity.y + toEntity.height
  });
  
  // Draw relationship line with gradient
  const gradient = ctx.createLinearGradient(
    startPoint.x, startPoint.y, 
    endPoint.x, endPoint.y
  );
  gradient.addColorStop(0, '#667eea');
  gradient.addColorStop(1, '#764ba2');
  
  ctx.strokeStyle = gradient;
  ctx.lineWidth = 2;
  ctx.lineCap = 'round';
  ctx.lineJoin = 'round';
  
  ctx.beginPath();
  ctx.moveTo(startPoint.x, startPoint.y);
  ctx.lineTo(endPoint.x, endPoint.y);
  ctx.stroke();
  
  // Draw arrow
  drawArrow(endPoint, startPoint);
  
  // Draw label
  drawRelationshipLabel(rel.label, startPoint, endPoint);
}
```

### Mathematical Arrow Rendering
```javascript
function drawArrow(to, from) {
  const angle = Math.atan2(to.y - from.y, to.x - from.x);
  const arrowLength = 12;
  const arrowAngle = Math.PI / 6;
  
  ctx.save();
  ctx.fillStyle = '#764ba2';
  ctx.beginPath();
  ctx.moveTo(to.x, to.y);
  ctx.lineTo(
    to.x - arrowLength * Math.cos(angle - arrowAngle),
    to.y - arrowLength * Math.sin(angle - arrowAngle)
  );
  ctx.lineTo(
    to.x - arrowLength * Math.cos(angle + arrowAngle),
    to.y - arrowLength * Math.sin(angle + arrowAngle)
  );
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}
```

### Canvas Label Rendering
```javascript
function drawRelationshipLabel(label, startPoint, endPoint) {
  const labelX = (startPoint.x + endPoint.x) / 2;
  const labelY = (startPoint.y + endPoint.y) / 2;
  
  // Measure text
  ctx.font = '13px -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif';
  ctx.fontWeight = '500';
  const textMetrics = ctx.measureText(label);
  const textWidth = textMetrics.width;
  const textHeight = 16;
  
  // Draw label background with rounded corners
  const padding = 8;
  const bgX = labelX - textWidth/2 - padding;
  const bgY = labelY - textHeight/2 - padding/2;
  const bgWidth = textWidth + padding * 2;
  const bgHeight = textHeight + padding;
  
  // Background with rounded corners
  ctx.fillStyle = 'rgba(255, 255, 255, 0.95)';
  ctx.beginPath();
  ctx.roundRect(bgX, bgY, bgWidth, bgHeight, 6);
  ctx.fill();
  
  ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)';
  ctx.lineWidth = 1;
  ctx.stroke();
  
  // Draw text
  ctx.fillStyle = '#1d1d1f';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(label, labelX, labelY);
}
```

### Visual Properties
- **Line Gradient**: Dynamic gradient from #667eea to #764ba2
- **Line Width**: 2px with rounded caps and joins
- **Arrow Style**: Filled triangular arrow pointing to target
- **Label Background**: Rounded white background with subtle border
- **Direction**: Unidirectional from source to target
- **Performance**: Hardware-accelerated Canvas rendering

## Relationship Management Interface

### Relationship List Display
Location: Within Relationship Creation Card

**Display Format:**
Each relationship appears as a summary item showing:
- Source entity name
- Direction indicator (→)
- Target entity name
- Relationship label
- Delete button (✕)

**CSS Structure:**
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

### Relationship Display Generation
```javascript
wrapper.textContent = `${rel.source} → ${rel.target}: ${rel.label}`;
```

## Relationship Deletion System

### Deletion Interface
- **Location**: Delete button (✕) in each relationship summary
- **Style**: Compact button aligned to the right
- **Confirmation**: None (immediate deletion)

### Deletion Workflow
1. **Array Cleanup**: Remove relationship from relationships array
2. **UI Updates**:
   - Refresh relationship list display
   - Trigger Canvas re-render
3. **Visual Cleanup**: Automatic Canvas redraw removes visual elements

### Canvas-Based Deletion Implementation
```javascript
deleteBtn.onclick = () => {
  relationships.splice(idx, 1);
  updateRelationshipList();
  requestRender(); // Canvas re-render
};
```

## Dynamic Updates and Synchronization

### Update Triggers
Relationship visualizations update automatically when:
- New relationship created
- Relationship deleted
- Entity positions change (drag operations)
- Entity names change
- Entities are deleted

### Canvas Render Function
The main render function handles complete re-rendering of all relationships:

```javascript
function render() {
  // Clear canvas
  ctx.clearRect(0, 0, canvasWidth, canvasHeight);
  
  // Draw layers in order
  drawGrid();
  drawRelationships();  // Behind entities
  drawEntities();       // On top
}

function drawRelationships() {
  relationships.forEach(rel => {
    drawRelationship(rel);
  });
}
```

## Performance Optimizations

### Canvas-Specific Performance
- **Batch Rendering**: All relationships drawn in single render cycle
- **Mathematical Precision**: Direct geometric calculations without DOM queries
- **Hardware Acceleration**: Canvas 2D context optimization
- **Efficient Updates**: Only renders when data changes

### Render Loop Optimization
```javascript
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

### Memory Efficiency
- **Pure Data Arrays**: Lightweight relationship objects
- **No DOM Elements**: Eliminates SVG element overhead
- **Mathematical Operations**: Direct coordinate calculations
- **Garbage Collection**: Minimal object creation during rendering

## Advanced Canvas Features

### Gradient Animation Capabilities
```javascript
// Optional: Animated gradients for relationships
function createAnimatedGradient(startPoint, endPoint, time) {
  const gradient = ctx.createLinearGradient(
    startPoint.x, startPoint.y, 
    endPoint.x, endPoint.y
  );
  
  const offset = (Math.sin(time * 0.002) + 1) * 0.5;
  gradient.addColorStop(offset, '#667eea');
  gradient.addColorStop(1 - offset, '#764ba2');
  
  return gradient;
}
```

### High-DPI Support
```javascript
// Automatic scaling for crisp visuals on retina displays
canvas.width = canvasWidth * devicePixelRatio;
canvas.height = canvasHeight * devicePixelRatio;
ctx.scale(devicePixelRatio, devicePixelRatio);
```

## Error Handling and Validation

### Relationship Validation
- **Entity Existence**: Checks if source and target entities exist
- **Self-Reference Prevention**: Blocks relationships from entity to itself
- **Duplicate Detection**: Optional prevention of duplicate relationships

### Graceful Degradation
```javascript
function drawRelationship(rel) {
  const fromEntity = entities.find(e => e.name === rel.source);
  const toEntity = entities.find(e => e.name === rel.target);
  
  // Graceful handling of missing entities
  if (!fromEntity || !toEntity) return;
  
  // Continue with rendering...
}
```

This Canvas-based relationship management system provides superior performance, mathematical precision, and beautiful visual rendering while maintaining clean separation between data models and presentation layers. 