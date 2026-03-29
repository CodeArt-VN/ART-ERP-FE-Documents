# NG8002 – Đánh giá và kế hoạch sửa

## Nguyên tắc

**Chỉ thêm @Input khi property được dùng trong logic.** Nếu chỉ thêm để build pass mà không dùng → bỏ qua.

---

## Đánh giá từng property

### 1. app-form-control: clearable, virtualScroll, readonly

| Property | Dùng trong component? | Kết luận |
|----------|------------------------|----------|
| clearable | Không – form-control không dùng, chỉ pass [field] xuống input-control | **Có giá trị** – cần forward xuống input-control (đã có @Input, đọc từ field) |
| virtualScroll | Tương tự | **Có giá trị** – forward xuống input-control |
| readonly | Input-control không có @Input readonly; field có thể có | **Cần kiểm tra** – input-control có xử lý readonly/disabled không? |

**Phân tích:** Form-control bọc input-control. Template truyền `[clearable]="true"` riêng, không qua field. Input-control đọc clearable từ `f.clearable` trong field setter. Nếu truyền `[clearable]` trên form-control mà không forward → input-control không nhận. **Thêm @Input và forward có giá trị** – bật clear/virtual scroll cho ng-select.

**readonly:** Không cần – FormControl đã có cơ chế disable form sẵn.

---

### 2. app-so-note: SOIsRemoveItemsWithZeroPriceOnOrderNote

| Dùng trong component? | Kết luận |
|------------------------|----------|
| **Không** – SaleOrderNoteComponent (printing) không dùng property này | **Bỏ qua** |

Component `src/app/components/printing/sale-order-note/` không có logic SOIsRemoveItemsWithZeroPriceOnOrderNote. Logic tương tự nằm ở `src/app/pages/SALE/sale-order-note/` (component khác). **Hành động:** Xóa `[SOIsRemoveItemsWithZeroPriceOnOrderNote]="true"` khỏi template request-detail (binding vô nghĩa).

---

### 3. app-e-chart: reportId

| Dùng trong component? | Kết luận |
|------------------------|----------|
| **Không** – EChartComponent nhận data qua [data], [comparitionData] | **Bỏ qua** |

EChartComponent render chart từ data đã fetch. Report-chart (parent) dùng reportId để fetch, rồi truyền rawData. EChartComponent không cần reportId. Template report-chart hiện không pass [reportId] cho app-e-chart → **bỏ qua**.

---

### 4. app-address: loadingMap

| Dùng trong component? | Kết luận |
|------------------------|----------|
| **Có** – AddressComponent có `@Input() set mapLoading(value)` gán `this.env.isMapLoaded = value` | **Có giá trị** – chỉ cần alias |

Template dùng `[(loadingMap)]` nhưng component có `mapLoading`. Thêm `@Input('loadingMap') set mapLoading(value)` (alias) là đủ. Two-way: setter đã gán env.isMapLoaded; nếu cần emit khi map load thì phải thêm @Output – cần xem logic thực tế.

---

### 5. app-staff-personnel-profile: pageConfig

| Dùng trong component? | Kết luận |
|------------------------|----------|
| **Có** – Template dùng pageConfig.canEdit, pageConfig.showSpinner, pageConfig.canDeleteAccount | **Có giá trị** – thêm @Input |

Component kế thừa PageBase nên có pageConfig, nhưng khi dùng như child nhận [pageConfig] từ parent (staff-detail) thì cần @Input. **Thêm @Input có giá trị.**

---

### 6. div / input: uploader (ng2FileSelect)

| Dùng? | Kết luận |
|-------|----------|
| Directive ng2-file-upload dùng [uploader] trên host | **Bỏ qua** – thuộc directive bên ngoài |

Đây là directive từ package. Sửa phải chỉnh directive hoặc package, không phải component của mình. **Bỏ qua** trừ khi muốn fork/sửa package.

---

## Tổng kết

| Property | Thêm @Input? | Lý do |
|----------|--------------|-------|
| **FormControl: clearable, virtualScroll** | Có | Forward xuống input-control, có tác dụng UX |
| **FormControl: readonly** | Xóa binding | FormControl đã có cơ chế disable form, xóa [readonly] khỏi template |
| **SOIsRemoveItemsWithZeroPriceOnOrderNote** | Xóa binding | Component không dùng, xóa khỏi template |
| **EChart: reportId** | Không | Component không dùng |
| **Address: loadingMap** | Có (alias) | Đã có mapLoading, chỉ cần alias |
| **StaffPersonnelProfile: pageConfig** | Có | Template dùng nhiều |
| **uploader (ng2FileSelect)** | Không | Thuộc directive bên ngoài |

**Nên sửa:** FormControl (clearable, virtualScroll), Address (loadingMap alias), StaffPersonnelProfile (pageConfig).

**Bỏ qua / xóa binding:** SOIsRemoveItemsWithZeroPriceOnOrderNote, readonly (FormControl đã có disable), reportId trên EChart, uploader.
