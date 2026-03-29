# EInvoiceService Documentation

## 📋 **Tổng quan**

`EInvoiceService` là service chuyên biệt để handle electronic invoice (hóa đơn điện tử) operations trong hệ thống ERP. Service này extend từ `exService` và provide comprehensive functionality cho việc tạo, quản lý và xử lý hóa đơn điện tử theo quy định của Việt Nam.

**Location**: `src/app/services/custom/einvoice.service.ts`  
**Injectable**: `providedIn: 'root'`  
**Extends**: `exService`

---

## 🔧 **Key Features**

### **AR Invoice Management**
- **Create from Sales Orders**: Tạo hóa đơn từ đơn hàng bán
- **Invoice Processing**: Xử lý và validate hóa đơn
- **Status Management**: Quản lý trạng thái hóa đơn
- **Compliance**: Tuân thủ quy định hóa đơn điện tử VN

### **Electronic Invoice Integration**
- **E-Invoice Generation**: Tạo hóa đơn điện tử
- **Digital Signature**: Ký số hóa đơn
- **Tax Authority Integration**: Kết nối với cơ quan thuế
- **Format Compliance**: Đảm bảo format theo quy định

---

## 🚀 **Core Methods**

### **AR Invoice Operations**
```typescript
CreateARInvoiceFromSOs(IDOrders: number[]): Promise<any>
ProcessInvoice(invoiceData: any): Promise<any>
ValidateInvoice(invoiceId: number): Promise<any>
```

### **E-Invoice Operations**
```typescript
GenerateEInvoice(invoiceId: number): Promise<any>
SignEInvoice(invoiceId: number, signature: string): Promise<any>
SubmitToTaxAuthority(invoiceId: number): Promise<any>
```

### **Status Management**
```typescript
UpdateInvoiceStatus(invoiceId: number, status: string): Promise<any>
GetInvoiceStatus(invoiceId: number): Promise<any>
CancelInvoice(invoiceId: number, reason: string): Promise<any>
```

---

## 🎨 **Usage Examples**

### **1. Create Invoice from Sales Orders**
```typescript
export class InvoiceCreationComponent {
    selectedOrders: any[] = [];
    
    constructor(private eInvoiceService: EInvoiceService) {}
    
    async createInvoiceFromOrders() {
        if (this.selectedOrders.length === 0) {
            this.env.showMessage('Please select orders to create invoice', 'warning');
            return;
        }
        
        try {
            const orderIds = this.selectedOrders.map(order => order.Id);
            
            // Show loading
            await this.env.showLoading('Creating invoice from orders...', 
                this.processInvoiceCreation(orderIds)
            );
            
        } catch (error) {
            console.error('Invoice creation failed:', error);
            this.env.showErrorMessage('Failed to create invoice');
        }
    }
    
    private async processInvoiceCreation(orderIds: number[]) {
        // Create AR Invoice from Sales Orders
        const result = await this.eInvoiceService.CreateARInvoiceFromSOs(orderIds);
        
        if (result.success) {
            this.env.showMessage('Invoice created successfully', 'success');
            
            // Navigate to invoice detail
            this.router.navigate(['/invoice', result.invoiceId]);
            
            // Optionally generate e-invoice immediately
            if (result.autoGenerateEInvoice) {
                await this.generateEInvoice(result.invoiceId);
            }
        } else {
            throw new Error(result.message || 'Invoice creation failed');
        }
        
        return result;
    }
    
    private async generateEInvoice(invoiceId: number) {
        try {
            const eInvoiceResult = await this.eInvoiceService.GenerateEInvoice(invoiceId);
            
            if (eInvoiceResult.success) {
                this.env.showMessage('E-Invoice generated successfully', 'success');
            }
        } catch (error) {
            console.error('E-Invoice generation failed:', error);
            // Don't show error to user as main invoice was created successfully
        }
    }
}
```

