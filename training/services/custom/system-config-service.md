# SYS_ConfigService Documentation

## 📋 **Tổng quan**

`SYS_ConfigService` là service để handle system configuration management trong hệ thống ERP. Service này extend từ `SYS_ConfigProvider` và provide functionality cho việc load, cache và manage các system configurations theo branch và inheritance hierarchy.

**Location**: `src/app/services/custom/system-config.service.ts`  
**Injectable**: `providedIn: 'root'`  
**Extends**: `SYS_ConfigProvider`

---

## 🔧 **Key Features**

### **Configuration Management**
- **Branch-specific Configs**: Load configurations theo branch cụ thể
- **Configuration Inheritance**: Support inherited configurations từ parent branches
- **Key-based Filtering**: Load specific configuration keys
- **JSON Value Parsing**: Automatic JSON parsing cho configuration values
- **Null Value Handling**: Handle null values với inherited fallbacks

### **Caching & Performance**
- **Automatic Caching**: Cache configurations để improve performance
- **Branch Context**: Integrate với EnvService cho branch context
- **Lazy Loading**: Load configurations on-demand

---

## 🚀 **Core Methods**

### **Configuration Loading**
```typescript
getConfig(IDBranch?: number, keys?: string[]): Promise<any>
```

### **Parameters**
- `IDBranch`: Branch ID (optional, defaults to current selected branch)
- `keys`: Array of configuration keys to load (optional, loads all if not specified)

### **Return Format**
```typescript
{
    [configKey: string]: any // Parsed JSON values
}
```

---

## 🎨 **Usage Examples**

### **1. Load All Configurations**
```typescript
export class SystemSettingsComponent implements OnInit {
    systemConfigs: any = {};
    loading: boolean = false;
    
    constructor(private configService: SYS_ConfigService) {}
    
    async ngOnInit() {
        await this.loadSystemConfigs();
    }
    
    async loadSystemConfigs() {
        this.loading = true;
        
        try {
            // Load all configurations for current branch
            this.systemConfigs = await this.configService.getConfig();
            
            console.log('System configurations loaded:', this.systemConfigs);
            
            // Apply configurations to UI
            this.applyConfigurations();
            
        } catch (error) {
            console.error('Failed to load system configurations:', error);
            this.env.showErrorMessage('Failed to load system settings');
        } finally {
            this.loading = false;
        }
    }
    
    private applyConfigurations() {
        // Apply theme configuration
        if (this.systemConfigs.AppTheme) {
            this.applyTheme(this.systemConfigs.AppTheme);
        }
        
        // Apply language configuration
        if (this.systemConfigs.DefaultLanguage) {
            this.env.setLang(this.systemConfigs.DefaultLanguage);
        }
        
        // Apply business rules
        if (this.systemConfigs.BusinessRules) {
            this.applyBusinessRules(this.systemConfigs.BusinessRules);
        }
    }
}
```

### **2. Load Specific Configuration Keys**
```typescript
export class FeatureConfigService {
    constructor(private configService: SYS_ConfigService) {}
    
    async loadFeatureConfigs(): Promise<any> {
        try {
            const featureKeys = [
                'EnableAdvancedSearch',
                'EnableBulkOperations',
                'EnableRealTimeUpdates',
                'MaxFileUploadSize',
                'SessionTimeout'
            ];
            
            const configs = await this.configService.getConfig(null, featureKeys);
            
            return {
                advancedSearch: configs.EnableAdvancedSearch || false,
                bulkOperations: configs.EnableBulkOperations || false,
                realTimeUpdates: configs.EnableRealTimeUpdates || true,
                maxFileSize: configs.MaxFileUploadSize || 10485760, // 10MB default
                sessionTimeout: configs.SessionTimeout || 1800000 // 30 minutes default
            };
            
        } catch (error) {
            console.error('Failed to load feature configurations:', error);
            
            // Return default configurations
            return this.getDefaultFeatureConfigs();
        }
    }
    
    async isFeatureEnabled(featureName: string): Promise<boolean> {
        try {
            const configs = await this.configService.getConfig(null, [featureName]);
            return configs[featureName] === true;
        } catch (error) {
            console.error(`Failed to check feature ${featureName}:`, error);
            return false;
        }
    }
    
    private getDefaultFeatureConfigs(): any {
        return {
            advancedSearch: false,
            bulkOperations: false,
            realTimeUpdates: true,
            maxFileSize: 10485760,
            sessionTimeout: 1800000
        };
    }
}
```

