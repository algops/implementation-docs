# Accordion Designs Implementation Plan

## Component Usage Validation
**✅ COMPONENT SOURCE CHECK:**
- [ ] Uses only components from `components/` folder
- [ ] Uses only components from `components/ui-elements/` subfolder
- [ ] **NO direct usage of `components/ui/` folder components**

## Overview
Enhance existing accordion components to better display JSON data from your data files. You already have a solid accordion system with `BaseAccordion` and specialized accordions like `SourceRequestAccordion`, `WorkflowPhaseAccordion`, etc.

## Current Accordion Components
- **BaseAccordion**: Universal accordion with grid data and empty states
- **SourceRequestAccordion**: Shows inference and status check forms
- **WorkflowPhaseAccordion**: Displays activities in workflow phases
- **ActivitySourceAccordion**: Shows activity-source relationships
- **SourceUsecasesAccordion**: Displays source use cases
- **SourceGuardrailAccordion**: Shows source guardrails
- **JsonAccordion**: Displays JSON data

## Phase 1: Pre-selected Data for Accordion Grids

### 1.1 SourceRequestAccordion Pre-selected Data
**Data Source**: Individual source JSON (e.g., `ai-company-researcher.json`)
**Pre-selected Grid Data**:
1. **Configuration Settings** - `configuration.max_concurrent_runs`, `configuration.timeout`, `configuration.max_retries`
2. **Template Status** - `templates.run_request_template` (formatted), `templates.status_request_template`, `templates.delivery_request_template`
3. **Performance Metrics** - `total_runs`, `successful_runs`, `failed_runs`, `success_rate`
4. **Source Type Info** - `source_type`, `delivery_type`, `status`
5. **Usage Statistics** - `average_run_duration`, `last_used_at`, `created_at`
6. **Owner Information** - `owner_org_name`, `owner_org_id`
7. **Activity Counts** - `activities_count`, `workflows_count`
8. **Request Templates** - `setup.run_request_template.request`, `setup.run_request_template.response`
9. **Status Translations** - `setup.run_request_template.response.status_translations`
10. **Webhook Configuration** - `setup.run_request_template.request.webhook_url`

### 1.2 WorkflowPhaseAccordion Pre-selected Data
**Data Source**: Individual workflow JSON (e.g., `ecommerce-product-discovery-recommendation.json`)
**Pre-selected Grid Data**:
1. **Phase Configuration** - `setup.data` (all phases with their configurations)
2. **Phase Progress** - Current phase status, completed phases, pending phases
3. **Activity Dependencies** - `setup.data.{phase}.destination.activity.id` relationships
4. **Phase Limits** - `setup.data.{phase}.setup.config.max_objects` for each phase
5. **Execution Statistics** - `total_executions`, `successful_executions`, `failed_executions`
6. **Workflow Metrics** - `success_rate`, `average_duration`, `activities_count`
7. **Phase Count** - `phases_count`, `activities_count`
8. **Runtime Stats** - `runtime_stats.total_executions`, `runtime_stats.success_rate`
9. **Metadata** - `metadata.owner_org_name`, `metadata.created_at`, `metadata.updated_at`
10. **Workflow Status** - `status`, `verification_status`, `last_used_at`

### 1.3 ActivitySourceAccordion Pre-selected Data
**Data Source**: Individual activity JSON (e.g., `ai-company-train-model.json`)
**Pre-selected Grid Data**:
1. **Source Relationships** - `setup.data` (source and target configurations)
2. **Activity Performance** - `runtime_stats.total_requests`, `runtime_stats.success_rate`
3. **Source Configuration** - `setup.data.ai_company_source.id`, `setup.data.ai_company_source.type`
4. **Target Configuration** - `setup.data.company_analysis_target.id`, `setup.data.company_analysis_target.type`
5. **Activity Metrics** - `runtime_stats.average_duration`, `runtime_stats.last_used_at`
6. **Workflow Connections** - `metadata.workflows` (associated workflows)
7. **Recent Activity** - `metadata.recent_requests` (recent request history)
8. **Owner Information** - `metadata.owner_org_name`, `metadata.owner_org_id`
9. **Activity Status** - `status`, `verification_status`, `type`
10. **Source Types** - Associated source types from setup data

