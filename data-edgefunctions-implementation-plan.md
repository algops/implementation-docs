# Data Implementation Plan

## Overview

This document provides the comprehensive implementation strategy for generating the target `sources.json` structure from the Supabase database. Based on the database schema analysis, this plan outlines the SQL queries and data transformation logic needed to create the enhanced sources list with runtime statistics, organization data, and activity/workflow relationships.

## Database Schema Analysis

### Key Tables and Relationships

**Core Tables:**
- `public.source` - Source definitions with templates and configuration
- `public.org` - Organization data with metrics
- `public.activity_request` - Activity execution requests
- `public.run_queue` - Job execution queue (partitioned by source)
- `public.activity` - Activity definitions
- `public.workflow` - Workflow definitions
- `public.workflow_request` - Workflow execution requests

**Key Relationships:**
- `source.owner_org_id` → `org.id`
- `activity_request.org_id` → `org.id`
- `activity_request.activity_id` → `activity.id`
- `run_queue.source_id` → `source.id`
- `run_queue.activity_request_id` → `activity_request.id`
- `activity_request.workflow_request_id` → `workflow_request.id`

## Target Data Structure Mapping

### Phase 1: Core Source Data Query

```sql
-- Base source data with organization information
SELECT 
    s.id,
    s.name,
    s.description,
    s.source_type,
    s.delivery_type,
    s.max_concurrent_runs,
    s.timeout,
    s.average_run_duration,
    s.created_at,
    s.updated_at,
    s.owner_org_id,
    o.name as owner_org_name,
    CASE 
        WHEN s.deleted_at IS NOT NULL THEN 'deleted'
        WHEN s.owner_org_id IS NULL THEN 'inactive'
        ELSE 'active'
    END as status
FROM public.source s
LEFT JOIN public.org o ON s.owner_org_id = o.id
WHERE s.deleted_at IS NULL
ORDER BY s.created_at DESC;
```

### Phase 2: Runtime Statistics Query

```sql
-- Runtime statistics per source
WITH source_stats AS (
    SELECT 
        rq.source_id,
        COUNT(*) as total_runs,
        COUNT(CASE WHEN rq.status = 'completed' THEN 1 END) as successful_runs,
        COUNT(CASE WHEN rq.status = 'failed' THEN 1 END) as failed_runs,
        MAX(rq.completed_at) as last_used_at,
        AVG(EXTRACT(EPOCH FROM (rq.completed_at - rq.started_at))) as avg_duration_seconds
    FROM public.run_queue rq
    WHERE rq.deleted_at IS NULL
    GROUP BY rq.source_id
)
SELECT 
    ss.source_id,
    ss.total_runs,
    ss.successful_runs,
    ss.failed_runs,
    ss.last_used_at,
    ROUND(ss.avg_duration_seconds) as average_run_duration,
    CASE 
        WHEN ss.total_runs > 0 
        THEN ROUND(ss.successful_runs::numeric / ss.total_runs::numeric, 3)
        ELSE 0
    END as success_rate
FROM source_stats ss;
```

### Phase 3: Activity and Workflow Counts Query

```sql
-- Activity and workflow counts per source
WITH source_activities AS (
    SELECT 
        rq.source_id,
        COUNT(DISTINCT ar.activity_id) as activities_count,
        COUNT(DISTINCT ar.workflow_request_id) as workflows_count,
        ARRAY_AGG(DISTINCT jsonb_build_object(
            'id', a.id,
            'name', a.name,
            'activity_type', at.name
        )) as activities
    FROM public.run_queue rq
    JOIN public.activity_request ar ON rq.activity_request_id = ar.id
    JOIN public.activity a ON ar.activity_id = a.id
    LEFT JOIN public.activity_type at ON a.activity_type_id = at.id
    WHERE rq.deleted_at IS NULL AND ar.deleted_at IS NULL
    GROUP BY rq.source_id
),
source_workflows AS (
    SELECT 
        rq.source_id,
        ARRAY_AGG(DISTINCT jsonb_build_object(
            'id', wr.id,
            'name', w.name
        )) as workflows
    FROM public.run_queue rq
    JOIN public.activity_request ar ON rq.activity_request_id = ar.id
    JOIN public.workflow_request wr ON ar.workflow_request_id = wr.id
    JOIN public.workflow w ON wr.workflow_id = w.id
    WHERE rq.deleted_at IS NULL AND ar.deleted_at IS NULL
    GROUP BY rq.source_id
)
SELECT 
    sa.source_id,
    sa.activities_count,
    sa.workflows_count,
    sa.activities,
    COALESCE(sw.workflows, '[]'::jsonb) as workflows
FROM source_activities sa
LEFT JOIN source_workflows sw ON sa.source_id = sw.source_id;
```

