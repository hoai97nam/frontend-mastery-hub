# 🟣 Disjoint-Set (Union-Find) — Tập hợp rời nhau

## 🧠 Định nghĩa dễ nhớ

> **Union-Find** giống như **hệ thống gia tộc**: ban đầu mỗi người là một gia tộc độc lập (chính họ là tộc trưởng). Khi hai người thuộc hai gia tộc khác nhau kết hôn, hai gia tộc **hợp nhất** (Union) và một tộc trưởng đại diện sẽ được chọn. Để kiểm tra xem hai người bất kỳ có thuộc cùng một gia tộc hay không, họ chỉ cần đi tìm tộc trưởng đại diện lớn nhất của mình (Find). Nếu tộc trưởng trùng nhau -> họ cùng một họ.

---

## ⏱️ Big-O Complexity

Với **n** phần tử, sử dụng cả 2 kỹ thuật tối ưu hóa là **Path Compression** (Nén đường đi) và **Union by Rank** (Hợp nhất theo cấp bậc):

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Find (Tìm đại diện) | **O(α(n))** ≈ O(1) | α là hàm Ackermann đảo ngược, tăng cực chậm |
| Union (Hợp nhất nhóm) | **O(α(n))** ≈ O(1) | Thực chất tương đương thời gian chạy Find |
| Space | **O(n)** | Cần 2 mảng lưu thông tin cha và rank |

*α(n) luôn < 5 cho tất cả các giá trị n thực tế trong vũ trụ (kể cả khi n là số nguyên tử trong vũ trụ).*

---

## 💻 Code JavaScript

### Cấu trúc Union-Find chuẩn chỉnh

```javascript
class UnionFind {
  constructor(size) {
    this.parent = new Array(size);
    this.rank = new Array(size).fill(0); // Độ sâu của cây để tối ưu khi gộp
    
    // Ban đầu mỗi phần tử tự trỏ vào chính nó (là tộc trưởng của chính mình)
    for (let i = 0; i < size; i++) {
      this.parent[i] = i;
    }
  }

  // FIND: Tìm tộc trưởng của phần tử i — O(α(n)) ✅
  find(i) {
    if (this.parent[i] === i) {
      return i;
    }
    // Path Compression: Gán trực tiếp cha của i bằng tộc trưởng lớn nhất
    // Giúp làm dẹt cây, các lần gọi Find sau chỉ mất O(1)
    this.parent[i] = this.find(this.parent[i]);
    return this.parent[i];
  }

  // UNION: Gộp nhóm chứa i và nhóm chứa j — O(α(n)) ✅
  union(i, j) {
    const rootI = this.find(i);
    const rootJ = this.find(j);

    if (rootI !== rootJ) {
      // Union by Rank: Cây thấp hơn trỏ vào cây cao hơn để tránh tăng chiều cao
      if (this.rank[rootI] < this.rank[rootJ]) {
        this.parent[rootI] = rootJ;
      } else if (this.rank[rootI] > this.rank[rootJ]) {
        this.parent[rootJ] = rootI;
      } else {
        this.parent[rootJ] = rootI;
        this.rank[rootI]++; // Chỉ tăng rank khi gộp 2 cây cùng chiều cao
      }
      return true; // Gộp thành công
    }
    return false; // Đã chung nhóm từ trước
  }

  // Kiểm tra 2 phần tử có chung nhóm không
  isConnected(i, j) {
    return this.find(i) === this.find(j);
  }
}

// Chạy thử nghiệm
const uf = new UnionFind(10); // 10 người độc lập: 0, 1, ..., 9
uf.union(1, 2);
uf.union(2, 5); // 1, 2, 5 chung một nhà
uf.union(3, 4);

console.log(uf.isConnected(1, 5)); // true
console.log(uf.isConnected(1, 3)); // false

uf.union(5, 3); // Gộp nhóm {1,2,5} với nhóm {3,4}
console.log(uf.isConnected(1, 4)); // true ✅
```

---

