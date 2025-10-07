# SearchNoAccent Pipe Documentation

## 📋 **Tổng quan**

`SearchNoAccent` pipe được sử dụng để search arrays với Vietnamese text, tự động bỏ qua dấu tiếng Việt. Pipe này đặc biệt hữu ích cho ứng dụng có nội dung tiếng Việt, cho phép user search mà không cần gõ dấu.

**Pipe Name**: `searchNoAccent`  
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
- `Array<any>`: Filtered array chứa items match với search terms (bỏ qua dấu)

---

## 🚀 **Basic Usage**

### **Vietnamese Text Search**
```typescript
// Component
export class VietnameseSearchPage extends PageBase {
    customers = [
        { id: 1, name: 'Nguyễn Văn Anh', city: 'Hà Nội', address: 'Đống Đa' },
        { id: 2, name: 'Trần Thị Bình', city: 'Hồ Chí Minh', address: 'Quận 1' },
        { id: 3, name: 'Lê Văn Cường', city: 'Đà Nẵng', address: 'Hải Châu' },
        { id: 4, name: 'Phạm Thị Dung', city: 'Cần Thơ', address: 'Ninh Kiều' }
    ];
    
    searchTerm = '';
    
    get searchConditions() {
        return {
            name: this.searchTerm
        };
    }
}
```

```html
<!-- Template -->
<div class="vietnamese-search">
    <ion-searchbar 
        [(ngModel)]="searchTerm"
        placeholder="Tìm kiếm khách hàng... (có thể gõ không dấu)"
        debounce="300">
    </ion-searchbar>
    
    <div class="search-examples">
        <p><small>Ví dụ: Gõ "nguyen van anh" sẽ tìm thấy "Nguyễn Văn Anh"</small></p>
    </div>
    
    <div class="customer-list">
        <ion-card *ngFor="let customer of customers | searchNoAccent: searchConditions">
            <ion-card-content>
                <h3>{{ customer.name }}</h3>
                <p>{{ customer.city }} - {{ customer.address }}</p>
            </ion-card-content>
        </ion-card>
    </div>
</div>
```

### **Multi-field Vietnamese Search**
```typescript
export class ProductVietnameseSearchPage extends PageBase {
    products = [
        { 
            id: 1, 
            name: 'Điện thoại thông minh', 
            description: 'Sản phẩm công nghệ cao cấp',
            brand: 'Samsung',
            category: 'Điện tử'
        },
        { 
            id: 2, 
            name: 'Máy tính xách tay', 
            description: 'Laptop cho doanh nhân',
            brand: 'Dell',
            category: 'Máy tính'
        },
        { 
            id: 3, 
            name: 'Tai nghe không dây', 
            description: 'Âm thanh chất lượng cao',
            brand: 'Sony',
            category: 'Phụ kiện'
        }
    ];
    
    searchQuery = '';
    
    get multiFieldSearch() {
        return {
            name: this.searchQuery,
            description: this.searchQuery,
            category: this.searchQuery
        };
    }
}
```

```html
<div class="product-search">
    <ion-searchbar 
        [(ngModel)]="searchQuery"
        placeholder="Tìm sản phẩm (tên, mô tả, danh mục)..."
        debounce="300">
    </ion-searchbar>
    
    <div class="search-tips">
        <ion-chip color="primary" outline="true">
            <ion-label>Tip: Gõ "dien thoai" → "Điện thoại"</ion-label>
        </ion-chip>
        <ion-chip color="secondary" outline="true">
            <ion-label>Tip: Gõ "may tinh" → "Máy tính"</ion-label>
        </ion-chip>
    </div>
    
    <div class="product-grid">
        <ion-card *ngFor="let product of products | searchNoAccent: multiFieldSearch">
            <ion-card-header>
                <ion-card-title>{{ product.name }}</ion-card-title>
                <ion-card-subtitle>{{ product.brand }} - {{ product.category }}</ion-card-subtitle>
            </ion-card-header>
            <ion-card-content>
                {{ product.description }}
            </ion-card-content>
        </ion-card>
    </div>
</div>
```

---

## 🎨 **Advanced Usage**

