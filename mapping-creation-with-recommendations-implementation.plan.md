<!-- 503fb37a-57af-472d-b1a1-b66d626d7329 20ac31f8-4d22-4f09-82f9-905ddfa47ba6 -->
# Mapping Creation Dialogs Implementation Plan

## Overview

Create three mapping dialogs integrated with JsonExplorer and mappers: `create-request-mapping-dialog.tsx`, `create-response-mapping-dialog.tsx`, and `create-result-mapping-dialog.tsx`. Include intelligent pattern matching with key similarity, value similarity, and AI-based recommendations.

## Architecture

### Responsibility Separation

**Dialogs:**
- Collect user input and validate
- Create mapping data structures
- Call `onSave(mappingData)` callback

**Mappers:**
- Receive mapping data via `onSave` callback
- Manage state and file persistence
- Trigger re-processing (assignMappingToFullPath, statistics)
- Expose data via ref for parent forms

### Data Flow

**Request Dialog:**
```
User Input → Dialog creates RequestAndResponseVariableMappings
→ onSave(mappings) → JsonRequestMapper updates state → Re-processes paths
```

**Response Dialog:**
```
User Input (variables + translations) → Dialog creates mappings + translations
→ onSave(mappings, translations) → JsonResponseMapper updates state → Re-processes
```

**Result Dialog:**
```
User Input (objects + datapoints + recommendations) → Dialog creates ObjectSourceMapping[] + DatapointSourceMapping[]
→ onSave(mappings) → JsonResultMapper updates state + writes to source-mappings.json → Re-processes
→ Pattern matcher auto-generates recommendations
```

### Selection State Architecture

**Data Management:**
- Mappers own `fullPaths` array (path, type, value, status, mappings)
- Mappers pass `fullPaths` to JsonExplorer via `onProcessedData` render prop
- JsonExplorer receives `fullPaths` as read-only prop

**UI State:**
- JsonExplorer manages `fullPathsStates` object (isSelected, isExpanded)
- Dialog is controlled component receiving `selectedPath` prop
- Dialog has no internal path state

**Selection Flow:**
- PathSearch/node click updates `fullPathsStates` selection
- JsonExplorer derives `selectedPath` from `fullPathsStates`
- Dialog receives `selectedPath` as prop
- PathSearch in dialog calls `onPathChange` callback
- JsonExplorer updates `fullPathsStates` and re-renders dialog

### Dialog Triggers

- "Create Mapping" button in JsonExplorer toolbar
- Keyboard shortcut: **Ctrl/Cmd+M**

## Dialog Button Logic

All dialogs follow this button pattern:

**Secondary Button:** "Save" - always visible

**Primary Button Logic:**
- Show "Save & create new" if unmapped items exist (paths, variables, datapoints)
- Show "Save & publish" if ALL conditions met:
  - No unmapped items remaining
  - Entire source setup is valid (all request/response/result mappings complete)
  - No validation errors anywhere in source configuration
- Hide primary button if:
  - No unmapped items remaining BUT source has validation errors

## Phase 1: Foundation & Shared Components

### 1.1 Update JsonExplorer Component

**File:** `components/json/interface/json-explorer.tsx`

**Implementation:**
- Add dialog state for `createMappingDialogOpen`
- Add keyboard shortcut handler (Ctrl/Cmd+M) to open mapping dialog
- Use existing selectedPath derivation from `fullPathsStates`
- Find full path data from `fullPaths` array
- Render CreateMappingDialog with selected path, onPathChange, and onSave callbacks

### 1.2 Update Selector Components Architecture

**Base Selector (`base-selector.tsx`):**
- Pure visual/styling component with no business logic
- Accepts universal data structure: `{ items: any[], recommended?: string[] }`
- Handles UI interactions, filtering, search, and display
- No knowledge of data source or transformation

**Specialized Selectors (datapoint-selector.tsx, objecttype-selector.tsx, source-variable-selector.tsx):**
- Import and transform data from JSON files or props
- Parse data structure into `{ items: any[], recommended?: string[] }` format
- Pass transformed data to BaseSelector via `dataWithRecommended` prop
- Handle domain-specific logic (filtering, recommendations)

