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

**Step 3.1: Factor Quality Refinement and Diversification**

**Purpose:** Enhance factor quality, diversity, and business relevance through creative manual editing

**Critical Requirement:** Edit factors ONE BY ONE using manual editing tools to avoid pattern creation and ensure maximum realism and diversity.

**Analysis Results from Generated Factors:**
- **Missing Factor Types:** 3 out of 9 types missing (frequency, reason, similarity)
- **Operator Over-Reliance:** 62.8% of guardrails use `is_not_null` operator (177 out of 282)
- **Generic Validations:** 17 factors lack business context with generic "is not empty" patterns
- **Repetitive Patterns:** Identical factor counts across object types (26 factors each)
- **Missing Business Context:** Generic questions and validations without industry expertise

**Holistic Factor Design Approach:**
- **Complete Datapoint Coverage:** For each object type, analyze ALL available datapoints to ensure comprehensive factor coverage
- **Interconnected Logic:** Create factors that consider relationships between different datapoints (e.g., company size vs. industry, person role vs. department)
- **Business Context Integration:** Apply industry-specific knowledge across all datapoints to create realistic validation scenarios
- **Diverse Factor Types:** Distribute factor types (volume, frequency, time, boolean, option, reason, entity, similarity, place) across all datapoints
- **Operator Variety:** Use different operators for similar datapoints to avoid repetitive patterns
- **Realistic Scenarios:** Create factors that reflect real-world business decisions and data quality requirements

**Refinement Strategy Per Object Type:**

**Company Factors:**
- Add legal entity validation, industry compliance, financial standards
- Convert some to `frequency` (update rates), `reason` (business decisions), `similarity` (competitor analysis)
- Use operators like `regex` (legal suffixes), `in` (industry lists), `between` (revenue ranges)
- Examples: "Company's name contains legal entity suffix", "How often does the company update its profile?"

**Person Factors:**
- Add professional validation, skill assessment, contact verification
- Convert to `frequency` (activity rates), `reason` (career changes), `similarity` (skill matching)
- Use operators like `regex` (email/phone), `contains` (skills), `in` (job titles)
- Examples: "Person's email follows valid format", "Why did the person change their role?"

**Product Factors:**
- Add e-commerce logic, pricing validation, inventory rules
- Convert to `frequency` (update rates), `reason` (price changes), `similarity` (competitor products)
- Use operators like `between` (price ranges), `regex` (SKU format), `greater_than` (minimum values)
- Examples: "Product's SKU follows standard format", "How similar is this product to competitors?"

**Vehicle Factors:**
- Add automotive standards, safety regulations, performance metrics
- Convert to `frequency` (maintenance), `reason` (recalls), `similarity` (model comparison)
- Use operators like `between` (year ranges), `in` (safety ratings), `regex` (VIN format)
- Examples: "Vehicle's year is within valid range", "How often does the vehicle require maintenance?"

**Property Factors:**
- Add real estate market logic, zoning regulations, valuation standards
- Convert to `frequency` (market updates), `reason` (price changes), `similarity` (comparable properties)
- Use operators like `between` (price/sqft), `in` (property types), `greater_than` (minimum values)
- Examples: "Property's price per square foot is realistic for area", "Why did the property price change?"

**Document Factors:**
- Add document management, compliance checks, version control
- Convert to `frequency` (revisions), `reason` (updates), `similarity` (document types)
- Use operators like `regex` (file formats), `between` (file sizes), `in` (document types)
- Examples: "Document's file type is supported", "How often is this document updated?"

**Jobpost Factors:**
- Add HR industry standards, salary validation, skill matching
- Convert to `frequency` (posting rates), `reason` (hiring needs), `similarity` (job matching)
- Use operators like `between` (salary ranges), `in` (job levels), `contains` (required skills)
- Examples: "Job's salary is within market range", "Why was this job posted?"

**Event Factors:**
- Add event management logic, capacity planning, scheduling validation
- Convert to `frequency` (event frequency), `reason` (cancellations), `similarity` (event types)
- Use operators like `between` (attendee counts), `in` (event categories), `greater_than` (minimum capacity)
- Examples: "Event's capacity is realistic for venue", "How often does this event occur?"

**Agent Factors:**
- Add AI/ML context, performance metrics, capability validation
- Convert to `frequency` (training frequency), `reason` (model updates), `similarity` (agent comparison)
- Use operators like `between` (accuracy ranges), `in` (capability types), `greater_than` (performance thresholds)
- Examples: "Agent's accuracy is above minimum threshold", "Why was the agent model updated?"

**Dealer Factors:**
- Add automotive dealer standards, inventory management, customer service
- Convert to `frequency` (inventory turnover), `reason` (sales changes), `similarity` (dealer comparison)
- Use operators like `between` (inventory levels), `in` (dealer types), `greater_than` (customer satisfaction)
- Examples: "Dealer's inventory turnover is within industry standards", "How similar is this dealer to competitors?"

**Organization Factors:**
- Add corporate governance, compliance standards, financial validation
- Convert to `frequency` (reporting frequency), `reason` (policy changes), `similarity` (organization comparison)
- Use operators like `between` (employee counts), `in` (organization types), `regex` (compliance codes)
- Examples: "Organization's employee count is realistic for industry", "Why did the organization change its structure?"

**Implementation Approach:**
1. **Manual Editing Only:** Use edit tools to modify factors one by one
2. **No Scripts:** Avoid automated pattern generation to prevent repetitive patterns
3. **Holistic Object Type Analysis:** Consider ALL datapoints for each object type to create diverse, realistic, and comprehensive factor coverage
4. **Creative Design:** Apply domain expertise and industry knowledge to each factor
5. **Diverse Operators:** Use all 19 operators with business-specific logic
6. **Realistic Values:** Apply industry standards, regulatory requirements, and business rules
7. **Natural Language:** Ensure factor names sound natural and business-relevant
8. **Cross-Datapoint Relationships:** Create factors that consider relationships between different datapoints within the same object type

**Quality Targets:**
- All 9 factor types represented with balanced distribution
- All 19 operators used with realistic business logic
- Zero generic "is not empty" validations
- Unique, creative factors per object type
- Business-relevant use cases and guardrails
- Industry-specific validation rules and standards

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

### Phase 3: Activity & Workflow Data Generation (Combined)

**Purpose:** Generate comprehensive activity detail JSONs, workflow JSONs, and their respective list views based on workflow requirements and real data sources

**Compatibility Requirement:** This phase must learn from Phase 1 generated source files and Phase 2 generated object types, datapoints, and factors to ensure activities and workflows reference correct IDs and maintain data consistency across all generated files.

**Target Folder Structure:**
```
data/
├── activities/                    # Activity detail files
│   ├── activities.json            # Overview list file for activities page
│   ├── company-enrichment.json    # Individual activity detail
│   ├── lead-extraction.json       # Individual activity detail
│   └── ... (other activities)
│
├── workflows/                     # Workflow detail files
│   ├── workflows.json             # Overview list file for workflows page
│   ├── lead-generation-pipeline.json # Individual workflow detail
│   ├── data-processing-pipeline.json # Individual workflow detail
│   └── ... (other workflows)
```

**Step 1: Define Workflow Patterns by Object Type Combinations**

**Purpose:** Create diverse workflow patterns that reflect different business scenarios and data processing approaches

**Learning Requirement:** Analyze Phase 1 generated source files and Phase 2 generated object types/datapoints/factors to ensure workflows reference correct IDs and maintain data consistency.

**Available Sources:**
1. **LinkedIn People Dataset** - Professional profiles (person data)
2. **Real Estate API** - Property listings (property data) 
3. **AI Company Researcher** - Company data enrichment (company data)
4. **ESG Agent** - Document analysis (document data)
5. **Job Market Scraper** - Job postings (jobpost data)
6. **E-commerce Scraper** - Product data (product data)
7. **Event Management System** - Event data (event data)
8. **Automotive Database** - Vehicle data (vehicle data)

**Workflow Patterns Based on Real Data:**

#### **Workflow 1: E-commerce Product Discovery & Recommendation**
**Pattern:** **Discovery → Enrichment → Recommendation**
**Object Types:** Product → Category → Review
**Business Logic:** Find products → Enrich with datapoints (category, rating, reviews_count) → Generate recommendations
**Activities:**
- **Source Products** (find products by category/price datapoints)
- **Enrich Products** (add category, rating, reviews_count datapoints)
- **Extract Categories** (extract category data from products)
- **Predict Recommendations** (predict product recommendations based on enriched data)

#### **Workflow 2: Real Estate Market Analysis**
**Pattern:** **Data Collection → Enrichment → Market Prediction**
**Object Types:** Property → Agent → Market Data
**Business Logic:** Collect properties → Enrich with agent data → Predict market trends
**Activities:**
- **Source Properties** (find properties by location/price datapoints)
- **Enrich Properties** (add agent, market data datapoints)
- **Extract Market Data** (extract market trends from properties)
- **Train Market Model** (train prediction model on market data)
- **Predict Market Trends** (predict future market conditions)

