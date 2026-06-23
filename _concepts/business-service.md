---
title: Business Service
---

# Business Service

A **Business Service** is a concept from the Business Architecture domain of [TOGAF](/posts/TOGAF/), defined in **Phase B** of the ADM. It represents a capability that the business exposes to its customers, partners, or other parts of the organization — described entirely in terms of business value and outcomes, with no reference to technology or implementation.

A Business Service is the entry point of the architecture: it defines *what the business delivers*. The three Information Systems and Technology architecture domains exist to support and realize it.

> At this level, the question is: *"What does the business offer, and to whom?"*

---

## How it relates to the other architecture domains

A Business Service does not exist in isolation. It is realized by a stack of architectural layers beneath it:

| Layer | Domain | Example |
|---|---|---|
| **Business Service** | Business Architecture | Customer Onboarding |
| **Application Service** | [Application Architecture](/concepts/application-architecture) | Customer Registration, Identity Verification |
| **Logical Application Component** | Application Architecture | Customer Management, Identity Provider |
| **Physical Application Component** | Application Architecture | `customer-service` on Kubernetes, Keycloak |
| **Technology Service** | [Technology Architecture](/concepts/technology-architecture) | Container Execution, Persistent Storage |
| **Logical Technology Component** | Technology Architecture | Container Platform, Database Platform |
| **Physical Technology Component** | Technology Architecture | AWS EKS, AWS RDS PostgreSQL |
| **Data Entity / Logical / Physical** | [Data Architecture](/concepts/data-architecture) | `Customer` entity → `Customer Profile` → `customers` table |

The Business Service sits at the top. Everything below it exists to deliver it.

---

## Examples

### Customer Onboarding

The capability to receive a new customer, verify their identity, and activate their account.

- **Who consumes it:** New customers
- **Realized by:** Customer Registration + Identity Verification application services
- **Business outcome:** An active, verified customer account ready to transact

---

### Order Fulfillment

The end-to-end capability of receiving an order, processing payment, and delivering the product to the customer.

- **Who consumes it:** Customers who have placed an order
- **Realized by:** Order Management + Payment Processing + Shipping application services
- **Business outcome:** A delivered order and a satisfied customer

---

### Account Self-Service

The capability for existing customers to manage their own account — update personal data, change payment methods, view order history.

- **Who consumes it:** Existing customers
- **Realized by:** Customer Profile Update + Order History + Payment Method Management application services
- **Business outcome:** Reduced support cost and improved customer autonomy

---

## Key characteristics

- **Business-facing:** Defined in terms a business stakeholder understands, not a developer
- **Technology-agnostic:** The same Business Service can be realized by completely different technology stacks
- **Outcome-oriented:** Describes the value delivered, not the steps to deliver it
- **Traceable:** Every technology and data decision in the architecture should be traceable back to a Business Service it supports
