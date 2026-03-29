# EChart Component Documentation

## 📋 **Tổng quan**

`EChartComponent` là component wrapper cho Apache ECharts library, cung cấp khả năng tạo các biểu đồ tương tác phong phú với nhiều loại chart khác nhau. Component hỗ trợ dynamic loading, custom scripts và responsive design.

**Selector**: `app-e-chart`  
**Location**: `src/app/components/visualizations/types/e-chart/e-chart.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() chartType: string = 'auto';                    // Chart type: 'auto', 'bar', 'line', 'fixed', 'manual'
@Input() viewMode: 'full' | 'mini' | 'dashboard';      // Display mode
@Input() chartOption: any = {};                         // ECharts option object
@Input() dimensions: string[] = [];                     // Data dimensions
@Input() viewDimension: string;                         // Primary dimension for viewing
@Input() compareBy: string[] = [];                      // Comparison dimensions
@Input() measureBy: string[] = [];                      // Measure dimensions
@Input() dataIntervalProperty: string;                  // Property for data intervals
@Input() dataIntervalType: string;                      // Type of data interval
@Input() chartScript: string;                           // Custom JavaScript for chart customization
@Input() data: any[] = [];                             // Main dataset
@Input() comparitionData: any[] = [];                  // Comparison dataset
```

### **Output Events**
```typescript
@Output() chartClick = new EventEmitter();              // Chart click events
@Output() dataChange = new EventEmitter();              // Data change events
```

---

## 🚀 **Basic Usage**

### **Simple Bar Chart**
```typescript
// Component
export class SalesReportPage extends PageBase {
    chartData = [
        { month: 'Jan', sales: 12000, target: 15000 },
        { month: 'Feb', sales: 15000, target: 15000 },
        { month: 'Mar', sales: 18000, target: 15000 },
        { month: 'Apr', sales: 14000, target: 15000 },
        { month: 'May', sales: 20000, target: 15000 }
    ];

    chartOption = {
        title: { text: 'Monthly Sales Report' },
        tooltip: { trigger: 'axis' },
        legend: { data: ['Sales', 'Target'] },
        xAxis: { 
            type: 'category',
            data: this.chartData.map(d => d.month)
        },
        yAxis: { type: 'value' },
        series: [
            {
                name: 'Sales',
                type: 'bar',
                data: this.chartData.map(d => d.sales),
                itemStyle: { color: '#3880ff' }
            },
            {
                name: 'Target',
                type: 'line',
                data: this.chartData.map(d => d.target),
                itemStyle: { color: '#ffc409' }
            }
        ]
    };

    onChartClick(params: any) {
        console.log('Chart clicked:', params);
        // Navigate to detailed view
        this.nav(`/sales-detail/${params.name}`, 'forward');
    }
}
```

```html
<!-- Template -->
<div class="chart-container" style="height: 400px;">
    <app-e-chart
        [chartType]="'manual'"
        [viewMode]="'full'"
        [chartOption]="chartOption"
        [data]="chartData"
        (chartClick)="onChartClick($event)">
    </app-e-chart>
</div>
```

---

## 📊 **Chart Types**

### **1. Auto Chart (chartType: 'auto')**
```typescript
export class AutoChartPage extends PageBase {
    // Automatically determines best chart type based on data
    autoChartConfig = {
        chartType: 'auto',
        viewMode: 'dashboard',
        dimensions: ['month', 'category'],
        viewDimension: 'month',
        measureBy: ['sales', 'profit'],
        dataIntervalProperty: 'date',
        dataIntervalType: 'month',
        data: this.salesData
    };
}
```

```html
<app-e-chart
    [chartType]="'auto'"
    [viewMode]="'dashboard'"
    [dimensions]="autoChartConfig.dimensions"
    [viewDimension]="autoChartConfig.viewDimension"
    [measureBy]="autoChartConfig.measureBy"
    [dataIntervalProperty]="autoChartConfig.dataIntervalProperty"
    [dataIntervalType]="autoChartConfig.dataIntervalType"
    [data]="autoChartConfig.data">
</app-e-chart>
```

