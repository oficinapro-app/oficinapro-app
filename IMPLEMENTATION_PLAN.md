# OficinaPro — Plano de Implementação V1

**Autor:** Manus AI (CTO & Gerente de Projeto)
**Data:** 09 de junho de 2026
**Versão:** 1.0
**Objetivo:** Este documento detalha o plano de implementação para a V1 do OficinaPro, um SaaS multiempresa para oficinas mecânicas. O plano é derivado da análise de toda a documentação existente (PRD, UI/UX, Database, Frontend, Backend, API, Component Library, Testing, Deployment, Roadmap, Marketing) e consiste em tarefas atômicas, sequenciais e altamente detalhadas, projetadas para serem implementadas de forma eficiente e sem ambiguidades.

## Visão Geral do Plano

O plano de implementação está dividido em fases lógicas, garantindo que as dependências sejam respeitadas e que o desenvolvimento progrida de forma estruturada. Cada tarefa é projetada para ser pequena e focada, permitindo que equipes ou IAs (como Claude, Cursor) possam trabalhar de forma independente em unidades de trabalho bem definidas.

**Stack Tecnológica:**
*   **Frontend:** Next.js 15, TypeScript, Tailwind CSS, shadcn/ui
*   **Backend/Database:** Supabase (PostgreSQL)
*   **Pagamentos:** Stripe
*   **Hospedagem:** Vercel
*   **Controle de Versão:** GitHub

## Ordem Ideal de Execução das Fases

1.  **Foundation:** Configuração inicial do projeto e ferramentas de desenvolvimento.
2.  **Supabase Core:** Configuração do Supabase, schema inicial do banco de dados, enums, triggers e RLS básicos.
3.  **Autenticação:** Implementação completa dos fluxos de login, cadastro, recuperação e alteração de senha.
4.  **Multiempresa Core:** Configuração de tenants, perfis e associação de usuários a tenants.
5.  **Layout Base:** Implementação do layout principal da aplicação (Sidebar, Header, Dashboard base).
6.  **Módulo Clientes:** CRUD completo para gestão de clientes.
7.  **Módulo Veículos:** CRUD completo para gestão de veículos.
8.  **Módulo Serviços:** CRUD completo para gestão de serviços.
9.  **Módulo Produtos:** CRUD completo para gestão de produtos.
10. **Módulo Ordens de Serviço:** Implementação do fluxo completo de OS.
11. **Módulo Financeiro:** Gestão de receitas, despesas, contas a receber e a pagar.
12. **Módulo Assinaturas:** Integração com Stripe para gestão de planos e webhooks.
13. **Módulo Configurações:** Gestão de perfil, empresa e usuários.
14. **Refinamentos e Testes:** Implementação de testes, otimizações de performance e segurança.

## Dependências entre Fases

*   **Foundation** -> Todas as outras fases.
*   **Supabase Core** -> Autenticação, Multiempresa Core, Módulos de Dados.
*   **Autenticação** -> Multiempresa Core, Layout Base, Módulos de Dados (para contexto de usuário).
*   **Multiempresa Core** -> Layout Base (para seleção de tenant), Módulos de Dados (para `tenant_id`).
*   **Layout Base** -> Todos os módulos de funcionalidade (para renderização das páginas).
*   **Módulos de Dados (Clientes, Veículos, Serviços, Produtos)** -> Ordens de Serviço, Financeiro.
*   **Ordens de Serviço** -> Financeiro (para geração de contas a receber).
*   **Assinaturas** -> Multiempresa Core (para status do tenant), Financeiro (para receitas de assinatura).
*   **Configurações** -> Multiempresa Core (para gestão de usuários/tenant).
*   **Refinamentos e Testes** -> Todas as fases de implementação.

## Fases e Tarefas Detalhadas

---

## Fase 1: Foundation

**Objetivo:** Configurar o ambiente de desenvolvimento e a estrutura base do projeto.

### Task 001: Inicializar projeto Next.js 15 com TypeScript
*   **Objetivo:** Criar um novo projeto Next.js com suporte a TypeScript.
*   **Descrição:** Utilizar `create-next-app` para iniciar o projeto, selecionando as opções para TypeScript, ESLint, Tailwind CSS, `src/` directory, App Router e sem Custom import alias.
*   **Arquivos envolvidos:** `package.json`, `tsconfig.json`, `next.config.js`, `postcss.config.js`, `tailwind.config.ts`, `globals.css`.
*   **Dependências:** Nenhuma.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Projeto Next.js criado e rodando localmente sem erros de compilação ou ESLint.
*   **Como testar:** Executar `npm install` e `npm run dev`. Acessar `http://localhost:3000` no navegador.

### Task 002: Configurar Tailwind CSS
*   **Objetivo:** Garantir que o Tailwind CSS esteja corretamente integrado e funcionando no projeto.
*   **Descrição:** Verificar `tailwind.config.ts` para paths de arquivos de conteúdo e `globals.css` para as diretivas `@tailwind`. Testar com uma classe Tailwind simples em `app/page.tsx`.
*   **Arquivos envolvidos:** `tailwind.config.ts`, `globals.css`, `app/page.tsx`.
*   **Dependências:** Task 001.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Classes Tailwind aplicadas em elementos HTML resultam na estilização esperada no navegador.
*   **Como testar:** Adicionar `className="text-blue-500 text-3xl"` a um `<h1>` em `app/page.tsx` e verificar se o texto fica azul e grande.

### Task 003: Configurar shadcn/ui (CLI)
*   **Objetivo:** Instalar e configurar a CLI do shadcn/ui para adicionar componentes ao projeto.
*   **Descrição:** Executar `npx shadcn-ui@latest init` e seguir as instruções, configurando o tema padrão, cores e fonte Inter. Escolher `New York` como style e `Inter` como base font. Configurar `globals.css` para importar a fonte Inter.
*   **Arquivos envolvidos:** `components.json`, `tailwind.config.ts`, `globals.css`, `lib/utils.ts`.
*   **Dependências:** Task 002.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** CLI do shadcn/ui configurada, `components.json` criado, `tailwind.config.ts` atualizado com as variáveis CSS do shadcn/ui, e a fonte Inter importada corretamente.
*   **Como testar:** Verificar a existência de `components.json` e as modificações nos arquivos `tailwind.config.ts` e `globals.css`.

### Task 004: Adicionar primeiro componente shadcn/ui (Button)
*   **Objetivo:** Validar a instalação do shadcn/ui adicionando um componente básico.
*   **Descrição:** Executar `npx shadcn-ui@latest add button` e importar o componente `Button` em `app/page.tsx` para testar.
*   **Arquivos envolvidos:** `components/ui/button.tsx`, `app/page.tsx`.
*   **Dependências:** Task 003.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** O componente `Button` é renderizado corretamente na página, com o estilo do shadcn/ui e a fonte Inter.
*   **Como testar:** Adicionar `<Button>Clique-me</Button>` em `app/page.tsx` e verificar a renderização no navegador.

### Task 005: Configurar ESLint e Prettier
*   **Objetivo:** Garantir a consistência do código e a qualidade através de linting e formatação automática.
*   **Descrição:** Configurar `.eslintrc.json` e `.prettierrc` com regras recomendadas para Next.js, TypeScript e Tailwind. Adicionar scripts `lint` e `format` ao `package.json`.
*   **Arquivos envolvidos:** `.eslintrc.json`, `.prettierrc`, `package.json`.
*   **Dependências:** Task 001.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** `npm run lint` e `npm run format` executam sem erros e formatam o código conforme as regras definidas.
*   **Como testar:** Introduzir um erro de linting ou formatação em um arquivo e executar os scripts para verificar a correção.

### Task 006: Estruturar pastas iniciais
*   **Objetivo:** Organizar a estrutura de pastas do projeto conforme o `FRONTEND_SPEC.md` e `BACKEND_SPEC.md`.
*   **Descrição:** Criar as pastas `app/`, `components/ui/`, `components/common/`, `components/modules/`, `hooks/`, `lib/`, `services/`, `types/`, `validations/`, `public/`, `styles/`.
*   **Arquivos envolvidos:** Estrutura de diretórios.
*   **Dependências:** Task 001.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Todas as pastas listadas existem na raiz do projeto.
*   **Como testar:** Verificar a estrutura de pastas no explorador de arquivos.

### Task 007: Configurar variáveis de ambiente (.env.local)
*   **Objetivo:** Preparar o ambiente para variáveis de ambiente sensíveis e públicas.
*   **Descrição:** Criar o arquivo `.env.local` e adicionar as variáveis `NEXT_PUBLIC_SUPABASE_URL` e `NEXT_PUBLIC_SUPABASE_ANON_KEY` (com valores placeholder por enquanto).
*   **Arquivos envolvidos:** `.env.local`, `.gitignore`.
*   **Dependências:** Nenhuma.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** `.env.local` criado e adicionado ao `.gitignore`. Variáveis acessíveis no código (ex: `process.env.NEXT_PUBLIC_SUPABASE_URL`).
*   **Como testar:** Adicionar um `console.log(process.env.NEXT_PUBLIC_SUPABASE_URL)` em `app/page.tsx` e verificar a saída no console do servidor.

---

## Fase 2: Supabase Core

**Objetivo:** Configurar a integração com Supabase e o esquema inicial do banco de dados.

### Task 008: Criar projeto Supabase
*   **Objetivo:** Provisionar uma nova instância de projeto no Supabase.
*   **Descrição:** Acessar o dashboard do Supabase, criar um novo projeto e obter as credenciais (URL e `anon_key`).
*   **Arquivos envolvidos:** `.env.local` (atualização).
*   **Dependências:** Nenhuma.
*   **Banco de dados envolvido:** Supabase Project.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Projeto Supabase criado e credenciais `NEXT_PUBLIC_SUPABASE_URL` e `NEXT_PUBLIC_SUPABASE_ANON_KEY` atualizadas em `.env.local`.
*   **Como testar:** Verificar as credenciais no dashboard do Supabase e no arquivo `.env.local`.

### Task 009: Instalar Supabase CLI
*   **Objetivo:** Habilitar o gerenciamento do Supabase localmente e a criação de migrations.
*   **Descrição:** Instalar a Supabase CLI globalmente ou via `npm` no projeto. Executar `supabase login`.
*   **Arquivos envolvidos:** `package.json` (se instalado localmente).
*   **Dependências:** Task 008.
*   **Banco de dados envolvido:** Supabase Project.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Supabase CLI instalada e autenticada com a conta do Supabase.
*   **Como testar:** Executar `supabase status` para verificar a conexão.

### Task 010: Inicializar Supabase localmente
*   **Objetivo:** Configurar o ambiente de desenvolvimento local do Supabase.
*   **Descrição:** Executar `supabase init` e `supabase start` para iniciar os serviços locais (PostgreSQL, Auth, Storage, etc.).
*   **Arquivos envolvidos:** `supabase/` directory.
*   **Dependências:** Task 009.
*   **Banco de dados envolvido:** Supabase Local.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Serviços Supabase locais rodando, acessíveis via `localhost`.
*   **Como testar:** Acessar o Studio local (`http://localhost:54323`) e verificar se os serviços estão ativos.

### Task 011: Criar migration 001_initial_schema.sql (Extensões e Funções base)
*   **Objetivo:** Criar a primeira migration para configurar extensões e funções SQL básicas.
*   **Descrição:** Criar o arquivo `supabase/migrations/2026MMDDHHMMSS_initial_schema.sql` (ou similar) e adicionar o SQL para `create extension if not exists pgcrypto;` e a função `set_updated_at()` conforme `DATABASE.md`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 010.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Migration criada com sucesso e aplicada ao banco de dados local.
*   **Como testar:** Executar `supabase migration up` e verificar no Studio local se a extensão e a função foram criadas.

