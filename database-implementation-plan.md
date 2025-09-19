# Database Implementation Plan

This document consolidates all documented planned changes, tasks, and implementation requirements extracted from the database documentation files.

## Overview

The database implementation plan covers the comprehensive Supabase database system including ETL workflows, activity processing, data mapping, testing strategies, and system architecture improvements.

## 1. Standardized State System Implementation (Phase 1 - Critical Priority)

### 1.1. State System Migration

**Priority: Critical**

**Tasks:**
- Create standardized state enums for all tables
- Implement state transition validation functions
- Update all existing functions to use new state system
- Create state synchronization mechanisms
- Implement state monitoring and reporting

**Current Status Mapping:**
- `pending` → `new`
- `validation` → `valid` (temporary state during validation)
- `ACTIVITY_PROCESSED` → `ready`
- `waiting` → `waiting` (with substates)
- `running` → `in_progress`
- `processing` → `in_progress`
- `missing_value` → `invalid_missing_value`
- `completed` → `done`
- `DONE` → `done`
- `failed` → `failed`

**New States to Implement:**
- `timedout` - For timeout handling
- `delivered` - For webhook delivery confirmation
- `invalid_json` - For JSON validation failures

### 1.2. Function Updates Required

**Priority: Critical**

**Functions Requiring State Updates:**

#### 1.2.1. ETL API Functions
- `etl_api.allocate_source_runs()` - Update status transitions from 'waiting' to 'in_progress'
- `etl_api.create_run()` - Update status creation logic
- `etl_api.create_run_enrichment()` - Update 'missing_value' to 'invalid_missing_value'
- `etl_api.create_run_extract()` - Update status creation
- `etl_api.create_run_sourcing()` - Update status creation
- `etl_api.process_activity_request_by_id()` - Update status flow: 'new' → 'valid' → 'ready'
- `etl_api.process_run_results_for_source()` - Update result processing status
- `etl_api.process_workflow_request()` - Update workflow status management
- `etl_api.reconstruct_values_to_activity_request()` - Update final status to 'done'
- `etl_api.store_object()` - Add validation status handling
- `etl_api.store_value()` - Add validation status handling

#### 1.2.2. Trigger Functions
- `etl_api.trigger_process_activity_request()` - Update trigger logic for new states
- `etl_api.trigger_process_workflow_request()` - Update workflow trigger states
- `public.mark_run_request_done()` - Update to use 'done' instead of 'DONE'
- `public.trigger_run_executor()` - Update run execution status handling
- `public.try_start_next_workflow_phase()` - Update phase progression states

#### 1.2.3. Utility Functions
- `public.deliver_activity_result()` - Add 'delivered' status after successful delivery
- `public.json_placeholder_replace()` - Add validation for 'invalid_json' state
- `etl_api.logger()` - Add state transition logging

### 1.3. Database Schema Changes

**Priority: Critical**

**Tasks:**
- Create state enum types for each table
- Add state transition validation constraints
- Create state history tracking tables
- Add state monitoring views
- Implement state synchronization triggers

**Schema Updates:**
```sql
-- Create standardized state enums
CREATE TYPE activity_request_state AS ENUM (
  'new', 'valid', 'invalid', 'waiting', 'ready', 
  'in_progress', 'timedout', 'failed', 'done', 'delivered'
);

CREATE TYPE run_queue_state AS ENUM (
  'new', 'valid', 'invalid', 'waiting', 'ready', 
  'in_progress', 'timedout', 'failed', 'done', 'delivered'
);

CREATE TYPE run_request_state AS ENUM (
  'new', 'valid', 'invalid', 'waiting', 'ready', 
  'in_progress', 'timedout', 'failed', 'done', 'delivered'
);

CREATE TYPE workflow_request_state AS ENUM (
  'new', 'valid', 'invalid', 'waiting', 'ready', 
  'in_progress', 'timedout', 'failed', 'done', 'delivered'
);

-- Add substate columns
ALTER TABLE activity_request ADD COLUMN state_substate text;
ALTER TABLE run_queue ADD COLUMN state_substate text;
ALTER TABLE run_request ADD COLUMN state_substate text;
ALTER TABLE workflow_request ADD COLUMN state_substate text;
```

