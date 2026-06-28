# 🟣 Rope — Cấu trúc dữ liệu chuỗi văn bản lớn

## 🧠 Định nghĩa dễ nhớ

> **Rope** giống như **ghép các sợi dây thừng nhỏ thành một sợi dây thừng lớn**: thay vì lưu toàn bộ cuốn tiểu thuyết 1000 trang vào một chuỗi (String) phẳng liên tục trong RAM, Rope chia nhỏ cuốn tiểu thuyết thành các đoạn văn ngắn (các chiếc lá của cây nhị phân) và quản lý chúng bằng cấu trúc Cây. Khi bạn chèn một câu mới vào trang 500, Rope không cần dịch chuyển hàng triệu ký tự phía sau, nó chỉ việc cắt đôi "sợi dây" tại trang 500 và chèn "sợi dây" mới vào giữa.

```
          [Weight: 9]
          /         \
     [Weight: 6]     "world" (len: 5)
     /         \
 "hello_"       "my_"
 (len: 6)       (len: 3)
```
- Node trung gian lưu **Weight** = tổng độ dài của các chuỗi thuộc **cây con bên trái**.
- Node lá lưu các chuỗi phẳng thực sự (Flat String).

---

## ⏱️ Big-O Complexity

So sánh thao tác trên chuỗi có tổng độ dài **n**:

| Thao tác | JavaScript String (Flat Array) | Rope (Cây nhị phân) | Ghi chú |
|----------|--------------------------------|---------------------|---------|
| Concatenate (Ghép) | O(n) | **O(1)** | Chỉ việc tạo node gốc mới trỏ về 2 cây cũ |
| Insert (Chèn ở giữa) | O(n) | **O(log n)** | Split cây, ghép cây con mới, rồi concat lại |
| Delete (Xóa ở giữa) | O(n) | **O(log n)** | Split cây thành 2 phần bỏ qua đoạn xóa, concat lại |
| Split (Cắt đôi) | O(n) | **O(log n)** | Tìm node tại chỉ số cắt và cơ cấu lại các liên kết |
| Report (Lấy ký tự thứ i) | **O(1)** | **O(log n)** | Đi từ gốc xuống lá dựa theo Weight |

---

## 💻 Code JavaScript

### Mô phỏng tối giản của Rope

```javascript
class RopeNode {
  constructor(str = '') {
    this.left = null;
    this.right = null;
    this.weight = str.length; // Tổng độ dài chuỗi bên trái
    this.value = str; // Chỉ có giá trị ở Node lá
  }
  
  isLeaf() {
    return !this.left && !this.right;
  }
}

class Rope {
  constructor(str = '') {
    this.root = str ? new RopeNode(str) : null;
  }

  // CONCATENATE — Ghép chuỗi trong O(1) ✅
  static concat(rope1, rope2) {
    if (!rope1.root) return rope2;
    if (!rope2.root) return rope1;

    const newRoot = new RopeNode();
    newRoot.left = rope1.root;
    newRoot.right = rope2.root;
    
    // Weight của gốc mới = tổng độ dài của cây con bên trái (rope1)
    newRoot.weight = Rope.#totalLength(rope1.root);
    
    const result = new Rope();
    result.root = newRoot;
    return result;
  }

  static #totalLength(node) {
    if (!node) return 0;
    if (node.isLeaf()) return node.weight;
    return node.weight + Rope.#totalLength(node.right);
  }

  // TRUY CẬP ký tự thứ i — O(log n)
  charAt(index) {
    return this.#charAtHelper(this.root, index);
  }

  #charAtHelper(node, i) {
    if (!node) return '';
    if (node.isLeaf()) {
      return node.value[i];
    }

    if (i < node.weight) {
      // i nhỏ hơn weight -> Ký tự nằm bên trái
      return this.#charAtHelper(node.left, i);
    } else {
      // i >= weight -> Ký tự nằm bên phải
      return this.#charAtHelper(node.right, i - node.weight);
    }
  }

  // Đổ toàn bộ cây ra String phẳng (Duyệt Inorder)
  toString() {
    const result = [];
    function traverse(node) {
      if (!node) return;
      if (node.isLeaf()) {
        result.push(node.value);
        return;
      }
      traverse(node.left);
      traverse(node.right);
    }
    traverse(this.root);
    return result.join('');
  }
}

// Chạy thử nghiệm
const r1 = new Rope("hello_");
const r2 = new Rope("my_");
const r3 = new Rope("world");

// Ghép: "hello_my_"
const r12 = Rope.concat(r1, r2);
// Ghép tiếp: "hello_my_world"
const r123 = Rope.concat(r12, r3);

console.log("Full String:", r123.toString()); // "hello_my_world"
console.log("Ký tự tại index 6:", r123.charAt(6)); // "m"
console.log("Ký tự tại index 9:", r123.charAt(9)); // "w"
```