#### **Workflow 3: Professional Network Building**
**Pattern:** **Profile Sourcing → Enrichment → Network Analysis**
**Object Types:** Person → Company → Network
**Business Logic:** Find professionals → Enrich with company data → Analyze network connections
**Activities:**
- **Source People** (find professionals by role/skills datapoints)
- **Enrich People** (add company, experience datapoints)
- **Extract Network Data** (extract connection patterns)
- **Train Network Model** (train network analysis model)
- **Predict Connections** (predict potential connections)

#### **Workflow 4: Job Market Intelligence**
**Pattern:** **Job Collection → Skill Analysis → Demand Prediction**
**Object Types:** Jobpost → Skills → Market Trends
**Business Logic:** Collect job postings → Analyze skill requirements → Predict demand
**Activities:**
- **Source Jobs** (find jobs by location/role datapoints)
- **Enrich Jobs** (add skills, requirements datapoints)
- **Extract Skill Data** (extract skill patterns from jobs)
- **Train Demand Model** (train demand prediction model)
- **Predict Job Demand** (predict future job market trends)

#### **Workflow 5: Document Intelligence & Classification**
**Pattern:** **Document Collection → Analysis → Classification**
**Object Types:** Document → Content → Categories
**Business Logic:** Collect documents → Analyze content → Classify by type
**Activities:**
- **Source Documents** (find documents by type/date datapoints)
- **Enrich Documents** (add content, metadata datapoints)
- **Extract Content Data** (extract key content features)
- **Train Classification Model** (train document classification model)
- **Predict Document Types** (predict document categories)

#### **Workflow 6: Event Management & Analytics**
**Pattern:** **Event Collection → Enrichment → Attendance Prediction**
**Object Types:** Event → Attendee → Analytics
**Business Logic:** Collect events → Enrich with attendee data → Predict attendance
**Activities:**
- **Source Events** (find events by date/location datapoints)
- **Enrich Events** (add attendee, capacity datapoints)
- **Extract Analytics Data** (extract attendance patterns)
- **Train Attendance Model** (train attendance prediction model)
- **Predict Event Success** (predict event attendance and success)

#### **Workflow 7: Automotive Market Analysis**
**Pattern:** **Vehicle Collection → Market Analysis → Price Prediction**
**Object Types:** Vehicle → Dealer → Market Data
**Business Logic:** Collect vehicles → Analyze market data → Predict pricing
**Activities:**
- **Source Vehicles** (find vehicles by make/model datapoints)
- **Enrich Vehicles** (add dealer, market datapoints)
- **Extract Market Data** (extract pricing trends)
- **Train Price Model** (train price prediction model)
- **Predict Vehicle Prices** (predict vehicle market values)

#### **Workflow 8: Company Intelligence & Research**
**Pattern:** **Company Discovery → Enrichment → Competitive Analysis**
**Object Types:** Company → Industry → Competitors
**Business Logic:** Find companies → Enrich with industry data → Analyze competition
**Activities:**
- **Source Companies** (find companies by industry/size datapoints)
- **Enrich Companies** (add industry, financial datapoints)
- **Extract Industry Data** (extract competitive intelligence)
- **Train Analysis Model** (train competitive analysis model)
- **Predict Market Position** (predict company market position)

**Step 2: Create Individual activity-detail.json Files (Detail View)**

**Purpose:** Populate activity detail pages with full configuration and metadata

**Learning Requirement:** Analyze Phase 1 generated source files and Phase 2 generated object types/datapoints/factors/targets to ensure activities reference correct IDs and maintain data consistency.

**Activity Type Setup Rules:**
- **Extract Activities:** Must have `source` → `target` relationship (defines what to extract and where to store)
- **Train Activities:** Must have `source` → `target` relationship (creates target-source for model training)
- **Predict Activities:** Must have `source` → `target` relationship (determines source and datapoint for predictions)
- **Enrich Activities:** Must have `source` → `factor` relationship (enriches existing objects with more data and applies filtering)
- **Source Activities:** Must have `factor` → `source` relationship (finds objects based on factor criteria)
- **Finetune Activities:** Must have `source` → `factor` relationship (fine-tunes existing models with new data)

**Target Structure:**
```json
{
  "id": "uuid",
  "name": "Activity Name",
  "description": "Detailed activity description",
  "type": "enrich|extract|source|train|predict|finetune",
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
      "type": "enrich|extract|source|train|predict|finetune",
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
    - type: string, must be one of: "enrich", "extract", "source", "train", "predict", "finetune"
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
   - Provide usage examples for each activity type (enrich, extract, source, train, predict, finetune)
   - Create troubleshooting guide for common activity validation errors
   - Document edge case handling and error recovery strategies for activities

**Step 3: Create Individual workflow-detail.json Files (Detail View)**

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

**Step 4: Workflow-Detail.json Generation Strategy**

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

**Step 5: Generate workflows.json (List View)**

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

**Step 6: Workflow Validation Utilities**

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

### Phase 4: Source API Simulation & Data Generation

**Purpose:** Create localhost API simulators that generate realistic random data matching the source schemas to test complete data pipelines with comprehensive logging for cross-checking with backend ETL processing

**Compatibility Requirement:** This phase must learn from all previously generated files (Phase 1 sources, Phase 2 object types/datapoints/factors/targets, Phase 3 activities and workflows) to ensure simulators generate data that matches the expected schemas and maintains complete data consistency across the entire system.

**Logging Requirement:** Each simulator must generate comprehensive statistics and logs that can be cross-checked with backend ETL logging to verify data integrity, performance metrics, and system reliability.

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
├── ml-model-trainer-api/          # ML Model Trainer simulator
├── ml-model-inference-api/        # ML Model Inference simulator
├── fireworks-finetune-api/        # Fireworks Fine-tuning simulator
└── shared/                        # Common utilities and logging
```

## Step 1: Source Configuration Updates

**Purpose:** Update existing source JSONs and add new sources to support comprehensive testing patterns

**Source Pattern Analysis:**

**Webhook-Based Sources (No Status Check/Delivery Endpoints):**
- Real Estate API - Uses webhook delivery
- E-commerce Scraper - Uses webhook delivery  
- AI Company Researcher - Uses webhook delivery
- ML Model Inference - Uses webhook delivery

**Endpoint-Based Sources (Have Status Check/Delivery Endpoints):**
- LinkedIn People Dataset - Full endpoint support
- ESG Agent - Endpoint support
- Job Market Scraper - Endpoint support
- Event Management System - Endpoint support
- Automotive Database - Endpoint support
- ML Model Trainer - Full endpoint support
- Fireworks Fine-tuning - Full endpoint support

**Request Structure Patterns:**

**Webhook Sources (Everything in Parent Level):**
- Real Estate API: `body.filters` (parent) vs `body.request_id` (parent)
- E-commerce Scraper: `body.filters` (parent) vs `body.request_id` (parent)
- AI Company Researcher: `body.query` (parent) vs `body.run_id` (parent)
- ML Model Inference: `body.inference_data` (parent) vs `body.run_id` (parent)

**Endpoint Sources (Mixed Patterns):**
- LinkedIn People Dataset: `body.filter.filters` (nested) vs `body.run_id` (parent)
- ESG Agent: `body.document_id` (parent) vs `body.run_id` (parent)
- Job Market Scraper: `body.search_params` (parent) vs `body.job_id` (parent)
- Event Management System: `body.event_query` (parent) vs `body.run_id` (parent)
- Automotive Database: `body.sql_query` (parent) vs `body.run_id` (parent)
- ML Model Trainer: `body.tags.training_data` (nested) vs `body.run_id` (parent)
- Fireworks Fine-tuning: `body.training_data` (parent) vs `body.job_id` (parent)

**Status Translation Patterns:**

**Same Translations for Status Check and Delivery:**
- LinkedIn People Dataset, Real Estate API, E-commerce Scraper, AI Company Researcher, Event Management System, Automotive Database, ML Model Inference

**Different Translations for Status Check vs Delivery:**
- ESG Agent: `analyzing` vs `processed`
- Job Market Scraper: `scraping` vs `completed`
- ML Model Trainer: `training` vs `ready`
- Fireworks Fine-tuning: `training` vs `ready`

**Required Source Updates:**

