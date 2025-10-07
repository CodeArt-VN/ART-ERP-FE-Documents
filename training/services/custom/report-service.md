# ReportService Documentation

## 📋 **Tổng quan**

`ReportService` là comprehensive service để handle business intelligence reports và data visualization trong hệ thống ERP. Service này extend từ `BI_ReportProvider` và provide advanced functionality cho việc tạo, quản lý và render các loại reports khác nhau.

**Location**: `src/app/services/custom/report.service.ts`  
**Injectable**: `providedIn: 'root'`  
**Extends**: `BI_ReportProvider`

---

## 🔧 **Key Features**

### **Report Management**
- **Dynamic Report Generation**: Tạo reports từ datasets và configurations
- **Real-time Data Tracking**: Reactive streams cho report data updates
- **Chart Integration**: ECharts integration với customizable options
- **Time Frame Management**: Flexible time period configurations
- **Dataset Management**: In-memory dataset caching và processing

### **Visualization Support**
- **Multiple Chart Types**: Bar, Line, Pie, Area, Scatter charts
- **Interactive Charts**: Drill-down, zoom, pan capabilities
- **Responsive Design**: Mobile-friendly chart rendering
- **Export Functions**: PDF, Excel, Image export support

---

## 🚀 **Core Properties & Methods**

### **Reactive Streams**
```typescript
reportDataTracking: Subject<any> // Report data updates
reportConfigTracking: Subject<any> // Report configuration updates
```

### **Configuration Objects**
```typescript
globalOptions: ReportGlobalOptions // Global report settings
commonOptions: any // Common chart options
reportDatasetList: any[] // In-memory datasets
```

### **Time Frame Management**
```typescript
interface TimeConfig {
    Type: 'Relative' | 'Absolute';
    IsPastDate: boolean;
    Period: 'Day' | 'Week' | 'Month' | 'Year';
    Amount: number;
}
```

---

## 🎨 **Usage Examples**

### **1. Basic Report Generation**
```typescript
export class ReportDashboardComponent implements OnInit {
    reports: any[] = [];
    reportData: any = {};
    
    constructor(private reportService: ReportService) {}
    
    ngOnInit() {
        this.setupReportTracking();
        this.loadDashboardReports();
    }
    
    setupReportTracking() {
        // Subscribe to report data updates
        this.reportService.reportDataTracking.subscribe(data => {
            this.handleReportDataUpdate(data);
        });
        
        // Subscribe to report config updates
        this.reportService.reportConfigTracking.subscribe(config => {
            this.handleReportConfigUpdate(config);
        });
    }
    
    async loadDashboardReports() {
        try {
            const reportConfigs = [
                { type: 'SalesOverview', timeFrame: 'ThisMonth' },
                { type: 'TopProducts', limit: 10 },
                { type: 'CustomerAnalytics', period: 'Quarter' }
            ];
            
            for (const config of reportConfigs) {
                const reportData = await this.generateReport(config);
                this.reports.push({
                    config: config,
                    data: reportData,
                    chartOptions: this.buildChartOptions(config, reportData)
                });
            }
            
        } catch (error) {
            console.error('Failed to load dashboard reports:', error);
            this.env.showErrorMessage('Failed to load reports');
        }
    }
    
    async generateReport(config: any): Promise<any> {
        // Set global time frame
        this.reportService.globalOptions.TimeFrame = this.buildTimeFrame(config.timeFrame);
        
        // Get dataset for report type
        const dataset = this.reportService.getDatasetByType(config.type);
        
        if (!dataset) {
            // Load data from server
            const serverData = await this.loadReportDataFromServer(config);
            return this.processReportData(serverData, config);
        }
        
        return this.processReportData(dataset.data, config);
    }
    
    private buildTimeFrame(timeFrame: string): any {
        const timeFrames = {
            'Today': {
                From: { Type: 'Relative', IsPastDate: true, Period: 'Day', Amount: 0 },
                To: { Type: 'Relative', IsPastDate: true, Period: 'Day', Amount: 0 }
            },
            'ThisWeek': {
                From: { Type: 'Relative', IsPastDate: true, Period: 'Week', Amount: 0 },
                To: { Type: 'Relative', IsPastDate: true, Period: 'Day', Amount: 0 }
            },
            'ThisMonth': {
                From: { Type: 'Relative', IsPastDate: true, Period: 'Month', Amount: 0 },
                To: { Type: 'Relative', IsPastDate: true, Period: 'Day', Amount: 0 }
            }
        };
        
        return timeFrames[timeFrame] || timeFrames['ThisMonth'];
    }
}
```

