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

---

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

---

#### 2.1. Một số keyword chính trong docker-compose

| Keyword	| Ý nghĩa |
| --- | --- |
| version	| Phiên bản cú pháp Docker Compose |
| services	| Danh sách các container cần chạy trong hệ thống |
| networks | Định nghĩa các mạng ảo dùng chung giữa các service |
| volumes	| Định nghĩa các ổ đĩa ảo dùng chung giữa các service |

---

#### 2.2. Các keyword mô tả service
`image`

Chỉ định image có sẵn (từ Docker Hub hoặc local) để chạy container.

```
services:
  database:
    image: mariadb:latest   # dùng image mariadb phiên bản mới nhất
```

`build`

Thay vì dùng image có sẵn, tự build image từ Dockerfile.

```
services:
  flask_api:
    build:
      context: ./flask_app    # thư mục chứa Dockerfile
      dockerfile: Dockerfile  # tên file (mặc định là "Dockerfile")
```

`container_name`

Đặt tên cụ thể cho container thay vì để Docker tự sinh tên ngẫu nhiên.

```
services:
  web:
    image: nginx
    container_name: my_nginx
```

`ports`

Map cổng theo cú pháp HOST:CONTAINER — cho phép truy cập từ bên ngoài máy host.

```
services:
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"   # truy cập http://localhost:3000 → grafana bên trong container
```

`environment`

Truyền biến môi trường vào trong container

```
services:
  mariadb:
    image: mariadb:latest
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: alert_db
      MYSQL_USER: admin
      MYSQL_PASSWORD: admin123
```

`env_file`

Đọc biến môi trường từ file .env bên ngoài — tránh lộ thông tin nhạy cảm trong compose file.

```
services:
  mariadb:
    env_file:
      - .env
```

`volumes`

Mount dữ liệu giữa host và container. Có 2 loại:

- Named volume: Docker tự quản lý vị trí lưu trữ.
- Bind mount: Mount trực tiếp thư mục/file từ máy host.

```
services:
  mariadb:
    volumes:
      - mariadb_data:/var/lib/mysql        # named volume
      - ./config/my.cnf:/etc/mysql/my.cnf  # bind mount (file cụ thể)
```

`networks`

Chỉ định container tham gia vào mạng nào. Một container có thể thuộc nhiều mạng.

```
services:
  flask_api:
    networks:
      - frontend_net
      - backend_net
```

`depend_on`

Xác định thứ tự khởi động — service này chỉ start sau khi các service phụ thuộc đã sẵn sàng.

```
services:
  flask_api:
    depends_on:
      - mariadb    # mariadb phải khởi động trước flask_api
      - influxdb
```

`restart`

Chính sách tự khởi động lại container khi bị crash hoặc khi Docker daemon restart

```
services:
  nodered:
    restart: always          # luôn luôn restart
    # restart: unless-stopped  # restart trừ khi bị dừng thủ công bằng docker stop
    # restart: on-failure      # chỉ restart khi thoát với mã lỗi khác 0
    # restart: no              # không bao giờ tự restart
```

`command`

Ghi đè lệnh mặc định (CMD) được định nghĩa trong image khi container khởi động.

```
services:
  flask_api:
    command: python app.py --port 5000
```

`healthcheck`

Định nghĩa câu lệnh kiểm tra định kỳ xem service có hoạt động đúng không.

```
services:
  mariadb:
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s   # kiểm tra mỗi 10 giây
      timeout: 5s     # timeout sau 5 giây
      retries: 5      # thử lại 5 lần trước khi đánh dấu "unhealthy"
```

`expose`

Mở cổng cho các container khác trong cùng network — không mở ra ngoài máy host.

```
services:
  flask_api:
    expose:
      - "5000"   # container khác trong cùng network có thể gọi :5000
                 # nhưng máy host bên ngoài không truy cập được
```

`entrypoint`

Ghi đè lệnh ENTRYPOINT mặc định của image.

```
services:
  flask_api:
    entrypoint: ["python", "-m", "flask", "run"]
```

`working_dir`

Đặt thư mục làm việc mặc định bên trong container.

```
services:
  flask_api:
    working_dir: /app
```

---

#### 2.3. Các keyword mô tả network

| Driver	| Ý nghĩa |
| --- | --- |
| bridge	| Mạng ảo riêng — các container trong cùng bridge giao tiếp được với nhau qua tên service |
| host	| Container dùng thẳng network interface của máy host (không cô lập network) |
| none	| Container không có kết nối mạng |

---

#### 2.4. Các keyword mô tả volumes

