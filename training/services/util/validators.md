# Custom Validators Documentation

## 📄 **Tổng quan**

File `validators.ts` chứa các custom validator classes để sử dụng với Angular Reactive Forms. Các validators này cung cấp validation logic tùy chỉnh cho các trường hợp cụ thể trong ứng dụng ART-ERP.

## 🎯 **Mục đích sử dụng**

- **Form validation**: Kiểm tra tính hợp lệ của form inputs
- **Custom rules**: Validation logic phức tạp không có sẵn trong Angular
- **User experience**: Cung cấp feedback tức thì cho user
- **Data integrity**: Đảm bảo dữ liệu nhập vào đúng format

## 📍 **File location**
```
src/app/services/util/validators.ts
```

## 🔧 **Dependencies**

```typescript
import { FormControl } from '@angular/forms';
```

## 🏗️ **Validator Classes**

### **1. CompareValidator**

#### **confirmPassword(control)**
Validator để xác nhận password khớp với password gốc.

```typescript
static confirmPassword(control: any): any
```

**Parameters:**
- `control`: FormControl cần validate

**Returns:** 
- `null` nếu valid
- `{ notvalid: true }` nếu không khớp

**Logic:**
- So sánh với field `newPassword` trong cùng FormGroup
- Trả về error nếu không khớp

**Example:**
```typescript
// Trong component
this.passwordForm = this.formBuilder.group({
    newPassword: ['', [Validators.required, Validators.minLength(8)]],
    confirmPassword: ['', [Validators.required, CompareValidator.confirmPassword]]
});

// Trong template
<ion-item>
    <ion-label position="stacked">Mật khẩu mới</ion-label>
    <ion-input 
        type="password" 
        formControlName="newPassword">
    </ion-input>
</ion-item>

<ion-item>
    <ion-label position="stacked">Xác nhận mật khẩu</ion-label>
    <ion-input 
        type="password" 
        formControlName="confirmPassword">
    </ion-input>
    <ion-note 
        slot="error" 
        *ngIf="passwordForm.get('confirmPassword')?.hasError('notvalid')">
        Mật khẩu xác nhận không khớp
    </ion-note>
</ion-item>
```

### **2. AgeValidator**

#### **isValid(control)**
Validator để kiểm tra tuổi hợp lệ (18-120 tuổi, số nguyên).

```typescript
static isValid(control: FormControl): any
```

**Parameters:**
- `control`: FormControl chứa giá trị tuổi

**Returns:**
- `null` nếu valid
- `{ 'not a number': true }` nếu không phải số
- `{ 'not a whole number': true }` nếu không phải số nguyên
- `{ 'too young': true }` nếu < 18 tuổi
- `{ 'not realistic': true }` nếu > 120 tuổi

**Example:**
```typescript
// Trong component
this.userForm = this.formBuilder.group({
    name: ['', Validators.required],
    age: ['', [Validators.required, AgeValidator.isValid]]
});

// Trong template
<ion-item>
    <ion-label position="stacked">Tuổi</ion-label>
    <ion-input 
        type="number" 
        formControlName="age">
    </ion-input>
    
    <!-- Error messages -->
    <ion-note slot="error" *ngIf="userForm.get('age')?.hasError('not a number')">
        Vui lòng nhập số
    </ion-note>
    <ion-note slot="error" *ngIf="userForm.get('age')?.hasError('not a whole number')">
        Tuổi phải là số nguyên
    </ion-note>
    <ion-note slot="error" *ngIf="userForm.get('age')?.hasError('too young')">
        Tuổi phải từ 18 trở lên
    </ion-note>
    <ion-note slot="error" *ngIf="userForm.get('age')?.hasError('not realistic')">
        Tuổi không hợp lệ (tối đa 120)
    </ion-note>
</ion-item>
```

### **3. UsernameValidator**

#### **checkUsername(control)**
Async validator để kiểm tra username có bị trùng không.

