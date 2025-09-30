# UI Tests Implementation Plan

## Component Usage Validation
**✅ COMPONENT SOURCE CHECK:**
- [ ] Uses only components from `components/` folder
- [ ] Uses only components from `components/ui-elements/` subfolder
- [ ] **NO direct usage of `components/ui/` folder components**

## Overview
Implement a comprehensive testing strategy for all UI components using Playwright for E2E testing, Jest for unit testing, React Testing Library for component testing, and MSW for API mocking to ensure reliability, accessibility, and performance across all application sections.

## Current Codebase Analysis

### **Component Architecture (200+ Components)**
- **UI Elements**: 50+ foundation components organized in 5 subcategories
  - **Badges**: 5 components (BaseBadge, ActivityBadge, StatusBadge, PrivateBetaBadge, Badge)
  - **Buttons**: 7 components (PrimaryButton, SecondaryButton, TextButton, SelectButton, OptionsButton, Button, ButtonGroup)
  - **Inputs**: 6 components (FormInput, FormTextarea, FormSelect, NumberInput, RadioGroup, Input)
  - **Text**: 4 components (Headline, Subheadline, Description, Text)
  - **Others**: AccordionGrid, Skeleton, ShareComponent, Divider, EmptyState, LastUsed, ProgressBar, SearchBar, Separator
- **Forms**: 13 form components with validation and dynamic behavior
  - **Core Forms**: FormField, FormSection, FormDragDrop
  - **Form Types**: SimpleForm, MultiFieldForm, DynamicForm, AutoSaveForm
  - **Specialized Forms**: SourceStatusCheckForm, SourceDeliveryEndpointForm, CustomizationOptionsForm, SourceInferenceForm, LoadBalancingForm
- **Cards**: 11 card components for information display
  - **Base Cards**: BaseCard, DashboardCard, CreateCard, AddActivityCard, AddSetupCard
  - **Content Cards**: ActivityCard, SourceCard, WorkflowCard, UsecaseCard, GuardrailCard, ObjectTypeCard
- **Tables**: 13 table components with comprehensive data management
  - **Core Tables**: BaseTable, DataTable, DataTableHeader, DataTableFooter, RowSelector
  - **Data Tables**: DatapointsTable, ObjectsTable, ValueTable, DatapointTable
  - **Specialized Tables**: PricingTable, PaymentOptionsTable, InvoiceTable, CollaborationTable, HistoryTable, SourceRecapitulationTable
- **Accordions**: 7 accordion components for collapsible content
  - **Core Accordions**: BaseAccordion, JsonAccordion
  - **Specialized Accordions**: SourceGuardrailAccordion, SourceRequestAccordion, SourceUsecasesAccordion, WorkflowPhaseAccordion, ActivitySourceAccordion
- **Selectors**: 11 selector components for data selection
  - **Core Selectors**: BaseSelector, SourceSelector, ObjectTypeSelector, WorkflowPhaseSelector
  - **Data Selectors**: DatapointSelector, UsecaseSelector, GuardrailSelector
  - **Configuration Selectors**: OrgSelector, RequestHeaderSelector, SourceVariableSelector, DatapointVariableSelector
- **ReactFlow**: 11 workflow visualization components
  - **Nodes**: BaseNode, SourceNode, TargetNode, DatapointNode, FactorNode, PlaceholderNode
  - **Edges**: BaseEdge, ActiveEdge, ErrorEdge
  - **Interactions**: ButtonNodeHandle, CanvasToolbar
- **Layout**: 13 layout and navigation components
  - **Core Layout**: MainLayout, Layout, Header, ContentWrapper, PageContent, PageControls, Grid
  - **Navigation**: NavMain, NavSecondary, NavRecent, NavUser, OrganizationSwitcher
- **JSON**: 11 JSON processing and visualization components
  - **Core JSON**: JsonRenderer, JsonNode, JsonNodeStatus, JsonExplorer
  - **Mappers**: JsonResponseMapper, JsonRequestMapper, JsonResultMapper
  - **Interface**: JsonSelector, JsonPath, ManageJsonsDialog, PathsExpandSelector
