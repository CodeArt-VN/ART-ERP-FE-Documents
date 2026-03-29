# ListToolbar Component Documentation

## 📋 **Tổng quan**

`ListToolbarComponent` là toolbar chính cho các list pages, cung cấp các action buttons như Add, Refresh, Export, Import, Delete và các workflow actions như Approve, Disapprove, Submit for Approval.

**Selector**: `app-list-toolbar`  
**Location**: `src/app/components/list-toolbar/list-toolbar.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() pageTitle: string;              // Page title
@Input() selectedTitle: string;          // Selected items title
@Input() pageConfig: any;                // Page configuration
@Input() selectedItems: any[];           // Currently selected items
@Input() query: any;                     // Current query parameters
@Input() page: any;                      // Page reference

// Visibility controls
@Input() ShowAdd: boolean = true;        // Show Add button
@Input() ShowSearch: boolean = true;     // Show Search
@Input() ShowRefresh: boolean = true;    // Show Refresh button
@Input() ShowArchive: boolean = true;    // Show Archive
@Input() ShowExport: boolean = true;     // Show Export button
@Input() ShowImport: boolean = true;     // Show Import button
@Input() ShowHelp: boolean = true;       // Show Help button
@Input() ShowFeature: boolean = true;    // Show Feature toggle
@Input() ShowPopover: boolean = false;   // Show Popover
@Input() NoBorder: boolean = false;      // Remove border
@Input() canSelect: boolean = true;      // Enable selection
@Input() ShowChangeTable: boolean = true; // Show table change
```

### **Output Events**
```typescript
// Basic actions
@Output() add = new EventEmitter();
@Output() refresh = new EventEmitter();
@Output() export = new EventEmitter();
@Output() import = new EventEmitter();
@Output() help = new EventEmitter();
@Output() unselect = new EventEmitter();
@Output() copy = new EventEmitter();
@Output() archiveItems = new EventEmitter();
@Output() deleteItems = new EventEmitter();

// Order workflow
@Output() submitOrdersForApproval = new EventEmitter();
@Output() approveOrders = new EventEmitter();
@Output() disapproveOrders = new EventEmitter();
@Output() cancelOrders = new EventEmitter();
@Output() submitOrders = new EventEmitter();
@Output() mergeOrders = new EventEmitter();
@Output() splitOrder = new EventEmitter();

// Invoice workflow
@Output() createARInvoice = new EventEmitter();
@Output() submitInvoicesForApproval = new EventEmitter();
@Output() approveInvoices = new EventEmitter();
@Output() disapproveInvoices = new EventEmitter();
@Output() cancelInvoices = new EventEmitter();
@Output() createEInvoice = new EventEmitter();
@Output() updateEInvoice = new EventEmitter();
@Output() signEInvoice = new EventEmitter();
@Output() syncEInvoice = new EventEmitter();

// Business Partner workflow
@Output() submitBusinessPartner = new EventEmitter();
@Output() approveBusinessPartner = new EventEmitter();
@Output() disapproveBusinessPartner = new EventEmitter();

// Other workflows
@Output() changeBranch = new EventEmitter();
@Output() changeTable = new EventEmitter();
@Output() presentPopover = new EventEmitter();
```

---

## 🚀 **Basic Usage**

### **Simple List Toolbar**
```typescript
// Component
export class UserListPage extends PageBase {
    selectedItems: any[] = [];
    pageConfig = {
        pageName: 'user',
        pageTitle: 'USER_LIST',
        isShowFeature: false
    };

    add() {
        this.nav('/user/user-detail', 'forward');
    }

    refresh() {
        this.loadData();
    }

    async deleteSelected() {
        if (this.selectedItems.length === 0) return;
        
        const confirmed = await this.env.actionConfirm(
            'delete', 
            this.selectedItems.length, 
            'users', 
            'DELETE_USERS'
        );
        
        if (confirmed) {
            await this.userService.delete(this.selectedItems);
            this.env.showMessage('DELETE_SUCCESS', 'success');
            this.refresh();
        }
    }
}
```

