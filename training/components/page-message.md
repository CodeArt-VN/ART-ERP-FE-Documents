# PageMessage Component Documentation

## 📋 **Tổng quan**

`PageMessageComponent` là component hiển thị empty state messages khi không có dữ liệu hoặc khi đang loading. Component này hỗ trợ custom images, messages và spinner states.

**Selector**: `app-page-message`  
**Location**: `src/app/components/page-message/page-message.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() itemsLength: number;            // Number of items (for conditional display)
@Input() showSpinner: boolean;           // Show loading spinner
@Input() message: string;                // Main message text
@Input() subMessage: string;             // Secondary message text
@Input() minHeight: string = '50vh';     // Minimum height of container
@Input() showImg: boolean = true;        // Show illustration image
@Input() imgSrc: string;                 // Custom image source
```

---

## 🚀 **Basic Usage**

### **Empty State Display**
```typescript
// Component
export class ProductListPage extends PageBase {
    products: any[] = [];
    loading = false;

    async loadProducts() {
        try {
            this.loading = true;
            const result = await this.productService.read();
            this.products = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        } finally {
            this.loading = false;
        }
    }
}
```

```html
<!-- Template -->
<div class="product-list">
    <!-- Data table when has data -->
    <app-data-table 
        *ngIf="products.length > 0"
        [rows]="products"
        [columns]="columns">
    </app-data-table>

    <!-- Empty state message -->
    <app-page-message
        *ngIf="products.length === 0"
        [itemsLength]="products.length"
        [showSpinner]="loading"
        [message]="'No products found'"
        [subMessage]="'Add your first product to get started'"
        [imgSrc]="'undraw_empty_cart.svg'">
    </app-page-message>
</div>
```

### **Loading State**
```typescript
export class OrderListPage extends PageBase {
    orders: any[] = [];
    loading = false;
    searchQuery = '';

    async searchOrders() {
        if (!this.searchQuery.trim()) {
            this.orders = [];
            return;
        }

        try {
            this.loading = true;
            const result = await this.orderService.search({
                keyword: this.searchQuery
            });
            this.orders = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        } finally {
            this.loading = false;
        }
    }
}
```

```html
<div class="search-container">
    <ion-searchbar
        [(ngModel)]="searchQuery"
        (ionInput)="searchOrders()"
        placeholder="Search orders...">
    </ion-searchbar>

    <!-- Results or empty state -->
    <div class="search-results">
        <app-data-table 
            *ngIf="orders.length > 0"
            [rows]="orders"
            [columns]="columns">
        </app-data-table>

        <app-page-message
            *ngIf="orders.length === 0"
            [showSpinner]="loading"
            [message]="loading ? 'Searching...' : 'No orders found'"
            [subMessage]="loading ? 'Please wait' : 'Try different search terms'"
            [imgSrc]="loading ? null : 'undraw_search_not_found.svg'">
        </app-page-message>
    </div>
</div>
```

---

## 🎨 **Custom Messages & Images**

### **Different Empty States**
```typescript
export class MultiStateMessagesPage extends PageBase {
    getEmptyStateConfig(context: string) {
        const configs = {
            'no-data': {
                message: 'No data available',
                subMessage: 'Data will appear here when available',
                imgSrc: 'undraw_no_data.svg'
            },
            'no-results': {
                message: 'No results found',
                subMessage: 'Try adjusting your search criteria',
                imgSrc: 'undraw_not_found.svg'
            },
            'no-permissions': {
                message: 'Access denied',
                subMessage: 'You don\'t have permission to view this content',
                imgSrc: 'undraw_access_denied.svg'
            },
            'offline': {
                message: 'You\'re offline',
                subMessage: 'Check your internet connection and try again',
                imgSrc: 'undraw_connection_lost.svg'
            },
            'error': {
                message: 'Something went wrong',
                subMessage: 'Please try again later',
                imgSrc: 'undraw_server_down.svg'
            }
        };

        return configs[context] || configs['no-data'];
    }
}
```

```html
<!-- Different contexts -->
<app-page-message
    *ngIf="showEmptyState"
    [message]="getEmptyStateConfig(currentContext).message"
    [subMessage]="getEmptyStateConfig(currentContext).subMessage"
    [imgSrc]="getEmptyStateConfig(currentContext).imgSrc">
</app-page-message>
```

