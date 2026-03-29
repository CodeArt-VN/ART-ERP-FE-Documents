# InputControl Component Documentation

## 📋 **Tổng quan**

`InputControlComponent` là universal input component hỗ trợ nhiều loại input khác nhau bao gồm text, select, date, tree view, branch picker, color picker, icon picker và nhiều hơn nữa.

**Selector**: `app-input-control`  
**Location**: `src/app/components/controls/input-control.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() field: InputControlField;       // Complete field configuration
@Input() form: FormGroup;                // Form group reference
@Input() type: string = 'text';          // Input type
@Input() id: string;                     // Form control ID
@Input() secondaryId: string;            // Secondary control ID (for ranges)
@Input() label: string;                  // Display label
@Input() placeholder: string;            // Placeholder text
@Input() dataSource: any[];              // Data for selects/dropdowns
@Input() bindValue: string = 'Code';     // Property to bind as value
@Input() bindLabel: string = 'Name';     // Property to display as label
@Input() multiple: boolean = false;      // Multi-selection
@Input() clearable: boolean = false;     // Show clear button
@Input() color: string = 'dark';         // Color theme
@Input() appendTo: string;               // Dropdown append target
```

### **Tree Configuration**
```typescript
@Input() isTree: boolean = false;        // Enable tree view
@Input() isCollapsed: boolean;           // Default collapse state
@Input() rootCollapsed: boolean = true;  // Root level collapse
@Input() searchFn: Function;             // Custom search function
```

### **Branch Configuration**
```typescript
@Input() selectedBranch: number;         // Current branch
@Input() showingType: string;            // Branch type filter
@Input() showingDisable: boolean;        // Show disabled branches
@Input() showingMode: string;            // Display mode
```

### **Output Events**
```typescript
@Output() change = new EventEmitter();       // Value changes
@Output() inputChange = new EventEmitter();  // Input events
@Output() select = new EventEmitter();       // Selection events
@Output() nav = new EventEmitter();          // Navigation events
```

---

## 🚀 **Basic Input Types**

### **Text Inputs**
```html
<!-- Basic text -->
<app-input-control [field]="{
    form: formGroup,
    id: 'name',
    type: 'text',
    label: 'Full Name',
    placeholder: 'Enter your name'
}"></app-input-control>

<!-- Email -->
<app-input-control [field]="{
    form: formGroup,
    id: 'email',
    type: 'email',
    label: 'Email Address',
    placeholder: 'Enter email'
}"></app-input-control>

<!-- Password -->
<app-input-control [field]="{
    form: formGroup,
    id: 'password',
    type: 'password',
    label: 'Password',
    placeholder: 'Enter password'
}"></app-input-control>

<!-- Number -->
<app-input-control [field]="{
    form: formGroup,
    id: 'age',
    type: 'number',
    label: 'Age',
    placeholder: 'Enter age'
}"></app-input-control>

<!-- Textarea -->
<app-input-control [field]="{
    form: formGroup,
    id: 'description',
    type: 'textarea',
    label: 'Description',
    placeholder: 'Enter description'
}"></app-input-control>
```

### **Selection Controls**
```html
<!-- Basic select -->
<app-input-control [field]="{
    form: formGroup,
    id: 'category',
    type: 'ng-select',
    label: 'Category',
    dataSource: categories,
    bindValue: 'Id',
    bindLabel: 'Name',
    clearable: true
}"></app-input-control>

<!-- Multi-select -->
<app-input-control [field]="{
    form: formGroup,
    id: 'tags',
    type: 'ng-select',
    label: 'Tags',
    dataSource: tags,
    bindValue: 'Id',
    bindLabel: 'Name',
    multiple: true,
    clearable: true
}"></app-input-control>

<!-- Async select -->
<app-input-control [field]="{
    form: formGroup,
    id: 'user',
    type: 'ng-select-async',
    label: 'User',
    dataSource: userDataSource,
    bindValue: 'Id',
    bindLabel: 'FullName'
}"></app-input-control>
```

---

## 🌳 **Tree View Controls**