### Task 012: Criar migration 002_enums.sql
*   **Objetivo:** Definir todos os tipos `ENUM` do PostgreSQL conforme `DATABASE.md`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar todos os `CREATE TYPE ... AS ENUM` para `tenant_status`, `tenant_role`, `membership_status`, `user_status`, `customer_type`, `record_status`, `fuel_type`, `work_order_status`, `financial_status`, `payment_method`, `transaction_type`, `billing_interval`, `subscription_status`, `notification_type`, `notification_status`, `setting_type`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 011.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Todos os ENUMs definidos no `DATABASE.md` são criados no banco de dados local.
*   **Como testar:** Executar `supabase migration up` e verificar no Studio local (`http://localhost:54323/project/default/database/extensions`) se os tipos foram criados.

### Task 013: Criar migration 003_tables.sql (tenants e profiles)
*   **Objetivo:** Criar as tabelas `tenants` e `profiles` com suas respectivas colunas, chaves primárias e constraints básicas.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `tenants` e `profiles` conforme `DATABASE.md`. Incluir `id`, `legal_name`, `trade_name`, `document`, `email`, `phone`, `whatsapp`, `logo_url`, `status`, `settings`, `created_at`, `updated_at` para `tenants`. Para `profiles`, incluir `id`, `full_name`, `avatar_url`, `created_at`, `updated_at`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 012.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabelas `tenants` e `profiles` criadas com as colunas e tipos corretos no banco de dados local.
*   **Como testar:** Executar `supabase migration up` e verificar no Studio local se as tabelas foram criadas e suas estruturas estão corretas.

### Task 014: Criar migration 004_tables_tenant_users.sql
*   **Objetivo:** Criar a tabela `tenant_users` para associar usuários a tenants.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `tenant_users` com `id`, `tenant_id`, `user_id`, `role`, `status`, `invited_by`, `created_at`, `updated_at`. Definir chaves estrangeiras para `tenants.id` e `profiles.id` e a constraint `UNIQUE (tenant_id, user_id)`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 013.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `tenant_users` criada com chaves estrangeiras e constraint de unicidade.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 015: Criar migration 005_tables_customers.sql
*   **Objetivo:** Criar a tabela `customers`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `customers` com `id`, `tenant_id`, `name`, `customer_type`, `document`, `email`, `phone`, `whatsapp`, `birth_date`, `observations`, `status`, `created_at`, `updated_at`, `deleted_at`. Definir chave estrangeira para `tenants.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 014.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `customers` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 016: Criar migration 006_tables_customer_addresses.sql
*   **Objetivo:** Criar a tabela `customer_addresses`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `customer_addresses` com `id`, `tenant_id`, `customer_id`, `zip_code`, `street`, `number`, `neighborhood`, `city`, `state`, `complement`, `is_main`, `created_at`, `updated_at`, `deleted_at`. Definir chaves estrangeiras para `tenants.id` e `customers.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 015.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `customer_addresses` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 017: Criar migration 007_tables_vehicles.sql
*   **Objetivo:** Criar a tabela `vehicles`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `vehicles` com `id`, `tenant_id`, `customer_id`, `brand`, `model`, `year`, `plate`, `chassis`, `color`, `fuel_type`, `mileage`, `observations`, `created_at`, `updated_at`, `deleted_at`. Definir chaves estrangeiras para `tenants.id` e `customers.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 016.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `vehicles` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 018: Criar migration 008_tables_services.sql
*   **Objetivo:** Criar a tabela `services`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `services` com `id`, `tenant_id`, `name`, `description`, `price`, `duration_minutes`, `is_active`, `created_at`, `updated_at`, `deleted_at`. Definir chave estrangeira para `tenants.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 017.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `services` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 019: Criar migration 009_tables_products.sql
*   **Objetivo:** Criar a tabela `products`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `products` com `id`, `tenant_id`, `name`, `description`, `sku`, `price`, `cost`, `stock_quantity`, `min_stock_quantity`, `is_active`, `created_at`, `updated_at`, `deleted_at`. Definir chave estrangeira para `tenants.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 018.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `products` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 020: Criar migration 010_tables_work_orders.sql
*   **Objetivo:** Criar a tabela `work_orders`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `work_orders` com `id`, `tenant_id`, `customer_id`, `vehicle_id`, `work_order_number`, `status`, `reported_problem`, `diagnosis`, `total_services_price`, `total_products_price`, `total_price`, `start_date`, `end_date`, `created_at`, `updated_at`, `deleted_at`. Definir chaves estrangeiras para `tenants.id`, `customers.id` e `vehicles.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 019.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `work_orders` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 021: Criar migration 011_tables_work_order_services.sql
*   **Objetivo:** Criar a tabela `work_order_services`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `work_order_services` com `id`, `tenant_id`, `work_order_id`, `service_id`, `quantity`, `price`, `created_at`, `updated_at`. Definir chaves estrangeiras para `tenants.id`, `work_orders.id` e `services.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 020.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `work_order_services` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 022: Criar migration 012_tables_work_order_products.sql
*   **Objetivo:** Criar a tabela `work_order_products`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `work_order_products` com `id`, `tenant_id`, `work_order_id`, `product_id`, `quantity`, `price`, `created_at`, `updated_at`. Definir chaves estrangeiras para `tenants.id`, `work_orders.id` e `products.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 021.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `work_order_products` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 023: Criar migration 013_tables_work_order_history.sql
*   **Objetivo:** Criar a tabela `work_order_history`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `work_order_history` com `id`, `tenant_id`, `work_order_id`, `user_id`, `action`, `old_status`, `new_status`, `details`, `created_at`. Definir chaves estrangeiras para `tenants.id`, `work_orders.id` e `profiles.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 022.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `work_order_history` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 024: Criar migration 014_tables_accounts_receivable.sql
*   **Objetivo:** Criar a tabela `accounts_receivable`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `accounts_receivable` com `id`, `tenant_id`, `work_order_id`, `customer_id`, `description`, `amount`, `due_date`, `paid_at`, `status`, `created_at`, `updated_at`, `deleted_at`. Definir chaves estrangeiras para `tenants.id`, `work_orders.id` e `customers.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 023.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `accounts_receivable` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 025: Criar migration 015_tables_accounts_payable.sql
*   **Objetivo:** Criar a tabela `accounts_payable`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `accounts_payable` com `id`, `tenant_id`, `description`, `amount`, `due_date`, `paid_at`, `status`, `created_at`, `updated_at`, `deleted_at`. Definir chave estrangeira para `tenants.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 024.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `accounts_payable` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 026: Criar migration 016_tables_financial_transactions.sql
*   **Objetivo:** Criar a tabela `financial_transactions`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `financial_transactions` com `id`, `tenant_id`, `work_order_id`, `account_receivable_id`, `account_payable_id`, `type`, `amount`, `payment_method`, `transaction_date`, `description`, `created_at`, `updated_at`. Definir chaves estrangeiras para `tenants.id`, `work_orders.id`, `accounts_receivable.id` e `accounts_payable.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 025.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `financial_transactions` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 027: Criar migration 017_tables_subscriptions.sql
*   **Objetivo:** Criar a tabela `subscriptions`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `subscriptions` com `id`, `tenant_id`, `stripe_customer_id`, `stripe_subscription_id`, `plan_id`, `status`, `current_period_start`, `current_period_end`, `cancel_at_period_end`, `created_at`, `updated_at`. Definir chave estrangeira para `tenants.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 026.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `subscriptions` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 028: Criar migration 018_tables_invoices.sql
*   **Objetivo:** Criar a tabela `invoices`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `invoices` com `id`, `tenant_id`, `subscription_id`, `stripe_invoice_id`, `amount`, `currency`, `status`, `invoice_pdf`, `created_at`, `updated_at`. Definir chaves estrangeiras para `tenants.id` e `subscriptions.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 027.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `invoices` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 029: Criar migration 019_tables_audit_logs.sql
*   **Objetivo:** Criar a tabela `audit_logs`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `audit_logs` com `id`, `tenant_id`, `user_id`, `action`, `entity_type`, `entity_id`, `old_data`, `new_data`, `ip_address`, `user_agent`, `created_at`. Definir chaves estrangeiras para `tenants.id` e `profiles.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 028.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `audit_logs` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 030: Criar migration 020_tables_notifications.sql
*   **Objetivo:** Criar a tabela `notifications`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `notifications` com `id`, `tenant_id`, `user_id`, `type`, `title`, `message`, `link`, `status`, `created_at`, `updated_at`. Definir chaves estrangeiras para `tenants.id` e `profiles.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 029.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `notifications` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 031: Criar migration 021_tables_settings.sql
*   **Objetivo:** Criar a tabela `settings`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL `CREATE TABLE` para `settings` com `id`, `tenant_id`, `key`, `value`, `type`, `created_at`, `updated_at`. Definir chave estrangeira para `tenants.id`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 030.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tabela `settings` criada com as colunas e tipos corretos.
*   **Como testar:** Executar `supabase migration up` e verificar a estrutura da tabela no Studio local.

### Task 032: Criar migration 022_triggers.sql (set_updated_at)
*   **Objetivo:** Aplicar o trigger `set_updated_at()` a todas as tabelas relevantes.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `CREATE TRIGGER set_updated_at_<table> BEFORE UPDATE ON <table> FOR EACH ROW EXECUTE FUNCTION set_updated_at();` para todas as tabelas que possuem a coluna `updated_at`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 031.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Triggers `set_updated_at` criados para todas as tabelas com `updated_at`.
*   **Como testar:** Executar `supabase migration up`. Inserir e atualizar um registro em uma tabela e verificar se `updated_at` é atualizado automaticamente.

### Task 033: Criar migration 023_functions.sql (is_active_tenant_member, has_tenant_role)
*   **Objetivo:** Criar funções SQL para auxiliar nas políticas RLS e lógica de negócio.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para as funções `is_active_tenant_member(p_tenant_id uuid, p_user_id uuid)` e `has_tenant_role(p_tenant_id uuid, p_user_id uuid, p_role tenant_role)` conforme `DATABASE.md`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 032.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Funções SQL criadas e funcionando corretamente.
*   **Como testar:** Executar `supabase migration up`. Testar as funções diretamente no SQL Editor do Supabase com dados de teste.

### Task 034: Criar migration 024_functions_generate_work_order_number.sql
*   **Objetivo:** Criar a função SQL para gerar números de OS sequenciais por tenant.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para a função `generate_work_order_number(p_tenant_id uuid)` conforme `DATABASE.md`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 033.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Função `generate_work_order_number` criada e gerando números sequenciais corretamente para diferentes tenants.
*   **Como testar:** Executar `supabase migration up`. Testar a função no SQL Editor do Supabase.

### Task 035: Criar migration 025_functions_create_audit_log.sql
*   **Objetivo:** Criar a função SQL para registrar logs de auditoria.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para a função `create_audit_log(p_tenant_id uuid, p_user_id uuid, p_action text, p_entity_type text, p_entity_id uuid, p_old_data jsonb, p_new_data jsonb)` conforme `DATABASE.md`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 034.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Função `create_audit_log` criada e funcionando corretamente.
*   **Como testar:** Executar `supabase migration up`. Testar a função no SQL Editor do Supabase.

### Task 036: Criar migration 026_rls_tenants.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `tenants`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE tenants ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `tenants` utilizando as funções auxiliares de `tenant_users` e `auth.uid()` conforme `DATABASE.md`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 035.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `tenants`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Apenas usuários autenticados e membros de um tenant podem ver/modificar seus próprios dados de tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `tenants` com diferentes usuários e roles via cliente Supabase.

### Task 037: Criar migration 027_rls_profiles.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `profiles`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `profiles`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 036.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `profiles`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Usuários podem ver/editar seu próprio perfil.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `profiles` com diferentes usuários.

### Task 038: Criar migration 028_rls_tenant_users.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `tenant_users`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE tenant_users ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `tenant_users`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 037.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `tenant_users`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Apenas membros de um tenant podem ver/gerenciar outros membros do mesmo tenant, respeitando roles.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `tenant_users` com diferentes usuários e roles.

