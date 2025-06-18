# Relationship Management Specification

## Overview
The Relationship Management system handles the creation, visualization, and management of connections between entities in the Entity Relationship Builder. Relationships are directional connections with labels that define how entities interact within the scene graph.

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
- **Display**: Centered on relationship line

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
   - Trigger visual line rendering

## Relationship Visualization System

### SVG Rendering Architecture
Relationships are visualized using SVG elements overlaid on the canvas:

**SVG Container Setup:**
```javascript
svg.innerHTML = `
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="9" refY="5" 
            markerWidth="6" markerHeight="6" orient="auto">
      <polygon points="0 0, 10 5, 0 10" fill="#555" />
    </marker>
  </defs>
`;
```

### Line Rendering Process
For each relationship, the system:

1. **Entity Position Calculation**
   ```javascript
   const from = entities.find(e => e.name === rel.source)?.el;
   const to = entities.find(e => e.name === rel.target)?.el;
   ```

2. **Center Point Calculation**
   ```javascript
   const x1 = from.offsetLeft + from.offsetWidth / 2;
   const y1 = from.offsetTop + from.offsetHeight / 2;
   const x2 = to.offsetLeft + to.offsetWidth / 2;
   const y2 = to.offsetTop + to.offsetHeight / 2;
   ```

3. **SVG Line Creation**
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

4. **Label Text Creation**
   ```javascript
   const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
   text.setAttribute('x', (x1 + x2) / 2);
   text.setAttribute('y', (y1 + y2) / 2 - 5);
   text.textContent = rel.label;
   ```

### Visual Properties
- **Line Color**: #555 (medium gray)
- **Line Width**: 2px
- **Arrow Style**: Filled polygon pointing to target
- **Label Position**: Centered between entities, 5px above line
- **Direction**: Unidirectional from source to target

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
  background: #f8f8f8;
  padding: 5px;
  margin-top: 4px;
  border: 1px solid #ccc;
  border-radius: 4px;
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
   - Trigger SVG line re-rendering
3. **Visual Cleanup**: Remove corresponding SVG elements

### Deletion Implementation
```javascript
deleteBtn.onclick = () => {
  relationships.splice(idx, 1);
  updateRelationshipList();
  updateLines();
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

### Update Function: `updateLines()`
This function handles complete re-rendering of all relationship lines:

1. **SVG Reset**: Clear existing lines and regenerate marker definitions
2. **Relationship Iteration**: Process each relationship in array
3. **Entity Resolution**: Find DOM elements for source and target
4. **Position Calculation**: Determine current entity center points
5. **Line Rendering**: Create SVG line and text elements
6. **DOM Insertion**: Add elements to SVG container

### Error Handling in Updates
```javascript
relationships.forEach(rel => {
  const from = entities.find(e => e.name === rel.source)?.el;
  const to = entities.find(e => e.name === rel.target)?.el;
  if (!from || !to) return; // Skip invalid relationships
  // ... rendering logic
});
```

## Entity Integration

### Dropdown Population
The relationship creation interface depends on entity availability:

**Update Function: `updateEntitySelectOptions()`**
```javascript
[sourceSelect, targetSelect].forEach(select => {
  select.innerHTML = '';
  entities.forEach(e => {
    const option = document.createElement('option');
    option.value = e.name;
    option.textContent = e.name;
    select.appendChild(option);
  });
});
```

### Entity Reference Management
- **Creation**: Relationships store entity names as string references
- **Updates**: Entity name changes require relationship updates
- **Deletion**: Orphaned relationships become unrenderable but remain in array

## Validation and Error Handling

### Input Validation Rules
1. **Source Selection**: Must be a valid entity name
2. **Target Selection**: Must be a valid entity name
3. **Different Entities**: Source ≠ Target requirement
4. **Label Requirement**: Non-empty relationship label

### Error Prevention Strategies
- **Dropdown Constraints**: Only valid entities available for selection
- **Existence Checks**: Verify entities exist before rendering
- **Graceful Degradation**: Skip invalid relationships in rendering

### Current Limitations
- **Duplicate Relationships**: No prevention of identical relationships
- **Orphaned Relationships**: Remain in array after entity deletion
- **Name Changes**: No automatic relationship updates when entity names change

## Performance Considerations

### Rendering Optimization
- **Complete Redraw**: Full SVG regeneration on updates
- **Entity Lookup**: Linear search for entity resolution
- **DOM Manipulation**: Efficient SVG element creation

### Scalability Factors
- **Memory Usage**: Linear growth with relationship count
- **Rendering Performance**: SVG handles moderate relationship numbers well
- **Update Frequency**: Triggered by various user actions

## Data Persistence

### Current Implementation
- **Storage**: In-memory JavaScript array
- **Session**: Data lost on page refresh
- **No DOM Backup**: Unlike entities, no dataset storage

### Data Structure
```javascript
const relationships = [
  { source: "Entity1", target: "Entity2", label: "connects to" },
  { source: "Entity2", target: "Entity3", label: "depends on" }
];
```

## Future Enhancement Opportunities

### Relationship Properties
- **Bidirectional relationships**
- **Relationship types and categories**
- **Visual styling options**
- **Relationship attributes**

### Advanced Features
- **Relationship editing after creation**
- **Multiple relationships between same entities**
- **Relationship templates and presets**
- **Curved line options for complex diagrams** 