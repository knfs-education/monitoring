Trong ví dụ đang sử dung **cAdvisor** đẻ thu thập log (có nhiều cách có thể tự code), cùng với đó cũng có các công cụ khác để thu thập như **Node Exporter**

**Node Exporter** và **cAdvisor** đều là các công cụ phổ biến để thu thập metrics, nhưng chúng có mục đích sử dụng và phạm vi khác nhau. Dưới đây là sự khác biệt chính giữa hai công cụ này:

---

### **1. Node Exporter**
- **Mục đích:** 
  - Thu thập các metrics về **máy chủ (host)** như CPU, bộ nhớ, ổ đĩa, và mạng.  
  - Dùng để giám sát tài nguyên **phần cứng** và các thông số của hệ điều hành.  
- **Phạm vi:**
  - Chỉ tập trung vào host chạy hệ điều hành (physical/virtual server).  
  - Không thu thập thông tin cụ thể của container.  

- **Ví dụ về các metrics được thu thập:**
  - CPU: `node_cpu_seconds_total` (thời gian sử dụng CPU).  
  - RAM: `node_memory_Active_bytes` (bộ nhớ đang hoạt động).  
  - Disk: `node_disk_io_time_seconds_total` (thời gian I/O ổ đĩa).  
  - Network: `node_network_receive_bytes_total` (dữ liệu nhận qua mạng).

---

### **2. cAdvisor**
- **Mục đích:** 
  - Thu thập metrics về **container**, tập trung vào môi trường chạy Docker.  
  - Giám sát tài nguyên được container sử dụng như CPU, RAM, Disk, và Network.  
- **Phạm vi:**
  - Theo dõi các container riêng lẻ chạy trên host.  
  - Không cung cấp các thông tin tổng quan về phần cứng của host (như Node Exporter).  

- **Ví dụ về các metrics được thu thập:**
  - Container CPU: `container_cpu_usage_seconds_total` (thời gian CPU của container).  
  - Container Memory: `container_memory_usage_bytes` (bộ nhớ container sử dụng).  
  - Container Disk: `container_fs_usage_bytes` (sử dụng disk của container).  
  - Container Network: `container_network_transmit_bytes_total` (dữ liệu gửi qua mạng).




**Thực tế:**  
- **Kết hợp cả hai.**  
  - Node Exporter giám sát toàn bộ hệ thống.  
  - cAdvisor theo dõi chi tiết các container.  
  - Cả hai có thể tích hợp với Prometheus để cung cấp cái nhìn tổng quan và chi tiết nhất.  
