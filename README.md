# Monitoring Stack: OpenTelemetry + Prometheus + Grafana

A simple 3-service monitoring setup for collecting, storing, and visualizing telemetry data.

## Services

### 📡 OpenTelemetry Collector
- **Purpose**: Receives and processes telemetry data (traces, metrics, logs)
- **OTLP gRPC**: Port 4317
- **OTLP HTTP**: Port 4318
- **Metrics endpoint**: Port 8889 (for Prometheus)
- **Health check**: Port 13133

### 📊 Prometheus
- **Purpose**: Time-series database for metrics
- **Web UI**: Port 9090
- **Data retention**: 200 hours

### 📈 Grafana
- **Purpose**: Visualization and dashboards
- **Web UI**: Port 3000
- **Default login**: admin/admin123

## Quick Start

### 1. Start the stack
```bash
docker-compose -f docker-compose.yml up -d
```

### 2. Access the services
- **Grafana**: http://localhost:3000 (admin/admin123)
- **Prometheus**: http://localhost:9090
- **OpenTelemetry Health**: http://localhost:13133

### 3. Send test data to OpenTelemetry

#### Test with OTLP HTTP:
```bash
curl -X POST http://localhost:4318/v1/traces \
  -H "Content-Type: application/json" \
  -d '{
    "resourceSpans": [{
      "resource": {
        "attributes": [{
          "key": "service.name",
          "value": {"stringValue": "test-app"}
        }]
      },
      "scopeSpans": [{
        "spans": [{
          "traceId": "12345678901234567890123456789012",
          "spanId": "1234567890123456",
          "name": "test-operation",
          "kind": 1,
          "startTimeUnixNano": "1609459200000000000",
          "endTimeUnixNano": "1609459201000000000"
        }]
      }]
    }]
  }'
```

#### Send metrics:
```bash
curl -X POST http://localhost:4318/v1/metrics \
  -H "Content-Type: application/json" \
  -d '{
    "resourceMetrics": [{
      "resource": {
        "attributes": [{
          "key": "service.name",
          "value": {"stringValue": "test-app"}
        }]
      },
      "scopeMetrics": [{
        "metrics": [{
          "name": "test_counter",
          "sum": {
            "dataPoints": [{
              "value": 42,
              "timeUnixNano": "1609459200000000000"
            }],
            "aggregationTemporality": 2
          }
        }]
      }]
    }]
  }'
```

## Data Flow

```
Your Application → OpenTelemetry Collector → Prometheus → Grafana
     (OTLP)              (processes)        (stores)   (visualizes)
```

## Configuration Files

- `config/otel-collector.yaml` - OpenTelemetry Collector configuration
- `config/prometheus.yml` - Prometheus scraping configuration  
- `config/grafana/datasources/` - Grafana data source configuration

## Management Commands

### View logs
```bash
docker-compose -f docker-compose.yml logs -f
```

### Stop services
```bash
docker-compose -f docker-compose.yml down
```

### Clean everything (removes data volumes)
```bash
docker-compose -f docker-compose.yml down -v
```

### Restart specific service
```bash
docker-compose -f docker-compose.yml restart grafana
```

## Next Steps

1. **Configure your applications** to send telemetry to:
   - OTLP gRPC: `http://localhost:4317`
   - OTLP HTTP: `http://localhost:4318`

2. **Create dashboards** in Grafana using the Prometheus data source

3. **Monitor metrics** in Prometheus at http://localhost:9090/targets

This setup provides a solid foundation for observability with industry-standard tools!
