# Catalog Implementation Plan

## Component Usage Validation
**✅ COMPONENT SOURCE CHECK:**
- [ ] Uses only components from `components/` folder
- [ ] Uses only components from `components/ui-elements/` subfolder
- [ ] **NO direct usage of `components/ui/` folder components**

## Overview
Implement a comprehensive catalog search system that provides global search functionality across activity types, factors, sources, activities, and workflows. The search component will be integrated into the header and accessible throughout the application, featuring a Command-based interface with configurable open/closed states.

## Phase 1: Architecture & Core Structure

### 1.1 Catalog Search Architecture
- **Search Component**: Command-based search interface with keyboard shortcuts
- **Search Provider**: Global search state management and data fetching
- **Catalog Database**: Unified catalog table with type enumeration
- **Search Engine**: Fuzzy search and filtering across all catalog types

### 1.2 Core Components Structure
- **CatalogSearch**: Main search component with Command interface
- **CatalogSearchTrigger**: Header button/trigger for opening search
- **CatalogSearchResults**: Results display with grouped categories
- **CatalogSearchItem**: Individual search result item component

### 1.3 Data Flow Design
- **Search Query**: User input and search state management
- **Catalog Data**: Fetched from unified catalog table
- **Search Results**: Filtered and grouped results by type
- **Selection Events**: Item selection and navigation handling

## Phase 2: Database Schema & Data Model

### 2.1 Catalog Table Enhancement
- **Type Enumeration**: Extend `catalog_type_enum` to include all searchable types
- **Owner Information**: Add owner/user information for each catalog item
- **Search Indexing**: Optimize table for search performance
- **Metadata Structure**: Standardize metadata for consistent search

### 2.2 Catalog Type Enum
```sql
CREATE TYPE "public"."catalog_type_enum" AS ENUM (
    'activity_type',
    'factor',
    'source',
    'activity',
    'workflow'
);
```

### 2.3 Catalog Table Structure
- **id**: UUID primary key
- **name**: Display name for search results
- **description**: Optional description for context
- **type**: Catalog type from enum
- **owner_id**: User/organization who owns the item
- **metadata**: JSONB for type-specific data
- **tags**: Array of searchable tags
- **created_at/updated_at**: Timestamp tracking

## Phase 3: Components & Features

### 3.1 CatalogSearch Component
- **Command Interface**: Built on existing Command component
- **Search Input**: Real-time search with debouncing
- **Keyboard Shortcuts**: Cmd/Ctrl+K to open search
- **Focus Management**: Proper focus handling and escape key support

### 3.2 CatalogSearchTrigger Component
- **Header Integration**: Button in header for search access
- **Keyboard Shortcut**: Visual indicator for Cmd+K shortcut
- **State Management**: Open/close search dialog
- **Accessibility**: ARIA labels and keyboard navigation

### 3.3 CatalogSearchResults Component
- **Grouped Results**: Results organized by catalog type
- **Empty States**: No results and loading states
- **Recent Items**: Show recently accessed catalog items
- **Quick Actions**: Context actions for each result type

### 3.4 CatalogSearchItem Component
- **Item Display**: Name, description, and type indicator
- **Type Icons**: Visual indicators for different catalog types
- **Owner Information**: Display owner/creator information
- **Selection Handling**: Click and keyboard selection

## Phase 4: Search Functionality & Integration

### 4.1 Search Engine
- **Fuzzy Search**: Implement fuzzy matching for better results
- **Type Filtering**: Filter results by catalog type
- **Tag Search**: Search within item tags
- **Metadata Search**: Search within JSONB metadata fields

### 4.2 Search Provider
- **Global State**: Search state management across application
- **Data Fetching**: API calls to fetch catalog data
- **Caching**: Cache search results for performance
- **Debouncing**: Optimize search input performance

### 4.3 Header Integration
- **Search Button**: Add search trigger to header component
- **Keyboard Shortcuts**: Global keyboard shortcut handling
- **Responsive Design**: Mobile-friendly search interface
- **Theme Integration**: Consistent with design system

## Phase 5: Advanced Features & User Experience

### 5.1 Search Intelligence
- **Recent Items**: Show recently accessed items
- **Popular Items**: Display frequently accessed items
- **Smart Suggestions**: Context-aware search suggestions
- **Search History**: Track and display search history

