# Notifications Component Documentation

## 📋 **Tổng quan**

`NotificationsComponent` là component quản lý và hiển thị notifications/alerts trong ứng dụng. Component này tích hợp với `EnvService` để hiển thị messages, alerts và real-time notifications.

**Selector**: `app-notifications`  
**Location**: `src/app/components/notifications/notifications.component.ts`

---

## 🚀 **Basic Usage**

### **Simple Notifications Display**
```typescript
// Component
export class NotificationsPage extends PageBase {
    notifications: any[] = [];
    unreadCount = 0;

    async ngOnInit() {
        await this.loadNotifications();
        this.subscribeToRealTimeNotifications();
    }

    async loadNotifications() {
        try {
            const result = await this.notificationService.read({
                userId: this.env.user.Id,
                take: 50,
                orderBy: 'CreatedDate desc'
            });
            this.notifications = result.data;
            this.updateUnreadCount();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    updateUnreadCount() {
        this.unreadCount = this.notifications.filter(n => !n.IsRead).length;
    }

    async markAsRead(notification: any) {
        try {
            await this.notificationService.markAsRead(notification.Id);
            notification.IsRead = true;
            this.updateUnreadCount();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async markAllAsRead() {
        try {
            await this.notificationService.markAllAsRead(this.env.user.Id);
            this.notifications.forEach(n => n.IsRead = true);
            this.updateUnreadCount();
            this.env.showMessage('ALL_NOTIFICATIONS_READ', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

```html
<!-- Template -->
<app-notifications
    [notifications]="notifications"
    [unreadCount]="unreadCount"
    (markAsRead)="markAsRead($event)"
    (markAllAsRead)="markAllAsRead()"
    (refresh)="loadNotifications()">
</app-notifications>
```

---

## 🔔 **Notification Types**

### **System Notifications**
```typescript
export class SystemNotifications {
    // Success notifications
    showSuccessNotification(message: string, data?: any) {
        this.env.showMessage(message, 'success');
        this.addToNotificationHistory({
            type: 'success',
            message,
            data,
            timestamp: new Date()
        });
    }

    // Error notifications
    showErrorNotification(error: any) {
        this.env.showErrorMessage(error);
        this.addToNotificationHistory({
            type: 'error',
            message: error.message || 'UNKNOWN_ERROR',
            error,
            timestamp: new Date()
        });
    }

    // Warning notifications
    showWarningNotification(message: string, data?: any) {
        this.env.showMessage(message, 'warning');
        this.addToNotificationHistory({
            type: 'warning',
            message,
            data,
            timestamp: new Date()
        });
    }

    // Info notifications
    showInfoNotification(message: string, data?: any) {
        this.env.showMessage(message, 'tertiary');
        this.addToNotificationHistory({
            type: 'info',
            message,
            data,
            timestamp: new Date()
        });
    }
}
```

### **Business Notifications**
```typescript
export class BusinessNotifications {
    // Order notifications
    notifyOrderStatusChange(order: any, oldStatus: string, newStatus: string) {
        const notification = {
            type: 'order_status',
            title: 'ORDER_STATUS_CHANGED',
            message: `Order ${order.Code} changed from ${oldStatus} to ${newStatus}`,
            data: { orderId: order.Id, oldStatus, newStatus },
            priority: 'normal',
            category: 'orders'
        };
        
        this.sendNotification(notification);
    }

    // Approval notifications
    notifyApprovalRequest(item: any, approver: any) {
        const notification = {
            type: 'approval_request',
            title: 'APPROVAL_REQUEST',
            message: `${item.Type} ${item.Code} requires your approval`,
            data: { itemId: item.Id, itemType: item.Type },
            priority: 'high',
            category: 'approvals',
            recipientId: approver.Id
        };
        
        this.sendNotification(notification);
    }

    // Inventory notifications
    notifyLowStock(product: any, currentStock: number, minStock: number) {
        const notification = {
            type: 'low_stock',
            title: 'LOW_STOCK_ALERT',
            message: `${product.Name} is running low (${currentStock}/${minStock})`,
            data: { productId: product.Id, currentStock, minStock },
            priority: 'high',
            category: 'inventory'
        };
        
        this.sendNotification(notification);
    }