## 🔧 Trong Framework & Thực tế

### 1. Cycle Detection (Phát hiện chu trình) trong đồ thị cực nhanh
```javascript
// Phát hiện chu trình trên đồ thị vô hướng thay vì dùng DFS/BFS
function hasCycle(numVertices, edges) {
  const uf = new UnionFind(numVertices);
  
  for (const [u, v] of edges) {
    // Nếu u và v đã cùng kết nối từ trước mà nay lại có thêm cạnh trực tiếp nối u-v
    // -> Xuất hiện vòng lặp!
    if (uf.isConnected(u, v)) {
      return true;
    }
    uf.union(u, v);
  }
  return false;
}

const edges = [[0, 1], [1, 2], [2, 0]]; // Tam giác kết nối
console.log("Đồ thị có chu trình?", hasCycle(3, edges)); // true
```

### 2. React Fiber Reconciliation — Dynamic Connectivity
```
Khi React chạy cơ chế so khớp (Reconciliation) danh sách phần tử con,
mỗi phần tử có thể di chuyển, bị xóa hoặc chèn mới.
Về mặt lý thuyết, việc track các kết nối lỏng lẻo và xác định xem component cũ có được 
reuse ở cây mới hay không sử dụng tư tưởng nhóm tập hợp tương tự Union-Find.
```

### 3. Thuật toán Kruskal tìm cây khung nhỏ nhất (MST)
```
Dùng trong việc quy hoạch thiết kế mạng cáp quang, mạng lưới giao thông sao cho 
chi phí nối các điểm là rẻ nhất nhưng vẫn đảm bảo tất cả mọi nơi đều được thông suốt.
Thuật toán Kruskal phân loại các cạnh theo trọng số, dùng Union-Find để kiểm tra 
kết nối và gom nhóm các điểm không tạo chu trình.
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### Sức mạnh của Path Compression (Nén đường đi)
Nếu không dùng Path Compression, việc Union liên tục có thể tạo ra một cây thẳng đứng (như Linked List). Lúc đó Find sẽ mất `O(n)`.

Khi có Path Compression:
```
Trước nén:  (A) ◄── (B) ◄── (C) ◄── (D)
Sau Find(D): (A) ◄── (B), (A) ◄── (C), (A) ◄── (D)  (Tất cả trỏ trực tiếp về gốc A)
```
Chỉ sau 1 lần tìm kiếm, cấu trúc cây lập tức bị bẻ phẳng, giúp toàn bộ các thao tác sau đó diễn ra tức thì.

---

## ❓ Bẫy phỏng vấn

### Q1: Cấp bậc Rank trong Union by Rank là gì? Có phải là chiều cao thực tế của cây?
```
Ban đầu Rank đại diện cho chiều cao của cây. 
Tuy nhiên, do kỹ thuật Path Compression làm bẻ phẳng cây liên tục lúc Find, 
Rank không còn là chiều cao thực tế nữa mà là một "giới hạn trên" (Upper Bound) 
của chiều cao cây. 
Đó là lý do người ta gọi nó là Rank chứ không đặt tên biến là Height.
```

### Q2: Tại sao Union-Find hiệu quả hơn DFS trong việc đếm số thành phần liên thông của đồ thị động (Dynamic Connectivity)?
```
DFS/BFS bắt buộc đồ thị phải tĩnh. Mỗi khi có thêm 1 cạnh mới được thêm vào, 
ta phải chạy lại DFS từ đầu: O(V + E).
Union-Find xử lý trực tuyến (Online): Khi có cạnh mới nối u-v, ta chỉ việc gọi `uf.union(u, v)` 
mất O(α(n)) ≈ O(1) để cập nhật nhóm liên thông ngay tại chỗ mà không cần quét lại cả đồ thị.
```

---

## 🔗 Xem thêm
- [Graph (Đồ thị)](./graph.md) — Tổng quan về đồ thị liên thông
- [Bloom Filter](../02-hash-based/bloom-filter.md) — Một cấu trúc xác suất khác
