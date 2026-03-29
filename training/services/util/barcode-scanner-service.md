# BarcodeScannerService Documentation

## 📄 **Tổng quan**

`BarcodeScannerService` là service chuyên dụng để quét mã vạch và QR code trong ứng dụng mobile ART-ERP. Service này sử dụng Capacitor Barcode Scanner plugin để truy cập camera thiết bị và xử lý các loại mã khác nhau.

## 🎯 **Mục đích sử dụng**

- **Quét mã vạch**: Sản phẩm, tài sản, kho hàng
- **Quét QR code**: Đơn hàng, thanh toán, thông tin liên hệ
- **Xử lý VietQR**: Mã QR thanh toán Việt Nam
- **Validation**: Kiểm tra tính hợp lệ của mã quét được

## 📍 **File location**
```
src/app/services/util/barcode-scanner.service.ts
```

## 🔧 **Dependencies**

```typescript
import { CapacitorBarcodeScanner, CapacitorBarcodeScannerOptions, CapacitorBarcodeScannerScanResult } from '@capacitor/barcode-scanner';
import { Capacitor } from '@capacitor/core';
import { EnvService } from '../core/env.service';
import { lib } from '../static/global-functions';
```

## 🏗️ **Properties**

### **Scanner Options**
```typescript
options: CapacitorBarcodeScannerOptions = {
    hint: 17,                    // Barcode format hint
    scanInstructions: '',        // Hướng dẫn quét
    scanOrientation: 3           // ADAPTIVE orientation
}
```

**Orientation Values:**
- `1`: PORTRAIT
- `2`: LANDSCAPE  
- `3`: ADAPTIVE (tự động)

## 🔨 **Methods**

### **1. Quét mã cơ bản**

#### **scan(type?, scanInstructions?)**
Quét mã vạch hoặc QR code với xử lý theo loại.

```typescript
scan(type: string = null, scanInstructions: string = null): Promise<string>
```

**Parameters:**
- `type` (optional): Loại mã cần quét ('SO', 'PO', etc.)
- `scanInstructions` (optional): Hướng dẫn hiển thị cho user

**Returns:** Promise với kết quả quét được

**Example:**
```typescript
// Quét mã đơn giản
this.barcodeService.scan().then(result => {
    console.log('Scanned code:', result);
}).catch(error => {
    console.error('Scan failed:', error);
});

// Quét mã đơn hàng với validation
this.barcodeService.scan('SO', 'Quét mã QR đơn hàng').then(orderCode => {
    console.log('Order code:', orderCode);
    this.loadOrder(orderCode);
}).catch(error => {
    console.error('Order scan failed:', error);
});
```

### **2. Xử lý kết quả quét**

#### **handleBarcodeScanResult(type, scanResult, scanInstructions?)**
Xử lý và validate kết quả quét theo loại mã.

```typescript
handleBarcodeScanResult(type: string, scanResult: CapacitorBarcodeScannerScanResult, scanInstructions?: string): Promise<string>
```

**Parameters:**
- `type`: Loại mã ('SO', 'PO', etc.)
- `scanResult`: Kết quả từ scanner
- `scanInstructions` (optional): Hướng dẫn cho user

**Returns:** Promise với mã đã được xử lý

**Example:**
```typescript
// Xử lý kết quả quét đơn hàng
this.barcodeService.handleBarcodeScanResult('SO', scanResult).then(orderCode => {
    // orderCode đã được extract từ QR code
    this.processOrder(orderCode);
});
```

### **3. Error handling**

#### **handleError(error)**
Xử lý và hiển thị lỗi quét mã một cách user-friendly.

```typescript
handleError(error: any): void
```

**Error Types Handled:**
- **Permission denied**: Không có quyền truy cập camera
- **Camera unavailable**: Camera không khả dụng
- **Process cancelled**: User hủy quét
- **Platform unavailable**: Không hỗ trợ trên platform hiện tại

## 🎨 **Usage Examples**

### **1. Quét mã sản phẩm trong kho**
```typescript
export class InventoryPage {
    constructor(private barcodeService: BarcodeScannerService) {}

    async scanProduct() {
        try {
            const productCode = await this.barcodeService.scan();
            
            // Tìm sản phẩm theo mã
            const product = await this.productService.getByCode(productCode);
            
            if (product) {
                this.selectedProduct = product;
                this.updateQuantity();
            } else {
                this.env.showMessage('Không tìm thấy sản phẩm', 'warning');
            }
        } catch (error) {
            console.error('Product scan failed:', error);
        }
    }

    async scanMultipleProducts() {
        const scannedProducts = [];
        
        while (true) {
            try {
                const code = await this.barcodeService.scan();
                scannedProducts.push(code);
                
                // Hỏi user có muốn tiếp tục không
                const continueScanning = await this.env.showPrompt(
                    'Đã quét được sản phẩm. Tiếp tục quét?',
                    'Quét thêm sản phẩm',
                    null,
                    'Tiếp tục',
                    'Hoàn thành'
                );
                
                if (!continueScanning) break;
            } catch (error) {
                break; // User cancelled or error occurred
            }
        }
        
        this.processScannedProducts(scannedProducts);
    }
}
```

