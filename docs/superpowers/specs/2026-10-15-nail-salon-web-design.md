# Website tiệm nail: đặc tả thiết kế (bản Demo)

- **Trạng thái:** BẢN NHÁP, chờ chủ dự án duyệt. **Chưa được code** cho tới khi duyệt xong tài liệu này và kế hoạch triển khai.
- **Ngày soạn:** 2026-10-15
- **Ngôn ngữ giao diện:** tiếng Việt. Giá hiển thị VNĐ, ví dụ `350.000đ`.
- **Dành cho:** người và các AI khác tiếp tục làm dự án. Mỗi mục ghi rõ là **ĐÃ CHỐT** (chủ dự án đã nói), **ĐỀ XUẤT** (mình gợi ý, chưa xác nhận) hoặc **GIẢ ĐỊNH** (cần xác nhận). Phần ĐỀ XUẤT và GIẢ ĐỊNH được tập hợp ở mục 16.
- **Vị trí dự án:** `C:\Thinh\Tiktok\nail`.

---

## 1. Mục tiêu và bối cảnh

Chủ dự án làm **bản demo để chào khách hàng là chủ tiệm nail** (1 tiệm), rồi bán theo 3 gói: Demo, Nâng cấp, Promax. Bản Demo phải:

1. Chạy thật trên web (không phải ảnh tĩnh): chủ tiệm tự cầm điện thoại đặt một lịch và thấy admin nhận được ngay.
2. Đẹp, mượt, ưu tiên điện thoại và tablet, vẫn dùng tốt trên máy tính.
3. Có dữ liệu mẫu nail thuyết phục.

Đây là **website** chạy trên trình duyệt, không phải app cài đặt. Các mẫu điều hướng kiểu app (thanh tab dưới) không dùng.

Hai phần của hệ thống:

- **Khách hàng:** xem dịch vụ và bảng giá, gallery, đặt lịch theo thợ và khung giờ, xem lịch sử, nhận nhắc lịch (mô phỏng), tích điểm.
- **Admin:** quản lý lịch hẹn, thợ, dịch vụ, khách hàng, doanh thu, khuyến mãi.

## 2. Phạm vi theo gói

| | **Demo (bản này)** | **Nâng cấp** | **Promax** |
|---|---|---|---|
| Khách | Trang chủ, gallery, bảng giá VNĐ, đặt lịch (dịch vụ → thợ hoặc bất kỳ → ngày → giờ trống), chống trùng giờ, khách vãng lai và đăng nhập SĐT + PIN, xem và hủy lịch | Đặt nhiều dịch vụ một lượt, đổi lịch, đánh giá thợ | Cọc hoặc thanh toán online (VietQR/MoMo), đặt cho nhóm |
| Điểm và ưu đãi | Tích điểm (tự cộng khi hoàn thành, hiển thị số điểm) | Đổi điểm lấy ưu đãi, hạng thành viên | Giới thiệu bạn bè, voucher sinh nhật |
| Khuyến mãi | Mã giảm giá cơ bản (1 loại phần trăm, có hạn dùng) | Khuyến mãi theo dịch vụ và khung giờ vắng | Chiến dịch tự động |
| Nhắc lịch | Mô phỏng trong web (banner và nút "Gửi nhắc") | Gửi thật qua email | SMS hoặc Zalo ZNS thật |
| Admin | Lịch theo ngày và trạng thái, thợ và giờ làm, dịch vụ, khách hàng, doanh thu và biểu đồ | Xuất Excel, phân quyền nhân viên | Nhiều chi nhánh, kho vật tư, lương và hoa hồng thợ |
| Khác | Dữ liệu mẫu, responsive điện thoại/tablet/máy tính, hiệu ứng mượt | Tên miền riêng, SEO (có thể chuyển sang Next.js) | PWA, thông báo đẩy, đa ngôn ngữ |

**Không làm trong Demo** (để dành gói sau): thanh toán, SMS/Zalo thật, đổi điểm, nhiều dịch vụ một lượt, nhiều chi nhánh, SEO nâng cao.

## 3. Nhật ký quyết định

| # | Quyết định | Mức | Ghi chú |
|---|---|---|---|
| 1 | Web, React, SQL online, 1 tiệm, tiếng Việt, ưu tiên điện thoại và tablet | ĐÃ CHỐT | |
| 2 | Backend và CSDL: **Supabase** (Postgres, Auth, API) | ĐÃ CHỐT | Chủ dự án chọn |
| 3 | Frontend: **Vite + React + TypeScript** (SPA), không dùng Next.js cho Demo | ĐÃ CHỐT | Chuyển Next.js khi bán SEO ở gói Nâng cấp |
| 4 | Khách có hai kiểu: **vãng lai** (không cần tài khoản) và **đăng nhập bằng SĐT + PIN 6 số** | ĐÃ CHỐT | |
| 5 | Chia gói Demo / Nâng cấp / Promax như mục 2 | ĐÃ CHỐT | Chủ dự án xác nhận "đã ổn" |
| 6 | Phong cách: **lai Glossy + Pastel**. Hero tím gradient, phần nội dung nền kem hồng sáng | ĐÃ CHỐT | Admin dùng nền sáng |
| 7 | Hero **3D có dự phòng**: Three.js tải lười, tự rơi về bản 2.5D khi máy yếu | ĐÃ CHỐT | Chi tiết ở mục 9.7 |
| 8 | Mock toàn bộ trang cho máy tính và điện thoại, ghi thành tài liệu trước khi code | ĐÃ CHỐT | Chính là tài liệu này |
| 9 | Điều hướng kiểu web: thanh menu trên, thu thành nút menu trên điện thoại. Không dùng thanh tab dưới | ĐỀ XUẤT | Do bản gốc là web, không phải app |
| 10 | Đặt lịch: máy tính hai cột (các bước và tóm tắt), điện thoại từng bước | ĐỀ XUẤT | Mục 11.4 |
| 11 | Admin: sidebar đầy đủ chữ trên máy tính, thu gọn icon trên tablet, menu nút trên điện thoại | ĐỀ XUẤT | Mục 11.10 |
| 12 | Tra cứu lịch của khách vãng lai bằng **SĐT + mã lịch 6 ký tự** (không chỉ SĐT) | ĐỀ XUẤT | Tránh người lạ xem lịch khi biết SĐT. Brainstorm đầu ghi "tra bằng SĐT", cần xác nhận |
| 13 | Hủy lịch được khi còn ít nhất 2 giờ trước giờ hẹn | GIẢ ĐỊNH | Cấu hình được trong `shop_settings` |
| 14 | Tên thương hiệu mẫu: **Lụa Nails** | GIẢ ĐỊNH | Chỉ là tên minh họa, đổi theo từng khách |

## 4. Công nghệ và thư viện

| Việc | Lựa chọn | Lý do |
|---|---|---|
| Build, dev server | **Vite** + **React 18** + **TypeScript** | Nhanh, ít cấu hình |
| Routing | **React Router** | Có URL cho từng trang, chia code theo route |
| Dữ liệu và cache | **TanStack Query** + **@supabase/supabase-js** | Cache, tải lại, trạng thái chờ và lỗi |
| Style và component | **Tailwind CSS** + **shadcn/ui** (Radix) | Truy cập tốt (dialog, tabs, calendar, bảng), dễ theo thương hiệu |
| Hiệu ứng | **Motion** (framer-motion) | Chuyển trang, hiện khi cuộn, phản hồi chạm |
| Cuộn mượt | **Lenis** (tùy chọn, chỉ trang chủ) | Cảm giác mượt, tắt khi bật giảm chuyển động |
| 3D hero | **three** (+ `@react-three/fiber`, `@react-three/drei` nếu cần) | Chỉ dùng ở hero, tải lười, có dự phòng |
| Gallery | **embla-carousel-react** + lightbox | Vuốt tốt trên điện thoại |
| Biểu đồ | **Recharts** (qua shadcn charts) | Đủ cho doanh thu |
| Form | **react-hook-form** + **zod** | Kiểm tra SĐT, bắt buộc nhập, thông báo lỗi tiếng Việt |
| Ngày giờ | **date-fns** (locale `vi`) + `react-day-picker` | Chọn ngày, tính khung giờ |
| Icon | **lucide-react** | Một bộ icon đồng nhất, không dùng emoji làm icon |
| Thông báo nhỏ | **sonner** | Toast không chiếm focus |
| Test | **Vitest** + Testing Library, **Playwright** (qua `playwright-cli`) | Theo quy tắc dự án |

