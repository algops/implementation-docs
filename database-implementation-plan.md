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

### 2.3. Pricing Model Implementation and Monitoring

**Priority: High**

**Tasks:**
- Create SQL functions for monitoring and controlling pricing model
- Implement usage counting functions for billing tiers
- Create pricing tier validation and enforcement

**Pricing Tiers:**

**Free Trial:**
- 1 source
- 3 user configurations
- 1 activity
- X runs/month
- Unlimited guardrails
- Unlimited use cases

**Standard ($49 per source, $290/month):**
- Best for internal usage
- 10 sources
- 50 activities
- X users
- X runs
- 10 workflows
- Catalog access
- API customization

**Enterprise ($890/month):**
- Best for scaling or running a product for external customers
- 50 sources
- 200 activities
- X users
- X runs
- 100 workflows
- Catalog access
- User-based rate-limits

**Custom:**
- Unlimited volume of anything
- Self hosted option

**SQL Functions to Implement:**
```sql
-- Count sources used by organization
CREATE OR REPLACE FUNCTION count_org_sources(p_org_id uuid) RETURNS integer;

-- Count activities used by organization
CREATE OR REPLACE FUNCTION count_org_activities(p_org_id uuid) RETURNS integer;

-- Count runs per month for organization
CREATE OR REPLACE FUNCTION count_org_runs_monthly(p_org_id uuid, p_month date) RETURNS integer;

-- Count workflows used by organization
CREATE OR REPLACE FUNCTION count_org_workflows(p_org_id uuid) RETURNS integer;

-- Check pricing tier compliance
CREATE OR REPLACE FUNCTION check_pricing_compliance(p_org_id uuid) RETURNS jsonb;

-- Get organization usage summary
CREATE OR REPLACE FUNCTION get_org_usage_summary(p_org_id uuid) RETURNS jsonb;
```

**Schema Enhancements:**
```sql
-- Add pricing tier to org table
ALTER TABLE public.org ADD COLUMN pricing_tier text DEFAULT 'free_trial';
ALTER TABLE public.org ADD COLUMN monthly_run_limit integer;
ALTER TABLE public.org ADD COLUMN source_limit integer;
ALTER TABLE public.org ADD COLUMN activity_limit integer;
ALTER TABLE public.org ADD COLUMN workflow_limit integer;
ALTER TABLE public.org ADD COLUMN user_limit integer;

-- Create usage tracking table
CREATE TABLE public.org_usage_tracking (
    id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
    org_id uuid REFERENCES public.org(id),
    usage_type text NOT NULL, -- 'sources', 'activities', 'runs', 'workflows', 'users'
    current_count integer DEFAULT 0,
    monthly_count integer DEFAULT 0,
    last_reset_date timestamp with time zone DEFAULT now(),
    created_at timestamp with time zone DEFAULT now(),
    updated_at timestamp with time zone DEFAULT now()
);
```

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

### 2.4. Factor Rules System and Guardrail Implementation

**Priority: High**

**Tasks:**
- Implement factor system for use cases and guardrails
- Create guardrail validator system
- Implement factor-based run creation logic
- Create new activity types with guardrail integration

**Factor Types (Already Implemented):**
- `volume` - Numeric values (employee count, revenue)
- `frequency` - Occurrence rates
- `time` - Temporal criteria
- `boolean` - True/false conditions
- `option` - Selection from predefined choices
- `reason` - Categorical reasoning
- `entity` - Object references
- `similarity` - Matching criteria
- `place` - Geographic or location-based

**Factor Usage Types (Already Implemented):**
- `guardrail` - Enforce run conditions and data quality criteria
- `usecase` - Define business criteria for data processing

### 2.5. Advanced Activity Types and Guardrail System

**Priority: High**

**Tasks:**
- Create guardrail validator function
- Implement factor-based run creation process
- Create new activity types with specialized workflows
- Implement training and fine-tuning capabilities
- Create export and prediction activity types

**Guardrail Validator Function:**
```sql
CREATE OR REPLACE FUNCTION validate_guardrails(
    p_activity_request_id uuid,
    p_object_id uuid
) RETURNS jsonb AS $$
DECLARE
    v_validation_result jsonb := '{"valid": true, "errors": []}';
    v_factor_record record;
    v_object_value text;
BEGIN
    -- Check all guardrail factors for the object
    FOR v_factor_record IN
        SELECT f.id, f.name, f.factor_type, f.operator, fs.setup
        FROM public.factor f
        JOIN public.factor_source fs ON fs.factor_id = f.id
        WHERE f.type = 'guardrail'
        AND f.object_type_id = (
            SELECT object_type_id FROM public.object WHERE id = p_object_id
        )
    LOOP
        -- Get object value for this factor
        SELECT v.value INTO v_object_value
        FROM public.value v
        JOIN public.datapoint dp ON dp.id = v.datapoint_id
        WHERE v.object_id = p_object_id
        AND dp.key = v_factor_record.setup->>'datapoint_key'
        LIMIT 1;
        
        -- Validate against factor criteria
        -- Implementation depends on factor_type and operator
        -- Return validation result
    END LOOP;
    
    RETURN v_validation_result;
END;
$$ LANGUAGE plpgsql;
```

**New Activity Types to Implement:**

**Training Activity Process:**
- Takes data from guardrails for particular objects sent to activity
- Creates a new source (duplicate) for training data
- Validates object compliance with guardrails before training
- Generates training datasets from validated objects

**Fine-tuning Activity Process:**
- Takes data from guardrails for particular objects sent to activity
- Sends validated data to fine-tuning service
- Source remains the same, model gets updated
- Validates object compliance with guardrails before fine-tuning

**Export Activity Process:**
- Creates results and sends them as payload to integration
- Handles mapping for export "source"
- Validates data quality through guardrails before export
- Formats data according to integration requirements

**Predict Activity Process:**
- Takes guardrails and target_source data
- Sends validated data to prediction model
- Uses guardrail compliance for prediction accuracy
- Returns predictions with confidence scores

**Activity Type Schema Updates:**
```sql
-- Add new activity types
INSERT INTO public.activity_type (name, description) VALUES
('training', 'Training activity for model improvement'),
('fine_tuning', 'Fine-tuning activity for model optimization'),
('export', 'Export activity for data integration'),
('predict', 'Prediction activity for model inference');

-- Add guardrail validation to activity table
ALTER TABLE public.activity ADD COLUMN guardrail_validation boolean DEFAULT false;
ALTER TABLE public.activity ADD COLUMN guardrail_factors uuid[];

-- Create guardrail validation tracking
CREATE TABLE public.guardrail_validation_log (
    id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
    activity_request_id uuid REFERENCES public.activity_request(id),
    object_id uuid REFERENCES public.object(id),
    validation_result jsonb,
    validation_timestamp timestamp with time zone DEFAULT now(),
    factor_validations jsonb
);
```

