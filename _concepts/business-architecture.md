---
title: Business Architecture
---

# Business Architecture

**Business Architecture** is one of the four domains of [TOGAF](/posts/TOGAF/), defined within **Phase B** of the ADM. It describes the business itself — its strategy, organization, capabilities, processes, and the services it delivers — entirely in terms a business stakeholder understands, with no reference to technology or implementation.

Its purpose is to establish the foundation that all other architecture domains serve. [Application Architecture](/concepts/application-architecture), [Data Architecture](/concepts/data-architecture), and [Technology Architecture](/concepts/technology-architecture) exist to realize what Business Architecture defines.

Business Architecture is the only domain that can involve **human actors** as executors of its capabilities. The other three domains describe automated systems. Business Architecture describes the world before, around, and above automation.

---

## Key Building Blocks

Business Architecture is not structured in three levels of abstraction like the other domains. Instead, it is composed of complementary building blocks, each describing a different dimension of how the business operates.

---

### Business Service

A **Business Service** is the externally visible capability that the business delivers to a customer, partner, or other consumer — described entirely in terms of business value and outcome, with no reference to technology.

It answers: *"What does the business offer, and to whom?"*

A Business Service does not prescribe who or what delivers it. It can be realized by people, by automated systems, or by a combination of both.

→ See [Business Service](/concepts/business-service) for the full definition and examples.

---

### Business Process

A **Business Process** is a sequence of activities — performed by human actors, automated systems, or both — that together produce the outcome promised by a Business Service.

While a Business Service says *what* is delivered, a Business Process describes *how* it is delivered step by step.

**Characteristics:**
- Has a defined start and end
- Produces a measurable outcome
- Can involve decisions, branches, and loops
- Can be fully manual, fully automated, or hybrid

**Example — "Atendimento ao Cliente" realized by a manual process:**

1. Customer sends a message via web or WhatsApp
2. Support agent reads the message
3. Agent checks the order status in the order system
4. Agent composes and sends a response
5. If unresolved: agent escalates to specialist team

**Example — same Business Service realized by an automated process:**

1. Customer sends a message via web or WhatsApp
2. Chatbot classifies the intent (FAQ or order query)
3. If FAQ: Python AI Service retrieves answer from Knowledge Base and responds
4. If order query: system checks authentication, queries Orders Microservice, responds with status
5. If unresolved: chatbot offers escalation to human agent

Both processes realize the same Business Service. The Business Service definition does not change when you automate it — only the process that delivers it changes.

---

### Business Function

A **Business Function** is a grouping of related business activities that share a common purpose — independent of how they are organized or by whom they are performed. It describes a stable area of business responsibility.

While a Business Process is a flow with a start and end, a Business Function is a persistent capability area that the business maintains over time.

**Examples:**
- Customer Support — the ongoing capability to handle customer inquiries
- Catalog Management — the ongoing capability to maintain and publish the product catalog
- Identity Management — the ongoing capability to manage user credentials and access

A Business Function can own multiple Business Processes and deliver multiple Business Services.

---

### Business Role

A **Business Role** is a named function performed within a business process — it describes *what kind of actor* is needed, not which specific person or system fills that role.

Roles are abstract: the same role can be played by a human in one context and by an automated system in another.

**Examples in "Atendimento ao Cliente":**

| Role | Played by (manual) | Played by (automated) |
|---|---|---|
| Message Receiver | Support agent | Chatbot Go Service |
| Intent Classifier | Support agent (reads and interprets) | Chatbot Python AI Service |
| Order Lookup | Support agent (queries system manually) | Orders Microservice |
| Responder | Support agent (writes reply) | Chatbot Go Service |

---

### Business Actor

A **Business Actor** is the specific entity — person, team, organization unit, or external system — that performs a Business Role in a given context.

| Business Role | Business Actor |
|---|---|
| Message Receiver | Chatbot Go Service |
| Intent Classifier | Chatbot Python AI Service (AWS Bedrock) |
| Order Lookup | Orders Microservice |
| Customer | End user (human) — initiates the process |

---

## How Business Architecture connects to the other domains

Business Architecture sits at the top of the architectural stack. The three levels beneath it exist to realize its definitions:

```
Business Architecture  ←── Phase B: what the business delivers and how
         │
         ▼
Application Architecture  ←── Phase C: what software capabilities support it
         │
         ▼
Data Architecture  ←── Phase C: what information those capabilities need
         │
         ▼
Technology Architecture  ←── Phase D: what infrastructure runs it all
```

Every architectural decision in the lower domains must be traceable back to a Business Service or Business Process in Business Architecture. If a technology component cannot be traced to a business need, it has no architectural justification.

---

## Example: Chatbot de Atendimento

From our [chatbot example](/examples/chatbot), the Business Architecture view of **"Atendimento ao Cliente"**:

**Business Service:** Atendimento ao Cliente — suporte pós-venda sem dependência de atendimento humano

**Business Process (automated):**
1. Customer sends message via web or WhatsApp
2. System authenticates the user (or redirects to login)
3. System classifies intent: FAQ or order query
4. System retrieves and returns the answer
5. If unresolved: system offers human escalation

**Business Function:** Customer Support — owns the Atendimento ao Cliente and any future support-related Business Services

**Business Roles:** Customer (initiator) · Intent Classifier · Order Lookup · Responder

**Business Actors:**
- Customer → end user (human)
- Intent Classifier → Chatbot Python AI Service
- Order Lookup → Orders Microservice
- Responder → Chatbot Go Service

---

## Summary

| Building Block | Answers | Can involve humans? |
|---|---|---|
| **Business Service** | What does the business deliver, and to whom? | Implicitly — delivery can be manual or automated |
| **Business Process** | How is it delivered, step by step? | Yes — steps can be manual, automated, or hybrid |
| **Business Function** | What persistent capability area owns it? | Yes — a team or a system |
| **Business Role** | What kind of actor is needed at each step? | Yes — a role is abstract; a human or system can fill it |
| **Business Actor** | Who or what fills that role concretely? | Yes — a person, team, or automated system |
