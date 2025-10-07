# StorageService Documentation

## 📄 **Tổng quan**

`StorageService` là wrapper service cho Ionic Storage, cung cấp persistent storage layer cho ứng dụng ART-ERP-FE. Service này handle initialization, provide async operations, và ensure data persistence across app sessions.

## 🎯 **Mục đích sử dụng**

- **Persistent Storage**: Lưu trữ data persistent qua app sessions
- **Async Operations**: Provide async interface cho storage operations
- **Initialization Management**: Handle storage initialization và ready state
- **Cross-platform Support**: Hoạt động trên web, iOS, và Android
- **Foundation Layer**: Base layer cho CacheManagementService

## 📍 **File location**
```
src/app/services/core/storage.service.ts
```

## 🔧 **Dependencies**

```typescript
import { Storage } from '@ionic/storage-angular';
import { BehaviorSubject, Observable } from 'rxjs';
```

## 🏗️ **Properties**

### **Private Properties**
```typescript
private _tracking$: BehaviorSubject<boolean>    // Tracking ready state
private _storage: Storage | null                // Ionic Storage instance
private _initialized: boolean                   // Initialization flag
private _initPromise: Promise<void> | null      // Initialization promise
```

## 🔨 **Core Methods**

### **1. Initialization**

#### **tracking()**
Observable để theo dõi trạng thái ready của storage.

```typescript
tracking(): Observable<boolean>
```

**Returns:** Observable<boolean> - true khi storage đã ready

**Example:**
```typescript
this.storageService.tracking().subscribe(isReady => {
    if (isReady) {
        console.log('Storage is ready');
        this.loadStoredData();
    }
});
```

#### **init()**
Initialize storage service.

```typescript
init(): Promise<void>
```

**Example:**
```typescript
await this.storageService.init();
console.log('Storage initialized');
```

### **2. Basic Operations**

#### **get(key)**
Lấy value từ storage theo key.

```typescript
get(key: string): Promise<any>
```

**Parameters:**
- `key`: Storage key

**Returns:** Promise với stored value hoặc null

**Example:**
```typescript
// Get stored value
const userPrefs = await this.storageService.get('userPreferences');
if (userPrefs) {
    console.log('User preferences:', userPrefs);
}

// Get with fallback
const theme = await this.storageService.get('theme') || 'default';
```

#### **set(key, value)**
Lưu value vào storage.

```typescript
set(key: string, value: any): Promise<void>
```

**Parameters:**
- `key`: Storage key
- `value`: Value to store (any serializable type)

**Example:**
```typescript
// Store simple value
await this.storageService.set('theme', 'dark');

// Store object
await this.storageService.set('userPreferences', {
    language: 'vi-VN',
    notifications: true,
    autoSave: false
});

// Store array
await this.storageService.set('recentItems', [1, 2, 3, 4, 5]);
```

#### **remove(key)**
Xóa key khỏi storage.

```typescript
remove(key: string): Promise<void>
```

**Example:**
```typescript
// Remove specific key
await this.storageService.remove('tempData');

// Remove user session data
await this.storageService.remove('authToken');
```

#### **clear()**
Xóa toàn bộ storage.

```typescript
clear(): Promise<void>
```

**Example:**
```typescript
// Clear all storage (use with caution)
await this.storageService.clear();
console.log('All storage cleared');
```

#### **keys()**
Lấy danh sách tất cả keys trong storage.

```typescript
keys(): Promise<string[]>
```

**Example:**
```typescript
// Get all storage keys
const allKeys = await this.storageService.keys();
console.log('Storage keys:', allKeys);

// Check if specific key exists
const keys = await this.storageService.keys();
const hasUserData = keys.includes('userData');
```

#### **length()**
Lấy số lượng items trong storage.

```typescript
length(): Promise<number>
```

**Example:**
```typescript
const itemCount = await this.storageService.length();
console.log(`Storage contains ${itemCount} items`);
```

#### **forEach(callback)**
Iterate qua tất cả items trong storage.

```typescript
forEach(callback: (value: any, key: string, iterationNumber: number) => void): Promise<void>
```

**Example:**
```typescript
// Iterate through all storage items
await this.storageService.forEach((value, key, index) => {
    console.log(`${index}: ${key} = ${JSON.stringify(value)}`);
});
```

## 🎨 **Usage Examples**

### **1. User Preferences Management**
```typescript
export class UserPreferencesService {
    constructor(private storageService: StorageService) {}

    async savePreferences(preferences: any) {
        try {
            await this.storageService.set('userPreferences', preferences);
            console.log('Preferences saved');
        } catch (error) {
            console.error('Failed to save preferences:', error);
        }
    }

    async loadPreferences() {
        try {
            const preferences = await this.storageService.get('userPreferences');
            return preferences || this.getDefaultPreferences();
        } catch (error) {
            console.error('Failed to load preferences:', error);
            return this.getDefaultPreferences();
        }
    }

    private getDefaultPreferences() {
        return {
            theme: 'default',
            language: 'vi-VN',
            notifications: true,
            autoSave: true
        };
    }
}
```

