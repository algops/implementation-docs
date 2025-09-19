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

### Phase 1: Source Data Generation

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

   **File Structure:**
   ```
   utils/
     validation-utils.tsx           // Single file with all validation functions
     types.ts                       // TypeScript interfaces
   ```

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

**Migration Insights (2025-07-25):**
- **Legacy columns removed:** `destination`, `json_path`, `result_destination` from `run_queue`
- **Column renamed:** `key_mapping_json` → `key_mapping` in `datapoint_source`
- **Test data shows:** Company object type with industry/employees/funding datapoints, Technology/Location/Size factors

**Step 1: Create object-types.json (List View)**

**Purpose:** Generate object types data based on actual database table structure

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

**Database Table:** `public.target`
**Fields:** id, name, key, setup, created_at, updated_at, deleted_at, datapoint_id

**Target-Source Relationship Table:** `public.target_source`
**Fields:** id, target_id, source_id, activity_id, setup, created_at, updated_at, deleted_at, key_mapping

**Common Target Setup Patterns:**
- `search_config` - Search criteria for extraction activities
- `query` - Search query string
- `filters` - Additional filtering criteria
- `key_mapping` - Mapping configuration for target-source relationships

**Target Structure:**
```json
{
  "targets": [
    {
      "id": "uuid",
      "name": "Company Target",
      "key": "company_search",
      "setup": {
        "search_config": {
          "query": "AI software companies",
          "filters": {
            "location": "San Francisco",
            "size": ">100"
          }
        }
      },
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
      "setup": {
        "search_config": {
          "query": "AI software companies"
        }
      },
      "created_at": "timestamp",
      "updated_at": "timestamp",
      "deleted_at": null,
      "key_mapping": "text"
    }
  ]
}
```

### Phase 3: Activity Data Generation

**Step 1: Create Individual activity-detail.json Files (Detail View)**

**Purpose:** Populate activity detail pages with full configuration and metadata

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

**Comprehensive Activity Population Plan for Edge Case Testing**

Based on analysis of the database structure, here's the strategic plan to populate each activity to cover all edge cases while maintaining some activities as "perfect" examples:

#### **Activity Classification & Strategy:**

**1. PERFECT ACTIVITIES (3 activities) - Complete, error-free examples**
- **Company Enrichment** (enrich type)
- **Lead Extraction** (extract type)  
- **Prospect Sourcing** (source type)

**Characteristics:**
- Complete setup configuration with all required fields
- Rich source mappings and destination configurations
- Realistic runtime statistics
- Complete metadata and relationships
- Proper verification status

**2. EDGE CASE ACTIVITIES (9 activities) - Each covering specific edge cases**

#### **Edge Case Distribution Plan:**

**Enrichment Activities (3 activities):**
- **Basic Company Enrichment** - Standard enrichment patterns
- **Complex Person Enrichment** - Complex nested mappings and filters
- **Failed Enrichment** - High failure rates and error conditions

**Extraction Activities (3 activities):**
- **Simple Lead Extraction** - Basic extraction with minimal setup
- **Complex Product Extraction** - Complex filters and multiple targets
- **New Extraction** - Zero requests, no runtime statistics

**Sourcing Activities (3 activities):**
- **Factor-Based Sourcing** - Multiple factors and guardrails
- **Simple Sourcing** - Minimal factor configuration
- **Inactive Sourcing** - Deleted/inactive status

#### **Specific Edge Cases to Cover Across All Activities:**

**Activity Type Edge Cases:**
1. **Enrich Activities**: Datapoint destinations, object enrichment patterns
2. **Extract Activities**: Target destinations, discovery patterns
3. **Source Activities**: Factor destinations, discovery criteria

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

**File Structure:**
```
utils/
  validation-utils.tsx           // Extend existing validation functions
  types.ts                       // Add activity-specific interfaces
```

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
