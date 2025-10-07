# IsNotDeleted Pipe Documentation

## 📋 **Tổng quan**

`IsNotDeleted` pipe được sử dụng để filter arrays, chỉ hiển thị các items chưa bị xóa (có `IsDeleted === false`). Pipe này thường được sử dụng trong soft delete systems.

**Pipe Name**: `isNotDeleted`  
**Location**: `src/app/pipes/filter.ts`  
**Type**: Pure Pipe

---

## 🔧 **Signature**

```typescript
transform(items: any[]): any[]
```

### **Parameters**
- `items` (Array<any>): Array cần filter

### **Returns**
- `Array<any>`: Filtered array chỉ chứa items có `IsDeleted === false`

---

## 🚀 **Basic Usage**

### **Simple Filtering**
```typescript
// Component
export class ActiveItemsPage extends PageBase {
    allItems = [
        { id: 1, name: 'Active Item 1', IsDeleted: false },
        { id: 2, name: 'Deleted Item', IsDeleted: true },
        { id: 3, name: 'Active Item 2', IsDeleted: false },
        { id: 4, name: 'Another Deleted', IsDeleted: true }
    ];
}
```

```html
<!-- Template -->
<div class="active-items">
    <h3>Active Items Only</h3>
    <div class="item-list">
        <div class="item" *ngFor="let item of allItems | isNotDeleted">
            <h4>{{ item.name }}</h4>
            <p>ID: {{ item.id }}</p>
        </div>
    </div>
</div>
```

### **With Data Table**
```typescript
export class ProductManagementPage extends PageBase {
    products = [
        { id: 1, name: 'Laptop', price: 1000, IsDeleted: false, status: 'active' },
        { id: 2, name: 'Mouse', price: 25, IsDeleted: true, status: 'deleted' },
        { id: 3, name: 'Keyboard', price: 75, IsDeleted: false, status: 'active' },
        { id: 4, name: 'Monitor', price: 300, IsDeleted: true, status: 'deleted' }
    ];
    
    showDeletedItems = false;
    
    get displayedProducts() {
        return this.showDeletedItems ? this.products : this.products.filter(p => !p.IsDeleted);
    }
}
```

```html
<div class="product-management">
    <div class="controls">
        <ion-toggle 
            [(ngModel)]="showDeletedItems"
            color="danger">
        </ion-toggle>
        <ion-label>Show Deleted Items</ion-label>
    </div>
    
    <!-- Using pipe when not showing deleted items -->
    <app-data-table 
        *ngIf="!showDeletedItems"
        [rows]="products | isNotDeleted"
        [columns]="columns">
    </app-data-table>
    
    <!-- Show all items when toggle is on -->
    <app-data-table 
        *ngIf="showDeletedItems"
        [rows]="products"
        [columns]="columns">
    </app-data-table>
</div>
```

---

## 🎨 **Advanced Usage**

### **Combined with Other Filters**
```typescript
export class CombinedFiltersPage extends PageBase {
    orders = [
        { id: 1, customer: 'John', status: 'completed', IsDeleted: false },
        { id: 2, customer: 'Jane', status: 'pending', IsDeleted: false },
        { id: 3, customer: 'Bob', status: 'cancelled', IsDeleted: true },
        { id: 4, customer: 'Alice', status: 'completed', IsDeleted: false }
    ];
    
    statusFilter = 'all';
    
    get filteredOrders() {
        let filtered = this.orders.filter(order => !order.IsDeleted);
        
        if (this.statusFilter !== 'all') {
            filtered = filtered.filter(order => order.status === this.statusFilter);
        }
        
        return filtered;
    }
}
```