### Task 039: Criar migration 029_rls_customers.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `customers`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE customers ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `customers`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 038.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `customers`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Clientes são isolados por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `customers` com diferentes tenants.

### Task 040: Criar migration 030_rls_customer_addresses.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `customer_addresses`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE customer_addresses ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `customer_addresses`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 039.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `customer_addresses`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Endereços de clientes são isolados por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `customer_addresses` com diferentes tenants.

### Task 041: Criar migration 031_rls_vehicles.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `vehicles`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE vehicles ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `vehicles`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 040.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `vehicles`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Veículos são isolados por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `vehicles` com diferentes tenants.

### Task 042: Criar migration 032_rls_services.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `services`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE services ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `services`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 041.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `services`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Serviços são isolados por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `services` com diferentes tenants.

### Task 043: Criar migration 033_rls_products.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `products`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE products ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `products`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 042.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `products`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Produtos são isolados por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `products` com diferentes tenants.

### Task 044: Criar migration 034_rls_work_orders.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `work_orders`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE work_orders ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `work_orders`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 043.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `work_orders`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Ordens de serviço são isoladas por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `work_orders` com diferentes tenants.

### Task 045: Criar migration 035_rls_work_order_services.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `work_order_services`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE work_order_services ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `work_order_services`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 044.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `work_order_services`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Serviços de OS são isolados por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `work_order_services` com diferentes tenants.

### Task 046: Criar migration 036_rls_work_order_products.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `work_order_products`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE work_order_products ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `work_order_products`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 045.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `work_order_products`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Produtos de OS são isolados por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `work_order_products` com diferentes tenants.

### Task 047: Criar migration 037_rls_work_order_history.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `work_order_history`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE work_order_history ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `work_order_history`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 046.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `work_order_history`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Histórico de OS é isolado por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `work_order_history` com diferentes tenants.

### Task 048: Criar migration 038_rls_accounts_receivable.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `accounts_receivable`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE accounts_receivable ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `accounts_receivable`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 047.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `accounts_receivable`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Contas a receber são isoladas por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `accounts_receivable` com diferentes tenants.

### Task 049: Criar migration 039_rls_accounts_payable.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `accounts_payable`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE accounts_payable ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `accounts_payable`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 048.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `accounts_payable`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Contas a pagar são isoladas por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `accounts_payable` com diferentes tenants.

### Task 050: Criar migration 040_rls_financial_transactions.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `financial_transactions`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE financial_transactions ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `financial_transactions`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 049.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `financial_transactions`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Transações financeiras são isoladas por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `financial_transactions` com diferentes tenants.

### Task 051: Criar migration 041_rls_subscriptions.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `subscriptions`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE subscriptions ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `subscriptions`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 050.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `subscriptions`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Assinaturas são isoladas por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `subscriptions` com diferentes tenants.

### Task 052: Criar migration 042_rls_invoices.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `invoices`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `invoices`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 051.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `invoices`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Faturas são isoladas por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `invoices` com diferentes tenants.

### Task 053: Criar migration 043_rls_audit_logs.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `audit_logs`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE audit_logs ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `audit_logs`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 052.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `audit_logs`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Logs de auditoria são isolados por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `audit_logs` com diferentes tenants.

### Task 054: Criar migration 044_rls_notifications.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `notifications`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `notifications`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 053.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `notifications`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Notificações são isoladas por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `notifications` com diferentes tenants.

