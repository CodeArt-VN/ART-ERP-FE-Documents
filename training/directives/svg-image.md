# SvgImage Directive Documentation

## 📋 **Tổng quan**

`SvgImageDirective` là directive để load và customize SVG images dynamically. Directive này fetch SVG files via XMLHttpRequest, inject chúng vào DOM, và apply custom styling như colors và classes.

**Selector**: `[appSvgImage]`  
**Location**: `src/app/directives/svg-image.directive.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() src: string;                    // SVG file path
@Input() defaultColor: string;           // Default color for SVG elements
```

---

## 🚀 **Basic Usage**

### **Simple SVG Loading**
```typescript
// Component
export class IconDisplayPage extends PageBase {
    iconPaths = {
        home: 'assets/icons/home.svg',
        user: 'assets/icons/user.svg',
        settings: 'assets/icons/settings.svg',
        notification: 'assets/icons/notification.svg'
    };
}
```

```html
<!-- Template -->
<div class="icon-showcase">
    <div class="icon-item">
        <img appSvgImage [src]="iconPaths.home" alt="Home">
        <span>Home</span>
    </div>
    
    <div class="icon-item">
        <img appSvgImage [src]="iconPaths.user" alt="User">
        <span>User</span>
    </div>
    
    <div class="icon-item">
        <img appSvgImage [src]="iconPaths.settings" alt="Settings">
        <span>Settings</span>
    </div>
</div>
```

### **Dynamic Icon System**
```typescript
export class DynamicIconPage extends PageBase {
    menuItems = [
        { id: 'dashboard', label: 'Dashboard', icon: 'dashboard.svg', color: '#3880ff' },
        { id: 'orders', label: 'Orders', icon: 'orders.svg', color: '#10dc60' },
        { id: 'products', label: 'Products', icon: 'products.svg', color: '#ffce00' },
        { id: 'customers', label: 'Customers', icon: 'customers.svg', color: '#f04141' }
    ];
    
    getIconPath(iconName: string): string {
        return `assets/icons/menu/${iconName}`;
    }
}
```

```html
<div class="dynamic-menu">
    <ion-list>
        <ion-item button *ngFor="let item of menuItems" (click)="navigateTo(item.id)">
            <div slot="start" class="menu-icon">
                <img appSvgImage 
                     [src]="getIconPath(item.icon)"
                     [defaultColor]="item.color"
                     [id]="'icon-' + item.id"
                     class="menu-svg-icon">
            </div>
            <ion-label>{{ item.label }}</ion-label>
        </ion-item>
    </ion-list>
</div>
```

---

## 🎨 **Advanced Usage**

### **Themed Icon System**
```typescript
export class ThemedIconsPage extends PageBase {
    currentTheme: 'light' | 'dark' = 'light';
    
    iconCategories = [
        {
            category: 'Navigation',
            icons: [
                { name: 'arrow-left', file: 'arrow-left.svg' },
                { name: 'arrow-right', file: 'arrow-right.svg' },
                { name: 'arrow-up', file: 'arrow-up.svg' },
                { name: 'arrow-down', file: 'arrow-down.svg' }
            ]
        },
        {
            category: 'Actions',
            icons: [
                { name: 'edit', file: 'edit.svg' },
                { name: 'delete', file: 'delete.svg' },
                { name: 'save', file: 'save.svg' },
                { name: 'cancel', file: 'cancel.svg' }
            ]
        }
    ];
    
    toggleTheme() {
        this.currentTheme = this.currentTheme === 'light' ? 'dark' : 'light';
    }
    
    getThemeColor(): string {
        return this.currentTheme === 'light' ? '#333333' : '#ffffff';
    }
}
```

