- Tự động quản lý cách lưu trữ object trong bucket
- Có thể cấu hình lifecycle rule để tự động chuyển các file từ lớp lưu trữ đắt tiền sang rẻ tiền **sau một khoảng thời gian sử dụng / điều kiện**
## Vì sao dùng?
--> Lưu trữ logs hoặc report mà ít truy cập --> tiết kiệm chi phí qua S3 Standard-IA hoặc S3 Glacier

## Cấu hình Lifecycle Rule
Cấu trúc
1. Transition: Di chuyển object sang lớp lưu trữ khác sau 1 khoảng thời gian không truy cập
2. Expiration: Xoá object tự động sau một khoảng thời gian (ví dụ 1 năm)
Ví dụ
* Standard --> Standard-IA sau 30 ngày không truy cập
* Standard-IA --> Glacier sau 90 ngày không truy cập
* Xoá object sau 365 ngày