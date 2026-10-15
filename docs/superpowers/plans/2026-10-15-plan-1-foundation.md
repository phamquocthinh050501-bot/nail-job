# Nền tảng, cấu hình và CSDL: Kế hoạch triển khai (Kế hoạch 1/3)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Dựng khung dự án Vite + React + TypeScript, bộ config dùng chung, các hàm thuần đã có test (giờ trống, định dạng, kiểm tra dữ liệu, điểm) và toàn bộ CSDL Supabase (bảng, ràng buộc chống trùng giờ, hàm RPC, RLS, dữ liệu mẫu) có test.

**Architecture:** Mọi quy tắc quan trọng nằm ở Postgres (chống trùng giờ bằng exclusion constraint, quyền bằng RLS, nghiệp vụ bằng hàm RPC). Frontend chỉ có hàm thuần để hiển thị giờ trống và kiểm tra form. Giá trị dùng chung nằm trong `src/config/`. Quy chuẩn ở `docs/CODING_STANDARDS.md`.

**Tech Stack:** Vite, React 18, TypeScript, Tailwind CSS v4, Vitest, ESLint, Prettier, zod, date-fns, Supabase (Postgres), thư viện `pg` cho script CSDL.

**Spec:** `docs/superpowers/specs/2026-10-15-nail-salon-web-design.md` (đọc kèm `docs/CODING_STANDARDS.md`).

**Các kế hoạch kế tiếp:** `2026-10-15-plan-2-customer-site.md` (trang khách), `2026-10-15-plan-3-admin.md` (trang admin). Làm đúng thứ tự 1 → 2 → 3.

**Khác biệt có chủ ý so với spec** (để một nguồn sự thật, ít code hơn):
- `shop_settings` **không có cột `name`**: tên thương hiệu nằm ở `src/config/site.ts`.
- `complete_booking` được gộp vào `set_booking_status` (một hàm giữ một bảng chuyển trạng thái).
- Không có `couponMath` ở TypeScript: giảm giá do RPC `validate_coupon` tính. Giao diện chỉ hiển thị.
- Trường dữ liệu trong TypeScript và JSON của RPC giữ `snake_case` như CSDL (không có lớp đổi tên).
- Chuẩn quy ước thứ trong tuần: `0 = Chủ nhật … 6 = Thứ bảy`.

## Global Constraints

- Giao diện tiếng Việt; giá hiển thị `350.000đ`, lưu số nguyên VNĐ.
- Múi giờ nghiệp vụ `Asia/Ho_Chi_Minh` (UTC+7, không đổi giờ mùa hè); giờ lưu `timestamptz`.
- Lưới giờ `slot_minutes = 30`; không đặt sát giờ: `lead_minutes = 60`; hủy được khi còn `cancel_hours = 2`; `points_per_10k = 1`.
- Trạng thái lịch: `pending`, `confirmed`, `completed`, `cancelled`. Chuyển hợp lệ: `pending → confirmed → completed`; `pending/confirmed → cancelled`.
- SĐT: 10 số bắt đầu bằng `0`; chấp nhận nhập có dấu cách, dấu gạch, `+84`.
- Không đưa khóa bí mật vào mã nguồn. `DATABASE_URL` **không** dùng tiền tố `VITE_`. Chỉ dùng anon key ở frontend.
- Không export mặc định; không emoji làm icon; hex màu chỉ ở `tokens.css` và `config/gallery.ts`.
- Mọi test hàm thuần chạy với múi giờ máy **khác** Việt Nam (`America/Los_Angeles`).

## Review Focus

1. Hai người cùng đặt một giờ của cùng một thợ: chỉ một người thành công, người kia nhận `slot_taken` (Task 7 test).
2. Đặt đúng giờ đóng cửa (19:00 cho dịch vụ 60 phút kết thúc 20:00 hợp lệ; 19:30 thì không) (Task 4 và Task 7 test).
3. SĐT nhập kiểu `+84 901 234 567` hoặc `0901-234-567` vẫn được chấp nhận và chuẩn hóa (Task 3 test).
4. Hoàn thành một lịch hai lần không cộng điểm hai lần (Task 7 test `bad_transition`).
5. Máy người dùng ở múi giờ khác Việt Nam vẫn thấy đúng giờ Việt Nam (Task 3 test, chạy với `TZ` khác).

---

## Task 1: Khung dự án và công cụ

**Files:**
- Create: `package.json` (qua scaffold), `vite.config.ts`, `tsconfig.app.json` (sửa), `eslint.config.js`, `.prettierrc`, `.gitignore`, `.env.example`, `README.md`, `src/test/setup.ts`, `src/test/globalSetup.ts`, `src/test/smoke.test.ts`, `src/index.css`

**Interfaces:**
- Produces: lệnh `npm run dev | build | typecheck | lint | format | test | check | db:migrate | db:seed | db:test`; bí danh import `@/` trỏ `src/`; test chạy trong `jsdom` với `globals`.

- [ ] **Step 1: Tạo khung Vite vào thư mục tạm rồi chuyển lên**

Thư mục `C:\Thinh\Tiktok\nail` đã có `docs/` nên không tạo thẳng vào đó. Chạy trong PowerShell:

```powershell
cd C:\Thinh\Tiktok\nail
git init
npm create vite@latest _scaffold -- --template react-ts
robocopy _scaffold . /E /XD node_modules /XF README.md
Remove-Item -Recurse -Force _scaffold
```

Expected: `robocopy` trả mã thoát nhỏ hơn 8 (bình thường). Trong thư mục có `package.json`, `index.html`, `src/`, `vite.config.ts`.

- [ ] **Step 2: Cài thư viện**

```powershell
npm install react-router-dom @tanstack/react-query @supabase/supabase-js react-hook-form zod @hookform/resolvers date-fns lucide-react sonner motion clsx tailwind-merge class-variance-authority
npm install -D tailwindcss @tailwindcss/vite vitest jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event prettier eslint-config-prettier eslint-plugin-jsx-a11y pg
```

Expected: cài xong không báo lỗi. (Các thư viện giao diện còn lại như `embla-carousel-react`, `react-day-picker`, `three` cài ở Kế hoạch 2 khi cần.)

- [ ] **Step 3: Ghi `vite.config.ts`**

```ts
import path from 'node:path';
import tailwindcss from '@tailwindcss/vite';
import react from '@vitejs/plugin-react';
import { defineConfig } from 'vitest/config';

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: { alias: { '@': path.resolve(__dirname, 'src') } },
  test: {
    environment: 'jsdom',
    globals: true,
    globalSetup: ['./src/test/globalSetup.ts'],
    setupFiles: ['./src/test/setup.ts'],
    css: false,
  },
});
```

- [ ] **Step 4: Thêm bí danh và kiểu vào `tsconfig.app.json`**

Mở `tsconfig.app.json`, trong `compilerOptions` thêm (giữ nguyên các khóa có sẵn):

```json
"paths": { "@/*": ["./src/*"] },
"types": ["vite/client", "vitest/globals", "@testing-library/jest-dom"]
```

- [ ] **Step 5: Ghi các tệp test chung**

`src/test/globalSetup.ts`:

```ts
// Chạy test ở múi giờ khác Việt Nam để chứng minh code không phụ thuộc máy.
export default function setup() {
  process.env.TZ = 'America/Los_Angeles';
}
```

`src/test/setup.ts`:

```ts
import '@testing-library/jest-dom/vitest';
```

`src/test/smoke.test.ts`:

```ts
describe('khung test', () => {
  it('chạy được và dùng múi giờ khác Việt Nam', () => {
    expect(1 + 1).toBe(2);
    expect(new Date(2030, 0, 1).getTimezoneOffset()).not.toBe(-420);
  });
});
```

- [ ] **Step 6: Ghi cấu hình lint, định dạng, môi trường**

`.prettierrc`:

```json
{ "singleQuote": true, "semi": true, "printWidth": 100, "trailingComma": "all" }
```

`eslint.config.js` (thay nội dung do Vite tạo):

```js
import js from '@eslint/js';
import globals from 'globals';
import reactHooks from 'eslint-plugin-react-hooks';
import reactRefresh from 'eslint-plugin-react-refresh';
import jsxA11y from 'eslint-plugin-jsx-a11y';
import prettier from 'eslint-config-prettier';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  { ignores: ['dist', 'node_modules', '.superpowers'] },
  {
    files: ['**/*.{ts,tsx}'],
    extends: [js.configs.recommended, ...tseslint.configs.recommended, jsxA11y.flatConfigs.recommended, prettier],
    languageOptions: { ecmaVersion: 2022, globals: { ...globals.browser, vi: 'readonly' } },
    plugins: { 'react-hooks': reactHooks, 'react-refresh': reactRefresh },
    rules: {
      ...reactHooks.configs.recommended.rules,
      'react-refresh/only-export-components': ['warn', { allowConstantExport: true }],
      'no-restricted-exports': ['error', { restrictDefaultExports: { direct: true } }],
    },
  },
  { files: ['vite.config.ts', 'eslint.config.js'], rules: { 'no-restricted-exports': 'off' } },
);
```

`.env.example`:

```
# Frontend (lộ ra trình duyệt, chỉ dùng anon key)
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=

# Chỉ cho script CSDL trên máy bạn. TUYỆT ĐỐI không đặt tiền tố VITE_.
# Dùng project Supabase THỬ NGHIỆM, không dùng project thật của khách.
DATABASE_URL=
```

Thêm vào `.gitignore` (nối cuối): các dòng `.env.local`, `.superpowers/`, `playwright-report/`, `test-results/`.

`src/index.css` (thay toàn bộ nội dung):

```css
@import 'tailwindcss';
```

- [ ] **Step 7: Sửa `package.json` phần `scripts`**

```json
"scripts": {
  "dev": "vite",
  "build": "tsc -b && vite build",
  "preview": "vite preview",
  "typecheck": "tsc -b",
  "lint": "eslint .",
  "format": "prettier --write .",
  "test": "vitest run",
  "test:watch": "vitest",
  "check": "npm run typecheck && npm run lint && npm run test",
  "db:migrate": "node --env-file=.env.local scripts/db.mjs migrate",
  "db:seed": "node --env-file=.env.local scripts/db.mjs seed",
  "db:test": "node --env-file=.env.local scripts/db.mjs test"
}
```

- [ ] **Step 8: Ghi `README.md` ngắn**

```markdown
# Website tiệm nail (bản Demo)

Đọc `docs/CODING_STANDARDS.md` trước, rồi spec trong `docs/superpowers/specs/` và các kế hoạch trong `docs/superpowers/plans/`.

Chạy: `npm install`, sao chép `.env.example` thành `.env.local` và điền, rồi `npm run dev`. Kiểm tra: `npm run check`.
```

- [ ] **Step 9: Chạy kiểm tra để thấy khung hoạt động**

Run: `npm run check`
Expected: typecheck, lint, test đều qua (1 test `khung test` PASS). Nếu `typecheck` báo lỗi kiểu ở các tệp mẫu của Vite (`App.tsx`), xóa nội dung mẫu: thay `src/App.tsx` bằng `export function App() { return <p>Lụa Nails</p>; }`, sửa `src/main.tsx` import `{ App }` thay vì mặc định, xóa `src/App.css`, `src/assets/`.

- [ ] **Step 10: Commit**

```bash
git add -A
git commit -m "chore: khung Vite React TS, Tailwind, Vitest, ESLint, Prettier"
```

---

## Task 2: Bộ config dùng chung

**Files:**
- Create: `src/config/site.ts`, `src/config/routes.ts`, `src/config/navigation.ts`, `src/config/statuses.ts`, `src/config/booking.ts`, `src/config/gallery.ts`, `src/config/copy.ts`
- Test: `src/config/statuses.test.ts`, `src/config/routes.test.ts`

**Interfaces:**
- Produces:
  - `SITE`, `WEEKDAY_LABELS`
  - `ROUTES`, `bookingUrl(params?: { serviceId?: string; designId?: string }): string`
  - `CUSTOMER_NAV: NavItem[]`, `ADMIN_NAV: AdminNavItem[]`, `type NavItem = { label: string; to: string }`
  - `BOOKING_STATUS`, `BOOKING_STATUS_LIST`, `canTransition(from, to): boolean`, `type BookingStatus`, `type StatusTone`
  - `BOOKING_STEPS`, `type BookingStepId`, `BOOKING_UI`
  - `GALLERY_STYLES`, `type GalleryStyle`
  - `COPY` (nhóm `actions`, `errors`, `validation`, `empty`, `coupon`)

- [ ] **Step 1: Viết test hỏng**

`src/config/statuses.test.ts`:

```ts
import { BOOKING_STATUS, BOOKING_STATUS_LIST, canTransition } from './statuses';

describe('BOOKING_STATUS', () => {
  it('có đủ 4 trạng thái theo thứ tự quy trình', () => {
    expect(BOOKING_STATUS_LIST).toEqual(['pending', 'confirmed', 'completed', 'cancelled']);
  });

  it('gắn nhãn tiếng Việt cố định', () => {
    expect(BOOKING_STATUS.pending.label).toBe('Chờ xác nhận');
    expect(BOOKING_STATUS.confirmed.label).toBe('Đã xác nhận');
    expect(BOOKING_STATUS.completed.label).toBe('Hoàn thành');
    expect(BOOKING_STATUS.cancelled.label).toBe('Đã hủy');
  });

  it('cho chuyển pending → confirmed → completed và hủy từ pending hoặc confirmed', () => {
    expect(canTransition('pending', 'confirmed')).toBe(true);
    expect(canTransition('confirmed', 'completed')).toBe(true);
    expect(canTransition('pending', 'cancelled')).toBe(true);
    expect(canTransition('confirmed', 'cancelled')).toBe(true);
  });

  it('không cho quay lui hoặc nhảy bước', () => {
    expect(canTransition('completed', 'pending')).toBe(false);
    expect(canTransition('cancelled', 'confirmed')).toBe(false);
    expect(canTransition('pending', 'completed')).toBe(false);
  });
});
```

`src/config/routes.test.ts`:

```ts
import { bookingUrl, ROUTES } from './routes';

describe('bookingUrl', () => {
  it('không có tham số thì trả đường dẫn gốc', () => {
    expect(bookingUrl()).toBe(ROUTES.booking);
  });

  it('điền sẵn dịch vụ và mẫu', () => {
    expect(bookingUrl({ serviceId: 'abc', designId: 'xyz' })).toBe('/dat-lich?dich-vu=abc&mau=xyz');
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/config`
Expected: FAIL (không tìm thấy `./statuses`, `./routes`).

- [ ] **Step 3: Viết các tệp config**

`src/config/site.ts`:

```ts
export const SITE = {
  brandName: 'Lụa Nails',
  tagline: 'Móng đẹp, đặt lịch trong 30 giây',
  timeZone: 'Asia/Ho_Chi_Minh',
  // Khách đăng nhập bằng SĐT + PIN; Supabase Auth cần email nên ta dựng email giả từ SĐT.
  authEmailDomain: 'khach.lua-nails.local',
} as const;

// 0 = Chủ nhật, khớp extract(dow) của Postgres và getUTCDay của JS.
export const WEEKDAY_LABELS = ['CN', 'T2', 'T3', 'T4', 'T5', 'T6', 'T7'] as const;
```

`src/config/routes.ts`:

```ts
export const ROUTES = {
  home: '/',
  services: '/dich-vu',
  gallery: '/mau-nail',
  booking: '/dat-lich',
  bookingSuccess: '/dat-lich/thanh-cong',
  lookup: '/tra-cuu',
  login: '/dang-nhap',
  account: '/tai-khoan',
  contact: '/lien-he',
  admin: {
    root: '/quan-tri',
    login: '/quan-tri/dang-nhap',
    appointments: '/quan-tri/lich-hen',
    services: '/quan-tri/dich-vu',
    staff: '/quan-tri/tho',
    customers: '/quan-tri/khach-hang',
    promos: '/quan-tri/khuyen-mai',
    revenue: '/quan-tri/doanh-thu',
  },
} as const;

export function bookingUrl(params: { serviceId?: string; designId?: string } = {}) {
  const query = new URLSearchParams();
  if (params.serviceId) query.set('dich-vu', params.serviceId);
  if (params.designId) query.set('mau', params.designId);
  const text = query.toString();
  return text ? `${ROUTES.booking}?${text}` : ROUTES.booking;
}
```

`src/config/statuses.ts`:

```ts
import { CalendarCheck, CheckCheck, Clock, XCircle, type LucideIcon } from 'lucide-react';

export type BookingStatus = 'pending' | 'confirmed' | 'completed' | 'cancelled';
export type StatusTone = 'warn' | 'info' | 'success' | 'danger';

type StatusDef = {
  label: string;
  tone: StatusTone;
  icon: LucideIcon;
  next: readonly BookingStatus[];
};

// Quy tắc chuyển trạng thái thật nằm ở hàm set_booking_status trong CSDL.
// Bảng này chỉ quyết định nút nào hiện ra; nếu đổi một nơi phải đổi nơi kia.
export const BOOKING_STATUS: Record<BookingStatus, StatusDef> = {
  pending: { label: 'Chờ xác nhận', tone: 'warn', icon: Clock, next: ['confirmed', 'cancelled'] },
  confirmed: {
    label: 'Đã xác nhận',
    tone: 'info',
    icon: CalendarCheck,
    next: ['completed', 'cancelled'],
  },
  completed: { label: 'Hoàn thành', tone: 'success', icon: CheckCheck, next: [] },
  cancelled: { label: 'Đã hủy', tone: 'danger', icon: XCircle, next: [] },
};

export const BOOKING_STATUS_LIST = Object.keys(BOOKING_STATUS) as BookingStatus[];

export const canTransition = (from: BookingStatus, to: BookingStatus) =>
  BOOKING_STATUS[from].next.includes(to);
```

`src/config/navigation.ts`:

```ts
import {
  BarChart3,
  CalendarDays,
  Contact,
  LayoutDashboard,
  Sparkles,
  Tag,
  Users,
  type LucideIcon,
} from 'lucide-react';
import { ROUTES } from './routes';

export type NavItem = { label: string; to: string };
export type AdminNavItem = NavItem & { icon: LucideIcon };

export const CUSTOMER_NAV: NavItem[] = [
  { label: 'Trang chủ', to: ROUTES.home },
  { label: 'Dịch vụ', to: ROUTES.services },
  { label: 'Mẫu nail', to: ROUTES.gallery },
  { label: 'Đặt lịch', to: ROUTES.booking },
  { label: 'Liên hệ', to: ROUTES.contact },
];

export const ADMIN_NAV: AdminNavItem[] = [
  { label: 'Tổng quan', to: ROUTES.admin.root, icon: LayoutDashboard },
  { label: 'Lịch hẹn', to: ROUTES.admin.appointments, icon: CalendarDays },
  { label: 'Dịch vụ', to: ROUTES.admin.services, icon: Sparkles },
  { label: 'Thợ và giờ', to: ROUTES.admin.staff, icon: Users },
  { label: 'Khách hàng', to: ROUTES.admin.customers, icon: Contact },
  { label: 'Khuyến mãi', to: ROUTES.admin.promos, icon: Tag },
  { label: 'Doanh thu', to: ROUTES.admin.revenue, icon: BarChart3 },
];
```

`src/config/booking.ts`:

```ts
export const BOOKING_STEPS = [
  { id: 'service', label: 'Dịch vụ' },
  { id: 'staff', label: 'Thợ' },
  { id: 'date', label: 'Ngày' },
  { id: 'time', label: 'Giờ' },
  { id: 'contact', label: 'Thông tin' },
] as const;

export type BookingStepId = (typeof BOOKING_STEPS)[number]['id'];

// Chỉ giá trị giao diện. slot_minutes, lead_minutes, cancel_hours nằm ở bảng shop_settings.
export const BOOKING_UI = {
  daysAhead: 14,
  noteMaxLength: 300,
  anyStaffLabel: 'Bất kỳ',
} as const;
```

`src/config/gallery.ts`:

```ts
// Màu ô giữ chỗ khi mẫu chưa có ảnh thật.
export const GALLERY_STYLES = {
  ombre: { label: 'Ombre', from: '#ffd1dc', to: '#f7a8c0' },
  french: { label: 'French', from: '#fff1e6', to: '#e5d4f5' },
  'dinh-da': { label: 'Đính đá', from: '#d8f4ff', to: '#8f6bff' },
  '3d': { label: '3D', from: '#ffe3c8', to: '#ff7ad9' },
  'mat-meo': { label: 'Mắt mèo', from: '#c9e8f5', to: '#7b2d8e' },
} as const;

export type GalleryStyle = keyof typeof GALLERY_STYLES;
```

`src/config/copy.ts`:

```ts
// Chữ giao diện dùng từ hai nơi trở lên. Chữ chỉ dùng ở một trang thì viết thẳng trong trang.
export const COPY = {
  actions: {
    book: 'Đặt lịch',
    confirm: 'Xác nhận',
    confirmBooking: 'Xác nhận đặt lịch',
    complete: 'Hoàn thành',
    cancelBooking: 'Hủy lịch',
    keepBooking: 'Giữ lịch',
    save: 'Lưu thay đổi',
    retry: 'Thử lại',
    close: 'Đóng',
    back: 'Quay lại',
    next: 'Tiếp tục',
    edit: 'Sửa',
    delete: 'Xóa',
    add: 'Thêm',
    login: 'Đăng nhập',
    logout: 'Đăng xuất',
  },
  errors: {
    generic: 'Có lỗi xảy ra. Vui lòng thử lại.',
    network: 'Chưa kết nối được. Kiểm tra mạng rồi thử lại.',
    forbidden: 'Bạn không có quyền thực hiện thao tác này.',
    badCredentials: 'SĐT hoặc PIN chưa đúng.',
    // Mã lỗi do hàm RPC trong CSDL ném ra, xem supabase/migrations/0002_functions.sql
    slot_taken: 'Giờ này vừa có người đặt. Chọn giờ khác nhé.',
    slot_past: 'Giờ này đã qua hoặc quá sát. Chọn giờ khác nhé.',
    slot_invalid: 'Giờ này tiệm không nhận lịch. Chọn giờ khác nhé.',
    bad_phone: 'SĐT chưa đúng. Nhập 10 số, bắt đầu bằng 0.',
    bad_name: 'Vui lòng nhập họ tên (tối thiểu 2 ký tự).',
    service_not_found: 'Dịch vụ này không còn nữa. Chọn dịch vụ khác nhé.',
    coupon_invalid: 'Mã giảm giá không dùng được. Kiểm tra lại mã.',
    booking_not_found: 'Không tìm thấy lịch. Kiểm tra lại SĐT và mã lịch.',
    cancel_not_allowed: 'Lịch này không thể hủy nữa.',
    cancel_too_late: 'Đã quá hạn hủy trực tuyến. Vui lòng gọi tiệm.',
    bad_transition: 'Không thể chuyển lịch sang trạng thái này.',
    phone_taken: 'SĐT này đã có tài khoản. Hãy đăng nhập.',
  },
  validation: {
    required: 'Vui lòng nhập thông tin này.',
    phone: 'SĐT chưa đúng. Nhập 10 số, bắt đầu bằng 0.',
    name: 'Vui lòng nhập họ tên (tối thiểu 2 ký tự).',
    noteTooLong: 'Ghi chú tối đa 300 ký tự.',
    pin: 'PIN gồm đúng 6 chữ số.',
    pinMismatch: 'Hai lần nhập PIN chưa khớp.',
  },
  empty: {
    noBookings: 'Chưa có lịch hẹn. Đặt lịch đầu tiên của bạn.',
    noResults: 'Không có kết quả phù hợp.',
  },
  // Lý do validate_coupon trả về (trường reason).
  coupon: {
    not_found: 'Mã không tồn tại.',
    inactive: 'Mã chưa được áp dụng.',
    expired: 'Mã đã hết hạn.',
    used_up: 'Mã đã hết lượt dùng.',
    min_total: 'Đơn chưa đạt mức tối thiểu để dùng mã.',
  },
} as const;

export type ErrorCode = keyof typeof COPY.errors;
```

- [ ] **Step 4: Chạy test để thấy qua**

Run: `npx vitest run src/config`
Expected: PASS (6 test).

- [ ] **Step 5: Commit**

```bash
git add src/config
git commit -m "feat(config): bộ config dùng chung (site, routes, navigation, statuses, booking, gallery, copy)"
```

---

## Task 3: Giờ Việt Nam, định dạng và chuẩn hóa SĐT

**Files:**
- Create: `src/lib/vnTime.ts`, `src/lib/format.ts`, `src/lib/phone.ts`
- Test: `src/lib/vnTime.test.ts`, `src/lib/format.test.ts`, `src/lib/phone.test.ts`

**Interfaces:**
- Consumes: `WEEKDAY_LABELS` từ `@/config/site`.
- Produces:
  - `vnToIso(day: string, minutes: number): string`, `isoToVn(iso: string): { day: string; minutes: number }`, `todayVn(now?: Date): string`, `weekdayOf(day: string): number`, `addDays(day: string, n: number): string`, `timeToMinutes(time: string): number`, `minutesToLabel(minutes: number): string`
  - `formatVnd(n: number): string`, `formatDuration(min: number): string`, `formatDayVi(day: string): string`, `formatDateTimeVi(iso: string): string`
  - `normalizePhone(input: string): string`, `isValidPhone(phone: string): boolean`

- [ ] **Step 1: Viết test hỏng**

`src/lib/vnTime.test.ts`:

