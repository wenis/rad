---
name: observability-setup
description: Sets up comprehensive observability including structured logging, metrics, distributed tracing, error tracking, and dashboards. Use when deploying to production OR investigating issues OR setting up monitoring OR improving system visibility.
allowed-tools: Read, Write, Edit, Bash
---

# Observability Setup

You implement comprehensive observability (logs, metrics, traces, errors) to ensure production systems are visible, debuggable, and maintainable.

## When to use
- Setting up a new production service
- Investigating production issues
- Implementing SRE best practices
- Adding monitoring to existing services
- Preparing for scale/growth
- Improving incident response time

## The Three Pillars of Observability

### 1. Logs (What happened?)
Timestamped records of events.

**Use for:** Debugging specific issues, audit trails

### 2. Metrics (How much/how many?)
Numerical measurements over time.

**Use for:** Dashboards, alerts, capacity planning

### 3. Traces (Where did time go?)
Request flow through distributed systems.

**Use for:** Performance debugging, understanding dependencies

**Plus:** Error tracking and alerting

## Tech Stack Recommendations

### All-in-One Solutions

**Datadog** (Easiest, paid)
- Logs, metrics, traces, errors in one platform
- Great UX, powerful dashboards
- Auto-instrumentation available
- Cost: ~$15-$31/host/month

**New Relic** (Comprehensive, paid)
- Full-stack observability
- AI-powered insights
- Good for enterprises
- Cost: ~$25-$99/user/month

**Elastic (ELK) Stack** (Self-hosted, complex)
- Elasticsearch + Logstash/Filebeat + Kibana
- Powerful search and analytics
- Free but requires infrastructure
- Steep learning curve

### Open Source Stack (Recommended for startups)

**Logs**: Loki (lightweight, Prometheus-style)
**Metrics**: Prometheus + Grafana
**Traces**: Tempo or Jaeger
**Errors**: Sentry (free tier)
**Dashboards**: Grafana

**Pros:** Free, flexible, vendor-independent
**Cons:** Requires setup and maintenance

### Simple Stack (Minimum viable)

**Logs**: CloudWatch Logs / Stackdriver
**Metrics**: CloudWatch / Cloud Monitoring
**Errors**: Sentry
**Dashboards**: Grafana Cloud (free tier)

## Implementation Examples

### 1. Structured Logging

**Python (structlog):**
```python
# logging_config.py
import structlog
import logging

def setup_logging():
    logging.basicConfig(
        format="%(message)s",
        level=logging.INFO,
    )

    structlog.configure(
        processors=[
            structlog.stdlib.filter_by_level,
            structlog.stdlib.add_logger_name,
            structlog.stdlib.add_log_level,
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.processors.StackInfoRenderer(),
            structlog.processors.format_exc_info,
            structlog.processors.JSONRenderer()
        ],
        wrapper_class=structlog.stdlib.BoundLogger,
        context_class=dict,
        logger_factory=structlog.stdlib.LoggerFactory(),
        cache_logger_on_first_use=True,
    )

# Usage
import structlog

logger = structlog.get_logger()

# Structured logging
logger.info(
    "user_login",
    user_id=user.id,
    email=user.email,
    ip_address=request.ip,
    user_agent=request.headers.get("User-Agent")
)

# With context
logger = logger.bind(request_id=request_id, user_id=user_id)
logger.info("processing_payment", amount=100.00, currency="USD")
logger.error("payment_failed", error=str(e), amount=100.00)
```

**Node.js (pino):**
```typescript
// logger.ts
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label }),
  },
  timestamp: pino.stdTimeFunctions.isoTime,
  ...(process.env.NODE_ENV === 'production'
    ? {}
    : { transport: { target: 'pino-pretty' } }),
});

// Usage
import { logger } from './logger';

logger.info({
  event: 'user_login',
  userId: user.id,
  email: user.email,
  ipAddress: req.ip,
}, 'User logged in');

logger.error({
  event: 'payment_failed',
  userId: user.id,
  amount: 100.00,
  error: error.message,
  stack: error.stack,
}, 'Payment processing failed');
```