    // Payment notifications
    notifyPaymentReceived(payment: any) {
        const notification = {
            type: 'payment_received',
            title: 'PAYMENT_RECEIVED',
            message: `Payment of ${payment.Amount} received for invoice ${payment.InvoiceCode}`,
            data: { paymentId: payment.Id, invoiceId: payment.InvoiceId },
            priority: 'normal',
            category: 'payments'
        };
        
        this.sendNotification(notification);
    }
}
```

---

## 🔄 **Real-time Notifications**

### **SignalR Integration**
```typescript
export class RealTimeNotifications {
    private hubConnection: any;

    async initializeSignalR() {
        try {
            this.hubConnection = new signalR.HubConnectionBuilder()
                .withUrl('/notificationHub', {
                    accessTokenFactory: () => this.env.getAuthToken()
                })
                .build();

            // Listen for notifications
            this.hubConnection.on('ReceiveNotification', (notification: any) => {
                this.handleRealTimeNotification(notification);
            });

            // Listen for broadcast messages
            this.hubConnection.on('BroadcastMessage', (message: any) => {
                this.handleBroadcastMessage(message);
            });

            await this.hubConnection.start();
            console.log('SignalR connected');
        } catch (error) {
            console.error('SignalR connection error:', error);
        }
    }

    handleRealTimeNotification(notification: any) {
        // Add to notifications list
        this.notifications.unshift(notification);
        
        // Show toast notification
        this.showToastNotification(notification);
        
        // Play sound if enabled
        if (this.notificationSettings.soundEnabled) {
            this.playNotificationSound();
        }
        
        // Show badge if app is in background
        if (document.hidden) {
            this.updateBadgeCount();
        }
    }

    handleBroadcastMessage(message: any) {
        // Handle system-wide messages
        if (message.type === 'maintenance') {
            this.env.showAlert(message.content, 'SYSTEM_MAINTENANCE', 'MAINTENANCE_NOTICE');
        } else if (message.type === 'announcement') {
            this.env.showMessage(message.content, 'tertiary');
        }
    }

    async sendNotificationToUser(userId: number, notification: any) {
        if (this.hubConnection) {
            try {
                await this.hubConnection.invoke('SendNotificationToUser', userId, notification);
            } catch (error) {
                console.error('Send notification error:', error);
            }
        }
    }

    async sendBroadcastMessage(message: any) {
        if (this.hubConnection) {
            try {
                await this.hubConnection.invoke('BroadcastMessage', message);
            } catch (error) {
                console.error('Broadcast message error:', error);
            }
        }
    }
}
```

### **Push Notifications**
```typescript
export class PushNotifications {
    async initializePushNotifications() {
        if ('serviceWorker' in navigator && 'PushManager' in window) {
            try {
                const registration = await navigator.serviceWorker.register('/sw.js');
                const subscription = await this.subscribeToPush(registration);
                await this.sendSubscriptionToServer(subscription);
            } catch (error) {
                console.error('Push notification setup error:', error);
            }
        }
    }

    async subscribeToPush(registration: ServiceWorkerRegistration) {
        const vapidPublicKey = 'YOUR_VAPID_PUBLIC_KEY';
        
        return await registration.pushManager.subscribe({
            userVisibleOnly: true,
            applicationServerKey: this.urlBase64ToUint8Array(vapidPublicKey)
        });
    }

    async sendSubscriptionToServer(subscription: PushSubscription) {
        try {
            await this.notificationService.savePushSubscription({
                userId: this.env.user.Id,
                subscription: JSON.stringify(subscription)
            });
        } catch (error) {
            console.error('Save subscription error:', error);
        }
    }

