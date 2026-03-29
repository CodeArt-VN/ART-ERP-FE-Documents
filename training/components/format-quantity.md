# FormatQuantity Component Documentation

## 📋 **Tổng quan**

`FormatQuantityComponent` là component hiển thị quantity với format đặc biệt, hỗ trợ split quantity list để hiển thị số lượng theo các đơn vị khác nhau (ví dụ: thùng, hộp, cái).

**Selector**: `app-format-quantity`  
**Location**: `src/app/components/format-quantity/format-quantity.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() splitQuantityList: any[];       // List of quantity breakdowns
@Input() quantity: number;               // Total quantity
```

---

## 🚀 **Basic Usage**

### **Simple Quantity Display**
```typescript
// Component
export class ProductQuantityPage extends PageBase {
    totalQuantity = 1250;
    
    quantityBreakdown = [
        { Unit: 'Thùng', Quantity: 10, UnitQuantity: 100 },  // 10 thùng x 100 = 1000
        { Unit: 'Hộp', Quantity: 5, UnitQuantity: 50 },     // 5 hộp x 50 = 250
        { Unit: 'Cái', Quantity: 0, UnitQuantity: 1 }       // 0 cái
    ];
}
```

```html
<!-- Template -->
<div class="quantity-display">
    <h4>Product Quantity</h4>
    
    <app-format-quantity
        [quantity]="totalQuantity"
        [splitQuantityList]="quantityBreakdown">
    </app-format-quantity>
</div>
```

### **Inventory Display**
```typescript
export class InventoryItemPage extends PageBase {
    inventoryItem = {
        ProductName: 'Coca Cola 330ml',
        TotalQuantity: 2400,
        QuantityBreakdown: [
            { Unit: 'Pallet', Quantity: 2, UnitQuantity: 1000, UnitName: 'Pallet' },
            { Unit: 'Carton', Quantity: 16, UnitQuantity: 24, UnitName: 'Thùng' },
            { Unit: 'Piece', Quantity: 8, UnitQuantity: 1, UnitName: 'Lon' }
        ]
    };

    calculateTotalFromBreakdown(): number {
        return this.inventoryItem.QuantityBreakdown.reduce((total, item) => {
            return total + (item.Quantity * item.UnitQuantity);
        }, 0);
    }
}
```

```html
<ion-card>
    <ion-card-header>
        <ion-card-title>{{ inventoryItem.ProductName }}</ion-card-title>
    </ion-card-header>
    
    <ion-card-content>
        <div class="inventory-quantity">
            <app-format-quantity
                [quantity]="inventoryItem.TotalQuantity"
                [splitQuantityList]="inventoryItem.QuantityBreakdown">
            </app-format-quantity>
        </div>
        
        <div class="quantity-verification">
            <small>
                Calculated: {{ calculateTotalFromBreakdown() }} 
                ({{ inventoryItem.TotalQuantity === calculateTotalFromBreakdown() ? 'Match' : 'Mismatch' }})
            </small>
        </div>
    </ion-card-content>
</ion-card>
```

---

## 🎨 **Advanced Display Formats**

### **Warehouse Quantity Display**
```typescript
export class WarehouseQuantityPage extends PageBase {
    warehouseStock = {
        ProductCode: 'PRD001',
        ProductName: 'iPhone 15 Pro',
        Locations: [
            {
                LocationCode: 'A-01-01',
                TotalQuantity: 150,
                QuantityBreakdown: [
                    { Unit: 'Box', Quantity: 15, UnitQuantity: 10, UnitName: 'Hộp' },
                    { Unit: 'Piece', Quantity: 0, UnitQuantity: 1, UnitName: 'Chiếc' }
                ]
            },
            {
                LocationCode: 'A-01-02', 
                TotalQuantity: 87,
                QuantityBreakdown: [
                    { Unit: 'Box', Quantity: 8, UnitQuantity: 10, UnitName: 'Hộp' },
                    { Unit: 'Piece', Quantity: 7, UnitQuantity: 1, UnitName: 'Chiếc' }
                ]
            }
        ]
    };

    getTotalStock(): number {
        return this.warehouseStock.Locations.reduce((total, location) => {
            return total + location.TotalQuantity;
        }, 0);
    }
}
```

```html
<div class="warehouse-stock">
    <div class="stock-header">
        <h3>{{ warehouseStock.ProductName }}</h3>
        <div class="total-stock">
            Total: {{ getTotalStock() }} units
        </div>
    </div>
    
    <div class="location-stock" *ngFor="let location of warehouseStock.Locations">
        <div class="location-info">
            <strong>{{ location.LocationCode }}</strong>
        </div>
        
        <app-format-quantity
            [quantity]="location.TotalQuantity"
            [splitQuantityList]="location.QuantityBreakdown">
        </app-format-quantity>
    </div>
</div>
```