**1. Update AI Company Researcher (Convert to Webhook):**
```json
{
  "templates": {
    "run_request_template": "{\"url\": \"https://ai-researcher.internal/api/analyze\", \"method\": \"POST\", \"headers\": {\"Content-Type\": \"application/json\"}, \"body\": {\"query\": \"<<$run_setup>>\", \"webhook_url\": \"<<$webhook_url>>\", \"run_id\": \"<<$id>>\"}}",
    "status_request_template": "{}",
    "delivery_request_template": "{}"
  },
  "setup": {
    "run_request_template": {
      "request": {
        "run_setup": ["body", "query"],
        "webhook_url": ["webhook_url"],
        "run_id": ["run_id"]
      },
      "response": {
        "status": ["status"],
        "status_translations": {
          "invalid": "invalid",
          "waiting": "waiting",
          "in_progress": "processing",
          "ready": "ready",
          "done": "completed",
          "failed": "failed",
          "timedout": "timeout"
        }
      }
    },
    "status_request_template": {},
    "delivery_request_template": {}
  }
}
```

**2. Add ML Model Trainer Source:**
```json
{
  "id": "ml-model-trainer",
  "name": "ML Model Trainer",
  "description": "MLflow-based model training service for machine learning model development",
  "source_type": "ML Platform",
  "delivery_type": "Endpoint",
  "configuration": {
    "max_concurrent_runs": 3,
    "timeout": 1800,
    "max_retries": 1,
    "auth_key": "mlflow_api_key"
  },
  "templates": {
    "run_request_template": "{\"url\": \"https://mlflow.internal/api/2.0/mlflow/runs/create\", \"method\": \"POST\", \"headers\": {\"Content-Type\": \"application/json\", \"Authorization\": \"Bearer <<$auth_key>>\"}, \"body\": {\"experiment_id\": \"<<$experiment_id>>\", \"run_name\": \"<<$run_name>>\", \"tags\": {\"training_data\": \"<<$run_setup>>\"}, \"webhook_url\": \"<<$webhook_url>>\", \"run_id\": \"<<$id>>\"}}",
    "status_request_template": "{\"url\": \"https://mlflow.internal/api/2.0/mlflow/runs/get\", \"method\": \"GET\", \"headers\": {\"Authorization\": \"Bearer <<$auth_key>>\"}, \"body\": {\"run_id\": \"<<$run_external_id>>\"}}",
    "delivery_request_template": "{\"url\": \"https://mlflow.internal/api/2.0/mlflow/artifacts/download\", \"method\": \"GET\", \"headers\": {\"Authorization\": \"Bearer <<$auth_key>>\"}, \"body\": {\"run_id\": \"<<$run_external_id>>\", \"path\": \"model\"}}"
  },
  "setup": {
    "run_request_template": {
      "request": {
        "run_setup": ["body", "tags", "training_data"],
        "webhook_url": ["webhook_url"],
        "run_id": ["run_id"]
      },
      "response": {
        "status": ["status"],
        "status_translations": {
          "invalid": "invalid",
          "waiting": "waiting",
          "in_progress": "training",
          "ready": "ready",
          "done": "completed",
          "failed": "failed",
          "timedout": "timeout"
        }
      }
    },
    "status_request_template": {
      "request": {
        "run_external_id": ["body", "run_id"]
      },
      "response": {
        "status": ["status"],
        "status_translations": {
          "invalid": "invalid",
          "waiting": "waiting",
          "in_progress": "training",
          "ready": "ready",
          "done": "completed",
          "failed": "failed",
          "timedout": "timeout"
        }
      }
    },
    "delivery_request_template": {
      "request": {
        "run_external_id": ["body", "run_id"]
      },
      "response": {
        "status": ["status"],
        "status_translations": {
          "invalid": "invalid",
          "waiting": "waiting",
          "in_progress": "training",
          "ready": "ready",
          "done": "completed",
          "failed": "failed",
          "timedout": "timeout"
        }
      }
    }
  }
}
```

**3. Add ML Model Inference Source (Webhook-based):**
```json
{
  "id": "ml-model-inference",
  "name": "ML Model Inference",
  "description": "MLflow-based model inference service for predictions",
  "source_type": "ML Platform",
  "delivery_type": "Webhook",
  "configuration": {
    "max_concurrent_runs": 10,
    "timeout": 300,
    "max_retries": 2,
    "auth_key": "mlflow_api_key"
  },
  "templates": {
    "run_request_template": "{\"url\": \"https://mlflow.internal/api/2.0/mlflow/runs/create\", \"method\": \"POST\", \"headers\": {\"Content-Type\": \"application/json\", \"Authorization\": \"Bearer <<$auth_key>>\"}, \"body\": {\"experiment_id\": \"<<$experiment_id>>\", \"run_name\": \"<<$run_name>>\", \"tags\": {\"inference_data\": \"<<$run_setup>>\"}, \"webhook_url\": \"<<$webhook_url>>\", \"run_id\": \"<<$id>>\"}}",
    "status_request_template": "{}",
    "delivery_request_template": "{}"
  },
  "setup": {
    "run_request_template": {
      "request": {
        "run_setup": ["body", "tags", "inference_data"],
        "webhook_url": ["webhook_url"],
        "run_id": ["run_id"]
      },
      "response": {
        "status": ["status"],
        "status_translations": {
          "invalid": "invalid",
          "waiting": "waiting",
          "in_progress": "inferencing",
          "ready": "ready",
          "done": "completed",
          "failed": "failed",
          "timedout": "timeout"
        }
      }
    },
    "status_request_template": {},
    "delivery_request_template": {}
  }
}
```

**4. Add Fireworks Fine-tuning Source:**
```json
{
  "id": "fireworks-finetune",
  "name": "Fireworks Fine-tuning",
  "description": "Fireworks.ai-based fine-tuning service for LLM models",
  "source_type": "LLM Platform",
  "delivery_type": "Endpoint",
  "configuration": {
    "max_concurrent_runs": 2,
    "timeout": 3600,
    "max_retries": 1,
    "auth_key": "fireworks_api_key"
  },
  "templates": {
    "run_request_template": "{\"url\": \"https://api.fireworks.ai/inference/v1/fine_tuning/jobs\", \"method\": \"POST\", \"headers\": {\"Content-Type\": \"application/json\", \"Authorization\": \"Bearer <<$auth_key>>\"}, \"body\": {\"model\": \"<<$base_model>>\", \"training_data\": \"<<$run_setup>>\", \"hyperparameters\": {\"learning_rate\": 0.001, \"batch_size\": 4}, \"webhook_url\": \"<<$webhook_url>>\", \"job_id\": \"<<$id>>\"}}",
    "status_request_template": "{\"url\": \"https://api.fireworks.ai/inference/v1/fine_tuning/jobs/<<$run_external_id>>\", \"method\": \"GET\", \"headers\": {\"Authorization\": \"Bearer <<$auth_key>>\"}, \"body\": {}}",
    "delivery_request_template": "{\"url\": \"https://api.fireworks.ai/inference/v1/fine_tuning/jobs/<<$run_external_id>>/model\", \"method\": \"GET\", \"headers\": {\"Authorization\": \"Bearer <<$auth_key>>\"}, \"body\": {}}"
  },
  "setup": {
    "run_request_template": {
      "request": {
        "run_setup": ["body", "training_data"],
        "webhook_url": ["webhook_url"],
        "run_id": ["job_id"]
      },
      "response": {
        "status": ["status"],
        "status_translations": {
          "invalid": "invalid",
          "waiting": "queued",
          "in_progress": "training",
          "ready": "ready",
          "done": "completed",
          "failed": "failed",
          "timedout": "timeout"
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
          "waiting": "queued",
          "in_progress": "training",
          "ready": "ready",
          "done": "completed",
          "failed": "failed",
          "timedout": "timeout"
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
          "waiting": "queued",
          "in_progress": "training",
          "ready": "ready",
          "done": "completed",
          "failed": "failed",
          "timedout": "timeout"
        }
      }
    }
  }
}
```

**5. Update Existing Sources with Missing Status Translations:**
- Add complete status translations for ESG Agent, Job Market Scraper, Event Management System, Automotive Database
- Ensure all sources have proper status_request_template and delivery_request_template configurations

## Step 2: Individual Simulator Specifications

### 1. LinkedIn People Dataset Simulator

**Purpose:** Generate realistic person profiles with professional information matching the LinkedIn People Dataset source schema

**Data Generation Patterns:**
- **Person Profiles:** Full names, email addresses, job titles, departments, salary ranges, hourly rates for contractors
- **Professional Data:** Skills arrays, phone numbers, LinkedIn profile URLs, years of experience, education backgrounds, professional specializations
- **Contact Information:** Email addresses that match person names, phone numbers with appropriate area codes, LinkedIn URLs using person names in usernames

**Object Type Mapping:** `person` object type with datapoints: name, email, role, department, salary, hourly_rate, skills, phone, linkedin, website, experience_years, education, specialization

**Expected Record Counts:** 50-500 people per run, varying by request parameters

### 2. AI Company Researcher Simulator

