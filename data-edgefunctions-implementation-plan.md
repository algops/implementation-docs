# Data Edge Functions Implementation Plan

## Overview

This document provides a comprehensive specification for edge functions that serve as the API layer between the frontend application and the Supabase database. These edge functions handle data transformation, business logic, and AI-powered generation capabilities while maintaining proper authentication, authorization, and error handling.

## Architecture Overview

The edge functions architecture consists of:

1. **Data API Functions** - Transform database queries into frontend-expected formats
2. **Business Logic Functions** - Handle complex operations and workflows  
3. **AI Generation Functions** - Dynamic AI-powered content generation
4. **Utility Functions** - Common functionality and data validation
5. **Authentication & Authorization** - Secure access control

## Comprehensive CRUD Edge Functions Specification

Based on the database schema analysis, here are the complete CRUD operations for all entities with proper RLS policy integration:

### 1. Core Entity CRUD APIs

#### 1.1. Sources API (`/api/sources`)
**RLS Policy:** `auth.user_in_org(owner_org_id)`
**CRUD Operations:**
- `GET /api/sources` - List sources (with stats)
- `GET /api/sources/{id}` - Get specific source
- `POST /api/sources` - Create new source
- `PUT /api/sources/{id}` - Update source
- `DELETE /api/sources/{id}` - Soft delete source

#### 1.2. Activities API (`/api/activities`)
**RLS Policy:** `auth.user_in_org(org_id)`
**CRUD Operations:**
- `GET /api/activities` - List activities (with stats)
- `GET /api/activities/{id}` - Get specific activity
- `POST /api/activities` - Create new activity
- `PUT /api/activities/{id}` - Update activity
- `DELETE /api/activities/{id}` - Soft delete activity

#### 1.3. Workflows API (`/api/workflows`)
**RLS Policy:** `auth.user_in_org(org_id)`
**CRUD Operations:**
- `GET /api/workflows` - List workflows (with stats)
- `GET /api/workflows/{id}` - Get specific workflow
- `POST /api/workflows` - Create new workflow
- `PUT /api/workflows/{id}` - Update workflow
- `DELETE /api/workflows/{id}` - Soft delete workflow

#### 1.4. Objects API (`/api/objects`)
**RLS Policy:** `auth.user_in_org(org_id)`
**CRUD Operations:**
- `GET /api/objects` - List objects (with values)
- `GET /api/objects/{id}` - Get specific object
- `POST /api/objects` - Create new object
- `PUT /api/objects/{id}` - Update object
- `DELETE /api/objects/{id}` - Soft delete object

#### 1.5. Values API (`/api/values`)
**RLS Policy:** Access via object.org_id
**CRUD Operations:**
- `GET /api/values` - List values (with details)
- `GET /api/values/{id}` - Get specific value
- `POST /api/values` - Create new value
- `PUT /api/values/{id}` - Update value
- `DELETE /api/values/{id}` - Soft delete value

#### 1.6. Datapoints API (`/api/datapoints`)
**RLS Policy:** Authenticated users only
**CRUD Operations:**
- `GET /api/datapoints` - List datapoints
- `GET /api/datapoints/{id}` - Get specific datapoint
- `POST /api/datapoints` - Create new datapoint
- `PUT /api/datapoints/{id}` - Update datapoint
- `DELETE /api/datapoints/{id}` - Soft delete datapoint

#### 1.7. Factors API (`/api/factors`)
**RLS Policy:** No org restriction (global)
**CRUD Operations:**
- `GET /api/factors` - List factors
- `GET /api/factors/{id}` - Get specific factor
- `POST /api/factors` - Create new factor
- `PUT /api/factors/{id}` - Update factor
- `DELETE /api/factors/{id}` - Soft delete factor

#### 1.8. Object Types API (`/api/object-types`)
**RLS Policy:** No org restriction (global)
**CRUD Operations:**
- `GET /api/object-types` - List object types
- `GET /api/object-types/{id}` - Get specific object type
- `POST /api/object-types` - Create new object type
- `PUT /api/object-types/{id}` - Update object type
- `DELETE /api/object-types/{id}` - Soft delete object type

#### 1.9. Activity Types API (`/api/activity-types`)
**RLS Policy:** No org restriction (global)
**CRUD Operations:**
- `GET /api/activity-types` - List activity types
- `GET /api/activity-types/{id}` - Get specific activity type
- `POST /api/activity-types` - Create new activity type
- `PUT /api/activity-types/{id}` - Update activity type
- `DELETE /api/activity-types/{id}` - Soft delete activity type

#### 1.10. Activity Requests API (`/api/activity-requests`)
**RLS Policy:** `auth.user_in_org(org_id)`
**CRUD Operations:**
- `GET /api/activity-requests` - List activity requests
- `GET /api/activity-requests/{id}` - Get specific activity request
- `POST /api/activity-requests` - Create new activity request
- `PUT /api/activity-requests/{id}` - Update activity request
- `DELETE /api/activity-requests/{id}` - Soft delete activity request

#### 1.11. Workflow Requests API (`/api/workflow-requests`)
**RLS Policy:** `auth.user_in_org(org_id)`
**CRUD Operations:**
- `GET /api/workflow-requests` - List workflow requests
- `GET /api/workflow-requests/{id}` - Get specific workflow request
- `POST /api/workflow-requests` - Create new workflow request
- `PUT /api/workflow-requests/{id}` - Update workflow request
- `DELETE /api/workflow-requests/{id}` - Soft delete workflow request

### 2. Implementation Architecture

#### 2.1. RLS Policy Mapping

Based on the database schema analysis, here's the RLS policy mapping for each entity:

**Organization-Scoped Entities:**
- `source` → `auth.user_in_org(owner_org_id)`
- `activity` → `auth.user_in_org(org_id)`
- `workflow` → `auth.user_in_org(org_id)`
- `object` → `auth.user_in_org(org_id)`
- `value` → Access via `object.org_id`
- `activity_request` → `auth.user_in_org(org_id)`
- `workflow_request` → `auth.user_in_org(org_id)`

**Authenticated-Only Entities:**
- `datapoint` → `authenticated` users only

**Global Entities (No Org Restriction):**
- `factor` → No org restriction
- `object_type` → No org restriction
- `activity_type` → No org restriction

#### 2.2. Standard CRUD Pattern

Each API follows this standard pattern:

```typescript
// Standard CRUD handler
serve(async (req) => {
  const supabase = createClient(/* ... */)
  
  const { data: { user }, error: authError } = await supabase.auth.getUser()
  if (authError || !user) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), { status: 401 })
  }

  const url = new URL(req.url)
  const method = req.method
  const entityId = url.pathname.split('/').pop()

  switch (method) {
    case 'GET':
      return entityId ? getEntity(supabase, entityId, user) : listEntities(supabase, user, url.searchParams)
    case 'POST':
      return createEntity(supabase, user, await req.json())
    case 'PUT':
      return updateEntity(supabase, entityId, user, await req.json())
    case 'DELETE':
      return deleteEntity(supabase, entityId, user)
    default:
      return new Response(JSON.stringify({ error: 'Method not allowed' }), { status: 405 })
  }
})
```

#### 2.3. Key Implementation Details

**Soft Deletes:**
- All entities use `deleted_at` timestamp for soft deletion
- Queries always filter `WHERE deleted_at IS NULL`
- DELETE operations set `deleted_at = NOW()`

**Pagination:**
- Standard `limit` and `offset` parameters
- Default limit: 100, max limit: 1000
- Response includes metadata with pagination info

**Data Transformation:**
- Database entities transformed to frontend-expected format
- Related entities joined and nested appropriately
- Stats calculated dynamically (success rates, counts, etc.)

**Authentication:**
- JWT token validation via `supabase.auth.getUser()`
- Organization context from `user.user_metadata.org_id`
- RLS policies enforced at database level

#### 2.4. Specific Entity Implementations

**Sources API (`/api/sources`):**
- Includes run statistics (total_runs, success_rate, last_used_at)
- Joins with org table for organization name
- Filters by `owner_org_id` for RLS

**Activities API (`/api/activities`):**
- Includes activity type and object type information
- Calculates request statistics (total_requests, success_rate)
- Filters by `org_id` for RLS

**Objects API (`/api/objects`):**
- Includes related values and source mappings
- Joins with object_type and org tables
- Filters by `org_id` for RLS

**Values API (`/api/values`):**
- Access controlled via object ownership
- Includes datapoint and source details
- Filters through object table for RLS

**Activity/Workflow Requests:**
- Include status tracking and progress information
- Support webhook callbacks and result storage
- Filter by `org_id` for RLS

### 3. AI Generation Functions

**Endpoint:** `GET /api/sources`

**Purpose:** Transform database source data into frontend-expected format

**Endpoint:** `GET /api/sources`

**Authentication:** Required (JWT token)

