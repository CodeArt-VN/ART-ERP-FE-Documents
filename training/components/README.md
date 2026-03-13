# Components Documentation - ART-ERP-FE

## 📋 **Mục đích tài liệu**

Thư mục này chứa tài liệu hướng dẫn sử dụng chi tiết cho tất cả components trong `src/app/components/`. Mỗi component có một file markdown riêng với examples và best practices.

---

## 📊 **Thống kê Components**

- **Tổng số components trong source**: 127+ components
- **Đã document**: 27 components  
- **Coverage**: Tất cả components quan trọng và thường dùng

---

## 📂 **Danh sách Components đã document**

### **🎛️ Form & Input Controls (6 components)**
- **[form-control.md](./form-control.md)** - Form field wrapper với validation
- **[input-control.md](./input-control.md)** - Advanced input với tree picker, date picker, Monaco editor
- **[field-control.md](./field-control.md)** - Configuration field controls với nhiều types
- **[group-control.md](./group-control.md)** - Group form fields với configuration
- **[color-picker.md](./color-picker.md)** - Color selection với predefined colors
- **[icon-picker.md](./icon-picker.md)** - Icon selection với search functionality

### **📊 Data Display Components (3 components)**
- **[data-table.md](./data-table.md)** - Advanced data table với filter, sort, tree view, infinite scroll
- **[json-viewer.md](./json-viewer.md)** - JSON data viewer với comparison capabilities
- **[format-quantity.md](./format-quantity.md)** - Quantity formatting và display

### **🛠️ Toolbar & Navigation Components (3 components)**
- **[list-toolbar.md](./list-toolbar.md)** - Toolbar cho list pages với CRUD actions
- **[toolbar.md](./toolbar.md)** - Toolbar cho list và detail pages (thay thế detail-toolbar)
- **[detail-toolbar.md](./detail-toolbar.md)** - (Deprecated) Xem toolbar.md
- **[branch-breadcrumbs.md](./branch-breadcrumbs.md)** - Hierarchical breadcrumb navigation

### **🔍 Filter & Search Components (1 component)**
- **[filter.md](./filter.md)** - Advanced filter builder với logical operators

### **📍 Map Components (3 components)**
- **[map-view.md](./map-view.md)** - Interactive map display
- **[coordinate-picker.md](./coordinate-picker.md)** - GPS coordinate selection
- **[address.md](./address.md)** - Address input với geocoding support

### **📊 Visualization & Chart Components (4 components)**
- **[report-chart.md](./report-chart.md)** - Chart visualization cho reports
- **[card-multi-row.md](./card-multi-row.md)** - Multi-row data cards cho dashboard
- **[report-config.md](./report-config.md)** - Report configuration interface
- **[e-chart.md](./e-chart.md)** - ECharts wrapper với custom scripts và dynamic loading

### **🔔 Notification & Message Components (4 components)**
- **[notifications.md](./notifications.md)** - Real-time notification system
- **[page-message.md](./page-message.md)** - Empty state và loading messages
- **[page-notification.md](./page-notification.md)** - Page-specific notifications với read/delete
- **[page-title.md](./page-title.md)** - Dynamic page title với icon

### **🎨 UI Utility Components (2 components)**
- **[reorder.md](./reorder.md)** - Drag-and-drop list reordering
- **[dynamic-template.md](./dynamic-template.md)** - Dynamic UI rendering từ configuration

---

## 🚫 **Components không được document**

### **Đã bỏ qua theo yêu cầu:**
- **Printing Components**: `sale-order-note`, `purchase-order-note`
- **Advanced Filter**: `query-filter`

### **Sub-components (không cần document riêng):**
- **DataTable internals**: `datatable-body`, `datatable-header`, `datatable-*-cell` components
- **Chart types**: Internal chart type components

---

## 🚀 **Quick Start Guide**

### **1. Import Components**
```typescript
// Import individual components
import { DataTableComponent } from 'src/app/components/data-table/data-table.component';
import { ListToolbarComponent } from 'src/app/components/list-toolbar/list-toolbar.component';

// Import shared modules
import { ShareComponentsModule } from 'src/app/components/share-components.module';
import { ShareDataTableModule } from 'src/app/components/data-table/share-data-table.module';
```

