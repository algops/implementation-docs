# JSON Explorer Implementation Plan

## Overview
Restructure the JSON system architecture to make JsonExplorer the central orchestrator that manages a unified data structure containing all paths and their statistics. The goal is to create a single source of truth for all path-related data while reusing existing utility functions.

## Architecture Goals

### Current Architecture:
```
JsonExplorer (Parent)
├── PathSearch (Search UI)
├── JsonRenderer (Rendering Engine)
│   ├── JsonAccordion (Collapsible sections)
│   ├── JsonNode (Individual elements)
│   └── rendering-utils.tsx (Rendering logic)
└── Mappers (Separate components)
    ├── JsonRequestMapper
    ├── JsonResponseMapper  
    ├── JsonResultMapper
    └── ProgressBar (Currently in mappers)
```

### New Architecture:
```
JsonExplorer (Central Orchestrator)
├── ProgressBar (Moved from mappers)
├── PathSearch (Search UI)
├── JsonRenderer (Pure Rendering - Same Level as Mappers)
│   ├── JsonAccordion
│   ├── JsonNode
│   └── rendering-utils.tsx
└── Mappers (Pure Mapping Logic - Same Level as Renderer)
    ├── JsonRequestMapper
    ├── JsonResponseMapper
    ├── JsonResultMapper
    └── mapping-utils.ts
```

## Implementation Steps

### Step 1: Create Unified Data Structure and Update Function Names

**Goal**: Define a single data structure that contains all paths and their statistics, plus rename functions for consistency.

**Data Structure**:
```typescript
interface BasePathData {
  status: 'mapped' | 'recommendation' | 'issue' | 'default'
  isExpanded?: boolean               // For accordion expansion state
  statistics: {                      // Statistics for accordion display
    mapped: number
    recommendations: number
    issues: number
    options: number
  }
}

interface FullPathData {
  status: 'mapped' | 'recommendation' | 'issue' | 'default'
  isSelected?: boolean               // For node selection state
  mappingCount?: number              // Number of available mappings for this node
  type: 'key' | 'value'             // Node type (key or value)
  value: any | null                 // Actual value (only for primitives, null for objects/arrays)
  mappings: string[]                // Array of mapping UUIDs that apply to this path
}
```

**JsonExplorer State**:
```typescript
const [basePaths, setBasePaths] = useState<Record<string, BasePathData>>({})
const [fullPaths, setFullPaths] = useState<Record<string, FullPathData>>({})
```

**Function Rename**:
```typescript
// In rendering-utils.tsx
export function generateParentPaths(path: string): string[] {  // Renamed from generateParentPrefixes
  const { pathArray: segments } = generateMappingFromPath(path)
  const prefixes: string[] = []
  let current = ''
  
  for (const segment of segments) {
    current = current ? `${current}.${segment}` : segment
    prefixes.push(current)
  }
  
  return prefixes
}
```

**onPathSelect Signature Update**:
```typescript
// Update signature across all components
onPathSelect: (path: string, parentPaths?: string[]) => void
```

**Real-World Example** (based on source-response.json and source-mappings.json):

**BasePaths** (for accordions):
```typescript
const basePaths = {
  // Root level - base paths for accordions
  "api_response": { 
    status: "mapped", 
    isExpanded: true,
    statistics: { mapped: 8, recommendations: 2, issues: 1, options: 22 }
  },
  
  // Nested object level - base paths for accordions
  "api_response.data": { 
    status: "mapped", 
    isExpanded: true,
    statistics: { mapped: 6, recommendations: 2, issues: 1, options: 18 }
  },
  "api_response.data.companies": { 
    status: "mapped", 
    isExpanded: true,
    statistics: { mapped: 4, recommendations: 1, issues: 0, options: 12 }
  },
  "api_response.data.people": { 
    status: "mapped", 
    isExpanded: false,
    statistics: { mapped: 2, recommendations: 1, issues: 1, options: 6 }
  },
  
  // Company objects - base paths for accordions
  "api_response.data.companies.tech_corp": { 
    status: "mapped", 
    isExpanded: true,
    statistics: { mapped: 3, recommendations: 0, issues: 0, options: 8 }
  },
  "api_response.data.companies.startup_inc": { 
    status: "mapped", 
    isExpanded: false,
    statistics: { mapped: 1, recommendations: 1, issues: 0, options: 4 }
  },
  
  // Nested company properties - base paths for accordions
  "api_response.data.companies.tech_corp.financial": { 
    status: "mapped", 
    isExpanded: true,
    statistics: { mapped: 1, recommendations: 0, issues: 0, options: 2 }
  },
  "api_response.data.companies.tech_corp.location": { 
    status: "mapped", 
    isExpanded: false,
    statistics: { mapped: 0, recommendations: 0, issues: 0, options: 2 }
  },
  
  // Array level - base paths for accordions
  "api_response.data.people.employees": { 
    status: "mapped", 
    isExpanded: false,
    statistics: { mapped: 2, recommendations: 1, issues: 1, options: 4 }
  },
  "api_response.data.companies.tech_corp.technologies": { 
    status: "default", 
    isExpanded: false,
    statistics: { mapped: 0, recommendations: 0, issues: 0, options: 4 }
  },
  
  // Array items rendered as accordions (when array item is object/array)
  "api_response.data.people.employees[0]": { 
    status: "mapped", 
    isExpanded: true,
    statistics: { mapped: 2, recommendations: 0, issues: 0, options: 4 }
  },
  "api_response.data.people.employees[1]": { 
    status: "mapped", 
    isExpanded: false,
    statistics: { mapped: 1, recommendations: 1, issues: 0, options: 3 }
  },
  "api_response.data.companies.tech_corp.technologies[0]": { 
    status: "default", 
    isExpanded: false,
    statistics: { mapped: 0, recommendations: 0, issues: 0, options: 1 }
  },
  "api_response.data.documents[0]": { 
    status: "mapped", 
    isExpanded: true,
    statistics: { mapped: 3, recommendations: 0, issues: 0, options: 6 }
  }
}
```

**FullPaths** (for nodes):
```typescript
const fullPaths = {
  // Object/Array keys - no values (null)
  "api_response": { status: "mapped", isSelected: false, mappingCount: 2, type: "key", value: null, mappings: ["1342107f-2903-4962-a94d-860de1e1a225", "d7313b3a-b033-4992-817c-ec73fdcd9751"] },
  "api_response.data": { status: "mapped", isSelected: false, mappingCount: 1, type: "key", value: null, mappings: [] },
  "api_response.data.companies": { status: "mapped", isSelected: false, mappingCount: 1, type: "key", value: null, mappings: [] },
  "api_response.data.people": { status: "mapped", isSelected: false, mappingCount: 1, type: "key", value: null, mappings: [] },
  "api_response.data.people.employees": { status: "mapped", isSelected: false, mappingCount: 1, type: "key", value: null, mappings: [] },
  
  // Primitive keys and values - have actual values
  "api_response.status": { status: "mapped", isSelected: false, mappingCount: 1, type: "key", value: "status", mappings: [] },
  "api_response.status": { status: "mapped", isSelected: false, mappingCount: 1, type: "value", value: "success", mappings: ["1342107f-2903-4962-a94d-860de1e1a225"] },
  
  "api_response.data.companies.tech_corp.website": { status: "mapped", isSelected: false, mappingCount: 2, type: "key", value: "website", mappings: [] },
  "api_response.data.companies.tech_corp.website": { status: "mapped", isSelected: true, mappingCount: 3, type: "value", value: "https://techcorp.com", mappings: ["m1n2o3p4-q5r6-7890-1234-567890123456", "2b4bb76e-fa24-4790-9e9b-ab05fcbd76f8", "2d37fb1b-0d63-4a60-be1c-79657fb8bcdf"] },
  
  "api_response.data.companies.tech_corp.name": { status: "mapped", isSelected: false, mappingCount: 1, type: "key", value: "name", mappings: [] },
  "api_response.data.companies.tech_corp.name": { status: "mapped", isSelected: false, mappingCount: 2, type: "value", value: "Tech Corp", mappings: ["v0w1x2y3-z4a5-6789-0123-456789012345", "y3z4a5b6-c7d8-9012-3456-789012345678"] },
  
  "api_response.data.companies.tech_corp.financial.revenue": { status: "recommendation", isSelected: false, mappingCount: 1, type: "key", value: "revenue", mappings: [] },
  "api_response.data.companies.tech_corp.financial.revenue": { status: "recommendation", isSelected: false, mappingCount: 1, type: "value", value: 1000000, mappings: ["c725dc12-4a0d-43cb-bcd4-a4242cc199b9"] },
  
  "api_response.data.companies.tech_corp.financial.currency": { status: "issue", isSelected: false, mappingCount: 1, type: "key", value: "currency", mappings: [] },
  "api_response.data.companies.tech_corp.financial.currency": { status: "issue", isSelected: false, mappingCount: 1, type: "value", value: "USD", mappings: ["56a2e0d6-d373-4348-8371-0d1b087c413e"] },
  
  // Array items - only values for primitives
  "api_response.data.people.employees[0]": { status: "mapped", isSelected: false, mappingCount: 0, type: "value", value: null, mappings: [] },
  "api_response.data.people.employees[0].email": { status: "mapped", isSelected: false, mappingCount: 1, type: "key", value: "email", mappings: [] },
  "api_response.data.people.employees[0].email": { status: "mapped", isSelected: false, mappingCount: 3, type: "value", value: "john.doe@techcorp.com", mappings: ["p4q5r6s7-t8u9-0123-4567-890123456789", "person-obj-1"] },
  
  "api_response.data.people.employees[0].name": { status: "mapped", isSelected: false, mappingCount: 1, type: "key", value: "name", mappings: [] },
  "api_response.data.people.employees[0].name": { status: "mapped", isSelected: false, mappingCount: 1, type: "value", value: "John Doe", mappings: ["5ca594e7-2180-433b-97a9-3634b7643d1e"] },
  
  // Technologies array items - primitive values
  "api_response.data.companies.tech_corp.technologies[0]": { status: "mapped", isSelected: false, mappingCount: 1, type: "value", value: "React", mappings: ["6e13adbd-67eb-4c86-951b-64cf4dda757e"] },
  "api_response.data.companies.tech_corp.technologies[1]": { status: "default", isSelected: false, mappingCount: 0, type: "value", value: "Node.js", mappings: [] },
  "api_response.data.companies.tech_corp.technologies[2]": { status: "default", isSelected: false, mappingCount: 0, type: "value", value: "TypeScript", mappings: [] },
  "api_response.data.companies.tech_corp.technologies[3]": { status: "default", isSelected: false, mappingCount: 0, type: "value", value: "PostgreSQL", mappings: [] }
}
```

### Step 2: Renderer Creates All Paths and Passes to Explorer

**Goal**: Modify JsonRenderer to generate the unified data structure and pass it to JsonExplorer.

**Current Functions Analysis**:

**`getStructurePaths(data)`** - Generates accordion paths:
- **Logic**: Traverses JSON structure, adds paths for objects/arrays that can be expanded
- **Output**: Set of paths like `["api_response", "api_response.data.people.employees", "api_response.data.people.employees[0]"]`
- **Purpose**: Accordion expansion state (becomes basePaths)

**`getApplicablePaths(data)`** - Generates node paths used also as search data:
- **Logic**: Traverses JSON structure, creates `:key` and `:value` paths for all nodes
- **Output**: Array of `{path, value, status}` objects like `[{path: "api_response:key", value: "api_response", status: "default"}]`
- **Purpose**: Search functionality + node paths (becomes fullPaths)

**New Functions to Create**:
```typescript
// Extract base path generation logic from getStructurePaths
function getBasePaths(data: any): string[] {
  // Pure path generation - just traverse and collect accordion paths
  // Returns: ["api_response", "api_response.data.people.employees", "api_response.data.people.employees[0]", ...]
}

// Extract full path generation logic from getApplicablePaths  
function getFullPaths(data: any): Array<{path: string, type: 'key' | 'value', value: any | null}> {
  // Pure path generation - just traverse and collect node paths with type and value
  // For 'key' type: extract key name from path (e.g., "website" from "api_response.data.companies.tech_corp.website")
  // For 'value' type: use getValueAtPath(data, path) to extract actual values for primitives, null for objects/arrays
  // Returns: [{path: "api_response", type: "key", value: "api_response"}, {path: "api_response", type: "value", value: {...}}, ...]
}

// Convert paths to our unified data structure
function generatePathData(data: any): {
  basePaths: Record<string, BasePathData>
  fullPaths: Record<string, FullPathData>
} {
  // 1. Call getBasePaths() to get accordion paths
  // 2. Call getFullPaths() to get node paths
  // 3. Convert to BasePathData and FullPathData structures
  // 4. Set default values (status: "default", isExpanded: false, etc.)
}
```

**JsonRenderer Changes**:
- Remove `getJsonPaths` and `getApplicablePaths` logic (moved to new functions)
- Call `generatePathData()` to get unified structure
- Pass `basePaths` and `fullPaths` to JsonExplorer via callback
- Receive updated path data from JsonExplorer for rendering

### Step 3: Update PathSearch to Use FullPaths

**Goal**: Modify PathSearch to read from the new fullPaths structure instead of old search results format.

**Current PathSearch Analysis**:
- **Input**: `searchResults` (array of `{path, value, status}` objects with :key/:value suffixes)
- **Input**: `selectedPath` (string)
- **Logic**: Filters results based on search query, calls `onPathSelect` when clicked
- **Expansion**: Uses `generateParentPrefixes` to expand parent accordions

**New PathSearch Integration**:
- **Input**: `fullPaths` (Record<string, FullPathData> with clean paths and type/value fields)
- **Input**: `selectedPath` (string) 
- **Logic**: Filter fullPaths based on search query, call `onPathSelect` when clicked
- **Expansion**: Use `generateParentPrefixes` to expand parent accordions in basePaths

**Current Navigation Logic** (from JsonExplorer):
```typescript
// When PathSearch calls onPathSelect(path)
const handlePathSelect = useCallback((path: string) => {
  // Strip suffix for accordion expansion logic
  const cleanPath = path.replace(/:key$|:value$/, '')
  
  // Force expand all parent accordions and the path itself
  const pathsToExpand = generateParentPaths(cleanPath)  // Renamed from generateParentPrefixes
  const newExpandedPaths = new Set(expandedPaths)
  pathsToExpand.forEach(path => newExpandedPaths.add(path))
  newExpandedPaths.add(cleanPath) // Also expand the selected path itself
  
  setExpandedPaths(newExpandedPaths)
}, [expandedPaths])
```

**Key Changes**:
1. **Convert fullPaths to search format**: Transform `Record<string, FullPathData>` to searchable array
2. **Update filtering logic**: Search through path strings and values from fullPaths
3. **Handle path selection**: Call `onPathSelect` with clean path and parent paths
4. **Update expansion logic**: Use `generateParentPaths` to expand parent accordions

**PathSearch Implementation**:
```typescript
// Update props interface
interface PathSearchProps {
  fullPaths: Record<string, FullPathData>  // Changed from searchResults: PathSearchResult[]
  onPathSelect: (path: string, parentPaths?: string[]) => void  // Updated signature
  placeholder?: string
  className?: string
  selectedPath?: string
}

// Convert fullPaths to searchable format
const searchResults = useMemo(() => {
  return Object.entries(fullPaths).map(([path, data]) => ({
    path,
    value: data.value,
    status: data.status
  }))
}, [fullPaths])

// Handle path selection with parent paths
const handleResultSelect = useCallback((path: string) => {
  // Generate parent paths for expansion
  const parentPaths = generateParentPaths(path)
  
  // Call onPathSelect with both path and parent paths
  onPathSelect(path, parentPaths)
  setSearchQuery('')
  setIsOpen(false)
}, [onPathSelect])
```

**Note**: This step focuses only on Search functionality.

**Status**: ✅ **COMPLETED** - PathSearch scrolling issue fixed
- Added automatic scrolling for arrow key navigation
- Used `scrollIntoView` with smooth behavior
- Added `data-search-result-index` attributes for reliable targeting
- Arrow keys now properly scroll selected items into view

**Testing Required**: 
- Test arrow key navigation in PathSearch dropdown
- Verify selected items scroll into view smoothly
- Test with long lists that exceed dropdown height
- Verify scrolling works in both directions (up/down)

### Step 4: Implement Bulk Expansion Controls

**Goal**: Add bulk expansion functionality with dropdown controls for different expansion strategies.

**Expansion Functions**:
```typescript
// Keep existing getStructurePaths function as is
function getStructurePaths(data: any): Set<string> {
  // Existing function - traverses JSON and returns all expandable paths
  // Returns: Set of all paths like ["api_response", "api_response.data.people.employees", "api_response.data.people.employees[0]", "api_response.data.people.employees[1]", ...]
}

// New function: Get structure paths from basePaths with [0] logic
function getStructurePathsFromBasePaths(basePaths: Record<string, BasePathData>): string[] {
  // Logic: Select structure paths from basePaths, but only [0] for arrays
  // - Include all object paths (e.g., "api_response", "api_response.data")
  // - Include array containers (e.g., "api_response.data.people.employees")
  // - For array items, only include [0] (first item as template)
  // - Exclude other array items ([1], [2], etc.)
  
  return Object.keys(basePaths).filter(path => {
    // Include root level paths
    if (!path.includes('.')) return true
    
    // Include object properties (not array items)
    if (!path.includes('[')) return true
    
    // For array items, only include [0] (first item as template)
    if (path.includes('[')) {
      return path.endsWith('[0]')
    }
    
    return false
  })
}

// New expansion functions
function getAllPaths(basePaths: Record<string, BasePathData>): string[] {
  // Return all base paths
  return Object.keys(basePaths)
}

function getIssuedPaths(basePaths: Record<string, BasePathData>): string[] {
  // Return all base paths with status "issue"
  return Object.entries(basePaths)
    .filter(([_, data]) => data.status === 'issue')
    .map(([path, _]) => path)
}

function getMappedPaths(basePaths: Record<string, BasePathData>): string[] {
  // Return all base paths with status "mapped"
  return Object.entries(basePaths)
    .filter(([_, data]) => data.status === 'mapped')
    .map(([path, _]) => path)
}

function getRecommendedPaths(basePaths: Record<string, BasePathData>): string[] {
  // Return all base paths with status "recommendation"
  return Object.entries(basePaths)
    .filter(([_, data]) => data.status === 'recommendation')
    .map(([path, _]) => path)
}

// Main expansion function that handles parent paths
function expandPaths(
  targetPaths: string[],
  basePaths: Record<string, BasePathData>,
  setBasePaths: (paths: Record<string, BasePathData>) => void
): void {
  const newBasePaths = { ...basePaths }
  
  targetPaths.forEach(path => {
    // Generate all parent paths for this target path
    const parentPaths = generateParentPaths(path)
    
    // Expand the target path and all its parents
    const pathsToExpand = [...parentPaths, path]
    
    pathsToExpand.forEach(expandPath => {
      if (newBasePaths[expandPath]) {
        newBasePaths[expandPath] = {
          ...newBasePaths[expandPath],
          isExpanded: true
        }
      }
    })
  })
  
  setBasePaths(newBasePaths)
}
```

**Expansion Options**:
```typescript
type ExpansionOption = 
  | 'show-all'      // Expand all base paths
  | 'show-issues'   // Expand paths with status "issue"
  | 'show-structure' // Expand structure paths (from getStructurePaths)
  | 'show-mapped'   // Expand paths with status "mapped"
  | 'show-recommended' // Expand paths with status "recommendation"

const expansionOptions = [
  { value: 'show-all', label: 'Show All' },
  { value: 'show-issues', label: 'Show Issues' },
  { value: 'show-structure', label: 'Show Structure' },
  { value: 'show-mapped', label: 'Show Mapped' },
  { value: 'show-recommended', label: 'Show Recommended' }
]
```

**JsonExplorer Integration**:
```typescript
// Add expansion state and handler
const [expansionMode, setExpansionMode] = useState<ExpansionOption>('show-structure')

const handleExpansionChange = useCallback((option: ExpansionOption) => {
  setExpansionMode(option)
  
  let targetPaths: string[] = []
  
  switch (option) {
    case 'show-all':
      targetPaths = getAllPaths(basePaths)
      break
    case 'show-issues':
      targetPaths = getIssuedPaths(basePaths)
      break
    case 'show-structure':
      targetPaths = getStructurePathsFromBasePaths(basePaths)
      break
    case 'show-mapped':
      targetPaths = getMappedPaths(basePaths)
      break
    case 'show-recommended':
      targetPaths = getRecommendedPaths(basePaths)
      break
  }
  
  expandPaths(targetPaths, basePaths, setBasePaths)
}, [basePaths, data])

// Add dropdown to JsonExplorer UI
<div className="px-4 py-2 flex-shrink-0">
  <Select value={expansionMode} onValueChange={handleExpansionChange}>
    <SelectTrigger className="w-48">
      <SelectValue placeholder="Expansion mode" />
    </SelectTrigger>
    <SelectContent>
      {expansionOptions.map(option => (
        <SelectItem key={option.value} value={option.value}>
          {option.label}
        </SelectItem>
      ))}
    </SelectContent>
  </Select>
</div>
```

**Files to Update**:
- `utils/json/rendering-utils.tsx` - Add expansion functions
- `components/json/interface/json-explorer.tsx` - Add dropdown and expansion logic
- `components/ui/select.tsx` - Ensure Select component is available

