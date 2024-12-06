# Monitoring?

## 1 Mục tiêu của monitoring

- Hiểu hệ thống: Xem cách ứng dụng hoạt động, hiệu suất, và khả năng đáp ứng.
- Phát hiện sự cố: Cảnh báo sớm khi xảy ra lỗi hoặc hiệu suất không đạt yêu cầu.
- Phân tích nguyên nhân gốc rễ: Giúp tìm hiểu tại sao một sự cố xảy ra.

### 2 Phân tích chi tiết các chỉ số cần log và ý nghĩa thực tế

#### 2.1 Tổng số yêu cầu (Request Count)

**Chỉ số:** `http_requests_total`.  
**Ý nghĩa:**  
   Đây là chỉ số cơ bản để theo dõi lưu lượng truy cập của ứng dụng. Bằng cách biết được số lượng yêu cầu gửi đến, bạn có thể:  
   - Phát hiện sự thay đổi đột ngột, ví dụ:
     - Tăng đột biến -> Có thể là tấn công (DDoS) hoặc chiến dịch quảng cáo.
     - Giảm bất thường -> Ứng dụng hoặc mạng có thể gặp sự cố.  
   - Hiểu mức độ sử dụng để lên kế hoạch mở rộng tài nguyên.  
**Thực tế:**  
   - Log theo các nhãn (label):  
     - **`method`**: Phân biệt giữa `GET`, `POST`,...  
     - **`status`**: Theo dõi trạng thái HTTP (200: thành công, 404: không tìm thấy, 500: lỗi server,...).  
   - Phân tích thời gian thực hoặc theo từng phút.  

**Ví dụ sử dụng PromQL:**  
```promql
sum(rate(http_requests_total[5m])) by (status)
```  
Truy vấn trên hiển thị tốc độ yêu cầu theo trạng thái trong 5 phút qua.

---

#### 2.2 Thời gian phản hồi (Response Time)

**Chỉ số:** `http_response_time_seconds`.  
**Ý nghĩa:**  
   Đo thời gian từ khi nhận request đến khi gửi phản hồi.  
   - Đảm bảo thời gian phản hồi nhanh để cải thiện trải nghiệm người dùng.  
   - Xác định các **bottlenecks** (ví dụ: API bên ngoài, truy vấn database chậm).  
**Thực tế:**  
   - Sử dụng `Histogram` để chia thời gian thành các khoảng (buckets) như: `<0.1s`, `0.5s`, `1s`, `>2s`. 
(<=0.1s: Các request siêu nhanh, <=0.5s: Request nhanh, <=1s: Vẫn chấp nhận được, >2s: Có vấn đề, gây ảnh hưởng trải nghiệm người dùng)
Bên cạnh đó nếu một tỷ lệ lớn request đôt ngột nằm trong bucket >2s, bạn biết cần tập trung cải thiện phần nào của ứng dụng (ví dụ, database query chậm, API bên ngoài, v.v.), vì có thể đây là dấu hiệu của **bottlenecks**
	
   - Tập trung vào **P95** hoặc **P99**: 95% hoặc 99% các yêu cầu nhanh nhất.  
   P95 (Percentile 95): 95% request có thời gian phản hồi nhanh hơn giá trị này và có thể phản ánh trải nghiệm của phần lớn người dùng.
   Ví dụ: Nếu P95 = ~0.7s, có nghĩa rằng 95% request mất dưới ~0.7s, điều này thường chấp nhận được.
   P99 (Percentile 99): 99% request có thời gian phản hồi nhanh hơn giá trị này và có thể phản ánh các trường hợp chậm bất thường. Những request này thường do lỗi hệ thống, kết nối yếu, hoặc tài nguyên giới hạn.


**Ví dụ PromQL:**  
```promql
histogram_quantile(0.95, sum(rate(http_response_time_seconds_bucket[5m])) by (le))
```  
Câu lệnh này tính thời gian phản hồi P95 trong 5 phút.

**Cảnh báo:**  
- Nếu P95 vượt quá 1 giây -> Có thể người dùng đang gặp trải nghiệm chậm.  

---