Giá VNĐ: `n.toLocaleString('vi-VN') + 'đ'` (ví dụ `350.000đ`), lưu số nguyên VNĐ trong CSDL. Không thêm thư viện cho việc này.

## 5. Kiến trúc và cấu trúc thư mục

```
Trình duyệt (Vite + React SPA)
   |-- supabase-js (anon key) --> Supabase
                                   |-- Auth (khách: SĐT+PIN, admin: email+mật khẩu)
                                   |-- Postgres (bảng, ràng buộc, RLS, hàm RPC)
```

Không có server riêng ở bản Demo. Mọi quy tắc quan trọng (không trùng giờ, quyền truy cập) được **ép ở Postgres**, không tin vào giao diện.

```
nail/
  docs/superpowers/specs/        tài liệu này
  supabase/
    migrations/                  SQL: bảng, ràng buộc, RLS, hàm RPC
    seed.sql                     dữ liệu mẫu (mục 13)
  src/
    app/                         router, providers, layout (CustomerLayout, AdminLayout)
    pages/customer/              home, services, gallery, booking, lookup, auth, account, contact
    pages/admin/                 login, dashboard, appointments, services, staff, customers, promos, revenue
    components/ui/               shadcn/ui (button, dialog, tabs...)
    components/                  ServiceCard, SlotGrid, StatusBadge, PriceText, Hero3D, ...
    features/booking/            computeSlots.ts, useAvailability.ts, bookingApi.ts
    features/points/             pointsMath.ts
    features/coupons/            couponMath.ts
    lib/                         supabase.ts, format.ts (VNĐ, ngày), auth.ts
    styles/                      tokens.css (biến thiết kế, mục 9)
  tests/
    unit/                        Vitest
    e2e/                         Playwright
```

Khóa Supabase đặt trong `.env.local` (không đưa vào git, không gửi vào chat). Chỉ dùng **anon key** ở trình duyệt. Không bao giờ dùng service-role key trong code web.

## 6. Mô hình dữ liệu (Postgres trên Supabase)

Tất cả giờ lưu `timestamptz`, hiển thị theo múi giờ `Asia/Ho_Chi_Minh` (UTC+7, không đổi giờ mùa hè). Tiền là `integer` (VNĐ, không số lẻ).

| Bảng | Cột chính | Ghi chú |
|---|---|---|
| `profiles` | `id` (= `auth.users.id`), `role` (`customer` \| `admin`), `full_name`, `phone` | Một dòng cho mỗi tài khoản đăng nhập |
| `customers` | `id`, `profile_id` (null nếu vãng lai), `full_name`, `phone` (duy nhất), `points_balance`, `created_at` | Khách vãng lai tạo theo SĐT; khi đăng ký cùng SĐT thì gắn `profile_id` |
| `staff` | `id`, `name`, `bio`, `avatar_url`, `active`, `sort` | Thợ |
| `service_categories` | `id`, `name`, `slug`, `sort` | Tay, Chân, Gel, Đắp bột, Chăm sóc |
| `services` | `id`, `category_id`, `name`, `description`, `price_vnd`, `duration_min`, `image_url`, `active`, `sort` | |
| `opening_hours` | `weekday` (0-6), `open_time`, `close_time`, `is_closed` | Giờ mở cửa cả tiệm |
| `staff_schedules` | `staff_id`, `weekday`, `start_time`, `end_time`, `is_off` | Giờ làm từng thợ, nằm trong giờ mở cửa |
| `bookings` | `id`, `code` (6 ký tự, duy nhất), `customer_id`, `service_id`, `staff_id`, `start_at`, `end_at`, `status`, `price_vnd`, `discount_vnd`, `total_vnd`, `coupon_id`, `note`, `created_at`, `confirmed_at`, `completed_at`, `cancelled_at`, `cancel_reason` | `price_vnd` là giá chụp lại lúc đặt (đổi giá sau không làm sai doanh thu cũ) |
| `coupons` | `id`, `code` (duy nhất), `percent` (1-100), `min_total_vnd`, `starts_at`, `ends_at`, `max_uses`, `used_count`, `active` | Demo chỉ có giảm theo phần trăm |
| `points_ledger` | `id`, `customer_id`, `booking_id`, `delta`, `reason`, `created_at` | Sổ điểm; `customers.points_balance` là tổng |
| `reminders` | `id`, `booking_id`, `remind_at`, `channel` (`in_app`), `sent_at` | Mô phỏng, không gửi tin thật |
| `gallery_items` | `id`, `title`, `style_tag`, `image_url`, `price_hint_vnd`, `featured`, `active`, `sort` | |
| `shop_settings` | một dòng: `name`, `address`, `phone`, `zalo`, `lat`, `lng`, `slot_minutes` (30), `lead_minutes` (60), `cancel_hours` (2), `points_per_10k` (1) | |

**Trạng thái `bookings.status`:** `pending` (chờ xác nhận), `confirmed` (đã xác nhận), `completed` (hoàn thành), `cancelled` (đã hủy).

Chuyển trạng thái hợp lệ: `pending → confirmed → completed`; `pending/confirmed → cancelled`. Không quay ngược từ `completed` hoặc `cancelled`.

### 6.1. Ràng buộc chống đặt trùng giờ (quan trọng nhất)

```sql
create extension if not exists btree_gist;

alter table bookings add constraint no_double_booking
  exclude using gist (
    staff_id with =,
    tstzrange(start_at, end_at, '[)') with &&
  ) where (status in ('pending','confirmed'));
```

Ràng buộc này ở Postgres nên hai khách bấm cùng lúc thì chỉ một người thành công, người còn lại nhận lỗi và giao diện tải lại giờ trống. `end_at = start_at + duration_min` của dịch vụ. Lịch đã `cancelled` hoặc `completed` không chiếm giờ.

### 6.2. Hàm RPC (Postgres function, `security definer`)

| Hàm | Ai gọi | Việc làm |
|---|---|---|
| `get_busy_intervals(day date, staff uuid default null)` | khách và admin | Trả các khoảng đã bận (`staff_id`, `start_at`, `end_at`), **không lộ thông tin khách**. Giao diện dùng để tính giờ trống |
| `create_booking(service uuid, staff uuid or null, start_at timestamptz, name text, phone text, note text, coupon text)` | khách | Kiểm tra giờ hợp lệ, tìm hoặc tạo `customers` theo SĐT, chọn thợ nếu "bất kỳ", áp mã, chèn `bookings`; trả `code` |
| `lookup_booking(phone text, code text)` | khách vãng lai | Trả đúng lịch khớp cả SĐT và mã |
| `cancel_booking(phone text, code text)` | khách vãng lai | Hủy nếu còn đủ `cancel_hours`, ngược lại báo lỗi rõ |
| `validate_coupon(code text, total int)` | khách | Trả số tiền giảm hoặc lý do không dùng được |
| `complete_booking(id uuid)` | admin | Đặt `completed`, cộng điểm vào `points_ledger` trong **cùng một giao dịch** |

Thợ "bất kỳ": hàm `create_booking` chọn thợ rảnh có **ít lịch nhất trong ngày**, hòa thì theo `sort`. Nếu ràng buộc ở 6.1 báo trùng (hai người cùng lúc), thử thợ rảnh kế tiếp, hết thì báo "Giờ này vừa có người đặt, vui lòng chọn giờ khác".

### 6.3. Row Level Security (RLS)

| Bảng | Khách chưa đăng nhập | Khách đã đăng nhập | Admin |
|---|---|---|---|
| `services`, `service_categories`, `staff`, `opening_hours`, `staff_schedules`, `gallery_items`, `shop_settings` | Chỉ đọc dòng `active` | Như cột bên trái | Đọc và ghi tất cả |
| `bookings`, `customers`, `points_ledger`, `reminders` | Không truy cập trực tiếp (chỉ qua RPC) | Đọc dòng của chính mình | Đọc và ghi tất cả |
| `coupons` | Không truy cập trực tiếp (chỉ qua `validate_coupon`) | Như bên trái | Đọc và ghi tất cả |
| `profiles` | Không | Đọc dòng của mình | Đọc tất cả |

