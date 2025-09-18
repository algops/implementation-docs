# JSON Renderer Implementation Plan

## Overview
This document outlines the complete refactoring of the JSON rendering system to work with the new `fullPath` format and handle all possible JSON structures consistently according to specific header display rules.

## Current Issues

### 1. Path Format Mismatch
- **Problem**: System mixes two approaches:
  - `fullPaths` array uses `{path: "api_response.data.name", type: "key"}` format
  - JsonNode components expect `path: string` and use `selectedPath === path` comparison
  - Search results generate paths with suffixes like `"api_response.data.name:key"`

### 2. Header Display Rules Not Implemented
- **Problem**: No logic for different header display based on JSON structure
- **Impact**: All accordions show the same header format regardless of content
- **Missing**: Rules for showing key nodes, value nodes, or item counts in headers

### 3. Selection Logic Issues
- **Problem**: When selecting search results, both key and value nodes get selected
- **Impact**: Mapping selector doesn't open because it can't distinguish between key and value
- **Root Cause**: `handlePathSelect` strips suffixes, making both nodes think they're selected

### 4. Inconsistent Accordion Structure
- **Problem**: Root level bypasses accordions entirely
- **Impact**: Inconsistent rendering patterns between root and nested levels
- **Missing**: Root accordion with statistics and proper structure

### 5. No Expansion Control for Single Primitive Pairs
- **Problem**: Single primitive key:value pairs still show expansion arrows
- **Impact**: Unnecessary UI complexity for simple values
- **Missing**: `disableExpansion` functionality

## Solution Architecture

### Core Principles
1. **Consistent Path Format**: All components use `fullPath` format with `path` and `type` fields
2. **Statistics-Driven Rendering**: Decisions based on actual counts, not assumptions
3. **Header Display Rules**: Specific logic for different JSON structures
4. **Type-Aware Selection**: Selection logic distinguishes between key and value types
5. **Comprehensive JSON Support**: Handle all possible JSON structures and edge cases

### Header Display Rules
1. **Multiple keys on same level** → Show `{ [x items] }` in header
2. **1 key on level** → Show key node in header  
3. **Primitive value next to 1 key** → Show value node in header
4. **Non-primitive value** → Show items count in header (instead of value node)
5. **Single primitive key:value pair** → Show key + value in header, disable expansion

## Implementation Plan

### Step 1: Update JsonAccordion Component
```typescript
interface JsonAccordionProps {
  // ... existing props
  itemsCount?: string  // Rename from customLabel
  disableExpansion?: boolean  // NEW: Disable expansion for single primitive pairs
}
```
- Add `disableExpansion` prop
- Rename `customLabel` to `itemsCount`
- Hide chevron when `disableExpansion` is true
- Prevent click handling when `disableExpansion` is true

### Step 2: Add Utility Functions
```typescript
// Count structural items for a specific level (basePath)
export function countItems(
  fullPaths: Array<FullPathData>,
  basePath: string,  // The level path to analyze
  data: any  // Original data to determine array vs object
): number

// Render structural count for display (arrays and objects)
export function renderItemsCount(
  data: any,
  basePath: string
): React.ReactNode

// Use getDataType for all type detection (replaces getValueType)
// getDataType(value) returns: 'string' | 'number' | 'boolean' | 'null' | 'array' | 'object' | 'undefined'
```
- **countItems()**: Count structural items (array items or object keys) for statistics
- **renderItemsCount()**: Render structural count (array length or object keys) for display
- **getDataType()**: Single source of truth for type detection throughout the system
- Distinguish between array items (type: "value") and object keys (type: "key") counting

### Step 3: Add New Rendering Functions
```typescript
// JsonRenderer orchestrates these 3 main functions:

// 1. Create root accordion structure (no header content yet)
export function renderRoot(data, fullPaths, basePaths, rootBasePath, ...)
// Purpose: Creates root JsonAccordion structure only
// Identified by: basePath = ""

// 2. Create nested accordion structures (no header content yet)
export function renderNestedLevel(data, basePath, levelBasePath, fullPaths, basePaths, ...)
// Purpose: Creates nested JsonAccordion structure for each level only
// Identified by: basePath (e.g., "api_response.data")

// 3. Process level content and determine header content (multi-step function)
export function renderLevelContent(data, basePath, fullPaths, basePaths, selectedPath, onNodeClick)
// Purpose: 
//   - Calls countItems() to get primitive count for this basePath
//   - Gets actual value from data using getValueAtPath() for structural count
//   - Determines header content based on primitive count vs structural count
//   - Renders primitive key-value pairs for accordion body using fullPaths
//   - Returns { headerContent, bodyContent }
// Identified by: basePath (accordion level)
// Content from: fullPaths filtered for this basePath + actual data values

// Supporting functions used by renderLevelContent (all work with fullPaths):

// Render items count with proper brackets
export function renderItemsCount(value: any): string
// Purpose: Creates "[3 items]" or "{2 items}" strings
// Used for: Header content when showing item counts

// Render individual nodes (identified by fullPaths)
export function renderKeyNode(key, path, fullPaths, selectedPath, onNodeClick)
// Purpose: Renders individual key node
// Identified by: fullPath with type: "key"

export function renderValueNode(value, path, fullPaths, selectedPath, onNodeClick)
// Purpose: Renders individual value node  
// Identified by: fullPath with type: "value"

// Render key-value pairs with colon separator
export function renderKeyValuePair(key, value, path, fullPaths, selectedPath, onNodeClick)
// Purpose: Combines key node + value node with colon, or key node + items count
// Uses: renderKeyNode() and renderValueNode() or renderItemsCount()
// Identified by: fullPath (key) + fullPath (value) pairs
```

### Step 4: Update JsonRenderer Component
- **JsonRenderer orchestrates all 3 main functions**:
  - `renderRoot()` for root accordion structure
  - `renderNestedLevel()` for each nested accordion structure
  - `renderLevelContent()` for each level's content and header decisions
- Use new architecture with statistics-based decisions
- Maintain backward compatibility

### Step 5: Update Selection Logic to Match fullPaths Format
- Update `JsonNode` components to use `path` and `type` from fullPaths
- Update selection comparisons to use `fullPath.path` and `fullPath.type`
- Fix search result selection to work with fullPath format
- Update all path-based logic to use fullPath structure

### Step 6: Update All References
- Replace `customLabel` with `itemsCount` throughout codebase
- Update all rendering function calls to use new architecture
- Update all path handling to use fullPath format

### Step 7: Clean Up Rendering Utils (Final Step)
After implementing the new rendering architecture, clean up the existing `utils/json/rendering-utils.tsx` file:

#### **Functions to Keep (Core Utilities):**
- `getDataType()` - **KEEP** - Can replace both `isPrimitive()` and `getValueType()`
- `getItemCount()` - **KEEP** - Rename to `renderItemsCount()` for consistency
- `generateParentPaths()` - **KEEP** - Still needed for expansion logic
- `renderPath()` - **KEEP** - Still needed for path display formatting

#### **Functions to Remove (Redundant):**
- `isPrimitive()` - **REMOVE** - Redundant with `getDataType()`
- `getValueType()` - **REMOVE** - Redundant with `getDataType()`

#### **Functions to Remove (Replaced by New Architecture):**
- `renderPrimitiveNode()` - Replaced by `renderKeyNode()` and `renderValueNode()`
- `renderArray()` - Replaced by new `renderLevelContent()` approach
- `renderObject()` - Replaced by new `renderLevelContent()` approach
- `renderJsonNode()` - Replaced by new `renderRoot()` and `renderNestedLevel()`

#### **Functions to Keep (Used by Other Components):**
- `getBasePaths()` - **KEEP** - Used by JsonResultMapper and JsonExplorer
- `getFullPaths()` - **KEEP** - Used by JsonResultMapper and JsonExplorer  
- `generatePathData()` - **KEEP** - Used by JsonResultMapper and JsonExplorer
- `generatePath()` - **KEEP** - Used by mapping-utils.ts
- `getPathKey()` - **KEEP** - Used by multiple components
- `getValueAtPath()` - **KEEP** - Used by multiple components

