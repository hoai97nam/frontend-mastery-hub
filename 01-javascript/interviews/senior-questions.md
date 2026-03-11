# Câu Hỏi Phỏng Vấn JavaScript - Cấp Độ Senior

> 5 câu hỏi "hạng nặng" kèm gợi ý trả lời dành cho Senior JavaScript Developer.

---

## Mục Lục

1. [Event Loop — Microtask vs. Macrotask](#1-event-loop--microtask-vs-macrotask)
2. [Closures và Memory Leak](#2-closures-và-memory-leak)
3. [Prototypal Inheritance vs. Class-based Inheritance](#3-prototypal-inheritance-vs-class-based-inheritance)
4. [Deep Clone Object Phức Tạp](#4-deep-clone-object-phức-tạp)
5. [Map/Set vs. WeakMap/WeakSet](#5-mapset-vs-weakmapweakset)

---

## 1. Event Loop — Microtask vs. Macrotask

**🎯 Mục đích:** Kiểm tra khả năng dự đoán luồng chạy của code không đồng bộ và tối ưu UI.

**💡 Gợi ý trả lời:**

Senior cần giải thích được rằng **Microtasks luôn được thực hiện hết sạch** trước khi Event Loop chuyển sang Macrotask tiếp theo hoặc thực hiện Render. Một vòng lặp Microtask vô tận (như Promise đệ quy) sẽ làm **treo UI hoàn toàn**.

---

## 2. Closures và Memory Leak

**🎯 Mục đích:** Kiểm tra kiến thức về quản lý bộ nhớ.

**💡 Gợi ý trả lời:**

Có, nếu closure giữ tham chiếu đến các object lớn trong khi bản thân closure đó không được giải phóng (ví dụ: gán vào một biến global hoặc listener không được `remove`).

**Cách phòng tránh:**
- Gán `null` cho biến sau khi dùng xong.
- Đảm bảo lifecycle của listener rõ ràng (luôn `removeEventListener` khi không cần).

---

## 3. Prototypal Inheritance vs. Class-based Inheritance

**🎯 Mục đích:** Kiểm tra sự hiểu biết về bản chất ngôn ngữ.

**💡 Gợi ý trả lời:**

JS là ngôn ngữ **dynamic**. Prototype cho phép các object chia sẻ phương thức trực tiếp mà không cần cấu trúc phân cấp cứng nhắc. Điều này giúp:
- **Tiết kiệm bộ nhớ** — các instance dùng chung phương thức qua prototype chain.
- **Linh hoạt hơn** trong việc mở rộng object lúc runtime.

---

## 4. Deep Clone Object Phức Tạp

**🎯 Mục đích:** Kiểm tra kinh nghiệm thực tế với dữ liệu (Date, RegExp, Function).

**💡 Gợi ý trả lời:**

Đừng chỉ dùng `JSON.parse(JSON.stringify())` — nó **mất hàm** và **lỗi với `Date`**. Senior nên đề cập đến:

| Phương pháp | Ưu điểm | Nhược điểm |
|---|---|---|
| `JSON.parse(JSON.stringify())` | Đơn giản | Mất `Function`, `Date`, `undefined` |
| `structuredClone()` | Native, hỗ trợ `Date`, `Map`, `Set` | Không clone `Function` |
| `lodash.cloneDeep()` | Xử lý hầu hết trường hợp | Cần cài thư viện |
| Hàm đệ quy tự viết | Kiểm soát hoàn toàn | Tốn công |

---

## 5. Map/Set vs. WeakMap/WeakSet

**🎯 Mục đích:** Kiểm tra kỹ năng tối ưu bộ nhớ cao cấp.

**💡 Gợi ý trả lời:**

Điểm mấu chốt là **`WeakMap` không ngăn cản Garbage Collector** dọn dẹp các key (object). Nó cực kỳ hữu ích khi bạn muốn gắn metadata vào một object mà không muốn "giữ chân" object đó trong bộ nhớ.

**Ví dụ thực tế:** Lưu cache dữ liệu cho DOM element — khi element bị xóa khỏi DOM, entry trong `WeakMap` sẽ tự động được dọn dẹp.