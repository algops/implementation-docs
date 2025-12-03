# Local Storage Implementation Plan

## **Current Local Storage Infrastructure Analysis**

### **1. Existing LocalStorageManager System**

**Core Infrastructure:**
- **Singleton Pattern**: `LocalStorageManager` class with single instance access
- **Prefix System**: All keys prefixed with `"algops_"` for namespace isolation
- **Type Safety**: Generic `StorageItem<T>` interface with timestamp and version support
- **Error Handling**: Comprehensive try-catch blocks with console error logging
- **SSR Safety**: All operations check `typeof window === "undefined"` for server-side rendering

**Current Storage Functions:**
- **Generic Operations**: `setItem()`, `getItem()`, `removeItem()`, `getAllKeys()`, `clear()`
- **Data-Specific Functions**: 
  - `saveSourceData()` / `getSourceData()` - Source management
  - `saveWorkflowData()` / `getWorkflowData()` - Workflow management  
  - `saveActivityData()` / `getActivityData()` - Activity management
  - `saveDashboardData()` / `getDashboardData()` - Dashboard state

### **2. Recent Items System**

**Current Implementation:**
- **Direct localStorage Access**: Uses `localStorage.getItem("recentItems")` directly (not through LocalStorageManager)
- **Custom Event System**: Dispatches `"recentItemsUpdated"` events for real-time UI updates
- **Deduplication Logic**: Removes duplicate URLs, maintains chronological order
- **Size Limiting**: Maximum 3 recent items with automatic cleanup
- **Data Structure**: Simple array of `{name, url, icon?}` objects

**Usage Pattern:**
- **Navigation Tracking**: Automatically adds items when users visit pages
- **Breadcrumb Integration**: Powers navigation breadcrumbs and recent page access
- **Cross-Component Communication**: Uses custom events for real-time updates

### **3. Form Auto-Save System**

**Current Implementation:**
- **Auto-Save Hook**: `useAutoSave()` hook in `hooks/use-auto-save.ts`
- **Interval-Based Saving**: Configurable save intervals (default 5 seconds)
- **Form-Specific Keys**: Uses `form_${formId}` pattern for unique form identification
- **BeforeUnload Protection**: Saves data when user leaves page
- **Toast Notifications**: User feedback for save success/failure
- **Storage Integration**: Uses LocalStorageManager for consistent data handling

**Current Usage:**
- **Form Persistence**: Saves form data automatically during user input
- **Recovery System**: Loads saved data when user returns to form
- **Error Handling**: Graceful degradation with user notifications

### **4. Table State Persistence**

**Current Implementation:**
- **Direct localStorage Access**: Uses `localStorage.getItem("table-state-${tableId}")` directly
- **State Serialization**: Saves complete table state including filters, sorting, pagination
- **Automatic Loading**: Restores state on component mount
- **Configurable Persistence**: Optional feature controlled by `persistState` prop

## **Current Codebase Structure Analysis**

### **1. Page Structure Analysis**

**Main Application Pages:**
- **Dashboard**: `/app/dashboard/page.tsx` - Main dashboard with data visualization
- **Sources**: `/app/sources/page.tsx` and `/app/sources/[id]/page.tsx` - Source management
- **Workflows**: `/app/workflows/page.tsx` and `/app/workflows/[id]/page.tsx` - Workflow management
- **Activities**: `/app/activities/page.tsx` and `/app/activities/[id]/page.tsx` - Activity tracking
- **Data**: `/app/data/page.tsx` and `/app/data/[objectType]/page.tsx` - Data management
- **Settings**: `/app/settings/page.tsx` - Application settings
- **Account**: `/app/account/page.tsx` - User account management
- **Components Showcase**: `/app/components-showcase/page.tsx` - Component demonstration

**Data Flow Pattern:**
- **Static Data Loading**: Most pages use `require()` for static JSON data files
- **Dynamic State Management**: React state for user interactions and UI state
- **Local Storage Integration**: Limited use for specific features (recent items, form auto-save)

### **2. JSON Mappers Analysis**

**Current JSON Mapper Components:**
- **JsonResultMapper**: `/components/json/response/json-result-mapper.tsx` - Main data processing component
- **JsonRequestMapper**: `/components/json/request/json-request-mapper.tsx` - Request data mapping
- **JsonResponseMapper**: `/components/json/response/json-response-mapper.tsx` - Response data mapping
- **JsonInferenceRequestMapper**: `/components/json/request/json-inference-request-mapper.tsx` - AI inference mapping
- **JsonStatusCheckRequestMapper**: `/components/json/request/json-status-check-request-mapper.tsx` - Status check mapping
- **JsonDeliveryRequestMapper**: `/components/json/request/json-delivery-request-mapper.tsx` - Delivery mapping

**Data Processing Flow:**
- **Static Data Sources**: Uses `require('@/data/source-response.json')` and `require('@/data/source-mappings.json')`
- **Dynamic Processing**: Processes data through mapping utilities and rendering functions
- **State Management**: Maintains processing state and statistics
- **Component Integration**: Renders processed data through JsonExplorer and related components

**Current Limitations:**
- **No File Upload**: Cannot handle user-uploaded JSON files
- **No Persistence**: No localStorage integration for file management
- **Static Data Only**: Limited to hardcoded data files
- **No File Switching**: Cannot switch between different JSON sources
- **No Comprehensive Data Persistence**: Missing system-wide data loss prevention
- **No Local JSON Storage**: No mechanism to store all data locally before backend sync

### **3. Dialogs Analysis**

**Current Dialog Components:**
- **Creation Dialogs**: `create-organization-dialog.tsx`, `create-source-dialog.tsx`, `create-usecase-factor-dialog.tsx`
- **Management Dialogs**: `manage-jsons-dialog.tsx`, `organization-selector-dialog.tsx`
- **Deployment Dialogs**: `deploy-activity-dialog.tsx`, `deploy-source-dialog.tsx`, `deploy-workflow-dialog.tsx`
- **Utility Dialogs**: `confirm-dialog.tsx`, `explore-value-dialog.tsx`, `status-translation-dialog.tsx`

