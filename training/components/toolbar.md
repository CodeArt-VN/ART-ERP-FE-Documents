# Toolbar Component Documentation

## 📋 **Tổng quan**

`ToolbarComponent` là generic toolbar component cung cấp các action buttons phổ biến cho pages. Component này tương tự `ListToolbarComponent` nhưng linh hoạt hơn và có thể customize nhiều hơn.

**Selector**: `app-toolbar`  
**Location**: `src/app/components/toolbar/toolbar.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() page: any;                      // Page reference
@Input() pageConfig: any;                // Page configuration
@Input() BackHref: string;               // Back navigation URL
@Input() pageTitle: string;              // Page title
@Input() selectedTitle: string;          // Selected items title
@Input() selectedItems: any[];           // Selected items
@Input() query: any = {};                // Query parameters
@Input() CenterTitle: boolean;           // Center align title
@Input() NoBorder: boolean = false;      // Remove border

// Visibility controls
@Input() ShowAdd: boolean = true;
@Input() ShowSearch: boolean = true;
@Input() ShowRefresh: boolean = true;
@Input() ShowExport: boolean = true;
@Input() ShowImport: boolean = true;
@Input() ShowCopy: boolean = true;
@Input() ShowChangeBranch: boolean = true;
@Input() ShowSubmit: boolean = true;
@Input() ShowApprove: boolean = true;
@Input() ShowDisapprove: boolean = true;
@Input() ShowMerge: boolean = true;
@Input() ShowSplit: boolean = true;
@Input() ShowCancel: boolean = true;
@Input() ShowArchive: boolean = true;
@Input() ShowDelete: boolean = true;
@Input() ShowHelp: boolean = true;
@Input() ShowFeature: boolean = false;

@Input() AcceptFile: string = '.xlsx';   // File upload filter
```

---

## 🚀 **Basic Usage**

### **Simple Page Toolbar**
```typescript
// Component
export class GenericPageWithToolbar extends PageBase {
    pageConfig = {
        pageName: 'generic-page',
        pageTitle: 'GENERIC_PAGE_TITLE',
        isShowFeature: false
    };

    selectedItems: any[] = [];

    // Toolbar actions are automatically handled by the component
    // It calls methods on the page instance (this.page.methodName())
}
```

```html
<!-- Template -->
<app-toolbar
    [page]="this"
    [pageConfig]="pageConfig"
    [selectedItems]="selectedItems"
    [ShowAdd]="true"
    [ShowRefresh]="true"
    [ShowExport]="true"
    [ShowImport]="true">
</app-toolbar>

<ion-content>
    <!-- Page content -->
</ion-content>
```

### **Customized Toolbar**
```typescript
export class CustomToolbarPage extends PageBase {
    pageConfig = {
        pageName: 'custom-page',
        pageTitle: 'CUSTOM_PAGE'
    };

    // Control which buttons to show
    showToolbarButtons = {
        add: true,
        refresh: true,
        export: false,
        import: false,
        delete: true,
        approve: false,
        submit: false
    };
}
```

```html
<app-toolbar
    [page]="this"
    [pageConfig]="pageConfig"
    [selectedItems]="selectedItems"
    [ShowAdd]="showToolbarButtons.add"
    [ShowRefresh]="showToolbarButtons.refresh"
    [ShowExport]="showToolbarButtons.export"
    [ShowImport]="showToolbarButtons.import"
    [ShowDelete]="showToolbarButtons.delete"
    [ShowApprove]="showToolbarButtons.approve"
    [ShowSubmit]="showToolbarButtons.submit">
</app-toolbar>
```

---

## 🎛️ **Advanced Features**

### **Popover Actions**
```typescript
export class ToolbarWithPopover extends PageBase {
    // The component automatically handles popover for additional actions
    // Popover is triggered by presentToolBarPopover() method
    
    // Additional custom actions can be added to the popover
    customPopoverActions = [
        { name: 'Custom Action 1', icon: 'star', action: () => this.customAction1() },
        { name: 'Custom Action 2', icon: 'heart', action: () => this.customAction2() }
    ];

    customAction1() {
        console.log('Custom action 1 executed');
    }

    customAction2() {
        console.log('Custom action 2 executed');
    }
}
```

### **File Import Configuration**
```html
<app-toolbar
    [page]="this"
    [AcceptFile]="'.xlsx,.csv,.json'"
    [ShowImport]="true">
</app-toolbar>
```

---

## 🎨 **Styling**

