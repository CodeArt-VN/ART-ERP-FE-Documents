# PageDataManagementService Documentation

## 📋 **Tổng quan**

`PageDataManagementService` là service để handle data management operations trong các pages của ứng dụng ERP. Service này provide functionality cho việc merge data sets, manage page items, và handle data consistency across different page operations.

**Location**: `src/app/services/page/data-management.service.ts`  
**Type**: Class (not Injectable - intended for instantiation per page)

---

## 🔧 **Key Features**

### **Data Management**
- **Item Merging**: Merge data sets without duplicates based on ID
- **Page Data Handling**: Manage items array for page operations
- **Provider Integration**: Work with page providers for data operations
- **Configuration Support**: Support page-specific configurations

### **Dependencies**
- **EnvService**: Environment and utility services
- **PageProvider**: Data provider for specific page/entity
- **PageConfig**: Configuration object for page behavior

---

## 🚀 **Constructor & Properties**

### **Constructor Parameters**
```typescript
constructor(
    env: EnvService,           // Environment service
    pageProvider: any,         // Data provider for the page
    pageConfig: any,          // Page configuration
    items: any[] = []         // Initial items array
)
```

### **Properties**
```typescript
private env: EnvService;      // Environment service instance
private pageProvider: any;    // Page data provider
private pageConfig: any;      // Page configuration
items: any[];                // Array of page items
```

---

## 🎨 **Core Methods**

### **Data Merging**
```typescript
mergeItems(currentSet: any[], newSet: any[]): any[]
```

**Purpose**: Merge two data sets ensuring no duplicate IDs  
**Parameters**:
- `currentSet`: Existing array of items
- `newSet`: New array of items to merge

**Returns**: Merged array with unique items based on ID

---

## 🎨 **Usage Examples**

### **1. Basic Page Data Management**
```typescript
export class ProductListPage extends PageBase {
    dataManager: PageDataManagementService;
    
    constructor(
        private env: EnvService,
        private productProvider: ProductProvider
    ) {
        super();
        
        // Initialize data management service
        this.dataManager = new PageDataManagementService(
            this.env,
            this.productProvider,
            this.pageConfig,
            []
        );
    }
    
    async loadData() {
        try {
            // Load initial data
            const result = await this.productProvider.read(this.query);
            
            // Set items in data manager
            this.dataManager.items = result.data || [];
            
            // Update local items reference
            this.items = this.dataManager.items;
            
        } catch (error) {
            console.error('Failed to load data:', error);
            this.env.showErrorMessage('Failed to load products');
        }
    }
    
    async loadMoreData() {
        try {
            // Load next page
            this.query.skip = this.dataManager.items.length;
            const result = await this.productProvider.read(this.query);
            
            // Merge new data with existing
            this.dataManager.items = this.dataManager.mergeItems(
                this.dataManager.items,
                result.data || []
            );
            
            // Update local reference
            this.items = this.dataManager.items;
            
        } catch (error) {
            console.error('Failed to load more data:', error);
            this.env.showErrorMessage('Failed to load more products');
        }
    }
    
    async refreshData() {
        try {
            // Reset query
            this.query.skip = 0;
            
            // Load fresh data
            const result = await this.productProvider.read(this.query);
            
            // Replace all items
            this.dataManager.items = result.data || [];
            this.items = this.dataManager.items;
            
        } catch (error) {
            console.error('Failed to refresh data:', error);
            this.env.showErrorMessage('Failed to refresh products');
        }
    }
}
```

