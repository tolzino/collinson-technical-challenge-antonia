# Priority Pass Taxi Integration Platform
Technical Architecture Submission

**Live Prototype:**  
https://client-solutions-architect-test.netlify.app/

This front-end demo prototype showcases a passenger flow from home -> journey planning -> taxi booking -> booking status -> terminal/lounges -> profile/entitlement.

Interactive elements include bottom navigation, journey/taxi booking flow, terminal selection, and key CTA buttons. Some actions are intentionally placeholders (e.g, profile/visit history, change airport) and do not call real APIs.

Purpose: demonstrate UX concept, screen transitions, and service integration experience before backend integration and production implementation.

---

## 📁 Repository Navigation

- 📊 [Architecture Diagrams](./diagrams)
- 👕 [T-Shirt Sizing Table](t-shirtsizing.md)

---

# Assumptions
- “Priority Pass Taxi Integration Platform” refers to the existing Priority Pass app/platform, extended with taxi booking + journey planning capabilities.
- Priority Pass already has (or can capture) the traveller’s flight details (flight number, departure time, airport).
- Taxi partner provides APIs for price estimate, availability, booking, and booking status.
- Payment processing is handled by the taxi partner or a PCI-compliant payment provider. Priority Pass does not store card details.
- MVP supports a single taxi partner, architecture supports adding more partners via adapters later.
- MVP supports airport drop-off (to airport). Airport pickup can be future scope.

---

# MVP Scope

- Show recommended departure time (based on flight + traffic/ETA).
- Show taxi options (ETA + estimated price).
- Create/cancel a taxi booking from within Priority Pass Application.
- Show booking status updates (confirmed/driver assigned/arriving).
- Two-way content surfacing: taxi app can show Priority Pass airport benefits, Priority Pass can show taxi inventory. Priority Pass benefits are exposed via partner-facing APIs, allowing taxi partners to surface relevant airport benefits contextually.


---

# Out of Scope

- Multi-taxi marketplace and dynamic partner selection
- Loyalty points redemption for taxi fares
- Full personalization and machine-learning recommendations
- Production-grade global multi-region active/active

---

# Executive Summary

This proposal outlines a cloud-native integration between Priority Pass and a global taxi partner to enhance the end-to-end airport journey experience.

The solution enables Priority Pass users to:

- Receive recommended departure times based on live flight and traffic data
- View real-time taxi availability and pricing
- Book and manage rides directly within the Priority Pass app
- Receive booking updates and notifications
- Surface contextual Priority Pass benefits during the journey

The architecture is designed around clear domain boundaries, operational resilience, and regulatory safety. It uses a GraphQL Backend-for-Frontend (BFF) to simplify mobile integration, domain-focused services to isolate responsibilities (Booking, Journey Planning, Inventory), and event-driven communication for scalable notification handling.

Payment processing is delegated to a PCI-compliant provider, ensuring that no cardholder data is stored or processed within the application. This minimizes compliance scope and reduces operational risk.

The solution prioritizes:

- Clear separation of concerns
- Scalable service design
- Idempotent and safe booking flows
- Observability and operational transparency
- Incremental extensibility for future partners

The prototype demonstrates the core user journey while the architectural design provides a production-ready foundation that can scale to additional partners, regions, and loyalty integrations over time.

---

# Priority Pass Taxi Integration Platform
## Technical Design Overview

---

## 1. Architecture Overview

### 1.1 System Context

The Priority Pass Taxi Integration Platform enables travellers to:

- Plan airport journeys
- Receive recommended departure times based on traffic and flight data
- View taxi options
- Book rides
- Track booking status
- Surface Priority Pass benefits in context of their journey

The platform integrates with:

- Taxi Partner API
- Payment Provider (PCI-compliant)
- Flight Data API
- Maps/Traffic API
- Push Notification Provider

The existing Priority Pass platform is extended to orchestrate taxi booking and journey planning by integrating with external travel services.

---

### 1.2 High-Level Architecture

The architecture follows a service-oriented approach deployed on AWS and structured around clear domain boundaries.

#### Core Containers

| Container | Responsibility |
|------------|----------------|
| GraphQL BFF / API Gateway | Single client-facing entry point |
| Booking Service | Manages taxi booking lifecycle |
| Journey Planning Service | Computes recommended departure times |
| Inventory & Partner Content Service | Retrieves availability & partner content |
| PostgreSQL | Booking persistence |
| Redis | Short-lived cache & performance optimization |
| Event Bus | Asynchronous event propagation |
| Observability | Logs, metrics, tracing |

---

### 1.3 Key Technical Decisions & Rationale

#### GraphQL BFF Pattern

The GraphQL Backend-for-Frontend simplifies mobile client interaction by:

- Aggregating multiple services behind a single endpoint
- Reducing over-fetching
- Allowing independent evolution of backend services
- Supporting mobile-specific response shaping

This avoids tight coupling between frontend and internal services.

---

#### Service Separation (Booking / Journey / Inventory)

Services are split by domain:

- Booking Service: transactional, stateful, idempotent
- Journey Planning Service: computational, read-heavy, latency-sensitive
- Inventory Service: availability and partner-facing logic

This separation:

- Improves maintainability
- Prevents cascading failures
- Enables independent scaling
- Aligns with domain-driven design principles

---

#### Event-Driven Notifications

An Event Bus is used for booking lifecycle events.

Booking → publishes BookingConfirmed → notification system consumes → push sent.

This:

- Decouples booking from notification delivery
- Prevents synchronous latency coupling
- Enables future extensions (analytics, loyalty triggers, partner integrations)

