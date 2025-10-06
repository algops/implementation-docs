# JSON Mappers Implementation Plan

## Overview
This document provides a comprehensive plan for implementing a unified JSON mapping architecture. The system will support three main mapping types: object/datapoint mappings (source-mappings.json), variable mappings (template-based), and request/response mappings. The architecture is designed to be flexible, reusable, and maintainable while providing clear separation of concerns.

## Current Architecture Status

### ✅ Implemented Components

#### 1. JsonResultMapper (Advanced - WORKING)
- **Purpose**: Handles complex object/datapoint source mappings from `source-mappings.json`
- **Data Structure**: `ObjectSourceMapping` and `DatapointSourceMapping` arrays
- **Mapping Logic**: Wildcard path matching with key/value distinction
- **Status**: ✅ **WORKING** - Recently updated with separate `objectMappings` and `datapointMappings` arrays
- **UUID Collision**: ✅ **FIXED** - Now properly handles same UUIDs in both object and datapoint mappings

#### 2. Core JSON Infrastructure (WORKING)
- **JsonExplorer**: Main container with search and navigation
- **JsonRenderer**: Core rendering engine with accordion support
- **JsonNode**: Individual interactive nodes with status-based styling
- **Rendering Utilities**: Complete path manipulation and rendering functions
- **Mapping Utilities**: Path matching, validation, and status determination

### ✅ Existing Components (TO BE ENHANCED)

#### 1. Template-Based Mappers (EXISTING - NEEDS ENHANCEMENT)
**Final Architecture**: 3 Mappers Total
1. **`JsonResultMapper`** - ✅ **EXISTING & WORKING** - Handles source-mappings.json (object/datapoint mappings)
2. **`JsonRequestMapper`** - ✅ **EXISTING - BASIC** - Needs enhancement for template variable mappings
3. **`JsonResponseMapper`** - ✅ **EXISTING - BASIC** - Needs enhancement for template response mappings

**Current Status**:
- **JsonResultMapper**: ✅ **COMPLETE** - Advanced mapper for source-mappings.json
- **JsonRequestMapper**: ✅ **EXISTS - BASIC** - Has basic variable mapping, needs template integration
- **JsonResponseMapper**: ✅ **EXISTS - BASIC** - Has basic response mapping, needs template integration

**Current Capabilities**:
- Basic variable mapping with `JsonRequestMapping` interface
- Basic response mapping with `JsonResponseMapping` interface
- Progress tracking and mapping summary display
- Integration with `JsonExplorer` for interactive mapping
- Sample data and mapping state management

**Required Enhancements**:
- **Template Integration**: Connect to new template JSON files (request_setup.json, etc.)
- **New Functions**: Integrate `assignVariableToFullPath()`, `assignMappingToVariable()`, `validateSetupCompletion()`
- **Enhanced Progress**: Use new validation results for detailed progress statistics
- **MappingSetup Props**: Add `MappingSetup` prop for template-based configuration

#### 3. Variable Mapping Functions
- **`assignVariableToFullPath()`**: Assigns variable mappings from template JSONs to fullPaths
- **`assignMappingToVariable()`**: Updates template JSON with new variable mappings
- **`validateTemplateCompletion()`**: Validates template completeness and correctness

## Template Structure Analysis

### Template JSON Files

#### 1. Example Template Structure (Webhook-based)
```json
{
  "35167ee5-bfd9-4221-8d47-b78096bdd0ac": {
    "name": "AI Company Researcher",
    "run_request_template": {
      "request": {
        "run_setup": ["body"],
        "webhook_url": ["webhook_url"],
        "run_id": ["id"]
      },
      "response": {
        "status": ["status"],
        "status_translations": {
          "ready": "completed",
          "processing": "running",
          "failed": "error"
        }
      }
    },
    "status_request_template": {
      "request": {},
      "response": {
        "status": ["status"],
        "status_translations": { "ready": "completed", "processing": "running", "failed": "error" }
      }
    },
    "delivery_request_template": {
      "request": {},
      "response": {
        "status": ["status"],
        "status_translations": { "ready": "completed", "processing": "running", "failed": "error" }
      }
    }
  }
}
```

#### 2. Example Template Structure (Endpoint-based)
```json
{
  "0bd5ae95-75c0-4f54-945b-91299c0c7a0b": {
    "name": "LinkedIn People Dataset",
    "run_request_template": {
      "request": {
        "run_setup": ["body", "filter", "filters"],
        "webhook_url": ["webhook_url"],
        "run_id": ["id"]
      },
      "response": {
        "status": ["status"],
        "status_translations": { "ready": "completed", "processing": "running", "failed": "error" }
      }
    },
    "status_request_template": {
      "request": { "run_external_id": ["url"] },
      "response": {
        "status": ["status"],
        "status_translations": { "ready": "completed", "processing": "running", "failed": "error" }
      }
    },
    "delivery_request_template": {
      "request": { "run_external_id": ["url"] },
      "response": {
        "status": ["status"],
        "status_translations": { "ready": "completed", "processing": "running", "failed": "error" }
      }
    }
  }
}
```

### Template Structure Patterns

#### Request Templates
- **Variable Mappings**: `["body"]`, `["webhook_url"]`, `["id"]` - JSON paths to extract for variables
- **External ID Mappings**: `["url"]` - For status/delivery requests that need external IDs
- **Multiple Path Support**: `["body", "filter", "filters"]` - Multiple JSON paths for same variable

#### Response Templates
- **Status Mappings**: `["status"]` - JSON path to extract status from response
- **Status Translations**: `{ready: "completed", processing: "running", failed: "error"}` - User-friendly labels
- **Consistent Pattern**: All templates use same status translation structure

## FullPaths Structure Extension

### Current Structure (UPDATED)
```typescript
fullPaths: Array<{
  path: string,
  type: 'key' | 'value',
  value: any | null,
  status: 'mapped' | 'recommendation' | 'issue' | 'default',
  isSelected?: boolean,
  objectMappings: string[],      // ✅ IMPLEMENTED - UUIDs from object_source_mappings
  datapointMappings: string[],   // ✅ IMPLEMENTED - UUIDs from datapoint_source_mappings
  variableMappings: string[]     // ❌ TO IMPLEMENT - Variable names like "run_setup", "webhook_url"
}>
```

### Mapping Type Distinction
- **Object Mappings**: Extract entire objects (companies, people, documents) - uses `objectMappings`
- **Datapoint Mappings**: Extract specific data points within objects - uses `datapointMappings`
- **Variable Mappings**: Map JSON paths to template variables - uses `variableMappings`

## Required Functions Design

### 1. Variable Mapping Functions

#### A. `assignVariableToFullPath()`
- **Purpose**: Assign variable mappings from template JSONs to fullPaths
- **Input**: 
  - `mappingSetup: MappingSetup` (template JSON, requestType, inputType)
  - `fullPaths` (existing fullPaths array)
- **Logic**: 
  - Extract correct template section: `templateMappings[sourceId][requestType][inputType]`
  - For each variable (run_setup, webhook_url, etc.), find matching JSON paths
  - Assign variable names to `variableMappings` array in fullPaths
  - Count total variables for progress calculation
- **Output**: 
  - Updated fullPaths with `variableMappings: string[]` populated
  - Variable count for progress bar

#### B. `assignMappingToVariable()`
- **Purpose**: Update template JSON with new variable mappings
- **Input**:
  - `mappingSetup: MappingSetup`
  - `variableName` (e.g., "run_setup", "webhook_url", "run_external_id")
  - `jsonPaths` (array of JSON paths like ["body", "filter"])
- **Logic**:
  - Navigate to correct template section: `templateMappings[sourceId][requestType][inputType]`
  - Update the variable mapping with new JSON paths
  - Handle both adding new variables and updating existing ones
- **Output**: Updated template JSON structure

#### C. `validateTemplateCompletion()`
- **Purpose**: Check if template is complete and valid
- **Input**:
  - `mappingSetup: MappingSetup`
- **Logic**:
  - **Delivery Option Validation**:
    - **Webhook-based (Priority)**: If `webhook_url` and request-mapping is mapped, template is complete
    - **Endpoint-based (Fallback)**: If `webhook_url` is NOT mapped, require the rest of `run_request_template`,  `status_request_template` and `delivery_request_template` to be mapped
  - **Variable Mapping Validation**: Check if all required variables are mapped to valid JSON paths
  - **JSON Path Validation**: Validate that mapped JSON paths exist in the data
  - **Status Translation Validation**: For response templates, validate `status_translations` are complete
- **Output**: Validation result with completion status, delivery method, and missing items
  ```typescript
  interface ValidationResult {
    isComplete: boolean
    deliveryMethod: "webhook" | "endpoint" | "incomplete"
    missingItems: string[]
    mappedVariables: string[]
    errors: string[]
  }
  ```

**Validation Logic Examples:**
- **Webhook-based Templates**: 
  - ✅ Complete if `webhook_url` is mapped (status/delivery requests are empty)
  - ❌ Incomplete if `webhook_url` is not mapped
- **Endpoint-based Templates**:
  - ✅ Complete if `webhook_url` is mapped OR both status/delivery requests are mapped
  - ❌ Incomplete if neither webhook nor endpoint requests are mapped

### 2. Parameter Structure
  ```typescript
interface MappingSetup {
  templateMappings: any,  // The template JSON containing variable mappings
  requestType: "run_request" | "status_request" | "delivery_request",
  inputType: "request" | "response"
}
```

### 3. Progress Bar Integration

#### A. ProgressBar Interface (Current)
The ProgressBar component expects statistics in this format:
```typescript
interface ProgressBarProps {
  statistics?: {
    success: {count: number, label: string},
    issue: {count: number, label: string},
    recommended: {count: number, label: string},
    default: {count: number, label: string}
  }
}
```

#### B. Statistics Translation
The `validateSetupCompletion()` function will return statistics in the correct ProgressBar format:
```typescript
progressStatistics: {
  success: { count: number, label: string },        // "Completed"
  issue: { count: number, label: string },          // "Incomplete" 
  recommended: { count: number, label: string },    // "Recommended"
  default: { count: number, label: string }         // "Remaining"
}
```

#### C. Usage Example
```tsx
const validationResult = validateSetupCompletion(mappingSetup, fullPaths)

<ProgressBar statistics={validationResult.progressStatistics} />
```

#### D. Dynamic Labels
The function will generate appropriate labels based on the setup type:
- **Request Setup**: "Completed Variables", "Incomplete Variables", etc.
- **Response Setup**: "Completed Fields", "Incomplete Fields", etc.

## Component Architecture

### 1. JsonRequestMapper (Parametrized)
- **Props**:
  - `data` (JSON data to map)
  - `mappingSetup: MappingSetup`
  - `onMappingUpdate` (callback for mapping changes)
- **Logic**:
  - Use `assignVariableToFullPath()` with `inputType: "request"`
  - Display JSON with variable mapping indicators
  - Show progress bar with variable mapping completion using `countVariableMappings()`
  - Allow editing variable mappings
  - Use `assignMappingToVariable()` to update template
- **Progress Bar Usage**:
  ```tsx
  const variableStats = countVariableMappings(mappingSetup, fullPaths)
  <ProgressBar statistics={variableStats} />
  ```

### 2. JsonResponseMapper (Parametrized)
- **Props**:
  - `data` (JSON data to map)
  - `mappingSetup: MappingSetup`
  - `onMappingUpdate` (callback for mapping changes)
- **Logic**:
  - Use `assignVariableToFullPath()` with `inputType: "response"`
  - Display JSON with variable mapping indicators
  - Show progress bar with variable mapping completion using `countVariableMappings()`
  - Allow editing variable mappings and status translations
  - Use `assignMappingToVariable()` to update template
- **Progress Bar Usage**:
  ```tsx
  const variableStats = countVariableMappings(mappingSetup, fullPaths)
  <ProgressBar statistics={variableStats} />
  ```

### 3. Usage Flow
1. **Form Selection**: User selects source, request type, and input type
2. **MappingSetup Creation**: Create `MappingSetup` object with template JSON and parameters
3. **Path Generation**: Generate fullPaths from JSON data
4. **Variable Assignment**: Use `assignVariableToFullPath()` to assign variable mappings
5. **Progress Calculation**: Use `countVariableMappings()` for progress bar
6. **Display**: Show JSON with variable mapping indicators and progress
7. **Editing**: User can edit variable mappings
8. **Update**: Use `assignMappingToVariable()` to update template
9. **Validation**: Use `validateTemplateCompletion()` to check completeness

## Implementation Plan

### Phase 1: Fix Type Inconsistencies (Critical)

#### **Issue 1: Type Inconsistencies in JsonResultMapper**

**File: `components/json/response/json-result-mapper.tsx`**

