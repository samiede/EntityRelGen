# Export System Specification - Canvas-Based Data Model

## Overview
The Export System provides comprehensive functionality to serialize and export the current Canvas-based scene graph state. The system offers JSON export with clipboard integration, supporting the Canvas entity data model with position, size, and visual properties.

## Export Interface

### User Interface Location
The export functionality is located in the "Scene Graph" control card within the controls panel.

**Interface Components:**
- **Export Button**: Triggers scene graph generation and display
- **Copy to Clipboard Button**: Copies scene graph to system clipboard with fallback support
- **Output Preview**: `<pre id="scene-output">` element for formatted display

### Button Layout
```html
<div class="control-card">
  <strong>Scene Graph</strong>
  <button onclick="exportSceneGraph()">Export</button>
  <button onclick="copySceneGraph()">Copy to Clipboard</button>
  <pre id="scene-output"></pre>
</div>
```

## Implemented Export Functions

### Primary Export Function: `exportSceneGraph()`

**Implementation Status:** ✅ **Fully Implemented**

**Functionality:**
1. **Canvas Data Collection**: Gathers current entities and relationships from Canvas data model
2. **JSON Serialization**: Converts Canvas data to structured JSON format
3. **Metadata Generation**: Includes version, timestamps, and statistics
4. **Display Output**: Shows formatted JSON in preview area

**Implementation:**
```javascript
function exportSceneGraph() {
  const sceneData = {
    entities: entities.map(entity => ({
      name: entity.name,
      attributes: entity.attributes,
      position: {
        x: entity.x,
        y: entity.y
      },
      size: {
        width: entity.width,
        height: entity.height
      },
      color: entity.color
    })),
    relationships: relationships.map(rel => ({
      source: rel.source,
      target: rel.target,
      label: rel.label
    })),
    metadata: {
      version: "2.0-canvas",
      created: new Date().toISOString(),
      entityCount: entities.length,
      relationshipCount: relationships.length,
      canvasSize: {
        width: canvasWidth,
        height: canvasHeight
      }
    }
  };
  
  const output = JSON.stringify(sceneData, null, 2);
  document.getElementById('scene-output').textContent = output;
}
```

### Clipboard Function: `copySceneGraph()`

**Implementation Status:** ✅ **Fully Implemented with Fallback**

**Functionality:**
1. **Scene Generation**: Generates current scene graph data
2. **Modern Clipboard API**: Uses navigator.clipboard when available
3. **Fallback Support**: Legacy document.execCommand for older browsers
4. **User Feedback**: Visual confirmation of copy operation

