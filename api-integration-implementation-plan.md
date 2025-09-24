# API Integration Implementation Plan: Migration from Static JSON to Supabase APIs

## Component Usage Validation
**✅ COMPONENT SOURCE CHECK:**
- [ ] Uses only components from `components/` folder
- [ ] Uses only components from `components/ui-elements/` subfolder
- [ ] **NO direct usage of `components/ui/` folder components**

## Overview
Migrate the entire UI from static JSON file-based data loading to Supabase's auto-generated APIs. This includes updating all pages, detail pages, buttons, actions, and data loading utilities to use real-time database operations instead of static file fetching.

## Migration Scope

### Pages Requiring Migration
1. **Dashboard Page** (`/dashboard`) - Real-time metrics and charts
2. **Data Pages** (`/data`, `/data/[objectType]`) - Dynamic data tables
3. **Sources Pages** (`/sources`, `/sources/[id]`) - Source management
4. **Activities Pages** (`/activities`, `/activities/[id]`) - Activity management  
5. **Workflows Pages** (`/workflows`, `/workflows/[id]`) - Workflow management
6. **Account Page** (`/account`) - User settings and preferences

### Components Requiring Migration
1. **Data Tables** - All table components with CRUD operations
2. **Dialogs** - Create, edit, and deploy dialogs
3. **Forms** - All form components with data persistence
4. **Cards** - Data display cards with real-time updates
5. **Hovercards** - Interactive data previews
6. **Buttons** - All action buttons with data operations

### Data Loading Utilities Requiring Migration
1. **Dialog Utils** - `loadDialogData()` function
2. **Local Storage** - Data persistence utilities
3. **Recent Items** - Navigation history tracking
4. **JSON Utils** - Data transformation utilities

## Phase 1: Core Infrastructure Setup

### 1.1 Supabase Client Configuration
**Files to Create:**
- `lib/supabase-client.ts` - Supabase client initialization
- `lib/api-client.ts` - Wrapper for Supabase operations
- `hooks/use-supabase-api.ts` - React hooks for data operations

**Configuration Requirements:**
- Environment variables setup (SUPABASE_URL, SUPABASE_ANON_KEY)
- Authentication context provider
- Row Level Security (RLS) policies
- Real-time subscriptions setup

### 1.2 Data Type Definitions
**Files to Create:**
- `types/supabase.ts` - Auto-generated Supabase types
- `types/api.ts` - Custom API response types
- `types/database.ts` - Database schema types

**Type Requirements:**
- Source, Activity, Workflow, Factor, Datapoint types
- API response wrapper types
- Error handling types
- Pagination and filtering types

### 1.3 Authentication System
**Files to Create:**
- `contexts/auth-context.tsx` - Authentication state management
- `hooks/use-auth.ts` - Authentication hooks
- `components/auth/auth-guard.tsx` - Route protection

**Authentication Features:**
- User login/logout
- Session management
- Organization-based access control
- JWT token handling

## Phase 2: Page-Level Migrations

### 2.1 Dashboard Page Migration (`/dashboard`)
**Current Implementation:**
- Loads static JSON from `/demo-and-testing-data/dashboard/dashboard.json`
- Uses `dashboardData` and `dashboardFullData` imports
- Local storage for dashboard preferences

**Migration Requirements:**
- Replace static imports with `useDashboard()` hook
- Implement real-time metrics updates
- Add loading states for chart data
- Update local storage to sync with database preferences

**Files to Modify:**
- `app/dashboard/page.tsx` - Main dashboard component
- `components/charts/cost-chart.tsx` - Cost metrics
- `components/charts/usage-chart.tsx` - Usage metrics
- `components/tables/history-table.tsx` - Activity history

### 2.2 Data Pages Migration (`/data`, `/data/[objectType]`)
**Current Implementation:**
- Static imports: `companiesData`, `personsData`, `jobpostsData`
- Object type configuration with static data
- Export functionality for static data

