# BusinessLogicService Documentation

## 📋 **Tổng quan**

`BusinessLogicService` là một placeholder service class được thiết kế để handle business logic operations trong các pages của ứng dụng ERP. Hiện tại service này chưa có implementation cụ thể nhưng được chuẩn bị để implement các business rules và logic processing.

**Location**: `src/app/services/page/business-logic.service.ts`  
**Type**: Class (not Injectable - intended for extension)

---

## 🔧 **Current Status**

### **Implementation Status**
- ✅ **Basic Structure**: Class structure đã được tạo
- ❌ **Methods**: Chưa có methods được implement
- ❌ **Properties**: Chưa có properties được define
- ❌ **Dependencies**: Chưa có dependencies được inject

### **Intended Purpose**
- **Business Rules Processing**: Xử lý các business rules phức tạp
- **Data Validation**: Validate business logic cho forms và operations
- **Workflow Management**: Quản lý workflows và approval processes
- **Calculation Engine**: Thực hiện các calculations phức tạp

---

## 🚀 **Proposed Implementation**

### **1. Enhanced Business Logic Service**
```typescript
import { Injectable } from '@angular/core';
import { EnvService } from '../core/env.service';
import { CommonService } from '../core/common.service';

@Injectable({
    providedIn: 'root'
})
export class BusinessLogicService {
    constructor(
        private env: EnvService,
        private commonService: CommonService
    ) {}
    
    // Business rule validation
    async validateBusinessRule(ruleType: string, data: any): Promise<ValidationResult> {
        const validators = {
            'sales_order': (data) => this.validateSalesOrder(data),
            'inventory_transaction': (data) => this.validateInventoryTransaction(data),
            'financial_entry': (data) => this.validateFinancialEntry(data),
            'approval_workflow': (data) => this.validateApprovalWorkflow(data)
        };
        
        const validator = validators[ruleType];
        
        if (!validator) {
            throw new Error(`Unknown business rule type: ${ruleType}`);
        }
        
        return validator(data);
    }
    
    // Calculate business metrics
    calculateBusinessMetrics(type: string, data: any): any {
        switch (type) {
            case 'order_total':
                return this.calculateOrderTotal(data);
            case 'discount_amount':
                return this.calculateDiscountAmount(data);
            case 'tax_amount':
                return this.calculateTaxAmount(data);
            case 'profit_margin':
                return this.calculateProfitMargin(data);
            default:
                throw new Error(`Unknown calculation type: ${type}`);
        }
    }
    
    // Process workflow steps
    async processWorkflowStep(workflowType: string, currentStep: string, data: any): Promise<WorkflowResult> {
        const workflows = {
            'purchase_approval': (step, data) => this.processPurchaseApproval(step, data),
            'sales_approval': (step, data) => this.processSalesApproval(step, data),
            'inventory_approval': (step, data) => this.processInventoryApproval(step, data)
        };
        
        const processor = workflows[workflowType];
        
        if (!processor) {
            throw new Error(`Unknown workflow type: ${workflowType}`);
        }
        
        return processor(currentStep, data);
    }
    
    // Private validation methods
    private validateSalesOrder(data: any): ValidationResult {
        const errors = [];
        
        // Validate customer
        if (!data.customerId) {
            errors.push('Customer is required');
        }
        
        // Validate items
        if (!data.items || data.items.length === 0) {
            errors.push('At least one item is required');
        }
        
        // Validate quantities
        data.items?.forEach((item, index) => {
            if (!item.quantity || item.quantity <= 0) {
                errors.push(`Invalid quantity for item ${index + 1}`);
            }
        });
        
        // Validate total amount
        const calculatedTotal = this.calculateOrderTotal(data);
        if (Math.abs(data.total - calculatedTotal) > 0.01) {
            errors.push('Order total does not match calculated amount');
        }
        
        return {
            isValid: errors.length === 0,
            errors: errors,
            warnings: []
        };
    }
    
    private calculateOrderTotal(data: any): number {
        let subtotal = 0;
        
        data.items?.forEach(item => {
            subtotal += (item.quantity || 0) * (item.unitPrice || 0);
        });
        
        const discountAmount = this.calculateDiscountAmount(data);
        const taxAmount = this.calculateTaxAmount(data);
        
        return subtotal - discountAmount + taxAmount;
    }
}

// Interfaces
interface ValidationResult {
    isValid: boolean;
    errors: string[];
    warnings: string[];
}

interface WorkflowResult {
    success: boolean;
    nextStep?: string;
    message?: string;
    data?: any;
}
```