**Response Format:**
```typescript
interface SourcesResponse {
  sources: Source[];
  metadata: {
    total_count: number;
    page: number;
    page_size: number;
    generated_at: string;
  };
}

interface Source {
  id: string;
  name: string;
  description: string;
  source_type: 'Dataset' | 'Agent' | 'Integration' | 'ML model' | 'LLM';
  delivery_type: 'Endpoint' | 'Webhook';
  max_concurrent_runs: number;
  timeout: number;
  average_run_duration: number;
  created_at: string;
  updated_at: string;
  status: 'active' | 'inactive' | 'deleted';
  owner_org_id: string;
  owner_org_name: string;
  last_used_at: string | null;
  total_runs: number;
  successful_runs: number;
  failed_runs: number;
  success_rate: number;
  activities_count: number;
  workflows_count: number;
}
```

**Implementation:**
```typescript
// Edge function: /supabase/functions/api-sources/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts"
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  try {
    // Authentication
    const supabase = createClient(
      Deno.env.get('SUPABASE_URL') ?? '',
      Deno.env.get('SUPABASE_ANON_KEY') ?? '',
      { global: { headers: { Authorization: req.headers.get('Authorization')! } } }
    )

    // Verify authentication
    const { data: { user }, error: authError } = await supabase.auth.getUser()
    if (authError || !user) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { 'Content-Type': 'application/json' }
      })
    }

    // Execute database query
    const { data, error } = await supabase.rpc('get_sources_with_stats', {
      org_id: user.user_metadata.org_id
    })

    if (error) {
      throw error
    }

    // Transform data to frontend format
    const transformedSources = data.map(transformSourceToFrontendFormat)

    return new Response(JSON.stringify({
      sources: transformedSources,
      metadata: {
        total_count: transformedSources.length,
        page: 1,
        page_size: transformedSources.length,
        generated_at: new Date().toISOString()
      }
    }), {
      status: 200,
      headers: { 'Content-Type': 'application/json' }
    })

  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' }
    })
  }
})

function transformSourceToFrontendFormat(dbSource: any): Source {
  return {
    id: dbSource.id,
    name: dbSource.name,
    description: dbSource.description,
    source_type: mapSourceType(dbSource.source_type),
    delivery_type: mapDeliveryType(dbSource.delivery_type),
    max_concurrent_runs: dbSource.max_concurrent_runs,
    timeout: dbSource.timeout,
    average_run_duration: dbSource.average_run_duration || 0,
    created_at: dbSource.created_at,
    updated_at: dbSource.updated_at,
    status: calculateStatus(dbSource),
    owner_org_id: dbSource.owner_org_id,
    owner_org_name: dbSource.owner_org_name,
    last_used_at: dbSource.last_used_at,
    total_runs: dbSource.total_runs || 0,
    successful_runs: dbSource.successful_runs || 0,
    failed_runs: dbSource.failed_runs || 0,
    success_rate: calculateSuccessRate(dbSource.successful_runs, dbSource.total_runs),
    activities_count: dbSource.activities_count || 0,
    workflows_count: dbSource.workflows_count || 0
  }
}
```

#### 1.2. Activities API (`/api/activities`)

**Endpoint:** `GET /api/activities`

**Purpose:** Provide activities data in frontend format

**Response Format:**
```typescript
interface ActivitiesResponse {
  activities: Activity[];
  metadata: {
    total_count: number;
    page: number;
    page_size: number;
    generated_at: string;
  };
}

interface Activity {
  id: string;
  name: string;
  description: string;
  type: 'enrich' | 'extract' | 'source' | 'training' | 'fine_tuning' | 'export' | 'predict';
  object_type: string;
  status: 'active' | 'inactive' | 'deleted';
  verification_status: boolean;
  owner_org_id: string;
  owner_org_name: string;
  created_at: string;
  updated_at: string;
  last_used_at: string | null;
  total_requests: number;
  successful_requests: number;
  failed_requests: number;
  success_rate: number;
  workflows_count: number;
  average_duration: number;
}
```

#### 1.3. Workflows API (`/api/workflows`)

**Endpoint:** `GET /api/workflows`

**Purpose:** Provide workflows data in frontend format

**Response Format:**
```typescript
interface WorkflowsResponse {
  workflows: Workflow[];
  metadata: {
    total_count: number;
    page: number;
    page_size: number;
    generated_at: string;
  };
}

interface Workflow {
  id: string;
  name: string;
  description: string;
  status: 'active' | 'inactive' | 'deleted';
  verification_status: boolean;
  owner_org_id: string;
  owner_org_name: string;
  created_at: string;
  updated_at: string;
  last_used_at: string | null;
  total_executions: number;
  successful_executions: number;
  failed_executions: number;
  success_rate: number;
  activities_count: number;
  phases_count: number;
  average_duration: number;
}
```

#### 1.4. Dashboard Data API (`/api/dashboard`)

**Endpoint:** `GET /api/dashboard`

**Purpose:** Provide dashboard analytics data

**Response Format:**
```typescript
interface DashboardResponse {
  timePeriods: string[];
  charts: {
    usage: ChartData;
    costs: ChartData;
    performance: ChartData;
    quality: ChartData;
  };
  summary: {
    total_sources: number;
    active_activities: number;
    completed_workflows: number;
    success_rate: number;
  };
}
```

#### 1.5. Data Entities API (`/api/data`)

**Purpose:** Provide access to core data entities (objects, values, datapoints)

##### 1.5.1. Objects API (`/api/data/objects`)

**Endpoint:** `GET /api/data/objects`

**Query Parameters:**
- `object_type` (optional): Filter by object type
- `org_id` (optional): Filter by organization
- `activity_request_id` (optional): Filter by activity request
- `limit` (optional): Number of objects to return (default: 100)
- `offset` (optional): Pagination offset (default: 0)
- `format` (optional): 'json' | 'csv' | 'excel' (default: 'json')

**Response Format:**
```typescript
interface ObjectsResponse {
  objects: Object[];
  metadata: {
    total_count: number;
    page: number;
    page_size: number;
    generated_at: string;
    download_url?: string; // For CSV/Excel formats
  };
}

interface Object {
  id: string;
  object_type: string;
  object_type_name: string;
  original_object_value_name: string;
  org_id: string;
  org_name: string;
  activity_request_id?: string;
  request_id?: string;
  created_at: string;
  updated_at: string;
  values: Value[];
  source_mappings: ObjectSourceMapping[];
}

interface Value {
  id: string;
  datapoint_id: string;
  datapoint_key: string;
  datapoint_name: string;
  value: string | number | boolean | object;
  confidence_score?: number;
  source_id: string;
  source_name: string;
  created_at: string;
}

interface ObjectSourceMapping {
  source_id: string;
  source_name: string;
  key_mapping: string[];
  key_mapping_type: 'key' | 'value';
}
```

**Implementation:**
```typescript
// Edge function: /supabase/functions/api-data-objects/index.ts
serve(async (req) => {
  try {
    const auth = await validateAuth(req)
    if (!auth) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { 'Content-Type': 'application/json' }
      })
    }

    const url = new URL(req.url)
    const objectType = url.searchParams.get('object_type')
    const limit = parseInt(url.searchParams.get('limit') || '100')
    const offset = parseInt(url.searchParams.get('offset') || '0')
    const format = url.searchParams.get('format') || 'json'

    // Execute database query
    const { data, error } = await supabase.rpc('get_objects_with_values', {
      p_org_id: auth.org_id,
      p_object_type: objectType,
      p_limit: limit,
      p_offset: offset
    })

    if (error) throw error

    // Transform data
    const objects = data.map(transformObjectToFrontendFormat)

    if (format === 'csv') {
      const csv = generateCSV(objects)
      return new Response(csv, {
        headers: {
          'Content-Type': 'text/csv',
          'Content-Disposition': `attachment; filename="objects_${Date.now()}.csv"`
        }
      })
    }

    if (format === 'excel') {
      const excelBuffer = await generateExcel(objects)
      return new Response(excelBuffer, {
        headers: {
          'Content-Type': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
          'Content-Disposition': `attachment; filename="objects_${Date.now()}.xlsx"`
        }
      })
    }

    return new Response(JSON.stringify({
      objects,
      metadata: {
        total_count: objects.length,
        page: Math.floor(offset / limit) + 1,
        page_size: limit,
        generated_at: new Date().toISOString()
      }
    }), {
      status: 200,
      headers: { 'Content-Type': 'application/json' }
    })

  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' }
    })
  }
})
```

##### 1.5.2. Values API (`/api/data/values`)

**Endpoint:** `GET /api/data/values`

**Query Parameters:**
- `object_id` (optional): Filter by specific object
- `datapoint_id` (optional): Filter by datapoint
- `source_id` (optional): Filter by source
- `activity_request_id` (optional): Filter by activity request
- `confidence_min` (optional): Minimum confidence score
- `limit` (optional): Number of values to return (default: 1000)
- `offset` (optional): Pagination offset (default: 0)
- `format` (optional): 'json' | 'csv' | 'excel' (default: 'json')

**Response Format:**
```typescript
interface ValuesResponse {
  values: ValueDetail[];
  metadata: {
    total_count: number;
    page: number;
    page_size: number;
    generated_at: string;
    download_url?: string;
  };
}

interface ValueDetail {
  id: string;
  object_id: string;
  object_name: string;
  object_type: string;
  datapoint_id: string;
  datapoint_key: string;
  datapoint_name: string;
  value: string | number | boolean | object;
  confidence_score?: number;
  source_id: string;
  source_name: string;
  activity_request_id?: string;
  created_at: string;
  updated_at: string;
}
```