**Enhanced Run Creation with Guardrails:**
```sql
CREATE OR REPLACE FUNCTION etl_api.create_run_with_guardrails(
    p_activity_request_id uuid
) RETURNS void AS $$
DECLARE
    v_activity_record record;
    v_object_record record;
    v_validation_result jsonb;
BEGIN
    -- Get activity details
    SELECT a.id, a.guardrail_validation, at.name as activity_type
    INTO v_activity_record
    FROM public.activity a
    JOIN public.activity_type at ON at.id = a.activity_type_id
    JOIN public.activity_request ar ON ar.activity_id = a.id
    WHERE ar.id = p_activity_request_id;
    
    -- Process each object in the request
    FOR v_object_record IN
        SELECT o.id, o.original_object_value_name
        FROM public.object o
        JOIN public.request_object ro ON ro.object_id = o.id
        WHERE ro.request_id = p_activity_request_id
    LOOP
        -- Validate guardrails if required
        IF v_activity_record.guardrail_validation THEN
            v_validation_result := validate_guardrails(p_activity_request_id, v_object_record.id);
            
            IF NOT (v_validation_result->>'valid')::boolean THEN
                -- Skip object or mark as invalid
                INSERT INTO public.guardrail_validation_log 
                (activity_request_id, object_id, validation_result)
                VALUES (p_activity_request_id, v_object_record.id, v_validation_result);
                CONTINUE;
            END IF;
        END IF;
        
        -- Create runs based on activity type
        CASE v_activity_record.activity_type
            WHEN 'training' THEN
                PERFORM etl_api.create_training_run(v_activity_record.id, p_activity_request_id, v_object_record.id);
            WHEN 'fine_tuning' THEN
                PERFORM etl_api.create_fine_tuning_run(v_activity_record.id, p_activity_request_id, v_object_record.id);
            WHEN 'export' THEN
                PERFORM etl_api.create_export_run(v_activity_record.id, p_activity_request_id, v_object_record.id);
            WHEN 'predict' THEN
                PERFORM etl_api.create_predict_run(v_activity_record.id, p_activity_request_id, v_object_record.id);
            ELSE
                PERFORM etl_api.create_run(p_activity_request_id);
        END CASE;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
```

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

### 4.2. Advanced Activity Type Functions

**Priority: High**

**Tasks:**
- Implement specialized functions for new activity types:
  - `etl_api.create_training_run()` - Training activity run creation
  - `etl_api.create_fine_tuning_run()` - Fine-tuning activity run creation
  - `etl_api.create_export_run()` - Export activity run creation
  - `etl_api.create_predict_run()` - Prediction activity run creation
  - `etl_api.process_training_results()` - Training result processing
  - `etl_api.process_fine_tuning_results()` - Fine-tuning result processing
  - `etl_api.process_export_results()` - Export result processing
  - `etl_api.process_predict_results()` - Prediction result processing

**Training Activity Functions:**
```sql
CREATE OR REPLACE FUNCTION etl_api.create_training_run(
    p_activity_id uuid,
    p_activity_request_id uuid,
    p_object_id uuid
) RETURNS void AS $$
DECLARE
    v_source_id uuid;
    v_training_source_id uuid;
    v_guardrail_data jsonb;
BEGIN
    -- Get original source ID
    SELECT source_id INTO v_source_id
    FROM public.run_queue
    WHERE activity_request_id = p_activity_request_id
    LIMIT 1;
    
    -- Create duplicate source for training data
    INSERT INTO public.source (name, source_type, setup, owner_org_id)
    SELECT 
        name || ' (Training)', 
        source_type, 
        setup || jsonb_build_object('training_mode', true, 'original_source_id', v_source_id),
        owner_org_id
    FROM public.source
    WHERE id = v_source_id
    RETURNING id INTO v_training_source_id;
    
    -- Collect guardrail data for training
    SELECT jsonb_agg(
        jsonb_build_object(
            'object_id', p_object_id,
            'factor_data', fs.setup,
            'object_values', (
                SELECT jsonb_object_agg(dp.key, v.value)
                FROM public.value v
                JOIN public.datapoint dp ON dp.id = v.datapoint_id
                WHERE v.object_id = p_object_id
            )
        )
    ) INTO v_guardrail_data
    FROM public.factor f
    JOIN public.factor_source fs ON fs.factor_id = f.id
    WHERE f.type = 'guardrail';
    
    -- Create training run with guardrail data
    INSERT INTO public.run_queue (
        activity_request_id,
        source_id,
        request_object_id,
        status,
        config
    ) VALUES (
        p_activity_request_id,
        v_training_source_id,
        (SELECT id FROM public.request_object WHERE request_id = p_activity_request_id AND object_id = p_object_id),
        'waiting',
        jsonb_build_object(
            'training_data', v_guardrail_data,
            'original_source_id', v_source_id
        )::text
    );
END;
$$ LANGUAGE plpgsql;
```

**Fine-tuning Activity Functions:**
```sql
CREATE OR REPLACE FUNCTION etl_api.create_fine_tuning_run(
    p_activity_id uuid,
    p_activity_request_id uuid,
    p_object_id uuid
) RETURNS void AS $$
DECLARE
    v_source_id uuid;
    v_guardrail_data jsonb;
BEGIN
    -- Get source ID (same as original)
    SELECT source_id INTO v_source_id
    FROM public.run_queue
    WHERE activity_request_id = p_activity_request_id
    LIMIT 1;
    
    -- Collect guardrail data for fine-tuning
    SELECT jsonb_agg(
        jsonb_build_object(
            'object_id', p_object_id,
            'factor_data', fs.setup,
            'object_values', (
                SELECT jsonb_object_agg(dp.key, v.value)
                FROM public.value v
                JOIN public.datapoint dp ON dp.id = v.datapoint_id
                WHERE v.object_id = p_object_id
            )
        )
    ) INTO v_guardrail_data
    FROM public.factor f
    JOIN public.factor_source fs ON fs.factor_id = f.id
    WHERE f.type = 'guardrail';
    
    -- Create fine-tuning run
    INSERT INTO public.run_queue (
        activity_request_id,
        source_id,
        request_object_id,
        status,
        config
    ) VALUES (
        p_activity_request_id,
        v_source_id,
        (SELECT id FROM public.request_object WHERE request_id = p_activity_request_id AND object_id = p_object_id),
        'waiting',
        jsonb_build_object(
            'fine_tuning_data', v_guardrail_data,
            'fine_tuning_mode', true
        )::text
    );
END;
$$ LANGUAGE plpgsql;
```

**Export Activity Functions:**
```sql
CREATE OR REPLACE FUNCTION etl_api.create_export_run(
    p_activity_id uuid,
    p_activity_request_id uuid,
    p_object_id uuid
) RETURNS void AS $$
DECLARE
    v_export_config jsonb;
    v_mapping_config jsonb;
BEGIN
    -- Get export configuration from activity setup
    SELECT setup->'export_config' INTO v_export_config
    FROM public.activity
    WHERE id = p_activity_id;
    
    -- Get mapping configuration for export
    SELECT jsonb_build_object(
        'export_mapping', v_export_config->>'mapping',
        'integration_endpoint', v_export_config->>'endpoint',
        'data_format', v_export_config->>'format'
    ) INTO v_mapping_config;
    
    -- Create export run
    INSERT INTO public.run_queue (
        activity_request_id,
        request_object_id,
        status,
        config
    ) VALUES (
        p_activity_request_id,
        (SELECT id FROM public.request_object WHERE request_id = p_activity_request_id AND object_id = p_object_id),
        'waiting',
        v_mapping_config::text
    );
END;
$$ LANGUAGE plpgsql;
```

