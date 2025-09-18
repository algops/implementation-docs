# Card Designs Implementation Plan

## Component Usage Validation
**✅ COMPONENT SOURCE CHECK:**
- [ ] Uses only components from `components/` folder
- [ ] Uses only components from `components/ui-elements/` subfolder
- [ ] **NO direct usage of `components/ui/` folder components**

## Overview
Implement specialized cards that display real-time database information, status, and key metrics for activities, workflows, sources, objects, and factors using the existing card components.

## Phase 1: Database-Focused Card Architecture

### 1.1 Card System Architecture
- **Base Card**: Foundation card component with database data integration
- **Specialized Cards**: Activity, workflow, source, object, and factor cards
- **Data Display**: Real-time status, metrics, and configuration information
- **Database Integration**: Direct connection to database tables and JSON fields

### 1.2 Core Components Structure
- **BaseCard**: Universal card wrapper with database data display capabilities
- **ActivityCard**: Display activity status and performance metrics
- **WorkflowCard**: Show workflow progress and phase information
- **SourceCard**: Display source health and configuration
- **ObjectTypeCard**: Show object coverage and data quality
- **UseCaseCard**: Display use case rules and validation
- **GuardrailCard**: Show guardrail rules and thresholds

### 1.3 Data Display Design
- **Real-time Status**: Current state from activity_request.status, run_queue.status
- **Performance Metrics**: Success rates, response times from run_result and run_queue
- **Configuration Data**: JSON templates from source tables and factor tables
- **Operational Data**: Queue status, health indicators from database metrics

## Phase 2: Specialized Card Components

### 2.1 ActivityCard Component
- **Activity Information**: Name, type, description from activity table
- **Status Display**: Current status from activity_request.status (pending, validation, ACTIVITY_PROCESSED, DONE)
- **Progress Tracking**: 
  - Completion percentage based on runs_in_queue vs completed runs
  - Estimated time based on average_run_duration from source table
- **Performance Metrics**: 
  - Success rate calculated from run_result vs run_queue
  - Average duration from source.average_run_duration
  - Last run time from run_request.created_at
- **Action Buttons**: Start, pause, stop, restart actions that call ETL functions

### 2.2 WorkflowCard Component
- **Workflow Information**: Name, description from workflow table
- **Phase Progress**: 
  - Current phase from activity_request.workflow_phase
  - Total phases calculated from workflow.setup JSON
  - Phase completion status from activity_request.status
- **Activity Status**: 
  - Activities in current phase from workflow.setup.data
  - Completion status from activity_request.status
  - Dependencies from workflow.setup.destination.activity.id
- **Time Estimates**: 
  - Estimated completion based on source.average_run_duration
  - Phase duration from run statistics
- **Workflow Actions**: Pause, resume, cancel workflow using ETL functions

### 2.3 SourceCard Component
- **Source Information**: Name, type, description from source table
- **Health Status**: 
  - Current health based on last successful run vs timeout
  - Queue health from run_queue status distribution
- **Performance Metrics**: 
  - Success rate from run_result vs run_request
  - Average response time from source.average_run_duration
  - Throughput from max_concurrent_runs and current active runs
- **Queue Status**: 
  - Current queue length from run_queue where status = 'waiting'
  - Processing capacity from source.max_concurrent_runs
- **Last Activity**: 
  - Last successful run from run_request where status = 'DONE'
  - Last error from run_queue.error
  - Next scheduled run based on queue position

### 2.4 ObjectTypeCard Component
- **Object Type Info**: Name, description from object_type table
- **Data Coverage**: 
  - Percentage calculated from value table vs object table
  - Coverage by datapoint from datapoint_source mappings
- **Last Update**: 
  - Last data refresh from value.updated_at
  - Update frequency from activity_request patterns
- **Validation Status**: 
  - Data quality from value.confidence scores
  - Validation errors from value.status_code
- **Coverage Actions**: Refresh data, validate objects using ETL functions

### 2.5 UseCaseCard Component
- **Use Case Information**: Name, type, factor_type from factor table
- **Rule Definition**: 
  - Criteria from factor.setup JSON
  - Thresholds from factor.value and factor_type
  - Object type and datapoint from factor.object_type_id and factor.datapoint_id
- **Application Status**: 
  - Where applied from factor_source table
  - Validation results from run_result analysis
- **Performance Impact**: 
  - Success rate from filtered vs total runs
  - Error frequency from use case violations
- **Configuration Actions**: Edit rules, adjust thresholds using factor table updates

