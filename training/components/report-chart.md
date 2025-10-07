# ReportChart Component Documentation

## 📋 **Tổng quan**

`ReportChartComponent` là component hiển thị các loại biểu đồ cho báo cáo, hỗ trợ nhiều định dạng chart như line, bar, pie, doughnut.

**Selector**: `app-report-chart`  
**Location**: `src/app/components/visualizations/report-chart/report-chart.component.ts`

---

## 🚀 **Basic Usage**

### **Sales Chart**
```typescript
// Component
export class SalesReportPage extends PageBase {
    chartData = {
        labels: ['Jan', 'Feb', 'Mar', 'Apr', 'May'],
        datasets: [{
            label: 'Sales',
            data: [12000, 15000, 18000, 14000, 20000],
            backgroundColor: '#3880ff'
        }]
    };

    chartOptions = {
        responsive: true,
        plugins: {
            title: {
                display: true,
                text: 'Monthly Sales Report'
            }
        }
    };
}
```

```html
<!-- Template -->
<app-report-chart
    [type]="'bar'"
    [data]="chartData"
    [options]="chartOptions">
</app-report-chart>
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
