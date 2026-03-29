# Source Code Architecture - ART-ERP-FE

## 📋 **Mục đích tài liệu**

Tài liệu này ghi lại các quy tắc cơ bản về kiến trúc source code của dự án ART-ERP-FE, phục vụ cho việc training nhân viên mới và AI Assistant.

---

## 🏗️ **Framework và Thư viện chính**

### **Core Framework**
- **Angular 20.0.0** - Framework chính cho frontend
- **Ionic 8.0.0** - Framework UI cho mobile/web hybrid app
- **Capacitor 7.0.0** - Native bridge cho iOS/Android

### **Thư viện UI/UX**
- **@ionic/angular** - Ionic components cho Angular
- **ionicons** - Icon library
- **@ng-select/ng-select** - Advanced select component
- **angular-gridster2** - Dashboard grid layout
- **@fullcalendar/angular** - Calendar component
- **swiper** - Touch slider component

### **Thư viện chức năng**
- **@ngx-translate/core** - Internationalization (i18n)
- **@microsoft/signalr** - Real-time communication
- **firebase** - Push notifications và analytics
- **rxjs** - Reactive programming
- **qrcode** - QR code generation
- **intro.js** - User onboarding tours

### **Thư viện bảo mật**
- **jsrsasign** - JWT và crypto operations
- **sha** - Hash functions

### **Development Tools**
- **@angular/service-worker** - PWA support
- **eslint** - Code linting
- **prettier** - Code formatting
- **typescript** - Type safety

---

## 📁 **Cấu trúc thư mục cốt lõi**

```
src/
├── app/
│   ├── components/          # Shared components
│   ├── directives/          # Custom directives
│   ├── guards/              # Route guards (AuthGuard)
│   ├── interfaces/          # TypeScript interfaces
│   ├── modals/              # Modal components
│   ├── models/              # Data models
│   ├── pages/               # Feature modules (submodules)
│   │   ├── ADMIN/           # Admin module (git submodule)
│   │   ├── POS/             # Point of Sale (git submodule)
│   │   ├── SALE/            # Sales module (git submodule)
│   │   ├── HRM/             # Human Resources (git submodule)
│   │   ├── CRM/             # Customer Relations (git submodule)
│   │   ├── ACCOUNTANT/      # Accounting (git submodule)
│   │   ├── FINANCIAL/       # Financial (git submodule)
│   │   ├── WMS/             # Warehouse Management (git submodule)
│   │   ├── PURCHASE/        # Purchase (git submodule)
│   │   ├── PROD/            # Production (git submodule)
│   │   ├── SHIP/            # Shipping (git submodule)
│   │   ├── BI/              # Business Intelligence (git submodule)
│   │   ├── APPROVAL/        # Approval workflows (git submodule)
│   │   ├── OST/             # Operations Support (git submodule)
│   │   ├── PM/              # Project Management (git submodule)
│   │   └── SYS/             # System utilities
│   ├── pipes/               # Custom pipes
│   ├── services/            # Business services
│   │   ├── core/            # Core services (EnvService, CommonService)
│   │   ├── auth/            # Authentication services
│   │   ├── util/            # Utility services
│   │   └── static/          # Static configurations
│   ├── app.component.ts     # Root component
│   ├── app.module.ts        # Root module
│   ├── app-routing.module.ts # Main routing
│   ├── page-base.ts         # Base class cho pages
│   └── share.module.ts      # Shared module
├── assets/                  # Static assets (images, icons)
├── environments/            # Environment configurations
├── theme/                   # SCSS styling
└── global.scss             # Global styles
```

### **Đặc điểm kiến trúc**

1. **Modular Architecture**: Mỗi module nghiệp vụ là git submodule riêng biệt
2. **Shared Components**: Components dùng chung trong `src/app/components/`
3. **Service Layer**: Tách biệt business logic vào services
4. **Page Base Class**: Tất cả pages extend từ `PageBase` class
5. **Environment-based Configuration**: Cấu hình theo môi trường (dev/prod)

---

## 🏷️ **Quy tắc đặt tên**

### **Files và Folders**
```typescript
// ✅ Đúng
user-profile.component.ts
sale-order.service.ts
pos-payment-modal.page.ts

// ❌ Sai
UserProfile.component.ts
saleorder.service.ts
POSPaymentModal.page.ts
```

### **Classes và Interfaces**
```typescript
// ✅ Đúng - PascalCase
export class UserProfileComponent { }
export class SaleOrderService { }
export interface UserProfile { }

// ❌ Sai
export class userProfileComponent { }
export class sale_order_service { }
```

### **Variables và Functions**
```typescript
// ✅ Đúng - camelCase
const userName = 'admin';
const isLoggedIn = true;
function getUserProfile() { }

// ❌ Sai
const user_name = 'admin';
const IsLoggedIn = true;
function get_user_profile() { }
```

