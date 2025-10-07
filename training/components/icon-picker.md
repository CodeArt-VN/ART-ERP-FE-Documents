# IconPicker Component Documentation

## 📋 **Tổng quan**

`IconPickerComponent` là component cho phép người dùng chọn icon từ một bộ sưu tập icons có sẵn. Component này hỗ trợ tìm kiếm theo tên và tags, hiển thị preview và emit selected icon.

**Selector**: `app-icon-picker`  
**Location**: `src/app/components/controls/icon-picker/icon-picker.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() icon: string;                   // Currently selected icon
@Input() color: string;                  // Icon color theme
```

### **Output Events**
```typescript
@Output() selected = new EventEmitter(); // Icon selection event
```

### **Internal Properties**
```typescript
items: any[];                           // Available icons list
keyword: string = '';                   // Search keyword
```

---

## 🚀 **Basic Usage**

### **Simple Icon Picker**
```typescript
// Component
export class IconSelectionPage extends PageBase {
    selectedIcon = 'home';
    iconColor = 'primary';

    onIconSelected(icon: any) {
        this.selectedIcon = icon.Name;
        console.log('Selected icon:', icon);
    }
}
```

```html
<!-- Template -->
<app-icon-picker
    [icon]="selectedIcon"
    [color]="iconColor"
    (selected)="onIconSelected($event)">
</app-icon-picker>

<!-- Display selected icon -->
<div class="selected-icon-preview">
    <ion-icon 
        [name]="selectedIcon" 
        [color]="iconColor">
    </ion-icon>
    <span>{{ selectedIcon }}</span>
</div>
```

### **With Form Integration**
```typescript
export class FormWithIconPage extends PageBase {
    formGroup: FormGroup;

    constructor(private formBuilder: FormBuilder) {
        super();
        this.formGroup = this.formBuilder.group({
            name: ['', Validators.required],
            icon: ['home', Validators.required],
            color: ['primary']
        });
    }

    onIconSelected(icon: any) {
        this.formGroup.get('icon').setValue(icon.Name);
        this.formGroup.get('icon').markAsDirty();
    }
}
```

```html
<form [formGroup]="formGroup">
    <app-input-control [field]="{
        form: formGroup,
        id: 'name',
        type: 'text',
        label: 'Name'
    }"></app-input-control>

    <div class="icon-selection">
        <ion-label>Select Icon</ion-label>
        <app-icon-picker
            [icon]="formGroup.get('icon').value"
            [color]="formGroup.get('color').value"
            (selected)="onIconSelected($event)">
        </app-icon-picker>
    </div>
</form>
```

---

## 🎨 **Available Icons**

### **Business Icons**
```typescript
const businessIcons = [
    { Name: 'business', Tags: 'business' },
    { Name: 'briefcase', Tags: 'briefcase,folder,organize' },
    { Name: 'contract', Tags: 'contract' },
    { Name: 'industry', Tags: 'industry' },
    { Name: 'warehouse', Tags: 'warehouse,inventory,wms' },
    { Name: 'storefront', Tags: 'storefront' }
];
```

### **Navigation Icons**
```typescript
const navigationIcons = [
    { Name: 'home', Tags: 'home,house' },
    { Name: 'arrow-back', Tags: 'arrow,back,chevron,left,navigation' },
    { Name: 'arrow-forward', Tags: 'arrow,chevron,forward,navigation,right' },
    { Name: 'menu', Tags: 'menu' },
    { Name: 'close', Tags: 'circle,close,delete,remove' }
];
```

### **Action Icons**
```typescript
const actionIcons = [
    { Name: 'add', Tags: 'add,circle,include,invite,plus' },
    { Name: 'create', Tags: 'create,edit' },
    { Name: 'trash', Tags: 'close,delete,remove,trash' },
    { Name: 'save', Tags: 'save' },
    { Name: 'refresh', Tags: 'circle,refresh,reload,renew,reset' }
];
```

### **Status Icons**
```typescript
const statusIcons = [
    { Name: 'checkmark', Tags: 'checkmark,circle' },
    { Name: 'close-circle', Tags: 'circle,close,delete,remove' },
    { Name: 'alert', Tags: '!,alert,attention,exclamation,notice,warning' },
    { Name: 'information', Tags: 'circle,help,information,knowledge' }
];
```

---

## 🔍 **Search Functionality**

### **Search by Name**
```html
<div class="icon-search">
    <ion-searchbar
        [(ngModel)]="keyword"
        placeholder="Search icons..."
        (ionInput)="filterIcons($event)">
    </ion-searchbar>
</div>

<div class="icon-grid">
    <div 
        *ngFor="let item of filteredIcons" 
        class="icon-item"
        [class.selected]="item.Name === selectedIcon"
        (click)="selectIcon(item)">
        <ion-icon [name]="item.Name"></ion-icon>
        <span>{{ item.Name }}</span>
    </div>
</div>
```