### **Custom Action Buttons**
```html
<app-page-message
    [message]="'No products in your inventory'"
    [subMessage]="'Start by adding your first product'"
    [imgSrc]="'undraw_add_product.svg'">
    
    <!-- Custom action buttons -->
    <div class="empty-state-actions" slot="actions">
        <ion-button 
            expand="block"
            (click)="addFirstProduct()">
            <ion-icon name="add" slot="start"></ion-icon>
            Add Product
        </ion-button>
        
        <ion-button 
            fill="outline"
            expand="block"
            (click)="importProducts()">
            <ion-icon name="cloud-upload" slot="start"></ion-icon>
            Import from File
        </ion-button>
    </div>
</app-page-message>
```

---

## 🎨 **Styling Customization**

### **Custom Styling**
```scss
// Component SCSS
app-page-message {
    .page-message-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        text-align: center;
        padding: 40px 20px;
        
        .message-image {
            max-width: 300px;
            width: 100%;
            height: auto;
            margin-bottom: 24px;
            opacity: 0.8;
        }
        
        .message-content {
            max-width: 400px;
            
            .main-message {
                font-size: 1.25rem;
                font-weight: 600;
                color: var(--ion-color-dark);
                margin-bottom: 8px;
            }
            
            .sub-message {
                font-size: 0.875rem;
                color: var(--ion-color-medium);
                line-height: 1.4;
                margin-bottom: 24px;
            }
        }
        
        .loading-spinner {
            margin-bottom: 16px;
            
            ion-spinner {
                width: 32px;
                height: 32px;
                --color: var(--ion-color-primary);
            }
        }
        
        .empty-state-actions {
            display: flex;
            flex-direction: column;
            gap: 12px;
            width: 100%;
            max-width: 280px;
            
            ion-button {
                --border-radius: 8px;
            }
        }
    }
}
```

### **Theme Variants**
```html
<!-- Success theme -->
<app-page-message
    class="success-theme"
    [message]="'All tasks completed!'"
    [subMessage]="'Great job! Everything is up to date'"
    [imgSrc]="'undraw_completed_tasks.svg'">
</app-page-message>

<!-- Warning theme -->
<app-page-message
    class="warning-theme"
    [message]="'Action required'"
    [subMessage]="'Please complete the required steps'"
    [imgSrc]="'undraw_warning.svg'">
</app-page-message>

<!-- Error theme -->
<app-page-message
    class="error-theme"
    [message]="'Connection failed'"
    [subMessage]="'Unable to load data. Please try again'"
    [imgSrc]="'undraw_server_down.svg'">
</app-page-message>
```

```scss
.success-theme {
    .main-message { color: var(--ion-color-success); }
}

.warning-theme {
    .main-message { color: var(--ion-color-warning); }
}

.error-theme {
    .main-message { color: var(--ion-color-danger); }
}
```

---

## 📱 **Responsive Design**

### **Mobile Optimization**
```scss
// Mobile-specific styles
@media (max-width: 768px) {
    app-page-message {
        .page-message-container {
            padding: 20px 16px;
            
            .message-image {
                max-width: 250px;
                margin-bottom: 20px;
            }
            
            .message-content {
                .main-message {
                    font-size: 1.125rem;
                }
                
                .sub-message {
                    font-size: 0.8125rem;
                }
            }
            
            .empty-state-actions {
                max-width: 100%;
                
                ion-button {
                    --padding-start: 16px;
                    --padding-end: 16px;
                }
            }
        }
    }
}
```

---

## 🔄 **Dynamic Content**

### **Context-Aware Messages**
```typescript
export class ContextAwareMessages extends PageBase {
    getContextualMessage(): any {
        if (this.loading) {
            return {
                message: 'Loading...',
                subMessage: 'Please wait while we fetch your data',
                showSpinner: true,
                imgSrc: null
            };
        }

        if (this.hasError) {
            return {
                message: 'Unable to load data',
                subMessage: 'Please check your connection and try again',
                showSpinner: false,
                imgSrc: 'undraw_server_down.svg'
            };
        }

        if (this.hasFilters && this.items.length === 0) {
            return {
                message: 'No results match your filters',
                subMessage: 'Try adjusting your search criteria',
                showSpinner: false,
                imgSrc: 'undraw_not_found.svg'
            };
        }

        return {
            message: 'No items found',
            subMessage: 'Add your first item to get started',
            showSpinner: false,
            imgSrc: 'undraw_empty_state.svg'
        };
    }
}
```