**Purpose:** Generate company data matching the AI Company Researcher source schema

**Data Generation Patterns:**
- **Company Information:** Website URLs, company names, industry classifications, employee counts, founding years
- **Financial Data:** Revenue, funding amounts, currency types with realistic ratios between revenue and employee count
- **Location Data:** City, state, country with consistent geographic relationships
- **Technical Data:** Technology stacks with complementary technologies, company status indicators

**Object Type Mapping:** `company` object type with datapoints: name, website, industry, employee_count, founded_year, revenue, funding, currency, city, state, country, technologies, status

**Expected Record Counts:** 20-200 companies per run, varying by industry filters

### 3. Automotive Database Simulator

**Purpose:** Generate vehicle records matching the Automotive Database source schema

**Data Generation Patterns:**
- **Vehicle Identification:** VIN numbers, make and model combinations, manufacturing years
- **Vehicle Specifications:** Colors, mileage ranges, condition status, fuel types, transmission types
- **Pricing Data:** Realistic vehicle pricing based on year, make, model, and condition

**Object Type Mapping:** `vehicle` object type with datapoints: vin, make, model, year, color, mileage, price, condition, fuel_type, transmission

**Expected Record Counts:** 100-1000 vehicles per run, varying by make/model filters

### 4. E-commerce Scraper Simulator

**Purpose:** Generate product catalog data matching the E-commerce Scraper source schema

**Data Generation Patterns:**
- **Product Information:** SKUs, product titles, descriptions, pricing, categories, brands
- **Inventory Data:** Availability status, customer ratings, review counts
- **Product Relationships:** Products belonging to categories with consistent categorization

**Object Type Mapping:** `product` object type with datapoints: sku, title, description, price, category, brand, availability, rating, reviews_count

**Expected Record Counts:** 200-2000 products per run, varying by category filters

### 5. Real Estate API Simulator

**Purpose:** Generate property listings matching the Real Estate API source schema

**Data Generation Patterns:**
- **Property Information:** Listing IDs, street addresses, city and state information, ZIP codes
- **Property Specifications:** Bedroom and bathroom counts, square footage, property types
- **Market Data:** Pricing, listing dates, status information with realistic address patterns

**Object Type Mapping:** `property` object type with datapoints: listing_id, street, city, state, zip_code, price, bedrooms, bathrooms, square_feet, property_type, listing_date, status

**Expected Record Counts:** 50-500 properties per run, varying by location filters

### 6. Job Market Scraper Simulator

**Purpose:** Generate job postings matching the Job Market Scraper source schema

**Data Generation Patterns:**
- **Job Information:** Titles, company names, locations, salary ranges, employment types
- **Requirements:** Experience level requirements, required skills arrays, job descriptions
- **Timing Data:** Posting dates, expiry dates with posting dates before expiry dates

**Object Type Mapping:** `jobpost` object type with datapoints: title, company, location, salary_range, employment_type, experience_level, skills_required, description, posted_date, expiry_date

**Expected Record Counts:** 100-1000 job postings per run, varying by location/industry filters

### 7. ESG Agent Simulator

**Purpose:** Generate document records matching the ESG Agent source schema

**Data Generation Patterns:**
- **Document Information:** Titles, document types, content snippets, authors, creation dates
- **File Data:** File sizes, formats, categories with realistic document naming patterns
- **Content Metadata:** Document categories, content types, author information

**Object Type Mapping:** `document` object type with datapoints: title, type, content, author, created_date, file_size, format, category

**Expected Record Counts:** 30-300 documents per run, varying by category filters

### 8. Event Management Simulator

**Purpose:** Generate event data matching the Event Management source schema

**Data Generation Patterns:**
- **Event Information:** Event names, dates, locations, descriptions, capacity limits, pricing
- **Organization Data:** Organizer information, status indicators with realistic event naming patterns
- **Scheduling:** Event dates in the future for upcoming events, realistic capacity and pricing

**Object Type Mapping:** `event` object type with datapoints: name, date, location, description, capacity, price, organizer, status

**Expected Record Counts:** 25-250 events per run, varying by date range filters

## Step 1: API Endpoint Simulation

**Purpose:** Create realistic API endpoints that exactly match the source template patterns from `@sources/` directory

**Simulator Architecture:** Each simulator must exactly replicate the API interface defined in the source JSONs from `@sources/` directory, including URL patterns, headers, authentication, request/response formats, and status translations.

**Source-Specific API Endpoints:**

**LinkedIn People Dataset Simulator:**
- **Run Request:** `POST https://api.linkedin.com/v2/people/search`
  - **Headers:** `Content-Type: application/json`, `Authorization: Bearer {auth_key}`
  - **Body:** `{"filter": {"filters": "{run_setup}"}, "max_objects": "{max_objects}", "webhook_url": "{webhook_url}", "run_id": "{id}"}`
  - **Response:** `{"status": "waiting", "run_id": "{generated_id}", "estimated_duration": 45}`
- **Status Check:** `GET https://api.linkedin.com/v2/runs/{run_external_id}/status`
  - **Headers:** `Authorization: Bearer {auth_key}`
  - **Response:** `{"status": "in_progress|ready|done|failed|timedout", "progress": 75, "timestamp": "2025-01-20T10:30:00Z"}`
- **Delivery:** `GET https://api.linkedin.com/v2/runs/{run_external_id}/download`
  - **Headers:** `Authorization: Bearer {auth_key}`
  - **Response:** `{"status": "done", "data": [...], "total_records": 150, "generated_at": "2025-01-20T10:35:00Z"}`

**Real Estate API Simulator:**
- **Run Request:** `POST https://api.realestate.com/v1/properties/search`
  - **Headers:** `Content-Type: application/json`, `X-API-Key: {auth_key}`
  - **Body:** `{"filters": "{run_setup}", "max_results": "{max_objects}", "callback_url": "{webhook_url}", "request_id": "{id}"}`
  - **Response:** `{"status": "waiting", "request_id": "{generated_id}", "estimated_duration": 78}`
- **Status Check:** `GET https://api.realestate.com/v1/requests/{run_external_id}/status`
  - **Headers:** `X-API-Key: {auth_key}`
  - **Response:** `{"status": "in_progress|ready|done|failed|timedout", "progress": 60, "timestamp": "2025-01-20T10:30:00Z"}`
- **Delivery:** `GET https://api.realestate.com/v1/requests/{run_external_id}/results`
  - **Headers:** `X-API-Key: {auth_key}`
  - **Response:** `{"status": "done", "data": [...], "total_records": 89, "generated_at": "2025-01-20T10:32:00Z"}`

**AI Company Researcher Simulator:**
- **Run Request:** `POST https://ai-researcher.internal/api/analyze`
  - **Headers:** `Content-Type: application/json`
  - **Body:** `{"query": "{run_setup}", "webhook_url": "{webhook_url}", "run_id": "{id}"}`
  - **Response:** `{"status": "waiting", "run_id": "{generated_id}", "estimated_duration": 95}`
- **Status Check:** `GET /api/status/{run_id}` (internal endpoint)
  - **Response:** `{"status": "processing|ready|failed", "progress": 85, "timestamp": "2025-01-20T10:30:00Z"}`
- **Delivery:** `GET /api/delivery/{run_id}` (internal endpoint)
  - **Response:** `{"status": "ready", "data": [...], "total_records": 25, "generated_at": "2025-01-20T10:32:00Z"}`

**ESG Agent Simulator:**
- **Run Request:** `POST https://esg-agent.internal/api/process`
  - **Headers:** `Content-Type: application/json`
  - **Body:** `{"document_id": "{run_setup}"}`
  - **Response:** `{"status": "queued", "run_id": "{generated_id}", "estimated_duration": 300}`
- **Status Check:** `GET /api/status/{run_id}` (internal endpoint)
  - **Response:** `{"status": "analyzing|completed|processed|error", "progress": 40, "timestamp": "2025-01-20T10:30:00Z"}`
- **Delivery:** `GET /api/delivery/{run_id}` (internal endpoint)
  - **Response:** `{"status": "processed", "data": [...], "total_records": 12, "generated_at": "2025-01-20T10:35:00Z"}`

**Job Market Scraper Simulator:**
- **Run Request:** `POST https://ml-scraper.internal/api/scrape`
  - **Headers:** `Content-Type: application/json`, `Authorization: Bearer {auth_key}`
  - **Body:** `{"search_params": "{run_setup}", "batch_size": "{max_objects}", "callback": "{webhook_url}", "job_id": "{id}"}`
  - **Response:** `{"status": "queued", "job_id": "{generated_id}", "estimated_duration": 420}`
- **Status Check:** `GET /api/status/{job_id}` (internal endpoint)
  - **Response:** `{"status": "scraping|ready|completed|failed|timeout", "progress": 30, "timestamp": "2025-01-20T10:30:00Z"}`
