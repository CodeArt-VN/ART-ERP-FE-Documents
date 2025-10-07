# PrintingService Documentation

## 📄 **Tổng quan**

`PrintingService` là service chuyên dụng để xử lý in ấn trong ứng dụng ART-ERP. Service này tích hợp với QZ Tray để kết nối và điều khiển máy in, hỗ trợ in nhiều loại nội dung (HTML, PDF, Image) với các tùy chọn in linh hoạt.

## 🎯 **Mục đích sử dụng**

- **In tài liệu**: Hóa đơn, báo cáo, nhãn, mã vạch
- **Quản lý máy in**: Kết nối, cấu hình, lấy danh sách máy in
- **In hàng loạt**: Xử lý nhiều job in cùng lúc
- **In từ xa**: Hỗ trợ in qua mạng với QZ Tray server

## 📍 **File location**
```
src/app/services/util/printing.service.ts
```

## 🔧 **Dependencies**

```typescript
import * as qz from 'qz-tray';
import { KJUR, KEYUTIL, stob64, hextorstr } from 'jsrsasign';
import { EnvService } from '../core/env.service';
import { SYS_ConfigProvider, SYS_PrinterProvider } from '../static/services.service';
import { SYS_ConfigService } from '../custom/system-config.service';
```

## 📊 **Interfaces**

### **printData Interface**
```typescript
interface printData {
    content: string;           // Nội dung cần in
    type: 'html' | 'pdf' | 'image';  // Loại nội dung
    options?: printOptions[];  // Tùy chọn in
    IDJob?: string;           // ID của job in
}
```

### **printOptions Interface**
```typescript
interface printOptions {
    printer?: string;         // Tên máy in
    host?: string;           // Host của QZ server
    port?: string;           // Port của QZ server
    isSecure?: boolean;      // Sử dụng HTTPS
    jobName?: string;        // Tên job in
    tray?: string;          // Khay giấy
    pages?: string;         // Trang cần in
    copies?: number;        // Số bản copy
    paperSize?: string;     // Kích thước giấy
    rotation?: number;      // Góc xoay
    scale?: number;         // Tỷ lệ phóng to/thu nhỏ
    duplex?: string;        // In 2 mặt
    orientation?: string;   // Hướng in
    cssStyle?: string;      // CSS tùy chỉnh
    autoStyle?: Element;    // Element tự động style
}
```

## 🏗️ **Properties**

### **Public Properties**
```typescript
public tracking: Subject<any>     // Subject theo dõi trạng thái in
isReady: boolean                  // Trạng thái sẵn sàng của service
printingServerConfig: any         // Cấu hình server in
```

## 🔨 **Methods**

### **1. Quản lý kết nối**

#### **getAvailablePrinters(branchId?)**
Lấy danh sách máy in khả dụng cho chi nhánh.

```typescript
getAvailablePrinters(branchId = null): Promise<{printers: string[], config: any}>
```

**Parameters:**
- `branchId` (optional): ID chi nhánh, null = chi nhánh hiện tại

**Returns:** Promise với danh sách máy in và config

**Example:**
```typescript
// Lấy máy in cho chi nhánh hiện tại
this.printingService.getAvailablePrinters().then(result => {
    console.log('Available printers:', result.printers);
    console.log('Config:', result.config);
});

// Lấy máy in cho chi nhánh cụ thể
this.printingService.getAvailablePrinters('BRANCH001').then(result => {
    console.log('Printers for BRANCH001:', result.printers);
});
```

#### **startConnection()**
Khởi tạo kết nối với QZ Tray server.

```typescript
startConnection(): Promise<void>
```

**Example:**
```typescript
this.printingService.startConnection().then(() => {
    console.log('Connected to QZ Tray');
}).catch(error => {
    console.error('Connection failed:', error);
});
```

### **2. Chức năng in**

#### **print(jobs)**
In hàng loạt với tối ưu hóa theo server và máy in.

```typescript
print(jobs: printData[]): Promise<any[]>
```

**Parameters:**
- `jobs`: Mảng các job in cần thực hiện

**Returns:** Promise với kết quả tất cả job in

**Example:**
```typescript
const printJobs: printData[] = [
    {
        content: '<h1>Invoice #001</h1><p>Total: $100</p>',
        type: 'html',
        options: [{
            printer: 'HP LaserJet',
            copies: 2,
            paperSize: 'A4'
        }]
    },
    {
        content: 'base64-pdf-content',
        type: 'pdf',
        options: [{
            printer: 'Canon Printer',
            orientation: 'landscape'
        }]
    }
];

this.printingService.print(printJobs).then(results => {
    console.log('Print results:', results);
}).catch(error => {
    console.error('Print failed:', error);
});
```

