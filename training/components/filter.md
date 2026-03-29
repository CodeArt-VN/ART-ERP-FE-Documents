# Filter Component Documentation

## 📋 **Tổng quan**

`FilterComponent` là advanced filter builder cho phép tạo complex queries với logical operators (AND/OR), comparison operators và nested conditions. Component này hỗ trợ drag-drop để sắp xếp conditions và dynamic form building.

**Selector**: `app-filter`  
**Location**: `src/app/components/filter/filter.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() item: FilterConfig;             // Filter configuration
@Input() schema: Schema;                 // Field schema for filtering
@Input() smallWidth: boolean = false;    // Compact layout mode
```

### **Output Events**
```typescript
@Output() change = new EventEmitter();   // Filter changes
@Output() submit = new EventEmitter();   // Filter submission
@Output() message = new EventEmitter();  // Status messages
```

### **FilterConfig Interface**
```typescript
interface FilterConfig {
    Dimension: string;                   // Field name or 'logical'
    Operator: string;                    // Comparison or logical operator
    Value?: any;                         // Filter value
    Logicals?: FilterConfig[];           // Nested conditions
}
```

### **Schema Interface**
```typescript
interface Schema {
    Fields: SchemaField[];
}

interface SchemaField {
    Code: string;                        // Field code
    Name: string;                        // Display name
    DataType?: string;                   // Data type (text, number, boolean)
}
```

---

## 🚀 **Basic Usage**

### **Simple Filter**
```typescript
// Component
export class ProductFilterPage {
    filterConfig: FilterConfig = {
        Dimension: 'logical',
        Operator: 'AND',
        Logicals: []
    };

    schema: Schema = {
        Fields: [
            { Code: 'name', Name: 'Product Name', DataType: 'text' },
            { Code: 'price', Name: 'Price', DataType: 'number' },
            { Code: 'isActive', Name: 'Active', DataType: 'boolean' },
            { Code: 'categoryId', Name: 'Category', DataType: 'number' }
        ]
    };

    onFilterChange(filter: FilterConfig) {
        console.log('Filter changed:', filter);
        this.applyFilter(filter);
    }

    onFilterSubmit(filter: FilterConfig) {
        console.log('Filter submitted:', filter);
        this.searchWithFilter(filter);
    }

    onFilterMessage(message: any) {
        console.log('Filter message:', message);
        if (message.logLevel === 'warning') {
            this.env.showMessage(message.message, 'warning');
        }
    }

    async applyFilter(filter: FilterConfig) {
        try {
            const query = this.buildQueryFromFilter(filter);
            const result = await this.productService.read(query);
            this.items = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

```html
<!-- Template -->
<app-filter
    [item]="filterConfig"
    [schema]="schema"
    [smallWidth]="false"
    (change)="onFilterChange($event)"
    (submit)="onFilterSubmit($event)"
    (message)="onFilterMessage($event)">
</app-filter>

<!-- Results -->
<app-data-table
    [rows]="items"
    [columns]="columns">
</app-data-table>
```

---

## 🔍 **Advanced Filtering**

### **Complex Filter Configuration**
```typescript
export class AdvancedFilterPage {
    // Pre-configured complex filter
    advancedFilter: FilterConfig = {
        Dimension: 'logical',
        Operator: 'AND',
        Logicals: [
            {
                Dimension: 'name',
                Operator: 'like',
                Value: 'Product'
            },
            {
                Dimension: 'logical',
                Operator: 'OR',
                Logicals: [
                    {
                        Dimension: 'price',
                        Operator: '>',
                        Value: 100
                    },
                    {
                        Dimension: 'categoryId',
                        Operator: 'IN',
                        Value: [1, 2, 3]
                    }
                ]
            }
        ]
    };

