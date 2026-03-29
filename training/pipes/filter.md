# Filter Pipe Documentation

## 📋 **Tổng quan**

`Filter` pipe được sử dụng để filter arrays dựa trên multiple conditions. Pipe này so sánh exact values và hỗ trợ filtering theo nhiều fields cùng lúc.

**Pipe Name**: `filter`  
**Location**: `src/app/pipes/filter.ts`  
**Type**: Impure Pipe (pure: false)

---

## 🔧 **Signature**

```typescript
transform(items: Array<any>, conditions: { [field: string]: any }): Array<any>
```

### **Parameters**
- `items` (Array<any>): Array cần filter
- `conditions` (Object): Object chứa field names và values để filter

### **Returns**
- `Array<any>`: Filtered array chỉ chứa items match với conditions

---

## 🚀 **Basic Usage**

### **Single Field Filter**
```typescript
// Component
export class ProductListPage extends PageBase {
    products = [
        { id: 1, name: 'Laptop', category: 'Electronics', status: 'active', price: 1000 },
        { id: 2, name: 'Phone', category: 'Electronics', status: 'inactive', price: 500 },
        { id: 3, name: 'Book', category: 'Education', status: 'active', price: 20 },
        { id: 4, name: 'Pen', category: 'Stationery', status: 'active', price: 5 }
    ];
    
    filterConditions = {
        status: 'active'  // Chỉ hiển thị products có status = 'active'
    };
}
```

```html
<!-- Template -->
<div class="product-list">
    <div class="product-item" *ngFor="let product of products | filter: filterConditions">
        <h3>{{ product.name }}</h3>
        <p>Category: {{ product.category }}</p>
        <p>Status: {{ product.status }}</p>
        <p>Price: {{ product.price | currency }}</p>
    </div>
</div>
```

### **Multiple Fields Filter**
```typescript
export class OrderListPage extends PageBase {
    orders = [
        { id: 1, customer: 'John', status: 'completed', type: 'online', amount: 100 },
        { id: 2, customer: 'Jane', status: 'pending', type: 'online', amount: 200 },
        { id: 3, customer: 'Bob', status: 'completed', type: 'offline', amount: 150 },
        { id: 4, customer: 'Alice', status: 'cancelled', type: 'online', amount: 75 }
    ];
    
    // Filter theo multiple conditions
    multipleConditions = {
        status: 'completed',
        type: 'online'
    };
    
    // Dynamic filter conditions
    currentFilters = {
        status: 'all',      // 'all' sẽ bị ignore
        type: 'online',     // Chỉ filter theo type
        customer: ''        // Empty string sẽ bị ignore
    };
}
```

```html
<div class="order-filters">
    <ion-select [(ngModel)]="currentFilters.status" placeholder="Select Status">
        <ion-select-option value="all">All Status</ion-select-option>
        <ion-select-option value="pending">Pending</ion-select-option>
        <ion-select-option value="completed">Completed</ion-select-option>
        <ion-select-option value="cancelled">Cancelled</ion-select-option>
    </ion-select>
    
    <ion-select [(ngModel)]="currentFilters.type" placeholder="Select Type">
        <ion-select-option value="all">All Types</ion-select-option>
        <ion-select-option value="online">Online</ion-select-option>
        <ion-select-option value="offline">Offline</ion-select-option>
    </ion-select>
</div>

<div class="order-list">
    <div class="order-item" *ngFor="let order of orders | filter: currentFilters">
        <h4>Order #{{ order.id }} - {{ order.customer }}</h4>
        <p>Status: {{ order.status }} | Type: {{ order.type }}</p>
        <p>Amount: {{ order.amount | currency }}</p>
    </div>
</div>
```

---

## 🎨 **Advanced Usage**

### **Dynamic Filter Builder**
```typescript
export class AdvancedFilterPage extends PageBase {
    employees = [
        { id: 1, name: 'John Doe', department: 'IT', position: 'Developer', salary: 5000, isActive: true },
        { id: 2, name: 'Jane Smith', department: 'HR', position: 'Manager', salary: 6000, isActive: true },
        { id: 3, name: 'Bob Johnson', department: 'IT', position: 'Tester', salary: 4000, isActive: false },
        { id: 4, name: 'Alice Brown', department: 'Finance', position: 'Analyst', salary: 4500, isActive: true }
    ];
    
    filterConfig = {
        department: { value: 'all', enabled: true },
        position: { value: 'all', enabled: false },
        isActive: { value: true, enabled: true }
    };
    
    get activeFilters(): any {
        const filters = {};
        
        Object.keys(this.filterConfig).forEach(key => {
            const config = this.filterConfig[key];
            if (config.enabled && config.value !== 'all' && config.value !== '') {
                filters[key] = config.value;
            }
        });
        
        return filters;
    }
    
    toggleFilter(filterName: string) {
        this.filterConfig[filterName].enabled = !this.filterConfig[filterName].enabled;
    }
    
    resetFilters() {
        Object.keys(this.filterConfig).forEach(key => {
            this.filterConfig[key].value = 'all';
            this.filterConfig[key].enabled = false;
        });
    }
    
    getFilteredCount(): number {
        return this.employees.filter(emp => {
            for (let field in this.activeFilters) {
                if (emp[field] !== this.activeFilters[field]) {
                    return false;
                }
            }
            return true;
        }).length;
    }
}
```