### 1.4. State Transition Validation

**Priority: Critical**

**Tasks:**
- Create state transition validation functions
- Implement state transition logging
- Add state consistency checks
- Create state recovery mechanisms
- Implement state monitoring alerts

**Validation Rules:**
- `new` → `valid` | `invalid`
- `valid` → `ready` | `waiting`
- `ready` → `in_progress`
- `in_progress` → `done` | `failed` | `timedout`
- `waiting` → `ready` | `invalid` | `timedout`
- `done` → `delivered`

### 1.5. State Monitoring and Reporting

**Priority: High**

**Tasks:**
- Create state monitoring dashboards
- Implement state transition tracking
- Add state performance metrics
- Create state alerting system
- Implement state debugging tools

## 2. Factor System and Natural Language Description Implementation

### 2.1. Factor Type Description System

**Tasks:**
- Create factor type description enum with question words
- Implement natural language description generation functions
- Update factor table structure to include operator field
- Create operator enum for all supported operations

**Factor Type Descriptions:**
```sql
CREATE TYPE factor_type_description AS ENUM (
  'How many',     -- volume
  'Is',           -- boolean
  'What',         -- option
  'When',         -- time
  'Where',        -- place
  'How often',    -- frequency
  'Why',          -- reason
  'What',         -- entity
  'How similar'   -- similarity
);
```

**Operator Enum:**
```sql
CREATE TYPE operator_enum AS ENUM (
  'equals', 'not_equals', 'greater_than', 'greater_equal',
  'less_than', 'less_equal', 'contains', 'not_contains',
  'starts_with', 'ends_with', 'in', 'not_in',
  'between', 'is_null', 'is_not_null', 'like', 'ilike',
  'regex', 'not_regex'
);
```

### 2.2. Factor Table Structure Update

**Tasks:**
- Add operator column to factor table
- Update factor table constraints
- Create factor description generation function

**Schema Updates:**
```sql
-- Add operator column to factor table
ALTER TABLE public.factor ADD COLUMN operator operator_enum;

-- Create factor description generation function
CREATE OR REPLACE FUNCTION generate_factor_description(
  factor_type factor_type_enum,
  object_type_name text,
  datapoint_name text,
  operator_name operator_enum,
  value text,
  is_guardrail boolean DEFAULT true
) RETURNS text AS $$
DECLARE
  description text;
  question_word text;
  operator_phrase text;
BEGIN
  -- Get question word for factor type
  CASE factor_type
    WHEN 'volume' THEN question_word := 'How many';
    WHEN 'boolean' THEN question_word := 'Is';
    WHEN 'option' THEN question_word := 'What';
    WHEN 'time' THEN question_word := 'When';
    WHEN 'place' THEN question_word := 'Where';
    WHEN 'frequency' THEN question_word := 'How often';
    WHEN 'reason' THEN question_word := 'Why';
    WHEN 'entity' THEN question_word := 'What';
    WHEN 'similarity' THEN question_word := 'How similar';
  END CASE;
  
  -- Get operator phrase
  CASE operator_name
    WHEN 'equals' THEN operator_phrase := 'is';
    WHEN 'not_equals' THEN operator_phrase := 'is not';
    WHEN 'greater_than' THEN operator_phrase := 'is more than';
    WHEN 'greater_equal' THEN operator_phrase := 'is at least';
    WHEN 'less_than' THEN operator_phrase := 'is less than';
    WHEN 'less_equal' THEN operator_phrase := 'is at most';
    WHEN 'contains' THEN operator_phrase := 'contains';
    WHEN 'not_contains' THEN operator_phrase := 'does not contain';
    WHEN 'starts_with' THEN operator_phrase := 'starts with';
    WHEN 'ends_with' THEN operator_phrase := 'ends with';
    WHEN 'in' THEN operator_phrase := 'is one of';
    WHEN 'not_in' THEN operator_phrase := 'is not one of';
    WHEN 'between' THEN operator_phrase := 'is between';
    WHEN 'is_null' THEN operator_phrase := 'is empty';
    WHEN 'is_not_null' THEN operator_phrase := 'is not empty';
    WHEN 'like' THEN operator_phrase := 'is like';
    WHEN 'ilike' THEN operator_phrase := 'is like';
    WHEN 'regex' THEN operator_phrase := 'matches';
    WHEN 'not_regex' THEN operator_phrase := 'does not match';
  END CASE;
  
  -- Generate description based on factor type and guardrail/use case
  IF is_guardrail THEN
    CASE factor_type
      WHEN 'boolean' THEN
        description := object_type_name || ' ' || operator_phrase || ' ' || datapoint_name;
      ELSE
        description := object_type_name || '''s ' || datapoint_name || ' ' || operator_phrase || ' ' || value;
    END CASE;
  ELSE
    CASE factor_type
      WHEN 'volume' THEN
        description := question_word || ' ' || datapoint_name || ' does the ' || object_type_name || ' have?';
      WHEN 'boolean' THEN
        description := question_word || ' the ' || object_type_name || ' ' || datapoint_name || '?';
      WHEN 'option' THEN
        description := question_word || ' is the ' || object_type_name || '''s ' || datapoint_name || '?';
      WHEN 'time' THEN
        description := question_word || ' was the ' || object_type_name || ' ' || datapoint_name || '?';
      WHEN 'place' THEN
        description := question_word || ' is the ' || object_type_name || ' ' || datapoint_name || '?';
      WHEN 'frequency' THEN
        description := question_word || ' does the ' || object_type_name || ' ' || datapoint_name || '?';
      WHEN 'reason' THEN
        description := question_word || ' does the ' || object_type_name || ' ' || datapoint_name || '?';
      WHEN 'entity' THEN
        description := question_word || ' ' || datapoint_name || ' does the ' || object_type_name || ' use?';
      WHEN 'similarity' THEN
        description := question_word || ' is the ' || object_type_name || '''s ' || datapoint_name || '?';
    END CASE;
  END IF;
  
  RETURN description;
END;
$$ LANGUAGE plpgsql;
```

