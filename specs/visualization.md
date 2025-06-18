# Visualization System Specification - Canvas-Based Rendering

## Design Philosophy

### "Thought Made Visible" through Canvas
The Entity Relationship Builder leverages HTML5 Canvas for unprecedented performance and visual fidelity. Every pixel is mathematically precise, every interaction renders at 60fps, creating a seamless environment where complex diagrams feel effortless and responsive.

### Core Design Principles
- **Mathematical Precision**: Pixel-perfect positioning with geometric algorithms
- **Performance Excellence**: 60fps rendering with hardware acceleration
- **Visual Elegance**: Beautiful gradients, shadows, and animations
- **Responsive Interactions**: Instant visual feedback on all operations
- **Scalable Architecture**: Hundreds of entities without performance loss

## Canvas Architecture - High-Performance Rendering

### Optimized Visual Hierarchy
The Canvas rendering system uses a carefully designed layer system for optimal visibility:
- **Entities**: Clean rectangles with top-left positioned names (no attribute clutter)
- **Relationships**: Rendered above entities to ensure visibility even when entities overlap
- **Grid**: Subtle background guidelines for alignment
- **Interaction**: Real-time hover and drag feedback

### Canvas Environment Setup
```javascript
function initCanvas() {
  const rect = canvasContainer.getBoundingClientRect();
  canvasWidth = rect.width;
  canvasHeight = rect.height;
  
  // High-DPI support for crisp visuals
  canvas.width = canvasWidth * devicePixelRatio;
  canvas.height = canvasHeight * devicePixelRatio;
  canvas.style.width = canvasWidth + 'px';
  canvas.style.height = canvasHeight + 'px';
  
  // Scale context for high-DPI displays
  ctx.scale(devicePixelRatio, devicePixelRatio);
}
```

### Render Loop Management
```javascript
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

function render() {
  // Clear canvas
  ctx.clearRect(0, 0, canvasWidth, canvasHeight);
  
  // Draw layers in order for optimal visibility
  drawGrid();
  drawEntities();       // Draw entities first
  drawRelationships();  // Draw relationships on top for visibility
}
```

### Canvas State Management
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

## Entity Visualization - Mathematical Beauty

### Canvas Entity Data Model
```javascript
const entity = {
  name: String,           // Display name
  attributes: Array,      // Entity attributes
  color: String,         // Base color
  x: Number,             // Canvas X position
  y: Number,             // Canvas Y position  
  width: Number,         // Entity width
  height: Number         // Entity height
};
```

### Entity Rendering Algorithm
```javascript
function drawEntity(entity) {
  const isHovered = entity === hoveredEntity;
  const isDragged = entity === dragTarget;
  
  // Create gradient for depth
  const gradient = ctx.createLinearGradient(
    entity.x, entity.y, 
    entity.x + entity.width, entity.y + entity.height
  );
  gradient.addColorStop(0, entity.color);
  gradient.addColorStop(1, ColorPalette.lighten(entity.color, 15));
  
  // Dynamic shadows for interaction feedback
  if (isHovered || isDragged) {
    ctx.save();
    ctx.shadowColor = 'rgba(0, 0, 0, 0.3)';
    ctx.shadowBlur = isDragged ? 20 : 15;
    ctx.shadowOffsetX = isDragged ? 0 : 2;
    ctx.shadowOffsetY = isDragged ? 0 : 4;
  }
  
  // Draw entity background
  ctx.fillStyle = gradient;
  ctx.fillRect(entity.x, entity.y, entity.width, entity.height);
  
  if (isHovered || isDragged) {
    ctx.restore();
  }
  
  // Draw border with hover effects
  ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)';
  ctx.lineWidth = isHovered ? 2 : 1;
  ctx.strokeRect(entity.x, entity.y, entity.width, entity.height);
  
  // Render clean entity name
  drawEntityText(entity);
}
```

### Clean Text Rendering
```javascript
function drawEntityText(entity) {
  // Set text properties for top-left positioning
  ctx.font = '14px -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif';
  ctx.fontWeight = '500';
  ctx.textAlign = 'left';
  ctx.textBaseline = 'top';
  ctx.fillStyle = '#1d1d1f';
  
  // Draw text at top-left corner with padding (no background)
  ctx.fillText(entity.name, entity.x + 8, entity.y + 8);
}
```

### Intelligent Color System
```javascript
const ColorPalette = {
  primary: [
    '#667eea', '#764ba2', '#f093fb', '#f5576c',
    '#4facfe', '#00f2fe', '#43e97b', '#38f9d7',
    '#ffecd2', '#fcb69f', '#a8edea', '#fed6e3'
  ],
  
  generateSemanticColor(entityName, attributes) {
    const hash = this.generateHash(entityName + attributes.join(''));
    return this.primary[hash % this.primary.length];
  },
  
  lighten(color, percent) {
    // Mathematical color manipulation
    const num = parseInt(color.replace("#", ""), 16);
    const amt = Math.round(2.55 * percent);
    const R = (num >> 16) + amt;
    const G = (num >> 8 & 0x00FF) + amt;
    const B = (num & 0x0000FF) + amt;
    return "#" + (0x1000000 + 
      (R < 255 ? R < 1 ? 0 : R : 255) * 0x10000 +
      (G < 255 ? G < 1 ? 0 : G : 255) * 0x100 +
      (B < 255 ? B < 1 ? 0 : B : 255)).toString(16).slice(1);
  }
};
```

