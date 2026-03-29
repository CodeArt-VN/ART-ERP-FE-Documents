# Services Documentation - ART-ERP-FE

## 📋 **Mục đích tài liệu**

Thư mục này chứa tài liệu hướng dẫn sử dụng chi tiết cho tất cả services trong `src/app/services/`. Services cung cấp business logic, data management, và utility functions cho toàn bộ ứng dụng.

---

## 📊 **Thống kê Services**

- **Tổng số services**: 25+ services
- **Đã document**: 15 services chính  
- **Coverage**: Tất cả services quan trọng và core functionality

---

## 📂 **Danh sách Services đã document**

### **🔧 Core Services (5 services)**
- **[CommonService](./core/common-service.md)** - HTTP client và API communication
- **[EnvService](./core/env-service.md)** - Environment management và app context
- **[ExtendService](./core/extend-service.md)** - Base service cho CRUD operations
- **[CacheManagementService](./core/cache-management.md)** - Data caching và performance optimization
- **[StorageService](./core/storage-service.md)** - Local storage và data persistence

### **🔐 Authentication Services (4 services)**
- **[AuthenticationService](./auth/authentication-service.md)** - User authentication và login
- **[ExternalAuthService](./auth/external-auth-service.md)** - Third-party authentication
- **[UserContextService](./auth/user-context-service.md)** - User session management
- **[UserProfileService](./auth/user-profile-service.md)** - User profile data management

### **🎯 Custom Services (6 services)**
- **[ReportService](./custom/report-service.md)** - Report generation và data visualization
- **[NotificationsService](./custom/notifications-service.md)** - Push notifications và messaging
- **[SystemConfigService](./custom/system-config-service.md)** - System configuration management
- **[EInvoiceService](./custom/einvoice-service.md)** - Electronic invoice processing
- **[PromotionService](./custom/promotion-service.md)** - Promotion và discount management
- **[CustomService](./custom/custom-service.md)** - Dynamic script loading và utilities

### **🛠️ Utility Services (4 services)**
- **[PrintingService](./util/printing-service.md)** - Document printing và PDF generation
- **[BarcodeScannerService](./util/barcode-scanner-service.md)** - Barcode scanning functionality
- **[TranslateLoaderService](./util/translate-loader-service.md)** - Dynamic translation loading
- **[UtilityService](./util/utility-service.md)** - General utility functions

### **📄 Page Services (3 services)**
- **[BusinessLogicService](./page/business-logic-service.md)** - Business rules và validation
- **[DataManagementService](./page/data-management-service.md)** - Data CRUD operations
- **[FormManagementService](./page/form-management-service.md)** - Form handling và validation

---

## 🚀 **Quick Start Guide**

### **1. Service Injection**
```typescript
// Import services
import { EnvService } from 'src/app/services/core/env.service';
import { CommonService } from 'src/app/services/core/common.service';

@Component({
    selector: 'app-sample',
    templateUrl: './sample.component.html'
})
export class SampleComponent {
    constructor(
        private env: EnvService,
        private commonService: CommonService
    ) {}
}
```

### **2. Basic Service Usage**
```typescript
// Using EnvService
export class SamplePage extends PageBase {
    async ngOnInit() {
        // Show loading
        await this.env.showLoading('Loading data...', this.loadData());
        
        // Show message
        this.env.showMessage('Data loaded successfully', 'success');
        
        // Check permissions
        const canEdit = await this.env.checkFormPermission('/users/edit');
    }
    
    async loadData() {
        try {
            const data = await this.commonService.connect('GET', 'api/users').toPromise();
            return data;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }
}
```

### **3. Service Extension Pattern**
```typescript
// Extending base service
import { ExtendService } from 'src/app/services/core/extend-service';

@Injectable({ providedIn: 'root' })
export class UserService extends ExtendService {
    apiPath = 'users';
    
    constructor(public commonService: CommonService) {
        super();
    }
    
    // Custom methods
    async getUsersByRole(role: string) {
        return await this.read({ Role: role });
    }
    
    async activateUser(userId: number) {
        return await this.commonService.connect('POST', `${this.apiPath}/${userId}/activate`).toPromise();
    }
}
```

---

## 📋 **Service Categories**

### **Core Services**
Essential services cho basic app functionality, HTTP communication, caching, và environment management.

### **Authentication Services**
Services để handle user authentication, session management, và security.

### **Custom Services**
Business-specific services cho reports, notifications, system configuration.

### **Utility Services**
Helper services cho printing, barcode scanning, translations, và general utilities.

### **Page Services**
Services để support page-level functionality như business logic và form management.

---

## 🎯 **Best Practices**