### Phase 4: Complete Sources Query

```sql
-- Complete sources data with all relationships
WITH source_base AS (
    SELECT 
        s.id,
        s.name,
        s.description,
        s.source_type,
        s.delivery_type,
        s.max_concurrent_runs,
        s.timeout,
        s.created_at,
        s.updated_at,
        s.owner_org_id,
        o.name as owner_org_name,
        CASE 
            WHEN s.deleted_at IS NOT NULL THEN 'deleted'
            WHEN s.owner_org_id IS NULL THEN 'inactive'
            ELSE 'active'
        END as status
    FROM public.source s
    LEFT JOIN public.org o ON s.owner_org_id = o.id
    WHERE s.deleted_at IS NULL
),
source_stats AS (
    SELECT 
        rq.source_id,
        COUNT(*) as total_runs,
        COUNT(CASE WHEN rq.status = 'completed' THEN 1 END) as successful_runs,
        COUNT(CASE WHEN rq.status = 'failed' THEN 1 END) as failed_runs,
        MAX(rq.completed_at) as last_used_at,
        ROUND(AVG(EXTRACT(EPOCH FROM (rq.completed_at - rq.started_at)))) as average_run_duration,
        CASE 
            WHEN COUNT(*) > 0 
            THEN ROUND(COUNT(CASE WHEN rq.status = 'completed' THEN 1 END)::numeric / COUNT(*)::numeric, 3)
            ELSE 0
        END as success_rate
    FROM public.run_queue rq
    WHERE rq.deleted_at IS NULL
    GROUP BY rq.source_id
),
source_activities AS (
    SELECT 
        rq.source_id,
        COUNT(DISTINCT ar.activity_id) as activities_count,
        ARRAY_AGG(DISTINCT jsonb_build_object(
            'id', a.id,
            'name', a.name,
            'activity_type', at.name
        )) as activities
    FROM public.run_queue rq
    JOIN public.activity_request ar ON rq.activity_request_id = ar.id
    JOIN public.activity a ON ar.activity_id = a.id
    LEFT JOIN public.activity_type at ON a.activity_type_id = at.id
    WHERE rq.deleted_at IS NULL AND ar.deleted_at IS NULL
    GROUP BY rq.source_id
),
source_workflows AS (
    SELECT 
        rq.source_id,
        COUNT(DISTINCT ar.workflow_request_id) as workflows_count,
        ARRAY_AGG(DISTINCT jsonb_build_object(
            'id', wr.id,
            'name', w.name
        )) as workflows
    FROM public.run_queue rq
    JOIN public.activity_request ar ON rq.activity_request_id = ar.id
    JOIN public.workflow_request wr ON ar.workflow_request_id = wr.id
    JOIN public.workflow w ON wr.workflow_id = w.id
    WHERE rq.deleted_at IS NULL AND ar.deleted_at IS NULL
    GROUP BY rq.source_id
)
SELECT 
    sb.*,
    COALESCE(ss.total_runs, 0) as total_runs,
    COALESCE(ss.successful_runs, 0) as successful_runs,
    COALESCE(ss.failed_runs, 0) as failed_runs,
    COALESCE(ss.last_used_at, sb.created_at) as last_used_at,
    COALESCE(ss.average_run_duration, 0) as average_run_duration,
    COALESCE(ss.success_rate, 0) as success_rate,
    COALESCE(sa.activities_count, 0) as activities_count,
    COALESCE(sw.workflows_count, 0) as workflows_count,
    COALESCE(sa.activities, '[]'::jsonb) as activities,
    COALESCE(sw.workflows, '[]'::jsonb) as workflows
FROM source_base sb
LEFT JOIN source_stats ss ON sb.id = ss.source_id
LEFT JOIN source_activities sa ON sb.id = sa.source_id
LEFT JOIN source_workflows sw ON sb.id = sw.source_id
ORDER BY sb.created_at DESC;
```

