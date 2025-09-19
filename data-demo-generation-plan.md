# Data Implementation Plan

## Overview

This document outlines the comprehensive plan for generating backend-compatible data structures based on the `source_rows.json` database export. The goal is to create two main data files: `sources.json` (list view) and individual `source.json` files (detail view) that can populate both the sources page and source detail pages.

## Analysis of source_rows.json Structure

### Database Table Fields

**Core Fields:**
- `id` - Unique identifier (UUID)
- `name` - Source display name
- `description` - Source description
- `source_type` - Type of source (Dataset, Agent, Integration, ML model, LLM)
- `created_at` - Creation timestamp
- `updated_at` - Last update timestamp

**Configuration Fields:**
- `max_concurrent_runs` - Maximum concurrent execution limit
- `delivery_type` - Delivery method (Endpoint, Webhook)
- `timeout` - Request timeout in seconds
- `auth_key` - Authentication key (optional)
- `owner_org_id` - Organization owner UUID
- `max_retries` - Maximum retry attempts
- `setup` - Source-specific setup configuration

**Template Fields:**
- `run_request_template` - JSON string for run request template
- `status_request_template` - JSON string for status check template
- `delivery_request_template` - JSON string for delivery template


**Metadata Fields:**
- `average_run_duration` - Average execution time in seconds
- `deleted_at` - Soft delete timestamp
- `idx` - Database index

### Template Structure Analysis

**Template Structure (stored as JSON strings):**
- `url` - API endpoint URL
- `body` - Request body (can be object or string with variables)
- `method` - HTTP method (GET, POST, etc.)
- `headers` - HTTP headers object

**Universal Template Example:**
```json
{
  "url": "https://api.example.com/endpoint",
  "body": {},
  "method": "POST",
  "headers": {
    "Content-Type": "application/json",
    "Authorization": "Bearer token"
  }
}
```

**Variable Substitution Patterns by Template Type:**

**Run Request Template:**
- `<<$run_setup>>` - Run setup parameters (filters, criteria)
- `<<$max_objects>>` - Maximum objects to fetch
- `<<$id>>` - Internal run ID
- `<<$webhook_url>>` - Webhook URL for results delivery

**Status Request Template:**
- `<<$run_external_id>>` - External run ID to check status

**Delivery Request Template:**
- `<<$run_external_id>>` - External run ID for download/retrieval

**Additional Variables:**
- `<<$webhook_url>>` - Webhook URL for delivery
- `<<$id>>` - Internal run ID

## Implementation Plan

### Phase 0: Data Folder Organization and Migration

**Purpose:** Prepare the data folder structure and migrate existing files to their proper locations

**Target Folder Structure:**
```
data/
├── 1.0/                       # 📦 Legacy/Outdated Files
│   ├── sources.json           # Old simple structure
│   ├── activities.json        # Old simple structure
│   ├── workflows.json         # Old simple structure
│   ├── objecttypes.json       # Old generic structure
│   ├── datapoints.json        # Old generic structure
│   ├── guardrails.json        # Old generic structure
│   ├── usecases.json          # Old generic structure
│   ├── source-mappings.json   # Old complex mappings
│   ├── source-response.json   # Old API responses
│   └── workflow-phases.json   # Not relevant
│
├── sources/                   # 🆕 New structure (prepare folders)
│   ├── sources.json           # Will be generated in Phase 1
│   ├── linkedin-people-dataset.json
│   ├── ai-company-researcher.json
│   ├── esg-agent.json
│   ├── job-market-scraper.json
│   ├── e-commerce-scraper.json
│   ├── event-management.json
│   ├── automotive-database.json
│   └── real-estate-api.json
│
├── activities/                # 🆕 New structure (prepare folders)
│   ├── activities.json        # Will be generated in Phase 3
│   ├── company-enrichment.json
│   ├── lead-extraction.json
│   ├── prospect-sourcing.json
│   ├── sales-forecasting-training.json
│   ├── revenue-prediction.json
│   └── ... (other activities)
│
├── workflows/                 # 🆕 New structure (prepare folders)
│   ├── workflows.json         # Will be generated in Phase 4
│   ├── lead-generation-pipeline.json
│   ├── data-processing-pipeline.json
│   ├── ml-training-pipeline.json
│   └── ... (other workflows)
│
├── object-types.json          # 🆕 Will be generated in Phase 2
├── datapoints.json            # 🆕 Will be generated in Phase 2
├── factors.json               # 🆕 Will be generated in Phase 2
├── targets.json               # 🆕 Will be generated in Phase 2
│
├── dashboard/                 # ✅ Keep current structure
│   ├── dashboard.json
│   └── dashboard-full.json
│
├── navigation.json            # ✅ Keep - up-to-date UI navigation
│
└── samples/                   # 🆕 Consolidated sample data
    ├── companies.json         # Moved from root
    ├── persons.json           # Moved from root
    ├── jobposts.json          # Moved from root
    ├── documents.json         # Moved from root
    ├── organizations.json     # Moved from root
    └── ... (other sample data)
```

**Step 1: Create 1.0 Legacy Folder**
- Create `data/1.0/` folder
- Move outdated files to `1.0/` folder:
  - `sources.json` (old simple structure)
  - `activities.json` (old simple structure)
  - `workflows.json` (old simple structure)
  - `objecttypes.json` (old generic structure)
  - `datapoints.json` (old generic structure)
  - `guardrails.json` (old generic structure)
  - `usecases.json` (old generic structure)
  - `source-mappings.json` (old complex mappings)
  - `source-response.json` (old API responses)
  - `workflow-phases.json` (not relevant)

**Step 2: Create New Folder Structure**
- Create `data/sources/` folder
- Create `data/activities/` folder
- Create `data/workflows/` folder
- Create `data/samples/` folder
- Keep `data/dashboard/` folder as-is
- Keep `data/navigation.json` as-is

**Step 3: Move Sample Data Files**
- Move `companies.json` from root to `data/samples/`
- Move `persons.json` from root to `data/samples/`
- Move `jobposts.json` from root to `data/samples/`
- Move `documents.json` from root to `data/samples/`
- Move `organizations.json` from root to `data/samples/`
- Move any other sample data files to `data/samples/`

**Step 4: Prepare Empty JSON Files**
- Create empty placeholder files in new folders:
  - `data/sources/sources.json` (empty, will be populated in Phase 1)
  - `data/activities/activities.json` (empty, will be populated in Phase 3)
  - `data/workflows/workflows.json` (empty, will be populated in Phase 4)
  - `data/object-types.json` (empty, will be populated in Phase 2)
  - `data/datapoints.json` (empty, will be populated in Phase 2)
  - `data/factors.json` (empty, will be populated in Phase 2)
  - `data/targets.json` (empty, will be populated in Phase 2)

**Step 5: Verify Folder Structure**
- Ensure all folders are created correctly
- Verify all files are in their proper locations
- Confirm legacy files are preserved in `1.0/` folder
- Check that sample data is consolidated in `samples/` folder

### Phase 1: Source Data Generation

**Purpose:** Generate comprehensive source detail JSONs and source list view based on existing source setups and mappings

**Compatibility Requirement:** This phase generates the foundational data. All subsequent phases must reference and learn from the generated source files to ensure compatibility.

**Target Folder Structure:**
```
data/sources/
├── sources.json                    # Overview list file for sources page
├── linkedin-people-dataset.json    # Individual source detail
├── ai-company-researcher.json      # Individual source detail
├── esg-agent.json                  # Individual source detail
├── job-market-scraper.json         # Individual source detail
├── e-commerce-scraper.json         # Individual source detail
├── event-management.json           # Individual source detail
├── automotive-database.json        # Individual source detail
└── real-estate-api.json            # Individual source detail
```

**Step 1: Create Individual source-detail.json Files (Detail View)**

**Purpose:** Populate source detail pages with full configuration

