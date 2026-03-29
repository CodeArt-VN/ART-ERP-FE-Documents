# SafeStyle Pipe Documentation

## 📋 **Tổng quan**

`SafeStyle` pipe được sử dụng để bypass Angular security restrictions cho CSS styles. Pipe này sử dụng `DomSanitizer.bypassSecurityTrustStyle()` để mark CSS styles as trusted.

**Pipe Name**: `safeStyle`  
**Location**: `src/app/pipes/filter.ts`  
**Type**: Pure Pipe

---

## 🔧 **Signature**

```typescript
transform(style: string): SafeStyle
```

### **Parameters**
- `style` (string): CSS style string cần bypass security

### **Returns**
- `SafeStyle`: Trusted CSS style có thể apply vào elements

---

## 🚀 **Basic Usage**

### **Dynamic Styling**
```typescript
// Component
export class DynamicStylingPage extends PageBase {
    headerStyle = 'background: linear-gradient(45deg, #ff6b6b, #4ecdc4); color: white; padding: 20px;';
    
    cardStyle = `
        background: #f8f9fa;
        border: 2px solid #dee2e6;
        border-radius: 12px;
        padding: 16px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    `;
    
    progressBarStyle = 'width: 75%; background: #28a745; height: 8px; border-radius: 4px;';
}
```

```html
<!-- Template -->
<div class="styled-content">
    <!-- Dynamic header styling -->
    <header [style]="headerStyle | safeStyle">
        <h1>Dynamic Header</h1>
    </header>
    
    <!-- Card with custom styling -->
    <div class="card" [style]="cardStyle | safeStyle">
        <h3>Custom Card</h3>
        <p>This card has dynamic styling applied.</p>
    </div>
    
    <!-- Progress bar -->
    <div class="progress-container">
        <div class="progress-bar" [style]="progressBarStyle | safeStyle"></div>
    </div>
</div>
```

---

## 🎨 **Advanced Usage**

### **Theme-based Styling**
```typescript
export class ThemePage extends PageBase {
    currentTheme: 'light' | 'dark' | 'custom' = 'light';
    customColors = {
        primary: '#3880ff',
        secondary: '#0cd1e8',
        tertiary: '#7044ff',
        success: '#10dc60',
        warning: '#ffce00',
        danger: '#f04141'
    };
    
    get themeStyles(): string {
        switch (this.currentTheme) {
            case 'light':
                return `
                    background: #ffffff;
                    color: #000000;
                    border: 1px solid #e0e0e0;
                `;
            case 'dark':
                return `
                    background: #1a1a1a;
                    color: #ffffff;
                    border: 1px solid #333333;
                `;
            case 'custom':
                return `
                    background: linear-gradient(135deg, ${this.customColors.primary}, ${this.customColors.secondary});
                    color: white;
                    border: 2px solid ${this.customColors.tertiary};
                `;
            default:
                return '';
        }
    }
    
    generateCustomStyle(config: any): string {
        const styles = [];
        
        if (config.backgroundColor) {
            styles.push(`background-color: ${config.backgroundColor}`);
        }
        
        if (config.textColor) {
            styles.push(`color: ${config.textColor}`);
        }
        
        if (config.borderRadius) {
            styles.push(`border-radius: ${config.borderRadius}px`);
        }
        
        if (config.padding) {
            styles.push(`padding: ${config.padding}px`);
        }
        
        return styles.join('; ');
    }
}
```

```html
<div class="theme-demo">
    <div class="theme-selector">
        <ion-segment [(ngModel)]="currentTheme">
            <ion-segment-button value="light">Light</ion-segment-button>
            <ion-segment-button value="dark">Dark</ion-segment-button>
            <ion-segment-button value="custom">Custom</ion-segment-button>
        </ion-segment>
    </div>
    
    <div class="themed-content" [style]="themeStyles | safeStyle">
        <h2>Themed Content</h2>
        <p>This content changes based on the selected theme.</p>
    </div>
</div>
```

