# PromotionService Documentation

## 📋 **Tổng quan**

`PromotionService` là service để handle promotion programs (chương trình khuyến mãi) trong hệ thống ERP. Service này manage việc load và cache danh sách promotions đang active để sử dụng trong các module bán hàng và marketing.

**Location**: `src/app/services/custom/promotion.service.ts`  
**Injectable**: `providedIn: 'root'`

---

## 🔧 **Key Features**

### **Promotion Management**
- **Load Active Promotions**: Lấy danh sách promotions đang hoạt động
- **Date-based Filtering**: Filter promotions theo ngày hiện tại
- **Status Filtering**: Chỉ lấy promotions đã được approved
- **In-memory Caching**: Cache promotions list để improve performance

### **Integration Points**
- **Sales Module**: Áp dụng promotions cho đơn hàng
- **Product Catalog**: Hiển thị promotions cho sản phẩm
- **Marketing Campaigns**: Quản lý campaigns và offers

---

## 🚀 **Core Methods**

### **Promotion Loading**
```typescript
getPromotions(): void
```

### **Properties**
```typescript
promotionList: any[] // Cached promotion list
```

---

## 🎨 **Usage Examples**

### **1. Load and Display Promotions**
```typescript
export class PromotionListComponent implements OnInit {
    promotions: any[] = [];
    loading: boolean = false;
    
    constructor(private promotionService: PromotionService) {}
    
    async ngOnInit() {
        await this.loadPromotions();
    }
    
    async loadPromotions() {
        this.loading = true;
        
        try {
            // Load promotions from service
            this.promotionService.getPromotions();
            
            // Wait for promotions to be loaded
            setTimeout(() => {
                this.promotions = this.promotionService.promotionList;
                this.loading = false;
            }, 1000);
            
        } catch (error) {
            console.error('Failed to load promotions:', error);
            this.env.showErrorMessage('Failed to load promotions');
            this.loading = false;
        }
    }
    
    getActivePromotions(): any[] {
        const now = new Date();
        
        return this.promotions.filter(promotion => {
            const startDate = new Date(promotion.StartDate);
            const endDate = new Date(promotion.EndDate);
            
            return now >= startDate && now <= endDate && promotion.Status === 'Approved';
        });
    }
    
    getPromotionsByProduct(productId: number): any[] {
        return this.promotions.filter(promotion => 
            promotion.ApplicableProducts?.includes(productId) ||
            promotion.ApplicableCategories?.includes(this.getProductCategory(productId))
        );
    }
}
```

### **2. Apply Promotions in Sales Order**
```typescript
export class SalesOrderService {
    constructor(private promotionService: PromotionService) {}
    
    async calculateOrderTotal(orderItems: any[]): Promise<any> {
        // Ensure promotions are loaded
        if (this.promotionService.promotionList.length === 0) {
            this.promotionService.getPromotions();
            await this.waitForPromotions();
        }
        
        let subtotal = 0;
        let totalDiscount = 0;
        const appliedPromotions = [];
        
        // Calculate subtotal
        orderItems.forEach(item => {
            subtotal += item.Quantity * item.UnitPrice;
        });
        
        // Apply promotions
        const applicablePromotions = this.getApplicablePromotions(orderItems, subtotal);
        
        for (const promotion of applicablePromotions) {
            const discount = this.calculatePromotionDiscount(promotion, orderItems, subtotal);
            
            if (discount > 0) {
                totalDiscount += discount;
                appliedPromotions.push({
                    promotionId: promotion.Id,
                    promotionName: promotion.Name,
                    discountAmount: discount,
                    discountType: promotion.DiscountType
                });
            }
        }
        
        return {
            subtotal: subtotal,
            totalDiscount: totalDiscount,
            total: subtotal - totalDiscount,
            appliedPromotions: appliedPromotions
        };
    }
    
    private getApplicablePromotions(orderItems: any[], subtotal: number): any[] {
        return this.promotionService.promotionList.filter(promotion => {
            // Check if promotion is currently active
            if (!this.isPromotionActive(promotion)) {
                return false;
            }
            
            // Check minimum order amount
            if (promotion.MinOrderAmount && subtotal < promotion.MinOrderAmount) {
                return false;
            }
            
            // Check applicable products/categories
            if (promotion.ApplicableProducts || promotion.ApplicableCategories) {
                return this.hasApplicableItems(promotion, orderItems);
            }
            
            return true;
        });
    }
    
    private calculatePromotionDiscount(promotion: any, orderItems: any[], subtotal: number): number {
        switch (promotion.DiscountType) {
            case 'Percentage':
                return subtotal * (promotion.DiscountValue / 100);
                
            case 'FixedAmount':
                return Math.min(promotion.DiscountValue, subtotal);
                
            case 'BuyXGetY':
                return this.calculateBuyXGetYDiscount(promotion, orderItems);
                
            case 'FreeShipping':
                return this.calculateShippingDiscount(orderItems);
                
            default:
                return 0;
        }
    }
    
    private calculateBuyXGetYDiscount(promotion: any, orderItems: any[]): number {
        let discount = 0;
        
        orderItems.forEach(item => {
            if (this.isItemApplicableForPromotion(item, promotion)) {
                const sets = Math.floor(item.Quantity / promotion.BuyQuantity);
                const freeItems = sets * promotion.GetQuantity;
                discount += freeItems * item.UnitPrice;
            }
        });
        
        return discount;
    }
    
    private async waitForPromotions(): Promise<void> {
        return new Promise((resolve) => {
            const checkPromotions = () => {
                if (this.promotionService.promotionList.length > 0) {
                    resolve();
                } else {
                    setTimeout(checkPromotions, 100);
                }
            };
            checkPromotions();
        });
    }
}
```

