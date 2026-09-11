# Bibliotecas e APIs do Sistema

> Documento complementar ao [Estudo de Caso](./01-estudo-de-caso-coleta-leads.md). Lista as bibliotecas, frameworks e APIs externas previstas para a implementação, organizadas por backend e frontend. Versões específicas devem ser definidas/travadas no início da implementação (`pom.xml` / `package.json`); aqui o objetivo é registrar a escolha e o motivo de cada uma.

---

## Backend (Java)

### Framework e core

| Biblioteca | Uso | Motivo |
|---|---|---|
| **Spring Boot** | Framework principal da aplicação (Web, IoC, configuração) | Produtividade e maturidade no ecossistema Java; base natural para os demais módulos Spring abaixo |
| **Spring Web (MVC)** | Exposição da API REST (leads, produtos, usuários, prospecção) | Padrão de fato para APIs REST em Spring Boot |
| **Spring Validation** | Validação de payloads de entrada (ex.: dados do formulário do site, cadastro de lead) | Evita lead/produto/usuário inconsistente já na borda da API |
| **springdoc-openapi** | Geração automática de documentação OpenAPI/Swagger | Necessário para documentar formalmente o contrato do **endpoint público de recepção de leads do site**, consumido por um time externo ao projeto |

### Persistência

| Biblioteca | Uso | Motivo |
|---|---|---|
| **Spring Data JPA (Hibernate)** | Acesso a dados / ORM para Lead, Produto, Usuário, Permissão, Conversa, Mensagem, Interação, Status do Funil | Reduz boilerplate de acesso a dados mantendo controle sobre as consultas |
| **PostgreSQL JDBC Driver** | Conexão com o banco PostgreSQL | Banco definido no estudo de caso |
| **Flyway** | Versionamento e migração de schema do banco | Rastreabilidade de mudanças de schema, essencial junto com TDD (migrações previsíveis por ambiente de teste) |

### Autenticação e permissões

| Biblioteca | Uso | Motivo |
|---|---|---|
| **Spring Security** | Autenticação e controle de acesso (RBAC) | Base para o modelo de papéis/permissões — sem tela de autocadastro, com criação de usuário restrita |
| **JJWT (Java JWT)** ou **Spring Authorization Server** | Emissão/validação de tokens de sessão da API interna | Autenticação stateless para o frontend Next.js consumir a API |
| **API Key / token de integração (implementação própria sobre Spring Security)** | Proteção do endpoint público que recebe leads do site | O site é externo ao projeto e não passa pelo login de usuário — precisa de um mecanismo de autenticação próprio (chave de API) |
| **Verificação de assinatura de webhook (implementação própria com `javax.crypto`/HMAC-SHA256)** | Validação da autenticidade das chamadas recebidas nos webhooks de mensagens (Meta/Telegram) | Cada provedor assina o payload do webhook; validar a assinatura evita processar mensagens forjadas |
| **Bucket4j** | Rate limiting dos endpoints públicos (recepção de leads do site e webhooks de mensagens) | Mitiga abuso/spam nesses endpoints, que são públicos por natureza |

### Testes (TDD)

| Biblioteca | Uso | Motivo |
|---|---|---|
| **JUnit 5** | Framework de testes unitários e de integração | Padrão do ecossistema Java; base do fluxo de TDD adotado no projeto |
| **Mockito** | Mocks/stubs em testes unitários (ex.: simular API do Google Maps, WhatsApp, Telegram) | Isola a lógica de negócio das integrações externas nos testes unitários |
| **Testcontainers** | Sobe um PostgreSQL real em container para testes de integração | Testes de persistência mais próximos do ambiente real, sem depender de banco em memória divergente do Postgres |
| **Spring Boot Test / MockMvc** | Testes de integração da camada web (controllers/API) | Testa o contrato da API (incluindo o endpoint público do site) de ponta a ponta |

### Integrações externas — mensageria com o lead (chat)

> Usa as mesmas plataformas (Meta/Telegram) das notificações internas abaixo, mas para um propósito diferente: aqui é a **conversa comercial com o lead**, não o aviso ao usuário do sistema.

| API | Uso | Motivo |
|---|---|---|
| **WhatsApp Business Platform (Cloud API, Meta)** — endpoint `messages` | Envio de mensagens do vendedor para o lead | Canal de chat definido no estudo de caso para leads de Meta/busca local |
| **WhatsApp Business Platform — Webhooks** | Recebimento das respostas do lead e status de entrega/leitura das mensagens enviadas | Necessário para trazer a resposta do cliente de volta para dentro do sistema |
| **Telegram Bot API** — método `sendMessage` | Envio de mensagens do vendedor para o lead | Canal de chat definido no estudo de caso |
| **Telegram Bot API — `setWebhook` / atualizações recebidas** | Recebimento das mensagens que o lead envia ao bot | Necessário para trazer a resposta do cliente de volta para dentro do sistema |

### Comunicação em tempo real

| Biblioteca | Uso | Motivo |
|---|---|---|
| **Spring WebSocket (STOMP sobre SockJS)** | Envia ao frontend, em tempo real, as mensagens recebidas via webhook e atualizações de status de entrega | Evita que o vendedor precise atualizar a página para ver a resposta do lead na conversa |

