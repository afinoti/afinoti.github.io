---
title: Chatbot de Atendimento
---

# Chatbot de Atendimento e App Android

Este exemplo mapeia dois sistemas reais aos dez conceitos arquiteturais do [TOGAF](/posts/TOGAF/): um [Business Service](/concepts/business-service), três níveis de [Application Architecture](/concepts/application-architecture), três níveis de [Data Architecture](/concepts/data-architecture), e três níveis de [Technology Architecture](/concepts/technology-architecture).

## Sobre os Sistemas

### Chatbot

O chatbot oferece atendimento ao cliente por dois canais: interface web (`/atendimento`) e WhatsApp. Entrega duas capacidades de negócio: **atendimento pós-venda** (FAQ e consulta de pedidos) e **descoberta de catálogo** (busca de produtos por linguagem natural, com retorno de carrossel).

Para consultas de pedido, o chatbot exige autenticação — caso contrário, redireciona para `/login`. Após autenticação, o usuário retorna ao canal de origem.

Internamente: frontend React (canal web), backend Go de orquestração, backend Python de atendimento no AWS Bedrock Runtime, e um **Fashion Agent** — microserviço Python separado no Bedrock Runtime — responsável pela busca de catálogo. O canal WhatsApp é integrado via **Braze**. Consultas de pedido vão ao Orders Microservice via API. O catálogo é acessado via **Catalog Search API** (SKUs por critério) e **Product API** (detalhes e preço por SKU). Autenticação via Auth Service em Go que encapsula Keycloak.

### App Android

O app Android (disponível na Google Play Store) também entrega **descoberta de catálogo** — o mesmo Business Service do chatbot, em canal diferente. Além da busca de produtos, o app oferece **estimativa de custo e prazo de entrega** antes da compra, funcionalidade não disponível no chatbot.

O app Flutter comunica-se com o backend através de um **BFF** (Backend for Frontend) exposto via AWS API Gateway. O BFF é um microserviço Go em Kubernetes que agrega chamadas ao catálogo e ao Delivery Estimator. O **Delivery Estimator** é um microserviço Go interno em Kubernetes que consulta uma API logística de terceiros para calcular frete.

---

## Business Services

O chatbot entrega dois Business Services distintos pelo mesmo canal. O canal (web ou WhatsApp) é compartilhado, mas as capacidades de negócio são independentes.

### Atendimento ao Cliente

A capacidade de suporte pós-venda: responder dúvidas e consultar pedidos sem depender de atendimento humano.

- **Quem consome:** clientes existentes com pedidos em aberto
- **Realizado por:** FAQ Query + Order Status Inquiry + User Authentication application services
- **Outcome de negócio:** redução do volume de chamados no suporte e autonomia do cliente

#### Service Quality

Os atributos não-funcionais que definem *quão bem* este Business Service deve operar:

| Atributo | Requisito |
|---|---|
| **Disponibilidade** | 99,5% em horário comercial (08h–22h, horário de Brasília) |
| **Performance — FAQ** | Resposta em até 3 segundos para 95% das requisições |
| **Performance — Pedido** | Resposta em até 5 segundos para 95% das requisições |
| **Segurança** | Dados de pedido acessíveis somente ao cliente autenticado; histórico de conversa não exposto a terceiros |
| **Idioma** | Suporte exclusivo em português brasileiro |
| **Canais suportados** | Web (`/atendimento`) e WhatsApp |

#### Contract

O acordo formal entre o provedor do serviço e seu consumidor interno:

- **Provedor:** time de produto (responsável pelo chatbot)
- **Consumidor:** área de CX (Central de Atendimento)
- **Vigência:** anual, renovável; revisão trimestral dos atributos de Quality
- **Compromisso:** entrega do Business Service dentro dos atributos de Quality definidos acima
- **Violação — P2:** disponibilidade abaixo de 99% por 7 dias corridos → abertura automática de incidente para revisão de capacidade
- **Violação — P1:** tempo de resposta de FAQ acima de 10 segundos em média por 1 hora contínua → acionamento imediato do time de engenharia

