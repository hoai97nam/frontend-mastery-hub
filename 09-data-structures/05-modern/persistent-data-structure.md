# 🟣 Persistent Data Structure — Cấu trúc dữ liệu bất biến hiệu năng cao

## 🧠 Định nghĩa dễ nhớ

> **Persistent Data Structure** giống như **tính năng chụp ảnh lưu trạng thái (Snapshot)** trong game: khi bạn đưa ra một quyết định mới, game không copy toàn bộ thế giới để tạo màn chơi mới (rất tốn bộ nhớ). Thay vào đó, nó tạo ra một nhánh mới chỉ lưu những thay đổi của bạn, còn lại **dùng chung** toàn bộ bản đồ và quái vật với màn chơi cũ.

Trong lập trình: Khi bạn cập nhật một Object/Array bất biến (Immutable), thay vì dùng `JSON.parse(JSON.stringify(obj))` để clone toàn bộ (O(n)), ta chỉ tạo mới các node nằm trên đường dẫn thay đổi, còn các nhánh không đổi sẽ được trỏ trực tiếp (Structural Sharing).

```
Cây cũ (v1):            Cây mới (v2) - sau khi thay đổi D thành D':
     A                       A' (mới)
    / \                     / \
   B   C                   B   C' (mới)
  / \                     / \ / \
 D   E                   D'  E   F
```
*Nhìn xem: Node B và E được chia sẻ hoàn toàn giữa v1 và v2! Không cần clone B và E.*

---

## ⏱️ Big-O Complexity

So sánh phép cập nhật (Update) với đối tượng có kích thước **n**:

| Cấu trúc | Deep Clone (`JSON.parse`) | Persistent DS (Trie-based) | Ghi chú |
|----------|--------------------------|----------------------------|---------|
| Read | **O(1)** | **O(log₃₂ n)** ≈ O(1) | O(log₃₂ n) cực kỳ nhanh (cây có 32 nhánh) |
| Update | O(n) | **O(log₃₂ n)** ≈ O(1) | Tiết kiệm bộ nhớ và thời gian |
| Space | O(n) | **O(log₃₂ n)** | Chỉ tiêu tốn thêm bộ nhớ cho đường dẫn thay đổi |

*Với n = 1,000,000, cây trie 32 nhánh chỉ cao tối đa 4 tầng -> `log₃₂ (1,000,000) ≈ 4` phép tính.*

---

## 💻 Code JavaScript

### Mô phỏng Structural Sharing đơn giản trên Binary Tree

```javascript
class Node {
  constructor(value, left = null, right = null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }
}

// Hàm cập nhật bất biến (không mutate cây cũ) — O(log n)
function updateTree(root, targetValue, newValue) {
  if (!root) return null;

  if (root.value === targetValue) {
    // Tạo node mới thay thế
    return new Node(newValue, root.left, root.right);
  }

  // Đệ quy nhánh trái
  const newLeft = updateTree(root.left, targetValue, newValue);
  if (newLeft !== root.left) {
    // Nhánh trái đổi -> Phải tạo node cha mới (structural sharing nhánh phải)
    return new Node(root.value, newLeft, root.right);
  }

  // Đệ quy nhánh phải
  const newRight = updateTree(root.right, targetValue, newValue);
  if (newRight !== root.right) {
    // Nhánh phải đổi -> Phải tạo node cha mới (structural sharing nhánh trái)
    return new Node(root.value, root.left, newRight);
  }

  // Không có gì thay đổi -> Trả về reference cũ
  return root;
}

// Chạy thử nghiệm
const rootV1 = new Node('A',
  new Node('B', new Node('D'), new Node('E')),
  new Node('C')
);

const rootV2 = updateTree(rootV1, 'D', 'D-mới');

console.log("v1 và v2 khác nhau gốc:", rootV1 !== rootV2); // true
console.log("Nhánh C được chia sẻ:", rootV1.right === rootV2.right); // true ✅
console.log("Node E được chia sẻ:", rootV1.left.right === rootV2.left.right); // true ✅
```

