# DateFriendly Pipe Documentation

## 📋 **Tổng quan**

`DateFriendly` pipe được sử dụng để format dates thành friendly, human-readable format với real-time updates. Pipe này sử dụng `lib.dateFormatFriendly()` và tự động update mỗi giây để hiển thị relative time như "2 minutes ago", "1 hour ago".

**Pipe Name**: `dateFriendly`  
**Location**: `src/app/pipes/friendly-format.pipe.ts`  
**Type**: Async Pipe (returns Observable)

---

## 🔧 **Signature**

```typescript
transform(date: string): Observable<string>
```

### **Parameters**
- `date` (string): Date string cần format

### **Returns**
- `Observable<string>`: Observable stream của friendly date string, updates mỗi giây

---

## 🚀 **Basic Usage**

### **Simple Date Display**
```typescript
// Component
export class ActivityFeedPage extends PageBase {
    activities = [
        { id: 1, action: 'User logged in', timestamp: '2024-12-07T10:30:00Z' },
        { id: 2, action: 'Order created', timestamp: '2024-12-07T09:15:00Z' },
        { id: 3, action: 'Payment processed', timestamp: '2024-12-06T14:20:00Z' },
        { id: 4, action: 'Email sent', timestamp: '2024-12-05T08:45:00Z' }
    ];
}
```

```html
<!-- Template -->
<div class="activity-feed">
    <div class="activity-item" *ngFor="let activity of activities">
        <div class="activity-content">
            <h4>{{ activity.action }}</h4>
            <p class="timestamp">{{ activity.timestamp | dateFriendly | async }}</p>
        </div>
    </div>
</div>
```

### **Real-time Updates**
```typescript
export class NotificationPage extends PageBase {
    notifications = [
        { 
            id: 1, 
            message: 'New order received', 
            createdAt: new Date().toISOString(),
            isRead: false 
        },
        { 
            id: 2, 
            message: 'Payment confirmed', 
            createdAt: new Date(Date.now() - 5 * 60 * 1000).toISOString(), // 5 minutes ago
            isRead: true 
        }
    ];
    
    addNotification(message: string) {
        this.notifications.unshift({
            id: Date.now(),
            message: message,
            createdAt: new Date().toISOString(),
            isRead: false
        });
    }
}
```

```html
<div class="notifications">
    <div class="notification-header">
        <h3>Notifications</h3>
        <ion-button (click)="addNotification('Test notification')" fill="outline">
            Add Test Notification
        </ion-button>
    </div>
    
    <div class="notification-list">
        <div class="notification-item" 
             *ngFor="let notification of notifications"
             [class.unread]="!notification.isRead">
            <div class="notification-content">
                <p>{{ notification.message }}</p>
                <small class="time-ago">{{ notification.createdAt | dateFriendly | async }}</small>
            </div>
            <div class="notification-status">
                <ion-badge *ngIf="!notification.isRead" color="primary">New</ion-badge>
            </div>
        </div>
    </div>
</div>
```

---

## 🎨 **Advanced Usage**

### **Chat Messages with Timestamps**
```typescript
export class ChatPage extends PageBase {
    messages = [
        { id: 1, user: 'John', text: 'Hello everyone!', timestamp: new Date(Date.now() - 2 * 60 * 1000).toISOString() },
        { id: 2, user: 'Jane', text: 'Hi John!', timestamp: new Date(Date.now() - 1 * 60 * 1000).toISOString() },
        { id: 3, user: 'Bob', text: 'How is everyone doing?', timestamp: new Date().toISOString() }
    ];
    
    newMessage = '';
    
    sendMessage() {
        if (this.newMessage.trim()) {
            this.messages.push({
                id: Date.now(),
                user: this.env.user.Name,
                text: this.newMessage,
                timestamp: new Date().toISOString()
            });
            this.newMessage = '';
        }
    }
    
    // Group messages by date for better organization
    get groupedMessages() {
        const groups = {};
        this.messages.forEach(message => {
            const date = new Date(message.timestamp).toDateString();
            if (!groups[date]) {
                groups[date] = [];
            }
            groups[date].push(message);
        });
        return groups;
    }
}
```

```html
<div class="chat-container">
    <div class="chat-messages">
        <div *ngFor="let date of objectKeys(groupedMessages)" class="message-group">
            <div class="date-separator">{{ date }}</div>
            
            <div class="message" *ngFor="let message of groupedMessages[date]">
                <div class="message-header">
                    <span class="username">{{ message.user }}</span>
                    <span class="timestamp">{{ message.timestamp | dateFriendly | async }}</span>
                </div>
                <div class="message-content">{{ message.text }}</div>
            </div>
        </div>
    </div>
    
    <div class="chat-input">
        <ion-item>
            <ion-input 
                [(ngModel)]="newMessage"
                placeholder="Type a message..."
                (keyup.enter)="sendMessage()">
            </ion-input>
            <ion-button (click)="sendMessage()" slot="end">
                <ion-icon name="send"></ion-icon>
            </ion-button>
        </ion-item>
    </div>
</div>
```