### Descoberta de Catálogo

A capacidade de explorar o catálogo de produtos e receber sugestões com informações suficientes para uma decisão de compra.

- **Quem consome:** qualquer visitante — não exige autenticação
- **Realizado por:** Product Catalog Search + Shipping Cost Estimation application services
- **Canais:** Chatbot (web e WhatsApp) e App Android
- **Outcome de negócio:** aumento de descoberta de produtos e conversão pré-venda

> O mesmo Business Service é realizado por dois sistemas distintos. O chatbot entrega Product Catalog Search via linguagem natural; o app entrega Product Catalog Search via navegação visual e acrescenta Shipping Cost Estimation. O Business Service não muda — o que muda são os Application Services disponíveis em cada canal.

---

## Application Architecture

### Application Services

Os três serviços que a aplicação precisa oferecer para realizar o Business Service:

| Application Service | O que é |
|---|---|
| **FAQ Query** | Capacidade de responder perguntas frequentes com base em uma base de conhecimento |
| **Order Status Inquiry** | Capacidade de consultar o status dos pedidos de compra de um cliente |
| **Product Catalog Search** | Capacidade de buscar produtos no catálogo por critério e retornar uma lista de resultados — disponível no chatbot (linguagem natural) e no app (navegação visual) |
| **Shipping Cost Estimation** | Capacidade de estimar custo e prazo de entrega de um produto para um CEP — disponível apenas no app |
| **User Authentication** | Capacidade de autenticar um usuário antes de acessar dados protegidos |

---

### Logical Application Components

Agrupamentos lógicos que realizam os Application Services, sem referência a tecnologia:

**Chatbot**
- Realiza: FAQ Query, Order Status Inquiry
- Depende de: Order Management (para consultar pedidos), Identity Provider (para validar sessão)
- Recebe mensagens de: React Frontend (canal web) e WhatsApp Connector (canal WhatsApp)

**WhatsApp Connector**
- Realiza: FAQ Query, Order Status Inquiry, Product Catalog Search (via canal WhatsApp)
- Depende de: Chatbot (repassa mensagens ao core), Identity Provider (redireciona para `/login` quando sessão ausente, e retorna ao WhatsApp após autenticação)
- Recebe mensagens de: WhatsApp (via Braze)

**Fashion Agent**
- Realiza: Product Catalog Search
- Depende de: Catalog Search (para obter lista de SKUs por critério), Product Catalog (para obter detalhes e preço de cada SKU)
- Retorna: lista estruturada de produtos renderizada como carrossel pelo frontend

**Catalog Search**
- Realiza: Product Catalog Search (parcialmente — busca por SKU)
- Expõe: API consumida pelo Fashion Agent e pelo App BFF
- Nota: sistema externo, não pertence ao domínio do chatbot nem do app

**Product Catalog**
- Realiza: Product Catalog Search (parcialmente — detalhes por SKU)
- Expõe: API consumida pelo Fashion Agent e pelo App BFF
- Nota: sistema externo, não pertence ao domínio do chatbot nem do app

**Android App**
- Realiza: Product Catalog Search, Shipping Cost Estimation (interface com o usuário)
- Depende de: App BFF (todas as chamadas de dados passam pelo BFF)
- Canal: Google Play Store, dispositivos Android

**App BFF**
- Agrega: chamadas ao Catalog Search, Product Catalog e Delivery Estimator
- Expõe: API unificada consumida pelo Android App via AWS API Gateway
- Depende de: Catalog Search, Product Catalog, Delivery Estimator

**Delivery Estimator**
- Realiza: Shipping Cost Estimation
- Expõe: API REST com entrada `{sku, cep}` e saída `{custo, prazo_dias}`; consumida pelo App BFF
- Depende de: Logistics API (terceiro)

