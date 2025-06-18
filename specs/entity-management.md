# Entity Management Specification

## Overview
The Entity Management system handles the complete lifecycle of entities within the Entity Relationship Builder, including creation, modification, visualization, and deletion. Entities represent the primary objects in the scene graph with associated attributes and visual properties.

## Entity Data Model

### Core Entity Structure
```javascript
{
  el: DOMElement,          // Reference to DOM element
  name: String,            // Unique entity identifier
  attributes: Array,       // List of entity attributes
  color: String           // Visual background color
}
```

### Entity Properties

#### Name
- **Type**: String
- **Requirements**: Non-empty, serves as identifier
- **Validation**: Must be present for entity creation
- **Usage**: Display text, relationship references, dropdown options

#### Attributes
- **Type**: Array of strings
- **Input Format**: Multi-line textarea (one attribute per line)
- **Processing**: Split by newline characters, trimmed
- **Storage**: JSON serialized in DOM dataset
- **Display**: Joined by newlines in edit interface

#### Color
- **Type**: Hexadecimal color string (#RRGGBB)
- **Generation**: Random color assignment at creation
- **Purpose**: Visual distinction between entities
- **Persistence**: Stored in DOM dataset

#### DOM Element
- **Type**: HTML div element
- **Classes**: `.entity`
- **Positioning**: Absolute within canvas
- **Interactivity**: Draggable, resizable

## Entity Creation Process

### User Interface
Location: Entity Creation Card in controls panel

**Input Fields:**
- `#entity-name`: Text input for entity name
- `#entity-attributes`: Textarea for attributes (multi-line)
- Submit button: "Add Entity"

### Creation Workflow
1. **Input Validation**
   ```javascript
   const name = document.getElementById('entity-name').value.trim();
   if (!name) return; // Prevent empty names
   ```

2. **Attribute Processing**
   ```javascript
   const attributes = document.getElementById('entity-attributes').value.trim().split('\n');
   ```

3. **Visual Property Generation**
   ```javascript
   const color = getRandomColor(); // Generate unique color
   ```

4. **DOM Element Creation**
   ```javascript
   const div = document.createElement('div');
   div.classList.add('entity');
   div.textContent = name;
   div.style.backgroundColor = color;
   ```

5. **Default Positioning and Sizing**
   - **Position**: (50px, 50px) from top-left
   - **Size**: 100px width × 60px height
   - **Behavior**: User-resizable and draggable

6. **Data Storage**
   ```javascript
   div.dataset.name = name;
   div.dataset.color = color;
   div.dataset.attributes = JSON.stringify(attributes);
   ```

7. **System Integration**
   - Add to entities array
   - Append to canvas
   - Enable drag functionality
   - Update dropdown options
   - Refresh entity list display

## Entity Editing System

### Inline Editing Interface
Each entity in the management list provides inline editing capabilities:

**Name Editing:**
- Input field pre-populated with current name
- Real-time updates to entity display and references
- Automatic dropdown option updates

**Attribute Editing:**
- Textarea with current attributes (one per line)
- Updates stored in DOM dataset
- No visual change to canvas element

### Edit Workflow
1. **Name Change Handling**
   ```javascript
   nameInput.onchange = () => {
     entity.name = nameInput.value;
     entity.el.textContent = entity.name;
     updateEntitySelectOptions();
     updateLines();
   };
   ```

2. **Attribute Updates**
   ```javascript
   textarea.onchange = () => {
     entity.attributes = textarea.value.trim().split('\n');
     entity.el.dataset.attributes = JSON.stringify(entity.attributes);
   };
   ```

## Entity Display and Visualization

### Canvas Representation
- **Visual Element**: Rounded rectangle with entity name
- **Background**: Randomly assigned color
- **Positioning**: User-controlled via drag-and-drop
- **Sizing**: User-controlled via CSS resize handles

### Management List Representation
Each entity appears as a summary card containing:
- Name input field for editing
- Attributes textarea for editing
- Delete button for removal
- Background color matching canvas entity

## Entity Interaction Features

### Drag and Drop Functionality
**Implementation:** `makeDraggable(el)` function

**Interaction Flow:**
1. **Mouse Down**: Capture offset and initiate drag mode
2. **Mouse Move**: Update position relative to canvas
3. **Mouse Up**: End drag mode and clean up
4. **Side Effects**: Trigger relationship line updates

**Technical Details:**
```javascript
el.addEventListener('mousedown', (e) => {
  if (e.target !== el) return; // Prevent child element interference
  isDragging = true;
  offsetX = e.offsetX;
  offsetY = e.offsetY;
  document.body.style.userSelect = 'none'; // Prevent text selection
});
```

### Resize Functionality
- **CSS Property**: `resize: both`
- **Constraints**: Minimum 50px width, 30px height
- **Behavior**: Native browser resize handles
- **Updates**: Automatic relationship line repositioning

## Entity Deletion System

### Deletion Interface
- **Location**: Delete button in each entity summary card
- **Label**: "Delete"
- **Confirmation**: None (immediate deletion)

### Deletion Workflow
1. **DOM Cleanup**: Remove entity element from canvas
2. **Data Cleanup**: Remove from entities array
3. **UI Updates**: 
   - Refresh entity management list
   - Update dropdown options
   - Refresh relationship lines
4. **Relationship Cleanup**: Orphaned relationships remain but cannot render

### Deletion Impact
- **Relationships**: Existing relationships with deleted entities become inactive
- **References**: Entity removed from all dropdown selections
- **Visual**: Canvas cleared of entity representation

## Entity Synchronization System

### Multi-View Consistency
The system maintains synchronization between:
- Canvas visual representation
- Management list interface
- Dropdown option lists
- Relationship system references

### Update Triggers
**Automatic updates occur when:**
- New entity created
- Entity name changed
- Entity deleted
- System initialization

### Synchronization Functions
- `updateEntityList()`: Refreshes management interface
- `updateEntitySelectOptions()`: Updates dropdown menus
- `updateLines()`: Redraws relationship connections

## Data Persistence

### Current Implementation
- **Storage**: In-memory JavaScript arrays
- **Session**: Data lost on page refresh
- **DOM Dataset**: Backup storage in element attributes

### DOM Dataset Storage
```javascript
element.dataset.name = entityName;
element.dataset.color = entityColor;
element.dataset.attributes = JSON.stringify(attributeArray);
```

## Validation and Error Handling

### Input Validation
- **Name Required**: Empty names prevent entity creation
- **Duplicate Names**: Currently allowed (potential conflict source)
- **Attribute Format**: Flexible text input, split by newlines

### Error Prevention
- **Null Checks**: Validation before processing
- **Default Values**: Fallback for missing data
- **Safe References**: Existence checks before DOM manipulation

## Performance Considerations

### Optimization Strategies
- **Lazy Updates**: UI refreshes only when necessary
- **Cached References**: DOM elements stored for efficient access
- **Minimal Redraws**: Targeted updates rather than full refreshes

### Scalability Factors
- **Memory Usage**: Linear growth with entity count
- **DOM Performance**: Manageable for moderate entity numbers
- **Event Handling**: One drag listener per entity 