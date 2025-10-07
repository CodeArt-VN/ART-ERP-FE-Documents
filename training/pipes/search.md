# Search Pipe Documentation

## 📋 **Tổng quan**

`Search` pipe được sử dụng để search arrays với string matching. Pipe này thực hiện case-insensitive search và hỗ trợ partial matching trong text fields.

**Pipe Name**: `search`  
**Location**: `src/app/pipes/filter.ts`  
**Type**: Impure Pipe (pure: false)

---

## 🔧 **Signature**

```typescript
transform(items: Array<any>, conditions: { [field: string]: any }): Array<any>
```

### **Parameters**
- `items` (Array<any>): Array cần search
- `conditions` (Object): Object chứa field names và search terms

### **Returns**
- `Array<any>`: Filtered array chứa items match với search terms

---

## 🚀 **Basic Usage**

### **Simple Text Search**
```typescript
// Component
export class CustomerSearchPage extends PageBase {
    customers = [
        { id: 1, name: 'John Doe', email: 'john@example.com', company: 'Tech Corp' },
        { id: 2, name: 'Jane Smith', email: 'jane@company.com', company: 'Design Studio' },
        { id: 3, name: 'Bob Johnson', email: 'bob@tech.com', company: 'Tech Corp' },
        { id: 4, name: 'Alice Brown', email: 'alice@design.com', company: 'Creative Agency' }
    ];
    
    searchConditions = {
        name: 'john'  // Tìm customers có name chứa 'john' (case-insensitive)
    };
    
    // Dynamic search
    searchTerm = '';
    
    get dynamicSearch() {
        return {
            name: this.searchTerm
        };
    }
}
```

```html
<!-- Template -->
<div class="search-container">
    <!-- Search input -->
    <ion-searchbar 
        [(ngModel)]="searchTerm"
        placeholder="Search customers..."
        debounce="300">
    </ion-searchbar>
    
    <!-- Results -->
    <div class="customer-list">
        <div class="customer-item" *ngFor="let customer of customers | search: dynamicSearch">
            <h3>{{ customer.name }}</h3>
            <p>{{ customer.email }}</p>
            <p>{{ customer.company }}</p>
        </div>
    </div>
</div>
```

### **Multiple Field Search**
```typescript
export class ProductSearchPage extends PageBase {
    products = [
        { id: 1, name: 'MacBook Pro', description: 'Powerful laptop for professionals', brand: 'Apple' },
        { id: 2, name: 'iPhone 15', description: 'Latest smartphone with advanced features', brand: 'Apple' },
        { id: 3, name: 'Galaxy S24', description: 'Android phone with great camera', brand: 'Samsung' },
        { id: 4, name: 'ThinkPad X1', description: 'Business laptop with durability', brand: 'Lenovo' }
    ];
    
    searchFields = {
        name: '',
        description: '',
        brand: ''
    };
    
    // Search across multiple fields
    globalSearch = '';
    
    get globalSearchConditions() {
        return {
            name: this.globalSearch,
            description: this.globalSearch,
            brand: this.globalSearch
        };
    }
}
```

```html
<div class="product-search">
    <!-- Individual field search -->
    <div class="search-filters">
        <ion-item>
            <ion-label>Product Name</ion-label>
            <ion-input [(ngModel)]="searchFields.name" placeholder="Search by name"></ion-input>
        </ion-item>
        
        <ion-item>
            <ion-label>Description</ion-label>
            <ion-input [(ngModel)]="searchFields.description" placeholder="Search description"></ion-input>
        </ion-item>
        
        <ion-item>
            <ion-label>Brand</ion-label>
            <ion-input [(ngModel)]="searchFields.brand" placeholder="Search brand"></ion-input>
        </ion-item>
    </div>
    
    <!-- Global search -->
    <ion-searchbar 
        [(ngModel)]="globalSearch"
        placeholder="Search all fields..."
        debounce="300">
    </ion-searchbar>
    
    <!-- Results with individual field search -->
    <div class="results-section">
        <h3>Filtered Results</h3>
        <div *ngFor="let product of products | search: searchFields">
            <h4>{{ product.name }}</h4>
            <p>{{ product.description }}</p>
            <small>Brand: {{ product.brand }}</small>
        </div>
    </div>
</div>
```

---

## 🎨 **Advanced Usage**