- **Delivery:** `GET /api/delivery/{job_id}` (internal endpoint)
  - **Response:** `{"status": "completed", "data": [...], "total_records": 200, "generated_at": "2025-01-20T10:37:00Z"}`

**E-commerce Scraper Simulator:**
- **Run Request:** `POST https://api.ecommerce-scraper.com/v2/scrape`
  - **Headers:** `Content-Type: application/json`, `X-API-Token: {auth_key}`, `X-Request-ID: {id}`
  - **Body:** `{"platforms": ["amazon", "ebay", "shopify"], "filters": "{run_setup}", "pagination": {"max_pages": 50, "per_page": "{max_objects}"}, "webhook_url": "{webhook_url}", "request_id": "{id}"}`
  - **Response:** `{"status": "waiting", "request_id": "{generated_id}", "estimated_duration": 156}`
- **Status Check:** `GET https://api.ecommerce-scraper.com/v2/requests/{run_external_id}/status`
  - **Headers:** `X-API-Token: {auth_key}`
  - **Response:** `{"status": "in_progress|ready|done|failed|timedout", "progress": 65, "timestamp": "2025-01-20T10:30:00Z"}`
- **Delivery:** `GET https://api.ecommerce-scraper.com/v2/requests/{run_external_id}/results`
  - **Headers:** `X-API-Token: {auth_key}`
  - **Response:** `{"status": "done", "data": [...], "total_records": 500, "generated_at": "2025-01-20T10:33:00Z"}`

**Event Management System Simulator:**
- **Run Request:** `POST https://event-llm.internal/api/process`
  - **Headers:** `Content-Type: application/json`, `Authorization: Bearer {auth_key}`
  - **Body:** `{"event_query": "{run_setup}", "date_range": {"start": "{start_date}", "end": "{end_date}"}, "webhook_url": "{webhook_url}", "run_id": "{id}"}`
  - **Response:** `{"status": "waiting", "run_id": "{generated_id}", "estimated_duration": 210}`
- **Status Check:** `GET https://event-llm.internal/api/status/{run_external_id}`
  - **Headers:** `Authorization: Bearer {auth_key}`
  - **Response:** `{"status": "in_progress|ready|done|failed|timedout", "progress": 80, "timestamp": "2025-01-20T10:30:00Z"}`
- **Delivery:** `GET /api/delivery/{run_id}` (internal endpoint)
  - **Response:** `{"status": "done", "data": [...], "total_records": 75, "generated_at": "2025-01-20T10:33:30Z"}`

**Automotive Database Simulator:**
- **Run Request:** `POST https://auto-db.internal/api/query`
  - **Headers:** `Content-Type: application/json`
  - **Body:** `{"sql_query": "{run_setup}", "webhook_url": "{webhook_url}", "run_id": "{id}"}`
  - **Response:** `{"status": "waiting", "run_id": "{generated_id}", "estimated_duration": 35}`
- **Status Check:** `GET /api/status/{run_id}` (internal endpoint)
  - **Response:** `{"status": "in_progress|ready|done|failed|timedout", "progress": 90, "timestamp": "2025-01-20T10:30:00Z"}`
- **Delivery:** `GET /api/delivery/{run_id}` (internal endpoint)
  - **Response:** `{"status": "done", "data": [...], "total_records": 120, "generated_at": "2025-01-20T10:30:35Z"}`

**Health Check Endpoint (All Simulators):**
- **URL Pattern:** `GET /api/health`
- **Response:** `{"status": "healthy", "uptime": "2h 15m", "source_type": "Dataset|Integration|Agent|ML model|LLM", "max_concurrent_runs": 50, "timeout": 60}`

## Step 2: Comprehensive Statistics Logging

**Purpose:** Generate detailed statistics that can be cross-checked with backend ETL processing

**Data Generation Statistics (Per Object Type):**

**Person Object Type (LinkedIn People Dataset):**
- **Record Counts:** 50-500 people per run, varying by filter criteria
- **Field Completeness:** name (100%), email (95%), phone (80%), linkedin_url (90%), skills (100%), experience_years (100%), current_company (85%), location (100%)
- **Data Quality Metrics:** Valid email format (95%), realistic experience years (100%), consistent location data (100%)
- **Schema Compliance:** All records match person datapoint schema from `@datapoints.json`

**Company Object Type (AI Company Researcher):**
- **Record Counts:** 10-100 companies per run, varying by query complexity
- **Field Completeness:** name (100%), industry (100%), size (100%), revenue (90%), website (95%), description (100%), founded_year (100%), location (100%)
- **Data Quality Metrics:** Valid website URLs (95%), realistic revenue-to-size ratios (100%), consistent industry classifications (100%)
- **Schema Compliance:** All records match company datapoint schema from `@datapoints.json`

**Property Object Type (Real Estate API):**
- **Record Counts:** 25-300 properties per run, varying by location and price filters
- **Field Completeness:** address (100%), price (100%), bedrooms (100%), bathrooms (100%), square_feet (95%), property_type (100%), listing_date (100%), agent_name (90%)
- **Data Quality Metrics:** Realistic price-to-size ratios (100%), valid property types (100%), consistent location data (100%)
- **Schema Compliance:** All records match property datapoint schema from `@datapoints.json`

**Product Object Type (E-commerce Scraper):**
- **Record Counts:** 100-1000 products per run, varying by platform and category filters
- **Field Completeness:** name (100%), price (100%), category (100%), brand (95%), description (100%), availability (100%), rating (90%), review_count (85%)
- **Data Quality Metrics:** Realistic price ranges per category (100%), valid product categories (100%), consistent brand information (100%)
- **Schema Compliance:** All records match product datapoint schema from `@datapoints.json`

**Vehicle Object Type (Automotive Database):**
- **Record Counts:** 50-200 vehicles per run, varying by make/model filters
- **Field Completeness:** make (100%), model (100%), year (100%), price (100%), mileage (100%), condition (100%), fuel_type (100%), transmission (100%)
- **Data Quality Metrics:** Realistic year-to-price relationships (100%), valid fuel types (100%), consistent make-model combinations (100%)
- **Schema Compliance:** All records match vehicle datapoint schema from `@datapoints.json`

**Document Object Type (ESG Agent):**
- **Record Counts:** 10-50 documents per run, varying by document type filters
- **Field Completeness:** title (100%), content (100%), document_type (100%), esg_category (100%), confidence_score (100%), processing_date (100%), source (100%)
- **Data Quality Metrics:** Valid ESG categories (100%), realistic confidence scores (100%), consistent document types (100%)
- **Schema Compliance:** All records match document datapoint schema from `@datapoints.json`

**Jobpost Object Type (Job Market Scraper):**
- **Record Counts:** 100-500 job postings per run, varying by search parameters
- **Field Completeness:** title (100%), company (100%), location (100%), salary_range (80%), job_type (100%), description (100%), requirements (100%), posted_date (100%)
- **Data Quality Metrics:** Realistic salary ranges per job type (100%), valid job types (100%), consistent company information (100%)
- **Schema Compliance:** All records match jobpost datapoint schema from `@datapoints.json`

**Event Object Type (Event Management System):**
- **Record Counts:** 25-250 events per run, varying by date range filters
- **Field Completeness:** name (100%), date (100%), location (100%), description (100%), capacity (100%), price (100%), organizer (100%), status (100%)
- **Data Quality Metrics:** Future event dates (100%), realistic capacity numbers (100%), consistent pricing (100%)
- **Schema Compliance:** All records match event datapoint schema from `@datapoints.json`

**Performance Statistics:**
- **Response Times:** Run request (50-200ms), Status check (10-50ms), Delivery (100-500ms)
- **Generation Duration:** Based on source configuration: LinkedIn (45s), Real Estate (78s), AI Researcher (95s), ESG Agent (300s), Job Scraper (420s), E-commerce (156s), Event Management (210s), Automotive (35s)
- **Status Transition Timing:** waiting → in_progress (1-5s), in_progress → ready (varies by source), ready → done (immediate)

**Data Content Analysis:**
- **Value Distributions:** Realistic ranges based on object type (e.g., person ages 22-65, company revenues $10K-$10B, property prices $50K-$5M)
- **Relationship Integrity:** Consistent cross-references (person → company, product → category, event → organizer)
- **Data Variety:** 80% unique values, 20% realistic duplicates per field type
- **Edge Case Coverage:** Empty fields (5%), malformed data (2%), extreme values (3%), missing relationships (1%)

**Error Simulation Tracking:**
- **Intentional Errors:** 5% timeout rate, 3% malformed JSON, 2% authentication failures
- **Error Recovery:** Automatic retry with exponential backoff, graceful degradation
- **Timeout Scenarios:** 10% of requests exceed estimated duration by 50%
- **Data Corruption:** 1% of records have intentionally malformed fields for testing

