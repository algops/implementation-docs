# Forms Implementation Plan

## Component Usage Validation
**✅ COMPONENT SOURCE CHECK:**
- [ ] Uses only components from `components/` folder
- [ ] Uses only components from `components/ui-elements/` subfolder
- [ ] **NO direct usage of `components/ui/` folder components**

## Overview
Enhance existing form components to better work with JSON data from your data files. You already have a comprehensive form system with specialized forms for different use cases.

## Current Form Components
- **RunRequestForm** (formerly InferenceForm): Form for run/execution requests to external sources with headers and mapping
- **DeliveryForm**: Form for delivery endpoint configuration
- **StatusCheckForm**: Form for status checking
- **CustomizationForm**: Form for customization settings
- **SettingsForm**: General settings form
- **FormField**: Individual form field component
- **FormSection**: Form section with collapsible functionality
- **FormDragDrop**: Drag and drop form component
- **SimpleForm**: Basic form component
- **DynamicForm**: Dynamic form generation (complex, needs refactoring)
- **MultiFieldForm**: Multi-field form component
- **AutoSaveForm**: Auto-saving form component
- **RequestHeaderForm**: Reusable header management component
- **LoadBalancingForm**: Reusable load balancing settings component

## Phase 1: Enhance Existing Forms for JSON Data

### 1.1 Enhance Form Components
- **DynamicForm**: Generate forms based on JSON data structure
- **AutoSaveForm**: Auto-save form data to JSON files
- **FormField**: Better handling of JSON data types
- **FormSection**: Collapsible sections based on JSON structure

### 1.2 Enhance Specialized Forms with Data Options

#### RunRequestForm (formerly InferenceForm)
**Data Source**: `sources.json`
**Purpose**: Configure the initial run/execution request to external sources (datasets, agents, integrations, ML models, LLMs)
**Form Fields**:
1. **Request Headers** - `run_request_template.headers` (JSON object)
2. **Content Type** - `run_request_template.content_type` (string)
3. **HTTP Method** - `run_request_template.method` (POST, PUT, PATCH)
4. **Request URL** - `run_request_template.url` (string with variables)
5. **Request Body** - `run_request_template.body` (JSON template)
6. **Timeout** - `timeout` (number in seconds)
7. **Max Objects** - `max_objects` (number)
8. **Variable Mapping** - `<<$run_setup>>`, `<<$max_objects>>` placeholders
9. **Authentication** - Auth headers and tokens
10. **Response Mapping** - Response field mapping configuration

**Note**: Renamed from InferenceForm to RunRequestForm to align with data layer terminology (run_request_template) and avoid confusion with ML-specific "inference" term.

#### DeliveryForm
**Data Source**: `sources.json`
**Form Fields**:
1. **Delivery URL** - `delivery_request_template.url` (string)
2. **HTTP Method** - `delivery_request_template.method` (POST, PUT, PATCH)
3. **Delivery Headers** - `delivery_request_template.headers` (JSON object)
4. **Delivery Body** - `delivery_request_template.body` (JSON template)
5. **External ID Mapping** - `<<$run_external_id>>` placeholder
6. **Retry Configuration** - Retry attempts and intervals
7. **Error Handling** - Error response handling
8. **Delivery Timeout** - Delivery request timeout
9. **Batch Size** - Number of results per delivery
10. **Delivery Schedule** - When to deliver results

#### StatusCheckForm
**Data Source**: `sources.json`
**Form Fields**:
1. **Run External ID Variable** - `<<$runexternalid>>` placeholder (string)
2. **Status URL** - `status_request_template.url` (string)
3. **Check Method** - `status_request_template.method` (GET, POST)
4. **Status Headers** - `status_request_template.headers` (JSON object)
5. **Polling Interval** - Time between status checks (seconds)
6. **Max Polling Time** - Maximum time to poll for status
7. **Status Mapping** - Map response fields to status values
8. **Success Criteria** - What constitutes a successful status
9. **Error Criteria** - What constitutes an error status
10. **Timeout Handling** - What to do on timeout

#### CustomizationForm
**Data Source**: `navigation.json` + `dashboard.json`
**Form Fields**:
1. **Customization Level** - Radio group selection (never, approval, usecases-only, anything)
2. **JSON Generation Instructions** - Textarea for AI instructions
3. **Theme Selection** - Light, dark, auto theme options
4. **Color Scheme** - Primary and accent color selection
5. **Layout Preferences** - Dashboard layout configuration
6. **Notification Settings** - Email, push notification preferences
7. **Language Selection** - UI language preference
8. **Timezone** - User timezone setting
9. **Date Format** - Date display format preference
10. **Number Format** - Number display format preference

#### SettingsForm
**Data Source**: `sources.json` + `navigation.json`
**Form Fields**:
1. **Load Balancing Configuration** - Concurrency and timeout settings
2. **Retry Configuration** - Retry attempts and delay settings
3. **Organization Settings** - Org name, description, contact info
4. **API Configuration** - API keys, endpoints, rate limits
5. **Data Retention** - Data retention policies and periods
6. **Security Settings** - Password policies, 2FA, access controls
7. **Integration Settings** - Third-party service configurations
8. **Performance Settings** - Caching, optimization preferences
9. **Backup Settings** - Backup frequency and retention
10. **Monitoring Settings** - Alert thresholds and notifications

### 1.3 Form Component Issues to Fix
- **Import Path Inconsistencies**: Fix incorrect import paths in DynamicForm
- **DynamicForm Refactoring**: Break down 448-line component into smaller parts
- **Missing Validation Utilities**: Add form validation helper functions
- **Component Standardization**: Ensure all forms use consistent patterns

### 1.4 Specific Form Updates Required

#### RunRequestForm Updates (formerly InferenceForm)
**Current Status**: Uses RequestHeaderForm correctly, missing JSON data integration
**Changes Needed**:
- Rename component from InferenceForm to RunRequestForm for terminology consistency
- Add timeout field from `sources.json` data
- Add max_objects field from `sources.json` data
- Pre-populate headers from `run_request_template.headers`
- Add request body template field

#### DeliveryForm Updates
**Current Status**: Missing delivery body template field, no JSON data integration
**Changes Needed**:
- Add FormTextarea for delivery body template
- Integrate with `sources.json` data
- Pre-populate from `delivery_request_template` fields
- Add retry configuration section

#### StatusCheckForm Updates
**Current Status**: Missing status URL and method fields, no JSON data integration
**Changes Needed**:
- Add FormInput for status URL
- Add FormSelect for HTTP method
- Integrate with `sources.json` data
- Pre-populate from `status_request_template` fields

#### CustomizationForm Updates
**Current Status**: Already well-implemented with CustomRadioGroup and JSON instructions
**Changes Needed**: None - form is correctly implemented

#### SettingsForm Updates
**Current Status**: Missing organization settings fields, no JSON data integration
**Changes Needed**:
- Add organization name/description fields
- Add API configuration section
- Integrate with `navigation.json` data

#### DynamicForm Updates
**Current Status**: 448 lines, too complex, incorrect import paths
**Changes Needed**:
- Break into smaller components: DynamicFormField, DynamicFormActions, DynamicFormValidation
- Fix import paths from `../inputs/` to `../ui-elements/inputs/`
- Add proper error handling
- Integrate with JSON data sources

### 1.5 JSON Data Integration
- **Form Generation**: Generate forms from JSON schemas
- **Data Validation**: Validate form data against JSON schemas
- **Auto-population**: Pre-populate forms with existing JSON data
- **Data Export**: Export form data to JSON format

### 1.6 Integration with Existing Components
- **Input Components**: Keep using FormInput, FormTextarea, FormSelect, CustomRadioGroup (already well-designed)
- **Selector Components**: Integrate SourceSelector, JsonSelector, BaseSelector for data selection needs
- **Search Components**: Add SearchBar for form filtering and searching capabilities

## Phase 2: Components & Features

### 2.1 BaseForm Component
- **Form Management**: Handle form submission and validation
- **State Control**: Form data and validation state
- **Event System**: Form events and notifications
- **Accessibility**: ARIA support and keyboard navigation

### 2.2 FormField Component
- **Field Wrapper**: Label, input, and error display
- **Validation Display**: Real-time validation feedback
- **Error Handling**: Clear error message presentation
- **Responsive Design**: Adapt to different screen sizes

### 2.3 FormInput Component
- **Input Types**: Text, email, password, number, url, etc.
- **Validation**: Real-time input validation
- **Auto-complete**: Smart input suggestions
- **Touch Support**: Mobile-friendly input interactions

### 2.4 FormSelect Component
- **Dropdown Options**: Searchable option lists
- **Multi-select**: Multiple option selection
- **Custom Options**: Dynamic option generation
- **Keyboard Navigation**: Full keyboard accessibility

### 2.5 FormTextarea Component
- **Auto-resize**: Dynamic height adjustment
- **Monospace Option**: For JSON/code input
- **Min/Max Rows**: Height constraints
- **Validation**: Real-time textarea validation

### 2.6 CustomRadioGroup Component
- **Radio Selection**: Single option selection
- **Option Descriptions**: Rich option descriptions
- **Custom Styling**: Consistent radio group appearance
- **Accessibility**: Full keyboard and screen reader support

## Phase 3: Interactions & User Experience

### 3.1 Form Interactions
- **Input Focus**: Clear focus indicators and states
- **Real-time Validation**: Immediate feedback on input
- **Auto-save**: Save form progress automatically
- **Smart Defaults**: Intelligent default value suggestions

### 3.2 Validation Experience
- **Clear Messages**: User-friendly error descriptions
- **Visual Indicators**: Color-coded validation states
- **Progressive Validation**: Validate as user types
- **Error Recovery**: Help users fix validation errors

### 3.3 Advanced Features
- **Conditional Fields**: Show/hide fields based on conditions
- **Field Dependencies**: Dynamic field relationships
- **File Uploads**: Drag-and-drop file handling
- **Rich Text**: WYSIWYG text editing capabilities

## Phase 4: Responsive Design & Accessibility

### 4.1 Responsive Layout
- **Mobile-First**: Touch-friendly form design
- **Adaptive Fields**: Responsive field sizing
- **Touch Targets**: Adequate size for mobile interaction
- **Orientation Support**: Portrait and landscape layouts

### 4.2 Accessibility Features
- **ARIA Labels**: Screen reader support
- **Keyboard Navigation**: Full keyboard accessibility
- **Focus Management**: Clear focus indicators
- **High Contrast**: Support for accessibility themes

### 4.3 Performance Optimization
- **Efficient Validation**: Optimize validation performance
- **Debounced Input**: Reduce validation frequency
- **Memory Management**: Clean up form resources
- **Smooth Interactions**: 60fps form interactions

## Phase 5: Advanced Features & Integration

### 5.1 Form Intelligence
- **Smart Validation**: Context-aware validation rules
- **Auto-completion**: Intelligent field suggestions
- **Error Prevention**: Proactive error detection
- **User Guidance**: Helpful form completion hints

### 5.2 Integration Features
- **API Integration**: Seamless backend communication
- **Data Binding**: Dynamic form population
- **Event System**: Form events for parent components
- **Plugin System**: Extensible form functionality

### 5.3 Testing & Validation
- **Unit Tests**: Component and utility function testing
- **Integration Tests**: End-to-end form workflow testing
- **Accessibility Tests**: Screen reader and keyboard testing
- **Performance Tests**: Validation and submission testing

## File Changes Required

### New Files
- `components/forms/form-validation.tsx` - Validation utilities
- `components/forms/form-submission.tsx` - Submission handling
- `hooks/use-form.ts` - Form state management hook
- `utils/form-utils.ts` - Form utility functions
- `components/forms/dynamic-form-field.tsx` - Extracted from DynamicForm
- `components/forms/dynamic-form-actions.tsx` - Extracted from DynamicForm
- `components/forms/dynamic-form-validation.tsx` - Extracted from DynamicForm

### Modified Files
- `components/forms/dynamic-form.tsx` - Refactor and fix import paths
- `components/forms/inference-form.tsx` → `components/forms/run-request-form.tsx` - Rename for terminology consistency, add timeout, max_objects, request body fields
- `components/forms/delivery-form.tsx` - Add delivery body template, retry configuration
- `components/forms/status-check-form.tsx` - Add status URL, method fields
- `components/forms/settings-form.tsx` - Add organization settings, API configuration
- `components/forms/index.ts` - Export RunRequestForm (renamed from InferenceForm) and other form components
- All form components - Standardize import paths and patterns

## Testing Checklist

### Unit Tests
- [ ] Form rendering and props
- [ ] Validation logic
- [ ] Event handling
- [ ] Error management

### Integration Tests
- [ ] Complete form workflow
- [ ] Validation system
- [ ] Submission handling
- [ ] Error recovery

### User Experience Tests
- [ ] Form interaction flow
- [ ] Mobile responsiveness
- [ ] Accessibility compliance
- [ ] Performance with complex forms

## Implementation Priority

### Phase 1: Critical Fixes (High Priority)
1. **Fix DynamicForm import paths** - Critical for functionality
2. **Refactor DynamicForm** - Break down 448-line component
3. **Add validation utilities** - Essential for form functionality

### Phase 2: Form Enhancements (Medium Priority)
1. **RunRequestForm** (rename from InferenceForm) - Add missing fields and JSON integration
2. **DeliveryForm** - Add delivery body template and retry configuration
3. **StatusCheckForm** - Add status URL and method fields
4. **SettingsForm** - Add organization settings

### Phase 3: Integration (Low Priority)
1. **Add selector components** - SourceSelector, JsonSelector integration
2. **Add search functionality** - SearchBar integration
3. **Standardize patterns** - Ensure consistent import paths

## Phase 6: Multi-File JSON Upload & Mapping Integration

### Overview
Implement a comprehensive multi-file JSON upload system that integrates FormDragDrop component with JsonRequestMapper, JsonResponseMapper, and JsonExplorer to enable users to upload multiple third-party JSON files, validate them, switch between them, and create variable mappings for request and response templates.

### Executive Summary

**Goal**: Enable multi-file JSON upload with visual validation, file switching, and integrated mapping across all three form types.

**Solution**: Create reusable SourceBodyForm component that encapsulates the entire workflow from file upload through JSON parsing, mapper processing, and explorer visualization.

**Impact**: 
- Reduce form code by ~330 lines total across 3 forms
- Standardize JSON upload behavior across all forms
- Enable users to upload, validate, and switch between multiple JSON files
- Integrate enhanced visual feedback into FormDragDrop
- Simplify form components to focus on their unique logic

**Key Components**:
1. **SourceBodyForm** (NEW) - Manages multi-file state, coordinates mapper and explorer
2. **FormDragDrop** (ENHANCED) - Adds JSON validation and improved visual feedback  
3. **JsonExplorer** (ENHANCED) - Accepts file management props for switching
4. **RunRequestForm** (RENAMED from InferenceForm) - Terminology alignment
5. **StatusCheckForm** (SIMPLIFIED) - Removes JSON boilerplate
6. **DeliveryForm** (SIMPLIFIED) - Removes JSON boilerplate

**User Experience**:
User uploads multiple JSON files → Files validated with visual feedback → User switches between files in dropdown → Active file processed through mapper → Explorer shows structure with mappings → User creates variable mappings → Switches to another file → Process repeats seamlessly

**Technical Approach**:
- FormDragDrop validates file format, size, and optionally JSON.parse()
- Form component uses SourceBodyForm which manages JsonFile array with parsed data
- SourceBodyForm tracks active file and passes its data to base mapper (JsonRequestMapper or JsonResponseMapper)
- Mapper processes single file at a time through 7-step pipeline generating basePaths, fullPaths, and statistics
- Explorer receives processed data plus file management callbacks for switching
- When user switches files, activeFileId updates, triggering mapper re-processing with new file's data
### 6.1 Component Architecture

#### SourceBodyForm Component
**File**: `components/forms/blocks/source-body-form.tsx` (NEW)
**Purpose**: Reusable component that encapsulates the complete workflow of uploading multiple JSON files, parsing them, managing active file selection, processing through base mapper components (JsonRequestMapper or JsonResponseMapper), and displaying in JsonExplorer with full mapping capabilities.

**Interface Definition**:
```
SourceBodyFormProps {
  label: string                              // Section title (e.g., "Request JSON Mapping")
  description?: string                       // Section description
  mapperType: 'request' | 'response' | 'result'        // Which base mapper to use (JsonRequestMapper, JsonResponseMapper, or JsonResultMapper)
  requestType: 'run_request' | 'status_request' | 'delivery_request'  // Request type identifier for mapping configuration
  mappings?: RequestAndResponseVariableMappings  // Variable mapping configuration
  onMappingsChange?: (mappings: any) => void     // Callback when mappings update
  maxFiles?: number                          // Maximum files allowed (default: 5)
  maxSize?: number                           // Maximum file size in MB (default: 10)
  className?: string                         // Additional CSS classes
  collapsible?: boolean                      // Enable collapsible section
  defaultExpanded?: boolean                  // Default expansion state
}

JsonFile {
  id: string              // Unique identifier generated from filename + timestamp
  name: string           // Original filename
  file: File             // Original File object reference
  data: any              // Parsed JSON data (JavaScript object)
  size: number           // File size in bytes
  lastModified: Date     // File modification timestamp
}
```

**Internal State Management**:
- `jsonFiles: JsonFile[]` - Array storing all uploaded JSON files with their parsed data
- `activeFileId: string | null` - ID of currently selected file being displayed in explorer
- `rawFiles: File[]` - Array of File objects for FormDragDrop value prop
- Derived state: `activeFile = jsonFiles.find(f => f.id === activeFileId)`
- Derived state: `activeJsonData = activeFile?.data || null`

**Component Structure**:
The component will contain three main sections in vertical layout:
1. Section header with label and description
2. FormDragDrop component for file upload
3. Conditional rendering of Mapper plus Explorer when activeJsonData exists

**Mapper Selection Logic**:
Component uses base mapper components with requestType configuration:
- When mapperType equals "request": use JsonRequestMapper from components/json/request/json-request-mapper.tsx
- When mapperType equals "response": use JsonResponseMapper from components/json/response/json-response-mapper.tsx
- Both mappers receive requestType prop to identify whether processing run_request (inference), status_request (status check), or delivery_request (delivery) mappings
- The requestType value is passed to mapper through requestAndResponseVariableMappings object which contains RequestResponseMappings data, requestType string, and inputType string

The base mappers (JsonRequestMapper and JsonResponseMapper) are production components located at components/json/request/json-request-mapper.tsx and components/json/response/json-response-mapper.tsx. Both components accept data as parsed JavaScript object (line 44 in both files), requestAndResponseVariableMappings configuration object (line 45 in both files), and onProcessedData callback returning ReactNode (line 46-51 in both files). The RequestAndResponseVariableMappings interface (lines 19-40 in both files) defines the mapping configuration structure including requestType field at line 38 which distinguishes between run_request, status_request, and delivery_request types. Both mappers process JSON through identical pipeline using generatePathData, assignVariableToFullPath, getBasePathStatus, getBasePathStatistics functions from utils/json/mapping-utils.tsx.

The specialized wrapper components (JsonInferenceRequestMapper at components/json/request/json-inference-request-mapper.tsx, JsonStatusCheckRequestMapper at components/json/request/json-status-check-request-mapper.tsx, JsonDeliveryRequestMapper at components/json/request/json-delivery-request-mapper.tsx) are demo/showcase components with hardcoded sample data (lines 13-23 in each). They wrap the base JsonRequestMapper and are not used in production forms. These remain as-is for demonstration purposes.

**Responsibilities**:
- Manage array of uploaded JSON files with their parsed data
- Track which file is currently active for display
- Parse JSON files when uploaded through FormDragDrop
- Select base mapper (JsonRequestMapper or JsonResponseMapper) based on mapperType prop
- Construct requestAndResponseVariableMappings object with requestType, inputType, and mapping data
- Pass active file's data to selected base mapper
- Render JsonExplorer with processed data from mapper
- Handle file selection, removal, and upload through JsonSelector UI
- Coordinate data flow between FormDragDrop, base Mapper, and Explorer components

### 6.2 FormDragDrop Enhancement

**File**: `components/forms/blocks/drag-drop-upload.tsx` (MODIFY)

**Current State** (lines 11-29):
The FormDragDropProps interface contains basic file upload props including label, value as File array, onChange callback, placeholder, error, description, required, disabled, className, multiple, accept, maxSize, maxFiles, loading, onFocus, and onBlur.

**Changes Required**:

**Interface Update** (add to FormDragDropProps after line 28):
Add new optional props:
- `validateJson?: boolean` - When true, component will read each uploaded JSON file using FileReader and attempt JSON.parse() to verify it contains valid JSON syntax
- `onJsonParsed?: (file: File, parsedData: any, fileName: string) => void` - Callback invoked for each successfully parsed JSON file, providing the File object, the parsed JavaScript object, and the filename

**Validation Logic Enhancement** (modify validateFile function around line 53):
Current validateFile function only checks file size against maxSize limit and file format against accept prop using extension and MIME type matching.

Enhanced validation should:
- Keep existing size validation checking file.size against maxSize in bytes
- Keep existing format validation using accept prop with extension and MIME type checking
- Return error string if validation fails, null if passes

**New Async JSON Validation Function** (add after validateFile function around line 74):
Create new function validateJsonContent that accepts File parameter and returns Promise resolving to object containing isValid boolean and optional error string. This function will:
- Create new FileReader instance
- Call reader.readAsText with the file
- In reader.onload handler, attempt JSON.parse on the result string
- If parse succeeds, resolve with isValid true
- If parse throws error, resolve with isValid false and error message from catch block
- In reader.onerror handler, resolve with isValid false

**File Handling Enhancement** (modify handleFiles function around line 76):
Current handleFiles processes FileList, validates each file, checks maxFiles limit, collects errors, and calls onChange with valid files.