### **2. Usage Examples**

#### **In Sales Order Page**
```typescript
export class SalesOrderPage extends PageBase {
    constructor(
        private businessLogic: BusinessLogicService,
        // ... other dependencies
    ) {
        super();
    }
    
    async validateOrder() {
        try {
            const validation = await this.businessLogic.validateBusinessRule('sales_order', this.item);
            
            if (!validation.isValid) {
                this.env.showErrorMessage(`Validation failed: ${validation.errors.join(', ')}`);
                return false;
            }
            
            if (validation.warnings.length > 0) {
                this.env.showMessage(`Warnings: ${validation.warnings.join(', ')}`, 'warning');
            }
            
            return true;
            
        } catch (error) {
            console.error('Order validation failed:', error);
            this.env.showErrorMessage('Validation error occurred');
            return false;
        }
    }
    
    async calculateTotals() {
        try {
            const total = this.businessLogic.calculateBusinessMetrics('order_total', this.item);
            const discount = this.businessLogic.calculateBusinessMetrics('discount_amount', this.item);
            const tax = this.businessLogic.calculateBusinessMetrics('tax_amount', this.item);
            
            this.item.subtotal = total + discount - tax;
            this.item.discountAmount = discount;
            this.item.taxAmount = tax;
            this.item.total = total;
            
        } catch (error) {
            console.error('Calculation failed:', error);
            this.env.showErrorMessage('Failed to calculate totals');
        }
    }
}
```

#### **In Approval Workflow**
```typescript
export class ApprovalWorkflowComponent {
    constructor(private businessLogic: BusinessLogicService) {}
    
    async processApproval(action: 'approve' | 'reject', comments?: string) {
        try {
            const workflowData = {
                documentId: this.documentId,
                action: action,
                comments: comments,
                userId: this.env.user.Id
            };
            
            const result = await this.businessLogic.processWorkflowStep(
                this.workflowType,
                this.currentStep,
                workflowData
            );
            
            if (result.success) {
                this.env.showMessage('Workflow processed successfully', 'success');
                
                if (result.nextStep) {
                    this.navigateToNextStep(result.nextStep);
                } else {
                    this.completeWorkflow();
                }
            } else {
                this.env.showErrorMessage(result.message || 'Workflow processing failed');
            }
            
        } catch (error) {
            console.error('Workflow processing failed:', error);
            this.env.showErrorMessage('Failed to process workflow');
        }
    }
}
```

### **3. Advanced Business Logic Patterns**

#### **Rule Engine Implementation**
```typescript
export class BusinessRuleEngine {
    private rules: Map<string, BusinessRule[]> = new Map();
    
    constructor(private businessLogic: BusinessLogicService) {
        this.initializeRules();
    }
    
    addRule(entityType: string, rule: BusinessRule) {
        if (!this.rules.has(entityType)) {
            this.rules.set(entityType, []);
        }
        
        this.rules.get(entityType).push(rule);
    }
    
    async executeRules(entityType: string, data: any): Promise<RuleExecutionResult> {
        const entityRules = this.rules.get(entityType) || [];
        const results = [];
        
        for (const rule of entityRules) {
            try {
                const result = await rule.execute(data);
                results.push(result);
                
                // Stop on critical errors
                if (!result.success && result.isCritical) {
                    break;
                }
            } catch (error) {
                results.push({
                    success: false,
                    isCritical: true,
                    message: `Rule execution failed: ${error.message}`,
                    ruleId: rule.id
                });
                break;
            }
        }
        
        return {
            success: results.every(r => r.success),
            results: results,
            criticalErrors: results.filter(r => !r.success && r.isCritical)
        };
    }
}

interface BusinessRule {
    id: string;
    name: string;
    description: string;
    execute(data: any): Promise<RuleResult>;
}

interface RuleResult {
    success: boolean;
    isCritical: boolean;
    message: string;
    ruleId: string;
}
```