**DatapointSelector Implementation:**
- File: `components/selectors/datapoint-selector.tsx`
- Import datapoints from `@/data/datapoints.json`
- Transform datapoints.json structure into `dataWithRecommended` format
- Map datapoint properties (id, name, description) to items array
- Include `recommendedIds` from pattern matcher in recommended array
- Pass transformed data to BaseSelector

**ObjectTypeSelector Implementation:**
- File: `components/selectors/objecttype-selector.tsx`
- Import object types from `@/data/object-types.json`
- Transform object-types.json structure into `dataWithRecommended` format
- Map object type properties (id, name, description) to items array
- Include `recommendedIds` from pattern matcher in recommended array
- Pass transformed data to BaseSelector

**SourceVariableSelector Implementation:**
- File: `components/selectors/source-variable-selector.tsx`
- Data source: Extract variable names from RequestAndResponseVariableMappings
- Accept props: existingMappings (RequestAndResponseVariableMappings), sourceId, inputType (request/response)
- Extract variables using `Object.keys(mappings.RequestResponseMappings[sourceId][inputType])`
- Transform to BaseSelector format with variable names as items
- Group by input type (Request Variables / Response Variables)
- Pass transformed data to BaseSelector via `dataWithRecommended` prop

### 1.3 Create Mapping Path Select Component

**File:** `components/json/interface/mapping-path-select.tsx`

**Features:**
- Display mode: JsonPath component (with wildcard visualization)
- Edit mode: PathSearch component (for path selection)
- "Edit" button toggles modes
- Calls `onPathChange` when user selects new path

### 1.4 Update JsonPath Component for Wildcard Visualization

**File:** `components/json/interface/json-path.tsx`

**Enhancements:**
- Detect `*` and `**` in path segments
- Display wildcard segments with special styling/badges (bg-primary/10, text-primary, rounded, xs font-semibold)
- Add info icon with tooltip for each wildcard type:
  - `*` → tooltip: "any item on this level"
  - `**` → tooltip: "any item below this level"
- Render wildcard segments inline with flex items-center gap-1

## Phase 2: Request Mapping Dialog

### 2.1 Create Request Mapping Dialog

**File:** `components/dialogs/create-request-mapping-dialog.tsx`

**Structure:** Single-step BaseDialog

**Props:**
- `open`: boolean - dialog open state
- `onOpenChange`: callback to change open state
- `selectedPath`: FullPathData | null - currently selected path
- `onPathChange`: callback for path changes
- `sourceId`: string - source identifier
- `sourceType`: 'run_request' | 'status_request' | 'delivery_request'
- `existingMappings`: RequestAndResponseVariableMappings object
- `onSave`: callback with updated RequestAndResponseVariableMappings

**Content:**
1. Path selector (shared component)
2. SourceVariableSelector (receives existingMappings, sourceId, inputType)
3. Validation (based on sourceType)
4. Status display (mapped/issue/remaining)

**Buttons:**
- Secondary: "Save"
- Primary: "Save & create new" (if unmapped exist) / "Save & publish" (if all valid)

### 2.2 Integrate with JsonRequestMapper

**File:** `components/json/request/json-request-mapper.tsx`

- Render dialog or accept as prop
- Pass necessary props from mapper state (existingMappings, sourceId)
- Handle `onSave` to update `requestAndResponseVariableMappings`
- Trigger re-processing

## Phase 3: Response Mapping Dialog

### 3.1 Create Response Mapping Dialog

**File:** `components/dialogs/create-response-mapping-dialog.tsx`

**Structure:** 2-step MultistepBaseDialog

**Props:**
- `open`: boolean - dialog open state
- `onOpenChange`: callback to change open state
- `selectedPath`: FullPathData | null - currently selected path
- `onPathChange`: callback for path changes
- `sourceId`: string - source identifier
- `existingMappings`: RequestAndResponseVariableMappings object
- `existingTranslations`: Record<string, Record<string, string>> - existing translation maps
- `onSave`: callback with updated mappings and translations

