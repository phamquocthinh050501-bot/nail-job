# Trang khách hàng: Kế hoạch triển khai (Kế hoạch 2/3)

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Dựng toàn bộ website phía khách: khung giao diện, trang chủ có hero 3D dự phòng, bảng giá, gallery, đặt lịch (chống trùng giờ), tra cứu và hủy lịch, đăng nhập SĐT + PIN, tài khoản (lịch sử, điểm, nhắc lịch), liên hệ; có test và kiểm thử trình duyệt ở 3 cỡ màn hình.

**Architecture:** Mỗi trang ghép các hook trong `features/*/queries.ts` (gọi `src/lib/api/*`) với component dùng lại trong `src/components`. Logic tính toán (giờ trống, bước đặt lịch, nhắc lịch, tệp lịch .ics) là hàm thuần có test. Mọi giá trị dùng chung lấy từ `src/config/`. Một bộ token màu duy nhất trong `src/styles/tokens.css`.

**Tech Stack:** React 18+, React Router, TanStack Query, Tailwind CSS v4, Radix UI (Dialog, Tabs, Switch), cva, Motion, Embla Carousel, Three.js (chỉ hero), react-hook-form + zod, Vitest + Testing Library, Playwright.

**Spec:** `docs/superpowers/specs/2026-10-15-nail-salon-web-design.md` (mục 9, 10, 11.1-11.8). **Quy chuẩn:** `docs/CODING_STANDARDS.md`. **Điều kiện:** Kế hoạch 1 đã xong (CSDL, config, hàm thuần, kiểu dữ liệu).

**Khác biệt có chủ ý so với spec:**
- Dùng **Radix UI trực tiếp** (Dialog, Tabs, Switch) thay vì chạy CLI `shadcn`, để chỉ có **một** bộ token màu (shadcn thêm bộ biến thứ hai). Cùng nền tảng Radix, cùng độ truy cập.
- Ô PIN là **một ô nhập** 6 số có nút hiện/ẩn (cho dán, dùng trình quản lý mật khẩu), không phải 6 ô rời.
- Thanh menu hiện đủ chữ từ rộng **768px**; dưới 768px dùng nút ☰ (spec nói 640px, 768px hợp với máy tính bảng dọc hơn).
- Lịch sử trong tài khoản là **danh sách dòng** co giãn (cùng một component `BookingCard`), không dựng bảng riêng.

## Global Constraints

- Giao diện tiếng Việt; giá `350.000đ` qua `formatVnd`; ngày giờ qua `lib/format.ts`; không `new Date().toLocaleString()` rải rác.
- Không export mặc định; không emoji làm icon (chỉ `lucide-react`); hex màu chỉ ở `tokens.css` và `config/gallery.ts`.
- Component và page **không** gọi `supabase` trực tiếp; mọi truy vấn qua `src/lib/api/*` và hook TanStack.
- Mọi truy vấn hiển thị qua `<QueryState>`; chữ dùng lại lấy từ `COPY`, trạng thái từ `BOOKING_STATUS`.
- Vùng chạm tối thiểu 44px (`min-h-11`); mọi ô nhập có nhãn hiển thị; lỗi đặt cạnh ô; vòng focus thấy rõ; `prefers-reduced-motion` tắt 3D và chuyển động lớn.
- Điểm ngắt: dưới 768px (điện thoại), từ 768px (`md:`), từ 1024px (`lg:`). Không cuộn ngang ở 375, 768, 1280.
- Hero 3D chỉ ở trang chủ, tải lười, có dự phòng 2.5D (điều kiện ở spec 9.7).
- Không đặt SĐT hay dữ liệu cá nhân trong URL; dữ liệu giữa các trang truyền bằng `navigate(..., { state })`.

## Review Focus

1. Giờ vừa bị người khác đặt khi bấm xác nhận: báo `slot_taken`, giữ nguyên thông tin đã nhập, đưa về bước Giờ và tải lại giờ trống (Task 8).
2. Đổi dịch vụ, thợ hoặc ngày sau khi đã chọn giờ: giờ cũ bị bỏ chọn, không gửi giờ không còn hợp lệ (Task 7 test reducer).
3. Ngày tiệm nghỉ hoặc thợ nghỉ: hiển thị "Tiệm nghỉ" hoặc lưới giờ mờ hết, không bấm được (Task 7 test `computeDayAvailability`).
4. Khách quá hạn hủy: nút hủy vô hiệu kèm số điện thoại bấm gọi được (Task 9 test).
5. Máy yếu, bật giảm chuyển động, tiết kiệm dữ liệu, hoặc FPS dưới 30: hero tự rơi về 2.5D (Task 4 test `shouldUse3D`).

---

## Task 1: Nền tảng giao diện (token, nguyên tử, lỗi, provider)

**Files:**
- Create: `src/styles/tokens.css`, `src/lib/cn.ts`, `src/lib/errors.ts`, `src/lib/queryClient.ts`, `src/lib/queryKeys.ts`
- Create: `src/components/ui/Button.tsx`, `Field.tsx`, `Chip.tsx`, `Skeleton.tsx`, `EmptyState.tsx`, `Dialog.tsx`, `ConfirmDialog.tsx`, `Drawer.tsx`, `TabBar.tsx`, `Switch.tsx`
- Create: `src/components/PriceText.tsx`, `DurationText.tsx`, `StatusBadge.tsx`, `QueryState.tsx`
- Modify: `src/index.css`, `index.html`, `src/config/copy.ts` (thêm `a11y`), `src/main.tsx`
- Test: `src/lib/errors.test.ts`, `src/components/StatusBadge.test.tsx`, `src/components/QueryState.test.tsx`

**Interfaces:**
- Produces:
  - `cn(...inputs: ClassValue[]): string`
  - `AppError(code: string)`, `toUserMessage(error: unknown): string`
  - `unwrap<T>(result: { data: T | null; error: { message: string } | null }): T` (trong `src/lib/api/unwrap.ts`)
  - `queryClient`, `queryKeys` (`categories`, `services`, `staff`, `gallery`, `settings`, `hours`, `schedules`, `busy(day)`, `myCustomer`, `myBookings`, `profile`)
  - `Button`, `LinkButton`, `buttonVariants` (biến thể `primary | gel | secondary | danger | ghost`, cỡ `md | sm`)
  - `Field` (render-prop `(a11y) => ReactNode`), `Input`, `Textarea`, `inputClass`
  - `Chip({ active, children, ...button })`, `Skeleton`, `EmptyState({ title, description?, action? })`
  - `Dialog({ open, onOpenChange, title, description?, children })`, `Drawer` (cùng props, `side?: 'left' | 'right'`)
  - `ConfirmDialog({ open, onOpenChange, title, description?, confirmLabel, cancelLabel?, tone?: 'danger' | 'primary', pending?, onConfirm })`
  - `TabBar({ value, onValueChange, items: { value: string; label: string; count?: number }[] })`
  - `Switch({ checked, onCheckedChange, label })`
  - `PriceText({ value })`, `DurationText({ minutes })`, `StatusBadge({ status })`
  - `QueryState<T>({ query, isEmpty?, empty?, skeleton?, children: (data: T) => ReactNode })`

- [ ] **Step 1: Cài thư viện giao diện**

```powershell
npm install @radix-ui/react-dialog @radix-ui/react-tabs @radix-ui/react-switch embla-carousel-react three
npm install -D @types/three
```

- [ ] **Step 2: Viết test hỏng**

`src/lib/errors.test.ts`:

```ts
import { COPY } from '@/config/copy';
import { AppError, toUserMessage } from './errors';

describe('toUserMessage', () => {
  it('đổi mã lỗi của CSDL thành câu tiếng Việt', () => {
    expect(toUserMessage(new AppError('slot_taken'))).toBe(COPY.errors.slot_taken);
    expect(toUserMessage(new Error('cancel_too_late'))).toBe(COPY.errors.cancel_too_late);
  });

  it('nhận ra lỗi mạng', () => {
    expect(toUserMessage(new TypeError('Failed to fetch'))).toBe(COPY.errors.network);
  });

  it('lỗi lạ thì báo chung, không lộ nội dung kỹ thuật', () => {
    expect(toUserMessage(new Error('duplicate key value violates ...'))).toBe(COPY.errors.generic);
    expect(toUserMessage(undefined)).toBe(COPY.errors.generic);
  });
});
```

`src/components/StatusBadge.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react';
import { BOOKING_STATUS, BOOKING_STATUS_LIST } from '@/config/statuses';
import { StatusBadge } from './StatusBadge';

describe('StatusBadge', () => {
  it.each(BOOKING_STATUS_LIST)('hiển thị nhãn chữ cho trạng thái %s (không chỉ màu)', (status) => {
    render(<StatusBadge status={status} />);
    expect(screen.getByText(BOOKING_STATUS[status].label)).toBeInTheDocument();
  });
});
```

`src/components/QueryState.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import type { UseQueryResult } from '@tanstack/react-query';
import { COPY } from '@/config/copy';
import { AppError } from '@/lib/errors';
import { QueryState } from './QueryState';

const fake = (partial: Partial<UseQueryResult<string[]>>) =>
  ({ isPending: false, isError: false, data: [], error: null, refetch: vi.fn(), ...partial }) as UseQueryResult<string[]>;

describe('QueryState', () => {
  it('đang tải thì hiện khung chờ có nhãn', () => {
    render(<QueryState query={fake({ isPending: true, data: undefined })}>{() => <p>xong</p>}</QueryState>);
    expect(screen.getByRole('status', { name: COPY.a11y.loading })).toBeInTheDocument();
  });

  it('lỗi thì hiện câu tiếng Việt và nút Thử lại gọi refetch', async () => {
    const refetch = vi.fn();
    render(
      <QueryState query={fake({ isError: true, error: new AppError('slot_taken'), refetch })}>
        {() => <p>xong</p>}
      </QueryState>,
    );
    expect(screen.getByText(COPY.errors.slot_taken)).toBeInTheDocument();
    await userEvent.click(screen.getByRole('button', { name: COPY.actions.retry }));
    expect(refetch).toHaveBeenCalled();
  });

  it('dữ liệu trống thì hiện phần empty', () => {
    render(
      <QueryState query={fake({ data: [] })} isEmpty={(d) => d.length === 0} empty={<p>Chưa có gì</p>}>
        {() => <p>xong</p>}
      </QueryState>,
    );
    expect(screen.getByText('Chưa có gì')).toBeInTheDocument();
  });

  it('có dữ liệu thì render children', () => {
    render(<QueryState query={fake({ data: ['a'] })}>{(d) => <p>{d.length} mục</p>}</QueryState>);
    expect(screen.getByText('1 mục')).toBeInTheDocument();
  });
});
```

- [ ] **Step 3: Chạy để thấy hỏng**

Run: `npx vitest run src/lib/errors.test.ts src/components`
Expected: FAIL (không tìm thấy module).

- [ ] **Step 4: Token màu, font, nền trang**

`src/styles/tokens.css`:

```css
@theme {
  --color-cream: #fff7f5;
  --color-surface: #ffffff;
  --color-blush: #fde6ee;
  --color-line: #f0cdd8;
  --color-ink: #3b2230;
  --color-berry: #7a3b55;
  --color-rose: #c2477a;
  --color-plum-1: #2a1450;
  --color-plum-2: #7b2d8e;
  --color-gel-pink: #ff7ad9;
  --color-violet: #8f6bff;
  --color-success: #1f7a3d;
  --color-warn: #9a5b00;
  --color-danger: #b3261e;
  --color-info: #2457a6;

  --font-heading: 'Plus Jakarta Sans', system-ui, sans-serif;
  --font-body: 'Be Vietnam Pro', system-ui, sans-serif;

  --radius-control: 12px;
  --radius-card: 16px;
  --radius-dialog: 20px;

  --shadow-card: 0 2px 8px rgb(122 59 85 / 0.08);
  --shadow-pop: 0 12px 32px rgb(42 20 80 / 0.18);

  --animate-float: float 6s ease-in-out infinite;
  @keyframes float {
    0%,
    100% {
      transform: translateY(0);
    }
    50% {
      transform: translateY(-10px);
    }
  }
}

@layer base {
  html {
    scroll-behavior: smooth;
  }
  body {
    @apply bg-cream font-body text-base leading-relaxed text-ink antialiased;
  }
  h1,
  h2,
  h3 {
    @apply font-heading text-berry;
  }
  @media (prefers-reduced-motion: reduce) {
    html {
      scroll-behavior: auto;
    }
    *,
    *::before,
    *::after {
      animation-duration: 0.01ms !important;
      transition-duration: 0.01ms !important;
    }
  }
}
```

`src/index.css` (thay toàn bộ):

```css
@import 'tailwindcss';
@import './styles/tokens.css';
```

Trong `index.html`, thêm vào `<head>` (tiếng Việt cần subset `vietnamese`) và đổi `<html lang="vi">`:

```html
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
  href="https://fonts.googleapis.com/css2?family=Be+Vietnam+Pro:wght@400;500;600&family=Plus+Jakarta+Sans:wght@700;800&display=swap&subset=vietnamese,latin"
  rel="stylesheet"
/>
<title>Lụa Nails</title>
```

- [ ] **Step 5: Thêm nhãn trợ năng vào `COPY`**

Trong `src/config/copy.ts`, thêm khóa `a11y` vào đối tượng `COPY` (trước `} as const;`):

```ts
  a11y: {
    openMenu: 'Mở menu',
    closeMenu: 'Đóng menu',
    skip: 'Bỏ qua tới nội dung chính',
    loading: 'Đang tải',
  },
```

- [ ] **Step 6: Tiện ích lỗi, query, khóa**

`src/lib/cn.ts`:

```ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export const cn = (...inputs: ClassValue[]) => twMerge(clsx(inputs));
```

`src/lib/errors.ts`:

```ts
import { COPY, type ErrorCode } from '@/config/copy';

// Mã lỗi ngắn (slot_taken, ...) do hàm RPC ném ra; message của lỗi chính là mã đó.
export class AppError extends Error {
  constructor(public code: string) {
    super(code);
  }
}

export function toUserMessage(error: unknown): string {
  const raw = error instanceof Error ? error.message : '';
  if (raw in COPY.errors) return COPY.errors[raw as ErrorCode];
  if (/failed to fetch|network|load failed/i.test(raw)) return COPY.errors.network;
  return COPY.errors.generic;
}
```

`src/lib/api/unwrap.ts`:

```ts
import { AppError } from '@/lib/errors';

export function unwrap<T>(result: { data: T | null; error: { message: string } | null }): T {
  if (result.error) throw new AppError(result.error.message);
  return result.data as T;
}
```

`src/lib/queryClient.ts`:

```ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: { queries: { staleTime: 60_000, retry: 1, refetchOnWindowFocus: false } },
});
```

`src/lib/queryKeys.ts`:

```ts
export const queryKeys = {
  categories: ['categories'],
  services: ['services'],
  staff: ['staff'],
  gallery: ['gallery'],
  settings: ['settings'],
  hours: ['opening-hours'],
  schedules: ['staff-schedules'],
  busy: (day: string) => ['busy', day],
  myCustomer: ['my-customer'],
  myBookings: ['my-bookings'],
  profile: ['profile'],
} as const;
```

- [ ] **Step 7: Nguyên tử giao diện**

`src/components/ui/Button.tsx`:

```tsx
import { cva, type VariantProps } from 'class-variance-authority';
import { forwardRef, type ButtonHTMLAttributes } from 'react';
import { Link, type LinkProps } from 'react-router-dom';
import { cn } from '@/lib/cn';

export const buttonVariants = cva(
  'inline-flex min-h-11 items-center justify-center gap-2 rounded-control px-5 font-semibold transition focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-rose disabled:cursor-not-allowed disabled:opacity-50',
  {
    variants: {
      variant: {
        primary: 'bg-rose text-white hover:bg-berry',
        // Chữ tối trên nền gradient hồng để đủ tương phản (trắng trên hồng sáng không đạt 4,5:1).
        gel: 'bg-gradient-to-r from-gel-pink to-violet text-plum-1 shadow-pop hover:brightness-105',
        secondary: 'border border-line bg-surface text-berry hover:bg-blush',
        danger: 'border border-danger bg-surface text-danger hover:bg-danger/10',
        ghost: 'text-berry hover:bg-blush',
      },
      size: { md: '', sm: 'min-h-9 px-3 text-sm' },
    },
    defaultVariants: { variant: 'primary', size: 'md' },
  },
);

type Variants = VariantProps<typeof buttonVariants>;

export const Button = forwardRef<
  HTMLButtonElement,
  ButtonHTMLAttributes<HTMLButtonElement> & Variants
>(({ className, variant, size, type = 'button', ...props }, ref) => (
  <button ref={ref} type={type} className={cn(buttonVariants({ variant, size }), className)} {...props} />
));
Button.displayName = 'Button';

export function LinkButton({ className, variant, size, ...props }: LinkProps & Variants) {
  return <Link className={cn(buttonVariants({ variant, size }), className)} {...props} />;
}
```

`src/components/ui/Field.tsx`:

```tsx
import { forwardRef, useId, type InputHTMLAttributes, type ReactNode, type TextareaHTMLAttributes } from 'react';
import { cn } from '@/lib/cn';

type A11yProps = { id: string; 'aria-describedby'?: string; 'aria-invalid'?: true };

type FieldProps = {
  label: string;
  error?: string;
  hint?: string;
  required?: boolean;
  children: (a11y: A11yProps) => ReactNode;
};

export function Field({ label, error, hint, required, children }: FieldProps) {
  const id = useId();
  const hintId = `${id}-hint`;
  const errorId = `${id}-error`;
  const describedBy = [hint && hintId, error && errorId].filter(Boolean).join(' ') || undefined;
  return (
    <div className="flex flex-col gap-1.5">
      <label htmlFor={id} className="text-sm font-medium text-berry">
        {label}
        {required && (
          <span aria-hidden="true" className="text-danger">
            {' *'}
          </span>
        )}
      </label>
      {children({ id, 'aria-describedby': describedBy, 'aria-invalid': error ? true : undefined })}
      {hint && (
        <p id={hintId} className="text-sm text-ink/70">
          {hint}
        </p>
      )}
      {error && (
        <p id={errorId} role="alert" className="text-sm text-danger">
          {error}
        </p>
      )}
    </div>
  );
}

export const inputClass =
  'min-h-11 w-full rounded-control border border-line bg-surface px-3 text-ink placeholder:text-ink/50 focus-visible:outline-2 focus-visible:outline-rose aria-[invalid=true]:border-danger';

export const Input = forwardRef<HTMLInputElement, InputHTMLAttributes<HTMLInputElement>>(
  ({ className, ...props }, ref) => <input ref={ref} className={cn(inputClass, className)} {...props} />,
);
Input.displayName = 'Input';

export const Textarea = forwardRef<HTMLTextAreaElement, TextareaHTMLAttributes<HTMLTextAreaElement>>(
  ({ className, ...props }, ref) => (
    <textarea ref={ref} className={cn(inputClass, 'min-h-24 py-2', className)} {...props} />
  ),
);
Textarea.displayName = 'Textarea';
```

`src/components/ui/Chip.tsx`:

```tsx
import type { ButtonHTMLAttributes } from 'react';
import { cn } from '@/lib/cn';

export function Chip({
  active,
  className,
  ...props
}: ButtonHTMLAttributes<HTMLButtonElement> & { active?: boolean }) {
  return (
    <button
      type="button"
      aria-pressed={active}
      className={cn(
        'inline-flex min-h-11 items-center rounded-full border px-4 text-sm font-medium transition focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-rose',
        active ? 'border-rose bg-rose text-white' : 'border-line bg-surface text-berry hover:bg-blush',
        className,
      )}
      {...props}
    />
  );
}
```

`src/components/ui/Skeleton.tsx`:

```tsx
import { COPY } from '@/config/copy';
import { cn } from '@/lib/cn';

export function Skeleton({ className }: { className?: string }) {
  return (
    <div
      role="status"
      aria-label={COPY.a11y.loading}
      className={cn('animate-pulse rounded-card bg-blush', className)}
    />
  );
}
```

`src/components/ui/EmptyState.tsx`:

```tsx
import type { ReactNode } from 'react';

export function EmptyState({
  title,
  description,
  action,
}: {
  title: string;
  description?: string;
  action?: ReactNode;
}) {
  return (
    <div className="flex flex-col items-center gap-3 rounded-card border border-dashed border-line p-8 text-center">
      <p className="font-heading text-lg font-bold text-berry">{title}</p>
      {description && <p className="max-w-prose text-ink/80">{description}</p>}
      {action}
    </div>
  );
}
```

`src/components/ui/Dialog.tsx`:

```tsx
import * as RadixDialog from '@radix-ui/react-dialog';
import { X } from 'lucide-react';
import type { ReactNode } from 'react';
import { COPY } from '@/config/copy';

type DialogProps = {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  title: string;
  description?: string;
  children?: ReactNode;
};

export function Dialog({ open, onOpenChange, title, description, children }: DialogProps) {
  return (
    <RadixDialog.Root open={open} onOpenChange={onOpenChange}>
      <RadixDialog.Portal>
        <RadixDialog.Overlay className="fixed inset-0 z-40 bg-plum-1/50" />
        <RadixDialog.Content className="fixed left-1/2 top-1/2 z-50 w-[calc(100%-2rem)] max-w-md -translate-x-1/2 -translate-y-1/2 rounded-dialog bg-surface p-6 shadow-pop">
          <RadixDialog.Title className="pr-10 font-heading text-xl font-bold text-berry">
            {title}
          </RadixDialog.Title>
          <RadixDialog.Description className={description ? 'mt-2 text-ink/80' : 'sr-only'}>
            {description ?? title}
          </RadixDialog.Description>
          {children && <div className="mt-4">{children}</div>}
          <RadixDialog.Close
            aria-label={COPY.actions.close}
            className="absolute right-3 top-3 grid size-11 place-items-center rounded-full hover:bg-blush"
          >
            <X aria-hidden="true" className="size-5" />
          </RadixDialog.Close>
        </RadixDialog.Content>
      </RadixDialog.Portal>
    </RadixDialog.Root>
  );
}
```

`src/components/ui/Drawer.tsx`:

```tsx
import * as RadixDialog from '@radix-ui/react-dialog';
import { X } from 'lucide-react';
import type { ReactNode } from 'react';
import { COPY } from '@/config/copy';
import { cn } from '@/lib/cn';

type DrawerProps = {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  title: string;
  side?: 'left' | 'right';
  children?: ReactNode;
};

export function Drawer({ open, onOpenChange, title, side = 'right', children }: DrawerProps) {
  return (
    <RadixDialog.Root open={open} onOpenChange={onOpenChange}>
      <RadixDialog.Portal>
        <RadixDialog.Overlay className="fixed inset-0 z-40 bg-plum-1/50" />
        <RadixDialog.Content
          className={cn(
            'fixed inset-y-0 z-50 flex w-[min(24rem,100%)] flex-col bg-surface p-6 shadow-pop',
            side === 'right' ? 'right-0' : 'left-0',
          )}
        >
          <RadixDialog.Title className="pr-10 font-heading text-xl font-bold text-berry">
            {title}
          </RadixDialog.Title>
          <RadixDialog.Description className="sr-only">{title}</RadixDialog.Description>
          <div className="mt-4 flex-1 overflow-y-auto">{children}</div>
          <RadixDialog.Close
            aria-label={COPY.actions.close}
            className="absolute right-3 top-3 grid size-11 place-items-center rounded-full hover:bg-blush"
          >
            <X aria-hidden="true" className="size-5" />
          </RadixDialog.Close>
        </RadixDialog.Content>
      </RadixDialog.Portal>
    </RadixDialog.Root>
  );
}
```

