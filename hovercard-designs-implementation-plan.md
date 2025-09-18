# Hovercard Designs Implementation Plan

## Component Usage Validation
**✅ COMPONENT SOURCE CHECK:**
- [ ] Uses only components from `components/` folder
- [ ] Uses only components from `components/ui-elements/` subfolder
- [ ] **NO direct usage of `components/ui/` folder components**

## Overview
Implement specialized hovercards that provide contextual database information, real-time status, and detailed insights on hover for various UI elements using the existing hovercard components.

## Phase 1: Database-Focused Hovercard Architecture

### 1.1 Hovercard System Architecture
- **Status Hovercards**: Real-time status and progress information from database
- **Performance Hovercards**: Metrics, trends, and comparisons from org table
- **Configuration Hovercards**: JSON template details and parameter information
- **Alert Hovercards**: Warning and error information from run_queue

### 1.2 Core Components Structure
- **BaseHovercard**: Universal hovercard wrapper with database data
- **ValueHovercard**: Display value details and metadata
- **ProgressHovercard**: Show progress and completion status
- **MappingHovercard**: Display mapping and relationship information
- **CustomHovercard**: Flexible hovercard for specific use cases

### 1.3 Data Display Design
- **Contextual Information**: Relevant data based on hover target from database
- **Real-time Updates**: Live status and metric updates from database triggers
- **Detailed Insights**: Expanded information beyond basic display
- **Action Items**: Quick actions and next steps using ETL functions

## Phase 2: Specialized Hovercard Components

### 2.1 ValueHovercard Component
- **Value Information**: 
  - Value details from value table
  - Confidence scores and validation status
  - Source and timestamp information
- **Metadata Display**: 
  - Object and datapoint relationships
  - Activity request context
  - Value history and changes
- **Quality Indicators**: 
  - Confidence scores from value.confidence
  - Status codes from value.status_code
  - Validation results and errors
- **Related Data**: 
  - Connected objects and values
  - Source information and performance
  - Activity and workflow context

### 2.2 ProgressHovercard Component
- **Progress Information**: 
  - Current progress from activity_request and run_queue
  - Completion percentages and estimates
  - Phase and workflow progress
- **Status Details**: 
  - Detailed status from database tables
  - Error information and resolution
  - Next steps and actions
- **Performance Metrics**: 
  - Success rates and timing
  - Queue status and processing
  - Resource utilization
- **Action Items**: 
  - Available actions and operations
  - Quick fixes and overrides
  - Status change options

### 2.3 MappingHovercard Component
- **Mapping Information**: 
  - Object and datapoint relationships
  - Source mappings and configurations
  - Factor and rule mappings
- **Configuration Details**: 
  - Mapping setup and parameters
  - Template configurations
  - Validation rules and thresholds
- **Usage Information**: 
  - Where mappings are applied
  - Performance impact and effectiveness
  - Change history and versioning
- **Management Options**: 
  - Edit and update mappings
  - Test and validate configurations
  - Import and export options

### 2.4 CustomHovercard Component
- **Flexible Content**: 
  - Adaptable content based on context
  - Dynamic data display
  - Custom formatting and layout
- **Context Awareness**: 
  - Hover target identification
  - Relevant data selection
  - Appropriate content display
- **Integration Capabilities**: 
  - Database data binding
  - Real-time updates
  - Interactive elements

## Phase 3: JSON Template Hovercards

### 3.1 Template Variable Hovercard
- **Variable Information**: 
  - Variable placeholders (<<$run_setup>>, <<$max_objects>>)
  - Variable resolution from utils.replace_request_placeholders()
  - Variable sources and dependencies
- **Template Structure**: 
  - HTTP method, URL, headers, body structure
  - Template validation and requirements
  - Template version and history
- **Usage Examples**: 
  - Sample resolved templates
  - Common use cases and patterns
  - Template testing examples

### 3.2 Configuration Hovercard
- **Setup Details**: 
  - Activity setup configuration from activity.setup
  - Workflow setup from workflow.setup JSON
  - Factor setup from factor.setup JSON
- **Parameter Information**: 
  - Configuration parameters and options
  - Parameter validation and limits
  - Parameter dependencies and relationships
