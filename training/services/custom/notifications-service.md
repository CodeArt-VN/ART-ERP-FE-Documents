# OSM_NotificationService Documentation

## 📋 **Tổng quan**

`OSM_NotificationService` là service để handle user notifications trong hệ thống OSM (Operations & Service Management). Service này extend từ `OSM_NotificationProvider` và provide comprehensive functionality cho việc đọc, quản lý và xử lý notifications.

**Location**: `src/app/services/custom/notifications.service.ts`  
**Injectable**: `providedIn: 'root'`  
**Extends**: `OSM_NotificationProvider`

---

## 🔧 **Key Features**

### **Notification Management**
- **Read Notifications**: Lấy danh sách notifications của user
- **Mark as Read/Unread**: Đánh dấu đã đọc/chưa đọc
- **Delete Notifications**: Xóa notifications
- **Count Management**: Đếm số notifications chưa đọc
- **Real-time Updates**: Cập nhật notifications real-time

### **Notification Types**
- **System Notifications**: Thông báo hệ thống
- **User Notifications**: Thông báo cá nhân
- **Alert Notifications**: Thông báo cảnh báo
- **Task Notifications**: Thông báo công việc

---

## 🚀 **Core Methods**

### **Read Operations**
```typescript
readNotification(query: any): Promise<any>
getNotificationCount(query: any): Promise<any>
```

### **Status Management**
```typescript
markAsRead(postDTO: any): Promise<any>
markAsUnRead(postDTO: any): Promise<any>
```

### **Delete Operations**
```typescript
deleteNotification(postDTO: any): Promise<any>
```

---

## 🎨 **Usage Examples**

### **1. Load User Notifications**
```typescript
export class NotificationListComponent implements OnInit {
    notifications: any[] = [];
    unreadCount: number = 0;
    loading: boolean = false;
    
    constructor(private notificationService: OSM_NotificationService) {}
    
    async ngOnInit() {
        await this.loadNotifications();
        await this.loadUnreadCount();
        
        // Setup real-time updates
        this.setupRealTimeUpdates();
    }
    
    async loadNotifications() {
        this.loading = true;
        
        try {
            const query = {
                page: 1,
                limit: 20,
                sortBy: 'CreatedDate',
                sortOrder: 'DESC'
            };
            
            const result = await this.notificationService.readNotification(query);
            
            this.notifications = result.data || [];
            
        } catch (error) {
            console.error('Failed to load notifications:', error);
            this.env.showErrorMessage('Failed to load notifications');
        } finally {
            this.loading = false;
        }
    }
    
    async loadUnreadCount() {
        try {
            const query = {
                status: 'unread'
            };
            
            const result = await this.notificationService.getNotificationCount(query);
            this.unreadCount = result.count || 0;
            
            // Update badge in header
            this.updateNotificationBadge(this.unreadCount);
            
        } catch (error) {
            console.error('Failed to load notification count:', error);
        }
    }
    
    private updateNotificationBadge(count: number) {
        this.env.publishEvent({
            Code: 'notification:count:updated',
            Value: { count }
        });
    }
    
    private setupRealTimeUpdates() {
        // Subscribe to real-time notification events
        this.env.getEvents().subscribe(event => {
            if (event.Code === 'notification:new') {
                this.handleNewNotification(event.Value);
            } else if (event.Code === 'notification:updated') {
                this.handleNotificationUpdate(event.Value);
            }
        });
    }
    
    private handleNewNotification(notification: any) {
        // Add new notification to the top of the list
        this.notifications.unshift(notification);
        
        // Update unread count
        if (!notification.IsRead) {
            this.unreadCount++;
            this.updateNotificationBadge(this.unreadCount);
        }
        
        // Show toast for important notifications
        if (notification.Priority === 'High') {
            this.env.showMessage(notification.Title, 'warning');
        }
    }
}
```

