# NumberFriendly Pipe Documentation

## 📋 **Tổng quan**

`NumberFriendly` pipe được sử dụng để format numbers thành friendly, human-readable format. Pipe này sử dụng `lib.currencyFormatFriendly()` để hiển thị numbers với proper formatting và units.

**Pipe Name**: `numberFriendly`  
**Location**: `src/app/pipes/friendly-format.pipe.ts`  
**Type**: Pure Pipe

---

## 🔧 **Signature**

```typescript
transform(value: any): string
```

### **Parameters**
- `value` (any): Number value cần format

### **Returns**
- `string`: Formatted number string hoặc original value nếu không phải number

---

## 🚀 **Basic Usage**

### **Simple Number Formatting**
```typescript
// Component
export class SalesReportPage extends PageBase {
    salesData = {
        revenue: 1234567.89,
        orders: 1500,
        customers: 850,
        avgOrderValue: 823.45,
        growth: 0.15
    };
    
    products = [
        { name: 'Laptop', price: 25000000, quantity: 50 },
        { name: 'Phone', price: 15000000, quantity: 120 },
        { name: 'Tablet', price: 8000000, quantity: 75 }
    ];
}
```

```html
<!-- Template -->
<div class="sales-dashboard">
    <div class="stats-grid">
        <div class="stat-card">
            <h3>Revenue</h3>
            <p class="stat-value">{{ salesData.revenue | numberFriendly }}</p>
        </div>
        
        <div class="stat-card">
            <h3>Orders</h3>
            <p class="stat-value">{{ salesData.orders | numberFriendly }}</p>
        </div>
        
        <div class="stat-card">
            <h3>Customers</h3>
            <p class="stat-value">{{ salesData.customers | numberFriendly }}</p>
        </div>
        
        <div class="stat-card">
            <h3>Avg Order Value</h3>
            <p class="stat-value">{{ salesData.avgOrderValue | numberFriendly }}</p>
        </div>
    </div>
    
    <div class="product-list">
        <h3>Products</h3>
        <div class="product-item" *ngFor="let product of products">
            <span class="product-name">{{ product.name }}</span>
            <span class="product-price">{{ product.price | numberFriendly }}</span>
            <span class="product-quantity">Qty: {{ product.quantity | numberFriendly }}</span>
        </div>
    </div>
</div>
```

### **Financial Data Display**
```typescript
export class FinancialReportPage extends PageBase {
    financialData = {
        totalAssets: 50000000000,      // 50 billion
        totalLiabilities: 25000000000,  // 25 billion
        netIncome: 5000000000,         // 5 billion
        cashFlow: 1500000000,          // 1.5 billion
        marketCap: 100000000000        // 100 billion
    };
    
    quarterlyResults = [
        { quarter: 'Q1 2024', revenue: 12500000000, profit: 1250000000 },
        { quarter: 'Q2 2024', revenue: 13200000000, profit: 1400000000 },
        { quarter: 'Q3 2024', revenue: 14100000000, profit: 1600000000 },
        { quarter: 'Q4 2024', revenue: 15200000000, profit: 1750000000 }
    ];
}
```

```html
<div class="financial-report">
    <div class="financial-summary">
        <h2>Financial Summary</h2>
        
        <div class="summary-grid">
            <div class="summary-item">
                <label>Total Assets</label>
                <span class="amount positive">{{ financialData.totalAssets | numberFriendly }}</span>
            </div>
            
            <div class="summary-item">
                <label>Total Liabilities</label>
                <span class="amount negative">{{ financialData.totalLiabilities | numberFriendly }}</span>
            </div>
            
            <div class="summary-item">
                <label>Net Income</label>
                <span class="amount positive">{{ financialData.netIncome | numberFriendly }}</span>
            </div>
            
            <div class="summary-item">
                <label>Cash Flow</label>
                <span class="amount positive">{{ financialData.cashFlow | numberFriendly }}</span>
            </div>
            
            <div class="summary-item">
                <label>Market Cap</label>
                <span class="amount highlight">{{ financialData.marketCap | numberFriendly }}</span>
            </div>
        </div>
    </div>
    
    <div class="quarterly-results">
        <h3>Quarterly Results</h3>
        <div class="results-table">
            <div class="table-header">
                <span>Quarter</span>
                <span>Revenue</span>
                <span>Profit</span>
            </div>
            
            <div class="table-row" *ngFor="let result of quarterlyResults">
                <span>{{ result.quarter }}</span>
                <span>{{ result.revenue | numberFriendly }}</span>
                <span>{{ result.profit | numberFriendly }}</span>
            </div>
        </div>
    </div>
</div>
```

