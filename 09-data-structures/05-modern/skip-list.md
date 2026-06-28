# 🟣 Skip List — Danh sách liên kết đa tầng

## 🧠 Định nghĩa dễ nhớ

> **Skip List** giống như **tàu điện ngầm Hà Nội hoặc TP.HCM**: bạn có tuyến chạy suốt (chỉ dừng ở ga lớn) và tuyến chạy thường (dừng ở tất cả ga nhỏ). Muốn đi từ ga 1 đến ga 100, bạn bắt tuyến chạy suốt đến ga 90, sau đó xuống ga 90 và đổi sang tuyến chạy thường để đi tiếp đến ga 100. Nhờ các "tầng" ga tàu tốc hành này, bạn không cần dừng ở 99 ga trung gian.

```
Tầng 3: [1] ──────────────────────────► [30] ──────────────────────────► [80] ──► null
Tầng 2: [1] ─────────────► [15] ──────► [30] ─────────────► [57] ──────► [80] ──► null
Tầng 1: [1] ──► [9] ──► [15] ──► [22] ──► [30] ──► [45] ──► [57] ──► [72] ──► [80] ──► null
```

---

## ⏱️ Big-O Complexity

| Thao tác | Average | Worst | Ghi chú |
|----------|---------|-------|---------|
| Search | **O(log n)** | O(n) | Cực kỳ hiếm khi rơi vào worst case |
| Insert | **O(log n)** | O(n) | Dùng tung đồng xu ngẫu nhiên để chọn tầng |
| Delete | **O(log n)** | O(n) | Xóa node ở tất cả các tầng chứa nó |
| Space | **O(n)** | O(n log n) | Trung bình mỗi node tốn khoảng 1.33 đến 2 con trỏ |

---

## 💻 Code JavaScript

### Skip List Node & Class Implementation

```javascript
const MAX_LEVEL = 16;
const P = 0.5; // Xác suất tung đồng xu để quyết định lên tầng mới

class SkipNode {
  constructor(key, value, level) {
    this.key = key;
    this.value = value;
    // Mảng chứa con trỏ `next` ở từng tầng (0 đến level-1)
    this.forward = new Array(level).fill(null);
  }
}

class SkipList {
  constructor() {
    this.level = 1; // Tầng cao nhất hiện tại của list
    // Node đầu làm Sentinel, chứa mảng forward kích thước tối đa
    this.header = new SkipNode(-Infinity, null, MAX_LEVEL);
  }

  // Quyết định ngẫu nhiên xem node mới sẽ được đẩy lên mấy tầng
  #randomLevel() {
    let lvl = 1;
    while (Math.random() < P && lvl < MAX_LEVEL) {
      lvl++;
    }
    return lvl;
  }

  // SEARCH — O(log n)
  search(key) {
    let current = this.header;
    
    // Duyệt từ tầng cao nhất xuống tầng thấp nhất
    for (let i = this.level - 1; i >= 0; i--) {
      while (current.forward[i] && current.forward[i].key < key) {
        current = current.forward[i];
      }
    }
    
    current = current.forward[0]; // Xuống tầng 0 và đi tiếp 1 bước
    
    if (current && current.key === key) {
      return current.value;
    }
    return null;
  }

  // INSERT — O(log n)
  insert(key, value) {
    const update = new Array(MAX_LEVEL).fill(null);
    let current = this.header;

    // Bước 1: Tìm vị trí chèn và lưu lại các node đứng trước ở từng tầng
    for (let i = this.level - 1; i >= 0; i--) {
      while (current.forward[i] && current.forward[i].key < key) {
        current = current.forward[i];
      }
      update[i] = current;
    }

    current = current.forward[0];

    // Bước 2: Nếu key đã tồn tại -> Cập nhật value
    if (current && current.key === key) {
      current.value = value;
      return;
    }

    // Bước 3: Nếu key chưa tồn tại -> Tạo node mới
    const rLevel = this.#randomLevel();
    
    // Nếu tầng ngẫu nhiên cao hơn tầng hiện tại của list
    if (rLevel > this.level) {
      for (let i = this.level; i < rLevel; i++) {
        update[i] = this.header;
      }
      this.level = rLevel;
    }

    const newNode = new SkipNode(key, value, rLevel);
    
    // Cập nhật các liên kết next ở từng tầng
    for (let i = 0; i < rLevel; i++) {
      newNode.forward[i] = update[i].forward[i];
      update[i].forward[i] = newNode;
    }
  }
}

// Chạy thử nghiệm
const list = new SkipList();
list.insert(3, "Ba");
list.insert(6, "Sáu");
list.insert(7, "Bảy");
list.insert(9, "Chín");

console.log(list.search(7)); // "Bảy"
console.log(list.search(5)); // null (không tìm thấy)
```

