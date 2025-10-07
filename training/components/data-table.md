# DataTable Component Documentation

## 📋 **Tổng quan**

`DataTableComponent` là component chính để hiển thị data dưới dạng table với các tính năng advanced như filtering, sorting, pagination, selection và tree view.

**Selector**: `app-data-table`  
**Location**: `src/app/components/data-table/data-table.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() rows: any[];                    // Data để hiển thị
@Input() columns: TableColumn[];         // Column configuration
@Input() selectedRows: any[];            // Selected rows (two-way binding)
@Input() query: any;                     // Query parameters cho filter
@Input() trackBy: string;                // Field để track changes
@Input() showSpinner: boolean;           // Hiển thị loading spinner
@Input() showFilter: boolean;            // Hiển thị filter row
@Input() isQueryLocalOnly: boolean;      // Local filtering vs server filtering
@Input() isTreeList: boolean;            // Enable tree view
```

### **Output Events**
```typescript
@Output() selectedRowsChange: EventEmitter<any[]>;  // Selection changes
@Output() filter: EventEmitter<any>;                // Filter events
@Output() sort: EventEmitter<any>;                  // Sort events
@Output() activate: EventEmitter<any>;              // Row activation
@Output() dataInfinite: EventEmitter<any>;          // Infinite scroll
```

---

## 🚀 **Basic Usage**

### **Simple Table**
```typescript
// Component
export class UserListPage {
    items: any[] = [];
    columns: TableColumn[] = [
        { property: 'name', header: 'Name', canSort: true },
        { property: 'email', header: 'Email', canSort: true },
        { property: 'status', header: 'Status', canFilter: true }
    ];
    selectedItems: any[] = [];

    onFilter(event: any) {
        console.log('Filter:', event.query);
        // Handle filter logic
    }

    onSort(event: any) {
        console.log('Sort:', event);
        // Handle sort logic
    }
}
```

```html
<!-- Template -->
<app-data-table
    [rows]="items"
    [columns]="columns"
    [(selectedRows)]="selectedItems"
    [showFilter]="true"
    (filter)="onFilter($event)"
    (sort)="onSort($event)">
</app-data-table>
```

---

## 📊 **Advanced Usage**

### **Custom Column Templates**
```html
<app-data-table [rows]="items" [columns]="columns">
    <!-- Custom header template -->
    <app-data-table-column property="name" header="User Name" [canSort]="true">
        <ng-template #headerTemplate>
            <ion-icon name="person"></ion-icon> User Name
        </ng-template>
        
        <ng-template #cellTemplate let-row let-index="index">
            <div class="user-cell">
                <ion-avatar>
                    <img [src]="row.avatar || 'assets/default-avatar.png'">
                </ion-avatar>
                <div>
                    <strong>{{ row.name }}</strong>
                    <div class="text-muted">{{ row.title }}</div>
                </div>
            </div>
        </ng-template>
    </app-data-table-column>

    <!-- Status column với color coding -->
    <app-data-table-column property="status" header="Status" [canFilter]="true">
        <ng-template #cellTemplate let-row>
            <ion-badge [color]="getStatusColor(row.status)">
                {{ row.status }}
            </ion-badge>
        </ng-template>
        
        <ng-template #filterTemplate let-column>
            <ion-select [(ngModel)]="column.filterValue" placeholder="All Status">
                <ion-select-option value="">All</ion-select-option>
                <ion-select-option value="active">Active</ion-select-option>
                <ion-select-option value="inactive">Inactive</ion-select-option>
            </ion-select>
        </ng-template>
    </app-data-table-column>

    <!-- Action column -->
    <app-data-table-column property="actions" header="Actions" [canSort]="false">
        <ng-template #cellTemplate let-row>
            <ion-button fill="clear" (click)="editUser(row)">
                <ion-icon name="create"></ion-icon>
            </ion-button>
            <ion-button fill="clear" color="danger" (click)="deleteUser(row)">
                <ion-icon name="trash"></ion-icon>
            </ion-button>
        </ng-template>
    </app-data-table-column>
</app-data-table>
```