---

## 🎨 **Advanced Usage**

### **Inventory Management**
```typescript
export class InventoryPage extends PageBase {
    inventoryItems = [
        { 
            sku: 'LAP001', 
            name: 'Gaming Laptop', 
            quantity: 25, 
            unitPrice: 35000000, 
            totalValue: 875000000 
        },
        { 
            sku: 'PHN001', 
            name: 'Smartphone', 
            quantity: 150, 
            unitPrice: 18000000, 
            totalValue: 2700000000 
        },
        { 
            sku: 'TAB001', 
            name: 'Tablet', 
            quantity: 75, 
            unitPrice: 12000000, 
            totalValue: 900000000 
        }
    ];
    
    get totalInventoryValue(): number {
        return this.inventoryItems.reduce((sum, item) => sum + item.totalValue, 0);
    }
    
    get lowStockItems(): any[] {
        return this.inventoryItems.filter(item => item.quantity < 30);
    }
}
```

```html
<div class="inventory-management">
    <div class="inventory-summary">
        <div class="summary-card">
            <h3>Total Inventory Value</h3>
            <p class="total-value">{{ totalInventoryValue | numberFriendly }}</p>
        </div>
        
        <div class="summary-card">
            <h3>Low Stock Items</h3>
            <p class="low-stock-count">{{ lowStockItems.length }} items</p>
        </div>
    </div>
    
    <div class="inventory-table">
        <div class="table-header">
            <span>SKU</span>
            <span>Product</span>
            <span>Quantity</span>
            <span>Unit Price</span>
            <span>Total Value</span>
            <span>Status</span>
        </div>
        
        <div class="table-row" 
             *ngFor="let item of inventoryItems"
             [class.low-stock]="item.quantity < 30">
            <span>{{ item.sku }}</span>
            <span>{{ item.name }}</span>
            <span class="quantity">{{ item.quantity | numberFriendly }}</span>
            <span class="price">{{ item.unitPrice | numberFriendly }}</span>
            <span class="total">{{ item.totalValue | numberFriendly }}</span>
            <span class="status">
                <ion-badge [color]="item.quantity < 30 ? 'warning' : 'success'">
                    {{ item.quantity < 30 ? 'Low Stock' : 'In Stock' }}
                </ion-badge>
            </span>
        </div>
    </div>
</div>
```

### **Analytics Dashboard**
```typescript
export class AnalyticsDashboardPage extends PageBase {
    websiteStats = {
        totalVisitors: 2500000,
        pageViews: 15000000,
        bounceRate: 0.35,
        avgSessionDuration: 180, // seconds
        conversionRate: 0.025
    };
    
    trafficSources = [
        { source: 'Organic Search', visitors: 1250000, percentage: 50 },
        { source: 'Direct', visitors: 500000, percentage: 20 },
        { source: 'Social Media', visitors: 375000, percentage: 15 },
        { source: 'Paid Ads', visitors: 250000, percentage: 10 },
        { source: 'Referral', visitors: 125000, percentage: 5 }
    ];
    
    formatPercentage(value: number): string {
        return (value * 100).toFixed(1) + '%';
    }
    
    formatDuration(seconds: number): string {
        const minutes = Math.floor(seconds / 60);
        const remainingSeconds = seconds % 60;
        return `${minutes}m ${remainingSeconds}s`;
    }
}
```