### **Order Timeline**
```typescript
export class OrderTimelinePage extends PageBase {
    orderEvents = [
        { status: 'Order Placed', timestamp: '2024-12-07T08:00:00Z', description: 'Order #12345 has been placed' },
        { status: 'Payment Confirmed', timestamp: '2024-12-07T08:05:00Z', description: 'Payment processed successfully' },
        { status: 'Processing', timestamp: '2024-12-07T09:00:00Z', description: 'Order is being prepared' },
        { status: 'Shipped', timestamp: '2024-12-07T14:30:00Z', description: 'Order has been shipped' },
        { status: 'Delivered', timestamp: null, description: 'Estimated delivery time' }
    ];
    
    getStatusIcon(status: string): string {
        const icons = {
            'Order Placed': 'receipt-outline',
            'Payment Confirmed': 'card-outline',
            'Processing': 'construct-outline',
            'Shipped': 'airplane-outline',
            'Delivered': 'checkmark-circle-outline'
        };
        return icons[status] || 'ellipse-outline';
    }
    
    getStatusColor(status: string): string {
        const colors = {
            'Order Placed': 'primary',
            'Payment Confirmed': 'success',
            'Processing': 'warning',
            'Shipped': 'tertiary',
            'Delivered': 'success'
        };
        return colors[status] || 'medium';
    }
}
```

```html
<div class="order-timeline">
    <h3>Order Timeline</h3>
    
    <div class="timeline">
        <div class="timeline-item" 
             *ngFor="let event of orderEvents; let last = last"
             [class.completed]="event.timestamp"
             [class.pending]="!event.timestamp">
            
            <div class="timeline-marker">
                <ion-icon 
                    [name]="getStatusIcon(event.status)"
                    [color]="event.timestamp ? getStatusColor(event.status) : 'medium'">
                </ion-icon>
            </div>
            
            <div class="timeline-content">
                <div class="timeline-header">
                    <h4>{{ event.status }}</h4>
                    <span class="timeline-time" *ngIf="event.timestamp">
                        {{ event.timestamp | dateFriendly | async }}
                    </span>
                    <span class="timeline-time pending" *ngIf="!event.timestamp">
                        Pending
                    </span>
                </div>
                <p>{{ event.description }}</p>
            </div>
            
            <div class="timeline-line" *ngIf="!last"></div>
        </div>
    </div>
</div>
```

### **Activity Dashboard**
```typescript
export class ActivityDashboardPage extends PageBase {
    recentActivities = [];
    
    async ngOnInit() {
        await this.loadRecentActivities();
        
        // Auto-refresh every 30 seconds
        setInterval(() => {
            this.loadRecentActivities();
        }, 30000);
    }
    
    async loadRecentActivities() {
        try {
            const activities = await this.activityService.getRecent(20);
            this.recentActivities = activities.map(activity => ({
                ...activity,
                timestamp: activity.CreatedDate || activity.UpdatedDate
            }));
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    getActivityIcon(type: string): string {
        const icons = {
            'login': 'log-in-outline',
            'logout': 'log-out-outline',
            'create': 'add-circle-outline',
            'update': 'create-outline',
            'delete': 'trash-outline',
            'view': 'eye-outline'
        };
        return icons[type] || 'information-circle-outline';
    }
}
```

```html
<div class="activity-dashboard">
    <div class="dashboard-header">
        <h3>Recent Activity</h3>
        <ion-button (click)="loadRecentActivities()" fill="clear">
            <ion-icon name="refresh" slot="start"></ion-icon>
            Refresh
        </ion-button>
    </div>
    
    <div class="activity-stream">
        <div class="activity-item" *ngFor="let activity of recentActivities">
            <div class="activity-icon">
                <ion-icon [name]="getActivityIcon(activity.Type)"></ion-icon>
            </div>
            
            <div class="activity-details">
                <div class="activity-description">
                    <strong>{{ activity.UserName }}</strong> {{ activity.Description }}
                </div>
                <div class="activity-meta">
                    <span class="activity-time">{{ activity.timestamp | dateFriendly | async }}</span>
                    <span class="activity-module">{{ activity.ModuleName }}</span>
                </div>
            </div>
        </div>
        
        <div class="empty-state" *ngIf="recentActivities.length === 0">
            <ion-icon name="time-outline"></ion-icon>
            <p>No recent activity</p>
        </div>
    </div>
</div>
```

