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

## Phase 10: Complete Architectural Redesign - Separation of Concerns (CRITICAL)

**Goal**: Implement proper separation of concerns by restructuring the entire JSON mapping architecture to eliminate circular dependencies, infinite re-render loops, and performance issues through clean state management and component responsibilities.

### 10.1 Current Architecture Problems

**Fundamental Issues Identified**:

1. **Separation of Concerns Violations**:
   - Mappers managing UI state (`isSelected`, `isExpanded`)
   - Mappers handling user interactions (`handlePathSelect`)
   - Complex state management with 5+ state variables
   - Mixing data processing with UI concerns

2. **Circular Dependencies**:
   - `fullPaths` in `useEffect` dependencies causing infinite loops
   - `handlePathSelect` updating `fullPaths` triggering unnecessary recalculations
   - State update chain reactions causing performance issues

3. **Performance Issues**:
   - Statistics recalculated on every path selection
   - Unnecessary re-renders due to state management problems
   - Console logs cluttering production code

4. **Architectural Confusion**:
   - Wrong source of truth for UI state
   - Complex state synchronization between components
   - Unclear component responsibilities

### 10.2 Correct Architecture Design

#### **10.2.1 Mappers (Data Processing Only)**
**Responsibilities**:
- Process JSON data and generate `basePaths` and `fullPaths`
- Apply mappings (object, datapoint, variable)
- Calculate statistics
- NO UI state management
- NO user interaction handling

**State Variables** (Only 3):
- `jsonProcessed` - Step 1 completion flag
- `mappingsProcessed` - Step 2 completion flag  
- `statisticsProcessed` - Step 3 completion flag

**Data Types** (Modify existing):
- Remove `isExpanded` from `BasePathData` interface
- Remove `isSelected` from `FullPathData` interface
- Keep existing `basePaths` and `fullPaths` structure without UI state

#### **10.2.2 JsonExplorer (UI State Management)**
**Responsibilities**:
- Manage all UI state (`isExpanded`, `isSelected`)
- Handle user interactions (`handlePathSelect`, `handleNodeExpand`)
- Enrich data with UI state before passing to renderer
- Coordinate between mappers and renderer

**State Variables**:
- `basePathsStates` - Record with path and `isExpanded` boolean for controlling accordions
- `fullPathsStates` - Record with path, type, and `isSelected` boolean for controlling json nodes

**Interface**:
```typescript
interface BasePathState {
  path: string
  isExpanded: boolean
}

interface FullPathState {
  path: string
  type: 'key' | 'value'
  isSelected: boolean
}
```

#### **10.2.3 JsonRenderer (Rendering Only)**
**Responsibilities**:
- Receive enriched data from explorer
- Handle rendering optimizations using useMemo
- Render accordions and nodes
- NO internal state management

**Performance Optimization**:
- Uses `useMemo` for `renderedContent` calculation
- Automatic dependency tracking for optimal performance
- Controlled by parent props (no internal state needed)

#### **10.2.4 JsonAccordion & JsonNode (Controlled Components)**
**Responsibilities**:
- Pure rendering components
- Fully controlled by parent
- No internal state management
- Receive all state through props

### 10.2.5 Complete Data Flow and Function Sequence

#### **Initial Data Processing Flow**

**Step 1: Data Ingestion (JsonResultMapper)**
- Function: `generatePathData(data)` receives raw JSON data
- Process: Traverses JSON structure, creates basePaths and fullPaths arrays
- Output: Clean data structures without any UI state
- State: Sets `jsonProcessed = true`

**Step 2: Mapping Application (All Mappers)**
- **JsonResultMapper**: `assignMappingToFullPath(mappings, fullPaths)` processes object/datapoint mappings
- **JsonRequestMapper**: `assignVariableToFullPath(mappingSetup, fullPaths)` processes request variable mappings
- **JsonResponseMapper**: `assignVariableToFullPath(mappingSetup, fullPaths)` processes response variable mappings
- Process: Matches JSON paths to mapping patterns, assigns mapping IDs to fullPaths
- Output: Updated fullPaths with objectMappings, datapointMappings, variableMappings arrays
- State: Sets `mappingsProcessed = true`