---

## 🔧 Trong Framework & Thực tế

### 1. Redux / Zustand — Quản lý State bất biến
```javascript
// Quy tắc số 1 của Redux: "Không bao giờ được thay đổi trực tiếp state"
// Phải return state mới.
const todoReducer = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      // Dùng spread operator [...] thực chất là cạn (shallow copy) của array,
      // chia sẻ reference của các item cũ -> O(n).
      return [...state, action.payload];
      
    default:
      return state;
  }
};
```

### 2. Immer.js — Tạo Draft State kỳ diệu
```javascript
// Immer dùng ES6 Proxy để track các thao tác ghi đè của bạn lên "draft" state
// và tự động xây dựng một Persistent Data Structure ở phía sau bằng structural sharing.
import { produce } from 'immer';

const baseState = {
  users: [{ name: 'Alice', active: true }, { name: 'Bob', active: false }],
  metadata: { count: 2 }
};

const nextState = produce(baseState, draft => {
  draft.users[0].active = false; // Nhìn giống như mutation trực tiếp
});

// Nhưng thực chất:
console.log(baseState !== nextState); // true
console.log(baseState.metadata === nextState.metadata); // true ✅ (Structural Sharing)
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### Bitmapped Vector Trie (Cơ chế của Immutable.js)
Để lưu trữ Array lớn một cách bất biến hiệu quả, Immutable.js không dùng Linked List hay Cây nhị phân. Họ dùng **Bitmapped Vector Trie** với bậc phân nhánh là 32 (32-way tree).
- Mỗi tầng của cây đại diện cho 5 bit của chỉ số mảng (2⁵ = 32).
- Ví dụ, để tìm phần tử thứ 105 (nhị phân: `00000000000000000000000001101001`):
  - Lấy 5 bit đầu tiên để chọn nhánh ở tầng 1.
  - Lấy 5 bit tiếp theo để chọn nhánh ở tầng 2.
  - ...
  -> Tìm kiếm và cập nhật mảng 1 triệu phần tử chỉ mất tối đa 4 bước nhảy con trỏ.

---

## ❓ Bẫy phỏng vấn

### Q1: Tại sao không dùng `structuredClone` để update state trong React/Redux?
```javascript
// structuredClone (ES2022) hoặc JSON.parse(JSON.stringify(state))
// sao chép sâu (deep clone) mọi ngóc ngách của object.
// Tác hại:
// 1. Tốn CPU và bộ nhớ cực lớn cho dự án phức tạp.
// 2. Phá vỡ cơ chế so sánh nông (Shallow Comparison) của React.
//    React.memo hoặc PureComponent so sánh reference: `prevObj === nextObj`.
//    Nếu deep clone toàn bộ, mọi component con dù dữ liệu không đổi cũng bị render lại 
//    vì reference đã thay đổi.
```

### Q2: Shallow Copy (Spread operator) có phải là Persistent Data Structure không?
```
Có, spread operator `const newObj = {...oldObj}` là hình thức đơn giản nhất của structural sharing. 
Nó tạo ra một object cha mới nhưng giữ nguyên tham chiếu (reference sharing) của tất cả các property con.
Tuy nhiên, nếu bạn có cấu trúc lồng sâu (Deeply nested), bạn phải làm spread thủ công nhiều tầng, 
rất dễ viết code dài dòng và dễ sai sót -> Nên dùng Immer.js.
```

---

## 🔗 Xem thêm
- [Binary Tree](../03-tree/binary-tree.md) — Kiến thức nền tảng về Cây
- [Trie](../03-tree/trie.md) — Cấu trúc phân nhánh tiền tố dùng cho Bitmapped Vector Trie
- [Immer.js documentation on Structural Sharing](https://immerjs.github.io/immer/docs/performance)
