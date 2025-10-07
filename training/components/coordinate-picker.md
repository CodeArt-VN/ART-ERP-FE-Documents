# CoordinatePicker Component Documentation

## 📋 **Tổng quan**

`CoordinatePickerComponent` là component cho phép người dùng chọn tọa độ GPS trên bản đồ hoặc nhập tọa độ thủ công.

**Selector**: `app-coordinate-picker`  
**Location**: `src/app/components/map-comps/coordinate-picker/coordinate-picker.component.ts`

---

## 🚀 **Basic Usage**

### **Location Picker**
```typescript
// Component
export class LocationFormPage extends PageBase {
    selectedCoordinates = { lat: 21.0285, lng: 105.8542 };

    onCoordinateSelected(coords: any) {
        this.selectedCoordinates = coords;
        this.formGroup.patchValue({
            latitude: coords.lat,
            longitude: coords.lng
        });
    }
}
```

```html
<!-- Template -->
<app-coordinate-picker
    [initialCoordinates]="selectedCoordinates"
    (coordinateSelected)="onCoordinateSelected($event)">
</app-coordinate-picker>
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