    // Extended schema with more field types
    extendedSchema: Schema = {
        Fields: [
            { Code: 'name', Name: 'Product Name', DataType: 'text' },
            { Code: 'code', Name: 'Product Code', DataType: 'text' },
            { Code: 'price', Name: 'Price', DataType: 'number' },
            { Code: 'cost', Name: 'Cost', DataType: 'number' },
            { Code: 'quantity', Name: 'Quantity', DataType: 'number' },
            { Code: 'isActive', Name: 'Active', DataType: 'boolean' },
            { Code: 'isFeatured', Name: 'Featured', DataType: 'boolean' },
            { Code: 'categoryId', Name: 'Category', DataType: 'number' },
            { Code: 'supplierId', Name: 'Supplier', DataType: 'number' },
            { Code: 'createdDate', Name: 'Created Date', DataType: 'date' },
            { Code: 'updatedDate', Name: 'Updated Date', DataType: 'date' }
        ]
    };
}
```

### **Dynamic Schema Loading**
```typescript
export class DynamicSchemaFilterPage {
    schema: Schema;
    filterConfig: FilterConfig;

    async ngOnInit() {
        await this.loadSchema();
        this.initializeFilter();
    }

    async loadSchema() {
        try {
            // Load schema from API
            const schemaData = await this.schemaService.getFilterSchema('product');
            this.schema = {
                Fields: schemaData.map(field => ({
                    Code: field.code,
                    Name: field.displayName,
                    DataType: field.dataType
                }))
            };
        } catch (error) {
            this.env.showErrorMessage(error);
            // Fallback to default schema
            this.schema = { Fields: [] };
        }
    }

    initializeFilter() {
        this.filterConfig = {
            Dimension: 'logical',
            Operator: 'AND',
            Logicals: []
        };
    }
}
```

---

## 🛠️ **Filter Operations**

### **Available Operators**

#### **Text Operators**
```typescript
const textOperators = [
    { code: '=', name: '= is' },
    { code: 'like', name: 'contains' },
    { code: '<>', name: '≠ does not equal' },
    { code: 'IN', name: 'in' },
    { code: 'NOT IN', name: 'not in' }
];
```

#### **Number Operators**
```typescript
const numberOperators = [
    { code: '=', name: '= equals' },
    { code: '>', name: '> greater than' },
    { code: '<', name: '< less than' },
    { code: '>=', name: '≥ greater than or equals' },
    { code: '<=', name: '≤ less than or equals' },
    { code: '<>', name: '≠ does not equal' }
];
```

#### **Boolean Operators**
```typescript
const booleanOperators = [
    { code: '1', name: 'true' },
    { code: '0', name: 'false' }
];
```

#### **Logical Operators**
```typescript
const logicalOperators = [
    { code: 'AND', name: 'AND' },
    { code: 'OR', name: 'OR' }
];
```

---

## 🔄 **Filter Building**

### **Programmatic Filter Creation**
```typescript
export class FilterBuilderPage {
    buildTextFilter(field: string, operator: string, value: string): FilterConfig {
        return {
            Dimension: field,
            Operator: operator,
            Value: value
        };
    }

    buildNumberFilter(field: string, operator: string, value: number): FilterConfig {
        return {
            Dimension: field,
            Operator: operator,
            Value: value
        };
    }

    buildRangeFilter(field: string, min: number, max: number): FilterConfig {
        return {
            Dimension: 'logical',
            Operator: 'AND',
            Logicals: [
                {
                    Dimension: field,
                    Operator: '>=',
                    Value: min
                },
                {
                    Dimension: field,
                    Operator: '<=',
                    Value: max
                }
            ]
        };
    }

    buildInFilter(field: string, values: any[]): FilterConfig {
        return {
            Dimension: field,
            Operator: 'IN',
            Value: values
        };
    }

    // Build complex filter
    buildComplexFilter(): FilterConfig {
        return {
            Dimension: 'logical',
            Operator: 'AND',
            Logicals: [
                this.buildTextFilter('name', 'like', 'Product'),
                {
                    Dimension: 'logical',
                    Operator: 'OR',
                    Logicals: [
                        this.buildNumberFilter('price', '>', 100),
                        this.buildInFilter('categoryId', [1, 2, 3])
                    ]
                }
            ]
        };
    }
}
```

### **Filter Validation**
```typescript
export class FilterValidationPage {
    validateFilter(filter: FilterConfig): boolean {
        if (!filter) return false;
        
        if (filter.Dimension === 'logical') {
            // Validate logical filter
            if (!filter.Operator || !['AND', 'OR'].includes(filter.Operator)) {
                return false;
            }
            
            if (!filter.Logicals || filter.Logicals.length === 0) {
                return false;
            }
            
            // Recursively validate nested filters
            return filter.Logicals.every(logical => this.validateFilter(logical));
        } else {
            // Validate field filter
            if (!filter.Dimension || !filter.Operator) {
                return false;
            }
            
            // Value is required for most operators
            if (!['1', '0'].includes(filter.Operator) && 
                (filter.Value === null || filter.Value === undefined || filter.Value === '')) {
                return false;
            }
            
            return true;
        }
    }

