# JsonViewer Component Documentation

## 📋 **Tổng quan**

`JsonViewerComponent` là component hiển thị và so sánh dữ liệu JSON dưới dạng tree view có thể expand/collapse. Component này hỗ trợ compare mode để hiển thị sự khác biệt giữa hai versions của data.

**Selector**: `app-json-viewer`  
**Location**: `src/app/components/json-viewer/json-viewer.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() item: any;                      // Current data object
@Input() oldItem: any;                   // Previous data for comparison
@Input() properties: any;                // Property name mappings
@Input() isCompare: boolean = true;      // Enable comparison mode
@Input() isShowDifference: boolean = true; // Show only differences
@Input() notShowProperties: string[];    // Properties to hide
```

### **Internal Properties**
```typescript
dataSource: any[] = [];                  // Flattened data structure
dataSourceState: any[] = [];             // Display state with tree info
isAllRowOpened: boolean = true;          // Expand/collapse all state
```

---

## 🚀 **Basic Usage**

### **Simple JSON Viewer**
```typescript
// Component
export class DataViewerPage extends PageBase {
    currentData: any = {
        Id: 12345,
        Name: 'John Doe',
        Email: 'john@example.com',
        Profile: {
            Age: 30,
            Department: 'IT',
            Skills: ['JavaScript', 'Angular', 'TypeScript']
        },
        Addresses: [
            { Type: 'Home', Street: '123 Main St', City: 'New York' },
            { Type: 'Work', Street: '456 Office Ave', City: 'New York' }
        ]
    };

    propertyMappings = {
        'Id': 'User ID',
        'Name': 'Full Name',
        'Email': 'Email Address',
        'Profile': 'User Profile',
        'Age': 'Age (years)',
        'Department': 'Department',
        'Skills': 'Technical Skills',
        'Addresses': 'Contact Addresses'
    };
}
```

```html
<!-- Template -->
<app-json-viewer
    [item]="currentData"
    [properties]="propertyMappings"
    [isCompare]="false">
</app-json-viewer>
```

### **Comparison Mode**
```typescript
export class DataComparisonPage extends PageBase {
    originalData: any = {
        Id: 12345,
        Name: 'John Doe',
        Email: 'john@example.com',
        Status: 'Active',
        Profile: {
            Age: 30,
            Department: 'IT'
        }
    };

    modifiedData: any = {
        Id: 12345,
        Name: 'John Smith',  // Changed
        Email: 'john.smith@example.com',  // Changed
        Status: 'Active',
        Profile: {
            Age: 31,  // Changed
            Department: 'IT',
            Position: 'Senior Developer'  // Added
        }
    };

    hiddenProperties = ['Id', 'CreatedDate', 'ModifiedDate'];
}
```

```html
<div class="comparison-container">
    <h3>Data Changes</h3>
    
    <app-json-viewer
        [item]="modifiedData"
        [oldItem]="originalData"
        [isCompare]="true"
        [isShowDifference]="true"
        [notShowProperties]="hiddenProperties"
        [properties]="propertyMappings">
    </app-json-viewer>
</div>
```

---

## 🔍 **Advanced Features**

### **Audit Trail Viewer**
```typescript
export class AuditTrailPage extends PageBase {
    auditEntries: any[] = [];
    selectedEntry: any = null;
    
    async loadAuditTrail(recordId: number) {
        try {
            const result = await this.auditService.getAuditTrail(recordId);
            this.auditEntries = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    viewChanges(entry: any) {
        this.selectedEntry = entry;
    }

    getPropertyDisplayName(key: string): string {
        const mappings = {
            'CustomerName': 'Customer Name',
            'OrderDate': 'Order Date',
            'TotalAmount': 'Total Amount',
            'OrderItems': 'Order Items',
            'ShippingAddress': 'Shipping Address'
        };
        return mappings[key] || key;
    }
}
```

```html
<div class="audit-trail-viewer">
    <!-- Audit entries list -->
    <div class="audit-list">
        <ion-list>
            <ion-item 
                *ngFor="let entry of auditEntries"
                button
                (click)="viewChanges(entry)"
                [class.selected]="selectedEntry?.Id === entry.Id">
                <ion-label>
                    <h3>{{ entry.Action }}</h3>
                    <p>{{ entry.ModifiedDate | date:'medium' }}</p>
                    <p>by {{ entry.ModifiedBy }}</p>
                </ion-label>
            </ion-item>
        </ion-list>
    </div>

    <!-- Changes viewer -->
    <div class="changes-viewer" *ngIf="selectedEntry">
        <h4>Changes Made</h4>
        <app-json-viewer
            [item]="selectedEntry.NewData"
            [oldItem]="selectedEntry.OldData"
            [isCompare]="true"
            [isShowDifference]="true">
        </app-json-viewer>
    </div>
</div>
```