Admin được xác định bằng `profiles.role = 'admin'`, kiểm tra trong policy bằng hàm `is_admin()`. Tài khoản admin do chủ dự án tạo tay trên Supabase, không có trang đăng ký admin.

## 7. Quy tắc nghiệp vụ

### 7.1. Tính giờ trống (`computeSlots`, hàm thuần, có unit test)

Vào: `giờ làm của thợ trong ngày`, `giờ mở cửa`, `danh sách khoảng bận`, `thời lượng dịch vụ (phút)`, `bây giờ`, `slot_minutes` (30), `lead_minutes` (60). Ra: danh sách giờ bắt đầu hợp lệ.

Một giờ bắt đầu `t` hợp lệ khi:
1. `t` nằm trong giờ làm của thợ và trong giờ mở cửa, và `t + thời lượng` không vượt giờ đóng cửa.
2. `[t, t + thời lượng)` không giao với khoảng bận nào của thợ đó.
3. `t >= bây giờ + lead_minutes` (không đặt giờ đã qua hoặc quá sát).
4. `t` nằm trên lưới `slot_minutes` (9:00, 9:30, 10:00...).

Thợ "bất kỳ": giờ hợp lệ nếu **có ít nhất một thợ** hợp lệ. Giao diện hiển thị giờ trống chung, còn thợ cụ thể được chọn khi gửi (6.2).

Giờ không hợp lệ vẫn hiển thị nhưng **mờ, gạch ngang, không bấm được**, kèm chú thích ("Đã có lịch" hoặc "Đã qua"), không ẩn hẳn (để khách thấy tiệm đông).

Ngày tiệm nghỉ (`is_closed`) hiển thị "Tiệm nghỉ ngày này" và vô hiệu hóa ngày trong bộ chọn ngày.

### 7.2. Đặt lịch

Đủ thông tin: dịch vụ, thợ hoặc bất kỳ, ngày, giờ, **họ tên** (bắt buộc), **SĐT** (bắt buộc, định dạng số điện thoại Việt Nam 10 số bắt đầu bằng 0), ghi chú (tùy chọn, tối đa 300 ký tự), mã giảm giá (tùy chọn). Sau khi gửi: trạng thái `pending`, hiển thị trang xác nhận với **mã lịch 6 ký tự**, tóm tắt và nút "Thêm vào lịch" (tải file `.ics`). Admin xác nhận thì thành `confirmed`.

Mã lịch dùng chữ cái in hoa và số, **bỏ các ký tự dễ nhầm** (0, O, 1, I, L), ví dụ `K7H2QM`.

### 7.3. Hủy lịch (khách)

Hủy được khi trạng thái `pending` hoặc `confirmed` và còn ít nhất `cancel_hours` (2 giờ) trước giờ hẹn. Cần hộp xác nhận ("Hủy lịch 10:00, 15/10? Bạn không thể hoàn tác"). Lịch bị hủy nhả giờ cho người khác. Khi quá hạn hủy: nút hủy bị vô hiệu hóa kèm lời nhắn "Đã quá hạn hủy trực tuyến, vui lòng gọi tiệm" và số điện thoại bấm gọi được.

### 7.4. Tích điểm (Demo)

Cộng điểm khi admin bấm **Hoàn thành**: `điểm = floor(total_vnd / 10000) * points_per_10k`. Ví dụ `350.000đ` được 35 điểm. Chỉ khách có `customers` (vãng lai hoặc đã đăng ký) mới có điểm. Khách vãng lai xem điểm khi đăng ký bằng cùng SĐT. Admin có thể chỉnh điểm tay (ghi vào sổ điểm kèm lý do). **Đổi điểm lấy ưu đãi không có trong Demo.**

### 7.5. Khuyến mãi (Demo)

Mã giảm theo phần trăm, có `min_total_vnd`, hạn dùng (`starts_at`, `ends_at`), giới hạn lượt (`max_uses`), bật/tắt. Chỉ một mã mỗi lịch. Khi nhập sai hoặc hết hạn, thông báo ngay dưới ô nhập, nêu lý do cụ thể ("Mã đã hết hạn ngày 30/09").

### 7.6. Nhắc lịch (mô phỏng)

- Khách đã đăng nhập hoặc vừa tra cứu: lịch `confirmed` sắp diễn ra trong 24 giờ hiện **banner nhắc** ở đầu trang tài khoản và trang tra cứu ("Ngày mai 10:00, Gel tay với Mai").
- Admin có nút **"Gửi nhắc (mô phỏng)"** trên lịch sắp tới: ghi `reminders.sent_at` và hiện toast "Đã gửi nhắc (mô phỏng)". Không có tin nhắn thật.

### 7.7. Doanh thu và thống kê

Doanh thu chỉ tính lịch `completed`, theo `total_vnd` (sau giảm giá) và theo ngày `completed_at` theo múi giờ Việt Nam. Lịch hôm nay: đếm theo `start_at` trong ngày, loại `cancelled`.

## 8. Xác thực và phân quyền

| Đối tượng | Cách | Chi tiết |
|---|---|---|
| Khách vãng lai | Không đăng nhập | Đặt lịch qua `create_booking`; tra cứu và hủy bằng SĐT + mã lịch |
| Khách đăng nhập | **SĐT + PIN 6 số** (Supabase Auth) | Supabase Auth cần email hoặc SĐT có xác thực SMS. Để không tốn SMS: dùng **email giả lập từ SĐT**, ví dụ `0901234567@khach.lua-nails.local`, mật khẩu là PIN 6 số (đủ độ dài tối thiểu 6 của Supabase). Giao diện chỉ hiện "SĐT" và "PIN", không lộ email giả. Tắt xác nhận email cho miền giả lập |
| Admin | **Email + mật khẩu** riêng | Tạo tay; `profiles.role = 'admin'` |

- Quên PIN: ở Demo, chủ tiệm đặt lại PIN trong trang admin (nút "Đặt lại PIN" trong chi tiết khách). Gói Nâng cấp đổi sang OTP SMS hoặc Zalo.
- Khi khách đăng ký bằng SĐT đã từng đặt lịch vãng lai, các lịch cũ **tự gắn** vào tài khoản (khớp `customers.phone`).
- Chống dò PIN: giới hạn số lần thử sai theo cài đặt rate limit của Supabase Auth. Thông báo sai chung chung ("SĐT hoặc PIN chưa đúng").
- Hạn chế đã biết: PIN 6 số yếu hơn mật khẩu thường. Chấp nhận cho Demo, ghi rõ để gói sau thay bằng OTP.

## 9. Hệ thống thiết kế

### 9.1. Màu (token trong `src/styles/tokens.css`, dùng biến CSS, không viết mã hex trong component)

| Token | Giá trị | Dùng cho |
|---|---|---|
| `--bg` | `#fff7f5` | Nền trang (kem hồng) |
| `--surface` | `#ffffff` | Thẻ, bảng, ô nhập |
| `--blush` | `#fde6ee` | Nền nhấn nhẹ, mục đang chọn |
| `--line` | `#f0cdd8` | Viền, đường kẻ |
| `--ink` | `#3b2230` | Chữ nội dung (tương phản cao trên nền sáng) |
| `--berry` | `#7a3b55` | Tiêu đề |
| `--rose` | `#c2477a` | Nút chính, giá, liên kết (chữ trắng trên nền này đạt 4,5:1) |
| `--plum-1` / `--plum-2` | `#2a1450` / `#7b2d8e` | Gradient hero |
| `--gel-pink` / `--violet` | `#ff7ad9` / `#8f6bff` | Điểm nhấn gradient của nút chính trên hero, hạt gel |
| `--success` `--warn` `--danger` | `#1f7a3d` `#9a5b00` `#b3261e` | Trạng thái (luôn kèm chữ và icon, không chỉ màu) |

Admin dùng cùng bộ token, nền sáng, bớt trang trí, ưu tiên đọc số liệu. Chế độ tối: **không có trong Demo**.

