---
title: Chatbot de Atendimento
---

# Chatbot de Atendimento

Este exemplo mapeia um sistema real de chatbot aos dez conceitos arquiteturais do [TOGAF](/posts/TOGAF/): um [Business Service](/concepts/business-service), três níveis de [Application Architecture](/concepts/application-architecture), três níveis de [Data Architecture](/concepts/data-architecture), e três níveis de [Technology Architecture](/concepts/technology-architecture).

## Sobre o Sistema

O sistema de chatbot oferece atendimento ao cliente via interface web (`/atendimento`). O usuário pode tirar dúvidas sobre FAQ e consultar seus pedidos de compra em aberto. Para consultas de pedido, o chatbot exige que o usuário esteja autenticado — caso contrário, redireciona para `/login`. Após a autenticação, o usuário retorna ao chatbot com acesso completo.

Internamente, o chatbot é composto por três partes: um frontend React, um backend Go de orquestração, e um backend Python rodando inferência de LLM. As consultas de pedido são feitas via API ao Orders Microservice. A autenticação é gerenciada por um Auth Service em Go que encapsula uma instância Keycloak.

---

## Business Service

**Atendimento ao Cliente**

A capacidade que o negócio expõe ao cliente: obter respostas a dúvidas e informações sobre pedidos sem depender de suporte humano.

- **Quem consome:** clientes existentes
- **Realizado por:** FAQ Query + Order Status Inquiry + User Authentication application services
- **Outcome de negócio:** redução do volume de chamados no suporte e autonomia do cliente

---

## Application Architecture

### Application Services

Os três serviços que a aplicação precisa oferecer para realizar o Business Service:

| Application Service | O que é |
|---|---|
| **FAQ Query** | Capacidade de responder perguntas frequentes com base em uma base de conhecimento |
| **Order Status Inquiry** | Capacidade de consultar o status dos pedidos de compra de um cliente |
| **User Authentication** | Capacidade de autenticar um usuário antes de acessar dados protegidos |

---

### Logical Application Components

Agrupamentos lógicos que realizam os Application Services, sem referência a tecnologia:

**Chatbot**
- Realiza: FAQ Query, Order Status Inquiry
- Depende de: Order Management (para consultar pedidos), Identity Provider (para validar sessão)
- Expõe: interface web em `/atendimento`

**Order Management**
- Realiza: Order Status Inquiry
- Expõe: API consumida pelo Chatbot

**Identity Provider**
- Realiza: User Authentication
- Expõe: API de autenticação consumida pelo Chatbot
- Redireciona para `/login` quando sessão ausente

---

### Physical Application Components

As implementações concretas, com tecnologia e modelo de deploy definidos:

| Physical Application Component | Realiza | Tecnologia e Deploy |
|---|---|---|
| **React Frontend** | Interface do Chatbot | React, servido em `/atendimento` |
| **Chatbot Go Service** | Orquestração do Chatbot | Go, container em Kubernetes |
| **Chatbot Python AI Service** | Inferência LLM do Chatbot | Python, AWS Bedrock Runtime |
| **Orders Microservice** | Order Management | Go, container em Kubernetes |
| **Auth Service** | Gateway do Identity Provider | Go, container em Kubernetes |
| **Keycloak** | Core do Identity Provider | Keycloak, container em Kubernetes |

> O Auth Service é um Physical Application Component separado do Keycloak: ele encapsula e expõe o Keycloak como uma API própria, isolando os consumidores da dependência direta com o produto.

---

## Data Architecture

### Data Entities

Os conceitos de negócio sobre os quais o sistema precisa guardar informação:

- **Customer** — o cliente que acessa o chatbot
- **Order** — um pedido de compra do cliente
- **FAQ Article** — um par de pergunta e resposta da base de conhecimento
- **Chat Message** — uma mensagem individual trocada entre o cliente e o chatbot
- **Conversation** — uma sessão de atendimento composta por uma sequência de Chat Messages

---

### Logical Data Components

Agrupamentos lógicos que estruturam as entidades com atributos e relacionamentos, sem definir tecnologia de storage:

**Customer Profile**
- Agrupa: Customer + dados de sessão de autenticação
- Relacionamentos: Customer autentica-se via Identity Provider

**Order History**
- Agrupa: Order + Order Items + status de entrega
- Relacionamentos: Order pertence a um Customer

**Knowledge Base**
- Agrupa: FAQ Article + categorias + metadata de relevância
- Consumida pelo Chatbot Python AI Service para geração de respostas

**Conversation History**
- Agrupa: Conversation + Chat Messages (sequência de trocas entre cliente e chatbot)
- Possui duas implementações físicas com propósitos distintos: acesso estruturado em tempo real (Go Service → PostgreSQL) e armazenamento bruto para contexto do LLM (Python AI Service → S3)

---

### Physical Data Components

As implementações concretas de storage, ligadas a tecnologia específica:

| Physical Data Component | Realiza | Tecnologia | Gravado por |
|---|---|---|---|
| **Chatbot PostgreSQL DB** | Customer Profile, Knowledge Base, Conversation History (estruturada) | PostgreSQL no AWS RDS | Chatbot Go Service |
| **Conversation Archive (S3)** | Conversation History (bruta) | AWS S3 | Chatbot Python AI Service |
| **Orders DB** | Order History | Definido pelo domínio do Orders Microservice (não exposto neste escopo) | Orders Microservice |
| **Keycloak DB** | Sessões e credenciais de usuários | Gerenciado internamente pelo Keycloak | Keycloak |

