# Kế hoạch: Bỏ app-detail-toolbar, thay bằng app-toolbar

> **Trạng thái**: ✅ Hoàn thành (Phase 1–4)

## 1. Tổng quan

- **Mục tiêu**: Thống nhất toolbar, dùng `app-toolbar` cho cả list và detail pages.
- **Phạm vi**: ~50 trang detail đang dùng `app-detail-toolbar`.
- **Lợi ích**: Một component duy nhất, dễ bảo trì, UX nhất quán.

---

## 2. So sánh API

| detail-toolbar | toolbar | Ghi chú |
|----------------|---------|---------|
| `[page]="this"` | `[page]="this"` | Giống nhau |
| `[title]` | `[CenterTitle]` | toolbar dùng CenterTitle cho title giữa |
| `[BackHref]` | `[BackHref]` | Giống nhau |
| `[ShowFeature]` | `[ShowFeature]` | Giống nhau |
| `[ShowDelete]` | `[ShowDelete]` | Giống nhau |
| `[ShowHelp]` | `[ShowHelp]` | Giống nhau |
| `[ShowRefresh]` | `page.pageConfig.ShowRefresh` | toolbar lấy từ pageConfig |
| `[NoBorder]` | `[NoBorder]` | Giống nhau |
| `page.isSubActive` | `page.pageConfig.isSubActive` | Khác property |
| `page.backSubPage()` | `page.backToMainView()` | Khác tên method |

---

## 3. Điều kiện tiên quyết

### 3.1 Sửa PageBase (nếu cần)

- `detail-toolbar` dùng `page.backSubPage()` nhưng PageBase chỉ có `backToMainView()`.
- **Hành động**: Thêm alias `backSubPage = backToMainView` trong PageBase, hoặc cập nhật detail-toolbar để dùng `backToMainView` (sẽ không cần nếu bỏ detail-toolbar).

### 3.2 Sửa app-toolbar (nếu cần)

- Đảm bảo `app-toolbar` hoạt động đúng khi `page.pageConfig.isDetailPage === true` (đã có).
- Kiểm tra `page.pageConfig.isSubActive` và `page.backToMainView()` (đã có).

---

## 4. Danh sách file cần thay thế

### 4.1 Thay thế đơn giản (chỉ `[page]="this"`)

```
src/app/pages/CRM/reward-category-detail/
src/app/pages/SHIP/shipping-route-detail/
src/app/pages/CRM/brand-detail/
src/app/pages/CRM/member-card-detail/
src/app/pages/OSM/notification-template-detail/
src/app/pages/POS/pos-booking-detail/
src/app/pages/HRM/shift-detail/
src/app/pages/SYS/profile/
src/app/pages/HRM/insurance-policy-detail/
src/app/pages/CRM/membership-loyalty-detail/
src/app/pages/FINANCIAL/tax-definition-detail/
src/app/pages/WMS/carton-detail/
src/app/pages/CRM/level-policy-detail/
src/app/pages/SHIP/vehicle-detail/
src/app/pages/SHIP/shipment-detail/
src/app/pages/HRM/paid-time-off-policy-detail/
src/app/pages/CRM/lead-detail/
src/app/pages/HRM/holiday-policy-detail/
src/app/pages/HRM/ptos-enrollment-detail/
src/app/pages/PR/pr-discount-policy-detail/
src/app/pages/HRM/work-rule-detail/
src/app/pages/HRM/insurance-enrollment-detail/
src/app/pages/HRM/staff-time-off-request-detail/
src/app/pages/PR/pr-voucher-policy-detail/
src/app/pages/HRM/timesheet-detail/
src/app/pages/CRM/loyalty-policy-detail/
src/app/pages/WMS/shipping-detail/
src/app/pages/HRM/contract-template-detail/
src/app/pages/CRM/campaign-detail/
src/app/pages/HRM/user-device-detail/
src/app/pages/HRM/checkin-gate/
src/app/pages/HRM/employee-policy-detail/
src/app/pages/CRM/mcp-detail/
src/app/pages/WMS/cycle-count-detail/
src/app/pages/WMS/packing-order-detail/
src/app/pages/PR/pr-deal-detail/
src/app/pages/OSM/notification-segment-detail/
src/app/pages/HRM/staff-policy-enrollment-detail/
src/app/pages/CRM/reward-detail/
src/app/pages/OSM/notification-detail/
src/app/pages/HRM/work-rule-violation-detail/
src/app/pages/WMS/zone-detail/
src/app/pages/WMS/outbound-order-detail/
src/app/pages/ACCOUNTANT/bank-account-detail/
src/app/pages/_template/flat-detail/
src/app/pages/HRM/leave-type-detail/
src/app/pages/WMS/location-detail/
src/app/pages/CRM/attendance-booking-detail/
src/app/pages/WMS/item-uom-detail/
src/app/pages/CRM/benefit-policy-detail/
src/app/pages/WMS/picking-order-detail/
src/app/pages/OST/office-detail/
```

