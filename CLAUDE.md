# Lead System

Sistema centralizado de coleta e qualificação de leads para uma software house que vende produtos digitais prontos e personalizados. Unifica dois tipos de lead em um único funil: **contato direto** (inbound — o cliente procura a empresa via Meta/Telegram/site) e **busca local** (outbound — a empresa prospecta ativamente via Google Maps/CNPJ). Depois do primeiro contato, toda a conversa com o lead passa a acontecer dentro do próprio sistema (chat integrado a WhatsApp/Telegram), sem precisar abrir o app do canal.

A documentação completa do domínio está em `00-documentation/` — leia-a antes de tomar decisões de modelagem, rota ou negócio:

- [`01-estudo-de-caso-coleta-leads.md`](00-documentation/01-estudo-de-caso-coleta-leads.md) — contexto de negócio, personas, controle de acesso, fluxos (inbound/outbound), funcionalidades do MVP, arquitetura de alto nível, riscos.
- [`02-bibliotecas-e-apis.md`](00-documentation/02-bibliotecas-e-apis.md) — bibliotecas e APIs externas escolhidas para backend e frontend, com o motivo de cada uma.
- [`03-entidades.md`](00-documentation/03-entidades.md) — entidades do modelo de dados, campos, tipos e relações (nomes em inglês).
- [`04-rotas-e-telas.md`](00-documentation/04-rotas-e-telas.md) — rotas do frontend (Next.js App Router) e da API pública, com a permissão exigida em cada uma.

## Estrutura do repositório

Este diretório é apenas um agrupador de documentação; **`backend/` e `frontend/` são repositórios Git independentes** (cada um com seu próprio `.git`, não são submodules). Rode comandos `git` de dentro de cada um deles, não na raiz.

- `00-documentation/` — estudo de caso e documentos técnicos complementares (fonte da verdade do domínio).
- `backend/` — ainda vazio (apenas inicializado com `git init`); a API Java/Spring Boot ainda não foi criada.
- `frontend/` — scaffold padrão do `create-next-app` (Next.js 16, React 19, TypeScript, Tailwind v4), ainda sem nenhuma tela do domínio implementada. Tem seu próprio `AGENTS.md`/`CLAUDE.md`.

## Stack (definida na documentação, ainda não implementada)

- **Backend**: Java + Spring Boot (Web, Validation, Data JPA/Hibernate, Security), PostgreSQL + Flyway, springdoc-openapi, JJWT, Bucket4j (rate limiting dos endpoints públicos), WebSocket (STOMP) para tempo real. Testes: JUnit 5, Mockito, Testcontainers, MockMvc.
- **Frontend**: Next.js (App Router) + TypeScript, componentes **exclusivamente do design system [base-ds]** (gap de componente → abrir issue lá antes de criar um fora do design system), TanStack Query, React Hook Form + Zod, dnd-kit (Kanban), @stomp/stompjs (chat em tempo real). Testes: Vitest/Jest, React Testing Library, Playwright.
- **Metodologia**: TDD obrigatório em backend e frontend — testes antes da implementação.

### TDD

- Testes **antes** da implementação, sempre
- Plano obrigatório em 3 seções antes de qualquer implementação:
  1. Problemas encontrados (arquivo, linha, diagnóstico)
  2. Testes a incluir/alterar (agrupados por arquivo)
  3. O que entra no projeto (tipos → backend → BFF → componentes → páginas)
- Antes de fazer qualquer alteração de código, salvar esse plano como arquivo em `plans/` na raiz do projeto afetado (`frontend/plans/` ou `backend/plans/`) — só começar a implementar depois de o plano estar salvo.
- Depois de salvar o plano, aguardar a confirmação do usuário antes de mexer no código — nunca começar a implementar só porque o plano foi salvo.

### UI — `base-ds` é obrigatório

Sempre usar `Button`, `Card`, `Dialog`, `FormField`, `FileUpload`, `Drawer`, `Sidebar`, `Navbar`, `Footer`, `Heading`, `Image`, `Skeleton`, `useToast`, `ToastProvider`.
Só usar elemento HTML nativo quando genuinamente não houver equivalente no base-ds — avisar antes.

Se a aplicação precisar de uma alteração em um componente existente do `base-ds` ou de um componente novo, abrir uma issue no repositório `indianous/base-ds` (GitHub) descrevendo a necessidade — antes de implementar workaround local.

## Verificação visual

Todas as verificações visuais (telas, fluxos no navegador, screenshots) são feitas pelo próprio usuário. Não rodar a aplicação no Chrome nem usar automação de navegador para conferir o resultado — não invocar a skill `claude-in-chrome` nem ferramentas `mcp__claude-in-chrome__*` neste projeto.

## Convenções importantes do domínio

- Nomes de entidades, campos e valores de enum: **inglês**. Descrições/documentação de negócio: **português**.
- Todo valor monetário é inteiro em **centavos** (`_cents`), nunca float/decimal (ex.: `Product.min_price_cents`, `Lead.estimated_budget_cents`).
- **Sem autocadastro**: não existe rota `/register` nem tela pública de criação de conta. Usuários só existem se criados por alguém com a permissão `CREATE_USER`. Não crie fluxos de signup público.
- Dois canais de mensageria contínua: WhatsApp (Meta) e Telegram. `Conversation`/`Message` registram o chat integrado; `Interaction` é reservada para ligações/observações que não passam pelo chat. O canal `WEBSITE` nunca tem `Conversation` própria.
- Leads de Meta/Telegram são cadastrados **manualmente** pelo vendedor (sem webhook de criação automática no MVP); leads do site chegam via `POST /api/public/leads` (endpoint público autenticado por token de integração, fora do modelo de permissões de usuário).
- Permissões seguem RBAC simples por `key` (ex.: `VIEW_ALL_LEADS`, `VIEW_OWN_LEADS`, `CREATE_USER`, `EDIT_CATALOG`, `VIEW_METRICS`, `TRIGGER_PROSPECTING`) — ver detalhes em `03-entidades.md`.