### 9.2. Chữ

- Tiêu đề: **Plus Jakarta Sans** (đậm 700-800). Nội dung: **Be Vietnam Pro** (400-600). Cả hai hỗ trợ đầy đủ dấu tiếng Việt.
- Cỡ chữ: `12 / 14 / 16 / 18 / 24 / 32 / 44` px. Nội dung tối thiểu **16px trên điện thoại**. Dòng chữ dài tối đa 65-75 ký tự. Chiều cao dòng 1,5 cho nội dung.
- Giá và số liệu dùng số cùng độ rộng (`font-variant-numeric: tabular-nums`) để cột giá thẳng hàng.
- Không viết hoa toàn bộ và giãn chữ cho nhãn; câu chữ viết hoa chữ đầu câu.
- Tải font bằng `font-display: swap`, chỉ tải các độ đậm cần dùng.

### 9.3. Khoảng cách, bo góc, bóng

- Thang khoảng cách 4/8: `4 8 12 16 24 32 48 64`.
- Bo góc phân cấp: ô nhập và nút `12px`, thẻ `16px`, hộp thoại `20px`, chip và nhãn trạng thái `999px`. Không dùng một mức bo cho mọi thứ.
- Bóng: hai mức (`--shadow-sm` cho thẻ, `--shadow-lg` cho hộp thoại và menu), màu bóng pha hồng tím nhạt.

### 9.4. Điểm ngắt responsive (theo **chiều rộng khung chứa**, mobile-first)

| Tên | Rộng | Bố cục |
|---|---|---|
| Điện thoại | dưới 640px | Một cột, menu thu thành nút, bảng đổi thành thẻ |
| Tablet | 640 tới 1023px | Hai cột, admin dùng sidebar thu gọn icon |
| Máy tính | từ 1024px | Nội dung tối đa `1120px` căn giữa, admin sidebar đầy đủ chữ |

Kiểm tra thủ công ở 375, 768, 1024, 1440 và chế độ ngang trên điện thoại. Không cuộn ngang ở bất kỳ cỡ nào.

### 9.5. Bộ component dùng chung

`Button` (chính, phụ, nguy hiểm, chữ), `IconButton` (luôn có `aria-label`), `Input`/`Select`/`Textarea` (nhãn hiển thị phía trên, lỗi ngay dưới, chữ gợi ý bền), `PhoneInput` (bàn phím số, `autocomplete="tel"`), `PinInput` (6 ô, nút hiện/ẩn, cho dán), `Chip`, `StatusBadge` (chữ + icon + màu), `PriceText` (`350.000đ`), `DurationText` (`60 phút`), `ServiceCard`, `ServiceRow`, `StaffAvatar`, `DateStrip`, `SlotGrid`, `Stepper`, `SummaryCard`, `Toast` (sonner), `Dialog`, `Drawer`, `Tabs`, `Accordion`, `DataTable` (chuyển thành thẻ ở điện thoại), `KpiCard`, `BarChart`, `EmptyState`, `Skeleton`, `Hero3D`, `Gallery` (lưới + lightbox), `Carousel`.

Mỗi component có đủ trạng thái: mặc định, hover, focus, nhấn, vô hiệu, đang tải, lỗi.

### 9.6. Trợ năng và tương tác (bắt buộc)

- Tương phản chữ thường tối thiểu **4,5:1**; thành phần giao diện 3:1. Không truyền đạt ý nghĩa chỉ bằng màu.
- **Vòng focus** nhìn thấy rõ (2px, màu đối lập) trên mọi phần tử bấm được. Thứ tự Tab đúng thứ tự nhìn. Có liên kết "Bỏ qua tới nội dung chính".
- Vùng chạm tối thiểu **44×44px** trên điện thoại, cách nhau tối thiểu 8px. Thanh dính (menu, thanh nút đặt lịch) không che phần tử đang focus.
- Nhãn hiển thị cho mọi ô nhập (không chỉ placeholder). Lỗi đặt cạnh ô, nối bằng `aria-describedby`. Sau khi gửi lỗi, focus nhảy tới ô lỗi đầu tiên.
- Biểu đồ luôn có **bảng số liệu thay thế** và mô tả văn bản.
- Hộp thoại: bẫy focus, đóng bằng phím Esc và nút đóng rõ ràng.
- Carousel: có nút tạm dừng nếu tự chạy; mặc định không tự chạy.
- Tôn trọng `prefers-reduced-motion`: tắt 3D, parallax, hiệu ứng chuyển động lớn.

### 9.7. Chuyển động và hero 3D

- Mỗi màn hình tối đa 1-2 chuyển động chính. Chuyển động phải có ý nghĩa (phản hồi hành động, thể hiện thay đổi), không chỉ trang trí. Hiện dần khi cuộn chỉ dùng ở trang chủ.
- Thời gian dùng bộ giá trị chung: `120ms` phản hồi nhấn, `200-300ms` mở/đóng, `400ms` chuyển trang. Thoát nhanh hơn vào (khoảng 60-70%). Chỉ animate `transform` và `opacity`.
- **Hero 3D có dự phòng:** canvas Three.js (hạt gel bóng, viên đá đính, nghiêng theo con trỏ hoặc chạm) **tải lười sau khi trang đã hiện**. Hiển thị bản **2.5D** (hạt gel bằng CSS, nổi nhẹ) khi: `prefers-reduced-motion`, `navigator.connection.saveData`, không có WebGL, thiết bị yếu (heuristic: `hardwareConcurrency <= 2` hoặc `deviceMemory <= 2`), hoặc đo FPS dưới 30 trong khoảng 1 giây đầu thì tự chuyển. Bản 2.5D là mặc định hiển thị ngay, 3D thay thế khi sẵn sàng.
- Chỉ dùng 3D ở hero trang chủ.

### 9.8. Hình ảnh

Định dạng WebP/AVIF, khai báo kích thước hoặc `aspect-ratio` (tránh nhảy bố cục), `loading="lazy"` cho ảnh dưới màn hình đầu, `srcset` cho nhiều cỡ. Demo dùng ảnh mẫu bản quyền thoải mái hoặc ảnh chủ tiệm cung cấp, ảnh trong mockup chỉ là ô màu.

## 10. Sơ đồ trang và URL

**Khách hàng**

| Trang | URL | Cần đăng nhập |
|---|---|---|
| Trang chủ | `/` | Không |
| Dịch vụ và bảng giá | `/dich-vu` | Không |
| Gallery | `/mau-nail` | Không |
| Đặt lịch | `/dat-lich` (nhận `?dich-vu=<id>&mau=<id>` để điền sẵn) | Không |
| Xác nhận đặt lịch | `/dat-lich/thanh-cong?ma=<code>` | Không |
| Tra cứu và hủy lịch | `/tra-cuu` | Không |
| Đăng nhập, đăng ký | `/dang-nhap` | Không |
| Tài khoản (lịch sử, điểm, nhắc) | `/tai-khoan` | Có |
| Liên hệ | `/lien-he` | Không |

**Admin** (tiền tố `/quan-tri`, chuyển hướng về đăng nhập nếu chưa có quyền)

| Trang | URL |
|---|---|
| Đăng nhập admin | `/quan-tri/dang-nhap` |
| Tổng quan | `/quan-tri` |
| Lịch hẹn | `/quan-tri/lich-hen` |
| Dịch vụ | `/quan-tri/dich-vu` |
| Thợ và giờ mở cửa | `/quan-tri/tho` |
| Khách hàng | `/quan-tri/khach-hang` |
| Khuyến mãi | `/quan-tri/khuyen-mai` |
| Doanh thu | `/quan-tri/doanh-thu` |

Mọi trang có URL riêng (chia sẻ được, nút Quay lại của trình duyệt hoạt động đúng, khôi phục vị trí cuộn và bộ lọc). Trang không tồn tại hiển thị trang 404 có nút về trang chủ.

## 11. Đặc tả từng trang

Mỗi trang gồm: mục đích, bố cục máy tính, bố cục điện thoại, trạng thái (tải, trống, lỗi), tương tác chính, và khung **Quyết định thiết kế** (ưu, nhược, đề xuất). Bố cục tablet nằm giữa hai bản (ghi chú khi khác).

