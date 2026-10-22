# TỪ ĐIỂN DỮ LIỆU – SMART CRM / MEKONG MOBILE

> **Phạm vi:** luồng L6 – kho dữ liệu bán hàng và báo cáo doanh thu. PostgreSQL Star Schema.
> **Cơ sở:** Bài tập 1 (12/10/2026), Thành phần 4, trang 7–8 và quy tắc tại trang 4–5.

## 1. Quy ước

- **Grain:** 1 dòng `fact_sales` = 1 dòng mặt hàng của 1 đơn hàng, tại 1 cửa hàng, trong 1 ngày. Đây **không phải** mức 1 dòng/đơn.
- **PK:** khóa chính; **FK:** khóa ngoại; **NK:** khóa nghiệp vụ (đề xuất dùng để đối chiếu dữ liệu nguồn).
- **Kiểu PostgreSQL, độ dài và quy tắc NULL:** đề xuất thiết kế để có thể triển khai; các tên bảng/cột lấy đúng từ tài liệu.
- **Unknown:** `store_id = 0`, `product_id = 0`, `customer_id = 0`, `date_key = 0` dành cho bản ghi không xác định, theo QT-03; với thiếu **mã đơn / ngày bán**, QT ngoại lệ yêu cầu đẩy vào `stg_order_errors`, không nạp Fact.
- **Tiền tệ:** giả định VND với 2 chữ số thập phân; nếu dữ liệu đầu vào có loại tiền khác thì cần bổ sung trường tiền tệ trước ETL.

## fact_sales

| Cột | Kiểu dữ liệu | Khóa | Ràng buộc / định dạng | Mô tả |
|---|---|---|---|---|
| `sales_fact_id` | `BIGINT` | PK | NOT NULL, IDENTITY | Khóa thay thế duy nhất cho từng dòng chi tiết bán hàng. |
| `order_code` | `VARCHAR(50)` | — | NOT NULL | Mã đơn hàng nguồn; một đơn có thể gồm nhiều dòng sản phẩm. |
| `date_key` | `INTEGER` | FK → dim_date.date_key | NOT NULL | Khóa ngày bán, đề xuất dạng YYYYMMDD; 0 dành cho Unknown. |
| `store_id` | `INTEGER` | FK → dim_store.store_id | NOT NULL | Tham chiếu cửa hàng phát sinh đơn hàng. |
| `product_id` | `INTEGER` | FK → dim_product.product_id | NOT NULL | Tham chiếu sản phẩm được bán. |
| `customer_id` | `INTEGER` | FK → dim_customer.customer_id | NOT NULL | Tham chiếu khách hàng; 0 khi không xác định. |
| `quantity` | `INTEGER` | — | NOT NULL, > 0 | Số lượng sản phẩm bán của dòng đơn hàng. |
| `unit_price` | `NUMERIC(18,2)` | — | NOT NULL, >= 0 | Đơn giá tại thời điểm bán (VND); không lấy giá hiện tại của sản phẩm. |
| `total_amount` | `NUMERIC(20,2)` | — | NOT NULL | Thành tiền (VND) = quantity × unit_price; sai lệch được tính lại trong ETL. |

## dim_store

| Cột | Kiểu dữ liệu | Khóa | Ràng buộc / định dạng | Mô tả |
|---|---|---|---|---|
| `store_id` | `INTEGER` | PK | NOT NULL, IDENTITY | Khóa thay thế cho cửa hàng; 0 = Unknown. |
| `store_code` | `VARCHAR(20)` | NK (đề xuất) | UNIQUE | Mã cửa hàng được chuẩn hóa giữa 24 file Excel. |
| `store_name` | `VARCHAR(150)` | — | NOT NULL | Tên hiển thị của cửa hàng. |
| `city` | `VARCHAR(100)` | — | Có thể NULL | Thành phố/khu vực phân tích doanh thu. |

## dim_product

| Cột | Kiểu dữ liệu | Khóa | Ràng buộc / định dạng | Mô tả |
|---|---|---|---|---|
| `product_id` | `INTEGER` | PK | NOT NULL, IDENTITY | Khóa thay thế cho sản phẩm; 0 = Unknown. |
| `product_code` | `VARCHAR(50)` | NK (đề xuất) | UNIQUE | Mã sản phẩm được chuẩn hóa từ dữ liệu nguồn. |
| `product_name` | `VARCHAR(200)` | — | NOT NULL | Tên sản phẩm. |
| `category` | `VARCHAR(100)` | — | Có thể NULL | Nhóm/danh mục sản phẩm để tổng hợp và xếp hạng. |

## dim_customer

| Cột | Kiểu dữ liệu | Khóa | Ràng buộc / định dạng | Mô tả |
|---|---|---|---|---|
| `customer_id` | `INTEGER` | PK | NOT NULL, IDENTITY | Khóa thay thế cho khách hàng; 0 = Unknown. |
| `customer_name` | `VARCHAR(150)` | — | Có thể NULL | Tên khách hàng theo dữ liệu bán hàng. |
| `phone` | `VARCHAR(10)` | — | NULL hoặc 10 chữ số | SĐT chuẩn hóa 0xxxxxxxxx từ +84/84/khoảng trắng. |

