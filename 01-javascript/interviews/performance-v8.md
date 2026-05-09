# Senior JS: Performance & V8 Internals

## 1. Tổng quan
Kiến thức về cách V8 Engine (hoặc SpiderMonkey, JavaScriptCore) hoạt động giúp Senior tối ưu code từ mức thấp, tránh các lỗi làm giảm hiệu năng hệ thống một cách âm thầm.

## 2. Câu hỏi "Sát thủ"

### Q1: Giải thích cơ chế Garbage Collection trong V8 và sự khác biệt giữa Scavenger và Mark-Sweep.
**Trả lời:**
V8 chia bộ nhớ (Heap) thành 2 vùng chính: **New Space** và **Old Space**.
- **Scavenger (Minor GC):** Hoạt động trên New Space. Dùng thuật toán Cheney để copy các object còn sống từ vùng "From" sang "To". Rất nhanh nhưng tốn bộ nhớ gấp đôi.
- **Mark-Sweep/Mark-Compact (Major GC):** Hoạt động trên Old Space. Dùng khi object sống sót qua vài chu kỳ Scavenger. Thuật toán này "đánh dấu" các object có thể truy cập từ root, sau đó "quét" để giải phóng bộ nhớ và "nén" để tránh phân mảnh.

## 3. Dấu hiệu Senior (The Senior Signal)
- **Junior:** "JS tự dọn rác, mình không cần lo."
- **Senior:** Đề cập đến **V8 Orinoco** (Parallel/Incremental/Concurrent marking) giúp giảm "Stop-the-world" time, và biết cách dùng Chrome DevTools Memory Tab để tìm **Retainers Chain**.

## 4. Case Study
**Bài toán:** Một ứng dụng Dashboard real-time bị chậm dần sau 2 tiếng sử dụng.
**Giải pháp:** Senior sử dụng Allocation Timeline để phát hiện các DOM nodes bị "rò rỉ" do chưa remove event listener trong các component bị destroy.

## 5. Code Lab: Tránh Deoptimization
```javascript
// BAD: Thay đổi shape của object làm mất Hidden Class
function Point(x, y) {
  this.x = x;
  this.y = y;
}
const p1 = new Point(1, 2);
const p2 = new Point(3, 4);
p2.z = 5; // V8 phải tạo Hidden Class mới cho p2 -> Deoptimization!

// GOOD: Luôn khởi tạo đầy đủ thuộc tính trong constructor
function Point(x, y, z = null) {
  this.x = x;
  this.y = y;
  this.z = z;
}
```