> **Um Logical Data Component, dois Physical Data Components.** O `Conversation History` é realizado por PostgreSQL e S3 simultaneamente — cada um servindo um propósito diferente: o Go Service salva no Postgres para consultas estruturadas em tempo real; o Python AI Service salva no S3 o histórico bruto para alimentar o contexto do LLM. Isso é válido no TOGAF: um Logical Data Component pode ter múltiplas implementações físicas quando os requisitos de acesso são diferentes.

---

## Technology Architecture

### Technology Services

Um Technology Service descreve **o que a infraestrutura precisa ser capaz de fazer** — um resultado repetível e com outcome definido, sem nomear produto ou categoria de produto. É a camada comportamental: define a necessidade, não quem a satisfaz.

| Technology Service | Outcome esperado |
|---|---|
| **Container Execution** | Executar workloads isolados e reproduzíveis como containers |
| **LLM Inference** | Invocar um modelo de linguagem para gerar respostas em linguagem natural |
| **Data Persistence** | Armazenar e recuperar dados com durabilidade — independente do modelo de armazenamento |
| **Identity Token Management** | Emitir, validar e revogar tokens de autenticação de usuários |

---

### Logical Technology Components

Um Logical Technology Component é uma **classe de produto de tecnologia**, independente de fornecedor ou versão específica. Não é o produto em si — é a categoria de produto que realiza um ou mais Technology Services. A distinção permite que o mesmo componente lógico seja realizado por produtos diferentes em ambientes diferentes.

**Container Orchestrator**
- Realiza: Container Execution
- O que é a classe: software que agenda, escala e monitora workloads containerizados (ex: Kubernetes, Nomad, Docker Swarm)
- Runs neste sistema: Chatbot Go Service, Orders Microservice, Auth Service, Keycloak

**Managed LLM Runtime**
- Realiza: LLM Inference
- O que é a classe: plataforma que executa ou expõe modelos de linguagem sob demanda, gerenciada pelo provedor (ex: AWS Bedrock, Azure OpenAI Service, Google Vertex AI)
- Runs neste sistema: Chatbot Python AI Service

**Relational Database Engine**
- Realiza: Data Persistence
- O que é a classe: sistema gerenciador de banco de dados relacional com suporte a transações (ex: PostgreSQL, MySQL, Oracle DB)
- Usado neste sistema: pelo Chatbot Go Service para persistência de dados do chatbot

**Object Store**
- Realiza: Data Persistence
- O que é a classe: serviço de armazenamento de objetos sem esquema, acessado por chave (ex: AWS S3, Google Cloud Storage, Azure Blob Storage, MinIO)
- Usado neste sistema: pelo Chatbot Python AI Service para gravar o histórico bruto de conversas

**Identity Server**
- Realiza: Identity Token Management
- O que é a classe: software especializado em autenticação e autorização, emissão de tokens OAuth2/OIDC (ex: Keycloak, Auth0, Okta)
- Usado neste sistema: pelo Auth Service como core de identidade

---

### Physical Technology Components

O produto concreto escolhido para implementar cada Logical Technology Component — com fornecedor, versão e configuração de deploy definidos:

| Physical Technology Component | Implementa | Produto |
|---|---|---|
| **Amazon EKS (Kubernetes)** | Container Orchestrator | Kubernetes gerenciado pela AWS |
| **AWS Bedrock Runtime** | Managed LLM Runtime | Amazon Bedrock Runtime (managed) |
| **AWS RDS PostgreSQL** | Relational Database Engine | PostgreSQL no Amazon RDS |
| **AWS S3** | Object Store | Amazon Simple Storage Service |
| **Keycloak on EKS** | Identity Server | Keycloak, container no Amazon EKS |

---

## Visão consolidada

| Conceito | Instância neste sistema |
|---|---|
| **Business Service** | Atendimento ao Cliente |
| **Application Service** | FAQ Query · Order Status Inquiry · User Authentication |
| **Logical Application Component** | Chatbot · Order Management · Identity Provider |
| **Physical Application Component** | React Frontend · Chatbot Go Service · Chatbot Python AI Service · Orders Microservice · Auth Service · Keycloak |
| **Data Entity** | Customer · Order · FAQ Article · Chat Message · Conversation |
| **Logical Data Component** | Customer Profile · Order History · Knowledge Base · Conversation History |
| **Physical Data Component** | Chatbot PostgreSQL (AWS RDS) · Conversation Archive (AWS S3) · Orders DB · Keycloak DB |
| **Technology Service** | Container Execution · LLM Inference · Data Persistence · Identity Token Management |
| **Logical Technology Component** | Container Orchestrator · Managed LLM Runtime · Relational Database Engine · Object Store · Identity Server |
| **Physical Technology Component** | Amazon EKS · AWS Bedrock Runtime · AWS RDS PostgreSQL · AWS S3 · Keycloak on EKS |

---

## Referências

As definições de **Technology Service** e **Logical Technology Component** utilizadas neste exemplo foram verificadas contra a documentação oficial do TOGAF e fontes especializadas:

- [The TOGAF Standard, Version 9.2 — Phase D: Technology Architecture](https://pubs.opengroup.org/architecture/togaf9-doc/arch/chap11.html) — Open Group
- [The TOGAF Standard, Version 9.2 — Glossary of Supplementary Definitions](https://pubs.opengroup.org/architecture/togaf9-doc/arch/apdxa.html) — Open Group
- [Logical Technology Component](https://www.tx.cz/en/terms/togaf-standard/logical-technology-component) — TAYLLORCOX TOGAF Reference
- [TOGAF Building Blocks and Services](http://grahamberrisford.com/00EAframeworks/01Fundamentals/TOGAF%20building%20blocks%20and%20services.htm) — Graham Berrisford
