# CommonService Documentation

## 📋 **Tổng quan**

`CommonService` là core HTTP client service của ứng dụng ART-ERP-FE, chịu trách nhiệm xử lý tất cả các API communications. Service này cung cấp unified interface cho HTTP requests, authentication headers, branch context, và error handling.

## 📍 **File location**
```
src/app/services/core/common.service.ts
```

## 🔧 **Dependencies**

```typescript
import { HttpClient, HttpHeaders, HttpParams } from '@angular/common/http';
import { EnvService } from './env.service';
import { ApiSetting } from '../static/api-setting';
import { lib } from '../static/global-functions';
```

## 🏗️ **Constructor**

```typescript
constructor(
    public http: HttpClient,
    public env: EnvService
) {}
```

## 🔨 **Core Methods**

### **1. connect(method, URL, data)**
Main method để thực hiện HTTP requests với automatic headers và branch context.

```typescript
connect(method: string, URL: string, data?: any): Observable<any>
```

**Parameters:**
- `method`: HTTP method ('GET', 'POST', 'PUT', 'DELETE', 'Login', 'UPLOAD')
- `URL`: API endpoint (relative hoặc absolute URL)
- `data`: Request payload (optional)

**Returns:** Observable với response data

**Automatic Features:**
- **Authentication headers** với Bearer token
- **Branch context** (IDBranch, SelectedBranch) tự động append
- **Data cleanup** (remove system fields như CreatedBy, ModifiedBy)
- **Content-Type** và App-Version headers
- **Dev mode detection** cho debugging

**Example:**
```typescript
// GET request
this.commonService.connect('GET', 'api/users').subscribe(users => {
    console.log('Users:', users);
});

// POST request với data
const newUser = { Name: 'John Doe', Email: 'john@example.com' };
this.commonService.connect('POST', 'api/users', newUser).subscribe(result => {
    console.log('User created:', result);
});

// PUT request để update
const updatedUser = { Id: 1, Name: 'Jane Doe', Email: 'jane@example.com' };
this.commonService.connect('PUT', 'api/users', updatedUser).subscribe(result => {
    console.log('User updated:', result);
});

// DELETE request
this.commonService.connect('DELETE', 'api/users/1').subscribe(result => {
    console.log('User deleted:', result);
});
```

### **2. getToken()**
Lấy authentication token từ storage.

```typescript
getToken(): string
```

**Returns:** Bearer token string hoặc empty string

**Example:**
```typescript
const token = this.commonService.getToken();
console.log('Current token:', token);
```

### **3. rq(method, URL, data)**
Alias method cho `connect()` với same functionality.

```typescript
rq(method: string, URL: string, data?: any): Observable<any>
```

**Example:**
```typescript
// Tương đương với connect()
this.commonService.rq('GET', 'api/products').subscribe(products => {
    console.log('Products:', products);
});
```

## 🎨 **Usage Examples**

### **1. Basic CRUD Operations**
```typescript
export class UserService extends ExtendService {
    
    async getUsers() {
        try {
            const users = await this.commonService.connect('GET', 'api/users').toPromise();
            return users;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }

    async createUser(user: any) {
        try {
            const result = await this.commonService.connect('POST', 'api/users', user).toPromise();
            this.env.showMessage('User created successfully', 'success');
            return result;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }

    async updateUser(user: any) {
        try {
            const result = await this.commonService.connect('PUT', 'api/users', user).toPromise();
            this.env.showMessage('User updated successfully', 'success');
            return result;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }

    async deleteUser(userId: number) {
        try {
            const result = await this.commonService.connect('DELETE', `api/users/${userId}`).toPromise();
            this.env.showMessage('User deleted successfully', 'success');
            return result;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }
}
```

### **2. Authentication Flow**
```typescript
export class AuthService {
    constructor(private commonService: CommonService, private env: EnvService) {}

    async login(credentials: { username: string, password: string }) {
        try {
            // Login method tự động set Basic Auth headers
            const tokenResponse = await this.commonService.connect('Login', 'auth/token', credentials).toPromise();
            
            // Store token for future requests
            await this.env.setStorage('authToken', tokenResponse.access_token);
            
            return tokenResponse;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }

    async refreshToken() {
        try {
            const refreshToken = await this.env.getStorage('refreshToken');
            const response = await this.commonService.connect('POST', 'auth/refresh', { 
                refresh_token: refreshToken 
            }).toPromise();
            
            await this.env.setStorage('authToken', response.access_token);
            return response;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }
}
```