#### **printHTML(content, options)**
In nội dung HTML.

```typescript
printHTML(content: string, options?: printOptions): Promise<any>
```

**Example:**
```typescript
const htmlContent = `
    <div style="text-align: center;">
        <h2>Hóa đơn bán hàng</h2>
        <p>Số HĐ: HD001</p>
        <p>Tổng tiền: 500,000 VNĐ</p>
    </div>
`;

this.printingService.printHTML(htmlContent, {
    printer: 'Thermal Printer',
    paperSize: '80mm',
    copies: 1
}).then(() => {
    console.log('HTML printed successfully');
});
```

#### **printPDF(base64Content, options)**
In file PDF từ base64 content.

```typescript
printPDF(base64Content: string, options?: printOptions): Promise<any>
```

**Example:**
```typescript
this.printingService.printPDF(pdfBase64, {
    printer: 'Office Printer',
    pages: '1-3',
    duplex: 'long-edge'
}).then(() => {
    console.log('PDF printed successfully');
});
```

#### **printImage(base64Content, options)**
In hình ảnh từ base64 content.

```typescript
printImage(base64Content: string, options?: printOptions): Promise<any>
```

**Example:**
```typescript
this.printingService.printImage(imageBase64, {
    printer: 'Photo Printer',
    paperSize: '4x6',
    orientation: 'landscape'
}).then(() => {
    console.log('Image printed successfully');
});
```

### **3. Cấu hình và quản lý**

#### **getConfig(branchId?)**
Lấy cấu hình in cho chi nhánh.

```typescript
getConfig(branchId?: string): Promise<any>
```

**Example:**
```typescript
this.printingService.getConfig().then(config => {
    console.log('Printing config:', config);
});
```

#### **initQZ()**
Khởi tạo QZ Tray với certificate và signature.

```typescript
initQZ(): void
```

## 🎨 **Usage Examples**

### **1. In hóa đơn đơn giản**
```typescript
export class InvoicePage {
    constructor(private printingService: PrintingService) {}

    async printInvoice(invoice: any) {
        const htmlContent = this.generateInvoiceHTML(invoice);
        
        try {
            await this.printingService.printHTML(htmlContent, {
                printer: 'Invoice Printer',
                paperSize: 'A4',
                copies: 2
            });
            
            console.log('Invoice printed successfully');
        } catch (error) {
            console.error('Print failed:', error);
        }
    }

    private generateInvoiceHTML(invoice: any): string {
        return `
            <div class="invoice">
                <h2>HÓA ĐƠN BÁN HÀNG</h2>
                <p>Số: ${invoice.Code}</p>
                <p>Ngày: ${invoice.Date}</p>
                <p>Khách hàng: ${invoice.CustomerName}</p>
                <p>Tổng tiền: ${invoice.TotalAmount}</p>
            </div>
        `;
    }
}
```

### **2. In hàng loạt với nhiều máy in**
```typescript
export class BulkPrintService {
    constructor(private printingService: PrintingService) {}

    async printMultipleDocuments(documents: any[]) {
        const printJobs: printData[] = documents.map(doc => ({
            content: this.generateDocumentHTML(doc),
            type: 'html',
            options: [{
                printer: doc.printerName,
                copies: doc.copies || 1,
                paperSize: doc.paperSize || 'A4'
            }],
            IDJob: doc.id
        }));

        try {
            const results = await this.printingService.print(printJobs);
            console.log('Bulk print completed:', results);
            return results;
        } catch (error) {
            console.error('Bulk print failed:', error);
            throw error;
        }
    }
}
```

### **3. Kiểm tra máy in khả dụng**
```typescript
export class PrinterManagementPage {
    printers: string[] = [];
    
    constructor(private printingService: PrintingService) {}

    async loadAvailablePrinters() {
        try {
            const result = await this.printingService.getAvailablePrinters();
            this.printers = result.printers;
            console.log('Available printers:', this.printers);
        } catch (error) {
            console.error('Failed to load printers:', error);
        }
    }

    async testPrint(printerName: string) {
        const testContent = '<h1>Test Print</h1><p>Printer test successful!</p>';
        
        try {
            await this.printingService.printHTML(testContent, {
                printer: printerName
            });
            console.log(`Test print successful on ${printerName}`);
        } catch (error) {
            console.error(`Test print failed on ${printerName}:`, error);
        }
    }
}
```

