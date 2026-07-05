# Log Analysis Assistant - OpenTelemetry Demo Fork

## Tong quan
Day la fork cua OpenTelemetry Astronomy Shop duoc tuy chinh cho bai toan CNPM:

- Chay ung dung microservices ecommerce demo.
- Thu thap telemetry (traces, metrics, logs, profiles) bang OpenTelemetry.
- Gui du lieu giamsat len Grafana Cloud thong qua OTel Collector local.

## Thong tin project
- Repository: https://github.com/corgilovetea26/log-analysis-demo
- Thu muc local: /Users/manhcuongizme/develop/hust/cnpm/opentelemetry-demo
- Runtime mode dang dung: khong chay local observability stack (Jaeger/Grafana/Prometheus local).

## Kien truc telemetry dang ap dung
1. Cac service gui OTLP ve `otel-collector` local trong Docker network.
2. `otel-collector` export len Grafana Cloud.
3. Frontend va backend deu di qua collector, tranh thieu du lieu telemetry.

## Cau hinh Grafana Cloud
Cap nhat file `.env.override` (khong commit file nay):

```dotenv
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
GRAFANA_CLOUD_OTLP_ENDPOINT=https://otlp-gateway-<region>.grafana.net/otlp
GRAFANA_CLOUD_INSTANCE_ID=<your_instance_id>
GRAFANA_CLOUD_API_TOKEN=<your_glc_token>
```

Collector duoc cau hinh tai `src/otel-collector/otelcol-config-extras.yml` voi:
- Exporter `otlp_http/grafana_cloud`
- Extension `basicauth/grafana_cloud`
- Pipelines traces/metrics/logs/profiles deu export cloud

## Chay project
Khoi dong app khong observability stack local:

```bash
make start-no-o11y
```

Dung toan bo:

```bash
make stop
```

Xem log collector:

```bash
docker compose --env-file .env --env-file .env.override -f compose.yaml -f compose.full.yaml -f compose.extras.yaml logs -f otel-collector
```

## Kiem tra tren Grafana Cloud
Sau khi app co traffic (tu user thao tac hoac load-generator), vao Explore va loc theo:
- `service.name = frontend`
- `service.name = checkout`
- `service.name = product-catalog`

Neu khong thay du lieu:
1. Kiem tra token Grafana Cloud con han va co scope ingest.
2. Kiem tra log `otel-collector` co loi 401/403/timeout khong.
3. Kiem tra bien trong `.env.override` da dung region endpoint.

## Nguon tham khao
- OpenTelemetry Demo goc: https://github.com/open-telemetry/opentelemetry-demo
- OpenTelemetry docs: https://opentelemetry.io/docs/
- Grafana Cloud OTLP ingest: https://grafana.com/docs/grafana-cloud/send-data/otlp/
