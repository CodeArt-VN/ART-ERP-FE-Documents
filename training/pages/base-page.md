# PageBase Documentation - ART-ERP-FE

## 📋 **Mục đích tài liệu**

Tài liệu này mô tả chi tiết `PageBase` - abstract base class cho tất cả pages/components trong ứng dụng ART-ERP-FE. Class này cung cấp các chức năng chuẩn cho data management, form handling, CRUD operations và UI interactions.

---

## 🏗️ **Tổng quan PageBase**

`PageBase` là abstract class được extend bởi tất cả pages trong ứng dụng. Nó cung cấp:

- **Data Management** - Loading, caching, pagination
- **Form Management** - Validation, auto-save, dirty tracking
- **CRUD Operations** - Create, Read, Update, Delete với UI feedback
- **Selection Management** - Multi-select với bulk operations
- **Navigation & Events** - Routing và event handling
- **Permission Management** - Role-based access control
- **Tree View Support** - Hierarchical data display

---

## 🔧 **Page Setup và Extending**

### **Extending PageBase**
```typescript
import { Component } from '@angular/core';
import { PageBase } from 'src/app/page-base';

@Component({
    selector: 'app-user-list',
    templateUrl: './user-list.page.html',
    styleUrls: ['./user-list.page.scss']
})
export class UserListPage extends PageBase {
    constructor(
        public pageProvider: UserService,
        public env: EnvService,
        public navCtrl: NavController,
        public route: ActivatedRoute,
        public modalController: ModalController,
        public alertCtrl: AlertController,
        public popoverCtrl: PopoverController,
        public loadingController: LoadingController,
        public cdr: ChangeDetectorRef
    ) {
        super();
        
        // Page configuration
        this.pageConfig = {
            ...this.pageConfig,
            pageName: 'user-list',
            pageTitle: 'User Management',
            isDetailPage: false,
            ShowAdd: true,
            ShowDelete: true,
            ShowExport: true
        };
    }
}
```

### **Page Properties**
```typescript
// Core properties
item: any = null;                    // Current item (detail page)
items: any[] = [];                   // List items (list page)
selectedItems: any[] = [];           // Selected items for bulk operations
query: any = {};                     // Query parameters for API calls
pageConfig: PageConfig = {};         // Page configuration and permissions

// Form properties
formGroup: FormGroup;                // Reactive form group
submitAttempt: boolean = false;      // Prevent double submission
isAutoSave: boolean = true;          // Enable auto-save functionality

// Services (injected)
env: EnvService;                     // Environment service
pageProvider: any;                   // Data provider service
navCtrl: NavController;              // Navigation controller
modalController: ModalController;    // Modal controller
```

---

## 📊 **Data Loading Functions**

### **loadData(event = null, forceReload = false)**
Load dữ liệu cho page (list hoặc detail).

```typescript
// Basic data loading
async ionViewDidEnter() {
    await this.env.ready;
    this.loadData();
}

// Force reload data
refreshData() {
    this.loadData(null, true);
}

// Load with pagination (infinite scroll)
loadMore(event: any) {
    this.loadData(event);
}

// Override loadData cho custom logic
loadData(event = null, forceReload = false) {
    // Custom pre-processing
    this.query.Status = 'active';
    
    // Call parent method
    super.loadData(event, forceReload);
}
```

### **preLoadData(event = null)**
Clear data và load lại từ đầu.

```typescript
// Refresh toàn bộ data
refresh() {
    this.preLoadData();
}

// Clear và reload khi filter thay đổi
onFilterChange() {
    this.selectedItems = [];
    this.preLoadData();
}
```

### **loadAnItem(event = null)**
Load một item cụ thể cho detail page.

```typescript
// Tự động được gọi khi pageConfig.isDetailPage = true
// Override để custom logic
loadAnItem(event = null) {
    // Custom pre-processing
    if (this.id) {
        this.query.IncludeRelatedData = true;
    }
    
    // Call parent method
    super.loadAnItem(event);
}
```

### **loadedData(event = null, ignoredFromGroup = false)**
Callback sau khi load data xong.

```typescript
// Override để custom post-processing
loadedData(event = null, ignoredFromGroup = false) {
    // Call parent method first
    super.loadedData(event, ignoredFromGroup);
    
    // Custom post-processing
    if (this.pageConfig.isDetailPage && this.item) {
        this.loadRelatedData();
    } else {
        this.calculateSummary();
    }
}

async loadRelatedData() {
    // Load additional data for detail page
    this.item.Orders = await this.orderService.read({
        CustomerId: this.item.Id
    });
}

calculateSummary() {
    // Calculate summary for list page
    this.totalAmount = this.items.reduce((sum, item) => sum + item.Amount, 0);
}
```