### **3. Branch-specific Configuration Loading**
```typescript
export class BranchConfigurationService {
    constructor(private configService: SYS_ConfigService) {}
    
    async loadBranchConfigurations(branchId: number): Promise<any> {
        try {
            const branchConfigs = await this.configService.getConfig(branchId);
            
            // Process configurations with inheritance
            const processedConfigs = this.processInheritedConfigs(branchConfigs);
            
            return processedConfigs;
            
        } catch (error) {
            console.error(`Failed to load configurations for branch ${branchId}:`, error);
            throw error;
        }
    }
    
    async compareBranchConfigurations(branchId1: number, branchId2: number): Promise<any> {
        try {
            const [configs1, configs2] = await Promise.all([
                this.configService.getConfig(branchId1),
                this.configService.getConfig(branchId2)
            ]);
            
            return this.compareConfigurations(configs1, configs2);
            
        } catch (error) {
            console.error('Failed to compare branch configurations:', error);
            throw error;
        }
    }
    
    private processInheritedConfigs(configs: any): any {
        const processed = {};
        
        for (const key in configs) {
            const config = configs[key];
            
            // Check if this is an inherited configuration
            if (config._isInherited) {
                processed[key] = {
                    value: config.value,
                    isInherited: true,
                    inheritedFrom: config._inheritedFrom
                };
            } else {
                processed[key] = {
                    value: config.value,
                    isInherited: false
                };
            }
        }
        
        return processed;
    }
    
    private compareConfigurations(configs1: any, configs2: any): any {
        const comparison = {
            same: {},
            different: {},
            onlyInFirst: {},
            onlyInSecond: {}
        };
        
        const allKeys = new Set([...Object.keys(configs1), ...Object.keys(configs2)]);
        
        for (const key of allKeys) {
            if (configs1[key] && configs2[key]) {
                if (JSON.stringify(configs1[key]) === JSON.stringify(configs2[key])) {
                    comparison.same[key] = configs1[key];
                } else {
                    comparison.different[key] = {
                        branch1: configs1[key],
                        branch2: configs2[key]
                    };
                }
            } else if (configs1[key]) {
                comparison.onlyInFirst[key] = configs1[key];
            } else {
                comparison.onlyInSecond[key] = configs2[key];
            }
        }
        
        return comparison;
    }
}
```

### **4. Configuration Caching Service**
```typescript
export class ConfigCacheService {
    private configCache = new Map<string, { data: any, timestamp: number }>();
    private cacheTimeout = 5 * 60 * 1000; // 5 minutes
    
    constructor(private configService: SYS_ConfigService) {}
    
    async getConfigWithCache(branchId?: number, keys?: string[]): Promise<any> {
        const cacheKey = this.generateCacheKey(branchId, keys);
        const cached = this.configCache.get(cacheKey);
        
        // Return cached data if valid
        if (cached && (Date.now() - cached.timestamp) < this.cacheTimeout) {
            return cached.data;
        }
        
        try {
            // Load fresh data
            const configs = await this.configService.getConfig(branchId, keys);
            
            // Cache the result
            this.configCache.set(cacheKey, {
                data: configs,
                timestamp: Date.now()
            });
            
            return configs;
            
        } catch (error) {
            // Return stale cache if available
            if (cached) {
                console.warn('Using stale cache due to error:', error);
                return cached.data;
            }
            
            throw error;
        }
    }
    
    invalidateCache(branchId?: number, keys?: string[]) {
        if (branchId || keys) {
            // Invalidate specific cache entry
            const cacheKey = this.generateCacheKey(branchId, keys);
            this.configCache.delete(cacheKey);
        } else {
            // Clear entire cache
            this.configCache.clear();
        }
    }
    
    private generateCacheKey(branchId?: number, keys?: string[]): string {
        const branch = branchId || 'current';
        const keyString = keys ? keys.sort().join(',') : 'all';
        return `${branch}:${keyString}`;
    }
    
    getCacheStats(): any {
        return {
            size: this.configCache.size,
            entries: Array.from(this.configCache.keys()),
            timeout: this.cacheTimeout
        };
    }
}
```