- **Dialogs**: 25 modal dialog components
  - **Setup Dialogs**: SetupDialog, CollaborationDialog, LoginDialog, LogoutDialog, SignUpDialog
  - **Creation Dialogs**: CreateGuardrailFactorDialog, CreateUsecaseFactorDialog, SetupGuardrailsDialog
  - **Deployment Dialogs**: DeployActivityDialog, DeploySourceDialog, DeployWorkflowDialog
  - **Management Dialogs**: AddSourceDialog, ConfirmDialog, ConfirmationDialog, CreateOrganizationDialog, CreateSourceDialog, AddSetupDialog
  - **Utility Dialogs**: StatusTranslationDialog, UpsellDialog, ExploreValueDialog, MultistepDialog, OrganizationSelectorDialog, SelectOrganizationDialog
- **Charts**: 3 data visualization components (UsageChart, CostChart, Chart)
- **Hovercards**: 7 hover card components
  - **Core Hovercards**: BaseHovercard, CustomHovercardExample, HovercardIntegrationTest
  - **Specialized Hovercards**: MappingHovercard, ValueHovercard, ProgressHovercard, PrivateBetaHovercard
- **Tabs**: 2 tab navigation components (SourceTabs, Tabs)
- **Navigation**: 1 navigation component (Navigation)
- **Others**: 15 utility components (AccordionGrid, Skeleton, ShareComponent, Divider, EmptyState, LastUsed, ProgressBar, SearchBar, Separator, OrganizationAvatar, UserAvatar, PathSearch, NavChat, StatsOverview)

### **Data Architecture & Integration**
- **Demo Data Repository**: Comprehensive testing data (309+ files)
  - **Sample Data**: Companies, persons, job posts, organizations, documents
  - **Source Data**: 283+ JSON configurations, 24+ JS files
  - **Activity Data**: 12+ activity configurations and metadata
  - **Workflow Data**: 5+ workflow configurations and execution data
  - **Legacy Data**: Version 1.0 data for reference and migration testing
- **Data Types System**: TypeScript interfaces for all data structures
- **Data Table Components**: Advanced data display with auto-generation, filtering, sorting
- **Selector System**: Data selection components with adapters and validation
- **Data Hooks**: Custom hooks for data management and state synchronization

### **Page Structure (9 Pages)**
- **Dashboard**: Main application dashboard (`/dashboard`)
- **Activities**: Activity tracking and management 
  - **Activities List**: `/activities` - Activity listing and management
  - **Activity Detail**: `/activities/[id]` - Individual activity details and configuration
- **Sources**: Data source management
  - **Sources List**: `/sources` - Source listing and management
  - **Source Detail**: `/sources/[id]` - Individual source configuration and testing
- **Workflows**: Workflow configuration
  - **Workflows List**: `/workflows` - Workflow listing and management
  - **Workflow Detail**: `/workflows/[id]` - Individual workflow design and execution
- **Data**: Data operations and management
  - **Data Overview**: `/data` - Object type navigation and data statistics
  - **Data Detail**: `/data/[objectType]` - Object-specific data display and management
- **Settings**: Application settings (`/settings`)
- **Account**: User account management (`/account`)
- **Components Showcase**: Interactive component examples (`/components-showcase`)

### **Existing Testing Infrastructure**
- **Jest Configuration**: Already configured with Next.js integration
- **React Testing Library**: Set up with @testing-library/jest-dom
- **Test Structure**: Basic test files exist for JSON components
- **Test Scripts**: Shell script for running specific test suites

## Phase 1: Architecture & Core Structure

### 1.1 Testing System Architecture
- **E2E Testing**: Playwright-based end-to-end testing framework
- **Unit Testing**: Jest-based unit and integration testing
- **Component Testing**: React Testing Library for component testing
- **API Mocking**: MSW for realistic API simulation

### 1.2 Core Testing Structure
- **Test Suites**: Organized test organization by feature area
- **Test Utilities**: Reusable testing helpers and utilities
- **Mock Data**: Comprehensive mock data for all scenarios
- **Test Configuration**: Environment-specific test configurations