---

## 🎨 **Styling**

### **Timestamp Styling**
```scss
.timestamp, .time-ago {
    font-size: 0.875rem;
    color: var(--ion-color-medium);
    font-style: italic;
}

.notification-item {
    display: flex;
    align-items: flex-start;
    padding: 12px;
    border-bottom: 1px solid var(--ion-color-light);
    
    &.unread {
        background-color: rgba(var(--ion-color-primary-rgb), 0.05);
        border-left: 3px solid var(--ion-color-primary);
    }
    
    .notification-content {
        flex: 1;
        
        p {
            margin: 0 0 4px 0;
            font-weight: 500;
        }
        
        .time-ago {
            font-size: 0.75rem;
            color: var(--ion-color-medium);
        }
    }
}

.timeline {
    position: relative;
    
    .timeline-item {
        display: flex;
        align-items: flex-start;
        margin-bottom: 24px;
        position: relative;
        
        &.completed .timeline-marker ion-icon {
            color: var(--ion-color-success);
        }
        
        &.pending .timeline-marker ion-icon {
            color: var(--ion-color-medium);
        }
        
        .timeline-marker {
            width: 40px;
            height: 40px;
            border-radius: 50%;
            background: var(--ion-color-light);
            display: flex;
            align-items: center;
            justify-content: center;
            margin-right: 16px;
            z-index: 2;
            
            ion-icon {
                font-size: 20px;
            }
        }
        
        .timeline-content {
            flex: 1;
            
            .timeline-header {
                display: flex;
                justify-content: space-between;
                align-items: center;
                margin-bottom: 4px;
                
                h4 {
                    margin: 0;
                    font-size: 1rem;
                    font-weight: 600;
                }
                
                .timeline-time {
                    font-size: 0.875rem;
                    color: var(--ion-color-medium);
                    
                    &.pending {
                        color: var(--ion-color-warning);
                        font-style: italic;
                    }
                }
            }
            
            p {
                margin: 0;
                color: var(--ion-color-medium-shade);
                line-height: 1.4;
            }
        }
        
        .timeline-line {
            position: absolute;
            left: 19px;
            top: 40px;
            width: 2px;
            height: 24px;
            background: var(--ion-color-light-shade);
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Memory Management**
```typescript
// ✅ Đúng - Use async pipe to auto-unsubscribe
// Template: {{ date | dateFriendly | async }}

// ❌ Sai - Manual subscription without cleanup
ngOnInit() {
    this.dateFriendlyPipe.transform(this.date).subscribe(result => {
        this.friendlyDate = result;
    }); // Memory leak risk
}
```

### **2. Performance Optimization**
```typescript
// ✅ Đúng - Use trackBy for lists with timestamps
trackByActivity(index: number, activity: any): any {
    return activity.id;
}

// ✅ Đúng - Limit real-time updates for large lists
get recentActivities() {
    return this.allActivities.slice(0, 20); // Limit to 20 items
}
```

### **3. Null/Undefined Handling**
```html
<!-- ✅ Đúng - Handle null dates -->
<span *ngIf="item.timestamp">
    {{ item.timestamp | dateFriendly | async }}
</span>
<span *ngIf="!item.timestamp" class="no-date">
    No date available
</span>
```

---

## 🚨 **Common Use Cases**

### **1. Social Media Feed**
```typescript
export class SocialFeedPage extends PageBase {
    posts: Post[] = [];
    
    // Each post shows "2 hours ago", "1 day ago", etc.
}
```

### **2. System Logs**
```typescript
export class SystemLogsPage extends PageBase {
    logs: LogEntry[] = [];
    
    // Show when each log entry was created
}
```

### **3. Comment Threads**
```typescript
export class CommentThreadPage extends PageBase {
    comments: Comment[] = [];
    
    // Show relative time for each comment
}
```

---

## ⚡ **Performance Considerations**

### **Timer Impact**
- Pipe creates a timer that updates every second
- Multiple instances create multiple timers
- Consider performance impact with many date displays

### **Optimization Strategies**
```typescript
// ✅ Đúng - Use OnPush change detection
@Component({
    changeDetection: ChangeDetectionStrategy.OnPush
})

// ✅ Đúng - Limit number of real-time dates
get displayedItems() {
    return this.allItems.slice(0, 50); // Limit items
}
```

---

## 🔄 **Integration with Global Functions**

The pipe uses `lib.dateFormatFriendly()` which provides:
- "just now" for very recent dates
- "X minutes ago" for recent times
- "X hours ago" for same day
- "X days ago" for recent dates
- Full date format for older dates

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