`src/components/ui/ConfirmDialog.tsx`:

```tsx
import { COPY } from '@/config/copy';
import { Button } from './Button';
import { Dialog } from './Dialog';

type ConfirmDialogProps = {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  title: string;
  description?: string;
  confirmLabel: string;
  cancelLabel?: string;
  tone?: 'danger' | 'primary';
  pending?: boolean;
  onConfirm: () => void;
};

// Nút an toàn (giữ nguyên) đặt trước và nổi bật; nút nguy hiểm đứng sau.
export function ConfirmDialog({
  open,
  onOpenChange,
  title,
  description,
  confirmLabel,
  cancelLabel = COPY.actions.close,
  tone = 'primary',
  pending,
  onConfirm,
}: ConfirmDialogProps) {
  return (
    <Dialog open={open} onOpenChange={onOpenChange} title={title} description={description}>
      <div className="flex flex-col-reverse gap-3 sm:flex-row sm:justify-end">
        <Button variant={tone === 'danger' ? 'danger' : 'primary'} disabled={pending} onClick={onConfirm}>
          {confirmLabel}
        </Button>
        <Button variant={tone === 'danger' ? 'primary' : 'secondary'} onClick={() => onOpenChange(false)}>
          {cancelLabel}
        </Button>
      </div>
    </Dialog>
  );
}
```

`src/components/ui/TabBar.tsx`:

```tsx
import * as Tabs from '@radix-ui/react-tabs';
import { cn } from '@/lib/cn';

type TabItem = { value: string; label: string; count?: number };

// Chỉ vẽ hàng tab; trang tự hiển thị nội dung theo `value`.
export function TabBar({
  value,
  onValueChange,
  items,
}: {
  value: string;
  onValueChange: (value: string) => void;
  items: TabItem[];
}) {
  return (
    <Tabs.Root value={value} onValueChange={onValueChange}>
      <Tabs.List className="flex gap-2 overflow-x-auto pb-1" aria-label="Nhóm">
        {items.map((item) => (
          <Tabs.Trigger
            key={item.value}
            value={item.value}
            className={cn(
              'inline-flex min-h-11 shrink-0 items-center gap-2 rounded-full border px-4 text-sm font-medium focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-rose',
              'data-[state=active]:border-rose data-[state=active]:bg-rose data-[state=active]:text-white',
              'data-[state=inactive]:border-line data-[state=inactive]:bg-surface data-[state=inactive]:text-berry',
            )}
          >
            {item.label}
            {item.count !== undefined && <span className="text-xs opacity-80">{item.count}</span>}
          </Tabs.Trigger>
        ))}
      </Tabs.List>
    </Tabs.Root>
  );
}
```

`src/components/ui/Switch.tsx`:

```tsx
import * as RadixSwitch from '@radix-ui/react-switch';
import { useId } from 'react';

export function Switch({
  checked,
  onCheckedChange,
  label,
}: {
  checked: boolean;
  onCheckedChange: (checked: boolean) => void;
  label: string;
}) {
  const id = useId();
  return (
    <div className="flex min-h-11 items-center gap-3">
      <RadixSwitch.Root
        id={id}
        checked={checked}
        onCheckedChange={onCheckedChange}
        className="relative h-7 w-12 shrink-0 rounded-full bg-line outline-offset-2 focus-visible:outline-2 focus-visible:outline-rose data-[state=checked]:bg-rose"
      >
        <RadixSwitch.Thumb className="block size-5 translate-x-1 rounded-full bg-white transition-transform data-[state=checked]:translate-x-6" />
      </RadixSwitch.Root>
      <label htmlFor={id} className="text-berry">
        {label}
      </label>
    </div>
  );
}
```

- [ ] **Step 8: Component dùng chung**

`src/components/PriceText.tsx`:

```tsx
import { formatVnd } from '@/lib/format';

export const PriceText = ({ value }: { value: number }) => (
  <span className="font-bold tabular-nums text-rose">{formatVnd(value)}</span>
);
```

`src/components/DurationText.tsx`:

```tsx
import { Clock } from 'lucide-react';
import { formatDuration } from '@/lib/format';

export const DurationText = ({ minutes }: { minutes: number }) => (
  <span className="inline-flex items-center gap-1 text-sm text-ink/80">
    <Clock aria-hidden="true" className="size-4" />
    {formatDuration(minutes)}
  </span>
);
```

`src/components/StatusBadge.tsx`:

```tsx
import { BOOKING_STATUS, type BookingStatus, type StatusTone } from '@/config/statuses';
import { cn } from '@/lib/cn';

const TONE_CLASS: Record<StatusTone, string> = {
  warn: 'bg-warn/10 text-warn',
  info: 'bg-info/10 text-info',
  success: 'bg-success/10 text-success',
  danger: 'bg-danger/10 text-danger',
};

// Luôn có chữ và icon, không truyền tin chỉ bằng màu.
export function StatusBadge({ status }: { status: BookingStatus }) {
  const { label, tone, icon: Icon } = BOOKING_STATUS[status];
  return (
    <span className={cn('inline-flex items-center gap-1 rounded-full px-3 py-1 text-sm font-medium', TONE_CLASS[tone])}>
      <Icon aria-hidden="true" className="size-4" />
      {label}
    </span>
  );
}
```

`src/components/QueryState.tsx`:

```tsx
import type { UseQueryResult } from '@tanstack/react-query';
import type { ReactNode } from 'react';
import { COPY } from '@/config/copy';
import { toUserMessage } from '@/lib/errors';
import { Button } from './ui/Button';
import { EmptyState } from './ui/EmptyState';
import { Skeleton } from './ui/Skeleton';

type Props<T> = {
  query: UseQueryResult<T>;
  isEmpty?: (data: T) => boolean;
  empty?: ReactNode;
  skeleton?: ReactNode;
  children: (data: T) => ReactNode;
};

// Một nơi duy nhất xử lý ba trạng thái: đang tải, lỗi, trống.
export function QueryState<T>({ query, isEmpty, empty, skeleton, children }: Props<T>) {
  if (query.isPending) return <>{skeleton ?? <Skeleton className="h-24" />}</>;
  if (query.isError) {
    return (
      <EmptyState
        title={toUserMessage(query.error)}
        action={
          <Button variant="secondary" onClick={() => query.refetch()}>
            {COPY.actions.retry}
          </Button>
        }
      />
    );
  }
  if (isEmpty?.(query.data)) return <>{empty ?? <EmptyState title={COPY.empty.noResults} />}</>;
  return <>{children(query.data)}</>;
}
```

- [ ] **Step 9: Chạy để thấy qua**

Run: `npx vitest run src/lib/errors.test.ts src/components`
Expected: PASS (errors 3, StatusBadge 4, QueryState 4).

- [ ] **Step 10: Commit**

```bash
git add -A
git commit -m "feat(ui): token màu, nguyên tử giao diện, xử lý lỗi và QueryState"
```

---

## Task 2: Khung trang, router và layout khách

**Files:**
- Create: `src/lib/api/catalog.ts`, `src/features/catalog/queries.ts`, `src/features/catalog/hours.ts`
- Create: `src/app/App.tsx`, `src/app/router.tsx`, `src/app/lazyPage.ts`, `src/app/CustomerLayout.tsx`
- Create: `src/components/layout/SiteHeader.tsx`, `SiteFooter.tsx`, `AccountLink.tsx`, `src/pages/NotFoundPage.tsx`
- Modify: `src/main.tsx`
- Test: `src/features/catalog/hours.test.ts`, `src/components/layout/SiteHeader.test.tsx`

**Interfaces:**
- Consumes: `CUSTOMER_NAV`, `ROUTES`, `SITE`, `COPY`, `Button/LinkButton`, `Drawer`, kiểu `db.ts`.
- Produces:
  - `fetchCategories | fetchServices | fetchStaff | fetchGallery | fetchShopSettings | fetchOpeningHours | fetchStaffSchedules` (trả `Promise<...>`)
  - Hook: `useCategories()`, `useServices()`, `useStaff()`, `useGallery()`, `useShopSettings()`, `useOpeningHours()`, `useStaffSchedules()` (đều trả `UseQueryResult`)
  - `summarizeHours(hours: OpeningHours[]): string`
  - `lazyPage(load, name): LazyExoticComponent`
  - `customerRoutes: RouteObject[]` (trong `router.tsx`), `router`

- [ ] **Step 1: Viết test hỏng**

`src/features/catalog/hours.test.ts`:

```ts
import type { OpeningHours } from '@/types/db';
import { summarizeHours } from './hours';

const day = (weekday: number, open = '09:00:00', close = '20:00:00', closed = false): OpeningHours => ({
  weekday,
  open_time: open,
  close_time: close,
  is_closed: closed,
});

describe('summarizeHours', () => {
  it('mọi ngày cùng giờ thì gộp một dòng', () => {
    const week = [0, 1, 2, 3, 4, 5, 6].map((d) => day(d));
    expect(summarizeHours(week)).toBe('Hàng ngày, 9:00 đến 20:00');
  });

  it('khác giờ giữa các ngày thì dẫn sang trang Liên hệ', () => {
    const week = [0, 1, 2, 3, 4, 5].map((d) => day(d)).concat(day(6, '10:00:00', '18:00:00'));
    expect(summarizeHours(week)).toBe('Xem giờ mở cửa từng ngày ở trang Liên hệ');
  });

  it('tiệm nghỉ cả tuần', () => {
    expect(summarizeHours([0, 1, 2, 3, 4, 5, 6].map((d) => day(d, '09:00', '20:00', true)))).toBe(
      'Tiệm tạm nghỉ',
    );
  });
});
```

`src/components/layout/SiteHeader.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MemoryRouter } from 'react-router-dom';
import { COPY } from '@/config/copy';
import { CUSTOMER_NAV } from '@/config/navigation';
import { SiteHeader } from './SiteHeader';

const renderHeader = () =>
  render(
    <MemoryRouter>
      <SiteHeader />
    </MemoryRouter>,
  );

describe('SiteHeader', () => {
  it('hiển thị mọi mục menu lấy từ config', () => {
    renderHeader();
    for (const item of CUSTOMER_NAV) {
      expect(screen.getAllByRole('link', { name: item.label }).length).toBeGreaterThan(0);
    }
  });

  it('có nút Đặt lịch luôn thấy', () => {
    renderHeader();
    expect(screen.getAllByRole('link', { name: COPY.actions.book }).length).toBeGreaterThan(0);
  });

  it('nút menu mở ngăn chứa liên kết điều hướng', async () => {
    renderHeader();
    await userEvent.click(screen.getByRole('button', { name: COPY.a11y.openMenu }));
    expect(screen.getByRole('dialog')).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/features/catalog src/components/layout`
Expected: FAIL (không tìm thấy module).

- [ ] **Step 3: API và hook danh mục**

`src/lib/api/catalog.ts`:

```ts
import { supabase } from '@/lib/supabase';
import type {
  GalleryItem,
  OpeningHours,
  Service,
  ServiceCategory,
  ShopSettings,
  Staff,
  StaffSchedule,
} from '@/types/db';
import { unwrap } from './unwrap';

export const fetchCategories = async (): Promise<ServiceCategory[]> =>
  unwrap(await supabase.from('service_categories').select('*').order('sort'));

export const fetchServices = async (): Promise<Service[]> =>
  unwrap(await supabase.from('services').select('*').order('sort'));

export const fetchStaff = async (): Promise<Staff[]> =>
  unwrap(await supabase.from('staff').select('*').order('sort'));

export const fetchGallery = async (): Promise<GalleryItem[]> =>
  unwrap(await supabase.from('gallery_items').select('*').order('sort'));

export const fetchShopSettings = async (): Promise<ShopSettings> =>
  unwrap(await supabase.from('shop_settings').select('*').limit(1).single());

export const fetchOpeningHours = async (): Promise<OpeningHours[]> =>
  unwrap(await supabase.from('opening_hours').select('*').order('weekday'));

export const fetchStaffSchedules = async (): Promise<StaffSchedule[]> =>
  unwrap(await supabase.from('staff_schedules').select('*'));
```

`src/features/catalog/queries.ts`:

```ts
import { useQuery } from '@tanstack/react-query';
import {
  fetchCategories,
  fetchGallery,
  fetchOpeningHours,
  fetchServices,
  fetchShopSettings,
  fetchStaff,
  fetchStaffSchedules,
} from '@/lib/api/catalog';
import { queryKeys } from '@/lib/queryKeys';

// Danh mục ít thay đổi nên giữ cache lâu hơn mặc định.
const CATALOG_STALE_MS = 5 * 60_000;

export const useCategories = () =>
  useQuery({ queryKey: queryKeys.categories, queryFn: fetchCategories, staleTime: CATALOG_STALE_MS });
export const useServices = () =>
  useQuery({ queryKey: queryKeys.services, queryFn: fetchServices, staleTime: CATALOG_STALE_MS });
export const useStaff = () =>
  useQuery({ queryKey: queryKeys.staff, queryFn: fetchStaff, staleTime: CATALOG_STALE_MS });
export const useGallery = () =>
  useQuery({ queryKey: queryKeys.gallery, queryFn: fetchGallery, staleTime: CATALOG_STALE_MS });
export const useShopSettings = () =>
  useQuery({ queryKey: queryKeys.settings, queryFn: fetchShopSettings, staleTime: CATALOG_STALE_MS });
export const useOpeningHours = () =>
  useQuery({ queryKey: queryKeys.hours, queryFn: fetchOpeningHours, staleTime: CATALOG_STALE_MS });
export const useStaffSchedules = () =>
  useQuery({ queryKey: queryKeys.schedules, queryFn: fetchStaffSchedules, staleTime: CATALOG_STALE_MS });
```

`src/features/catalog/hours.ts`:

```ts
import { minutesToLabel, timeToMinutes } from '@/lib/vnTime';
import type { OpeningHours } from '@/types/db';

const label = (time: string) => minutesToLabel(timeToMinutes(time));

export function summarizeHours(hours: OpeningHours[]): string {
  const open = hours.filter((h) => !h.is_closed);
  if (open.length === 0) return 'Tiệm tạm nghỉ';
  const same = open.every(
    (h) => h.open_time === open[0].open_time && h.close_time === open[0].close_time,
  );
  if (open.length === 7 && same) {
    return `Hàng ngày, ${label(open[0].open_time)} đến ${label(open[0].close_time)}`;
  }
  return 'Xem giờ mở cửa từng ngày ở trang Liên hệ';
}
```

- [ ] **Step 4: Layout, header, footer, router**

`src/app/lazyPage.ts`:

```ts
import { lazy, type ComponentType } from 'react';

// Các trang export có tên; lazy() cần export mặc định nên đổi tại đây, một chỗ duy nhất.
export const lazyPage = <T extends Record<string, ComponentType>>(load: () => Promise<T>, name: keyof T) =>
  lazy(async () => ({ default: (await load())[name] }));
```

`src/components/layout/AccountLink.tsx`:

```tsx
import { Link } from 'react-router-dom';
import { COPY } from '@/config/copy';
import { ROUTES } from '@/config/routes';

// Task 10 đổi thành hiện tên khách khi đã đăng nhập.
export function AccountLink() {
  return (
    <Link className="min-h-11 content-center px-2 font-medium text-berry hover:underline" to={ROUTES.login}>
      {COPY.actions.login}
    </Link>
  );
}
```

`src/components/layout/SiteHeader.tsx`:

```tsx
import { Menu } from 'lucide-react';
import { useState } from 'react';
import { Link, NavLink } from 'react-router-dom';
import { COPY } from '@/config/copy';
import { CUSTOMER_NAV } from '@/config/navigation';
import { ROUTES } from '@/config/routes';
import { SITE } from '@/config/site';
import { cn } from '@/lib/cn';
import { Button, LinkButton } from '@/components/ui/Button';
import { Drawer } from '@/components/ui/Drawer';
import { AccountLink } from './AccountLink';

const navLinkClass = ({ isActive }: { isActive: boolean }) =>
  cn(
    'min-h-11 content-center rounded-control px-3 font-medium focus-visible:outline-2 focus-visible:outline-rose',
    isActive ? 'bg-blush text-rose' : 'text-berry hover:bg-blush',
  );

export function SiteHeader() {
  const [menuOpen, setMenuOpen] = useState(false);
  return (
    <header className="sticky top-0 z-30 border-b border-line bg-cream/95 backdrop-blur">
      <div className="mx-auto flex max-w-[1120px] items-center justify-between gap-3 px-4 py-2 md:px-8">
        <Link to={ROUTES.home} className="font-heading text-xl font-extrabold text-berry">
          {SITE.brandName}
        </Link>
        <nav aria-label="Chính" className="hidden items-center gap-1 md:flex">
          {CUSTOMER_NAV.map((item) => (
            <NavLink key={item.to} to={item.to} end={item.to === ROUTES.home} className={navLinkClass}>
              {item.label}
            </NavLink>
          ))}
        </nav>
        <div className="flex items-center gap-2">
          <span className="hidden md:inline">
            <AccountLink />
          </span>
          <LinkButton to={ROUTES.booking} size="sm">
            {COPY.actions.book}
          </LinkButton>
          <Button
            variant="ghost"
            className="px-3 md:hidden"
            aria-label={COPY.a11y.openMenu}
            onClick={() => setMenuOpen(true)}
          >
            <Menu aria-hidden="true" className="size-6" />
          </Button>
        </div>
      </div>
      <Drawer open={menuOpen} onOpenChange={setMenuOpen} title={SITE.brandName}>
        <nav aria-label="Menu" className="flex flex-col gap-1" onClick={() => setMenuOpen(false)}>
          {CUSTOMER_NAV.map((item) => (
            <NavLink key={item.to} to={item.to} end={item.to === ROUTES.home} className={navLinkClass}>
              {item.label}
            </NavLink>
          ))}
          <AccountLink />
        </nav>
      </Drawer>
    </header>
  );
}
```

`src/components/layout/SiteFooter.tsx`:

```tsx
import { MapPin, Phone } from 'lucide-react';
import { Link } from 'react-router-dom';
import { ROUTES } from '@/config/routes';
import { SITE } from '@/config/site';
import { summarizeHours } from '@/features/catalog/hours';
import { useOpeningHours, useShopSettings } from '@/features/catalog/queries';

export function SiteFooter() {
  const settings = useShopSettings().data;
  const hours = useOpeningHours().data;
  return (
    <footer className="mt-16 border-t border-line bg-surface">
      <div className="mx-auto grid max-w-[1120px] gap-6 px-4 py-10 md:grid-cols-3 md:px-8">
        <div>
          <p className="font-heading text-xl font-extrabold text-berry">{SITE.brandName}</p>
          <p className="mt-1 text-ink/80">{SITE.tagline}</p>
        </div>
        <ul className="space-y-2 text-ink/90">
          {settings && (
            <>
              <li className="flex gap-2">
                <MapPin aria-hidden="true" className="mt-1 size-4 shrink-0 text-rose" />
                {settings.address}
              </li>
              <li className="flex gap-2">
                <Phone aria-hidden="true" className="mt-1 size-4 shrink-0 text-rose" />
                <a className="underline" href={`tel:${settings.phone}`}>
                  {settings.phone}
                </a>
              </li>
            </>
          )}
          {hours && <li>{summarizeHours(hours)}</li>}
        </ul>
        <ul className="space-y-1">
          <li>
            <Link className="underline" to={ROUTES.lookup}>
              Tra cứu lịch hẹn
            </Link>
          </li>
          <li>
            <Link className="underline" to={ROUTES.contact}>
              Liên hệ
            </Link>
          </li>
        </ul>
      </div>
    </footer>
  );
}
```

`src/app/CustomerLayout.tsx`:

```tsx
import { Outlet } from 'react-router-dom';
import { COPY } from '@/config/copy';
import { SiteFooter } from '@/components/layout/SiteFooter';
import { SiteHeader } from '@/components/layout/SiteHeader';

export function CustomerLayout() {
  return (
    <>
      <a
        href="#main"
        className="sr-only focus:not-sr-only focus:fixed focus:left-2 focus:top-2 focus:z-50 focus:rounded-control focus:bg-surface focus:px-4 focus:py-2"
      >
        {COPY.a11y.skip}
      </a>
      <SiteHeader />
      <main id="main" tabIndex={-1} className="outline-none">
        <Outlet />
      </main>
      <SiteFooter />
    </>
  );
}
```

`src/pages/NotFoundPage.tsx`:

```tsx
import { LinkButton } from '@/components/ui/Button';
import { ROUTES } from '@/config/routes';

export function NotFoundPage() {
  return (
    <div className="mx-auto flex max-w-[1120px] flex-col items-center gap-4 px-4 py-24 text-center">
      <h1 className="text-3xl font-extrabold">Không tìm thấy trang</h1>
      <p>Đường dẫn này không tồn tại hoặc đã đổi.</p>
      <LinkButton to={ROUTES.home}>Về trang chủ</LinkButton>
    </div>
  );
}
```

`src/app/router.tsx` (các trang được thêm dần ở các Task sau; hiện chỉ có 404 để khung chạy được):

```tsx
import { Suspense } from 'react';
import { createBrowserRouter, type RouteObject } from 'react-router-dom';
import { Skeleton } from '@/components/ui/Skeleton';
import { NotFoundPage } from '@/pages/NotFoundPage';
import { CustomerLayout } from './CustomerLayout';

const fallback = <Skeleton className="mx-auto my-16 h-64 max-w-[1120px]" />;

// Mỗi trang là một { path, element } dùng Suspense vì được tải lười.
export const page = (path: string, Page: React.ComponentType): RouteObject => ({
  path,
  element: (
    <Suspense fallback={fallback}>
      <Page />
    </Suspense>
  ),
});

export const customerRoutes: RouteObject[] = [];

export const router = createBrowserRouter([
  { element: <CustomerLayout />, children: [...customerRoutes, { path: '*', element: <NotFoundPage /> }] },
]);
```

`src/app/App.tsx`:

```tsx
import { QueryClientProvider } from '@tanstack/react-query';
import { RouterProvider } from 'react-router-dom';
import { Toaster } from 'sonner';
import { queryClient } from '@/lib/queryClient';
import { router } from './router';

export function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router} />
      <Toaster position="top-center" richColors />
    </QueryClientProvider>
  );
}
```

Thay `src/main.tsx`:

```tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import { App } from '@/app/App';
import './index.css';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

Xóa tệp `src/App.tsx` cũ nếu còn.

- [ ] **Step 5: Chạy để thấy qua**

Run: `npx vitest run src/features/catalog src/components/layout`
Expected: PASS (hours 3, SiteHeader 3). `SiteHeader` không cần Supabase nên không cần `.env.local`.

Run: `npm run check`
Expected: typecheck, lint, test qua.

- [ ] **Step 6: Commit**

```bash
git add -A
git commit -m "feat(app): router, layout khách, header/footer và hook danh mục"
```

---

## Task 3: Trang chủ và hero 3D có dự phòng

**Files:**
- Create: `src/features/hero/heroMode.ts`, `src/components/hero/HeroArt.tsx`, `Orbs2p5d.tsx`, `Hero3D.tsx`
- Create: `src/components/ServiceCard.tsx`, `src/components/Carousel.tsx`, `src/components/GalleryTile.tsx`
- Create: `src/pages/customer/HomePage.tsx`, `src/pages/customer/home/HeroSection.tsx`, `InfoStrip.tsx`, `FeaturedServices.tsx`, `FeaturedGallery.tsx`, `TeamSection.tsx`, `PromoStrip.tsx`
- Modify: `src/app/router.tsx`
- Test: `src/features/hero/heroMode.test.ts`

**Interfaces:**
- Consumes: hook danh mục (Task 2), `bookingUrl`, `GALLERY_STYLES`, `PriceText`, `DurationText`, `QueryState`.
- Produces:
  - `type HeroEnv`, `shouldUse3D(env: HeroEnv): boolean`, `readHeroEnv(): HeroEnv`
  - `HeroArt()`; `Hero3D({ onLowFps: () => void })`
  - `ServiceCard({ service })`, `GalleryTile({ item, onOpen? })`, `Carousel({ children })`
  - `HomePage`

- [ ] **Step 1: Viết test hỏng**

`src/features/hero/heroMode.test.ts`:

```ts
import { shouldUse3D, type HeroEnv } from './heroMode';

