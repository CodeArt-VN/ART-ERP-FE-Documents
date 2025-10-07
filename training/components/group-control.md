# GroupControl Component Documentation

## 📋 **Tổng quan**

`GroupControlComponent` là component để nhóm và quản lý các form fields, hỗ trợ configuration và navigation đến config grid. Component này thường được sử dụng để tạo các form sections có thể customize được.

**Selector**: `app-group-control`  
**Location**: `src/app/components/group-control/group-control.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() title: string;                  // Group title
@Input() hideBorder: boolean;            // Hide border styling
@Input() fields: any[] = [];             // Array of field configurations
```

### **Output Events**
```typescript
@Output() onChange = new EventEmitter(); // Field change events
```

---

## 🚀 **Basic Usage**

### **Simple Form Group**
```typescript
// Component
export class UserFormPage extends PageBase {
    personalInfoFields = [
        { Code: 'Name', Label: 'Full Name', Type: 'text', Required: true },
        { Code: 'Email', Label: 'Email Address', Type: 'email', Required: true },
        { Code: 'Phone', Label: 'Phone Number', Type: 'tel', Required: false }
    ];

    addressFields = [
        { Code: 'Street', Label: 'Street Address', Type: 'text', Required: true },
        { Code: 'City', Label: 'City', Type: 'text', Required: true },
        { Code: 'PostalCode', Label: 'Postal Code', Type: 'text', Required: false }
    ];

    onFieldChange(data: any) {
        console.log('Field changed:', data);
        this.formGroup.patchValue(data);
    }
}
```

```html
<!-- Template -->
<form [formGroup]="formGroup">
    <!-- Personal Information Group -->
    <app-group-control
        [title]="'Personal Information'"
        [fields]="personalInfoFields"
        [hideBorder]="false"
        (onChange)="onFieldChange($event)">
    </app-group-control>

    <!-- Address Information Group -->
    <app-group-control
        [title]="'Address Information'"
        [fields]="addressFields"
        [hideBorder]="false"
        (onChange)="onFieldChange($event)">
    </app-group-control>
</form>
```

---

## 🎨 **Advanced Features**

### **Dynamic Field Configuration**
```typescript
export class DynamicFormPage extends PageBase {
    formGroups: any[] = [];

    async loadFormConfiguration(formType: string) {
        try {
            const config = await this.formConfigService.getFormConfig(formType);
            this.formGroups = this.buildFormGroups(config);
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    buildFormGroups(config: any): any[] {
        return config.sections.map(section => ({
            title: section.title,
            fields: section.fields.map(field => ({
                Code: field.code,
                Label: field.label,
                Type: field.type,
                Required: field.required,
                Options: field.options
            }))
        }));
    }

    onGroupFieldChange(groupIndex: number, data: any) {
        console.log(`Group ${groupIndex} field changed:`, data);
        this.updateFormData(groupIndex, data);
    }
}
```

```html
<div class="dynamic-form">
    <app-group-control
        *ngFor="let group of formGroups; let i = index"
        [title]="group.title"
        [fields]="group.fields"
        (onChange)="onGroupFieldChange(i, $event)">
    </app-group-control>
</div>
```

---

## 🔧 **Best Practices**

### **1. Field Configuration Structure**
```typescript
// ✅ Đúng - Structured field config
interface FieldConfig {
    Code: string;                        // Unique field identifier
    Label: string;                       // Display label
    Type: string;                        // Input type
    Required: boolean;                   // Required validation
    Options?: any[];                     // For select fields
    Validation?: any;                    // Custom validation rules
    DefaultValue?: any;                  // Default value
}
```

### **2. Group Organization**
```typescript
// ✅ Đúng - Logical grouping
const formGroups = [
    {
        title: 'Basic Information',
        fields: [/* basic fields */]
    },
    {
        title: 'Contact Details', 
        fields: [/* contact fields */]
    },
    {
        title: 'Preferences',
        fields: [/* preference fields */]
    }
];
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
