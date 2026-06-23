---
title: Application Architecture
---

# Application Architecture

**Application Architecture** is one of the four domains of [TOGAF](/posts/TOGAF/), defined within **Phase C: Information Systems Architecture** of the ADM alongside [Data Architecture](/concepts/data-architecture). It describes the individual application systems, their responsibilities, and how they interact with each other and with the rest of the enterprise.

Its purpose is to ensure that application capabilities are clearly defined, logically organized, and traceable from business needs down to the software that is actually deployed.

Application Architecture is structured in three levels of abstraction, from the most conceptual to the most concrete:

---

## Application Service

An **Application Service** is the most abstract level. It represents a **capability** that the application landscape must provide to support a business process — defined in terms of *what it does*, not *what implements it*.

Application Services are technology-agnostic and business-facing. They describe functionality the business requires, with no reference to any specific system or technology.

**Examples:** `Customer Registration`, `Payment Processing`, `Product Search`, `Authentication`

> At this level, the question is: *"What capabilities does the business need from its applications?"*

---

## Logical Application Component

A **Logical Application Component** is the intermediate level. It is a named, bounded unit of application functionality that realizes one or more Application Services — grouping related capabilities together and defining the interfaces and dependencies between components, without specifying any technology or deployment details.

Logical Application Components appear in **component diagrams** and define the architecture of the application landscape at a logical level.

**Examples:** `Customer Management`, `Payment Gateway`, `Identity Provider`, `Product Catalog`

> At this level, the question is: *"How is application functionality grouped, bounded, and connected?"*

---

## Physical Application Component

A **Physical Application Component** is the most concrete level. It is a specific, deployable unit of software — tied to a technology stack, a deployment model, and a runtime environment.

Physical Application Components are what actually runs. They are the result of implementing one or more Logical Application Components using a chosen technology.

**Examples:** A Spring Boot microservice packaged as a Docker image; a specific Salesforce org; a Keycloak instance running on Kubernetes; an AWS Lambda function.

> At this level, the question is: *"What software is actually deployed and running?"*

---

## Example: the same concept across three levels

To understand how the three levels relate, consider a single business need — **"a customer must be able to register an account"** — seen through each lens:

### Level 1 — Application Service: `Customer Registration`

**What it is:** An entry in the **Application Service Catalog** — a document or architecture diagram that lists the capabilities the application landscape must provide, mapped to business processes.

**What it looks like:** A named entry in an application service catalog, mapped to a business process — no technology attached:

- **Business Process:** Onboarding → requires → **Application Service:** Customer Registration

**Where it lives:** Architecture repository — service catalog or capability map artifact produced during TOGAF Phase C.

---

### Level 2 — Logical Application Component: `Customer Management`

**What it is:** A named boundary in a **component diagram** that groups related application services (`Customer Registration`, `Customer Profile Update`, `Account Deactivation`) and defines how it interfaces with other components (e.g., depends on `Identity Provider`, exposes an API to `Order Management`).

**What it is NOT:** It is not a specific codebase, framework, or deployment unit. It is an architecture artifact — a box in a component diagram that says: *"these capabilities belong together and are managed as a unit, with these interfaces."*

**What it looks like:** A box in a component diagram with its interfaces defined — no codebase, no technology:

**Customer Management**
- Provides: Customer Registration, Customer Profile Update, Account Deactivation
- Depends on: Identity Provider
- Exposes API to: Order Management

**Where it lives:** Architecture repository — logical application component diagram, typically produced in a modeling tool (e.g. Archi, Sparx EA, draw.io) during TOGAF Phase C.

---

### Level 3 — Physical Application Component: `customer-service` (Spring Boot on Kubernetes)

**What it is:** The actual deployable software that implements the `Customer Management` logical component — a specific technology choice with a specific deployment model.

**What it looks like:** A deployment descriptor or infrastructure-as-code artifact:

```yaml
# kubernetes/customer-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: customer-service
spec:
  replicas: 2
  template:
    spec:
      containers:
        - name: customer-service
          image: myorg/customer-service:2.4.1
```

**Where it lives:** The source code repository (deployment manifests, Helm charts, Docker Compose files) — and once deployed, the running container or process in the infrastructure.

---

## Summary

| Level | Artifact | Runtime Instance |
|---|---|---|
| **Application Service** | Entry in the Application Service Catalog, capability map diagram | None — exists only as a design artifact |
| **Logical Application Component** | Component diagram in a modeling tool (Archi, Sparx EA, draw.io) | None — exists only as a design artifact |
| **Physical Application Component** | Deployment descriptor (Kubernetes manifest, Helm chart, Docker Compose, IaC) | The actual running software — a container, a process, a SaaS instance |
