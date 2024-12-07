
## 1. Chạy hệ thống

**Khởi động dịch vụ:**
   ```bash
   docker-compose up --build
   ```

**Kiểm tra:**
   - **Node.js App:** [http://localhost:3000](http://localhost:3000)  
   - **cAdvisor:** [http://localhost:8080](http://localhost:8090)  
   - **Prometheus:** [http://localhost:9090](http://localhost:9090)  
   - **Grafana:** [http://localhost:3001](http://localhost:3001)  

---

## 2. Sử dụng Grafana

**Đăng nhập vào Grafana:**  
   - Mặc định tài khoản:  
     - Username: `admin`  
     - Password: `admin`.  

**Cấu hình Data Source:**
   - Vào **Connections > Data Sources > Add Data Source/Add new Datasource**.  
   - Chọn **Prometheus** và nhập URL: `http://prometheus:9090`.  

**Tạo Dashboard:**  
   - Sử dụng các query mẫu:  
     - CPU Usage:  
       ```promql
       rate(container_cpu_usage_seconds_total[1m])
       ```
     - Memory Usage:  
       ```promql
       container_memory_usage_bytes
       ```
     - Network I/O:  
       ```promql
       rate(container_network_receive_bytes_total[1m])
       ```
**Lý do chọn các câu Query(PromQL) trên**

a. Thời gian thu thập (Time Window)
- **`1m` (1 phút)** được sử dụng trong các biểu thức `rate()` vì đây là khoảng thời gian phổ biến để tính toán **tốc độ thay đổi** trong một đơn vị thời gian, nhằm giảm thiểu các biến động nhỏ và lấy được xu hướng chính. (như đơn vị mốc)

- **CPU Usage (`rate(container_cpu_usage_seconds_total[1m])`):**
  - `rate()` tính toán **tốc độ thay đổi** của số liệu trong khoảng thời gian cụ thể (ở đây là 1 phút).
  - **1 phút** được chọn vì:
    - **Độ chính xác tốt:** Với CPU, việc sử dụng một khoảng thời gian ngắn như 1 phút giúp phản ánh chính xác hơn biến động trong quá trình xử lý, mà không làm mất đi độ chính xác.
    - **Tính ổn định:** Dữ liệu CPU có thể thay đổi nhanh chóng, và 1 phút đủ để giảm thiểu ảnh hưởng của những sự thay đổi ngắn hạn nhưng vẫn đảm bảo tính chính xác.

- **Memory Usage (`container_memory_usage_bytes`):**
  - Bộ nhớ thường không thay đổi nhanh như CPU, nên không cần phải tính toán tốc độ thay đổi qua **`rate()`**.
  - Thay vào đó, ta trực tiếp sử dụng giá trị hiện tại (`container_memory_usage_bytes`) để đo lường bộ nhớ đang sử dụng.
  - **Sử dụng ngay giá trị hiện tại** là hợp lý vì bộ nhớ thường ổn định hơn và không có sự thay đổi nhanh chóng như CPU.

- **Network I/O (`rate(container_network_receive_bytes_total[1m])`):**
  - Tương tự như CPU, lưu lượng mạng (network I/O) có thể thay đổi nhanh chóng và có sự biến động lớn.
  - Dùng **1 phút** làm cửa sổ tính toán giúp làm mịn dữ liệu và cung cấp cái nhìn rõ hơn về xu hướng sử dụng mạng.


b. Lý do chọn 1 phút thay vì các khoảng thời gian khác (ví dụ: 5 phút, 10 phút, 15 phút)

- **Độ chính xác và hiệu suất:**
  - **1 phút** là một thỏa hiệp hợp lý giữa độ chính xác và tài nguyên hệ thống. Quá ngắn có thể dẫn đến các dữ liệu quá nhiễu, trong khi quá dài có thể làm mất đi tính phản ánh ngay lập tức về hệ thống.
  - Đối với các môi trường có tính chất thay đổi nhanh (như ứng dụng web, API), 1 phút đủ để quan sát xu hướng sử dụng tài nguyên mà không quá dễ bị ảnh hưởng bởi sự thay đổi tức thời.

- **Phổ biến và dễ sử dụng:** 
  - **1 phút** là khoảng thời gian mặc định được cộng đồng Prometheus và nhiều công cụ giám sát khác sử dụng, vì nó đáp ứng được đa số nhu cầu theo dõi và cung cấp độ phản hồi nhanh.

---

c. Điều chỉnh thời gian (Time Window)
- Tùy vào yêu cầu và tính chất của hệ thống, bạn có thể điều chỉnh cửa sổ thời gian (ví dụ: **5 phút** hoặc **10 phút**) nếu:
  - **Hệ thống ít biến động:** Nếu hệ thống của bạn ổn định, không có sự thay đổi lớn trong ngắn hạn, bạn có thể sử dụng **5 phút** hoặc **10 phút** để làm mượt dữ liệu hơn.
  - **Lưu lượng cao và cần độ chính xác cao:** Nếu hệ thống có yêu cầu giám sát nghiêm ngặt, bạn có thể sử dụng các khoảng thời gian ngắn hơn (như **30 giây**) để giám sát chi tiết hơn.


---

### 3. Luồng

 
- **cAdvisor:** Giám sát chi tiết hệ thống container Node.js.  
- **Prometheus:** Thu thập metrics từ cAdvisor.  
- **Grafana:** Dashboard trực quan để hiển thị trạng thái container Node.js.

