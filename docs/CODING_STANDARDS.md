# Quy chuẩn code (một bộ duy nhất cho cả dự án)

Đọc file này trước khi viết bất kỳ dòng code nào. Mục tiêu: **đơn giản, dễ hiểu, dễ bảo trì**. Với mỗi việc chỉ có **một cách làm**; nếu thấy cần làm khác thì sửa quy chuẩn này trước, rồi mới làm khác.

Tài liệu liên quan: [spec thiết kế](superpowers/specs/2026-10-15-nail-salon-web-design.md), các kế hoạch trong `docs/superpowers/plans/`.

## 1. Nguyên tắc

1. **Config trước, code sau.** Giá trị dùng từ hai nơi trở lên (nhãn trạng thái, đường dẫn, menu, chữ giao diện dùng chung, bước đặt lịch, kiểu gallery) nằm trong `src/config/` và được import. Không gõ lại chuỗi hoặc số ở nhiều chỗ.
2. **Một nguồn sự thật.** Giờ trống, giá, điểm, quyền, chống trùng giờ do **CSDL** quyết định. TypeScript chỉ hiển thị và dự đoán. Nếu cùng một quy tắc phải có ở hai nơi (ví dụ chuyển trạng thái lịch), ghi chú comment ở cả hai nơi chỉ về nhau.
3. **Nhỏ và thẳng.** Hàm làm một việc. Tệp dưới khoảng 150 dòng. Không trừu tượng hóa khi chưa có lần lặp thứ ba. Không thêm tùy chọn mà chưa ai cần.
4. **Tên trường giữ nguyên như CSDL** (`snake_case`: `start_at`, `price_vnd`). Không có lớp chuyển đổi sang camelCase. Biến và hàm của riêng TypeScript dùng camelCase.
5. **Không đoán.** Dữ liệu vào từ người dùng và từ mạng được kiểm tra (zod) trước khi dùng.

## 2. Cấu trúc thư mục và chiều phụ thuộc

```
src/
  config/        giá trị dùng chung (mục 3)               không import từ thư mục khác ngoài lucide-react
  lib/           hàm thuần và kết nối (format, vnTime, supabase, api/)
  features/<tên>/  logic theo nghiệp vụ: queries.ts, schemas.ts, hàm thuần, hook
  components/    giao diện dùng lại: ui/ (nguyên tử), rồi component ghép
  pages/         ghép feature + component thành một trang, không chứa logic nghiệp vụ
  app/           router, provider, layout
  test/          setup chung cho test
```

Chiều phụ thuộc chỉ **đi xuống**: `pages → features → components → lib → config`. Không import ngược. `config` và `lib` không biết gì về React page.

## 3. Config (`src/config/`)

| Tệp | Chứa |
|---|---|
| `site.ts` | Tên thương hiệu, câu khẩu hiệu, múi giờ, tên miền email giả lập, nhãn thứ trong tuần |
| `routes.ts` | `ROUTES` (mọi đường dẫn) và hàm dựng URL (`bookingUrl`) |
| `navigation.ts` | Danh sách menu khách và menu admin, dùng chung cho thanh trên, ngăn ☰ và sidebar |
| `statuses.ts` | Trạng thái lịch: nhãn, tông màu, icon, các bước chuyển hợp lệ |
| `booking.ts` | Các bước đặt lịch, số ngày đặt trước, độ dài ghi chú |
| `gallery.ts` | Các kiểu nail (nhãn, màu ô giữ chỗ) |
| `copy.ts` | Chữ giao diện dùng chung: nút, lỗi, kiểm tra dữ liệu, trạng thái trống |

Quy tắc dùng config:
- Thêm một mục mới (một trạng thái, một mục menu, một kiểu gallery) = sửa **một** chỗ trong config, giao diện tự cập nhật.
- Chữ chỉ dùng ở **một** trang thì viết thẳng trong trang. Chữ dùng từ hai nơi trở lên thì đưa vào `copy.ts`.
- Cài đặt do chủ tiệm chỉnh (giờ mở cửa, địa chỉ, SĐT, `slot_minutes`, `lead_minutes`, `cancel_hours`, `points_per_10k`) nằm ở **bảng CSDL** (`shop_settings`, `opening_hours`), đọc qua `useShopSettings()`. Config chỉ chứa thứ lập trình viên đổi.
- Hex màu **chỉ** nằm ở `src/styles/tokens.css` (và `config/gallery.ts` cho ô giữ chỗ). Component dùng token Tailwind (`bg-surface`, `text-ink`, `text-rose`).

## 4. Đặt tên

| Loại | Quy tắc | Ví dụ |
|---|---|---|
| Thư mục | `kebab-case` hoặc một từ | `features/booking`, `pages/customer` |
| Component React | tệp `PascalCase.tsx`, export có tên | `SlotGrid.tsx` → `export function SlotGrid` |
| Hook | `useXxx.ts`, một hook mỗi tệp khi lớn | `useBookingDraft.ts` |
| Hàm thuần, tiện ích | tệp `camelCase.ts` | `slots.ts`, `format.ts` |
| Kiểu, interface | `PascalCase`, **không** tiền tố `I` | `type Service = ...` |
| Hằng cấu hình | `UPPER_SNAKE` cho đối tượng config | `BOOKING_STATUS`, `ROUTES` |
| Test | cạnh tệp: `slots.test.ts`; tên test nêu hành vi | `it('bỏ giờ chồng lên lịch đã có')` |
| Commit | Conventional Commits | `feat(booking): thêm lưới giờ trống` |

