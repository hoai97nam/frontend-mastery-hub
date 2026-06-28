# 🔵 Linear Data Structures — Tuyến tính

> **Tuyến tính** = dữ liệu xếp thành **một hàng dọc**, mỗi phần tử có đúng một người đứng trước và một người đứng sau (trừ đầu/cuối).

Đây là nhóm **nền tảng nhất** — mọi cấu trúc phức tạp hơn đều xây dựng trên những thứ này.

---

## Các cấu trúc trong nhóm

| DS | Truy cập | Tìm kiếm | Chèn | Xóa | Khi nào dùng |
|----|----------|----------|------|-----|---------------|
| [Array](./array.md) | O(1) | O(n) | O(n) | O(n) | Dữ liệu có index, random access |
| [Linked List](./linked-list.md) | O(n) | O(n) | O(1)* | O(1)* | Chèn/xóa nhiều, không cần random access |
| [Stack](./stack.md) | O(n) | O(n) | O(1) | O(1) | Undo/redo, call stack, parsing |
| [Queue](./queue.md) | O(n) | O(n) | O(1) | O(1) | Task scheduling, BFS, event loop |
| [Deque](./deque.md) | O(n) | O(n) | O(1) | O(1) | Sliding window, cả hai đầu |

*ở đầu hoặc nút đã biết

---

## Chọn cái nào?

```
Cần truy cập theo vị trí (index)?     → Array
Chèn/xóa nhiều ở giữa?               → Linked List
Cần LIFO (cái cuối vào, ra trước)?   → Stack
Cần FIFO (cái đầu vào, ra trước)?    → Queue
Cần cả hai đầu linh hoạt?            → Deque
```

---

## Kết nối thực tế

- **JavaScript**: `Array`, `Array` as Stack (`push/pop`), `Array` as Queue (`push/shift`)
- **React**: Component tree traversal dùng Stack (DFS)
- **Node.js**: Event Loop dùng Queue (Callback Queue, Microtask Queue)
- **Browser**: History API (back/forward) dùng 2 Stacks