Khung chung cho khách hàng: **thanh trên cố định** (logo, menu: Trang chủ, Dịch vụ, Mẫu nail, Đặt lịch, Liên hệ; phải: Tra cứu lịch, Đăng nhập hoặc tên khách, nút chính **Đặt lịch**), chân trang (địa chỉ, giờ mở cửa, SĐT, liên kết). Trên điện thoại: logo, nút **Đặt lịch** nhỏ và nút menu ☰ mở ngăn bên có đầy đủ liên kết, mỗi liên kết cao tối thiểu 48px. Trên điện thoại có thể thêm **thanh nút "Đặt lịch" dính đáy** ở trang chủ và dịch vụ (không phải thanh tab).

### 11.1. Trang chủ `/`

**Mục đích:** gây ấn tượng, trả lời nhanh "tiệm ở đâu, mở lúc nào, giá thế nào", dẫn khách tới đặt lịch.

**Máy tính (từ trên xuống):**
1. **Hero** nền tím gradient, bên trái tiêu đề "Móng đẹp, đặt lịch trong 30 giây", mô tả ngắn, nút chính **Chọn dịch vụ**, nút phụ **Xem mẫu nail**; bên phải hạt gel 3D (hoặc 2.5D). Phía dưới hero cong nhẹ chuyển sang nền kem.
2. **Dải thông tin:** giờ mở cửa (9:00 đến 20:00 hàng ngày), địa chỉ, SĐT; mỗi mục có icon, SĐT bấm gọi được.
3. **Dịch vụ nổi bật:** 4 thẻ (ảnh, tên, `350.000đ`, `60 phút`, nút Đặt) và liên kết "Xem toàn bộ bảng giá".
4. **Mẫu nail nổi bật:** carousel vuốt ngang với nút "Đặt mẫu này".
5. **Đội ngũ thợ:** 3 thẻ (ảnh, tên, giới thiệu một câu).
6. **Khuyến mãi và tích điểm:** một dải ("Khách mới giảm 15% với mã `LUA15`", "Mỗi 10.000đ được 1 điểm").
7. **Chân trang.**

**Điện thoại:** hero một cột (chữ trên, hạt gel nhỏ dưới hoặc nền), nút chính toàn chiều rộng; dải thông tin xếp dọc; dịch vụ nổi bật thành danh sách cuộn ngang hoặc 1 cột; carousel vuốt tự nhiên; thanh nút "Đặt lịch" dính đáy.

**Trạng thái:** khi tải xem skeleton cho thẻ; lỗi tải hiện thông báo và nút "Thử lại"; nếu thiếu dữ liệu một khối thì ẩn khối đó, không hiện khung trống.

**Quyết định thiết kế:**
- Ưu: ấn tượng đầu mạnh nhờ hero glossy; thông tin quyết định (giờ, địa chỉ, giá) lộ ra ngay; một nút chính duy nhất trên mỗi vùng.
- Nhược: hero 3D nặng, cần dự phòng; nhiều khối nên trang dài.
- Đề xuất: giữ 7 khối nhưng cho phép ẩn từng khối theo cấu hình; hero 3D đặt sau cổng dự phòng ở 9.7; điểm tải quan trọng (LCP) là tiêu đề, không phải canvas.

### 11.2. Dịch vụ và bảng giá `/dich-vu`

**Mục đích:** xem giá và thời lượng theo nhóm, chọn dịch vụ để đặt.

**Máy tính:** tiêu đề trang; **tab nhóm** (Tất cả, Tay, Chân, Gel, Đắp bột, Chăm sóc) có số lượng; danh sách dịch vụ **hai cột**, mỗi dòng: tên, mô tả một dòng, chip `60 phút`, giá `350.000đ` (đậm, màu `--rose`), nút **Chọn** dẫn tới `/dat-lich?dich-vu=<id>`.

**Điện thoại:** tab nhóm cuộn ngang (có chỉ báo); danh sách **một cột**, nút Chọn đủ 44px; thanh nút "Đặt lịch" dính đáy.

**Trạng thái:** nhóm trống hiện "Nhóm này chưa có dịch vụ" kèm nút xem nhóm khác; tải xem skeleton.

**Quyết định thiết kế:**
- Ưu: thấy giá, thời lượng và nút chọn cùng lúc; nối thẳng sang đặt lịch.
- Nhược: 5 nhóm trên màn nhỏ phải cuộn ngang; muốn so giá giữa nhóm phải đổi tab (có tab "Tất cả" để bù).
- Đề xuất: dùng tab + dòng dịch vụ; dịch vụ nổi bật ở trang chủ dùng thẻ có ảnh; giá luôn dạng `350.000đ`.

### 11.3. Mẫu nail (gallery) `/mau-nail`

**Mục đích:** gợi cảm hứng, đặt theo mẫu.

**Máy tính:** hàng **chip lọc** theo kiểu (Tất cả, Ombre, French, Đính đá, 3D, Mắt mèo); lưới so le (masonry) **4 cột**; mỗi ô có nhãn kiểu và giá gợi ý; bấm mở **lightbox** (ảnh lớn, mũi tên chuyển, Esc đóng, nút **Đặt mẫu này** dẫn `/dat-lich?mau=<id>`).

**Điện thoại:** masonry **2 cột**; chip lọc cuộn ngang; lightbox toàn màn hình, vuốt để chuyển ảnh, có nút đóng rõ ràng.

**Trạng thái:** không có kết quả cho bộ lọc hiện "Chưa có mẫu kiểu này" và nút xóa bộ lọc; ảnh tải lười với khung giữ chỗ để không nhảy bố cục.

**Quyết định thiết kế:**
- Ưu: lọc giúp khách tìm đúng kiểu; masonry đẹp, nhìn được nhiều mẫu.
- Nhược: cần gắn nhãn kiểu cho từng ảnh (thêm việc nhập liệu); nhiều ảnh phải tối ưu tải.
- Đề xuất: lọc + masonry cho trang đầy đủ; trang chủ chỉ dùng carousel nổi bật.

### 11.4. Đặt lịch `/dat-lich`

**Mục đích:** hoàn tất đặt lịch trong vài thao tác, không bao giờ cho đặt trùng giờ.

**Các bước:** 1 Dịch vụ, 2 Thợ (hoặc "Bất kỳ"), 3 Ngày, 4 Giờ, 5 Thông tin của bạn.

**Máy tính:** **hai cột**.
- Cột trái: thanh tiến trình 5 bước; mỗi bước là một khối gập mở. Bước đã xong thu gọn thành một dòng tóm tắt có nút **Sửa**. Bước hiện tại mở sẵn. Bước **Thợ**: thẻ chọn (ảnh, tên, ô "Bất kỳ" mặc định, ghi "Gợi ý: nhanh nhất"). Bước **Ngày**: dải 14 ngày (T2 15, T3 16...), ngày tiệm nghỉ bị vô hiệu. Bước **Giờ**: lưới giờ (`9:00`, `9:30`...), giờ không hợp lệ mờ và gạch ngang. Bước **Thông tin**: Họ tên, SĐT, Ghi chú, mã giảm giá (hộp nhỏ "Có mã giảm giá?").
- Cột phải (**dính khi cuộn**): thẻ **Tóm tắt**: dịch vụ, thợ, ngày giờ bắt đầu và kết thúc dự kiến, giá, giảm giá, **Tổng**, dòng "Bạn sẽ nhận 35 điểm sau khi hoàn thành", nút chính **Xác nhận đặt lịch** (chỉ bật khi đủ thông tin hợp lệ).

**Điện thoại:** **từng bước một màn hình**, thanh tiến trình trên cùng ("Bước 4/5"), nút **Quay lại** và **Tiếp tục** ở **thanh dính đáy** kèm dòng tổng ("350.000đ · 60 phút"). Bước cuối hiện tóm tắt và nút **Xác nhận đặt lịch**.

