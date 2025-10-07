# CardMultiRow Component Documentation

## 📋 **Tổng quan**

`CardMultiRowComponent` là component hiển thị dữ liệu dưới dạng card với nhiều rows, thường dùng cho dashboard widgets hoặc summary displays.

**Selector**: `app-card-multi-row`  
**Location**: `src/app/components/visualizations/card-multi-row/card-multi-row.component.ts`

---

## 🚀 **Basic Usage**

### **Dashboard Summary Card**
```typescript
// Component
export class DashboardPage extends PageBase {
    summaryData = [
        { label: 'Total Orders', value: '1,234', icon: 'receipt', color: 'primary' },
        { label: 'Revenue', value: '$45,678', icon: 'cash', color: 'success' },
        { label: 'Customers', value: '567', icon: 'people', color: 'tertiary' },
        { label: 'Products', value: '89', icon: 'cube', color: 'warning' }
    ];
}
```

```html
<!-- Template -->
<app-card-multi-row
    [title]="'Business Overview'"
    [data]="summaryData">
</app-card-multi-row>
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