### **Smart Search with Suggestions**
```typescript
export class SmartVietnameseSearchPage extends PageBase {
    locations = [
        { name: 'Hà Nội', region: 'Miền Bắc', districts: ['Ba Đình', 'Hoàn Kiếm', 'Đống Đa'] },
        { name: 'Hồ Chí Minh', region: 'Miền Nam', districts: ['Quận 1', 'Quận 3', 'Thủ Đức'] },
        { name: 'Đà Nẵng', region: 'Miền Trung', districts: ['Hải Châu', 'Thanh Khê', 'Sơn Trà'] },
        { name: 'Cần Thơ', region: 'Đồng bằng sông Cửu Long', districts: ['Ninh Kiều', 'Bình Thủy'] }
    ];
    
    searchTerm = '';
    searchSuggestions: string[] = [];
    showSuggestions = false;
    
    onSearchInput(value: string) {
        this.searchTerm = value;
        this.generateSuggestions(value);
    }
    
    generateSuggestions(term: string) {
        if (!term.trim()) {
            this.searchSuggestions = [];
            this.showSuggestions = false;
            return;
        }
        
        const suggestions = new Set<string>();
        
        // Generate suggestions from location names
        this.locations.forEach(location => {
            if (this.matchesNoAccent(location.name, term)) {
                suggestions.add(location.name);
            }
            
            // Add district suggestions
            location.districts.forEach(district => {
                if (this.matchesNoAccent(district, term)) {
                    suggestions.add(`${district}, ${location.name}`);
                }
            });
        });
        
        this.searchSuggestions = Array.from(suggestions).slice(0, 5);
        this.showSuggestions = this.searchSuggestions.length > 0;
    }
    
    selectSuggestion(suggestion: string) {
        this.searchTerm = suggestion;
        this.showSuggestions = false;
    }
    
    private matchesNoAccent(text: string, term: string): boolean {
        const normalizedText = this.removeAccents(text.toLowerCase());
        const normalizedTerm = this.removeAccents(term.toLowerCase());
        return normalizedText.includes(normalizedTerm);
    }
    
    private removeAccents(str: string): string {
        // Same logic as in the pipe
        str = str.toLowerCase();
        str = str.replace(/à|á|ạ|ả|ã|â|ầ|ấ|ậ|ẩ|ẫ|ă|ằ|ắ|ặ|ẳ|ẵ/g, 'a');
        str = str.replace(/è|é|ẹ|ẻ|ẽ|ê|ề|ế|ệ|ể|ễ/g, 'e');
        str = str.replace(/ì|í|ị|ỉ|ĩ/g, 'i');
        str = str.replace(/ò|ó|ọ|ỏ|õ|ô|ồ|ố|ộ|ổ|ỗ|ơ|ờ|ớ|ợ|ở|ỡ/g, 'o');
        str = str.replace(/ù|ú|ụ|ủ|ũ|ư|ừ|ứ|ự|ử|ữ/g, 'u');
        str = str.replace(/ỳ|ý|ỵ|ỷ|ỹ/g, 'y');
        str = str.replace(/đ/g, 'd');
        str = str.replace(/\u0300|\u0301|\u0303|\u0309|\u0323/g, '');
        str = str.replace(/\u02C6|\u0306|\u031B/g, '');
        return str;
    }
}
```

```html
<div class="smart-search">
    <div class="search-container">
        <ion-searchbar 
            [(ngModel)]="searchTerm"
            (ionInput)="onSearchInput($event.detail.value)"
            placeholder="Tìm địa điểm..."
            debounce="200">
        </ion-searchbar>
        
        <!-- Search suggestions -->
        <div class="suggestions" *ngIf="showSuggestions">
            <ion-item 
                button 
                *ngFor="let suggestion of searchSuggestions"
                (click)="selectSuggestion(suggestion)">
                <ion-icon name="location-outline" slot="start"></ion-icon>
                <ion-label>{{ suggestion }}</ion-label>
            </ion-item>
        </div>
    </div>
    
    <!-- Search results -->
    <div class="location-results">
        <ion-card *ngFor="let location of locations | searchNoAccent: { name: searchTerm }">
            <ion-card-header>
                <ion-card-title>{{ location.name }}</ion-card-title>
                <ion-card-subtitle>{{ location.region }}</ion-card-subtitle>
            </ion-card-header>
            <ion-card-content>
                <div class="districts">
                    <ion-chip *ngFor="let district of location.districts" color="primary" outline="true">
                        {{ district }}
                    </ion-chip>
                </div>
            </ion-card-content>
        </ion-card>
    </div>
</div>
```

### **Special "deals" Search Feature**
```typescript
export class ProductDealsSearchPage extends PageBase {
    products = [
        {
            id: 1,
            name: 'Laptop Gaming',
            UoMs: [{
                PriceList: [
                    { Price: 20000000, NewPrice: 18000000 }, // Has deal
                    { Price: 19000000 }
                ]
            }]
        },
        {
            id: 2,
            name: 'Smartphone',
            UoMs: [{
                PriceList: [
                    { Price: 15000000 }, // No deal
                    { Price: 14000000 }
                ]
            }]
        }
    ];
    
    searchTerm = '';
    
    get searchConditions() {
        return {
            name: this.searchTerm
        };
    }
    
    // Special method to find products with deals
    searchDeals() {
        this.searchTerm = 'deals'; // Special keyword
    }
    
    clearSearch() {
        this.searchTerm = '';
    }
}
```