## Step 3: Value Generation System

**Purpose:** Create field-specific generators that produce realistic data relationships based on actual datapoints from `@datapoints.json`

**Field-Specific Generators (Per Object Type):**

**Person Object Type Generators:**
- **name:** Realistic first and last names from professional databases
- **email:** Generated from name using patterns: firstname.lastname@company.com, f.lastname@company.com
- **phone:** US phone numbers with area codes matching location data
- **linkedin_url:** https://linkedin.com/in/firstname-lastname-{random_suffix}
- **skills:** Professional skills arrays (5-15 skills) with realistic combinations
- **experience_years:** 0-40 years based on realistic career progression
- **current_company:** Company names from realistic business database
- **location:** US cities with consistent state/zip code combinations

**Company Object Type Generators:**
- **name:** Realistic company names with proper suffixes (Inc, LLC, Corp, Ltd)
- **industry:** Valid industries from predefined list matching company type
- **size:** Employee count (1-100,000) with realistic distribution
- **revenue:** Annual revenue matching company size and industry
- **website:** https://{company_name}.com with realistic domain patterns
- **description:** Company descriptions with industry-specific terminology
- **founded_year:** 1800-2024 with realistic founding patterns
- **location:** Headquarters cities with consistent geographic data

**Property Object Type Generators:**
- **address:** Realistic street addresses in specified cities
- **price:** Property values based on location, size, and type
- **bedrooms:** 1-6 bedrooms with realistic distribution
- **bathrooms:** 1-4 bathrooms matching bedroom count
- **square_feet:** Square footage matching bedroom/bathroom count
- **property_type:** Single Family, Condo, Townhouse, Multi-family
- **listing_date:** Recent dates within last 6 months
- **agent_name:** Realistic real estate agent names

**Product Object Type Generators:**
- **name:** Product names with brand and model information
- **price:** Realistic price ranges based on category and brand
- **category:** Electronics, Clothing, Home & Garden, Sports, Books
- **brand:** Realistic brand names matching category
- **description:** Product descriptions with technical specifications
- **availability:** In Stock, Limited Stock, Out of Stock
- **rating:** 1.0-5.0 stars with realistic distribution
- **review_count:** 0-10,000 reviews matching rating patterns

**Vehicle Object Type Generators:**
- **make:** Toyota, Honda, Ford, BMW, Mercedes, Tesla, etc.
- **model:** Realistic model names matching make
- **year:** 2010-2024 with realistic age distribution
- **price:** Vehicle values based on make, model, year, condition
- **mileage:** 0-200,000 miles with realistic patterns
- **condition:** Excellent, Good, Fair, Poor
- **fuel_type:** Gasoline, Electric, Hybrid, Diesel
- **transmission:** Automatic, Manual, CVT

**Document Object Type Generators:**
- **title:** ESG document titles with professional terminology
- **content:** Realistic document content with ESG keywords
- **document_type:** Report, Policy, Assessment, Statement
- **esg_category:** Environmental, Social, Governance
- **confidence_score:** 0.0-1.0 with realistic distribution
- **processing_date:** Recent dates within last 30 days
- **source:** Document source organizations

**Jobpost Object Type Generators:**
- **title:** Professional job titles with realistic seniority levels
- **company:** Company names matching job market data
- **location:** US cities with job market presence
- **salary_range:** Realistic salary ranges based on title and location
- **job_type:** Full-time, Part-time, Contract, Internship
- **description:** Job descriptions with required qualifications
- **requirements:** Skills and experience requirements
- **posted_date:** Recent dates within last 30 days

**Event Object Type Generators:**
- **name:** Event names with professional conference terminology
- **date:** Future dates within next 12 months
- **location:** Conference centers and venues in major cities
- **description:** Event descriptions with agenda information
- **capacity:** 50-10,000 attendees with realistic distribution
- **price:** Ticket prices based on event type and capacity
- **organizer:** Event management companies and organizations
- **status:** Active, Sold Out, Cancelled, Postponed

**Realistic Data Relationships:**
- **Email Consistency:** Email addresses match person names and company domains
- **Website Consistency:** Company websites use company names in domains
- **Geographic Consistency:** Phone numbers use appropriate area codes for locations
- **Professional Consistency:** LinkedIn URLs use person names in usernames
- **Skill Relationships:** Skills arrays contain related professional skills for job titles
- **Technology Relationships:** Technology stacks include complementary technologies
- **Financial Relationships:** Realistic ratios between revenue and employee count
- **Location Relationships:** Addresses, phone numbers, and companies in same geographic area
- **Industry Relationships:** Company industries match job postings and skill requirements

## Step 4: API Response Simulation

**Purpose:** Simulate realistic API responses with proper timing and status progression matching source configurations

**Run Request Response (Per Source):**
- **LinkedIn People Dataset:** Creates run_id, estimates 45s duration, stores filter criteria and max_objects
- **Real Estate API:** Creates request_id, estimates 78s duration, stores filters and max_results
- **AI Company Researcher:** Creates run_id, estimates 95s duration, stores query parameters
- **ESG Agent:** Creates run_id, estimates 300s duration, stores document_id
- **Job Market Scraper:** Creates job_id, estimates 420s duration, stores search_params and batch_size
- **E-commerce Scraper:** Creates request_id, estimates 156s duration, stores platforms, filters, and pagination
- **Event Management:** Creates run_id, estimates 210s duration, stores event_query and date_range
- **Automotive Database:** Creates run_id, estimates 35s duration, stores sql_query

**Status Check Response (Per Source):**
- **LinkedIn:** Returns status from waiting → in_progress → ready → done with progress 0-100%
- **Real Estate:** Returns status from waiting → in_progress → ready → done with progress 0-100%
- **AI Researcher:** Returns status from waiting → processing → ready with progress 0-100%
- **ESG Agent:** Returns status from queued → analyzing → completed → processed with progress 0-100%
- **Job Scraper:** Returns status from queued → scraping → ready → completed with progress 0-100%
- **E-commerce:** Returns status from waiting → in_progress → ready → done with progress 0-100%
- **Event Management:** Returns status from waiting → in_progress → ready → done with progress 0-100%
- **Automotive:** Returns status from waiting → in_progress → ready → done with progress 0-100%

**Data Delivery Response (Per Object Type):**
- **Person Data:** Returns array of person objects with all required fields populated
- **Company Data:** Returns array of company objects with industry-specific information
- **Property Data:** Returns array of property objects with location and pricing data
- **Product Data:** Returns array of product objects with category and platform information
- **Vehicle Data:** Returns array of vehicle objects with make/model specifications
- **Document Data:** Returns array of document objects with ESG classification
- **Jobpost Data:** Returns array of job posting objects with salary and requirements
- **Event Data:** Returns array of event objects with scheduling and capacity information

**Response Timing Simulation:**
- **Immediate Response:** Run request returns within 50-200ms
- **Status Progression:** waiting (1-5s) → in_progress (varies by source) → ready (immediate)
- **Data Generation:** Background process generates data during in_progress status
- **Delivery Response:** Returns complete dataset within 100-500ms when ready

## Step 5: Integration with Backend

**Purpose:** Ensure simulators work seamlessly with existing backend infrastructure

**Edge Function Integration:**
- **Environment Variable Switching:** Use `SIMULATOR_MODE=true` to switch from real APIs to simulators
- **URL Mapping:** Map real API URLs to localhost simulator endpoints (e.g., `api.linkedin.com` → `localhost:3001`)
- **Header Preservation:** Maintain exact header formats and authentication methods from source templates
- **Request/Response Compatibility:** Ensure edge functions work identically with both simulators and real APIs
- **Logging Integration:** Log all edge function interactions with run_id correlation for cross-checking

**Database Integration:**
- **Schema Compatibility:** Write generated data to local database tables using exact schema from `@database/` migrations
- **ID Consistency:** Generate UUIDs that are consistent across all related records (person → company, product → category)
- **Referential Integrity:** Maintain foreign key relationships between generated records
- **Value Table Population:** Populate `value` table with generated data matching datapoint schemas
- **Activity Integration:** Link generated data to activity runs and workflow executions

**Configuration Management (Per Simulator):**
- **Port Configuration:** Each simulator runs on unique port (3001-3008)
- **Base API Paths:** Match exact URL patterns from source templates
- **Data Type Support:** Configure supported object types and datapoints per simulator
- **Record Limits:** Set maximum records per run based on source configuration
- **Response Delays:** Configure realistic response times matching source timeouts
- **Error Rates:** Set configurable error injection rates for testing
- **Authentication:** Configure auth keys matching source configurations

