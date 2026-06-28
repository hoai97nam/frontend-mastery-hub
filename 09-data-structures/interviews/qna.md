# 🎯 Q&A Phỏng vấn Cấu trúc dữ liệu (Data Structures)

Hệ thống câu hỏi phỏng vấn cấu trúc dữ liệu từ cơ bản đến nâng cao, thiết kế riêng cho các ứng viên Frontend (từ Middle đến Senior/Architect).

---

## 🔵 Cấp độ: Junior / Middle

### **Q1: Tại sao trong JavaScript `Array.unshift()` và `Array.shift()` lại chậm hơn nhiều so với `Array.push()` và `Array.pop()`?**
**A:** 
- `push()` và `pop()` thao tác ở **cuối mảng**. Khi thực hiện, JavaScript engine chỉ việc ghi đè hoặc xóa giá trị tại ô nhớ cuối cùng trong `O(1)` (hoặc khấu hao `O(1)` khi resize).
- `unshift()` và `shift()` thao tác ở **đầu mảng** (chỉ số `0`). Để thêm hoặc xóa ở đầu, engine bắt buộc phải dịch chuyển (shift) tất cả `n-1` phần tử còn lại sang phải hoặc sang trái một đơn vị vị trí bộ nhớ. Thao tác này có độ phức tạp thời gian là **`O(n)`**.
- **Lời khuyên**: Nếu cần thêm/xóa ở đầu nhiều, hãy cân nhắc sử dụng **Linked List** hoặc **Deque** thay vì Array phẳng.

---

### **Q2: `Map` và `Object` trong JavaScript khác nhau như thế nào? Khi nào nên chọn `Map`?**
**A:**
1. **Kiểu dữ liệu của Key**:
   - `Object`: Key chỉ có thể là String hoặc Symbol. Nếu truyền Object khác làm key, nó tự chuyển thành chuỗi `"[object Object]"`.
   - `Map`: Key có thể là bất kỳ kiểu dữ liệu nào (Object, Function, NaN, Primitive).
2. **Thứ tự phần tử**:
   - `Object`: Thứ tự lặp không được đảm bảo chính xác tuyệt đối (đặc biệt là các key dạng số).
   - `Map`: Luôn đảm bảo thứ tự chèn (Insertion Order) khi lặp qua các phần tử.
3. **Thuộc tính kích thước**:
   - `Object`: Muốn biết độ dài phải dùng `Object.keys(obj).length` (mất `O(n)`).
   - `Map`: Truy cập thuộc tính `.size` có sẵn trong `O(1)`.
4. **Hiệu năng**:
   - `Map` được tối ưu hóa đặc biệt cho các kịch bản thêm/xóa phần tử liên tục lúc runtime.
- **Khi nào chọn Map**: Khi key không phải là string, khi cần giữ thứ tự chèn, khi thực hiện thêm/xóa liên tục, hoặc khi cần kiểm tra số lượng phần tử nhanh chóng.

---

### **Q3: Trình bày cơ chế hoạt động của JavaScript Call Stack. Lỗi "Maximum call stack size exceeded" xảy ra khi nào?**
**A:**
- JavaScript engine hoạt động theo cơ chế đơn luồng (single-threaded) và sử dụng một **Call Stack** (LIFO) để theo dõi các lượt thực thi hàm.
- Khi một hàm được gọi, một frame chứa tham số và biến cục bộ của nó được push lên đỉnh Call Stack. Khi hàm chạy xong, frame đó được pop ra khỏi stack.
- Lỗi **"Maximum call stack size exceeded"** (Stack Overflow) xảy ra khi Call Stack bị quá tải dung lượng cho phép. Thường xảy ra do một hàm đệ quy không có điều kiện dừng (vòng lặp vô tận) hoặc đệ quy quá sâu khiến các frame xếp chồng lên nhau liên tục mà không có cơ hội pop ra.

---