Enhanced logic when validateJson is true:
- After basic file validation, for each valid file, call validateJsonContent function
- Wait for all JSON validation promises using Promise.all
- For files with valid JSON (isValid true), call onJsonParsed callback if provided, passing file, parsedData, and file.name
- Still call onChange with File array (unchanged behavior for backward compatibility)
- Update dragError state to include JSON validation errors

**UI Enhancement** (modify file list rendering around lines 230-260):
Current file list shows basic file cards with File icon (line 238), file name in span (line 240-242), formatted file size (line 243-245), and remove button using ghost variant Button (lines 249-256).

Enhanced file card should incorporate improved visual patterns:
- Keep existing File icon from lucide-react
- Keep existing file name and formatted size display
- Add FileText icon from lucide-react for better visual distinction
- NEW: When validateJson is true, add validation status section after file size showing:
  - For valid JSON: green checkmark icon (Check from lucide-react) with text "Valid JSON" in green color using text-green-600 class
  - For invalid JSON: red X icon with text "Invalid JSON" in red color using text-destructive class
  - Badge component wrapping status with appropriate variant (success or destructive)
- NEW: Add properties count display when JSON is valid showing "X properties" in muted text color
- Keep existing remove button but enhance hover state using destructive variant pattern
- Add subtle border and background color changes on hover
- Consider adding preview truncation showing first 200 characters of JSON in monospace font within expandable section

**Success State Pattern**:
When file is successfully validated, the card should transform to show success state:
- Border changes to success color (green)
- Background uses subtle green tint (bg-green-50 or similar)
- FileText icon in green (text-green-600)
- File name remains prominent
- Success indicator clearly visible
- Optional JSON preview section in muted background

**Error State Pattern**:
When file fails validation:
- Border changes to destructive color (red)
- Background uses subtle red tint
- Error icon in red (text-destructive)
- Error message displayed below file name
- Clear indication of what failed (format, size, or JSON parse)

### 6.3 JsonExplorer Enhancement

**File**: `components/json/interface/json-explorer.tsx` (MODIFY)

**Current State** (lines 33-41):
JsonExplorerProps interface accepts data, mappings, basePaths, fullPaths, statistics, mapperType, and options props.

**Interface Update** (add to JsonExplorerProps after line 40):
Add new file management props:
- `files?: JsonFile[]` - Array of all uploaded JSON files (where JsonFile matches the interface defined in SourceBodyForm)
- `currentFile?: string | null` - ID of currently active/selected file
- `onFileSelect?: (fileId: string) => void` - Callback when user selects different file from dropdown
- `onFileRemove?: (fileId: string) => void` - Callback when user removes a file
- `onFileUpload?: (file: File) => void` - Callback when user uploads new file through dialog

**Current JsonSelector Integration** (lines 406-418):
Currently hardcoded with empty arrays:
- currentSource set to string "current-source"
- availableSources set to empty array
- onSourceChange, onUploadNew, onRemoveSource set to empty functions
- files set to empty array
- onFileSelect, onFileRemove, onFileUpload set to empty functions

**Enhanced JsonSelector Integration** (replace lines 406-418):
Pass actual data from props:
- currentSource should be currentFile prop value or undefined
- availableSources should map files prop to array of objects with id from file.id, name from file.name, and description showing file.propertiesCount plus "properties" text
- onSourceChange should call onFileSelect prop function
- onUploadNew should call existing setJsonDialogOpen with true (keep current behavior)
- onRemoveSource should call onFileRemove prop function
- files should pass files prop directly
- onFileSelect should call onFileSelect prop function
- onFileRemove should call onFileRemove prop function  
- onFileUpload should call onFileUpload prop function
- openDialog and onDialogOpenChange remain unchanged (existing local state)

**Backward Compatibility**:
All new props are optional with safe fallbacks. If files prop is undefined or empty array, component behaves exactly as before with empty state. This ensures existing usages continue working without modification.

### 6.4 Form Component Simplification

**Files**: 
- `components/forms/inference-form.tsx` (MODIFY)
- `components/forms/status-check-form.tsx` (MODIFY)
- `components/forms/delivery-form.tsx` (MODIFY)

#### InferenceForm Changes

**Current State** (lines 9-23):
InferenceFormData interface contains headers, method, url, loadBalancing object, requestJsonFiles, responseJsonFiles, requestJsonData, responseJsonData, requestVariableMappings, and responseVariableMappings fields.

**Interface Simplification** (replace lines 9-23):
Remove all JSON-related fields since SourceBodyForm manages them internally. Keep only:
- headers: RequestHeader[]
- method: string
- url: string
- loadBalancing object with concurrency and timeout

**State Initialization** (lines 38-49):
Remove all JSON-related state initialization since it moves to SourceBodyForm component.

**Handler Removal** (lines 55-91):
Remove all JSON-related handlers: handleRequestJsonFilesChange, handleResponseJsonFilesChange, handleRequestJsonDataChange, handleResponseJsonDataChange, handleRequestVariableMappingsChange, handleResponseVariableMappingsChange. These handlers are no longer needed as SourceBodyForm manages this internally.

**Import Updates** (lines 1-7):
Remove FormDragDrop, JsonRequestMapper, JsonResponseMapper imports. Add import for SourceBodyForm from blocks folder.

**Form Rendering** (lines 115-142 and 144-171):
Current rendering shows heading, FormDragDrop component, and JsonRequestMapper with onProcessedData callback returning null.

Replace entire Request JSON Mapping section (lines 115-142) with single SourceBodyForm component:
- label prop set to "Request JSON Mapping"
- description prop set to "Upload JSON files to map request variables"
- mapperType prop set to "request"
- mappings prop can be undefined or pass from parent if needed
- Use default maxFiles and maxSize values

Replace entire Response JSON Mapping section (lines 144-171) with single SourceBodyForm component:
- label prop set to "Response JSON Mapping"
- description prop set to "Upload JSON files to map response variables"
- mapperType prop set to "response"
- mappings prop can be undefined or pass from parent if needed
- Use default maxFiles and maxSize values

**Result**: Form component reduced from approximately 194 lines to approximately 80 lines, with all JSON upload and mapping complexity moved to reusable SourceBodyForm component.

#### StatusCheckForm Changes

**Current State** (lines 8-18):
StatusCheckFormData interface contains statusUrl, checkMethod, headers, requestJsonFiles, responseJsonFiles, requestJsonData, responseJsonData, requestVariableMappings, and responseVariableMappings fields.

**Interface Simplification** (replace lines 8-18):
Remove all JSON-related fields. Keep only:
- statusUrl: string
- checkMethod: string
- headers: RequestHeader[]

**State Initialization** (lines 33-43):
Remove all JSON-related state initialization, keeping only statusUrl, checkMethod, and headers.

**Handler Removal** (lines 49-79):
Remove handlers: handleRequestJsonFilesChange, handleResponseJsonFilesChange, handleRequestJsonDataChange, handleResponseJsonDataChange, handleRequestVariableMappingsChange, handleResponseVariableMappingsChange.

**Import Updates** (lines 1-6):
Remove FormDragDrop, JsonRequestMapper, JsonResponseMapper imports. Add SourceBodyForm import.

**Form Rendering** (lines 102-129 and 131-158):
Replace Request JSON Mapping section (lines 102-129) with SourceBodyForm component using mapperType "request".
Replace Response JSON Mapping section (lines 131-158) with SourceBodyForm component using mapperType "response".

**Result**: Form component reduced from approximately 169 lines to approximately 60 lines.

#### DeliveryForm Changes

**Current State** (lines 8-18):
DeliveryFormData interface contains url, method, headers, requestJsonFiles, responseJsonFiles, requestJsonData, responseJsonData, requestVariableMappings, and responseVariableMappings fields.

**Interface Simplification** (replace lines 8-18):
Remove all JSON-related fields. Keep only:
- url: string
- method: string
- headers: RequestHeader[]

**State Initialization** (lines 33-43):
Remove all JSON-related state initialization, keeping only url, method, and headers.

**Handler Removal** (lines 49-79):
Remove all JSON-related handlers matching pattern from other forms.

**Import Updates** (lines 1-6):
Remove FormDragDrop, JsonRequestMapper, JsonResponseMapper imports. Add SourceBodyForm import.

**Form Rendering** (lines 103-130 and 132-159):
Replace Request JSON Mapping section with SourceBodyForm component using mapperType "request" and label "Delivery Request Mapping".
Replace Response JSON Mapping section with SourceBodyForm component using mapperType "response" and label "Delivery Response Mapping".

**Result**: Form component reduced from approximately 170 lines to approximately 60 lines.

### 6.5 Data Flow Architecture

#### User Upload Flow
User drags and drops or selects JSON file through FormDragDrop component inside SourceBodyForm. FormDragDrop component validates file format by checking file extension against accept prop which is set to ".json". FormDragDrop validates file size by checking file.size in bytes against maxSize prop converted to bytes (maxSize multiplied by 1024 multiplied by 1024). If validateJson prop is true, FormDragDrop reads file content using FileReader.readAsText, then attempts JSON.parse on the result. If parsing succeeds without throwing error, file is considered valid JSON. FormDragDrop calls onChange callback with array of File objects. If onJsonParsed callback is provided and JSON parsing succeeded, FormDragDrop calls onJsonParsed with three arguments: the File object, the parsed JavaScript object from JSON.parse, and the filename string from file.name property.

#### SourceBodyForm Processing Flow
When onJsonParsed callback fires in SourceBodyForm component, the handleJsonParsed function creates new JsonFile object. The id field is generated by concatenating fileName string with hyphen and current timestamp from Date.now(). The name field stores the fileName parameter. The file field stores reference to original File object. The data field stores the parsedData parameter which is the parsed JavaScript object. The size field stores file.size property. The lastModified field creates new Date object from file.lastModified timestamp. The propertiesCount field counts root properties using Object.keys on parsedData and getting array length. This new JsonFile object is appended to jsonFiles state array. If jsonFiles array was previously empty (length equals zero), the new file's id is automatically set as activeFileId to display it immediately.

#### Active File Management
SourceBodyForm maintains activeFileId state tracking which file is currently displayed. Component derives activeFile by calling find method on jsonFiles array matching file where file.id equals activeFileId. Component derives activeJsonData by accessing data property of activeFile or null if activeFile is undefined. When user selects different file from JsonSelector dropdown, onFileSelect callback updates activeFileId state to the selected fileId parameter. This state change triggers React re-render. Since activeJsonData is derived state, it automatically updates to the newly selected file's data property. The Mapper component receives new data through its data prop, which triggers the Mapper's useEffect hook watching data changes (line 72 in json-request-mapper.tsx and json-response-mapper.tsx). Mapper resets all processing states and re-runs entire processing pipeline with new file's data.

#### File Removal Flow
When user clicks remove button for a file in JsonSelector or ManageJsonsDialog, onFileRemove callback is triggered with the fileId parameter. SourceBodyForm's handleFileRemove function filters jsonFiles array removing the file where id equals fileId. If the removed fileId equals current activeFileId, function selects new active file by taking first element from remaining filtered array or setting to null if array is empty. Function also updates rawFiles array by mapping filtered jsonFiles array back to File objects through file property.

#### Mapper Processing Pipeline
When activeJsonData changes, appropriate mapper component (JsonRequestMapper or JsonResponseMapper based on mapperType prop) receives the data. The mapper runs sequential processing pipeline using multiple useEffect hooks:

Step 1 - JSON Processing (mapper line 93-100): When data prop changes, useEffect detects change and calls generatePathData function from mapping-utils.tsx passing the data parameter. Function returns object containing basePaths as Record mapping string keys to BasePathData objects, and fullPaths as array of FullPathData objects. These represent the complete hierarchical structure of the JSON with every path catalogued. The basePaths record contains paths for accordion nodes (objects and arrays). The fullPaths array contains every key and value node in the structure. Both are stored in component state and jsonProcessed flag set to true.

Step 2 - Variable Mapping (mapper line 103-112): When jsonProcessed is true and requestAndResponseVariableMappings prop exists, useEffect calls assignVariableToFullPath function passing mappings and fullPaths array. This function mutates fullPaths array in place, adding variableMappings property to each FullPathData object where a mapping exists. For each variable in the mappings configuration, it finds matching path in fullPaths and assigns the variable name. Sets mappingsProcessed flag to true.

Step 3 - Status Calculation (mapper line 115-124): When mappingsProcessed is true, useEffect calls getBasePathStatus function passing basePaths record and fullPaths array. This function analyzes the mapping status of each path and updates status property on both BasePathData and FullPathData objects. Status values are "mapped" when variable mapping exists, "issue" when mapping has problems, "recommended" for suggested mappings, or "default" for unmapped paths. Sets statusesProcessed flag to true.

Step 4 - Statistics Generation (mapper line 127-136): When statusesProcessed is true, useEffect calls getBasePathStatistics function passing basePaths and fullPaths. Function aggregates statistics across all paths counting how many have each status type. Updates statistics property on BasePathData objects with counts for mappings, recommendations, issues, and options. Sets statisticsProcessed flag to true.

Step 5 - Data Ready (mapper line 139-143): When statisticsProcessed is true, useEffect sets dataReady flag to true indicating all processing complete.

Step 6 - Statistics Calculation (mapper line 146-151): useMemo hook watching statisticsProcessed flag calls getVariableMappingStatistics function when ready. This function generates the MappingStatistics object with four categories: success containing count, label, and array of mapping objects with path and variable properties; issue with same structure; recommended with same structure; default with same structure. This statistics object is memoized to prevent recalculation on every render.

Step 7 - Render (mapper line 154-160): When dataReady is true and onProcessedData callback exists and statistics object exists, mapper calls onProcessedData function passing object containing basePaths, fullPaths, statistics, and dataReady boolean. The onProcessedData callback is expected to return ReactNode which mapper returns directly. If conditions not met, mapper returns div with text "Processing...".

#### Explorer Rendering
SourceBodyForm's onProcessedData callback returns JsonExplorer component. Explorer receives multiple props: data prop gets activeJsonData (original parsed JSON object), basePaths prop gets processedData.basePaths from mapper, fullPaths prop gets processedData.fullPaths from mapper, statistics prop gets processedData.statistics from mapper, mapperType prop gets the mapperType from SourceBodyForm props ("request" or "response"). Additionally, Explorer receives file management props: files prop gets jsonFiles array from SourceBodyForm state, currentFile prop gets activeFileId from SourceBodyForm state, onFileSelect prop gets handleFileSelect function from SourceBodyForm, onFileRemove prop gets handleFileRemove function from SourceBodyForm, onFileUpload prop gets handleFileUpload function from SourceBodyForm.

Explorer displays four main sections: ProgressBar showing statistics for active file with bars for success, issue, recommended, and default counts; header section with PathSearch component for searching paths in active file, PathsExpandSelector for expanding paths by status category, and JsonSelector dropdown populated with files array allowing user to switch between uploaded files; JsonRenderer displaying the active file's JSON structure with mapping status indicators, variable assignments, and interactive path selection. When user selects different file from JsonSelector dropdown, onFileSelect callback fires, SourceBodyForm updates activeFileId, derived activeJsonData changes, Mapper re-processes new file, Explorer re-renders with new file's data.

### 6.6 Implementation Steps

**Step 1: Create SourceBodyForm Component**
Create new file at path components/forms/blocks/source-body-form.tsx. Import JsonRequestMapper from components/json/request/json-request-mapper. Import JsonResponseMapper from components/json/response/json-response-mapper. Import JsonExplorer from components/json/interface/json-explorer. Import FormDragDrop from ./drag-drop-upload. Define SourceBodyFormProps interface with properties: label as string, description as optional string, mapperType as union type "request" or "response", requestType as union type "run_request" or "status_request" or "delivery_request", mappings as optional RequestAndResponseVariableMappings, onMappingsChange as optional function accepting mappings parameter, maxFiles as optional number defaulting to 5, maxSize as optional number defaulting to 10, className as optional string, collapsible as optional boolean, defaultExpanded as optional boolean. Define JsonFile interface with properties: id as string, name as string, file as File type, data as any type, size as number, lastModified as Date. Create functional component named SourceBodyForm accepting SourceBodyFormProps destructured props. Initialize state jsonFiles using useState with empty array of JsonFile type. Initialize state activeFileId using useState with null as initial value. Initialize state rawFiles using useState with empty array of File type. Create derived constant activeFile by calling jsonFiles.find with predicate checking if f.id equals activeFileId. Create derived constant activeJsonData by accessing activeFile?.data with null fallback using or operator. Define handleFilesChange callback using useCallback accepting files parameter of File array type, setting rawFiles state to files parameter. Define handleJsonParsed callback using useCallback accepting file of File type, parsedData of any type, and fileName of string type. Inside function, create newJsonFile object with id field as template literal concatenating fileName, hyphen, and Date.now() result, name field as fileName parameter, file field as file parameter, data field as parsedData parameter, size field as file.size property, lastModified field as new Date constructed from file.lastModified. Call setJsonFiles with prev parameter, creating updated array by spreading prev and appending newJsonFile. Check if prev.length equals zero and if true call setActiveFileId with newJsonFile.id to auto-select first uploaded file. Define handleFileSelect callback using useCallback accepting fileId parameter, calling setActiveFileId with fileId. Define handleFileRemove callback using useCallback accepting fileId parameter. Inside function, call setJsonFiles with updater filtering prev array removing file where id equals fileId. Check if activeFileId equals fileId and if true, select first file from filtered array or null if empty using optional chaining on filtered[0]?.id. Update rawFiles by mapping filtered array to file property. Define handleFileUpload callback for dialog upload integration. Create MapperComponent variable using ternary operator checking if mapperType equals "request" then JsonRequestMapper else JsonResponseMapper. Construct mappingsConfig object for mapper with structure matching RequestAndResponseVariableMappings interface including RequestResponseMappings from mappings prop or empty object, requestType from requestType prop, and inputType from mapperType prop. Return JSX with div container, header section with h3 showing label and optional p showing description, FormDragDrop component with props: label "Upload JSON Files", value as rawFiles, onChange as handleFilesChange, onJsonParsed as handleJsonParsed, accept as ".json", validateJson as true, multiple as true, maxFiles from props, maxSize from props. Add conditional rendering checking activeJsonData truthy, rendering MapperComponent with props: data as activeJsonData, requestAndResponseVariableMappings as mappingsConfig, onProcessedData as arrow function accepting processedData parameter returning JsonExplorer component with props: data as activeJsonData, basePaths as processedData.basePaths, fullPaths as processedData.fullPaths, statistics as processedData.statistics, mapperType from props, files as jsonFiles, currentFile as activeFileId, onFileSelect as handleFileSelect, onFileRemove as handleFileRemove, onFileUpload as handleFileUpload.

**Step 2: Enhance FormDragDrop Component**
Open file components/forms/blocks/drag-drop-upload.tsx. 

**Interface Changes** (after line 28):
Add validateJson optional boolean property and onJsonParsed optional function property with signature accepting file File, parsedData any, and fileName string parameters.

**New JSON Validation Function** (after validateFile around line 74):
Add async function validateJsonContent accepting file File parameter and returning Promise of object with isValid boolean, parsedData any, and optional error string. Function creates FileReader instance, sets up onload handler that attempts JSON.parse on result string, resolving with isValid true and parsedData on success or isValid false with error message on catch. Sets up onerror handler resolving with isValid false. Calls readAsText with file parameter.

**File Handling Enhancement** (modify handleFiles around line 76):
After basic validation loop collecting validFiles array, add conditional block when validateJson prop is true. Map validFiles to validateJsonContent promises. Use await Promise.all to get all validation results as array. Create new state to track validation results per file. Iterate validation results with index and for files with isValid true, call onJsonParsed callback if provided passing file, parsedData from result, and file.name. Update dragError state to include JSON validation errors from results with isValid false.

**UI Visual Enhancement** (completely restructure file list rendering around lines 230-260):
Replace basic file card structure with enhanced cards incorporating improved visual patterns.

For each file card container (currently lines 233-258):
- Change outer div from simple flex container to enhanced card with conditional classes:
  - Base: "flex items-center justify-between p-3 bg-muted rounded-lg border-2 transition-all"
  - When validated and valid: add "border-green-500 bg-green-50" classes
  - When validated and invalid: add "border-destructive bg-destructive/5" classes
  - Default (not validated): keep "border-transparent"
  - Add hover state: "hover:border-muted-foreground/30"

- Replace File icon (line 238) with conditional rendering:
  - When validated and valid: FileText icon with "text-green-600" class
  - When validated and invalid: X icon with "text-destructive" class
  - Default: existing File icon with "text-muted-foreground"

- Enhance file info section (lines 239-246):
  - Keep file name in first line with truncation
  - Keep formatted file size in second line
  - NEW: When validated and valid, add third line showing properties count as "X properties" in text-xs text-muted-foreground
  - NEW: Add validation status badge component:
    - For valid: Badge with variant="default" showing "Valid JSON" with Check icon
    - For invalid: Badge with variant="destructive" showing "Invalid JSON" with X icon

- Optional preview section:
  - When validated and valid, add expandable section below file info
  - Show truncated JSON preview in monospace font (first 100 characters)
  - Use bg-muted p-2 rounded text-xs classes
  - Add "Show more" button to expand full preview in dialog

- Enhance remove button (lines 249-256):
  - Keep existing ghost variant and icon-only button
  - Enhance hover state to "hover:bg-destructive/10 hover:text-destructive"
  - Ensure button remains visible in all card states

Add import for Check icon from lucide-react at top of file. Add import for Badge component from appropriate UI components folder if not already imported.

