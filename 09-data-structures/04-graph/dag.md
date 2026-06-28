# 🔴 Directed Acyclic Graph (DAG) — Đồ thị có hướng không vòng

## 🧠 Định nghĩa dễ nhớ

> **DAG** giống như **thứ tự các môn học ở trường đại học**: môn Lập trình Cơ bản là điều kiện tiên quyết của môn Cấu trúc Dữ liệu, Cấu trúc Dữ liệu là điều kiện tiên quyết của Thuật toán Nâng cao. Mối liên kết đi theo một chiều (có hướng) và **không bao giờ có chuyện quay đầu lại tạo thành vòng lặp vô tận** (không vòng).

Nếu có vòng lặp (Ví dụ: A cần B, B cần C, C lại cần A) thì hệ thống sẽ bị khóa cứng (deadlock), không thể xác định thứ tự thực thi.

---

## ⏱️ Big-O Complexity

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Topological Sort | **O(V + E)** | Khám phá tất cả đỉnh và cạnh |
| Cycle Detection | **O(V + E)** | Bằng DFS hoặc thuật toán Kahn |
| Dependency Resolution | **O(V + E)** | Tương đương Topological Sort |

---

## 💻 Code JavaScript

### Cài đặt Topological Sort bằng thuật toán Kahn (BFS-based)

```javascript
class DAG {
  constructor() {
    this.adjacencyList = new Map();
    this.inDegree = new Map(); // Đếm số cạnh đi vào từng đỉnh
  }

  addVertex(v) {
    if (!this.adjacencyList.has(v)) {
      this.adjacencyList.set(v, new Set());
      this.inDegree.set(v, 0);
    }
  }

  // Thêm cạnh có hướng từ v1 -> v2 (v1 là điều kiện tiên quyết của v2)
  addEdge(v1, v2) {
    this.addVertex(v1);
    this.addVertex(v2);
    
    if (!this.adjacencyList.get(v1).has(v2)) {
      this.adjacencyList.get(v1).add(v2);
      this.inDegree.set(v2, this.inDegree.get(v2) + 1);
    }
  }

  // Thuật toán Kahn tìm Topological Sort & Kiểm tra chu trình
  topologicalSort() {
    const queue = [];
    const result = [];

    // Bước 1: Cho tất cả các đỉnh có in-degree = 0 vào queue (đỉnh không có phụ thuộc)
    for (const [vertex, degree] of this.inDegree.entries()) {
      if (degree === 0) queue.push(vertex);
    }

    // Bước 2: Duyệt BFS
    while (queue.length) {
      const u = queue.shift();
      result.push(u);

      for (const v of this.adjacencyList.get(u) || []) {
        // Giảm in-degree của các node con
        this.inDegree.set(v, this.inDegree.get(v) - 1);
        
        // Nếu không còn phụ thuộc nào (in-degree = 0) -> Push vào queue
        if (this.inDegree.get(v) === 0) {
          queue.push(v);
        }
      }
    }

    // Nếu số lượng phần tử đã duyệt nhỏ hơn tổng số đỉnh -> Có chu trình (vòng lặp)!
    if (result.length !== this.adjacencyList.size) {
      throw new Error("Đồ thị chứa chu trình (Cycle Detected)! Không thể sắp xếp.");
    }

    return result;
  }
}

// Demo
const dag = new DAG();
dag.addEdge('React', 'Redux');
dag.addEdge('JavaScript', 'React');
dag.addEdge('HTML', 'JavaScript');
dag.addEdge('CSS', 'React');

try {
  console.log("Thứ tự học tập tối ưu:", dag.topologicalSort());
  // ['HTML', 'CSS', 'JavaScript', 'React', 'Redux']
} catch (error) {
  console.error(error.message);
}
```

---

## 🔧 Trong Framework & Thực tế

### 1. Webpack / Vite / Yarn Workspaces — Module Dependency Resolution
```
Khi bạn viết:
// main.js
import { helper } from './utils.js';
import { Button } from './components.js';

Webpack/Vite phân tích mã nguồn và xây dựng một DAG (Dependency Graph). 
Mỗi file là 1 node, mỗi lệnh import tạo ra 1 cạnh có hướng (main.js -> utils.js).
Để đóng gói (bundle) code, Webpack chạy Topological Sort để build/inject các module con trước, 
tránh tình trạng file con chưa định nghĩa mà file cha đã gọi.
```

