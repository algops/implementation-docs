# Hovercard Designs Implementation Plan

## Component Usage Validation
**✅ COMPONENT SOURCE CHECK:**
- [ ] Uses only components from `components/` folder
- [ ] Uses only components from `components/ui-elements/` subfolder
- [ ] **NO direct usage of `components/ui/` folder components**

## Overview
Implement specialized hovercards that provide contextual database information, real-time status, and detailed insights on hover for various UI elements using standardized hovercard components.

## CRITICAL: Hovercard Standardization Requirements

### BaseHovercard Component - Core Infrastructure

**Purpose**: Single source of truth for all hovercard behavior, ensuring consistency across all contextual information displays.

**Size System**:
```typescript
interface HovercardSizeConfig {
  size?: 'sm' | 'md' | 'lg'
  // sm: w-64 (256px) - Compact info, status badges, simple tooltips
  // md: w-80 (320px) - Standard content, mapping details, value information
  // lg: w-96 (384px) - Complex content, progress statistics, detailed insights
}
```

**Visual Styling System**:
- Background: `bg-content` - Hovercard content background color
- Shadow: `shadow-xl` - Extra large drop shadow for elevation
- Border: `border` - Standard 1px border (simpler than dialogs)
- Border Color: Standard border color from theme
- Ensures consistent visual hierarchy across all hovercards
- Provides clear depth separation from background content

**Animation System** (Pattern A - Standardized - Same as Dialogs):
- Use `useAnimation()` hook with destructured `{ ref, className: animationClass }`
- Apply ref to HoverCardContent root element
- Apply animationClass to content wrapper div
- Consistent fade-in and scale animations for all hovercards
- Configurable animation duration and easing
- Disabled animations option for accessibility preferences

**Timing System**:
```typescript
interface HovercardTimingConfig {
  openDelay?: number      // Delay before showing (default: 700ms)
  closeDelay?: number     // Delay before hiding (default: 0ms)
}
```

**Positioning System**:
```typescript
interface HovercardPositionConfig {
  side?: "top" | "right" | "bottom" | "left"     // Default: "top"
  align?: "start" | "center" | "end"             // Default: "center"
  followCursor?: boolean                         // Dynamic positioning (default: false)
}
```

**Behavior System**:
```typescript
interface HovercardBehavior {
  allowMouseClose?: boolean    // Close on mouse movement (default: true)
  closeOnScroll?: boolean      // Close on scroll (default: true)
  persistOnHover?: boolean     // Keep open when hovering content (default: true)
}
```

**Button System Standardization**:
- Only use custom buttons: PrimaryButton, SecondaryButton, TextButton
- Never use standard Button component from shadcn/ui
- Consistent action layout with appropriate button types
- Primary actions use PrimaryButton (rare in hovercards)
- Secondary actions use SecondaryButton
- Link-style actions use TextButton (most common)
- Loading states built into button components
- Always use `bg-transparent` override for buttons inside hovercards

**Badge System Standardization**:
- Only use BaseBadge from custom-ui-elements
- Never use standard Badge component from shadcn/ui
- Use state prop for semantic meaning: `default | active | recommended | warning | error`
- Use variant prop for visual style: `default | secondary | destructive | outline`
- Consistent sizing with size prop: `sm | default | lg`

**Callback Naming Conventions**:
```typescript
interface HovercardCallbacks {
  onAction?: () => void                          // Primary action (most common)
  onSecondaryAction?: () => void                 // Secondary action
  onNavigate?: () => void                        // Navigation/exploration action
  onShowMore?: () => void                        // Expand to see more details
  onShowAll?: (filter?: string) => void         // Show all items of a type
  onSelect?: (item: any) => void                 // Select an item
}
```

**Core Props Interface**:
```typescript
interface BaseHovercardProps {
  // Content
  children: React.ReactNode                      // Trigger element
  content: React.ReactNode                       // Hovercard content
  
  // Configuration
  size?: 'sm' | 'md' | 'lg'                      // Default: 'md'
  
  // Positioning
  side?: "top" | "right" | "bottom" | "left"     // Default: "top"
  align?: "start" | "center" | "end"             // Default: "center"
  followCursor?: boolean                         // Default: false
  
  // Timing
  openDelay?: number                             // Default: 700
  closeDelay?: number                            // Default: 0
  
  // Behavior
  allowMouseClose?: boolean                      // Default: true
  closeOnScroll?: boolean                        // Default: true
  persistOnHover?: boolean                       // Default: true
  
  // Customization
  className?: string                             // Additional classes for trigger
  contentClassName?: string                      // Additional classes for content
}

// Note: Visual styling is built into BaseHovercard and not configurable via props:
// - bg-content (hovercard background)
// - shadow-xl (drop shadow)
// - border (standard 1px border)
```