---

#### Data Layer Strategy

**PostgreSQL**

- Durable storage for bookings and linked accounts
- Strong transactional guarantees
- ACID compliance for financial operations

**Redis**

- Caches journey computations and availability data
- Reduces latency for traffic-heavy queries
- Short TTL to tolerate minor staleness

---

## 2. PCI Compliance Approach

The system is designed to minimize PCI scope.

### Key Decisions

- No card data is stored or processed by the Priority Pass Integration Platform.
- Payments are delegated to a PCI-compliant Payment Provider or Taxi Partner.
- Only tokenized payment references are handled internally.
- All payment processing occurs externally over secure HTTPS.
- Sensitive data is never persisted in application databases.

### Result

The platform avoids direct PCI DSS scope by:

- Not storing Primary Account Numbers (PAN)
- Not transmitting raw card data
- Not performing payment authorization logic

This significantly reduces compliance burden and operational risk.

---

## 3. Correctness & Reliability Properties

The system is designed with the following guarantees:

**Idempotent Booking**  
Duplicate booking submissions will not create multiple rides.

**No Double Charge**  
Payment coordination is performed before booking confirmation is finalized.

**Transactional Integrity**  
Booking persistence uses database transactions.

**Eventual Consistency**  
Notifications are eventually consistent via event streaming.

**Cache Tolerance**  
Redis cache failures degrade gracefully to upstream API calls.

**Observability**  
All services emit logs, metrics, and traces for:

- Booking lifecycle
- External API failures
- Latency thresholds

---

## 4. Omissions & Trade-offs

### 4.1 Multi-Region Active/Active

Not implemented in this prototype.

Future improvement:

- Multi-region deployment
- Read replicas
- Cross-region event streaming

---

### 4.2 Orchestration

The booking flow is coordinated inside a single service rather than a distributed saga framework.

Trade-off:

- Simpler MVP
- Lower operational complexity

Future:

- Introduce orchestration framework if multi-partner expansion occurs.

---

### 4.3 Dynamic Surge Pricing Logic

Taxi pricing is assumed to be externally calculated.

Future:

- Add price lock windows
- Retry logic on surge changes

---

### 4.4 Advanced Security Hardening

Not implemented:

- Rate limiting
- WAF
- DDoS protection
- Fine-grained RBAC

Advanced security hardening is referenced at infrastructure level (AWS Architecture Diagram) but not deeply elaborated in this document.

---

### 4.5 Inventory Sharing in Taxi App User Interface

The prototype focuses on the Priority Pass user journey.

Bidirectional inventory surfacing (Taxi app → PP content) is described architecturally but not fully prototyped visually.

---

## 5. Prototype Overview

The React prototype demonstrates:

- Lounge browsing
- Journey planning
- Taxi option selection
- Booking confirmation
- Contextual Priority Pass benefit surfacing

The UI prioritizes clarity of flow over full production polish.

---

## 6. AI Usage

AI tooling was used as a productivity accelerator during this submission.

Specifically, it helped to:
- Accelerate diagram iteration and layout refinement
- Pressure-test architectural completeness and identify potential gaps
- Generate structured documentation drafts for refinement
-  Scaffold portions of the frontend prototype to reduce implementation time

All architectural decisions, service boundaries, trade-offs, and compliance considerations were defined, reviewed, and validated manually. AI was used to improve efficiency and presentation polish — not to replace architectural reasoning.

---

# Deployment Model (AWS)

## Overview

The Priority Pass Taxi Integration Platform is deployed on AWS using a container-based runtime.

Services are isolated by domain (GraphQL BFF, Booking, Journey Planning, Inventory) and deployed behind a managed ingress layer.

The design supports horizontal scaling, resilient integration with external partners, and strong observability.

---

## Runtime & Networking

- VPC (multi-AZ) with public/private subnets
- Public entry via Application Load Balancer (ALB) or API Gateway
- Private service networking for internal service-to-service calls
- Security Groups restrict access between tiers (ingress → services → data)

---

## Compute

### Option A — Kubernetes (EKS)

- Each service runs as a Kubernetes Deployment (HPA enabled)
- Ingress controller routes traffic to GraphQL BFF
- Separate namespaces per environment (dev/stage/prod)

### Option B — ECS Fargate

- Each service runs as a Fargate task
- ALB routes to GraphQL BFF service
- Auto scaling policies based on CPU/latency

---

## Data Layer

- Amazon RDS for PostgreSQL (Multi-AZ)
- Amazon ElastiCache for Redis (short TTL caching)

---

## Messaging / Events

### MVP Option — SNS + SQS

- Booking Service publishes booking events to SNS
- Subscribers consume via SQS

### Scale Option — Amazon MSK (Kafka)

- Higher throughput
- Replayable event streams
- Better for multi-consumer architecture

---

## Secrets & Configuration

- AWS Secrets Manager for partner API keys and OAuth secrets
- SSM Parameter Store for non-sensitive config
- KMS encryption at rest

---

## Observability & Operations

- Datadog for logs, metrics, tracing
- Correlation IDs propagated through services
- Alerts for:
  - Booking failure rate
  - Partner API latency
  - Queue backlog
  - Payment anomalies

---

## Security Considerations

- TLS everywhere
- IAM least privilege
- Optional WAF at edge
- Rate limiting at API gateway

---

## Availability & Scaling Strategy

- Multi-AZ deployment
- Stateless services scale horizontally
- Redis loss tolerated
- External partner failures handled via timeouts, retries, and circuit breakers