### 4.2 Có tham số đặc biệt

| File | Thay đổi |
|------|----------|
| `close-order/close-order.page.html` | `[BackHref]="pageConfig.pageName+'/'+id"` `[ShowDelete]="false"` → giữ nguyên trên app-toolbar |
| `sale-order-mobile-detail/` | `[title]="'#'+item.Id"` → `[CenterTitle]="'#'+item.Id"` |
| `sale-order-mobile-viewer/` | `[title]="'#'+item.Id"` → `[CenterTitle]="'#'+item.Id"` |
| `item-uom-detail/` | `[title]="'#'+product?.Id"` → `[CenterTitle]="'#'+product?.Id"` |

### 4.3 Có custom content (ng-content)

Các trang có nút/button tùy chỉnh trong toolbar:

- `shipping-detail` – ion-buttons slot="primary"
- `cycle-count-detail` – ion-buttons slot="primary"
- `packing-order-detail` – ion-buttons
- `picking-order-detail` – ion-buttons
- `outbound-order-detail` – ion-buttons
- `paid-time-off-policy-detail` – ng-content
- `pr-voucher-policy-detail` – ion-button
- `ar-invoice-detail` – đã dùng app-toolbar (tham khảo)
- `sale-quotation-detail` – đã dùng app-toolbar (tham khảo)

**Cách xử lý**: Đặt custom content trong `<app-toolbar>` (ng-content mặc định của toolbar) hoặc dùng `[endSlot]` nếu cần vị trí cuối.

---

## 5. Các bước thực hiện

### Phase 1: Chuẩn bị app-toolbar

1. Kiểm tra app-toolbar hoạt động đúng với `page.pageConfig.isDetailPage === true`.
2. (Tùy chọn) Thêm alias `backSubPage` trong PageBase nếu còn tham chiếu.
3. Đảm bảo `page.isSubActive` → `page.pageConfig.isSubActive` (detail-toolbar dùng `page.isSubActive`, có thể là lỗi hoặc alias).

### Phase 2: Thay thế từng nhóm

1. **Nhóm 1 – Đơn giản**: ~40 trang chỉ có `<app-detail-toolbar [page]="this"></app-detail-toolbar>`  
   - Thay bằng: `<app-toolbar [page]="this"></app-toolbar>`

2. **Nhóm 2 – Có tham số**: 4 trang có `BackHref`, `ShowDelete`, `title`  
   - Map sang `BackHref`, `ShowDelete`, `CenterTitle`.

3. **Nhóm 3 – Có custom content**: ~10 trang  
   - Giữ nguyên nội dung bên trong, đổi selector từ `app-detail-toolbar` sang `app-toolbar`.

### Phase 3: Dọn dẹp

1. Xóa `DetailToolbarComponent` khỏi `share-components.module.ts`.
2. Xóa thư mục `src/app/components/detail-toolbar/`.
3. Cập nhật `src/theme/common.scss`: bỏ `app-detail-toolbar` khỏi selector (nếu cần).
4. Cập nhật `.docs/training/components/detail-toolbar.md` hoặc xóa nếu không còn dùng.
5. Cập nhật `replace.translate.js` nếu có key riêng cho detail-toolbar.

### Phase 4: Kiểm tra

1. Chạy build: `ng build` hoặc `npm run build`.
2. Test thủ công các trang detail quan trọng.
3. Kiểm tra responsive (mobile/desktop).
4. Kiểm tra sub-page / feature panel (isSubActive, backToMainView).

---

## 6. Rủi ro và lưu ý

| Rủi ro | Cách xử lý |
|--------|------------|
| `page.isSubActive` vs `page.pageConfig.isSubActive` | Toolbar dùng `pageConfig.isSubActive`; cần đảm bảo page luôn set đúng. |
| `backSubPage` vs `backToMainView` | Toolbar dùng `backToMainView`; PageBase đã có sẵn. |
| Custom content slot khác | Kiểm tra từng trang có ng-content; dùng `[endSlot]` nếu cần. |
| `page.item` vs `page.Item` | detail-toolbar dùng `page.item` (chữ thường); toolbar cũng dùng `page.item`. |

---

## 7. modal-detail-toolbar

- `app-modal-detail-toolbar` hiện không được dùng trong template nào.
- Có thể xóa cùng lúc với detail-toolbar hoặc để lại nếu có kế hoạch dùng sau.

---

## 8. Thứ tự ưu tiên

1. Phase 1 – Chuẩn bị.
2. Phase 2 – Nhóm 1 (đơn giản).
3. Phase 2 – Nhóm 2 (có tham số).
4. Phase 2 – Nhóm 3 (có custom content).
5. Phase 3 – Dọn dẹp.
6. Phase 4 – Kiểm tra.