### **Tree View Table**
```typescript
// Component
export class CategoryTreePage {
    categories: any[] = [];
    columns: TableColumn[] = [
        { property: 'name', header: 'Category Name', canSort: true },
        { property: 'code', header: 'Code', canSort: true },
        { property: 'itemCount', header: 'Items', canSort: true }
    ];

    async loadData() {
        // Load hierarchical data
        const result = await this.categoryService.read();
        this.categories = result.data;
    }
}
```

```html
<app-data-table
    [rows]="categories"
    [columns]="columns"
    [isTreeList]="true"
    [showFilter]="true">
    
    <app-data-table-column property="name" header="Category">
        <ng-template #cellTemplate let-row>
            <div [style.margin-left.px]="row.level * 20">
                <ion-button 
                    *ngIf="row.HasChild" 
                    fill="clear" 
                    size="small"
                    (click)="toggleRow(row)">
                    <ion-icon [name]="row.showdetail ? 'chevron-down' : 'chevron-forward'"></ion-icon>
                </ion-button>
                <span>{{ row.name }}</span>
            </div>
        </ng-template>
    </app-data-table-column>
</app-data-table>
```

---

## 🔍 **Filtering**

### **Server-side Filtering**
```typescript
export class ProductListPage {
    query: any = {};
    
    onFilter(event: any) {
        // Update query và gọi API
        Object.assign(this.query, event.query);
        this.loadData();
    }

    async loadData() {
        const result = await this.productService.read(this.query);
        this.items = result.data;
    }
}
```

### **Client-side Filtering**
```html
<app-data-table
    [rows]="items"
    [columns]="columns"
    [isQueryLocalOnly]="true"
    [showFilter]="true">
</app-data-table>
```

### **Custom Filter Controls**
```typescript
columns: TableColumn[] = [
    {
        property: 'dateCreated',
        header: 'Created Date',
        canFilter: true,
        filterControlType: 'time-frame'  // Special time range filter
    },
    {
        property: 'category',
        header: 'Category',
        canFilter: true,
        filterControlType: 'select',
        filterOptions: [
            { value: 'electronics', label: 'Electronics' },
            { value: 'clothing', label: 'Clothing' }
        ]
    }
];
```

---

## 📋 **Selection Management**

### **Single Selection**
```html
<app-data-table
    [rows]="items"
    [columns]="columns"
    [(selectedRows)]="selectedItems"
    (selectedRowsChange)="onSelectionChange($event)">
</app-data-table>
```

### **Multi Selection với Actions**
```typescript
export class OrderListPage {
    selectedOrders: any[] = [];

    onSelectionChange(selected: any[]) {
        this.selectedOrders = selected;
        this.updateToolbarActions();
    }

    updateToolbarActions() {
        // Enable/disable actions based on selection
        const canApprove = this.selectedOrders.every(order => 
            order.status === 'pending'
        );
        this.pageConfig.ShowApprove = canApprove;
    }

    async approveSelected() {
        if (this.selectedOrders.length === 0) return;
        
        try {
            await this.orderService.approve(this.selectedOrders);
            this.env.showMessage('APPROVE_SUCCESS', 'success');
            this.refresh();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

---

## 🎨 **Styling & Themes**

### **Custom Cell Styling**
```scss
// Component SCSS
.user-cell {
    display: flex;
    align-items: center;
    gap: 12px;

    ion-avatar {
        width: 32px;
        height: 32px;
    }

    .text-muted {
        font-size: 0.875rem;
        color: var(--ion-color-medium);
    }
}