```html
<div class="analytics-dashboard">
    <div class="stats-overview">
        <h2>Website Analytics</h2>
        
        <div class="stats-grid">
            <div class="stat-card">
                <div class="stat-icon">
                    <ion-icon name="people-outline"></ion-icon>
                </div>
                <div class="stat-content">
                    <h4>Total Visitors</h4>
                    <p class="stat-number">{{ websiteStats.totalVisitors | numberFriendly }}</p>
                </div>
            </div>
            
            <div class="stat-card">
                <div class="stat-icon">
                    <ion-icon name="eye-outline"></ion-icon>
                </div>
                <div class="stat-content">
                    <h4>Page Views</h4>
                    <p class="stat-number">{{ websiteStats.pageViews | numberFriendly }}</p>
                </div>
            </div>
            
            <div class="stat-card">
                <div class="stat-icon">
                    <ion-icon name="time-outline"></ion-icon>
                </div>
                <div class="stat-content">
                    <h4>Avg Session Duration</h4>
                    <p class="stat-number">{{ formatDuration(websiteStats.avgSessionDuration) }}</p>
                </div>
            </div>
            
            <div class="stat-card">
                <div class="stat-icon">
                    <ion-icon name="trending-up-outline"></ion-icon>
                </div>
                <div class="stat-content">
                    <h4>Conversion Rate</h4>
                    <p class="stat-number">{{ formatPercentage(websiteStats.conversionRate) }}</p>
                </div>
            </div>
        </div>
    </div>
    
    <div class="traffic-sources">
        <h3>Traffic Sources</h3>
        <div class="source-list">
            <div class="source-item" *ngFor="let source of trafficSources">
                <div class="source-info">
                    <span class="source-name">{{ source.source }}</span>
                    <span class="source-visitors">{{ source.visitors | numberFriendly }} visitors</span>
                </div>
                <div class="source-percentage">
                    {{ source.percentage }}%
                </div>
            </div>
        </div>
    </div>
</div>
```

### **E-commerce Product Display**
```typescript
export class ProductDisplayPage extends PageBase {
    product = {
        id: 1,
        name: 'Premium Laptop',
        originalPrice: 45000000,
        salePrice: 38000000,
        discount: 7000000,
        stockQuantity: 25,
        soldCount: 150,
        rating: 4.8,
        reviewCount: 89
    };
    
    get discountPercentage(): number {
        return (this.product.discount / this.product.originalPrice) * 100;
    }
    
    get savings(): number {
        return this.product.originalPrice - this.product.salePrice;
    }
}
```

```html
<div class="product-display">
    <div class="product-header">
        <h1>{{ product.name }}</h1>
        <div class="product-rating">
            <span class="rating">{{ product.rating }}/5</span>
            <span class="review-count">({{ product.reviewCount | numberFriendly }} reviews)</span>
        </div>
    </div>
    
    <div class="product-pricing">
        <div class="price-section">
            <div class="current-price">
                <span class="sale-price">{{ product.salePrice | numberFriendly }}</span>
                <span class="currency">VND</span>
            </div>
            
            <div class="original-price">
                <span class="crossed-out">{{ product.originalPrice | numberFriendly }} VND</span>
            </div>
            
            <div class="savings-info">
                <span class="savings-amount">Save {{ savings | numberFriendly }} VND</span>
                <span class="discount-percentage">({{ discountPercentage.toFixed(0) }}% off)</span>
            </div>
        </div>
    </div>
    
    <div class="product-availability">
        <div class="stock-info">
            <ion-icon name="cube-outline"></ion-icon>
            <span>{{ product.stockQuantity | numberFriendly }} in stock</span>
        </div>
        
        <div class="sales-info">
            <ion-icon name="trending-up-outline"></ion-icon>
            <span>{{ product.soldCount | numberFriendly }} sold</span>
        </div>
    </div>
</div>
```

---

## 🎨 **Styling**