**Logistics API**
- Realiza: Shipping Cost Estimation (cálculo efetivo de frete)
- Expõe: API consumida pelo Delivery Estimator
- Nota: sistema externo de terceiro (logística), não pertence ao domínio do app

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
| **React Frontend** | Interface do Chatbot (canal web) | React, servido em `/atendimento` |
| **Chatbot Go Service** | Orquestração do Chatbot | Go, container em Kubernetes |
| **Chatbot Python AI Service** | Inferência LLM do Chatbot | Python, AWS Bedrock Runtime |
| **Fashion Agent Service** | Fashion Agent | Python, AWS Bedrock Runtime (microserviço separado) |
| **Braze** | WhatsApp Connector | SaaS — recebe mensagens do WhatsApp e repassa ao Chatbot Go Service |
| **Catalog Search API** | Catalog Search | API externa — retorna lista de SKUs por critério |
| **Product API** | Product Catalog | API externa — retorna detalhes e preço por SKU |
| **Flutter Android App** | Android App | Flutter, distribuído via Google Play Store |
| **App BFF Service** | App BFF | Go, container em Kubernetes — exposto via AWS API Gateway |
| **Delivery Estimator Service** | Delivery Estimator | Go, container em Kubernetes |
| **Logistics API** | Logistics API | API externa de terceiro — cálculo de frete |
| **Orders Microservice** | Order Management | Go, container em Kubernetes |
| **Auth Service** | Gateway do Identity Provider | Go, container em Kubernetes |
| **Keycloak** | Core do Identity Provider | Keycloak, container em Kubernetes |

> O Auth Service é um Physical Application Component separado do Keycloak: ele encapsula e expõe o Keycloak como uma API própria, isolando os consumidores da dependência direta com o produto.

> O Braze é um Physical Application Component do tipo SaaS: não roda em infraestrutura própria, mas realiza o WhatsApp Connector ao intermediar a comunicação entre o WhatsApp Business API e o backend do chatbot. O fluxo de autenticação pelo WhatsApp segue o mesmo caminho do canal web — quando o usuário não está autenticado, o Braze envia um link de login via WhatsApp; após a autenticação no Keycloak, o usuário é redirecionado de volta ao WhatsApp.

---

## Data Architecture

### Data Entities

Os conceitos de negócio sobre os quais o sistema precisa guardar informação:

- **Customer** — o cliente que acessa o chatbot
- **Order** — um pedido de compra do cliente
- **FAQ Article** — um par de pergunta e resposta da base de conhecimento
- **Chat Message** — uma mensagem individual trocada entre o cliente e o chatbot, com atributo `channel` (web | whatsapp)
- **Conversation** — uma sessão de atendimento composta por uma sequência de Chat Messages, com atributo `channel` que identifica a origem
- **Product** — um item do catálogo com descrição, atributos (cor, tamanho) e preço
- **SKU** — a unidade de estoque que identifica univocamente uma variação de produto

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

**Product Catalog**
- Agrupa: Product + SKU + atributos de produto (cor, tamanho, imagens, preço)
- Pertence ao domínio externo de catálogo — não é armazenado pelo chatbot; é consumido via API pelo Fashion Agent

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

> **Logical Data Component sem Physical Data Component próprio.** O `Product Catalog` não tem Physical Data Component no domínio do chatbot — os dados vivem em sistemas externos acessados via API. Isso é válido: o TOGAF não exige que cada Logical Data Component seja realizado por storage próprio. Quando o dado pertence a outro domínio e é consumido via API, o Physical Data Component pertence ao domínio dono daquele dado.

---

## Technology Architecture

### Technology Services

Um Technology Service descreve **o que a infraestrutura precisa ser capaz de fazer** — um resultado repetível e com outcome definido, sem nomear produto ou categoria de produto. É a camada comportamental: define a necessidade, não quem a satisfaz.

