# Pages Implementation Plan

## Component Usage Validation
**✅ COMPONENT SOURCE CHECK:**
- [ ] Uses only components from `components/` folder
- [ ] Uses only components from `components/ui-elements/` subfolder
- **NO direct usage of `components/ui/` folder components**

## Overview
Implement comprehensive page layouts for the main entity pages (sources, activities, workflows) and their detail pages, using existing components to display data from JSON files with proper navigation and user experience.

## Phase 1: Sources Pages

### 1.1 Sources List Page
**Data Source**: `sources.json`
**Page Layout Options**:
1. **Page Header** - Title, description, and action buttons
2. **Filters Section** - Filter by source type, delivery type, status
3. **Search Bar** - Search sources by name or description
4. **Stats Cards** - Total sources, active sources, success rate overview
5. **Sources Grid** - Grid of SourceCard components
6. **Pagination** - Page navigation for large datasets
7. **Bulk Actions** - Select multiple sources for batch operations

**Stats Cards Display Options**:
1. **Total Sources** - `sources.length`
2. **Active Sources** - `sources.filter(s => s.status === 'active').length`
3. **Average Success Rate** - `sources.reduce((acc, s) => acc + s.success_rate, 0) / sources.length`
4. **Total Runs** - `sources.reduce((acc, s) => acc + s.total_runs, 0)`
5. **Successful Runs** - `sources.reduce((acc, s) => acc + s.successful_runs, 0)`
6. **Failed Runs** - `sources.reduce((acc, s) => acc + s.failed_runs, 0)`
7. **Average Duration** - `sources.reduce((acc, s) => acc + s.average_run_duration, 0) / sources.length`
8. **Dataset Sources** - `sources.filter(s => s.source_type === 'Dataset').length`
9. **Integration Sources** - `sources.filter(s => s.source_type === 'Integration').length`
10. **Agent Sources** - `sources.filter(s => s.source_type === 'Agent').length`

**Filter Options**:
1. **Source Type** - Dataset, Integration, Agent, ML model, LLM
2. **Delivery Type** - Endpoint, Webhook
3. **Status** - Active, Inactive, Error
4. **Success Rate Range** - High (>90%), Medium (70-90%), Low (<70%)
5. **Last Used** - Today, This week, This month, Older
6. **Owner Organization** - Filter by organization

### 1.2 Source Detail Page
**Data Source**: Individual source JSON (e.g., `ai-company-researcher.json`)
**Page Layout Options**:
1. **Page Header** - Source name, status badge, action buttons
2. **Overview Section** - Key metrics and status information
3. **Configuration Section** - Source configuration details
4. **Templates Section** - Request/response templates
5. **Performance Section** - Charts and metrics
6. **Activities Section** - Related activities
7. **Workflows Section** - Related workflows
8. **History Section** - Recent runs and changes

**Overview Section Display Options**:
1. **Source Name** - `name`
2. **Description** - `description`
3. **Source Type** - `source_type` (with badge)
4. **Delivery Type** - `delivery_type` (with badge)
5. **Status** - `status` (with status indicator)
6. **Success Rate** - `success_rate` (as percentage with trend)
7. **Total Runs** - `total_runs` (with chart)
8. **Average Duration** - `average_run_duration` (in seconds)
9. **Last Used** - `last_used_at` (formatted date)
10. **Created Date** - `created_at` (formatted date)
11. **Updated Date** - `updated_at` (formatted date)
12. **Owner Organization** - `owner_org_name`

**Configuration Section Display Options**:
1. **Max Concurrent Runs** - `configuration.max_concurrent_runs`
2. **Timeout** - `configuration.timeout` (in seconds)
3. **Max Retries** - `configuration.max_retries`
4. **Auth Key** - `configuration.auth_key` (masked)
5. **Request Template** - `templates.run_request_template` (formatted JSON)
6. **Status Template** - `templates.status_request_template` (formatted JSON)
7. **Delivery Template** - `templates.delivery_request_template` (formatted JSON)

