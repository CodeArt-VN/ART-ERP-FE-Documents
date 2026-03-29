# CustomService Documentation

## 📋 **Tổng quan**

`CustomService` là service để handle custom API connections và business logic không thuộc về các module cụ thể. Service này extend từ `exService` và provide flexible API connection methods cho các custom endpoints.

**Location**: `src/app/services/custom/custom.service.ts`  
**Injectable**: `providedIn: 'root'`  
**Extends**: `exService`

---

## 🔧 **Key Features**

### **Custom API Connection**
- **Flexible HTTP Methods**: Support GET, POST, PUT, DELETE
- **Dynamic URL Building**: Build URLs với environment domain
- **Custom Headers**: Support custom headers cho API calls
- **Error Handling**: Comprehensive error handling với retry logic
- **Promise-based**: Promise interface cho async operations

### **Third-party Integration**
- **Address Services**: Integration với address subdivision providers
- **External APIs**: Support cho external service connections
- **Custom Endpoints**: Flexible endpoint configuration

---

## 🚀 **Core Methods**

### **API Connection**
```typescript
API_connect(URL: string, query: any, method: string = 'GET'): Promise<any>
```

### **Address Services**
```typescript
getAddressSubdivision(query: any): Promise<any>
getProvinces(): Promise<any>
getDistricts(provinceId: string): Promise<any>
getWards(districtId: string): Promise<any>
```

### **Custom Endpoints**
```typescript
callCustomEndpoint(endpoint: string, data: any, method: string): Promise<any>
processCustomData(data: any): Promise<any>
```

---

## 🎨 **Usage Examples**

### **1. Basic API Connection**
```typescript
export class CustomDataComponent {
    constructor(private customService: CustomService) {}
    
    async loadCustomData() {
        try {
            const query = {
                page: 1,
                limit: 20,
                filter: 'active'
            };
            
            const result = await this.customService.API_connect(
                '/api/custom/data',
                query,
                'GET'
            );
            
            console.log('Custom data loaded:', result);
            return result;
            
        } catch (error) {
            console.error('Failed to load custom data:', error);
            this.env.showErrorMessage('Failed to load data');
        }
    }
    
    async saveCustomData(data: any) {
        try {
            const result = await this.customService.API_connect(
                '/api/custom/data',
                data,
                'POST'
            );
            
            this.env.showMessage('Data saved successfully', 'success');
            return result;
            
        } catch (error) {
            console.error('Failed to save data:', error);
            this.env.showErrorMessage('Failed to save data');
        }
    }
}
```

### **2. Address Services Integration**
```typescript
export class AddressFormComponent {
    provinces: any[] = [];
    districts: any[] = [];
    wards: any[] = [];
    
    constructor(private customService: CustomService) {}
    
    async ngOnInit() {
        await this.loadProvinces();
    }
    
    async loadProvinces() {
        try {
            this.provinces = await this.customService.getProvinces();
        } catch (error) {
            console.error('Failed to load provinces:', error);
        }
    }
    
    async onProvinceChange(provinceId: string) {
        try {
            this.districts = await this.customService.getDistricts(provinceId);
            this.wards = []; // Reset wards
        } catch (error) {
            console.error('Failed to load districts:', error);
        }
    }
    
    async onDistrictChange(districtId: string) {
        try {
            this.wards = await this.customService.getWards(districtId);
        } catch (error) {
            console.error('Failed to load wards:', error);
        }
    }
}
```

### **3. Custom Business Logic**
```typescript
export class BusinessProcessService {
    constructor(private customService: CustomService) {}
    
    async processBusinessRule(data: any): Promise<any> {
        try {
            // Step 1: Validate data
            const validationResult = await this.customService.API_connect(
                '/api/validation/business-rule',
                { data },
                'POST'
            );
            
            if (!validationResult.isValid) {
                throw new Error(validationResult.message);
            }
            
            // Step 2: Process business logic
            const processResult = await this.customService.API_connect(
                '/api/process/business-rule',
                { data, validationId: validationResult.id },
                'POST'
            );
            
            // Step 3: Update related entities
            await this.updateRelatedEntities(processResult.affectedEntities);
            
            return processResult;
            
        } catch (error) {
            console.error('Business rule processing failed:', error);
            throw error;
        }
    }
    
    private async updateRelatedEntities(entities: any[]) {
        const updatePromises = entities.map(entity => 
            this.customService.API_connect(
                `/api/entities/${entity.type}/${entity.id}`,
                entity.data,
                'PUT'
            )
        );
        
        await Promise.all(updatePromises);
    }
}
```