### **2. Notification Actions**
```typescript
export class NotificationActionsService {
    constructor(private notificationService: OSM_NotificationService) {}
    
    async markNotificationAsRead(notificationId: number) {
        try {
            const postDTO = {
                Id: notificationId,
                IsRead: true,
                ReadDate: new Date().toISOString()
            };
            
            await this.notificationService.markAsRead(postDTO);
            
            // Update local state
            this.updateNotificationStatus(notificationId, true);
            
            // Decrease unread count
            this.decreaseUnreadCount();
            
        } catch (error) {
            console.error('Failed to mark notification as read:', error);
            this.env.showErrorMessage('Failed to update notification');
        }
    }
    
    async markNotificationAsUnread(notificationId: number) {
        try {
            const postDTO = {
                Id: notificationId,
                IsRead: false,
                ReadDate: null
            };
            
            await this.notificationService.markAsUnRead(postDTO);
            
            // Update local state
            this.updateNotificationStatus(notificationId, false);
            
            // Increase unread count
            this.increaseUnreadCount();
            
        } catch (error) {
            console.error('Failed to mark notification as unread:', error);
            this.env.showErrorMessage('Failed to update notification');
        }
    }
    
    async deleteNotification(notificationId: number) {
        const confirmed = await this.env.showPrompt(
            'Are you sure you want to delete this notification?',
            'Delete Notification',
            'Delete'
        );
        
        if (confirmed) {
            try {
                const postDTO = {
                    Id: notificationId
                };
                
                await this.notificationService.deleteNotification(postDTO);
                
                // Remove from local list
                this.removeNotificationFromList(notificationId);
                
                this.env.showMessage('Notification deleted successfully', 'success');
                
            } catch (error) {
                console.error('Failed to delete notification:', error);
                this.env.showErrorMessage('Failed to delete notification');
            }
        }
    }
    
    async markAllAsRead() {
        try {
            const unreadNotifications = this.notifications.filter(n => !n.IsRead);
            
            if (unreadNotifications.length === 0) {
                this.env.showMessage('No unread notifications', 'info');
                return;
            }
            
            // Show loading
            await this.env.showLoading('Marking all as read...', 
                this.processMarkAllAsRead(unreadNotifications)
            );
            
        } catch (error) {
            console.error('Failed to mark all as read:', error);
            this.env.showErrorMessage('Failed to update notifications');
        }
    }
    
    private async processMarkAllAsRead(notifications: any[]) {
        const promises = notifications.map(notification => 
            this.notificationService.markAsRead({
                Id: notification.Id,
                IsRead: true,
                ReadDate: new Date().toISOString()
            })
        );
        
        await Promise.all(promises);
        
        // Update local state
        notifications.forEach(notification => {
            notification.IsRead = true;
            notification.ReadDate = new Date().toISOString();
        });
        
        // Reset unread count
        this.unreadCount = 0;
        this.updateNotificationBadge(0);
        
        this.env.showMessage('All notifications marked as read', 'success');
    }
}
```

### **3. Notification Filtering & Search**
```typescript
export class NotificationFilterComponent {
    notifications: any[] = [];
    filteredNotifications: any[] = [];
    
    filterOptions = {
        status: 'all', // all, read, unread
        priority: 'all', // all, high, medium, low
        type: 'all', // all, system, user, alert, task
        dateRange: null
    };
    
    searchTerm: string = '';
    
    constructor(private notificationService: OSM_NotificationService) {}
    
    async applyFilters() {
        let filtered = [...this.notifications];
        
        // Filter by status
        if (this.filterOptions.status !== 'all') {
            const isRead = this.filterOptions.status === 'read';
            filtered = filtered.filter(n => n.IsRead === isRead);
        }
        
        // Filter by priority
        if (this.filterOptions.priority !== 'all') {
            filtered = filtered.filter(n => 
                n.Priority?.toLowerCase() === this.filterOptions.priority
            );
        }
        
        // Filter by type
        if (this.filterOptions.type !== 'all') {
            filtered = filtered.filter(n => 
                n.Type?.toLowerCase() === this.filterOptions.type
            );
        }
        
        // Filter by date range
        if (this.filterOptions.dateRange) {
            const { start, end } = this.filterOptions.dateRange;
            filtered = filtered.filter(n => {
                const notificationDate = new Date(n.CreatedDate);
                return notificationDate >= start && notificationDate <= end;
            });
        }
        
        // Apply search term
        if (this.searchTerm) {
            const searchLower = this.searchTerm.toLowerCase();
            filtered = filtered.filter(n => 
                n.Title?.toLowerCase().includes(searchLower) ||
                n.Content?.toLowerCase().includes(searchLower)
            );
        }
        
        this.filteredNotifications = filtered;
    }
    
    async loadNotificationsWithFilter() {
        try {
            const query = {
                ...this.buildQueryFromFilters(),
                page: 1,
                limit: 50
            };
            
            const result = await this.notificationService.readNotification(query);
            
            this.notifications = result.data || [];
            this.applyFilters();
            
        } catch (error) {
            console.error('Failed to load filtered notifications:', error);
        }
    }
    
    private buildQueryFromFilters(): any {
        const query: any = {};
        
        if (this.filterOptions.status !== 'all') {
            query.IsRead = this.filterOptions.status === 'read';
        }
        
        if (this.filterOptions.priority !== 'all') {
            query.Priority = this.filterOptions.priority;
        }
        
        if (this.filterOptions.type !== 'all') {
            query.Type = this.filterOptions.type;
        }
        
        if (this.filterOptions.dateRange) {
            query.DateFrom = this.filterOptions.dateRange.start.toISOString();
            query.DateTo = this.filterOptions.dateRange.end.toISOString();
        }
        
        return query;
    }
}
```

