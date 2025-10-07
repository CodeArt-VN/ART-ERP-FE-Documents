# FormControl Component Documentation

## 📋 **Tổng quan**

`FormControlComponent` là wrapper component cho form inputs, cung cấp consistent styling, validation display và label management cho tất cả form controls trong ứng dụng.

**Selector**: `app-form-control`  
**Location**: `src/app/components/controls/form-control.component.ts`

---

## 🔧 **Properties**

### **Input Properties**
```typescript
@Input() field: InputControlField;       // Complete field configuration
@Input() form: FormGroup;                // Form group reference
@Input() type: string = 'text';          // Input type
@Input() id: string;                     // Form control ID
@Input() label: string;                  // Display label
```

### **Output Events**
```typescript
@Output() change = new EventEmitter();           // Value change events
@Output() controlChange = new EventEmitter();    // Control-specific events
@Output() nav = new EventEmitter();              // Navigation events
```

### **InputControlField Interface**
```typescript
interface InputControlField {
    form: FormGroup;
    id: string;
    type: string;
    label?: string;
    placeholder?: string;
    dataSource?: any[];
    bindValue?: string;
    bindLabel?: string;
    multiple?: boolean;
    clearable?: boolean;
    noCheckDirty?: boolean;
    color?: string;
    appendTo?: string;
    branchConfig?: BranchConfig;
    treeConfig?: TreeConfig;
}
```

---

## 🚀 **Basic Usage**

### **Simple Text Input**
```typescript
// Component
export class UserFormPage extends PageBase {
    formGroup: FormGroup;

    constructor(private formBuilder: FormBuilder) {
        super();
        this.formGroup = this.formBuilder.group({
            name: ['', Validators.required],
            email: ['', [Validators.required, Validators.email]],
            phone: ['']
        });
    }
}
```

```html
<!-- Template -->
<form [formGroup]="formGroup">
    <app-form-control [field]="{
        form: formGroup,
        id: 'name',
        type: 'text',
        label: 'Full Name',
        placeholder: 'Enter full name'
    }"></app-form-control>

    <app-form-control [field]="{
        form: formGroup,
        id: 'email',
        type: 'email',
        label: 'Email Address',
        placeholder: 'Enter email'
    }"></app-form-control>

    <app-form-control [field]="{
        form: formGroup,
        id: 'phone',
        type: 'tel',
        label: 'Phone Number',
        placeholder: 'Enter phone'
    }"></app-form-control>
</form>
```

---

## 📝 **Input Types**

### **Text Inputs**
```html
<!-- Basic text -->
<app-form-control [field]="{
    form: formGroup,
    id: 'name',
    type: 'text',
    label: 'Name'
}"></app-form-control>

<!-- Email -->
<app-form-control [field]="{
    form: formGroup,
    id: 'email',
    type: 'email',
    label: 'Email'
}"></app-form-control>

<!-- Password -->
<app-form-control [field]="{
    form: formGroup,
    id: 'password',
    type: 'password',
    label: 'Password'
}"></app-form-control>

<!-- Number -->
<app-form-control [field]="{
    form: formGroup,
    id: 'age',
    type: 'number',
    label: 'Age'
}"></app-form-control>

<!-- Textarea -->
<app-form-control [field]="{
    form: formGroup,
    id: 'description',
    type: 'textarea',
    label: 'Description'
}"></app-form-control>
```

### **Selection Inputs**
```html
<!-- Select dropdown -->
<app-form-control [field]="{
    form: formGroup,
    id: 'category',
    type: 'select',
    label: 'Category',
    dataSource: categories,
    bindValue: 'id',
    bindLabel: 'name'
}"></app-form-control>

<!-- Multi-select -->
<app-form-control [field]="{
    form: formGroup,
    id: 'tags',
    type: 'select',
    label: 'Tags',
    dataSource: tags,
    bindValue: 'id',
    bindLabel: 'name',
    multiple: true
}"></app-form-control>

<!-- Radio buttons -->
<app-form-control [field]="{
    form: formGroup,
    id: 'gender',
    type: 'radio',
    label: 'Gender',
    dataSource: [
        { value: 'male', label: 'Male' },
        { value: 'female', label: 'Female' }
    ]
}"></app-form-control>

<!-- Checkbox -->
<app-form-control [field]="{
    form: formGroup,
    id: 'isActive',
    type: 'checkbox',
    label: 'Active Status'
}"></app-form-control>
```

### **Date & Time Inputs**
```html
<!-- Date picker -->
<app-form-control [field]="{
    form: formGroup,
    id: 'birthDate',
    type: 'date',
    label: 'Birth Date'
}"></app-form-control>

<!-- Time picker -->
<app-form-control [field]="{
    form: formGroup,
    id: 'startTime',
    type: 'time',
    label: 'Start Time'
}"></app-form-control>

<!-- DateTime picker -->
<app-form-control [field]="{
    form: formGroup,
    id: 'appointmentDate',
    type: 'datetime',
    label: 'Appointment Date'
}"></app-form-control>
```

---

## 🎨 **Advanced Features**

