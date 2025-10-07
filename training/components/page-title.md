# PageTitle Component Documentation

## 📋 **Tổng quan**

`PageTitleComponent` là component đơn giản để hiển thị title của page với icon, color và remark. Component này tự động lấy thông tin từ `pageConfig` và hiển thị consistent title format.

**Selector**: `app-page-title`  
**Location**: `src/app/components/page-title/page-title.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() icon: string;                   // Page icon
@Input() color: string;                  // Icon color
@Input() title: string;                  // Page title
@Input() remark: string;                 // Additional remark
@Input() pageName: string;               // Page identifier
@Input() pageConfig: any;                // Complete page configuration
```

---

## 🚀 **Basic Usage**

### **Simple Page Title**
```typescript
// Component
export class UserListPage extends PageBase {
    pageConfig = {
        pageName: 'user-list',
        pageTitle: 'USER_MANAGEMENT',
        pageIcon: 'people',
        pageColor: 'primary',
        pageRemark: 'Manage system users'
    };
}
```

```html
<!-- Template -->
<app-page-title [pageConfig]="pageConfig"></app-page-title>

<!-- Or individual properties -->
<app-page-title
    [icon]="'people'"
    [color]="'primary'"
    [title]="'USER_MANAGEMENT'"
    [remark]="'Manage system users'">
</app-page-title>
```

### **Dynamic Page Title**
```typescript
export class DynamicTitlePage extends PageBase {
    pageConfig: any = {};

    ngOnInit() {
        this.updatePageTitle();
    }

    updatePageTitle() {
        if (this.isEditMode) {
            this.pageConfig = {
                pageTitle: 'EDIT_PRODUCT',
                pageIcon: 'create',
                pageColor: 'warning',
                pageRemark: `Editing: ${this.item.name}`
            };
        } else {
            this.pageConfig = {
                pageTitle: 'NEW_PRODUCT',
                pageIcon: 'add',
                pageColor: 'success',
                pageRemark: 'Create new product'
            };
        }
    }
}
```

---

## 🎨 **Styling & Themes**

### **Custom Styling**
```scss
// Component SCSS
app-page-title {
    .page-title-container {
        display: flex;
        align-items: center;
        padding: 16px;
        background: var(--ion-color-light);
        border-bottom: 1px solid var(--ion-color-medium);
        
        .page-icon {
            font-size: 1.5rem;
            margin-right: 12px;
            
            &.primary { color: var(--ion-color-primary); }
            &.success { color: var(--ion-color-success); }
            &.warning { color: var(--ion-color-warning); }
            &.danger { color: var(--ion-color-danger); }
        }
        
        .page-info {
            flex: 1;
            
            .page-title {
                font-size: 1.25rem;
                font-weight: 600;
                color: var(--ion-color-dark);
                margin: 0;
            }
            
            .page-remark {
                font-size: 0.875rem;
                color: var(--ion-color-medium);
                margin: 4px 0 0 0;
            }
        }
    }
}
```

### **Theme Variants**
```html
<!-- Primary theme -->
<app-page-title
    [icon]="'business'"
    [color]="'primary'"
    [title]="'DASHBOARD'"
    class="primary-theme">
</app-page-title>

<!-- Success theme -->
<app-page-title
    [icon]="'checkmark-circle'"
    [color]="'success'"
    [title]="'COMPLETED_TASKS'"
    class="success-theme">
</app-page-title>

<!-- Dark theme -->
<app-page-title
    [icon]="'settings'"
    [color]="'dark'"
    [title]="'SYSTEM_SETTINGS'"
    class="dark-theme">
</app-page-title>
```

---

## 📱 **Responsive Design**

### **Mobile Optimization**
```scss
// Mobile styles
@media (max-width: 768px) {
    app-page-title {
        .page-title-container {
            padding: 12px 16px;
            
            .page-icon {
                font-size: 1.25rem;
                margin-right: 8px;
            }
            
            .page-info {
                .page-title {
                    font-size: 1.125rem;
                }
                
                .page-remark {
                    font-size: 0.8125rem;
                }
            }
        }
    }
}
```

---

## 🔧 **Integration Patterns**

### **With Page Layout**
```html
<!-- Standard page layout -->
<ion-header>
    <ion-toolbar>
        <ion-title>{{ pageConfig.pageTitle | translate }}</ion-title>
    </ion-toolbar>
</ion-header>

<ion-content>
    <app-page-title [pageConfig]="pageConfig"></app-page-title>
    
    <!-- Page content -->
    <div class="page-content">
        <!-- Your content here -->
    </div>
</ion-content>
```

### **With Breadcrumbs**
```html
<app-page-title [pageConfig]="pageConfig"></app-page-title>

<app-branch-breadcrumbs 
    [items]="breadcrumbItems">
</app-branch-breadcrumbs>

<div class="page-content">
    <!-- Content -->
</div>
```

### **With Toolbar Integration**
```html
<app-page-title [pageConfig]="pageConfig"></app-page-title>

<app-list-toolbar
    [pageConfig]="pageConfig"
    [selectedItems]="selectedItems"
    (add)="add()"
    (refresh)="refresh()">
</app-list-toolbar>

<app-data-table
    [rows]="items"
    [columns]="columns">
</app-data-table>
```