#### **Functions to Move to mapping-utils.ts:**
- `updateBasePathStatistics()` - **MOVE** - Belongs with other mapping functions

#### **Functions to Keep (Expansion Logic):**
- `getStructurePathsFromBasePaths()` - Still needed for "Show Structure"
- `getAllPaths()` - Still needed for "Show All"
- `getIssuedPaths()` - Still needed for "Show Issues"
- `getMappedPaths()` - Still needed for "Show Mapped"
- `getRecommendedPaths()` - Still needed for "Show Recommended"
- `expandPaths()` - Still needed for expansion logic
- `showOnlyPaths()` - Still needed for "Show Only" functionality

#### **Cleanup Actions:**
1. **Remove old rendering functions** that are replaced by new architecture
2. **Keep core utility functions** that are still needed
3. **Keep expansion logic functions** that are still needed
4. **Rename `getItemCount()` to `renderItemsCount()`** for consistency
5. **Replace `isPrimitive()` calls with `getDataType()`** throughout the codebase
6. **Update imports** in components that use the removed functions
7. **Remove unused exports** from the `jsonRenderingUtils` object
8. **Update documentation** to reflect the new architecture

#### **Replace `isPrimitive()` with `getDataType()`:**
```typescript
// Replace all instances of:
if (isPrimitive(value)) { ... }

// With:
const dataType = getDataType(value)
if (dataType !== 'array' && dataType !== 'object') { ... }

```

#### **Functions that need `isPrimitive()` replacement:**
- `getItemCount()` - Replace `isPrimitive(value)` check with `getDataType(value)`
- `getFullPaths()` - Replace `isPrimitive(item)` and `isPrimitive(value)` checks with `getDataType()`
- `renderPrimitiveNode()` - Replace `isPrimitiveValue` checks (but this function will be removed)
- `renderArray()` - Replace `isPrimitiveValue` checks (but this function will be removed)
- `renderObject()` - Replace `isPrimitiveValue` checks (but this function will be removed)
- `renderJsonNode()` - Replace `isPrimitive(data)` check (but this function will be removed)

#### **Final File Structure:**
```typescript
// utils/json/rendering-utils.tsx (cleaned up)
export {
  // Core utilities (keep - 4 functions)
  getDataType, // replaces isPrimitive and getValueType
  renderItemsCount, // renamed from getItemCount, uses getDataType
  generateParentPaths,
  renderPath,
  
  // Path generation (keep - 3 functions)
  getBasePaths, // Used by JsonResultMapper and JsonExplorer
  getFullPaths, // Used by JsonResultMapper and JsonExplorer
  generatePathData, // Used by JsonResultMapper and JsonExplorer
  
  // Path utilities (keep - 3 functions)
  generatePath, // Used by mapping-utils.ts
  getPathKey, // Used by multiple components
  getValueAtPath, // Used by multiple components
  
  // Expansion logic (keep - 7 functions)
  getStructurePathsFromBasePaths,
  getAllPaths,
  getIssuedPaths,
  getMappedPaths,
  getRecommendedPaths,
  expandPaths,
  showOnlyPaths,
  
  // New rendering functions (add - 7 functions)
  countItems, // uses getDataType for type detection
  renderRoot,
  renderNestedLevel,
  renderLevelContent, // uses getDataType for header decisions
  renderKeyNode,
  renderValueNode,
  renderKeyValuePair // uses getDataType for value type detection
}
```

#### **Final Optimization Summary:**
- **Total functions**: 21 (down from 30 - 30% reduction)
- **Single type detection**: `getDataType()` replaces `isPrimitive()` and `getValueType()`
- **Minimal redundancy**: All functions use `getDataType()` for type detection
- **Clean architecture**: Functions work together efficiently
- **Maintained functionality**: All existing features preserved

## Key Principles
1. **fullPaths is source of truth** - everything else gets updated to match its format
2. **Statistics-driven rendering** - decisions based on actual counts, not assumptions
3. **No breaking changes** - keep existing functions for backward compatibility
4. **Clean architecture** - new functions are purpose-built for the new system

## basePaths vs fullPaths Relationship

### **basePaths (Level Statistics)**
- **Purpose**: Contains statistics and metadata for each level
- **Structure**: `Record<string, BasePathData>` where key is the level path
- **Content**: Level statistics (total items, mapped items, status, etc.)
- **Used for**: Header rendering decisions, level structure
- **Example**: `basePaths['api_response.data']` contains statistics for that level

### **fullPaths (Individual Key-Value Pairs)**
- **Purpose**: Contains individual key-value pairs with mappings
- **Structure**: `Array<FullPathData>` with `path` and `type` fields
- **Content**: Individual key/value entries with mapping status
- **Used for**: Content rendering, individual node rendering
- **Example**: `fullPaths.filter(p => p.path.startsWith('api_response.data'))` gets all items for that level

### **Connection Between Them**
- **basePaths** provides level overview and header decisions
- **fullPaths** provides detailed content for each level
- **Filtering**: `fullPaths.filter(p => p.path.startsWith(levelPath))` gets content for a level
- **Statistics**: `basePaths[levelPath]` provides level statistics for headers

### **Critical Identification System**
- **Accordions are identified by basePaths** - Each accordion represents a level in the JSON structure
  - Root accordion: `basePath = ""`
  - Nested accordion: `basePath = "api_response.data"`
- **Nodes/itemsCount are identified by fullPaths** - Each individual key/value pair or item count
  - Key node: `fullPath = { path: "api_response.data.name", type: "key" }`
  - Value node: `fullPath = { path: "api_response.data.name", type: "value" }`
  - Items count: Derived from fullPaths for that basePath level

## What We're NOT Doing
- ❌ Modifying existing `renderArray()` and `renderObject()` functions
- ❌ Changing the fullPath data structure
- ❌ Adding suffixes to paths (fullPaths already has correct format)

## What We ARE Doing
- ✅ Using fullPath format as the standard
- ✅ Creating new rendering functions that work with fullPath format
- ✅ Implementing statistics-based rendering decisions
- ✅ Adding `disableExpansion` functionality for single primitive pairs
- ✅ Fixing selection logic to work with fullPath format

## Rendering Workflow

### Step-by-Step Rendering Process
1. **Start with basePaths and statistics** - Use existing `basePaths` with level statistics from `updateBasePathStatistics`
2. **JsonRenderer orchestrates 3 main functions**:
   - **renderRoot()** - Creates root accordion structure (no header content yet)
   - **renderNestedLevel()** - Creates nested accordion structures (no header content yet)
   - **renderLevelContent()** - Multi-step function for each level:
     - Calls `countItems(fullPaths, basePaths, basePath)` for level statistics
     - Determines header content based on statistics
     - Renders primitive key-value pairs for accordion body
     - Returns both header and body content

### Detailed Flow
```typescript
// JsonRenderer orchestrates all rendering:
export function JsonRenderer({ data, basePaths, fullPaths, ... }) {
  // 1. Start with basePaths (already has statistics from updateBasePathStatistics)
  const rootBasePath = basePaths[''] // Root level
  const nestedBasePaths = Object.keys(basePaths).filter(path => path !== '')
  
  // 2. Create accordion structures (no header content yet)
  const rootAccordion = renderRoot(data, fullPaths, basePaths, rootBasePath, ...)
  
  const nestedAccordions = nestedBasePaths.map(basePath => {
    const levelBasePath = basePaths[basePath]
    return renderNestedLevel(data, basePath, levelBasePath, fullPaths, basePaths, ...)
  })
  
  // 3. Process content and determine headers for each level
  const levelContents = Object.keys(basePaths).map(basePath => {
    const { headerContent, bodyContent } = renderLevelContent(basePath, fullPaths, basePaths, selectedPath, onNodeClick)
    return { basePath, headerContent, bodyContent }
  })
  
  return (
    <div>
      {rootAccordion}
      {nestedAccordions}
      {levelContents.map(({ basePath, headerContent, bodyContent }) => (
        <JsonAccordion
          key={basePath}
          basePath={basePath}
          headerContent={headerContent}
          bodyContent={bodyContent}
          {...otherProps}
        />
      ))}
    </div>
  )
}

// renderLevelContent is a multi-step function:
function renderLevelContent(basePath, fullPaths, basePaths, selectedPath, onNodeClick) {
  // Step 1: Count items for this specific basePath
  const levelCounts = countItems(fullPaths, basePaths, basePath)
  
  // Step 2: Determine header content based on statistics
  const headerContent = determineHeaderContent(levelCounts, basePath, fullPaths, ...)
  
  // Step 3: Render primitive key-value pairs for accordion body
  const bodyContent = renderPrimitiveContent(basePath, fullPaths, ...)
  
  // Step 4: Return both header and body content
  return { headerContent, bodyContent }
}
```

