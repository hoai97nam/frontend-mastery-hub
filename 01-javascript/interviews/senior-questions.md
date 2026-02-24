Dưới đây là 5 câu hỏi "hạng nặng" kèm theo gợi ý cách trả lời để bạn đưa vào file interviews/senior-js.md.

Câu 1: Giải thích cơ chế Event Loop và sự khác biệt về ưu tiên giữa Microtask và Macrotask.
Mục đích: Kiểm tra khả năng dự đoán luồng chạy của code không đồng bộ và tối ưu UI.

Gợi ý trả lời: Senior cần giải thích được rằng Microtasks luôn được thực hiện hết sạch trước khi Event Loop chuyển sang Macrotask tiếp theo hoặc thực hiện Render. Một vòng lặp Microtask vô tận (như Promise đệ quy) sẽ làm treo UI hoàn toàn.

Câu 2: Closures có thể gây ra Memory Leak không? Làm thế nào để phòng tránh?
Mục đích: Kiểm tra kiến thức về quản lý bộ nhớ.

Gợi ý trả lời: Có, nếu closure giữ tham chiếu đến các object lớn trong khi bản thân closure đó không được giải phóng (ví dụ: gán vào một biến global hoặc listener không được remove). Cách tránh: Gán null cho biến sau khi dùng xong hoặc đảm bảo lifecycle của listener rõ ràng.

Câu 3: So sánh Prototypal Inheritance và Class-based Inheritance. Tại sao JS chọn Prototype?
Mục đích: Kiểm tra sự hiểu biết về bản chất ngôn ngữ.

Gợi ý trả lời: JS là "dynamic". Prototype cho phép các object chia sẻ phương thức trực tiếp mà không cần cấu trúc phân cấp cứng nhắc. Điều này giúp tiết kiệm bộ nhớ và linh hoạt hơn trong việc mở rộng object lúc runtime.

Câu 4: Làm thế nào để thực hiện "Deep Clone" một object phức tạp chứa các hàm và kiểu dữ liệu đặc biệt (Date, RegExp)?
Mục đích: Kiểm tra kinh nghiệm thực tế với dữ liệu.

Gợi ý trả lời: Đừng chỉ nói JSON.parse(JSON.stringify()) vì nó mất hàm và lỗi với Date. Senior nên đề cập đến structuredClone() (API mới của browser) hoặc các thư viện như lodash (cloneDeep), hoặc viết hàm đệ quy xử lý từng trường hợp đặc biệt.

Câu 5: Sự khác biệt giữa Map/Set và WeakMap/WeakSet là gì? Khi nào thì dùng "Weak"?
Mục đích: Kiểm tra kỹ năng tối ưu bộ nhớ cao cấp.

Gợi ý trả lời: Điểm mấu chốt là WeakMap không ngăn cản Garbage Collector dọn dẹp các key (object). Nó cực kỳ hữu ích khi bạn muốn gắn metadata vào một object mà không muốn "giữ chân" object đó trong bộ nhớ (ví dụ: lưu cache dữ liệu cho DOM element).