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
### 1. Tổng quan hệ thống
#### 1.1. Giới thiệu 
Dự án xây dựng một hệ thống giám sát và cảnh báo tự động theo thời gian thực (App Monitor + Alert Data Realtime) chạy trên nền tảng Docker Container. Đối tượng giám sát thực tế được lựa chọn ở đây là dữ liệu thời tiết thu thập trực tiếp thông qua API.

Các thành phần cốt lõi trong hệ thống:
- Node-RED: Đóng vai trò là trung tâm điều phối dữ liệu (ETL Workflow). Thực hiện tác vụ lấy dữ liệu động liên tục, phân tích dị thường, ghi đồng thời vào 2 cơ sở dữ liệu và kích hoạt bot Telegram gửi cảnh báo khi có sự cố về giá.
- MariaDB: Cơ sở dữ liệu quan hệ (RDBMS) dùng để lưu trữ trạng thái tức thời (giá trị mới nhất) nhằm phục vụ các truy vấn nhanh của ứng dụng Web Client.
- InfluxDB (v1.8): Cơ sở dữ liệu chuỗi thời gian (Time-series Database) tối ưu cho việc lưu trữ dữ liệu lịch sử, phục vụ vẽ biểu đồ phân tích xu hướng theo thời gian.
- Flask API (Python): Xây dựng dịch vụ API nội bộ kết nối trực tiếp với MariaDB, cung cấp endpoint endpoint trả về dữ liệu giá mới nhất dạng JSON cho giao diện người dùng.
- Nginx: Web Server phân phối giao diện Frontend tĩnh và đồng thời đóng vai trò làm Reverse Proxy điều hướng luồng request từ trình duyệt client sang Flask API một cách an toàn thông qua cấu hình mạng nội bộ Docker.
- Grafana: Nền tảng trực quan hóa dữ liệu mạnh mẽ, kết nối với InfluxDB để vẽ biểu đồ kỹ thuật và cho phép nhúng trực tiếp vào giao diện Frontend qua thẻ Iframe không cần đăng nhập.

---

#### 1.2. Cấu trúc thư mục dự án

```
weather-monitor/
├── docker-compose.yml
├── README.md
├── nginx/
│   ├── default.conf
│   └── html/
│       ├── index.html
├── flask_api/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py
├── nodered_data/          
├── influxdb_data/         
└── mariadb_data/          
```

---

#### 1.3. Danh sách service và cổng

| Tên Service (Docker) | Cổng Nội bộ (Container) | Cổng Công khai (Host) | Mục đích sử dụng |
| :--- | :--- | :--- | :--- |
| `web_nginx` | `80` | **`8085`** | Giao diện Frontend (HTML/JS) & Reverse Proxy |
| `weather_flask` | `5000` | Không mở (Ẩn) | Cung cấp API lấy dữ liệu tức thời từ MariaDB |
| `weather_nodered` | `1880` | **`1882`** | Thu thập dữ liệu thời tiết, lưu DB, Gửi Telegram |
| `weather_mariadb` | `3306` | **`3309`** | Lưu trữ dữ liệu thời tiết tức thời (Latest) |
| `weather_influxdb` | `8086` | **`8089`** | Lưu trữ dữ liệu lịch sử (Time-series) |
| `weather_grafana` | `3000` | **`3002`** | Trực quan hóa biểu đồ lịch sử (Nhúng Iframe) |

> **Lưu ý mạng nội bộ:** Các service giao tiếp với nhau bằng tên service bên trong mạng chung `weather_network` (Ví dụ: Flask kết nối tới DB qua host là `weather_mariadb` chứ không dùng IP).

---