**Predict Activity Functions:**
```sql
CREATE OR REPLACE FUNCTION etl_api.create_predict_run(
    p_activity_id uuid,
    p_activity_request_id uuid,
    p_object_id uuid
) RETURNS void AS $$
DECLARE
    v_target_id uuid;
    v_guardrail_data jsonb;
    v_prediction_config jsonb;
BEGIN
    -- Get target configuration
    SELECT jsonb_path_query_first(setup, '$.** ? (@.type == "target")') ->> 'id'
    INTO v_target_id
    FROM public.activity
    WHERE id = p_activity_id;
    
    -- Collect guardrail data for prediction
    SELECT jsonb_agg(
        jsonb_build_object(
            'object_id', p_object_id,
            'factor_data', fs.setup
        )
    ) INTO v_guardrail_data
    FROM public.factor f
    JOIN public.factor_source fs ON fs.factor_id = f.id
    WHERE f.type = 'guardrail';
    
    -- Get target source setup
    SELECT setup INTO v_prediction_config
    FROM public.target_source
    WHERE target_id = v_target_id::uuid;
    
    -- Create prediction run
    INSERT INTO public.run_queue (
        activity_request_id,
        request_object_id,
        status,
        config
    ) VALUES (
        p_activity_request_id,
        (SELECT id FROM public.request_object WHERE request_id = p_activity_request_id AND object_id = p_object_id),
        'waiting',
        jsonb_build_object(
            'prediction_config', v_prediction_config,
            'guardrail_data', v_guardrail_data,
            'target_id', v_target_id
        )::text
    );
END;
$$ LANGUAGE plpgsql;
```

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

### 6.2. Comprehensive RLS Policies Implementation

**Priority: High**

**Tasks:**
- Create detailed RLS policies for all user-facing tables
- Implement granular permissions for different user actions
- Ensure proper data isolation between organizations

**RLS Policy Requirements:**

**Source Table Policies:**
- Can user read the source? (Yes, if user is in same org as source owner)
- Can user create new source? (Yes, if user is authenticated and in org)
- Can user update the source? (Yes, if user is in same org as source owner)
- Can user duplicate the source? (Yes, if user can read source)

**Activity Table Policies:**
- Can user read the activity? (Yes, if user is in same org as activity owner)
- Can user create new activity? (Yes, if user is authenticated and in org)
- Can user update the activity? (Yes, if user is in same org as activity owner)

**Workflow Table Policies:**
- Can user read the workflow? (Yes, if user is in same org as workflow owner)
- Can user create new workflow? (Yes, if user is authenticated and in org)
- Can user update the workflow? (Yes, if user is in same org as workflow owner)

**Value Table Policies:**
- Can user read the value? (Yes, if user is in same org as value owner)

**Object Table Policies:**
- Can user read the object? (Yes, if user is in same org as object owner)

**SQL Implementation:**
```sql
-- Source RLS Policies
CREATE POLICY "Users can read sources from their org"
ON public.source FOR SELECT
TO authenticated
USING (auth.user_in_org(owner_org_id));

CREATE POLICY "Users can create sources in their org"
ON public.source FOR INSERT
TO authenticated
WITH CHECK (auth.user_in_org(owner_org_id));

CREATE POLICY "Users can update sources in their org"
ON public.source FOR UPDATE
TO authenticated
USING (auth.user_in_org(owner_org_id))
WITH CHECK (auth.user_in_org(owner_org_id));

-- Activity RLS Policies
CREATE POLICY "Users can read activities from their org"
ON public.activity FOR SELECT
TO authenticated
USING (auth.user_in_org(org_id));

CREATE POLICY "Users can create activities in their org"
ON public.activity FOR INSERT
TO authenticated
WITH CHECK (auth.user_in_org(org_id));

CREATE POLICY "Users can update activities in their org"
ON public.activity FOR UPDATE
TO authenticated
USING (auth.user_in_org(org_id))
WITH CHECK (auth.user_in_org(org_id));

-- Workflow RLS Policies
CREATE POLICY "Users can read workflows from their org"
ON public.workflow FOR SELECT
TO authenticated
USING (auth.user_in_org(org_id));

CREATE POLICY "Users can create workflows in their org"
ON public.workflow FOR INSERT
TO authenticated
WITH CHECK (auth.user_in_org(org_id));

CREATE POLICY "Users can update workflows in their org"
ON public.workflow FOR UPDATE
TO authenticated
USING (auth.user_in_org(org_id))
WITH CHECK (auth.user_in_org(org_id));

-- Value RLS Policies
CREATE POLICY "Users can read values from their org"
ON public.value FOR SELECT
TO authenticated
USING (
    EXISTS (
        SELECT 1 FROM public.object o
        WHERE o.id = value.object_id
        AND auth.user_in_org(o.org_id)
    )
);

-- Object RLS Policies
CREATE POLICY "Users can read objects from their org"
ON public.object FOR SELECT
TO authenticated
USING (auth.user_in_org(org_id));

-- Activity Request RLS Policies
CREATE POLICY "Users can read activity requests from their org"
ON public.activity_request FOR SELECT
TO authenticated
USING (auth.user_in_org(org_id));

CREATE POLICY "Users can create activity requests in their org"
ON public.activity_request FOR INSERT
TO authenticated
WITH CHECK (auth.user_in_org(org_id));

-- Workflow Request RLS Policies
CREATE POLICY "Users can read workflow requests from their org"
ON public.workflow_request FOR SELECT
TO authenticated
USING (
    EXISTS (
        SELECT 1 FROM public.workflow w
        WHERE w.id = workflow_request.workflow_id
        AND auth.user_in_org(w.org_id)
    )
);

CREATE POLICY "Users can create workflow requests in their org"
ON public.workflow_request FOR INSERT
TO authenticated
WITH CHECK (
    EXISTS (
        SELECT 1 FROM public.workflow w
        WHERE w.id = workflow_request.workflow_id
        AND auth.user_in_org(w.org_id)
    )
);
```

### 6.3. Schema Cleanup and Optimization

**Priority: High**

**Tasks:**
- Remove legacy columns from run_queue and partitions
- Clean up unused copy tables
- Standardize column naming conventions
- Optimize database schema for performance

**Cleanup Actions:**
```sql
-- NOTE: These cleanup actions are NOT included in the latest migration (20250724140658_remote_schema.sql)
-- They need to be applied separately via the 20250725160000_cleanup_legacy_columns.sql migration

-- Remove legacy columns from run_queue and its partitions
ALTER TABLE public.run_queue DROP COLUMN IF EXISTS destination;
ALTER TABLE public.run_queue DROP COLUMN IF EXISTS json_path;
ALTER TABLE public.run_queue DROP COLUMN IF EXISTS result_destination;

-- Remove legacy columns from run_queue partitions
ALTER TABLE IF EXISTS public.run_queue_0bd5ae9575c04f54945b91299c0c7a0b DROP COLUMN IF EXISTS destination;
ALTER TABLE IF EXISTS public.run_queue_0bd5ae9575c04f54945b91299c0c7a0b DROP COLUMN IF EXISTS json_path;
ALTER TABLE IF EXISTS public.run_queue_0bd5ae9575c04f54945b91299c0c7a0b DROP COLUMN IF EXISTS result_destination;

-- Additional partition cleanup (repeat for all partitions)
-- [Similar statements for all other partition tables]

-- Remove copy tables if not needed
DROP TABLE IF EXISTS public.run_queue_copy;
DROP TABLE IF EXISTS public.run_request_copy;
DROP TABLE IF EXISTS public.run_result_copy;

-- Rename key_mapping_json to key_mapping for consistent naming
ALTER TABLE public.datapoint_source RENAME COLUMN key_mapping_json TO key_mapping;

-- Remove setup_json from datapoint_source if marked for deletion
ALTER TABLE IF EXISTS public.datapoint_source DROP COLUMN IF EXISTS setup_json;

-- Update store_value function to use consistent key_mapping naming
CREATE OR REPLACE FUNCTION etl_api.store_value(p_source_id uuid, p_request_id uuid, payload jsonb, p_org_id uuid)
RETURNS void
LANGUAGE plpgsql
AS $function$
-- [Updated function implementation with consistent naming]
$function$;
```

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
1. Database testing framework implementation ✅
2. Organization metrics and analytics
3. Source template system
4. Factor rules system and guardrail implementation
5. ETL function enhancements ✅
6. Pricing model implementation and monitoring
7. Schema cleanup and optimization
8. Comprehensive RLS policies implementation