**Simulator-Specific Configurations:**
- **LinkedIn Simulator:** Port 3001, supports person object type, max 500 records, 45s timeout
- **Real Estate Simulator:** Port 3002, supports property object type, max 300 records, 78s timeout
- **AI Researcher Simulator:** Port 3003, supports company object type, max 100 records, 95s timeout
- **ESG Agent Simulator:** Port 3004, supports document object type, max 50 records, 300s timeout
- **Job Scraper Simulator:** Port 3005, supports jobpost object type, max 500 records, 420s timeout
- **E-commerce Simulator:** Port 3006, supports product object type, max 1000 records, 156s timeout
- **Event Management Simulator:** Port 3007, supports event object type, max 250 records, 210s timeout
- **Automotive Simulator:** Port 3008, supports vehicle object type, max 200 records, 35s timeout

## Step 6: Testing Scenarios

**Purpose:** Create comprehensive test scenarios that cover all edge cases and error conditions

**Edge Case Simulation (Per Source):**
- **Empty Responses:** 5% of runs return empty data arrays
- **API Errors:** 3% authentication failures, 2% malformed JSON responses
- **Timeout Scenarios:** 10% of runs exceed estimated duration by 50%
- **Partial Data:** 2% of runs return incomplete records with missing fields
- **Rate Limiting:** Simulate 429 responses with retry-after headers
- **Network Errors:** 1% connection timeouts and network failures
- **Data Corruption:** 1% of records have intentionally malformed fields

**Data Volume Testing (Per Object Type):**
- **Small Datasets:** 10-50 records for quick testing and development
- **Medium Datasets:** 50-500 records for normal operation testing
- **Large Datasets:** 500-2000 records for performance testing
- **Stress Test Datasets:** 2000+ records for scalability testing
- **Volume Metrics:** Log processing times, memory usage, and throughput rates

**Realistic Data Patterns (Per Object Type):**
- **Person-Company Relationships:** Generate people working at realistic companies
- **Product-Category Relationships:** Generate products with proper category classifications
- **Property-Location Relationships:** Generate properties in consistent geographic areas
- **Job-Company Relationships:** Generate job postings from realistic companies
- **Event-Organizer Relationships:** Generate events with consistent organizer information
- **Vehicle-Make-Model Relationships:** Generate vehicles with proper make/model combinations
- **Document-ESG Relationships:** Generate documents with appropriate ESG classifications

**Error Recovery Testing:**
- **Retry Logic:** Test exponential backoff for failed requests
- **Graceful Degradation:** Test system behavior with partial data
- **Fallback Mechanisms:** Test alternative data sources when primary fails
- **Data Validation:** Test validation of corrupted or incomplete data

## Step 7: Cross-Checking Integration

**Purpose:** Enable comprehensive comparison between simulator output and backend ETL processing

**Logging Format Standardization:**
- **Log Format:** JSON format with consistent structure across all simulators
- **Required Fields:** run_id, timestamp, simulator_name, endpoint, status, duration, record_count
- **Correlation IDs:** Include run_id in all log entries for backend correlation
- **Timestamp Format:** ISO 8601 format with timezone information
- **Field Naming:** Use consistent snake_case naming convention

**Data Integrity Verification:**
- **Record Counts:** Log exact number of records generated per object type
- **Field Completeness:** Log percentage of populated vs empty fields per datapoint
- **Data Quality Metrics:** Log count of malformed records and validation failures
- **Relationship Integrity:** Log count of valid cross-references between related records
- **Schema Compliance:** Log percentage of records matching expected datapoint schemas

**Performance Comparison:**
- **Response Times:** Log API endpoint response times (run, status, delivery)
- **Generation Duration:** Log data generation time per object type
- **Throughput Rates:** Log records generated per second
- **Status Transitions:** Log timing of status changes (waiting → in_progress → ready)
- **Memory Usage:** Log memory consumption during data generation

**Error Handling Verification:**
- **Intentional Errors:** Log all injected errors with expected vs actual behavior
- **Error Recovery:** Log success/failure rates of retry mechanisms
- **Timeout Scenarios:** Log which requests exceeded timeout thresholds
- **Data Corruption:** Log validation results for intentionally malformed data
- **System Resilience:** Log system behavior under various error conditions

**Cross-Checking Reports:**
- **Daily Summary:** Generate daily reports comparing simulator vs backend metrics
- **Performance Analysis:** Compare response times and throughput rates
- **Data Quality Analysis:** Compare data completeness and validation results
- **Error Analysis:** Compare error rates and recovery success rates
- **Relationship Analysis:** Verify cross-reference integrity between systems

This comprehensive simulation system provides realistic test data that matches actual source schemas while generating detailed logs that can be cross-checked with backend ETL processing to ensure data integrity, verify performance metrics, and validate system reliability across the entire pipeline.

### Phase 5: Factor-Source Request Template Generation

**Purpose:** Generate `factor_source` records that contain request body templates for `run_setup` to enable proper API requests to source simulators

**Compatibility Requirement:** This phase must learn from all previously generated files (Phase 1 sources, Phase 2 factors, Phase 3 activities) to ensure `factor_source` records reference correct IDs and create realistic request templates that work with the simulators from Phase 4.

**Target File:** `data/factor-sources.json`

**Database Table:** `public.factor_source`
**Fields:** id, factor_id, source_id, setup, key_mapping, created_at, updated_at, deleted_at

**Step 1: Factor-Source Mapping Analysis**

**Purpose:** Determine which factors should be associated with which sources based on activities and create realistic request templates

**Source-Factor Analysis by Activity:**

**Stackable Sources (Multiple factors per request):**
- **LinkedIn People Dataset** (person object type) - 39 factors
- **Real Estate API** (property object type) - 36 factors  
- **E-commerce Scraper** (product object type) - 39 factors
- **Job Market Scraper** (jobpost object type) - 39 factors
- **Event Management System** (event object type) - 39 factors
- **Automotive Database** (vehicle object type) - 39 factors
- **ML Model Trainer** (company object type) - 39 factors
- **Fireworks Fine-tuning** (company object type) - 39 factors

**Standalone Sources (Individual factors per request):**
- **AI Company Researcher** (company object type) - 39 factors
- **ESG Agent** (document object type) - 36 factors
- **ML Model Inference** (company object type) - 39 factors

**Step 2: Request Template Generation Strategy**

**Purpose:** Create realistic `factor_source.setup` JSON that contains parts of the request body for `run_setup` to be sent to source simulators

**Template Generation Rules:**
- **Guardrail Factors:** Map to concrete request parameters (filters, criteria, validation rules)
- **Use Case Factors:** Map to data inclusion flags and open-ended query parameters
- **Stackable Requests:** Combine multiple guardrail factors into single request with combined filters
- **Standalone Requests:** Create individual requests for use case factors or complex business logic factors
- **No max_objects/max_results:** These are separate variables, not part of `factor_source.setup`
- **Empty key_mapping:** Keep `key_mapping` field empty as specified

**Step 3: Stackable Factor-Source Records (104 records)**

**LinkedIn People Dataset (13 stackable groups):**
- **Group 1:** Use case factor + 2 guardrail factors for `name` datapoint
- **Group 2:** Use case factor + 2 guardrail factors for `email` datapoint
- **Group 3:** Use case factor + 2 guardrail factors for `role` datapoint
- **Group 4:** Use case factor + 2 guardrail factors for `department` datapoint
- **Group 5:** Use case factor + 2 guardrail factors for `salary` datapoint
- **Group 6:** Use case factor + 2 guardrail factors for `hourly_rate` datapoint
- **Group 7:** Use case factor + 2 guardrail factors for `skills` datapoint
- **Group 8:** Use case factor + 2 guardrail factors for `phone` datapoint
- **Group 9:** Use case factor + 2 guardrail factors for `linkedin` datapoint
- **Group 10:** Use case factor + 2 guardrail factors for `website` datapoint
- **Group 11:** Use case factor + 2 guardrail factors for `experience_years` datapoint
- **Group 12:** Use case factor + 2 guardrail factors for `education` datapoint
- **Group 13:** Use case factor + 2 guardrail factors for `specialization` datapoint

**Real Estate API (12 stackable groups):**
- **Group 1:** Use case factor + 2 guardrail factors for `listing_id` datapoint
- **Group 2:** Use case factor + 2 guardrail factors for `street` datapoint
- **Group 3:** Use case factor + 2 guardrail factors for `city` datapoint
- **Group 4:** Use case factor + 2 guardrail factors for `state` datapoint
- **Group 5:** Use case factor + 2 guardrail factors for `zip_code` datapoint
- **Group 6:** Use case factor + 2 guardrail factors for `price` datapoint
- **Group 7:** Use case factor + 2 guardrail factors for `bedrooms` datapoint
- **Group 8:** Use case factor + 2 guardrail factors for `bathrooms` datapoint
- **Group 9:** Use case factor + 2 guardrail factors for `square_feet` datapoint
- **Group 10:** Use case factor + 2 guardrail factors for `property_type` datapoint
- **Group 11:** Use case factor + 2 guardrail factors for `listing_date` datapoint
- **Group 12:** Use case factor + 2 guardrail factors for `status` datapoint