### 1.3 Data Flow Design
- **Test Data**: Mock data generation and management
- **Test Execution**: Parallel and sequential test execution
- **Result Reporting**: Comprehensive test result reporting
- **Continuous Integration**: Automated testing in CI/CD pipeline

## Phase 2: Components & Features

### 2.1 Playwright E2E Testing

#### **Page-Level E2E Tests**
- **Dashboard Page**: Data visualization, stats cards, navigation flows, metrics display
- **Activities Pages**: Activity listing, filtering, detail views, status updates, execution monitoring
- **Sources Pages**: Source management, configuration, testing, deployment, data validation
- **Workflows Pages**: Workflow creation, editing, visualization, execution, phase management
- **Data Pages**: 
  - **Data Overview**: Object type navigation, card interactions, data statistics
  - **Data Detail Pages**: Table rendering, filtering, sorting, export functionality, bulk actions
  - **Object Type Navigation**: Companies, persons, job posts, organizations, documents
- **Settings Page**: Configuration changes, user preferences, system settings
- **Account Page**: Profile management, authentication flows, user data

#### **Cross-Page User Journeys**
- **Complete Workflow Setup**: Create source → Configure → Test → Deploy → Monitor
- **Data Exploration Flow**: Browse data → Filter → Select → Map → Process
- **Data Management Flow**: Import data → Validate → Transform → Export → Archive
- **User Onboarding**: Account setup → First source → Basic workflow → Dashboard
- **Data Analysis Workflow**: Select object type → Apply filters → Analyze patterns → Generate insights

#### **Browser & Device Testing**
- **Cross-Browser**: Chrome, Firefox, Safari, Edge
- **Responsive Design**: Mobile (320px), Tablet (768px), Desktop (1920px)
- **Touch Interactions**: Mobile gestures, swipe navigation
- **Performance**: Page load times, interaction responsiveness

### 2.2 Jest Unit Testing

#### **Component Categories Testing**

**UI Elements (50+ components)**
- **Badges**: BaseBadge, ActivityBadge, StatusBadge, PrivateBetaBadge, Badge
- **Buttons**: PrimaryButton, SecondaryButton, TextButton, SelectButton, OptionsButton, Button, ButtonGroup
- **Inputs**: FormInput, FormTextarea, FormSelect, NumberInput, RadioGroup, Input
- **Text**: Headline, Subheadline, Description, Text
- **Others**: AccordionGrid, Skeleton, ShareComponent, Divider, EmptyState, LastUsed, ProgressBar, SearchBar, Separator

**Forms (13 components)**
- **Core Forms**: FormField, FormSection, FormDragDrop
- **Form Types**: SimpleForm, MultiFieldForm, DynamicForm, AutoSaveForm
- **Specialized Forms**: SourceStatusCheckForm, SourceDeliveryEndpointForm, CustomizationOptionsForm, SourceInferenceForm, LoadBalancingForm
- **Validation**: Form validation, error handling, submission logic

**Cards (11 components)**
- **Base Cards**: BaseCard, DashboardCard, CreateCard, AddActivityCard, AddSetupCard
- **Content Cards**: ActivityCard, SourceCard, WorkflowCard, UsecaseCard, GuardrailCard, ObjectTypeCard
- **Display Logic**: Card rendering, data binding, interaction handling

**Tables (13 components)**
- **Core Tables**: BaseTable, DataTable, DataTableHeader, DataTableFooter, RowSelector
- **Data Tables**: DatapointsTable, ObjectsTable, ValueTable, DatapointTable
- **Specialized Tables**: PricingTable, PaymentOptionsTable, InvoiceTable, CollaborationTable, HistoryTable, SourceRecapitulationTable
- **Table Features**: Virtual scrolling, sorting, filtering, row selection, auto-column generation
- **Table Renderers**: Text, Tags, Status, Percentage, Number, Date, Currency, Badge renderers

**ReactFlow Components (11 components)**
- **Nodes**: BaseNode, SourceNode, TargetNode, DatapointNode, FactorNode, PlaceholderNode
- **Edges**: BaseEdge, ActiveEdge, ErrorEdge
- **Interactions**: ButtonNodeHandle, CanvasToolbar

