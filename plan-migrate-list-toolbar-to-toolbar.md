# Kế hoạch: Bỏ app-list-toolbar, thay bằng app-toolbar

## 1. Tổng quan

- **Mục tiêu**: Thống nhất toolbar, dùng `app-toolbar` cho cả list và detail pages.
- **Phạm vi**: ~35 trang list đang dùng `app-list-toolbar`.
- **Trạng thái**: Hoàn thành (2025-03). Đã xóa `ListToolbarComponent`.
- **Độ phức tạp**: Cao hơn detail-toolbar vì list-toolbar có nhiều domain-specific outputs và ngOnChanges.

---

## 2. So sánh API

### 2.1 Input

| list-toolbar | toolbar | Ghi chú |
|--------------|---------|---------|
| `[pageConfig]` | `page.pageConfig` | toolbar lấy từ [page] |
| `[selectedItems]` | `page.selectedItems` | toolbar lấy từ [page] |
| `[query]` | `page.query` | toolbar lấy từ [page] |
| `[page]` (optional) | `[page]` (bắt buộc) | toolbar yêu cầu page |
| `[pageTitle]` | `[pageTitle]` | toolbar có input này |
| `[selectedTitle]` | — | toolbar dùng `SELECTED_ITEMS` mặc định |
| `[ShowAdd]`, `[ShowSearch]`, ... | `[ShowAdd]`, `[ShowSearch]`, ... | Giống nhau |
| `[ShowPopover]` | — | Cần thêm hoặc dùng ng-content |
| `[ShowChangeTable]` | — | Cần thêm hoặc ng-content |

### 2.2 Pattern gọi action

| list-toolbar | toolbar |
|--------------|---------|
| `(add)="add()"` → emit → parent | `(click)="page.add()"` trực tiếp |
| `(refresh)="refresh()"` | `page.refresh()` |
| Tất cả actions qua EventEmitter | Gọi trực tiếp method trên page |

### 2.3 Sub-page / Back (flow chuẩn theo app-toolbar)

| list-toolbar | toolbar (chuẩn) |
|--------------|-----------------|
| `pageConfig.isMainPageActive` | `pageConfig.isSubActive` |
| `page.backSubPage()` | `page.backToMainView()` |

**Flow chuẩn app-toolbar**: Dùng `isSubActive` + `backToMainView()`. Không thêm `isMainPageActive` hay `backSubPage` vào toolbar. List pages khi migrate sẽ dùng flow toolbar.

### 2.4 Ng-content slots

| list-toolbar | toolbar |
|--------------|---------|
| `select="[title]"` (start) | `select="[startTitle]"` |
| default (slot end) | default (slot end) |
| `select="[selected]"` (khi có selected) | `select="[tollbarSelected]"` |

---

## 3. Domain-specific actions (list-toolbar có, toolbar chưa có)

Các action cần xử lý qua **ng-content** (page tự thêm button):

| Action | Trang dùng | Cách xử lý |
|--------|------------|------------|
| `submitReceipt` | goods-receiving | Custom button [tollbarSelected] hoặc default |
| `submitOrdersForApproval` | sale-order, purchase-order | toolbar có submitForApproval – map canSubmit |
| `createReceipt` | sale-order | Custom button |
| `createEInvoice` | ar-invoice | ar-invoice đã dùng toolbar + [tollbarSelected] |
| `presentPopover` | sale-order-mobile | Custom button |
| `submitDealForApproval`, `approveDeal` | deal pages | Custom hoặc extend toolbar |
| `submitVoucherForApproval`, `approveVoucher` | voucher pages | Custom |
| `submitDiscountForApproval`, `approveDiscount` | discount pages | Custom |
| `changeTable` | POS | Custom |
| `cancelOrder` | POS | Custom |

**Chiến lược**: Action generic (add, refresh, approve, merge, split...) dùng toolbar sẵn. Action domain-specific dùng ng-content `[tollbarSelected]` hoặc default slot.

---

## 4. ngOnChanges trong list-toolbar

list-toolbar có ~200 dòng `ngOnChanges` để show/hide buttons theo:
- `pageConfig.pageName` (sale-order, purchase-order, arinvoice, business-partner, request, adjustment)
- `selectedItems` status

