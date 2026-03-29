# EnvService Documentation - ART-ERP-FE

## 📋 **Mục đích tài liệu**

Tài liệu này mô tả chi tiết `EnvService` - service trung tâm của ứng dụng ART-ERP-FE, cung cấp các chức năng quản lý môi trường, thông báo, storage, authentication và nhiều tiện ích khác.

---

## 🏗️ **Tổng quan EnvService**

`EnvService` là service cốt lõi được inject vào hầu hết các component/page trong ứng dụng. Nó cung cấp:

- **Environment Management** - Quản lý cấu hình môi trường
- **User Interface** - Thông báo, alert, loading
- **Storage Management** - Local storage và cache
- **Authentication Support** - Hỗ trợ xác thực và phân quyền
- **Branch Management** - Quản lý chi nhánh và phòng ban
- **System Data** - Status, Type, Branch data
- **Event System** - Pub/Sub pattern cho communication

---

## 💉 **Injection và Setup**

```typescript
// ✅ Đúng - Inject EnvService
constructor(
    public env: EnvService,
    // ... other dependencies
) {}

// ❌ Sai - Không inject EnvService
constructor() {
    // Missing EnvService dependency
}
```

---

## 🔔 **Message & Notification Functions**

### **showMessage(message, color = '', value = null, duration = 5000, showCloseButton = false, subHeader = '', header = '')**
Hiển thị thông báo toast hoặc alert cho user.

```typescript
// Thông báo thành công
this.env.showMessage('SAVE_SUCCESS', 'success');

// Thông báo lỗi
this.env.showMessage('SAVE_FAILED', 'danger');

// Thông báo với giá trị dynamic
this.env.showMessage('ITEM_SAVED', 'success', { value: 'Product A' });

// Alert với nút đóng
this.env.showMessage('IMPORTANT_MESSAGE', 'warning', null, 0, true, 'SubHeader', 'Header');
```

**Parameters:**
- `message`: Message key để translate
- `color`: 'success', 'danger', 'warning', 'primary', etc.
- `value`: Object chứa giá trị để bind vào message
- `duration`: Thời gian hiển thị (ms), 0 = không tự đóng
- `showCloseButton`: true = hiển thị alert, false = toast
- `subHeader`: Sub header cho alert
- `header`: Header cho alert

### **showErrorMessage(err)**
Hiển thị thông báo lỗi từ API response.

```typescript
try {
    const result = await this.service.save(data);
    this.env.showMessage('SAVE_SUCCESS', 'success');
} catch (error) {
    this.env.showErrorMessage(error); // Tự động parse error message
}
```

### **showAlert(message, subHeader = null, header = null, okText = 'Ok')**
**@deprecated** - Sử dụng `showMessage` với `showCloseButton = true` thay thế.

### **showPrompt(message, subHeader = null, header = null, okText = 'Ok', cancelText = 'Cancel', inputs = null)**
Hiển thị dialog xác nhận với user.

```typescript
// Confirm dialog đơn giản
try {
    await this.env.showPrompt('DELETE_CONFIRM', 'Are you sure?', 'Delete Item');
    // User clicked OK
    await this.deleteItem();
} catch {
    // User clicked Cancel
}

// Prompt với input
try {
    const result = await this.env.showPrompt(
        'ENTER_QUANTITY', 
        'Please enter quantity', 
        'Update Stock',
        'OK',
        'Cancel',
        [{ name: 'quantity', type: 'number', placeholder: 'Enter quantity' }]
    );
    console.log('User entered:', result.quantity);
} catch {
    // User cancelled
}
```

### **showLoading(message, promise)**
Hiển thị loading spinner trong khi thực hiện async operation.

```typescript
// Với promise function
const result = await this.env.showLoading('SAVING_DATA', () => this.service.save(data));

// Với promise trực tiếp
const result = await this.env.showLoading('LOADING_DATA', this.service.getData());
```

### **actionConfirm(action, length, itemName, title, promise)**
Hiển thị confirm dialog cho các action với nhiều items.

```typescript
// Confirm delete nhiều items
try {
    const result = await this.env.actionConfirm(
        'delete', 
        selectedItems.length, 
        'products', 
        'Product Management',
        () => this.service.deleteMultiple(selectedItems)
    );
    this.env.showMessage('DELETE_SUCCESS', 'success');
} catch (error) {
    // User cancelled hoặc có lỗi
}
```

---

## 🌐 **Translation Functions**

### **translateResource(resource)**
Translate message key thành text hiển thị.