### Step 5: Determine BasePath Statuses and Accordion Shadows

**Goal**: Determine statuses for basePaths that will control accordion shadow colors and visual indicators.

**Current Accordion Shadow System**:
```typescript
// From json-accordion.tsx - 4 core statuses with shadow colors
const statusShadows = {
  default: "shadow-default",
  recommendation: "shadow-recommendation", 
  mapped: "shadow-success",
  issue: "shadow-warning"
}

const insetStatusShadows = {
  default: "shadow-inset-default",
  recommendation: "shadow-inset-recommendation",
  mapped: "shadow-inset-success", 
  issue: "shadow-inset-warning"
}
```

**BasePath Status Determination**:
```typescript
// Function to determine basePath status based on children fullPaths
function setPathStatus(
  basePath: string,
  fullPaths: Record<string, FullPathData>
): 'mapped' | 'recommendation' | 'issue' | 'default' {
  // Find all children fullPaths that belong to this basePath
  const childrenPaths = Object.keys(fullPaths).filter(fullPath => 
    fullPath.startsWith(basePath + '.') || fullPath.startsWith(basePath + '[')
  )
  
  // Check children statuses with priority: issue > recommendation > mapped > default
  for (const path of childrenPaths) {
    const status = fullPaths[path]?.status || 'default'
    if (status === 'issue') return 'issue'
  }
  
  for (const path of childrenPaths) {
    const status = fullPaths[path]?.status || 'default'
    if (status === 'recommendation') return 'recommendation'
  }
  
  for (const path of childrenPaths) {
    const status = fullPaths[path]?.status || 'default'
    if (status === 'mapped') return 'mapped'
  }
  
  return 'default'
}

// Function to update all basePath statuses
function determineBasePathStatuses(
  basePaths: Record<string, BasePathData>,
  fullPaths: Record<string, FullPathData>
): Record<string, BasePathData> {
  const updatedBasePaths = { ...basePaths }
  
  Object.keys(basePaths).forEach(basePath => {
    const status = setPathStatus(basePath, fullPaths)
    updatedBasePaths[basePath] = {
      ...updatedBasePaths[basePath],
      status
    }
  })
  
  return updatedBasePaths
}
```

**Status Priority System**:
```typescript
// Status priority (highest to lowest) - determines basePath status
const statusPriority = {
  'issue': 4,        // Highest priority - yellow shadow
  'recommendation': 3, // Light blue shadow  
  'mapped': 2,       // Green shadow
  'default': 1       // Lowest priority - default shadow
}
```

**Accordion Shadow Integration**:
```typescript
// JsonRenderer will pass basePath status to JsonAccordion
<JsonAccordion
  accordionStatus={basePaths[path]?.status || 'default'}
  // ... other props
/>

// JsonAccordion will use this status for shadow colors
const primaryStatus = accordionStatus || 'default'
const shadowClass = statusShadows[primaryStatus] || statusShadows.default
```

**Visual Indicators**:
- **`default`** → Default shadow (neutral)
- **`mapped`** → Green shadow (`shadow-success`)
- **`recommendation`** → Light blue shadow (`shadow-recommendation`) - hsl(187 100% 42%)
- **`issue`** → Yellow shadow (`shadow-warning`) - hsl(43 96% 56%)

**Files to Update**:
- `utils/json/mapping-utils.ts` - Add status determination functions
- `components/json/interface/json-renderer.tsx` - Pass basePath status to accordions
- `components/accordions/json-accordion.tsx` - Use basePath status for shadows

### Step 6: Set Up Statuses in FullPaths

**Goal**: JsonResultMapper reads source-mappings.json and assigns mapping statuses to fullPaths using existing functions with simple renames.

**Functions to Reuse and Rename**:
- **`isPathAtLevel()`** → **`validateMappingforFullPath()`** - Keep existing wildcard logic
- **`getNodeStatus()`** → **`getFullPathState()`** - Keep existing priority logic

**New Functions to Create**:
- **`assignMappingsToFullPaths(mappings: Mappings, fullPaths: Record<string, FullPathData>): Record<string, FullPathData>`**
  - Iterate through mappings and fullPaths
  - Use `validateMappingforFullPath()` to check if mapping applies (handles wildcards)
  - Use `getFullPathState()` to determine status
  - Assign mapping UUID to `mappings` array
  - Update `status` and `mappingCount`

- **`updateMappingCount(fullPaths: Record<string, FullPathData>): Record<string, FullPathData>`**
  - Update `mappingCount` based on `mappings` array length
  - Called when mappings are assigned

**FullPathData Structure** (already defined in Step 1):
```typescript
interface FullPathData {
  status: 'mapped' | 'recommendation' | 'issue' | 'default'
  isSelected?: boolean
  mappingCount?: number
  type: 'key' | 'value'
  value: any | null
  mappings: string[] // Array of mapping UUIDs
}
```

**Implementation Process**:
1. **JsonResultMapper** reads `source-mappings.json`
2. **For each mapping**:
   - Use `generatePathFromMapping()` to create fullPath
   - Use `validateMappingforFullPath()` to check if mapping applies (handles wildcards)
   - Use `getFullPathState()` to determine status
   - Assign mapping UUID to `mappings` array
   - Update `status` and `mappingCount`
3. **Update fullPaths** with assigned mappings and statuses

**Wildcard Handling**:
- **Single Star (`*`)**: Matches any field at current level
- **Double Star (`**`)**: Matches any field at any nested level
- Existing `isPathAtLevel` function already handles wildcards correctly
- No new wildcard functions needed

**Integration Points**:
- **JsonResultMapper**: Calls `assignMappingsToFullPaths` with mappings data
- **JsonExplorer**: Receives updated `fullPaths` with statuses and mappings
- **JsonRenderer**: Uses status information for visual indicators
- **PathSearch**: Can filter by status using the updated data

**Testing Considerations**:
- Test with various wildcard patterns
- Verify status assignment priority
- Test mapping count updates
- Ensure wildcard matching works correctly

### Step 7: Mapping Statistics and Progress Bar Content

**Goal**: Modify `countMappings` function to work with `basePaths`/`fullPaths` structure and update statistics accordingly.

**Function to Modify**:
- **`countMappings()`** - Update to work with new data structure

**New Function Signature**:
```typescript
function countMappings(
  basePath: string, 
  fullPaths: Record<string, FullPathData>
): {mapped: number, recommendations: number, issues: number, options: number}
```

**How it works**:
1. **Filter fullPaths** by basePath scope (paths that start with basePath)
2. **Count by status**:
   - `mapped` - count fullPaths with status "mapped"
   - `recommendations` - count fullPaths with status "recommendation" 
   - `issues` - count fullPaths with status "issue"
   - `options` - count fullPaths with status "default" (remaining mapping options)
3. **Update basePath statistics** with calculated counts

**Implementation Process**:
1. **For root level** (`basePath = ""`):
   - Count all fullPaths
   - Update root basePath statistics
   - Pass to ProgressBar

2. **For each accordion** (`basePath = "api_response"`, etc.):
   - Filter fullPaths that start with basePath
   - Count statuses for that scope
   - Update basePath.statistics
   - Pass to JsonAccordion

**Integration Points**:
- **JsonExplorer**: Call `countMappings` for root and all basePaths
- **ProgressBar**: Receive root-level statistics
- **JsonAccordion**: Receive accordion-specific statistics from basePathData.statistics

**Files to Update**:
- `utils/json/mapping-utils.ts` - Modify `countMappings` function
- `components/json/interface/json-explorer.tsx` - Call `countMappings` and update basePaths
- `components/others/progress-bar.tsx` - No changes needed (same input format)

### Step 8: Direct Integration with Rendering System

**Goal**: Update JsonRenderer and rendering utilities to work directly with the new `basePaths`/`fullPaths` structure without unnecessary conversions.

**Integration Strategy**:
- **JsonRenderer** receives `basePaths` and `fullPaths` directly from JsonExplorer
- **JsonNode** components use `fullPaths` data directly for individual key/value pairs
- **JsonAccordion** components use `basePaths` data directly for collapsible sections
- **Rendering utilities** work with the new data structure without conversion

**JsonRenderer Props Update**:
```typescript
interface JsonRendererProps {
  data: any
  basePaths: Record<string, BasePathData>  // For accordions
  fullPaths: Record<string, FullPathData>  // For nodes
  onPathSelect?: (path: string, parentPaths?: string[]) => void
  // ... other props
}
```

**JsonNode Props Update**:
```typescript
interface JsonNodeProps {
  fullPathData: FullPathData  // Direct access to new format
  // ... other props
}
```

**JsonAccordion Props Update**:
```typescript
interface JsonAccordionProps {
  basePathData: BasePathData  // Direct access to new format
  // ... other props
}
```

**Rendering Functions Updates**:
- **`renderPrimitiveNode()`** - Use `fullPaths` directly for node data
- **`renderArrayNode()`** - Use `basePaths` for accordion, `fullPaths` for nodes
- **`renderObjectNode()`** - Use `basePaths` for accordion, `fullPaths` for nodes
- **`renderJsonNode()`** - Pass correct data to child renderers

**Path Generation Replacement**:
- Replace `getJsonPaths()` with direct access to `basePaths` keys
- Replace `getApplicablePaths()` with direct access to `fullPaths` keys
- Update path selection to use new signature with parent paths

**Benefits**:
- No unnecessary data conversions
- Better performance (no transformation overhead)
- Clearer separation of concerns
- Single source of truth for all data

### Step 9: Rename Rendering Functions for Clarity

**Goal**: Rename rendering functions to better reflect what they actually render (accordions + nodes vs just nodes).

**Functions to Rename**:
- **`renderArrayNode()`** → **`renderArray()`** - Renders array accordion + nodes inside
- **`renderObjectNode()`** → **`renderObject()`** - Renders object accordion + nodes inside
- **`renderPrimitiveNode()`** - Keep as is (only renders individual nodes)
- **`renderJsonNode()`** - Keep as is (dispatcher function)

**Why This Naming is Better**:
- **`renderArray()`** and **`renderObject()`** clearly indicate they handle entire data structures
- They create accordions as containers and render nodes inside
- **`renderPrimitiveNode()`** keeps "Node" because it only renders individual nodes
- Names accurately reflect what each function renders

**Files to Update**:
- `utils/json/rendering-utils.tsx` - Rename function definitions and exports
- `components/json/interface/json-renderer.tsx` - Update function calls
- Any other files that import these functions

**Function Responsibilities**:
- **`renderPrimitiveNode()`** - Uses `fullPaths` for individual nodes only
- **`renderArray()`** - Uses `basePaths` for accordion, `fullPaths` for nodes inside
- **`renderObject()`** - Uses `basePaths` for accordion, `fullPaths` for nodes inside
- **`renderJsonNode()`** - Dispatcher that calls appropriate function based on data type

## Data Flow

### New Data Flow:
```
JsonExplorer (Central Hub)
├── Receives: Raw JSON data + Mappings
├── Calls: generateUnifiedPathData() → BasePathData[] + FullPathData[]
├── Calls: processMappingData() → Updated BasePathData[] + FullPathData[]
├── Manages: basePaths + fullPaths state
├── Passes: basePaths + fullPaths to Renderer
├── Passes: root statistics to ProgressBar
└── Controls: ProgressBar, Search, UI state
```

### Component Responsibilities:

**JsonExplorer (Main Controller)**:
- **UI Management**: Controls all UI components and layout
- **Progress Bar**: Manages and displays progress bar
- **State Coordination**: Orchestrates communication between renderer and mappers
- **Search & Navigation**: Maintains search functionality and navigation state
- **Data Flow**: Receives data from mappers, passes rendering info to renderer

**JsonRenderer (Pure Rendering)**:
- **Rendering Only**: Focuses solely on rendering JSON structures
- **No State Management**: Receives all data from JsonExplorer
- **No Progress Logic**: No longer manages progress calculations
- **Clean Interface**: Simple props for data and rendering info

**Mappers (Pure Mapping Logic)**:
- **Mapping Only**: Focus solely on mapping logic and calculations
- **No UI Components**: No longer contain ProgressBar or other UI elements
- **Data Processing**: Generate mapping data and statistics
- **Clean Output**: Provide structured data to JsonExplorer

## Available Functions

### Mapping Functions (mapping-utils.ts)

#### Path Translation Functions
- **`generateMappingFromPath`**
  - Input: A path string like "companies.tech_corp.name:key"
  - How it works: Takes a path and splits it into parts, removing special suffixes like ":key" or ":value" and converting it to an array format
  - Output: An object with the path as an array, the type (key/value), and the clean path without suffixes

- **`getMappingType`**
  - Input: A path string with potential ":key" or ":value" suffix
  - How it works: Checks if the path ends with ":key" or ":value" to determine the node type
  - Output: Returns "key", "value", or undefined

- **`generatePathFromMapping`**
  - Input: An array of path segments and a node type ("key" or "value")
  - How it works: Joins the path segments with dots and adds the appropriate suffix
  - Output: Both a base path (without suffix) and full path (with suffix)

#### Path Matching Functions
- **`isMappingApplicableToPath`**
  - Input: A mapping path array and an accordion path array
  - How it works: Checks if a mapping can be applied to a specific path by comparing path segments, allowing wildcards (* and **) to match any values
  - Output: True if the mapping applies to the path, false otherwise

- **`isMappingInJson`**
  - Input: A mapping path array and JSON data
  - How it works: Recursively traverses the JSON data to verify that the mapped fields actually exist in the data structure
  - Output: True if the mapped fields exist in the JSON, false otherwise

- **`isPathAtLevel`**
  - Input: A JSON path string and a mapping path array
  - How it works: Uses pattern matching to check if a path matches a mapping pattern, handling wildcards for flexible matching
  - Output: True if the path matches the mapping pattern at the correct level

#### Statistics and Analysis Functions
- **`countMappings`**
  - Input: An accordion path, mappings data, and JSON data
  - How it works: Counts how many mappings are applicable to a specific path, categorizing them by status (mapped, recommendations, issues)
  - Output: An object with counts for each mapping status and options count

- **`getNodeStatus`**
  - Input: A path, mappings data, data type, and optional node type
  - How it works: Determines the visual status of a node by checking all applicable mappings and returning the highest priority status
  - Output: Status string ("default", "mapped", "recommendation", or "issue")

- **`getApplicableMappings`**
  - Input: A path, mappings data, and node type
  - How it works: Finds all mappings that can be applied to a specific path and node type, filtering by compatibility
  - Output: Array of mapping options with their details

- **`getCurrentMapping`**
  - Input: A path and mappings data
  - How it works: Finds the currently active mapping for a specific path by matching path patterns and types
  - Output: Object with source, target, and type information, or undefined if no mapping exists

- **`countMappingOptions`**
  - Input: An accordion path and JSON data
  - How it works: Counts the total number of mappable paths under a specific accordion scope
  - Output: Number representing total mappable paths

- **`getAccordionLevelStatus`**
  - Input: An accordion path, mappings data, and JSON data
  - How it works: Determines the status of an accordion by checking the status of its immediate children
  - Output: Status string based on the highest priority status of child elements

#### Management Functions
- **`handleMappingChange`**
  - Input: A path, mapping ID, mapping options, and change callback
  - How it works: Handles the process of changing a mapping by finding the selected mapping and calling the change callback
  - Output: No return value, triggers the callback function

- **`generateMappingOptions`**
  - Input: Mappings data
  - How it works: Creates a flat list of all available mapping options from all sources
  - Output: Array of mapping options with their details

- **`generateNodeInfo`**
  - Input: JSON data and mappings data
  - How it works: Pre-processes all paths in the JSON data to generate rendering information including status, mapping options, and accordion status
  - Output: Object with rendering information for every path in the JSON structure

### Rendering Functions (rendering-utils.tsx)

#### Utility Functions
- **`isPrimitive`**
  - Input: Any value
  - How it works: Checks if a value is a primitive type (string, number, boolean, null, undefined)
  - Output: True if primitive, false if object or array

- **`getDataType`**
  - Input: Any value
  - How it works: Determines the data type of a value, handling special cases like arrays and null
  - Output: String representing the data type

- **`getItemCount`**
  - Input: Any value
  - How it works: Creates a formatted count display for arrays and objects, showing item counts with proper singular/plural handling
  - Output: Formatted string like "[3 items]" or "{1 item}"

- **`generatePath`**
  - Input: Parent path, key, and optional node type
  - How it works: Creates a complete path string by combining parent path with key, handling arrays and objects differently
  - Output: Complete path string with optional node type suffix

- **`getPathKey`**
  - Input: A path string
  - How it works: Extracts the last key segment from a path by finding the last separator
  - Output: The last key segment

- **`getValueAtPath`**
  - Input: JSON object and path string
  - How it works: Traverses the JSON object using the path to retrieve the value at that location
  - Output: The value at the specified path or undefined

- **`getStructurePaths`**
  - Input: JSON data
  - How it works: Recursively traverses the JSON structure to find all possible paths that can be expanded
  - Output: Set of all expandable paths

- **`generateParentPrefixes`**
  - Input: A target path
  - How it works: Creates all parent path prefixes needed to expand to a nested path
  - Output: Array of parent paths

- **`renderPath`**
  - Input: A path string and optional value
  - How it works: Formats a path for display by replacing dots with chevrons and adding value information for value nodes
  - Output: Formatted path string for display

#### Main Rendering Functions
- **`renderPrimitiveNode`**
  - Input: Primitive value, path, depth, and various rendering parameters
  - How it works: Renders primitive values as JsonNode components, handling both array items and object properties differently
  - Output: React node with key/value JsonNode components or single value node

- **`renderArrayNode`**
  - Input: Array data, path, depth, and various rendering parameters
  - How it works: Renders arrays as collapsible JsonAccordion components with array items, handling nested structures
  - Output: React node with JsonAccordion containing array items

- **`renderObjectNode`**
  - Input: Object data, path, depth, and various rendering parameters
  - How it works: Renders objects as collapsible JsonAccordion components with object properties
  - Output: React node with JsonAccordion containing object properties

- **`renderJsonNode`**
  - Input: Any JSON data, path, depth, and various rendering parameters
  - How it works: Main dispatcher function that determines the data type and calls the appropriate renderer
  - Output: React node rendered by the appropriate renderer function

## Benefits of This Approach

1. **Single Source of Truth**: All path data in one place
2. **Efficient Updates**: Only re-render what changed
3. **Clean Separation**: Each component has clear responsibilities
4. **Easy Testing**: Mock the pathDataMap for component testing
5. **Scalable**: Easy to add new path-related features
6. **Reuse Existing Code**: Leverage all current utility functions
7. **No New Functions**: Modify existing functions rather than creating new ones

## Step 10: Clean Up and Integration

**Goal**: Fix the broken data flow, remove unused functions, and properly integrate all components with the new architecture.

### **Current State Analysis**

**What's Working:**
- ✅ All new functions exist and are properly implemented
- ✅ Function renames and signature updates are complete
- ✅ JsonRenderer generates `pathData` with `basePaths` and `fullPaths`
- ✅ PathSearch scrolling fix is working
- ✅ Basic component structure is in place
- ✅ ProgressBar is properly integrated in JsonExplorer
- ✅ PathSearch is properly integrated in JsonExplorer
- ✅ Expansion controls UI is implemented

**What's Broken:**
- ❌ **Data Flow is Completely Broken**: JsonExplorer never receives or uses the generated `pathData`
- ❌ **basePaths State is Empty**: JsonExplorer has `basePaths` state but it's never populated
- ❌ **No Mapping Integration**: Mapping data is never processed or assigned to `fullPaths`
- ❌ **No Status Calculation**: Accordion statuses are never calculated from `fullPaths`
- ❌ **No Statistics Calculation**: ProgressBar and accordion statistics are never calculated
- ❌ **Expansion Controls Don't Work**: All expansion functions fail because `basePaths` is empty
- ❌ **Props Mismatch**: JsonRenderer still uses old props instead of new data structure
- ❌ **Unused Imports**: Several unused imports in components
- ❌ **Unused Refs**: Unused refs in JsonExplorer
- ❌ **Duplicate ProgressBar**: ProgressBar appears in both JsonExplorer and showcase page

### **Critical Issues to Fix**

#### **Issue 1: Missing Data Flow Integration**
**Problem**: JsonRenderer generates `pathData` but JsonExplorer never receives it
**Current**: JsonRenderer calls `onPathsGenerated` with just path strings
**Needed**: JsonExplorer needs to receive and manage `basePaths` and `fullPaths` data

#### **Issue 2: Empty basePaths State**
**Problem**: JsonExplorer's `basePaths` state is never populated with actual data
**Current**: `basePaths` is always empty `{}`
**Needed**: `basePaths` must be populated when `handlePathsGenerated` is called

#### **Issue 3: No Mapping Processing**
**Problem**: Mapping data is never processed or assigned to `fullPaths`
**Current**: `fullPaths` always have `status: 'default'` and empty `mappings: []`
**Needed**: JsonExplorer must call `assignMappingsToFullPaths()` when mapping data changes

#### **Issue 4: No Status Calculation**
**Problem**: Accordion statuses are never calculated from `fullPaths`
**Current**: All accordions have `status: 'default'`
**Needed**: JsonExplorer must call `determineBasePathStatuses()` when `fullPaths` change

