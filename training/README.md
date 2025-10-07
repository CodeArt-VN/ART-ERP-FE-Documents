# ART-ERP Frontend Training Documentation

## 📚 **Tổng quan**

Bộ tài liệu training toàn diện cho dự án ART-ERP Frontend, bao gồm architecture, components, services, pipes, directives và best practices. Tài liệu này được thiết kế để training cho nhân viên mới và AI assistants.

## 🎯 **Mục đích**

- **Onboarding**: Hướng dẫn nhân viên mới làm quen với codebase
- **Reference**: Tài liệu tham khảo cho developers
- **Standards**: Quy tắc và best practices
- **AI Training**: Training material cho AI assistants

## 📋 **Danh mục tài liệu**

### **🏗️ 1. Architecture & Governance**

#### **[📖 Source Code Architect](./source-code-architect.md)**
- **Mô tả**: Quy tắc cơ bản và governance của dự án
- **Nội dung**:
  - Framework và thư viện chính (Angular 20 + Ionic 8 + Capacitor 7)
  - Cấu trúc thư mục và files cốt lõi
  - Quy tắc đặt tên (kebab-case, PascalCase, camelCase)
  - Nguyên tắc bảo mật (JWT, permissions, validation)
  - Performance guidelines (lazy loading, OnPush, caching)
  - Mobile development practices
  - Branch strategy và workflow

---

### **🔧 2. Services Documentation**

#### **[📖 Services Overview](./services/README.md)**
- **Mô tả**: Tổng quan về architecture và patterns của services
- **Nội dung**: Service layers, dependency injection, caching strategies

#### **🏢 Core Services**
- **[CommonService](./services/core/common-service.md)** - HTTP communication và API client
- **[EnvService](./services/core/env-service.md)** - Environment management và app context
- **[ExtendService](./services/core/extend-service.md)** - Base service cho CRUD operations với caching

#### **🔐 Authentication Services**
- **[AuthenticationService](./services/auth/authentication-service.md)** - User authentication, session management, MFA

#### **🛠️ Utility Services**
- **[PrintingService](./services/util/printing-service.md)** - In ấn với QZ Tray integration
- **[BarcodeScannerService](./services/util/barcode-scanner-service.md)** - Quét mã vạch/QR code cho mobile
- **[DynamicTranslateLoaderService](./services/util/translate-loader-service.md)** - Dynamic i18n loading
- **[UtilityService](./services/util/utility-service.md)** - Utility functions và recommendations
- **[Custom Validators](./services/util/validators.md)** - Form validation với custom rules

---

### **🧩 3. Components Documentation**

#### **[📖 Components Overview](./components/README.md)**
- **Mô tả**: Tổng quan về 27+ components đã document
- **Thống kê**: 127+ components trong source, 27 đã document
- **Best practices**: Patterns và usage guidelines

#### **📝 Form & Input Controls (6 components)**
- **[FormControlComponent](./components/form-control.md)** - Wrapper cho form controls
- **[InputControlComponent](./components/input-control.md)** - Universal input với Monaco editor
- **[FieldControlComponent](./components/field-control.md)** - Configuration field controls
- **[GroupControlComponent](./components/group-control.md)** - Form group với validation
- **[ColorPickerComponent](./components/color-picker.md)** - Color selection component
- **[IconPickerComponent](./components/icon-picker.md)** - Ionic icon picker

#### **📊 Data Display (3 components)**
- **[DataTableComponent](./components/data-table.md)** - Advanced table với filtering, sorting, tree view
- **[JsonViewerComponent](./components/json-viewer.md)** - JSON display và comparison
- **[FormatQuantityComponent](./components/format-quantity.md)** - Quantity formatting với split details

#### **🔧 Toolbar & Navigation (4 components)**
- **[ListToolbarComponent](./components/list-toolbar.md)** - Toolbar cho list pages
- **[DetailToolbarComponent](./components/detail-toolbar.md)** - Toolbar cho detail pages
- **[ToolbarComponent](./components/toolbar.md)** - Generic toolbar với actions
- **[BranchBreadcrumbsComponent](./components/branch-breadcrumbs.md)** - Hierarchical navigation

