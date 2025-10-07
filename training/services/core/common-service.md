# CommonService Documentation

## 📋 **Tổng quan**

`CommonService` là core service để handle tất cả HTTP communications trong ứng dụng. Service này cung cấp methods để connect với APIs, handle authentication, caching, và error management.

**Location**: `src/app/services/core/common.service.ts`

---

## 🔧 **Key Methods**

### **HTTP Communication**
```typescript
connect(method: string, url: string, data?: any): Observable<any>
connectPromise(method: string, url: string, data?: any): Promise<any>
```

### **Authentication**
```typescript
setToken(token: string): void
getAuthHeaders(): HttpHeaders
```

### **Caching**
```typescript
clearCache(): void
getCachedData(key: string): any
setCachedData(key: string, data: any): void
```

---

## 🚀 **Basic Usage**

### **GET Request**
```typescript
export class DataService extends ExtendService {
    async loadUsers() {
        try {
            const users = await this.commonService.connect('GET', 'api/users').toPromise();
            return users;
        } catch (error) {
            this.env.showErrorMessage(error);
            throw error;
        }
    }
}
```

### **POST Request**
```typescript
async createUser(userData: any) {
    try {
        const result = await this.commonService.connect('POST', 'api/users', userData).toPromise();
        this.env.showMessage('User created successfully', 'success');
        return result;
    } catch (error) {
        this.env.showErrorMessage(error);
        throw error;
    }
}
```

### **File Upload**
```typescript
async uploadFile(file: File) {
    const formData = new FormData();
    formData.append('file', file);
    
    try {
        const result = await this.commonService.connect('POST', 'api/upload', formData).toPromise();
        return result;
    } catch (error) {
        this.env.showErrorMessage(error);
        throw error;
    }
}
```

---

## 🎯 **Best Practices**

### **1. Error Handling**
```typescript
// ✅ Đúng - Always handle errors
async apiCall() {
    try {
        const result = await this.commonService.connect('GET', 'api/data').toPromise();
        return result;
    } catch (error) {
        this.env.showErrorMessage(error);
        throw error; // Re-throw for caller
    }
}
```

### **2. Authentication**
```typescript
// ✅ Đúng - Service automatically handles auth headers
// No manual token management needed
```

### **3. Caching**
```typescript
// ✅ Đúng - Use caching for frequently accessed data
// CommonService handles caching automatically based on service configuration
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
