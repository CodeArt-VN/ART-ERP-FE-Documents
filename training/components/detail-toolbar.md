# DetailToolbar Component Documentation

## 📋 **Tổng quan**

`DetailToolbarComponent` là toolbar cho các detail/form pages, cung cấp các action buttons cơ bản như Back, Refresh, Delete và Help cho single item operations.

**Selector**: `app-detail-toolbar`  
**Location**: `src/app/components/detail-toolbar/detail-toolbar.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() page: any;                      // Page reference
@Input() title: string;                  // Toolbar title
@Input() BackHref: string;               // Back navigation URL
@Input() ShowFeature: boolean = false;   // Show feature toggle
@Input() ShowDelete: boolean = true;     // Show delete button
@Input() ShowHelp: boolean = true;       // Show help button
@Input() ShowRefresh: boolean = true;    // Show refresh button
@Input() NoBorder: boolean = false;      // Remove border styling
```

### **Output Events**
```typescript
@Output() refresh = new EventEmitter();  // Refresh action
@Output() delete = new EventEmitter();   // Delete action
@Output() help = new EventEmitter();     // Help action
```

---

## 🚀 **Basic Usage**

### **Simple Detail Toolbar**
```typescript
// Component
export class UserDetailPage extends PageBase {
    item: any = {};
    pageConfig = {
        pageName: 'user-detail',
        pageTitle: 'USER_DETAIL',
        canDelete: true
    };

    async refresh() {
        if (this.id) {
            await this.loadData();
        }
    }

    async deleteItem() {
        if (!this.item.Id) return;
        
        const confirmed = await this.env.actionConfirm(
            'delete', 
            1, 
            'user', 
            'DELETE_USER'
        );
        
        if (confirmed) {
            try {
                await this.userService.delete([this.item]);
                this.env.showMessage('DELETE_SUCCESS', 'success');
                this.nav('/user/user', 'back');
            } catch (error) {
                this.env.showErrorMessage(error);
            }
        }
    }

    showHelp() {
        this.env.showAlert('HELP_CONTENT', 'USER_HELP', 'HELP');
    }
}
```

```html
<!-- Template -->
<app-detail-toolbar
    [page]="this"
    [title]="pageConfig.pageTitle"
    [BackHref]="'/user/user'"
    [ShowDelete]="pageConfig.canDelete && item.Id"
    [ShowRefresh]="true"
    [ShowHelp]="true"
    (refresh)="refresh()"
    (delete)="deleteItem()"
    (help)="showHelp()">
</app-detail-toolbar>

<ion-content>
    <!-- Form content -->
    <form [formGroup]="formGroup">
        <!-- Form fields -->
    </form>
</ion-content>
```

---

## 📝 **Form Integration**

### **With Form Validation**
```typescript
export class ProductDetailPage extends PageBase {
    formGroup: FormGroup;
    item: any = {};
    isEditing = false;

    constructor(
        private formBuilder: FormBuilder,
        private productService: ProductService
    ) {
        super();
        this.buildForm();
    }

    buildForm() {
        this.formGroup = this.formBuilder.group({
            name: ['', Validators.required],
            code: ['', Validators.required],
            price: [0, [Validators.required, Validators.min(0)]],
            description: ['']
        });
    }

    async loadData() {
        if (this.id) {
            try {
                this.item = await this.productService.getAnItem(this.id);
                this.formGroup.patchValue(this.item);
                this.isEditing = true;
            } catch (error) {
                this.env.showErrorMessage(error);
            }
        }
    }

    async save() {
        if (this.formGroup.invalid) {
            this.markFormGroupTouched();
            return;
        }

        try {
            const formValue = this.formGroup.getRawValue();
            const savedItem = await this.productService.save({
                ...this.item,
                ...formValue
            });
            
            this.item = savedItem;
            this.env.showMessage('SAVE_SUCCESS', 'success');
            
            if (!this.isEditing) {
                this.nav(`/product/product-detail/${savedItem.Id}`, 'forward');
            }
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async refresh() {
        if (this.isEditing) {
            await this.loadData();
        } else {
            this.formGroup.reset();
            this.item = {};
        }
    }
}
```

```html
<app-detail-toolbar
    [page]="this"
    [title]="isEditing ? 'EDIT_PRODUCT' : 'NEW_PRODUCT'"
    [BackHref]="'/product/product'"
    [ShowDelete]="isEditing && item.Id"
    (refresh)="refresh()"
    (delete)="deleteItem()">
    
    <!-- Custom save button -->
    <ion-button 
        slot="end"
        [disabled]="formGroup.invalid"
        (click)="save()">
        <ion-icon name="save"></ion-icon>
        Save
    </ion-button>
</app-detail-toolbar>
```

---

## 🔄 **Workflow Integration**

