# Báo Cáo Phân Tích Hệ Thống OpenEMS

## 1. Hệ thống này là gì?

**OpenEMS (Open Energy Management System)** là một nền tảng mã nguồn mở dạng module dành cho các ứng dụng quản lý năng lượng. Hệ thống này được thiết kế để giám sát, điều khiển và tích hợp các hệ thống lưu trữ năng lượng (ESS), năng lượng tái tạo (như điện mặt trời), trạm sạc xe điện (EVCS), bơm nhiệt và các thiết bị điện khác.

Mục tiêu chính của OpenEMS là tối ưu hóa việc sử dụng năng lượng (ví dụ: tối đa hóa việc tự tiêu thụ điện mặt trời, cắt giảm đỉnh tải, hoặc sạc xe điện thông minh) thông qua các thuật toán điều khiển linh hoạt.

## 2. Các thành phần chính của hệ thống

Hệ thống được chia thành **3 phần chính** hoạt động tương tác chặt chẽ với nhau:

### A. OpenEMS Edge (Phần Biên)
*   **Vị trí:** Chạy tại chỗ (on-site), thường là trên các gateway công nghiệp hoặc máy tính nhúng (như Raspberry Pi, FENECON EMS).
*   **Vai trò:** Đây là "bộ não" cục bộ. Nó giao tiếp trực tiếp với phần cứng (biến tần, đồng hồ đo, pin), thu thập dữ liệu và thực thi các thuật toán điều khiển thời gian thực.
*   **Công nghệ:** Java, OSGi (Open Services Gateway initiative). Kiến trúc module cho phép thêm/bớt driver thiết bị mà không cần dừng hệ thống.

### B. OpenEMS Backend (Phần Máy Chủ/Cloud)
*   **Vị trí:** Chạy trên máy chủ đám mây (Cloud).
*   **Vai trò:** Kết nối hàng nghìn thiết bị OpenEMS Edge phân tán. Nó cung cấp khả năng tổng hợp dữ liệu, giám sát từ xa, quản lý người dùng và xác thực.
*   **Công nghệ:** Java, OSGi. Sử dụng các cơ sở dữ liệu chuỗi thời gian (Time-series Database) như InfluxDB hoặc TimescaleDB để lưu trữ dữ liệu lịch sử.

### C. OpenEMS UI (Giao diện người dùng)
*   **Vị trí:** Chạy trên trình duyệt web hoặc ứng dụng di động (Android/iOS).
*   **Vai trò:** Hiển thị dữ liệu thời gian thực (Live View), biểu đồ lịch sử (History View) và cho phép người dùng thay đổi cài đặt hệ thống.
*   **Công nghệ:**
    *   Frontend: **Angular 20** và **Ionic 8**.
    *   Biểu đồ: Chart.js, D3.
    *   Mobile wrapper: Capacitor (cho phép đóng gói thành ứng dụng native).
    *   Giao tiếp: WebSocket (thời gian thực) và REST API.

---

## 3. Danh sách các chức năng

Hệ thống cung cấp một loạt các chức năng phong phú nhờ kiến trúc module:

### Chức năng Giao tiếp & Tích hợp (Bridges & Drivers)
*   **Giao thức công nghiệp:** Modbus (TCP, RTU), MQTT, M-Bus, OneWire, REST API, WebSocket.
*   **Tích hợp phần cứng:** Hỗ trợ sẵn hàng trăm thiết bị:
    *   **Pin lưu trữ:** Fenecon, BYD, BMW, Tesla Powerwall, LG Chem, Pylontech.
    *   **Biến tần (Inverters):** SMA, SolarEdge, Fronius, KACO, GoodWe, Kostal, Victron.
    *   **Trạm sạc xe điện (EVCS):** Keba, Mennekes, Hardy Barth, ABL, Webasto.
    *   **Đồng hồ đo (Meters):** Janitza, Socomec, Siemens, Carlo Gavazzi.
    *   **IO/Smart Home:** Shelly, Wago, Siemens Logo, GPIO.

### Chức năng Điều khiển (Controllers)
Các thuật toán thông minh chạy trên OpenEMS Edge:
1.  **Quản lý Lưu trữ năng lượng (ESS):**
    *   **Self-Consumption Optimization:** Tối ưu hóa việc tự tiêu thụ điện mặt trời.
    *   **Peak Shaving:** Cắt giảm đỉnh tải để giảm chi phí tiền điện theo công suất.
    *   **Balancing:** Cân bằng pha lưới điện.
    *   **Time-of-Use (ToU):** Sạc/xả pin dựa trên biểu giá điện theo giờ.
2.  **Quản lý Sạc xe điện (EVCS):**
    *   Sạc bằng năng lượng mặt trời dư thừa.
    *   Cân bằng tải giữa nhiều trạm sạc.
3.  **Quản lý Tải (Load Management):**
    *   Điều khiển bơm nhiệt (Heat Pump - SG Ready).
    *   Bật/tắt thiết bị dựa trên ngưỡng công suất hoặc trạng thái pin (SoC).
4.  **Dự báo (Predictors):**
    *   Dự báo sản lượng điện mặt trời và nhu cầu tiêu thụ dựa trên dữ liệu lịch sử và thời tiết.

### Chức năng Giám sát & Báo cáo
*   Giám sát trực quan dòng năng lượng (Energy Flow) theo thời gian thực.
*   Báo cáo lịch sử tiêu thụ và sản xuất năng lượng.
*   Hệ thống cảnh báo lỗi (Alerting) qua email hoặc thông báo.

---

## 4. Độ phức tạp của hệ thống

Hệ thống OpenEMS có **độ phức tạp cao** vì các lý do sau:

1.  **Kiến trúc Phân tán (Distributed Architecture):** Phải đồng bộ trạng thái giữa thiết bị phần cứng thực tế (Edge), giao diện người dùng (UI) và máy chủ đám mây (Backend) với độ trễ thấp.
2.  **Tính Module hóa (OSGi):** Hệ thống được chia nhỏ thành hàng trăm "Bundle" (gói) riêng biệt. Việc quản lý sự phụ thuộc (dependencies) và vòng đời (lifecycle) của các module này đòi hỏi hiểu biết sâu về Java OSGi.
3.  **Đa dạng Phần cứng:** Hệ thống phải xử lý sự khác biệt về giao thức và hành vi của hàng trăm loại thiết bị từ các nhà sản xuất khác nhau, nhưng vẫn phải cung cấp một giao diện trừu tượng thống nhất (API) cho các thuật toán điều khiển.
4.  **Yêu cầu Thời gian thực (Real-time):** OpenEMS Edge hoạt động như một bộ điều khiển PLC, yêu cầu xử lý chu kỳ điều khiển nhanh và ổn định để đảm bảo an toàn điện lưới.
5.  **Công nghệ Đa dạng:** Lập trình viên cần nắm vững cả Backend (Java, Database, Networking, Multi-threading) và Frontend (TypeScript, Angular, RxJS, Charting).

Tóm lại, đây là một hệ thống **cấp độ doanh nghiệp (enterprise-grade)**, mạnh mẽ và có khả năng mở rộng cao, phù hợp cho từ hộ gia đình đến các dự án năng lượng quy mô công nghiệp.