**Implementation Rules**:
1. **All hovercards MUST extend BaseHovercard** - No direct HoverCard component usage
2. **Size must be specified via size prop** - No hardcoded width classes in contentClassName
3. **Buttons must be from custom-ui-elements** - No shadcn Button usage
4. **Badges must be from custom-ui-elements** - No shadcn Badge usage
5. **Callbacks must follow naming conventions** - Consistent naming across all hovercards
6. **Animation must use useAnimation() hook** - Same pattern as dialogs
7. **Content must use bg-content** - Consistent background across all hovercards
8. **No bg override in contentClassName** - Visual styling is standardized

**Migration Checklist for Existing Hovercards**:
- [ ] Remove hardcoded width classes (w-64, w-80, w-96) from contentClassName
- [ ] Add size prop with appropriate value ('sm' | 'md' | 'lg')
- [ ] Replace any Button imports with custom-ui-element buttons
- [ ] Replace any Badge imports with BaseBadge
- [ ] Add useAnimation() hook for consistent animations
- [ ] Ensure all callbacks follow naming conventions
- [ ] Remove any bg-* overrides from contentClassName
- [ ] Apply bg-transparent to all buttons for proper visual hierarchy
- [ ] Ensure border is standard (not border-2)

## Phase 1: Database-Focused Hovercard Architecture

### 1.1 Hovercard System Architecture
- **Status Hovercards**: Real-time status and progress information from database
- **Performance Hovercards**: Metrics, trends, and comparisons from org table
- **Configuration Hovercards**: JSON template details and parameter information
- **Alert Hovercards**: Warning and error information from run_queue

### 1.2 Core Components Structure with Data Options

#### BaseHovercard
**Data Source**: Universal wrapper for all hovercards
**Display Options**:
1. **Content Rendering** - Flexible content display system
2. **Positioning** - Dynamic positioning based on screen space
3. **Animation** - Smooth entrance/exit animations
4. **Accessibility** - ARIA labels and keyboard support
5. **Responsive Design** - Adapt to different screen sizes

#### ValueHovercard
**Data Source**: `datapoints.json`, `object-types.json`
**Display Options**:
1. **Value Details** - `value` field with formatting
2. **Confidence Score** - `confidence` percentage
3. **Status Code** - `status_code` with color coding
4. **Object Type** - `object_type_id` (resolved name)
5. **Datapoint** - `datapoint_id` (resolved name)
6. **Source Information** - Source that generated the value
7. **Timestamp** - `created_at` and `updated_at`
8. **Validation Status** - Whether value passed validation
9. **Related Values** - Other values for same object
10. **History** - Value change history over time

#### ProgressHovercard
**Data Source**: `activities.json`, `sources.json`, `workflows.json`
**Display Options**:
1. **Current Status** - `status` with visual indicator
2. **Progress Percentage** - Completion percentage
3. **Time Elapsed** - Time since start
4. **Estimated Time Remaining** - Based on average duration
5. **Steps Completed** - Number of completed steps
6. **Total Steps** - Total number of steps
7. **Current Phase** - Current workflow phase
8. **Next Action** - What happens next
9. **Error Information** - Any errors encountered
10. **Performance Metrics** - Success rate, average duration

#### MappingHovercard
**Data Source**: `sources.json`, `factors.json`
**Display Options**:
1. **Mapping Type** - Source mapping, factor mapping, etc.
2. **Source Information** - Source name and type
3. **Target Information** - What's being mapped to
4. **Mapping Rules** - JSON template structure
5. **Variable Placeholders** - `<<$run_setup>>`, `<<$max_objects>>`
6. **Validation Rules** - Mapping validation criteria
7. **Usage Statistics** - How often mapping is used
8. **Last Updated** - When mapping was last modified
9. **Dependencies** - Other mappings this depends on
10. **Test Results** - Recent mapping test results

#### CustomHovercard
**Data Source**: Context-dependent
**Display Options**:
1. **Dynamic Content** - Content based on hover target
2. **Contextual Data** - Relevant data for specific context
3. **Interactive Elements** - Buttons, links, forms
4. **Real-time Updates** - Live data refresh
5. **Custom Styling** - Context-specific styling
6. **Action Buttons** - Quick actions available
7. **Related Information** - Links to related data
8. **Status Indicators** - Visual status representation
9. **Performance Metrics** - Relevant performance data
10. **Configuration Options** - Settings and preferences

