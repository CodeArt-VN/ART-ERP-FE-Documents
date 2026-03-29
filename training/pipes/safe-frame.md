# SafeFrame Pipe Documentation

## 📋 **Tổng quan**

`SafeFrame` pipe được sử dụng để bypass Angular security restrictions cho iframe URLs. Pipe này sử dụng `DomSanitizer.bypassSecurityTrustResourceUrl()` để mark URLs as trusted.

**Pipe Name**: `safeFrame`  
**Location**: `src/app/pipes/filter.ts`  
**Type**: Pure Pipe

---

## 🔧 **Signature**

```typescript
transform(url: string): SafeResourceUrl
```

### **Parameters**
- `url` (string): URL cần bypass security

### **Returns**
- `SafeResourceUrl`: Trusted resource URL có thể sử dụng trong iframe

---

## 🚀 **Basic Usage**

### **Iframe Embedding**
```typescript
// Component
export class DocumentViewerPage extends PageBase {
    documentUrl = 'https://docs.google.com/document/d/abc123/edit';
    pdfUrl = 'https://example.com/document.pdf';
    reportUrl = '/api/reports/generate?id=123&format=html';
}
```

```html
<!-- Template -->
<div class="document-viewer">
    <!-- Google Docs iframe -->
    <iframe 
        [src]="documentUrl | safeFrame"
        width="100%" 
        height="600px"
        frameborder="0">
    </iframe>
    
    <!-- PDF viewer -->
    <iframe 
        [src]="pdfUrl | safeFrame"
        width="100%" 
        height="800px">
    </iframe>
    
    <!-- Internal report -->
    <iframe 
        [src]="reportUrl | safeFrame"
        class="report-frame">
    </iframe>
</div>
```

---

## 🎨 **Advanced Usage**

### **Dynamic URL Loading**
```typescript
export class EmbeddedContentPage extends PageBase {
    currentUrl: string = '';
    
    loadDocument(documentId: string) {
        this.currentUrl = `/api/documents/${documentId}/preview`;
    }
    
    loadExternalContent(url: string) {
        // Validate URL before using
        if (this.isValidUrl(url)) {
            this.currentUrl = url;
        } else {
            this.env.showMessage('Invalid URL', 'danger');
        }
    }
    
    private isValidUrl(url: string): boolean {
        try {
            new URL(url);
            return true;
        } catch {
            return false;
        }
    }
}
```

```html
<div class="content-loader">
    <ion-segment [(ngModel)]="contentType" (ionChange)="onContentTypeChange()">
        <ion-segment-button value="document">
            <ion-label>Document</ion-label>
        </ion-segment-button>
        <ion-segment-button value="report">
            <ion-label>Report</ion-label>
        </ion-segment-button>
        <ion-segment-button value="external">
            <ion-label>External</ion-label>
        </ion-segment-button>
    </ion-segment>
    
    <div class="iframe-container" *ngIf="currentUrl">
        <iframe 
            [src]="currentUrl | safeFrame"
            [title]="contentTitle"
            width="100%" 
            height="600px"
            (load)="onFrameLoad()"
            (error)="onFrameError()">
        </iframe>
    </div>
</div>
```

### **Report Embedding**
```typescript
export class ReportViewerPage extends PageBase {
    reportConfig = {
        baseUrl: '/api/reports',
        defaultParams: {
            format: 'html',
            theme: 'light'
        }
    };
    
    generateReportUrl(reportId: string, params: any = {}): string {
        const queryParams = { ...this.reportConfig.defaultParams, ...params };
        const queryString = Object.keys(queryParams)
            .map(key => `${key}=${encodeURIComponent(queryParams[key])}`)
            .join('&');
            
        return `${this.reportConfig.baseUrl}/${reportId}?${queryString}`;
    }
    
    viewSalesReport(dateRange: any) {
        const url = this.generateReportUrl('sales-summary', {
            startDate: dateRange.start,
            endDate: dateRange.end,
            branchId: this.env.selectedBranch.Id
        });
        
        this.currentReportUrl = url;
    }
}
```