**State Management Patterns:**
- **Local State**: Most dialogs use `useState` for form data and UI state
- **No Persistence**: Dialog state is lost when dialog is closed
- **No Auto-Save**: No automatic saving of dialog content
- **No Recovery**: No way to restore unsaved dialog content

**Form Integration:**
- **Form Components**: Dialogs contain various form elements (inputs, selects, textareas)
- **Validation**: Some dialogs have form validation logic
- **Submission**: Forms submit data to parent components or external APIs
- **No Draft Saving**: No mechanism to save draft content

### **4. Selectors Analysis**

**Current Selector Components:**
- **Base Selector**: `base-selector.tsx` - Generic selector with popover and dialog variants
- **Data Selectors**: `datapoint-selector.tsx`, `objecttype-selector.tsx`, `source-selector.tsx`
- **Form Selectors**: `guardrail-selector.tsx`, `usecase-selector.tsx`, `workflow-phase-selector.tsx`
- **JSON Selectors**: `json-selector.tsx`, `paths-expand-selector.tsx` - New components for JSON management

**Storage Integration:**
- **Base Selector**: Has `storageKey` prop for localStorage persistence
- **Selection Persistence**: Saves selected values to localStorage when selector closes
- **No Auto-Save**: No automatic saving during selection process
- **Limited Scope**: Only saves final selection, not intermediate states

### **5. Forms Analysis**

**Current Form Components:**
- **Main Forms**: `customization-form.tsx`, `delivery-form.tsx`, `inference-form.tsx`, `settings-form.tsx`, `status-check-form.tsx`
- **Form Blocks**: `dynamic-form.tsx`, `form-drag-drop.tsx`, `form-field.tsx`, `form-section.tsx`
- **Specialized Forms**: `load-balancing-form.tsx`, `multi-field-form.tsx`, `request-header-form.tsx`, `simple-form.tsx`

**Auto-Save Implementation:**
- **Hook-Based**: `useAutoSave()` hook provides auto-save functionality
- **Configurable Intervals**: Adjustable save intervals for different form types
- **Error Handling**: Toast notifications for save success/failure
- **BeforeUnload Protection**: Saves data when user leaves page
- **Form-Specific Keys**: Each form gets unique localStorage key

**Current Limitations:**
- **Limited Scope**: Only main forms have auto-save, not dialog forms
- **No File Upload**: No integration with file upload functionality
- **No Draft Management**: No way to manage multiple drafts
- **No Recovery UI**: No user interface for recovering saved drafts

## **Comprehensive Data Persistence System (Phase 8 Enhancement)**

### **System-Wide Data Loss Prevention**

**Core Principle**: All components read from local JSON storage first, with backend sync as secondary operation.

**Data Flow Architecture**:
1. **Local-First Approach**: All data operations start with localStorage
2. **Backend Sync**: Data pushed to backend via buttons/edge functions
3. **Data Integrity**: Maintain consistency between local and remote data
4. **Recovery System**: Automatic recovery from localStorage when backend unavailable

**Storage Strategy**:
- **Primary Storage**: localStorage for all user data and configurations
- **Backup Storage**: Backend database for persistence and sharing
- **Sync Mechanism**: Manual triggers via buttons and edge functions
- **Conflict Resolution**: Local data takes precedence, with user confirmation for conflicts

### **Enhanced Data Folder Integration**

**Current Data Structure Enhancement**:
- **Local JSON Management**: Edit JSON files in localStorage instead of direct file access
- **Data Transfer Triggers**: Buttons to sync local changes to backend
- **Edge Function Integration**: Use edge functions for data transfer operations
- **Fallback System**: If localStorage solution not feasible, redesign data folder structure

**Implementation Options**:
1. **Option A - LocalStorage Integration**: 
   - Store all data in localStorage with JSON editing interface
   - Provide sync buttons to transfer to backend via edge functions
   - Maintain current data folder structure as reference

2. **Option B - Data Folder Redesign**:
   - Redesign current data folder to work with localStorage
   - Implement real-time JSON editing with automatic localStorage sync
   - Create unified data management system

### **Component Integration Requirements**

**All Components Must**:
- Read data exclusively from localStorage
- Save all changes to localStorage immediately
- Provide sync buttons for backend transfer
- Handle offline/online state gracefully
- Maintain data consistency across components

**Enhanced Components**:
- **Source Management**: All source data in localStorage with sync options
- **Activity Management**: Complete activity configurations in localStorage
- **Workflow Management**: Workflow definitions and executions in localStorage
- **Form Components**: All form data persisted to localStorage
- **Dialog Components**: Dialog state and content in localStorage
- **Mapper Components**: Mapping configurations and results in localStorage

## **Comprehensive Local Storage Feature Development Plan**

### **1. JSON File Management System**

**File Storage Structure:**
- **File Metadata**: Store file name, size, last modified date, properties count
- **File Content**: Store parsed JSON data as JavaScript objects
- **File Relationships**: Track which files are currently active/selected
- **File History**: Maintain upload history and file access patterns

**Storage Keys Pattern:**
- **Individual Files**: `json_file_${fileId}` - Complete file data with metadata
- **File List**: `json_files_list` - Array of all available file IDs
- **Current Source**: `current_json_source` - Currently active file ID
- **File History**: `json_files_history` - Chronological list of file access

**File Upload Process:**
- **File Selection**: User selects JSON file via file input or drag-and-drop
- **File Validation**: Parse and validate JSON structure
- **Metadata Calculation**: Count properties, calculate file size, generate timestamps
- **Storage**: Save file data to localStorage with unique ID
- **UI Update**: Dispatch custom events to update all related components
- **Integration**: Update JsonResultMapper to use stored file data

**File Management Features:**
- **File Switching**: Allow users to switch between different JSON files
- **File Removal**: Delete files with confirmation dialogs
- **File Renaming**: Allow users to rename files for better organization
- **File Duplication**: Create copies of existing files
- **File Export**: Download files back to user's device

### **2. Form Draft Management System**

**Draft Storage Structure:**
- **Form Drafts**: Store incomplete form data for recovery
- **Draft Metadata**: Track creation date, last modified, form type
- **Draft Relationships**: Link drafts to specific forms or dialogs
- **Draft Cleanup**: Automatic cleanup of old or completed drafts

