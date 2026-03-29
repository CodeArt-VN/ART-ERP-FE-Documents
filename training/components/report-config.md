# ReportConfig Component Documentation

## 📋 **Tổng quan**

`ReportConfigComponent` là component cấu hình báo cáo, cho phép người dùng thiết lập các tham số, filters và format cho reports.

**Selector**: `app-report-config`  
**Location**: `src/app/components/visualizations/report-config/report-config.component.ts`

---

## 🚀 **Basic Usage**

### **Report Configuration**
```typescript
// Component
export class ReportConfigPage extends PageBase {
    reportConfig = {
        type: 'sales-summary',
        dateRange: {
            from: new Date(),
            to: new Date()
        },
        groupBy: 'month',
        filters: [],
        format: 'chart'
    };

    onConfigChange(config: any) {
        this.reportConfig = config;
        this.generateReport();
    }

    async generateReport() {
        try {
            const result = await this.reportService.generate(this.reportConfig);
            this.reportData = result.data;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

```html
<!-- Template -->
<app-report-config
    [config]="reportConfig"
    (configChange)="onConfigChange($event)">
</app-report-config>
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
