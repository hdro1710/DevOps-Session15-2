# BÀI THỰC HÀNH: ĐIỀU CHỈNH SCRAPE INTERVAL VÀ EVALUATION INTERVAL TRONG PROMETHEUS

## 1. Mục tiêu

- Hiểu vai trò của `scrape_interval` và `evaluation_interval` trong Prometheus.
- Biết cách điều chỉnh chu kỳ thu thập dữ liệu và đánh giá các rules.
- Thực hành chỉnh sửa file cấu hình `prometheus.yml`.
- Biết cách áp dụng và xác minh cấu hình mới thông qua việc restart Prometheus và sử dụng API.

## 2. Bối cảnh

Prometheus được cấu hình với chu kỳ thu thập dữ liệu và đánh giá rules quá ngắn, có thể làm tăng số lượng request, mức sử dụng CPU và RAM.

Để giảm tải cho hệ thống, cần điều chỉnh hai tham số trong phần `global` từ `1s` thành `15s`.

## 3. Các bước thực hiện

### Bước 1: Kiểm tra cấu hình hiện tại

Sử dụng lệnh:

```bash
cat prometheus.yml
```

Kết quả cho thấy cấu hình hiện tại đã có:

```yaml
global:
  scrape_interval: 15s
```

Tuy nhiên, chưa khai báo `evaluation_interval`.

### Bước 2: Chỉnh sửa file cấu hình

Mở file bằng lệnh:

```bash
nano prometheus.yml
```

Bổ sung `evaluation_interval: 15s` để phần `global` hoàn chỉnh:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
```

Các cấu hình `scrape_configs` của những job `prometheus`, `dummy-backend` và `node` được giữ nguyên.

### Bước 3: Kiểm tra cấu hình Docker Compose

Thực hiện:

```bash
sudo docker compose config -q
```

Kết quả: Không xuất hiện lỗi, cho thấy cấu hình Docker Compose hợp lệ.

### Bước 4: Khởi động lại Prometheus

Áp dụng cấu hình mới bằng lệnh:

```bash
sudo docker compose restart prometheus
```

Kiểm tra trạng thái container:

```bash
docker ps
```

Kết quả: Container `prometheus` đã khởi động lại và chuyển sang trạng thái `Up`. Các container Grafana, Node Exporter và những dịch vụ khác vẫn hoạt động.

### Bước 5: Xác minh cấu hình qua Prometheus API

Sử dụng API kiểm tra cấu hình đang được Prometheus nạp:

```bash
sudo docker exec prometheus wget -qO- http://localhost:9090/api/v1/status/config
```

Kết quả trả về có trạng thái:

```json
{
  "status": "success"
}
```

Trong nội dung YAML trả về, xác nhận được:

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s
```

Đồng thời, cả ba job `prometheus`, `dummy-backend` và `node` đều có `scrape_interval: 15s`.

## 4. Kết quả đạt được

- Đã thiết lập `scrape_interval` thành `15s`.
- Đã bổ sung và thiết lập `evaluation_interval` thành `15s`.
- Prometheus đã được restart thành công.
- API xác nhận Prometheus đã nạp cấu hình mới.
- Các container trong hệ thống giám sát vẫn hoạt động sau khi restart Prometheus.

## 5. Kết luận

Qua bài thực hành, đã nắm được cách điều chỉnh chu kỳ thu thập dữ liệu và chu kỳ đánh giá rules trong Prometheus thông qua file `prometheus.yml`.

Việc tăng chu kỳ từ `1s` lên `15s` giúp giảm tần suất thu thập và xử lý metrics, qua đó có thể hạn chế tải cho Prometheus. Tuy nhiên, bài thực hành chưa tiến hành đo đạc CPU/RAM trước và sau khi thay đổi nên chưa có số liệu định lượng để xác nhận mức giảm tải thực tế.

Cần phân biệt `scrape_interval` với `evaluation_interval`: tham số thứ nhất quyết định tần suất thu thập metrics từ các target, trong khi tham số thứ hai quyết định tần suất đánh giá recording rules và alerting rules. Tần suất tự động làm mới dashboard Grafana được cấu hình độc lập.