#### **Issue 5: No Statistics Calculation**
**Problem**: ProgressBar and accordion statistics are never calculated
**Current**: ProgressBar shows static `{mapped: 0, recommendations: 0, issues: 0, total: 0}`
**Needed**: JsonExplorer must call `countMappings()` for root and accordion statistics

#### **Issue 6: Props Mismatch**
**Problem**: JsonRenderer still uses old `nodeRenderingInfo` and `accordionStats` props
**Current**: JsonRenderer receives old props but should use new data structure
**Needed**: JsonRenderer should receive `basePaths` and `fullPaths` as props

### **Functions to Remove**

#### **Completely Unused Functions (Safe to Remove):**
- `getStructurePaths()` - Replaced by `getBasePaths()`
- `isMappingApplicableToPath()` - Replaced by `validateMappingforFullPath()`
- `isMappingInJson()` - No longer needed with new structure
- `countMappingOptions()` - Replaced by `countMappings()`
- `getAccordionLevelStatus()` - Replaced by `setPathStatus()`

#### **Potentially Unused Functions (Need Verification):**
- `getCurrentMapping()` - May not be needed
- `getApplicableMappings()` - May not be needed
- `handleMappingChange()` - May not be needed
- `generateMappingOptions()` - May not be needed
- `generateNodeInfo()` - Replaced by new path generation

#### **Unused Imports to Remove:**
- JsonRenderer: `getStructurePaths` import (line 9)
- JsonExplorer: `Divider` import (line 6) - not used anywhere
- JsonExplorer: All expansion function imports (until data flow is fixed)

#### **Unused Refs to Remove:**
- JsonExplorer: `selectedNodeRef` (line 61) - declared but never used

#### **Duplicate Components to Remove:**
- Showcase page: Remove duplicate ProgressBar instances (lines 1917, 1935, 1939, 1943)
- Keep only the ProgressBar in JsonExplorer

### **Detailed Function Analysis Based on Implementation Plan**

#### **Functions That SHOULD Be Used (Currently Not Used)**

**1. `assignMappingsToFullPaths()` - CRITICAL MISSING**
- **Plan Reference**: Step 6 - "JsonResultMapper reads source-mappings.json and assigns mapping statuses to fullPaths"
- **Current Status**: ✅ Implemented but ❌ Never called
- **Should Be Used**: YES - This is the core function that processes mapping data and assigns statuses to fullPaths
- **Where It Should Be Called**: JsonExplorer when mapping data changes
- **Evidence**: Step 6 explicitly states "JsonResultMapper calls `assignMappingsToFullPaths` with mappings data"

**2. `determineBasePathStatuses()` - CRITICAL MISSING**
- **Plan Reference**: Step 5 - "Determine statuses for basePaths that will control accordion shadow colors"
- **Current Status**: ✅ Implemented but ❌ Never called
- **Should Be Used**: YES - This calculates accordion statuses based on child fullPaths
- **Where It Should Be Called**: JsonExplorer when fullPaths change
- **Evidence**: Step 5 states "JsonRenderer will pass the determined `basePath` status to `JsonAccordion`"

**3. `setPathStatus()` - CRITICAL MISSING**
- **Plan Reference**: Step 5 - "Function to determine basePath status based on children fullPaths"
- **Current Status**: ✅ Implemented but ❌ Never called
- **Should Be Used**: YES - This is called by `determineBasePathStatuses()`
- **Where It Should Be Called**: Inside `determineBasePathStatuses()`
- **Evidence**: Step 5 explicitly defines this function for status priority logic

**4. `countMappings()` - CRITICAL MISSING**
- **Plan Reference**: Step 7 - "Modify `countMappings` function to work with `basePaths`/`fullPaths` structure"
- **Current Status**: ✅ Implemented but ❌ Never called
- **Should Be Used**: YES - This calculates statistics for ProgressBar and accordions
- **Where It Should Be Called**: JsonExplorer for root level and each accordion
- **Evidence**: Step 7 states "For root level: Count all fullPaths, Update root basePath statistics, Pass to ProgressBar"

**5. `updateMappingCount()` - CRITICAL MISSING**
- **Plan Reference**: Step 6 - "Update `mappingCount` based on `mappings` array length"
- **Current Status**: ✅ Implemented but ❌ Never called
- **Should Be Used**: YES - This updates mapping counts when mappings are assigned
- **Where It Should Be Called**: After `assignMappingsToFullPaths()`
- **Evidence**: Step 6 states "Called when mappings are assigned"

#### **Functions That Are CORRECTLY Used**

**1. `generatePathData()` - ✅ CORRECTLY USED**
- **Plan Reference**: Step 2 - "Renderer Creates All Paths and Passes to Explorer"
- **Current Status**: ✅ Implemented and ✅ Called by JsonRenderer
- **Evidence**: JsonRenderer calls this in `useMemo(() => generatePathData(data), [data])`

**2. `getBasePaths()` - ✅ CORRECTLY USED**
- **Plan Reference**: Step 2 - "Extract base path generation logic from getStructurePaths"
- **Current Status**: ✅ Implemented and ✅ Called by `generatePathData()`
- **Evidence**: `generatePathData()` calls `getBasePaths(data)`

**3. `getFullPaths()` - ✅ CORRECTLY USED**
- **Plan Reference**: Step 2 - "Extract full path generation logic from getApplicablePaths"
- **Current Status**: ✅ Implemented and ✅ Called by `generatePathData()`
- **Evidence**: `generatePathData()` calls `getFullPaths(data)`

**4. `generateParentPaths()` - ✅ CORRECTLY USED**
- **Plan Reference**: Step 1 - "Rename generateParentPrefixes to generateParentPaths"
- **Current Status**: ✅ Implemented and ✅ Called by PathSearch and JsonExplorer
- **Evidence**: PathSearch calls this in `handleResultSelect`

**5. `renderArray()`, `renderObject()`, `renderPrimitiveNode()`, `renderJsonNode()` - ✅ CORRECTLY USED**
- **Plan Reference**: Step 9 - "Rename Rendering Functions for Clarity"
- **Current Status**: ✅ Implemented and ✅ Called by JsonRenderer
- **Evidence**: JsonRenderer imports and calls these functions

#### **Functions That Are CORRECTLY NOT USED (Safe to Remove)**

**1. `getStructurePaths()` - ✅ CORRECTLY REPLACED**
- **Plan Reference**: Step 2 - "Replace getStructurePaths with getBasePaths"
- **Current Status**: ✅ Replaced by `getBasePaths()`
- **Evidence**: Step 2 states "Remove `getJsonPaths` and `getApplicablePaths` logic (moved to new functions)"

**2. `isMappingApplicableToPath()` - ✅ CORRECTLY REPLACED**
- **Plan Reference**: Step 6 - "isPathAtLevel() → validateMappingforFullPath()"
- **Current Status**: ✅ Replaced by `validateMappingforFullPath()`
- **Evidence**: Step 6 states "Use `validateMappingforFullPath()` to check if mapping applies"

**3. `isMappingInJson()` - ✅ CORRECTLY NOT NEEDED**
- **Plan Reference**: Not mentioned in new architecture
- **Current Status**: ✅ Not needed with new structure
- **Evidence**: New architecture uses `fullPaths` structure instead of JSON traversal

**4. `countMappingOptions()` - ✅ CORRECTLY REPLACED**
- **Plan Reference**: Step 7 - "countMappingOptions() → countMappings()"
- **Current Status**: ✅ Replaced by `countMappings()`
- **Evidence**: Step 7 states "Modify `countMappings` function to work with new data structure"

**5. `getAccordionLevelStatus()` - ✅ CORRECTLY REPLACED**
- **Plan Reference**: Step 5 - "getAccordionLevelStatus() → setPathStatus()"
- **Current Status**: ✅ Replaced by `setPathStatus()`
- **Evidence**: Step 5 states "Function to determine basePath status based on children fullPaths"

#### **Functions That Need Verification (May Be Used Later)**

**1. `getCurrentMapping()` - NEEDS VERIFICATION**
- **Plan Reference**: Not explicitly mentioned in new architecture
- **Current Status**: ✅ Implemented but ❓ Not used
- **Should Be Used**: MAYBE - Could be useful for showing current mapping in UI
- **Evidence**: No clear reference in implementation plan

**2. `getApplicableMappings()` - NEEDS VERIFICATION**
- **Plan Reference**: Not explicitly mentioned in new architecture
- **Current Status**: ✅ Implemented but ❓ Not used
- **Should Be Used**: MAYBE - Could be useful for mapping dropdowns
- **Evidence**: No clear reference in implementation plan

**3. `handleMappingChange()` - NEEDS VERIFICATION**
- **Plan Reference**: Not explicitly mentioned in new architecture
- **Current Status**: ✅ Implemented but ❓ Not used
- **Should Be Used**: MAYBE - Could be useful for mapping changes
- **Evidence**: No clear reference in implementation plan

**4. `generateMappingOptions()` - NEEDS VERIFICATION**
- **Plan Reference**: Not explicitly mentioned in new architecture
- **Current Status**: ✅ Implemented but ❓ Not used
- **Should Be Used**: MAYBE - Could be useful for mapping dropdowns
- **Evidence**: No clear reference in implementation plan

**5. `generateNodeInfo()` - ✅ CORRECTLY REPLACED**
- **Plan Reference**: Step 8 - "Replace getJsonPaths() with direct access to basePaths keys"
- **Current Status**: ✅ Replaced by new path generation
- **Evidence**: Step 8 states "Replace `getJsonPaths()` with direct access to `basePaths` keys"

#### **Functions That Are PARTIALLY USED (Need Integration)**

**1. `validateMappingforFullPath()` - PARTIALLY USED**
- **Plan Reference**: Step 6 - "Use validateMappingforFullPath() to check if mapping applies"
- **Current Status**: ✅ Implemented but ❌ Only called by `assignMappingsToFullPaths()` which is never called
- **Should Be Used**: YES - This is the core wildcard matching function
- **Evidence**: Step 6 states "Use `validateMappingforFullPath()` to check if mapping applies (handles wildcards)"

**2. `getFullPathState()` - PARTIALLY USED**
- **Plan Reference**: Step 6 - "Use getFullPathState() to determine status"
- **Current Status**: ✅ Implemented but ❌ Only called by `assignMappingsToFullPaths()` which is never called
- **Should Be Used**: YES - This determines fullPath status from mappings
- **Evidence**: Step 6 states "Use `getFullPathState()` to determine status"

**3. `getStructurePathsFromBasePaths()` - PARTIALLY USED**
- **Plan Reference**: Step 4 - "Get structure paths from basePaths with [0] logic"
- **Current Status**: ✅ Implemented and ✅ Called by JsonExplorer but ❌ Doesn't work because basePaths is empty
- **Should Be Used**: YES - This is used for "Show Structure" expansion
- **Evidence**: Step 4 states "JsonExplorer integration with a dropdown"

**4. `getAllPaths()`, `getIssuedPaths()`, `getMappedPaths()`, `getRecommendedPaths()` - PARTIALLY USED**
- **Plan Reference**: Step 4 - "New expansion functions"
- **Current Status**: ✅ Implemented and ✅ Called by JsonExplorer but ❌ Don't work because basePaths is empty
- **Should Be Used**: YES - These are used for different expansion modes
- **Evidence**: Step 4 states "JsonExplorer integration with a dropdown"

**5. `expandPaths()` - PARTIALLY USED**
- **Plan Reference**: Step 4 - "Main expansion function that handles parent paths"
- **Current Status**: ✅ Implemented and ✅ Called by JsonExplorer but ❌ Doesn't work because basePaths is empty
- **Should Be Used**: YES - This is the main expansion function
- **Evidence**: Step 4 states "Main expansion function that handles parent paths"

### **Integration Steps Required**

#### **Step 10.1: Fix JsonExplorer Data Management**
1. **Receive pathData from JsonRenderer**: Update `handlePathsGenerated` to receive `basePaths` and `fullPaths`
2. **Populate basePaths state**: Initialize `basePaths` with received data
3. **Add mapping processing**: Call `assignMappingsToFullPaths()` when mapping data changes
4. **Add status calculation**: Call `determineBasePathStatuses()` when `fullPaths` change
5. **Add statistics calculation**: Call `countMappings()` for root and accordion statistics

#### **Step 10.2: Fix JsonRenderer Props**
1. **Remove old props**: Remove `nodeRenderingInfo` and `accordionStats` props
2. **Add new props**: Add `basePaths` and `fullPaths` props
3. **Update rendering logic**: Use new props instead of old ones
4. **Pass data to JsonExplorer**: Send `pathData` back to JsonExplorer

#### **Step 10.3: Fix JsonAccordion Integration**
1. **Update props**: Receive `basePathData` instead of individual props
2. **Use status for shadows**: Apply status-based shadow colors
3. **Display statistics**: Show calculated statistics in accordion headers

#### **Step 10.4: Fix ProgressBar Integration**
1. **Calculate root statistics**: Call `countMappings("", fullPaths)` for root level
2. **Pass to ProgressBar**: Update ProgressBar with calculated statistics

#### **Step 10.5: Fix Expansion Controls**
1. **Populate basePaths**: Ensure `basePaths` is populated with actual data
2. **Update expansion logic**: Use populated `basePaths` for expansion functions
3. **Pass to JsonRenderer**: Send expansion state to JsonRenderer

#### **Step 10.6: Update JsonResultMapper**
1. **Remove old imports**: Remove `generateNodeInfo` and old `countMappings` imports
2. **Add new imports**: Import new functions (`assignMappingsToFullPaths`, `determineBasePathStatuses`, `countMappings`)
3. **Update function calls**: Replace `generateNodeInfo` with new path generation
4. **Update countMappings calls**: Use new signature `countMappings(basePath, fullPaths)`
5. **Remove ProgressBar**: JsonResultMapper should not render ProgressBar (moved to JsonExplorer)

#### **Step 10.7: Remove Unused Code**
1. **Comment out unused functions**: Wrap obsolete functions in comments before deleting
2. **Remove unused imports**: Clean up imports in all components
3. **Remove unused refs**: Remove unused refs in JsonExplorer
4. **Remove duplicate components**: Remove duplicate ProgressBar instances in showcase
5. **Remove unused exports**: Clean up exports in utility files
6. **Delete commented functions**: After verification, delete the commented-out functions

**Safe Cleanup Process:**
```typescript
// STEP 1: Comment out unused functions (don't delete yet)
/*
export function getStructurePaths(data: any): Set<string> {
  // ... function body
}

export function isMappingApplicableToPath(mappingPath: string[], accordionPath: string[]): boolean {
  // ... function body
}

export function isMappingInJson(mappingPath: string[], jsonData: any): boolean {
  // ... function body
}

export function countMappingOptions(accordionPath: string, jsonData: any): number {
  // ... function body
}

export function getAccordionLevelStatus(accordionPath: string, mappings: Mappings, jsonData: any): JsonNodeStatus {
  // ... function body
}
*/

// STEP 2: Remove unused imports from components
// JsonRenderer: Remove getStructurePaths import
// JsonExplorer: Remove expansion function imports (until data flow is fixed)

// STEP 3: Test that everything still works
// Run the application and verify no errors

// STEP 4: Delete commented functions after verification
// Remove the entire commented blocks
```

**Functions to Comment Out (Safe to Remove):**
- `getStructurePaths()` - Replaced by `getBasePaths()`
- `isMappingApplicableToPath()` - Replaced by `validateMappingforFullPath()`
- `isMappingInJson()` - No longer needed with new structure
- `countMappingOptions()` - Replaced by `countMappings()`
- `getAccordionLevelStatus()` - Replaced by `setPathStatus()`
- `generateNodeInfo()` - Replaced by new path generation

**Functions to Keep (Will Be Useful):**
- `getCurrentMapping()` - Useful for UI (showing current mapping)
- `getApplicableMappings()` - Useful for mapping dropdowns
- `handleMappingChange()` - Useful for mapping changes
- `generateMappingOptions()` - Useful for mapping dropdowns

**Additional Cleanup Items:**
- **JsonRenderer**: Remove `getStructurePaths` import (line 9)
- **JsonExplorer**: Remove `Divider` import (line 6)
- **JsonExplorer**: Remove `selectedNodeRef` (line 61)
- **Showcase page**: Remove duplicate ProgressBar instances (lines 1917, 1935, 1939, 1943)
- **JsonResultMapper**: Still uses old `generateNodeInfo` and `countMappings` - needs updating

### **Expected Outcome After Cleanup**

**Working Features:**
- ✅ Accordion statuses based on child node priorities
- ✅ ProgressBar displaying calculated root statistics
- ✅ Expansion controls working with actual data
- ✅ Accordion statistics showing calculated values
- ✅ Proper data flow between all components
- ✅ No infinite loops or circular dependencies
- ✅ Clean codebase with no unused functions

**Data Flow:**
```
JsonExplorer (Central Hub)
├── Receives: Raw JSON data + Mappings
├── Calls: generatePathData() → basePaths + fullPaths
├── Calls: assignMappingsToFullPaths() → Updated fullPaths with mappings
├── Calls: determineBasePathStatuses() → Updated basePaths with statuses
├── Calls: countMappings() → Statistics for ProgressBar and accordions
├── Manages: basePaths + fullPaths state
├── Passes: basePaths + fullPaths to JsonRenderer
├── Passes: root statistics to ProgressBar
└── Controls: ProgressBar, Search, Expansion, UI state
```

**Component Responsibilities:**
- **JsonExplorer**: Central orchestrator managing all data and UI
- **JsonRenderer**: Pure rendering using `basePaths` and `fullPaths`
- **JsonAccordion**: Displays accordion with status-based shadows and statistics
- **ProgressBar**: Shows calculated root-level statistics
- **PathSearch**: Works with `fullPaths` for search functionality

## Step 11: Corrected Data Flow Implementation

**Goal**: Implement the corrected end-to-end data flow with proper validation and function sequencing.

### **Complete End-to-End Flow with Triggers**

#### **Step 1: Components Page Mount**
- **Trigger**: Page loads
- **Action**: Imports source-response.json and source-mappings.json
- **Result**: Raw data and mappings available immediately

#### **Step 2: JsonExplorer Mount**
- **Trigger**: JsonExplorer component mounts
- **Action**: Receives raw data from components page
- **Function Called**: generatePathData(rawData)
- **Result**: Creates initial basePaths and fullPaths structures
- **State Update**: setBasePaths() and setFullPaths()

#### **Step 3: JsonResultMapper Mount**
- **Trigger**: JsonResultMapper component mounts
- **Action**: Receives basePaths, fullPaths from JsonExplorer and mappings from components page
- **Validation**: Checks if it has all necessary data (basePaths, fullPaths, mappings)
- **Function Sequence** (only when validation passes):
  1. assignMappingsToFullPaths(mappings, fullPaths) → Updates fullPaths with mapping statuses
  2. updateMappingCount(updatedFullPaths) → Updates mapping counts in fullPaths
  3. determineBasePathStatuses(basePaths, updatedFullPaths) → Updates basePaths with accordion statuses
  4. countMappings("", updatedBasePaths) → Calculates root statistics
- **Callback**: onProcessedData(processedData) to JsonExplorer

#### **Step 4: JsonExplorer State Update**
- **Trigger**: onProcessedData callback from JsonResultMapper
- **Action**: Receives processed data
- **State Update**: setBasePaths(), setFullPaths(), setProgressStats()
- **Result**: JsonExplorer state updated with final processed data

#### **Step 5: JsonRenderer Mount**
- **Trigger**: JsonExplorer state update
- **Action**: Receives final basePaths and fullPaths from JsonExplorer
- **Result**: Renders UI with correct mapping statuses and statistics

### **Key Points:**
- assignMappingsToFullPaths is triggered by JsonResultMapper once it has all necessary data
- Processing happens in JsonResultMapper with proper validation
- Data flows: Components → JsonExplorer → JsonResultMapper → JsonExplorer → JsonRenderer
- No circular dependencies, each step triggers the next

### **Function Analysis for Correct Processing Order**

#### **assignMappingsToFullPaths(mappings, fullPaths)**
- **Input**: Raw mappings and fullPaths
- **Action**: Applies mapping statuses to individual nodes in fullPaths
- **Output**: Updated fullPaths with mapping statuses
- **Trigger**: When JsonResultMapper receives both mappings and fullPaths

#### **updateMappingCount(updatedFullPaths)**
- **Input**: fullPaths with mapping statuses
- **Action**: Counts and updates mapping statistics within fullPaths
- **Output**: fullPaths with updated mapping counts
- **Trigger**: After assignMappingsToFullPaths completes

#### **determineBasePathStatuses(basePaths, updatedFullPaths)**
- **Input**: Original basePaths and updated fullPaths with mapping statuses
- **Action**: Calculates accordion-level statuses based on child node statuses
- **Output**: Updated basePaths with accordion statuses
- **Trigger**: After updateMappingCount completes

#### **countMappings("", updatedBasePaths)**
- **Input**: Updated basePaths with accordion statuses
- **Action**: Calculates root-level statistics for progress bar
- **Output**: Root statistics object
- **Trigger**: After determineBasePathStatuses completes

### **Corrected Step 3: JsonResultMapper Processing**

**Trigger**: JsonResultMapper component mounts
**Action**: Receives basePaths, fullPaths from JsonExplorer and mappings from components page

