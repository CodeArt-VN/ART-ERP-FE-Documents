# Address Component Documentation

## 📋 **Tổng quan**

`AddressComponent` là component để nhập và quản lý địa chỉ, hỗ trợ tích hợp với bản đồ và geocoding.

**Selector**: `app-address`  
**Location**: `src/app/components/map-comps/address/address.component.ts`

---

## 🚀 **Basic Usage**

### **Address Input Form**
```typescript
// Component
export class AddressFormPage extends PageBase {
    addressData = {
        street: '',
        ward: '',
        district: '',
        city: '',
        country: 'Vietnam',
        postalCode: ''
    };

    onAddressChange(address: any) {
        this.addressData = address;
        this.formGroup.patchValue(address);
    }
}
```

```html
<!-- Template -->
<app-address
    [address]="addressData"
    (addressChange)="onAddressChange($event)">
</app-address>
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