**Step 3: Enhance JsonExplorer Component**
Open file components/json/interface/json-explorer.tsx. In JsonExplorerProps interface after line 40, add five optional properties: files with JsonFile array type, currentFile with string or null type, onFileSelect function accepting fileId string, onFileRemove function accepting fileId string, and onFileUpload function accepting file File parameter. In component body, locate JsonSelector rendering at lines 406-418. Replace currentSource prop value from hardcoded "current-source" string to currentFile prop with fallback to undefined. Replace availableSources prop from empty array to conditional expression checking if files prop exists then mapping files array to objects with id from file.id, name from file.name, and description as template literal showing file.propertiesCount plus " properties" text, with fallback to empty array if files undefined. Replace onSourceChange prop from empty function to onFileSelect prop with fallback to empty function. Keep onUploadNew prop pointing to local setJsonDialogOpen function (unchanged). Replace onRemoveSource prop from empty function to onFileRemove prop with fallback to empty function. Replace files prop from empty array to files prop with fallback to empty array. Replace onFileSelect prop from empty function to onFileSelect prop with fallback to empty function. Replace onFileRemove prop from empty function to onFileRemove prop with fallback to empty function. Replace onFileUpload prop from empty function to onFileUpload prop with fallback to empty function. Keep openDialog and onDialogOpenChange pointing to local state (unchanged).

**Step 4: Update RunRequestForm**
Open file components/forms/run-request-form.tsx. At line 1, remove imports for FormDragDrop from drag-drop-upload, JsonRequestMapper from json request folder, and JsonResponseMapper from json response folder. Add import for SourceBodyForm from blocks/source-body-form. In RunRequestFormData interface around lines 9-23, remove all fields except headers, method, url, and loadBalancing. In state initialization around lines 38-49, remove all JSON-related initial values. Remove all handler functions between lines 55-91 except handleHeadersChange and handleLoadBalancingChange. In form return statement, locate Request JSON Mapping section starting at line 115. Delete entire section from opening div through closing div at line 142. Replace with single SourceBodyForm component tag with props: label equals "Request JSON Mapping", description equals "Upload JSON files to map request variables", mapperType equals "request", requestType equals "run_request". Locate Response JSON Mapping section starting at line 144. Delete entire section through closing div at line 171. Replace with single SourceBodyForm component tag with props: label equals "Response JSON Mapping", description equals "Upload JSON files to map response variables", mapperType equals "response", requestType equals "run_request".

**Step 5: Update StatusCheckForm**
Open file components/forms/status-check-form.tsx. Apply same import changes as InferenceForm at lines 1-6 removing FormDragDrop, JsonRequestMapper, JsonResponseMapper and adding SourceBodyForm. In StatusCheckFormData interface around lines 8-18, remove all JSON-related fields keeping only statusUrl, checkMethod, and headers. In state initialization around lines 33-43, remove JSON-related values. Remove JSON handler functions between lines 49-79. In form rendering, replace Request JSON Mapping section (lines 102-129) with SourceBodyForm component with props: label equals "Request JSON Mapping", description equals "Upload JSON files to map status check request variables", mapperType equals "request", requestType equals "status_request". Replace Response JSON Mapping section (lines 131-158) with SourceBodyForm component with props: label equals "Response JSON Mapping", description equals "Upload JSON files to map status check response variables", mapperType equals "response", requestType equals "status_request".

**Step 6: Update DeliveryForm**
Open file components/forms/delivery-form.tsx. Apply same import changes at lines 1-6 removing FormDragDrop, JsonRequestMapper, JsonResponseMapper and adding SourceBodyForm. In DeliveryFormData interface around lines 8-18, remove JSON-related fields keeping url, method, and headers. In state initialization around lines 33-43, remove JSON-related values. Remove JSON handler functions between lines 49-79. Replace Request JSON Mapping section (lines 103-130) with SourceBodyForm component with props: label equals "Delivery Request Mapping", description equals "Upload JSON files to map delivery request variables", mapperType equals "request", requestType equals "delivery_request". Replace Response JSON Mapping section (lines 132-159) with SourceBodyForm component with props: label equals "Delivery Response Mapping", description equals "Upload JSON files to map delivery response variables", mapperType equals "response", requestType equals "delivery_request".

### 6.7 Testing Requirements

**FormDragDrop JSON Validation**:
Test file format validation with non-JSON files expecting rejection. Test file size validation with files exceeding maxSize expecting rejection. Test JSON validation with valid JSON file expecting onJsonParsed callback to fire with correctly parsed object. Test JSON validation with invalid JSON syntax expecting error state and no onJsonParsed callback. Test multiple file upload with mix of valid and invalid JSON expecting only valid files to trigger callbacks.

**SourceBodyForm Multi-File Management**:
Test uploading first file expecting automatic selection as active file. Test uploading multiple files expecting all files stored in jsonFiles array. Test file selection from JsonSelector expecting activeFileId update and explorer showing correct file's data. Test file removal when file is not active expecting file removed from array without affecting active file. Test file removal when file is active expecting next file in array to become active automatically. Test file removal when only one file exists expecting activeFileId to become null and explorer to hide.

**Mapper Integration**:
Test request mapperType expecting JsonRequestMapper to receive data and process through pipeline. Test response mapperType expecting JsonResponseMapper to receive data and process through pipeline. Test mapper receiving null data expecting "Processing..." message display. Test mapper receiving valid parsed object expecting sequential execution of five processing steps with flags updating correctly. Test mapper processing completion expecting onProcessedData callback to fire with basePaths, fullPaths, statistics, and dataReady true.

**Explorer File Switching**:
Test switching between files in JsonSelector expecting explorer to update with new file's basePaths, fullPaths, and statistics. Test that ProgressBar statistics update correctly when switching files. Test that PathSearch results update to match active file's paths. Test that JsonRenderer displays correct JSON structure for active file. Test that mapping status indicators update to match active file's mappings.

**Form Component Integration**:
Test RunRequestForm with SourceBodyForm for both request and response sections. Test StatusCheckForm with SourceBodyForm for both request and response sections. Test DeliveryForm with SourceBodyForm for both request and response sections. Test that forms maintain clean separation between form metadata (method, url, headers) and JSON mapping concerns. Test form submission includes all necessary data from SourceBodyForm components.

**Terminology Consistency**:
Test that RunRequestForm component name matches run_request_template terminology in data files. Verify all JSX usages work correctly with RunRequestForm component name in app/sources/[id]/page.tsx and app/components-showcase/forms/page.tsx.

## Success Criteria

1. **Functionality**: Robust forms with comprehensive validation
2. **Performance**: Smooth interactions and efficient validation
3. **Usability**: Intuitive form interactions requiring minimal training
4. **Accessibility**: Full WCAG 2.1 AA compliance
5. **Responsiveness**: Seamless experience across all device sizes
6. **Integration**: Easy integration with existing components


## Phase 7: SourceBodyForm Card-like Component with Status Shadows

### Overview
Transform the existing SourceBodyForm into a card-based JsonBodyUpload component that provides unified text input and file upload functionality with status-based visual feedback and seamless mode switching.

### Executive Summary

**Goal**: Create a unified JsonBodyUpload component that combines text input and file upload in a single card interface with status-based shadows and visual feedback.

**Solution**: Enhance SourceBodyForm to include card wrapper with status shadows, unified interaction modes, and "Setup Mappings" button for mapper activation.

**Key Features**:
1. **Card-Based Design**: Wrapped in card with inner (inside) drop shadows that change color based on validation status
2. **Unified Interface**: No toggle buttons - single component handling both text input and file upload
3. **Three Interaction Modes**: Upload button click, drag & drop, and text input activation
4. **Status-Based Styling**: Visual feedback through shadow colors (default, success, warning, error)
5. **Setup Mappings Button**: Triggers transition to mapper/explorer view when valid JSON present

**User Experience**:
User sees card with upload area → Clicks upload button OR drags file OR clicks text area → JSON validated with visual feedback after leaving the component → "Setup Mappings" button appears → User clicks to enter mapper mode → Seamless transition between text and file modes

**Technical Approach**:
- Enhance SourceBodyForm with borders similar to cards and status shadow system
- Add text input mode that activates when clicking outside upload button
- Implement status determination logic based on JSON validation results
- Add "Setup Mappings" button that shows/hides based on valid JSON presence across the text or uploaded files list + add a text button "Back to edit"
- Maintain existing multi-file management and mapper integration

### 7.1 SourceBodyForm Component Enhancement

**File**: `components/forms/blocks/source-body-form.tsx` (ENHANCE)

**Current State**: Basic form component with FormDragDrop and conditional mapper/explorer rendering (renamed from source-body-form.tsx).

**Enhancement Requirements**:

**Container Wrapper Implementation**:
- Wrap entire component with `border border-border rounded-lg` and status-based shadows
- Use existing shadow utility classes: `shadow-default`, `shadow-success`, `shadow-warning`, `shadow-destructive`
- Implement status determination logic based on JSON validation state
- Add hover states using `hover:shadow-hover-primary` pattern from existing components
- Apply smooth transitions using `transition-shadow duration-200` from existing animation system

**Status System Integration**:
- **Default**: Neutral shadow (no content, no validation)
- **Success**: Green shadow (valid JSON, ready for mapping)
- **Warning**: Yellow shadow (validation issues detected)
- **Error**: Red shadow (critical errors preventing submission)

**Unified Interaction Modes**:
- **Upload Button Click**: Opens system file picker dialog
- **Drag & Drop**: File drops anywhere on component area
- **Text Input**: Clicking anywhere outside upload button enables textarea mode

**Content Rules by Mapper Type**:
- **Request Type**: `RequestMetadataForm` + `SourceBodyForm` with request mapper (no action button inside)
- **Response Type**: `SourceBodyForm` with response mapper + integrated "Get response by sending request" button
- **Result Type**: `SourceBodyForm` with result mapper + integrated "Get results by sending set up requests" button

**Integrated Action Buttons** (within SourceBodyForm):
- **Response Type**: Show "Get response by sending request" button when `mapperType="response"`
- **Result Type**: Show "Get results by sending set up requests" button when `mapperType="result"`
- **Request Type**: No action button inside SourceBodyForm
- Buttons appear after file upload area, before mapper/explorer
- Use `PrimaryButton` component with default styling
- Center positioning within the card layout

**Setup Mappings Button**:
- Appears when valid JSON is present
- Triggers transition to mapper/explorer view
- Positioned prominently within card interface
- Use `PrimaryButton` component with default styling
- Center positioning within the card layout

**Visual Enhancement**:
- Enhanced file cards with validation status indicators using `BaseBadge` component
- Success/error state visual feedback using existing shadow utilities
- Smooth animations using `animationClasses.default` from existing animation system
- Consistent styling with existing design system using `BaseCard` patterns
- Use existing border utilities: `border border-border rounded-lg`
- Apply `bg-content-component text-content-component-foreground` for card styling

### 7.2 Status Shadow System Implementation

**Shadow Classes** (inspired by JsonAccordion):
- **Default**: `shadow-default` (neutral, no validation)
- **Success**: `shadow-success` (green, all validations pass)
- **Warning**: `shadow-warning` (yellow, validation issues)
- **Error**: `shadow-destructive` (red, critical errors)

**Hover State Shadow Classes**:
- **Default Hover**: `shadow-primary` (blue, interactive state)
- **Success Hover**: `shadow-primary` (blue, interactive state)
- **Warning Hover**: `shadow-primary` (blue, interactive state)
- **Error Hover**: `shadow-primary` (blue, interactive state)

**Status Determination Logic**:
will be done by the validation utils (to be specified later on)

**Animation System**:
- Use `animationClasses.default` for consistent animations across the component
- Smooth transitions between status changes using `transition-shadow duration-200`
- Hover state enhancements with `hover:shadow-hover-primary` pattern
- Loading state indicators using existing `Loader2` component from lucide-react
- Success/error state animations using existing animation patterns
- Hover state works with cursor or file to be dropped using existing hover detection

### 7.3 Custom Components and Design System Integration

**Button Components** (integrated into SourceBodyForm):
- **Setup Mappings Button**: Use `PrimaryButton` component with default styling
- **Back to Edit Button**: Use `TextButton` component for secondary action
- **File Upload Button**: Use `SecondaryButton` component for file upload actions
- **Response Action Button**: Use `PrimaryButton` with text "Get response by sending request" (integrated in SourceBodyForm)
- **Result Action Button**: Use `PrimaryButton` with text "Get results by sending set up requests" (integrated in SourceBodyForm)

**Badge Components**:
- **Validation Status**: Use `BaseBadge` component with appropriate variants
- **File Status Indicators**: Use `BaseBadge` with `variant="default"` for success, `variant="destructive"` for errors

**Container Styling**:
- **Main Container**: Use `border border-border rounded-lg` with status-based shadows
- **File Cards**: Use existing card patterns with `bg-content-component` and `text-content-component-foreground`

**Design Tokens**:
- **Shadows**: `shadow-default`, `shadow-success`, `shadow-warning`, `shadow-destructive`, `shadow-hover-primary`
- **Colors**: Use existing CSS variables: `--success`, `--warning`, `--destructive`, `--primary`
- **Borders**: `border border-border rounded-lg` for consistent border styling
- **Backgrounds**: `bg-content-component` for card backgrounds, `bg-background` for main areas
- **Text**: `text-content-component-foreground` for card text, `text-muted-foreground` for descriptions

**Animation Classes**:
- **Transitions**: `animationClasses.default` for consistent animation timing
- **Shadow Transitions**: `transition-shadow duration-200` for smooth shadow changes
- **Hover Effects**: `hover:shadow-hover-primary` for interactive feedback

### 7.4 JsonResultMapper Integration for Objects and Datapoint Mappings

**JsonResultMapper Requirements**:
- **Current Interface**: Expects `objectsAndDatapointsMappings` prop with different structure
- **Integration Need**: Extend `SourceBodyForm` to support result mapper type
- **Mapping Types**: Objects and datapoint mappings (different from request/response variable mappings)

**Required Extensions**:
- **SourceBodyForm**: Add support for `mapperType: 'result'` with `JsonResultMapper`
- **Mapping Interface**: Extend interface to handle both mapping types
- **Data Flow**: Ensure `JsonResultMapper` receives proper `objectsAndDatapointsMappings` data
- **Explorer Integration**: `JsonExplorer` should work with result mapper's different data structure

**Implementation Approach**:
- Extend `SourceBodyForm` to conditionally render `JsonResultMapper` when `mapperType="result"`
- Extend existing interfaces to accept `objectsAndDatapointsMappings`
- Ensure proper data flow from `SourceBodyForm` → `JsonResultMapper` → `JsonExplorer`
- Maintain existing request/response functionality while adding result support

## Phase 8: Source Request Response Accordion Component

### Overview
Implement SourceRequestResponseAccordion component that renders different mapper types based on the accordion type prop, with status-based shadows and visual dividers.

### Executive Summary

**Goal**: Create a flexible accordion component that renders different mapper types (request, response, result) based on the accordion type prop.

**Solution**: Build SourceRequestResponseAccordion that accepts accordion type and renders the appropriate SourceBodyForm component with the correct mapper type.

**Key Features**:
1. **Component Integration**: Uses `BaseAccordion` + multiple components based on type:
   - **Request**: `RequestMetadataForm` + `SourceBodyForm` (request mapper)
   - **Response**: `SourceBodyForm` (response mapper) 
   - **Result**: `SourceBodyForm` (result mapper)
2. **Status Propagation**: Child component statuses bubble up to accordion level
3. **Visual Dividers**: Clear separation between sections using `Divider` component
4. **Status-Based Shadows**: Accordion shadows reflect child component validation state
5. **Flexible Content**: Accordion content determined by type prop

**User Experience**:
Accordion receives type prop → Renders SourceBodyForm with appropriate mapper → Status indicators show validation state → Visual dividers separate sections clearly → Status bubbles up to accordion level

**Technical Approach**:
- Create self-contained accordion component that accepts only type prop
- Internal logic determines title, description, and SourceBodyForm configuration
- Implement status propagation from child components
- Add visual dividers between sections
- Organize content with proper spacing and typography
- Integrate with existing shadow utility system

**Accordion Types**:
- **Request Type**: Renders SourceBodyForm with request mapper
- **Response Type**: Renders SourceBodyForm with response mapper  
- **Result Type**: Renders SourceBodyForm with result mapper

**Future Integration Note**:
- Forms will integrate accordions in next phase

### 8.1 SourceRequestResponseAccordion Component

**File**: `components/accordions/source-request-response-accordion.tsx` (ENHANCE)

**Current State**: Basic accordion wrapper using `BaseAccordion` with empty content.

**Enhancement Requirements**:

**Accordion Props**:
- Accept `type` prop with values: "request", "response", "result"
- All other logic handled internally by the accordion component
- Use `BaseAccordion` component as foundation

**Accordion Structure**:
- Single accordion section with type-specific content using `BaseAccordion`
- `RequestMetadataForm` component (Request type only)
- `PrimaryButton` component (Response and Result types only)
- Content area that renders either `SourceBodyForm` or mapper components based on data state
- `Divider` component between sections for visual separation
- Status-based shadow system using existing shadow utilities
- Content and configuration determined internally by type prop

**Internal Logic**:
- **Request Type**: 
  - Title: "Request"
  - Description: "Configure request metadata and upload JSON files to map request variables"
  - Components: `RequestMetadataForm` + `SourceBodyForm` with request mapper (no action button inside)
- **Response Type**: 
  - Title: "Response"
  - Description: "Upload JSON files to map response variables"
  - Components: `SourceBodyForm` with response mapper (includes "Get response by sending request" button)
- **Result Type**: 
  - Title: "Results"
  - Description: "Upload JSON files to map result variables"
  - Components: `SourceBodyForm` with result mapper (includes "Get results by sending set up requests" button)

**Status Integration**:
- Monitor child component validation states from `SourceBodyForm`
- Propagate status to accordion level using existing shadow utilities
- Apply appropriate shadow classes: `shadow-default`, `shadow-success`, `shadow-warning`, `shadow-destructive`
- Handle status changes dynamically with `hover:shadow-hover-primary` pattern

**Content Organization**:
- Clear section headers and descriptions
- Proper spacing between sections
- Visual hierarchy with typography
- Responsive layout considerations

**Visual Dividers**:
- Horizontal dividers between sections
- Subtle styling that doesn't interfere with content
- Consistent with design system
- Responsive behavior

### 8.2 Visual Dividers and Section Organization

**Divider Implementation**:
- Use `Divider` component with `orientation="horizontal"`
- Apply `color="border"` and `opacity={0.5}` for subtle styling
- Use `firstSpacing="10%"` and `lastSpacing="10%"` for consistent spacing
- Ensure responsive behavior with existing component

**Section Organization**:
- Standardized spacing and alignment used in other areas of the codebase

**Content Structure**:
- Request section: SourceBodyForm with request mapping
- Response section: SourceBodyForm with response mapping
- Result section: SourceBodyForm with result mapping
- Status indicators for each section
- Clear visual separation

## Phase 9: SourceBodyUpload Component Refactoring and Form Integration

### Overview
Refactor the existing `SourceBodyForm` component (renamed to `SourceBodyUpload`) to be a pure file upload component that only handles JSON file upload and text input, removing all mapper integration. Move mapper orchestration to the parent form components (`RunRequestForm`, `StatusCheckForm`, `DeliveryForm`).

### Executive Summary

**Goal**: Transform `SourceBodyForm` into a focused `SourceBodyUpload` component that only handles file upload and text input, with parent forms orchestrating mapper integration.

**Solution**: 
1. **Simplify SourceBodyUpload**: Remove all mapper components, keep only file upload + text input
2. **Add Data Callbacks**: Provide `onJsonDataChange` callback to pass data to parent forms
3. **Update Form Components**: Add mapper orchestration logic to each form component
4. **Fix Import References**: Update all imports from `source-body-form` to `source-body-upload`

**Key Features**:
1. **Pure Upload Component**: Only handles file upload, text input, and JSON parsing
2. **Data Callback System**: Passes JSON data to parent forms via `onJsonDataChange`
3. **Form Orchestration**: Parent forms handle mapper integration and data flow
4. **Simplified Interface**: Clean, focused component with minimal props
5. **Backward Compatibility**: Maintains existing functionality while simplifying architecture

**User Experience**:
User uploads files or enters text → SourceBodyUpload validates and parses JSON → Data passed to parent form → Parent form orchestrates mapper integration → Mapper processes data → Results displayed in form

**Technical Approach**:
- Remove all mapper imports and components from SourceBodyUpload
- Add `onJsonDataChange` callback prop for data communication
- Update form components to handle mapper orchestration
- Fix all import references from old component name
- Maintain existing file upload and text input functionality

### 9.1 SourceBodyUpload Component Refactoring

**File**: `components/forms/blocks/source-body-upload.tsx` (REFACTOR)

**Current State**: Complex component with mapper integration, file management, and text input.

**Refactoring Requirements**:

**Remove Mapper Integration**:
- Remove imports: `JsonRequestMapper`, `JsonResponseMapper`, `JsonResultMapper`, `JsonExplorer`
- Remove mapper-related props: `mapperType`, `requestType`, `mappings`, `onMappingsChange`
- Remove mapper state and logic
- Remove mapper rendering and conditional logic

**Simplify Interface**:
```typescript
interface SourceBodyUploadProps {
  label: string
  description?: string
  maxFiles?: number
  maxSize?: number
  className?: string
  // NEW: Data callback for parent forms
  onJsonDataChange?: (data: {
    files: JsonFile[]
    activeFile: JsonFile | null
    textData: any | null
    combinedData: any | null
  }) => void
}
```

**Keep Core Functionality**:
- File upload with drag & drop
- Text input with JSON parsing
- File management (add, remove, select)
- JSON validation and parsing
- Visual feedback and status indicators

**Remove Action Buttons**:
- Remove "Get Response" button (moved to form/accordion)
- Remove "Generate Response" button (moved to form/accordion)
- Remove all mapper-related action buttons
- Keep only "Upload JSON" and "Setup Mappings" buttons

**Add Data Callback**:
- Call `onJsonDataChange` whenever JSON data changes
- Pass current files, active file, text data, and combined data
- Allow parent forms to receive and process the data

**Maintain Visual Design**:
- Keep existing card-based design with status shadows
- Keep file badges and visual feedback (using new FileBadge component)
- Keep text input with JSON syntax highlighting
- Keep drag & drop functionality

### 9.2 Form Component Updates

**Files to Update**:
- `components/forms/run-request-form.tsx`
- `components/forms/status-check-form.tsx` 
- `components/forms/delivery-form.tsx`