**Step 1: Variable Assignment**
- Path selector
- SourceVariableSelector (receives existingMappings, sourceId, inputType)
- canProceed: path and variable selected

**Step 2: Status Translations**
- Detect status values from selected path
- Input fields for each status translation
- Preview
- canProceed: Always (warnings only)

**Buttons:** Same as request dialog

### 3.2 Integrate with JsonResponseMapper

**File:** `components/json/response/json-response-mapper.tsx`

- Render dialog
- Handle `onSave` to update mappings and translations
- Trigger re-processing

## Phase 4: Result Mapping Dialog & Pattern Matching

### 4.1 Create Result Mapping Dialog

**File:** `components/dialogs/create-result-mapping-dialog.tsx`

**Structure:** 3-step MultistepBaseDialog

**Props:**
- `open`: boolean - dialog open state
- `onOpenChange`: callback to change open state
- `selectedPath`: FullPathData | null - currently selected path
- `onPathChange`: callback for path changes
- `sourceId`: string - source identifier
- `activityTypeId`: string - activity type for filtering
- `jsonData`: any - JSON data for pattern matching
- `fullPaths`: FullPathData[] - all paths for child detection
- `availableObjectTypes`: Array<{id, name}>
- `availableDatapoints`: Array<{id, name, object_type_id}>
- `existingMappings`: object with object_source_mappings and datapoint_source_mappings arrays
- `onSave`: callback with updated object and datapoint mappings

**Step 1: Object Assignment**
- Path selector with type (key/value) from fullPaths
- ObjectTypeSelector (with pattern-matched recommendations)
- Description field
- canProceed: Always (warnings only)

**Step 2: Datapoint Assignment**
- Path selector with type (key/value) from fullPaths
- DatapointSelector (filtered by object type, with pattern-matched recommendations)
- Key mapping type
- "Add Another Datapoint" button for multiple
- canProceed: Always (warnings only)

**Step 3: Recommended Mappings**
- Display pattern matching suggestions from all three matchers
- Checklist of recommendations with:
  - Suggested path
  - Object type or datapoint
  - Confidence score
  - Source (key similarity / value similarity / AI)
  - Checkbox to accept
- Bulk select/deselect
- "Re-run AI matcher" button for remaining unmapped paths
- canProceed: Always (optional step)

**Buttons:**
- Secondary: "Save"
- Primary: "Save & create new" / "Save & publish" (based on remaining + validation state)

### 4.2 Pattern Matcher Functions in mapping-utils.tsx

**File:** `utils/json/mapping-utils.tsx` *(add to existing file)*

**Core Architecture:**

The pattern matcher orchestrates three matching strategies in sequence:

**1. Key Similarity Matcher (`keySimilarityMatcher`):**
- Algorithm: Combination similarity scoring
  - Jaro-Winkler similarity (0.6 weight): handles typos, similar names
  - Substring contains matching (0.4 weight): handles partial matches
- Process:
  1. Get all child fullPaths under object mapping pattern (direct children only)
  2. Extract key from each fullPath (last segment)
  3. Compare to all datapoint keys using combined similarity
  4. Filter matches with score >= 0.9 (parametrized threshold)
  5. Return pairs: `{ fullPath, datapointId, score, method: 'key_similarity' }`
- Function signature: `keySimilarityMatcher(objectMapping, fullPaths, datapoints, threshold)`
- Helper function: `calculateSimilarity(str1, str2)` combines Jaro-Winkler (0.6) + Substring (0.4)

**2. Value Similarity Matcher (`valueSimilarityMatcher`):**
- Process:
  1. Get value from each fullPath (type: 'value' paths only)
  2. For each datapoint, get all its values from JSON
  3. Compare fullPath value to all datapoint values using similarity
  4. Count matching/similar values for each datapoint
  5. Rank datapoints by match count
  6. Return datapoints meeting threshold (e.g., 3+ matching values)
  7. Output: `{ fullPath, datapointId, matchCount, score, method: 'value_similarity' }`