### 2.3. Factor Description Matrix

**Tasks:**
- Document complete factor type formula matrix
- Create operator phrase mapping
- Implement validation for factor descriptions

**Factor Type Formula Matrix:**

| Factor Type | Guardrail Formula | Use Case Formula | Example Guardrail | Example Use Case |
|-------------|-------------------|------------------|-------------------|------------------|
| **Volume** | `{object_type}'s {datapoint} {operator_phrase} {value}` | `How many {datapoint} does the {object_type} have?` | `Company's employees is more than 100` | `How many employees does the company have?` |
| **Boolean** | `{object_type} {operator_phrase} {datapoint}` | `Is the {object_type} {datapoint}?` | `Company is public` | `Is the company public?` |
| **Option** | `{object_type}'s {datapoint} {operator_phrase} {value}` | `What is the {object_type}'s {datapoint}?` | `Company's industry is technology` | `What is the company's industry?` |
| **Time** | `{object_type}'s {datapoint} {operator_phrase} {value}` | `When was the {object_type} {datapoint}?` | `Company's founded_date is after 2020` | `When was the company founded?` |
| **Place** | `{object_type}'s {datapoint} {operator_phrase} {value}` | `Where is the {object_type} {datapoint}?` | `Company's location is in San Francisco` | `Where is the company located?` |
| **Frequency** | `{object_type}'s {datapoint} {operator_phrase} {value}` | `How often does the {object_type} {datapoint}?` | `Company's update_frequency is more than weekly` | `How often does the company update?` |
| **Reason** | `{object_type}'s {datapoint} {operator_phrase} {value}` | `Why does the {object_type} {datapoint}?` | `Company's success_reason is innovation` | `Why does the company succeed?` |
| **Entity** | `{object_type}'s {datapoint} {operator_phrase} {value}` | `What {datapoint} does the {object_type} use?` | `Company's technology is React` | `What technology does the company use?` |
| **Similarity** | `{object_type}'s {datapoint} {operator_phrase} {value}` | `How similar is the {object_type}'s {datapoint}?` | `Company's profile is similar to TechCorp` | `How similar is the company's profile?` |

**Operator Phrases Matrix:**

