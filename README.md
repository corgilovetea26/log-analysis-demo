# Log Analysis Assistant - OpenTelemetry Demo Fork

## Tổng quan
Đây là bản fork của OpenTelemetry Astronomy Shop được tùy chỉnh cho bài toán CNPM:

- Chạy ứng dụng microservices ecommerce demo.
- Thu thập telemetry (traces, metrics, logs, profiles) bằng OpenTelemetry.
- Gửi dữ liệu giám sát lên Grafana Cloud thông qua OTel Collector local.

## Thông tin project
- Repository: https://github.com/corgilovetea26/log-analysis-demo

## Kiến trúc telemetry đang áp dụng
1. Các service gửi OTLP về `otel-collector` local trong Docker network.
2. `otel-collector` export lên Grafana Cloud.
3. Frontend và backend đều đi qua collector, tránh thiếu dữ liệu telemetry.

## Cấu hình Grafana Cloud
Cập nhật file `.env.override` (không commit file này):

```dotenv
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
GRAFANA_CLOUD_OTLP_ENDPOINT=https://otlp-gateway-<region>.grafana.net/otlp
GRAFANA_CLOUD_INSTANCE_ID=<your_instance_id>
GRAFANA_CLOUD_API_TOKEN=<your_glc_token>
```

Collector được cấu hình tại `src/otel-collector/otelcol-config-extras.yml` với:
- Exporter `otlp_http/grafana_cloud`
- Extension `basicauth/grafana_cloud`
- Pipelines traces/metrics/logs/profiles đều export cloud

## Chạy project
Khởi động app không observability stack local:

```bash
make start-no-o11y
```

Dừng toàn bộ:

```bash
make stop
```

Xem log collector:

```bash
docker compose --env-file .env --env-file .env.override -f compose.yaml -f compose.full.yaml -f compose.extras.yaml logs -f otel-collector
```

## Kiểm tra trên Grafana Cloud
Sau khi app có traffic (từ user thao tác hoặc load-generator), vào Explore và lọc theo:
- `service.name = frontend`
- `service.name = checkout`
- `service.name = product-catalog`

Nếu không thấy dữ liệu:
1. Kiểm tra token Grafana Cloud còn hạn và có scope ingest.
2. Kiểm tra log `otel-collector` có lỗi 401/403/timeout không.
3. Kiểm tra biến trong `.env.override` đã đúng region endpoint.