### **4. Notification Templates & Formatting**
```typescript
export class NotificationDisplayService {
    constructor(private notificationService: OSM_NotificationService) {}
    
    formatNotificationContent(notification: any): any {
        return {
            ...notification,
            formattedTitle: this.formatTitle(notification),
            formattedContent: this.formatContent(notification),
            formattedDate: this.formatDate(notification.CreatedDate),
            icon: this.getNotificationIcon(notification),
            color: this.getNotificationColor(notification)
        };
    }
    
    private formatTitle(notification: any): string {
        if (notification.Type === 'system') {
            return `🔧 ${notification.Title}`;
        } else if (notification.Type === 'alert') {
            return `⚠️ ${notification.Title}`;
        } else if (notification.Type === 'task') {
            return `📋 ${notification.Title}`;
        }
        
        return notification.Title;
    }
    
    private formatContent(notification: any): string {
        // Parse and format content based on type
        if (notification.ContentType === 'json') {
            try {
                const content = JSON.parse(notification.Content);
                return this.formatJsonContent(content);
            } catch {
                return notification.Content;
            }
        }
        
        return notification.Content;
    }
    
    private formatJsonContent(content: any): string {
        if (content.template) {
            return this.applyTemplate(content.template, content.data);
        }
        
        return JSON.stringify(content, null, 2);
    }
    
    private applyTemplate(template: string, data: any): string {
        let result = template;
        
        for (const key in data) {
            const placeholder = `{{${key}}}`;
            result = result.replace(new RegExp(placeholder, 'g'), data[key]);
        }
        
        return result;
    }
    
    private formatDate(dateString: string): string {
        const date = new Date(dateString);
        const now = new Date();
        const diffMs = now.getTime() - date.getTime();
        const diffMins = Math.floor(diffMs / (1000 * 60));
        const diffHours = Math.floor(diffMins / 60);
        const diffDays = Math.floor(diffHours / 24);
        
        if (diffMins < 1) {
            return 'Just now';
        } else if (diffMins < 60) {
            return `${diffMins} minutes ago`;
        } else if (diffHours < 24) {
            return `${diffHours} hours ago`;
        } else if (diffDays < 7) {
            return `${diffDays} days ago`;
        } else {
            return date.toLocaleDateString();
        }
    }
    
    private getNotificationIcon(notification: any): string {
        const iconMap = {
            'system': 'settings-outline',
            'alert': 'warning-outline',
            'task': 'checkbox-outline',
            'user': 'person-outline',
            'default': 'notifications-outline'
        };
        
        return iconMap[notification.Type] || iconMap['default'];
    }
    
    private getNotificationColor(notification: any): string {
        if (!notification.IsRead) {
            return 'primary';
        }
        
        const colorMap = {
            'High': 'danger',
            'Medium': 'warning',
            'Low': 'medium',
            'default': 'medium'
        };
        
        return colorMap[notification.Priority] || colorMap['default'];
    }
}
```

---

## 🔧 **Advanced Features**