| Operator | Phrase | Example |
|----------|--------|---------|
| **equals** | `is` | `Company's industry is technology` |
| **not_equals** | `is not` | `Company's industry is not healthcare` |
| **greater_than** | `is more than` | `Company's employees is more than 100` |
| **greater_equal** | `is at least` | `Company's employees is at least 50` |
| **less_than** | `is less than` | `Company's employees is less than 1000` |
| **less_equal** | `is at most` | `Company's employees is at most 500` |
| **contains** | `contains` | `Company's description contains AI` |
| **not_contains** | `does not contain` | `Company's description does not contain blockchain` |
| **starts_with** | `starts with` | `Company's name starts with Tech` |
| **ends_with** | `ends with` | `Company's name ends with Corp` |
| **in** | `is one of` | `Company's industry is one of technology, healthcare` |
| **not_in** | `is not one of` | `Company's industry is not one of retail, finance` |
| **between** | `is between` | `Company's employees is between 100 and 500` |
| **is_null** | `is empty` | `Company's description is empty` |
| **is_not_null** | `is not empty` | `Company's description is not empty` |
| **like** | `is like` | `Company's name is like Tech%` |
| **ilike** | `is like` | `Company's name is like tech%` |
| **regex** | `matches` | `Company's name matches ^[A-Z]` |
| **not_regex** | `does not match` | `Company's name does not match ^[0-9]` |

## 3. Database Testing Implementation

### 1.1. Activity Type Testing Framework

**Priority: High**

**Tasks:**
- Implement comprehensive testing strategy for three main activity types: Enrichment, Extraction, and Sourcing
- Deploy pgTAP framework for database testing
- Create test functions for each activity type:
  - `test_enrichment_basic_flow()` - Single company enrichment
  - `test_enrichment_multi_object()` - Multiple companies enrichment
  - `test_enrichment_missing_values()` - Missing/unresolvable variables handling
  - `test_extract_basic_flow()` - Basic extraction with search criteria
  - `test_extract_with_filters()` - Extraction with location/size filters
  - `test_sourcing_basic_flow()` - Basic sourcing with single factor
  - `test_sourcing_multi_factor()` - Sourcing with multiple factors

**Test Categories:**
- Unit Tests: Individual function testing
- Integration Tests: Complete activity flows
- Error Tests: Edge cases and error conditions
- Performance Tests: Large dataset handling

**Implementation Details:**
- Use pgTAP framework for rich assertion library
- Implement test isolation with setup/cleanup functions
- Create realistic test data with actual JSON configurations
- Test error conditions and edge cases

### 1.2. Test Data Management

**Tasks:**
- Create setup functions for each activity type:
  - `setup_enrichment_test_data()`
  - `setup_extract_test_data()`
  - `setup_sourcing_test_data()`
- Implement comprehensive cleanup functions
- Create test data structures for:
  - Organization data
  - Object types (company, person)
  - Datapoints (industry, employees, funding)
  - Sources (AlgOps API Client)
  - Mappings (object_source, datapoint_source)

### 1.3. Cross-Activity Integration Tests

**Tasks:**
- Implement multi-phase workflow testing
- Test activity type routing
- Implement error handling tests
- Create workflow phase testing scenarios

## 2. Database Schema Enhancements

### 2.1. Organization Data and Metrics

**Priority: High**

**Tasks:**
- Enhance `org` table with comprehensive metrics for analytics and billing
- Implement chart-ready data queries for:
  - Monthly request volume
  - Source usage breakdown
  - Activity performance metrics
  - Data quality metrics
  - Cost tracking

**Key Metrics to Implement:**
- Request Volume: `request_count` - Total activity requests
- Execution Volume: `run_count` - Total runs executed
- Data Volume: `object_count` - Total objects stored
- Data Breadth: `datapoint_count` - Total datapoints available
- Usage Limits: `free_objects` - Remaining free tier objects
- Billing Data: `last_charge`, `next_charge` - Payment tracking

### 2.2. Source Template System

**Priority: High**

**Tasks:**
- Implement HTTP request templates for third-party API integrations
- Create template variable system:
  - `<<$run_setup>>` - Configuration from activity setup
  - `<<$max_objects>>` - Maximum objects to process
  - `<<$run_external_id>>` - External ID from third-party service
- Implement template processing with `utils.replace_request_placeholders()`