### 1.3 Data Display Design
- **Contextual Information**: Relevant data based on hover target from database
- **Real-time Updates**: Live status and metric updates from database triggers
- **Detailed Insights**: Expanded information beyond basic display
- **Action Items**: Quick actions and next steps using ETL functions

## Phase 2: Specialized Hovercard Components - Standardized Implementation

### 2.0 Specialized Hovercard Pattern

**All specialized hovercards MUST follow this pattern**:

```typescript
// 1. Import BaseHovercard and custom-ui-elements only
import { BaseHovercard } from "./base-hovercard"
import { PrimaryButton, SecondaryButton, TextButton } from "@/components/ui/custom-ui-elements/buttons"
import { BaseBadge } from "@/components/ui/custom-ui-elements/badges"

// 2. Define props with standardized callback names
interface SpecializedHovercardProps {
  children: React.ReactNode
  // Domain-specific data props
  data?: SomeDataType
  // Standardized callbacks (use: onAction, onNavigate, onShowMore, onShowAll)
  onAction?: () => void
  onNavigate?: () => void
  // Standard BaseHovercard overrides
  size?: 'sm' | 'md' | 'lg'     // Default based on content complexity
  delay?: number                // Override default 700ms if needed
}

// 3. Build content internally as React.ReactNode
export function SpecializedHovercard({ 
  children, 
  data,
  onAction,
  onNavigate,
  size = "md",  // Choose appropriate default
  delay = 700 
}: SpecializedHovercardProps) {
  const content = (
    <div className="space-y-3">
      {/* Content structure using custom-ui-elements */}
      <div>
        <h4 className="text-sm font-semibold">Title</h4>
        {/* Data display */}
      </div>
      
      {/* Badges for status/state */}
      <BaseBadge state="active" size="sm">Status</BaseBadge>
      
      {/* Actions using custom buttons with bg-transparent */}
      {onAction && (
        <TextButton size="sm" onClick={onAction} className="bg-transparent">
          Action Text
        </TextButton>
      )}
    </div>
  )

  // 4. Pass to BaseHovercard with appropriate configuration
  return (
    <BaseHovercard 
      size={size}
      openDelay={delay}
      content={content}
    >
      {children}
    </BaseHovercard>
  )
}
```

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

### New Files - Utility Infrastructure
- `utils/hovercards/hovercard-utils.ts` - Size and positioning utilities (mirrors dialog-utils pattern)
- `utils/hovercards/hovercard-types.ts` - TypeScript type definitions for hovercards
- `utils/hovercards/index.ts` - Export utilities for easy importing

### Modified Files - Core Infrastructure
- `components/hovercards/base-hovercard.tsx` - **CRITICAL REFACTOR**:
  - Add size system with getSizeClassName utility
  - Add useAnimation() hook integration
  - Remove hardcoded w-96 default width
  - Add bg-content and shadow-xl to contentClassName
  - Ensure border (not border-2)
  - Add comprehensive TypeScript interface
  - Remove contentClassName width overrides

### Modified Files - Specialized Hovercards
- `components/hovercards/value-hovercard.tsx` - **STANDARDIZE**:
  - Replace Button with custom-ui-elements buttons
  - Remove hardcoded widths
  - Add size prop (default: 'md')
  - Standardize callbacks: onNavigate instead of onExplore
  - Add bg-transparent to buttons
  
- `components/hovercards/progress-hovercard.tsx` - **STANDARDIZE**:
  - Remove hardcoded widths
  - Add size prop (default: 'lg' for complex statistics)
  - Keep onShowAll (matches convention)
  - Add bg-transparent to TextButton
  - Ensure BaseBadge usage is consistent
  
- `components/hovercards/mapping-hovercard.tsx` - **STANDARDIZE**:
  - Remove w-64 hardcoded width
  - Add size prop (default: 'sm')
  - Rename onChooseMapping to onAction
  - Add bg-transparent to buttons
  - Ensure BaseBadge usage
  
- `components/hovercards/private-beta-hovercard.tsx` - **STANDARDIZE**:
  - Remove w-80 hardcoded width
  - Add size prop (default: 'md')
  - Rename onJoinWaitlist to onAction
  - Already uses TextButton (good)
  - Add bg-transparent to button
  
