#  APP MONITOR + ALERT DATA REALTIME
- Môn học: Phát triển ứng dụng với mã nguồn mở - TEE0421
- Họ và tên: Nguyễn Trung Hiếu
- MSSV: K225480106019

---

# YÊU CẦU BÀI TẬP
## LÝ THUYẾT
```
+ docker là gì? 
+ các keyword được sử dụng trong docker-compose.yml
  để mô tả 1 service, network, volume,...
  liệt kê + ý nghĩa của từ khoá đó + ví dụ minh hoạ
+ ưu điểm khi triển app sử dụng docker là gì?
+ dùng docker: tạo app, test app OK trên laptop cá nhân
  giờ muốn triển khai app này trên máy chủ thật ko có internet
  thì các bước cần làm là?
```
## THỰC HÀNH ÁP DỤNG
```
sử dụng docker compose có nhiều serivce 
và các thành phần cần thiết để tạo thành ứng dụng:
 + nodered liên tục lấy dữ liệu từ nguồn nào đó (chứng khoán, thời tiết, giá vàng,...)
   nguồn thực tế, số liệu luôn động sau thời gian ngắn
 + nodered lưu trữ dữ liệu vào 2 database: mariadb để lưu giá trị tức thời
   lưu lịch sử vào influxdb
 + sử dụng grafana để trực quan hoá dữ liệu: vẽ biểu đồ
 + sử dụng nginx để làm webserver
   chạy 1 trang web html+js+css làm front-end
   js: lấy dữ liệu tức thời trong mariadb qua (ajax | socket) 
       gọi api (api tự build bằng Flask giống bt1)
       api trả về giá trị tức thời trong mariadb
       hiển thị lên web, auto hiển thị số mới khi thay đổi
   sử dụng iframe để gọi grafana
   hiển thị biểu đồ dữ liệu lịch sử của thông số đã lưu
 + QUAN SÁT DỮ LIỆU LỊCH SỬ => GIÁ TRỊ BẤT THƯỜNG
   (VD MIỀN A..B: OK, DƯỚI A: ALERT LOW, TRÊN B: ALERT HIGH)
 + nodered: kết hợp bot Telegram
   khi dữ liệu not OK, thì gửi tin nhắn từ bot => group trên telegram
   group đã add bot vào: (nhóm đã có 2 người), add thêm 1875746636 thành 3 người
   mỗi khi bot gửi dữ liệu vào nhóm: mọi member of group đều nhận đc
   nội dung alert: tường minh, có value gây alert

 xuất tất cả các container ra file nén.
 xoá mọi container đang chạy
 load lại các container  từ file nén để khôi phục các container đã xoá
```
---

# BÀI LÀM
## PHẦN 1: LÝ THUYẾT
### 1. Docker là gì?
Docker là một nền tảng mã nguồn mở giúp đóng gói ứng dụng và toàn bộ môi trường chạy của nó (thư viện, cấu hình, dependencies) vào một đơn vị gọi là Container.

Trong phát triển phần mềm truyền thống, thường xảy ra tình huống: ứng dụng chạy tốt trên máy lập trình viên nhưng lại lỗi khi triển khai lên server — do khác biệt về phiên bản thư viện, hệ điều hành, hoặc cấu hình môi trường. Docker giải quyết vấn đề này bằng cách đóng gói toàn bộ môi trường vào container, đảm bảo ứng dụng chạy giống hệt nhau trên mọi máy.

Một số khái niệm chính:
| Khái niệm | Mô tả |
| --- | --- |
| image | Bản thiết kế (blueprint) của container — chỉ đọc, không sửa được. Giống như class trong OOP. |
| container | Instance đang chạy được tạo từ image. Giống như object được khởi tạo từ class.|
| Dockerfile | 	File hướng dẫn từng bước để build một image tùy chỉnh. |
| Docker Hub | Kho lưu trữ image công khai trực tuyến (tương tự GitHub nhưng dành cho image). |
| volume | Cơ chế lưu trữ dữ liệu bền vững bên ngoài container, không bị mất khi container bị xoá |
| network | Mạng ảo để các container giao tiếp với nhau một cách an toàn và có kiểm soát. |
| docker compose | Công cụ định nghĩa và chạy nhiều container cùng lúc qua file docker-compose.yml. |

### 2. Các keyword trong file `docker-compose.yml`
File docker-compose.yml là file cấu hình YAML dùng để mô tả toàn bộ hệ thống multi-container. 

Cấu trúc tổng quát:
```
version: '3.8'      # phiên bản cú pháp Compose

services:           # định nghĩa các container
  ten_service:
    ...

networks:           # định nghĩa mạng ảo
  ten_network:
    ...

volumes:            # định nghĩa ổ đĩa ảo
  ten_volume:
    ...
```
