# Directives Documentation - ART-ERP-FE

## 📋 **Mục đích tài liệu**

Thư mục này chứa tài liệu hướng dẫn sử dụng chi tiết cho tất cả directives trong `src/app/directives/`. Directives được sử dụng để extend HTML elements với custom behaviors và functionality.

---

## 📊 **Thống kê Directives**

- **Tổng số directives**: 5 directives
- **Đã document**: 5 directives  
- **Coverage**: 100% directives có trong project

---

## 📂 **Danh sách Directives đã document**

### **🌐 Internationalization Directives (1 directive)**
- **[TranslateResource](./translate-resource.md)** - Auto translate content based on language settings

### **🎨 UI Enhancement Directives (2 directives)**
- **[SvgImage](./svg-image.md)** - Load và customize SVG images dynamically
- **[ScrollbarTheme](./scrollbar-theme.md)** - Apply custom scrollbar styling

### **🖨️ Print Directives (1 directive)**
- **[PrintFix](./print-fix.md)** - Fix styling issues khi print

### **📊 Report Directives (1 directive)**
- **[RptFilter](./rpt-filter.md)** - Report filter directive (placeholder)

---

## 🚀 **Quick Start Guide**

### **1. Import Directives Module**
```typescript
// Import directives module
import { ShareDirectivesModule } from 'src/app/directives/share-directives.module';

@NgModule({
    imports: [
        CommonModule,
        ShareDirectivesModule  // Add this
    ],
    // ...
})
export class YourModule {}
```

### **2. Basic Usage Patterns**
```html
<!-- Translation directive -->
<span [appTranslateResource]="item" 
      [nameProperty]="'Name'" 
      [foreignNameProperty]="'ForeignName'">
</span>

<!-- SVG image directive -->
<img appSvgImage 
     [src]="'assets/icons/custom-icon.svg'" 
     [defaultColor]="'#3880ff'">

<!-- Scrollbar theme -->
<div appScrollbarTheme class="scroll-y">
    <!-- Scrollable content -->
</div>

<!-- Print fix -->
<ion-content appPrintFix>
    <!-- Content that needs print fixes -->
</ion-content>
```

### **3. Common Combinations**
```html
<!-- Multi-language content with custom styling -->
<div appScrollbarTheme class="scroll-y">
    <div class="item" *ngFor="let item of items">
        <img appSvgImage [src]="item.iconPath">
        <span [appTranslateResource]="item"></span>
    </div>
</div>

<!-- Print-friendly scrollable content -->
<ion-content appPrintFix appScrollbarTheme>
    <div class="content-area">
        <!-- Your content -->
    </div>
</ion-content>
```

---

## 📋 **Directive Categories**

### **Internationalization**
Directives để handle multi-language content và localization.

### **UI Enhancement**
Directives để improve user interface với custom styling và behaviors.

### **Print Support**
Directives để ensure proper formatting khi print documents.

### **Report Features**
Directives specific cho report functionality và data filtering.

---

## 🎯 **Best Practices**

### **1. Performance Considerations**
```typescript
// ✅ Đúng - Unsubscribe trong ngOnDestroy
ngOnDestroy() {
    this.subscription?.unsubscribe();
}

// ❌ Sai - Không cleanup subscriptions
// Memory leaks có thể xảy ra
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Handle null/undefined values
updateContent() {
    if (this.item && this.el?.nativeElement) {
        this.el.nativeElement.innerHTML = this.item.name;
    }
}

// ❌ Sai - Không check null values
updateContent() {
    this.el.nativeElement.innerHTML = this.item.name; // Potential error
}
```

### **3. DOM Manipulation Safety**
```typescript
// ✅ Đúng - Safe DOM access
if (this.el?.nativeElement?.shadowRoot) {
    const styleElement = this.el.nativeElement.shadowRoot.querySelector('style');
    if (styleElement) {
        styleElement.append(stylesheet);
    }
}

// ❌ Sai - Unsafe DOM access
this.el.nativeElement.shadowRoot.querySelector('style').append(stylesheet);
```

### **4. Directive Naming**
```typescript
// ✅ Đúng - Descriptive selector names
@Directive({
    selector: '[appTranslateResource]'  // Clear purpose
})

// ❌ Sai - Generic names
@Directive({
    selector: '[appDirective]'  // Too generic
})
```

---

## 🔧 **Development Guidelines**