**Migration Requirements:**
- Replace static imports with `useObjectTypes()` hook
- Implement dynamic data fetching for each object type
- Add real-time updates for data changes
- Update export functionality to work with database data

**Files to Modify:**
- `app/data/page.tsx` - Data overview page
- `app/data/[objectType]/page.tsx` - Object type detail page
- `components/cards/objecttype-card.tsx` - Object type cards
- `components/tables/data-table.tsx` - Data display table

### 2.3 Sources Pages Migration (`/sources`, `/sources/[id]`)
**Current Implementation:**
- Uses `loadDialogData('sources')` for data loading
- Static source configuration and validation
- Local storage for source data

**Migration Requirements:**
- Replace `loadDialogData` with `useSources()` and `useSource(id)` hooks
- Implement CRUD operations for sources
- Add real-time source status updates
- Update source validation to work with database constraints

**Files to Modify:**
- `app/sources/page.tsx` - Sources listing page
- `app/sources/[id]/page.tsx` - Source detail page
- `components/dialogs/create-source-dialog.tsx` - Source creation
- `components/forms/source-form.tsx` - Source editing forms

### 2.4 Activities Pages Migration (`/activities`, `/activities/[id]`)
**Current Implementation:**
- Uses `loadDialogData('activities')` for data loading
- Static activity configuration and deployment
- Local storage for activity data

**Migration Requirements:**
- Replace `loadDialogData` with `useActivities()` and `useActivity(id)` hooks
- Implement activity deployment to database
- Add real-time activity status monitoring
- Update activity validation and testing

**Files to Modify:**
- `app/activities/page.tsx` - Activities listing page
- `app/activities/[id]/page.tsx` - Activity detail page
- `components/dialogs/deploy-activity-dialog.tsx` - Activity deployment
- `components/cards/activity-card.tsx` - Activity display cards

### 2.5 Workflows Pages Migration (`/workflows`, `/workflows/[id]`)
**Current Implementation:**
- Uses `loadDialogData('workflows')` for data loading
- Static workflow configuration and phases
- Local storage for workflow data

**Migration Requirements:**
- Replace `loadDialogData` with `useWorkflows()` and `useWorkflow(id)` hooks
- Implement workflow execution tracking
- Add real-time workflow status updates
- Update workflow phase management

**Files to Modify:**
- `app/workflows/page.tsx` - Workflows listing page
- `app/workflows/[id]/page.tsx` - Workflow detail page
- `components/dialogs/deploy-workflow-dialog.tsx` - Workflow deployment
- `components/accordions/workflow-phase-accordion.tsx` - Phase management

## Phase 3: Component-Level Migrations

### 3.1 Data Table Components Migration
**Current Implementation:**
- Static data passed as props
- Local state management for sorting/filtering
- Mock action handlers (console.log)

**Migration Requirements:**
- Replace static data with database queries
- Implement real-time data updates
- Add proper CRUD action handlers
- Update pagination to work with database queries

**Files to Modify:**
- `components/tables/data-table/data-table.tsx` - Main data table
- `components/tables/data-table/data-table-footer.tsx` - Pagination and bulk actions
- `components/tables/history-table.tsx` - Activity history
- `components/tables/datapoint-table.tsx` - Datapoint display

**Action Handlers to Implement:**
- View: Navigate to detail page
- Edit: Open edit dialog with current data
- Delete: Confirm and delete from database
- Bulk Export: Export selected items
- Bulk Edit: Batch update operations
- Bulk Delete: Batch delete operations

### 3.2 Dialog Components Migration
**Current Implementation:**
- Uses `loadDialogData()` for dropdown options
- Static form validation
- Mock save operations

**Migration Requirements:**
- Replace `loadDialogData()` with database queries
- Implement real form submission to database
- Add proper error handling and validation
- Update success/error feedback

