# A Hands-On DevOps Project: Migrating OpenTelemetry Demo to AWS

## Project Overview

I migrated the open-source OpenTelemetry Demo—a pre-built microservices application that demonstrates observability best practices—to AWS infrastructure. This was my first major end-to-end DevOps project. My objective was to deploy and operate this system in a production-relevant environment with proper scalability, observability, and fault tolerance reflective of real-world e-commerce applications.

This implementation focused heavily on building secure, autoscaling infrastructure on AWS using Terraform and EKS, while prioritizing production readiness, cost-awareness, and system observability. While I maintained all core services from the demo, I focused the majority of my work on three services that most directly impact user-facing load and traffic: **Product Catalog**, **Recommendation**, and **Frontend Proxy**.

---
## ▶Click the video below to watch

[![Watch the video](https://img.youtube.com/vi/_yjDVuh-XzA/maxresdefault.jpg)](https://www.youtube.com/watch?v=_yjDVuh-XzA)

---

## System Architecture

### Infrastructure Design

**AWS Foundation:**

* **EKS Cluster:** Deployed in private subnets for improved security
* **VPC Configuration:**

  * CIDR: `10.0.0.0/16` with DNS hostnames enabled
  * Public Subnets: `10.0.4.0/24`, `10.0.5.0/24`, `10.0.6.0/24`
  * Private Subnets: `10.0.1.0/24`, `10.0.2.0/24`, `10.0.3.0/24`
  * 3 Availability Zones
* **NAT Gateway:** One per Availability Zone for high availability
* **ALB Ingress:** Internet-facing with IP targets and host-based routing

**EKS Cluster Configuration:**

* Kubernetes version: `1.30`
* Node groups (started with `t3.medium`, upgraded to `t3.large` to handle service load):

  * min: 1
  * desired: 2
  * max: 4
* Autoscaling enabled

**State Management:**

* Terraform backend:

  * S3 bucket: `prroj-emo-terraform-eks-state-s3-bucket`
  * DynamoDB table: `terraform-eks-state-locks`
  * Server-side encryption enabled
  * PAY\_PER\_REQUEST billing model for cost-efficiency

### Microservices Architecture

The OpenTelemetry demo includes a range of services, all of which were containerized and deployed within the EKS cluster. I placed primary focus on the **Product Catalog**, **Recommendation**, and **Frontend Proxy** services, implementing targeted scaling policies, telemetry instrumentation, and memory/resource tuning to simulate production load.

Services deployed:

* Frontend
* Product Catalog
* Recommendation
* Checkout
* Shipping
* Payment
* OpenTelemetry Collector
* Grafana & Prometheus

### Architecture Diagram

```
┌────────────┐   ┌─────────────────┐   ┌────────────────┐
│  Frontend  │   │    Load Balancer│   │ ALB Ingress    │
│ (Next.js)  │──▶│ (Envoy + ALB)   │──▶│ + Host Routing │
└────────────┘   └─────────────────┘   └────────────────┘
        │                    │                   │
┌───────▼────────┐   ┌───────▼────────┐   ┌───────▼────────┐
│ Product Catalog│   │ Recommendation │   │ Checkout       │
│ (Go + OTEL)    │   │ (Go + OTEL)    │   │ (Go + OTEL)    │
└────────────────┘   └────────────────┘   └────────────────┘
   ▲        ▲
   │        │
   │        └───▶ Intensive work done on deployment and scaling
   │
   └────────────▶ Core focus of architecture tuning and monitoring
        │                    │                   │
        ▼                    ▼                   ▼
   ┌──────────────┐   ┌──────────────┐   ┌─────────────────┐
   │ Kafka Broker │   │ Feature Flags│   │ OTEL Collector   │
   └──────────────┘   └──────────────┘   └─────────────────┘
```

## Key Technical Changes Implemented

### Infrastructure as Code (Terraform)

* Used modular Terraform architecture with clear separation of VPC, EKS, and state modules
* Remote backend via encrypted S3 bucket and DynamoDB state locking
* AWS-specific tagging enabled service discovery and monitoring integrations

### Kubernetes Configuration

* Renamed services for consistency and namespace clarity (e.g., `recommendation` → `opentelemetry-demo-recommendationservice`)
* Tuned memory and CPU resources based on observed service needs

  * Product Catalog: 20Mi
  * Recommendation: 500Mi
  * Frontend Proxy: 50Mi
* Used official and custom container images as needed

### Observability

* Maintained and enhanced OpenTelemetry Collector configuration

  * Standardized environment variables across services
  * Enabled OTLP gRPC protocol for better telemetry delivery
* Retained core Grafana dashboards and Prometheus metrics (e.g., `app_recommendations_counter_total`)
* Implemented structured logging and container-level health checks

### Deployment Strategy

### CI/CD Pipeline

The Product Catalog service is integrated with a GitHub Actions CI/CD pipeline defined in `.github/workflows/product-catalog-ci.yaml`. The pipeline automates the full lifecycle of changes to this service:

* **Build & Unit Test:** Go code is compiled and tested on each pull request to `main`.
* **Linting:** `golangci-lint` ensures code adheres to quality standards.
* **Docker Image Build:** A Docker image is built and pushed to Docker Hub using secure credentials.
* **Kubernetes Manifest Update:** The Kubernetes deployment YAML is updated with the new image tag.
* **Auto-Commit & Push:** Changes to the manifest are committed and pushed back to `main` (with a fast-forward strategy).

This pipeline ensures that any change to the product catalog service is automatically built, validated, containerized, and prepared for deployment — reducing manual intervention and deployment risk.

* Used blue-green deployments to ensure zero-downtime rollouts
* Ingress managed via ALB Ingress Controller
* Security best practices enforced: non-root containers, defined user IDs, resource quotas

### Key Architectural Decisions

* Removed PostgreSQL for simplification; simulated stateless accounting service
* Managed feature flags via ConfigMaps to simulate various failure scenarios (e.g., product catalog failure, Kafka queue errors)
* Kafka and OTEL collector deployed to ensure asynchronous communication and distributed tracing

## Challenges and Solutions

| Challenge                                       | Solution                                              |
| ----------------------------------------------- | ----------------------------------------------------- |
| Safe infrastructure collaboration               | S3 + DynamoDB backend with state locking              |
| Secure service communication in private subnets | One NAT Gateway per AZ + proper subnet tagging        |
| Maintaining observability during migration      | Unified OTEL setup with compatible metrics collection |

## Areas for Improvement

* Extend CI/CD coverage beyond Product Catalog to all services. Integrate multi-service pipelines using GitHub Actions or ArgoCD, add deployment approvals, rollback mechanisms, and cross-environment promotion.
* Add Amazon RDS to reintroduce persistent database use cases
* Expand alerting and synthetic monitoring coverage
* Add autoscaling (HPA, Cluster Autoscaler) for cost-performance optimization

## Conclusion

This project demonstrates my ability to translate theoretical DevOps knowledge into a scalable, observable, cloud-native production system. Through targeted service tuning, resilient infrastructure setup, and observability maintenance, I’ve developed a platform-ready deployment that balances tradeoffs in performance, cost, and complexity. It showcases my capacity to own and ship DevOps systems end-to-end.