```typescript
// Translate đơn giản
const message = await this.env.translateResource('WELCOME_MESSAGE');

// Translate với parameters
const message = await this.env.translateResource({
    code: 'ITEM_COUNT_MESSAGE',
    count: 5,
    type: 'products'
});

// Translate array
const messages = await this.env.translateResource([
    'SAVE_SUCCESS',
    'DELETE_SUCCESS',
    'UPDATE_SUCCESS'
]);
```

### **setLang(value?: string)**
Thay đổi ngôn ngữ ứng dụng.

```typescript
// Chuyển sang tiếng Việt
await this.env.setLang('vi-VN');

// Chuyển sang tiếng Anh
await this.env.setLang('en-US');

// Sử dụng ngôn ngữ mặc định
await this.env.setLang();
```

---

## 💾 **Storage Functions**

### **getStorage(key: string, query: any = null, branch: string = this.selectedBranch)**
Lấy dữ liệu từ storage/cache.

```typescript
// Lấy data đơn giản
const userData = await this.env.getStorage('UserProfile');

// Lấy data với query
const orders = await this.env.getStorage('SaleOrders', { status: 'active' });

// Lấy data từ branch khác
const branchData = await this.env.getStorage('BranchSettings', null, 'branch-123');
```

### **setStorage(key: string, value: any, config: CacheConfig = null, branch: string = this.selectedBranch, serviceName: string = 'Manual')**
Lưu dữ liệu vào storage/cache.

```typescript
// Lưu data đơn giản
await this.env.setStorage('UserPreferences', preferences);

// Lưu với cache config
await this.env.setStorage('ProductList', products, {
    enable: true,
    timeToLive: 300000 // 5 minutes
});

// Lưu vào branch cụ thể
await this.env.setStorage('BranchSettings', settings, null, 'branch-123');
```

### **setCookie(name, value, days)**
Lưu cookie.

```typescript
// Lưu cookie 7 ngày
this.env.setCookie('userTheme', 'dark', 7);

// Lưu cookie session (không có expiry)
this.env.setCookie('tempData', 'value', null);
```

### **getCookie(name)**
Lấy giá trị cookie.

```typescript
const theme = this.env.getCookie('userTheme'); // 'dark' hoặc null
```

### **clearCookie(name)**
Xóa cookie.

```typescript
this.env.clearCookie('userTheme');
```

---

## 🏢 **Branch Management Functions**

### **loadBranch()**
Load và build branch tree từ user permissions.

```typescript
// Thường được gọi tự động khi user login
await this.env.loadBranch();
```

### **changeBranch(branchId)**
Thay đổi branch hiện tại.

```typescript
// Chuyển sang branch khác
this.env.changeBranch('branch-123');

// Lắng nghe event branch changed
this.env.getEvents().subscribe(event => {
    if (event.Code === 'BRANCH_SWITCHED') {
        // Reload data for new branch
        this.loadData();
    }
});
```

### **getBranch(Id: number, AllChild = false)**
Lấy danh sách branch con.

```typescript
// Lấy branch con trực tiếp
const childBranches = await this.env.getBranch(parentId);

// Lấy tất cả branch con (flat tree)
const allChildBranches = await this.env.getBranch(parentId, true);
```

### **searchBranch(predicate: (branch: any) => boolean)**
Tìm kiếm branch theo điều kiện.

```typescript
// Tìm branch theo tên
const branches = await this.env.searchBranch(branch => 
    branch.Name.toLowerCase().includes('sales')
);

// Tìm branch theo type
const warehouses = await this.env.searchBranch(branch => 
    branch.Type === 'Warehouse'
);
```

### **getWarehouses(getParents = true, IgnoredSelectedBranch = false)**
Lấy danh sách kho.

```typescript
// Lấy kho trong branch hiện tại (bao gồm parent)
const warehouses = await this.env.getWarehouses();

// Lấy chỉ kho, không bao gồm parent
const warehousesOnly = await this.env.getWarehouses(false);

// Lấy tất cả kho (bỏ qua branch filter)
const allWarehouses = await this.env.getWarehouses(true, true);
```

### **getJobTitle(Id: number, AllChild = false)**
Lấy danh sách chức vụ.

```typescript
// Lấy chức vụ con trực tiếp
const jobTitles = await this.env.getJobTitle(parentId);

// Lấy tất cả chức vụ con (flat tree)
const allJobTitles = await this.env.getJobTitle(parentId, true);
```

---

## 📊 **System Data Functions**

### **getStatus(Code: string)**
Lấy danh sách status theo parent code.

```typescript
// Lấy status của đơn hàng
const orderStatuses = await this.env.getStatus('SaleOrderStatus');

// Lấy status của sản phẩm
const productStatuses = await this.env.getStatus('ProductStatus');
```

### **getType(Code: string, AllChild = false)**
Lấy danh sách type theo parent code.

