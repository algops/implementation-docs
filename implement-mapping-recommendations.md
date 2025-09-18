# Mapping Recommendations Implementation

## Overview
Implement intelligent, data-driven recommendations for creating object and datapoint mappings by analyzing JSON structure, content patterns, and applying deterministic rules.

## Recommendation Systems

### 1. JSON Key to Object Type Mapping Recommendations

#### **High-Level Logic:**
The system analyzes the JSON structure to automatically suggest what types of objects can be extracted, based on two main approaches:

**Key-Based Detection:**
- Scan JSON keys for known object type patterns (e.g., "companies", "people", "documents")
- Look for organizational structures that suggest entity types
- Identify repeated patterns that indicate object collections

**Value-Based Detection:**
- Analyze actual data values for recognizable patterns (emails, URLs, IDs)
- Detect data types that strongly suggest object categories
- Find consistent value structures across different paths

**Output:** List of recommended object types with confidence scores and suggested extraction paths.

#### **Implementation Approach:**
```typescript
// Analyze JSON structure to recommend object types based on key patterns
const detectObjectTypesFromKeys = (jsonData: any) => {
  const recommendations = [];
  
  // Company detection patterns
  if (hasKeys(jsonData, ['companies', 'organizations', 'partners'])) {
    recommendations.push({
      objectType: 'company',
      confidence: 0.9,
      reasoning: 'Found company-related keys: companies, organizations, partners',
      suggestedPath: 'api_response.data.companies.*'
    });
  }
  
  // Person detection patterns
  if (hasKeys(jsonData, ['people', 'employees', 'users', 'contractors'])) {
    recommendations.push({
      objectType: 'person',
      confidence: 0.85,
      reasoning: 'Found person-related keys: people, employees, users',
      suggestedPath: 'api_response.data.people.**.email'
    });
  }
  
  // Document detection patterns
  if (hasKeys(jsonData, ['documents', 'files', 'reports'])) {
    recommendations.push({
      objectType: 'document',
      confidence: 0.8,
      reasoning: 'Found document-related keys: documents, files, reports',
      suggestedPath: 'api_response.data.documents.*.id'
    });
  }
  
  return recommendations;
};

// Analyze actual values to determine object types
const detectObjectTypesFromValues = (jsonData: any) => {
  const recommendations = [];
  
  // Email detection (person objects)
  const emailPatterns = findValuesByPattern(jsonData, /^[^\s@]+@[^\s@]+\.[^\s@]+$/);
  if (emailPatterns.length > 0) {
    recommendations.push({
      objectType: 'person',
      confidence: 0.95,
      reasoning: `Found ${emailPatterns.length} email addresses`,
      suggestedPath: 'api_response.data.**.email'
    });
  }
  
  // URL/Website detection (company objects)
  const urlPatterns = findValuesByPattern(jsonData, /^https?:\/\/.+/);
  if (urlPatterns.length > 0) {
    recommendations.push({
      objectType: 'company',
      confidence: 0.9,
      reasoning: `Found ${urlPatterns.length} website URLs`,
      suggestedPath: 'api_response.data.**.website'
    });
  }
  
  // ID detection (any entity)
  const idPatterns = findValuesByPattern(jsonData, /^(doc_|job_|entity_|company_)/);
  if (idPatterns.length > 0) {
    recommendations.push({
      objectType: 'entity',
      confidence: 0.8,
      reasoning: `Found ${idPatterns.length} structured IDs`,
      suggestedPath: 'api_response.data.**.id'
    });
  }
  
  return recommendations;
};
```

---

### 2. Deterministic Rules for Wildcard Recommendations

#### **High-Level Logic:**
The system applies rule-based logic to determine when and how to use wildcards, ensuring consistent and predictable recommendations:

**Array Wildcard (*) Rules:**
- **Array Traversal Rule:** If a path leads to an array, use `*` to traverse all elements
- **Consistent Structure Rule:** If multiple objects share the same property structure, use `*` for consistency
- **Index Replacement Rule:** Replace specific array indices with `*` for flexibility