```ts
import {
  addDays,
  isoToVn,
  minutesToLabel,
  timeToMinutes,
  todayVn,
  vnToIso,
  weekdayOf,
} from './vnTime';

describe('vnTime', () => {
  it('dựng chuỗi ISO có +07:00', () => {
    expect(vnToIso('2030-03-04', 570)).toBe('2030-03-04T09:30:00+07:00');
  });

  it('đọc đúng ngày và phút theo giờ Việt Nam', () => {
    expect(isoToVn('2030-03-04T09:30:00+07:00')).toEqual({ day: '2030-03-04', minutes: 570 });
  });

  it('qua nửa đêm UTC vẫn đúng ngày Việt Nam', () => {
    expect(isoToVn('2030-03-03T18:00:00Z')).toEqual({ day: '2030-03-04', minutes: 60 });
  });

  it('todayVn lấy ngày theo giờ Việt Nam, không theo máy', () => {
    expect(todayVn(new Date('2030-03-03T18:00:00Z'))).toBe('2030-03-04');
  });

  it('weekdayOf: 0 là Chủ nhật, thứ Hai là 1', () => {
    expect(weekdayOf('2030-03-04')).toBe(1);
    expect(weekdayOf('2030-03-10')).toBe(0);
  });

  it('addDays qua tháng và qua năm', () => {
    expect(addDays('2030-02-28', 1)).toBe('2030-03-01');
    expect(addDays('2030-12-31', 1)).toBe('2031-01-01');
    expect(addDays('2030-03-01', -1)).toBe('2030-02-28');
  });

  it('timeToMinutes đọc giờ dạng HH:MM:SS của Postgres', () => {
    expect(timeToMinutes('09:30:00')).toBe(570);
    expect(timeToMinutes('20:00')).toBe(1200);
  });

  it('minutesToLabel không đệm số 0 ở giờ', () => {
    expect(minutesToLabel(540)).toBe('9:00');
    expect(minutesToLabel(1185)).toBe('19:45');
  });
});
```

`src/lib/format.test.ts`:

```ts
import { formatDateTimeVi, formatDayVi, formatDuration, formatVnd } from './format';

describe('format', () => {
  it('formatVnd dùng dấu chấm ngăn nghìn và hậu tố đ', () => {
    expect(formatVnd(350000)).toBe('350.000đ');
    expect(formatVnd(4850000)).toBe('4.850.000đ');
    expect(formatVnd(0)).toBe('0đ');
  });

  it('formatDuration', () => {
    expect(formatDuration(60)).toBe('60 phút');
  });

  it('formatDayVi gồm thứ, ngày/tháng/năm', () => {
    expect(formatDayVi('2030-03-04')).toBe('T2, 04/03/2030');
  });

  it('formatDateTimeVi đọc theo giờ Việt Nam', () => {
    expect(formatDateTimeVi('2030-03-04T10:00:00+07:00')).toBe('T2, 04/03/2030, 10:00');
    expect(formatDateTimeVi('2030-03-04T09:30:00+07:00')).toBe('T2, 04/03/2030, 9:30');
  });
});
```

`src/lib/phone.test.ts`:

```ts
import { isValidPhone, normalizePhone } from './phone';

describe('phone', () => {
  it('bỏ dấu cách và dấu gạch', () => {
    expect(normalizePhone('0901 234 567')).toBe('0901234567');
    expect(normalizePhone('0901-234-567')).toBe('0901234567');
  });

  it('đổi +84 và 84 thành 0', () => {
    expect(normalizePhone('+84 901 234 567')).toBe('0901234567');
    expect(normalizePhone('84901234567')).toBe('0901234567');
  });

  it('isValidPhone chỉ nhận 10 số bắt đầu bằng 0', () => {
    expect(isValidPhone('0901234567')).toBe(true);
    expect(isValidPhone('901234567')).toBe(false);
    expect(isValidPhone('12345')).toBe(false);
    expect(isValidPhone('09012345678')).toBe(false);
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/lib`
Expected: FAIL (không tìm thấy module).

- [ ] **Step 3: Viết code**

`src/lib/vnTime.ts`:

```ts
// Việt Nam là UTC+7 quanh năm nên không cần thư viện múi giờ.
// Một "ngày của tiệm" là chuỗi 'YYYY-MM-DD'; một giờ trong ngày là số phút kể từ 0:00.
const VN_OFFSET_MS = 7 * 60 * 60 * 1000;
const DAY_MS = 24 * 60 * 60 * 1000;

const pad = (n: number) => String(n).padStart(2, '0');

export function vnToIso(day: string, minutes: number): string {
  return `${day}T${pad(Math.floor(minutes / 60))}:${pad(minutes % 60)}:00+07:00`;
}

export function isoToVn(iso: string): { day: string; minutes: number } {
  const shifted = new Date(new Date(iso).getTime() + VN_OFFSET_MS);
  return {
    day: shifted.toISOString().slice(0, 10),
    minutes: shifted.getUTCHours() * 60 + shifted.getUTCMinutes(),
  };
}

export const todayVn = (now: Date = new Date()) => isoToVn(now.toISOString()).day;

// 0 = Chủ nhật.
export const weekdayOf = (day: string) => new Date(`${day}T00:00:00Z`).getUTCDay();

export const addDays = (day: string, n: number) =>
  new Date(Date.parse(`${day}T00:00:00Z`) + n * DAY_MS).toISOString().slice(0, 10);

export const timeToMinutes = (time: string) => {
  const [h, m] = time.split(':').map(Number);
  return h * 60 + m;
};

export const minutesToLabel = (minutes: number) =>
  `${Math.floor(minutes / 60)}:${pad(minutes % 60)}`;
```

`src/lib/format.ts`:

```ts
import { WEEKDAY_LABELS } from '@/config/site';
import { isoToVn, minutesToLabel, weekdayOf } from './vnTime';

export const formatVnd = (n: number) => `${n.toLocaleString('vi-VN')}đ`;

export const formatDuration = (min: number) => `${min} phút`;

export function formatDayVi(day: string) {
  const [y, m, d] = day.split('-');
  return `${WEEKDAY_LABELS[weekdayOf(day)]}, ${d}/${m}/${y}`;
}

export function formatDateTimeVi(iso: string) {
  const { day, minutes } = isoToVn(iso);
  return `${formatDayVi(day)}, ${minutesToLabel(minutes)}`;
}
```

`src/lib/phone.ts`:

```ts
export function normalizePhone(input: string): string {
  const cleaned = input.replace(/[^\d+]/g, '');
  if (cleaned.startsWith('+84')) return `0${cleaned.slice(3)}`;
  if (cleaned.startsWith('84') && cleaned.length === 11) return `0${cleaned.slice(2)}`;
  return cleaned;
}

export const isValidPhone = (phone: string) => /^0\d{9}$/.test(phone);
```

- [ ] **Step 4: Chạy để thấy qua**

Run: `npx vitest run src/lib`
Expected: PASS (15 test). Test chạy ở múi giờ `America/Los_Angeles` nhờ `globalSetup`.

- [ ] **Step 5: Commit**

```bash
git add src/lib
git commit -m "feat(lib): giờ Việt Nam, định dạng VNĐ/ngày giờ, chuẩn hóa SĐT"
```

---

## Task 4: Tính giờ trống (hàm thuần)

**Files:**
- Create: `src/features/booking/slots.ts`
- Test: `src/features/booking/slots.test.ts`

**Interfaces:**
- Consumes: `isoToVn`, `timeToMinutes` từ `@/lib/vnTime`.
- Produces:
  - `type Interval = { start: number; end: number }` (phút kể từ 0:00, nửa mở `[start, end)`)
  - `computeSlots(input: { windows: Interval[]; busy: Interval[]; durationMin: number; stepMin: number; earliestStartMin?: number }): number[]`
  - `mergeSlots(lists: number[][]): number[]`
  - `type SlotView = { minutes: number; available: boolean }`
  - `buildSlotGrid(input: { open: Interval | null; durationMin: number; stepMin: number; available: number[] }): SlotView[]`
  - `intersect(a: Interval, b: Interval): Interval | null`
  - `openWindow(hours?: { open_time: string; close_time: string; is_closed: boolean }): Interval | null`
  - `staffWorkWindow(schedule: { start_time: string; end_time: string; is_off: boolean } | undefined, open: Interval | null): Interval | null`
  - `busyToIntervals(rows: { start_at: string; end_at: string }[]): Interval[]`
  - `earliestStartFor(day: string, nowIso: string, leadMin: number): number`

- [ ] **Step 1: Viết test hỏng**

`src/features/booking/slots.test.ts`:

```ts
import {
  buildSlotGrid,
  busyToIntervals,
  computeSlots,
  earliestStartFor,
  intersect,
  mergeSlots,
  openWindow,
  staffWorkWindow,
} from './slots';

describe('computeSlots', () => {
  it('liệt kê mọi giờ trên lưới khi không có lịch bận, kể cả giờ kết thúc đúng giờ đóng cửa', () => {
    const r = computeSlots({
      windows: [{ start: 540, end: 660 }],
      busy: [],
      durationMin: 60,
      stepMin: 30,
    });
    expect(r).toEqual([540, 570, 600]); // 9:00, 9:30, 10:00 (10:00 + 60 = 11:00 = giờ đóng)
  });

  it('bỏ giờ chồng lên lịch bận, giữ giờ kề nhau', () => {
    const r = computeSlots({
      windows: [{ start: 540, end: 780 }],
      busy: [{ start: 600, end: 660 }],
      durationMin: 60,
      stepMin: 30,
    });
    expect(r).toEqual([540, 660, 690, 720]);
  });

  it('bỏ giờ trước earliestStartMin và làm tròn lên lưới', () => {
    const r = computeSlots({
      windows: [{ start: 540, end: 720 }],
      busy: [],
      durationMin: 60,
      stepMin: 30,
      earliestStartMin: 545,
    });
    expect(r).toEqual([570, 600, 630, 660]);
  });

  it('hỗ trợ nhiều khung làm việc (nghỉ trưa)', () => {
    const r = computeSlots({
      windows: [
        { start: 540, end: 660 },
        { start: 780, end: 900 },
      ],
      busy: [],
      durationMin: 60,
      stepMin: 60,
    });
    expect(r).toEqual([540, 600, 780, 840]);
  });

  it('thời lượng dài hơn khung thì không có giờ nào', () => {
    expect(
      computeSlots({ windows: [{ start: 540, end: 600 }], busy: [], durationMin: 120, stepMin: 30 }),
    ).toEqual([]);
  });

  it('earliestStartMin = Infinity (ngày đã qua) thì không có giờ nào', () => {
    expect(
      computeSlots({
        windows: [{ start: 540, end: 1200 }],
        busy: [],
        durationMin: 60,
        stepMin: 30,
        earliestStartMin: Infinity,
      }),
    ).toEqual([]);
  });
});

describe('mergeSlots', () => {
  it('gộp giờ của nhiều thợ, bỏ trùng, sắp tăng dần', () => {
    expect(mergeSlots([[540, 600], [570, 600]])).toEqual([540, 570, 600]);
  });
});

describe('buildSlotGrid', () => {
  it('trả toàn bộ giờ trên lưới và đánh dấu giờ nào còn trống', () => {
    const grid = buildSlotGrid({
      open: { start: 540, end: 660 },
      durationMin: 60,
      stepMin: 30,
      available: [540, 600],
    });
    expect(grid).toEqual([
      { minutes: 540, available: true },
      { minutes: 570, available: false },
      { minutes: 600, available: true },
    ]);
  });

  it('ngày tiệm nghỉ thì lưới rỗng', () => {
    expect(buildSlotGrid({ open: null, durationMin: 60, stepMin: 30, available: [] })).toEqual([]);
  });
});

describe('cửa sổ làm việc', () => {
  it('intersect lấy phần chung hoặc null', () => {
    expect(intersect({ start: 540, end: 720 }, { start: 600, end: 900 })).toEqual({
      start: 600,
      end: 720,
    });
    expect(intersect({ start: 540, end: 600 }, { start: 600, end: 900 })).toBeNull();
  });

  it('openWindow đọc giờ Postgres, trả null khi tiệm nghỉ', () => {
    expect(openWindow({ open_time: '09:00:00', close_time: '20:00:00', is_closed: false })).toEqual({
      start: 540,
      end: 1200,
    });
    expect(openWindow({ open_time: '09:00:00', close_time: '20:00:00', is_closed: true })).toBeNull();
    expect(openWindow(undefined)).toBeNull();
  });

  it('staffWorkWindow nằm trong giờ mở cửa, null khi thợ nghỉ', () => {
    const open = { start: 540, end: 1200 };
    expect(
      staffWorkWindow({ start_time: '12:00:00', end_time: '21:00:00', is_off: false }, open),
    ).toEqual({ start: 720, end: 1200 });
    expect(staffWorkWindow({ start_time: '09:00', end_time: '20:00', is_off: true }, open)).toBeNull();
    expect(staffWorkWindow(undefined, open)).toBeNull();
  });
});

describe('busyToIntervals', () => {
  it('đổi lịch bận sang phút theo giờ Việt Nam', () => {
    expect(
      busyToIntervals([
        { start_at: '2030-03-04T10:00:00+07:00', end_at: '2030-03-04T11:30:00+07:00' },
      ]),
    ).toEqual([{ start: 600, end: 690 }]);
  });
});

describe('earliestStartFor', () => {
  const now = '2030-03-04T09:10:00+07:00';
  it('hôm nay: bây giờ cộng thời gian đệm', () => {
    expect(earliestStartFor('2030-03-04', now, 60)).toBe(9 * 60 + 10 + 60);
  });
  it('ngày sau: không giới hạn', () => {
    expect(earliestStartFor('2030-03-05', now, 60)).toBe(0);
  });
  it('ngày đã qua: không còn giờ nào', () => {
    expect(earliestStartFor('2030-03-03', now, 60)).toBe(Infinity);
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/features/booking/slots.test.ts`
Expected: FAIL (không tìm thấy `./slots`).