### **Approval Workflow**
```typescript
export class OrderDetailPage extends PageBase {
    item: any = {};
    
    get canSubmitForApproval(): boolean {
        return this.item.Status === 'New' || this.item.Status === 'Unapproved';
    }
    
    get canApprove(): boolean {
        return this.item.Status === 'Submitted';
    }
    
    get canCancel(): boolean {
        return !['Canceled', 'Delivered'].includes(this.item.Status);
    }

    async submitForApproval() {
        try {
            await this.orderService.submitForApproval([this.item]);
            this.env.showMessage('SUBMIT_SUCCESS', 'success');
            await this.refresh();
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async approve() {
        const confirmed = await this.env.actionConfirm(
            'approve', 
            1, 
            'order'
        );
        
        if (confirmed) {
            try {
                await this.orderService.approve([this.item]);
                this.env.showMessage('APPROVE_SUCCESS', 'success');
                await this.refresh();
            } catch (error) {
                this.env.showErrorMessage(error);
            }
        }
    }

    async cancel() {
        const reason = await this.env.showPrompt(
            'CANCEL_REASON',
            'Enter cancellation reason'
        );
        
        if (reason) {
            try {
                await this.orderService.cancel([this.item], reason);
                this.env.showMessage('CANCEL_SUCCESS', 'success');
                await this.refresh();
            } catch (error) {
                this.env.showErrorMessage(error);
            }
        }
    }
}
```

```html
<app-detail-toolbar
    [page]="this"
    [title]="'ORDER_DETAIL'"
    [BackHref]="'/order/sale-order'"
    [ShowDelete]="item.Status === 'New'"
    (refresh)="refresh()"
    (delete)="deleteItem()">
    
    <!-- Workflow buttons -->
    <ion-button 
        *ngIf="canSubmitForApproval"
        fill="outline"
        (click)="submitForApproval()">
        <ion-icon name="send"></ion-icon>
        Submit
    </ion-button>
    
    <ion-button 
        *ngIf="canApprove"
        color="success"
        (click)="approve()">
        <ion-icon name="checkmark"></ion-icon>
        Approve
    </ion-button>
    
    <ion-button 
        *ngIf="canCancel"
        color="warning"
        (click)="cancel()">
        <ion-icon name="close"></ion-icon>
        Cancel
    </ion-button>
</app-detail-toolbar>
```

---

## 📱 **Modal Integration**

### **Modal Detail Toolbar**
```typescript
export class ProductModalPage extends PageBase {
    @Input() item: any = {};
    @Input() isModal = true;

    constructor(
        private modalController: ModalController,
        private productService: ProductService
    ) {
        super();
    }

    async save() {
        if (this.formGroup.invalid) return;

        try {
            const savedItem = await this.productService.save(this.formGroup.value);
            this.modalController.dismiss(savedItem, 'save');
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    close() {
        this.modalController.dismiss(null, 'cancel');
    }

    async deleteItem() {
        const confirmed = await this.env.actionConfirm('delete', 1, 'product');
        
        if (confirmed) {
            try {
                await this.productService.delete([this.item]);
                this.modalController.dismiss(this.item, 'delete');
            } catch (error) {
                this.env.showErrorMessage(error);
            }
        }
    }
}
```

```html
<ion-header>
    <app-detail-toolbar
        [page]="this"
        [title]="item.Id ? 'EDIT_PRODUCT' : 'NEW_PRODUCT'"
        [ShowDelete]="item.Id"
        [ShowRefresh]="false"
        [NoBorder]="true"
        (delete)="deleteItem()">
        
        <!-- Modal specific buttons -->
        <ion-button 
            slot="start"
            fill="clear"
            (click)="close()">
            <ion-icon name="close"></ion-icon>
        </ion-button>
        
        <ion-button 
            slot="end"
            [disabled]="formGroup.invalid"
            (click)="save()">
            <ion-icon name="save"></ion-icon>
            Save
        </ion-button>
    </app-detail-toolbar>
</ion-header>

<ion-content>
    <!-- Modal content -->
</ion-content>
```

---

## 🎨 **Customization**

### **Custom Actions**
```html
<app-detail-toolbar
    [page]="this"
    [title]="pageConfig.pageTitle"
    [BackHref]="'/invoice/arinvoice'"
    [ShowDelete]="false"
    (refresh)="refresh()">
    
    <!-- Custom action buttons -->
    <ion-button 
        slot="end"
        fill="outline"
        (click)="duplicate()">
        <ion-icon name="copy"></ion-icon>
        Duplicate
    </ion-button>
    
    <ion-button 
        slot="end"
        fill="outline"
        (click)="print()">
        <ion-icon name="print"></ion-icon>
        Print
    </ion-button>
    
    <ion-button 
        slot="end"
        fill="outline"
        (click)="export()">
        <ion-icon name="download"></ion-icon>
        Export
    </ion-button>
    
    <ion-button 
        slot="end"
        color="primary"
        [disabled]="formGroup.invalid"
        (click)="save()">
        <ion-icon name="save"></ion-icon>
        Save
    </ion-button>
</app-detail-toolbar>
```

