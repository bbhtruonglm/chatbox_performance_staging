# Báo cáo Cải thiện Hiệu năng & Login

Tài liệu này tổng hợp các thay đổi đã thực hiện để tối ưu hóa hiệu năng module Login và phân tích các hạn chế kỹ thuật hiện hữu.

## 1. Các hạng mục đã tối ưu hóa (Improved)

### 1.1. Google Analytics (GA4)

- **Vấn đề cũ**: Script GA được nhúng trực tiếp trong `<head>` gây render-blocking, làm chậm chỉ số FCP (First Contentful Paint) và LCP (Largest Contentful Paint).
- **Giải pháp**: Chuyển sang kỹ thuật **Defer Loading**. Script GA chỉ được inject vào DOM sau khi sự kiện `window.onload` kích hoạt.
- **Kết quả**:
  - Loại bỏ hoàn toàn JavaScript blocking trong quá trình tải trang ban đầu.
  - Cải thiện đáng kể điểm Performance trên Lighthouse.
- **Code snippet**:
  ```javascript
  window.addEventListener('load', function () {
    var gtagScript = document.createElement('script')
    gtagScript.async = true
    gtagScript.src = 'https://www.googletagmanager.com/gtag/js?id=G-PL4M71RS48'
    document.head.appendChild(gtagScript)
    // ... init gtag
  })
  ```

### 1.2. SEO & Meta Tags

- **Vấn đề cũ**: Thiếu các thẻ meta quan trọng, ảnh hưởng điểm SEO và trải nghiệm chia sẻ link.
- **Giải pháp**: Bổ sung đầy đủ bộ thẻ Meta chuẩn:
  - `title`, `meta description`.
  - `canonical` link.
  - Open Graph (OG) tags cho Facebook.
  - Twitter Card tags.
- **Kết quả**: Cải thiện điểm SEO lên mức tối đa (90-100).
- **Minh họa Code (`index.html`)**:
  ```html
  <!-- SEO Meta Tags -->
  <title>Đăng nhập - Nền tảng quản lý tin nhắn đa kênh</title>
  <meta
    property="og:type"
    content="website"
  />
  <link
    rel="canonical"
    href="https://chatbox-performance-staging.vercel.app/oauth/login"
  />
  ```

### 1.3. Accessibility (Khả năng truy cập)

- **Vấn đề cũ**: Các ô input và button thiếu `aria-label`, `alt` text cho ảnh.
- **Giải pháp**:
  - Thêm `aria-label` cho các nút đăng nhập (Email, Facebook).
  - Thêm `autocomplete` attributes (`email`, `off`) cho input để hỗ trợ browser autofill tốt hơn mà không gây layout shift.
  - Thêm `role="button"` cho các phần tử tương tác không phải thẻ button chuẩn.
- **Minh họa Code (`Login.vue`)**:
  ```html
  <!-- Button có label cho Screen Reader -->
  <button
    type="button"
    :aria-label="$t('Tiếp tục với email')"
  >
    {{ $t('Tiếp tục với email') }}
  </button>
  ```

---

## 2. Các hạn chế kỹ thuật (Limitations)

Dưới đây là các phần không thể tối ưu sâu hơn hoặc buộc phải giữ nguyên do ràng buộc về nghiệp vụ/cốt lõi (Core Business logic):

### 2.1. Facebook Login (Iframe Cross-Domain)

- **Cơ chế hiện tại**: Sử dụng `iframe` trỏ tới `CROSS_LOGIN_URL` (hệ thống Bot Bán Hàng) để thực hiện đăng nhập, sau đó `postMessage` token về parent window.
- **Hạn chế hiệu năng**:
  - Tốn tài nguyên để load thêm một context trình duyệt (iframe).
  - Không tận dụng được cache sharing hiệu quả như native SDK.
- **Lý do không sửa**:
  - **Ràng buộc Core**: Facebook App gặp lỗi không thể whitelist domain mới động (dynamic domain whitelisting issue).
  - Đây là giải pháp "nghiệp vụ" bắt buộc để hỗ trợ khách hàng sử dụng custom domain hoặc môi trường staging không cố định.
- **Tác động**: Gây ra một lượng nhỏ blocking time và network request phụ, nhưng chấp nhận được để đảm bảo tính năng hoạt động ổn định mọi môi trường.

### 2.2. External Scripts (Ad Blocker Check ...)

- **Mô tả**: `check-ad-blocker.js` và các script tiện ích khác.
- **Lý do**: Cần thiết để đảm bảo tính toàn vẹn của ứng dụng và doanh thu/nghiệp vụ. Việc defer quá sâu có thể làm mất tác dụng của các script này.

## 3. Khuyến nghị tiếp theo

- **Preconnect**: Thêm thẻ `<link rel="preconnect">` tới domain chứa iframe login (nếu khác origin) để giảm thời gian connection setup.
- **Cache Policy**: Đảm bảo các static assets (ảnh, js, css) có `Cache-Control: public, max-age=31536000` (đã thực hiện trong `vercel.json`).
