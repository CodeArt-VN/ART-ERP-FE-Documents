# MapView Component Documentation

## 📋 **Tổng quan**

`MapViewComponent` là component hiển thị bản đồ tương tác, hỗ trợ hiển thị markers, routes và các tính năng bản đồ khác cho ứng dụng ERP.

**Selector**: `app-map-view`  
**Location**: `src/app/components/map-comps/map-view/map-view.component.ts`

---

## 🚀 **Basic Usage**

### **Simple Map Display**
```typescript
// Component
export class LocationViewPage extends PageBase {
    mapCenter = { lat: 21.0285, lng: 105.8542 }; // Hanoi
    markers = [
        { lat: 21.0285, lng: 105.8542, title: 'Head Office', info: 'Main headquarters' },
        { lat: 21.0245, lng: 105.8412, title: 'Branch 1', info: 'District 1 branch' }
    ];
}
```

```html
<!-- Template -->
<app-map-view
    [center]="mapCenter"
    [markers]="markers"
    [zoom]="12">
</app-map-view>
```

---

## 🗺️ **Advanced Features**

### **Delivery Route Tracking**
```typescript
export class DeliveryTrackingPage extends PageBase {
    deliveryRoute = {
        start: { lat: 21.0285, lng: 105.8542 },
        end: { lat: 21.0245, lng: 105.8412 },
        waypoints: [
            { lat: 21.0265, lng: 105.8502 },
            { lat: 21.0255, lng: 105.8462 }
        ]
    };

    vehicleLocation = { lat: 21.0265, lng: 105.8502 };
}
```

### **Store Locator**
```typescript
export class StoreLocatorPage extends PageBase {
    stores = [
        {
            id: 1,
            name: 'Store A',
            address: '123 Main St, Hanoi',
            lat: 21.0285,
            lng: 105.8542,
            phone: '024-1234-5678'
        }
    ];
}
```

---

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