### Task 055: Criar migration 045_rls_settings.sql
*   **Objetivo:** Habilitar RLS e definir políticas para a tabela `settings`.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `ALTER TABLE settings ENABLE ROW LEVEL SECURITY;` e as políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` para `settings`.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 054.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** RLS para `settings`.
*   **Critérios de aceite:** RLS habilitado e políticas aplicadas. Configurações são isoladas por tenant.
*   **Como testar:** Executar `supabase migration up`. Testar o acesso à tabela `settings` com diferentes tenants.

### Task 056: Criar migration 046_indexes.sql
*   **Objetivo:** Criar índices compostos com `tenant_id` para otimização de performance.
*   **Descrição:** Criar um novo arquivo de migration e adicionar o SQL para `CREATE INDEX` para todas as tabelas multi-tenant que se beneficiam de índices compostos com `tenant_id` e outras colunas frequentemente usadas em `WHERE` ou `ORDER BY` (ex: `customers (tenant_id, name)`, `vehicles (tenant_id, plate)`).
*   **Arquivos envolvidos:** `supabase/migrations/*.sql`.
*   **Dependências:** Task 055.
*   **Banco de dados envolvido:** PostgreSQL.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Índices criados com sucesso no banco de dados local.
*   **Como testar:** Executar `supabase migration up`. Verificar a existência dos índices no Studio local.

### Task 057: Gerar tipos TypeScript do Supabase
*   **Objetivo:** Gerar automaticamente os tipos TypeScript para o esquema do banco de dados.
*   **Descrição:** Executar `supabase gen types typescript --project-id 

### Task 058: Configurar cliente Supabase no frontend
*   **Objetivo:** Inicializar o cliente Supabase no frontend para interagir com o banco de dados e autenticação.
*   **Descrição:** Criar `lib/supabase.ts` para configurar o cliente Supabase usando `createClient` do `@supabase/supabase-js`. Exportar instâncias para uso em Server Actions e componentes de cliente.
*   **Arquivos envolvidos:** `lib/supabase.ts`, `app/layout.tsx` (ou similar para provedor de contexto).
*   **Dependências:** Task 007, Task 008, Task 057.
*   **Banco de dados envolvido:** Supabase.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Cliente Supabase configurado e acessível no frontend.
*   **Como testar:** Adicionar um `console.log` do cliente Supabase em um componente de teste.

### Task 059: Criar provedor de autenticação (AuthProvider)
*   **Objetivo:** Gerenciar o estado de autenticação do usuário em toda a aplicação.
*   **Descrição:** Criar um `AuthProvider` (ou usar o contexto de autenticação do Supabase) para envolver a aplicação e fornecer o estado do usuário (`user`, `session`, `tenant_id`, `role`) para os componentes filhos. Utilizar `supabase.auth.onAuthStateChange`.
*   **Arquivos envolvidos:** `components/providers/AuthProvider.tsx`, `app/layout.tsx`, `hooks/useAuth.ts`.
*   **Dependências:** Task 058.
*   **Banco de dados envolvido:** Supabase Auth.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Estado de autenticação disponível globalmente na aplicação.
*   **Como testar:** Criar um componente simples que exiba o `user.email` se autenticado.

---

## Fase 3: Autenticação

**Objetivo:** Implementar os fluxos completos de login, cadastro, recuperação e alteração de senha.

### Task 060: Implementar página de Login (UI)
*   **Objetivo:** Criar a interface da página de login conforme `UI_UX.md`.
*   **Descrição:** Desenvolver a página `app/(auth)/login/page.tsx` com os campos de e-mail e senha, botões e links para recuperação de senha e cadastro. Utilizar `shadcn/ui` `Card`, `Input`, `Button`, `FormField`.
*   **Arquivos envolvidos:** `app/(auth)/login/page.tsx`, `components/ui/card.tsx`, `components/ui/input.tsx`, `components/ui/button.tsx`, `components/common/FormField.tsx`.
*   **Dependências:** Task 004, Task 006.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Card`, `Input`, `Button`, `FormField`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Página de login renderizada corretamente com todos os elementos visuais.
*   **Como testar:** Acessar `/login` no navegador.

### Task 061: Implementar validação de formulário de Login (Frontend)
*   **Objetivo:** Adicionar validação de e-mail e senha no formulário de login.
*   **Descrição:** Utilizar Zod para definir o schema de validação para e-mail (formato válido, obrigatório) e senha (obrigatório). Integrar com `react-hook-form` e `FormField` para exibir mensagens de erro.
*   **Arquivos envolvidos:** `validations/auth.ts`, `app/(auth)/login/page.tsx`.
*   **Dependências:** Task 060.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `FormField`.
*   **Regras de negócio:** Validação de e-mail e senha.
*   **Critérios de aceite:** Mensagens de erro exibidas para campos vazios ou e-mail inválido antes da submissão.
*   **Como testar:** Tentar submeter o formulário de login com campos vazios ou e-mail inválido.

### Task 062: Criar Server Action para Login
*   **Objetivo:** Implementar a lógica de autenticação no backend usando Server Actions.
*   **Descrição:** Criar `server-actions/auth.ts` (ou `app/(auth)/login/actions.ts`) com uma função `signIn` que recebe e-mail e senha. Utilizar `AuthService.signInWithPassword` e redirecionar para `/dashboard` em caso de sucesso. Tratar erros e retornar mensagens apropriadas.
*   **Arquivos envolvidos:** `server-actions/auth.ts`, `services/auth.ts`, `app/(auth)/login/page.tsx`.
*   **Dependências:** Task 058, Task 061.
*   **Banco de dados envolvido:** Supabase Auth.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Autenticação de usuário.
*   **Critérios de aceite:** Usuário consegue fazer login com credenciais válidas e é redirecionado. Erros de autenticação são tratados.
*   **Como testar:** Testar o login com credenciais válidas e inválidas.

### Task 063: Integrar formulário de Login com Server Action
*   **Objetivo:** Conectar o formulário de login do frontend com a Server Action de login.
*   **Descrição:** No `app/(auth)/login/page.tsx`, chamar a Server Action `signIn` no `onSubmit` do formulário. Gerenciar estados de loading e erro da submissão.
*   **Arquivos envolvidos:** `app/(auth)/login/page.tsx`.
*   **Dependências:** Task 062.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Login funcional do frontend ao backend.
*   **Como testar:** Realizar login completo no navegador.

### Task 064: Implementar página de Cadastro (UI)
*   **Objetivo:** Criar a interface da página de cadastro conforme `UI_UX.md`.
*   **Descrição:** Desenvolver a página `app/(auth)/register/page.tsx` com campos para nome completo, e-mail, senha, confirmar senha e nome da oficina. Utilizar `shadcn/ui` `Card`, `Input`, `Button`, `FormField`.
*   **Arquivos envolvidos:** `app/(auth)/register/page.tsx`.
*   **Dependências:** Task 060.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Card`, `Input`, `Button`, `FormField`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Página de cadastro renderizada corretamente com todos os elementos visuais.
*   **Como testar:** Acessar `/register` no navegador.

### Task 065: Implementar validação de formulário de Cadastro (Frontend)
*   **Objetivo:** Adicionar validação de todos os campos do formulário de cadastro.
*   **Descrição:** Utilizar Zod para definir o schema de validação para nome completo (obrigatório), e-mail (formato válido, obrigatório), senha (mínimo 6 caracteres, obrigatório) e confirmar senha (igual à senha), nome da oficina (obrigatório). Integrar com `react-hook-form` e `FormField`.
*   **Arquivos envolvidos:** `validations/auth.ts`, `app/(auth)/register/page.tsx`.
*   **Dependências:** Task 064.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `FormField`.
*   **Regras de negócio:** Validação de dados de cadastro.
*   **Critérios de aceite:** Mensagens de erro exibidas para campos inválidos ou vazios antes da submissão.
*   **Como testar:** Tentar submeter o formulário de cadastro com dados inválidos.

### Task 066: Criar Server Action para Cadastro
*   **Objetivo:** Implementar a lógica de cadastro de usuário e criação de tenant no backend.
*   **Descrição:** Criar `server-actions/auth.ts` (ou `app/(auth)/register/actions.ts`) com uma função `signUp` que recebe os dados do formulário. Utilizar `AuthService.signUp` para criar o usuário no Supabase Auth. Em seguida, chamar `TenantService.createTenantWithOwner` para criar o tenant e associar o usuário como owner. Redirecionar para `/dashboard` em caso de sucesso. Tratar erros.
*   **Arquivos envolvidos:** `server-actions/auth.ts`, `services/auth.ts`, `services/tenant.ts`, `repositories/tenantRepository.ts`.
*   **Dependências:** Task 058, Task 065.
*   **Banco de dados envolvido:** Supabase Auth, `tenants`, `profiles`, `tenant_users`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Criação de usuário, criação de tenant, associação de owner.
*   **Critérios de aceite:** Novo usuário e tenant criados no Supabase. Usuário redirecionado.
*   **Como testar:** Testar o cadastro com dados válidos e inválidos. Verificar o banco de dados.

### Task 067: Integrar formulário de Cadastro com Server Action
*   **Objetivo:** Conectar o formulário de cadastro do frontend com a Server Action de cadastro.
*   **Descrição:** No `app/(auth)/register/page.tsx`, chamar a Server Action `signUp` no `onSubmit` do formulário. Gerenciar estados de loading e erro da submissão.
*   **Arquivos envolvidos:** `app/(auth)/register/page.tsx`.
*   **Dependências:** Task 066.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Cadastro funcional do frontend ao backend.
*   **Como testar:** Realizar cadastro completo no navegador.

### Task 068: Implementar página de Recuperação de Senha (UI)
*   **Objetivo:** Criar a interface da página de recuperação de senha conforme `UI_UX.md`.
*   **Descrição:** Desenvolver a página `app/(auth)/forgot-password/page.tsx` com campo para e-mail e botão de envio. Utilizar `shadcn/ui` `Card`, `Input`, `Button`, `FormField`.
*   **Arquivos envolvidos:** `app/(auth)/forgot-password/page.tsx`.
*   **Dependências:** Task 060.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Card`, `Input`, `Button`, `FormField`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Página de recuperação de senha renderizada corretamente.
*   **Como testar:** Acessar `/forgot-password` no navegador.

### Task 069: Implementar validação de formulário de Recuperação de Senha (Frontend)
*   **Objetivo:** Adicionar validação de e-mail no formulário de recuperação de senha.
*   **Descrição:** Utilizar Zod para definir o schema de validação para e-mail (formato válido, obrigatório). Integrar com `react-hook-form` e `FormField`.
*   **Arquivos envolvidos:** `validations/auth.ts`, `app/(auth)/forgot-password/page.tsx`.
*   **Dependências:** Task 068.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `FormField`.
*   **Regras de negócio:** Validação de e-mail.
*   **Critérios de aceite:** Mensagens de erro exibidas para e-mail inválido.
*   **Como testar:** Tentar submeter o formulário com e-mail inválido.

### Task 070: Criar Server Action para Recuperação de Senha
*   **Objetivo:** Implementar a lógica de envio de e-mail de recuperação de senha.
*   **Descrição:** Criar `server-actions/auth.ts` com uma função `sendPasswordResetEmail` que recebe o e-mail. Utilizar `AuthService.sendPasswordResetEmail` do Supabase. Retornar sucesso sem revelar se o e-mail existe.
*   **Arquivos envolvidos:** `server-actions/auth.ts`, `services/auth.ts`.
*   **Dependências:** Task 058, Task 069.
*   **Banco de dados envolvido:** Supabase Auth.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Envio de e-mail de recuperação.
*   **Critérios de aceite:** E-mail de recuperação enviado para e-mails válidos. Mensagem de sucesso genérica.
*   **Como testar:** Testar o envio com e-mail válido e verificar a caixa de entrada.

### Task 071: Integrar formulário de Recuperação de Senha com Server Action
*   **Objetivo:** Conectar o formulário de recuperação de senha do frontend com a Server Action.
*   **Descrição:** No `app/(auth)/forgot-password/page.tsx`, chamar a Server Action `sendPasswordResetEmail` no `onSubmit`. Gerenciar estados de loading e feedback de sucesso/erro.
*   **Arquivos envolvidos:** `app/(auth)/forgot-password/page.tsx`.
*   **Dependências:** Task 070.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Fluxo de recuperação de senha funcional.
*   **Como testar:** Realizar o fluxo completo de recuperação de senha no navegador.

### Task 072: Implementar página de Alteração de Senha (UI)
*   **Objetivo:** Criar a interface da página de alteração de senha conforme `UI_UX.md`.
*   **Descrição:** Desenvolver a página `app/(auth)/update-password/page.tsx` com campos para nova senha e confirmar nova senha. Utilizar `shadcn/ui` `Card`, `Input`, `Button`, `FormField`.
*   **Arquivos envolvidos:** `app/(auth)/update-password/page.tsx`.
*   **Dependências:** Task 060.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Card`, `Input`, `Button`, `FormField`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Página de alteração de senha renderizada corretamente.
*   **Como testar:** Acessar `/update-password` (geralmente via link de e-mail).

### Task 073: Implementar validação de formulário de Alteração de Senha (Frontend)
*   **Objetivo:** Adicionar validação de nova senha e confirmação.
*   **Descrição:** Utilizar Zod para definir o schema de validação para nova senha (mínimo 6 caracteres, obrigatório) e confirmar senha (igual à nova senha). Integrar com `react-hook-form` e `FormField`.
*   **Arquivos envolvidos:** `validations/auth.ts`, `app/(auth)/update-password/page.tsx`.
*   **Dependências:** Task 072.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `FormField`.
*   **Regras de negócio:** Validação de senha.
*   **Critérios de aceite:** Mensagens de erro exibidas para senhas inválidas ou não coincidentes.
*   **Como testar:** Tentar submeter o formulário com senhas inválidas.

### Task 074: Criar Server Action para Alteração de Senha
*   **Objetivo:** Implementar a lógica de redefinição de senha no backend.
*   **Descrição:** Criar `server-actions/auth.ts` com uma função `updatePassword` que recebe a nova senha. Utilizar `AuthService.updateUser` do Supabase. Redirecionar para `/login` em caso de sucesso.
*   **Arquivos envolvidos:** `server-actions/auth.ts`, `services/auth.ts`.
*   **Dependências:** Task 058, Task 073.
*   **Banco de dados envolvido:** Supabase Auth.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Redefinição de senha.
*   **Critérios de aceite:** Senha do usuário atualizada com sucesso.
*   **Como testar:** Testar a alteração de senha após receber o link de recuperação.

### Task 075: Integrar formulário de Alteração de Senha com Server Action
*   **Objetivo:** Conectar o formulário de alteração de senha do frontend com a Server Action.
*   **Descrição:** No `app/(auth)/update-password/page.tsx`, chamar a Server Action `updatePassword` no `onSubmit`. Gerenciar estados de loading e feedback de sucesso/erro.
*   **Arquivos envolvidos:** `app/(auth)/update-password/page.tsx`.
*   **Dependências:** Task 074.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Fluxo de alteração de senha funcional.
*   **Como testar:** Realizar o fluxo completo de alteração de senha no navegador.

### Task 076: Implementar Logout (Server Action e Frontend)
*   **Objetivo:** Permitir que o usuário encerre sua sessão.
*   **Descrição:** Criar uma Server Action `signOut` em `server-actions/auth.ts` que chama `AuthService.signOut`. No frontend, adicionar um botão de logout (ex: no `UserMenu`) que invoca essa Server Action e redireciona para `/login`.
*   **Arquivos envolvidos:** `server-actions/auth.ts`, `services/auth.ts`, `components/common/UserMenu.tsx`.
*   **Dependências:** Task 058.
*   **Banco de dados envolvido:** Supabase Auth.
*   **Componentes envolvidos:** `UserMenu`, `Button`.
*   **Regras de negócio:** Encerramento de sessão.
*   **Critérios de aceite:** Usuário consegue fazer logout e é redirecionado para a página de login.
*   **Como testar:** Fazer login, depois logout.

### Task 077: Configurar redirecionamento de rotas protegidas
*   **Objetivo:** Proteger rotas da aplicação que exigem autenticação.
*   **Descrição:** Implementar um middleware ou lógica no `app/layout.tsx` para verificar se o usuário está autenticado. Se não estiver, redirecionar para `/login`. Se estiver, garantir que o `tenant_id` e `role` estejam disponíveis no contexto.
*   **Arquivos envolvidos:** `middleware.ts` (opcional), `app/layout.tsx`, `components/providers/AuthProvider.tsx`.
*   **Dependências:** Task 059.
*   **Banco de dados envolvido:** Supabase Auth.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Proteção de rotas.
*   **Critérios de aceite:** Usuários não autenticados são redirecionados de rotas protegidas. Usuários autenticados acessam rotas protegidas.
*   **Como testar:** Tentar acessar `/dashboard` sem estar logado.

---

## Fase 4: Multiempresa Core

**Objetivo:** Configurar a lógica central para o modelo multiempresa, incluindo tenants, perfis e permissões.

### Task 078: Criar `TenantService` e `TenantRepository`
*   **Objetivo:** Encapsular a lógica de negócio e acesso a dados para tenants.
*   **Descrição:** Criar `services/tenant.ts` e `repositories/tenantRepository.ts`. O `TenantRepository` deve ter métodos para `createTenant`, `getTenantById`, `updateTenant`, `addTenantUser`, `getTenantUsers`. O `TenantService` deve orquestrar essas operações e incluir lógica de negócio (ex: `createTenantWithOwner`).
*   **Arquivos envolvidos:** `services/tenant.ts`, `repositories/tenantRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 013, Task 014, Task 058.
*   **Banco de dados envolvido:** `tenants`, `tenant_users`, `profiles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Gestão de tenants e usuários de tenant.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD de tenants e `tenant_users`.
*   **Como testar:** Testar os métodos diretamente no backend (via testes unitários ou console).

### Task 079: Implementar `useTenant` Hook
*   **Objetivo:** Fornecer o contexto do tenant atual para os componentes do frontend.
*   **Descrição:** Criar `hooks/useTenant.ts` que utiliza o `AuthProvider` (ou contexto similar) para expor o `tenant_id` e a `tenant_role` do usuário logado. Isso será crucial para todas as operações multiempresa no frontend.
*   **Arquivos envolvidos:** `hooks/useTenant.ts`, `components/providers/AuthProvider.tsx`.
*   **Dependências:** Task 059, Task 078.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** `useTenant` retorna o `tenant_id` e `tenant_role` corretos para o usuário logado.
*   **Como testar:** Usar o hook em um componente e exibir os valores.

### Task 080: Configurar `tenant_id` no contexto de RLS (Supabase Auth)
*   **Objetivo:** Garantir que o `tenant_id` do usuário esteja disponível para as políticas RLS.
*   **Descrição:** Configurar o Supabase Auth para incluir o `tenant_id` no JWT do usuário. Isso geralmente envolve uma função de banco de dados que busca o `tenant_id` da tabela `tenant_users` e o adiciona ao `app_metadata` do usuário ou ao `claims` do JWT.
*   **Arquivos envolvidos:** `supabase/migrations/*.sql` (para a função de claims), Supabase Dashboard (configuração de JWT claims).
*   **Dependências:** Task 038.
*   **Banco de dados envolvido:** Supabase Auth, `tenant_users`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Isolamento de dados via RLS.
*   **Critérios de aceite:** O JWT do usuário autenticado contém o `tenant_id` no `auth.jwt()`.
*   **Como testar:** Inspecionar o JWT retornado após o login.

### Task 081: Criar Server Action para obter dados do Tenant
*   **Objetivo:** Permitir que o frontend obtenha as informações do tenant atual.
*   **Descrição:** Criar uma Server Action em `server-actions/tenant.ts` que chama `TenantService.getTenantById` usando o `tenant_id` do contexto do usuário. Esta ação será usada para exibir informações da oficina no frontend.
*   **Arquivos envolvidos:** `server-actions/tenant.ts`, `services/tenant.ts`.
*   **Dependências:** Task 078, Task 079.
*   **Banco de dados envolvido:** `tenants`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Server Action retorna os dados corretos do tenant.
*   **Como testar:** Chamar a Server Action de um componente e exibir os dados.

### Task 082: Implementar página de Configurações da Empresa (UI)
*   **Objetivo:** Permitir que o owner/admin visualize e edite as informações da oficina.
*   **Descrição:** Criar a página `app/(app)/settings/company/page.tsx` com campos para `legal_name`, `trade_name`, `document`, `email`, `phone`, `whatsapp`, `logo_url`. Utilizar `shadcn/ui` `Input`, `Button`, `FormField`.
*   **Arquivos envolvidos:** `app/(app)/settings/company/page.tsx`.
*   **Dependências:** Task 060 (componentes UI).
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Input`, `Button`, `FormField`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Página de configurações da empresa renderizada corretamente.
*   **Como testar:** Acessar `/settings/company` no navegador.

### Task 083: Implementar validação de formulário de Configurações da Empresa (Frontend)
*   **Objetivo:** Adicionar validação aos campos do formulário de configurações da empresa.
*   **Descrição:** Utilizar Zod para definir o schema de validação para os campos do tenant. Integrar com `react-hook-form` e `FormField`.
*   **Arquivos envolvidos:** `validations/tenant.ts`, `app/(app)/settings/company/page.tsx`.
*   **Dependências:** Task 082.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `FormField`.
*   **Regras de negócio:** Validação de dados do tenant.
*   **Critérios de aceite:** Mensagens de erro exibidas para campos inválidos.
*   **Como testar:** Tentar submeter o formulário com dados inválidos.

### Task 084: Criar Server Action para atualizar dados do Tenant
*   **Objetivo:** Permitir que o owner/admin atualize as informações da oficina.
*   **Descrição:** Criar uma Server Action em `server-actions/tenant.ts` que chama `TenantService.updateTenant`. A ação deve verificar a permissão do usuário (`owner` ou `admin`) e o `tenant_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/tenant.ts`, `services/tenant.ts`.
*   **Dependências:** Task 078, Task 079, Task 083.
*   **Banco de dados envolvido:** `tenants`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Atualização de dados do tenant, controle de acesso.
*   **Critérios de aceite:** Dados do tenant atualizados com sucesso apenas por usuários autorizados.
*   **Como testar:** Testar a atualização com diferentes perfis de usuário.

### Task 085: Integrar formulário de Configurações da Empresa com Server Action
*   **Objetivo:** Conectar o formulário de configurações da empresa com a Server Action de atualização.
*   **Descrição:** No `app/(app)/settings/company/page.tsx`, chamar a Server Action `updateTenant` no `onSubmit`. Gerenciar estados de loading e feedback de sucesso/erro.
*   **Arquivos envolvidos:** `app/(app)/settings/company/page.tsx`.
*   **Dependências:** Task 084.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Atualização de dados da empresa funcional.
*   **Como testar:** Realizar a atualização de dados da empresa no navegador.

### Task 086: Implementar upload de logo da oficina (Frontend UI)
*   **Objetivo:** Permitir que o owner/admin faça upload do logotipo da oficina.
*   **Descrição:** Adicionar um componente de upload de arquivo na página de configurações da empresa. Utilizar `shadcn/ui` `Input` (type file) ou um componente customizado de drag-and-drop. Exibir preview da imagem.
*   **Arquivos envolvidos:** `app/(app)/settings/company/page.tsx`, `components/common/FileUpload.tsx` (opcional).
*   **Dependências:** Task 082.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Input`, `Image`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Componente de upload de logo renderizado e funcional para seleção de arquivo.
*   **Como testar:** Selecionar um arquivo de imagem para upload.

### Task 087: Criar Server Action para upload de logo
*   **Objetivo:** Implementar a lógica de upload de arquivo para o Supabase Storage.
*   **Descrição:** Criar uma Server Action em `server-actions/tenant.ts` (ou `server-actions/storage.ts`) que recebe o arquivo. Utilizar `SupabaseStorageService.uploadFile` para fazer o upload para o bucket `logos` no caminho `tenant/{tenant_id}/logo.png`. Atualizar `logo_url` na tabela `tenants`.
*   **Arquivos envolvidos:** `server-actions/tenant.ts`, `services/storage.ts`, `repositories/tenantRepository.ts`.
*   **Dependências:** Task 078, Task 079.
*   **Banco de dados envolvido:** Supabase Storage, `tenants`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Upload de arquivo, atualização de URL.
*   **Critérios de aceite:** Arquivo de logo enviado para o Supabase Storage e URL atualizada no banco de dados.
*   **Como testar:** Testar o upload de logo e verificar o Storage e o banco de dados.

### Task 088: Integrar upload de logo com Server Action
*   **Objetivo:** Conectar o componente de upload de logo do frontend com a Server Action.
*   **Descrição:** No `app/(app)/settings/company/page.tsx`, chamar a Server Action de upload no evento de seleção de arquivo. Gerenciar estados de loading e feedback.
*   **Arquivos envolvidos:** `app/(app)/settings/company/page.tsx`.
*   **Dependências:** Task 087.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Upload de logo funcional do frontend ao backend.
*   **Como testar:** Realizar o upload de logo no navegador.

### Task 089: Implementar página de Gerenciamento de Usuários (UI)
*   **Objetivo:** Permitir que o owner/admin visualize, convide e gerencie usuários da oficina.
*   **Descrição:** Criar a página `app/(app)/settings/users/page.tsx` com uma `DataTable` para listar `tenant_users`. Incluir botões para convidar novo usuário e editar permissões.
*   **Arquivos envolvidos:** `app/(app)/settings/users/page.tsx`, `components/common/DataTable.tsx`, `components/ui/button.tsx`.
*   **Dependências:** Task 060 (componentes UI), Task 079.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `DataTable`, `Button`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Página de gerenciamento de usuários renderizada com a lista de usuários.
*   **Como testar:** Acessar `/settings/users` no navegador.

### Task 090: Criar Server Action para listar usuários do Tenant
*   **Objetivo:** Obter a lista de usuários associados ao tenant atual.
*   **Descrição:** Criar uma Server Action em `server-actions/tenant.ts` que chama `TenantService.getTenantUsers` usando o `tenant_id` do contexto. A ação deve retornar os `tenant_users` com suas `profiles` associadas.
*   **Arquivos envolvidos:** `server-actions/tenant.ts`, `services/tenant.ts`, `repositories/tenantRepository.ts`.
*   **Dependências:** Task 078, Task 079.
*   **Banco de dados envolvido:** `tenant_users`, `profiles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Listagem de usuários por tenant.
*   **Critérios de aceite:** Server Action retorna a lista correta de usuários do tenant.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 091: Integrar listagem de usuários com `DataTable`
*   **Objetivo:** Exibir a lista de usuários do tenant na `DataTable`.
*   **Descrição:** No `app/(app)/settings/users/page.tsx`, carregar os dados dos usuários usando a Server Action e passá-los para o componente `DataTable`. Configurar as colunas da tabela para exibir nome, e-mail, role e status.
*   **Arquivos envolvidos:** `app/(app)/settings/users/page.tsx`.
*   **Dependências:** Task 089, Task 090.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `DataTable`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Lista de usuários exibida corretamente na tabela.
*   **Como testar:** Acessar a página e verificar a tabela.

### Task 092: Implementar modal de Convite de Usuário (UI)
*   **Objetivo:** Criar a interface para convidar um novo usuário para a oficina.
*   **Descrição:** Criar um `Modal` ou `Drawer` com um formulário para e-mail e seleção de `role` (admin, employee). Utilizar `shadcn/ui` `Dialog` ou `Drawer`, `Input`, `Select`, `Button`, `FormField`.
*   **Arquivos envolvidos:** `components/modules/tenant/InviteUserModal.tsx`, `app/(app)/settings/users/page.tsx`.
*   **Dependências:** Task 060 (componentes UI).
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Dialog`/`Drawer`, `Input`, `Select`, `Button`, `FormField`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Modal de convite renderizado e funcional.
*   **Como testar:** Abrir o modal na página de gerenciamento de usuários.

### Task 093: Implementar validação de formulário de Convite de Usuário (Frontend)
*   **Objetivo:** Adicionar validação aos campos do formulário de convite.
*   **Descrição:** Utilizar Zod para definir o schema de validação para e-mail (formato válido, obrigatório) e role (obrigatório, um dos `tenant_role` enums). Integrar com `react-hook-form` e `FormField`.
*   **Arquivos envolvidos:** `validations/tenant.ts`, `components/modules/tenant/InviteUserModal.tsx`.
*   **Dependências:** Task 092.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `FormField`.
*   **Regras de negócio:** Validação de convite.
*   **Critérios de aceite:** Mensagens de erro exibidas para campos inválidos.
*   **Como testar:** Tentar submeter o formulário com dados inválidos.

### Task 094: Criar Server Action para convidar usuário
*   **Objetivo:** Implementar a lógica de convite de usuário no backend.
*   **Descrição:** Criar uma Server Action em `server-actions/tenant.ts` que recebe e-mail e role. Utilizar `AuthService.inviteUserByEmail` do Supabase para enviar o convite. Em seguida, criar um registro em `tenant_users` com `status=\'invited\'` e `invited_by` o `user_id` do usuário logado. A ação deve verificar a permissão do usuário (`owner` ou `admin`).
*   **Arquivos envolvidos:** `server-actions/tenant.ts`, `services/auth.ts`, `services/tenant.ts`, `repositories/tenantRepository.ts`.
*   **Dependências:** Task 078, Task 079, Task 093.
*   **Banco de dados envolvido:** Supabase Auth, `tenant_users`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Convite de usuário, controle de acesso.
*   **Critérios de aceite:** E-mail de convite enviado. Registro `tenant_users` criado com status `invited`.
*   **Como testar:** Testar o convite com diferentes perfis de usuário. Verificar o e-mail e o banco de dados.

### Task 095: Integrar modal de Convite de Usuário com Server Action
*   **Objetivo:** Conectar o formulário de convite do frontend com a Server Action.
*   **Descrição:** No `components/modules/tenant/InviteUserModal.tsx`, chamar a Server Action de convite no `onSubmit`. Gerenciar estados de loading e feedback de sucesso/erro. Fechar o modal após sucesso.
*   **Arquivos envolvidos:** `components/modules/tenant/InviteUserModal.tsx`.
*   **Dependências:** Task 094.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Fluxo de convite de usuário funcional.
*   **Como testar:** Realizar o convite de usuário no navegador.

### Task 096: Implementar modal de Edição de Permissões de Usuário (UI)
*   **Objetivo:** Criar a interface para editar a `role` de um usuário existente na oficina.
*   **Descrição:** Criar um `Modal` ou `Drawer` com um `Select` para a `role` do usuário. Utilizar `shadcn/ui` `Dialog` ou `Drawer`, `Select`, `Button`, `FormField`.
*   **Arquivos envolvidos:** `components/modules/tenant/EditUserRoleModal.tsx`, `app/(app)/settings/users/page.tsx`.
*   **Dependências:** Task 060 (componentes UI).
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Dialog`/`Drawer`, `Select`, `Button`, `FormField`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Modal de edição de permissões renderizado e funcional.
*   **Como testar:** Abrir o modal para um usuário existente.

### Task 097: Implementar validação de formulário de Edição de Permissões (Frontend)
*   **Objetivo:** Adicionar validação à seleção de `role`.
*   **Descrição:** Utilizar Zod para definir o schema de validação para `role` (obrigatório, um dos `tenant_role` enums). Integrar com `react-hook-form` e `FormField`.
*   **Arquivos envolvidos:** `validations/tenant.ts`, `components/modules/tenant/EditUserRoleModal.tsx`.
*   **Dependências:** Task 096.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `FormField`.
*   **Regras de negócio:** Validação de role.
*   **Critérios de aceite:** Mensagens de erro exibidas para role inválida.
*   **Como testar:** Tentar submeter o formulário com role inválida.

### Task 098: Criar Server Action para atualizar permissões de usuário
*   **Objetivo:** Implementar a lógica de atualização da `role` de um usuário no backend.
*   **Descrição:** Criar uma Server Action em `server-actions/tenant.ts` que recebe `tenant_user_id` e a nova `role`. Utilizar `TenantService.updateTenantUserRole`. A ação deve verificar a permissão do usuário (`owner` ou `admin`) e impedir que um `owner` mude sua própria `role` ou a `role` de outro `owner`.
*   **Arquivos envolvidos:** `server-actions/tenant.ts`, `services/tenant.ts`, `repositories/tenantRepository.ts`.
*   **Dependências:** Task 078, Task 079, Task 097.
*   **Banco de dados envolvido:** `tenant_users`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Atualização de role, controle de acesso, regras de negócio para owner.
*   **Critérios de aceite:** Role do usuário atualizada com sucesso apenas por usuários autorizados e respeitando as regras de owner.
*   **Como testar:** Testar a atualização de role com diferentes perfis e cenários.

### Task 099: Integrar modal de Edição de Permissões com Server Action
*   **Objetivo:** Conectar o formulário de edição de permissões do frontend com a Server Action.
*   **Descrição:** No `components/modules/tenant/EditUserRoleModal.tsx`, chamar a Server Action de atualização de role no `onSubmit`. Gerenciar estados de loading e feedback de sucesso/erro. Fechar o modal após sucesso.
*   **Arquivos envolvidos:** `components/modules/tenant/EditUserRoleModal.tsx`.
*   **Dependências:** Task 098.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Fluxo de edição de permissões funcional.
*   **Como testar:** Realizar a edição de permissões no navegador.

### Task 100: Implementar funcionalidade de Remover Usuário (UI)
*   **Objetivo:** Permitir que o owner/admin remova um usuário da oficina.
*   **Descrição:** Adicionar um botão de "Remover" na linha de cada usuário na `DataTable` da página de gerenciamento de usuários. Integrar com um `ConfirmDialog`.
*   **Arquivos envolvidos:** `app/(app)/settings/users/page.tsx`, `components/common/DataTable.tsx`, `components/common/ConfirmDialog.tsx`.
*   **Dependências:** Task 089, Task 091.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button`, `ConfirmDialog`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Botão de remover e diálogo de confirmação renderizados e funcionais.
*   **Como testar:** Clicar no botão de remover e abrir o diálogo.

### Task 101: Criar Server Action para remover usuário
*   **Objetivo:** Implementar a lógica de remoção de usuário do tenant no backend.
*   **Descrição:** Criar uma Server Action em `server-actions/tenant.ts` que recebe `tenant_user_id`. Utilizar `TenantService.removeTenantUser` para alterar o `status` do `tenant_user` para `inactive` (soft delete). A ação deve verificar a permissão do usuário (`owner` ou `admin`) e impedir que um `owner` remova a si mesmo ou outro `owner`.
*   **Arquivos envolvidos:** `server-actions/tenant.ts`, `services/tenant.ts`, `repositories/tenantRepository.ts`.
*   **Dependências:** Task 078, Task 079, Task 100.
*   **Banco de dados envolvido:** `tenant_users`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Remoção de usuário, controle de acesso, regras de negócio para owner.
*   **Critérios de aceite:** Status do `tenant_user` alterado para `inactive` apenas por usuários autorizados e respeitando as regras de owner.
*   **Como testar:** Testar a remoção de usuário com diferentes perfis e cenários.

### Task 102: Integrar remoção de usuário com Server Action
*   **Objetivo:** Conectar o botão de remover usuário do frontend com a Server Action.
*   **Descrição:** No `app/(app)/settings/users/page.tsx` (ou no componente `DataTable`), chamar a Server Action de remoção de usuário após a confirmação no `ConfirmDialog`. Gerenciar estados de loading e feedback de sucesso/erro. Atualizar a lista de usuários após a remoção.
*   **Arquivos envolvidos:** `app/(app)/settings/users/page.tsx`.
*   **Dependências:** Task 101.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback), `ConfirmDialog`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Fluxo de remoção de usuário funcional.
*   **Como testar:** Realizar a remoção de usuário no navegador.

### Task 103: Implementar página de Perfil do Usuário (UI)
*   **Objetivo:** Permitir que o usuário visualize e edite suas informações de perfil.
*   **Descrição:** Criar a página `app/(app)/settings/profile/page.tsx` com campos para `full_name`, `email` (somente leitura), `avatar_url`. Utilizar `shadcn/ui` `Input`, `Button`, `FormField`.
*   **Arquivos envolvidos:** `app/(app)/settings/profile/page.tsx`.
*   **Dependências:** Task 060 (componentes UI).
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Input`, `Button`, `FormField`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Página de perfil renderizada corretamente.
*   **Como testar:** Acessar `/settings/profile` no navegador.

### Task 104: Implementar validação de formulário de Perfil do Usuário (Frontend)
*   **Objetivo:** Adicionar validação aos campos do formulário de perfil.
*   **Descrição:** Utilizar Zod para definir o schema de validação para `full_name` (obrigatório). Integrar com `react-hook-form` e `FormField`.
*   **Arquivos envolvidos:** `validations/profile.ts`, `app/(app)/settings/profile/page.tsx`.
*   **Dependências:** Task 103.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `FormField`.
*   **Regras de negócio:** Validação de nome completo.
*   **Critérios de aceite:** Mensagens de erro exibidas para nome inválido.
*   **Como testar:** Tentar submeter o formulário com nome vazio.

### Task 105: Criar Server Action para atualizar Perfil do Usuário
*   **Objetivo:** Permitir que o usuário atualize suas informações de perfil.
*   **Descrição:** Criar uma Server Action em `server-actions/profile.ts` que chama `ProfileService.updateProfile`. A ação deve usar o `user_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/profile.ts`, `services/profile.ts`, `repositories/profileRepository.ts`.
*   **Dependências:** Task 079, Task 104.
*   **Banco de dados envolvido:** `profiles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Atualização de perfil.
*   **Critérios de aceite:** Perfil do usuário atualizado com sucesso.
*   **Como testar:** Testar a atualização de perfil.

### Task 106: Integrar formulário de Perfil do Usuário com Server Action
*   **Objetivo:** Conectar o formulário de perfil do frontend com a Server Action.
*   **Descrição:** No `app/(app)/settings/profile/page.tsx`, chamar a Server Action `updateProfile` no `onSubmit`. Gerenciar estados de loading e feedback de sucesso/erro.
*   **Arquivos envolvidos:** `app/(app)/settings/profile/page.tsx`.
*   **Dependências:** Task 105.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Atualização de perfil funcional.
*   **Como testar:** Realizar a atualização de perfil no navegador.

### Task 107: Implementar upload de avatar do usuário (Frontend UI)
*   **Objetivo:** Permitir que o usuário faça upload de uma imagem de avatar.
*   **Descrição:** Adicionar um componente de upload de arquivo na página de perfil. Exibir preview da imagem.
*   **Arquivos envolvidos:** `app/(app)/settings/profile/page.tsx`, `components/common/FileUpload.tsx` (reutilizar ou adaptar).
*   **Dependências:** Task 103.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Input`, `Image`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Componente de upload de avatar renderizado e funcional.
*   **Como testar:** Selecionar um arquivo de imagem para upload.

### Task 108: Criar Server Action para upload de avatar
*   **Objetivo:** Implementar a lógica de upload de arquivo para o Supabase Storage para avatares.
*   **Descrição:** Criar uma Server Action em `server-actions/profile.ts` (ou `server-actions/storage.ts`) que recebe o arquivo. Utilizar `SupabaseStorageService.uploadFile` para fazer o upload para o bucket `avatars` no caminho `user/{user_id}/avatar.png`. Atualizar `avatar_url` na tabela `profiles`.
*   **Arquivos envolvidos:** `server-actions/profile.ts`, `services/storage.ts`, `repositories/profileRepository.ts`.
*   **Dependências:** Task 105.
*   **Banco de dados envolvido:** Supabase Storage, `profiles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Upload de arquivo, atualização de URL.
*   **Critérios de aceite:** Arquivo de avatar enviado para o Supabase Storage e URL atualizada no banco de dados.
*   **Como testar:** Testar o upload de avatar e verificar o Storage e o banco de dados.

### Task 109: Integrar upload de avatar com Server Action
*   **Objetivo:** Conectar o componente de upload de avatar do frontend com a Server Action.
*   **Descrição:** No `app/(app)/settings/profile/page.tsx`, chamar a Server Action de upload no evento de seleção de arquivo. Gerenciar estados de loading e feedback.
*   **Arquivos envolvidos:** `app/(app)/settings/profile/page.tsx`.
*   **Dependências:** Task 108.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Upload de avatar funcional do frontend ao backend.
*   **Como testar:** Realizar o upload de avatar no navegador.

### Task 110: Implementar página de Preferências do Usuário (UI)
*   **Objetivo:** Permitir que o usuário configure preferências pessoais (ex: tema claro/escuro, notificações).
*   **Descrição:** Criar a página `app/(app)/settings/preferences/page.tsx` com opções para tema (usando `shadcn/ui` `Switch` ou `RadioGroup`) e notificações (checkboxes).
*   **Arquivos envolvidos:** `app/(app)/settings/preferences/page.tsx`.
*   **Dependências:** Task 060 (componentes UI).
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Switch`, `RadioGroup`, `Checkbox`.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Página de preferências renderizada corretamente.
*   **Como testar:** Acessar `/settings/preferences` no navegador.

### Task 111: Criar Server Action para atualizar Preferências do Usuário
*   **Objetivo:** Permitir que o usuário atualize suas preferências.
*   **Descrição:** Criar uma Server Action em `server-actions/profile.ts` que chama `ProfileService.updatePreferences`. As preferências podem ser armazenadas como `jsonb` na tabela `profiles` ou em uma nova tabela `user_settings`.
*   **Arquivos envolvidos:** `server-actions/profile.ts`, `services/profile.ts`, `repositories/profileRepository.ts`.
*   **Dependências:** Task 105, Task 110.
*   **Banco de dados envolvido:** `profiles` (ou `user_settings`).
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Atualização de preferências.
*   **Critérios de aceite:** Preferências do usuário atualizadas com sucesso.
*   **Como testar:** Testar a atualização de preferências.

### Task 112: Integrar formulário de Preferências do Usuário com Server Action
*   **Objetivo:** Conectar o formulário de preferências do frontend com a Server Action.
*   **Descrição:** No `app/(app)/settings/preferences/page.tsx`, chamar a Server Action `updatePreferences` no `onSubmit` ou `onChange` dos controles. Gerenciar estados de loading e feedback.
*   **Arquivos envolvidos:** `app/(app)/settings/preferences/page.tsx`.
*   **Dependências:** Task 111.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Button` (estado de loading), `Toast` (para feedback).
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Atualização de preferências funcional.
*   **Como testar:** Realizar a atualização de preferências no navegador.

### Task 113: Implementar tema claro/escuro (Frontend)
*   **Objetivo:** Permitir que o usuário alterne entre tema claro e escuro.
*   **Descrição:** Utilizar `next-themes` ou uma solução customizada com Tailwind CSS para gerenciar o tema. Integrar com a preferência salva no perfil do usuário.
*   **Arquivos envolvidos:** `app/layout.tsx`, `components/ui/theme-toggle.tsx`, `tailwind.config.ts`.
*   **Dependências:** Task 003, Task 112.
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** `Switch` ou `Button` para alternar tema.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Tema da aplicação muda entre claro e escuro e persiste entre sessões.
*   **Como testar:** Alternar o tema e recarregar a página.

### Task 114: Criar `ProfileService` e `ProfileRepository`
*   **Objetivo:** Encapsular a lógica de negócio e acesso a dados para perfis de usuário.
*   **Descrição:** Criar `services/profile.ts` e `repositories/profileRepository.ts`. O `ProfileRepository` deve ter métodos para `getProfileById`, `updateProfile`, `updatePreferences`. O `ProfileService` deve orquestrar essas operações.
*   **Arquivos envolvidos:** `services/profile.ts`, `repositories/profileRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 013.
*   **Banco de dados envolvido:** `profiles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Gestão de perfis de usuário.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD de perfis.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 115: Criar Server Action para obter Perfil do Usuário
*   **Objetivo:** Permitir que o frontend obtenha as informações do perfil do usuário logado.
*   **Descrição:** Criar uma Server Action em `server-actions/profile.ts` que chama `ProfileService.getProfileById` usando o `user_id` do contexto do usuário.
*   **Arquivos envolvidos:** `server-actions/profile.ts`, `services/profile.ts`.
*   **Dependências:** Task 114.
*   **Banco de dados envolvido:** `profiles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Nenhuma.
*   **Critérios de aceite:** Server Action retorna os dados corretos do perfil.
*   **Como testar:** Chamar a Server Action de um componente e exibir os dados.

### Task 116: Criar `AuthService`
*   **Objetivo:** Encapsular a lógica de autenticação com Supabase Auth.
*   **Descrição:** Criar `services/auth.ts` com métodos para `signInWithPassword`, `signUp`, `sendPasswordResetEmail`, `updateUser`, `signOut`. Utilizar o cliente Supabase configurado em `lib/supabase.ts`.
*   **Arquivos envolvidos:** `services/auth.ts`, `lib/supabase.ts`.
*   **Dependências:** Task 058.
*   **Banco de dados envolvido:** Supabase Auth.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Lógica de autenticação.
*   **Critérios de aceite:** Métodos de autenticação implementados e funcionando com Supabase Auth.
*   **Como testar:** Testar cada método individualmente.

### Task 117: Criar `StorageService`
*   **Objetivo:** Encapsular a lógica de upload e gerenciamento de arquivos no Supabase Storage.
*   **Descrição:** Criar `services/storage.ts` com métodos para `uploadFile`, `deleteFile`, `getPublicUrl`. Utilizar o cliente Supabase configurado em `lib/supabase.ts`.
*   **Arquivos envolvidos:** `services/storage.ts`, `lib/supabase.ts`.
*   **Dependências:** Task 058.
*   **Banco de dados envolvido:** Supabase Storage.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Gerenciamento de arquivos.
*   **Critérios de aceite:** Métodos de storage implementados e funcionando com Supabase Storage.
*   **Como testar:** Testar cada método individualmente.

### Task 118: Criar `NotificationService`
*   **Objetivo:** Encapsular a lógica de criação e gerenciamento de notificações internas.
*   **Descrição:** Criar `services/notification.ts` com métodos para `createNotification`, `getNotificationsByUserId`, `markNotificationAsRead`. Utilizar `NotificationRepository`.
*   **Arquivos envolvidos:** `services/notification.ts`, `repositories/notificationRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 030.
*   **Banco de dados envolvido:** `notifications`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Gerenciamento de notificações.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD de notificações.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 119: Criar `AuditService`
*   **Objetivo:** Encapsular a lógica de registro de logs de auditoria.
*   **Descrição:** Criar `services/audit.ts` com um método `createAuditLog`. Utilizar `AuditRepository` e a função SQL `create_audit_log`.
*   **Arquivos envolvidos:** `services/audit.ts`, `repositories/auditRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 029, Task 035.
*   **Banco de dados envolvido:** `audit_logs`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Registro de auditoria.
*   **Critérios de aceite:** Serviço e repositório criados para registro de auditoria.
*   **Como testar:** Testar o método diretamente no backend.

### Task 120: Criar `SubscriptionService`
*   **Objetivo:** Encapsular a lógica de gerenciamento de assinaturas e integração com Stripe.
*   **Descrição:** Criar `services/subscription.ts` com métodos para `startTrial`, `upgradeSubscription`, `cancelSubscription`, `syncSubscriptionStatus`. Utilizar `SubscriptionRepository` e `StripeService`.
*   **Arquivos envolvidos:** `services/subscription.ts`, `repositories/subscriptionRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 027.
*   **Banco de dados envolvido:** `subscriptions`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Gerenciamento de assinaturas.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD de assinaturas.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 121: Criar `StripeService`
*   **Objetivo:** Encapsular a interação direta com a API do Stripe.
*   **Descrição:** Criar `services/stripe.ts` com métodos para `createCheckoutSession`, `createCustomerPortalSession`, `handleWebhookEvent`. Utilizar a biblioteca `stripe-node`.
*   **Arquivos envolvidos:** `services/stripe.ts`, `lib/stripe.ts`.
*   **Dependências:** Task 007 (variáveis de ambiente Stripe).
*   **Banco de dados envolvido:** Nenhum.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Integração com Stripe.
*   **Critérios de aceite:** Métodos de Stripe implementados e funcionando com a API do Stripe.
*   **Como testar:** Testar cada método individualmente.

### Task 122: Criar `SettingsService`
*   **Objetivo:** Encapsular a lógica de gerenciamento de configurações do tenant.
*   **Descrição:** Criar `services/settings.ts` com métodos para `getSetting`, `updateSetting`. Utilizar `SettingsRepository`.
*   **Arquivos envolvidos:** `services/settings.ts`, `repositories/settingsRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 031.
*   **Banco de dados envolvido:** `settings`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Gerenciamento de configurações.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD de configurações.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 123: Criar `CustomerService` e `CustomerRepository`
*   **Objetivo:** Encapsular a lógica de negócio e acesso a dados para clientes.
*   **Descrição:** Criar `services/customer.ts` e `repositories/customerRepository.ts`. O `CustomerRepository` deve ter métodos para `createCustomer`, `getCustomerById`, `updateCustomer`, `deleteCustomer` (soft delete), `listCustomers`. O `CustomerService` deve orquestrar essas operações e incluir lógica de negócio (ex: validações adicionais).
*   **Arquivos envolvidos:** `services/customer.ts`, `repositories/customerRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 015.
*   **Banco de dados envolvido:** `customers`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** CRUD de clientes.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD de clientes.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 124: Criar `VehicleService` e `VehicleRepository`
*   **Objetivo:** Encapsular a lógica de negócio e acesso a dados para veículos.
*   **Descrição:** Criar `services/vehicle.ts` e `repositories/vehicleRepository.ts`. O `VehicleRepository` deve ter métodos para `createVehicle`, `getVehicleById`, `updateVehicle`, `deleteVehicle` (soft delete), `listVehiclesByCustomer`.
*   **Arquivos envolvidos:** `services/vehicle.ts`, `repositories/vehicleRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 017.
*   **Banco de dados envolvido:** `vehicles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** CRUD de veículos.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD de veículos.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 125: Criar `ServiceCatalogService` e `ServiceCatalogRepository`
*   **Objetivo:** Encapsular a lógica de negócio e acesso a dados para serviços do catálogo.
*   **Descrição:** Criar `services/serviceCatalog.ts` e `repositories/serviceCatalogRepository.ts`. O `ServiceCatalogRepository` deve ter métodos para `createService`, `getServiceById`, `updateService`, `deleteService` (soft delete), `listServices`.
*   **Arquivos envolvidos:** `services/serviceCatalog.ts`, `repositories/serviceCatalogRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 018.
*   **Banco de dados envolvido:** `services`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** CRUD de serviços do catálogo.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD de serviços.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 126: Criar `ProductCatalogService` e `ProductCatalogRepository`
*   **Objetivo:** Encapsular a lógica de negócio e acesso a dados para produtos do catálogo.
*   **Descrição:** Criar `services/productCatalog.ts` e `repositories/productCatalogRepository.ts`. O `ProductCatalogRepository` deve ter métodos para `createProduct`, `getProductById`, `updateProduct`, `deleteProduct` (soft delete), `listProducts`.
*   **Arquivos envolvidos:** `services/productCatalog.ts`, `repositories/productCatalogRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 019.
*   **Banco de dados envolvido:** `products`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** CRUD de produtos do catálogo.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD de produtos.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 127: Criar `WorkOrderService` e `WorkOrderRepository`
*   **Objetivo:** Encapsular a lógica de negócio e acesso a dados para ordens de serviço.
*   **Descrição:** Criar `services/workOrder.ts` e `repositories/workOrderRepository.ts`. O `WorkOrderRepository` deve ter métodos para `createWorkOrder`, `getWorkOrderById`, `updateWorkOrder`, `deleteWorkOrder` (soft delete), `listWorkOrders`, `addServiceToWorkOrder`, `addProductToWorkOrder`, `updateWorkOrderStatus`, `addWorkOrderHistory`.
*   **Arquivos envolvidos:** `services/workOrder.ts`, `repositories/workOrderRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 020, Task 021, Task 022, Task 023.
*   **Banco de dados envolvido:** `work_orders`, `work_order_services`, `work_order_products`, `work_order_history`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** CRUD e fluxo de OS.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para CRUD e gestão de OS.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 128: Criar `FinancialService` e Repositórios Financeiros
*   **Objetivo:** Encapsular a lógica de negócio e acesso a dados para o módulo financeiro.
*   **Descrição:** Criar `services/financial.ts` e repositórios para `accountsReceivableRepository.ts`, `accountsPayableRepository.ts`, `financialTransactionRepository.ts`. Implementar métodos para `createAccountReceivable`, `createAccountPayable`, `recordTransaction`, `listAccountsReceivable`, `listAccountsPayable`, `getFinancialSummary`.
*   **Arquivos envolvidos:** `services/financial.ts`, `repositories/accountsReceivableRepository.ts`, `repositories/accountsPayableRepository.ts`, `repositories/financialTransactionRepository.ts`, `types/entities.ts`.
*   **Dependências:** Task 024, Task 025, Task 026.
*   **Banco de dados envolvido:** `accounts_receivable`, `accounts_payable`, `financial_transactions`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Gestão financeira.
*   **Critérios de aceite:** Serviços e repositórios criados com métodos básicos para gestão financeira.
*   **Como testar:** Testar os métodos diretamente no backend.

### Task 129: Criar Server Action para listar Clientes
*   **Objetivo:** Permitir que o frontend obtenha a lista de clientes do tenant atual.
*   **Descrição:** Criar uma Server Action em `server-actions/customer.ts` que chama `CustomerService.listCustomers` usando o `tenant_id` do contexto. Incluir parâmetros para paginação, busca e filtros.
*   **Arquivos envolvidos:** `server-actions/customer.ts`, `services/customer.ts`.
*   **Dependências:** Task 123.
*   **Banco de dados envolvido:** `customers`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Listagem de clientes.
*   **Critérios de aceite:** Server Action retorna a lista correta de clientes, respeitando paginação e filtros.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 130: Criar Server Action para criar Cliente
*   **Objetivo:** Permitir que o frontend crie um novo cliente.
*   **Descrição:** Criar uma Server Action em `server-actions/customer.ts` que chama `CustomerService.createCustomer`. A ação deve validar os dados de entrada e usar o `tenant_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/customer.ts`, `services/customer.ts`.
*   **Dependências:** Task 123.
*   **Banco de dados envolvido:** `customers`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Criação de cliente.
*   **Critérios de aceite:** Novo cliente criado com sucesso.
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 131: Criar Server Action para obter Cliente por ID
*   **Objetivo:** Permitir que o frontend obtenha os detalhes de um cliente específico.
*   **Descrição:** Criar uma Server Action em `server-actions/customer.ts` que chama `CustomerService.getCustomerById` usando o `tenant_id` do contexto e o `customer_id`.
*   **Arquivos envolvidos:** `server-actions/customer.ts`, `services/customer.ts`.
*   **Dependências:** Task 123.
*   **Banco de dados envolvido:** `customers`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Obtenção de detalhes do cliente.
*   **Critérios de aceite:** Server Action retorna os dados corretos do cliente.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 132: Criar Server Action para atualizar Cliente
*   **Objetivo:** Permitir que o frontend atualize as informações de um cliente.
*   **Descrição:** Criar uma Server Action em `server-actions/customer.ts` que chama `CustomerService.updateCustomer`. A ação deve validar os dados de entrada e usar o `tenant_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/customer.ts`, `services/customer.ts`.
*   **Dependências:** Task 123.
*   **Banco de dados envolvido:** `customers`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Atualização de cliente.
*   **Critérios de aceite:** Cliente atualizado com sucesso.
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 133: Criar Server Action para soft delete de Cliente
*   **Objetivo:** Permitir que o frontend realize o soft delete de um cliente.
*   **Descrição:** Criar uma Server Action em `server-actions/customer.ts` que chama `CustomerService.deleteCustomer`. A ação deve usar o `tenant_id` do contexto e o `customer_id`.
*   **Arquivos envolvidos:** `server-actions/customer.ts`, `services/customer.ts`.
*   **Dependências:** Task 123.
*   **Banco de dados envolvido:** `customers`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Soft delete de cliente.
*   **Critérios de aceite:** Cliente marcado como deletado (`deleted_at` preenchido).
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 134: Criar Server Action para listar Veículos por Cliente
*   **Objetivo:** Permitir que o frontend obtenha a lista de veículos de um cliente específico.
*   **Descrição:** Criar uma Server Action em `server-actions/vehicle.ts` que chama `VehicleService.listVehiclesByCustomer` usando o `tenant_id` do contexto e o `customer_id`.
*   **Arquivos envolvidos:** `server-actions/vehicle.ts`, `services/vehicle.ts`.
*   **Dependências:** Task 124.
*   **Banco de dados envolvido:** `vehicles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Listagem de veículos por cliente.
*   **Critérios de aceite:** Server Action retorna a lista correta de veículos do cliente.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 135: Criar Server Action para criar Veículo
*   **Objetivo:** Permitir que o frontend crie um novo veículo para um cliente.
*   **Descrição:** Criar uma Server Action em `server-actions/vehicle.ts` que chama `VehicleService.createVehicle`. A ação deve validar os dados de entrada e usar o `tenant_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/vehicle.ts`, `services/vehicle.ts`.
*   **Dependências:** Task 124.
*   **Banco de dados envolvido:** `vehicles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Criação de veículo.
*   **Critérios de aceite:** Novo veículo criado com sucesso.
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 136: Criar Server Action para obter Veículo por ID
*   **Objetivo:** Permitir que o frontend obtenha os detalhes de um veículo específico.
*   **Descrição:** Criar uma Server Action em `server-actions/vehicle.ts` que chama `VehicleService.getVehicleById` usando o `tenant_id` do contexto e o `vehicle_id`.
*   **Arquivos envolvidos:** `server-actions/vehicle.ts`, `services/vehicle.ts`.
*   **Dependências:** Task 124.
*   **Banco de dados envolvido:** `vehicles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Obtenção de detalhes do veículo.
*   **Critérios de aceite:** Server Action retorna os dados corretos do veículo.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 137: Criar Server Action para atualizar Veículo
*   **Objetivo:** Permitir que o frontend atualize as informações de um veículo.
*   **Descrição:** Criar uma Server Action em `server-actions/vehicle.ts` que chama `VehicleService.updateVehicle`. A ação deve validar os dados de entrada e usar o `tenant_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/vehicle.ts`, `services/vehicle.ts`.
*   **Dependências:** Task 124.
*   **Banco de dados envolvido:** `vehicles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Atualização de veículo.
*   **Critérios de aceite:** Veículo atualizado com sucesso.
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 138: Criar Server Action para soft delete de Veículo
*   **Objetivo:** Permitir que o frontend realize o soft delete de um veículo.
*   **Descrição:** Criar uma Server Action em `server-actions/vehicle.ts` que chama `VehicleService.deleteVehicle`. A ação deve usar o `tenant_id` do contexto e o `vehicle_id`.
*   **Arquivos envolvidos:** `server-actions/vehicle.ts`, `services/vehicle.ts`.
*   **Dependências:** Task 124.
*   **Banco de dados envolvido:** `vehicles`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Soft delete de veículo.
*   **Critérios de aceite:** Veículo marcado como deletado (`deleted_at` preenchido).
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 139: Criar Server Action para listar Serviços do Catálogo
*   **Objetivo:** Permitir que o frontend obtenha a lista de serviços do catálogo do tenant atual.
*   **Descrição:** Criar uma Server Action em `server-actions/serviceCatalog.ts` que chama `ServiceCatalogService.listServices` usando o `tenant_id` do contexto. Incluir parâmetros para paginação, busca e filtros.
*   **Arquivos envolvidos:** `server-actions/serviceCatalog.ts`, `services/serviceCatalog.ts`.
*   **Dependências:** Task 125.
*   **Banco de dados envolvido:** `services`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Listagem de serviços.
*   **Critérios de aceite:** Server Action retorna a lista correta de serviços, respeitando paginação e filtros.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 140: Criar Server Action para criar Serviço
*   **Objetivo:** Permitir que o frontend crie um novo serviço no catálogo.
*   **Descrição:** Criar uma Server Action em `server-actions/serviceCatalog.ts` que chama `ServiceCatalogService.createService`. A ação deve validar os dados de entrada e usar o `tenant_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/serviceCatalog.ts`, `services/serviceCatalog.ts`.
*   **Dependências:** Task 125.
*   **Banco de dados envolvido:** `services`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Criação de serviço.
*   **Critérios de aceite:** Novo serviço criado com sucesso.
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 141: Criar Server Action para obter Serviço por ID
*   **Objetivo:** Permitir que o frontend obtenha os detalhes de um serviço específico.
*   **Descrição:** Criar uma Server Action em `server-actions/serviceCatalog.ts` que chama `ServiceCatalogService.getServiceById` usando o `tenant_id` do contexto e o `service_id`.
*   **Arquivos envolvidos:** `server-actions/serviceCatalog.ts`, `services/serviceCatalog.ts`.
*   **Dependências:** Task 125.
*   **Banco de dados envolvido:** `services`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Obtenção de detalhes do serviço.
*   **Critérios de aceite:** Server Action retorna os dados corretos do serviço.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 142: Criar Server Action para atualizar Serviço
*   **Objetivo:** Permitir que o frontend atualize as informações de um serviço.
*   **Descrição:** Criar uma Server Action em `server-actions/serviceCatalog.ts` que chama `ServiceCatalogService.updateService`. A ação deve validar os dados de entrada e usar o `tenant_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/serviceCatalog.ts`, `services/serviceCatalog.ts`.
*   **Dependências:** Task 125.
*   **Banco de dados envolvido:** `services`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Atualização de serviço.
*   **Critérios de aceite:** Serviço atualizado com sucesso.
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 143: Criar Server Action para soft delete de Serviço
*   **Objetivo:** Permitir que o frontend realize o soft delete de um serviço.
*   **Descrição:** Criar uma Server Action em `server-actions/serviceCatalog.ts` que chama `ServiceCatalogService.deleteService`. A ação deve usar o `tenant_id` do contexto e o `service_id`.
*   **Arquivos envolvidos:** `server-actions/serviceCatalog.ts`, `services/serviceCatalog.ts`.
*   **Dependências:** Task 125.
*   **Banco de dados envolvido:** `services`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Soft delete de serviço.
*   **Critérios de aceite:** Serviço marcado como deletado (`deleted_at` preenchido).
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 144: Criar Server Action para listar Produtos do Catálogo
*   **Objetivo:** Permitir que o frontend obtenha a lista de produtos do catálogo do tenant atual.
*   **Descrição:** Criar uma Server Action em `server-actions/productCatalog.ts` que chama `ProductCatalogService.listProducts` usando o `tenant_id` do contexto. Incluir parâmetros para paginação, busca e filtros.
*   **Arquivos envolvidos:** `server-actions/productCatalog.ts`, `services/productCatalog.ts`.
*   **Dependências:** Task 126.
*   **Banco de dados envolvido:** `products`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Listagem de produtos.
*   **Critérios de aceite:** Server Action retorna a lista correta de produtos, respeitando paginação e filtros.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 145: Criar Server Action para criar Produto
*   **Objetivo:** Permitir que o frontend crie um novo produto no catálogo.
*   **Descrição:** Criar uma Server Action em `server-actions/productCatalog.ts` que chama `ProductCatalogService.createProduct`. A ação deve validar os dados de entrada e usar o `tenant_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/productCatalog.ts`, `services/productCatalog.ts`.
*   **Dependências:** Task 126.
*   **Banco de dados envolvido:** `products`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Criação de produto.
*   **Critérios de aceite:** Novo produto criado com sucesso.
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 146: Criar Server Action para obter Produto por ID
*   **Objetivo:** Permitir que o frontend obtenha os detalhes de um produto específico.
*   **Descrição:** Criar uma Server Action em `server-actions/productCatalog.ts` que chama `ProductCatalogService.getProductById` usando o `tenant_id` do contexto e o `product_id`.
*   **Arquivos envolvidos:** `server-actions/productCatalog.ts`, `services/productCatalog.ts`.
*   **Dependências:** Task 126.
*   **Banco de dados envolvido:** `products`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Obtenção de detalhes do produto.
*   **Critérios de aceite:** Server Action retorna os dados corretos do produto.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 147: Criar Server Action para atualizar Produto
*   **Objetivo:** Permitir que o frontend atualize as informações de um produto.
*   **Descrição:** Criar uma Server Action em `server-actions/productCatalog.ts` que chama `ProductCatalogService.updateProduct`. A ação deve validar os dados de entrada e usar o `tenant_id` do contexto.
*   **Arquivos envolvidos:** `server-actions/productCatalog.ts`, `services/productCatalog.ts`.
*   **Dependências:** Task 126.
*   **Banco de dados envolvido:** `products`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Atualização de produto.
*   **Critérios de aceite:** Produto atualizado com sucesso.
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 148: Criar Server Action para soft delete de Produto
*   **Objetivo:** Permitir que o frontend realize o soft delete de um produto.
*   **Descrição:** Criar uma Server Action em `server-actions/productCatalog.ts` que chama `ProductCatalogService.deleteProduct`. A ação deve usar o `tenant_id` do contexto e o `product_id`.
*   **Arquivos envolvidos:** `server-actions/productCatalog.ts`, `services/productCatalog.ts`.
*   **Dependências:** Task 126.
*   **Banco de dados envolvido:** `products`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Soft delete de produto.
*   **Critérios de aceite:** Produto marcado como deletado (`deleted_at` preenchido).
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.

### Task 149: Criar Server Action para listar Ordens de Serviço
*   **Objetivo:** Permitir que o frontend obtenha a lista de ordens de serviço do tenant atual.
*   **Descrição:** Criar uma Server Action em `server-actions/workOrder.ts` que chama `WorkOrderService.listWorkOrders` usando o `tenant_id` do contexto. Incluir parâmetros para paginação, busca e filtros.
*   **Arquivos envolvidos:** `server-actions/workOrder.ts`, `services/workOrder.ts`.
*   **Dependências:** Task 127.
*   **Banco de dados envolvido:** `work_orders`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Listagem de ordens de serviço.
*   **Critérios de aceite:** Server Action retorna a lista correta de ordens de serviço, respeitando paginação e filtros.
*   **Como testar:** Chamar a Server Action e verificar o retorno.

### Task 150: Criar Server Action para criar Ordem de Serviço
*   **Objetivo:** Permitir que o frontend crie uma nova ordem de serviço.
*   **Descrição:** Criar uma Server Action em `server-actions/workOrder.ts` que chama `WorkOrderService.createWorkOrder`. A ação deve validar os dados de entrada, usar o `tenant_id` do contexto e gerar o `work_order_number` usando a função SQL.
*   **Arquivos envolvidos:** `server-actions/workOrder.ts`, `services/workOrder.ts`.
*   **Dependências:** Task 127, Task 034.
*   **Banco de dados envolvido:** `work_orders`.
*   **Componentes envolvidos:** Nenhum.
*   **Regras de negócio:** Criação de ordem de serviço, geração de número de OS.
*   **Critérios de aceite:** Nova ordem de serviço criada com sucesso e número de OS gerado.
*   **Como testar:** Chamar a Server Action e verificar o banco de dados.
