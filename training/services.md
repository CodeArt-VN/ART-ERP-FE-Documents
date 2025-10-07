# Provider Service Documentation - ART-ERP-FE

## 📋 **Mục đích tài liệu**

Tài liệu này mô tả chi tiết `providerService` (ExtendService) - base class cho tất cả business services trong ứng dụng ART-ERP-FE. Service này cung cấp interface chuẩn cho CRUD operations với hỗ trợ caching và data access patterns nhất quán.

---

## 🏗️ **Tổng quan Provider Service**

`providerService` là abstract base class được extend bởi tất cả business services trong ứng dụng. Nó cung cấp:

- **Standardized CRUD Operations** - Create, Read, Update, Delete
- **Caching Support** - Local cache với TTL và invalidation
- **Search Functionality** - Local và server-side search
- **File Operations** - Import, Export, Upload, Download
- **Approval Workflow** - Submit, Approve, Disapprove
- **Branch Management** - Multi-branch data handling

---

## 🔧 **Service Setup và Configuration**

### **Extending Provider Service**
```typescript
import { Injectable } from '@angular/core';
import { providerService } from 'src/app/services/core/extend-service';
import { CommonService } from 'src/app/services/core/common.service';

@Injectable({
    providedIn: 'root'
})
export class UserService extends providerService {
    constructor(public commonService: CommonService) {
        super();
        this.commonService = commonService;
        
        // Required configurations
        this.apiPath = {
            getList: { method: 'GET', url: () => 'CRM/Contact/User' },
            getItem: { method: 'GET', url: () => 'CRM/Contact/User' },
            getSearchList: { method: 'GET', url: () => 'CRM/Contact/User/Search' },
            post: { method: 'POST', url: () => 'CRM/Contact/User' },
            put: { method: 'PUT', url: () => 'CRM/Contact/User' },
            delete: { method: 'DELETE', url: () => 'CRM/Contact/User' }
        };
        
        // Service configuration
        this.serviceName = 'UserService';
        this.allowCache = true;
        this.searchField = ['Name', 'Email', 'Phone'];
        
        // Cache configuration
        this.cacheConfig = {
            enable: true,
            timeToLive: 300000, // 5 minutes
            query: null
        };
    }
}
```

### **Service Properties**
```typescript
// API configuration
apiPath: any;                    // API endpoints configuration
serviceName: string;             // Service identifier
allowCache: boolean;             // Enable/disable caching
searchField: string[];           // Fields for local search
cacheConfig: CacheConfig;        // Cache settings
showCommandRules: any[];         // UI command visibility rules
commonService: CommonService;    // Reference to CommonService
```

---

## 📖 **Data Reading Functions**

### **read(query?: any, forceReload = false)**
Đọc danh sách items với hỗ trợ cache và pagination.

```typescript
// Basic read với cache
const result = await this.userService.read();
console.log('Total users:', result.count);
console.log('User list:', result.data);

// Read với query parameters
const result = await this.userService.read({
    Page: 1,
    PageSize: 10,
    Keyword: 'john',
    Status: 'active'
});

// Force reload từ server (bypass cache)
const result = await this.userService.read(null, true);

// Read với sorting và filtering
const result = await this.userService.read({
    Page: 1,
    PageSize: 20,
    SortBy: 'Name',
    SortDirection: 'asc',
    Filter: JSON.stringify({ Department: 'IT' })
});
```

**Return Type:**
```typescript
{
    count: number,    // Total number of items
    data: any[]       // Array of items for current page
}
```

### **readServer(query?: any)**
Đọc dữ liệu trực tiếp từ server, bỏ qua cache.

```typescript
// Read từ server với query
const result = await this.userService.readServer({
    Page: 1,
    PageSize: 50,
    IncludeDeleted: true
});

// Sử dụng khi cần data real-time
const liveData = await this.userService.readServer();
```

### **getAnItem(Id, UID = '')**
Lấy một item cụ thể theo ID.

```typescript
// Lấy user theo ID
const user = await this.userService.getAnItem(123);

// Lấy user với UID cụ thể
const user = await this.userService.getAnItem(123, 'unique-identifier');

// Handle không tìm thấy
try {
    const user = await this.userService.getAnItem(999);
    if (user) {
        console.log('User found:', user);
    }
} catch (error) {
    console.log('User not found');
}
```

### **search(query?: any)**
Thực hiện search operation qua search API endpoint.