### **Constants**
```typescript
// ✅ Đúng - UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com';
const MAX_RETRY_COUNT = 3;

// ❌ Sai
const apiBaseUrl = 'https://api.example.com';
const maxRetryCount = 3;
```

### **Component Selectors**
```typescript
// ✅ Đúng - kebab-case với prefix 'app'
@Component({
  selector: 'app-user-profile',
  // ...
})

// ❌ Sai
@Component({
  selector: 'UserProfile',
  // ...
})
```

### **Component Class Suffixes**
```typescript
// ✅ Đúng - Sử dụng 'Page' hoặc 'Component'
export class UserProfilePage { }
export class HeaderComponent { }

// ❌ Sai
export class UserProfile { }
export class Header { }
```

---

## 🔧 **Nguyên tắc Code**

### **1. Console Logging Rules**
```typescript
// ✅ Đúng - Sử dụng dog check cho dev logs
import { dog } from 'src/environments/environment';

dog && console.log('User data:', userData);
dog && console.warn('API response:', response);
dog && console.error('Error occurred:', error);

// ❌ Sai - Log trực tiếp
console.log('User data:', userData);
console.warn('API response:', response);
```

### **2. EnvService Usage Rules**
```typescript
// ✅ Đúng - Sử dụng EnvService methods
constructor(private env: EnvService) {}

// Message & Notification
this.env.showMessage('SUCCESS_MESSAGE', 'success');
this.env.showErrorMessage(error);
this.env.showAlert('ALERT_MESSAGE', 'SubHeader', 'Header');

// Translation
const translatedText = await this.env.translateResource('MESSAGE_KEY');

// Storage & Cookie
const value = await this.env.getStorage('key');
await this.env.setStorage('key', value);

// ❌ Sai - Tự viết hàm mới
private showCustomMessage(message: string) {
    // Don't do this - use env.showMessage instead
}
```

### **3. Service Architecture**
```typescript
// ✅ Đúng - Extend từ base service
export class UserService extends ExtendService {
    constructor() {
        super();
        this.apiPath = {
            method: 'GET',
            url: function () { return ApiSetting.apiDomain('CRM/Contact/User') }
        };
    }
}

// ❌ Sai - Tạo service riêng biệt
export class UserService {
    // Don't create standalone services
}
```

### **4. Page Base Class Usage**
```typescript
// ✅ Đúng - Extend từ PageBase
export class UserProfilePage extends PageBase {
    constructor(
        public pageProvider: UserService,
        public env: EnvService,
        // ... other dependencies
    ) {
        super();
    }
}

// ❌ Sai - Không extend PageBase
export class UserProfilePage {
    // Missing base functionality
}
```

---

## 🔒 **Nguyên tắc bảo mật**

### **1. Authentication Flow**
```typescript
// Token-based authentication với JWT
// Flow: Login → Token → UserContext → AuthGuard
```

### **2. Authorization Patterns**
```typescript
// ✅ Đúng - Kiểm tra permission
const hasPermission = await this.env.checkFormPermission('/function/code');
if (hasPermission) {
    // Thực hiện action
}

// ❌ Sai - Không kiểm tra permission
// Thực hiện action trực tiếp
```

### **3. Data Security**
```typescript
// ✅ Đúng - Sử dụng HTTPS và Bearer token
headers: {
    'Authorization': 'Bearer ' + token,
    'Content-Type': 'application/json'
}

// ❌ Sai - Không có authentication header
headers: {
    'Content-Type': 'application/json'
}
```

### **4. Input Validation**
```typescript
// ✅ Đúng - Validate input
if (!data || !data.Id) {
    this.env.showMessage('INVALID_DATA', 'danger');
    return;
}

// ❌ Sai - Không validate
// Sử dụng data trực tiếp
```

### **5. Error Handling**
```typescript
// ✅ Đúng - Handle errors properly
try {
    const result = await this.service.save(data);
    this.env.showMessage('SAVE_SUCCESS', 'success');
} catch (error) {
    this.env.showErrorMessage(error);
}

// ❌ Sai - Không handle errors
const result = await this.service.save(data);
```

---

## 📊 **Service Layer Architecture**

### **Core Services**
1. **EnvService** - Environment và utility functions
2. **CommonService** - HTTP communication layer
3. **CacheManagementService** - Data caching
4. **StorageService** - Local storage management
5. **AuthenticationService** - User authentication
6. **UserContextService** - User session management

### **Provider Services Pattern**
```typescript
// Tất cả business services extend từ ExtendService
export class BusinessService extends ExtendService {
    // Cấu hình API path
    // Implement CRUD operations
    // Handle caching
}
```

