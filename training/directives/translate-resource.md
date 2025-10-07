# TranslateResource Directive Documentation

## 📋 **Tổng quan**

`TranslateResourceDirective` là directive để tự động translate content dựa trên language settings. Directive này tự động switch giữa default language và foreign language content, và update real-time khi user thay đổi ngôn ngữ.

**Selector**: `[appTranslateResource]`  
**Location**: `src/app/directives/translate-resource.directive.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() appTranslateResource: any;      // Object chứa translation data
@Input() nameProperty: string = 'Name';  // Property name cho default language
@Input() foreignNameProperty: string = 'ForeignName'; // Property name cho foreign language
```

---

## 🚀 **Basic Usage**

### **Simple Translation**
```typescript
// Component
export class ProductListPage extends PageBase {
    products = [
        { 
            Id: 1, 
            Name: 'Laptop', 
            ForeignName: 'Máy tính xách tay',
            Description: 'Gaming laptop',
            ForeignDescription: 'Laptop chơi game'
        },
        { 
            Id: 2, 
            Name: 'Phone', 
            ForeignName: 'Điện thoại',
            Description: 'Smartphone',
            ForeignDescription: 'Điện thoại thông minh'
        }
    ];
}
```

```html
<!-- Template -->
<div class="product-list">
    <div class="product-item" *ngFor="let product of products">
        <!-- Auto-translate product name -->
        <h3 [appTranslateResource]="product"></h3>
        
        <!-- Auto-translate description with custom properties -->
        <p [appTranslateResource]="product"
           [nameProperty]="'Description'"
           [foreignNameProperty]="'ForeignDescription'">
        </p>
    </div>
</div>
```

### **Category Navigation**
```typescript
export class CategoryMenuPage extends PageBase {
    categories = [
        { 
            Id: 1, 
            Name: 'Electronics', 
            ForeignName: 'Điện tử',
            Icon: 'laptop-outline'
        },
        { 
            Id: 2, 
            Name: 'Clothing', 
            ForeignName: 'Quần áo',
            Icon: 'shirt-outline'
        },
        { 
            Id: 3, 
            Name: 'Books', 
            ForeignName: 'Sách',
            Icon: 'book-outline'
        }
    ];
}
```

```html
<div class="category-menu">
    <ion-item button *ngFor="let category of categories" (click)="selectCategory(category)">
        <ion-icon [name]="category.Icon" slot="start"></ion-icon>
        <ion-label [appTranslateResource]="category"></ion-label>
        <ion-icon name="chevron-forward" slot="end"></ion-icon>
    </ion-item>
</div>
```

---

## 🎨 **Advanced Usage**

### **Dynamic Content Translation**
```typescript
export class DynamicContentPage extends PageBase {
    contentItems = [];
    
    async loadContent(categoryId: string) {
        try {
            const items = await this.contentService.getItems(categoryId);
            
            // Items come with Name and ForeignName properties
            this.contentItems = items.map(item => ({
                ...item,
                // Ensure properties exist
                Name: item.Name || item.Title || 'Untitled',
                ForeignName: item.ForeignName || item.ForeignTitle || item.Name
            }));
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

```html
<div class="dynamic-content">
    <div class="content-grid">
        <ion-card *ngFor="let item of contentItems">
            <ion-card-header>
                <!-- Auto-translate title -->
                <ion-card-title [appTranslateResource]="item"></ion-card-title>
                
                <!-- Auto-translate subtitle with custom properties -->
                <ion-card-subtitle 
                    [appTranslateResource]="item"
                    [nameProperty]="'Subtitle'"
                    [foreignNameProperty]="'ForeignSubtitle'">
                </ion-card-subtitle>
            </ion-card-header>
            
            <ion-card-content>
                <!-- Auto-translate content -->
                <p [appTranslateResource]="item"
                   [nameProperty]="'Content'"
                   [foreignNameProperty]="'ForeignContent'">
                </p>
            </ion-card-content>
        </ion-card>
    </div>
</div>
```

### **Form Labels Translation**
```typescript
export class MultiLanguageFormPage extends PageBase {
    formLabels = {
        personalInfo: {
            Name: 'Personal Information',
            ForeignName: 'Thông tin cá nhân'
        },
        fullName: {
            Name: 'Full Name',
            ForeignName: 'Họ và tên'
        },
        email: {
            Name: 'Email Address',
            ForeignName: 'Địa chỉ email'
        },
        phone: {
            Name: 'Phone Number',
            ForeignName: 'Số điện thoại'
        },
        address: {
            Name: 'Address',
            ForeignName: 'Địa chỉ'
        }
    };
    
