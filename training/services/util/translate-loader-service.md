# DynamicTranslateLoaderService Documentation

## 📄 **Tổng quan**

`DynamicTranslateLoaderService` là service triển khai `TranslateLoader` interface để tải các file ngôn ngữ (i18n) một cách động. Service này hỗ trợ tải ngôn ngữ từ nhiều nguồn khác nhau: assets local, tenant server, hoặc fallback strategies.

## 🎯 **Mục đích sử dụng**

- **Multi-tenant i18n**: Tải ngôn ngữ theo từng tenant
- **Hybrid app support**: Hỗ trợ cả web và mobile platforms
- **Fallback strategy**: Tự động fallback khi tải thất bại
- **Caching integration**: Tích hợp với cache management system
- **Network-first loading**: Ưu tiên tải từ server, fallback về assets

## 📍 **File location**
```
src/app/services/util/translate-loader.service.ts
```

## 🔧 **Dependencies**

```typescript
import { HttpClient } from '@angular/common/http';
import { TranslateLoader } from '@ngx-translate/core';
import { Observable, of, throwError, from } from 'rxjs';
import { catchError, retry, timeout, switchMap, take, mergeMap } from 'rxjs/operators';
import { Capacitor } from '@capacitor/core';
import { StorageService } from '../core/storage.service';
import { CacheManagementService } from '../core/cache-management.service';
```

## 🏗️ **Properties**

### **Platform Detection**
```typescript
private readonly isHybrid: boolean = Capacitor.getPlatform() !== 'web'
```

## 🔨 **Methods**

### **1. Core Translation Loading**

#### **getTranslation(lang)**
Tải file ngôn ngữ theo strategy được cấu hình.

```typescript
getTranslation(lang: string): Observable<any>
```

**Parameters:**
- `lang`: Mã ngôn ngữ ('vi', 'en', 'cache')

**Returns:** Observable chứa translation object

**Loading Strategy:**
1. **Hybrid apps**: Luôn tải từ assets
2. **Web apps**: Tải từ tenant server, fallback về assets
3. **Cache mode**: Sử dụng ngôn ngữ đã cache

**Example:**
```typescript
// Service tự động được sử dụng bởi TranslateModule
// Không cần gọi trực tiếp trong application code

// Cấu hình trong app.module.ts
TranslateModule.forRoot({
    loader: {
        provide: TranslateLoader,
        useClass: DynamicTranslateLoaderService,
        deps: [HttpClient, CacheManagementService]
    }
})
```

### **2. Private Loading Methods**

#### **loadFromTenant(serverUrl, lang)**
Tải ngôn ngữ từ tenant server cụ thể.

```typescript
private loadFromTenant(serverUrl: string, lang: string): Observable<any>
```

**Parameters:**
- `serverUrl`: URL của tenant server
- `lang`: Mã ngôn ngữ

**Returns:** Observable với translation data

**Endpoint Format:**
```
{serverUrl}/uploads/i18n/{lang}.json
```

#### **loadFromAssets(lang)**
Tải ngôn ngữ từ assets local.

```typescript
private loadFromAssets(lang: string): Observable<any>
```

**Parameters:**
- `lang`: Mã ngôn ngữ

**Returns:** Observable với translation data

**Asset Path:**
```
./assets/i18n/{lang}.json
```

## 🎨 **Usage Examples**

### **1. Cấu hình trong App Module**
```typescript
// app.module.ts
import { TranslateModule, TranslateLoader } from '@ngx-translate/core';
import { DynamicTranslateLoaderService } from './services/util/translate-loader.service';

@NgModule({
    imports: [
        TranslateModule.forRoot({
            loader: {
                provide: TranslateLoader,
                useClass: DynamicTranslateLoaderService,
                deps: [HttpClient, CacheManagementService]
            },
            defaultLanguage: 'vi'
        })
    ]
})
export class AppModule {}
```