- Function signature: `valueSimilarityMatcher(objectMapping, fullPaths, datapoints, jsonData, matchThreshold)`

**3. AI Matcher (`aiMatcher`):**
- Process:
  1. Receive fullPaths that didn't get recommendations from other matchers
  2. For each unmapped fullPath: extract key and value
  3. Call edge function: `api-mapping-suggestions`
  4. Send: `{ key, value, availableDatapoints[], availableObjectTypes[] }`
  5. AI analyzes context and returns best match
  6. Return: `{ fullPath, datapointId/objectTypeId, confidence, method: 'ai', reasoning }`
- Function signature: `aiMatcher(unmappedPaths, datapoints, objectTypes)` - returns Promise

**Main Orchestrator (`generateMappingRecommendations`):**
- Purpose: Executes matchers in sequence, tracks results
- Parameters:
  - `objectMappings`: ObjectSourceMapping[]
  - `fullPaths`: FullPathData[]
  - `datapoints`: DatapointData[]
  - `objectTypes`: ObjectTypeData[]
  - `jsonData`: any
  - `useAI`: boolean (true on initial upload, false on re-processing)
- Process:
  1. Filter to value-type paths only
  2. Run key similarity matcher, collect results, remove matched paths
  3. Run value similarity matcher on remaining paths, collect results, remove matched paths
  4. Run AI matcher on remaining paths (only if useAI is true), collect results
- Returns: recommendations array, unmappedPaths array, and statistics object with counts

**Wildcard Pattern Understanding:**
- Single `*`: Matches direct children at current level only
  - `["companies", "*", "website"]` → matches `["companies", "[0]", "website"]`, `["companies", "[1]", "website"]`
  - Goes through all array items at that level, finds the specific key
- Double `**`: Matches any nested levels (all depths)
  - `["**", "website"]` → matches ANY path ending with "website" at any depth
  - `["data", "**", "email"]` → matches `["data", "users", "email"]`, `["data", "contacts", "users", "email"]`
- **Critical Rule:** Wildcards MUST end with a specific key or array number
  - ✅ Valid: `["companies", "*", "name"]`, `["data", "**", "email"]`, `["items", "[0]"]`
  - ❌ Invalid: `["companies", "*"]`, `["data", "**"]` (wildcards cannot be the last element)
  - Reason: You must specify what you're extracting/mapping to; wildcards are for traversal patterns
- Used in `validateMappingforFullPath()` to match fullPaths to mapping patterns

**Execution Timing:**

**Flow 1: Initial JSON Upload**
```
Upload JSON → generatePathData() → Get objectsAndDatapointsMappings → 
Pattern Matcher (useAI=true) → Generate recommendations for unmapped fullPaths (type='value' only) → 
Display in Step 3
```

**Flow 2: Mapping Update (Re-processing)**
```
User saves mapping → Mapper updates objectsAndDatapointsMappings → 
Trigger re-processing → Pattern Matcher (useAI=false) → 
Re-match remaining paths without AI
```

### 4.3 Create AI Mapping Suggestions Edge Function

**File:** `edgefunctions/supabase/functions/api-mapping-suggestions/index.ts`

**Purpose:** New edge function specifically for AI-powered mapping recommendations

**Input:**
- `key`: string - the path key to match
- `value`: any - the value at the path
- `availableDatapoints`: Array with id, name, description
- `availableObjectTypes`: Array with id, name, description

**Output:**
- `recommendation`: object containing:
  - `type`: 'datapoint' | 'objecttype'
  - `id`: string - matched item ID
  - `confidence`: number (0-1)
  - `reasoning`: string - explanation for match

### 4.4 Integrate with JsonResultMapper

**File:** `components/json/response/json-result-mapper.tsx`