**JSON Components (11 components)**
- **Core JSON**: JsonRenderer, JsonNode, JsonNodeStatus, JsonExplorer
- **Mappers**: JsonResponseMapper, JsonRequestMapper, JsonResultMapper
- **Interface**: JsonSelector, JsonPath, ManageJsonsDialog, PathsExpandSelector

### 2.3 React Testing Library

#### **Component Interaction Testing**
- **Form Interactions**: Input validation, field updates, submission handling
- **Table Interactions**: Row selection, sorting, filtering, pagination
- **Dialog Interactions**: Modal opening/closing, form submission, cancellation
- **Navigation Interactions**: Menu navigation, breadcrumb navigation, tab switching
- **JSON Explorer Interactions**: Path selection, expansion, mapping operations

#### **State Management Testing**
- **Component State**: Local state updates, prop changes, lifecycle events
- **Form State**: Field values, validation states, submission states
- **Table State**: Selection state, sort state, filter state
- **Dialog State**: Open/closed states, form data persistence

### 2.4 MSW API Mocking

#### **API Endpoint Mocking**
- **Data Sources API**: Source CRUD operations, configuration, testing
- **Activities API**: Activity listing, status updates, execution monitoring
- **Workflows API**: Workflow CRUD, execution, status management
- **Data API**: Data retrieval, filtering, object type management, export functionality
- **User API**: Authentication, profile management, preferences
- **Demo Data API**: Sample data endpoints, mock data generation, data validation

#### **Network Condition Testing**
- **Slow Networks**: 3G simulation, timeout handling
- **Network Failures**: Connection loss, server errors, retry logic
- **API Errors**: 400, 401, 403, 404, 500 error handling

### 2.5 Data Testing & Validation

#### **Data Structure Testing**
- **Sample Data Validation**: Companies, persons, job posts, organizations, documents
- **Source Data Testing**: 283+ JSON configurations, data integrity validation
- **Activity Data Testing**: 12+ activity configurations, metadata validation
- **Workflow Data Testing**: 5+ workflow configurations, execution data validation
- **Legacy Data Testing**: Version 1.0 data compatibility and migration testing

#### **Data Table Testing**
- **Auto-Column Generation**: Dynamic column creation from data structure
- **Data Type Detection**: Automatic render type detection (text, number, date, currency, etc.)
- **Filtering & Sorting**: Column-based filtering, multi-column sorting, search functionality
- **Virtual Scrolling**: Large dataset performance, memory optimization
- **Row Selection**: Single/multi-select, bulk operations, selection persistence
- **Export Functionality**: JSON export, CSV export, data formatting

#### **Data Selector Testing**
- **Selector Adapters**: RequestHeader, Source, Usecase, Generic adapters
- **Data Transformation**: Data structure transformation, validation, error handling
- **Multi-Selection**: Multiple item selection, selection limits, validation
- **Search & Filtering**: Real-time search, tag-based filtering, group filtering
- **Data Persistence**: Selection persistence, storage key management

#### **Data Hook Testing**
- **useSelectorData Hook**: Data processing, filtering, sorting, grouping
- **useTableState Hook**: Table state management, pagination, selection state
- **Data Validation**: Input validation, type checking, error handling
- **Performance Testing**: Large dataset handling, memory usage, render optimization

## Phase 3: Interactions & User Experience

### 3.1 Test Scenarios
- **Happy Path**: Successful user journey testing
- **Error Paths**: Error handling and recovery testing
- **Edge Cases**: Boundary condition and edge case testing
- **Accessibility**: Screen reader and keyboard navigation testing

### 3.2 Test Data Management
- **Mock Data Generation**: Automated mock data creation
- **Data Variants**: Different data scenarios and states
- **Data Cleanup**: Test data isolation and cleanup
- **Data Consistency**: Cross-test data consistency

### 3.3 Advanced Testing Features
- **Parallel Execution**: Concurrent test execution for speed
- **Test Retries**: Automatic retry for flaky tests
- **Performance Monitoring**: Test execution performance tracking
- **Coverage Reporting**: Code coverage analysis and reporting

## Phase 4: Responsive Design & Accessibility

### 4.1 Responsive Testing