**Files to Modify:**
- `components/dialogs/multistep-dialog.tsx` - Multi-step forms
- `components/dialogs/create-source-dialog.tsx` - Source creation
- `components/dialogs/deploy-activity-dialog.tsx` - Activity deployment
- `components/dialogs/deploy-workflow-dialog.tsx` - Workflow deployment

**Form Actions to Implement:**
- Create: Insert new records to database
- Update: Update existing records
- Deploy: Execute deployment operations
- Test: Validate configurations
- Save Draft: Save incomplete forms

### 3.3 Form Components Migration
**Current Implementation:**
- Static form configurations
- Local state management
- Mock submission handlers

**Migration Requirements:**
- Connect forms to database operations
- Implement real-time validation
- Add auto-save functionality
- Update form submission flow

**Files to Modify:**
- `components/forms/dynamic-form.tsx` - Dynamic form builder
- `components/forms/inference-form.tsx` - API inference forms
- `components/forms/status-check-form.tsx` - Status checking
- `components/forms/blocks/auto-save-form.tsx` - Auto-save functionality

**Form Features to Implement:**
- Auto-save: Periodic saving to database
- Real-time validation: Database constraint checking
- Draft management: Save and restore incomplete forms
- Field dependencies: Dynamic field updates based on database state

### 3.4 Card Components Migration
**Current Implementation:**
- Static data display
- Mock action buttons
- Local state for interactions

**Migration Requirements:**
- Connect to real-time data sources
- Implement actual action handlers
- Add loading states for data updates
- Update status indicators

**Files to Modify:**
- `components/cards/activity-card.tsx` - Activity display
- `components/cards/dashboard-card.tsx` - Dashboard metrics
- `components/cards/objecttype-card.tsx` - Object type display
- `components/cards/base-card.tsx` - Base card functionality

**Card Actions to Implement:**
- Quick Actions: Fast operations (start, stop, restart)
- Status Updates: Real-time status monitoring
- Navigation: Link to detail pages
- Context Menus: Right-click actions

## Phase 4: Utility Functions Migration

### 4.1 Data Loading Utilities Migration
**Current Implementation:**
- `loadDialogData()` function with static JSON paths
- Hardcoded file paths for different data sources
- No error handling for network failures

**Migration Requirements:**
- Replace with Supabase query functions
- Implement proper error handling and retry logic
- Add loading states and caching
- Support for pagination and filtering

**Files to Modify:**
- `utils/dialog-utils.ts` - Main data loading utility
- `utils/api-utils.ts` - API helper functions
- `utils/data-transformation.ts` - Data format conversion

**New Functions to Create:**
- `loadSources()` - Load sources from database
- `loadActivities()` - Load activities from database
- `loadWorkflows()` - Load workflows from database
- `loadFactors()` - Load factors from database
- `loadDatapoints()` - Load datapoints from database

### 4.2 Local Storage Migration
**Current Implementation:**
- Saves data to browser localStorage
- No synchronization with database
- Version management for data formats

**Migration Requirements:**
- Sync localStorage with database
- Implement offline-first approach
- Add conflict resolution for data changes
- Maintain backward compatibility

**Files to Modify:**
- `utils/local-storage.ts` - Local storage management
- `utils/sync-utils.ts` - Database synchronization
- `utils/offline-utils.ts` - Offline data handling

**New Features to Implement:**
- Auto-sync: Periodic synchronization with database
- Conflict Resolution: Handle concurrent edits
- Offline Queue: Queue operations when offline
- Data Migration: Migrate existing localStorage data

### 4.3 Recent Items Migration
**Current Implementation:**
- Tracks navigation history in localStorage
- Simple array-based storage
- No persistence across sessions

**Migration Requirements:**
- Store recent items in database
- Sync across devices and sessions
- Add user-specific recent items
- Implement smart suggestions

**Files to Modify:**
- `utils/recent-items.ts` - Recent items management
- `components/layout/main-layout.tsx` - Navigation integration

