# JavaScript Library Functions - ART-ERP-FE

## 📋 **Mục đích tài liệu**

Tài liệu này mô tả các utility functions có sẵn trong `src/app/services/static/global-functions.ts`, giúp developer sử dụng hiệu quả các hàm tiện ích đã được xây dựng sẵn.

---

## 📦 **Import và Sử dụng**

```typescript
import { lib } from 'src/app/services/static/global-functions';

// Sử dụng
const formattedDate = lib.dateFormat(new Date(), 'dd/mm/yyyy');
```

---

## ⏰ **Time & Date Functions**

### **formatTimeConfig(time: TimeConfig, isPrevious = false)**
Format hiển thị cấu hình thời gian theo dạng human-readable.

```typescript
// Ví dụ sử dụng
const timeConfig = { Type: 'Relative', Amount: 2, Period: 'Day', IsPastDate: true };
const result = lib.formatTimeConfig(timeConfig); // "2 days ago"
const previous = lib.formatTimeConfig(timeConfig, true); // "Previous 2 days"
```

### **calcTimeValue(timeConfig: TimeConfig, isFullfillDate = false, baseDate = null)**
Tính toán giá trị Date từ TimeConfig.

```typescript
// Ví dụ sử dụng
const timeConfig = { Type: 'Relative', Amount: 7, Period: 'Day', IsPastDate: false };
const futureDate = lib.calcTimeValue(timeConfig); // Date 7 ngày sau
const endOfDay = lib.calcTimeValue(timeConfig, true); // 23:59:59 của ngày đó
```

### **dateFormat(date, term = 'yyyy-mm-dd', failReturn = '')**
Format ngày tháng theo nhiều định dạng khác nhau.

```typescript
// Các format hỗ trợ
lib.dateFormat(new Date(), 'dd/mm/yyyy');      // "07/12/2024"
lib.dateFormat(new Date(), 'yyyy-mm-dd');      // "2024-12-07"
lib.dateFormat(new Date(), 'dd/mm/yy hh:MM');  // "07/12/24 14:30"
lib.dateFormat(new Date(), 'hh:MM');           // "14:30"
lib.dateFormat(new Date(), 'weekday');         // "Sat"
lib.dateFormat(new Date(), 'yyMMdd');          // "241207"
```

### **dateFormatFriendly(date)**
Format ngày tháng theo dạng thân thiện (relative time).

```typescript
lib.dateFormatFriendly(new Date(Date.now() - 3600000)); // "1 giờ trước"
lib.dateFormatFriendly(new Date(Date.now() - 86400000)); // "1 ngày trước"
```

### **getWeekDates(date)**
Lấy tất cả ngày trong tuần của ngày được chỉ định.

```typescript
const weekDates = lib.getWeekDates(new Date()); // Array 7 ngày từ Chủ nhật đến Thứ 7
```

### **getMonthWeekDates(date)**
Lấy tất cả ngày trong tháng theo dạng lịch (6 tuần x 7 ngày).

```typescript
const monthDates = lib.getMonthWeekDates(new Date()); // Array 42 ngày
```

### **getStartEndDates(start, end)**
Tạo array các ngày từ start đến end.

```typescript
const dateRange = lib.getStartEndDates('2024-12-01', '2024-12-07');
// [{ Date: 2024-12-01 }, { Date: 2024-12-02 }, ...]
```

---

## 💰 **Currency & Number Functions**

### **formatMoney(amount, decimalCount = 2, decimal = '.', thousands = ',')**
Format số tiền với phân cách hàng nghìn.

```typescript
lib.formatMoney(1234567.89);           // "1,234,567.89"
lib.formatMoney(1234567, 0, '', '.');  // "1.234.567"
lib.formatMoney(1234567, 0);           // "1,234,567"
```

### **currencyFormat(currency, contryCode = 'vi-VN', currencyCode = 'vnd')**
Format tiền tệ theo chuẩn locale.