### Phase 3 (Medium-term - High Priority)
1. Workflow phase dependencies
2. Activity type flow implementation ✅
3. Data mapping system ✅
4. Trigger system ✅
5. Edge function integration ✅
6. Advanced activity types (training, fine-tuning, export, predict)
7. Guardrail validator system implementation

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

### Phase 6 (Final Phase - Critical Priority)
1. **Comprehensive Database Testing Implementation**
2. **Function and Trigger Testing Suite**
3. **Integration Testing Framework**
4. **Performance and Load Testing**
5. **Security and Compliance Testing**

## 11. Comprehensive Database Testing Implementation (Phase 6 - Final Phase)

### 11.1. Function Testing Suite

**Priority: Critical**

**Tasks:**
- Create comprehensive test coverage for all database functions
- Implement unit tests for each function with edge cases
- Create integration tests for function interactions
- Implement performance benchmarks for critical functions

#### 11.1.1. ETL API Functions Testing

**Core ETL Functions:**
```sql
-- Test etl_api.allocate_source_runs()
CREATE OR REPLACE FUNCTION test_allocate_source_runs()
RETURNS SETOF TEXT AS $$
DECLARE
  v_source_id uuid := gen_random_uuid();
  v_count integer;
BEGIN
  -- Test with valid parameters
  PERFORM setup_test_source_data(v_source_id);
  SELECT etl_api.allocate_source_runs(v_source_id, 'test_template', 5, '{"test": true}'::jsonb) INTO v_count;
  RETURN NEXT ok(v_count >= 0, 'allocate_source_runs should return non-negative count');
  
  -- Test with no available slots
  SELECT etl_api.allocate_source_runs(v_source_id, 'test_template', 0, '{"test": true}'::jsonb) INTO v_count;
  RETURN NEXT is(v_count, 0, 'allocate_source_runs should return 0 when no slots available');
  
  -- Test with invalid source_id
  RETURN NEXT throws_ok(
    'SELECT etl_api.allocate_source_runs(''invalid-uuid'', ''template'', 5, ''{}'')',
    'Should throw error for invalid source_id'
  );
  
  PERFORM cleanup_test_source_data(v_source_id);
END;
$$ LANGUAGE plpgsql;

-- Test etl_api.create_run()
CREATE OR REPLACE FUNCTION test_create_run()
RETURNS SETOF TEXT AS $$
DECLARE
  v_activity_request_id uuid := gen_random_uuid();
  v_run_count integer;
BEGIN
  -- Setup test data
  PERFORM setup_test_activity_request_data(v_activity_request_id);
  
  -- Test run creation
  PERFORM etl_api.create_run(v_activity_request_id);
  
  -- Verify run was created
  SELECT COUNT(*) INTO v_run_count FROM public.run_queue WHERE activity_request_id = v_activity_request_id;
  RETURN NEXT ok(v_run_count > 0, 'create_run should create runs in queue');
  
  -- Test with invalid activity_request_id
  RETURN NEXT throws_ok(
    'SELECT etl_api.create_run(''invalid-uuid'')',
    'Should throw error for invalid activity_request_id'
  );
  
  PERFORM cleanup_test_activity_request_data(v_activity_request_id);
END;
$$ LANGUAGE plpgsql;

-- Test etl_api.store_object()
CREATE OR REPLACE FUNCTION test_store_object()
RETURNS SETOF TEXT AS $$
DECLARE
  v_source_id uuid := gen_random_uuid();
  v_request_id uuid := gen_random_uuid();
  v_org_id uuid := gen_random_uuid();
  v_object_count integer;
  v_payload jsonb := '[{"company": "test.com", "industry": "technology"}]'::jsonb;
BEGIN
  -- Setup test data
  PERFORM setup_test_object_storage_data(v_source_id, v_request_id, v_org_id);
  
  -- Test object storage
  PERFORM etl_api.store_object(v_source_id, v_request_id, v_payload, v_org_id);
  
  -- Verify objects were created
  SELECT COUNT(*) INTO v_object_count FROM public.object WHERE org_id = v_org_id;
  RETURN NEXT ok(v_object_count > 0, 'store_object should create objects');
  
  -- Test with empty payload
  PERFORM etl_api.store_object(v_source_id, v_request_id, '[]'::jsonb, v_org_id);
  RETURN NEXT pass('store_object should handle empty payload gracefully');
  
  PERFORM cleanup_test_object_storage_data(v_source_id, v_request_id, v_org_id);
END;
$$ LANGUAGE plpgsql;

-- Test etl_api.store_value()
CREATE OR REPLACE FUNCTION test_store_value()
RETURNS SETOF TEXT AS $$
DECLARE
  v_source_id uuid := gen_random_uuid();
  v_request_id uuid := gen_random_uuid();
  v_org_id uuid := gen_random_uuid();
  v_value_count integer;
  v_payload jsonb := '[{"company": "test.com", "industry": "technology", "employees": 100}]'::jsonb;
BEGIN
  -- Setup test data
  PERFORM setup_test_value_storage_data(v_source_id, v_request_id, v_org_id);
  
  -- Test value storage
  PERFORM etl_api.store_value(v_source_id, v_request_id, v_payload, v_org_id);
  
  -- Verify values were created
  SELECT COUNT(*) INTO v_value_count FROM public.value WHERE activity_request_id = v_request_id;
  RETURN NEXT ok(v_value_count > 0, 'store_value should create values');
  
  -- Test with malformed payload
  PERFORM etl_api.store_value(v_source_id, v_request_id, '{"invalid": "json"}'::jsonb, v_org_id);
  RETURN NEXT pass('store_value should handle malformed payload gracefully');
  
  PERFORM cleanup_test_value_storage_data(v_source_id, v_request_id, v_org_id);
END;
$$ LANGUAGE plpgsql;
```