##### 1.5.3. Datapoints API (`/api/data/datapoints`)

**Endpoint:** `GET /api/data/datapoints`

**Query Parameters:**
- `object_type` (optional): Filter by object type
- `source_id` (optional): Filter by source
- `include_mappings` (optional): Include source mappings (default: false)

**Response Format:**
```typescript
interface DatapointsResponse {
  datapoints: Datapoint[];
  metadata: {
    total_count: number;
    generated_at: string;
  };
}

interface Datapoint {
  id: string;
  key: string;
  name: string;
  description?: string;
  object_type_id: string;
  object_type_name: string;
  data_type: string;
  is_required: boolean;
  created_at: string;
  updated_at: string;
  source_mappings?: DatapointSourceMapping[];
}

interface DatapointSourceMapping {
  source_id: string;
  source_name: string;
  key_mapping: string[];
  setup?: Record<string, any>;
}
```

##### 1.5.4. Data Export API (`/api/data/export`)

**Endpoint:** `POST /api/data/export`

**Purpose:** Create custom data exports with filtering and formatting

**Request Format:**
```typescript
interface DataExportRequest {
  export_type: 'objects' | 'values' | 'combined';
  filters: {
    object_types?: string[];
    datapoints?: string[];
    sources?: string[];
    date_range?: {
      start: string;
      end: string;
    };
    confidence_min?: number;
  };
  format: 'json' | 'csv' | 'excel';
  options?: {
    include_metadata?: boolean;
    group_by?: string;
    aggregation?: Record<string, string>;
  };
}
```

**Response Format:**
```typescript
interface DataExportResponse {
  export_id: string;
  status: 'processing' | 'completed' | 'failed';
  download_url?: string;
  expires_at: string;
  metadata: {
    total_records: number;
    file_size?: number;
    generated_at: string;
  };
}
```

**Implementation:**
```typescript
// Edge function: /supabase/functions/api-data-export/index.ts
serve(async (req) => {
  try {
    const auth = await validateAuth(req)
    if (!auth) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { 'Content-Type': 'application/json' }
      })
    }

    const request: DataExportRequest = await req.json()
    const exportId = crypto.randomUUID()

    // Create export job in database
    await supabase.from('data_export_jobs').insert({
      id: exportId,
      org_id: auth.org_id,
      user_id: auth.user.id,
      export_type: request.export_type,
      filters: request.filters,
      format: request.format,
      options: request.options,
      status: 'processing'
    })

    // Start background processing
    processExportAsync(exportId, request, auth.org_id)

    return new Response(JSON.stringify({
      export_id: exportId,
      status: 'processing',
      expires_at: new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString(), // 24 hours
      metadata: {
        generated_at: new Date().toISOString()
      }
    }), {
      status: 202,
      headers: { 'Content-Type': 'application/json' }
    })

  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' }
    })
  }
})

async function processExportAsync(exportId: string, request: DataExportRequest, orgId: string) {
  try {
    // Execute complex query based on export type
    let data
    if (request.export_type === 'objects') {
      data = await supabase.rpc('export_objects_data', {
        p_org_id: orgId,
        p_filters: request.filters,
        p_options: request.options
      })
    } else if (request.export_type === 'values') {
      data = await supabase.rpc('export_values_data', {
        p_org_id: orgId,
        p_filters: request.filters,
        p_options: request.options
      })
    } else {
      data = await supabase.rpc('export_combined_data', {
        p_org_id: orgId,
        p_filters: request.filters,
        p_options: request.options
      })
    }

    // Generate file based on format
    let fileData
    let fileName
    let contentType

    if (request.format === 'csv') {
      fileData = generateCSV(data.data)
      fileName = `export_${exportId}.csv`
      contentType = 'text/csv'
    } else if (request.format === 'excel') {
      fileData = await generateExcel(data.data)
      fileName = `export_${exportId}.xlsx`
      contentType = 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
    } else {
      fileData = JSON.stringify(data.data)
      fileName = `export_${exportId}.json`
      contentType = 'application/json'
    }

    // Upload to storage
    const { data: uploadData, error: uploadError } = await supabase.storage
      .from('exports')
      .upload(fileName, fileData, {
        contentType,
        upsert: false
      })

    if (uploadError) throw uploadError

    // Get public URL
    const { data: urlData } = supabase.storage
      .from('exports')
      .getPublicUrl(fileName)

    // Update export job
    await supabase.from('data_export_jobs').update({
      status: 'completed',
      download_url: urlData.publicUrl,
      file_size: fileData.length,
      completed_at: new Date().toISOString()
    }).eq('id', exportId)

  } catch (error) {
    // Update export job with error
    await supabase.from('data_export_jobs').update({
      status: 'failed',
      error_message: error.message,
      completed_at: new Date().toISOString()
    }).eq('id', exportId)
  }
}
```

### 2. Business Logic Functions

#### 2.1. Activity Execution (`/api/activities/execute`)

**Endpoint:** `POST /api/activities/execute`

**Purpose:** Execute activities with proper validation and workflow management

**Request Format:**
```typescript
interface ExecuteActivityRequest {
  activity_id: string;
  objects: Array<{
    id?: string;
    data: Record<string, any>;
  }>;
  setup?: Record<string, any>;
  webhook_url?: string;
}
```

#### 2.2. Workflow Execution (`/api/workflows/execute`)

**Endpoint:** `POST /api/workflows/execute`

**Purpose:** Execute workflows with phase management

**Request Format:**
```typescript
interface ExecuteWorkflowRequest {
  workflow_id: string;
  input_data: Record<string, any>;
  webhook_url?: string;
  options?: {
    max_objects?: number;
    timeout?: number;
  };
}
```

### 3. AI Generation Functions

#### 3.1. AI Content Generator (`/api/ai/generate`)

**Purpose:** Dynamic AI-powered content generation with configurable prompts and schemas

**Endpoint:** `POST /api/ai/generate`

**Request Format:**
```typescript
interface AIGenerateRequest {
  scenario: 'source_description' | 'activity_description' | 'workflow_description' | 'factor_description' | 'custom';
  context: {
    object_type?: string;
    datapoints?: string[];
    source_type?: string;
    activity_type?: string;
    custom_prompt?: string;
  };
  output_schema: {
    type: 'json' | 'text' | 'markdown';
    structure?: Record<string, any>;
    max_length?: number;
  };
  options?: {
    model?: 'gpt-4' | 'gpt-3.5-turbo' | 'claude-3';
    temperature?: number;
    max_tokens?: number;
  };
}
```

**Response Format:**
```typescript
interface AIGenerateResponse {
  content: string | Record<string, any>;
  metadata: {
    model_used: string;
    tokens_used: number;
    generation_time: number;
    confidence_score?: number;
  };
  suggestions?: string[];
}
```