---

## 🔍 **Search và Filter Functions**

### **search(ev)**
Tìm kiếm theo keyword.

```typescript
// Tự động được gọi từ search input
// Override để custom search logic
search(ev) {
    const keyword = ev.target.value;
    
    // Custom validation
    if (keyword.length < 2 && keyword !== '') {
        return;
    }
    
    // Custom query modification
    this.query.SearchFields = ['Name', 'Email', 'Phone'];
    
    // Call parent method
    super.search(ev);
}

// Programmatic search
performSearch(keyword: string) {
    this.query.Keyword = keyword;
    this.query.Skip = 0;
    this.pageConfig.isEndOfData = false;
    this.loadData('search');
}
```

### **onDatatableFilter(e)**
Handle filter từ datatable component.

```typescript
// Tự động được gọi từ datatable filter
onDatatableFilter(e) {
    // Custom filter processing
    if (e.query.DateFrom) {
        e.query.DateFrom = this.formatDate(e.query.DateFrom);
    }
    
    // Call parent method
    super.onDatatableFilter(e);
}

// Custom filter method
applyCustomFilter(filterData: any) {
    Object.assign(this.query, filterData);
    this.refresh();
}
```

### **openAdvanceFilter(callback?: (data: any) => void)**
Mở advanced filter modal.

```typescript
// Open advanced filter
async openFilter() {
    await this.openAdvanceFilter((data) => {
        // Custom callback after filter applied
        this.processFilteredData(data);
    });
}

// Override để custom filter config
getAdvaneFilterConfig() {
    if (!this.query._AdvanceConfig) {
        this.query._AdvanceConfig = {
            // Custom filter configuration
            Schema: {
                Code: 'CustomUserFilter',
                Type: 'Form'
            },
            // ... other config
        };
    }
}
```

---

## ✏️ **Form Management Functions**

### **saveChange(publishEventCode?)**
Lưu thay đổi form (full form save).

```typescript
// Basic save
async save() {
    try {
        const result = await this.saveChange();
        console.log('Saved item ID:', result);
    } catch (error) {
        console.error('Save failed:', error);
    }
}

// Save với custom event
async saveWithEvent() {
    await this.saveChange('custom-event-code');
}

// Override để custom save logic
saveChange(publishEventCode = this.pageConfig.pageName) {
    // Custom validation
    if (!this.validateCustomRules()) {
        this.env.showMessage('CUSTOM_VALIDATION_ERROR', 'danger');
        return Promise.reject('Custom validation failed');
    }
    
    // Custom data processing
    this.item.LastModified = new Date();
    
    // Call parent method
    return super.saveChange(publishEventCode);
}
```

### **saveChange2(form?, publishEventCode?, provider?)**
Lưu chỉ dirty fields (optimized save).

```typescript
// Auto-save với dirty tracking
setupAutoSave() {
    this.formGroup.valueChanges.pipe(
        debounceTime(1000),
        distinctUntilChanged()
    ).subscribe(() => {
        if (this.formGroup.dirty && this.pageConfig.systemConfig?.IsAutoSave) {
            this.saveChange2();
        }
    });
}

// Save specific form
async saveSubForm(subForm: FormGroup) {
    try {
        await this.saveChange2(subForm, 'sub-form-saved', this.subFormProvider);
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

### **setFormValues(data, form?, instantly?, forceSave?)**
Set giá trị cho form với auto-save support.

```typescript
// Set form values với auto-save
updateFormData(newData: any) {
    this.setFormValues(newData, this.formGroup, false, false);
}

// Set values và save ngay lập tức
updateAndSave(newData: any) {
    this.setFormValues(newData, this.formGroup, true, true);
}

// Batch update multiple fields
batchUpdateFields(updates: any[]) {
    updates.forEach(update => {
        this.setFormValues(update.data, update.form, false, false);
    });
    
    // Save all changes
    this.saveChange2();
}
```

---

## 🔄 **CRUD Operations Functions**

### **add()**
Thêm item mới.

```typescript
// Basic add (navigate to detail page)
addNew() {
    this.add(); // Navigate to /page-name/0
}