// Status badges
.status-badge {
    &.active { --background: var(--ion-color-success); }
    &.inactive { --background: var(--ion-color-medium); }
    &.pending { --background: var(--ion-color-warning); }
}
```

### **Responsive Design**
```typescript
columns: TableColumn[] = [
    { 
        property: 'name', 
        header: 'Name', 
        canSort: true,
        width: '200px',
        minWidth: '150px'
    },
    { 
        property: 'email', 
        header: 'Email', 
        canSort: true,
        hideOnMobile: true  // Ẩn trên mobile
    }
];
```

---

## ⚡ **Performance Optimization**

### **Virtual Scrolling**
```html
<app-data-table
    [rows]="items"
    [columns]="columns"
    [trackBy]="'id'"
    [showSpinner]="loading">
</app-data-table>
```

### **Lazy Loading**
```typescript
export class LargeDataPage {
    items: any[] = [];
    loading = false;
    hasMore = true;

    onInfiniteScroll(event: any) {
        if (this.loading || !this.hasMore) return;

        this.loading = true;
        this.loadMoreData().then(() => {
            this.loading = false;
            event.target.complete();
        });
    }

    async loadMoreData() {
        const result = await this.service.read({
            ...this.query,
            skip: this.items.length,
            take: 50
        });

        if (result.data.length === 0) {
            this.hasMore = false;
        } else {
            this.items = [...this.items, ...result.data];
        }
    }
}
```

---

## 🚨 **Common Patterns**

### **1. Master-Detail Pattern**
```typescript
export class OrderListPage {
    selectedOrder: any = null;

    onRowActivate(event: any) {
        this.selectedOrder = event.row;
        this.loadOrderDetails(event.row.id);
    }

    async loadOrderDetails(orderId: number) {
        const details = await this.orderService.getDetails(orderId);
        this.selectedOrder.details = details;
    }
}
```

### **2. Inline Editing**
```html
<app-data-table-column property="quantity" header="Quantity">
    <ng-template #cellTemplate let-row let-editing="editing">
        <ion-input 
            *ngIf="editing; else displayTemplate"
            [(ngModel)]="row.quantity"
            type="number"
            (ionBlur)="saveQuantity(row)">
        </ion-input>
        
        <ng-template #displayTemplate>
            <span (click)="startEdit(row)">{{ row.quantity }}</span>
        </ng-template>
    </ng-template>
</app-data-table-column>
```

### **3. Conditional Formatting**
```html
<app-data-table-column property="amount" header="Amount">
    <ng-template #cellTemplate let-row>
        <span [class]="getAmountClass(row.amount)">
            {{ row.amount | currency }}
        </span>
    </ng-template>
</app-data-table-column>
```

```typescript
getAmountClass(amount: number): string {
    if (amount < 0) return 'text-danger';
    if (amount > 1000) return 'text-success';
    return 'text-dark';
}
```

---

## 🔧 **Best Practices**

### **1. Column Configuration**
```typescript
// ✅ Đúng - Detailed column config
columns: TableColumn[] = [
    {
        property: 'name',
        header: 'CUSTOMER_NAME',  // Translation key
        canSort: true,
        canFilter: true,
        width: '200px',
        required: true
    }
];

// ❌ Sai - Minimal config
columns = [{ property: 'name' }];
```

### **2. Performance**
```typescript
// ✅ Đúng - Use trackBy
<app-data-table [trackBy]="'id'" [rows]="items">

// ❌ Sai - No tracking
<app-data-table [rows]="items">
```

### **3. Error Handling**
```typescript
// ✅ Đúng - Handle errors
onFilter(event: any) {
    try {
        this.loading = true;
        await this.loadData(event.query);
    } catch (error) {
        this.env.showErrorMessage(error);
    } finally {
        this.loading = false;
    }
}
```

---

## 📱 **Mobile Considerations**

### **Responsive Columns**
```typescript
columns: TableColumn[] = [
    { property: 'name', header: 'Name', priority: 1 },
    { property: 'email', header: 'Email', priority: 2, hideOnMobile: true },
    { property: 'phone', header: 'Phone', priority: 3, hideOnTablet: true }
];
```

### **Touch Interactions**
```html
<app-data-table
    [rows]="items"
    [columns]="columns"
    (activate)="onRowTap($event)"
    (longPress)="onRowLongPress($event)">
</app-data-table>
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