**Implementation:**
```typescript
// Edge function: /supabase/functions/api-ai-generate/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts"
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

interface PromptTemplate {
  system: string;
  user: string;
  examples?: Array<{input: any, output: any}>;
}

const PROMPT_TEMPLATES: Record<string, PromptTemplate> = {
  source_description: {
    system: `You are an expert data source analyst. Generate compelling, accurate descriptions for data sources that highlight their capabilities and use cases.`,
    user: `Generate a professional description for a ${context.source_type} data source that provides ${context.datapoints?.join(', ')} data for ${context.object_type} objects. Focus on accuracy, reliability, and practical applications.`,
    examples: [
      {
        input: { source_type: 'API', datapoints: ['company_info', 'funding'], object_type: 'company' },
        output: 'Comprehensive company intelligence API providing detailed business information, funding history, and growth metrics for market research and lead generation.'
      }
    ]
  },
  activity_description: {
    system: `You are a workflow automation expert. Create clear, actionable descriptions for data processing activities.`,
    user: `Generate a description for a ${context.activity_type} activity that processes ${context.object_type} objects using ${context.source_type} data sources.`,
    examples: [
      {
        input: { activity_type: 'enrichment', object_type: 'company', source_type: 'API' },
        output: 'Enriches company profiles with comprehensive business data including industry classification, employee count, and funding information from reliable data sources.'
      }
    ]
  },
  workflow_description: {
    system: `You are a business process architect. Design workflow descriptions that explain complex multi-step data processing pipelines.`,
    user: `Create a workflow description that ${context.custom_prompt || 'processes data through multiple stages'} involving ${context.object_type} objects.`,
    examples: [
      {
        input: { object_type: 'company', custom_prompt: 'discovers, enriches, and analyzes companies' },
        output: 'End-to-end company intelligence workflow that discovers new companies, enriches their profiles with detailed business data, and performs market analysis for strategic decision making.'
      }
    ]
  },
  factor_description: {
    system: `You are a data quality specialist. Generate natural language descriptions for data filtering and validation criteria.`,
    user: `Create a clear description for a ${context.factor_type} factor that validates ${context.datapoint} for ${context.object_type} objects using ${context.operator} ${context.value}.`,
    examples: [
      {
        input: { factor_type: 'volume', datapoint: 'employees', object_type: 'company', operator: 'greater_than', value: '100' },
        output: 'How many employees does the company have? (Must be more than 100 employees)'
      }
    ]
  }
}

serve(async (req) => {
  try {
    // Authentication
    const supabase = createClient(
      Deno.env.get('SUPABASE_URL') ?? '',
      Deno.env.get('SUPABASE_ANON_KEY') ?? '',
      { global: { headers: { Authorization: req.headers.get('Authorization')! } } }
    )

    const { data: { user }, error: authError } = await supabase.auth.getUser()
    if (authError || !user) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { 'Content-Type': 'application/json' }
      })
    }

    const request: AIGenerateRequest = await req.json()
    const startTime = Date.now()

    // Validate request
    if (!request.scenario || !request.context) {
      return new Response(JSON.stringify({ error: 'Missing required fields' }), {
        status: 400,
        headers: { 'Content-Type': 'application/json' }
      })
    }

    // Get prompt template
    const template = PROMPT_TEMPLATES[request.scenario]
    if (!template) {
      return new Response(JSON.stringify({ error: 'Invalid scenario' }), {
        status: 400,
        headers: { 'Content-Type': 'application/json' }
      })
    }

    // Build dynamic prompt
    const systemPrompt = template.system
    const userPrompt = buildDynamicPrompt(template.user, request.context, template.examples)

    // Call OpenAI API
    const openaiResponse = await callOpenAI({
      model: request.options?.model || 'gpt-4',
      temperature: request.options?.temperature || 0.7,
      max_tokens: request.options?.max_tokens || 500,
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt }
      ]
    })

    // Transform response based on output schema
    const content = transformResponse(openaiResponse, request.output_schema)

    return new Response(JSON.stringify({
      content,
      metadata: {
        model_used: request.options?.model || 'gpt-4',
        tokens_used: openaiResponse.usage?.total_tokens || 0,
        generation_time: Date.now() - startTime
      },
      suggestions: generateSuggestions(request.scenario, request.context)
    }), {
      status: 200,
      headers: { 'Content-Type': 'application/json' }
    })

  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { 'Content-Type': 'application/json' }
    })
  }
})

function buildDynamicPrompt(template: string, context: any, examples?: any[]): string {
  let prompt = template
  
  // Replace context variables
  Object.keys(context).forEach(key => {
    const value = context[key]
    if (Array.isArray(value)) {
      prompt = prompt.replace(new RegExp(`\\$\\{${key}\\}`, 'g'), value.join(', '))
    } else {
      prompt = prompt.replace(new RegExp(`\\$\\{${key}\\}`, 'g'), value || '')
    }
  })

  // Add examples if provided
  if (examples && examples.length > 0) {
    prompt += '\n\nExamples:\n'
    examples.forEach((example, index) => {
      prompt += `${index + 1}. Input: ${JSON.stringify(example.input)}\n`
      prompt += `   Output: ${example.output}\n\n`
    })
  }

  return prompt
}

async function callOpenAI(params: any) {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${Deno.env.get('OPENAI_API_KEY')}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(params)
  })

  if (!response.ok) {
    throw new Error(`OpenAI API error: ${response.statusText}`)
  }

  return await response.json()
}

function transformResponse(openaiResponse: any, outputSchema: any): any {
  const content = openaiResponse.choices[0]?.message?.content

  if (outputSchema.type === 'json') {
    try {
      return JSON.parse(content)
    } catch {
      // If parsing fails, return as structured text
      return { text: content }
    }
  }

  return content
}

function generateSuggestions(scenario: string, context: any): string[] {
  const suggestions = []
  
  switch (scenario) {
    case 'source_description':
      suggestions.push('Consider adding performance metrics', 'Include data quality information')
      break
    case 'activity_description':
      suggestions.push('Add expected processing time', 'Include input/output examples')
      break
    case 'workflow_description':
      suggestions.push('Break down into phases', 'Add success criteria')
      break
    case 'factor_description':
      suggestions.push('Add validation examples', 'Include edge case handling')
      break
  }

  return suggestions
}
```

#### 3.2. AI Schema Generator (`/api/ai/generate-schema`)

**Purpose:** Generate JSON schemas for data structures based on natural language descriptions

**Endpoint:** `POST /api/ai/generate-schema`

**Request Format:**
```typescript
interface SchemaGenerateRequest {
  description: string;
  object_type: string;
  requirements: {
    required_fields: string[];
    optional_fields?: string[];
    data_types?: Record<string, string>;
  };
  examples?: Array<Record<string, any>>;
}
```

#### 3.3. AI Factor Generator (`/api/ai/generate-factors`)

**Purpose:** Generate factor configurations for data validation and filtering

**Endpoint:** `POST /api/ai/generate-factors`

**Request Format:**
```typescript
interface FactorGenerateRequest {
  object_type: string;
  datapoints: string[];
  use_case: string;
  guardrail_criteria?: string[];
  custom_requirements?: string;
}
```

### 4. Utility Functions

#### 4.1. Data Validation (`/api/utils/validate`)

**Purpose:** Validate data structures against schemas

#### 4.2. Data Transformation (`/api/utils/transform`)

**Purpose:** Transform data between different formats

#### 4.3. Health Check (`/api/health`)

**Purpose:** System health monitoring

**Endpoint:** `GET /api/health`

**Response Format:**
```typescript
interface HealthResponse {
  status: 'healthy' | 'degraded' | 'unhealthy';
  services: {
    database: ServiceStatus;
    openai: ServiceStatus;
    edge_functions: ServiceStatus;
  };
  timestamp: string;
  version: string;
}

interface ServiceStatus {
  status: 'up' | 'down' | 'degraded';
  response_time?: number;
  last_check: string;
}
```

### 5. Authentication & Authorization

#### 5.1. JWT Token Validation

All edge functions implement consistent authentication:

```typescript
async function validateAuth(req: Request): Promise<{user: any, org_id: string} | null> {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL') ?? '',
    Deno.env.get('SUPABASE_ANON_KEY') ?? '',
    { global: { headers: { Authorization: req.headers.get('Authorization')! } } }
  )

  const { data: { user }, error } = await supabase.auth.getUser()
  if (error || !user) {
    return null
  }

  return {
    user,
    org_id: user.user_metadata.org_id
  }
}
```

#### 5.2. Rate Limiting

Implement rate limiting for AI generation endpoints:

```typescript
async function checkRateLimit(userId: string, endpoint: string): Promise<boolean> {
  const key = `rate_limit:${userId}:${endpoint}`
  const current = await redis.get(key)
  
  if (current && parseInt(current) >= RATE_LIMITS[endpoint]) {
    return false
  }
  
  await redis.incr(key)
  await redis.expire(key, 3600) // 1 hour
  return true
}
```

## Database Query Functions

The following PostgreSQL functions support the edge functions:

### Data Entity Functions

#### get_objects_with_values()

```sql
CREATE OR REPLACE FUNCTION get_objects_with_values(
  p_org_id uuid,
  p_object_type text DEFAULT NULL,
  p_limit integer DEFAULT 100,
  p_offset integer DEFAULT 0
)
RETURNS TABLE(
  id uuid,
  object_type text,
  object_type_name text,
  original_object_value_name text,
  org_id uuid,
  org_name text,
  activity_request_id uuid,
  request_id uuid,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  values jsonb,
  source_mappings jsonb
) AS $$
BEGIN
  RETURN QUERY
  WITH object_data AS (
SELECT 
      o.id,
      ot.name as object_type,
      ot.name as object_type_name,
      o.original_object_value_name,
      o.org_id,
      org.name as org_name,
      o.activity_request_id,
      o.request_id,
      o.created_at,
      o.updated_at
    FROM public.object o
    JOIN public.object_type ot ON o.object_type_id = ot.id
    JOIN public.org org ON o.org_id = org.id
    WHERE o.deleted_at IS NULL
    AND (p_org_id IS NULL OR o.org_id = p_org_id)
    AND (p_object_type IS NULL OR ot.name = p_object_type)
    ORDER BY o.created_at DESC
    LIMIT p_limit
    OFFSET p_offset
  ),
  object_values AS (
    SELECT 
      v.object_id,
      jsonb_agg(
        jsonb_build_object(
          'id', v.id,
          'datapoint_id', v.datapoint_id,
          'datapoint_key', dp.key,
          'datapoint_name', dp.name,
          'value', v.value,
          'confidence_score', v.confidence_score,
          'source_id', v.source_id,
          'source_name', s.name,
          'created_at', v.created_at
        )
      ) as values
    FROM public.value v
    JOIN public.datapoint dp ON v.datapoint_id = dp.id
    LEFT JOIN public.source s ON v.source_id = s.id
    WHERE v.deleted_at IS NULL
    GROUP BY v.object_id
  ),
  object_source_mappings AS (
    SELECT 
      os.object_type_id,
      jsonb_agg(
        jsonb_build_object(
          'source_id', os.source_id,
          'source_name', s.name,
          'key_mapping', os.key_mapping,
          'key_mapping_type', os.key_mapping_type
        )
      ) as source_mappings
    FROM public.object_source os
    JOIN public.source s ON os.source_id = s.id
    GROUP BY os.object_type_id
  )
  SELECT 
    od.id,
    od.object_type,
    od.object_type_name,
    od.original_object_value_name,
    od.org_id,
    od.org_name,
    od.activity_request_id,
    od.request_id,
    od.created_at,
    od.updated_at,
    COALESCE(ov.values, '[]'::jsonb) as values,
    COALESCE(osm.source_mappings, '[]'::jsonb) as source_mappings
  FROM object_data od
  LEFT JOIN object_values ov ON od.id = ov.object_id
  LEFT JOIN object_source_mappings osm ON od.object_type = osm.object_type_id;
END;
$$ LANGUAGE plpgsql;
```