### 1.4 SourceUsecasesAccordion Pre-selected Data
**Data Source**: `factors.json` (filtered by `type: "use-case"`)
**Pre-selected Grid Data**:
1. **Use Case Definitions** - `name`, `description`, `factor_type`
2. **Object Type Mapping** - `object_type_id` (linked to object-types.json)
3. **Datapoint Mapping** - `datapoint_id` (linked to datapoints.json)
4. **Factor Criteria** - `operator`, `value` (validation criteria)
5. **Setup Configuration** - `setup` (factor-specific settings)
6. **Factor Status** - `created_at`, `updated_at`, `deleted_at`
7. **Use Case Count** - Total count of use case factors
8. **Active Use Cases** - Count of non-deleted use cases
9. **Factor Types** - Breakdown by `factor_type` (frequency, boolean, numeric, text)
10. **Object Type Distribution** - Count by `object_type_id`

### 1.5 SourceGuardrailAccordion Pre-selected Data
**Data Source**: `factors.json` (filtered by `type: "guardrail"`)
**Pre-selected Grid Data**:
1. **Guardrail Definitions** - `name`, `description`, `factor_type`
2. **Object Type Mapping** - `object_type_id` (linked to object-types.json)
3. **Datapoint Mapping** - `datapoint_id` (linked to datapoints.json)
4. **Validation Rules** - `operator`, `value` (validation criteria)
5. **Setup Configuration** - `setup` (guardrail-specific settings)
6. **Guardrail Status** - `created_at`, `updated_at`, `deleted_at`
7. **Guardrail Count** - Total count of guardrail factors
8. **Active Guardrails** - Count of non-deleted guardrails
9. **Factor Types** - Breakdown by `factor_type` (frequency, boolean, numeric, text)
10. **Object Type Distribution** - Count by `object_type_id`

## Phase 2: Enhance Existing Accordions for JSON Data

### 2.1 Enhance BaseAccordion
- **Add JSON Data Support**: Display complex JSON structures from your data files
- **Improve Grid Data**: Better display of metrics and statistics
- **Add Loading States**: Handle data loading gracefully
- **Enhance Empty States**: More helpful empty state messages

### 1.2 Enhance Specialized Accordions with Grid Data

#### SourceRequestAccordion
**Data Source**: `sources.json`
**Grid Data Options**:
1. **Source Type** - `source_type` (Dataset, Integration, etc.)
2. **Delivery Type** - `delivery_type` (Endpoint, Webhook, etc.)
3. **Status** - `status` (active, inactive, etc.)
4. **Success Rate** - `success_rate` (as percentage)
5. **Total Runs** - `total_runs`
6. **Successful Runs** - `successful_runs`
7. **Failed Runs** - `failed_runs`
8. **Average Duration** - `average_run_duration` (in seconds)
9. **Max Concurrent Runs** - `max_concurrent_runs`
10. **Timeout** - `timeout` (in seconds)
11. **Activities Count** - `activities_count`
12. **Workflows Count** - `workflows_count`
13. **Last Used** - `last_used_at` (formatted date)
14. **Created Date** - `created_at` (formatted date)
15. **Updated Date** - `updated_at` (formatted date)

#### WorkflowPhaseAccordion
**Data Source**: `workflows.json`
**Grid Data Options**:
1. **Activities Count** - `activities.length` (from activities array)
2. **Total Executions** - `total_executions`
3. **Successful Executions** - `successful_executions`
4. **Failed Executions** - `failed_executions`
5. **Success Rate** - `success_rate` (as percentage)
6. **Average Duration** - `average_duration` (in seconds)
7. **Phases Count** - `phases_count`
8. **Status** - `status` (active, inactive, etc.)
9. **Verification Status** - `verification_status` (true/false)
10. **Last Used** - `last_used_at` (formatted date)
11. **Created Date** - `created_at` (formatted date)
12. **Updated Date** - `updated_at` (formatted date)
13. **Owner Org** - `owner_org_name`