**Trạng thái và xử lý lỗi:**
- Khi đổi dịch vụ hoặc thợ ở bước sau, các giờ đã chọn được **kiểm tra lại**; nếu không còn hợp lệ thì bỏ chọn và nhắc nhẹ.
- Lúc gửi mà giờ vừa bị người khác đặt: thông báo "Giờ này vừa có người đặt. Chọn giờ khác nhé", giữ nguyên thông tin đã nhập, tải lại giờ trống và đưa người dùng về bước Giờ.
- Nút gửi vô hiệu và hiện vòng quay khi đang gửi (chống bấm hai lần).
- Mất mạng: giữ dữ liệu đã nhập, báo "Chưa gửi được, thử lại".
- Sau khi gửi thành công: chuyển sang `/dat-lich/thanh-cong`, hiện mã lịch, tóm tắt, nút "Thêm vào lịch (.ics)", "Tra cứu lịch" và "Đăng ký để tích điểm" (nếu chưa đăng nhập).

**Quyết định thiết kế:**
- Ưu: hai cột trên máy tính cho thấy toàn cảnh và tổng tiền; từng bước trên điện thoại ít rối, tỉ lệ hoàn tất cao; một logic bước dùng chung hai bố cục.
- Nhược: nhiều trạng thái cần xử lý (đổi dịch vụ, giờ vừa bị đặt); bước Thợ có thể dư nếu khách chọn "Bất kỳ".
- Đề xuất: giữ năm bước nhưng cho "Bất kỳ" làm mặc định để khách bấm nhanh; Demo chỉ chọn **một** dịch vụ (đặt nhiều dịch vụ ở gói Nâng cấp); thanh tổng tiền luôn thấy ở mọi cỡ màn hình.

### 11.5. Tra cứu và hủy lịch `/tra-cuu`

**Mục đích:** khách vãng lai xem và hủy lịch đã đặt.

**Máy tính và điện thoại (cùng cấu trúc, một cột căn giữa, rộng tối đa 560px):**
1. Hộp tra cứu: ô **SĐT** và ô **Mã lịch**, nút **Tra cứu**. Dòng gợi ý "Mã lịch có trong màn hình xác nhận khi bạn đặt".
2. Kết quả: thẻ lịch hẹn (ngày giờ, dịch vụ, thợ, tổng tiền, **StatusBadge**), banner nhắc nếu lịch trong 24 giờ, nút **Hủy lịch** (kiểu nguy hiểm, phụ).
3. **Hộp thoại xác nhận hủy**: tiêu đề "Hủy lịch hẹn lúc 10:00, 15/10?", nút **Giữ lịch** (mặc định, nổi bật) và **Hủy lịch**; hủy xong hiện toast "Đã hủy lịch".
4. Nếu đã quá hạn hủy: nút vô hiệu và lời nhắn kèm số điện thoại gọi được.

**Trạng thái:** không tìm thấy hiện "Không tìm thấy lịch. Kiểm tra lại SĐT và mã lịch" kèm cách khắc phục; đang tra cứu hiện skeleton.

**Quyết định thiết kế:**
- Ưu: ngắn gọn, đủ cho khách không có tài khoản; hộp thoại xác nhận tránh hủy nhầm.
- Nhược: khách quên mã lịch thì không tra được (phải gọi tiệm hoặc đăng ký tài khoản).
- Đề xuất: yêu cầu cả SĐT và mã (an toàn hơn chỉ SĐT); thêm liên kết "Quên mã? Gọi tiệm" và gợi ý đăng ký tài khoản để xem mọi lịch.

### 11.6. Đăng nhập và đăng ký `/dang-nhap`

**Mục đích:** khách đăng nhập bằng SĐT + PIN để xem lịch sử và điểm.

**Máy tính:** hai khối cạnh nhau: trái là bảng thương hiệu gradient (lợi ích: "Xem lịch sử, tích điểm mỗi lần làm"), phải là biểu mẫu có **hai tab** Đăng nhập / Đăng ký. **Điện thoại:** chỉ biểu mẫu, bảng thương hiệu thu thành một dòng.

**Đăng nhập:** SĐT, PIN 6 số (nút hiện/ẩn, cho dán), nút **Đăng nhập**, liên kết "Quên PIN? Liên hệ tiệm". **Đăng ký:** Họ tên, SĐT, tạo PIN 6 số, nhập lại PIN, nút **Tạo tài khoản**. Dưới biểu mẫu: "Không cần tài khoản? Đặt lịch ngay" dẫn `/dat-lich`.

**Trạng thái:** sai SĐT hoặc PIN báo chung "SĐT hoặc PIN chưa đúng" và giữ SĐT đã nhập; SĐT đã đăng ký báo "SĐT này đã có tài khoản, hãy đăng nhập"; đăng ký xong tự đăng nhập và các lịch cũ cùng SĐT được gắn vào.

**Quyết định thiết kế:**
- Ưu: không tốn SMS, dùng SĐT quen thuộc với khách Việt; cho phép dán và trình quản lý mật khẩu.
- Nhược: PIN 6 số yếu hơn mật khẩu; quên PIN phải nhờ tiệm đặt lại.
- Đề xuất: chấp nhận cho Demo, thông báo rõ ở mục 16; gói Nâng cấp thay bằng OTP.

### 11.7. Tài khoản `/tai-khoan`

**Mục đích:** xem lịch sắp tới, lịch sử, điểm.

**Máy tính:** hàng đầu là lời chào ("Chào Lan") và **thẻ điểm** (gradient, số điểm lớn, "Còn 60 điểm nữa tới mốc 200 điểm", thanh tiến trình, ghi chú "Đổi điểm có ở gói sau" nếu muốn hiển thị). Hai cột: trái **Lịch sắp tới** (kèm banner nhắc, nút Hủy), phải **Lịch sử** (bảng: ngày, dịch vụ, thợ, giá, trạng thái, điểm `+35`). Nút "Đặt lịch mới" và "Đăng xuất". **Điện thoại:** một cột xếp theo thứ tự thẻ điểm, lịch sắp tới, lịch sử dạng thẻ.

**Trạng thái:** chưa có lịch hiện "Chưa có lịch hẹn. Đặt lịch đầu tiên của bạn" và nút Đặt lịch; chưa đăng nhập thì chuyển về `/dang-nhap` và quay lại sau khi đăng nhập.

**Quyết định thiết kế:**
- Ưu: điểm thưởng nổi bật tạo động lực quay lại; một nơi duy nhất cho mọi lịch của khách.
- Nhược: ở Demo điểm chưa đổi được nên có thể gây hụt hẫng.
- Đề xuất: hiển thị mốc điểm và thanh tiến trình để thấy giá trị, ghi chú rõ phần đổi điểm thuộc gói Nâng cấp khi trình diễn.

### 11.8. Liên hệ `/lien-he`

**Máy tính:** hai cột: trái thông tin (địa chỉ, SĐT bấm gọi, Zalo, **bảng giờ mở cửa theo thứ trong tuần** và đánh dấu hôm nay), phải **bản đồ** (nhúng Google Maps hoặc ảnh tĩnh kèm nút **Chỉ đường** mở ứng dụng bản đồ). **Điện thoại:** một cột, thông tin trước, bản đồ sau; nút gọi và nút chỉ đường to, dễ bấm.

**Quyết định thiết kế:**
- Ưu: trả lời nhanh nhu cầu liên hệ; bảng giờ làm rõ.
- Nhược: nhúng bản đồ làm tải nặng.
- Đề xuất: Demo dùng ảnh bản đồ tĩnh có nút "Chỉ đường" mở Google Maps, không nhúng iframe.

### 11.9. Đăng nhập admin `/quan-tri/dang-nhap`

Thẻ căn giữa: logo, tiêu đề "Đăng nhập quản trị", **Email**, **Mật khẩu** (nút hiện/ẩn), nút **Đăng nhập**. Sai thông tin báo "Email hoặc mật khẩu chưa đúng". Không có liên kết đăng ký. Tài khoản không phải admin bị từ chối và đưa về trang chủ. Điện thoại: thẻ toàn chiều rộng với lề 16px.

### 11.10. Khung admin