```typescript
static checkUsername(control: FormControl): Promise<any>
```

**Parameters:**
- `control`: FormControl chứa username

**Returns:**
- Promise resolve `null` nếu valid
- Promise resolve `{ 'username taken': true }` nếu username đã tồn tại

**Logic:**
- Kiểm tra username 'greg' (demo purpose)
- Có thể mở rộng để check với server

**Example:**
```typescript
// Trong component
this.registrationForm = this.formBuilder.group({
    username: ['', 
        [Validators.required, Validators.minLength(3)],
        [UsernameValidator.checkUsername] // Async validator
    ],
    email: ['', [Validators.required, Validators.email]]
});

// Trong template
<ion-item>
    <ion-label position="stacked">Username</ion-label>
    <ion-input 
        type="text" 
        formControlName="username">
    </ion-input>
    
    <!-- Loading indicator for async validation -->
    <ion-spinner 
        slot="end" 
        *ngIf="registrationForm.get('username')?.pending">
    </ion-spinner>
    
    <!-- Error message -->
    <ion-note 
        slot="error" 
        *ngIf="registrationForm.get('username')?.hasError('username taken')">
        Username đã được sử dụng
    </ion-note>
</ion-item>
```

## 🎨 **Advanced Usage Examples**

### **1. Comprehensive Registration Form**
```typescript
export class RegistrationPage {
    registrationForm: FormGroup;

    constructor(private formBuilder: FormBuilder) {
        this.createForm();
    }

    createForm() {
        this.registrationForm = this.formBuilder.group({
            username: ['', 
                [Validators.required, Validators.minLength(3)],
                [UsernameValidator.checkUsername]
            ],
            age: ['', [Validators.required, AgeValidator.isValid]],
            password: ['', [Validators.required, Validators.minLength(8)]],
            confirmPassword: ['', [Validators.required, CompareValidator.confirmPassword]],
            email: ['', [Validators.required, Validators.email]]
        });
    }

    async onSubmit() {
        if (this.registrationForm.valid) {
            const formData = this.registrationForm.value;
            // Process registration
            console.log('Registration data:', formData);
        } else {
            this.markFormGroupTouched();
        }
    }

    private markFormGroupTouched() {
        Object.keys(this.registrationForm.controls).forEach(key => {
            const control = this.registrationForm.get(key);
            control?.markAsTouched();
        });
    }

    // Helper methods for template
    isFieldInvalid(fieldName: string): boolean {
        const field = this.registrationForm.get(fieldName);
        return !!(field && field.invalid && (field.dirty || field.touched));
    }

    getFieldError(fieldName: string): string {
        const field = this.registrationForm.get(fieldName);
        if (field?.errors) {
            const errors = field.errors;
            
            if (errors['required']) return 'Trường này là bắt buộc';
            if (errors['minlength']) return `Tối thiểu ${errors['minlength'].requiredLength} ký tự`;
            if (errors['email']) return 'Email không hợp lệ';
            if (errors['notvalid']) return 'Mật khẩu xác nhận không khớp';
            if (errors['not a number']) return 'Vui lòng nhập số';
            if (errors['too young']) return 'Tuổi phải từ 18 trở lên';
            if (errors['username taken']) return 'Username đã được sử dụng';
        }
        
        return '';
    }
}
```

