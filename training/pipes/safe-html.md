# SafeHtml Pipe Documentation

## 📋 **Tổng quan**

`SafeHtml` pipe được sử dụng để bypass Angular security restrictions cho HTML content. Pipe này sử dụng `DomSanitizer.bypassSecurityTrustHtml()` để mark HTML content as trusted.

**Pipe Name**: `safeHtml`  
**Location**: `src/app/pipes/filter.ts`  
**Type**: Pure Pipe

---

## 🔧 **Signature**

```typescript
transform(html: string): SafeHtml
```

### **Parameters**
- `html` (string): HTML content cần bypass security

### **Returns**
- `SafeHtml`: Trusted HTML content có thể render trong DOM

---

## 🚀 **Basic Usage**

### **Rich Text Content**
```typescript
// Component
export class ContentDisplayPage extends PageBase {
    articleContent = `
        <h2>Product Features</h2>
        <ul>
            <li><strong>Advanced Analytics</strong> - Real-time reporting</li>
            <li><em>User Management</em> - Role-based access control</li>
            <li><a href="/docs">Documentation</a> - Comprehensive guides</li>
        </ul>
        <p>Contact us at <a href="mailto:support@company.com">support@company.com</a></p>
    `;
    
    notificationHtml = `
        <div class="notification-content">
            <h3>System Update</h3>
            <p>New features available in <strong>version 2.1</strong></p>
        </div>
    `;
}
```

```html
<!-- Template -->
<div class="content-area">
    <!-- Article content -->
    <article [innerHTML]="articleContent | safeHtml"></article>
    
    <!-- Notification content -->
    <div class="notification" [innerHTML]="notificationHtml | safeHtml"></div>
</div>
```

---

## 🎨 **Advanced Usage**

### **Dynamic Content Loading**
```typescript
export class DynamicContentPage extends PageBase {
    htmlContent: string = '';
    
    async loadContent(contentId: string) {
        try {
            this.showLoading = true;
            const response = await this.contentService.getHtmlContent(contentId);
            
            // Sanitize content before using safeHtml
            this.htmlContent = this.sanitizeContent(response.html);
        } catch (error) {
            this.env.showErrorMessage(error);
        } finally {
            this.showLoading = false;
        }
    }
    
    private sanitizeContent(html: string): string {
        // Remove potentially dangerous elements
        const dangerousElements = ['script', 'object', 'embed', 'form'];
        let sanitized = html;
        
        dangerousElements.forEach(element => {
            const regex = new RegExp(`<${element}[^>]*>.*?<\/${element}>`, 'gi');
            sanitized = sanitized.replace(regex, '');
        });
        
        return sanitized;
    }
}
```

```html
<div class="dynamic-content">
    <ion-loading [isOpen]="showLoading" message="Loading content..."></ion-loading>
    
    <div class="content-wrapper" *ngIf="htmlContent && !showLoading">
        <div [innerHTML]="htmlContent | safeHtml"></div>
    </div>
    
    <div class="empty-state" *ngIf="!htmlContent && !showLoading">
        <p>No content available</p>
    </div>
</div>
```

### **Email Template Rendering**
```typescript
export class EmailPreviewPage extends PageBase {
    emailTemplate: string = '';
    templateVariables: any = {};
    
    async loadEmailTemplate(templateId: string) {
        try {
            const template = await this.emailService.getTemplate(templateId);
            this.emailTemplate = this.processTemplate(template.html, this.templateVariables);
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
    
    private processTemplate(template: string, variables: any): string {
        let processed = template;
        
        // Replace template variables
        Object.keys(variables).forEach(key => {
            const placeholder = `{{${key}}}`;
            processed = processed.replace(new RegExp(placeholder, 'g'), variables[key]);
        });
        
        return processed;
    }
    
    updateVariable(key: string, value: string) {
        this.templateVariables[key] = value;
        this.loadEmailTemplate(this.currentTemplateId);
    }
}
```

