# 🧠 Câu Hỏi Phỏng Vấn Chuyên Sâu & Tình Huống Micro Frontends

> Tổng hợp các câu hỏi phỏng vấn Concept nâng cao và 7 Tình huống thiết kế hệ thống (System Design/Troubleshooting) thực tế dành cho Senior Frontend Developer / Frontend Architect.

---

## Mục Lục

1. [Câu Hỏi Concept Nâng Cao](#1-câu-hỏi-concept-nâng-cao)
   - [Câu 1: Cơ chế phân giải Shared Dependencies của Module Federation](#câu-1-cơ-chế-phân-giải-shared-dependencies-của-module-federation)
   - [Câu 2: Những hạn chế lớn nhất khi sử dụng Shadow DOM để cô lập CSS](#câu-2-những-hạn-chế-lớn-nhất-khi-sử-dụng-shadow-dom-để-cô-lập-css)
   - [Câu 3: Giải quyết tranh chấp Routing khi chuyển trang giữa các MFE](#câu-3-giải-quyết-tranh-chấp-routing-khi-chuyển-trang-giữa-các-mfe)
2. [Tình Huống Thực Tế & Thiết Kế Hệ Thống (Scenario-based)](#2-tình-huống-thực-tế--thiết-kế-hệ-thống-scenario-based)
   - [Tình huống 1: Di chuyển Monolith khổng lồ sang MFE](#tình-huống-1-di-chuyển-monolith-khổng-lồ-sang-mfe)
   - [Tình huống 2: Chia sẻ component bị crash do lỗi phá vỡ tương thích (Breaking Change)](#tình-huống-2-chia-sẻ-component-bị-crash-do-lỗi-phá-vỡ-tương-thích-breaking-change)
   - [Tình huống 3: Đồng bộ và chia sẻ dữ liệu đa chiều thời gian thực (Real-time Sync)](#tình-huống-3-đồng-bộ-và-chia-sẻ-dữ-liệu-đa-chiều-thời-gian-real-time-sync)
   - [Tình huống 4: Rò rỉ bộ nhớ (Memory Leak) khi chuyển trang giữa các MFE](#tình-huống-4-rò-rỉ-bộ-nhớ-memory-leak-khi-chuyển-trang-giữa-các-mfe)
   - [Tình huống 5: Tối ưu hóa bundle size và giải quyết trùng lặp thư viện](#tình-huống-5-tối-ưu-hóa-bundle-size-và-giải-quyết-trùng-lặp-thư-viện)
   - [Tình huống 6: Đảm bảo độc lập Deploy (Independent Deploy) không cần chạm Host](#tình-huống-6-đảm-bảo-độc-lập-deploy-independent-deploy-không-cần-chạm-host)
   - [Tình huống 7: Phòng chống tấn công bảo mật XSS chéo giữa các MFE](#tình-huống-7-phòng-chống-tấn-công-bảo-mật-xss-chéo-giữa-các-mfe)
3. [Thang Đánh Giá Ứng Viên (Evaluation Criteria)](#3-thang-đánh-giá-ứng-viên-evaluation-criteria)

---

## 1. Câu Hỏi Concept Nâng Cao

### Câu 1: Cơ chế phân giải Shared Dependencies của Module Federation

> **Câu hỏi:** *Module Federation giải quyết việc chia sẻ dependencies (như React, Lodash) như thế nào dưới ứng dụng runtime? Điều gì xảy ra nếu Host dùng React v18 và Remote dùng React v17?*

**Điểm cần nghe ở Senior/Architect:**
- **Cơ chế Version Negotiation:** Khi chạy, Webpack sẽ kiểm tra cấu hình `shared` trong file `remoteEntry.js` của tất cả các MFE và so sánh phiên bản được khai báo. Bản có phiên bản cao nhất phù hợp với semantic version (`requiredVersion`) sẽ được chọn để tải.
- **Tình huống lệch Version (v18 vs v17):**
  - Nếu cấu hình là **`singleton: true`**: Webpack bắt buộc chỉ tải 1 bản React duy nhất. Nó sẽ cố gắng chọn bản cao nhất (v18). Trình duyệt sẽ in cảnh báo (warning) ở console vì Remote yêu cầu bản v17 nhưng bị ép dùng v18. Nếu có `strictVersion: true`, ứng dụng sẽ vỡ ngay lập tức (throw error).
  - Nếu **không** cấu hình `singleton: true`: Webpack sẽ tải cả 2 bản React riêng biệt để cung cấp cho Host (v18) và Remote (v17). Điều này làm tăng kích thước bundle nhưng đảm bảo tính tương thích. Đối với React, việc tải 2 bản runtime thường dẫn đến lỗi crash giao diện do đụng độ Context và Element references.

---

### Câu 2: Những hạn chế lớn nhất khi sử dụng Shadow DOM để cô lập CSS

> **Câu hỏi:** *Mặc dù Shadow DOM mang lại khả năng cô lập CSS tuyệt đối, tại sao nhiều dự án lớn vẫn từ chối sử dụng nó cho các MFE con?*

**Điểm cần nghe ở Senior/Architect:**
- **Thư viện UI bên thứ ba bị mất Style:** CSS của các thư viện dùng chung (như Tailwind CSS, Ant Design) nạp ngoài document gốc không thể xuyên qua Shadow DOM để áp dụng cho các phần tử bên trong. Mỗi MFE phải tự nạp lại file CSS đó vào Shadow Root của mình, gây lãng phí bộ nhớ và băng thông mạng.
- **Lỗi hiển thị của Modal/Tooltip (Portals):** Các React Component dạng Portal sẽ gắn (mount) trực tiếp vào `<body>` (ngoài Shadow DOM) để hiển thị đè lên toàn trang. CSS định nghĩa trong Shadow DOM của MFE sẽ không áp dụng được cho Portal này, dẫn đến vỡ giao diện của Modal.
- **Truy cập DOM bị hạn chế:** Các đoạn code phân tích hành vi người dùng (như Google Analytics, Hotjar, GTM) hoặc thư viện xử lý Drag & Drop sẽ gặp khó khăn khi tìm kiếm phần tử bằng các câu lệnh `document.querySelector` thông thường do ranh giới Shadow Root.

---

### Câu 3: Giải quyết tranh chấp Routing khi chuyển trang giữa các MFE

> **Câu hỏi:** *Khi tích hợp nhiều SPA độc lập vào cùng một Shell App, làm sao để tránh tình trạng cả 2 Router (Host và Remote) tranh giành quyền kiểm soát thanh địa chỉ URL của trình duyệt?*

**Điểm cần nghe ở Senior/Architect:**
- **Tách biệt vai trò Router:** Shell App sở hữu `Browser History` để trực tiếp đọc/ghi URL trình duyệt. Các Remote MFE chỉ sử dụng `Memory History` (không tự ý thay đổi địa chỉ URL bên ngoài).
- **Cơ chế Lắng nghe và Ủy quyền (Navigation Syncing):**
  1. Khi người dùng click link chuyển trang bên trong Remote: Remote Router cập nhật trạng thái trong bộ nhớ của nó, sau đó gọi một callback để báo cho Host biết đường dẫn mới.
  2. Host App nhận thông tin và cập nhật `Browser History` tương ứng để thay đổi URL hiển thị.
  3. Khi người dùng click nút Back/Forward của trình duyệt: Host Router bắt được sự kiện thay đổi URL, sau đó gọi hàm API nội bộ của Remote App để cập nhật thủ công Memory Router bên trong Remote.

---

## 2. Tình Huống Thực Tế & Thiết Kế Hệ Thống (Scenario-based)

### Tình huống 1: Di chuyển Monolith khổng lồ sang MFE

> **Đề bài:** *Bạn được giao nhiệm vụ chuyển đổi một hệ thống quản lý bán hàng (E-commerce Admin) cũ viết bằng Angular sang kiến trúc Micro Frontends. Hệ thống có hàng trăm trang và đang phình to khó bảo trì. Bạn sẽ lập kế hoạch tích hợp và lựa chọn công nghệ như thế nào?*

* **Điểm cần nghe (Senior Expectation):**
  - **Phân tách nghiệp vụ (Domain-driven Design):** Không chia cắt theo chiều ngang (UI, Logic, API) mà chia theo chiều dọc (Sub-domains): Dashboard, Đơn hàng, Kho hàng, Khách hàng. Mỗi sub-domain là một MFE do một team chịu trách nhiệm độc lập.
  - **Lựa chọn Tích Hợp:** Vì đang chạy Angular Monolith cũ, không thể chuyển đổi ngay 100% sang React/Vue. Giải pháp ban đầu là dùng **JS Injection** hoặc **Web Components** làm cầu nối tích hợp. Host App mới (viết bằng React hoặc chính Angular mới) sẽ load động các MFE cũ qua Custom Elements để giảm thiểu tối đa việc viết lại code từ đầu.
  - **Chiến lược Strangler Fig Pattern:** Không đập đi xây lại toàn bộ cùng lúc. Tạo Host App mới, di chuyển từng module nhỏ (ví dụ module Khách hàng) sang MFE trước, trỏ router về MFE mới, giữ nguyên các module khác chạy trên hệ thống monolith cũ qua iframe hoặc redirect. Dần dần bóp chết monolith cũ cho đến khi hoàn thành chuyển đổi.
* **Điểm trừ (Red Flags):**
  - Đề xuất viết lại toàn bộ (Rewrite) hệ thống bằng Module Federation ngay lập tức (quá rủi ro, tốn thời gian, gián đoạn kinh doanh).
  - Đề xuất chia nhỏ ở mức quá chi tiết (ví dụ: mỗi nút bấm, mỗi form nhỏ là một MFE riêng) -> Gây cực hình cho việc cấu hình, build và làm giảm hiệu năng hệ thống (Over-engineering).

---

### Tình huống 2: Chia sẻ component bị crash do lỗi phá vỡ tương thích (Breaking Change)

> **Đề bài:** *Hệ thống của bạn có 3 team độc lập phát triển 3 MFE khác nhau. Cả 3 MFE đều sử dụng chung một Component `<SharedButton>` từ MFE Core UI của team Platform. Một ngày nọ, team Platform nâng cấp `<SharedButton>`, thay đổi kiểu dữ liệu của prop `theme` từ dạng chuỗi (`'dark' | 'light'`) sang dạng Object (`{ mode: 'dark', primaryColor: 'blue' }`). Ngay lập tức, 2 MFE của các team khác bị lỗi hiển thị và crash runtime. Bạn giải quyết bài toán này như thế nào để tránh lặp lại?*

* **Điểm cần nghe (Senior Expectation):**
  - **Sử dụng Semantic Versioning (SemVer) & NPM Registry:** Cách an toàn nhất để chia sẻ Component dùng chung có tính logic nghiệp vụ cao là đóng gói chúng thành npm package độc lập và quản lý phiên bản rõ ràng (`v1.2.0`, `v2.0.0`).
  - **Áp dụng Contract Testing:** Triển khai **Contract Testing (như Pact)** trên CI/CD. Khi team Platform sửa đổi code của `<SharedButton>`, pipeline sẽ tự chạy test để verify xem có phá vỡ "hợp đồng" props hiện tại mà các MFE khác đang gọi hay không. Nếu có lỗi breaking change, build pipeline sẽ bị fail ngay lập tức trước khi deploy.
  - **Phiên bản hóa Runtime (với Module Federation):** Nếu bắt buộc phải share runtime qua Module Federation, team Platform phải hỗ trợ cả hai version đồng thời tại entrypoint. Expose ra path mới như `'./Button-v2'` bên cạnh `'./Button'` cũ để các team khác có thời gian migrate dần dần mà không bị sập ứng dụng.
* **Điểm trừ (Red Flags):**
  - Chỉ đề xuất giải pháp thủ công như: "khi nào sửa thì chat thông báo trên Slack để các team cùng biết và sửa theo" (thiếu tự động hóa, dễ quên và sai sót).
  - Không biết khái niệm Contract Testing hoặc SemVer trong việc kiểm soát phiên bản UI Kit.

---

### Tình huống 3: Đồng bộ và chia sẻ dữ liệu đa chiều thời gian thực (Real-time Sync)

> **Đề bài:** *Ứng dụng của bạn gồm Host App và 2 MFE con: MFE Giỏ Hàng (Cart) và MFE Thanh Toán (Checkout) hiển thị song song trên màn hình. Khi người dùng thay đổi số lượng sản phẩm ở MFE Cart, MFE Checkout phải cập nhật ngay lập tức tổng tiền. Ngược lại, nếu MFE Checkout áp mã giảm giá thành công, MFE Cart cũng phải hiển thị lại giá đã giảm. Cả 2 MFE đều được viết bằng 2 framework khác nhau (React & Vue). Bạn sẽ thiết kế giải pháp đồng bộ dữ liệu này như thế nào để đảm bảo tính reactive, tránh race conditions và mất mát dữ liệu?*

* **Điểm cần nghe (Senior Expectation):**
  - **Lựa chọn Custom Events kết hợp Payload sạch:** Vì khác tech stack (React & Vue), không thể dùng các store chuyên dụng của React như Redux/Zustand truyền thẳng xuống Vue. Sử dụng **Custom Events** gửi qua đối tượng toàn cục `window` là giải pháp tối ưu.
  - **Quy tắc luồng dữ liệu hai chiều sạch (Bi-directional Event Flow):**
    - MFE Cart sở hữu thông tin giỏ hàng (Single Source of Truth cho Cart Items). Khi đổi số lượng, Cart dispatch event `CART:UPDATED` với payload chứa danh sách sản phẩm và số lượng. MFE Checkout lắng nghe và tự tính lại tiền.
    - MFE Checkout sở hữu thông tin khuyến mãi. Khi áp mã, Checkout dispatch event `CHECKOUT:DISCOUNT_APPLIED` chứa số tiền giảm và mã code. MFE Cart lắng nghe để cập nhật lại nhãn giá trên UI.
  - **Xử lý Race Conditions:** Đánh dấu timestamp hoặc transaction ID vào payload của event để đảm bảo nếu tin nhắn cũ đến sau tin nhắn mới (do lag mạng hoặc xử lý bất đồng bộ chậm), MFE sẽ bỏ qua và chỉ cập nhật dữ liệu mới nhất.
* **Điểm trừ (Red Flags):**
  - Sử dụng LocalStorage để ghi và đọc liên tục trong luồng real-time (tệ cho hiệu năng vì disk I/O chậm, không reactive, phải dùng cơ chế setInterval để poll liên tục).
  - Thiết kế luồng dữ liệu vòng tròn kín (Circular dependency): Cart đổi -> bắn event -> Checkout nhận -> tính toán xong bắn ngược lại event y hệt -> Cart lại nhận và bắn tiếp -> gây ra vòng lặp vô tận (Infinite render loop).

---

### Tình huống 4: Rò rỉ bộ nhớ (Memory Leak) khi chuyển trang giữa các MFE

> **Đề bài:** *Sau khi triển khai kiến trúc MFE được vài tuần, khách hàng phàn nàn ứng dụng chạy rất mượt lúc đầu, nhưng nếu họ sử dụng liên tục khoảng 30 phút, chuyển qua lại giữa màn hình "Quản lý Đơn hàng" (MFE A) và "Báo cáo doanh thu" (MFE B), trình duyệt bắt đầu đơ và sập tab (Out of Memory). Bạn nghi ngờ có memory leak. Hãy mô tả quy trình debug và cách sửa lỗi.*

* **Điểm cần nghe (Senior Expectation):**
  - **Quy trình Debug bằng Chrome DevTools:**
    1. Mở **Memory Tab** trong Chrome DevTools. Chạy ứng dụng ở trạng thái ban đầu và chụp ảnh bộ nhớ (Take Heap Snapshot 1).
    2. Thực hiện thao tác chuyển trang qua lại giữa MFE A và MFE B khoảng 10-15 lần. Chụp ảnh bộ nhớ thứ hai (Take Heap Snapshot 2).
    3. Sử dụng bộ lọc **"Comparison"** giữa Snapshot 2 và Snapshot 1, tìm kiếm các đối tượng bị giữ lại không được giải phóng (thường là các đối tượng `Detached HTMLElement` hoặc closure của React/Vue component).
  - **Nguyên nhân phổ biến:** Quên dọn dẹp các event listener khi unmount ứng dụng con. Khi Host dỡ bỏ DOM của MFE A để mount MFE B, các callback hàm đăng ký qua `window.addEventListener('EVENT_NAME', callback)` vẫn giữ tham chiếu tới instance của MFE A, ngăn không cho bộ thu gom rác (Garbage Collector) dọn dẹp.
  - **Cách khắc phục:**
    - Trong React, đảm bảo dọn dẹp ở hàm clean-up của `useEffect`.
    - Viết một lifecycle handler chung ở hàm unmount của MFE con để tự động gỡ toàn bộ các event listeners đã tạo lúc khởi chạy.
* **Điểm trừ (Red Flags):**
  - Không biết cách sử dụng Heap Snapshot để tìm rò rỉ bộ nhớ.
  - Khuyên người dùng tắt bớt app khác hoặc cài thêm RAM cho máy tính.

---

### Tình huống 5: Tối ưu hóa bundle size và giải quyết trùng lặp thư viện

> **Đề bài:** *Bạn phát hiện ra tổng dung lượng tải lần đầu của hệ thống lên tới 5MB JS. Phân tích qua bundle analyzer cho thấy cả Host và 3 Remote MFE đều tự đóng gói riêng các thư viện như `lodash`, `moment-timezone` và `axios` vào bundle của mình. Bạn sẽ cấu hình tối ưu hóa như thế nào để chia sẻ các thư viện này ở runtime?*

* **Điểm cần nghe (Senior Expectation):**
  - **Cấu hình Shared Dependencies trong Module Federation:** Khai báo các thư viện dùng chung trong mục `shared` của Webpack/Vite plugin ở cả Host và các Remote.
  - **Sử dụng Dynamic Import:** Đảm bảo toàn bộ việc import components được thực hiện qua Dynamic Import (`React.lazy` và `import()`) để Webpack có thể phân tách (chunking) file JS hiệu quả.
  - **Tận dụng Import Maps cho các thư viện phi trạng thái (Stateless Libs):** Với các thư viện tiện ích như `lodash` hoặc `axios`, thay vì build qua Module Federation, có thể loại trừ chúng khỏi quá trình build (dùng cấu hình `externals` của Webpack) và nạp chúng một lần duy nhất qua Import Maps bằng đường link CDN phiên bản cụ thể (ví dụ: `https://esm.sh/lodash@4.17.21`).
* **Điểm trừ (Red Flags):**
  - Gộp tất cả các thư viện vào một file bundle khổng lồ của Host App và bắt các Remote App phải cài local không qua share (không tối ưu được runtime độc lập).
  - Không hiểu vai trò của thuộc tính `externals` trong Webpack/Rollup.

---

### Tình huống 6: Đảm bảo độc lập Deploy (Independent Deploy) không cần chạm Host

> **Đề bài:** *Nhóm MFE Khách Hàng vừa hoàn thành tính năng mới và muốn triển khai lên Production ngay lập tức. Tuy nhiên, Host App hiện tại đang được cấu hình cứng địa chỉ remoteEntry của MFE Khách Hàng trỏ đến link build cụ thể của version cũ: `https://cdn.company.com/customer/v1.0.0/remoteEntry.js`. Làm thế nào để team Khách hàng tự deploy phiên bản v1.1.0 mà không cần sửa đổi bất kỳ dòng code nào hay deploy lại Host App?*

* **Điểm cần nghe (Senior Expectation):**
  - **Cách 1: Sử dụng Import Maps động (Dynamic Import Maps):** Host App tải file `importmap.json` từ một API config server khi bắt đầu chạy. Khi deploy version mới, CI/CD của team Khách Hàng chỉ cần chạy một cURL request cập nhật bản ghi trong cơ sở dữ liệu config server trỏ `@mfe/customer` sang `.../customer/v1.1.0/remoteEntry.js`. Host App load trang tiếp theo sẽ tự động nhận URL mới.
  - **Cách 2: Cấu hình URL tĩnh trỏ tới alias thư mục mới nhất (Symlink/CDN Alias):** Phía Host App luôn gọi địa chỉ: `https://cdn.company.com/customer/latest/remoteEntry.js`. Khi deploy, script CI/CD của MFE Khách Hàng sẽ thực hiện build, đẩy assets lên thư mục phiên bản `/customer/v1.1.0/`, sau đó tạo liên kết trỏ thư mục `/customer/latest/` sang `/customer/v1.1.0/` trên CDN (hoặc rewrite URL ở Nginx/Cloudflare Worker) và chạy lệnh xóa cache CDN.
* **Điểm trừ (Red Flags):**
  - Khuyên cứ mỗi lần deploy remote thì phải commit sửa file config của Host App và deploy lại cả hai (mất đi ý nghĩa của kiến trúc Micro Frontends).
  - Không biết cách xử lý bộ nhớ đệm (Cache Busting) của tệp tin `remoteEntry.js` dẫn đến việc người dùng nhận mã nguồn cũ do cache trình duyệt lưu trữ quá lâu.

---

### Tình huống 7: Phòng chống tấn công bảo mật XSS chéo giữa các MFE

> **Đề bài:** *MFE Thanh toán của bạn chứa các form nhập thông tin thẻ ngân hàng nhạy cảm. Tuy nhiên, hệ thống lại nhúng chung một MFE tin tức/quảng cáo của bên thứ ba. Nếu MFE bên thứ ba bị hacker tấn công chèn mã độc XSS (Cross-Site Scripting), họ có thể đọc được thông tin thẻ nhập ở MFE Thanh toán vì chúng chạy chung một DOM và window object. Bạn thiết kế lớp phòng thủ bảo mật nào cho trường hợp này?*

* **Điểm cần nghe (Senior Expectation):**
  - **Giải pháp Cô Lập Tuyệt Đối (Iframe Sandbox):** Với các module xử lý thanh toán nhạy cảm hoặc tích hợp từ bên thứ ba không tin cậy, bắt buộc phải sử dụng **Iframe** thay vì Module Federation hay Web Components. Cấu hình thẻ iframe với thuộc tính `sandbox="allow-scripts allow-forms"` và không cho phép `allow-same-origin` nếu muốn cô lập hoàn toàn Cookie và LocalStorage của Host khỏi iframe.
  - **Content Security Policy (CSP):** Thiết lập chính sách CSP nghiêm ngặt trên HTTP Header của Host App:
    - Giới hạn các domain CDN được phép tải file JS (`script-src`).
    - Ngăn chặn inline script (`unsafe-inline`).
    - Cấu hình `connect-src` để giới hạn các domain API mà JS được phép gửi dữ liệu về (ngăn hacker gửi dữ liệu thẻ đánh cắp được ra ngoài server của họ).
  - **Bảo mật Cookie:** Sử dụng Cookie với các cờ `HttpOnly`, `Secure`, và `SameSite=Strict` để JS của MFE bị nhiễm độc không thể đọc được Session Token của người dùng.
* **Điểm trừ (Red Flags):**
  - Cho rằng chạy Module Federation hay Web Components là an toàn vì chúng chạy ở các cổng khác nhau (sai lầm nghiêm trọng: khi tích hợp vào client, tất cả JS đều chạy chung ngữ cảnh bảo mật của Host App).
  - Không biết khái niệm Content Security Policy (CSP).

---

## 3. Thang Đánh Giá Ứng Viên (Evaluation Criteria)

### 🔴 Junior / Mid Level MFE Developer
- **Đặc điểm:** Biết cách chạy dự án MFE có sẵn, hiểu cơ bản Module Federation là gì.
- **Biểu hiện phỏng vấn:**
  - Trả lời được các câu hỏi lý thuyết cơ bản (định nghĩa MFE, các loại tích hợp build-time vs runtime).
  - Biết cách import/export component bằng Module Federation.
  - Gặp khó khăn lớn khi giải quyết vấn đề đụng độ routing, conflict CSS hoặc xử lý lỗi khi remote bị sập.

### 🟡 Senior MFE Developer
- **Đặc điểm:** Có kinh nghiệm setup hệ thống MFE từ đầu, giải quyết được các vấn đề runtime khó.
- **Biểu hiện phỏng vấn:**
  - Trả lời trôi chảy cơ chế version negotiation và tầm quan trọng của `singleton` config.
  - Hiểu sâu về memory leak do event listener và biết cách dọn dẹp đúng cách.
  - Có giải pháp rõ ràng về cô lập CSS (CSS Modules, Tailwind prefixes) và đồng bộ routing (Memory vs Browser history).
  - Biết áp dụng Error Boundary để bảo vệ ứng dụng.

### 🟢 Lead / Principal Architect (MFE Specialist)
- **Đặc điểm:** Tầm nhìn toàn diện về hệ thống, tối ưu hóa quy trình deploy, bảo mật và hiệu năng cho hàng trăm MFE.
- **Biểu hiện phỏng vấn:**
  - Đưa ra giải pháp Strangler Fig Pattern chi tiết để di chuyển các hệ thống monolith khổng lồ không gây gián đoạn kinh doanh.
  - Xây dựng được chiến lược CI/CD deploy độc lập thông qua Dynamic Import Maps hoặc CDN Rewrites.
  - Phân tích sâu sắc các rủi ro bảo mật (XSS, CSP) và đưa ra kiến trúc phòng thủ đa tầng (Iframe sandbox cho module nhạy cảm, Module Federation cho module nội bộ).
  - Thiết kế các quy chuẩn giao tiếp (data flow) rõ ràng để quản lý team độc lập quy mô lớn, áp dụng Contract Testing để đảm bảo chất lượng.
