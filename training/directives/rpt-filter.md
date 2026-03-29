# RptFilterDirective Documentation

## 📄 **Tổng quan**

`RptFilterDirective` là một placeholder directive trong ứng dụng ART-ERP-FE. Hiện tại directive này chưa có implementation cụ thể và có thể được sử dụng cho report filtering functionality trong tương lai.

## 🎯 **Mục đích sử dụng**

- **Report Filtering**: Placeholder cho report filter functionality
- **Future Implementation**: Chuẩn bị cho tính năng filter reports
- **Directive Structure**: Provide basic directive structure
- **Extensibility**: Ready for future enhancements

## 📍 **File location**
```
src/app/directives/rpt-filter.directive.ts
```

## 🔧 **Directive Configuration**

```typescript
@Directive({
    selector: '[appRptFilter]'
})
```

## 🏗️ **Current Implementation**

```typescript
import { Directive } from '@angular/core';

@Directive({
    selector: '[appRptFilter',
})
export class RptFilterDirective {
    constructor() {}
}
```

## 🎨 **Current Usage**

### **Basic Usage (Placeholder)**
```html
<!-- Apply report filter directive -->
<div appRptFilter>
    <app-report-component>
        <!-- Report content -->
    </app-report-component>
</div>
```

## 🚀 **Potential Future Implementation**

### **1. Report Filter Interface**
```typescript
interface ReportFilter {
    field: string;
    operator: 'equals' | 'contains' | 'startsWith' | 'endsWith' | 'greaterThan' | 'lessThan';
    value: any;
    dataType: 'string' | 'number' | 'date' | 'boolean';
}

interface ReportFilterConfig {
    filters: ReportFilter[];
    logicalOperator: 'AND' | 'OR';
    caseSensitive: boolean;
}
```

### **2. Enhanced Directive Implementation**
```typescript
@Directive({
    selector: '[appRptFilter]'
})
export class RptFilterDirective implements OnInit, OnDestroy {
    @Input() filterConfig: ReportFilterConfig;
    @Input() dataSource: any[];
    @Output() filteredData = new EventEmitter<any[]>();
    
    private subscription: Subscription;
    
    constructor(private el: ElementRef) {}
    
    ngOnInit() {
        this.applyFilters();
        this.setupFilterWatcher();
    }
    
    ngOnDestroy() {
        this.subscription?.unsubscribe();
    }
    
    private applyFilters() {
        if (!this.dataSource || !this.filterConfig) {
            this.filteredData.emit(this.dataSource);
            return;
        }
        
        const filtered = this.dataSource.filter(item => 
            this.evaluateFilters(item, this.filterConfig)
        );
        
        this.filteredData.emit(filtered);
    }
    
    private evaluateFilters(item: any, config: ReportFilterConfig): boolean {
        const results = config.filters.map(filter => 
            this.evaluateFilter(item, filter)
        );
        
        return config.logicalOperator === 'AND' 
            ? results.every(result => result)
            : results.some(result => result);
    }
    
    private evaluateFilter(item: any, filter: ReportFilter): boolean {
        const fieldValue = this.getFieldValue(item, filter.field);
        const filterValue = filter.value;
        
        switch (filter.operator) {
            case 'equals':
                return fieldValue === filterValue;
            case 'contains':
                return String(fieldValue).toLowerCase()
                    .includes(String(filterValue).toLowerCase());
            case 'startsWith':
                return String(fieldValue).toLowerCase()
                    .startsWith(String(filterValue).toLowerCase());
            case 'endsWith':
                return String(fieldValue).toLowerCase()
                    .endsWith(String(filterValue).toLowerCase());
            case 'greaterThan':
                return Number(fieldValue) > Number(filterValue);
            case 'lessThan':
                return Number(fieldValue) < Number(filterValue);
            default:
                return true;
        }
    }
    
    private getFieldValue(item: any, fieldPath: string): any {
        return fieldPath.split('.').reduce((obj, key) => obj?.[key], item);
    }
}
```

### **3. Usage Examples (Future)**
```html
<!-- Basic report filtering -->
<div appRptFilter 
     [dataSource]="reportData"
     [filterConfig]="filterConfig"
     (filteredData)="onDataFiltered($event)">
    
    <app-report-table [data]="filteredReportData">
    </app-report-table>
</div>

<!-- Advanced filtering with multiple conditions -->
<div appRptFilter 
     [dataSource]="salesData"
     [filterConfig]="{
         filters: [
             { field: 'amount', operator: 'greaterThan', value: 1000, dataType: 'number' },
             { field: 'status', operator: 'equals', value: 'completed', dataType: 'string' }
         ],
         logicalOperator: 'AND',
         caseSensitive: false
     }"
     (filteredData)="updateChart($event)">
     
    <app-sales-chart [data]="filteredSalesData">
    </app-sales-chart>
</div>
```

