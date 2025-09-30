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