**Implementation:**
```javascript
function copySceneGraph() {
  exportSceneGraph(); // Generate current scene data
  const output = document.getElementById('scene-output').textContent;
  
  if (navigator.clipboard && window.isSecureContext) {
    // Modern Clipboard API
    navigator.clipboard.writeText(output).then(() => {
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

## Canvas-Based Data Serialization

### Canvas Entity Data Model
The export system serializes Canvas entities with complete positional and visual data:

**Canvas Entity Serialization:**
```javascript
{
  name: entity.name,              // Entity identifier
  attributes: entity.attributes,   // Array of attributes
  position: {
    x: entity.x,                  // Canvas X coordinate
    y: entity.y                   // Canvas Y coordinate
  },
  size: {
    width: entity.width,          // Entity width in pixels
    height: entity.height         // Entity height in pixels
  },
  color: entity.color            // Semantic color value
}
```

### Relationship Serialization
Relationships maintain their simple structure:
```javascript
{
  source: relationship.source,    // Source entity name
  target: relationship.target,    // Target entity name
  label: relationship.label      // Relationship description
}
```

### Enhanced Metadata
Canvas-based export includes additional metadata:
```javascript
{
  version: "2.0-canvas",         // Canvas version identifier
  created: "2024-01-01T00:00:00Z", // ISO timestamp
  entityCount: 5,                // Total entities
  relationshipCount: 8,          // Total relationships
  canvasSize: {
    width: 1200,                 // Canvas width
    height: 800                  // Canvas height
  }
}
```

## Complete Export Format

### JSON Structure
**Comprehensive Export Format:**
```json
{
  "entities": [
    {
      "name": "User",
      "attributes": ["id", "name", "email", "created_at"],
      "position": { "x": 150, "y": 200 },
      "size": { "width": 120, "height": 80 },
      "color": "#667eea"
    },
    {
      "name": "Order", 
      "attributes": ["id", "amount", "date", "status"],
      "position": { "x": 400, "y": 300 },
      "size": { "width": 120, "height": 80 },
      "color": "#764ba2"
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
    "version": "2.0-canvas",
    "created": "2024-01-01T12:00:00.000Z",
    "entityCount": 2,
    "relationshipCount": 1,
    "canvasSize": {
      "width": 1200,
      "height": 800
    }
  }
}
```

## User Feedback System

### Copy Feedback Implementation
**Visual Feedback Function:**
```javascript
function showCopyFeedback(message) {
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

### Success Feedback Messages
- **Export Success**: "Scene graph exported!"
- **Copy Success**: "Scene graph copied to clipboard!"
- **Copy Failure**: "Copy failed. Please manually select and copy the text."

## Alternative Export Formats

### Simplified Text Format
**Use Case**: Quick sharing and documentation
```javascript
function exportSimpleText() {
  let output = "ENTITIES:\n";
  entities.forEach(entity => {
    output += `- ${entity.name}`;
    if (entity.attributes.length > 0) {
      output += ` [${entity.attributes.join(', ')}]`;
    }
    output += `\n`;
  });
  
  output += "\nRELATIONSHIPS:\n";
  relationships.forEach(rel => {
    output += `- ${rel.source} → ${rel.target} (${rel.label})\n`;
  });
  
  return output;
}
```

### CSV Export for Entities
**Use Case**: Spreadsheet integration
```javascript
function exportEntitiesCSV() {
  let csv = "Name,X,Y,Width,Height,Color,Attributes\n";
  entities.forEach(entity => {
    csv += `"${entity.name}",${entity.x},${entity.y},${entity.width},${entity.height},"${entity.color}","${entity.attributes.join(';')}"\n`;
  });
  return csv;
}
```

## Import Functionality (Future Enhancement)

### JSON Import Structure
**Potential Import Function:**
```javascript
function importSceneGraph(jsonData) {
  try {
    const sceneData = JSON.parse(jsonData);
    
    // Clear current scene
    entities.length = 0;
    relationships.length = 0;
    
    // Import entities
    sceneData.entities.forEach(entityData => {
      const entity = {
        name: entityData.name,
        attributes: entityData.attributes || [],
        x: entityData.position.x,
        y: entityData.position.y,
        width: entityData.size.width,
        height: entityData.size.height,
        color: entityData.color
      };
      entities.push(entity);
    });
    
    // Import relationships
    sceneData.relationships.forEach(relData => {
      relationships.push({
        source: relData.source,
        target: relData.target,
        label: relData.label
      });
    });
    
    // Update UI and render
    updateEntitySelectOptions();
    updateEntityList();
    updateRelationshipList();
    requestRender();
    
  } catch (error) {
    console.error('Import failed:', error);
    alert('Failed to import scene graph. Please check the format.');
  }
}
```

## Performance Considerations

### Export Optimization
- **Direct Object Access**: No DOM element queries required
- **Efficient Mapping**: Array.map for clean data transformation  
- **JSON Serialization**: Native JSON.stringify for performance
- **Memory Efficient**: No intermediate data structures

### Large Dataset Support
- **Streaming Export**: Potential for large scene graphs
- **Chunked Processing**: For hundreds of entities
- **Compression**: Optional gzip compression for large exports

## Security Considerations

### Data Sanitization
- **XSS Prevention**: Text content properly escaped in JSON
- **Input Validation**: Robust parsing for import functionality
- **Safe References**: No executable code in exported data

### Clipboard Security
- **Secure Context**: Uses modern Clipboard API when available
- **Fallback Safety**: Legacy method with proper cleanup
- **User Control**: Clear feedback on copy operations

## Browser Compatibility

### Clipboard API Support
- **Modern Browsers**: Full Clipboard API support
- **Legacy Support**: document.execCommand fallback
- **Feature Detection**: Graceful degradation based on browser capabilities

### JSON Support
- **Universal Support**: JSON.stringify/parse available in all modern browsers
- **Error Handling**: Robust error catching for malformed data

This Canvas-based export system provides comprehensive data serialization with modern clipboard integration, supporting the full Canvas entity data model while maintaining backwards compatibility and providing excellent user feedback. 