---

## 🎨 **Styling Guidelines**

### **SCSS Structure**
```scss
// Variables first
$primary-color: #3880ff;
$secondary-color: #0cd1e8;

// Mixins
@mixin button-style {
    border-radius: 4px;
    padding: 8px 16px;
}

// Component styles
.user-profile {
    &__header {
        background: $primary-color;
    }
    
    &__content {
        padding: 16px;
    }
}
```

### **CSS Class Naming (BEM)**
```scss
// ✅ Đúng - BEM methodology
.user-profile { }
.user-profile__header { }
.user-profile__content { }
.user-profile--active { }

// ❌ Sai
.userProfile { }
.user_profile_header { }
```

---

## 🚀 **Performance Guidelines**

### **1. Lazy Loading**
```typescript
// ✅ Đúng - Lazy load modules
{
    path: 'user',
    loadChildren: () => import('./user/user.module').then(m => m.UserModule)
}
```

### **2. OnPush Change Detection**
```typescript
// ✅ Đúng - Sử dụng OnPush khi có thể
@Component({
    changeDetection: ChangeDetectionStrategy.OnPush
})
```

### **3. Caching Strategy**
```typescript
// ✅ Đúng - Enable cache cho data ít thay đổi
this.cacheConfig = {
    cache: true,
    ttl: 300000 // 5 minutes
};
```

---

## 📱 **Mobile-First Guidelines**

### **1. Responsive Design**
```scss
// Mobile first approach
.component {
    // Mobile styles (default)
    
    @media (min-width: 768px) {
        // Tablet styles
    }
    
    @media (min-width: 1024px) {
        // Desktop styles
    }
}
```

### **2. Touch-Friendly UI**
```scss
// Minimum touch target size
.button {
    min-height: 44px;
    min-width: 44px;
}
```

---

## 🧪 **Testing Guidelines**

### **1. Unit Tests**
```typescript
// Test files: *.spec.ts
describe('UserService', () => {
    it('should create user', () => {
        // Test implementation
    });
});
```

### **2. E2E Tests**
```typescript
// E2E files: *.e2e-spec.ts
describe('User Profile Page', () => {
    it('should display user information', () => {
        // E2E test implementation
    });
});
```

---

## 📚 **Documentation Standards**

### **1. Code Comments**
```typescript
/**
 * Lấy thông tin user profile từ API
 * @param userId ID của user
 * @returns Promise<UserProfile> Thông tin user
 */
async getUserProfile(userId: string): Promise<UserProfile> {
    // Implementation
}
```

### **2. README Files**
- Mỗi module có README.md riêng
- Ghi lại purpose, usage, và examples
- Update khi có thay đổi major

---

## ⚡ **Build và Deployment**

### **Environment Configurations**
```typescript
// environment.ts (development)
export const environment = {
    production: false,
    apiUrl: 'http://localhost:3000'
};

// environment.prod.ts (production)
export const environment = {
    production: true,
    apiUrl: 'https://api.production.com'
};
```

### **Build Commands**
```bash
# Development build
npm run build

# Production build
npm run build-prod

# Mobile builds
ionic cap build ios --no-build
ionic cap build android --no-build
```

---

## 🔄 **Git Workflow**

### **Submodule Management**
```bash
# Update all submodules
git submodule update --init --recursive
git submodule foreach 'git checkout main && git pull origin main'

# Commit submodule changes
git submodule foreach 'git add . && git commit -m "Update" && git push'
```

### **Branch Strategy**
- `main` - Production ready code
- Mỗi nhân viên tạo branch theo dạng: `TEN-NHAN-VIEN/ten-chuc-nang` (ví dụ: `HIEU/sale-order-hotfix`)


## 📋 **Checklist cho Developer mới**

### **Setup Environment**
- [ ] Clone repository với submodules
- [ ] Install dependencies: `npm install`
- [ ] Setup IDE với ESLint và Prettier
- [ ] Đọc tài liệu architecture này

### **Before Coding**
- [ ] Hiểu business requirements
- [ ] Check existing components/services
- [ ] Follow naming conventions
- [ ] Use EnvService methods thay vì tự viết

### **Code Review Checklist**
- [ ] Code tuân thủ ESLint rules
- [ ] Sử dụng `dog && console.log()` cho dev logs
- [ ] Extend từ PageBase cho pages
- [ ] Sử dụng EnvService methods
- [ ] Handle errors properly
- [ ] Add appropriate comments

### **Testing**
- [ ] Unit tests cho business logic
- [ ] Manual testing trên mobile/desktop
- [ ] Check performance impact
- [ ] Verify security requirements

---

**Tài liệu này sẽ được cập nhật thường xuyên khi có thay đổi trong architecture hoặc coding standards.**

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