## Header Display Logic Implementation

### Primitive vs Non-Primitive Value Detection
```typescript
// Use getDataType for all type detection (replaces getValueType)
// Direct usage in code:
const dataType = getDataType(value)
if (dataType !== 'array' && dataType !== 'object') {
  // Handle primitive value
}

// Render items count with proper brackets for arrays and objects
export function renderItemsCount(value: any): string {
  if (Array.isArray(value)) {
    return `[${value.length} ${value.length === 1 ? 'item' : 'items'}]`
  }
  if (typeof value === 'object' && value !== null) {
    const keys = Object.keys(value)
    return `{${keys.length} ${keys.length === 1 ? 'item' : 'items'}}`
  }
  return String(value)
}
```

### Header Content Decision Logic
```typescript
// Based on level statistics and value types
function determineHeaderContent(levelCount: LevelStatistics, fullPath: FullPathData) {
  // Rule 1: Multiple keys on same level → Show items count
  if (levelCount.keyItems > 1) {
    return { type: 'itemsCount', content: renderItemsCount(fullPath.value) }
  }
  
  // Rule 2: 1 key on level → Show key node
  if (levelCount.keyItems === 1 && levelCount.valueItems === 0) {
    return { type: 'keyNode', content: renderKeyNode(...) }
  }
  
  // Rule 3: Primitive value next to 1 key → Show value node
  if (levelCount.keyItems === 1 && levelCount.primitiveItems === 1) {
    return { type: 'valueNode', content: renderValueNode(...) }
  }
  
  // Rule 4: Non-primitive value → Show items count
  const dataType = getDataType(fullPath.value)
  if (dataType === 'array' || dataType === 'object') {
    return { type: 'itemsCount', content: renderItemsCount(fullPath.value) }
  }
  
  // Rule 5: Single primitive key:value pair → Show key + value, disable expansion
  if (levelCount.keyItems === 1 && levelCount.primitiveItems === 1) {
    return { 
      type: 'keyValuePair', 
      content: renderKeyValuePair(key, value, path, fullPaths, selectedPath, onNodeClick),
      disableExpansion: true 
    }
  }
}

// renderKeyValuePair function responsibilities:
// 1. Combines key node + value node with colon: "key: value"
// 2. Combines key node + items count with colon: "key: [3 items]"
// 3. Handles proper spacing and alignment
// 4. Manages selection state for both key and value parts
```

## Rendering Data Flow with State Management

### **1. Start with basePaths and Statistics**
- **Input**: JSON data, fullPaths array, basePaths object (with statistics from `updateBasePathStatistics`)
- **What it does**: Uses existing `basePaths` that already contain level statistics
- **Output**: Ready-to-use level statistics for rendering decisions

### **2. Create Accordion Structures (No Header Content Yet)**
- **Function**: `renderRoot(data, fullPaths, basePaths, rootBasePath, ...)`
- **What it does**: Creates root JsonAccordion structure only (accordion shell)
- **Does NOT**: Determine header content (that happens in renderLevelContent)

- **Function**: `renderNestedLevel(data, basePath, levelBasePath, fullPaths, basePaths, ...)`
- **What it does**: Creates nested JsonAccordion structure for each level (accordion shell)
- **Does NOT**: Determine header content (that happens in renderLevelContent)

### **3. Process Level Content and Determine Headers (Multi-step)**
- **Function**: `renderLevelContent(basePath, fullPaths, basePaths, selectedPath, onNodeClick)`
- **What it does**: Multi-step function that:
  - **Step 1**: Calls `countItems(fullPaths, basePaths, basePath)` for level statistics
  - **Step 2**: Determines header content based on statistics
  - **Step 3**: Renders primitive key-value pairs for accordion body
  - **Step 4**: Returns `{ headerContent, bodyContent }`
- **Uses**: `basePath` parameter to know which level it's working on
- **Output**: Both header content and body content for that specific level

### **4. Determine Header Content (Inside renderLevelContent)**
- **Function**: `getDataType(value)` and helper functions (used by decision logic)
- **What it does**: Checks if a value is primitive, array, or object using `getDataType()`
- **Used by**: Header decision logic inside `renderLevelContent` to choose between showing key nodes, value nodes, or item counts

### **5. Combine Key + Value (Inside renderLevelContent)**
- **Function**: `renderKeyValuePair(key, value, path, fullPaths, selectedPath, onNodeClick)`
- **What it does**: Takes a key and value, then combines them with a colon - either "key: value" or "key: [items]"
- **Uses**: `renderKeyNode()`, `renderValueNode()`, and `renderItemsCount()` to build the combination
- **Used by**: Inside `renderLevelContent` for both headers and body content

### **6. Render Individual Components (Inside renderLevelContent)**
- **Functions**: `renderKeyNode()`, `renderValueNode()`, `renderItemsCount()`
- **What they do**: 
  - `renderKeyNode()`: Shows just the key part
  - `renderValueNode()`: Shows just the value part  
  - `renderItemsCount()`: Shows "[3 items]" or "{2 items}" with proper brackets
- **Used by**: Inside `renderLevelContent` by `renderKeyValuePair()` and header decision logic

### **7. Display in Accordion with State Management**
- **Component**: `JsonAccordion`
- **What it does**: Takes the rendered content from `renderLevelContent` and displays it in a collapsible accordion
- **Uses**: The `{ headerContent, bodyContent }` output from `renderLevelContent`
- **New features**: `itemsCount` prop for showing counts, `disableExpansion` for single primitive pairs
- **State Management**: 
  - `expanded` state controlled by parent (JsonExplorer)
  - `selectedPath` state for highlighting selected nodes
  - `onExpandChange` callback to update parent state
  - `onNodeClick` callback to update selection state

### **8. State Management Flow**