**Step 3: Statistics Calculation (JsonResultMapper)**
- Function: `getBasePathStatus(basePaths, fullPaths)` determines status for each basePath
- Function: `getBasePathStatistics(updatedBasePaths, fullPaths)` calculates statistics
- Process: Aggregates mapping counts and status information for progress tracking
- Output: BasePaths with status and statistics, fullPaths with mapping assignments
- State: Sets `statisticsProcessed = true`

**Step 4: Data Handoff (JsonResultMapper → JsonExplorer)**
- Function: `onProcessedData` callback passes processed data
- Data Structure: `{basePaths, fullPaths, statistics, dataReady: true}`
- Process: Mapper calls callback with clean data, no UI state included

#### **UI State Management Flow**

**Step 5: UI State Initialization (JsonExplorer)**
- Function: `useState` hooks create empty state objects
- Process: Initializes basePathsStates and fullPathsStates as empty records
- Data Structure: `Record<string, BasePathState>` and `Record<string, FullPathState>`
- Purpose: Separate UI state from data processing

**Step 6: State Derivation (JsonExplorer)**
- **Obsolete**: `fullPaths.find(path => path.isSelected)` - current derivation from fullPaths
- **New**: `Object.values(fullPathsStates).find()` derives selectedPath and selectedType
- Process: Searches fullPathsStates for isSelected true, extracts path and type
- Output: Current selected path and type for rendering
- Purpose: Single source of truth for selection state

#### **User Interaction Flow**

**Step 7: Path Selection (JsonExplorer)**
- **Obsolete**: `setFullPaths(fullPaths.map(...))` - current selection in fullPaths
- **New**: `handlePathSelect(path, parentPaths, type)` manages selection
- Process: Clears all previous selections, sets new selection in fullPathsStates
- Data Structure: Updates `fullPathsStates[path:type] = {path, type, isSelected: true}`
- Trigger: User clicks on JSON node in renderer

**Step 8: Accordion Expansion (JsonExplorer)**
- **Obsolete**: `setExpandedPaths(new Set(...))` - current expansion in Set
- **New**: `handleAccordionExpand(path)` manages accordion expansion
- Process: Toggles isExpanded state for specific basePath
- Data Structure: Updates `basePathsStates[path] = {path, isExpanded: boolean}`
- Trigger: User clicks on accordion header in renderer

#### **Rendering Flow**

**Step 9: Data Preparation (JsonExplorer → JsonRenderer)**
- Function: Props passing with both data and UI state
- Data Structure: `{basePaths, fullPaths, basePathsStates, fullPathsStates}`
- Process: Passes clean data and separate UI state to renderer
- Purpose: Renderer receives everything needed for rendering

**Step 10: State Derivation (JsonRenderer)**
- **Obsolete**: `expandedPaths` prop - current direct prop passing
- **New**: `useMemo` derives expandedPaths from basePathsStates
- Process: Converts basePathsStates record to Set of expanded paths
- Data Structure: `Set<string>` of expanded path strings
- Purpose: Converts UI state to format expected by rendering functions

**Step 11: Rendering Optimization (JsonRenderer)**
- Function: `useMemo` for `renderedContent` calculation
- Process: Automatically recalculates when dependencies change
- Dependencies: `[data, basePaths, fullPaths, basePathsStates, fullPathsStates, onNodeClick, onAccordionExpand, expandedPaths]`
- Purpose: Optimal performance with automatic dependency tracking

**Step 12: Component Rendering (JsonRenderer → JsonAccordion/JsonNode)**
- Function: `renderBase`, `renderNestedLevel` call rendering utilities
- Process: Passes data and UI state to individual components
- Data Structure: Props include `isExpanded`, `isSelected` from state objects
- Purpose: Controlled components receive all state from parent

#### **Data Structure Flow**

**Clean Data (Mappers)**:
- BasePaths: `{path: {status, statistics}}` - no UI state
- FullPaths: `{path, type, value, status, objectMappings, datapointMappings, variableMappings}` - no UI state

**UI State (JsonExplorer)**:
- BasePathsStates: `{path: {path, isExpanded}}` - accordion control
- FullPathsStates: `{path:type: {path, type, isSelected}}` - node selection control