// Override để custom add logic
add() {
    // Custom pre-add logic
    if (!this.checkAddPermission()) {
        this.env.showMessage('NO_ADD_PERMISSION', 'danger');
        return;
    }
    
    // Call parent method
    super.add();
}

// Add với modal
async addWithModal() {
    const modal = await this.modalController.create({
        component: UserDetailModal,
        componentProps: {
            item: { Id: 0, ...this.getDefaultValues() }
        }
    });
    
    modal.onDidDismiss().then(result => {
        if (result.data) {
            this.refresh();
        }
    });
    
    await modal.present();
}
```

### **delete(publishEventCode?)**
Xóa items đã chọn.

```typescript
// Delete selected items
async deleteSelected() {
    if (this.selectedItems.length === 0) {
        this.env.showMessage('NO_ITEMS_SELECTED', 'warning');
        return;
    }
    
    await this.delete();
}

// Override để custom delete logic
delete(publishEventCode = this.pageConfig.pageName) {
    // Custom validation
    const hasActiveItems = this.selectedItems.some(item => item.Status === 'active');
    if (hasActiveItems) {
        this.env.showMessage('CANNOT_DELETE_ACTIVE_ITEMS', 'danger');
        return;
    }
    
    // Call parent method
    super.delete(publishEventCode);
}

