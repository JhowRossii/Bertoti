# Minha Contribuição — Projeto SophIA

> Um assistente de recomendação de livros construído como um **agente de IA** com arquitetura baseada em design patterns.
> Stack: Java 21 · Spring Boot 4 · Spring AI · PostgreSQL + pgvector · Ollama · JavaFX.

Este documento descreve a minha parte no projeto: o que eu fiz, onde apliquei e o que aprendi.

---

## O projeto, em uma frase

A SophIA recebe um pedido em linguagem natural ("quero um romance de época"), entende a intenção, busca no catálogo combinando **busca semântica** (por significado, com embeddings) e **filtros estruturados**, e escreve uma recomendação — tudo com IA rodando localmente.

---

## Minha parte no projeto

Fiquei com **três design patterns** da arquitetura do agente.

### ⛓️ Chain of Responsibility — os advisors

Processa a requisição em uma *corrente* de elos, em volta do agente. Implementei advisors que **validam e normalizam** a entrada antes de o agente rodar (podendo **interromper** a corrente se a pergunta for inválida) e que **ajustam a saída** depois que o agente termina. O último elo da corrente é a execução do próprio agente — o que me obrigou a entender bem a ideia de "lógica em volta" (antes e depois).

**Onde:** `agent/advisor/` (`Advisor`, `AdvisorChain`, `InputGuardAdvisor`, `ResponsePolishAdvisor`).

### 👀 Observer — o monitoramento

O agente é o *sujeito* e notifica os observers a cada transição de estado. Implementei a interface `AgentObserver` e dois observadores: um de **log** (registra cada transição e comando) e um de **métricas** (execuções, transições, falhas). Isso tirou o monitoramento de dentro da lógica do agente e deixou fácil adicionar novos monitores.

**Onde:** `agent/observer/` (`AgentObserver`, `LoggingObserver`, `MetricsObserver`).

### 🔌 Injeção de Dependência — a cola do sistema

Garanti que todas as peças fossem conectadas por **injeção de dependência** do Spring, incluindo o truque de injetar uma **lista** de beans (`List<Command>`, `List<Planner>`, `List<StateHandler>`) para que os registries e o agente se montem sozinhos. Resultado: adicionar uma peça nova é só criar uma classe `@Component`.

### 🔧 Integração e o que faltava para funcionar perfeitamente

Fui o responsável por fazer o projeto **rodar de verdade**, resolvendo problemas reais:

- **Confiabilidade da busca por gênero/autor:** troquei a extração de filtros para casar a pergunta com os **valores reais do catálogo**, mais um **fallback semântico** quando um filtro não encontra nada.
- **Performance:** reduzi o número de chamadas ao modelo por pergunta e ajustei limites de geração e timeout, derrubando o tempo de resposta.
- **Carga automática:** um *auto-seed* que popula o catálogo e gera os embeddings sozinho no primeiro boot.
- **Subida do ambiente:** depurei erros de inicialização (Docker Compose, conexão com o PostgreSQL) até o projeto subir redondo.

---

## O que eu aprendi

- **Design patterns na prática** — vendo Chain of Responsibility, Observer e Injeção de Dependência resolverem problemas reais e se encaixarem com State Machine, Command, Strategy e Composite.
- **Arquitetura de agente de IA** — a ideia de planejar → agir → observar → repetir, e como uma máquina de estados organiza esse ciclo.
- **RAG e busca semântica** — o que é um **embedding**, como a **distância de cosseno** no **pgvector** encontra o que tem significado parecido, e como combinar isso com filtros estruturados.
- **Spring Boot e Injeção de Dependência** — como o framework monta e conecta os objetos.
- **Docker** — subir um ecossistema (aplicação + PostgreSQL + Ollama) e entender por que cada peça precisa estar no ar.
- **Depuração de verdade** — ler stack traces, isolar a causa e corrigir, em vez de só tentar de novo.

---

## O que levo desse projeto

Saí com uma noção muito mais concreta de **como padrões de projeto deixam um sistema organizado, testável e fácil de estender** — e de como uma aplicação de IA de verdade se monta por trás (dados, embeddings, orquestração e geração). Mais do que implementar um padrão isolado, aprendi a fazê-los **trabalharem juntos** dentro de uma arquitetura que funciona de ponta a ponta — e a assumir a parte menos glamourosa e mais importante: fazer tudo **funcionar**.

---

*Projeto desenvolvido em equipe. A visão completa do projeto e os créditos de todos os integrantes estão no `README.md` principal.*