### **2. Enhanced Data Management Service**
```typescript
export class EnhancedPageDataManagementService extends PageDataManagementService {
    private cache = new Map<string, any>();
    private changeTracker = new Map<string, any>();
    
    constructor(
        env: EnvService,
        pageProvider: any,
        pageConfig: any,
        items: any[] = []
    ) {
        super(env, pageProvider, pageConfig, items);
    }
    
    // Add item with change tracking
    addItem(item: any): void {
        // Generate ID if not exists
        if (!item.Id) {
            item.Id = this.generateTempId();
        }
        
        // Add to items
        this.items.push(item);
        
        // Track as new item
        this.changeTracker.set(item.Id, {
            action: 'create',
            item: item,
            timestamp: new Date()
        });
    }
    
    // Update item with change tracking
    updateItem(itemId: string, updates: any): boolean {
        const itemIndex = this.items.findIndex(item => item.Id === itemId);
        
        if (itemIndex === -1) {
            return false;
        }
        
        // Store original for rollback
        const original = { ...this.items[itemIndex] };
        
        // Apply updates
        Object.assign(this.items[itemIndex], updates);
        
        // Track change
        this.changeTracker.set(itemId, {
            action: 'update',
            original: original,
            updated: this.items[itemIndex],
            timestamp: new Date()
        });
        
        return true;
    }
    
    // Remove item with change tracking
    removeItem(itemId: string): boolean {
        const itemIndex = this.items.findIndex(item => item.Id === itemId);
        
        if (itemIndex === -1) {
            return false;
        }
        
        const removedItem = this.items[itemIndex];
        
        // Remove from items
        this.items.splice(itemIndex, 1);
        
        // Track removal
        this.changeTracker.set(itemId, {
            action: 'delete',
            item: removedItem,
            timestamp: new Date()
        });
        
        return true;
    }
    
    // Get pending changes
    getPendingChanges(): any[] {
        return Array.from(this.changeTracker.values());
    }
    
    // Clear change tracking
    clearChangeTracking(): void {
        this.changeTracker.clear();
    }
    
    // Rollback changes
    rollbackChanges(): void {
        const changes = Array.from(this.changeTracker.values());
        
        // Process in reverse order
        changes.reverse().forEach(change => {
            switch (change.action) {
                case 'create':
                    this.items = this.items.filter(item => item.Id !== change.item.Id);
                    break;
                    
                case 'update':
                    const updateIndex = this.items.findIndex(item => item.Id === change.original.Id);
                    if (updateIndex !== -1) {
                        this.items[updateIndex] = change.original;
                    }
                    break;
                    
                case 'delete':
                    this.items.push(change.item);
                    break;
            }
        });
        
        this.clearChangeTracking();
    }
    
    // Advanced merge with conflict resolution
    mergeItemsAdvanced(currentSet: any[], newSet: any[], conflictResolver?: (current: any, new: any) => any): any[] {
        const mergedMap = new Map<string, any>();
        const conflicts = [];
        
        // Add current items
        currentSet.forEach(item => {
            mergedMap.set(item.Id, item);
        });
        
        // Process new items
        newSet.forEach(newItem => {
            const existingItem = mergedMap.get(newItem.Id);
            
            if (existingItem) {
                // Conflict detected
                if (conflictResolver) {
                    const resolved = conflictResolver(existingItem, newItem);
                    mergedMap.set(newItem.Id, resolved);
                } else {
                    // Default: use newer timestamp or new item
                    const useNew = !existingItem.ModifiedDate || 
                                  !newItem.ModifiedDate ||
                                  new Date(newItem.ModifiedDate) > new Date(existingItem.ModifiedDate);
                    
                    if (useNew) {
                        mergedMap.set(newItem.Id, newItem);
                    }
                    
                    conflicts.push({
                        id: newItem.Id,
                        existing: existingItem,
                        new: newItem,
                        resolved: useNew ? newItem : existingItem
                    });
                }
            } else {
                mergedMap.set(newItem.Id, newItem);
            }
        });
        
        // Log conflicts if any
        if (conflicts.length > 0) {
            console.warn('Data merge conflicts detected:', conflicts);
        }
        
        return Array.from(mergedMap.values());
    }
    
    // Cache management
    cacheItems(key: string): void {
        this.cache.set(key, [...this.items]);
    }
    
    restoreFromCache(key: string): boolean {
        const cached = this.cache.get(key);
        
        if (cached) {
            this.items = [...cached];
            return true;
        }
        
        return false;
    }
    
    clearCache(): void {
        this.cache.clear();
    }
    
    // Utility methods
    private generateTempId(): string {
        return `temp_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    }
    
    // Data validation
    validateItems(): ValidationResult {
        const errors = [];
        const warnings = [];
        
        this.items.forEach((item, index) => {
            if (!item.Id) {
                errors.push(`Item at index ${index} missing ID`);
            }
            
            if (this.pageConfig.requiredFields) {
                this.pageConfig.requiredFields.forEach(field => {
                    if (!item[field]) {
                        errors.push(`Item ${item.Id || index} missing required field: ${field}`);
                    }
                });
            }
        });
        
        // Check for duplicates
        const ids = this.items.map(item => item.Id);
        const duplicates = ids.filter((id, index) => ids.indexOf(id) !== index);
        
        if (duplicates.length > 0) {
            errors.push(`Duplicate IDs found: ${duplicates.join(', ')}`);
        }
        
        return {
            isValid: errors.length === 0,
            errors: errors,
            warnings: warnings
        };
    }
}