**Chiến lược**: Logic tương tự đưa vào page (ví dụ `showCommandBySelectedRows()`) hoặc thêm vào toolbar khi `pageConfig.pageName` khớp. ar-invoice đã có `showCommandBySelectedRows()` – có thể áp dụng pattern tương tự.

---

## 5. Danh sách trang cần migrate

### 5.1 Đơn giản (chỉ CRUD cơ bản)

~25 trang: cycle-count, vehicle, bill-of-materials, user-device, gantt, picking-order, warehouse, office, packing-order, receivable-debt, carton, shipping-route, item-group, system-status, system-type, mcp, shipping, pr-discount-policy, setting, putaway-strategy, general-ledger, tax-definition, storer, zone, outbound-order, approval-rule, saleman-debt, ...

### 5.2 Có custom content (ng-content)

- **checkin-log**: [title] = ng-select timesheet, default = Today/Prev/Next/Filter
- **goods-receiving**: submitReceipt
- **price-report**: ion-button + toggle
- **sale-order-mobile**: ShowPopover, mergeOrders, splitOrder, submitOrdersForApproval, presentPopover
- **branch-payroll-report**: custom content
- **item**: custom content
- **checkin-log**: custom content phức tạp

### 5.3 Đặc biệt

- **packing-order**: Đang comment app-list-toolbar
- **approval-template**: Đang comment app-list-toolbar

---

## 6. Flow nút Back giả (app-toolbar – chuẩn)

Toolbar có **nút Back giả** dùng để ẩn/hiện nội dung trên màn hình nhỏ (mobile):

1. **Vị trí**: `ion-buttons slot="start"` với `class="ion-hide-md-up"` → chỉ hiện trên mobile (ẩn trên desktop).
2. **Điều kiện**: `*ngIf="page.pageConfig.isSubActive"` → chỉ hiện khi feature panel đang mở.
3. **Hành vi**: Click → `page.backToMainView()` → `isSubActive = false` → đóng feature panel, hiện main content.
4. **Liên kết với toggleFeature()**: Khi user bấm "Show feature", `toggleFeature()` set `isSubActive = isShowFeature`. Khi `isSubActive = true`, trên mobile hiện nút Back giả để quay lại main view.

**Không** thêm `isMainPageActive` hay `backSubPage` vào toolbar. Flow chuẩn là `isSubActive` + `backToMainView()`.

---

## 7. Các phase thực hiện

### Phase 1: Hướng dẫn sử dụng app-toolbar đúng cách (không sửa toolbar)

**Mục tiêu**: List pages dùng app-toolbar đúng cách để giữ nguyên nghiệp vụ, không thay đổi app-toolbar.

#### 7.1.1 Nguyên tắc: Không sửa app-toolbar

- Toolbar đã chuẩn (flow Back giả, isSubActive, backToMainView).
- List pages phải **adapt** để dùng đúng API toolbar.

#### 7.1.2 Cách dùng toolbar cho list page

**Template:**
```html
<app-toolbar [page]="this">
  <!-- Custom buttons (domain-specific) qua ng-content -->
  <ion-button *ngIf="pageConfig.canSubmitReceipt" (click)="submitReceipt()" ...>...</ion-button>
</app-toolbar>
```

**Page (TS):**
- Truyền `[page]="this"` – toolbar lấy `pageConfig`, `selectedItems`, `query` từ page.
- `pageConfig.isDetailPage = false` (mặc định cho list).
- Các flag: `ShowAdd`, `ShowSearch`, `ShowRefresh`, `ShowExport`, `ShowImport`, `ShowArchive`, `ShowHelp`, `ShowFeature`.
- Các quyền: `canAdd`, `canExport`, `canImport`, `canDelete`, `canApprove`, `canMerge`, `canSplit`, ...

**Feature panel (isSubActive / backToMainView):**
- Khi user bấm "Show feature", `toggleFeature()` (PageBase) set `isSubActive = isShowFeature`.
- Trên mobile, nút Back giả hiện khi `isSubActive = true`, gọi `backToMainView()`.
- List page **không** dùng `isMainPageActive` hay `backSubPage` – dùng `isSubActive` và `backToMainView()`.

#### 7.1.3 Giữ nghiệp vụ: Dynamic buttons theo status

