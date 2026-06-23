---
title: Service, Quality e Contract
---

# Service, Quality e Contract

No TOGAF, o [Business Service](/concepts/business-service) define *o que* o negócio entrega. Mas essa definição por si só está incompleta: ela não diz *quão bem* o serviço deve operar, nem *com quem* o negócio firmou esse compromisso.

O trio **Service, Quality e Contract** não introduz novos artefatos arquiteturais — ele complementa o Business Service com duas dimensões que ele sozinho não cobre. O "Service" do trio *é* o próprio Business Service. Os conceitos genuinamente novos aqui são **Quality** e **Contract**.

---

## Quality

**Service Quality** (ou Quality of Service — QoS) são os atributos **não-funcionais** que definem *quão bem* um serviço executa sua função. Esses atributos não estão na lógica do serviço; estão nas condições sob as quais ele opera.

O TOGAF define Service Quality como uma característica não-funcional que qualifica o desempenho de um serviço. Exemplos:

| Atributo | Pergunta que responde |
|---|---|
| **Disponibilidade** | O serviço estará acessível quando necessário? |
| **Performance** | Quão rápido ele responde? |
| **Segurança** | Quem pode acessá-lo e como é protegido? |
| **Escalabilidade** | Ele sustenta picos de demanda? |
| **Confiabilidade** | Ele entrega resultados consistentes? |
| **Manutenibilidade** | Com que facilidade pode ser alterado ou corrigido? |

Service Quality é o que transforma um Business Service de uma intenção vaga em um compromisso mensurável. "Consulta de status de pedido" é uma intenção. "Consulta de status de pedido disponível 99,9% do tempo, com resposta em até 2 segundos para 95% das requisições" é um compromisso.

---

## Contract

Um **Contract** (ou Service Contract) é o acordo formal entre o **provedor** do serviço e seu **consumidor**. Ele une os dois conceitos anteriores: especifica *qual* serviço é fornecido e *com qual qualidade*.

Um Contract responde:

- O que está sendo entregue (a especificação funcional do Service)?
- Sob quais condições de qualidade (os atributos de Service Quality)?
- O que acontece quando o provedor não cumpre (penalidades, fallback)?
- Por quanto tempo esse acordo é válido?

No contexto de negócio, um Contract pode ser um SLA formal com um cliente externo. No contexto interno de uma organização, pode ser um acordo entre times sobre o nível de serviço que um sistema garante aos seus consumidores.

---

## Como Quality e Contract completam o Business Service

O Business Service é o ponto de partida — ele define a capacidade que o negócio expõe. Mas apenas o Business Service não é suficiente para governar a entrega. É preciso qualificar como ele opera e formalizar o compromisso com quem o consome.

```
Business Service  ←── o "Service" do trio; já definido em Phase B
       │
       ├── Quão bem entrega? ────────────── Quality (atributos não-funcionais)
       │
       └── Com quem e sob quais termos? ─── Contract (acordo formal)
```

Quality e Contract juntos tornam o Business Service **governável**: ele pode ser monitorado, auditado e renegociado quando necessário.

---

## Exemplo: Chatbot de Atendimento

Usando o Business Service **"Atendimento ao Cliente"** do nosso [exemplo do chatbot](/examples/chatbot):

**Quality**
- Disponibilidade: 99,5% em horário comercial
- Tempo de resposta: até 3 segundos para respostas de FAQ; até 5 segundos para consulta de pedido
- Segurança: acesso restrito a usuários autenticados; histórico de conversa não exposto a terceiros
- Canal: disponível via web e WhatsApp

**Contract**
- Firmado entre: time de produto (provedor) e área de CX (consumidora)
- Compromisso: os atributos de Quality acima, revisados trimestralmente
- Violação: abertura automática de incidente se disponibilidade cair abaixo de 99% em 30 dias

---

## Por que isso importa na prática

Um Business Service sem Quality é uma promessa sem prazo. Um Business Service sem Contract é uma promessa sem dono.

A maioria das falhas de arquitetura não acontece porque o serviço foi mal especificado funcionalmente — acontece porque os atributos de qualidade nunca foram explicitados, ou porque não há um acordo claro sobre quem é responsável por mantê-los. Service, Quality e Contract existem para tornar esses compromissos explícitos e rastreáveis.

---

## Referências

- [The Open Group — TOGAF Standard, Version 9.2: Service Quality](https://pubs.opengroup.org/architecture/togaf9-doc/arch/chap34.html)
- [The Open Group — TOGAF Standard, Version 9.2: Architecture Contracts](https://pubs.opengroup.org/architecture/togaf9-doc/arch/chap23.html)