```html
<div class="deals-search">
    <div class="search-controls">
        <ion-searchbar 
            [(ngModel)]="searchTerm"
            placeholder="Tìm sản phẩm hoặc gõ 'deals' để xem khuyến mãi">
        </ion-searchbar>
        
        <div class="quick-actions">
            <ion-button (click)="searchDeals()" fill="outline" color="success">
                <ion-icon name="pricetag" slot="start"></ion-icon>
                Xem Khuyến Mãi
            </ion-button>
            <ion-button (click)="clearSearch()" fill="clear">
                Xóa Tìm Kiếm
            </ion-button>
        </div>
    </div>
    
    <div class="product-results">
        <ion-card *ngFor="let product of products | searchNoAccent: searchConditions">
            <ion-card-header>
                <ion-card-title>{{ product.name }}</ion-card-title>
            </ion-card-header>
            <ion-card-content>
                <div *ngFor="let uom of product.UoMs">
                    <div *ngFor="let price of uom.PriceList">
                        <div *ngIf="price.NewPrice" class="deal-price">
                            <span class="old-price">{{ price.Price | currency:'VND' }}</span>
                            <span class="new-price">{{ price.NewPrice | currency:'VND' }}</span>
                            <ion-badge color="danger">SALE</ion-badge>
                        </div>
                        <div *ngIf="!price.NewPrice" class="regular-price">
                            {{ price.Price | currency:'VND' }}
                        </div>
                    </div>
                </div>
            </ion-card-content>
        </ion-card>
    </div>
</div>
```

---

## 🔧 **Best Practices**

### **1. Search Input Optimization**
```typescript
// ✅ Đúng - Debounce Vietnamese search
onVietnameseSearchInput(value: string) {
    clearTimeout(this.searchTimeout);
    this.searchTimeout = setTimeout(() => {
        this.searchTerm = value.trim();
    }, 300);
}
```

### **2. Search Highlighting**
```typescript
// ✅ Đúng - Highlight search results
highlightVietnameseText(text: string, searchTerm: string): string {
    if (!searchTerm) return text;
    
    const normalizedText = this.removeAccents(text.toLowerCase());
    const normalizedTerm = this.removeAccents(searchTerm.toLowerCase());
    
    // Find matches and highlight original text
    // Implementation depends on your highlighting needs
    return text; // Simplified
}
```

### **3. Performance with Large Datasets**
```typescript
// ✅ Đúng - Limit search results
get limitedSearchResults() {
    const results = this.allItems.filter(item => 
        this.matchesVietnameseSearch(item, this.searchTerm)
    );
    return results.slice(0, 50); // Limit to 50 results
}
```

---

## 🚨 **Common Use Cases**

### **1. Customer Directory**
```typescript
export class CustomerDirectoryPage extends PageBase {
    customers: Customer[] = [];
    searchTerm = '';
    
    get customerSearch() {
        return {
            name: this.searchTerm,
            address: this.searchTerm,
            company: this.searchTerm
        };
    }
}
```

### **2. Location Search**
```typescript
export class LocationSearchPage extends PageBase {
    locations: Location[] = [];
    locationSearch = '';
    
    get locationConditions() {
        return {
            name: this.locationSearch,
            district: this.locationSearch
        };
    }
}
```

### **3. Product Search with Vietnamese Names**
```typescript
export class VietnameseProductPage extends PageBase {
    products: Product[] = [];
    productSearch = '';
    
    get productConditions() {
        return {
            name: this.productSearch,
            description: this.productSearch
        };
    }
}
```

---

## 🌟 **Special Features**

### **1. Accent Removal Algorithm**
The pipe removes Vietnamese accents using comprehensive character mapping:
- **à, á, ạ, ả, ã, â, ầ, ấ, ậ, ẩ, ẫ, ă, ằ, ắ, ặ, ẳ, ẵ** → **a**
- **è, é, ẹ, ẻ, ẽ, ê, ề, ế, ệ, ể, ễ** → **e**
- **ì, í, ị, ỉ, ĩ** → **i**
- **ò, ó, ọ, ỏ, õ, ô, ồ, ố, ộ, ổ, ỗ, ơ, ờ, ớ, ợ, ở, ỡ** → **o**
- **ù, ú, ụ, ủ, ũ, ư, ừ, ứ, ự, ử, ữ** → **u**
- **ỳ, ý, ỵ, ỷ, ỹ** → **y**
- **đ** → **d**

### **2. Special "deals" Keyword**
When search term is "deals", the pipe looks for products with `NewPrice` in their price lists.

---

## ⚠️ **Limitations**

1. **Vietnamese-specific**: Chỉ hỗ trợ tiếng Việt
2. **Performance**: Impure pipe với accent removal có thể chậm với large datasets
3. **Exact substring matching**: Không support fuzzy search
4. **Special deals logic**: Hard-coded cho specific data structure

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
