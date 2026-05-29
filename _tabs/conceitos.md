---
layout: page
icon: fas fa-book
order: 5
---

# Conceitos

Definições e conceitos fundamentais da computação.

---

## App (Aplicação)

> Definição baseada na metodologia [The Twelve-Factor App](https://12factor.net/codebase)

Uma **app** é uma unidade de software rastreada em um único sistema de controle de versão — chamado de **codebase**. Em sistemas centralizados (como Subversion), o codebase é um único repositório. Em sistemas distribuídos (como Git), é o conjunto de repositórios que compartilham um commit raiz.

### Codebase

A metodologia estabelece uma correlação rígida: **um codebase, uma app**. Isso implica:

- Se há múltiplos codebases, trata-se de um **sistema distribuído**, não de uma única app — cada componente é uma app separada.
- Múltiplas apps compartilhando o mesmo codebase violam o princípio. O código compartilhado deve ser extraído em bibliotecas independentes.

### Deploys

Um **deploy** é uma instância em execução da app. Exemplos:

- O ambiente de **produção** (site ao vivo)
- Um ou mais ambientes de **staging**
- A **cópia local** do desenvolvedor

Existe apenas **um codebase** por app, mas pode haver **múltiplos deploys**. O codebase é o mesmo em todos os deploys, porém versões diferentes podem estar ativas em cada ambiente — o desenvolvedor pode ter commits ainda não promovidos para staging, e staging pode ter commits ainda não promovidos para produção.
