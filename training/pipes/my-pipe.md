# MyPipe Documentation

## 📋 **Tổng quan**

`MyPipe` là một example pipe đơn giản để convert string thành lowercase. Đây là pipe mẫu để demonstratebasic pipe implementation và có thể được sử dụng làm template cho custom pipes khác.

**Pipe Name**: `myPipe`  
**Location**: `src/app/pipes/filter.ts`  
**Type**: Pure Pipe

---

## 🔧 **Signature**

```typescript
transform(value: string, ...args): string
```

### **Parameters**
- `value` (string): String cần convert
- `...args` (any[]): Additional arguments (không được sử dụng trong implementation hiện tại)

### **Returns**
- `string`: Lowercase version của input string

---

## 🚀 **Basic Usage**

### **Simple Text Conversion**
```typescript
// Component
export class TextFormattingPage extends PageBase {
    sampleTexts = [
        'HELLO WORLD',
        'Mixed Case Text',
        'UPPERCASE TITLE',
        'CamelCaseExample'
    ];
    
    userInput = 'SAMPLE INPUT TEXT';
}
```

```html
<!-- Template -->
<div class="text-formatting">
    <h3>Text Conversion Examples</h3>
    
    <div class="example-list">
        <div class="example-item" *ngFor="let text of sampleTexts">
            <div class="original">Original: {{ text }}</div>
            <div class="converted">Lowercase: {{ text | myPipe }}</div>
        </div>
    </div>
    
    <div class="user-input-demo">
        <ion-item>
            <ion-label>Enter text:</ion-label>
            <ion-input [(ngModel)]="userInput" placeholder="Type something..."></ion-input>
        </ion-item>
        
        <div class="output">
            <p>You typed: {{ userInput }}</p>
            <p>Lowercase: {{ userInput | myPipe }}</p>
        </div>
    </div>
</div>
```

---

## 🎨 **Advanced Usage**

### **Form Field Normalization**
```typescript
export class UserRegistrationPage extends PageBase {
    registrationForm = {
        username: '',
        email: '',
        firstName: '',
        lastName: ''
    };
    
    onSubmit() {
        // Normalize data before submission
        const normalizedData = {
            username: this.registrationForm.username.toLowerCase(),
            email: this.registrationForm.email.toLowerCase(),
            firstName: this.registrationForm.firstName,
            lastName: this.registrationForm.lastName
        };
        
        this.submitRegistration(normalizedData);
    }
    
    // Alternative: Use pipe in template for display
    get displayData() {
        return {
            username: this.registrationForm.username,
            email: this.registrationForm.email,
            fullName: `${this.registrationForm.firstName} ${this.registrationForm.lastName}`
        };
    }
}
```

```html
<form class="registration-form">
    <ion-item>
        <ion-label>Username</ion-label>
        <ion-input [(ngModel)]="registrationForm.username" name="username"></ion-input>
    </ion-item>
    
    <ion-item>
        <ion-label>Email</ion-label>
        <ion-input [(ngModel)]="registrationForm.email" name="email" type="email"></ion-input>
    </ion-item>
    
    <ion-item>
        <ion-label>First Name</ion-label>
        <ion-input [(ngModel)]="registrationForm.firstName" name="firstName"></ion-input>
    </ion-item>
    
    <ion-item>
        <ion-label>Last Name</ion-label>
        <ion-input [(ngModel)]="registrationForm.lastName" name="lastName"></ion-input>
    </ion-item>
    
    <!-- Preview section -->
    <div class="preview-section">
        <h4>Preview</h4>
        <p>Username: {{ displayData.username | myPipe }}</p>
        <p>Email: {{ displayData.email | myPipe }}</p>
        <p>Full Name: {{ displayData.fullName }}</p>
    </div>
    
    <ion-button (click)="onSubmit()" expand="block">Register</ion-button>
</form>
```

### **Data Normalization for Search**
```typescript
export class SearchablePage extends PageBase {
    items = [
        { id: 1, name: 'Product A', category: 'ELECTRONICS' },
        { id: 2, name: 'Product B', category: 'CLOTHING' },
        { id: 3, name: 'Product C', category: 'BOOKS' }
    ];
    
    searchTerm = '';
    
    get filteredItems() {
        if (!this.searchTerm) return this.items;
        
        const normalizedSearch = this.searchTerm.toLowerCase();
        return this.items.filter(item => 
            item.name.toLowerCase().includes(normalizedSearch) ||
            item.category.toLowerCase().includes(normalizedSearch)
        );
    }
    
    // Alternative: Use pipe for display normalization
    normalizeForDisplay(text: string): string {
        return text.toLowerCase();
    }
}
```

```html
<div class="searchable-content">
    <ion-searchbar 
        [(ngModel)]="searchTerm"
        placeholder="Search products..."
        debounce="300">
    </ion-searchbar>
    
    <div class="results">
        <div class="item-card" *ngFor="let item of filteredItems">
            <h4>{{ item.name }}</h4>
            <p class="category">Category: {{ item.category | myPipe }}</p>
        </div>
    </div>
    
    <div class="search-info" *ngIf="searchTerm">
        <p>Searching for: "{{ searchTerm | myPipe }}"</p>
        <p>Found {{ filteredItems.length }} results</p>
    </div>
</div>
```

---

## 🔧 **Extending MyPipe**