### 2.6 GuardrailCard Component
- **Guardrail Information**: Name, type, factor_type from factor table
- **Rule Definition**: 
  - Criteria from factor.setup JSON
  - Thresholds from factor.value and factor_type
  - Object type and datapoint from factor.object_type_id and factor.datapoint_id
- **Application Status**: 
  - Where applied from factor_source table
  - Validation results from run_result analysis
- **Performance Impact**: 
  - Success rate from filtered vs total runs
  - Error frequency from guardrail violations
- **Configuration Actions**: Edit rules, adjust thresholds using factor table updates

## Phase 3: JSON Template Display & Configuration

### 3.1 Template Display in Cards
- **run_request_template**: 
  - Pretty-printed JSON with syntax highlighting
  - Variable placeholders (<<$run_setup>>, <<$max_objects>>) highlighted
  - HTTP method, URL, headers, body structure
- **delivery_request_template**: 
  - Result retrieval configuration
  - External ID handling (<<$run_external_id>>)
- **status_request_template**: 
  - Status checking configuration
  - Polling intervals and error handling

### 3.2 Template Configuration in Cards
- **Variable Resolution**: 
  - Show resolved values from utils.replace_request_placeholders()
  - Display actual HTTP requests with resolved variables
- **Template Validation**: 
  - Validate JSON structure and required fields
  - Check for missing variable placeholders
- **Template Testing**: 
  - Test templates with sample data
  - Preview actual HTTP requests

## Phase 4: Data Visualization & Metrics

### 4.1 Status Indicators
- **Visual Status**: Color-coded status indicators based on database values
- **Progress Bars**: Completion percentage from runs_in_queue calculations
- **Health Indicators**: Traffic light style based on source health metrics
- **Alert Badges**: Warning and error notifications from run_queue.error

### 4.2 Metric Display
- **Numerical Metrics**: Large, prominent metric values from org table
- **Trend Indicators**: Up/down arrows with percentages from historical data
- **Comparison Data**: Current vs previous period from org table trends
- **Target Tracking**: Progress toward usage limits (free_objects)

### 4.3 Interactive Elements
- **Action Buttons**: Primary and secondary actions that call ETL functions
- **Expandable Content**: Detailed information from related database tables
- **Quick Actions**: Context menu with common database operations
- **Status Updates**: Real-time status refresh from database triggers

## Phase 5: Database Integration & Real-time Updates

### 5.1 Real-time Data
- **Database Triggers**: Listen to changes in activity_request, run_queue, run_result
- **Live Updates**: Real-time status and metric updates
- **Performance Monitoring**: Live performance metric updates
- **Queue Monitoring**: Real-time queue status updates

### 5.2 Data Binding
- **Dynamic Content**: Content from database tables and JSON fields
- **Template Resolution**: Real-time template variable resolution
- **Status Calculation**: Dynamic status and progress calculations
- **Metric Aggregation**: Real-time metric calculations and updates

## File Changes Required

### Modified Files
- `components/cards/activity-card.tsx` - Update to display database metrics
- `components/cards/workflow-card.tsx` - Update to show workflow progress
- `components/cards/source-card.tsx` - Update to display source health
- `components/cards/objecttype-card.tsx` - Update to show data coverage
- `components/cards/usecase-card.tsx` - Update to display use case rules
- `components/cards/guardrail-card.tsx` - Update to show guardrail rules
- `components/cards/base-card.tsx` - Enhance with database data capabilities
- `components/cards/index.ts` - Ensure all existing cards are exported

### No New Files Required
- All specialized card components already exist
- Focus on updating existing components with database integration

## Testing Checklist

### Unit Tests
- [ ] Card rendering with database data
- [ ] Card state management for real-time updates
- [ ] Event handling for database operations
- [ ] Data binding and metric calculations

### Integration Tests
- [ ] Complete card workflow with database
- [ ] Card interactions with ETL functions
- [ ] Content rendering from database
- [ ] Error handling for database failures

### User Experience Tests
- [ ] Card interaction flow with real-time data
- [ ] Mobile responsiveness with database metrics
- [ ] Accessibility compliance for data display
- [ ] Performance with many cards and live updates

## Success Criteria

1. **Data Display**: Clear presentation of real-time database status and metrics
2. **JSON Integration**: Effective display and editing of JSON templates
3. **User Experience**: Intuitive understanding of system state at a glance
4. **Performance**: Efficient rendering and real-time updates
5. **Accessibility**: Full WCAG 2.1 AA compliance
6. **Responsiveness**: Seamless experience across all device sizes
7. **Database Integration**: Seamless connection to existing database structure
