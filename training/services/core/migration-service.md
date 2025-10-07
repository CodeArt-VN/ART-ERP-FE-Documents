# MigrationService Documentation

## 📋 **Tổng quan**

`MigrationService` là service để handle data migration khi có version changes hoặc server changes. Service này đảm bảo cache được clear appropriately và data consistency được maintain khi ứng dụng update.

**Location**: `src/app/services/core/migration.service.ts`  
**Injectable**: `providedIn: 'root'`

---

## 🔧 **Key Features**

### **Migration Detection**
- **Version Change Detection**: So sánh app version với stored version
- **Server Change Detection**: Detect tenant/server switching
- **Cache Management**: Clear specific cache keys based on migration type
- **Configuration-based**: Support skip migration via environment settings

### **Migration Result Interface**
```typescript
interface MigrationResult {
    versionChanged: boolean;
    serverChanged: boolean;
    clearedKeys: string[];
    success: boolean;
}
```

---

## 🚀 **Core Methods**

### **Main Migration Method**
```typescript
executeMigration(cache: CacheManagementService): Promise<MigrationResult>
```

### **Version Comparison**
```typescript
compareVersions(storedVersion: string, currentVersion: string): boolean
```

### **Cache Management**
```typescript
clearCacheKeys(keys: string[], cache: CacheManagementService): Promise<void>
getCurrentServer(): Promise<string>
```

---

## 🎨 **Usage Examples**

### **1. Basic Migration Execution**
```typescript
export class AppInitService {
    constructor(
        private migrationService: MigrationService,
        private cache: CacheManagementService
    ) {}
    
    async initializeApp() {
        // Execute migration on app startup
        const migrationResult = await this.migrationService.executeMigration(this.cache);
        
        if (migrationResult.success) {
            console.log('Migration completed successfully:', migrationResult);
            
            if (migrationResult.versionChanged) {
                console.log('App version updated, cache cleared');
            }
            
            if (migrationResult.serverChanged) {
                console.log('Server changed, tenant-specific cache cleared');
            }
        }
    }
}
```

### **2. Migration with Error Handling**
```typescript
export class StartupService {
    constructor(private migrationService: MigrationService) {}
    
    async performStartupMigration() {
        try {
            const result = await this.migrationService.executeMigration(this.cache);
            
            if (!result.success) {
                throw new Error('Migration failed');
            }
            
            // Log migration details
            this.logMigrationResult(result);
            
            // Proceed with app initialization
            await this.continueAppInit();
            
        } catch (error) {
            console.error('Migration error:', error);
            // Handle migration failure
            await this.handleMigrationFailure(error);
        }
    }
    
    private logMigrationResult(result: MigrationResult) {
        console.log('Migration Summary:', {
            versionChanged: result.versionChanged,
            serverChanged: result.serverChanged,
            clearedKeys: result.clearedKeys.length,
            success: result.success
        });
    }
}
```

### **3. Conditional Migration**
```typescript
export class ConfigService {
    constructor(private migrationService: MigrationService) {}
    
    async checkAndMigrate() {
        // Check if migration is needed
        const needsMigration = await this.checkMigrationNeeded();
        
        if (needsMigration) {
            const result = await this.migrationService.executeMigration(this.cache);
            
            if (result.versionChanged) {
                // Handle version-specific migrations
                await this.handleVersionMigration();
            }
            
            if (result.serverChanged) {
                // Handle server-specific migrations
                await this.handleServerMigration();
            }
        }
    }
    
    private async checkMigrationNeeded(): Promise<boolean> {
        const storedVersion = await this.cache.app.version;
        const currentVersion = environment.appVersion;
        
        return storedVersion !== currentVersion;
    }
}
```

### **4. Migration with User Notification**
```typescript
export class MigrationManagerService {
    constructor(
        private migrationService: MigrationService,
        private env: EnvService
    ) {}
    
    async performMigrationWithNotification() {
        // Show loading during migration
        await this.env.showLoading('Updating application...', 
            this.executeMigrationProcess()
        );
    }
    
    private async executeMigrationProcess() {
        const result = await this.migrationService.executeMigration(this.cache);
        
        if (result.success) {
            let message = 'Application updated successfully';
            
            if (result.versionChanged) {
                message += '. New features are now available!';
            }
            
            if (result.serverChanged) {
                message += '. Server configuration updated.';
            }
            
            this.env.showMessage(message, 'success');
        } else {
            this.env.showMessage('Migration failed. Please restart the application.', 'danger');
        }
        
        return result;
    }
}
```