### **2. Quét QR thanh toán VietQR**
```typescript
export class PaymentPage {
    constructor(private barcodeService: BarcodeScannerService) {}

    async scanPaymentQR() {
        try {
            const qrCode = await this.barcodeService.scan();
            
            // Kiểm tra nếu là VietQR
            if (qrCode.startsWith('000201')) {
                const qrContent = lib.readVietQRCode(qrCode);
                
                this.paymentInfo = {
                    bankAccount: qrContent.bankAccount,
                    amount: qrContent.amount,
                    message: qrContent.message,
                    bankCode: qrContent.bankCode
                };
                
                this.showPaymentConfirmation();
            } else {
                this.env.showMessage('Mã QR không hợp lệ', 'warning');
            }
        } catch (error) {
            console.error('Payment QR scan failed:', error);
        }
    }
}
```

### **3. Quét mã đơn hàng với validation**
```typescript
export class OrderScanPage {
    constructor(private barcodeService: BarcodeScannerService) {}

    async scanOrderCode() {
        try {
            // Quét với type 'SO' để tự động validate
            const orderCode = await this.barcodeService.scan('SO', 'Quét mã QR đơn hàng');
            
            // Load thông tin đơn hàng
            const order = await this.orderService.getByCode(orderCode);
            
            if (order) {
                this.currentOrder = order;
                this.loadOrderDetails();
            } else {
                this.env.showMessage('Không tìm thấy đơn hàng', 'warning');
            }
        } catch (error) {
            if (!error.message?.includes('cancelled')) {
                this.env.showMessage('Lỗi quét mã đơn hàng', 'danger');
            }
        }
    }

    async scanOrderWithRetry() {
        let attempts = 0;
        const maxAttempts = 3;
        
        while (attempts < maxAttempts) {
            try {
                const orderCode = await this.barcodeService.scan('SO');
                return orderCode;
            } catch (error) {
                attempts++;
                
                if (attempts >= maxAttempts) {
                    this.env.showMessage('Đã thử quét 3 lần. Vui lòng nhập mã thủ công.', 'warning');
                    break;
                }
                
                // Hiển thị thông báo thử lại
                await this.env.showAlert(
                    `Quét không thành công (lần ${attempts}/${maxAttempts}). Thử lại?`,
                    'Quét lại mã QR'
                );
            }
        }
        
        return null;
    }
}
```

### **4. Quét mã với custom validation**
```typescript
export class AssetManagementPage {
    constructor(private barcodeService: BarcodeScannerService) {}

    async scanAssetTag() {
        try {
            const assetCode = await this.barcodeService.scan();
            
            // Custom validation cho mã tài sản
            if (!this.isValidAssetCode(assetCode)) {
                this.env.showMessage('Mã tài sản không đúng định dạng', 'warning');
                return;
            }
            
            // Kiểm tra tài sản có tồn tại không
            const asset = await this.assetService.getByCode(assetCode);
            
            if (asset) {
                this.processAsset(asset);
            } else {
                // Hỏi user có muốn tạo tài sản mới không
                const createNew = await this.env.showPrompt(
                    'Tài sản chưa tồn tại. Tạo mới?',
                    'Tài sản mới',
                    null,
                    'Tạo mới',
                    'Hủy'
                );
                
                if (createNew) {
                    this.createNewAsset(assetCode);
                }
            }
        } catch (error) {
            console.error('Asset scan failed:', error);
        }
    }

    private isValidAssetCode(code: string): boolean {
        // Mã tài sản phải có format: AS-YYYY-NNNN
        const pattern = /^AS-\d{4}-\d{4}$/;
        return pattern.test(code);
    }
}
```

