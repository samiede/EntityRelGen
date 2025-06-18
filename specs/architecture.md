# Application Architecture Specification

## Overview
The Entity Relationship Builder follows a client-side architecture pattern with separation of concerns between presentation, logic, and data management layers.

## Technical Architecture

### Core Technologies
- **HTML5**: Semantic markup and structure
- **CSS3**: Styling, layout, and visual presentation
- **Vanilla JavaScript**: Business logic, event handling, and DOM manipulation
- **SVG**: Vector graphics for relationship visualization

### Application Structure

#### HTML Structure
```html
<body>
  <div id="canvas">           <!-- Visualization area -->
    <svg id="lines"></svg>    <!-- Relationship rendering -->
  </div>
  <div id="controls">         <!-- Control panels -->
    <!-- Control cards for different functionalities -->
  </div>
</body>
```

#### CSS Architecture
- **Layout System**: Flexbox-based split-panel design
- **Component Styling**: Card-based control panels with consistent theming
- **Responsive Elements**: Flexible entity sizing with resize capabilities
- **Visual Hierarchy**: Layered z-index management for canvas elements

#### JavaScript Architecture

##### Data Models
```javascript
const entities = [];        // Array of entity objects
const relationships = [];   // Array of relationship objects
```

**Entity Object Structure:**
```javascript
{
  el: DOMElement,          // DOM reference
  name: String,            // Entity identifier
  attributes: Array,       // List of attributes
  color: String           // Visual identifier
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

##### Core Functions

**Entity Management:**
- `addCustomEntity()` - Creates new entities
- `updateEntityList()` - Synchronizes entity display
- `updateEntitySelectOptions()` - Updates dropdown options

**Relationship Management:**
- `createRelationship()` - Establishes entity connections
- `updateRelationshipList()` - Manages relationship display
- `updateLines()` - Renders visual connections

**Interaction Handling:**
- `makeDraggable()` - Implements drag-and-drop functionality
- Event listeners for mouse interactions

**Utility Functions:**
- `getRandomColor()` - Generates entity colors
- SVG rendering functions for visual elements

## Design Patterns

### Observer Pattern
- Real-time synchronization between data models and UI
- Automatic updates when entities or relationships change

### Factory Pattern
- Dynamic DOM element creation for entities and relationships
- Consistent object instantiation patterns

### Event-Driven Architecture
- Mouse event handling for drag-and-drop interactions
- Form event handling for user inputs

## Data Flow

1. **User Input** → Form controls capture user data
2. **Data Processing** → JavaScript functions validate and process input
3. **Model Updates** → Arrays are updated with new/modified data
4. **View Synchronization** → UI elements are updated to reflect changes
5. **Visual Rendering** → Canvas and SVG elements display current state

## Performance Considerations

### Optimization Strategies
- **DOM Manipulation**: Minimal direct DOM access with cached references
- **Event Delegation**: Efficient event handling for dynamic elements
- **Rendering Optimization**: SVG updates only when necessary
- **Memory Management**: Proper cleanup when removing entities

### Scalability Factors
- In-memory storage suitable for moderate entity counts
- SVG rendering performance for complex relationship networks
- Event listener management for interactive elements

## Security Considerations
- **XSS Prevention**: Text content insertion without HTML interpretation
- **Input Validation**: Basic validation for entity names and attributes
- **Client-Side Only**: No server communication reduces attack surface

## Browser Compatibility
- Modern browser support for HTML5, CSS3, and ES6+ features
- SVG support for relationship visualization
- Standard DOM APIs for maximum compatibility 