# Infrastructure Monitoring Stack

A production-oriented infrastructure monitoring stack built using Promethus, Grafana, Alertmanager and exporters.

## Objectives

- Monitor infrastructure and system resources
- Collect and query time-series metrics
- Build operational dashboards
- Configure alerting for infrastructure failure
- Stimulate production incidents
- Practice troubleshooting and incident response
- Document monitoring architecture and operational procedures

## Technology Stack

- Prometheus
- Grafana
- Alertmanager
- Node Exporter
- Docker
- Linux
- Git


## Project Architecture


The monitoring architecture will be documented as the project evolves.


## Monitoring Capabilities

- CPU utilization
- Memory utilization
- Disk utilization
- System load
- Network statistics
- Service availabiltiy
- Container metrics


## Incident Scenarios

THe project will include controlled simulation such as:

- High CPU utilization
- Memory pressure
- Disk exhaustion
- Service failure
- Container failure
- High resource utilization

## Repository Structure

```text
prometheus/   Prometheus configuration
grafana/      Grafana configuration and dashboards
alertmanager/ Alertmanager configuration
scripts/      Operational and testing scripts
docs/         Architecture and incident documentation