```typescript
lib.currencyFormat(1234567, 'vi-VN', 'vnd'); // "1.234.567 ₫"
lib.currencyFormat(1234567, 'en-US', 'usd'); // "$1,234,567.00"
```

### **currencyFormatFriendly(amount)**
Format số tiền dạng rút gọn (K, M, B).

```typescript
lib.currencyFormatFriendly(1500);      // "1.5K"
lib.currencyFormatFriendly(1500000);   // "1.5M"
lib.currencyFormatFriendly(1500000000); // "1.5B"
```

### **DocTienBangChu(SoTien)**
Đọc số tiền bằng chữ (tiếng Việt).

```typescript
lib.DocTienBangChu(1234567); // "Một triệu hai trăm ba mươi tư nghìn năm trăm sáu mươi bảy đồng"
```

### **isNumeric(value)**
Kiểm tra giá trị có phải số không.

```typescript
lib.isNumeric('123');    // true
lib.isNumeric('abc');    // false
lib.isNumeric(123);      // true
```

### **paddingNumber(number, count)**
Thêm số 0 vào đầu để đủ độ dài.

```typescript
lib.paddingNumber(5, 3);   // "005"
lib.paddingNumber(123, 5); // "00123"
```

---

## 🔧 **Object & Array Utilities**

### **deepAssign(target, ...sources)**
Deep merge objects (tương tự Object.assign nhưng deep).

```typescript
const target = { a: { b: 1 } };
const source = { a: { c: 2 } };
lib.deepAssign(target, source); // { a: { b: 1, c: 2 } }
```

### **copyPropertiesValue(fromItem, toItem)**
Copy tất cả properties từ object này sang object khác.

```typescript
const from = { name: 'John', age: 30 };
const to = {};
lib.copyPropertiesValue(from, to); // to = { name: 'John', age: 30 }
```

### **cloneObject(source)**
Clone object (shallow clone qua JSON).

```typescript
const original = { name: 'John', data: { age: 30 } };
const cloned = lib.cloneObject(original);
```

### **getObject(path, obj)**
Lấy giá trị từ object theo đường dẫn string.

```typescript
const obj = { user: { profile: { name: 'John' } } };
const name = lib.getObject('user.profile.name', obj); // "John"
```

### **getAttrib(Term, Lst, GetAttrib = 'Name', defaultValue = '', FindAttrib = 'Id')**
Tìm và lấy attribute từ array objects.

```typescript
const users = [
    { Id: 1, Name: 'John', Age: 30 },
    { Id: 2, Name: 'Jane', Age: 25 }
];
const name = lib.getAttrib(1, users); // "John"
const age = lib.getAttrib(2, users, 'Age'); // 25
```

### **sumInArray(arr, property)**
Tính tổng một property trong array.

```typescript
const items = [{ amount: 100 }, { amount: 200 }, { amount: 300 }];
const total = lib.sumInArray(items, 'amount'); // 600
```

---

## 🌳 **Tree Data Structure Functions**

### **listToTree(list, childrenFieldName = 'children', assign = null)**
Chuyển flat list thành tree structure.

```typescript
const flatList = [
    { Id: 1, Name: 'Root', IDParent: null },
    { Id: 2, Name: 'Child 1', IDParent: 1 },
    { Id: 3, Name: 'Child 2', IDParent: 1 }
];
const tree = lib.listToTree(flatList);
// [{ Id: 1, Name: 'Root', children: [{ Id: 2, ... }, { Id: 3, ... }] }]
```

### **treeToList(tree, childrenFieldName = 'children', parentFieldName = 'IDParent', idFieldName = 'Id')**
Chuyển tree structure thành flat list.

```typescript
const tree = [{ Id: 1, children: [{ Id: 2 }, { Id: 3 }] }];
const flatList = lib.treeToList(tree);
// [{ Id: 1, IDParent: null }, { Id: 2, IDParent: 1 }, { Id: 3, IDParent: 1 }]
```

### **searchTree(ls, term, allParent = true, allChildren = true)**
Tìm kiếm trong tree data.