#### **JsonRenderer Rendering Flow State Management (1:1 with flow steps):**
```typescript
// JsonRenderer component with rendering flow states
export function JsonRenderer({ data, basePaths, fullPaths, ... }) {
  // Rendering flow states - 1:1 mapping with flow steps
  const [renderingState, setRenderingState] = useState<{
    renderingLevels: boolean
    countingLevels: boolean
    gettingValueTypes: boolean
    renderingPairs: boolean
    renderingNodes: boolean
    displaying: boolean
  }>({
    renderingLevels: false,
    countingLevels: false,
    gettingValueTypes: false,
    renderingPairs: false,
    renderingNodes: false,
    displaying: false
  })
  
  // Logging state for rendering flow
  const [renderLogs, setRenderLogs] = useState<Array<{
    step: string
    timestamp: number
    duration?: number
    path?: string
  }>>([])
  
  // renderRoot and renderNestedLevel functions with state management
  const renderLevels = useCallback((data, basePaths, fullPaths, ...) => {
    const stepStart = performance.now()
    setRenderingState(prev => ({ ...prev, renderingLevels: true }))
    
    console.log('🔍 JsonRenderer: renderingLevels state - creating accordion structures', {
      step: 'renderingLevels',
      basePathsCount: Object.keys(basePaths).length,
      timestamp: new Date().toISOString()
    })
    
    // Create accordion structures (no header content yet)
    const rootAccordion = renderRoot(data, fullPaths, basePaths, ...)
    const nestedAccordions = Object.keys(basePaths)
      .filter(path => path !== '')
      .map(basePath => renderNestedLevel(data, basePath, basePaths, ...))
    
    const stepEnd = performance.now()
    setRenderingState(prev => ({ ...prev, renderingLevels: false }))
    setRenderLogs(prev => [...prev, {
      step: 'renderingLevels',
      timestamp: stepStart,
      duration: stepEnd - stepStart
    }])
    
    return { rootAccordion, nestedAccordions }
  }, [])
  
  // countItems function with state management (called inside renderLevelContent)
  const countItems = useCallback((fullPaths, basePaths, basePath) => {
    const stepStart = performance.now()
    setRenderingState(prev => ({ ...prev, countingLevels: true }))
    
    console.log('🔍 JsonRenderer: countingLevels state - analyzing fullPaths for basePath', {
      step: 'countingLevels',
      basePath,
      fullPathsCount: fullPaths.length,
      timestamp: new Date().toISOString()
    })
    
    // Count items for this specific basePath
    const levelCounts = analyzeLevelStatistics(fullPaths, basePaths, basePath)
    
    const stepEnd = performance.now()
    setRenderingState(prev => ({ ...prev, countingLevels: false }))
    setRenderLogs(prev => [...prev, {
      step: 'countingLevels',
      timestamp: stepStart,
      duration: stepEnd - stepStart,
      path: basePath
    }])
    
    return levelCounts
  }, [])
  
  // getDataType function with state management (called inside renderLevelContent)
  const getDataType = useCallback((value) => {
    const stepStart = performance.now()
    setRenderingState(prev => ({ ...prev, gettingValueTypes: true }))
    
    console.log('🔍 JsonRenderer: gettingValueTypes state - determining value type', {
      step: 'gettingValueTypes',
      value: typeof value,
      timestamp: new Date().toISOString()
    })
    
    // Determine detailed value type
    let dataType
    if (value === null) dataType = 'null'
    else if (Array.isArray(value)) dataType = 'array'
    else if (typeof value === 'object') dataType = 'object'
    else dataType = typeof value
    
    const stepEnd = performance.now()
    setRenderingState(prev => ({ ...prev, gettingValueTypes: false }))
    setRenderLogs(prev => [...prev, {
      step: 'gettingValueTypes',
      timestamp: stepStart,
      duration: stepEnd - stepStart
    }])
    
    return dataType
  }, [])
  
  // renderKeyValuePair function with state management (called inside renderLevelContent)
  const renderKeyValuePair = useCallback((key, value, path, fullPaths, selectedPath, onNodeClick) => {
    const stepStart = performance.now()
    setRenderingState(prev => ({ ...prev, renderingPairs: true }))
    
    console.log('🔍 JsonRenderer: renderingPairs state - combining key and value', {
      step: 'renderingPairs',
      key,
      path,
      timestamp: new Date().toISOString()
    })
    
    // Combine key node + value node with colon, or key node + items count
    const keyNode = renderKeyNode(key, path, fullPaths, selectedPath, onNodeClick)
    const dataType = getDataType(value)
    const valueNode = (dataType !== 'array' && dataType !== 'object')
      ? renderValueNode(value, path, fullPaths, selectedPath, onNodeClick)
      : renderItemsCount(value)
    
    const stepEnd = performance.now()
    setRenderingState(prev => ({ ...prev, renderingPairs: false }))
    setRenderLogs(prev => [...prev, {
      step: 'renderingPairs',
      timestamp: stepStart,
      duration: stepEnd - stepStart,
      path
    }])
    
    return <div className="key-value-pair">{keyNode}: {valueNode}</div>
  }, [])
  
  // renderKeyNode, renderValueNode, renderItemsCount with state management (called inside renderLevelContent)
  const renderNodes = useCallback((type, data, path, fullPaths, selectedPath, onNodeClick) => {
    const stepStart = performance.now()
    setRenderingState(prev => ({ ...prev, renderingNodes: true }))
    
    console.log('🔍 JsonRenderer: renderingNodes state - rendering individual components', {
      step: 'renderingNodes',
      type,
      path,
      timestamp: new Date().toISOString()
    })
    
    // Render individual components based on type
    let nodeComponent
    if (type === 'key') {
      nodeComponent = renderKeyNode(data, path, fullPaths, selectedPath, onNodeClick)
    } else if (type === 'value') {
      nodeComponent = renderValueNode(data, path, fullPaths, selectedPath, onNodeClick)
    } else if (type === 'itemsCount') {
      nodeComponent = renderItemsCount(data)
    }
    
    const stepEnd = performance.now()
    setRenderingState(prev => ({ ...prev, renderingNodes: false }))
    setRenderLogs(prev => [...prev, {
      step: 'renderingNodes',
      timestamp: stepStart,
      duration: stepEnd - stepStart,
      path
    }])
    
    return nodeComponent
  }, [])
  
  // JsonAccordion rendering with state management (final display)
  const renderAccordion = useCallback((props) => {
    const stepStart = performance.now()
    setRenderingState(prev => ({ ...prev, displaying: true }))
    
    console.log('🔍 JsonRenderer: displaying state - rendering accordion', {
      step: 'displaying',
      path: props.path,
      timestamp: new Date().toISOString()
    })
    
    // Display accordion with header and body content from renderLevelContent
    const { headerContent, bodyContent } = props
    
    const stepEnd = performance.now()
    setRenderingState(prev => ({ ...prev, displaying: false }))
    setRenderLogs(prev => [...prev, {
      step: 'displaying',
      timestamp: stepStart,
      duration: stepEnd - stepStart,
      path: props.path
    }])
    
    return <JsonAccordion {...props} headerContent={headerContent} bodyContent={bodyContent} />
  }, [])
}
```

#### **State Management Benefits:**
- **Real-time flow tracking** - See exactly which rendering step is executing
- **Performance monitoring** - Measure time spent in each step
- **Debug rendering issues** - Identify where rendering fails or slows down
- **Production monitoring** - Track rendering performance in real-time
- **Flow visualization** - Complete visibility into rendering pipeline

## Simple Flow:
1. **Start with basePaths and statistics** → Use existing `basePaths` with level statistics
2. **JsonRenderer orchestrates 3 main functions**:
   - **renderRoot()** → Creates root accordion structure (no header content yet)
   - **renderNestedLevel()** → Creates nested accordion structures (no header content yet)
   - **renderLevelContent()** → Multi-step function for each level:
     - **Count items** → `countItems(fullPaths, basePaths, basePath)` for level statistics
     - **Determine header content** → Based on statistics and header display rules
     - **Render primitive content** → `renderKeyValuePair()` combines key + value with colons
     - **Render individual pieces** → `renderKeyNode()`, `renderValueNode()`, `renderItemsCount()`
     - **Return content** → `{ headerContent, bodyContent }`
3. **Show in accordion** → `JsonAccordion` displays header + content with state management

## Expected Outcome
- Consistent rendering for all JSON structures according to header display rules
- Proper selection behavior for key and value nodes
- Unified accordion structure at all levels
- Comprehensive JSON support including edge cases
- Maintainable and extensible codebase

## Migration Strategy

### 1. Backward Compatibility
- Maintain existing interfaces during transition
- Keep old rendering functions unchanged
- Gradual migration of components
- Fallback for old path format

### 2. Testing
- Comprehensive test coverage
- Edge case testing
- Performance benchmarking

### 3. Documentation
- Update component documentation
- Provide migration guides
- Examples for common use cases

## Technical Details

### FullPath Format
```typescript
interface FullPathData {
  path: string;           // "api_response.data.name" (without suffix)
  type: 'key' | 'value';  // The type indicator
  value: any | null;      // The actual value
  status: 'mapped' | 'recommendation' | 'issue' | 'default';
  isSelected?: boolean;
  mappingCount?: number;
  mappings: string[];
}
```

