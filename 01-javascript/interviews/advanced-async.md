# Senior JS: Advanced Asynchronous Patterns

## 1. Tổng quan
Xử lý bất đồng bộ là "đặc sản" của JS. Senior cần làm chủ các luồng dữ liệu phức tạp, tránh race conditions và tối ưu hóa tài nguyên.

## 2. Câu hỏi "Sát thủ"

### Q1: Làm thế nào để giới hạn số lượng request đồng thời (Concurrency Limit)?
**Trả lời:**
Không nên dùng `Promise.all()` cho 1000 requests cùng lúc vì sẽ làm nghẽn network. Ta cần dùng một hàng đợi (Queue) hoặc Semaphore.
Dùng một hàm wrapper để chỉ cho phép tối đa N promises được thực thi tại một thời điểm.

## 3. Dấu hiệu Senior (The Senior Signal)
- **Junior:** Chỉ biết `async/await` và `then/catch`.
- **Senior:** Biết dùng **AbortController** để hủy request khi component unmount, dùng **Async Iterators** để xử lý stream dữ liệu lớn, và hiểu sâu về **Microtask Queue starvation**.

## 4. Case Study
**Bài toán:** Upload 100 files lớn lên server.
**Giải pháp:** Sử dụng **Worker Threads** để tính toán mã hash (SHA-256) của file ở background, tránh làm treo Main Thread (UI).

## 5. Code Lab: AbortController
```javascript
const controller = new AbortController();
const signal = controller.signal;

setTimeout(() => controller.abort(), 5000); // Hủy sau 5s

fetch('/long-api', { signal })
  .then(res => res.json())
  .catch(err => {
    if (err.name === 'AbortError') {
      console.log('Fetch aborted by user/timeout');
    }
  });
```
