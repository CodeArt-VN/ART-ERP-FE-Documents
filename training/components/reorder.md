# Reorder Component Documentation

## 📋 **Tổng quan**

`ReorderComponent` là component hỗ trợ drag & drop để sắp xếp lại thứ tự các items trong danh sách. Component này tự động cập nhật sort property và emit events khi có thay đổi.

**Selector**: `app-reorder`  
**Location**: `src/app/components/reorder/reorder.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() items: any[];                   // List of items to reorder
@Input() labelField: string = 'Name';    // Field to display as label
@Input() sortProperty: string = 'Sort';  // Property to store sort order
```

### **Output Events**
```typescript
@Output() reoder = new EventEmitter();   // Emits reordered items
```

---

## 🚀 **Basic Usage**

### **Simple List Reordering**
```typescript
// Component
export class MenuReorderPage extends PageBase {
    menuItems = [
        { Id: 1, Name: 'Dashboard', Sort: 0, Icon: 'analytics' },
        { Id: 2, Name: 'Users', Sort: 1, Icon: 'people' },
        { Id: 3, Name: 'Products', Sort: 2, Icon: 'cube' },
        { Id: 4, Name: 'Orders', Sort: 3, Icon: 'receipt' },
        { Id: 5, Name: 'Reports', Sort: 4, Icon: 'bar-chart' }
    ];

    onItemsReordered(reorderedItems: any[]) {
        this.menuItems = reorderedItems;
        this.saveMenuOrder();
    }

    async saveMenuOrder() {
        try {
            await this.menuService.updateOrder(this.menuItems);
            this.env.showMessage('MENU_ORDER_SAVED', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

```html
<!-- Template -->
<div class="menu-reorder">
    <h3>Customize Menu Order</h3>
    
    <app-reorder
        [items]="menuItems"
        [labelField]="'Name'"
        [sortProperty]="'Sort'"
        (reoder)="onItemsReordered($event)">
    </app-reorder>
</div>
```

### **Category Reordering**
```typescript
export class CategoryOrderPage extends PageBase {
    categories = [
        { Id: 10, Name: 'Electronics', DisplayOrder: 1, IsActive: true },
        { Id: 20, Name: 'Clothing', DisplayOrder: 2, IsActive: true },
        { Id: 30, Name: 'Books', DisplayOrder: 3, IsActive: true },
        { Id: 40, Name: 'Home & Garden', DisplayOrder: 4, IsActive: true }
    ];

    onCategoriesReordered(reorderedCategories: any[]) {
        this.categories = reorderedCategories;
        this.updateCategoryOrder();
    }