### **Creating Custom Directives**
```typescript
@Directive({
    selector: '[appCustomDirective]',
    standalone: false
})
export class CustomDirective implements OnInit, OnDestroy {
    @Input() customProperty: string;
    
    private subscription: Subscription;
    
    constructor(
        private el: ElementRef,
        private renderer: Renderer2
    ) {}
    
    ngOnInit() {
        // Initialize directive
        this.applyCustomBehavior();
    }
    
    ngOnDestroy() {
        // Cleanup
        this.subscription?.unsubscribe();
    }
    
    private applyCustomBehavior() {
        // Directive logic
    }
}
```

### **Testing Directives**
```typescript
describe('CustomDirective', () => {
    let directive: CustomDirective;
    let fixture: ComponentFixture<TestComponent>;
    
    beforeEach(() => {
        TestBed.configureTestingModule({
            declarations: [CustomDirective, TestComponent]
        });
        
        fixture = TestBed.createComponent(TestComponent);
        directive = new CustomDirective(fixture.elementRef);
    });
    
    it('should apply custom behavior', () => {
        directive.ngOnInit();
        expect(/* test condition */).toBeTruthy();
    });
});
```

### **Module Registration**
```typescript
@NgModule({
    declarations: [
        // ... existing directives
        CustomDirective
    ],
    exports: [
        // ... existing directives
        CustomDirective
    ]
})
export class ShareDirectivesModule {}
```

---

## 🚨 **Common Pitfalls**

### **1. Memory Leaks**
```typescript
// ❌ Sai - Không unsubscribe
constructor(private env: EnvService) {
    this.env.languageTracking.subscribe(/* handler */);
}

// ✅ Đúng - Proper cleanup
ngOnDestroy() {
    this.subscription?.unsubscribe();
}
```

### **2. DOM Access Timing**
```typescript
// ❌ Sai - Access DOM trong constructor
constructor(private el: ElementRef) {
    this.el.nativeElement.innerHTML = 'content'; // Too early
}

// ✅ Đúng - Access DOM trong ngOnInit
ngOnInit() {
    this.el.nativeElement.innerHTML = 'content';
}
```

### **3. Shadow DOM Compatibility**
```typescript
// ❌ Sai - Assume regular DOM
const style = document.createElement('style');
document.head.appendChild(style);

// ✅ Đúng - Handle Shadow DOM
const shadowRoot = this.el.nativeElement.shadowRoot;
if (shadowRoot) {
    const style = document.createElement('style');
    shadowRoot.appendChild(style);
}
```

---

## 📚 **Related Documentation**

- **[Components](../components/README.md)** - Components that use these directives
- **[Services](../services/README.md)** - Services integrated with directives
- **[Global Functions](../js-lib.md)** - Utility functions used by directives

---

## 🔄 **Usage Examples**

### **Multi-language Application**
```html
<!-- Product list with translations -->
<div class="product-list" appScrollbarTheme>
    <div class="product-item" *ngFor="let product of products">
        <img appSvgImage [src]="product.iconPath">
        <h3 [appTranslateResource]="product"></h3>
        <p [appTranslateResource]="product" 
           [nameProperty]="'Description'"
           [foreignNameProperty]="'ForeignDescription'">
        </p>
    </div>
</div>
```

### **Print-friendly Reports**
```html
<!-- Report with print optimization -->
<ion-content appPrintFix appScrollbarTheme>
    <div class="report-header">
        <h1 [appTranslateResource]="reportTitle"></h1>
    </div>
    
    <div class="report-content">
        <!-- Report data -->
    </div>
</ion-content>
```

### **Custom Icon System**
```html
<!-- Dynamic SVG icons -->
<div class="icon-grid">
    <div class="icon-item" *ngFor="let icon of iconList">
        <img appSvgImage 
             [src]="'assets/icons/' + icon.name + '.svg'"
             [defaultColor]="icon.color"
             [class]="'icon-' + icon.size">
        <span [appTranslateResource]="icon"></span>
    </div>
</div>
```

---

## ⚠️ **Browser Compatibility**

### **Shadow DOM Support**
- **Chrome**: Full support
- **Firefox**: Full support  
- **Safari**: Full support
- **Edge**: Full support
- **IE**: Not supported (use polyfills if needed)

### **SVG Support**
- **Modern browsers**: Full support
- **Older browsers**: May need fallbacks

---

**Tài liệu này sẽ được cập nhật khi có directives mới hoặc thay đổi.**

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