**E-commerce Scraper (13 stackable groups):**
- **Group 1:** Use case factor + 2 guardrail factors for `sku` datapoint
- **Group 2:** Use case factor + 2 guardrail factors for `title` datapoint
- **Group 3:** Use case factor + 2 guardrail factors for `description` datapoint
- **Group 4:** Use case factor + 2 guardrail factors for `price` datapoint
- **Group 5:** Use case factor + 2 guardrail factors for `category` datapoint
- **Group 6:** Use case factor + 2 guardrail factors for `brand` datapoint
- **Group 7:** Use case factor + 2 guardrail factors for `availability` datapoint
- **Group 8:** Use case factor + 2 guardrail factors for `rating` datapoint
- **Group 9:** Use case factor + 2 guardrail factors for `reviews_count` datapoint

**Job Market Scraper (13 stackable groups):**
- **Group 1:** Use case factor + 2 guardrail factors for `title` datapoint
- **Group 2:** Use case factor + 2 guardrail factors for `company` datapoint
- **Group 3:** Use case factor + 2 guardrail factors for `location` datapoint
- **Group 4:** Use case factor + 2 guardrail factors for `salary_range` datapoint
- **Group 5:** Use case factor + 2 guardrail factors for `employment_type` datapoint
- **Group 6:** Use case factor + 2 guardrail factors for `experience_level` datapoint
- **Group 7:** Use case factor + 2 guardrail factors for `skills_required` datapoint
- **Group 8:** Use case factor + 2 guardrail factors for `description` datapoint
- **Group 9:** Use case factor + 2 guardrail factors for `posted_date` datapoint
- **Group 10:** Use case factor + 2 guardrail factors for `expiry_date` datapoint

**Event Management System (13 stackable groups):**
- **Group 1:** Use case factor + 2 guardrail factors for `name` datapoint
- **Group 2:** Use case factor + 2 guardrail factors for `date` datapoint
- **Group 3:** Use case factor + 2 guardrail factors for `location` datapoint
- **Group 4:** Use case factor + 2 guardrail factors for `description` datapoint
- **Group 5:** Use case factor + 2 guardrail factors for `capacity` datapoint
- **Group 6:** Use case factor + 2 guardrail factors for `price` datapoint
- **Group 7:** Use case factor + 2 guardrail factors for `organizer` datapoint
- **Group 8:** Use case factor + 2 guardrail factors for `status` datapoint

**Automotive Database (13 stackable groups):**
- **Group 1:** Use case factor + 2 guardrail factors for `vin` datapoint
- **Group 2:** Use case factor + 2 guardrail factors for `make` datapoint
- **Group 3:** Use case factor + 2 guardrail factors for `model` datapoint
- **Group 4:** Use case factor + 2 guardrail factors for `year` datapoint
- **Group 5:** Use case factor + 2 guardrail factors for `color` datapoint
- **Group 6:** Use case factor + 2 guardrail factors for `mileage` datapoint
- **Group 7:** Use case factor + 2 guardrail factors for `price` datapoint
- **Group 8:** Use case factor + 2 guardrail factors for `condition` datapoint
- **Group 9:** Use case factor + 2 guardrail factors for `fuel_type` datapoint
- **Group 10:** Use case factor + 2 guardrail factors for `transmission` datapoint

**ML Model Trainer (13 stackable groups):**
- **Group 1:** Use case factor + 2 guardrail factors for `name` datapoint
- **Group 2:** Use case factor + 2 guardrail factors for `website` datapoint
- **Group 3:** Use case factor + 2 guardrail factors for `industry` datapoint
- **Group 4:** Use case factor + 2 guardrail factors for `employee_count` datapoint
- **Group 5:** Use case factor + 2 guardrail factors for `founded_year` datapoint
- **Group 6:** Use case factor + 2 guardrail factors for `revenue` datapoint
- **Group 7:** Use case factor + 2 guardrail factors for `funding` datapoint
- **Group 8:** Use case factor + 2 guardrail factors for `currency` datapoint
- **Group 9:** Use case factor + 2 guardrail factors for `city` datapoint
- **Group 10:** Use case factor + 2 guardrail factors for `state` datapoint
- **Group 11:** Use case factor + 2 guardrail factors for `country` datapoint
- **Group 12:** Use case factor + 2 guardrail factors for `technologies` datapoint
- **Group 13:** Use case factor + 2 guardrail factors for `status` datapoint

**Fireworks Fine-tuning (13 stackable groups):**
- **Group 1:** Use case factor + 2 guardrail factors for `name` datapoint
- **Group 2:** Use case factor + 2 guardrail factors for `website` datapoint
- **Group 3:** Use case factor + 2 guardrail factors for `industry` datapoint
- **Group 4:** Use case factor + 2 guardrail factors for `employee_count` datapoint
- **Group 5:** Use case factor + 2 guardrail factors for `founded_year` datapoint
- **Group 6:** Use case factor + 2 guardrail factors for `revenue` datapoint
- **Group 7:** Use case factor + 2 guardrail factors for `funding` datapoint
- **Group 8:** Use case factor + 2 guardrail factors for `currency` datapoint
- **Group 9:** Use case factor + 2 guardrail factors for `city` datapoint
- **Group 10:** Use case factor + 2 guardrail factors for `state` datapoint
- **Group 11:** Use case factor + 2 guardrail factors for `country` datapoint
- **Group 12:** Use case factor + 2 guardrail factors for `technologies` datapoint
- **Group 13:** Use case factor + 2 guardrail factors for `status` datapoint

**Step 4: Standalone Factor-Source Records (114 records)**

**AI Company Researcher (39 individual records):**
- Each factor gets its own individual request
- Guardrail factors: Concrete filters and criteria for company search
- Use case factors: Open-ended query parameters for data inclusion

**ESG Agent (36 individual records):**
- Each factor gets its own individual request
- Guardrail factors: Document validation criteria and content filters
- Use case factors: Document analysis parameters and data extraction flags

**ML Model Inference (39 individual records):**
- Each factor gets its own individual request
- Guardrail factors: Inference parameters and model validation criteria
- Use case factors: Prediction parameters and data inclusion flags

**Step 5: Request Template Examples**

**Stackable Request Template (LinkedIn People Dataset):**
```json
{
  "id": "uuid",
  "factor_id": "use-case-name-factor-id",
  "source_id": "linkedin-people-dataset-id",
  "setup": {
    "filters": {
      "location": "San Francisco",
      "experience_years": ">5",
      "skills": ["Python", "JavaScript"],
      "department": "Engineering"
    },
    "include_contact_info": true,
    "include_social_profiles": true
  },
  "key_mapping": "",
  "created_at": "timestamp",
  "updated_at": "timestamp",
  "deleted_at": null
}
```

**Standalone Request Template (AI Company Researcher):**
```json
{
  "id": "uuid",
  "factor_id": "company-size-guardrail-id",
  "source_id": "ai-company-researcher-id",
  "setup": {
    "query": "AI companies in San Francisco with 50-200 employees",
    "industry_focus": "technology",
    "funding_stage": "Series A",
    "include_financials": true
  },
  "key_mapping": "",
  "created_at": "timestamp",
  "updated_at": "timestamp",
  "deleted_at": null
}
```

**Step 6: Factor-Source Validation**

**Purpose:** Ensure all `factor_source` records are properly structured and reference valid IDs

**Validation Requirements:**
- **ID Validation:** All `factor_id` and `source_id` must exist in respective tables
- **Setup Validation:** `setup` JSON must contain realistic request parameters
- **Key Mapping:** `key_mapping` field must be empty string
- **Referential Integrity:** Factor and source must be compatible (same object type)
- **Template Consistency:** Request templates must match source API patterns from Phase 4

**Step 7: Integration with Simulators**

**Purpose:** Ensure `factor_source` records work seamlessly with source simulators

**Integration Requirements:**
- **Request Compatibility:** `factor_source.setup` must generate valid requests for simulators
- **Parameter Mapping:** Request parameters must map to simulator input fields
- **Response Handling:** Simulator responses must be processable by backend ETL
- **Error Handling:** Invalid `factor_source.setup` must generate appropriate error responses
- **Testing:** All `factor_source` records must be testable with corresponding simulators

**Total Factor-Source Records: 218**
- **Stackable Sources:** 104 records (8 sources × 13 groups each)
- **Standalone Sources:** 114 records (3 sources × individual factors)

This comprehensive factor-source mapping system provides realistic request templates that enable proper API communication with source simulators while maintaining data consistency and referential integrity across the entire system.