#### **🔍 Filter & Search (1 component)**
- **[FilterComponent](./components/filter.md)** - Advanced filter builder với logical operators

#### **🗺️ Map Components (3 components)**
- **[MapViewComponent](./components/map-view.md)** - Interactive map display
- **[CoordinatePickerComponent](./components/coordinate-picker.md)** - Location selection
- **[AddressComponent](./components/address.md)** - Address input và validation

#### **📈 Visualization & Chart Components (4 components)**
- **[ReportChartComponent](./components/report-chart.md)** - Business reporting charts
- **[CardMultiRowComponent](./components/card-multi-row.md)** - Multi-row data cards
- **[ReportConfigComponent](./components/report-config.md)** - Chart configuration
- **[EChartComponent](./components/e-chart.md)** - ECharts wrapper với advanced features

#### **🔔 Notification & Message Components (4 components)**
- **[NotificationsComponent](./components/notifications.md)** - Real-time notifications
- **[PageMessageComponent](./components/page-message.md)** - Empty states và loading messages
- **[PageNotificationComponent](./components/page-notification.md)** - Page-specific notifications
- **[PageTitleComponent](./components/page-title.md)** - Page headers với icons

#### **⚙️ UI Utility Components (2 components)**
- **[ReorderComponent](./components/reorder.md)** - Drag-and-drop reordering
- **[DynamicTemplateComponent](./components/dynamic-template.md)** - Dynamic content rendering

---

### **🔄 4. Pipes Documentation**

#### **[📖 Pipes Overview](./pipes/README.md)**
- **Mô tả**: Tổng quan về 9 custom pipes
- **Categories**: Security, filtering, search, formatting

#### **🔒 Security Pipes (3 pipes)**
- **[SafeFramePipe](./pipes/safe-frame.md)** - Bypass security cho resource URLs
- **[SafeHtmlPipe](./pipes/safe-html.md)** - Bypass security cho HTML content
- **[SafeStylePipe](./pipes/safe-style.md)** - Bypass security cho CSS styles

#### **🔍 Filter & Search Pipes (4 pipes)**
- **[filterProperties](./pipes/filter.md)** - Filter arrays với conditions
- **[searchProperties](./pipes/search.md)** - Search arrays case-insensitive
- **[searchNoAccents](./pipes/search-no-accent.md)** - Search với Vietnamese accent removal
- **[isNotDeleted](./pipes/is-not-deleted.md)** - Filter non-deleted items

#### **📅 Formatting Pipes (2 pipes)**
- **[DateFriendlyPipe](./pipes/date-friendly.md)** - Relative date formatting ("5 minutes ago")
- **[NumberFriendlyPipe](./pipes/number-friendly.md)** - Localized currency formatting

---

### **📐 5. Directives Documentation**

#### **[📖 Directives Overview](./directives/README.md)**
- **Mô tả**: Tổng quan về custom directives
- **Categories**: Translation, UI enhancement, styling

#### **🌐 Translation & Content (1 directive)**
- **[TranslateResourceDirective](./directives/translate-resource.md)** - Dynamic content translation

#### **🎨 UI Enhancement (1 directive)**
- **[SvgImageDirective](./directives/svg-image.md)** - SVG loading và customization

---

### **📚 6. Utility Libraries**

#### **[📖 Global Functions Library](./js-lib.md)**
- **Mô tả**: Tổng hợp 50+ utility functions trong global-functions.ts
- **Categories**:
  - **Time & Date**: formatTimeConfig, dateFormat, dateFormatFriendly
  - **Currency & Number**: formatMoney, currencyFormat, isNumeric
  - **Object & Array**: deepAssign, cloneObject, getObject
  - **Tree Data**: listToTree, treeToList, searchTree, buildFlatTree
  - **String Processing**: generateUID, personNameFormat, URLFormat
  - **UI & Display**: hexToRgba, fileSizeFormat, getCssVariableValue
  - **Banking & QR**: genBankTransferQRCode, readVietQRCode, calcCRC