**Go (zerolog):**
```go
// logger.go
package main

import (
    "github.com/rs/zerolog"
    "github.com/rs/zerolog/log"
    "os"
)

func SetupLogger() {
    zerolog.TimeFieldFormat = zerolog.TimeFormatUnix

    if os.Getenv("ENV") == "development" {
        log.Logger = log.Output(zerolog.ConsoleWriter{Out: os.Stderr})
    }
}

// Usage
log.Info().
    Str("event", "user_login").
    Str("user_id", user.ID).
    Str("email", user.Email).
    Str("ip_address", r.RemoteAddr).
    Msg("User logged in")

log.Error().
    Err(err).
    Str("event", "payment_failed").
    Float64("amount", 100.00).
    Msg("Payment processing failed")
```

### 2. Metrics with Prometheus

**Python (prometheus_client):**
```python
# metrics.py
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time

# Define metrics
http_requests_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

http_request_duration_seconds = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

active_users = Gauge(
    'active_users',
    'Number of active users'
)

# Middleware
def track_request(method, endpoint):
    def decorator(func):
        def wrapper(*args, **kwargs):
            start = time.time()
            try:
                result = func(*args, **kwargs)
                status = 200
                return result
            except Exception as e:
                status = 500
                raise
            finally:
                duration = time.time() - start
                http_requests_total.labels(
                    method=method,
                    endpoint=endpoint,
                    status=status
                ).inc()
                http_request_duration_seconds.labels(
                    method=method,
                    endpoint=endpoint
                ).observe(duration)
        return wrapper
    return decorator

# Usage
@track_request('GET', '/api/users')
def get_users():
    # Handler logic
    return users

# Start metrics server
if __name__ == '__main__':
    start_http_server(8000)  # Metrics on :8000/metrics
```

**Node.js (prom-client):**
```typescript
// metrics.ts
import promClient from 'prom-client';

// Enable default metrics (CPU, memory, etc.)
promClient.collectDefaultMetrics();

export const httpRequestsTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status'],
});

export const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'route'],
  buckets: [0.1, 0.5, 1, 2, 5],
});

export const activeConnections = new promClient.Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
});

// Middleware
export function metricsMiddleware(req, res, next) {
  const start = Date.now();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;

    httpRequestsTotal.labels(req.method, req.route?.path || req.path, res.statusCode).inc();
    httpRequestDuration.labels(req.method, req.route?.path || req.path).observe(duration);
  });

  next();
}

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(await promClient.register.metrics());
});
```

### 3. Distributed Tracing (OpenTelemetry)

**Python:**
```python
# tracing.py
from opentelemetry import trace
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
from opentelemetry.instrumentation.sqlalchemy import SQLAlchemyInstrumentor

def setup_tracing(app, service_name="my-service"):
    # Set up tracer
    trace.set_tracer_provider(TracerProvider())
    tracer = trace.get_tracer(__name__)

    # Export to Tempo/Jaeger
    otlp_exporter = OTLPSpanExporter(
        endpoint="http://localhost:4317",
        insecure=True
    )
    span_processor = BatchSpanProcessor(otlp_exporter)
    trace.get_tracer_provider().add_span_processor(span_processor)

    # Auto-instrument FastAPI
    FastAPIInstrumentor.instrument_app(app)

    # Auto-instrument SQLAlchemy
    SQLAlchemyInstrumentor().instrument(engine=engine)

    return tracer

# Manual instrumentation
tracer = trace.get_tracer(__name__)

@tracer.start_as_current_span("process_order")
def process_order(order_id):
    span = trace.get_current_span()
    span.set_attribute("order.id", order_id)
    span.set_attribute("order.amount", order.amount)

    try:
        result = payment_service.charge(order)
        span.set_attribute("payment.status", "success")
        return result
    except Exception as e:
        span.set_status(Status(StatusCode.ERROR))
        span.record_exception(e)
        raise
```