**Rendering Optimization (JsonRenderer)**:
- useMemo: `renderedContent` - automatic performance optimization
- Dependencies: `[data, basePaths, fullPaths, basePathsStates, fullPathsStates, onNodeClick, onAccordionExpand, expandedPaths]`

**Component Props (JsonAccordion/JsonNode)**:
- Receives: `isExpanded`, `isSelected` from parent state
- Receives: `onClick` callbacks to communicate with parent
- No internal state management
- Fully controlled by parent components

#### **User Interaction Callback Flow**

**When User Clicks JsonNode**:
- JsonNode calls `onClick(path, type)` callback
- JsonExplorer updates `fullPathsStates` directly
- JsonNode receives new `isSelected` prop and re-renders

**When User Clicks JsonAccordion**:
- JsonAccordion calls `onAccordionExpand(path)` callback
- JsonExplorer updates `basePathsStates` directly
- JsonAccordion receives new `isExpanded` prop and re-renders

**Standard React Pattern**:
1. User clicks component
2. Component calls parent callback
3. Parent updates state directly
4. Parent re-renders with new state
5. Component receives new props and re-renders

#### **Function Responsibilities Summary**

**JsonResultMapper Functions**:
- `generatePathData()` - creates clean data structures
- `assignMappingToFullPath()` - applies mappings to data
- `getBasePathStatus()` - determines path statuses
- `getBasePathStatistics()` - calculates statistics

**JsonExplorer Functions**:
- `handlePathSelect()` - manages node selection state
- `handleAccordionExpand()` - manages accordion expansion state
- State derivation functions - extract current UI state

**JsonRenderer Functions**:
- `useMemo` for `expandedPaths` - derive expanded paths from basePathsStates
- `useMemo` for `renderedContent` - optimize rendering performance
- Rendering utilities - create components with proper state

**Child Component Functions**:
- Pure rendering functions - display data with received state
- No state management functions - fully controlled

### 10.3 Required Changes

#### **10.3.1 Type Definitions (types/json.ts)**

**File**: `types/json.ts`

**Change 1.1: Remove `isExpanded` from BasePathData interface (line 9)**
- **Current**: `isExpanded?: boolean`
- **Action**: Delete line 9 completely
- **Result**: BasePathData will only contain `status` and `statistics`

**Change 1.2: Remove `isSelected` from FullPathData interface (line 23)**
- **Current**: `isSelected?: boolean`
- **Action**: Delete line 23 completely
- **Result**: FullPathData will only contain data properties, no UI state

**Change 1.3: Add BasePathState interface (after line 16)**
- **Action**: Insert new interface after BasePathData
- **Code**:
```typescript
export interface BasePathState {
  path: string
  isExpanded: boolean
}
```

**Change 1.4: Add FullPathState interface (after BasePathState)**
- **Action**: Insert new interface after BasePathState
- **Code**:
```typescript
export interface FullPathState {
  path: string
  type: 'key' | 'value'
  isSelected: boolean
}
```

**Change 1.5: Remove RenderingState interface (not needed)**
- **Action**: Skip this change - JsonRenderer uses useMemo for performance, no internal state needed
- **Reasoning**: Current JsonRenderer is already optimized with useMemo and controlled by parent props

#### **10.3.2 Mappers (All Three) - JsonResultMapper Example**

**File**: `components/json/response/json-result-mapper.tsx`

**Change 2.1: Remove all console.log statements (lines 74-79, 81, 94-100, 102-108, 110-116, 118-124, 126-132, 134-140, 142-148, 150-156, 158-164, 166-172, 174-180, 182-188, 190-196, 198-204, 206-212, 214-220, 222-228, 230-236, 241-245, 252-258, 261-268, 280)**
- **Action**: Delete all lines containing `console.log` statements
- **Total lines to delete**: 28 lines

**Change 2.2: Remove `isSelected` from fullPaths state type (line 85)**
- **Current**: `isSelected?: boolean,`
- **Action**: Delete `isSelected?: boolean,` from the type definition
- **Result**: fullPaths will not contain isSelected property