### **Basic Tree Select**
```typescript
// Component
export class CategoryFormPage {
    categories: any[] = [];
    
    async loadCategories() {
        const result = await this.categoryService.read();
        this.categories = result.data;
    }
}
```

```html
<app-input-control [field]="{
    form: formGroup,
    id: 'categoryId',
    type: 'ng-select',
    label: 'Category',
    dataSource: categories,
    bindValue: 'Id',
    bindLabel: 'Name',
    treeConfig: {
        isTree: true,
        isCollapsed: true,
        searchFnDefault: true
    }
}"></app-input-control>
```

### **Advanced Tree Configuration**
```html
<app-input-control [field]="{
    form: formGroup,
    id: 'departmentId',
    type: 'ng-select',
    label: 'Department',
    dataSource: departments,
    bindValue: 'Id',
    bindLabel: 'Name',
    treeConfig: {
        isTree: true,
        isCollapsed: false,
        rootCollapsed: false,
        searchFnDefault: true
    }
}"></app-input-control>
```

---

## 🏢 **Branch Controls**

### **Branch Picker**
```html
<!-- Basic branch select -->
<app-input-control [field]="{
    form: formGroup,
    id: 'branchId',
    type: 'ng-select-branch',
    label: 'Branch',
    branchConfig: {
        selectedBranch: env.selectedBranch,
        showingType: 'Department',
        showingDisable: false,
        showingMode: 'showSelectedAndChildren'
    }
}"></app-input-control>

<!-- Warehouse only -->
<app-input-control [field]="{
    form: formGroup,
    id: 'warehouseId',
    type: 'ng-select-branch',
    label: 'Warehouse',
    branchConfig: {
        selectedBranch: env.selectedBranch,
        showingType: 'Warehouse',
        showingMode: 'showAll'
    }
}"></app-input-control>

<!-- Exclude certain types -->
<app-input-control [field]="{
    form: formGroup,
    id: 'assignedBranch',
    type: 'ng-select-branch',
    label: 'Assigned Branch',
    branchConfig: {
        selectedBranch: env.selectedBranch,
        showingType: 'ne_[Warehouse,TitlePosition]',
        showingDisable: true
    }
}"></app-input-control>
```

---

## 🎨 **UI Controls**

### **Color Picker**
```html
<app-input-control [field]="{
    form: formGroup,
    id: 'themeColor',
    type: 'color',
    label: 'Theme Color',
    color: 'primary'
}"></app-input-control>
```

### **Icon Picker**
```html
<app-input-control [field]="{
    form: formGroup,
    id: 'menuIcon',
    type: 'icon',
    label: 'Menu Icon'
}"></app-input-control>

<!-- Icon with color -->
<app-input-control [field]="{
    form: formGroup,
    id: 'statusIcon',
    type: 'icon-color',
    label: 'Status Icon'
}"></app-input-control>
```

---

## 📅 **Date & Time Controls**

### **Date Picker**
```html
<app-input-control [field]="{
    form: formGroup,
    id: 'birthDate',
    type: 'date',
    label: 'Birth Date'
}"></app-input-control>
```

### **Time Frame Picker**
```html
<app-input-control [field]="{
    form: formGroup,
    id: 'reportPeriod',
    type: 'time-frame',
    label: 'Report Period',
    secondaryId: 'reportPeriodTo'
}"></app-input-control>
```

### **Custom Date Range**
```typescript
// Component setup for date range
this.formGroup = this.formBuilder.group({
    dateFrom: this.formBuilder.group({
        Type: ['Relative'],
        IsPastDate: [true],
        Period: ['Day'],
        Amount: [7],
        Value: [null],
        IsNull: [false]
    }),
    dateTo: this.formBuilder.group({
        Type: ['Relative'],
        IsPastDate: [false],
        Period: ['Day'],
        Amount: [0],
        Value: [null],
        IsNull: [false]
    })
});
```

---

## 🔧 **Advanced Features**