### Selection Logic
```typescript
// Old (Broken)
const isSelected = selectedPath === path;

// New (Fixed)
const isSelected = selectedPath === `${path}:${type}`;
```

### Accordion Structure
```typescript
// Root Level
<JsonAccordion
  basePath=""
  basePathData={rootData}
  fullPaths={rootFullPaths}
  depth={0}
  shadowType="outside"
/>

// Nested Level
<JsonAccordion
  basePath="api_response.data"
  basePathData={nestedData}
  fullPaths={nestedFullPaths}
  depth={1}
  shadowType="inner"
/>
```

## Benefits

### 1. Consistency
- All levels use accordion structure
- Unified path format throughout system
- Consistent selection behavior
- Header display rules applied uniformly

### 2. Flexibility
- Handle any JSON structure
- Support for complex nested data
- Statistics-driven rendering decisions
- Extensible for future requirements

### 3. Performance
- Optimized rendering for large datasets
- Efficient selection and expansion
- Minimal re-rendering
- Statistics calculated once, used everywhere

### 4. Maintainability
- Clear separation of concerns
- Modular rendering functions
- No breaking changes to existing code
- Easy to test and debug

## Success Criteria

### 1. Functional Requirements
- ✅ All JSON structures render correctly according to header display rules
- ✅ Selection works for both key and value nodes
- ✅ Mapping selector opens correctly
- ✅ Accordion structure is consistent
- ✅ Search results work properly
- ✅ Single primitive pairs don't show expansion arrows

### 2. Performance Requirements
- ✅ Large datasets render efficiently
- ✅ Selection and expansion are responsive
- ✅ Memory usage is optimized
- ✅ Statistics calculation is efficient

### 3. Quality Requirements
- ✅ Code is maintainable and testable
- ✅ Documentation is comprehensive
- ✅ Edge cases are handled gracefully
- ✅ Backward compatibility maintained

## Conclusion

This implementation plan provides a comprehensive solution for refactoring the JSON rendering system. By implementing the new rendering functions with statistics-driven decisions and updating the path format throughout the system, we can achieve:

1. **Consistent rendering** for all JSON structures according to specific header display rules
2. **Proper selection behavior** for key and value nodes
3. **Unified accordion structure** at all levels
4. **Comprehensive JSON support** including edge cases
5. **Maintainable and extensible** codebase with no breaking changes

The approach ensures a smooth transition while maintaining system stability and performance, with fullPaths as the source of truth for all rendering decisions.

## Step 8: Integration with JsonResultMapper Architecture

**Goal**: Integrate the new rendering architecture with the existing JsonResultMapper → JsonExplorer → JsonRenderer data flow, ensuring proper prop passing and component hierarchy.

### **Architecture Flow**

```
JsonResultMapper (Parent - Data Processor)
├── Receives: Raw JSON data + Mappings
├── Processes: generatePathData() → basePaths + fullPaths
├── Processes: assignMappingsToFullPaths() → Updated fullPaths
├── Processes: determineBasePathStatuses() → Updated basePaths
├── Processes: countMappings() → Statistics
└── Renders: JsonExplorer as child with basePaths and fullPaths

JsonExplorer (Child - Pure UI Component)
├── Receives: basePaths and fullPaths data from JsonResultMapper
├── Manages: UI state (expansion, selection, search)
├── Renders: PathSearch, ProgressBar, ExpansionControls
└── Renders: JsonRenderer with basePaths and fullPaths

JsonRenderer (Child - New Rendering Architecture)
├── Receives: basePaths + fullPaths from JsonExplorer
├── Uses: New rendering functions (renderRoot, renderNestedLevel, etc.)
├── Generates: headerContent and bodyContent using new functions
├── Renders: JsonAccordion with headerContent and bodyContent props
└── Renders: JsonNode components with new architecture
```

### **Required Changes**

#### **1. JsonResultMapper Integration**
- **Keep**: JsonResultMapper as parent component (working correctly)
- **Update**: Pass `basePaths` and `fullPaths` to JsonExplorer (not `processedData`)
- **Remove**: Any references to `processedData` in component hierarchy