### **Configuration Diff Viewer**
```typescript
export class ConfigDiffPage extends PageBase {
    currentConfig: any = {};
    defaultConfig: any = {};
    
    async loadConfigurations() {
        try {
            const [current, defaults] = await Promise.all([
                this.configService.getCurrentConfig(),
                this.configService.getDefaultConfig()
            ]);
            
            this.currentConfig = current;
            this.defaultConfig = defaults;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    resetToDefaults() {
        this.env.actionConfirm('reset', 1, 'configuration', 'RESET_CONFIG', 
            this.configService.resetToDefaults()
        ).then(() => {
            this.loadConfigurations();
            this.env.showMessage('CONFIG_RESET_SUCCESS', 'success');
        });
    }
}
```

```html
<div class="config-diff">
    <div class="diff-header">
        <h3>Configuration Changes</h3>
        <ion-button 
            fill="outline" 
            color="warning"
            (click)="resetToDefaults()">
            Reset to Defaults
        </ion-button>
    </div>

    <app-json-viewer
        [item]="currentConfig"
        [oldItem]="defaultConfig"
        [isCompare]="true"
        [isShowDifference]="true">
    </app-json-viewer>
</div>
```

---

## 🎨 **UI Customization**

### **Custom Styling**
```scss
// Component SCSS
app-json-viewer {
    .json-viewer-container {
        border: 1px solid var(--ion-color-medium);
        border-radius: 8px;
        overflow: hidden;
        
        .json-header {
            background: var(--ion-color-light);
            padding: 12px 16px;
            border-bottom: 1px solid var(--ion-color-medium);
            
            .expand-all-button {
                --color: var(--ion-color-primary);
            }
        }
        
        .json-content {
            max-height: 500px;
            overflow-y: auto;
            
            .json-row {
                display: flex;
                align-items: center;
                padding: 8px 16px;
                border-bottom: 1px solid var(--ion-color-light);
                
                &:hover {
                    background: var(--ion-color-light-tint);
                }
                
                &.has-changes {
                    background: rgba(var(--ion-color-warning-rgb), 0.1);
                    border-left: 3px solid var(--ion-color-warning);
                }
                
                &.added {
                    background: rgba(var(--ion-color-success-rgb), 0.1);
                    border-left: 3px solid var(--ion-color-success);
                }
                
                &.removed {
                    background: rgba(var(--ion-color-danger-rgb), 0.1);
                    border-left: 3px solid var(--ion-color-danger);
                }
                
                .indent {
                    width: 20px;
                    
                    &.level-1 { margin-left: 20px; }
                    &.level-2 { margin-left: 40px; }
                    &.level-3 { margin-left: 60px; }
                    &.level-4 { margin-left: 80px; }
                }
                
                .expand-button {
                    width: 24px;
                    height: 24px;
                    --color: var(--ion-color-medium);
                }
                
                .property-name {
                    font-weight: 500;
                    color: var(--ion-color-dark);
                    min-width: 150px;
                }
                
                .property-value {
                    flex: 1;
                    
                    .current-value {
                        color: var(--ion-color-dark);
                    }
                    
                    .old-value {
                        color: var(--ion-color-medium);
                        text-decoration: line-through;
                        margin-left: 8px;
                    }
                    
                    .value-type {
                        font-size: 0.75rem;
                        color: var(--ion-color-medium);
                        font-style: italic;
                    }
                }
            }
        }
    }
}
```

### **Diff Highlighting**
```html
<div class="json-row" [ngClass]="{
    'has-changes': hasChanges(row),
    'added': isAdded(row),
    'removed': isRemoved(row)
}">
    <div class="property-value">
        <!-- Current value -->
        <span class="current-value">{{ row.Value }}</span>
        
        <!-- Old value (if different) -->
        <span 
            *ngIf="row.OldValue !== undefined && row.Value !== row.OldValue"
            class="old-value">
            {{ row.OldValue }}
        </span>
        
        <!-- Change indicator -->
        <ion-icon 
            *ngIf="hasChanges(row)"
            name="arrow-forward"
            class="change-indicator">
        </ion-icon>
    </div>
</div>
```

---

## 🔄 **Data Processing**

