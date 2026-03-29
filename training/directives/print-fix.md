# PrintFixDirective Documentation

## 📄 **Tổng quan**

`PrintFixDirective` là directive để fix CSS issues khi in (print) các elements có scrollable content. Directive này inject CSS rules vào shadow DOM để đảm bảo content hiển thị đúng khi in.

## 🎯 **Mục đích sử dụng**

- **Print CSS Fix**: Fix scrollable elements khi in
- **Shadow DOM Injection**: Inject CSS vào shadow DOM
- **Print Media Query**: Sử dụng @media print để apply styles chỉ khi in
- **Overflow Fix**: Fix overflow issues trong print mode

## 📍 **File location**
```
src/app/directives/print-fix.directive.ts
```

## 🔧 **Directive Configuration**

```typescript
@Directive({
    selector: '[appPrintFix]',
    standalone: false,
})
```

## 🏗️ **Implementation**

### **CSS Rules Injected**
```css
@media print {
    .inner-scroll {
        position: inherit !important; 
        overflow: hidden !important;
    }
}
```

### **Shadow DOM Injection Logic**
```typescript
constructor(el: ElementRef) {
    const stylesheet = `@media print {.inner-scroll {position: inherit !important; overflow: hidden !important;}}`;
    const styleElmt = el.nativeElement.shadowRoot.querySelector('style');

    if (styleElmt) {
        styleElmt.append(stylesheet);
    } else {
        const barStyle = document.createElement('style');
        barStyle.append(stylesheet);
        el.nativeElement.shadowRoot.appendChild(barStyle);
    }
}
```

## 🎨 **Usage Examples**

### **1. Basic Usage**
```html
<!-- Apply print fix to scrollable container -->
<div class="scroll-container" appPrintFix>
    <div class="inner-scroll">
        <p>Long content that might cause print issues...</p>
        <p>More content...</p>
    </div>
</div>
```

### **2. Data Table Print Fix**
```html
<!-- Fix data table printing issues -->
<div class="data-table-container" appPrintFix>
    <table class="inner-scroll">
        <thead>
            <tr>
                <th>Column 1</th>
                <th>Column 2</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Data 1</td>
                <td>Data 2</td>
            </tr>
        </tbody>
    </table>
</div>
```

### **3. Modal Print Fix**
```html
<!-- Fix modal content printing -->
<ion-modal>
    <div class="modal-content" appPrintFix>
        <div class="inner-scroll">
            <h2>Modal Title</h2>
            <p>Modal content that needs to print correctly...</p>
        </div>
    </div>
</ion-modal>
```

### **4. List Print Fix**
```html
<!-- Fix scrollable list printing -->
<div class="list-container" appPrintFix>
    <ion-list class="inner-scroll">
        <ion-item *ngFor="let item of items">
            <ion-label>{{ item.name }}</ion-label>
        </ion-item>
    </ion-list>
</div>
```

## 🔧 **Technical Details**

### **Shadow DOM Requirement**
- Directive yêu cầu element có shadow DOM
- CSS được inject vào shadow DOM của element
- Nếu không có shadow DOM, directive sẽ tạo mới

### **Print Media Query**
- CSS chỉ apply khi in (@media print)
- Không ảnh hưởng đến display bình thường
- Override position và overflow properties

### **CSS Properties Fixed**
- `position: inherit !important` - Reset position
- `overflow: hidden !important` - Hide overflow content

## 📋 **Best Practices**

### **1. Use on Scrollable Containers**
```html
<!-- ✅ Đúng - Apply to scrollable containers -->
<div class="scrollable-content" appPrintFix>
    <div class="inner-scroll">
        <!-- Content -->
    </div>
</div>

<!-- ❌ Sai - Don't apply to non-scrollable elements -->
<div class="static-content" appPrintFix>
    <p>Static text</p>
</div>
```

### **2. Ensure Inner-Scroll Class**
```html
<!-- ✅ Đúng - Use inner-scroll class for target elements -->
<div appPrintFix>
    <div class="inner-scroll">
        <!-- Scrollable content -->
    </div>
</div>

<!-- ❌ Sai - Missing inner-scroll class -->
<div appPrintFix>
    <div>
        <!-- Content won't be fixed -->
    </div>
</div>
```

### **3. Test Print Functionality**
```typescript
// ✅ Test print functionality
testPrint() {
    window.print();
    // Verify that scrollable content displays correctly
}
```

## 🎯 **Common Use Cases**

### **1. Data Tables**
```html
<div class="table-wrapper" appPrintFix>
    <table class="inner-scroll">
        <!-- Large table content -->
    </table>
</div>
```

### **2. Report Content**
```html
<div class="report-container" appPrintFix>
    <div class="inner-scroll">
        <h1>Report Title</h1>
        <div class="report-content">
            <!-- Long report content -->
        </div>
    </div>
</div>
```

### **3. Form Printing**
```html
<form class="print-form" appPrintFix>
    <div class="inner-scroll">
        <div class="form-section">
            <!-- Form fields -->
        </div>
    </div>
</form>
```

### **4. Modal Dialogs**
```html
<ion-modal>
    <div class="modal-wrapper" appPrintFix>
        <div class="inner-scroll">
            <ion-content>
                <!-- Modal content -->
            </ion-content>
        </div>
    </div>
</ion-modal>
```

## 🚨 **Limitations**

### **Shadow DOM Dependency**
- Requires elements with shadow DOM
- May not work with all component types
- Browser compatibility considerations

### **CSS Specificity**
- Uses !important declarations
- May conflict with other print styles
- Limited to .inner-scroll class targeting

## 🔧 **Integration Examples**

### **With Data Table Component**
```html
<app-data-table appPrintFix>
    <!-- Data table content -->
</app-data-table>
```

### **With Custom Components**
```typescript
@Component({
    template: `
        <div class="component-wrapper" appPrintFix>
            <div class="inner-scroll">
                <ng-content></ng-content>
            </div>
        </div>
    `
})
export class PrintableComponent {}
```

### **With Ionic Components**
```html
<ion-content appPrintFix>
    <div class="inner-scroll">
        <ion-list>
            <!-- List items -->
        </ion-list>
    </div>
</ion-content>
```

## 📱 **Browser Support**

- **Chrome/Edge**: Full support
- **Firefox**: Full support  
- **Safari**: Full support
- **Mobile browsers**: Limited (print functionality varies)

## 🎯 **Performance Considerations**

- **Lightweight**: Minimal performance impact
- **One-time injection**: CSS injected once during construction
- **Print-only**: No runtime performance impact during normal use

PrintFixDirective cung cấp simple solution để fix common printing issues với scrollable content trong Angular applications, đặc biệt hữu ích cho reports và data tables.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