    private urlBase64ToUint8Array(base64String: string): Uint8Array {
        const padding = '='.repeat((4 - base64String.length % 4) % 4);
        const base64 = (base64String + padding)
            .replace(/-/g, '+')
            .replace(/_/g, '/');

        const rawData = window.atob(base64);
        const outputArray = new Uint8Array(rawData.length);

        for (let i = 0; i < rawData.length; ++i) {
            outputArray[i] = rawData.charCodeAt(i);
        }
        return outputArray;
    }
}
```

---

## 🎨 **Notification UI Components**

### **Toast Notifications**
```typescript
export class ToastNotifications {
    async showToastNotification(notification: any) {
        const toast = await this.toastController.create({
            header: notification.title,
            message: notification.message,
            duration: this.getToastDuration(notification.priority),
            color: this.getToastColor(notification.type),
            position: 'top',
            buttons: [
                {
                    text: 'View',
                    handler: () => this.viewNotification(notification)
                },
                {
                    text: 'Dismiss',
                    role: 'cancel'
                }
            ]
        });

        await toast.present();
    }

    private getToastDuration(priority: string): number {
        switch (priority) {
            case 'high': return 8000;
            case 'normal': return 5000;
            case 'low': return 3000;
            default: return 5000;
        }
    }

    private getToastColor(type: string): string {
        switch (type) {
            case 'success': return 'success';
            case 'error': return 'danger';
            case 'warning': return 'warning';
            case 'info': return 'tertiary';
            default: return 'primary';
        }
    }
}
```

### **Notification Center**
```html
<div class="notification-center">
    <!-- Header -->
    <div class="notification-header">
        <h3>Notifications</h3>
        <div class="notification-actions">
            <ion-button 
                fill="clear" 
                size="small"
                (click)="markAllAsRead()"
                [disabled]="unreadCount === 0">
                Mark All Read
            </ion-button>
            <ion-button 
                fill="clear" 
                size="small"
                (click)="clearAll()">
                Clear All
            </ion-button>
        </div>
    </div>

    <!-- Filters -->
    <div class="notification-filters">
        <ion-segment [(ngModel)]="selectedFilter" (ionChange)="filterNotifications($event.detail.value)">
            <ion-segment-button value="all">
                All ({{ notifications.length }})
            </ion-segment-button>
            <ion-segment-button value="unread">
                Unread ({{ unreadCount }})
            </ion-segment-button>
            <ion-segment-button value="important">
                Important
            </ion-segment-button>
        </ion-segment>
    </div>

    <!-- Notification List -->
    <div class="notification-list">
        <div 
            *ngFor="let notification of filteredNotifications" 
            class="notification-item"
            [class.unread]="!notification.IsRead"
            [class.important]="notification.priority === 'high'"
            (click)="viewNotification(notification)">
            
            <!-- Icon -->
            <div class="notification-icon">
                <ion-icon [name]="getNotificationIcon(notification.type)"></ion-icon>
            </div>
            
            <!-- Content -->
            <div class="notification-content">
                <div class="notification-title">{{ notification.title | translate }}</div>
                <div class="notification-message">{{ notification.message }}</div>
                <div class="notification-time">{{ notification.createdDate | timeAgo }}</div>
            </div>
            
            <!-- Actions -->
            <div class="notification-actions">
                <ion-button 
                    fill="clear" 
                    size="small"
                    (click)="markAsRead(notification); $event.stopPropagation()">
                    <ion-icon name="checkmark"></ion-icon>
                </ion-button>
                <ion-button 
                    fill="clear" 
                    size="small"
                    color="danger"
                    (click)="deleteNotification(notification); $event.stopPropagation()">
                    <ion-icon name="trash"></ion-icon>
                </ion-button>
            </div>
        </div>
    </div>
</div>
```

---

## ⚙️ **Notification Settings**

### **User Preferences**
```typescript
export class NotificationSettings {
    settings = {
        emailNotifications: true,
        pushNotifications: true,
        soundEnabled: true,
        vibrationEnabled: true,
        categories: {
            orders: { enabled: true, priority: 'normal' },
            approvals: { enabled: true, priority: 'high' },
            inventory: { enabled: true, priority: 'normal' },
            payments: { enabled: true, priority: 'normal' },
            system: { enabled: true, priority: 'low' }
        },
        quietHours: {
            enabled: false,
            startTime: '22:00',
            endTime: '08:00'
        }
    };

    async loadSettings() {
        try {
            const saved = await this.env.getStorage('notificationSettings');
            if (saved) {
                this.settings = { ...this.settings, ...saved };
            }
        } catch (error) {
            console.error('Load notification settings error:', error);
        }
    }