### **Number Display Styling**
```scss
.stat-value, .stat-number {
    font-size: 2rem;
    font-weight: 700;
    color: var(--ion-color-primary);
    margin: 8px 0;
}

.amount {
    font-weight: 600;
    font-size: 1.25rem;
    
    &.positive {
        color: var(--ion-color-success);
    }
    
    &.negative {
        color: var(--ion-color-danger);
    }
    
    &.highlight {
        color: var(--ion-color-primary);
        font-weight: 700;
    }
}

.price-section {
    .sale-price {
        font-size: 2.5rem;
        font-weight: 700;
        color: var(--ion-color-danger);
    }
    
    .original-price {
        .crossed-out {
            text-decoration: line-through;
            color: var(--ion-color-medium);
            font-size: 1.25rem;
        }
    }
    
    .savings-amount {
        color: var(--ion-color-success);
        font-weight: 600;
    }
    
    .discount-percentage {
        color: var(--ion-color-warning);
        font-weight: 500;
    }
}

.inventory-table {
    .quantity, .price, .total {
        font-family: 'Courier New', monospace;
        font-weight: 600;
    }
    
    .total {
        color: var(--ion-color-primary);
    }
    
    .table-row.low-stock {
        background-color: rgba(var(--ion-color-warning-rgb), 0.1);
        border-left: 3px solid var(--ion-color-warning);
    }
}
```

---

## 🔧 **Best Practices**

### **1. Input Validation**
```typescript
// ✅ Đúng - Pipe handles non-numeric values gracefully
// Returns original value if not numeric

// ✅ Đúng - Additional validation in component
formatNumber(value: any): string {
    if (typeof value === 'number' && !isNaN(value)) {
        return value | numberFriendly; // In template
    }
    return 'N/A';
}
```

### **2. Null/Undefined Handling**
```html
<!-- ✅ Đúng - Handle null values -->
<span *ngIf="item.amount !== null && item.amount !== undefined">
    {{ item.amount | numberFriendly }}
</span>
<span *ngIf="item.amount === null || item.amount === undefined">
    No data
</span>
```

### **3. Performance with Large Lists**
```typescript
// ✅ Đúng - Use trackBy for performance
trackByItem(index: number, item: any): any {
    return item.id;
}
```

---

## 🚨 **Common Use Cases**

### **1. Financial Reports**
```typescript
export class FinancialReportsPage extends PageBase {
    // Display revenue, profit, expenses with friendly formatting
    financialMetrics: FinancialMetric[] = [];
}
```

### **2. Sales Dashboards**
```typescript
export class SalesDashboardPage extends PageBase {
    // Show sales figures, order counts, customer numbers
    salesData: SalesData = {};
}
```

### **3. Inventory Management**
```typescript
export class InventoryPage extends PageBase {
    // Display quantities, prices, total values
    inventoryItems: InventoryItem[] = [];
}
```

### **4. Analytics & Metrics**
```typescript
export class AnalyticsPage extends PageBase {
    // Show website traffic, user counts, engagement metrics
    analyticsData: AnalyticsData = {};
}
```

---

## 🔄 **Integration with Global Functions**

The pipe uses `lib.currencyFormatFriendly()` which provides:
- Proper thousand separators
- Decimal formatting
- Currency-appropriate formatting
- Handles large numbers gracefully

---

## ⚠️ **Limitations**

1. **Currency Assumption**: Formats as currency by default
2. **Fixed Format**: Cannot customize number format
3. **No Locale Support**: Uses fixed formatting rules
4. **No Unit Conversion**: Doesn't convert units (K, M, B)

---

## 🔄 **Alternative Implementations**

### **Custom Number Formatting**
```typescript
// For more control, create custom formatters
formatLargeNumber(value: number): string {
    if (value >= 1000000000) {
        return (value / 1000000000).toFixed(1) + 'B';
    } else if (value >= 1000000) {
        return (value / 1000000).toFixed(1) + 'M';
    } else if (value >= 1000) {
        return (value / 1000).toFixed(1) + 'K';
    }
    return value.toString();
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