#### **[📖 Environment Service Guide](./env-service.md)**
- **Mô tả**: Chi tiết về EnvService - central service của ứng dụng
- **Features**:
  - Message & notification system
  - Storage management (localStorage, cookies)
  - Branch context và permissions
  - Event system và network monitoring
  - Translation và multi-language support

#### **[📖 Base Service Pattern](./services.md)**
- **Mô tả**: ExtendService pattern cho standardized CRUD operations
- **Features**:
  - Caching strategies với TTL
  - Approval workflow (submit, approve, disapprove)
  - File operations (import, export, upload, download)
  - Bulk operations và error handling

---

## 🚀 **Quick Start Guide**

### **Cho Developers mới:**
1. **Bắt đầu với**: [Source Code Architect](./source-code-architect.md)
2. **Hiểu Services**: [Services Overview](./services/README.md)
3. **Học Components**: [Components Overview](./components/README.md)
4. **Sử dụng Utilities**: [Global Functions](./js-lib.md) và [EnvService](./env-service.md)

### **Cho AI Assistants:**
1. **Architecture Rules**: [Source Code Architect](./source-code-architect.md)
2. **Service Patterns**: [ExtendService](./services/core/extend-service.md) và [EnvService](./services/core/env-service.md)
3. **Component Usage**: [Components README](./components/README.md)
4. **Utility Functions**: [Global Functions Library](./js-lib.md)

---

## 📊 **Thống kê Documentation**

| Category | Files | Status |
|----------|-------|--------|
| **Architecture** | 1 | ✅ Complete |
| **Services** | 6 | ✅ Core Complete |
| **Components** | 27 | ✅ Major Complete |
| **Pipes** | 9 | ✅ Complete |
| **Directives** | 2 | 🔄 Partial |
| **Utilities** | 3 | ✅ Complete |
| **Total** | **48+** | **85% Complete** |

---

## 🎯 **Best Practices Summary**

### **🔧 Development Practices**
- **Extend PageBase** cho tất cả pages
- **Extend ExtendService** cho business services  
- **Sử dụng EnvService** cho messages, storage, permissions
- **Console logging**: `dog && console.log()` cho dev mode
- **Naming conventions**: kebab-case files, PascalCase classes, camelCase variables

### **🏗️ Architecture Patterns**
- **Modular design** với Git submodules
- **Service layers**: Core → Custom → Page services
- **Component hierarchy**: Base → Specialized components
- **Caching strategy**: TTL-based với auto-refresh

### **⚡ Performance Guidelines**
- **Lazy loading** cho modules và routes
- **OnPush change detection** cho performance
- **Caching** cho API calls và static data
- **Tree shaking** và code splitting

### **🔒 Security Standards**
- **JWT authentication** với auto-refresh
- **Permission checking** với `env.checkFormPermission()`
- **Input validation** với custom validators
- **HTTPS** cho production environments

---

## 🔄 **Cập nhật và Bảo trì**

### **Quy trình cập nhật documentation:**
1. **Code changes** → Update relevant documentation
2. **New components** → Add to components folder
3. **New services** → Add to services folder  
4. **Architecture changes** → Update source-code-architect.md

### **Review schedule:**
- **Monthly**: Review và update documentation
- **Release**: Sync documentation với code changes
- **Quarterly**: Architecture review và improvements

---

## 📞 **Hỗ trợ**

### **Khi cần hỗ trợ:**
1. **Tìm trong documentation** này trước
2. **Check source code** với examples trong docs
3. **Hỏi team lead** nếu không tìm thấy thông tin
4. **Update documentation** sau khi giải quyết vấn đề

### **Contribution:**
- **Phát hiện lỗi**: Tạo issue hoặc fix trực tiếp
- **Thêm tính năng**: Update documentation cùng với code
- **Cải thiện docs**: Pull request với improvements

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