### **4. External API Integration**
```typescript
export class ExternalIntegrationService {
    constructor(private customService: CustomService) {}
    
    async syncWithExternalSystem(syncData: any) {
        try {
            // Prepare data for external system
            const externalData = this.transformToExternalFormat(syncData);
            
            // Send to external API
            const syncResult = await this.customService.API_connect(
                '/api/external/sync',
                externalData,
                'POST'
            );
            
            // Process sync result
            if (syncResult.success) {
                await this.updateSyncStatus(syncData.id, 'completed');
                this.env.showMessage('Sync completed successfully', 'success');
            } else {
                await this.updateSyncStatus(syncData.id, 'failed');
                throw new Error(syncResult.error);
            }
            
            return syncResult;
            
        } catch (error) {
            console.error('External sync failed:', error);
            await this.updateSyncStatus(syncData.id, 'error');
            this.env.showErrorMessage('Sync failed');
            throw error;
        }
    }
    
    private transformToExternalFormat(data: any): any {
        // Transform internal data format to external API format
        return {
            externalId: data.id,
            externalData: {
                name: data.name,
                description: data.description,
                metadata: data.customFields
            },
            timestamp: new Date().toISOString()
        };
    }
    
    private async updateSyncStatus(id: string, status: string) {
        await this.customService.API_connect(
            `/api/sync-status/${id}`,
            { status, updatedAt: new Date() },
            'PUT'
        );
    }
}
```

---

## 🔧 **Advanced Features**

### **1. Batch Operations**
```typescript
export class BatchOperationService {
    constructor(private customService: CustomService) {}
    
    async processBatchOperations(operations: any[]) {
        const batchSize = 10;
        const results = [];
        
        for (let i = 0; i < operations.length; i += batchSize) {
            const batch = operations.slice(i, i + batchSize);
            
            try {
                const batchResult = await this.customService.API_connect(
                    '/api/batch/process',
                    { operations: batch },
                    'POST'
                );
                
                results.push(...batchResult.results);
                
                // Progress update
                const progress = Math.round(((i + batch.length) / operations.length) * 100);
                this.env.publishEvent({
                    Code: 'batch:progress',
                    Value: { progress, processed: i + batch.length, total: operations.length }
                });
                
            } catch (error) {
                console.error(`Batch ${i / batchSize + 1} failed:`, error);
                // Continue with next batch
            }
        }
        
        return results;
    }
}
```

### **2. Caching Strategy**
```typescript
export class CachedCustomService extends CustomService {
    private cache = new Map<string, { data: any, timestamp: number }>();
    private cacheTimeout = 5 * 60 * 1000; // 5 minutes
    
    async API_connect_cached(URL: string, query: any, method: string = 'GET'): Promise<any> {
        if (method !== 'GET') {
            return super.API_connect(URL, query, method);
        }
        
        const cacheKey = this.generateCacheKey(URL, query);
        const cached = this.cache.get(cacheKey);
        
        if (cached && (Date.now() - cached.timestamp) < this.cacheTimeout) {
            console.log('Returning cached data for:', cacheKey);
            return cached.data;
        }
        
        try {
            const result = await super.API_connect(URL, query, method);
            
            this.cache.set(cacheKey, {
                data: result,
                timestamp: Date.now()
            });
            
            return result;
        } catch (error) {
            // Return cached data if available, even if expired
            if (cached) {
                console.warn('API failed, returning stale cache:', error);
                return cached.data;
            }
            throw error;
        }
    }
    
    private generateCacheKey(URL: string, query: any): string {
        return `${URL}:${JSON.stringify(query)}`;
    }
    
    clearCache() {
        this.cache.clear();
    }
}
```