**Import Updates**:
```typescript
// OLD
import { SourceBodyForm } from "./blocks/source-body-upload"

// NEW  
import { SourceBodyUpload } from "./blocks/source-body-upload"
```

**Add Mapper Integration**:
Each form component needs to:
1. Import required mapper components
2. Add mapper state management
3. Handle `onJsonDataChange` callbacks
4. Render mappers when JSON data is available
5. Pass processed data to mappers

**RunRequestForm Updates**:
```typescript
// Add imports
import { JsonRequestMapper } from "@/components/json/request/json-request-mapper"
import { JsonResponseMapper } from "@/components/json/response/json-response-mapper"
import { JsonExplorer } from "@/components/json/interface/json-explorer"

// Add state for JSON data
const [requestJsonData, setRequestJsonData] = useState<any>(null)
const [responseJsonData, setResponseJsonData] = useState<any>(null)
const [requestMappings, setRequestMappings] = useState<any>(null)
const [responseMappings, setResponseMappings] = useState<any>(null)

// Add handlers
const handleRequestJsonDataChange = (data: any) => {
  setRequestJsonData(data.combinedData)
}

const handleResponseJsonDataChange = (data: any) => {
  setResponseJsonData(data.combinedData)
}

// Add action button handlers
const handleGetResponse = () => {
  // Handle get response logic
  console.log("Get response clicked")
}

// Update JSX to include mappers and action buttons
<SourceBodyUpload
  label="Request JSON Mapping"
  description="Upload JSON files to map request variables"
  onJsonDataChange={handleRequestJsonDataChange}
/>

{requestJsonData && (
  <JsonRequestMapper
    data={requestJsonData}
    requestAndResponseVariableMappings={requestMappings}
    onProcessedData={(processedData) => (
      <JsonExplorer
        data={requestJsonData}
        basePaths={processedData.basePaths}
        fullPaths={processedData.fullPaths}
        statistics={processedData.statistics}
        mapperType="request"
      />
    )}
  />
)}

<SourceBodyUpload
  label="Response JSON Mapping"
  description="Upload JSON files to map response variables"
  onJsonDataChange={handleResponseJsonDataChange}
/>

{responseJsonData && (
  <div className="space-y-4">
    {/* Action Button - controlled by form */}
    <div className="flex justify-center">
      <PrimaryButton onClick={handleGetResponse}>
        Get Response
      </PrimaryButton>
    </div>
    
    <JsonResponseMapper
      data={responseJsonData}
      requestAndResponseVariableMappings={responseMappings}
      onProcessedData={(processedData) => (
        <JsonExplorer
          data={responseJsonData}
          basePaths={processedData.basePaths}
          fullPaths={processedData.fullPaths}
          statistics={processedData.statistics}
          mapperType="response"
        />
      )}
    />
  </div>
)}
```

**StatusCheckForm Updates**:
Similar pattern with request and response mappers for status check functionality, with "Get Status" action button controlled by the form.

**DeliveryForm Updates**:
Similar pattern with request and response mappers for delivery functionality, with "Get Delivery" action button controlled by the form.

**Accordion Integration**:
The `SourceRequestResponseAccordion` will also control action buttons:
- **Response Type**: "Get response by sending request" button
- **Result Type**: "Get results by sending set up requests" button
- **Request Type**: No action button (only metadata + upload)

### 9.3 Data Flow Architecture

**SourceBodyUpload → Form Component**:
1. User uploads files or enters text
2. SourceBodyUpload validates and parses JSON
3. `onJsonDataChange` callback fires with data
4. Form component receives data and updates state
5. Form component renders appropriate mapper
6. Mapper processes data and renders explorer

**Form Component → Mapper**:
1. Form component passes JSON data to mapper
2. Mapper processes data through pipeline
3. Mapper calls `onProcessedData` with results
4. Form component renders `JsonExplorer` with processed data
5. Explorer displays interactive JSON structure

**Data Structure**:
```typescript
interface JsonDataChangeEvent {
  files: JsonFile[]           // All uploaded files
  activeFile: JsonFile | null // Currently selected file
  textData: any | null        // Parsed text input data
  combinedData: any | null    // Combined data from files + text
}
```

### 9.4 Import Reference Updates

**Files to Update**:
- `components/forms/run-request-form.tsx`
- `components/forms/status-check-form.tsx`
- `components/forms/delivery-form.tsx`
- `components/accordions/source-request-response-accordion.tsx`

**Search and Replace**:
```typescript
// OLD
import { SourceBodyForm } from "./blocks/source-body-upload"

// NEW
import { SourceBodyUpload } from "./blocks/source-body-upload"

// OLD
<SourceBodyForm

// NEW
<SourceBodyUpload
```

**Component Usage Updates**:
- Update all JSX usage from `<SourceBodyForm` to `<SourceBodyUpload`
- Update prop names to match new interface
- Add `onJsonDataChange` callbacks where needed

### 9.5 Implementation Steps

**Step 1: Refactor SourceBodyUpload Component**
1. Remove all mapper imports and components
2. Simplify interface to only include upload-related props
3. Add `onJsonDataChange` callback prop
4. Remove mapper rendering logic
5. Keep file upload and text input functionality
6. Test component in isolation

**Step 2: Update Form Components**
1. Update imports from `SourceBodyForm` to `SourceBodyUpload`
2. Add mapper imports (`JsonRequestMapper`, `JsonResponseMapper`, `JsonExplorer`)
3. Add JSON data state management
4. Add `onJsonDataChange` handlers
5. Add mapper rendering logic
6. Test each form component

**Step 3: Update Accordion Component**
1. Update import reference
2. Update component usage
3. Test accordion functionality

**Step 4: Testing and Validation**
1. Test file upload functionality
2. Test text input functionality
3. Test data flow from upload to mappers
4. Test mapper integration in forms
5. Test accordion integration

### 9.6 Benefits of Refactoring

**Simplified Architecture**:
- SourceBodyUpload focuses only on file upload and text input
- Form components handle their specific mapper requirements
- Clear separation of concerns
- Easier to maintain and test

**Improved Reusability**:
- SourceBodyUpload can be used in any context needing JSON upload
- Form components can choose their own mapper strategy
- More flexible component composition

**Better Data Flow**:
- Explicit data callbacks instead of internal state management
- Parent components control data processing
- Clearer data flow and debugging

**Reduced Complexity**:
- Smaller, focused components
- Less internal state management
- Easier to understand and modify

### 9.7 Success Criteria

1. **Functionality**: All existing functionality preserved
2. **Architecture**: Clean separation between upload and mapper concerns
3. **Data Flow**: Explicit data callbacks working correctly
4. **Reusability**: SourceBodyUpload can be used independently
5. **Maintainability**: Easier to modify and extend components
6. **Testing**: Each component can be tested in isolation

---

## Phase 10: Form Structure Implementation

### Overview
Restructure all form components to follow the correct page structure pattern where forms contain SourceRequestResponseAccordion components instead of individual form blocks.

### Correct Form Structure Pattern

#### Page Structure:
```
TabsContent
└── Form Component (e.g., RunRequestForm)
    ├── Form Headline/Title
    ├── SourceRequestResponseAccordion type="request"
    │   ├── RequestMetadataForm (from blocks)
    │   └── SourceBodyUpload (from blocks)
    └── SourceRequestResponseAccordion type="response"
        ├── "Get Response/Result" Button (already in accordion)
        └── SourceBodyUpload (from blocks)
```

#### Key Points:
1. **Tab** contains **Form(s)**
2. **Form** includes **headline + SourceRequestResponseAccordion components**
3. **SourceRequestResponseAccordion** already handles:
   - Request accordion: RequestMetadataForm + SourceBodyUpload
   - Response accordion: "Get Response/Result" button + SourceBodyUpload
   - Result accordion: "Get Results" button + SourceBodyUpload
4. **"Save" button** goes in **PageHeader's actionButtons** (top right corner)
5. **"Get Response/Result" button** is already built into the accordion

### Forms to Restructure

#### 1. RunRequestForm
**Current Structure:**
- Form container with individual components stacked vertically
- RequestMetadataForm for method, URL, headers
- Two separate SourceBodyUpload components for request and response JSON
- LoadBalancingForm for concurrency and timeout
- Submit button at the bottom right

**Target Structure:**
- Subheadline component with "Run Request Configuration"
- Description component with explanatory text
- Request accordion containing:
  - RequestMetadataForm (method, URL, headers)
  - SourceBodyUpload OR JsonRequestMapper (based on file availability)
  - LoadBalancingForm (concurrency, timeout)
- Response accordion containing:
  - PrimaryButton "Get Response" (in accordion actions)
  - SourceBodyUpload OR JsonResponseMapper (based on file availability)
- No submit button (moved to page header)

#### 2. StatusCheckForm
**Current Structure:**
- Form container with individual components stacked vertically
- RequestMetadataForm for status check method, URL, headers
- Two separate SourceBodyUpload components for request and response JSON
- Submit button at the bottom right

**Target Structure:**
- Subheadline component with "Status Check Configuration"
- Description component with explanatory text
- Request accordion containing:
  - RequestMetadataForm (method, URL, headers)
  - SourceBodyUpload OR JsonRequestMapper (based on file availability)
- Response accordion containing:
  - PrimaryButton "Get Response" (in accordion actions)
  - SourceBodyUpload OR JsonResponseMapper (based on file availability)
- No submit button (moved to page header)

#### 3. DeliveryForm
**Current Structure:**
- Form container with individual components stacked vertically
- RequestMetadataForm for delivery method, URL, headers
- Two separate SourceBodyUpload components for request and response JSON
- Submit button at the bottom right

**Target Structure:**
- Subheadline component with "Delivery Configuration"
- Description component with explanatory text
- Request accordion containing:
  - RequestMetadataForm (method, URL, headers)
  - SourceBodyUpload OR JsonRequestMapper (based on file availability)
- Response accordion containing:
  - PrimaryButton "Get Response" (in accordion actions)
  - SourceBodyUpload OR JsonResultMapper (based on file availability)
- No submit button (moved to page header)

### Component Mapping Details

#### Conditional Rendering Logic:
- **When no files uploaded**: Show SourceBodyUpload component
- **When files are available**: Show appropriate Mapper component

#### Mapper Component Usage:
- **JsonRequestMapper**: Used for request JSON mapping in all forms
- **JsonResponseMapper**: Used for response JSON mapping in RunRequestForm and StatusCheckForm  
- **JsonResultMapper**: Used for response JSON mapping in DeliveryForm (handles results instead of responses)

#### Source Delivery Type Parameterization:
Forms will accept a `sourceDeliveryType` prop with values: `"endpoint" | "webhook" | "response"`

**Source Delivery Type Behavior:**
- **endpoint**: Shows full forms as defined (all accordions visible)
- **webhook**: Hides request accordion from DeliveryForm only
- **response**: Hides response accordion from RunRequestForm AND request accordion from DeliveryForm

#### Component Hierarchy by Source Delivery Type:

**Endpoint Source Delivery Type (Full Forms):**
```
Form Component
├── Subheadline (form title)
├── Description (explanatory text)
├── Request Accordion
│   ├── RequestMetadataForm (method, URL, headers)
│   ├── SourceBodyUpload OR JsonRequestMapper (conditional)
│   └── LoadBalancingForm (RunRequestForm only)
└── Response Accordion
    ├── PrimaryButton "Get Response" (in accordion actions)
    └── SourceBodyUpload OR JsonResponseMapper/JsonResultMapper (conditional)
```

**Webhook Source Delivery Type:**
```
RunRequestForm (unchanged)
├── Request Accordion
└── Response Accordion

StatusCheckForm (unchanged)
├── Request Accordion
└── Response Accordion

DeliveryForm (modified)
├── Subheadline (form title)
├── Description (explanatory text)
└── Response Accordion (request accordion hidden)
    ├── PrimaryButton "Get Response" (in accordion actions)
    └── SourceBodyUpload OR JsonResultMapper (conditional)
```

**Response Source Delivery Type:**
```
RunRequestForm (modified)
├── Subheadline (form title)
├── Description (explanatory text)
├── Request Accordion (response accordion hidden)
│   ├── RequestMetadataForm (method, URL, headers)
│   ├── SourceBodyUpload OR JsonRequestMapper (conditional)
│   └── LoadBalancingForm (concurrency, timeout)

StatusCheckForm (unchanged)
├── Request Accordion
└── Response Accordion

DeliveryForm (modified)
├── Subheadline (form title)
├── Description (explanatory text)
└── Response Accordion (request accordion hidden)
    ├── PrimaryButton "Get Response" (in accordion actions)
    └── SourceBodyUpload OR JsonResultMapper (conditional)
```

### Implementation Steps

#### Step 1: Update RunRequestForm
1. Add `sourceDeliveryType` prop with type `"endpoint" | "webhook" | "response"`
2. Remove individual form blocks
3. Add Subheadline and Description components
4. Replace with SourceRequestResponseAccordion components
5. Add LoadBalancingForm in BaseAccordion (always visible)
6. Implement conditional rendering based on sourceDeliveryType:
   - **endpoint/webhook**: Show both Request and Response accordions
   - **response**: Show only Request accordion (hide Response accordion)
7. Remove submit button (will be in PageHeader)
8. Update form data structure to match accordion props
9. Add proper callback handlers

#### Step 2: Update StatusCheckForm
1. Add `sourceDeliveryType` prop with type `"endpoint" | "webhook" | "response"`
2. Remove individual form blocks
3. Add Subheadline and Description components
4. Replace with SourceRequestResponseAccordion components
5. Implement conditional rendering based on sourceDeliveryType:
   - **All source delivery types**: Show both Request and Response accordions (unchanged)
6. Remove submit button (will be in PageHeader)
7. Update form data structure to match accordion props
8. Add proper callback handlers

#### Step 3: Update DeliveryForm
1. Add `sourceDeliveryType` prop with type `"endpoint" | "webhook" | "response"`
2. Remove individual form blocks
3. Add Subheadline and Description components
4. Replace with SourceRequestResponseAccordion components
5. Implement conditional rendering based on sourceDeliveryType:
   - **endpoint**: Show both Request and Response accordions
   - **webhook**: Show only Response accordion (hide Request accordion)
   - **response**: Show only Response accordion (hide Request accordion)
6. Remove submit button (will be in PageHeader)
7. Update form data structure to match accordion props
8. Add proper callback handlers

#### Step 4: Implement Upload/Mapper Orchestration
Each form needs to orchestrate between SourceBodyUpload and mapper components:

**Orchestration Logic:**
- **When no files uploaded**: Show SourceBodyUpload component
- **When files are available**: Show appropriate mapper component with JSON data

**Data Flow Pattern:**
```typescript
// State management for orchestration
const [jsonData, setJsonData] = useState<any>(null)
const [hasFiles, setHasFiles] = useState(false)

// Handle data from SourceBodyUpload
const handleJsonDataChange = (data: {
  files: JsonFile[]
  activeFile: JsonFile | null
  textData: any | null
  combinedData: any | null
}) => {
  setJsonData(data.combinedData)
  setHasFiles(data.files.length > 0)
}

// Conditional rendering in accordions
{hasFiles ? (
  // Show mapper when files are available
  <JsonRequestMapper 
    data={jsonData}
    requestAndResponseVariableMappings={mappings}
    onProcessedData={(processedData) => {
      // Handle processed mapping data
    }}
  />
) : (
  // Show upload component when no files
  <SourceBodyUpload
    label="Request JSON Mapping"
    onJsonDataChange={handleJsonDataChange}
  />
)}
```

**Implementation Requirements:**
1. Add state management for JSON data and file presence
2. Implement conditional rendering logic in each accordion
3. Handle data flow between upload and mapper components
4. Pass existing mappings to mapper components
5. Handle processed data from mappers
6. Ensure proper cleanup when switching between upload and mapper

#### Step 5: Update Form Interfaces
1. Add `sourceDeliveryType` prop to the mentioned 3 form interfaces
2. Update form data structures to handle conditional accordion data
3. Add orchestration state management interfaces
4. Ensure proper TypeScript typing for conditional rendering

---

## Phase 11: Enhanced Mapper Integration with Upload Functionality

**Goal**: Integrate SourceBodyUpload functionality directly into mapper components and update forms to use the enhanced mappers, eliminating the need for complex form orchestration.

#### Step 1: Enhance Mapper Components
Update all three mapper components (JsonRequestMapper, JsonResponseMapper, JsonResultMapper) to include upload functionality:

**Interface Updates**:
- Make `data` prop optional (`data?: any`)
- Add `showUpload?: boolean` prop (default: true)
- Add `uploadLabel?: string` prop for customization
- Add `onDataChange?: (data: any) => void` callback

**Upload Integration**:
- When `data` is null/undefined AND `showUpload` is true: render SourceBodyUpload component
- When `data` exists: process through existing mapper logic
- Handle seamless transition from upload to mapping mode
- Process uploaded data through existing mapper pipeline

**State Management**:
- Add upload state: `jsonFiles`, `activeFileId`, `textInput`, `isRequesting`, `requestError`
- Handle data flow from upload to mapper processing internally
- Call `onDataChange` when data becomes available from upload

#### Step 2: Update RunRequestForm
Modify RunRequestForm to use enhanced mapper components:

**Remove Dead Code**:
- Remove `requestJsonData` and `responseJsonData` from form state
- Remove `handleRequestJsonDataChange` and `handleResponseJsonDataChange` handlers
- Remove conditional rendering logic for upload/mapping

**Update Mapper Integration**:
- Pass `data={formData.requestJsonData}` to JsonRequestMapper (can be null)
- Pass `data={formData.responseJsonData}` to JsonResponseMapper (can be null)
- Add `onDataChange` callbacks to update form state
- Add `showUpload={true}` and `uploadLabel` props

**Simplify Form Logic**:
- Remove placeholder text rendering
- Let mappers handle upload/mapping display internally
- Forms only need to handle data callbacks

#### Step 3: Update StatusCheckForm
Apply identical changes as RunRequestForm:
- Same dead code removal
- Same mapper integration updates
- Same form logic simplification

#### Step 4: Update DeliveryForm
Apply identical changes as RunRequestForm:
- Same dead code removal
- Same mapper integration updates
- Same form logic simplification
- Note: Uses JsonResultMapper for response mapping instead of JsonResponseMapper

#### Step 5: Update SourceRequestResponseAccordion
**No Changes Needed**:
- Accordion already passes mapper components as props
- Mappers handle upload functionality internally
- Accordion just renders the mapper components
- No upload-specific logic needed in accordion

#### Step 6: Keep SourceBodyUpload Component
**Keep Component**:
- Component remains as standalone component
- Used internally by enhanced mapper components
- Available for showcase pages and other use cases
- No changes needed to existing implementation

### 11.7 Expected Results After Integration

#### **11.7.1 Form Behavior**
- Forms show upload interface when no data is available
- Forms show mapping interface when data is available
- Seamless transition between upload and mapping modes
- No placeholder text or dead functionality
- Clean, simple form logic

#### **11.7.2 User Experience**
- Users can upload files directly in form workflows
- Consistent upload experience across all mapper types
- No jarring component switches
- Real functionality instead of placeholder text
- Unified interface for all data input methods

#### **11.7.3 Code Quality**
- Eliminated dead code and unused handlers
- Simplified form logic and state management
- Consistent patterns across all forms
- Better maintainability and readability
- Reduced complexity and potential bugs

### 11.8 Files to Modify

**Form Components**:
1. `components/forms/run-request-form.tsx`
2. `components/forms/status-check-form.tsx`
3. `components/forms/delivery-form.tsx`

**No Changes Needed**:
- `components/accordions/source-request-response-accordion.tsx`
- `components/json/interface/source-body-upload.tsx`

### 11.9 Validation Criteria

**Success Indicators**:
- All forms can upload files and process them through mappers
- No placeholder text appears in form workflows
- Seamless transition from upload to mapping in all forms
- No dead code or unused functionality remains
- Consistent upload experience across all mapper types
- Forms are simpler and easier to maintain

**Testing Approach**:
- Test file upload in all three form types
- Test text input in all form types
- Test API request functionality in all form types
- Verify no placeholder text appears
- Confirm seamless transition from upload to mapping
- Test form submission with uploaded data
- Verify data flows correctly through entire workflow

### 11.10 Critical Implementation Notes

**DO NOT**:
- Leave dead code or unused handlers in forms
- Use complex conditional rendering in forms
- Duplicate upload logic across forms
- Remove SourceBodyUpload component (keep for internal use)

**DO**:
- Remove all dead code and unused functionality
- Simplify form logic to just data callbacks
- Use consistent patterns across all forms
- Test upload functionality in actual form workflows
- Verify seamless integration with enhanced mappers

This phase completes the form integration by updating forms to work seamlessly with the enhanced mapper components that include upload functionality.

## Phase 12: Visual Component Updates and UX Enhancements

### Overview
Update visual styling and user experience across multiple components including tabs, drag-drop upload, source body upload, and select components with specific shadow effects, typography changes, and interaction improvements.

### Executive Summary

**Goal**: Enhance visual consistency and user experience across form components with specific shadow effects, typography improvements, and interaction states.

**Solution**: Update four key components with precise visual specifications including shadow systems, typography scaling, content restructuring, and interaction improvements.

**Key Features**:
1. **Tabs Component**: XL drop shadow with inner shadows for active tabs, typography scaling, and hover states
2. **Drag-Drop Upload**: Background styling, thinner borders, secondary button integration, and content restructuring
3. **Source Body Upload**: Tab content alignment, conditional visibility, fixed height management, and transition timing
4. **Select Component**: Pointer cursor for dropdown interactions

**User Experience**:
Consistent visual hierarchy with proper shadow depth, improved typography scaling, better content organization, and enhanced interaction feedback across all form components.

**Technical Approach**:
- Apply existing shadow utility classes with specific sizing and positioning
- Implement typography scaling using Tailwind text size classes
- Restructure content layout with proper spacing and alignment
- Add conditional visibility props for flexible component usage
- Enhance interaction states with appropriate cursor and hover effects

### 12.1 Tabs Component Visual Updates