### **Order Quantity Display**
```typescript
export class OrderQuantityPage extends PageBase {
    orderItems = [
        {
            ProductName: 'Pepsi 500ml',
            OrderedQuantity: 720,
            DeliveredQuantity: 600,
            RemainingQuantity: 120,
            OrderedBreakdown: [
                { Unit: 'Case', Quantity: 30, UnitQuantity: 24, UnitName: 'Thùng' }
            ],
            DeliveredBreakdown: [
                { Unit: 'Case', Quantity: 25, UnitQuantity: 24, UnitName: 'Thùng' }
            ],
            RemainingBreakdown: [
                { Unit: 'Case', Quantity: 5, UnitQuantity: 24, UnitName: 'Thùng' }
            ]
        }
    ];
}
```

```html
<div class="order-quantities" *ngFor="let item of orderItems">
    <h4>{{ item.ProductName }}</h4>
    
    <div class="quantity-comparison">
        <div class="quantity-section">
            <label>Ordered:</label>
            <app-format-quantity
                [quantity]="item.OrderedQuantity"
                [splitQuantityList]="item.OrderedBreakdown">
            </app-format-quantity>
        </div>
        
        <div class="quantity-section">
            <label>Delivered:</label>
            <app-format-quantity
                [quantity]="item.DeliveredQuantity"
                [splitQuantityList]="item.DeliveredBreakdown">
            </app-format-quantity>
        </div>
        
        <div class="quantity-section">
            <label>Remaining:</label>
            <app-format-quantity
                [quantity]="item.RemainingQuantity"
                [splitQuantityList]="item.RemainingBreakdown">
            </app-format-quantity>
        </div>
    </div>
</div>
```

---

## 🎨 **UI Customization**

### **Custom Styling**
```scss
// Component SCSS
app-format-quantity {
    .quantity-display {
        display: flex;
        flex-wrap: wrap;
        gap: 8px;
        align-items: center;
        
        .total-quantity {
            font-weight: 600;
            color: var(--ion-color-primary);
            font-size: 1.1rem;
        }
        
        .quantity-breakdown {
            display: flex;
            gap: 12px;
            flex-wrap: wrap;
            
            .unit-quantity {
                display: flex;
                align-items: center;
                gap: 4px;
                padding: 4px 8px;
                background: var(--ion-color-light);
                border-radius: 4px;
                font-size: 0.875rem;
                
                .quantity-value {
                    font-weight: 500;
                    color: var(--ion-color-dark);
                }
                
                .unit-name {
                    color: var(--ion-color-medium);
                }
                
                &.zero-quantity {
                    opacity: 0.5;
                    .quantity-value {
                        color: var(--ion-color-medium);
                    }
                }
            }
        }
        
        .quantity-separator {
            color: var(--ion-color-medium);
            margin: 0 4px;
        }
    }
}
```

### **Compact Display Mode**
```html
<app-format-quantity
    [quantity]="item.quantity"
    [splitQuantityList]="item.breakdown"
    class="compact-mode">
</app-format-quantity>
```

```scss
.compact-mode {
    app-format-quantity {
        .quantity-display {
            .quantity-breakdown {
                .unit-quantity {
                    padding: 2px 6px;
                    font-size: 0.75rem;
                    
                    &.zero-quantity {
                        display: none; // Hide zero quantities in compact mode
                    }
                }
            }
        }
    }
}
```

---

## 🔢 **Quantity Calculations**

### **Unit Conversion Helper**
```typescript
export class QuantityCalculator {
    static calculateBreakdown(totalQuantity: number, unitDefinitions: any[]): any[] {
        let remaining = totalQuantity;
        const breakdown = [];
        
        // Sort by unit quantity descending (largest units first)
        const sortedUnits = unitDefinitions.sort((a, b) => b.UnitQuantity - a.UnitQuantity);
        
        for (const unit of sortedUnits) {
            const quantity = Math.floor(remaining / unit.UnitQuantity);
            remaining = remaining % unit.UnitQuantity;
            
            breakdown.push({
                Unit: unit.Unit,
                UnitName: unit.UnitName,
                Quantity: quantity,
                UnitQuantity: unit.UnitQuantity
            });
        }
        
        return breakdown;
    }
    
    static calculateTotal(breakdown: any[]): number {
        return breakdown.reduce((total, item) => {
            return total + (item.Quantity * item.UnitQuantity);
        }, 0);
    }
    
    static validateBreakdown(totalQuantity: number, breakdown: any[]): boolean {
        const calculatedTotal = this.calculateTotal(breakdown);
        return calculatedTotal === totalQuantity;
    }
}
```

### **Dynamic Quantity Updates**
```typescript
export class DynamicQuantityPage extends PageBase {
    unitDefinitions = [
        { Unit: 'Pallet', UnitName: 'Pallet', UnitQuantity: 1000 },
        { Unit: 'Carton', UnitName: 'Thùng', UnitQuantity: 24 },
        { Unit: 'Piece', UnitName: 'Cái', UnitQuantity: 1 }
    ];
    
    totalQuantity = 0;
    quantityBreakdown: any[] = [];
    
    updateQuantity(newTotal: number) {
        this.totalQuantity = newTotal;
        this.quantityBreakdown = QuantityCalculator.calculateBreakdown(
            newTotal, 
            this.unitDefinitions
        );
    }
    
    updateBreakdownQuantity(unitIndex: number, newQuantity: number) {
        this.quantityBreakdown[unitIndex].Quantity = newQuantity;
        this.totalQuantity = QuantityCalculator.calculateTotal(this.quantityBreakdown);
    }
}
```

