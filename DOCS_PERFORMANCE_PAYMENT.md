# Báo cáo Cải thiện Hiệu năng Payment (Thanh toán)

Tài liệu này tổng hợp các thay đổi đã thực hiện để tối ưu hóa hiệu năng module Payment và phân tích các hạn chế tồn đọng.

## 1. Các hạng mục đã tối ưu hóa (Improved)

### 1.1. Lazy Load Component (Code Splitting)

- **Vấn đề cũ**:
  - Các Modal nặng như `UpgradeModal` (chứa logic thanh toán, bảng giá) và `IncQuota` được import trực tiếp vào trang thông tin gói cước (`PackInfo.vue`).
  - Điều này khiến trình duyệt phải tải và parse toàn bộ JavaScript của các modal này ngay khi người dùng vào trang Payment, dù họ chưa chắc đã click "Nâng cấp".
- **Giải pháp**:
  - Áp dụng kỹ thuật **Async Components** với `defineAsyncComponent`.
  - Modal chỉ được tải về (fetch JS chunk) khi người dùng thực sự cần hiển thị chúng (hoặc khi component cha render). Lưu ý: Với `v-if` bên trong template, chunk sẽ được load khi điều kiện đúng hoặc khi component cha mount (tùy thuộc vào việc nó nằm trong `v-if` hay chỉ là ref). Tuy nhiên, việc tách ra thành file riêng giúp main bundle nhẹ hơn.
- **Kết quả**:
  - Giảm kích thước bundle ban đầu của trang Payment (`Info.vue`).
  - Tăng tốc độ render lần đầu (First Paint).
- **Minh họa Code (`PackInfo.vue`)**:

  ```javascript
  import { defineAsyncComponent } from 'vue'

  // Chuyển từ Static Import sang Async Import
  const UpgradeModal = defineAsyncComponent(() => import('./UpgradeModal.vue'))
  const IncQuota = defineAsyncComponent(() => import('./IncQuota.vue'))
  ```

### 1.2. Accessibility & Layout Stabilization (Kế thừa)

- **Vấn đề cũ**: Các thành phần UI trong Payment (Card, Button) bị lỗi thiếu label hoặc CLS.
- **Giải pháp**: Đã được xử lý chung trong đợt tối ưu Dashboard (áp dụng cho toàn bộ `CardItem`).

---

## 2. Các hạn chế kỹ thuật (Limititations & Pending)

### 2.1. API Wallet Fetching

- **Hiện trạng**: Component `AccountInfo` tự gọi API `read_wallet` khi mounted.
- **Vấn đề**: Khi chuyển đổi qua lại giữa các tab con trong Payment (ví dụ Info -> Recharge -> Info), API này bị gọi lại mỗi lần component unmount/remount.
- **Hướng giải quyết (Future)**: Cache dữ liệu Wallet vào `OrgStore` hoặc sử dụng `keep-alive` cho router view payment.

### 2.2. 3rd Party Payment Scripts

- **Mô tả**: Nếu tích hợp cổng thanh toán (VNPay, Momo SDK...), các script này thường nặng và block main thread.
- **Giải pháp hiện tại**: Đang tải khi cần thiết (Lazy), nhưng vẫn phụ thuộc vào tốc độ phản hồi của bên thứ 3.