**Activity Processing Functions:**
```sql
-- Test etl_api.create_run_enrichment()
CREATE OR REPLACE FUNCTION test_create_run_enrichment()
RETURNS SETOF TEXT AS $$
DECLARE
  v_activity_id uuid := gen_random_uuid();
  v_activity_request_id uuid := gen_random_uuid();
  v_run_count integer;
BEGIN
  -- Setup test data
  PERFORM setup_test_enrichment_data(v_activity_id, v_activity_request_id);
  
  -- Test enrichment run creation
  PERFORM etl_api.create_run_enrichment(v_activity_id, v_activity_request_id);
  
  -- Verify runs were created
  SELECT COUNT(*) INTO v_run_count FROM public.run_queue WHERE activity_request_id = v_activity_request_id;
  RETURN NEXT ok(v_run_count > 0, 'create_run_enrichment should create runs');
  
  -- Test with missing datapoints
  PERFORM setup_test_enrichment_missing_datapoints(v_activity_id, v_activity_request_id);
  PERFORM etl_api.create_run_enrichment(v_activity_id, v_activity_request_id);
  
  -- Verify missing_value status
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.run_queue WHERE activity_request_id = v_activity_request_id AND status = 'missing_value'),
    'create_run_enrichment should handle missing datapoints'
  );
  
  PERFORM cleanup_test_enrichment_data(v_activity_id, v_activity_request_id);
END;
$$ LANGUAGE plpgsql;

-- Test etl_api.create_run_extract()
CREATE OR REPLACE FUNCTION test_create_run_extract()
RETURNS SETOF TEXT AS $$
DECLARE
  v_activity_id uuid := gen_random_uuid();
  v_activity_request_id uuid := gen_random_uuid();
  v_run_count integer;
BEGIN
  -- Setup test data
  PERFORM setup_test_extract_data(v_activity_id, v_activity_request_id);
  
  -- Test extract run creation
  PERFORM etl_api.create_run_extract(v_activity_id, v_activity_request_id);
  
  -- Verify runs were created
  SELECT COUNT(*) INTO v_run_count FROM public.run_queue WHERE activity_request_id = v_activity_request_id;
  RETURN NEXT ok(v_run_count > 0, 'create_run_extract should create runs');
  
  -- Test with invalid target configuration
  PERFORM setup_test_extract_invalid_target(v_activity_id, v_activity_request_id);
  RETURN NEXT throws_ok(
    'SELECT etl_api.create_run_extract(' || quote_literal(v_activity_id) || ', ' || quote_literal(v_activity_request_id) || ')',
    'Should throw error for invalid target configuration'
  );
  
  PERFORM cleanup_test_extract_data(v_activity_id, v_activity_request_id);
END;
$$ LANGUAGE plpgsql;

-- Test etl_api.create_run_sourcing()
CREATE OR REPLACE FUNCTION test_create_run_sourcing()
RETURNS SETOF TEXT AS $$
DECLARE
  v_activity_id uuid := gen_random_uuid();
  v_activity_request_id uuid := gen_random_uuid();
  v_run_count integer;
BEGIN
  -- Setup test data
  PERFORM setup_test_sourcing_data(v_activity_id, v_activity_request_id);
  
  -- Test sourcing run creation
  PERFORM etl_api.create_run_sourcing(v_activity_id, v_activity_request_id);
  
  -- Verify runs were created
  SELECT COUNT(*) INTO v_run_count FROM public.run_queue WHERE activity_request_id = v_activity_request_id;
  RETURN NEXT ok(v_run_count > 0, 'create_run_sourcing should create runs');
  
  -- Test with multiple factors
  PERFORM setup_test_sourcing_multi_factor(v_activity_id, v_activity_request_id);
  PERFORM etl_api.create_run_sourcing(v_activity_id, v_activity_request_id);
  
  -- Verify combined factor processing
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.run_queue WHERE activity_request_id = v_activity_request_id AND config LIKE '%factor%'),
    'create_run_sourcing should handle multiple factors'
  );
  
  PERFORM cleanup_test_sourcing_data(v_activity_id, v_activity_request_id);
END;
$$ LANGUAGE plpgsql;
```

#### 11.1.2. Utility Functions Testing

**JSON Processing Functions:**
```sql
-- Test utils.jsonb_resolve_object_names()
CREATE OR REPLACE FUNCTION test_jsonb_resolve_object_names()
RETURNS SETOF TEXT AS $$
DECLARE
  v_input jsonb := '{"company": "test.com", "website": "https://test.com"}'::jsonb;
  v_path jsonb := '["company"]'::jsonb;
  v_result text;
BEGIN
  -- Test with key mapping type
  SELECT object_name INTO v_result FROM utils.jsonb_resolve_object_names(v_input, v_path, 'key') LIMIT 1;
  RETURN NEXT is(v_result, 'test.com', 'jsonb_resolve_object_names should extract company name');
  
  -- Test with value mapping type
  SELECT object_name INTO v_result FROM utils.jsonb_resolve_object_names(v_input, '["website"]'::jsonb, 'value') LIMIT 1;
  RETURN NEXT is(v_result, 'test.com', 'jsonb_resolve_object_names should extract domain from URL');
  
  -- Test with invalid path
  SELECT COUNT(*) INTO v_result FROM utils.jsonb_resolve_object_names(v_input, '["nonexistent"]'::jsonb, 'key');
  RETURN NEXT is(v_result, 0, 'jsonb_resolve_object_names should return empty for invalid path');
END;
$$ LANGUAGE plpgsql;

-- Test utils.replace_request_placeholders()
CREATE OR REPLACE FUNCTION test_replace_request_placeholders()
RETURNS SETOF TEXT AS $$
DECLARE
  v_input text := 'Hello <<$name>>, your ID is <<$id>>';
  v_replacements jsonb := '{"name": "John", "id": "123"}'::jsonb;
  v_result text;
BEGIN
  -- Test placeholder replacement
  SELECT utils.replace_request_placeholders(v_input, v_replacements) INTO v_result;
  RETURN NEXT is(v_result, 'Hello John, your ID is 123', 'replace_request_placeholders should replace placeholders');
  
  -- Test with missing placeholder
  SELECT utils.replace_request_placeholders(v_input, '{"name": "John"}'::jsonb) INTO v_result;
  RETURN NEXT ok(v_result LIKE '%<<$id>>%', 'replace_request_placeholders should leave missing placeholders');
  
  -- Test with empty replacements
  SELECT utils.replace_request_placeholders(v_input, '{}'::jsonb) INTO v_result;
  RETURN NEXT is(v_result, v_input, 'replace_request_placeholders should return original for empty replacements');
END;
$$ LANGUAGE plpgsql;

-- Test utils.resolve_json_variables()
CREATE OR REPLACE FUNCTION test_resolve_json_variables()
RETURNS SETOF TEXT AS $$
DECLARE
  v_input text := 'Search for <<$company>> in <<$location>>';
  v_source_id uuid := gen_random_uuid();
  v_activity_request_id uuid := gen_random_uuid();
  v_object_id uuid := gen_random_uuid();
  v_result text;
  v_all_resolved boolean;
BEGIN
  -- Setup test data
  PERFORM setup_test_variable_resolution_data(v_source_id, v_activity_request_id, v_object_id);
  
  -- Test variable resolution
  SELECT result, all_resolved INTO v_result, v_all_resolved 
  FROM utils.resolve_json_variables(v_input, v_source_id, v_activity_request_id, v_object_id);
  
  RETURN NEXT ok(v_result LIKE '%test%', 'resolve_json_variables should resolve variables');
  RETURN NEXT is(v_all_resolved, true, 'resolve_json_variables should indicate all resolved');
  
  -- Test with unresolvable variables
  SELECT all_resolved INTO v_all_resolved 
  FROM utils.resolve_json_variables('<<$nonexistent>>', v_source_id, v_activity_request_id, v_object_id);
  
  RETURN NEXT is(v_all_resolved, false, 'resolve_json_variables should indicate unresolved variables');
  
  PERFORM cleanup_test_variable_resolution_data(v_source_id, v_activity_request_id, v_object_id);
END;
$$ LANGUAGE plpgsql;
```

**Public Utility Functions:**
```sql
-- Test public.safe_jsonb()
CREATE OR REPLACE FUNCTION test_safe_jsonb()
RETURNS SETOF TEXT AS $$
DECLARE
  v_result jsonb;
BEGIN
  -- Test with valid JSON
  SELECT public.safe_jsonb('{"test": "value"}') INTO v_result;
  RETURN NEXT ok(v_result IS NOT NULL, 'safe_jsonb should parse valid JSON');
  
  -- Test with invalid JSON
  SELECT public.safe_jsonb('invalid json') INTO v_result;
  RETURN NEXT is(v_result, NULL, 'safe_jsonb should return NULL for invalid JSON');
  
  -- Test with null input
  SELECT public.safe_jsonb(NULL) INTO v_result;
  RETURN NEXT is(v_result, NULL, 'safe_jsonb should handle NULL input');
END;
$$ LANGUAGE plpgsql;

-- Test public.clean_safe_jsonb()
CREATE OR REPLACE FUNCTION test_clean_safe_jsonb()
RETURNS SETOF TEXT AS $$
DECLARE
  v_result jsonb;
BEGIN
  -- Test with control characters
  SELECT public.clean_safe_jsonb('{"test": "value\nwith\r\nnewlines"}') INTO v_result;
  RETURN NEXT ok(v_result IS NOT NULL, 'clean_safe_jsonb should clean control characters');
  
  -- Test with invalid JSON
  SELECT public.clean_safe_jsonb('invalid json') INTO v_result;
  RETURN NEXT is(v_result, NULL, 'clean_safe_jsonb should return NULL for invalid JSON');
END;
$$ LANGUAGE plpgsql;
```