### **2. E-Invoice Processing Workflow**
```typescript
export class EInvoiceProcessingService {
    constructor(private eInvoiceService: EInvoiceService) {}
    
    async processEInvoiceWorkflow(invoiceId: number): Promise<any> {
        try {
            // Step 1: Validate invoice data
            const validationResult = await this.eInvoiceService.ValidateInvoice(invoiceId);
            
            if (!validationResult.isValid) {
                throw new Error(`Validation failed: ${validationResult.errors.join(', ')}`);
            }
            
            // Step 2: Generate e-invoice
            const generationResult = await this.eInvoiceService.GenerateEInvoice(invoiceId);
            
            if (!generationResult.success) {
                throw new Error('E-Invoice generation failed');
            }
            
            // Step 3: Sign e-invoice (if digital signature is required)
            if (generationResult.requiresSignature) {
                const signatureResult = await this.signEInvoice(invoiceId);
                
                if (!signatureResult.success) {
                    throw new Error('Digital signature failed');
                }
            }
            
            // Step 4: Submit to tax authority
            const submissionResult = await this.eInvoiceService.SubmitToTaxAuthority(invoiceId);
            
            if (submissionResult.success) {
                // Update invoice status
                await this.eInvoiceService.UpdateInvoiceStatus(invoiceId, 'SUBMITTED');
                
                return {
                    success: true,
                    invoiceId: invoiceId,
                    eInvoiceNumber: submissionResult.eInvoiceNumber,
                    submissionDate: submissionResult.submissionDate
                };
            } else {
                throw new Error('Tax authority submission failed');
            }
            
        } catch (error) {
            // Update status to error
            await this.eInvoiceService.UpdateInvoiceStatus(invoiceId, 'ERROR');
            
            console.error('E-Invoice processing failed:', error);
            throw error;
        }
    }
    
    private async signEInvoice(invoiceId: number): Promise<any> {
        // Get digital signature (implementation depends on signature provider)
        const signature = await this.getDigitalSignature(invoiceId);
        
        return this.eInvoiceService.SignEInvoice(invoiceId, signature);
    }
    
    private async getDigitalSignature(invoiceId: number): Promise<string> {
        // Implementation depends on digital signature provider
        // This could be HSM, smart card, or cloud-based signature
        return 'digital_signature_data';
    }
}
```

### **3. Invoice Status Management**
```typescript
export class InvoiceStatusComponent {
    invoice: any = {};
    statusHistory: any[] = [];
    
    constructor(private eInvoiceService: EInvoiceService) {}
    
    async ngOnInit() {
        const invoiceId = this.route.snapshot.params['id'];
        await this.loadInvoiceStatus(invoiceId);
    }
    
    async loadInvoiceStatus(invoiceId: number) {
        try {
            const statusResult = await this.eInvoiceService.GetInvoiceStatus(invoiceId);
            
            this.invoice = statusResult.invoice;
            this.statusHistory = statusResult.statusHistory;
            
            // Set up real-time status updates
            this.setupStatusTracking(invoiceId);
            
        } catch (error) {
            console.error('Failed to load invoice status:', error);
            this.env.showErrorMessage('Failed to load invoice status');
        }
    }
    
    async cancelInvoice() {
        const confirmed = await this.env.showPrompt(
            'Are you sure you want to cancel this invoice?',
            'Cancel Invoice',
            'Confirm Cancellation'
        );
        
        if (confirmed) {
            try {
                const reason = await this.getCancellationReason();
                
                await this.eInvoiceService.CancelInvoice(this.invoice.Id, reason);
                
                this.env.showMessage('Invoice cancelled successfully', 'success');
                await this.loadInvoiceStatus(this.invoice.Id);
                
            } catch (error) {
                console.error('Invoice cancellation failed:', error);
                this.env.showErrorMessage('Failed to cancel invoice');
            }
        }
    }
    
    private async getCancellationReason(): Promise<string> {
        const reasonPrompt = await this.env.showPrompt(
            'Please provide a reason for cancellation:',
            'Cancellation Reason',
            'Submit',
            'Cancel',
            [{
                name: 'reason',
                type: 'textarea',
                placeholder: 'Enter cancellation reason...'
            }]
        );
        
        return reasonPrompt?.reason || 'No reason provided';
    }
    
    private setupStatusTracking(invoiceId: number) {
        // Subscribe to real-time status updates
        this.env.getEvents().subscribe(event => {
            if (event.Code === 'invoice:status:updated' && event.Value.invoiceId === invoiceId) {
                this.loadInvoiceStatus(invoiceId);
            }
        });
    }
}
```