**Change 1.1: Update the FullPathData interface (lines 58-65)**
- Remove the old `mappings: string[]` field
- Add the missing `variableMappings: string[]` field
- Keep the existing `objectMappings: string[]` and `datapointMappings: string[]` fields

**Change 1.2: Update the JsonResultMapperProps interface (line 72)**
- Update the fullPaths type in the onProcessedData callback to match the new FullPathData structure
- Remove references to the old `mappings` field
- Add the new `variableMappings` field

#### **Issue 2: Missing variableMappings Field**

**File: `components/json/interface/json-explorer.tsx`**

**Change 2.1: Update the FullPathData interface (lines 42-51)**
- Add the missing `variableMappings: string[]` field
- Keep the existing `objectMappings: string[]` and `datapointMappings: string[]` fields

**File: `components/json/interface/json-renderer.tsx`**

**Change 2.2: Update the FullPathData interface (lines 45-53)**
- Add the missing `variableMappings: string[]` field
- Keep the existing `objectMappings: string[]` and `datapointMappings: string[]` fields

**Change 2.3: Update the JsonRendererProps interface (line 58)**
- Update the fullPaths type to include the new `variableMappings` field
- Remove references to the old `mappings` field

#### **Issue 3: Inconsistent Function Signatures**

**File: `utils/json/rendering-utils.tsx`**

**Change 3.1: Update generatePathData function (lines 395-433)**
- Modify the fullPaths initialization to include `variableMappings: []` for each path
- Remove any references to the old `mappings` field