```typescript
const treeData = [
    { Id: 1, Code: 'A001', Name: 'Category A' },
    { Id: 2, Code: 'B001', Name: 'Category B' }
];
const results = lib.searchTree(treeData, 'category a'); // Tìm theo Code hoặc Name
```

### **markNestedNode(ls, Id, flagProperty = 'flag', revert = false)**
Đánh dấu tất cả node con của một node.

```typescript
const treeData = [/* tree data */];
lib.markNestedNode(treeData, 1, 'selected', false); // Đánh dấu selected = true cho tất cả con của node Id = 1
```

---

## 🔤 **String Processing Functions**

### **generateUID(prefix = '', suffix = '', length = null, isUpperCase = true, isBreak = false, breakPartLength = 4, breakChar = '-', radix = 36)**
Tạo unique ID với nhiều tùy chọn.

```typescript
lib.generateUID();                           // "K8J2L9M4N"
lib.generateUID('USER', '', 16, true);       // "USERK8J2L9M4N123"
lib.generateUID('', '', 16, true, true, 4);  // "K8J2-L9M4-N123-P567"
```

### **generateCode(radix = 36)**
Tạo code ngắn dựa trên timestamp.

```typescript
lib.generateCode(); // "k8j2l9m4"
```

### **personNameFormat(name)**
Format tên người theo chuẩn (viết hoa chữ cái đầu, loại bỏ ký tự đặc biệt).

```typescript
lib.personNameFormat('nguyễn văn a'); // "Nguyễn Văn A"
```

### **URLFormat(str)**
Chuyển string thành dạng URL-friendly (loại bỏ dấu, ký tự đặc biệt).

```typescript
lib.URLFormat('Nguyễn Văn A - Giám đốc'); // "nguyen-van-a-giam-doc"
```

### **rempveSpecialCharacter(str)**
Loại bỏ dấu tiếng Việt nhưng giữ nguyên ký tự đặc biệt khác.

```typescript
lib.rempveSpecialCharacter('Nguyễn Văn A'); // "Nguyen Van A"
```

---

## 🎨 **UI & Display Functions**

### **hexToRgba(hex, alpha)**
Chuyển màu hex sang rgba.

```typescript
lib.hexToRgba('#FF5733', 0.5); // "rgba(255, 87, 51, 0.5)"
```

### **colorLightenDarken(color, percent)**
Làm sáng hoặc tối màu.

```typescript
lib.colorLightenDarken('#FF5733', 20);  // Sáng hơn 20%
lib.colorLightenDarken('#FF5733', -20); // Tối hơn 20%
```

### **getCssVariableValue(variableName)**
Lấy giá trị CSS variable.

```typescript
const primaryColor = lib.getCssVariableValue('--ion-color-primary');
```

### **fileSizeFormat(bytes, si = true)**
Format kích thước file.

```typescript
lib.fileSizeFormat(1024);      // "1.0 kB"
lib.fileSizeFormat(1048576);   // "1.0 MB"
lib.fileSizeFormat(0);         // "--"
```

---

## 💳 **Banking & QR Code Functions**

### **genBankTransferQRCode(bankCode, bankAccount, amount, message)**
Tạo QR code chuyển tiền theo chuẩn TCCS TCVN 03:2018.

```typescript
const qrCode = lib.genBankTransferQRCode('vcb', '1234567890', 100000, 'Thanh toan hoa don');
// Trả về string để tạo QR code
```

### **readVietQRCode(code)**
Đọc thông tin từ VietQR code.

```typescript
const info = lib.readVietQRCode(qrCodeString);
// { bankCode: 'vcb', bankAccount: '1234567890', amount: 100000, message: 'Thanh toan' }
```

### **genVietQRDeeplink(openApp, bankCode, bankAccount, amount, message)**
Tạo deeplink mở app ngân hàng.

```typescript
const deeplink = lib.genVietQRDeeplink('bidv', 'vcb', '1234567890', 100000, 'Thanh toan');
// "https://dl.vietqr.io/pay?ba=1234567890@vcb&am=100000&tn=Thanh toan&app=bidv"
```

