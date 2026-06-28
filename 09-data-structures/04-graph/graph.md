# 🔴 Graph — Đồ thị tổng quát

## 🧠 Định nghĩa dễ nhớ

> **Graph** giống như **mạng lưới bạn bè trên Facebook**: mỗi tài khoản là một đỉnh (node), và mối quan hệ bạn bè giữa hai người là một cạnh (edge). Để biết bạn và một người lạ có bao nhiêu "bạn chung", chúng ta duyệt qua mạng lưới này bằng BFS/DFS.

Đồ thị có 2 dạng kết nối chính:
- **Undirected (Vô hướng)**: Nếu A là bạn của B, thì B cũng là bạn của A.
- **Directed (Có hướng)**: A follow B trên Twitter/Instagram, nhưng B chưa chắc follow lại A.

---

## ⏱️ Big-O Complexity

Biểu diễn bằng **Adjacency List**:

| Thao tác | Complexity | Ghi chú |
|----------|-----------|---------|
| Add Vertex | **O(1)** | Thêm key mới vào Map/Object |
| Add Edge | **O(1)** | Push phần tử mới vào mảng kề |
| Remove Edge | **O(E/V)** | Phải tìm và xóa phần tử trong mảng kề |
| Remove Vertex | **O(V + E)** | Xóa key và mọi liên kết liên quan |
| Query (Check Edge) | **O(V)** | Trường hợp tệ nhất là duyệt hết mảng kề |
| Space | **O(V + E)** | Bộ nhớ tối ưu cho đa số đồ thị thực tế |

---

## 💻 Code JavaScript

### Graph class sử dụng Adjacency List

```javascript
class Graph {
  constructor() {
    this.adjacencyList = new Map();
  }

  // Thêm đỉnh mới
  addVertex(vertex) {
    if (!this.adjacencyList.has(vertex)) {
      this.adjacencyList.set(vertex, new Set());
    }
  }

  // Thêm cạnh (Vô hướng)
  addEdge(v1, v2) {
    this.addVertex(v1);
    this.addVertex(v2);
    this.adjacencyList.get(v1).add(v2);
    this.adjacencyList.get(v2).add(v1);
  }

  // Xóa cạnh
  removeEdge(v1, v2) {
    if (this.adjacencyList.has(v1)) this.adjacencyList.get(v1).delete(v2);
    if (this.adjacencyList.has(v2)) this.adjacencyList.get(v2).delete(v1);
  }

  // Xóa đỉnh
  removeVertex(vertex) {
    if (!this.adjacencyList.has(vertex)) return;
    
    // Xóa mọi cạnh liên quan trước
    for (const neighbor of this.adjacencyList.get(vertex)) {
      this.adjacencyList.get(neighbor).delete(vertex);
    }
    
    // Xóa đỉnh khỏi map
    this.adjacencyList.delete(vertex);
  }

  // DFS đệ quy
  dfsRecursive(start) {
    const result = [];
    const visited = new Set();
    const list = this.adjacencyList;

    function traverse(vertex) {
      if (!vertex) return;
      visited.add(vertex);
      result.push(vertex);
      
      for (const neighbor of list.get(vertex) || []) {
        if (!visited.has(neighbor)) {
          traverse(neighbor);
        }
      }
    }

    traverse(start);
    return result;
  }

  // BFS sử dụng Queue
  bfs(start) {
    const queue = [start];
    const result = [];
    const visited = new Set([start]);

    while (queue.length) {
      const current = queue.shift();
      result.push(current);

      for (const neighbor of this.adjacencyList.get(current) || []) {
        if (!visited.has(neighbor)) {
          visited.add(neighbor);
          queue.push(neighbor);
        }
      }
    }
    return result;
  }
}

// Chạy thử nghiệm
const g = new Graph();
g.addEdge('A', 'B');
g.addEdge('A', 'C');
g.addEdge('B', 'D');
g.addEdge('C', 'E');
g.addEdge('D', 'E');
g.addEdge('D', 'F');

console.log("BFS:", g.bfs('A')); // ['A', 'B', 'C', 'D', 'E', 'F']
console.log("DFS:", g.dfsRecursive('A')); // ['A', 'B', 'D', 'E', 'C', 'F']
```

---

## 🔧 Trong Framework & Thực tế