    formData = {
        fullName: '',
        email: '',
        phone: '',
        address: ''
    };
}
```

```html
<form class="multilang-form">
    <!-- Section header -->
    <div class="form-section">
        <h2 [appTranslateResource]="formLabels.personalInfo"></h2>
        
        <!-- Form fields with translated labels -->
        <ion-item>
            <ion-label [appTranslateResource]="formLabels.fullName" position="stacked"></ion-label>
            <ion-input [(ngModel)]="formData.fullName" name="fullName"></ion-input>
        </ion-item>
        
        <ion-item>
            <ion-label [appTranslateResource]="formLabels.email" position="stacked"></ion-label>
            <ion-input [(ngModel)]="formData.email" name="email" type="email"></ion-input>
        </ion-item>
        
        <ion-item>
            <ion-label [appTranslateResource]="formLabels.phone" position="stacked"></ion-label>
            <ion-input [(ngModel)]="formData.phone" name="phone" type="tel"></ion-input>
        </ion-item>
        
        <ion-item>
            <ion-label [appTranslateResource]="formLabels.address" position="stacked"></ion-label>
            <ion-textarea [(ngModel)]="formData.address" name="address"></ion-textarea>
        </ion-item>
    </div>
</form>
```

### **Data Table with Translation**
```typescript
export class TranslatedDataTablePage extends PageBase {
    tableColumns = [
        {
            prop: 'name',
            header: { Name: 'Product Name', ForeignName: 'Tên sản phẩm' }
        },
        {
            prop: 'category',
            header: { Name: 'Category', ForeignName: 'Danh mục' }
        },
        {
            prop: 'price',
            header: { Name: 'Price', ForeignName: 'Giá' }
        },
        {
            prop: 'status',
            header: { Name: 'Status', ForeignName: 'Trạng thái' }
        }
    ];
    
    tableData = [
        {
            name: { Name: 'Gaming Laptop', ForeignName: 'Laptop Gaming' },
            category: { Name: 'Electronics', ForeignName: 'Điện tử' },
            price: 25000000,
            status: { Name: 'In Stock', ForeignName: 'Còn hàng' }
        },
        {
            name: { Name: 'Wireless Mouse', ForeignName: 'Chuột không dây' },
            category: { Name: 'Accessories', ForeignName: 'Phụ kiện' },
            price: 500000,
            status: { Name: 'Out of Stock', ForeignName: 'Hết hàng' }
        }
    ];
}
```

```html
<div class="translated-table">
    <table class="data-table">
        <thead>
            <tr>
                <th *ngFor="let col of tableColumns">
                    <span [appTranslateResource]="col.header"></span>
                </th>
            </tr>
        </thead>
        <tbody>
            <tr *ngFor="let row of tableData">
                <td [appTranslateResource]="row.name"></td>
                <td [appTranslateResource]="row.category"></td>
                <td>{{ row.price | currency:'VND' }}</td>
                <td [appTranslateResource]="row.status"></td>
            </tr>
        </tbody>
    </table>
</div>
```

---

## 🔄 **Real-time Language Switching**

### **Language Toggle Component**
```typescript
export class LanguageTogglePage extends PageBase {
    currentLanguage = 'en';
    
    toggleLanguage() {
        this.currentLanguage = this.currentLanguage === 'en' ? 'vi' : 'en';
        this.env.setLang(this.currentLanguage);
    }
    
    // Sample content that will auto-update
    pageContent = {
        title: { Name: 'Welcome to Our Store', ForeignName: 'Chào mừng đến cửa hàng' },
        subtitle: { Name: 'Find the best products', ForeignName: 'Tìm những sản phẩm tốt nhất' },
        description: { 
            Name: 'We offer high-quality products at competitive prices', 
            ForeignName: 'Chúng tôi cung cấp sản phẩm chất lượng cao với giá cạnh tranh' 
        }
    };
}
```

```html
<div class="language-demo">
    <!-- Language toggle -->
    <div class="language-controls">
        <ion-button (click)="toggleLanguage()" fill="outline">
            <ion-icon name="language" slot="start"></ion-icon>
            Switch to {{ currentLanguage === 'en' ? 'Vietnamese' : 'English' }}
        </ion-button>
    </div>
    
    <!-- Content that auto-updates -->
    <div class="demo-content">
        <h1 [appTranslateResource]="pageContent.title"></h1>
        <h2 [appTranslateResource]="pageContent.subtitle"></h2>
        <p [appTranslateResource]="pageContent.description"></p>
    </div>