**Storage Keys Pattern:**
- **Form Drafts**: `form_draft_${formId}_${draftId}` - Individual draft data
- **Draft Lists**: `form_drafts_${formId}` - List of all drafts for specific form
- **Active Drafts**: `active_form_drafts` - Currently open draft forms
- **Draft History**: `draft_history` - Chronological list of all draft activity

**Draft Management Features:**
- **Auto-Save**: Automatically save form data as user types
- **Draft Recovery**: Restore unsaved form data when user returns
- **Draft Comparison**: Show differences between current and saved versions
- **Draft Cleanup**: Remove old drafts automatically or manually
- **Draft Sharing**: Allow users to share draft content with others

**Integration Points:**
- **Dialog Forms**: All creation and management dialogs
- **Main Forms**: All primary application forms
- **Multi-Step Forms**: Complex forms with multiple steps
- **Form Validation**: Integrate with existing validation systems

### **3. Mapper Content Persistence System**

**Mapper Storage Structure:**
- **Mapping Data**: Store incomplete mapping configurations
- **Mapping Metadata**: Track mapping type, source data, target schema
- **Mapping State**: Store current mapping progress and user selections
- **Mapping History**: Maintain history of mapping changes and iterations

**Storage Keys Pattern:**
- **Mapping Drafts**: `mapping_draft_${mapperId}_${draftId}` - Individual mapping draft
- **Active Mappings**: `active_mappings` - Currently open mapping sessions
- **Mapping Templates**: `mapping_templates` - Reusable mapping configurations
- **Mapping History**: `mapping_history` - Historical mapping activity

**Mapper Persistence Features:**
- **Session Recovery**: Restore mapping sessions after browser restart
- **Progress Tracking**: Save mapping progress and allow resumption
- **Template Management**: Save and reuse mapping configurations
- **Version Control**: Track changes and allow rollback to previous versions
- **Collaboration**: Share mapping configurations between users

**Integration with JSON Mappers:**
- **JsonResultMapper**: Persist mapping results and processing state
- **JsonRequestMapper**: Save request mapping configurations
- **JsonResponseMapper**: Store response mapping settings
- **Custom Mappers**: Extend persistence to all mapper components

### **4. Dialog State Persistence System**

**Dialog Storage Structure:**
- **Dialog State**: Store dialog content and user input
- **Dialog Metadata**: Track dialog type, creation time, last activity
- **Dialog Relationships**: Link dialogs to specific data or forms
- **Dialog History**: Maintain history of dialog interactions

**Storage Keys Pattern:**
- **Dialog Drafts**: `dialog_draft_${dialogType}_${dialogId}` - Individual dialog state
- **Active Dialogs**: `active_dialogs` - Currently open dialog sessions
- **Dialog Templates**: `dialog_templates` - Reusable dialog configurations
- **Dialog History**: `dialog_history` - Historical dialog activity

**Dialog Persistence Features:**
- **Content Recovery**: Restore dialog content when user returns
- **Progress Saving**: Save multi-step dialog progress
- **Draft Management**: Manage multiple dialog drafts
- **Auto-Close Recovery**: Restore content after accidental dialog closure
- **Template System**: Save and reuse dialog configurations

**Dialog Types to Support:**
- **Creation Dialogs**: Organization, source, usecase factor creation
- **Management Dialogs**: JSON file management, organization selection
- **Deployment Dialogs**: Activity, source, workflow deployment
- **Utility Dialogs**: Confirmation, exploration, status translation

### **5. Backend Integration and Cleanup System**

**Backend Communication Pattern:**
- **Save Confirmation**: Receive confirmation when data is successfully saved to backend
- **Error Handling**: Handle save failures and retry mechanisms
- **Data Synchronization**: Ensure localStorage and backend data consistency
- **Conflict Resolution**: Handle conflicts between localStorage and backend data

**Cleanup Triggers:**
- **Successful Save**: Clear localStorage when data is confirmed saved
- **User Action**: Allow manual cleanup of localStorage data
- **Time-Based**: Automatic cleanup of old or stale data
- **Size-Based**: Cleanup when localStorage approaches size limits

**Cleanup Process:**
- **Data Validation**: Verify data was successfully saved to backend
- **Selective Cleanup**: Remove only successfully saved data
- **Backup Creation**: Create backup before cleanup for recovery
- **User Notification**: Inform user about cleanup actions
- **Audit Trail**: Maintain log of cleanup activities

**Backend Integration Points:**
- **API Endpoints**: Integrate with existing API save endpoints
- **WebSocket Updates**: Real-time updates for collaborative features
- **Batch Operations**: Efficient bulk save and cleanup operations
- **Error Recovery**: Handle network failures and retry mechanisms

### **6. Event System and Real-Time Updates**

**Custom Event System:**
- **File Events**: `jsonFilesUpdated`, `currentJsonSourceChanged`, `fileUploaded`, `fileRemoved`
- **Form Events**: `formDraftSaved`, `formDraftLoaded`, `formAutoSaved`, `formCleared`
- **Mapper Events**: `mappingDraftSaved`, `mappingProgressUpdated`, `mappingCompleted`
- **Dialog Events**: `dialogDraftSaved`, `dialogStateChanged`, `dialogClosed`
- **General Events**: `localStorageUpdated`, `dataSynchronized`, `cleanupCompleted`

**Event Handling:**
- **Component Updates**: Automatically update UI when data changes
- **Cross-Component Communication**: Share data between unrelated components
- **State Synchronization**: Keep multiple components in sync
- **User Notifications**: Inform users about important changes

**Event Integration:**
- **Existing Events**: Extend current `recentItemsUpdated` event system
- **New Event Types**: Add comprehensive event system for all features
- **Event Bubbling**: Proper event propagation and handling
- **Event Cleanup**: Remove event listeners when components unmount

### **7. Storage Optimization and Management**

**Storage Efficiency:**
- **Data Compression**: Compress large JSON data before storage
- **Lazy Loading**: Load data only when needed
- **Incremental Updates**: Update only changed data portions
- **Data Deduplication**: Avoid storing duplicate data