### 2. Cấu hình 
#### 2.1. Cấu hình file `docker-compose.yml`
```
# version: '3.8'

networks:
  weather_network:
    driver: bridge

services:
  # 1. Database Lưu Tức Thời
  weather_mariadb:
    image: mariadb:10.11
    container_name: weather_mariadb
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: weather_db
      MYSQL_USER: weather_user
      MYSQL_PASSWORD: weather_password
    ports:
      - "3309:3306" 
    volumes:
      - ./mariadb_data:/var/lib/mysql
    networks:
      - weather_network
    restart: always

  # 2. Database Lưu Lịch Sử
  weather_influxdb:
    image: influxdb:1.8
    container_name: weather_influxdb
    environment:
      - INFLUXDB_DB=weather_history
      - INFLUXDB_ADMIN_USER=admin
      - INFLUXDB_ADMIN_PASSWORD=adminpassword
    ports:
      - "8089:8086"
    volumes:
      - ./influxdb_data:/var/lib/influxdb
    networks:
      - weather_network
    restart: always

  # 3. Node-RED (Trung tâm xử lý ETL)
  weather_nodered:
    image: nodered/node-red:latest
    container_name: weather_nodered
    ports:
      - "1882:1880"
    volumes:
      - ./nodered_data:/data
    networks:
      - weather_network
    restart: always

  # 4. Grafana (Vẽ biểu đồ và cho phép nhúng Iframe)
  weather_grafana:
    image: grafana/grafana:latest
    container_name: weather_grafana
    ports:
      - "3002:3000"
    environment:
      - GF_SECURITY_ALLOW_EMBEDDING=true # Cho phép nhúng vào Iframe HTML
      - GF_AUTH_ANONYMOUS_ENABLED=true   # Cho phép xem biểu đồ không cần login
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Viewer
    volumes:
      - ./grafana_data:/var/lib/grafana
    networks:
      - weather_network
    restart: always
    depends_on:
      - weather_influxdb

  # 5. Flask API (Sẽ build từ code ở Giai đoạn sau)
  weather_flask:
    build: ./flask_api
    container_name: weather_flask
    # Không cần export ports ra ngoài vì Nginx sẽ gọi nó từ bên trong mạng Docker
    networks:
      - weather_network
    depends_on:
      - weather_mariadb
    restart: always

  # 6. Nginx Web Server & Reverse Proxy
  web_nginx:
    image: nginx:latest
    container_name: web_nginx_weather
    ports:
      - "8085:80" 
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./nginx/html:/usr/share/nginx/html
    networks:
      - weather_network
    depends_on:
      - weather_flask
    restart: always
```

---

#### 2.2. Xây dựng Flask API
##### Bước 1: Khai báo các thư viện cần thiết trong `flask_api/requirements.txt`
Khai báo các thư viện Python cần thiết để kết nối MariaDB và chạy API.

```
Flask==3.0.3
mysql-connector-python==8.3.0
Flask-Cors==4.0.1
```
---

##### Bước 2: Edit file `app.py`
Đoạn code này sẽ tạo một API endpoint `/api/weather` để kết nối vào MariaDB (sử dụng tên service weather_mariadb làm host) và lấy ra bản ghi thời tiết mới nhất.
```
from flask import Flask, jsonify
from flask_cors import CORS
import mysql.connector as mysql
import os
import time

app = Flask(__name__)
CORS(app) # Cho phép gọi API từ bên ngoài nếu cần

def get_db_connection():
    # Thử kết nối lại nhiều lần đề phòng MariaDB khởi động chậm hơn Flask
    for i in range(5):
        try:
            conn = mysql.connect(
                host='weather_mariadb', # Tên service trong docker-compose
                user='weather_user',
                password='weather_password',
                database='weather_db'
            )
            return conn
        except mysql.Error:
            time.sleep(2)
    return None

# Tạo bảng tự động nếu chưa có (để hệ thống không bị lỗi khi mới start)
def init_db():
    conn = get_db_connection()
    if conn:
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS weather_live (
                id INT AUTO_INCREMENT PRIMARY KEY,
                temperature FLOAT,
                humidity FLOAT,
                rainfall FLOAT,
                wind_speed FLOAT,
                timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
            )
        """)
        # Chèn thử 1 dòng dữ liệu mẫu ban đầu nếu bảng trống
        cursor.execute("SELECT COUNT(*) FROM weather_live")
        if cursor.fetchone()[0] == 0:
            cursor.execute("""
                INSERT INTO weather_live (temperature, humidity, rainfall, wind_speed) 
                VALUES (27.5, 80.0, 0.0, 3.5)
            """)
        conn.commit()
        cursor.close()
        conn.close()

@app.route('/api/weather', methods=['GET'])
def get_weather():
    conn = get_db_connection()
    if not conn:
        return jsonify({"error": "Không thể kết nối cơ sở dữ liệu"}), 500
    
    cursor = conn.cursor(dictionary=True)
    # Lấy bản ghi mới nhất dựa vào timestamp hoặc ID
    cursor.execute("SELECT temperature, humidity, rainfall, wind_speed, timestamp FROM weather_live ORDER BY id DESC LIMIT 1")
    result = cursor.fetchone()
    
    cursor.close()
    conn.close()
    
    if result:
        # Định dạng lại thời gian thành chuỗi để JSON hóa
        result['timestamp'] = result['timestamp'].strftime('%Y-%m-%d %H:%M:%S')
        return jsonify(result)
    return jsonify({"message": "Chưa có dữ liệu"}), 404

if __name__ == '__main__':
    init_db()
    # Chạy ở port 5000 bên trong container
    app.run(host='0.0.0.0', port=5000)
```