#### get_values_with_details()

```sql
CREATE OR REPLACE FUNCTION get_values_with_details(
  p_org_id uuid,
  p_object_id uuid DEFAULT NULL,
  p_datapoint_id uuid DEFAULT NULL,
  p_source_id uuid DEFAULT NULL,
  p_activity_request_id uuid DEFAULT NULL,
  p_confidence_min numeric DEFAULT NULL,
  p_limit integer DEFAULT 1000,
  p_offset integer DEFAULT 0
)
RETURNS TABLE(
  id uuid,
  object_id uuid,
  object_name text,
  object_type text,
  datapoint_id uuid,
  datapoint_key text,
  datapoint_name text,
  value text,
  confidence_score numeric,
  source_id uuid,
  source_name text,
  activity_request_id uuid,
  created_at timestamp with time zone,
  updated_at timestamp with time zone
) AS $$
BEGIN
  RETURN QUERY
SELECT 
    v.id,
    v.object_id,
    o.original_object_value_name as object_name,
    ot.name as object_type,
    v.datapoint_id,
    dp.key as datapoint_key,
    dp.name as datapoint_name,
    v.value::text,
    v.confidence_score,
    v.source_id,
    s.name as source_name,
    v.activity_request_id,
    v.created_at,
    v.updated_at
  FROM public.value v
  JOIN public.object o ON v.object_id = o.id
  JOIN public.object_type ot ON o.object_type_id = ot.id
  JOIN public.datapoint dp ON v.datapoint_id = dp.id
  LEFT JOIN public.source s ON v.source_id = s.id
  WHERE v.deleted_at IS NULL
  AND (p_org_id IS NULL OR o.org_id = p_org_id)
  AND (p_object_id IS NULL OR v.object_id = p_object_id)
  AND (p_datapoint_id IS NULL OR v.datapoint_id = p_datapoint_id)
  AND (p_source_id IS NULL OR v.source_id = p_source_id)
  AND (p_activity_request_id IS NULL OR v.activity_request_id = p_activity_request_id)
  AND (p_confidence_min IS NULL OR v.confidence_score >= p_confidence_min)
  ORDER BY v.created_at DESC
  LIMIT p_limit
  OFFSET p_offset;
END;
$$ LANGUAGE plpgsql;
```

#### get_datapoints_with_mappings()

```sql
CREATE OR REPLACE FUNCTION get_datapoints_with_mappings(
  p_org_id uuid,
  p_object_type text DEFAULT NULL,
  p_source_id uuid DEFAULT NULL,
  p_include_mappings boolean DEFAULT false
)
RETURNS TABLE(
  id uuid,
  key text,
  name text,
  description text,
  object_type_id uuid,
  object_type_name text,
  data_type text,
  is_required boolean,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  source_mappings jsonb
) AS $$
BEGIN
  RETURN QUERY
  WITH datapoint_base AS (
    SELECT 
      dp.id,
      dp.key,
      dp.name,
      dp.description,
      dp.object_type_id,
      ot.name as object_type_name,
      dp.data_type,
      dp.is_required,
      dp.created_at,
      dp.updated_at
    FROM public.datapoint dp
    JOIN public.object_type ot ON dp.object_type_id = ot.id
    WHERE dp.deleted_at IS NULL
    AND (p_object_type IS NULL OR ot.name = p_object_type)
  ),
  datapoint_source_mappings AS (
    SELECT 
      dsm.datapoint_id,
      jsonb_agg(
        jsonb_build_object(
          'source_id', dsm.source_id,
          'source_name', s.name,
          'key_mapping', dsm.key_mapping,
          'setup', dsm.setup
        )
      ) as source_mappings
    FROM public.datapoint_source dsm
    JOIN public.source s ON dsm.source_id = s.id
    WHERE (p_source_id IS NULL OR dsm.source_id = p_source_id)
    GROUP BY dsm.datapoint_id
)
SELECT 
    db.id,
    db.key,
    db.name,
    db.description,
    db.object_type_id,
    db.object_type_name,
    db.data_type,
    db.is_required,
    db.created_at,
    db.updated_at,
    CASE 
      WHEN p_include_mappings THEN COALESCE(dsm.source_mappings, '[]'::jsonb)
      ELSE '[]'::jsonb
    END as source_mappings
  FROM datapoint_base db
  LEFT JOIN datapoint_source_mappings dsm ON db.id = dsm.datapoint_id;
END;
$$ LANGUAGE plpgsql;
```

#### export_objects_data()

```sql
CREATE OR REPLACE FUNCTION export_objects_data(
  p_org_id uuid,
  p_filters jsonb,
  p_options jsonb
)
RETURNS TABLE(
  export_data jsonb
) AS $$
DECLARE
  v_object_types text[];
  v_date_start timestamp with time zone;
  v_date_end timestamp with time zone;
  v_include_metadata boolean;
BEGIN
  -- Parse filters
  v_object_types := COALESCE(
    (SELECT ARRAY(SELECT jsonb_array_elements_text(p_filters->'object_types'))),
    ARRAY[]::text[]
  );
  
  v_date_start := (p_filters->'date_range'->>'start')::timestamp with time zone;
  v_date_end := (p_filters->'date_range'->>'end')::timestamp with time zone;
  v_include_metadata := COALESCE((p_options->>'include_metadata')::boolean, false);

  RETURN QUERY
  WITH export_objects AS (
    SELECT 
      o.id,
      ot.name as object_type,
      o.original_object_value_name,
      o.created_at,
      o.updated_at,
      jsonb_object_agg(dp.key, v.value) as values,
      CASE 
        WHEN v_include_metadata THEN
          jsonb_build_object(
            'org_id', o.org_id,
            'activity_request_id', o.activity_request_id,
            'request_id', o.request_id,
            'confidence_scores', jsonb_object_agg(dp.key, v.confidence_score)
          )
        ELSE NULL
      END as metadata
    FROM public.object o
    JOIN public.object_type ot ON o.object_type_id = ot.id
    LEFT JOIN public.value v ON o.id = v.object_id AND v.deleted_at IS NULL
    LEFT JOIN public.datapoint dp ON v.datapoint_id = dp.id
    WHERE o.deleted_at IS NULL
    AND o.org_id = p_org_id
    AND (array_length(v_object_types, 1) IS NULL OR ot.name = ANY(v_object_types))
    AND (v_date_start IS NULL OR o.created_at >= v_date_start)
    AND (v_date_end IS NULL OR o.created_at <= v_date_end)
    GROUP BY o.id, ot.name, o.original_object_value_name, o.created_at, o.updated_at, o.org_id, o.activity_request_id, o.request_id
  )
  SELECT jsonb_agg(export_objects) as export_data FROM export_objects;
END;
$$ LANGUAGE plpgsql;
```

#### export_values_data()

```sql
CREATE OR REPLACE FUNCTION export_values_data(
  p_org_id uuid,
  p_filters jsonb,
  p_options jsonb
)
RETURNS TABLE(
  export_data jsonb
) AS $$
DECLARE
  v_datapoints text[];
  v_sources uuid[];
  v_confidence_min numeric;
BEGIN
  -- Parse filters
  v_datapoints := COALESCE(
    (SELECT ARRAY(SELECT jsonb_array_elements_text(p_filters->'datapoints'))),
    ARRAY[]::text[]
  );
  
  v_sources := COALESCE(
    (SELECT ARRAY(SELECT jsonb_array_elements_text(p_filters->'sources')::uuid)),
    ARRAY[]::uuid[]
  );
  
  v_confidence_min := (p_filters->>'confidence_min')::numeric;

  RETURN QUERY
  SELECT jsonb_agg(
    jsonb_build_object(
      'id', v.id,
      'object_id', v.object_id,
      'object_name', o.original_object_value_name,
      'object_type', ot.name,
      'datapoint_key', dp.key,
      'datapoint_name', dp.name,
      'value', v.value,
      'confidence_score', v.confidence_score,
      'source_name', s.name,
      'created_at', v.created_at,
      'updated_at', v.updated_at
    )
  ) as export_data
  FROM public.value v
  JOIN public.object o ON v.object_id = o.id
  JOIN public.object_type ot ON o.object_type_id = ot.id
  JOIN public.datapoint dp ON v.datapoint_id = dp.id
  LEFT JOIN public.source s ON v.source_id = s.id
  WHERE v.deleted_at IS NULL
  AND o.org_id = p_org_id
  AND (array_length(v_datapoints, 1) IS NULL OR dp.key = ANY(v_datapoints))
  AND (array_length(v_sources, 1) IS NULL OR v.source_id = ANY(v_sources))
  AND (v_confidence_min IS NULL OR v.confidence_score >= v_confidence_min);
END;
$$ LANGUAGE plpgsql;
```