```html
<!-- Template -->
<app-list-toolbar
    [pageTitle]="pageConfig.pageTitle"
    [pageConfig]="pageConfig"
    [selectedItems]="selectedItems"
    [ShowAdd]="true"
    [ShowRefresh]="true"
    [ShowExport]="true"
    [ShowImport]="false"
    (add)="add()"
    (refresh)="refresh()"
    (export)="export()"
    (deleteItems)="deleteSelected()">
</app-list-toolbar>
```

---

## 📋 **Workflow Actions**

### **Order Management**
```typescript
export class SaleOrderListPage extends PageBase {
    selectedOrders: any[] = [];
    pageConfig = {
        pageName: 'sale-order',
        pageTitle: 'SALE_ORDER_LIST'
    };

    async submitForApproval() {
        if (this.selectedOrders.length === 0) return;
        
        try {
            await this.saleOrderService.submitForApproval(this.selectedOrders);
            this.env.showMessage('SUBMIT_SUCCESS', 'success');
            this.refresh();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async approveOrders() {
        if (this.selectedOrders.length === 0) return;
        
        const confirmed = await this.env.actionConfirm(
            'approve', 
            this.selectedOrders.length, 
            'orders'
        );
        
        if (confirmed) {
            await this.saleOrderService.approve(this.selectedOrders);
            this.env.showMessage('APPROVE_SUCCESS', 'success');
            this.refresh();
        }
    }

    async disapproveOrders() {
        if (this.selectedOrders.length === 0) return;
        
        const reason = await this.env.showPrompt(
            'DISAPPROVE_REASON',
            'Enter reason for disapproval'
        );
        
        if (reason) {
            await this.saleOrderService.disapprove(this.selectedOrders, reason);
            this.env.showMessage('DISAPPROVE_SUCCESS', 'success');
            this.refresh();
        }
    }

    async cancelOrders() {
        if (this.selectedOrders.length === 0) return;
        
        const confirmed = await this.env.actionConfirm(
            'cancel', 
            this.selectedOrders.length, 
            'orders'
        );
        
        if (confirmed) {
            await this.saleOrderService.cancel(this.selectedOrders);
            this.env.showMessage('CANCEL_SUCCESS', 'success');
            this.refresh();
        }
    }
}
```

```html
<app-list-toolbar
    [pageConfig]="pageConfig"
    [selectedItems]="selectedOrders"
    (add)="add()"
    (refresh)="refresh()"
    (submitOrdersForApproval)="submitForApproval()"
    (approveOrders)="approveOrders()"
    (disapproveOrders)="disapproveOrders()"
    (cancelOrders)="cancelOrders()"
    (deleteItems)="deleteSelected()">
</app-list-toolbar>
```

### **Invoice Management**
```typescript
export class ARInvoiceListPage extends PageBase {
    selectedInvoices: any[] = [];
    pageConfig = {
        pageName: 'arinvoice',
        pageTitle: 'AR_INVOICE_LIST'
    };

    async createEInvoice() {
        if (this.selectedInvoices.length === 0) return;
        
        try {
            const result = await this.arInvoiceService.createEInvoice(this.selectedInvoices);
            this.env.showMessage('EINVOICE_CREATED', 'success');
            this.refresh();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async updateEInvoice() {
        if (this.selectedInvoices.length === 0) return;
        
        try {
            await this.arInvoiceService.updateEInvoice(this.selectedInvoices);
            this.env.showMessage('EINVOICE_UPDATED', 'success');
            this.refresh();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async signEInvoice() {
        if (this.selectedInvoices.length === 0) return;
        
        try {
            await this.arInvoiceService.signEInvoice(this.selectedInvoices);
            this.env.showMessage('EINVOICE_SIGNED', 'success');
            this.refresh();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

```html
<app-list-toolbar
    [pageConfig]="pageConfig"
    [selectedItems]="selectedInvoices"
    (createEInvoice)="createEInvoice()"
    (updateEInvoice)="updateEInvoice()"
    (signEInvoice)="signEInvoice()"
    (syncEInvoice)="syncEInvoice()"
    (submitInvoicesForApproval)="submitForApproval()"
    (approveInvoices)="approveInvoices()"
    (disapproveInvoices)="disapproveInvoices()">