**Recursive Wildcard (**) Rules:**
- **Multi-Level Search Rule:** If a property exists at multiple nesting depths, use `**` for comprehensive search
- **Deep Nesting Rule:** If dealing with deeply nested structures (3+ levels), use `**` for efficiency
- **Variable Depth Rule:** If property depth varies across different data branches, use `**` for flexibility

**Output:** Specific wildcard recommendations with reasoning and confidence levels.

#### **Implementation Approach:**
```typescript
const recommendArrayWildcards = (jsonData: any, path: string) => {
  const recommendations = [];
  
  // Rule 1: If path leads to array, use * for traversal
  if (isArrayAtPath(jsonData, path)) {
    recommendations.push({
      wildcard: '*',
      confidence: 0.95,
      reasoning: 'Path leads to array - use * for array traversal',
      example: `${path}.*.property`
    });
  }
  
  // Rule 2: If multiple objects with same structure, use * for consistency
  if (hasConsistentObjectStructure(jsonData, path)) {
    recommendations.push({
      wildcard: '*',
      confidence: 0.9,
      reasoning: 'Multiple objects with consistent structure detected',
      example: `${path}.*.commonProperty`
    });
  }
  
  // Rule 3: If path contains numeric indices, replace with *
  if (path.includes('[0]') || path.includes('[1]')) {
    recommendations.push({
      wildcard: '*',
      confidence: 0.85,
      reasoning: 'Replace specific array indices with wildcard for flexibility',
      example: path.replace(/\[\d+\]/g, '*')
    });
  }
  
  return recommendations;
};

const recommendRecursiveWildcards = (jsonData: any, path: string) => {
  const recommendations = [];
  
  // Rule 1: If searching for specific property across multiple nesting levels
  if (hasPropertyAtMultipleDepths(jsonData, path, 'email')) {
    recommendations.push({
      wildcard: '**',
      confidence: 0.9,
      reasoning: 'Property found at multiple nesting levels - use ** for recursive search',
      example: `${path}.**.email`
    });
  }
  
  // Rule 2: If dealing with deeply nested organizational structures
  if (hasDeepNesting(jsonData, path, 3)) {
    recommendations.push({
      wildcard: '**',
      confidence: 0.85,
      reasoning: 'Deep nesting detected - use ** for comprehensive search',
      example: `${path}.**.targetProperty`
    });
  }
  
  // Rule 3: If property could be at variable depths
  if (hasVariableDepthProperty(jsonData, path, 'id')) {
    recommendations.push({
      wildcard: '**',
      confidence: 0.8,
      reasoning: 'Property depth varies - use ** for flexible search',
      example: `${path}.**.id`
    });
  }
  
  return recommendations;
};
```

---

### 3. Datapoint Key + Value Recommendations

#### **High-Level Logic:**
The system analyzes the content within object contexts to suggest which specific data points should be extracted, considering both structure and content:

**Key-Based Detection:**
- Identify common properties for each object type (e.g., companies have "industry", "revenue")
- Map JSON keys to standard datapoint categories
- Consider object type context when suggesting properties

**Value-Based Detection:**
- Analyze actual data values to determine data types and categories
- Detect patterns in values that suggest specific datapoints
- Identify arrays, numeric fields, and text fields for appropriate extraction

**Output:** Recommended datapoints with extraction paths and value type classifications.