- `components/hovercards/custom-hovercard-example.tsx` - **STANDARDIZE**:
  - Remove w-64 hardcoded width
  - Add size prop (default: 'sm')
  - Example should demonstrate standardized pattern
  
- `components/hovercards/index.ts` - Update exports if needed

### Implementation Priority

**Phase 1 - Infrastructure** (Do First):
1. Create `utils/hovercards/hovercard-types.ts`
2. Create `utils/hovercards/hovercard-utils.ts`
3. Create `utils/hovercards/index.ts`
4. Update `components/hovercards/base-hovercard.tsx`

**Phase 2 - Standardization** (Do Second):
5. Update all specialized hovercards to use new BaseHovercard
6. Ensure all custom-ui-elements imports are correct
7. Remove all hardcoded widths
8. Standardize all callback names

**Phase 3 - Database Integration** (Do Last):
9. Add database data integration to specialized hovercards
10. Implement real-time updates
11. Add loading states and error handling

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

## Hovercard Size Recommendations

### By Component Type
- **BaseHovercard**: Default 'md' unless overridden
- **ValueHovercard**: 'md' - Standard value display with metadata
- **ProgressHovercard**: 'lg' - Complex statistics and multiple status items
- **MappingHovercard**: 'sm' - Compact mapping configuration display
- **PrivateBetaHovercard**: 'md' - Marketing message with CTA
- **CustomHovercardExample**: 'sm' - Simple informational tooltip

### Size Selection Guidelines
- Use 'sm' (w-64) for:
  - Simple tooltips and hints
  - Status indicators
  - Quick reference information
  - Single-action hovercards
  
- Use 'md' (w-80) for:
  - Standard information display
  - Value details with metadata
  - Configuration snippets
  - 1-3 action buttons
  
- Use 'lg' (w-96) for:
  - Complex statistics
  - Multiple data sections
  - Detailed insights
  - 4+ action items
  - Progress tracking with history

## Comparison: Hovercards vs Dialogs

### Similarities (Standardized Across Both)
1. **Base component pattern** - Single source of truth
2. **Size system** - sm/md/lg (hovercards) vs sm/md/lg/xl/full (dialogs)
3. **Custom-ui-elements only** - No shadcn Button/Badge usage
4. **Animation system** - Same useAnimation() hook
5. **Visual styling** - bg-content, shadow-xl
6. **Utility structure** - Separate utils folder with types and helpers
7. **TypeScript interfaces** - Comprehensive prop definitions
8. **Callback conventions** - Standardized naming patterns

### Differences (Context-Appropriate)
1. **Border thickness**: Hovercards use `border` (1px), dialogs use `border-2` (2px)
2. **Size count**: Hovercards have 3 sizes, dialogs have 5 sizes
3. **Close behavior**: Hovercards close on scroll/movement, dialogs have preventClose system
4. **Timing**: Hovercards have openDelay/closeDelay, dialogs don't need delays
5. **Positioning**: Hovercards have side/align/followCursor, dialogs are always centered
6. **Backdrop**: Dialogs have bg-muted/30 backdrop, hovercards don't
7. **Footer buttons**: Dialogs always have confirm/cancel, hovercards use contextual actions
8. **Button styling**: Hovercard buttons need bg-transparent, dialog buttons don't

### Why These Differences Matter
- **Hovercards** are lightweight, contextual, and non-blocking
- **Dialogs** are prominent, modal, and require user decision
- Both follow the same architectural principles but serve different UX purposes
- Standardization ensures consistency within each category while allowing category-specific optimizations

## Success Criteria

1. **Standardization Compliance**: 100% adherence to size system, button usage, and callback naming conventions
2. **Visual Consistency**: All hovercards use bg-content, shadow-xl, border, and custom-ui-elements
3. **Animation Uniformity**: All hovercards use useAnimation() hook with same timing/easing as dialogs
4. **Type Safety**: Comprehensive TypeScript interfaces with proper defaults documented
5. **Contextual Information**: Relevant and helpful information on hover from database
6. **JSON Integration**: Effective display of JSON templates and configurations
7. **User Experience**: Smooth and intuitive hover interactions
8. **Performance**: Fast loading and real-time updates
9. **Accessibility**: Full WCAG 2.1 AA compliance
10. **Responsiveness**: Seamless experience across all device sizes
11. **Data Integration**: Seamless database data display and interaction
12. **Migration Complete**: All existing hovercards follow standardized pattern with no legacy code