### **5. Configuration Validation Service**
```typescript
export class ConfigValidationService {
    constructor(private configService: SYS_ConfigService) {}
    
    async validateConfigurations(branchId?: number): Promise<any> {
        try {
            const configs = await this.configService.getConfig(branchId);
            const validationResults = [];
            
            // Validate each configuration
            for (const key in configs) {
                const validation = this.validateConfigValue(key, configs[key]);
                validationResults.push(validation);
            }
            
            return {
                isValid: validationResults.every(v => v.isValid),
                results: validationResults,
                errors: validationResults.filter(v => !v.isValid)
            };
            
        } catch (error) {
            console.error('Configuration validation failed:', error);
            throw error;
        }
    }
    
    private validateConfigValue(key: string, value: any): any {
        const validators = {
            'SessionTimeout': (val) => this.validateSessionTimeout(val),
            'MaxFileUploadSize': (val) => this.validateFileSize(val),
            'DefaultLanguage': (val) => this.validateLanguage(val),
            'AppTheme': (val) => this.validateTheme(val),
            'EnableAdvancedSearch': (val) => this.validateBoolean(val)
        };
        
        const validator = validators[key];
        
        if (validator) {
            return validator(value);
        }
        
        // Default validation - just check if value is not null
        return {
            key: key,
            isValid: value !== null && value !== undefined,
            message: value === null ? 'Value is null' : 'Valid'
        };
    }
    
    private validateSessionTimeout(value: any): any {
        const timeout = parseInt(value);
        const isValid = !isNaN(timeout) && timeout >= 300000 && timeout <= 86400000; // 5 min to 24 hours
        
        return {
            key: 'SessionTimeout',
            isValid: isValid,
            message: isValid ? 'Valid' : 'Session timeout must be between 5 minutes and 24 hours'
        };
    }
    
    private validateFileSize(value: any): any {
        const size = parseInt(value);
        const isValid = !isNaN(size) && size >= 1048576 && size <= 104857600; // 1MB to 100MB
        
        return {
            key: 'MaxFileUploadSize',
            isValid: isValid,
            message: isValid ? 'Valid' : 'File size must be between 1MB and 100MB'
        };
    }
    
    private validateLanguage(value: any): any {
        const supportedLanguages = ['en', 'vi', 'zh', 'ja', 'ko'];
        const isValid = typeof value === 'string' && supportedLanguages.includes(value);
        
        return {
            key: 'DefaultLanguage',
            isValid: isValid,
            message: isValid ? 'Valid' : `Language must be one of: ${supportedLanguages.join(', ')}`
        };
    }
    
    private validateTheme(value: any): any {
        const supportedThemes = ['light', 'dark', 'auto'];
        const isValid = typeof value === 'string' && supportedThemes.includes(value);
        
        return {
            key: 'AppTheme',
            isValid: isValid,
            message: isValid ? 'Valid' : `Theme must be one of: ${supportedThemes.join(', ')}`
        };
    }
    
    private validateBoolean(value: any): any {
        const isValid = typeof value === 'boolean';
        
        return {
            key: 'Boolean Config',
            isValid: isValid,
            message: isValid ? 'Valid' : 'Value must be true or false'
        };
    }
}
```

---

## 🔧 **Advanced Features**

### **1. Configuration Hierarchy Management**
```typescript
export class ConfigHierarchyService {
    constructor(private configService: SYS_ConfigService) {}
    
    async getConfigurationHierarchy(branchId: number): Promise<any> {
        try {
            // Get branch hierarchy
            const branchHierarchy = await this.getBranchHierarchy(branchId);
            
            // Load configurations for each level
            const hierarchyConfigs = {};
            
            for (const branch of branchHierarchy) {
                const configs = await this.configService.getConfig(branch.Id);
                hierarchyConfigs[branch.Id] = {
                    branch: branch,
                    configs: configs,
                    level: branch.Level
                };
            }
            
            return this.buildConfigurationTree(hierarchyConfigs);
            
        } catch (error) {
            console.error('Failed to build configuration hierarchy:', error);
            throw error;
        }
    }
    
    private buildConfigurationTree(hierarchyConfigs: any): any {
        const tree = {};
        
        // Process from root to leaf
        const sortedLevels = Object.keys(hierarchyConfigs).sort((a, b) => 
            hierarchyConfigs[a].level - hierarchyConfigs[b].level
        );
        
        for (const branchId of sortedLevels) {
            const branchData = hierarchyConfigs[branchId];
            
            for (const configKey in branchData.configs) {
                if (!tree[configKey]) {
                    tree[configKey] = {
                        key: configKey,
                        hierarchy: []
                    };
                }
                
                tree[configKey].hierarchy.push({
                    branchId: branchId,
                    branchName: branchData.branch.Name,
                    level: branchData.level,
                    value: branchData.configs[configKey],
                    isInherited: this.isConfigInherited(branchData.configs[configKey])
                });
            }
        }
        
        return tree;
    }
}
```