### **3. File Upload**
```typescript
export class FileService {
    constructor(private commonService: CommonService, private env: EnvService) {}

    async uploadFile(file: File, additionalData?: any) {
        try {
            const formData = new FormData();
            formData.append('file', file);
            
            if (additionalData) {
                Object.keys(additionalData).forEach(key => {
                    formData.append(key, additionalData[key]);
                });
            }

            const result = await this.commonService.connect('UPLOAD', 'api/files/upload', formData).toPromise();
            this.env.showMessage('File uploaded successfully', 'success');
            return result;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }
}
```

### **4. Branch-Specific Data Loading**
```typescript
export class ProductService extends ExtendService {
    
    async getProductsByBranch(branchId?: string) {
        try {
            const data = branchId ? { IDBranch: branchId } : {};
            const products = await this.commonService.connect('GET', 'api/products', data).toPromise();
            return products;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }

    async getProductsIgnoreBranch() {
        try {
            // IgnoredBranch = true sẽ không append branch context
            const products = await this.commonService.connect('GET', 'api/products', { 
                IgnoredBranch: true 
            }).toPromise();
            return products;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }
}
```

### **5. Advanced Query với Parameters**
```typescript
export class ReportService {
    constructor(private commonService: CommonService, private env: EnvService) {}

    async getAdvancedReport(config: any) {
        try {
            const queryData = {
                _AdvanceConfig: config,  // Sẽ được encode thành base64
                DateFrom: '2024-01-01',
                DateTo: '2024-12-31',
                Status: ['Active', 'Pending']  // Array sẽ được JSON.stringify
            };

            const report = await this.commonService.connect('GET', 'api/reports/advanced', queryData).toPromise();
            return report;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }
}
```

## 🔒 **Security Features**

### **1. Automatic Authentication**
```typescript
// CommonService tự động thêm headers:
headers = new HttpHeaders({
    Authorization: this.getToken(),           // Bearer token
    'Content-Type': 'application/json',
    'Data-type': 'json',
    'App-Version': environment.appVersion
});
```

### **2. Data Sanitization**
```typescript
// Tự động remove system fields trước khi gửi request:
delete data.IsDeleted;
delete data.CreatedBy;
delete data.ModifiedBy;
delete data.CreatedDate;
delete data.ModifiedDate;
delete data.levels;
delete data.show;
delete data.showdetail;
delete data._state;
delete data.checked;
delete data.selected;
```

### **3. Branch Context Security**
```typescript
// Tự động append branch context nếu không có IgnoredBranch:
if (!data.hasOwnProperty('IgnoredBranch')) {
    URL += '?IDBranch=' + this.env.selectedBranchAndChildren;
    URL += '&SelectedBranch=' + this.env.selectedBranch;
}
```

## ⚡ **Performance Features**

### **1. Automatic URL Resolution**
```typescript
// Tự động resolve URL:
if (URL.indexOf('http') != 0) {
    URL = method != 'Login' 
        ? ApiSetting.apiDomain(URL)    // API domain cho regular requests
        : ApiSetting.appDomain(URL);   // App domain cho login
}
```

### **2. Efficient Parameter Handling**
```typescript
// GET requests: Convert data thành URL parameters
// POST/PUT requests: Send data trong request body
// Arrays và objects được properly serialized
```

### **3. Development Mode Support**
```typescript
// Tự động detect dev mode và thêm header:
if (!environment.production || GlobalData?.Token?.dev == 'test') {
    headers.append('IsDevMode', 'true');
}
```

## 🚨 **Error Handling**

### **1. HTTP Error Responses**
```typescript
// CommonService sẽ throw errors cho:
// - Network errors
// - HTTP status errors (4xx, 5xx)
// - Authentication failures
// - Validation errors

try {
    const result = await this.commonService.connect('GET', 'api/data').toPromise();
    return result;
} catch (error) {
    // Error object contains:
    // - error.status: HTTP status code
    // - error.message: Error message
    // - error.error: Server error details
    this.env.showErrorMessage(error);
    throw error;
}
```