### Integrações externas — notificações

| API | Uso | Motivo |
|---|---|---|
| **WhatsApp Business Platform (Cloud API, Meta)** | Envio de notificação por WhatsApp ao usuário | Canal de notificação escolhido pelo usuário quando fora do sistema |
| **Telegram Bot API** | Envio de notificação por Telegram ao usuário | Canal de notificação escolhido pelo usuário quando fora do sistema |
| **Provedor de e-mail transacional** (ex.: Amazon SES, SendGrid ou Resend — a definir) | Envio de notificação por e-mail ao usuário | Canal de notificação escolhido pelo usuário quando fora do sistema |

### Integrações externas — prospecção (busca local)

| API | Uso | Motivo |
|---|---|---|
| **Google Maps Platform — Places API** | Busca automatizada de empresas locais por região/segmento/palavra-chave | Fonte definida no estudo de caso para leads de busca local |
| **BrasilAPI (CNPJ)** ou **ReceitaWS** (a confirmar qual provedor de dados públicos de CNPJ atende melhor o volume/custo do projeto) | Busca de empresas por CNPJ/CNAE/situação cadastral | Fonte pública de dados de empresas definida no estudo de caso; evita integração direta e instável com o site da Receita Federal |
| **Spring Scheduler (`@Scheduled`)** ou **Quartz** (se precisar de agendamento mais robusto) | Disparo periódico das buscas de prospecção por região | Automatiza a busca local sem depender de acionamento manual a cada vez |

---

## Frontend (Next.js)

### Framework e core

| Biblioteca | Uso | Motivo |
|---|---|---|
| **Next.js (App Router)** | Framework do dashboard/admin do sistema | Definido no estudo de caso |
| **React** | Biblioteca de UI subjacente ao Next.js | Requisito do Next.js |
| **TypeScript** | Tipagem estática do frontend | Reduz erros de integração com a API (contratos de Lead, Produto, Usuário, Permissão) e facilita TDD no frontend |
| **base-ds** ([github.com/indianous/base-ds](https://github.com/indianous/base-ds)) | Componentes de interface (botões, formulários, cards, tabelas, etc.) | Design system definido no estudo de caso — toda tela deve usar seus componentes; gaps viram issue no repositório |

### Dados e formulários

| Biblioteca | Uso | Motivo |
|---|---|---|
| **TanStack Query (React Query)** | Busca, cache e sincronização de dados da API (leads, funil, métricas) | Simplifica estados de loading/erro/refetch do dashboard sem gerenciar cache manualmente |
| **React Hook Form** | Gerenciamento de formulários (cadastro manual de lead, cadastro de usuário, catálogo de produtos) | Performance e controle simples de formulários complexos |
| **Zod** | Validação de esquema dos formulários no frontend | Mesma linguagem de validação pode ser espelhada nas regras de validação do backend, reduzindo divergência |

### Autenticação (frontend)

| Biblioteca | Uso | Motivo |
|---|---|---|
| **Auth.js (NextAuth)** ou integração customizada com o JWT emitido pelo backend (a decidir na implementação) | Sessão de login do usuário no dashboard | Sem tela de autocadastro — login apenas para usuários já cadastrados por um administrador |

### Interface específica do domínio

| Biblioteca | Uso | Motivo |
|---|---|---|
| **dnd-kit** | Drag-and-drop dos cards de lead no dashboard estilo Kanban | Move leads entre etapas do funil (Novo → Contatado → Proposta → Negociação → Fechado/Perdido) |
| **date-fns** | Formatação e cálculo de datas (tempo de primeira resposta, tempo até fechamento) | Usado nas métricas de sucesso do funil |
| **@stomp/stompjs** | Cliente WebSocket (STOMP) para a tela de conversa/chat do lead | Recebe mensagens novas e atualizações de status em tempo real do backend, sem polling |

### Testes (TDD)

| Biblioteca | Uso | Motivo |
|---|---|---|
| **Vitest** ou **Jest** (a decidir conforme integração com Next.js) | Testes unitários de componentes e lógica do frontend | Base do fluxo de TDD no frontend |
| **React Testing Library** | Testes de componentes focados em comportamento visível ao usuário | Complementa o testador de unidade, evitando testes acoplados a detalhes de implementação |
| **Playwright** | Testes end-to-end (ex.: fluxo de login, cadastro manual de lead, mover card no Kanban) | Cobre fluxos completos que dependem da integração real entre frontend e API |

---

## Observações

- O **site institucional com o formulário de contato do lead está fora do escopo deste projeto** (ver estudo de caso) — por isso ele não aparece nas listas acima: o único ponto de contato entre ele e este sistema é o endpoint público de API descrito na seção de backend.
- APIs de canais futuros (outras redes sociais além de Meta/Telegram) serão adicionadas a este documento conforme forem priorizadas.
- A **central de mensagens** cobre apenas os canais Meta e Telegram — o formulário do site não é um canal de conversa contínua, apenas de captação pontual (ver `04-rotas-e-telas.md`).
- Provedores de e-mail transacional e de dados de CNPJ estão listados como opções a confirmar — a escolha final depende de custo, limites de uso e volume esperado, e deve ser validada antes do início da implementação da integração correspondente.
