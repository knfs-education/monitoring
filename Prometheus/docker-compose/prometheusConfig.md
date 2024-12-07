Trong tệp cấu hình Prometheus (`prometheus.yml`), mỗi trường và thuộc tính đóng vai trò quan trọng trong việc thu thập dữ liệu, đánh giá và quản lý các chỉ số. Dưới đây là giải thích chi tiết cho từng thuộc tính và cách cấu hình chúng một cách hiệu quả:

---

### **1. `global`: Cấu hình chung**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  scrape_timeout: 10s
```

#### **Giải thích các thuộc tính:**
- **`scrape_interval`**  
  - Tần suất Prometheus gửi yêu cầu thu thập dữ liệu từ các targets (mặc định: `1m`).  
  - Ví dụ:
    ```yaml
    scrape_interval: 15s
    ```
    - Phù hợp cho hệ thống thời gian thực hoặc cần dữ liệu chi tiết.

- **`evaluation_interval`**  
  - Tần suất đánh giá các biểu thức (ví dụ: tính toán, tổng hợp, cảnh báo).  
  - Giá trị phổ biến là bằng với `scrape_interval` để dữ liệu mới luôn được đánh giá.

- **`scrape_timeout`**  
  - Thời gian tối đa cho một yêu cầu thu thập dữ liệu từ target trước khi bị hủy (mặc định: `10s`).  
  - Nên nhỏ hơn hoặc bằng `scrape_interval`.

---

### **2. `scrape_configs`: Cấu hình thu thập dữ liệu**
```yaml
scrape_configs:
  - job_name: 'my-job'
    scrape_interval: 30s
    scrape_timeout: 10s
    metrics_path: '/metrics'
    scheme: 'http'
    static_configs:
      - targets:
          - 'localhost:8080'
```

#### **Các thuộc tính quan trọng:**

1. **`job_name`**  
   - Tên của job (mục giám sát), giúp phân biệt giữa các nguồn dữ liệu khác nhau.  
   - Ví dụ:
     ```yaml
     job_name: 'cadvisor'
     ```

2. **`scrape_interval`** (tùy chọn, ghi đè `global`)  
   - Tần suất thu thập riêng cho job này.  
   - Ví dụ:
     ```yaml
     scrape_interval: 30s
     ```

3. **`scrape_timeout`** (tùy chọn, ghi đè `global`)  
   - Thời gian tối đa để hoàn thành yêu cầu scrape của job này.  
   - Ví dụ:
     ```yaml
     scrape_timeout: 10s
     ```

4. **`metrics_path`**  
   - Đường dẫn nơi Prometheus thu thập số liệu từ target (mặc định: `/metrics`).  
   - Ví dụ:
     ```yaml
     metrics_path: '/custom_metrics'
     ```

5. **`scheme`**  
   - Giao thức HTTP hoặc HTTPS để kết nối với target.  
   - Ví dụ:
     ```yaml
     scheme: 'https'
     ```

6. **`static_configs`**  
   - Dùng để liệt kê danh sách cố định các target (endpoint).  
   - Ví dụ:
     ```yaml
     static_configs:
       - targets:
           - 'cadvisor:8080'
           - 'node-exporter:9100'
     ```

---

### **3. `relabel_configs`: Điều chỉnh nhãn (Labels)**
`relabel_configs` giúp thay đổi, xóa hoặc thêm nhãn trước khi dữ liệu được lưu.

Ví dụ cấu hình:
```yaml
scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets:
          - 'cadvisor:8080'
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        replacement: 'cadvisor-instance'
      - action: drop
        regex: 'localhost:.*'
        source_labels: [__address__]
```

#### **Các thuộc tính quan trọng:**
1. **`source_labels`**  
   - Các nhãn nguồn dùng để tạo hoặc chỉnh sửa nhãn.  
   - Ví dụ:
     ```yaml
     source_labels: [__address__]
     ```

2. **`target_label`**  
   - Nhãn đích sẽ được sửa đổi.  
   - Ví dụ:
     ```yaml
     target_label: instance
     ```

3. **`replacement`**  
   - Giá trị thay thế cho nhãn đích.  
   - Ví dụ:
     ```yaml
     replacement: 'custom-instance'
     ```

4. **`action`**  
   - Hành động cần thực hiện (ví dụ: `drop`, `keep`, `replace`).  
   - Ví dụ:
     ```yaml
     action: drop
     ```

5. **`regex`**  
   - Biểu thức chính quy dùng để khớp giá trị nhãn.  
   - Ví dụ:
     ```yaml
     regex: 'localhost:.*'
     ```

---

### **4. Ví dụ cấu hình nâng cao: Nhiều job và dynamic targets**
```yaml
scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets:
          - 'cadvisor:8080'

  - job_name: 'node-exporter'
    static_configs:
      - targets:
          - 'node-exporter:9100'

  - job_name: 'kubernetes'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_name]
        target_label: pod_name
```

#### **Giải thích:**
1. **Job 1:** `cadvisor` thu thập từ endpoint cố định.
2. **Job 2:** `node-exporter` giám sát tài nguyên của máy chủ.
3. **Job 3:** `kubernetes` tự động phát hiện các pods trong cluster.

- **`kubernetes_sd_configs`**  
  - Tự động phát hiện endpoint từ Kubernetes API.  
  - **`role: pod`**: Theo dõi pod-level.

- **Relabeling**:
  - **`__meta_kubernetes_pod_name`**: Gắn nhãn tên pod từ Kubernetes vào dữ liệu.

---

### **5. Gợi ý tối ưu**
1. **Đặt `scrape_interval` hợp lý:**
   - Hệ thống quan trọng: 15s - 30s.
   - Hệ thống ít biến động: 1m hoặc lâu hơn.

2. **Sử dụng `relabel_configs` để tối ưu dữ liệu:**
   - Loại bỏ các nhãn không cần thiết.
   - Tập trung vào các nhãn quan trọng như: `instance`, `environment`.

3. **Tận dụng dynamic targets:**  
   - Dùng `kubernetes_sd_configs` hoặc `file_sd_configs` để tự động phát hiện.

4. **Kiểm tra cấu hình trước khi chạy:**
   ```bash
   prometheus --config.file=prometheus.yml --test
   ```