### **Custom Templates**
```html
<app-form-control [field]="fieldConfig">
    <ng-template appInputControlTemplate>
        <div class="custom-input-wrapper">
            <ion-icon name="search" slot="start"></ion-icon>
            <ion-input 
                [formControlName]="fieldConfig.id"
                [placeholder]="fieldConfig.placeholder">
            </ion-input>
            <ion-button fill="clear" slot="end">
                <ion-icon name="close"></ion-icon>
            </ion-button>
        </div>
    </ng-template>
</app-form-control>
```

### **Conditional Display**
```typescript
// Component
export class ConditionalFormPage {
    formGroup: FormGroup;
    showAdvanced = false;

    constructor(private formBuilder: FormBuilder) {
        this.formGroup = this.formBuilder.group({
            type: ['basic'],
            basicField: [''],
            advancedField: ['']
        });

        // Watch for type changes
        this.formGroup.get('type').valueChanges.subscribe(value => {
            this.showAdvanced = value === 'advanced';
            
            if (this.showAdvanced) {
                this.formGroup.get('advancedField').setValidators(Validators.required);
            } else {
                this.formGroup.get('advancedField').clearValidators();
            }
            this.formGroup.get('advancedField').updateValueAndValidity();
        });
    }
}
```

```html
<app-form-control [field]="{
    form: formGroup,
    id: 'type',
    type: 'select',
    label: 'Type',
    dataSource: [
        { value: 'basic', label: 'Basic' },
        { value: 'advanced', label: 'Advanced' }
    ]
}"></app-form-control>

<app-form-control 
    *ngIf="showAdvanced"
    [field]="{
        form: formGroup,
        id: 'advancedField',
        type: 'text',
        label: 'Advanced Setting'
    }">
</app-form-control>
```

---

## ✅ **Validation**

### **Built-in Validators**
```typescript
// Component
this.formGroup = this.formBuilder.group({
    name: ['', [Validators.required, Validators.minLength(2)]],
    email: ['', [Validators.required, Validators.email]],
    age: ['', [Validators.required, Validators.min(18), Validators.max(100)]],
    website: ['', Validators.pattern(/^https?:\/\/.+/)],
    confirmPassword: ['', Validators.required]
}, {
    validators: this.passwordMatchValidator
});

passwordMatchValidator(form: FormGroup) {
    const password = form.get('password');
    const confirmPassword = form.get('confirmPassword');
    
    if (password.value !== confirmPassword.value) {
        confirmPassword.setErrors({ passwordMismatch: true });
    } else {
        confirmPassword.setErrors(null);
    }
}
```

### **Custom Validators**
```typescript
// Custom async validator
emailExistsValidator(control: AbstractControl): Observable<ValidationErrors | null> {
    if (!control.value) return of(null);
    
    return this.userService.checkEmailExists(control.value).pipe(
        map(exists => exists ? { emailExists: true } : null),
        catchError(() => of(null))
    );
}

// Usage
this.formGroup = this.formBuilder.group({
    email: ['', 
        [Validators.required, Validators.email],
        [this.emailExistsValidator.bind(this)]
    ]
});
```

### **Validation Messages**
```html
<app-form-control [field]="{
    form: formGroup,
    id: 'email',
    type: 'email',
    label: 'Email Address'
}">
    <!-- Custom validation messages -->
    <div class="validation-messages" *ngIf="formGroup.get('email').invalid && formGroup.get('email').touched">
        <div *ngIf="formGroup.get('email').errors?.required" class="error-message">
            Email is required
        </div>
        <div *ngIf="formGroup.get('email').errors?.email" class="error-message">
            Please enter a valid email address
        </div>
        <div *ngIf="formGroup.get('email').errors?.emailExists" class="error-message">
            This email is already registered
        </div>
    </div>
</app-form-control>
```

---

## 🎛️ **Event Handling**

### **Value Changes**
```typescript
export class FormEventPage {
    formGroup: FormGroup;

    ngOnInit() {
        // Listen to specific field changes
        this.formGroup.get('category').valueChanges.subscribe(categoryId => {
            this.loadSubCategories(categoryId);
        });

        // Listen to form changes
        this.formGroup.valueChanges.pipe(
            debounceTime(300),
            distinctUntilChanged()
        ).subscribe(formValue => {
            this.autoSave(formValue);
        });
    }

    onFieldChange(event: any) {
        console.log('Field changed:', event);
        // Handle specific field change
    }

    onControlChange(event: any) {
        console.log('Control event:', event);
        // Handle control-specific events
    }

    async loadSubCategories(categoryId: number) {
        if (!categoryId) {
            this.subCategories = [];
            return;
        }

        try {
            this.subCategories = await this.categoryService.getSubCategories(categoryId);
        } catch (error) {
            this.env.showErrorMessage(error);
        }
    }

    async autoSave(formValue: any) {
        if (this.formGroup.valid && this.formGroup.dirty) {
            try {
                await this.service.autoSave(formValue);
                console.log('Auto-saved');
            } catch (error) {
                console.error('Auto-save failed:', error);
            }
        }
    }
}
```