**Function Sequence:**
1. **assignMappingsToFullPaths(mappings, fullPaths)** → Updates fullPaths with mapping statuses
2. **updateMappingCount(updatedFullPaths)** → Updates mapping counts in fullPaths  
3. **determineBasePathStatuses(basePaths, updatedFullPaths)** → Updates basePaths with accordion statuses
4. **countMappings("", updatedBasePaths)** → Calculates root statistics

**Result**: Complete processed data with mapping statuses and statistics
**Callback**: onProcessedData(processedData) to JsonExplorer

This sequence ensures each function receives the correct input from the previous step and produces the right output for the next step.

### **Step 11: Comprehensive Implementation Guide**

**Goal**: Provide a complete, step-by-step implementation guide to fix the broken data flow and implement the corrected architecture.

#### **11.1: Current State Analysis**

**Critical Issues Identified:**

**JsonExplorer Problems:**
- **Lines 96-120**: `initialPathData` useMemo/useEffect causing infinite loops
- **Lines 84-94**: `handlePathDataGenerated` redundant in corrected flow
- **Lines 200-207**: Unnecessary `searchResults` conversion
- **Lines 365-372**: JsonResultMapper placed after JsonRenderer (wrong order)
- **Missing**: Immediate path generation on mount

**JsonRenderer Problems:**
- **Lines 112-128**: Conditional path generation causing conflicts
- **Lines 33-39**: Unused `NodeRenderingInfo` interface
- **Missing**: Always use provided basePaths/fullPaths from parent

**JsonResultMapper Problems:**
- **Lines 107-113**: Syntax error (missing opening brace)
- **Lines 139-166**: Unused mapping logic cluttering component
- **Missing**: Focus on pure data processing

#### **11.2: Implementation Order & Strategy**

**Phase 1: Fix Critical Syntax Errors**
1. Fix JsonResultMapper syntax error (prevents compilation)
2. Test that app runs without errors

**Phase 2: Simplify JsonRenderer**
1. Remove conditional path generation logic
2. Remove unused interfaces and callbacks
3. Make it always use provided data

**Phase 3: Fix JsonExplorer Data Flow**
1. Remove broken initialPathData logic
2. Add immediate path generation on mount
3. Reorder JsonResultMapper before JsonRenderer
4. Remove redundant callbacks

**Phase 4: Clean Up JsonResultMapper**
1. Remove unused mapping logic
2. Focus on pure data processing
3. Ensure proper validation

**Phase 5: Integration Testing**
1. Test complete data flow
2. Verify mapping statistics work
3. Confirm no infinite loops

#### **11.3: Detailed Implementation Steps**

##### **Step 11.3.1: Fix JsonResultMapper Syntax Error**

**File**: `components/json/response/json-result-mapper.tsx`
**Lines**: 107-113

**Current Code (BROKEN):**
```typescript
// If no data yet, return empty data (don't process mappings)
if (!basePaths || !fullPaths || Object.keys(basePaths).length === 0 || Object.keys(fullPaths).length === 0) 
  return {
    basePaths: basePaths || {},
    fullPaths: fullPaths || {},
    rootStats: { mapped: 0, recommendations: 0, issues: 0, options: 0 }
  }
}
```

**Fixed Code:**
```typescript
// If no data yet, return empty data (don't process mappings)
if (!basePaths || !fullPaths || Object.keys(basePaths).length === 0 || Object.keys(fullPaths).length === 0) {
  return {
    basePaths: basePaths || {},
    fullPaths: fullPaths || {},
    rootStats: { mapped: 0, recommendations: 0, issues: 0, options: 0 }
  }
}
```

**Validation**: App should compile without syntax errors.

##### **Step 11.3.2: Simplify JsonRenderer**

**File**: `components/json/interface/json-renderer.tsx`

**Remove Lines 33-39 (NodeRenderingInfo interface):**
```typescript
// REMOVE THIS ENTIRE INTERFACE
interface NodeRenderingInfo {
  status: JsonNodeStatus
  mappingOptions: Array<{ id: string; label: string; value: string; description?: string; tags?: string[]; status: string }>
  currentMapping?: { source: string; target: string; type: string }
  mappingCount?: number
  accordionStatus?: JsonNodeStatus
}
```

**Remove Lines 112-128 (conditional path generation):**
```typescript
// REMOVE THIS ENTIRE useMemo BLOCK
const pathData = useMemo(() => {
  if (Object.keys(basePaths).length > 0 && Object.keys(fullPaths).length > 0) {
    return { basePaths, fullPaths }
  }
  return generatePathData(data)
}, [data, basePaths, fullPaths])

// REMOVE THIS ENTIRE useEffect BLOCK
useEffect(() => {
  if (Object.keys(basePaths).length === 0 && Object.keys(fullPaths).length === 0) {
    onPathDataGenerated?.(pathData)
  }
}, [pathData, basePaths, fullPaths, onPathDataGenerated])
```

**Replace with simple data usage:**
```typescript
// Use provided data directly
const pathData = { basePaths, fullPaths }
```

**Remove onPathDataGenerated prop from interface:**
```typescript
interface JsonRendererProps {
  data: any
  basePaths?: Record<string, BasePathData>
  fullPaths?: Record<string, FullPathData>
  options?: JsonRendererOptions
  onPathSelect?: (path: string, parentPaths?: string[]) => void
  className?: string
  expandedPaths?: Set<string>
  selectedPath?: string
  onNodeExpand?: (path: string) => void
  onNodeClick?: (path: string) => void
  // REMOVE: onPathDataGenerated?: (pathData: {basePaths: Record<string, BasePathData>, fullPaths: Record<string, FullPathData>}) => void
}
```

**Validation**: JsonRenderer should only use provided basePaths/fullPaths.

##### **Step 11.3.3: Fix JsonExplorer Data Flow**

**File**: `components/json/interface/json-explorer.tsx`

**Remove Lines 84-94 (handlePathDataGenerated):**
```typescript
// REMOVE THIS ENTIRE FUNCTION
const handlePathDataGenerated = useCallback((pathData: {basePaths: Record<string, BasePathData>, fullPaths: Record<string, FullPathData>}) => {
  console.log('JsonExplorer: Received path data:', pathData)
  setBasePaths(pathData.basePaths)
  setFullPaths(pathData.fullPaths)
  const basePathKeys = Object.keys(pathData.basePaths)
  setExpandedPaths(new Set(basePathKeys))
}, [])
```

**Remove Lines 96-120 (initialPathData logic):**
```typescript
// REMOVE THIS ENTIRE useMemo AND useEffect
const initialPathData = useMemo(() => {
  if (data && Object.keys(basePaths).length === 0 && Object.keys(fullPaths).length === 0) {
    console.log('JsonExplorer: Generating initial paths from raw data')
    const pathData = generatePathData(data)
    console.log('JsonExplorer: Generated path data', { 
      basePathsCount: Object.keys(pathData.basePaths).length, 
      fullPathsCount: Object.keys(pathData.fullPaths).length 
    })
    return pathData
  }
  return { basePaths: {}, fullPaths: {} }
}, [data, basePaths, fullPaths])

useEffect(() => {
  if (Object.keys(initialPathData.basePaths).length > 0 && Object.keys(initialPathData.fullPaths).length > 0) {
    setBasePaths(initialPathData.basePaths)
    setFullPaths(initialPathData.fullPaths)
    const basePathKeys = Object.keys(initialPathData.basePaths)
    setExpandedPaths(new Set(basePathKeys))
  }
}, [initialPathData])
```

**Add immediate path generation on mount:**
```typescript
// Add this useEffect after the state declarations
useEffect(() => {
  if (data && Object.keys(basePaths).length === 0 && Object.keys(fullPaths).length === 0) {
    console.log('JsonExplorer: Generating initial paths from raw data')
    const pathData = generatePathData(data)
    console.log('JsonExplorer: Generated path data', { 
      basePathsCount: Object.keys(pathData.basePaths).length, 
      fullPathsCount: Object.keys(pathData.fullPaths).length 
    })
    
    setBasePaths(pathData.basePaths)
    setFullPaths(pathData.fullPaths)
    
    // Initialize expansion state with base paths
    const basePathKeys = Object.keys(pathData.basePaths)
    setExpandedPaths(new Set(basePathKeys))
  }
}, [data]) // Only depend on data, not basePaths/fullPaths
```

**Remove Lines 200-207 (searchResults conversion):**
```typescript
// REMOVE THIS ENTIRE useEffect
useEffect(() => {
  const searchResults = Object.entries(fullPaths).map(([path, data]) => ({
    path,
    value: data.value,
    status: data.status
  }))
  setSearchResults(searchResults)
}, [fullPaths])
```

**Update PathSearch to work directly with fullPaths:**
```typescript
// Replace the searchResults prop with fullPaths
<PathSearch
  fullPaths={fullPaths}  // Changed from searchResults={searchResults}
  onPathSelect={handlePathSelect}
  placeholder="Search"
  selectedPath={selectedPath}
/>
```

**Move JsonResultMapper above JsonRenderer (Lines 365-372):**
```typescript
{/* JSON Content using JsonRenderer - Scrollable */}
<div className="flex-1 overflow-auto p-4" ref={containerRef}>
  {/* JsonResultMapper processes mappings and updates data - MOVED ABOVE */}
  <JsonResultMapper
    data={data}
    mappings={mappings}
    basePaths={basePaths}
    fullPaths={fullPaths}
    onProcessedData={handleProcessedData}
  />
  
  <JsonRenderer
    data={data}
    basePaths={basePaths}
    fullPaths={fullPaths}
    options={{
      showMetadata: options.showMetadata || false,
      compactMode: options.compactMode || false
    }}
    expandedPaths={expandedPaths}
    selectedPath={selectedPath}
    onNodeExpand={handleNodeExpand}
    onNodeClick={handleNodeClick}
    // REMOVE: onPathDataGenerated={handlePathDataGenerated}
  />
</div>
```

**Validation**: JsonExplorer should generate paths immediately on mount and pass them to both JsonResultMapper and JsonRenderer.

##### **Step 11.3.4: Clean Up JsonResultMapper**

**File**: `components/json/response/json-result-mapper.tsx`

**Remove Lines 139-166 (unused mapping logic):**
```typescript
// REMOVE THIS ENTIRE useMemo BLOCK
const mappingOptions = useMemo(() => {
  if (!mappings.sources) return []
  return mappings.sources.flatMap(source => [
    ...source.object_source_mappings.map(mapping => ({
      id: mapping.id,
      label: mapping.description,
      value: mapping.id,
      description: mapping.description,
      tags: mapping.key_mapping
    })),
    ...source.datapoint_source_mappings.map(mapping => ({
      id: mapping.id,
      label: mapping.description,
      value: mapping.id,
      description: mapping.description,
      tags: mapping.key_mapping
    }))
  ])
}, [mappings.sources])

// REMOVE THIS ENTIRE FUNCTION
const handleMappingChange = useCallback((path: string, mappingId: string) => {
  if (onMappingChange) {
    const mapping = mappingOptions.find(m => m.id === mappingId)
    onMappingChange(path, mapping)
  }
}, [onMappingChange, mappingOptions])
```

**Remove onMappingChange from interface:**
```typescript
interface JsonResultMapperProps {
  data: any
  mappings?: Mappings
  basePaths: Record<string, BasePathData>
  fullPaths: Record<string, FullPathData>
  onProcessedData?: (processedData: {
    basePaths: Record<string, BasePathData>
    fullPaths: Record<string, FullPathData>
    rootStats: {mapped: number, recommendations: number, issues: number, options: number}
  }) => void
  // REMOVE: onMappingChange?: (path: string, mapping: any) => void
}
```

**Validation**: JsonResultMapper should focus only on data processing.

#### **11.4: Integration Testing Checklist**

**Phase 1 Validation:**
- [ ] App compiles without syntax errors
- [ ] No console errors on page load
- [ ] Mappers tab loads by default

**Phase 2 Validation:**
- [ ] JsonRenderer uses provided data only
- [ ] No conditional path generation
- [ ] Clean component interface

**Phase 3 Validation:**
- [ ] JsonExplorer generates paths on mount
- [ ] JsonResultMapper processes before JsonRenderer
- [ ] No infinite loops in useEffect
- [ ] PathSearch works with fullPaths directly

**Phase 4 Validation:**
- [ ] JsonResultMapper focuses on data processing
- [ ] No unused mapping logic
- [ ] Clean component interface

**Phase 5 Final Validation:**
- [ ] Complete data flow works end-to-end
- [ ] Mapping statistics display correctly
- [ ] Accordion statuses show proper colors
- [ ] ProgressBar shows calculated statistics
- [ ] No "Maximum update depth exceeded" errors
- [ ] Search functionality works
- [ ] Expansion controls work

#### **11.5: Troubleshooting Guide**

**If "Maximum update depth exceeded" error occurs:**
1. Check for circular dependencies in useEffect
2. Ensure JsonResultMapper doesn't trigger JsonExplorer updates that trigger JsonResultMapper
3. Verify onProcessedData callback doesn't cause infinite loops

**If mapping statistics show 0:**
1. Verify JsonResultMapper receives basePaths and fullPaths
2. Check that assignMappingsToFullPaths is called
3. Ensure onProcessedData callback updates JsonExplorer state

**If accordion statuses are "default":**
1. Verify determineBasePathStatuses is called
2. Check that fullPaths have proper status values
3. Ensure basePaths are updated with processed data

**If search doesn't work:**
1. Verify PathSearch receives fullPaths prop
2. Check that fullPaths contain proper data structure
3. Ensure searchResults conversion is removed

#### **11.6: Expected Outcome**

After completing all steps:

**Working Features:**
- ✅ JsonExplorer generates paths immediately on mount
- ✅ JsonResultMapper processes mappings and updates data
- ✅ JsonRenderer displays processed data with correct statuses
- ✅ Mapping statistics show calculated values (not 0)
- ✅ Accordion statuses show proper colors based on child nodes
- ✅ ProgressBar displays root-level statistics
- ✅ Search functionality works with fullPaths
- ✅ Expansion controls work with populated basePaths
- ✅ No infinite loops or circular dependencies
- ✅ Clean, maintainable code structure

**Data Flow:**
```
Components Page → JsonExplorer → generatePathData() → basePaths/fullPaths
JsonExplorer → JsonResultMapper → assignMappingsToFullPaths() → updated fullPaths
JsonResultMapper → determineBasePathStatuses() → updated basePaths
JsonResultMapper → countMappings() → statistics
JsonResultMapper → JsonExplorer → onProcessedData() → final state
JsonExplorer → JsonRenderer → display with correct statuses
```

This comprehensive guide provides everything needed to implement the corrected data flow successfully.


## Step 12: Remove Suffixes & Make JsonExplorer Self-Contained

**Goal**: Transform the JSON system from a **dual-path suffix approach** (`:key`/`:value`) to a **single-path type-based approach** where JsonExplorer works independently without JsonResultMapper, using only fullPaths with embedded type information.

### **Core Changes**

#### **1. Data Structure Transformation**
- **From**: `Record<string, FullPathData>` with suffixed keys (`"path:key"`, `"path:value"`)
- **To**: `FullPathData[]` with embedded type field (`{path: "path", type: "key"}`)

#### **2. JsonExplorer Independence**
- **Remove**: JsonResultMapper dependency
- **Add**: Direct mapping processing within JsonExplorer
- **Result**: Single component handling all JSON processing

### **Functions to Modify (No New Functions Except One)**

#### **1. Data Structure Generation Functions**

**`generatePathData(data: any)` - MODIFY**
- Change return type from `Record<string, FullPathData>` to `FullPathData[]`
- Remove suffix generation in `getFullPaths()` call
- Create array of objects instead of object with suffixed keys

**`getFullPaths(data: any)` - MODIFY**
- Remove all `${path}:key` and `${path}:value` suffix generation
- Create separate entries with same path but different type field
- Return array instead of object

**`getBasePaths(data: any)` - MODIFY**
- No changes needed (already returns clean paths)

#### **2. Mapping Processing Functions**

**`assignMappingsToFullPaths(mappings, fullPaths)` - MODIFY**
- Update to work with array format instead of object format
- Use `find()` to locate entries by path and type
- Update array entries in place

**`updateMappingCount(fullPaths)` - MODIFY**
- Update to work with array format
- Update mapping counts in array entries

**`determineBasePathStatuses(basePaths, fullPaths)` - MODIFY**
- Update to work with array format for fullPaths
- Use `filter()` to find children by path prefix

**`countMappings(basePath, fullPaths)` - MODIFY**
- Update to work with array format
- Use `filter()` to count by status

#### **3. New Function to Create**

**`updateBasePathStatistics(basePaths, fullPaths)` - CREATE**
- Takes basePaths and fullPaths (array format)
- Updates the statistics field for each basePath based on its children's fullPaths
- Returns the updated basePaths with correct statistics
- Eliminates the need for separate mappedBasePaths/mappedFullPaths

#### **4. Rendering Functions**

**`renderPrimitiveNode()` - MODIFY**
- Update to work with array format for fullPaths
- Use `find()` to locate key/value entries

**`renderArray()` - MODIFY**
- Update to work with array format
- Use `filter()` to find children entries

**`renderObject()` - MODIFY**
- Update to work with array format
- Use `filter()` to find children entries

### **JsonExplorer Flow Changes**

#### **New JsonExplorer Flow**
1. **Generate paths**: `generatePathData()` → returns `{basePaths, fullPaths[]}`
2. **Assign mappings**: `assignMappingsToFullPaths(mappings, fullPaths[])` → updates fullPaths array
3. **Update mapping counts**: `updateMappingCount(fullPaths[])` → updates counts in array
4. **Update basePath statuses**: `determineBasePathStatuses(basePaths, fullPaths[])` → updates basePaths
5. **Update basePath statistics**: `updateBasePathStatistics(basePaths, fullPaths[])` → updates statistics
6. **Calculate root statistics**: `countMappings("", fullPaths[])` → for progress bar
7. **Use same data**: Pass updated basePaths and fullPaths[] to JsonRenderer and PathSearch

#### **Eliminate JsonResultMapper**
- **Remove**: JsonResultMapper component entirely
- **Move**: All mapping processing logic into JsonExplorer
- **Result**: Single component handling all JSON processing

#### **Remove mappedBasePaths/mappedFullPaths from JsonExplorer State**
- **Remove from state**: `mappedBasePaths` and `mappedFullPaths` state variables
- **Remove from component creation**: All references to `mappedBasePaths` and `mappedFullPaths` in JSX
- **Update JsonRenderer props**: Pass original `basePaths` and `fullPaths` instead of mapped versions
- **Update PathSearch props**: Pass original `fullPaths` instead of `mappedFullPaths`
- **Result**: JsonExplorer works directly with original data structures, updating them in place

### **Data Flow**

```
JsonExplorer (Self-Contained)
├── Receives: Raw JSON data + Mappings
├── Calls: generatePathData() → basePaths + fullPaths[]
├── Calls: assignMappingsToFullPaths() → Updated fullPaths[] with mappings
├── Calls: updateMappingCount() → Updated counts in fullPaths[]
├── Calls: determineBasePathStatuses() → Updated basePaths with statuses
├── Calls: updateBasePathStatistics() → Updated basePaths with statistics
├── Calls: countMappings() → Root statistics for ProgressBar
├── Manages: basePaths + fullPaths[] state
├── Passes: basePaths + fullPaths[] to JsonRenderer
├── Passes: root statistics to ProgressBar
└── Controls: ProgressBar, Search, UI state
```

### **Benefits**

1. **Simplified Architecture**: Single component handling all processing
2. **No Suffixes**: Clean paths without `:key`/`:value` suffixes
3. **Type-Based**: Uses embedded type field instead of path suffixes
4. **Self-Contained**: JsonExplorer works independently
5. **Eliminates Redundancy**: No separate JsonResultMapper needed
6. **Cleaner Data Flow**: Direct processing within JsonExplorer
7. **Easier Maintenance**: Single component to maintain and debug


## Step 13: Implement Refined State Management to Prevent Infinite Loops

**Goal**: Implement a robust state management strategy to prevent "Maximum update depth exceeded" errors by separating processing control states from core data states, ensuring predictable data flow without infinite re-render loops.

### **Core Problem Analysis**

**Current Issue**: The `useEffect` hook for `processMappings` has `[mappings, basePaths, fullPaths]` as dependencies. Since `processMappings` updates `basePaths` and `fullPaths` state, this creates an infinite loop: `processMappings` → `setBasePaths`/`setFullPaths` → dependencies change → `processMappings` runs again.

**Root Causes**:
1. **Circular Dependencies**: Processing function depends on data it modifies
2. **Data Structure Mismatch**: Treating array `fullPaths` as object with `Object.keys()`
3. **State Update Race Conditions**: Multiple state updates triggering each other
4. **Missing Processing Control**: No way to track processing state independently

### **New State Management Strategy**

#### **1. Processing Control States (Add These)**
```typescript
const [jsonProcessed, setJsonProcessed] = useState(false)
const [pathsGenerated, setPathsGenerated] = useState(false)
const [mappingsProcessed, setMappingsProcessed] = useState(false)
const [mappingStatisticsProcessed, setMappingStatisticsProcessed] = useState(false)
const [readyToRender, setReadyToRender] = useState(false)
```

#### **2. Core Data States (Keep These)**
```typescript
const [basePaths, setBasePaths] = useState<Record<string, BasePathData>>({})
const [fullPaths, setFullPaths] = useState<Array<FullPathData>>([])
```

### **Function Flow with Descriptive Names**

