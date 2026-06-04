# BTVN5_APP-MONITOR-ALERT-DATA-REALTIME
## PHẦN 1: LÝ THUYẾT
1. Docker là gì?
   Docker là một nền tảng mã nguồn mở cho phép lập trình viên đóng gói ứng dụng và tất cả các thành phần phụ thuộc (thư viện, môi trường, cấu hình...) vào trong một đơn vị duy nhất gọi là Container.
   Bản chất: Khác với ảo hóa truyền thống (Virtual Machine - cần chạy cả một hệ điều hành khách), Docker chia sẻ chung nhân (Kernel) của hệ điều hành host, giúp container khởi động trong vài giây và cực kỳ nhẹ.

3. Các từ khóa trong docker-compose.ymldocker-compose giúp quản lý đa container (multi-container) chỉ bằng một file cấu hình duy nhất
   
   <img width="454" height="666" alt="image" src="https://github.com/user-attachments/assets/1a6f0113-08a1-4996-8392-de989784fbbc" />
   
4. Ưu điểm khi triển khai ứng dụng bằng Docker
   
Nhất quán môi trường: Giải quyết triệt để câu nói kinh điển "Nhưng trên máy em vẫn chạy được mà!". Chạy ở laptop thế nào thì lên server y như vậy.

- Tiết kiệm tài nguyên: Container nhẹ hơn VM rất nhiều, chạy được nhiều service trên cùng một phần cứng.

- Triển khai nhanh chóng: Khởi động, tắt, hoặc tái cấu trúc hệ thống chỉ bằng 1-2 câu lệnh.

- Cách ly an toàn: Các ứng dụng chạy độc lập, lỗi của container này không làm sập container khác.

5. Quy trình triển khai ứng dụng lên Máy chủ Offline (Không có Internet)

Bước 1 (Tại laptop có Internet): Tải tất cả các image cần thiết về và đóng gói thành file nén .tar:
docker save -o my_images.tar nodered/node-red mariadb:10.6 influxdb:2.7 grafana/grafana nginx:alpine

Bước 2 (Chuyển file): Copy file my_images.tar và toàn bộ thư mục code (bao gồm file docker-compose.yml, mã nguồn Flask, Frontend) vào USB hoặc ổ cứng di động, đem cắm vào Máy chủ thật.

Bước 3 (Tại máy chủ Offline): Bung file nén để nạp các image vào Docker của Server:docker load -i my_images.tar

Bước 4 (Khởi chạy): Di chuyển vào thư mục chứa file docker-compose.yml trên server và chạy lệnh:docker compose up -d

## PHẦN 2: THIẾT LẬP CẤU TRÚC THƯ MỤC DỰ ÁN

## Mở Terminal/CMD trên laptop của bạn, chọn một thư mục bất kỳ và gõ chuỗi lệnh sau để tạo cấu trúc cây thư mục:

mkdir bt5_monitor_app

cd bt5_monitor_app

mkdir flask-api

mkdir frontend

## PHẦN 3: VIẾT CODE CHI TIẾT CHO TỪNG THÀNH PHẦN

## 1. Cấu phần Flask API (Lấy dữ liệu từ MariaDB)

<img width="1643" height="519" alt="image" src="https://github.com/user-attachments/assets/c7f3bff7-61b5-4d10-aa9d-e1ae7739605c" />

## Tiếp tục trong thư mục flask-api, tạo file app.py

<img width="1356" height="1002" alt="image" src="https://github.com/user-attachments/assets/641b0549-427a-47c6-b4ba-7e3df8b6ee3d" />

## 2. Cấu phần Giao diện Frontend (Nginx thực thi)

Vào thư mục frontend, tạo file tên là index.html. File này sẽ tự động gọi API Flask mỗi 2 giây để cập nhật số thực tế và nhúng Grafana qua thẻ iframe