```html
<div class="themed-icons" [class]="'theme-' + currentTheme">
    <div class="theme-controls">
        <ion-button (click)="toggleTheme()" fill="outline">
            <ion-icon name="color-palette" slot="start"></ion-icon>
            Switch to {{ currentTheme === 'light' ? 'Dark' : 'Light' }} Theme
        </ion-button>
    </div>
    
    <div class="icon-categories">
        <div class="category" *ngFor="let category of iconCategories">
            <h3>{{ category.category }}</h3>
            <div class="icon-grid">
                <div class="icon-card" *ngFor="let icon of category.icons">
                    <img appSvgImage 
                         [src]="'assets/icons/' + icon.file"
                         [defaultColor]="getThemeColor()"
                         [class]="'themed-icon icon-' + icon.name">
                    <span>{{ icon.name }}</span>
                </div>
            </div>
        </div>
    </div>
</div>
```

### **Interactive SVG Icons**
```typescript
export class InteractiveSvgPage extends PageBase {
    interactiveIcons = [
        { 
            name: 'heart', 
            file: 'heart.svg', 
            states: {
                default: '#cccccc',
                hover: '#ff6b6b',
                active: '#e74c3c'
            },
            isActive: false
        },
        { 
            name: 'star', 
            file: 'star.svg', 
            states: {
                default: '#cccccc',
                hover: '#f39c12',
                active: '#e67e22'
            },
            isActive: false
        }
    ];
    
    toggleIcon(icon: any) {
        icon.isActive = !icon.isActive;
    }
    
    getIconColor(icon: any): string {
        return icon.isActive ? icon.states.active : icon.states.default;
    }
}
```

```html
<div class="interactive-icons">
    <h3>Interactive SVG Icons</h3>
    <div class="icon-row">
        <div class="interactive-icon" 
             *ngFor="let icon of interactiveIcons"
             (click)="toggleIcon(icon)"
             [class.active]="icon.isActive">
            
            <img appSvgImage 
                 [src]="'assets/icons/interactive/' + icon.file"
                 [defaultColor]="getIconColor(icon)"
                 class="clickable-svg">
            
            <span>{{ icon.name }}</span>
        </div>
    </div>
</div>
```

### **Status Indicator Icons**
```typescript
export class StatusIconsPage extends PageBase {
    systemStatus = [
        { service: 'Database', status: 'online', icon: 'database.svg' },
        { service: 'API Server', status: 'online', icon: 'server.svg' },
        { service: 'Cache', status: 'warning', icon: 'cache.svg' },
        { service: 'Storage', status: 'offline', icon: 'storage.svg' }
    ];
    
    getStatusColor(status: string): string {
        const colors = {
            'online': '#10dc60',
            'warning': '#ffce00',
            'offline': '#f04141',
            'unknown': '#92949c'
        };
        return colors[status] || colors['unknown'];
    }
    
    getStatusText(status: string): string {
        const texts = {
            'online': 'Online',
            'warning': 'Warning',
            'offline': 'Offline',
            'unknown': 'Unknown'
        };
        return texts[status] || texts['unknown'];
    }
}
```

```html
<div class="status-dashboard">
    <h3>System Status</h3>
    <div class="status-grid">
        <div class="status-card" 
             *ngFor="let item of systemStatus"
             [class]="'status-' + item.status">
            
            <div class="status-icon">
                <img appSvgImage 
                     [src]="'assets/icons/status/' + item.icon"
                     [defaultColor]="getStatusColor(item.status)"
                     class="status-svg">
            </div>
            
            <div class="status-info">
                <h4>{{ item.service }}</h4>
                <span class="status-text" [style.color]="getStatusColor(item.status)">
                    {{ getStatusText(item.status) }}
                </span>
            </div>
        </div>
    </div>
</div>
```

---

## 🎨 **Styling**