#### 2.3 Tỷ lệ lỗi (Error Rate)

**Chỉ số:** `http_errors_total`.  
**Ý nghĩa:**  
   Giám sát số lượng lỗi xảy ra trong ứng dụng.  
   - Phát hiện sự cố sớm khi tỷ lệ lỗi tăng.  
   - Phân loại lỗi:  
     - **4xx:** Lỗi từ phía người dùng (yêu cầu không hợp lệ).  
     - **5xx:** Lỗi từ phía server.  
**Thực tế:**  
   - Theo dõi tỷ lệ lỗi (errors per request):  
     ```promql
     rate(http_errors_total[5m]) / rate(http_requests_total[5m])
     ```  

**Cảnh báo:**  
- Tỷ lệ lỗi vượt 5-10% -> Có thể là vấn đề nghiêm trọng cần khắc phục. (phụ thuôc vào loại ứng dụng).
Ví dụ với blog cá nhân tỷ lệ này trên 10-15% vẫn chấp nhận được, nhưng với các hệ thống giao dịch, thanh toán 3% cũng là vấn đề cần suy nghĩ

**Vài trường hợp cần chú ý**
- Lỗi tăng đột biến:
Ví dụ: Tỷ lệ lỗi từ 2% tăng lên 7% trong vòng 5 phút -> Có thể do lỗi mới triển khai hoặc tài nguyên không đủ.
- Lỗi kéo dài liên tục:
Duy trì trên 5% trong 10 phút hoặc lâu hơn -> Cần điều tra ngay lập tức.
- Lỗi liên quan đến một dịch vụ cụ thể:
Ví dụ: Lỗi từ một API hoặc endpoint tăng lên vượt ngưỡng trong khi phần còn lại bình thường.
---

#### 2.4 Số lượng kết nối (Active Connections)

**Chỉ số:** `active_connections`.  
**Ý nghĩa:**  
   Giám sát số lượng kết nối đang mở:  
   - Xác định liệu server có bị quá tải không.  
   - Kiểm tra nếu có nhiều kết nối "treo" (idle connections).  
**Thực tế:**  
   - Giới hạn số lượng kết nối bằng cấu hình server (ví dụ: trong Node.js, bạn có thể giới hạn kết nối tối đa qua `server.maxConnections`).  

---

#### 2.5 Sử dụng tài nguyên (CPU và RAM)

**Chỉ số:**  
   - **CPU:** `node_cpu_seconds_total`.  
   - **RAM:** `node_memory_usage_bytes`.  
**Ý nghĩa:**  
   - Đảm bảo tài nguyên không bị quá tải.  
   - Cảnh báo sớm khi ứng dụng gần đạt giới hạn tài nguyên.  

**Cảnh báo:**  
- CPU vượt 80% trong 5 phút liên tục.  
- RAM sử dụng trên 90%.  

**Ví dụ PromQL:**  
```promql
100 * (1 - avg(rate(node_cpu_seconds_total[1m])) by (instance))
```  
Tính tỷ lệ sử dụng CPU trung bình theo instance.

---

### 3 Lời khuyên thực tế khi triển khai monitoring

**Log bao nhiêu là đủ?**  
   - Tập trung vào các chỉ số quan trọng nhất: Request Count, Response Time, Error Rate.  
   - Với tài nguyên hệ thống, chỉ log nếu bạn không có các công cụ khác như CloudWatch hoặc Datadog.  

**Khi nào cảnh báo?**  
   - Cảnh báo cần dựa trên ngưỡng thực tế của hệ thống:  
     - 95% thời gian phản hồi vượt 1 giây.  
     - Tỷ lệ lỗi vượt 5-10%.  
     - CPU hoặc RAM vượt 80-90%.  

**Khi cảnh báo, nên rà soát gì?**  
   - **Tổng quan:** Kiểm tra logs Prometheus để xem chỉ số nào bất thường.  
   - **Chi tiết:** Dùng tracing (ví dụ: Jaeger) để theo dõi từng request cụ thể.  
   - **Tài nguyên:** Dùng công cụ như `top`, `htop`, hoặc dashboard Grafana để kiểm tra CPU/RAM.  