# Day 08 — Testing & Quality — PHPUnit (Feature/Unit, Fakes) | React Testing Library + MSW

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- **Feature vs Unit tests**: khi nào viết; cấu trúc thư mục test.
- **Fakes**: `Mail::fake()`, `Notification::fake()`, `Bus::fake()`; assert sent/dispatched.
- **Factories & Seeders**: chuẩn bị dữ liệu nhanh, deterministic.
- **CI basics**: chạy test headless, .env.testing, sqlite in-memory hoặc mysql service.
**Frontend (React)**
- **React Testing Library (RTL)**: test theo góc nhìn user (role, text).
- **MSW** (Mock Service Worker): fake API layer để test flows.
- **Accessibility assertions**: `getByRole`, `aria-*`, error `role=\"alert\"`.

---

## Backend (BE)
> **Mục tiêu phỏng vấn:** phân biệt Feature/Unit, sử dụng Fakes đúng cách, và chuẩn bị dữ liệu bằng Factory.
**Factories & Seed**
```php
// database/factories/PostFactory.php
public function definition(){
  return ['title'=>$this->faker->sentence, 'body'=>$this->faker->paragraph, 'author_id'=>User::factory()];
}
// database/seeders/DatabaseSeeder.php
User::factory()->count(5)->create();
Post::factory()->count(20)->create();
```
**Feature Test — API contract**
```php
public function test_list_posts_returns_data(){
  Post::factory()->count(3)->create();
  $res = $this->getJson('/api/posts');
  $res->assertOk()->assertJsonStructure(['data'=>[['id','title']]]);
}
```
**Fakes — Mail/Notification/Bus**
```php
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Notification;
use Illuminate\Support\Facades\Bus;

Mail::fake(); Notification::fake(); Bus::fake();

// hành động phát mail/notification/job
// ...

// assert
Mail::assertNothingSent(); // hoặc Mail::assertSent(Mailable::class);
Notification::assertSentTo($user, ResetPassword::class);
Bus::assertDispatched(SendReportJob::class);
```

---

## Frontend (FE)
> **Mục tiêu phỏng vấn:** viết test dựa vào hành vi người dùng; mock API bằng MSW, kiểm tra a11y.
**React Testing Library — basic flow**
```tsx
import { render, screen, waitFor } from '@testing-library/react';
import App from './App';

test('renders list with loading state', async () => {
  render(<App/>);
  expect(screen.getByText(/Loading/i)).toBeInTheDocument();
  await waitFor(()=> expect(screen.getByRole('list')).toBeInTheDocument());
});
```
**MSW — mock API layer**
```ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/posts', () => HttpResponse.json({ data:[{id:1,title:'Hello'}] })),
  http.post('/api/posts', async ({ request }) => {
    const body = await request.json();
    if(!body.title) return HttpResponse.json({ code:422, message:'Validation', errors:{title:['required']} }, { status:422 });
    return HttpResponse.json({ id: 2, ...body }, { status:201 });
  }),
];
```
**Accessibility assertions**
```tsx
expect(screen.getByRole('alert')).toHaveTextContent(/required/i);
expect(screen.getByRole('button', {name:/submit/i})).toBeEnabled();
```

---

## English for Interview (skills only)
**Q&A (Testing)**
- *What do you test at unit vs feature levels?* — Unit for pure logic/small scope; Feature for API contracts, integration of layers.
- *How do you handle external side effects in tests?* — Use Laravel fakes for mail/queue/notifications; assert dispatched/sent.
- *How do you keep tests reliable and fast?* — Seed minimal data via factories, isolate time/ID generation, run in parallel where possible.


---

### Activity
- Viết 2 Feature tests cho `/posts` (index/store).
- Thêm 1 Unit test cho helper/Service nhỏ.
- Thiết lập MSW + 2 handlers cho list & create; viết 1 test RTL.

### Practice Tasks
- Laravel: tạo factories + 2 Feature tests; sử dụng Mail/Notification/Bus fakes.
- React: cấu hình MSW + RTL, viết test cho flow loading → success → error.
- Viết README ngắn mô tả cách chạy test (BE/FE).

### Acceptance Criteria
- Test BE/FE chạy pass xanh; không phụ thuộc mạng ngoài.
- Fakes/handlers che phủ side effects; assertions meaningful (không brittle).
- README có lệnh chạy test và ghi chú môi trường.

### Docs & References
- Laravel Testing: https://laravel.com/docs/testing
- Laravel Mail/Notification/Queue Fakes: https://laravel.com/docs/testing#available-assertions
- React Testing Library: https://testing-library.com/docs/react-testing-library/intro/
- MSW: https://mswjs.io/docs