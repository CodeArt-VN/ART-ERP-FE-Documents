# ColorPicker Component Documentation

## 📋 **Tổng quan**

`ColorPickerComponent` là component cho phép người dùng chọn màu từ palette có sẵn hoặc nhập mã màu tùy chỉnh. Component hỗ trợ các format màu phổ biến như HEX, RGB và named colors.

**Selector**: `app-color-picker`  
**Location**: `src/app/components/controls/color-picker/color-picker.component.ts`

---

## 🚀 **Basic Usage**

### **Simple Color Picker**
```typescript
// Component
export class ThemeSettingsPage extends PageBase {
    formGroup: FormGroup;
    
    constructor(private formBuilder: FormBuilder) {
        super();
        this.formGroup = this.formBuilder.group({
            primaryColor: ['#3880ff'],
            secondaryColor: ['#0cd1e8'],
            accentColor: ['#f4f5f8']
        });
    }
    
    onColorChange(color: string, field: string) {
        this.formGroup.get(field).setValue(color);
        this.applyColorTheme();
    }
    
    applyColorTheme() {
        const colors = this.formGroup.value;
        // Apply colors to theme
        document.documentElement.style.setProperty('--ion-color-primary', colors.primaryColor);
        document.documentElement.style.setProperty('--ion-color-secondary', colors.secondaryColor);
    }
}
```

```html
<!-- Template -->
<form [formGroup]="formGroup">
    <!-- Primary Color -->
    <app-input-control [field]="{
        form: formGroup,
        id: 'primaryColor',
        type: 'color',
        label: 'Primary Color'
    }" (change)="onColorChange($event, 'primaryColor')">
    </app-input-control>
    
    <!-- Secondary Color -->
    <app-input-control [field]="{
        form: formGroup,
        id: 'secondaryColor',
        type: 'color',
        label: 'Secondary Color'
    }" (change)="onColorChange($event, 'secondaryColor')">
    </app-input-control>
</form>
```

---

## 🎨 **Color Formats**

### **Supported Formats**
```typescript
// HEX Colors
const hexColors = [
    '#FF0000',  // Red
    '#00FF00',  // Green
    '#0000FF',  // Blue
    '#FFFF00',  // Yellow
    '#FF00FF',  // Magenta
    '#00FFFF'   // Cyan
];

// RGB Colors
const rgbColors = [
    'rgb(255, 0, 0)',      // Red
    'rgb(0, 255, 0)',      // Green
    'rgb(0, 0, 255)',      // Blue
];

// Named Colors
const namedColors = [
    'red', 'green', 'blue', 'yellow',
    'orange', 'purple', 'pink', 'brown',
    'black', 'white', 'gray'
];

// Ionic Colors
const ionicColors = [
    'primary', 'secondary', 'tertiary',
    'success', 'warning', 'danger',
    'light', 'medium', 'dark'
];
```

### **Color Palette Configuration**
```typescript
export class CustomColorPalette {
    // Brand colors
    brandColors = [
        { name: 'Primary', value: '#3880ff', category: 'brand' },
        { name: 'Secondary', value: '#0cd1e8', category: 'brand' },
        { name: 'Tertiary', value: '#7044ff', category: 'brand' }
    ];
    
    // Status colors
    statusColors = [
        { name: 'Success', value: '#10dc60', category: 'status' },
        { name: 'Warning', value: '#ffce00', category: 'status' },
        { name: 'Danger', value: '#f04141', category: 'status' }
    ];
    
    // Neutral colors
    neutralColors = [
        { name: 'Light', value: '#f4f5f8', category: 'neutral' },
        { name: 'Medium', value: '#989aa2', category: 'neutral' },
        { name: 'Dark', value: '#222428', category: 'neutral' }
    ];
    
    getAllColors() {
        return [
            ...this.brandColors,
            ...this.statusColors,
            ...this.neutralColors
        ];
    }
}
```

---

## 🎛️ **Advanced Features**

### **Custom Color Picker with Categories**
```typescript
export class AdvancedColorPicker {
    colorCategories = [
        {
            name: 'Brand Colors',
            colors: [
                { name: 'Primary Blue', value: '#3880ff' },
                { name: 'Ocean Blue', value: '#0cd1e8' },
                { name: 'Royal Purple', value: '#7044ff' }
            ]
        },
        {
            name: 'Status Colors',
            colors: [
                { name: 'Success Green', value: '#10dc60' },
                { name: 'Warning Yellow', value: '#ffce00' },
                { name: 'Error Red', value: '#f04141' }
            ]
        },
        {
            name: 'Custom Colors',
            colors: [
                { name: 'Coral', value: '#ff6b6b' },
                { name: 'Mint', value: '#51cf66' },
                { name: 'Lavender', value: '#845ef7' }
            ]
        }
    ];
    
    selectedCategory = 'Brand Colors';
    customColor = '#000000';
    
    onCategoryChange(category: string) {
        this.selectedCategory = category;
    }
    
    onCustomColorChange(color: string) {
        this.customColor = color;
        this.addToCustomColors(color);
    }
    
    addToCustomColors(color: string) {
        const customCategory = this.colorCategories.find(cat => 
            cat.name === 'Custom Colors'
        );
        
        if (customCategory && !customCategory.colors.find(c => c.value === color)) {
            customCategory.colors.push({
                name: `Custom ${customCategory.colors.length + 1}`,
                value: color
            });
        }
    }
}
```

