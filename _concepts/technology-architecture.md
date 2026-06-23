---
title: Technology Architecture
---

# Technology Architecture

**Technology Architecture** is one of the four domains of [TOGAF](/posts/TOGAF/), defined within **Phase D** of the ADM. It describes the technology infrastructure — hardware, software platforms, and services — required to support the [Application Architecture](/concepts/application-architecture) and [Data Architecture](/concepts/data-architecture).

Its purpose is to ensure that technology choices are driven by architecture needs, not by ad-hoc decisions — and that infrastructure capabilities are clearly defined, logically organized, and traceable from application requirements down to the technology that is actually running.

Technology Architecture is structured in three levels of abstraction, from the most conceptual to the most concrete:

---

## Technology Service

A **Technology Service** is the most abstract level. It represents a **technology capability** that the infrastructure must provide to support applications and data — defined in terms of *what is needed*, not *what provides it*.

Technology Services are vendor-agnostic and platform-agnostic. They describe infrastructure requirements without referencing any specific product or technology.

**Examples:** `Compute`, `Persistent Storage`, `Message Brokering`, `Load Balancing`, `Container Execution`, `Secret Management`

> At this level, the question is: *"What infrastructure capabilities does the architecture require?"*

---

## Logical Technology Component

A **Logical Technology Component** is the intermediate level. It is a named, bounded infrastructure capability that realizes one or more Technology Services — grouping related capabilities together and defining interfaces and dependencies between components, without specifying any vendor, product, or deployment detail.

Logical Technology Components appear in **technology component diagrams** and define the infrastructure landscape at a logical level.

**Examples:** `Container Platform`, `Database Platform`, `Message Bus`, `API Gateway`, `Identity Platform`, `Observability Platform`

> At this level, the question is: *"How are infrastructure capabilities grouped, bounded, and connected?"*

---

## Physical Technology Component

A **Physical Technology Component** is the most concrete level. It is a specific technology product, platform, or piece of infrastructure — tied to a vendor, a version, and a deployment configuration.

Physical Technology Components are what actually runs. They are the result of implementing one or more Logical Technology Components using a chosen product or service.

**Examples:** Kubernetes 1.29 on AWS EKS; PostgreSQL 15 on AWS RDS; Apache Kafka on Confluent Cloud; HashiCorp Vault on a dedicated VM; AWS Application Load Balancer.

> At this level, the question is: *"What technology is actually deployed and running?"*

---

## Example: the same concept across three levels

To understand how the three levels relate, consider a single infrastructure need — **"applications must be deployed and run reliably as containers"** — seen through each lens:

### Level 1 — Technology Service: `Container Execution`

**What it is:** An entry in the **Technology Service Catalog** — a document or architecture diagram that lists the infrastructure capabilities required, mapped to application needs.

**What it looks like:** A named entry in a technology service catalog, mapped to an application need — no product or vendor attached:

- **Application need:** deploy `customer-service` → requires → **Technology Service:** Container Execution

**Where it lives:** Architecture repository — technology service catalog or infrastructure capability map artifact produced during TOGAF Phase D.

---

### Level 2 — Logical Technology Component: `Container Platform`

**What it is:** A named boundary in a **technology component diagram** that groups related technology services (`Container Execution`, `Service Discovery`, `Auto-scaling`) and defines how it interfaces with other components (e.g., depends on `Persistent Storage`, exposes runtime to `Application Layer`).

**What it is NOT:** It is not a specific product, cloud provider, or configuration. It is an architecture artifact — a box in a technology diagram that says: *"these infrastructure capabilities belong together and are managed as a unit, with these interfaces."*

**What it looks like:** A box in a technology component diagram with its capabilities and dependencies defined — no vendor, no product:

**Container Platform**
- Provides: Container Execution, Service Discovery, Auto-scaling
- Depends on: Persistent Storage
- Exposes runtime to: Application Layer

**Where it lives:** Architecture repository — logical technology component diagram, typically produced in a modeling tool (e.g. Archi, Sparx EA, draw.io) during TOGAF Phase D.

---

### Level 3 — Physical Technology Component: Kubernetes 1.29 on AWS EKS

**What it is:** The actual infrastructure that implements the `Container Platform` logical component — a specific product, version, and cloud provider configuration.

**What it looks like:** An Infrastructure-as-Code file (e.g. a Terraform `.tf` file, a CloudFormation stack, or a Pulumi program) that declares the specific resource — with cluster name, version, cloud region, IAM roles, and network configuration all explicitly defined.

**Where it lives:** The IaC repository (Terraform, CloudFormation, Pulumi) — and once applied, the actual running cluster or infrastructure resource in the cloud or data center.

---

## Summary

| Level | Artifact | Runtime Instance |
|---|---|---|
| **Technology Service** | Entry in the Technology Service Catalog, infrastructure capability map diagram | None — exists only as a design artifact |
| **Logical Technology Component** | Technology component diagram in a modeling tool (Archi, Sparx EA, draw.io) | None — exists only as a design artifact |
| **Physical Technology Component** | Infrastructure-as-Code (Terraform, CloudFormation, Pulumi, Helm) | The actual running infrastructure — a cluster, a managed database, a cloud service, a VM |
