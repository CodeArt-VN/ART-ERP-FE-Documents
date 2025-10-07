# DynamicTemplate Component Documentation

## 📋 **Tổng quan**

`DynamicTemplateComponent` là component render dynamic templates dựa trên configuration, cho phép tạo UI linh hoạt từ data configuration.

**Selector**: `app-dynamic-template`  
**Location**: `src/app/components/dynamic-template/dynamic-template.component.ts`

---

## 🚀 **Basic Usage**

### **Dynamic Form Rendering**
```typescript
// Component
export class DynamicFormPage extends PageBase {
    templateConfig = {
        type: 'form',
        fields: [
            { type: 'text', name: 'name', label: 'Name', required: true },
            { type: 'email', name: 'email', label: 'Email', required: true },
            { type: 'select', name: 'role', label: 'Role', options: ['Admin', 'User'] }
        ]
    };

    onTemplateChange(data: any) {
        console.log('Template data changed:', data);
    }
}
```

```html
<!-- Template -->
<app-dynamic-template
    [config]="templateConfig"
    (dataChange)="onTemplateChange($event)">
</app-dynamic-template>
```

---

## 🎨 **Advanced Features**

### **Dynamic Dashboard Widgets**
```typescript
export class DynamicDashboardPage extends PageBase {
    widgetConfigs = [
        {
            type: 'chart',
            title: 'Sales Overview',
            chartType: 'line',
            dataSource: 'sales-api'
        },
        {
            type: 'table',
            title: 'Recent Orders',
            columns: ['id', 'customer', 'amount', 'status'],
            dataSource: 'orders-api'
        }
    ];
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