**File**: `components/ui/custom-ui-elements/buttons/tabs.tsx` (MODIFY)

**Current State Analysis** (lines 16-32):
- Container: `flex space-x-1 rounded-lg bg-muted p-1` (line 18)
- Button styling: `variant={tab.isActive ? "default" : "ghost"}` with `size="sm"` (lines 22-23)
- Active button: `bg-interactive text-foreground` (line 25)
- Transition: `transition-all` (line 25)

**Required Changes**:

**Container Shadow Enhancement** (modify line 18):
- Current: `"flex space-x-1 rounded-lg bg-muted p-1"`
- New: `"flex space-x-1 rounded-lg bg-interactive p-1 shadow-default"`
- Change background to `bg-interactive` and add outer XL drop shadow using existing `shadow-default` utility class

**Active Tab Inner Shadow and Border** (modify line 25):
- Current: `"flex-1 transition-all", tab.isActive && "bg-background text-foreground"`
- New: `"flex-1 transition-all", tab.isActive && "bg-interactive text-foreground shadow-inset-default border"`
- Add inner shadow using existing `shadow-inset-default` utility class and border for active tabs
- Inactive tabs: No border (default state)

**Typography Scaling** (modify line 25):
- Current: No specific text size classes
- New: Add text size classes for active/inactive states
- Active tabs: `text-base` (larger font)
- Inactive tabs: `text-sm` (smaller font)
- Implementation: `tab.isActive ? "text-base" : "text-sm"`

**Hover State Enhancement** (modify line 25):
- Current: No hover-specific styling
- New: Add hover state with text color change and blue inner shadow
- Hover text: Use existing text button hover pattern
- Hover shadow: `hover:shadow-inset-hover-primary`
- Implementation: Add `hover:text-primary hover:shadow-inset-hover-primary` classes

**Complete Updated Button Styling** (line 25):
```typescript
className={cn(
  "flex-1 transition-all",
  tab.isActive 
    ? "bg-interactive text-foreground text-base shadow-inset-default border" 
    : "text-sm",
  "hover:text-primary hover:shadow-inset-hover-primary"
)}
```

### 12.2 Drag-Drop Upload Component Visual Updates

**File**: `components/forms/blocks/drag-drop-upload.tsx` (MODIFY)

**Current State Analysis** (lines 154-210):
- Container: `border-2 border-dashed rounded-lg p-6 text-center transition-all duration-200` (line 156)
- Hover states: `hover:border-primary/50 hover:bg-muted/50 hover:shadow-hover-primary` (line 157)
- Content structure: Upload icon, placeholder text, format info, size info, files info (lines 180-210)

**Required Changes**:

**Background Styling** (modify line 156):
- Current: No background class
- New: Add `bg-interactive` class for consistent background
- Updated: `"border border-dashed rounded-lg p-6 text-center transition-all duration-200 bg-interactive"`

**Border Thickness and Color** (modify line 156):
- Current: `border-2 border-dashed`
- New: `border border-dashed` (thinner border)
- Color: Use standard border color (existing `border` class)

**Default Shadow** (modify line 156):
- Current: No default shadow
- New: Add `shadow-default` for consistent drop shadow
- Updated: `"border border-dashed rounded-lg p-6 text-center transition-all duration-200 bg-interactive shadow-default"`

**Hover State Update** (modify line 157):
- Current: `hover:border-primary/50 hover:bg-muted/50 hover:shadow-hover-primary`
- New: Keep blue shadow, remove background change
- Updated: `hover:shadow-hover-primary`

**Content Restructuring** (modify lines 180-210):
- Current: Single paragraph with placeholder text
- New: Multi-line content structure with secondary button
- Line 1: "Drop .json files here or" (text)
- Line 2: "Browse files" (SecondaryButton component)
- Line 3: "5 files, 10MBs each maximum" (constraint info)

**Remove Description Section** (modify lines 221-223):
- Current: Conditional description rendering
- New: Remove entire description section
- Delete lines 221-223: `{description && (<p className="text-xs text-muted-foreground">{description}</p>)}`

**Add Secondary Button** (modify lines 180-210):
- Import SecondaryButton component at top of file
- Add button after upload icon and before constraint text
- Button text: "Browse files"
- Button styling: Use existing SecondaryButton component
- Button functionality: NONE - purely visual indicator, whole drag-drop area is interactive
- Button will change hover state with the whole drag-drop area (blue dropshadow)

**Add Pointer Cursor** (modify line 156):
- Current: No cursor specification
- New: Add `cursor-pointer` class
- Updated: `"border border-dashed rounded-lg p-6 text-center transition-all duration-200 bg-interactive shadow-default cursor-pointer"`

**Updated Content Structure** (lines 180-210):
```typescript
<div className="space-y-2">
  <Upload className={cn(
    "h-8 w-8 mx-auto",
    isDragOver ? "text-primary" : "text-muted-foreground"
  )} />
  
  <div className="space-y-1">
    <p className={cn(
      "text-sm",
      isDragOver ? "text-primary font-medium" : "text-muted-foreground"
    )}>
      Drop .json files here or
    </p>
    
    <SecondaryButton
      onClick={handleBrowseClick}
      size="sm"
      className="mx-auto"
    >
      Browse files
    </SecondaryButton>
    
    <p className="text-xs text-muted-foreground">
      {maxFiles} files, {maxSize}MBs each maximum
    </p>
  </div>
</div>
```

### 12.3 Source Body Upload Component Updates

**File**: `components/json/interface/source-body-upload.tsx` (MODIFY)

**Current State Analysis** (lines 268-358):
- Tabs structure: Three tabs (request, upload, text) with TabsList and TabsContent (lines 268-273)
- Request tab: Centered content with Card wrapper (lines 276-303)
- Upload tab: FormDragDrop component (lines 306-317)
- Text tab: Textarea with validation and save button (lines 320-357)

**Required Changes**:

**Add Conditional Visibility Props** (modify interface lines 23-37):
- Current: No visibility props
- New: Add props to control tab visibility
- Add to SourceBodyUploadProps interface:
```typescript
showRequestTab?: boolean
showUploadTab?: boolean  
showTextTab?: boolean
```
- Default values: `showRequestTab: false`, `showUploadTab: true`, `showTextTab: true`

**Request Tab Content Alignment** (modify lines 276-303):
- Current: `text-center` class on Card content (line 279)
- New: Change to `text-left` for left alignment
- Updated: `<div className="space-y-4">` (remove text-center)

**Fixed Height Management** (modify line 268):
- Current: `fixedHeight = "min-h-[400px]"` (line 45)
- New: Calculate proper fixed height considering all elements and states
- Keep fixed height on Tabs container to prevent layout shifts
- Calculate height to accommodate: tabs header + content area + "Uploaded Files" section
- Account for different states: empty state, with files, validation states
- Ensure height remains consistent regardless of file uploads or content changes

**Tab Visibility Implementation** (modify lines 268-273):
- Current: Hardcoded three TabsTrigger elements
- New: Conditional rendering based on props with tab container hiding
- Count visible tabs: `const visibleTabsCount = [showRequestTab, showUploadTab, showTextTab].filter(Boolean).length`
- Hide TabsList when only one tab is visible: `{visibleTabsCount > 1 && (<TabsList>...)}`
- Implementation:
```typescript
const visibleTabsCount = [showRequestTab, showUploadTab, showTextTab].filter(Boolean).length

<Tabs defaultValue="upload" className={cn("w-full", fixedHeight)}>
  {visibleTabsCount > 1 && (
    <TabsList className="grid w-full grid-cols-3">
      {showRequestTab && <TabsTrigger value="request">Get by Request</TabsTrigger>}
      {showUploadTab && <TabsTrigger value="upload">Upload File</TabsTrigger>}
      {showTextTab && <TabsTrigger value="text">Paste as Text</TabsTrigger>}
    </TabsList>
  )}
```

**Text Tab Transition Timing** (modify lines 331-355):
- Current: Validation feedback shows immediately on text input
- New: Only show transition after "Save as JSON file" button click
- Remove immediate validation display
- Keep validation logic but only show UI feedback after save action

**Height Constraint Implementation**:
- Keep `fixedHeight` prop on Tabs container (line 268) for layout stability
- Calculate appropriate height value considering:
  - TabsList height (~40px)
  - Content area height (varies by tab type)
  - "Uploaded Files" section height when visible (~80px)
  - Padding and spacing between elements
- Apply `min-h-[200px]` to textarea (line 328) - already exists
- Ensure consistent height regardless of content state changes

### 12.4 Select Component Cursor Update

**File**: `components/ui/custom-ui-elements/inputs/select.tsx` (MODIFY)

**Current State Analysis** (lines 66-78 and 142-153):
- SelectTrigger styling: Multiple className applications with shadow and transition classes
- Current cursor: No specific cursor class applied

**Required Changes**:

**Add Pointer Cursor and Background to SelectTrigger** (modify lines 67-72 and 143-148):
- Current: No cursor specification or background class
- New: Add `cursor-pointer` class and `bg-interactive` background to both FormSelect and FormFieldSelect components
- Updated className for FormSelect (lines 67-72):
```typescript
className={cn(
  getShadowClass(),
  'transition-shadow duration-200',
  disabled && 'opacity-50 cursor-not-allowed',
  'data-[placeholder]:text-muted-foreground',
  'bg-interactive cursor-pointer' // Add these lines
)}
```
- Updated className for FormFieldSelect (lines 143-148):
```typescript
className={cn(
  getShadowClass(),
  'transition-shadow duration-200',
  disabled && 'opacity-50 cursor-not-allowed',
  'data-[placeholder]:text-muted-foreground',
  'bg-interactive cursor-pointer' // Add these lines
)}
```

### 12.5 Implementation Steps

**Step 1: Update Tabs Component**
1. Add `shadow-default` to container div (line 18)
2. Add inner shadow and typography scaling to active tabs (line 25)
3. Add hover states with text color and inner shadow (line 25)
4. Test visual hierarchy and interaction states

**Step 2: Update Drag-Drop Upload Component**
1. Add `bg-interactive` and `shadow-default` to container (line 156)
2. Change border from `border-2` to `border` (line 156)
3. Update hover states to remove background change (line 157)
4. Add `cursor-pointer` class (line 156)
5. Restructure content with three-line layout (lines 180-210)
6. Add SecondaryButton component and import
7. Remove description section (lines 221-223)
8. Test new content structure and interactions

**Step 3: Update Source Body Upload Component**
1. Add conditional visibility props to interface (lines 23-37)
2. Change request tab alignment from center to left (line 279)
3. Implement conditional tab rendering (lines 269-273)
4. Remove fixed height from Tabs container (line 268)
5. Update text tab transition timing (lines 331-355)
6. Test conditional visibility and alignment

**Step 4: Update Select Component**
1. Add `cursor-pointer` to FormSelect SelectTrigger (lines 67-72)
2. Add `cursor-pointer` to FormFieldSelect SelectTrigger (lines 143-148)
3. Test cursor behavior on both components

**Step 5: Testing and Validation**
1. Test all shadow effects and visual hierarchy
2. Test typography scaling on tabs
3. Test hover states and interactions
4. Test conditional visibility in source body upload
5. Test content restructuring in drag-drop upload
6. Test cursor behavior on select components

### 12.6 Success Criteria

**Visual Consistency**:
- All components use consistent shadow system
- Typography scaling creates clear visual hierarchy
- Hover states provide appropriate feedback

**User Experience**:
- Clear visual distinction between active/inactive states
- Intuitive interaction patterns
- Proper content organization and alignment

**Functionality**:
- Conditional visibility works correctly
- All interactions respond appropriately
- Content restructuring maintains usability

**Code Quality**:
- Clean implementation using existing utility classes
- Proper prop interfaces for flexibility
- Maintainable and consistent patterns

This phase enhances the visual consistency and user experience across form components with specific shadow effects, typography improvements, and interaction enhancements.

## Phase 13: Input and Selector Component Standardization

### Overview
Standardize all input and selector components with consistent shadow systems, status indication patterns, border styling, and background colors to create a unified visual language across all form components.

### Executive Summary

**Goal**: Create visual consistency across inputs and selectors through standardized shadows, status icons, borders, backgrounds, and text colors.

**Solution**: Implement dual status system with colored validation shadows plus semantic status icons positioned on the right side of components.

**Key Features**:
1. **Universal Shadow System**: All components use `shadow-default` with colored validation shadows
2. **Status Icons on Right**: Validation and semantic icons positioned right side (right-bottom for textarea)
3. **Consistent Borders**: All use `border-muted` by default, `border-primary` on focus
4. **Unified Backgrounds**: All use `bg-interactive` for triggers and dropdowns
5. **Text Color States**: `text-foreground` when filled, `text-muted-foreground` for placeholders

**User Experience**:
Consistent visual feedback across all form components, clear status indication through both shadows and icons, unified interaction patterns for inputs and selectors.

**Technical Approach**:
- Apply shadow-default to all base UI components
- Keep validation shadow system (default/success/error/warning) in custom inputs
- Add status icon support with right-side positioning
- Standardize border and background colors
- Update selectors to match input visual patterns

### 13.1 Universal Pattern Across ALL Components

#### Status Indication System:
1. **Colored Shadows** - Validation states (default, success, error, warning)
2. **Status Icons** - Semantic meaning with custom colors per icon positioned on RIGHT

#### Visual Standards:
- **Default Shadow**: `shadow-default` on all components
- **Hover Shadow**: `hover:shadow-hover-primary` on all components
- **Border**: `border-muted` default, `border-primary` on focus
- **Background**: `bg-interactive` for all triggers and dropdowns
- **Text**: `text-foreground` when filled, `text-muted-foreground` for placeholder
- **Transition**: `transition-shadow duration-200` for smooth animations

### 13.2 Base UI Components Enhancement

#### 13.2.1 Input Component
**File**: `components/ui/input.tsx` (line 11)

**Current State**: `border border-muted bg-interactive`

**Changes Required**:
- ➕ Add `shadow-default`
- ➕ Add `transition-shadow duration-200`
- ✅ Keep: `bg-interactive`, `border-muted`, `text-foreground`, `placeholder:text-muted-foreground`

**Updated className**:
```
"flex h-10 w-full rounded-md border border-muted bg-interactive px-3 py-2 text-base shadow-default transition-shadow duration-200 file:border-0 file:bg-transparent file:text-sm file:font-medium file:text-foreground placeholder:text-muted-foreground focus-visible:outline-none focus-visible:border-primary disabled:cursor-not-allowed disabled:opacity-50 md:text-sm hover:shadow-hover-primary"
```

#### 13.2.2 Textarea Component
**File**: `components/ui/textarea.tsx` (line 12)

**Current State**: `border border-muted bg-interactive`

**Changes Required**:
- ➕ Add `shadow-default`
- ➕ Add `transition-shadow duration-200`
- ✅ Keep: `bg-interactive`, `border-muted`

**Updated className**:
```
"flex min-h-[80px] w-full rounded-md border border-muted bg-interactive px-3 py-2 text-base shadow-default transition-shadow duration-200 placeholder:text-muted-foreground focus-visible:outline-none focus-visible:border-primary disabled:cursor-not-allowed disabled:opacity-50 md:text-sm hover:shadow-hover-primary"
```

#### 13.2.3 Select Component
**File**: `components/ui/select.tsx`

**SelectTrigger Changes** (line 22):
- ➕ Add `shadow-default`
- ➕ Add `transition-shadow duration-200`
- ➕ Change `border-input` → `border-muted`
- ✅ Keep: `bg-interactive`

**Updated className**:
```
"flex h-10 w-full items-center justify-between rounded-md border border-muted bg-interactive px-3 py-2 text-sm shadow-default transition-shadow duration-200 placeholder:text-muted-foreground focus:outline-none focus:border-primary disabled:cursor-not-allowed disabled:opacity-50 hover:shadow-hover-primary [&>span]:line-clamp-1"
```

**SelectContent Changes** (line 78):
- ➕ Change `bg-popover text-popover-foreground` → `bg-interactive text-interactive-foreground`

### 13.3 Custom Input Components Enhancement

All custom inputs maintain their validation shadow system and add status icon support.

#### 13.3.1 Short Text Input (FormInput)
**File**: `components/ui/custom-ui-elements/inputs/short-text-input.tsx`

**Keep**:
- ✅ `state` prop (line 37)
- ✅ `getShadowClass()` function (lines 72-83, 202-213)
- ✅ Validation shadows (success/error/warning)

**Add**:
- ➕ `statusIcon?: React.ReactNode` prop to interfaces (lines 29, 176)
- ➕ Icon positioning: RIGHT side inside input
- ➕ Icon wrapper: `absolute right-2 top-1/2 -translate-y-1/2`
- ➕ Padding adjustment: `pr-10` when icon present

**Implementation Pattern**:
```typescript
interface FormInputProps {
  // ... existing props
  statusIcon?: React.ReactNode
}

// In render:
<div className="relative">
  <Input
    className={cn(
      getShadowClass(),
      'transition-shadow duration-200',
      statusIcon && 'pr-10',
      className
    )}
    {...props}
  />
  {statusIcon && (
    <div className="absolute right-2 top-1/2 -translate-y-1/2">
      {statusIcon}
    </div>
  )}
</div>
```

#### 13.3.2 Long Text Input (FormTextarea)
**File**: `components/ui/custom-ui-elements/inputs/long-text-input.tsx`

**Keep**:
- ✅ `state` prop (line 39)
- ✅ `getShadowClass()` function (lines 79-90, 215-226)
- ✅ Validation shadows

**Add**:
- ➕ `statusIcon?: React.ReactNode` prop to interfaces (lines 28, 185)
- ➕ Icon positioning: RIGHT BOTTOM corner
- ➕ Icon wrapper: `absolute right-2 bottom-2`
- ⚠️ Note: "Add variable" button at `right-2 top-2`, icon at `right-2 bottom-2`

#### 13.3.3 Number Input
**File**: `components/ui/custom-ui-elements/inputs/number-input.tsx`

**Keep**:
- ✅ `state` prop (line 25)
- ✅ `getShadowClass()` functions (lines 72-98)
- ✅ Validation shadows

**Add**:
- ➕ `statusIcon?: React.ReactNode` prop (line 11)
- ➕ Icon positioning: RIGHT side of component
- ➕ Replace: `shadow-default hover:shadow-hover-primary` on input and buttons

#### 13.3.4 Radio Group
**File**: `components/ui/custom-ui-elements/inputs/radio-group.tsx`

**Keep**:
- ✅ `state` prop (line 23)
- ✅ `getShadowClass()` function (lines 45-57)
- ✅ Validation shadows

**Add**:
- ➕ `statusIcon?: React.ReactNode` prop (line 14)
- ➕ Icon position: Next to label in FormFieldWrapper
- ➕ Pass icon to FormFieldWrapper component

#### 13.3.5 Select Input (FormSelect)
**File**: `components/ui/custom-ui-elements/inputs/select.tsx`

**Keep**:
- ✅ `state` prop (lines 24, 105)
- ✅ `getShadowClass()` function (lines 43-55, 121-133)
- ✅ Validation shadows

**Add**:
- ➕ `statusIcon?: React.ReactNode` prop to interfaces (lines 13, 96)
- ➕ Icon positioning: RIGHT side inside SelectTrigger
- ➕ Icon wrapper: `absolute right-8 top-1/2 -translate-y-1/2` (avoids chevron)
- ➕ Padding: `pr-16` on SelectTrigger when icon present

**SelectContent Changes**:
- ➕ Change to: `bg-interactive text-interactive-foreground`

### 13.4 FormFieldWrapper Enhancement

**File**: `components/forms/blocks/form-validation.tsx`

**Add**:
- ➕ `statusIcon?: React.ReactNode` prop
- ➕ Display icon next to label

**Implementation**:
```typescript
{label && (
  <label className="...">
    {label}
    {required && <span>*</span>}
    {statusIcon && <span className="ml-2 inline-flex">{statusIcon}</span>}
  </label>
)}
```

### 13.5 Selector Components Standardization

#### 13.5.1 BaseSelector Component
**File**: `components/selectors/base-selector.tsx`

**Remove**:
- ❌ `displayColorClass` prop
- ❌ Semantic text colors in trigger (line 246)

**Keep**:
- ✅ Selected items with `bg-success/10` and `text-success`
- ✅ Recommended items with `bg-recommendation/10` and recommendation color
- ✅ Icons in dropdown items (MapPin, Lightbulb, etc.)

**Add**:
- ➕ `state?: 'default' | 'success' | 'error' | 'warning'` prop
- ➕ `getShadowClass()` function (same pattern as inputs)
- ➕ `statusIcon?: React.ReactNode` prop
- ➕ Pass shadow class and icon to SelectButton
- ➕ Text: `text-foreground` when selected, `text-muted-foreground` for placeholder

**CommandContent**:
- ✅ Ensure `bg-interactive`

#### 13.5.2 SelectButton Component
**File**: `components/ui/custom-ui-elements/buttons/select-button.tsx`

**Remove**:
- ❌ `borderColorClass` prop (line 19)
- ❌ `chevronColorClass` prop (line 20)
- ❌ Semantic border logic (line 47)
- ❌ Chevron color logic (line 58)

**Add**:
- ➕ `statusIcon?: React.ReactNode` prop
- ➕ `shadowClass?: string` prop
- ➕ Icon positioning: RIGHT side, before chevron
- ➕ Apply shadow class to SecondaryButton
- ✅ Keep: `border-primary` when `isOpen`

**Layout Structure**:
```typescript
<SecondaryButton className={shadowClass}>
  <span className="truncate">{children}</span>
  {statusIcon && <span className="ml-2 flex-shrink-0">{statusIcon}</span>}
  <ChevronsUpDown className="ml-2 h-4 w-4 ..." />
</SecondaryButton>
```

#### 13.5.3 PathsExpandSelector Component
**File**: `components/selectors/paths-expand-selector.tsx`

**Remove**:
- ❌ `getDisplayColorClass()` function (lines 34-49)
- ❌ `displayColorClass` prop to BaseSelector (line 65)