#### **Breakpoint Testing**
- **Mobile**: 320px - 767px (Portrait/Landscape)
- **Tablet**: 768px - 1023px (Portrait/Landscape)
- **Desktop**: 1024px - 1919px
- **Large Desktop**: 1920px+

#### **Component Responsive Behavior**
- **Layout Components**: Header, Navigation, Sidebar responsiveness
- **Table Components**: Horizontal scrolling, column hiding, mobile optimization
- **Form Components**: Field stacking, button layout, input sizing
- **Card Components**: Grid layout, card sizing, content overflow
- **Dialog Components**: Modal sizing, positioning, mobile adaptation

#### **Touch Interaction Testing**
- **Mobile Gestures**: Swipe, pinch, tap, long press
- **Touch Targets**: Minimum 44px touch target size validation
- **Scroll Behavior**: Smooth scrolling, scroll boundaries
- **Orientation Changes**: Layout adaptation, state preservation

### 4.2 Accessibility Testing

#### **WCAG 2.1 AA Compliance**
- **Color Contrast**: 4.5:1 ratio for normal text, 3:1 for large text
- **Focus Management**: Visible focus indicators, logical tab order
- **Screen Reader**: ARIA labels, roles, descriptions, live regions
- **Keyboard Navigation**: Full functionality via keyboard only

#### **Component-Specific Accessibility**
- **Form Components**: Label associations, error announcements, field validation
- **Table Components**: Column headers, row/column navigation, sort indicators
- **Dialog Components**: Focus trapping, escape key handling, ARIA modal
- **Navigation Components**: Skip links, landmark regions, current page indication
- **JSON Explorer**: Keyboard navigation, ARIA tree structure, status announcements

#### **Accessibility Testing Tools**
- **Automated**: axe-core, Lighthouse accessibility audit
- **Manual**: Screen reader testing (NVDA, JAWS, VoiceOver)
- **Keyboard**: Tab navigation, arrow keys, Enter/Space activation
- **Visual**: High contrast mode, zoom testing (up to 200%)

### 4.3 Performance Testing

#### **Core Web Vitals**
- **Largest Contentful Paint (LCP)**: < 2.5 seconds
- **First Input Delay (FID)**: < 100 milliseconds
- **Cumulative Layout Shift (CLS)**: < 0.1

#### **Page Performance Metrics**
- **Time to First Byte (TTFB)**: < 600ms
- **First Contentful Paint (FCP)**: < 1.8 seconds
- **Time to Interactive (TTI)**: < 3.8 seconds

#### **Component Performance**
- **Large Data Tables**: Virtual scrolling performance, render optimization
- **JSON Explorer**: Large JSON file handling, expansion performance
- **ReactFlow Components**: Canvas rendering, node interaction performance
- **Form Components**: Validation performance, submission handling
- **Chart Components**: Data visualization rendering, interaction responsiveness

## Phase 5: Advanced Features & Integration

### 5.1 Testing Intelligence
- **Smart Test Generation**: Automated test case generation
- **Test Prioritization**: Intelligent test execution ordering
- **Flaky Test Detection**: Automatic flaky test identification
- **Test Impact Analysis**: Change impact on test coverage

### 5.2 Integration Features
- **CI/CD Integration**: Automated testing in deployment pipeline
- **Test Reporting**: Comprehensive test result reporting
- **Test Analytics**: Test execution analytics and insights
- **Test Maintenance**: Automated test maintenance and updates

### 5.3 Testing & Validation
- **Test Quality**: Test reliability and maintainability
- **Test Coverage**: Comprehensive feature and code coverage
- **Test Performance**: Fast and efficient test execution
- **Test Documentation**: Clear test documentation and examples

## File Changes Required

### New Files

#### **E2E Testing Infrastructure**
- `tests/e2e/playwright.config.ts` - Playwright configuration
- `tests/e2e/pages/dashboard.spec.ts` - Dashboard page E2E tests
- `tests/e2e/pages/activities.spec.ts` - Activities pages E2E tests
- `tests/e2e/pages/sources.spec.ts` - Sources pages E2E tests
- `tests/e2e/pages/workflows.spec.ts` - Workflows pages E2E tests
- `tests/e2e/pages/data.spec.ts` - Data pages E2E tests
- `tests/e2e/flows/user-journey.spec.ts` - Complete user journey tests
- `tests/e2e/flows/workflow-setup.spec.ts` - Workflow setup flow tests
- `tests/e2e/utils/test-helpers.ts` - E2E test utilities