### **4. In với tracking và progress**
```typescript
export class PrintTrackingService {
    constructor(private printingService: PrintingService) {
        // Subscribe to print tracking events
        this.printingService.tracking.subscribe(event => {
            console.log('Print event:', event);
            this.handlePrintEvent(event);
        });
    }

    private handlePrintEvent(event: any) {
        switch (event.type) {
            case 'started':
                console.log(`Print job ${event.jobId} started`);
                break;
            case 'completed':
                console.log(`Print job ${event.jobId} completed`);
                break;
            case 'error':
                console.error(`Print job ${event.jobId} failed:`, event.error);
                break;
        }
    }
}
```

## 🔒 **Security Features**

### **Certificate-based Authentication**
```typescript
// QZ Tray sử dụng certificate để xác thực
initQZ() {
    // Certificate và private key được cấu hình trong service
    // Đảm bảo kết nối an toàn với QZ Tray server
}
```

### **Secure Connection**
```typescript
// Hỗ trợ HTTPS cho kết nối từ xa
const options: printOptions = {
    host: 'print-server.company.com',
    port: '8443',
    isSecure: true  // Sử dụng HTTPS
};
```

## ⚡ **Performance Features**

### **Batch Processing**
- Nhóm các job in theo server và máy in
- Tối ưu hóa số lượng kết nối
- Xử lý song song khi có thể

### **Connection Pooling**
- Tái sử dụng kết nối QZ Tray
- Tự động reconnect khi mất kết nối
- Cache cấu hình máy in

## 🚨 **Error Handling**

### **Common Error Scenarios**
```typescript
try {
    await this.printingService.print(jobs);
} catch (error) {
    if (error.message.includes('QZ Tray not running')) {
        // QZ Tray chưa được khởi động
        this.showQZTrayInstallationGuide();
    } else if (error.message.includes('Printer not found')) {
        // Máy in không tồn tại
        this.showPrinterSelectionDialog();
    } else if (error.message.includes('Permission denied')) {
        // Không có quyền in
        this.showPermissionError();
    } else {
        // Lỗi khác
        this.showGenericError(error);
    }
}
```

## 📋 **Best Practices**

### **1. Kiểm tra kết nối trước khi in**
```typescript
async safePrint(content: string) {
    try {
        await this.printingService.startConnection();
        await this.printingService.printHTML(content);
    } catch (error) {
        console.error('Print failed:', error);
    }
}
```

### **2. Sử dụng default printer từ config**
```typescript
// Không cần chỉ định printer cụ thể
// Service sẽ sử dụng default printer từ cấu hình chi nhánh
await this.printingService.printHTML(content);
```

### **3. Batch printing cho hiệu suất tốt**
```typescript
// Thay vì in từng document riêng lẻ
// Gom nhóm thành batch để tối ưu performance
const jobs = documents.map(doc => ({
    content: doc.html,
    type: 'html',
    options: [{ printer: doc.printer }]
}));

await this.printingService.print(jobs);
```

### **4. Handle print events**
```typescript
// Subscribe to tracking events để theo dõi tiến trình
this.printingService.tracking.subscribe(event => {
    // Update UI based on print status
    this.updatePrintProgress(event);
});
```

## 🔧 **Configuration**

### **System Configuration**
```typescript
// Cấu hình trong SYS_Config
{
    PrintingHost: 'localhost',      // QZ Tray host
    PrintingPort: 8181,            // QZ Tray port
    PrintingIsSecure: false,       // Use HTTPS
    DefaultPrinter: 'HP_LaserJet'  // Default printer code
}
```

### **Branch-specific Configuration**
```typescript
// Mỗi chi nhánh có thể có cấu hình máy in riêng
// Service tự động load config theo chi nhánh hiện tại
```

## 🎯 **Integration với Components**

### **Với List Pages**
```typescript
// Trong list-toolbar component
async exportAndPrint() {
    const data = await this.exportData();
    const htmlContent = this.generateReportHTML(data);
    
    await this.printingService.printHTML(htmlContent, {
        paperSize: 'A4',
        orientation: 'landscape'
    });
}
```

### **Với Detail Pages**
```typescript
// Trong detail page
async printDocument() {
    const htmlContent = this.generateDocumentHTML(this.item);
    
    await this.printingService.printHTML(htmlContent, {
        copies: this.printCopies,
        printer: this.selectedPrinter
    });
}
```

PrintingService cung cấp giải pháp in ấn hoàn chỉnh và mạnh mẽ cho ứng dụng ART-ERP, hỗ trợ đầy đủ các tính năng từ in đơn giản đến in hàng loạt phức tạp.