**Storage Monitoring:**
- **Size Tracking**: Monitor localStorage usage and warn when approaching limits
- **Performance Metrics**: Track storage operation performance
- **Error Monitoring**: Monitor and report storage errors
- **Usage Analytics**: Track storage usage patterns

**Storage Maintenance:**
- **Automatic Cleanup**: Remove old or unused data automatically
- **Manual Cleanup**: Provide user interface for manual storage management
- **Data Export**: Allow users to export their localStorage data
- **Data Import**: Allow users to import previously exported data

**Storage Security:**
- **Data Validation**: Validate all data before storage
- **Access Control**: Ensure proper access control for sensitive data
- **Data Encryption**: Encrypt sensitive data before storage
- **Audit Logging**: Log all storage operations for security auditing

### **8. User Experience and Interface Integration**

**User Interface Features:**
- **Storage Status**: Show current localStorage usage and status
- **Draft Management**: User interface for managing saved drafts
- **File Management**: Comprehensive file management interface
- **Recovery Options**: Easy recovery of lost or unsaved data

**User Notifications:**
- **Auto-Save Indicators**: Visual indicators when data is being saved
- **Save Confirmations**: Confirmations when data is successfully saved
- **Error Notifications**: Clear error messages for storage failures
- **Cleanup Notifications**: Inform users about cleanup activities

**User Controls:**
- **Storage Preferences**: User settings for storage behavior
- **Cleanup Controls**: Manual controls for data cleanup
- **Export/Import**: Tools for data backup and restoration
- **Privacy Controls**: Options for data privacy and security

**Integration with Existing UI:**
- **Component Integration**: Seamless integration with existing components
- **Design Consistency**: Maintain consistent design language
- **Accessibility**: Ensure accessibility compliance
- **Responsive Design**: Work across all device sizes

### **9. Implementation Phases and Priorities**

**Phase 1: System-Wide Data Persistence (Highest Priority)**
- Implement local-first data architecture
- Create comprehensive localStorage integration for all components
- Add sync buttons and edge function integration
- Implement data loss prevention system
- Redesign data folder structure for localStorage compatibility

**Phase 2: Core Infrastructure (High Priority)**
- Extend LocalStorageManager with new data types
- Implement JSON file management system
- Add comprehensive event system
- Create basic cleanup mechanisms

**Phase 3: Form and Dialog Persistence (High Priority)**
- Implement form draft management
- Add dialog state persistence
- Create auto-save functionality
- Add recovery mechanisms

**Phase 4: Mapper Integration (Medium Priority)**
- Integrate with JSON mappers
- Add mapping content persistence
- Implement template system
- Add collaboration features

**Phase 5: Backend Integration (Medium Priority)**
- Implement backend communication
- Add save confirmation system
- Create cleanup triggers
- Add error handling

**Phase 6: Advanced Features (Low Priority)**
- Add storage optimization
- Implement advanced user controls
- Create analytics and monitoring
- Add security features

**Phase 7: Polish and Testing (Low Priority)**
- Comprehensive testing
- Performance optimization
- User experience improvements
- Documentation completion

**Phase 8: Form Isolation and Pattern Unification (Critical Priority)**
- Fix form instance isolation for multiple forms on same page
- Unify ID generation patterns across codebase
- Standardize localStorage access patterns
- Add tab isolation support to LocalStorageManager
- Implement component-scoped DOM queries

### **10. Technical Implementation Details**

**Data Structure Design:**
- **Consistent Interfaces**: Create consistent interfaces for all data types
- **Version Management**: Implement versioning for data structure changes
- **Migration Support**: Support migration between different data versions
- **Backward Compatibility**: Ensure compatibility with existing data

**Error Handling Strategy:**
- **Graceful Degradation**: Continue operation even when storage fails
- **User Communication**: Clear communication about storage issues
- **Recovery Mechanisms**: Automatic and manual recovery options
- **Logging and Monitoring**: Comprehensive error logging and monitoring

**Performance Considerations:**
- **Lazy Loading**: Load data only when needed
- **Batch Operations**: Group multiple operations for efficiency
- **Caching Strategy**: Implement intelligent caching
- **Memory Management**: Proper memory management for large datasets

**Testing Strategy:**
- **Unit Testing**: Test individual storage functions
- **Integration Testing**: Test component integration
- **Performance Testing**: Test with large datasets
- **User Testing**: Test user experience and workflows

This comprehensive plan provides a detailed roadmap for implementing a robust localStorage system that integrates seamlessly with the existing codebase while providing powerful new features for data persistence, recovery, and management.

---

## **Phase 8: Form Isolation and Pattern Unification**

### **Critical Issue: Multiple Forms on Same Page**

**Problem Context:**
When multiple RequestMetadataForm instances exist on the same page (for example, in forms showcase with RunRequestForm, DeliveryForm, and StatusCheckForm all displayed simultaneously), selecting an option in any form's request-metadata section redirects focus to the input field in the first form instance on the page. Additionally, when the same page is open in multiple browser tabs, there are potential conflicts in localStorage keys and no isolation between tab instances.

**Root Causes Identified:**

1. **Global DOM Queries**: RequestMetadataForm uses `document.querySelector()` which searches the entire document and always returns the first matching element, regardless of which component instance triggered the action.

2. **Non-Unique Data Attributes**: All RequestMetadataForm instances use identical `data-metadata="url"` and `data-header-id` attributes without any instance-level namespacing.

3. **Weak ID Generation**: Header IDs are generated using `Date.now().toString()` which can create collisions when multiple forms render simultaneously or users click "Add Header" within the same millisecond.

4. **Hardcoded Initial IDs**: All forms start with a header that has `id: "1"`, creating duplicate IDs across multiple form instances until user interaction occurs.

5. **No Form Instance Identifier**: Components lack unique instance identifiers to namespace their elements and distinguish themselves from other instances.

6. **No Tab Isolation**: Auto-save functionality uses `form_${formId}` keys without tab-specific identifiers, causing conflicts when the same form is open in multiple browser tabs.

7. **Inconsistent localStorage Access**: Sixty percent of direct localStorage access bypasses the LocalStorageManager, leading to inconsistent prefixing, error handling, and lack of centralized control.