#### **Implementation Approach:**
```typescript
const recommendDatapointsFromKeys = (jsonData: any, objectContext: string) => {
  const recommendations = [];
  
  // Company datapoints
  if (objectContext.includes('company')) {
    const companyKeys = ['name', 'industry', 'employee_count', 'founded_year', 'revenue', 'funding'];
    companyKeys.forEach(key => {
      if (hasKeyInContext(jsonData, objectContext, key)) {
        recommendations.push({
          datapoint: key,
          confidence: 0.9,
          reasoning: `Company object typically has ${key} property`,
          suggestedPath: key
        });
      }
    });
  }
  
  // Person datapoints
  if (objectContext.includes('person')) {
    const personKeys = ['name', 'role', 'department', 'salary', 'email', 'phone'];
    personKeys.forEach(key => {
      if (hasKeyInContext(objectContext, key)) {
        recommendations.push({
          datapoint: key,
          confidence: 0.9,
          reasoning: `Person object typically has ${key} property`,
          suggestedPath: key
        });
      }
    });
  }
  
  return recommendations;
};

const recommendDatapointsFromValues = (jsonData: any, objectContext: string) => {
  const recommendations = [];
  
  // Analyze value types and patterns
  const contextData = getDataAtPath(jsonData, objectContext);
  
  // Numeric values (counts, amounts, years)
  const numericFields = findNumericFields(contextData);
  numericFields.forEach(field => {
    if (field.key === 'employee_count' || field.key === 'founded_year') {
      recommendations.push({
        datapoint: field.key,
        confidence: 0.95,
        reasoning: `Numeric field: ${field.key} = ${field.value}`,
        suggestedPath: field.key,
        valueType: 'numeric'
      });
    }
  });
  
  // Text values (names, descriptions, categories)
  const textFields = findTextFields(contextData);
  textFields.forEach(field => {
    if (field.key === 'industry' || field.key === 'status') {
      recommendations.push({
        datapoint: field.key,
        confidence: 0.9,
        reasoning: `Text field: ${field.key} = "${field.value}"`,
        suggestedPath: field.key,
        valueType: 'text'
      });
    }
  });
  
  // Array values (skills, technologies, tags)
  const arrayFields = findArrayFields(contextData);
  arrayFields.forEach(field => {
    if (field.key === 'technologies' || field.key === 'skills') {
      recommendations.push({
        datapoint: field.key,
        confidence: 0.85,
        reasoning: `Array field: ${field.key} with ${field.value.length} items`,
        suggestedPath: `${field.key}.0`,
        valueType: 'array'
      });
    }
  });
  
  return recommendations;
};
```

---

## Overall Recommendation Flow

### **Step-by-Step Process:**

1. **Analyze JSON Structure** → Detect potential object types
2. **Recommend Object Mappings** → Suggest extraction paths with wildcards
3. **Analyze Object Contexts** → Identify available datapoints
4. **Recommend Datapoint Mappings** → Suggest property extraction paths
5. **Validate Recommendations** → Ensure mappings follow system constraints

### **Smart Recommendation Engine:**
```typescript
const generateMappingRecommendations = (jsonData: any) => {
  // Step 1: Detect object types
  const objectRecommendations = [
    ...detectObjectTypesFromKeys(jsonData),
    ...detectObjectTypesFromValues(jsonData)
  ];
  
  // Step 2: For each object type, recommend wildcards
  const wildcardRecommendations = objectRecommendations.map(obj => ({
    ...obj,
    wildcards: [
      ...recommendArrayWildcards(jsonData, obj.suggestedPath),
      ...recommendRecursiveWildcards(jsonData, obj.suggestedPath)
    ]
  }));
  
  // Step 3: For each object, recommend datapoints
  const datapointRecommendations = wildcardRecommendations.map(obj => ({
    ...obj,
    datapoints: [
      ...recommendDatapointsFromKeys(jsonData, obj.suggestedPath),
      ...recommendDatapointsFromValues(jsonData, obj.suggestedPath)
    ]
  }));
  
  return datapointRecommendations;
};
```

---

## Key Benefits

1. **Deterministic:** Rules are based on clear patterns and data analysis
2. **Context-Aware:** Recommendations consider the specific JSON structure
3. **Confidence Scoring:** Each recommendation has a confidence level
4. **Explainable:** Clear reasoning for each recommendation
5. **Flexible:** Handles various JSON structures and patterns
6. **Validated:** Recommendations can be tested against actual data

## Implementation Notes

- **Performance:** Use efficient JSON traversal algorithms for large datasets
- **Caching:** Cache analysis results for repeated recommendations
- **Validation:** Check that all recommendations follow mapping system constraints
- **Testing:** Test with various JSON structures and edge cases
- **User Experience:** Present recommendations with clear explanations and confidence levels

This system provides intelligent, data-driven suggestions for creating mappings while maintaining the deterministic nature required for reliable automation.