---

## 🧮 **Advanced Tree Operations**

### **buildFlatTree(items, treeState, isAllRowOpened = true, root = null)**
Xây dựng flat tree với state management (show/hide nodes).

```typescript
const items = [/* hierarchical data */];
const treeState = [/* saved expand/collapse state */];
const flatTree = await lib.buildFlatTree(items, treeState, true);
```

### **sumTreeListByParentId(treeList, sumFields = [], idFieldName = 'Id', parentIdFieldName = 'IDParent')**
Tính tổng các field trong tree theo hierarchy.

```typescript
const treeData = [
    { Id: 1, Amount: 100, IDParent: null },
    { Id: 2, Amount: 50, IDParent: 1 },
    { Id: 3, Amount: 30, IDParent: 1 }
];
const result = lib.sumTreeListByParentId(treeData, ['Amount']);
// Node Id=1 sẽ có Amount = 180 (100 + 50 + 30)
```

---

## 📍 **Utility Functions**

### **calcDistance2Points(lat1, lon1, lat2, lon2)**
Tính khoảng cách giữa 2 điểm GPS.

```typescript
const distance = lib.calcDistance2Points(21.0285, 105.8542, 21.0245, 105.8412);
// Khoảng cách tính bằng mét
```

### **calcCRC(input)**
Tính CRC checksum cho string.

```typescript
const crc = lib.calcCRC('Hello World'); // Trả về số CRC
```

---

## 🎨 **Predefined Colors**

Library cung cấp sẵn array màu để sử dụng:

```typescript
// Sử dụng màu có sẵn
const randomColor = lib.Colors[Math.floor(Math.random() * lib.Colors.length)];

// Hoặc lấy màu theo index
const primaryColor = lib.Colors[0]; // "#FF5733"
```

---

## 💡 **Best Practices**

### **1. Import đúng cách**
```typescript
// ✅ Đúng
import { lib } from 'src/app/services/static/global-functions';

// ❌ Sai
import * as lib from 'src/app/services/static/global-functions';
```

### **2. Sử dụng type-safe**
```typescript
// ✅ Đúng - Kiểm tra input
if (lib.isNumeric(value)) {
    const formatted = lib.formatMoney(parseFloat(value));
}

// ❌ Sai - Không kiểm tra
const formatted = lib.formatMoney(value);
```

### **3. Handle errors**
```typescript
// ✅ Đúng - Handle null/undefined
const formatted = lib.dateFormat(date, 'dd/mm/yyyy', 'N/A');

// ❌ Sai - Không handle
const formatted = lib.dateFormat(date, 'dd/mm/yyyy');
```

### **4. Sử dụng constants**
```typescript
// ✅ Đúng - Sử dụng constants cho format
const DATE_FORMAT = 'dd/mm/yyyy';
const formatted = lib.dateFormat(date, DATE_FORMAT);

// ❌ Sai - Hard-code format
const formatted = lib.dateFormat(date, 'dd/mm/yyyy');
```

---

## 🚨 **Lưu ý quan trọng**

### **Performance**
- `cloneObject()` sử dụng JSON.parse/stringify - không dùng cho objects có functions
- Tree operations có thể chậm với data lớn (>1000 nodes)
- `buildFlatTree()` là async function - cần await

### **Data Types**
- Hầu hết functions không validate input - cần kiểm tra trước khi sử dụng
- Date functions expect Date object hoặc valid date string
- Tree functions expect objects có Id và IDParent fields

### **Browser Compatibility**
- Một số functions sử dụng modern JS features
- QR code functions chỉ hỗ trợ một số ngân hàng Việt Nam
- CSS variable functions cần browser hỗ trợ CSS custom properties

---

**Tài liệu này sẽ được cập nhật khi có thêm functions mới hoặc thay đổi existing functions.**

**Version**: 1.0.0  
**Last Updated**: December 2024  
**Maintained by**: Development Team
