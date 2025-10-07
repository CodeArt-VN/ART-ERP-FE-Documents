# ExtendService (providerService) Documentation

## 📋 **Tổng quan**

`providerService` (also known as `ExtendService`) là base service class cung cấp standardized interface cho CRUD operations với caching support. Service này acts as wrapper around `CommonService` để provide consistent data access patterns across different modules.

**Location**: `src/app/services/core/extend-service.ts`  
**Injectable**: `providedIn: 'root'`

---

## 🔧 **Key Properties**

### **Configuration Properties**
```typescript
apiPath: any;                    // API path configuration
searchField: string[] = [];      // Fields to search in local searches
allowCache: boolean = true;      // Enable/disable caching
serviceName: string = '';        // Service identifier
showCommandRules: any[] = [];    // UI command rules
cacheConfig: CacheConfig;        // Cache configuration
commonService: CommonService;    // Reference to CommonService
```

---

## 🚀 **Core Methods**

### **Data Reading**
```typescript
getAnItem(Id: any, UID: string = ''): Promise<any>
read(query: any = null, forceReload = false): Promise<{count: number, data: any[]}>
readServer(query: any = null): Promise<{count: number, data: any[]}>
search(query: any = null): Observable<any>
```

### **Data Modification**
```typescript
save(item: any, isForceCreate = false): Promise<any>
delete(items: any): Promise<any>
cancel(items: any): Promise<any>
disable(items: any, IsDisabled = true): Promise<any>
changeBranch(item: any): Promise<any>
```

### **Approval Workflow**
```typescript
submitForApproval(items: any, apiPath: any): Promise<any>
approve(items: any, apiPath: any): Promise<any>
disapprove(items: any, apiPath: any): Promise<any>
```

### **File Operations**
```typescript
import(fileToUpload: File): Promise<any>
export(query: any): Promise<any>
upload(fileToUpload: File): Promise<any>
download(query: any): Promise<any>
```

---

## 🚀 **Basic Usage**

### **Creating a Custom Service**
```typescript
@Injectable({ providedIn: 'root' })
export class UserService extends providerService {
    constructor(public commonService: CommonService) {
        super();
        this.apiPath = UserApiPath;
        this.serviceName = 'UserService';
        this.allowCache = true;
        this.searchField = ['Name', 'Email', 'Phone'];
        this.cacheConfig = {
            enable: true,
            timeToLive: 60, // 1 hour
            expireAction: 'remove'
        };
    }
    
    // Custom methods
    async getUsersByRole(role: string) {
        return await this.read({ Role: role });
    }
    
    async activateUser(userId: number) {
        const user = await this.getAnItem(userId);
        user.IsActive = true;
        return await this.save(user);
    }
}
```

### **Basic CRUD Operations**
```typescript
export class UserManagementPage extends PageBase {
    users: User[] = [];
    
    constructor(private userService: UserService) {
        super();
    }
    
    async ngOnInit() {
        await this.loadUsers();
    }
    
    // READ operations
    async loadUsers() {
        try {
            const result = await this.userService.read();
            this.users = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    async loadUser(id: number) {
        try {
            const user = await this.userService.getAnItem(id);
            return user;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    // CREATE/UPDATE operations
    async saveUser(user: User) {
        try {
            const result = await this.userService.save(user);
            this.env.showMessage('User saved successfully', 'success');
            await this.loadUsers(); // Refresh list
            return result;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    // DELETE operations
    async deleteUsers(users: User[]) {
        try {
            const confirmed = await this.env.actionConfirm('delete', users.length, 'users');
            if (confirmed) {
                await this.userService.delete(users);
                this.env.showMessage('Users deleted successfully', 'success');
                await this.loadUsers();
            }
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

---

## 🎨 **Advanced Usage**

### **Search Operations**
```typescript
export class ProductSearchPage extends PageBase {
    products: Product[] = [];
    searchQuery = '';
    
    constructor(private productService: ProductService) {
        super();
    }
    