**Performance Section Display Options**:
1. **Success Rate Chart** - Trend over time
2. **Run Duration Chart** - Average duration over time
3. **Runs per Day Chart** - Daily run volume
4. **Error Rate Chart** - Failed runs over time
5. **Queue Status** - Current queue length and processing
6. **Resource Utilization** - CPU, memory usage
7. **Cost Analysis** - Cost per run, total costs
8. **Performance Metrics** - Response time, throughput

## Phase 2: Activities Pages

### 2.1 Activities List Page
**Data Source**: `activities.json`
**Page Layout Options**:
1. **Page Header** - Title, description, and action buttons
2. **Filters Section** - Filter by type, status, success rate
3. **Search Bar** - Search activities by name or description
4. **Stats Cards** - Total activities, active activities, success rate overview
5. **Activities Grid** - Grid of ActivityCard components
6. **Pagination** - Page navigation for large datasets
7. **Bulk Actions** - Select multiple activities for batch operations

**Stats Cards Display Options**:
1. **Total Activities** - `activities.length`
2. **Active Activities** - `activities.filter(a => a.status === 'active').length`
3. **Average Success Rate** - `activities.reduce((acc, a) => acc + a.success_rate, 0) / activities.length`
4. **Total Requests** - `activities.reduce((acc, a) => acc + a.total_requests, 0)`
5. **Average Duration** - `activities.reduce((acc, a) => acc + a.average_duration, 0) / activities.length`
6. **Last Used** - Most recent `last_used_at` across activities
7. **Source Count** - Total unique sources across activities
8. **Workflow Count** - Total unique workflows across activities

**Filter Options**:
1. **Activity Type** - Data processing, ML training, Analysis, etc.
2. **Status** - Active, Inactive, Error, Pending
3. **Success Rate Range** - High (>90%), Medium (70-90%), Low (<70%)
4. **Last Used** - Today, This week, This month, Older
5. **Owner Organization** - Filter by organization
6. **Source Type** - Filter by associated source types

### 2.2 Activity Detail Page
**Data Source**: Individual activity JSON, activity_request table data
**Page Layout Options**:
1. **Page Header** - User-defined activity name, subheadline as outcome + target from description
2. **Content Tabs** - Tabbed interface for different sections

**Page Header Display Options**:
1. **Activity Name** - User-defined name (editable)
2. **Subheadline** - Outcome + target (rendered from description)
3. **Status Badge** - Current activity status
4. **Action Buttons** - Edit, delete, clone, export

**Content Tabs Display Options**:
1. **Setup Tab**
   - **BaseAccordion** components per source with "Add source" button below last accordion
   - **Source Name** - Accordion title is the source name
   - **Source Content** - **UsecaseCard** and **GuardrailCard** components (factors) from setup data inside accordion content
   - **Add Source Button** - Primary button below last accordion
   - **Visual Differentiation** - Guardrails and use cases visually different due to content
2. **Statistics Tab**
   - **Data Source**: `activity_request` table data
   - **Charts and Metrics** - Performance analytics and visualizations
   - **Key Performance Indicators** - Success rates, durations, usage statistics
3. **History Tab**
   - **Data Source**: `activity_request` table data
   - **Request History** - Complete history of activity requests
   - **Filtering and Search** - Filter by date, status, outcome
   - **Export Options** - Export historical data

## Phase 3: Workflows Pages

### 3.1 Workflows List Page
**Data Source**: `workflows.json`
**Page Layout Options**:
1. **Page Header** - Title, description, and action buttons
2. **Filters Section** - Filter by status, success rate, phases
3. **Search Bar** - Search workflows by name or description
4. **Stats Cards** - Total workflows, active workflows, success rate overview
5. **Workflows Grid** - Grid of WorkflowCard components
6. **Pagination** - Page navigation for large datasets
7. **Bulk Actions** - Select multiple workflows for batch operations