### 1. Facebook Friend Recommendation (BFS)
```javascript
// BFS tìm bạn của bạn (2nd-degree connections)
function recommendFriends(graph, userId) {
  const recommendations = new Map();
  const visited = new Set([userId]);
  const queue = [[userId, 0]]; // [currentId, distance]

  // Cho trước direct friends của userId
  const directFriends = graph.adjacencyList.get(userId) || new Set();

  while (queue.length) {
    const [current, dist] = queue.shift();

    if (dist > 2) break; // Chỉ gợi ý bạn của bạn (Level 2)

    for (const neighbor of graph.adjacencyList.get(current) || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push([neighbor, dist + 1]);

        // Nếu là bạn level 2 và không phải là bạn trực tiếp
        if (dist === 1 && !directFriends.has(neighbor)) {
          recommendations.set(neighbor, (recommendations.get(neighbor) || 0) + 1);
        }
      }
    }
  }
  // Trả về danh sách xếp hạng theo số bạn chung nhiều nhất
  return [...recommendations.entries()].sort((a, b) => b[1] - a[1]).map(e => e[0]);
}
```

### 2. URL Router / Page Navigation Graph
```javascript
// Single Page Applications (React, Vue, Angular routers)
// Hệ thống định tuyến thực tế là một Graph có hướng.
// Mỗi route là một node, việc navigate từ trang này sang trang kia tạo ra các cạnh.
const routes = {
  '/': ['/login', '/products'],
  '/login': ['/dashboard', '/reset-password'],
  '/products': ['/cart', '/login'],
  '/cart': ['/checkout'],
  '/checkout': ['/dashboard']
};
// Router có thể kiểm tra xem từ trang hiện tại user có đi được đến trang đích không bằng BFS/DFS.
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### 1. Cycle Detection (Phát hiện chu trình)
Trong đồ thị vô hướng, nếu một đỉnh kề của đỉnh hiện tại đã được visit và **không phải là node cha trực tiếp** của đỉnh hiện tại, thì đồ thị chứa vòng lặp (cycle).

```javascript
function hasCycleUndirected(graph) {
  const visited = new Set();
  
  function dfs(vertex, parent) {
    visited.add(vertex);
    
    for (const neighbor of graph.adjacencyList.get(vertex) || []) {
      if (!visited.has(neighbor)) {
        if (dfs(neighbor, vertex)) return true;
      } else if (neighbor !== parent) {
        return true; // Đã ghé thăm và không phải cha -> Có vòng!
      }
    }
    return false;
  }

  for (const vertex of graph.adjacencyList.keys()) {
    if (!visited.has(vertex)) {
      if (dfs(vertex, null)) return true;
    }
  }
  return false;
}
```

### 2. Biểu diễn Graph bằng mảng dẹt (Flat Array) để tối ưu hóa bộ nhớ
Trong WebGL hoặc Web Assembly, truyền Map hoặc JSON Node phức tạp vào shader/memory buffer rất chậm. Thay vào đó, người ta flatten đồ thị thành 2 mảng phẳng:
- `offsets`: `[0, 2, 4, 6]` (vị trí bắt đầu của danh sách kề của đỉnh i)
- `edges`: `[1, 2, 0, 3, 0, 3, 1, 2]` (danh sách các đỉnh đích)

---

## ❓ Bẫy phỏng vấn

### Q1: Phân biệt BFS và DFS khi tìm đường đi ngắn nhất?
```
BFS: Tìm đường đi ngắn nhất trên đồ thị KHÔNG CÓ TRỌNG SỐ (mọi cạnh bằng nhau).
    Bởi vì BFS mở rộng theo từng layer. Layer k chứa các node có khoảng cách k bước từ điểm xuất phát.
DFS: KHÔNG ĐẢM BẢO đường đi ngắn nhất. Nó đi sâu nhất có thể, có thể tìm thấy đích ở độ sâu 100 
    trong khi có đường đi ngắn hơn ở độ sâu 2.
    Để tìm đường đi ngắn nhất trên đồ thị CÓ TRỌNG SỐ, ta dùng Dijkstra (dùng Heap).
```

### Q2: Gặp lỗi Stack Overflow khi chạy DFS trên đồ thị lớn?
```javascript
// Nếu đồ thị có dạng 1 đường thẳng dài (Skewed) với 100,000 node, 
// việc đệ quy DFS sẽ làm tràn JavaScript Call Stack.
// Giải pháp: Chuyển DFS từ đệ quy sang sử dụng vòng lặp với một explicit Stack lưu trong heap.
function dfsIterative(graph, start) {
  const stack = [start];
  const visited = new Set([start]);
  const result = [];

  while (stack.length) {
    const vertex = stack.pop();
    result.push(vertex);

    for (const neighbor of graph.adjacencyList.get(vertex) || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        stack.push(neighbor);
      }
    }
  }
  return result;
}
```

---

## 🔗 Xem thêm
- [DAG (Đồ thị không vòng có hướng)](./dag.md) — Dùng cho module dependency resolution
- [Disjoint Set (Union-Find)](../05-modern/disjoint-set.md) — Dùng để kiểm tra liên thông đồ thị nhanh nhất
- [Trie](../03-tree/trie.md) — Một dạng đồ thị có hướng đặc biệt biểu diễn tiền tố từ vựng