**Change 2.3: Remove `statusesProcessed` state variable (line 91)**
- **Current**: `const [statusesProcessed, setStatusesProcessed] = useState(false)`
- **Action**: Delete entire line 91

**Change 2.4: Remove `handlePathSelect` function (lines 240-250)**
- **Action**: Delete entire function including the useCallback wrapper
- **Lines to delete**: 11 lines (240-250)

**Change 2.5: Remove `onPathSelect` from props interface (line 59)**
- **Current**: `onPathSelect: (path: string | undefined, parentPaths?: string[], type?: 'key' | 'value') => void`
- **Action**: Delete entire line 59

**Change 2.6: Remove `onPathSelect` from onProcessedData callback (line 275)**
- **Current**: `onPathSelect: handlePathSelect`
- **Action**: Delete `onPathSelect: handlePathSelect,` from the return object

**Change 2.7: Remove `isSelected` from onProcessedData fullPaths type (line 56)**
- **Current**: `isSelected?: boolean,`
- **Action**: Delete `isSelected?: boolean,` from the fullPaths type in onProcessedData

**Change 2.8: Remove Step 3 useEffect (lines 189-212)**
- **Action**: Delete entire useEffect block for statuses processing
- **Lines to delete**: 24 lines (189-212)

**Change 2.9: Update Step 4 useEffect dependencies (line 237)**
- **Current**: `}, [statusesProcessed, statisticsProcessed, fullPaths])`
- **Action**: Change to `}, [mappingsProcessed, statisticsProcessed, fullPaths])`

**Change 2.10: Update Step 4 useEffect condition (line 221)**
- **Current**: `if (statusesProcessed && !statisticsProcessed)`
- **Action**: Change to `if (mappingsProcessed && !statisticsProcessed)`

**Change 2.11: Update Step 4 useEffect to call getBasePathStatus first (lines 223-230)**
- **Current**: Direct call to `getBasePathStatistics`
- **Action**: Add call to `getBasePathStatus` first, then `getBasePathStatistics`
- **New code**:
```typescript
const statusedBasePaths = getBasePathStatus(basePaths, fullPaths)
const statisticsBasePaths = getBasePathStatistics(statusedBasePaths, fullPaths)
setBasePaths(statisticsBasePaths)
```

#### **10.3.3 JsonExplorer**

**File**: `components/json/interface/json-explorer.tsx`

**Change 3.1: Add BasePathState and FullPathState imports (after line 13)**
- **Action**: Add imports after existing imports
- **Code**:
```typescript
import { BasePathState, FullPathState } from '../../../types/json'
```

**Change 3.2: Add basePathsStates state management (after line 55)**
- **Action**: Add new state after searchQuery
- **Code**:
```typescript
const [basePathsStates, setBasePathsStates] = useState<Record<string, BasePathState>>({})
```

**Change 3.3: Add fullPathsStates state management (after basePathsStates)**
- **Action**: Add new state after basePathsStates
- **Code**:
```typescript
const [fullPathsStates, setFullPathsStates] = useState<Record<string, FullPathState>>({})
```

**Change 3.4: Remove selectedPathData derivation (lines 61-63)**
- **Action**: Delete lines 61-63 completely
- **Current code to delete**:
```typescript
const selectedPathData = fullPaths.find(path => path.isSelected)
const selectedPath = selectedPathData?.path
const selectedType = selectedPathData?.type
```

**Change 3.5: Create selectedPath and selectedType from fullPathsStates (after line 60)**
- **Action**: Add new derivation logic
- **Code**:
```typescript
const selectedPathState = Object.values(fullPathsStates).find(state => state.isSelected)
const selectedPath = selectedPathState?.path
const selectedType = selectedPathState?.type
```

**Change 3.6: Remove console.log statements (lines 65-71)**
- **Action**: Delete entire console.log block
- **Lines to delete**: 7 lines (65-71)

**Change 3.7: Add handlePathSelect function (after line 82)**
- **Action**: Add new function to manage fullPathsStates
- **Code**:
```typescript
const handlePathSelect = useCallback((path: string | undefined, parentPaths?: string[], type?: 'key' | 'value') => {
  setFullPathsStates(prev => {
    const newStates = { ...prev }
    // Clear all previous selections
    Object.keys(newStates).forEach(key => {
      newStates[key] = { ...newStates[key], isSelected: false }
    })
    // Set new selection
    if (path && type) {
      const stateKey = `${path}:${type}`
      newStates[stateKey] = { path, type, isSelected: true }
    }
    return newStates
  })
}, [])
```

