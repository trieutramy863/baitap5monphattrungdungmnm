# baitap5monphattrungdungmnm
bt5:
  - lý thuyết: 
    + docker là gì? 
    + các keyword được sử dụng trong docker-compose.yml
      để mô tả 1 service, network, volume,...
      liệt kê + ý nghĩa của từ khoá đó + ví dụ minh hoạ
    + ưu điểm khi triển app sử dụng docker là gì?
    + dùng docker: tạo app, test app OK trên laptop cá nhân
      giờ muốn triển khai app này trên máy chủ thật ko có internet
      thì các bước cần làm là?
  - thực hành áp dụng: APP MONITOR + ALERT DATA REALTIME
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

# A.Lý thuyết
Docker là gì?

Docker là một nền tảng mã nguồn mở giúp đóng gói, triển khai và chạy ứng dụng trong các môi trường độc lập gọi là Container. Container chứa toàn bộ những gì ứng dụng cần để hoạt động như mã nguồn, thư viện, cấu hình và các thành phần phụ thuộc.

Nhờ Docker, ứng dụng có thể chạy giống nhau trên nhiều môi trường khác nhau như máy tính cá nhân, máy chủ vật lý hoặc môi trường đám mây mà không gặp vấn đề khác biệt về cấu hình.

Các thành phần chính của Docker
Docker Image: Mẫu được sử dụng để tạo Container.
Docker Container: Phiên bản đang chạy của Image.
Dockerfile: Tệp mô tả cách xây dựng Docker Image.
Docker Compose: Công cụ quản lý và chạy nhiều Container cùng lúc.
Docker Registry: Kho lưu trữ Docker Image như Docker Hub.

Ví dụ chạy một Container Nginx:

docker run nginx
Các keyword thường dùng trong docker-compose.yml

Ví dụ cấu trúc tổng quát:

version: "3.9"

services:
  web:
    image: nginx
    ports:
      - "80:80"

networks:
  app_network:

volumes:
  app_data:
1. version

Xác định phiên bản cú pháp Docker Compose.

version: "3.9"
2. services

Khai báo các dịch vụ (containers) cần chạy.

services:
  web:
3. image

Chỉ định Image được sử dụng để tạo Container.

image: nginx:latest
4. build

Xây dựng Image từ Dockerfile.

build: .

Hoặc:

build:
  context: .
  dockerfile: Dockerfile
5. container_name

Đặt tên cho Container.

container_name: myapp
6. ports

Ánh xạ cổng giữa Host và Container.

ports:
  - "8080:80"

Ý nghĩa:

Cổng 8080 trên máy chủ.
Chuyển tiếp tới cổng 80 trong Container.
7. environment

Khai báo biến môi trường.

environment:
  DB_HOST: mysql
  DB_PORT: 3306
8. volumes

Gắn thư mục hoặc Docker Volume.

volumes:
  - mysql_data:/var/lib/mysql

Hoặc:

volumes:
  - ./src:/app
9. networks

Khai báo mạng cho các Container giao tiếp với nhau.

networks:
  - app_network

Khai báo mạng:

networks:
  app_network:
10. depends_on

Khai báo sự phụ thuộc giữa các Service.

depends_on:
  - mysql

Docker sẽ khởi động MySQL trước Application.

11. restart

Chính sách tự động khởi động lại.

restart: always

Các giá trị thường dùng:

restart: no
restart: on-failure
restart: unless-stopped
restart: always
12. command

Ghi đè lệnh mặc định của Container.

command: npm start
13. hostname

Đặt hostname cho Container.

hostname: webserver
14. working_dir

Thư mục làm việc bên trong Container.

working_dir: /app
15. entrypoint

Lệnh được thực thi đầu tiên khi Container khởi động.

entrypoint:
  - python
  - app.py
16. healthcheck

Kiểm tra trạng thái hoạt động của Container.

healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 30s
  timeout: 10s
  retries: 3
Ví dụ Docker Compose hoàn chỉnh
version: "3.9"

services:
  app:
    build: .
    container_name: myapp

    ports:
      - "3000:3000"

    environment:
      DB_HOST: mysql
      DB_PORT: 3306

    depends_on:
      - mysql

    networks:
      - app_network

  mysql:
    image: mysql:8

    environment:
      MYSQL_ROOT_PASSWORD: 123456

    volumes:
      - mysql_data:/var/lib/mysql

    networks:
      - app_network

volumes:
  mysql_data:

networks:
  app_network:
Ưu điểm khi triển khai ứng dụng bằng Docker
1. Môi trường đồng nhất

Ứng dụng chạy giống nhau trên máy lập trình viên và máy chủ triển khai.

2. Triển khai nhanh

Chỉ cần:

docker compose up -d

Toàn bộ hệ thống sẽ được khởi động.

3. Dễ mở rộng

Có thể chạy nhiều bản sao của ứng dụng:

docker compose up --scale web=5
4. Cô lập ứng dụng