### **2. Cache Implementation**
```typescript
export class SimpleCacheService {
    constructor(private storageService: StorageService) {}

    async cacheData(key: string, data: any, ttl: number = 3600) {
        const cacheItem = {
            data: data,
            timestamp: Date.now(),
            ttl: ttl * 1000 // Convert to milliseconds
        };

        await this.storageService.set(`cache_${key}`, cacheItem);
    }

    async getCachedData(key: string) {
        try {
            const cacheItem = await this.storageService.get(`cache_${key}`);
            
            if (!cacheItem) return null;

            // Check if cache is expired
            const now = Date.now();
            if (now - cacheItem.timestamp > cacheItem.ttl) {
                await this.storageService.remove(`cache_${key}`);
                return null;
            }

            return cacheItem.data;
        } catch (error) {
            console.error('Failed to get cached data:', error);
            return null;
        }
    }

    async clearExpiredCache() {
        const keys = await this.storageService.keys();
        const cacheKeys = keys.filter(key => key.startsWith('cache_'));

        for (const key of cacheKeys) {
            const cacheItem = await this.storageService.get(key);
            if (cacheItem) {
                const now = Date.now();
                if (now - cacheItem.timestamp > cacheItem.ttl) {
                    await this.storageService.remove(key);
                }
            }
        }
    }
}
```

### **3. Session Management**
```typescript
export class SessionService {
    constructor(private storageService: StorageService) {}

    async saveSession(sessionData: any) {
        const session = {
            ...sessionData,
            lastActivity: Date.now(),
            sessionId: this.generateSessionId()
        };

        await this.storageService.set('currentSession', session);
    }

    async getSession() {
        const session = await this.storageService.get('currentSession');
        
        if (!session) return null;

        // Check session timeout (24 hours)
        const sessionTimeout = 24 * 60 * 60 * 1000;
        if (Date.now() - session.lastActivity > sessionTimeout) {
            await this.clearSession();
            return null;
        }

        return session;
    }

    async updateLastActivity() {
        const session = await this.getSession();
        if (session) {
            session.lastActivity = Date.now();
            await this.storageService.set('currentSession', session);
        }
    }

    async clearSession() {
        await this.storageService.remove('currentSession');
    }

    private generateSessionId(): string {
        return Date.now().toString(36) + Math.random().toString(36).substr(2);
    }
}
```

### **4. Offline Data Management**
```typescript
export class OfflineDataService {
    constructor(private storageService: StorageService) {}

    async saveOfflineData(type: string, data: any[]) {
        const offlineKey = `offline_${type}`;
        await this.storageService.set(offlineKey, {
            data: data,
            timestamp: Date.now(),
            synced: false
        });
    }

    async getOfflineData(type: string) {
        const offlineKey = `offline_${type}`;
        const offlineData = await this.storageService.get(offlineKey);
        return offlineData?.data || [];
    }

    async markAsSynced(type: string) {
        const offlineKey = `offline_${type}`;
        const offlineData = await this.storageService.get(offlineKey);
        
        if (offlineData) {
            offlineData.synced = true;
            offlineData.syncedAt = Date.now();
            await this.storageService.set(offlineKey, offlineData);
        }
    }

    async getPendingSyncData() {
        const keys = await this.storageService.keys();
        const offlineKeys = keys.filter(key => key.startsWith('offline_'));
        const pendingData = [];

        for (const key of offlineKeys) {
            const data = await this.storageService.get(key);
            if (data && !data.synced) {
                pendingData.push({
                    type: key.replace('offline_', ''),
                    data: data.data,
                    timestamp: data.timestamp
                });
            }
        }

        return pendingData;
    }
}
```

### **5. Storage Debugging và Monitoring**
```typescript
export class StorageDebugService {
    constructor(private storageService: StorageService) {}

    async getStorageInfo() {
        const keys = await this.storageService.keys();
        const length = await this.storageService.length();
        
        const info = {
            totalKeys: length,
            keys: keys,
            estimatedSize: 0
        };

        // Calculate estimated storage size
        for (const key of keys) {
            const value = await this.storageService.get(key);
            const size = JSON.stringify(value).length;
            info.estimatedSize += size;
        }

        return info;
    }

    async exportStorage() {
        const keys = await this.storageService.keys();
        const exportData = {};

        for (const key of keys) {
            exportData[key] = await this.storageService.get(key);
        }

        return exportData;
    }

    async importStorage(data: any) {
        for (const [key, value] of Object.entries(data)) {
            await this.storageService.set(key, value);
        }
    }

    async cleanupStorage() {
        const keys = await this.storageService.keys();
        
        // Remove temporary keys
        const tempKeys = keys.filter(key => 
            key.startsWith('temp_') || 
            key.startsWith('cache_') ||
            key.includes('_expired')
        );

        for (const key of tempKeys) {
            await this.storageService.remove(key);
        }

        console.log(`Cleaned up ${tempKeys.length} temporary keys`);
    }
}
```

