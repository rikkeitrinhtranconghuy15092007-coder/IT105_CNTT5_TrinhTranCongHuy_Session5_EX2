# Báo cáo Thiết kế: Phân hệ Đổi trả & Hoàn tiền (Refund)
**Dự án:** RikkeiShop

## 1. Định nghĩa Nghiệp vụ & Bẫy Dữ Liệu

*   **Quy tắc 1 (Thời hạn):** Khách hàng chỉ được phép tạo yêu cầu Đổi trả/Hoàn tiền trong vòng đúng 7 ngày kể từ thời điểm đơn hàng cập nhật trạng thái "Đã giao thành công".
*   **Quy tắc 2 (Bằng chứng):** Bắt buộc phải đính kèm tối thiểu 1 video mở hộp (unbox) không qua chỉnh sửa và tối đa 3 hình ảnh chụp rõ chi tiết lỗi của sản phẩm.
*   **Quy tắc 3 (Quy trình thu hồi):** Hệ thống tự động điều phối Shipper đến tận nhà thu hồi hàng trong 48h. Việc hoàn tiền chỉ kích hoạt sau khi Kho xác nhận hàng thu hồi hợp lệ.
*   **Edge Case (Gian lận tráo hàng):** Khách hàng gửi trả sản phẩm giả mạo (Serial Number/IMEI không khớp với dữ liệu xuất kho ban đầu). Hệ thống lập tức hủy yêu cầu hoàn tiền, cập nhật trạng thái "Blacklist" cho tài khoản để chặn các giao dịch tương lai, và gửi thông báo từ chối kèm hình ảnh bằng chứng cho khách hàng.

## 2. Sơ đồ Use Case Diagram Tổng thể

```mermaid
usecaseDiagram
    actor Khách_Hàng as "Khách hàng"
    actor CSKH as "Nhân viên CSKH"
    actor Kho as "Kiểm định Kho"
    actor Shipper as "Shipper"
    actor Cổng_Thanh_Toán as "Cổng Thanh Toán"

    usecase UC1 as "Tạo yêu cầu Đổi trả"
    usecase UC2 as "Tải lên bằng chứng (Video/Ảnh)"
    usecase UC3 as "Duyệt yêu cầu sơ bộ"
    usecase UC4 as "Điều phối lấy hàng"
    usecase UC5 as "Kiểm định hàng trả về"
    usecase UC6 as "Xử lý hoàn tiền (Refund)"
    usecase UC7 as "Khóa tài khoản (Blacklist)"

    Khách_Hàng --> UC1
    UC1 ..> UC2 : <<extend>>
    
    CSKH --> UC3
    UC3 ..> UC4 : <<include>> (Nếu duyệt)
    Shipper --> UC4
    
    Kho --> UC5
    UC5 ..> UC6 : <<include>> (Nếu hợp lệ)
    UC5 ..> UC7 : <<extend>> (Nếu tráo hàng)
    
    Cổng_Thanh_Toán --> UC6
```

## 3. Sơ đồ Activity Diagram (Luồng Cốt Lõi)

```mermaid
activediagram
|Khách Hàng|
start
:Bấm "Yêu cầu trả hàng";
:Điền lý do & Tải video/ảnh;

|Nhân viên CSKH|
:Kiểm tra bằng chứng sơ bộ;
if (Hợp lệ?) then (Không)
  :Từ chối yêu cầu;
  |Khách Hàng|
  :Nhận thông báo từ chối;
  stop
else (Có)
  :Duyệt & Tạo mã vận đơn thu hồi;
  
  |Shipper|
  :Đến địa chỉ khách thu hồi hàng;
  :Giao hàng hoàn về Kho;
  
  |Kiểm định Kho|
  :Kiểm tra tình trạng & Serial Number;
  if (Khớp dữ liệu xuất kho?) then (Không - Gian lận)
    :Đánh dấu "Tráo hàng";
    |Hệ thống|
    :Khóa tài khoản (Blacklist);
    :Gửi email từ chối hoàn tiền;
    stop
  else (Có - Hợp lệ)
    :Xác nhận nhập kho thành công;
    
    |Hệ thống|
    :Kích hoạt API Hoàn tiền;
    :Cập nhật trạng thái "Đã hoàn tiền";
    
    |Khách Hàng|
    :Nhận tiền & Thông báo thành công;
    stop
  endif
endif
```

## 4. Hướng dẫn Triển khai & Nộp bài

*   Sao chép trực tiếp nội dung file Markdown này lên kho lưu trữ Github của bạn; Github sẽ tự động biên dịch mã Mermaid thành hình ảnh sơ đồ chuyên nghiệp.
*   Nếu bắt buộc dùng Draw.io, bạn có thể copy đoạn code Mermaid trên, vào Draw.io chọn **Arrange > Insert > Advanced > Mermaid** để phần mềm tự động vẽ ra các khối hình, sau đó tinh chỉnh lại màu sắc.