- Render dialog
- Pass activityTypeId, jsonData, fullPaths for pattern matching
- **Initial Upload:** Auto-run pattern matcher with `useAI=true` on JSON upload/processing
- **Re-processing:** Run pattern matcher with `useAI=false` when mappings update
- Handle `onSave` to update state + write to source-mappings.json
- Trigger re-processing
- Store pattern matcher recommendations in state for Step 3

## Phase 5: Integration & Features

### 5.1 Mapper Parent Components

**Files:** delivery-form.tsx, run-request-form.tsx, etc.

- Pass dialog props to mappers (existingMappings, sourceId)
- Handle mapping changes
- Validation integration

### 5.2 Edit Existing Mappings

**All dialogs:**
- Populate fields from existing mapping data when path has mappings
- Show "Edit Mapping" in title
- Allow modification and deletion

### 5.3 Batch Mapping Creation

**"Save & create new" behavior:**
- Save mapping
- Reset fields
- Keep dialog open
- Pre-select next unmapped path
- Show progress counter

## Phase 6: Polish

### 6.1 Validation & Feedback

- Real-time validation
- Warning badges for issues
- Tooltip explanations

### 6.2 Empty States

- No variables: Show message
- All mapped: Congratulations

### 6.3 Loading & Errors

- Loading states for pattern matching
- Error handling for AI matcher
- Optimistic updates

### 6.4 Accessibility

- Keyboard navigation
- Screen reader support
- Focus management
- ARIA labels

## Files

**New:**
- `components/dialogs/create-request-mapping-dialog.tsx`
- `components/dialogs/create-response-mapping-dialog.tsx`
- `components/dialogs/create-result-mapping-dialog.tsx`
- `components/dialogs/shared/mapping-path-select.tsx`
- `edgefunctions/supabase/functions/api-mapping-suggestions/index.ts`

**Modified:**
- `components/json/interface/json-explorer.tsx` (add dialog state, keyboard shortcut, dialog rendering)
- `components/json/interface/json-path.tsx` (add wildcard visualization with tooltips)
- `components/json/request/json-request-mapper.tsx` (integrate request dialog)
- `components/json/response/json-response-mapper.tsx` (integrate response dialog)
- `components/json/response/json-result-mapper.tsx` (integrate result dialog, pattern matcher)
- `components/selectors/base-selector.tsx` (ensure universal data support)
- `components/selectors/datapoint-selector.tsx` (add recommendation support)
- `components/selectors/objecttype-selector.tsx` (add recommendation support)
- `components/selectors/source-variable-selector.tsx` (extract from RequestAndResponseVariableMappings)
- `utils/json/mapping-utils.tsx` (add pattern matcher functions: keySimilarityMatcher, valueSimilarityMatcher, aiMatcher, generateMappingRecommendations)
- Forms using mappers (pass correct props)

### To-dos

- [ ] Phase 1: Create foundation - Update JsonExplorer, implement selector architecture (base as visual-only, specialized handle data), create path-selector shared component
- [ ] Phase 2: Implement request mapping dialog (single-step) with SourceVariableSelector integration
- [ ] Phase 3: Implement response mapping dialog (2-step with translations and SourceVariableSelector)
- [ ] Phase 4.1: Create pattern-matcher utility with keySimilarityMatcher (Jaro-Winkler 0.6 + Substring 0.4, threshold 0.9)
- [ ] Phase 4.2: Implement valueSimilarityMatcher in pattern-matcher (value comparison, match counting)
- [ ] Phase 4.3: Create api-mapping-suggestions edge function for AI-based matching
- [ ] Phase 4.4: Implement aiMatcher in pattern-matcher (calls edge function for unmapped paths)
- [ ] Phase 4.5: Implement result mapping dialog (3-step) with pattern matching integration (auto-run + manual trigger)
- [ ] Phase 4.6: Integrate result dialog with JsonResultMapper (auto-run pattern matcher on upload, store recommendations)
- [ ] Phase 5: Complete dialog integration with mappers and forms (sourceSetup passing, validation)
- [ ] Phase 6: Polish, validation feedback, loading states for pattern matching, edge cases, accessibility