### **Pattern Analysis Across Codebase**

**Current ID Generation Patterns:**

**Pattern One - React.useId()**: 
- Location: chart.tsx line 46, form.tsx line 79
- Use Case: Component instance identification for accessibility and DOM relationships
- Characteristics: React-native, SSR-safe, guaranteed unique per component instance
- Format: Returns colon-separated string like `:r1:` that needs sanitization for use in data attributes
- Current Usage: Chart component creates `chart-${id || uniqueId.replace(/:/g, "")}` for clean attribute values
- Recommendation: Use for all component-level instance identification

**Pattern Two - Date.now() Only**:
- Location: request-metadata.tsx lines 82, 97, and initial header line 45
- Use Case: Runtime header ID generation
- Characteristics: Timestamp in milliseconds, no uniqueness guarantee
- Problem: Collision risk when operations happen in same millisecond
- Recommendation: Replace with Pattern Three

**Pattern Three - Date.now() + Math.random()**:
- Location: source-body-upload.tsx line 103
- Format: `Date.now()-${Math.random().toString(36).substr(2, 9)}`
- Use Case: File upload IDs and runtime unique identifiers
- Characteristics: Timestamp plus random alphanumeric suffix, excellent uniqueness
- Current Usage: File management system for uploaded JSON files
- Recommendation: Standard for all runtime ID generation

**Pattern Four - Counter-Based genId()**:
- Location: use-toast.ts lines 28-33
- Use Case: Sequential toast notification IDs
- Characteristics: Simple counter modulo MAX_SAFE_INTEGER
- Intentional Design: Sequential IDs appropriate for notification ordering
- Recommendation: Keep as-is, do not change (pattern serves specific purpose)

**Pattern Five - nanoid Package**:
- Location: package.json dependencies
- Current Status: Available but completely unused in UI code
- Recommendation: Do not introduce, existing patterns are sufficient

**Current LocalStorage Access Patterns:**

**Pattern One - LocalStorageManager (Preferred)**:
- Location: utils/local-storage.ts
- Features: Prefix system with `algops_`, error handling, type safety, timestamp tracking, version support
- Current Usage: hooks/use-auto-save.ts uses `storage.setItem()`
- Characteristics: Singleton pattern, comprehensive error handling, SSR-safe checks
- Problem: Only forty percent adoption across codebase

**Pattern Two - Direct localStorage with Prefix**:
- Location: recent-items.ts
- Format: Hardcoded `algops_recent_items` key
- Problem: Duplicates prefix logic, bypasses centralized management
- Missing: Timestamp tracking, version support, centralized error handling

**Pattern Three - Direct localStorage without Prefix**:
- Location: base-selector.tsx line 188, use-table-state.ts lines 38 and 65
- Format: Raw keys like `${storageKey}` or `table-state-${tableId}`
- Problem: No namespace isolation, potential key collisions with other applications
- Missing: All benefits of LocalStorageManager

**Pattern Four - Mixed Approach**:
- Location: hooks/use-auto-save.ts
- Uses LocalStorageManager but implements custom key pattern `form_${formId}`
- Problem: No tab isolation, conflicts across browser tabs

**Current DOM Query Patterns:**

**Pattern One - Global Document Queries (Problematic)**:
- Location: request-metadata.tsx lines 130, 141
- Scope: Entire document
- Problem: Cannot distinguish between multiple component instances
- Current Usage: Only file using this anti-pattern

**Pattern Two - Ref-Based Component Scoping (Standard)**:
- Location: short-text-input.tsx, json-explorer.tsx, base-hovercard.tsx
- Pattern: `const inputRef = useRef<HTMLInputElement>(null)` then `inputRef.current.focus()`
- Scope: Component instance only
- Characteristics: Type-safe, instance-isolated, follows React best practices
- Current Usage: One hundred percent of components except request-metadata

**Pattern Three - Container Ref with querySelector (Advanced)**:
- Location: json-explorer.tsx lines 109-110
- Pattern: `const containerRef = useRef<HTMLDivElement>(null)` then `containerRef.current?.querySelector()`
- Scope: Component subtree only
- Use Case: When querying child elements within component
- Characteristics: Scoped queries, null-safe with optional chaining

**Current Data Attribute Patterns:**

**Pattern One - Dynamic Namespaced (Correct)**:
- Location: chart.tsx line 52
- Format: `data-chart={chartId}` where chartId includes unique instance ID
- Usage: Enables scoped CSS and queries without collisions
- Characteristics: Instance-specific, collision-free

**Pattern Two - Static Non-Namespaced (Problematic)**:
- Location: request-metadata.tsx line 164
- Format: `data-metadata="url"` identical across all instances
- Problem: Indistinguishable between component instances

**Pattern Three - Dynamic Non-Namespaced (Partial)**:
- Location: request-metadata.tsx line 180
- Format: `data-header-id={header.id}` varies per header but not per component
- Problem: Header IDs could collide between different form instances

### **Comprehensive Solution Architecture**

### **1. RequestMetadataForm Component Isolation**

**File Location**: `/components/forms/blocks/request-metadata.tsx`

**Component Instance Identification:**

Add React.useId() hook at component initialization to generate unique instance identifier. Store in constant named `instanceId`. The raw React ID contains colons which are invalid in CSS selectors and data attributes, so create sanitized version by replacing all colons with empty string, stored in constant named `cleanId`. This approach follows the exact pattern established in chart.tsx line 47 where chart ID is constructed as `chart-${id || uniqueId.replace(/:/g, "")}`.

Optionally accept `instanceId` prop to allow parent components to provide explicit instance identifiers for specific use cases. If prop is provided, use it directly; otherwise use the generated React ID. This enables both automatic instance isolation and manual control when needed.

**Container Reference for Scoped Queries:**

Create container ref using `useRef<HTMLDivElement>(null)` at component top level. This ref will point to the root element of the component and enable all DOM queries to be scoped to this specific component instance rather than the entire document.

Wrap the entire component return JSX in a div element with the ref attached. This div becomes the query scope boundary. All querySelector operations will use `containerRef.current?.querySelector()` pattern instead of `document.querySelector()`.