**Target Structure:**
```json
{
  "id": "uuid",
  "name": "Source Name",
  "description": "Source description",
  "source_type": "Dataset",
  "delivery_type": "Endpoint",
  "configuration": {
    "max_concurrent_runs": 50,
    "timeout": 60,
    "max_retries": 3,
    "auth_key": "optional_auth_key"
  },
  "templates": {
    "run_request_template": "{\"url\": \"https://api.example.com/run\", \"method\": \"POST\", \"headers\": {\"Content-Type\": \"application/json\"}, \"body\": {\"filter\": {\"filters\": \"<<$run_setup>>\"}}}",
    "status_request_template": "{\"url\": \"https://api.example.com/status/<<$run_external_id>>\", \"method\": \"GET\", \"headers\": {\"Authorization\": \"Bearer token\"}, \"body\": {}}",
    "delivery_request_template": "{\"url\": \"https://api.example.com/download/<<$run_external_id>>\", \"method\": \"GET\", \"headers\": {\"Authorization\": \"Bearer token\"}, \"body\": {}}"
  },
  "setup": {
    "run_request_template": {
      "request": {
        "run_setup": ["body", "filter", "filters"],
        "webhook_url": ["webhook_url"],
        "run_id": ["id"]
      },
      "response": {
        "status": ["status"],
    "status_translations": {
          "invalid": "invalid",
          "waiting": "waiting", 
          "in_progress": "in_progress",
          "ready": "ready",
          "done": "done",
          "failed": "failed",
          "timedout": "timedout"
        }
      }
    },
    "status_request_template": {
      "request": {
        "run_external_id": ["url"]
      },
      "response": {
        "status": ["status"],
        "status_translations": {
          "invalid": "invalid",
          "waiting": "waiting", 
          "in_progress": "in_progress",
          "ready": "ready",
          "done": "done",
          "failed": "failed",
          "timedout": "timedout"
        }
      }
    },
    "delivery_request_template": {
      "request": {
        "run_external_id": ["url"]
      },
      "response": {
        "status": ["status"],
    "status_translations": {
          "invalid": "invalid",
          "waiting": "waiting", 
          "in_progress": "in_progress",
          "ready": "ready",
          "done": "done",
          "failed": "failed",
          "timedout": "timedout"
        }
      }
    }
  },
  "metadata": {
    "owner_org_id": "uuid",
    "owner_org_name": "Organization Name",
    "created_at": "2025-03-31T11:41:01.452728Z",
    "updated_at": "2025-03-31T14:02:56.342106Z",
    "last_used_at": "2025-03-31T14:02:56.342106Z",
    "average_run_duration": 45,
    "total_runs": 150,
    "successful_runs": 142,
    "failed_runs": 8,
    "success_rate": 0.947,
    "activities_count": 12,
    "workflows_count": 5,
    "activities": [
      {
        "id": "uuid",
        "name": "Activity Name",
        "activity_type": "sourcing|extraction|enrichment"
      }
    ],
    "workflows": [
      {
        "id": "uuid", 
        "name": "Workflow Name"
      }
    ]
  }
}
```

**Step 2: source-setup.json Enhancement Strategy**

**Comprehensive Source Population Plan for Edge Case Testing**

Based on analysis of all 8 sources, here's the strategic plan to populate each source to cover all edge cases while maintaining some sources as "perfect" examples:

#### **Source Classification & Strategy:**

**1. PERFECT SOURCES (2 sources) - Complete, error-free examples**
- **LinkedIn People Dataset** (Dataset type)
- **Real Estate API** (Integration type)

**Characteristics:**
- All 7 status translation keys present and correct
- Complete template suite (run, status, delivery)
- Rich mapping data with complex JSON paths
- Realistic runtime statistics
- Complete metadata and relationships

**2. EDGE CASE SOURCES (6 sources) - Each covering specific edge cases**

#### **Edge Case Distribution Plan:**

**AI Company Researcher** (Agent type) - **Empty Template Edge Cases**
- **Edge Cases Covered:**
  - Empty status_request_template: `{}`
  - Empty delivery_request_template: `{}`
  - Simple body-based requests only
  - Missing status/delivery functionality
- **Data Characteristics:**
  - Only run_request_template populated
  - Simple JSON body structure
  - Agent-specific configuration

**ESG Agent** (Agent type) - **Minimal Configuration Edge Cases**
- **Edge Cases Covered:**
  - Minimal template configuration
  - Empty status/delivery templates
  - Simple request structure
  - Basic status translations only
- **Data Characteristics:**
  - Minimal setup complexity
  - Basic variable mappings
  - Agent processing patterns

**Job Market Scraper** (ML model type) - **ML Processing Edge Cases**
- **Edge Cases Covered:**
  - ML-specific processing patterns
  - Batch processing configurations
  - Empty status/delivery templates
  - ML model status codes
- **Data Characteristics:**
  - ML processing templates
  - Batch-oriented configurations
  - Model-specific status handling

**E-commerce Scraper** (Integration type) - **Complex Integration Edge Cases**
- **Edge Cases Covered:**
  - Complex pagination handling
  - Multiple request parameters
  - Integration-specific patterns
  - Complex variable mappings
- **Data Characteristics:**
  - Full template suite
  - Complex request structures
  - Integration patterns

**Event Management System** (LLM type) - **LLM Processing Edge Cases**
- **Edge Cases Covered:**
  - LLM-specific processing
  - Complex date range handling
  - LLM status patterns
  - Event-specific configurations
- **Data Characteristics:**
  - LLM processing templates
  - Event management patterns
  - Complex filtering

**Automotive Database** (LLM type) - **Database Edge Cases**
- **Edge Cases Covered:**
  - Database-specific patterns
  - Simple body requests
  - Empty status/delivery templates
  - Database query patterns
- **Data Characteristics:**
  - Database-oriented templates
  - Simple request structures
  - Database-specific configurations

#### **Specific Edge Cases to Cover Across All Sources:**

**Status Translation Edge Cases:**
1. **LinkedIn People Dataset**: All 7 keys present and correct
2. **Real Estate API**: All 7 keys present and correct
3. **AI Company Researcher**: Missing 4 keys (only 3 present)
4. **ESG Agent**: Incomplete status translations (5 out of 7)
5. **Job Market Scraper**: ML-specific status codes
6. **E-commerce Scraper**: Integration-specific status codes
7. **Event Management**: LLM-specific status codes
8. **Automotive Database**: Database-specific status codes

**Template Complexity Edge Cases:**
1. **Complex Templates**: LinkedIn, Real Estate (full HTTP structures)
2. **Simple Templates**: AI Company Researcher, ESG Agent (body-only)
3. **Empty Templates**: AI Company Researcher, ESG Agent, Job Market, Automotive
4. **Partial Templates**: E-commerce, Event Management (some empty)