### 5.2 Navigation Integration
- **Direct Navigation**: Navigate to specific pages from results
- **Context Actions**: Type-specific actions for each result
- **Breadcrumb Updates**: Update breadcrumbs when navigating
- **URL Integration**: Sync search state with URL parameters

### 5.3 Performance Optimization
- **Lazy Loading**: Load search results on demand
- **Virtual Scrolling**: Handle large result sets efficiently
- **Search Debouncing**: Optimize search input performance
- **Result Caching**: Cache frequently accessed results

## Phase 6: Responsive Design & Accessibility

### 6.1 Responsive Layout
- **Mobile-First**: Touch-friendly search interface
- **Adaptive Results**: Responsive result list layout
- **Touch Targets**: Adequate size for mobile interaction
- **Orientation Support**: Portrait and landscape layouts

### 6.2 Accessibility Features
- **ARIA Labels**: Screen reader support for all elements
- **Keyboard Navigation**: Full keyboard accessibility
- **Focus Management**: Clear focus indicators and flow
- **High Contrast**: Support for accessibility themes

### 6.3 Performance Optimization
- **Efficient Rendering**: Optimize search result rendering
- **Memory Management**: Clean up search resources
- **Smooth Animations**: 60fps search transitions
- **Loading States**: Clear loading and error states

## Phase 7: Testing & Validation

### 7.1 Unit Tests
- **Component Rendering**: Search component rendering tests
- **Search Logic**: Search algorithm and filtering tests
- **State Management**: Search state and provider tests
- **Event Handling**: Search event and interaction tests

### 7.2 Integration Tests
- **Header Integration**: Search integration with header
- **Navigation Flow**: Complete search to navigation workflow
- **Data Fetching**: API integration and data loading tests
- **Keyboard Shortcuts**: Global shortcut functionality tests

### 7.3 User Experience Tests
- **Search Flow**: Complete search user journey
- **Mobile Responsiveness**: Mobile search experience
- **Accessibility Compliance**: Screen reader and keyboard testing
- **Performance Testing**: Search performance with large datasets

## File Changes Required

### New Files
- `components/search/catalog-search.tsx`
- `components/search/catalog-search-trigger.tsx`
- `components/search/catalog-search-results.tsx`
- `components/search/catalog-search-item.tsx`
- `components/search/catalog-search-provider.tsx`
- `hooks/use-catalog-search.ts`
- `hooks/use-keyboard-shortcuts.ts`
- `utils/search-utils.ts`
- `types/catalog.ts`

### Modified Files
- `components/layout/header.tsx` - Add search trigger
- `components/search/index.ts` - Export search components
- Database migration - Update catalog_type_enum and catalog table

### Database Changes
- Update `catalog_type_enum` to include all searchable types
- Add `owner_id` column to catalog table
- Add search indexes for performance
- Create catalog data seeding script

## Testing Checklist

### Unit Tests
- [ ] Search component rendering
- [ ] Search logic and filtering
- [ ] Keyboard shortcut handling
- [ ] State management

### Integration Tests
- [ ] Header integration
- [ ] Search to navigation flow
- [ ] Data fetching and caching
- [ ] Global keyboard shortcuts

### User Experience Tests
- [ ] Search interaction flow
- [ ] Mobile responsiveness
- [ ] Accessibility compliance
- [ ] Performance with large datasets

## Success Criteria

1. **Functionality**: Fast, accurate search across all catalog types
2. **Performance**: Sub-200ms search response times
3. **Usability**: Intuitive search requiring minimal training
4. **Accessibility**: Full WCAG 2.1 AA compliance
5. **Responsiveness**: Seamless experience across all device sizes
6. **Integration**: Seamless integration with existing header and navigation

## Implementation Notes

### Search Algorithm
- Implement fuzzy search using Fuse.js or similar library
- Support partial matching and typo tolerance
- Weight results by relevance and recency
- Group results by catalog type for better organization

### Keyboard Shortcuts
- Cmd/Ctrl+K: Open search dialog
- Escape: Close search dialog
- Arrow keys: Navigate results
- Enter: Select highlighted result
- Tab: Cycle through result groups

### Data Structure
- Standardize catalog item structure across all types
- Include searchable metadata for each type
- Implement consistent owner/creator information
- Support tagging system for better categorization

### Performance Considerations
- Implement search result pagination for large datasets
- Use virtual scrolling for result lists
- Cache search results and metadata
- Optimize database queries with proper indexing