### **2. Custom Validator Factory**
```typescript
export class CustomValidatorFactory {
    /**
     * Create password confirmation validator for specific field
     */
    static createPasswordConfirmValidator(passwordFieldName: string) {
        return (control: AbstractControl): ValidationErrors | null => {
            if (!control.parent) return null;
            
            const password = control.parent.get(passwordFieldName);
            const confirmPassword = control;
            
            if (password && confirmPassword && password.value !== confirmPassword.value) {
                return { passwordMismatch: true };
            }
            
            return null;
        };
    }

    /**
     * Create age range validator
     */
    static createAgeRangeValidator(minAge: number, maxAge: number) {
        return (control: FormControl): ValidationErrors | null => {
            const age = control.value;
            
            if (isNaN(age)) return { notANumber: true };
            if (age % 1 !== 0) return { notWholeNumber: true };
            if (age < minAge) return { tooYoung: { minAge } };
            if (age > maxAge) return { tooOld: { maxAge } };
            
            return null;
        };
    }

    /**
     * Create async username validator with server check
     */
    static createUsernameValidator(userService: any) {
        return (control: FormControl): Promise<ValidationErrors | null> => {
            if (!control.value) return Promise.resolve(null);
            
            return userService.checkUsernameAvailability(control.value)
                .toPromise()
                .then((available: boolean) => {
                    return available ? null : { usernameTaken: true };
                })
                .catch(() => {
                    // Handle error - assume username is available
                    return null;
                });
        };
    }
}

// Usage
this.form = this.formBuilder.group({
    password: ['', Validators.required],
    confirmPassword: ['', [
        Validators.required,
        CustomValidatorFactory.createPasswordConfirmValidator('password')
    ]],
    age: ['', [
        Validators.required,
        CustomValidatorFactory.createAgeRangeValidator(16, 65)
    ]],
    username: ['', 
        [Validators.required],
        [CustomValidatorFactory.createUsernameValidator(this.userService)]
    ]
});
```

### **3. Real-time Validation with Debounce**
```typescript
export class SmartValidationComponent implements OnInit {
    form: FormGroup;
    usernameCheckSubject = new Subject<string>();

    constructor(
        private formBuilder: FormBuilder,
        private userService: UserService
    ) {
        this.createForm();
        this.setupUsernameValidation();
    }

    ngOnInit() {
        // Setup real-time validation with debounce
        this.usernameCheckSubject.pipe(
            debounceTime(500),
            distinctUntilChanged(),
            switchMap(username => this.userService.checkUsername(username))
        ).subscribe(result => {
            const usernameControl = this.form.get('username');
            if (result.taken) {
                usernameControl?.setErrors({ usernameTaken: true });
            } else {
                // Clear username taken error but keep other errors
                const errors = usernameControl?.errors;
                if (errors) {
                    delete errors['usernameTaken'];
                    const hasOtherErrors = Object.keys(errors).length > 0;
                    usernameControl?.setErrors(hasOtherErrors ? errors : null);
                }
            }
        });
    }

    createForm() {
        this.form = this.formBuilder.group({
            username: ['', [Validators.required, Validators.minLength(3)]],
            age: ['', [Validators.required, AgeValidator.isValid]],
            password: ['', [Validators.required, Validators.minLength(8)]],
            confirmPassword: ['', [Validators.required, CompareValidator.confirmPassword]]
        });
    }

    setupUsernameValidation() {
        this.form.get('username')?.valueChanges.subscribe(value => {
            if (value && value.length >= 3) {
                this.usernameCheckSubject.next(value);
            }
        });
    }
}
```

### **4. Cross-field Validation**
```typescript
export class CrossFieldValidators {
    /**
     * Validate that end date is after start date
     */
    static dateRange(startDateField: string, endDateField: string) {
        return (formGroup: AbstractControl): ValidationErrors | null => {
            const startDate = formGroup.get(startDateField)?.value;
            const endDate = formGroup.get(endDateField)?.value;
            
            if (startDate && endDate && new Date(startDate) >= new Date(endDate)) {
                return { dateRangeInvalid: true };
            }
            
            return null;
        };
    }

    /**
     * Validate that at least one field is filled
     */
    static atLeastOne(fields: string[]) {
        return (formGroup: AbstractControl): ValidationErrors | null => {
            const hasValue = fields.some(field => {
                const control = formGroup.get(field);
                return control && control.value && control.value.trim();
            });
            
            return hasValue ? null : { atLeastOneRequired: true };
        };
    }
}

// Usage
this.form = this.formBuilder.group({
    startDate: ['', Validators.required],
    endDate: ['', Validators.required],
    phone: [''],
    email: ['']
}, {
    validators: [
        CrossFieldValidators.dateRange('startDate', 'endDate'),
        CrossFieldValidators.atLeastOne(['phone', 'email'])
    ]
});
```