```typescript
// Lấy type con trực tiếp
const productTypes = await this.env.getType('ProductType');

// Lấy tất cả type con (flat tree)
const allProductTypes = await this.env.getType('ProductType', true);
```

### **getTypeAsync(Code: string, AllChild = false)**
Phiên bản async của getType (không dùng Promise wrapper).

```typescript
const productTypes = await this.env.getTypeAsync('ProductType', true);
```

---

## 🔐 **Permission Functions**

### **checkFormPermission(functionCode)**
Kiểm tra quyền truy cập form/function.

```typescript
// Kiểm tra quyền trước khi vào trang
const hasPermission = await this.env.checkFormPermission('/sale/order');
if (!hasPermission) {
    this.router.navigate(['/not-authorized']);
    return;
}

// Kiểm tra quyền cho button/action
const canDelete = await this.env.checkFormPermission('/sale/order/delete');
this.showDeleteButton = canDelete;
```

---

## 📡 **Event System Functions**

### **publishEvent(data: any)**
Publish event để communication giữa các component.

```typescript
// Publish custom event
this.env.publishEvent({
    Code: 'CUSTOM_EVENT',
    Value: { id: 123, name: 'Product A' }
});

// Publish system event
this.env.publishEvent({
    Code: 'DATA_UPDATED',
    Type: 'Product',
    Ids: [1, 2, 3]
});
```

### **getEvents(): Subject<any>**
Lắng nghe events từ system.

```typescript
// Subscribe to all events
this.env.getEvents().subscribe(event => {
    switch (event.Code) {
        case 'BRANCH_SWITCHED':
            this.onBranchChanged();
            break;
        case 'DATA_UPDATED':
            this.onDataUpdated(event.Value);
            break;
        case 'CUSTOM_EVENT':
            this.onCustomEvent(event.Value);
            break;
    }
});

// Unsubscribe trong ngOnDestroy
ngOnDestroy() {
    this.subscription?.unsubscribe();
}
```

---

## 🌐 **Network & Status Functions**

### **trackOnline()**
Monitor network status.

```typescript
// Track network status
this.env.trackOnline().subscribe(isOnline => {
    if (isOnline) {
        this.env.showMessage('BACK_ONLINE', 'success');
        this.syncOfflineData();
    } else {
        this.env.showMessage('OFFLINE_MODE', 'warning');
    }
});
```

### **showBarMessage(id = '', icon = '', color = '', isShow = true, isBlink = false)**
Hiển thị message bar ở top app.

```typescript
// Hiển thị warning bar
this.env.showBarMessage('network-warning', 'wifi-off', 'warning', true, true);

// Ẩn message bar
this.env.showBarMessage('network-warning', '', '', false);
```

---

## 🏗️ **Properties và Getters**

### **Readonly Properties**
```typescript
// App information
this.env.app.version;        // "v1.0.0"
this.env.app.isOnline;       // true/false
this.env.app.theme;          // "default-theme"

// Current user
this.env.user;               // UserProfile object hoặc null
this.env.user?.Id;           // User ID
this.env.user?.FullName;     // User full name

// Language
this.env.language.current;   // "vi-VN"
this.env.language.default;   // "vi-VN"
this.env.language.isDefault; // true/false

// Branch data
this.env.selectedBranch;     // Current branch ID
this.env.branchList;         // Flat branch list
this.env.jobTitleList;       // Job title list

// App path
this.env.appPath;            // Full app URL path
```

### **Ready Promise**
```typescript
// Đợi EnvService sẵn sàng
await this.env.ready;
console.log('EnvService is ready!');
```

---

## 📋 **Common Usage Patterns**

### **1. Page Initialization Pattern**
```typescript
export class SamplePage extends PageBase {
    constructor(
        public pageProvider: SampleService,
        public env: EnvService,
        // ... other dependencies
    ) {
        super();
    }

    async ionViewDidEnter() {
        // Đợi env ready
        await this.env.ready;
        
        // Check permission
        const hasPermission = await this.env.checkFormPermission('/sample/page');
        if (!hasPermission) {
            this.env.showMessage('ACCESS_DENIED', 'danger');
            return;
        }

        // Load data
        this.loadData();
    }
}
```

### **2. CRUD Operations Pattern**
```typescript
async saveItem() {
    try {
        const result = await this.env.showLoading('SAVING', () => 
            this.pageProvider.save(this.item)
        );
        
        this.env.showMessage('SAVE_SUCCESS', 'success');
        this.env.publishEvent({ Code: 'ITEM_SAVED', Value: result });
        
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}

async deleteItems() {
    try {
        await this.env.actionConfirm(
            'delete',
            this.selectedItems.length,
            'items',
            'Item Management',
            () => this.pageProvider.deleteMultiple(this.selectedItems)
        );
        
        this.env.showMessage('DELETE_SUCCESS', 'success');
        this.loadData();
        
    } catch (error) {
        if (error !== 'User abort action') {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **3. Event Communication Pattern**
```typescript
// Component A - Publisher
onDataChanged() {
    this.env.publishEvent({
        Code: 'PRODUCT_UPDATED',
        Value: { productId: this.product.Id, changes: this.changes }
    });
}