const strong: HeroEnv = {
  reducedMotion: false,
  saveData: false,
  hasWebGL: true,
  cores: 8,
  memoryGb: 8,
};

describe('shouldUse3D', () => {
  it('máy mạnh, không hạn chế thì dùng 3D', () => {
    expect(shouldUse3D(strong)).toBe(true);
  });

  it('người dùng bật giảm chuyển động thì không dùng 3D', () => {
    expect(shouldUse3D({ ...strong, reducedMotion: true })).toBe(false);
  });

  it('bật tiết kiệm dữ liệu thì không dùng 3D', () => {
    expect(shouldUse3D({ ...strong, saveData: true })).toBe(false);
  });

  it('không có WebGL thì không dùng 3D', () => {
    expect(shouldUse3D({ ...strong, hasWebGL: false })).toBe(false);
  });

  it('máy yếu (ít nhân hoặc ít RAM) thì không dùng 3D', () => {
    expect(shouldUse3D({ ...strong, cores: 2 })).toBe(false);
    expect(shouldUse3D({ ...strong, memoryGb: 2 })).toBe(false);
  });

  it('không biết dung lượng RAM (trình duyệt không báo) thì vẫn cho dùng', () => {
    expect(shouldUse3D({ ...strong, memoryGb: undefined })).toBe(true);
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/features/hero`
Expected: FAIL (không tìm thấy `./heroMode`).

- [ ] **Step 3: Logic chọn chế độ hero**

`src/features/hero/heroMode.ts`:

```ts
export type HeroEnv = {
  reducedMotion: boolean;
  saveData: boolean;
  hasWebGL: boolean;
  cores: number;
  memoryGb: number | undefined;
};

export function shouldUse3D(env: HeroEnv): boolean {
  if (env.reducedMotion || env.saveData || !env.hasWebGL) return false;
  if (env.cores <= 2) return false;
  if (env.memoryGb !== undefined && env.memoryGb <= 2) return false;
  return true;
}

type NavigatorExtras = Navigator & { deviceMemory?: number; connection?: { saveData?: boolean } };

export function readHeroEnv(): HeroEnv {
  const nav = navigator as NavigatorExtras;
  let hasWebGL = false;
  try {
    hasWebGL = !!document.createElement('canvas').getContext('webgl');
  } catch {
    hasWebGL = false;
  }
  return {
    reducedMotion: window.matchMedia('(prefers-reduced-motion: reduce)').matches,
    saveData: nav.connection?.saveData === true,
    hasWebGL,
    cores: nav.hardwareConcurrency ?? 4,
    memoryGb: nav.deviceMemory,
  };
}
```

- [ ] **Step 4: Chạy để thấy qua**

Run: `npx vitest run src/features/hero`
Expected: PASS (6 test).

- [ ] **Step 5: Hero 2.5D, 3D và bộ chọn**

`src/components/hero/Orbs2p5d.tsx`:

```tsx
// Bản dự phòng nhẹ: ba hạt gel bằng CSS, nổi nhẹ. Luôn hiển thị ngay, 3D phủ lên khi sẵn sàng.
export function Orbs2p5d() {
  return (
    <>
      <div className="absolute right-4 top-4 size-32 animate-float rounded-full bg-[radial-gradient(circle_at_30%_25%,#fff,var(--color-gel-pink)_40%,var(--color-plum-2))] shadow-pop motion-reduce:animate-none sm:size-44" />
      <div className="absolute bottom-6 right-28 size-16 animate-float rounded-full bg-[radial-gradient(circle_at_30%_25%,#fff,#7ee7ff_45%,#3b6fd8)] [animation-delay:-2s] motion-reduce:animate-none sm:size-24" />
      <div className="absolute bottom-10 right-6 size-10 animate-float rounded-full bg-[radial-gradient(circle_at_30%_25%,#fff,#ffd36e_50%,#e08a1e)] [animation-delay:-4s] motion-reduce:animate-none sm:size-14" />
    </>
  );
}
```

`src/components/hero/Hero3D.tsx`:

```tsx
import { useEffect, useRef } from 'react';
import * as THREE from 'three';

type Props = { onLowFps: () => void };

const LOW_FPS = 30;
const SAMPLE_MS = 1500;

export function Hero3D({ onLowFps }: Props) {
  const host = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const el = host.current;
    if (!el) return;

    const renderer = new THREE.WebGLRenderer({ alpha: true, antialias: true });
    renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
    renderer.setSize(el.clientWidth, el.clientHeight);
    el.appendChild(renderer.domElement);

    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(40, el.clientWidth / el.clientHeight, 0.1, 100);
    camera.position.z = 9;
    scene.add(new THREE.AmbientLight(0xffffff, 0.55));
    const pink = new THREE.PointLight(0xff7ad9, 1.6, 30);
    pink.position.set(-5, 4, 6);
    const cyan = new THREE.PointLight(0x7ee7ff, 1.4, 30);
    cyan.position.set(6, -3, 5);
    const white = new THREE.DirectionalLight(0xffffff, 0.9);
    white.position.set(2, 5, 6);
    scene.add(pink, cyan, white);

    const gloss = (color: number) =>
      new THREE.MeshPhysicalMaterial({ color, roughness: 0.12, metalness: 0.05, clearcoat: 1, clearcoatRoughness: 0.04 });
    const bodies = [
      { geo: new THREE.SphereGeometry(1.5, 48, 48), mat: gloss(0xff5fd0), x: 2.6, y: 0.6, speed: 1 },
      { geo: new THREE.SphereGeometry(0.85, 48, 48), mat: gloss(0x6fdcff), x: 0.4, y: -2.2, speed: 1.4 },
      { geo: new THREE.SphereGeometry(0.55, 48, 48), mat: gloss(0xffc24d), x: 2.9, y: -2.4, speed: 1.9 },
      {
        geo: new THREE.OctahedronGeometry(0.7, 0),
        mat: new THREE.MeshPhysicalMaterial({ color: 0xd8f4ff, roughness: 0.05, metalness: 0.6, clearcoat: 1, flatShading: true }),
        x: 0.2,
        y: 2.4,
        speed: 1.2,
      },
    ].map((b, i) => {
      const mesh = new THREE.Mesh(b.geo, b.mat);
      mesh.position.set(b.x, b.y, 0);
      scene.add(mesh);
      return { ...b, mesh, phase: i * 1.7 };
    });

    let pointerX = 0;
    let pointerY = 0;
    const onPointer = (e: PointerEvent) => {
      const box = el.getBoundingClientRect();
      pointerX = ((e.clientX - box.left) / box.width - 0.5) * 2;
      pointerY = ((e.clientY - box.top) / box.height - 0.5) * 2;
    };
    window.addEventListener('pointermove', onPointer);

    let raf = 0;
    let frames = 0;
    let sampleStart = performance.now();
    let checked = false;
    const loop = (now: number) => {
      const t = now / 1000;
      for (const b of bodies) {
        b.mesh.position.y = b.y + Math.sin(t * b.speed + b.phase) * 0.35;
        b.mesh.rotation.y = t * 0.5 * b.speed;
        b.mesh.rotation.x = t * 0.3 * b.speed;
      }
      camera.position.x += (pointerX * 1.2 - camera.position.x) * 0.06;
      camera.position.y += (-pointerY * 0.8 - camera.position.y) * 0.06;
      camera.lookAt(0, 0, 0);
      renderer.render(scene, camera);

      if (document.hidden) {
        sampleStart = now; // tab ẩn thì không đo, tránh kết luận sai là máy yếu
        frames = 0;
      } else if (!checked) {
        frames += 1;
        if (now - sampleStart > SAMPLE_MS) {
          checked = true;
          if ((frames * 1000) / (now - sampleStart) < LOW_FPS) onLowFps();
        }
      }
      raf = requestAnimationFrame(loop);
    };
    raf = requestAnimationFrame(loop);

    return () => {
      cancelAnimationFrame(raf);
      window.removeEventListener('pointermove', onPointer);
      for (const b of bodies) {
        b.geo.dispose();
        b.mat.dispose();
      }
      renderer.dispose();
      renderer.domElement.remove();
    };
  }, [onLowFps]);

  return <div ref={host} className="absolute inset-0" />;
}
```

`src/components/hero/HeroArt.tsx`:

```tsx
import { lazy, Suspense, useCallback, useEffect, useState } from 'react';
import { readHeroEnv, shouldUse3D } from '@/features/hero/heroMode';
import { Orbs2p5d } from './Orbs2p5d';

const Hero3D = lazy(() => import('./Hero3D').then((m) => ({ default: m.Hero3D })));

// Bản 2.5D hiện ngay. 3D chỉ tải sau khi trang đã vẽ và chỉ khi máy cho phép;
// nếu đo thấy giật (dưới 30 FPS) thì tự gỡ 3D, về lại 2.5D.
export function HeroArt() {
  const [use3d, setUse3d] = useState(false);
  useEffect(() => setUse3d(shouldUse3D(readHeroEnv())), []);
  const fallBack = useCallback(() => setUse3d(false), []);

  return (
    <div aria-hidden="true" className="relative h-64 sm:h-96">
      <Orbs2p5d />
      {use3d && (
        <Suspense fallback={null}>
          <Hero3D onLowFps={fallBack} />
        </Suspense>
      )}
    </div>
  );
}
```

- [ ] **Step 6: Component dùng lại và các khối trang chủ**

`src/components/ServiceCard.tsx`:

```tsx
import { bookingUrl } from '@/config/routes';
import { COPY } from '@/config/copy';
import type { Service } from '@/types/db';
import { DurationText } from './DurationText';
import { PriceText } from './PriceText';
import { LinkButton } from './ui/Button';

export function ServiceCard({ service }: { service: Service }) {
  return (
    <article className="flex flex-col gap-2 rounded-card border border-line bg-surface p-4 shadow-card">
      <h3 className="text-lg font-bold">{service.name}</h3>
      <div className="flex items-center justify-between">
        <PriceText value={service.price_vnd} />
        <DurationText minutes={service.duration_min} />
      </div>
      <LinkButton variant="secondary" size="sm" className="mt-2" to={bookingUrl({ serviceId: service.id })}>
        {COPY.actions.book}
      </LinkButton>
    </article>
  );
}
```

`src/components/GalleryTile.tsx`:

```tsx
import { GALLERY_STYLES, type GalleryStyle } from '@/config/gallery';
import { formatVnd } from '@/lib/format';
import type { GalleryItem } from '@/types/db';

// Chưa có ảnh thật thì hiện ô gradient theo kiểu nail.
export function GalleryTile({ item, onOpen }: { item: GalleryItem; onOpen?: () => void }) {
  const style = GALLERY_STYLES[item.style_tag as GalleryStyle];
  const content = (
    <>
      {item.image_url ? (
        <img src={item.image_url} alt={item.title} loading="lazy" className="size-full object-cover" />
      ) : (
        <div
          className="size-full"
          style={style ? { background: `linear-gradient(135deg, ${style.from}, ${style.to})` } : undefined}
        />
      )}
      <span className="absolute inset-x-2 bottom-2 rounded-full bg-plum-1/70 px-3 py-1 text-sm text-white">
        {item.title}
        {item.price_hint_vnd ? `, ${formatVnd(item.price_hint_vnd)}` : ''}
      </span>
    </>
  );
  const className = 'relative block aspect-[4/5] w-full overflow-hidden rounded-card';
  return onOpen ? (
    <button type="button" onClick={onOpen} aria-label={`Xem mẫu ${item.title}`} className={className}>
      {content}
    </button>
  ) : (
    <div className={className}>{content}</div>
  );
}
```

`src/components/Carousel.tsx`:

```tsx
import useEmblaCarousel from 'embla-carousel-react';
import type { ReactNode } from 'react';

// Vuốt ngang, không tự chạy (người dùng không phải tìm nút tạm dừng).
export function Carousel({ children }: { children: ReactNode }) {
  const [viewport] = useEmblaCarousel({ align: 'start', dragFree: true });
  return (
    <div ref={viewport} className="overflow-hidden">
      <div className="flex gap-3">{children}</div>
    </div>
  );
}
```

`src/pages/customer/home/HeroSection.tsx`:

```tsx
import { HeroArt } from '@/components/hero/HeroArt';
import { LinkButton } from '@/components/ui/Button';
import { ROUTES } from '@/config/routes';

export function HeroSection() {
  return (
    <section className="relative overflow-hidden bg-gradient-to-br from-plum-1 via-plum-1 to-plum-2 text-white">
      <div className="mx-auto grid max-w-[1120px] items-center gap-4 px-4 pb-16 pt-10 md:grid-cols-2 md:px-8">
        <div>
          <h1 className="text-4xl font-extrabold leading-tight text-white md:text-5xl">
            Móng đẹp, đặt lịch trong 30 giây
          </h1>
          <p className="mt-3 max-w-md text-white/85">
            Chọn dịch vụ, thợ và giờ còn trống. Không cần tài khoản, có mã lịch để tra cứu hoặc hủy.
          </p>
          <div className="mt-6 flex flex-wrap gap-3">
            <LinkButton variant="gel" to={ROUTES.booking}>
              Chọn dịch vụ
            </LinkButton>
            <LinkButton variant="secondary" to={ROUTES.gallery}>
              Xem mẫu nail
            </LinkButton>
          </div>
        </div>
        <HeroArt />
      </div>
      <div aria-hidden="true" className="absolute inset-x-0 bottom-0 h-8 rounded-t-[50%] bg-cream" />
    </section>
  );
}
```

`src/pages/customer/home/InfoStrip.tsx`:

```tsx
import { Clock, MapPin, Phone } from 'lucide-react';
import { summarizeHours } from '@/features/catalog/hours';
import { useOpeningHours, useShopSettings } from '@/features/catalog/queries';

export function InfoStrip() {
  const settings = useShopSettings().data;
  const hours = useOpeningHours().data;
  if (!settings) return null;
  const items = [
    { icon: Clock, text: hours ? summarizeHours(hours) : '' },
    { icon: MapPin, text: settings.address },
    { icon: Phone, text: settings.phone, href: `tel:${settings.phone}` },
  ];
  return (
    <section className="mx-auto -mt-2 max-w-[1120px] px-4 md:px-8">
      <ul className="grid gap-3 rounded-card border border-line bg-surface p-4 shadow-card md:grid-cols-3">
        {items.map(({ icon: Icon, text, href }) => (
          <li key={text} className="flex items-center gap-3">
            <Icon aria-hidden="true" className="size-5 shrink-0 text-rose" />
            {href ? <a className="underline" href={href}>{text}</a> : <span>{text}</span>}
          </li>
        ))}
      </ul>
    </section>
  );
}
```

`src/pages/customer/home/FeaturedServices.tsx`:

```tsx
import { Link } from 'react-router-dom';
import { QueryState } from '@/components/QueryState';
import { ServiceCard } from '@/components/ServiceCard';
import { ROUTES } from '@/config/routes';
import { useServices } from '@/features/catalog/queries';

const FEATURED_COUNT = 4;

export function FeaturedServices() {
  const query = useServices();
  return (
    <section className="mx-auto mt-12 max-w-[1120px] px-4 md:px-8">
      <div className="flex items-end justify-between">
        <h2 className="text-2xl font-extrabold">Dịch vụ phổ biến</h2>
        <Link className="font-medium text-rose underline" to={ROUTES.services}>
          Xem toàn bộ bảng giá
        </Link>
      </div>
      <div className="mt-4 grid gap-4 sm:grid-cols-2 lg:grid-cols-4">
        <QueryState query={query}>
          {(services) => services.slice(0, FEATURED_COUNT).map((s) => <ServiceCard key={s.id} service={s} />)}
        </QueryState>
      </div>
    </section>
  );
}
```

`src/pages/customer/home/FeaturedGallery.tsx`:

```tsx
import { Link } from 'react-router-dom';
import { Carousel } from '@/components/Carousel';
import { GalleryTile } from '@/components/GalleryTile';
import { QueryState } from '@/components/QueryState';
import { bookingUrl, ROUTES } from '@/config/routes';
import { useGallery } from '@/features/catalog/queries';

export function FeaturedGallery() {
  const query = useGallery();
  return (
    <section className="mx-auto mt-12 max-w-[1120px] px-4 md:px-8">
      <div className="flex items-end justify-between">
        <h2 className="text-2xl font-extrabold">Mẫu nail nổi bật</h2>
        <Link className="font-medium text-rose underline" to={ROUTES.gallery}>
          Xem tất cả mẫu
        </Link>
      </div>
      <div className="mt-4">
        <QueryState query={query}>
          {(items) => (
            <Carousel>
              {items
                .filter((i) => i.featured)
                .map((item) => (
                  <div key={item.id} className="w-56 shrink-0">
                    <GalleryTile item={item} />
                    <Link
                      className="mt-2 block min-h-11 content-center text-center font-medium text-rose underline"
                      to={bookingUrl({ designId: item.id })}
                    >
                      Đặt mẫu này
                    </Link>
                  </div>
                ))}
            </Carousel>
          )}
        </QueryState>
      </div>
    </section>
  );
}
```

`src/pages/customer/home/TeamSection.tsx`:

```tsx
import { QueryState } from '@/components/QueryState';
import { useStaff } from '@/features/catalog/queries';

export function TeamSection() {
  const query = useStaff();
  return (
    <section className="mx-auto mt-12 max-w-[1120px] px-4 md:px-8">
      <h2 className="text-2xl font-extrabold">Đội ngũ thợ</h2>
      <div className="mt-4 grid gap-4 sm:grid-cols-3">
        <QueryState query={query}>
          {(staff) =>
            staff.map((s) => (
              <article key={s.id} className="rounded-card border border-line bg-surface p-4 shadow-card">
                <div className="grid size-14 place-items-center rounded-full bg-blush font-heading text-xl font-extrabold text-rose">
                  {s.name.charAt(0)}
                </div>
                <h3 className="mt-3 text-lg font-bold">{s.name}</h3>
                {s.bio && <p className="mt-1 text-ink/80">{s.bio}</p>}
              </article>
            ))
          }
        </QueryState>
      </div>
    </section>
  );
}
```

`src/pages/customer/home/PromoStrip.tsx`:

```tsx
export function PromoStrip() {
  return (
    <section className="mx-auto mt-12 max-w-[1120px] px-4 md:px-8">
      <div className="grid gap-4 rounded-card bg-blush p-6 md:grid-cols-2">
        <div>
          <h2 className="text-xl font-extrabold">Khách mới giảm 15%</h2>
          <p className="mt-1">
            Nhập mã <strong>LUA15</strong> khi đặt lịch (đơn từ 200.000đ).
          </p>
        </div>
        <div>
          <h2 className="text-xl font-extrabold">Tích điểm mỗi lần làm</h2>
          <p className="mt-1">Mỗi 10.000đ được 1 điểm. Đăng ký bằng SĐT để xem điểm của bạn.</p>
        </div>
      </div>
    </section>
  );
}
```

`src/pages/customer/HomePage.tsx`:

```tsx
import { FeaturedGallery } from './home/FeaturedGallery';
import { FeaturedServices } from './home/FeaturedServices';
import { HeroSection } from './home/HeroSection';
import { InfoStrip } from './home/InfoStrip';
import { PromoStrip } from './home/PromoStrip';
import { TeamSection } from './home/TeamSection';

export function HomePage() {
  return (
    <>
      <HeroSection />
      <InfoStrip />
      <FeaturedServices />
      <FeaturedGallery />
      <TeamSection />
      <PromoStrip />
    </>
  );
}
```

- [ ] **Step 7: Đăng ký route trang chủ**

Trong `src/app/router.tsx`, thêm import và đổi `customerRoutes`:

```tsx
import { lazyPage } from './lazyPage';
import { ROUTES } from '@/config/routes';

const HomePage = lazyPage(() => import('@/pages/customer/HomePage'), 'HomePage');

export const customerRoutes: RouteObject[] = [page(ROUTES.home, HomePage)];
```

- [ ] **Step 8: Kiểm tra bằng trình duyệt**

Chuẩn bị: điền `VITE_SUPABASE_URL` và `VITE_SUPABASE_ANON_KEY` của project **thử nghiệm** trong `.env.local`, đã chạy `npm run db:seed`.

Run: `npm run dev` rồi mở `http://localhost:5173`.
Expected: thấy hero tím có hạt gel (3D sau một lúc nếu máy mạnh), dải giờ mở cửa và địa chỉ, 4 dịch vụ, carousel mẫu, 3 thợ, dải khuyến mãi. Thu cửa sổ về 375px: một cột, nút ☰ mở ngăn menu, không cuộn ngang. Bật "giảm chuyển động" trong hệ điều hành thì chỉ còn hạt gel 2.5D.

Run: `npm run check`
Expected: qua.

- [ ] **Step 9: Commit**

```bash
git add -A
git commit -m "feat(home): trang chủ với hero 3D có dự phòng 2.5D"
```

---

## Task 4: Bảng giá dịch vụ

**Files:**
- Create: `src/components/ServiceRow.tsx`, `src/pages/customer/ServicesPage.tsx`
- Modify: `src/app/router.tsx`
- Test: `src/pages/customer/ServicesPage.test.tsx`

**Interfaces:**
- Consumes: `useServices`, `useCategories`, `TabBar`, `QueryState`, `bookingUrl`.
- Produces: `ServiceRow({ service, onSelect? })` (có `onSelect` thì là nút, không thì là liên kết sang `/dat-lich?dich-vu=<id>`); `ServicesPage`.

- [ ] **Step 1: Viết test hỏng**

`src/pages/customer/ServicesPage.test.tsx`:

```tsx
import { render, screen, within } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MemoryRouter } from 'react-router-dom';
import type { Service, ServiceCategory } from '@/types/db';
import { ServicesPage } from './ServicesPage';

const categories: ServiceCategory[] = [
  { id: 'c1', name: 'Tay', slug: 'tay', sort: 1 },
  { id: 'c2', name: 'Gel', slug: 'gel', sort: 2 },
];
const service = (id: string, category_id: string, name: string, price: number): Service => ({
  id, category_id, name, description: null, price_vnd: price, duration_min: 60,
  image_url: null, active: true, sort: 1,
});
const services = [service('s1', 'c1', 'Sơn thường', 120000), service('s2', 'c2', 'Gel tay', 350000)];

vi.mock('@/features/catalog/queries', () => ({
  useCategories: () => ({ isPending: false, isError: false, data: categories }),
  useServices: () => ({ isPending: false, isError: false, data: services }),
}));

const renderPage = () => render(<MemoryRouter><ServicesPage /></MemoryRouter>);

describe('ServicesPage', () => {
  it('mặc định hiện mọi dịch vụ với giá VNĐ và thời lượng', () => {
    renderPage();
    expect(screen.getByText('Sơn thường')).toBeInTheDocument();
    expect(screen.getByText('Gel tay')).toBeInTheDocument();
    expect(screen.getByText('350.000đ')).toBeInTheDocument();
    expect(screen.getAllByText('60 phút').length).toBe(2);
  });

  it('chọn tab nhóm thì chỉ còn dịch vụ của nhóm đó', async () => {
    renderPage();
    await userEvent.click(screen.getByRole('tab', { name: /Gel/ }));
    expect(screen.queryByText('Sơn thường')).not.toBeInTheDocument();
    expect(screen.getByText('Gel tay')).toBeInTheDocument();
  });

  it('nút Chọn dẫn sang đặt lịch với dịch vụ điền sẵn', () => {
    renderPage();
    const row = screen.getByText('Gel tay').closest('li')!;
    expect(within(row).getByRole('link', { name: 'Chọn' })).toHaveAttribute('href', '/dat-lich?dich-vu=s2');
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/pages/customer/ServicesPage.test.tsx`
Expected: FAIL (không tìm thấy `./ServicesPage`).

- [ ] **Step 3: Viết code**

`src/components/ServiceRow.tsx`:

```tsx
import { bookingUrl } from '@/config/routes';
import type { Service } from '@/types/db';
import { DurationText } from './DurationText';
import { PriceText } from './PriceText';
import { Button, LinkButton } from './ui/Button';

type Props = { service: Service; onSelect?: (service: Service) => void; selected?: boolean };

// Dùng ở bảng giá (liên kết sang đặt lịch) và ở bước chọn dịch vụ (nút chọn tại chỗ).
export function ServiceRow({ service, onSelect, selected }: Props) {
  return (
    <li className="flex items-center justify-between gap-3 rounded-card border border-line bg-surface p-4 shadow-card">
      <div className="min-w-0">
        <h3 className="font-bold text-berry">{service.name}</h3>
        {service.description && <p className="text-sm text-ink/80">{service.description}</p>}
        <div className="mt-1 flex items-center gap-3">
          <PriceText value={service.price_vnd} />
          <DurationText minutes={service.duration_min} />
        </div>
      </div>
      {onSelect ? (
        <Button size="sm" variant={selected ? 'primary' : 'secondary'} aria-pressed={selected} onClick={() => onSelect(service)}>
          {selected ? 'Đã chọn' : 'Chọn'}
        </Button>
      ) : (
        <LinkButton size="sm" variant="secondary" to={bookingUrl({ serviceId: service.id })}>
          Chọn
        </LinkButton>
      )}
    </li>
  );
}
```

`src/pages/customer/ServicesPage.tsx`:

```tsx
import { useState } from 'react';
import { QueryState } from '@/components/QueryState';
import { ServiceRow } from '@/components/ServiceRow';
import { TabBar } from '@/components/ui/TabBar';
import { EmptyState } from '@/components/ui/EmptyState';
import { useCategories, useServices } from '@/features/catalog/queries';

const ALL = 'all';

export function ServicesPage() {
  const [category, setCategory] = useState(ALL);
  const services = useServices();
  const categories = useCategories().data ?? [];

  const countOf = (id: string) => (services.data ?? []).filter((s) => s.category_id === id).length;
  const tabs = [
    { value: ALL, label: 'Tất cả', count: services.data?.length },
    ...categories.map((c) => ({ value: c.id, label: c.name, count: countOf(c.id) })),
  ];

  return (
    <div className="mx-auto max-w-[1120px] px-4 py-10 md:px-8">
      <h1 className="text-3xl font-extrabold">Dịch vụ và bảng giá</h1>
      <div className="mt-6">
        <TabBar value={category} onValueChange={setCategory} items={tabs} />
      </div>
      <ul className="mt-6 grid gap-4 md:grid-cols-2">
        <QueryState query={services}>
          {(all) => {
            const shown = category === ALL ? all : all.filter((s) => s.category_id === category);
            if (shown.length === 0) return <EmptyState title="Nhóm này chưa có dịch vụ" />;
            return shown.map((s) => <ServiceRow key={s.id} service={s} />);
          }}
        </QueryState>
      </ul>
    </div>
  );
}
```

Trong `src/app/router.tsx` thêm:

```tsx
const ServicesPage = lazyPage(() => import('@/pages/customer/ServicesPage'), 'ServicesPage');
// ...và thêm vào customerRoutes:
page(ROUTES.services, ServicesPage),
```

- [ ] **Step 4: Chạy để thấy qua**

Run: `npx vitest run src/pages/customer/ServicesPage.test.tsx`
Expected: PASS (3 test).

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat(services): trang bảng giá theo nhóm"
```

---

## Task 5: Gallery mẫu nail

**Files:**
- Create: `src/pages/customer/GalleryPage.tsx`, `src/components/GalleryLightbox.tsx`
- Modify: `src/app/router.tsx`
- Test: `src/pages/customer/GalleryPage.test.tsx`

**Interfaces:**
- Consumes: `useGallery`, `GALLERY_STYLES`, `GalleryTile`, `Chip`, `Dialog`, `bookingUrl`.
- Produces: `GalleryLightbox({ items, index, onIndexChange, onClose })`; `GalleryPage`.

- [ ] **Step 1: Viết test hỏng**

`src/pages/customer/GalleryPage.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { MemoryRouter } from 'react-router-dom';
import type { GalleryItem } from '@/types/db';
import { GalleryPage } from './GalleryPage';

const item = (id: string, style_tag: string, title: string): GalleryItem => ({
  id, title, style_tag, image_url: null, price_hint_vnd: 380000, featured: false, active: true, sort: 1,
});
const items = [item('g1', 'ombre', 'Ombre hồng'), item('g2', 'french', 'French trắng')];

vi.mock('@/features/catalog/queries', () => ({
  useGallery: () => ({ isPending: false, isError: false, data: items }),
}));

const renderPage = () => render(<MemoryRouter><GalleryPage /></MemoryRouter>);

describe('GalleryPage', () => {
  it('lọc theo kiểu: chọn French thì ẩn Ombre', async () => {
    renderPage();
    expect(screen.getByRole('button', { name: /Ombre hồng/ })).toBeInTheDocument();
    await userEvent.click(screen.getByRole('button', { name: 'French' }));
    expect(screen.queryByRole('button', { name: /Ombre hồng/ })).not.toBeInTheDocument();
    expect(screen.getByRole('button', { name: /French trắng/ })).toBeInTheDocument();
  });

  it('bấm ô mở khung xem lớn có nút Đặt mẫu này dẫn tới đặt lịch với mẫu đó', async () => {
    renderPage();
    await userEvent.click(screen.getByRole('button', { name: /Ombre hồng/ }));
    expect(screen.getByRole('dialog')).toBeInTheDocument();
    expect(screen.getByRole('link', { name: 'Đặt mẫu này' })).toHaveAttribute('href', '/dat-lich?mau=g1');
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/pages/customer/GalleryPage.test.tsx`
Expected: FAIL.

- [ ] **Step 3: Viết code**

`src/components/GalleryLightbox.tsx`:

```tsx
import { ChevronLeft, ChevronRight } from 'lucide-react';
import { GalleryTile } from '@/components/GalleryTile';
import { Button, LinkButton } from '@/components/ui/Button';
import { Dialog } from '@/components/ui/Dialog';
import { bookingUrl } from '@/config/routes';
import type { GalleryItem } from '@/types/db';

type Props = {
  items: GalleryItem[];
  index: number | null;
  onIndexChange: (index: number) => void;
  onClose: () => void;
};

export function GalleryLightbox({ items, index, onIndexChange, onClose }: Props) {
  const item = index === null ? undefined : items[index];
  const step = (d: number) => onIndexChange((index! + d + items.length) % items.length);
  return (
    <Dialog open={!!item} onOpenChange={(open) => !open && onClose()} title={item?.title ?? 'Mẫu nail'}>
      {item && (
        <div className="flex flex-col gap-4">
          <GalleryTile item={item} />
          <div className="flex items-center justify-between gap-2">
            <Button variant="ghost" aria-label="Mẫu trước" onClick={() => step(-1)}>
              <ChevronLeft aria-hidden="true" />
            </Button>
            <LinkButton to={bookingUrl({ designId: item.id })}>Đặt mẫu này</LinkButton>
            <Button variant="ghost" aria-label="Mẫu sau" onClick={() => step(1)}>
              <ChevronRight aria-hidden="true" />
            </Button>
          </div>
        </div>
      )}
    </Dialog>
  );
}
```

`src/pages/customer/GalleryPage.tsx`:

```tsx
import { useState } from 'react';
import { GalleryLightbox } from '@/components/GalleryLightbox';
import { GalleryTile } from '@/components/GalleryTile';
import { QueryState } from '@/components/QueryState';
import { Button } from '@/components/ui/Button';
import { Chip } from '@/components/ui/Chip';
import { EmptyState } from '@/components/ui/EmptyState';
import { GALLERY_STYLES } from '@/config/gallery';
import { useGallery } from '@/features/catalog/queries';

const ALL = 'all';

export function GalleryPage() {
  const [style, setStyle] = useState(ALL);
  const [open, setOpen] = useState<number | null>(null);
  const query = useGallery();

  return (
    <div className="mx-auto max-w-[1120px] px-4 py-10 md:px-8">
      <h1 className="text-3xl font-extrabold">Mẫu nail</h1>
      <div className="mt-6 flex gap-2 overflow-x-auto pb-1">
        <Chip active={style === ALL} onClick={() => setStyle(ALL)}>Tất cả</Chip>
        {Object.entries(GALLERY_STYLES).map(([key, def]) => (
          <Chip key={key} active={style === key} onClick={() => setStyle(key)}>
            {def.label}
          </Chip>
        ))}
      </div>
      <div className="mt-6">
        <QueryState query={query}>
          {(all) => {
            const shown = style === ALL ? all : all.filter((i) => i.style_tag === style);
            if (shown.length === 0) {
              return (
                <EmptyState
                  title="Chưa có mẫu kiểu này"
                  action={<Button variant="secondary" onClick={() => setStyle(ALL)}>Xem tất cả mẫu</Button>}
                />
              );
            }
            return (
              <>
                <ul className="grid grid-cols-2 gap-3 md:grid-cols-3 lg:grid-cols-4">
                  {shown.map((item, i) => (
                    <li key={item.id}>
                      <GalleryTile item={item} onOpen={() => setOpen(i)} />
                    </li>
                  ))}
                </ul>
                <GalleryLightbox items={shown} index={open} onIndexChange={setOpen} onClose={() => setOpen(null)} />
              </>
            );
          }}
        </QueryState>
      </div>
    </div>
  );
}
```

Trong `src/app/router.tsx` thêm:

```tsx
const GalleryPage = lazyPage(() => import('@/pages/customer/GalleryPage'), 'GalleryPage');
// ...và thêm vào customerRoutes:
page(ROUTES.gallery, GalleryPage),
```

- [ ] **Step 4: Chạy để thấy qua**

Run: `npx vitest run src/pages/customer/GalleryPage.test.tsx`
Expected: PASS (2 test).

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat(gallery): lưới mẫu nail, lọc theo kiểu và khung xem lớn"
```

---

## Task 6: Logic đặt lịch (bước, giờ trống, gọi API)

**Files:**
- Create: `src/features/booking/draft.ts`, `src/features/booking/availability.ts`, `src/features/booking/types.ts`
- Create: `src/lib/api/bookings.ts`, `src/features/booking/queries.ts`
- Test: `src/features/booking/draft.test.ts`, `src/features/booking/availability.test.ts`

**Interfaces:**
- Consumes: `BOOKING_STEPS`, `computeSlots` và các hàm ở `slots.ts`, `weekdayOf`, kiểu `db.ts`.
- Produces:
  - `type BookingDraft = { step: BookingStepId; service_id?: string; staff_id: string | null; day?: string; minutes?: number }`
  - `initialDraft(serviceId?: string): BookingDraft`, `bookingReducer(state, action): BookingDraft`, `canEnter(draft, step): boolean`
  - `type BookingAction = { type: 'service'; id: string } | { type: 'staff'; id: string | null } | { type: 'day'; day: string } | { type: 'time'; minutes: number } | { type: 'go'; step: BookingStepId } | { type: 'next' } | { type: 'back' }`
  - `computeDayAvailability(input): { open: Interval | null; grid: SlotView[] }`
  - `type BookingView`, `type BookingReceipt` (trong `types.ts`)
  - API: `fetchBusy(day)`, `createBooking(input)`, `lookupBooking(phone, code)`, `cancelBooking(phone, code)`, `validateCoupon(code, total)`
  - Hook: `useBusy(day)`, `useAvailability(args)`, `useCreateBooking()`, `useLookupBooking()`, `useCancelBooking()`, `useValidateCoupon()`

- [ ] **Step 1: Viết test hỏng cho các bước đặt lịch**

`src/features/booking/draft.test.ts`:

```ts
import { bookingReducer, canEnter, initialDraft, type BookingDraft } from './draft';

describe('bookingReducer', () => {
  it('bắt đầu ở bước Dịch vụ, mặc định thợ "bất kỳ" (null)', () => {
    expect(initialDraft()).toEqual({ step: 'service', staff_id: null });
  });

  it('vào từ trang bảng giá với dịch vụ điền sẵn thì nhảy sang bước Thợ', () => {
    expect(initialDraft('s1')).toEqual({ step: 'staff', service_id: 's1', staff_id: null });
  });

  it('chọn lần lượt dịch vụ, thợ, ngày, giờ thì tự sang bước kế tiếp', () => {
    let d = initialDraft();
    d = bookingReducer(d, { type: 'service', id: 's1' });
    expect(d.step).toBe('staff');
    d = bookingReducer(d, { type: 'staff', id: 'mai' });
    expect(d.step).toBe('date');
    d = bookingReducer(d, { type: 'day', day: '2030-03-05' });
    expect(d.step).toBe('time');
    d = bookingReducer(d, { type: 'time', minutes: 600 });
    expect(d).toEqual({ step: 'contact', service_id: 's1', staff_id: 'mai', day: '2030-03-05', minutes: 600 });
  });

  it('đổi dịch vụ, thợ hoặc ngày thì bỏ giờ đã chọn (giờ cũ có thể không còn hợp lệ)', () => {
    const full: BookingDraft = { step: 'contact', service_id: 's1', staff_id: 'mai', day: '2030-03-05', minutes: 600 };
    expect(bookingReducer(full, { type: 'service', id: 's2' }).minutes).toBeUndefined();
    expect(bookingReducer(full, { type: 'staff', id: null }).minutes).toBeUndefined();
    expect(bookingReducer(full, { type: 'day', day: '2030-03-06' }).minutes).toBeUndefined();
  });

  it('không cho nhảy tới bước chưa đủ điều kiện', () => {
    const start = initialDraft();
    expect(bookingReducer(start, { type: 'go', step: 'contact' }).step).toBe('service');
    expect(canEnter(start, 'time')).toBe(false);
    expect(canEnter(initialDraft('s1'), 'date')).toBe(true);
  });

  it('back và next đi theo thứ tự bước, back ở bước đầu thì đứng yên', () => {
    const d = initialDraft('s1');
    expect(bookingReducer(d, { type: 'back' }).step).toBe('service');
    expect(bookingReducer(initialDraft(), { type: 'back' }).step).toBe('service');
    expect(bookingReducer(d, { type: 'next' }).step).toBe('date');
  });
});
```

- [ ] **Step 2: Viết test hỏng cho giờ trống theo ngày**

`src/features/booking/availability.test.ts`:

```ts
import type { OpeningHours, Staff, StaffSchedule } from '@/types/db';
import { computeDayAvailability } from './availability';

const DAY = '2030-03-04'; // thứ Hai
const staff = (id: string): Staff => ({ id, name: id, bio: null, avatar_url: null, active: true, sort: 1 });
const hours = (weekday: number, closed = false): OpeningHours => ({
  weekday, open_time: '09:00:00', close_time: '12:00:00', is_closed: closed,
});
const sched = (staff_id: string, off = false): StaffSchedule => ({
  staff_id, weekday: 1, start_time: '09:00:00', end_time: '12:00:00', is_off: off,
});
const base = {
  day: DAY,
  nowIso: '2030-03-01T08:00:00+07:00',
  durationMin: 60,
  settings: { slot_minutes: 30, lead_minutes: 60 },
  staff: [staff('mai'), staff('linh')],
  openingHours: [hours(1)],
  schedules: [sched('mai'), sched('linh')],
  busy: [] as { staff_id: string; start_at: string; end_at: string }[],
};
const times = (r: ReturnType<typeof computeDayAvailability>) =>
  r.grid.filter((s) => s.available).map((s) => s.minutes);

describe('computeDayAvailability', () => {
  it('không có lịch bận: mọi giờ trên lưới đều trống, kể cả giờ kết thúc đúng lúc đóng cửa', () => {
    const r = computeDayAvailability({ ...base, staffId: null });
    expect(times(r)).toEqual([540, 570, 600, 630, 660]);
    expect(r.grid.map((s) => s.minutes)).toEqual([540, 570, 600, 630, 660]);
  });

  it('thợ cụ thể bận: giờ đó mờ; thợ "bất kỳ": còn trống nếu có thợ khác rảnh', () => {
    const busy = [{ staff_id: 'mai', start_at: '2030-03-04T10:00:00+07:00', end_at: '2030-03-04T11:00:00+07:00' }];
    expect(times(computeDayAvailability({ ...base, staffId: 'mai', busy }))).toEqual([540, 660]);
    expect(times(computeDayAvailability({ ...base, staffId: null, busy }))).toEqual([540, 570, 600, 630, 660]);
  });

  it('cả hai thợ cùng bận một giờ thì giờ đó hết chỗ với "bất kỳ"', () => {
    const slot = { start_at: '2030-03-04T10:00:00+07:00', end_at: '2030-03-04T11:00:00+07:00' };
    const busy = [{ staff_id: 'mai', ...slot }, { staff_id: 'linh', ...slot }];
    expect(times(computeDayAvailability({ ...base, staffId: null, busy }))).toEqual([540, 660]);
  });

  it('ngày tiệm nghỉ: không có lưới giờ', () => {
    const r = computeDayAvailability({ ...base, staffId: null, openingHours: [hours(1, true)] });
    expect(r.open).toBeNull();
    expect(r.grid).toEqual([]);
  });

  it('thợ được chọn nghỉ hôm đó: lưới còn nhưng không giờ nào trống', () => {
    const r = computeDayAvailability({ ...base, staffId: 'mai', schedules: [sched('mai', true), sched('linh')] });
    expect(r.grid.length).toBeGreaterThan(0);
    expect(times(r)).toEqual([]);
  });

  it('hôm nay: bỏ giờ đã qua và giờ sát (lead 60 phút)', () => {
    const r = computeDayAvailability({ ...base, staffId: null, nowIso: '2030-03-04T09:10:00+07:00' });
    expect(times(r)).toEqual([630, 660]); // sớm nhất 10:10 làm tròn lên 10:30
  });
});
```

- [ ] **Step 3: Chạy để thấy hỏng**

Run: `npx vitest run src/features/booking`
Expected: FAIL (không tìm thấy `./draft`, `./availability`).

- [ ] **Step 4: Viết reducer các bước**

`src/features/booking/draft.ts`:

```ts
import { BOOKING_STEPS, type BookingStepId } from '@/config/booking';

export type BookingDraft = {
  step: BookingStepId;
  service_id?: string;
  staff_id: string | null; // null = "bất kỳ"
  day?: string;
  minutes?: number;
};

export type BookingAction =
  | { type: 'service'; id: string }
  | { type: 'staff'; id: string | null }
  | { type: 'day'; day: string }
  | { type: 'time'; minutes: number }
  | { type: 'go'; step: BookingStepId }
  | { type: 'next' }
  | { type: 'back' };

const order = BOOKING_STEPS.map((s) => s.id);

export const initialDraft = (serviceId?: string): BookingDraft =>
  serviceId ? { step: 'staff', service_id: serviceId, staff_id: null } : { step: 'service', staff_id: null };

export function canEnter(draft: BookingDraft, step: BookingStepId): boolean {
  switch (step) {
    case 'service':
      return true;
    case 'staff':
    case 'date':
      return !!draft.service_id;
    case 'time':
      return !!draft.service_id && !!draft.day;
    case 'contact':
      return !!draft.service_id && !!draft.day && draft.minutes !== undefined;
  }
}

const move = (draft: BookingDraft, delta: number): BookingDraft => {
  const target = order[order.indexOf(draft.step) + delta];
  return target && canEnter(draft, target) ? { ...draft, step: target } : draft;
};

// Đổi dịch vụ, thợ hoặc ngày thì giờ đã chọn có thể không còn hợp lệ nên bỏ đi.
export function bookingReducer(draft: BookingDraft, action: BookingAction): BookingDraft {
  switch (action.type) {
    case 'service':
      return { ...draft, service_id: action.id, minutes: undefined, step: 'staff' };
    case 'staff':
      return { ...draft, staff_id: action.id, minutes: undefined, step: 'date' };
    case 'day':
      return { ...draft, day: action.day, minutes: undefined, step: 'time' };
    case 'time':
      return { ...draft, minutes: action.minutes, step: 'contact' };
    case 'go':
      return canEnter(draft, action.step) ? { ...draft, step: action.step } : draft;
    case 'next':
      return move(draft, 1);
    case 'back':
      return move(draft, -1);
  }
}
```

Lưu ý: bước `staff` luôn "đủ điều kiện đi tiếp" vì mặc định là "bất kỳ"; nút Tiếp tục ở bước Thợ vì vậy luôn bật.

- [ ] **Step 5: Viết hàm giờ trống theo ngày**

`src/features/booking/availability.ts`:

```ts
import { weekdayOf } from '@/lib/vnTime';
import type { OpeningHours, ShopSettings, Staff, StaffSchedule } from '@/types/db';
import {
  buildSlotGrid,
  busyToIntervals,
  computeSlots,
  earliestStartFor,
  mergeSlots,
  openWindow,
  staffWorkWindow,
  type Interval,
  type SlotView,
} from './slots';

type Input = {
  day: string;
  nowIso: string;
  durationMin: number;
  staffId: string | null;
  settings: Pick<ShopSettings, 'slot_minutes' | 'lead_minutes'>;
  openingHours: OpeningHours[];
  schedules: StaffSchedule[];
  staff: Staff[];
  busy: { staff_id: string; start_at: string; end_at: string }[];
};

export function computeDayAvailability(input: Input): { open: Interval | null; grid: SlotView[] } {
  const { day, nowIso, durationMin, staffId, settings, openingHours, schedules, staff, busy } = input;
  const weekday = weekdayOf(day);
  const open = openWindow(openingHours.find((h) => h.weekday === weekday));
  const stepMin = settings.slot_minutes;
  const earliestStartMin = earliestStartFor(day, nowIso, settings.lead_minutes);

  const candidates = staffId ? staff.filter((s) => s.id === staffId) : staff;
  const perStaff = candidates.map((s) => {
    const window = staffWorkWindow(
      schedules.find((x) => x.staff_id === s.id && x.weekday === weekday),
      open,
    );
    if (!window) return [];
    return computeSlots({
      windows: [window],
      busy: busyToIntervals(busy.filter((b) => b.staff_id === s.id)),
      durationMin,
      stepMin,
      earliestStartMin,
    });
  });

  return { open, grid: buildSlotGrid({ open, durationMin, stepMin, available: mergeSlots(perStaff) }) };
}
```

- [ ] **Step 6: Chạy để thấy qua**

Run: `npx vitest run src/features/booking`
Expected: PASS (slots 13, draft 6, availability 6).

- [ ] **Step 7: Kiểu, API và hook (không có test đơn vị, được phủ bởi E2E ở Task 11)**

`src/features/booking/types.ts`:

```ts
import type { BookingStatus } from '@/config/statuses';

// Dạng một lịch hẹn để hiển thị. Khớp JSON do lookup_booking trả về.
export type BookingView = {
  code: string;
  status: BookingStatus;
  start_at: string;
  end_at: string;
  service_name: string;
  staff_name: string;
  total_vnd: number;
  customer_name?: string;
};

// Khớp JSON do create_booking trả về.
export type BookingReceipt = {
  code: string;
  start_at: string;
  end_at: string;
  staff_id: string;
  total_vnd: number;
  discount_vnd: number;
};

// Khớp JSON do validate_coupon trả về.
export type CouponCheck = {
  valid: boolean;
  percent?: number;
  discount_vnd?: number;
  reason?: 'not_found' | 'inactive' | 'expired' | 'used_up' | 'min_total';
  min_total_vnd?: number;
  ends_at?: string;
};
```

`src/lib/api/bookings.ts`:

```ts
import type { BookingReceipt, BookingView, CouponCheck } from '@/features/booking/types';
import { normalizePhone } from '@/lib/phone';
import { supabase } from '@/lib/supabase';
import { unwrap } from './unwrap';

export type Busy = { staff_id: string; start_at: string; end_at: string };

export const fetchBusy = async (day: string): Promise<Busy[]> =>
  unwrap(await supabase.rpc('get_busy_intervals', { p_day: day }));

export type CreateBookingInput = {
  service_id: string;
  staff_id: string | null;
  start_at: string;
  full_name: string;
  phone: string;
  note?: string;
  coupon?: string;
};

export const createBooking = async (i: CreateBookingInput): Promise<BookingReceipt> =>
  unwrap(
    await supabase.rpc('create_booking', {
      p_service: i.service_id,
      p_staff: i.staff_id,
      p_start: i.start_at,
      p_name: i.full_name,
      p_phone: normalizePhone(i.phone),
      p_note: i.note || null,
      p_coupon: i.coupon || null,
    }),
  );

export const lookupBooking = async (phone: string, code: string): Promise<BookingView | null> =>
  unwrap(await supabase.rpc('lookup_booking', { p_phone: normalizePhone(phone), p_code: code }));

export const cancelBooking = async (phone: string, code: string): Promise<void> => {
  unwrap(await supabase.rpc('cancel_booking', { p_phone: normalizePhone(phone), p_code: code }));
};

export const validateCoupon = async (code: string, total: number): Promise<CouponCheck> =>
  unwrap(await supabase.rpc('validate_coupon', { p_code: code, p_total: total }));
```

`src/features/booking/queries.ts`:

```ts
import { useMutation, useQuery, useQueryClient } from '@tanstack/react-query';
import {
  cancelBooking,
  createBooking,
  fetchBusy,
  lookupBooking,
  validateCoupon,
  type CreateBookingInput,
} from '@/lib/api/bookings';
import { queryKeys } from '@/lib/queryKeys';
import { useOpeningHours, useShopSettings, useStaff, useStaffSchedules } from '@/features/catalog/queries';
import { useMemo } from 'react';
import { computeDayAvailability } from './availability';

// Giờ bận thay đổi nhanh nên cache ngắn và tự làm mới.
export const useBusy = (day?: string) =>
  useQuery({
    queryKey: queryKeys.busy(day ?? ''),
    queryFn: () => fetchBusy(day!),
    enabled: !!day,
    staleTime: 10_000,
    refetchInterval: 30_000,
  });

export function useAvailability(args: { day?: string; durationMin?: number; staffId: string | null }) {
  const busy = useBusy(args.day);
  const settings = useShopSettings();
  const hours = useOpeningHours();
  const schedules = useStaffSchedules();
  const staff = useStaff();
  const ready = busy.data && settings.data && hours.data && schedules.data && staff.data;

  const result = useMemo(() => {
    if (!ready || !args.day || !args.durationMin) return undefined;
    return computeDayAvailability({
      day: args.day,
      nowIso: new Date().toISOString(),
      durationMin: args.durationMin,
      staffId: args.staffId,
      settings: settings.data!,
      openingHours: hours.data!,
      schedules: schedules.data!,
      staff: staff.data!,
      busy: busy.data!,
    });
  }, [ready, args.day, args.durationMin, args.staffId, busy.data, settings.data, hours.data, schedules.data, staff.data]);

  return { isPending: !result, isError: busy.isError, refetch: busy.refetch, ...result };
}

export function useCreateBooking() {
  const client = useQueryClient();
  return useMutation({
    mutationFn: (input: CreateBookingInput) => createBooking(input),
    onSettled: (_data, _error, input) =>
      client.invalidateQueries({ queryKey: queryKeys.busy(input.start_at.slice(0, 10)) }),
  });
}

export const useLookupBooking = () =>
  useMutation({ mutationFn: (v: { phone: string; code: string }) => lookupBooking(v.phone, v.code) });

export const useCancelBooking = () =>
  useMutation({ mutationFn: (v: { phone: string; code: string }) => cancelBooking(v.phone, v.code) });

export const useValidateCoupon = () =>
  useMutation({ mutationFn: (v: { code: string; total: number }) => validateCoupon(v.code, v.total) });
```

Lưu ý quan trọng: `onSettled` dùng `input.start_at.slice(0, 10)` là ngày ISO ở múi giờ `+07:00` khi `start_at` được dựng bằng `vnToIso` (luôn bắt đầu bằng `YYYY-MM-DD` của ngày Việt Nam), nên khóa `busy(day)` khớp với ngày đang xem.

- [ ] **Step 8: Kiểm tra**

Run: `npm run check`
Expected: typecheck, lint, test qua.

- [ ] **Step 9: Commit**

```bash
git add -A
git commit -m "feat(booking): reducer các bước, giờ trống theo ngày và lớp gọi API đặt lịch"
```

---

## Task 7: Giao diện đặt lịch

**Files:**
- Create: `src/components/booking/StepSection.tsx`, `StaffStep.tsx`, `DateStrip.tsx`, `SlotGrid.tsx`, `TimeStep.tsx`, `ContactStep.tsx`, `SummaryCard.tsx`, `BookingBar.tsx`
- Create: `src/pages/customer/BookingPage.tsx`
- Modify: `src/app/router.tsx`
- Test: `src/components/booking/SlotGrid.test.tsx`, `src/components/booking/DateStrip.test.tsx`, `src/components/booking/SummaryCard.test.tsx`

**Interfaces:**
- Consumes: `useBookingDraft` bằng `useReducer(bookingReducer, initialDraft(...))`, `useAvailability`, `useCreateBooking`, `useValidateCoupon`, `ServiceRow`, `contactSchema`, `pointsFor`, `BOOKING_STEPS`, `BOOKING_UI`, `toUserMessage`.
- Produces:
  - `SlotGrid({ grid, value, onSelect })` (`grid: SlotView[]`, `value?: number`, `onSelect: (minutes: number) => void`)
  - `DateStrip({ value, onSelect, openingHours })`
  - `SummaryCard({ serviceName?, staffName, day?, minutes?, durationMin?, price?, discount, points, children })`
  - `StepSection({ index, id, label, current, done, summary?, onEdit, children })`
  - `BookingPage`

- [ ] **Step 1: Viết test hỏng**

`src/components/booking/SlotGrid.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { SlotGrid } from './SlotGrid';

const grid = [
  { minutes: 540, available: true },
  { minutes: 570, available: false },
  { minutes: 600, available: true },
];

describe('SlotGrid', () => {
  it('hiện giờ dạng 9:00 và vô hiệu hóa giờ không còn trống', () => {
    render(<SlotGrid grid={grid} onSelect={() => {}} />);
    expect(screen.getByRole('button', { name: /9:00/ })).toBeEnabled();
    expect(screen.getByRole('button', { name: /9:30/ })).toBeDisabled();
  });

  it('bấm giờ trống thì gọi onSelect với số phút', async () => {
    const onSelect = vi.fn();
    render(<SlotGrid grid={grid} onSelect={onSelect} />);
    await userEvent.click(screen.getByRole('button', { name: /10:00/ }));
    expect(onSelect).toHaveBeenCalledWith(600);
  });

  it('đánh dấu giờ đang chọn bằng aria-pressed', () => {
    render(<SlotGrid grid={grid} value={600} onSelect={() => {}} />);
    expect(screen.getByRole('button', { name: /10:00/ })).toHaveAttribute('aria-pressed', 'true');
  });

  it('ngày tiệm nghỉ (lưới rỗng) thì báo rõ, không để trống trơn', () => {
    render(<SlotGrid grid={[]} onSelect={() => {}} />);
    expect(screen.getByText('Tiệm nghỉ ngày này')).toBeInTheDocument();
  });
});
```

`src/components/booking/DateStrip.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { BOOKING_UI } from '@/config/booking';
import { addDays, todayVn, weekdayOf } from '@/lib/vnTime';
import type { OpeningHours } from '@/types/db';
import { DateStrip } from './DateStrip';

const hoursClosedOn = (closedWeekday: number): OpeningHours[] =>
  [0, 1, 2, 3, 4, 5, 6].map((weekday) => ({
    weekday, open_time: '09:00:00', close_time: '20:00:00', is_closed: weekday === closedWeekday,
  }));

describe('DateStrip', () => {
  it('hiện đủ số ngày đặt trước trong config', () => {
    render(<DateStrip openingHours={hoursClosedOn(-1)} onSelect={() => {}} />);
    expect(screen.getAllByRole('button')).toHaveLength(BOOKING_UI.daysAhead);
  });

  it('ngày tiệm nghỉ bị vô hiệu hóa', () => {
    const today = todayVn();
    const closed = weekdayOf(addDays(today, 1));
    render(<DateStrip openingHours={hoursClosedOn(closed)} onSelect={() => {}} />);
    const disabled = screen.getAllByRole('button').filter((b) => (b as HTMLButtonElement).disabled);
    expect(disabled.length).toBeGreaterThan(0);
  });

  it('bấm ngày còn mở thì trả về chuỗi ngày YYYY-MM-DD', async () => {
    const onSelect = vi.fn();
    render(<DateStrip openingHours={hoursClosedOn(-1)} onSelect={onSelect} />);
    await userEvent.click(screen.getAllByRole('button')[2]);
    expect(onSelect).toHaveBeenCalledWith(addDays(todayVn(), 2));
  });
});
```

`src/components/booking/SummaryCard.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react';
import { SummaryCard } from './SummaryCard';

describe('SummaryCard', () => {
  it('chưa chọn gì thì nhắc chọn dịch vụ', () => {
    render(<SummaryCard staffName="Bất kỳ" discount={0} points={0} />);
    expect(screen.getByText('Chưa chọn dịch vụ')).toBeInTheDocument();
  });

  it('tổng = giá trừ giảm giá, kèm điểm sẽ nhận', () => {
    render(
      <SummaryCard
        serviceName="Gel tay" staffName="Mai" day="2030-03-04" minutes={600}
        durationMin={60} price={350000} discount={35000} points={31}
      />,
    );
    expect(screen.getByText('Gel tay')).toBeInTheDocument();
    expect(screen.getByText('315.000đ')).toBeInTheDocument();
    expect(screen.getByText(/31 điểm/)).toBeInTheDocument();
    expect(screen.getByText(/10:00 đến 11:00/)).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/components/booking`
Expected: FAIL (không tìm thấy module).

- [ ] **Step 3: Viết các component đặt lịch**

`src/components/booking/SlotGrid.tsx`:

```tsx
import { cn } from '@/lib/cn';
import { minutesToLabel } from '@/lib/vnTime';
import type { SlotView } from '@/features/booking/slots';

type Props = { grid: SlotView[]; value?: number; onSelect: (minutes: number) => void };

export function SlotGrid({ grid, value, onSelect }: Props) {
  if (grid.length === 0) return <p className="rounded-card bg-blush p-4">Tiệm nghỉ ngày này</p>;
  return (
    <div>
      <ul aria-label="Chọn giờ" className="grid grid-cols-3 gap-2 sm:grid-cols-4 lg:grid-cols-6">
        {grid.map((slot) => (
          <li key={slot.minutes}>
            <button
              type="button"
              disabled={!slot.available}
              aria-pressed={value === slot.minutes}
              onClick={() => onSelect(slot.minutes)}
              className={cn(
                'min-h-11 w-full rounded-control border text-center font-medium tabular-nums focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-rose',
                value === slot.minutes
                  ? 'border-rose bg-rose text-white'
                  : 'border-line bg-surface text-berry hover:bg-blush',
                !slot.available && 'cursor-not-allowed opacity-40 line-through hover:bg-surface',
              )}
            >
              {minutesToLabel(slot.minutes)}
              {!slot.available && <span className="sr-only"> (đã có lịch hoặc đã qua)</span>}
            </button>
          </li>
        ))}
      </ul>
      <p className="mt-2 text-sm text-ink/70">Giờ mờ gạch ngang là đã có lịch hoặc đã qua.</p>
    </div>
  );
}
```

`src/components/booking/DateStrip.tsx`:

```tsx
import { BOOKING_UI } from '@/config/booking';
import { WEEKDAY_LABELS } from '@/config/site';
import { cn } from '@/lib/cn';
import { addDays, todayVn, weekdayOf } from '@/lib/vnTime';
import type { OpeningHours } from '@/types/db';

type Props = { value?: string; onSelect: (day: string) => void; openingHours: OpeningHours[] };

export function DateStrip({ value, onSelect, openingHours }: Props) {
  const today = todayVn();
  const days = Array.from({ length: BOOKING_UI.daysAhead }, (_, i) => addDays(today, i));
  return (
    <ul aria-label="Chọn ngày" className="flex gap-2 overflow-x-auto pb-2">
      {days.map((day) => {
        const weekday = weekdayOf(day);
        const closed = openingHours.find((h) => h.weekday === weekday)?.is_closed ?? false;
        return (
          <li key={day}>
            <button
              type="button"
              disabled={closed}
              aria-pressed={value === day}
              onClick={() => onSelect(day)}
              className={cn(
                'flex min-h-16 w-16 flex-col items-center justify-center rounded-control border focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-rose',
                value === day ? 'border-rose bg-rose text-white' : 'border-line bg-surface text-berry hover:bg-blush',
                closed && 'cursor-not-allowed opacity-40',
              )}
            >
              <span className="text-xs">{WEEKDAY_LABELS[weekday]}</span>
              <span className="text-lg font-bold tabular-nums">{Number(day.slice(8))}</span>
              {closed && <span className="sr-only">Tiệm nghỉ</span>}
            </button>
          </li>
        );
      })}
    </ul>
  );
}
```

`src/components/booking/StaffStep.tsx`:

```tsx
import { BOOKING_UI } from '@/config/booking';
import { cn } from '@/lib/cn';
import type { Staff } from '@/types/db';

type Props = { staff: Staff[]; value: string | null; onSelect: (id: string | null) => void };

export function StaffStep({ staff, value, onSelect }: Props) {
  const options: { id: string | null; name: string; note?: string }[] = [
    { id: null, name: BOOKING_UI.anyStaffLabel, note: 'Gợi ý: có giờ sớm nhất' },
    ...staff.map((s) => ({ id: s.id, name: s.name, note: s.bio ?? undefined })),
  ];
  return (
    <ul className="grid gap-3 sm:grid-cols-2">
      {options.map((o) => (
        <li key={o.id ?? 'any'}>
          <button
            type="button"
            aria-pressed={value === o.id}
            onClick={() => onSelect(o.id)}
            className={cn(
              'flex min-h-16 w-full items-center gap-3 rounded-card border p-3 text-left focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-rose',
              value === o.id ? 'border-rose bg-blush' : 'border-line bg-surface hover:bg-blush',
            )}
          >
            <span className="grid size-10 shrink-0 place-items-center rounded-full bg-blush font-heading font-extrabold text-rose">
              {o.name.charAt(0)}
            </span>
            <span>
              <span className="block font-bold text-berry">{o.name}</span>
              {o.note && <span className="block text-sm text-ink/80">{o.note}</span>}
            </span>
          </button>
        </li>
      ))}
    </ul>
  );
}
```

`src/components/booking/StepSection.tsx`:

```tsx
import type { ReactNode } from 'react';
import { Check } from 'lucide-react';
import { COPY } from '@/config/copy';
import { cn } from '@/lib/cn';
import { Button } from '@/components/ui/Button';

type Props = {
  index: number;
  label: string;
  current: boolean;
  done: boolean;
  summary?: string;
  onEdit: () => void;
  children: ReactNode;
};

// Điện thoại: chỉ bước hiện tại hiện ra. Từ 768px: mọi bước hiện, bước đã xong thu gọn thành một dòng tóm tắt.
export function StepSection({ index, label, current, done, summary, onEdit, children }: Props) {
  return (
    <section
      aria-labelledby={`step-${index}`}
      className={cn('rounded-card border border-line bg-surface p-4 shadow-card', !current && 'hidden md:block')}
    >
      <div className="flex items-center justify-between gap-3">
        <h2 id={`step-${index}`} className="flex items-center gap-2 text-lg font-bold">
          <span
            className={cn(
              'grid size-7 place-items-center rounded-full text-sm',
              done ? 'bg-success text-white' : 'bg-blush text-berry',
            )}
          >
            {done ? <Check aria-hidden="true" className="size-4" /> : index + 1}
          </span>
          {label}
        </h2>
        {done && !current && (
          <Button variant="ghost" size="sm" onClick={onEdit}>
            {COPY.actions.edit}
            <span className="sr-only"> {label}</span>
          </Button>
        )}
      </div>
      {current ? <div className="mt-4">{children}</div> : summary && <p className="mt-1 text-ink/80">{summary}</p>}
    </section>
  );
}
```

`src/components/booking/SummaryCard.tsx`:

```tsx
import type { ReactNode } from 'react';
import { PriceText } from '@/components/PriceText';
import { formatDayVi } from '@/lib/format';
import { minutesToLabel } from '@/lib/vnTime';

type Props = {
  serviceName?: string;
  staffName: string;
  day?: string;
  minutes?: number;
  durationMin?: number;
  price?: number;
  discount: number;
  points: number;
  children?: ReactNode;
};

export function SummaryCard(p: Props) {
  const total = (p.price ?? 0) - p.discount;
  return (
    <aside aria-label="Tóm tắt đặt lịch" className="rounded-card border border-line bg-surface p-4 shadow-card">
      <h2 className="text-lg font-bold">Tóm tắt</h2>
      {!p.serviceName ? (
        <p className="mt-2 text-ink/80">Chưa chọn dịch vụ</p>
      ) : (
        <dl className="mt-3 space-y-2">
          <div className="flex justify-between gap-3">
            <dt className="text-ink/80">Dịch vụ</dt>
            <dd className="font-medium">{p.serviceName}</dd>
          </div>
          <div className="flex justify-between gap-3">
            <dt className="text-ink/80">Thợ</dt>
            <dd className="font-medium">{p.staffName}</dd>
          </div>
          {p.day && p.minutes !== undefined && (
            <div className="flex justify-between gap-3">
              <dt className="text-ink/80">Thời gian</dt>
              <dd className="text-right font-medium">
                {formatDayVi(p.day)}
                <br />
                {minutesToLabel(p.minutes)} đến {minutesToLabel(p.minutes + (p.durationMin ?? 0))}
              </dd>
            </div>
          )}
          {p.discount > 0 && (
            <div className="flex justify-between gap-3">
              <dt className="text-ink/80">Giảm giá</dt>
              <dd className="font-medium text-success">-<PriceText value={p.discount} /></dd>
            </div>
          )}
          <div className="flex items-center justify-between gap-3 border-t border-line pt-2 text-lg">
            <dt className="font-bold">Tổng</dt>
            <dd><PriceText value={total} /></dd>
          </div>
          <p className="text-sm text-ink/80">Bạn sẽ nhận {p.points} điểm sau khi hoàn thành.</p>
        </dl>
      )}
      {p.children && <div className="mt-4">{p.children}</div>}
    </aside>
  );
}
```

Lưu ý: ở `SummaryCard`, ô giá giảm hiển thị `-` rồi `PriceText`; test `315.000đ` khớp phần Tổng.

`src/components/booking/BookingBar.tsx`:

```tsx
import { COPY } from '@/config/copy';
import { formatDuration, formatVnd } from '@/lib/format';
import { Button } from '@/components/ui/Button';

type Props = {
  total?: number;
  durationMin?: number;
  canBack: boolean;
  canNext: boolean;
  isLast: boolean;
  submitting: boolean;
  onBack: () => void;
  onNext: () => void;
};

// Thanh dính đáy chỉ có trên điện thoại. Từ 768px thì Tóm tắt nằm ở cột phải.
export function BookingBar(p: Props) {
  return (
    <div className="fixed inset-x-0 bottom-0 z-20 flex items-center gap-3 border-t border-line bg-surface p-3 md:hidden">
      <div className="min-w-0 flex-1 text-sm">
        {p.total !== undefined && <p className="font-bold text-rose">{formatVnd(p.total)}</p>}
        {p.durationMin !== undefined && <p className="text-ink/80">{formatDuration(p.durationMin)}</p>}
      </div>
      <Button variant="secondary" disabled={!p.canBack} onClick={p.onBack}>
        {COPY.actions.back}
      </Button>
      {p.isLast ? (
        <Button type="submit" form="contact-form" disabled={p.submitting}>
          {COPY.actions.confirmBooking}
        </Button>
      ) : (
        <Button disabled={!p.canNext} onClick={p.onNext}>
          {COPY.actions.next}
        </Button>
      )}
    </div>
  );
}
```

`src/components/booking/ContactStep.tsx`:

```tsx
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { Button } from '@/components/ui/Button';
import { Field, Input, Textarea } from '@/components/ui/Field';
import { BOOKING_UI } from '@/config/booking';
import { contactSchema, type ContactForm } from '@/features/booking/schemas';
import type { CouponCheck } from '@/features/booking/types';

type Props = {
  onSubmit: (values: ContactForm) => void;
  onApplyCoupon: (code: string) => void;
  coupon?: CouponCheck;
  couponMessage?: string;
  checkingCoupon: boolean;
};

// Nút gửi nằm ở SummaryCard (máy tính) hoặc BookingBar (điện thoại) và gắn vào biểu mẫu này qua form="contact-form".
export function ContactStep({ onSubmit, onApplyCoupon, coupon, couponMessage, checkingCoupon }: Props) {
  const { register, handleSubmit, getValues, formState: { errors } } = useForm<ContactForm>({
    resolver: zodResolver(contactSchema),
  });
  return (
    <form id="contact-form" onSubmit={handleSubmit(onSubmit)} noValidate className="grid gap-4">
      <Field label="Họ tên" required error={errors.full_name?.message}>
        {(a) => <Input {...a} {...register('full_name')} autoComplete="name" />}
      </Field>
      <Field label="Số điện thoại" required error={errors.phone?.message} hint="Dùng để tra cứu và nhắc lịch">
        {(a) => <Input {...a} {...register('phone')} type="tel" inputMode="tel" autoComplete="tel" />}
      </Field>
      <Field label="Ghi chú (không bắt buộc)" error={errors.note?.message}>
        {(a) => <Textarea {...a} {...register('note')} maxLength={BOOKING_UI.noteMaxLength + 50} />}
      </Field>
      <details>
        <summary className="min-h-11 cursor-pointer content-center font-medium text-rose">Có mã giảm giá?</summary>
        <div className="mt-2 flex items-end gap-2">
          <div className="flex-1">
            <Field label="Mã giảm giá" error={coupon && !coupon.valid ? couponMessage : undefined}>
              {(a) => <Input {...a} {...register('coupon')} autoCapitalize="characters" />}
            </Field>
          </div>
          <Button variant="secondary" disabled={checkingCoupon} onClick={() => onApplyCoupon(getValues('coupon') ?? '')}>
            Áp dụng
          </Button>
        </div>
        {coupon?.valid && <p className="mt-1 text-success">Đã áp dụng mã, giảm {coupon.percent}%.</p>}
      </details>
    </form>
  );
}
```

- [ ] **Step 4: Chạy để thấy qua (phần component)**

Run: `npx vitest run src/components/booking`
Expected: PASS (SlotGrid 4, DateStrip 3, SummaryCard 2).

- [ ] **Step 5: Trang đặt lịch**

`src/pages/customer/BookingPage.tsx`:

```tsx
import { useQueryClient } from '@tanstack/react-query';
import { useEffect, useReducer, useState } from 'react';
import { useNavigate, useSearchParams } from 'react-router-dom';
import { toast } from 'sonner';
import { BookingBar } from '@/components/booking/BookingBar';
import { ContactStep } from '@/components/booking/ContactStep';
import { DateStrip } from '@/components/booking/DateStrip';
import { StaffStep } from '@/components/booking/StaffStep';
import { StepSection } from '@/components/booking/StepSection';
import { SummaryCard } from '@/components/booking/SummaryCard';
import { TimeStep } from '@/components/booking/TimeStep';
import { QueryState } from '@/components/QueryState';
import { ServiceRow } from '@/components/ServiceRow';
import { Button } from '@/components/ui/Button';
import { BOOKING_STEPS, BOOKING_UI } from '@/config/booking';
import { COPY } from '@/config/copy';
import { ROUTES } from '@/config/routes';
import { useOpeningHours, useServices, useShopSettings, useStaff } from '@/features/catalog/queries';
import { bookingReducer, canEnter, initialDraft } from '@/features/booking/draft';
import { useAvailability, useCreateBooking, useValidateCoupon } from '@/features/booking/queries';
import type { ContactForm } from '@/features/booking/schemas';
import type { CouponCheck } from '@/features/booking/types';
import { pointsFor } from '@/features/points/pointsMath';
import { formatDayVi, formatVnd } from '@/lib/format';
import { queryKeys } from '@/lib/queryKeys';
import { toUserMessage } from '@/lib/errors';
import { minutesToLabel, vnToIso } from '@/lib/vnTime';

export function BookingPage() {
  const navigate = useNavigate();
  const client = useQueryClient();
  const [params] = useSearchParams();
  const [draft, dispatch] = useReducer(bookingReducer, params.get('dich-vu') ?? undefined, initialDraft);
  const [coupon, setCoupon] = useState<CouponCheck>();

  const services = useServices();
  const staff = useStaff();
  const hours = useOpeningHours();
  const settings = useShopSettings();
  const service = services.data?.find((s) => s.id === draft.service_id);
  const staffName = staff.data?.find((s) => s.id === draft.staff_id)?.name ?? BOOKING_UI.anyStaffLabel;
  const availability = useAvailability({ day: draft.day, durationMin: service?.duration_min, staffId: draft.staff_id });
  const create = useCreateBooking();
  const check = useValidateCoupon();

  // Giá đổi (đổi dịch vụ) thì mã giảm giá cũ không còn đúng.
  useEffect(() => setCoupon(undefined), [draft.service_id]);

  const discount = coupon?.valid ? (coupon.discount_vnd ?? 0) : 0;
  const total = (service?.price_vnd ?? 0) - discount;
  const points = pointsFor(total, settings.data?.points_per_10k ?? 1);

  const applyCoupon = (code: string) => {
    if (!service || !code.trim()) return;
    check.mutate({ code, total: service.price_vnd }, { onSuccess: setCoupon, onError: (e) => toast.error(toUserMessage(e)) });
  };

  const submit = (values: ContactForm) => {
    if (!service || !draft.day || draft.minutes === undefined) return;
    create.mutate(
      {
        service_id: service.id,
        staff_id: draft.staff_id,
        start_at: vnToIso(draft.day, draft.minutes),
        full_name: values.full_name,
        phone: values.phone,
        note: values.note,
        coupon: coupon?.valid ? values.coupon : undefined,
      },
      {
        onSuccess: (receipt) =>
          navigate(ROUTES.bookingSuccess, {
            state: { receipt, service_name: service.name, phone: values.phone, full_name: values.full_name },
          }),
        onError: (error) => {
          toast.error(toUserMessage(error));
          // Giờ vừa bị người khác đặt: giữ thông tin đã nhập, về bước Giờ và tải lại giờ trống.
          if (error instanceof Error && ['slot_taken', 'slot_past', 'slot_invalid'].includes(error.message)) {
            dispatch({ type: 'day', day: draft.day! });
            void client.invalidateQueries({ queryKey: queryKeys.busy(draft.day!) });
          }
        },
      },
    );
  };

  const stepIndex = BOOKING_STEPS.findIndex((s) => s.id === draft.step);
  const isLast = draft.step === 'contact';
  const canNext = canEnter(draft, BOOKING_STEPS[stepIndex + 1]?.id ?? 'contact') && !isLast;
  // Một bước coi là xong khi dữ liệu của nó đã có.
  const doneMap: Record<(typeof BOOKING_STEPS)[number]['id'], boolean> = {
    service: !!draft.service_id,
    staff: !!draft.service_id,
    date: !!draft.day,
    time: draft.minutes !== undefined,
    contact: false,
  };
  const summaries: Record<string, string | undefined> = {
    service: service?.name,
    staff: staffName,
    date: draft.day && formatDayVi(draft.day),
    time: draft.minutes !== undefined ? minutesToLabel(draft.minutes) : undefined,
  };

  return (
    <div className="mx-auto max-w-[1120px] px-4 pb-28 pt-8 md:px-8 md:pb-10">
      <h1 className="text-3xl font-extrabold">Đặt lịch</h1>
      <p className="mt-1 text-ink/80" aria-live="polite">
        Bước {stepIndex + 1}/{BOOKING_STEPS.length}: {BOOKING_STEPS[stepIndex].label}
      </p>
      <div className="mt-6 grid gap-6 md:grid-cols-[1fr_20rem]">
        <div className="grid content-start gap-4">
          {BOOKING_STEPS.map((step, i) => (
            <StepSection
              key={step.id}
              index={i}
              label={step.label}
              current={draft.step === step.id}
              done={doneMap[step.id]}
              summary={summaries[step.id]}
              onEdit={() => dispatch({ type: 'go', step: step.id })}
            >
              {step.id === 'service' && (
                <ul className="grid gap-3">
                  <QueryState query={services}>
                    {(all) =>
                      all.map((s) => (
                        <ServiceRow key={s.id} service={s} selected={s.id === draft.service_id} onSelect={(x) => dispatch({ type: 'service', id: x.id })} />
                      ))
                    }
                  </QueryState>
                </ul>
              )}
              {step.id === 'staff' && (
                <QueryState query={staff}>
                  {(all) => <StaffStep staff={all} value={draft.staff_id} onSelect={(id) => dispatch({ type: 'staff', id })} />}
                </QueryState>
              )}
              {step.id === 'date' && (
                <QueryState query={hours}>
                  {(h) => <DateStrip value={draft.day} openingHours={h} onSelect={(day) => dispatch({ type: 'day', day })} />}
                </QueryState>
              )}
              {step.id === 'time' && (
                <TimeStep availability={availability} value={draft.minutes} onSelect={(minutes) => dispatch({ type: 'time', minutes })} />
              )}
              {step.id === 'contact' && (
                <ContactStep
                  onSubmit={submit}
                  onApplyCoupon={applyCoupon}
                  coupon={coupon}
                  couponMessage={coupon && !coupon.valid ? COPY.coupon[coupon.reason ?? 'not_found'] : undefined}
                  checkingCoupon={check.isPending}
                />
              )}
            </StepSection>
          ))}
        </div>
        <div className="hidden md:block md:sticky md:top-24 md:self-start">
          <SummaryCard
            serviceName={service?.name}
            staffName={staffName}
            day={draft.day}
            minutes={draft.minutes}
            durationMin={service?.duration_min}
            price={service?.price_vnd}
            discount={discount}
            points={points}
          >
            <Button type="submit" form="contact-form" className="w-full" disabled={!isLast || create.isPending}>
              {COPY.actions.confirmBooking}
            </Button>
          </SummaryCard>
        </div>
      </div>
      <BookingBar
        total={service ? total : undefined}
        durationMin={service?.duration_min}
        canBack={stepIndex > 0}
        canNext={canNext}
        isLast={isLast}
        submitting={create.isPending}
        onBack={() => dispatch({ type: 'back' })}
        onNext={() => dispatch({ type: 'next' })}
      />
    </div>
  );
}
```

`useAvailability` trả đối tượng tự dựng (không phải `UseQueryResult`) nên bước Giờ dùng component riêng `TimeStep`, xử lý đủ ba trạng thái tải, lỗi, có dữ liệu:

`src/components/booking/TimeStep.tsx`:

```tsx
import { COPY } from '@/config/copy';
import { Button } from '@/components/ui/Button';
import { EmptyState } from '@/components/ui/EmptyState';
import { Skeleton } from '@/components/ui/Skeleton';
import type { SlotView } from '@/features/booking/slots';
import { SlotGrid } from './SlotGrid';

type Props = {
  availability: { isPending: boolean; isError: boolean; refetch: () => unknown; grid?: SlotView[] };
  value?: number;
  onSelect: (minutes: number) => void;
};

export function TimeStep({ availability, value, onSelect }: Props) {
  if (availability.isError) {
    return (
      <EmptyState
        title={COPY.errors.network}
        action={<Button variant="secondary" onClick={() => availability.refetch()}>{COPY.actions.retry}</Button>}
      />
    );
  }
  if (availability.isPending) return <Skeleton className="h-40" />;
  return <SlotGrid grid={availability.grid ?? []} value={value} onSelect={onSelect} />;
}
```

Trong `router.tsx` thêm:

```tsx
const BookingPage = lazyPage(() => import('@/pages/customer/BookingPage'), 'BookingPage');
// ...và thêm vào customerRoutes:
page(ROUTES.booking, BookingPage),
```

- [ ] **Step 6: Kiểm tra bằng trình duyệt**

Run: `npm run dev`, mở `/dat-lich`.
Expected: Máy tính (rộng 1280): các bước hiện dạng khối gập mở, tóm tắt dính ở cột phải, chọn xong bước thì tự sang bước kế. Điện thoại (375): chỉ một bước, thanh Quay lại/Tiếp tục dính đáy kèm tổng tiền. Chọn dịch vụ từ `/dich-vu` (nút Chọn) thì vào thẳng bước Thợ. Giờ đã qua hôm nay bị mờ gạch ngang. Ngày tiệm nghỉ bị vô hiệu.

Run: `npm run check`
Expected: qua.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "feat(booking): trang đặt lịch 5 bước, hai cột trên máy tính, từng bước trên điện thoại"
```

---

## Task 8: Xác nhận đặt lịch, tra cứu và hủy lịch

**Files:**
- Create: `src/features/booking/ics.ts`, `src/features/booking/cancel.ts`, `src/features/booking/reminder.ts`
- Create: `src/components/BookingCard.tsx`, `src/components/CancelBookingDialog.tsx`, `src/components/ReminderBanner.tsx`
- Create: `src/pages/customer/BookingSuccessPage.tsx`, `src/pages/customer/LookupPage.tsx`
- Modify: `src/app/router.tsx`
- Test: `src/features/booking/ics.test.ts`, `cancel.test.ts`, `reminder.test.ts`

**Interfaces:**
- Consumes: `BookingView`, `BookingReceipt`, `useLookupBooking`, `useCancelBooking`, `ConfirmDialog`, `StatusBadge`, `useShopSettings`.
- Produces:
  - `buildIcs(event): string`, `downloadIcs(filename: string, content: string): void`
  - `canCancelOnline(args: { status: BookingStatus; start_at: string; nowIso: string; cancelHours: number }): boolean`
  - `needsReminder(args: { status: BookingStatus; start_at: string; nowIso: string }): boolean`
  - `BookingCard({ booking, actions? })`, `CancelBookingDialog({ open, onOpenChange, phone, code, onCancelled })`, `ReminderBanner({ booking })`
  - `BookingSuccessPage`, `LookupPage`

- [ ] **Step 1: Viết test hỏng**

`src/features/booking/ics.test.ts`:

```ts
import { buildIcs } from './ics';

describe('buildIcs', () => {
  const ics = buildIcs({
    uid: 'K7H2QM@lua-nails',
    title: 'Gel tay tại Lụa Nails',
    startIso: '2030-03-04T10:00:00+07:00',
    endIso: '2030-03-04T11:00:00+07:00',
    location: '123 Đường Hoa Lan, TP.HCM',
    description: 'Mã lịch: K7H2QM',
  });

  it('có khung VCALENDAR/VEVENT', () => {
    expect(ics.startsWith('BEGIN:VCALENDAR')).toBe(true);
    expect(ics).toContain('BEGIN:VEVENT');
    expect(ics.trim().endsWith('END:VCALENDAR')).toBe(true);
  });

  it('đổi giờ Việt Nam sang UTC', () => {
    expect(ics).toContain('DTSTART:20300304T030000Z');
    expect(ics).toContain('DTEND:20300304T040000Z');
  });

  it('thoát dấu phẩy và chấm phẩy theo chuẩn iCalendar', () => {
    expect(ics).toContain('LOCATION:123 Đường Hoa Lan\\, TP.HCM');
  });

  it('dùng CRLF làm dấu xuống dòng', () => {
    expect(ics).toContain('\r\n');
  });
});
```

`src/features/booking/cancel.test.ts`:

```ts
import { canCancelOnline } from './cancel';

const base = { start_at: '2030-03-04T10:00:00+07:00', cancelHours: 2 };

describe('canCancelOnline', () => {
  it('còn hơn 2 giờ thì hủy được', () => {
    expect(canCancelOnline({ ...base, status: 'confirmed', nowIso: '2030-03-04T07:00:00+07:00' })).toBe(true);
  });
  it('đúng 2 giờ trước giờ hẹn vẫn hủy được; ít hơn thì không', () => {
    expect(canCancelOnline({ ...base, status: 'pending', nowIso: '2030-03-04T08:00:00+07:00' })).toBe(true);
    expect(canCancelOnline({ ...base, status: 'pending', nowIso: '2030-03-04T08:01:00+07:00' })).toBe(false);
  });
  it('lịch đã hoàn thành hoặc đã hủy thì không hủy được', () => {
    expect(canCancelOnline({ ...base, status: 'completed', nowIso: '2030-03-01T07:00:00+07:00' })).toBe(false);
    expect(canCancelOnline({ ...base, status: 'cancelled', nowIso: '2030-03-01T07:00:00+07:00' })).toBe(false);
  });
});
```

`src/features/booking/reminder.test.ts`:

```ts
import { needsReminder } from './reminder';

const start = '2030-03-04T10:00:00+07:00';

describe('needsReminder', () => {
  it('lịch đã xác nhận bắt đầu trong 24 giờ tới thì cần nhắc', () => {
    expect(needsReminder({ status: 'confirmed', start_at: start, nowIso: '2030-03-03T12:00:00+07:00' })).toBe(true);
  });
  it('còn hơn 24 giờ thì chưa nhắc', () => {
    expect(needsReminder({ status: 'confirmed', start_at: start, nowIso: '2030-03-03T09:00:00+07:00' })).toBe(false);
  });
  it('lịch đã qua thì không nhắc', () => {
    expect(needsReminder({ status: 'confirmed', start_at: start, nowIso: '2030-03-04T10:30:00+07:00' })).toBe(false);
  });
  it('chưa xác nhận, đã hủy, hoàn thành thì không nhắc', () => {
    for (const status of ['pending', 'cancelled', 'completed'] as const) {
      expect(needsReminder({ status, start_at: start, nowIso: '2030-03-03T12:00:00+07:00' })).toBe(false);
    }
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/features/booking/ics.test.ts src/features/booking/cancel.test.ts src/features/booking/reminder.test.ts`
Expected: FAIL.

- [ ] **Step 3: Viết hàm thuần**

`src/features/booking/ics.ts`:

```ts
type IcsEvent = {
  uid: string;
  title: string;
  startIso: string;
  endIso: string;
  location: string;
  description?: string;
};

const toUtc = (iso: string) => new Date(iso).toISOString().replace(/[-:]/g, '').replace(/\.\d{3}/, '');
const escapeText = (text: string) => text.replace(/\\/g, '\\\\').replace(/[,;]/g, (m) => `\\${m}`).replace(/\n/g, '\\n');

export function buildIcs(e: IcsEvent): string {
  return [
    'BEGIN:VCALENDAR',
    'VERSION:2.0',
    'PRODID:-//Lua Nails//Dat lich//VI',
    'BEGIN:VEVENT',
    `UID:${e.uid}`,
    `DTSTAMP:${toUtc(new Date().toISOString())}`,
    `DTSTART:${toUtc(e.startIso)}`,
    `DTEND:${toUtc(e.endIso)}`,
    `SUMMARY:${escapeText(e.title)}`,
    `LOCATION:${escapeText(e.location)}`,
    ...(e.description ? [`DESCRIPTION:${escapeText(e.description)}`] : []),
    'END:VEVENT',
    'END:VCALENDAR',
    '',
  ].join('\r\n');
}

export function downloadIcs(filename: string, content: string) {
  const url = URL.createObjectURL(new Blob([content], { type: 'text/calendar;charset=utf-8' }));
  const link = document.createElement('a');
  link.href = url;
  link.download = filename;
  link.click();
  URL.revokeObjectURL(url);
}
```

`src/features/booking/cancel.ts`:

```ts
import type { BookingStatus } from '@/config/statuses';

const HOUR_MS = 60 * 60 * 1000;

// Chỉ để bật/tắt nút. CSDL (hàm cancel_booking) mới là nơi quyết định thật.
export function canCancelOnline(a: {
  status: BookingStatus;
  start_at: string;
  nowIso: string;
  cancelHours: number;
}): boolean {
  if (a.status !== 'pending' && a.status !== 'confirmed') return false;
  return new Date(a.start_at).getTime() - a.cancelHours * HOUR_MS >= new Date(a.nowIso).getTime();
}
```

`src/features/booking/reminder.ts`:

```ts
import type { BookingStatus } from '@/config/statuses';

const DAY_MS = 24 * 60 * 60 * 1000;

export function needsReminder(a: { status: BookingStatus; start_at: string; nowIso: string }): boolean {
  if (a.status !== 'confirmed') return false;
  const wait = new Date(a.start_at).getTime() - new Date(a.nowIso).getTime();
  return wait > 0 && wait <= DAY_MS;
}
```

- [ ] **Step 4: Chạy để thấy qua**

Run: `npx vitest run src/features/booking`
Expected: PASS (thêm 4 + 3 + 4 test).

- [ ] **Step 5: Component và trang**

`src/components/BookingCard.tsx`:

```tsx
import type { ReactNode } from 'react';
import { PriceText } from '@/components/PriceText';
import { StatusBadge } from '@/components/StatusBadge';
import type { BookingView } from '@/features/booking/types';
import { formatDateTimeVi } from '@/lib/format';

// Dùng ở trang xác nhận, tra cứu và tài khoản.
export function BookingCard({ booking, actions }: { booking: BookingView; actions?: ReactNode }) {
  return (
    <article className="rounded-card border border-line bg-surface p-4 shadow-card">
      <div className="flex flex-wrap items-center justify-between gap-2">
        <h3 className="text-lg font-bold">{booking.service_name}</h3>
        <StatusBadge status={booking.status} />
      </div>
      <dl className="mt-2 grid gap-1 text-ink/90">
        <div className="flex gap-2"><dt className="text-ink/70">Thời gian:</dt><dd>{formatDateTimeVi(booking.start_at)}</dd></div>
        <div className="flex gap-2"><dt className="text-ink/70">Thợ:</dt><dd>{booking.staff_name}</dd></div>
        <div className="flex gap-2"><dt className="text-ink/70">Mã lịch:</dt><dd className="font-bold tracking-wider">{booking.code}</dd></div>
        <div className="flex gap-2"><dt className="text-ink/70">Tổng:</dt><dd><PriceText value={booking.total_vnd} /></dd></div>
      </dl>
      {actions && <div className="mt-3 flex flex-wrap gap-2">{actions}</div>}
    </article>
  );
}
```

`src/components/ReminderBanner.tsx`:

```tsx
import { BellRing } from 'lucide-react';
import { needsReminder } from '@/features/booking/reminder';
import type { BookingView } from '@/features/booking/types';
import { formatDateTimeVi } from '@/lib/format';

export function ReminderBanner({ booking }: { booking: BookingView }) {
  if (!needsReminder({ status: booking.status, start_at: booking.start_at, nowIso: new Date().toISOString() })) {
    return null;
  }
  return (
    <p role="status" className="flex items-center gap-2 rounded-card bg-blush p-3 text-berry">
      <BellRing aria-hidden="true" className="size-5 shrink-0" />
      Sắp tới giờ hẹn: {formatDateTimeVi(booking.start_at)}, {booking.service_name} với {booking.staff_name}.
    </p>
  );
}
```

`src/components/CancelBookingDialog.tsx`:

```tsx
import { toast } from 'sonner';
import { ConfirmDialog } from '@/components/ui/ConfirmDialog';
import { COPY } from '@/config/copy';
import { useCancelBooking } from '@/features/booking/queries';
import { toUserMessage } from '@/lib/errors';

type Props = {
  open: boolean;
  onOpenChange: (open: boolean) => void;
  phone: string;
  code: string;
  onCancelled: () => void;
};

// Dùng chung ở trang tra cứu và trang tài khoản.
export function CancelBookingDialog({ open, onOpenChange, phone, code, onCancelled }: Props) {
  const cancel = useCancelBooking();
  return (
    <ConfirmDialog
      open={open}
      onOpenChange={onOpenChange}
      tone="danger"
      title="Hủy lịch hẹn này?"
      description="Bạn không thể hoàn tác. Giờ này sẽ được nhả cho người khác."
      confirmLabel={COPY.actions.cancelBooking}
      cancelLabel={COPY.actions.keepBooking}
      pending={cancel.isPending}
      onConfirm={() =>
        cancel.mutate(
          { phone, code },
          {
            onSuccess: () => {
              toast.success('Đã hủy lịch');
              onOpenChange(false);
              onCancelled();
            },
            onError: (e) => toast.error(toUserMessage(e)),
          },
        )
      }
    />
  );
}
```

`src/pages/customer/BookingSuccessPage.tsx`:

```tsx
import { CalendarPlus, CheckCircle2 } from 'lucide-react';
import { Link, Navigate, useLocation } from 'react-router-dom';
import { BookingCard } from '@/components/BookingCard';
import { Button, LinkButton } from '@/components/ui/Button';
import { ROUTES } from '@/config/routes';
import { SITE } from '@/config/site';
import { buildIcs, downloadIcs } from '@/features/booking/ics';
import type { BookingReceipt, BookingView } from '@/features/booking/types';
import { useShopSettings } from '@/features/catalog/queries';
import { useStaff } from '@/features/catalog/queries';

type SuccessState = { receipt: BookingReceipt; service_name: string; phone: string; full_name: string };

export function BookingSuccessPage() {
  const state = useLocation().state as SuccessState | null;
  const settings = useShopSettings().data;
  const staff = useStaff().data;
  // Tải lại trang thì mất state: không còn dữ liệu để hiện, đưa khách sang trang tra cứu.
  if (!state) return <Navigate to={ROUTES.lookup} replace />;

  const { receipt, service_name } = state;
  const booking: BookingView = {
    code: receipt.code,
    status: 'pending',
    start_at: receipt.start_at,
    end_at: receipt.end_at,
    service_name,
    staff_name: staff?.find((s) => s.id === receipt.staff_id)?.name ?? '',
    total_vnd: receipt.total_vnd,
  };

  return (
    <div className="mx-auto max-w-xl px-4 py-12">
      <div className="flex items-center gap-3 text-success">
        <CheckCircle2 aria-hidden="true" className="size-8" />
        <h1 className="text-2xl font-extrabold text-success">Đã gửi yêu cầu đặt lịch</h1>
      </div>
      <p className="mt-2">Tiệm sẽ xác nhận sớm. Hãy lưu mã lịch để tra cứu hoặc hủy.</p>
      <p className="mt-4 rounded-card bg-blush p-4 text-center">
        Mã lịch của bạn: <strong className="block font-heading text-3xl tracking-widest text-rose">{receipt.code}</strong>
      </p>
      <div className="mt-4">
        <BookingCard booking={booking} />
      </div>
      <div className="mt-4 flex flex-wrap gap-3">
        <Button
          variant="secondary"
          onClick={() =>
            downloadIcs(
              `lich-${receipt.code}.ics`,
              buildIcs({
                uid: `${receipt.code}@lua-nails`,
                title: `${service_name} tại ${SITE.brandName}`,
                startIso: receipt.start_at,
                endIso: receipt.end_at,
                location: settings?.address ?? '',
                description: `Mã lịch: ${receipt.code}`,
              }),
            )
          }
        >
          <CalendarPlus aria-hidden="true" className="size-5" />
          Thêm vào lịch
        </Button>
        <LinkButton variant="secondary" to={ROUTES.lookup}>Tra cứu lịch</LinkButton>
      </div>
      <p className="mt-6">
        Muốn tích điểm và xem lịch sử? <Link className="font-medium text-rose underline" to={ROUTES.login}>Đăng ký bằng SĐT</Link>.
      </p>
    </div>
  );
}
```

Gộp hai dòng `import ... from '@/features/catalog/queries'` thành một khi viết tệp.

`src/pages/customer/LookupPage.tsx`:

```tsx
import { zodResolver } from '@hookform/resolvers/zod';
import { Phone } from 'lucide-react';
import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { toast } from 'sonner';
import { z } from 'zod';
import { BookingCard } from '@/components/BookingCard';
import { CancelBookingDialog } from '@/components/CancelBookingDialog';
import { ReminderBanner } from '@/components/ReminderBanner';
import { Button } from '@/components/ui/Button';
import { EmptyState } from '@/components/ui/EmptyState';
import { Field, Input } from '@/components/ui/Field';
import { COPY } from '@/config/copy';
import { canCancelOnline } from '@/features/booking/cancel';
import { useLookupBooking } from '@/features/booking/queries';
import { phoneSchema } from '@/features/booking/schemas';
import type { BookingView } from '@/features/booking/types';
import { useShopSettings } from '@/features/catalog/queries';
import { toUserMessage } from '@/lib/errors';

const schema = z.object({
  phone: phoneSchema,
  code: z.string().trim().min(6, 'Mã lịch gồm 6 ký tự.').max(6, 'Mã lịch gồm 6 ký tự.'),
});
type Form = z.infer<typeof schema>;

export function LookupPage() {
  const { register, handleSubmit, formState: { errors } } = useForm<Form>({ resolver: zodResolver(schema) });
  const lookup = useLookupBooking();
  const settings = useShopSettings().data;
  const [found, setFound] = useState<{ booking: BookingView; phone: string } | null | undefined>();
  const [confirming, setConfirming] = useState(false);

  const onSubmit = (v: Form) =>
    lookup.mutate(
      { phone: v.phone, code: v.code.toUpperCase() },
      { onSuccess: (booking) => setFound(booking ? { booking, phone: v.phone } : null), onError: (e) => toast.error(toUserMessage(e)) },
    );

  const booking = found?.booking;
  const cancellable =
    booking && settings
      ? canCancelOnline({ status: booking.status, start_at: booking.start_at, nowIso: new Date().toISOString(), cancelHours: settings.cancel_hours })
      : false;

  return (
    <div className="mx-auto max-w-xl px-4 py-10">
      <h1 className="text-3xl font-extrabold">Tra cứu lịch hẹn</h1>
      <form onSubmit={handleSubmit(onSubmit)} noValidate className="mt-6 grid gap-4">
        <Field label="Số điện thoại" required error={errors.phone?.message}>
          {(a) => <Input {...a} {...register('phone')} type="tel" inputMode="tel" autoComplete="tel" />}
        </Field>
        <Field label="Mã lịch" required error={errors.code?.message} hint="Có trong màn hình xác nhận khi bạn đặt lịch">
          {(a) => <Input {...a} {...register('code')} autoCapitalize="characters" maxLength={6} />}
        </Field>
        <Button type="submit" disabled={lookup.isPending}>Tra cứu</Button>
      </form>

      <div className="mt-8 grid gap-4">
        {found === null && (
          <EmptyState title={COPY.errors.booking_not_found} description="Quên mã lịch? Hãy gọi tiệm để được hỗ trợ." />
        )}
        {booking && (
          <>
            <ReminderBanner booking={booking} />
            <BookingCard
              booking={booking}
              actions={
                cancellable ? (
                  <Button variant="danger" onClick={() => setConfirming(true)}>{COPY.actions.cancelBooking}</Button>
                ) : booking.status === 'pending' || booking.status === 'confirmed' ? (
                  <a className="inline-flex min-h-11 items-center gap-2 font-medium text-rose underline" href={`tel:${settings?.phone ?? ''}`}>
                    <Phone aria-hidden="true" className="size-4" />
                    {COPY.errors.cancel_too_late} {settings?.phone}
                  </a>
                ) : null
              }
            />
            <CancelBookingDialog
              open={confirming}
              onOpenChange={setConfirming}
              phone={found!.phone}
              code={booking.code}
              onCancelled={() => setFound({ booking: { ...booking, status: 'cancelled' }, phone: found!.phone })}
            />
          </>
        )}
      </div>
    </div>
  );
}
```

Trong `router.tsx` thêm:

```tsx
const BookingSuccessPage = lazyPage(() => import('@/pages/customer/BookingSuccessPage'), 'BookingSuccessPage');
const LookupPage = lazyPage(() => import('@/pages/customer/LookupPage'), 'LookupPage');
// ...và thêm vào customerRoutes:
page(ROUTES.bookingSuccess, BookingSuccessPage),
page(ROUTES.lookup, LookupPage),
```

- [ ] **Step 6: Kiểm tra**

Run: `npm run check`
Expected: qua. Kiểm tra tay: đặt lịch xong thấy mã 6 ký tự và nút "Thêm vào lịch" tải được tệp `.ics`; tra cứu đúng thì thấy lịch; sai mã thì thấy câu "Không tìm thấy lịch"; hủy lịch có hộp thoại xác nhận với nút "Giữ lịch" nổi bật.

- [ ] **Step 7: Commit**

```bash
git add -A
git commit -m "feat(booking): trang xác nhận, tra cứu, hủy lịch và tệp lịch .ics"
```

---

## Task 9: Đăng nhập SĐT + PIN và trang tài khoản

**Files:**
- Create: `src/lib/auth.ts`, `src/features/auth/AuthProvider.tsx`, `src/features/auth/RequireAuth.tsx`, `src/features/auth/queries.ts`, `src/features/points/milestone.ts`
- Create: `src/components/ui/PinField.tsx`, `src/pages/customer/LoginPage.tsx`, `src/pages/customer/AccountPage.tsx`
- Modify: `src/components/layout/AccountLink.tsx`, `src/app/App.tsx`, `src/app/router.tsx`, `src/config/booking.ts` (thêm `LOYALTY`)
- Test: `src/lib/auth.test.ts`, `src/features/points/milestone.test.ts`

**Interfaces:**
- Consumes: `loginSchema`, `registerSchema`, `normalizePhone`, `SITE.authEmailDomain`, `BookingCard`, `ReminderBanner`, `CancelBookingDialog`.
- Produces:
  - `phoneToEmail(phone: string): string`, `signInWithPhone({ phone, pin })`, `signUpWithPhone({ phone, pin, full_name })`, `signOut()`
  - `AuthProvider`, `useAuth(): { session: Session | null; loading: boolean }`, `RequireAuth`
  - `useMyCustomer()`, `useMyBookings()`, `useProfile()` (đều trả `UseQueryResult`)
  - `progressToMilestone(balance: number, milestone: number): { current: number; remaining: number; percent: number }`
  - `LOYALTY = { milestone: 200 }`

**Điều kiện trước (làm tay một lần trên Supabase):** Authentication → Providers → Email: **tắt "Confirm email"** (để đăng ký xong có phiên ngay, vì email là giả lập từ SĐT).

- [ ] **Step 1: Viết test hỏng**

`src/lib/auth.test.ts`:

```ts
import { SITE } from '@/config/site';
import { phoneToEmail } from './auth';

describe('phoneToEmail', () => {
  it('dựng email giả lập từ SĐT đã chuẩn hóa', () => {
    expect(phoneToEmail('0901234567')).toBe(`0901234567@${SITE.authEmailDomain}`);
  });

  it('SĐT nhập kiểu khác nhau cho cùng một email (cùng một tài khoản)', () => {
    expect(phoneToEmail('0901 234 567')).toBe(phoneToEmail('+84901234567'));
  });
});
```

`src/features/points/milestone.test.ts`:

```ts
import { progressToMilestone } from './milestone';

describe('progressToMilestone', () => {
  it('140 điểm với mốc 200: còn 60 điểm, 70%', () => {
    expect(progressToMilestone(140, 200)).toEqual({ current: 140, remaining: 60, percent: 70 });
  });
  it('đúng mốc thì bắt đầu vòng mới', () => {
    expect(progressToMilestone(200, 200)).toEqual({ current: 0, remaining: 200, percent: 0 });
  });
  it('vượt mốc thì tính phần dư', () => {
    expect(progressToMilestone(230, 200)).toEqual({ current: 30, remaining: 170, percent: 15 });
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/lib/auth.test.ts src/features/points`
Expected: FAIL.

- [ ] **Step 3: Viết hàm thuần**

`src/lib/auth.ts`:

```ts
import { SITE } from '@/config/site';
import { AppError } from '@/lib/errors';
import { normalizePhone } from '@/lib/phone';
import { supabase } from '@/lib/supabase';

// Supabase Auth cần email; ta dựng email giả từ SĐT để khách chỉ phải nhớ SĐT và PIN.
export const phoneToEmail = (phone: string) => `${normalizePhone(phone)}@${SITE.authEmailDomain}`;

export async function signInWithPhone(v: { phone: string; pin: string }) {
  const { error } = await supabase.auth.signInWithPassword({ email: phoneToEmail(v.phone), password: v.pin });
  if (error) throw new AppError('badCredentials');
}

export async function signUpWithPhone(v: { phone: string; pin: string; full_name: string }) {
  const { error } = await supabase.auth.signUp({ email: phoneToEmail(v.phone), password: v.pin });
  if (error) throw new AppError(/already/i.test(error.message) ? 'phone_taken' : error.message);
  const registered = await supabase.rpc('register_customer', { p_name: v.full_name, p_phone: normalizePhone(v.phone) });
  if (registered.error) throw new AppError(registered.error.message);
}

export const signOut = async () => {
  await supabase.auth.signOut();
};
```

`src/features/points/milestone.ts`:

```ts
export function progressToMilestone(balance: number, milestone: number) {
  const current = balance % milestone;
  return { current, remaining: milestone - current, percent: Math.round((current / milestone) * 100) };
}
```

Trong `src/config/booking.ts` thêm:

```ts
// Mốc điểm hiển thị ở trang tài khoản. Đổi điểm lấy ưu đãi thuộc gói Nâng cấp.
export const LOYALTY = { milestone: 200 } as const;
```

- [ ] **Step 4: Chạy để thấy qua**

Run: `npx vitest run src/lib/auth.test.ts src/features/points`
Expected: PASS.

- [ ] **Step 5: Phiên đăng nhập, truy vấn và bảo vệ trang**

`src/features/auth/AuthProvider.tsx`:

```tsx
import type { Session } from '@supabase/supabase-js';
import { createContext, useContext, useEffect, useState, type ReactNode } from 'react';
import { supabase } from '@/lib/supabase';

type AuthState = { session: Session | null; loading: boolean };
const AuthContext = createContext<AuthState>({ session: null, loading: true });

export function AuthProvider({ children }: { children: ReactNode }) {
  const [state, setState] = useState<AuthState>({ session: null, loading: true });
  useEffect(() => {
    void supabase.auth.getSession().then(({ data }) => setState({ session: data.session, loading: false }));
    const { data } = supabase.auth.onAuthStateChange((_event, session) => setState({ session, loading: false }));
    return () => data.subscription.unsubscribe();
  }, []);
  return <AuthContext.Provider value={state}>{children}</AuthContext.Provider>;
}

// eslint-disable-next-line react-refresh/only-export-components -- hook đi kèm provider, giữ chung một tệp
export const useAuth = () => useContext(AuthContext);
```

`src/features/auth/RequireAuth.tsx`:

```tsx
import { Navigate, Outlet, useLocation } from 'react-router-dom';
import { Skeleton } from '@/components/ui/Skeleton';
import { ROUTES } from '@/config/routes';
import { useAuth } from './AuthProvider';

export function RequireAuth() {
  const { session, loading } = useAuth();
  const location = useLocation();
  if (loading) return <Skeleton className="mx-auto my-16 h-64 max-w-[1120px]" />;
  if (!session) return <Navigate to={ROUTES.login} state={{ from: location.pathname }} replace />;
  return <Outlet />;
}
```

`src/features/auth/queries.ts`:

```ts
import { useQuery } from '@tanstack/react-query';
import { queryKeys } from '@/lib/queryKeys';
import { supabase } from '@/lib/supabase';
import { unwrap } from '@/lib/api/unwrap';
import type { BookingView } from '@/features/booking/types';
import type { Customer } from '@/types/db';
import { useAuth } from './AuthProvider';

type Profile = { id: string; role: 'customer' | 'admin'; full_name: string; phone: string };

export function useProfile() {
  const { session } = useAuth();
  return useQuery({
    queryKey: [...queryKeys.profile, session?.user.id],
    enabled: !!session,
    queryFn: async (): Promise<Profile> =>
      unwrap(await supabase.from('profiles').select('*').eq('id', session!.user.id).single()),
  });
}

export function useMyCustomer() {
  const { session } = useAuth();
  return useQuery({
    queryKey: [...queryKeys.myCustomer, session?.user.id],
    enabled: !!session,
    queryFn: async (): Promise<Customer> =>
      unwrap(await supabase.from('customers').select('*').eq('profile_id', session!.user.id).single()),
  });
}

export function useMyBookings() {
  const { session } = useAuth();
  return useQuery({
    queryKey: [...queryKeys.myBookings, session?.user.id],
    enabled: !!session,
    queryFn: async (): Promise<BookingView[]> => {
      const rows = unwrap(
        await supabase
          .from('bookings')
          .select('code, status, start_at, end_at, total_vnd, services(name), staff(name)')
          .order('start_at', { ascending: false }),
      ) as unknown as {
        code: string; status: BookingView['status']; start_at: string; end_at: string; total_vnd: number;
        services: { name: string }; staff: { name: string };
      }[];
      return rows.map((r) => ({
        code: r.code, status: r.status, start_at: r.start_at, end_at: r.end_at, total_vnd: r.total_vnd,
        service_name: r.services.name, staff_name: r.staff.name,
      }));
    },
  });
}
```

Cập nhật `src/components/layout/AccountLink.tsx` để hiện tên khi đã đăng nhập:

```tsx
import { Link } from 'react-router-dom';
import { COPY } from '@/config/copy';
import { ROUTES } from '@/config/routes';
import { useAuth } from '@/features/auth/AuthProvider';

export function AccountLink() {
  const { session } = useAuth();
  return (
    <Link
      className="min-h-11 content-center px-2 font-medium text-berry hover:underline"
      to={session ? ROUTES.account : ROUTES.login}
    >
      {session ? 'Tài khoản' : COPY.actions.login}
    </Link>
  );
}
```

Bọc `AuthProvider` trong `src/app/App.tsx`: đặt `<AuthProvider>` giữa `QueryClientProvider` và `RouterProvider`.

- [ ] **Step 6: Ô PIN, trang đăng nhập và tài khoản**

`src/components/ui/PinField.tsx`:

```tsx
import { Eye, EyeOff } from 'lucide-react';
import { forwardRef, useState, type InputHTMLAttributes } from 'react';
import { inputClass } from './Field';
import { cn } from '@/lib/cn';

// Một ô 6 số, cho dán và trình quản lý mật khẩu, có nút hiện/ẩn.
export const PinField = forwardRef<HTMLInputElement, InputHTMLAttributes<HTMLInputElement>>(
  ({ className, ...props }, ref) => {
    const [shown, setShown] = useState(false);
    return (
      <div className="relative">
        <input
          ref={ref}
          type={shown ? 'text' : 'password'}
          inputMode="numeric"
          maxLength={6}
          className={cn(inputClass, 'pr-12 tracking-[0.4em]', className)}
          {...props}
        />
        <button
          type="button"
          aria-label={shown ? 'Ẩn PIN' : 'Hiện PIN'}
          onClick={() => setShown((s) => !s)}
          className="absolute right-1 top-0 grid size-11 place-items-center text-berry"
        >
          {shown ? <EyeOff aria-hidden="true" className="size-5" /> : <Eye aria-hidden="true" className="size-5" />}
        </button>
      </div>
    );
  },
);
PinField.displayName = 'PinField';
```

`src/pages/customer/LoginPage.tsx`:

```tsx
import { zodResolver } from '@hookform/resolvers/zod';
import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { Link, useLocation, useNavigate } from 'react-router-dom';
import { toast } from 'sonner';
import { Button } from '@/components/ui/Button';
import { Field, Input } from '@/components/ui/Field';
import { PinField } from '@/components/ui/PinField';
import { TabBar } from '@/components/ui/TabBar';
import { ROUTES } from '@/config/routes';
import { loginSchema, registerSchema, type LoginForm, type RegisterForm } from '@/features/auth/schemas';
import { signInWithPhone, signUpWithPhone } from '@/lib/auth';
import { toUserMessage } from '@/lib/errors';

export function LoginPage() {
  const [mode, setMode] = useState<'login' | 'register'>('login');
  const navigate = useNavigate();
  const from = (useLocation().state as { from?: string } | null)?.from ?? ROUTES.account;
  const done = () => navigate(from, { replace: true });
  const fail = (e: unknown) => toast.error(toUserMessage(e));

  return (
    <div className="mx-auto grid max-w-4xl gap-8 px-4 py-10 md:grid-cols-2">
      <div className="hidden rounded-card bg-gradient-to-br from-plum-1 to-plum-2 p-8 text-white md:block">
        <h1 className="text-3xl font-extrabold text-white">Xem lịch sử, tích điểm mỗi lần làm</h1>
        <p className="mt-3 text-white/85">Đăng nhập bằng SĐT và PIN 6 số. Không cần tài khoản vẫn đặt lịch được.</p>
      </div>
      <div>
        <h1 className="mb-4 text-2xl font-extrabold md:hidden">Đăng nhập</h1>
        <TabBar
          value={mode}
          onValueChange={(v) => setMode(v as 'login' | 'register')}
          items={[{ value: 'login', label: 'Đăng nhập' }, { value: 'register', label: 'Đăng ký' }]}
        />
        <div className="mt-6">{mode === 'login' ? <LoginForm onDone={done} onError={fail} /> : <RegisterForm onDone={done} onError={fail} />}</div>
        <p className="mt-6">
          Không cần tài khoản? <Link className="font-medium text-rose underline" to={ROUTES.booking}>Đặt lịch ngay</Link>.
        </p>
      </div>
    </div>
  );
}

type FormProps = { onDone: () => void; onError: (e: unknown) => void };

function LoginForm({ onDone, onError }: FormProps) {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<LoginForm>({ resolver: zodResolver(loginSchema) });
  return (
    <form noValidate className="grid gap-4" onSubmit={handleSubmit(async (v) => { try { await signInWithPhone(v); onDone(); } catch (e) { onError(e); } })}>
      <Field label="Số điện thoại" required error={errors.phone?.message}>
        {(a) => <Input {...a} {...register('phone')} type="tel" inputMode="tel" autoComplete="username" />}
      </Field>
      <Field label="PIN 6 số" required error={errors.pin?.message}>
        {(a) => <PinField {...a} {...register('pin')} autoComplete="current-password" />}
      </Field>
      <Button type="submit" disabled={isSubmitting}>Đăng nhập</Button>
      <p className="text-sm text-ink/80">Quên PIN? Liên hệ tiệm để được đặt lại.</p>
    </form>
  );
}

function RegisterForm({ onDone, onError }: FormProps) {
  const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm<RegisterForm>({ resolver: zodResolver(registerSchema) });
  return (
    <form noValidate className="grid gap-4" onSubmit={handleSubmit(async (v) => { try { await signUpWithPhone(v); onDone(); } catch (e) { onError(e); } })}>
      <Field label="Họ tên" required error={errors.full_name?.message}>
        {(a) => <Input {...a} {...register('full_name')} autoComplete="name" />}
      </Field>
      <Field label="Số điện thoại" required error={errors.phone?.message}>
        {(a) => <Input {...a} {...register('phone')} type="tel" inputMode="tel" autoComplete="username" />}
      </Field>
      <Field label="Tạo PIN 6 số" required error={errors.pin?.message}>
        {(a) => <PinField {...a} {...register('pin')} autoComplete="new-password" />}
      </Field>
      <Field label="Nhập lại PIN" required error={errors.pin_confirm?.message}>
        {(a) => <PinField {...a} {...register('pin_confirm')} autoComplete="new-password" />}
      </Field>
      <Button type="submit" disabled={isSubmitting}>Tạo tài khoản</Button>
    </form>
  );
}
```

`src/pages/customer/AccountPage.tsx`:

```tsx
import { useQueryClient } from '@tanstack/react-query';
import { useState } from 'react';
import { BookingCard } from '@/components/BookingCard';
import { CancelBookingDialog } from '@/components/CancelBookingDialog';
import { QueryState } from '@/components/QueryState';
import { ReminderBanner } from '@/components/ReminderBanner';
import { Button, LinkButton } from '@/components/ui/Button';
import { EmptyState } from '@/components/ui/EmptyState';
import { LOYALTY } from '@/config/booking';
import { COPY } from '@/config/copy';
import { ROUTES } from '@/config/routes';
import { useMyBookings, useMyCustomer } from '@/features/auth/queries';
import { canCancelOnline } from '@/features/booking/cancel';
import type { BookingView } from '@/features/booking/types';
import { useShopSettings } from '@/features/catalog/queries';
import { progressToMilestone } from '@/features/points/milestone';
import { signOut } from '@/lib/auth';
import { queryKeys } from '@/lib/queryKeys';

export function AccountPage() {
  const client = useQueryClient();
  const customer = useMyCustomer();
  const bookings = useMyBookings();
  const settings = useShopSettings().data;
  const [cancelling, setCancelling] = useState<BookingView | null>(null);
  const now = new Date().toISOString();

  return (
    <div className="mx-auto max-w-[1120px] px-4 py-10 md:px-8">
      <div className="flex flex-wrap items-center justify-between gap-3">
        <QueryState query={customer}>{(c) => <h1 className="text-3xl font-extrabold">Chào {c.full_name}</h1>}</QueryState>
        <div className="flex gap-2">
          <LinkButton to={ROUTES.booking}>Đặt lịch mới</LinkButton>
          <Button variant="secondary" onClick={() => void signOut()}>{COPY.actions.logout}</Button>
        </div>
      </div>

      <QueryState query={customer}>
        {(c) => {
          const p = progressToMilestone(c.points_balance, LOYALTY.milestone);
          return (
            <section aria-label="Điểm thưởng" className="mt-6 rounded-card bg-gradient-to-br from-plum-1 to-plum-2 p-6 text-white">
              <p className="text-white/85">Điểm của bạn</p>
              <p className="font-heading text-5xl font-extrabold tabular-nums">{c.points_balance}</p>
              <div className="mt-3 h-2 overflow-hidden rounded-full bg-white/25" role="progressbar" aria-valuenow={p.percent} aria-valuemin={0} aria-valuemax={100} aria-label="Tiến độ tới mốc điểm">
                <div className="h-full bg-gel-pink" style={{ width: `${p.percent}%` }} />
              </div>
              <p className="mt-2 text-white/85">Còn {p.remaining} điểm tới mốc {LOYALTY.milestone}. Đổi điểm lấy ưu đãi có ở gói sau.</p>
            </section>
          );
        }}
      </QueryState>

      <section className="mt-8">
        <h2 className="text-2xl font-extrabold">Lịch hẹn</h2>
        <div className="mt-4 grid gap-4">
          <QueryState query={bookings} isEmpty={(b) => b.length === 0} empty={<EmptyState title={COPY.empty.noBookings} action={<LinkButton to={ROUTES.booking}>Đặt lịch</LinkButton>} />}>
            {(all) =>
              all.map((b) => (
                <div key={b.code} className="grid gap-2">
                  <ReminderBanner booking={b} />
                  <BookingCard
                    booking={b}
                    actions={
                      settings && canCancelOnline({ status: b.status, start_at: b.start_at, nowIso: now, cancelHours: settings.cancel_hours }) ? (
                        <Button variant="danger" onClick={() => setCancelling(b)}>{COPY.actions.cancelBooking}</Button>
                      ) : null
                    }
                  />
                </div>
              ))
            }
          </QueryState>
        </div>
      </section>

      {cancelling && customer.data && (
        <CancelBookingDialog
          open
          onOpenChange={(open) => !open && setCancelling(null)}
          phone={customer.data.phone}
          code={cancelling.code}
          onCancelled={() => {
            setCancelling(null);
            void client.invalidateQueries({ queryKey: queryKeys.myBookings });
          }}
        />
      )}
    </div>
  );
}
```

Trong `router.tsx` thêm (trang tài khoản nằm trong `RequireAuth`):

```tsx
import { RequireAuth } from '@/features/auth/RequireAuth';

const LoginPage = lazyPage(() => import('@/pages/customer/LoginPage'), 'LoginPage');
const AccountPage = lazyPage(() => import('@/pages/customer/AccountPage'), 'AccountPage');
// ...và thêm vào customerRoutes:
page(ROUTES.login, LoginPage),
{ element: <RequireAuth />, children: [page(ROUTES.account, AccountPage)] },
```

- [ ] **Step 7: Kiểm tra**

Run: `npm run check`
Expected: qua. Kiểm tra tay: đăng ký bằng SĐT mới và PIN `123456` thì vào trang tài khoản; đăng xuất rồi đăng nhập lại; đăng ký SĐT đã từng đặt lịch vãng lai thì thấy lịch cũ ở "Lịch hẹn"; PIN sai báo "SĐT hoặc PIN chưa đúng."; truy cập `/tai-khoan` khi chưa đăng nhập thì chuyển sang `/dang-nhap`.

- [ ] **Step 8: Commit**

```bash
git add -A
git commit -m "feat(auth): đăng nhập và đăng ký bằng SĐT + PIN, trang tài khoản có điểm và nhắc lịch"
```

---

## Task 10: Trang liên hệ

**Files:**
- Create: `src/pages/customer/ContactPage.tsx`
- Modify: `src/app/router.tsx`
- Test: `src/pages/customer/ContactPage.test.tsx`

**Interfaces:**
- Consumes: `useShopSettings`, `useOpeningHours`, `WEEKDAY_LABELS`, `todayVn`, `weekdayOf`, `minutesToLabel`, `timeToMinutes`.
- Produces: `ContactPage`.

- [ ] **Step 1: Viết test hỏng**

`src/pages/customer/ContactPage.test.tsx`:

```tsx
import { render, screen } from '@testing-library/react';
import type { OpeningHours } from '@/types/db';
import { ContactPage } from './ContactPage';

const hours: OpeningHours[] = [0, 1, 2, 3, 4, 5, 6].map((weekday) => ({
  weekday, open_time: '09:00:00', close_time: '20:00:00', is_closed: weekday === 0,
}));

vi.mock('@/features/catalog/queries', () => ({
  useShopSettings: () => ({
    isPending: false, isError: false,
    data: { address: '123 Đường Hoa Lan', phone: '0901234567', zalo: '0901234567', map_url: 'https://maps.example/x' },
  }),
  useOpeningHours: () => ({ isPending: false, isError: false, data: hours }),
}));

describe('ContactPage', () => {
  it('SĐT bấm gọi được và có nút chỉ đường mở bản đồ', () => {
    render(<ContactPage />);
    expect(screen.getByRole('link', { name: '0901234567' })).toHaveAttribute('href', 'tel:0901234567');
    expect(screen.getByRole('link', { name: /Chỉ đường/ })).toHaveAttribute('href', 'https://maps.example/x');
  });

  it('bảng giờ mở cửa đủ 7 ngày, ngày nghỉ ghi rõ "Nghỉ"', () => {
    render(<ContactPage />);
    expect(screen.getAllByRole('row')).toHaveLength(7);
    expect(screen.getByText('Nghỉ')).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Chạy để thấy hỏng**

Run: `npx vitest run src/pages/customer/ContactPage.test.tsx`
Expected: FAIL.

- [ ] **Step 3: Viết code**

`src/pages/customer/ContactPage.tsx`:

```tsx
import { MapPin, Phone } from 'lucide-react';
import { QueryState } from '@/components/QueryState';
import { cn } from '@/lib/cn';
import { WEEKDAY_LABELS } from '@/config/site';
import { useOpeningHours, useShopSettings } from '@/features/catalog/queries';
import { minutesToLabel, timeToMinutes, todayVn, weekdayOf } from '@/lib/vnTime';

const label = (time: string) => minutesToLabel(timeToMinutes(time));

export function ContactPage() {
  const settings = useShopSettings();
  const hours = useOpeningHours();
  const today = weekdayOf(todayVn());

  return (
    <div className="mx-auto max-w-[1120px] px-4 py-10 md:px-8">
      <h1 className="text-3xl font-extrabold">Liên hệ</h1>
      <div className="mt-6 grid gap-6 md:grid-cols-2">
        <div className="grid content-start gap-6">
          <QueryState query={settings}>
            {(s) => (
              <ul className="space-y-3">
                <li className="flex gap-3"><MapPin aria-hidden="true" className="mt-1 size-5 shrink-0 text-rose" />{s.address}</li>
                <li className="flex gap-3">
                  <Phone aria-hidden="true" className="mt-1 size-5 shrink-0 text-rose" />
                  <a className="underline" href={`tel:${s.phone}`}>{s.phone}</a>
                </li>
                {s.zalo && (
                  <li className="pl-8">
                    <a className="underline" href={`https://zalo.me/${s.zalo}`}>Nhắn Zalo</a>
                  </li>
                )}
              </ul>
            )}
          </QueryState>
          <QueryState query={hours}>
            {(rows) => (
              <table className="w-full rounded-card border border-line bg-surface">
                <caption className="sr-only">Giờ mở cửa</caption>
                <tbody>
                  {rows.map((h) => (
                    <tr key={h.weekday} className={cn('border-b border-line last:border-0', h.weekday === today && 'bg-blush font-bold')}>
                      <th scope="row" className="p-3 text-left font-medium">{WEEKDAY_LABELS[h.weekday]}{h.weekday === today && ' (hôm nay)'}</th>
                      <td className="p-3 text-right tabular-nums">{h.is_closed ? 'Nghỉ' : `${label(h.open_time)} đến ${label(h.close_time)}`}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            )}
          </QueryState>
        </div>
        <QueryState query={settings}>
          {(s) => (
            <div className="flex min-h-64 flex-col items-center justify-center gap-4 rounded-card bg-gradient-to-br from-blush to-surface p-6 text-center">
              <MapPin aria-hidden="true" className="size-10 text-rose" />
              <p className="text-ink/80">{s.address}</p>
              {s.map_url && (
                <a className="inline-flex min-h-11 items-center rounded-control bg-rose px-5 font-semibold text-white hover:bg-berry" href={s.map_url} target="_blank" rel="noreferrer">
                  Chỉ đường
                </a>
              )}
            </div>
          )}
        </QueryState>
      </div>
    </div>
  );
}
```

Trong `router.tsx` thêm:

```tsx
const ContactPage = lazyPage(() => import('@/pages/customer/ContactPage'), 'ContactPage');
// ...và thêm vào customerRoutes:
page(ROUTES.contact, ContactPage),
```

- [ ] **Step 4: Chạy để thấy qua**

Run: `npx vitest run src/pages/customer/ContactPage.test.tsx`
Expected: PASS (2 test).

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "feat(contact): trang liên hệ có giờ mở cửa từng ngày và chỉ đường"
```

---

## Task 11: Kiểm thử trình duyệt luồng khách (Playwright)

**Files:**
- Create: `playwright.config.ts`, `tests/e2e/helpers.ts`, `tests/e2e/booking.spec.ts`, `tests/e2e/double-booking.spec.ts`, `tests/e2e/auth.spec.ts`, `tests/e2e/responsive.spec.ts`
- Modify: `package.json` (script `e2e`), `.gitignore` đã có sẵn mục Playwright

**Interfaces:**
- Consumes: giao diện đang chạy trên `http://localhost:5173` nối với project Supabase **thử nghiệm** đã nạp dữ liệu mẫu.
- Produces: lệnh `npm run e2e`; bộ test chạy ở 3 cỡ màn hình.

- [ ] **Step 1: Cài Playwright**

```powershell
npm install -D @playwright/test
npx playwright install chromium
```

Thêm vào `scripts` của `package.json`: `"e2e": "playwright test"`.

- [ ] **Step 2: Cấu hình**

`playwright.config.ts`:

```ts
import { defineConfig } from '@playwright/test';

const viewports = [
  { name: 'dien-thoai', width: 375, height: 812 },
  { name: 'tablet', width: 768, height: 1024 },
  { name: 'may-tinh', width: 1280, height: 800 },
];

export default defineConfig({
  testDir: 'tests/e2e',
  timeout: 60_000,
  fullyParallel: false,
  use: { baseURL: 'http://localhost:5173', locale: 'vi-VN', timezoneId: 'Asia/Ho_Chi_Minh', trace: 'retain-on-failure' },
  projects: viewports.map((v) => ({ name: v.name, use: { viewport: { width: v.width, height: v.height } } })),
  webServer: { command: 'npm run dev', url: 'http://localhost:5173', reuseExistingServer: true },
});
```

`tests/e2e/helpers.ts`:

```ts
import type { Page } from '@playwright/test';

// SĐT ngẫu nhiên hợp lệ để mỗi lần chạy không đụng dữ liệu cũ.
export const randomPhone = () => `09${String(Math.floor(Math.random() * 1e8)).padStart(8, '0')}`;

// Đi qua các bước Dịch vụ, Thợ, Ngày, Giờ; dừng ở bước nhập thông tin.
// Chọn ngày thứ dayIndex (từ 0 là hôm nay) để tránh giờ đã qua của hôm nay.
export async function pickFirstSlot(page: Page, serviceName = 'Gel tay', dayIndex = 2) {
  await page.goto('/dat-lich');
  await page.getByRole('listitem').filter({ hasText: serviceName }).getByRole('button', { name: 'Chọn' }).first().click();
  await page.getByRole('button', { name: /Bất kỳ/ }).click();
  await page.getByRole('list', { name: 'Chọn ngày' }).getByRole('button').nth(dayIndex).click();
  await page.getByRole('list', { name: 'Chọn giờ' }).getByRole('button', { disabled: false }).first().click();
}
```

Lưu ý: `pickFirstSlot` dựa vào hai danh sách có nhãn trợ năng `Chọn ngày` (`DateStrip`) và `Chọn giờ` (`SlotGrid`) ở Task 7 và vào dòng dịch vụ (`ServiceRow`, thẻ `li`). Giữ hàm này là nơi duy nhất mô tả các thao tác đặt lịch trong E2E; nếu một bộ chọn không khớp thì sửa bộ chọn theo tên hiển thị, không thêm `data-testid`.

- [ ] **Step 3: Viết test luồng đặt lịch và hủy**

`tests/e2e/booking.spec.ts`:

```ts
import { expect, test } from '@playwright/test';
import { pickFirstSlot, randomPhone } from './helpers';

test('khách vãng lai đặt lịch, tra cứu bằng SĐT + mã, rồi hủy', async ({ page }) => {
  const phone = randomPhone();
  await pickFirstSlot(page);

  await page.getByLabel(/Họ tên/).fill('Khách Thử E2E');
  await page.getByLabel(/Số điện thoại/).fill(phone);
  await page.getByRole('button', { name: 'Xác nhận đặt lịch' }).last().click();

  await expect(page.getByRole('heading', { name: 'Đã gửi yêu cầu đặt lịch' })).toBeVisible();
  const code = (await page.locator('strong.font-heading').innerText()).trim();
  expect(code).toMatch(/^[A-HJ-KM-NP-Z2-9]{6}$/);

  await page.goto('/tra-cuu');
  await page.getByLabel(/Số điện thoại/).fill(phone);
  await page.getByLabel(/Mã lịch/).fill(code);
  await page.getByRole('button', { name: 'Tra cứu' }).click();
  await expect(page.getByText('Chờ xác nhận')).toBeVisible();

  await page.getByRole('button', { name: 'Hủy lịch' }).click();
  await page.getByRole('dialog').getByRole('button', { name: 'Hủy lịch' }).click();
  await expect(page.getByText('Đã hủy', { exact: true })).toBeVisible();
});

test('tra cứu sai mã thì báo không tìm thấy', async ({ page }) => {
  await page.goto('/tra-cuu');
  await page.getByLabel(/Số điện thoại/).fill(randomPhone());
  await page.getByLabel(/Mã lịch/).fill('ZZZZZZ');
  await page.getByRole('button', { name: 'Tra cứu' }).click();
  await expect(page.getByText(/Không tìm thấy lịch/)).toBeVisible();
});
```

- [ ] **Step 4: Viết test chống trùng giờ (hai người cùng giờ cùng thợ)**

`tests/e2e/double-booking.spec.ts`:

```ts
import { expect, test } from '@playwright/test';
import { pickFirstSlot, randomPhone } from './helpers';

test('hai người cùng chọn một giờ của cùng một thợ: chỉ một người đặt được', async ({ browser }) => {
  const a = await (await browser.newContext()).newPage();
  const b = await (await browser.newContext()).newPage();

  // Cả hai đi tới bước nhập thông tin với cùng thợ, cùng ngày, cùng giờ trống đầu tiên.
  for (const page of [a, b]) {
    await pickFirstSlot(page);
  }
  const fill = async (page: typeof a, name: string) => {
    await page.getByLabel(/Họ tên/).fill(name);
    await page.getByLabel(/Số điện thoại/).fill(randomPhone());
  };
  await fill(a, 'Người A');
  await fill(b, 'Người B');

  await a.getByRole('button', { name: 'Xác nhận đặt lịch' }).last().click();
  await expect(a.getByRole('heading', { name: 'Đã gửi yêu cầu đặt lịch' })).toBeVisible();

  // Người B bấm sau: nếu hệ thống đã gán thợ "bất kỳ" cho A, B có thể nhận thợ khác và thành công.
  // Để ép trùng, test này chọn "bất kỳ" nên chấp nhận một trong hai kết quả hợp lệ, nhưng không bao giờ
  // được có hai lịch cùng thợ cùng giờ (ràng buộc ở CSDL, kiểm tra trong supabase/tests).
  await b.getByRole('button', { name: 'Xác nhận đặt lịch' }).last().click();
  const ok = b.getByRole('heading', { name: 'Đã gửi yêu cầu đặt lịch' });
  const taken = b.getByText('Giờ này vừa có người đặt. Chọn giờ khác nhé.');
  await expect(ok.or(taken)).toBeVisible();
});
```

- [ ] **Step 5: Viết test đăng ký/đăng nhập và test không cuộn ngang**

`tests/e2e/auth.spec.ts`:

```ts
import { expect, test } from '@playwright/test';
import { randomPhone } from './helpers';

test('đăng ký bằng SĐT + PIN, đăng xuất rồi đăng nhập lại, thấy điểm', async ({ page }) => {
  const phone = randomPhone();
  await page.goto('/dang-nhap');
  await page.getByRole('tab', { name: 'Đăng ký' }).click();
  await page.getByLabel(/Họ tên/).fill('Khách Đăng Ký');
  await page.getByLabel(/Số điện thoại/).fill(phone);
  await page.getByLabel('Tạo PIN 6 số').fill('123456');
  await page.getByLabel('Nhập lại PIN').fill('123456');
  await page.getByRole('button', { name: 'Tạo tài khoản' }).click();

  await expect(page.getByRole('heading', { name: /Chào Khách Đăng Ký/ })).toBeVisible();
  await expect(page.getByText('Điểm của bạn')).toBeVisible();

  await page.getByRole('button', { name: 'Đăng xuất' }).click();
  await page.goto('/dang-nhap');
  await page.getByLabel(/Số điện thoại/).fill(phone);
  await page.getByLabel(/PIN 6 số/).fill('123456');
  await page.getByRole('button', { name: 'Đăng nhập' }).last().click();
  await expect(page.getByRole('heading', { name: /Chào Khách Đăng Ký/ })).toBeVisible();
});

test('PIN sai thì báo lỗi chung, không lộ SĐT có tồn tại hay không', async ({ page }) => {
  await page.goto('/dang-nhap');
  await page.getByLabel(/Số điện thoại/).fill(randomPhone());
  await page.getByLabel(/PIN 6 số/).fill('000000');
  await page.getByRole('button', { name: 'Đăng nhập' }).last().click();
  await expect(page.getByText('SĐT hoặc PIN chưa đúng.')).toBeVisible();
});

test('vào tài khoản khi chưa đăng nhập thì chuyển sang đăng nhập', async ({ page }) => {
  await page.goto('/tai-khoan');
  await expect(page).toHaveURL(/dang-nhap/);
});
```

`tests/e2e/responsive.spec.ts`:

```ts
import { expect, test } from '@playwright/test';

const pages = ['/', '/dich-vu', '/mau-nail', '/dat-lich', '/tra-cuu', '/dang-nhap', '/lien-he'];

for (const path of pages) {
  test(`${path} không cuộn ngang và có đúng một tiêu đề h1`, async ({ page }) => {
    await page.goto(path);
    await page.waitForLoadState('networkidle');
    const overflow = await page.evaluate(() => document.documentElement.scrollWidth > window.innerWidth);
    expect(overflow, 'không được cuộn ngang').toBe(false);
    await expect(page.locator('h1')).toHaveCount(1);
  });
}

test('có liên kết bỏ qua tới nội dung chính, focus được bằng bàn phím', async ({ page }) => {
  await page.goto('/');
  await page.keyboard.press('Tab');
  await expect(page.getByRole('link', { name: 'Bỏ qua tới nội dung chính' })).toBeFocused();
});
```

Lưu ý: trang `/dang-nhap` trên điện thoại có hai tiêu đề `h1` (một ẩn bằng `md:hidden`, một ẩn trên điện thoại). Playwright đếm cả phần tử bị ẩn bằng CSS nên test sẽ báo 2. Sửa **giao diện** cho đúng một `h1`: trong `LoginPage.tsx` đổi khối bên trái (chỉ có từ `md`) thành thẻ `<h2>` thay vì `<h1>`.

- [ ] **Step 6: Chạy test trình duyệt**

Yêu cầu: `.env.local` trỏ tới project Supabase **thử nghiệm** đã `npm run db:seed`, và đã tắt "Confirm email" (Task 9).

Run: `npm run e2e`
Expected: PASS ở cả 3 cỡ màn hình. Test thất bại thì sửa bộ chọn trong `helpers.ts` hoặc sửa lỗi giao diện thật, rồi chạy lại. Có thể mở trình duyệt xem trực tiếp bằng skill `playwright-cli` để quan sát luồng khách trên điện thoại (375px), tablet (768px) và máy tính (1280px).

- [ ] **Step 7: Hoàn tất**

Run: `npm run check`
Expected: qua.

Chạy `/graphify` để cập nhật sơ đồ code, sau đó `/code-review` trên toàn bộ thay đổi của Kế hoạch 2. Sửa các lỗi nghiêm trọng được báo.

- [ ] **Step 8: Commit**

```bash
git add -A
git commit -m "test(e2e): luồng khách (đặt, tra cứu, hủy, đăng ký) ở 3 cỡ màn hình"
```

---

## Self-Review (đã chạy)

**Spec coverage (mục 9, 10, 11.1 đến 11.8):** 9.1-9.3 token màu, chữ, bo góc, bóng (Task 1); 9.4 điểm ngắt (ghi chú khác biệt `md`/`lg`); 9.5 bộ component (Task 1, 2, 4, 5, 7, 8); 9.6 trợ năng (nhãn, focus, 44px, `aria-live`, liên kết bỏ qua, `role="alert"`, bảng thay thế ở Kế hoạch 3); 9.7 hero 3D và dự phòng (Task 3); 9.8 ảnh `loading="lazy"` (GalleryTile); mục 10 các URL khách (Task 2 đến 10); 11.1 trang chủ (Task 3); 11.2 bảng giá (Task 4); 11.3 gallery (Task 5); 11.4 đặt lịch (Task 6, 7); 11.5 tra cứu (Task 8); 11.6 đăng nhập (Task 9); 11.7 tài khoản (Task 9); 11.8 liên hệ (Task 10); 7.2 mã lịch và `.ics` (Task 8); 7.3 hủy lịch (Task 8); 7.6 nhắc lịch mô phỏng phía khách (Task 8, 9); 14 kiểm thử và E2E (Task 11). Phần admin (nút "Gửi nhắc") thuộc Kế hoạch 3.

**Placeholder scan:** không có "TBD" bỏ ngỏ. Hai chỗ có hướng dẫn tinh chỉnh bằng lời ở Task 7 Step 5 (biểu thức `done`, thay `as never` bằng `TimeStep`) đã ghi rõ nội dung cần viết và phải làm ngay trong bước đó.

**Nhất quán kiểu và tên:** `BookingView`, `BookingReceipt`, `CouponCheck` định nghĩa ở Task 6 và dùng ở Task 7, 8, 9; `useAvailability`, `useCreateBooking`, `useLookupBooking`, `useCancelBooking`, `useValidateCoupon` khớp giữa Task 6 và các trang; `canCancelOnline`, `needsReminder` khớp giữa Task 8 và Task 9; `COPY.a11y.*` thêm ở Task 1 và dùng ở Task 2; `queryKeys.busy(day)` khớp ở Task 6 và Task 7. Trường `snake_case` khớp với JSON của RPC ở Kế hoạch 1.

**Review Focus:** 1 → Task 7 (`onError` đưa về bước Giờ, `invalidateQueries`) và Task 11 (test trùng giờ); 2 → Task 6 (test reducer bỏ giờ khi đổi dịch vụ, thợ, ngày); 3 → Task 6 (test `computeDayAvailability` ngày nghỉ, thợ nghỉ) và Task 7 (`SlotGrid`, `DateStrip`); 4 → Task 8 (test `canCancelOnline`, nút gọi tiệm); 5 → Task 3 (test `shouldUse3D`, đo FPS trong `Hero3D`).
