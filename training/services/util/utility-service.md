# UtilityService Documentation

## 📄 **Tổng quan**

`UtilityService` là một service utility cơ bản trong ứng dụng ART-ERP. Hiện tại service này chỉ là một placeholder và chưa có implementation cụ thể. Service này có thể được mở rộng để chứa các utility functions chung cho toàn bộ ứng dụng.

## 🎯 **Mục đích sử dụng**

- **Utility functions**: Các hàm tiện ích chung
- **Helper methods**: Các method hỗ trợ xử lý dữ liệu
- **Common operations**: Các thao tác thường dùng
- **Cross-cutting concerns**: Logic dùng chung giữa các modules

## 📍 **File location**
```
src/app/services/util/utility.service.ts
```

## 🔧 **Current Implementation**

```typescript
import { Injectable } from '@angular/core';

@Injectable({
    providedIn: 'root',
})
export class UtilityService {
    constructor() {}
}
```

## 🚀 **Potential Extensions**

### **1. Data Transformation Utilities**
```typescript
export class UtilityService {
    /**
     * Convert array to lookup object
     */
    arrayToLookup<T>(array: T[], keyField: string): { [key: string]: T } {
        return array.reduce((lookup, item) => {
            lookup[item[keyField]] = item;
            return lookup;
        }, {});
    }

    /**
     * Group array by field
     */
    groupBy<T>(array: T[], keyField: string): { [key: string]: T[] } {
        return array.reduce((groups, item) => {
            const key = item[keyField];
            if (!groups[key]) {
                groups[key] = [];
            }
            groups[key].push(item);
            return groups;
        }, {});
    }

    /**
     * Deep clone object
     */
    deepClone<T>(obj: T): T {
        return JSON.parse(JSON.stringify(obj));
    }
}
```

### **2. Validation Utilities**
```typescript
export class UtilityService {
    /**
     * Check if email is valid
     */
    isValidEmail(email: string): boolean {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }

    /**
     * Check if phone number is valid (Vietnam format)
     */
    isValidPhoneVN(phone: string): boolean {
        const phoneRegex = /^(\+84|84|0)(3|5|7|8|9)[0-9]{8}$/;
        return phoneRegex.test(phone);
    }

    /**
     * Check if string is empty or whitespace
     */
    isEmptyOrWhitespace(str: string): boolean {
        return !str || str.trim().length === 0;
    }
}
```

### **3. File Utilities**
```typescript
export class UtilityService {
    /**
     * Convert file to base64
     */
    fileToBase64(file: File): Promise<string> {
        return new Promise((resolve, reject) => {
            const reader = new FileReader();
            reader.readAsDataURL(file);
            reader.onload = () => resolve(reader.result as string);
            reader.onerror = error => reject(error);
        });
    }

    /**
     * Download data as file
     */
    downloadAsFile(data: any, filename: string, type: string = 'application/json') {
        const blob = new Blob([JSON.stringify(data, null, 2)], { type });
        const url = window.URL.createObjectURL(blob);
        
        const link = document.createElement('a');
        link.href = url;
        link.download = filename;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        
        window.URL.revokeObjectURL(url);
    }

    /**
     * Get file extension
     */
    getFileExtension(filename: string): string {
        return filename.slice((filename.lastIndexOf('.') - 1 >>> 0) + 2);
    }
}
```

### **4. Date Utilities**
```typescript
export class UtilityService {
    /**
     * Format date to Vietnamese format
     */
    formatDateVN(date: Date): string {
        return date.toLocaleDateString('vi-VN');
    }

    /**
     * Get start of day
     */
    getStartOfDay(date: Date): Date {
        const result = new Date(date);
        result.setHours(0, 0, 0, 0);
        return result;
    }

    /**
     * Get end of day
     */
    getEndOfDay(date: Date): Date {
        const result = new Date(date);
        result.setHours(23, 59, 59, 999);
        return result;
    }

    /**
     * Calculate business days between dates
     */
    getBusinessDaysBetween(startDate: Date, endDate: Date): number {
        let count = 0;
        const current = new Date(startDate);
        
        while (current <= endDate) {
            const dayOfWeek = current.getDay();
            if (dayOfWeek !== 0 && dayOfWeek !== 6) { // Not Sunday or Saturday
                count++;
            }
            current.setDate(current.getDate() + 1);
        }
        
        return count;
    }
}
```

### **5. String Utilities**
```typescript
export class UtilityService {
    /**
     * Generate random string
     */
    generateRandomString(length: number): string {
        const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
        let result = '';
        for (let i = 0; i < length; i++) {
            result += chars.charAt(Math.floor(Math.random() * chars.length));
        }
        return result;
    }

    /**
     * Capitalize first letter
     */
    capitalizeFirst(str: string): string {
        return str.charAt(0).toUpperCase() + str.slice(1);
    }

    /**
     * Convert to slug
     */
    toSlug(str: string): string {
        return str
            .toLowerCase()
            .normalize('NFD')
            .replace(/[\u0300-\u036f]/g, '') // Remove accents
            .replace(/[^a-z0-9 -]/g, '') // Remove special chars
            .replace(/\s+/g, '-') // Replace spaces with -
            .replace(/-+/g, '-'); // Replace multiple - with single -
    }
}
```