### **Formula Editor**
```html
<app-input-control [field]="{
    form: formGroup,
    id: 'calculationFormula',
    type: 'formula',
    label: 'Calculation Formula',
    dataSource: availableFields
}"></app-input-control>
```

### **Async Data Source**
```typescript
// Component
export class AsyncSelectPage {
    userDataSource = this.buildSelectDataSource(
        (term: string) => this.userService.search({ keyword: term }),
        false
    );

    buildSelectDataSource(searchFunction: Function, buildFlatTree = false) {
        return this.formManagementService.createSelectDataSource(searchFunction, buildFlatTree);
    }
}
```

```html
<app-input-control [field]="{
    form: formGroup,
    id: 'assignedUser',
    type: 'ng-select-async',
    label: 'Assigned User',
    dataSource: userDataSource,
    bindValue: 'Id',
    bindLabel: 'FullName'
}"></app-input-control>
```

### **Custom Search Function**
```typescript
// Component
export class CustomSearchPage {
    customSearchFn = (term: string, item: any) => {
        if (!term) return true;
        
        const searchTerm = term.toLowerCase();
        return item.Name.toLowerCase().includes(searchTerm) ||
               item.Code.toLowerCase().includes(searchTerm) ||
               item.Description?.toLowerCase().includes(searchTerm);
    };
}
```

```html
<app-input-control [field]="{
    form: formGroup,
    id: 'product',
    type: 'ng-select',
    label: 'Product',
    dataSource: products,
    treeConfig: {
        isTree: true,
        searchFn: customSearchFn
    }
}"></app-input-control>
```

---

## 📱 **Mobile Optimization**

### **Touch-Friendly Controls**
```html
<!-- Mobile-optimized select -->
<app-input-control 
    [field]="{
        form: formGroup,
        id: 'category',
        type: 'ng-select',
        label: 'Category',
        dataSource: categories,
        appendTo: 'body'
    }"
    class="mobile-select">
</app-input-control>
```

### **Responsive Configuration**
```scss
// Mobile styles
@media (max-width: 768px) {
    app-input-control {
        .ng-select {
            --min-height: 44px;
            font-size: 16px; // Prevent zoom on iOS
        }
        
        .tree-select {
            .ng-option {
                padding: 12px 16px;
                min-height: 44px;
            }
        }
    }
}
```

---

## 🎛️ **Event Handling**

### **Value Changes**
```typescript
export class EventHandlingPage {
    onFieldChange(event: any) {
        console.log('Field changed:', event);
        
        // Handle specific field changes
        if (event.field === 'category') {
            this.loadSubCategories(event.value);
        }
    }

    onInputChange(event: any) {
        console.log('Input change:', event);
        
        // Real-time validation or formatting
        if (event.field === 'phone') {
            this.formatPhoneNumber(event.value);
        }
    }

    onSelect(event: any) {
        console.log('Selection:', event);
        
        // Handle selection events
        this.loadRelatedData(event.value);
    }

    async loadSubCategories(categoryId: number) {
        if (!categoryId) {
            this.subCategories = [];
            return;
        }

        try {
            this.subCategories = await this.categoryService.getSubCategories(categoryId);
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

```html
<app-input-control 
    [field]="{
        form: formGroup,
        id: 'category',
        type: 'ng-select',
        label: 'Category',
        dataSource: categories
    }"
    (change)="onFieldChange($event)"
    (inputChange)="onInputChange($event)"
    (select)="onSelect($event)">
</app-input-control>
```

---

## 🎨 **Styling & Customization**

### **Custom Styling**
```scss
// Component SCSS
app-input-control {
    .custom-select {
        .ng-select {
            --border-radius: 8px;
            --border-color: var(--ion-color-medium);
            
            &.ng-select-focused {
                --border-color: var(--ion-color-primary);
                box-shadow: 0 0 0 2px rgba(var(--ion-color-primary-rgb), 0.2);
            }
        }
        
        .ng-option {
            &.ng-option-highlighted {
                background-color: var(--ion-color-primary-tint);
                color: var(--ion-color-primary-contrast);
            }
            
            &.ng-option-selected {
                background-color: var(--ion-color-primary);
                color: var(--ion-color-primary-contrast);
            }
        }
    }
    
    .tree-select {
        .ng-option {
            &.tree-level-1 { padding-left: 20px; }
            &.tree-level-2 { padding-left: 40px; }
            &.tree-level-3 { padding-left: 60px; }
        }
    }
}
```

### **Theme Variants**
```html
<!-- Primary theme -->
<app-input-control 
    [field]="fieldConfig"
    class="primary-theme">