```html
<div class="color-picker-advanced">
    <!-- Category Selector -->
    <ion-segment [(ngModel)]="selectedCategory" (ionChange)="onCategoryChange($event.detail.value)">
        <ion-segment-button *ngFor="let category of colorCategories" [value]="category.name">
            {{ category.name }}
        </ion-segment-button>
    </ion-segment>
    
    <!-- Color Grid -->
    <div class="color-grid">
        <div 
            *ngFor="let color of getSelectedCategoryColors()" 
            class="color-swatch"
            [style.background-color]="color.value"
            [title]="color.name"
            (click)="selectColor(color.value)">
        </div>
    </div>
    
    <!-- Custom Color Input -->
    <div class="custom-color-section">
        <ion-item>
            <ion-label>Custom Color</ion-label>
            <input 
                type="color" 
                [(ngModel)]="customColor"
                (change)="onCustomColorChange($event.target.value)">
        </ion-item>
    </div>
</div>
```

---

## 🎨 **Color Utilities**

### **Color Conversion Functions**
```typescript
export class ColorUtils {
    // Convert HEX to RGB
    hexToRgb(hex: string): { r: number, g: number, b: number } | null {
        const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
        return result ? {
            r: parseInt(result[1], 16),
            g: parseInt(result[2], 16),
            b: parseInt(result[3], 16)
        } : null;
    }
    
    // Convert RGB to HEX
    rgbToHex(r: number, g: number, b: number): string {
        return "#" + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1);
    }
    
    // Convert HEX to HSL
    hexToHsl(hex: string): { h: number, s: number, l: number } | null {
        const rgb = this.hexToRgb(hex);
        if (!rgb) return null;
        
        const r = rgb.r / 255;
        const g = rgb.g / 255;
        const b = rgb.b / 255;
        
        const max = Math.max(r, g, b);
        const min = Math.min(r, g, b);
        let h: number, s: number;
        const l = (max + min) / 2;
        
        if (max === min) {
            h = s = 0; // achromatic
        } else {
            const d = max - min;
            s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
            
            switch (max) {
                case r: h = (g - b) / d + (g < b ? 6 : 0); break;
                case g: h = (b - r) / d + 2; break;
                case b: h = (r - g) / d + 4; break;
                default: h = 0;
            }
            h /= 6;
        }
        
        return {
            h: Math.round(h * 360),
            s: Math.round(s * 100),
            l: Math.round(l * 100)
        };
    }
    
    // Get color brightness (0-255)
    getColorBrightness(hex: string): number {
        const rgb = this.hexToRgb(hex);
        if (!rgb) return 0;
        
        // Using relative luminance formula
        return Math.round(0.299 * rgb.r + 0.587 * rgb.g + 0.114 * rgb.b);
    }
    
    // Determine if color is light or dark
    isLightColor(hex: string): boolean {
        return this.getColorBrightness(hex) > 127;
    }
    
    // Generate complementary color
    getComplementaryColor(hex: string): string {
        const rgb = this.hexToRgb(hex);
        if (!rgb) return hex;
        
        return this.rgbToHex(255 - rgb.r, 255 - rgb.g, 255 - rgb.b);
    }
    
    // Generate color variations
    generateColorVariations(baseColor: string): any {
        const hsl = this.hexToHsl(baseColor);
        if (!hsl) return {};
        
        return {
            lighter: this.hslToHex(hsl.h, hsl.s, Math.min(hsl.l + 20, 100)),
            light: this.hslToHex(hsl.h, hsl.s, Math.min(hsl.l + 10, 100)),
            base: baseColor,
            dark: this.hslToHex(hsl.h, hsl.s, Math.max(hsl.l - 10, 0)),
            darker: this.hslToHex(hsl.h, hsl.s, Math.max(hsl.l - 20, 0))
        };
    }
    
    private hslToHex(h: number, s: number, l: number): string {
        l /= 100;
        const a = s * Math.min(l, 1 - l) / 100;
        const f = (n: number) => {
            const k = (n + h / 30) % 12;
            const color = l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1);
            return Math.round(255 * color).toString(16).padStart(2, '0');
        };
        return `#${f(0)}${f(8)}${f(4)}`;
    }
}
```

---

## 🎨 **Theme Integration**

### **Dynamic Theme Colors**
```typescript
export class ThemeColorManager {
    private colorUtils = new ColorUtils();
    