### **Custom Toolbar Styling**
```scss
// Component SCSS
app-toolbar {
    .toolbar-container {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 12px 16px;
        background: var(--ion-color-light);
        border-bottom: 1px solid var(--ion-color-medium);
        
        &.no-border {
            border-bottom: none;
        }
        
        .toolbar-start {
            display: flex;
            align-items: center;
            gap: 12px;
            
            .back-button {
                --color: var(--ion-color-primary);
            }
            
            .toolbar-title {
                font-size: 1.25rem;
                font-weight: 600;
                color: var(--ion-color-dark);
                
                &.center-title {
                    text-align: center;
                    flex: 1;
                }
            }
        }
        
        .toolbar-end {
            display: flex;
            align-items: center;
            gap: 8px;
            
            .action-button {
                --border-radius: 6px;
                height: 36px;
                
                &.primary-action {
                    --background: var(--ion-color-primary);
                    --color: var(--ion-color-primary-contrast);
                }
                
                &.secondary-action {
                    --background: transparent;
                    --color: var(--ion-color-medium);
                    --border-color: var(--ion-color-medium);
                }
            }
            
            .popover-trigger {
                --color: var(--ion-color-medium);
            }
        }
    }
}
```

---

## 🔧 **Method Integration**

### **Required Page Methods**
```typescript
// Your page component should implement these methods for toolbar integration:

export class ToolbarIntegratedPage extends PageBase {
    // Basic actions
    add() {
        // Handle add action
        this.nav('/item/new', 'forward');
    }

    refresh() {
        // Handle refresh action
        this.loadData();
    }

    export() {
        // Handle export action
        this.exportData();
    }

    import(event: any) {
        // Handle import action
        this.importData(event);
    }

    delete() {
        // Handle delete action
        this.deleteSelected();
    }

    // Workflow actions
    submit() {
        // Handle submit action
        this.submitForApproval();
    }

    approve() {
        // Handle approve action
        this.approveSelected();
    }

    disapprove() {
        // Handle disapprove action
        this.disapproveSelected();
    }

    // Feature toggle
    toggleFeature() {
        // Handle feature toggle
        this.pageConfig.isShowFeature = !this.pageConfig.isShowFeature;
    }

    // Advanced filter
    openAdvanceFilter() {
        // Handle advanced filter
        this.showAdvancedFilterModal();
    }
}
```

---

## 📱 **Mobile Optimization**

### **Responsive Toolbar**
```scss
// Mobile-specific styles
@media (max-width: 768px) {
    app-toolbar {
        .toolbar-container {
            flex-wrap: wrap;
            gap: 8px;
            
            .toolbar-start {
                flex: 1;
                min-width: 200px;
                
                .toolbar-title {
                    font-size: 1.125rem;
                }
            }
            
            .toolbar-end {
                flex-wrap: wrap;
                
                .action-button {
                    height: 40px;
                    min-width: 40px;
                    
                    // Hide text on mobile, show only icons
                    .button-inner {
                        flex-direction: column;
                        
                        span {
                            display: none;
                        }
                    }
                }
                
                // Show popover for additional actions on mobile
                .popover-trigger {
                    display: block;
                }
            }
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Method Implementation**
```typescript
// ✅ Đúng - Implement all required methods
export class ProperToolbarPage extends PageBase {
    // Always implement methods that toolbar buttons will call
    add() { /* implementation */ }
    refresh() { /* implementation */ }
    delete() { /* implementation */ }
    // ... other methods
}

// ❌ Sai - Missing method implementations
export class ImproperToolbarPage extends PageBase {
    // Toolbar buttons will fail if methods are not implemented
}
```

### **2. Button Visibility Logic**
```typescript
// ✅ Đúng - Dynamic button visibility
get toolbarConfig() {
    return {
        ShowAdd: this.canAdd,
        ShowDelete: this.canDelete && this.selectedItems.length > 0,
        ShowApprove: this.canApprove && this.hasItemsToApprove,
        ShowExport: this.items.length > 0
    };
}
```

### **3. Error Handling**
```typescript
// ✅ Đúng - Handle toolbar action errors
async delete() {
    try {
        if (this.selectedItems.length === 0) {
            this.env.showMessage('NO_ITEMS_SELECTED', 'warning');
            return;
        }
        
        const confirmed = await this.env.actionConfirm('delete', this.selectedItems.length, 'items');
        if (confirmed) {
            await this.service.delete(this.selectedItems);
            this.env.showMessage('DELETE_SUCCESS', 'success');
            this.refresh();
        }
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
