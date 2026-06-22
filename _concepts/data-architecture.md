---
title: Data Architecture
---

# Data Architecture

**Data Architecture** is one of the four domains of [TOGAF](/posts/TOGAF/), defined within **Phase C: Information Systems Architecture** of the ADM. It describes the structure of an organization's logical and physical data assets and the resources used to manage them.

Its purpose is to ensure that data is treated as a strategic asset — consistently defined, well-governed, and aligned with business needs regardless of the technology used to store or process it.

Data Architecture is structured in three levels of abstraction, from the most conceptual to the most concrete:

---

## Data Entity

A **Data Entity** is the most abstract level. It represents a meaningful business concept — a "thing" that the organization needs to track information about.

Data Entities are technology-agnostic and business-facing. They appear in a **Conceptual Data Model** and are defined purely in terms of business semantics, not implementation details.

**Examples:** `Customer`, `Product`, `Order`, `Invoice`, `Employee`

> At this level, the question is: *"What does the business care about?"*

---

## Logical Data Component

A **Logical Data Component** is the intermediate level. It groups and structures Data Entities into logical units, defining their attributes, relationships, and business rules — independent of any specific technology or storage system.

Logical Data Components appear in a **Logical Data Model** and describe *how* data is organized and related, not *where* or *how* it is physically stored.

**Examples:** A `Customer Profile` component that bundles `Customer`, `Address`, and `ContactInfo` entities; an `Order Management` component grouping `Order`, `OrderItem`, and `Product`.

> At this level, the question is: *"How is data structured and related?"*

---

## Physical Data Component

A **Physical Data Component** is the most concrete level. It represents the actual implementation of data storage — tied to a specific technology, database system, or infrastructure.

Physical Data Components appear in a **Physical Data Model** and define tables, columns, indexes, data types, partitioning strategies, and other technology-specific details.

**Examples:** A PostgreSQL table `customers` with columns `id`, `name`, `email`; an S3 bucket `orders-raw/` storing JSON files; a Redis cache for session data.

> At this level, the question is: *"Where and how is data physically stored?"*

---

## Example: the same concept across three levels

To understand how the three levels relate, consider a single business concept — **"Customer"** — seen through each lens:

### Level 1 — Data Entity: `Customer`

**What it is:** An entry in the organization's **Entity Catalog** or **Business Glossary** — a document (or wiki page) that names and describes every business concept the organization tracks.

**What it looks like:** A named box in a conceptual diagram, with no attributes, no data types, no technology — just the concept and its relationships to other entities.

```
[Customer] --places--> [Order]
[Order]    --contains--> [Product]
```

**Where it lives:** Architecture repository — conceptual data model, business glossary, or entity catalog artifact produced during TOGAF Phase C.

---

### Level 2 — Logical Data Component: `Customer Profile`

**What it is:** A named **subject area** in a Logical Data Model — a grouping that brings together the `Customer`, `Address`, and `ContactInfo` entities with their attributes and relationships, but without specifying any database technology.

**What it is NOT:** It is not a code class, package, or microservice. It is an architecture artifact — a box in a logical ER diagram or architecture drawing that says: *"these entities belong together and are managed as a unit."*

**What it looks like:** A logical ER diagram with attributes and cardinalities, but no specific data types or storage details:

```
Customer Profile
├── Customer  { id, name, email }
│     └── has many → Address { id, street, city, zip }
└── has one  → ContactInfo { phone, preferred_channel }
```

**Where it lives:** Architecture repository — logical data model artifact, typically a diagram in a modeling tool (e.g. Archi, Sparx EA, draw.io) produced during TOGAF Phase C.

---

### Level 3 — Physical Data Component: `customers` table

**What it is:** The actual storage implementation — a concrete artifact tied to a specific technology.

**What it looks like:** A DDL script, a database schema, or an infrastructure spec:

```sql
CREATE TABLE customers (
  id    SERIAL PRIMARY KEY,
  name  VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL
);
```

Or, for a non-relational implementation: an S3 path `s3://data-lake/customers/`, a MongoDB collection, or a Kafka topic — whatever the chosen technology dictates.

**Where it lives:** The actual database, data lake, or storage system — and documented in the Physical Data Model artifact in the architecture repository.

---

## Summary

| Level | Artifact | Runtime Instance |
|---|---|---|
| **Data Entity** | Entry in the Entity Catalog or Business Glossary (conceptual diagram, wiki page) | None — exists only as a design artifact |
| **Logical Data Component** | JSON Schema, Avro, Protobuf, logical ER diagram | None — exists only as a design artifact |
| **Physical Data Component** | Migration file, DDL script | The actual table in the database, Kafka topic, S3 bucket, or any running storage construct |