### **4. Batch Invoice Processing**
```typescript
export class BatchInvoiceService {
    constructor(private eInvoiceService: EInvoiceService) {}
    
    async processBatchInvoices(orderGroups: any[]): Promise<any> {
        const results = [];
        const errors = [];
        
        for (let i = 0; i < orderGroups.length; i++) {
            const group = orderGroups[i];
            
            try {
                // Create invoice for each group
                const invoiceResult = await this.eInvoiceService.CreateARInvoiceFromSOs(
                    group.orderIds
                );
                
                if (invoiceResult.success) {
                    // Process e-invoice workflow
                    const eInvoiceResult = await this.processEInvoiceWorkflow(
                        invoiceResult.invoiceId
                    );
                    
                    results.push({
                        groupId: group.id,
                        invoiceId: invoiceResult.invoiceId,
                        eInvoiceNumber: eInvoiceResult.eInvoiceNumber,
                        status: 'success'
                    });
                } else {
                    throw new Error(invoiceResult.message);
                }
                
                // Update progress
                this.env.publishEvent({
                    Code: 'batch:invoice:progress',
                    Value: {
                        processed: i + 1,
                        total: orderGroups.length,
                        percentage: ((i + 1) / orderGroups.length) * 100
                    }
                });
                
            } catch (error) {
                console.error(`Batch invoice ${i + 1} failed:`, error);
                
                errors.push({
                    groupId: group.id,
                    error: error.message,
                    orderIds: group.orderIds
                });
            }
        }
        
        return {
            success: results.length > 0,
            processed: results.length,
            failed: errors.length,
            results: results,
            errors: errors
        };
    }
    
    private async processEInvoiceWorkflow(invoiceId: number): Promise<any> {
        // Reuse the workflow from EInvoiceProcessingService
        const processingService = new EInvoiceProcessingService(this.eInvoiceService);
        return processingService.processEInvoiceWorkflow(invoiceId);
    }
}
```

---

## 🔧 **Advanced Features**

### **1. Invoice Template Management**
```typescript
export class InvoiceTemplateService {
    constructor(private eInvoiceService: EInvoiceService) {}
    
    async applyInvoiceTemplate(invoiceId: number, templateId: string): Promise<any> {
        try {
            const template = await this.getInvoiceTemplate(templateId);
            const invoice = await this.getInvoiceData(invoiceId);
            
            // Apply template formatting
            const formattedInvoice = this.formatInvoiceWithTemplate(invoice, template);
            
            // Update invoice with template
            return this.eInvoiceService.UpdateInvoiceFormat(invoiceId, formattedInvoice);
            
        } catch (error) {
            console.error('Template application failed:', error);
            throw error;
        }
    }
    
    private formatInvoiceWithTemplate(invoice: any, template: any): any {
        return {
            ...invoice,
            format: template.format,
            layout: template.layout,
            branding: template.branding,
            customFields: template.customFields
        };
    }
}
```

### **2. Tax Compliance Validation**
```typescript
export class TaxComplianceService {
    constructor(private eInvoiceService: EInvoiceService) {}
    
    async validateTaxCompliance(invoiceData: any): Promise<any> {
        const validationResults = [];
        
        // Validate VAT rates
        const vatValidation = this.validateVATRates(invoiceData);
        validationResults.push(vatValidation);
        
        // Validate customer information
        const customerValidation = this.validateCustomerInfo(invoiceData.customer);
        validationResults.push(customerValidation);
        
        // Validate invoice format
        const formatValidation = this.validateInvoiceFormat(invoiceData);
        validationResults.push(formatValidation);
        
        // Validate amounts and calculations
        const amountValidation = this.validateAmounts(invoiceData);
        validationResults.push(amountValidation);
        
        const hasErrors = validationResults.some(result => !result.isValid);
        
        return {
            isValid: !hasErrors,
            validations: validationResults,
            errors: validationResults
                .filter(result => !result.isValid)
                .map(result => result.error)
        };
    }
    
    private validateVATRates(invoiceData: any): any {
        const validVATRates = [0, 5, 8, 10]; // Valid VAT rates in Vietnam
        
        for (const item of invoiceData.items) {
            if (!validVATRates.includes(item.vatRate)) {
                return {
                    isValid: false,
                    error: `Invalid VAT rate: ${item.vatRate}% for item ${item.name}`
                };
            }
        }
        
        return { isValid: true };
    }
    
    private validateCustomerInfo(customer: any): any {
        const requiredFields = ['name', 'address', 'taxCode'];
        
        for (const field of requiredFields) {
            if (!customer[field]) {
                return {
                    isValid: false,
                    error: `Missing required customer field: ${field}`
                };
            }
        }
        
        // Validate tax code format (Vietnamese tax code format)
        const taxCodeRegex = /^\d{10}(-\d{3})?$/;
        if (!taxCodeRegex.test(customer.taxCode)) {
            return {
                isValid: false,
                error: 'Invalid tax code format'
            };
        }
        
        return { isValid: true };
    }
}
```

