# FieldControl Component Documentation

## 📋 **Tổng quan**

`FieldControlComponent` là component chuyên dụng để render các field controls với configuration data, hỗ trợ nhiều loại input khác nhau như select-staff, select-contact, select-branch, string, number, textarea, toggle. Component này thường được sử dụng trong configuration forms.

**Selector**: `app-field-control`  
**Location**: `src/app/components/controls/field-control.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() field: any;                     // Field configuration object
```

### **Output Events**
```typescript
@Output() onChange = new EventEmitter(); // Field value change events
```

---

## 🚀 **Basic Usage**

### **Configuration Form Field**
```typescript
// Component
export class ConfigurationPage extends PageBase {
    configFields = [
        {
            type: 'string',
            label: 'Company Name',
            labelForId: 'companyName',
            data: { ValueObject: 'ART Company' },
            disabled: false,
            remark: 'Enter your company name'
        },
        {
            type: 'select-staff',
            label: 'Default Manager',
            labelForId: 'defaultManager',
            data: { ValueObject: null },
            DataSource: this.staffService.getStaff(),
            SearchInput: new Subject<string>(),
            SearchLoading: false,
            multiple: false,
            bindLabel: 'FullName',
            bindValue: 'Id',
            imgPath: '/assets/staff-photos/',
            disabled: false
        },
        {
            type: 'toggle',
            label: 'Enable Notifications',
            labelForId: 'enableNotifications',
            data: { ValueObject: true },
            disabled: false,
            remark: 'Turn on/off system notifications'
        }
    ];

    onFieldChange(data: any) {
        console.log('Field changed:', data);
        this.saveConfiguration(data);
    }
}
```

```html
<!-- Template -->
<div class="configuration-form">
    <app-field-control
        *ngFor="let field of configFields"
        [field]="field"
        (onChange)="onFieldChange($event)">
    </app-field-control>
</div>
```

---

## 🎛️ **Field Types**

### **1. Staff Selection (select-staff)**
```typescript
const staffField = {
    type: 'select-staff',
    label: 'Assign Staff',
    labelForId: 'assignedStaff',
    data: { ValueObject: null },
    DataSource: this.staffService.getStaff(),
    SearchInput: new Subject<string>(),
    SearchLoading: false,
    multiple: true,                      // Allow multiple selection
    bindLabel: 'FullName',
    bindValue: 'Id',
    imgPath: '/assets/staff-photos/',
    disabled: false
};
```

### **2. Contact Selection (select-contact)**
```typescript
const contactField = {
    type: 'select-contact',
    label: 'Primary Contact',
    labelForId: 'primaryContact',
    data: { ValueObject: null },
    DataSource: this.contactService.getContacts(),
    SearchInput: new Subject<string>(),
    SearchLoading: false,
    multiple: false,
    bindLabel: 'Name',
    bindValue: 'Id',
    disabled: false
};
```

### **3. Branch Selection (select-branch)**
```typescript
const branchField = {
    type: 'select-branch',
    label: 'Target Branch',
    labelForId: 'targetBranch',
    data: { ValueObject: null },
    items: this.branchList,
    bindLabel: 'Name',
    bindValue: 'Id',
    disabled: false
};
```

### **4. Generic Select (select)**
```typescript
const selectField = {
    type: 'select',
    label: 'Status',
    labelForId: 'status',
    data: { ValueObject: null },
    items: [
        { Name: 'Active', Value: 'active', Color: 'success' },
        { Name: 'Inactive', Value: 'inactive', Color: 'medium' },
        { Name: 'Pending', Value: 'pending', Color: 'warning' }
    ],
    bindLabel: 'Name',
    bindValue: 'Value',
    disabled: false
};
```

### **5. Text Input (string)**
```typescript
const stringField = {
    type: 'string',
    label: 'Description',
    labelForId: 'description',
    data: { ValueObject: '' },
    disabled: false,
    remark: 'Enter description here'
};
```

### **6. Number Input (number)**
```typescript
const numberField = {
    type: 'number',
    label: 'Max Items',
    labelForId: 'maxItems',
    data: { ValueObject: 100 },
    disabled: false,
    remark: 'Maximum number of items allowed'
};
```

### **7. Textarea (textarea)**
```typescript
const textareaField = {
    type: 'textarea',
    label: 'Notes',
    labelForId: 'notes',
    data: { ValueObject: '' },
    disabled: false,
    remark: 'Additional notes or comments'
};
```

### **8. Toggle Switch (toggle)**
```typescript
const toggleField = {
    type: 'toggle',
    label: 'Is Active',
    labelForId: 'isActive',
    data: { ValueObject: true },
    disabled: false,
    remark: 'Enable or disable this feature'
};
```

---

## 🔄 **Advanced Features**