**New Features to Implement:**
- User-specific recent items
- Cross-device synchronization
- Smart suggestions based on usage
- Recent items analytics

### 4.4 JSON Utilities Migration
**Current Implementation:**
- Static JSON processing utilities
- No real-time data transformation
- Limited validation capabilities

**Migration Requirements:**
- Connect to real-time data sources
- Implement dynamic data transformation
- Add comprehensive validation
- Support for complex data structures

**Files to Modify:**
- `utils/json/mapping-utils.tsx` - Data mapping utilities
- `utils/json/rendering-utils.tsx` - Data rendering utilities
- `utils/validation-utils.ts` - Data validation utilities

**New Features to Implement:**
- Real-time data transformation
- Dynamic schema validation
- Complex data structure support
- Performance optimization for large datasets

## Phase 5: Button and Action Migrations

### 5.1 Primary Action Buttons Migration
**Current Implementation:**
- Mock action handlers (console.log)
- No actual data persistence
- Static button states

**Migration Requirements:**
- Implement real database operations
- Add proper loading states
- Update success/error feedback
- Handle edge cases and errors

**Files to Modify:**
- `components/ui-elements/buttons/primary-button.tsx` - Primary actions
- `components/ui-elements/buttons/secondary-button.tsx` - Secondary actions
- `components/forms/dynamic-form-actions.tsx` - Form actions
- `components/pages/page-header.tsx` - Page-level actions

**Actions to Implement:**
- Save: Persist data to database
- Create: Insert new records
- Update: Modify existing records
- Delete: Remove records with confirmation
- Deploy: Execute deployment operations
- Test: Validate configurations
- Export: Generate and download data

### 5.2 Table Action Buttons Migration
**Current Implementation:**
- Mock row actions (console.log)
- No bulk operation handling
- Static action menus

**Migration Requirements:**
- Implement real CRUD operations
- Add bulk action support
- Update action availability based on permissions
- Add confirmation dialogs for destructive actions

**Files to Modify:**
- `components/tables/data-table/data-table.tsx` - Row actions
- `components/tables/data-table/data-table-footer.tsx` - Bulk actions
- `components/ui-elements/buttons/options-button.tsx` - Action menus

**Table Actions to Implement:**
- View: Navigate to detail page
- Edit: Open edit dialog
- Delete: Remove with confirmation
- Copy: Duplicate records
- Export: Download selected data
- Bulk Edit: Batch update operations
- Bulk Delete: Batch remove operations

### 5.3 Dialog Action Buttons Migration
**Current Implementation:**
- Mock form submission
- No validation feedback
- Static button states

**Migration Requirements:**
- Implement real form submission
- Add validation and error handling
- Update button states based on form validity
- Add progress indicators for long operations

**Files to Modify:**
- `components/dialogs/multistep-dialog.tsx` - Multi-step actions
- `components/dialogs/create-source-dialog.tsx` - Creation actions
- `components/dialogs/deploy-activity-dialog.tsx` - Deployment actions
- `components/forms/dynamic-form.tsx` - Form submission

**Dialog Actions to Implement:**
- Next/Previous: Navigate between steps
- Submit: Complete form submission
- Cancel: Close without saving
- Save Draft: Save incomplete forms
- Deploy: Execute deployment
- Test: Validate configuration

### 5.4 Card Action Buttons Migration
**Current Implementation:**
- Mock quick actions
- No status updates
- Static action availability

**Migration Requirements:**
- Implement real quick actions
- Add real-time status updates
- Update action availability based on state
- Add loading indicators for actions

**Files to Modify:**
- `components/cards/activity-card.tsx` - Activity actions
- `components/cards/dashboard-card.tsx` - Dashboard actions
- `components/cards/objecttype-card.tsx` - Object type actions
- `components/hovercards/mapping-hovercard.tsx` - Mapping actions