### **2. Interactive Chart Component**
```typescript
export class InteractiveReportComponent {
    @Input() reportConfig: any;
    @Input() reportData: any;
    
    chartOptions: any = {};
    selectedDataPoint: any = null;
    
    constructor(private reportService: ReportService) {}
    
    ngOnInit() {
        this.buildInteractiveChart();
    }
    
    buildInteractiveChart() {
        this.chartOptions = {
            ...this.reportService.commonOptions,
            title: {
                text: this.reportConfig.title,
                left: 'center'
            },
            tooltip: {
                trigger: 'item',
                formatter: this.formatTooltip.bind(this)
            },
            legend: {
                orient: 'vertical',
                left: 'left'
            },
            series: [{
                name: this.reportConfig.seriesName,
                type: this.reportConfig.chartType || 'bar',
                data: this.processChartData(this.reportData),
                emphasis: {
                    itemStyle: {
                        shadowBlur: 10,
                        shadowOffsetX: 0,
                        shadowColor: 'rgba(0, 0, 0, 0.5)'
                    }
                }
            }]
        };
        
        // Add drill-down capability
        if (this.reportConfig.enableDrillDown) {
            this.chartOptions.series[0].cursor = 'pointer';
        }
    }
    
    onChartClick(event: any) {
        if (this.reportConfig.enableDrillDown) {
            this.handleDrillDown(event.data);
        }
        
        this.selectedDataPoint = event.data;
        
        // Emit selection event
        this.reportService.reportDataTracking.next({
            type: 'selection',
            reportId: this.reportConfig.id,
            selectedData: event.data
        });
    }
    
    async handleDrillDown(dataPoint: any) {
        try {
            const drillDownConfig = {
                ...this.reportConfig.drillDownConfig,
                parentData: dataPoint,
                filters: {
                    [this.reportConfig.drillDownField]: dataPoint.id
                }
            };
            
            const drillDownData = await this.reportService.generateDrillDownReport(drillDownConfig);
            
            // Navigate to drill-down view or update current chart
            this.showDrillDownReport(drillDownData, drillDownConfig);
            
        } catch (error) {
            console.error('Drill-down failed:', error);
            this.env.showErrorMessage('Failed to load detailed data');
        }
    }
    
    private formatTooltip(params: any): string {
        const data = params.data;
        
        return `
            <div class="report-tooltip">
                <strong>${params.name}</strong><br/>
                Value: ${this.formatValue(data.value)}<br/>
                Percentage: ${data.percentage}%
                ${data.trend ? `<br/>Trend: ${data.trend}` : ''}
            </div>
        `;
    }
    
    private processChartData(rawData: any[]): any[] {
        return rawData.map(item => ({
            name: item.name || item.label,
            value: item.value || item.count,
            id: item.id,
            percentage: this.calculatePercentage(item.value, rawData),
            trend: item.trend,
            color: this.getItemColor(item)
        }));
    }
}
```