### 11.2. Trigger Testing Suite

**Priority: Critical**

**Tasks:**
- Test all trigger functions for proper behavior
- Verify trigger execution order and dependencies
- Test trigger error handling and rollback scenarios

#### 11.2.1. ETL Trigger Functions Testing

```sql
-- Test etl_api.trigger_process_activity_request()
CREATE OR REPLACE FUNCTION test_trigger_process_activity_request()
RETURNS SETOF TEXT AS $$
DECLARE
  v_activity_request_id uuid := gen_random_uuid();
  v_workflow_request_id uuid := gen_random_uuid();
  v_run_count integer;
BEGIN
  -- Setup test data
  PERFORM setup_test_trigger_activity_data(v_activity_request_id, v_workflow_request_id);
  
  -- Test trigger execution
  INSERT INTO public.activity_request (id, workflow_request_id, activity_id, org_id, content, status, workflow_phase)
  VALUES (v_activity_request_id, v_workflow_request_id, gen_random_uuid(), gen_random_uuid(), '{}', 'PENDING', 1);
  
  -- Verify trigger processed the request
  SELECT COUNT(*) INTO v_run_count FROM public.run_queue WHERE activity_request_id = v_activity_request_id;
  RETURN NEXT ok(v_run_count > 0, 'trigger_process_activity_request should create runs');
  
  -- Test phase dependency
  PERFORM setup_test_trigger_phase_dependency(v_workflow_request_id);
  
  -- Verify phase 2 waits for phase 1
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.activity_request WHERE workflow_request_id = v_workflow_request_id AND workflow_phase = 2 AND status = 'PENDING'),
    'trigger_process_activity_request should respect phase dependencies'
  );
  
  PERFORM cleanup_test_trigger_activity_data(v_activity_request_id, v_workflow_request_id);
END;
$$ LANGUAGE plpgsql;

-- Test etl_api.handle_snapshot_response()
CREATE OR REPLACE FUNCTION test_handle_snapshot_response()
RETURNS SETOF TEXT AS $$
DECLARE
  v_snapshot_id text := 'test-snapshot-123';
  v_run_id uuid := gen_random_uuid();
  v_source_id uuid := gen_random_uuid();
BEGIN
  -- Setup test data
  PERFORM setup_test_snapshot_data(v_snapshot_id, v_run_id, v_source_id);
  
  -- Test trigger execution
  INSERT INTO public.snapshot_run_rel (snapshot_id, run_id, source_id)
  VALUES (v_snapshot_id, v_run_id, v_source_id);
  
  -- Verify trigger processed snapshot
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.run_log WHERE source_id = v_source_id),
    'handle_snapshot_response should log snapshot processing'
  );
  
  PERFORM cleanup_test_snapshot_data(v_snapshot_id, v_run_id, v_source_id);
END;
$$ LANGUAGE plpgsql;
```

### 11.3. Integration Testing Framework

**Priority: Critical**

**Tasks:**
- Create end-to-end workflow testing
- Test complete activity processing flows
- Verify data consistency across all stages
- Test error recovery and rollback scenarios

#### 11.3.1. Workflow Processing Integration Tests

```sql
-- Test complete enrichment workflow
CREATE OR REPLACE FUNCTION test_complete_enrichment_workflow()
RETURNS SETOF TEXT AS $$
DECLARE
  v_workflow_request_id uuid := gen_random_uuid();
  v_activity_request_id uuid := gen_random_uuid();
  v_run_count integer;
  v_object_count integer;
  v_value_count integer;
BEGIN
  -- Setup complete workflow data
  PERFORM setup_complete_enrichment_workflow(v_workflow_request_id, v_activity_request_id);
  
  -- Start workflow processing
  PERFORM etl_api.process_workflow_request(v_workflow_request_id);
  
  -- Verify runs were created
  SELECT COUNT(*) INTO v_run_count FROM public.run_queue WHERE activity_request_id = v_activity_request_id;
  RETURN NEXT ok(v_run_count > 0, 'Complete enrichment workflow should create runs');
  
  -- Simulate run completion
  PERFORM simulate_run_completion(v_activity_request_id);
  
  -- Verify objects were stored
  SELECT COUNT(*) INTO v_object_count FROM public.object WHERE request_id = v_activity_request_id;
  RETURN NEXT ok(v_object_count > 0, 'Complete enrichment workflow should store objects');
  
  -- Verify values were stored
  SELECT COUNT(*) INTO v_value_count FROM public.value WHERE activity_request_id = v_activity_request_id;
  RETURN NEXT ok(v_value_count > 0, 'Complete enrichment workflow should store values');
  
  -- Verify activity request completion
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.activity_request WHERE id = v_activity_request_id AND status = 'DONE'),
    'Complete enrichment workflow should complete activity request'
  );
  
  PERFORM cleanup_complete_enrichment_workflow(v_workflow_request_id, v_activity_request_id);
END;
$$ LANGUAGE plpgsql;

-- Test multi-phase workflow processing
CREATE OR REPLACE FUNCTION test_multi_phase_workflow()
RETURNS SETOF TEXT AS $$
DECLARE
  v_workflow_request_id uuid := gen_random_uuid();
  v_phase1_request_id uuid := gen_random_uuid();
  v_phase2_request_id uuid := gen_random_uuid();
BEGIN
  -- Setup multi-phase workflow
  PERFORM setup_multi_phase_workflow(v_workflow_request_id, v_phase1_request_id, v_phase2_request_id);
  
  -- Start workflow processing
  PERFORM etl_api.process_workflow_request(v_workflow_request_id);
  
  -- Verify phase 1 starts immediately
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.run_queue WHERE activity_request_id = v_phase1_request_id),
    'Multi-phase workflow should start phase 1 immediately'
  );
  
  -- Verify phase 2 waits for phase 1
  RETURN NEXT ok(
    NOT EXISTS(SELECT 1 FROM public.run_queue WHERE activity_request_id = v_phase2_request_id),
    'Multi-phase workflow should not start phase 2 until phase 1 completes'
  );
  
  -- Complete phase 1
  PERFORM simulate_run_completion(v_phase1_request_id);
  PERFORM etl_api.try_start_next_workflow_phase(v_phase1_request_id);
  
  -- Verify phase 2 starts after phase 1 completion
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.run_queue WHERE activity_request_id = v_phase2_request_id),
    'Multi-phase workflow should start phase 2 after phase 1 completion'
  );
  
  PERFORM cleanup_multi_phase_workflow(v_workflow_request_id, v_phase1_request_id, v_phase2_request_id);
END;
$$ LANGUAGE plpgsql;
```

#### 11.3.2. Activity Type Integration Tests