### **Enhanced Version with Options**
```typescript
@Pipe({
    name: 'enhancedMyPipe',
    standalone: false
})
export class EnhancedMyPipe implements PipeTransform {
    transform(value: string, options?: string): string {
        if (!value) return value;
        
        switch (options) {
            case 'upper':
                return value.toUpperCase();
            case 'title':
                return this.toTitleCase(value);
            case 'camel':
                return this.toCamelCase(value);
            case 'lower':
            default:
                return value.toLowerCase();
        }
    }
    
    private toTitleCase(str: string): string {
        return str.replace(/\w\S*/g, (txt) => 
            txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase()
        );
    }
    
    private toCamelCase(str: string): string {
        return str.replace(/(?:^\w|[A-Z]|\b\w)/g, (word, index) => 
            index === 0 ? word.toLowerCase() : word.toUpperCase()
        ).replace(/\s+/g, '');
    }
}
```

### **Usage of Enhanced Version**
```html
<div class="text-transformations">
    <p>Original: {{ text }}</p>
    <p>Lowercase: {{ text | enhancedMyPipe }}</p>
    <p>Uppercase: {{ text | enhancedMyPipe:'upper' }}</p>
    <p>Title Case: {{ text | enhancedMyPipe:'title' }}</p>
    <p>Camel Case: {{ text | enhancedMyPipe:'camel' }}</p>
</div>
```

---

## 🎨 **Styling**

### **Text Transformation Display**
```scss
.text-formatting {
    .example-item {
        margin: 16px 0;
        padding: 12px;
        border: 1px solid var(--ion-color-medium);
        border-radius: 8px;
        
        .original {
            font-weight: 600;
            color: var(--ion-color-dark);
            margin-bottom: 4px;
        }
        
        .converted {
            color: var(--ion-color-primary);
            font-style: italic;
        }
    }
    
    .user-input-demo {
        margin-top: 24px;
        
        .output {
            padding: 16px;
            background: var(--ion-color-light);
            border-radius: 8px;
            margin-top: 12px;
            
            p {
                margin: 4px 0;
                
                &:first-child {
                    font-weight: 500;
                }
                
                &:last-child {
                    color: var(--ion-color-primary);
                    font-family: monospace;
                }
            }
        }
    }
}

.preview-section {
    margin: 16px 0;
    padding: 16px;
    background: var(--ion-color-light-tint);
    border-radius: 8px;
    border-left: 4px solid var(--ion-color-primary);
    
    h4 {
        margin: 0 0 12px 0;
        color: var(--ion-color-primary);
    }
    
    p {
        margin: 4px 0;
        font-family: monospace;
        font-size: 0.9rem;
    }
}
```

---

## 🔧 **Best Practices**

### **1. Input Validation**
```typescript
// ✅ Đúng - Handle edge cases
transform(value: string, ...args): string {
    if (!value || typeof value !== 'string') {
        return value; // Return original if not a string
    }
    return value.toLowerCase();
}

// ❌ Sai - No validation
transform(value: string, ...args): string {
    return value.toLowerCase(); // Will error if value is null/undefined
}
```

### **2. Performance Considerations**
```typescript
// ✅ Đúng - Pure pipe for simple transformations
@Pipe({ name: 'myPipe', pure: true })

// ❌ Sai - Impure pipe for simple transformations
@Pipe({ name: 'myPipe', pure: false }) // Unnecessary performance impact
```

### **3. Reusability**
```typescript
// ✅ Đúng - Make pipes configurable
transform(value: string, caseType: 'lower' | 'upper' = 'lower'): string {
    return caseType === 'upper' ? value.toUpperCase() : value.toLowerCase();
}
```

---

## 🚨 **Common Use Cases**

### **1. Form Data Normalization**
```typescript
export class FormPage extends PageBase {
    // Normalize user input for consistency
    normalizeInput(value: string): string {
        return value.toLowerCase().trim();
    }
}
```

### **2. Search Functionality**
```typescript
export class SearchPage extends PageBase {
    // Normalize search terms for better matching
    searchItems(term: string) {
        const normalizedTerm = term.toLowerCase();
        // Search logic here
    }
}
```

### **3. Display Formatting**
```typescript
export class DisplayPage extends PageBase {
    // Format text for consistent display
    items = [
        { name: 'ITEM ONE' },
        { name: 'Item Two' },
        { name: 'item three' }
    ];
}
```

---

## 🔄 **Creating Custom Pipes**

### **Template for New Pipes**
```typescript
@Pipe({
    name: 'customPipe',
    pure: true, // Set to false if pipe depends on external state
    standalone: false
})
export class CustomPipe implements PipeTransform {
    transform(value: any, ...args: any[]): any {
        // Input validation
        if (!value) return value;
        
        // Transformation logic
        // ...
        
        return transformedValue;
    }
}
```

### **Registration in Module**
```typescript
@NgModule({
    declarations: [
        // ... other pipes
        CustomPipe
    ],
    exports: [
        // ... other pipes
        CustomPipe
    ]
})
export class PipesModule {}
```

---

## 📚 **Learning Purpose**

`MyPipe` serves as a simple example for:
1. **Basic Pipe Structure**: Shows minimal pipe implementation
2. **Transform Method**: Demonstrates the core transform function
3. **Pure Pipe Behavior**: Example of stateless transformation
4. **Template Usage**: How to use pipes in templates
5. **Starting Point**: Base for creating more complex pipes

---

## ⚠️ **Limitations**

1. **Single Transformation**: Only converts to lowercase
2. **No Configuration**: Cannot specify transformation type
3. **String Only**: Designed for string inputs only
4. **Basic Implementation**: No advanced features or error handling

---

## 🔄 **Alternatives**

### **Built-in Angular Pipes**
```html
<!-- Use built-in pipes for common transformations -->
<p>{{ text | lowercase }}</p>
<p>{{ text | uppercase }}</p>
<p>{{ text | titlecase }}</p>
```

### **Component Methods**
```typescript
// For complex logic, use component methods
formatText(text: string): string {
    return text.toLowerCase().trim().replace(/\s+/g, ' ');
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