</app-input-control>

<!-- Success theme -->
<app-input-control 
    [field]="fieldConfig"
    class="success-theme">
</app-input-control>

<!-- Dark theme -->
<app-input-control 
    [field]="fieldConfig"
    class="dark-theme">
</app-input-control>
```

---

## 🔧 **Best Practices**

### **1. Field Configuration**
```typescript
// ✅ Đúng - Complete configuration
const fieldConfig: InputControlField = {
    form: this.formGroup,
    id: 'category',
    type: 'ng-select',
    label: 'PRODUCT_CATEGORY',
    placeholder: 'SELECT_CATEGORY',
    dataSource: this.categories,
    bindValue: 'Id',
    bindLabel: 'Name',
    clearable: true,
    multiple: false
};

// ❌ Sai - Minimal configuration
const fieldConfig = {
    form: this.formGroup,
    id: 'category',
    type: 'ng-select'
};
```

### **2. Performance Optimization**
```typescript
// ✅ Đúng - Async data loading
buildAsyncDataSource(searchFn: Function) {
    return this.formManagementService.createSelectDataSource(searchFn, false);
}

// ✅ Đúng - Debounced search
searchUsers = debounceTime(300)(
    (term: string) => this.userService.search({ keyword: term })
);

// ❌ Sai - Loading all data upfront
async loadAllUsers() {
    this.users = await this.userService.getAll(); // Có thể chậm
}
```

### **3. Error Handling**
```typescript
// ✅ Đúng - Proper error handling
async loadDataSource() {
    try {
        this.loading = true;
        this.categories = await this.categoryService.read();
    } catch (error) {
        this.env.showErrorMessage(error);
        this.categories = [];
    } finally {
        this.loading = false;
    }
}
```

---

## 🚨 **Common Patterns**

### **1. Dependent Dropdowns**
```typescript
export class DependentDropdownPage {
    categories: any[] = [];
    subCategories: any[] = [];
    products: any[] = [];

    ngOnInit() {
        this.loadCategories();
        
        // Watch category changes
        this.formGroup.get('categoryId').valueChanges.subscribe(categoryId => {
            this.loadSubCategories(categoryId);
            this.formGroup.get('subCategoryId').setValue(null);
            this.formGroup.get('productId').setValue(null);
        });

        // Watch subcategory changes
        this.formGroup.get('subCategoryId').valueChanges.subscribe(subCategoryId => {
            this.loadProducts(subCategoryId);
            this.formGroup.get('productId').setValue(null);
        });
    }
}
```

### **2. Dynamic Form Fields**
```typescript
export class DynamicFieldsPage {
    fieldConfigs: InputControlField[] = [];

    async loadFormSchema(formType: string) {
        const schema = await this.schemaService.getFormSchema(formType);
        this.buildFieldConfigs(schema);
    }

    buildFieldConfigs(schema: any) {
        this.fieldConfigs = schema.fields.map(field => ({
            form: this.formGroup,
            id: field.id,
            type: field.type,
            label: field.label,
            dataSource: field.options,
            bindValue: field.bindValue || 'value',
            bindLabel: field.bindLabel || 'label'
        }));
    }
}
```

### **3. Conditional Visibility**
```typescript
export class ConditionalFieldsPage {
    showAdvancedFields = false;

    ngOnInit() {
        this.formGroup.get('userType').valueChanges.subscribe(userType => {
            this.showAdvancedFields = userType === 'admin';
            
            if (this.showAdvancedFields) {
                this.addAdvancedValidators();
            } else {
                this.removeAdvancedValidators();
            }
        });
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