**Change 3.8: Add handleAccordionExpand function (after handlePathSelect)**
- **Action**: Add new function to manage basePathsStates
- **Code**:
```typescript
const handleAccordionExpand = useCallback((path: string) => {
  setBasePathsStates(prev => ({
    ...prev,
    [path]: { path, isExpanded: !prev[path]?.isExpanded }
  }))
}, [])
```

**Change 3.9: Update JsonRenderer props (around line 280)**
- **Action**: Replace expandedPaths, selectedPath, selectedType with basePathsStates, fullPathsStates
- **Current**: `expandedPaths={expandedPaths} selectedPath={selectedPath} selectedType={selectedType}`
- **New**: `basePathsStates={basePathsStates} fullPathsStates={fullPathsStates}`

#### **10.3.4 JsonRenderer**

**File**: `components/json/interface/json-renderer.tsx`

**Change 4.1: Add BasePathState and FullPathState imports (after line 10)**
- **Action**: Add imports after existing imports
- **Code**:
```typescript
import { BasePathState, FullPathState } from '../../types/json'
```

**Change 4.2: Add basePathsStates and fullPathsStates to props interface (around line 36)**
- **Action**: Replace expandedPaths, selectedPath, selectedType with new props
- **Remove**: `expandedPaths?: Set<string>`, `selectedPath?: string`, `selectedType?: 'key' | 'value'`
- **Add**: `basePathsStates?: Record<string, BasePathState>`, `fullPathsStates?: Record<string, FullPathState>`

**Change 4.3: Remove renderingState management (not needed)**
- **Action**: Skip this change - JsonRenderer uses useMemo for performance optimization
- **Reasoning**: Current JsonRenderer is already optimized with useMemo and controlled by parent props

**Change 4.4: Remove all console.log statements**
- **Action**: Find and delete all console.log statements throughout the file
- **Note**: This will require scanning the entire file for console.log statements

**Change 4.5: Update expandedPaths logic (around line 90)**
- **Current**: Uses expandedPaths prop
- **Action**: Change to derive from basePathsStates
- **New code**:
```typescript
const expandedPaths = useMemo(() => {
  const expanded = new Set<string>()
  Object.values(basePathsStates || {}).forEach(state => {
    if (state.isExpanded) expanded.add(state.path)
  })
  return expanded
}, [basePathsStates])
```

**Change 4.6: Update renderedContent useMemo dependencies (around line 117)**
- **Current**: `[data, basePaths, fullPaths, selectedPath, onNodeClick, onNodeExpand, expandedPaths]`
- **Action**: Replace selectedPath with basePathsStates and fullPathsStates, keep expandedPaths
- **New dependencies**: `[data, basePaths, fullPaths, basePathsStates, fullPathsStates, onNodeClick, onAccordionExpand, expandedPaths]`
- **Reasoning**: selectedPath is derived from fullPathsStates, so we need the source state instead

#### **10.3.5 JsonAccordion & JsonNode**

**File**: `components/accordions/json-accordion.tsx`

**Change 5.1: Remove all internal state management**
- **Action**: Find and remove all useState calls for internal state
- **Note**: This requires scanning the entire file for useState usage

**Change 5.2: Add isExpanded prop to interface (around line 15)**
- **Action**: Add new prop to JsonNodeData interface
- **Code**: `isExpanded?: boolean`

**Change 5.3: Remove all console.log statements**
- **Action**: Find and delete all console.log statements throughout the file

**File**: `components/json/node/json-node.tsx`

**Change 5.4: Remove all internal state management**
- **Action**: Find and remove all useState calls for internal state
- **Note**: This requires scanning the entire file for useState usage

**Change 5.5: Add isSelected prop to interface (around line 16)**
- **Action**: Add new prop to JsonNodeProps interface
- **Code**: `isSelected?: boolean`

**Change 5.6: Remove all console.log statements**
- **Action**: Find and delete all console.log statements throughout the file

