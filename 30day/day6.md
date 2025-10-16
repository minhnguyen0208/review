# Day 06 — Queues & SQS

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- Queues & SQS
**Frontend (React)**
- **File input + progress**; hiển thị % và trạng thái.
- **(Tuỳ chọn)** Drag&Drop UX; retry on network errors.

---

## Backend (BE)
> **Mục tiêu phỏng vấn:** nắm quy trình upload an toàn, pre‑signed URL, và cách stream/serve file.
**Storage & Validation (Uploads) — trước khi xem code**
```php
// config/filesystems.php: set 's3' disk; .env has AWS creds
$request->validate([
  'file' => 'required|file|max:10240|mimes:jpg,png,pdf'
]);
$path = $request->file('file')->store('uploads', 's3'); // returns key
return response()->json(['key'=>$path], 201);
```
**Pre‑signed URL (server generates, client uploads)**
```php
// Controller
$key = 'uploads/'.Str::uuid().'.jpg';
$client = Storage::disk('s3')->getClient();
$cmd = $client->getCommand('PutObject', [
  'Bucket' => config('filesystems.disks.s3.bucket'),
  'Key'    => $key,
  'ACL'    => 'public-read',
  'ContentType' => 'image/jpeg',
]);
$requestUrl = (string)$client->createPresignedRequest($cmd, '+10 minutes')->getUri();
return ['key'=>$key, 'url'=>$requestUrl];
```
**Streamed Download (CSV/large files)**
```php
return Storage::disk('s3')->download($key); // or stream using StreamedResponse
```

---

## Frontend (FE)
> **Mục tiêu phỏng vấn:** UX upload có progress, không block, error rõ ràng.
**React — File input & progress**
```tsx
const [progress, setProgress] = useState(0);

async function uploadViaSignedUrl(file: File){
  const meta = await (await fetch('/api/uploads/sign?type='+file.type)).json();
  const xhr = new XMLHttpRequest();
  xhr.upload.onprogress = (e)=> setProgress(Math.round((e.loaded/e.total)*100));
  xhr.open('PUT', meta.url);
  xhr.setRequestHeader('Content-Type', file.type);
  xhr.send(file);
  // then save meta.key to your DB if needed
}
```
**Drag & Drop (tuỳ chọn)**
```tsx
function onDrop(e: React.DragEvent<HTMLDivElement>){
  e.preventDefault();
  const f = e.dataTransfer.files?.[0];
  if(f) uploadViaSignedUrl(f);
}
```

---

## English for Interview (skills only)
**English — Q&A (File uploads)**
- *How do you upload large files safely?* — Pre‑signed URLs, validate type/size server‑side, client shows progress; consider multipart for >5GB.
- *How do you secure access to files?* — Private bucket + time‑limited signed URLs; avoid exposing raw S3 keys; audit access.
- *How do you avoid blocking the API?* — Upload directly to storage; server only issues signed URL and records metadata.


---

### Activity
- BE 2–3h: implement 1 feature or refactor
- 2 LeetCode (E/M)
- write unit tests
- summarize learnings. | FE 2h: extend a React component/page
- add tests
- check a11y
- document decisions. | English 1–1.5h: shadow 15‑20 min
- mock interview Q&A
- record & review. | Queues/Cache: jobs, retries, backoff, RateLimiter
- measure throughput
- log timings and memory. | DevOps: containerize sample app
- healthcheck
- CI job
- readme with run/deploy steps.

### Practice Tasks
- Tạo API ký pre‑signed URL; lưu metadata file (key, content‑type, size).
- FE: chọn file → upload bằng signed URL → hiển thị progress; xử lý lỗi network.
- (Tuỳ chọn) thêm drag&drop; limit loại file & size theo quy định.

### Acceptance Criteria
- Upload thành công với progress hiển thị; BE validate đúng (mimes/size).
- Signed URL hết hạn đúng; không lộ secret; metadata được lưu.
- Download/serve hoạt động; lỗi có envelope chuẩn `{code,message}`.

### Docs & References
- Laravel Filesystem: https://laravel.com/docs/filesystem
- AWS S3 pre‑signed URLs (AWS SDK for PHP)
- React (Forms & Effects): https://react.dev/learn