### **Advanced Search Implementation**
```typescript
export class AdvancedIconPicker {
    allIcons: any[] = [];
    filteredIcons: any[] = [];
    searchKeyword = '';
    selectedCategory = 'all';

    categories = [
        { value: 'all', label: 'All Icons' },
        { value: 'business', label: 'Business' },
        { value: 'navigation', label: 'Navigation' },
        { value: 'action', label: 'Actions' },
        { value: 'status', label: 'Status' }
    ];

    ngOnInit() {
        this.allIcons = this.loadAllIcons();
        this.filteredIcons = [...this.allIcons];
    }

    filterIcons(event: any) {
        this.searchKeyword = event.target.value.toLowerCase();
        this.applyFilters();
    }

    onCategoryChange(category: string) {
        this.selectedCategory = category;
        this.applyFilters();
    }

    applyFilters() {
        let filtered = [...this.allIcons];

        // Filter by category
        if (this.selectedCategory !== 'all') {
            filtered = filtered.filter(icon => 
                icon.Tags.includes(this.selectedCategory)
            );
        }

        // Filter by search keyword
        if (this.searchKeyword) {
            filtered = filtered.filter(icon => 
                icon.Name.toLowerCase().includes(this.searchKeyword) ||
                icon.Tags.toLowerCase().includes(this.searchKeyword)
            );
        }

        this.filteredIcons = filtered;
    }

    selectIcon(icon: any) {
        this.selectedIcon = icon.Name;
        this.selected.emit(icon);
    }
}
```

---

## 🎨 **UI Customization**

### **Grid Layout**
```scss
// Component SCSS
.icon-picker {
    .icon-search {
        margin-bottom: 16px;
        
        ion-searchbar {
            --background: var(--ion-color-light);
            --border-radius: 8px;
        }
    }
    
    .icon-categories {
        margin-bottom: 16px;
        
        ion-segment {
            --background: var(--ion-color-light);
        }
    }
    
    .icon-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
        gap: 12px;
        max-height: 400px;
        overflow-y: auto;
        
        .icon-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 12px;
            border: 2px solid transparent;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.2s ease;
            
            &:hover {
                background: var(--ion-color-light);
                border-color: var(--ion-color-medium);
            }
            
            &.selected {
                background: var(--ion-color-primary-tint);
                border-color: var(--ion-color-primary);
            }
            
            ion-icon {
                font-size: 24px;
                margin-bottom: 4px;
            }
            
            span {
                font-size: 0.75rem;
                text-align: center;
                word-break: break-word;
            }
        }
    }
}
```

### **Modal Integration**
```typescript
export class IconPickerModal {
    selectedIcon: string;
    
    constructor(private modalController: ModalController) {}

    async openIconPicker() {
        const modal = await this.modalController.create({
            component: IconPickerModalPage,
            cssClass: 'icon-picker-modal',
            componentProps: {
                selectedIcon: this.selectedIcon
            }
        });

        await modal.present();
        
        const { data } = await modal.onWillDismiss();
        if (data && data.icon) {
            this.selectedIcon = data.icon.Name;
            this.onIconSelected(data.icon);
        }
    }
}
```

```html
<!-- Modal trigger -->
<ion-item button (click)="openIconPicker()">
    <ion-icon [name]="selectedIcon || 'add'" slot="start"></ion-icon>
    <ion-label>
        <h3>Icon</h3>
        <p>{{ selectedIcon || 'Select an icon' }}</p>
    </ion-label>
    <ion-icon name="chevron-forward" slot="end"></ion-icon>
</ion-item>
```

---

## 📱 **Mobile Optimization**

### **Touch-Friendly Design**
```scss
// Mobile-specific styles
@media (max-width: 768px) {
    .icon-picker {
        .icon-grid {
            grid-template-columns: repeat(auto-fill, minmax(60px, 1fr));
            gap: 8px;
            
            .icon-item {
                padding: 8px;
                min-height: 60px;
                
                ion-icon {
                    font-size: 20px;
                }
                
                span {
                    font-size: 0.7rem;
                }
            }
        }
    }
}
```

### **Responsive Modal**
```typescript
export class ResponsiveIconPicker {
    async openIconPicker() {
        const isMobile = window.innerWidth < 768;
        
        const modal = await this.modalController.create({
            component: IconPickerModalPage,
            cssClass: isMobile ? 'fullscreen-modal' : 'icon-picker-modal',
            breakpoints: isMobile ? [0, 1] : [0, 0.5, 1],
            initialBreakpoint: isMobile ? 1 : 0.5
        });

        await modal.present();
    }
}
```

---

## 🔧 **Advanced Features**