## ⚡ **Performance Considerations**

### **1. Initialization Pattern**
```typescript
// ✅ Wait for storage to be ready
export class MyService {
    constructor(private storageService: StorageService) {
        this.storageService.tracking().subscribe(isReady => {
            if (isReady) {
                this.initializeService();
            }
        });
    }
}
```

### **2. Batch Operations**
```typescript
// ✅ Efficient batch operations
async saveBatchData(items: any[]) {
    const promises = items.map((item, index) => 
        this.storageService.set(`item_${index}`, item)
    );
    
    await Promise.all(promises);
}
```

### **3. Error Handling**
```typescript
// ✅ Proper error handling
async safeGet(key: string, defaultValue: any = null) {
    try {
        const value = await this.storageService.get(key);
        return value !== null ? value : defaultValue;
    } catch (error) {
        console.error(`Storage get error for key ${key}:`, error);
        return defaultValue;
    }
}
```

## 📋 **Best Practices**

### **1. Always Check Initialization**
```typescript
// ✅ Wait for storage ready
this.storageService.tracking().subscribe(isReady => {
    if (isReady) {
        // Safe to use storage operations
    }
});

// ❌ Don't use storage immediately
// this.storageService.get('key'); // May fail if not initialized
```

### **2. Handle Errors Gracefully**
```typescript
// ✅ Proper error handling
try {
    await this.storageService.set('key', value);
} catch (error) {
    console.error('Storage operation failed:', error);
    // Provide fallback or user feedback
}

// ❌ No error handling
// await this.storageService.set('key', value); // May throw unhandled errors
```

### **3. Use Meaningful Key Names**
```typescript
// ✅ Descriptive key names
await this.storageService.set('user_preferences_v2', preferences);
await this.storageService.set('cache_product_list_branch_123', products);

// ❌ Generic key names
await this.storageService.set('data', preferences);
await this.storageService.set('temp', products);
```

### **4. Implement Data Versioning**
```typescript
// ✅ Version your stored data
const dataWithVersion = {
    version: '1.0',
    data: actualData,
    timestamp: Date.now()
};
await this.storageService.set('user_data', dataWithVersion);

// When reading, check version
const stored = await this.storageService.get('user_data');
if (stored?.version !== '1.0') {
    // Handle migration or reset
}
```

### **5. Clean Up Unused Data**
```typescript
// ✅ Regular cleanup
async performMaintenance() {
    const keys = await this.storageService.keys();
    
    // Remove old temporary data
    const oldTempKeys = keys.filter(key => {
        return key.startsWith('temp_') && 
               this.isOlderThan(key, 24 * 60 * 60 * 1000); // 24 hours
    });
    
    for (const key of oldTempKeys) {
        await this.storageService.remove(key);
    }
}
```

## 🚨 **Error Scenarios**

### **Common Errors**
```typescript
// Storage not initialized
try {
    await this.storageService.get('key');
} catch (error) {
    if (error.message.includes('not initialized')) {
        // Wait for initialization
        await this.storageService.init();
    }
}

// Storage quota exceeded
try {
    await this.storageService.set('large_data', hugeObject);
} catch (error) {
    if (error.name === 'QuotaExceededError') {
        // Clean up old data or notify user
        await this.cleanupOldData();
    }
}

// Serialization errors
try {
    await this.storageService.set('circular_ref', objectWithCircularRef);
} catch (error) {
    if (error.message.includes('circular')) {
        // Handle circular reference
        const cleanObject = this.removeCircularReferences(objectWithCircularRef);
        await this.storageService.set('circular_ref', cleanObject);
    }
}
```

## 🔧 **Integration với Other Services**

### **CacheManagementService Integration**
```typescript
// StorageService là foundation cho CacheManagementService
export class CacheManagementService {
    constructor(private storage: StorageService) {
        // CacheManagementService sử dụng StorageService internally
    }
}
```

### **EnvService Integration**
```typescript
// EnvService sử dụng StorageService cho persistent storage
export class EnvService {
    async setStorage(key: string, value: any) {
        // Delegates to StorageService through CacheManagementService
        return this.cacheService.set(key, value);
    }
}
```

StorageService cung cấp reliable, cross-platform storage foundation cho toàn bộ ứng dụng ART-ERP-FE với proper initialization handling và error management.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