**Add**:
- ➕ `getStatusIcon()` function:
```typescript
const getStatusIcon = (value?: string) => {
  switch (value) {
    case 'show-mapped':
      return <MapPin className="h-4 w-4 text-success" />
    case 'show-issues':
      return <AlertTriangle className="h-4 w-4 text-warning" />
    case 'show-recommended':
      return <Lightbulb className="h-4 w-4 text-recommendation" />
    case 'show-structure':
      return <Layers className="h-4 w-4 text-primary" />
    case 'show-all':
      return <Grid className="h-4 w-4 text-muted-foreground" />
    default:
      return null
  }
}
```
- ➕ Pass `statusIcon={getStatusIcon(value)}` to BaseSelector
- ➕ Optionally pass `state` prop for validation shadows

#### 13.5.4 JsonSelector Component
**File**: `components/selectors/json-selector.tsx`

**Changes**:
- ✅ No status icons needed (files don't have semantic status)
- ➕ Add shadow-default via base-selector
- ➕ Ensure dropdowns use `bg-interactive`

#### 13.5.5 Other Selectors
**Files**: source-selector.tsx, datapoint-selector.tsx, workflow-phase-selector.tsx, etc.

**Changes**:
- ➕ Add status icons where semantic meaning exists
- ➕ Each can define custom icon set with custom colors
- Most won't need status icons (pure data selection)

### 13.6 Status Icon Patterns

#### 13.6.1 Validation Icons (for Inputs)
```typescript
import { CheckCircle, XCircle, AlertCircle } from "lucide-react"

const validationIcons = {
  default: null,
  success: <CheckCircle className="h-4 w-4 text-success" />,
  error: <XCircle className="h-4 w-4 text-destructive" />,
  warning: <AlertCircle className="h-4 w-4 text-warning" />
}
```

#### 13.6.2 Semantic Icons (for Selectors)
```typescript
import { MapPin, Lightbulb, AlertTriangle, Layers, Grid } from "lucide-react"

// Example for paths-expand-selector
const statusIcons = {
  'show-mapped': <MapPin className="h-4 w-4 text-success" />,
  'show-issues': <AlertTriangle className="h-4 w-4 text-warning" />,
  'show-recommended': <Lightbulb className="h-4 w-4 text-recommendation" />,
  'show-structure': <Layers className="h-4 w-4 text-primary" />,
  'show-all': <Grid className="h-4 w-4 text-muted-foreground" />
}
```

### 13.7 Icon Positioning Summary

| Component | Icon Position | Wrapper Classes | Padding Adjustment |
|-----------|---------------|-----------------|-------------------|
| **FormInput** | Right inside | `absolute right-2 top-1/2 -translate-y-1/2` | `pr-10` |
| **FormTextarea** | Right bottom corner | `absolute right-2 bottom-2` | Optional padding |
| **NumberInput** | Right side | After number controls | As needed |
| **RadioGroup** | Next to label | `ml-2 inline-flex` in FormFieldWrapper | None |
| **FormSelect** | Right inside | `absolute right-8 top-1/2 -translate-y-1/2` | `pr-16` |
| **BaseSelector** | Before chevron | `ml-2 flex-shrink-0` between text/chevron | None |

### 13.8 Complete Status System Matrix

| Component | Colored Shadows | Status Icons | Icon Location | Icon Purpose |
|-----------|----------------|--------------|---------------|--------------|
| **Input** | ✅ Validation | ✅ Optional | Right inside | Validation state |
| **Textarea** | ✅ Validation | ✅ Optional | Right-bottom | Validation state |
| **Number** | ✅ Validation | ✅ Optional | Right side | Validation state |
| **Radio** | ✅ Validation | ✅ Optional | Next to label | Validation state |
| **Select (input)** | ✅ Validation | ✅ Optional | Right inside | Validation state |
| **Base Selector** | ✅ Validation | ✅ Optional | Before chevron | Semantic meaning |
| **Paths Selector** | ✅ Validation | ✅ Required | Before chevron | Show/filter type |
| **JSON Selector** | ✅ Validation | ❌ None | - | File selection |
| **Other Selectors** | ✅ Validation | ⚠️ Contextual | Before chevron | Data-specific |

### 13.9 Implementation Steps

#### Step 1: Update Base UI Components (3 files)
1. Add `shadow-default transition-shadow duration-200` to input.tsx
2. Add `shadow-default transition-shadow duration-200` to textarea.tsx
3. Add `shadow-default transition-shadow duration-200` to select.tsx
4. Change SelectContent to `bg-interactive text-interactive-foreground`
5. Test all base components

#### Step 2: Enhance Custom Input Components (5 files)
1. Add `statusIcon` prop to all input interfaces
2. Implement icon positioning for each component type
3. Add padding adjustments where needed
4. Keep existing validation shadows and state props
5. Test each input type with and without icons

#### Step 3: Update FormFieldWrapper (1 file)
1. Add `statusIcon` prop to interface
2. Implement icon display next to label
3. Test with radio group component

#### Step 4: Standardize Selector Components (7+ files)
1. Update BaseSelector with shadow system and icon support
2. Update SelectButton to accept shadow and icon props
3. Remove semantic color props from all selectors
4. Add status icon functions to paths-expand-selector
5. Ensure all selectors use bg-interactive for dropdowns
6. Test selector visual consistency

#### Step 5: Testing and Validation
1. Test shadow system across all components
2. Test icon positioning in all locations
3. Test validation states with colored shadows
4. Test semantic icons in selectors
5. Test hover states and transitions
6. Test disabled states
7. Verify visual consistency across component types

### 13.10 New Imports Required

**Lucide React Icons**:
```typescript
import { 
  CheckCircle, XCircle, AlertCircle,  // Validation icons
  MapPin, Lightbulb, AlertTriangle,    // Semantic icons
  Layers, Grid                          // Additional UI icons
} from "lucide-react"
```

### 13.11 Files to Modify

**Base UI Components (3 files)**:
1. `components/ui/input.tsx`
2. `components/ui/textarea.tsx`
3. `components/ui/select.tsx`

**Custom Input Components (5 files)**:
4. `components/ui/custom-ui-elements/inputs/short-text-input.tsx`
5. `components/ui/custom-ui-elements/inputs/long-text-input.tsx`
6. `components/ui/custom-ui-elements/inputs/number-input.tsx`
7. `components/ui/custom-ui-elements/inputs/radio-group.tsx`
8. `components/ui/custom-ui-elements/inputs/select.tsx`

**Form Components (1 file)**:
9. `components/forms/blocks/form-validation.tsx`

**Selector Components (7+ files)**:
10. `components/selectors/base-selector.tsx`
11. `components/ui/custom-ui-elements/buttons/select-button.tsx`
12. `components/selectors/paths-expand-selector.tsx`
13. `components/selectors/json-selector.tsx`
14. `components/selectors/source-selector.tsx`
15. `components/selectors/datapoint-selector.tsx`
16. `components/selectors/workflow-phase-selector.tsx`
17. Other selector files as needed

**Total Files**: ~16 files

### 13.12 Success Criteria

**Visual Consistency**:
- All inputs and selectors use consistent shadow system
- Status indication through both shadows and icons
- Unified border and background colors
- Consistent text color states

**Functional Requirements**:
- Validation shadows work correctly
- Status icons display appropriately
- Icons positioned correctly on right side
- Hover states provide proper feedback
- Disabled states work correctly

**Code Quality**:
- Clean, maintainable code patterns
- Consistent prop interfaces
- Proper TypeScript typing
- Reusable icon patterns
- Well-documented components

**User Experience**:
- Clear visual feedback for all states
- Intuitive status indication
- Smooth transitions and animations
- Consistent interaction patterns
- Accessible for all users

This phase completes the standardization of inputs and selectors, creating a unified visual language with consistent shadow systems, status indication patterns, and interaction behaviors across all form components.

## Phase 14: Complete Bidirectional File Management Integration

### Overview
Integrate SourceBodyUpload's internal file management with ManageJsonsDialog and JsonSelector to create a unified file management system where files flow bidirectionally between the initial upload interface and the management dialog, all orchestrated by mapper components.

### Executive Summary

**Goal**: Connect SourceBodyUpload's internal file management with ManageJsonsDialog so all uploaded files (whether uploaded through SourceBodyUpload's initial upload interface or through ManageJsonsDialog later) appear in a unified file list accessible from JsonSelector dropdown within JsonExplorer.

**Solution**: Lift file state from SourceBodyUpload to mapper level, making SourceBodyUpload a controlled component that accepts files as props and uses callbacks for modifications, while mappers maintain central file state authority.

**Key Features**:
1. **State Elevation**: Files and activeFileId managed at mapper level, not in SourceBodyUpload
2. **Controlled SourceBodyUpload**: Accepts files/activeFileId props, uses parent callbacks for changes
3. **Mapper Orchestration**: Mappers own file state and coordinate all file operations
4. **Unified File List**: All files visible in JsonSelector regardless of upload source
5. **Bidirectional Flow**: Files added via SourceBodyUpload or ManageJsonsDialog appear in unified list
6. **Automatic Rendering**: Automatic transition from upload to explorer when JSON validated

**Current Behavior**:
- SourceBodyUpload maintains internal jsonFiles state
- Files uploaded through SourceBodyUpload automatically trigger mapper rendering
- No "Continue" button - seamless automatic transition to JsonExplorer
- Files remain trapped inside SourceBodyUpload, not accessible to JsonExplorer/ManageJsonsDialog

**Target Behavior**:
- Mappers maintain jsonFiles and activeFileId state
- SourceBodyUpload becomes controlled component receiving files as props
- Automatic rendering preserved - still transitions seamlessly to JsonExplorer
- Files flow to JsonExplorer and appear in JsonSelector dropdown
- ManageJsonsDialog shows all uploaded files
- Adding files via dialog adds to mapper's files state
- All components stay synchronized through mapper state

**User Experience**:
User uploads file → File added to mapper state → Automatic transition to JsonExplorer → JsonSelector shows file in dropdown → User clicks dropdown → ManageJsonsDialog displays file → User uploads another file via dialog → Both files appear everywhere → User can switch between files → All components synchronized

**Technical Approach**:
- Standardize JsonFile interface across all components
- Convert SourceBodyUpload to controlled component pattern
- Add file management state and callbacks to all three mappers
- Update forms to pass file props from mappers to JsonExplorer
- Maintain automatic rendering behavior while adding file management

### 14.1 Unified JsonFile Interface

**Current State**: Three different JsonFile interfaces exist across components with inconsistent property names.

**SourceBodyUpload's JsonFile** (line 13):
```typescript
interface JsonFile {
  id: string
  name: string
  content: any
  lastModified: Date
  propertiesCount: number
}
```

**ManageJsonsDialog's JsonFile** (line 13):
```typescript
interface JsonFile {
  id: string
  name: string
  size: number
  lastModified: Date
  propertiesCount: number
  data: any
}
```

**JsonExplorer's JsonFile** (line 33):
```typescript
interface JsonFile {
  id: string
  name: string
  file: File
  data: any
  size: number
  lastModified: Date
}
```

**Standardized Interface**:
```typescript
interface JsonFile {
  id: string                    // Unique identifier
  name: string                  // Filename
  content: any                  // Parsed JSON data (standardize on 'content')
  size: number                  // File size in bytes
  lastModified: Date            // Last modified timestamp
  propertiesCount: number       // Count of root properties
}
```

**Required Updates**:
- **ManageJsonsDialog**: Change `data` property to `content` (line 19)
- **JsonSelector**: Change `data` property to `content` (line 13)
- **JsonExplorer**: Change `data` property to `content` OR map in forms when passing
- **SourceBodyUpload**: Already uses `content` ✓

### 14.2 SourceBodyUpload Component Refactoring

**File**: `components/json/interface/source-body-upload.tsx` (REFACTOR)

**Current State** (lines 52-57):
- Internal state: `jsonFiles` and `activeFileId` managed internally
- Automatic notification via `onJsonDataChange` callback
- No external control over file state

**Target State**: Controlled component accepting files from parent

**Props Interface Update**:
```typescript
interface SourceBodyUploadProps {
  // NEW: Controlled state props
  files?: JsonFile[]
  activeFileId?: string | null
  onFilesChange?: (files: JsonFile[]) => void
  onActiveFileIdChange?: (id: string | null) => void
  
  // Existing props
  label?: string
  description?: string
  maxFiles?: number
  maxSize?: number
  className?: string
  fixedHeight?: string
  showRequestTab?: boolean
  showUploadTab?: boolean
  showTextTab?: boolean
  onJsonDataChange?: (data: {
    files: JsonFile[]
    activeFile: JsonFile | null
    textData: any | null
    combinedData: any | null
  }) => void
}
```

**State Conversion** (lines 52-57):
```typescript
// BEFORE (internal state):
const [jsonFiles, setJsonFiles] = useState<JsonFile[]>([])
const [activeFileId, setActiveFileId] = useState<string | null>(null)

// AFTER (use props with fallback):
const jsonFiles = files || []
const activeFileId = activeFileId || null

// Keep internal state for text input and request
const [textInput, setTextInput] = useState<string>("")
const [isRequesting, setIsRequesting] = useState<boolean>(false)
const [requestError, setRequestError] = useState<string>("")
const [activeTab, setActiveTab] = useState<string>("upload")
```

**handleFileUpload Update** (lines 89-126):
```typescript
const handleFileUpload = useCallback((uploadedFiles: File[]) => {
  const newFiles: JsonFile[] = []
  
  uploadedFiles.forEach((file) => {
    const reader = new FileReader()
    
    reader.onload = (e) => {
      try {
        const content = JSON.parse(e.target?.result as string)
        const jsonFile: JsonFile = {
          id: `file-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
          name: file.name,
          content,
          size: file.size,  // ADD size property
          lastModified: new Date(file.lastModified),
          propertiesCount: Object.keys(content).length
        }
        
        newFiles.push(jsonFile)
        
        if (newFiles.length === uploadedFiles.length) {
          // Call parent callback instead of setState
          const updated = [...jsonFiles, ...newFiles]
          onFilesChange?.(updated)
          
          // Auto-select first file if none selected
          if (!activeFileId && updated.length > 0) {
            onActiveFileIdChange?.(updated[0].id)
          }
        }
      } catch (error) {
        console.error("Failed to parse JSON file:", file.name, error)
      }
    }
    
    reader.readAsText(file)
  })
}, [jsonFiles, activeFileId, onFilesChange, onActiveFileIdChange])
```

**handleGetByRequest Update** (lines 130-171):
```typescript
const handleGetByRequest = useCallback(async () => {
  setIsRequesting(true)
  setRequestError("")
  
  try {
    await new Promise(resolve => setTimeout(resolve, 2000))
    
    const mockResponse = { /* ... */ }
    
    const newFile: JsonFile = {
      id: `request-${Date.now()}`,
      name: "API Response",
      content: mockResponse,
      size: JSON.stringify(mockResponse).length,  // Calculate size
      lastModified: new Date(),
      propertiesCount: Object.keys(mockResponse).length
    }
    
    // Call parent callback
    const updated = [...jsonFiles, newFile]
    onFilesChange?.(updated)
    onActiveFileIdChange?.(newFile.id)
    
  } catch (error) {
    setRequestError("Failed to fetch data from request")
  } finally {
    setIsRequesting(false)
  }
}, [jsonFiles, onFilesChange, onActiveFileIdChange])
```

**handleSaveAsJsonFile Update** (lines 185-214):
```typescript
const handleSaveAsJsonFile = useCallback(() => {
  if (!hasValidTextData) return
  
  try {
    const jsonData = JSON.parse(textInput)
    const jsonString = JSON.stringify(jsonData)
    
    const newFile: JsonFile = {
      id: `text-${Date.now()}`,
      name: `pasted-${new Date().toISOString().slice(0, 19).replace(/:/g, '-')}.json`,
      content: jsonData,
      size: jsonString.length,  // Calculate size
      lastModified: new Date(),
      propertiesCount: Object.keys(jsonData).length
    }
    
    // Call parent callback
    const updated = [...jsonFiles, newFile]
    onFilesChange?.(updated)
    
    if (!activeFileId) {
      onActiveFileIdChange?.(newFile.id)
    }
    
    setTextInput("")
  } catch (error) {
    console.error('Failed to save JSON file:', error)
  }
}, [textInput, hasValidTextData, jsonFiles, activeFileId, onFilesChange, onActiveFileIdChange])
```

### 14.3 Mapper Component Enhancements

**Files**: 
- `components/json/response/json-result-mapper.tsx`
- `components/json/request/json-request-mapper.tsx`
- `components/json/response/json-response-mapper.tsx`

**Add File State** (after line 96 in all mappers):
```typescript
// Add file management state
const [jsonFiles, setJsonFiles] = useState<JsonFile[]>([])
const [activeFileId, setActiveFileId] = useState<string | null>(null)
```

**Update handleUploadDataChange** (lines 99-108 in all mappers):
```typescript
const handleUploadDataChange = useCallback((uploadData: {
  files: JsonFile[]
  activeFile: JsonFile | null
  textData: any | null
  combinedData: any | null
}) => {
  // Store files in mapper state
  setJsonFiles(uploadData.files)
  
  // Store active file ID
  if (uploadData.activeFile) {
    setActiveFileId(uploadData.activeFile.id)
  }
  
  // Notify parent form of data change
  if (onDataChange && uploadData.combinedData) {
    onDataChange(uploadData.combinedData)
  }
}, [onDataChange])
```

**Add File Management Callbacks** (after handleUploadDataChange):
```typescript
// Handle files change from SourceBodyUpload
const handleFilesChange = useCallback((files: JsonFile[]) => {
  setJsonFiles(files)
}, [])

// Handle active file change from SourceBodyUpload or JsonExplorer
const handleActiveFileIdChange = useCallback((id: string | null) => {
  setActiveFileId(id)
  
  // Update data when active file changes
  const file = jsonFiles.find(f => f.id === id)
  if (file && onDataChange) {
    onDataChange(file.content)
  }
}, [jsonFiles, onDataChange])

// Handle file removal from JsonExplorer/ManageJsonsDialog
const handleFileRemove = useCallback((fileId: string) => {
  setJsonFiles(prev => {
    const updated = prev.filter(f => f.id !== fileId)
    
    // If removed file was active, select next file
    if (activeFileId === fileId) {
      const nextFile = updated.length > 0 ? updated[0] : null
      setActiveFileId(nextFile?.id || null)
      
      // Update data with next file
      if (nextFile && onDataChange) {
        onDataChange(nextFile.content)
      } else if (onDataChange) {
        onDataChange(null)
      }
    }
    
    return updated
  })
}, [activeFileId, onDataChange])