```
volumes:
  mariadb_data:       # Docker tự quản lý vị trí lưu trữ
    driver: local

  influxdb_data:
    driver: local

  grafana_data:
    driver: local
```
Named volume được lưu tại /var/lib/docker/volumes/ trên máy host. Dữ liệu tồn tại độc lập với vòng đời container

---

### 3. Ưu điểm khi triển khai ứng dụng bằng Docker

| STT |	Ưu điểm	| Giải thích |
| --- | --- | --- |
| 1 |	Nhất quán môi trường	| Dev, test, staging, production chạy giống hệt nhau. Xoá bỏ vấn đề "chạy được trên máy tôi".|
| 2	| Cô lập (Isolation)	| Mỗi service chạy trong container riêng, không ảnh hưởng lẫn nhau. Lỗi ở một service không kéo đổ cả hệ thống.|
| 3 |	Triển khai nhanh chóng	| Chỉ cần một lệnh docker compose up -d để khởi động toàn bộ hệ thống nhiều service.|
| 4	| Dễ dàng scale	| Tăng số lượng instance của một service nhanh chóng: docker compose scale web=3 | 
| 5	| Dễ rollback	| Quay về phiên bản cũ đơn giản bằng cách đổi tag image và restart.|
| 6	| Tiết kiệm tài nguyên	| Container nhẹ hơn nhiều so với Virtual Machine vì dùng chung kernel với máy host, không cần boot OS riêng. |
| 7 |	Quản lý dependencies	| Mỗi container tự mang dependencies riêng — không bao giờ xảy ra xung đột phiên bản giữa các service. |
| 8	| Dễ backup và restore	| Export/import image hoặc volume chỉ với vài lệnh đơn giản. |
| 9	| CI/CD thân thiện	| Tích hợp dễ dàng vào các pipeline tự động hoá (GitHub Actions, GitLab CI,...).|
| 10	| Tái sử dụng |	Image được build một lần, có thể chạy ở bất kỳ đâu có Docker mà không cần cấu hình lại. |

---

### 4. Triển khai lên máy chủ thật không có internet
#### Bước 1: Trên Laptop : Build và Pull tất cả các images
```
# Pull tất cả images được khai báo trong docker-compose.yml
docker compose pull

# Build các image tự viết (nếu có dùng keyword "build:" trong compose file)
docker compose build
```
#### Bước 2: Trên Laptop: Export images ra file nén

```
# Export từng image riêng lẻ
docker save mariadb:10.11      -o mariadb.tar
docker save influxdb:2.7       -o influxdb.tar
docker save grafana/grafana    -o grafana.tar
docker save nodered/node-red   -o nodered.tar
docker save nginx:alpine       -o nginx.tar
docker save my_flask_api:latest -o flask_api.tar

# Export tất cả vào 1 file nén duy nhất (khuyến nghị)
docker save \
  mariadb:10.11 \
  influxdb:2.7 \
  grafana/grafana \
  nodered/node-red \
  nginx:alpine \
  my_flask_api:latest \
  | gzip > all_images.tar.gz
```
#### Bước 3: Copy lên máy chủ

```
# Dùng SCP (truyền qua SSH)
scp all_images.tar.gz user@192.168.1.100:/home/user/

# Copy toàn bộ project (bao gồm docker-compose.yml, config, source code,...)
scp -r ./my_project user@192.168.1.100:/home/user/
```
Nếu không có SSH, có thể copy qua USB, ổ cứng ngoài, hoặc mạng nội bộ LAN.

#### Bước 4: Load images từ file

```
# Load images từ file nén
docker load -i all_images.tar.gz
# Hoặc:
gunzip -c all_images.tar.gz | docker load

# Kiểm tra images đã được load thành công
docker images
```

#### Bước 5:  Trên Máy chủ: Vào thư mục project

```
cd /home/user/my_project
ls -la
# Kiểm tra có đủ file: docker-compose.yml, .env, nginx.conf, frontend/, flask_api/,...
```
#### Bước 6: Khởi động hệ thống

```
# Chạy toàn bộ hệ thống ở chế độ nền (detached mode)
docker compose up -d

# Kiểm tra trạng thái các container
docker compose ps

# Xem log nếu có lỗi
docker compose logs -f
```

#### Một số lệnh quan trọng

`docker compose up -d`	Khởi động tất cả service ở chế độ nền

`docker compose down`	Dừng và xoá tất cả container (giữ volumes)

`docker compose ps`	Xem trạng thái các container

`docker compose logs -f`	Xem log realtime

`docker save IMAGE -o file.tar`	Export image ra file

`docker load -i file.tar`	Load image từ file

`docker images`	Liệt kê tất cả images hiện có

`docker compose pull`	Pull images từ registry

`docker compose build`	Build images từ Dockerfile

---

## PHẦN 2: THỰC HÀNH
