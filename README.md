## Cloud Architecture Showcase

A portfolio-friendly collection of cloud architecture case studies and diagrams. Each case study is a production‑minded design with rationale, security posture, and CI/CD strategy, written in a concise, engineering‑report style.

- **Focus**: High‑performance, horizontally scalable, resilient systems with strong observability
- **Vendors**: Starts with AWS; more providers may be added over time

---

### Highlights

- **Case Study 1 – AWS Microservices (ECS on Fargate)**
  - Route 53 → CloudFront → ALB (TLS termination, path‑based routing)
  - Microservices on ECS Fargate (User, Payment, Notification)
  - Polyglot persistence: Aurora (relational), DynamoDB (NoSQL) + ElastiCache
  - Async decoupling via SQS; observability via CloudWatch
  - CI/CD: CodeCommit → CodePipeline → CodeBuild → ECR → ECS

Preview:

![AWS Microservices – ECS on Fargate](cloud/Arsitektur-Microservices.drawio.png)

- Read the full write‑up: `cloud/README.md`
- Diagram source: `cloud/Arsitektur-Microservices.drawio.png` (created with diagrams.net/draw.io)

---

### Repository Structure

```
cloud-architecture-showcase/
├─ README.md                        # This file
└─ cloud/
   ├─ README.md                     # Case Study 1: AWS Microservices on ECS Fargate
   └─ Arsitektur-Microservices.drawio.png  # Diagram (PNG)
```

---

### How to Use

- **Browse**: Open each case study’s `README.md` for the narrative, diagrams, and rationale
- **Reuse**: Copy architecture sections (components, rationale, security, CI/CD) as templates for your own projects
- **Export**: Edit diagrams in diagrams.net (or Lucidchart) and export to PNG/SVG for your portfolio or docs

---

### Roadmap

- EKS‑based variant (service mesh, gRPC, canary/blue‑green)
- Serverless variant (API Gateway + Lambda + EventBridge)
- Multi‑Region active/passive with Route 53 ARC and data replication strategies
- Deeper observability with OpenTelemetry and Managed Prometheus/Grafana

---

### Contact

- Maintainer: Daffa Haidar
- Feedback and ideas are welcome—open an issue or PR. 