Mỗi ứng dụng chạy trong một Container riêng biệt, tránh xung đột thư viện và phiên bản.

5. Dễ sao lưu và di chuyển

Chỉ cần sao lưu Docker Image và Docker Volume để chuyển sang máy chủ khác.

6. Tiết kiệm tài nguyên

Container nhẹ hơn máy ảo vì dùng chung Kernel của hệ điều hành.

7. Hỗ trợ CI/CD

Dễ dàng tích hợp với Jenkins, GitLab CI/CD, GitHub Actions và Kubernetes.

Triển khai ứng dụng Docker lên máy chủ không có Internet

Giả sử ứng dụng đã được kiểm tra và chạy thành công trên laptop cá nhân.

Bước 1: Build Docker Image
docker build -t myapp:v1 .

Kiểm tra Image:

docker images
Bước 2: Xuất Image ra file
docker save -o myapp_v1.tar myapp:v1

Nếu dự án sử dụng nhiều Image:

docker save -o project.tar myapp:v1 mysql:8 nginx:latest
Bước 3: Chuẩn bị các file triển khai

Sao chép các file:

docker-compose.yml
.env
project.tar
Bước 4: Chuyển sang máy chủ

Sử dụng:

USB
Ổ cứng ngoài
Mạng LAN nội bộ
Bước 5: Cài Docker trên máy chủ

Nếu máy chủ không có Internet, cần tải bộ cài Docker từ máy có Internet và chép sang trước.

Kiểm tra:

docker --version
Bước 6: Nạp Docker Image vào máy chủ
docker load -i project.tar

Kiểm tra:

docker images
Bước 7: Khởi động hệ thống
docker compose up -d

Hoặc:

docker-compose up -d
Bước 8: Kiểm tra trạng thái

Xem Container đang chạy:

docker ps

Xem log:

docker logs myapp

Kiểm tra dịch vụ:

curl http://localhost
Quy trình triển khai tổng quát
Laptop phát triển ứng dụng
        |
        v
Build Docker Image
        |
        v
docker save
        |
        v
File .tar
        |
        v
USB hoặc mạng LAN
        |
        v
Máy chủ không có Internet
        |
        v
docker load
        |
        v
docker compose up -d
        |
        v
Ứng dụng hoạt động

Kết luận: Docker giúp chuẩn hóa môi trường chạy ứng dụng, đơn giản hóa việc triển khai, giảm lỗi do khác biệt cấu hình và cho phép triển khai thuận tiện ngay cả trong các môi trường không có kết nối Internet.
# B. Thực hành
- Tạo thư mục

  <img width="820" height="101" alt="image" src="https://github.com/user-attachments/assets/4c84ff7b-7331-41e1-bf08-44df055bfe2e" />

-  tạo cấu trúc thư mục project.

<img width="812" height="463" alt="image" src="https://github.com/user-attachments/assets/9b28f707-75cc-4868-b2c8-32f03360ffe4" />

- Tạo nano docker-compose.yml

<img width="821" height="57" alt="image" src="https://github.com/user-attachments/assets/1062de42-31d5-4388-9dcf-0d6e67751c40" />

<img width="966" height="698" alt="image" src="https://github.com/user-attachments/assets/590cadd1-693f-4043-b4d6-7f5215332f97" />

- Viết requirements.txt

<img width="914" height="636" alt="image" src="https://github.com/user-attachments/assets/f4604c39-5b67-47e2-8c88-e57f45ab3def" />

- Viết dockerfile

<img width="946" height="1016" alt="image" src="https://github.com/user-attachments/assets/7f40e4c0-2b7c-4784-8edf-dc1d893c6b8c" />

- Viết Flask API (app.py)

<img width="959" height="996" alt="image" src="https://github.com/user-attachments/assets/1c10ded2-926d-4d1c-bde2-5f6c00ad0b08" />

- Mở file SQL

Gõ:

nano mysql-init/init.sql

<img width="918" height="700" alt="image" src="https://github.com/user-attachments/assets/6f6743b9-0d60-4cb0-b2d7-9701e502ed34" />

- chạy toàn bộ container lần đầu.
   + docker compose up -d

<img width="826" height="387" alt="image" src="https://github.com/user-attachments/assets/05c9313b-4fcd-45f3-939f-be537be65ad5" />
<img width="819" height="417" alt="image" src="https://github.com/user-attachments/assets/5fb60c3f-f26c-4606-8cad-8f7344b7ad50" />

- kiểm tra:
   + docker ps

<img width="813" height="497" alt="image" src="https://github.com/user-attachments/assets/89a3a4ef-0288-4a87-bc72-f7c5f2b504ad" />

- Test lại bằng trình duyệt:
    + http://192.168.1.15:5000

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/43228bd5-21b8-4f41-8b62-d103a8a7eba2" />

- Tiếp theo test NodeRed:
 + http://192.168.1.15:1880

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/753209c7-3940-4aa6-b00e-fce172de4404" />

- Test Grafana:
   + http://192.168.1.15:3000

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/0ac78683-2fb2-4a3e-bc94-9cdf8b69f67a" />

