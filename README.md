# Infrastructure Monitoring Stack

A containerized infrastructure monitoring stack built using Prometheus,
Grafana, Alertmanager, Node Exporter, and Blackbox Exporter.

The project demonstrates metrics collection, visualization, alert evaluation,
endpoint availability monitoring, and incident simulation using Docker Compose.

## Objectives

- Collect and query time-series metrics using Prometheus.
- Visualize infrastructure metrics using Grafana dashboards.
- Configure alert rules for infrastructure and endpoint failures.
- Route firing alerts through Alertmanager.
- Monitor HTTP endpoint availability using Blackbox Exporter.
- Simulate failures and validate monitoring and alerting behavior.
- Practice troubleshooting and incident response workflows.

## Technology Stack

| Technology | Purpose |
|---|---|
| Prometheus | Metrics collection, storage, PromQL queries, and alert evaluation |
| Grafana | Metrics visualization and operational dashboards |
| Alertmanager | Alert grouping, routing, and notification management |
| Node Exporter | Exposes system metrics for Prometheus |
| Blackbox Exporter | Probes HTTP endpoints and exposes probe metrics |
| Docker | Runs the monitoring components as containers |
| Docker Compose | Defines and manages the multi-container stack |
| Linux | Configuration, command-line operations, and troubleshooting |
| Git | Version control |

## Architecture

```text
                 Monitored HTTP Endpoints
                    Google / GitHub
                           |
                           v
                  +-------------------+
                  | Blackbox Exporter |
                  +-------------------+
                           |
                           | Probe metrics
                           v
+----------------+   +-------------------+
| Node Exporter  |-->|    Prometheus     |
+----------------+   +-------------------+
                             |
                 +-----------+-----------+
                 |                       |
                 v                       v
          +-------------+         +-------------+
          |   Grafana   |         | Alertmanager|
          +-------------+         +-------------+
                                        |
                                        v
                              Configured Receivers
```

Prometheus scrapes metrics from Node Exporter and Blackbox Exporter.
Grafana queries Prometheus to visualize collected metrics.

Prometheus evaluates configured alert rules and sends firing alerts to
Alertmanager, which groups and routes them to configured receivers.

## Monitoring Capabilities

### Resource Monitoring

The Grafana dashboard includes panels for:

- CPU utilization
- Memory utilization
- Filesystem utilization
- Network receive throughput

Additional PromQL queries can be used to inspect system load and other
available metrics.

### HTTP Endpoint Monitoring

Blackbox Exporter performs HTTP probes against configured endpoints.

The `probe_success` metric indicates whether a probe succeeded:

- `1`: Probe succeeded.
- `0`: Probe failed.

Prometheus collects these metrics and evaluates endpoint availability alerts.

### Alerting

The configured Prometheus rules cover:

| Alert | Condition |
|---|---|
| `HighCPUUsage` | CPU utilization exceeds 80% for 5 minutes |
| `HighMemoryUsage` | Memory utilization exceeds 90% for 5 minutes |
| `HighDiskUsage` | Filesystem utilization exceeds 85% for 5 minutes |
| `NodeExporterDown` | Node Exporter is unavailable for 1 minute |
| `HTTPEndpointDown` | An HTTP endpoint probe fails for 1 minute |

Alertmanager receives and routes firing alerts according to its configuration.

Email delivery requires working SMTP credentials and is tracked separately
from the verified Prometheus-to-Alertmanager alerting path.

## Example PromQL Queries

### CPU Utilization

```promql
100 * (
  1 - avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

### Memory Utilization

```promql
100 * (
  1 - (
    node_memory_MemAvailable_bytes
    /
    node_memory_MemTotal_bytes
  )
)
```

### Filesystem Utilization

```promql
100 * (
  1 -
  node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"}
  /
  node_filesystem_size_bytes{fstype!~"tmpfs|overlay"}
)
```

### Network Receive Throughput

```promql
rate(node_network_receive_bytes_total[5m])
```

### HTTP Probe Status

```promql
probe_success
```

### Node Exporter Availability

```promql
up{job="node-exporter"}
```

## Incident Testing

The project includes controlled testing of monitoring and alerting behavior.

### Node Exporter Failure

- Stop Node Exporter.
- Observe its `up` metric become `0`.
- Verify that `NodeExporterDown` fires.
- Confirm that Alertmanager receives the alert.
- Restart Node Exporter and verify recovery.

### HTTP Endpoint Failure

- Temporarily configure an intentionally invalid test endpoint.
- Observe `probe_success` become `0`.
- Verify that `HTTPEndpointDown` enters the firing state.
- Confirm that Alertmanager receives the alert.
- Remove the test endpoint and verify recovery.

These tests validate the monitoring and alert evaluation path without
intentionally disrupting real production services.

## Repository Structure

```text
infrastructure-monitoring/
├── alertmanager/
│   └── alertmanager.yml
├── blackbox/
│   └── blackbox.yml
├── grafana/
├── prometheus/
│   ├── prometheus.yml
│   └── rules/
│       └── alerts.yml
├── scripts/
├── docs/
├── .gitignore
├── docker-compose.yml
└── README.md
```

## Running the Stack

### Prerequisites

- Docker Desktop with WSL integration, or a compatible Docker Engine.
- Docker Compose.
- Git.

### Start the Services

Clone the repository and navigate to the project directory.

Start the stack:

```bash
docker compose up -d
```

Check service status:

```bash
docker compose ps
```

Inspect logs:

```bash
docker compose logs --tail=50 prometheus
```

### Access the Interfaces

| Service | Local URL |
|---|---|
| Prometheus | http://localhost:9090 |
| Grafana | http://localhost:3000 |
| Alertmanager | http://localhost:9093 |
| Blackbox Exporter | http://localhost:9115 |

Node Exporter exposes its metrics on port `9100`.

### Validate the Configuration

Validate Prometheus configuration:

```bash
docker compose exec prometheus \
  promtool check config /etc/prometheus/prometheus.yml
```

Validate alert rules:

```bash
docker compose exec prometheus \
  promtool check rules /etc/prometheus/rules/alerts.yml
```

Validate Docker Compose configuration:

```bash
docker compose config --quiet
```

## Troubleshooting

### Target Is Down

Inspect Prometheus targets at:

http://localhost:9090/targets

Check the relevant service logs:

```bash
docker compose logs --tail=50 prometheus
docker compose logs --tail=50 blackbox-exporter
```

### Alert Is Not Firing

- Check the PromQL expression in the Prometheus UI.
- Inspect the alert state at `/alerts`.
- Inspect configured rules at `/rules`.
- Verify the alert's `for` duration has elapsed.
- Confirm that Prometheus has an Alertmanager configured.

### Historical Metrics Remain Visible

Prometheus stores time-series data. Removing a scrape target does not
immediately erase its historical metrics.

Distinguish between historical samples and active scrape targets when
troubleshooting.

## Environment and Limitations

This project was developed using Ubuntu under WSL2 and Docker Desktop.

The current Node Exporter configuration exposes metrics visible from its
container environment. It should not be treated as a complete representation
of the Windows host's physical resources.

For production Linux hosts, deploy Node Exporter with an appropriate
host-level configuration and secure its network exposure.

Production deployment would also require appropriate access controls,
network restrictions, secret management, persistent storage planning,
and a tested notification configuration.

## Future Improvements

- Complete and verify SMTP email notifications.
- Add automated deployment and configuration validation to CI.
- Add further dashboards and operational documentation.
- Evaluate persistent storage, retention, and backup requirements.
- Harden service access and secret management.