**Data Attribute Namespacing:**

Current `data-metadata="url"` attribute on line 164 must become `data-metadata={cleanId}` to namespace with instance identifier. This ensures each form instance has unique data attributes.

Current `data-header-id={header.id}` attribute on line 180 must become `data-header-id={\`${cleanId}-${header.id}\`}` to combine instance ID with header ID. This prevents header ID collisions between different form instances.

**DOM Query Updates:**

Line 130 currently uses `document.querySelector('[data-metadata="url"] input')` which queries the entire document. Replace with `containerRef.current?.querySelector(\`[data-metadata="${cleanId}"] input\`)` to query only within this component instance and target this instance's specific metadata attribute.

Line 141 currently uses `document.querySelector(\`[data-header-id="${id}"]\` input)` which queries the entire document. Replace with `containerRef.current?.querySelector(\`[data-header-id="${cleanId}-${id}"]\` input)` to query only within this component instance and target the correctly namespaced header ID.

Add null checks using optional chaining operator to handle cases where ref is not yet attached or element doesn't exist. The pattern should be `containerRef.current?.querySelector()?.focus()` with appropriate type assertions only after null checks.

**Header ID Generation Fix:**

Lines 82 and 97 currently use `Date.now().toString()` for header ID generation. This creates potential for collisions when operations happen in the same millisecond or when multiple form instances initialize simultaneously.

Replace with the established pattern from source-body-upload.tsx line 103: `\`${Date.now()}-${Math.random().toString(36).substr(2, 9)}\``. This combines timestamp with random alphanumeric suffix providing nine additional characters of uniqueness.

Line 45 initializes headers array with hardcoded `id: "1"`. This means every RequestMetadataForm instance starts with the same ID. Replace with the same Date.now() plus random pattern to ensure uniqueness from initialization.

**Focus Behavior Verification:**

After implementing scoped queries and namespaced attributes, when user selects a method in the FormSelect component, the handleMethodChange function will execute setTimeout to focus the URL input. The query `containerRef.current?.querySelector(\`[data-metadata="${cleanId}"] input\`)` will now target only the URL input within the same form instance that triggered the selection, not the first form on the page.

Similarly, when user selects a header type in handleTypeChange, the query will target only the header value input within the same form instance and for the specific header ID that was changed.

### **2. Create Centralized ID Generation Utility**

**File Location**: `/utils/id-generator.ts` (new file)

**Utility Functions:**

Create `generateRuntimeId` function that takes optional string prefix parameter and returns string. Implementation combines `Date.now()` with `Math.random().toString(36).substr(2, 9)` to create format like `1234567890123-a8c3k2m9p`. If prefix provided, prepend with hyphen separator resulting in format like `header-1234567890123-a8c3k2m9p`.

Create `generateComponentId` function that wraps React.useId() and sanitizes the result. This function must be a hook (prefix with use) because it calls React.useId(). Returns the ID with all colons replaced by empty string to make it safe for use in data attributes and CSS selectors.

**Integration Points:**

Update request-metadata.tsx to import and use `generateRuntimeId('header')` for all header ID generation instead of inline Date.now() calls. This centralizes the pattern and makes it easier to update the algorithm in the future.

Update source-body-upload.tsx to import and use `generateRuntimeId('file')` instead of inline implementation. This ensures consistency and reduces code duplication.

Any future components that need runtime IDs should use this utility instead of implementing their own ID generation logic.

**Benefits of Centralization:**

Single source of truth for ID generation algorithms. If the pattern needs to change in the future (for example, switching to crypto.randomUUID() when browser support is universal), only one file needs updating.

Consistent ID format across entire codebase makes debugging easier and patterns more predictable.

Prevents developers from using weak patterns like Date.now() alone by providing the correct implementation as the path of least resistance.

Testable and mockable ID generation for unit tests that need deterministic IDs.

### **3. LocalStorageManager Enhancement for Tab Isolation**

**File Location**: `/utils/local-storage.ts`

**Tab Instance Identification:**

Add private property `tabId` to LocalStorageManager class initialized in constructor. Generate tab ID using the same runtime ID pattern: `\`tab-${Date.now()}-${Math.random().toString(36).substr(2, 9)}\``.

Store tab ID in instance but not in localStorage because each tab should have its own unique identifier that doesn't persist. The tab ID is session-specific and should be regenerated every time the page loads in any tab.

**Storage Method Enhancement:**

Extend `setItem` method signature to accept optional configuration object as third parameter with properties: version (existing), tabScoped (new boolean), and any future options.

When `tabScoped` is true, modify the storage key to include tab ID: `${this.prefix}${key}_${this.tabId}`. When false, use existing behavior: `${this.prefix}${key}`.

Default `tabScoped` to false to maintain backward compatibility with all existing storage operations. Only opt-in features like form auto-save should explicitly enable tab scoping.

**Storage Retrieval Enhancement:**

Extend `getItem` method to accept same optional configuration object. When `tabScoped` is true, attempt to retrieve with tab-scoped key first. If not found, optionally fall back to non-scoped key for migration scenarios.

This fallback behavior enables graceful migration where existing stored data can be read even after switching to tab-scoped storage, preventing data loss for users with existing saved forms.

**Key Cleanup Enhancement:**

Extend `getAllKeys` method to support filtering by tab scope. Add optional parameter to return only tab-scoped keys, only non-scoped keys, or all keys (default).

This enables selective cleanup of tab-specific data when user closes tab (using beforeunload event) while preserving shared data that should persist across tabs.

Add `clearTabScoped` method that removes all keys containing the current tab ID. This provides easy cleanup mechanism when user explicitly clears their session or when implementing automatic cleanup policies.

**Migration Support:**

Add method `migrateToTabScoped` that takes a key and migrates existing non-scoped data to tab-scoped format. This helps transition existing features to use tab isolation without losing user data.

Check if non-scoped key exists, if yes, read the data, write to tab-scoped key, optionally remove non-scoped key based on configuration parameter. Return boolean indicating success.

### **4. Auto-Save Hook Tab Isolation Integration**

**File Location**: `/hooks/use-auto-save.ts`