### **3. E-Invoice Archive Management**
```typescript
export class EInvoiceArchiveService {
    constructor(private eInvoiceService: EInvoiceService) {}
    
    async archiveInvoice(invoiceId: number): Promise<any> {
        try {
            // Get invoice data
            const invoice = await this.eInvoiceService.GetInvoiceData(invoiceId);
            
            // Create archive entry
            const archiveData = {
                invoiceId: invoiceId,
                eInvoiceNumber: invoice.eInvoiceNumber,
                archiveDate: new Date(),
                retentionPeriod: 10 * 365, // 10 years as per Vietnamese law
                archiveFormat: 'XML',
                digitalSignature: invoice.digitalSignature
            };
            
            // Store in archive system
            const archiveResult = await this.storeInArchive(archiveData);
            
            // Update invoice status
            await this.eInvoiceService.UpdateInvoiceStatus(invoiceId, 'ARCHIVED');
            
            return archiveResult;
            
        } catch (error) {
            console.error('Invoice archiving failed:', error);
            throw error;
        }
    }
    
    async retrieveFromArchive(eInvoiceNumber: string): Promise<any> {
        try {
            const archiveEntry = await this.findInArchive(eInvoiceNumber);
            
            if (!archiveEntry) {
                throw new Error('Invoice not found in archive');
            }
            
            // Verify digital signature
            const isValid = await this.verifyDigitalSignature(archiveEntry);
            
            if (!isValid) {
                throw new Error('Archive integrity check failed');
            }
            
            return archiveEntry;
            
        } catch (error) {
            console.error('Archive retrieval failed:', error);
            throw error;
        }
    }
}
```

---

## 📋 **Best Practices**

### **1. Error Handling**
```typescript
// ✅ Comprehensive error handling for e-invoice operations
async createInvoiceSafely(orderIds: number[]) {
    try {
        const result = await this.eInvoiceService.CreateARInvoiceFromSOs(orderIds);
        return result;
    } catch (error) {
        if (error.code === 'INVALID_ORDERS') {
            this.env.showErrorMessage('Some orders are invalid for invoice creation');
        } else if (error.code === 'DUPLICATE_INVOICE') {
            this.env.showErrorMessage('Invoice already exists for these orders');
        } else if (error.code === 'TAX_VALIDATION_FAILED') {
            this.env.showErrorMessage('Tax validation failed. Please check order details.');
        } else {
            this.env.showErrorMessage('Failed to create invoice');
        }
        throw error;
    }
}
```

### **2. Status Tracking**
```typescript
// ✅ Implement comprehensive status tracking
trackInvoiceStatus(invoiceId: number) {
    const statusSteps = [
        'CREATED',
        'VALIDATED', 
        'E_INVOICE_GENERATED',
        'SIGNED',
        'SUBMITTED',
        'APPROVED'
    ];
    
    // Update UI based on current status
    this.updateStatusIndicator(statusSteps);
}
```

### **3. Compliance Validation**
```typescript
// ✅ Always validate compliance before processing
async validateBeforeProcessing(invoiceData: any) {
    const complianceResult = await this.validateTaxCompliance(invoiceData);
    
    if (!complianceResult.isValid) {
        throw new Error(`Compliance validation failed: ${complianceResult.errors.join(', ')}`);
    }
    
    return true;
}
```

---

## 🚨 **Security & Compliance**

### **1. Digital Signature Security**
```typescript
// ✅ Secure digital signature handling
async secureSignInvoice(invoiceId: number) {
    // Use secure signature provider
    const signatureProvider = new SecureSignatureProvider();
    
    // Get invoice hash for signing
    const invoiceHash = await this.generateInvoiceHash(invoiceId);
    
    // Sign with secure key
    const signature = await signatureProvider.sign(invoiceHash);
    
    // Verify signature before storing
    const isValid = await signatureProvider.verify(invoiceHash, signature);
    
    if (!isValid) {
        throw new Error('Digital signature verification failed');
    }
    
    return this.eInvoiceService.SignEInvoice(invoiceId, signature);
}
```

### **2. Data Privacy**
```typescript
// ✅ Protect sensitive invoice data
sanitizeInvoiceData(invoiceData: any): any {
    return {
        ...invoiceData,
        // Remove or mask sensitive fields
        customer: {
            ...invoiceData.customer,
            personalId: this.maskPersonalId(invoiceData.customer.personalId)
        }
    };
}
```

EInvoiceService cung cấp comprehensive solution cho electronic invoice management với full compliance theo quy định Việt Nam, digital signature support, và enterprise-grade security features.

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
