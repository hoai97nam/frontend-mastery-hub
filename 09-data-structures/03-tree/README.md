# 🌳 Tree Data Structures — Cấu trúc Cây

> **Cây** = dữ liệu có **quan hệ phân cấp** (cha-con). Hiệu quả hơn Linear khi cần **tìm kiếm có tổ chức**.

---

## Các cấu trúc trong nhóm

| DS | Tìm kiếm | Chèn | Xóa | Khi nào dùng |
|----|----------|------|-----|--------------|
| [Binary Tree](./binary-tree.md) | O(n) | O(n) | O(n) | Biểu diễn cấu trúc phân cấp |
| [BST](./binary-search-tree.md) | O(log n)* | O(log n)* | O(log n)* | Tìm kiếm có thứ tự |
| [Heap](./heap.md) | O(n) | O(log n) | O(log n) | Priority queue, kth element |
| [Trie](./trie.md) | O(L) | O(L) | O(L) | Prefix search, autocomplete |
| [Segment Tree](./segment-tree.md) | O(log n) | O(log n) | O(log n) | Range queries |
| [Red-Black Tree](./red-black-tree.md) | O(log n) | O(log n) | O(log n) | Self-balancing BST |

*BST: O(log n) average, O(n) worst (unbalanced)
*L = length of key (Trie)

---

## Chọn cái nào?

```
Tìm min/max nhanh?              → Heap
Prefix search / autocomplete?   → Trie
Range sum/min/max queries?      → Segment Tree
Ordered map/set (guaranteed)?   → Red-Black Tree
DOM/file system structure?      → Binary Tree (generic)
Đơn giản, không cần balance?   → BST (nhưng cẩn thận worst case)
```

---

## Cấu trúc chung

```javascript
class TreeNode {
  constructor(val) {
    this.val = val;
    this.left = null;
    this.right = null;
  }
}

// 4 cách traversal
function inorder(node)   { if(!node) return; inorder(node.left); visit(node); inorder(node.right); }
function preorder(node)  { if(!node) return; visit(node); preorder(node.left); preorder(node.right); }
function postorder(node) { if(!node) return; postorder(node.left); postorder(node.right); visit(node); }
// BFS = level-order → dùng Queue
```

---

## Kết nối thực tế

- **React**: JSX compile thành Virtual DOM tree → Binary Tree traversal
- **Browser**: DOM = N-ary tree, CSS Selector dùng tree matching
- **Node.js**: AST (Abstract Syntax Tree) trong Babel, ESLint
- **Webpack**: Dependency Tree để tính toán bundle
- **Redux DevTools**: State history tree