---

## 🔧 Trong Framework & Thực tế

### 1. Redis Sorted Sets (zset)
```
Trong Redis, Sorted Set là cấu trúc lưu các phần tử đi kèm điểm số (score), tự động sắp xếp.
Khi số lượng phần tử nhỏ, Redis dùng ziplist để tiết kiệm bộ nhớ.
Khi số lượng phần tử lớn (> 128 phần tử hoặc kích thước > 64 bytes), Redis chuyển sang dùng kết hợp:
- Hash Map: Để truy cập O(1) từ member sang score.
- Skip List: Để thực hiện các phép truy vấn khoảng (Range queries) trong O(log n).

zrangebyscore myzset 10 20 (Tìm các phần tử có score từ 10 đến 20) chạy cực kỳ hiệu quả nhờ cấu trúc Skip List.
```

### 2. LevelDB / RocksDB (LSM-Tree MemTable)
```
LevelDB (database dạng Key-Value do Google phát triển) lưu trữ dữ liệu mới ghi vào RAM trong một cấu trúc gọi là MemTable.
MemTable yêu cầu ghi nhanh và luôn luôn giữ thứ tự để đổ xuống đĩa (flush to SSTable).
LevelDB lựa chọn Skip List làm cấu trúc chính cho MemTable thay vì Red-Black Tree.
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### Tại sao Redis chọn Skip List thay vì Cây tự cân bằng (AVL / Red-Black Tree)?
1. **Dễ code hơn**: Thuật toán tự cân bằng cây như xoay trái, xoay phải, đổi màu của Red-Black Tree rất phức tạp và dễ gặp lỗi. Skip List chỉ cần danh sách liên kết và hàm tung đồng xu ngẫu nhiên.
2. **Truy vấn khoảng (Range Query) tốt hơn**: Trong Skip List, tầng 0 là một Singly/Doubly Linked List hoàn chỉnh. Khi đã tìm thấy phần tử bắt đầu bằng O(log n), ta chỉ việc duyệt tuyến tính ga tiếp theo ở tầng 0 để lấy khoảng dữ liệu cực nhanh. Với cây, ta phải thực hiện inorder traversal phức tạp.
3. **Hỗ trợ Concurrency tốt hơn**: Việc ghi đè song song (Concurrent write) lên cây nhị phân cân bằng đòi hỏi khóa toàn bộ cây (lock) vì quá trình cân bằng lại cấu trúc có thể làm thay đổi từ root đến leaf. Skip List chỉ cập nhật liên kết của các node xung quanh điểm chèn -> Có thể lock cục bộ.

---

## ❓ Bẫy phỏng vấn

### Q1: Skip List hoạt động dựa trên cấu trúc dữ liệu nền tảng nào?
```
Dựa trên Singly Linked List (Danh sách liên kết đơn).
Để tăng tốc độ tìm kiếm từ O(n) lên O(log n), nó bổ sung thêm các "index pointers" (forward array) 
đóng vai trò ga tàu nhanh bỏ qua (skip) các node trung gian.
```

### Q2: Chuyện gì xảy ra nếu hàm tung đồng xu trong Skip List luôn trả về tầng 1 (hoặc tầng MAX)?
```
Nếu luôn trả về tầng 1: Skip List biến thành một Singly Linked List bình thường -> Tìm kiếm O(n).
Nếu luôn trả về tầng MAX: Mỗi node đều có con trỏ next ở tất cả các tầng -> Vừa tốn bộ nhớ vô ích, 
vừa mất tính phân tầng tốc hành -> Tìm kiếm vẫn bị chậm.
-> Hàm ngẫu nhiên phân phối hình học (Geometric Distribution) là chìa khóa đảm bảo hiệu năng O(log n).
```

---

## 🔗 Xem thêm
- [Linked List](../01-linear/linked-list.md) — Cơ sở của Skip List
- [Binary Search Tree](../03-tree/binary-search-tree.md) — So sánh tốc độ O(log n)