---

## ⚙️ **Configuration**

### **Environment Settings**
```typescript
// environment.ts
export const environment = {
    appVersion: '2.1.0',
    migrationSettings: {
        skipMigration: false,
        enableVersionCheck: true,
        enableServerCheck: true
    },
    cacheKeysToClearOnNewVersion: [
        'userSettings',
        'appConfig',
        'menuCache',
        'permissionCache'
    ],
    cacheKeysToClearOnServerChange: [
        'branchList',
        'userProfile',
        'serverConfig',
        'tenantSettings'
    ]
};

// environment.prod.ts
export const environment = {
    appVersion: '2.1.0',
    migrationSettings: {
        skipMigration: false,
        enableVersionCheck: true,
        enableServerCheck: true
    },
    cacheKeysToClearOnNewVersion: [
        'userSettings',
        'appConfig',
        'menuCache'
    ],
    cacheKeysToClearOnServerChange: [
        'branchList',
        'userProfile',
        'serverConfig'
    ]
};
```

### **Migration Configuration Interface**
```typescript
interface MigrationSettings {
    skipMigration: boolean;
    enableVersionCheck: boolean;
    enableServerCheck: boolean;
    customMigrationRules?: MigrationRule[];
}

interface MigrationRule {
    fromVersion: string;
    toVersion: string;
    cacheKeysTooClear: string[];
    customActions?: () => Promise<void>;
}
```

---

## 🔧 **Advanced Usage**

### **1. Custom Migration Rules**
```typescript
export class CustomMigrationService extends MigrationService {
    private customRules: MigrationRule[] = [
        {
            fromVersion: '1.0.0',
            toVersion: '2.0.0',
            cacheKeysTooClear: ['oldUserData', 'deprecatedSettings'],
            customActions: async () => {
                // Migrate old data format to new format
                await this.migrateUserDataFormat();
            }
        }
    ];
    
    async executeCustomMigration(cache: CacheManagementService): Promise<MigrationResult> {
        const baseResult = await super.executeMigration(cache);
        
        // Apply custom migration rules
        for (const rule of this.customRules) {
            if (this.shouldApplyRule(rule)) {
                await this.applyCustomRule(rule, cache);
                baseResult.clearedKeys.push(...rule.cacheKeysTooClear);
            }
        }
        
        return baseResult;
    }
    
    private shouldApplyRule(rule: MigrationRule): boolean {
        const storedVersion = this.cache.app.version;
        return this.isVersionInRange(storedVersion, rule.fromVersion, rule.toVersion);
    }
}
```

### **2. Migration Rollback**
```typescript
export class MigrationRollbackService {
    constructor(private migrationService: MigrationService) {}
    
    async rollbackMigration(backupData: any): Promise<void> {
        try {
            // Restore backed up cache data
            await this.restoreCacheData(backupData);
            
            // Reset version to previous
            await this.resetVersionToPrevious();
            
            console.log('Migration rolled back successfully');
        } catch (error) {
            console.error('Rollback failed:', error);
            throw error;
        }
    }
    
    private async createBackup(): Promise<any> {
        return {
            version: await this.cache.app.version,
            tenant: await this.cache.app.tenant,
            cacheData: await this.cache.getCacheRegistry()
        };
    }
}
```

### **3. Migration Progress Tracking**
```typescript
export class MigrationProgressService {
    private migrationProgress$ = new BehaviorSubject<MigrationProgress>({
        step: 'idle',
        progress: 0,
        message: ''
    });
    
    async executeMigrationWithProgress(): Promise<MigrationResult> {
        this.updateProgress('starting', 0, 'Initializing migration...');
        
        try {
            this.updateProgress('version-check', 25, 'Checking version changes...');
            const versionChanged = await this.checkVersionChange();
            
            this.updateProgress('server-check', 50, 'Checking server changes...');
            const serverChanged = await this.checkServerChange();
            
            this.updateProgress('cache-clear', 75, 'Clearing cache...');
            await this.clearRequiredCache();
            
            this.updateProgress('complete', 100, 'Migration completed');
            
            return {
                versionChanged,
                serverChanged,
                clearedKeys: this.getClearedKeys(),
                success: true
            };
        } catch (error) {
            this.updateProgress('error', 0, `Migration failed: ${error.message}`);
            throw error;
        }
    }
    
    private updateProgress(step: string, progress: number, message: string) {
        this.migrationProgress$.next({ step, progress, message });
    }
    
    getProgress(): Observable<MigrationProgress> {
        return this.migrationProgress$.asObservable();
    }
}
```