### **3. Promotion Display Component**
```typescript
export class PromotionCardComponent {
    @Input() promotion: any;
    
    constructor(private promotionService: PromotionService) {}
    
    get isActive(): boolean {
        const now = new Date();
        const startDate = new Date(this.promotion.StartDate);
        const endDate = new Date(this.promotion.EndDate);
        
        return now >= startDate && now <= endDate && this.promotion.Status === 'Approved';
    }
    
    get timeRemaining(): string {
        if (!this.isActive) {
            return 'Expired';
        }
        
        const now = new Date();
        const endDate = new Date(this.promotion.EndDate);
        const diffMs = endDate.getTime() - now.getTime();
        
        const days = Math.floor(diffMs / (1000 * 60 * 60 * 24));
        const hours = Math.floor((diffMs % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
        
        if (days > 0) {
            return `${days} days remaining`;
        } else if (hours > 0) {
            return `${hours} hours remaining`;
        } else {
            return 'Ending soon';
        }
    }
    
    get discountText(): string {
        switch (this.promotion.DiscountType) {
            case 'Percentage':
                return `${this.promotion.DiscountValue}% OFF`;
            case 'FixedAmount':
                return `$${this.promotion.DiscountValue} OFF`;
            case 'BuyXGetY':
                return `Buy ${this.promotion.BuyQuantity} Get ${this.promotion.GetQuantity} Free`;
            case 'FreeShipping':
                return 'FREE SHIPPING';
            default:
                return 'Special Offer';
        }
    }
    
    applyPromotion() {
        // Add promotion to cart or current order
        this.env.publishEvent({
            Code: 'promotion:apply',
            Value: {
                promotionId: this.promotion.Id,
                promotionCode: this.promotion.Code
            }
        });
        
        this.env.showMessage(`Promotion "${this.promotion.Name}" applied!`, 'success');
    }
}
```

### **4. Enhanced Promotion Service**
```typescript
export class EnhancedPromotionService extends PromotionService {
    private promotionCache = new Map<string, any>();
    private cacheExpiry = 5 * 60 * 1000; // 5 minutes
    
    constructor(
        public commonService: CommonService,
        public programProvider: PR_ProgramProvider,
        private env: EnvService
    ) {
        super(commonService, programProvider);
    }
    
    async getPromotionsAsync(): Promise<any[]> {
        const cacheKey = 'active_promotions';
        const cached = this.promotionCache.get(cacheKey);
        
        if (cached && (Date.now() - cached.timestamp) < this.cacheExpiry) {
            return cached.data;
        }
        
        try {
            const result = await this.programProvider.read({
                BetweenDate: new Date(),
                Status: 'Approved'
            });
            
            const promotions = result.data || [];
            
            // Cache the result
            this.promotionCache.set(cacheKey, {
                data: promotions,
                timestamp: Date.now()
            });
            
            // Update the promotionList property
            this.promotionList = promotions;
            
            return promotions;
            
        } catch (error) {
            console.error('Failed to load promotions:', error);
            
            // Return cached data if available, even if expired
            if (cached) {
                return cached.data;
            }
            
            throw error;
        }
    }
    
    getPromotionById(promotionId: number): any {
        return this.promotionList.find(p => p.Id === promotionId);
    }
    
    getPromotionByCode(promotionCode: string): any {
        return this.promotionList.find(p => p.Code === promotionCode);
    }
    
    validatePromotionCode(code: string): Promise<any> {
        return new Promise((resolve, reject) => {
            const promotion = this.getPromotionByCode(code);
            
            if (!promotion) {
                reject({ message: 'Invalid promotion code' });
                return;
            }
            
            if (!this.isPromotionActive(promotion)) {
                reject({ message: 'Promotion has expired' });
                return;
            }
            
            if (promotion.UsageLimit && promotion.UsedCount >= promotion.UsageLimit) {
                reject({ message: 'Promotion usage limit exceeded' });
                return;
            }
            
            resolve(promotion);
        });
    }
    
    private isPromotionActive(promotion: any): boolean {
        const now = new Date();
        const startDate = new Date(promotion.StartDate);
        const endDate = new Date(promotion.EndDate);
        
        return now >= startDate && 
               now <= endDate && 
               promotion.Status === 'Approved' &&
               !promotion.IsDeleted;
    }
    
    async refreshPromotions(): Promise<void> {
        this.promotionCache.clear();
        await this.getPromotionsAsync();
        
        // Notify components about promotion updates
        this.env.publishEvent({
            Code: 'promotions:updated',
            Value: { promotions: this.promotionList }
        });
    }
    
    getPromotionsByCategory(categoryId: number): any[] {
        return this.promotionList.filter(promotion => 
            promotion.ApplicableCategories?.includes(categoryId)
        );
    }
    
    getPromotionsByDateRange(startDate: Date, endDate: Date): any[] {
        return this.promotionList.filter(promotion => {
            const promoStart = new Date(promotion.StartDate);
            const promoEnd = new Date(promotion.EndDate);
            
            return (promoStart <= endDate && promoEnd >= startDate);
        });
    }
}
```

