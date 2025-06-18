# User Interface Specification

## Overview
The Entity Relationship Builder features a split-panel interface design optimized for simultaneous visualization and control operations. The interface prioritizes usability with clear visual hierarchy and intuitive interaction patterns.

## Layout Architecture

### Primary Layout Structure
```
┌─────────────────────────────────────────────────────────┐
│                    Main Container                        │
│  (display: flex)                                        │
│                                                         │
│  ┌─────────────────────┐  ┌─────────────────────────┐   │
│  │                     │  │                         │   │
│  │    Canvas Area      │  │    Controls Panel       │   │
│  │    (flex: 2)        │  │    (flex: 1)            │   │
│  │                     │  │                         │   │
│  └─────────────────────┘  └─────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Canvas Area (`#canvas`)
- **Purpose**: Primary workspace for entity visualization and interaction
- **Dimensions**: Takes 2/3 of screen width, full viewport height
- **Border**: 2px solid #ccc
- **Positioning**: Relative container for absolute-positioned entities
- **Overflow**: Hidden to contain draggable elements
- **Child Elements**:
  - SVG overlay for relationship lines
  - Dynamic entity elements

### Controls Panel (`#controls`)
- **Purpose**: Control interface for all application functions
- **Dimensions**: Takes 1/3 of screen width, full viewport height
- **Border**: Left border 2px solid #ccc
- **Scrolling**: Vertical scroll for overflow content (overflow-y: auto)
- **Padding**: 10px internal spacing

## Control Card System

### Card Structure
Each functional area is organized as a control card with consistent styling:

```css
.control-card {
  border-radius: 10px;
  padding: 10px;
  margin-bottom: 10px;
  background: #eee;
}
```

### Card Types

#### 1. Entity Creation Card
- **Title**: "Create Entity"
- **Components**:
  - Text input for entity name
  - Textarea for attributes (multi-line)
  - Submit button ("Add Entity")

#### 2. Relationship Creation Card
- **Title**: "Create Relationship"
- **Components**:
  - Source entity dropdown
  - Target entity dropdown
  - Relationship label input
  - Submit button ("Add Relationship")
  - Dynamic relationship list display

#### 3. Entity Management Card
- **Title**: "Entities"
- **Components**:
  - Dynamic list of entity summaries
  - Individual entity editing interfaces

#### 4. Scene Graph Export Card
- **Title**: "Scene Graph"
- **Components**:
  - Export button
  - Copy to clipboard button
  - Output preview area

## Entity Visual Design

### Entity Styling
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

### Visual Properties
- **Shape**: Rounded rectangles (10px border-radius)
- **Colors**: Randomly generated background colors
- **Text**: Black text for contrast
- **Sizing**: User-resizable with minimum constraints
- **Positioning**: Draggable absolute positioning
- **Cursor**: Move cursor to indicate interactivity

## Form Element Styling

### Consistent Form Design
```css
input, textarea, select, button {
  margin-top: 5px;
  display: block;
  width: 100%;
}
```

### Element Types
- **Text Inputs**: Single-line text fields
- **Textareas**: Multi-line text areas for attributes
- **Select Dropdowns**: Entity selection for relationships
- **Buttons**: Action triggers with consistent styling

## Visual Feedback Elements

### Entity Summary Cards
```css
.entity-summary {
  border: 1px solid #aaa;
  border-radius: 5px;
  padding: 5px;
  margin-bottom: 10px;
  background: #fff;
}
```

### Relationship Summary Items
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

## SVG Visualization Layer

### SVG Container
- **Positioning**: Absolute overlay covering entire canvas
- **Pointer Events**: Disabled (pointer-events: none)
- **Z-Index**: Behind entities (z-index: 0)
- **Size**: 100% width and height of canvas

### Visual Elements
- **Relationship Lines**: 2px stroke width, #555 color
- **Arrow Markers**: Directional indicators for relationships
- **Text Labels**: Centered on relationship lines

## Color System

### Color Generation
- **Method**: Random hexadecimal color generation
- **Format**: 6-digit hex codes (#RRGGBB)
- **Application**: Entity background colors for visual distinction

### Color Usage
- **Entity Backgrounds**: Randomly generated unique colors
- **UI Elements**: Consistent grayscale palette
- **Text**: High contrast black text on colored backgrounds

## Responsive Design Considerations

### Flexibility Features
- **Entity Resizing**: CSS resize property for user control
- **Flexible Layout**: Flexbox for panel proportions
- **Scrollable Controls**: Overflow handling for control panel

### Accessibility Features
- **Cursor Indicators**: Visual feedback for interactive elements
- **Color Contrast**: Black text on colored backgrounds
- **Clear Visual Hierarchy**: Distinct sections and groupings

## Interaction Design

### User Interaction Patterns
- **Drag and Drop**: Primary interaction for entity positioning
- **Form Submission**: Standard form controls for data entry
- **Dynamic Updates**: Real-time visual feedback
- **Delete Actions**: Clear deletion buttons with confirmation

### Visual States
- **Hover States**: Cursor changes for interactive elements
- **Active States**: Visual feedback during interactions
- **Selected States**: Focus indicators for form elements 