<img width="958" height="1194" alt="image" src="https://github.com/user-attachments/assets/1c00f49b-a9f3-40f0-b00b-faff1207a61a" />

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/0d133291-e122-4890-8dfb-14e69446565f" />


- Vào InfluxDB

<img width="940" height="1158" alt="image" src="https://github.com/user-attachments/assets/3c5b44b8-c3dc-4c07-a993-6b498a860506" />

- http://192.168.1.15:5000/weather

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/ac2e5b5d-e522-46f6-97aa-70ba24fad7ba" />

# Grafana — vẽ biểu đồ từ InfluxDB
  - Vào trình duyệt: http://192.168.18.128:3000
     + Thêm Data Source InfluxDB
     + Menu trái → Connections → Data sources
     + Nhấn Add data source
- Chọn InfluxDB

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/baebfc0c-e61a-4652-9951-a34110e43cdc" />

- Data source đã kết nối thành công! Bây giờ tạo Dashboard: Menu trái → Dashboards → New → New dashboard → Nhấn Add visualization → Chọn data source InfluxDB vừa tạo

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/c30f6ab4-5cfb-4786-b961-12adafadac3c" />

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/e024c59c-f2e5-4da8-9b93-9155794eb09e" />

- Tạo bot
 <img width="828" height="1792" alt="image" src="https://github.com/user-attachments/assets/521b3a22-f878-4c7b-a01e-3bc498012def" />

- KÉO TELEGRAM SENDER : Kéo: telegram sender Nối: Function Alert với Telegram Sender -> Cấu hình -> done -> deploy
( để lấy id nhóm chat ta truy cập vào telegram -> tìm @userinfobot -> nhấn start -> ấn group/ hoặc my group sau đó tìm nhóm chat cần lấy id )

<img width="1920" height="1200" alt="Screenshot 2026-06-10 152701" src="https://github.com/user-attachments/assets/20e13c56-84e9-4c77-8795-a9e7f6371f8c" />

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/b6978667-c1fa-4552-87b0-8a2316e0dc0a" />

- KẾT QUẢ

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/7e5b7072-edc2-48eb-a680-eed60b5416d8" />

<img width="828" height="1792" alt="image" src="https://github.com/user-attachments/assets/1d31b9f0-1dd2-48cf-acaf-14855eefa3b1" />

- Tạo giao diện HTML
+ Mở file:
- nano frontend/index.html

<img width="960" height="1017" alt="image" src="https://github.com/user-attachments/assets/f4ef9cae-4272-4a82-8b5f-31d8f5687b78" />

- nano frontend/style.css

<img width="1291" height="1045" alt="image" src="https://github.com/user-attachments/assets/d6bdd208-0db4-419f-be3b-e815495a2539" />

- nano frontend/script.js

<img width="1291" height="1119" alt="image" src="https://github.com/user-attachments/assets/023d15a2-363a-4779-ae34-b533370d9200" />

- nano nginx/default.conf

<img width="1292" height="1131" alt="image" src="https://github.com/user-attachments/assets/7d9b085a-a8c3-4452-9367-fc6f2cc1e6f8" />

- Khởi động lại cd ~/bt5-monitor docker compose up -d
- sau đó kiểm tra xem đã chạy thành công chưa
- mở web
Vào:
 http://192.168.1.15:8080

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/bfb71f23-6101-40c8-a00d-ed5a3968edf6" />

<img width="1920" height="1200" alt="image" src="https://github.com/user-attachments/assets/6567dd7d-e59f-41af-b4a6-98278494e267" />

# Export/Import Docker
- xem các image đang dùng : docker images

<img width="845" height="237" alt="image" src="https://github.com/user-attachments/assets/d90af78d-c7da-4b5b-810d-45cb9f24c275" />

- Tạo thư mục backup
   + Trong ~/bt5-monitor

<img width="560" height="56" alt="image" src="https://github.com/user-attachments/assets/1886071c-4117-4c38-9ba9-09dc475b3bc4" />

EXPORT TOÀN BỘ IMAGE

<img width="346" height="62" alt="image" src="https://github.com/user-attachments/assets/5a3ee569-b5f0-4462-ad64-1197aa56f09b" />

DỪNG HỆ THỐNG Trong thư mục project: cd ~/bt5-monitor docker compose down Kiểm tra: docker ps

<img width="1625" height="274" alt="image" src="https://github.com/user-attachments/assets/97684caf-47a4-4b11-8510-1067f612c770" />

Load lại từ file nén

<img width="867" height="184" alt="image" src="https://github.com/user-attachments/assets/4d4f858d-75c4-4af0-8090-3719fcc2cc5e" />

- khởi động lại các containers: docker compose -f ~/bt5-monitor/docker-compose.yml up -d
- - kiểm tra lại -
<img width="1853" height="453" alt="image" src="https://github.com/user-attachments/assets/6c37e7e5-6ff4-433d-92c3-ee5091163366" />