### **Q4: Giải thích ứng dụng của Queue trong cơ chế hoạt động của Event Loop ở trình duyệt?**
**A:**
Event Loop quản lý việc thực thi các tác vụ bất đồng bộ thông qua các hàng đợi (Queues - FIFO):
1. **Task Queue (Macrotask Queue)**: Chứa các tác vụ như `setTimeout`, `setInterval`, I/O, event listeners.
2. **Microtask Queue**: Chứa các tác vụ ưu tiên cao hơn như `Promise.then` callbacks, `MutationObserver`, `queueMicrotask`.

**Luồng hoạt động**:
- Sau khi Call Stack trống, Event Loop sẽ kiểm tra và thực thi **tất cả** các tác vụ trong **Microtask Queue** cho tới khi hàng đợi này trống hoàn toàn.
- Sau đó, nó mới lấy **duy nhất một** tác vụ từ **Macrotask Queue** đẩy lên Call Stack để chạy, rồi lại quay lại dọn sạch Microtask Queue.

---

## 🟢 Cấp độ: Senior

### **Q5: Giải thích cấu trúc React Fiber. Tại sao React lại tái cấu trúc công cụ từ cây đệ quy sang mô hình Linked List?**
**A:**
- Ở phiên bản React cổ điển (React 15 trở về trước), quá trình so khớp Virtual DOM (Reconciliation) sử dụng thuật toán duyệt cây đệ quy (DFS). Một khi đã chạy, luồng thực thi chiếm đóng Call Stack của trình duyệt và **không thể dừng lại** cho đến khi duyệt xong. Với cây lớn, điều này làm đơ khung hình (drop frame), gây giật lag UI khi user scroll hoặc gõ phím.
- Từ React 16, kiến trúc **React Fiber** được ra đời. Mỗi component được biểu diễn bằng một nút **Fiber Node**, được liên kết chéo với nhau dưới dạng **Doubly Linked List** thông qua 3 con trỏ chính:
  - `.child`: Trỏ đến phần tử con đầu tiên.
  - `.sibling`: Trỏ đến phần tử anh em kế tiếp.
  - `.return`: Trỏ ngược về component cha.
- **Lợi ích**: Nhờ cấu trúc Linked List, React không cần Call Stack đệ quy của JS để duyệt cây nữa. React có thể tự quản lý một con trỏ trạng thái ảo (`nextUnitOfWork`). Khi có tác vụ ưu tiên cao hơn từ trình duyệt (như user gõ chữ), React có thể **tạm dừng (Pause)** render, nhường luồng cho trình duyệt vẽ UI, sau đó **tiếp tục (Resume)** từ vị trí con trỏ cũ hoặc **hủy bỏ (Abort)** render nếu dữ liệu đã lỗi thời.

---

### **Q6: Làm thế nào để tránh Memory Leak khi lưu dữ liệu cache theo từng Object trong JavaScript? Hãy so sánh `Map` và `WeakMap`?**
**A:**
- Nếu dùng `Map` thông thường, khi ta gán một Object làm key:
  ```javascript
  let user = { name: "Alice" };
  const cache = new Map();
  cache.set(user, "data_cache");
  user = null; // Hủy biến ngoài
  ```
  Object `{ name: "Alice" }` vẫn không được giải phóng bởi bộ thu gom rác (Garbage Collector - GC) vì nó vẫn đang bị `Map` tham chiếu mạnh (Strong Reference). Điều này tích tụ lâu ngày gây ra hiện tượng tràn bộ nhớ (Memory Leak).
- Khi dùng `WeakMap`:
  - Key của `WeakMap` bắt buộc phải là một Object.
  - `WeakMap` giữ các tham chiếu yếu (Weak Reference) tới key. Khi biến `user` ngoài bị gán bằng `null` và không còn tham chiếu nào khác ngoài `WeakMap`, GC sẽ tự động dọn dẹp cả key và value tương ứng ra khỏi RAM trong đợt quét tiếp theo.
  - Vì key có thể bị xóa bất cứ lúc nào một cách ngẫu nhiên, `WeakMap` không hỗ trợ các phương thức duyệt qua danh sách như `.keys()`, `.values()`, `.entries()`, `.clear()` hay thuộc tính `.size`.

---