**Node.js:**
```typescript
// tracing.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-grpc';

export function setupTracing(serviceName: string) {
  const sdk = new NodeSDK({
    serviceName,
    traceExporter: new OTLPTraceExporter({
      url: 'http://localhost:4317',
    }),
    instrumentations: [getNodeAutoInstrumentations()],
  });

  sdk.start();

  process.on('SIGTERM', () => {
    sdk.shutdown().then(() => process.exit(0));
  });
}

// Manual spans
import { trace } from '@opentelemetry/api';

const tracer = trace.getTracer('my-service');

async function processOrder(orderId: string) {
  return await tracer.startActiveSpan('process_order', async (span) => {
    span.setAttribute('order.id', orderId);

    try {
      const result = await paymentService.charge(order);
      span.setAttribute('payment.status', 'success');
      return result;
    } catch (error) {
      span.recordException(error);
      span.setStatus({ code: SpanStatusCode.ERROR });
      throw error;
    } finally {
      span.end();
    }
  });
}
```

### 4. Error Tracking (Sentry)

**Python:**
```python
# sentry_config.py
import sentry_sdk
from sentry_sdk.integrations.fastapi import FastApiIntegration
from sentry_sdk.integrations.sqlalchemy import SqlalchemyIntegration

def setup_sentry(app):
    sentry_sdk.init(
        dsn=os.environ['SENTRY_DSN'],
        environment=os.environ.get('ENV', 'development'),
        traces_sample_rate=0.1,  # 10% of transactions traced
        profiles_sample_rate=0.1,
        integrations=[
            FastApiIntegration(),
            SqlalchemyIntegration(),
        ],
    )

# Usage - automatic for unhandled exceptions
# Manual error capture
from sentry_sdk import capture_exception, capture_message

try:
    risky_operation()
except Exception as e:
    capture_exception(e)
    # Also add context
    sentry_sdk.set_context("order", {
        "id": order.id,
        "amount": order.amount,
    })
    sentry_sdk.set_user({"id": user.id, "email": user.email})
    capture_exception(e)

# Breadcrumbs for debugging
sentry_sdk.add_breadcrumb(
    category='order',
    message='Order validation started',
    level='info',
)
```

**Node.js:**
```typescript
// sentry.ts
import * as Sentry from '@sentry/node';
import { ProfilingIntegration } from '@sentry/profiling-node';

export function setupSentry(app: Express) {
  Sentry.init({
    dsn: process.env.SENTRY_DSN,
    environment: process.env.NODE_ENV,
    tracesSampleRate: 0.1,
    profilesSampleRate: 0.1,
    integrations: [
      new Sentry.Integrations.Http({ tracing: true }),
      new Sentry.Integrations.Express({ app }),
      new ProfilingIntegration(),
    ],
  });

  // Request handler must be first
  app.use(Sentry.Handlers.requestHandler());
  app.use(Sentry.Handlers.tracingHandler());

  // Routes...

  // Error handler must be last
  app.use(Sentry.Handlers.errorHandler());
}

// Manual error capture
import * as Sentry from '@sentry/node';

try {
  await riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { section: 'payment' },
    extra: { orderId, amount },
    user: { id: user.id, email: user.email },
  });
}

// Breadcrumbs
Sentry.addBreadcrumb({
  category: 'order',
  message: 'Order validation started',
  level: 'info',
});
```

### 5. Dashboard Setup (Grafana)