    async saveSettings() {
        try {
            await this.env.setStorage('notificationSettings', this.settings);
            this.env.showMessage('SETTINGS_SAVED', 'success');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    shouldShowNotification(notification: any): boolean {
        // Check if category is enabled
        const categorySettings = this.settings.categories[notification.category];
        if (!categorySettings?.enabled) {
            return false;
        }

        // Check quiet hours
        if (this.settings.quietHours.enabled && this.isQuietHours()) {
            return notification.priority === 'high';
        }

        return true;
    }

    private isQuietHours(): boolean {
        const now = new Date();
        const currentTime = now.getHours() * 100 + now.getMinutes();
        
        const startTime = this.parseTime(this.settings.quietHours.startTime);
        const endTime = this.parseTime(this.settings.quietHours.endTime);

        if (startTime > endTime) {
            // Overnight quiet hours
            return currentTime >= startTime || currentTime <= endTime;
        } else {
            // Same day quiet hours
            return currentTime >= startTime && currentTime <= endTime;
        }
    }

    private parseTime(timeString: string): number {
        const [hours, minutes] = timeString.split(':').map(Number);
        return hours * 100 + minutes;
    }
}
```

---

## 📱 **Mobile Integration**

### **Badge Count**
```typescript
export class BadgeManager {
    async updateBadgeCount() {
        const count = this.unreadCount;
        
        // Update app badge (Capacitor)
        if (Capacitor.isNativePlatform()) {
            try {
                await Badge.set({ count });
            } catch (error) {
                console.error('Badge update error:', error);
            }
        }
        
        // Update PWA badge
        if ('setAppBadge' in navigator) {
            try {
                await (navigator as any).setAppBadge(count);
            } catch (error) {
                console.error('PWA badge update error:', error);
            }
        }
        
        // Update favicon badge
        this.updateFaviconBadge(count);
    }

    private updateFaviconBadge(count: number) {
        const canvas = document.createElement('canvas');
        canvas.width = 32;
        canvas.height = 32;
        
        const ctx = canvas.getContext('2d');
        if (!ctx) return;
        
        // Draw base favicon
        const favicon = new Image();
        favicon.onload = () => {
            ctx.drawImage(favicon, 0, 0, 32, 32);
            
            if (count > 0) {
                // Draw badge
                ctx.fillStyle = '#ff0000';
                ctx.beginPath();
                ctx.arc(24, 8, 8, 0, 2 * Math.PI);
                ctx.fill();
                
                // Draw count
                ctx.fillStyle = '#ffffff';
                ctx.font = '10px Arial';
                ctx.textAlign = 'center';
                ctx.fillText(count.toString(), 24, 12);
            }
            
            // Update favicon
            const link = document.querySelector("link[rel*='icon']") as HTMLLinkElement;
            if (link) {
                link.href = canvas.toDataURL();
            }
        };
        favicon.src = '/assets/icon/favicon.ico';
    }
}
```

---

## 🔧 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Đúng - Efficient notification handling
export class OptimizedNotifications {
    private notificationQueue: any[] = [];
    private isProcessing = false;
    
    async addNotification(notification: any) {
        this.notificationQueue.push(notification);
        
        if (!this.isProcessing) {
            this.processNotificationQueue();
        }
    }
    
    private async processNotificationQueue() {
        this.isProcessing = true;
        
        while (this.notificationQueue.length > 0) {
            const notification = this.notificationQueue.shift();
            await this.processNotification(notification);
            
            // Small delay to prevent overwhelming the UI
            await new Promise(resolve => setTimeout(resolve, 100));
        }
        
        this.isProcessing = false;
    }
}
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Robust error handling
async sendNotification(notification: any) {
    try {
        await this.notificationService.send(notification);
    } catch (error) {
        // Fallback to local notification
        this.addToLocalQueue(notification);
        console.error('Send notification error:', error);
    }
}
```

### **3. Memory Management**
```typescript
// ✅ Đúng - Limit notification history
maintainNotificationHistory() {
    const maxNotifications = 100;
    
    if (this.notifications.length > maxNotifications) {
        this.notifications = this.notifications.slice(0, maxNotifications);
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
