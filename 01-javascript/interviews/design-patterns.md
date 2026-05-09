# Senior JS: Design Patterns & Architecture

## 1. Tổng quan
Senior không chỉ viết code chạy được, mà phải viết code có thể bảo trì và mở rộng. Design Patterns là "ngôn ngữ chung" để giải quyết các vấn đề kiến trúc lặp lại.

## 2. Câu hỏi "Sát thủ"

### Q1: Khi nào bạn chọn Composition thay vì Inheritance? Cho ví dụ thực tế.
**Trả lời:**
"Composition over Inheritance" giúp tránh cấu trúc phân cấp cứng nhắc.
- **Inheritance:** Phù hợp khi có quan hệ "is-a" chặt chẽ.
- **Composition:** Phù hợp khi muốn lắp ghép tính năng linh hoạt ("has-a" hoặc "can-do").
**Ví dụ:** Thay vì class `SmartPhone` kế thừa từ `Camera` và `Phone`, ta nên compose `CameraModule` và `PhoneModule` vào `SmartPhone`.

## 3. Dấu hiệu Senior (The Senior Signal)
- **Junior:** Áp dụng pattern một cách máy móc (over-engineering).
- **Senior:** Biết khi nào **KHÔNG** nên dùng pattern. Giải thích được sự khác biệt giữa **Observer Pattern** (chặt chẽ) và **Pub/Sub Pattern** (loose coupling qua trung gian).

## 4. Case Study
**Bài toán:** Hệ thống Plugin cho một trình soạn thảo văn bản.
**Giải pháp:** Sử dụng **Strategy Pattern** để cho phép người dùng thay đổi thuật toán format (Markdown, HTML, BBCode) tại runtime mà không sửa code lõi.

## 5. Code Lab: Singleton Pattern (Thread-safe-ish in JS)
```javascript
const Database = (function () {
  let instance;
  function createInstance() {
    return new Object("Database Instance");
  }
  return {
    getInstance: function () {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    },
  };
})();
```