---

## 📋 **Development Roadmap**

### **Phase 1: Core Implementation**
1. **Basic Structure**: Implement core service with dependency injection
2. **Validation Framework**: Create validation methods for common entities
3. **Calculation Engine**: Implement business calculations
4. **Error Handling**: Add comprehensive error handling

### **Phase 2: Advanced Features**
1. **Rule Engine**: Implement configurable business rule engine
2. **Workflow Management**: Add workflow processing capabilities
3. **Audit Trail**: Add logging and audit trail functionality
4. **Performance Optimization**: Optimize for large datasets

### **Phase 3: Integration**
1. **Page Integration**: Integrate with existing pages
2. **API Integration**: Connect with backend business logic APIs
3. **Real-time Updates**: Add real-time business rule updates
4. **Testing**: Comprehensive unit and integration testing

---

## 🎯 **Best Practices for Implementation**

### **1. Separation of Concerns**
```typescript
// ✅ Separate validation, calculation, and workflow logic
class BusinessLogicService {
    // Validation methods
    validateEntity(type: string, data: any): ValidationResult { }
    
    // Calculation methods
    calculateMetrics(type: string, data: any): CalculationResult { }
    
    // Workflow methods
    processWorkflow(type: string, step: string, data: any): WorkflowResult { }
}
```

### **2. Configuration-Driven Rules**
```typescript
// ✅ Use configuration for business rules
const businessRules = {
    salesOrder: {
        validation: [
            { field: 'customerId', required: true },
            { field: 'items', minLength: 1 },
            { field: 'total', type: 'number', min: 0 }
        ],
        calculations: [
            { type: 'subtotal', formula: 'sum(items.quantity * items.unitPrice)' },
            { type: 'tax', formula: 'subtotal * taxRate' }
        ]
    }
};
```

### **3. Async Operations**
```typescript
// ✅ Handle async business logic properly
async validateWithExternalSystems(data: any): Promise<ValidationResult> {
    try {
        const [customerValid, inventoryValid] = await Promise.all([
            this.validateCustomer(data.customerId),
            this.validateInventoryAvailability(data.items)
        ]);
        
        return {
            isValid: customerValid && inventoryValid,
            errors: [],
            warnings: []
        };
    } catch (error) {
        return {
            isValid: false,
            errors: [error.message],
            warnings: []
        };
    }
}
```

---

## 🚨 **Security Considerations**

### **1. Input Validation**
```typescript
// ✅ Validate all inputs
private validateInput(data: any): boolean {
    if (!data || typeof data !== 'object') {
        throw new Error('Invalid input data');
    }
    
    // Sanitize string inputs
    this.sanitizeStringFields(data);
    
    return true;
}
```

### **2. Permission Checks**
```typescript
// ✅ Check permissions before processing business logic
async processBusinessLogic(type: string, data: any): Promise<any> {
    const hasPermission = await this.env.checkFormPermission(`/business-logic/${type}`);
    
    if (!hasPermission) {
        throw new Error('Insufficient permissions');
    }
    
    return this.executeBusinessLogic(type, data);
}
```

BusinessLogicService được thiết kế để trở thành central hub cho tất cả business logic operations trong ứng dụng ERP, với extensible architecture và comprehensive validation framework.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