```html
<div class="email-preview">
    <div class="template-controls">
        <ion-item>
            <ion-label>Customer Name</ion-label>
            <ion-input 
                [(ngModel)]="templateVariables.customerName"
                (ionInput)="updateVariable('customerName', $event.detail.value)">
            </ion-input>
        </ion-item>
        
        <ion-item>
            <ion-label>Order Number</ion-label>
            <ion-input 
                [(ngModel)]="templateVariables.orderNumber"
                (ionInput)="updateVariable('orderNumber', $event.detail.value)">
            </ion-input>
        </ion-item>
    </div>
    
    <div class="email-content">
        <h3>Email Preview</h3>
        <div class="email-body" [innerHTML]="emailTemplate | safeHtml"></div>
    </div>
</div>
```

### **Rich Text Editor Integration**
```typescript
export class RichTextEditorPage extends PageBase {
    editorContent: string = '';
    previewMode: boolean = false;
    
    onEditorChange(content: string) {
        this.editorContent = content;
    }
    
    togglePreview() {
        this.previewMode = !this.previewMode;
    }
    
    saveContent() {
        if (this.validateHtmlContent(this.editorContent)) {
            this.contentService.save({
                html: this.editorContent,
                lastModified: new Date()
            });
        }
    }
    
    private validateHtmlContent(html: string): boolean {
        // Basic validation
        const parser = new DOMParser();
        const doc = parser.parseFromString(html, 'text/html');
        
        // Check for parsing errors
        const errorNode = doc.querySelector('parsererror');
        if (errorNode) {
            this.env.showMessage('Invalid HTML content', 'danger');
            return false;
        }
        
        return true;
    }
}
```

```html
<div class="rich-text-editor">
    <div class="editor-toolbar">
        <ion-button (click)="togglePreview()" fill="outline">
            {{ previewMode ? 'Edit' : 'Preview' }}
        </ion-button>
        <ion-button (click)="saveContent()" color="primary">
            Save
        </ion-button>
    </div>
    
    <div class="editor-content">
        <!-- Editor mode -->
        <div *ngIf="!previewMode" class="editor-area">
            <textarea 
                [(ngModel)]="editorContent"
                (ngModelChange)="onEditorChange($event)"
                rows="20"
                placeholder="Enter HTML content...">
            </textarea>
        </div>
        
        <!-- Preview mode -->
        <div *ngIf="previewMode" class="preview-area">
            <div [innerHTML]="editorContent | safeHtml"></div>
        </div>
    </div>
</div>
```

---

## 🔒 **Security Considerations**

### **Content Sanitization**
```typescript
export class SecureContentPage extends PageBase {
    private allowedTags = ['p', 'div', 'span', 'h1', 'h2', 'h3', 'h4', 'h5', 'h6', 
                          'ul', 'ol', 'li', 'strong', 'em', 'a', 'img', 'br'];
    
    private allowedAttributes = ['href', 'src', 'alt', 'title', 'class'];
    
    sanitizeHtml(html: string): string {
        const parser = new DOMParser();
        const doc = parser.parseFromString(html, 'text/html');
        
        // Remove disallowed elements
        const allElements = doc.querySelectorAll('*');
        allElements.forEach(element => {
            if (!this.allowedTags.includes(element.tagName.toLowerCase())) {
                element.remove();
                return;
            }
            
            // Remove disallowed attributes
            Array.from(element.attributes).forEach(attr => {
                if (!this.allowedAttributes.includes(attr.name)) {
                    element.removeAttribute(attr.name);
                }
            });
            
            // Validate href attributes
            if (element.hasAttribute('href')) {
                const href = element.getAttribute('href');
                if (!this.isValidUrl(href)) {
                    element.removeAttribute('href');
                }
            }
        });
        
        return doc.body.innerHTML;
    }
    
    private isValidUrl(url: string): boolean {
        try {
            const urlObj = new URL(url);
            return ['http:', 'https:', 'mailto:'].includes(urlObj.protocol);
        } catch {
            return false;
        }
    }
}
```