- [ ] **Step 3: Viết code**

`src/features/booking/slots.ts`:

```ts
import { isoToVn, timeToMinutes } from '@/lib/vnTime';

// Mọi khoảng đều là phút kể từ 0:00 của một ngày, nửa mở [start, end).
export type Interval = { start: number; end: number };

type SlotInput = {
  windows: Interval[];
  busy: Interval[];
  durationMin: number;
  stepMin: number;
  earliestStartMin?: number;
};

export function computeSlots({
  windows,
  busy,
  durationMin,
  stepMin,
  earliestStartMin = 0,
}: SlotInput): number[] {
  const slots: number[] = [];
  for (const w of windows) {
    const first = Math.ceil(Math.max(w.start, earliestStartMin) / stepMin) * stepMin;
    for (let t = first; t + durationMin <= w.end; t += stepMin) {
      const clash = busy.some((b) => t < b.end && b.start < t + durationMin);
      if (!clash) slots.push(t);
    }
  }
  return slots;
}

export const mergeSlots = (lists: number[][]) =>
  [...new Set(lists.flat())].sort((a, b) => a - b);

export type SlotView = { minutes: number; available: boolean };

export function buildSlotGrid(input: {
  open: Interval | null;
  durationMin: number;
  stepMin: number;
  available: number[];
}): SlotView[] {
  const { open, durationMin, stepMin, available } = input;
  if (!open) return [];
  const free = new Set(available);
  const grid: SlotView[] = [];
  for (let t = Math.ceil(open.start / stepMin) * stepMin; t + durationMin <= open.end; t += stepMin) {
    grid.push({ minutes: t, available: free.has(t) });
  }
  return grid;
}

export function intersect(a: Interval, b: Interval): Interval | null {
  const start = Math.max(a.start, b.start);
  const end = Math.min(a.end, b.end);
  return start < end ? { start, end } : null;
}

export function openWindow(
  hours?: { open_time: string; close_time: string; is_closed: boolean },
): Interval | null {
  if (!hours || hours.is_closed) return null;
  return { start: timeToMinutes(hours.open_time), end: timeToMinutes(hours.close_time) };
}

export function staffWorkWindow(
  schedule: { start_time: string; end_time: string; is_off: boolean } | undefined,
  open: Interval | null,
): Interval | null {
  if (!schedule || schedule.is_off || !open) return null;
  return intersect(
    { start: timeToMinutes(schedule.start_time), end: timeToMinutes(schedule.end_time) },
    open,
  );
}

// CSDL không cho lịch vắt qua nửa đêm nên đổi sang phút trong ngày là đủ.
export const busyToIntervals = (rows: { start_at: string; end_at: string }[]): Interval[] =>
  rows.map((r) => ({ start: isoToVn(r.start_at).minutes, end: isoToVn(r.end_at).minutes }));

export function earliestStartFor(day: string, nowIso: string, leadMin: number): number {
  const now = isoToVn(nowIso);
  if (day < now.day) return Infinity;
  return day === now.day ? now.minutes + leadMin : 0;
}
```

- [ ] **Step 4: Chạy để thấy qua**

Run: `npx vitest run src/features/booking/slots.test.ts`
Expected: PASS (13 test).

- [ ] **Step 5: Commit**

```bash
git add src/features/booking
git commit -m "feat(booking): tính giờ trống bằng hàm thuần, có test biên"
```

---

## Task 5: Kiểm tra dữ liệu form và tính điểm

**Files:**
- Create: `src/features/booking/schemas.ts`, `src/features/auth/schemas.ts`, `src/features/points/pointsMath.ts`
- Test: `src/features/booking/schemas.test.ts`, `src/features/auth/schemas.test.ts`, `src/features/points/pointsMath.test.ts`

**Interfaces:**
- Consumes: `COPY`, `BOOKING_UI`, `isValidPhone`, `normalizePhone`.
- Produces:
  - `phoneSchema` (trong `booking/schemas.ts`, dùng lại ở auth), `contactSchema`, `type ContactForm`
  - `pinSchema`, `loginSchema`, `registerSchema`, `type LoginForm`, `type RegisterForm` (trong `auth/schemas.ts`)
  - `pointsFor(totalVnd: number, per10k: number): number`

- [ ] **Step 1: Viết test hỏng**

`src/features/booking/schemas.test.ts`:

```ts
import { COPY } from '@/config/copy';
import { contactSchema } from './schemas';

const valid = { full_name: 'Lan', phone: '0901 234 567' };

describe('contactSchema', () => {
  it('nhận SĐT có dấu cách', () => {
    expect(contactSchema.safeParse(valid).success).toBe(true);
  });

  it('nhận SĐT dạng +84', () => {
    expect(contactSchema.safeParse({ ...valid, phone: '+84 901 234 567' }).success).toBe(true);
  });

  it('báo lỗi SĐT sai bằng chữ tiếng Việt cố định', () => {
    const r = contactSchema.safeParse({ ...valid, phone: '12345' });
    expect(r.success).toBe(false);
    if (!r.success) expect(r.error.issues[0].message).toBe(COPY.validation.phone);
  });

  it('họ tên tối thiểu 2 ký tự sau khi bỏ khoảng trắng', () => {
    expect(contactSchema.safeParse({ ...valid, full_name: ' A ' }).success).toBe(false);
  });

  it('ghi chú tối đa 300 ký tự', () => {
    expect(contactSchema.safeParse({ ...valid, note: 'a'.repeat(300) }).success).toBe(true);
    expect(contactSchema.safeParse({ ...valid, note: 'a'.repeat(301) }).success).toBe(false);
  });
});
```

`src/features/auth/schemas.test.ts`:

```ts
import { COPY } from '@/config/copy';
import { loginSchema, registerSchema } from './schemas';

describe('loginSchema', () => {
  it('PIN phải đúng 6 chữ số', () => {
    expect(loginSchema.safeParse({ phone: '0901234567', pin: '123456' }).success).toBe(true);
    expect(loginSchema.safeParse({ phone: '0901234567', pin: '12345' }).success).toBe(false);
    expect(loginSchema.safeParse({ phone: '0901234567', pin: '12345a' }).success).toBe(false);
  });
});

describe('registerSchema', () => {
  const base = { full_name: 'Lan', phone: '0901234567', pin: '123456', pin_confirm: '123456' };

  it('nhận dữ liệu hợp lệ', () => {
    expect(registerSchema.safeParse(base).success).toBe(true);
  });

  it('báo lỗi ở ô nhập lại khi hai PIN không khớp', () => {
    const r = registerSchema.safeParse({ ...base, pin_confirm: '654321' });
    expect(r.success).toBe(false);
    if (!r.success) {
      expect(r.error.issues[0].path).toEqual(['pin_confirm']);
      expect(r.error.issues[0].message).toBe(COPY.validation.pinMismatch);
    }
  });
});
```

`src/features/points/pointsMath.test.ts`:

```ts
import { pointsFor } from './pointsMath';

describe('pointsFor', () => {
  it('350.000đ được 35 điểm', () => {
    expect(pointsFor(350000, 1)).toBe(35);
  });
  it('làm tròn xuống theo mỗi 10.000đ', () => {
    expect(pointsFor(359999, 1)).toBe(35);
    expect(pointsFor(9999, 1)).toBe(0);
  });
  it('nhân theo hệ số của tiệm', () => {
    expect(pointsFor(350000, 2)).toBe(70);
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/features`
Expected: FAIL (không tìm thấy các module `schemas`, `pointsMath`).

- [ ] **Step 3: Viết code**

`src/features/booking/schemas.ts`:

```ts
import { z } from 'zod';
import { BOOKING_UI } from '@/config/booking';
import { COPY } from '@/config/copy';
import { isValidPhone, normalizePhone } from '@/lib/phone';

// Người dùng gõ tự do (có dấu cách, +84); gọi normalizePhone() trước khi gửi lên server.
export const phoneSchema = z
  .string()
  .refine((value) => isValidPhone(normalizePhone(value)), COPY.validation.phone);

export const contactSchema = z.object({
  full_name: z.string().trim().min(2, COPY.validation.name),
  phone: phoneSchema,
  note: z.string().max(BOOKING_UI.noteMaxLength, COPY.validation.noteTooLong).optional(),
  coupon: z.string().trim().optional(),
});

export type ContactForm = z.infer<typeof contactSchema>;
```

`src/features/auth/schemas.ts`:

```ts
import { z } from 'zod';
import { COPY } from '@/config/copy';
import { phoneSchema } from '@/features/booking/schemas';

export const pinSchema = z.string().regex(/^\d{6}$/, COPY.validation.pin);

export const loginSchema = z.object({ phone: phoneSchema, pin: pinSchema });

export const registerSchema = loginSchema
  .extend({
    full_name: z.string().trim().min(2, COPY.validation.name),
    pin_confirm: z.string(),
  })
  .refine((data) => data.pin === data.pin_confirm, {
    message: COPY.validation.pinMismatch,
    path: ['pin_confirm'],
  });

export type LoginForm = z.infer<typeof loginSchema>;
export type RegisterForm = z.infer<typeof registerSchema>;
```

`src/features/points/pointsMath.ts`:

```ts
// Chỉ để xem trước "bạn sẽ nhận N điểm". Số điểm thật do CSDL tính khi lịch hoàn thành
// (hàm set_booking_status); giữ hai công thức giống nhau.
export const pointsFor = (totalVnd: number, per10k: number) =>
  Math.floor(totalVnd / 10000) * per10k;
```

- [ ] **Step 4: Chạy để thấy qua**

Run: `npm run test`
Expected: PASS toàn bộ (config, lib, features, smoke).

- [ ] **Step 5: Commit**

```bash
git add src/features
git commit -m "feat: schema kiểm tra form (đặt lịch, đăng nhập, đăng ký) và công thức điểm"
```

---

## Task 6: Công cụ CSDL, bảng và ràng buộc chống trùng giờ

**Files:**
- Create: `scripts/db.mjs`, `supabase/migrations/0001_schema.sql`, `supabase/tests/_fixtures.sql`, `supabase/tests/001_schema.test.sql`

**Interfaces:**
- Consumes: biến `DATABASE_URL` trong `.env.local`.
- Produces: lệnh `npm run db:migrate | db:seed | db:test`; các bảng trong spec mục 6 (không có cột `name` ở `shop_settings`); ràng buộc `no_double_booking`; hàm giúp test `pg_temp.expect_error(sql text, expected text)` và `pg_temp.ins(code, staff, start, end, status)`.

**Điều kiện trước:** tạo một project Supabase **thử nghiệm** (vùng Singapore). Vào Project Settings → Database → Connection string (Session pooler), sao chép vào `DATABASE_URL` trong `.env.local` (tạo từ `.env.example`). Không dán chuỗi này vào chat hay commit.

- [ ] **Step 1: Viết script chạy SQL**

`scripts/db.mjs`:

```js
import { readdir, readFile } from 'node:fs/promises';
import path from 'node:path';
import pg from 'pg';

const command = process.argv[2];
const url = process.env.DATABASE_URL;
if (!url) {
  console.error('Thiếu DATABASE_URL trong .env.local (chỉ dùng project Supabase THỬ NGHIỆM).');
  process.exit(1);
}

const inDir = (...parts) => path.resolve('supabase', ...parts);
const read = (...parts) => readFile(inDir(...parts), 'utf8');
const sqlFiles = async (dir) => (await readdir(inDir(dir))).filter((f) => f.endsWith('.sql')).sort();

// shortcut: tắt kiểm tra chứng chỉ để kết nối pooler của Supabase dễ dàng, chỉ dùng cho project thử nghiệm.
const client = new pg.Client({ connectionString: url, ssl: { rejectUnauthorized: false } });

async function migrate() {
  await client.query(
    'create table if not exists _migrations (name text primary key, applied_at timestamptz not null default now())',
  );
  const done = new Set((await client.query('select name from _migrations')).rows.map((r) => r.name));
  for (const file of (await sqlFiles('migrations')).filter((f) => !f.startsWith('_'))) {
    if (done.has(file)) continue;
    await client.query('begin');
    try {
      await client.query(await read('migrations', file));
      await client.query('insert into _migrations (name) values ($1)', [file]);
      await client.query('commit');
      console.log(`đã chạy ${file}`);
    } catch (error) {
      await client.query('rollback');
      throw new Error(`${file}: ${error.message}`);
    }
  }
}

async function seed() {
  await client.query(await read('seed.sql'));
  console.log('đã nạp dữ liệu mẫu');
}

// Mỗi tệp test chạy trong một giao dịch và luôn được rollback, nên không làm bẩn dữ liệu.
async function runTests() {
  const fixtures = await read('tests', '_fixtures.sql');
  let failed = 0;
  for (const file of (await sqlFiles('tests')).filter((f) => f.endsWith('.test.sql'))) {
    await client.query('begin');
    try {
      await client.query(fixtures + '\n' + (await read('tests', file)));
      console.log(`PASS ${file}`);
    } catch (error) {
      failed += 1;
      console.log(`FAIL ${file}: ${error.message}`);
    } finally {
      await client.query('rollback');
    }
  }
  if (failed) process.exitCode = 1;
}

const commands = { migrate, seed, test: runTests };
if (!commands[command]) {
  console.error('Dùng: migrate | seed | test');
  process.exit(1);
}
await client.connect();
try {
  await commands[command]();
} finally {
  await client.end();
}
```