**Card Actions to Implement:**
- Start/Stop: Control execution state
- Restart: Restart operations
- Configure: Open configuration
- Monitor: View real-time status
- Edit: Quick edit functionality
- Delete: Remove with confirmation

## Phase 6: Testing and Validation

### 6.1 Unit Testing Migration
**Current Implementation:**
- Tests for static data handling
- Mock data in tests
- No API integration testing

**Migration Requirements:**
- Update tests for database operations
- Add API mocking for tests
- Test error handling scenarios
- Add performance testing

**Files to Create/Modify:**
- `tests/api/api-client.test.ts` - API client testing
- `tests/hooks/use-supabase-api.test.ts` - Hook testing
- `tests/components/data-table.test.tsx` - Component testing
- `tests/utils/dialog-utils.test.ts` - Utility testing

### 6.2 Integration Testing
**Current Implementation:**
- No integration testing
- Static data scenarios only

**Migration Requirements:**
- End-to-end workflow testing
- Database integration testing
- Real-time update testing
- Error scenario testing

**Test Scenarios:**
- Complete CRUD workflows
- Real-time data synchronization
- Offline/online transitions
- Error handling and recovery
- Performance under load

### 6.3 User Acceptance Testing
**Current Implementation:**
- Manual testing with static data
- No user feedback collection

**Migration Requirements:**
- User workflow testing
- Performance validation
- Accessibility testing
- Cross-browser compatibility

**Testing Areas:**
- All page navigation flows
- Form submission workflows
- Data table operations
- Real-time updates
- Error handling
- Mobile responsiveness

## File Changes Required

### New Files to Create
**Core Infrastructure:**
- `lib/supabase-client.ts` - Supabase client configuration
- `lib/api-client.ts` - API wrapper functions
- `hooks/use-supabase-api.ts` - React hooks for data operations
- `contexts/auth-context.tsx` - Authentication state management
- `hooks/use-auth.ts` - Authentication hooks
- `components/auth/auth-guard.tsx` - Route protection

**Type Definitions:**
- `types/supabase.ts` - Auto-generated Supabase types
- `types/api.ts` - Custom API response types
- `types/database.ts` - Database schema types

**Utility Functions:**
- `utils/sync-utils.ts` - Database synchronization
- `utils/offline-utils.ts` - Offline data handling
- `utils/data-transformation.ts` - Data format conversion
- `utils/validation-utils.ts` - Data validation utilities

**Testing Files:**
- `tests/api/api-client.test.ts` - API client testing
- `tests/hooks/use-supabase-api.test.ts` - Hook testing
- `tests/components/data-table.test.tsx` - Component testing
- `tests/utils/dialog-utils.test.ts` - Utility testing

### Files to Modify
**Pages (6 files):**
- `app/dashboard/page.tsx` - Dashboard with real-time data
- `app/data/page.tsx` - Data overview with database queries
- `app/data/[objectType]/page.tsx` - Object type details
- `app/sources/page.tsx` - Sources listing
- `app/sources/[id]/page.tsx` - Source details
- `app/activities/page.tsx` - Activities listing
- `app/activities/[id]/page.tsx` - Activity details
- `app/workflows/page.tsx` - Workflows listing
- `app/workflows/[id]/page.tsx` - Workflow details
- `app/account/page.tsx` - Account settings

**Components (25+ files):**
- `components/tables/data-table/data-table.tsx` - Main data table
- `components/tables/data-table/data-table-footer.tsx` - Pagination and bulk actions
- `components/dialogs/multistep-dialog.tsx` - Multi-step forms
- `components/dialogs/create-source-dialog.tsx` - Source creation
- `components/dialogs/deploy-activity-dialog.tsx` - Activity deployment
- `components/dialogs/deploy-workflow-dialog.tsx` - Workflow deployment
- `components/forms/dynamic-form.tsx` - Dynamic form builder
- `components/forms/inference-form.tsx` - API inference forms
- `components/forms/status-check-form.tsx` - Status checking
- `components/cards/activity-card.tsx` - Activity display
- `components/cards/dashboard-card.tsx` - Dashboard metrics
- `components/cards/objecttype-card.tsx` - Object type display
- `components/hovercards/mapping-hovercard.tsx` - Mapping actions
- `components/ui-elements/buttons/primary-button.tsx` - Primary actions
- `components/ui-elements/buttons/secondary-button.tsx` - Secondary actions
- `components/forms/dynamic-form-actions.tsx` - Form actions
- `components/pages/page-header.tsx` - Page-level actions
- `components/layout/main-layout.tsx` - Navigation integration