```typescript
// Search users
this.userService.search({
    Keyword: 'john doe',
    Department: 'IT',
    Status: 'active'
}).subscribe(results => {
    console.log('Search results:', results);
});

// Advanced search với multiple criteria
this.userService.search({
    Name: 'john',
    Email: '@company.com',
    CreatedFrom: '2024-01-01',
    CreatedTo: '2024-12-31'
}).subscribe(results => {
    this.searchResults = results;
});
```

---

## ✏️ **Data Modification Functions**

### **save(item, isForceCreate = false)**
Lưu item (tạo mới hoặc cập nhật).

```typescript
// Tạo user mới
const newUser = {
    Name: 'John Doe',
    Email: 'john@example.com',
    Phone: '0123456789',
    Department: 'IT'
};
const savedUser = await this.userService.save(newUser);

// Cập nhật user existing
const existingUser = {
    Id: 123,
    Name: 'John Updated',
    Email: 'john.updated@example.com'
};
const updatedUser = await this.userService.save(existingUser);

// Force tạo mới (ngay cả khi có Id)
const duplicateUser = await this.userService.save(existingUser, true);

// Save với error handling
try {
    const result = await this.userService.save(userData);
    this.env.showMessage('SAVE_SUCCESS', 'success');
    return result;
} catch (error) {
    this.env.showErrorMessage(error);
    throw error;
}
```

### **delete(items)**
Xóa items (soft delete).

```typescript
// Xóa một item
const result = await this.userService.delete([{ Id: 123 }]);

// Xóa nhiều items
const itemsToDelete = [
    { Id: 123 },
    { Id: 456 },
    { Id: 789 }
];
const result = await this.userService.delete(itemsToDelete);

// Delete với confirmation
try {
    await this.env.actionConfirm(
        'delete',
        selectedItems.length,
        'users',
        'User Management',
        () => this.userService.delete(selectedItems)
    );
    this.env.showMessage('DELETE_SUCCESS', 'success');
    this.loadData(); // Reload list
} catch (error) {
    if (error !== 'User abort action') {
        this.env.showErrorMessage(error);
    }
}
```

### **disable(items, IsDisabled = true)**
Enable/disable items.

```typescript
// Disable users
const result = await this.userService.disable([
    { Id: 123 },
    { Id: 456 }
]);

// Enable users
const result = await this.userService.disable([
    { Id: 123 },
    { Id: 456 }
], false);

// Toggle user status
async toggleUserStatus(user: any) {
    try {
        const newStatus = !user.IsDisabled;
        await this.userService.disable([{ Id: user.Id }], newStatus);
        user.IsDisabled = newStatus;
        
        const message = newStatus ? 'USER_DISABLED' : 'USER_ENABLED';
        this.env.showMessage(message, 'success');
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

---

## 🔄 **Approval Workflow Functions**

### **submitForApproval(items, apiPath)**
Gửi items để phê duyệt.

```typescript
// Submit users for approval
const itemsToSubmit = [
    { Id: 123 },
    { Id: 456 }
];

try {
    const result = await this.userService.submitForApproval(
        itemsToSubmit, 
        this.userService.apiPath
    );
    this.env.showMessage('SUBMIT_SUCCESS', 'success');
    this.loadData();
} catch (error) {
    this.env.showErrorMessage(error);
}
```

### **approve(items, apiPath)**
Phê duyệt items.

```typescript
// Approve pending users
const itemsToApprove = [
    { Id: 123 },
    { Id: 456 }
];

try {
    await this.env.actionConfirm(
        'approve',
        itemsToApprove.length,
        'users',
        'User Approval',
        () => this.userService.approve(itemsToApprove, this.userService.apiPath)
    );
    this.env.showMessage('APPROVE_SUCCESS', 'success');
    this.loadPendingApprovals();
} catch (error) {
    if (error !== 'User abort action') {
        this.env.showErrorMessage(error);
    }
}
```

### **disapprove(items, apiPath)**
Từ chối phê duyệt items.

```typescript
// Disapprove users
const itemsToDisapprove = [
    { Id: 123, Reason: 'Incomplete information' },
    { Id: 456, Reason: 'Invalid credentials' }
];

try {
    const result = await this.userService.disapprove(
        itemsToDisapprove, 
        this.userService.apiPath
    );
    this.env.showMessage('DISAPPROVE_SUCCESS', 'success');
    this.loadPendingApprovals();
} catch (error) {
    this.env.showErrorMessage(error);
}
```

### **cancel(items)**
Hủy bỏ operations đang pending.

```typescript
// Cancel pending operations
const itemsToCancel = [
    { Id: 123 },
    { Id: 456 }
];

