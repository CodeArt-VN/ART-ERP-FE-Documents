# CacheManagementService Documentation

## 📄 **Tổng quan**

`CacheManagementService` là service quản lý cache toàn diện cho ứng dụng ART-ERP-FE. Service này cung cấp caching layer với TTL (Time To Live), auto-refresh, retry mechanisms, và application state management.

## 🎯 **Mục đích sử dụng**

- **Application State Management**: Quản lý app version, tenant, user profile, selected branch
- **Intelligent Caching**: TTL-based caching với auto-refresh và retry logic
- **Performance Optimization**: Reduce API calls và improve response time
- **Offline Support**: Cache data cho offline functionality
- **Migration Support**: Handle app updates và data migrations

## 📍 **File location**
```
src/app/services/core/cache-management.service.ts
```

## 🔧 **Dependencies**

```typescript
import { StorageService } from './storage.service';
import { MigrationService } from './migration.service';
import { ICacheItem, CacheConfig, CacheStrategy, ICacheStats, RetryConfig } from '../static/search-config';
```

## 🏗️ **Properties**

### **Application State**
```typescript
public app = {
    version: null,           // App version
    tenant: string,          // Current tenant URL
    userId: null,           // Current user ID
    lang: string,           // Current language
    theme: string,          // Current theme
    token: string,          // Auth token
    userProfile: any,       // User profile object
    selectedBranch: string  // Selected branch ID
}
```

### **Cache Configuration**
```typescript
private defaultConfig: CacheConfig = {
    enable: true,
    timeToLive: 60,         // 1 minute TTL
    expireAction: 'remove',
    maintenanceInterval: 15, // 15 seconds
    autoRefresh: true,
    retryConfig: {
        maxRetries: 3,
        retryInterval: 5    // 5 minutes
    },
    valueKey: 'Cache_'
}
```

## 🔨 **Core Methods**

### **1. Tracking và Initialization**

#### **tracking()**
Observable để theo dõi trạng thái ready của cache service.

```typescript
tracking(): Observable<boolean>
```

**Returns:** Observable<boolean> - true khi service đã ready

**Example:**
```typescript
this.cacheService.tracking().subscribe(isReady => {
    if (isReady) {
        console.log('Cache service is ready');
        this.loadData();
    }
});
```

### **2. Cache Operations**

#### **get(key, strategy?, branch?)**
Lấy data từ cache với strategy và branch context.

```typescript
get(key: string, strategy: CacheStrategy = 'auto', branch?: string): Promise<any>
```

**Parameters:**
- `key`: Cache key
- `strategy`: 'auto' | 'cache-first' | 'network-first' | 'cache-only' | 'network-only'
- `branch`: Branch context (optional)

**Example:**
```typescript
// Auto strategy - sử dụng cache nếu valid, otherwise fetch từ network
const userData = await this.cacheService.get('UserProfile', 'auto');

// Cache-first strategy
const products = await this.cacheService.get('Products', 'cache-first', 'branch-123');

// Network-first strategy
const orders = await this.cacheService.get('Orders', 'network-first');
```

#### **set(key, value, config?, branch?, serviceName?)**
Lưu data vào cache với configuration.

```typescript
set(key: string, value: any, config?: CacheConfig, branch?: string, serviceName?: string): Promise<void>
```

**Parameters:**
- `key`: Cache key
- `value`: Data cần cache
- `config`: Cache configuration (optional)
- `branch`: Branch context (optional)
- `serviceName`: Service name for tracking (optional)

**Example:**
```typescript
// Basic caching
await this.cacheService.set('UserPreferences', preferences);

// Caching với custom TTL
await this.cacheService.set('ProductList', products, {
    enable: true,
    timeToLive: 300, // 5 minutes
    autoRefresh: true
});

// Branch-specific caching
await this.cacheService.set('BranchData', data, null, 'branch-123', 'BranchService');
```

#### **remove(key, branch?)**
Xóa item khỏi cache.

```typescript
remove(key: string, branch?: string): Promise<void>
```

**Example:**
```typescript
// Remove cache item
await this.cacheService.remove('UserProfile');

// Remove branch-specific item
await this.cacheService.remove('BranchData', 'branch-123');
```

#### **clear()**
Xóa toàn bộ cache.

```typescript
clear(): Promise<void>
```

**Example:**
```typescript
// Clear all cache
await this.cacheService.clear();
```

### **3. Root Storage Operations**

#### **getRoot(key)**
Lấy data từ root storage (không có branch context).

```typescript
getRoot(key: string): Promise<any>
```

#### **setRoot(key, value)**
Lưu data vào root storage.

```typescript
setRoot(key: string, value: any): Promise<void>
```

#### **removeRoot(key)**
Xóa data từ root storage.

```typescript
removeRoot(key: string): Promise<void>
```