#### **2. JsonExplorer Integration**
- **Remove**: JsonResultMapper import and usage (it's the parent, not child)
- **Fix**: Remove all `processedData` references from useEffect dependencies
- **Update**: Receive `basePaths` and `fullPaths` as props from JsonResultMapper
- **Pass**: `basePaths` and `fullPaths` to JsonRenderer

#### **3. JsonRenderer Integration**
- **Keep**: New rendering architecture (renderRoot, renderNestedLevel, etc.)
- **Update**: Use new rendering functions to generate `headerContent` and `bodyContent`
- **Pass**: Generated `headerContent` and `bodyContent` to JsonAccordion

#### **4. JsonAccordion Integration**
- **Keep**: Current props interface (`headerContent`, `bodyContent`, etc.)
- **Update**: Receive `headerContent` and `bodyContent` from JsonRenderer
- **Remove**: Any old architecture props that conflict with new system

### **Data Flow**

1. **Clear Separation**: JsonResultMapper handles data processing, JsonExplorer handles UI, JsonRenderer handles rendering
2. **No processedData**: Eliminates confusing `processedData` references throughout the system
3. **New Architecture**: JsonRenderer uses new rendering functions while maintaining clean prop interfaces
4. **Proper Hierarchy**: JsonResultMapper → JsonExplorer → JsonRenderer → JsonAccordion

### **Expected Outcome**

- ✅ JsonResultMapper processes data and passes `basePaths` and `fullPaths` to JsonExplorer
- ✅ JsonExplorer manages UI state and passes data to JsonRenderer
- ✅ JsonRenderer uses new architecture to generate `headerContent` and `bodyContent`
- ✅ JsonAccordion receives proper props from new rendering system
- ✅ No `processedData` references or import issues
- ✅ Clean component hierarchy with proper data flow

This integration step ensures the new rendering architecture works seamlessly with the existing JsonResultMapper system while maintaining clean component boundaries and proper data flow.

## Step 9: Fix JsonRenderer Implementation

**Goal**: Fix the fundamental misunderstanding of how the basePath/fullPath system works and implement the correct rendering logic.

### **Root Cause Analysis**

The current JsonRenderer has a **fundamental misunderstanding** of the basePath/fullPath system:

#### **How the basePath/fullPath System Actually Works**

**BasePaths = Accordion Structure & Statistics**
- **Purpose**: Define accordion hierarchy and provide statistics for accordion headers
- **Structure**: Each basePath represents one accordion level
- **Order**: First basePath = root accordion, subsequent basePaths = nested accordions
- **Content**: Only contains accordion metadata (status, statistics, expansion state)

**FullPaths = Node Content Within Accordions**
- **Purpose**: Define individual key/value nodes within each accordion
- **Structure**: Same paths as basePaths but with additional type/mapping info
- **Types**: Each path appears twice - once as `type: "key"` and once as `type: "value"`
- **Content**: Contains node rendering data (status, mappings, selection state)

#### **Example from source-response.json:**

**BasePaths (Accordion Structure):**
```typescript
{
  "api_response": { status: "mapped", statistics: {...} },           // Root accordion
  "api_response.data": { status: "mapped", statistics: {...} },      // Nested accordion
  "api_response.data.companies": { status: "mapped", statistics: {...} }, // Nested accordion
  "api_response.data.companies.tech_corp": { status: "mapped", statistics: {...} }, // Nested accordion
  // ... more nested accordions
}
```

**FullPaths (Node Content):**
```typescript
[
  // Within "api_response" accordion:
  { path: "api_response", type: "key", value: "api_response", status: "mapped" },
  // NO value entry for api_response because it's an object!
  
  // Within "api_response.data" accordion:
  { path: "api_response.data", type: "key", value: "data", status: "mapped" },
  // NO value entry for api_response.data because it's an object!
  
  // Within "api_response.data.companies.tech_corp" accordion:
  { path: "api_response.data.companies.tech_corp.name", type: "key", value: "name", status: "mapped" },
  { path: "api_response.data.companies.tech_corp.name", type: "value", value: "Tech Corporation", status: "mapped" },
  
  // Within "api_response.data.companies.tech_corp.technologies" accordion:
  { path: "api_response.data.companies.tech_corp.technologies", type: "key", value: "technologies", status: "mapped" },
  // NO value entry for technologies because it's an array!
  
  // Array items (primitives only):
  { path: "api_response.data.companies.tech_corp.technologies[0]", type: "value", value: "React", status: "mapped" },
  { path: "api_response.data.companies.tech_corp.technologies[1]", type: "value", value: "Node.js", status: "mapped" },
  // ... more primitive values
]
```

### **Corrected Renderer Architecture**

#### **renderRoot() - Root Accordion (First BasePath)**
- **Input**: The first/shortest basePath (e.g., `"api_response"`)
- **Purpose**: Create the root accordion
- **Content**: Uses fullPaths to populate header and body content

#### **renderNestedLevel() - All Other Accordions**
- **Input**: All other basePaths (e.g., `"api_response.data"`, `"api_response.data.companies"`, etc.)
- **Purpose**: Create nested accordions in hierarchical order
- **Content**: Uses fullPaths to populate header and body content

#### **renderLevelContent() - Accordion Content**
- **Input**: A specific basePath and all fullPaths
- **Purpose**: Generate header and body content for that accordion
- **Logic**: 
  - Filter fullPaths that belong to this basePath
  - Create header content (key nodes, items count, etc.)
  - Create body content (key-value pairs, nested structures)

### **Current Problems in JsonRenderer**

#### **1. Wrong Root Detection**
```typescript
// WRONG: Looking for empty string
const rootBasePath = pathData.basePaths['']

// CORRECT: Get the first/shortest basePath (root accordion)
const rootBasePath = Object.keys(pathData.basePaths)
  .sort((a, b) => a.length - b.length)[0] // "api_response"
```

#### **2. Wrong Nested Detection**
```typescript
// WRONG: Filtering out empty string
const nestedBasePaths = Object.keys(pathData.basePaths).filter(path => path !== '')

// CORRECT: Filter out the root basePath
const nestedBasePaths = Object.keys(pathData.basePaths)
  .filter(path => path !== rootBasePath)
  .sort((a, b) => a.length - b.length) // Sort by hierarchy depth
```

#### **3. Missing Hierarchical Logic**
The renderer should understand that:
- `"api_response"` → Root accordion
- `"api_response.data"` → Nested accordion inside root
- `"api_response.data.companies"` → Nested accordion inside data
- `"api_response.data.companies.tech_corp"` → Nested accordion inside companies

#### **4. Empty renderRoot/renderNestedLevel Functions**
These functions are currently empty shells. They should:
- **renderRoot**: Create accordion for the root basePath
- **renderNestedLevel**: Create accordions for nested basePaths in correct order

### **What the Renderer Should Do**

1. **Identify Root**: Find the shortest basePath (`"api_response"`)
2. **Create Root Accordion**: Use renderRoot() with root basePath
3. **Create Nested Accordions**: Use renderNestedLevel() for all other basePaths in hierarchical order
4. **Populate Content**: Use renderLevelContent() to fill each accordion with its fullPaths

### **Key Insight**

The key insight is that **basePaths define the accordion structure**, and **fullPaths provide the content within each accordion**. The renderer needs to respect this hierarchy and use the basePath order to create the proper nested structure.

### **Critical Understanding: FullPaths Only Contain Primitive Values**

**Important**: The `getFullPaths()` function only creates value entries for **primitive values** (strings, numbers, booleans, null). For nested objects and arrays, it only creates **key entries** and then recurses into the structure.

**This means:**
- ✅ **Primitive values**: Have both key and value entries in fullPaths
- ❌ **Nested objects**: Only have key entries, no value entries
- ❌ **Arrays**: Only have key entries, no value entries
- ✅ **Array items (if primitive)**: Have value entries

**Impact on Rendering:**
- **renderItemsCount()**: Must get actual values from the original data using `getValueAtPath()`
- **countItems()**: Must understand that nested structures won't have value entries
- **Header logic**: Must use actual data values, not fullPaths values for nested structures
- **Body content**: Must filter fullPaths correctly to find only primitive key/value pairs

### **Corrected Counting Logic**

#### **Function Responsibilities**

**countItems() - Count Structural Items**
- **Purpose**: Count structural items (array items or object keys) that are **directly nested** under a specific basePath
- **Logic**: 
  - For arrays: Count `type: "value"` entries (array items) that are directly nested
  - For objects: Count `type: "key"` entries (object keys) that are directly nested
  - **Critical**: Only count items at the immediate level, not deeper nested items
- **Returns**: Number (count of directly nested structural items)
- **Usage**: For statistics and other purposes, not for display

**renderItemsCount() - Render Structural Count for Display**
- **Purpose**: Render the items count display (e.g., "[4 items]", "{6 items}")
- **Logic**: 
  1. Get actual value from data using `getValueAtPath(data, basePath)`
  2. Use `getDataType()` to determine if it's array or object
  3. For arrays: Count array length
  4. For objects: Count object keys
  5. Render appropriate brackets: `[count]` for arrays, `{count}` for objects
- **Returns**: React.ReactNode (the rendered count display)
- **Usage**: For display purposes only

#### **Counting Rules**

**For Arrays:**
- **countItems()**: Counts array items (primitive values in fullPaths)
- **renderItemsCount()**: Shows array length (e.g., `[4 items]`)
- **Example**: Array `["React", "Node.js", "PostgreSQL", "AWS"]`
  - countItems(): 4 (array items)
  - renderItemsCount(): `[4 items]` (array length)

**For Objects:**
- **countItems()**: Counts object keys (type: "key" entries) that are **directly nested**
- **renderItemsCount()**: Shows object keys count (e.g., `{3 items}`)
- **Example**: Object `{ name: "John", age: 30, address: {...} }`
  - countItems(): 3 (directly nested object keys: name, age, address)
  - renderItemsCount(): `{3 items}` (object keys: name, age, address)
  - **Note**: Does NOT count nested items inside `address` object

**Key Insight**: We count **structural items** (array length or object keys) for display, not primitive key-value pairs.

#### **Updated Function Architecture**

```typescript
// countItems() - Pure counting function
function countItems(fullPaths: FullPathData[], basePath: string, data: any): number {
  // Filter fullPaths that are DIRECTLY nested under this basePath
  const levelPaths = fullPaths.filter(p => {
    if (basePath === '') {
      // Root level - paths that don't contain dots or brackets
      return !p.path.includes('.') && !p.path.includes('[')
    } else {
      // Nested level - paths that start with basePath but don't go deeper
      return p.path.startsWith(basePath) && 
             !p.path.substring(basePath.length + 1).includes('.') &&
             !p.path.substring(basePath.length + 1).includes('[')
    }
  })
  
  // Get actual value to determine if it's array or object
  const actualValue = getValueAtPath(data, basePath)
  const dataType = getDataType(actualValue)
  
  if (dataType === 'array') {
    // For arrays: Count array items (type: "value" entries) that are directly nested
    return levelPaths.filter(p => p.type === "value").length
  } else if (dataType === 'object') {
    // For objects: Count object keys (type: "key" entries) that are directly nested
    return levelPaths.filter(p => p.type === "key").length
  }
  
  return 1 // Primitive value
}

// renderItemsCount() - Rendering function that uses actual data
function renderItemsCount(data: any, basePath: string): React.ReactNode {
  // Get actual value from data
  const actualValue = getValueAtPath(data, basePath)
  const dataType = getDataType(actualValue)
  
  // Count structural items
  let count: number
  if (dataType === 'array') {
    count = actualValue.length // Array length
  } else if (dataType === 'object') {
    count = Object.keys(actualValue).length // Object keys count
  } else {
    count = 1 // Primitive value
  }
  
  // Render appropriate brackets
  if (dataType === 'array') {
    return `[${count} items]`
  } else if (dataType === 'object') {
    return `{${count} items}`
  }
  
  return `${count} items`
}
```

#### **Mapping Count Handling**

**Critical**: Each fullPath entry has its own `mappingCount` that is specific to its `type` (key vs value). The system correctly tracks:

- **Key entries**: `{ path: "api_response.data.name", type: "key", mappingCount: 2, mappings: [...] }`
- **Value entries**: `{ path: "api_response.data.name", type: "value", mappingCount: 1, mappings: [...] }`

**Rendering Functions Must Distinguish Types:**

```typescript
// renderKeyNode() - Gets mapping count for key type
function renderKeyNode(key: string, path: string, fullPaths: FullPathData[], ...): React.ReactNode {
  const keyData = fullPaths.find(p => p.path === path && p.type === 'key')
  return (
    <JsonNode
      path={path}
      status={keyData.status || 'default'}
      mappingCount={keyData.mappingCount || 0} // Key-specific mapping count
      isSelected={selectedPath === path}
    />
  )
}

// renderValueNode() - Gets mapping count for value type  
function renderValueNode(value: any, path: string, fullPaths: FullPathData[], ...): React.ReactNode {
  const valueData = fullPaths.find(p => p.path === path && p.type === 'value')
  return (
    <JsonNode
      path={path}
      status={valueData.status || 'default'}
      mappingCount={valueData.mappingCount || 0} // Value-specific mapping count
      isSelected={selectedPath === path}
    />
  )
}
```

**Key Points:**
- ✅ **Current system correctly tracks**: Each fullPath entry has its own `mappingCount` based on its `type`
- ✅ **Mapping assignment works correctly**: `assignMappingsToFullPaths()` assigns mappings to specific key/value entries
- ✅ **Mapping count updates correctly**: `updateMappingCount()` updates counts for each specific entry
- ✅ **Rendering functions must distinguish**: Use `fullPaths.find(p => p.path === path && p.type === 'key/value')` to get the correct mapping count

### **Implementation Steps**

#### **Step 9.1: Fix Root BasePath Detection**
```typescript
// Replace line 136 in JsonRenderer
const rootBasePath = Object.keys(pathData.basePaths)
  .sort((a, b) => a.length - b.length)[0] // Get shortest path (root)
```

#### **Step 9.2: Fix Nested BasePath Detection**
```typescript
// Replace line 137 in JsonRenderer
const nestedBasePaths = Object.keys(pathData.basePaths)
  .filter(path => path !== rootBasePath)
  .sort((a, b) => a.length - b.length) // Sort by hierarchy depth
```

#### **Step 9.3: Implement renderRoot Function**
```typescript
export function renderRoot(
  data: any,
  fullPaths: Array<FullPathData>,
  basePaths: Record<string, BasePathData>,
  rootBasePath: string,
  selectedPath?: string,
  onNodeClick?: (path: string, label: string) => void,
  onExpandChange?: (expanded: boolean) => void
): React.ReactNode {
  // Create root accordion for the root basePath
  const rootBasePathData = basePaths[rootBasePath]
  
  return (
    <JsonAccordion
      key={rootBasePath}
      itemsCount={rootBasePath}
      expanded={rootBasePathData?.isExpanded || false}
      onExpandChange={onExpandChange}
      selectedPath={selectedPath}
      onNodeClick={onNodeClick}
    >
      {/* Content will be populated by renderLevelContent */}
    </JsonAccordion>
  )
}
```

#### **Step 9.4: Implement renderNestedLevel Function**
```typescript
export function renderNestedLevel(
  data: any,
  basePath: string,
  levelBasePath: BasePathData,
  fullPaths: Array<FullPathData>,
  basePaths: Record<string, BasePathData>,
  selectedPath?: string,
  onNodeClick?: (path: string, label: string) => void,
  onExpandChange?: (expanded: boolean) => void
): React.ReactNode {
  // Create nested accordion for this basePath
  return (
    <JsonAccordion
      key={basePath}
      itemsCount={basePath}
      expanded={levelBasePath?.isExpanded || false}
      onExpandChange={onExpandChange}
      selectedPath={selectedPath}
      onNodeClick={onNodeClick}
    >
      {/* Content will be populated by renderLevelContent */}
    </JsonAccordion>
  )
}
```

#### **Step 9.5: Fix JsonRenderer Accordion Creation Logic**
```typescript
// Replace lines 164-201 in JsonRenderer
const rootAccordion = rootBasePath ? (
  <JsonAccordion
    key={rootBasePath}
    itemsCount={rootBasePath}
    expanded={pathData.basePaths[rootBasePath]?.isExpanded || false}
    onExpandChange={(expanded) => onNodeExpand?.(rootBasePath)}
    selectedPath={selectedPath}
    onNodeClick={onNodeClick}
  >
    {renderLevelContent(rootBasePath, pathData.fullPaths, pathData.basePaths, selectedPath, onNodeClick).bodyContent}
  </JsonAccordion>
) : null

const nestedAccordions = nestedBasePaths.map(basePath => {
  const levelBasePath = pathData.basePaths[basePath]
  const levelContent = renderLevelContent(basePath, pathData.fullPaths, pathData.basePaths, selectedPath, onNodeClick)
  
  return (
    <JsonAccordion
      key={basePath}
      itemsCount={basePath}
      expanded={levelBasePath?.isExpanded || false}
      onExpandChange={(expanded) => onNodeExpand?.(basePath)}
      selectedPath={selectedPath}
      onNodeClick={onNodeClick}
    >
      {levelContent.bodyContent}
    </JsonAccordion>
  )
}).filter(Boolean)
```

#### **Step 9.6: Update renderLevelContent to Handle Primitive-Only FullPaths**
```typescript
// Update renderLevelContent function to correctly handle the fact that fullPaths only contain primitives
export function renderLevelContent(
  data: any,
  basePath: string,
  fullPaths: Array<FullPathData>,
  basePaths: Record<string, BasePathData>,
  selectedPath?: string,
  onNodeClick?: (path: string, label: string) => void
): { headerContent: React.ReactNode, bodyContent: React.ReactNode } {
  
  // Step 1: Count structural items for this specific basePath
  const structuralCount = countItems(fullPaths, basePath, data)
  
  // Step 2: Get actual value from data for structural count (not from fullPaths)
  const actualValue = getValueAtPath(data, basePath)
  const dataType = getDataType(actualValue)
  
  // Step 3: Determine header content based on statistics and actual data
  const headerContent = determineHeaderContent(structuralCount, basePath, actualValue, dataType, fullPaths, selectedPath, onNodeClick)
  
  // Step 4: Render primitive key-value pairs for accordion body (only primitives exist in fullPaths)
  const bodyContent = renderPrimitiveContent(basePath, fullPaths, selectedPath, onNodeClick)
  
  return { headerContent, bodyContent }
}
```

#### **Step 9.7: Update determineHeaderContent to Use Actual Data Values**
```typescript
// Update determineHeaderContent to get actual values from data, not fullPaths
function determineHeaderContent(
  structuralCount: number, // Count of structural items from countItems()
  basePath: string,
  actualValue: any, // Get from getValueAtPath(data, basePath)
  dataType: string, // Get from getDataType(actualValue)
  fullPaths: Array<FullPathData>,
  selectedPath?: string,
  onNodeClick?: (path: string, label: string) => void
): React.ReactNode {
  
  // Rule 1: Multiple items on same level → Show items count
  if (structuralCount > 1) {
    return renderItemsCount(actualValue, basePath) // Use actual value for structural count
  }
  
  // Rule 2: 1 item on level → Show key node
  if (structuralCount === 1 && dataType === 'object') {
    const keyEntry = fullPaths.find(p => p.path === basePath && p.type === 'key')
    return renderKeyNode(keyEntry?.value || basePath, basePath, fullPaths, selectedPath, onNodeClick)
  }
  
  // Rule 3: Array with 1 item → Show items count
  if (structuralCount === 1 && dataType === 'array') {
    return renderItemsCount(actualValue, basePath) // Use actual value for structural count
  }
  
  // Rule 4: Non-primitive value → Show items count
  if (dataType === 'array' || dataType === 'object') {
    return renderItemsCount(actualValue, basePath) // Use actual value for structural count
  }
  
  // Rule 5: Single primitive key:value pair → Show key + value, disable expansion
  if (structuralCount === 1 && dataType !== 'array' && dataType !== 'object') {
    const keyEntry = fullPaths.find(p => p.path === basePath && p.type === 'key')
    const valueEntry = fullPaths.find(p => p.path === basePath && p.type === 'value')
    return renderKeyValuePair(keyEntry?.value, valueEntry?.value, basePath, fullPaths, selectedPath, onNodeClick)
  }
}
```

#### **Step 9.8: Ensure Type-Specific Mapping Count Handling**
```typescript
// CRITICAL: All rendering functions must distinguish between key and value types for mapping counts

// renderKeyNode() - Must get key-specific mapping count
function renderKeyNode(key: string, path: string, fullPaths: FullPathData[], selectedPath?: string, onNodeClick?: (path: string, label: string) => void): React.ReactNode {
  const keyData = fullPaths.find(p => p.path === path && p.type === 'key')
  return (
    <JsonNode
      path={path}
      status={keyData?.status || 'default'}
      mappingCount={keyData?.mappingCount || 0} // Key-specific mapping count
      isSelected={selectedPath === path}
      onMappingChange={() => onNodeClick?.(path, key)}
    />
  )
}

// renderValueNode() - Must get value-specific mapping count
function renderValueNode(value: any, path: string, fullPaths: FullPathData[], selectedPath?: string, onNodeClick?: (path: string, label: string) => void): React.ReactNode {
  const valueData = fullPaths.find(p => p.path === path && p.type === 'value')
  return (
    <JsonNode
      path={path}
      status={valueData?.status || 'default'}
      mappingCount={valueData?.mappingCount || 0} // Value-specific mapping count
      isSelected={selectedPath === path}
      onMappingChange={() => onNodeClick?.(path, String(value))}
    />
  )
}

// renderKeyValuePair() - Must handle both key and value mapping counts separately
function renderKeyValuePair(key: string, value: any, path: string, fullPaths: FullPathData[], selectedPath?: string, onNodeClick?: (path: string, label: string) => void): React.ReactNode {
  const keyNode = renderKeyNode(key, path, fullPaths, selectedPath, onNodeClick)
  const valueNode = renderValueNode(value, path, fullPaths, selectedPath, onNodeClick)
  
  return (
    <div className="key-value-pair">
      {keyNode}: {valueNode}
    </div>
  )
}
```

**Key Requirements:**
- ✅ **Always use `fullPaths.find(p => p.path === path && p.type === 'key/value')`** to get the correct entry
- ✅ **Each JsonNode gets its own mapping count** based on its specific type
- ✅ **Key and value nodes can have different mapping counts** - this is correct behavior
- ✅ **Selection logic must include type** - use `${path}:${type}` format for selection

### **Expected Outcome**

After implementing these fixes:

- ✅ **Correct Root Detection**: Finds the actual root basePath (`"api_response"`)
- ✅ **Proper Hierarchy**: Creates accordions in correct nested order
- ✅ **Working renderRoot/renderNestedLevel**: Functions create proper accordion structures
- ✅ **Content Population**: Each accordion gets populated with its fullPaths content
- ✅ **Hierarchical Structure**: Accordions are properly nested according to basePath hierarchy

### **Key Benefits**

1. **Correct Understanding**: Renderer now understands basePath/fullPath system correctly
2. **Proper Hierarchy**: Accordions are created in the right nested order
3. **Content Integration**: Each accordion gets its proper content from fullPaths
4. **Maintainable Code**: Clear separation between structure (basePaths) and content (fullPaths)

This step fixes the fundamental architectural misunderstanding and ensures the renderer works correctly with the basePath/fullPath system.


## Step 10: Fix Current Critical Issues

**Goal**: Fix counting logic, array value rendering, and accordion status styling.

### **Issue 1: Incorrect Counting Logic**

**Problem**: `countItems` function counts from `basePaths` instead of `fullPaths`, causing wrong counts in headers.

**Root Cause**: `countItems` uses `basePaths` filtering instead of `fullPaths` counting.

**Current State**:
- `api_response` shows `{1 item}` instead of `{3 items}`
- `data` shows `{6 items}` instead of `{4 items}`
- `tech_corp` shows `{21 items}` instead of `{9 items}`

**Solution**:
1. **Fix `countItems` function** (lines 311-359):
   - Change signature: `countItems(fullPaths, basePath)` (remove `data` and `allBasePaths`)
   - Filter `fullPaths` for direct children: `fullPaths.filter(p => isDirectChild(p.path, basePath))`
   - Count `type: 'key'` entries for objects, `type: 'value'` entries for arrays

**Expected Result**: `api_response : {3 items}`, `data : {4 items}`, `tech_corp : {9 items}`

### **Issue 2: Array Values Not Rendering**

**Problem**: Arrays show counts instead of individual values in accordion body.

**Root Cause**: `getFullPaths` doesn't create value entries for array items, and `renderPrimitiveContent` doesn't handle array values.

**Solution**:
1. **Update `getFullPaths` function** (lines 216-230):
   - Add value entries for ALL array items: `paths.push({path: itemPath, type: 'value', value: item})`

2. **Update `renderPrimitiveContent` function** (lines 1005-1046):
   - Add array value rendering: filter `fullPaths` for array values and render as `JsonNode` components

**Expected Result**: `technologies` shows individual values (React, Node.js, Python) as value nodes

### **Issue 3: Missing Status-Based Shadows**

**Problem**: Accordions don't show visual status indicators from `basePaths` status.

**Root Cause**: `renderRoot` and `renderNestedLevel` functions don't pass `accordionStatus` prop.

**Solution**:
1. **Update `renderRoot` function** (lines 819-852):
   - Add `accordionStatus={basePaths[rootBasePath].status}` prop to `JsonAccordion`

2. **Update `renderNestedLevel` function** (lines 867-899):
   - Add `accordionStatus={basePaths[basePath].status}` prop to `JsonAccordion`

**Expected Result**: Accordions show different shadows/colors based on mapping status

### **Implementation Order**

1. **Fix `countItems` to use `fullPaths`** (Issue 1) - Change from `basePaths` filtering to `fullPaths` counting
2. **Update `getFullPaths` for array values** (Issue 2) - Add value entries for ALL array items
3. **Update `renderPrimitiveContent`** (Issue 2) - Add array value rendering logic
4. **Update `renderRoot` and `renderNestedLevel`** (Issue 3) - Add `accordionStatus` props

### **Functions to Modify**

1. **`countItems` function** (lines 311-359):
   - Change signature: `countItems(fullPaths, basePath)` (remove `data` and `allBasePaths`)
   - For objects: count `type: 'key'` entries that start with `basePath + '.'` and have no additional dots
   - For arrays: count `type: 'value'` entries that start with `basePath + '['` and have no additional dots

2. **`getFullPaths` function** (lines 216-230):
   - Add value entries for ALL array items: `paths.push({path: itemPath, type: 'value', value: item})`

3. **`renderPrimitiveContent` function** (lines 1005-1046):
   - Add array value rendering: filter `fullPaths` for array values and render as `JsonNode` components

4. **`renderRoot` function** (lines 819-852):
   - Add `accordionStatus={basePaths[rootBasePath].status}` prop to `JsonAccordion`

5. **`renderNestedLevel` function** (lines 867-899):
   - Add `accordionStatus={basePaths[basePath].status}` prop to `JsonAccordion`

### **Expected Results**

- **Counting**: Correct item counts (`api_response : {3 items}`, `data : {4 items}`, `tech_corp : {9 items}`)
- **Arrays**: Individual array values rendered as expandable nodes (React, Node.js, Python)
- **Shadows**: Status-based visual indicators on accordions