**Không dùng export mặc định** (`export default`), trừ tệp mà công cụ bắt buộc (`vite.config.ts`).

## 5. Dữ liệu

- Chỉ **một** nơi gọi Supabase: `src/lib/api/*.ts`. Mỗi hàm trả dữ liệu đã có kiểu và ném `AppError` khi lỗi.
- Component và page **không** gọi `supabase` trực tiếp. Chúng dùng hook TanStack Query khai báo ở `features/<tên>/queries.ts`.
- Khóa query tập trung ở `lib/queryKeys.ts`.
- Kiểu dữ liệu bảng ở `src/types/db.ts`, tên trường giống hệt CSDL.
- Tiền là **số nguyên VNĐ**; hiển thị bằng `formatVnd`. Giờ là chuỗi ISO; hiển thị qua `lib/vnTime.ts` và `lib/format.ts`, không tự `new Date().toLocaleString()` rải rác.

## 6. Lỗi và trạng thái tải

- CSDL ném mã lỗi ngắn (`slot_taken`, `coupon_invalid`...). `lib/errors.ts` chuyển mã đó thành câu tiếng Việt lấy từ `COPY.errors`. Component **chỉ hiển thị** câu đó.
- Mọi truy vấn hiển thị qua `<QueryState>`: tự xử lý đang tải (skeleton), lỗi (nút Thử lại), trống (lời mời hành động). Không tự viết lại ba trạng thái này ở từng trang.
- Không nuốt lỗi bằng `catch {}` rỗng.

## 7. Form

- `react-hook-form` + `zod`. Schema đặt trong `features/<tên>/schemas.ts`. Thông báo lỗi lấy từ `COPY.validation`.
- Mọi ô nhập dùng `<Field>` (nhãn hiển thị, lỗi bên dưới, gợi ý). Không dùng placeholder thay nhãn.
- Trang admin thêm/sửa dữ liệu dùng bộ điều khiển bằng config (`ResourcePage`), không viết bảng và form riêng cho từng loại.

## 8. Style

- Tailwind, chỉ dùng token trong `tokens.css`. Biến thể component dùng `cva`. Ghép class bằng `cn()` (`lib/cn.ts`).
- Responsive mobile-first. Điểm ngắt: dưới 640px (điện thoại), từ 640 (tablet), từ 1024 (máy tính). Không cuộn ngang.
- Trợ năng bắt buộc: nhãn cho ô nhập, `aria-label` cho nút chỉ có icon, vòng focus nhìn thấy, vùng chạm tối thiểu 44px trên điện thoại, không truyền tin chỉ bằng màu.
- Icon chỉ dùng `lucide-react`. Không dùng emoji làm icon.

## 9. Test (viết trước)

1. Mỗi tính năng: **viết test hỏng trước**, rồi mới viết code cho test qua (TDD).
2. Hàm thuần: Vitest (`*.test.ts` cạnh tệp). Chạy với múi giờ máy khác Việt Nam để chứng minh không phụ thuộc máy.
3. CSDL: kịch bản SQL trong `supabase/tests/`, chạy bằng `npm run db:test` trên project Supabase **thử nghiệm**.
4. Luồng người dùng: Playwright (qua `playwright-cli`) cho cả luồng khách và luồng admin, ở rộng 375, 768, 1280.
5. Trước khi báo xong: `npm run check` (kiểm tra kiểu, lint, test) phải xanh, rồi chạy `/code-review`.

## 10. Comment và nợ kỹ thuật

- Chỉ comment **vì sao**, không comment cái code đã nói rõ.
- Đường tắt có giới hạn ghi theo mẫu một dòng: `// shortcut: <giới hạn>, <khi nào nâng cấp>`. Gom bằng `/ponytail-debt`.
- Không để `TODO` không có người nhận. Việc chưa làm thì ghi vào kế hoạch.

## 11. Công cụ

- Định dạng: Prettier (`singleQuote`, `semi`, `printWidth: 100`, `trailingComma: all`). Lint: ESLint (typescript-eslint, react-hooks, jsx-a11y). Chạy `npm run format`, `npm run lint`.
- Lệnh chuẩn: `npm run dev`, `npm run check`, `npm run test`, `npm run db:migrate`, `npm run db:seed`, `npm run db:test`.
- Khóa bí mật chỉ trong `.env.local` (không commit). Biến `VITE_*` lộ ra trình duyệt, **không đặt khóa bí mật** (như `DATABASE_URL`) với tiền tố `VITE_`. Chỉ dùng anon key ở frontend.
- Sau thay đổi lớn về cấu trúc: chạy `/graphify` để cập nhật sơ đồ code.

## 12. Danh sách kiểm tra khi tạo một trang mới

1. Đường dẫn đã có trong `config/routes.ts`, mục menu đã có trong `config/navigation.ts`?
2. Dữ liệu lấy qua hook `features/*/queries.ts`, hiển thị qua `<QueryState>`?
3. Chữ dùng lại đã lấy từ `COPY`, trạng thái từ `BOOKING_STATUS`?
4. Có test viết trước? Có cả bản điện thoại và máy tính?
5. Dùng lại component sẵn có (`Field`, `PriceText`, `StatusBadge`, `EmptyState`) thay vì viết mới?