| Technology Service | Outcome esperado |
|---|---|
| **Container Execution** | Executar workloads isolados e reproduzíveis como containers |
| **LLM Inference** | Invocar um modelo de linguagem para gerar respostas em linguagem natural |
| **Data Persistence** | Armazenar e recuperar dados com durabilidade — independente do modelo de armazenamento |
| **External Channel Communication** | Enviar e receber mensagens via plataformas externas de mensageria (WhatsApp, SMS, push) |
| **API Management** | Expor, rotear e proteger APIs HTTP para clientes externos (mobile, web, parceiros) |
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

**Messaging Platform Adapter**
- Realiza: External Channel Communication
- O que é a classe: plataforma SaaS de engajamento multicanal que conecta sistemas internos ao WhatsApp Business API e outros canais externos (ex: Braze, Twilio, Infobip)
- Usado neste sistema: pelo WhatsApp Connector para receber e enviar mensagens via WhatsApp

**API Gateway**
- Realiza: API Management
- O que é a classe: serviço gerenciado que recebe requisições HTTP de clientes externos, aplica autenticação, rate limiting e roteamento, e repassa ao backend (ex: AWS API Gateway, Kong, Apigee)
- Usado neste sistema: expõe o App BFF Service ao Flutter Android App

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
| **Braze** | Messaging Platform Adapter | Braze SaaS — plataforma de engajamento multicanal |
| **AWS API Gateway** | API Gateway | Amazon API Gateway — gerenciado pela AWS |
| **Keycloak on EKS** | Identity Server | Keycloak, container no Amazon EKS |

---

## Visão consolidada

| Conceito | Instância neste sistema |
|---|---|
| **Business Service** | Atendimento ao Cliente · Descoberta de Catálogo |
| **Application Service** | FAQ Query · Order Status Inquiry · Product Catalog Search · Shipping Cost Estimation · User Authentication |
| **Logical Application Component** | Chatbot · WhatsApp Connector · Fashion Agent · Android App · App BFF · Delivery Estimator · Catalog Search · Product Catalog · Logistics API · Order Management · Identity Provider |
| **Physical Application Component** | React Frontend · Chatbot Go Service · Chatbot Python AI Service · Fashion Agent Service · Braze · Flutter Android App · App BFF Service · Delivery Estimator Service · Logistics API · Catalog Search API · Product API · Orders Microservice · Auth Service · Keycloak |
| **Data Entity** | Customer · Order · FAQ Article · Chat Message · Conversation · Product · SKU |
| **Logical Data Component** | Customer Profile · Order History · Knowledge Base · Conversation History · Product Catalog |
| **Physical Data Component** | Chatbot PostgreSQL (AWS RDS) · Conversation Archive (AWS S3) · Orders DB · Keycloak DB |
| **Technology Service** | Container Execution · LLM Inference · Data Persistence · External Channel Communication · API Management · Identity Token Management |
| **Logical Technology Component** | Container Orchestrator · Managed LLM Runtime · Relational Database Engine · Object Store · Messaging Platform Adapter · API Gateway · Identity Server |
| **Physical Technology Component** | Amazon EKS · AWS Bedrock Runtime · AWS RDS PostgreSQL · AWS S3 · Braze · AWS API Gateway · Keycloak on EKS |

---

## Referências

As definições de **Technology Service** e **Logical Technology Component** utilizadas neste exemplo foram verificadas contra a documentação oficial do TOGAF e fontes especializadas:

- [The TOGAF Standard, Version 9.2 — Phase D: Technology Architecture](https://pubs.opengroup.org/architecture/togaf9-doc/arch/chap11.html) — Open Group
- [The TOGAF Standard, Version 9.2 — Glossary of Supplementary Definitions](https://pubs.opengroup.org/architecture/togaf9-doc/arch/apdxa.html) — Open Group
- [Logical Technology Component](https://www.tx.cz/en/terms/togaf-standard/logical-technology-component) — TAYLLORCOX TOGAF Reference
- [TOGAF Building Blocks and Services](http://grahamberrisford.com/00EAframeworks/01Fundamentals/TOGAF%20building%20blocks%20and%20services.htm) — Graham Berrisford
