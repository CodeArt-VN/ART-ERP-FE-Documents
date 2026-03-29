# ScrollbarThemeDirective Documentation

## 📄 **Tổng quan**

`ScrollbarThemeDirective` là directive để apply custom scrollbar styling cho elements. Directive này inject CSS để customize scrollbar appearance theo theme của ứng dụng, chỉ khi `environment.showScrollbar` được enable.

## 🎯 **Mục đích sử dụng**

- **Custom Scrollbar Styling**: Apply theme-based scrollbar styles
- **Conditional Application**: Chỉ apply khi showScrollbar enabled
- **Shadow DOM Injection**: Inject CSS vào shadow DOM
- **Theme Integration**: Sử dụng CSS variables từ theme system
- **Cross-browser Support**: Webkit scrollbar styling

## 📍 **File location**
```
src/app/directives/scrollbar-theme.directive.ts
```

## 🔧 **Directive Configuration**

```typescript
@Directive({
    selector: '[appScrollbarTheme]',
    standalone: false,
})
```

## 🏗️ **Implementation**

### **Environment Check**
```typescript
if (environment.showScrollbar) {
    // Apply scrollbar styling
}
```

### **CSS Rules Applied**
```css
.scroll-y::-webkit-scrollbar {
    width: 6px;
}

.scroll-y::-webkit-scrollbar-track {
    background: var(--menu-right-border-color);
    background: transparent;
}

::-webkit-scrollbar-corner {
    background: var(--menu-right-border-color);
}

.scroll-y::-webkit-scrollbar-thumb {
    border-radius: 0px;
    background: var(--ion-color-primary);
    background: linear-gradient(var(--menu-background-top), var(--main-background-top));
    border: 1px solid rgba(var(--ion-color-primary-rgb,0,0,0),.25);
}

.scroll-y::-webkit-scrollbar-thumb:hover {
    background: linear-gradient(var(--ion-color-primary), var(--ion-color-primary));
}
```

### **Horizontal Scroll Support**
```css
/* Applied when element has 'scrollx' class */
.inner-scroll {
    overflow-x: auto !important;
}
```

## 🎨 **Usage Examples**

### **1. Basic Vertical Scrollbar**
```html
<!-- Apply custom scrollbar to vertical scroll container -->
<div class="scroll-y" appScrollbarTheme>
    <div class="content">
        <p>Long content that requires scrolling...</p>
        <!-- More content -->
    </div>
</div>
```

### **2. Horizontal Scrollbar**
```html
<!-- Apply custom scrollbar with horizontal scroll support -->
<div class="scroll-y scrollx" appScrollbarTheme>
    <div class="inner-scroll">
        <table style="width: 2000px;">
            <!-- Wide table content -->
        </table>
    </div>
</div>
```

### **3. Data Table with Custom Scrollbar**
```html
<!-- Custom scrollbar for data table -->
<div class="table-container scroll-y" appScrollbarTheme>
    <app-data-table>
        <!-- Table content -->
    </app-data-table>
</div>
```

### **4. Modal with Themed Scrollbar**
```html
<!-- Modal with custom scrollbar -->
<ion-modal>
    <ion-content class="scroll-y" appScrollbarTheme>
        <div class="modal-content">
            <!-- Long modal content -->
        </div>
    </ion-content>
</ion-modal>
```

### **5. List with Custom Scrollbar**
```html
<!-- List container with themed scrollbar -->
<div class="list-wrapper scroll-y" appScrollbarTheme>
    <ion-list>
        <ion-item *ngFor="let item of items">
            <ion-label>{{ item.name }}</ion-label>
        </ion-item>
    </ion-list>
</div>
```

## 🎨 **CSS Variables Used**

### **Theme Variables**
```css
--menu-right-border-color    /* Track background */
--menu-background-top        /* Gradient start */
--main-background-top        /* Gradient end */
--ion-color-primary          /* Thumb color */
--ion-color-primary-rgb      /* Thumb border */
```

### **Custom Properties**
- **Scrollbar width**: 6px
- **Border radius**: 0px (square corners)
- **Hover effect**: Solid primary color gradient

## 🔧 **Configuration**

### **Environment Setting**
```typescript
// environment.ts
export const environment = {
    showScrollbar: true,  // Enable custom scrollbar
    // ... other settings
};

// environment.prod.ts  
export const environment = {
    showScrollbar: false, // Disable in production
    // ... other settings
};
```

### **Required CSS Classes**
```css
/* Required class for vertical scrolling */
.scroll-y {
    overflow-y: auto;
}

/* Optional class for horizontal scrolling */
.scrollx {
    /* Enables horizontal scroll support */
}

/* Inner content for horizontal scroll */
.inner-scroll {
    /* Container for horizontally scrollable content */
}
```

## 📋 **Best Practices**