## dim_date

| Cột | Kiểu dữ liệu | Khóa | Ràng buộc / định dạng | Mô tả |
|---|---|---|---|---|
| `date_key` | `INTEGER` | PK | NOT NULL | Mã ngày theo dạng YYYYMMDD, ví dụ 20261022; 0 = Unknown. |
| `full_date` | `DATE` | — | UNIQUE; NULL ở Unknown | Ngày bán chuẩn ISO YYYY-MM-DD. |
| `day_of_week` | `SMALLINT` | — | 1–7 hoặc NULL Unknown | Thứ trong tuần theo ISO: Thứ Hai=1, Chủ nhật=7. |
| `month` | `SMALLINT` | — | 1–12 hoặc NULL Unknown | Tháng của ngày bán. |
| `quarter` | `SMALLINT` | — | 1–4 hoặc NULL Unknown | Quý của ngày bán. |
| `year` | `SMALLINT` | — | Có thể NULL Unknown | Năm dương lịch của ngày bán. |

## 2. Liên kết PK/FK

| Bảng con | Khóa ngoại | Bảng cha | Khóa chính | Quan hệ |
|---|---|---|---|---|
| fact_sales | date_key | dim_date | date_key | N:1 |
| fact_sales | store_id | dim_store | store_id | N:1 |
| fact_sales | product_id | dim_product | product_id | N:1 |
| fact_sales | customer_id | dim_customer | customer_id | N:1 |

## 3. Quy tắc ETL và chất lượng dữ liệu

| Mã | Trường / bước | Quy tắc thực thi | Kết quả lỗi |
|---|---|---|---|
| QT-01 | dim_customer.phone | Bỏ khoảng trắng/ký tự phân tách, chuyển +84/84 thành 0; kiểm tra 10 số và bắt đầu 0 | Không hợp lệ → ghi lỗi/để NULL theo quyết định nghiệp vụ; không tự tạo SĐT |
| QT-02 | fact_sales.total_amount | Tính lại bằng quantity × unit_price đã ép kiểu | Sai lệch nguồn → ghi log và dùng số được tính |
| QT-03 | Tất cả FK của fact_sales | Tra cứu dimension, không có bản ghi → dùng khóa Unknown=0 | Không phát sinh FK mồ côi |
| QT-04 | Giao dịch đã nạp | Không xóa vật lý, lưu vết nạp / soft delete khi triển khai | Không mất lịch sử giao dịch |
| UC-01 ngoại lệ 3a | 24 file Excel | File không đọc được → ghi etl_error.log và tiếp tục các file khác | Cảnh báo Data Analyst |
| UC-01 ngoại lệ 4a | order_code, ngày bán | Trường bắt buộc trống → chuyển stg_order_errors | Không nạp fact_sales |
| Kiểm tra bổ sung | quantity, unit_price | quantity > 0, unit_price >= 0; giá trị số hợp lệ | Cách ly dòng để kiểm tra; không tự bịa số |

## 4. Các điểm cần xác nhận trước ETL

1. **Mã duy nhất của dòng hàng:** tài liệu chưa có `order_line_id`/`line_number`. Hai dòng cùng `order_code` + `product_id` có thể hợp lệ; không tự áp unique trên cặp này. Phải xác định cách nhận diện bản ghi và nạp lại idempotent.
2. **Khách hàng:** tài liệu chỉ có `customer_name` và `phone`; chưa chốt khóa nghiệp vụ khách hàng khi SĐT thiếu/trùng. Không áp UNIQUE trên SĐT nếu chưa làm rõ.
3. **Lịch sử thay đổi:** tài liệu chưa quyết định dùng SCD Type 1 hay Type 2 cho thông tin cửa hàng/sản phẩm/khách hàng. Bộ cột đang phù hợp cập nhật Type 1; Type 2 cần thêm effective dates và cờ hiện hành.
4. **Doanh thu thực nhận:** chưa có hoàn trả, giảm giá, thuế, tình trạng đơn. `total_amount` hiện là thành tiền gộp của dòng hàng, không phải doanh thu ròng.
5. **Thời gian:** chỉ có `date_key`, phân tích theo ngày trở lên; chưa có giờ phút giao dịch.

## 5. Danh mục file

- `01_star_schema.drawio`: bảng Fact–Dimension và liên kết PK/FK.
- `02_etl_architecture.drawio`: kiến trúc Batch ETL, staging, ngoại lệ và đích DW.
- `03_use_case.drawio`: UC-01…UC-04 và 3 actor.
- `04_dashboard_wireframes.drawio`: 3 màn hình dashboard theo đề bài.
- `warehouse_schema.sql`: DDL PostgreSQL đề xuất kèm bản ghi Unknown.
- `data_dictionary.xlsx`: 5 trang dữ liệu, một trang quy ước.