### **5. Batch scanning với progress tracking**
```typescript
export class BatchScanPage {
    scannedItems: string[] = [];
    scanProgress = 0;
    
    constructor(private barcodeService: BarcodeScannerService) {}

    async startBatchScan(targetCount: number) {
        this.scannedItems = [];
        this.scanProgress = 0;
        
        for (let i = 0; i < targetCount; i++) {
            try {
                const code = await this.barcodeService.scan(
                    null, 
                    `Quét item ${i + 1}/${targetCount}`
                );
                
                // Kiểm tra trùng lặp
                if (!this.scannedItems.includes(code)) {
                    this.scannedItems.push(code);
                    this.scanProgress = ((i + 1) / targetCount) * 100;
                    
                    // Update UI
                    this.updateScanProgress();
                } else {
                    this.env.showMessage('Mã đã được quét trước đó', 'warning');
                    i--; // Quét lại
                }
            } catch (error) {
                // User cancelled or error
                break;
            }
        }
        
        this.completeBatchScan();
    }

    private updateScanProgress() {
        // Update progress bar và danh sách items
        console.log(`Progress: ${this.scanProgress}%`);
        console.log('Scanned items:', this.scannedItems);
    }

    private completeBatchScan() {
        this.env.showMessage(
            `Hoàn thành quét ${this.scannedItems.length} items`, 
            'success'
        );
        
        // Process scanned items
        this.processScannedItems(this.scannedItems);
    }
}
```

## 🔒 **Platform Support**

### **Mobile Only**
```typescript
// Service chỉ hoạt động trên mobile platforms
if (Capacitor.getPlatform() == 'web') {
    this.env.showMessage('Barcode scanner is not available on this platform.', 'warning');
    return;
}
```

### **Permission Handling**
```typescript
// Service tự động xử lý camera permissions
// Hiển thị thông báo user-friendly khi bị từ chối quyền
```

## 🎯 **Supported Barcode Types**

### **QR Code Processing**
- **VietQR**: Mã QR thanh toán Việt Nam
- **Order QR**: Mã QR đơn hàng (format: O:orderCode)
- **Generic QR**: Các loại QR code khác

### **Barcode Formats**
- **Code 128**: Mã vạch sản phẩm
- **Code 39**: Mã tài sản, thiết bị
- **EAN/UPC**: Mã vạch quốc tế
- **Data Matrix**: Mã 2D cho thiết bị nhỏ

## 📋 **Best Practices**

### **1. Luôn handle errors gracefully**
```typescript
try {
    const code = await this.barcodeService.scan();
    // Process code
} catch (error) {
    if (!error.message?.includes('cancelled')) {
        // Only show error if not user cancellation
        this.env.showMessage('Quét mã không thành công', 'warning');
    }
}
```

### **2. Provide clear instructions**
```typescript
// Cung cấp hướng dẫn rõ ràng cho user
const code = await this.barcodeService.scan('SO', 'Đưa camera gần mã QR đơn hàng');
```

### **3. Validate scanned codes**
```typescript
// Luôn validate mã sau khi quét
const code = await this.barcodeService.scan();
if (this.isValidCode(code)) {
    this.processCode(code);
} else {
    this.env.showMessage('Mã không hợp lệ', 'warning');
}
```

### **4. Handle duplicate scans**
```typescript
// Tránh quét trùng lặp trong batch operations
if (!this.scannedCodes.includes(code)) {
    this.scannedCodes.push(code);
} else {
    this.env.showMessage('Mã đã được quét', 'warning');
}
```

## 🚨 **Error Scenarios**

### **Common Errors**
```typescript
// Permission denied
"Permission denied. Please enable camera access."

// Camera not available  
"Camera not available. Please check your camera settings."

// Platform not supported
"Barcode scanner is not available on this platform."

// Process cancelled
"Couldn't scan because the process was cancelled."
```

### **Error Recovery**
```typescript
async scanWithFallback() {
    try {
        return await this.barcodeService.scan();
    } catch (error) {
        // Fallback to manual input
        return await this.env.showPrompt(
            'Quét mã không thành công. Nhập mã thủ công:',
            'Nhập mã',
            null,
            'OK',
            'Hủy',
            [{ name: 'code', placeholder: 'Nhập mã...' }]
        );
    }
}
```

## 🔧 **Integration với Components**

### **Với Input Control**
```typescript
// Trong input-control component
async openBarcodeScanner() {
    try {
        const code = await this.barcodeService.scan();
        this.form.get(this.field.Name)?.setValue(code);
    } catch (error) {
        // Handle error
    }
}
```

### **Với List Pages**
```typescript
// Trong list page
async quickScan() {
    try {
        const code = await this.barcodeService.scan();
        this.query.Code = code;
        this.loadData();
    } catch (error) {
        // Handle error
    }
}
```

BarcodeScannerService cung cấp giải pháp quét mã hoàn chỉnh và dễ sử dụng cho ứng dụng mobile ART-ERP, hỗ trợ đầy đủ các loại mã vạch và QR code phổ biến.