---

## 🎨 **Styling**

### **Content Styling**
```scss
.content-area {
    // Style for safe HTML content
    ::ng-deep [innerHTML] {
        h1, h2, h3, h4, h5, h6 {
            color: var(--ion-color-dark);
            margin: 16px 0 8px 0;
            font-weight: 600;
        }
        
        p {
            line-height: 1.6;
            margin: 8px 0;
            color: var(--ion-color-dark);
        }
        
        ul, ol {
            margin: 8px 0;
            padding-left: 20px;
            
            li {
                margin: 4px 0;
                line-height: 1.5;
            }
        }
        
        a {
            color: var(--ion-color-primary);
            text-decoration: none;
            
            &:hover {
                text-decoration: underline;
            }
        }
        
        strong {
            font-weight: 600;
            color: var(--ion-color-dark);
        }
        
        em {
            font-style: italic;
            color: var(--ion-color-medium-shade);
        }
        
        img {
            max-width: 100%;
            height: auto;
            border-radius: 4px;
            margin: 8px 0;
        }
    }
}

.email-preview {
    .email-body {
        border: 1px solid var(--ion-color-medium);
        border-radius: 8px;
        padding: 16px;
        background: white;
        min-height: 300px;
        
        ::ng-deep [innerHTML] {
            font-family: Arial, sans-serif;
            font-size: 14px;
            line-height: 1.6;
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Content Validation**
```typescript
// ✅ Đúng - Validate và sanitize content
loadTrustedContent(html: string) {
    const sanitized = this.sanitizeHtml(html);
    this.trustedContent = sanitized;
}

// ❌ Sai - Sử dụng raw user input
loadUserContent(userHtml: string) {
    this.content = userHtml; // XSS risk
}
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Handle malformed HTML
processHtmlContent(html: string) {
    try {
        const parser = new DOMParser();
        const doc = parser.parseFromString(html, 'text/html');
        
        if (doc.querySelector('parsererror')) {
            throw new Error('Invalid HTML');
        }
        
        return this.sanitizeHtml(html);
    } catch (error) {
        this.env.showMessage('Invalid HTML content', 'danger');
        return '';
    }
}
```

### **3. Performance**
```typescript
// ✅ Đúng - Cache processed content
private contentCache = new Map<string, string>();

getProcessedContent(contentId: string, html: string): string {
    if (this.contentCache.has(contentId)) {
        return this.contentCache.get(contentId);
    }
    
    const processed = this.sanitizeHtml(html);
    this.contentCache.set(contentId, processed);
    return processed;
}
```

---

## 🚨 **Common Use Cases**

### **1. CMS Content Display**
```typescript
export class CMSContentPage extends PageBase {
    displayArticle(articleId: string) {
        this.cmsService.getArticle(articleId).then(article => {
            this.articleHtml = this.sanitizeHtml(article.content);
        });
    }
}
```

### **2. Notification Messages**
```typescript
export class NotificationPage extends PageBase {
    showRichNotification(notification: any) {
        this.notificationHtml = this.sanitizeHtml(notification.htmlContent);
    }
}
```

### **3. Product Descriptions**
```typescript
export class ProductDetailPage extends PageBase {
    loadProductDescription(productId: string) {
        this.productService.getProduct(productId).then(product => {
            this.descriptionHtml = this.sanitizeHtml(product.description);
        });
    }
}
```

---

## ⚠️ **Security Warnings**

1. **XSS Prevention** - Luôn sanitize user-generated content
2. **Trusted Sources Only** - Chỉ sử dụng với content từ trusted sources
3. **Content Validation** - Validate HTML structure trước khi render
4. **Attribute Filtering** - Remove dangerous attributes như `onclick`, `onload`
5. **URL Validation** - Validate URLs trong `href` và `src` attributes

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