**Enable Tab-Scoped Storage:**

Update all `storage.setItem` calls in useAutoSave hook to pass `{ tabScoped: true }` as third parameter. This ensures each browser tab maintains independent form drafts.

Update all `storage.getItem` calls to pass `{ tabScoped: true }` as second parameter. This retrieves only the draft specific to the current tab.

**User Experience Implications:**

With tab isolation, each browser tab gets independent form auto-save state. If user opens same form in three tabs and makes different edits in each, each tab maintains its own draft independently.

When user closes a tab, the beforeunload handler saves the final state to tab-scoped storage. This data remains available if user reopens the form in a new tab and can be offered as recovery option.

Optionally implement tab-scoped draft list UI that shows all available drafts across different tab sessions with timestamps, allowing users to choose which draft to restore.

**Cross-Tab Awareness:**

Consider adding optional notification system that detects when same form is open in multiple tabs. Use localStorage events to detect when other tabs make changes.

Display non-intrusive indicator showing "This form is also open in X other tabs" to prevent user confusion about why their changes in one tab don't appear in another.

Provide optional "sync with other tab" action that allows user to pull changes from another tab's draft if desired.

### **5. Standardize All LocalStorage Access**

**Enforcement Strategy:**

Establish rule that all direct localStorage access must be replaced with LocalStorageManager calls. No exceptions unless explicitly documented with architectural decision record.

Create ESLint rule or code review checklist to catch direct localStorage usage in pull requests. Add comments in code explaining why LocalStorageManager must be used.

**Migration Plan for Existing Code:**

**File: base-selector.tsx (line 188)**
Currently uses `localStorage.setItem(storageKey, JSON.stringify(selectedValues))` directly. Replace with `storage.setItem(storageKey, selectedValues)` which handles JSON stringification internally.

Benefit: Adds timestamp tracking, consistent error handling, and prefix namespacing to prevent key collisions with other applications or browser extensions.

**File: use-table-state.ts (lines 38, 65)**
Currently uses `localStorage.getItem(\`table-state-${tableId}\`)` and corresponding setItem. Replace with `storage.getItem(\`table-state-${tableId}\`)` and `storage.setItem(\`table-state-${tableId}\`, state)`.

Benefit: Adds error handling so table state failures don't crash the component. Adds prefix to prevent collisions. Enables future version migrations if table state structure changes.

**File: recent-items.ts (lines 14, 34)**
Currently uses direct localStorage with hardcoded `algops_recent_items` key. Replace with `storage.getItem('recent_items')` and `storage.setItem('recent_items', items)` where the manager handles prefix automatically.

Benefit: Removes duplicate prefix logic. Centralizes error handling. Makes it possible to migrate recent items structure in future using version parameter.

**Verification:**

After migration, search entire codebase for `localStorage.` pattern outside of LocalStorageManager implementation file. Should find zero results in application code.

Test each migrated component to verify functionality remains identical. Pay special attention to error scenarios where localStorage might fail or be disabled.

### **6. Parent Form Component Integration**

**Files Affected**: 
- `/components/forms/run-request-form.tsx`
- `/components/forms/delivery-form.tsx`
- `/components/forms/status-check-form.tsx`

**Instance ID Prop Addition:**

Each parent form component should optionally accept `instanceId` prop of type string. This prop is passed through to RequestMetadataForm component.

When forms are used in isolation (one per page), no instanceId prop is needed. React.useId() in RequestMetadataForm will handle it automatically.

When forms are displayed together (like in forms showcase page), the showcase page can pass explicit instance IDs like `instanceId="run-request"`, `instanceId="delivery"`, `instanceId="status-check"` to ensure predictable and debuggable IDs.

**Auto-Save Integration:**

Each parent form that uses auto-save should specify unique formId that makes sense in context. For example, RunRequestForm could use formId based on source ID if editing specific source: `form_run_request_${sourceId}`.

When displaying multiple forms on showcase page, use descriptive formIds like `form_showcase_run_request`, `form_showcase_delivery`, etc. to prevent confusion in localStorage.

**Testing Multiple Forms:**

Forms showcase page at `/app/components-showcase/forms/page.tsx` currently displays RunRequestForm, DeliveryForm, StatusCheckForm, and SourceRequestResponseAccordion components all on same page.

After implementing isolation fixes, test this page specifically by:
- Selecting method in first form, verify focus goes to first form's URL input
- Selecting method in second form, verify focus goes to second form's URL input
- Selecting method in third form, verify focus goes to third form's URL input
- Adding header in first form, verify it only affects first form
- Adding header in second form, verify it only affects second form
- Verifying each form maintains independent state

**Multi-Tab Testing:**

Open forms showcase page in two browser tabs. Make different edits to the RunRequestForm in each tab. Verify that auto-save in each tab maintains separate drafts without conflict.

Open same source edit page in multiple tabs. Make changes in each tab independently. Verify each tab's auto-save works correctly and doesn't interfere with other tabs.

### **7. Data Attribute and CSS Selector Standards**

**Naming Convention:**

All data attributes used for component instance identification must follow pattern: `data-{component-name}={instanceId}` where component-name is kebab-case and instanceId is the sanitized React.useId() result.

All data attributes used for element identification within components must follow pattern: `data-{element-type}-{element-id}={compositeId}` where compositeId combines instance ID and element ID like `${instanceId}-${elementId}`.

**querySelector Best Practices:**

Never use `document.querySelector()` or `document.querySelectorAll()` in component code. Always use component-scoped ref-based queries.

Pattern for querying own elements: `containerRef.current?.querySelector('[data-attribute="value"]')`

Pattern for querying child component elements: `containerRef.current?.querySelector('[data-component] [data-element]')`

Always use optional chaining when querying to handle cases where ref is not mounted or element doesn't exist yet.

**CSS Selector Naming:**

When styling based on data attributes, use attribute selectors: `[data-metadata="url"]` for exact match or `[data-metadata*="url"]` for partial match.

When combining instance and element IDs, use partial match selectors: `[data-header-id*="${headerId}"]` will match `${instanceId}-${headerId}`.