- [ ] **Step 2: Viết dữ liệu giả cho test (fixtures)**

`supabase/tests/_fixtures.sql`:

```sql
-- Chạy đầu mỗi tệp test, trong cùng giao dịch (sẽ rollback). Xóa sạch dữ liệu để test độc lập.
truncate bookings, points_ledger, reminders, customers, services, service_categories,
  staff_schedules, staff, opening_hours, coupons, shop_settings, gallery_items
  restart identity cascade;

insert into shop_settings (address, phone) values ('123 Đường Mẫu, Quận 1', '0901234567');

insert into opening_hours (weekday, open_time, close_time)
  select d, '09:00', '20:00' from generate_series(0, 6) d;

insert into staff (id, name, sort) values
  ('00000000-0000-0000-0000-00000000a001', 'Mai', 1),
  ('00000000-0000-0000-0000-00000000a002', 'Linh', 2);

insert into staff_schedules (staff_id, weekday, start_time, end_time)
  select s.id, d, '09:00', '20:00' from staff s, generate_series(0, 6) d;

insert into service_categories (id, name, slug)
  values ('00000000-0000-0000-0000-00000000ca01', 'Gel', 'gel');

insert into services (id, category_id, name, price_vnd, duration_min)
  values ('00000000-0000-0000-0000-000000005e01', '00000000-0000-0000-0000-00000000ca01', 'Gel tay', 350000, 60);

insert into customers (id, full_name, phone)
  values ('00000000-0000-0000-0000-00000000c001', 'Khách thử', '0911111111');

insert into coupons (code, percent, starts_at, ends_at)
  values ('TEST10', 10, now() - interval '1 day', now() + interval '30 days');

-- Chèn nhanh một lịch cho khách thử, dịch vụ Gel tay.
create function pg_temp.ins(p_code text, p_staff uuid, p_start text, p_end text,
  p_status booking_status default 'confirmed') returns void language sql as $$
  insert into bookings (code, customer_id, service_id, staff_id, start_at, end_at, status, price_vnd, total_vnd)
  values (p_code, '00000000-0000-0000-0000-00000000c001', '00000000-0000-0000-0000-000000005e01',
    p_staff, p_start::timestamptz, p_end::timestamptz, p_status, 350000, 350000);
$$;

-- Khẳng định một câu lệnh phải lỗi đúng thông điệp mong đợi.
create function pg_temp.expect_error(p_sql text, p_expected text) returns void language plpgsql as $$
begin
  execute p_sql;
  raise exception 'FAIL: mong đợi lỗi "%" nhưng không có lỗi nào', p_expected;
exception when others then
  if sqlerrm <> p_expected then
    raise exception 'FAIL: mong đợi "%", nhận được "%"', p_expected, sqlerrm;
  end if;
end $$;
```

- [ ] **Step 2b: Viết test chống trùng giờ (hỏng vì chưa có bảng)**

`supabase/tests/001_schema.test.sql`:

```sql
-- Chống trùng giờ: cùng thợ, khoảng giờ giao nhau, lịch còn hiệu lực.
select pg_temp.ins('AAAAAA', '00000000-0000-0000-0000-00000000a001', '2030-03-04 10:00+07', '2030-03-04 11:00+07');

-- chồng lên lịch AAAAAA thì bị từ chối
select pg_temp.expect_error(
  $q$select pg_temp.ins('AAAAAB', '00000000-0000-0000-0000-00000000a001', '2030-03-04 10:30+07', '2030-03-04 11:30+07')$q$,
  'conflicting key value violates exclusion constraint "no_double_booking"');

-- kề nhau (kết thúc đúng lúc bắt đầu) thì được
select pg_temp.ins('AAAAAC', '00000000-0000-0000-0000-00000000a001', '2030-03-04 11:00+07', '2030-03-04 12:00+07');

-- thợ khác cùng giờ thì được
select pg_temp.ins('AAAAAD', '00000000-0000-0000-0000-00000000a002', '2030-03-04 10:00+07', '2030-03-04 11:00+07');

-- lịch đã hủy không chiếm giờ
update bookings set status = 'cancelled' where code = 'AAAAAA';
select pg_temp.ins('AAAAAE', '00000000-0000-0000-0000-00000000a001', '2030-03-04 10:00+07', '2030-03-04 11:00+07');

do $$ begin
  assert (select count(*) from bookings) = 5, 'phải có đúng 5 lịch sau các thao tác hợp lệ';
end $$;
```

- [ ] **Step 3: Chạy để thấy hỏng**

Run: `npm run db:test`
Expected: `FAIL 001_schema.test.sql: ... relation "shop_settings" does not exist` (chưa có bảng). Lệnh thoát với mã lỗi.

- [ ] **Step 4: Viết migration bảng**

`supabase/migrations/0001_schema.sql`:

```sql
create extension if not exists btree_gist;

create type booking_status as enum ('pending', 'confirmed', 'completed', 'cancelled');
create type user_role as enum ('customer', 'admin');

-- weekday: 0 = Chủ nhật ... 6 = Thứ bảy (khớp extract(dow) của Postgres và getUTCDay của JS)

create table profiles (
  id uuid primary key references auth.users (id) on delete cascade,
  role user_role not null default 'customer',
  full_name text not null,
  phone text not null unique check (phone ~ '^0[0-9]{9}$'),
  created_at timestamptz not null default now()
);

create table customers (
  id uuid primary key default gen_random_uuid(),
  profile_id uuid unique references profiles (id) on delete set null,
  full_name text not null,
  phone text not null unique check (phone ~ '^0[0-9]{9}$'),
  points_balance integer not null default 0 check (points_balance >= 0),
  admin_note text,
  created_at timestamptz not null default now()
);

create table staff (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  bio text,
  avatar_url text,
  active boolean not null default true,
  sort integer not null default 0
);

create table service_categories (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  slug text not null unique,
  sort integer not null default 0
);

create table services (
  id uuid primary key default gen_random_uuid(),
  category_id uuid not null references service_categories (id),
  name text not null,
  description text,
  price_vnd integer not null check (price_vnd > 0),
  duration_min integer not null check (duration_min > 0),
  image_url text,
  active boolean not null default true,
  sort integer not null default 0
);

create table opening_hours (
  weekday smallint primary key check (weekday between 0 and 6),
  open_time time not null,
  close_time time not null,
  is_closed boolean not null default false,
  check (close_time > open_time)
);

create table staff_schedules (
  staff_id uuid not null references staff (id) on delete cascade,
  weekday smallint not null check (weekday between 0 and 6),
  start_time time not null,
  end_time time not null,
  is_off boolean not null default false,
  primary key (staff_id, weekday),
  check (end_time > start_time)
);

create table coupons (
  id uuid primary key default gen_random_uuid(),
  code text not null unique check (code ~ '^[A-Z0-9]{3,20}$'),
  percent smallint not null check (percent between 1 and 100),
  min_total_vnd integer not null default 0 check (min_total_vnd >= 0),
  starts_at timestamptz not null,
  ends_at timestamptz not null,
  max_uses integer check (max_uses is null or max_uses > 0),
  used_count integer not null default 0,
  active boolean not null default true,
  check (ends_at > starts_at)
);

create table bookings (
  id uuid primary key default gen_random_uuid(),
  code text not null unique,
  customer_id uuid not null references customers (id),
  service_id uuid not null references services (id),
  staff_id uuid not null references staff (id),
  start_at timestamptz not null,
  end_at timestamptz not null,
  status booking_status not null default 'pending',
  price_vnd integer not null,
  discount_vnd integer not null default 0,
  total_vnd integer not null,
  coupon_id uuid references coupons (id),
  note text,
  created_at timestamptz not null default now(),
  confirmed_at timestamptz,
  completed_at timestamptz,
  cancelled_at timestamptz,
  cancel_reason text,
  check (end_at > start_at),
  constraint no_double_booking exclude using gist (
    staff_id with =,
    tstzrange(start_at, end_at, '[)') with &&
  ) where (status in ('pending', 'confirmed'))
);
create index bookings_start_idx on bookings (start_at);
create index bookings_customer_idx on bookings (customer_id);

create table points_ledger (
  id uuid primary key default gen_random_uuid(),
  customer_id uuid not null references customers (id) on delete cascade,
  booking_id uuid references bookings (id) on delete set null,
  delta integer not null check (delta <> 0),
  reason text not null,
  created_at timestamptz not null default now()
);

create table reminders (
  id uuid primary key default gen_random_uuid(),
  booking_id uuid not null references bookings (id) on delete cascade,
  remind_at timestamptz not null,
  channel text not null default 'in_app',
  sent_at timestamptz
);

create table gallery_items (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  style_tag text not null,
  image_url text,
  price_hint_vnd integer,
  featured boolean not null default false,
  active boolean not null default true,
  sort integer not null default 0
);

-- Chỉ một dòng. Tên thương hiệu nằm ở src/config/site.ts, không ở đây.
create table shop_settings (
  id boolean primary key default true check (id),
  address text not null,
  phone text not null,
  zalo text,
  map_url text,
  slot_minutes integer not null default 30 check (slot_minutes in (15, 30, 60)),
  lead_minutes integer not null default 60 check (lead_minutes >= 0),
  cancel_hours integer not null default 2 check (cancel_hours >= 0),
  points_per_10k integer not null default 1 check (points_per_10k >= 0)
);
```

- [ ] **Step 5: Chạy migration rồi test**

Run: `npm run db:migrate`
Expected: `đã chạy 0001_schema.sql`.

Run: `npm run db:test`
Expected: `PASS 001_schema.test.sql`.

- [ ] **Step 6: Commit**

```bash
git add scripts supabase
git commit -m "feat(db): bảng, ràng buộc chống trùng giờ và công cụ chạy SQL"
```

---

## Task 7: Hàm RPC, phân quyền RLS và test

**Files:**
- Create: `supabase/migrations/0002_functions.sql`, `supabase/migrations/0003_rls.sql`, `supabase/tests/010_functions.test.sql`, `supabase/tests/020_rls.test.sql`

**Interfaces:**
- Consumes: bảng của Task 6.
- Produces (RPC, trả `jsonb` khóa `snake_case`):
  - `get_busy_intervals(p_day date, p_staff uuid default null)` → bảng `(staff_id, start_at, end_at)`
  - `create_booking(p_service uuid, p_staff uuid, p_start timestamptz, p_name text, p_phone text, p_note text default null, p_coupon text default null)` → `{ code, start_at, end_at, staff_id, total_vnd, discount_vnd }`
  - `lookup_booking(p_phone text, p_code text)` → `{ code, status, start_at, end_at, service_name, staff_name, total_vnd, customer_name }` hoặc `null`
  - `cancel_booking(p_phone text, p_code text)` → `{ code, status }`
  - `validate_coupon(p_code text, p_total integer)` → `{ valid, discount_vnd?, percent?, reason?, min_total_vnd?, ends_at? }`
  - `set_booking_status(p_id uuid, p_status booking_status, p_reason text default null)` (admin) → `{ id, status }`
  - `adjust_points(p_customer uuid, p_delta integer, p_reason text)` (admin)
  - `register_customer(p_name text, p_phone text)` (đã đăng nhập)
  - `is_admin()` → boolean
- Mã lỗi (`sqlerrm`): `bad_phone`, `bad_name`, `service_not_found`, `slot_past`, `slot_invalid`, `slot_taken`, `coupon_invalid`, `booking_not_found`, `cancel_not_allowed`, `cancel_too_late`, `forbidden`, `bad_transition`, `bad_adjustment`, `customer_not_found`, `unauthenticated`, `phone_taken`.

- [ ] **Step 1: Viết test hỏng cho hàm RPC**

`supabase/tests/010_functions.test.sql`:

```sql
-- đặt lịch thành công, mã lịch 6 ký tự, tổng tiền = giá dịch vụ
do $$
declare r jsonb;
begin
  r := create_booking('00000000-0000-0000-0000-000000005e01', '00000000-0000-0000-0000-00000000a001',
    '2030-03-04 10:00+07', 'An', '0901234567');
  assert length(r->>'code') = 6, 'mã lịch phải có 6 ký tự';
  assert (r->>'total_vnd')::int = 350000, 'tổng tiền phải là 350000';
  assert (select count(*) from bookings where status = 'pending') = 1, 'lịch mới ở trạng thái pending';
end $$;

-- thợ "bất kỳ": hai khách cùng giờ nhận hai thợ khác nhau, người thứ ba hết chỗ
do $$
declare r1 jsonb; r2 jsonb;
begin
  r1 := create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-05 10:00+07', 'B', '0902222222');
  r2 := create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-05 10:00+07', 'C', '0903333333');
  assert r1->>'staff_id' <> r2->>'staff_id', 'hai khách cùng giờ phải nhận hai thợ khác nhau';
end $$;
select pg_temp.expect_error(
  $q$select create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-05 10:00+07', 'D', '0904444444')$q$,
  'slot_taken');

-- giờ đã qua, ngoài giờ mở cửa, lệch lưới
select pg_temp.expect_error(
  $q$select create_booking('00000000-0000-0000-0000-000000005e01', null, now(), 'E', '0905555555')$q$,
  'slot_past');
select pg_temp.expect_error(
  $q$select create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-06 21:00+07', 'E', '0905555555')$q$,
  'slot_invalid');
select pg_temp.expect_error(
  $q$select create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-06 10:15+07', 'E', '0905555555')$q$,
  'slot_invalid');

-- đúng giờ đóng cửa: 19:00 + 60 phút = 20:00 hợp lệ, 19:30 thì không
select create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-06 19:00+07', 'F', '0906666666');
select pg_temp.expect_error(
  $q$select create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-06 19:30+07', 'G', '0907777777')$q$,
  'slot_invalid');

-- dữ liệu vào sai
select pg_temp.expect_error(
  $q$select create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-07 10:00+07', 'H', '12345')$q$,
  'bad_phone');
select pg_temp.expect_error(
  $q$select create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-07 10:00+07', 'H', '0908888888', 'x', 'KHONGCO')$q$,
  'coupon_invalid');

-- mã giảm giá: giảm 10%, tăng used_count
do $$
declare r jsonb;
begin
  r := create_booking('00000000-0000-0000-0000-000000005e01', null, '2030-03-07 10:00+07', 'I', '0909999999', null, 'test10');
  assert (r->>'discount_vnd')::int = 35000, 'giảm 10% của 350000 là 35000';
  assert (r->>'total_vnd')::int = 315000, 'tổng sau giảm là 315000';
  assert (select used_count from coupons where code = 'TEST10') = 1, 'used_count phải tăng lên 1';
  assert (validate_coupon('TEST10', 350000)->>'valid')::boolean, 'mã còn dùng được';
  assert validate_coupon('NOPE', 350000)->>'reason' = 'not_found', 'mã lạ trả not_found';
end $$;

-- tra cứu và hủy
do $$
declare r jsonb; l jsonb;
begin
  r := create_booking('00000000-0000-0000-0000-000000005e01', '00000000-0000-0000-0000-00000000a002',
    '2030-03-08 10:00+07', 'K', '0910000001');
  l := lookup_booking('0910000001', r->>'code');
  assert l->>'service_name' = 'Gel tay', 'tra cứu đúng dịch vụ';
  assert lookup_booking('0910000002', r->>'code') is null, 'sai SĐT không thấy lịch';
  perform cancel_booking('0910000001', r->>'code');
  assert (select status from bookings where code = r->>'code') = 'cancelled', 'đã hủy';
  -- giờ được nhả cho người khác
  perform create_booking('00000000-0000-0000-0000-000000005e01', '00000000-0000-0000-0000-00000000a002',
    '2030-03-08 10:00+07', 'L', '0910000003');
end $$;

-- hủy quá hạn: lịch bắt đầu sau 1 giờ nữa (chèn trực tiếp, bỏ qua kiểm tra giờ đệm)
select pg_temp.ins('LATE01', '00000000-0000-0000-0000-00000000a002', (now() + interval '1 hour')::text,
  (now() + interval '2 hours')::text);
select pg_temp.expect_error($q$select cancel_booking('0911111111', 'LATE01')$q$, 'cancel_too_late');
select pg_temp.expect_error($q$select cancel_booking('0911111111', 'NOPE99')$q$, 'booking_not_found');

-- hoạt động của admin: tạo tài khoản admin và khách
insert into auth.users (id, email) values
  ('00000000-0000-0000-0000-00000000ad01', 'admin@test.local'),
  ('00000000-0000-0000-0000-00000000bb01', 'khach@test.local');
insert into profiles (id, role, full_name, phone) values
  ('00000000-0000-0000-0000-00000000ad01', 'admin', 'Admin', '0900000001'),
  ('00000000-0000-0000-0000-00000000bb01', 'customer', 'Khách', '0900000002');

select pg_temp.ins('DONE01', '00000000-0000-0000-0000-00000000a001', '2030-03-09 10:00+07', '2030-03-09 11:00+07');

-- khách thường không được đổi trạng thái
select set_config('request.jwt.claims', '{"sub":"00000000-0000-0000-0000-00000000bb01","role":"authenticated"}', true);
set local role authenticated;
do $$ begin
  begin
    perform set_booking_status((select id from bookings where code = 'DONE01'), 'completed');
    raise exception 'FAIL: khách thường không được hoàn thành lịch';
  exception when others then
    if sqlerrm <> 'forbidden' then raise exception 'FAIL: mong đợi forbidden, nhận %', sqlerrm; end if;
  end;
end $$;
reset role;

-- admin hoàn thành lịch: cộng 35 điểm một lần, lần hai bị từ chối
select set_config('request.jwt.claims', '{"sub":"00000000-0000-0000-0000-00000000ad01","role":"authenticated"}', true);
set local role authenticated;
select set_booking_status((select id from bookings where code = 'DONE01'), 'completed');
do $$ begin
  begin
    perform set_booking_status((select id from bookings where code = 'DONE01'), 'completed');
    raise exception 'FAIL: hoàn thành hai lần phải bị từ chối';
  exception when others then
    if sqlerrm <> 'bad_transition' then raise exception 'FAIL: mong đợi bad_transition, nhận %', sqlerrm; end if;
  end;
end $$;
select adjust_points('00000000-0000-0000-0000-00000000c001', 5, 'Tặng điểm thử');
reset role;

do $$ begin
  assert (select points_balance from customers where id = '00000000-0000-0000-0000-00000000c001') = 40,
    'điểm = 35 (hoàn thành) + 5 (chỉnh tay)';
  assert (select count(*) from points_ledger where booking_id is not null) = 1, 'chỉ một dòng sổ điểm cho lịch';
end $$;
```

- [ ] **Step 2: Viết test hỏng cho RLS**

`supabase/tests/020_rls.test.sql`:

```sql
insert into auth.users (id, email) values
  ('00000000-0000-0000-0000-00000000ad01', 'admin@test.local'),
  ('00000000-0000-0000-0000-00000000bb01', 'khach@test.local');
insert into profiles (id, role, full_name, phone) values
  ('00000000-0000-0000-0000-00000000ad01', 'admin', 'Admin', '0900000001'),
  ('00000000-0000-0000-0000-00000000bb01', 'customer', 'Khách', '0911111111');
update customers set profile_id = '00000000-0000-0000-0000-00000000bb01'
  where id = '00000000-0000-0000-0000-00000000c001';
insert into customers (id, full_name, phone) values ('00000000-0000-0000-0000-00000000c002', 'Người lạ', '0922222222');

select pg_temp.ins('OWN001', '00000000-0000-0000-0000-00000000a001', '2030-03-04 10:00+07', '2030-03-04 11:00+07');
insert into bookings (code, customer_id, service_id, staff_id, start_at, end_at, price_vnd, total_vnd)
  values ('OTH001', '00000000-0000-0000-0000-00000000c002', '00000000-0000-0000-0000-000000005e01',
    '00000000-0000-0000-0000-00000000a002', '2030-03-04 10:00+07', '2030-03-04 11:00+07', 350000, 350000);
insert into services (category_id, name, price_vnd, duration_min, active)
  values ('00000000-0000-0000-0000-00000000ca01', 'Dịch vụ ẩn', 100000, 30, false);

-- khách chưa đăng nhập: thấy dịch vụ đang bật, không thấy lịch, khách, mã giảm giá
set local role anon;
do $$ begin
  assert (select count(*) from services) = 1, 'anon chỉ thấy dịch vụ đang bật';
  assert (select count(*) from bookings) = 0, 'anon không đọc được lịch';
  assert (select count(*) from customers) = 0, 'anon không đọc được khách';
  assert (select count(*) from coupons) = 0, 'anon không đọc được mã giảm giá';
  begin
    perform set_booking_status('00000000-0000-0000-0000-000000000000', 'confirmed');
    raise exception 'FAIL: anon không được gọi hàm admin';
  exception when insufficient_privilege then null;
  end;
end $$;
reset role;

-- khách đã đăng nhập: chỉ thấy lịch và dữ liệu của mình, không sửa được danh mục
select set_config('request.jwt.claims', '{"sub":"00000000-0000-0000-0000-00000000bb01","role":"authenticated"}', true);
set local role authenticated;
do $$ begin
  assert (select count(*) from bookings) = 1, 'khách chỉ thấy 1 lịch của mình';
  assert (select count(*) from customers) = 1, 'khách chỉ thấy hồ sơ của mình';
  begin
    insert into services (category_id, name, price_vnd, duration_min)
      values ('00000000-0000-0000-0000-00000000ca01', 'Chui vào', 1, 30);
    raise exception 'FAIL: khách không được thêm dịch vụ';
  exception when insufficient_privilege then null;
  end;
end $$;
reset role;

-- admin: thấy tất cả
select set_config('request.jwt.claims', '{"sub":"00000000-0000-0000-0000-00000000ad01","role":"authenticated"}', true);
set local role authenticated;
do $$ begin
  assert (select count(*) from bookings) = 2, 'admin thấy cả 2 lịch';
  assert (select count(*) from services) = 2, 'admin thấy cả dịch vụ ẩn';
end $$;
reset role;
```

- [ ] **Step 3: Chạy để thấy hỏng**

Run: `npm run db:test`
Expected: `FAIL 010_functions.test.sql: function create_booking(...) does not exist` và `FAIL 020_rls.test.sql` (anon vẫn đọc được vì chưa bật RLS). `PASS 001_schema.test.sql`.

- [ ] **Step 4: Viết các hàm RPC**

`supabase/migrations/0002_functions.sql`:

```sql
-- Đầu ngày (0:00 giờ Việt Nam) của một ngày.
create or replace function vn_day_start(p_day date) returns timestamptz
language sql immutable as $$ select p_day::timestamp at time zone 'Asia/Ho_Chi_Minh' $$;

create or replace function is_admin() returns boolean
language sql stable security definer set search_path = public as $$
  select exists (select 1 from profiles where id = auth.uid() and role = 'admin');
$$;

-- Mã lịch 6 ký tự, bỏ các ký tự dễ nhầm (0, O, 1, I, L).
create or replace function gen_booking_code() returns text language plpgsql as $$
declare
  chars constant text := 'ABCDEFGHJKMNPQRSTUVWXYZ23456789';
  v_code text;
  i int;
begin
  loop
    v_code := '';
    for i in 1..6 loop
      v_code := v_code || substr(chars, 1 + floor(random() * length(chars))::int, 1);
    end loop;
    exit when not exists (select 1 from bookings where bookings.code = v_code);
  end loop;
  return v_code;
end $$;

-- Các khoảng đã bận của một ngày, không lộ thông tin khách.
create or replace function get_busy_intervals(p_day date, p_staff uuid default null)
returns table (staff_id uuid, start_at timestamptz, end_at timestamptz)
language sql stable security definer set search_path = public as $$
  select b.staff_id, b.start_at, b.end_at
  from bookings b
  where b.status in ('pending', 'confirmed')
    and b.start_at >= vn_day_start(p_day)
    and b.start_at < vn_day_start(p_day + 1)
    and (p_staff is null or b.staff_id = p_staff);
$$;

create or replace function create_booking(
  p_service uuid, p_staff uuid, p_start timestamptz,
  p_name text, p_phone text, p_note text default null, p_coupon text default null
) returns jsonb language plpgsql security definer set search_path = public as $$
declare
  s services;
  cfg shop_settings;
  c coupons;
  v_open opening_hours;
  v_booking bookings;
  v_end timestamptz;
  v_local timestamp;
  v_end_local timestamp;
  v_dow int;
  v_min int;
  v_discount int := 0;
  v_coupon_id uuid;
  v_customer uuid;
  v_tried boolean := false;
  cand record;
begin
  if p_phone !~ '^0[0-9]{9}$' then raise exception 'bad_phone'; end if;
  if length(trim(coalesce(p_name, ''))) < 2 then raise exception 'bad_name'; end if;

  select * into s from services where id = p_service and active;
  if not found then raise exception 'service_not_found'; end if;
  select * into cfg from shop_settings limit 1;

  v_end := p_start + make_interval(mins => s.duration_min);
  if p_start < now() + make_interval(mins => cfg.lead_minutes) then raise exception 'slot_past'; end if;

  v_local := p_start at time zone 'Asia/Ho_Chi_Minh';
  v_end_local := v_end at time zone 'Asia/Ho_Chi_Minh';
  v_dow := extract(dow from v_local)::int;
  v_min := extract(hour from v_local)::int * 60 + extract(minute from v_local)::int;
  if v_min % cfg.slot_minutes <> 0 or v_end_local::date <> v_local::date then
    raise exception 'slot_invalid';
  end if;

  select * into v_open from opening_hours where weekday = v_dow;
  if not found or v_open.is_closed
     or v_local::time < v_open.open_time or v_end_local::time > v_open.close_time then
    raise exception 'slot_invalid';
  end if;

  if p_coupon is not null and length(trim(p_coupon)) > 0 then
    select * into c from coupons
      where code = upper(trim(p_coupon)) and active
        and now() between starts_at and ends_at
        and s.price_vnd >= min_total_vnd
        and (max_uses is null or used_count < max_uses);
    if not found then raise exception 'coupon_invalid'; end if;
    v_discount := floor(s.price_vnd * c.percent / 100.0)::int;
    v_coupon_id := c.id;
  end if;

  insert into customers (full_name, phone) values (trim(p_name), p_phone)
    on conflict (phone) do update set phone = excluded.phone
    returning id into v_customer;

  -- Thợ rảnh có ít lịch nhất trong ngày được thử trước; hòa thì theo thứ tự sort.
  for cand in
    select st.id
    from staff st
    join staff_schedules sc on sc.staff_id = st.id and sc.weekday = v_dow and not sc.is_off
    where st.active
      and (p_staff is null or st.id = p_staff)
      and v_local::time >= sc.start_time and v_end_local::time <= sc.end_time
    order by (
      select count(*) from bookings b
      where b.staff_id = st.id and b.status in ('pending', 'confirmed')
        and b.start_at >= vn_day_start(v_local::date) and b.start_at < vn_day_start(v_local::date + 1)
    ), st.sort
  loop
    v_tried := true;
    begin
      insert into bookings (code, customer_id, service_id, staff_id, start_at, end_at,
                            price_vnd, discount_vnd, total_vnd, coupon_id, note)
      values (gen_booking_code(), v_customer, p_service, cand.id, p_start, v_end,
              s.price_vnd, v_discount, s.price_vnd - v_discount, v_coupon_id, nullif(trim(p_note), ''))
      returning * into v_booking;
      exit;
    exception when exclusion_violation then
      null; -- thợ này vừa có người đặt, thử thợ kế tiếp
    end;
  end loop;

  if v_booking.id is null then
    raise exception '%', case when v_tried then 'slot_taken' else 'slot_invalid' end;
  end if;

  if v_coupon_id is not null then
    update coupons set used_count = used_count + 1
      where id = v_coupon_id and (max_uses is null or used_count < max_uses);
    if not found then raise exception 'coupon_invalid'; end if;
  end if;

  return jsonb_build_object(
    'code', v_booking.code, 'start_at', v_booking.start_at, 'end_at', v_booking.end_at,
    'staff_id', v_booking.staff_id, 'total_vnd', v_booking.total_vnd, 'discount_vnd', v_booking.discount_vnd);
end $$;

create or replace function lookup_booking(p_phone text, p_code text) returns jsonb
language sql stable security definer set search_path = public as $$
  select jsonb_build_object(
    'code', b.code, 'status', b.status, 'start_at', b.start_at, 'end_at', b.end_at,
    'service_name', sv.name, 'staff_name', st.name, 'total_vnd', b.total_vnd,
    'customer_name', c.full_name)
  from bookings b
  join customers c on c.id = b.customer_id
  join services sv on sv.id = b.service_id
  join staff st on st.id = b.staff_id
  where b.code = upper(trim(p_code)) and c.phone = p_phone;
$$;

create or replace function cancel_booking(p_phone text, p_code text) returns jsonb
language plpgsql security definer set search_path = public as $$
declare
  b bookings;
  cfg shop_settings;
begin
  select bk.* into b from bookings bk join customers c on c.id = bk.customer_id
    where bk.code = upper(trim(p_code)) and c.phone = p_phone;
  if not found then raise exception 'booking_not_found'; end if;
  if b.status not in ('pending', 'confirmed') then raise exception 'cancel_not_allowed'; end if;
  select * into cfg from shop_settings limit 1;
  if b.start_at - make_interval(hours => cfg.cancel_hours) < now() then
    raise exception 'cancel_too_late';
  end if;
  update bookings set status = 'cancelled', cancelled_at = now(), cancel_reason = 'Khách hủy'
    where id = b.id;
  return jsonb_build_object('code', b.code, 'status', 'cancelled');
end $$;

create or replace function validate_coupon(p_code text, p_total integer) returns jsonb
language plpgsql stable security definer set search_path = public as $$
declare c coupons;
begin
  select * into c from coupons where code = upper(trim(p_code));
  if not found then return jsonb_build_object('valid', false, 'reason', 'not_found'); end if;
  if not c.active or now() < c.starts_at then
    return jsonb_build_object('valid', false, 'reason', 'inactive');
  end if;
  if now() > c.ends_at then
    return jsonb_build_object('valid', false, 'reason', 'expired', 'ends_at', c.ends_at);
  end if;
  if c.max_uses is not null and c.used_count >= c.max_uses then
    return jsonb_build_object('valid', false, 'reason', 'used_up');
  end if;
  if p_total < c.min_total_vnd then
    return jsonb_build_object('valid', false, 'reason', 'min_total', 'min_total_vnd', c.min_total_vnd);
  end if;
  return jsonb_build_object('valid', true, 'percent', c.percent,
    'discount_vnd', floor(p_total * c.percent / 100.0)::int);
end $$;

-- Một hàm giữ cả bảng chuyển trạng thái và việc cộng điểm khi hoàn thành.
-- Cập nhật cùng lúc src/config/statuses.ts nếu đổi quy tắc.
create or replace function set_booking_status(p_id uuid, p_status booking_status, p_reason text default null)
returns jsonb language plpgsql security definer set search_path = public as $$
declare
  b bookings;
  cfg shop_settings;
  pts int;
begin
  if not is_admin() then raise exception 'forbidden'; end if;
  select * into b from bookings where id = p_id for update;
  if not found then raise exception 'booking_not_found'; end if;
  if not ((b.status = 'pending' and p_status in ('confirmed', 'cancelled'))
       or (b.status = 'confirmed' and p_status in ('completed', 'cancelled'))) then
    raise exception 'bad_transition';
  end if;

  update bookings set
    status = p_status,
    confirmed_at = case when p_status = 'confirmed' then now() else confirmed_at end,
    completed_at = case when p_status = 'completed' then now() else completed_at end,
    cancelled_at = case when p_status = 'cancelled' then now() else cancelled_at end,
    cancel_reason = case when p_status = 'cancelled' then p_reason else cancel_reason end
  where id = p_id;

  if p_status = 'completed' then
    select * into cfg from shop_settings limit 1;
    pts := (b.total_vnd / 10000) * cfg.points_per_10k;
    if pts > 0 then
      insert into points_ledger (customer_id, booking_id, delta, reason)
        values (b.customer_id, b.id, pts, 'Hoàn thành lịch ' || b.code);
      update customers set points_balance = points_balance + pts where id = b.customer_id;
    end if;
  end if;
  return jsonb_build_object('id', p_id, 'status', p_status);
end $$;

create or replace function adjust_points(p_customer uuid, p_delta integer, p_reason text) returns jsonb
language plpgsql security definer set search_path = public as $$
begin
  if not is_admin() then raise exception 'forbidden'; end if;
  if p_delta = 0 or length(trim(coalesce(p_reason, ''))) < 3 then raise exception 'bad_adjustment'; end if;
  update customers set points_balance = points_balance + p_delta where id = p_customer;
  if not found then raise exception 'customer_not_found'; end if;
  insert into points_ledger (customer_id, delta, reason) values (p_customer, p_delta, trim(p_reason));
  return jsonb_build_object('customer_id', p_customer);
end $$;

-- Gọi sau khi khách đăng ký tài khoản: tạo hồ sơ và gắn các lịch đặt vãng lai cùng SĐT.
create or replace function register_customer(p_name text, p_phone text) returns jsonb
language plpgsql security definer set search_path = public as $$
begin
  if auth.uid() is null then raise exception 'unauthenticated'; end if;
  if p_phone !~ '^0[0-9]{9}$' then raise exception 'bad_phone'; end if;
  insert into profiles (id, full_name, phone) values (auth.uid(), trim(p_name), p_phone)
    on conflict (id) do nothing;
  insert into customers (profile_id, full_name, phone) values (auth.uid(), trim(p_name), p_phone)
    on conflict (phone) do update set profile_id = excluded.profile_id
    where customers.profile_id is null or customers.profile_id = excluded.profile_id;
  if not exists (select 1 from customers where phone = p_phone and profile_id = auth.uid()) then
    raise exception 'phone_taken';
  end if;
  return jsonb_build_object('ok', true);
end $$;
```

- [ ] **Step 5: Viết RLS và quyền gọi hàm**

`supabase/migrations/0003_rls.sql`:

```sql
alter table profiles enable row level security;
alter table customers enable row level security;
alter table staff enable row level security;
alter table service_categories enable row level security;
alter table services enable row level security;
alter table opening_hours enable row level security;
alter table staff_schedules enable row level security;
alter table coupons enable row level security;
alter table bookings enable row level security;
alter table points_ledger enable row level security;
alter table reminders enable row level security;
alter table gallery_items enable row level security;
alter table shop_settings enable row level security;

-- Admin toàn quyền trên mọi bảng.
do $$
declare t text;
begin
  foreach t in array array['profiles', 'customers', 'staff', 'service_categories', 'services',
    'opening_hours', 'staff_schedules', 'coupons', 'bookings', 'points_ledger', 'reminders',
    'gallery_items', 'shop_settings']
  loop
    execute format(
      'create policy admin_all on %I for all to authenticated using (is_admin()) with check (is_admin())', t);
  end loop;
end $$;

-- Danh mục công khai, chỉ đọc (dòng đang bật).
create policy public_read on services for select using (active);
create policy public_read on staff for select using (active);
create policy public_read on gallery_items for select using (active);
create policy public_read on service_categories for select using (true);
create policy public_read on opening_hours for select using (true);
create policy public_read on staff_schedules for select using (true);
create policy public_read on shop_settings for select using (true);

-- Khách đã đăng nhập chỉ đọc dữ liệu của mình.
create policy own_read on profiles for select to authenticated using (id = auth.uid());
create policy own_read on customers for select to authenticated using (profile_id = auth.uid());
create policy own_read on bookings for select to authenticated
  using (customer_id in (select id from customers where profile_id = auth.uid()));
create policy own_read on points_ledger for select to authenticated
  using (customer_id in (select id from customers where profile_id = auth.uid()));
create policy own_read on reminders for select to authenticated
  using (booking_id in (
    select b.id from bookings b join customers c on c.id = b.customer_id where c.profile_id = auth.uid()));

-- Quyền gọi hàm: Supabase mặc định cho mọi vai trò gọi, nên thu hồi rồi cấp lại rõ ràng.
revoke all on function set_booking_status(uuid, booking_status, text) from public, anon;
revoke all on function adjust_points(uuid, integer, text) from public, anon;
revoke all on function register_customer(text, text) from public, anon;
grant execute on function set_booking_status(uuid, booking_status, text) to authenticated;
grant execute on function adjust_points(uuid, integer, text) to authenticated;
grant execute on function register_customer(text, text) to authenticated;

grant execute on function create_booking(uuid, uuid, timestamptz, text, text, text, text) to anon, authenticated;
grant execute on function lookup_booking(text, text) to anon, authenticated;
grant execute on function cancel_booking(text, text) to anon, authenticated;
grant execute on function validate_coupon(text, integer) to anon, authenticated;
grant execute on function get_busy_intervals(date, uuid) to anon, authenticated;
```

- [ ] **Step 6: Chạy migration rồi test**

Run: `npm run db:migrate`
Expected: `đã chạy 0002_functions.sql`, `đã chạy 0003_rls.sql`.

Run: `npm run db:test`
Expected: `PASS` cho cả 3 tệp. Nếu `insert into auth.users` báo thiếu cột bắt buộc, thêm các cột mà lỗi nêu (thường là `instance_id`, `aud`, `role`) vào cả hai tệp test, rồi chạy lại.

- [ ] **Step 7: Commit**

```bash
git add supabase
git commit -m "feat(db): hàm RPC đặt/tra cứu/hủy lịch, đổi trạng thái, RLS và test"
```

---

## Task 8: Dữ liệu mẫu, Supabase client và kiểu dữ liệu

**Files:**
- Create: `supabase/seed.sql`, `src/lib/supabase.ts`, `src/types/db.ts`
- Modify: `README.md` (thêm hướng dẫn tạo admin)

**Interfaces:**
- Produces: `supabase` (client), kiểu `Service`, `ServiceCategory`, `Staff`, `OpeningHours`, `StaffSchedule`, `GalleryItem`, `ShopSettings`, `Booking`, `Customer`, `Coupon`, `PointsLedgerRow` (tên trường giống CSDL).

- [ ] **Step 1: Viết dữ liệu mẫu**

`supabase/seed.sql` (chạy lại nhiều lần được):