---

## 🚨 **Common Use Cases**

### **1. List Pages**
```typescript
export class ProductListPage extends PageBase {
    pageConfig = {
        pageName: 'product-list',
        pageTitle: 'PRODUCT_MANAGEMENT',
        pageIcon: 'cube',
        pageColor: 'primary',
        pageRemark: 'Manage product catalog'
    };
}
```

### **2. Detail Pages**
```typescript
export class ProductDetailPage extends PageBase {
    pageConfig: any = {};
    
    ngOnInit() {
        if (this.id) {
            this.pageConfig = {
                pageTitle: 'PRODUCT_DETAIL',
                pageIcon: 'information-circle',
                pageColor: 'tertiary',
                pageRemark: `Product ID: ${this.id}`
            };
        } else {
            this.pageConfig = {
                pageTitle: 'NEW_PRODUCT',
                pageIcon: 'add-circle',
                pageColor: 'success',
                pageRemark: 'Create new product'
            };
        }
    }
}
```

### **3. Report Pages**
```typescript
export class SalesReportPage extends PageBase {
    pageConfig = {
        pageName: 'sales-report',
        pageTitle: 'SALES_REPORT',
        pageIcon: 'bar-chart',
        pageColor: 'success',
        pageRemark: 'Sales analytics and insights'
    };
}
```

### **4. Settings Pages**
```typescript
export class SystemSettingsPage extends PageBase {
    pageConfig = {
        pageName: 'system-settings',
        pageTitle: 'SYSTEM_SETTINGS',
        pageIcon: 'settings',
        pageColor: 'medium',
        pageRemark: 'Configure system preferences'
    };
}
```

---

## 🔧 **Best Practices**

### **1. Consistent Naming**
```typescript
// ✅ Đúng - Consistent page config
pageConfig = {
    pageName: 'user-management',     // kebab-case
    pageTitle: 'USER_MANAGEMENT',    // UPPER_CASE for translation
    pageIcon: 'people',              // Ionic icon name
    pageColor: 'primary',            // Ionic color
    pageRemark: 'Manage users'       // Clear description
};

// ❌ Sai - Inconsistent naming
pageConfig = {
    pageName: 'UserManagement',      // Wrong case
    pageTitle: 'User Management',    // Not translation key
    pageIcon: 'ion-people',          // Wrong icon format
    pageColor: '#3880ff',            // Direct color value
    pageRemark: ''                   // Empty remark
};
```

### **2. Translation Integration**
```typescript
// ✅ Đúng - Use translation keys
pageConfig = {
    pageTitle: 'USER_MANAGEMENT',    // Will be translated
    pageRemark: 'USER_MANAGEMENT_DESC'
};

// Component template
// <app-page-title [pageConfig]="pageConfig"></app-page-title>
// Title will automatically be translated by the component
```

### **3. Dynamic Updates**
```typescript
// ✅ Đúng - Update page title based on context
updatePageTitle(context: any) {
    if (context.isEditing) {
        this.pageConfig.pageTitle = 'EDIT_ITEM';
        this.pageConfig.pageIcon = 'create';
        this.pageConfig.pageColor = 'warning';
        this.pageConfig.pageRemark = `Editing: ${context.item.name}`;
    } else {
        this.pageConfig.pageTitle = 'VIEW_ITEM';
        this.pageConfig.pageIcon = 'eye';
        this.pageConfig.pageColor = 'tertiary';
        this.pageConfig.pageRemark = `Viewing: ${context.item.name}`;
    }
}
```

---

## 🎯 **Advanced Features**

### **Custom Icon Integration**
```html
<app-page-title [pageConfig]="pageConfig">
    <!-- Custom icon slot -->
    <ng-container slot="icon">
        <img [src]="customIconUrl" alt="Custom Icon" class="custom-icon">
    </ng-container>
</app-page-title>
```

### **Action Buttons Integration**
```html
<app-page-title [pageConfig]="pageConfig">
    <!-- Action buttons slot -->
    <ng-container slot="actions">
        <ion-button fill="clear" (click)="showInfo()">
            <ion-icon name="information-circle"></ion-icon>
        </ion-button>
        
        <ion-button fill="clear" (click)="showHelp()">
            <ion-icon name="help-circle"></ion-icon>
        </ion-button>
    </ng-container>
</app-page-title>
```

### **Status Indicator**
```typescript
export class StatusAwarePage extends PageBase {
    pageConfig: any = {};
    
    updatePageStatus(status: string) {
        const statusConfig = {
            'active': { color: 'success', icon: 'checkmark-circle' },
            'pending': { color: 'warning', icon: 'time' },
            'error': { color: 'danger', icon: 'alert-circle' },
            'disabled': { color: 'medium', icon: 'ban' }
        };
        
        const config = statusConfig[status] || statusConfig['active'];
        
        this.pageConfig = {
            ...this.pageConfig,
            pageColor: config.color,
            pageIcon: config.icon,
            pageRemark: `Status: ${status.toUpperCase()}`
        };
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