### **4. Component Integration (Future)**
```typescript
export class ReportPage {
    reportData: any[] = [];
    filteredReportData: any[] = [];
    
    filterConfig: ReportFilterConfig = {
        filters: [
            {
                field: 'date',
                operator: 'greaterThan',
                value: '2024-01-01',
                dataType: 'date'
            },
            {
                field: 'category',
                operator: 'contains',
                value: 'sales',
                dataType: 'string'
            }
        ],
        logicalOperator: 'AND',
        caseSensitive: false
    };
    
    onDataFiltered(filteredData: any[]) {
        this.filteredReportData = filteredData;
        this.updateReportMetrics();
    }
    
    updateFilter(field: string, value: any) {
        const filter = this.filterConfig.filters.find(f => f.field === field);
        if (filter) {
            filter.value = value;
        }
    }
    
    addFilter(filter: ReportFilter) {
        this.filterConfig.filters.push(filter);
    }
    
    removeFilter(field: string) {
        this.filterConfig.filters = this.filterConfig.filters
            .filter(f => f.field !== field);
    }
}
```

## 🔧 **Integration Possibilities**

### **With Report Components**
```html
<!-- Integration with existing report components -->
<div appRptFilter [dataSource]="data" (filteredData)="updateReport($event)">
    <app-report-chart [data]="filteredData"></app-report-chart>
    <app-report-table [data]="filteredData"></app-report-table>
</div>
```

### **With Filter Component**
```html
<!-- Combine with filter component -->
<app-filter (filterChange)="updateReportFilter($event)"></app-filter>
<div appRptFilter [filterConfig]="reportFilter" [dataSource]="reportData">
    <app-report-content></app-report-content>
</div>
```

### **With Data Table**
```html
<!-- Apply to data table -->
<div appRptFilter [dataSource]="tableData" (filteredData)="onTableFilter($event)">
    <app-data-table [rows]="filteredTableData"></app-data-table>
</div>
```

## 📋 **Development Guidelines**

### **1. Future Implementation Considerations**
- **Performance**: Implement efficient filtering algorithms
- **Memory**: Handle large datasets appropriately
- **Reactivity**: Support reactive data sources
- **Validation**: Validate filter configurations

### **2. API Design Principles**
- **Flexibility**: Support various filter types and operators
- **Composability**: Allow complex filter combinations
- **Extensibility**: Easy to add new operators and data types
- **Type Safety**: Strong TypeScript typing

### **3. Testing Strategy**
```typescript
// Unit tests for future implementation
describe('RptFilterDirective', () => {
    it('should filter data based on equals operator', () => {
        // Test implementation
    });
    
    it('should handle AND/OR logical operators', () => {
        // Test implementation
    });
    
    it('should support nested field paths', () => {
        // Test implementation
    });
});
```

## 🎯 **Current Status**

### **Implementation Status**
- ✅ **Basic Structure**: Directive shell created
- ❌ **Functionality**: No implementation yet
- ❌ **Input/Output**: No properties defined
- ❌ **Logic**: No filtering logic implemented

### **Next Steps**
1. **Define Requirements**: Specify filtering requirements
2. **Design API**: Create input/output interface
3. **Implement Logic**: Add filtering functionality
4. **Add Tests**: Create comprehensive test suite
5. **Documentation**: Update documentation with real examples

## 🚨 **Current Limitations**

- **No Functionality**: Directive currently does nothing
- **Placeholder Only**: Exists as structural placeholder
- **No API**: No inputs, outputs, or methods defined
- **No Integration**: Not integrated with any components

## 🔮 **Future Enhancements**

### **Advanced Features**
- **Custom Operators**: Support for custom filter operators
- **Async Filtering**: Support for server-side filtering
- **Filter Persistence**: Save and restore filter states
- **Filter Templates**: Predefined filter configurations
- **Performance Optimization**: Virtual scrolling integration

### **UI Integration**
- **Filter Builder**: Visual filter configuration UI
- **Quick Filters**: Predefined common filters
- **Filter History**: Track and reuse previous filters
- **Export Filters**: Export filtered data

RptFilterDirective hiện tại là placeholder directive sẵn sàng cho future implementation của report filtering functionality trong ứng dụng ART-ERP-FE.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