**Utilities (8 files):**
- `utils/dialog-utils.ts` - Data loading utilities
- `utils/local-storage.ts` - Local storage management
- `utils/recent-items.ts` - Recent items management
- `utils/json/mapping-utils.tsx` - Data mapping utilities
- `utils/json/rendering-utils.tsx` - Data rendering utilities
- `utils/api-utils.ts` - API helper functions
- `utils/types.ts` - Type definitions
- `utils/animations.ts` - Animation utilities

## Migration Checklist

### Phase 1: Infrastructure Setup
- [ ] Set up Supabase client configuration
- [ ] Install required dependencies (@supabase/supabase-js)
- [ ] Create authentication context and hooks
- [ ] Set up environment variables
- [ ] Create API client wrapper functions
- [ ] Generate TypeScript types from database schema

### Phase 2: Page Migrations
- [ ] Migrate dashboard page to real-time data
- [ ] Migrate data pages to database queries
- [ ] Migrate sources pages with CRUD operations
- [ ] Migrate activities pages with deployment
- [ ] Migrate workflows pages with execution tracking
- [ ] Update account page with user preferences

### Phase 3: Component Migrations
- [ ] Update data tables with real CRUD operations
- [ ] Migrate dialogs to database operations
- [ ] Update forms with real-time validation
- [ ] Migrate cards with live status updates
- [ ] Update buttons with actual action handlers
- [ ] Migrate hovercards with real-time data

### Phase 4: Utility Migrations
- [ ] Replace loadDialogData with database queries
- [ ] Update local storage with database sync
- [ ] Migrate recent items to database storage
- [ ] Update JSON utilities for real-time data
- [ ] Create data transformation utilities
- [ ] Implement validation utilities

### Phase 5: Action Migrations
- [ ] Implement primary action buttons
- [ ] Update table action buttons
- [ ] Migrate dialog action buttons
- [ ] Update card action buttons
- [ ] Add bulk operation support
- [ ] Implement confirmation dialogs

### Phase 6: Testing and Validation
- [ ] Update unit tests for database operations
- [ ] Add integration testing
- [ ] Implement user acceptance testing
- [ ] Test error handling scenarios
- [ ] Validate performance improvements
- [ ] Test offline functionality

## Success Criteria

1. **Complete Migration**: All static JSON usage replaced with Supabase APIs
2. **Real-time Updates**: Live data synchronization across all components
3. **CRUD Operations**: Full create, read, update, delete functionality
4. **Error Handling**: Comprehensive error handling and user feedback
5. **Performance**: Improved loading times and reduced bundle size
6. **User Experience**: Seamless transition with no functionality loss
7. **Testing**: Complete test coverage for all migrated functionality
8. **Documentation**: Updated documentation reflecting new API usage

## Risk Mitigation

### Data Loss Prevention
- Implement data migration scripts
- Create backup procedures
- Test migration with production data copies
- Implement rollback procedures

### Performance Considerations
- Implement caching strategies
- Optimize database queries
- Add loading states for better UX
- Monitor API response times

### User Experience
- Maintain existing UI/UX patterns
- Add progressive loading indicators
- Implement offline fallbacks
- Provide clear error messages

This comprehensive migration plan ensures a systematic approach to moving from static JSON files to a fully functional Supabase-based application with real-time capabilities, proper error handling, and improved user experience.