#### **1. processJson() - "Take raw JSON and create path structure"**
```typescript
const processJson = useCallback(() => {
  if (data && !jsonProcessed) {
    console.log('🔍 JsonExplorer: Processing JSON data...')
    const pathData = generatePathData(data)
    setBasePaths(pathData.basePaths)
    setFullPaths(pathData.fullPaths)
    setJsonProcessed(true)
    setPathsGenerated(true)
  }
}, [data, jsonProcessed])
```

#### **2. processMappings() - "Apply mappings to the current path data"**
```typescript
const processMappings = useCallback(() => {
  if (pathsGenerated && mappings.sources && fullPaths.length > 0) {
    if (mappingsProcessed) return // Skip if already processed
    
    setIsProcessingMappings(true)
    setProcessingError(null)
    
    try {
      console.log('🔍 JsonExplorer: Processing mappings...')
      
      const updatedFullPaths = assignMappingsToFullPaths(mappings, fullPaths)
      const finalFullPaths = updateMappingCount(updatedFullPaths)
      const updatedBasePaths = determineBasePathStatuses(basePaths, finalFullPaths)
      const finalBasePaths = updateBasePathStatistics(updatedBasePaths, finalFullPaths)
      
      setBasePaths(finalBasePaths)
      setFullPaths(finalFullPaths)
      setMappingsProcessed(true)
      setMappingStatisticsProcessed(true)
      setReadyToRender(true)
      
      console.log('🔍 JsonExplorer: Mapping processing completed')
    } catch (error) {
      setProcessingError(error.message)
    } finally {
      setIsProcessingMappings(false)
    }
  }
}, [pathsGenerated, mappings, mappingsProcessed])
```

#### **3. handleDataChange() - "Reset everything when new data comes in"**
```typescript
const handleDataChange = useCallback(() => {
  if (data) {
    setJsonProcessed(false)
    setPathsGenerated(false)
    setMappingsProcessed(false)
    setMappingStatisticsProcessed(false)
    setReadyToRender(false)
  }
}, [data])
```

### **useEffect Structure**
```typescript
// 1. Handle data changes
useEffect(() => handleDataChange(), [data])

// 2. Process JSON when needed
useEffect(() => processJson(), [data, jsonProcessed])

// 3. Process mappings when ready
useEffect(() => processMappings(), [mappings, pathsGenerated])

// 4. Render when ready
useEffect(() => {
  if (readyToRender) {
    console.log('🔍 JsonExplorer: Ready to render!')
  }
}, [readyToRender])
```

### **Comprehensive Codebase Analysis**

#### **Files Requiring Changes**

**1. JsonExplorer (`components/json/interface/json-explorer.tsx`)**
- **Add**: 5 new processing control states
- **Modify**: `processMappings` function to use new state management
- **Update**: `useEffect` hooks to use new dependency structure
- **Rename**: `initializeData` function to `processJson`
- **Remove**: Current infinite loop-causing `useEffect` dependencies

**2. JsonRenderer (`components/json/interface/json-renderer.tsx`)**
- **No Changes Required**: Already works with provided `basePaths` and `fullPaths`
- **Current State**: ✅ Compatible with new state management

**3. PathSearch (`components/others/path-search.tsx`)**
- **No Changes Required**: Already works with `Array<FullPathData>` format
- **Current State**: ✅ Compatible with new state management

**4. JsonAccordion (`components/accordions/json-accordion.tsx`)**
- **No Changes Required**: Already works with `BasePathData` and status information
- **Current State**: ✅ Compatible with new state management

**5. JsonNode (`components/json/node/json-node.tsx`)**
- **No Changes Required**: Already works with individual `FullPathData` entries
- **Current State**: ✅ Compatible with new state management

**6. ProgressBar (`components/others/progress-bar.tsx`)**
- **No Changes Required**: Already works with statistics object format
- **Current State**: ✅ Compatible with new state management

#### **Rendering Utilities (`utils/json/rendering-utils.tsx`)**
- **No Changes Required**: All functions already work with new data structures
- **Current State**: ✅ Compatible with new state management

#### **Mapping Utilities (`utils/json/mapping-utils.ts`)**
- **No Changes Required**: All functions already work with new data structures
- **Current State**: ✅ Compatible with new state management

### **Data Flow Benefits**

#### **Clear Progression**
```
jsonProcessed → pathsGenerated → mappingsProcessed → mappingStatisticsProcessed → readyToRender
```

#### **Easy Debugging**
- Can see exactly which step failed
- Each state has a clear purpose
- Processing control separated from data

#### **Controlled Rendering**
- Only render when `readyToRender` is true
- No premature rendering with incomplete data
- Predictable state transitions

#### **No Infinite Loops**
- Each step depends on the previous one being complete
- Processing states don't depend on data they modify
- Clear separation of concerns

### **Implementation Steps**

#### **Step 13.1: Add Processing Control States**
```typescript
// Add these 5 new states to JsonExplorer
const [jsonProcessed, setJsonProcessed] = useState(false)
const [pathsGenerated, setPathsGenerated] = useState(false)
const [mappingsProcessed, setMappingsProcessed] = useState(false)
const [mappingStatisticsProcessed, setMappingStatisticsProcessed] = useState(false)
const [readyToRender, setReadyToRender] = useState(false)
```

#### **Step 13.2: Create New Processing Functions**
```typescript
// Replace current processMappings with these three functions
const processJson = useCallback(() => { /* ... */ }, [data, jsonProcessed])
const processMappings = useCallback(() => { /* ... */ }, [pathsGenerated, mappings, mappingsProcessed])
const handleDataChange = useCallback(() => { /* ... */ }, [data])
```

#### **Step 13.3: Update useEffect Structure**
```typescript
// Replace current useEffect hooks with new structure
useEffect(() => handleDataChange(), [data])
useEffect(() => processJson(), [data, jsonProcessed])
useEffect(() => processMappings(), [mappings, pathsGenerated])
useEffect(() => { if (readyToRender) { /* render logic */ } }, [readyToRender])
```

#### **Step 13.4: Remove Infinite Loop Dependencies**
```typescript
// Remove this problematic useEffect
useEffect(() => {
  processMappings()
}, [mappings, basePaths, fullPaths]) // ❌ This causes infinite loops
```

### **Expected Outcome**

**Working Features**:
- ✅ No "Maximum update depth exceeded" errors
- ✅ Predictable data flow progression
- ✅ Clear debugging with state names
- ✅ Controlled rendering timing
- ✅ All existing functionality preserved

**Data Flow**:
```
User loads JSON → handleDataChange() → processJson() → pathsGenerated
User loads mappings → processMappings() → mappingsProcessed → readyToRender
JsonExplorer renders → All components work with final data
```

**Benefits**:
- **No Infinite Loops**: Processing control separated from data
- **Clear State Management**: Each state has a specific purpose
- **Easy Debugging**: Can see exactly which step failed
- **Predictable Flow**: Data → processJson → processMappings → render
- **Maintainable**: Clear separation of concerns

the ## Step 14: Reorganize Architecture - Move Data Processing to JsonResultMapper

**Goal**: Reorganize the JSON explorer architecture to separate concerns properly. Move all data processing, mapping logic, and state management to JsonResultMapper, while making JsonExplorer a pure UI component library that works with JsonRenderer, accordions, and nodes.

### **Current Architecture Problems**

The JSON explorer currently has a **mixed architecture** where data processing, mapping logic, and UI components are intertwined:

1. **JsonExplorer** (545 lines) - Main orchestrator that handles:
   - Data processing (JSON parsing, path generation)
   - Mapping processing (applying mappings to paths)
   - UI state management (expansion, selection, search)
   - Rendering coordination

2. **JsonRenderer** (183 lines) - Pure rendering component that:
   - Takes processed data and renders the JSON tree
   - Uses utility functions for rendering logic
   - Handles controlled expansion and selection

3. **Utility Files**:
   - **rendering-utils.tsx** (930 lines) - Contains both data processing AND rendering logic
   - **mapping-utils.ts** (963 lines) - Contains mapping logic and data processing

4. **UI Components**:
   - **JsonAccordion** (422 lines) - Complex accordion with mapping functionality
   - **JsonNode** (183 lines) - Individual nodes with mapping capabilities

### **Problems Identified**

1. **Mixed Concerns**: The `JsonExplorer` component handles both data processing and UI orchestration
2. **Utility Bloat**: `rendering-utils.tsx` contains both data processing and rendering functions
3. **Complex State Management**: Multiple processing states and complex useEffect chains
4. **Tight Coupling**: UI components have direct mapping logic embedded
5. **Data Flow Complexity**: Data flows through multiple processing steps with complex state updates

### **New Architecture**

```
JsonResultMapper (Orchestration + Data Processing + State Management)
├── Data Processing Logic
├── Mapping Processing Logic  
├── State Management
└── Coordinates with UI components

JsonExplorer (UI Component Library)
├── PathSearch (Search UI)
├── ProgressBar (Progress UI)
├── Expansion Controls (UI)
└── JsonRenderer (Pure Rendering - No Changes)
    ├── JsonAccordion
    ├── JsonNode
    └── rendering-utils.tsx (Pure rendering functions)
```

### **What Should Be Moved**

#### **FROM JsonExplorer TO JsonResultMapper:**

1. **Data Processing Logic** (Lines 121-150 in JsonExplorer):
   ```typescript
   // Move these functions to JsonResultMapper
   const processJson = useCallback(() => { ... })
   const processMappings = useCallback(() => { ... })
   const handleDataChange = useCallback(() => { ... })
   ```

2. **Processing Control States** (Lines 90-96):
   ```typescript
   // Move these states to JsonResultMapper
   const [jsonProcessed, setJsonProcessed] = useState(false)
   const [pathsGenerated, setPathsGenerated] = useState(false)
   const [mappingsProcessed, setMappingsProcessed] = useState(false)
   const [mappingStatisticsProcessed, setMappingStatisticsProcessed] = useState(false)
   const [dataRendered, setDataRendered] = useState(false)
   ```

3. **Data State Management** (Lines 74-76):
   ```typescript
   // Move these states to JsonResultMapper
   const [basePaths, setBasePaths] = useState<Record<string, BasePathData>>({})
   const [fullPaths, setFullPaths] = useState<Array<{...}>>([])
   ```

4. **Progress Statistics** (Line 88):
   ```typescript
   // Move to JsonResultMapper
   const [progressStats, setProgressStats] = useState<{...}>({...})
   ```

5. **Processing useEffect Chains** (Lines 257-285):
   ```typescript
   // Move all processing useEffects to JsonResultMapper
   useEffect(() => { handleDataChange() }, [data, handleDataChange])
   useEffect(() => { processJson() }, [data, jsonProcessed, processJson])
   useEffect(() => { processMappings() }, [mappings, pathsGenerated, processMappings])
   ```

#### **KEEP IN JsonExplorer (UI Component Library):**

1. **UI State Management** (Lines 67-72):
   ```typescript
   // Keep in JsonExplorer as UI component library
   const [searchQuery, setSearchQuery] = useState('')
   const [expandedPaths, setExpandedPaths] = useState<Set<string>>(new Set())
   const [selectedPath, setSelectedPath] = useState<string | undefined>(undefined)
   const [scrollToPath, setScrollToPath] = useState<string | undefined>(undefined)
   const [expansionMode, setExpansionMode] = useState<ExpansionOption>('show-structure')
   ```

2. **UI Event Handlers** (Lines 287-417):
   ```typescript
   // Keep in JsonExplorer as UI component library
   const handleExpansionChange = useCallback((option: ExpansionOption) => { ... })
   const handlePathSelect = useCallback((path: string, parentPaths?: string[]) => { ... })
   const handleNodeExpand = useCallback((path: string) => { ... })
   const handleNodeClick = useCallback((path: string, label?: string) => { ... })
   const handleDeselect = useCallback(() => { ... })
   ```

3. **UI useEffect for Expansion** (Lines 347-365):
   ```typescript
   // Keep in JsonExplorer as UI component library
   useEffect(() => { handleExpansionChange('show-structure') }, [dataRendered, handleExpansionChange])
   ```

4. **UI Rendering Logic** (Lines 487-542):
   ```typescript
   // Keep in JsonExplorer as UI component library
   return (
     <div className="h-full flex flex-col" ref={explorerRef}>
       <PathSearch ... />
       <ProgressBar ... />
       <ExpansionControls ... />
       <JsonRenderer ... />
     </div>
   )
   ```

#### **NO CHANGES TO JsonRenderer:**
- JsonRenderer remains a pure rendering component
- Takes processed data as props
- No data processing logic
- No state management
- Only rendering and UI event handling

### **How to Move**

#### **1. Update JsonResultMapper Interface:**
```typescript
interface JsonResultMapperProps {
  data: any
  mappings?: Mappings
  onProcessedData?: (processedData: {
    basePaths: Record<string, BasePathData>
    fullPaths: Array<{...}>
    progressStats: {mapped: number, recommendations: number, issues: number, total: number}
    dataRendered: boolean
  }) => void
}
```

#### **2. JsonResultMapper becomes the Orchestrator:**
```typescript
export function JsonResultMapper({ data, mappings, onProcessedData }: JsonResultMapperProps) {
  // All data processing logic moved here
  // All state management moved here
  // All processing useEffects moved here
  
  // Notify parent when data is ready
  useEffect(() => {
    if (onProcessedData && dataRendered) {
      onProcessedData({
        basePaths,
        fullPaths,
        progressStats,
        dataRendered
      })
    }
  }, [basePaths, fullPaths, progressStats, dataRendered, onProcessedData])
  
  return null // Pure data processor
}
```

#### **3. JsonExplorer becomes UI Component Library:**
```typescript
export function JsonExplorer({ data, mappings, options, onPathSelect }: JsonExplorerProps) {
  // Only UI state management
  const [expandedPaths, setExpandedPaths] = useState<Set<string>>(new Set())
  const [selectedPath, setSelectedPath] = useState<string | undefined>(undefined)
  // ... other UI states
  
  // Receive processed data from JsonResultMapper
  const [processedData, setProcessedData] = useState(null)
  
  // Only UI event handlers
  const handleExpansionChange = useCallback((option: ExpansionOption) => { ... })
  const handlePathSelect = useCallback((path: string, parentPaths?: string[]) => { ... })
  // ... other UI handlers
  
  return (
    <div className="h-full flex flex-col">
      <JsonResultMapper 
        data={data} 
        mappings={mappings} 
        onProcessedData={setProcessedData}
      />
      
      {processedData?.dataRendered && (
        <>
          <PathSearch fullPaths={processedData.fullPaths} onPathSelect={handlePathSelect} />
          <ProgressBar statistics={processedData.progressStats} />
          <ExpansionControls onExpansionChange={handleExpansionChange} />
          <JsonRenderer 
            data={data}
            basePaths={processedData.basePaths}
            fullPaths={processedData.fullPaths}
            expandedPaths={expandedPaths}
            selectedPath={selectedPath}
            onNodeExpand={handleNodeExpand}
            onNodeClick={handleNodeClick}
          />
        </>
      )}
    </div>
  )
}
```

#### **4. Clean Up rendering-utils.tsx:**
Move data processing functions to JsonResultMapper, keep only pure rendering functions:
```typescript
// Keep in rendering-utils.tsx (pure rendering)
export function renderPrimitiveNode(...)
export function renderArray(...)
export function renderObject(...)
export function renderJsonNode(...)

// Move to JsonResultMapper (data processing)
export function generatePathData(...)
export function getBasePaths(...)
export function getFullPaths(...)
```

### **Benefits of This Reorganization**

1. **Clear Separation of Concerns**: 
   - JsonResultMapper: Pure data processing and orchestration
   - JsonExplorer: Pure UI component library
   - JsonRenderer: Pure rendering engine (unchanged)

2. **Simplified Data Flow**:
   - Data processing happens in one place
   - UI components focus on presentation
   - State management is centralized and predictable

3. **Better Maintainability**:
   - Each component has a single responsibility
   - Easier to test individual components
   - Clearer debugging and troubleshooting

4. **Improved Performance**:
   - No unnecessary data conversions
   - Better performance (no transformation overhead)
   - Single source of truth for all data

5. **Easier Testing**:
   - Mock the data processors for component testing
   - Test UI components independently
   - Test data processing logic separately

### **Implementation Steps**

1. **Move Data Processing Logic**: Transfer all processing functions from JsonExplorer to JsonResultMapper
2. **Move State Management**: Transfer all data-related state from JsonExplorer to JsonResultMapper
3. **Update JsonExplorer**: Make it a pure UI component library that receives processed data
4. **Clean Up Utilities**: Separate data processing functions from rendering functions
5. **Update Interfaces**: Ensure all components have clean, focused interfaces
6. **Test Integration**: Verify the new architecture works end-to-end

This reorganization creates a much cleaner, more maintainable architecture where each component has a clear, single responsibility.

## Step 15: JSON Explorer Header Iteration Implementation Plan

### Overview
Restructure the JSON Explorer header section to include PathSearch, ProgressBar, Create Mapping button, and custom selectors for JSON switching and uploading, while fixing the path selection issue where both key and value nodes get selected.

### Current ProgressBar Implementation Analysis

**How ProgressBar Currently Works:**
1. **Location**: ProgressBar is rendered by `JsonRenderer` via the `renderBase()` function in `rendering-utils.tsx`
2. **Data Source**: Gets statistics from `rootBasePathData?.statistics` (root level basePath data)
3. **Rendering**: Returns both `accordion` and `progressBar` from `renderBase()` function
4. **Integration**: JsonRenderer calls `renderBase()` for the root level and gets both components
5. **Current Issue**: ProgressBar is embedded within the JSON content area, not in the header

**Current Data Flow:**
```
JsonExplorer → JsonRenderer → renderBase() → ProgressBar + JsonAccordion
```

### New Header Structure Design

#### **Refined Layout Design:**
```
┌─────────────────────────────────────────────────────────┐
│ [████████████████████████████████████████████████████] │ ← Progress Bar (full-width interactive border)
├─────────────────────────────────────────────────────────┤
│ [🔍 Search JSON paths...                    ] [X] [Create Mapping] [Add JSON] │ ← Search + Create + Add
├─────────────────────────────────────────────────────────┤
│ [JSON Source ▼] [Show Paths ▼]                         │ ← Configuration Selectors
└─────────────────────────────────────────────────────────┘
```

#### **Key Design Elements:**

**1. Progress Bar as Interactive Border**
- **Full-width** across the content wrapper
- **Interactive** - can be clicked to show detailed statistics
- **Visual separator** between header and content
- **Status indicator** showing overall mapping progress

**2. Search + Create Mapping Row**
- **PathSearch** with integrated clear button (X) on the right side
- **Create Mapping** primary button aligned to the right
- **Add JSON** secondary button (opens same dialog as JsonSelector)
- **Logical grouping** of primary actions

**3. Configuration Selectors Row**
- **JSON Source Selector** (Switch/Upload combined) - includes option to remove JSON in dialog
- **Show Paths Selector** (replaces current Select) - resets to placeholder when accordions are manually expanded/collapsed
- **Custom divider** between the two rows for visual separation

**4. Header Styling**
- **Bottom border** with "xl" shadow to prevent accordion content from going underneath
- **Fixed positioning** to stay above scrollable content
- **Consistent spacing** and alignment

#### **Component Hierarchy:**
```
JsonExplorer (Main Container)
├── Header Section (New consolidated header with xl shadow)
│   ├── Progress Bar (full-width interactive border)
│   ├── Search + Create Row
│   │   ├── PathSearch (with integrated X button)
│   │   └── Create Mapping Button
│   ├── Custom Divider
│   └── Configuration Row
│       ├── JSON Source Selector (Switch/Upload + Remove)
│       └── Show Paths Selector (resets on manual accordion changes)
└── Content Section (scrollable, goes underneath header)
    └── JsonRenderer (remove ProgressBar from here)
```

#### **JsonRenderer Returns Both Components, JsonExplorer Positions Them**

**Core Concept:**
- **JsonRenderer** continues to control data and rendering logic for both accordion and ProgressBar
- **JsonExplorer** receives both components and positions them where needed in the layout
- **Single source of truth** for statistics data (no duplication)

**Implementation:**
```typescript
// JsonRenderer returns both components
const renderResult = {
  accordion: <JsonAccordion ... />,
  progressBar: <ProgressBar statistics={rootStats} />
}

// JsonExplorer receives both via callback
onRenderComplete={(result) => setRenderResult(result)}

// JsonExplorer positions them
<div className="header-section shadow-xl border-b">
  <PathSearch />
  {renderResult.progressBar} {/* Positioned in header */}
</div>
<div className="content-section">
  {renderResult.accordion} {/* Positioned in content */}
</div>
```

### Path Selection Issue Fix

#### **Current Problem:**
- PathSearch calls `onPathSelect(path)` with clean path (e.g., `"api_response.data.name"`)
- Both key and value nodes check `selectedPath === path` and both get selected
- `fullPaths` already has `type: 'key' | 'value'` field to distinguish them

#### **The Fix: Use Type from fullPaths**

**Step 1: Update PathSearch to Pass Type**
```typescript
// In PathSearch - include type in search results
const searchResults = useMemo(() => {
  return fullPaths.map(fullPathEntry => ({
    path: fullPathEntry.path,
    value: fullPathEntry.value,
    status: fullPathEntry.status,
    type: fullPathEntry.type // Add type to search results
  }))
}, [fullPaths])

// Update handleResultSelect to pass type
const handleResultSelect = useCallback((path: string, type: 'key' | 'value') => {
  const parentPaths = generateParentPaths(path)
  onPathSelect(path, parentPaths, type) // Pass type as third parameter
  setSearchQuery('')
  setIsOpen(false)
}, [onPathSelect])
```