## Data Transformation Logic

### 1. Source Type Mapping

```typescript
const sourceTypeMapping = {
  'dataset': 'Dataset',
  'agent': 'Agent', 
  'integration': 'Integration',
  'ml_model': 'ML model',
  'llm': 'LLM'
};
```

### 2. Delivery Type Mapping

```typescript
const deliveryTypeMapping = {
  'endpoint': 'Endpoint',
  'webhook': 'Webhook'
};
```

### 3. Status Calculation

```typescript
function calculateStatus(source: any): string {
  if (source.deleted_at) return 'deleted';
  if (!source.owner_org_id) return 'inactive';
  return 'active';
}
```

### 4. Success Rate Calculation

```typescript
function calculateSuccessRate(totalRuns: number, successfulRuns: number): number {
  if (totalRuns === 0) return 0;
  return Math.round((successfulRuns / totalRuns) * 1000) / 1000;
}
```

## Implementation Strategy

### Phase 1: Database Query Implementation

1. **Create SQL Query Functions**
   - Implement the complete sources query as a PostgreSQL function
   - Add error handling and performance optimization
   - Create indexes for better query performance

2. **Data Validation**
   - Validate UUID formats
   - Check timestamp formats
   - Ensure numeric values are within expected ranges

### Phase 2: Data Processing Pipeline

1. **Extract Data**
   - Execute the complete sources query
   - Handle database connection and error cases
   - Implement retry logic for failed queries

2. **Transform Data**
   - Apply source type and delivery type mappings
   - Calculate derived fields (status, success_rate)
   - Format timestamps to ISO 8601 format

3. **Generate Output**
   - Create the target JSON structure
   - Sort sources by creation date (newest first)
   - Add metadata (generation timestamp, version)

### Phase 3: Individual Source Detail Generation

1. **Source Configuration Extraction**
   ```sql
   SELECT 
       s.id,
       s.name,
       s.description,
       s.source_type,
       s.delivery_type,
       s.max_concurrent_runs,
       s.timeout,
       s.auth_key,
       s.run_request_template,
       s.status_request_template,
       s.delivery_request_template,
       s.setup,
       s.owner_org_id,
       s.created_at,
       s.updated_at,
       s.average_run_duration
   FROM public.source s
   WHERE s.id = $1 AND s.deleted_at IS NULL;
   ```

2. **Template Processing**
   - Parse JSON template strings
   - Validate template structure
   - Apply variable substitution patterns

3. **Setup Configuration Processing**
   - Parse setup JSON configuration
   - Validate mapping structures
   - Apply source-specific configurations

## Error Handling and Edge Cases

### 1. Missing Data Handling
- Handle NULL values for optional fields
- Provide default values for missing statistics
- Gracefully handle missing organization data

### 2. Data Type Validation
- Validate UUID formats
- Check timestamp validity
- Ensure numeric values are positive

### 3. Performance Considerations
- Implement query timeouts
- Add database connection pooling
- Use pagination for large datasets

## Testing Strategy

### 1. Unit Tests
- Test individual query functions
- Validate data transformation logic
- Test error handling scenarios

### 2. Integration Tests
- Test complete data generation pipeline
- Validate output against target schema
- Test with various database states

### 3. Performance Tests
- Test query performance with large datasets
- Validate memory usage during processing
- Test concurrent data generation

## Success Criteria

1. **Data Accuracy**
   - All sources have correct runtime statistics
   - Organization relationships are properly mapped
   - Activity and workflow counts are accurate

2. **Performance**
   - Query execution time < 5 seconds
   - Memory usage < 100MB for 1000 sources
   - No database connection timeouts

3. **Reliability**
   - Handles all edge cases gracefully
   - Provides meaningful error messages
   - Maintains data consistency

## Next Steps

1. **Implement Database Queries**
   - Create the complete sources query function
   - Add necessary database indexes
   - Test query performance

2. **Build Data Processing Pipeline**
   - Implement data extraction logic
   - Add transformation and validation
   - Create output generation

3. **Add Testing and Validation**
   - Implement comprehensive test suite
   - Add data validation checks
   - Create performance benchmarks

4. **Deploy and Monitor**
   - Deploy to development environment
   - Monitor performance and errors
   - Iterate based on feedback