### **3. Report Export Service**
```typescript
export class ReportExportService {
    constructor(private reportService: ReportService) {}
    
    async exportReportToPDF(reportConfig: any, chartElement: HTMLElement): Promise<void> {
        try {
            // Import jsPDF dynamically
            const jsPDF = await import('jspdf');
            const html2canvas = await import('html2canvas');
            
            // Capture chart as image
            const canvas = await html2canvas.default(chartElement);
            const imgData = canvas.toDataURL('image/png');
            
            // Create PDF
            const pdf = new jsPDF.default();
            
            // Add title
            pdf.setFontSize(16);
            pdf.text(reportConfig.title, 20, 20);
            
            // Add chart image
            const imgWidth = 170;
            const imgHeight = (canvas.height * imgWidth) / canvas.width;
            pdf.addImage(imgData, 'PNG', 20, 30, imgWidth, imgHeight);
            
            // Add metadata
            this.addReportMetadata(pdf, reportConfig, imgHeight + 40);
            
            // Save PDF
            pdf.save(`${reportConfig.title}_${new Date().toISOString().split('T')[0]}.pdf`);
            
            this.env.showMessage('Report exported to PDF successfully', 'success');
            
        } catch (error) {
            console.error('PDF export failed:', error);
            this.env.showErrorMessage('Failed to export PDF');
        }
    }
    
    async exportReportToExcel(reportData: any[], reportConfig: any): Promise<void> {
        try {
            // Import xlsx dynamically
            const XLSX = await import('xlsx');
            
            // Prepare data for Excel
            const excelData = this.prepareExcelData(reportData, reportConfig);
            
            // Create workbook
            const wb = XLSX.utils.book_new();
            
            // Add main data sheet
            const ws = XLSX.utils.json_to_sheet(excelData.mainData);
            XLSX.utils.book_append_sheet(wb, ws, 'Report Data');
            
            // Add summary sheet if available
            if (excelData.summaryData) {
                const summaryWs = XLSX.utils.json_to_sheet(excelData.summaryData);
                XLSX.utils.book_append_sheet(wb, summaryWs, 'Summary');
            }
            
            // Save Excel file
            XLSX.writeFile(wb, `${reportConfig.title}_${new Date().toISOString().split('T')[0]}.xlsx`);
            
            this.env.showMessage('Report exported to Excel successfully', 'success');
            
        } catch (error) {
            console.error('Excel export failed:', error);
            this.env.showErrorMessage('Failed to export Excel');
        }
    }
    
    async exportReportToImage(chartElement: HTMLElement, format: 'png' | 'jpg' = 'png'): Promise<void> {
        try {
            const html2canvas = await import('html2canvas');
            
            const canvas = await html2canvas.default(chartElement, {
                backgroundColor: '#ffffff',
                scale: 2 // Higher quality
            });
            
            // Create download link
            const link = document.createElement('a');
            link.download = `report_${Date.now()}.${format}`;
            link.href = canvas.toDataURL(`image/${format}`);
            
            // Trigger download
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            
            this.env.showMessage('Report exported as image successfully', 'success');
            
        } catch (error) {
            console.error('Image export failed:', error);
            this.env.showErrorMessage('Failed to export image');
        }
    }
    
    private prepareExcelData(reportData: any[], reportConfig: any): any {
        const mainData = reportData.map(item => ({
            'Name': item.name || item.label,
            'Value': item.value || item.count,
            'Percentage': item.percentage ? `${item.percentage}%` : '',
            'Category': item.category || '',
            'Date': item.date ? new Date(item.date).toLocaleDateString() : ''
        }));
        
        const summaryData = [
            { 'Metric': 'Total Records', 'Value': reportData.length },
            { 'Metric': 'Report Generated', 'Value': new Date().toLocaleString() },
            { 'Metric': 'Report Type', 'Value': reportConfig.type },
            { 'Metric': 'Time Period', 'Value': this.getTimePeriodDescription(reportConfig) }
        ];
        
        return { mainData, summaryData };
    }
}
```

### **4. Real-time Report Updates**
```typescript
export class RealTimeReportService extends ReportService {
    private updateInterval: any;
    private websocketConnection: any;
    
    constructor() {
        super();
        this.setupRealTimeUpdates();
    }
    
    setupRealTimeUpdates() {
        // Setup WebSocket connection for real-time data
        this.websocketConnection = new WebSocket(environment.websocketUrl);
        
        this.websocketConnection.onmessage = (event) => {
            const data = JSON.parse(event.data);
            this.handleRealTimeUpdate(data);
        };
        
        // Setup periodic updates as fallback
        this.updateInterval = setInterval(() => {
            this.refreshActiveReports();
        }, 30000); // Update every 30 seconds
    }
    
    private handleRealTimeUpdate(data: any) {
        switch (data.type) {
            case 'sales_update':
                this.updateSalesReports(data.payload);
                break;
            case 'inventory_update':
                this.updateInventoryReports(data.payload);
                break;
            case 'customer_update':
                this.updateCustomerReports(data.payload);
                break;
        }
    }
    
    private updateSalesReports(salesData: any) {
        // Find and update sales-related datasets
        const salesDatasets = this.reportDatasetList.filter(ds => 
            ds.type === 'SalesOrder' || ds.code.includes('Sales')
        );
        
        salesDatasets.forEach(dataset => {
            // Update dataset with new data
            this.updateDataset(dataset, salesData);
            
            // Notify subscribers
            this.reportDataTracking.next({
                type: 'data_update',
                datasetId: dataset.id,
                updatedData: salesData
            });
        });
    }
    
    enableAutoRefresh(reportId: string, intervalMs: number = 60000) {
        const refreshTimer = setInterval(async () => {
            try {
                const updatedData = await this.refreshReportData(reportId);
                
                this.reportDataTracking.next({
                    type: 'auto_refresh',
                    reportId: reportId,
                    data: updatedData
                });
                
            } catch (error) {
                console.error(`Auto refresh failed for report ${reportId}:`, error);
            }
        }, intervalMs);
        
        // Store timer reference for cleanup
        this.storeRefreshTimer(reportId, refreshTimer);
    }
    
    disableAutoRefresh(reportId: string) {
        const timer = this.getRefreshTimer(reportId);
        if (timer) {
            clearInterval(timer);
            this.removeRefreshTimer(reportId);
        }
    }
    
    ngOnDestroy() {
        // Cleanup
        if (this.updateInterval) {
            clearInterval(this.updateInterval);
        }
        
        if (this.websocketConnection) {
            this.websocketConnection.close();
        }
        
        // Clear all refresh timers
        this.clearAllRefreshTimers();
    }
}
```

---

## 🔧 **Advanced Features**