**Step 2: Update JsonExplorer to Handle Type-Specific Selection**
```typescript
// In JsonExplorer - add selectedType state
const [selectedType, setSelectedType] = useState<'key' | 'value' | undefined>(undefined)

// Update handlePathSelect to accept type
const handlePathSelect = useCallback((path: string, parentPaths?: string[], type?: 'key' | 'value') => {
  setSelectedPath(path)
  setSelectedType(type) // Store the selected type
  // ... rest of the logic
}, [expandedPaths, onPathSelect])
```

**Step 3: Update Rendering Functions to Use Type-Specific Selection**
```typescript
// In renderKeyNode and renderValueNode
isSelected={selectedPath === path && selectedType === type}
```

### Show Paths Selector Reset Behavior

#### **Requirement:**
The Show Paths selector should reset to placeholder (not chosen anything) when any accordion is expanded/collapsed manually.

#### **Implementation Strategy:**
```typescript
// In JsonExplorer - track manual accordion changes
const [showPathsValue, setShowPathsValue] = useState<ExpansionOption | undefined>(undefined)

// Track when accordions are manually expanded/collapsed
const handleNodeExpand = useCallback((path: string) => {
  // Reset showPaths selector to placeholder when manual expansion occurs
  setShowPathsValue(undefined)
  
  // Handle the actual expansion
  setExpandedPaths(prev => {
    const newSet = new Set(prev)
    newSet.add(path)
    return newSet
  })
}, [])

// Only allow showPaths to have a value when it's programmatically set
const handleShowPathsChange = useCallback((value: ExpansionOption) => {
  setShowPathsValue(value)
  // Handle expansion logic
}, [])

// Reset showPaths when manual changes occur
useEffect(() => {
  // Reset showPaths selector when expandedPaths change manually
  setShowPathsValue(undefined)
}, [expandedPaths])
```

### Accordion Grid Display Logic

#### **Requirement:**
When there are 0 mappings/issues/recommendations/options, don't display that cell in the accordion grid. Only display cells with 1 or more items.

#### **Implementation Strategy:**
- Implement conditional rendering in JsonAccordion component
- Only display cells when count > 0
- Use map-pin icon for mappings, lightbulb for recommendations, warning for issues
- Remove icon from options cell only
- Create dynamic grid that adjusts based on available data

#### **Key Points:**
- **Conditional Rendering**: Only render cells when count > 0
- **Icon Removal**: Remove icon only from "options" cell, keep icons in mappings/issues/recommendations
- **Dynamic Grid**: Grid adjusts automatically based on available data
- **Clean UI**: No empty cells cluttering the interface

### New Components to Create

#### **1. JsonSelector**
```typescript
// components/json/interface/json-selector.tsx
interface JsonSelectorProps {
  currentSource?: string
  availableSources?: Array<{id: string, name: string, description?: string}>
  onSourceChange?: (sourceId: string) => void
  onUploadNew?: () => void
  onRemoveSource?: () => void
  // Dialog can be opened from multiple triggers
  openDialog?: boolean
  onDialogOpenChange?: (open: boolean) => void
}
// Uses BaseSelector with dialog variant
// Dialog can be triggered from both JsonSelector and Add JSON button
```

#### **2. PathsExpandSelector**
```typescript
// components/json/interface/paths-expand-selector.tsx
interface PathsExpandSelectorProps {
  value?: ExpansionOption // Optional - can be undefined for placeholder
  onValueChange: (value: ExpansionOption) => void
  options: Array<{value: ExpansionOption, label: string}>
  placeholder?: string
}
// Uses BaseSelector with popover variant (replaces current Select)
// Resets to placeholder when accordions are manually changed
```

#### **3. Action Buttons**
- **Create Mapping**: Use existing `PrimaryButton` component directly
- **Add JSON**: Use existing `SecondaryButton` component directly
- **No new components needed** - use existing button components in JsonExplorer

### Manage JSONs Dialog Design

#### **Dialog Structure:**
```
┌─────────────────────────────────────────────────────────┐
│ Manage JSON Files                                    [X] │ ← Header
├─────────────────────────────────────────────────────────┤
│ ┌─────────────────────────────────────────────────────┐ │
│ │ File List (Scrollable)                             │ │
│ │ ┌─────────────────────────────────────────────────┐ │ │
│ │ │ 📄 source-response.json        [Select] [Remove] │ │ │ ← Clickable Row (except Remove)
│ │ └─────────────────────────────────────────────────┘ │ │
│ │ ┌─────────────────────────────────────────────────┐ │ │
│ │ │ 📄 sample-data.json           [Select] [Remove] │ │ │ ← Clickable Row (except Remove)
│ │ └─────────────────────────────────────────────────┘ │ │
│ │ ┌─────────────────────────────────────────────────┐ │ │
│ │ │ 📄 test-data.json             [Select] [Remove] │ │ │ ← Clickable Row (except Remove)
│ │ └─────────────────────────────────────────────────┘ │ │
│ └─────────────────────────────────────────────────────┘ │
│ ┌─────────────────────────────────────────────────────┐ │
│ │ Upload New JSON                                    │ │ ← Upload Area
│ │ [Drag & Drop Area]                                 │ │
│ └─────────────────────────────────────────────────────┘ │
├─────────────────────────────────────────────────────────┤
│                                    [Cancel] [Apply]    │ ← Footer
└─────────────────────────────────────────────────────────┘
```

#### **Key Features:**

**1. File List (Scrollable)**
- **Scrollable container** for multiple JSON files
- **File items** with name, size, last modified
- **Clickable rows** - entire row is selectable (except Remove button)
- **Select button** for each file (handled by JsonSelector)
- **Remove button** for each file (explicit action, not part of row click)
- **Empty state** when no files available

**2. Drag & Drop Area (Full Dialog)**
- **Entire dialog** is a drop zone for JSON files
- **Visual feedback** when dragging over
- **File validation** (JSON format, size limits)
- **Upload progress** indication

**3. Upload Section (Bottom)**
- **Small JsonUpload component** below the list
- **Consistent styling** with existing upload component
- **File input** for manual selection

**4. File Management**
- **Add new files** via drag & drop or upload
- **Remove files** with confirmation
- **Select active file** (communicates with JsonSelector)
- **File metadata** display (size, date, properties count)

#### **Component Integration:**
```typescript
// ManageJsonsDialog component
interface ManageJsonsDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  files: JsonFile[]
  selectedFile?: string
  onFileSelect: (fileId: string) => void
  onFileRemove: (fileId: string) => void
  onFileUpload: (file: File) => void
}

interface JsonFile {
  id: string
  name: string
  size: number
  lastModified: Date
  propertiesCount: number
  data: any
}
```

#### **Dialog Behavior:**
- **Selection handling**: File selection updates JsonSelector value
- **Row click selection**: Entire row is clickable for selection (except Remove button)
- **Remove confirmation**: Ask before deleting files
- **Upload feedback**: Show progress and validation errors
- **Keyboard navigation**: Arrow keys for file list navigation
- **Accessibility**: Proper ARIA labels and focus management

### Shared Dialog Implementation

#### **Multiple Trigger Support:**
The JsonSelector dialog can be opened from two different triggers:

1. **JsonSelector Button**: Shows current source + dropdown to switch/upload
2. **Add JSON Button**: Directly opens dialog for uploading new JSON

#### **Implementation Strategy:**
```typescript
// In JsonExplorer - shared dialog state
const [jsonDialogOpen, setJsonDialogOpen] = useState(false)

// JsonSelector can be controlled externally
<JsonSelector
  currentSource={currentSource}
  availableSources={availableSources}
  onSourceChange={handleSourceChange}
  onUploadNew={handleUploadNew}
  onRemoveSource={handleRemoveSource}
  openDialog={jsonDialogOpen}
  onDialogOpenChange={setJsonDialogOpen}
/>

// Add JSON button triggers same dialog
<SecondaryButton
  onClick={() => setJsonDialogOpen(true)}
  disabled={false}
>
  Add JSON
</SecondaryButton>
```

#### **Benefits:**
- **Consistent UX**: Same dialog interface from both triggers
- **Code Reuse**: Single dialog component handles all JSON management
- **Flexible Access**: Users can access JSON management from multiple entry points
- **Rich File Management**: Full CRUD operations for JSON files
- **Drag & Drop Support**: Intuitive file upload experience

### Implementation Steps

#### **Step 15.1: Fix Path Selection Issue**
1. Update PathSearch to include type in search results (add `type: fullPathEntry.type` to searchResults)
2. Update PathSearch to pass type to onPathSelect (modify `handleResultSelect` to pass type as third parameter)
3. Update JsonExplorer to handle selectedType state (add `selectedType` state and update `handlePathSelect`)
4. **Note**: Rendering functions are not needed - `isSelected` and search results should work with fullPath's "type" (value/key) field that already exists

#### **Step 15.2: Create New Selector Components**
1. Create JsonSelector using BaseSelector with dialog variant
2. Create PathsExpandSelector using BaseSelector with popover variant
3. Create ManageJsonsDialog component with file list and upload functionality
4. Use existing PrimaryButton and SecondaryButton components directly in JsonExplorer

#### **Step 15.3: Implement Approach 1 for ProgressBar**
1. Update JsonRenderer to return both accordion and progressBar (modify `renderBase()` to return object with both components)
2. Update JsonExplorer to receive both components via callback (add `onRenderComplete` callback pattern similar to existing component patterns)
3. Update JsonExplorer to position ProgressBar in header (move ProgressBar from content area to header section)
4. Remove ProgressBar from JsonRenderer's internal rendering (keep only in `renderBase()` return object)
5. **Consideration**: Follow similar patterns used in other components that return multiple elements or use callback patterns for component communication

#### **Step 15.4: Update Header Layout**
1. Create new consolidated header structure with bottom-only xl shadow (`shadow-xl border-b` classes) to create the effect where scrollable content appears to go underneath the fixed header
2. Position PathSearch, ProgressBar, and action buttons (PrimaryButton + SecondaryButton)
3. Replace current Select with PathsExpandSelector
4. Implement showPaths reset behavior on manual accordion changes
5. Connect SecondaryButton to same dialog as JsonSelector
6. **Use existing Separator component** (`/components/others/separator.tsx`) for custom divider between sections
7. **Use predefined CSS styles** and existing Tailwind classes for consistent styling
8. Test integration of all components

### File Structure After Implementation

```
components/json/interface/
├── json-explorer.tsx (updated header structure + path selection fix + showPaths reset + buttons)
├── json-renderer.tsx (returns both accordion and progressBar)
├── json-selector.tsx (new)
├── paths-expand-selector.tsx (new)
└── manage-jsons-dialog.tsx (new)

components/others/
├── path-search.tsx (updated to pass type information)
└── progress-bar.tsx (no changes, just moved to header)

components/ui-elements/buttons/
├── primary-button.tsx (used directly in JsonExplorer)
└── secondary-button.tsx (used directly in JsonExplorer)

utils/json/
└── rendering-utils.tsx (update renderKeyNode/renderValueNode for type-specific selection)
```

This implementation provides a clean, organized header with all requested functionality while fixing the path selection issue, implementing smart reset behavior for the showPaths selector, and ensuring proper visual separation with xl shadow.

## Step 16: JsonExplorer Custom Card Structure with Edge-to-Edge Progress Bar

### Overview
Transform JsonExplorer into a self-contained card component that duplicates BaseCard's visual design while providing edge-to-edge progress bar functionality and responsive height management. The component will break out of PageContent padding for the progress bar while maintaining proper card structure for the content.

### Current Structure Analysis

#### **Padding Sources:**
1. **PageContent**: `p-4` (16px padding on all sides)
2. **ContentWrapper**: No padding, but has `border` and `rounded-lg` 
3. **JsonExplorer inner div**: `p-6` (24px padding on all sides)

#### **Current Layout Hierarchy:**
```
MainContent
└── PageContent (p-4 padding)
    └── Tabs
        └── Mappers Tab
            └── ContentWrapper (border, rounded-lg, no padding)
                └── div (p-6 padding)
                    └── JsonResultMapper
                        └── JsonExplorer
                            └── Header (shadow-xl border-b)
                                └── Progress Bar (w-full)
```

### New JsonExplorer Card Structure

#### **Custom Container Implementation:**
The JsonExplorer will become a BaseCard-like container that duplicates the visual design of BaseCard components from the outside perspective. The container will use the same background, border, rounded corners, and shadow styling as BaseCard. The key difference is that it will use negative margins to break out of the PageContent padding, allowing the progress bar to span edge-to-edge while maintaining the card structure for the content.

The container will be structured with three main sections: a progress bar at the top that spans the full width, a header content section with consistent padding and xl bottom shadow at the bottom, and a scrollable content area at the bottom. The progress bar will be positioned to break out of the page padding while the header and content sections will maintain proper card-like spacing.

### Responsive Height Management

#### **Height Calculation Strategy:**
- **Use existing pattern**: Follow the same approach as DataTable with `calc(100vh-24.5rem)` for consistency
- **Fixed header**: Progress Bar and control sections remain visible at all times
- **Scrollable content**: Only the JsonRenderer (accordions) scrolls within the calculated height
- **Consistent with tables**: Use the same height calculation that accounts for main navigation, breadcrumbs, and page structure

#### **Height Management Implementation:**
The JsonExplorer will use the same height calculation pattern as the existing DataTable component, which uses `calc(100vh-24.5rem)` to account for the main page navigation, breadcrumbs, and page structure. This approach ensures consistency across the application and leverages the proven calculation that already works for similar full-screen components.

The 24.5rem value represents the combined height of the main page header, breadcrumbs navigation, page padding, and other UI elements that need to be excluded from the available viewport height. This calculation will be applied to the scrollable content area while keeping the progress bar and header controls fixed and visible at all times.

### Progress Bar Modifications (JsonExplorer-specific)

#### **Edge-to-Edge Implementation:**
- **Remove shadow**: Make Progress Bar appear flat when used inside JsonExplorer
- **Full-width implementation**: Ensure it spans the complete width of the JsonExplorer container
- **Responsive design**: Progress Bar should scale with JsonExplorer's width changes
- **Visual integration**: Make it look like a true "interactive border" at the top of the card

#### **Progress Bar Styling:**
The Progress Bar will be modified specifically for use within JsonExplorer to appear as a flat, integrated element rather than a standalone component. The shadow will be removed to create a seamless appearance with the card structure. The progress bar will use `rounded-t-lg` (top corners only) to match the container's `rounded-lg` corner radius, creating a seamless integration where the progress bar appears as a true "interactive border" at the top of the card.

The progress bar will be wrapped in a container that removes shadows and ensures it appears as a true "interactive border" at the top of the card. This will create a visual separation between the header and content while maintaining the edge-to-edge functionality and proper corner radius alignment.

### Header Layout Restructuring

#### **Header Layout Distribution:**
- **Search Row**: Search takes 4/5 width, Create Mapping takes 1/5 width
- **Selectors Row**: Show paths (2/5), Select JSON (2/5), Add JSON (1/5)
- **Grid Implementation**: Use CSS Grid with custom column fractions `grid-cols-[2fr_2fr_1fr]`
- **Responsive Behavior**: On smaller screens, stack vertically with full width

#### **Responsive Grid Implementation:**
- **Search Row**: Use `grid grid-cols-[4fr_1fr] gap-3` for 4/5 and 1/5 distribution
- **Selectors Row**: Use `grid grid-cols-[2fr_2fr_1fr] gap-3` for 2/5, 2/5, 1/5 distribution
- **Mobile Responsive**: Use `grid-cols-1 gap-3` on small screens for vertical stacking
- **Breakpoint Strategy**: Apply responsive classes `sm:grid-cols-[4fr_1fr]` and `sm:grid-cols-[2fr_2fr_1fr]`