// Handle file upload from ManageJsonsDialog
const handleFileUpload = useCallback((file: File) => {
  const reader = new FileReader()
  
  reader.onload = (e) => {
    try {
      const content = JSON.parse(e.target?.result as string)
      const newFile: JsonFile = {
        id: `dialog-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
        name: file.name,
        content,
        size: file.size,
        lastModified: new Date(file.lastModified),
        propertiesCount: Object.keys(content).length
      }
      
      setJsonFiles(prev => {
        const updated = [...prev, newFile]
        
        // Auto-select if no active file
        if (!activeFileId) {
          setActiveFileId(newFile.id)
          if (onDataChange) {
            onDataChange(newFile.content)
          }
        }
        
        return updated
      })
    } catch (error) {
      console.error("Failed to parse JSON file:", file.name, error)
    }
  }
  
  reader.readAsText(file)
}, [activeFileId, onDataChange])
```

**Update SourceBodyUpload Rendering** (lines 234-242 for JsonResultMapper, 179-184 for Request/Response):
```typescript
if (!data && showUpload) {
  return (
    <SourceBodyUpload
      files={jsonFiles}
      activeFileId={activeFileId}
      onFilesChange={handleFilesChange}
      onActiveFileIdChange={handleActiveFileIdChange}
      onJsonDataChange={handleUploadDataChange}
    />
  )
}
```

**Update onProcessedData Call** (lines 256-279 for JsonResultMapper, 199-206 for Request/Response):
```typescript
if (dataReady && onProcessedData && statistics) {
  return onProcessedData({
    basePaths,
    fullPaths,
    statistics,
    dataReady: true,
    // NEW: Add file management props
    files: jsonFiles,
    activeFileId,
    onFileSelect: handleActiveFileIdChange,
    onFileRemove: handleFileRemove,
    onFileUpload: handleFileUpload
  })
}
```

**Update Props Interface**:
```typescript
interface JsonResultMapperProps {
  data?: any
  objectsAndDatapointsMappings?: Mappings
  showUpload?: boolean
  uploadLabel?: string
  onDataChange?: (data: any) => void
  onProcessedData?: (processedData: {
    basePaths: Record<string, BasePathData>
    fullPaths: Array<FullPathData>
    statistics: MappingStatistics
    dataReady: boolean
    // NEW props for file management
    files: JsonFile[]
    activeFileId: string | null
    onFileSelect: (fileId: string) => void
    onFileRemove: (fileId: string) => void
    onFileUpload: (file: File) => void
  }) => React.ReactNode
}
```

### 14.4 Form Component Updates

**Files**:
- `components/forms/run-request-form.tsx`
- `components/forms/status-check-form.tsx`
- `components/forms/delivery-form.tsx`

**Update onProcessedData Callback** (lines 84-100 in RunRequestForm, similar in others):
```typescript
onProcessedData={(processedData) => (
  <JsonExplorer
    data={{}}
    basePaths={processedData.basePaths}
    fullPaths={processedData.fullPaths}
    statistics={processedData.statistics as any}
    mapperType="request"
    // NEW: Pass file management props
    files={processedData.files?.map(f => ({
      ...f,
      data: f.content,  // JsonExplorer expects 'data' property
      file: undefined as any  // Optional File object not available
    }))}
    currentFile={processedData.activeFileId}
    onFileSelect={processedData.onFileSelect}
    onFileRemove={processedData.onFileRemove}
    onFileUpload={processedData.onFileUpload}
    options={{
      showMapping: true,
      showStatus: true,
      showAccordion: true,
      defaultExpanded: false,
      showMetadata: true,
      compactMode: false
    }}
  />
)}
```

### 14.5 JsonExplorer Integration

**File**: `components/json/interface/json-explorer.tsx`

**Current State**: Already accepts all necessary props (lines 50-56, 66-70):
- `files?: JsonFile[]`
- `currentFile?: string | null`
- `onFileSelect?: (fileId: string) => void`
- `onFileRemove?: (fileId: string) => void`
- `onFileUpload?: (file: File) => void`

**Integration**: Passes props to JsonSelector (lines 410-426), which passes to ManageJsonsDialog (lines 83-91).

**Only Fix Needed**: JsonExplorer's JsonFile interface expects `data` property, but we standardized on `content`.

**Options**:
- **Option A**: Update JsonExplorer interface to use `content`
- **Option B**: Map in forms when passing (shown in 14.4 above)

### 14.6 ManageJsonsDialog Updates

**File**: `components/dialogs/manage-jsons-dialog.tsx`

**Interface Update** (lines 13-20):
```typescript
interface JsonFile {
  id: string
  name: string
  content: any  // Changed from 'data'
  size: number
  lastModified: Date
  propertiesCount: number
}
```

**No other changes needed** - component already handles everything correctly.

### 14.7 Complete Data Flow Diagrams

#### Flow 1: Initial Upload via SourceBodyUpload

```
User uploads file
    ↓
SourceBodyUpload.handleFileUpload
    ↓
onFilesChange(updated files) → Mapper.handleFilesChange
    ↓
Mapper.setJsonFiles(files)
    ↓
onActiveFileIdChange(fileId) → Mapper.handleActiveFileIdChange
    ↓
Mapper.setActiveFileId(id)
Mapper.onDataChange(file.content) → Form updates data state
    ↓
onJsonDataChange(uploadData) → Mapper.handleUploadDataChange
    ↓
Mapper receives combinedData
Form's data state updated → triggers re-render
    ↓
Mapper processes data → dataReady = true
    ↓
Mapper calls onProcessedData with files, activeFileId, callbacks
    ↓
Form renders JsonExplorer with file management props
    ↓
JsonExplorer shows JsonSelector with files in dropdown
```

#### Flow 2: Upload via ManageJsonsDialog

```
User clicks JsonSelector dropdown
    ↓
JsonSelector opens (variant="dialog")
    ↓
ManageJsonsDialog renders showing existing files
    ↓
User uploads file via FormDragDrop in dialog
    ↓
Dialog.handleFilesChange → Dialog.handleFileUpload
    ↓
onFileUpload(file) → JsonSelector prop
    ↓
JsonExplorer prop → onFileUpload
    ↓
Mapper.handleFileUpload
    ↓
FileReader parses file
    ↓
Mapper.setJsonFiles([...prev, newFile])
If no active file: setActiveFileId(newFile.id)
If no active file: onDataChange(newFile.content)
    ↓
Form's data state updated → triggers re-render
    ↓
Mapper processes new file → calls onProcessedData
    ↓
JsonExplorer re-renders with updated files
    ↓
ManageJsonsDialog shows new file in list
```

#### Flow 3: File Selection from Dialog

```
User clicks file in ManageJsonsDialog
    ↓
Dialog calls onFileSelect(fileId)
    ↓
JsonSelector prop → JsonExplorer prop
    ↓
Mapper.handleActiveFileIdChange(fileId)
    ↓
Mapper.setActiveFileId(fileId)
Mapper.onDataChange(file.content)
    ↓
Form's data state updated → triggers re-render
    ↓
Mapper processes with new active file
    ↓
JsonExplorer re-renders showing new active file
    ↓
ManageJsonsDialog highlights new active file
JsonSelector dropdown shows new selection
```

#### Flow 4: File Removal from Dialog

```
User clicks delete button in ManageJsonsDialog
    ↓
ConfirmDialog appears
    ↓
User confirms → onFileRemove(fileId)
    ↓
JsonSelector prop → JsonExplorer prop
    ↓
Mapper.handleFileRemove(fileId)
    ↓
Mapper.setJsonFiles(filtered)
If removed was active:
    - setActiveFileId(nextFile?.id || null)
    - onDataChange(nextFile?.content || null)
    ↓
Form's data state updated
    ↓
Mapper re-processes
    ↓
JsonExplorer re-renders without removed file
    ↓
ManageJsonsDialog shows updated file list
If no files remain: shows empty state
```

### 14.8 User Experience Scenarios

#### Scenario 1: Initial Upload
1. Form renders with no data
2. Mapper shows SourceBodyUpload (controlled by files state)
3. User drags JSON file to upload area
4. File parses, added to Mapper.jsonFiles state
5. Auto-selected as activeFileId
6. Mapper's data prop updates with file.content
7. **Automatic transition** → JsonExplorer renders
8. JsonSelector dropdown shows uploaded file
9. User can click dropdown → sees ManageJsonsDialog with file listed

#### Scenario 2: Multiple File Upload
1. User pastes JSON in "Paste as Text" tab
2. Clicks "Save as JSON file"
3. File added to Mapper.jsonFiles
4. JsonExplorer already visible, JsonSelector updates
5. User clicks "Add JSON" in JsonSelector
6. ManageJsonsDialog opens showing both files
7. User uploads third file via dialog
8. Dialog closes, all three files in dropdown
9. Active file remains selected

#### Scenario 3: Switching Files
1. User has 3 files loaded
2. Clicks JsonSelector dropdown
3. Selects different file
4. Mapper.handleActiveFileIdChange updates activeFileId
5. Mapper calls onDataChange with new file's content
6. Form updates data state
7. Mapper re-processes new file
8. JsonExplorer re-renders with new file's JSON structure
9. ProgressBar shows new file's statistics
10. ManageJsonsDialog (if open) highlights new selection

#### Scenario 4: File Management
1. User opens ManageJsonsDialog
2. Sees all uploaded files with metadata
3. Clicks file to select → becomes active
4. Clicks delete on another file → confirm dialog
5. Confirms deletion → file removed from list
6. Dropdown updates to show remaining files
7. Active file unchanged, continues displaying
8. Can upload more files, repeat cycle

### 14.9 Implementation Steps

#### Step 1: Standardize JsonFile Interface
1. Update ManageJsonsDialog to use `content` instead of `data`
2. Update JsonSelector to use `content` instead of `data`
3. Update JsonExplorer to use `content` OR add mapping in forms
4. Verify SourceBodyUpload already uses `content`
5. Test interface consistency

#### Step 2: Refactor SourceBodyUpload
1. Add `files`, `activeFileId`, `onFilesChange`, `onActiveFileIdChange` props
2. Remove internal `useState` for files and activeFileId
3. Use props with fallback instead of internal state
4. Update `handleFileUpload` to call parent callbacks
5. Update `handleGetByRequest` to call parent callbacks
6. Update `handleSaveAsJsonFile` to call parent callbacks
7. Add `size` property when creating JsonFile objects
8. Test controlled behavior

#### Step 3: Enhance All Three Mappers
1. Add `jsonFiles` and `activeFileId` state to each mapper
2. Update `handleUploadDataChange` to capture files from uploadData
3. Add `handleFilesChange` callback
4. Add `handleActiveFileIdChange` callback
5. Add `handleFileRemove` callback
6. Add `handleFileUpload` callback for dialog uploads
7. Pass controlled props to SourceBodyUpload
8. Pass file management props to onProcessedData callback
9. Update mapper interfaces to include file props
10. Test each mapper independently

#### Step 4: Update Form Components
1. Update `onProcessedData` callback in RunRequestForm
2. Map file props from mappers to JsonExplorer
3. Convert `content` to `data` if needed for JsonExplorer
4. Repeat for StatusCheckForm
5. Repeat for DeliveryForm
6. Test forms with file management

#### Step 5: Testing and Validation
1. Test initial file upload through SourceBodyUpload
2. Test files appearing in JsonSelector dropdown
3. Test opening ManageJsonsDialog and seeing files
4. Test uploading files through ManageJsonsDialog
5. Test file selection from dropdown
6. Test file removal from dialog
7. Test switching between files
8. Test automatic rendering still works
9. Test all three form types
10. Verify bidirectional file flow

### 14.10 Files to Modify

**Component Files (9 files)**:
1. `components/json/interface/source-body-upload.tsx` - Convert to controlled component
2. `components/json/response/json-result-mapper.tsx` - Add file management
3. `components/json/request/json-request-mapper.tsx` - Add file management
4. `components/json/response/json-response-mapper.tsx` - Add file management
5. `components/forms/run-request-form.tsx` - Pass file props to explorer
6. `components/forms/status-check-form.tsx` - Pass file props to explorer
7. `components/forms/delivery-form.tsx` - Pass file props to explorer
8. `components/dialogs/manage-jsons-dialog.tsx` - Update interface
9. `components/selectors/json-selector.tsx` - Update interface

**Optional (if not mapping in forms)**:
10. `components/json/interface/json-explorer.tsx` - Update interface

### 14.11 Success Criteria

**Functionality**:
- Files uploaded via SourceBodyUpload appear in JsonSelector dropdown
- Files uploaded via ManageJsonsDialog appear in all views
- File selection works from JsonSelector
- File removal works from ManageJsonsDialog
- Active file synchronizes across all components
- Automatic rendering still works seamlessly

**Architecture**:
- Clean separation: SourceBodyUpload for upload, Mappers for state, JsonExplorer for display
- Controlled component pattern properly implemented
- Single source of truth for file state (mappers)
- Bidirectional data flow working correctly

**User Experience**:
- No jarring transitions or component switches
- Files visible everywhere they should be
- Intuitive file management interface
- Consistent behavior across all forms
- No duplicate files or state synchronization issues

**Code Quality**:
- Consistent JsonFile interface across all components
- Clean, maintainable callback patterns
- Proper TypeScript typing throughout
- No unnecessary re-renders or state updates
- Well-documented component relationships

This phase completes the file management integration, creating a unified system where files flow seamlessly between SourceBodyUpload, mappers, JsonExplorer, and ManageJsonsDialog, all while preserving the automatic rendering behavior that transitions smoothly from upload to mapping views.

## Phase 15: Architectural Refactor - Component State Ownership and Form Orchestration

### Overview
Eliminate unnecessary re-renders by refactoring to proper component-level state ownership. Components manage their own state internally and expose data to parent forms via refs. Forms become orchestrators responsible for collecting component data, validation, and assembling final JSON templates for submission.

### Executive Summary

**Goal**: Fix the re-rendering cascade by moving state ownership from forms to components, implementing ref-based data collection, and establishing forms as orchestrators rather than state managers.

**Current Architecture Problem**:
Forms lift all state from components:
- Form owns metadata state (method, url, headers)
- Form owns load balancing state (concurrency, timeout)
- Form owns JSON data state (requestData, responseData)
- Every keystroke in ANY component updates form state
- Form re-renders trigger cascade through all children
- Mappers reprocess JSON on every form re-render
- 100ms+ lag, broken focus management

**Root Cause**: State ownership violation - forms manage state that belongs to components, creating tight coupling and re-render cascade.

**Solution**: Move state ownership to components. Forms collect data only on submit via refs. Components become autonomous, forms become orchestrators.

**Impact**:
- Typing in URL: No form re-render, no cascade
- Focus redirection: Works immediately
- JSON reprocessing: Only when JSON actually changes
- Component independence: Each component manages itself
- Form simplicity: Just orchestration and validation
- User experience: Instant, smooth, professional

### 15.1 Complete Component Inventory and Current State Analysis

#### Form Components (3 Total)

**RunRequestForm** - `components/forms/run-request-form.tsx`
- Current state ownership: headers, method, url, loadBalancing, requestData, responseData
- Child components: RequestMetadataForm, JsonRequestMapper, JsonResponseMapper, LoadBalancingForm
- Unique features: LoadBalancingForm (concurrency, timeout)
- Output template: run_request_template with request/response variable mappings

**StatusCheckForm** - `components/forms/status-check-form.tsx`
- Current state ownership: statusUrl, checkMethod, headers, requestData, responseData
- Child components: RequestMetadataForm, JsonRequestMapper, JsonResponseMapper
- **⚠️ Issue 1 - Unnecessary Field Transformation**: Stores as checkMethod/statusUrl but transforms to method/url for RequestMetadataForm (lines 145-148), no other form does this
- Output template: status_request_template with request/response variable mappings
- **Fix Required**: Standardize to use method/url like RunRequestForm and DeliveryForm

**DeliveryForm** - `components/forms/delivery-form.tsx`
- Current state ownership: url, method, headers, requestData, resultData
- Child components: RequestMetadataForm, JsonRequestMapper, JsonResultMapper
- Unique features: Uses JsonResultMapper (not JsonResponseMapper)
- **⚠️ Issue 2 - Variable Name Mismatch**: Variable named `responseMapper` but contains JsonResultMapper (line 99), should be `resultMapper`
- Output template: delivery_request_template with object/datapoint mappings (not variable mappings)
- **Fix Required**: Rename `responseMapper` variable to `resultMapper` for clarity

#### Form Sub-Components (2 Total)

**RequestMetadataForm** - `components/forms/blocks/request-metadata.tsx`
- Current state: Has internal state for metadata and headers (lines 43-46)
- Current props: Receives initial values + onChange callbacks
- Current behavior: Hybrid - has internal state but calls parent callbacks on every change
- Current problem: Duplicate state causes unnecessary re-renders and focus issues
- Used by: All three forms (with different field mappings)

**LoadBalancingForm** - `components/forms/blocks/load-balancing.tsx`
- Current state: Has internal state for concurrency and timeout (lines 27-28)
- Current props: Receives initial values + onChange callbacks  
- Current behavior: Hybrid - has internal state but calls parent callbacks on every change
- Current problem: Every keystroke triggers parent form re-render
- Used by: Only RunRequestForm

#### Mapper Components (3 Total)

**JsonRequestMapper** - `components/json/request/json-request-mapper.tsx`
- Current props interface (lines 44-61):
  - `data?: any` - JSON data to process
  - `requestAndResponseVariableMappings: RequestAndResponseVariableMappings` - variable mapping config
  - `showUpload?: boolean` - whether to show upload interface
  - `uploadLabel?: string` - upload button label
  - `onDataChange?: (data: any) => void` - callback when data changes
  - `onProcessedData?: (processedData) => React.ReactNode` - render prop for processed data
- Internal state: basePaths, fullPaths, processing flags, jsonFiles, activeFileId
- Processing pipeline: 6 sequential useEffects (lines 185-256)
- File management: Built-in (lines 84-86)
- Current problem: Reprocesses on every prop change, even when data unchanged
- Used by: All three forms

**JsonResponseMapper** - `components/json/response/json-response-mapper.tsx`
- Current props interface (lines 44-61):
  - Same as JsonRequestMapper
  - Uses `requestAndResponseVariableMappings` for response variable mappings
- Internal state: Same as JsonRequestMapper
- Processing pipeline: Identical to JsonRequestMapper
- File management: Built-in (lines 84-86)
- Current problem: Same reprocessing issue as JsonRequestMapper
- Used by: RunRequestForm and StatusCheckForm

**JsonResultMapper** - `components/json/response/json-result-mapper.tsx`
- Current props interface (lines 52-69):
  - `data?: any` - JSON data to process
  - `objectsAndDatapointsMappings?: Mappings` - object/datapoint mapping config (different from variable mappings)
  - `showUpload?: boolean` - whether to show upload interface
  - `uploadLabel?: string` - upload button label
  - `onDataChange?: (data: any) => void` - callback when data changes
  - `onProcessedData?: (processedData) => React.ReactNode` - render prop for processed data
- Internal state: basePaths, fullPaths (with objectMappings, datapointMappings), processing flags, jsonFiles, activeFileId
- Processing pipeline: Similar to other mappers but with object/datapoint logic
- File management: Built-in
- Current problem: Same reprocessing issue
- Used by: Only DeliveryForm
- Key difference: Maps to object types and datapoints instead of variables

#### Supporting Components (Not Modified in This Phase)

**SourceRequestResponseAccordion** - `components/accordions/source-request-response-accordion.tsx`
- Purpose: Wrapper that organizes RequestMetadataForm and mappers
- Current behavior: Receives mapper components as props, renders them
- No state management: Just presentation wrapper
- Not modified: No changes needed, receives components as children

**SourceBodyUpload** - `components/json/interface/source-body-upload.tsx`
- Purpose: File upload interface with tabs (upload/text/request)
- Current behavior: Controlled component (after Phase 14)
- Props: files, activeFileId, onFilesChange, onActiveFileIdChange, onJsonDataChange
- Used by: All three mappers internally
- Not modified: Already has proper interface

**JsonExplorer** - `components/json/interface/json-explorer.tsx`
- Purpose: Displays processed JSON with mappings
- Current behavior: Receives processed data from mappers
- Props: basePaths, fullPaths, statistics, files, currentFile, file management callbacks
- Used by: All mappers via onProcessedData render prop
- Not modified: Just receives data, doesn't manage it

#### Identified Inconsistencies to Fix

**Issue 1: StatusCheckForm Field Name Transformation**
- **Location**: StatusCheckForm lines 10-11, 29-30, 145, 147-148
- **Problem**: Uses `checkMethod` and `statusUrl` internally, transforms to/from `method` and `url` for RequestMetadataForm
- **Impact**: Unnecessary complexity, no other form does this, potential for bugs
- **Fix**: Standardize to use `method` and `url` like RunRequestForm and DeliveryForm
- **Benefit**: Consistent field naming, simpler code, no transformation logic

**Issue 2: DeliveryForm Variable Name Mismatch**
- **Location**: DeliveryForm lines 99, 157
- **Problem**: Variable named `responseMapper` contains `JsonResultMapper` component
- **Impact**: Confusing naming, misleading variable name
- **Fix**: Rename variable from `responseMapper` to `resultMapper`
- **Benefit**: Clear naming that matches component type and state variable (resultData)

Both fixes are incorporated into Phase 15 implementation steps and will be applied during form refactoring.

### 15.2 Target Architecture - Component State Ownership with Ref-Based Data Collection

#### Architecture Principles

**Component Autonomy**:
- Each component owns and manages its own state
- Components are self-contained and reusable
- Components don't notify parents on intermediate changes
- Components expose final data via refs when requested

**Form Orchestration**:
- Forms hold refs to all child components
- Forms don't manage component state
- Forms collect data only when needed (validation, submit)
- Forms assemble component data into final JSON templates
- Forms handle overall validation logic

**Data Flow**:
- User interaction → Component internal state updates (no parent notification)
- Form submit → Form calls ref.current.getData() on each component
- Component returns its current state via ref
- Form assembles all component data into JSON template
- Form validates complete data
- Form calls onSubmit with assembled template

#### Ref-Based Interface Pattern

**Component Exposes Data via useImperativeHandle**:
```typescript
interface ComponentRefHandle {
  getData: () => ComponentData
  validate: () => boolean | ValidationErrors
  reset?: () => void
}

export const ComponentName = forwardRef<ComponentRefHandle, ComponentProps>((props, ref) => {
  const [internalState, setInternalState] = useState(...)
  
  useImperativeHandle(ref, () => ({
    getData: () => internalState,
    validate: () => /* validation logic */,
    reset: () => setInternalState(initialState)
  }))
  
  return (/* component UI */)
})
```

**Form Collects Data via Refs**:
```typescript
export function FormName({ onSubmit }) {
  const componentRef = useRef<ComponentRefHandle>(null)
  
  const handleSubmit = (e) => {
    e.preventDefault()
    
    const data = componentRef.current?.getData()
    const isValid = componentRef.current?.validate()
    
    if (isValid) {
      onSubmit(assembleTemplate(data))
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <ComponentName ref={componentRef} />
    </form>
  )
}
```

### 15.3 Component-Specific Refactoring Requirements

#### 15.3.1 RequestMetadataForm Refactoring

**File**: `components/forms/blocks/request-metadata.tsx`

**Current State** (lines 43-46):
- Has internal state for metadata and headers
- Calls parent callbacks on every change (immediate notification)
- Hybrid pattern causes duplicate state and re-render issues

**Target State**: Autonomous component with ref-based data exposure

**New Props Interface**:
```typescript
interface RequestMetadataFormProps {
  initialMetadata?: { method: string, url: string }
  initialHeaders?: RequestHeader[]
  // Remove: onMetadataChange, onHeadersChange (no parent notifications)
}
```

**New Ref Interface**:
```typescript
interface RequestMetadataFormRef {
  getData: () => {
    metadata: { method: string, url: string }
    headers: RequestHeader[]
  }
  validate: () => boolean | {
    metadataErrors?: string[]
    headerErrors?: Array<{id: string, error: string}>
  }
  reset: () => void
}
```

**Internal State Management**:
- Keeps internal state for metadata and headers
- Removes all parent callback calls
- User types → internal state updates only
- No form re-renders triggered
- Focus management works correctly

**Implementation Approach**:
- Convert to forwardRef component
- Add useImperativeHandle for getData, validate, reset
- Remove onMetadataChange and onHeadersChange calls
- Keep all existing UI and interaction logic
- Ensure headers always has at least one empty row

**Validation Logic**:
- getData: Returns current state as-is
- validate: Checks method is set, url is valid, headers are complete
- reset: Returns to initial values

#### 15.3.2 LoadBalancingForm Refactoring

**File**: `components/forms/blocks/load-balancing.tsx`

**Current State** (lines 27-28):
- Has internal state for concurrency and timeout
- Calls parent callbacks on every change
- Same hybrid pattern as RequestMetadataForm

**Target State**: Autonomous component with ref-based data exposure

**New Props Interface**:
```typescript
interface LoadBalancingFormProps {
  initialConcurrency?: number
  initialTimeout?: number
  // Remove: onConcurrencyChange, onTimeoutChange
  title?: string
  description?: string
  collapsible?: boolean
  defaultExpanded?: boolean
}
```

**New Ref Interface**:
```typescript
interface LoadBalancingFormRef {
  getData: () => {
    concurrency: number
    timeout: number
  }
  validate: () => boolean | {
    concurrencyError?: string
    timeoutError?: string
  }
  reset: () => void
}
```

**Internal State Management**:
- Keeps internal state for concurrency and timeout
- Removes parent callback calls
- User types → internal state updates only
- No form re-renders

**Implementation Approach**:
- Convert to forwardRef component
- Add useImperativeHandle for getData, validate, reset
- Remove onConcurrencyChange and onTimeoutChange calls
- Keep all existing NumberInput logic

**Validation Logic**:
- validate: Checks concurrency > 0, timeout > 0
- getData: Returns current values
- reset: Returns to initial values (10, 30)

#### 15.3.3 JsonRequestMapper Refactoring

**File**: `components/json/request/json-request-mapper.tsx`

**Current Behavior**:
- Already manages its own state (basePaths, fullPaths, jsonFiles, activeFileId)
- Uses render prop pattern (onProcessedData) to pass processed data to parent
- Has onDataChange callback that notifies parent of raw JSON changes

**Target State**: Add ref-based data exposure while keeping existing render prop

**Keep Current Props**:
- data?: any (for pre-loading existing JSON)
- requestAndResponseVariableMappings (for existing mappings to display)
- showUpload, uploadLabel (UI controls)
- onProcessedData (render prop for JsonExplorer - keep as is)

**New Ref Interface**:
```typescript
interface JsonRequestMapperRef {
  getData: () => {
    jsonData: any | null
    files: JsonFile[]
    activeFileId: string | null
    variableMappings: {
      [variableName: string]: {
        path: string[]
        status: string
      }
    }
  }
  validate: () => boolean | {
    jsonError?: string
    mappingErrors?: string[]
  }
  reset: () => void
}
```

**Implementation Approach**:
- Convert to forwardRef component
- Add useImperativeHandle
- Keep all existing processing pipeline (generatePathData, assignVariableToFullPath, etc.)
- Keep onProcessedData render prop pattern
- Remove onDataChange callback (parent gets data via ref.current.getData())
- getData extracts variable mappings from processed fullPaths

**Data Return Structure**:
- jsonData: Current active file's content
- files: All uploaded JSON files
- activeFileId: Currently selected file
- variableMappings: Extracted from fullPaths where variableMappings exist

**Validation Logic**:
- validate: Checks if JSON is uploaded, if required variables are mapped
- Returns errors for missing required mappings

#### 15.3.4 JsonResponseMapper Refactoring

**File**: `components/json/response/json-response-mapper.tsx`

**Current Behavior**:
- Identical to JsonRequestMapper
- Uses requestAndResponseVariableMappings for response variables
- Same state management and processing pipeline

**Target State**: Add ref-based data exposure (same as JsonRequestMapper)

**New Ref Interface**: (Identical to JsonRequestMapper)
```typescript
interface JsonResponseMapperRef {
  getData: () => {
    jsonData: any | null
    files: JsonFile[]
    activeFileId: string | null
    variableMappings: {
      [variableName: string]: {
        path: string[]
        status: string
        translations?: { [key: string]: string }
      }
    }
  }
  validate: () => boolean | { jsonError?: string, mappingErrors?: string[] }
  reset: () => void
}
```

**Implementation Approach**: Same as JsonRequestMapper

**Note**: Response mapper includes optional translations field in variableMappings

#### 15.3.5 JsonResultMapper Refactoring

**File**: `components/json/response/json-result-mapper.tsx`

**Current Behavior**:
- Similar to other mappers but with different mapping structure
- Uses objectsAndDatapointsMappings instead of requestAndResponseVariableMappings
- Processes object type mappings and datapoint mappings

**Target State**: Add ref-based data exposure with object/datapoint structure

**New Ref Interface**: (Different from request/response mappers)
```typescript
interface JsonResultMapperRef {
  getData: () => {
    jsonData: any | null
    files: JsonFile[]
    activeFileId: string | null
    objectMappings: Array<{
      object_type_id: string
      key_mapping: string[]
      key_mapping_type: "value" | "key"
      status: "mapped" | "recommendation" | "issue"
    }>
    datapointMappings: Array<{
      datapoint_id: string
      object_type_id: string
      key_mapping: string[]
      key_mapping_type: "value" | "key"
      status: "mapped" | "recommendation" | "issue"
    }>
  }
  validate: () => boolean | {
    jsonError?: string
    objectMappingErrors?: string[]
    datapointMappingErrors?: string[]
  }
  reset: () => void
}
```

**Implementation Approach**:
- Convert to forwardRef component
- Add useImperativeHandle
- Keep all existing processing pipeline
- Keep onProcessedData render prop
- Remove onDataChange callback
- getData extracts object/datapoint mappings from processed fullPaths

**Data Return Structure**:
- jsonData: Current active file's content
- files: All uploaded JSON files
- activeFileId: Currently selected file
- objectMappings: Extracted from fullPaths.objectMappings
- datapointMappings: Extracted from fullPaths.datapointMappings

**Validation Logic**:
- validate: Checks if JSON uploaded, if required object types mapped, if required datapoints mapped
- Returns separate errors for object mappings vs datapoint mappings

### 15.4 Form-Specific Implementations

#### 15.4.1 RunRequestForm Refactoring

**File**: `components/forms/run-request-form.tsx`

**Current State**:
- Owns all state: formData (method, url, headers, loadBalancing), requestData, responseData
- Passes data via props to all components
- Every component change triggers form re-render

**Target State**: Ref-based orchestrator

**New Form Structure**:
```typescript
interface RunRequestFormProps {
  initialData?: {
    metadata?: { method: string, url: string }
    headers?: RequestHeader[]
    loadBalancing?: { concurrency: number, timeout: number }
    // Can include initial JSON data if editing existing source
  }
  onSubmit: (template: RunRequestTemplate) => void
  isLoading?: boolean
}

interface RunRequestTemplate {
  run_request_template: {
    method: string
    url: string
    headers: RequestHeader[]
    body?: any // From request JSON
    // Variable mappings embedded in body template
  }
  response_template?: {
    // Variable mappings from response JSON
  }
  timeout: number
  max_concurrency: number
}
```

**Ref Declarations**:
```typescript
const requestMetadataRef = useRef<RequestMetadataFormRef>(null)
const loadBalancingRef = useRef<LoadBalancingFormRef>(null)
const requestMapperRef = useRef<JsonRequestMapperRef>(null)
const responseMapperRef = useRef<JsonResponseMapperRef>(null)
```

**Form Submit Handler**:
```typescript
const handleSubmit = (e: FormEvent) => {
  e.preventDefault()
  
  // Collect data from all components via refs
  const metadataData = requestMetadataRef.current?.getData()
  const loadBalancingData = loadBalancingRef.current?.getData()
  const requestMapperData = requestMapperRef.current?.getData()
  const responseMapperData = responseMapperRef.current?.getData()
  
  // Validate all components
  const metadataValid = requestMetadataRef.current?.validate()
  const loadBalancingValid = loadBalancingRef.current?.validate()
  const requestMapperValid = requestMapperRef.current?.validate()
  const responseMapperValid = responseMapperRef.current?.validate()
  
  if (!metadataValid || !loadBalancingValid || !requestMapperValid || !responseMapperValid) {
    // Show validation errors
    return
  }
  
  // Assemble run_request_template
  const template: RunRequestTemplate = {
    run_request_template: {
      method: metadataData.metadata.method,
      url: metadataData.metadata.url,
      headers: metadataData.headers,
      body: requestMapperData.jsonData,
      // Embed variable mappings from requestMapperData.variableMappings
    },
    response_template: {
      // Embed variable mappings from responseMapperData.variableMappings
    },
    timeout: loadBalancingData.timeout,
    max_concurrency: loadBalancingData.concurrency
  }
  
  onSubmit(template)
}
```

**Render Structure**:
```typescript
return (
  <form onSubmit={handleSubmit}>
    <SourceRequestResponseAccordion type="request" requestType="run_request">
      <RequestMetadataForm ref={requestMetadataRef} initialMetadata={...} />
      <JsonRequestMapper ref={requestMapperRef} />
    </SourceRequestResponseAccordion>
    
    <SourceRequestResponseAccordion type="response" requestType="run_request">
      <JsonResponseMapper ref={responseMapperRef} />
    </SourceRequestResponseAccordion>
    
    <LoadBalancingForm ref={loadBalancingRef} />
    
    <PrimaryButton type="submit">Save Configuration</PrimaryButton>
  </form>
)
```

**Key Changes**:
- No formData state
- No requestData/responseData state
- No onChange callbacks
- Data collected only on submit via refs
- Component changes don't trigger form re-renders

#### 15.4.2 StatusCheckForm Refactoring

**File**: `components/forms/status-check-form.tsx`

**Current State**:
- Owns state: formData (statusUrl, checkMethod, headers), requestData, responseData
- Same re-render issues as RunRequestForm
- **Issue 1**: Unnecessary field name transformation between form and component

**Issue 1 Fix - Standardize Field Names**:

**Current Problem**:
- Lines 10-11: FormData interface uses `statusUrl` and `checkMethod`
- Lines 29-30: State initialization uses `statusUrl` and `checkMethod`
- Line 145: Transforms to `{ method: formData.checkMethod, url: formData.statusUrl }` when passing to RequestMetadataForm
- Lines 147-148: Transforms back `checkMethod = metadata.method`, `statusUrl = metadata.url`
- No other form does this transformation

**Solution - Standardize to method/url**:
- Change FormData interface fields from `statusUrl, checkMethod` to `url, method`
- Remove transformation logic (lines 145, 147-148)
- Match RunRequestForm and DeliveryForm pattern
- Simpler, clearer, consistent

**Target State**: Ref-based orchestrator with standardized field names

**New Form Structure**:
```typescript
interface StatusCheckFormProps {
  initialData?: {
    metadata?: { method: string, url: string } // Changed from checkMethod/statusUrl
    headers?: RequestHeader[]
  }
  onSubmit: (template: StatusCheckTemplate) => void
  isLoading?: boolean
}

interface StatusCheckTemplate {
  status_request_template: {
    method: string
    url: string
    headers: RequestHeader[]
    body?: any // From request JSON
  }
  status_response_template?: {
    // Variable mappings from response JSON
  }
}
```

**Ref Declarations**:
```typescript
const requestMetadataRef = useRef<RequestMetadataFormRef>(null)
const requestMapperRef = useRef<JsonRequestMapperRef>(null)
const responseMapperRef = useRef<JsonResponseMapperRef>(null)
```

**Form Submit Handler**:
```typescript
const handleSubmit = (e: FormEvent) => {
  e.preventDefault()
  
  const metadataData = requestMetadataRef.current?.getData()
  const requestMapperData = requestMapperRef.current?.getData()
  const responseMapperData = responseMapperRef.current?.getData()
  
  // Validate
  const metadataValid = requestMetadataRef.current?.validate()
  const requestMapperValid = requestMapperRef.current?.validate()
  const responseMapperValid = responseMapperRef.current?.validate()
  
  if (!metadataValid || !requestMapperValid || !responseMapperValid) {
    return
  }
  
  // Assemble status_request_template
  const template: StatusCheckTemplate = {
    status_request_template: {
      method: metadataData.metadata.method, // No transformation needed after fix
      url: metadataData.metadata.url, // No transformation needed after fix
      headers: metadataData.headers,
      body: requestMapperData.jsonData,
    },
    status_response_template: {
      // Embed variable mappings from responseMapperData.variableMappings
    }
  }
  
  onSubmit(template)
}
```

**Key Differences from RunRequestForm**:
- No LoadBalancingForm component
- Otherwise identical pattern (after Issue 1 fix)

#### 15.4.3 DeliveryForm Refactoring

**File**: `components/forms/delivery-form.tsx`

**Current State**:
- Owns state: formData (url, method, headers), requestData, resultData
- Uses JsonResultMapper (not JsonResponseMapper)
- Same re-render issues
- **Issue 2**: Variable named `responseMapper` but contains JsonResultMapper

**Issue 2 Fix - Rename Variable to Match Component**:

**Current Problem**:
- Line 99: Variable declared as `const responseMapper = (`
- Line 100: Contains `<JsonResultMapper` component (not JsonResponseMapper)
- Line 157: Passed as `resultMapper={responseMapper}` to accordion
- Naming mismatch: variable name suggests response but it's actually result

**Solution - Rename to resultMapper**:
- Change variable declaration from `responseMapper` to `resultMapper` (line 99)
- Update accordion prop to use same variable: `resultMapper={resultMapper}` (line 157)
- Matches component type (JsonResultMapper)
- Consistent with state variable naming (resultData)
- Clear and unambiguous

**Target State**: Ref-based orchestrator with standardized naming

**New Form Structure**:
```typescript
interface DeliveryFormProps {
  initialData?: {
    metadata?: { method: string, url: string }
    headers?: RequestHeader[]
  }
  onSubmit: (template: DeliveryTemplate) => void
  isLoading?: boolean
}

interface DeliveryTemplate {
  delivery_request_template: {
    method: string
    url: string
    headers: RequestHeader[]
    body?: any // From request JSON
  }
  object_source_mappings: Array<ObjectSourceMapping>
  datapoint_source_mappings: Array<DatapointSourceMapping>
}
```

**Ref Declarations**:
```typescript
const requestMetadataRef = useRef<RequestMetadataFormRef>(null)
const requestMapperRef = useRef<JsonRequestMapperRef>(null)
const resultMapperRef = useRef<JsonResultMapperRef>(null) // Note: ResultMapper, not ResponseMapper
```

**Form Submit Handler**:
```typescript
const handleSubmit = (e: FormEvent) => {
  e.preventDefault()
  
  const metadataData = requestMetadataRef.current?.getData()
  const requestMapperData = requestMapperRef.current?.getData()
  const resultMapperData = resultMapperRef.current?.getData()
  
  // Validate
  const metadataValid = requestMetadataRef.current?.validate()
  const requestMapperValid = requestMapperRef.current?.validate()
  const resultMapperValid = resultMapperRef.current?.validate()
  
  if (!metadataValid || !requestMapperValid || !resultMapperValid) {
    return
  }
  
  // Assemble delivery_request_template
  const template: DeliveryTemplate = {
    delivery_request_template: {
      method: metadataData.metadata.method,
      url: metadataData.metadata.url,
      headers: metadataData.headers,
      body: requestMapperData.jsonData,
    },
    object_source_mappings: resultMapperData.objectMappings,
    datapoint_source_mappings: resultMapperData.datapointMappings
  }
  
  onSubmit(template)
}
```

**Key Differences from Other Forms**:
- Uses JsonResultMapper instead of JsonResponseMapper
- Returns objectMappings and datapointMappings instead of variableMappings
- Different template structure (object/datapoint mappings)
- Variable renamed from `responseMapper` to `resultMapper` (Issue 2 fix)

### 15.5 Implementation Steps

#### Step 1: Refactor Form Sub-Components (2 components)

**RequestMetadataForm**:
1. Convert to forwardRef component
2. Remove all onChange callback props
3. Remove internal state sync with props
4. Add useImperativeHandle with getData, validate, reset
5. Keep all existing UI logic
6. Test focus redirection works correctly

**LoadBalancingForm**:
1. Convert to forwardRef component  
2. Remove onConcurrencyChange and onTimeoutChange props
3. Add useImperativeHandle with getData, validate, reset
4. Keep all existing NumberInput logic
5. Test component isolation

#### Step 2: Refactor Mapper Components (3 components)

**JsonRequestMapper**:
1. Convert to forwardRef component
2. Add useImperativeHandle with getData, validate, reset
3. Remove onDataChange callback
4. Keep onProcessedData render prop (for JsonExplorer)
5. getData extracts variableMappings from processed fullPaths
6. Test JSON processing still works

**JsonResponseMapper**:
1. Same changes as JsonRequestMapper
2. Include translations in variableMappings extraction
3. Test with response data

**JsonResultMapper**:
1. Convert to forwardRef component
2. Add useImperativeHandle with getData, validate, reset
3. Remove onDataChange callback
4. getData extracts objectMappings and datapointMappings
5. Test object/datapoint mapping extraction

#### Step 3: Refactor Form Components (3 forms)

**RunRequestForm**:
1. Remove all state (formData, requestData, responseData)
2. Add refs for all child components
3. Remove all onChange callbacks
4. Implement handleSubmit with ref.current.getData()
5. Assemble run_request_template from component data
6. Test form submission
7. Test that typing doesn't cause form re-renders

**StatusCheckForm**:
1. **Issue 1 Fix**: Change FormData interface fields from `statusUrl, checkMethod` to `url, method`
2. **Issue 1 Fix**: Update state initialization to use `method, url` instead of `checkMethod, statusUrl`
3. **Issue 1 Fix**: Remove transformation logic at lines 145, 147-148
4. Remove all state (formData, requestData, responseData)
5. Add refs for all child components
6. Remove all onChange callbacks
7. Implement handleSubmit with ref.current.getData()
8. Assemble status_request_template from component data (no transformation needed)
9. Test form submission
10. Verify field names are consistent throughout

**DeliveryForm**:
1. **Issue 2 Fix**: Rename variable from `responseMapper` to `resultMapper` (line 99)
2. **Issue 2 Fix**: Update accordion prop from `resultMapper={responseMapper}` to `resultMapper={resultMapper}` (line 157)
3. Remove all state (formData, requestData, resultData)
4. Add refs for all child components  
5. Remove all onChange callbacks
6. Implement handleSubmit with ref.current.getData()
7. Assemble delivery_request_template with object/datapoint mappings
8. Test form submission
9. Verify variable naming is clear throughout

#### Step 4: Update Accordion Integration

**SourceRequestResponseAccordion**:
- No changes needed
- Already receives components as props
- Continues to render them in accordion wrapper

#### Step 5: Verify Naming and Consistency Fixes

**Issue 1 Verification (StatusCheckForm)**:
1. Verify FormData interface uses `method, url` (not `checkMethod, statusUrl`)
2. Verify state initialization uses `method, url`
3. Verify no transformation logic exists in accordion props
4. Verify form submit handler uses direct field access
5. Test that StatusCheckForm behaves identically to RunRequestForm

**Issue 2 Verification (DeliveryForm)**:
1. Verify variable named `resultMapper` (not `responseMapper`)
2. Verify accordion receives `resultMapper={resultMapper}`
3. Verify all references use `resultMapper` consistently
4. Test that naming is clear and matches component type

#### Step 6: Testing and Validation

1. Test typing in URL field - should not trigger form re-render
2. Test changing method dropdown - focus should redirect to URL
3. Test adding/removing headers - should not trigger form re-render
4. Test uploading JSON files - should work as before
5. Test switching between files - should work as before
6. Test form submission collects all data correctly
7. Test validation shows errors correctly
8. Test all three forms have identical patterns (after Issue 1 and 2 fixes)
9. Verify StatusCheckForm uses method/url consistently
10. Verify DeliveryForm uses resultMapper naming consistently
11. Verify no console errors or warnings
12. Measure performance improvements

### 15.6 Files to Modify

**Form Sub-Components (2 files)**:
1. `components/forms/blocks/request-metadata.tsx` - Add forwardRef + useImperativeHandle
2. `components/forms/blocks/load-balancing.tsx` - Add forwardRef + useImperativeHandle

**Mapper Components (3 files)**:
3. `components/json/request/json-request-mapper.tsx` - Add forwardRef + useImperativeHandle
4. `components/json/response/json-response-mapper.tsx` - Add forwardRef + useImperativeHandle
5. `components/json/response/json-result-mapper.tsx` - Add forwardRef + useImperativeHandle

**Form Components (3 files)**:
6. `components/forms/run-request-form.tsx` - Remove state, add refs, collect data on submit
7. `components/forms/status-check-form.tsx` - **Issue 1 Fix**: Standardize to method/url fields, remove transformation, remove state, add refs
8. `components/forms/delivery-form.tsx` - **Issue 2 Fix**: Rename responseMapper to resultMapper, remove state, add refs

**Total Files**: 8 files

**Specific Line Changes for Issue Fixes**:

**StatusCheckForm** (Issue 1):
- Lines 10-12: Change `statusUrl: string, checkMethod: string` to `url: string, method: string`
- Lines 29-31: Change state init to `url: initialData.url, method: initialData.method`
- Line 145: Remove transformation, use `{ method: formData.method, url: formData.url }`
- Lines 147-148: Remove transformation, use `updateField('method', metadata.method)` and `updateField('url', metadata.url)`

**DeliveryForm** (Issue 2):
- Line 99: Change `const responseMapper = (` to `const resultMapper = (`
- Line 157: Change `resultMapper={responseMapper}` to `resultMapper={resultMapper}`

### 15.7 Success Criteria

**Performance**:
- Typing in URL responds instantly (no form re-render)
- Focus redirection works 100% of the time
- No unnecessary JSON reprocessing
- Component changes isolated to component only
- Smooth, lag-free user experience

**Functionality**:
- All form features work identically to before
- JSON upload and processing unchanged
- File management works correctly
- Form submission collects all data via refs
- Validation works correctly
- Template assembly works correctly

**Architecture**:
- Components own their state
- Forms orchestrate via refs
- No lifted state in forms
- Clean component boundaries
- Easy to test in isolation

**Consistency**:
- All forms use identical ref-based pattern
- All components expose identical ref interface structure
- StatusCheckForm standardized to use method/url (Issue 1 fixed)
- DeliveryForm uses resultMapper variable name (Issue 2 fixed)
- Code structure is consistent across all forms
- Easy to maintain and extend

**Code Quality**:
- No circular dependencies
- No prop drilling
- Clear data flow via refs
- Well-documented ref interfaces
- TypeScript types correct throughout

**User Experience**:
- Instant feedback on all interactions
- Smooth interactions throughout
- No stuttering or lag
- Focus management works correctly
- Professional, polished feel

This phase eliminates the re-rendering cascade by moving state ownership to components and implementing proper ref-based data collection, creating autonomous components and simple orchestrating forms.