### **SVG Icon Styling**
```scss
// Component SCSS
.icon-showcase {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
    gap: 16px;
    padding: 16px;
    
    .icon-item {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 12px;
        border-radius: 8px;
        background: var(--ion-color-light);
        transition: transform 0.2s ease;
        
        &:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }
        
        img[appSvgImage] {
            width: 32px;
            height: 32px;
            margin-bottom: 8px;
            
            // SVG-specific styling
            &.replaced-svg {
                fill: currentColor;
            }
        }
        
        span {
            font-size: 0.875rem;
            color: var(--ion-color-medium);
        }
    }
}

.themed-icons {
    &.theme-light {
        background: #ffffff;
        color: #333333;
        
        .themed-icon {
            filter: none;
        }
    }
    
    &.theme-dark {
        background: #1a1a1a;
        color: #ffffff;
        
        .themed-icon {
            filter: brightness(0) invert(1);
        }
    }
    
    .icon-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
        gap: 12px;
        
        .icon-card {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 12px;
            border-radius: 6px;
            border: 1px solid var(--ion-color-medium);
            transition: all 0.2s ease;
            
            &:hover {
                border-color: var(--ion-color-primary);
                background: rgba(var(--ion-color-primary-rgb), 0.05);
            }
            
            img {
                width: 24px;
                height: 24px;
                margin-bottom: 6px;
            }
        }
    }
}

.interactive-icons {
    .icon-row {
        display: flex;
        gap: 20px;
        justify-content: center;
        
        .interactive-icon {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 16px;
            border-radius: 12px;
            cursor: pointer;
            transition: all 0.3s ease;
            border: 2px solid transparent;
            
            &:hover {
                background: rgba(var(--ion-color-primary-rgb), 0.1);
                transform: scale(1.05);
            }
            
            &.active {
                border-color: var(--ion-color-primary);
                background: rgba(var(--ion-color-primary-rgb), 0.1);
            }
            
            .clickable-svg {
                width: 40px;
                height: 40px;
                transition: all 0.3s ease;
            }
        }
    }
}

.status-dashboard {
    .status-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 16px;
        
        .status-card {
            display: flex;
            align-items: center;
            padding: 16px;
            border-radius: 8px;
            background: var(--ion-color-light);
            border-left: 4px solid transparent;
            
            &.status-online {
                border-left-color: #10dc60;
            }
            
            &.status-warning {
                border-left-color: #ffce00;
            }
            
            &.status-offline {
                border-left-color: #f04141;
            }
            
            .status-icon {
                margin-right: 12px;
                
                .status-svg {
                    width: 32px;
                    height: 32px;
                }
            }
            
            .status-info {
                h4 {
                    margin: 0 0 4px 0;
                    font-size: 1rem;
                    color: var(--ion-color-dark);
                }
                
                .status-text {
                    font-size: 0.875rem;
                    font-weight: 500;
                }
            }
        }
    }
}

// Global SVG styling
img[appSvgImage] {
    // Will be replaced with actual SVG
    &.replaced-svg {
        // Custom fill class applied by directive
        .cus-fill {
            fill: var(--ion-color-primary);
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. SVG Optimization**
```typescript
// ✅ Đúng - Optimize SVG files
// - Remove unnecessary metadata
// - Minimize file size
// - Use consistent viewBox
// - Remove inline styles that conflict with theming
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Handle loading errors
// Directive should handle XMLHttpRequest errors gracefully
// Provide fallback images or icons
```

### **3. Performance**
```typescript
// ✅ Đúng - Cache loaded SVGs
// Consider implementing caching for frequently used SVGs
// Avoid loading same SVG multiple times
```

---

## 🚨 **Common Use Cases**

### **1. Icon Libraries**
```typescript
export class IconLibraryPage extends PageBase {
    // Custom icon system with SVG files
}
```

### **2. Status Indicators**
```typescript
export class StatusPage extends PageBase {
    // Dynamic status icons with color coding
}
```

### **3. Interactive Elements**
```typescript
export class InteractivePage extends PageBase {
    // Clickable SVG icons with state changes
}
```

---

## ⚠️ **Limitations**

1. **XMLHttpRequest**: Uses synchronous XMLHttpRequest (consider async alternatives)
2. **CORS**: May have CORS issues with external SVG files
3. **Performance**: Loads SVGs individually (no batching)
4. **Browser Support**: Requires modern browser support for SVG

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
