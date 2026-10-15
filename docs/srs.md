# THÀNH PHẦN 1: BẢN SRS RÚT GỌN

## 1. Giới thiệu và phạm vi
- **Bối cảnh:** Chuỗi bán lẻ Mekong Mobile gồm 24 cửa hàng bán lẻ có dữ liệu bán hàng lưu rải rác dưới dạng file Excel định dạng không đồng nhất. Ban giám đốc và Quản lý cửa hàng cần một giải pháp tập trung để trích xuất, làm sạch, chuẩn hóa và nạp dữ liệu vào kho dữ liệu (Data Warehouse) nhằm phục vụ phân tích doanh thu đa chiều.
- **Phạm vi nghiệp vụ:** Trích xuất dữ liệu đơn hàng từ các file Excel cửa hàng, thực hiện làm sạch và chuẩn hóa dữ liệu, nạp vào kho dữ liệu theo mô hình Fact/Dimension và xây dựng báo cáo/dashboard doanh thu nhiều chiều.
- **Điều CHỦ Ý KHÔNG LÀM (MoSCoW - WON'T HAVE):**
  - Không thực hiện dự báo doanh thu bằng các mô hình Machine Learning/AI nâng cao trong phạm vi bài tập này.
  - Không xử lý các giao dịch bán hàng realtime (chỉ xử lý dữ liệu theo lô - Batch ETL).
  - Không xây dựng tính năng phân quyền dữ liệu mức dòng chi tiết cho từng nhân viên bán hàng.

### Bảng thuật ngữ
| Thuật ngữ | Khái niệm / Định nghĩa | Tên đối tượng kĩ thuật |
| :--- | :--- | :--- |
| Đơn hàng | Một giao dịch mua bán sản phẩm tại cửa hàng | `stg_order`, `fact_sales` |
| Kho dữ liệu | Cơ sở dữ liệu tập trung lưu trữ dữ liệu phục vụ phân tích | Data Warehouse |
| Bảng Fact | Bảng chứa các chỉ số đo lường doanh thu, số lượng | `fact_sales` |
| Bảng Dimension | Bảng chứa các chiều phân tích (Thời gian, Cửa hàng, Sản phẩm, Khách hàng) | `dim_store`, `dim_product`, `dim_date`, `dim_customer` |
| Mức độ chi tiết | Mức độ ghi nhận dữ liệu trong bảng fact | Grain: Mỗi dòng tương ứng 1 dòng sản phẩm trong 1 đơn hàng |

## 2. Các bên liên quan và vai trò
| Vai trò | Mô tả nhiệm vụ | Quyền hạn dữ liệu |
| :--- | :--- | :--- |
| Quản lý cửa hàng | Theo dõi doanh thu, hiệu quả bán hàng tại cửa hàng phụ trách. | Được xem báo cáo doanh thu cửa hàng mình; Không sửa đổi dữ liệu kho. |
| Ban giám đốc | Đánh giá tổng quan doanh thu toàn chuỗi, so sánh cửa hàng và dòng sản phẩm. | Được truy cập tất cả dashboard doanh thu toàn chuỗi và xuất báo cáo tổng hợp. |
| Data Analyst (DA) | Xây dựng pipeline ETL, làm sạch dữ liệu và vận hành kho dữ liệu. | Có quyền chạy pipeline ETL, truy vấn trực tiếp vào kho dữ liệu. |

## 3. Yêu cầu chức năng (Functional Requirements & User Stories)
### User Stories
- **US01:** Là Data Analyst, tôi muốn trích xuất dữ liệu đơn hàng từ các file dữ liệu của 24 cửa hàng để có nguồn dữ liệu phục vụ phân tích doanh thu. (MoSCoW: MUST)
- **US02:** Là Data Analyst, tôi muốn phát hiện và xử lý các lỗi dữ liệu bán hàng như sai định dạng ngày, số lượng, đơn giá và thành tiền để dữ liệu có thể sử dụng cho phân tích. (MoSCoW: MUST)
- **US03:** Là Data Analyst, tôi muốn chuẩn hóa thông tin cửa hàng, sản phẩm, khách hàng và thời gian để dữ liệu từ nhiều cửa hàng có cùng cấu trúc và dễ phân tích. (MoSCoW: MUST)
- **US04:** Là Data Analyst, tôi muốn nạp dữ liệu bán hàng đã xử lý vào kho dữ liệu theo mô hình fact và dimension để hỗ trợ truy vấn phân tích doanh thu. (MoSCoW: MUST)
- **US05:** Là Quản lý cửa hàng, tôi muốn xem doanh thu theo ngày, tháng và năm để theo dõi tình hình kinh doanh. (MoSCoW: MUST)
- **US06:** Là Ban giám đốc, tôi muốn xem và so sánh doanh thu giữa các cửa hàng và sản phẩm để có cái nhìn tổng quan về hoạt động bán hàng. (MoSCoW: SHOULD)
- **US07:** Là Ban giám đốc, tôi muốn xem các chỉ số và biểu đồ doanh thu trên dashboard để theo dõi và hỗ trợ việc ra quyết định. (MoSCoW: SHOULD)

### Yêu cầu Chức năng (FR)
| Mã FR | Tên chức năng | Mô tả chi tiết |
| :--- | :--- | :--- |
| FR-01 | Extract | Hệ thống cho phép đọc và trích xuất dữ liệu đơn hàng từ 24 file Excel nguồn vào bảng tạm Staging. |
| FR-02 | Data Cleaning | Hệ thống thực hiện phát hiện, ép kiểu và xử lý lỗi dữ liệu (ngày sai định dạng, số lượng không hợp lệ, thành tiền lệch). |
| FR-03 | Standardization | Hệ thống chuẩn hóa mã cửa hàng, mã sản phẩm, định dạng số điện thoại khách hàng theo quy chuẩn chung. |
| FR-04 | DW Loading | Hệ thống thực hiện biến đổi và nạp (Load) dữ liệu vào bảng `fact_sales` và các bảng `dim_store`, `dim_product`, `dim_customer`, `dim_date`. |
| FR-05 | Time Analysis | Hệ thống cung cấp khả năng truy vấn và hiển thị tổng doanh thu, số đơn hàng theo Ngày/Tháng/Quý/Năm. |
| FR-06 | Store/Product Analysis | Hệ thống hỗ trợ tổng hợp doanh thu theo từng Cửa hàng, Nhóm sản phẩm và top Sản phẩm bán chạy. |
| FR-07 | Dashboard Visualization | Màn hình Dashboard hiển thị biểu đồ xu hướng doanh thu, biểu đồ tỷ trọng và bộ lọc đa chiều. |

## 4. Yêu cầu phi chức năng (Non-Functional Requirements - NFR)
- **NFR-01 [Hiệu năng ETL]:** Thời gian thực hiện ETL trích xuất và làm sạch 26.000 bản ghi đơn hàng từ 24 file Excel phải hoàn tất dưới 30 giây.
- **NFR-02 [Hiệu năng Truy vấn]:** Thời gian phản hồi của các truy vấn báo cáo tổng hợp doanh thu trên kho dữ liệu phải dưới 1.5 giây với cơ sở dữ liệu 100.000 bản ghi.
- **NFR-03 [Độ chính xác dữ liệu]:** Tỷ lệ dữ liệu sạch nạp thành công vào bảng `fact_sales` đạt 100% tính đúng đắn theo quy tắc (Thành tiền = Số lượng x Đơn giá).
- **NFR-04 [Dung lượng & Lưu trữ]:** Cơ sở dữ liệu Data Warehouse hỗ trợ lưu trữ tối thiểu 5.000.000 dòng bản ghi mà không làm suy giảm thời gian phản hồi quá 20%.

## 5. Ràng buộc và quy tắc nghiệp vụ (Business Rules)
- **QT-01:** Số điện thoại khách hàng phải chuẩn hóa về dạng 10 chữ số bắt đầu bằng 0 (chuyển đổi các dạng +84, 84, khoảng trắng).
- **QT-02:** Thành tiền của mỗi dòng sản phẩm trong đơn hàng bắt buộc phải bằng Số lượng x Đơn giá tại thời điểm bán. Nếu dữ liệu Excel bị lệch, hệ thống tự động tính lại theo công thức chuẩn.
- **QT-03:** Mỗi dòng dữ liệu nạp vào bảng `fact_sales` phải tham chiếu hợp lệ đến khóa chính của các bảng Dimension (`dim_store`, `dim_product`, `dim_customer`, `dim_date`). Nếu không tìm thấy, phải gán vào giá trị mặc định "Unknown/Chưa xác định".
- **QT-04:** Không xóa vật lý dữ liệu giao dịch bán hàng (soft delete hoặc lưu vết lịch sử nạp).

## 6. Bảng truy vết yêu cầu (Traceability Matrix)
| Mã FR | User Story | Use Case liên quan | Độ ưu tiên (MoSCoW) |
| :--- | :--- | :--- | :--- |
| FR-01 | US01 | UC-01 | MUST |
| FR-02 | US02 | UC-01 | MUST |
| FR-03 | US03 | UC-01 | MUST |
| FR-04 | US04 | UC-02 | MUST |
| FR-05 | US05 | UC-03 | MUST |
| FR-06 | US06 | UC-03 | SHOULD |
| FR-07 | US07 | UC-04 | SHOULD |