### **1. Service Architecture**
```typescript
// ✅ Đúng - Follow service hierarchy
export class CustomService extends ExtendService {
    // Inherit base CRUD functionality
    // Add custom business logic
}

// ✅ Đúng - Proper dependency injection
constructor(
    private env: EnvService,
    private commonService: CommonService
) {
    super(); // Call parent constructor
}
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Consistent error handling
async getData() {
    try {
        const result = await this.commonService.connect('GET', 'api/data').toPromise();
        return result;
    } catch (error) {
        this.env.showErrorMessage(error);
        throw error; // Re-throw for caller handling
    }
}
```

### **3. Caching Strategy**
```typescript
// ✅ Đúng - Use caching for performance
export class DataService extends ExtendService {
    allowCache = true;
    cacheConfig = {
        ttl: 300000, // 5 minutes
        maxItems: 100
    };
}
```

### **4. Service Configuration**
```typescript
// ✅ Đúng - Configure service properly
export class ProductService extends ExtendService {
    apiPath = 'products';
    searchField = 'Name';
    allowCache = true;
    serviceName = 'ProductService';
}
```

---

## 🔧 **Development Guidelines**

### **Creating New Services**
```typescript
@Injectable({ providedIn: 'root' })
export class NewService extends ExtendService {
    apiPath = 'new-endpoint';
    
    constructor(public commonService: CommonService) {
        super();
    }
    
    // Custom methods
    async customMethod(params: any) {
        return await this.commonService.connect('POST', `${this.apiPath}/custom`, params).toPromise();
    }
}
```

### **Service Testing**
```typescript
describe('NewService', () => {
    let service: NewService;
    let httpMock: HttpTestingController;
    
    beforeEach(() => {
        TestBed.configureTestingModule({
            imports: [HttpClientTestingModule],
            providers: [NewService]
        });
        
        service = TestBed.inject(NewService);
        httpMock = TestBed.inject(HttpTestingController);
    });
    
    it('should fetch data', () => {
        const mockData = { id: 1, name: 'Test' };
        
        service.getAnItem(1).then(result => {
            expect(result).toEqual(mockData);
        });
        
        const req = httpMock.expectOne('api/new-endpoint/1');
        expect(req.request.method).toBe('GET');
        req.flush(mockData);
    });
});
```

### **Service Registration**
```typescript
// In app.module.ts or feature module
@NgModule({
    providers: [
        // Core services are usually providedIn: 'root'
        // Custom services can be provided at module level
        CustomService,
        UtilityService
    ]
})
export class AppModule {}
```

---

## 🚨 **Common Patterns**

### **1. Data Loading Pattern**
```typescript
export class DataPage extends PageBase {
    items: any[] = [];
    loading = false;
    
    async loadData() {
        try {
            this.loading = true;
            this.items = await this.dataService.read();
        } catch (error) {
            this.env.showErrorMessage(error);
        } finally {
            this.loading = false;
        }
    }
}
```

### **2. Form Submission Pattern**
```typescript
export class FormPage extends PageBase {
    async saveForm() {
        try {
            if (this.formGroup.valid) {
                const result = await this.dataService.save(this.formGroup.value);
                this.env.showMessage('Saved successfully', 'success');
                this.nav('/list', 'back');
            } else {
                this.env.showMessage('Please check form data', 'warning');
            }
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

### **3. Permission Check Pattern**
```typescript
export class SecurePage extends PageBase {
    canEdit = false;
    canDelete = false;
    
    async ngOnInit() {
        this.canEdit = await this.env.checkFormPermission('/users/edit');
        this.canDelete = await this.env.checkFormPermission('/users/delete');
    }
}
```

---

## 📚 **Related Documentation**

- **[Components](../components/README.md)** - Components that use these services
- **[Pipes](../pipes/README.md)** - Pipes that integrate with services
- **[Global Functions](../js-lib.md)** - Utility functions used by services

---

## 🔄 **Service Integration Examples**

### **Complete CRUD Example**
```typescript
export class UserManagementPage extends PageBase {
    users: User[] = [];
    selectedUsers: User[] = [];
    
    constructor(
        private userService: UserService,
        private env: EnvService
    ) {
        super();
    }
    
    async ngOnInit() {
        await this.loadUsers();
    }
    
    async loadUsers() {
        try {
            this.users = await this.userService.read();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    async deleteUsers() {
        try {
            const confirmed = await this.env.actionConfirm('delete', this.selectedUsers.length, 'users');
            if (confirmed) {
                await this.userService.delete(this.selectedUsers);
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

## ⚠️ **Performance Considerations**

### **1. Caching Strategy**
- Use caching for frequently accessed data
- Configure appropriate TTL values
- Clear cache when data changes

### **2. HTTP Optimization**
- Batch API calls when possible
- Use pagination for large datasets
- Implement request debouncing

### **3. Memory Management**
- Unsubscribe from observables
- Clear large data sets when not needed
- Use OnPush change detection strategy

---

**Tài liệu này sẽ được cập nhật khi có services mới hoặc thay đổi.**

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
