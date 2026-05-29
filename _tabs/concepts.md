---
layout: page
icon: fas fa-book
order: 5
---

# Concepts

Definitions and fundamental concepts in computer science.

---

## App (Application)

> Definition based on the [The Twelve-Factor App](https://12factor.net/codebase) methodology

An **app** is a unit of software tracked in a single version control system — called a **codebase**. In centralized systems (such as Subversion), the codebase is a single repository. In distributed systems (such as Git), it is the set of repositories that share a common root commit.

### Codebase

The methodology establishes a strict one-to-one correlation: **one codebase, one app**. This implies:

- If there are multiple codebases, it is a **distributed system**, not a single app — each component is its own app.
- Multiple apps sharing the same codebase violate the principle. Shared code should be extracted into independent libraries.

### Deploys

A **deploy** is a running instance of the app. Examples:

- The **production** environment (the live site)
- One or more **staging** environments
- The developer's **local copy**

There is only **one codebase** per app, but there can be **many deploys**. The codebase is the same across all deploys, but different versions may be active in each environment — a developer may have commits not yet promoted to staging, while staging may have commits not yet promoted to production.