#### export_combined_data()

```sql
CREATE OR REPLACE FUNCTION export_combined_data(
  p_org_id uuid,
  p_filters jsonb,
  p_options jsonb
)
RETURNS TABLE(
  export_data jsonb
) AS $$
BEGIN
  RETURN QUERY
  WITH combined_data AS (
    SELECT 
      o.id as object_id,
      o.original_object_value_name,
      ot.name as object_type,
      o.created_at as object_created_at,
      jsonb_object_agg(
        dp.key, 
        jsonb_build_object(
          'value', v.value,
          'confidence_score', v.confidence_score,
          'source_name', s.name,
          'created_at', v.created_at
        )
      ) as values_data
    FROM public.object o
    JOIN public.object_type ot ON o.object_type_id = ot.id
    LEFT JOIN public.value v ON o.id = v.object_id AND v.deleted_at IS NULL
    LEFT JOIN public.datapoint dp ON v.datapoint_id = dp.id
    LEFT JOIN public.source s ON v.source_id = s.id
    WHERE o.deleted_at IS NULL
    AND o.org_id = p_org_id
    GROUP BY o.id, o.original_object_value_name, ot.name, o.created_at
  )
  SELECT jsonb_agg(combined_data) as export_data FROM combined_data;
END;
$$ LANGUAGE plpgsql;
```

### get_sources_with_stats()

```sql
CREATE OR REPLACE FUNCTION get_sources_with_stats(p_org_id uuid)
RETURNS TABLE(
  id uuid,
  name text,
  description text,
  source_type text,
  delivery_type text,
  max_concurrent_runs integer,
  timeout integer,
  average_run_duration integer,
  created_at timestamp with time zone,
  updated_at timestamp with time zone,
  owner_org_id uuid,
  owner_org_name text,
  last_used_at timestamp with time zone,
  total_runs bigint,
  successful_runs bigint,
  failed_runs bigint,
  activities_count bigint,
  workflows_count bigint
) AS $$
BEGIN
  RETURN QUERY
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
      s.average_run_duration
    FROM public.source s
    LEFT JOIN public.org o ON s.owner_org_id = o.id
    WHERE s.deleted_at IS NULL
    AND (p_org_id IS NULL OR s.owner_org_id = p_org_id)
),
source_stats AS (
    SELECT 
        rq.source_id,
        COUNT(*) as total_runs,
        COUNT(CASE WHEN rq.status = 'completed' THEN 1 END) as successful_runs,
        COUNT(CASE WHEN rq.status = 'failed' THEN 1 END) as failed_runs,
      MAX(rq.completed_at) as last_used_at
    FROM public.run_queue rq
    WHERE rq.deleted_at IS NULL
    GROUP BY rq.source_id
),
source_activities AS (
    SELECT 
        rq.source_id,
      COUNT(DISTINCT ar.activity_id) as activities_count
    FROM public.run_queue rq
    JOIN public.activity_request ar ON rq.activity_request_id = ar.id
    WHERE rq.deleted_at IS NULL AND ar.deleted_at IS NULL
    GROUP BY rq.source_id
),
source_workflows AS (
    SELECT 
        rq.source_id,
      COUNT(DISTINCT ar.workflow_request_id) as workflows_count
    FROM public.run_queue rq
    JOIN public.activity_request ar ON rq.activity_request_id = ar.id
    WHERE rq.deleted_at IS NULL AND ar.deleted_at IS NULL
    AND ar.workflow_request_id IS NOT NULL
    GROUP BY rq.source_id
)
SELECT 
    sb.id,
    sb.name,
    sb.description,
    sb.source_type,
    sb.delivery_type,
    sb.max_concurrent_runs,
    sb.timeout,
    COALESCE(sb.average_run_duration, 0),
    sb.created_at,
    sb.updated_at,
    sb.owner_org_id,
    sb.owner_org_name,
    ss.last_used_at,
    COALESCE(ss.total_runs, 0),
    COALESCE(ss.successful_runs, 0),
    COALESCE(ss.failed_runs, 0),
    COALESCE(sa.activities_count, 0),
    COALESCE(sw.workflows_count, 0)
FROM source_base sb
LEFT JOIN source_stats ss ON sb.id = ss.source_id
LEFT JOIN source_activities sa ON sb.id = sa.source_id
LEFT JOIN source_workflows sw ON sb.id = sw.source_id
ORDER BY sb.created_at DESC;
END;
$$ LANGUAGE plpgsql;
```

## Implementation Strategy

### Phase 1: Core Data API Functions

1. **Create Source API Function**
   - Implement `/api/sources` endpoint
   - Add database query function `get_sources_with_stats()`
   - Test data transformation and authentication

2. **Create Activities API Function**
   - Implement `/api/activities` endpoint
   - Add database query function `get_activities_with_stats()`
   - Test activity data transformation

3. **Create Workflows API Function**
   - Implement `/api/workflows` endpoint
   - Add database query function `get_workflows_with_stats()`
   - Test workflow data transformation

### Phase 2: AI Generation Functions

1. **Create AI Content Generator**
   - Implement `/api/ai/generate` endpoint
   - Add OpenAI integration with dynamic prompts
   - Test various generation scenarios

2. **Create AI Schema Generator**
   - Implement `/api/ai/generate-schema` endpoint
   - Add schema generation logic
   - Test schema validation

3. **Create AI Factor Generator**
   - Implement `/api/ai/generate-factors` endpoint
   - Add factor generation logic
   - Test factor validation

### Phase 3: Business Logic Functions

1. **Create Activity Execution Function**
   - Implement `/api/activities/execute` endpoint
   - Add validation and workflow management
   - Test execution scenarios

2. **Create Workflow Execution Function**
   - Implement `/api/workflows/execute` endpoint
   - Add phase management logic
   - Test multi-phase workflows

### Phase 4: Utility Functions

1. **Create Health Check Function**
   - Implement `/api/health` endpoint
   - Add service monitoring
   - Test health reporting

2. **Create Data Validation Function**
   - Implement `/api/utils/validate` endpoint
   - Add schema validation
   - Test validation scenarios

## Error Handling and Validation

### Common Error Response Format

```typescript
interface ErrorResponse {
  error: string;
  code?: string;
  details?: any;
  timestamp: string;
  request_id?: string;
}
```

### Validation Patterns

```typescript
// Request validation
function validateRequest<T>(request: any, schema: any): T {
  // Implementation for request validation
}

// Response transformation
function transformResponse<T>(data: any, transformer: (item: any) => T): T[] {
  // Implementation for response transformation
}
```

## Security Considerations

### Authentication
- All endpoints require JWT authentication
- User context includes organization ID for data filtering
- Rate limiting on AI generation endpoints

### Authorization
- Organization-based data access control
- Function-level permissions
- Audit logging for sensitive operations

### Data Protection
- Input validation and sanitization
- Output filtering for sensitive data
- Error message sanitization

## Performance Optimization

### Caching Strategy
- Cache frequently accessed data
- Implement cache invalidation
- Use Redis for session storage

### Database Optimization
- Optimize query performance
- Add appropriate indexes
- Use connection pooling

### Edge Function Optimization
- Minimize cold start times
- Optimize bundle size
- Implement request batching

## Monitoring and Logging

### Logging Strategy
- Structured logging with correlation IDs
- Performance metrics collection
- Error tracking and alerting

### Monitoring
- Health check endpoints
- Performance dashboards
- Usage analytics

## Testing Strategy

### Unit Testing
- Test individual functions
- Mock external dependencies
- Validate data transformations

### Integration Testing
- Test end-to-end workflows
- Validate API contracts
- Test error scenarios

### Performance Testing
- Load testing for high-traffic endpoints
- Stress testing for AI generation
- Database performance testing

## Deployment Strategy

### Edge Function Deployment
- Use Supabase CLI for deployment
- Implement blue-green deployment
- Monitor deployment health

### Environment Management
- Separate environments for development/staging/production
- Environment-specific configuration
- Secret management

## Testing Strategy

### 1. Unit Testing for Edge Functions

#### 1.1. Test Framework Setup

```typescript
// Test utilities: /supabase/functions/_shared/test-utils.ts
import { assertEquals, assertExists } from "https://deno.land/std@0.168.0/testing/asserts.ts"
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

export class TestHelper {
  private supabase: any
  
  constructor() {
    this.supabase = createClient(
      Deno.env.get('SUPABASE_URL') ?? '',
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY') ?? ''
    )
  }

  async createTestUser(): Promise<{user: any, token: string}> {
    // Create test user and return JWT token
  }

  async createTestOrg(): Promise<any> {
    // Create test organization
  }

  async cleanupTestData(): Promise<void> {
    // Clean up test data
  }

  async makeAuthenticatedRequest(endpoint: string, options: RequestInit, token: string): Promise<Response> {
    return fetch(`https://your-project.supabase.co/functions/v1${endpoint}`, {
      ...options,
      headers: {
        ...options.headers,
        'Authorization': `Bearer ${token}`
      }
    })
  }
}
```

#### 1.2. Data API Tests

##### Sources API Tests (`/api/sources`)

```typescript
// Test file: /supabase/functions/api-sources/test.ts
import { TestHelper } from '../_shared/test-utils.ts'