#### ActivitySourceAccordion
**Data Source**: `sources.json` (filtered by activity)
**Grid Data Options**:
1. **Sources Count** - `sources.length`
2. **Active Sources** - `sources.filter(s => s.status === 'active').length`
3. **Average Success Rate** - `sources.reduce((acc, s) => acc + s.success_rate, 0) / sources.length`
4. **Total Runs** - `sources.reduce((acc, s) => acc + s.total_runs, 0)`
5. **Total Successful Runs** - `sources.reduce((acc, s) => acc + s.successful_runs, 0)`
6. **Total Failed Runs** - `sources.reduce((acc, s) => acc + s.failed_runs, 0)`
7. **Average Duration** - `sources.reduce((acc, s) => acc + s.average_run_duration, 0) / sources.length`
8. **Max Concurrent Total** - `sources.reduce((acc, s) => acc + s.max_concurrent_runs, 0)`
9. **Dataset Sources** - `sources.filter(s => s.source_type === 'Dataset').length`
10. **Integration Sources** - `sources.filter(s => s.source_type === 'Integration').length`
11. **Endpoint Sources** - `sources.filter(s => s.delivery_type === 'Endpoint').length`
12. **Webhook Sources** - `sources.filter(s => s.delivery_type === 'Webhook').length`

#### SourceUsecasesAccordion
**Data Source**: `factors.json` (filtered by `type: "use-case"`)
**Grid Data Options**:
1. **Use Cases Count** - `usecases.length`
2. **Active Use Cases** - `usecases.filter(u => u.isActive).length`
3. **Factor Types** - `usecases.map(u => u.factor_type).filter((v, i, a) => a.indexOf(v) === i).length`
4. **Frequency Factors** - `usecases.filter(u => u.factor_type === 'frequency').length`
5. **Reason Factors** - `usecases.filter(u => u.factor_type === 'reason').length`
6. **Similarity Factors** - `usecases.filter(u => u.factor_type === 'similarity').length`
7. **Object Types** - `usecases.map(u => u.object_type_id).filter((v, i, a) => a.indexOf(v) === i).length`
8. **Datapoints** - `usecases.map(u => u.datapoint_id).filter((v, i, a) => a.indexOf(v) === i).length`
9. **With Operators** - `usecases.filter(u => u.operator !== null).length`
10. **With Values** - `usecases.filter(u => u.value !== null).length`
11. **Created This Year** - `usecases.filter(u => new Date(u.created_at).getFullYear() === new Date().getFullYear()).length`
12. **Updated This Year** - `usecases.filter(u => new Date(u.updated_at).getFullYear() === new Date().getFullYear()).length`

#### SourceGuardrailAccordion
**Data Source**: `factors.json` (filtered by `type: "guardrail"`)
**Grid Data Options**:
1. **Guardrails Count** - `guardrails.length`
2. **Active Guardrails** - `guardrails.filter(g => g.isActive).length`
3. **Factor Types** - `guardrails.map(g => g.factor_type).filter((v, i, a) => a.indexOf(v) === i).length`
4. **Boolean Factors** - `guardrails.filter(g => g.factor_type === 'boolean').length`
5. **Regex Factors** - `guardrails.filter(g => g.operator === 'regex').length`
6. **Starts With Factors** - `guardrails.filter(g => g.operator === 'starts_with').length`
7. **Object Types** - `guardrails.map(g => g.object_type_id).filter((v, i, a) => a.indexOf(v) === i).length`
8. **Datapoints** - `guardrails.map(g => g.datapoint_id).filter((v, i, a) => a.indexOf(v) === i).length`
9. **With Operators** - `guardrails.filter(g => g.operator !== null).length`
10. **With Values** - `guardrails.filter(g => g.value !== null).length`
11. **Created This Year** - `guardrails.filter(g => new Date(g.created_at).getFullYear() === new Date().getFullYear()).length`
12. **Updated This Year** - `guardrails.filter(g => new Date(g.updated_at).getFullYear() === new Date().getFullYear()).length`

### 1.3 Data Display Design
- **Configuration Data**: Source templates, workflow phases, factor rules
- **Performance Metrics**: Success rates, run times, error counts
- **Status Information**: Current state, last updates, next scheduled runs
- **Operational Data**: Queue lengths, processing volumes, health indicators