---

## 📋 **Best Practices**

### **1. Caching Strategy**
```typescript
// ✅ Implement proper caching
async loadPromotionsWithCache() {
    const cacheKey = 'promotions_cache';
    const cached = await this.env.getStorage(cacheKey);
    
    if (cached && this.isCacheValid(cached)) {
        this.promotionList = cached.data;
        return;
    }
    
    // Load fresh data
    this.getPromotions();
    
    // Cache the result
    setTimeout(() => {
        this.env.setStorage(cacheKey, {
            data: this.promotionList,
            timestamp: Date.now()
        });
    }, 1000);
}
```

### **2. Error Handling**
```typescript
// ✅ Graceful error handling
async loadPromotionsSafely() {
    try {
        this.getPromotions();
    } catch (error) {
        console.error('Promotion loading failed:', error);
        
        // Use fallback or cached data
        const fallbackPromotions = await this.getFallbackPromotions();
        this.promotionList = fallbackPromotions;
    }
}
```

### **3. Performance Optimization**
```typescript
// ✅ Optimize promotion calculations
calculatePromotionsOptimized(orderItems: any[]): any {
    // Pre-filter applicable promotions
    const applicablePromotions = this.promotionList.filter(p => 
        this.isPromotionActive(p) && this.quickEligibilityCheck(p, orderItems)
    );
    
    // Calculate discounts only for applicable promotions
    return applicablePromotions.map(promotion => 
        this.calculatePromotionDiscount(promotion, orderItems)
    );
}
```

---

## 🚨 **Security Considerations**

### **1. Promotion Validation**
```typescript
// ✅ Validate promotion eligibility
validatePromotionEligibility(promotion: any, user: any, order: any): boolean {
    // Check user eligibility
    if (promotion.EligibleUserTypes && 
        !promotion.EligibleUserTypes.includes(user.Type)) {
        return false;
    }
    
    // Check usage limits
    if (promotion.UsageLimit && 
        promotion.UsedCount >= promotion.UsageLimit) {
        return false;
    }
    
    // Check per-user limits
    if (promotion.PerUserLimit && 
        this.getUserPromotionUsage(user.Id, promotion.Id) >= promotion.PerUserLimit) {
        return false;
    }
    
    return true;
}
```

### **2. Discount Validation**
```typescript
// ✅ Validate calculated discounts
validateDiscountAmount(discount: number, orderTotal: number): number {
    // Ensure discount doesn't exceed order total
    const maxDiscount = orderTotal * 0.99; // Max 99% discount
    
    return Math.min(discount, maxDiscount);
}
```

---

## 🎯 **Integration Examples**

### **With Shopping Cart**
```typescript
export class ShoppingCartService {
    constructor(private promotionService: PromotionService) {}
    
    async updateCartWithPromotions() {
        const cartItems = this.getCartItems();
        const promotions = this.promotionService.promotionList;
        
        // Apply automatic promotions
        const autoPromotions = promotions.filter(p => p.IsAutoApply);
        
        for (const promotion of autoPromotions) {
            if (this.isPromotionApplicable(promotion, cartItems)) {
                this.applyPromotionToCart(promotion);
            }
        }
    }
}
```

### **With Product Display**
```typescript
export class ProductComponent {
    @Input() product: any;
    availablePromotions: any[] = [];
    
    constructor(private promotionService: PromotionService) {}
    
    ngOnInit() {
        this.loadProductPromotions();
    }
    
    loadProductPromotions() {
        this.availablePromotions = this.promotionService.promotionList.filter(promotion =>
            promotion.ApplicableProducts?.includes(this.product.Id) ||
            promotion.ApplicableCategories?.includes(this.product.CategoryId)
        );
    }
}
```

PromotionService cung cấp essential functionality cho promotion management với caching, validation, và integration support cho sales và marketing modules trong ERP system.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