const result = await this.userService.cancel(itemsToCancel);
```

---

## 📁 **File Operations Functions**

### **import(fileToUpload: File)**
Import dữ liệu từ file.

```typescript
// Import users từ Excel file
async onFileImport(event: any) {
    const file = event.target.files[0];
    if (!file) return;

    try {
        const result = await this.env.showLoading(
            'IMPORTING_DATA',
            () => this.userService.import(file)
        );
        
        this.env.showMessage('IMPORT_SUCCESS', 'success', {
            count: result.SuccessCount,
            errors: result.ErrorCount
        });
        
        if (result.Errors?.length > 0) {
            console.log('Import errors:', result.Errors);
        }
        
        this.loadData(); // Reload list
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}

// HTML template
// <input type="file" accept=".xlsx,.xls,.csv" (change)="onFileImport($event)">
```

### **export(query)**
Export dữ liệu ra file.

```typescript
// Export all users
async exportUsers() {
    try {
        const result = await this.env.showLoading(
            'EXPORTING_DATA',
            () => this.userService.export({
                Format: 'excel',
                FileName: 'Users_Export'
            })
        );
        
        this.env.showMessage('EXPORT_SUCCESS', 'success');
        // File sẽ được download tự động
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}

// Export với filter
async exportFilteredUsers() {
    const exportQuery = {
        Format: 'excel',
        FileName: 'Active_Users',
        Status: 'active',
        Department: this.selectedDepartment,
        DateFrom: this.dateFrom,
        DateTo: this.dateTo
    };
    
    const result = await this.userService.export(exportQuery);
}
```

### **upload(fileToUpload: File)**
Upload file lên server.

```typescript
// Upload user avatar
async uploadAvatar(userId: number, file: File) {
    try {
        const result = await this.env.showLoading(
            'UPLOADING_FILE',
            () => this.userService.upload(file)
        );
        
        // Update user với file URL
        const user = await this.userService.getAnItem(userId);
        user.AvatarUrl = result.FileUrl;
        await this.userService.save(user);
        
        this.env.showMessage('UPLOAD_SUCCESS', 'success');
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}

// Upload multiple files
async uploadMultipleFiles(files: FileList) {
    const uploadPromises = Array.from(files).map(file => 
        this.userService.upload(file)
    );
    
    try {
        const results = await Promise.all(uploadPromises);
        this.env.showMessage('UPLOAD_ALL_SUCCESS', 'success');
        return results;
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

### **download(query)**
Download file từ server.

```typescript
// Download user report
async downloadUserReport(userId: number) {
    try {
        const result = await this.userService.download({
            Id: userId,
            Format: 'pdf',
            ReportType: 'detailed'
        });
        
        this.env.showMessage('DOWNLOAD_SUCCESS', 'success');
        // File sẽ được download tự động
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}

// Download với custom filename
async downloadCustomReport() {
    const downloadQuery = {
        ReportType: 'summary',
        Format: 'excel',
        FileName: `User_Report_${new Date().toISOString().split('T')[0]}`,
        IncludeInactive: false
    };
    
    const result = await this.userService.download(downloadQuery);
}
```

---

## 🏢 **Branch Management Functions**

### **changeBranch(item)**
Thay đổi branch cho items.

```typescript
// Move user to different branch
async moveUserToBranch(userId: number, newBranchId: number) {
    try {
        const result = await this.userService.changeBranch({
            Id: userId,
            IDBranch: newBranchId
        });
        
        this.env.showMessage('BRANCH_CHANGE_SUCCESS', 'success');
        this.loadData();
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}

// Bulk branch change
async moveToBranch(userIds: number[], branchId: number) {
    const promises = userIds.map(id => 
        this.userService.changeBranch({
            Id: id,
            IDBranch: branchId
        })
    );
    
    try {
        await Promise.all(promises);
        this.env.showMessage('BULK_BRANCH_CHANGE_SUCCESS', 'success');
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

---

## 📋 **Common Usage Patterns**

### **1. Standard CRUD Page Pattern**
```typescript
export class UserListPage extends PageBase {
    items: any[] = [];
    selectedItems: any[] = [];
    query = { Page: 1, PageSize: 20 };
    
    constructor(
        public pageProvider: UserService,
        public env: EnvService
    ) {
        super();
    }

    async ionViewDidEnter() {
        await this.env.ready;
        this.loadData();
    }

    async loadData() {
        try {
            const result = await this.pageProvider.read(this.query);
            this.items = result.data;
            this.totalItems = result.count;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async saveItem(item: any) {
        try {
            const result = await this.env.showLoading(
                'SAVING',
                () => this.pageProvider.save(item)
            );
            
            this.env.showMessage('SAVE_SUCCESS', 'success');
            this.loadData();
            return result;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }

    async deleteSelectedItems() {
        if (this.selectedItems.length === 0) {
            this.env.showMessage('NO_ITEMS_SELECTED', 'warning');
            return;
        }

        try {
            await this.env.actionConfirm(
                'delete',
                this.selectedItems.length,
                'users',
                'User Management',
                () => this.pageProvider.delete(this.selectedItems)
            );
            
            this.selectedItems = [];
            this.loadData();
        } catch (error) {
            if (error !== 'User abort action') {
                this.env.showErrorMessage(error);
            }
        }
    }
}
```

### **2. Search và Filter Pattern**
```typescript
export class UserSearchPage {
    searchQuery = {
        Keyword: '',
        Department: '',
        Status: '',
        Page: 1,
        PageSize: 20
    };
    
    searchResults: any[] = [];
    isSearching = false;

    async performSearch() {
        if (!this.searchQuery.Keyword?.trim()) {
            this.env.showMessage('ENTER_SEARCH_KEYWORD', 'warning');
            return;
        }

        this.isSearching = true;
        try {
            // Sử dụng search method cho real-time search
            this.pageProvider.search(this.searchQuery).subscribe(results => {
                this.searchResults = results;
                this.isSearching = false;
            });
        } catch (error) {
            this.isSearching = false;
            this.env.showErrorMessage(error);
        }
    }

    async loadWithFilter() {
        try {
            // Sử dụng read method cho filtered list
            const result = await this.pageProvider.read(this.searchQuery);
            this.searchResults = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    clearSearch() {
        this.searchQuery = {
            Keyword: '',
            Department: '',
            Status: '',
            Page: 1,
            PageSize: 20
        };
        this.searchResults = [];
    }
}
```

### **3. Cache Management Pattern**
```typescript
export class UserService extends providerService {
    constructor(public commonService: CommonService) {
        super();
        this.setupService();
    }

    private setupService() {
        this.commonService = commonService;
        this.apiPath = UserApiPaths;
        this.serviceName = 'UserService';
        this.searchField = ['Name', 'Email', 'Phone', 'Code'];
        
        // Cache configuration
        this.allowCache = true;
        this.cacheConfig = {
            enable: true,
            timeToLive: 300000, // 5 minutes
            query: null
        };
    }

    // Override read method để custom cache behavior
    async read(query: any = null, forceReload = false) {
        // Force reload khi có filter quan trọng
        if (query?.IncludeDeleted || query?.RealTimeData) {
            forceReload = true;
        }

        return super.read(query, forceReload);
    }

    // Method để clear cache manually
    async clearCache() {
        const cacheKey = this.apiPath.getList.url();
        await this.commonService.env.setStorage(cacheKey, null);
    }

    // Method để refresh cache
    async refreshCache(query: any = null) {
        await this.clearCache();
        return this.read(query, true);
    }
}
```

### **4. Approval Workflow Pattern**
```typescript
export class UserApprovalPage {
    pendingItems: any[] = [];
    
    async loadPendingApprovals() {
        try {
            const result = await this.pageProvider.read({
                Status: 'pending_approval',
                Page: 1,
                PageSize: 50
            });
            this.pendingItems = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async approveSelected() {
        const selectedItems = this.pendingItems.filter(item => item._isChecked);
        
        if (selectedItems.length === 0) {
            this.env.showMessage('NO_ITEMS_SELECTED', 'warning');
            return;
        }

        try {
            await this.env.actionConfirm(
                'approve',
                selectedItems.length,
                'users',
                'User Approval',
                () => this.pageProvider.approve(selectedItems, this.pageProvider.apiPath)
            );
            
            this.loadPendingApprovals();
        } catch (error) {
            if (error !== 'User abort action') {
                this.env.showErrorMessage(error);
            }
        }
    }

    async disapproveWithReason(item: any) {
        try {
            const reason = await this.env.showPrompt(
                'ENTER_DISAPPROVAL_REASON',
                'Please provide reason for disapproval',
                'Disapprove User',
                'Submit',
                'Cancel',
                [{ name: 'reason', type: 'textarea', placeholder: 'Enter reason...' }]
            );

            item.DisapprovalReason = reason.reason;
            
            await this.pageProvider.disapprove([item], this.pageProvider.apiPath);
            this.env.showMessage('DISAPPROVE_SUCCESS', 'success');
            this.loadPendingApprovals();
            
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

### **1. Service Configuration**
```typescript
// ✅ Đúng - Đầy đủ configuration
export class ProductService extends providerService {
    constructor(public commonService: CommonService) {
        super();
        this.commonService = commonService;
        this.apiPath = ProductApiPaths;
        this.serviceName = 'ProductService';
        this.allowCache = true;
        this.searchField = ['Name', 'Code', 'Barcode'];
        this.cacheConfig = {
            enable: true,
            timeToLive: 600000 // 10 minutes cho product data
        };
    }
}

// ❌ Sai - Thiếu configuration
export class ProductService extends providerService {
    constructor(public commonService: CommonService) {
        super();
        // Missing required configurations
    }
}
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Proper error handling
async saveProduct(product: any) {
    try {
        const result = await this.productService.save(product);
        this.env.showMessage('SAVE_SUCCESS', 'success');
        return result;
    } catch (error) {
        this.env.showErrorMessage(error);
        throw error; // Re-throw để caller handle
    }
}

// ❌ Sai - Không handle error
async saveProduct(product: any) {
    const result = await this.productService.save(product);
    return result;
}
```

### **3. Cache Usage**
```typescript
// ✅ Đúng - Smart cache usage
async loadProducts(forceRefresh = false) {
    // Force refresh khi cần data mới nhất
    const result = await this.productService.read(this.query, forceRefresh);
    this.products = result.data;
}

// Load cached data cho performance
async loadCachedProducts() {
    const result = await this.productService.read(this.query, false);
    this.products = result.data;
}

// ❌ Sai - Luôn force reload
async loadProducts() {
    const result = await this.productService.read(this.query, true);
    this.products = result.data;
}
```

### **4. Bulk Operations**
```typescript
// ✅ Đúng - Efficient bulk operations
async deleteMultipleProducts(productIds: number[]) {
    const items = productIds.map(id => ({ Id: id }));
    
    try {
        await this.env.actionConfirm(
            'delete',
            items.length,
            'products',
            'Product Management',
            () => this.productService.delete(items)
        );
        
        this.loadProducts();
    } catch (error) {
        if (error !== 'User abort action') {
            this.env.showErrorMessage(error);
        }
    }
}

// ❌ Sai - Individual operations
async deleteMultipleProducts(productIds: number[]) {
    for (const id of productIds) {
        await this.productService.delete([{ Id: id }]);
    }
}
```

---

## ⚠️ **Lưu ý quan trọng**

### **Performance Considerations**
- **Cache TTL**: Set appropriate cache time dựa trên data volatility
- **Search Fields**: Chỉ include fields thực sự cần search
- **Bulk Operations**: Sử dụng bulk APIs thay vì individual calls
- **Force Reload**: Chỉ dùng khi thực sự cần data real-time

### **Security Considerations**
- **Permission Check**: Luôn check permission trước khi call service methods
- **Data Validation**: Validate data trước khi save
- **File Upload**: Validate file type và size trước khi upload

### **Error Handling**
- **Use showErrorMessage()**: Để handle API errors automatically
- **Re-throw Errors**: Trong service methods để caller có thể handle
- **User Feedback**: Luôn show message cho user về kết quả operations

---

## 🔧 **Advanced Patterns**

### **Custom Service Methods**
```typescript
export class OrderService extends providerService {
    // Custom method cho business logic
    async submitOrder(orderId: number) {
        const apiPath = {
            method: 'POST',
            url: () => `SALE/Order/${orderId}/Submit`
        };
        
        return this.commonService.connect(
            apiPath.method, 
            apiPath.url(), 
            null
        ).toPromise();
    }

    // Override save method cho custom logic
    async save(item: any, isForceCreate = false) {
        // Custom validation
        if (!item.CustomerName?.trim()) {
            throw new Error('Customer name is required');
        }

        // Call parent save method
        return super.save(item, isForceCreate);
    }
}
```

### **Service Composition**
```typescript
export class ComplexBusinessService {
    constructor(
        private userService: UserService,
        private productService: ProductService,
        private orderService: OrderService,
        private env: EnvService
    ) {}

    async createOrderWithValidation(orderData: any) {
        try {
            // Validate customer
            const customer = await this.userService.getAnItem(orderData.CustomerId);
            if (!customer) {
                throw new Error('Customer not found');
            }

            // Validate products
            const productPromises = orderData.Items.map(item => 
                this.productService.getAnItem(item.ProductId)
            );
            const products = await Promise.all(productPromises);

            // Create order
            const order = await this.orderService.save(orderData);
            
            this.env.showMessage('ORDER_CREATED_SUCCESS', 'success');
            return order;
            
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }
}
```

---

**Tài liệu này sẽ được cập nhật khi có thêm methods mới hoặc thay đổi existing functionality.**

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