### **2. Common Error Scenarios**
```typescript
// 401 Unauthorized - Token expired
catch (error) {
    if (error.status === 401) {
        // Redirect to login
        this.router.navigate(['/login']);
        return;
    }
    this.env.showErrorMessage(error);
}

// 403 Forbidden - No permission
catch (error) {
    if (error.status === 403) {
        this.env.showMessage('ACCESS_DENIED', 'danger');
        return;
    }
    this.env.showErrorMessage(error);
}

// 500 Server Error
catch (error) {
    if (error.status >= 500) {
        this.env.showMessage('SERVER_ERROR', 'danger');
        return;
    }
    this.env.showErrorMessage(error);
}
```

## 📋 **Best Practices**

### **1. Always Use CommonService for API Calls**
```typescript
// ✅ Đúng - Sử dụng CommonService
const data = await this.commonService.connect('GET', 'api/users').toPromise();

// ❌ Sai - Direct HttpClient usage
const data = await this.http.get('api/users').toPromise();
```

### **2. Handle Errors Properly**
```typescript
// ✅ Đúng - Proper error handling
try {
    const result = await this.commonService.connect('POST', 'api/data', payload).toPromise();
    this.env.showMessage('SUCCESS', 'success');
    return result;
} catch (error) {
    this.env.showErrorMessage(error);
    throw error;
}

// ❌ Sai - No error handling
const result = await this.commonService.connect('POST', 'api/data', payload).toPromise();
```

### **3. Use Appropriate HTTP Methods**
```typescript
// ✅ Đúng - Correct method usage
this.commonService.connect('GET', 'api/users');           // Read
this.commonService.connect('POST', 'api/users', user);    // Create
this.commonService.connect('PUT', 'api/users', user);     // Update
this.commonService.connect('DELETE', 'api/users/1');      // Delete

// ❌ Sai - Wrong methods
this.commonService.connect('POST', 'api/users');          // Should be GET
this.commonService.connect('GET', 'api/users', user);     // Should be POST
```

### **4. Branch Context Management**
```typescript
// ✅ Đúng - Let CommonService handle branch context
const data = await this.commonService.connect('GET', 'api/products').toPromise();

// ✅ Đúng - Explicit branch override
const data = await this.commonService.connect('GET', 'api/products', {
    IDBranch: 'specific-branch-id'
}).toPromise();

// ✅ Đúng - Ignore branch context
const data = await this.commonService.connect('GET', 'api/global-data', {
    IgnoredBranch: true
}).toPromise();

// ❌ Sai - Manual branch parameter
const data = await this.commonService.connect('GET', 
    `api/products?IDBranch=${branchId}`
).toPromise();
```

### **5. Observable vs Promise Usage**
```typescript
// ✅ Đúng - Observable cho reactive programming
this.commonService.connect('GET', 'api/users').subscribe(users => {
    this.users = users;
});

// ✅ Đúng - Promise cho async/await
const users = await this.commonService.connect('GET', 'api/users').toPromise();

// ✅ Đúng - Observable với operators
this.commonService.connect('GET', 'api/users').pipe(
    map(users => users.filter(u => u.IsActive)),
    catchError(error => {
        this.env.showErrorMessage(error);
        return of([]);
    })
).subscribe(activeUsers => {
    this.activeUsers = activeUsers;
});
```

## 🔧 **Integration với ExtendService**

```typescript
// ExtendService sử dụng CommonService internally
export class MyService extends ExtendService {
    constructor() {
        super();
        // this.commonService đã available
    }

    async customApiCall() {
        // Có thể sử dụng trực tiếp CommonService
        return this.commonService.connect('GET', 'api/custom-endpoint').toPromise();
    }
}
```

## 🎯 **Special Method Handling**

### **Login Method**
```typescript
// Login method có special handling:
// - Sử dụng Basic Authentication thay vì Bearer
// - Content-Type: application/x-www-form-urlencoded
// - Sử dụng appDomain thay vì apiDomain

const credentials = { username: 'user', password: 'pass' };
this.commonService.connect('Login', 'auth/token', credentials).subscribe(token => {
    // Handle token response
});
```

### **Upload Method**
```typescript
// Upload method cho file uploads:
// - Không set Content-Type (để browser tự set với boundary)
// - Support FormData objects

const formData = new FormData();
formData.append('file', file);
this.commonService.connect('UPLOAD', 'api/upload', formData).subscribe(result => {
    // Handle upload result
});
```

CommonService là foundation của tất cả API communications trong ART-ERP-FE, cung cấp consistent interface với automatic authentication, branch context, và error handling.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team