**Prometheus + Grafana docker-compose:**
```yaml
# docker-compose.observability.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    depends_on:
      - prometheus

  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yml:/etc/loki/local-config.yaml
      - loki-data:/loki

  tempo:
    image: grafana/tempo:latest
    ports:
      - "3200:3200"   # Tempo HTTP
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
    volumes:
      - ./tempo-config.yml:/etc/tempo.yaml
      - tempo-data:/tmp/tempo
    command: ["-config.file=/etc/tempo.yaml"]

volumes:
  prometheus-data:
  grafana-data:
  loki-data:
  tempo-data:
```

**Prometheus config:**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'my-app'
    static_configs:
      - targets: ['app:8000']  # Your app metrics endpoint

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

**Grafana dashboard JSON:**
```json
{
  "dashboard": {
    "title": "Application Metrics",
    "panels": [
      {
        "title": "Request Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])"
          }
        ],
        "type": "graph"
      },
      {
        "title": "Error Rate",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
          }
        ],
        "type": "graph"
      },
      {
        "title": "P95 Latency",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
          }
        ],
        "type": "graph"
      }
    ]
  }
}
```

## Alerting Setup

**Prometheus AlertManager:**
```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'slack'

receivers:
  - name: 'slack'
    slack_configs:
      - api_url: 'YOUR_SLACK_WEBHOOK_URL'
        channel: '#alerts'
        title: 'Alert: {{ .CommonLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

# Alert rules
groups:
  - name: app_alerts
    interval: 30s
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          description: 'Error rate is {{ $value }} (threshold: 0.05)'

      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          description: 'P95 latency is {{ $value }}s (threshold: 1s)'
```

## Complete Setup Checklist

### Application Code
- [ ] Add structured logging (JSON format)
- [ ] Instrument metrics (requests, latency, errors)
- [ ] Add distributed tracing
- [ ] Integrate error tracking (Sentry)
- [ ] Add health check endpoint
- [ ] Add metrics endpoint (/metrics)

### Infrastructure
- [ ] Deploy Prometheus for metrics
- [ ] Deploy Grafana for dashboards
- [ ] Deploy Loki for logs (or use CloudWatch)
- [ ] Deploy Tempo/Jaeger for traces
- [ ] Set up AlertManager
- [ ] Configure log aggregation

### Dashboards
- [ ] Request rate
- [ ] Error rate
- [ ] Latency (P50, P95, P99)
- [ ] Resource usage (CPU, memory)
- [ ] Database performance
- [ ] Business metrics (signups, purchases, etc.)

### Alerts
- [ ] High error rate (> 5%)
- [ ] High latency (P95 > 1s)
- [ ] Service down
- [ ] High memory usage (> 80%)
- [ ] Database connection pool exhausted

## Best Practices

✅ **DO:**
- Use structured logging (JSON)
- Include request IDs in all logs
- Sample traces (10-30%) to reduce overhead
- Create dashboards for your SLIs
- Alert on symptoms, not causes
- Set up on-call rotation
- Review dashboards regularly

❌ **DON'T:**
- Log sensitive data (passwords, tokens)
- Trace 100% of requests (expensive)
- Create alerts that fire constantly
- Alert on metrics you can't act on
- Ignore alert fatigue
- Log excessively (DEBUG in production)

## Instructions

1. **Choose stack**: All-in-one (Datadog) vs open source (Prometheus + Grafana)
2. **Implement logging**: Add structured logging library
3. **Add metrics**: Instrument key operations
4. **Set up tracing**: OpenTelemetry auto-instrumentation
5. **Integrate error tracking**: Sentry or similar
6. **Deploy infrastructure**: Docker Compose or cloud services
7. **Create dashboards**: Key metrics visualization
8. **Configure alerts**: High error rate, latency, downtime
9. **Test**: Generate load, trigger errors, verify visibility
10. **Document**: Runbook for common issues

## Constraints

- Must not log sensitive data (PII, secrets)
- Metrics endpoint must not require auth (for Prometheus)
- Tracing overhead must be < 5% (use sampling)
- Logs must be structured (JSON)
- Must include request IDs for correlation
- Dashboards must load in < 3 seconds