### **1. Notification Grouping**
```typescript
export class NotificationGroupingService {
    constructor(private notificationService: OSM_NotificationService) {}
    
    groupNotificationsByDate(notifications: any[]): any {
        const groups = {};
        
        notifications.forEach(notification => {
            const date = new Date(notification.CreatedDate);
            const dateKey = date.toDateString();
            
            if (!groups[dateKey]) {
                groups[dateKey] = {
                    date: dateKey,
                    displayDate: this.getDisplayDate(date),
                    notifications: []
                };
            }
            
            groups[dateKey].notifications.push(notification);
        });
        
        return Object.values(groups).sort((a: any, b: any) => 
            new Date(b.date).getTime() - new Date(a.date).getTime()
        );
    }
    
    groupNotificationsByType(notifications: any[]): any {
        const groups = {};
        
        notifications.forEach(notification => {
            const type = notification.Type || 'other';
            
            if (!groups[type]) {
                groups[type] = {
                    type: type,
                    displayName: this.getTypeDisplayName(type),
                    count: 0,
                    notifications: []
                };
            }
            
            groups[type].notifications.push(notification);
            groups[type].count++;
        });
        
        return Object.values(groups);
    }
    
    private getDisplayDate(date: Date): string {
        const today = new Date();
        const yesterday = new Date(today);
        yesterday.setDate(yesterday.getDate() - 1);
        
        if (date.toDateString() === today.toDateString()) {
            return 'Today';
        } else if (date.toDateString() === yesterday.toDateString()) {
            return 'Yesterday';
        } else {
            return date.toLocaleDateString();
        }
    }
    
    private getTypeDisplayName(type: string): string {
        const typeNames = {
            'system': 'System Notifications',
            'alert': 'Alerts',
            'task': 'Tasks',
            'user': 'User Messages',
            'other': 'Other'
        };
        
        return typeNames[type] || type;
    }
}
```

### **2. Notification Preferences**
```typescript
export class NotificationPreferencesService {
    constructor(private notificationService: OSM_NotificationService) {}
    
    async updateNotificationPreferences(preferences: any): Promise<any> {
        try {
            const result = await this.notificationService.updatePreferences(preferences);
            
            // Update local preferences
            await this.env.setStorage('notificationPreferences', preferences);
            
            this.env.showMessage('Preferences updated successfully', 'success');
            
            return result;
        } catch (error) {
            console.error('Failed to update preferences:', error);
            this.env.showErrorMessage('Failed to update preferences');
            throw error;
        }
    }
    
    async getNotificationPreferences(): Promise<any> {
        try {
            // Try to get from storage first
            let preferences = await this.env.getStorage('notificationPreferences');
            
            if (!preferences) {
                // Fetch from server
                preferences = await this.notificationService.getPreferences();
                await this.env.setStorage('notificationPreferences', preferences);
            }
            
            return preferences;
        } catch (error) {
            console.error('Failed to get preferences:', error);
            // Return default preferences
            return this.getDefaultPreferences();
        }
    }
    
    private getDefaultPreferences(): any {
        return {
            emailNotifications: true,
            pushNotifications: true,
            soundEnabled: true,
            notificationTypes: {
                system: true,
                alert: true,
                task: true,
                user: true
            },
            quietHours: {
                enabled: false,
                start: '22:00',
                end: '08:00'
            }
        };
    }
}
```

---

## 📋 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Implement pagination for large notification lists
async loadNotificationsWithPagination(page: number = 1) {
    const query = {
        page: page,
        limit: 20, // Reasonable page size
        sortBy: 'CreatedDate',
        sortOrder: 'DESC'
    };
    
    return this.notificationService.readNotification(query);
}

// ✅ Use virtual scrolling for large lists
// In template: <ion-virtual-scroll [items]="notifications">
```

### **2. Error Handling**
```typescript
// ✅ Graceful error handling
async handleNotificationAction(action: string, notificationId: number) {
    try {
        switch (action) {
            case 'read':
                await this.markAsRead(notificationId);
                break;
            case 'delete':
                await this.deleteNotification(notificationId);
                break;
        }
    } catch (error) {
        // Don't show error for non-critical operations
        console.warn(`Notification ${action} failed:`, error);
    }
}
```

### **3. Real-time Updates**
```typescript
// ✅ Efficient real-time updates
setupNotificationUpdates() {
    // Use WebSocket or Server-Sent Events
    this.env.getEvents().pipe(
        filter(event => event.Code.startsWith('notification:')),
        debounceTime(100) // Prevent too frequent updates
    ).subscribe(event => {
        this.handleNotificationEvent(event);
    });
}
```

---

## 🚨 **Security Considerations**

### **1. Data Validation**
```typescript
// ✅ Validate notification data
validateNotificationData(notification: any): boolean {
    return notification && 
           typeof notification.Id === 'number' &&
           typeof notification.Title === 'string' &&
           notification.Title.length <= 200;
}
```

### **2. XSS Prevention**
```typescript
// ✅ Sanitize notification content
sanitizeNotificationContent(content: string): string {
    // Remove potentially dangerous HTML tags
    return content.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');
}
```

OSM_NotificationService cung cấp comprehensive notification management với real-time updates, advanced filtering, và user preference support cho enterprise applications.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
