# 🧠 Câu Hỏi Phỏng Vấn Chuyên Sâu & Tình Huống React Component Patterns

> Tổng hợp các câu hỏi phỏng vấn nâng cao và các tình huống thiết kế hệ thống thực tế về React Component Patterns dành cho vị trí Middle & Senior Frontend Developer.

---

## 1. Câu Hỏi Concept Nâng Cao

### Câu 1: Tối ưu hóa Re-render trong Compound Component bằng Context API

> **Câu hỏi:** *Khi xây dựng Compound Component sử dụng React Context (ví dụ Accordion), làm thế nào để tránh việc tất cả các con trực tiếp đều bị re-render khi chỉ có một phần tử thay đổi trạng thái?*

**Điểm cần nghe ở Senior Developer:**
- **Vấn đề bản chất:** Mỗi khi `openIndex` trong Context Provider thay đổi, tất cả các component con tiêu thụ Context qua `useContext` đều sẽ render lại vì tham chiếu của value Context bị tạo mới.
- **Giải pháp tối ưu:**
  1. **Chia tách Context:** Tách Context làm hai phần: `AccordionStateContext` (chứa dữ liệu `openIndex`) và `AccordionDispatchContext` (chứa hàm cập nhật `toggleIndex` được bọc trong `useCallback`). Các component con chỉ muốn gọi hàm trigger hành động sẽ không bị ảnh hưởng khi state thay đổi.
  2. **Áp dụng React.memo cho con:** Bọc các component con (như `AccordionItem` hoặc `AccordionHeader`) bằng `React.memo`. Đồng thời, tránh truyền inline objects hoặc functions vào value của Provider; hãy dùng `useMemo` để tính toán value Context:
     ```jsx
     const contextValue = useMemo(() => ({ openIndex }), [openIndex]);
     ```
  3. **Tận dụng `children` prop:** Tách cấu trúc DOM phức tạp ra khỏi logic trạng thái và render chúng qua prop `children` để React tự bỏ qua đối với các nhánh cây component không đổi.

---

### Câu 2: Sự thoái trào của HOC và Render Props trước Custom Hooks

> **Câu hỏi:** *Tại sao Custom Hooks lại nhanh chóng trở thành tiêu chuẩn thay thế cho HOC và Render Props trong việc tái sử dụng logic? Có trường hợp nào bạn vẫn chọn HOC hoặc Render Props không?*

**Điểm cần nghe ở Senior Developer:**
- **Tại sao Custom Hooks thắng thế:**
  - **Giảm độ phức tạp cây DOM (Wrapper Hell):** HOC và Render Props tạo ra các component ảo bọc ngoài, khiến cây component phình to (nhìn thấy rõ trên React DevTools). Custom Hooks chia sẻ logic ở mức độ logic JS thuần túy, không tạo thêm node DOM nào.
  - **Dễ đọc và bảo trì (Flat structure):** Hooks cho phép viết code phẳng tuần tự, tránh callback lồng nhau (Callback Hell của Render Props) và không gây mập mờ nguồn gốc của props (HOC thường tiêm prop ảo vào component con mà không có kiểu dữ liệu rõ ràng).
  - **Hợp tác dễ dàng:** Có thể gọi nhiều hooks trong cùng một component và trao đổi dữ liệu qua lại dễ dàng. Với HOC, việc kết hợp 3-4 HOC cùng lúc cực kỳ phức tạp và dễ trùng lặp tên props.
- **Khi nào vẫn chọn HOC/Render Props:**
  - **HOC:** Khi muốn viết Page Guards (Authentication, Authorization) ở tầng cấu hình router mà không muốn sửa bất cứ dòng code nào bên trong trang đó. Hoặc khi làm việc với Class Components trong các dự án cũ.
  - **Render Props:** Khi xây dựng các Component dùng chung ở thư viện UI cần cấu hình giao diện động cực cao (như virtualized lists hoặc các component hiển thị danh sách dạng thẻ/bảng tùy biến render từng dòng).

---

### Câu 3: Bản chất và cách khắc phục lỗi Ref Forwarding khi dùng HOC

> **Câu hỏi:** *Tại sao gán `ref` cho một component được bọc bởi HOC thì ref đó lại không hoạt động như mong muốn? Bạn khắc phục nó như thế nào?*

**Điểm cần nghe ở Senior/Middle Developer:**
- **Nguyên nhân:** Refs không được truyền qua giống như các props thông thường. Khi bọc component bằng HOC, React coi `ref` là thuộc tính của Component bọc ngoài cùng (HOC wrapper) chứ không phải của Component gốc bên trong. Do đó, ref sẽ trỏ tới đối tượng wrapper đó (thường là null hoặc class instance của wrapper) chứ không trỏ tới DOM element thực tế của component gốc.
- **Giải pháp:** Sử dụng API `React.forwardRef` để bắt lấy `ref` được truyền từ cha, sau đó truyền nó xuống component con dưới dạng một prop thông thường (ví dụ đặt tên là `forwardedRef`), rồi gán nó vào component con gốc:
  ```jsx
  export function withLogging(WrappedComponent) {
    function LogProps(props) {
      const { forwardedRef, ...rest } = props;
      return <WrappedComponent ref={forwardedRef} {...rest} />;
    }
    // Sử dụng forwardRef để chuyển tiếp ref
    return React.forwardRef((props, ref) => (
      <LogProps {...props} forwardedRef={ref} />
    ));
  }
  ```