- **Configuration History**: 
  - Configuration change history
  - Version tracking and rollback
  - Configuration validation status

## Phase 4: Interactive Features & Data Display

### 4.1 Real-time Information
- **Live Updates**: 
  - Real-time status and metric updates from database
  - Live progress and completion updates
  - Immediate status change notifications
- **Progress Tracking**: 
  - Live progress and completion updates
  - Real-time queue status updates
  - Performance monitoring updates
- **Status Changes**: 
  - Immediate status change notifications
  - Status transition tracking
  - Status history and patterns

### 4.2 Detailed Insights
- **Expanded Data**: 
  - Comprehensive information beyond basic display
  - Detailed metrics and statistics
  - Historical data and trends
- **Historical Context**: 
  - Historical trends and patterns
  - Performance over time
  - Growth and improvement patterns
- **Comparative Analysis**: 
  - Current vs previous performance
  - Benchmark comparisons
  - Trend analysis and predictions
- **Predictive Insights**: 
  - Trend analysis and predictions
  - Performance forecasting
  - Resource planning recommendations

### 4.3 Quick Actions
- **Primary Actions**: 
  - Most common actions and operations
  - Status-specific actions
  - Emergency actions and overrides
- **Context Menus**: 
  - Right-click action menus
  - Context-sensitive operations
  - Quick access to common functions
- **Keyboard Shortcuts**: 
  - Quick keyboard access to actions
  - Shortcut customization
  - Accessibility improvements
- **Bulk Operations**: 
  - Multi-item action support
  - Batch operations and processing
  - Bulk status updates

## Phase 5: Integration & Advanced Features

### 5.1 Database Integration
- **Real-time Data**: 
  - Live database updates and refresh
  - Real-time status and metric updates
  - Live performance monitoring
- **Data Binding**: 
  - Dynamic content from database
  - Real-time data binding
  - Live metric calculations
- **Error Handling**: 
  - Graceful database error display
  - Error recovery and retry
  - User-friendly error messages
- **Loading States**: 
  - Skeleton loading for data
  - Progressive data loading
  - Loading indicators and progress

### 5.2 Advanced Features
- **Smart Positioning**: 
  - Intelligent hovercard positioning
  - Screen space optimization
  - Responsive positioning
- **Content Preloading**: 
  - Anticipate content needs
  - Background data loading
  - Performance optimization
- **Customization**: 
  - User-configurable hovercard content
  - Personalized information display
  - Custom hovercard layouts
- **Analytics**: 
  - Track hovercard usage and effectiveness
  - User interaction patterns
  - Performance optimization

## File Changes Required

### Modified Files
- `components/hovercards/value-hovercard.tsx` - Update to display database value details
- `components/hovercards/progress-hovercard.tsx` - Update to show progress and status
- `components/hovercards/mapping-hovercard.tsx` - Update to display mapping information
- `components/hovercards/base-hovercard.tsx` - Enhance with database data capabilities
- `components/hovercards/index.ts` - Ensure all existing hovercards are exported

### No New Files Required
- All specialized hovercard components already exist
- Focus on updating existing components with database integration

## Testing Checklist

### Unit Tests
- [ ] Hovercard rendering with database data
- [ ] Positioning calculations and collision detection
- [ ] Event handling for real-time updates
- [ ] Content rendering from database

### Integration Tests
- [ ] Complete hovercard workflow with database
- [ ] Trigger interactions with real-time data
- [ ] Content display from database
- [ ] Error handling for database failures

### User Experience Tests
- [ ] Hovercard interaction flow with real-time data
- [ ] Mobile responsiveness with database metrics
- [ ] Accessibility compliance for data display
- [ ] Performance with many hovercards and live updates

## Success Criteria

1. **Contextual Information**: Relevant and helpful information on hover from database
2. **JSON Integration**: Effective display of JSON templates and configurations
3. **User Experience**: Smooth and intuitive hover interactions
4. **Performance**: Fast loading and real-time updates
5. **Accessibility**: Full WCAG 2.1 AA compliance
6. **Responsiveness**: Seamless experience across all device sizes
7. **Data Integration**: Seamless database data display and interaction