**Máy tính:** **sidebar trái đầy đủ chữ** (rộng 220px): Tổng quan, Lịch hẹn (kèm số lịch chờ xác nhận), Dịch vụ, Thợ và giờ, Khách hàng, Khuyến mãi, Doanh thu, Đăng xuất (tách riêng ở cuối). Mục đang mở được tô nền `--blush` và đậm chữ. Thanh trên: tên trang, ngày hôm nay, ảnh đại diện.
**Tablet:** sidebar **thu gọn thành cột icon** (64px) có chú thích khi rê hoặc focus, bấm biểu tượng menu để mở rộng.
**Điện thoại:** sidebar ẩn, thanh trên có nút ☰ mở ngăn bên; bảng đổi thành thẻ; nút hành động (Xác nhận, Hoàn thành) **to, đủ 44px** vì chủ tiệm hay dùng điện thoại giữa ca làm.

Mọi trang admin: trạng thái tải (skeleton), lỗi (có nút Thử lại), trống (có lời mời hành động).

### 11.11. Tổng quan `/quan-tri`

**Máy tính:** hàng **4 thẻ KPI**: Lịch hôm nay, Chờ xác nhận, Doanh thu hôm nay, Doanh thu tuần (kèm so sánh với tuần trước bằng chữ và icon, không chỉ màu). Dưới: **biểu đồ cột doanh thu 7 ngày** (có bảng số liệu thay thế mở được), bên cạnh **danh sách lịch sắp tới** hôm nay (giờ, tên khách, dịch vụ, thợ, nút nhanh) và **Top dịch vụ**.
**Điện thoại:** KPI thành lưới 2 cột; biểu đồ rộng toàn chiều; danh sách lịch sắp tới dạng thẻ.

**Quyết định thiết kế:**
- Ưu: chủ tiệm nhìn là hiểu tình hình; thao tác nhanh ngay từ danh sách sắp tới.
- Nhược: dễ nhồi nhiều số liệu.
- Đề xuất: giữ 4 KPI cố định, một biểu đồ, hai danh sách; chi tiết đẩy sang trang Doanh thu.

### 11.12. Lịch hẹn `/quan-tri/lich-hen`

**Mục đích:** việc hằng ngày của chủ tiệm.

**Máy tính:** thanh công cụ: bộ chọn ngày (`‹` `T3 15/10/2026` `›` và nút **Hôm nay**), **chip trạng thái kèm số** (Chờ xác nhận 3, Đã xác nhận 8, Hoàn thành 4, Đã hủy 1), ô tìm tên hoặc SĐT. **Bảng**: Giờ, Khách (tên, SĐT), Dịch vụ, Thợ, Giá, Trạng thái, Hành động. Hành động theo trạng thái: `pending` → **Xác nhận** và **Hủy**; `confirmed` → **Hoàn thành**, **Hủy** và **Gửi nhắc (mô phỏng)**; `completed` và `cancelled` chỉ xem. Có nút **Thêm lịch** (admin đặt hộ khách, dùng cùng logic chống trùng giờ).
**Điện thoại:** bảng đổi thành **thẻ** (giờ lớn, tên khách, dịch vụ và thợ, chip trạng thái, hai nút hành động đầy chiều ngang); chip trạng thái cuộn ngang.

**Hành động nguy hiểm:** Hủy và Hoàn thành có hộp thoại xác nhận; Hủy hỏi lý do (danh sách chọn nhanh và ô ghi chú). Sau khi thao tác hiện toast có **Hoàn tác** trong vài giây khi hợp lý (Hoàn thành thì không cho hoàn tác vì đã cộng điểm; hộp thoại xác nhận là chốt chặn).

**Trạng thái:** ngày không có lịch hiện "Chưa có lịch trong ngày này"; đổi ngày giữ bộ lọc.

**Quyết định thiết kế:**
- Ưu: đủ bộ lọc cho việc hằng ngày; hành động hiện đúng theo trạng thái, giảm bấm nhầm.
- Nhược: bảng nhiều cột trên tablet có thể chật.
- Đề xuất: trên tablet ẩn cột Giá, hiện khi mở rộng dòng; nút hành động chính luôn thấy; bộ chọn ngày mặc định là hôm nay.

### 11.13. Dịch vụ `/quan-tri/dich-vu`

**Máy tính:** nút **Thêm dịch vụ**, bộ lọc nhóm, bảng: Tên, Nhóm, Giá (`350.000đ`), Thời lượng (`60 phút`), Hiển thị (công tắc), Sửa, Xóa. Bấm Sửa hoặc Thêm mở **ngăn bên phải (drawer)** với biểu mẫu: Tên, Nhóm, Giá (VNĐ, chỉ nhập số, tự định dạng có dấu chấm), Thời lượng (phút, bội của 30 hoặc tuỳ chọn), Mô tả, Ảnh, công tắc Hiển thị. **Điện thoại:** danh sách thẻ, drawer thành trang toàn màn hình.
**Ràng buộc:** giá và thời lượng phải là số nguyên dương. **Xóa** chỉ khi chưa có lịch dùng dịch vụ; nếu đã có thì đề nghị **ẩn** thay vì xóa ("Dịch vụ đã có lịch hẹn, hãy ẩn thay vì xóa").

### 11.14. Thợ và giờ mở cửa `/quan-tri/tho`

**Máy tính:** hai khối. **Thợ:** thẻ mỗi thợ (ảnh, tên, trạng thái làm việc) và nút Thêm thợ; bấm một thợ mở **lịch làm theo thứ** (T2 đến CN: giờ bắt đầu, giờ kết thúc, hoặc công tắc Nghỉ). **Giờ mở cửa tiệm:** bảng 7 dòng (thứ, mở, đóng, công tắc Nghỉ). Quy tắc: giờ làm của thợ phải nằm trong giờ mở cửa, hệ thống báo lỗi cạnh ô nếu vi phạm. **Điện thoại:** hai khối xếp dọc, mỗi dòng giờ cao tối thiểu 44px.
**Lưu ý:** đổi giờ làm **không** tự hủy lịch đã đặt; nếu thay đổi ảnh hưởng lịch đã có thì hiện cảnh báo liệt kê các lịch bị ảnh hưởng để chủ tiệm tự xử lý.

### 11.15. Khách hàng `/quan-tri/khach-hang`

**Máy tính:** ô tìm tên hoặc SĐT; bảng: Tên, SĐT, Số lần đến, Điểm, Lần cuối, Ghi chú; bấm một dòng mở **ngăn chi tiết** (lịch sử lịch hẹn, sổ điểm, ghi chú nội bộ, nút **Chỉnh điểm** có lý do bắt buộc, nút **Đặt lại PIN** nếu khách có tài khoản). **Điện thoại:** danh sách thẻ, chi tiết toàn màn hình. Dữ liệu cá nhân chỉ admin thấy; không hiện cho thợ.

### 11.16. Khuyến mãi `/quan-tri/khuyen-mai`

**Máy tính:** nút **Tạo mã**; danh sách thẻ mã (ví dụ `LUA15`, giảm `15%`, đơn tối thiểu `200.000đ`, hạn `30/11/2026`, đã dùng `12/50`, công tắc Bật). Biểu mẫu tạo/sửa trong drawer: mã (chữ hoa và số), phần trăm (1-100), đơn tối thiểu, ngày bắt đầu và kết thúc, giới hạn lượt, bật/tắt. **Điện thoại:** thẻ một cột. Kiểm tra: mã không trùng, ngày kết thúc sau ngày bắt đầu.

### 11.17. Doanh thu `/quan-tri/doanh-thu`

**Máy tính:** bộ chuyển kỳ **Hôm nay / Tuần này / Tháng này**; **tổng doanh thu** lớn (`28.400.000đ`) kèm số lịch hoàn thành; **biểu đồ cột** theo ngày (có bảng số liệu thay thế); **doanh thu theo dịch vụ** (thanh ngang, có nhãn giá trị); **doanh thu theo thợ** (bảng). Nút **Xuất Excel** hiển thị nhưng ghi nhãn "Gói Nâng cấp" và vô hiệu ở Demo. **Điện thoại:** biểu đồ đơn giản hóa (ít mốc trục), bảng thành thẻ.
**Quy tắc số liệu:** chỉ tính lịch `completed`, theo `total_vnd` (7.7). Số liệu khi chưa có dữ liệu hiển thị "Chưa có doanh thu trong kỳ này".