**Stats Cards Display Options**:
1. **Total Workflows** - `workflows.length`
2. **Active Workflows** - `workflows.filter(w => w.status === 'active').length`
3. **Average Success Rate** - `workflows.reduce((acc, w) => acc + w.success_rate, 0) / workflows.length`
4. **Total Executions** - `workflows.reduce((acc, w) => acc + w.total_executions, 0)`
5. **Successful Executions** - `workflows.reduce((acc, w) => acc + w.successful_executions, 0)`
6. **Failed Executions** - `workflows.reduce((acc, w) => acc + w.failed_executions, 0)`
7. **Average Duration** - `workflows.reduce((acc, w) => acc + w.average_duration, 0) / workflows.length`
8. **Activities Count** - Total activities across workflows
9. **Phases Count** - Total phases across workflows

**Filter Options**:
1. **Status** - Active, Inactive, Error, Pending
2. **Success Rate Range** - High (>90%), Medium (70-90%), Low (<70%)
3. **Phases Count** - 1-2, 3-5, 6-10, 10+
4. **Activities Count** - 1-5, 6-15, 16-30, 30+
5. **Last Used** - Today, This week, This month, Older
6. **Owner Organization** - Filter by organization
7. **Verification Status** - Verified, Unverified

### 3.2 Workflow Detail Page
**Data Source**: Individual workflow JSON, workflow_request table data
**Page Layout Options**:
1. **Page Header** - Workflow name, status badge, action buttons
2. **Content Tabs** - Tabbed interface for different sections

**Page Header Display Options**:
1. **Workflow Name** - `name`
2. **Description** - `description`
3. **Status** - `status` (with status indicator)
4. **Action Buttons** - Edit, delete, clone, export

**Content Tabs Display Options**:
1. **Setup Tab**
   - **BaseAccordion** components per phase with "Add phase" button below last accordion
   - **Phase Name** - Accordion title is the phase name
   - **Phase Content** - Display from workflow setup data
   - **Always Show Phase 1** - Even with blank workflow detail
   - **Add Phase Button** - Primary button under last accordion
   - **Phase Activities** - **ActivityCard** components for each activity in the phase
2. **Statistics Tab**
   - **Data Source**: `workflow_request` table data
   - **Charts and Metrics** - Performance analytics and visualizations
   - **Key Performance Indicators** - Success rates, durations, execution statistics
   - **Phase Performance** - Statistics per workflow phase
3. **History Tab**
   - **Data Source**: `workflow_request` table data
   - **Execution History** - Complete history of workflow executions
   - **Filtering and Search** - Filter by date, status, phase, outcome
   - **Export Options** - Export historical execution data

## Phase 4: Common Page Components

### 4.1 Page Header Component
**Display Options**:
1. **Page Title** - Dynamic title based on page type
2. **Page Description** - Brief description of the page
3. **Action Buttons** - Create, refresh, export, settings
4. **Breadcrumbs** - Navigation hierarchy
5. **Status Indicators** - Overall system status
6. **User Context** - Current user and organization

### 4.2 Filters Section Component
**Display Options**:
1. **Filter Dropdowns** - Multiple filter options
2. **Search Input** - Text search functionality
3. **Date Range Picker** - Date-based filtering
4. **Quick Filters** - Predefined filter combinations
5. **Filter Chips** - Active filter indicators
6. **Clear Filters** - Reset all filters

### 4.3 Stats Cards Section Component
**Display Options**:
1. **Metric Cards** - Key performance indicators
2. **Trend Indicators** - Up/down arrows with percentages
3. **Comparison Data** - Current vs previous period
4. **Progress Bars** - Completion percentages
5. **Status Badges** - Health and status indicators
6. **Interactive Cards** - Clickable cards for drill-down

### 4.4 Bento Grid Component
**Component Location**: `components/layout/grid.tsx` (enhanced)
**Utility Functions**: `utils/layout-utils.tsx` (new file)

#### 4.4.1 Bento Grid Implementation Specification

