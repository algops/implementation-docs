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
- **Auto-Save Hook**: `useAutoSave()` hook in `auto-save-form.tsx`
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
- **Form Blocks**: `auto-save-form.tsx`, `dynamic-form.tsx`, `form-drag-drop.tsx`, `form-field.tsx`, `form-section.tsx`
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