    async searchProducts() {
        try {
            // Using search method (returns Observable)
            this.productService.search({ 
                Keyword: this.searchQuery,
                Category: 'Electronics'
            }).subscribe(results => {
                this.products = results;
            });
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    async loadProductsWithFilter() {
        try {
            // Using read method with query
            const result = await this.productService.read({
                Page: 1,
                PageSize: 20,
                Status: 'Active',
                Category: 'Electronics'
            });
            
            this.products = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **File Operations**
```typescript
export class DataManagementPage extends PageBase {
    constructor(private dataService: DataService) {
        super();
    }
    
    // Import data from file
    async importData(event: any) {
        const file = event.target.files[0];
        if (!file) return;
        
        try {
            await this.env.showLoading('Importing data...', 
                this.dataService.import(file)
            );
            
            this.env.showMessage('Data imported successfully', 'success');
            await this.loadData(); // Refresh data
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    // Export data
    async exportData() {
        try {
            const query = {
                Format: 'excel',
                DateFrom: this.dateFrom,
                DateTo: this.dateTo,
                Status: 'Active'
            };
            
            await this.dataService.export(query);
            this.env.showMessage('Export completed', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    // Upload file
    async uploadFile(event: any) {
        const file = event.target.files[0];
        if (!file) return;
        
        try {
            const result = await this.dataService.upload(file);
            this.env.showMessage('File uploaded successfully', 'success');
            return result;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    // Download file
    async downloadFile(itemId: number) {
        try {
            await this.dataService.download({ Id: itemId });
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **Approval Workflow**
```typescript
export class OrderApprovalPage extends PageBase {
    orders: Order[] = [];
    selectedOrders: Order[] = [];
    
    constructor(private orderService: OrderService) {
        super();
    }
    
    // Submit for approval
    async submitForApproval() {
        try {
            const confirmed = await this.env.actionConfirm(
                'submit for approval', 
                this.selectedOrders.length, 
                'orders'
            );
            
            if (confirmed) {
                await this.orderService.submitForApproval(
                    this.selectedOrders, 
                    this.orderService.apiPath
                );
                
                this.env.showMessage('Orders submitted for approval', 'success');
                await this.loadOrders();
            }
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    // Approve orders
    async approveOrders() {
        try {
            const confirmed = await this.env.actionConfirm(
                'approve', 
                this.selectedOrders.length, 
                'orders'
            );
            
            if (confirmed) {
                await this.orderService.approve(
                    this.selectedOrders, 
                    this.orderService.apiPath
                );
                
                this.env.showMessage('Orders approved successfully', 'success');
                await this.loadOrders();
            }
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    // Disapprove orders
    async disapproveOrders() {
        try {
            const confirmed = await this.env.actionConfirm(
                'disapprove', 
                this.selectedOrders.length, 
                'orders'
            );
            
            if (confirmed) {
                await this.orderService.disapprove(
                    this.selectedOrders, 
                    this.orderService.apiPath
                );
                
                this.env.showMessage('Orders disapproved', 'warning');
                await this.loadOrders();
            }
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **Caching Configuration**
```typescript
@Injectable({ providedIn: 'root' })
export class CachedDataService extends providerService {
    constructor(public commonService: CommonService) {
        super();
        this.apiPath = DataApiPath;
        this.serviceName = 'CachedDataService';
        
        // Configure caching
        this.allowCache = true;
        this.cacheConfig = {
            enable: true,
            timeToLive: 30, // 30 minutes
            expireAction: 'update', // Auto-refresh when expired
            autoRefresh: true,
            retryConfig: {
                maxRetries: 3,
                retryInterval: 5 // 5 minutes
            }
        };
    }
    
    // Force reload from server
    async forceReload() {
        return await this.read(null, true); // forceReload = true
    }
    
    // Read from server only
    async readFromServer(query: any = null) {
        return await this.readServer(query);
    }
}
```

---

## 🔧 **Best Practices**

### **1. Service Configuration**
```typescript
// ✅ Đúng - Complete service setup
@Injectable({ providedIn: 'root' })
export class ProductService extends providerService {
    constructor(public commonService: CommonService) {
        super();
        this.apiPath = ProductApiPath;
        this.serviceName = 'ProductService';
        this.allowCache = true;
        this.searchField = ['Name', 'Code', 'Description'];
        this.cacheConfig = {
            enable: true,
            timeToLive: 60,
            expireAction: 'remove'
        };
    }
}
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Proper error handling
async loadData() {
    try {
        const result = await this.dataService.read();
        this.items = result.data;
    } catch (error) {
        this.env.showErrorMessage(error);
        // Handle specific error cases
        if (error.status === 404) {
            this.items = [];
        }
    }
}
```

### **3. Cache Management**
```typescript
// ✅ Đúng - Smart cache usage
export class SmartCacheService extends providerService {
    constructor(public commonService: CommonService) {
        super();
        this.allowCache = true;
        this.cacheConfig = {
            enable: true,
            timeToLive: 60, // 1 hour for stable data
            expireAction: 'update' // Auto-refresh
        };
    }
    
    // Force refresh when data changes
    async saveItem(item: any) {
        const result = await this.save(item);
        // Force reload to get fresh data
        await this.read(null, true);
        return result;
    }
}
```

### **4. Bulk Operations**
```typescript
// ✅ Đúng - Efficient bulk operations
async bulkUpdate(items: any[]) {
    try {
        // Process in batches to avoid overwhelming server
        const batchSize = 10;
        const batches = [];
        
        for (let i = 0; i < items.length; i += batchSize) {
            batches.push(items.slice(i, i + batchSize));
        }
        
        for (const batch of batches) {
            await Promise.all(batch.map(item => this.save(item)));
        }
        
        this.env.showMessage('Bulk update completed', 'success');
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

---

## 🚨 **Common Use Cases**

### **1. Master Data Services**
```typescript
export class CategoryService extends providerService {
    // For master data with caching
    allowCache = true;
    cacheConfig = { timeToLive: 120 }; // 2 hours
}
```

### **2. Transaction Services**
```typescript
export class OrderService extends providerService {
    // For transactional data
    allowCache = false; // Always get fresh data
}
```

### **3. Report Services**
```typescript
export class ReportService extends providerService {
    // For reports with short cache
    cacheConfig = { timeToLive: 5 }; // 5 minutes
}
```

---

## ⚠️ **Important Notes**

1. **Always extend providerService** for consistent API patterns
2. **Configure caching appropriately** based on data volatility
3. **Use proper error handling** in all operations
4. **Set up apiPath correctly** for your endpoints
5. **Consider performance** when enabling/disabling cache

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