## 📋 **Best Practices**

### **1. Error Message Management**
```typescript
export class ValidationMessageService {
    private errorMessages = {
        required: 'Trường này là bắt buộc',
        email: 'Email không hợp lệ',
        minlength: 'Tối thiểu {requiredLength} ký tự',
        maxlength: 'Tối đa {requiredLength} ký tự',
        notvalid: 'Mật khẩu xác nhận không khớp',
        'not a number': 'Vui lòng nhập số',
        'too young': 'Tuổi phải từ 18 trở lên',
        'username taken': 'Username đã được sử dụng'
    };

    getErrorMessage(errors: ValidationErrors): string {
        const errorKey = Object.keys(errors)[0];
        let message = this.errorMessages[errorKey] || 'Dữ liệu không hợp lệ';
        
        // Replace placeholders
        if (errors[errorKey] && typeof errors[errorKey] === 'object') {
            Object.keys(errors[errorKey]).forEach(key => {
                message = message.replace(`{${key}}`, errors[errorKey][key]);
            });
        }
        
        return message;
    }
}
```

### **2. Reusable Validation Component**
```typescript
@Component({
    selector: 'app-form-field',
    template: `
        <ion-item>
            <ion-label position="stacked">{{ label }}</ion-label>
            <ng-content></ng-content>
            <ion-note 
                slot="error" 
                *ngIf="control?.invalid && (control?.dirty || control?.touched)">
                {{ getErrorMessage() }}
            </ion-note>
        </ion-item>
    `
})
export class FormFieldComponent {
    @Input() label: string;
    @Input() control: AbstractControl;

    constructor(private validationService: ValidationMessageService) {}

    getErrorMessage(): string {
        if (this.control?.errors) {
            return this.validationService.getErrorMessage(this.control.errors);
        }
        return '';
    }
}

// Usage in template
<app-form-field label="Username" [control]="form.get('username')">
    <ion-input formControlName="username"></ion-input>
</app-form-field>
```

### **3. Performance Optimization**
```typescript
// Sử dụng OnPush change detection cho validation-heavy forms
@Component({
    selector: 'app-registration',
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class RegistrationComponent {
    // Minimize validation calls
    private validationDebounce = new Subject<void>();

    ngOnInit() {
        // Debounce validation to improve performance
        this.validationDebounce.pipe(
            debounceTime(300)
        ).subscribe(() => {
            this.cdr.detectChanges();
        });

        this.form.valueChanges.subscribe(() => {
            this.validationDebounce.next();
        });
    }
}
```

## 🔧 **Integration với EnvService**

```typescript
export class EnhancedValidators {
    constructor(private env: EnvService) {}

    /**
     * Async validator with server-side validation
     */
    createServerValidator(endpoint: string, errorMessage: string) {
        return (control: FormControl): Observable<ValidationErrors | null> => {
            if (!control.value) return of(null);

            return this.env.commonService.connect('POST', endpoint, { value: control.value })
                .pipe(
                    map(result => result.isValid ? null : { serverValidation: { message: errorMessage } }),
                    catchError(() => of(null)) // Assume valid on error
                );
        };
    }

    /**
     * Validator with user-friendly error display
     */
    createValidatorWithMessage(validatorFn: ValidatorFn, errorKey: string, message: string) {
        return (control: AbstractControl): ValidationErrors | null => {
            const result = validatorFn(control);
            if (result) {
                // Show user-friendly message
                this.env.showMessage(message, 'warning');
                return { [errorKey]: true };
            }
            return null;
        };
    }
}
```

Custom validators cung cấp validation logic mạnh mẽ và linh hoạt cho ứng dụng ART-ERP, đảm bảo data integrity và user experience tốt.