// Soft delete với custom logic
async softDelete(items: any[]) {
    try {
        await this.pageProvider.disable(items, true);
        this.env.showMessage('ITEMS_ARCHIVED', 'success');
        this.removeSelectedItems();
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

---

## 🔄 **Approval Workflow Functions**

### **submitForApproval()**
Gửi items để phê duyệt.

```typescript
// Submit selected items
async submitSelected() {
    if (!this.validateForSubmission()) {
        return;
    }
    
    await this.submitForApproval();
}

// Override để custom submit logic
submitForApproval() {
    // Custom validation
    const invalidItems = this.selectedItems.filter(item => !item.IsComplete);
    if (invalidItems.length > 0) {
        this.env.showMessage('INCOMPLETE_ITEMS_CANNOT_SUBMIT', 'danger');
        return;
    }
    
    // Call parent method
    super.submitForApproval();
}
```

### **approve() / disapprove()**
Phê duyệt/từ chối items.

```typescript
// Approve selected items
async approveSelected() {
    await this.approve();
}

// Disapprove với lý do
async disapproveWithReason() {
    try {
        const reason = await this.env.showPrompt(
            'ENTER_DISAPPROVAL_REASON',
            'Please provide reason',
            'Disapprove Items',
            'Submit',
            'Cancel',
            [{ name: 'reason', type: 'textarea' }]
        );
        
        // Add reason to items
        this.selectedItems.forEach(item => {
            item.DisapprovalReason = reason.reason;
        });
        
        await this.disapprove();
    } catch (error) {
        // User cancelled
    }
}
```

---

## 📁 **File Operations Functions**

### **import(event) / export()**
Import/Export dữ liệu.

```typescript
// Import file
async onFileImport(event: any) {
    await this.import(event);
}

// Export với custom query
async exportFiltered() {
    // Add export-specific parameters
    const exportQuery = {
        ...this.query,
        ExportFormat: 'excel',
        IncludeHeaders: true
    };
    
    this.query = exportQuery;
    await this.export();
}

// Override import để custom processing
async import(event) {
    // Custom validation
    const file = event.target.files[0];
    if (!this.validateImportFile(file)) {
        return;
    }
    
    // Call parent method
    super.import(event);
}

validateImportFile(file: File): boolean {
    const allowedTypes = ['.xlsx', '.xls', '.csv'];
    const fileExtension = file.name.toLowerCase().substr(file.name.lastIndexOf('.'));
    
    if (!allowedTypes.includes(fileExtension)) {
        this.env.showMessage('INVALID_FILE_TYPE', 'danger');
        return false;
    }
    
    if (file.size > 10 * 1024 * 1024) { // 10MB
        this.env.showMessage('FILE_TOO_LARGE', 'danger');
        return false;
    }
    
    return true;
}
```

---

## 🎯 **Selection Management Functions**

### **changeSelection(item, event?)**
Quản lý selection của items.

```typescript
// Handle single selection
onItemSelect(item: any, event: any) {
    this.changeSelection(item, event);
}

// Select all items
selectAll() {
    this.items.forEach(item => {
        if (!item.checked) {
            item.checked = true;
            this.selectedItems.push(item);
        }
    });
    this.selectedItems = [...this.selectedItems];
}

// Clear all selections
clearSelection() {
    this.unselect();
}

// Custom selection logic
selectByCondition(condition: (item: any) => boolean) {
    this.items.forEach(item => {
        if (condition(item) && !item.checked) {
            item.checked = true;
            this.selectedItems.push(item);
        }
    });
    this.selectedItems = [...this.selectedItems];
}
```

### **showCommandBySelectedRows(selectedRows)**
Hiển thị commands dựa trên items đã chọn.

```typescript
// Tự động được gọi khi selection thay đổi
// Override để custom command logic
showCommandBySelectedRows(selectedRows) {
    // Call parent method first
    super.showCommandBySelectedRows(selectedRows);
    
    // Custom command logic
    const hasLockedItems = selectedRows.some(item => item.IsLocked);
    this.pageConfig.ShowUnlock = hasLockedItems;
    this.pageConfig.ShowLock = !hasLockedItems;
    
    // Disable certain commands for specific statuses
    const hasApprovedItems = selectedRows.some(item => item.Status === 'approved');
    if (hasApprovedItems) {
        this.pageConfig.ShowEdit = false;
        this.pageConfig.ShowDelete = false;
    }
}
```

---

## 🌳 **Tree View Functions**

### **buildFlatTree(items, treeState, isAllRowOpened?)**
Xây dựng flat tree từ hierarchical data.

```typescript
// Build tree view
async loadTreeData() {
    const result = await this.pageProvider.read(this.query);
    const treeData = await this.buildFlatTree(result.data, this.treeState, true);
    this.items = treeData;
}

// Toggle tree node
toggleTreeNode(item: any) {
    this.toggleRow(this.items, item, true);
}

// Expand/collapse all nodes
toggleAllNodes() {
    this.toggleRowAll(this.items);
}
```

### **Tree Navigation Functions**
```typescript
// Search in tree
searchInTree(term: string) {
    const matchedIds = this.searchResultIdList.ids = 
        lib.searchTreeReturnId(this.items, term);
    
    // Show only matched items and their parents
    this.items.forEach(item => {
        item.show = matchedIds.includes(item.Id);
    });
}

// Show/hide nested folders
showHideNestedFolder(parentId: any, isShow: boolean) {
    this.showHideAllNestedFolder(this.items, parentId, isShow, true);
}
```

---

## 🎛️ **UI Control Functions**

### **Navigation Functions**
```typescript
// Navigate to different pages
goToDetail(id: number) {
    this.nav(`${this.pageConfig.pageName}/${id}`);
}

// Navigate with data
goToPageWithData(url: string, data: any) {
    this.nav(url, 'forward', { state: data });
}

// Go back
goBack() {
    this.navCtrl.back();
}

// Close modal
async closeModal() {
    await this.modalController.dismiss();
}
```

### **Sorting Functions**
```typescript
// Toggle sort for column
sortBy(field: string) {
    this.sortToggle(field);
}

// Custom sort handling
onSort(event: any) {
    this.pageConfig.sort = event;
    this.parseSort();
    this.refresh();
}

// Multi-column sort
applySorting(sortConfig: any[]) {
    this.pageConfig.sort = sortConfig;
    this.refresh();
}
```

---

## 📋 **Common Usage Patterns**

### **1. Standard List Page Pattern**
```typescript
export class ProductListPage extends PageBase {
    constructor(
        public pageProvider: ProductService,
        public env: EnvService,
        // ... other dependencies
    ) {
        super();
        
        this.pageConfig = {
            ...this.pageConfig,
            pageName: 'product-list',
            pageTitle: 'Product Management',
            isDetailPage: false,
            ShowAdd: true,
            ShowDelete: true,
            ShowExport: true,
            ShowImport: true
        };
    }

    async ionViewDidEnter() {
        await this.env.ready;
        this.loadData();
    }

    // Custom search logic
    search(ev) {
        const keyword = ev.target.value;
        if (keyword.length >= 2 || keyword === '') {
            this.query.Keyword = keyword;
            this.query.SearchFields = ['Name', 'Code', 'Barcode'];
            this.query.Skip = 0;
            this.pageConfig.isEndOfData = false;
            this.loadData('search');
        }
    }

    // Custom filter
    onCategoryFilter(categoryId: number) {
        this.query.CategoryId = categoryId;
        this.refresh();
    }
}
```

### **2. Standard Detail Page Pattern**
```typescript
export class ProductDetailPage extends PageBase {
    constructor(
        public pageProvider: ProductService,
        public env: EnvService,
        public route: ActivatedRoute,
        public formBuilder: FormBuilder,
        // ... other dependencies
    ) {
        super();
        
        this.pageConfig = {
            ...this.pageConfig,
            pageName: 'product-detail',
            pageTitle: 'Product Details',
            isDetailPage: true
        };

        // Setup form
        this.formGroup = this.formBuilder.group({
            Id: [0],
            Name: ['', Validators.required],
            Code: ['', Validators.required],
            Price: [0, [Validators.required, Validators.min(0)]],
            CategoryId: ['', Validators.required]
        });
    }

    async ionViewDidEnter() {
        await this.env.ready;
        this.id = this.route.snapshot.paramMap.get('id');
        this.loadData();
    }

    // Custom validation
    validateCustomRules(): boolean {
        if (this.item.Price <= 0) {
            this.env.showMessage('PRICE_MUST_BE_POSITIVE', 'danger');
            return false;
        }
        return true;
    }

    // Custom save logic
    async saveChange(publishEventCode = this.pageConfig.pageName) {
        if (!this.validateCustomRules()) {
            return Promise.reject('Validation failed');
        }
        
        return super.saveChange(publishEventCode);
    }
}
```

### **3. Tree View Page Pattern**
```typescript
export class CategoryTreePage extends PageBase {
    treeState: any[] = [];
    
    constructor(
        public pageProvider: CategoryService,
        public env: EnvService,
        // ... other dependencies
    ) {
        super();
        
        this.pageConfig = {
            ...this.pageConfig,
            pageName: 'category-tree',
            pageTitle: 'Category Tree',
            isDetailPage: false
        };
    }

    async loadData(event = null, forceReload = false) {
        try {
            const result = await this.pageProvider.read(this.query, forceReload);
            const treeData = await this.buildFlatTree(result.data, this.treeState, true);
            this.items = treeData;
            this.loadedData(event);
        } catch (error) {
            this.env.showErrorMessage(error);
            this.loadedData(event);
        }
    }

    // Tree-specific methods
    onNodeToggle(item: any) {
        this.toggleRow(this.items, item, true);
    }

    expandAll() {
        this.isAllRowOpened = true;
        this.toggleRowAll(this.items);
    }

    collapseAll() {
        this.isAllRowOpened = false;
        this.toggleRowAll(this.items);
    }
}
```

### **4. Approval Workflow Page Pattern**
```typescript
export class OrderApprovalPage extends PageBase {
    constructor(
        public pageProvider: OrderService,
        public env: EnvService,
        // ... other dependencies
    ) {
        super();
        
        this.pageConfig = {
            ...this.pageConfig,
            pageName: 'order-approval',
            pageTitle: 'Order Approval',
            ShowSubmit: true,
            ShowApprove: true,
            ShowDisapprove: true
        };
        
        // Load only pending orders
        this.query.Status = 'pending_approval';
    }

    // Custom approval logic
    async approve() {
        const invalidItems = this.selectedItems.filter(item => 
            !item.IsComplete || item.TotalAmount <= 0
        );
        
        if (invalidItems.length > 0) {
            this.env.showMessage('INVALID_ITEMS_CANNOT_APPROVE', 'danger');
            return;
        }
        
        super.approve();
    }

    // Batch approval with custom logic
    async batchApprove() {
        const validItems = this.selectedItems.filter(item => 
            item.IsComplete && item.TotalAmount > 0
        );
        
        if (validItems.length === 0) {
            this.env.showMessage('NO_VALID_ITEMS_TO_APPROVE', 'warning');
            return;
        }
        
        try {
            await this.env.actionConfirm(
                'approve',
                validItems.length,
                'orders',
                'Order Approval',
                () => this.pageProvider.approve(validItems)
            );
            
            this.refresh();
        } catch (error) {
            if (error !== 'User abort action') {
                this.env.showErrorMessage(error);
            }
        }
    }
}
```

---

## 🚨 **Best Practices**

### **1. Page Configuration**
```typescript
// ✅ Đúng - Đầy đủ configuration
constructor() {
    super();
    this.pageConfig = {
        ...this.pageConfig,
        pageName: 'user-management',
        pageTitle: 'User Management',
        isDetailPage: false,
        ShowAdd: true,
        ShowDelete: true,
        ShowExport: true,
        ShowImport: true
    };
}

// ❌ Sai - Thiếu configuration
constructor() {
    super();
    // Missing page configuration
}
```

### **2. Lifecycle Management**
```typescript
// ✅ Đúng - Proper lifecycle
async ionViewDidEnter() {
    await this.env.ready;
    this.loadData();
}

ngOnDestroy() {
    super.ngOnDestroy();
    // Custom cleanup
    this.customSubscription?.unsubscribe();
}

// ❌ Sai - Không đợi env ready
ionViewDidEnter() {
    this.loadData(); // Có thể fail nếu env chưa ready
}
```

### **3. Error Handling**
```typescript
// ✅ Đúng - Proper error handling
async saveData() {
    try {
        await this.saveChange();
        // Success handling
    } catch (error) {
        // Error đã được handle bởi PageBase
        // Custom error handling nếu cần
        this.handleCustomError(error);
    }
}

// ❌ Sai - Không handle error
async saveData() {
    await this.saveChange(); // Có thể throw error
}
```

### **4. Form Management**
```typescript
// ✅ Đúng - Proper form setup
ngOnInit() {
    super.ngOnInit();
    
    this.formGroup = this.formBuilder.group({
        Id: [0],
        Name: ['', Validators.required],
        Email: ['', [Validators.required, Validators.email]]
    });
    
    // Setup auto-save
    this.formGroup.valueChanges.pipe(
        debounceTime(1000),
        distinctUntilChanged()
    ).subscribe(() => {
        if (this.formGroup.dirty && this.pageConfig.systemConfig?.IsAutoSave) {
            this.saveChange2();
        }
    });
}

// ❌ Sai - Không setup form properly
ngOnInit() {
    super.ngOnInit();
    // Missing form setup
}
```

---

## ⚠️ **Lưu ý quan trọng**

### **Performance Considerations**
- **Pagination**: Sử dụng `query.Skip` và `query.Take` cho large datasets
- **Infinite Scroll**: Enable `pageConfig.infiniteScroll` cho mobile experience
- **Tree View**: Limit tree depth và use virtual scrolling cho large trees
- **Auto-save**: Debounce form changes để tránh too many API calls

### **Security Considerations**
- **Permission Check**: PageBase tự động check permissions dựa trên `pageConfig`
- **Data Validation**: Luôn validate data trước khi save
- **XSS Prevention**: Sanitize user input trong search và form fields

### **Mobile Considerations**
- **Touch Interactions**: PageBase support touch gestures cho selection
- **Responsive UI**: Use `pageConfig` flags để hide/show features trên mobile
- **Offline Support**: Cache data locally khi possible

---

## 🔧 **Advanced Patterns**

### **Custom Page Configuration**
```typescript
// Dynamic page config based on user role
ngOnInit() {
    super.ngOnInit();
    
    // Customize based on user permissions
    if (this.env.user.Role === 'viewer') {
        this.pageConfig.ShowAdd = false;
        this.pageConfig.ShowDelete = false;
        this.pageConfig.ShowEdit = false;
    }
    
    // Customize based on data status
    if (this.item?.Status === 'approved') {
        this.pageConfig.canEdit = false;
        this.formGroup?.disable();
    }
}
```

### **Custom Event Handling**
```typescript
// Override events method để handle custom events
events(event: any) {
    switch (event.Code) {
        case 'custom-data-updated':
            this.handleCustomDataUpdate(event.Value);
            break;
        case 'external-system-sync':
            this.syncWithExternalSystem();
            break;
        default:
            super.events(event);
    }
}

handleCustomDataUpdate(data: any) {
    // Custom logic for data update
    const updatedItem = this.items.find(item => item.Id === data.Id);
    if (updatedItem) {
        Object.assign(updatedItem, data);
        this.cdr.detectChanges();
    }
}
```

### **Multi-Provider Pages**
```typescript
// Page sử dụng nhiều providers
export class ComplexPage extends PageBase {
    secondaryProvider: SecondaryService;
    
    constructor(
        public pageProvider: PrimaryService,
        public secondaryProvider: SecondaryService,
        // ... other dependencies
    ) {
        super();
    }

    async loadData(event = null, forceReload = false) {
        try {
            // Load primary data
            await super.loadData(event, forceReload);
            
            // Load secondary data
            await this.loadSecondaryData();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async loadSecondaryData() {
        const secondaryData = await this.secondaryProvider.read({
            RelatedId: this.item?.Id
        });
        this.item.SecondaryData = secondaryData.data;
    }
}
```

---

**Tài liệu này sẽ được cập nhật khi có thêm methods mới hoặc thay đổi existing functionality.**

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