#### **Component Testing Infrastructure**
- `tests/components/setup.tsx` - Component test setup
- `tests/components/ui-elements/` - UI elements component tests
- `tests/components/forms/` - Form components tests
- `tests/components/cards/` - Card components tests
- `tests/components/tables/` - Table components tests
- `tests/components/accordions/` - Accordion components tests
- `tests/components/selectors/` - Selector components tests
- `tests/components/reactflow/` - ReactFlow components tests
- `tests/components/json/` - JSON components tests
- `tests/components/dialogs/` - Dialog components tests
- `tests/components/layout/` - Layout components tests

#### **Unit Testing Infrastructure**
- `tests/unit/utils/` - Utility function tests
- `tests/unit/hooks/` - Custom hook tests
- `tests/unit/lib/` - Library function tests

#### **Mock Data & Utilities**
- `tests/mocks/handlers.ts` - MSW API handlers
- `tests/mocks/browser.ts` - Browser API mocks
- `tests/fixtures/test-data.ts` - Comprehensive test data
- `tests/fixtures/components/` - Component-specific test data
- `tests/fixtures/demo-data/` - Demo data repository integration
- `tests/fixtures/sample-data/` - Sample data for testing (companies, persons, job posts)
- `tests/fixtures/source-data/` - Source configuration test data
- `tests/fixtures/activity-data/` - Activity configuration test data
- `tests/fixtures/workflow-data/` - Workflow configuration test data
- `tests/utils/test-utils.tsx` - Testing utilities and helpers
- `tests/utils/accessibility.ts` - Accessibility testing helpers
- `tests/utils/performance.ts` - Performance testing utilities
- `tests/utils/data-utils.ts` - Data testing utilities and helpers

#### **Configuration Files**
- `tests/config/jest.config.js` - Enhanced Jest configuration
- `tests/config/playwright.config.ts` - Playwright configuration
- `tests/config/msw.config.ts` - MSW configuration
- `.github/workflows/test.yml` - CI/CD testing workflow

### Modified Files
- `package.json` - Add testing dependencies and scripts
- `next.config.mjs` - Configure testing environment
- `tsconfig.json` - Include test file patterns
- `jest.config.js` - Enhanced configuration (already exists)
- `jest.setup.js` - Enhanced setup (already exists)

## Testing Checklist

### E2E Tests
- [ ] **Dashboard Page**: Stats cards, navigation, data visualization, metrics display
- [ ] **Activities Pages**: Listing, filtering, detail views, status updates, execution monitoring
- [ ] **Sources Pages**: CRUD operations, configuration, testing, deployment, data validation
- [ ] **Workflows Pages**: Creation, editing, visualization, execution, phase management
- [ ] **Data Pages**: 
  - [ ] **Data Overview**: Object type navigation, card interactions, data statistics
  - [ ] **Data Detail Pages**: Table rendering, filtering, sorting, export functionality, bulk actions
  - [ ] **Object Type Navigation**: Companies, persons, job posts, organizations, documents
- [ ] **Settings Page**: Configuration changes, user preferences, system settings
- [ ] **Account Page**: Profile management, authentication, user data
- [ ] **Cross-Page Flows**: Complete user journeys, workflow setup, data exploration
- [ ] **Cross-Browser Testing**: Chrome, Firefox, Safari, Edge
- [ ] **Responsive Design**: Mobile, tablet, desktop layouts
- [ ] **Performance Testing**: Page load times, interaction responsiveness

### Component Tests (200+ Components)