list-toolbar có ngOnChanges theo `selectedItems` status. Cách thay thế:

1. **PageBase.showCommandBySelectedRows()**: Đã có – cập nhật `pageConfig.ShowApprove`, `ShowMerge`, ... theo rules.
2. **toolbarCommandRules**: Provider đăng ký rules (PURCHASE_Order, AC_ARInvoice, ...). Page dùng `(selectedRowsChange)="showCommandBySelectedRows($event)"` trên data-table.
3. **Toolbar** đọc `pageConfig.ShowApprove`, `pageConfig.canMerge`, ... – không cần sửa.

**Ví dụ ar-invoice**: Dùng toolbar + `showCommandBySelectedRows` + custom buttons `[tollbarSelected]` với `*ngIf="pageConfig.ShowCreateEInvoice"`.

#### 7.1.4 Domain-specific actions

Actions toolbar chưa có (submitReceipt, createReceipt, presentPopover, ...):

- Thêm **custom buttons** qua ng-content (default slot hoặc `[tollbarSelected]`).
- Button gọi method trên page: `(click)="submitReceipt()"`.
- Dùng `*ngIf="pageConfig.canXxx"` để hiển thị theo quyền.

#### 7.1.5 Ng-content slots

| list-toolbar | toolbar | Cách dùng |
|--------------|---------|-----------|
| `[title]` | `[startTitle]` | `<ng-container startTitle>...</ng-container>` |
| default | default | Custom buttons trong `<app-toolbar>...</app-toolbar>` |
| `[selected]` | `[tollbarSelected]` | `<ion-button tollbarSelected ...>` |

#### 7.1.6 Kiểm tra trước khi migrate

1. Page extend PageBase, có `pageConfig`, `selectedItems`, `query`.
2. Page có các method: `add()`, `refresh()`, `export()`, `import()`, `help()`, `unselect()`, `archiveItems()`, `delete()`, ...
3. Nếu có status-based logic: provider có `showCommandRules`, data-table có `(selectedRowsChange)="showCommandBySelectedRows($event)"`.
4. Tham khảo: ar-invoice, incoming-payment, outgoing-payment (đã dùng toolbar).

### Phase 2: Nhóm đơn giản

1. ~25 trang chỉ dùng CRUD cơ bản.
2. Thay: `app-list-toolbar` + inputs/events → `app-toolbar [page]="this"`.
3. Xóa: `[pageConfig]`, `[selectedItems]`, `[query]`, tất cả `(xxx)="yyy()"`.

### Phase 3: Nhóm có custom content

1. Giữ custom content, đổi selector.
2. Map `[title]` → `[startTitle]`.
3. Map `[selected]` → `[tollbarSelected]`.
4. Domain-specific actions: thêm custom buttons qua ng-content.

### Phase 4: Nhóm đặc biệt (sale-order-mobile, goods-receiving, ...)

1. Các trang có logic phức tạp (ShowPopover, submitReceipt, mergeOrders...).
2. Dùng ng-content cho actions chưa có trong toolbar.
3. Đảm bảo page có đủ methods (submitReceipt, mergeSaleOrders, ...).

### Phase 5: Dọn dẹp

1. Xóa `ListToolbarComponent` khỏi share-components.module.
2. Xóa thư mục `list-toolbar/`.
3. Cập nhật `common.scss`.
4. Cập nhật docs.

### Phase 6: Kiểm tra

1. Build.
2. Test thủ công các list pages quan trọng.
3. Kiểm tra responsive, sub-page, feature panel.

---

## 8. Rủi ro

| Rủi ro | Cách xử lý |
|--------|------------|
| Thiếu domain actions | Dùng ng-content, page tự thêm button |
| ngOnChanges logic phức tạp | Chuyển logic vào page hoặc mở rộng toolbar |
| List page dùng `isMainPageActive`/`backSubPage` | Chuyển sang `isSubActive`/`backToMainView` khi migrate |

---

## 9. Thứ tự ưu tiên

1. Phase 1 – Chuẩn bị.
2. Phase 2 – Nhóm đơn giản (~25 trang).
3. Phase 3 – Nhóm có custom content.
4. Phase 4 – Nhóm đặc biệt.
5. Phase 5 – Dọn dẹp.
6. Phase 6 – Kiểm tra.