---

## 2. Tình Huống Thực Tế & Thiết Kế Hệ Thống (Scenario-based)

### Tình huống 1: Tối ưu hóa hiệu năng Form nhập liệu khổng lồ

> **Đề bài:** *Bạn tiếp quản một trang quản lý sản phẩm có form nhập liệu chứa hơn 50 inputs (text, select, checkbox, upload...). Hệ thống đang viết ở dạng Controlled Component (lưu trữ tất cả các inputs trong một object state duy nhất ở component cha). Người dùng phản ánh UI bị đơ giật nghiêm trọng khi họ gõ chữ nhanh vào ô input bất kỳ. Bạn xử lý vấn đề này như thế nào?*

* **Hướng giải quyết (Senior Expectation):**
  - **Phân tích nguyên nhân:** Vì dùng Controlled Component cấp cha, mỗi khi người dùng gõ một ký tự, state cha thay đổi, kích hoạt quá trình re-render toàn bộ 50 inputs cùng lúc.
  - **Giải pháp 1: Chuyển sang Uncontrolled Components (Nếu được phép)**
    Sử dụng uncontrolled inputs kết hợp `useRef` hoặc dùng thư viện quản lý form tối ưu như **React Hook Form**. Thư viện này đăng ký (register) input thông qua ref và chỉ re-render tại đúng ô input đang thay đổi hoặc khi submit form.
  - **Giải pháp 2: Cô lập State (State Isolation)**
    Không lưu tất cả dữ liệu form tại một state duy nhất. Tạo một component con bọc riêng từng ô input (ví dụ `<FormInput />`) tự quản lý state gõ chữ cục bộ (`localValue`). Chỉ đẩy dữ liệu lên cha bằng cơ chế debounce (trễ thời gian) hoặc gửi một lần duy nhất khi người dùng chuyển sang input khác (sự kiện `onBlur`).
  - **Giải pháp 3: Sử dụng Context tối ưu**
    Bọc form bằng một Provider, nhưng chỉ cập nhật các DOM Node trực tiếp qua ref mà không trigger re-render component cha.

---

### Tình huống 2: Thiết kế Autocomplete/Combobox cho Enterprise Design System

> **Đề bài:** *Bạn được giao thiết kế một Component `<Combobox>` cho thư viện dùng chung của tập đoàn. Component này phải đáp ứng các tiêu chuẩn: hỗ trợ tìm kiếm trực tiếp, tùy biến giao diện dropdown tùy ý, hỗ trợ phím mũi tên lên/xuống/Enter để chọn, và cho phép các dự án con đồng bộ trạng thái này vào cơ sở dữ liệu của họ. Bạn chọn những patterns nào và thiết kế kiến trúc ra sao?*

* **Hướng giải quyết (Senior Expectation):**
  - **Kiến trúc Compound Component:**
    Để đảm bảo khả năng tùy biến UI tùy ý cho các dự án con, ta chia nhỏ combobox thành:
    ```jsx
    <Combobox>
      <Combobox.Input placeholder="Tìm kiếm..." />
      <Combobox.Dropdown>
        <Combobox.Item value="VN">Việt Nam</Combobox.Item>
        <Combobox.Item value="US">Mỹ</Combobox.Item>
      </Combobox.Dropdown>
    </Combobox>
    ```
    Mẫu thiết kế này giúp các dự án con thoải mái bọc thêm CSS, chèn icon hay phân chia nhóm (Groups) trong dropdown mà không làm gãy logic bên trong. State ẩn (dropdown đang mở hay đóng, item nào đang được hover bằng phím) sẽ được chia sẻ ngầm qua `ComboboxContext`.
  - **Áp dụng State Reducer Pattern:**
    Các hành vi bấm phím (ví dụ bấm phím Down để xuống item tiếp theo) có thể cần thay đổi ở một vài dự án cụ thể (ví dụ: họ muốn khi bấm mũi tên xuống ở item cuối cùng thì nó tự cuộn ngược lên đầu). Ta cung cấp prop `customReducer` để họ cấu hình hành vi xử lý sự kiện phím này.
  - **Áp dụng Control Props Pattern:**
    Component mặc định tự quản lý item được chọn (`selectedItem`). Tuy nhiên, nếu dự án con truyền prop `value` và `onChange`, component sẽ chuyển sang chế độ được kiểm soát, cho phép đồng bộ dễ dàng với State Manager của họ (như Redux hoặc Zustand).