### **Custom Icon Sets**
```typescript
export class CustomIconPicker {
    customIcons = [
        { Name: 'logo-ca', Tags: 'ca,codeart,logo' },
        { Name: 'logo-codeart', Tags: 'ca,codeart,logo' },
        { Name: 'quotation', Tags: 'price,quotation,money,document' },
        { Name: 'purchase-request', Tags: 'price,quotation,money,document' }
    ];

    ionicIcons = [
        { Name: 'home', Tags: 'home,house' },
        { Name: 'settings', Tags: 'options,settings' },
        // ... other Ionic icons
    ];

    getAllIcons() {
        return [
            ...this.customIcons,
            ...this.ionicIcons
        ];
    }
}
```

### **Icon Preview**
```html
<div class="icon-preview">
    <div class="preview-section">
        <h4>Preview</h4>
        <div class="preview-icons">
            <ion-icon 
                [name]="selectedIcon" 
                class="preview-small">
            </ion-icon>
            <ion-icon 
                [name]="selectedIcon" 
                class="preview-medium">
            </ion-icon>
            <ion-icon 
                [name]="selectedIcon" 
                class="preview-large">
            </ion-icon>
        </div>
    </div>
    
    <div class="icon-info">
        <h4>Icon Information</h4>
        <p><strong>Name:</strong> {{ selectedIcon }}</p>
        <p><strong>Tags:</strong> {{ getIconTags(selectedIcon) }}</p>
        <p><strong>Usage:</strong> &lt;ion-icon name="{{ selectedIcon }}"&gt;&lt;/ion-icon&gt;</p>
    </div>
</div>
```

### **Favorites System**
```typescript
export class IconPickerWithFavorites {
    favoriteIcons: string[] = [];

    async loadFavorites() {
        try {
            const saved = await this.env.getStorage('favoriteIcons');
            this.favoriteIcons = saved || [];
        } catch (error) {
            console.error('Load favorites error:', error);
        }
    }

    async toggleFavorite(iconName: string) {
        const index = this.favoriteIcons.indexOf(iconName);
        
        if (index > -1) {
            this.favoriteIcons.splice(index, 1);
        } else {
            this.favoriteIcons.push(iconName);
        }

        try {
            await this.env.setStorage('favoriteIcons', this.favoriteIcons);
        } catch (error) {
            console.error('Save favorites error:', error);
        }
    }

    isFavorite(iconName: string): boolean {
        return this.favoriteIcons.includes(iconName);
    }
}
```

---

## 🔧 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Đúng - Virtual scrolling for large icon sets
export class OptimizedIconPicker {
    visibleIcons: any[] = [];
    allIcons: any[] = [];
    
    loadVisibleIcons() {
        // Only render visible icons
        this.visibleIcons = this.allIcons.slice(0, 50);
    }
    
    onScroll(event: any) {
        // Load more icons on scroll
        if (event.target.scrollTop + event.target.clientHeight >= event.target.scrollHeight - 100) {
            this.loadMoreIcons();
        }
    }
}
```

### **2. Accessibility**
```html
<!-- ✅ Đúng - Accessible icon picker -->
<div 
    class="icon-item"
    role="button"
    tabindex="0"
    [attr.aria-label]="'Select ' + item.Name + ' icon'"
    [attr.aria-pressed]="item.Name === selectedIcon"
    (click)="selectIcon(item)"
    (keydown.enter)="selectIcon(item)"
    (keydown.space)="selectIcon(item)">
    <ion-icon [name]="item.Name"></ion-icon>
    <span>{{ item.Name }}</span>
</div>
```

### **3. Error Handling**
```typescript
// ✅ Đúng - Handle missing icons
getIconSrc(iconName: string): string {
    try {
        // Check if icon exists
        return `assets/icons/${iconName}.svg`;
    } catch (error) {
        // Fallback to default icon
        return 'assets/icons/default.svg';
    }
}
```

---

## 🚨 **Common Use Cases**

### **1. Menu Configuration**
```typescript
export class MenuConfigPage {
    menuItems = [
        { name: 'Dashboard', icon: 'analytics', route: '/dashboard' },
        { name: 'Users', icon: 'people', route: '/users' },
        { name: 'Settings', icon: 'settings', route: '/settings' }
    ];

    updateMenuIcon(index: number, icon: any) {
        this.menuItems[index].icon = icon.Name;
        this.saveMenuConfiguration();
    }
}
```

### **2. Status Indicators**
```typescript
export class StatusConfigPage {
    statusTypes = [
        { name: 'Active', icon: 'checkmark-circle', color: 'success' },
        { name: 'Pending', icon: 'time', color: 'warning' },
        { name: 'Inactive', icon: 'close-circle', color: 'danger' }
    ];
}
```

### **3. Category Management**
```typescript
export class CategoryManagementPage {
    categories = [];

    addCategory() {
        const newCategory = {
            name: '',
            icon: 'folder',
            color: 'primary'
        };
        this.categories.push(newCategory);
    }

    updateCategoryIcon(category: any, icon: any) {
        category.icon = icon.Name;
        this.saveCategories();
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
