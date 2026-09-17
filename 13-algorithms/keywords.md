# 🔑 Senior Keywords — Algorithms, Sorting & Filtering

> Tổng hợp các từ khóa, cơ chế và thuật ngữ cốt lõi về Thuật toán Sắp xếp, Lọc dữ liệu và Xử lý Performance ở cấp độ **Senior / Architect** phục vụ cho việc tra cứu và phỏng vấn.

---

## 📑 Mục Lục (Table of Contents)
- [📋 Bảng Tra Cứu Từ Khóa Senior](#-bảng-tra-cứu-từ-khóa-senior)
- [🎯 Chi Tiết Theo Nhóm Chuyên Sâu](#-chi-tiết-theo-nhóm-chuyên-sâu)
  - [1. Cơ chế bên dưới (Under the Hood)](#1-cơ-chế-bên-dưới-under-the-hood)
  - [2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)](#2-tối-ưu-hiệu-năng--bộ-nhớ-performance--memory)
  - [3. Kiến trúc UI & Large Dataset Handling (UI Architecture)](#3-kiến-trúc-ui--large-dataset-handling-ui-architecture)
  - [4. Bẫy phỏng vấn & Case Studies (Interview Triggers)](#4-bẫy-phỏng-vấn--case-studies-interview-triggers)

---

## 📋 Bảng Tra Cứu Từ Khóa Senior

| Phân loại | Thuật ngữ / Keyword | Senior Pitch (Điểm đắt giá) | Từ khóa tìm kiếm mở rộng |
|-----------|---------------------|-----------------------------|--------------------------|
| **Sorting** | **Stable Sort (Tính ổn định)** | Thuật toán sắp xếp giữ nguyên thứ tự tương quan ban đầu của các phần tử có cùng giá trị key. Cực kỳ quan trọng khi người dùng click sắp xếp theo nhiều cột liên tiếp trên UI Data Table. | Stable vs Unstable sorting algorithm UI Data Table |
| **Sorting Engine** | **TimSort (V8 Hybrid Sort)** | Thuật toán sắp xếp mặc định của JavaScript V8 (`Array.prototype.sort()`), kết hợp giữa Insertion Sort (dữ liệu nhỏ/runs) và Merge Sort (gộp các runs đã sort). | V8 Array.prototype.sort implementation TimSort |
| **Searching** | **Binary Search Lower / Upper Bound** | Kỹ thuật tìm vị trí phần tử đầu tiên hoặc cuối cùng thỏa điều kiện trong mảng đã sắp xếp trong thời gian $O(\log n)$, làm nền tảng cho Virtual Scroll & Data Windowing. | Binary search lower_bound upper_bound JS |
| **Optimization** | **Two Pointers Technique** | Kỹ thuật dùng 2 con trỏ (chạy cùng chiều hoặc ngược chiều) biến các bài toán lặp lồng nhau $O(n^2)$ thành lặp đơn $O(n)$ trên mảng đã sort. | Two pointers pattern array manipulation |
| **Optimization** | **Sliding Window Technique** | Kỹ thuật duy trì một cửa sổ kích thước động hoặc cố định để tính toán kết quả liên tục trên subarray/substring với độ phức tạp $O(n)$. | Sliding window algorithm substring array |
| **Text Search** | **Trie Prefix Search & Autocomplete** | Cấu trúc cây tiền tố cho phép tìm kiếm và gợi ý từ theo prefix với thời gian $O(L)$ (chỉ phụ thuộc độ dài từ nhập vào, không phụ thuộc tổng số từ $N$). | Trie data structure autocomplete typeahead |
| **Fuzzy Search** | **Levenshtein Distance / Bitap** | Thuật toán đo số bước chèn, xóa, thay thế ký tự tối thiểu để biến chuỗi A thành chuỗi B, ứng dụng trong ô tìm kiếm thông minh tự sửa lỗi gõ sai. | Edit distance Levenshtein fuzzy search JS |
| **Data Structure** | **QuickSelect (Top-K Selection)** | Thuật toán hoán vị mảng dựa trên Pivot của QuickSort để tìm phần tử lớn thứ $K$ hoặc Top $K$ phần tử với độ phức tạp trung bình $O(n)$ (nhanh hơn Sort mảng $O(n \log n)$). | Quickselect algorithm Top-K items JS |
| **Performance** | **Off-Main-Thread Processing (Web Workers)** | Đẩy các tác vụ lọc/sắp xếp bộ dữ liệu lớn ($100.000+$ bản ghi) ra Web Worker để không gây nghẽn UI Main Thread (0ms Jank). | Web Worker transferrable objects array sorting |
| **Performance** | **Time Slicing / Idle Callbacks** | Chia nhỏ tác vụ lọc/sắp xếp thành các chunks nhỏ thực thi trong `requestIdleCallback` hoặc `scheduler.yield()` để giữ FPS màn hình mượt mà 60fps. | Time slicing JS scheduler.yield requestIdleCallback |

---

## 🎯 Chi Tiết Theo Nhóm Chuyên Sâu

### 1. Cơ chế bên dưới (Under the Hood)
- **V8 Engine Array.sort Internals**: Hiểu cách V8 chuyển đổi giữa InsertionSort (khi subarray $\le 10$ phần tử) và TimSort cho mảng lớn. Biết được tại sao custom comparator trong JS phải trả về số âm (`< 0`), số dương (`> 0`), hoặc `0`.
- **In-Place Sorting vs Out-of-Place**: QuickSort là in-place (Space $O(\log n)$ call stack), trong khi MergeSort cần mảng phụ (Space $O(n)$). Trình bày được ảnh hưởng của bộ nhớ Heap/Stack khi thực thi trên trình duyệt.
- **Pivot Selection Strategies**: Tránh trường hợp Worst Case $O(n^2)$ của QuickSort bằng các kỹ thuật chọn Pivot: Median-of-Three hoặc Random Pivot.

### 2. Tối ưu hiệu năng & Bộ nhớ (Performance & Memory)
- **Multi-Field In-Memory Filtering Index**: Xây dựng mảng Index hoặc Hash Map từ trước (Pre-indexing) để khi người dùng gõ filter, hệ thống tìm kiếm trong $O(1)$ hoặc $O(\log n)$ thay vì lặp qua từng object $O(N)$.
- **Avoid Garbage Collection Spikes**: Tránh tạo ra quá nhiều object/array rác trong vòng lặp sort/filter bằng cách reuse mảng hoặc dùng TypedArrays (`Int32Array`, `Float64Array`).

### 3. Kiến trúc UI & Large Dataset Handling (UI Architecture)
- **Stable Multi-Column Sorting Engine**: Cách viết Comparator hàm tổng quát hỗ trợ xếp hạng theo thứ tự ưu tiên các cột (ví dụ: `SortBy: [ {field: 'status', dir: 'asc'}, {field: 'price', dir: 'desc'} ]`).
- **Paginated & Virtual Scroll Filtering Pipeline**: Kết hợp Binary Search với Virtual Scroll window để chỉ render 20-50 phần tử đang xuất hiện trên màn hình viewport, dù tổng số kết quả lọc được lên tới $100.000+$ items.

### 4. Bẫy phỏng vấn & Case Studies (Interview Triggers)
- **Bẫy `Array.prototype.sort()` của JavaScript**: Tại sao `[10, 2, 5, 1].sort()` lại cho ra `[1, 10, 2, 5]`? (Vì JS mặc định ép kiểu phần tử thành `string` rồi so sánh theo UTF-16 code units!).
- **Bẫy Stable Sort trong UI**: Tại sao khi sort lại một cột đã được sort trước đó, thứ tự các hàng cùng giá trị bị nhảy xáo trộn? (Do dùng thuật toán Unstable sort như QuickSort gốc).
- **Top K Element Trap**: Tại sao không nên dùng `array.sort().slice(0, K)` khi $N = 1.000.000$ và $K = 10$? (Giải thích lý do QuickSelect $O(N)$ hoặc Min-Heap $O(N \log K)$ hiệu quả hơn gấp nhiều lần).