## 🎨 **Usage Examples**

### **1. Sử dụng hiện tại (Empty service)**
```typescript
export class MyComponent {
    constructor(private utilityService: UtilityService) {
        // Service hiện tại chưa có methods nào
        console.log('UtilityService injected');
    }
}
```

### **2. Sử dụng sau khi mở rộng**
```typescript
export class DataProcessingComponent {
    constructor(private utilityService: UtilityService) {}

    processData(items: any[]) {
        // Group items by category
        const grouped = this.utilityService.groupBy(items, 'CategoryId');
        
        // Convert to lookup for fast access
        const lookup = this.utilityService.arrayToLookup(items, 'Id');
        
        // Clone data for modification
        const cloned = this.utilityService.deepClone(items);
        
        return { grouped, lookup, cloned };
    }

    validateUserInput(email: string, phone: string) {
        const isEmailValid = this.utilityService.isValidEmail(email);
        const isPhoneValid = this.utilityService.isValidPhoneVN(phone);
        
        return { isEmailValid, isPhoneValid };
    }

    exportData(data: any[], filename: string) {
        this.utilityService.downloadAsFile(data, `${filename}.json`);
    }
}
```

## 📋 **Recommended Extensions**

### **1. Thêm vào global-functions.ts thay vì UtilityService**
```typescript
// Nhiều utility functions đã có trong global-functions.ts
// Nên sử dụng lib object thay vì tạo service mới

import { lib } from '../static/global-functions';

// Sử dụng existing functions
const formattedDate = lib.dateFormat(new Date());
const formattedMoney = lib.formatMoney(1000000);
const uid = lib.generateUID();
```

### **2. Tạo Specialized Services**
```typescript
// Thay vì một UtilityService lớn, tạo các services chuyên biệt:

// FileUtilityService - Xử lý files
// ValidationUtilityService - Validation logic  
// DateUtilityService - Date operations
// StringUtilityService - String manipulations
```

### **3. Sử dụng Angular Pipes**
```typescript
// Nhiều utility functions nên implement như pipes
// Để sử dụng trong templates

// date-format.pipe.ts
// currency-format.pipe.ts  
// string-transform.pipe.ts
```

## 🔧 **Integration Patterns**

### **Với Global Functions**
```typescript
import { lib } from '../static/global-functions';

export class UtilityService {
    // Wrapper cho global functions
    formatDate(date: Date): string {
        return lib.dateFormat(date);
    }

    formatCurrency(amount: number): string {
        return lib.formatMoney(amount);
    }

    generateId(): string {
        return lib.generateUID();
    }
}
```

### **Với EnvService**
```typescript
export class UtilityService {
    constructor(private env: EnvService) {}

    async showProcessingMessage<T>(
        message: string, 
        operation: Promise<T>
    ): Promise<T> {
        return this.env.showLoading(message, operation);
    }

    async confirmAction(message: string): Promise<boolean> {
        try {
            await this.env.showAlert(message, 'Xác nhận');
            return true;
        } catch {
            return false;
        }
    }
}
```

## 📋 **Best Practices**

### **1. Tránh tạo God Service**
```typescript
// ❌ Không nên - Service quá lớn
export class UtilityService {
    // 50+ methods cho mọi thứ
}

// ✅ Nên - Tách thành các services nhỏ
export class DateUtilityService { }
export class FileUtilityService { }
export class ValidationUtilityService { }
```

### **2. Sử dụng existing solutions**
```typescript
// ✅ Sử dụng global-functions.ts cho utility functions
import { lib } from '../static/global-functions';

// ✅ Sử dụng EnvService cho common operations  
constructor(private env: EnvService) {}

// ✅ Sử dụng Angular built-ins khi có thể
import { DatePipe, CurrencyPipe } from '@angular/common';
```

### **3. Pure functions**
```typescript
// ✅ Utility methods nên là pure functions
export class UtilityService {
    // Pure function - không side effects
    formatText(input: string): string {
        return input.trim().toLowerCase();
    }

    // ❌ Không nên - có side effects
    formatAndSaveText(input: string): string {
        const result = input.trim().toLowerCase();
        localStorage.setItem('lastFormatted', result); // Side effect
        return result;
    }
}
```

## 🎯 **Kết luận**

`UtilityService` hiện tại là một placeholder service. Thay vì mở rộng service này, nên:

1. **Sử dụng `global-functions.ts`** cho utility functions
2. **Tạo specialized services** cho các chức năng cụ thể
3. **Sử dụng Angular pipes** cho template transformations
4. **Leverage existing services** như `EnvService` cho common operations

Service này có thể được giữ lại như một lightweight wrapper hoặc được refactor thành các services chuyên biệt hơn.
