# LGTM Stack Deployment with Docker

This guide provides instructions to deploy the LGTM stack (Loki, Grafana, Tempo,
Mimir) using Docker, including environment variable configurations

## Table of Contents

- [Quick Start (Development)](#quick-start-development)
- [Production Deployment](#production-deployment)
- [Environment Variables](#environment-variables)
- [Configuration Options](#configuration-options)
- [Troubleshooting](#troubleshooting)

---

## Quick Start (Development)

For local testing/demos using the preconfigured image:

```
docker run -p 3000:3000 -p 4317:4317 -p 4318:4318 --rm -ti grafana/otel-lgtm
```

Access Grafana at `http://localhost:3000` (admin/admin).\
_Note: This image is not production-ready_[1].

---

## Compose setup

### 1. Clone Configuration

```
git clone https://github.com/amreltanawy/LGTMStackDocker
cd LGTMStackDocker
```

### 2. Docker Compose Setup

```
#docker-compose.yaml

version: '3.8'

services:
  loki:
    image: grafana/loki:2.9.4
    ports:
      - "3100:3100"
    volumes:
      - ./data/loki:/loki
      - ./config/loki/loki-config.yaml:/etc/loki/local-config.yaml
    command: -config.file=/etc/loki/local-config.yaml

  tempo:
    image: grafana/tempo:2.3.1
    ports:
      - "3200:3200"
      - "4317:4317"  # OTLP gRPC
    volumes:
      - ./data/tempo:/tmp/tempo

  mimir:
    image: grafana/mimir:2.11.0
    ports:
      - "9090:9090"  # Prometheus-compatible endpoint
    volumes:
      - ./data/mimir:/data
    command: -config.file=/etc/mimir/mimir.yaml

  grafana:
    image: grafana/grafana-oss:11.6.1
    ports:
      - "3000:3000"
    volumes:
      - ./data/grafana:/var/lib/grafana
      - ./config/grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=YourSecurePassword123!
      - GF_FEATURE_TOGGLES_ENABLE=tempoSearch tracesSampling
    depends_on:
      - loki
      - tempo
      - mimir
```

### 3. Start Stack

```
docker compose -f docker-compose.prod.yaml up -d
```

---

## Environment Variables

### Core Components

| Component   | Variable                      | Purpose            | Example                      |
| ----------- | ----------------------------- | ------------------ | ---------------------------- |
| **Grafana** | `GF_SECURITY_ADMIN_PASSWORD`  | Admin password     | `YourSecurePass123!`[4]      |
| **Loki**    | `LOKI_STORAGE_TYPE`           | Storage backend    | `s3`, `gcs`, `filesystem`[3] |
| **Tempo**   | `TEMPO_STORAGE_TRACE_BACKEND` | Trace storage      | `s3`, `gcs`[5]               |
| **Mimir**   | `MIMIR_CONFIG_YAML`           | Custom config path | `/etc/mimir/mimir.yaml`      |

### OTLP Collector

```
environment:

OTEL_EXPORTER_OTLP_ENDPOINT=http://tempo:4317

OTEL_RESOURCE_ATTRIBUTES=service.name=myapp
```

---

## Configuration Options

### 1. Authentication

#### docker-compose override example

```
services:
grafana:
environment:
- GF_AUTH_GENERIC_OAUTH_ENABLED=true
- GF_AUTH_GITHUB_CLIENT_ID=${GITHUB_CLIENT_ID}
```

### 2. External Storage

#### Loki S3 configuration

```
LOKI_STORAGE_S3_BUCKET=lgtm-logs-${ENV} #<Bucket name>
AWS_REGION=us-east-1
```

---

## For advanced configurations, refer to:

- [Loki Documentation][https://grafana.com/docs/loki/latest/]
- [Grafana Tempo Documentation][https://grafana.com/docs/tempo/latest/]
- [Mimir Documentation][https://grafana.com/docs/mimir/latest/]
- [Grafana Docker Configuration
  Docs][https://grafana.com/docs/grafana/latest/setup-grafana/configure-docker/]
- [Grafana Configuration
  Guide][https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/]