```html
<app-page-message
    *ngIf="shouldShowMessage"
    [message]="getContextualMessage().message"
    [subMessage]="getContextualMessage().subMessage"
    [showSpinner]="getContextualMessage().showSpinner"
    [imgSrc]="getContextualMessage().imgSrc">
</app-page-message>
```

### **Internationalization**
```typescript
export class I18nPageMessage extends PageBase {
    getLocalizedMessage(key: string): string {
        const messages = {
            'NO_DATA': this.translate.instant('NO_DATA_AVAILABLE'),
            'NO_RESULTS': this.translate.instant('NO_SEARCH_RESULTS'),
            'LOADING': this.translate.instant('LOADING_PLEASE_WAIT'),
            'ERROR': this.translate.instant('ERROR_OCCURRED')
        };

        return messages[key] || messages['NO_DATA'];
    }
}
```

---

## 🔧 **Integration Patterns**

### **With Data Tables**
```html
<div class="data-container">
    <app-list-toolbar
        [selectedItems]="selectedItems"
        (add)="add()"
        (refresh)="refresh()">
    </app-list-toolbar>

    <app-data-table 
        *ngIf="items.length > 0"
        [rows]="items"
        [columns]="columns"
        [(selectedRows)]="selectedItems">
    </app-data-table>

    <app-page-message
        *ngIf="items.length === 0"
        [showSpinner]="loading"
        [message]="loading ? 'Loading data...' : 'No items found'"
        [subMessage]="loading ? 'Please wait' : 'Click Add to create your first item'">
    </app-page-message>
</div>
```

### **With Search Results**
```typescript
export class SearchResultsPage extends PageBase {
    searchResults: any[] = [];
    searchQuery = '';
    searching = false;

    get shouldShowMessage(): boolean {
        return !this.searching && 
               this.searchQuery.length > 0 && 
               this.searchResults.length === 0;
    }

    get messageConfig() {
        if (this.searching) {
            return {
                message: 'Searching...',
                subMessage: 'Please wait',
                showSpinner: true
            };
        }

        return {
            message: `No results for "${this.searchQuery}"`,
            subMessage: 'Try different keywords or check spelling',
            showSpinner: false
        };
    }
}
```

---

## 🔧 **Best Practices**

### **1. Conditional Display Logic**
```typescript
// ✅ Đúng - Clear conditions
get shouldShowEmptyState(): boolean {
    return !this.loading && 
           !this.hasError && 
           this.items.length === 0;
}

get shouldShowErrorState(): boolean {
    return !this.loading && this.hasError;
}

get shouldShowLoadingState(): boolean {
    return this.loading;
}
```

### **2. Accessibility**
```html
<!-- ✅ Đúng - Accessible empty state -->
<app-page-message
    role="status"
    [attr.aria-live]="loading ? 'polite' : 'off'"
    [attr.aria-label]="message + '. ' + subMessage"
    [message]="message"
    [subMessage]="subMessage">
</app-page-message>
```

### **3. Performance**
```typescript
// ✅ Đúng - Optimize image loading
getOptimizedImageSrc(imageName: string): string {
    if (!imageName) return null;
    
    // Use WebP format if supported
    const supportsWebP = this.checkWebPSupport();
    const extension = supportsWebP ? '.webp' : '.svg';
    
    return `assets/images/${imageName}${extension}`;
}
```

---

## 🚨 **Common Use Cases**

### **1. Dashboard Widgets**
```typescript
export class DashboardWidget {
    widgetData: any[] = [];
    
    get emptyStateConfig() {
        return {
            message: 'No data for this period',
            subMessage: 'Data will appear when available',
            imgSrc: 'undraw_no_data_chart.svg'
        };
    }
}
```

### **2. File Uploads**
```typescript
export class FileUploadArea {
    uploadedFiles: any[] = [];
    
    get uploadEmptyState() {
        return {
            message: 'No files uploaded',
            subMessage: 'Drag and drop files here or click to browse',
            imgSrc: 'undraw_upload_files.svg'
        };
    }
}
```

### **3. Notification Center**
```typescript
export class NotificationCenter {
    notifications: any[] = [];
    
    get notificationEmptyState() {
        return {
            message: 'All caught up!',
            subMessage: 'No new notifications',
            imgSrc: 'undraw_all_done.svg'
        };
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