### **Complex Object Handling**
```typescript
export class JsonDataProcessor {
    processComplexObject(data: any): any[] {
        const result = [];
        
        const processValue = (value: any, key: string, parentId?: string) => {
            const item = {
                Id: this.generateId(),
                Property: key,
                Value: null,
                OldValue: null,
                IDParent: parentId,
                Type: this.getValueType(value)
            };
            
            if (Array.isArray(value)) {
                item.Value = `Array (${value.length} items)`;
                result.push(item);
                
                value.forEach((arrayItem, index) => {
                    processValue(arrayItem, `[${index}]`, item.Id);
                });
            } else if (value && typeof value === 'object') {
                item.Value = 'Object';
                result.push(item);
                
                Object.keys(value).forEach(objKey => {
                    processValue(value[objKey], objKey, item.Id);
                });
            } else {
                item.Value = this.formatValue(value);
                result.push(item);
            }
        };
        
        Object.keys(data).forEach(key => {
            processValue(data[key], key);
        });
        
        return result;
    }
    
    getValueType(value: any): string {
        if (value === null) return 'null';
        if (Array.isArray(value)) return 'array';
        if (typeof value === 'object') return 'object';
        if (typeof value === 'string') return 'string';
        if (typeof value === 'number') return 'number';
        if (typeof value === 'boolean') return 'boolean';
        return 'unknown';
    }
    
    formatValue(value: any): string {
        if (value === null || value === undefined) return 'null';
        if (typeof value === 'string') return `"${value}"`;
        if (typeof value === 'boolean') return value ? 'true' : 'false';
        return String(value);
    }
}
```

### **String Parsing & Validation**
```typescript
export class JsonStringParser {
    safeParseJson(str: string): any {
        if (!str || typeof str !== 'string') {
            return str;
        }
        
        try {
            // Clean up common JSON formatting issues
            let cleanStr = str
                .replace(/([{,]\s*)([A-Za-z0-9_]+)\s*:/g, '$1"$2":') // Add quotes to keys
                .replace(/:\s*'([^']*)'/g, ': "$1"'); // Replace single quotes with double
            
            return JSON.parse(cleanStr);
        } catch (error) {
            // If parsing fails, return original string
            return str;
        }
    }
    
    isJsonString(str: string): boolean {
        try {
            JSON.parse(str);
            return true;
        } catch {
            return false;
        }
    }
    
    formatJsonString(obj: any): string {
        try {
            return JSON.stringify(obj, null, 2);
        } catch {
            return String(obj);
        }
    }
}
```

---

## 📱 **Mobile Optimization**

### **Responsive Layout**
```scss
// Mobile-specific styles
@media (max-width: 768px) {
    app-json-viewer {
        .json-viewer-container {
            .json-content {
                .json-row {
                    flex-direction: column;
                    align-items: flex-start;
                    padding: 12px 16px;
                    
                    .property-name {
                        min-width: auto;
                        margin-bottom: 4px;
                        font-size: 0.875rem;
                    }
                    
                    .property-value {
                        width: 100%;
                        font-size: 0.8125rem;
                        
                        .old-value {
                            display: block;
                            margin-left: 0;
                            margin-top: 4px;
                        }
                    }
                    
                    .indent {
                        &.level-1 { margin-left: 10px; }
                        &.level-2 { margin-left: 20px; }
                        &.level-3 { margin-left: 30px; }
                    }
                }
            }
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Đúng - Limit deep nesting
export class OptimizedJsonViewer {
    maxDepth = 10;
    maxArrayItems = 100;
    
    processData(data: any, depth = 0): any[] {
        if (depth > this.maxDepth) {
            return [{ 
                Property: '...', 
                Value: 'Maximum depth reached',
                Type: 'warning'
            }];
        }
        
        // Process with depth tracking
        return this.buildDataSource(data, depth);
    }
}
```

### **2. Memory Management**
```typescript
// ✅ Đúng - Clean up large objects
ngOnDestroy() {
    this.dataSource = [];
    this.dataSourceState = [];
}
```

### **3. Error Handling**
```typescript
// ✅ Đúng - Handle malformed data
buildDataSource() {
    try {
        if (!this.item) {
            this.dataSource = [];
            return;
        }
        
        // Process data safely
        this.processItem(this.item);
    } catch (error) {
        console.error('JSON viewer error:', error);
        this.dataSource = [{
            Property: 'Error',
            Value: 'Unable to display data',
            Type: 'error'
        }];
    }
}
```

---

## 🚨 **Common Use Cases**

### **1. API Response Viewer**
```typescript
export class ApiResponseViewer {
    apiResponse: any;
    
    async testApiCall() {
        try {
            this.apiResponse = await this.apiService.getData();
        } catch (error) {
            this.apiResponse = { error: error.message };
        }
    }
}
```

### **2. Form Data Inspector**
```typescript
export class FormDataInspector {
    formData: any;
    originalData: any;
    
    onFormChange() {
        this.formData = this.formGroup.getRawValue();
    }
    
    showChanges() {
        return JSON.stringify(this.formData) !== JSON.stringify(this.originalData);
    }
}
```

### **3. Configuration Manager**
```typescript
export class ConfigurationManager {
    currentConfig: any;
    defaultConfig: any;
    
    async loadConfig() {
        this.currentConfig = await this.configService.get();
        this.defaultConfig = await this.configService.getDefaults();
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