**Research Foundation**:
- **Bin Packing Algorithm**: Based on 2D rectangular bin packing optimization research
- **First-Fit Decreasing (FFD)**: Proven algorithm for optimal space utilization in 2D layouts
- **CSS Grid Integration**: Leverages modern CSS Grid capabilities for responsive positioning
- **Content-Driven Sizing**: Card dimensions reflect data significance and content importance

**Core Functionality**:
1. **Dynamic Card Sizing** - Cards sized based on total runs relative to other cards
2. **Quadrant-Based Distribution** - Divide total numbers into 4 quadrants for proportional sizing
3. **Width Calculation** - Range from 1/6th to 2/3rds of container width
4. **Height Calculation** - Minimum and maximum based on width and content length
5. **Proportional Distribution** - Largest total runs = 2/3rds width, smallest = 1/6th width
6. **Content-Based Height** - Height determined by description length and card width
7. **Responsive Layout** - Adapts to different screen sizes
8. **Bin Packing Optimization** - Uses FFD algorithm to minimize empty spaces and maximize layout efficiency

#### 4.4.2 Layout Utils Functions Specification

**File**: `utils/layout-utils.ts`

**Function 1: `determineCardSize(totalRuns: number, description: string, gridColumns: number)`**
- **Purpose**: Calculate grid spans based on total runs, description length, and available columns
- **Input**: totalRuns number, description string, gridColumns number
- **Process**: 
  - Calculate maximum width as half of gridColumns: `maxWidth = Math.floor(gridColumns / 2)`
  - Calculate width spans based on total runs (1 span minimum, maxWidth maximum)
  - Calculate rows based on description length + calculated width
  - Determine optimal grid-column and grid-row spans
  - Apply constraints: minimum 1x1, maximum maxWidth x 3
- **Output**: Object with `{cols: number, rows: number}` representing grid spans

**Function 2: `determineGridLayout(cards: CardData[])`**
- **Purpose**: Optimize card placement using bin packing algorithm for space efficiency
- **Algorithm**: First-Fit Decreasing (FFD) bin packing algorithm
- **Research Basis**: Based on bin packing optimization research for 2D rectangular items
- **Input**: Array of card data with calculated spans
- **Process**:
  - Sort cards by size (largest first) - FFD algorithm requirement
  - Use first-fit decreasing algorithm for optimal space utilization
  - Place cards in optimal grid positions to minimize empty spaces
  - Handle overflow and repositioning for edge cases
  - Apply CSS Grid auto-placement for remaining positioning
- **Output**: Array of positioned cards with grid coordinates and spans

#### 4.4.3 Enhanced Grid Component Specification

**Component**: `components/layout/grid.tsx`

**New Props Interface**:
- `bentoMode: boolean` - Enable bento grid layout
- `dataSource: 'sources' | 'activities' | 'workflows'` - JSON data source to read from
- `gridColumns: number` - Number of grid columns (default: 6)
- `gridGap: string` - Grid gap size (default: "1rem")

**Dynamic Width Calculation**:
- **Maximum Card Width**: Always half of available columns (gridColumns / 2)
- **Examples**: 4 columns = max 2 spans, 6 columns = max 3 spans, 8 columns = max 4 spans
- **Width Categories**: Dynamically calculated based on gridColumns value
- **Formula**: `maxWidth = Math.floor(gridColumns / 2)`

**Enhanced Functionality**:
1. **Data Reading** - Read data from sources.json, activities.json, or workflows.json based on dataSource prop
2. **Bento Mode Detection** - When bentoMode is true, orchestrate layout utils functions
3. **Card Size Calculation** - Call `determineCardSize()` for each card based on total runs and description
4. **Grid Layout Calculation** - Call `determineGridLayout()` to optimize card placement
5. **Grid Span Application** - Apply calculated grid-column and grid-row spans to cards
6. **CSS Grid Generation** - Generate `grid-template-columns: repeat(6, 1fr)` and auto-rows
7. **Responsive Adaptation** - Adjust grid columns for different screen sizes
8. **Auto-placement** - Let CSS Grid handle remaining positioning automatically