### **Search with Debouncing**
```typescript
export class AdvancedSearchPage extends PageBase {
    searchTerm = '';
    searchResults: any[] = [];
    isSearching = false;
    private searchTimeout: any;
    
    onSearchChange(term: string) {
        this.isSearching = true;
        
        // Clear previous timeout
        if (this.searchTimeout) {
            clearTimeout(this.searchTimeout);
        }
        
        // Debounce search
        this.searchTimeout = setTimeout(() => {
            this.performSearch(term);
        }, 500);
    }
    
    performSearch(term: string) {
        if (!term.trim()) {
            this.searchResults = [];
            this.isSearching = false;
            return;
        }
        
        // Simulate API call or complex search
        setTimeout(() => {
            const searchConditions = {
                name: term,
                description: term,
                tags: term
            };
            
            // Use pipe logic or custom search
            this.searchResults = this.allItems.filter(item => {
                return this.matchesSearchTerm(item, term);
            });
            
            this.isSearching = false;
        }, 200);
    }
    
    private matchesSearchTerm(item: any, term: string): boolean {
        const searchFields = ['name', 'description', 'tags'];
        const lowerTerm = term.toLowerCase();
        
        return searchFields.some(field => {
            const fieldValue = item[field];
            return fieldValue && fieldValue.toLowerCase().includes(lowerTerm);
        });
    }
}
```

```html
<div class="advanced-search">
    <div class="search-header">
        <ion-searchbar 
            [(ngModel)]="searchTerm"
            (ionInput)="onSearchChange($event.detail.value)"
            placeholder="Search items..."
            [showClearButton]="'focus'">
        </ion-searchbar>
        
        <ion-spinner *ngIf="isSearching" name="crescent"></ion-spinner>
    </div>
    
    <div class="search-results">
        <div *ngIf="searchTerm && !isSearching && searchResults.length === 0" class="no-results">
            <ion-icon name="search-outline"></ion-icon>
            <p>No results found for "{{ searchTerm }}"</p>
        </div>
        
        <div class="result-item" *ngFor="let item of searchResults">
            <h4 [innerHTML]="highlightSearchTerm(item.name, searchTerm)"></h4>
            <p [innerHTML]="highlightSearchTerm(item.description, searchTerm)"></p>
        </div>
    </div>
</div>
```

### **Search with Filters Combination**
```typescript
export class CombinedSearchFilterPage extends PageBase {
    items = [
        { name: 'Product A', category: 'Electronics', price: 100, inStock: true },
        { name: 'Product B', category: 'Clothing', price: 50, inStock: false },
        { name: 'Service A', category: 'Electronics', price: 200, inStock: true }
    ];
    
    searchTerm = '';
    filters = {
        category: 'all',
        inStock: 'all'
    };
    
    get combinedResults() {
        let results = this.items;
        
        // Apply search first
        if (this.searchTerm) {
            const searchConditions = { name: this.searchTerm };
            results = results.filter(item => {
                return item.name.toLowerCase().includes(this.searchTerm.toLowerCase());
            });
        }
        
        // Apply filters
        if (this.filters.category !== 'all') {
            results = results.filter(item => item.category === this.filters.category);
        }
        
        if (this.filters.inStock !== 'all') {
            results = results.filter(item => item.inStock === (this.filters.inStock === 'true'));
        }
        
        return results;
    }
}
```

---

## 🔧 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Đúng - Debounce search input
onSearchInput(event: any) {
    clearTimeout(this.searchDebounce);
    this.searchDebounce = setTimeout(() => {
        this.searchTerm = event.detail.value;
    }, 300);
}

// ❌ Sai - Search on every keystroke
onSearchInput(event: any) {
    this.searchTerm = event.detail.value; // Too frequent updates
}
```

### **2. Empty State Handling**
```typescript
// ✅ Đúng - Handle empty search terms
get searchConditions() {
    return this.searchTerm.trim() ? { name: this.searchTerm } : {};
}

// Show all items when search is empty
get displayItems() {
    return this.searchTerm.trim() 
        ? this.items | search: this.searchConditions
        : this.items;
}
```

### **3. Case Sensitivity**
```typescript
// Search pipe is already case-insensitive
// But for custom search logic:
customSearch(items: any[], term: string) {
    const lowerTerm = term.toLowerCase();
    return items.filter(item => 
        item.name.toLowerCase().includes(lowerTerm)
    );
}
```

---

## 🚨 **Common Use Cases**

### **1. Employee Directory Search**
```typescript
export class EmployeeDirectoryPage extends PageBase {
    employees: Employee[] = [];
    searchTerm = '';
    
    get searchConditions() {
        return {
            name: this.searchTerm,
            email: this.searchTerm,
            department: this.searchTerm
        };
    }
}
```

### **2. Product Catalog Search**
```typescript
export class ProductCatalogPage extends PageBase {
    products: Product[] = [];
    searchQuery = '';
    
    get productSearch() {
        return {
            name: this.searchQuery,
            description: this.searchQuery,
            sku: this.searchQuery
        };
    }
}
```

### **3. Document Search**
```typescript
export class DocumentSearchPage extends PageBase {
    documents: Document[] = [];
    searchText = '';
    
    get documentSearch() {
        return {
            title: this.searchText,
            content: this.searchText,
            author: this.searchText
        };
    }
}
```

---

## ⚠️ **Limitations**

1. **Case-insensitive only**: Không support case-sensitive search
2. **Simple substring matching**: Không support regex hoặc fuzzy search
3. **Performance**: Impure pipe có thể impact performance với large datasets
4. **No highlighting**: Không tự động highlight search terms

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
