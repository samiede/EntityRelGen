# Entity Management Specification - Canvas-Based Architecture

## Overview
The Entity Management system handles the complete lifecycle of entities within the Canvas-based Entity Relationship Builder. Entities are now pure JavaScript objects that are rendered on the HTML5 Canvas with mathematical precision and 60fps performance.

## Canvas Entity Data Model

### Core Entity Structure
```javascript
{
  name: String,            // Entity identifier
  attributes: Array,       // List of entity attributes
  color: String,          // Base color for rendering
  x: Number,              // Canvas X position
  y: Number,              // Canvas Y position
  width: Number,          // Entity width in pixels
  height: Number          // Entity height in pixels
}
```

### Entity Properties

#### Name
- **Type**: String
- **Requirements**: Non-empty, serves as identifier
- **Validation**: Must be present for entity creation
- **Usage**: Canvas text rendering, relationship references, dropdown options

#### Attributes
- **Type**: Array of strings
- **Input Format**: Multi-line textarea (one attribute per line)
- **Processing**: Split by newline, filtered for empty strings
- **Display**: Rendered on Canvas if entity height allows

#### Color
- **Type**: Hexadecimal color string (#RRGGBB)
- **Generation**: Semantic color based on entity characteristics
- **Purpose**: Canvas gradient generation and visual distinction
- **Algorithm**: Hash-based semantic color assignment

#### Canvas Position
- **x, y**: Canvas coordinates in pixels
- **Constraints**: Bounded within Canvas dimensions
- **Precision**: Floating-point for smooth positioning
- **Updates**: Real-time during drag operations

#### Dimensions
- **width, height**: Entity size in pixels
- **Default**: 120px × 80px for new entities
- **Constraints**: Minimum viable size for text rendering
- **Responsive**: Can be adjusted for attribute display

## Entity Creation Process

### User Interface
Location: Entity Creation Card in controls panel

**Input Fields:**
- `#entity-name`: Text input for entity name
- `#entity-attributes`: Textarea for attributes (multi-line)
- Submit button: "Add Entity"

### Canvas-Based Creation Workflow
1. **Input Validation**
   ```javascript
   const name = document.getElementById('entity-name').value.trim();
   if (!name) {
     // Show visual validation feedback
     return;
   }
   ```

2. **Attribute Processing**
   ```javascript
   const attributes = attributesText ? 
     attributesText.split('\n').filter(attr => attr.trim()) : [];
   ```

3. **Semantic Color Generation**
   ```javascript
   const color = getSemanticColor(name, attributes);
   ```

4. **Position Calculation**
   ```javascript
   const x = Math.max(50, Math.random() * (canvasWidth - 170));
   const y = Math.max(50, Math.random() * (canvasHeight - 130));
   ```

5. **Entity Object Creation**
   ```javascript
   const entity = {
     name: name,
     attributes: attributes,
     color: color,
     x: Math.max(0, Math.min(canvasWidth - 120, x)),
     y: Math.max(0, Math.min(canvasHeight - 80, y)),
     width: 120,
     height: 80
   };
   ```

6. **System Integration**
   - Add to entities array
   - Update dropdown options
   - Request Canvas render
   - Update entity management list

### Enhanced Entity Creation Function
```javascript
function createEntityAt(x, y, name = 'New Entity', attributes = []) {
  const color = getSemanticColor(name, attributes);
  
  const entity = {
    name: name,
    attributes: attributes,
    color: color,
    x: Math.max(0, Math.min(canvasWidth - 120, x)),
    y: Math.max(0, Math.min(canvasHeight - 80, y)),
    width: 120,
    height: 80
  };
  
  entities.push(entity);
  updateEntitySelectOptions();
  updateEntityList();
  requestRender();
  
  return entity;
}
```

## Canvas Rendering System

### Entity Drawing Algorithm
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

## Canvas Interaction System

### Hit Detection Algorithm
```javascript
function hitTest(pos) {
  // Test entities from top to bottom (reverse order for layering)
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

### Drag and Drop System
```javascript
function handleMouseDown(e) {
  mousePos = getMousePosition(e);
  const hitEntity = hitTest(mousePos);
  
  if (hitEntity) {
    isDragging = true;
    dragTarget = hitEntity;
    dragOffset = {
      x: mousePos.x - hitEntity.x,
      y: mousePos.y - hitEntity.y
    };
    canvas.style.cursor = 'grabbing';
  }
}

function handleMouseMove(e) {
  mousePos = getMousePosition(e);
  
  if (isDragging && dragTarget) {
    // Update position with bounds checking
    dragTarget.x = mousePos.x - dragOffset.x;
    dragTarget.y = mousePos.y - dragOffset.y;
    
    // Keep entities within Canvas bounds
    dragTarget.x = Math.max(0, Math.min(canvasWidth - dragTarget.width, dragTarget.x));
    dragTarget.y = Math.max(0, Math.min(canvasHeight - dragTarget.height, dragTarget.y));
    
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

## Entity Editing System

### Control Panel Interface
Each entity in the management list provides editing capabilities:

**Name Editing:**
- Input field pre-populated with current name
- Real-time updates to entity data and Canvas rendering
- Automatic dropdown option synchronization

**Attribute Editing:**
- Textarea with current attributes (one per line)
- Updates entity data array
- Triggers Canvas re-render for attribute display

### Edit Workflow
```javascript
function updateEntityList() {
  entityList.innerHTML = '';
  entities.forEach((entity, idx) => {
    const wrapper = document.createElement('div');
    wrapper.className = 'entity-summary';
    wrapper.style.backgroundColor = entity.color;

    const nameInput = document.createElement('input');
    nameInput.value = entity.name;
    nameInput.onchange = () => {
      entity.name = nameInput.value;
      updateEntitySelectOptions();
      requestRender(); // Canvas re-render
    };
    wrapper.appendChild(nameInput);

    const textarea = document.createElement('textarea');
    textarea.value = entity.attributes.join('\n');
    textarea.onchange = () => {
      entity.attributes = textarea.value.trim().split('\n').filter(attr => attr.trim());
      requestRender(); // Canvas re-render
    };
    wrapper.appendChild(textarea);

    // Delete button implementation
    const deleteBtn = document.createElement('button');
    deleteBtn.textContent = 'Delete';
    deleteBtn.onclick = () => {
      entities.splice(idx, 1);
      updateEntityList();
      updateEntitySelectOptions();
      requestRender();
    };
    wrapper.appendChild(deleteBtn);

    entityList.appendChild(wrapper);
  });
}
```

## Entity Resizing

The Canvas-based system provides intuitive entity resizing through visual resize handles:

#### Resize Handle System
- **Visual Indicator**: Blue resize handle appears in the bottom-right corner when hovering over an entity
- **Handle Dimensions**: 12x12 pixel interactive area (6px from corner in each direction)
- **Visual Style**: Blue (#4A90E2) square with white border for clear visibility
- **Cursor Feedback**: Cursor changes to "se-resize" when hovering over resize handle

#### Resize Interaction
- **Activation**: Click and drag on the resize handle to begin resizing
- **Resize Direction**: Southeast corner handle supports diagonal resizing (width and height simultaneously)
- **Real-time Feedback**: Entity dimensions update in real-time during drag operation
- **Smooth Performance**: Resize operations trigger render requests for 60fps updates

#### Size Constraints
- **Minimum Dimensions**: 
  - Minimum width: 80 pixels
  - Minimum height: 60 pixels
- **Canvas Boundaries**: Entities cannot be resized beyond canvas boundaries
- **Automatic Clamping**: Resize operations respect both minimum size and canvas bounds

#### Technical Implementation
- **State Management**: Uses `isResizing`, `resizeTarget`, and `resizeHandle` state variables
- **Mouse Event Handling**: Integrated with existing mouse event system
- **Collision Detection**: `getResizeHandle()` function determines if mouse is over resize handle
- **Coordinate System**: Uses Canvas coordinate system for precise positioning

#### User Experience
- **Discoverability**: Resize handles only appear on hover to maintain clean interface
- **Visual Feedback**: Entity shadows and highlighting indicate interactive state
- **Cursor Changes**: Contextual cursor changes guide user interaction
- **Responsive Design**: Works seamlessly with existing drag-and-drop functionality

## Entity Lifecycle Management

### Creation Lifecycle
1. **User Input** → Form validation
2. **Data Processing** → Pure object creation
3. **Canvas Integration** → Add to entities array
4. **Visual Rendering** → Canvas draw operation
5. **UI Synchronization** → Update control panels

### Update Lifecycle
1. **Property Change** → Direct object mutation
2. **Render Request** → Batched Canvas update
3. **UI Refresh** → Control panel synchronization

### Deletion Lifecycle
1. **User Action** → Delete button click
2. **Data Cleanup** → Remove from entities array
3. **Relationship Cleanup** → Orphaned relationships handled
4. **Visual Update** → Canvas re-render
5. **UI Update** → Control panel refresh

This Canvas-based entity management system provides superior performance, mathematical precision, and smooth 60fps interactions while maintaining a clean separation between data and presentation layers. 