---

## 📋 **Best Practices**

### **1. Migration Safety**
```typescript
// ✅ Always backup before migration
async safeMigration() {
    const backup = await this.createBackup();
    
    try {
        const result = await this.migrationService.executeMigration(this.cache);
        return result;
    } catch (error) {
        await this.rollbackMigration(backup);
        throw error;
    }
}

// ✅ Validate migration result
async validateMigration(result: MigrationResult) {
    if (!result.success) {
        throw new Error('Migration validation failed');
    }
    
    // Additional validation logic
    await this.verifyDataIntegrity();
}
```

### **2. Error Handling**
```typescript
// ✅ Comprehensive error handling
async handleMigrationError(error: any) {
    console.error('Migration error:', error);
    
    // Log error details
    await this.logMigrationError(error);
    
    // Notify user
    this.env.showErrorMessage('Migration failed. Please contact support.');
    
    // Attempt recovery
    await this.attemptRecovery();
}
```

### **3. Performance Optimization**
```typescript
// ✅ Optimize cache clearing
async optimizedCacheClear(keys: string[]) {
    // Clear in batches to avoid blocking UI
    const batchSize = 10;
    for (let i = 0; i < keys.length; i += batchSize) {
        const batch = keys.slice(i, i + batchSize);
        await this.clearCacheBatch(batch);
        
        // Allow UI to update
        await new Promise(resolve => setTimeout(resolve, 10));
    }
}
```

---

## 🚨 **Error Handling**

### **Common Error Scenarios**
```typescript
// Version comparison errors
try {
    const versionChanged = this.compareVersions(storedVersion, currentVersion);
} catch (error) {
    console.error('Version comparison failed:', error);
    // Fallback to force migration
}

// Cache clearing errors
try {
    await this.clearCacheKeys(keys, cache);
} catch (error) {
    console.error('Cache clearing failed:', error);
    // Continue with partial success
}

// Server detection errors
try {
    const currentServer = await this.getCurrentServer();
} catch (error) {
    console.error('Server detection failed:', error);
    // Use fallback server
}
```

---

## 🎯 **Integration Examples**

### **With App Module**
```typescript
@NgModule({
    providers: [
        MigrationService,
        {
            provide: APP_INITIALIZER,
            useFactory: (migration: MigrationService, cache: CacheManagementService) => {
                return () => migration.executeMigration(cache);
            },
            deps: [MigrationService, CacheManagementService],
            multi: true
        }
    ]
})
export class AppModule {}
```

### **With Route Guards**
```typescript
@Injectable()
export class MigrationGuard implements CanActivate {
    constructor(private migrationService: MigrationService) {}
    
    async canActivate(): Promise<boolean> {
        const result = await this.migrationService.executeMigration(this.cache);
        return result.success;
    }
}
```

### **With Service Worker**
```typescript
export class ServiceWorkerMigrationService {
    constructor(private migrationService: MigrationService) {}
    
    async handleAppUpdate() {
        // Service worker detected app update
        const result = await this.migrationService.executeMigration(this.cache);
        
        if (result.versionChanged) {
            // Reload app to apply updates
            window.location.reload();
        }
    }
}
```

---

## 📱 **Mobile Considerations**

### **Offline Migration**
```typescript
async handleOfflineMigration() {
    if (!navigator.onLine) {
        // Queue migration for when online
        await this.queueMigrationForLater();
        return { success: true, deferred: true };
    }
    
    return await this.migrationService.executeMigration(this.cache);
}
```

### **Storage Limitations**
```typescript
async checkStorageBeforeMigration() {
    if ('storage' in navigator && 'estimate' in navigator.storage) {
        const estimate = await navigator.storage.estimate();
        const availableSpace = estimate.quota - estimate.usage;
        
        if (availableSpace < this.MINIMUM_SPACE_REQUIRED) {
            throw new Error('Insufficient storage for migration');
        }
    }
}
```

MigrationService cung cấp robust solution để handle app updates và data migrations, đảm bảo smooth user experience khi có version changes hoặc server switching trong enterprise environment.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