**Example:**
```typescript
// Root storage operations
await this.cacheService.setRoot('AppVersion', '1.0.0');
const version = await this.cacheService.getRoot('AppVersion');
await this.cacheService.removeRoot('TempData');
```

### **4. Cache Management**

#### **keys()**
Lấy danh sách tất cả cache keys.

```typescript
keys(): string[]
```

#### **getCacheStats()**
Lấy thống kê cache usage.

```typescript
getCacheStats(): ICacheStats
```

#### **maintain()**
Thực hiện maintenance (cleanup expired items).

```typescript
maintain(): Promise<void>
```

**Example:**
```typescript
// Get cache statistics
const stats = this.cacheService.getCacheStats();
console.log('Cache stats:', stats);

// Manual maintenance
await this.cacheService.maintain();

// Get all cache keys
const keys = this.cacheService.keys();
console.log('Cache keys:', keys);
```

## 🎨 **Usage Examples**

### **1. Service Integration Pattern**
```typescript
export class ProductService extends ExtendService {
    constructor(
        private cacheService: CacheManagementService
    ) {
        super();
    }

    async getProducts(forceRefresh = false) {
        const strategy = forceRefresh ? 'network-first' : 'auto';
        
        try {
            const products = await this.cacheService.get('ProductList', strategy);
            return products;
        } catch (error) {
            console.error('Failed to get products:', error);
            throw error;
        }
    }

    async saveProduct(product: any) {
        try {
            const result = await this.commonService.connect('POST', 'api/products', product).toPromise();
            
            // Invalidate cache after save
            await this.cacheService.remove('ProductList');
            
            return result;
        } catch (error) {
            throw error;
        }
    }
}
```

### **2. Branch-Aware Caching**
```typescript
export class BranchDataService {
    constructor(private cacheService: CacheManagementService) {}

    async getBranchData(branchId: string) {
        const cacheKey = 'BranchSettings';
        
        try {
            // Get branch-specific cached data
            const data = await this.cacheService.get(cacheKey, 'auto', branchId);
            return data;
        } catch (error) {
            console.error('Failed to get branch data:', error);
            throw error;
        }
    }

    async updateBranchData(branchId: string, data: any) {
        const cacheKey = 'BranchSettings';
        
        try {
            // Save to API
            const result = await this.api.updateBranchData(branchId, data);
            
            // Update cache
            await this.cacheService.set(cacheKey, result, {
                enable: true,
                timeToLive: 600 // 10 minutes
            }, branchId);
            
            return result;
        } catch (error) {
            throw error;
        }
    }
}
```

### **3. Application State Management**
```typescript
export class AppStateService {
    constructor(private cacheService: CacheManagementService) {}

    async initializeApp() {
        // Wait for cache service to be ready
        await new Promise(resolve => {
            this.cacheService.tracking().subscribe(isReady => {
                if (isReady) resolve(true);
            });
        });

        // Access application state
        const appState = this.cacheService.app;
        console.log('Current user:', appState.userProfile);
        console.log('Selected branch:', appState.selectedBranch);
        console.log('Current language:', appState.lang);
        
        return appState;
    }

    async switchBranch(branchId: string) {
        // Update selected branch
        this.cacheService.app.selectedBranch = branchId;
        
        // Persist to storage
        await this.cacheService.set('SelectedBranch', branchId);
        
        // Clear branch-specific cache
        const keys = this.cacheService.keys();
        for (const key of keys) {
            if (key.includes('branch-specific')) {
                await this.cacheService.remove(key);
            }
        }
    }
}
```

### **4. Cache Strategy Examples**
```typescript
export class DataService {
    constructor(private cacheService: CacheManagementService) {}

    // Cache-first: Prefer cache, fallback to network
    async getCachedData() {
        return this.cacheService.get('StaticData', 'cache-first');
    }

    // Network-first: Always try network, fallback to cache
    async getFreshData() {
        return this.cacheService.get('DynamicData', 'network-first');
    }

    // Cache-only: Only use cache, no network calls
    async getOfflineData() {
        return this.cacheService.get('OfflineData', 'cache-only');
    }

    // Network-only: Always fetch from network, no cache
    async getRealTimeData() {
        return this.cacheService.get('RealTimeData', 'network-only');
    }

    // Auto: Intelligent strategy based on TTL and availability
    async getSmartData() {
        return this.cacheService.get('SmartData', 'auto');
    }
}
```

### **5. Performance Monitoring**
```typescript
export class CacheMonitoringService {
    constructor(private cacheService: CacheManagementService) {}

    getCachePerformance() {
        const stats = this.cacheService.getCacheStats();
        
        return {
            totalItems: stats.totalItems,
            hitRate: stats.hits / (stats.hits + stats.misses) * 100,
            memoryUsage: stats.memoryUsage,
            expiredItems: stats.expiredItems,
            errorRate: stats.errors / stats.totalRequests * 100
        };
    }

    async optimizeCache() {
        // Get performance metrics
        const performance = this.getCachePerformance();
        
        if (performance.hitRate < 50) {
            console.warn('Low cache hit rate, consider adjusting TTL');
        }
        
        if (performance.expiredItems > 100) {
            console.log('Running cache maintenance...');
            await this.cacheService.maintain();
        }
        
        return performance;
    }
}
```