#### **10.3.6 Mapping Utilities (mapping-utils.tsx)**

**File**: `utils/json/mapping-utils.tsx`

**Change 6.1: Remove all console.log statements**
- **Action**: Find and delete all console.log statements throughout the file
- **Note**: This requires scanning the entire file for console.log statements

**Change 6.2: Keep optimization logic for assignMappingToFullPath and assignVariableToFullPath**
- **Action**: Ensure these functions only create new arrays when changes are made
- **Note**: Current implementation should already have this optimization

#### **10.3.7 Rendering Utilities (rendering-utils.tsx)**

**File**: `utils/json/rendering-utils.tsx`

**Change 7.1: Remove all console.log statements**
- **Action**: Find and delete all console.log statements throughout the file
- **Note**: This requires scanning the entire file for console.log statements

**Change 7.2: Optimize rendering functions for better performance**
- **Action**: Review and optimize any performance bottlenecks in rendering functions
- **Note**: This may require performance analysis of the current functions

### 10.4 Implementation Steps

#### **Step 1: Update Type Definitions**
- Modify `types/json.ts` to remove UI state from data interfaces
- Add new UI state interfaces
- Add rendering state interface

#### **Step 2: Clean Up Mappers**
- Remove all console.log statements
- Remove UI state from fullPaths
- Remove user interaction handling
- Simplify to 3 processing states only
- Clean data generation only

#### **Step 3: Update JsonExplorer**
- Add UI state management (`basePathsStates`, `fullPathsStates`)
- Add enrichment functions
- Handle all user interactions
- Remove console.log statements
- Update prop interfaces

#### **Step 4: Update JsonRenderer**
- Update prop interfaces to use basePathsStates and fullPathsStates
- Remove console.log statements
- Update useMemo dependencies for optimal performance
- Keep existing useMemo optimization approach

#### **Step 5: Update Child Components**
- Make JsonAccordion and JsonNode fully controlled
- Remove internal state management
- Update prop interfaces
- Remove console.log statements

#### **Step 6: Clean Up Utilities**
- Remove console.log statements from all utility functions
- Optimize performance where needed

### 10.5 Expected Benefits

#### **10.5.1 Performance Improvements**
- No circular dependencies
- No infinite re-render loops
- Minimal re-renders with proper state management
- Better performance with optimized rendering

#### **10.5.2 Maintainability**
- Clear separation of concerns
- Each component has single responsibility
- Easier debugging with clear state ownership
- Cleaner, more readable code

#### **10.5.3 Architecture Benefits**
- Proper data flow from mappers → explorer → renderer
- UI state managed in one place (JsonExplorer)
- Rendering optimized with useMemo (JsonRenderer)
- Data processing isolated in mappers

### 10.6 Files to Modify

**Type Definitions**:
1. `types/json.ts`

**Mapper Components**:
2. `components/json/response/json-result-mapper.tsx`
3. `components/json/request/json-request-mapper.tsx`
4. `components/json/response/json-response-mapper.tsx`

**Interface Components**:
5. `components/json/interface/json-explorer.tsx`
6. `components/json/interface/json-renderer.tsx`

**Child Components**:
7. `components/accordions/json-accordion.tsx`
8. `components/json/node/json-node.tsx`

**Utility Files**:
9. `utils/json/mapping-utils.tsx`
10. `utils/json/rendering-utils.tsx`

### 10.7 Validation Criteria

**Success Indicators**:
- No "Maximum update depth exceeded" errors
- No circular dependency warnings
- Components render once and complete processing
- Clear separation of concerns
- UI state managed in JsonExplorer only
- Data processing isolated in mappers
- Rendering optimized with useMemo in JsonRenderer
- No console.log statements in production code
- Better performance with minimal re-renders

**Testing Approach**:
- Monitor terminal output for infinite re-renders
- Verify component responsibilities are clear
- Test UI state management in JsonExplorer
- Test data processing in mappers
- Test rendering performance with useMemo optimization in JsonRenderer
- Confirm no console.log statements remain

### 10.8 Critical Implementation Notes

**DO NOT**:
- Mix UI state with data state
- Handle user interactions in mappers
- Manage internal state in child components
- Leave console.log statements in production code
- Create circular dependencies between components