// Component B - Subscriber
ngOnInit() {
    this.eventSubscription = this.env.getEvents().subscribe(event => {
        if (event.Code === 'PRODUCT_UPDATED') {
            this.onProductUpdated(event.Value);
        }
    });
}

ngOnDestroy() {
    this.eventSubscription?.unsubscribe();
}
```

### **4. Branch-Aware Data Loading**
```typescript
async loadData() {
    // Lắng nghe branch changes
    this.env.getEvents().subscribe(event => {
        if (event.Code === 'BRANCH_SWITCHED') {
            this.loadBranchData();
        }
    });

    // Load data cho branch hiện tại
    this.loadBranchData();
}

async loadBranchData() {
    const branchId = this.env.selectedBranch;
    const data = await this.pageProvider.getByBranch(branchId);
    this.items = data;
}
```

---

## 🚨 **Best Practices**

### **1. Error Handling**
```typescript
// ✅ Đúng - Sử dụng showErrorMessage
try {
    await this.service.save(data);
} catch (error) {
    this.env.showErrorMessage(error);
}

// ❌ Sai - Tự handle error message
try {
    await this.service.save(data);
} catch (error) {
    this.env.showMessage(error.message, 'danger');
}
```

### **2. Permission Checking**
```typescript
// ✅ Đúng - Check permission trước khi action
async deleteItem() {
    const canDelete = await this.env.checkFormPermission('/module/delete');
    if (!canDelete) {
        this.env.showMessage('ACCESS_DENIED', 'danger');
        return;
    }
    // Proceed with delete
}

// ❌ Sai - Không check permission
async deleteItem() {
    // Delete without permission check
}
```

### **3. Event Subscription Management**
```typescript
// ✅ Đúng - Unsubscribe trong ngOnDestroy
private eventSubscription: Subscription;

ngOnInit() {
    this.eventSubscription = this.env.getEvents().subscribe(event => {
        // Handle event
    });
}

ngOnDestroy() {
    this.eventSubscription?.unsubscribe();
}

// ❌ Sai - Không unsubscribe
ngOnInit() {
    this.env.getEvents().subscribe(event => {
        // Memory leak!
    });
}
```

### **4. Loading States**
```typescript
// ✅ Đúng - Sử dụng showLoading
async loadData() {
    const data = await this.env.showLoading('LOADING_DATA', () => 
        this.service.getData()
    );
    this.items = data;
}

// ❌ Sai - Tự quản lý loading
async loadData() {
    this.isLoading = true;
    try {
        this.items = await this.service.getData();
    } finally {
        this.isLoading = false;
    }
}
```

---

## ⚠️ **Deprecated Methods**

Các methods sau đây đã deprecated, không nên sử dụng:

- `showAlert()` - Sử dụng `showMessage()` với `showCloseButton = true`
- `enablePermissionNode()` - Internal method, không dùng
- `getChildrenDepartmentID()` - Internal method, không dùng

---

## 🔧 **Advanced Usage**

### **Custom Event Types**
```typescript
// Define custom event constants
const CUSTOM_EVENTS = {
    PRODUCT_SELECTED: 'product:selected',
    CART_UPDATED: 'cart:updated',
    ORDER_STATUS_CHANGED: 'order:status:changed'
};

// Publish typed events
this.env.publishEvent({
    Code: CUSTOM_EVENTS.PRODUCT_SELECTED,
    Value: { productId: 123, variant: 'red' }
});
```

### **Conditional UI Based on Permissions**
```typescript
// Template
<ion-button *ngIf="canEdit" (click)="editItem()">
    Edit
</ion-button>

// Component
async ngOnInit() {
    this.canEdit = await this.env.checkFormPermission('/module/edit');
    this.canDelete = await this.env.checkFormPermission('/module/delete');
}
```

### **Multi-language Support**
```typescript
// Setup language change listener
ngOnInit() {
    this.env.languageTracking.subscribe(language => {
        this.currentLanguage = language.current;
        this.reloadLocalizedData();
    });
}

// Change language programmatically
async changeLanguage(langCode: string) {
    await this.env.setLang(langCode);
    this.env.showMessage('LANGUAGE_CHANGED', 'success');
}
```

---

**Tài liệu này sẽ được cập nhật khi có thêm methods mới hoặc thay đổi existing methods.**

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