interface ValidationResult {
    isValid: boolean;
    errors: string[];
    warnings: string[];
}
```

### **3. Real-time Data Synchronization**
```typescript
export class RealTimeDataManagementService extends EnhancedPageDataManagementService {
    private websocket: WebSocket;
    private syncEnabled: boolean = false;
    
    constructor(
        env: EnvService,
        pageProvider: any,
        pageConfig: any,
        items: any[] = []
    ) {
        super(env, pageProvider, pageConfig, items);
        this.setupRealTimeSync();
    }
    
    private setupRealTimeSync(): void {
        if (this.pageConfig.enableRealTimeSync) {
            this.initializeWebSocket();
        }
        
        // Listen for environment events
        this.env.getEvents().subscribe(event => {
            if (event.Code === 'data:updated' && event.Value.entityType === this.pageConfig.entityType) {
                this.handleDataUpdate(event.Value);
            }
        });
    }
    
    private initializeWebSocket(): void {
        try {
            this.websocket = new WebSocket(this.env.websocketUrl);
            
            this.websocket.onopen = () => {
                this.syncEnabled = true;
                console.log('Real-time sync enabled');
                
                // Subscribe to entity updates
                this.websocket.send(JSON.stringify({
                    action: 'subscribe',
                    entityType: this.pageConfig.entityType,
                    branchId: this.env.selectedBranch
                }));
            };
            
            this.websocket.onmessage = (event) => {
                const data = JSON.parse(event.data);
                this.handleRealTimeUpdate(data);
            };
            
            this.websocket.onclose = () => {
                this.syncEnabled = false;
                console.log('Real-time sync disconnected');
                
                // Attempt reconnection
                setTimeout(() => this.initializeWebSocket(), 5000);
            };
            
        } catch (error) {
            console.error('Failed to initialize WebSocket:', error);
        }
    }
    
    private handleRealTimeUpdate(data: any): void {
        switch (data.action) {
            case 'create':
                this.handleRemoteCreate(data.item);
                break;
            case 'update':
                this.handleRemoteUpdate(data.item);
                break;
            case 'delete':
                this.handleRemoteDelete(data.itemId);
                break;
        }
    }
    
    private handleRemoteCreate(item: any): void {
        // Check if item already exists
        const exists = this.items.find(existing => existing.Id === item.Id);
        
        if (!exists) {
            this.items.push(item);
            
            // Notify UI
            this.env.publishEvent({
                Code: 'data:remote:create',
                Value: { item: item, entityType: this.pageConfig.entityType }
            });
        }
    }
    
    private handleRemoteUpdate(item: any): void {
        const index = this.items.findIndex(existing => existing.Id === item.Id);
        
        if (index !== -1) {
            // Check for conflicts with local changes
            const hasLocalChanges = this.changeTracker.has(item.Id);
            
            if (hasLocalChanges) {
                this.handleUpdateConflict(this.items[index], item);
            } else {
                this.items[index] = item;
                
                // Notify UI
                this.env.publishEvent({
                    Code: 'data:remote:update',
                    Value: { item: item, entityType: this.pageConfig.entityType }
                });
            }
        }
    }
    
    private handleRemoteDelete(itemId: string): void {
        const index = this.items.findIndex(item => item.Id === itemId);
        
        if (index !== -1) {
            // Check for local changes
            const hasLocalChanges = this.changeTracker.has(itemId);
            
            if (hasLocalChanges) {
                this.handleDeleteConflict(this.items[index]);
            } else {
                this.items.splice(index, 1);
                
                // Notify UI
                this.env.publishEvent({
                    Code: 'data:remote:delete',
                    Value: { itemId: itemId, entityType: this.pageConfig.entityType }
                });
            }
        }
    }
    