    getFilterErrors(filter: FilterConfig): string[] {
        const errors: string[] = [];
        
        if (!this.validateFilter(filter)) {
            errors.push('Invalid filter configuration');
        }
        
        // Add specific validation errors
        this.collectFilterErrors(filter, errors);
        
        return errors;
    }

    private collectFilterErrors(filter: FilterConfig, errors: string[]): void {
        if (filter.Dimension === 'logical') {
            if (filter.Logicals) {
                filter.Logicals.forEach(logical => {
                    this.collectFilterErrors(logical, errors);
                });
            }
        } else {
            // Validate specific field types
            const field = this.schema.Fields.find(f => f.Code === filter.Dimension);
            if (field) {
                if (field.DataType === 'number' && isNaN(Number(filter.Value))) {
                    errors.push(`${field.Name} must be a number`);
                }
            }
        }
    }
}
```

---

## 🔄 **Query Conversion**

### **Convert Filter to Query**
```typescript
export class FilterQueryConverter {
    convertFilterToQuery(filter: FilterConfig): any {
        if (!filter || !this.validateFilter(filter)) {
            return {};
        }

        return this.processFilter(filter);
    }

    private processFilter(filter: FilterConfig): any {
        if (filter.Dimension === 'logical') {
            const conditions = filter.Logicals.map(logical => 
                this.processFilter(logical)
            );
            
            if (filter.Operator === 'AND') {
                return { $and: conditions };
            } else {
                return { $or: conditions };
            }
        } else {
            return this.buildFieldCondition(filter);
        }
    }

    private buildFieldCondition(filter: FilterConfig): any {
        const field = filter.Dimension;
        const operator = filter.Operator;
        const value = filter.Value;

        switch (operator) {
            case '=':
                return { [field]: value };
            case '<>':
                return { [field]: { $ne: value } };
            case '>':
                return { [field]: { $gt: value } };
            case '<':
                return { [field]: { $lt: value } };
            case '>=':
                return { [field]: { $gte: value } };
            case '<=':
                return { [field]: { $lte: value } };
            case 'like':
                return { [field]: { $regex: value, $options: 'i' } };
            case 'IN':
                return { [field]: { $in: Array.isArray(value) ? value : [value] } };
            case 'NOT IN':
                return { [field]: { $nin: Array.isArray(value) ? value : [value] } };
            case '1':
                return { [field]: true };
            case '0':
                return { [field]: false };
            default:
                return { [field]: value };
        }
    }

    // Convert to SQL WHERE clause
    convertFilterToSQL(filter: FilterConfig): string {
        if (!filter || !this.validateFilter(filter)) {
            return '';
        }

        return this.processSQLFilter(filter);
    }

    private processSQLFilter(filter: FilterConfig): string {
        if (filter.Dimension === 'logical') {
            const conditions = filter.Logicals
                .map(logical => this.processSQLFilter(logical))
                .filter(condition => condition.length > 0);
            
            if (conditions.length === 0) return '';
            if (conditions.length === 1) return conditions[0];
            
            const operator = filter.Operator === 'AND' ? ' AND ' : ' OR ';
            return `(${conditions.join(operator)})`;
        } else {
            return this.buildSQLCondition(filter);
        }
    }