    async updateCategoryOrder() {
        try {
            const updates = this.categories.map(cat => ({
                Id: cat.Id,
                DisplayOrder: cat.DisplayOrder
            }));
            
            await this.categoryService.bulkUpdateOrder(updates);
            this.env.showMessage('CATEGORY_ORDER_UPDATED', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

```html
<ion-card>
    <ion-card-header>
        <ion-card-title>Category Display Order</ion-card-title>
        <ion-card-subtitle>Drag to reorder categories</ion-card-subtitle>
    </ion-card-header>
    
    <ion-card-content>
        <app-reorder
            [items]="categories"
            [labelField]="'Name'"
            [sortProperty]="'DisplayOrder'"
            (reoder)="onCategoriesReordered($event)">
        </app-reorder>
    </ion-card-content>
</ion-card>
```

---

## 🎨 **Advanced Features**

### **Custom Item Template**
```html
<app-reorder
    [items]="menuItems"
    [labelField]="'Name'"
    [sortProperty]="'Sort'"
    (reoder)="onItemsReordered($event)">
    
    <!-- Custom item template -->
    <ng-template #itemTemplate let-item let-index="index">
        <div class="custom-reorder-item">
            <div class="item-icon">
                <ion-icon [name]="item.Icon"></ion-icon>
            </div>
            <div class="item-content">
                <h4>{{ item.Name }}</h4>
                <p>{{ item.Description }}</p>
            </div>
            <div class="item-status">
                <ion-badge [color]="item.IsActive ? 'success' : 'medium'">
                    {{ item.IsActive ? 'Active' : 'Inactive' }}
                </ion-badge>
            </div>
        </div>
    </ng-template>
</app-reorder>
```

### **Conditional Reordering**
```typescript
export class ConditionalReorderPage extends PageBase {
    items = [];
    isEditMode = false;
    canReorder = false;

    async ngOnInit() {
        this.canReorder = await this.env.checkFormPermission('/menu/reorder');
    }

    toggleEditMode() {
        this.isEditMode = !this.isEditMode;
    }

    onItemsReordered(reorderedItems: any[]) {
        if (!this.canReorder) {
            this.env.showMessage('NO_PERMISSION_REORDER', 'warning');
            return;
        }

        this.items = reorderedItems;
        this.saveOrder();
    }
}
```

```html
<div class="reorder-container">
    <div class="reorder-header">
        <h3>Menu Items</h3>
        <ion-button 
            *ngIf="canReorder"
            fill="outline"
            (click)="toggleEditMode()">
            {{ isEditMode ? 'Done' : 'Reorder' }}
        </ion-button>
    </div>

    <app-reorder
        *ngIf="isEditMode"
        [items]="items"
        (reoder)="onItemsReordered($event)">
    </app-reorder>

    <ion-list *ngIf="!isEditMode">
        <ion-item *ngFor="let item of items">
            <ion-label>{{ item.Name }}</ion-label>
        </ion-item>
    </ion-list>
</div>
```

---

## 🎨 **Styling**

### **Custom Reorder Styling**
```scss
// Component SCSS
app-reorder {
    .reorder-list {
        ion-reorder-group {
            ion-item {
                --background: var(--ion-color-light);
                --border-radius: 8px;
                margin-bottom: 8px;
                
                &.reorder-active {
                    --background: var(--ion-color-primary-tint);
                    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
                    transform: scale(1.02);
                }
                
                ion-reorder {
                    --color: var(--ion-color-medium);
                    
                    &:hover {
                        --color: var(--ion-color-primary);
                    }
                }
                
                .reorder-content {
                    display: flex;
                    align-items: center;
                    gap: 12px;
                    padding: 8px 0;
                    
                    .reorder-icon {
                        width: 32px;
                        height: 32px;
                        display: flex;
                        align-items: center;
                        justify-content: center;
                        background: var(--ion-color-primary-tint);
                        border-radius: 6px;
                        
                        ion-icon {
                            color: var(--ion-color-primary);
                        }
                    }
                    
                    .reorder-info {
                        flex: 1;
                        
                        .item-name {
                            font-weight: 500;
                            color: var(--ion-color-dark);
                        }
                        
                        .item-description {
                            font-size: 0.875rem;
                            color: var(--ion-color-medium);
                            margin-top: 2px;
                        }
                    }
                    
                    .reorder-order {
                        font-size: 0.875rem;
                        color: var(--ion-color-medium);
                        font-weight: 500;
                    }
                }
            }
        }
    }
}
```

---

## 📱 **Mobile Optimization**

### **Touch-Friendly Design**
```scss
// Mobile-specific styles
@media (max-width: 768px) {
    app-reorder {
        .reorder-list {
            ion-reorder-group {
                ion-item {
                    --min-height: 60px; // Larger touch target
                    
                    ion-reorder {
                        --size: 24px; // Larger reorder handle
                    }
                    
                    .reorder-content {
                        .reorder-icon {
                            width: 40px;
                            height: 40px;
                        }
                        
                        .reorder-info {
                            .item-name {
                                font-size: 1rem;
                            }
                        }
                    }
                }
            }
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Đúng - Efficient reordering
onItemsReordered(reorderedItems: any[]) {
    // Update only if order actually changed
    const hasOrderChanged = this.hasOrderChanged(this.items, reorderedItems);
    
    if (hasOrderChanged) {
        this.items = reorderedItems;
        this.debouncedSave();
    }
}

private hasOrderChanged(original: any[], reordered: any[]): boolean {
    return !original.every((item, index) => 
        item.Id === reordered[index].Id
    );
}

private debouncedSave = debounceTime(500)(() => {
    this.saveOrder();
});
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Handle reorder errors
async saveOrder() {
    try {
        this.saving = true;
        await this.service.updateOrder(this.items);
        this.env.showMessage('ORDER_SAVED', 'success');
    } catch (error) {
        this.env.showErrorMessage(error);
        // Revert to original order on error
        await this.loadOriginalOrder();
    } finally {
        this.saving = false;
    }
}
```

### **3. Accessibility**
```html
<!-- ✅ Đúng - Accessible reorder -->
<ion-reorder-group 
    [disabled]="!canReorder"
    (ionItemReorder)="onReorder($event)"
    role="list"
    aria-label="Reorderable menu items">
    
    <ion-item 
        *ngFor="let item of items; trackBy: trackByFn"
        role="listitem"
        [attr.aria-label]="'Menu item: ' + item.Name">
        
        <ion-label>{{ item.Name }}</ion-label>
        
        <ion-reorder 
            slot="end"
            [attr.aria-label]="'Reorder ' + item.Name">
        </ion-reorder>
    </ion-item>
</ion-reorder-group>
```

---

## 🚨 **Common Use Cases**

### **1. Dashboard Widget Ordering**
```typescript
export class DashboardCustomization {
    widgets = [
        { Id: 1, Name: 'Sales Chart', Type: 'chart', Order: 1 },
        { Id: 2, Name: 'Recent Orders', Type: 'list', Order: 2 },
        { Id: 3, Name: 'Quick Stats', Type: 'stats', Order: 3 }
    ];

    onWidgetReorder(reorderedWidgets: any[]) {
        this.widgets = reorderedWidgets;
        this.saveDashboardLayout();
    }
}
```

### **2. Form Field Ordering**
```typescript
export class FormBuilder {
    formFields = [
        { Id: 1, Name: 'Name', Type: 'text', Required: true, Order: 1 },
        { Id: 2, Name: 'Email', Type: 'email', Required: true, Order: 2 },
        { Id: 3, Name: 'Phone', Type: 'tel', Required: false, Order: 3 }
    ];

    onFieldReorder(reorderedFields: any[]) {
        this.formFields = reorderedFields;
        this.updateFormSchema();
    }
}
```

### **3. Navigation Menu Customization**
```typescript
export class NavigationCustomization {
    navigationItems = [
        { Id: 1, Name: 'Home', Route: '/home', Order: 1 },
        { Id: 2, Name: 'Products', Route: '/products', Order: 2 },
        { Id: 3, Name: 'Orders', Route: '/orders', Order: 3 }
    ];

    onNavigationReorder(reorderedItems: any[]) {
        this.navigationItems = reorderedItems;
        this.updateNavigationConfig();
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
