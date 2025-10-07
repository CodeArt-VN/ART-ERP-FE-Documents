# Pipes Documentation - ART-ERP-FE

## 📋 **Mục đích tài liệu**

Thư mục này chứa tài liệu hướng dẫn sử dụng chi tiết cho tất cả pipes trong `src/app/pipes/`. Pipes được sử dụng để transform data trong templates một cách hiệu quả và tái sử dụng.

---

## 📊 **Thống kê Pipes**

- **Tổng số pipes**: 10 pipes
- **Đã document**: 10 pipes  
- **Coverage**: 100% pipes có trong project

---

## 📂 **Danh sách Pipes đã document**

### **🔒 Security Pipes (3 pipes)**
- **[SafeFrame](./safe-frame.md)** - Bypass security cho iframe URLs
- **[SafeHtml](./safe-html.md)** - Bypass security cho HTML content
- **[SafeStyle](./safe-style.md)** - Bypass security cho CSS styles

### **🔍 Filter & Search Pipes (4 pipes)**
- **[Filter](./filter.md)** - Filter arrays theo multiple conditions
- **[Search](./search.md)** - Search arrays với string matching
- **[SearchNoAccent](./search-no-accent.md)** - Search arrays bỏ qua dấu tiếng Việt
- **[IsNotDeleted](./is-not-deleted.md)** - Filter items chưa bị xóa

### **📅 Format Pipes (2 pipes)**
- **[DateFriendly](./date-friendly.md)** - Format date thành friendly format với real-time update
- **[NumberFriendly](./number-friendly.md)** - Format numbers thành friendly format

### **🛠️ Utility Pipes (1 pipe)**
- **[MyPipe](./my-pipe.md)** - Convert string to lowercase (example pipe)

---

## 🚀 **Quick Start Guide**

### **1. Import Pipes Module**
```typescript
// Import pipes module
import { PipesModule } from 'src/app/pipes/pipes.module';

@NgModule({
    imports: [
        CommonModule,
        PipesModule  // Add this
    ],
    // ...
})
export class YourModule {}
```

### **2. Basic Usage Patterns**
```html
<!-- Security pipes -->
<iframe [src]="url | safeFrame"></iframe>
<div [innerHTML]="htmlContent | safeHtml"></div>
<div [style]="cssStyle | safeStyle"></div>

<!-- Filter pipes -->
<div *ngFor="let item of items | filter: filterConditions">{{ item.name }}</div>
<div *ngFor="let item of items | search: searchConditions">{{ item.name }}</div>
<div *ngFor="let item of items | searchNoAccent: searchConditions">{{ item.name }}</div>
<div *ngFor="let item of items | isNotDeleted">{{ item.name }}</div>

<!-- Format pipes -->
<span>{{ date | dateFriendly | async }}</span>
<span>{{ amount | numberFriendly }}</span>

<!-- Utility pipes -->
<span>{{ text | myPipe }}</span>
```

### **3. Common Combinations**
```html
<!-- Filter + Search combination -->
<div *ngFor="let item of items | filter: statusFilter | searchNoAccent: searchTerm">
    <h3>{{ item.name }}</h3>
    <p>{{ item.description }}</p>
    <small>{{ item.createdDate | dateFriendly | async }}</small>
    <span class="amount">{{ item.amount | numberFriendly }}</span>
</div>

<!-- Security + Format combination -->
<div [innerHTML]="content | safeHtml"></div>
<div class="timestamp">{{ lastUpdate | dateFriendly | async }}</div>
```

---

## 📋 **Pipe Categories**

### **Security Pipes**
Pipes để bypass Angular security restrictions một cách an toàn cho trusted content.

### **Filter & Search Pipes**
Pipes để filter và search arrays trong templates, hỗ trợ multiple conditions và Vietnamese text.

### **Format Pipes**
Pipes để format data thành human-readable format, bao gồm dates và numbers.

### **Utility Pipes**
General-purpose pipes cho các transformations đơn giản.

---

## 🎯 **Best Practices**

### **1. Performance Considerations**
```typescript
// ✅ Đúng - Sử dụng pure pipes khi có thể
@Pipe({ name: 'myPipe', pure: true })

// ⚠️ Cẩn thận - Impure pipes có thể impact performance
@Pipe({ name: 'filter', pure: false })  // Chỉ dùng khi cần thiết
```

### **2. Security**
```html
<!-- ✅ Đúng - Chỉ sử dụng safe pipes với trusted content -->
<div [innerHTML]="trustedHtml | safeHtml"></div>

<!-- ❌ Sai - Không sử dụng với user input trực tiếp -->
<div [innerHTML]="userInput | safeHtml"></div>  <!-- XSS risk -->
```

### **3. Filter Performance**
```typescript
// ✅ Đúng - Filter trong component thay vì template cho large datasets
export class MyComponent {
    filteredItems: any[] = [];
    
    filterItems() {
        this.filteredItems = this.items.filter(item => item.status === 'active');
    }
}
```

### **4. Async Pipes**
```html
<!-- ✅ Đúng - Sử dụng async pipe cho Observables -->
<span>{{ date | dateFriendly | async }}</span>

<!-- ❌ Sai - Subscribe trong component không cần thiết -->
<span>{{ friendlyDate }}</span>  <!-- Nếu đã subscribe trong component -->
```

---

## 🔧 **Development Guidelines**

### **Creating Custom Pipes**
```typescript
@Pipe({
    name: 'customPipe',
    pure: true,  // Set false nếu cần detect changes trong objects/arrays
    standalone: false
})
export class CustomPipe implements PipeTransform {
    transform(value: any, ...args: any[]): any {
        // Transformation logic
        return transformedValue;
    }
}
```

### **Testing Pipes**
```typescript
describe('CustomPipe', () => {
    let pipe: CustomPipe;
    
    beforeEach(() => {
        pipe = new CustomPipe();
    });
    
    it('should transform value correctly', () => {
        const result = pipe.transform('input', 'arg1', 'arg2');
        expect(result).toBe('expected output');
    });
});
```

### **Performance Tips**
1. **Pure vs Impure**: Sử dụng pure pipes khi có thể
2. **Memoization**: Cache expensive calculations
3. **Limit Usage**: Tránh sử dụng quá nhiều pipes trong một template
4. **Component Logic**: Move complex logic to component methods

---

## 🚨 **Common Pitfalls**

### **1. Impure Pipe Overuse**
```html
<!-- ❌ Sai - Impure pipe chạy mỗi change detection cycle -->
<div *ngFor="let item of largeArray | filter: complexConditions">
```

### **2. Security Bypass Misuse**
```html
<!-- ❌ Sai - Bypass security với untrusted content -->
<div [innerHTML]="userGeneratedContent | safeHtml"></div>
```

### **3. Memory Leaks**
```typescript
// ❌ Sai - Không unsubscribe Observable pipes
// DateFriendlyPipe tự động handle subscription, nhưng custom pipes cần cẩn thận
```

---

## 📚 **Related Documentation**

- **[Global Functions](../js-lib.md)** - Utility functions used by pipes
- **[Components](../components/README.md)** - Components that use these pipes
- **[Services](../services.md)** - Services for data transformation

---

**Tài liệu này sẽ được cập nhật khi có pipes mới hoặc thay đổi.**

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