Deno.test("Sources API - Get sources with authentication", async () => {
  const helper = new TestHelper()
  const { user, token } = await helper.createTestUser()
  
  try {
    const response = await helper.makeAuthenticatedRequest(
      '/api/sources',
      { method: 'GET' },
      token
    )
    
    assertEquals(response.status, 200)
    const data = await response.json()
    assertExists(data.sources)
    assertExists(data.metadata)
    assertEquals(typeof data.sources, 'object')
    assertEquals(data.metadata.total_count, expect.any(Number))
  } finally {
    await helper.cleanupTestData()
  }
})

Deno.test("Sources API - Unauthorized access", async () => {
  const response = await fetch('https://your-project.supabase.co/functions/v1/api/sources')
  assertEquals(response.status, 401)
})

Deno.test("Sources API - Data transformation accuracy", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    // Create test source data
    await helper.createTestSource()
    
    const response = await helper.makeAuthenticatedRequest(
      '/api/sources',
      { method: 'GET' },
      token
    )
    
    const data = await response.json()
    const source = data.sources[0]
    
    // Test data transformation
    assertEquals(typeof source.id, 'string')
    assertEquals(typeof source.name, 'string')
    assertEquals(typeof source.source_type, 'string')
    assertEquals(typeof source.total_runs, 'number')
    assertEquals(typeof source.success_rate, 'number')
    assertEquals(source.success_rate >= 0 && source.success_rate <= 1, true)
  } finally {
    await helper.cleanupTestData()
  }
})
```

##### Activities API Tests (`/api/activities`)

```typescript
// Test file: /supabase/functions/api-activities/test.ts
Deno.test("Activities API - Get activities with filters", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    const response = await helper.makeAuthenticatedRequest(
      '/api/activities',
      { method: 'GET' },
      token
    )
    
    assertEquals(response.status, 200)
    const data = await response.json()
    
    // Test response structure
    assertExists(data.activities)
    assertExists(data.metadata)
    
    // Test activity data structure
    if (data.activities.length > 0) {
      const activity = data.activities[0]
      assertEquals(typeof activity.id, 'string')
      assertEquals(typeof activity.type, 'string')
      assertEquals(['enrich', 'extract', 'source', 'training', 'fine_tuning', 'export', 'predict'].includes(activity.type), true)
      assertEquals(typeof activity.success_rate, 'number')
    }
  } finally {
    await helper.cleanupTestData()
  }
})
```

##### Data Objects API Tests (`/api/data/objects`)

```typescript
// Test file: /supabase/functions/api-data-objects/test.ts
Deno.test("Data Objects API - Get objects with values", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    // Create test objects and values
    await helper.createTestObjectWithValues()
    
    const response = await helper.makeAuthenticatedRequest(
      '/api/data/objects',
      { method: 'GET' },
      token
    )
    
    assertEquals(response.status, 200)
    const data = await response.json()
    
    assertExists(data.objects)
    assertExists(data.metadata)
    
    if (data.objects.length > 0) {
      const object = data.objects[0]
      assertEquals(typeof object.id, 'string')
      assertEquals(typeof object.object_type, 'string')
      assertExists(object.values)
      assertExists(object.source_mappings)
    }
  } finally {
    await helper.cleanupTestData()
  }
})

Deno.test("Data Objects API - CSV export", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    await helper.createTestObjectWithValues()
    
    const response = await helper.makeAuthenticatedRequest(
      '/api/data/objects?format=csv',
      { method: 'GET' },
      token
    )
    
    assertEquals(response.status, 200)
    assertEquals(response.headers.get('Content-Type'), 'text/csv')
    assertEquals(response.headers.get('Content-Disposition')?.includes('.csv'), true)
    
    const csvData = await response.text()
    assertExists(csvData)
    assertEquals(csvData.includes('id,name,object_type'), true)
  } finally {
    await helper.cleanupTestData()
  }
})

Deno.test("Data Objects API - Pagination", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    // Create multiple test objects
    await helper.createMultipleTestObjects(25)
    
    // Test first page
    const response1 = await helper.makeAuthenticatedRequest(
      '/api/data/objects?limit=10&offset=0',
      { method: 'GET' },
      token
    )
    
    const data1 = await response1.json()
    assertEquals(data1.objects.length, 10)
    assertEquals(data1.metadata.page, 1)
    
    // Test second page
    const response2 = await helper.makeAuthenticatedRequest(
      '/api/data/objects?limit=10&offset=10',
      { method: 'GET' },
      token
    )
    
    const data2 = await response2.json()
    assertEquals(data2.objects.length, 10)
    assertEquals(data2.metadata.page, 2)
    
    // Verify different objects on different pages
    assertEquals(data1.objects[0].id !== data2.objects[0].id, true)
  } finally {
    await helper.cleanupTestData()
  }
})
```

##### Data Export API Tests (`/api/data/export`)

```typescript
// Test file: /supabase/functions/api-data-export/test.ts
Deno.test("Data Export API - Create export job", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    await helper.createTestDataForExport()
    
    const exportRequest = {
      export_type: 'objects',
      filters: {
        object_types: ['company'],
        date_range: {
          start: '2025-01-01T00:00:00Z',
          end: '2025-01-31T23:59:59Z'
        }
      },
      format: 'json',
      options: {
        include_metadata: true
      }
    }
    
    const response = await helper.makeAuthenticatedRequest(
      '/api/data/export',
      {
        method: 'POST',
        body: JSON.stringify(exportRequest),
        headers: { 'Content-Type': 'application/json' }
      },
      token
    )
    
    assertEquals(response.status, 202)
    const data = await response.json()
    
    assertExists(data.export_id)
    assertEquals(data.status, 'processing')
    assertExists(data.expires_at)
    
    // Wait for processing and check status
    await new Promise(resolve => setTimeout(resolve, 2000))
    
    const statusResponse = await helper.makeAuthenticatedRequest(
      `/api/data/export/${data.export_id}/status`,
      { method: 'GET' },
      token
    )
    
    const statusData = await statusResponse.json()
    assertEquals(['processing', 'completed', 'failed'].includes(statusData.status), true)
  } finally {
    await helper.cleanupTestData()
  }
})

Deno.test("Data Export API - Invalid request", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    const invalidRequest = {
      export_type: 'invalid_type',
      filters: {},
      format: 'invalid_format'
    }
    
    const response = await helper.makeAuthenticatedRequest(
      '/api/data/export',
      {
        method: 'POST',
        body: JSON.stringify(invalidRequest),
        headers: { 'Content-Type': 'application/json' }
      },
      token
    )
    
    assertEquals(response.status, 400)
    const error = await response.json()
    assertExists(error.error)
  } finally {
    await helper.cleanupTestData()
  }
})
```

#### 1.3. AI Generation API Tests

```typescript
// Test file: /supabase/functions/api-ai-generate/test.ts
Deno.test("AI Generate API - Source description generation", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    const request = {
      scenario: 'source_description',
      context: {
        source_type: 'API',
        datapoints: ['company_info', 'funding'],
        object_type: 'company'
      },
      output_schema: {
        type: 'text',
        max_length: 200
      },
      options: {
        model: 'gpt-3.5-turbo',
        temperature: 0.7
      }
    }
    
    const response = await helper.makeAuthenticatedRequest(
      '/api/ai/generate',
      {
        method: 'POST',
        body: JSON.stringify(request),
        headers: { 'Content-Type': 'application/json' }
      },
      token
    )
    
    assertEquals(response.status, 200)
    const data = await response.json()
    
    assertExists(data.content)
    assertEquals(typeof data.content, 'string')
    assertExists(data.metadata)
    assertEquals(data.metadata.model_used, 'gpt-3.5-turbo')
    assertExists(data.metadata.tokens_used)
    assertExists(data.suggestions)
    assertEquals(Array.isArray(data.suggestions), true)
  } finally {
    await helper.cleanupTestData()
  }
})

Deno.test("AI Generate API - Rate limiting", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    const request = {
      scenario: 'source_description',
      context: { source_type: 'API', object_type: 'company' },
      output_schema: { type: 'text' }
    }
    
    // Make multiple rapid requests
    const promises = Array(10).fill(null).map(() => 
      helper.makeAuthenticatedRequest(
        '/api/ai/generate',
        {
          method: 'POST',
          body: JSON.stringify(request),
          headers: { 'Content-Type': 'application/json' }
        },
        token
      )
    )
    
    const responses = await Promise.all(promises)
    
    // At least one should be rate limited
    const rateLimited = responses.some(r => r.status === 429)
    assertEquals(rateLimited, true)
  } finally {
    await helper.cleanupTestData()
  }
})

