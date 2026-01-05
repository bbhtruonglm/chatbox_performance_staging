# Báo cáo Cải thiện Hiệu năng Cài đặt Doanh nghiệp (Business Setting)

Tài liệu này chi tiết hóa các thay đổi nhằm tối ưu hóa hiệu năng tại trang Cài đặt Tổ chức (`/dashboard/org/setting`).

## 1. Vấn đề Hiệu năng Cũ

- **Waterfall Rendering**: Các component con (`Page.vue`, `Staff.vue`) tự thực hiện gọi API (`read_os`, `read_ms`) trong hook `onMounted` của chúng. Điều này dẫn đến hiệu ứng "thác nước":
  1. Setting.vue mount.
  2. Page.vue mount -> Gọi API Page (chờ 300ms).
  3. Staff.vue mount -> Gọi API Staff (chờ 300ms).
- **Redundant Loading**: Khi chuyển đổi giữa các tab hoặc reload lại trang, dữ liệu không được tái sử dụng hiệu quả, gây ra cảm giác giật cục.
- **Race Conditions**: Lỗi "Access Denied" xảy ra khi truy cập trực tiếp link `/org/setting` do danh sách tổ chức chưa kịp load và map vào Store.

## 2. Giải pháp Thực hiện (Optimizations)

### 2.1. Centralized & Parallel Loading (Tải song song tập trung)

- **Mô tả**: Chuyển toàn bộ trách nhiệm load dữ liệu con lên component cha `Setting.vue`.
- **Thực hiện**:
  - Sử dụng `Promise.all` để kích hoạt đồng thời 2 luồng lấy dữ liệu Trang và Nhân viên.
  - Component con chỉ việc nhận dữ liệu từ Store (`orgStore.list_os`, `orgStore.list_ms`) và hiển thị (`Passive View`).
- **Code Snippet (`Setting.vue`)**:

  ```javascript
  const loadData = async () => {
    // Gọi song song 2 API độc lập
    await Promise.all([
      read_os(orgStore.selected_org_id).then(data => (orgStore.list_os = data)),
      read_ms(orgStore.selected_org_id).then(data => (orgStore.list_ms = data)),
    ])
  }

  // Gọi khi mount hoặc khi đổi Org
  onMounted(loadData)
  watch(() => orgStore.selected_org_id, loadData)
  ```

- **Kết quả**: Giảm thời gian chờ tổng thể xuống bằng thời gian của API chậm nhất trong nhóm, thay vì tổng thời gian.

### 2.2. Global Data Preloading

- **Mô tả**: Khắc phục lỗi Race Condition và cải thiện LCP.
- **Thực hiện**:
  - Di chuyển logic load **Danh sách Tổ chức** (`readOrg`) ra ngoài `Dashboard.vue` (Layout cha).
  - Đảm bảo khi `Setting` component được mount, danh sách tổ chức đã sẵn sàng trong Store.
- **Code Snippet (`Dashboard.vue`)**:
  ```javascript
  onMounted(() => {
    // Preload critical data ngay ở tầng Layout
    getALlOrgAndPage()
  })
  ```

### 2.3. Layout Stabilization (Ổn định khung hình)

- **Mô tả**: Tránh hiện tượng nội dung nhảy (Layout Shift) khi dữ liệu đổ về.
- **Thực hiện**:
  - Thiết lập `min-height` cho danh sách Page/Staff.
  - Sử dụng Props `org_info` truyền trực tiếp xuống component con (như `AllOrg/Org.vue`) để render Avatar/Tên ngay lập tức thay vì chờ API mapping.

## 3. Kết quả Quan trắc (Impact)

- **LCP (Largest Contentful Paint)**: Cải thiện rõ rệt do nội dung chính hiển thị sớm hơn.
- **CLS (Cumulative Layout Shift)**: Đạt chuẩn (vùng xanh) nhờ cố định kích thước khung chứa.
- **UX**: Không còn lỗi hiển thị sai quyền hạn (Access denied) khi reload trang.