#### 4.4.4 Dynamic Card Size Categories

**Dynamic Width Categories (based on gridColumns)**:
- **Maximum Width**: Always `Math.floor(gridColumns / 2)` spans
- **Width Distribution**: Divided into 4 categories based on total runs quartiles
- **Examples**:
  - 4 columns: max 2 spans (Width 1: 1 span, Width 2: 2 spans)
  - 6 columns: max 3 spans (Width 1: 1 span, Width 2: 2 spans, Width 3: 3 spans)
  - 8 columns: max 4 spans (Width 1: 1 span, Width 2: 2 spans, Width 3: 3 spans, Width 4: 4 spans)

**4 Height Categories (in rows)**:
- **Height 1**: 1 row - Short descriptions
- **Height 2**: 2 rows - Medium descriptions
- **Height 3**: 3 rows - Long descriptions
- **Height 4**: 4 rows - Very long descriptions

**Size Assignment Logic**:
- **Width**: Based on total runs quartiles relative to maximum width
- **Height**: Based on description length + width factor
- **Formula**: `Math.max(1, Math.min(maxWidth, Math.ceil((totalRuns / maxTotalRuns) * maxWidth)))`

#### 4.4.5 Responsive Behavior

**Mobile (< 768px)**:
- All cards stack vertically
- Full width for each card
- Height based on content only

**Tablet (768px - 1024px)**:
- 2-column grid maximum
- Bento sizing applied within columns
- Adjusted height calculations

**Desktop (1024px - 1440px)**:
- Full bento grid layout
- All quadrants active
- Complete responsive sizing

**Large Desktop (> 1440px)**:
- Enhanced spacing
- Larger minimum card sizes
- Optimized for wide screens

#### 4.4.6 Integration Points

**Card Components**:
- Pass calculated dimensions as props
- Apply generated CSS classes
- Handle content overflow within calculated bounds

**Page Components**:
- Provide card data with total runs
- Set container width dynamically
- Handle layout updates on data changes

**Animation System**:
- Smooth transitions when cards resize
- Staggered animations for card appearance
- Layout shift prevention during updates

## Phase 5: New Page Creation

### 5.1 Source Page
**Data Source**: Individual source configuration
**Page Layout Options**:
1. **Page Header** - Source name, status badge, action buttons
2. **Content Tabs** - Tabbed interface for different sections
3. **Save Button** - Primary save action

**Page Header Display Options**:
1. **Source Name** - Dynamic source name
2. **Status Badge** - Current source status
3. **Save Button** - Primary save action
4. **Action Buttons** - Edit, delete, clone, export

**Content Tabs Display Options**:
1. **Execution Tab**
   - **BaseAccordion** (title: "Request") with **InferenceForm** component
   - **BaseAccordion** (title: "Response") with **JsonResponseMapper** component
   - **LoadBalancingForm** component for concurrency and timeout configuration
2. **Customization Tab**
   - **BaseAccordion** (title: "Source Use Cases") with **UsecaseCard** components (factors with type: "use-case")
   - **BaseAccordion** (title: "Source Guardrails") with **GuardrailCard** components (factors with type: "guardrail")
   - **Customization Options** - User-based guardrails and use cases
   - **Subheadline** - "Can users have their own custom?"
   - **Radio Buttons** - Never, Yes but after approval, Only use-cases, Yes they can do anything
   - **JSON Generation Instructions** - (only visible to admin)
3. **Delivery Tab**
   - **BaseAccordion** (title: "Output mappings") with **JsonResponseMapper** component
   - **BaseAccordion** (title: "Status Check Request") with **StatusCheckForm** component
   - **BaseAccordion** (title: "Deliver Requesty") with **DeliveryForm** component

### 5.2 Data Page Enhancement
**Data Source**: Object type data files
**Page Layout Options**:
1. **Page Header** - Data type name, description, export button
2. **Data Table Component** - Complete content as data table
3. **Export as JSON** - Download functionality