Deno.test("AI Generate API - Invalid scenario", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    const request = {
      scenario: 'invalid_scenario',
      context: {},
      output_schema: { type: 'text' }
    }
    
    const response = await helper.makeAuthenticatedRequest(
      '/api/ai/generate',
      {
        method: 'POST',
        body: JSON.stringify(request),
        headers: { 'Content-Type': 'application/json' }
      },
      token
    )
    
    assertEquals(response.status, 400)
    const error = await response.json()
    assertEquals(error.error, 'Invalid scenario')
  } finally {
    await helper.cleanupTestData()
  }
})
```

### 2. Integration Testing

#### 2.1. End-to-End Workflow Tests

```typescript
// Test file: /tests/integration/workflow-tests.ts
Deno.test("Complete Data Pipeline Integration", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    // 1. Create activity
    const activityResponse = await helper.createTestActivity()
    const activityId = activityResponse.id
    
    // 2. Execute activity
    const executeResponse = await helper.makeAuthenticatedRequest(
      '/api/activities/execute',
      {
        method: 'POST',
        body: JSON.stringify({
          activity_id: activityId,
          objects: [{ data: { company: 'test.com' } }]
        }),
        headers: { 'Content-Type': 'application/json' }
      },
      token
    )
    
    assertEquals(executeResponse.status, 200)
    
    // 3. Wait for processing
    await new Promise(resolve => setTimeout(resolve, 5000))
    
    // 4. Check that objects were created
    const objectsResponse = await helper.makeAuthenticatedRequest(
      '/api/data/objects',
      { method: 'GET' },
      token
    )
    
    const objectsData = await objectsResponse.json()
    assertEquals(objectsData.objects.length > 0, true)
    
    // 5. Check that values were stored
    const valuesResponse = await helper.makeAuthenticatedRequest(
      '/api/data/values',
      { method: 'GET' },
      token
    )
    
    const valuesData = await valuesResponse.json()
    assertEquals(valuesData.values.length > 0, true)
    
    // 6. Test data export
    const exportResponse = await helper.makeAuthenticatedRequest(
      '/api/data/export',
      {
        method: 'POST',
        body: JSON.stringify({
          export_type: 'combined',
          format: 'json',
          filters: {}
        }),
        headers: { 'Content-Type': 'application/json' }
      },
      token
    )
    
    assertEquals(exportResponse.status, 202)
  } finally {
    await helper.cleanupTestData()
  }
})
```

### 3. Performance Testing

#### 3.1. Load Testing

```typescript
// Test file: /tests/performance/load-tests.ts
Deno.test("API Load Testing", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    const concurrentRequests = 50
    const requestPromises = []
    
    for (let i = 0; i < concurrentRequests; i++) {
      requestPromises.push(
        helper.makeAuthenticatedRequest(
          '/api/sources',
          { method: 'GET' },
          token
        )
      )
    }
    
    const startTime = Date.now()
    const responses = await Promise.all(requestPromises)
    const endTime = Date.now()
    
    const responseTime = endTime - startTime
    const successCount = responses.filter(r => r.status === 200).length
    
    assertEquals(successCount, concurrentRequests)
    assertEquals(responseTime < 10000, true) // Under 10 seconds
    
    // Check average response time
    const avgResponseTime = responseTime / concurrentRequests
    assertEquals(avgResponseTime < 2000, true) // Under 2 seconds average
  } finally {
    await helper.cleanupTestData()
  }
})
```

### 4. Security Testing

#### 4.1. Authentication and Authorization Tests

```typescript
// Test file: /tests/security/auth-tests.ts
Deno.test("Cross-Organization Data Isolation", async () => {
  const helper = new TestHelper()
  
  // Create two test organizations
  const org1 = await helper.createTestOrg()
  const org2 = await helper.createTestOrg()
  
  const { user: user1, token: token1 } = await helper.createTestUser(org1.id)
  const { user: user2, token: token2 } = await helper.createTestUser(org2.id)
  
  try {
    // Create data for org1
    await helper.createTestObject(org1.id)
    
    // User1 should see org1 data
    const response1 = await helper.makeAuthenticatedRequest(
      '/api/data/objects',
      { method: 'GET' },
      token1
    )
    
    const data1 = await response1.json()
    assertEquals(data1.objects.length > 0, true)
    
    // User2 should not see org1 data
    const response2 = await helper.makeAuthenticatedRequest(
      '/api/data/objects',
      { method: 'GET' },
      token2
    )
    
    const data2 = await response2.json()
    assertEquals(data2.objects.length, 0)
  } finally {
    await helper.cleanupTestData()
  }
})

Deno.test("SQL Injection Prevention", async () => {
  const helper = new TestHelper()
  const { token } = await helper.createTestUser()
  
  try {
    const maliciousQuery = "'; DROP TABLE objects; --"
    
    const response = await helper.makeAuthenticatedRequest(
      `/api/data/objects?object_type=${encodeURIComponent(maliciousQuery)}`,
      { method: 'GET' },
      token
    )
    
    // Should not return error, but should handle safely
    assertEquals(response.status, 200)
    
    const data = await response.json()
    assertExists(data.objects)
    assertEquals(Array.isArray(data.objects), true)
  } finally {
    await helper.cleanupTestData()
  }
})
```

### 5. Test Data Management

#### 5.1. Test Data Factory

```typescript
// Test data factory: /supabase/functions/_shared/test-data-factory.ts
export class TestDataFactory {
  private supabase: any
  
  constructor(supabase: any) {
    this.supabase = supabase
  }
  
  async createTestOrg(): Promise<any> {
    const { data, error } = await this.supabase
      .from('org')
      .insert({
        name: `Test Org ${Date.now()}`,
        setup: { test: true }
      })
      .select()
      .single()
    
    if (error) throw error
    return data
  }
  
  async createTestSource(orgId: string): Promise<any> {
    const { data, error } = await this.supabase
      .from('source')
      .insert({
        name: `Test Source ${Date.now()}`,
        source_type: 'API',
        delivery_type: 'Endpoint',
        owner_org_id: orgId,
        setup: { test: true }
      })
      .select()
      .single()
    
    if (error) throw error
    return data
  }
  
  async createTestObjectWithValues(orgId: string): Promise<any> {
    // Create object type
    const { data: objectType } = await this.supabase
      .from('object_type')
      .insert({ name: 'test_company' })
      .select()
      .single()
    
    // Create datapoint
    const { data: datapoint } = await this.supabase
      .from('datapoint')
      .insert({
        key: 'industry',
        name: 'Industry',
        object_type_id: objectType.id,
        data_type: 'text'
      })
      .select()
      .single()
    
    // Create object
    const { data: object } = await this.supabase
      .from('object')
      .insert({
        object_type_id: objectType.id,
        org_id: orgId,
        original_object_value_name: 'Test Company'
      })
      .select()
      .single()
    
    // Create value
    const { data: value } = await this.supabase
      .from('value')
      .insert({
        object_id: object.id,
        datapoint_id: datapoint.id,
        value: 'Technology',
        confidence_score: 0.95
      })
      .select()
      .single()
    
    return { object, value, datapoint, objectType }
  }
  
  async cleanupTestData(): Promise<void> {
    // Clean up in reverse dependency order
    await this.supabase.from('value').delete().like('value', '%Test%')
    await this.supabase.from('object').delete().like('original_object_value_name', '%Test%')
    await this.supabase.from('datapoint').delete().like('name', '%Test%')
    await this.supabase.from('object_type').delete().like('name', '%test%')
    await this.supabase.from('source').delete().like('name', '%Test%')
    await this.supabase.from('org').delete().like('name', '%Test%')
  }
}
```

### 6. Test Automation

#### 6.1. CI/CD Integration

```yaml
# .github/workflows/edge-functions-tests.yml
name: Edge Functions Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Deno
      uses: denoland/setup-deno@v1
      with:
        deno-version: v1.x
    
    - name: Setup Supabase CLI
      run: |
        npm install -g supabase
        supabase start
    
    - name: Run Unit Tests
      run: |
        cd supabase/functions
        deno test --allow-all --parallel
    
    - name: Run Integration Tests
      run: |
        cd tests/integration
        deno test --allow-all
    
    - name: Run Performance Tests
      run: |
        cd tests/performance
        deno test --allow-all
    
    - name: Run Security Tests
      run: |
        cd tests/security
        deno test --allow-all
    
    - name: Generate Test Coverage
      run: |
        deno test --coverage=coverage --allow-all
        deno coverage coverage --html
    
    - name: Upload Coverage
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage/lcov.info
```

## Success Criteria

1. **Functionality**
   - All API endpoints work correctly
   - Data transformation is accurate
   - AI generation produces quality content
   - 100% test coverage for critical paths

2. **Performance**
   - Response times under 2 seconds
   - AI generation under 10 seconds
   - 99.9% uptime
   - Load testing passes with 50+ concurrent users

3. **Security**
   - Proper authentication and authorization
   - No data leaks or security vulnerabilities
   - Audit trail for all operations
   - All security tests pass

4. **Scalability**
   - Handle increased load
   - Efficient resource utilization
   - Proper error handling under stress
   - Performance tests pass consistently

5. **Test Coverage**
   - Unit tests for all edge functions
   - Integration tests for complete workflows
   - Performance tests for load scenarios
   - Security tests for vulnerabilities
   - 90%+ code coverage