```html
<div class="report-viewer">
    <div class="report-controls">
        <ion-button (click)="viewSalesReport(dateRange)">
            Load Sales Report
        </ion-button>
        <ion-button (click)="viewInventoryReport()">
            Load Inventory Report
        </ion-button>
    </div>
    
    <div class="report-frame" *ngIf="currentReportUrl">
        <iframe 
            [src]="currentReportUrl | safeFrame"
            class="report-iframe"
            title="Report Viewer">
        </iframe>
    </div>
</div>
```

---

## 🔒 **Security Considerations**

### **URL Validation**
```typescript
export class SecureEmbedPage extends PageBase {
    private allowedDomains = [
        'docs.google.com',
        'drive.google.com', 
        'your-domain.com',
        'trusted-partner.com'
    ];
    
    isUrlAllowed(url: string): boolean {
        try {
            const urlObj = new URL(url);
            return this.allowedDomains.includes(urlObj.hostname);
        } catch {
            return false;
        }
    }
    
    loadTrustedContent(url: string) {
        if (this.isUrlAllowed(url)) {
            this.trustedUrl = url;
        } else {
            this.env.showAlert('URL not allowed', 'Security Warning', 'This URL is not in the allowed domains list.');
        }
    }
}
```

### **Content Security Policy**
```html
<!-- Add CSP headers in index.html -->
<meta http-equiv="Content-Security-Policy" 
      content="frame-src 'self' https://docs.google.com https://drive.google.com;">
```

---

## 🎨 **Styling**

### **Responsive Iframe**
```scss
.iframe-container {
    position: relative;
    width: 100%;
    height: 0;
    padding-bottom: 56.25%; // 16:9 aspect ratio
    
    iframe {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        border: none;
        border-radius: 8px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    }
}

.document-viewer {
    .report-iframe {
        width: 100%;
        min-height: 600px;
        border: 1px solid var(--ion-color-medium);
        border-radius: 4px;
        
        @media (max-width: 768px) {
            min-height: 400px;
        }
    }
}

.loading-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(255, 255, 255, 0.8);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 10;
    
    &.hidden {
        display: none;
    }
}
```

---

## 🔧 **Best Practices**

### **1. URL Validation**
```typescript
// ✅ Đúng - Validate URLs before using
validateAndSetUrl(url: string) {
    if (this.isValidUrl(url) && this.isUrlAllowed(url)) {
        this.frameUrl = url;
    } else {
        this.handleInvalidUrl(url);
    }
}

// ❌ Sai - Sử dụng URL trực tiếp từ user input
setUrl(userInput: string) {
    this.frameUrl = userInput; // Security risk
}
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Handle iframe load errors
onFrameLoad() {
    this.isLoading = false;
    this.hasError = false;
}

onFrameError() {
    this.isLoading = false;
    this.hasError = true;
    this.env.showMessage('Failed to load content', 'danger');
}
```

### **3. Loading States**
```html
<!-- ✅ Đúng - Show loading state -->
<div class="iframe-wrapper">
    <div class="loading-overlay" [class.hidden]="!isLoading">
        <ion-spinner></ion-spinner>
        <span>Loading content...</span>
    </div>
    
    <iframe 
        [src]="url | safeFrame"
        (load)="onFrameLoad()"
        (error)="onFrameError()">
    </iframe>
</div>
```

---

## 🚨 **Common Use Cases**

### **1. Document Preview**
```typescript
export class DocumentPreviewPage extends PageBase {
    previewDocument(documentId: string) {
        const previewUrl = `/api/documents/${documentId}/preview`;
        this.documentUrl = previewUrl;
    }
}
```

### **2. External Integration**
```typescript
export class IntegrationPage extends PageBase {
    loadGoogleDocs(docId: string) {
        const url = `https://docs.google.com/document/d/${docId}/edit`;
        this.externalUrl = url;
    }
}
```

### **3. Report Dashboard**
```typescript
export class ReportDashboardPage extends PageBase {
    loadDashboard(dashboardId: string) {
        const url = `/reports/dashboard/${dashboardId}`;
        this.dashboardUrl = url;
    }
}
```

---

## ⚠️ **Security Warnings**

1. **Chỉ sử dụng với trusted URLs** - Không bypass security cho user-generated URLs
2. **Validate domains** - Chỉ allow URLs từ trusted domains
3. **CSP headers** - Set up Content Security Policy properly
4. **HTTPS only** - Chỉ sử dụng HTTPS URLs trong production
5. **Input sanitization** - Validate và sanitize URL inputs

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