### **Conditional Styling**
```scss
// Component SCSS
app-detail-toolbar {
    .toolbar-container {
        background: var(--ion-color-light);
        border-bottom: 1px solid var(--ion-color-medium);
        padding: 8px 16px;
        
        &.no-border {
            border-bottom: none;
        }
        
        .toolbar-title {
            font-size: 1.125rem;
            font-weight: 600;
            color: var(--ion-color-dark);
        }
        
        .action-buttons {
            display: flex;
            gap: 8px;
            align-items: center;
            
            ion-button {
                --border-radius: 6px;
                height: 32px;
                
                &.save-button {
                    --background: var(--ion-color-primary);
                    --color: var(--ion-color-primary-contrast);
                }
                
                &.delete-button {
                    --background: var(--ion-color-danger);
                    --color: var(--ion-color-danger-contrast);
                }
            }
        }
    }
}
```

---

## 📱 **Mobile Optimization**

### **Responsive Layout**
```scss
// Mobile styles
@media (max-width: 768px) {
    app-detail-toolbar {
        .toolbar-container {
            padding: 8px 12px;
            
            .toolbar-title {
                font-size: 1rem;
            }
            
            .action-buttons {
                gap: 4px;
                
                ion-button {
                    height: 36px;
                    min-width: 36px;
                    
                    // Hide text on mobile, show only icons
                    .button-inner {
                        flex-direction: column;
                        
                        ion-icon {
                            margin: 0;
                        }
                        
                        span {
                            display: none;
                        }
                    }
                }
            }
        }
    }
}
```

### **Touch-Friendly Actions**
```html
<app-detail-toolbar
    [page]="this"
    [title]="pageConfig.pageTitle"
    class="mobile-optimized">
    
    <!-- Mobile action sheet -->
    <ion-button 
        *ngIf="isMobile"
        fill="clear"
        (click)="presentActionSheet()">
        <ion-icon name="ellipsis-vertical"></ion-icon>
    </ion-button>
</app-detail-toolbar>
```

```typescript
async presentActionSheet() {
    const actionSheet = await this.actionSheetController.create({
        header: 'Actions',
        buttons: [
            {
                text: 'Save',
                icon: 'save',
                handler: () => this.save()
            },
            {
                text: 'Duplicate',
                icon: 'copy',
                handler: () => this.duplicate()
            },
            {
                text: 'Delete',
                icon: 'trash',
                role: 'destructive',
                handler: () => this.deleteItem()
            },
            {
                text: 'Cancel',
                icon: 'close',
                role: 'cancel'
            }
        ]
    });
    
    await actionSheet.present();
}
```

---

## 🔧 **Best Practices**

### **1. Permission Checking**
```typescript
// ✅ Đúng - Check permissions
async ngOnInit() {
    this.canEdit = await this.env.checkFormPermission('/product/edit');
    this.canDelete = await this.env.checkFormPermission('/product/delete');
    
    this.pageConfig.canDelete = this.canDelete && this.item.Id;
}

// ❌ Sai - No permission check
ngOnInit() {
    this.pageConfig.canDelete = true;
}
```

### **2. Form State Management**
```typescript
// ✅ Đúng - Proper form state handling
get hasUnsavedChanges(): boolean {
    return this.formGroup.dirty && this.formGroup.valid;
}

async canDeactivate(): Promise<boolean> {
    if (this.hasUnsavedChanges) {
        return await this.env.showPrompt(
            'UNSAVED_CHANGES',
            'You have unsaved changes. Do you want to leave?'
        );
    }
    return true;
}
```

### **3. Error Handling**
```typescript
// ✅ Đúng - Comprehensive error handling
async deleteItem() {
    if (!this.item.Id) {
        this.env.showMessage('NO_ITEM_TO_DELETE', 'warning');
        return;
    }

    try {
        const confirmed = await this.env.actionConfirm('delete', 1, 'item');
        
        if (confirmed) {
            await this.service.delete([this.item]);
            this.env.showMessage('DELETE_SUCCESS', 'success');
            this.nav(this.BackHref, 'back');
        }
    } catch (error) {
        this.env.showErrorMessage(error);
    }
}
```

---

## 🚨 **Common Patterns**

### **1. Auto-Save**
```typescript
export class AutoSaveDetailPage extends PageBase {
    autoSaveTimer: any;
    
    ngOnInit() {
        // Auto-save every 30 seconds
        this.formGroup.valueChanges.pipe(
            debounceTime(30000),
            distinctUntilChanged()
        ).subscribe(() => {
            if (this.formGroup.valid && this.formGroup.dirty) {
                this.autoSave();
            }
        });
    }

    async autoSave() {
        try {
            await this.service.save(this.formGroup.value);
            console.log('Auto-saved');
        } catch (error) {
            console.error('Auto-save failed:', error);
        }
    }
}
```

### **2. Draft Management**
```typescript
export class DraftDetailPage extends PageBase {
    isDraft = false;
    
    async saveDraft() {
        try {
            this.item.IsDraft = true;
            await this.service.save(this.item);
            this.env.showMessage('DRAFT_SAVED', 'success');
            this.isDraft = true;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async publish() {
        try {
            this.item.IsDraft = false;
            await this.service.save(this.item);
            this.env.showMessage('PUBLISHED_SUCCESS', 'success');
            this.isDraft = false;
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