```html
<div class="advanced-filter">
    <div class="filter-controls">
        <h3>Filter Options</h3>
        
        <!-- Department Filter -->
        <ion-item>
            <ion-checkbox 
                [(ngModel)]="filterConfig.department.enabled"
                slot="start">
            </ion-checkbox>
            <ion-label>Department</ion-label>
            <ion-select 
                [(ngModel)]="filterConfig.department.value"
                [disabled]="!filterConfig.department.enabled"
                slot="end">
                <ion-select-option value="all">All</ion-select-option>
                <ion-select-option value="IT">IT</ion-select-option>
                <ion-select-option value="HR">HR</ion-select-option>
                <ion-select-option value="Finance">Finance</ion-select-option>
            </ion-select>
        </ion-item>
        
        <!-- Position Filter -->
        <ion-item>
            <ion-checkbox 
                [(ngModel)]="filterConfig.position.enabled"
                slot="start">
            </ion-checkbox>
            <ion-label>Position</ion-label>
            <ion-select 
                [(ngModel)]="filterConfig.position.value"
                [disabled]="!filterConfig.position.enabled"
                slot="end">
                <ion-select-option value="all">All</ion-select-option>
                <ion-select-option value="Developer">Developer</ion-select-option>
                <ion-select-option value="Manager">Manager</ion-select-option>
                <ion-select-option value="Tester">Tester</ion-select-option>
                <ion-select-option value="Analyst">Analyst</ion-select-option>
            </ion-select>
        </ion-item>
        
        <!-- Active Status Filter -->
        <ion-item>
            <ion-checkbox 
                [(ngModel)]="filterConfig.isActive.enabled"
                slot="start">
            </ion-checkbox>
            <ion-label>Active Only</ion-label>
            <ion-toggle 
                [(ngModel)]="filterConfig.isActive.value"
                [disabled]="!filterConfig.isActive.enabled"
                slot="end">
            </ion-toggle>
        </ion-item>
        
        <div class="filter-actions">
            <ion-button (click)="resetFilters()" fill="outline">
                Reset Filters
            </ion-button>
            <span class="filter-count">
                Showing {{ getFilteredCount() }} of {{ employees.length }} employees
            </span>
        </div>
    </div>
    
    <div class="employee-list">
        <div class="employee-card" *ngFor="let employee of employees | filter: activeFilters">
            <h4>{{ employee.name }}</h4>
            <p>{{ employee.department }} - {{ employee.position }}</p>
            <p>Salary: {{ employee.salary | currency }}</p>
            <ion-badge [color]="employee.isActive ? 'success' : 'medium'">
                {{ employee.isActive ? 'Active' : 'Inactive' }}
            </ion-badge>
        </div>
    </div>
</div>
```

### **Filter with Data Table**
```typescript
export class DataTableFilterPage extends PageBase {
    tableData = [
        { id: 1, product: 'Laptop', brand: 'Dell', category: 'Electronics', inStock: true, rating: 4.5 },
        { id: 2, product: 'Mouse', brand: 'Logitech', category: 'Electronics', inStock: false, rating: 4.0 },
        { id: 3, product: 'Keyboard', brand: 'Corsair', category: 'Electronics', inStock: true, rating: 4.8 },
        { id: 4, product: 'Monitor', brand: 'Samsung', category: 'Electronics', inStock: true, rating: 4.3 }
    ];
    
    tableFilters = {
        category: 'all',
        brand: 'all',
        inStock: 'all'
    };
    
    columns = [
        { prop: 'product', name: 'Product' },
        { prop: 'brand', name: 'Brand' },
        { prop: 'category', name: 'Category' },
        { prop: 'inStock', name: 'In Stock' },
        { prop: 'rating', name: 'Rating' }
    ];
    
    get filteredData() {
        return this.tableData.filter(item => {
            // Convert filter conditions for pipe
            const conditions = {};
            
            if (this.tableFilters.category !== 'all') {
                conditions['category'] = this.tableFilters.category;
            }
            
            if (this.tableFilters.brand !== 'all') {
                conditions['brand'] = this.tableFilters.brand;
            }
            
            if (this.tableFilters.inStock !== 'all') {
                conditions['inStock'] = this.tableFilters.inStock === 'true';
            }
            
            // Apply filter logic manually or use pipe
            for (let field in conditions) {
                if (item[field] !== conditions[field]) {
                    return false;
                }
            }
            return true;
        });
    }
    
    clearFilters() {
        this.tableFilters = {
            category: 'all',
            brand: 'all',
            inStock: 'all'
        };
    }
}
```