### **Data-driven Styling**
```typescript
export class DataVisualizationPage extends PageBase {
    chartData = [
        { label: 'Sales', value: 75, color: '#28a745' },
        { label: 'Marketing', value: 60, color: '#007bff' },
        { label: 'Support', value: 90, color: '#ffc107' },
        { label: 'Development', value: 85, color: '#dc3545' }
    ];
    
    getBarStyle(item: any): string {
        return `
            width: ${item.value}%;
            background-color: ${item.color};
            height: 24px;
            border-radius: 12px;
            transition: width 0.3s ease;
        `;
    }
    
    getIndicatorStyle(status: string): string {
        const statusColors = {
            'active': '#28a745',
            'inactive': '#6c757d',
            'pending': '#ffc107',
            'error': '#dc3545'
        };
        
        return `
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background-color: ${statusColors[status] || '#6c757d'};
            display: inline-block;
            margin-right: 8px;
        `;
    }
}
```

```html
<div class="data-visualization">
    <h3>Performance Chart</h3>
    <div class="chart-container">
        <div class="chart-item" *ngFor="let item of chartData">
            <div class="chart-label">{{ item.label }}</div>
            <div class="chart-bar-container">
                <div class="chart-bar" [style]="getBarStyle(item) | safeStyle"></div>
                <span class="chart-value">{{ item.value }}%</span>
            </div>
        </div>
    </div>
    
    <h3>Status Indicators</h3>
    <div class="status-list">
        <div class="status-item">
            <span [style]="getIndicatorStyle('active') | safeStyle"></span>
            Active Services
        </div>
        <div class="status-item">
            <span [style]="getIndicatorStyle('pending') | safeStyle"></span>
            Pending Tasks
        </div>
        <div class="status-item">
            <span [style]="getIndicatorStyle('error') | safeStyle"></span>
            Error Reports
        </div>
    </div>
</div>
```

### **Responsive Styling**
```typescript
export class ResponsivePage extends PageBase {
    screenSize: 'mobile' | 'tablet' | 'desktop' = 'desktop';
    
    ngOnInit() {
        this.detectScreenSize();
        window.addEventListener('resize', () => this.detectScreenSize());
    }
    
    detectScreenSize() {
        const width = window.innerWidth;
        if (width < 768) {
            this.screenSize = 'mobile';
        } else if (width < 1024) {
            this.screenSize = 'tablet';
        } else {
            this.screenSize = 'desktop';
        }
    }
    
    get responsiveGridStyle(): string {
        const gridConfigs = {
            mobile: 'grid-template-columns: 1fr; gap: 8px;',
            tablet: 'grid-template-columns: repeat(2, 1fr); gap: 12px;',
            desktop: 'grid-template-columns: repeat(3, 1fr); gap: 16px;'
        };
        
        return `
            display: grid;
            ${gridConfigs[this.screenSize]}
            padding: 16px;
        `;
    }
    
    get responsiveCardStyle(): string {
        const cardConfigs = {
            mobile: 'padding: 12px; font-size: 14px;',
            tablet: 'padding: 16px; font-size: 15px;',
            desktop: 'padding: 20px; font-size: 16px;'
        };
        
        return `
            background: white;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
            ${cardConfigs[this.screenSize]}
        `;
    }
}
```

```html
<div class="responsive-layout" [style]="responsiveGridStyle | safeStyle">
    <div class="card" 
         *ngFor="let item of items" 
         [style]="responsiveCardStyle | safeStyle">
        <h4>{{ item.title }}</h4>
        <p>{{ item.description }}</p>
    </div>
</div>
```

---

## 🔒 **Security Considerations**

