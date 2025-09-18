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

### Phase 1: Create sources.json (List View)

**Purpose:** Populate the sources page with a list of all sources

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
  ]
}
```

**Data Generation Strategy:**
- Use existing 8 demo sources + 2 additional from source_rows.json
- Map `source_type` from source_rows.json to our demo sources
- Generate realistic `average_run_duration` based on source type
- Set appropriate `max_concurrent_runs` and `timeout` values
- Create `status` field (active/inactive/deleted)
- Join with `org` table for `owner_org_name`
- Calculate `last_used_at` from `run_queue.completed_at` MAX
- Calculate run statistics from `run_queue` table (total, successful, failed)
- Calculate `success_rate` from run statistics
- Join with `activity` table via `run_queue` for activities list
- Join with `workflow` table via `activity_request.workflow_request_id` for workflows list
- Generate realistic counts and relationships

### Phase 2: Create Individual source.json Files (Detail View)

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
    "run_external_id_mapping": ["snapshot_id"],
    "status_mapping": {
      "status": "ready"
    },
    "status_translations": {
      "pending": "running",
      "completed": "success",
      "failed": "error"
    }
  },
  "metadata": {
    "owner_org_id": "uuid",
    "created_at": "2025-03-31T11:41:01.452728Z",
    "updated_at": "2025-03-31T14:02:56.342106Z",
    "average_run_duration": 45
  }
}
```

### Phase 3: Template Generation Strategy

**For Each Source Type:**

#### Dataset Sources (LinkedIn, Crunchbase, etc.)
- Use BrightData API pattern from source_rows.json
- Include `run_external_id_mapping` and `status_mapping`
- Set appropriate timeouts (30-60s)
- Template pattern:
  ```json
  {
    "url": "https://api.brightdata.com/datasets/filter",
    "body": {
      "filter": {
        "filters": "<<$run_setup>>",
        "operator": "and"
      },
      "dataset_id": "<<$dataset_id>>",
      "records_limit": "<<$max_objects>>"
    },
    "method": "POST",
    "headers": {
      "Content-Type": "application/json",
      "Authorization": "Bearer <<$auth_token>>"
    }
  }
  ```

#### Agent Sources (AI Company Researcher)
- Use LLM API pattern
- Simpler templates, no status/delivery templates
- Longer timeouts (180s)
- Template pattern:
  ```json
  {
    "url": "https://llmresearcher-108f-8000.prg1.zerops.app/run",
    "body": "<<$run_setup>>",
    "method": "POST",
    "headers": {
      "Content-Type": "application/json"
    }
  }
  ```

#### Integration Sources (AlgOps API, Database)
- Internal API patterns
- Custom authentication
- Variable timeout settings
- Template pattern:
  ```json
  {
    "url": "https://api.algops.com/<<$endpoint>>",
    "body": {
      "query": "<<$query>>",
      "org_id": "<<$org_id>>"
    },
    "method": "POST",
    "headers": {
      "Authorization": "Bearer <<$auth_token>>"
    }
  }
  ```

#### ML Model Sources (Forecasting)
- ML service patterns
- Webhook delivery
- Processing time considerations
- Template pattern:
  ```json
  {
    "url": "https://ml.algops.com/forecast",
    "body": {
      "model": "forecasting_v2",
      "data": "<<$run_setup>>"
    },
    "method": "POST",
    "headers": {
      "Content-Type": "application/json"
    }
  }
  ```

#### LLM Sources (LLM Analyzer)
- Language model API patterns
- Streaming responses
- Token-based authentication
- Template pattern:
  ```json
  {
    "url": "https://llm.algops.com/analyze",
    "body": {
      "prompt": "<<$prompt>>",
      "context": "<<$context>>"
    },
    "method": "POST",
    "headers": {
      "Content-Type": "application/json",
      "Authorization": "Bearer <<$auth_token>>"
    }
  }
  ```

### Phase 4: Variable Substitution Patterns

**Common Variables:**
- `<<$run_setup>>` - Source-specific setup parameters
- `<<$max_objects>>` - Maximum objects to process
- `<<$run_external_id>>` - External run identifier
- `<<$auth_token>>` - Authentication token
- `<<$org_id>>` - Organization identifier

**Source-Specific Variables:**
- Dataset sources: `<<$dataset_id>>`, `<<$filters>>`
- Agent sources: `<<$prompt>>`, `<<$context>>`
- Integration sources: `<<$endpoint>>`, `<<$query>>`
- ML sources: `<<$model>>`, `<<$data>>`
- LLM sources: `<<$prompt>>`, `<<$context>>`