## Relationship Visualization - Mathematical Precision

### Edge-to-Edge Connection Algorithm
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
  
  // Calculate centers and boundaries
  const fromCenter = {
    x: fromEntity.x + fromEntity.width / 2,
    y: fromEntity.y + fromEntity.height / 2
  };
  const toCenter = {
    x: toEntity.x + toEntity.width / 2,
    y: toEntity.y + toEntity.height / 2
  };
  
  // Get precise edge intersection points
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

## Interaction System - Precise Hit Detection

### Canvas Event Handling
```javascript
function setupCanvasEvents() {
  canvas.addEventListener('mousedown', handleMouseDown);
  canvas.addEventListener('mousemove', handleMouseMove);
  canvas.addEventListener('mouseup', handleMouseUp);
  canvas.addEventListener('mouseleave', handleMouseLeave);
  canvas.addEventListener('dblclick', handleDoubleClick);
}

function getMousePosition(e) {
  const rect = canvas.getBoundingClientRect();
  return {
    x: e.clientX - rect.left,
    y: e.clientY - rect.top
  };
}
```

### Efficient Hit Testing
```javascript
function hitTest(pos) {
  // Test entities from top to bottom (reverse order)
  for (let i = entities.length - 1; i >= 0; i--) {
    const entity = entities[i];
    if (pos.x >= entity.x && pos.x <= entity.x + entity.width &&
        pos.y >= entity.y && pos.y <= entity.y + entity.height) {
      return entity;
    }
  }
  return null;
}
```

### Smooth Drag Operations
```javascript
function handleMouseMove(e) {
  mousePos = getMousePosition(e);
  
  if (isDragging && dragTarget) {
    // Update position with constraints
    dragTarget.x = mousePos.x - dragOffset.x;
    dragTarget.y = mousePos.y - dragOffset.y;
    
    // Keep within bounds
    dragTarget.x = Math.max(0, Math.min(canvasWidth - dragTarget.width, dragTarget.x));
    dragTarget.y = Math.max(0, Math.min(canvasHeight - dragTarget.height, dragTarget.y));
    
    // Request render for smooth animation
    requestRender();
  } else {
    // Handle hover effects
    const hitEntity = hitTest(mousePos);
    if (hitEntity !== hoveredEntity) {
      hoveredEntity = hitEntity;
      canvas.style.cursor = hitEntity ? 'grab' : 'default';
      requestRender();
    }
  }
}
```

## Visual Effects - Canvas-Specific Enhancements

### Dynamic Grid System
```javascript
function drawGrid() {
  const gridSize = 20;
  ctx.strokeStyle = 'rgba(255, 255, 255, 0.1)';
  ctx.lineWidth = 0.5;
  
  ctx.beginPath();
  for (let x = 0; x <= canvasWidth; x += gridSize) {
    ctx.moveTo(x, 0);
    ctx.lineTo(x, canvasHeight);
  }
  for (let y = 0; y <= canvasHeight; y += gridSize) {
    ctx.moveTo(0, y);
    ctx.lineTo(canvasWidth, y);
  }
  ctx.stroke();
}
```

### Label Rendering with Rounded Backgrounds
```javascript
function drawRelationshipLabel(label, startPoint, endPoint) {
  const labelX = (startPoint.x + endPoint.x) / 2;
  const labelY = (startPoint.y + endPoint.y) / 2;
  
  // Measure text
  ctx.font = '13px -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif';
  const textWidth = ctx.measureText(label).width;
  const padding = 8;
  
  // Draw rounded background
  ctx.fillStyle = 'rgba(255, 255, 255, 0.95)';
  ctx.beginPath();
  ctx.roundRect(
    labelX - textWidth/2 - padding, 
    labelY - 8, 
    textWidth + padding * 2, 
    16, 
    6
  );
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

## Performance Optimization

### Canvas-Specific Optimizations
- **High-DPI Scaling**: Automatic device pixel ratio handling
- **Efficient Clearing**: Full canvas clear on each frame
- **Batched Rendering**: Single render per frame via requestAnimationFrame
- **Mathematical Precision**: No DOM layout calculations
- **Memory Efficiency**: Single Canvas element vs. hundreds of DOM nodes

### Render Performance
- **60fps Target**: Consistent smooth animations
- **Hardware Acceleration**: Canvas 2D context optimization
- **Minimal State Changes**: Efficient Canvas API usage
- **Optimized Drawing**: Minimal path operations and state saves

This revolutionary visualization system transforms the act of creating entity-relationship diagrams from a mechanical task into an inspiring creative experience, where technology becomes invisible and ideas flow naturally from thought to visual reality. 