**Change 3.2: Update renderKeyNode function (lines 770-794)**
- Change the fullPaths parameter type to include `variableMappings: string[]`
- Update the manual mapping count calculation to include `variableMappings` in the addition
- Keep the manual calculation approach (don't use `countMappings()`)
- Remove references to the old `mappings` field

**Change 3.3: Update renderValueNode function (lines 806-830)**
- Change the fullPaths parameter type to include `variableMappings: string[]`
- Update the manual mapping count calculation to include `variableMappings` in the addition
- Keep the manual calculation approach (don't use `countMappings()`)
- Remove references to the old `mappings` field

**Change 3.4: Update renderKeyValuePair function (lines 843-868)**
- Change the fullPaths parameter type to use the new structure with separate arrays
- Remove references to the old `mappings` field

**Change 3.5: Update renderBase function (lines 885-926)**
- Change the fullPaths parameter type to use the new structure with separate arrays
- Remove references to the old `mappings` field

**Change 3.6: Update renderNestedLevel function (lines 941-977)**
- Change the fullPaths parameter type to use the new structure with separate arrays
- Remove references to the old `mappings` field

**Change 3.7: Update renderLevelContent function (lines 989-1034)**
- Change the fullPaths parameter type to use the new structure with separate arrays
- Remove references to the old `mappings` field

**Change 3.8: Update renderPrimitiveContent function (lines 1083-1169)**
- Change the fullPaths parameter type to use the new structure with separate arrays
- Remove references to the old `mappings` field

**Change 3.9: Update determineHeaderContent function (lines 1045-1073)**
- Change the fullPaths parameter type to use the new structure with separate arrays
- Remove references to the old `mappings` field

**Change 3.10: Update updateBasePathStatistics function (lines 657-753)**
- The function signature is already correct, but verify it handles the new structure properly
- Ensure it counts from both objectMappings and datapointMappings arrays

**Change 3.11: Update countMappings function in mapping-utils.tsx (lines 179-240)**
- Add `variableMappings: string[]` to the fullPaths parameter type
- Include `variableMappings` in the collection of all mapping IDs (alongside objectMappings and datapointMappings)
- The deduplication and counting logic remains exactly the same

#### **Summary of Phase 1 Changes**

**Total Files to Modify: 5**
- `components/json/response/json-result-mapper.tsx` (2 changes)
- `components/json/interface/json-explorer.tsx` (1 change)
- `components/json/interface/json-renderer.tsx` (2 changes)
- `utils/json/rendering-utils.tsx` (10 changes)
- `utils/json/mapping-utils.tsx` (1 change)

**Total Changes: 16**

**What This Accomplishes:**
1. **Fixes type inconsistencies** - All interfaces will match the actual data structure
2. **Adds missing variableMappings field** - Enables template-based functionality
3. **Updates all function signatures** - All rendering functions will work with the new structure
4. **Eliminates compilation errors** - TypeScript will no longer complain about type mismatches
5. **Prepares for template functionality** - The foundation will be ready for Phase 2
6. **Unifies mapping counting** - All functions use the same `countMappings()` logic

### Phase 2: Create Variable Mapping Functions

**Goal**: Create specialized functions for template-based variable mapping and validation.

#### **2.1 Template JSON Structures**

**Note**: Each request type will have separate JSON files with both request and response sections.

##### **`request_setup.json`**
  ```typescript
{
  "35167ee5-bfd9-4221-8d47-b78096bdd0ac": {
    "name": "AI Company Researcher",
    "request": {
      "run_setup": {
        path: ["body"],                    // existing path mapping
        status: "mapped"                   // NEW: string status
      },
      "webhook_url": {
        path: ["webhook_url"],             // existing path mapping  
        status: "issue"                    // NEW: string status
      },
      "run_id": {
        path: ["id"],                      // existing path mapping
        status: "remaining"                // NEW: string status
      }
    },
    "response": {
      "status": {
        path: ["status"],                  // existing path mapping
        status: "mapped",                  // NEW: string status
        translations: {                    // existing translations (no individual statuses)
          "ready": "completed",
          "processing": "running", 
          "failed": "error"
        }
      }
    }
  }
}
```

##### **`status_check_setup.json`**
  ```typescript
{
  "35167ee5-bfd9-4221-8d47-b78096bdd0ac": {
    "name": "AI Company Researcher", 
    "request": {
      "run_external_id": {
        path: ["run_external_id"],         // NEW: status check variables
        status: "mapped"
      },
      "check_type": {
        path: ["type"],
        status: "remaining"
      }
    },
    "response": {
      "status": {
        path: ["status"],
        status: "mapped",
        translations: {
          "ready": "completed",
          "processing": "running", 
          "failed": "error"
        }
      },
      "progress": {
        path: ["progress"],
        status: "issue",
        translations: {
          "0": "not_started",
          "50": "in_progress",
          "100": "completed"
        }
      }
    }
  }
}
```

##### **`delivery_setup.json`**
```typescript
{
  "35167ee5-bfd9-4221-8d47-b78096bdd0ac": {
    "name": "AI Company Researcher",
    "request": {
      "run_external_id": {
        path: ["run_external_id"],         // NEW: delivery variables
        status: "mapped"
      },
      "delivery_method": {
        path: ["method"],
        status: "remaining"
      },
      "priority": {
        path: ["priority"],
        status: "issue"
      }
    },
    "response": {
      "status": {
        path: ["status"],
        status: "mapped",
        translations: {
          "ready": "completed",
          "processing": "running", 
          "failed": "error"
        }
      },
      "delivery_status": {
        path: ["delivery_status"],
        status: "remaining",
        translations: {
          "sent": "delivered",
          "pending": "waiting",
          "failed": "error"
        }
      }
    }
  }
}
```

##### **Updated MappingSetup Interface**
```typescript
interface MappingSetup {
  templateMappings: {
    [sourceId: string]: {
      name: string,
      request?: {
        [variableName: string]: {
          path: string[],
          status: string  // "mapped" | "issue" | "remaining"
        }
      },
      response?: {
        [variableName: string]: {
          path: string[],
          status: string,  // "mapped" | "issue" | "remaining"
          translations?: { [key: string]: string }  // no individual statuses
        }
      }
    }
  },
  requestType: "run_request" | "status_request" | "delivery_request",
  inputType: "request" | "response"
}
```

#### **2.2 Dynamic Function Requirements**

**All functions must work dynamically with:**
- **Any request type**: `run_request`, `status_request`, `delivery_request`
- **Any variable names**: Functions should iterate through all variables in the template
- **Any source ID**: Functions should work with any source ID in the template

**⚠️ CRITICAL: No Hardcoded Variable Names**
- **Functions must NOT hardcode any variable names** (e.g., "run_setup", "webhook_url", "run_external_id")
- **Functions must work with the JSON structure only** - iterate through all variables dynamically
- **The JSON structure will remain consistent** - functions should rely on this structure, not specific variable names
- **Variable names can change** - functions must adapt to any variable names in the template

#### **Function 1: `assignVariableToFullPath()`**
**Purpose**: Assign variable mappings from template JSONs to fullPaths
**Input**: `mappingSetup`, `fullPaths`
**Logic**:
- Extract template section: `templateMappings[sourceId][inputType]` (dynamic based on requestType)
- For each variable in the template section (dynamic iteration):
  - Use `generatePathFromMapping()` to convert template paths in arrays to fullPath format
  - Find matching fullPaths entries where `type === "value"`
  - Match using exact path comparison
  - Assign variable names to `variableMappings` array
**Output**: Updated fullPaths with `variableMappings` populated

**Dynamic Requirements**:
- Works with any request type (run_request, status_request, delivery_request)
- Iterates through all variables in the template section dynamically
- Handles missing request/response sections gracefully
- **NO hardcoded variable names** - works with any variable names in the JSON structure

#### **Function 2: `getRequestVariableMappingStatus()`**
**Purpose**: Check request variable mapping existence and set statuses in BOTH fullPaths and mappingSetup
**Input**: `mappingSetup`, `fullPaths`
**Logic**:
- Extract template section: `templateMappings[sourceId][inputType]` (dynamic based on requestType)
- For each variable in template (dynamic iteration):
  - Check if variable has not null in array
  - If NOT exist: set status to "remaining" in mappingSetup only (not fullPath because we don't know which to assign)
  - If exist: call `assignVariableToFullPath()` to generate path from mapping
  - If exactly ONE fullPath matched: set status to "mapped" in both
  - If >1 or 0 fullPaths matched but path exists in mapping: set status to "issue" everywhere
**Output**: Updated fullPaths + mappingSetup with statuses

**Status Values**:
- `"mapped"` - Variable has path mapping AND exactly one fullPath match
- `"issue"` - Variable has path mapping BUT multiple/no fullPath matches  
- `"remaining"` - Variable has no path mapping

**Dynamic Requirements**:
- Works with any request type and any variable names
- Sets statuses for all variables in the template
- **NO hardcoded variable names** - works with any variable names in the JSON structure

#### **Function 3: `getResponseVariableMappingStatus()`**
**Purpose**: Check response variable mapping existence, set statuses, and validate translations
**Input**: `mappingSetup`, `fullPaths`
**Logic**:
- Duplicated logic from `getRequestVariableMappingStatus()` (dynamic iteration)
- Additional step: For response keys, check if all translations have value (not NULL or empty value of the key)
- If path mapping exists AND translations complete: set status to "mapped"
- If path mapping exists but translations incomplete or translations complete but mapping is missing: set status to "issue" to mappingSetup and additionally also the fullPath (if any matches, thus we know it)
- If no path mapping and no translations (all are null or empty): set status to "remaining" in mapping setup only (not fullPath because we don't know which to assign)
**Output**: Updated fullPaths + mappingSetup with statuses

**Status Values**:
- `"mapped"` - Variable has path mapping AND exactly one fullPath match AND all translations complete
- `"issue"` - Variable has path mapping BUT multiple/no fullPath matches OR incomplete translations
- `"remaining"` - Variable has no path mapping

**Dynamic Requirements**:
- Works with any request type and any response variable names
- Validates translations for all response variables dynamically
- **NO hardcoded variable names** - works with any variable names in the JSON structure

#### **Function 4: `assignMappingToVariable()`**
**Purpose**: Update template JSON with new variable mappings
**Input**: `mappingSetup`, `variableName`, `jsonPaths`
**Logic**:
- Navigate to correct template section (dynamic based on requestType and inputType)
- Use `generateMappingFromPath()` to convert paths
- Update variable mapping (works with any variable name)
**Output**: Updated mappingSetup

**Dynamic Requirements**:
- Works with any request type and any variable name
- Handles missing request/response sections gracefully
- Updates any variable in the template structure
- **NO hardcoded variable names** - works with any variable names in the JSON structure

#### **Function 5: `getVariableMappingStatistics()`**
**Purpose**: Convert mappingSetup to ProgressBar format (works alongside `getObjectAndDatapointMappingStatistics()` below)
**Input**: `mappingSetup`
**Logic**:
- Count variables by status from mappingSetup (dynamic iteration through all variables),
- Map statuses: "mapped" → success, "issue" → issue, "recommended" → recommended, "remaining" → default
- Generate labels using `getCountWithSingularPlural()`
- Return ProgressBar format
**Output**: `{success: {count, Mappings}, issue: {count, Issues}, recommended: {count, Recommendations}, default: {count, Remaining}}`

**Status Mapping**:
- `"mapped"` → `success` (completed mappings)
- `"issue"` → `issue` (problematic mappings)
- `"recommended"` → `recommended` (recommended mappings - handled by future functions)
- `"remaining"` → `default` (unmapped variables)

**Dynamic Requirements**:
- Works with any request type and any variable structure
- Counts all variables in the template section dynamically
- Handles missing request/response sections gracefully
- **NO hardcoded variable names** - works with any variable names in the JSON structure

**Note**: All Phase 2 functions should be implemented in `utils/json/mapping-utils.tsx`

### Phase 3: Function Renames and JsonResultMapper Updates

**Goal**: Rename functions for clarity, update JsonResultMapper prop naming, and integrate statistics calculation.

#### **4.1 JsonResultMapper Statistics Integration**

**Goal**: Add statistics calculation to JsonResultMapper and update JsonExplorer to use pre-calculated statistics.

##### **JsonResultMapper Changes**
- **Add statistics calculation**: Call `getObjectAndDatapointMappingStatistics(basePaths)` after step 4
- **Update onProcessedData callback**: Add `statistics` to the returned object
- **Update interface**: Add `statistics` to `JsonResultMapperProps.onProcessedData` return type

##### **JsonExplorer Changes**
- **Add statistics prop**: `statistics?: {success: {count: number, label: string}, issue: {count: number, label: string}, recommended: {count: number, label: string}, default: {count: number, label: string}}`
- **Update ProgressBar call**: Pass `statistics` prop instead of calling `getProgressStatisticsFromBasePaths(basePaths)`
- **Maintain backward compatibility**: Fall back to `getProgressStatisticsFromBasePaths(basePaths)` if `statistics` not provided

#### **4.2 Function Renames**

#### **Function Renames Required**

##### **1. `determineBasePathStatuses` → `getBasePathStatus`**
**Current References**:
- **Export**: `utils/json/mapping-utils.tsx:474`
- **Import**: `components/json/response/json-result-mapper.tsx:8`
- **Usage**: `components/json/response/json-result-mapper.tsx:197`
- **Documentation**: 25 references in implementation docs

##### **2. `updateBasePathStatistics` → `getBasePathStatistics`**
**Current References**:
- **Export**: `utils/json/rendering-utils.tsx:658`
- **Import**: `components/json/response/json-result-mapper.tsx:10`
- **Usage**: `components/json/response/json-result-mapper.tsx:198`
- **Documentation**: 18 references in implementation docs

##### **3. `mappings` prop → `objectsAndDatapointsMappings`**
**Current References**:
- **Interface**: `components/json/response/json-result-mapper.tsx:71`
- **Usage**: `components/json/response/json-result-mapper.tsx:89, 156, 183, 264`
- **Components-showcase**: `app/components-showcase/page.tsx:1778, 1820`
- **Sources page**: `app/sources/[id]/page.tsx:70`
- **Documentation**: 41 references across implementation docs

##### **4. `getProgressStatisticsFromBasePaths` → `getObjectAndDatapointMappingStatistics`**
**Current References**:
- **Export**: `utils/json/rendering-utils.tsx:1178`
- **Import**: `components/json/interface/json-explorer.tsx:12`
- **Usage**: `components/json/interface/json-explorer.tsx:280`
- **Documentation**: 4 references in implementation docs

#### **Key Benefits of Corrected Approach**
- ✅ **Proper sequence**: Check mapping existence first, then assign if exists
- ✅ **Correct status logic**: "remaining" for missing, "mapped" for exact match, "issue" for multiple/no matches
- ✅ **Translation validation**: Response mappers check translations completeness
- ✅ **Status consistency**: Both fullPaths and mappingSetup get updated statuses
- ✅ **Maximum code reuse**: Same processing pipeline as JsonResultMapper
- ✅ **Same UX**: ProgressBar works identically with different data source
- ✅ **Clear naming**: Function names reflect their actual purpose
- ✅ **Consistent prop naming**: `objectsAndDatapointsMappings` clearly indicates the data type

**Phase 3 Implementation Changes** (JsonResultMapper & JsonExplorer):

##### **1. JsonResultMapper Changes**
- **Add statistics calculation**: Call `getObjectAndDatapointMappingStatistics(basePaths)` after step 4
- **Update onProcessedData callback**: Add `statistics` to the returned object
- **Update interface**: Add `statistics` to `JsonResultMapperProps.onProcessedData` return type

##### **2. JsonExplorer Changes**
- **Add statistics prop**: `statistics?: {success: {count: number, label: string}, issue: {count: number, label: string}, recommended: {count: number, label: string}, default: {count: number, label: string}}`
- **Update ProgressBar call**: Pass `statistics` prop instead of calling `getProgressStatisticsFromBasePaths(basePaths)`
- **Maintain backward compatibility**: Fall back to `getProgressStatisticsFromBasePaths(basePaths)` if `statistics` not provided

##### **3. ProgressBar Changes**
- **No changes needed**: Already supports both `statistics` and `basePaths` props (lines 124-129)

**Reason**: Cleaner separation of concerns - mappers handle their own statistics calculation, JsonExplorer just displays them


### Phase 4: Create JsonRequestMapper and JsonResponseMapper

**Goal**: Create template-based mappers that inherit from JsonResultMapper architecture but load template JSON instead of source-mappings.json.

#### **JsonRequestMapper Component**
**Architecture**: Copy JsonResultMapper processing pipeline
**Props**:
  ```typescript
interface JsonRequestMapperProps {
  data: any
  mappingSetup: MappingSetup
  onProcessedData?: (processedData: {
    basePaths: Record<string, BasePathData>
    fullPaths: Array<FullPathData>
    dataRendered: boolean
    onPathSelect: (path: string | undefined, parentPaths?: string[], type?: 'key' | 'value') => void
  }) => React.ReactNode
}
```

**Processing Pipeline** (inherited from JsonResultMapper):
1. `generatePathData(data)` → Creates basePaths and fullPaths
2. `getRequestVariableMappingStatus(mappingSetup, fullPaths)` → Checks mappings and sets statuses in both
3. `getBasePathStatus(basePaths, fullPaths)` → Updates basePath statuses - **SAME**
4. `getBasePathStatistics(basePaths, fullPaths)` → Calculates statistics - **SAME**
5. `getVariableMappingStatistics(mappingSetup)` → Calculates ProgressBar statistics - **NEW**
6. Pass `{basePaths, fullPaths, dataRendered, onPathSelect, statistics}` to JsonExplorer via `onProcessedData` callback

**Statistics Integration**:
- **Calculate statistics**: Call `getVariableMappingStatistics(mappingSetup)` after step 4
- **Pass to JsonExplorer**: Include `statistics` in `onProcessedData` callback return object
- **Update interface**: Add `statistics` to mapper props `onProcessedData` return type

**Key Differences from JsonResultMapper**:
- Uses `mappingSetup` prop instead of `objectsAndDatapointsMappings` prop
- Uses `getRequestVariableMappingStatus()` instead of `assignMappingsToFullPaths()`
- Same basePaths/fullPaths system for UI
- Same `getBasePathStatus()` and `getBasePathStatistics()`
- Additional step: `getVariableMappingStatistics(mappingSetup)` for ProgressBar

#### **JsonResponseMapper Component**
**Architecture**: Same as JsonRequestMapper but for response templates
**Props**: Same as JsonRequestMapper
**Processing Pipeline**: Same as JsonRequestMapper but uses `getResponseVariableMappingStatus()`
**Key Differences**: Works with response templates and validates translations

#### **JsonExplorer Integration**

**Current Architecture Analysis**:
- **JsonResultMapper**: Processes data → calls `onProcessedData` with `{basePaths, fullPaths, dataRendered, onPathSelect}`
- **JsonExplorer**: Receives props → calls `getProgressStatisticsFromBasePaths(basePaths)` internally (line 280)
- **ProgressBar**: Receives statistics via `statistics` prop OR calculates from `basePaths` prop (lines 124-129)

**Updated Architecture**:
- **JsonResultMapper**: 
  - Calculates statistics using `getObjectAndDatapointMappingStatistics(basePaths)` (renamed from `getProgressStatisticsFromBasePaths`)
  - Passes `{basePaths, fullPaths, dataRendered, onPathSelect, statistics}` to `onProcessedData`
  
- **Template Mappers**: 
  - Calculates statistics using `getVariableMappingStatistics(mappingSetup)`
  - Passes `{basePaths, fullPaths, dataRendered, onPathSelect, statistics}` to `onProcessedData`

- **JsonExplorer**: 
  - Receives `statistics` prop from mapper
  - Passes `statistics` directly to ProgressBar (no internal calculation)
  - Falls back to `basePaths` calculation if `statistics` not provided (backward compatibility)

**Phase 4 Implementation Changes** (Template Mappers):

##### **Template Mappers Changes**  
- **Add statistics calculation**: Call `getVariableMappingStatistics(mappingSetup)` after step 4
- **Update onProcessedData callback**: Add `statistics` to the returned object
- **Update interface**: Add `statistics` to mapper props `onProcessedData` return type


## LocalStorage Integration for JSON Mappers

### Overview
The JSON mappers can leverage the comprehensive localStorage system planned in `local-storage-implementation-plan.md` to provide persistent mapping sessions, template management, and seamless data recovery. This integration addresses both mapper-specific needs and broader form persistence requirements.

### Current Mapper Limitations
- **No File Persistence**: Cannot save uploaded JSON files between sessions
- **No Mapping Recovery**: Mapping progress is lost on page refresh
- **No Template System**: Cannot save and reuse mapping configurations
- **No Session Management**: Cannot resume interrupted mapping sessions
- **Static Data Only**: Limited to hardcoded data files

### LocalStorage Integration Options

#### **Option 1: JSON File Management Integration**

**Storage Keys Pattern:**
```typescript
// File management (from localStorage plan)
json_file_${fileId}           // Complete file data with metadata
json_files_list              // Array of all available file IDs
current_json_source          // Currently active file ID
json_files_history           // Chronological file access

// Mapper-specific extensions
mapper_file_${fileId}        // File-specific mapping configurations
mapper_file_metadata_${fileId} // File metadata for mappers
```

**Features:**
- **File Upload Persistence**: Save uploaded JSON files to localStorage
- **File Switching**: Switch between different JSON sources seamlessly
- **File History**: Track which files have been used for mapping
- **File Metadata**: Store file properties, size, upload date for mappers
- **File Recovery**: Restore files after browser restart

**Integration Points:**
- **JsonResultMapper**: Use stored files instead of static `require()` calls
- **JsonRequestMapper/JsonResponseMapper**: Load template files from localStorage
- **JsonExplorer**: Display available files in file selector
- **File Upload Dialog**: Save uploaded files to localStorage

#### **Option 2: Mapping Session Persistence**

**Storage Keys Pattern:**
```typescript
// Mapping sessions (from localStorage plan)
mapping_draft_${mapperId}_${draftId}  // Individual mapping draft
active_mappings                       // Currently open mapping sessions
mapping_templates                     // Reusable mapping configurations
mapping_history                       // Historical mapping activity

// Mapper-specific extensions
mapper_session_${sessionId}           // Complete mapping session state
mapper_progress_${sessionId}          // Mapping progress and statistics
mapper_selections_${sessionId}        // User selections and mappings
```

**Features:**
- **Session Recovery**: Restore mapping sessions after browser restart
- **Progress Tracking**: Save mapping progress and allow resumption
- **Selection Persistence**: Remember user's mapping selections
- **Statistics Persistence**: Save calculated mapping statistics
- **Auto-Save**: Automatically save mapping progress during work

**Integration Points:**
- **All Mappers**: Save current mapping state periodically
- **JsonExplorer**: Restore expansion state and selected paths
- **ProgressBar**: Restore calculated statistics
- **PathSearch**: Remember search queries and selections

#### **Option 3: Template Management System**

**Storage Keys Pattern:**
```typescript
// Template management
mapping_templates                     // Reusable mapping configurations
template_${templateId}                // Individual template data
template_metadata_${templateId}       // Template metadata and descriptions
user_templates                        // User-created templates
system_templates                      // Pre-built system templates

// Mapper-specific extensions
request_template_${templateId}        // Request mapping templates
response_template_${templateId}       // Response mapping templates
source_template_${templateId}         // Source mapping templates
```

**Features:**
- **Template Creation**: Save successful mapping configurations as templates
- **Template Reuse**: Apply templates to new mapping sessions
- **Template Sharing**: Share templates between users
- **Template Versioning**: Track template changes and versions
- **Template Categories**: Organize templates by type and purpose

**Integration Points:**
- **JsonRequestMapper**: Save/load request mapping templates
- **JsonResponseMapper**: Save/load response mapping templates
- **JsonResultMapper**: Save/load source mapping templates
- **Template Selector**: UI for choosing and managing templates

#### **Option 4: Form Integration for Mapper Dialogs**

**Storage Keys Pattern:**
```typescript
// Form drafts (from localStorage plan)
form_draft_${formId}_${draftId}       // Individual draft data
form_drafts_${formId}                 // List of all drafts for specific form
active_form_drafts                    // Currently open draft forms
draft_history                         // Chronological draft activity

// Mapper-specific form extensions
mapper_form_draft_${formId}_${draftId} // Mapper-specific form drafts
mapper_dialog_state_${dialogId}       // Dialog state for mapper dialogs
mapper_form_auto_save_${formId}       // Auto-save data for mapper forms
```

**Features:**
- **Dialog Persistence**: Save mapper dialog state and content
- **Form Auto-Save**: Auto-save mapper form data during input
- **Draft Recovery**: Restore unsaved mapper form data
- **Multi-Draft Support**: Manage multiple mapping drafts
- **Form Validation**: Persist form validation state

**Integration Points:**
- **ManageJsonsDialog**: Persist file management dialog state
- **JsonSelector**: Save selector state and selections
- **PathsExpandSelector**: Remember expansion preferences
- **Mapping Forms**: Auto-save mapping configuration forms

#### **Option 5: Event System Integration**

**Custom Events for Mappers:**
```typescript
// File events
jsonFilesUpdated                    // Files list changed
currentJsonSourceChanged           // Active file switched
fileUploaded                       // New file uploaded
fileRemoved                        // File deleted

// Mapper events
mappingDraftSaved                  // Mapping draft saved
mappingProgressUpdated             // Mapping progress changed
mappingCompleted                   // Mapping session completed
mappingTemplateSaved               // Template saved
mappingSessionRestored             // Session restored

// Form events
mapperFormDraftSaved               // Mapper form draft saved
mapperFormAutoSaved                // Mapper form auto-saved
mapperDialogStateChanged           // Mapper dialog state changed
```

**Features:**
- **Real-Time Updates**: Update UI when mapping data changes
- **Cross-Component Communication**: Share data between mapper components
- **State Synchronization**: Keep all mapper components in sync
- **User Notifications**: Inform users about mapping progress and changes

### Implementation Strategy

#### **Phase 1: Core Mapper Persistence (High Priority)**
1. **File Management**: Integrate JSON file upload and storage
2. **Session Recovery**: Save and restore mapping sessions
3. **Progress Tracking**: Persist mapping progress and statistics
4. **Basic Auto-Save**: Auto-save mapping state during work

#### **Phase 2: Template System (Medium Priority)**
1. **Template Creation**: Save mapping configurations as templates
2. **Template Management**: UI for managing templates
3. **Template Application**: Apply templates to new sessions
4. **Template Sharing**: Share templates between users

#### **Phase 3: Advanced Form Integration (Medium Priority)**
1. **Dialog Persistence**: Save mapper dialog state
2. **Form Auto-Save**: Auto-save mapper forms
3. **Draft Management**: Manage multiple mapping drafts
4. **Recovery UI**: User interface for recovering lost work

#### **Phase 4: Event System and Real-Time Updates (Low Priority)**
1. **Event System**: Implement comprehensive event system
2. **Real-Time Sync**: Keep components synchronized
3. **User Notifications**: Notify users of important changes
4. **Advanced Features**: Collaboration and sharing features

### Data Structure Design for Mappers

#### **Mapping Session Structure:**
  ```typescript
interface MappingSession {
  id: string
  type: 'source' | 'request' | 'response'
  fileId: string
  templateId?: string
  createdAt: Date
  lastModified: Date
  progress: {
    mapped: number
    recommendations: number
    issues: number
    options: number
  }
  state: {
    basePaths: Record<string, BasePathData>
    fullPaths: Array<FullPathData>
    expandedPaths: Set<string>
    selectedPath?: string
  }
  mappings: {
    objectMappings: string[]
    datapointMappings: string[]
    variableMappings: string[]
  }
  metadata: {
    name: string
    description?: string
    tags: string[]
    isTemplate: boolean
  }
}
```

#### **Template Structure:**
  ```typescript
interface MappingTemplate {
  id: string
  name: string
  description: string
  type: 'source' | 'request' | 'response'
  category: string
  tags: string[]
  createdAt: Date
  lastUsed: Date
  usageCount: number
  configuration: {
    mappingSetup: MappingSetup
    defaultExpansion: ExpansionOption
    customLabels?: Record<string, string>
  }
  isPublic: boolean
  authorId: string
}
```

### Benefits of LocalStorage Integration

#### **For Mappers:**
- **Session Continuity**: Resume mapping work after browser restart
- **File Management**: Upload and manage multiple JSON files
- **Template System**: Save and reuse successful mapping configurations
- **Progress Tracking**: Never lose mapping progress
- **Auto-Save**: Automatic saving prevents data loss

#### **For Forms:**
- **Draft Management**: Save incomplete form data
- **Auto-Save**: Automatic saving during form input
- **Recovery System**: Restore lost form data
- **Multi-Draft Support**: Work on multiple drafts simultaneously
- **Form State Persistence**: Remember form state across sessions

#### **For Users:**
- **Seamless Experience**: No data loss during work
- **Flexibility**: Work with multiple files and configurations
- **Efficiency**: Reuse successful mapping patterns
- **Reliability**: Robust data persistence and recovery
- **Collaboration**: Share templates and configurations

### Integration with Existing Systems

#### **LocalStorageManager Extension:**
  ```typescript
// Add mapper-specific functions to LocalStorageManager
class LocalStorageManager {
  // Existing functions...
  
  // Mapper-specific functions
  saveMappingSession(session: MappingSession): void
  getMappingSession(sessionId: string): MappingSession | null
  getAllMappingSessions(): MappingSession[]
  saveMappingTemplate(template: MappingTemplate): void
  getMappingTemplate(templateId: string): MappingTemplate | null
  getAllMappingTemplates(): MappingTemplate[]
  saveMapperFormDraft(formId: string, draft: any): void
  getMapperFormDraft(formId: string): any | null
}
```

#### **Event System Integration:**
  ```typescript
// Extend existing event system
const mapperEvents = {
  MAPPING_SESSION_SAVED: 'mappingSessionSaved',
  MAPPING_SESSION_LOADED: 'mappingSessionLoaded',
  MAPPING_TEMPLATE_SAVED: 'mappingTemplateSaved',
  MAPPING_TEMPLATE_APPLIED: 'mappingTemplateApplied',
  MAPPER_FORM_AUTO_SAVED: 'mapperFormAutoSaved',
  MAPPER_DIALOG_STATE_CHANGED: 'mapperDialogStateChanged'
}
```

This comprehensive localStorage integration provides the JSON mappers with robust persistence capabilities while supporting the broader form and dialog persistence needs of the application.


### Phase 4: Integration and Testing
1. **Test with real template data** - Use example template structures
2. **Validate progress calculations** - Ensure accurate variable counting
3. **Test template updates** - Verify `assignMappingToVariable()` works correctly
4. **Test validation** - Ensure `validateTemplateCompletion()` catches issues

## Key Benefits

### 1. Unified Architecture
- **Only 3 components**: JsonResultMapper (existing) + JsonRequestMapper (new) + JsonResponseMapper (new)
- **Single JSON structure**: All mappers work with template JSON files
- **No hardcoded formats**: No `{{variableName}}` patterns, uses template-based extraction

### 2. Template-Driven Approach
- **Dynamic Configuration**: Mappers adapt based on sourceId and templateType
- **Flexible Patterns**: Support for `["body"]`, `["webhook_url"]`, `["status"]` patterns
- **Status Translation**: Built-in translation system for response values

### 3. Easy Extension
- **New Sources**: Add new data sources through JSON configuration
- **New Templates**: Add new template types without code changes
- **Consistent Behavior**: All mappers use same underlying JsonExplorer system

### 4. Progress Tracking
- **Variable Counting**: Clear progress indication for variable mapping completion
- **Real-time Updates**: Progress updates as mappings are created/modified
- **Validation**: Ensures template completeness before submission

## Current State Assessment

### ✅ What's Working
- **JsonResultMapper**: Fully functional with separate `objectMappings` and `datapointMappings` arrays
- **Core JSON Infrastructure**: JsonExplorer, JsonRenderer, JsonNode all working
- **Rendering Utilities**: Complete path manipulation and rendering functions
- **Mapping Utilities**: Path matching, validation, and status determination
- **UUID Collision Handling**: Fixed to properly handle same UUIDs in different mapping types
- **Progress Tracking**: Working progress bar system
- **Status System**: 4 core statuses with visual feedback

### ❌ What's Missing
- **Variable Mappings**: `variableMappings: string[]` field in fullPaths structure
- **Template-Based Mappers**: JsonRequestMapper and JsonResponseMapper for template JSONs
- **Variable Mapping Functions**: `assignVariableToFullPath()`, `assignMappingToVariable()`, `validateTemplateCompletion()`
- **Template Integration**: Support for template-based variable mapping structures
- **Variable Progress Tracking**: Progress calculation for variable mapping completion

### 🔧 What Needs Implementation
1. **Extend fullPaths structure** to include `variableMappings: string[]`
2. **Create variable mapping functions** for template-based mappings
3. **Create template-based mappers** that work with template JSON structures
4. **Update existing functions** to handle `variableMappings` field
5. **Add progress tracking** for variable mapping completion
6. **Test integration** with real template data

## Summary

This implementation plan provides a comprehensive roadmap for creating a unified JSON mapping architecture that supports three main mapping types:

1. **Object/Datapoint Mappings** (source-mappings.json) - ✅ **IMPLEMENTED**
2. **Variable Mappings** (template-based) - ❌ **TO IMPLEMENT**
3. **Request/Response Mappings** (template-based) - ❌ **TO IMPLEMENT**

The architecture is designed to be:
- **Unified**: Only 3 components total
- **Flexible**: Template-driven approach with configuration
- **Reusable**: Same underlying JsonExplorer system
- **Maintainable**: Clear separation of concerns
- **Extensible**: Easy to add new sources and templates

The next steps are to implement the variable mapping functions and create the template-based mappers that will work with template JSON structures.

## Phase 5: Manage-Mapping-Dialog Implementation

**Goal**: Create a comprehensive dialog system for managing mappings with two-step workflow: mapping discovery/configuration and mapping duplication/scaling.

### 5.1 System Understanding

Based on analysis of the current mapping system, we have:

**Three Mapping Types:**
- **Object/Datapoint Mappings** (source-mappings.json) - Complex wildcard patterns with UUIDs
- **Variable Mappings** (template-based) - Simple path arrays for request/response variables  
- **Request/Response Mappings** (template-based) - Nested structure with translations

**Path System:**
- **JSON Paths**: Full paths like `"api_response.data.companies.0.website:value"`
- **Mapping Paths**: Arrays like `["api_response", "data", "companies", "*", "website"]`
- **Wildcard Support**: `*` (single level) and `**` (recursive) patterns

**Current Utilities:**
- `generatePathFromMapping()` - Convert mapping array to JSON path
- `generateMappingFromPath()` - Convert JSON path to mapping array
- `validateMappingforFullPath()` - Check if mapping matches JSON path
- `assignMappingsToFullPaths()` - Assign mappings to fullPaths
- `assignMappingToVariable()` - Update variable mappings

### 5.2 Dialog Structure: Two-Step Workflow

#### **Step 1: Mapping Discovery & Configuration**
- **Purpose**: Find existing mapping details and configure wildcard patterns
- **Input**: Selected JSON path from the explorer
- **Output**: Configured mapping with wildcard choices

#### **Step 2: Mapping Duplication & Scaling**
- **Purpose**: Create multiple similar mappings for different datapoints/objects
- **Input**: Configured mapping from Step 1
- **Output**: Multiple new mappings with scaled patterns

### 5.3 Step 1: Mapping Discovery & Configuration

**Component Name**: `MappingDiscoveryStep`

**Core Functions Needed:**
1. **`analyzeJsonPath()`** - Parse selected JSON path and extract structure
2. **`suggestWildcardPatterns()`** - Generate wildcard suggestions based on JSON structure
3. **`validateWildcardPattern()`** - Validate user-selected wildcard patterns
4. **`previewMappingImpact()`** - Show which paths would be affected by the mapping

**Workflow:**
1. **Path Analysis**: Take selected JSON path (e.g., `"api_response.data.companies.0.website:value"`)
2. **Structure Detection**: Identify array indices, object keys, and value types
3. **Wildcard Suggestions**: Propose wildcard patterns:
   - `["api_response", "data", "companies", "*", "website"]` (for all companies)
   - `["api_response", "data", "companies", "**", "website"]` (for nested company structures)
   - `["api_response", "data", "*", "website"]` (for all top-level objects with website)
4. **Interactive Selection**: User chooses wildcard pattern with visual preview
5. **Impact Preview**: Show all matching paths that would be affected
6. **Mapping Type Selection**: Choose between object, datapoint, or variable mapping

**UI Components:**
- **Path Display**: Show original JSON path with syntax highlighting
- **Wildcard Selector**: Interactive tree view of path segments with wildcard options
- **Preview Panel**: Live preview of affected paths
- **Mapping Type Radio**: Object/Datapoint/Variable selection
- **Validation Feedback**: Real-time validation of selected pattern

### 5.4 Step 2: Mapping Duplication & Scaling

**Component Name**: `MappingScalingStep`

**Core Functions Needed:**
1. **`generateScaledMappings()`** - Create multiple mappings based on pattern
2. **`detectSimilarStructures()`** - Find similar JSON structures for scaling
3. **`validateScaledMappings()`** - Ensure all generated mappings are valid
4. **`batchCreateMappings()`** - Create multiple mappings in one operation

**Workflow:**
1. **Pattern Analysis**: Take configured mapping from Step 1
2. **Structure Discovery**: Scan JSON data for similar structures
3. **Scaling Options**: Present scaling options:
   - **Horizontal Scaling**: Apply to similar objects (companies → people, products, etc.)
   - **Vertical Scaling**: Apply to nested structures (level 1 → level 2, level 3, etc.)
   - **Pattern Scaling**: Apply to different data types (website → email, phone, address)
4. **Batch Configuration**: Configure multiple mappings at once
5. **Preview & Validation**: Show all mappings that will be created
6. **Batch Creation**: Create all mappings in one operation

**UI Components:**
- **Scaling Options**: Checkboxes for different scaling approaches
- **Structure Browser**: Tree view of similar JSON structures
- **Batch Configuration**: Table view of mappings to be created
- **Preview Panel**: Show impact of all mappings
- **Progress Indicator**: Show creation progress

### 5.5 Core Dialog Component

**Component Name**: `ManageMappingDialog`

**Props Interface:**
```typescript
interface ManageMappingDialogProps {
  open: boolean
  onOpenChange: (open: boolean) => void
  selectedPath: string // JSON path from explorer
  mappingType: 'object' | 'datapoint' | 'variable'
  onMappingCreated: (mappings: Mapping[]) => void
  jsonData: any // Full JSON data for analysis
  existingMappings: Mappings // Current mappings for validation
}
```

**State Management:**
- **Current Step**: 1 or 2
- **Selected Path**: Original JSON path
- **Wildcard Pattern**: User-selected pattern
- **Mapping Type**: Object/Datapoint/Variable
- **Scaling Options**: Selected scaling approaches
- **Generated Mappings**: Mappings to be created
- **Validation State**: Current validation status

### 5.6 New Utility Functions Needed

**Path Analysis Functions:**
1. **`analyzeJsonPathStructure()`** - Extract structure information from JSON path
2. **`suggestWildcardPatterns()`** - Generate wildcard suggestions
3. **`findSimilarPaths()`** - Find similar paths in JSON data
4. **`validateWildcardPattern()`** - Validate wildcard pattern against JSON structure

**Mapping Generation Functions:**
1. **`generateScaledMappings()`** - Create multiple mappings from pattern
2. **`detectSimilarStructures()`** - Find similar JSON structures
3. **`batchCreateMappings()`** - Create multiple mappings
4. **`previewMappingImpact()`** - Show affected paths

**Integration Functions:**
1. **`integrateWithExistingMappings()`** - Merge with existing mappings
2. **`validateMappingConflicts()`** - Check for mapping conflicts
3. **`updateMappingStatuses()`** - Update fullPaths statuses after creation

### 5.7 Dialog Integration Points

**With JsonExplorer:**
- **Trigger**: Right-click on JSON path → "Manage Mapping"
- **Context**: Pass selected path and mapping type
- **Callback**: Receive created mappings and update UI

**With Mapping System:**
- **Object Mappings**: Integrate with `assignMappingsToFullPaths()`
- **Variable Mappings**: Integrate with `assignMappingToVariable()`
- **Status Updates**: Update fullPaths and basePaths statuses

**With Progress Bar:**
- **Statistics Update**: Refresh mapping statistics after creation
- **Status Propagation**: Update progress bar counts

### 5.8 Advanced Features

**Smart Suggestions:**
- **Pattern Recognition**: Analyze JSON structure to suggest optimal wildcards
- **Conflict Detection**: Warn about potential mapping conflicts
- **Impact Analysis**: Show how many paths will be affected

**Batch Operations:**
- **Bulk Creation**: Create multiple mappings efficiently
- **Template Application**: Apply mapping patterns to similar structures
- **Validation Pipeline**: Comprehensive validation before creation

**User Experience:**
- **Visual Path Editor**: Interactive path editing with wildcard insertion
- **Live Preview**: Real-time preview of mapping effects
- **Undo/Redo**: Support for undoing mapping operations
- **Export/Import**: Save and load mapping configurations

### 5.9 Implementation Phases

#### **Phase 5.1: Core Dialog Structure**
- Create `ManageMappingDialog` component with two-step workflow
- Implement basic dialog navigation and state management
- Add dialog integration with JsonExplorer

#### **Phase 5.2: Path Analysis Functions**
- Implement `analyzeJsonPathStructure()` and related functions
- Create wildcard suggestion system
- Add pattern validation logic

#### **Phase 5.3: Mapping Generation Functions**
- Implement scaling and duplication functions
- Create batch mapping creation system
- Add conflict detection and validation

#### **Phase 5.4: UI Components**
- Build interactive path editor
- Create wildcard selector component
- Implement preview and validation panels

#### **Phase 5.5: Integration & Testing**
- Integrate with existing mapping system
- Test with real JSON data and mappings
- Add comprehensive error handling and user feedback

This comprehensive dialog system will provide powerful mapping management capabilities while integrating seamlessly with the existing JSON mapping architecture.

## Phase 6: Fix Infinite Loop State Management Issues (CRITICAL)

**Goal**: Resolve the infinite re-render loop in all three JSON mappers by fixing state management and rendering logic.

### 6.1 Problem Analysis

**Current Issue**: All three mappers (`JsonResultMapper`, `JsonRequestMapper`, `JsonResponseMapper`) are stuck in an infinite loop where:
- Components render with all processing states `false`
- Components return `null` because `dataRendered` is false
- Components unmount immediately, preventing useEffects from running
- States never get updated, causing infinite re-renders

**Root Cause**: The components return `null` when processing is incomplete, which unmounts them before useEffects can execute. This creates a chicken-and-egg problem where the component needs to process data to render, but can't process data because it's unmounted.

### 6.2 State Management Simplification

**Remove These States:**
- `pathsGenerated` - Redundant with `jsonProcessed`
- `mappingStatisticsProcessed` - Redundant with `mappingsProcessed` 
- `dataRendered` - Causes the unmounting problem

**Keep These States:**
- `jsonProcessed` - Step 1 completion flag
- `mappingsProcessed` - Step 2 completion flag
- `statisticsProcessed` - Step 3 completion flag
- `basePaths` - Data state
- `fullPaths` - Data state

### 6.3 Processing Pipeline Redesign

**Step 1: Generate Paths from JSON**
- Trigger: `data` changes
- Process: Call `generatePathData(data)`
- Set: `basePaths`, `fullPaths`, `jsonProcessed = true`
- Dependencies: `[data, jsonProcessed]`

**Step 2: Apply Mappings**
- Trigger: `jsonProcessed` is true and mappings available
- Process: Apply mappings to `fullPaths`
- Set: Updated `fullPaths`, `mappingsProcessed = true`
- Dependencies: `[fullPaths, mappings, mappingsProcessed]`

**Step 3: Calculate Statistics**
- Trigger: `mappingsProcessed` is true
- Process: Calculate final statistics for `basePaths`
- Set: Updated `basePaths`, `statisticsProcessed = true`
- Dependencies: `[basePaths, fullPaths, statisticsProcessed]`

### 6.4 Rendering Logic Fix

**Current (Broken):**
```typescript
if (dataRendered && onProcessedData) {
  return onProcessedData({...})
}
return null  // ← This causes unmounting
```

**New (Fixed):**
```typescript
if (statisticsProcessed && onProcessedData) {
  return onProcessedData({...})
}
return <div>Processing...</div>  // ← Always render something
```

### 6.5 Implementation Steps

**For All 3 Mappers:**

#### **Step 1: Remove Problematic States**
- Delete `pathsGenerated`, `mappingStatisticsProcessed`, `dataRendered` state declarations
- Remove all references to these states in useEffects and rendering logic

#### **Step 2: Update useEffect Dependencies**
- Step 1: `[data, jsonProcessed]` (no change)
- Step 2: `[fullPaths, mappings, mappingsProcessed]` (change from `pathsGenerated` to `fullPaths`)
- Step 3: `[basePaths, fullPaths, statisticsProcessed]` (new step)

#### **Step 3: Update Processing Logic**
- Step 1: Set `jsonProcessed = true` after generating paths
- Step 2: Set `mappingsProcessed = true` after applying mappings
- Step 3: Set `statisticsProcessed = true` after calculating statistics

#### **Step 4: Fix Rendering Logic**
- Change condition from `dataRendered` to `statisticsProcessed`
- Replace `return null` with `return <div>Processing...</div>`

#### **Step 5: Update State Reset Logic**
- Reset only the 3 core states: `jsonProcessed`, `mappingsProcessed`, `statisticsProcessed`

### 6.6 Files to Modify

1. `components/json/response/json-result-mapper.tsx`
2. `components/json/request/json-request-mapper.tsx` 
3. `components/json/response/json-response-mapper.tsx`

### 6.7 Expected Outcome

After implementation:
- Components will always render something, preventing unmounting
- useEffects will run and complete the processing pipeline
- States will progress from `false` to `true` in sequence
- Infinite loop will be eliminated
- Components will render processed data when `statisticsProcessed` is true

### 6.8 Validation Criteria

**Success Indicators:**
- No "Maximum update depth exceeded" errors
- Components render once and complete processing
- All processing states progress from `false` to `true`
- useEffects execute and complete successfully
- Components render processed data instead of returning null

**Testing Approach:**
- Monitor terminal output for infinite re-renders
- Verify useEffect logs appear in console
- Confirm states progress through the pipeline
- Test with different JSON data and mapping configurations

## Phase 7: Code Cleanup and Optimization

**Goal**: Remove unused code, excessive logging, and optimize the mappers for production use.

### 7.1 High Priority Cleanup

**Remove Unused Imports:**
- `JsonExplorer` import from both request and response mappers
- `FullPathData` import (only `BasePathData` is used)
- Unused destructured variable `updatedRequestAndResponseVariableMappings`

**Remove Excessive Console Logging:**
- 90% of debug logs added for troubleshooting infinite loop
- Keep only error logging and essential production logs
- Remove render-time logging that creates objects on every render

**Remove Unused Props:**
- `className` prop from both request and response mappers

### 7.2 Medium Priority Cleanup

**Extract Duplicated Code:**
- Move `RequestAndResponseVariableMappings` interface to shared types file
- Consolidate duplicate `setBasePaths` logic in statistics effects

**Clean Up Comments:**
- Remove outdated section headers and comments
- Remove "moved from JsonExplorer" references

### 7.3 Low Priority Cleanup

**Type Definitions:**
- Extract inline type definitions to shared types
- Simplify complex callback type definitions

### 7.4 Expected Results

**File Size Reduction:** ~30-40% smaller files
**Performance Improvement:** Less object creation on render
**Maintainability:** Much cleaner and easier to understand code
**Readability:** Focus on actual functionality instead of debug noise

### 7.5 Files to Clean

1. `components/json/response/json-result-mapper.tsx`
2. `components/json/request/json-request-mapper.tsx`
3. `components/json/response/json-response-mapper.tsx`

## Phase 8: Fix Circular Dependency Issues (CRITICAL)

**Goal**: Resolve the circular dependency causing infinite re-render loops by restructuring state management and function organization.

### 8.1 Root Cause Analysis

**Primary Issue**: `getBasePathStatistics` function is called inside `setBasePaths` in all three mappers, creating a circular dependency chain:
1. `setBasePaths` called → triggers re-render
2. Re-render → `useEffect` runs → calls `setBasePaths` again
3. `setBasePaths` called → triggers re-render again
4. Infinite loop continues

**Secondary Issues**:
- Multiple `setBasePaths` calls in different useEffects
- Inconsistent `fullPaths` references between useEffects
- `getBasePathStatistics` function in wrong location (rendering-utils instead of mapping-utils)
- State update chain reactions causing unnecessary re-runs

### 8.2 Function Organization Fix

**Move Functions to Correct Locations**:
- **Move `getBasePathStatistics`** from `utils/json/rendering-utils.tsx` to `utils/json/mapping-utils.tsx`
- **Keep `getObjectAndDatapointMappingStatistics`** in `utils/json/mapping-utils.tsx` (already correct)
- **Note**: The functions `getPathsByStatus` and `getVariablePathsByStatus` do not exist in the current codebase - they were only planned but never implemented.

### 8.3 State Management Restructuring

**Eliminate Circular Dependencies**:
- **Remove `getBasePathStatistics` calls from inside `setBasePaths`**
- **Call `getBasePathStatistics` separately** after state updates
- **Consolidate multiple `setBasePaths` calls** into single operations
- **Fix `fullPaths` reference consistency** across all useEffects

### 8.4 Processing Pipeline Fix

**Keep Existing Processing Flow** (3 steps only):
1. **Step 1**: Generate paths from JSON data
   - Call `generatePathData(data)`
   - Set `basePaths`, `fullPaths`, `jsonProcessed = true`
   - Dependencies: `[data, jsonProcessed]`

2. **Step 2**: Apply mappings to fullPaths
   - Call mapping functions (`assignMappingsToFullPaths`, `getRequestVariableMappingStatus`, etc.)
   - Update `fullPaths` with mappings
   - Set `mappingsProcessed = true`
   - Dependencies: `[jsonProcessed, mappings, mappingsProcessed]`

3. **Step 3**: Update basePath statuses and calculate statistics
   - Call `getBasePathStatus(basePaths, fullPaths)` to update statuses
   - Call `getBasePathStatistics(basePaths, fullPaths)` **separately** (not inside setBasePaths)
   - Update `basePaths` with both statuses and statistics
   - Set `statisticsProcessed = true`
   - Dependencies: `[mappingsProcessed, fullPaths]`

### 8.5 Specific Implementation Changes

**For All Three Mappers**:

#### **8.5.1 Remove Circular Dependencies**
- **Remove `getBasePathStatistics` calls from inside `setBasePaths`**
- **Call `getBasePathStatistics` separately** in Step 3 useEffect (not inside setBasePaths)
- **Use consistent `fullPaths` references** across all useEffects

#### **8.5.2 Update Import Statements**
- **Change import source** for `getBasePathStatistics` from `rendering-utils` to `mapping-utils`
- **Remove unused imports** (none needed - functions don't exist)

#### **8.5.3 Restructure useEffect Logic**
- **Step 1 useEffect**: `[data, jsonProcessed]` - Generate paths only
- **Step 2 useEffect**: `[jsonProcessed, mappings, mappingsProcessed]` - Apply mappings only
- **Step 3 useEffect**: `[mappingsProcessed, fullPaths]` - Update basePath statuses AND calculate statistics

#### **8.5.4 Fix State Update Patterns**
- **Single `setBasePaths` call per useEffect** (no multiple calls)
- **No nested state updates** (no `setBasePaths` inside `setBasePaths`)
- **Consistent state references** (use same `fullPaths` variable throughout)

### 8.6 Files to Modify

**Utility Files**:
1. **`utils/json/rendering-utils.tsx`**:
   - Remove `getBasePathStatistics` function (move to mapping-utils)
   - Note: `getPathsByStatus` and `getVariablePathsByStatus` don't exist (were never implemented)

2. **`utils/json/mapping-utils.tsx`**:
   - Add `getBasePathStatistics` function (moved from rendering-utils)

**Mapper Components**:
3. **`components/json/response/json-result-mapper.tsx`**:
   - Change import source for `getBasePathStatistics`
   - Restructure useEffect logic to eliminate circular dependencies
   - Fix state update patterns

4. **`components/json/request/json-request-mapper.tsx`**:
   - Change import source for `getBasePathStatistics`
   - Restructure useEffect logic to eliminate circular dependencies
   - Fix state update patterns

5. **`components/json/response/json-response-mapper.tsx`**:
   - Change import source for `getBasePathStatistics`
   - Restructure useEffect logic to eliminate circular dependencies
   - Fix state update patterns

### 8.7 Expected Outcome

**After Implementation**:
- **No circular dependencies** - `getBasePathStatistics` called separately from state updates
- **No infinite loops** - Clean state management without nested updates
- **Proper function organization** - Statistics functions in mapping-utils, rendering functions in rendering-utils
- **Consistent state flow** - Clear progression through processing steps
- **Better performance** - Fewer unnecessary re-renders and state updates

### 8.8 Validation Criteria

**Success Indicators**:
- No "Maximum update depth exceeded" errors
- No circular dependency warnings
- Components render once and complete processing
- All processing states progress from `false` to `true` in sequence
- useEffects execute without triggering additional re-renders
- Statistics calculations work correctly without infinite loops

**Testing Approach**:
- Monitor terminal output for infinite re-renders
- Verify useEffect execution order and completion
- Test with different JSON data and mapping configurations
- Confirm statistics are calculated correctly
- Validate that components render processed data successfully

### 8.9 Critical Implementation Notes

**DO NOT**:
- Call `getBasePathStatistics` inside `setBasePaths` functions
- Use multiple `setBasePaths` calls in the same useEffect
- Mix different `fullPaths` references in the same component
- Call state update functions inside other state update functions

**DO**:
- Call `getBasePathStatistics` separately in Step 3 useEffect (not inside setBasePaths)
- Use consistent `fullPaths` references throughout the component
- Structure useEffects with clear, single responsibilities
- Test each step of the processing pipeline independently

## Phase 9: Fix State Management Architecture and Circular Dependencies (CRITICAL)

**Goal**: Resolve the fundamental state management issues causing infinite re-render loops, stale closures, and performance problems by implementing a proper 5-step processing pipeline with correct dependencies.

### 9.1 Root Cause Analysis

**Primary Issues Identified**:

1. **Stale Closure Problem**: 
   - JsonResultMapper and JsonRequestMapper Step 3 useEffect use `fullPaths` inside `setBasePaths` callback but `fullPaths` is missing from dependency array
   - When `handlePathSelect` updates `isSelected` in `fullPaths`, Step 3 doesn't re-run with fresh data
   - Statistics are calculated with stale `basePaths` containing old mapping data

2. **Performance Issue**:
   - `handlePathSelect` only updates `isSelected` property in `fullPaths`
   - `isSelected` changes should NOT trigger expensive statistics recalculation
   - Current architecture recalculates statistics on every path selection

3. **Mapping Update Issue**:
   - When mappings are edited, the system doesn't properly retrigger processing
   - Once `mappingsProcessed` is true, Step 2 never runs again even if mapping props change
   - Statistics become stale when mappings are updated

4. **Circular Dependency Issue**:
   - Step 3 useEffect has `fullPaths` in dependency array
   - `handlePathSelect` updates `fullPaths` → triggers Step 3 → recalculates statistics unnecessarily
   - This creates performance bottlenecks and potential infinite loops

### 9.2 Current Architecture Problems

**All three mappers have identical issues**:

1. **JsonResultMapper** (lines 125-139): Missing `fullPaths` in dependency array
2. **JsonRequestMapper** (lines 120-134): Missing `fullPaths` in dependency array  
3. **JsonResponseMapper** (lines 120-134): HAS `fullPaths` in dependency array

**Step 3 useEffect in ALL mappers**:
- Uses `fullPaths` inside `setBasePaths` callback
- But `fullPaths` is missing from dependency array in JsonResultMapper and JsonRequestMapper
- This creates **stale closures** - the useEffect uses old `fullPaths` values
- When `handlePathSelect` updates `isSelected`, Step 3 doesn't re-run

### 9.3 What `handlePathSelect` Actually Influences

**DOES**:
- Updates `isSelected` property in `fullPaths` array
- Triggers UI re-renders for visual selection highlighting
- Opens mapping selectors in JsonNode components
- Updates scroll position in JsonExplorer

**DOES NOT**:
- Affect statistics calculations - `isSelected` is never used in any statistics functions
- Affect mapping data - `objectMappings`, `datapointMappings`, `variableMappings` are unchanged
- Affect status calculations - `status` field is unchanged
- Affect basePaths - no basePaths are modified

### 9.4 Correct 5-Step Processing Pipeline

**Step 1: Parse JSON**
- **Purpose**: Generate `basePaths` and `fullPaths` from raw data
- **Dependencies**: `[data]`
- **Triggers**: When data changes
- **Actions**: Call `generatePathData(data)`, set `basePaths`, `fullPaths`, `jsonProcessed = true`

**Step 2: Apply Mappings**
- **Purpose**: Update `fullPaths` with mapping assignments
- **Dependencies**: `[data, mappings, jsonProcessed]`
- **Triggers**: When data or mappings change
- **Actions**: Call mapping functions, update `fullPaths`, set `mappingsProcessed = true`

**Step 3: Set Up Statuses**
- **Purpose**: Update `basePaths` with status information
- **Dependencies**: `[mappingsProcessed, fullPaths]`
- **Triggers**: When mappings are processed
- **Actions**: Call `getBasePathStatus(currentBasePaths, fullPaths)`, set `statusesProcessed = true`

**Step 4: Calculate Statistics**
- **Purpose**: Update `basePaths` with statistics
- **Dependencies**: `[statusesProcessed, fullPaths]`
- **Triggers**: When statuses are set up
- **Actions**: Call `getBasePathStatistics(updatedBasePaths, fullPaths)`, set `statisticsProcessed = true`

**Step 5: Render**
- **Purpose**: Pass current state to JsonExplorer
- **Dependencies**: None (no useEffect)
- **Triggers**: On every render
- **Actions**: Use current state values, pass data to JsonExplorer

### 9.5 Required State Changes

**Add Missing State Variable**:
- Add `statusesProcessed` state to all three mappers
- This separates status setup from statistics calculation

**Add Mapping Reset Logic**:
- Add useEffect to reset `mappingsProcessed`, `statusesProcessed`, and `statisticsProcessed` when mappings change
- This ensures mappings are reprocessed when edited

### 9.6 Specific Implementation Changes

**For All Three Mappers**:

#### **9.6.1 Add Missing State Variable**
- Add `const [statusesProcessed, setStatusesProcessed] = useState(false)` after line 82 (after `statisticsProcessed`)

#### **9.6.2 Add Mapping Reset Logic**
- Add new useEffect after the data reset useEffect (after line 93):
  ```typescript
  // Reset when mappings change
  useEffect(() => {
    if (objectsAndDatapointsMappings.sources) { // or requestAndResponseVariableMappings
      setMappingsProcessed(false)
      setStatusesProcessed(false)
      setStatisticsProcessed(false)
    }
  }, [objectsAndDatapointsMappings]) // or requestAndResponseVariableMappings
  ```

#### **9.6.3 Fix Step 3 useEffect (Set Up Statuses)**
- **Current**: Mixes status setup and statistics calculation
- **Fix**: Only set up statuses, remove statistics calculation
- **Dependencies**: Add `fullPaths` to dependency array
- **Actions**: Call only `getBasePathStatus`, set `statusesProcessed = true`

#### **9.6.4 Add Step 4 useEffect (Calculate Statistics)**
- **New useEffect**: Calculate statistics when statuses are set up
- **Dependencies**: `[statusesProcessed, fullPaths]`
- **Actions**: Call `getBasePathStatistics`, set `statisticsProcessed = true`

#### **9.6.5 Update Data Reset Logic**
- **Current**: Resets `jsonProcessed`, `mappingsProcessed`, `statisticsProcessed`
- **Fix**: Also reset `statusesProcessed`

### 9.7 Files to Modify

**All Three Mapper Components**:
1. `components/json/response/json-result-mapper.tsx`
2. `components/json/request/json-request-mapper.tsx`
3. `components/json/response/json-response-mapper.tsx`

### 9.8 Expected Behavior After Fix

**Initial Load**:
- Data → Parse → Apply Mappings → Set Statuses → Calculate Statistics → Render

**Path Selection**:
- Only updates `isSelected` → triggers UI re-render only

**Mapping Edits**:
- Mappings change → resets all flags except `jsonProcessed` → reprocesses mappings and everything after it → updates statistics

**Data Changes**:
- New data → resets all flags → full reprocessing

### 9.9 Validation Criteria

**Success Indicators**:
- No "Maximum update depth exceeded" errors
- No circular dependency warnings
- Components render once and complete processing
- All processing states progress from `false` to `true` in sequence
- useEffects execute without triggering additional re-renders
- Statistics calculations work correctly without infinite loops
- `handlePathSelect` only triggers UI updates, not data processing
- Mapping edits properly retrigger processing pipeline

**Testing Approach**:
- Monitor terminal output for infinite re-renders
- Verify useEffect execution order and completion
- Test with different JSON data and mapping configurations
- Confirm statistics are calculated correctly
- Validate that components render processed data successfully
- Test path selection performance (should be fast)
- Test mapping edit retriggering

### 9.10 Critical Implementation Notes

**DO NOT**:
- Call `getBasePathStatistics` inside `setBasePaths` functions
- Use multiple `setBasePaths` calls in the same useEffect
- Mix different `fullPaths` references in the same component
- Call state update functions inside other state update functions
- Put `fullPaths` in dependency arrays unless the useEffect actually uses `fullPaths` data (not just `isSelected`)

**DO**:
- Call `getBasePathStatistics` separately in Step 4 useEffect (not inside setBasePaths)
- Use consistent `fullPaths` references throughout the component
- Structure useEffects with clear, single responsibilities
- Test each step of the processing pipeline independently
- Ensure `handlePathSelect` only updates `isSelected` and triggers UI updates
- Ensure mapping changes properly reset downstream processing flags

## Phase 10: SourceBodyUpload Integration into Mapper Components (CRITICAL)

**Goal**: Integrate SourceBodyUpload functionality directly into all three mapper components (JsonRequestMapper, JsonResponseMapper, JsonResultMapper) to eliminate the need for complex form orchestration and provide seamless upload-to-mapping workflow.

### 10.1 Current Architecture Problems

**Fundamental Issues Identified**:

1. **Disconnected Upload Functionality**:
   - SourceBodyUpload component exists but only used in showcase pages
   - Forms show placeholder text "Upload JSON files to see mapping" with no actual upload capability
   - No integration between upload component and mapper components
   - Users cannot upload files in actual form workflows

2. **Complex Form Orchestration**:
   - Forms have dead code: `requestJsonData`, `responseJsonData` state variables that are never used
   - Forms have unused handlers: `handleRequestJsonDataChange`, `handleResponseJsonDataChange` that are never called
   - Complex conditional rendering logic needed to switch between upload and mapper components
   - State management complexity between separate upload and mapper components

3. **Poor User Experience**:
   - No seamless transition from upload to mapping
   - Jarring component switches when data becomes available
   - Inconsistent interface across different mapper types
   - No way to upload files in real form workflows

4. **Maintenance Issues**:
   - SourceBodyUpload component isolated from actual usage
   - Duplicate state management patterns across forms
   - Complex integration requirements for each new form
   - Difficult to maintain consistent upload behavior

### 10.2 Correct Architecture Design

#### **10.2.1 Enhanced Mapper Components (Data Processing + Upload)**
**Responsibilities**:
- Process JSON data and generate `basePaths` and `fullPaths`
- Apply mappings (object, datapoint, variable)
- Calculate statistics
- Handle file upload, text input, and API requests
- Provide seamless transition from upload to mapping
- NO UI state management for selection/expansion
- NO user interaction handling for JSON exploration

**Enhanced Interface**:
```typescript
interface JsonRequestMapperProps {
  data?: any  // Make optional - when null, show upload
  requestAndResponseVariableMappings: RequestAndResponseVariableMappings
  showUpload?: boolean  // Enable/disable upload functionality (default: true)
  uploadLabel?: string  // Customize upload component label
  onDataChange?: (data: any) => void  // Handle data from upload
  onProcessedData?: (processedData: {...}) => React.ReactNode
}
```

**State Variables** (Enhanced):
- `jsonProcessed` - Step 1 completion flag
- `mappingsProcessed` - Step 2 completion flag  
- `statisticsProcessed` - Step 3 completion flag
- `jsonFiles` - Array of uploaded files
- `activeFileId` - Currently selected file ID
- `textInput` - Text input content
- `isRequesting` - API request loading state
- `requestError` - API request error state

**Upload Integration Logic**:
- When `data` is null/undefined AND `showUpload` is true: render SourceBodyUpload component
- When `data` exists: process and render mapping results
- Handle data flow from upload to mapper processing internally
- Seamless transition without component switching

### 10.3 Implementation Strategy

#### **10.3.1 Mapper Component Enhancement Pattern**

**For All Three Mappers (JsonRequestMapper, JsonResponseMapper, JsonResultMapper)**:

**Step 1: Update Interface**
- Make `data` prop optional (`data?: any`)
- Add `showUpload?: boolean` prop (default: true)
- Add `uploadLabel?: string` prop for customization
- Add `onDataChange?: (data: any) => void` callback

**Step 2: Add Upload State Management**
- Add `jsonFiles` state for uploaded files
- Add `activeFileId` state for selected file
- Add `textInput` state for text input
- Add `isRequesting` state for API requests
- Add `requestError` state for error handling

**Step 3: Implement Upload Integration Logic**
- Create `handleJsonDataChange` callback to process upload data
- Add conditional rendering: show upload when no data, show mapper when data exists
- Handle seamless transition from upload to mapping mode
- Process uploaded data through existing mapper pipeline

**Step 4: Update Rendering Logic**
- When `!data && showUpload`: render SourceBodyUpload component
- When `!data && !showUpload`: render placeholder text
- When `data`: process through existing mapper logic
- Call `onDataChange` when data becomes available from upload

#### **10.3.2 Data Flow Architecture**

**Upload to Mapping Flow**:
1. User uploads file/pastes text/makes API request
2. SourceBodyUpload processes input and calls `handleJsonDataChange`
3. Mapper updates internal state and calls `onDataChange` callback
4. Form receives data via `onDataChange` and updates form state
5. Mapper receives data via `data` prop and processes through mapping pipeline
6. Seamless transition from upload UI to mapping UI

**Form Integration**:
- Forms pass `data` prop to mappers (can be null/undefined)
- Forms handle `onDataChange` callback to update form state
- No complex orchestration state needed
- No conditional rendering logic in forms

### 10.4 Specific Implementation Steps

#### **10.4.1 Update JsonRequestMapper**

**File**: `components/json/request/json-request-mapper.tsx`

**Change 1: Update Interface**
- Change `data: any` to `data?: any` (make optional)
- Add `showUpload?: boolean` prop (default: true)
- Add `uploadLabel?: string` prop
- Add `onDataChange?: (data: any) => void` callback

**Change 2: Add Upload State Management**
- Add `const [jsonFiles, setJsonFiles] = useState<JsonFile[]>([])`
- Add `const [activeFileId, setActiveFileId] = useState<string | null>(null)`
- Add `const [textInput, setTextInput] = useState<string>("")`
- Add `const [isRequesting, setIsRequesting] = useState<boolean>(false)`
- Add `const [requestError, setRequestError] = useState<string>("")`

**Change 3: Add Upload Integration Logic**
- Create `handleJsonDataChange` callback to process upload data
- Add conditional rendering logic before existing processing pipeline
- When `!data && showUpload`: render SourceBodyUpload component
- When `!data && !showUpload`: render placeholder text
- When `data`: continue with existing mapper logic

**Change 4: Update Data Processing Pipeline**
- Modify existing useEffect to handle data from upload
- Call `onDataChange` when data becomes available
- Reset processing states when new data is received

#### **10.4.2 Update JsonResponseMapper**

**File**: `components/json/response/json-response-mapper.tsx`

**Apply identical changes as JsonRequestMapper**:
- Same interface updates
- Same state management additions
- Same upload integration logic
- Same data processing pipeline updates

#### **10.4.3 Update JsonResultMapper**

**File**: `components/json/response/json-result-mapper.tsx`

**Apply identical changes as JsonRequestMapper**:
- Same interface updates
- Same state management additions
- Same upload integration logic
- Same data processing pipeline updates

#### **10.4.4 Update Form Components**

**File**: `components/forms/run-request-form.tsx`

**Change 1: Remove Dead Code**
- Remove `requestJsonData` and `responseJsonData` from form state
- Remove `handleRequestJsonDataChange` and `handleResponseJsonDataChange` handlers
- Remove unused state management logic

**Change 2: Update Mapper Integration**
- Pass `data={formData.requestJsonData}` to JsonRequestMapper (can be null)
- Pass `data={formData.responseJsonData}` to JsonResponseMapper (can be null)
- Add `onDataChange` callbacks to update form state
- Add `showUpload={true}` and `uploadLabel` props

**Change 3: Simplify Form Logic**
- Remove conditional rendering logic
- Remove placeholder text rendering
- Let mappers handle upload/mapping display internally

**Apply same changes to**:
- `components/forms/status-check-form.tsx`
- `components/forms/delivery-form.tsx`

#### **10.4.5 Update SourceRequestResponseAccordion**

**File**: `components/accordions/source-request-response-accordion.tsx`

**No changes needed** - accordion already passes mapper components as props
- Mappers handle upload functionality internally
- Accordion just renders the mapper components
- No upload-specific logic needed in accordion

### 10.5 Expected Benefits

#### **10.5.1 Simplified Architecture**
- **Single Component Responsibility**: Each mapper handles both upload and mapping
- **Eliminated Orchestration**: No complex state management between separate components
- **Consistent Interface**: Same upload experience across all mapper types
- **Reduced Complexity**: 80% reduction in form integration complexity

#### **10.5.2 Better User Experience**
- **Seamless Transition**: No jarring component switches from upload to mapping
- **Consistent Workflow**: Same upload process for all mapper types
- **Real Upload Functionality**: Users can actually upload files in form workflows
- **Unified Interface**: Single component handles all data input methods

#### **10.5.3 Improved Maintainability**
- **Self-Contained Components**: Mappers are independent and reusable
- **Easier Testing**: Single component to test upload and mapping functionality
- **Cleaner Forms**: Forms only need to handle data callbacks, not upload logic
- **Better Code Organization**: Upload logic lives where it's used

#### **10.5.4 Performance Benefits**
- **No Component Switching**: Eliminates re-mounting and re-initialization
- **Optimized State Management**: Upload state managed internally by mappers
- **Reduced Re-renders**: No complex conditional rendering in forms
- **Better Memory Usage**: Single component instance instead of multiple

### 10.6 Files to Modify

**Mapper Components**:
1. `components/json/request/json-request-mapper.tsx`
2. `components/json/response/json-response-mapper.tsx`
3. `components/json/response/json-result-mapper.tsx`

**Form Components**:
4. `components/forms/run-request-form.tsx`
5. `components/forms/status-check-form.tsx`
6. `components/forms/delivery-form.tsx`

**No Changes Needed**:
- `components/accordions/source-request-response-accordion.tsx` (already passes mappers as props)
- `components/json/interface/source-body-upload.tsx` (used internally by mappers)

### 10.7 Validation Criteria

**Success Indicators**:
- Upload functionality works in all three mapper types
- Seamless transition from upload to mapping mode
- Forms can upload files and process them through mappers
- No complex orchestration state needed in forms
- Consistent upload experience across all mappers
- Mappers handle both upload and mapping internally
- Forms only need to handle data callbacks

**Testing Approach**:
- Test file upload in JsonRequestMapper, JsonResponseMapper, JsonResultMapper
- Test text input in all mapper types
- Test API request functionality in all mapper types
- Test seamless transition from upload to mapping
- Test form integration with all three form types
- Verify no placeholder text appears when upload is enabled
- Confirm data flows correctly from upload to mapping

### 10.8 Critical Implementation Notes

**DO NOT**:
- Create separate upload components for each mapper
- Mix upload state with mapping state in forms
- Use complex conditional rendering in forms
- Duplicate upload logic across components
- Leave dead code in forms

**DO**:
- Integrate upload functionality directly into mappers
- Make data prop optional in all mappers
- Handle upload state internally in mappers
- Use consistent upload interface across all mappers
- Keep forms simple with just data callbacks
- Test upload functionality in actual form workflows

This integration approach eliminates the need for complex form orchestration while providing seamless upload-to-mapping functionality directly in the mapper components.

### 10.9 SourceBodyUpload Component Integration

**File**: `components/json/interface/source-body-upload.tsx`

**Component Status**:
- Component remains as standalone component
- Used internally by enhanced mapper components
- Available for showcase pages and other use cases
- No changes needed to existing implementation

**Integration Pattern**:
- Enhanced mappers import and use SourceBodyUpload internally
- When `data` is null/undefined and `showUpload` is true: render SourceBodyUpload
- SourceBodyUpload calls `onJsonDataChange` callback with uploaded data
- Mapper processes the data through existing mapping pipeline
- Seamless transition from upload UI to mapping UI

**Benefits**:
- **Reusability**: SourceBodyUpload remains available for other use cases
- **Maintainability**: Single source of truth for upload functionality
- **No Duplication**: Enhanced mappers leverage existing, tested component
- **Clear Separation**: Upload and mapping concerns remain separate

This completes the mapper component enhancement by integrating SourceBodyUpload functionality directly into the mapper components while keeping the component reusable.

## Phase 11: SourceBodyUpload Integration with Manual Transition Control (CRITICAL)

**Goal**: Add manual transition control to SourceBodyUpload component and fix file management issues.

### 11.1 Current Architecture Analysis

**Actual Architecture**:
- **Forms** → **Accordions** → **Mappers** → **SourceBodyUpload** (when no data) OR **JsonExplorer** (when data exists)
- Mappers already have SourceBodyUpload integrated but missing transition control
- SourceBodyUpload handles file upload and text input
- Missing: Continue button to switch from upload to mapping
- Problem: FormDragDrop has JSON parsing logic that should be in SourceBodyUpload

**Current State**:
- Mappers show SourceBodyUpload when `!data && showUpload`
- SourceBodyUpload handles file upload and text input
- No manual transition control - automatic switching based on data
- FormDragDrop has JSON parsing logic that should be in SourceBodyUpload

### 11.2 Required Changes

#### **11.2.1 SourceBodyUpload Component Updates**

**File**: `components/json/interface/source-body-upload.tsx`

**New Props** (lines 22-34):
```typescript
interface SourceBodyUploadProps {
  // ... existing props
  onJsonShow?: () => void  // Callback for continue button
  fixedHeight?: string     // Fixed height for consistent layout
}
```

**Continue Button** (after line 350):
- Add conditional "Continue - setup mappings" button
- Show when: `jsonFiles.length > 0 || hasValidTextData`
- Call `onJsonShow` callback

**Save as JSON File Button** (in text tab, around line 333):
- Add "Save as JSON file" button (secondarybutton) when text input has valid JSON
- Create real downloadable .json file using Blob

**Fixed Height** (line 209):
- Add `fixedHeight` prop with default "min-h-[400px]"
- Apply to main container

**JSON Processing** (lines 258-269):
- Remove `validateJson` and `onJsonParsed` from FormDragDrop usage
- Move JSON parsing logic from FormDragDrop to SourceBodyUpload

#### **11.2.2 FormDragDrop Component Simplification**

**File**: `components/forms/blocks/drag-drop-upload.tsx`

**Remove JSON Logic**:
- Remove `validateJson` prop (line 30)
- Remove `onJsonParsed` prop (line 31)
- Remove `validateJsonContent` function (lines 89-120)
- Remove JSON validation in `handleFiles` (lines 149-172)
- Remove file list rendering (lines 302-370)

**Simplify handleFiles** (lines 122-176):
- Only validate file size and type
- Pass raw File objects to onChange
- No JSON parsing or validation

#### **11.2.3 FileBadge Component Enhancement**

**File**: `components/ui/custom-ui-elements/badges/file-badge.tsx`

**Make Entire Area Clickable** (lines 49-77):
- Add `onClick={onRemove}` to main container div (line 50)
- Add `cursor-pointer` and `hover:bg-destructive/10` classes
- Remove `onClick={onSelect}` from span (line 62)

### 11.3 Mapper Integration

#### **11.3.1 Mapper State Management**
**Current Problem**: Mappers have SourceBodyUpload but missing transition control
**Correct Approach**: Add continue button callback and transition logic

**Required Changes in Mappers**:
- Add `onJsonShow` callback prop to SourceBodyUpload usage
- Add `handleJsonShow` function to switch from upload to mapping
- Add `height` prop for consistent layout

**Files to Modify**:
- `components/json/request/json-request-mapper.tsx` (lines where SourceBodyUpload is used)
- `components/json/response/json-response-mapper.tsx` (lines where SourceBodyUpload is used)
- `components/json/response/json-result-mapper.tsx` (lines where SourceBodyUpload is used)

#### **11.3.2 Transition Logic**
**Current Problem**: No way to switch from upload to mapping
**Correct Approach**: Continue button triggers transition

**Handler Logic**:
- `handleJsonShow`: Set `showUpload = false`, trigger mapping processing
- Show JsonExplorer with first active file from Mappers

### 11.4 Files to Modify

**Core Components**:
1. `components/json/interface/source-body-upload.tsx` - Add continue button, save button for text tab, fixed height
2. `components/forms/blocks/drag-drop-upload.tsx` - Remove JSON logic, simplify file handling
3. `components/ui/custom-ui-elements/badges/file-badge.tsx` - Make entire area clickable

**Mapper Components**:
4. `components/json/request/json-request-mapper.tsx` - Add continue button callback
5. `components/json/response/json-response-mapper.tsx` - Add continue button callback
6. `components/json/response/json-result-mapper.tsx` - Add continue button callback

**No Changes Needed**:
- `components/accordions/source-request-response-accordion.tsx` - Already passes mappers as props
- Form components - Mappers handle upload internally

### 11.5 Implementation Priority

**Critical**:
1. Add `onJsonShow` prop and continue button to SourceBodyUpload
2. Remove JSON parsing from FormDragDrop
3. Make FileBadge entire area clickable
4. Add continue button callbacks to all three mappers

**High**:
1. Add "Save as JSON file" button for text input
2. Implement fixed height for consistent layout
3. Move JSON processing logic to SourceBodyUpload


### 11.6 Expected Behavior

**Initial Load**:
- Mapper shows SourceBodyUpload when no data
- User uploads files or pastes text
- "Continue - setup mappings" button appears when valid data exists

**Continue Button Click**:
- Calls `onJsonShow` callback
- Mapper switches from SourceBodyUpload to JsonExplorer
- Shows first active file from uploaded data
- Seamless transition from upload UI to mapping UI

**File Management**:
- FormDragDrop only handles file selection (no JSON parsing)
- SourceBodyUpload handles all JSON processing
- Entire FileBadge area is clickable for deletion
- "Save as JSON file" creates real downloadable files

### 11.7 Validation Criteria

**Success Indicators**:
- Continue button appears when valid data is uploaded
- Clicking continue button switches from SourceBodyUpload to JsonExplorer
- First active file is displayed in JsonExplorer
- FormDragDrop only handles file selection, no JSON parsing
- Entire FileBadge area is clickable for deletion
- "Save as JSON file" creates real downloadable files
- Fixed height container maintains consistent layout

**Testing Approach**:
- Test file upload in all three mapper types
- Test text input and "Save as JSON file" functionality
- Test "Continue" button transition to mapper view
- Test FileBadge deletion with entire area clickable
- Test FormDragDrop with various file types (non-JSON should be rejected)
- Test SourceBodyUpload with invalid JSON (should show error)
- Test fixed height container with different content states
- Verify no automatic transitions occur without user action

### 11.8 Critical Implementation Notes (CORRECTED)

**DO NOT**:
- Use automatic transitions from upload to mapping
- Keep JSON parsing logic in FormDragDrop
- Create virtual files for text input
- Make only the "×" button clickable in FileBadge
- Use variable height containers

**DO**:
- Use manual transition control with continue button
- Move all JSON processing to SourceBodyUpload
- Create actual downloadable .json files
- Make entire FileBadge area clickable for deletion
- Use fixed height containers for consistent layout
- Test all functionality in actual form workflows
- Ensure proper separation of concerns between components

This phase provides a clean, user-controlled approach to JSON upload and mapping integration while maintaining proper separation of concerns and improving the overall user experience.

## Phase 12: Fix Flat Array Rendering Issue (CRITICAL)

**Goal**: Fix JsonRenderer to properly render all array items in flat arrays instead of only the first item.

### 12.1 Current Problem

**Issue**: JsonRenderer only renders the first array item when processing flat arrays like `[{item0}, {item1}, {item2}, ...]`

**Root Cause**: JsonRenderer architecture assumes hierarchical JSON structures with parent-child relationships, but flat arrays have sibling relationships at the same level.

**Current Logic**:
1. Find root basePath (shortest path = "0")
2. Look for children of "0" (paths starting with "0." or "0[")
3. Array items 1-9 are siblings of "0", not children
4. Result: Only item "0" gets rendered

### 12.2 Required Changes

#### 12.2.1 JsonRenderer Component Updates

**File**: `components/json/interface/json-renderer.tsx`

**Current Logic (lines 45-65)**:
```typescript
const rootBasePath = Object.keys(basePaths)
  .sort((a, b) => a.length - b.length)[0] // Selects shortest path as root

const renderAccordionWithChildren = (basePath: string): React.ReactNode => {
  const directChildren = findNestedLevel(basePath, allBasePaths)
  // ... renders only children of basePath
}
```

**New Logic**:
```typescript
const rootBasePaths = getRootBasePaths(basePaths) // Get all root-level paths

const renderAccordionWithChildren = (basePath: string): React.ReactNode => {
  const directChildren = findNestedLevel(basePath, allBasePaths)
  // ... renders children of basePath
}

const renderRootLevel = (): React.ReactNode => {
  return rootBasePaths.map(basePath => {
    const result = renderAccordionWithChildren(basePath)
    return result
  })
}
```

**New Function**: `getRootBasePaths(basePaths)`
- Purpose: Identify all root-level basePaths in flat arrays
- Logic: Find basePaths that are not children of any other basePath
- Return: Array of root-level basePath strings

#### 12.2.2 New Utility Function

**File**: `utils/json/rendering-utils.tsx`

**Add Function**:
```typescript
export function getRootBasePaths(basePaths: Record<string, BasePathData>): string[] {
  const allPaths = Object.keys(basePaths)
  const rootPaths: string[] = []
  
  for (const path of allPaths) {
    const isChild = allPaths.some(otherPath => 
      otherPath !== path && 
      (otherPath.startsWith(path + '.') || otherPath.startsWith(path + '['))
    )
    
    if (!isChild) {
      rootPaths.push(path)
    }
  }
  
  return rootPaths.sort((a, b) => a.length - b.length)
}
```

#### 12.2.3 Update JsonRenderer Rendering Logic

**File**: `components/json/interface/json-renderer.tsx`

**Replace Current Logic (lines 45-65)**:
```typescript
const rootBasePath = Object.keys(basePaths)
  .sort((a, b) => a.length - b.length)[0]

if (!rootBasePath) return { rootAccordion: null, nestedAccordions: [] }

const allBasePaths = Object.keys(basePaths)

const renderAccordionWithChildren = (basePath: string): React.ReactNode | { accordion: React.ReactNode, progressBar: React.ReactNode } => {
  // ... existing logic
}

const rootResult = renderAccordionWithChildren(rootBasePath)
```

**With New Logic**:
```typescript
const rootBasePaths = getRootBasePaths(basePaths)

if (rootBasePaths.length === 0) return { rootAccordion: null, nestedAccordions: [] }

const allBasePaths = Object.keys(basePaths)

const renderAccordionWithChildren = (basePath: string): React.ReactNode | { accordion: React.ReactNode, progressBar: React.ReactNode } => {
  // ... existing logic
}

const renderRootLevel = (): React.ReactNode => {
  return rootBasePaths.map(basePath => {
    const result = renderAccordionWithChildren(basePath)
    if (result && typeof result === 'object' && 'accordion' in result) {
      return result.accordion
    }
    return result
  })
}

const rootAccordions = renderRootLevel()
```

#### 12.2.4 Update JsonRenderer Return Logic

**File**: `components/json/interface/json-renderer.tsx`

**Replace Current Return (lines 80-90)**:
```typescript
return {
  rootAccordion: rootResult && typeof rootResult === 'object' && 'accordion' in rootResult 
    ? rootResult.accordion 
    : rootResult,
  nestedAccordions: []
}
```

**With New Return**:
```typescript
return {
  rootAccordion: rootAccordions.length === 1 ? rootAccordions[0] : null,
  nestedAccordions: rootAccordions.length > 1 ? rootAccordions : []
}
```

### 12.3 Files to Modify

1. **`components/json/interface/json-renderer.tsx`**:
   - Add import for `getRootBasePaths`
   - Replace root basePath selection logic
   - Add `renderRootLevel` function
   - Update return logic for multiple root accordions

2. **`utils/json/rendering-utils.tsx`**:
   - Add `getRootBasePaths` function

### 12.4 Expected Behavior After Fix

**Flat Array `[{item0}, {item1}, {item2}, ...]`**:
- `getRootBasePaths` returns `["0", "1", "2", ...]`
- `renderRootLevel` renders all 10 accordions
- All array items are displayed in the UI

**Hierarchical Object `{user: {profile: {name: "John"}}}`**:
- `getRootBasePaths` returns `["user"]`
- `renderRootLevel` renders single root accordion
- Existing behavior preserved

**Mixed Structure `{users: [{name: "John"}, {name: "Jane"}]}`**:
- `getRootBasePaths` returns `["users"]`
- `renderRootLevel` renders single root accordion
- Array items rendered as children of "users"

### 12.5 Validation Criteria

**Success Indicators**:
- All array items in flat arrays are rendered
- Hierarchical objects continue to work as before
- Mixed structures work correctly
- No performance degradation
- No breaking changes to existing functionality

**Testing Approach**:
- Test with flat array JSON (like source_rows.json)
- Test with hierarchical object JSON
- Test with mixed structure JSON
- Verify all array items are visible and interactive
- Confirm existing functionality still works