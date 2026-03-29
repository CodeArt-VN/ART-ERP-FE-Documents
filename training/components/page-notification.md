# PageNotification Component Documentation

## 📋 **Tổng quan**

`PageNotificationComponent` là component quản lý notifications cho từng page cụ thể, extend từ `PageBase` và tích hợp với notification services để hiển thị, đọc và xóa notifications.

**Selector**: `app-page-notification`  
**Location**: `src/app/components/page-notification/page-notification.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() pageName: string;               // Page identifier for filtering notifications
```

---

## 🚀 **Basic Usage**

### **Page-Specific Notifications**
```typescript
// Component
export class OrderPageNotifications extends PageBase {
    pageName = 'order-management';

    async ngOnInit() {
        await this.loadPageNotifications();
    }

    async loadPageNotifications() {
        // Notifications will be filtered by pageName automatically
        await this.loadData();
    }
}
```

```html
<!-- Template -->
<div class="page-notifications">
    <app-page-notification
        [pageName]="pageName">
    </app-page-notification>
</div>
```

### **Integration with Page Layout**
```html
<ion-header>
    <ion-toolbar>
        <ion-title>Orders</ion-title>
        <ion-buttons slot="end">
            <ion-button (click)="showNotifications()">
                <ion-icon name="notifications"></ion-icon>
                <ion-badge *ngIf="unreadCount > 0">{{ unreadCount }}</ion-badge>
            </ion-button>
        </ion-buttons>
    </ion-toolbar>
</ion-header>

<ion-content>
    <!-- Page notifications panel -->
    <app-page-notification
        *ngIf="showNotificationPanel"
        [pageName]="'order-management'">
    </app-page-notification>

    <!-- Main content -->
    <div class="page-content">
        <!-- Your page content -->
    </div>
</ion-content>
```

---

## 🔔 **Notification Actions**

### **Read Notification**
```typescript
// The component automatically handles reading notifications
// When user clicks on a notification, it calls readNoti(notification)
// This marks the notification as read and publishes an event

onNotificationClick(notification: any) {
    // Automatically handled by the component
    // - Updates ReadDate
    // - Publishes notification event
    // - Can navigate to related page/item
}
```

### **Delete Notification**
```typescript
// The component handles deletion with confirmation
// When user clicks delete, it shows a confirmation prompt
// Then removes the notification from the list

onDeleteNotification(notification: any, event: Event) {
    // Automatically handled by the component
    // - Shows confirmation prompt
    // - Deletes notification via API
    // - Updates local list
    // - Publishes notification event
}
```

---

## 🎨 **Styling & Layout**

### **Custom Notification Styling**
```scss
// Component SCSS
app-page-notification {
    .notification-container {
        max-height: 400px;
        overflow-y: auto;
        
        .notification-item {
            display: flex;
            align-items: flex-start;
            padding: 12px 16px;
            border-bottom: 1px solid var(--ion-color-light);
            cursor: pointer;
            transition: background-color 0.2s ease;
            
            &:hover {
                background-color: var(--ion-color-light-tint);
            }
            
            &.unread {
                background-color: rgba(var(--ion-color-primary-rgb), 0.05);
                border-left: 3px solid var(--ion-color-primary);
            }
            
            .notification-content {
                flex: 1;
                margin-right: 12px;
                
                .notification-title {
                    font-weight: 500;
                    color: var(--ion-color-dark);
                    margin-bottom: 4px;
                }
                
                .notification-message {
                    font-size: 0.875rem;
                    color: var(--ion-color-medium);
                    line-height: 1.4;
                }
                
                .notification-time {
                    font-size: 0.75rem;
                    color: var(--ion-color-medium);
                    margin-top: 4px;
                }
            }
            
            .notification-actions {
                display: flex;
                gap: 8px;
                
                ion-button {
                    --padding-start: 8px;
                    --padding-end: 8px;
                    height: 32px;
                }
            }
        }
    }
    
    .empty-notifications {
        text-align: center;
        padding: 40px 20px;
        color: var(--ion-color-medium);
        
        ion-icon {
            font-size: 3rem;
            margin-bottom: 16px;
        }
    }
}
```

---

## 🔧 **Integration Patterns**

### **With Page Header**
```html
<ion-header>
    <ion-toolbar>
        <ion-title>{{ pageTitle }}</ion-title>
        
        <!-- Notification bell with badge -->
        <ion-buttons slot="end">
            <ion-button 
                fill="clear"
                (click)="toggleNotifications()">
                <ion-icon name="notifications"></ion-icon>
                <ion-badge 
                    *ngIf="unreadNotificationCount > 0"
                    color="danger">
                    {{ unreadNotificationCount }}
                </ion-badge>
            </ion-button>
        </ion-buttons>
    </ion-toolbar>
    
    <!-- Notification panel -->
    <div 
        *ngIf="showNotifications"
        class="notification-panel">
        <app-page-notification
            [pageName]="currentPageName">
        </app-page-notification>
    </div>
</ion-header>
```

### **With Sidebar/Drawer**
```html
<ion-menu side="end" contentId="main-content">
    <ion-header>
        <ion-toolbar>
            <ion-title>Notifications</ion-title>
        </ion-toolbar>
    </ion-header>
    
    <ion-content>
        <app-page-notification
            [pageName]="currentPageName">
        </app-page-notification>
    </ion-content>
</ion-menu>
```

---

## 🔧 **Best Practices**

### **1. Page Name Convention**
```typescript
// ✅ Đúng - Consistent page naming
const pageNames = {
    ORDER_MANAGEMENT: 'order-management',
    USER_MANAGEMENT: 'user-management', 
    PRODUCT_CATALOG: 'product-catalog',
    INVENTORY_CONTROL: 'inventory-control'
};
```

### **2. Event Handling**
```typescript
// ✅ Đúng - Listen to notification events
ngOnInit() {
    this.env.getEvents().subscribe(event => {
        if (event.Code === 'EVENT_TYPE.APP.NOTIFICATION') {
            this.updateNotificationCount();
        }
    });
}
```

### **3. Performance**
```typescript
// ✅ Đúng - Efficient notification loading
async loadNotifications() {
    try {
        // Only load notifications for current page
        this.query.Form = this.pageName;
        this.query.Take = 20; // Limit initial load
        
        await this.loadData();
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

---

## 🚨 **Common Use Cases**

### **1. Order Management Notifications**
```typescript
export class OrderManagementPage {
    pageName = 'order-management';
    
    // Notifications for:
    // - New orders received
    // - Order status changes
    // - Payment confirmations
    // - Shipping updates
}
```

### **2. Inventory Notifications**
```typescript
export class InventoryPage {
    pageName = 'inventory-control';
    
    // Notifications for:
    // - Low stock alerts
    // - Reorder points reached
    // - Stock movements
    // - Expiry warnings
}
```

### **3. User Management Notifications**
```typescript
export class UserManagementPage {
    pageName = 'user-management';
    
    // Notifications for:
    // - New user registrations
    // - Permission changes
    // - Account activations
    // - Security alerts
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