---

## 🔧 Trong Framework & Thực tế

### 1. VS Code / Monaco Editor — Quản lý văn bản mã nguồn
```
Khi bạn mở một file code lớn (ví dụ file JSON raw dung lượng 50MB) trong VS Code.
Nếu dùng JS String bình thường:
Mỗi khi bạn gõ thêm 1 ký tự vào dòng đầu tiên, JS engine phải phân bổ lại RAM và 
dịch chuyển toàn bộ 50 triệu ký tự phía sau -> Trình duyệt/Editor giật lag lập tức.

VS Code sử dụng cấu trúc Piece Table (một biến thể của Rope) để quản lý văn bản.
Mọi thao tác gõ phím, xóa chữ, copy-paste chỉ tốn O(log n) hoặc O(1) thao tác liên kết cây.
```

### 2. Cấu trúc bộ đệm Terminal (Terminal Buffers)
```
Các chương trình Terminal (như Xterm.js dùng trong VS Code Terminal) lưu trữ lịch sử log 
đầu ra khổng lồ. Việc ghi thêm log mới vào đuôi được tối ưu hóa bằng cách gom nhóm chuỗi 
và quản lý theo dạng danh sách phân mảnh, tương tự tư tưởng của Rope.
```

---

## 🕵️ Kỹ thuật ẩn — Ít người nhận ra

### Phép chèn (Insert) hoạt động như thế nào trong Rope?
Để chèn chuỗi `S` vào vị trí `i` trên Rope `R`:
1. **Split**: Cắt Rope `R` tại vị trí `i` thành 2 Rope con: `R_left` và `R_right`.
2. **Concat 1**: Ghép `R_left` với Rope chứa chuỗi mới `S` -> thu được `R_new_left`.
3. **Concat 2**: Ghép `R_new_left` với `R_right` -> thu được Rope hoàn chỉnh.
-> Cả quá trình chỉ tốn `O(log n)` và hoàn toàn bất biến (Immutable).

---

## ❓ Bẫy phỏng vấn

### Q1: Tại sao không dùng Array chứa các dòng (Array of Strings) để làm Editor?
```
Dùng `Array of Strings` (mỗi phần tử mảng là 1 dòng code) tốt hơn String phẳng, 
nhưng vẫn có nhược điểm:
- Khi chèn một dòng mới vào giữa, hoặc chèn một đoạn văn bản chứa nhiều ký tự xuống dòng (Multiline paste).
  Ta phải dịch chuyển toàn bộ index của mảng: O(n).
- Khi có một dòng code cực dài (không xuống dòng), việc sửa đổi trên dòng đó vẫn mất O(L) với L là độ dài dòng.
-> Rope giải quyết triệt để nhờ việc phân chia nhị phân cả về chiều dọc lẫn chiều ngang.
```

### Q2: Điểm yếu của Rope so với String phẳng là gì?
```
- Tốn bộ nhớ hơn: Mỗi node trung gian tốn RAM lưu các thuộc tính cây (left, right, weight).
- Truy cập ngẫu nhiên (Random Access) chậm hơn: String phẳng truy cập index `[i]` trong O(1). 
  Rope cần O(log n) để duyệt từ gốc xuống.
- Phức tạp trong cài đặt thuật toán so khớp chuỗi (Regex, IndexOf).
```

---

## 🔗 Xem thêm
- [Binary Tree](../03-tree/binary-tree.md) — Cơ sở cấu trúc của Rope
- [Persistent Data Structure](./persistent-data-structure.md) — Rope kết hợp tính bất biến
- [VS Code text buffer architecture](https://code.visualstudio.com/blogs/2018/03/23/text-buffer-reimplementation)
