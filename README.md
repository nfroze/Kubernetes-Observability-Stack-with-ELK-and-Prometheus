# Kubernetes Observability Stack with ELK and Prometheus

A complete logging and metrics platform deployed on AWS EKS, combining the ELK stack for centralised log aggregation with Prometheus and Grafana for real-time metrics monitoring.

## Overview

Production systems generate vast amounts of logs and metrics across distributed services. Without centralised observability, debugging issues becomes a time-consuming process of SSH-ing into individual nodes and grepping through scattered log files. This project solves that by deploying a unified observability platform where all cluster logs flow into Elasticsearch for searching and analysis, while Prometheus scrapes metrics for performance monitoring.

The infrastructure provisions an EKS cluster using Terraform with environment-specific configurations for dev, staging, and production. On top of that, the Kubernetes layer deploys the ELK stack (Elasticsearch, Logstash, Kibana) via Helm, with Fluentd running as a DaemonSet to collect container logs from every node. Prometheus and Grafana handle the metrics side, scraping cluster metrics and visualising them through dashboards.

This demonstrates production-grade thinking: namespace isolation between monitoring and logging workloads, RBAC-scoped service accounts for Fluentd, resource limits on all containers, and environment-specific infrastructure sizing (t3.medium for dev, t3.large for production with higher node counts).

## Architecture

Logs flow from containers through the Fluentd DaemonSet, which tails `/var/log/containers/*.log` on each node, enriches entries with Kubernetes metadata (pod name, namespace, labels), and forwards them to Elasticsearch. Logstash provides an additional processing layer for beats input, while Kibana exposes the search and visualisation interface.

Metrics flow separately through Prometheus, which scrapes endpoints across the cluster using service discovery. Grafana connects to Prometheus as a data source for building dashboards. Both stacks run in isolated namespaces (logging and monitoring) to maintain clear boundaries.

The Terraform layer manages the VPC, subnets, NAT gateway, and EKS cluster. Environment-specific tfvars files control infrastructure sizing—dev uses a single NAT gateway and smaller nodes, while production uses multiple NAT gateways across availability zones with larger instances and higher node counts.

## Tech Stack

**Infrastructure**: AWS EKS, VPC, Terraform  
**Logging**: Elasticsearch, Logstash, Kibana, Fluentd  
**Monitoring**: Prometheus, Grafana  
**Orchestration**: Kubernetes, Helm  

## Key Decisions

- **Fluentd DaemonSet over sidecar pattern**: DaemonSet runs one collector per node rather than one per pod, reducing resource overhead and simplifying configuration. Trade-off is less granular control per application.

- **Separate namespaces for logging and monitoring**: Isolates resource quotas, RBAC policies, and failure domains. A misbehaving Elasticsearch cluster won't impact Prometheus scraping.

- **Environment-specific infrastructure sizing**: Dev uses t3.medium with 1-3 nodes and a single NAT gateway to minimise costs. Production uses t3.large with 3-6 nodes and redundant NAT gateways for availability.

- **Helm for complex stacks, raw manifests for custom resources**: ELK and Prometheus deployed via Helm values for maintainability; Fluentd uses custom manifests for fine-grained control over the DaemonSet configuration and RBAC.

## Screenshots

![EKS cluster in AWS Console](screenshots/eks-cluster.png)

![Terraform deployment](screenshots/terraform-apply.png)

![All pods running across namespaces](screenshots/all-pods-running.png)

![Grafana dashboard with metrics](screenshots/grafana-dashboard.png)

![Prometheus targets](screenshots/prometheus-targets.png)

![Kibana homepage](screenshots/kibana-homepage.png)

![Kibana Discover with logs](screenshots/kibana-discover-logs.png)

![Git feature branches](screenshots/git-branches.png)

## Author

**Noah Frost**

- Website: [noahfrost.co.uk](https://noahfrost.co.uk)
- GitHub: [github.com/nfroze](https://github.com/nfroze)
- LinkedIn: [linkedin.com/in/nfroze](https://linkedin.com/in/nfroze)