### **2. Sử dụng với EnvService**
```typescript
export class MyComponent {
    constructor(private env: EnvService) {}

    async changeLanguage(lang: string) {
        // EnvService sẽ sử dụng DynamicTranslateLoaderService
        await this.env.setLang(lang);
        
        // Translation sẽ được tải tự động
        const message = await this.env.translateResource('WELCOME_MESSAGE');
        console.log('Translated message:', message);
    }
}
```

### **3. Multi-tenant Language Loading**
```typescript
export class TenantService {
    constructor(
        private cacheService: CacheManagementService,
        private translateService: TranslateService
    ) {}

    async switchTenant(tenantUrl: string) {
        // Update tenant in cache
        this.cacheService.app.tenant = tenantUrl;
        
        // Reload current language from new tenant
        const currentLang = this.translateService.currentLang;
        await this.translateService.reloadLang(currentLang).toPromise();
        
        console.log(`Switched to tenant: ${tenantUrl}`);
    }
}
```

### **4. Fallback Strategy Implementation**
```typescript
export class LanguageService {
    constructor(private translateService: TranslateService) {}

    async loadLanguageWithFallback(lang: string) {
        try {
            // Try to load from tenant first
            await this.translateService.use(lang).toPromise();
            console.log(`Loaded ${lang} from tenant`);
        } catch (error) {
            console.warn(`Failed to load ${lang} from tenant, using assets`);
            // DynamicTranslateLoaderService will automatically fallback to assets
        }
    }

    async preloadLanguages(languages: string[]) {
        const loadPromises = languages.map(lang => 
            this.translateService.getTranslation(lang).toPromise()
        );
        
        try {
            await Promise.all(loadPromises);
            console.log('All languages preloaded');
        } catch (error) {
            console.warn('Some languages failed to preload:', error);
        }
    }
}
```

### **5. Cache Integration**
```typescript
export class CachedTranslationService {
    constructor(
        private translateService: TranslateService,
        private cacheService: CacheManagementService
    ) {}

    async loadCachedTranslation() {
        // Load from cache using special 'cache' language code
        await this.translateService.use('cache').toPromise();
        
        // This will use the language stored in cacheService.app.lang
        console.log('Loaded cached translation');
    }

    async refreshTranslationCache() {
        const currentLang = this.cacheService.app.lang;
        
        // Force reload from server
        await this.translateService.reloadLang(currentLang).toPromise();
        
        console.log(`Refreshed ${currentLang} translation cache`);
    }
}
```

## 🔧 **Configuration**

### **Environment Configuration**
```typescript
// environment.ts
export const environment = {
    languageStrategy: {
        networkFirst: true,        // Ưu tiên tải từ network
        cacheTimeout: 3600000,     // Cache timeout (1 hour)
        retryAttempts: 3           // Số lần retry khi thất bại
    },
    appDomain: 'https://api.company.com'  // Default tenant URL
};
```

### **Loading Strategy Configuration**
```typescript
// Hybrid apps (mobile)
isHybrid = true  → Always load from assets

// Web apps  
isHybrid = false → Load from tenant server, fallback to assets
```

## ⚡ **Performance Features**

### **1. Timeout Protection**
```typescript
// Tự động timeout sau 3 giây để tránh blocking UI
return this.http.get(endpoint).pipe(
    timeout(3000),
    catchError(error => throwError(() => error))
);
```

### **2. Automatic Fallback**
```typescript
// Tự động fallback từ tenant server về assets
return this.loadFromTenant(tenant, lang).pipe(
    catchError(error => {
        console.warn('Failed to load from tenant, falling back to assets');
        return this.loadFromAssets(lang);
    })
);
```

### **3. Cache Integration**
```typescript
// Tích hợp với CacheManagementService để tracking tenant changes
return this.storage.tracking().pipe(
    switchMap(tracking => {
        if (tracking) {
            const tenant = this.storage.app.tenant;
            return this.loadFromTenant(tenant, lang);
        }
        return this.loadFromAssets(lang);
    })
);
```

## 🚨 **Error Handling**