```html
<div class="data-table-with-filters">
    <div class="table-filters">
        <ion-row>
            <ion-col size="4">
                <ion-select [(ngModel)]="tableFilters.category" placeholder="Category">
                    <ion-select-option value="all">All Categories</ion-select-option>
                    <ion-select-option value="Electronics">Electronics</ion-select-option>
                    <ion-select-option value="Clothing">Clothing</ion-select-option>
                    <ion-select-option value="Books">Books</ion-select-option>
                </ion-select>
            </ion-col>
            <ion-col size="4">
                <ion-select [(ngModel)]="tableFilters.brand" placeholder="Brand">
                    <ion-select-option value="all">All Brands</ion-select-option>
                    <ion-select-option value="Dell">Dell</ion-select-option>
                    <ion-select-option value="Logitech">Logitech</ion-select-option>
                    <ion-select-option value="Corsair">Corsair</ion-select-option>
                    <ion-select-option value="Samsung">Samsung</ion-select-option>
                </ion-select>
            </ion-col>
            <ion-col size="4">
                <ion-select [(ngModel)]="tableFilters.inStock" placeholder="Stock Status">
                    <ion-select-option value="all">All Items</ion-select-option>
                    <ion-select-option value="true">In Stock</ion-select-option>
                    <ion-select-option value="false">Out of Stock</ion-select-option>
                </ion-select>
            </ion-col>
        </ion-row>
        
        <ion-button (click)="clearFilters()" fill="outline" size="small">
            Clear Filters
        </ion-button>
    </div>
    
    <!-- Using pipe directly in template -->
    <app-data-table 
        [rows]="tableData | filter: tableFilters"
        [columns]="columns">
    </app-data-table>
    
    <!-- Or using computed property -->
    <app-data-table 
        [rows]="filteredData"
        [columns]="columns">
    </app-data-table>
</div>
```

---

## ⚡ **Performance Considerations**

### **Impure Pipe Impact**
```typescript
// Filter pipe is impure (pure: false), which means:
// - It runs on every change detection cycle
// - Can impact performance with large datasets
// - Consider alternatives for better performance

export class PerformanceOptimizedPage extends PageBase {
    largeDataset: any[] = []; // 1000+ items
    filteredItems: any[] = [];
    currentFilters = {};
    
    // ✅ Better approach: Filter in component
    applyFilters() {
        this.filteredItems = this.largeDataset.filter(item => {
            for (let field in this.currentFilters) {
                if (this.currentFilters[field] === 'all' || this.currentFilters[field] === '') {
                    continue;
                }
                if (item[field] !== this.currentFilters[field]) {
                    return false;
                }
            }
            return true;
        });
    }
    
    onFilterChange() {
        // Debounce filter application
        clearTimeout(this.filterTimeout);
        this.filterTimeout = setTimeout(() => {
            this.applyFilters();
        }, 300);
    }
}
```

---

## 🔧 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Đúng - Use component filtering for large datasets
filterInComponent() {
    this.filteredItems = this.items.filter(item => 
        this.matchesFilters(item, this.filters)
    );
}

// ⚠️ Cẩn thận - Pipe filtering with large datasets
// <div *ngFor="let item of largeArray | filter: conditions">
```

### **2. Filter Condition Management**
```typescript
// ✅ Đúng - Clean filter conditions
get cleanFilters() {
    const filters = {};
    Object.keys(this.rawFilters).forEach(key => {
        const value = this.rawFilters[key];
        if (value !== 'all' && value !== '' && value !== null && value !== undefined) {
            filters[key] = value;
        }
    });
    return filters;
}
```

### **3. Type Safety**
```typescript
// ✅ Đúng - Type-safe filter conditions
interface FilterConditions {
    status?: string;
    category?: string;
    isActive?: boolean;
}

applyTypedFilters(conditions: FilterConditions) {
    return this.items.filter(item => {
        return Object.keys(conditions).every(key => 
            item[key] === conditions[key]
        );
    });
}
```

---

## 🚨 **Common Use Cases**

### **1. Product Catalog Filtering**
```typescript
export class ProductCatalogPage extends PageBase {
    products: Product[] = [];
    filters = {
        category: 'all',
        priceRange: 'all',
        availability: 'all'
    };
}
```

### **2. Employee Directory**
```typescript
export class EmployeeDirectoryPage extends PageBase {
    employees: Employee[] = [];
    filters = {
        department: 'all',
        position: 'all',
        status: 'active'
    };
}
```

### **3. Order Management**
```typescript
export class OrderManagementPage extends PageBase {
    orders: Order[] = [];
    filters = {
        status: 'all',
        paymentMethod: 'all',
        dateRange: 'all'
    };
}
```

---

## ⚠️ **Limitations & Alternatives**

### **Limitations**
1. **Performance**: Impure pipe runs on every change detection
2. **Exact Match Only**: Không support partial matching
3. **No Complex Logic**: Chỉ support simple equality checks

### **Alternatives**
```typescript
// For complex filtering, use component methods
complexFilter(items: any[], conditions: any): any[] {
    return items.filter(item => {
        // Custom complex logic here
        return this.customFilterLogic(item, conditions);
    });
}

// For search functionality, use search pipe instead
// For partial matching, use searchNoAccent pipe
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