**Page Header Display Options**:
1. **Data Type Title** - Object type name
2. **Description** - Brief description of data type
3. **Export as JSON Button** - Triggers JSON download from current view

**Data Table Component Display Options**:
1. **Complete Content** - Data table as whole page content
2. **JSON Export** - Download current view data as JSON file
3. **Table Features** - Sorting, filtering, pagination

## Phase 6: Navigation and Routing

### 6.1 Page Navigation
**Navigation Options**:
1. **Main Navigation** - Sources, Activities, Workflows
2. **Breadcrumbs** - Page hierarchy navigation
3. **Quick Actions** - Common actions from any page
4. **Search** - Global search functionality
5. **Recent Items** - Recently viewed items
6. **Favorites** - Bookmarked items

### 6.2 Detail Page Navigation
**Navigation Options**:
1. **Back Button** - Return to list page
2. **Edit Button** - Edit current item
3. **Delete Button** - Delete current item
4. **Clone Button** - Create copy of current item
5. **Export Button** - Export item data
6. **Share Button** - Share item with others
7. **Related Items** - Navigate to related items

## File Changes Required

### New Files
- `app/sources/page.tsx` - Sources list page
- `app/sources/[id]/page.tsx` - Source detail page with Execution/Customization/Delivery tabs using **BaseAccordion**, **InferenceForm**, **JsonResponseMapper**, **LoadBalancingForm**, **UsecaseCard**, **GuardrailCard**, **StatusCheckForm**, **DeliveryForm**
- `app/activities/page.tsx` - Activities list page
- `app/activities/[id]/page.tsx` - Activity detail page with Setup/Statistics/History tabs using **BaseAccordion**, **UsecaseCard**, **GuardrailCard**
- `app/workflows/page.tsx` - Workflows list page
- `app/workflows/[id]/page.tsx` - Workflow detail page with Setup/Statistics/History tabs using **BaseAccordion**, **ActivityCard**
- `components/pages/page-header.tsx` - Reusable page header
- `components/pages/filters-section.tsx` - Reusable filters
- `components/pages/stats-cards.tsx` - Reusable stats cards
- `components/others/stats-overview.tsx` - Statistics overview component
- `utils/layout-utils.ts` - Bento grid layout calculation utilities

### Modified Files
- `components/layout/grid.tsx` - Enhanced with bento grid functionality and dynamic sizing
- `components/cards/source-card.tsx` - Enhanced with stats overview, status, owner info, last used
- `components/cards/activity-card.tsx` - Enhanced with stats overview, status, owner info, last used
- `components/cards/workflow-card.tsx` - Enhanced with stats overview, status, owner info, last used
- `components/cards/usecase-card.tsx` - Enhanced with factor definition badges
- `components/cards/guardrail-card.tsx` - Enhanced with factor definition badges
- `components/others/last-used.tsx` - Enhanced with opacity based on time intervals
- `components/others/progress-bar.tsx` - Enhanced for stats overview usage
- `components/accordions/activity-source-accordion.tsx` - Enhanced for activity detail pages with **UsecaseCard** and **GuardrailCard** components
- `components/accordions/workflow-phase-accordion.tsx` - Enhanced for workflow detail pages with **ActivityCard** components
- `components/accordions/base-accordion.tsx` - Base accordion component used throughout detail pages
- `components/accordions/` - Enhanced for detail pages
- `components/charts/` - Enhanced for detail pages

## Success Criteria

1. **Data Display**: Clear presentation of JSON data in appropriate page layouts
2. **User Experience**: Intuitive navigation between list and detail pages
3. **Performance**: Efficient rendering of large datasets
4. **Accessibility**: Full WCAG 2.1 AA compliance
5. **Responsiveness**: Seamless experience across all device sizes
6. **Component Reusability**: Reusable components across different pages
7. **Data Integration**: Seamless connection to JSON data sources
