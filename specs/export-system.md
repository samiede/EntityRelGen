# Export System Specification

## Overview
The Export System provides functionality to serialize and export the current scene graph state, allowing users to save, share, or integrate their entity-relationship diagrams with external systems. The system offers both manual export and clipboard integration capabilities.

## Export Interface

### User Interface Location
The export functionality is located in the "Scene Graph" control card within the controls panel.

**Interface Components:**
- **Export Button**: Triggers scene graph generation and display
- **Copy to Clipboard Button**: Copies scene graph to system clipboard
- **Output Preview**: `<pre id="scene-output">` element for display

### Button Layout
```html
<div class="control-card">
  <strong>Scene Graph</strong>
  <button onclick="exportSceneGraph()">Export</button>
  <button onclick="copySceneGraph()">Copy to Clipboard</button>
  <pre id="scene-output"></pre>
</div>
```

## Export Functions

### Primary Export Function: `exportSceneGraph()`

**Current Implementation Status:**
- **Function Declaration**: Referenced in HTML onclick handler
- **Implementation**: Not present in current codebase
- **Expected Behavior**: Generate and display scene graph representation

**Intended Functionality:**
1. **Data Collection**: Gather current entities and relationships
2. **Serialization**: Convert data to exportable format
3. **Display**: Show formatted output in preview area

### Clipboard Function: `copySceneGraph()`

**Current Implementation Status:**
- **Function Declaration**: Referenced in HTML onclick handler
- **Implementation**: Not present in current codebase
- **Expected Behavior**: Copy scene graph to system clipboard

**Intended Functionality:**
1. **Scene Generation**: Generate scene graph data
2. **Clipboard Integration**: Use Clipboard API or fallback methods
3. **User Feedback**: Indicate successful copy operation

## Data Serialization Architecture

### Scene Graph Data Model
The export system should serialize the complete application state:

**Core Data Structure:**
```javascript
const sceneGraph = {
  entities: [
    {
      name: String,
      attributes: Array,
      position: { x: Number, y: Number },
      size: { width: Number, height: Number },
      color: String
    }
  ],
  relationships: [
    {
      source: String,
      target: String,
      label: String
    }
  ],
  metadata: {
    version: String,
    created: Date,
    entityCount: Number,
    relationshipCount: Number
  }
}
```

### Entity Serialization
Each entity should capture:

**Positional Data:**
```javascript
{
  position: {
    x: entity.el.offsetLeft,
    y: entity.el.offsetTop
  },
  size: {
    width: entity.el.offsetWidth,
    height: entity.el.offsetHeight
  }
}
```

**Visual Properties:**
```javascript
{
  color: entity.color,
  name: entity.name,
  attributes: entity.attributes
}
```

### Relationship Serialization
Relationships maintain their current simple structure:
```javascript
{
  source: relationship.source,
  target: relationship.target,
  label: relationship.label
}
```

## Export Format Options

### JSON Format (Recommended)
**Advantages:**
- Structured data representation
- Easy parsing for re-import
- Standard format for data exchange
- Readable and editable

**Example Output:**
```json
{
  "entities": [
    {
      "name": "User",
      "attributes": ["id", "name", "email"],
      "position": { "x": 50, "y": 50 },
      "size": { "width": 100, "height": 60 },
      "color": "#ff5733"
    }
  ],
  "relationships": [
    {
      "source": "User",
      "target": "Order",
      "label": "places"
    }
  ],
  "metadata": {
    "version": "1.0",
    "created": "2024-01-01T00:00:00Z",
    "entityCount": 2,
    "relationshipCount": 1
  }
}
```

### Alternative Formats

#### Simplified Text Format
**Use Case**: Quick sharing and documentation
**Example:**
```
ENTITIES:
- User [id, name, email]
- Order [id, amount, date]

RELATIONSHIPS:
- User → Order (places)
- Order → Product (contains)
```

#### Graph Description Language
**Use Case**: Integration with graph analysis tools
**Example:**
```
digraph SceneGraph {
  User [label="User\nid, name, email"];
  Order [label="Order\nid, amount, date"];
  User -> Order [label="places"];
}
```

## Implementation Specifications

### Export Function Implementation
```javascript
function exportSceneGraph() {
  const sceneData = {
    entities: entities.map(entity => ({
      name: entity.name,
      attributes: entity.attributes,
      position: {
        x: entity.el.offsetLeft,
        y: entity.el.offsetTop
      },
      size: {
        width: entity.el.offsetWidth,
        height: entity.el.offsetHeight
      },
      color: entity.color
    })),
    relationships: relationships.map(rel => ({
      source: rel.source,
      target: rel.target,
      label: rel.label
    })),
    metadata: {
      version: "1.0",
      created: new Date().toISOString(),
      entityCount: entities.length,
      relationshipCount: relationships.length
    }
  };
  
  const output = JSON.stringify(sceneData, null, 2);
  document.getElementById('scene-output').textContent = output;
}
```