    private buildSQLCondition(filter: FilterConfig): string {
        const field = filter.Dimension;
        const operator = filter.Operator;
        const value = filter.Value;

        switch (operator) {
            case '=':
                return `${field} = '${value}'`;
            case '<>':
                return `${field} <> '${value}'`;
            case '>':
                return `${field} > ${value}`;
            case '<':
                return `${field} < ${value}`;
            case '>=':
                return `${field} >= ${value}`;
            case '<=':
                return `${field} <= ${value}`;
            case 'like':
                return `${field} LIKE '%${value}%'`;
            case 'IN':
                const inValues = Array.isArray(value) ? value : [value];
                return `${field} IN (${inValues.map(v => `'${v}'`).join(', ')})`;
            case 'NOT IN':
                const notInValues = Array.isArray(value) ? value : [value];
                return `${field} NOT IN (${notInValues.map(v => `'${v}'`).join(', ')})`;
            case '1':
                return `${field} = 1`;
            case '0':
                return `${field} = 0`;
            default:
                return `${field} = '${value}'`;
        }
    }
}
```

---

## 💾 **Filter Persistence**

### **Save/Load Filters**
```typescript
export class FilterPersistencePage {
    savedFilters: { [key: string]: FilterConfig } = {};

    async saveFilter(name: string, filter: FilterConfig) {
        try {
            this.savedFilters[name] = lib.cloneObject(filter);
            await this.env.setStorage('savedFilters', this.savedFilters);
            this.env.showMessage('FILTER_SAVED', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async loadFilter(name: string): Promise<FilterConfig | null> {
        try {
            const savedFilters = await this.env.getStorage('savedFilters') || {};
            return savedFilters[name] || null;
        } catch (error) {
            this.env.showErrorMessage(error);
            return null;
        }
    }

    async loadAllSavedFilters() {
        try {
            this.savedFilters = await this.env.getStorage('savedFilters') || {};
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async deleteFilter(name: string) {
        try {
            delete this.savedFilters[name];
            await this.env.setStorage('savedFilters', this.savedFilters);
            this.env.showMessage('FILTER_DELETED', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

---

## 🎨 **UI Customization**

### **Custom Styling**
```scss
// Component SCSS
app-filter {
    .filter-container {
        border: 1px solid var(--ion-color-medium);
        border-radius: 8px;
        padding: 16px;
        background: var(--ion-color-light);
        
        &.small-width {
            max-width: 400px;
        }
        
        .filter-group {
            margin-bottom: 12px;
            padding: 12px;
            border: 1px dashed var(--ion-color-medium);
            border-radius: 6px;
            
            &.logical-group {
                background: rgba(var(--ion-color-primary-rgb), 0.05);
            }
            
            .filter-condition {
                display: flex;
                align-items: center;
                gap: 8px;
                margin-bottom: 8px;
                
                .field-select {
                    flex: 1;
                    min-width: 120px;
                }
                
                .operator-select {
                    flex: 1;
                    min-width: 100px;
                }
                
                .value-input {
                    flex: 2;
                    min-width: 150px;
                }
                
                .remove-button {
                    --color: var(--ion-color-danger);
                }
            }
        }
        
        .add-condition-button {
            --background: var(--ion-color-success);
            --color: var(--ion-color-success-contrast);
            margin-top: 8px;
        }
        
        .filter-actions {
            display: flex;
            justify-content: space-between;
            margin-top: 16px;
            
            .action-button {
                --border-radius: 6px;
            }
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Đúng - Debounce filter changes
onFilterChange = debounceTime(300)((filter: FilterConfig) => {
    this.applyFilter(filter);
});

// ✅ Đúng - Validate before processing
processFilter(filter: FilterConfig) {
    if (!this.validateFilter(filter)) {
        this.env.showMessage('INVALID_FILTER', 'warning');
        return;
    }
    
    this.applyFilter(filter);
}
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Comprehensive error handling
async applyFilter(filter: FilterConfig) {
    try {
        this.loading = true;
        const query = this.convertFilterToQuery(filter);
        const result = await this.service.read(query);
        this.items = result.data;
    } catch (error) {
        this.env.showErrorMessage(error);
        // Reset to previous working filter
        this.filterConfig = this.lastValidFilter;
    } finally {
        this.loading = false;
    }
}
```

### **3. User Experience**
```typescript
// ✅ Đúng - Provide filter presets
filterPresets = {
    'Active Products': {
        Dimension: 'isActive',
        Operator: '1',
        Value: true
    },
    'High Value Items': {
        Dimension: 'price',
        Operator: '>',
        Value: 1000
    },
    'Recent Items': {
        Dimension: 'createdDate',
        Operator: '>=',
        Value: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) // 30 days ago
    }
};
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