    private async handleUpdateConflict(localItem: any, remoteItem: any): Promise<void> {
        // Show conflict resolution dialog
        const resolution = await this.env.showPrompt(
            'Data conflict detected. Choose resolution:',
            'Conflict Resolution',
            'Resolve',
            'Cancel',
            [
                {
                    name: 'resolution',
                    type: 'radio',
                    options: [
                        { text: 'Keep local changes', value: 'local' },
                        { text: 'Accept remote changes', value: 'remote' },
                        { text: 'Merge changes', value: 'merge' }
                    ]
                }
            ]
        );
        
        if (resolution?.resolution) {
            switch (resolution.resolution) {
                case 'local':
                    // Keep local, ignore remote
                    break;
                case 'remote':
                    // Accept remote, discard local
                    const index = this.items.findIndex(item => item.Id === localItem.Id);
                    if (index !== -1) {
                        this.items[index] = remoteItem;
                        this.changeTracker.delete(localItem.Id);
                    }
                    break;
                case 'merge':
                    // Attempt to merge
                    const merged = this.mergeItems([localItem], [remoteItem])[0];
                    const index2 = this.items.findIndex(item => item.Id === localItem.Id);
                    if (index2 !== -1) {
                        this.items[index2] = merged;
                    }
                    break;
            }
        }
    }
    
    ngOnDestroy(): void {
        if (this.websocket) {
            this.websocket.close();
        }
    }
}
```

---

## 📋 **Best Practices**

### **1. Data Consistency**
```typescript
// ✅ Always validate data before merging
validateBeforeMerge(items: any[]): boolean {
    return items.every(item => 
        item.Id && 
        typeof item.Id === 'string' && 
        item.Id.trim() !== ''
    );
}

// ✅ Use immutable operations when possible
mergeItemsImmutable(currentSet: any[], newSet: any[]): any[] {
    const result = [...currentSet];
    
    newSet.forEach(newItem => {
        const existingIndex = result.findIndex(item => item.Id === newItem.Id);
        
        if (existingIndex !== -1) {
            result[existingIndex] = { ...result[existingIndex], ...newItem };
        } else {
            result.push({ ...newItem });
        }
    });
    
    return result;
}
```

### **2. Performance Optimization**
```typescript
// ✅ Use Map for O(1) lookups in large datasets
mergeItemsOptimized(currentSet: any[], newSet: any[]): any[] {
    if (currentSet.length > 1000 || newSet.length > 1000) {
        return this.mergeItemsWithMap(currentSet, newSet);
    }
    
    return this.mergeItems(currentSet, newSet);
}

private mergeItemsWithMap(currentSet: any[], newSet: any[]): any[] {
    const mergedMap = new Map();
    
    // Use Map for efficient lookups
    currentSet.forEach(item => mergedMap.set(item.Id, item));
    newSet.forEach(item => mergedMap.set(item.Id, item));
    
    return Array.from(mergedMap.values());
}
```

### **3. Error Handling**
```typescript
// ✅ Handle merge errors gracefully
safeMergeItems(currentSet: any[], newSet: any[]): any[] {
    try {
        // Validate inputs
        if (!Array.isArray(currentSet) || !Array.isArray(newSet)) {
            throw new Error('Invalid input: both parameters must be arrays');
        }
        
        return this.mergeItems(currentSet, newSet);
        
    } catch (error) {
        console.error('Merge operation failed:', error);
        
        // Return current set as fallback
        return currentSet;
    }
}
```

---

## 🚨 **Security Considerations**

### **1. Data Validation**
```typescript
// ✅ Validate item structure
private validateItemStructure(item: any): boolean {
    if (!item || typeof item !== 'object') {
        return false;
    }
    
    // Check required fields
    if (!item.Id || typeof item.Id !== 'string') {
        return false;
    }
    
    // Sanitize string fields
    Object.keys(item).forEach(key => {
        if (typeof item[key] === 'string') {
            item[key] = this.sanitizeString(item[key]);
        }
    });
    
    return true;
}
```

### **2. Access Control**
```typescript
// ✅ Check permissions before data operations
async validateDataAccess(operation: string): Promise<boolean> {
    const hasPermission = await this.env.checkFormPermission(
        `/${this.pageConfig.entityType}/${operation}`
    );
    
    if (!hasPermission) {
        throw new Error(`Insufficient permissions for ${operation} operation`);
    }
    
    return true;
}
```

PageDataManagementService cung cấp robust foundation cho data management operations trong pages với merge capabilities, change tracking, và real-time synchronization support.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