## Phase 2: Specialized Accordion Components

### 2.1 Source Configuration Accordion
- **Source Details**: Name, type, description, owner organization
- **Template Display**: run_request_template, delivery_request_template JSON
- **Performance Metrics**: Average run duration, success rate, max concurrent runs
- **Health Status**: Current status, last successful run, error logs
- **Configuration Options**: Timeout settings, authentication, delivery type

### 2.2 Workflow Phase Accordion
- **Phase Definitions**: Display workflow.setup JSON structure
- **Phase Status**: Current phase, completed phases, next phase
- **Activity Mapping**: Activities in each phase, dependencies
- **Progress Tracking**: Phase completion percentage, estimated time
- **Phase Configuration**: Phase-specific settings and parameters

### 2.3 Factor Accordion (Guardrails & Use Cases)
- **Factor Type**: Guardrail vs Use Case classification
- **Rule Definition**: Factor criteria and validation logic
- **Source Mapping**: Which sources use this factor
- **Threshold Settings**: Guardrail limits and use case criteria
- **Validation Status**: Current validation state, last check

### 2.4 Performance Accordion
- **Run Statistics**: Total runs, success/failure counts, average duration
- **Queue Information**: Current queue length, processing status
- **Error Analysis**: Error types, frequency, resolution status
- **Resource Usage**: Memory, CPU, API call statistics
- **Trend Analysis**: Performance over time, improvement suggestions

## Phase 3: Data Visualization & Interaction

### 3.1 Configuration Display
- **JSON Viewer**: Pretty-printed configuration data
- **Template Editor**: Editable configuration templates
- **Validation**: Real-time configuration validation
- **Version History**: Configuration change tracking

### 3.2 Performance Visualization
- **Metrics Charts**: Success rates, response times, error trends
- **Status Indicators**: Visual health and status representation
- **Progress Bars**: Completion percentages and time estimates
- **Alert System**: Performance threshold alerts

### 3.3 Interactive Features
- **Expandable Sections**: Collapsible detailed information
- **Real-time Updates**: Live data refresh capabilities
- **Action Buttons**: Quick actions (restart, pause, configure)
- **Filter Options**: Data filtering and search capabilities

## Phase 4: Responsive Design & Accessibility

### 4.1 Responsive Layout
- **Mobile-First**: Touch-friendly accordion design
- **Adaptive Content**: Responsive data display
- **Touch Targets**: Adequate size for mobile interaction
- **Orientation Support**: Portrait and landscape layouts

### 4.2 Accessibility Features
- **ARIA Labels**: Screen reader support for data
- **Keyboard Navigation**: Full keyboard accessibility
- **Focus Management**: Clear focus indicators
- **High Contrast**: Support for accessibility themes

## Phase 5: Integration & Advanced Features

### 5.1 Database Integration
- **Real-time Data**: Live database updates
- **Data Binding**: Dynamic content from database
- **Error Handling**: Graceful database error display
- **Loading States**: Skeleton loading for data

### 5.2 Advanced Features
- **Data Export**: Export accordion data to various formats
- **Print Support**: Print-friendly accordion layouts
- **Search & Filter**: Advanced data filtering capabilities
- **Customization**: User-configurable accordion layouts

## File Changes Required

### New Files
- `components/accordions/source-configuration-accordion.tsx`
- `components/accordions/workflow-phase-accordion.tsx`
- `components/accordions/factor-accordion.tsx`
- `components/accordions/performance-accordion.tsx`
- `components/accordions/json-viewer.tsx`
- `components/accordions/metrics-display.tsx`

### Modified Files
- `components/accordions/index.ts` - Export new specialized accordions
- Existing accordion components - Update to use new base components

## Success Criteria

1. **Data Display**: Clear presentation of database configuration and performance data
2. **User Experience**: Intuitive navigation through complex data structures
3. **Performance**: Efficient rendering of large datasets
4. **Accessibility**: Full WCAG 2.1 AA compliance
5. **Responsiveness**: Seamless experience across all device sizes