### **Q7: Trình bày sự khác biệt giữa cấu trúc String phẳng thông thường và cấu trúc Rope trong việc xử lý văn bản lớn? VS Code sử dụng cấu trúc nào?**
**A:**
- **String phẳng**: Lưu trữ liên tục trong bộ nhớ RAM. 
  - Ưu điểm: Lấy ký tự tại vị trí `i` cực nhanh trong `O(1)`.
  - Nhược điểm: Khi chèn hoặc xóa ở giữa một chuỗi lớn, trình duyệt phải cấp phát vùng RAM mới và dịch chuyển tất cả ký tự phía sau (`O(n)`).
- **Rope**: Cây nhị phân mà ở đó mỗi node lá chứa các chuỗi ngắn hơn, node trung gian chứa trọng số độ dài của các node lá bên trái.
  - Ưu điểm: Các thao tác chèn, xóa, chia đôi, ghép chuỗi lớn chỉ mất **`O(log n)`** vì bản chất chỉ là tạo/ngắt các con trỏ kết nối trên cây, không dịch chuyển mảng ký tự phẳng.
  - Nhược điểm: Việc lấy ký tự ngẫu nhiên mất `O(log n)` do phải duyệt từ gốc xuống lá.
- **VS Code** sử dụng cấu trúc **Piece Table** (một cấu trúc dữ liệu tối ưu hóa của Rope) để quản lý bộ đệm văn bản, giúp mở các file code hàng trăm MB vẫn mượt mà.

---

### **Q8: Bloom Filter là gì? Kể tên một ứng dụng thực tế của Bloom Filter trong kiến trúc hệ thống Web?**
**A:**
- **Bloom Filter** là cấu trúc dữ liệu xác suất (Probabilistic DS) cực kỳ tiết kiệm bộ nhớ, dùng để kiểm tra một phần tử có tồn tại trong tập hợp hay không.
- Đặc điểm:
  - Trả về **False**: Chắc chắn 100% phần tử KHÔNG có trong tập hợp (No False Negative).
  - Trả về **True**: Phần tử CÓ THỂ có trong tập hợp, thỉnh thoảng có sai số trùng lặp (False Positive).
- **Ứng dụng thực tế**:
  1. **Chặn website độc hại trên Chrome**: Chrome tải một Bloom Filter dung lượng nhẹ (~2MB) chứa mã băm các trang web độc hại về máy local của user. Khi user click link, Chrome check local trước. Nếu trả về False (an toàn 100%), Chrome mở trang ngay mà không cần tốn thời gian query server Google.
  2. **Cache Stampede / CDN**: Chặn các request phá hoại (request các bài viết không tồn tại để ép DB query liên tục gây sập hệ thống). Bloom Filter lưu danh sách ID bài viết hiện có, nếu ID request không có trong Bloom Filter (False) -> Reject ngay lập tức tại tầng CDN mà không đụng tới Database.

---

## 🔴 Cấp độ: Senior+ / Architect (Tình huống thiết kế thực tế)

### **Tình huống 1: Thiết kế tính năng Undo / Redo cho một ứng dụng vẽ sơ đồ tư duy (Mindmap) chạy trên trình duyệt**
**Yêu cầu**: Tính năng phải chạy mượt mà, hỗ trợ tối thiểu 100 bước undo, và tránh ngốn RAM khi sơ đồ có kích thước lớn (hàng vạn đối tượng).

**Giải pháp đề xuất**:
1. **Cấu trúc dữ liệu nền tảng**: Dùng 2 cấu trúc **Stack** (`undoStack` và `redoStack`) kết hợp với **Persistent Data Structure (Structural Sharing)**.
2. **Quản lý State bất biến**:
   - Không lưu trạng thái theo dạng chuỗi JSON thô (deep clone) vì sẽ làm tràn RAM sau vài chục thao tác vẽ.
   - Sử dụng thư viện như `Immer.js` hoặc cấu trúc cây có chia sẻ liên kết. Khi người dùng di chuyển một nhánh con trên sơ đồ, ta tạo ra một node cha mới trỏ tới nhánh con vừa di chuyển, đồng thời tái sử dụng (chia sẻ reference) của tất cả các nhánh con khác không thay đổi.