### **Network Errors**
```typescript
// Tự động fallback khi có lỗi network
private loadFromTenant(serverUrl: string, lang: string): Observable<any> {
    return this.http.get(endpoint).pipe(
        timeout(3000),
        catchError(error => {
            console.warn(`Failed to load from tenant: ${error.message}`);
            return throwError(() => error);
        })
    );
}
```

### **Invalid URL Handling**
```typescript
// Xử lý URL không hợp lệ
try {
    const url = new URL(serverUrl);
    const endpoint = `${url.origin}/uploads/i18n/${lang}.json`;
    // ... load from endpoint
} catch (error) {
    console.warn(`Invalid tenant URL: ${serverUrl}`);
    return throwError(() => error);
}
```

### **Asset Loading Fallback**
```typescript
// Luôn trả về empty object thay vì crash app
private loadFromAssets(lang: string): Observable<any> {
    return this.http.get(endpoint).pipe(
        catchError(error => {
            console.error(`Failed to load from assets: ${error.message}`);
            // Return empty object to prevent app crash
            return of({});
        })
    );
}
```

## 📋 **Best Practices**

### **1. Preload Critical Languages**
```typescript
// Preload các ngôn ngữ quan trọng khi app khởi động
export class AppInitService {
    async initializeApp() {
        const criticalLanguages = ['vi', 'en'];
        
        for (const lang of criticalLanguages) {
            try {
                await this.translateService.getTranslation(lang).toPromise();
            } catch (error) {
                console.warn(`Failed to preload ${lang}`);
            }
        }
    }
}
```

### **2. Handle Tenant Switching**
```typescript
// Reload translation khi switch tenant
async switchTenant(newTenant: string) {
    this.cacheService.app.tenant = newTenant;
    
    // Reload current language from new tenant
    const currentLang = this.translateService.currentLang;
    await this.translateService.reloadLang(currentLang).toPromise();
}
```

### **3. Graceful Degradation**
```typescript
// Luôn có fallback strategy
async getTranslation(key: string): Promise<string> {
    try {
        return await this.translateService.get(key).toPromise();
    } catch (error) {
        // Return key as fallback
        return key;
    }
}
```

### **4. Monitor Loading Performance**
```typescript
// Log performance metrics
async loadLanguageWithMetrics(lang: string) {
    const startTime = performance.now();
    
    try {
        await this.translateService.use(lang).toPromise();
        const loadTime = performance.now() - startTime;
        console.log(`Language ${lang} loaded in ${loadTime}ms`);
    } catch (error) {
        console.error(`Failed to load ${lang}:`, error);
    }
}
```

## 🔒 **Security Considerations**

### **URL Validation**
```typescript
// Validate tenant URL trước khi load
private isValidTenantUrl(url: string): boolean {
    try {
        const parsedUrl = new URL(url);
        return parsedUrl.protocol === 'https:' || parsedUrl.protocol === 'http:';
    } catch {
        return false;
    }
}
```

### **Content Validation**
```typescript
// Validate translation content
private validateTranslationContent(content: any): boolean {
    return content && typeof content === 'object' && !Array.isArray(content);
}
```

## 🎯 **Integration Points**

### **Với EnvService**
```typescript
// EnvService sử dụng TranslateService với DynamicTranslateLoaderService
async translateResource(resource: any): Promise<string> {
    // Translation được tải tự động qua DynamicTranslateLoaderService
    return this.translate.get(resource.code, resource.value).toPromise();
}
```

### **Với CacheManagementService**
```typescript
// Tracking tenant changes để reload translation
this.storage.tracking().subscribe(tracking => {
    if (tracking && this.storage.app.tenant) {
        // Tenant changed, reload current language
        this.reloadCurrentLanguage();
    }
});
```

DynamicTranslateLoaderService cung cấp giải pháp tải ngôn ngữ linh hoạt và mạnh mẽ cho ứng dụng multi-tenant ART-ERP, đảm bảo trải nghiệm người dùng mượt mà trong mọi tình huống.