#### **UI Elements (50+ components)**
- [ ] **Badges**: BaseBadge, ActivityBadge, StatusBadge, PrivateBetaBadge, Badge
- [ ] **Buttons**: PrimaryButton, SecondaryButton, TextButton, SelectButton, OptionsButton, Button, ButtonGroup
- [ ] **Inputs**: FormInput, FormTextarea, FormSelect, NumberInput, RadioGroup, Input
- [ ] **Text**: Headline, Subheadline, Description, Text
- [ ] **Others**: AccordionGrid, Skeleton, ShareComponent, Divider, EmptyState, LastUsed, ProgressBar, SearchBar, Separator

#### **Forms (13 components)**
- [ ] **Core Forms**: FormField, FormSection, FormDragDrop
- [ ] **Form Types**: SimpleForm, MultiFieldForm, DynamicForm, AutoSaveForm
- [ ] **Specialized Forms**: SourceStatusCheckForm, SourceDeliveryEndpointForm, CustomizationOptionsForm, SourceInferenceForm, LoadBalancingForm
- [ ] **Validation**: Form validation, error handling, submission logic

#### **Cards (11 components)**
- [ ] **Base Cards**: BaseCard, DashboardCard, CreateCard, AddActivityCard, AddSetupCard
- [ ] **Content Cards**: ActivityCard, SourceCard, WorkflowCard, UsecaseCard, GuardrailCard, ObjectTypeCard
- [ ] **Display Logic**: Card rendering, data binding, interaction handling

#### **Tables (13 components)**
- [ ] **Core Tables**: BaseTable, DataTable, DataTableHeader, DataTableFooter, RowSelector
- [ ] **Data Tables**: DatapointsTable, ObjectsTable, ValueTable, DatapointTable
- [ ] **Specialized Tables**: PricingTable, PaymentOptionsTable, InvoiceTable, CollaborationTable, HistoryTable, SourceRecapitulationTable
- [ ] **Table Features**: Virtual scrolling, sorting, filtering, row selection, auto-column generation
- [ ] **Table Renderers**: Text, Tags, Status, Percentage, Number, Date, Currency, Badge renderers

#### **ReactFlow (11 components)**
- [ ] **Nodes**: BaseNode, SourceNode, TargetNode, DatapointNode, FactorNode, PlaceholderNode
- [ ] **Edges**: BaseEdge, ActiveEdge, ErrorEdge
- [ ] **Interactions**: ButtonNodeHandle, CanvasToolbar

#### **JSON Components (11 components)**
- [ ] **Core JSON**: JsonRenderer, JsonNode, JsonNodeStatus, JsonExplorer
- [ ] **Mappers**: JsonResponseMapper, JsonRequestMapper, JsonResultMapper
- [ ] **Interface**: JsonSelector, JsonPath, ManageJsonsDialog, PathsExpandSelector

#### **Layout & Navigation (13 components)**
- [ ] **Core Layout**: MainLayout, Layout, Header, ContentWrapper, PageContent, PageControls, Grid
- [ ] **Navigation**: NavMain, NavSecondary, NavRecent, NavUser, OrganizationSwitcher

#### **Dialogs (25 components)**
- [ ] **Setup Dialogs**: SetupDialog, CollaborationDialog, LoginDialog, LogoutDialog, SignUpDialog
- [ ] **Creation Dialogs**: CreateGuardrailFactorDialog, CreateUsecaseFactorDialog, SetupGuardrailsDialog
- [ ] **Deployment Dialogs**: DeployActivityDialog, DeploySourceDialog, DeployWorkflowDialog
- [ ] **Management Dialogs**: AddSourceDialog, ConfirmDialog, ConfirmationDialog, CreateOrganizationDialog, CreateSourceDialog, AddSetupDialog
- [ ] **Utility Dialogs**: StatusTranslationDialog, UpsellDialog, ExploreValueDialog, MultistepDialog, OrganizationSelectorDialog, SelectOrganizationDialog

### Unit Tests
- [ ] **Utility Functions**: JSON processing, form validation, layout utilities
- [ ] **Custom Hooks**: Form handling, state management, data fetching
- [ ] **Library Functions**: Data transformation, API helpers, validation logic
- [ ] **Error Handling**: Error boundaries, exception handling, fallback UI

