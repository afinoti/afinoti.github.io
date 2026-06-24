---
title: Business Capability
---

# Business Capability

Uma **Business Capability** é o que o negócio **é capaz de fazer** — uma habilidade delimitada e estável que o negócio possui ou precisa possuir, independente de como está organizado, de quem executa, ou de que tecnologia usa.

É um conceito interno: não tem receptor, não tem compromisso com ninguém. É a resposta para *"o negócio consegue fazer isso?"* — não *como*, não *quem*, não *com quê*.

> Uma Business Capability não muda quando o negócio automatiza um processo ou faz uma reorganização interna. O que muda é quem ou o quê a executa — a habilidade em si permanece.

---

## Como se distingue dos conceitos relacionados

| Conceito | Perspectiva | Pergunta | Muda quando automatiza? |
|---|---|---|---|
| **Business Capability** | Interna — o que possuímos | *"Conseguimos fazer isso?"* | Não |
| **Business Service** | Externa — o que entregamos | *"O que oferecemos, e para quem?"* | Não |
| ~~**Business Function**~~ | *(não utilizado nesta arquitetura — o papel que exerceria é coberto por Business Capability)* | — | — |
| **Business Process** | Execução — como entregamos | *"Como isso acontece, passo a passo?"* | Sim |

A distinção central entre **Business Capability** e **Business Service** é de perspectiva:

- A capability existe independente de ser oferecida a alguém — é um fato interno sobre o que o negócio consegue fazer.
- O Business Service é o momento em que o negócio pega uma capability e a estrutura como uma entrega com receptor e compromisso de qualidade.

A mesma capability pode suportar múltiplos Business Services para consumidores diferentes.

---

## Hierarquia de capabilities

Business Capabilities são hierárquicas. Uma capability de alto nível se decompõe em capabilities mais granulares, até o nível onde cada uma mapeia claramente para uma responsabilidade de sistema ou time.

```
L1 — Marketing                          ← decisão estratégica
  L2 — Customer Communication           ← área tática
    L3 — Email Delivery                 ← operacional; mapeia para Application Service
    L3 — SMS Delivery
    L3 — Push Notification
  L2 — Campaign Management
    L3 — Audience Segmentation
    L3 — Campaign Scheduling
```

| Nível | Usado para |
|---|---|
| **L1** | Planejamento estratégico — *"em quais grandes áreas precisamos investir?"* |
| **L2** | Priorização de iniciativas — *"qual área tática está fraca?"* |
| **L3** | Mapeamento para arquitetura — *"o que o software precisa suportar?"* |

A decomposição vai até o nível onde cada capability corresponde a uma responsabilidade clara. Em domínios menores, L2 já é suficiente para mapear para Application Services.

---

## Relação com os outros domínios

É no nível mais granular que as Business Capabilities se conectam à [Application Architecture](/concepts/application-architecture): cada capability operacional é suportada por um ou mais **Application Services**.

```
Business Capability (L3): Email Delivery
  └── suportada por → Application Service: Notification Service
                            └── realizado por → Logical Application Component: Notification Platform
                                                      └── Physical: SendGrid / AWS SES
```

Uma capability pode suportar múltiplos [Business Services](/concepts/business-service), e um Business Service pode requerer múltiplas capabilities:

```
Business Capability: Email Delivery
  └── suporta → Business Service: "Notificação de Pedido"  (consumidor: cliente final)
  └── suporta → Business Service: "Relatório de Vendas"    (consumidor: time interno)
  └── suporta → Business Service: "Confirmação de Cadastro" (consumidor: novo usuário)
```

---

## Exemplo: Chatbot de Atendimento

Do nosso [exemplo do chatbot](/examples/chatbot), as Business Capabilities necessárias para realizar o [Business Service](/concepts/business-service) **"Atendimento ao Cliente"**:

| Business Capability | Nível | Application Service que suporta |
|---|---|---|
| Customer Support | L1 | — |
| Customer Authentication | L2 | Auth Service |
| Intent Classification | L2 | Chatbot Python AI Service |
| Order Status Retrieval | L2 | Orders Microservice |
| Human Escalation | L2 | Chatbot Go Service |

O domínio é pequeno o suficiente para que o L2 mapeie diretamente para Application Services. Em domínios maiores, seria necessário um L3.

---

## Por que o conceito existe

Business Capabilities são a ferramenta central do **Capability-Based Planning**: avaliar quais habilidades o negócio possui, em que nível de maturidade, e onde investir para fechar lacunas.

Um **Capability Heat Map** é o artefato típico — cada capability recebe uma avaliação de maturidade (baixa / média / alta), revelando onde a arquitetura precisa evoluir antes de qualquer decisão de tecnologia.

> A pergunta estratégica não é *"que sistema vamos construir?"* — é *"que capability nos falta, e o que precisamos para desenvolvê-la?"*