## ⚡ **Cache Strategies**

### **1. Auto Strategy**
```typescript
// Intelligent caching - recommended for most use cases
const data = await this.cacheService.get('key', 'auto');
```
- Sử dụng cache nếu còn valid (within TTL)
- Fetch từ network nếu cache expired hoặc không tồn tại
- Auto-refresh cache khi có data mới

### **2. Cache-First Strategy**
```typescript
// Prefer cache over network - good for static data
const data = await this.cacheService.get('key', 'cache-first');
```
- Luôn check cache trước
- Chỉ fetch từ network nếu cache không có
- Tốt cho static data ít thay đổi

### **3. Network-First Strategy**
```typescript
// Prefer fresh data - good for dynamic content
const data = await this.cacheService.get('key', 'network-first');
```
- Luôn try fetch từ network trước
- Fallback về cache nếu network fail
- Tốt cho data thường xuyên thay đổi

### **4. Cache-Only Strategy**
```typescript
// Offline mode - only use cached data
const data = await this.cacheService.get('key', 'cache-only');
```
- Chỉ sử dụng cache, không network calls
- Tốt cho offline functionality
- Throw error nếu cache không có

### **5. Network-Only Strategy**
```typescript
// Always fresh - bypass cache completely
const data = await this.cacheService.get('key', 'network-only');
```
- Luôn fetch từ network
- Không sử dụng cache
- Tốt cho real-time data

## 🔧 **Configuration Options**

### **CacheConfig Interface**
```typescript
interface CacheConfig {
    enable: boolean;              // Enable/disable caching
    timeToLive: number;          // TTL in seconds
    expireAction: 'remove' | 'refresh'; // Action when expired
    maintenanceInterval: number;  // Maintenance interval in seconds
    autoRefresh: boolean;        // Auto-refresh expired items
    retryConfig: RetryConfig;    // Retry configuration
    valueKey: string;            // Storage key prefix
}
```

### **RetryConfig Interface**
```typescript
interface RetryConfig {
    maxRetries: number;          // Maximum retry attempts
    retryInterval: number;       // Interval between retries (seconds)
}
```

## 📋 **Best Practices**

### **1. Choose Appropriate Cache Strategy**
```typescript
// ✅ Static data - use cache-first
const countries = await this.cacheService.get('Countries', 'cache-first');

// ✅ Dynamic data - use auto or network-first
const orders = await this.cacheService.get('Orders', 'auto');

// ✅ Real-time data - use network-only
const liveData = await this.cacheService.get('LiveData', 'network-only');
```

### **2. Set Appropriate TTL**
```typescript
// ✅ Short TTL for frequently changing data
await this.cacheService.set('Orders', orders, { timeToLive: 60 }); // 1 minute

// ✅ Long TTL for static data
await this.cacheService.set('Countries', countries, { timeToLive: 86400 }); // 1 day
```

### **3. Handle Cache Invalidation**
```typescript
// ✅ Invalidate related cache after updates
async updateProduct(product: any) {
    await this.api.updateProduct(product);
    
    // Invalidate related caches
    await this.cacheService.remove('ProductList');
    await this.cacheService.remove(`Product_${product.Id}`);
}
```

### **4. Monitor Cache Performance**
```typescript
// ✅ Regular performance monitoring
setInterval(() => {
    const stats = this.cacheService.getCacheStats();
    if (stats.hitRate < 0.5) {
        console.warn('Cache hit rate is low:', stats.hitRate);
    }
}, 60000); // Check every minute
```

### **5. Branch-Aware Caching**
```typescript
// ✅ Use branch context for multi-tenant data
const branchData = await this.cacheService.get('Settings', 'auto', this.currentBranch);

// ✅ Clear branch cache when switching
async switchBranch(newBranchId: string) {
    await this.cacheService.remove('Settings', this.currentBranch);
    this.currentBranch = newBranchId;
}
```

## 🚨 **Error Handling**

```typescript
try {
    const data = await this.cacheService.get('key', 'auto');
    return data;
} catch (error) {
    if (error.message.includes('Cache miss')) {
        // Handle cache miss
        console.log('Data not in cache, fetching from API');
    } else if (error.message.includes('Network error')) {
        // Handle network error
        console.error('Network failed, using stale cache if available');
    } else {
        // Handle other errors
        console.error('Cache operation failed:', error);
    }
    throw error;
}
```

CacheManagementService cung cấp intelligent caching layer với advanced features như TTL, auto-refresh, retry mechanisms, và application state management cho ứng dụng ART-ERP-FE.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