### **1. Custom Chart Types**
```typescript
export class CustomChartService {
    constructor(private reportService: ReportService) {}
    
    createGaugeChart(data: any, config: any): any {
        return {
            series: [{
                name: config.name,
                type: 'gauge',
                detail: { formatter: '{value}%' },
                data: [{
                    value: data.value,
                    name: data.name
                }],
                min: config.min || 0,
                max: config.max || 100,
                axisLine: {
                    lineStyle: {
                        width: 30,
                        color: [
                            [0.3, '#fd666d'],
                            [0.7, '#37a2da'],
                            [1, '#67e0e3']
                        ]
                    }
                }
            }]
        };
    }
    
    createHeatmapChart(data: any[], config: any): any {
        return {
            tooltip: {
                position: 'top'
            },
            grid: {
                height: '50%',
                top: '10%'
            },
            xAxis: {
                type: 'category',
                data: config.xAxisData,
                splitArea: { show: true }
            },
            yAxis: {
                type: 'category',
                data: config.yAxisData,
                splitArea: { show: true }
            },
            visualMap: {
                min: config.min,
                max: config.max,
                calculable: true,
                orient: 'horizontal',
                left: 'center',
                bottom: '15%'
            },
            series: [{
                name: config.name,
                type: 'heatmap',
                data: data,
                label: {
                    show: true
                },
                emphasis: {
                    itemStyle: {
                        shadowBlur: 10,
                        shadowColor: 'rgba(0, 0, 0, 0.5)'
                    }
                }
            }]
        };
    }
}
```

### **2. Report Scheduling**
```typescript
export class ReportSchedulingService {
    constructor(private reportService: ReportService) {}
    
    scheduleReport(reportConfig: any, schedule: any): Promise<any> {
        return new Promise((resolve, reject) => {
            const scheduledReport = {
                id: this.generateScheduleId(),
                reportConfig: reportConfig,
                schedule: schedule, // { frequency: 'daily', time: '09:00', recipients: [...] }
                isActive: true,
                createdDate: new Date(),
                nextRun: this.calculateNextRun(schedule)
            };
            
            // Store scheduled report
            this.storeScheduledReport(scheduledReport);
            
            // Setup timer
            this.setupReportTimer(scheduledReport);
            
            resolve(scheduledReport);
        });
    }
    
    private async executeScheduledReport(scheduledReport: any) {
        try {
            // Generate report
            const reportData = await this.reportService.generateReport(scheduledReport.reportConfig);
            
            // Export report
            const exportedFile = await this.exportReport(reportData, scheduledReport.reportConfig);
            
            // Send to recipients
            await this.sendReportToRecipients(exportedFile, scheduledReport.schedule.recipients);
            
            // Update next run time
            scheduledReport.nextRun = this.calculateNextRun(scheduledReport.schedule);
            scheduledReport.lastRun = new Date();
            
            this.updateScheduledReport(scheduledReport);
            
        } catch (error) {
            console.error('Scheduled report execution failed:', error);
            this.handleScheduledReportError(scheduledReport, error);
        }
    }
}
```

---

## 📋 **Best Practices**

### **1. Performance Optimization**
```typescript
// ✅ Optimize large datasets
processLargeDataset(data: any[]): any[] {
    // Use pagination for large datasets
    const pageSize = 1000;
    const chunks = [];
    
    for (let i = 0; i < data.length; i += pageSize) {
        chunks.push(data.slice(i, i + pageSize));
    }
    
    return chunks.map(chunk => this.processDataChunk(chunk));
}

// ✅ Use virtual scrolling for large reports
// In template: <cdk-virtual-scroll-viewport>
```

### **2. Memory Management**
```typescript
// ✅ Clean up subscriptions and timers
ngOnDestroy() {
    this.reportService.reportDataTracking.unsubscribe();
    this.reportService.reportConfigTracking.unsubscribe();
    
    if (this.refreshTimer) {
        clearInterval(this.refreshTimer);
    }
}
```

### **3. Error Handling**
```typescript
// ✅ Graceful error handling
async generateReportSafely(config: any) {
    try {
        return await this.reportService.generateReport(config);
    } catch (error) {
        if (error.code === 'INSUFFICIENT_DATA') {
            this.env.showMessage('Not enough data for this report', 'warning');
        } else if (error.code === 'TIMEOUT') {
            this.env.showMessage('Report generation timed out. Please try again.', 'warning');
        } else {
            this.env.showErrorMessage('Failed to generate report');
        }
        
        // Return empty dataset
        return { data: [], message: 'No data available' };
    }
}
```

ReportService cung cấp comprehensive business intelligence solution với advanced charting, real-time updates, export capabilities, và scheduling features cho enterprise reporting needs.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