---

##### Bước 3: Edit file `Dockerfile`
File này dùng để Docker đóng gói ứng dụng Flask thành một Image riêng.

```
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

---

#### 2.3. Cấu hình Nginx làm Web Server & Reverse Proxy
- File: `nginx/default.conf`
- File này cấu hình Nginx lắng nghe ở cổng 80 (nội bộ container). Nếu user vào trang chủ / thì trả về file HTML, nếu vào tuyến đường /api/ thì Nginx sẽ "bắn" request đó sang cho container Flask xử lý.

```
server {
    listen 80;
    server_name localhost;

    # 1. Định tuyến cho Frontend (HTML/JS/CSS tĩnh)
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ =404;
    }

    # 2. Định tuyến Reverse Proxy cho Flask API
    location /api/ {
        proxy_pass http://weather_flask:5000; # Tên service Flask và port nội bộ của nó
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
---

#### 2.4. Tạo file giao diện web
Viết file index.html làm frontend cho hệ thống.

```
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hệ thống Giám sát Thời tiết Hệ thống IoT</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
    
    <style>
        /* --- CSS STYLE TOÀN DIỆN --- */
        :root {
            --bg-color: #f0f2f5;
            --card-bg: #ffffff;
            --text-main: #1a1f36;
            --text-sub: #697386;
            --primary: #0061ff;
            --primary-light: #e0ecff;
            --success: #00cc66;
            --danger: #ff3333;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--bg-color);
            color: var(--text-main);
            padding: 30px 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .container {
            width: 100%;
            max-width: 1200px;
        }

        header {
            margin-bottom: 30px;
            text-align: center;
            width: 100%;
        }

        header h1 {
            font-size: 2.2rem;
            font-weight: 700;
            color: var(--text-main);
            letter-spacing: -0.5px;
        }

        header p {
            color: var(--text-sub);
            margin-top: 5px;
            font-size: 1rem;
        }

        /* Lưới hiển thị các thông số tức thời */
        .live-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
            gap: 20px;
            margin-bottom: 40px;
            width: 100%;
        }

        /* Thẻ Card cho từng thông số */
        .card {
            background: var(--card-bg);
            border-radius: 16px;
            padding: 24px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 2px 4px -1px rgba(0, 0, 0, 0.03);
            display: flex;
            flex-direction: column;
            position: relative;
            overflow: hidden;
            transition: transform 0.2s ease, box-shadow 0.2s ease;
        }

        .card:hover {
            transform: translateY(-4px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }

        /* Hiệu ứng đường kẻ màu ở đỉnh card */
        .card::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 4px;
            background: var(--primary);
        }
        .card.temp::before { background: #ff5e62; }
        .card.humidity::before { background: #00c6ff; }
        .card.rain::before { background: #0072ff; }
        .card.wind::before { background: #56ab2f; }

        .card-title {
            font-size: 0.9rem;
            font-weight: 600;
            text-transform: uppercase;
            color: var(--text-sub);
            letter-spacing: 0.5px;
            margin-bottom: 12px;
        }

        .card-value {
            font-size: 2.5rem;
            font-weight: 700;
            color: var(--text-main);
            line-height: 1;
            margin-bottom: 8px;
            display: flex;
            align-items: baseline;
        }

        .card-value span.unit {
            font-size: 1.2rem;
            font-weight: 500;
            color: var(--text-sub);
            margin-left: 4px;
        }

        /* Trạng thái cập nhật thời gian */
        .status-bar {
            background: var(--card-bg);
            padding: 12px 24px;
            border-radius: 30px;
            display: inline-flex;
            align-items: center;
            font-size: 0.85rem;
            color: var(--text-sub);
            box-shadow: 0 2px 4px rgba(0,0,0,0.02);
            margin-bottom: 30px;
        }

        .pulse-dot {
            width: 8px;
            height: 8px;
            background-color: var(--success);
            border-radius: 50%;
            margin-right: 10px;
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(0.9); opacity: 1; box-shadow: 0 0 0 0 rgba(0, 204, 102, 0.7); }
            70% { transform: scale(1); opacity: 1; box-shadow: 0 0 0 6px rgba(0, 204, 102, 0); }
            100% { transform: scale(0.9); opacity: 0; }
        }

        /* Khu vực nhúng đồ thị Grafana */
        .chart-section {
            background: var(--card-bg);
            border-radius: 16px;
            padding: 24px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
            width: 100%;
        }

        .chart-section h2 {
            font-size: 1.3rem;
            font-weight: 600;
            margin-bottom: 20px;
            color: var(--text-main);
        }

        .iframe-container {
            position: relative;
            width: 100%;
            padding-top: 45%; /* Tỷ lệ màn hình 16:9 hoặc xấp xỉ */
            overflow: hidden;
            border-radius: 8px;
            background: #f8f9fa;
            border: 1px solid #e3e8ee;
        }

        .iframe-container iframe {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            border: none;
        }
    </style>
</head>
<body>

    <div class="container">
        <header>
            <h1>WEATHER MONITORING DASHBOARD</h1>
            <p>Hệ thống giám sát và phân tích dữ liệu thời tiết thời gian thực</p>
        </header>

        <center>
            <div class="status-bar">
                <div class="pulse-dot"></div>
                Dữ liệu tức thời (MariaDB): Cập nhật tự động sau mỗi <span id="update-countdown" style="font-weight:bold; margin: 0 3px;">5</span> giây. Lúc: <span id="time-stamp" style="margin-left: 5px; font-weight: 600;">--:--:--</span>
            </div>
        </center>

        <section class="live-grid">
            <div class="card temp">
                <div class="card-title">Nhiệt độ</div>
                <div class="card-value"><span id="temp-val">--</span><span class="unit">°C</span></div>
            </div>

            <div class="card humidity">
                <div class="card-title">Độ ẩm</div>
                <div class="card-value"><span id="humid-val">--</span><span class="unit">%</span></div>
            </div>

            <div class="card rain">
                <div class="card-title">Lượng mưa</div>
                <div class="card-value"><span id="rain-val">--</span><span class="unit">mm</span></div>
            </div>

            <div class="card wind">
                <div class="card-title">Tốc độ gió</div>
                <div class="card-value"><span id="wind-val">--</span><span class="unit">m/s</span></div>
            </div>
        </section>

        <section class="chart-section">
            <h2>Phân tích Xu hướng Lịch sử (Grafana - InfluxDB)</h2>
            <div class="iframe-container">
                <iframe src="http://localhost:3002/d-solo/your-dashboard-id/weather-history?orgId=1&refresh=5s&panelId=1" width="100%" height="100%" frameborder="0"></iframe>
            </div>
        </section>
    </div>

    <script>
        async function fetchLiveWeather() {
            try {
                // Gọi API thông qua Reverse Proxy của Nginx
                const response = await fetch('/api/weather');
                if (response.ok) {
                    const data = await response.json();
                    
                    // Cập nhật giá trị vào các thẻ HTML tương ứng
                    document.getElementById('temp-val').innerText = data.temperature !== undefined ? data.temperature : '--';
                    document.getElementById('humid-val').innerText = data.humidity !== undefined ? data.humidity : '--';
                    document.getElementById('rain-val').innerText = data.rainfall !== undefined ? data.rainfall : '--';
                    document.getElementById('wind-val').innerText = data.wind_speed !== undefined ? data.wind_speed : '--';
                    
                    // Cập nhật nhãn thời gian hiển thị
                    document.getElementById('time-stamp').innerText = data.timestamp || '--';
                } else {
                    console.error("API trả về lỗi hoặc chưa có dữ liệu.");
                }
            } catch (error) {
                console.error("Lỗi kết nối tới API endpoint:", error);
            }
        }

        // Gọi ngay lần đầu tiên khi tải trang
        fetchLiveWeather();

        // Thiết lập vòng lặp tự động lấy dữ liệu sau mỗi 5 giây (5000ms)
        setInterval(fetchLiveWeather, 5000);
    </script>
</body>
</html>
```

---

#### 2.5. Khởi động hệ thống
Sau khi hoàn tất cấu hình, build và khởi động toàn bộ hệ thống:
```
docker compose up -d --build
```

<img width="2342" height="1210" alt="image" src="https://github.com/user-attachments/assets/40160939-a7fa-4b1b-9bce-25d5b86c1e1b" />

Kiểm tra trạng thái các container 
```
docker ps
```
<img width="2340" height="518" alt="image" src="https://github.com/user-attachments/assets/0bd87b51-5b8d-46ca-a7c0-158b8c229b6c" />

---

#### 2.6. Cấu hình NODERED để tự động hóa luồng dữ liệu
Trong giai đoạn này, Node-RED sẽ đóng vai trò đầu não thực hiện 4 nhiệm vụ liên tục:
- Cào dữ liệu thời tiết thực tế từ API công khai. (mỗi 10-30 giây)
- Lưu trạng thái mới nhất vào MariaDB.
- Lưu lịch sử vào InfluxDB (để Grafana vẽ biểu đồ).
- Phân tích dị thường và kích hoạt Telegram Bot gửi tin nhắn cảnh báo vào Group.

##### Bước 1: Chuẩn bị thư viện (nodes) trong Node-RED
Truy cập Node-RED qua địa chỉ `http://<IP_máy_chủ_Ubuntu>:1882`
```
http://192.168.164.129:1882
```

Click vào Menu (3 dấu gạch ngang góc trên bên phải) -> Chọn Manage palette.Chuyển sang thẻ Install, tìm kiếm và nhấn Install lần lượt 3 thư viện sau:
```
node-red-node-mysql (Kết nối MariaDB)

node-red-contrib-influxdb (Kết nối InfluxDB)

node-red-contrib-telegrambot (Kết nối Telegram)
```
<img width="3071" height="1744" alt="image" src="https://github.com/user-attachments/assets/1784410b-2783-4fa0-b6df-b1009d0d5f2d" />

##### Bước 2: Chuẩn bị thông tin Telegram Bot
Trước khi viết Flow, cần chuẩn bị thông tin từ Telegram:

Bot Token: Chat với @BotFather trên Telegram, gõ lệnh /newbot, đặt tên cho bot. Sau khi tạo xong, @BotFather sẽ cấp một chuỗi Token. Copy token này để bước sau dán vào Nodered.

<img width="3070" height="1744" alt="image" src="https://github.com/user-attachments/assets/f5039601-2f7e-4c50-a797-2c70351dfacd" />

Tạo nhóm chat có bot để cảnh báo:
- Tạo một Group mới trên Telegram, thêm các thành viên vào (bao gồm cả tài khoản ID 1875746636 theo yêu cầu bài tập).
- Thêm cả con Bot vừa tạo ở trên vào nhóm này với quyền Admin (để nó có quyền gửi tin nhắn).

<img width="3070" height="1743" alt="image" src="https://github.com/user-attachments/assets/eb5db58b-2a79-4df1-9909-2df1d76e7975" />

##### Bước 3: Thiết kế luồng dữ liệu
kéo các node và điền các thông tin:

###### Node inject để thiết lập mỗi 10s lấy dữ liệu một lần
<img width="3071" height="1594" alt="image" src="https://github.com/user-attachments/assets/965f7bb7-9e7c-485a-8cb4-d40c08db0504" />

###### Node http request để cào dữ liệu thực 
<img width="3066" height="1594" alt="image" src="https://github.com/user-attachments/assets/e4690025-871a-4127-a28d-4eee0269a496" />

###### Node function chuẩn hóa và tách luồng dữ liệu
<img width="3070" height="1594" alt="image" src="https://github.com/user-attachments/assets/0c6d21ae-4b99-46d4-af18-4bbb44a7dec4" />

###### Node mysql
<img width="3065" height="1587" alt="image" src="https://github.com/user-attachments/assets/6a20ec8f-9e5c-4781-addd-47ca217f7a2e" />

###### Node influxdb out
<img width="3067" height="1591" alt="image" src="https://github.com/user-attachments/assets/cd2d523f-3785-4df6-8e6f-8772c15ac511" />

###### Node switch kiểm tra ngưỡng nhiệt độ
<img width="3071" height="1596" alt="image" src="https://github.com/user-attachments/assets/97dd6649-3add-49f3-af16-2c8d402794ce" />

###### Function node tạo cảnh báo low, high
<img width="3063" height="1606" alt="image" src="https://github.com/user-attachments/assets/d5dcc7d3-9de0-4fc6-812e-e81ce9034b8f" />

###### Node sender để bắn cảnh báo tới telegram 
<img width="3069" height="1593" alt="Screenshot 2026-06-07 174122" src="https://github.com/user-attachments/assets/2248d881-283a-4ed2-8eea-a28d20826914" />

##### Bước 4: Deploy và kiểm tra
Bấm nút `Deploy` màu đỏ trên góc phải màn hình để lưu và chạy.

<img width="3066" height="1601" alt="image" src="https://github.com/user-attachments/assets/e43f079c-0541-4435-9b5d-d2f8e2e8c182" />

###### Truy cập vào giao diện web để xem kết quả `http://192.168.164.129`
<img width="3071" height="1735" alt="image" src="https://github.com/user-attachments/assets/4021a7ec-0b05-462a-92d0-0da720d9f9d9" />

> Chú ý: Vì nhiệt độ đang ở mức bình thường, nên để có cảnh báo đẩy về telegram, em sẽ sửa lại ngưỡng bất thường high từ 40 độ còn 30 độ.

###### Kết quả cảnh báo khi nhiệt độ vượt ngưỡng 30 độ:
<img width="3071" height="1729" alt="image" src="https://github.com/user-attachments/assets/9c4872db-0d91-4744-817b-92f4aabaada2" />

> Sau khi test cảnh báo thành công, em đưa về mức nhiệt cảnh báo cao là 38.

#### 2.7. Cấu hình Grafana kết nối InfluxDB
##### Bước 1: Đăng nhập grafana
- Truy cập `http://192.168.164.129:3002` để vào Grafana
- Đăng nhập và đổi mật khẩu (nếu cần)

<img width="3071" height="1729" alt="image" src="https://github.com/user-attachments/assets/06afeaa8-4a0a-465c-968a-6c63d0fe966a" />

##### Bước 2: Thêm datasource 
- Tại thanh menu bên trái, chọn Connections ->  Data sources -> Add data source.

<img width="3057" height="1711" alt="image" src="https://github.com/user-attachments/assets/0080cda5-be09-4e69-b0fa-cbd1badd8598" />

- Chọn InfluxDB.
- Cấu hình các thông số chính xác như sau:

| Trường | Thông tin | 
| --- | --- |
| URL|  http://weather_influxdb:8086 (Gọi bằng tên service nội bộ) |
| Database |  weather_history |
| User | admin |
| Password | adminpassword |

- Kéo xuống dưới cùng ấn Save & test. Nếu hiện thông báo màu xanh "Data source is working" là thành công!