### Data Testing
- [ ] **Sample Data Validation**: Companies, persons, job posts, organizations, documents
- [ ] **Source Data Testing**: 283+ JSON configurations, data integrity validation
- [ ] **Activity Data Testing**: 12+ activity configurations, metadata validation
- [ ] **Workflow Data Testing**: 5+ workflow configurations, execution data validation
- [ ] **Data Table Testing**: Auto-column generation, data type detection, filtering, sorting
- [ ] **Data Selector Testing**: Selector adapters, data transformation, multi-selection
- [ ] **Data Hook Testing**: useSelectorData, useTableState, data validation, performance
- [ ] **Export Functionality**: JSON export, CSV export, data formatting, download handling
- [ ] **Data Persistence**: Selection persistence, storage key management, state synchronization

### Integration Tests
- [ ] **Component Integration**: Multi-component interactions, data flow
- [ ] **API Integration**: Mock API responses, error handling, loading states
- [ ] **State Management**: Context providers, state synchronization
- [ ] **Form Integration**: Multi-step forms, validation chains, submission flows

### Accessibility Tests
- [ ] **WCAG 2.1 AA Compliance**: Color contrast, focus management, screen reader
- [ ] **Keyboard Navigation**: Tab order, arrow keys, Enter/Space activation
- [ ] **ARIA Implementation**: Labels, roles, descriptions, live regions
- [ ] **Screen Reader Testing**: NVDA, JAWS, VoiceOver compatibility

### Performance Tests
- [ ] **Core Web Vitals**: LCP < 2.5s, FID < 100ms, CLS < 0.1
- [ ] **Page Performance**: TTFB < 600ms, FCP < 1.8s, TTI < 3.8s
- [ ] **Component Performance**: Large data handling, virtual scrolling, canvas rendering
- [ ] **Memory Testing**: Memory leaks, performance degradation

## Success Criteria

1. **Coverage**: 90%+ code coverage across all 200+ components
2. **Performance**: All tests complete within acceptable time limits
   - Unit tests: < 30 seconds total
   - Component tests: < 2 minutes total
   - E2E tests: < 10 minutes total
3. **Reliability**: Stable test execution with minimal flaky tests (< 5% flaky rate)
4. **Accessibility**: Full WCAG 2.1 AA compliance validation for all components
5. **Responsiveness**: Comprehensive responsive design testing across all breakpoints
6. **Integration**: Seamless CI/CD pipeline integration with automated testing
7. **Cross-Browser**: 100% functionality across Chrome, Firefox, Safari, Edge
8. **Performance Metrics**: All Core Web Vitals meet Google's recommended thresholds
9. **User Journey Coverage**: Complete end-to-end testing of all critical user flows
10. **Component Quality**: All components pass accessibility, performance, and interaction tests

## Implementation Priority

### **Phase 1 (High Priority)**
- Core UI Elements (badges, buttons, inputs)
- Form components and validation
- Layout and navigation components
- Critical user journeys (dashboard, sources, workflows)

### **Phase 2 (Medium Priority)**
- Card and table components
- Dialog and modal components
- JSON processing components
- Advanced form components

### **Phase 3 (Lower Priority)**
- ReactFlow components
- Chart and visualization components
- Specialized selectors and accordions
- Performance optimization tests

## Dependencies Required

### **Testing Libraries**
```json
{
  "devDependencies": {
    "@playwright/test": "^1.40.0",
    "@testing-library/jest-dom": "^6.1.0",
    "@testing-library/react": "^14.0.0",
    "@testing-library/user-event": "^14.5.0",
    "msw": "^2.0.0",
    "jest-environment-jsdom": "^29.7.0",
    "axe-core": "^4.8.0",
    "@axe-core/react": "^4.8.0"
  }
}
```

### **Test Scripts**
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:accessibility": "jest --testPathPattern=accessibility",
    "test:performance": "jest --testPathPattern=performance",
    "test:data": "jest --testPathPattern=data",
    "test:components": "jest --testPathPattern=components",
    "test:unit": "jest --testPathPattern=unit",
    "test:integration": "jest --testPathPattern=integration",
    "test:data-tables": "jest --testPathPattern=data-table",
    "test:selectors": "jest --testPathPattern=selector",
    "test:demo-data": "jest --testPathPattern=demo-data"
  }
}
```