```html
<app-form-control 
    [field]="{
        form: formGroup,
        id: 'category',
        type: 'select',
        label: 'Category',
        dataSource: categories
    }"
    (change)="onFieldChange($event)"
    (controlChange)="onControlChange($event)">
</app-form-control>
```

---

## 🎨 **Styling & Themes**

### **Custom Styling**
```scss
// Component SCSS
app-form-control {
    .form-control-wrapper {
        margin-bottom: 16px;
        
        .control-label {
            font-weight: 500;
            margin-bottom: 8px;
            color: var(--ion-color-dark);
            
            &.required::after {
                content: ' *';
                color: var(--ion-color-danger);
            }
        }
        
        .control-input {
            --border-radius: 8px;
            --padding-start: 12px;
            --padding-end: 12px;
            
            &.ng-invalid.ng-touched {
                --border-color: var(--ion-color-danger);
            }
            
            &.ng-valid.ng-touched {
                --border-color: var(--ion-color-success);
            }
        }
        
        .validation-message {
            font-size: 0.875rem;
            color: var(--ion-color-danger);
            margin-top: 4px;
        }
    }
}
```

### **Theme Variants**
```html
<!-- Filled style -->
<app-form-control 
    [field]="{
        form: formGroup,
        id: 'name',
        type: 'text',
        label: 'Name'
    }"
    class="filled-style">
</app-form-control>

<!-- Outline style -->
<app-form-control 
    [field]="{
        form: formGroup,
        id: 'email',
        type: 'email',
        label: 'Email'
    }"
    class="outline-style">
</app-form-control>
```

---

## 📱 **Responsive Design**

### **Mobile Optimization**
```scss
// Responsive form controls
@media (max-width: 768px) {
    app-form-control {
        .form-control-wrapper {
            margin-bottom: 12px;
            
            .control-label {
                font-size: 0.875rem;
            }
            
            .control-input {
                --min-height: 44px; // Touch-friendly height
                font-size: 16px; // Prevent zoom on iOS
            }
        }
    }
}
```

### **Touch Interactions**
```html
<app-form-control 
    [field]="{
        form: formGroup,
        id: 'date',
        type: 'date',
        label: 'Date'
    }"
    class="touch-optimized">
</app-form-control>
```

---

## 🔧 **Best Practices**

### **1. Field Configuration**
```typescript
// ✅ Đúng - Complete field config
const fieldConfig: InputControlField = {
    form: this.formGroup,
    id: 'email',
    type: 'email',
    label: 'EMAIL_ADDRESS', // Translation key
    placeholder: 'ENTER_EMAIL',
    clearable: true
};

// ❌ Sai - Incomplete config
const fieldConfig = {
    form: this.formGroup,
    id: 'email'
};
```

### **2. Validation Strategy**
```typescript
// ✅ Đúng - Comprehensive validation
this.formGroup = this.formBuilder.group({
    email: ['', 
        [Validators.required, Validators.email],
        [this.asyncEmailValidator.bind(this)]
    ]
});

// ❌ Sai - No validation
this.formGroup = this.formBuilder.group({
    email: ['']
});
```

### **3. Error Handling**
```typescript
// ✅ Đúng - Proper error handling
onSubmit() {
    if (this.formGroup.invalid) {
        this.markFormGroupTouched();
        this.showValidationErrors();
        return;
    }
    
    this.saveData();
}

markFormGroupTouched() {
    Object.keys(this.formGroup.controls).forEach(key => {
        this.formGroup.get(key).markAsTouched();
    });
}
```

---

## 🚨 **Common Patterns**

### **1. Dynamic Forms**
```typescript
export class DynamicFormPage {
    formGroup: FormGroup;
    formFields: InputControlField[] = [];

    async loadFormSchema(formType: string) {
        const schema = await this.schemaService.getFormSchema(formType);
        this.buildDynamicForm(schema);
    }

    buildDynamicForm(schema: any) {
        const formControls = {};
        this.formFields = [];

        schema.fields.forEach(field => {
            formControls[field.id] = [field.defaultValue || '', field.validators || []];
            this.formFields.push({
                form: this.formGroup,
                id: field.id,
                type: field.type,
                label: field.label,
                dataSource: field.options
            });
        });

        this.formGroup = this.formBuilder.group(formControls);
    }
}
```

### **2. Wizard Forms**
```typescript
export class WizardFormPage {
    currentStep = 0;
    steps = ['basic', 'details', 'confirmation'];
    
    stepForms: { [key: string]: FormGroup } = {
        basic: this.formBuilder.group({
            name: ['', Validators.required],
            email: ['', [Validators.required, Validators.email]]
        }),
        details: this.formBuilder.group({
            address: ['', Validators.required],
            phone: ['', Validators.required]
        })
    };

    nextStep() {
        const currentForm = this.stepForms[this.steps[this.currentStep]];
        
        if (currentForm.invalid) {
            this.markFormGroupTouched(currentForm);
            return;
        }

        this.currentStep++;
    }

    previousStep() {
        this.currentStep--;
    }
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