### 2. Yarn / NPM Monorepo Task Runner
```javascript
// Nếu dự án monorepo có cấu trúc dependencies như sau:
const projects = {
  'core-ui': [], // Không phụ thuộc ai
  'utils': [],   // Không phụ thuộc ai
  'dashboard': ['core-ui', 'utils'], // Phụ thuộc vào core-ui và utils
  'app': ['dashboard'] // Phụ thuộc vào dashboard
};

// Khi bạn chạy lệnh `yarn build`:
// Task runner sẽ sử dụng DAG để build `core-ui` và `utils` song song trước, 
// sau đó build `dashboard`, và cuối cùng mới build `app`.
```

### 3. Git Commit History
```
Lịch sử commit của Git thực chất là một DAG.
Mỗi commit trỏ ngược về commit cha (parent commit).
Khi bạn merge 2 nhánh, commit merge mới sẽ trỏ về 2 commit cha cùng lúc.
Vì lịch sử chỉ đi lùi về quá khứ, không có commit hiện tại trỏ ngược về commit tương lai -> DAG.
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### Phát hiện vòng lặp Import (Circular Dependencies) trong Webpack
Trong thực tế phát triển, việc import chéo (A import B, B lại import A) là cực kỳ nguy hiểm, dẫn đến biến bị `undefined` lúc runtime.

Các công cụ như `circular-dependency-plugin` chạy một thuật toán duyệt DAG để phát hiện chu trình trước khi build:

```javascript
// Minh họa thuật toán phát hiện chu trình sử dụng DFS 3 màu (White, Gray, Black)
function hasCircularDependency(graph) {
  const visited = new Map(); // node -> 'white' | 'gray' | 'black'
  
  // Khởi tạo tất cả node là WHITE (chưa thăm)
  for (const vertex of graph.adjacencyList.keys()) {
    visited.set(vertex, 'white');
  }

  function dfs(u) {
    visited.set(u, 'gray'); // Đang thăm (được cho vào call stack)

    for (const v of graph.adjacencyList.get(u) || []) {
      if (visited.get(v) === 'gray') {
        return true; // Gặp node xám trong call stack -> Có vòng lặp!
      }
      if (visited.get(v) === 'white') {
        if (dfs(v)) return true;
      }
    }

    visited.set(u, 'black'); // Đã thăm xong toàn bộ nhánh con
    return false;
  }

  for (const vertex of graph.adjacencyList.keys()) {
    if (visited.get(vertex) === 'white') {
      if (dfs(vertex)) return true;
    }
  }
  return false;
}
```

---

## ❓ Bẫy phỏng vấn

### Q1: Tại sao Topological Sort chỉ hoạt động trên DAG mà không hoạt động trên đồ thị thông thường?
```
Topological Sort tìm một thứ tự tuyến tính mà ở đó mọi cạnh u -> v thì u luôn đứng trước v.
Nếu đồ thị có chu trình (vòng lặp), ví dụ A -> B -> C -> A.
Để hoàn thành A thì cần B, cần B thì cần C, cần C lại cần A.
Không có điểm bắt đầu (in-degree luôn >= 1 cho mọi node trong vòng lặp) -> Bế tắc.
```

### Q2: Thuật toán Kahn hoạt động dựa trên cơ chế nào?
```
Dựa trên in-degree (bậc vào) của các đỉnh:
1. Đỉnh có in-degree = 0 không phụ thuộc vào bất kỳ đỉnh nào khác -> Có thể xử lý ngay.
2. Xử lý xong một đỉnh thì xóa bỏ các cạnh đi ra từ nó -> Giúp giảm in-degree của các đỉnh kề.
3. Lặp lại quá trình này. Nếu cuối cùng vẫn còn đỉnh chưa được duyệt thì chắc chắn đồ thị có chu trình.
```

---

## 🔗 Xem thêm
- [Graph (Đồ thị tổng quát)](./graph.md) — Khái niệm đồ thị cơ bản
- [Rope](../05-modern/rope.md) — Cấu trúc dữ liệu hiện đại cho xử lý chuỗi lớn
- [Webpack official documentation on dependency graph](https://webpack.js.org/concepts/dependency-graph/)