    applyThemeColor(colorName: string, colorValue: string) {
        const variations = this.colorUtils.generateColorVariations(colorValue);
        
        // Apply base color
        document.documentElement.style.setProperty(`--ion-color-${colorName}`, colorValue);
        
        // Apply RGB values
        const rgb = this.colorUtils.hexToRgb(colorValue);
        if (rgb) {
            document.documentElement.style.setProperty(
                `--ion-color-${colorName}-rgb`, 
                `${rgb.r},${rgb.g},${rgb.b}`
            );
        }
        
        // Apply contrast color
        const contrastColor = this.colorUtils.isLightColor(colorValue) ? '#000000' : '#ffffff';
        document.documentElement.style.setProperty(`--ion-color-${colorName}-contrast`, contrastColor);
        
        // Apply variations
        document.documentElement.style.setProperty(`--ion-color-${colorName}-shade`, variations.dark);
        document.documentElement.style.setProperty(`--ion-color-${colorName}-tint`, variations.light);
    }
    
    resetThemeColors() {
        const defaultColors = {
            primary: '#3880ff',
            secondary: '#0cd1e8',
            tertiary: '#7044ff',
            success: '#10dc60',
            warning: '#ffce00',
            danger: '#f04141',
            light: '#f4f5f8',
            medium: '#989aa2',
            dark: '#222428'
        };
        
        Object.entries(defaultColors).forEach(([name, color]) => {
            this.applyThemeColor(name, color);
        });
    }
    
    saveThemeColors(colors: any) {
        localStorage.setItem('themeColors', JSON.stringify(colors));
    }
    
    loadThemeColors(): any {
        const saved = localStorage.getItem('themeColors');
        return saved ? JSON.parse(saved) : null;
    }
}
```

---

## 📱 **Mobile Optimization**

### **Touch-Friendly Color Picker**
```scss
// Mobile-optimized color picker
.color-picker-mobile {
    .color-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(44px, 1fr));
        gap: 8px;
        padding: 16px;
        
        .color-swatch {
            width: 44px;
            height: 44px;
            border-radius: 8px;
            border: 2px solid transparent;
            cursor: pointer;
            transition: all 0.2s ease;
            
            &:hover, &:focus {
                transform: scale(1.1);
                border-color: var(--ion-color-primary);
            }
            
            &.selected {
                border-color: var(--ion-color-primary);
                box-shadow: 0 0 0 2px rgba(var(--ion-color-primary-rgb), 0.3);
            }
        }
    }
    
    .custom-color-input {
        input[type="color"] {
            width: 100%;
            height: 44px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
        }
    }
}

@media (max-width: 768px) {
    .color-picker-mobile {
        .color-grid {
            grid-template-columns: repeat(6, 1fr);
            
            .color-swatch {
                width: 40px;
                height: 40px;
            }
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Accessibility**
```typescript
// ✅ Đúng - Accessible color picker
export class AccessibleColorPicker {
    getColorName(hex: string): string {
        const colorNames = {
            '#FF0000': 'Red',
            '#00FF00': 'Green',
            '#0000FF': 'Blue',
            '#FFFF00': 'Yellow',
            '#FF00FF': 'Magenta',
            '#00FFFF': 'Cyan',
            '#000000': 'Black',
            '#FFFFFF': 'White'
        };
        
        return colorNames[hex.toUpperCase()] || hex;
    }
    
    getContrastRatio(color1: string, color2: string): number {
        const lum1 = this.getLuminance(color1);
        const lum2 = this.getLuminance(color2);
        
        const brightest = Math.max(lum1, lum2);
        const darkest = Math.min(lum1, lum2);
        
        return (brightest + 0.05) / (darkest + 0.05);
    }
    
    private getLuminance(hex: string): number {
        const rgb = this.colorUtils.hexToRgb(hex);
        if (!rgb) return 0;
        
        const [r, g, b] = [rgb.r, rgb.g, rgb.b].map(c => {
            c = c / 255;
            return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
        });
        
        return 0.2126 * r + 0.7152 * g + 0.0722 * b;
    }
}
```

### **2. Performance**
```typescript
// ✅ Đúng - Optimized color operations
export class OptimizedColorPicker {
    private colorCache = new Map<string, any>();
    
    getCachedColorInfo(hex: string): any {
        if (this.colorCache.has(hex)) {
            return this.colorCache.get(hex);
        }
        
        const info = {
            hex,
            rgb: this.colorUtils.hexToRgb(hex),
            hsl: this.colorUtils.hexToHsl(hex),
            brightness: this.colorUtils.getColorBrightness(hex),
            isLight: this.colorUtils.isLightColor(hex)
        };
        
        this.colorCache.set(hex, info);
        return info;
    }
    
    clearColorCache() {
        this.colorCache.clear();
    }
}
```

### **3. Validation**
```typescript
// ✅ Đúng - Color validation
export class ColorValidator {
    isValidHex(hex: string): boolean {
        return /^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/.test(hex);
    }
    
    isValidRgb(rgb: string): boolean {
        return /^rgb\(\s*(\d{1,3})\s*,\s*(\d{1,3})\s*,\s*(\d{1,3})\s*\)$/.test(rgb);
    }
    
    normalizeColor(color: string): string {
        // Remove whitespace
        color = color.trim();
        
        // Convert to uppercase for hex
        if (color.startsWith('#')) {
            color = color.toUpperCase();
            
            // Convert 3-digit hex to 6-digit
            if (color.length === 4) {
                color = '#' + color[1] + color[1] + color[2] + color[2] + color[3] + color[3];
            }
        }
        
        return color;
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
