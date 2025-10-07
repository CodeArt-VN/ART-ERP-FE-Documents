# EnvService Documentation

## 📋 **Tổng quan**

`EnvService` là central service để manage application environment, user context, messages, storage, permissions, và system events. Đây là service quan trọng nhất được sử dụng trong hầu hết các components.

**Location**: `src/app/services/core/env.service.ts`

---

## 🔧 **Key Features**

### **Message & Notification System**
```typescript
showMessage(message: string, color: string): void
showErrorMessage(error: any): void
showLoading(message: string, promise: Promise<any>): Promise<any>
actionConfirm(action: string, count: number, itemName: string): Promise<boolean>
```

### **Storage Management**
```typescript
getStorage(key: string): Promise<any>
setStorage(key: string, value: any): Promise<void>
setCookie(name: string, value: string, days: number): void
getCookie(name: string): string
```

### **Permission System**
```typescript
checkFormPermission(functionCode: string): Promise<boolean>
```

### **Branch & Context Management**
```typescript
loadBranch(): Promise<void>
changeBranch(): void
get selectedBranch(): any
```

### **Event System**
```typescript
publishEvent(data: any): void
getEvents(): Observable<any>
```

---

## 🚀 **Basic Usage**

### **Messages & Notifications**
```typescript
export class SamplePage extends PageBase {
    async saveData() {
        try {
            await this.env.showLoading('Saving...', this.dataService.save(this.data));
            this.env.showMessage('Data saved successfully', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    async deleteItems() {
        const confirmed = await this.env.actionConfirm('delete', this.selectedItems.length, 'items');
        if (confirmed) {
            await this.dataService.delete(this.selectedItems);
            this.env.showMessage('Items deleted', 'success');
        }
    }
}
```

### **Storage Operations**
```typescript
async saveUserPreferences() {
    await this.env.setStorage('userPrefs', {
        theme: 'dark',
        language: 'vi',
        notifications: true
    });
}

async loadUserPreferences() {
    const prefs = await this.env.getStorage('userPrefs');
    if (prefs) {
        this.applyPreferences(prefs);
    }
}
```

### **Permission Checks**
```typescript
async ngOnInit() {
    this.canEdit = await this.env.checkFormPermission('/users/edit');
    this.canDelete = await this.env.checkFormPermission('/users/delete');
    this.canApprove = await this.env.checkFormPermission('/orders/approve');
}
```

---

## 🎯 **Best Practices**

### **1. Always Use EnvService for Messages**
```typescript
// ✅ Đúng - Use EnvService methods
this.env.showMessage('Success message', 'success');
this.env.showErrorMessage(error);

// ❌ Sai - Don't use other notification methods
this.toastController.create({...}); // Don't do this
```

### **2. Check Permissions Before Actions**
```typescript
// ✅ Đúng - Check permissions first
async deleteItem() {
    const canDelete = await this.env.checkFormPermission('/items/delete');
    if (canDelete) {
        // Proceed with delete
    } else {
        this.env.showMessage('No permission to delete', 'warning');
    }
}
```

### **3. Use Event System for Communication**
```typescript
// ✅ Đúng - Publish events for cross-component communication
this.env.publishEvent({
    Code: 'DATA_UPDATED',
    Value: { entityType: 'users', action: 'create' }
});

// Listen to events
this.env.getEvents().subscribe(event => {
    if (event.Code === 'DATA_UPDATED') {
        this.refreshData();
    }
});
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