**Template Types:**
- `run_request_template` - HTTP request structure for API calls
- `status_request_template` - Status checking requests
- `delivery_request_template` - Result retrieval requests

### 2.3. Factor Rules System

**Priority: High**

**Tasks:**
- Implement factor system for use cases and guardrails
- Create factor types:
  - `volume` - Numeric values (employee count, revenue)
  - `frequency` - Occurrence rates
  - `time` - Temporal criteria
  - `boolean` - True/false conditions
  - `option` - Selection from predefined choices
  - `reason` - Categorical reasoning
  - `entity` - Object references
  - `similarity` - Matching criteria
  - `place` - Geographic or location-based

**Use Cases vs Guardrails:**
- Use Cases: Define business criteria for data processing
- Guardrails: Enforce run conditions and data quality criteria

## 3. ETL and Workflow System Improvements

### 3.1. Workflow Phase Dependencies

**Priority: High**

**Tasks:**
- Implement multi-phase workflow system with explicit dependencies
- Create phase progression logic:
  - Sequential execution (phases 1, 2, 3, 4, 5)
  - Activity dependencies via `destination.activity.id`
  - Phase waiting mechanism
  - Status tracking with `workflow_phase` field
  - Automatic progression with `etl_api.try_start_next_workflow_phase()`

**Workflow Structure:**
```json
{
  "data": {
    "activity_key": {
      "id": "activity-uuid",
      "type": "activity",
      "setup": {
        "config": {
          "phase": 1,
          "max_objects": 100
        }
      },
      "destination": {
        "activity": {
          "id": ["dependent-activity-uuid"]
        }
      }
    }
  }
}
```

### 3.2. Activity Type Flow Implementation

**Priority: High**

**Tasks:**
- Implement ENRICHMENT activity flow:
  - Input processing with object storage
  - Run creation with datapoint targeting
  - Run execution via Edge functions
  - Result processing and aggregation
  - Webhook delivery

- Implement EXTRACT activity flow:
  - Search criteria processing
  - Extraction run creation
  - New object discovery
  - Result processing and storage

- Implement SOURCING activity flow:
  - Sourcing criteria processing
  - Filter configuration
  - Object qualification
  - Result processing

### 3.3. Data Mapping System

**Priority: High**

**Tasks:**
- Implement sophisticated mapping system for JSON payload processing
- Create object extraction system using `utils.jsonb_resolve_object_names()`
- Implement value extraction with `key_mapping_json` paths
- Create mapping configuration tables:
  - `object_source` - Object type to source mappings
  - `datapoint_source` - Datapoint to source mappings

**Path Pattern Syntax:**
- `"*"` - Match any key at current level
- `"**"` - Recursively search all nested objects
- `"specific_key"` - Match exact key name
- `["path", "elements"]` - Array of path elements

## 4. Database Function Enhancements

### 4.1. ETL API Functions

**Priority: High**

**Tasks:**
- Implement comprehensive ETL function suite:
  - `etl_api.allocate_source_runs()` - Job allocation
  - `etl_api.create_run()` - Run creation dispatcher
  - `etl_api.create_run_enrichment()` - Enrichment run creation
  - `etl_api.create_run_extract()` - Extraction run creation
  - `etl_api.create_run_sourcing()` - Sourcing run creation
  - `etl_api.process_activity_request_by_id()` - Activity processing
  - `etl_api.process_workflow_request()` - Workflow processing
  - `etl_api.store_object()` - Object storage
  - `etl_api.store_value()` - Value storage
  - `etl_api.reconstruct_values_to_activity_request()` - Result aggregation

### 4.2. Trigger System

**Priority: High**

**Tasks:**
- Implement automated trigger system:
  - `after_insert_workflow_request` - Workflow expansion
  - `trg_process_activity_request` - Activity processing
  - `trigger_run_request_execute` - Edge function execution
  - `trg_mark_run_done` - Status updates
  - `trg_sync_datapoint_source` - Data synchronization

### 4.3. Edge Function Integration

**Priority: High**

**Tasks:**
- Implement Edge function integration for run execution
- Create webhook delivery system
- Implement result processing and aggregation
- Create status checking and monitoring

## 5. Data Quality and Performance

### 5.1. Error Handling and Logging