3. **Luồng xử lý**:
   - Khi có hành động mới: Push snapshot mới vào `undoStack`, đồng thời clear `redoStack`. Nếu `undoStack.length > 100`, pop phần tử cũ nhất ở đáy stack (implement Stack bằng Deque hoặc Ring Buffer để hỗ trợ giới hạn kích thước dễ dàng).
   - Khi nhấn **Undo**: Pop trạng thái hiện tại khỏi `undoStack`, push vào `redoStack`, sau đó cập nhật UI bằng trạng thái ở đỉnh `undoStack` mới.
   - Khi nhấn **Redo**: Pop khỏi `redoStack`, push vào `undoStack`, cập nhật UI.

---

### **Tình huống 2: Xây dựng hệ thống tự động gợi ý từ khóa (Autocomplete/Search Suggestion) thời gian thực chạy ở client-side cho trang thương mại điện tử lớn**
**Yêu cầu**: Tìm kiếm phản hồi < 50ms khi người dùng gõ phím, hỗ trợ tìm kiếm không dấu/có dấu tiếng Việt, giới hạn dung lượng lưu trữ offline trong LocalStorage < 5MB cho 50,000 từ khóa phổ biến nhất.

**Giải pháp đề xuất**:
1. **Cấu trúc dữ liệu nền tảng**: **Trie (Cây tiền tố)**.
2. **Tối ưu hóa bộ nhớ**:
   - Chuyển cấu trúc Trie dạng Object lồng nhau (`{ children: { a: { children: {} } } }`) thành dạng **Radix Tree (Compressed Trie)**. Gom các chuỗi ký tự đơn không phân nhánh thành một node duy nhất (ví dụ: gom `a -> p -> p -> l -> e` thành `apple`), giúp giảm số lượng object con trỏ trong JS tới 60%.
   - Chuẩn hóa text: Chuyển tất cả ký tự có dấu thành không dấu và lưu kèm vào Trie node để hỗ trợ tìm kiếm không dấu.
3. **Truy vấn nhanh**:
   - Mỗi node trong Trie sẽ lưu trữ thêm một mảng nhỏ chứa **Top 5 từ khóa hoàn chỉnh phổ biến nhất** thuộc nhánh con đó (được tính toán trước từ lúc build cây ở server).
   - Khi người dùng gõ chữ, ta chỉ cần duyệt đi xuống theo ký tự đến node cuối cùng trong `O(L)` (L là độ dài từ đang gõ). Ngay tại node đó, ta đọc mảng Top 5 có sẵn và trả về kết quả lập tức mà không cần chạy thuật toán DFS đi xuống các lá để gom từ lúc runtime.

---

### **Tình huống 3: Thiết kế tính năng hiển thị số lượng người truy cập độc nhất (Unique Viewers) theo thời gian thực cho một livestream có hàng triệu người xem đồng thời**
**Yêu cầu**: Số lượng phải cập nhật liên tục từng giây, chấp nhận sai số nhỏ (< 1%), chi phí phần cứng máy chủ tối thiểu.

**Giải pháp đề xuất**:
1. **Cấu trúc dữ liệu nền tảng**: **HyperLogLog (HLL)**.
2. **Kiến trúc hệ thống**:
   - Không sử dụng `HashSet` (Redis Set) để lưu danh sách ID người xem vì với 10 triệu người xem, Redis sẽ ngốn hàng GB RAM chỉ để lưu chuỗi String ID, và thao tác kiểm tra trùng lặp sẽ bị nghẽn CPU.
   - Sử dụng lệnh `PFADD livestream_123 <userId>` của Redis HyperLogLog. Mỗi khi user gửi heartbeat lên server, ta đẩy ID của họ vào HLL.
   - Cấu trúc HLL của Redis chỉ tiêu tốn đúng **12KB RAM** cố định cho mỗi livestream.
   - Khi cần hiển thị số lượng, gọi lệnh `PFCOUNT livestream_123` để lấy giá trị ước lượng trong `O(1)` với sai số dưới 1%, hoàn toàn đáp ứng yêu cầu real-time livestream.