**DO**:
- Keep mappers focused on data processing only
- Manage all UI state in JsonExplorer
- Make child components fully controlled
- Implement proper separation of concerns
- Clean up all debugging code
- Test each component's responsibility independently

This architectural redesign will create a clean, maintainable, and performant system that properly separates data processing, UI state management, and rendering concerns.

## Phase 11: Source Body Upload Component Redesign

### Analysis of Current Mappers

#### JsonResultMapper
- **Data Input**: `data: any` prop - expects raw JSON data
- **Processing**: Uses `generatePathData(data)` to create basePaths and fullPaths
- **Mapping Type**: Object/datapoint mappings from source-mappings.json
- **Output**: `onProcessedData` callback with processed data and statistics

#### JsonRequestMapper  
- **Data Input**: `data: any` prop - expects raw JSON data
- **Processing**: Uses `generatePathData(data)` + variable mapping functions
- **Mapping Type**: Request variable mappings from template JSONs
- **Output**: `onProcessedData` callback with processed data and statistics

#### JsonResponseMapper
- **Data Input**: `data: any` prop - expects raw JSON data  
- **Processing**: Uses `generatePathData(data)` + response variable mapping functions
- **Mapping Type**: Response variable mappings from template JSONs
- **Output**: `onProcessedData` callback with processed data and statistics


### Redesigned Component Architecture

#### Simple 3-Section Layout
```typescript
interface SourceBodyUploadProps {
  label?: string
  description?: string
  maxFiles?: number
  maxSize?: number
  className?: string
  onJsonDataChange: (data: {
    files: Array<{id: string, name: string, content: any}>
    activeFile?: string
    textData?: string
    combinedData?: any
  }) => void
}
```

#### Three Main Sections
1. **Get by Request Section**
   - PrimaryButton with dynamic text based on mapper type
   - Simple loading state during request
   - Error display for failed requests
   - No complex logic - just button and states

2. **Upload File Section**  
   - FormDragDrop component (existing)
   - FileBadge components for uploaded files (existing)
   - Simple file management without complex state

3. **Paste as Text Section**
   - FormTextarea component (existing)
   - JSON validation and parsing
   - Simple edit/view mode toggle

#### State Management (Simplified)
```typescript
// Only essential state
const [jsonFiles, setJsonFiles] = useState<JsonFile[]>([])
const [activeFileId, setActiveFileId] = useState<string | null>(null)
const [textInput, setTextInput] = useState<string>("")
const [isRequesting, setIsRequesting] = useState<boolean>(false)
const [requestError, setRequestError] = useState<string>("")
```

### Implementation Plan

#### Step 1: Create New Component Structure
- Create new `source-body-upload.tsx` in `components/json/interface/` directory
- Implement proper tab system with 3 tabs: "Get by Request", "Upload File", "Paste as Text"
- Use existing components: FormDragDrop, FormTextarea, PrimaryButton, FileBadge
- Implement clean state management with only 5 state variables

#### Step 2: Implement Core Features
- **Get by Request**: Simple button with loading/error states
- **Upload File**: Use FormDragDrop with file management
- **Paste as Text**: Use FormTextarea with JSON validation
- **JSON Rendering**: Single JsonView component for all sections

#### Step 3: Data Flow Integration
- Single `onJsonDataChange` callback for all input methods
- Consistent data structure: `{files, activeFile, textData, combinedData}`
- Automatic JSON parsing and validation
- Clean integration with existing mapper components

#### Step 4: Remove Cursor Issues
- No custom tab components
- No complex hover states
- Use existing UI components without modifications
- Simple CSS transitions only

#### Step 5: Testing and Validation
- Test with all three mapper types
- Verify data flow from input to mapper
- Test error handling and edge cases
- Performance testing for large JSON files

### Expected Benefits
- **Simplified Architecture**: 80% reduction in component complexity
- **Better Performance**: No cursor flickering or excessive re-renders
- **Easier Maintenance**: Clear separation of concerns
- **Better Integration**: Seamless integration with existing mappers
- **Reusable Components**: Leverages existing FormDragDrop, FormTextarea, etc.