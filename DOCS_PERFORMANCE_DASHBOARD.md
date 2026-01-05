# Báo cáo Cải thiện Hiệu năng Dashboard

Tài liệu này tổng hợp các thay đổi đã thực hiện để tối ưu hóa hiệu năng khu vực Dashboard (Bảng điều khiển) và các hạn chế tồn đọng.

## 1. Các hạng mục đã tối ưu hóa (Improved)

### 1.1. Chiến lược tải dữ liệu (Data Fetching Strategy)

- **Vấn đề cũ**:
  - Gặp tình trạng **API Waterfall** (gọi API nối tiếp nhau). Component cha load xong mới đến component con load dữ liệu.
  - Dữ liệu Global (Danh sách Tổ chức, Page) được load rải rác ở nhiều nơi (`SelectPage`, `OrgSetting`), gây trùng lặp request hoặc lỗi "Access denied" khi reload trang con.
- **Giải pháp**:
  - **Centralized Preloading**: Di chuyển logic gọi API nền tảng (`getALlOrgAndPage`) lên `onMounted` của `Dashboard.vue`. Dữ liệu được fetch ngay khi khung Dashboard xuất hiện.
  - **Parallel Execution**: Sử dụng `Promise.all` để gọi song song các API độc lập (`readOrg`, `getListActivePage`).
  - **Delegated Loading**: Tại trang Cài đặt (`/org/setting`), component cha `Setting.vue` chịu trách nhiệm fetch toàn bộ dữ liệu con (`Page`, `Staff`) song song, thay vì để từng tab con tự fetch.
- **Kết quả**:
  - Giảm thời gian chờ (TTFB) cho các màn hình con.
  - Loại bỏ các request dư thừa.
  - Đảm bảo tính nhất quán dữ liệu (Data Consistency).
- **Minh họa Code (`Dashboard.vue` & `Setting.vue`)**:

  ```javascript
  // Dashboard.vue: Preload Global Data
  onMounted(() => {
    // Gọi song song 2 API lấy Org và Page ngay lập tức
    getALlOrgAndPage()
  })

  // Setting.vue: Load data song song cho các tab con
  const loadData = async () => {
    await Promise.all([
      read_os(org_id), // Lấy danh sách trang
      read_ms(org_id), // Lấy danh sách nhân viên
    ])
  }
  ```

### 1.2. Layout Shift (CLS) & LCP

- **Vấn đề cũ**:
  - Các danh sách (Page, Staff) khi chưa có dữ liệu sẽ có height = 0, sau khi load xong đẩy nội dung bên dưới xuống (Layout Shift).
  - Ảnh thiếu thuộc tính `width/height` gây nhảy layout khi ảnh load xong.
- **Giải pháp**:
  - Thiết lập `min-height` an toàn (`min-h-[100px]`) cho các container danh sách, giữ chỗ trước khi dữ liệu đổ về.
  - Bổ sung thuộc tính `width` và `height` rõ ràng cho các thẻ `img`.
- **Minh họa Code (`Page.vue`)**:
  ```html
  <!-- Container giữ chiều cao tối thiểu để tránh CLS -->
  <div class="grid ... min-h-[100px]">
    <template v-for="os of orgStore.list_os">...</template>
  </div>
  ```

### 1.3. Accessibility (Trong Dashboard)

- **Vấn đề cũ**: Các ô input tìm kiếm (Search Page, Search Staff) và các nút Toggle thiếu label.
- **Giải pháp**:
  - Bổ sung `aria-label` cho toàn bộ các input search và select box.
  - Thêm `alt` text cho Avatar tổ chức/page.

---

## 2. Các hạn chế kỹ thuật (Limititations & Pending)

### 2.1. Bundle Size (Kích thước gói tin)

- **Hiện trạng**: Cảnh báo từ Vite cho thấy các chunk `index.js` có kích thước > 500kB.
- **Nguyên nhân**:
  - Việc import trọn gói các thư viện nặng (hoặc chưa tree-shake tối đa).
  - Chưa tách nhỏ code (Code Splitting) cho các màn hình ít sử dụng (ví dụ: các Modal cài đặt sâu, các màn hình phụ).
- **Hướng giải quyết (Future)**:
  - Áp dụng `defineAsyncComponent` để lazy load các component Modal nặng.
  - Cấu hình `rollupOptions.output.manualChunks` để chia nhỏ vendor chunk.

### 2.2. Hình ảnh từ External CDN

- **Mô tả**: Một số hình ảnh (Avatar mặc định, Logo partner) được load từ `static.retion.ai`.
- **Vấn đề**: Các tài nguyên này đôi khi thiếu header `Cache-Control` dài hạn, gây cảnh báo của Lighthouse.
- **Giới hạn**: Do tài nguyên nằm trên server CDN khác, không thể cấu hình cache header thông qua `vercel.json` của dự án frontend này.

### 2.3. Hiệu năng CSS

- **Hiện trạng**: File CSS bundle khá lớn.
- **Nguyên nhân**: Sử dụng TailwindCSS nhưng có thể có các style thừa hoặc config chưa tối ưu purge.