### **Style Validation**
```typescript
export class SecureStylePage extends PageBase {
    private dangerousProperties = [
        'expression', 'javascript:', 'vbscript:', 'onload', 'onerror',
        'behavior', '-moz-binding', 'filter'
    ];
    
    validateStyle(style: string): boolean {
        const lowercaseStyle = style.toLowerCase();
        
        // Check for dangerous properties
        for (const dangerous of this.dangerousProperties) {
            if (lowercaseStyle.includes(dangerous)) {
                this.env.showMessage('Unsafe style detected', 'danger');
                return false;
            }
        }
        
        return true;
    }
    
    sanitizeStyle(style: string): string {
        // Remove potentially dangerous content
        let sanitized = style;
        
        // Remove javascript: URLs
        sanitized = sanitized.replace(/javascript:/gi, '');
        
        // Remove expression() calls
        sanitized = sanitized.replace(/expression\s*\([^)]*\)/gi, '');
        
        // Remove behavior property
        sanitized = sanitized.replace(/behavior\s*:[^;]*/gi, '');
        
        return sanitized;
    }
    
    applySecureStyle(style: string): string {
        if (this.validateStyle(style)) {
            return this.sanitizeStyle(style);
        }
        return '';
    }
}
```

---

## 🎨 **Styling Examples**

### **Animation Styles**
```typescript
export class AnimationPage extends PageBase {
    pulseAnimation = `
        animation: pulse 2s infinite;
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }
    `;
    
    slideInAnimation = `
        animation: slideIn 0.5s ease-out;
        @keyframes slideIn {
            from { transform: translateX(-100%); opacity: 0; }
            to { transform: translateX(0); opacity: 1; }
        }
    `;
}
```

### **Layout Styles**
```typescript
export class LayoutPage extends PageBase {
    flexCenterStyle = `
        display: flex;
        align-items: center;
        justify-content: center;
        min-height: 200px;
    `;
    
    gridLayoutStyle = `
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
        gap: 20px;
        padding: 20px;
    `;
}
```

---

## 🔧 **Best Practices**

### **1. Style Validation**
```typescript
// ✅ Đúng - Validate styles before applying
applyStyle(style: string) {
    if (this.validateStyle(style)) {
        this.elementStyle = this.sanitizeStyle(style);
    }
}

// ❌ Sai - Apply styles without validation
applyStyle(style: string) {
    this.elementStyle = style; // Security risk
}
```

### **2. Performance**
```typescript
// ✅ Đúng - Cache computed styles
private styleCache = new Map<string, string>();

getComputedStyle(config: any): string {
    const key = JSON.stringify(config);
    if (this.styleCache.has(key)) {
        return this.styleCache.get(key);
    }
    
    const style = this.generateStyle(config);
    this.styleCache.set(key, style);
    return style;
}
```

### **3. Maintainability**
```typescript
// ✅ Đúng - Use style builders
class StyleBuilder {
    private styles: string[] = [];
    
    background(color: string): this {
        this.styles.push(`background-color: ${color}`);
        return this;
    }
    
    padding(value: string): this {
        this.styles.push(`padding: ${value}`);
        return this;
    }
    
    build(): string {
        return this.styles.join('; ');
    }
}
```

---

## 🚨 **Common Use Cases**

### **1. Dynamic Themes**
```typescript
export class ThemeService {
    generateThemeStyle(theme: any): string {
        return new StyleBuilder()
            .background(theme.backgroundColor)
            .color(theme.textColor)
            .border(`1px solid ${theme.borderColor}`)
            .build();
    }
}
```

### **2. Data Visualization**
```typescript
export class ChartComponent {
    getBarStyle(percentage: number, color: string): string {
        return `width: ${percentage}%; background-color: ${color}; height: 20px;`;
    }
}
```

### **3. Conditional Styling**
```typescript
export class StatusComponent {
    getStatusStyle(status: string): string {
        const colors = { active: 'green', inactive: 'gray', error: 'red' };
        return `color: ${colors[status]}; font-weight: bold;`;
    }
}
```

---

## ⚠️ **Security Warnings**

1. **No JavaScript** - Không bao giờ include JavaScript trong styles
2. **Validate Input** - Luôn validate style strings trước khi apply
3. **Sanitize Content** - Remove dangerous CSS properties
4. **Trusted Sources** - Chỉ sử dụng với styles từ trusted sources
5. **CSP Compliance** - Ensure styles comply với Content Security Policy

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