### **Inherited Configuration**
```typescript
const inheritedField = {
    type: 'string',
    label: 'Company Policy',
    labelForId: 'companyPolicy',
    data: { 
        ValueObject: 'Local Policy',
        _InheritedConfig: {
            IDBranch: 1,
            _Branches: [
                { Id: 1, Name: 'Head Office' },
                { Id: 2, Name: 'Regional Office' }
            ]
        }
    },
    showInherited: false,
    disabled: false
};
```

### **Dynamic Search for Staff/Contact**
```typescript
export class ConfigPage extends PageBase {
    setupStaffSearch(field: any) {
        field.SearchInput.pipe(
            debounceTime(300),
            distinctUntilChanged(),
            switchMap(term => {
                field.SearchLoading = true;
                return this.staffService.search(term);
            })
        ).subscribe(results => {
            field.SearchLoading = false;
            field.DataSource = of(results);
        });
    }
}
```

---

## 🎨 **Styling**

### **Custom Field Styling**
```scss
// Component SCSS
app-field-control {
    .c-control {
        margin-bottom: 16px;
        
        .c-label {
            font-weight: 500;
            color: var(--ion-color-dark);
            margin-bottom: 8px;
            display: block;
            
            .clickable {
                font-size: 0.875rem;
                margin-left: 8px;
                text-decoration: none;
                
                &.danger {
                    color: var(--ion-color-danger);
                }
                
                &.success {
                    color: var(--ion-color-success);
                }
            }
        }
        
        .c-input {
            width: 100%;
            padding: 8px 12px;
            border: 1px solid var(--ion-color-medium);
            border-radius: 4px;
            
            &:focus {
                border-color: var(--ion-color-primary);
                outline: none;
            }
            
            &.disable {
                background-color: var(--ion-color-light);
                color: var(--ion-color-medium);
            }
        }
        
        // Staff/Contact selection styling
        ng-select {
            .ng-option {
                padding: 12px;
                
                ion-avatar {
                    width: 32px;
                    height: 32px;
                    margin-right: 12px;
                    display: inline-block;
                    vertical-align: middle;
                }
                
                .important {
                    color: var(--ion-color-primary);
                    font-weight: 500;
                }
            }
            
            .ng-value {
                ion-chip {
                    margin: 2px;
                    
                    ion-avatar {
                        width: 24px;
                        height: 24px;
                    }
                }
            }
        }
        
        // Branch selection styling
        .ng-option {
            ion-text {
                font-weight: 500;
            }
        }
        
        // Inherited config styling
        .breadcrumbs {
            margin-top: 8px;
            padding: 8px;
            background-color: var(--ion-color-light-tint);
            border-radius: 4px;
            opacity: 0.8;
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Field Configuration Structure**
```typescript
// ✅ Đúng - Complete field configuration
interface FieldConfig {
    type: 'select-staff' | 'select-contact' | 'select-branch' | 'select' | 'string' | 'number' | 'textarea' | 'toggle';
    label: string;
    labelForId: string;
    data: {
        ValueObject: any;
        _InheritedConfig?: any;
    };
    disabled?: boolean;
    remark?: string;
    // Type-specific properties
    DataSource?: Observable<any[]>;
    items?: any[];
    multiple?: boolean;
    bindLabel?: string;
    bindValue?: string;
    SearchInput?: Subject<string>;
    SearchLoading?: boolean;
    imgPath?: string;
    showInherited?: boolean;
}
```

### **2. Reset Functionality**
```typescript
// ✅ Đúng - Handle reset properly
resetField(field: any) {
    field.data.ValueObject = null;
    this.onFieldChange(field.data);
}

// Reset all fields
resetAllFields() {
    this.configFields.forEach(field => {
        if (!field.data._InheritedConfig) {
            this.resetField(field);
        }
    });
}
```

### **3. Validation**
```typescript
// ✅ Đúng - Add field validation
validateField(field: any): boolean {
    if (field.required && !field.data.ValueObject) {
        this.env.showMessage(`${field.label} is required`, 'warning');
        return false;
    }
    return true;
}

validateAllFields(): boolean {
    return this.configFields.every(field => this.validateField(field));
}
```

---

## 🚨 **Common Use Cases**

### **1. System Configuration**
```typescript
export class SystemConfigPage extends PageBase {
    systemFields = [
        { type: 'string', label: 'System Name', /* ... */ },
        { type: 'select-staff', label: 'System Admin', /* ... */ },
        { type: 'toggle', label: 'Maintenance Mode', /* ... */ }
    ];
}
```

### **2. User Preferences**
```typescript
export class UserPreferencesPage extends PageBase {
    preferenceFields = [
        { type: 'select', label: 'Language', /* ... */ },
        { type: 'select', label: 'Theme', /* ... */ },
        { type: 'toggle', label: 'Email Notifications', /* ... */ }
    ];
}
```

### **3. Branch Settings**
```typescript
export class BranchSettingsPage extends PageBase {
    branchFields = [
        { type: 'select-branch', label: 'Parent Branch', /* ... */ },
        { type: 'select-staff', label: 'Branch Manager', /* ... */ },
        { type: 'textarea', label: 'Branch Description', /* ... */ }
    ];
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