```html
<div class="order-filters">
    <ion-select [(ngModel)]="statusFilter" placeholder="Filter by Status">
        <ion-select-option value="all">All Active Orders</ion-select-option>
        <ion-select-option value="pending">Pending</ion-select-option>
        <ion-select-option value="completed">Completed</ion-select-option>
        <ion-select-option value="cancelled">Cancelled</ion-select-option>
    </ion-select>
</div>

<!-- Method 1: Using pipe in template -->
<div class="orders-pipe">
    <h3>Using Pipe (Active Orders Only)</h3>
    <div *ngFor="let order of orders | isNotDeleted">
        <p>{{ order.customer }} - {{ order.status }}</p>
    </div>
</div>

<!-- Method 2: Using computed property -->
<div class="orders-computed">
    <h3>Using Computed Property (With Status Filter)</h3>
    <div *ngFor="let order of filteredOrders">
        <p>{{ order.customer }} - {{ order.status }}</p>
    </div>
</div>
```

### **Soft Delete Management**
```typescript
export class SoftDeleteManagementPage extends PageBase {
    items = [];
    viewMode: 'active' | 'deleted' | 'all' = 'active';
    
    get displayItems() {
        switch (this.viewMode) {
            case 'active':
                return this.items.filter(item => !item.IsDeleted);
            case 'deleted':
                return this.items.filter(item => item.IsDeleted);
            case 'all':
            default:
                return this.items;
        }
    }
    
    softDelete(item: any) {
        item.IsDeleted = true;
        item.DeletedDate = new Date();
        item.DeletedBy = this.env.user.Id;
        
        // Update in backend
        this.itemService.softDelete(item.Id).then(() => {
            this.env.showMessage('Item moved to trash', 'success');
        });
    }
    
    restore(item: any) {
        item.IsDeleted = false;
        item.DeletedDate = null;
        item.DeletedBy = null;
        
        // Update in backend
        this.itemService.restore(item.Id).then(() => {
            this.env.showMessage('Item restored', 'success');
        });
    }
    
    permanentDelete(item: any) {
        this.env.actionConfirm('permanently delete', 1, 'item', 'Warning', 
            () => this.itemService.permanentDelete(item.Id)
        ).then(() => {
            const index = this.items.indexOf(item);
            if (index > -1) {
                this.items.splice(index, 1);
            }
            this.env.showMessage('Item permanently deleted', 'success');
        });
    }
}
```

```html
<div class="soft-delete-management">
    <div class="view-controls">
        <ion-segment [(ngModel)]="viewMode">
            <ion-segment-button value="active">
                <ion-label>Active Items</ion-label>
            </ion-segment-button>
            <ion-segment-button value="deleted">
                <ion-label>Deleted Items</ion-label>
            </ion-segment-button>
            <ion-segment-button value="all">
                <ion-label>All Items</ion-label>
            </ion-segment-button>
        </ion-segment>
    </div>
    
    <div class="item-list">
        <!-- Active items with pipe -->
        <div *ngIf="viewMode === 'active'">
            <div class="item-card" *ngFor="let item of items | isNotDeleted">
                <div class="item-content">
                    <h4>{{ item.name }}</h4>
                    <p>{{ item.description }}</p>
                </div>
                <div class="item-actions">
                    <ion-button (click)="softDelete(item)" color="danger" fill="outline">
                        <ion-icon name="trash" slot="start"></ion-icon>
                        Delete
                    </ion-button>
                </div>
            </div>
        </div>
        
        <!-- Deleted items -->
        <div *ngIf="viewMode === 'deleted'">
            <div class="item-card deleted" *ngFor="let item of items" [hidden]="!item.IsDeleted">
                <div class="item-content">
                    <h4>{{ item.name }}</h4>
                    <p>{{ item.description }}</p>
                    <small>Deleted: {{ item.DeletedDate | date }}</small>
                </div>
                <div class="item-actions">
                    <ion-button (click)="restore(item)" color="success" fill="outline">
                        <ion-icon name="refresh" slot="start"></ion-icon>
                        Restore
                    </ion-button>
                    <ion-button (click)="permanentDelete(item)" color="danger">
                        <ion-icon name="trash" slot="start"></ion-icon>
                        Permanent Delete
                    </ion-button>
                </div>
            </div>
        </div>
        
        <!-- All items -->
        <div *ngIf="viewMode === 'all'">
            <div class="item-card" 
                 *ngFor="let item of items" 
                 [class.deleted]="item.IsDeleted">
                <div class="item-content">
                    <h4>{{ item.name }}</h4>
                    <p>{{ item.description }}</p>
                    <ion-badge [color]="item.IsDeleted ? 'danger' : 'success'">
                        {{ item.IsDeleted ? 'Deleted' : 'Active' }}
                    </ion-badge>
                </div>
            </div>
        </div>
    </div>
</div>
```