```sql
-- Test enrichment activity integration
CREATE OR REPLACE FUNCTION test_enrichment_activity_integration()
RETURNS SETOF TEXT AS $$
DECLARE
  v_activity_request_id uuid := gen_random_uuid();
  v_run_count integer;
  v_completed_runs integer;
BEGIN
  -- Setup enrichment activity
  PERFORM setup_enrichment_activity_integration(v_activity_request_id);
  
  -- Process activity request
  PERFORM etl_api.process_activity_request_by_id(v_activity_request_id);
  
  -- Verify runs created
  SELECT COUNT(*) INTO v_run_count FROM public.run_queue WHERE activity_request_id = v_activity_request_id;
  RETURN NEXT ok(v_run_count > 0, 'Enrichment activity should create runs');
  
  -- Simulate run execution and completion
  PERFORM simulate_enrichment_run_execution(v_activity_request_id);
  
  -- Verify runs completed
  SELECT COUNT(*) INTO v_completed_runs FROM public.run_queue WHERE activity_request_id = v_activity_request_id AND status = 'completed';
  RETURN NEXT ok(v_completed_runs > 0, 'Enrichment activity should complete runs');
  
  -- Verify result processing
  PERFORM etl_api.process_run_results_for_source(gen_random_uuid());
  
  -- Verify values stored
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.value WHERE activity_request_id = v_activity_request_id),
    'Enrichment activity should store values after completion'
  );
  
  PERFORM cleanup_enrichment_activity_integration(v_activity_request_id);
END;
$$ LANGUAGE plpgsql;

-- Test extract activity integration
CREATE OR REPLACE FUNCTION test_extract_activity_integration()
RETURNS SETOF TEXT AS $$
DECLARE
  v_activity_request_id uuid := gen_random_uuid();
  v_new_object_count integer;
BEGIN
  -- Setup extract activity
  PERFORM setup_extract_activity_integration(v_activity_request_id);
  
  -- Process activity request
  PERFORM etl_api.process_activity_request_by_id(v_activity_request_id);
  
  -- Simulate extract execution
  PERFORM simulate_extract_run_execution(v_activity_request_id);
  
  -- Verify new objects discovered
  SELECT COUNT(*) INTO v_new_object_count FROM public.object WHERE request_id = v_activity_request_id;
  RETURN NEXT ok(v_new_object_count > 0, 'Extract activity should discover new objects');
  
  -- Verify object source mappings
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.object_source WHERE source_id IS NOT NULL),
    'Extract activity should create object source mappings'
  );
  
  PERFORM cleanup_extract_activity_integration(v_activity_request_id);
END;
$$ LANGUAGE plpgsql;

-- Test sourcing activity integration
CREATE OR REPLACE FUNCTION test_sourcing_activity_integration()
RETURNS SETOF TEXT AS $$
DECLARE
  v_activity_request_id uuid := gen_random_uuid();
  v_qualified_object_count integer;
BEGIN
  -- Setup sourcing activity
  PERFORM setup_sourcing_activity_integration(v_activity_request_id);
  
  -- Process activity request
  PERFORM etl_api.process_activity_request_by_id(v_activity_request_id);
  
  -- Simulate sourcing execution
  PERFORM simulate_sourcing_run_execution(v_activity_request_id);
  
  -- Verify object qualification
  SELECT COUNT(*) INTO v_qualified_object_count FROM public.object WHERE request_id = v_activity_request_id;
  RETURN NEXT ok(v_qualified_object_count > 0, 'Sourcing activity should qualify objects');
  
  -- Verify factor filtering
  RETURN NEXT ok(
    EXISTS(SELECT 1 FROM public.value WHERE activity_request_id = v_activity_request_id),
    'Sourcing activity should apply factor filtering'
  );
  
  PERFORM cleanup_sourcing_activity_integration(v_activity_request_id);
END;
$$ LANGUAGE plpgsql;
```

### 11.4. Performance and Load Testing

**Priority: High**

**Tasks:**
- Create performance benchmarks for critical functions
- Test system behavior under load
- Verify scalability of database operations
- Monitor resource usage and optimization opportunities

#### 11.4.1. Performance Benchmark Tests

```sql
-- Test function performance benchmarks
CREATE OR REPLACE FUNCTION test_function_performance()
RETURNS SETOF TEXT AS $$
DECLARE
  v_start_time timestamp;
  v_end_time timestamp;
  v_duration interval;
  v_activity_request_id uuid := gen_random_uuid();
BEGIN
  -- Test etl_api.process_activity_request_by_id performance
  PERFORM setup_performance_test_data(v_activity_request_id);
  
  v_start_time := clock_timestamp();
  PERFORM etl_api.process_activity_request_by_id(v_activity_request_id);
  v_end_time := clock_timestamp();
  v_duration := v_end_time - v_start_time;
  
  RETURN NEXT ok(v_duration < interval '1 second', 'process_activity_request_by_id should complete within 1 second');
  
  -- Test etl_api.store_object performance with large payload
  v_start_time := clock_timestamp();
  PERFORM etl_api.store_object(gen_random_uuid(), v_activity_request_id, generate_large_test_payload(), gen_random_uuid());
  v_end_time := clock_timestamp();
  v_duration := v_end_time - v_start_time;
  
  RETURN NEXT ok(v_duration < interval '2 seconds', 'store_object should handle large payloads within 2 seconds');
  
  PERFORM cleanup_performance_test_data(v_activity_request_id);
END;
$$ LANGUAGE plpgsql;

-- Test concurrent processing performance
CREATE OR REPLACE FUNCTION test_concurrent_processing_performance()
RETURNS SETOF TEXT AS $$
DECLARE
  v_concurrent_requests integer := 10;
  v_start_time timestamp;
  v_end_time timestamp;
  v_duration interval;
BEGIN
  -- Setup concurrent test data
  PERFORM setup_concurrent_test_data(v_concurrent_requests);
  
  v_start_time := clock_timestamp();
  
  -- Simulate concurrent activity request processing
  PERFORM simulate_concurrent_activity_processing(v_concurrent_requests);
  
  v_end_time := clock_timestamp();
  v_duration := v_end_time - v_start_time;
  
  RETURN NEXT ok(v_duration < interval '10 seconds', 'Concurrent processing should complete within 10 seconds');
  
  -- Verify no deadlocks or data corruption
  RETURN NEXT ok(
    NOT EXISTS(SELECT 1 FROM public.activity_request WHERE status = 'ERROR'),
    'Concurrent processing should not cause errors'
  );
  
  PERFORM cleanup_concurrent_test_data(v_concurrent_requests);
END;
$$ LANGUAGE plpgsql;
```

### 11.5. Security and Compliance Testing

**Priority: High**

**Tasks:**
- Test RLS policy enforcement
- Verify data isolation between organizations
- Test authentication and authorization
- Validate audit trail functionality

#### 11.5.1. Security Testing Suite