### **1. Use with Scroll-Y Class**
```html
<!-- ✅ Đúng - Use with scroll-y class -->
<div class="scroll-y" appScrollbarTheme>
    <div class="content">
        <!-- Scrollable content -->
    </div>
</div>

<!-- ❌ Sai - Missing scroll-y class -->
<div appScrollbarTheme>
    <div class="content">
        <!-- Scrollbar styling won't apply -->
    </div>
</div>
```

### **2. Environment-Based Usage**
```typescript
// ✅ Check environment before applying
@Component({
    template: `
        <div class="scroll-y" 
             [appScrollbarTheme]="showCustomScrollbar">
            <!-- Content -->
        </div>
    `
})
export class MyComponent {
    showCustomScrollbar = environment.showScrollbar;
}
```

### **3. Combine with Horizontal Scroll**
```html
<!-- ✅ Proper horizontal scroll setup -->
<div class="scroll-y scrollx" appScrollbarTheme>
    <div class="inner-scroll">
        <div style="width: 1500px;">
            <!-- Wide content -->
        </div>
    </div>
</div>
```

## 🎯 **Common Use Cases**

### **1. Data Tables**
```html
<div class="data-table-wrapper scroll-y" appScrollbarTheme>
    <table class="data-table">
        <thead>
            <tr>
                <th>Column 1</th>
                <th>Column 2</th>
            </tr>
        </thead>
        <tbody>
            <tr *ngFor="let row of data">
                <td>{{ row.col1 }}</td>
                <td>{{ row.col2 }}</td>
            </tr>
        </tbody>
    </table>
</div>
```

### **2. Sidebar Navigation**
```html
<div class="sidebar scroll-y" appScrollbarTheme>
    <nav class="navigation">
        <a *ngFor="let item of menuItems" 
           [routerLink]="item.route">
            {{ item.label }}
        </a>
    </nav>
</div>
```

### **3. Content Areas**
```html
<div class="content-area scroll-y" appScrollbarTheme>
    <article>
        <h1>Article Title</h1>
        <p>Long article content...</p>
    </article>
</div>
```

### **4. Form Containers**
```html
<form class="form-container scroll-y" appScrollbarTheme>
    <div class="form-sections">
        <div class="form-section" *ngFor="let section of formSections">
            <!-- Form fields -->
        </div>
    </div>
</form>
```

## 🚨 **Browser Support**

### **Webkit Browsers**
- **Chrome**: Full support
- **Safari**: Full support
- **Edge**: Full support (Chromium-based)

### **Non-Webkit Browsers**
- **Firefox**: Limited support (uses different scrollbar API)
- **IE**: No support

### **Fallback Behavior**
```css
/* Fallback for non-webkit browsers */
.scroll-y {
    scrollbar-width: thin;
    scrollbar-color: var(--ion-color-primary) var(--menu-right-border-color);
}
```

## 🔧 **Integration Examples**

### **With Ionic Components**
```html
<ion-content class="scroll-y" appScrollbarTheme>
    <ion-list>
        <ion-item *ngFor="let item of items">
            <ion-label>{{ item.name }}</ion-label>
        </ion-item>
    </ion-list>
</ion-content>
```

### **With Angular Material**
```html
<mat-sidenav-content class="scroll-y" appScrollbarTheme>
    <div class="content">
        <!-- Material content -->
    </div>
</mat-sidenav-content>
```

### **With Custom Components**
```typescript
@Component({
    selector: 'app-scrollable-container',
    template: `
        <div class="container scroll-y" appScrollbarTheme>
            <ng-content></ng-content>
        </div>
    `,
    styles: [`
        .container {
            height: 400px;
            overflow-y: auto;
        }
    `]
})
export class ScrollableContainerComponent {}
```

## 🎨 **Theme Customization**

### **CSS Variable Override**
```css
/* Custom theme variables */
:root {
    --scrollbar-width: 8px;
    --scrollbar-thumb-color: #007bff;
    --scrollbar-track-color: #f1f1f1;
}

/* Override directive styles */
.custom-scrollbar.scroll-y::-webkit-scrollbar {
    width: var(--scrollbar-width);
}

.custom-scrollbar.scroll-y::-webkit-scrollbar-thumb {
    background: var(--scrollbar-thumb-color);
}
```

### **Dynamic Theme Support**
```typescript
@Component({
    template: `
        <div class="scroll-y" 
             appScrollbarTheme
             [style.--scrollbar-thumb-color]="themeColor">
            <!-- Content -->
        </div>
    `
})
export class ThemedScrollComponent {
    themeColor = '#007bff';
    
    changeTheme(color: string) {
        this.themeColor = color;
    }
}
```

## 📱 **Mobile Considerations**

### **Touch Devices**
- Custom scrollbars may not appear on touch devices
- Native scrolling behavior is preserved
- Directive has no negative impact on mobile performance

### **Responsive Design**
```css
/* Hide custom scrollbar on mobile */
@media (max-width: 768px) {
    .scroll-y::-webkit-scrollbar {
        display: none;
    }
}
```

ScrollbarThemeDirective cung cấp elegant solution để customize scrollbar appearance theo theme system, enhancing user experience trên desktop browsers với conditional application based on environment settings.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