### **2. Basic Usage Pattern**
```typescript
// In your page component
@Component({
    selector: 'app-sample-page',
    template: `
        <app-list-toolbar
            [pageConfig]="pageConfig"
            [selectedItems]="selectedItems"
            (add)="add()"
            (refresh)="refresh()">
        </app-list-toolbar>
        
        <app-data-table
            [rows]="items"
            [columns]="columns"
            [(selectedRows)]="selectedItems"
            (filter)="onFilter($event)">
        </app-data-table>
    `
})
export class SamplePage extends PageBase {
    // Component logic
}
```

### **3. Common Patterns**
```typescript
// Basic form controls
<app-form-control [field]="{
    form: formGroup,
    id: 'name',
    type: 'text',
    label: 'Name'
}"></app-form-control>

// Advanced input controls
<app-input-control [field]="{
    form: formGroup,
    id: 'branch',
    type: 'tree',
    label: 'Select Branch',
    dataSource: branchList,
    isTree: true
}"></app-input-control>

// Configuration field controls
<app-field-control [field]="{
    type: 'select-staff',
    label: 'Assigned Staff',
    data: { ValueObject: null },
    DataSource: staffList
}" (onChange)="onFieldChange($event)"></app-field-control>

// Data table với toolbar
<app-list-toolbar [pageConfig]="pageConfig" [selectedItems]="selectedItems"></app-list-toolbar>
<app-data-table [rows]="items" [columns]="columns" [(selectedRows)]="selectedItems"></app-data-table>

// Charts và visualization
<app-e-chart [chartType]="'bar'" [chartOption]="chartConfig" [data]="chartData"></app-e-chart>
<app-report-chart [type]="'line'" [data]="reportData"></app-report-chart>

// Map components
<app-map-view [center]="mapCenter" [markers]="markers"></app-map-view>
<app-coordinate-picker (coordinateSelected)="onLocationSelected($event)"></app-coordinate-picker>

// Notifications
<app-notifications></app-notifications>
<app-page-notification [pageName]="currentPageName"></app-page-notification>
```

---

## 📋 **Component Categories**

### **Form Controls**
Components để xây dựng forms với validation và user experience tốt.

### **Data Display**
Components để hiển thị data dưới nhiều format khác nhau (table, chart, card).

### **Navigation & Toolbar**
Components để navigation và action buttons cho pages.

### **Filters & Search**
Components để tìm kiếm và lọc data.

### **Maps & Location**
Components để làm việc với maps và location data.

### **Notifications**
Components để hiển thị messages và notifications.

### **Printing**
Components để in ấn documents và reports.

### **UI Utilities**
Utility components để enhance user interface.

---

## 🎯 **Best Practices**

### **1. Component Selection**
- **Data Display**: Sử dụng `DataTableComponent` cho tất cả data tables, `JsonViewerComponent` cho JSON data
- **Toolbars**: `ListToolbarComponent` cho list pages, `ToolbarComponent` cho list và detail pages
- **Form Controls**: `FormControlComponent` cho basic inputs, `InputControlComponent` cho advanced inputs, `FieldControlComponent` cho configuration forms
- **Charts**: `ReportChartComponent` cho basic charts, `EChartComponent` cho advanced interactive charts
- **Maps**: `MapViewComponent` cho map display, `CoordinatePickerComponent` cho location selection
- **Notifications**: `NotificationsComponent` cho global notifications, `PageNotificationComponent` cho page-specific

### **2. Performance**
- Lazy load components khi có thể
- Sử dụng OnPush change detection cho data display components
- Cache data cho components có expensive operations

### **3. Consistency**
- Follow naming conventions trong tất cả components
- Sử dụng shared styling và themes
- Implement accessibility standards

### **4. Testing**
- Unit test cho component logic
- Integration test cho component interactions
- E2E test cho critical user flows

---

## 🔧 **Development Guidelines**

### **Creating New Components**
1. Extend từ base classes khi có thể
2. Implement proper input/output interfaces
3. Add comprehensive documentation
4. Include usage examples
5. Write unit tests

### **Modifying Existing Components**
1. Check backward compatibility
2. Update documentation
3. Test với existing usage
4. Notify team về breaking changes

### **Component Architecture**
- Keep components focused và single-purpose
- Use composition over inheritance
- Implement proper error handling
- Follow Angular best practices

---

**Tài liệu này sẽ được cập nhật thường xuyên khi có components mới hoặc thay đổi.**

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