#### **Consistent Spacing:**
- **All sections**: Use `py-4` for consistent 16px vertical spacing
- **Header content**: Use `px-6 py-4` (inheriting BaseCard's padding)
- **Content area**: Use `px-6 py-4` (inheriting BaseCard's padding)
- **Progress Bar**: No padding (truly edge-to-edge)

### Component Structure (Final)

```
JsonExplorer (BaseCard-like container with calculated height)
├── Progress Bar (edge-to-edge, flat, no shadow)
├── Header Content (px-6 py-4)
│   ├── Search Row (py-4, responsive width)
│   └── Selectors Row (py-4, responsive grid: "Show, Select JSON, Upload JSON")
└── Content (px-6 py-4, calculated height, overflow-auto)
    └── JsonRenderer (scrollable accordions, full width)
```

### Integration with Page Layout

#### **PageContent Compatibility:**
- **Break out of padding**: Use `-mx-4` to counteract PageContent padding
- **Maintain card structure**: Keep BaseCard-like appearance
- **Responsive behavior**: Adapt to different screen sizes while maintaining full-screen utilization
- **Consistent spacing**: Maintain proper spacing with other page elements

#### **Layout Integration:**
The components-showcase page will be updated to remove the ContentWrapper dependency and pass the calculated height directly to JsonExplorer. The JsonResultMapper will continue to process the data and pass the processed results to JsonExplorer, but JsonExplorer will now handle its own container structure and height management.

The integration will ensure that JsonExplorer receives all necessary data including the calculated container height, allowing it to manage its own responsive behavior and full-screen utilization. This eliminates the need for external ContentWrapper components while maintaining the same data flow and processing logic.

### Benefits of This Approach

1. **Perfect Control**: Complete control over spacing and layout
2. **Edge-to-Edge Progress Bar**: Can easily break out with `-mx-4`
3. **Consistent Spacing**: All `py-4` and `px-6` throughout
4. **ContentWrapper-like**: Uses the same base classes as ContentWrapper
5. **No Unnecessary Structure**: No CardHeader/CardContent complexity
6. **Flexible**: Easy to modify for future needs
7. **Responsive**: Adapts to different screen sizes
8. **Full-Screen Utilization**: Uses available viewport height efficiently

### Implementation Steps

#### **Step 16.1: Update JsonExplorer Container Structure**
1. Replace current container with BaseCard-like structure
2. Add `-mx-4` to break out of PageContent padding
3. Implement edge-to-edge progress bar positioning
4. Add consistent `px-6 py-4` spacing throughout

#### **Step 16.2: Implement Responsive Height Management**
1. Add height calculation logic with viewport measurements
2. Implement window resize event handling
3. Set minimum height constraints for usability
4. Update content area to use calculated height

#### **Step 16.3: Modify Progress Bar for JsonExplorer**
1. Remove shadow for flat appearance
2. Ensure edge-to-edge positioning
3. Make responsive to container width changes
4. Integrate as "interactive border" at top

#### **Step 16.4: Reorder Header Selectors**
1. Change order to "Show paths, Select JSON, Add JSON"
2. Maintain 3-column grid layout
3. Ensure responsive behavior
4. Test selector functionality

#### **Step 16.5: Update Components Showcase Integration**
1. Remove ContentWrapper from Mappers tab
2. Pass calculated height to JsonExplorer
3. Ensure proper responsive behavior
4. Test full-screen utilization

### Expected Outcome

**Working Features:**
- ✅ Progress Bar truly edge-to-edge (breaks out of PageContent padding)
- ✅ JsonExplorer looks identical to BaseCard from the outside
- ✅ Consistent `py-4` spacing throughout
- ✅ Responsive height management using full viewport
- ✅ Fixed header with scrollable content
- ✅ Reordered selectors: "Show, Select JSON, Upload JSON"
- ✅ Clean, maintainable card structure

**Visual Result:**
- Progress Bar spans from edge to edge of the viewport
- Header controls remain visible at all times
- JSON content scrolls within calculated height
- Card maintains BaseCard's visual design
- Responsive behavior across all screen sizes

## Step 17: Universal Height Management Hook and Safe JsonExplorer Implementation

### Overview
Create a reusable height calculation hook that can be used across multiple components (DataTable, JsonExplorer, forms, etc.) and implement the safe JsonExplorer improvements that won't break existing functionality.

### Universal Height Management Hook

#### **Hook Design:**
Create a custom hook `useViewportHeight` that provides consistent height calculations across the application. This hook will handle viewport measurements, window resize events, and provide calculated heights for different component types.

#### **Hook Features:**
- **Consistent Calculation**: Uses the same `calc(100vh-24.5rem)` pattern across all components
- **Responsive Updates**: Automatically recalculates on window resize
- **Minimum Height Constraints**: Ensures usability with minimum height limits
- **Component-Specific Adjustments**: Allows for component-specific height modifications
- **Performance Optimized**: Debounced resize handling to prevent excessive calculations

#### **Hook Implementation:**
The hook will be placed in `hooks/use-viewport-height.ts` and will provide a standardized way to calculate available viewport height for any component that needs full-screen or near-full-screen behavior.

#### **Usage Across Components:**
- **DataTable**: Already uses `calc(100vh-24.5rem)` - can be refactored to use the hook
- **JsonExplorer**: Will use the hook for its scrollable content area
- **Forms**: Can use the hook for full-screen form layouts
- **Future Components**: Any component needing viewport height calculations

### Safe JsonExplorer Improvements

#### **Definitely Safe Changes:**
These changes can be implemented without risk of breaking existing functionality:

1. **State Management Simplification**:
   - Consolidate `expansionMode` and `showPathsValue` into single `selectedExpansionMode` state
   - Remove redundant state reset logic that runs too frequently
   - Implement clear naming that indicates placeholder state

2. **Event Handling Cleanup**:
   - Simplify click detection system by removing nested conditions
   - Remove debug logging statements for production readiness
   - Streamline path selection logic to single, clean approach

3. **Data Processing Optimization**:
   - Remove outdated path cleaning for `:key` or `:value` suffixes (no longer used)
   - Consolidate repeated path processing logic into single functions
   - Remove unused imports and refs

4. **Code Cleanup**:
   - Remove unused `selectedNodeRef` and other unused refs
   - Clean up unused imports like `Divider`
   - Remove debug console.log statements throughout

5. **Component Structure**:
   - Reorder selectors to "Show paths, Select JSON, Add JSON"
   - Implement consistent spacing with `py-4` throughout
   - Use existing `PrimaryButton` and `SecondaryButton` components directly

#### **JSON File Handling Analysis**:
The current JSON processing system is designed to handle any valid JSON structure:

- **`generatePathData(data: any)`**: Accepts any valid JSON data type
- **`getBasePaths(data: any)`**: Traverses any JSON structure recursively
- **`getFullPaths(data: any)`**: Handles any JSON data type (objects, arrays, primitives)
- **Type Safety**: Uses `any` type for maximum flexibility with any JSON structure
- **Error Handling**: JSON parsing includes try-catch blocks for invalid JSON
- **Validation**: File upload includes JSON format validation

**Conclusion**: The system is already designed to handle any valid JSON file structure. The mapping system works with the specific structure in `source-mappings.json`, but the JSON rendering and path generation works with any valid JSON.

### Implementation Steps

#### **Step 17.1: Create Universal Height Management Hook**
1. Create `hooks/use-viewport-height.ts` with standardized height calculation
2. Include responsive updates and minimum height constraints
3. Add component-specific adjustment options
4. Implement debounced resize handling for performance

#### **Step 17.2: Implement Safe JsonExplorer Improvements**
1. **State Management**: Consolidate expansion mode states into single variable
2. **Event Handling**: Simplify click detection and remove debug logging
3. **Data Processing**: Remove outdated path cleaning and consolidate logic
4. **Code Cleanup**: Remove unused imports, refs, and debug statements
5. **Component Structure**: Reorder selectors and implement consistent spacing

#### **Step 17.3: Refactor DataTable to Use Height Hook**
1. Replace hardcoded `calc(100vh-24.5rem)` with `useViewportHeight` hook
2. Ensure consistent behavior across all components
3. Test that DataTable still works correctly

#### **Step 17.4: Update JsonExplorer to Use Height Hook**
1. Replace current height calculation with `useViewportHeight` hook
2. Ensure responsive behavior works correctly
3. Test that JsonExplorer maintains proper height management

#### **Step 17.5: Test JSON File Compatibility**
1. Test with various JSON file structures (nested objects, arrays, primitives)
2. Verify that path generation works with any valid JSON
3. Ensure mapping system works correctly with different JSON structures
4. Test error handling with invalid JSON files

### Expected Outcome

**Working Features:**
- ✅ Universal height management hook for consistent viewport calculations
- ✅ Simplified JsonExplorer state management
- ✅ Cleaned up event handling and data processing
- ✅ Removed unused code and debug statements
- ✅ Reordered selectors: "Show paths, Select JSON, Add JSON"
- ✅ Consistent spacing and component structure
- ✅ Any valid JSON file structure supported
- ✅ No breaking changes to existing functionality

**Benefits:**
- **Reusable Height Management**: Single hook for all components needing viewport height
- **Cleaner Code**: Removed redundancy and unused code
- **Better Performance**: Optimized event handling and data processing
- **Maintainable**: Clear state management and component structure
- **Flexible**: Supports any valid JSON file structure
- **Consistent**: Standardized height calculations across the application

## Step 18: Universal ProgressBar with Dynamic Labels and Clean Visual Design

### Overview
Create a universal ProgressBar component that supports dynamic labels for different mapping contexts, removes text overlays for clean visual design, and integrates proper hovercard functionality. This includes creating a universal singular/plural utility and translation functions to support various mapping types.

### Current ProgressBar Analysis

**Current Implementation:**
- **Fixed Labels**: Hardcoded "Mapped", "Recommendations", "Issues", "Options"
- **Text Overlays**: Labels with numbers displayed inside progress sections (can get truncated)
- **Fixed Status Types**: Only supports 4 status types: "done", "suggested", "error", "todo"
- **Hovercard Issues**: Shows individual steps instead of summary statistics
- **No Dynamic Support**: Cannot change labels based on mapping context

**Required Changes:**
- **Dynamic Labels**: Accept labels as props instead of hardcoded values
- **Clean Visual**: Remove text overlays from progress bar sections
- **Enhanced Hovercard**: Show summary statistics instead of individual steps
- **Universal Utility**: Create reusable singular/plural logic
- **Translation Function**: Convert basePaths to ProgressBar format

### Implementation Steps

#### **Step 18.1: Create Universal Singular/Plural Utility**
**File**: `utils/singular-plural.ts`
- Extract existing singular/plural logic from `renderItemsCount` function and `accordion-grid.tsx`
- Create hardcoded word list with mapping/mappings, issue/issues, recommendation/recommendations, option/options, item/items
- Implement reusable function that takes count and word type, returns appropriate singular/plural form
- **Found locations**: `renderItemsCount` (utils/json/rendering-utils.tsx) and `getPredefinedLabel` (components/others/accordion-grid.tsx)

#### **Step 18.2: Create Progress Statistics Translation Function**
**File**: `utils/json/rendering-utils.tsx`
- Add new function `getProgressStatisticsFromBasePaths`
- Input: basePaths with current statistics structure (mapped, recommendations, issues, options)
- Output: new statistics format with count and label objects for each status
- Map current statuses: mapped→success, recommendations→recommended, issues→issue, options→default
- Use singular/plural utility to generate proper labels
- Return structure: {success: {count, label}, issue: {count, label}, recommended: {count, label}, default: {count, label}}

#### **Step 18.3: Update ProgressBar Component Interface**
**File**: `components/others/progress-bar.tsx`
- Rename variants: 'default'/'json-explorer' → 'standalone'/'header'
- Update statistics prop interface to accept new format with count and label objects
- Remove label display from progress bar sections entirely (no text overlays)
- Keep only colored sections for visual progress indication
- Fix hovercard content to show statistics summary instead of individual steps
- Map external status types to internal rendering types

#### **Step 18.4: Fix Hovercard Content Display**
**File**: `components/others/progress-bar.tsx`
- Replace current steps-based hovercard content with statistics-based content
- Display 4 status types with their counts and labels in clean format
- Show color indicators matching progress bar sections
- Use proper spacing and typography for readability

#### **Step 18.5: Update JsonExplorer Integration**
**File**: `components/json/interface/json-explorer.tsx`
- Call `getProgressStatisticsFromBasePaths` function before passing data to ProgressBar
- Pass translated statistics instead of raw basePaths
- Change variant from 'json-explorer' to 'header'
- Ensure data flows through translation function

#### **Step 18.6: Update Existing Singular/Plural Usage**
**File**: `utils/json/rendering-utils.tsx`
- **Modify** `renderItemsCount` function to use new utility (don't replace completely)
- Keep same functionality but use centralized utility for consistency

**File**: `components/others/accordion-grid.tsx`
- **Modify** `getPredefinedLabel` function to use new utility
- Replace hardcoded singular/plural logic with utility function
- Maintain same functionality with improved consistency

### Found Singular/Plural Usage Locations

1. **`utils/json/rendering-utils.tsx`** - `renderItemsCount` function (item/items)
2. **`components/others/accordion-grid.tsx`** - `getPredefinedLabel` function (mapping/mappings, recommendation/recommendations, issue/issues, option/options)
3. **`components/others/progress-bar.tsx`** - `calculateProgressSections` function (label.replace(/s$/, ''))

### Expected Outcome

**Working Features:**
- ✅ Universal ProgressBar with dynamic labels based on mapping context
- ✅ Clean visual design without truncated text overlays
- ✅ Enhanced hovercard showing summary statistics
- ✅ Universal singular/plural utility for consistent labeling
- ✅ Translation function for basePaths to ProgressBar format
- ✅ Context-appropriate labeling for different mapping types
- ✅ Maintained responsive design and accessibility
- ✅ Backward compatibility with existing usage

**Visual Improvements:**
- **Clean Progress Bar**: No text overlays, only colored sections
- **Detailed Hovercard**: Comprehensive statistics display on hover
- **Context Awareness**: Labels that match the current mapping type
- **Professional Appearance**: Clean, consistent labeling across all contexts
- **No Truncation**: All text visible in hovercard regardless of section width

**Integration Benefits:**
- **Universal Usage**: Single component works with any mapping context
- **Easy Maintenance**: Centralized singular/plural logic
- **Consistent UX**: Same visual behavior across different mappers
- **Future-Proof**: Easy to add new mapping types and labels
- **Clean Architecture**: Separation of concerns with translation functions

## Step 19: Enhanced Progress Hovercard with Key Lists and Instant Display

### Overview
Enhance the progress hovercard to display detailed key lists from fullPaths data, show up to 3 paths per status with "Show all" functionality, and implement instant hover display without delay. This creates a comprehensive overview of mapping progress with actionable details.

### Current Progress Hovercard Analysis

**Current Implementation:**
- **Basic Statistics**: Shows only counts and labels for each status
- **No Key Details**: Cannot see which specific paths are mapped, have issues, or recommendations
- **Hover Delay**: Uses default hover delay which can feel sluggish
- **Limited Context**: No way to see actual path information for troubleshooting

**Required Enhancements:**
- **Key Lists Display**: Show actual paths from fullPaths data for each status
- **Limited Display**: Show up to 3 paths per status with "and X more" indicator
- **Show All Functionality**: "Show all" buttons that trigger expansion controls
- **Instant Hover**: Remove hover delay for immediate feedback
- **Path Rendering**: Use JsonPath component for consistent path display

### Implementation Steps

#### **Step 19.1: Create Path Extraction Utility Function**
**File**: `utils/json/rendering-utils.tsx`
- Add new function `getPathsByStatus(fullPaths: Array<FullPathData>)`
- Input: Array of fullPath objects with path, type, value, status fields
- Output: Object with arrays of fullPath objects per status: `{mappings: FullPathData[], issues: FullP-athData[], recommendations: FullPathData[], default: FullPathData[]}`
- Filter fullPaths by status and return actual fullPath objects (not just strings)
- Limit to top 3 per status for initial display
- Include total count for "and X more" functionality

#### **Step 19.2: Update Progress Hovercard Interface**
**File**: `components/hovercards/progress-hovercard.tsx`
- Add new prop `enhancedKeys?: {mappings: FullPathData[], issues: FullPathData[], recommendations: FullPathData[], default: FullPathData[]}`
- Add new prop `onShowAll?: (status: 'mapped' | 'issue' | 'recommendation' | 'default') => void`
- Update interface to accept both basic statistics and enhanced key lists
- Maintain backward compatibility with existing usage

#### **Step 19.3: Implement Enhanced Hovercard Content**
**File**: `components/hovercards/progress-hovercard.tsx`
- Import `JsonPath` component from `@/components/json/interface/json-path`
- Display statistics summary (existing functionality)
- Add sections for each status with key lists when enhancedKeys provided
- Show up to 3 paths per status using `JsonPath` component with proper props:
  - `path`: fullPath.path
  - `value`: fullPath.value
  - `type`: fullPath.type
  - `mappingStatus`: fullPath.status
- Display "and X more" when there are additional paths
- Add "Show all" buttons that call onShowAll callback
- Use proper status labels: "Mappings", "Issues", "Recommendations", "Remaining"

#### **Step 19.4: Implement Instant Hover Display**
**File**: `components/hovercards/progress-hovercard.tsx`
- Remove hover delay by setting `openDelay={0}` on HoverCard
- Ensure immediate display when hovering over progress bar
- Maintain smooth animations and transitions
- Test responsiveness across different devices

#### **Step 19.5: Update ProgressBar Integration**
**File**: `components/others/progress-bar.tsx`
- Add enhancedKeys prop to ProgressBarProps interface
- Add onShowAll prop for expansion control integration
- Pass enhancedKeys to ProgressHovercard component
- Ensure data flows from JsonExplorer through ProgressBar to Hovercard

#### **Step 19.6: Update JsonExplorer Integration**
**File**: `components/json/interface/json-explorer.tsx`
- Call `getPathsByStatus(fullPaths)` to extract key lists
- Pass enhancedKeys to ProgressBar component
- Implement onShowAll callback that triggers expansion controls
- Map status types correctly: mapped→mappings, issue→issues, recommendation→recommendations, default→default

#### **Step 19.7: Connect Show All Functionality**
**File**: `components/json/interface/json-explorer.tsx`
- Implement onShowAll callback that calls existing expansion functions
- Map status to appropriate expansion mode:
  - 'mapped' → 'show-mapped'
  - 'issue' → 'show-issues' 
  - 'recommendation' → 'show-recommended'
  - 'default' → 'show-all'
- Ensure expansion controls work with the enhanced hovercard

### Data Flow

```
JsonExplorer
├── fullPaths (Array<FullPathData>)
├── getPathsByStatus(fullPaths) → enhancedKeys
├── ProgressBar
│   ├── statistics (basic counts)
│   └── enhancedKeys (path details)
│       └── ProgressHovercard
│           ├── Statistics summary
│           ├── Key lists (up to 3 per status)
│           ├── "and X more" indicators
│           └── "Show all" buttons → onShowAll → expansion controls
```

### Expected Outcome

**Working Features:**
- ✅ Enhanced hovercard showing detailed key lists from fullPaths
- ✅ Up to 3 paths per status with "and X more" indicators
- ✅ "Show all" buttons that trigger appropriate expansion controls
- ✅ Instant hover display without delay
- ✅ **JsonPath component integration** for consistent path rendering with proper props
- ✅ Backward compatibility with existing ProgressBar usage
- ✅ Proper status mapping and expansion control integration

**Visual Improvements:**
- **Detailed Overview**: See exactly which paths are mapped, have issues, or need attention
- **Actionable Information**: "Show all" buttons provide direct access to relevant sections
- **Immediate Feedback**: Instant hover response for better user experience
- **Consistent Path Display**: JsonPath component ensures uniform path formatting
- **Clean Organization**: Clear sections for each status type with proper labeling

**Integration Benefits:**
- **Enhanced Debugging**: Quickly identify specific paths that need attention
- **Improved Workflow**: Direct access to relevant sections via "Show all" buttons
- **Better Context**: Understand mapping progress at a detailed level
- **Consistent UX**: Same path rendering as the main JSON explorer using `JsonPath` component
- **Status Icons**: Visual indicators (CheckCircle, Info, AlertCircle) for each path status
- **Proper Formatting**: Consistent path display with chevrons and value formatting
- **Future-Proof**: Easy to extend with additional status types or functionality

## Step 20: Enhanced Progress Hovercard Improvements

### Overview
Enhance the progress hovercard with icon standardization, text button integration, accurate "and more" count display, clickable JsonPath components, dynamic hovercard positioning, and enhanced visual distinction. This creates a comprehensive, interactive progress overview with consistent styling and improved user experience.

### Current Progress Hovercard Analysis

**Current Implementation:**
- **Inconsistent Icons**: Uses CheckCircle/Info/AlertCircle instead of AccordionGrid's MapPin/Lightbulb/AlertTriangle
- **Plain Buttons**: Uses basic HTML buttons instead of custom TextButton component
- **Generic "And More"**: Shows "and more..." instead of actual count like "+ X more"
- **Non-Clickable Paths**: JsonPath components are not interactive
- **Static Positioning**: Hovercard appears in center, doesn't follow cursor
- **Basic Shadow**: Uses default shadow, not distinguished from other white components

**Required Enhancements:**
- **Icon Standardization**: Use same icons as AccordionGrid (MapPin, Lightbulb, AlertTriangle)
- **Text Button Integration**: Replace plain buttons with custom TextButton component
- **Accurate Count Display**: Show actual count as "+ X more" instead of generic text
- **Clickable JsonPath Components**: Make paths clickable with same functionality as search
- **Dynamic Positioning**: Hovercard should follow cursor within progress bar bounds
- **Enhanced Visual Distinction**: Add xl drop shadow to distinguish from other components

### Implementation Steps

#### **Step 20.1: Update JsonPath Component for Clickability**
**File**: `components/json/interface/json-path.tsx`
- Add `onClick?: (path: string, type: 'key' | 'value') => void` prop to JsonPathProps interface
- Add clickable wrapper div with hover effects and cursor pointer
- Call onClick when JsonPath is clicked, passing path and type
- Ensure clickable behavior matches search results functionality
- Add proper accessibility attributes (role, tabIndex, onKeyDown)

#### **Step 20.2: Update ProgressHovercard Component**
**File**: `components/hovercards/progress-hovercard.tsx`
- Import correct icons (MapPin, Lightbulb, AlertTriangle) to match AccordionGrid
- Import TextButton component from `@/components/ui-elements/buttons/text-button`
- Replace icon logic to match AccordionGrid exactly:
  - Mappings: MapPin icon
  - Issues: AlertTriangle icon  
  - Recommendations: Lightbulb icon
- Replace "Show all" buttons with TextButton components using "sm" size
- Add onPathSelect prop to interface for clickable JsonPath functionality
- Calculate and display actual "more" counts (total - 3 displayed) as "+ X more"
- Pass onPathSelect to JsonPath components for clickability
- Add click handlers for JsonPath components that trigger path selection

#### **Step 20.3: Update ProgressBar Component**
**File**: `components/others/progress-bar.tsx`
- Add onPathSelect prop to ProgressBarProps interface
- Pass onPathSelect to ProgressHovercard component
- Implement cursor tracking for dynamic positioning
- Add mouse move event handling to track cursor position within progress bar
- Calculate hovercard position based on cursor location
- Ensure hovercard stays within progress bar bounds and viewport

#### **Step 20.4: Update JsonExplorer Component**
**File**: `components/json/interface/json-explorer.tsx`
- Pass onPathSelect callback to ProgressBar component
- Ensure the same handlePathSelect function works for both search and hovercard
- Implement path selection logic that handles both key and value types
- Update path selection to work with JsonPath components in hovercard

#### **Step 20.5: Update BaseHovercard Component**
**File**: `components/hovercards/base-hovercard.tsx`
- Add xl drop shadow class to HoverCardContent for enhanced visual distinction
- Add dynamic positioning support based on cursor location
- Add prop to control positioning behavior (static vs dynamic)
- Ensure hovercard follows cursor within trigger bounds
- Add viewport boundary detection to prevent hovercard from going outside screen
- Implement smooth positioning transitions

#### **Step 20.6: Update Data Calculation for "More" Counts**
**File**: `utils/json/rendering-utils.tsx`
- Modify getPathsByStatus to also return total counts per status
- Use total counts to calculate "more" counts in ProgressHovercard
- Ensure accurate count calculation for "and X more" display
- Handle edge cases where there are exactly 3 or fewer items

#### **Step 20.7: Update Show Selector Colors and Labels**
**File**: `components/selectors/paths-expand-selector.tsx`
- Update placeholder from "Show paths..." to "Show only..."
- Update option labels to: "Mappings", "Issues", "Recommendations", "Everything", "Structure"
- Add color coding for each option based on status:
  - "Mappings" → Green (`text-success`)
  - "Issues" → Yellow (`text-warning`)
  - "Recommendations" → Light Blue (`text-recommendation`)
  - "Everything" → Muted Gray (`text-muted-foreground`)
  - "Structure" → Primary Blue (`text-primary`)

#### **Step 20.8: Update Progress Hovercard Section Colors**
**File**: `components/hovercards/progress-hovercard.tsx`
- Apply color coding to section headlines:
  - "Mappings" → Green (`text-success`)
  - "Issues" → Yellow (`text-warning`)
  - "Recommendations" → Light Blue (`text-recommendation`)
  - "Remaining" → Muted Gray (`text-muted-foreground`)

#### **Step 20.9: Fix Manage JSON Dialog Overlay**
**File**: `components/ui/dialog.tsx`
- Change overlay from `bg-black/80` to `bg-muted/70`
- Use white or gray background with 70% opacity for better visibility
- Ensure dialog content remains clearly visible with lighter overlay

### Data Flow

```
JsonExplorer
├── fullPaths (Array<FullPathData>)
├── getPathsByStatus(fullPaths) → enhancedKeys with total counts
├── ProgressBar
│   ├── statistics (basic counts)
│   ├── enhancedKeys (path details with counts)
│   ├── onPathSelect (path selection callback)
│   └── ProgressHovercard
│       ├── Statistics summary
│       ├── Key lists (up to 3 per status)
│       ├── "+ X more" indicators (accurate counts)
│       ├── "Show all" buttons (TextButton components)
│       ├── Clickable JsonPath components
│       └── Dynamic positioning (follows cursor)
```

### Expected Outcome

**Working Features:**
- ✅ Icon standardization matching AccordionGrid (MapPin, Lightbulb, AlertTriangle)
- ✅ TextButton integration for consistent button styling
- ✅ Accurate "and more" count display as "+ X more"
- ✅ Clickable JsonPath components with same functionality as search
- ✅ Dynamic hovercard positioning that follows cursor within progress bar
- ✅ Enhanced xl drop shadow for visual distinction
- ✅ Color-coded show selector options with status-based colors
- ✅ Color-coded progress hovercard section headlines
- ✅ Lighter dialog overlay for better visibility
- ✅ Updated placeholder text: "Show only..." instead of "Show paths..."
- ✅ Updated option labels: "Mappings", "Issues", "Recommendations", "Everything", "Structure"
- ✅ Consistent styling and behavior across all components
- ✅ Improved user experience with interactive elements

**Visual Improvements:**
- **Consistent Icons**: Same visual language as AccordionGrid component
- **Professional Buttons**: Custom TextButton styling for better UX
- **Accurate Information**: Real counts instead of generic "and more" text
- **Interactive Paths**: Clickable JsonPath components for direct navigation
- **Dynamic Positioning**: Hovercard follows cursor for better usability
- **Enhanced Visibility**: xl drop shadow distinguishes hovercard from other elements
- **Color-Coded Options**: Show selector options use status-based colors for easy identification
- **Color-Coded Sections**: Progress hovercard section headlines match their status colors
- **Better Dialog Visibility**: Lighter overlay improves readability and user experience
- **Clear Labeling**: Updated placeholder and option labels provide better context

**Integration Benefits:**
- **Unified Interaction**: Same path selection behavior everywhere
- **Consistent Styling**: Matching icons and components across the application
- **Better UX**: Interactive elements with proper feedback and positioning
- **Enhanced Debugging**: Clickable paths for quick navigation and troubleshooting
- **Professional Appearance**: Consistent visual design and interaction patterns
- **Future-Proof**: Easy to extend with additional functionality and styling

### Implementation Order

1. **Update JsonPath Component**: Add onClick functionality and accessibility
2. **Update ProgressHovercard**: Icons, TextButton, counts, clickable paths
3. **Update ProgressBar**: onPathSelect prop, cursor tracking
4. **Update JsonExplorer**: Pass onPathSelect to ProgressBar
5. **Update BaseHovercard**: xl shadow, dynamic positioning
6. **Update Data Calculation**: Total counts for "more" display
7. **Update Show Selector**: Colors, labels, and placeholder text
8. **Update Progress Hovercard Sections**: Color-coded section headlines
9. **Fix Dialog Overlay**: Lighter overlay for better visibility

This comprehensive plan addresses all the identified requirements while maintaining backward compatibility and ensuring a consistent, enhanced user experience.

## Step 21: Mapper Type Integration and Variable Path Support

### Overview
Integrate mapper type support throughout the component hierarchy to enable different data display formats for result mappers (JSON paths) versus request/response mappers (variable:path format). This creates a unified system that can handle both object/datapoint mappings and variable mappings with appropriate rendering for each context.

### Current State Analysis

**Current Implementation:**
- **Single Data Format**: All mappers use the same `enhancedKeys` structure with JSON paths
- **No Mapper Type Distinction**: Components don't know whether they're displaying result, request, or response data
- **Limited Variable Support**: Variable mappings exist but aren't properly displayed in progress hovercard
- **Generic Labels**: All mappers use "options" label instead of context-appropriate "remaining" for variables

**Required Enhancements:**
- **Mapper Type Propagation**: Pass `mapperType` through component hierarchy
- **Data Format Distinction**: Different data structures for JSON paths vs variable:path format
- **Conditional Rendering**: ProgressHovercard renders differently based on mapper type
- **Variable Path Support**: Display variable names with their associated JSON paths
- **Context-Appropriate Labels**: "options" for result mappers, "remaining" for request/response mappers

### Implementation Steps

#### **Step 21.1: Rename `enhancedKeys` to `mappingsShortlist`**
**Files**: `components/others/progress-bar.tsx`, `components/hovercards/progress-hovercard.tsx`, `components/json/interface/json-explorer.tsx`, `utils/json/rendering-utils.tsx`
- Update interface name from `EnhancedKeys` to `MappingsShortlist`
- Update all prop names and references across components
- Update `getPathsByStatus` return type to use new interface name
- Ensure consistent naming throughout the codebase

#### **Step 21.2: Add `mapperType` prop throughout component hierarchy**
**Files**: `components/others/progress-bar.tsx`, `components/hovercards/progress-hovercard.tsx`, `components/json/interface/json-explorer.tsx`, `components/json/interface/json-renderer.tsx`
- Add `mapperType?: 'result' | 'request' | 'response'` to all relevant component interfaces
- Pass `mapperType` prop from each mapper component down the hierarchy
- Ensure JsonExplorer and JsonRenderer receive `mapperType` for future use

#### **Step 21.3: Create `VariablePathData` interface and `getVariablePathsByStatus` function**
**File**: `utils/json/rendering-utils.tsx`
- Create `VariablePathData` interface with `variableName: string`, `path: string`, `status: 'mapped' | 'issue' | 'recommendation' | 'default'`
- Create `getVariablePathsByStatus` function that:
  - Takes `requestAndResponseVariableMappings` and `fullPaths` as parameters
  - Reuses logic from `getVariableMappingStatistics()` (variable extraction, status determination, remaining calculation)
  - Returns up to 3 items per status for hovercard display
  - Handles variable:path format for request/response mappers

#### **Step 21.4: Update each mapper component**
**Files**: `components/json/response/json-result-mapper.tsx`, `components/json/request/json-request-mapper.tsx`, `components/json/response/json-response-mapper.tsx`
- **JsonResultMapper**: 
  - Pass `mapperType="result"`
  - Call `getPathsByStatus(fullPaths)` for `mappingsShortlist`
  - Use `getObjectAndDatapointMappingStatistics(basePaths)` for statistics
- **JsonRequestMapper**: 
  - Pass `mapperType="request"`
  - Call `getVariablePathsByStatus(requestAndResponseVariableMappings, fullPaths)` for `mappingsShortlist`
  - Use `getVariableMappingStatistics(requestAndResponseVariableMappings, fullPaths)` for statistics
- **JsonResponseMapper**: 
  - Pass `mapperType="response"`
  - Call `getVariablePathsByStatus(requestAndResponseVariableMappings, fullPaths)` for `mappingsShortlist`
  - Use `getVariableMappingStatistics(requestAndResponseVariableMappings, fullPaths)` for statistics

#### **Step 21.5: Update ProgressHovercard rendering logic**
**File**: `components/hovercards/progress-hovercard.tsx`
- Add conditional rendering based on `mapperType` for the "default" section
- **Result mapper**: Show `JsonPath` components (existing behavior)
- **Request/Response mappers**: Show variable:path format:
  - **Variable name** as small text subheadline
  - **JSON path** below it (using `JsonPath` component)
  - Format: `variableName: path` display
- Ensure proper styling and layout for both formats

#### **Step 21.6: Update show selector placeholder**
**File**: `components/selectors/paths-expand-selector.tsx`
- Change placeholder from "Show only..." to "Show..."
- Maintain all existing functionality and styling

### Data Flow

```
Mapper Component
├── mapperType: 'result' | 'request' | 'response'
├── mappingsShortlist: FullPathData[] | VariablePathData[]
├── statistics: {success, issue, recommended, default}
└── ProgressBar
    └── ProgressHovercard
        ├── Result mapper: JsonPath components
        └── Request/Response mappers: variable:path format
```

### Expected Outcome

**Working Features:**
- ✅ Unified `mappingsShortlist` naming throughout codebase
- ✅ `mapperType` prop propagated through component hierarchy
- ✅ Different data generation functions for different mapper types
- ✅ Conditional rendering in ProgressHovercard based on mapper type
- ✅ Variable:path format display for request/response mappers
- ✅ Context-appropriate labels ("options" vs "remaining")
- ✅ Reused logic from `getVariableMappingStatistics()` for consistency
- ✅ Updated show selector placeholder: "Show..."

**Visual Improvements:**
- **Context-Aware Display**: Different formats for different mapper types
- **Variable Integration**: Proper display of variable names with their paths
- **Consistent Naming**: `mappingsShortlist` provides clearer intent than `enhancedKeys`
- **Appropriate Labels**: "remaining" for variables, "options" for JSON paths
- **Clean Placeholder**: "Show..." is more concise and clear

**Integration Benefits:**
- **Unified System**: Single codebase handles both mapping types
- **Future-Proof**: Easy to extend with additional mapper types
- **Consistent Logic**: Reused functions ensure consistent behavior
- **Clear Separation**: Different rendering logic for different contexts
- **Maintainable**: Clear naming and structure for easier maintenance

### Implementation Order

1. **Rename `enhancedKeys` to `mappingsShortlist`**: Update all interfaces and references
2. **Add `mapperType` prop**: Propagate through component hierarchy
3. **Create `VariablePathData` interface**: Define data structure for variable paths
4. **Create `getVariablePathsByStatus` function**: Reuse logic from existing functions
5. **Update mapper components**: Pass `mapperType` and call appropriate functions
6. **Update ProgressHovercard**: Add conditional rendering logic
7. **Update show selector placeholder**: Change to "Show..."

This comprehensive plan creates a unified system that can handle both object/datapoint mappings and variable mappings with appropriate rendering for each context, while maintaining backward compatibility and ensuring consistent behavior across all mapper types.

## Step 22: Clean Code Refactoring - Unified Statistics Interface

### Problem Analysis
The current implementation has redundant data structures and complex data flow:
- `VariablePathData` duplicates information already available in `FullPathData`
- `MappingsShortlist` interface creates unnecessary complexity
- Separate `statistics` and `mappingsShortlist` props create data duplication
- Functions like `getPathsByStatus` and `getVariablePathsByStatus` are redundant with existing statistics functions

### Solution Overview
Implement a clean, unified data flow through an enhanced `statistics` interface that includes both counts and mappings data. This eliminates all redundant interfaces and creates a single source of truth for progress bar and hovercard data.

### Detailed Implementation Plan

#### Step 22.1: Enhance Statistics Interface
- **New Statistics Format**:
  ```typescript
  statistics: {
    success: {
      count: number, 
      label: string, 
      mappings: Array<{path: string, variable?: string}>
    },
    issue: {
      count: number, 
      label: string, 
      mappings: Array<{path: string, variable?: string}>
    },
    recommended: {
      count: number, 
      label: string, 
      mappings: Array<{path: string, variable?: string}>
    },
    default: {
      count: number, 
      label: string, 
      mappings: Array<{path: string, variable?: string}>
    }
  }
  ```

#### Step 22.2: Modify Existing Statistics Functions
- **`getObjectAndDatapointMappingStatistics`** in `utils/json/mapping-utils.tsx`:
  - Move from `utils/json/rendering-utils.tsx` to `utils/json/mapping-utils.tsx`
  - Add `fullPaths: FullPathData[]` parameter
  - Return enhanced statistics with mappings array (ALL items, not just top 3)
  - Process all `fullPaths` by status, create mappings with `{path, variable?}`
  - **Note**: Uses "options" label for default status (not "remaining")

- **`getVariableMappingStatistics`** in `utils/json/mapping-utils.tsx`:
  - Already correctly located in mapping-utils
  - Modify to return enhanced statistics with mappings array (ALL items, not just top 3)
  - Process all variables by status, create mappings with `{path, variable}`
  - **Note**: Uses "remaining" label for default status (not "options")
  - **Critical**: Must count variables WITHOUT paths as "remaining" in default status

#### Step 22.3: Update Mappers to Generate Enhanced Statistics
- **JsonResultMapper**: Call `getObjectAndDatapointMappingStatistics(basePaths, fullPaths)` for complete statistics
- **JsonRequestMapper & JsonResponseMapper**: Call `getVariableMappingStatistics()` for complete statistics
- **Remove**: All `mappingsShortlist` generation and passing from all mappers

#### Step 22.4: Update JsonExplorer
- **Remove**: `mappingsShortlist` calculation and prop passing
- **Remove**: `getPathsByStatus` import
- **Keep**: Only pass `statistics` prop to ProgressBar

#### Step 22.5: Update ProgressBar Interface
- **Remove**: `mappingsShortlist?: MappingsShortlist` prop
- **Remove**: `MappingsShortlist` interface
- **Keep**: Only `statistics` prop (with new enhanced interface)

#### Step 22.6: Update ProgressHovercard Interface
- **Remove**: `mappingsShortlist?: MappingsShortlist` prop
- **Remove**: `MappingsShortlist` and `VariablePathData` interfaces
- **Update**: Rendering logic to use `statistics.success.mappings`, `statistics.issue.mappings`, etc.
- **Handle**: Both `path` and `variable` fields in mappings
- **Display Logic**:
  - **Result Mappers**: Show only `path` (no variable field)
  - **Request/Response Mappers**: Show `variable` as subheadline, `path` below it
  - **Remaining Variables**: Show only `variable` with "No path mapped" message
  - **Mapped Variables**: Show `variable: path` format

#### Step 22.7: Remove Redundant Code
- **Delete Functions**:
  - `getPathsByStatus` in `utils/json/rendering-utils.tsx`
  - `getVariablePathsByStatus` in `utils/json/rendering-utils.tsx`
- **Delete Interfaces**:
  - `VariablePathData` (from all files)
  - `MappingsShortlist` (from all files)
- **Remove**: All `mappingsShortlist` related code throughout codebase

### Key Benefits
- **Single source of truth**: All data flows through enhanced `statistics` interface
- **No data duplication**: Eliminates separate `mappingsShortlist` prop
- **Clean architecture**: Mappers generate complete statistics, components consume them
- **Type safety**: Simple, unified interface with clear data structure
- **Maintainability**: Fewer interfaces, cleaner data flow

### Data Flow
```
JsonResultMapper
├── getObjectAndDatapointMappingStatistics(basePaths, fullPaths)
└── statistics: {success: {count, label, mappings}, issue: {...}, ...} → JsonExplorer

JsonRequestMapper/JsonResponseMapper  
├── getVariableMappingStatistics(requestAndResponseVariableMappings, fullPaths)
└── statistics: {success: {count, label, mappings}, issue: {...}, ...} → JsonExplorer

JsonExplorer
├── Pass statistics to ProgressBar
└── ProgressBar → ProgressHovercard

ProgressHovercard
├── Use statistics.success.mappings for mapped items
├── Use statistics.issue.mappings for issue items
└── Handle both path and variable fields in mappings
```

This approach creates a clean, unified system where all progress bar and hovercard data flows through a single enhanced statistics interface, eliminating all redundant code and interfaces.

## Complete Reference List for Clean Implementation

### **FILES TO DELETE COMPLETELY**

#### 1. **No files to delete completely** - all changes are modifications

### **FILES TO MODIFY**

#### **A. UTILITY FILES**

##### **1. `utils/json/rendering-utils.tsx`**
**REMOVE:**
- `VariablePathData` interface (lines 34-38)
- `getPathsByStatus` function (lines 1240-1280)
- `getVariablePathsByStatus` function (lines 1288-1413)

**MOVE:**
- `getObjectAndDatapointMappingStatistics` function → `utils/json/mapping-utils.tsx`

##### **2. `utils/json/mapping-utils.tsx`**
**ADD:**
- `getObjectAndDatapointMappingStatistics` function (moved from rendering-utils)
- Add `fullPaths: FullPathData[]` parameter
- Return enhanced statistics with mappings array

**MODIFY:**
- `getVariableMappingStatistics` function:
  - Add mappings array to return value
  - Process ALL variables (not just top 3)
  - Return enhanced statistics format

#### **B. MAPPER COMPONENTS**

##### **3. `components/json/result/json-result-mapper.tsx`**
**MODIFY:**
- Import: Change `getObjectAndDatapointMappingStatistics` from `rendering-utils` to `mapping-utils`
- Function call: Add `fullPaths` parameter to `getObjectAndDatapointMappingStatistics(basePaths, fullPaths)`

##### **4. `components/json/request/json-request-mapper.tsx`**
**REMOVE:**
- Import: `getVariablePathsByStatus` from `rendering-utils`
- `mappingsShortlist` from `onProcessedData` interface (lines 71-76)
- `mappingsShortlist` generation (lines 285-286)
- `mappingsShortlist` passing (line 301)
- Console log references to `mappingsShortlist` (lines 288-290)

**MODIFY:**
- `onProcessedData` interface: Remove `mappingsShortlist` field
- `getVariableMappingStatistics` call: Already correct, will return enhanced statistics

##### **5. `components/json/response/json-response-mapper.tsx`**
**REMOVE:**
- Import: `getVariablePathsByStatus` from `rendering-utils`
- `mappingsShortlist` from `onProcessedData` interface (lines 71-76)
- `mappingsShortlist` generation (lines 285-286)
- `mappingsShortlist` passing (line 301)
- Console log references to `mappingsShortlist` (lines 288-290)

**MODIFY:**
- `onProcessedData` interface: Remove `mappingsShortlist` field
- `getVariableMappingStatistics` call: Already correct, will return enhanced statistics

#### **C. CORE COMPONENTS**

##### **6. `components/json/interface/json-explorer.tsx`**
**REMOVE:**
- Import: `getPathsByStatus` from `rendering-utils`
- `mappingsShortlist` calculation (lines 151-154)
- `mappingsShortlist` prop passing to ProgressBar (line 331)

**MODIFY:**
- Import: Change `getObjectAndDatapointMappingStatistics` from `rendering-utils` to `mapping-utils`
- Function call: Add `fullPaths` parameter to `getObjectAndDatapointMappingStatistics(basePaths, fullPaths)`

##### **7. `components/others/progress-bar.tsx`**
**REMOVE:**
- `MappingsShortlist` interface (lines 20-25)
- `mappingsShortlist?: MappingsShortlist` prop (line 34)
- `mappingsShortlist` parameter from function signature (line 116)
- `mappingsShortlist` passing to ProgressHovercard (line 169)

**MODIFY:**
- `ProgressBarProps` interface: Remove `mappingsShortlist` field
- Function signature: Remove `mappingsShortlist` parameter

##### **8. `components/hovercards/progress-hovercard.tsx`**
**REMOVE:**
- `VariablePathData` interface (lines 16-20)
- `MappingsShortlist` interface (lines 22-27)
- `mappingsShortlist?: MappingsShortlist` prop (line 31)
- `mappingsShortlist` parameter from function signature (line 46)
- `mappingsShortlist` usage in rendering logic (lines 53, 55, 59, 65, 71, 77)
- Type casting: `const variablePath = item as VariablePathData` (line 126)

**MODIFY:**
- `ProgressHovercardProps` interface: Remove `mappingsShortlist` field
- Function signature: Remove `mappingsShortlist` parameter
- Rendering logic: Use `statistics.success.mappings`, `statistics.issue.mappings`, etc.
- Add conditional rendering for different mapper types:
  - Result mappers: Show only `path`
  - Request/Response mappers: Show `variable` + `path`
  - Remaining variables: Show only `variable` with "No path mapped"

#### **D. SHOWCASE PAGE**

##### **9. `app/components-showcase/page.tsx`**
**NO CHANGES NEEDED** - Already passes correct props

### **DETAILED CHANGES BY COMPONENT**

#### **Interface Changes**

##### **New Enhanced Statistics Interface:**
```typescript
statistics: {
  success: {
    count: number, 
    label: string, 
    mappings: Array<{path: string, variable?: string}>
  },
  issue: {
    count: number, 
    label: string, 
    mappings: Array<{path: string, variable?: string}>
  },
  recommended: {
    count: number, 
    label: string, 
    mappings: Array<{path: string, variable?: string}>
  },
  default: {
    count: number, 
    label: string, 
    mappings: Array<{path: string, variable?: string}>
  }
}
```

##### **Removed Interfaces:**
- `VariablePathData` (from all files)
- `MappingsShortlist` (from all files)

#### **Function Changes**

##### **Moved Functions:**
- `generatePath` → Move from `rendering-utils.tsx` to `mapping-utils.tsx`
- `generateMappingFromPath` → Move from `rendering-utils.tsx` to `mapping-utils.tsx`
- `generatePathFromMapping` → Move from `rendering-utils.tsx` to `mapping-utils.tsx`
- `getObjectAndDatapointMappingStatistics` → Move from `rendering-utils.tsx` to `mapping-utils.tsx`

##### **Renamed Functions:**
- `generateParentPaths` → `getParentPaths` (in `rendering-utils.tsx`)

##### **Renamed Interfaces:**
- `EnhancedStatistics` → `MappingStatistics` in `mapping-utils.tsx` (lines 27-32)

##### **Deleted Functions:**
- `getPathsByStatus` (from rendering-utils)
- `getVariablePathsByStatus` (from rendering-utils)

#### **Import Changes**

##### **Updated Imports:**
- `components/json/result/json-result-mapper.tsx`: Change import source for `getObjectAndDatapointMappingStatistics`
- `components/json/interface/json-explorer.tsx`: Change import source for `getObjectAndDatapointMappingStatistics`
- `components/json/request/json-request-mapper.tsx`: Remove `getVariablePathsByStatus` import
- `components/json/response/json-response-mapper.tsx`: Remove `getVariablePathsByStatus` import

### **SUMMARY**
- **Total Files Modified: 8**
- **Total Lines Removed: ~200**
- **Total Interfaces Removed: 2**
- **Total Functions Removed: 2**
- **Total Functions Modified: 2**

This comprehensive cleanup will eliminate all redundant code and create a clean, unified data flow through the enhanced statistics interface!

complet