Avoid using IDs (`#`) for styling component elements. Use data attributes or classes instead. IDs must be globally unique which conflicts with reusable components.

### **8. Event System Integration**

**LocalStorage Change Events:**

When LocalStorageManager performs setItem, removeItem, or clear operations, dispatch custom event `algopsStorageChange` with detail object containing: key, operation (set/remove/clear), tabId, timestamp.

Components can listen for these events to react to storage changes: `window.addEventListener('algopsStorageChange', handleStorageChange)`

This enables cross-component reactivity and real-time updates when storage changes occur.

**Cross-Tab Communication:**

Browser's native storage event fires when localStorage changes in other tabs. LocalStorageManager should listen for storage events and re-dispatch as `algopsStorageChange` events with additional metadata.

This enables components to detect when other tabs modify shared storage and optionally sync or show notifications.

For tab-scoped storage, ignore storage events that don't match current tab ID to prevent unnecessary processing.

**Form-Specific Events:**

When form auto-save triggers, dispatch `algopsFormAutoSaved` event with formId and tabId in detail. Components can show save indicators or sync status based on these events.

When form draft is loaded from storage, dispatch `algopsFormDraftLoaded` event. This enables analytics tracking or user notifications about recovered drafts.

### **9. Backward Compatibility and Migration**

**Existing Data Preservation:**

Before implementing tab isolation for auto-save, check if non-scoped form data exists in localStorage. If found, migrate to tab-scoped format or offer user choice to restore.

Display one-time migration notification explaining that forms now save separately per tab and any existing saved drafts have been preserved in current tab.

**Version Tracking:**

Use LocalStorageManager's version parameter to mark storage structure versions. Current implementation should be version "2.0" indicating second generation with tab support.

When reading old version data, automatically migrate to new structure. Store version number with data to enable future migrations.

**Rollback Capability:**

Maintain ability to rollback to non-scoped storage if tab isolation causes issues. Provide admin setting or feature flag to disable tab scoping if needed.

If rollback occurs, merge tab-scoped drafts into single non-scoped draft by timestamp (most recent wins) to prevent data loss.

### **10. Performance Optimization**

**Lazy Initialization:**

Don't query localStorage on every component render. Initialize storage data in useEffect with empty dependency array to run once on mount.

Cache storage reads in component state to avoid repeated localStorage access. Only re-read when storage change events indicate data was modified.

**Debounced Saves:**

Auto-save should debounce storage writes to prevent excessive localStorage operations. Current 5-second interval is reasonable but ensure it uses debouncing not throttling.

Debouncing means resetting timer on each change, so rapid typing results in single save 5 seconds after user stops typing. This is more efficient than saving every 5 seconds regardless of activity.

**Batch Operations:**

When saving multiple related items (like form metadata, form data, and form validation state), use single storage operation with combined object rather than three separate operations.

This reduces localStorage access overhead and ensures atomic updates where all related data saves together or fails together.

**Storage Size Monitoring:**

Implement periodic check of localStorage usage to warn users before hitting quota limits. Most browsers provide 5-10MB localStorage quota.

Calculate approximate size of stored data by JSON stringifying and checking byte length. Show warning when usage exceeds 80% of estimated quota.

Provide storage management UI allowing users to clear old drafts, exported data, or cached files to free up space.

### **11. Testing Strategy**

**Unit Tests for ID Generation:**

Test generateRuntimeId produces unique IDs across multiple rapid calls. Test with loop of 1000 calls ensuring no duplicates.

Test generateRuntimeId with prefix produces correct format: `prefix-timestamp-randomstring`.

Test that generated IDs are valid for use in data attributes and CSS selectors (no special characters that need escaping).

**Component Isolation Tests:**

Mount multiple RequestMetadataForm instances in test environment. Verify each has unique instanceId.

Simulate user interaction in one instance (selecting method) and verify focus changes only affect that instance's elements, not other instances.

Test data attribute values are unique across instances by querying rendered DOM for duplicate attribute values.

**LocalStorage Manager Tests:**

Test tab-scoped setItem and getItem correctly add and retrieve data with tab ID in key.

Test non-scoped operations remain unchanged for backward compatibility.

Test getAllKeys correctly filters tab-scoped vs non-scoped keys.

Test clearTabScoped removes only current tab's data, leaving other tabs' data and shared data intact.

**Integration Tests:**

Test forms showcase page with multiple forms ensuring each operates independently.

Test multi-tab scenario where same form is open in multiple tabs with different drafts.

Test auto-save in one tab doesn't interfere with auto-save in another tab.

Test migration from old non-scoped storage to new tab-scoped storage preserves data.

**Browser Compatibility:**

Test in Chrome, Firefox, Safari, and Edge to ensure localStorage behavior is consistent.

Test in private/incognito mode where localStorage might behave differently.

Test with localStorage disabled (some browsers allow this) and verify graceful degradation.

Test with localStorage quota exceeded and verify appropriate error handling and user feedback.

### **12. Documentation Requirements**

**Code Documentation:**

Add JSDoc comments to all new utility functions explaining parameters, return values, and usage examples.

Document the instanceId pattern in RequestMetadataForm component with comments explaining why it exists and how it works.

Add inline comments at all ref-based querySelector calls explaining why containerRef is used instead of document.

**Developer Guidelines:**

Create document explaining form isolation pattern and when to use instanceId prop.

Document the ID generation utilities and when to use each one (React.useId for components, generateRuntimeId for data).

Create localStorage best practices guide mandating use of LocalStorageManager and explaining tab scoping.

**Migration Guide:**

Document step-by-step process for migrating existing components to use standardized patterns.

Provide code examples showing before/after for common scenarios like adding instance isolation to reusable form components.

List all files modified in this phase with explanation of changes for future reference.

**User-Facing Documentation:**

Explain tab-scoped auto-save behavior in user help documentation.

Document how to manage saved drafts across multiple tabs.

Explain storage quota warnings and how users can free up space.

This comprehensive phase ensures form components work correctly with multiple instances on the same page and in multiple browser tabs, while standardizing patterns across the codebase and integrating seamlessly with the LocalStorage system. All solutions are based on existing patterns found in the codebase, ensuring consistency and maintainability.