### **3. Retry Logic**
```typescript
export class ResilientCustomService extends CustomService {
    async API_connect_with_retry(
        URL: string, 
        query: any, 
        method: string = 'GET',
        maxRetries: number = 3
    ): Promise<any> {
        let lastError: any;
        
        for (let attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                return await super.API_connect(URL, query, method);
            } catch (error) {
                lastError = error;
                
                if (attempt < maxRetries && this.isRetryableError(error)) {
                    const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
                    console.warn(`Attempt ${attempt} failed, retrying in ${delay}ms:`, error);
                    await new Promise(resolve => setTimeout(resolve, delay));
                } else {
                    break;
                }
            }
        }
        
        throw lastError;
    }
    
    private isRetryableError(error: any): boolean {
        // Retry on network errors, timeouts, and 5xx server errors
        return error.status >= 500 || 
               error.status === 0 || 
               error.name === 'TimeoutError';
    }
}
```

---

## 📋 **Best Practices**

### **1. Error Handling**
```typescript
// ✅ Comprehensive error handling
async handleApiCall() {
    try {
        const result = await this.customService.API_connect('/api/data', {}, 'GET');
        return result;
    } catch (error) {
        if (error.status === 401) {
            // Handle authentication error
            this.router.navigate(['/login']);
        } else if (error.status === 403) {
            // Handle authorization error
            this.env.showErrorMessage('Access denied');
        } else if (error.status >= 500) {
            // Handle server error
            this.env.showErrorMessage('Server error. Please try again later.');
        } else {
            // Handle other errors
            this.env.showErrorMessage('An error occurred');
        }
        throw error;
    }
}
```

### **2. Request Optimization**
```typescript
// ✅ Optimize requests
async loadDataOptimized() {
    const query = {
        fields: 'id,name,status', // Only request needed fields
        limit: 50,                // Reasonable page size
        cache: true              // Enable caching if supported
    };
    
    return this.customService.API_connect('/api/data', query, 'GET');
}
```

### **3. Progress Tracking**
```typescript
// ✅ Track long-running operations
async processLargeDataset(data: any[]) {
    const total = data.length;
    let processed = 0;
    
    for (const item of data) {
        await this.customService.API_connect('/api/process', item, 'POST');
        processed++;
        
        // Update progress
        this.env.publishEvent({
            Code: 'process:progress',
            Value: { processed, total, percentage: (processed / total) * 100 }
        });
    }
}
```

---

## 🚨 **Security Considerations**

### **1. Input Validation**
```typescript
// ✅ Validate inputs before API calls
validateApiInput(data: any): boolean {
    if (!data || typeof data !== 'object') {
        return false;
    }
    
    // Check for required fields
    const requiredFields = ['id', 'name'];
    for (const field of requiredFields) {
        if (!data[field]) {
            return false;
        }
    }
    
    // Sanitize string inputs
    for (const key in data) {
        if (typeof data[key] === 'string') {
            data[key] = this.sanitizeString(data[key]);
        }
    }
    
    return true;
}
```

### **2. URL Validation**
```typescript
// ✅ Validate URLs to prevent injection
validateUrl(url: string): boolean {
    // Only allow relative URLs or approved domains
    const allowedDomains = [environment.appDomain];
    
    if (url.startsWith('/')) {
        return true; // Relative URL
    }
    
    try {
        const urlObj = new URL(url);
        return allowedDomains.includes(urlObj.origin);
    } catch {
        return false;
    }
}
```

---

## 🎯 **Integration Examples**

### **With Form Components**
```typescript
export class CustomFormComponent {
    constructor(private customService: CustomService) {}
    
    async onSubmit(formData: any) {
        if (!this.validateForm(formData)) {
            return;
        }
        
        try {
            const result = await this.customService.API_connect(
                '/api/form/submit',
                formData,
                'POST'
            );
            
            this.env.showMessage('Form submitted successfully', 'success');
            this.router.navigate(['/success']);
            
        } catch (error) {
            this.env.showErrorMessage('Form submission failed');
        }
    }
}
```

### **With Data Tables**
```typescript
export class CustomDataTableComponent {
    constructor(private customService: CustomService) {}
    
    async loadTableData(query: any) {
        try {
            const result = await this.customService.API_connect(
                '/api/table/data',
                query,
                'GET'
            );
            
            return {
                data: result.items,
                count: result.totalCount
            };
            
        } catch (error) {
            console.error('Failed to load table data:', error);
            return { data: [], count: 0 };
        }
    }
}
```

CustomService cung cấp flexible foundation cho custom API integrations và business logic, với comprehensive error handling và optimization features cho enterprise applications.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