---

## 🔧 **Best Practices**

### **1. Performance Considerations**
```typescript
// ✅ Đúng - Use pipe for simple filtering
get activeItems() {
    return this.items | isNotDeleted; // In template
}

// ✅ Đúng - Use component method for complex logic
get filteredItems() {
    return this.items
        .filter(item => !item.IsDeleted)
        .filter(item => this.matchesOtherCriteria(item));
}
```

### **2. Consistent Data Structure**
```typescript
// ✅ Đúng - Ensure all items have IsDeleted property
interface BaseEntity {
    Id: number;
    IsDeleted: boolean;
    DeletedDate?: Date;
    DeletedBy?: number;
}

// ❌ Sai - Inconsistent data structure
// Some items might not have IsDeleted property
```

### **3. Soft Delete Implementation**
```typescript
// ✅ Đúng - Proper soft delete
async softDeleteItem(id: number) {
    await this.service.update(id, {
        IsDeleted: true,
        DeletedDate: new Date(),
        DeletedBy: this.env.user.Id
    });
}

// ❌ Sai - Hard delete when expecting soft delete
async deleteItem(id: number) {
    await this.service.delete(id); // Permanently removes from DB
}
```

---

## 🚨 **Common Use Cases**

### **1. Product Catalog**
```typescript
export class ProductCatalogPage extends PageBase {
    products: Product[] = [];
    
    // Show only active products to customers
    get activeProducts() {
        return this.products.filter(p => !p.IsDeleted);
    }
}
```

### **2. User Management**
```typescript
export class UserManagementPage extends PageBase {
    users: User[] = [];
    
    // Admin can see all users, regular users see only active
    get visibleUsers() {
        return this.env.user.isAdmin 
            ? this.users 
            : this.users.filter(u => !u.IsDeleted);
    }
}
```

### **3. Document Management**
```typescript
export class DocumentManagementPage extends PageBase {
    documents: Document[] = [];
    showTrash = false;
    
    get displayedDocuments() {
        return this.showTrash 
            ? this.documents.filter(d => d.IsDeleted)
            : this.documents.filter(d => !d.IsDeleted);
    }
}
```

---

## 🎨 **Styling**

### **Visual Distinction for Deleted Items**
```scss
.item-card {
    padding: 16px;
    margin: 8px 0;
    border-radius: 8px;
    border: 1px solid var(--ion-color-medium);
    
    &.deleted {
        opacity: 0.6;
        background-color: var(--ion-color-light-shade);
        border-color: var(--ion-color-danger);
        
        .item-content {
            text-decoration: line-through;
            color: var(--ion-color-medium);
        }
    }
    
    &:not(.deleted) {
        background-color: white;
        
        &:hover {
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        }
    }
}

.view-controls {
    margin-bottom: 16px;
    
    ion-segment-button {
        --indicator-color: var(--ion-color-primary);
    }
}
```

---

## ⚠️ **Limitations**

1. **Fixed Property Name**: Chỉ check property `IsDeleted`, không configurable
2. **Boolean Only**: Chỉ support `IsDeleted === false`, không support other values
3. **No Parameters**: Không thể customize deletion criteria
4. **Pure Pipe**: Không detect changes trong objects, chỉ detect array reference changes

---

## 🔄 **Alternative Implementations**

### **Custom Configurable Filter**
```typescript
// For more flexibility, create custom filter
@Pipe({ name: 'activeItems' })
export class ActiveItemsPipe implements PipeTransform {
    transform(items: any[], deletedField = 'IsDeleted'): any[] {
        return items.filter(item => !item[deletedField]);
    }
}

// Usage: items | activeItems:'IsArchived'
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