</app-list-toolbar>
```

---

## 📤 **Import/Export Functions**

### **Export Data**
```typescript
export class ProductListPage extends PageBase {
    async exportData() {
        try {
            this.env.showLoading('EXPORTING_DATA', 
                this.productService.export(this.query)
            );
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async exportSelected() {
        if (this.selectedItems.length === 0) {
            this.env.showMessage('NO_ITEMS_SELECTED', 'warning');
            return;
        }

        try {
            const ids = this.selectedItems.map(item => item.Id);
            this.env.showLoading('EXPORTING_SELECTED', 
                this.productService.export({ Ids: ids })
            );
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **Import Data**
```typescript
export class ProductListPage extends PageBase {
    async importData(event: any) {
        const file = event.target.files[0];
        if (!file) return;

        try {
            this.env.showLoading('IMPORTING_DATA',
                this.productService.import(file)
            );
            this.refresh();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    downloadTemplate() {
        // Download import template
        const templateUrl = '/assets/templates/product-import-template.xlsx';
        window.open(templateUrl, '_blank');
    }
}
```

```html
<app-list-toolbar
    [pageConfig]="pageConfig"
    [selectedItems]="selectedItems"
    [ShowImport]="true"
    [ShowExport]="true"
    (export)="exportData()"
    (import)="importData($event)">
    
    <!-- Custom import help -->
    <ion-button 
        fill="clear" 
        slot="end"
        (click)="downloadTemplate()">
        <ion-icon name="download"></ion-icon>
        Template
    </ion-button>
</app-list-toolbar>
```

---

## 🔄 **Dynamic Button Visibility**

### **Status-Based Actions**
```typescript
export class OrderListPage extends PageBase {
    selectedOrders: any[] = [];
    
    // Toolbar sẽ tự động ẩn/hiện buttons dựa trên status
    ngOnChanges() {
        // ListToolbar component tự động handle logic này
        // dựa trên pageConfig.pageName và selectedItems status
    }

    // Custom visibility logic
    get canSubmitForApproval(): boolean {
        return this.selectedOrders.every(order => 
            ['New', 'Unapproved'].includes(order.Status)
        );
    }

    get canApprove(): boolean {
        return this.selectedOrders.every(order => 
            order.Status === 'Submitted'
        );
    }

    get canCancel(): boolean {
        return this.selectedOrders.every(order => 
            !['Canceled', 'Delivered'].includes(order.Status)
        );
    }
}
```

### **Permission-Based Actions**
```typescript
export class SecureListPage extends PageBase {
    showAddButton = false;
    showDeleteButton = false;
    showApproveButton = false;

    async ngOnInit() {
        await this.checkPermissions();
    }

    async checkPermissions() {
        this.showAddButton = await this.env.checkFormPermission('/user/create');
        this.showDeleteButton = await this.env.checkFormPermission('/user/delete');
        this.showApproveButton = await this.env.checkFormPermission('/user/approve');
    }
}
```

```html
<app-list-toolbar
    [pageConfig]="pageConfig"
    [selectedItems]="selectedItems"
    [ShowAdd]="showAddButton"
    [ShowArchive]="showDeleteButton"
    (add)="add()"
    (deleteItems)="deleteSelected()"
    (approveOrders)="approveSelected()">
</app-list-toolbar>
```

---

## 🎨 **Customization**

### **Custom Buttons**
```html
<app-list-toolbar
    [pageConfig]="pageConfig"
    [selectedItems]="selectedItems">
    
    <!-- Add custom buttons -->
    <ion-button 
        slot="end"
        fill="outline"
        (click)="customAction()">
        <ion-icon name="settings"></ion-icon>
        Settings
    </ion-button>
    
    <ion-button 
        slot="end"
        fill="solid"
        color="success"
        (click)="bulkUpdate()">
        <ion-icon name="create"></ion-icon>
        Bulk Update
    </ion-button>
</app-list-toolbar>
```

### **Custom Styling**
```scss
// Component SCSS
app-list-toolbar {
    .toolbar-container {
        background: var(--ion-color-light);
        border-bottom: 1px solid var(--ion-color-medium);
        padding: 12px 16px;
        
        .toolbar-title {
            font-size: 1.25rem;
            font-weight: 600;
            color: var(--ion-color-dark);
        }
        
        .selected-info {
            font-size: 0.875rem;
            color: var(--ion-color-medium);
            margin-left: 8px;
        }
        
        .action-buttons {
            display: flex;
            gap: 8px;
            
            ion-button {
                --border-radius: 6px;
                height: 36px;
                
                &.primary-action {
                    --background: var(--ion-color-primary);
                    --color: var(--ion-color-primary-contrast);
                }
                
                &.danger-action {
                    --background: var(--ion-color-danger);
                    --color: var(--ion-color-danger-contrast);
                }
            }
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
    app-list-toolbar {
        .toolbar-container {
            flex-direction: column;
            gap: 12px;
            
            .toolbar-title {
                font-size: 1.125rem;
            }
            
            .action-buttons {
                flex-wrap: wrap;
                justify-content: center;
                
                ion-button {
                    min-width: 120px;
                    height: 40px;
                }
            }
        }
    }
}
```

### **Touch-Friendly Actions**
```html
<app-list-toolbar
    [pageConfig]="pageConfig"
    [selectedItems]="selectedItems"
    class="mobile-optimized">
    
    <!-- Mobile-specific popover -->
    <ion-button 
        *ngIf="isMobile"
        fill="clear"
        (click)="presentActionSheet()">
        <ion-icon name="ellipsis-vertical"></ion-icon>
    </ion-button>
</app-list-toolbar>
```

---

## 🔧 **Best Practices**

### **1. Event Handling**
```typescript
// ✅ Đúng - Proper error handling
async approveSelected() {
    if (this.selectedItems.length === 0) {
        this.env.showMessage('NO_ITEMS_SELECTED', 'warning');
        return;
    }

    try {
        const confirmed = await this.env.actionConfirm(
            'approve', 
            this.selectedItems.length, 
            'items'
        );
        
        if (confirmed) {
            await this.service.approve(this.selectedItems);
            this.env.showMessage('APPROVE_SUCCESS', 'success');
            this.refresh();
        }
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}

// ❌ Sai - No validation or error handling
async approveSelected() {
    await this.service.approve(this.selectedItems);
    this.refresh();
}
```

### **2. Permission Checking**
```typescript
// ✅ Đúng - Check permissions
async ngOnInit() {
    this.canAdd = await this.env.checkFormPermission('/user/create');
    this.canDelete = await this.env.checkFormPermission('/user/delete');
}

// ❌ Sai - No permission check
ngOnInit() {
    this.canAdd = true;
    this.canDelete = true;
}
```

### **3. Loading States**
```typescript
// ✅ Đúng - Show loading during operations
async deleteSelected() {
    this.loading = true;
    try {
        await this.env.showLoading('DELETING_ITEMS',
            this.service.delete(this.selectedItems)
        );
        this.refresh();
    } finally {
        this.loading = false;
    }
}
```

---

## 🚨 **Common Patterns**

### **1. Bulk Operations**
```typescript
export class BulkOperationsPage extends PageBase {
    async bulkUpdate(field: string, value: any) {
        if (this.selectedItems.length === 0) return;
        
        const confirmed = await this.env.actionConfirm(
            'update',
            this.selectedItems.length,
            'items'
        );
        
        if (confirmed) {
            const updates = this.selectedItems.map(item => ({
                ...item,
                [field]: value
            }));
            
            await this.service.bulkUpdate(updates);
            this.env.showMessage('BULK_UPDATE_SUCCESS', 'success');
            this.refresh();
        }
    }

    async bulkChangeStatus(status: string) {
        await this.bulkUpdate('Status', status);
    }

    async bulkChangeBranch(branchId: number) {
        await this.bulkUpdate('BranchId', branchId);
    }
}
```

### **2. Conditional Actions**
```typescript
export class ConditionalActionsPage extends PageBase {
    get availableActions(): string[] {
        const actions = [];
        
        if (this.canSubmitForApproval) actions.push('submit');
        if (this.canApprove) actions.push('approve');
        if (this.canCancel) actions.push('cancel');
        if (this.canDelete) actions.push('delete');
        
        return actions;
    }

    get canSubmitForApproval(): boolean {
        return this.selectedItems.every(item => 
            ['New', 'Unapproved'].includes(item.Status)
        );
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