### Clipboard Function Implementation
```javascript
function copySceneGraph() {
  exportSceneGraph(); // Generate current scene data
  const output = document.getElementById('scene-output').textContent;
  
  if (navigator.clipboard && window.isSecureContext) {
    // Modern Clipboard API
    navigator.clipboard.writeText(output).then(() => {
      // Provide user feedback
      showCopyFeedback("Scene graph copied to clipboard!");
    }).catch(err => {
      console.error('Failed to copy: ', err);
      fallbackCopyToClipboard(output);
    });
  } else {
    // Fallback for older browsers
    fallbackCopyToClipboard(output);
  }
}

function fallbackCopyToClipboard(text) {
  const textArea = document.createElement('textarea');
  textArea.value = text;
  document.body.appendChild(textArea);
  textArea.focus();
  textArea.select();
  try {
    document.execCommand('copy');
    showCopyFeedback("Scene graph copied to clipboard!");
  } catch (err) {
    console.error('Fallback copy failed: ', err);
    showCopyFeedback("Copy failed. Please manually select and copy the text.");
  }
  document.body.removeChild(textArea);
}
```

## User Feedback System

### Copy Feedback Implementation
```javascript
function showCopyFeedback(message) {
  // Create temporary feedback element
  const feedback = document.createElement('div');
  feedback.textContent = message;
  feedback.style.cssText = `
    position: fixed;
    top: 20px;
    right: 20px;
    background: #4CAF50;
    color: white;
    padding: 10px 20px;
    border-radius: 5px;
    z-index: 1000;
    transition: opacity 0.3s;
  `;
  
  document.body.appendChild(feedback);
  
  // Auto-remove after 3 seconds
  setTimeout(() => {
    feedback.style.opacity = '0';
    setTimeout(() => {
      document.body.removeChild(feedback);
    }, 300);
  }, 3000);
}
```

## Export Validation

### Data Integrity Checks
Before export, the system should validate:

**Entity Validation:**
- All entities have valid names
- Position and size data are numeric
- Color values are valid hex codes
- Attributes are properly formatted arrays

**Relationship Validation:**
- Source and target entities exist
- Labels are non-empty strings
- No circular references in critical relationships

**System Validation:**
- Current data state is consistent
- No orphaned relationships
- All DOM elements are properly positioned

### Error Handling
```javascript
function validateSceneData(sceneData) {
  const errors = [];
  
  // Validate entities
  sceneData.entities.forEach((entity, index) => {
    if (!entity.name || typeof entity.name !== 'string') {
      errors.push(`Entity ${index}: Invalid name`);
    }
    if (!Array.isArray(entity.attributes)) {
      errors.push(`Entity ${index}: Invalid attributes`);
    }
    if (typeof entity.position?.x !== 'number' || typeof entity.position?.y !== 'number') {
      errors.push(`Entity ${index}: Invalid position`);
    }
  });
  
  // Validate relationships
  sceneData.relationships.forEach((rel, index) => {
    const sourceExists = sceneData.entities.some(e => e.name === rel.source);
    const targetExists = sceneData.entities.some(e => e.name === rel.target);
    
    if (!sourceExists) {
      errors.push(`Relationship ${index}: Source entity '${rel.source}' not found`);
    }
    if (!targetExists) {
      errors.push(`Relationship ${index}: Target entity '${rel.target}' not found`);
    }
  });
  
  return errors;
}
```

## Browser Compatibility

### Clipboard API Support
- **Modern Browsers**: Chrome 66+, Firefox 63+, Safari 13.1+
- **Secure Context**: HTTPS required for Clipboard API
- **Fallback**: document.execCommand for older browsers

### JSON Support
- **Universal Support**: All modern browsers support JSON
- **Stringify Options**: Pretty printing with indentation
- **Error Handling**: Handle circular references if present

## Security Considerations

### Data Privacy
- **Client-Side Only**: No server transmission of sensitive data
- **Local Processing**: All serialization occurs in browser
- **User Control**: Explicit user action required for export

### Clipboard Security
- **Permission-Based**: Modern browsers require user permission
- **Secure Context**: HTTPS requirement for Clipboard API
- **Fallback Safety**: Manual copy selection as backup

## Performance Considerations

### Export Efficiency
- **Data Size**: JSON serialization scales with entity/relationship count
- **DOM Access**: Position/size queries require DOM measurement
- **Memory Usage**: Temporary objects created during serialization

### Optimization Strategies
- **Lazy Evaluation**: Generate export data only when requested
- **Caching**: Cache positional data if frequent exports expected
- **Chunked Processing**: For large diagrams, consider chunked serialization

## Future Enhancement Opportunities

### Import Functionality
- **Reverse Process**: Parse and recreate scene from exported data
- **File Upload**: Support for .json file import
- **Validation**: Imported data validation and error handling

### Export Formats
- **Image Export**: Canvas-to-image conversion for visual sharing
- **PDF Generation**: Document format for reports
- **Various Text Formats**: CSV, XML, YAML alternatives

### Advanced Features
- **Selective Export**: Choose specific entities/relationships
- **Template Creation**: Save reusable diagram templates
- **Version Control**: Track changes and export history
- **Compression**: Optimize export size for large diagrams 