**Priority: Medium**

**Tasks:**
- Implement comprehensive error handling with `etl_api.etl_log`
- Create logging system with severity levels
- Implement exception handling blocks
- Create troubleshooting and debugging tools

### 5.2. Performance Optimization

**Priority: Medium**

**Tasks:**
- Implement performance monitoring
- Create indexing strategy for key fields
- Implement query optimization
- Create performance metrics tracking

### 5.3. Data Validation

**Priority: Medium**

**Tasks:**
- Implement input validation
- Create data normalization functions
- Implement consistency checks
- Create data quality metrics

## 6. Security and Access Control

### 6.1. Row-Level Security

**Priority: High**

**Tasks:**
- Implement RLS policies for all major tables
- Create organization-based access control
- Implement user authentication and authorization
- Create audit trail system

### 6.2. Data Protection

**Priority: High**

**Tasks:**
- Implement soft delete system with `deleted_at`
- Create data retention policies
- Implement backup and recovery procedures
- Create data encryption for sensitive fields

## 7. Monitoring and Analytics

### 7.1. System Monitoring

**Priority: Medium**

**Tasks:**
- Implement system health monitoring
- Create performance dashboards
- Implement alerting system
- Create usage analytics

### 7.2. Business Intelligence

**Priority: Medium**

**Tasks:**
- Create analytics queries for business metrics
- Implement reporting system
- Create data visualization support
- Implement trend analysis

## 8. Migration and Deployment

### 8.1. Database Migrations

**Priority: High**

**Tasks:**
- Implement migration system for schema changes
- Create rollback procedures
- Implement data migration scripts
- Create deployment automation

### 8.2. Testing and Validation

**Priority: High**

**Tasks:**
- Implement comprehensive test suite
- Create integration testing
- Implement performance testing
- Create user acceptance testing

## 9. Documentation and Maintenance

### 9.1. Documentation Updates

**Priority: Medium**

**Tasks:**
- Keep function documentation up to date
- Create user guides and tutorials
- Implement API documentation
- Create troubleshooting guides

### 9.2. System Maintenance

**Priority: Medium**

**Tasks:**
- Implement regular maintenance procedures
- Create cleanup and archiving processes
- Implement system updates and patches
- Create monitoring and alerting

## 10. Future Enhancements

### 10.1. Scalability Improvements

**Priority: Low**

**Tasks:**
- Implement horizontal scaling
- Create load balancing
- Implement caching strategies
- Create microservices architecture

### 10.2. Advanced Features

**Priority: Low**

**Tasks:**
- Implement machine learning integration
- Create advanced analytics
- Implement real-time processing
- Create AI-powered insights

## Implementation Timeline

### Phase 1 (Immediate - Critical Priority)
1. **Standardized State System Implementation**
   - State system migration
   - Function updates for new states
   - Database schema changes
   - State transition validation
   - State monitoring and reporting

### Phase 2 (Short-term - High Priority)
1. Database testing framework implementation
2. Organization metrics and analytics
3. Source template system
4. Factor rules system
5. ETL function enhancements

### Phase 3 (Medium-term - High Priority)
1. Workflow phase dependencies
2. Activity type flow implementation
3. Data mapping system
4. Trigger system
5. Edge function integration

### Phase 4 (Medium-term - Medium Priority)
1. Error handling and logging
2. Performance optimization
3. Data validation
4. Security enhancements
5. Monitoring and analytics

### Phase 5 (Long-term - Low Priority)
1. Scalability improvements
2. Advanced features
3. Documentation updates
4. System maintenance
5. Future enhancements

## Success Criteria

- All activity types process correctly
- Data flows through the system properly
- Status updates occur as expected
- Results are stored in correct tables
- Error conditions are handled gracefully
- Performance meets requirements
- Security policies are enforced
- Monitoring provides visibility
- Documentation is comprehensive and up-to-date

## Dependencies

- Supabase CLI installation and configuration
- Edge function development and deployment
- Frontend application integration
- Third-party API integrations
- Testing infrastructure setup
- Monitoring and logging systems

---

This implementation plan provides a comprehensive roadmap for database development, testing, and maintenance. Each section includes specific tasks, priorities, and implementation details to guide the development process.