---

## 📱 **Mobile Optimization**

### **Responsive Layout**
```scss
// Mobile-specific styles
@media (max-width: 768px) {
    app-format-quantity {
        .quantity-display {
            flex-direction: column;
            align-items: flex-start;
            gap: 6px;
            
            .total-quantity {
                font-size: 1rem;
            }
            
            .quantity-breakdown {
                width: 100%;
                
                .unit-quantity {
                    flex: 1;
                    justify-content: space-between;
                    min-width: 80px;
                }
            }
        }
    }
}
```

### **Touch-Friendly Editing**
```html
<div class="editable-quantity" *ngIf="isEditing">
    <div class="quantity-inputs">
        <div 
            *ngFor="let unit of quantityBreakdown; let i = index"
            class="unit-input">
            <ion-item>
                <ion-label position="stacked">{{ unit.UnitName }}</ion-label>
                <ion-input
                    type="number"
                    [(ngModel)]="unit.Quantity"
                    (ionInput)="updateBreakdownQuantity(i, $event.target.value)">
                </ion-input>
            </ion-item>
        </div>
    </div>
    
    <div class="calculated-total">
        Total: {{ totalQuantity }}
    </div>
</div>
```

---

## 🔧 **Integration Patterns**

### **With Data Tables**
```html
<app-data-table [rows]="inventoryItems" [columns]="columns">
    <app-data-table-column property="quantity" header="Quantity">
        <ng-template #cellTemplate let-row>
            <app-format-quantity
                [quantity]="row.TotalQuantity"
                [splitQuantityList]="row.QuantityBreakdown">
            </app-format-quantity>
        </ng-template>
    </app-data-table-column>
</app-data-table>
```

### **With Forms**
```html
<form [formGroup]="orderForm">
    <div class="quantity-section">
        <ion-label>Order Quantity</ion-label>
        
        <!-- Total quantity input -->
        <ion-input
            type="number"
            formControlName="totalQuantity"
            (ionInput)="updateQuantityBreakdown()">
        </ion-input>
        
        <!-- Breakdown display -->
        <app-format-quantity
            [quantity]="orderForm.get('totalQuantity').value"
            [splitQuantityList]="calculatedBreakdown">
        </app-format-quantity>
    </div>
</form>
```

---

## 🔧 **Best Practices**

### **1. Data Validation**
```typescript
// ✅ Đúng - Validate quantity data
validateQuantityData(quantity: number, breakdown: any[]): boolean {
    if (!quantity || quantity < 0) return false;
    if (!breakdown || breakdown.length === 0) return false;
    
    const calculatedTotal = breakdown.reduce((sum, item) => {
        return sum + (item.Quantity * item.UnitQuantity);
    }, 0);
    
    return calculatedTotal === quantity;
}
```

### **2. Performance Optimization**
```typescript
// ✅ Đúng - Cache calculations
private calculationCache = new Map<string, any[]>();

getQuantityBreakdown(quantity: number, units: any[]): any[] {
    const cacheKey = `${quantity}-${JSON.stringify(units)}`;
    
    if (this.calculationCache.has(cacheKey)) {
        return this.calculationCache.get(cacheKey);
    }
    
    const breakdown = this.calculateBreakdown(quantity, units);
    this.calculationCache.set(cacheKey, breakdown);
    return breakdown;
}
```

### **3. Error Handling**
```typescript
// ✅ Đúng - Handle edge cases
displayQuantity(quantity: number, breakdown: any[]): string {
    try {
        if (!quantity && quantity !== 0) return 'N/A';
        if (quantity === 0) return '0';
        if (!breakdown || breakdown.length === 0) return quantity.toString();
        
        return this.formatQuantityBreakdown(breakdown);
    } catch (error) {
        console.error('Quantity display error:', error);
        return quantity?.toString() || 'Error';
    }
}
```

---

## 🚨 **Common Use Cases**

### **1. Inventory Management**
```typescript
export class InventoryManagement {
    displayStockLevels(product: any) {
        return {
            current: product.CurrentStock,
            breakdown: product.StockBreakdown,
            minimum: product.MinimumStock,
            maximum: product.MaximumStock
        };
    }
}
```

### **2. Order Processing**
```typescript
export class OrderProcessing {
    calculateOrderQuantities(orderItems: any[]) {
        return orderItems.map(item => ({
            ...item,
            breakdown: this.calculateBreakdown(item.quantity, item.unitDefinitions)
        }));
    }
}
```

### **3. Warehouse Operations**
```typescript
export class WarehouseOperations {
    displayPickingQuantities(pickList: any[]) {
        return pickList.map(item => ({
            ...item,
            pickingBreakdown: this.optimizePickingUnits(item.quantity, item.availableUnits)
        }));
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