### **2. Line Chart (chartType: 'line')**
```typescript
export class TrendChartPage extends PageBase {
    lineChartOption = {
        title: { text: 'Sales Trend' },
        xAxis: { type: 'category' },
        yAxis: { type: 'value' },
        dataset: {
            dimensions: ['month', 'sales', 'target'],
            source: this.trendData
        },
        series: [
            { type: 'line', name: 'Sales' },
            { type: 'line', name: 'Target' }
        ]
    };
}
```

### **3. Custom Script Chart**
```typescript
export class CustomChartPage extends PageBase {
    customScript = `
        // Custom chart modifications
        option.title.textStyle = {
            color: '#333',
            fontSize: 18,
            fontWeight: 'bold'
        };
        
        option.series.forEach(series => {
            series.smooth = true;
            series.symbolSize = 8;
        });
        
        // Add custom tooltip
        option.tooltip = {
            formatter: function(params) {
                return params.seriesName + ': ' + li.formatMoney(params.value);
            }
        };
    `;

    baseChartOption = {
        title: { text: 'Revenue Chart' },
        // ... other options
    };
}
```

```html
<app-e-chart
    [chartType]="'manual'"
    [chartOption]="baseChartOption"
    [chartScript]="customScript"
    [data]="revenueData">
</app-e-chart>
```

---

## 🎨 **Advanced Features**

### **Dashboard Mini Charts**
```typescript
export class DashboardPage extends PageBase {
    miniCharts = [
        {
            title: 'Sales',
            type: 'line',
            viewMode: 'mini',
            data: this.salesData,
            option: {
                grid: { top: 10, right: 10, bottom: 10, left: 10 },
                xAxis: { show: false },
                yAxis: { show: false },
                series: [{ type: 'line', smooth: true, symbol: 'none' }]
            }
        },
        {
            title: 'Orders',
            type: 'bar',
            viewMode: 'mini',
            data: this.orderData,
            option: {
                grid: { top: 10, right: 10, bottom: 10, left: 10 },
                xAxis: { show: false },
                yAxis: { show: false },
                series: [{ type: 'bar' }]
            }
        }
    ];
}
```

```html
<div class="dashboard-charts">
    <div class="mini-chart" *ngFor="let chart of miniCharts">
        <h3>{{ chart.title }}</h3>
        <div style="height: 100px;">
            <app-e-chart
                [chartType]="chart.type"
                [viewMode]="chart.viewMode"
                [chartOption]="chart.option"
                [data]="chart.data">
            </app-e-chart>
        </div>
    </div>
</div>
```

### **Comparison Charts**
```typescript
export class ComparisonChartPage extends PageBase {
    currentYearData = [/* current year sales data */];
    previousYearData = [/* previous year sales data */];

    comparisonChartOption = {
        title: { text: 'Year-over-Year Comparison' },
        legend: { data: ['2024', '2023'] },
        xAxis: { type: 'category', data: ['Jan', 'Feb', 'Mar', 'Apr', 'May'] },
        yAxis: { type: 'value' },
        series: [
            {
                name: '2024',
                type: 'bar',
                data: this.currentYearData,
                itemStyle: { color: '#3880ff' }
            },
            {
                name: '2023',
                type: 'bar',
                data: this.previousYearData,
                itemStyle: { color: '#92949c' }
            }
        ]
    };
}
```

---

## 🎛️ **Interactive Features**

### **Chart Click Handling**
```typescript
export class InteractiveChartPage extends PageBase {
    onChartClick(params: any) {
        switch (params.componentType) {
            case 'series':
                this.handleSeriesClick(params);
                break;
            case 'xAxis':
                this.handleAxisClick(params);
                break;
            default:
                console.log('Chart clicked:', params);
        }
    }

    handleSeriesClick(params: any) {
        const { seriesName, name, value } = params;
        
        // Show detailed modal
        this.showDetailModal({
            series: seriesName,
            category: name,
            value: value
        });
    }

    handleAxisClick(params: any) {
        // Filter data by clicked axis value
        this.filterDataByCategory(params.value);
    }

    async showDetailModal(data: any) {
        const modal = await this.modalController.create({
            component: ChartDetailModal,
            componentProps: { data }
        });
        
        await modal.present();
    }
}
```