```sql
insert into shop_settings (address, phone, zalo, map_url)
  values ('123 Đường Hoa Lan, Phường 2, Quận Phú Nhuận, TP.HCM', '0901234567', '0901234567',
          'https://maps.google.com/?q=123+Duong+Hoa+Lan+Phu+Nhuan')
  on conflict (id) do nothing;

insert into opening_hours (weekday, open_time, close_time)
  select d, '09:00', '20:00' from generate_series(0, 6) d on conflict (weekday) do nothing;

insert into staff (id, name, bio, sort) values
  ('00000000-0000-0000-0000-0000000a0001', 'Mai', 'Tay nghề 8 năm, giỏi nail art và đính đá.', 1),
  ('00000000-0000-0000-0000-0000000a0002', 'Linh', 'Chuyên gel và French tỉ mỉ.', 2),
  ('00000000-0000-0000-0000-0000000a0003', 'Trang', 'Chuyên đắp bột và chăm sóc chân.', 3)
  on conflict (id) do nothing;

-- Mai và Linh làm cả tuần (Linh nghỉ thứ Hai), Trang chỉ làm chiều.
insert into staff_schedules (staff_id, weekday, start_time, end_time, is_off)
  select s.id, d,
    case when s.name = 'Trang' then '12:00'::time else '09:00'::time end,
    '20:00'::time,
    (s.name = 'Linh' and d = 1)
  from staff s, generate_series(0, 6) d
  on conflict (staff_id, weekday) do nothing;

insert into service_categories (id, name, slug, sort) values
  ('00000000-0000-0000-0000-0000000c0001', 'Tay', 'tay', 1),
  ('00000000-0000-0000-0000-0000000c0002', 'Chân', 'chan', 2),
  ('00000000-0000-0000-0000-0000000c0003', 'Gel', 'gel', 3),
  ('00000000-0000-0000-0000-0000000c0004', 'Đắp bột', 'dap-bot', 4),
  ('00000000-0000-0000-0000-0000000c0005', 'Chăm sóc', 'cham-soc', 5)
  on conflict (id) do nothing;

insert into services (id, category_id, name, description, price_vnd, duration_min, sort) values
  ('00000000-0000-0000-0000-0000000e0001', '00000000-0000-0000-0000-0000000c0001', 'Sơn thường', 'Làm sạch móng và sơn một màu.', 120000, 30, 1),
  ('00000000-0000-0000-0000-0000000e0002', '00000000-0000-0000-0000-0000000c0001', 'Cắt da và dưỡng', 'Cắt da, tạo form, dưỡng ẩm.', 150000, 30, 2),
  ('00000000-0000-0000-0000-0000000e0003', '00000000-0000-0000-0000-0000000c0002', 'Sơn chân', 'Làm sạch và sơn móng chân.', 140000, 30, 1),
  ('00000000-0000-0000-0000-0000000e0004', '00000000-0000-0000-0000-0000000c0003', 'Gel tay', 'Sơn gel bền màu, bóng đẹp.', 350000, 60, 1),
  ('00000000-0000-0000-0000-0000000e0005', '00000000-0000-0000-0000-0000000c0003', 'Gel chân', 'Sơn gel móng chân.', 380000, 60, 2),
  ('00000000-0000-0000-0000-0000000e0006', '00000000-0000-0000-0000-0000000c0004', 'Đắp bột', 'Nối và đắp bột tạo form móng.', 450000, 90, 1),
  ('00000000-0000-0000-0000-0000000e0007', '00000000-0000-0000-0000-0000000c0005', 'Chăm sóc tay', 'Ngâm, tẩy da chết, mát-xa tay.', 200000, 45, 1),
  ('00000000-0000-0000-0000-0000000e0008', '00000000-0000-0000-0000-0000000c0005', 'Chăm sóc chân', 'Ngâm, tẩy da chết, mát-xa chân.', 220000, 45, 2)
  on conflict (id) do nothing;

delete from gallery_items;
insert into gallery_items (title, style_tag, price_hint_vnd, featured, sort) values
  ('Ombre hồng phấn', 'ombre', 380000, true, 1), ('Ombre tím khói', 'ombre', 380000, false, 2),
  ('French trắng cổ điển', 'french', 350000, true, 3), ('French đính nơ', 'french', 420000, false, 4),
  ('Đính đá lấp lánh', 'dinh-da', 480000, true, 5), ('Đá pha lê ánh tím', 'dinh-da', 520000, false, 6),
  ('Nổi 3D hoa nhí', '3d', 550000, false, 7), ('Nổi 3D giọt gel', '3d', 550000, true, 8),
  ('Mắt mèo xanh biển', 'mat-meo', 400000, false, 9), ('Mắt mèo hồng', 'mat-meo', 400000, true, 10),
  ('Ombre cam sữa', 'ombre', 380000, false, 11), ('French đỏ rượu', 'french', 360000, false, 12);

insert into coupons (code, percent, min_total_vnd, starts_at, ends_at, max_uses)
  values ('LUA15', 15, 200000, now() - interval '1 day', now() + interval '90 days', 50)
  on conflict (code) do nothing;

insert into customers (full_name, phone) values
  ('Nguyễn Thu Lan', '0911000001'), ('Trần Minh Anh', '0911000002'), ('Lê Hồng Nhung', '0911000003'),
  ('Phạm Bảo Châu', '0911000004'), ('Võ Ngọc Hân', '0911000005')
  on conflict (phone) do nothing;

-- Lịch mẫu rải 7 ngày quanh hôm nay để biểu đồ doanh thu có dữ liệu.
-- Cột đầu: mã, SĐT khách, dịch vụ, thợ (tên), lệch ngày, giờ, trạng thái.
delete from bookings where code like 'DEMO%';
insert into bookings (code, customer_id, service_id, staff_id, start_at, end_at, status,
                      price_vnd, total_vnd, completed_at)
select d.code, c.id, sv.id, st.id,
       start_at, start_at + make_interval(mins => sv.duration_min), d.status::booking_status,
       sv.price_vnd, sv.price_vnd,
       case when d.status = 'completed' then start_at end
from (values
  ('DEMO01', '0911000001', 'Gel tay',      'Mai',   -6, '10:00', 'completed'),
  ('DEMO02', '0911000002', 'Đắp bột',      'Trang', -5, '14:00', 'completed'),
  ('DEMO03', '0911000003', 'Gel chân',     'Linh',  -4, '11:00', 'completed'),
  ('DEMO04', '0911000004', 'Chăm sóc tay', 'Mai',   -3, '15:30', 'completed'),
  ('DEMO05', '0911000005', 'Gel tay',      'Linh',  -2, '09:30', 'completed'),
  ('DEMO06', '0911000001', 'Đắp bột',      'Mai',   -1, '13:00', 'completed'),
  ('DEMO07', '0911000002', 'Gel tay',      'Mai',    0, '10:00', 'confirmed'),
  ('DEMO08', '0911000003', 'Sơn thường',   'Linh',   0, '11:30', 'pending'),
  ('DEMO09', '0911000004', 'Chăm sóc chân','Trang',  0, '15:00', 'pending'),
  ('DEMO10', '0911000005', 'Gel chân',     'Mai',    1, '14:00', 'confirmed'),
  ('DEMO11', '0911000001', 'Đắp bột',      'Trang',  2, '16:00', 'pending'),
  ('DEMO12', '0911000002', 'Gel tay',      'Linh',   3, '10:30', 'confirmed')
) as d(code, phone, service, staff_name, day_offset, at_time, status)
join customers c on c.phone = d.phone
join services sv on sv.name = d.service
join staff st on st.name = d.staff_name
cross join lateral (
  select ((current_date + d.day_offset)::timestamp + d.at_time::time) at time zone 'Asia/Ho_Chi_Minh' as start_at
) t(start_at);
```

Lưu ý: câu `insert ... select` dùng `start_at` từ `lateral`; nếu Postgres báo cột `start_at` không xác định ở danh sách chọn, đổi `start_at` thành `t.start_at` ở ba chỗ trong danh sách chọn.

- [ ] **Step 2: Nạp dữ liệu mẫu và kiểm tra**

Run: `npm run db:seed`
Expected: `đã nạp dữ liệu mẫu`. Chạy lần hai cũng thành công (không lỗi trùng).

Run: `npm run db:test`
Expected: vẫn `PASS` cả 3 tệp (fixtures xóa sạch rồi rollback, không làm mất dữ liệu mẫu).

- [ ] **Step 3: Viết Supabase client và kiểu dữ liệu**

`src/lib/supabase.ts`:

```ts
import { createClient } from '@supabase/supabase-js';

const url = import.meta.env.VITE_SUPABASE_URL;
const anonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

if (!url || !anonKey) {
  throw new Error('Thiếu VITE_SUPABASE_URL hoặc VITE_SUPABASE_ANON_KEY trong .env.local');
}

export const supabase = createClient(url, anonKey);
```

`src/types/db.ts`:

```ts
import type { BookingStatus } from '@/config/statuses';

// Tên trường giống hệt CSDL (snake_case). Không có lớp đổi tên.
export type ServiceCategory = { id: string; name: string; slug: string; sort: number };

export type Service = {
  id: string;
  category_id: string;
  name: string;
  description: string | null;
  price_vnd: number;
  duration_min: number;
  image_url: string | null;
  active: boolean;
  sort: number;
};

export type Staff = {
  id: string;
  name: string;
  bio: string | null;
  avatar_url: string | null;
  active: boolean;
  sort: number;
};

export type OpeningHours = {
  weekday: number;
  open_time: string;
  close_time: string;
  is_closed: boolean;
};

export type StaffSchedule = {
  staff_id: string;
  weekday: number;
  start_time: string;
  end_time: string;
  is_off: boolean;
};

export type GalleryItem = {
  id: string;
  title: string;
  style_tag: string;
  image_url: string | null;
  price_hint_vnd: number | null;
  featured: boolean;
  active: boolean;
  sort: number;
};

export type ShopSettings = {
  address: string;
  phone: string;
  zalo: string | null;
  map_url: string | null;
  slot_minutes: number;
  lead_minutes: number;
  cancel_hours: number;
  points_per_10k: number;
};

export type Customer = {
  id: string;
  profile_id: string | null;
  full_name: string;
  phone: string;
  points_balance: number;
  admin_note: string | null;
  created_at: string;
};

export type Booking = {
  id: string;
  code: string;
  customer_id: string;
  service_id: string;
  staff_id: string;
  start_at: string;
  end_at: string;
  status: BookingStatus;
  price_vnd: number;
  discount_vnd: number;
  total_vnd: number;
  coupon_id: string | null;
  note: string | null;
  created_at: string;
  cancel_reason: string | null;
};

export type Coupon = {
  id: string;
  code: string;
  percent: number;
  min_total_vnd: number;
  starts_at: string;
  ends_at: string;
  max_uses: number | null;
  used_count: number;
  active: boolean;
};

export type PointsLedgerRow = {
  id: string;
  customer_id: string;
  booking_id: string | null;
  delta: number;
  reason: string;
  created_at: string;
};
```

- [ ] **Step 4: Thêm hướng dẫn tạo admin vào `README.md`**

Nối vào cuối `README.md`:

```markdown
## Tạo tài khoản admin (làm tay, không có trang đăng ký admin)

1. Supabase → Authentication → Users → Add user (email + mật khẩu mạnh, bật Auto Confirm).
2. SQL Editor, chạy (thay `<id>` bằng id của user vừa tạo):
   `insert into profiles (id, role, full_name, phone) values ('<id>', 'admin', 'Chủ tiệm', '0900000000');`
```

- [ ] **Step 5: Kiểm tra toàn bộ**

Run: `npm run check`
Expected: typecheck, lint, test qua. (`src/lib/supabase.ts` chưa được import ở đâu nên không cần biến môi trường khi chạy test.)

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "feat: dữ liệu mẫu, Supabase client và kiểu dữ liệu bảng"
```

---

## Self-Review (đã chạy)

**Spec coverage (phần thuộc Kế hoạch 1):** mục 4 (công nghệ, một phần: cài ở Task 1, phần giao diện cài ở Kế hoạch 2); mục 5 (khung thư mục: `config`, `lib`, `features`, `types`, `supabase`); mục 6 (bảng, ràng buộc 6.1, RPC 6.2, RLS 6.3: Task 6-7); mục 7.1 (giờ trống: Task 4), 7.2 (đặt lịch: `create_booking`), 7.3 (hủy: `cancel_booking`), 7.4 (điểm: `set_booking_status`, `pointsFor`), 7.5 (khuyến mãi: `validate_coupon`), 7.7 (doanh thu: dữ liệu mẫu, truy vấn ở Kế hoạch 3); mục 8 (đăng ký/đăng nhập: `register_customer`, `phoneToEmail` ở Kế hoạch 2); mục 13 (dữ liệu mẫu: Task 8); mục 14 (unit test, DB test: Task 3-7). Chưa thuộc kế hoạch này: giao diện, E2E, nhắc lịch mô phỏng (Kế hoạch 2-3), thống kê doanh thu (Kế hoạch 3).

**Placeholder scan:** không có "TBD" hay "xử lý lỗi phù hợp" bỏ ngỏ.

**Nhất quán kiểu:** tên `computeSlots`, `mergeSlots`, `buildSlotGrid`, `openWindow`, `staffWorkWindow`, `busyToIntervals`, `earliestStartFor`, `Interval`, `SlotView` dùng thống nhất; trường RPC `snake_case` khớp `src/types/db.ts`; mã lỗi trong `COPY.errors` khớp `raise exception` của các hàm SQL (`slot_taken`, `slot_past`, `slot_invalid`, `bad_phone`, `bad_name`, `service_not_found`, `coupon_invalid`, `booking_not_found`, `cancel_not_allowed`, `cancel_too_late`, `bad_transition`, `phone_taken`).

**Review Focus:** 1 → Task 7 (`slot_taken`); 2 → Task 4 (kết thúc đúng giờ đóng) và Task 7 (19:00 và 19:30); 3 → Task 3 và Task 5; 4 → Task 7 (`bad_transition`, một dòng sổ điểm); 5 → Task 1 (`globalSetup`) và Task 3.