## 12. Chi tiết nội dung (văn bản giao diện)

- Tên trạng thái cố định: **Chờ xác nhận**, **Đã xác nhận**, **Hoàn thành**, **Đã hủy**. Một hành động giữ cùng tên trong toàn luồng (nút "Xác nhận" cho toast "Đã xác nhận").
- Nút nói rõ việc sẽ xảy ra: "Xác nhận đặt lịch", "Hủy lịch", "Lưu thay đổi" (không dùng "Submit", "OK").
- Thông báo lỗi nêu nguyên nhân và cách sửa, không xin lỗi chung chung: "SĐT chưa đúng. Nhập 10 số, bắt đầu bằng 0."
- Trạng thái trống là lời mời hành động: "Chưa có lịch hẹn. Đặt lịch đầu tiên của bạn."
- Ngày giờ dạng Việt: `T3, 15/10/2026, 10:00`, thời lượng `60 phút`, giá `350.000đ`.

## 13. Dữ liệu mẫu (`supabase/seed.sql`)

- **Tiệm:** Lụa Nails, địa chỉ mẫu, giờ mở cửa 9:00 đến 20:00 mỗi ngày (cấu hình sửa được), `slot_minutes` 30.
- **Thợ:** Mai, Linh, Trang (mỗi thợ một lịch làm khác nhau để thể hiện logic).
- **Nhóm và dịch vụ mẫu (giá VNĐ, thời lượng phút):** Tay: Sơn thường `120.000đ`/30, Cắt da và dưỡng `150.000đ`/30; Gel: Gel tay `350.000đ`/60, Gel chân `380.000đ`/60; Đắp bột: Đắp bột `450.000đ`/90; Chăm sóc: Chăm sóc tay `200.000đ`/45, Chăm sóc chân `220.000đ`/45. (Chủ dự án có thể thay theo bảng giá thật của khách.)
- **Gallery:** 12 mục, 5 kiểu (Ombre, French, Đính đá, 3D, Mắt mèo), có mục `featured`.
- **Khách mẫu và lịch mẫu:** vài khách, vài lịch ở đủ bốn trạng thái, lịch rải trong 7 ngày để biểu đồ doanh thu có dữ liệu đẹp.
- **Mã giảm giá mẫu:** `LUA15` (giảm 15%, đơn tối thiểu `200.000đ`, còn hạn).
- **Tài khoản admin mẫu:** tạo thủ công trên Supabase, **không** đưa mật khẩu vào seed hoặc tài liệu.

## 14. Kế hoạch kiểm thử

Theo quy tắc dự án: **viết test trước** cho mỗi tính năng, kiểm chứng bằng `playwright-cli` trên cả luồng khách và luồng admin, chạy `/code-review` trước khi báo xong.

**Unit test (Vitest) cho logic thuần:**
- `computeSlots`: giờ trống đúng theo giờ làm, thời lượng, khoảng bận, `lead_minutes`, giờ đóng cửa, ngày nghỉ, thợ "bất kỳ", ranh giới (đúng giờ đóng cửa, hai lịch liền kề không chồng).
- `formatVnd`, định dạng ngày giờ Việt, `pointsMath` (35 điểm cho `350.000đ`, làm tròn xuống), `couponMath` (giảm %, đơn tối thiểu, hết hạn).
- Kiểm tra dữ liệu `zod`: SĐT Việt Nam hợp lệ và không hợp lệ.

**Kiểm tra CSDL (SQL hoặc test tích hợp trên Supabase thử nghiệm):**
- Ràng buộc `no_double_booking`: chèn hai lịch chồng giờ cùng thợ phải lỗi; lịch `cancelled` không chiếm giờ; hai lịch kề nhau (kết thúc đúng lúc bắt đầu) được phép.
- RLS: khách không đọc được lịch của người khác; không gọi được hàm admin; chỉ đọc dòng `active`.
- `complete_booking` cộng điểm đúng và chỉ một lần.

**E2E (Playwright qua `playwright-cli`), chạy ở 375, 768, 1280:**
1. Khách vãng lai: chọn dịch vụ → thợ → ngày → giờ → nhập thông tin → nhận mã → tra cứu bằng SĐT + mã → hủy.
2. **Chống trùng giờ:** hai ngữ cảnh trình duyệt cùng chọn một giờ và thợ, chỉ một người thành công, người còn lại nhận thông báo và được đưa về chọn giờ khác.
3. Đăng ký, đăng nhập SĐT + PIN, thấy lịch cũ gắn vào, thấy điểm.
4. Admin: đăng nhập, thấy lịch mới ở "Chờ xác nhận", **Xác nhận**, **Hoàn thành**, kiểm tra điểm khách và doanh thu hôm nay tăng.
5. Admin: thêm, sửa, ẩn dịch vụ; đổi giờ làm thợ; tạo mã giảm giá; khách dùng mã khi đặt.
6. Kiểm tra trợ năng cơ bản (phím Tab, focus, nhãn) và `prefers-reduced-motion`.

## 15. Quy trình làm việc cho AI khác

1. **Chưa code** cho tới khi chủ dự án duyệt tài liệu này **và** kế hoạch triển khai (`writing-plans`).
2. Làm theo thứ tự: `writing-plans` → `executing-plans` hoặc `subagent-driven-development` → `test-driven-development` → `verification-before-completion` → `/code-review`.
3. Mọi màn hình UI: thiết kế theo `ui-ux-pro-max` + `frontend-design` trước khi code, tuân theo mục 9.
4. Sau mỗi thay đổi lớn về cấu trúc: chạy lại `/graphify` để cập nhật sơ đồ code.
5. Ưu tiên thay đổi nhỏ nhất vẫn đủ; ghi nợ kỹ thuật bằng comment `shortcut:` (xem `/ponytail-debt`).
6. Không đưa khóa Supabase, mật khẩu, PIN thật vào mã nguồn hay tài liệu. Chỉ dùng biến môi trường.
7. Các mockup tham khảo đã dựng trong phiên brainstorm nằm ở `.superpowers/brainstorm/` (HTML): `style-directions.html` (3 hướng phong cách), `hero-3d-compare.html` (hero 3D thật và 2.5D), `components-round2.html` (mẫu component bản điện thoại, **một phần đã lỗi thời** vì kiểu app, thay bằng mục 11 của tài liệu này).

## 16. Giả định, đề xuất cần chủ dự án xác nhận

1. **Tên thương hiệu** "Lụa Nails" và logo: dùng tạm hay có tên thật? Khi bán cho khách khác thì đổi tên và màu theo thương hiệu của họ.
2. **Tra cứu của khách vãng lai** bằng SĐT + mã lịch (thay vì chỉ SĐT như brainstorm đầu): đồng ý chưa?
3. **Chính sách hủy:** hủy trực tuyến được khi còn ít nhất 2 giờ. Có đổi không?
4. **Giờ đặt tối thiểu trước:** 60 phút (không đặt sát giờ). Có đổi không?
5. **Điểm:** 1 điểm cho mỗi 10.000đ, đổi điểm để gói sau: đúng ý chưa?
6. **Mã giảm giá mẫu** `LUA15` giảm 15%: giữ hay đổi?
7. **Bảng giá và dịch vụ mẫu** ở mục 13 là số minh họa. Có bảng giá thật của khách đầu tiên không?
8. **Ảnh nail:** dùng ảnh mẫu miễn phí (cần nguồn ảnh) hay chủ tiệm cung cấp ảnh thật?
9. **Tài khoản Supabase:** chủ dự án tạo project, đưa khóa vào `.env.local` (không chat). Cần chọn vùng máy chủ gần Việt Nam (ví dụ Singapore).
10. **Hạn chế đã biết:** PIN 6 số yếu hơn mật khẩu; email giả lập từ SĐT là giải pháp tạm cho Demo; hero 3D chưa đo FPS trên điện thoại tầm trung (đo khi làm); khung Next.js/SEO để dành cho gói Nâng cấp.
11. **Chưa quyết:** có cần kiểm thử CSDL bằng Supabase chạy cục bộ (cần Docker) hay dùng project Supabase thử nghiệm riêng? Mặc định đề xuất project thử nghiệm riêng để khỏi cài Docker.