### **2. Configuration Change Tracking**
```typescript
export class ConfigChangeTrackingService {
    private changeHistory: any[] = [];
    
    constructor(private configService: SYS_ConfigService) {}
    
    async trackConfigurationChanges(branchId: number): Promise<void> {
        // Load current configurations
        const currentConfigs = await this.configService.getConfig(branchId);
        
        // Compare with previous snapshot
        const previousSnapshot = this.getLastSnapshot(branchId);
        
        if (previousSnapshot) {
            const changes = this.detectChanges(previousSnapshot.configs, currentConfigs);
            
            if (changes.length > 0) {
                this.recordChanges(branchId, changes);
                this.notifyConfigurationChanges(branchId, changes);
            }
        }
        
        // Save current snapshot
        this.saveSnapshot(branchId, currentConfigs);
    }
    
    private detectChanges(oldConfigs: any, newConfigs: any): any[] {
        const changes = [];
        
        // Check for modified and deleted configurations
        for (const key in oldConfigs) {
            if (!newConfigs[key]) {
                changes.push({
                    type: 'deleted',
                    key: key,
                    oldValue: oldConfigs[key],
                    newValue: null
                });
            } else if (JSON.stringify(oldConfigs[key]) !== JSON.stringify(newConfigs[key])) {
                changes.push({
                    type: 'modified',
                    key: key,
                    oldValue: oldConfigs[key],
                    newValue: newConfigs[key]
                });
            }
        }
        
        // Check for new configurations
        for (const key in newConfigs) {
            if (!oldConfigs[key]) {
                changes.push({
                    type: 'added',
                    key: key,
                    oldValue: null,
                    newValue: newConfigs[key]
                });
            }
        }
        
        return changes;
    }
    
    private recordChanges(branchId: number, changes: any[]): void {
        const changeRecord = {
            branchId: branchId,
            timestamp: new Date(),
            changes: changes,
            userId: this.env.user?.Id
        };
        
        this.changeHistory.push(changeRecord);
        
        // Keep only last 100 changes
        if (this.changeHistory.length > 100) {
            this.changeHistory = this.changeHistory.slice(-100);
        }
    }
    
    getChangeHistory(branchId?: number): any[] {
        if (branchId) {
            return this.changeHistory.filter(record => record.branchId === branchId);
        }
        
        return this.changeHistory;
    }
}
```

---

## 📋 **Best Practices**

### **1. Error Handling**
```typescript
// ✅ Graceful error handling with fallbacks
async loadConfigSafely(keys: string[]) {
    try {
        return await this.configService.getConfig(null, keys);
    } catch (error) {
        console.error('Config loading failed:', error);
        
        // Return default configurations
        return this.getDefaultConfigurations(keys);
    }
}
```

### **2. Performance Optimization**
```typescript
// ✅ Batch configuration requests
async loadMultipleBranchConfigs(branchIds: number[]) {
    const promises = branchIds.map(id => 
        this.configService.getConfig(id)
    );
    
    return Promise.all(promises);
}
```

### **3. Configuration Validation**
```typescript
// ✅ Validate configurations before use
validateAndApplyConfig(configs: any) {
    const validatedConfigs = {};
    
    for (const key in configs) {
        if (this.isValidConfigValue(key, configs[key])) {
            validatedConfigs[key] = configs[key];
        } else {
            console.warn(`Invalid configuration value for ${key}:`, configs[key]);
            validatedConfigs[key] = this.getDefaultValue(key);
        }
    }
    
    return validatedConfigs;
}
```

---

## 🚨 **Security Considerations**

### **1. Configuration Access Control**
```typescript
// ✅ Check permissions before loading sensitive configs
async loadSensitiveConfigs(keys: string[]) {
    const hasPermission = await this.env.checkFormPermission('/system/config');
    
    if (!hasPermission) {
        throw new Error('Insufficient permissions to access system configurations');
    }
    
    return this.configService.getConfig(null, keys);
}
```

### **2. Configuration Sanitization**
```typescript
// ✅ Sanitize configuration values
sanitizeConfigValue(key: string, value: any): any {
    if (typeof value === 'string') {
        // Remove potentially dangerous content
        return value.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');
    }
    
    return value;
}
```

SYS_ConfigService cung cấp robust system configuration management với inheritance support, caching, validation, và change tracking cho enterprise applications.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
