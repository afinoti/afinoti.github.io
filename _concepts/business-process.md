---
title: Business Process
---

# Business Process

Um **Business Process** é uma sequência de atividades — executadas por atores humanos, sistemas automatizados, ou ambos — que juntas produzem o resultado prometido por um [Business Service](/concepts/business-service).

Se o Business Service responde *o que* o negócio entrega, o Business Process responde *como* essa entrega acontece, passo a passo.

> Um Business Service pode ser realizado por diferentes Business Processes. Automatizar um processo não altera o Business Service — altera apenas como ele é entregue.

---

## Características

- **Tem início e fim definidos** — é disparado por um evento e termina com um resultado
- **Produz um outcome mensurável** — o resultado prometido pelo Business Service
- **Envolve atores** — humanos, sistemas automatizados, ou ambos em diferentes etapas
- **Pode ter ramificações** — decisões, caminhos alternativos, tratamento de exceção
- **Pode ser as-is ou to-be** — o TOGAF usa Business Processes para descrever o estado atual e o estado alvo da arquitetura

---

## Business Process e Business Service

Os dois conceitos são complementares e não se substituem:

| Conceito | Responde | Muda quando automatiza? |
|---|---|---|
| **Business Service** | O que o negócio entrega, e para quem | Não |
| **Business Process** | Como aquela entrega acontece, passo a passo | Sim |

A separação é intencional: ela permite que a mesma capacidade de negócio seja realizada por processos completamente diferentes — manual hoje, automatizado amanhã — sem que o compromisso com o consumidor mude.

---

## Atores em um Business Process

Um Business Process é executado por **Business Actors** desempenhando **Business Roles**. Um Role é abstrato — descreve *que tipo* de ator é necessário. Um Actor é concreto — é quem ou o que preenche aquele Role.

O mesmo Role pode ser preenchido por um humano em um processo manual e por um sistema em um processo automatizado. Isso é o que torna a transição entre os dois possível sem alterar a definição do Business Service.

---

## Exemplo: Atendimento ao Cliente

Do nosso [exemplo do chatbot](/examples/chatbot), o Business Service **"Atendimento ao Cliente"** realizado por dois processos diferentes:

### Processo manual (as-is)

| Passo | Atividade | Ator |
|---|---|---|
| 1 | Cliente envia mensagem | Cliente (humano) |
| 2 | Atendente lê e interpreta a mensagem | Atendente de suporte (humano) |
| 3 | Atendente consulta o sistema de pedidos | Atendente de suporte (humano) |
| 4 | Atendente redige e envia a resposta | Atendente de suporte (humano) |
| 5 (exceção) | Atendente escala para especialista | Atendente de suporte (humano) |

### Processo automatizado (to-be)

| Passo | Atividade | Ator |
|---|---|---|
| 1 | Cliente envia mensagem via web ou WhatsApp | Cliente (humano) |
| 2 | Sistema verifica autenticação | Chatbot Go Service |
| 2a (exceção) | Redireciona para `/login` se não autenticado | Auth Service |
| 3 | Sistema classifica a intenção (FAQ ou consulta de pedido) | Chatbot Python AI Service |
| 4a | Se FAQ: recupera resposta da base de conhecimento e responde | Chatbot Python AI Service |
| 4b | Se pedido: consulta o Orders Microservice e responde com o status | Chatbot Go Service |
| 5 (exceção) | Se não resolvido: oferece escalonamento para atendente humano | Chatbot Go Service |

O Business Service não mudou. O que mudou foi quem executa cada passo — e a consequência arquitetural dessa mudança são os Application Services, componentes e infraestrutura documentados nos outros níveis da arquitetura.

---

## Relação com os outros domínios

O Business Process é o ponto de partida para derivar os requisitos dos domínios abaixo:

- Cada **passo automatizado** de um Business Process requer um ou mais **Application Services**
- Cada Application Service é realizado por **Logical e Physical Application Components**
- Os dados manipulados em cada passo tornam-se **Data Entities e Data Components**
- A infraestrutura que executa tudo isso é descrita pela **Technology Architecture**

```
Business Process  →  cada passo automatizado
                          │
                          ▼
                  Application Service  →  o que o software precisa fazer
                          │
                          ▼
                  Logical Application Component  →  quem faz
                          │
                          ▼
                  Physical Application Component  →  o que está rodando
```

---

## Referências

- [The Open Group — TOGAF Standard, Version 9.2: Phase B — Business Architecture](https://pubs.opengroup.org/architecture/togaf9-doc/arch/chap08.html)
