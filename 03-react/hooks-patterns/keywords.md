# 🔑 Senior Keywords — React Hooks Architecture & Concurrent Rendering

> Từ khóa chuyên sâu về Fiber Reconciler, Hooks Internals và Concurrent Mode.

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| React Architecture | `Fiber Node Structure` | Cấu trúc dữ liệu dạng Doubly Linked List biểu diễn cây thành phần React, hỗ trợ ngắt nghỉ công việc (Incremental Rendering) | `React Fiber architecture linked list workInProgress` |
| Hooks Internals | `Hook Dispatcher & Linked List` | Hooks được lưu dưới dạng danh sách liên kết đơn trên Fiber node, yêu cầu thứ tự gọi tuyệt đối không đổi | `React Hooks dispatcher linked list execution order` |
| Concurrent React | `useTransition & Priority Lanes` | Phân loại cập nhật UI thành Urgent (nhập liệu) và Non-urgent / Transition (lọc danh sách) để giữ ứng dụng responsive 60fps | `React useTransition priority lanes non-blocking` |
| State Synchronization | `useSyncExternalStore` | Hook đọc dữ liệu từ store ngoài React một cách an toàn, chống hiện tượng Tearing trong Concurrent Rendering | `React useSyncExternalStore tearing concurrent rendering` |
| Pitfall | `Stale Closure in Hooks` | Lỗi closure trong `useEffect`/`useCallback` giữ lại giá trị state/props cũ do thiếu dependency array chuẩn | `React stale closure useEffect dependency array` |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)

- **WorkInProgress Tree**: React duy trì 2 cây Fiber: Current Tree (đang hiển thị) và WorkInProgress Tree (đang tính toán ngầm), đổi chỗ sau khi commit.
- **useLayoutEffect vs useEffect**: LayoutEffect chạy đồng bộ ngay sau khi DOM mutation nhưng trước khi trình duyệt vẽ (Paint); Effect chạy bất đồng bộ sau Paint.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)

- **useDeferredValue Debouncing Alternative**: Hạ mức ưu tiên của value để React tự động hoãn render phần đắt đỏ khi user gõ phím nhanh.
- **Custom Hook Abstraction**: Đóng gói logic stateful phức tạp thành custom hook nguyên tử tái sử dụng.

### 3. Bẫy phỏng vấn & Case Studies (Interview Triggers)

- **Infinite Loop in useEffect**: Phân tích nguyên nhân truyền object/array inline làm thay đổi tham chiếu khiến Effect kích hoạt liên tục.
- **Why Rules of Hooks Exist**: Giải thích dựa trên kiến trúc danh sách liên kết đơn của Fiber giải thích tại sao không được gọi Hook trong vòng lặp/điều kiện.