</div>
```

---

## 🎨 **Styling**

### **Language-specific Styling**
```scss
// Component SCSS
.multilang-content {
    // Base styles
    [appTranslateResource] {
        transition: opacity 0.2s ease;
        
        &.updating {
            opacity: 0.7;
        }
    }
    
    // Language-specific adjustments
    &.lang-vi {
        [appTranslateResource] {
            font-family: 'Roboto', sans-serif;
            line-height: 1.6;
        }
    }
    
    &.lang-en {
        [appTranslateResource] {
            font-family: 'Arial', sans-serif;
            line-height: 1.4;
        }
    }
}

.form-section {
    margin: 20px 0;
    
    h2[appTranslateResource] {
        color: var(--ion-color-primary);
        border-bottom: 2px solid var(--ion-color-primary);
        padding-bottom: 8px;
        margin-bottom: 16px;
    }
    
    ion-label[appTranslateResource] {
        font-weight: 500;
        color: var(--ion-color-dark);
    }
}

.data-table {
    width: 100%;
    border-collapse: collapse;
    
    th[appTranslateResource] {
        background: var(--ion-color-light);
        padding: 12px;
        text-align: left;
        font-weight: 600;
        border-bottom: 2px solid var(--ion-color-medium);
    }
    
    td[appTranslateResource] {
        padding: 10px 12px;
        border-bottom: 1px solid var(--ion-color-light);
    }
}
```

---

## 🔧 **Best Practices**

### **1. Data Structure**
```typescript
// ✅ Đúng - Consistent property naming
interface TranslatableItem {
    Name: string;
    ForeignName?: string;
    // Other properties...
}

// ✅ Đúng - Fallback values
const item = {
    Name: 'Default Name',
    ForeignName: foreignName || 'Default Name' // Fallback
};
```

### **2. Performance Optimization**
```typescript
// ✅ Đúng - Reuse translation objects
const sharedLabels = {
    save: { Name: 'Save', ForeignName: 'Lưu' },
    cancel: { Name: 'Cancel', ForeignName: 'Hủy' },
    delete: { Name: 'Delete', ForeignName: 'Xóa' }
};

// ❌ Sai - Create new objects repeatedly
// Creates unnecessary objects in templates
```

### **3. Memory Management**
```typescript
// ✅ Đúng - Directive handles subscription cleanup
// No manual cleanup needed in components

// ✅ Đúng - Avoid memory leaks in custom implementations
ngOnDestroy() {
    this.languageSubscription?.unsubscribe();
}
```

---

## 🚨 **Common Use Cases**

### **1. E-commerce Product Catalog**
```typescript
export class ProductCatalogPage extends PageBase {
    // Products with Vietnamese translations
    products: Product[] = [];
}
```

### **2. Multi-language Forms**
```typescript
export class ContactFormPage extends PageBase {
    // Form labels in multiple languages
    formLabels: FormLabels = {};
}
```

### **3. Navigation Menus**
```typescript
export class NavigationPage extends PageBase {
    // Menu items with translations
    menuItems: MenuItem[] = [];
}
```

### **4. Data Tables & Lists**
```typescript
export class DataListPage extends PageBase {
    // Table headers and data with translations
    columns: TableColumn[] = [];
    data: TableRow[] = [];
}
```

---

## ⚠️ **Limitations**

1. **Property Names**: Requires specific property naming convention (Name/ForeignName)
2. **Object Structure**: Works with objects, not primitive strings
3. **Language Detection**: Relies on EnvService language tracking
4. **HTML Content**: Sets innerHTML directly (be careful with user-generated content)

---

## 🔄 **Integration with EnvService**

The directive integrates with `EnvService` for:
- **Language tracking**: Subscribes to language changes
- **Default language detection**: Checks `env.language.isDefault`
- **Automatic updates**: Updates content when language changes

---

## 🛠️ **Troubleshooting**

### **Common Issues**

1. **Content not updating**: Check if object has required properties
2. **Memory leaks**: Directive handles cleanup automatically
3. **Missing translations**: Provide fallback values
4. **Performance issues**: Avoid creating objects in templates

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
