# DetailToolbar (Deprecated)

> **Đã thay thế**: `DetailToolbarComponent` đã được gộp vào `ToolbarComponent`.
> Dùng `app-toolbar` cho detail pages. Xem **[toolbar.md](./toolbar.md)**.

## Migration

```html
<!-- Cũ -->
<app-detail-toolbar [page]="this"></app-detail-toolbar>

<!-- Mới -->
<app-toolbar [page]="this"></app-toolbar>
```

**Mapping**: `[title]` → `[CenterTitle]`, các input khác giữ nguyên.