### **Dynamic Data Updates**
```typescript
export class RealTimeChartPage extends PageBase {
    chartOption = { /* initial chart config */ };
    
    ngOnInit() {
        // Update chart data every 30 seconds
        setInterval(() => {
            this.updateChartData();
        }, 30000);
    }

    async updateChartData() {
        try {
            const newData = await this.dataService.getLatestData();
            
            // Update chart option with new data
            this.chartOption = {
                ...this.chartOption,
                series: [{
                    ...this.chartOption.series[0],
                    data: newData
                }]
            };
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

---

## 🎨 **Styling & Themes**

### **Custom Chart Themes**
```typescript
export class ThemedChartPage extends PageBase {
    darkThemeOption = {
        backgroundColor: '#1e1e1e',
        title: {
            textStyle: { color: '#ffffff' }
        },
        legend: {
            textStyle: { color: '#ffffff' }
        },
        xAxis: {
            axisLine: { lineStyle: { color: '#ffffff' } },
            axisLabel: { color: '#ffffff' }
        },
        yAxis: {
            axisLine: { lineStyle: { color: '#ffffff' } },
            axisLabel: { color: '#ffffff' },
            splitLine: { lineStyle: { color: '#333333' } }
        }
    };

    applyTheme(isDark: boolean) {
        if (isDark) {
            Object.assign(this.chartOption, this.darkThemeOption);
        } else {
            // Apply light theme
            this.chartOption.backgroundColor = '#ffffff';
            // ... other light theme properties
        }
    }
}
```

### **Responsive Design**
```scss
// Component SCSS
.chart-container {
    width: 100%;
    height: 400px;
    
    @media (max-width: 768px) {
        height: 300px;
    }
    
    @media (max-width: 480px) {
        height: 250px;
    }
    
    app-e-chart {
        width: 100%;
        height: 100%;
        display: block;
    }
}

.dashboard-charts {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 16px;
    
    .mini-chart {
        background: var(--ion-color-light);
        padding: 16px;
        border-radius: 8px;
        
        h3 {
            margin: 0 0 8px 0;
            font-size: 1rem;
            color: var(--ion-color-dark);
        }
    }
}
```

---

## 🔧 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Đúng - Lazy load ECharts library
async loadChart() {
    // Chart will auto-load ECharts library when needed
    this.showChart = true;
}

// ✅ Đúng - Dispose chart on destroy
ngOnDestroy() {
    if (this.chart) {
        this.chart.dispose();
    }
}
```

### **2. Error Handling**
```typescript
// ✅ Đúng - Handle chart script errors
onChartScriptError(error: any) {
    console.warn('Chart script error:', error);
    this.env.showMessage('Chart customization failed', 'warning');
    // Fallback to default chart
    this.useDefaultChart();
}
```

### **3. Data Validation**
```typescript
// ✅ Đúng - Validate chart data
validateChartData(data: any[]): boolean {
    if (!data || data.length === 0) {
        this.showEmptyChart();
        return false;
    }
    
    // Check data structure
    const requiredFields = ['x', 'y'];
    const isValid = data.every(item => 
        requiredFields.every(field => item.hasOwnProperty(field))
    );
    
    if (!isValid) {
        this.env.showMessage('Invalid chart data format', 'danger');
        return false;
    }
    
    return true;
}
```

---

## 🚨 **Common Use Cases**

### **1. Sales Dashboard**
```typescript
export class SalesDashboard extends PageBase {
    // Multiple chart types for comprehensive view
    charts = [
        { type: 'line', title: 'Sales Trend', data: this.salesTrend },
        { type: 'bar', title: 'Product Performance', data: this.productSales },
        { type: 'pie', title: 'Sales by Region', data: this.regionSales }
    ];
}
```

### **2. Financial Reports**
```typescript
export class FinancialReports extends PageBase {
    // Complex financial charts with custom calculations
    revenueChart = {
        type: 'manual',
        script: this.getFinancialChartScript(),
        data: this.financialData
    };
}
```

### **3. Real-time Monitoring**
```typescript
export class MonitoringDashboard extends PageBase {
    // Real-time data updates
    setupRealTimeCharts() {
        this.websocketService.connect().subscribe(data => {
            this.updateChartData(data);
        });
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