*<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Hệ thống giám sát dữ liệu Realtime</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 30px; background-color: #f5f7fa; color: #333; }
        .container { max-width: 1200px; margin: auto; }
        .card { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); margin-bottom: 20px; text-align: center; }
        .price-text { font-size: 48px; color: #e74c3c; font-weight: bold; margin: 15px 0; }
        .time-text { color: #7f8c8d; font-style: italic; }
        iframe { width: 100%; height: 450px; border: none; border-radius: 10px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
    </style>
</head>
<body>
    <div class="container">
        <h2>BÀI TẬP 5: HỆ THỐNG GIÁ SÁT REALTIME & CẢNH BÁO TELEGRAM</h2>
        
        <div class="card">
            <h3>GIÁ TRỊ TỨC THỜI (Từ MariaDB qua Flask API)</h3>
            <div class="price-text" id="price-value">Đang tải...</div>
            <div class="time-text" id="time-value">Cập nhật lúc: --:--:--</div>
        </div>

        <div class="card">
            <h3>BIỂU ĐỒ LỊCH SỬ DỮ LIỆU (Nhúng Grafana Iframe)</h3>
            <iframe src="http://localhost:3000/d-solo/r-O7Z8_Vk/gold-monitor?orgId=1&panelId=1&refresh=5s" width="100%" height="450"></iframe>
        </div>
    </div>

    <script>
        function updateRealtimeData() {
            // Gọi API của Flask để lấy giá trị mới nhất
            fetch('http://localhost:5000/api/realtime')
                .then(response => response.json())
                .then(data => {
                    if(data && data.price) {
                        document.getElementById('price-value').innerText = "$" + data.price;
                        document.getElementById('time-value').innerText = "Cập nhật lúc: " + data.updated_at;
                    }
                })
                .catch(error => {
                    console.error("Lỗi kết nối API:", error);
                    document.getElementById('price-value').innerText = "Lỗi API";
                });
        }

        // Tự động kích hoạt lấy dữ liệu mới sau mỗi 2 giây (2000 ms)
        setInterval(updateRealtimeData, 2000);
        // Chạy lần đầu ngay khi load trang
        updateRealtimeData();
    </script>
</body>
</html>*

## 3. File điều phối Docker Compose Tổng thể

Quay lại thư mục gốc bt5_monitor_app, tạo file docker-compose.yml để liên kết cả 6 thành phần lại với nhau trong cùng một mạng ảo monitor_net.

<img width="1163" height="1032" alt="image" src="https://github.com/user-attachments/assets/42fd64ee-68cc-44fe-8ad5-a0ff4e7bbf4e" />
<img width="1035" height="978" alt="image" src="https://github.com/user-attachments/assets/5b899a79-f7e7-4df0-8d05-4bb8bc3af575" />
<img width="844" height="667" alt="image" src="https://github.com/user-attachments/assets/dff2c257-b3aa-438a-a4bb-02ffc67efe31" />

## PHẦN 4: KHỞI CHẠY VÀ CẤU HÌNH LIÊN KẾT (NODE-RED, TELEGRAM, GRAFANA)

Bước 1: Bật toàn bộ hệ thống
Tại thư mục gốc bt5_monitor_app, bật Terminal lên và gõ:docker compose up -d
<img width="1245" height="336" alt="image" src="https://github.com/user-attachments/assets/17b6e1bb-9fa0-4127-9483-ffd14d52ecb0" />

## Bước 2: Tạo Bảng trong MariaDb

Vì Flask và Node-RED cần một bảng cấu trúc để lưu giá trị tức thời, bạn hãy truy cập vào MariaDB bên trong container bằng lệnh này để tạo bảng:   
docker exec -it mariadb_server mysql -u root -prootpassword -e "USE monitor_db; CREATE TABLE IF NOT EXISTS gold_price (id INT AUTO_INCREMENT PRIMARY KEY, price DOUBLE, updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP);"


## Bước 3: Cấu hình Bot Telegram và Tạo Group 3 Người

Mở Telegram, tìm kiếm @BotFather. Gõ lệnh /newbot, đặt tên cho Bot là LyBitCoin_bot và nhận chuỗi Token bí mật:8933665636:AAFBF0-N63fF7DJrL0URB-dbB30pN1oONBw

Tạo một Group mới trên Telegram. Thêm bạn bè vào.

Mời chính con Bot vừa tạo ở trên vào nhóm này .

Để lấy Chat ID của Group này, add thêm con bot @MissRose_bot vào nhóm, gõ lệnh /id, nó sẽ trả về một dãy số âm. Đó chính là Chat ID của nhóm.

Sau khi lấy được id nhóm rồi thì thăng chức admin cho con bot LyBitCoin.

## Bước 4: Thiết lập Flow xử lý trong Node-RED

Truy cập trình duyệt theo địa chỉ: http://localhost:1880.

Ta tiến hành kéo các khối (node) để thực hiện logic:Lấy dữ liệu động: Dùng node Inject thiết lập lặp lại mỗi 5 giây Nối vào node HTTP Request.

URL lấy dữ liệu thực tế (Ví dụ tỷ giá Bitcoin từ CoinGecko công khai, dữ liệu biến động liên tục không cần API Key): https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd

Đọc dữ liệu JSON: Nối đầu ra vào node JSON để chuyển chuỗi text thành Object của Javascript. Lúc này giá tiền sẽ nằm ở biến msg.payload.bitcoin.usd.

Lưu trữ vào MariaDB: Kéo node Function để viết câu lệnh chèn dữ liệu thực tế vào MariaDB:

<img width="1045" height="234" alt="image" src="https://github.com/user-attachments/assets/0254f271-b7ba-4e3d-be67-c8d3da710e06" />

Lưu trữ vào InfluxDB: Nối đầu ra của node JSON vào node InfluxDB out. Điền cấu hình Server là http://influxdb_server:8086, điền Token, Tổ chức (bkuorg) và Bucket (monitor_bucket) đã thiết lập trong file docker-compose.

<img width="1166" height="842" alt="image" src="https://github.com/user-attachments/assets/4142df4b-3e13-4690-9224-86d32bcb056a" />

Xử lý Cảnh báo (Alert Logic) & Gửi Telegram: Kéo một node Function khác để phân tích giá trị bất thường

<img width="1277" height="756" alt="image" src="https://github.com/user-attachments/assets/5f3449e4-a85f-4145-84aa-293d8b58e689" />

## Tổng quan nodered

<img width="1346" height="692" alt="image" src="https://github.com/user-attachments/assets/9af8a890-7eab-4914-98cd-640958001df8" />

## Bước 5: Cấu hình Grafana Vẽ Biểu Đồ
Truy cập http://localhost:3000 (Tài khoản mặc định: admin / admin).

Vào mục Connections $\rightarrow$ Data Sources $\rightarrow$ Chọn InfluxDB. 

Cấu hình URL kết nối là http://influxdb_server:8086, 
chọn ngôn ngữ truy vấn là Flux và kết nối trực tiếp tới bucket monitor_bucket.Tạo một Dashboard mới, thêm một Panel đồ thị để vẽ đường biến thiên giá Bitcoin theo thời gian thực.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/24550b64-c304-4b0f-9715-39b74f78a6ef" />

Nhấp vào dấu 3 chấm góc Panel $\rightarrow$ Chọn Share $\rightarrow$ Chọn tab Embed $\rightarrow$ Sao chép đường link trong thuộc tính src và cập nhật lại vào thẻ iframe trong file frontend/index.html của bạn để đồng bộ hiển thị lên trang web chính.

<img width="1216" height="176" alt="image" src="https://github.com/user-attachments/assets/d5eae41b-86d3-4773-a0b2-c8b1be0f28bc" />

## Giao diện tổng quan bitcoin

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/5022498b-d167-46b1-bce8-fa9dce0f4631" />


## Test Bot cảnh báo (Ngưỡng cảnh báo từ 60000 đến 95000)

<img width="1125" height="2436" alt="image" src="https://github.com/user-attachments/assets/7096ba6e-0fd9-40c9-9e7e-482e82e7d183" />
<img width="1125" height="2436" alt="image" src="https://github.com/user-attachments/assets/de6af17c-d2f9-4495-ad55-b6510c4420e7" />


## PHẦN 5: QUY TRÌNH "XUẤT - XÓA - KHÔI PHỤC" CONTAINER THEO ĐỀ BÀI

### Bước 1: Đóng gói trạng thái Container hiện tại thành Image mới
Vì các container đang chạy chứa các cấu hình bạn vừa thiết lập trực tiếp (ví dụ như các luồng flow trong Node-RED), ta cần chụp lại trạng thái này thành các bản Image Backup:
### Bước 2: Xuất các Image này ra một file nén vật lý .tar
docker save -o exercise5_backup.tar nodered_backup_img mariadb_backup_img flask_api_service

<img width="1105" height="294" alt="image" src="https://github.com/user-attachments/assets/ad378ea9-5fa6-4ad5-b8da-6a42186f3628" />

### Bước 3: Xóa sạch toàn bộ hệ thống đang chạy để mô phỏng sự cố sập nguồn

<img width="1033" height="499" alt="image" src="https://github.com/user-attachments/assets/daf7a9d8-5291-48d6-a68a-ffb487952955" />

 Tắt và xóa toàn bộ container thuộc docker-compose
docker compose down

 Xóa triệt để mọi container đang chạy khác nếu có trên máy
docker rm -f $(docker ps -aq)

 Xóa các Image gốc để máy tính không còn dữ liệu cũ
docker rmi nodered_backup_img mariadb_backup_img


### Bước 4: Nạp lại dữ liệu từ file nén để khôi phục
docker load -i exercise5_backup.tar
<img width="1117" height="515" alt="image" src="https://github.com/user-attachments/assets/75ac35c0-75f6-477f-8503-415418e9469e" />