**Runtime Statistics Edge Cases:**
1. **Perfect Performance**: LinkedIn (95%+ success rate)
2. **Poor Performance**: AI Company Researcher (60% success rate)
3. **New Source**: ESG Agent (0 runs, no statistics)
4. **High Volume**: Real Estate (10,000+ runs)
5. **Failed Source**: Job Market (high failure rate)
6. **Inconsistent Data**: E-commerce (math doesn't add up)
7. **Missing Data**: Event Management (null timestamps)
8. **Extreme Values**: Automotive (very high/low values)

**Configuration Edge Cases:**
1. **Standard Config**: LinkedIn, Real Estate (normal values)
2. **Minimal Config**: AI Company Researcher (low concurrency)
3. **High Performance**: E-commerce (high concurrency, long timeout)
4. **Missing Fields**: ESG Agent (missing auth_key)
5. **Invalid Values**: Job Market (negative timeout)
6. **Extreme Values**: Event Management (very high retries)
7. **Basic Config**: Automotive (minimal settings)

**Mapping Complexity Edge Cases:**
1. **Complex Mappings**: LinkedIn (deep nested paths, arrays)
2. **Simple Mappings**: AI Company Researcher (basic paths)
3. **Empty Mappings**: ESG Agent (minimal mappings)
4. **Invalid Mappings**: Job Market (broken paths)
5. **Array Mappings**: E-commerce (array index access)
6. **Wildcard Mappings**: Real Estate (wildcard patterns)
7. **Mixed Mappings**: Event Management (key/value mixed)
8. **Database Mappings**: Automotive (database-specific paths)

This plan ensures comprehensive edge case coverage while maintaining 2 perfect examples for reference, allowing the frontend to be thoroughly tested against all possible data variations and error conditions.

**Step 3: Generate sources.json (List View)**

**Purpose:** Create sources list view by extracting data from individual source-detail.json files

**Target File:** `data/sources/sources.json`

**Implementation:**
- Create simple script to extract data from all source-detail.json files
- Generate sources.json with consistent data structure
- Ensure data consistency between list and detail views
- Include all runtime statistics and metadata

**Target Structure:**
```json
{
  "sources": [
    {
      "id": "uuid",
      "name": "Source Name",
      "description": "Source description",
      "source_type": "Dataset|Agent|Integration|ML model|LLM",
      "delivery_type": "Endpoint|Webhook",
      "max_concurrent_runs": 50,
      "timeout": 60,
      "average_run_duration": 45,
      "created_at": "2025-03-31T11:41:01.452728Z",
      "updated_at": "2025-03-31T14:02:56.342106Z",
      "status": "active|inactive|deleted",
      "owner_org_id": "uuid",
      "owner_org_name": "Organization Name",
      "last_used_at": "2025-03-31T14:02:56.342106Z",
      "total_runs": 150,
      "successful_runs": 142,
      "failed_runs": 8,
      "success_rate": 0.947,
      "activities_count": 12,
      "workflows_count": 5
    }
  ]
}
```


**Step 4: Source Validation Utilities**

**Step 4.1: Create Validation Utilities**

   **Target Files:**
   - `utils/validation-utils.tsx` - Single file with all validation functions
   - `utils/types.ts` - TypeScript interfaces

   **Source Structure Validator**
   - **Function Name:** `validateSourceStructure(sourceData: SourceDetailData)`
   - **Purpose:** Validates that each source-detail.json file contains all required fields and proper data types
   - **Validation Logic:**
     - **Required Fields Check:** Must contain exactly these 9 fields: id, name, description, source_type, delivery_type, configuration, templates, setup, metadata
     - **Field Type Validation:**
       - id: string, must match UUID v4 regex pattern: /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i
       - name: string, length between 1-100 characters, no leading/trailing whitespace
       - description: string, length between 0-500 characters
       - source_type: string, must be one of: "Dataset", "Agent", "Integration", "ML model", "LLM"
       - delivery_type: string, must be one of: "Endpoint", "Webhook"
       - configuration: object, must contain exactly: max_concurrent_runs (number), timeout (number), retries (number)
       - templates: object, must contain exactly: run_request_template, status_request_template, delivery_request_template
       - setup: object, must contain exactly: run_request_template, status_request_template, delivery_request_template
       - metadata: object, must contain exactly: owner_org_id, runtime_stats, activities, workflows
     - **No Extra Fields:** Object must not contain any fields beyond the 9 required fields
     - **No Missing Fields:** All 9 required fields must be present

   **Template Structure Validator**
   - **Function Name:** `validateTemplateStructure(templates: TemplatesData)`
   - **Purpose:** Validates that all three template types have proper request and response sections
   - **Validation Logic:**
     - **Template Types Check:** Must contain exactly 3 template types: run_request_template, status_request_template, delivery_request_template
     - **Each Template Structure:**
       - Must be an object with exactly 2 fields: request, response
       - request: object, must contain variable mappings as arrays of strings
       - response: object, must contain exactly 2 fields: status (array of strings), status_translations (object)
     - **Request Section Validation:**
       - All values must be arrays of strings
       - Array elements must be non-empty strings (length > 0)
       - No null, undefined, or non-string elements in arrays
     - **Response Section Validation:**
       - status: array of strings, must contain at least 1 element
       - status_translations: object, must contain exactly 7 keys (see Status Translation Validator)
     - **No Extra Fields:** Each template must not contain fields beyond request and response
     - **No Missing Fields:** Each template must contain both request and response sections

   **Status Translation Validator**
   - **Function Name:** `validateStatusTranslations(statusTranslations: StatusTranslations)`
   - **Purpose:** Validates that all status_translations objects contain exactly 7 required keys
   - **Validation Logic:**
     - **Required Keys Check:** Must contain exactly these 7 keys in this exact order: "invalid", "waiting", "in_progress", "ready", "done", "failed", "timedout"
     - **Key Value Validation:**
       - Each value must be a string
       - String length must be between 1-50 characters
       - No leading or trailing whitespace
       - No null, undefined, or empty string values
       - Values must be valid status identifiers (alphanumeric and underscores only)
     - **No Extra Keys:** Object must not contain any keys beyond the 7 required keys
     - **No Missing Keys:** All 7 required keys must be present
     - **Consistency Check:** All status_translations objects across all 3 template types must have identical key-value pairs

   **Mapping Path Validator**
   - **Function Name:** `validateMappingPaths(setup: SetupData)`
   - **Purpose:** Validates that all mapping paths are valid JSON paths and reference actual fields
   - **Validation Logic:**
     - **JSON Path Syntax Validation:**
       - Must match regex pattern: /^[a-zA-Z_][a-zA-Z0-9_]*(\.[a-zA-Z_][a-zA-Z0-9_]*)*(\[[0-9]+\])?$/
       - Valid examples: "body", "body.filter", "response.status", "data.items[0]", "result.users[1].name"
       - Invalid examples: ".body", "body.", "body..filter", "body[", "body[]", "body[abc]"
     - **Path Structure Rules:**
       - Must start with alphanumeric character or underscore
       - Property names must be alphanumeric or underscore only
       - Array indices must be non-negative integers
       - No consecutive dots or dots at start/end
       - Maximum path depth: 10 levels
     - **Array Index Validation:**
       - Array indices must be non-negative integers (0, 1, 2, etc.)
       - No negative indices (-1, -2, etc.)
       - No non-integer indices (1.5, "0", etc.)
     - **Path Length Validation:**
       - Maximum total path length: 200 characters
       - Each property name: 1-50 characters
     - **No Circular References:** Paths must not reference themselves or create circular dependencies

   **Configuration Validator**
   - **Function Name:** `validateConfiguration(config: ConfigurationData)`
   - **Purpose:** Validates that configuration values are within acceptable ranges and consistent
   - **Validation Logic:**
     - **Required Fields Check:** Must contain exactly 3 fields: max_concurrent_runs, timeout, retries
     - **Field Type Validation:**
       - max_concurrent_runs: integer, range 1-1000
       - timeout: integer, range 1-3600 (seconds)
       - retries: integer, range 0-10
     - **Value Range Validation:**
       - max_concurrent_runs: Must be between 1 and 1000 (inclusive)
       - timeout: Must be between 1 and 3600 seconds (inclusive)
       - retries: Must be between 0 and 10 (inclusive)
     - **Source Type Consistency:**
       - Dataset sources: max_concurrent_runs >= 10, timeout >= 30
       - Agent sources: max_concurrent_runs >= 1, timeout >= 60
       - Integration sources: max_concurrent_runs >= 5, timeout >= 120
       - ML model sources: max_concurrent_runs >= 1, timeout >= 300
       - LLM sources: max_concurrent_runs >= 1, timeout >= 180
     - **No Extra Fields:** Object must not contain fields beyond the 3 required fields
     - **No Missing Fields:** All 3 required fields must be present

   **Metadata Validator**
   - **Function Name:** `validateMetadata(metadata: MetadataData)`
   - **Purpose:** Validates that runtime statistics are mathematically consistent and all referenced IDs exist
   - **Validation Logic:**
     - **Required Fields Check:** Must contain exactly 4 fields: owner_org_id, runtime_stats, activities, workflows
     - **Runtime Stats Validation:**
       - Must contain exactly 6 fields: total_runs, successful_runs, failed_runs, success_rate, activities_count, workflows_count
       - total_runs: integer >= 0
       - successful_runs: integer >= 0
       - failed_runs: integer >= 0
       - success_rate: number between 0.0 and 1.0 (inclusive)
       - activities_count: integer >= 0
       - workflows_count: integer >= 0
     - **Mathematical Consistency:**
       - successful_runs + failed_runs = total_runs (exact match)
       - success_rate = successful_runs / total_runs (when total_runs > 0)
       - success_rate = 0.0 (when total_runs = 0)
     - **Count Consistency:**
       - activities_count = activities.length (exact match)
       - workflows_count = workflows.length (exact match)
     - **ID Validation:**
       - owner_org_id: string, must match UUID v4 regex pattern
       - activities: array of objects, each must have id field matching UUID v4 pattern
       - workflows: array of objects, each must have id field matching UUID v4 pattern
     - **Timestamp Validation:**
       - All timestamp fields must be valid ISO 8601 format
       - created_at <= updated_at <= last_used_at (when all present)
     - **No Extra Fields:** Each object must not contain fields beyond required schema

**Step 4.2: Implement Validation Tests**
   - Test structure validation with malformed source-detail.json files
   - Test mapping accuracy with invalid JSON paths and missing variables
   - Test edge case handling with extreme values and boundary conditions
   - Test performance with large datasets and complex mappings
   - Test frontend display behavior with various data states and error conditions

**Step 4.3: Documentation and Examples**
   - Document template patterns and their usage
   - Provide usage examples for each source type
   - Create troubleshooting guide for common validation errors
   - Document edge case handling and error recovery strategies

### Phase 2: Object Types, Datapoints, and Factors Data Generation

**Purpose:** Generate comprehensive object types, datapoints, factors, and targets based on actual source mappings and database schema

**Compatibility Requirement:** This phase must learn from Phase 1 generated source files to ensure object types and datapoints align with actual source mappings. All generated data must be compatible with existing source structures.

**Target Folder Structure:**
```
data/
├── object-types.json          # Single list file with all object types
├── datapoints.json            # Single list file with all datapoints
├── factors.json               # Single list file with all factors
└── targets.json               # Single list file with all targets
```

**Migration Insights (2025-07-25):**
- **Legacy columns removed:** `destination`, `json_path`, `result_destination` from `run_queue`
- **Column renamed:** `key_mapping_json` → `key_mapping` in `datapoint_source`
- **Test data shows:** Company object type with industry/employees/funding datapoints, Technology/Location/Size factors

**Step 1: Create object-types.json (List View)**

**Purpose:** Generate object types data based on actual database table structure

**Learning Requirement:** Analyze Phase 1 generated source files to extract all object types referenced in source mappings and ensure complete coverage.

**Target File:** `data/object-types.json`

**Database Table:** `public.object_type`
**Fields:** id, name, description, created_at, updated_at, deleted_at

**Target Structure:**
```json
{
  "object_types": [
    {
      "id": "uuid",
      "name": "company",
      "description": "Business organizations and companies",
      "created_at": "timestamp",
      "updated_at": "timestamp",
      "deleted_at": null
    }
  ]
}
```

**Step 2: Create datapoints.json (List View)**

**Purpose:** Generate datapoints data based on actual database table structure

**Learning Requirement:** Analyze Phase 1 generated source files to extract all datapoint mappings and ensure datapoints align with actual source field names and object type relationships.

**Target File:** `data/datapoints.json`

**Database Table:** `public.datapoint`
**Fields:** id, key, object_type_id, setup, description, created_at, updated_at, deleted_at, name

**Target Structure:**
```json
{
  "datapoints": [
    {
      "id": "uuid",
      "key": "industry",
      "name": "Industry",
      "object_type_id": "uuid",
      "description": "Primary industry classification",
      "created_at": "timestamp",
      "updated_at": "timestamp",
      "deleted_at": null
    }
  ]
}
```

**Step 3: Create factors.json (List View)**

**Purpose:** Generate factors data based on actual database table structure

**Learning Requirement:** Analyze Phase 1 generated source files and Phase 2 generated object types/datapoints to ensure factors reference correct object types and datapoints, maintaining referential integrity.

**Target File:** `data/factors.json`

**Database Table:** `public.factor`
**Fields:** id, name, key, setup, created_at, updated_at, deleted_at, type, object_type_id, datapoint_id, value, factor_type, operator

**Factor Types:** volume, frequency, time, boolean, option, reason, entity, similarity, place

**Factor Type Descriptions:**
- `volume` - "How many" (quantities, counts, amounts)
- `boolean` - "Is" (true/false conditions)
- `option` - "What" (selections from choices)
- `time` - "When" (dates, durations, periods)
- `place` - "Where" (locations, positions, areas)
- `frequency` - "How often" (rates, intervals, occurrences)
- `reason` - "Why" (causes, motivations, explanations)
- `entity` - "What" (objects, things, items)
- `similarity` - "How similar" (comparisons, likeness)

**Operators:** equals, not_equals, greater_than, greater_equal, less_than, less_equal, contains, not_contains, starts_with, ends_with, in, not_in, between, is_null, is_not_null, like, ilike, regex, not_regex

**Common Factor Types Used:**
- `boolean` - Technology Usage, Location-based filters
- `volume` - Company Size, Employee Count ranges  
- `option` - Location selection, Industry categories

**Natural Language Description Generation:**
- **Guardrails:** `{object_type}'s {datapoint} {operator_phrase} {value}` (e.g., "Company's employees is more than 100")
- **Use Cases:** `{question_word} {datapoint} does the {object_type} have?` (e.g., "How many employees does the company have?")

**Target Structure:**
```json
{
  "factors": [
    {
      "id": "uuid",
      "name": "Technology Usage",
      "key": "technology",
      "description": "Company is public",
      "factor_type": "guardrail",
      "created_at": "timestamp",
      "updated_at": "timestamp",
      "deleted_at": null,
      "type": "boolean",
      "object_type_id": "uuid",
      "datapoint_id": "uuid",
      "value": "true",
      "operator": "equals"
    }
  ]
}
```

**Step 4: Create targets.json (List View)**

**Purpose:** Generate targets data based on actual database table structure

**Learning Requirement:** Analyze Phase 1 generated source files and Phase 2 generated object types/datapoints to ensure targets reference correct datapoints and maintain consistency with source mappings.

**Target File:** `data/targets.json`

**Database Table:** `public.target`
**Fields:** id, name, key, setup, created_at, updated_at, deleted_at, datapoint_id

**Target-Source Relationship Table:** `public.target_source`
**Fields:** id, target_id, source_id, activity_id, setup, created_at, updated_at, deleted_at, datapoint_id

**Target Usage Across Activity Types:**
- **Extract Activities:** Choose target objects with datapoint values (search and extract)
- **Train Activities:** Create target-source relationships for model training (e.g., forecasting models)
- **Predict Activities:** Determine exact source and datapoint for predictions

**Common Target Setup Patterns:**
- `search_config` - Search criteria for extraction activities
- `query` - Search query string for finding target objects
- `filters` - Additional filtering criteria
- `model_config` - Model training configuration for train activities
- `prediction_config` - Prediction parameters for predict activities

**Target Structure:**
```json
{
  "targets": [
    {
      "id": "uuid",
      "name": "Company Target",
      "key": "company_search",
      "setup": {},
      "created_at": "timestamp",
      "updated_at": "timestamp",
      "deleted_at": null,
      "datapoint_id": "uuid"
    }
  ]
}
```

**Target-Source Structure:**
```json
{
  "target_sources": [
    {
      "id": "uuid",
      "target_id": "uuid",
      "source_id": "uuid",
      "activity_id": "uuid",
      "setup": {},
      "created_at": "timestamp",
      "updated_at": "timestamp",
      "deleted_at": null,
      "datapoint_id": "uuid"
    }
  ]
}
```

**Comprehensive Datapoint Analysis:**

**Object Types Identified from Sources:**
- **Company** (b2c3d4e5-f6g7-8901-bcde-f23456789012) - From AI Company Researcher, LinkedIn
- **Person** (c3d4e5f6-g7h8-9012-cdef-345678901234) - From LinkedIn People Dataset
- **Product** (prod-001) - From E-commerce Scraper
- **Vehicle** (veh-001) - From Automotive Database
- **Property** (prop-001) - From Real Estate API
- **Document** (d4e5f6g7-h8i9-0123-def0-456789012345) - From ESG Agent
- **Job Post** (e5f6g7h8-i9j0-1234-ef01-567890123456) - From Job Market Scraper
- **Event** (event-001) - From Event Management
- **Agent** (agent-001) - From Real Estate API
- **Dealer** (dealer-001) - From Automotive Database
- **Category** (cat-001) - From E-commerce Scraper

**Datapoints Identified from Source Mappings:**

**Company Object Type Datapoints:**
- `name` - Company name
- `website` - Company website URL
- `industry` - Industry classification
- `employee_count` - Number of employees
- `founded_year` - Year company was founded
- `revenue` - Annual revenue
- `funding` - Total funding raised
- `currency` - Currency type
- `city` - Company city location
- `state` - Company state location
- `country` - Company country location
- `technologies` - Technology stack used
- `status` - Company status (active/inactive)

**Person Object Type Datapoints:**
- `name` - Person's full name
- `email` - Email address
- `role` - Job role/position
- `department` - Department name
- `salary` - Salary amount
- `hourly_rate` - Hourly rate for contractors
- `skills` - Array of skills
- `phone` - Phone number
- `linkedin` - LinkedIn profile URL
- `website` - Personal website
- `experience_years` - Years of experience
- `education` - Education background
- `specialization` - Professional specialization

**Product Object Type Datapoints:**
- `sku` - Product SKU
- `title` - Product title
- `description` - Product description
- `price` - Product price
- `category` - Product category
- `brand` - Product brand
- `availability` - Stock availability
- `rating` - Customer rating
- `reviews_count` - Number of reviews

**Vehicle Object Type Datapoints:**
- `vin` - Vehicle Identification Number
- `make` - Vehicle make
- `model` - Vehicle model
- `year` - Manufacturing year
- `color` - Vehicle color
- `mileage` - Vehicle mileage
- `price` - Vehicle price
- `condition` - Vehicle condition
- `fuel_type` - Fuel type
- `transmission` - Transmission type

**Property Object Type Datapoints:**
- `listing_id` - Property listing ID
- `street` - Street address
- `city` - Property city
- `state` - Property state
- `zip_code` - ZIP code
- `price` - Property price
- `bedrooms` - Number of bedrooms
- `bathrooms` - Number of bathrooms
- `square_feet` - Property size
- `property_type` - Type of property
- `listing_date` - Date listed
- `status` - Listing status

**Job Post Object Type Datapoints:**
- `title` - Job title
- `company` - Company name
- `location` - Job location
- `salary_range` - Salary range
- `employment_type` - Full-time/Part-time/Contract
- `experience_level` - Required experience level
- `skills_required` - Required skills
- `description` - Job description
- `posted_date` - Date posted
- `expiry_date` - Application deadline

**Document Object Type Datapoints:**
- `title` - Document title
- `type` - Document type
- `content` - Document content
- `author` - Document author
- `created_date` - Creation date
- `file_size` - File size
- `format` - File format
- `category` - Document category

**Event Object Type Datapoints:**
- `name` - Event name
- `date` - Event date
- `location` - Event location
- `description` - Event description
- `capacity` - Maximum attendees
- `price` - Event price
- `organizer` - Event organizer
- `status` - Event status

**Agent Object Type Datapoints:**
- `name` - Agent name
- `email` - Agent email
- `phone` - Agent phone
- `license_number` - Real estate license
- `agency` - Real estate agency
- `specialization` - Market specialization
- `years_experience` - Years of experience

**Dealer Object Type Datapoints:**
- `name` - Dealer name
- `address` - Dealer address
- `phone` - Dealer phone
- `website` - Dealer website
- `inventory_count` - Number of vehicles
- `rating` - Customer rating

**Category Object Type Datapoints:**
- `name` - Category name
- `description` - Category description
- `parent_category` - Parent category
- `product_count` - Number of products
- `is_active` - Category status

**Comprehensive Coverage Strategy:**
- **Primary Datapoints:** Essential fields for each object type (name, id, status)
- **Business Datapoints:** Fields used in business logic (revenue, employees, price)
- **Contact Datapoints:** Communication fields (email, phone, address)
- **Metadata Datapoints:** System and tracking fields (created_at, updated_at, source)
- **Relationship Datapoints:** Fields linking to other objects (company_id, category_id)
- **Industry-Specific Datapoints:** Specialized fields for each domain

### Phase 3: Activity Data Generation

**Purpose:** Generate comprehensive activity detail JSONs and activity list view based on database schema and source relationships

**Compatibility Requirement:** This phase must learn from Phase 1 generated source files and Phase 2 generated object types, datapoints, and factors to ensure activities reference correct IDs and maintain data consistency across all generated files.

**Target Folder Structure:**
```
data/activities/
├── activities.json                # Overview list file for activities page
├── company-enrichment.json        # Individual activity detail
├── lead-extraction.json           # Individual activity detail
├── prospect-sourcing.json         # Individual activity detail
├── sales-forecasting-training.json # Individual activity detail
├── revenue-prediction.json        # Individual activity detail
└── ... (other activities)
```

**Step 1: Create Individual activity-detail.json Files (Detail View)**

**Purpose:** Populate activity detail pages with full configuration and metadata

**Learning Requirement:** Analyze Phase 1 generated source files and Phase 2 generated object types/datapoints/factors/targets to ensure activities reference correct IDs and maintain data consistency.

**Activity Type Setup Rules:**
- **Extract Activities:** Must have `source` → `target` relationship (defines what to extract and where to store)
- **Train Activities:** Must have `source` → `target` relationship (creates target-source for model training)
- **Predict Activities:** Must have `source` → `target` relationship (determines source and datapoint for predictions)
- **Enrich Activities:** Must have `source` → `factor` relationship (enriches existing objects with more data and applies filtering)
- **Source Activities:** Must have `factor` → `source` relationship (finds objects based on factor criteria)

**Target Structure:**
```json
{
  "id": "uuid",
  "name": "Activity Name",
  "description": "Detailed activity description",
  "type": "enrich|extract|source|train|finetune",
    "object_type": "company|person|product|vehicle|document|event|jobpost|property|organization|contact|transaction|article|report|task|project|message|meeting|regulation|market|competitor|user|metric|review",
  "status": "active|inactive|deleted",
  "verification_status": true|false,
  "setup": {
    "data": {
      "source1": {
        "id": "source-uuid",
        "type": "source",
        "destination": {
          "datapoint": {
            "id": ["industry-dp-id", "employees-dp-id"]
          }
        }
      },
      "factor1": {
        "id": "tech-factor-id",
        "type": "factor",
        "destination": {
          "source": {
            "id": ["algops-source-id"]
          }
        }
      }
    }
  },
  "runtime_stats": {
    "total_requests": 150,
    "successful_requests": 142,
    "failed_requests": 8,
    "success_rate": 0.947,
    "average_duration": 45.2,
    "last_used_at": "timestamp"
  },
  "metadata": {
    "owner_org_id": "uuid",
    "owner_org_name": "Organization Name",
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "workflows": [...],
    "recent_requests": [...]
  }
}
```

**Step 2: Activity-Detail.json Generation Strategy**

**Learning Requirement:** Analyze Phase 1 generated source files and Phase 2 generated object types/datapoints/factors/targets to ensure activity generation strategy aligns with available data and maintains referential integrity.

**Comprehensive Activity Population Plan for Edge Case Testing**

Based on analysis of the database structure, here's the strategic plan to populate each activity to cover all edge cases while maintaining some activities as "perfect" examples:

#### **Activity Classification & Strategy:**

**1. PERFECT ACTIVITIES (5 activities) - Complete, error-free examples**
- **Company Enrichment** (enrich type) - `source` → `factor` relationship
- **Lead Extraction** (extract type) - `source` → `target` relationship
- **Prospect Sourcing** (source type) - `factor` → `source` relationship
- **Sales Forecasting Training** (train type) - `source` → `target` relationship
- **Revenue Prediction** (predict type) - `source` → `target` relationship

**Characteristics:**
- Complete setup configuration with all required fields
- Rich source mappings and destination configurations
- Realistic runtime statistics
- Complete metadata and relationships
- Proper verification status

**2. EDGE CASE ACTIVITIES (15 activities) - Each covering specific edge cases**

#### **Edge Case Distribution Plan:**

**Enrichment Activities (3 activities):**
- **Basic Company Enrichment** - Standard `source` → `factor` patterns
- **Complex Person Enrichment** - Complex nested mappings and filters
- **Failed Enrichment** - High failure rates and error conditions

**Extraction Activities (3 activities):**
- **Simple Lead Extraction** - Basic `source` → `target` with minimal setup
- **Complex Product Extraction** - Complex filters and multiple targets
- **New Extraction** - Zero requests, no runtime statistics

**Sourcing Activities (3 activities):**
- **Factor-Based Sourcing** - Multiple `factor` → `source` relationships
- **Simple Sourcing** - Minimal factor configuration
- **Inactive Sourcing** - Deleted/inactive status

**Training Activities (3 activities):**
- **Basic Model Training** - Simple `source` → `target` for forecasting
- **Complex ML Training** - Multiple targets and complex model configs
- **Failed Training** - Training errors and model failures

**Prediction Activities (3 activities):**
- **Revenue Prediction** - Standard `source` → `target` prediction
- **Demand Forecasting** - Complex prediction with multiple factors
- **New Prediction** - Recently created, no prediction history

#### **Specific Edge Cases to Cover Across All Activities:**

**Activity Type Edge Cases:**
1. **Enrich Activities**: `source` → `factor` relationships, object enrichment patterns
2. **Extract Activities**: `source` → `target` relationships, discovery patterns
3. **Source Activities**: `factor` → `source` relationships, discovery criteria
4. **Train Activities**: `source` → `target` relationships, model training patterns
5. **Predict Activities**: `source` → `target` relationships, prediction patterns

**Setup Configuration Edge Cases:**
1. **Complex Setup**: Multiple sources, nested configurations
2. **Simple Setup**: Single source, basic configuration
3. **Empty Setup**: Minimal or missing configuration
4. **Invalid Setup**: Malformed JSON, missing required fields

**Runtime Statistics Edge Cases:**
1. **Perfect Performance**: 95%+ success rate, consistent performance
2. **Poor Performance**: 60% success rate, high failure rates
3. **New Activity**: 0 requests, no runtime data
4. **High Volume**: 10,000+ requests, high throughput
5. **Inconsistent Data**: Mathematical inconsistencies
6. **Missing Data**: Null timestamps, incomplete statistics

**Status and Verification Edge Cases:**
1. **Active Verified**: Complete, verified activities
2. **Active Unverified**: Complete but unverified activities
3. **Inactive**: Soft-deleted activities
4. **Deleted**: Hard-deleted activities

**Object Type Edge Cases:**
1. **Business Activities**: Business-focused processing (companies, people, organizations)
2. **Product Activities**: Product-focused processing (products, vehicles, transactions)
3. **Content Activities**: Content-focused processing (documents, articles, reports, jobposts)
4. **Event Activities**: Event-focused processing (events, meetings, properties)
5. **Project Activities**: Project-focused processing (tasks, projects, messages)
6. **Analytics Activities**: Performance and feedback processing (metrics, reviews, users)
7. **Mixed Activities**: Multiple object types across categories

**Step 3: Generate activities.json (List View)**

**Purpose:** Create activities list view by extracting data from individual activity-detail.json files

**Target File:** `data/activities/activities.json`

**Implementation:**
- Create simple script to extract data from all activity-detail.json files
- Generate activities.json with consistent data structure
- Ensure data consistency between list and detail views
- Include all runtime statistics and metadata
- Extract related data (sources, datapoints, object_types) from setup JSON

**Target Structure:**
```json
{
  "activities": [
    {
      "id": "uuid",
      "name": "Activity Name",
      "description": "Activity description",
      "type": "enrich|extract|source|train|finetune",
      "object_type": "company|person|product|vehicle|document|event|jobpost|property|organization|transaction|article|report|task|project|message|meeting|user|metric|review",
      "status": "active|inactive|deleted",
      "verification_status": true|false,
      "owner_org_id": "uuid",
      "owner_org_name": "Organization Name",
      "created_at": "timestamp",
      "updated_at": "timestamp",
      "last_used_at": "timestamp",
      "total_requests": 150,
      "successful_requests": 142,
      "failed_requests": 8,
      "success_rate": 0.947,
      "workflows_count": 5,
      "average_duration": 45.2
    }
  ]
}
```

**Step 4: Activity Validation Utilities**

**Step 4.1: Extend Validation Utilities**

**Target Files:**
- `utils/validation-utils.tsx` - Extend existing validation functions
- `utils/types.ts` - Add activity-specific interfaces

**Activity Structure Validator**
- **Function Name:** `validateActivity(activityData: ActivityDetailData)`
- **Purpose:** Validates that each activity-detail.json file contains all required fields and proper data types
- **Validation Logic:**
  - **Required Fields Check:** Must contain exactly these 8 fields: id, name, description, type, object_type, status, verification_status, setup, runtime_stats, metadata
  - **Field Type Validation:**
    - id: string, must match UUID v4 regex pattern
    - name: string, length between 1-100 characters
    - description: string, length between 0-500 characters
    - type: string, must be one of: "enrich", "extract", "source", "train", "finetune"
    - object_type: string, must be one of: "company", "person", "product", "vehicle", "document", "event", "jobpost", "property", "organization", "transaction", "article", "report", "task", "project", "message", "meeting", "user", "metric", "review"
    - status: string, must be one of: "active", "inactive", "deleted"
    - verification_status: boolean
    - setup: object, must contain data section
    - runtime_stats: object, must contain performance metrics
    - metadata: object, must contain organizational data

**Activity Setup Validator**
- **Function Name:** `validateActivitySetup(setup: ActivitySetupData)`
- **Purpose:** Validates that activity setup configuration is properly structured
- **Validation Logic with Error Messages:**

  **1. Data Section Structure:**
  - **Missing data section**: "Activity setup must contain 'data' section"
  - **Empty data section**: "Activity setup 'data' section cannot be empty"
  - **Extra sections**: "Activity setup must only contain 'data' section, found additional sections: {section_names}"

  **2. Object Types Validation:**
  - **Missing id field**: "Object '{object_name}' is missing required 'id' field"
  - **Missing type field**: "Object '{object_name}' is missing required 'type' field"
  - **Invalid type**: "Object '{object_name}' has invalid type '{type}', must be one of: source, factor, target"
  - **Invalid UUID format**: "Object '{object_name}' has invalid UUID format for id: '{id}'"

  **3. ID Validation:**
  - **Source not found**: "Source with id '{source_id}' does not exist in database"
  - **Factor not found**: "Factor with id '{factor_id}' does not exist in database"
  - **Target not found**: "Target with id '{target_id}' does not exist in database"

  **4. Destination Validation:**
  - **Missing destination**: "Source '{source_name}' is missing required 'destination' field"
  - **Missing target in destination**: "Source '{source_name}' destination must contain 'target' field"
  - **Missing target.id**: "Source '{source_name}' destination.target must contain 'id' array"
  - **Invalid target.id format**: "Source '{source_name}' destination.target.id must be array of UUIDs"
  - **Target has destination**: "Target '{target_name}' cannot have destination field (targets are endpoints)"
  - **Missing destination**: "Factor '{factor_name}' is missing required 'destination' field"
  - **Missing target in destination**: "Factor '{factor_name}' destination must contain 'target' field"
  - **Missing target.id**: "Factor '{factor_name}' destination.target must contain 'id' array"
  - **Invalid target.id format**: "Factor '{factor_name}' destination.target.id must be array of UUIDs"

  **5. Relationship Validation:**
  - **Source-target relationship missing**: "Source '{source_id}' and target '{target_id}' relationship does not exist in source_target table"
  - **Factor-target relationship missing**: "Factor '{factor_id}' and target '{target_id}' relationship does not exist in factor_target table"

  **6. Structure Validation:**
  - **Invalid object structure**: "Object '{object_name}' must be a valid JSON object"
  - **Invalid destination structure**: "Object '{object_name}' destination must be a valid JSON object"
  - **Invalid target.id structure**: "Object '{object_name}' destination.target.id must be an array"

  **7. Business Logic Validation:**
  - **No sources present**: "Activity setup must contain at least one source"
  - **No factors present**: "Activity setup must contain at least one factor"
  - **Unreachable targets**: "Target '{target_id}' is not referenced by any source or factor"
  - **Empty target.id array**: "Source '{source_name}' destination.target.id array cannot be empty"
  - **Empty target.id array**: "Factor '{factor_name}' destination.target.id array cannot be empty"

  **8. Data Consistency Validation:**
  - **Duplicate object names**: "Object name '{object_name}' appears multiple times in setup"
  - **Duplicate IDs**: "ID '{id}' appears multiple times in setup"
  - **Circular reference**: "Circular reference detected in object '{object_name}'"

**Activity Runtime Stats Validator**
- **Function Name:** `validateActivityRuntimeStats(stats: RuntimeStatsData)`
- **Purpose:** Validates that runtime statistics are mathematically consistent
- **Validation Logic:**
  - **Mathematical Consistency:**
    - successful_requests + failed_requests = total_requests (exact match)
    - success_rate = successful_requests / total_requests (when total_requests > 0)
    - success_rate = 0.0 (when total_requests = 0)
  - **Value Range Validation:**
    - All counts must be non-negative integers
    - success_rate must be between 0.0 and 1.0
    - average_duration must be positive number
  - **Timestamp Validation:**
    - last_used_at must be valid ISO 8601 format
    - last_used_at must not be in the future

**Step 4.2: Implement Activity Validation Tests**
   - Test activity structure validation with malformed activity-detail.json files
   - Test setup configuration validation with invalid source/factor/target relationships
   - Test edge case handling with extreme values and boundary conditions
   - Test performance with large datasets and complex activity configurations
   - Test frontend display behavior with various activity data states and error conditions

**Step 4.3: Activity Documentation and Examples**
   - Document activity setup patterns and their usage
   - Provide usage examples for each activity type (enrich, extract, source, train, finetune)
   - Create troubleshooting guide for common activity validation errors
   - Document edge case handling and error recovery strategies for activities

### Phase 4: Workflow Data Generation

**Purpose:** Generate comprehensive workflow detail JSONs and workflow list view based on database schema and activity relationships

**Compatibility Requirement:** This phase must learn from all previously generated files (Phase 1 sources, Phase 2 object types/datapoints/factors/targets, Phase 3 activities) to ensure workflows reference correct activity IDs and maintain complete data consistency across the entire system.

**Target Folder Structure:**
```
data/workflows/
├── workflows.json                  # Overview list file for workflows page
├── lead-generation-pipeline.json   # Individual workflow detail
├── data-processing-pipeline.json   # Individual workflow detail
├── ml-training-pipeline.json       # Individual workflow detail
└── ... (other workflows)
```

**Step 1: Create Individual workflow-detail.json Files (Detail View)**

**Purpose:** Populate workflow detail pages with full configuration and execution metadata

**Learning Requirement:** Analyze all previously generated files (Phase 1 sources, Phase 2 object types/datapoints/factors/targets, Phase 3 activities) to ensure workflows reference correct activity IDs and maintain complete data consistency across the entire system.

**Workflow Types & Patterns:**
- **Lead Generation Workflow:** Source → Enrich → Extract → Train
- **Data Processing Workflow:** Extract → Enrich → Predict
- **ML Pipeline Workflow:** Train → Predict → Extract
- **Content Workflow:** Source → Extract → Enrich
- **Analytics Workflow:** Extract → Train → Predict

**Target Structure:**
- Workflow basic information (id, name, description, status, verification_status)
- Setup configuration with activity chain and phase progression
- Runtime statistics (total_executions, successful_executions, failed_executions, success_rate, average_duration, last_used_at)
- Metadata (owner_org_id, owner_org_name, created_at, updated_at, activities_count, phases_count, workflow_requests_count)

**Step 2: Workflow-Detail.json Generation Strategy**

**Learning Requirement:** Reference all previously generated files to ensure workflow generation strategy creates realistic workflows that properly chain activities and maintain data flow consistency.

**Comprehensive Workflow Population Plan for Edge Case Testing**

**1. PERFECT WORKFLOWS (3 workflows) - Complete, error-free examples**
- **Lead Generation Pipeline** - Source → Enrich → Extract → Train
- **Data Processing Pipeline** - Extract → Enrich → Predict  
- **ML Training Pipeline** - Train → Predict → Extract

**2. EDGE CASE WORKFLOWS (12 workflows) - Each covering specific edge cases**

**Workflow Type Edge Cases:**
- **Simple Workflows:** 2-3 activities, basic phase progression
- **Complex Workflows:** 5+ activities, multiple dependencies
- **Failed Workflows:** High failure rates, broken dependencies
- **New Workflows:** Zero executions, no runtime data
- **Inactive Workflows:** Deleted/inactive status

**Phase Progression Edge Cases:**
- **Sequential Phases:** 1 → 2 → 3 → 4 progression
- **Parallel Phases:** Multiple activities in same phase
- **Dependency Chains:** Complex activity dependencies
- **Broken Dependencies:** Missing or invalid activity references

**Setup Configuration Edge Cases:**
- **Complex Setup:** Multiple activities, nested configurations
- **Simple Setup:** 2 activities, basic configuration
- **Empty Setup:** Minimal or missing configuration
- **Invalid Setup:** Malformed JSON, missing required fields

**Runtime Statistics Edge Cases:**
- **Perfect Performance:** 95%+ success rate, consistent execution
- **Poor Performance:** 60% success rate, high failure rates
- **New Workflow:** 0 executions, no runtime data
- **High Volume:** 100+ executions, high throughput
- **Inconsistent Data:** Mathematical inconsistencies
- **Missing Data:** Null timestamps, incomplete statistics

**Step 3: Generate workflows.json (List View)**

**Purpose:** Create workflows list view by extracting data from individual workflow-detail.json files

**Target File:** `data/workflows/workflows.json`

**Implementation:**
- Create simple script to extract data from all workflow-detail.json files
- Generate workflows.json with consistent data structure
- Ensure data consistency between list and detail views
- Include all runtime statistics and metadata

**Target Structure:**
- Workflow basic information (id, name, description, status, verification_status)
- Organization data (owner_org_id, owner_org_name)
- Timestamps (created_at, updated_at, last_used_at)
- Runtime statistics (total_executions, successful_executions, failed_executions, success_rate, activities_count, phases_count, average_duration)

**Step 4: Workflow Validation Utilities**

**Purpose:** Create validation functions for workflow-specific concerns only

**Target Files:**
- `utils/validation-utils.tsx` - Extend existing validation functions
- `utils/types.ts` - Add workflow-specific interfaces

**Workflow-Specific Validation Functions:**

**1. validateWorkflowStructure(workflow)**
**Validates basic workflow structure and required fields:**
- Workflow has required fields: id, name, description, status, setup
- Status is valid: active, inactive, deleted
- Verification_status is boolean
- Setup is valid JSONB object
- Metadata contains required fields: owner_org_id, created_at, updated_at

**2. validateWorkflowSetup(workflow.setup)**
**Validates workflow setup JSON structure and logic:**

**Setup Structure Validation:**
- Setup.data exists and is an object
- Each activity in setup.data has required fields: id, type, setup, destination
- Type is always "activity"
- Setup.config contains phase (integer) and max_objects (integer)
- Destination.activity.id is an array of valid UUIDs

**Phase Progression Validation:**
- Phase numbers are sequential integers starting from 1
- No gaps in phase numbering (1, 2, 3, not 1, 3, 5)
- Phase 1 activities have no dependencies (no incoming destination.activity.id references)
- Each phase has at least one activity
- Activities in same phase can run in parallel

**Dependency Chain Validation:**
- All destination.activity.id references point to valid activities in the same workflow
- No circular dependencies (activity A → activity B → activity A)
- No self-references (activity A → activity A)
- Dependency chains are acyclic (can be represented as DAG)

**Activity Reference Validation:**
- All activity IDs in destination.activity.id arrays exist in setup.data
- Activity IDs are valid UUIDs
- No duplicate activity IDs in the workflow

**3. validateWorkflowMetadata(workflow.metadata, workflow.runtime_stats)**
**Validates metadata consistency and runtime statistics:**

**Metadata Validation:**
- Owner_org_id is valid UUID
- Activities_count matches actual count of activities in setup.data
- Phases_count matches highest phase number in setup
- Workflow_requests_count is non-negative integer
- Timestamps are valid ISO format

**Runtime Statistics Validation:**
- Total_executions = successful_executions + failed_executions
- Success_rate = successful_executions / total_executions (if total > 0)
- Success_rate is between 0 and 1
- Average_duration is non-negative number
- Last_used_at is valid timestamp (if not null)
- All numeric fields are consistent

**Mathematical Consistency:**
- If total_executions = 0, then successful_executions = 0 and failed_executions = 0
- If total_executions > 0, then success_rate calculation is accurate
- Activities_count and phases_count match actual setup structure

### Phase 5: Source API Simulation & Data Generation

**Purpose:** Create localhost API simulators that generate realistic random data matching the source schemas to test complete data pipelines

**Compatibility Requirement:** This phase must learn from all previously generated files (Phase 1 sources, Phase 2 object types/datapoints/factors/targets, Phase 3 activities, Phase 4 workflows) to ensure simulators generate data that matches the expected schemas and maintains complete data consistency across the entire system.

**Target Structure:**
```
data/simulators/
├── linkedin-people-api/           # LinkedIn People Dataset simulator
├── ai-company-researcher-api/     # AI Company Researcher simulator
├── automotive-database-api/       # Automotive Database simulator
├── e-commerce-scraper-api/        # E-commerce Scraper simulator
├── esg-agent-api/                 # ESG Agent simulator
├── event-management-api/          # Event Management simulator
├── job-market-scraper-api/        # Job Market Scraper simulator
├── real-estate-api/              # Real Estate API simulator
└── shared/                        # Common utilities
```

**Step 1: API Endpoint Simulation**

**Purpose:** Create realistic API endpoints that match the source template patterns

**API Endpoint Structure:**
Each simulator exposes three main endpoints that match the real source API patterns:

**Run Request Endpoint:** Accepts run setup parameters including filters, criteria, maximum objects to fetch, and webhook URL. Returns a unique run ID and estimated completion time. Simulates the initial request to start data processing.

**Status Check Endpoint:** Takes a run ID and returns the current execution status (waiting, in_progress, ready, failed). Includes progress percentage, creation timestamp, and completion timestamp. Simulates checking the progress of a running job.

**Delivery Endpoint:** Takes a run ID and returns the generated data when status is ready. Includes total record count, generation timestamp, and the actual data payload. Simulates downloading the final results.

**Health Check Endpoint:** Simple endpoint that returns service availability status for monitoring purposes.

**Step 2: Data Generation Patterns**

**Purpose:** Generate realistic data that matches the source schemas and object types

**LinkedIn People Dataset Simulator:**
Generates realistic person profiles with professional information including email addresses, full names, job titles, departments, salary ranges, hourly rates for contractors, skill arrays, phone numbers, LinkedIn profile URLs, years of experience, education backgrounds, and professional specializations. Uses realistic name generators and professional terminology.

**AI Company Researcher Simulator:**
Generates company data including website URLs, company names, industry classifications, employee counts, founding years, financial information (revenue, funding, currency), location data (city, state, country), technology stacks, and company status. Uses realistic company naming patterns and industry-specific data.

**Automotive Database Simulator:**
Generates vehicle records with VIN numbers, make and model combinations, manufacturing years, colors, mileage ranges, pricing, condition status, fuel types, and transmission types. Uses realistic automotive terminology and realistic value ranges.

**E-commerce Scraper Simulator:**
Generates product catalog data including SKUs, product titles, descriptions, pricing, categories, brands, availability status, customer ratings, and review counts. Uses realistic e-commerce product naming and categorization.

**Real Estate API Simulator:**
Generates property listings with listing IDs, street addresses, city and state information, ZIP codes, pricing, bedroom and bathroom counts, square footage, property types, listing dates, and status information. Uses realistic address patterns and property characteristics.

**Job Market Scraper Simulator:**
Generates job postings with titles, company names, locations, salary ranges, employment types, experience level requirements, required skills arrays, job descriptions, posting dates, and expiry dates. Uses realistic job market terminology and requirements.

**ESG Agent Simulator:**
Generates document records with titles, document types, content snippets, authors, creation dates, file sizes, formats, and categories. Uses realistic document naming and metadata patterns.

**Event Management Simulator:**
Generates event data including event names, dates, locations, descriptions, capacity limits, pricing, organizer information, and status indicators. Uses realistic event naming and scheduling patterns.

**Step 3: Value Generation System**

**Purpose:** Create field-specific generators that produce realistic data relationships

**Field-Specific Generators:**
Text fields use appropriate generators for names, emails, websites, and descriptions. Numeric fields use realistic ranges for counts, prices, and measurements. Date fields generate appropriate timestamps within reasonable ranges. Array fields create realistic collections of related items. Boolean fields generate true/false values based on realistic probabilities. Enum fields select from predefined lists of valid options. Location fields generate realistic geographic data. ID fields create unique identifiers using appropriate formats.

**Realistic Data Relationships:**
Email addresses match the person's name. Company websites use the company name in the domain. Phone numbers use appropriate area codes for the location. LinkedIn URLs use the person's name in the username. Skills arrays contain related professional skills. Technology stacks include complementary technologies. Financial data uses realistic ratios between revenue and employee count.

**Step 4: API Response Simulation**

**Purpose:** Simulate realistic API responses with proper timing and status progression

**Run Request Response:**
Creates a unique run identifier, estimates processing time based on the requested data volume, stores the run configuration in memory, and returns the run ID with status and estimated duration. Simulates the initial acknowledgment of a data processing request.

**Status Check Response:**
Retrieves the stored run information, calculates elapsed time since creation, determines current status based on processing progress, updates the run status if needed, and returns current status with progress percentage and timestamps. Simulates checking the progress of a running data processing job.

**Data Delivery Response:**
Verifies the run is in ready status, generates the requested data based on the original run setup, calculates total record count, and returns the complete dataset with metadata. Simulates the final delivery of processed data.

**Step 5: Integration with Backend**

**Purpose:** Ensure simulators work seamlessly with the existing backend infrastructure

**Edge Function Integration:**
Replace real API calls with simulator calls during development by using environment variables to switch between simulator and production endpoints. Maintain the same API interface so edge functions work identically with both simulators and real APIs.

**Database Integration:**
Simulators write generated data to local database tables using the same schema as the real system. Edge functions read from simulator-generated data maintaining referential integrity. Generated IDs are consistent across all related records.

**Configuration Management:**
Each simulator has its own configuration file specifying port numbers, base API paths, supported data types, maximum record limits, response delays, and error rates. This allows fine-tuning each simulator independently.

**Step 6: Testing Scenarios**

**Purpose:** Create comprehensive test scenarios that cover all edge cases and error conditions

**Edge Case Simulation:**
Configure simulators to occasionally return empty responses, generate API errors, introduce response delays, return partial data with missing fields, produce malformed JSON responses, and simulate rate limiting. This tests the system's error handling and resilience.

**Timeout and Error Simulation:**
Simulators randomly generate responses that take longer than the configured timeout period, causing the system to timeout and handle the error gracefully. Simulators also randomly return various error responses including network errors, authentication failures, and server errors.

**Data Volume Testing:**
Generate small datasets with 10-100 records for quick testing, medium datasets with 100-1000 records for normal operation testing, large datasets with 1000-10000 records for performance testing, and stress test datasets with 10000+ records for scalability testing.

**Realistic Data Patterns:**
Generate data that follows realistic business patterns such as companies having multiple employees, products belonging to categories, properties in the same geographic area, and jobs from the same companies. Maintain referential integrity across related records.

**Step 7: Realistic Data Relationships**

**Purpose:** Ensure generated data maintains logical consistency and business relationships

**Cross-Reference Consistency:**
Ensure that when a person works for a company, their company information matches the company data. When a product belongs to a category, the category exists in the category data. When a property is listed by an agent, the agent information is consistent.

**Temporal Consistency:**
Generate dates that make sense chronologically. Job postings should have posting dates before expiry dates. Company founding years should be before current year. Event dates should be in the future for upcoming events.

**Geographic Consistency:**
Ensure location data is realistic with cities matching states, states matching countries, and ZIP codes matching geographic areas. Use consistent location naming and formatting.

**Step 8: Random Response Variations**

**Purpose:** Simulate real-world API behavior with unpredictable responses and timing

**Response Time Variations:**
Simulators randomly vary response times from immediate responses to responses that exceed timeout periods. Some requests complete quickly while others take significantly longer than expected, testing the system's timeout handling.

**Data Quality Variations:**
Generate responses with varying data quality including complete data, partial data with missing fields, malformed data structures, and completely empty responses. This tests the system's data validation and error handling capabilities.

**Status Variations:**
Randomly return different status codes and error conditions including successful responses, temporary failures, permanent failures, and unexpected status codes. This tests the system's status handling and error recovery mechanisms.

**Volume Variations:**
Generate responses with varying data volumes from empty responses to responses with thousands of records. Some requests return no data while others return massive datasets, testing the system's data processing capabilities.

This simulation system provides realistic test data that matches actual source schemas while allowing comprehensive testing of the entire pipeline from source APIs through edge functions to the database, all running on localhost for development and testing purposes. The random variations and error conditions ensure the system is robust and handles real-world scenarios effectively.
