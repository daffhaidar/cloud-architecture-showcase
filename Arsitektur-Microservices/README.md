### AWS Microservices on ECS Fargate (High Performance, Scalable, Resilient)

This document presents my solution for the case study: a high‑performance microservices architecture on AWS that is horizontally scalable, resilient, observable, and independently deployable. The design intentionally favors managed services and operational simplicity using Amazon ECS with AWS Fargate.

- Services: User, Payment, Notification (extensible)
- Communication: HTTP/HTTPS via ALB; asynchronous messaging via SQS
- Data: Polyglot persistence (Aurora for relational, DynamoDB for NoSQL), with ElastiCache for in‑memory caching
- CI/CD: CodeCommit → CodePipeline → CodeBuild → ECR → ECS
- Observability: Amazon CloudWatch for metrics, logs, and traces

### Case Study 1: High‑Performance Microservices Architecture on AWS

- **Title**: Case Study: Designing a High‑Performance Microservices Architecture on AWS
- **Challenge**: DAF is developing a microservices‑based application and now needs an infrastructure that delivers high performance, horizontal scalability, and resilience. The system consists of several services (User, Payment, Notification, etc.), communicates synchronously via REST/gRPC, and adopts polyglot persistence (different databases per service).
- **Tasks**:
  - High performance (low latency, high throughput)
  - Horizontally scalable
  - Resilient to partial failures (failover, auto‑healing)
  - Strong monitoring and observability
  - CI/CD that supports independent deployments per service
- **Deliverables**:
  - Infrastructure diagram (draw.io / diagrams.net / Lucidchart / equivalent)
  - Per‑component architecture explanation
  - Rationale for AWS service choices (e.g., ECS vs EKS vs Lambda, ALB vs API Gateway)
  - Security approach (IAM, secrets, network isolation)
- **My Solution**: The following diagram illustrates the architecture I propose.

  ![Architecture – ECS on Fargate](./Arsitektur-Microservices.drawio.png)

- **Technical Explanation**: To address this challenge, I designed a layered architecture that routes users through Route 53 and CloudFront to a public ALB, which terminates TLS and performs path‑based routing to ECS services running on AWS Fargate inside private subnets. Services communicate synchronously over HTTP(S) and asynchronously via Amazon SQS. Data is managed using a polyglot approach: Amazon Aurora for relational consistency and Amazon DynamoDB for low‑latency, high‑throughput access, with Amazon ElastiCache as an in‑memory cache for hot paths. CI/CD is automated using CodeCommit → CodePipeline → CodeBuild, publishing images to Amazon ECR and rolling deployments to ECS. Amazon CloudWatch centralizes metrics, logs, and traces for visibility and alerting. I chose Fargate for its serverless operational model, ALB for cost‑effective L7 routing, and Aurora for resilient multi‑AZ relational storage. Security is enforced through VPC isolation, strict Security Groups, least‑privilege IAM roles per service, AWS Secrets Manager for secret rotation, and KMS/TLS for encryption in transit and at rest. Detailed component explanations, rationale, and security posture are provided in the sections below.

---

### Component‑by‑Component Explanation

- **Amazon Route 53 & CloudFront**

  - Users reach the application via Route 53, which routes traffic to CloudFront. As a CDN, CloudFront serves static content (images, CSS, JS) from edge locations closest to users, materially reducing latency.

- **Application Load Balancer (ALB)**

  - Deployed in public subnets as the primary entry point from CloudFront. ALB terminates TLS and intelligently distributes requests to the appropriate microservice using path‑based routing (for example, `/users` → User Service, `/payments` → Payment Service).

- **Amazon ECS on AWS Fargate**

  - Each microservice (User, Payment, Notification) runs as a Docker container orchestrated by ECS with the Fargate launch type. Fargate is serverless for containers—no server management is required. AWS handles provisioning, patching, and scaling of the underlying infrastructure.

- **Private Subnets**

  - All microservices and databases are placed in private subnets and are not directly reachable from the internet. This is a foundational security practice that protects the application and data layers.

- **Polyglot Persistence (Aurora & DynamoDB)**

  - The architecture applies the best database for each service. Amazon Aurora (MySQL/PostgreSQL‑compatible) stores relational data that requires consistency (for example, user data). Amazon DynamoDB (NoSQL) is used for ultra‑low latency and high throughput workloads (for example, payment logs or notification metadata).

- **Amazon SQS & ElastiCache**

  - Amazon SQS enables asynchronous, decoupled communication between services, improving resilience and enabling smoothing of traffic spikes. Amazon ElastiCache provides in‑memory caching to reduce database read pressure and accelerate response times for hot paths.

- **CI/CD Pipeline (AWS CodeSuite)**

  - Developers push code to AWS CodeCommit, which triggers AWS CodePipeline. AWS CodeBuild compiles, tests, and builds Docker images, then stores them in Amazon ECR (Elastic Container Registry). The pipeline deploys the new version to Amazon ECS safely.

- **Amazon CloudWatch**
  - Acts as the observability center, collecting metrics, logs, and traces from all components for monitoring, analysis, and proactive alerting.

---

### Rationale for AWS Service Choices

- **ECS on Fargate vs. EKS (Kubernetes)**

  - Fargate is chosen for operational simplicity and lower starting cost. It lets the team focus on application development without deep Kubernetes expertise. It delivers strong scalability and security out of the box. EKS is a fit for organizations standardized on Kubernetes or those requiring a highly customized ecosystem.

- **Application Load Balancer (ALB) vs. API Gateway**

  - ALB is the primary load balancer due to its efficiency and cost‑effectiveness for high‑throughput HTTP/S traffic. API Gateway excels at API management—third‑party authentication, rate limiting, and response caching—and can be placed in front of ALB if those features become critical later.

- **Amazon Aurora vs. Standard RDS**
  - Aurora offers superior performance, availability, and scalability when compared with standard RDS. With automatic data replication across three Availability Zones, Aurora provides significantly higher resilience.

---

### Security Approach

- **Network Isolation**

  - VPC with public and private subnets is the foundation. Only the components that must be internet‑facing (ALB) are placed in public subnets. Security Groups act as stateful firewalls and allow only necessary flows between services (for example, only the User Service can connect to Aurora on port 3306).

- **IAM (Identity and Access Management)**

  - The principle of least privilege is strictly enforced. Each ECS service runs with its own IAM role containing only the permissions it truly needs (for example, the Payment Service has access only to its DynamoDB tables).

- **Secrets Management**

  - Sensitive information—database credentials, API keys, tokens—is never stored in code. All secrets are stored and rotated securely using AWS Secrets Manager and accessed at runtime through IAM roles.

- **Encryption**
  - All data is encrypted both in transit (TLS) and at rest using AWS KMS across Aurora, DynamoDB, and S3.

---

### Scalability & Resilience

- Horizontal scaling of ECS services based on CPU/memory utilization and queue depth.
- SQS absorbs bursts and supports retry/DLQ patterns for robust error handling.
- Aurora replicas and DynamoDB adaptive capacity sustain high read/write throughput.
- Health checks and rolling updates enable safe deployments; failed tasks are auto‑replaced.

---

### CI/CD Summary

- Source: CodeCommit
- Orchestration: CodePipeline
- Build/Test: CodeBuild produces Docker images and pushes to ECR
- Deploy: ECS pulls from ECR and updates services without downtime

---

This ECS on Fargate solution delivers low latency via CloudFront and ALB, scales horizontally with serverless containers, remains resilient through decoupled asynchronous processing, and maintains strong security and observability by design.