### Phase 5: File Organization

**Directory Structure:**
```
data/
├── sources.json (list view)
├── sources/
│   ├── linkedin-people-dataset/
│   │   ├── source.json (detail view)
│   │   ├── mappings.json
│   │   └── people.json (response data)
│   ├── ai-company-researcher/
│   │   ├── source.json
│   │   ├── mappings.json
│   │   └── companies.json
│   ├── esg-agent/
│   │   ├── source.json
│   │   ├── mappings.json
│   │   └── documents.json
│   ├── job-market-scraper/
│   │   ├── source.json
│   │   ├── mappings.json
│   │   └── jobposts.json
│   ├── e-commerce-scraper/
│   │   ├── source.json
│   │   ├── mappings.json
│   │   └── products.json
│   ├── event-management/
│   │   ├── source.json
│   │   ├── mappings.json
│   │   └── events.json
│   ├── real-estate-api/
│   │   ├── source.json
│   │   ├── mappings.json
│   │   └── properties.json
│   └── automotive-database/
│       ├── source.json
│       ├── mappings.json
│       └── vehicles.json
```

### Phase 6: Validation Requirements

#### sources.json Validation
- Required fields: `id`, `name`, `description`, `source_type`
- Valid `source_type` enum values: `["Dataset", "Agent", "Integration", "ML model", "LLM"]`
- Valid `delivery_type` enum values: `["Endpoint", "Webhook"]`
- Valid UUID format for `id`
- Valid ISO date format for timestamps
- Non-negative integer values for `max_concurrent_runs`, `timeout`, `average_run_duration`

#### source.json Validation
- Template structure validation (url, method, headers, body)
- Variable substitution pattern validation
- Setup structure validation
- Configuration value validation
- Required templates based on source type

### Phase 7: Edge Cases for Testing

#### Template Edge Cases
- Invalid JSON strings in template fields
- Missing variable substitutions (`<<$run_setup>>`, `<<$max_objects>>`, `<<$run_external_id>>`)
- Empty template strings
- Malformed JSON in template strings
- Missing required fields (url, method, headers, body)
- Invalid HTTP methods
- Malformed URLs

#### Setup Edge Cases
- Invalid `run_external_id_mapping` array formats
- Malformed `status_mapping` objects
- Circular `status_translations`
- Missing required setup for source type
- Invalid JSON strings in setup fields

#### Configuration Edge Cases
- Invalid timeout values (negative, too high)
- Invalid `max_concurrent_runs` (zero, negative)
- Missing required configuration for source type
- Invalid `delivery_type` for source type
- Missing authentication for sources that require it

#### Data Type Edge Cases
- Null values in expected fields
- Empty strings vs null values
- Invalid UUID formats
- Invalid timestamp formats
- Type mismatches (string vs number)

#### Integration Edge Cases
- Mappings that reference non-existent response fields
- Mappings for fields that exist but are null/empty
- Mappings that expect arrays but get objects
- Mappings that expect objects but get primitives
- Variable substitution failures

### Phase 8: Implementation Steps

1. **Create Validation Utilities**
   - `utils/validation/mapping-validator.ts`
   - `utils/validation/source-setup-validator.ts`
   - `utils/validation/index.ts`

2. **Generate sources.json**
   - Map existing demo sources to source_rows.json structure
   - Add 2 additional sources from source_rows.json
   - Apply proper data transformations

3. **Generate Individual source.json Files**
   - Create detailed source configurations
   - Generate appropriate templates for each source type
   - Add proper mappings and translations

4. **Create Edge Case Test Data**
   - Generate invalid templates for testing
   - Create malformed mappings
   - Add configuration edge cases

5. **Implement Validation Tests**
   - Test structure validation
   - Test mapping accuracy
   - Test edge case handling
   - Test performance with large datasets

6. **Documentation and Examples**
   - Document template patterns
   - Provide usage examples
   - Create troubleshooting guide

## Success Criteria

- All generated JSON files pass validation
- Templates match expected patterns for each source type
- Variable substitutions work correctly
- Edge cases are properly handled
- Performance is acceptable with large datasets
- Documentation is comprehensive and clear

## Next Steps

1. Review and approve this implementation plan
2. Create validation utilities
3. Generate initial data files
4. Implement comprehensive testing
5. Deploy and validate in development environment
6. Iterate based on feedback and testing results