```sql
-- Test RLS policy enforcement
CREATE OR REPLACE FUNCTION test_rls_policy_enforcement()
RETURNS SETOF TEXT AS $$
DECLARE
  v_user_id uuid := gen_random_uuid();
  v_org1_id uuid := gen_random_uuid();
  v_org2_id uuid := gen_random_uuid();
  v_source_id uuid := gen_random_uuid();
  v_access_count integer;
BEGIN
  -- Setup test data with multiple organizations
  PERFORM setup_rls_test_data(v_user_id, v_org1_id, v_org2_id, v_source_id);
  
  -- Set user context to org1
  PERFORM set_user_context(v_user_id, v_org1_id);
  
  -- Test user can access org1 data
  SELECT COUNT(*) INTO v_access_count FROM public.source WHERE owner_org_id = v_org1_id;
  RETURN NEXT ok(v_access_count > 0, 'User should access data from their organization');
  
  -- Test user cannot access org2 data
  SELECT COUNT(*) INTO v_access_count FROM public.source WHERE owner_org_id = v_org2_id;
  RETURN NEXT is(v_access_count, 0, 'User should not access data from other organizations');
  
  -- Test user cannot insert into other organization
  RETURN NEXT throws_ok(
    'INSERT INTO public.source (name, owner_org_id) VALUES (''test'', ' || quote_literal(v_org2_id) || ')',
    'Should not allow inserting into other organization'
  );
  
  PERFORM cleanup_rls_test_data(v_user_id, v_org1_id, v_org2_id, v_source_id);
END;
$$ LANGUAGE plpgsql;

-- Test authentication bypass attempts
CREATE OR REPLACE FUNCTION test_authentication_bypass_attempts()
RETURNS SETOF TEXT AS $$
BEGIN
  -- Test unauthenticated access attempts
  RETURN NEXT throws_ok(
    'SELECT * FROM public.activity',
    'Should not allow unauthenticated access'
  );
  
  -- Test service role access
  PERFORM set_service_role_context();
  RETURN NEXT lives_ok(
    'SELECT COUNT(*) FROM public.activity',
    'Service role should have access to all data'
  );
  
  -- Test authenticated user access
  PERFORM set_authenticated_user_context();
  RETURN NEXT lives_ok(
    'SELECT COUNT(*) FROM public.activity WHERE org_id = current_user_org_id()',
    'Authenticated users should access their organization data'
  );
END;
$$ LANGUAGE plpgsql;
```

### 11.6. Test Data Management and Setup

**Priority: High**

**Tasks:**
- Create comprehensive test data setup functions
- Implement test isolation and cleanup procedures
- Create realistic test scenarios with proper data relationships
- Implement test data validation and consistency checks

#### 11.6.1. Test Data Setup Functions

```sql
-- Comprehensive test data setup
CREATE OR REPLACE FUNCTION setup_comprehensive_test_environment()
RETURNS void AS $$
BEGIN
  -- Create test organizations
  INSERT INTO public.org (id, name) VALUES 
  ('test-org-1', 'Test Organization 1'),
  ('test-org-2', 'Test Organization 2');
  
  -- Create test object types
  INSERT INTO public.object_type (id, name) VALUES 
  ('company-type', 'company'),
  ('person-type', 'person');
  
  -- Create test datapoints
  INSERT INTO public.datapoint (id, key, object_type_id, name) VALUES 
  ('industry-dp', 'industry', 'company-type', 'Industry'),
  ('employees-dp', 'employees', 'company-type', 'Employee Count'),
  ('name-dp', 'name', 'person-type', 'Full Name'),
  ('email-dp', 'email', 'person-type', 'Email Address');
  
  -- Create test sources
  INSERT INTO public.source (id, name, owner_org_id) VALUES 
  ('test-source-1', 'Test Source 1', 'test-org-1'),
  ('test-source-2', 'Test Source 2', 'test-org-2');
  
  -- Create test activity types
  INSERT INTO public.activity_type (id, name, description) VALUES 
  ('enrich-type', 'enrich', 'Enrichment activity'),
  ('extract-type', 'extract', 'Extraction activity'),
  ('source-type', 'source', 'Sourcing activity');
  
  -- Create test mappings
  INSERT INTO public.object_source (id, object_type_id, source_id, key_mapping, key_mapping_type) VALUES 
  ('test-os-1', 'company-type', 'test-source-1', '["company"]', 'key'),
  ('test-os-2', 'person-type', 'test-source-1', '["person"]', 'key');
  
  INSERT INTO public.datapoint_source (datapoint_id, source_id, key_mapping) VALUES 
  ('industry-dp', 'test-source-1', ARRAY['industry']),
  ('employees-dp', 'test-source-1', ARRAY['employee_count']),
  ('name-dp', 'test-source-1', ARRAY['full_name']),
  ('email-dp', 'test-source-1', ARRAY['email']);
END;
$$ LANGUAGE plpgsql;

-- Test data cleanup
CREATE OR REPLACE FUNCTION cleanup_comprehensive_test_environment()
RETURNS void AS $$
BEGIN
  -- Clean up in reverse order to respect foreign key constraints
  DELETE FROM public.datapoint_source WHERE source_id IN ('test-source-1', 'test-source-2');
  DELETE FROM public.object_source WHERE source_id IN ('test-source-1', 'test-source-2');
  DELETE FROM public.source WHERE id IN ('test-source-1', 'test-source-2');
  DELETE FROM public.activity_type WHERE id IN ('enrich-type', 'extract-type', 'source-type');
  DELETE FROM public.datapoint WHERE id IN ('industry-dp', 'employees-dp', 'name-dp', 'email-dp');
  DELETE FROM public.object_type WHERE id IN ('company-type', 'person-type');
  DELETE FROM public.org WHERE id IN ('test-org-1', 'test-org-2');
END;
$$ LANGUAGE plpgsql;
```

### 11.7. Test Execution and Reporting

**Priority: High**

**Tasks:**
- Create automated test execution framework
- Implement test result reporting and analysis
- Create test coverage metrics and monitoring
- Implement continuous integration test pipeline

#### 11.7.1. Test Suite Runner

```sql
-- Master test suite runner
CREATE OR REPLACE FUNCTION run_comprehensive_database_tests()
RETURNS TABLE(test_category text, test_name text, result text, execution_time interval) AS $$
DECLARE
  v_start_time timestamp;
  v_end_time timestamp;
  v_duration interval;
BEGIN
  -- Setup test environment
  PERFORM setup_comprehensive_test_environment();
  
  -- Function Tests
  v_start_time := clock_timestamp();
  PERFORM test_allocate_source_runs();
  v_end_time := clock_timestamp();
  v_duration := v_end_time - v_start_time;
  RETURN QUERY SELECT 'Functions', 'allocate_source_runs', 'PASS', v_duration;
  
  v_start_time := clock_timestamp();
  PERFORM test_create_run();
  v_end_time := clock_timestamp();
  v_duration := v_end_time - v_start_time;
  RETURN QUERY SELECT 'Functions', 'create_run', 'PASS', v_duration;
  
  -- Continue with all other function tests...
  
  -- Trigger Tests
  v_start_time := clock_timestamp();
  PERFORM test_trigger_process_activity_request();
  v_end_time := clock_timestamp();
  v_duration := v_end_time - v_start_time;
  RETURN QUERY SELECT 'Triggers', 'trigger_process_activity_request', 'PASS', v_duration;
  
  -- Integration Tests
  v_start_time := clock_timestamp();
  PERFORM test_complete_enrichment_workflow();
  v_end_time := clock_timestamp();
  v_duration := v_end_time - v_start_time;
  RETURN QUERY SELECT 'Integration', 'complete_enrichment_workflow', 'PASS', v_duration;
  
  -- Performance Tests
  v_start_time := clock_timestamp();
  PERFORM test_function_performance();
  v_end_time := clock_timestamp();
  v_duration := v_end_time - v_start_time;
  RETURN QUERY SELECT 'Performance', 'function_performance', 'PASS', v_duration;
  
  -- Security Tests
  v_start_time := clock_timestamp();
  PERFORM test_rls_policy_enforcement();
  v_end_time := clock_timestamp();
  v_duration := v_end_time - v_start_time;
  RETURN QUERY SELECT 'Security', 'rls_policy_enforcement', 'PASS', v_duration;
  
  -- Cleanup test environment
  PERFORM cleanup_comprehensive_test_environment();
END;
$$ LANGUAGE plpgsql;
```

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
