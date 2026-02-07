<!--
**Owner:** KamoEllen
**Repo:** [Ecommerce-Devops-Pipeline](https://github.com/KamoEllen/Ecommerce-Devops-Pipeline)
**Status:** Completed

---
-->

## **Table of Contents**

1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Infrastructure Design](#infrastructure-design)
4. [Microservices Architecture](#microservices-architecture)
5. [Observability](#observability)
6. [CI/CD Pipeline](#cicd-pipeline)
7. [Key Architectural Decisions](#key-architectural-decisions)
8. [Challenges and Solutions](#challenges-and-solutions)
9. [Areas for Improvement](#areas-for-improvement)
10. [Conclusion](#conclusion)

---

## **Project Overview**

Migrated the **OpenTelemetry Demo** microservices application to **AWS EKS**, implementing:

* Secure, autoscaling infrastructure
* Observability with OTEL, Prometheus, Grafana
* Production-ready CI/CD pipelines

**Key Services:** Frontend Proxy, Product Catalog, Recommendation

**Objectives:**

* Deploy and operate a cloud-native microservices system
* Maintain scalability and observability for user-facing services
* Implement automated, zero-downtime CI/CD deployments

---

## **System Architecture**

### **High-Level Visual **

![Ecommerce DevOps Pipeline Diagram](https://github.com/KamoEllen/Ecommerce-Devops-Pipeline/blob/main/diagram.png)

**Diagram Highlights:**

* **Infrastructure:** AWS VPC, EKS cluster, public/private subnets, NAT Gateway, ALB Ingress
* **Microservices:** Frontend Proxy (Next.js), Product Catalog (Go + OTEL), Recommendation (Go + OTEL), Checkout (Go + OTEL), Kafka Broker
* **Observability:** OTEL Collector → Prometheus → Grafana
* **CI/CD Pipeline:** GitHub Actions → Build & Test → Lint → Docker Push → K8s Deployment → Blue-Green Deployment

---

## **Infrastructure Design**

**AWS Foundation:**

* **VPC:** 10.0.0.0/16, public/private subnets across 3 AZs
* **EKS Cluster:** Private subnets, autoscaling node groups
* **ALB Ingress:** Internet-facing routing
* **Terraform State:** S3 + DynamoDB for encrypted remote state with locking

**EKS Node Configuration:**

| Parameter           | Value                |
| ------------------- | -------------------- |
| Kubernetes Version  | v1.30                |
| Node Type           | t3.medium → t3.large |
| Min / Desired / Max | 1 / 2 / 4            |
| Autoscaling         | Enabled              |

---

## **Microservices Architecture**

| Service            | Runtime       | Notes                         |
| ------------------ | ------------- | ----------------------------- |
| Frontend Proxy     | Next.js       | Public-facing, routed via ALB |
| Product Catalog    | Go + OTEL     | 20Mi, targeted scaling        |
| Recommendation     | Go + OTEL     | 500Mi, targeted scaling       |
| Checkout           | Go + OTEL     | Core functionality            |
| Kafka Broker       | Containerized | Async messaging               |
| OTEL Collector     | Containerized | Aggregates traces/metrics     |
| Prometheus/Grafana | Containerized | Observability dashboards      |

**Resource Allocation Example:**

* Product Catalog → 20Mi
* Recommendation → 500Mi
* Frontend Proxy → 50Mi

---

## **Observability**

* All services instrumented with **OTEL** (traces + metrics)
* Metrics exported to **Prometheus**, visualized on **Grafana**
* Container-level health checks for reliability

---

## **CI/CD Pipeline**

**Product Catalog Service:**

* GitHub Actions → Compile & Unit Test → Lint (golangci-lint) → Docker Build & Push → K8s Deployment (Blue-Green)

**Benefits:**

* Fully automated, zero-downtime deployments
* Reduced manual deployment risk

---

## **Key Architectural Decisions**

* Stateless microservices for simplicity
* Kafka for asynchronous cross-service communication
* Observability via OTEL Collector + Prometheus/Grafana
* Feature flags via ConfigMaps for controlled testing

---

## **Challenges and Solutions**

| Challenge                         | Solution                                     |
| --------------------------------- | -------------------------------------------- |
| Safe infrastructure collaboration | Terraform backend: S3 + DynamoDB locking     |
| Secure service communication      | NAT Gateway per AZ, private subnets          |
| Observability migration           | Unified OTEL + Prometheus/Grafana dashboards |
| Scaling services                  | Resource tuning, autoscaling node groups     |

---

## **Areas for Improvement**

* Extend CI/CD coverage to all services (multi-service pipelines, ArgoCD)
* Add persistent storage (Amazon RDS)
* Improve alerting and synthetic monitoring
* Implement HPA & Cluster Autoscaler for cost/performance optimization

---

## **Conclusion**

This project demonstrates **end-to-end ownership of a cloud-native microservices system** with production-grade DevOps practices:

* Scalable infrastructure
* Complete observability
* Secure deployment patterns
* Automated, zero-downtime CI/CD
