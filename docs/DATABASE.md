# OficinaPro — Documentação do Banco de Dados

**Autor:** Manus AI  
**Data:** 09 de junho de 2026  
**Versão:** 1.0  
**Finalidade:** Este documento detalha a arquitetura do banco de dados PostgreSQL para o sistema OficinaPro, um SaaS multiempresa. Ele serve como referência para a implementação das migrations no Supabase, garantindo a integridade, segurança e escalabilidade dos dados.

## 1. Arquitetura do Banco

O banco de dados do OficinaPro é projetado para operar em um ambiente **multi-tenant (multiempresa)**, onde milhares de oficinas compartilham a mesma infraestrutura de banco de dados, mas com total isolamento lógico de seus dados. A base tecnológica é o **PostgreSQL** gerenciado pelo **Supabase**, que oferece recursos nativos para autenticação, Row Level Security (RLS) e armazenamento de arquivos.

### 1.1 Modelo Multi-tenant

O OficinaPro adota o modelo **single application, shared database, tenant isolation by row**. Isso significa que todas as oficinas utilizam a mesma instância do banco de dados, mas cada registro de dados operacionais é associado a um `tenant_id` específico. Este `tenant_id` é uma chave estrangeira para a tabela `tenants`, que representa cada oficina.

**Princípios:**
*   **`tenant_id` Obrigatório:** Todas as tabelas que armazenam dados específicos de uma oficina (clientes, veículos, ordens de serviço, produtos, etc.) devem incluir uma coluna `tenant_id` do tipo `UUID` e ser `NOT NULL`.
*   **Isolamento Lógico:** O acesso aos dados é estritamente controlado para garantir que um usuário de uma oficina não possa visualizar, modificar ou excluir dados pertencentes a outra oficina.

### 1.2 Estratégia de Isolamento

O isolamento de dados é implementado em múltiplas camadas para garantir robustez:

*   **Row Level Security (RLS):** É a principal camada de isolamento no nível do banco de dados. Políticas RLS são aplicadas a todas as tabelas multi-tenant para filtrar automaticamente as linhas visíveis para cada usuário autenticado, com base no `tenant_id` ao qual ele pertence e seu perfil de acesso. Isso impede que consultas diretas ou acidentais exponham dados entre tenants.
*   **Validação no Backend/Server Actions:** Antes de qualquer operação de escrita ou leitura que não seja diretamente coberta por RLS (ex: criação de um novo tenant, operações administrativas globais), o backend (Next.js Server Actions/Route Handlers) deve validar o `tenant_id` do usuário e a permissão para a ação solicitada.
*   **Storage Segregado:** Arquivos (como logotipos) serão armazenados no Supabase Storage em caminhos que incluem o `tenant_id` (ex: `tenant/{tenant_id}/logos/logo.png`), garantindo que o acesso a arquivos também seja isolado.

### 1.3 Row Level Security (RLS)

O RLS é um recurso do PostgreSQL que permite definir políticas de segurança diretamente nas tabelas, controlando o acesso a linhas individuais. No Supabase, o RLS é fundamental para a segurança multi-tenant. [1]

*   **Habilitação:** O RLS será habilitado em todas as tabelas que contêm dados específicos de tenants (`ALTER TABLE <table_name> ENABLE ROW LEVEL SECURITY;`).
*   **Políticas:** Serão criadas políticas para as operações `SELECT`, `INSERT`, `UPDATE` e `DELETE` para cada tabela multi-tenant. Essas políticas utilizarão funções auxiliares como `auth.uid()` (do Supabase Auth) e funções customizadas para verificar a associação do usuário ao `tenant_id` e seu perfil.

### 1.4 Índices

Índices serão criados para otimizar o desempenho das consultas, especialmente em tabelas grandes e frequentemente acessadas. Para tabelas multi-tenant, índices compostos com `tenant_id` serão prioritários para acelerar as buscas e garantir que as políticas RLS sejam eficientes.

**Exemplo:** `CREATE INDEX idx_customers_tenant_id_name ON customers (tenant_id, name);`

### 1.5 Constraints

Constraints (restrições) serão utilizadas para garantir a integridade dos dados no nível do banco de dados. Isso inclui chaves primárias (`PRIMARY KEY`), chaves estrangeiras (`FOREIGN KEY`), restrições de unicidade (`UNIQUE`), restrições de verificação (`CHECK`) e nulidade (`NOT NULL`).

**Exemplo:** `ALTER TABLE vehicles ADD CONSTRAINT unique_plate_per_tenant UNIQUE (tenant_id, plate);`

### 1.6 Triggers

Triggers (gatilhos) serão empregados para automatizar tarefas no banco de dados, como a atualização de timestamps, a geração de números sequenciais e a auditoria de alterações.

*   **`updated_at` automático:** Um trigger genérico será criado para atualizar automaticamente a coluna `updated_at` em todas as tabelas que a possuírem, sempre que uma linha for modificada.
*   **Auditoria:** Triggers podem ser usados para registrar alterações críticas em uma tabela de `audit_logs`.
*   **Geração de número da OS:** Um trigger ou função será responsável por gerar o número sequencial da Ordem de Serviço por tenant.

### 1.7 Soft Delete

Para a maioria das entidades operacionais (clientes, veículos, ordens de serviço, produtos, serviços), será implementado o **soft delete** (exclusão lógica). Em vez de remover fisicamente os registros, uma coluna `deleted_at` do tipo `TIMESTAMPTZ` será atualizada com a data e hora da exclusão. Isso permite manter o histórico, recuperar dados e simplificar a auditoria.

*   Consultas padrão devem filtrar registros onde `deleted_at IS NULL`.
*   Políticas RLS e outras lógicas de negócio devem considerar o status de `deleted_at`.

### 1.8 Auditoria

Um sistema de auditoria será implementado para registrar ações críticas realizadas pelos usuários, como criação, atualização e exclusão de registros importantes. Isso é essencial para rastreabilidade, segurança e conformidade.

*   **Tabela `audit_logs`:** Armazenará informações sobre quem fez o quê, quando, em qual tenant e em qual entidade.
*   **Triggers/Funções:** Serão utilizados para popular automaticamente a tabela de auditoria para ações específicas.

## 2. Enums

Enums (tipos enumerados) do PostgreSQL serão utilizados para padronizar e restringir os valores de certas colunas, garantindo a consistência dos dados e melhorando a legibilidade do esquema. Eles também são úteis para integrar com TypeScript, gerando tipos seguros.

```sql
-- Extensões recomendadas
create extension if not exists pgcrypto;

-- Enums principais
create type tenant_status as enum (
  'trial',
  'active',
  'past_due',
  'cancelled',
  'suspended'
);

create type tenant_role as enum (
  'owner',
  'admin',
  'employee'
);

create type membership_status as enum (
  'active',
  'inactive',
  'invited'
);

create type user_status as enum (
  'active',
  'inactive',
  'invited'
);

create type customer_type as enum (
  'individual',
  'company'
);

create type record_status as enum (
  'active',
  'inactive'
);

create type fuel_type as enum (
  'gasoline',
  'ethanol',
  'flex',
  'diesel',
  'electric',
  'hybrid',
  'other'
);

create type work_order_status as enum (
  'draft',
  'open',
  'diagnosis',
  'waiting_approval',
  'approved',
  'in_progress',
  'completed',
  'delivered',
  'cancelled',
  'reopened'
);

create type financial_status as enum (
  'pending',
  'paid',
  'overdue',
  'cancelled'
);

create type payment_method as enum (
  'cash',
  'pix',
  'credit_card',
  'debit_card',
  'bank_transfer',
  'boleto',
  'other'
);

create type transaction_type as enum (
  'income',
  'expense'
);

create type billing_interval as enum (
  'monthly',
  'yearly'
);

create type subscription_status as enum (
  'trialing',
  'active',
  'past_due',
  'canceled',
  'unpaid'
);

create type notification_type as enum (
  'system',
  'alert',
  'info',
  'warning'
);

create type notification_status as enum (
  'unread',
  'read',
  'archived'
);

create type setting_type as enum (
  'general',
  'financial',
  'operational'
);
```

## 3. Tabelas

Esta seção detalha cada tabela do banco de dados, incluindo seu objetivo, campos, tipos de dados, chaves, índices, relacionamentos e políticas de Row Level Security (RLS). Para cada tabela, será fornecido o SQL `CREATE TABLE` completo.

### 3.1 `tenants`

**Objetivo:** Armazenar as informações de cada oficina (tenant) que utiliza o sistema OficinaPro. Esta é a tabela central para o modelo multi-tenant.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da oficina. |
| `legal_name` | `text` | `FALSE` | | | Razão social da empresa. |
| `trade_name` | `text` | `FALSE` | | | Nome fantasia da oficina. |
| `document` | `text` | `TRUE` | | | CNPJ ou CPF da oficina. |
| `state_registration` | `text` | `TRUE` | | | Inscrição Estadual da oficina. |
| `email` | `text` | `FALSE` | | | E-mail principal de contato da oficina. |
| `phone` | `text` | `TRUE` | | | Telefone principal da oficina. |
| `whatsapp` | `text` | `TRUE` | | | Número de WhatsApp da oficina. |
| `logo_url` | `text` | `TRUE` | | | URL do logotipo da oficina no Supabase Storage. |
| `status` | `tenant_status` | `FALSE` | `'trial'` | | Status atual da assinatura da oficina. |
| `settings` | `jsonb` | `TRUE` | `'{}'` | | Configurações específicas da oficina (ex: numeração OS, moeda). |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:** Nenhuma.

**Índices:**
*   `idx_tenants_email` ON `tenants` (`email`)
*   `idx_tenants_document` ON `tenants` (`document`)

**Constraints:**
*   `UNIQUE (email)`
*   `UNIQUE (document)` (se `document` for obrigatório)

**Relacionamentos:**
*   `tenants` 1:N `tenant_users` (Uma oficina pode ter muitos usuários associados).
*   `tenants` 1:N `customers` (Uma oficina pode ter muitos clientes).
*   `tenants` 1:N `vehicles` (Uma oficina pode ter muitos veículos).
*   `tenants` 1:N `services` (Uma oficina pode ter muitos serviços cadastrados).
*   `tenants` 1:N `products` (Uma oficina pode ter muitos produtos cadastrados).
*   `tenants` 1:N `work_orders` (Uma oficina pode ter muitas ordens de serviço).
*   `tenants` 1:N `accounts_receivable` (Uma oficina pode ter muitas contas a receber).
*   `tenants` 1:N `accounts_payable` (Uma oficina pode ter muitas contas a pagar).
*   `tenants` 1:N `financial_transactions` (Uma oficina pode ter muitas transações financeiras).
*   `tenants` 1:N `subscriptions` (Uma oficina pode ter um histórico de assinaturas).
*   `tenants` 1:N `audit_logs` (Uma oficina pode ter muitos logs de auditoria).
*   `tenants` 1:N `notifications` (Uma oficina pode ter muitas notificações).
*   `tenants` 1:N `settings` (Uma oficina pode ter muitas configurações).

**Políticas RLS:**
*   **`SELECT`:** Proprietários e administradores podem visualizar todos os dados da sua oficina.
*   **`INSERT`:** Apenas usuários com permissão de `owner` podem criar novos tenants (geralmente via processo de onboarding).
*   **`UPDATE`:** Proprietários e administradores podem atualizar os dados da sua oficina.
*   **`DELETE`:** Apenas proprietários podem 
excluir (soft delete) sua oficina.

```sql
-- SQL CREATE TABLE for tenants
CREATE TABLE tenants (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  legal_name text NOT NULL,
  trade_name text NOT NULL,
  document text UNIQUE,
  state_registration text,
  email text NOT NULL UNIQUE,
  phone text,
  whatsapp text,
  logo_url text,
  status tenant_status NOT NULL DEFAULT 'trial',
  settings jsonb DEFAULT '{}'::jsonb,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for tenants
ALTER TABLE tenants ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenants_select_policy
ON tenants FOR SELECT
TO authenticated
USING (EXISTS (SELECT 1 FROM tenant_users tu WHERE tu.tenant_id = tenants.id AND tu.user_id = auth.uid() AND tu.status = 'active'));

CREATE POLICY tenants_insert_policy
ON tenants FOR INSERT
TO authenticated
WITH CHECK (EXISTS (SELECT 1 FROM tenant_users tu WHERE tu.tenant_id = tenants.id AND tu.user_id = auth.uid() AND tu.status = 'active' AND tu.role = 'owner'));

CREATE POLICY tenants_update_policy
ON tenants FOR UPDATE
TO authenticated
USING (EXISTS (SELECT 1 FROM tenant_users tu WHERE tu.tenant_id = tenants.id AND tu.user_id = auth.uid() AND tu.status = 'active' AND tu.role IN ('owner', 'admin')))
WITH CHECK (EXISTS (SELECT 1 FROM tenant_users tu WHERE tu.tenant_id = tenants.id AND tu.user_id = auth.uid() AND tu.status = 'active' AND tu.role IN ('owner', 'admin')));

CREATE POLICY tenants_delete_policy
ON tenants FOR DELETE
TO authenticated
USING (EXISTS (SELECT 1 FROM tenant_users tu WHERE tu.tenant_id = tenants.id AND tu.user_id = auth.uid() AND tu.status = 'active' AND tu.role = 'owner'));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_tenants
BEFORE UPDATE ON tenants
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.2 `tenant_users`

**Objetivo:** Gerenciar a associação de usuários a cada oficina (tenant), definindo o perfil (role) e o status de cada membro. Um usuário pode ser membro de múltiplas oficinas.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da associação. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina. |
| `user_id` | `uuid` | `FALSE` | | FK | Referência ao perfil do usuário. |
| `role` | `tenant_role` | `FALSE` | `employee` | | Perfil do usuário na oficina (owner, admin, employee). |
| `status` | `membership_status` | `FALSE` | `invited` | | Status da associação (active, inactive, invited). |
| `invited_by` | `uuid` | `TRUE` | | FK | Usuário que convidou (se aplicável). |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `user_id` REFERENCES `profiles(id)` ON DELETE CASCADE
*   `invited_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_tenant_users_tenant_id` ON `tenant_users` (`tenant_id`)
*   `idx_tenant_users_user_id` ON `tenant_users` (`user_id`)
*   `idx_tenant_users_tenant_user` ON `tenant_users` (`tenant_id`, `user_id`)

**Constraints:**
*   `UNIQUE (tenant_id, user_id)` (Garante que um usuário só pode ter uma associação por tenant).

**Relacionamentos:**
*   `tenant_users` N:1 `tenants` (Muitos membros pertencem a uma oficina).
*   `tenant_users` N:1 `profiles` (Muitos membros se referem a um perfil de usuário).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos de um tenant podem visualizar as associações de usuários da sua própria oficina.
*   **`INSERT`:** Apenas `owner` ou `admin` podem convidar novos usuários para sua oficina.
*   **`UPDATE`:** Apenas `owner` ou `admin` podem alterar o perfil ou status de usuários na sua oficina.
*   **`DELETE`:** Apenas `owner` ou `admin` podem remover usuários da sua oficina.

```sql
-- SQL CREATE TABLE for tenant_users
CREATE TABLE tenant_users (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  user_id uuid NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  role tenant_role NOT NULL DEFAULT 'employee',
  status membership_status NOT NULL DEFAULT 'invited',
  invited_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  UNIQUE (tenant_id, user_id)
);

-- RLS for tenant_users
ALTER TABLE tenant_users ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_users_select_policy
ON tenant_users FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id));

CREATE POLICY tenant_users_insert_policy
ON tenant_users FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY tenant_users_update_policy
ON tenant_users FOR UPDATE
TO authenticated
USING (has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]))
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY tenant_users_delete_policy
ON tenant_users FOR DELETE
TO authenticated
USING (has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_tenant_users
BEFORE UPDATE ON tenant_users
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.3 `profiles`

**Objetivo:** Armazenar informações adicionais dos usuários autenticados pelo Supabase Auth, complementando os dados básicos do `auth.users`. Esta tabela é 1:1 com `auth.users` e é a base para o `user_id` em `tenant_users`.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | | PK/FK | ID do usuário do Supabase Auth (`auth.users.id`). |
| `full_name` | `text` | `TRUE` | | | Nome completo do usuário. |
| `phone` | `text` | `TRUE` | | | Telefone de contato do usuário. |
| `avatar_url` | `text` | `TRUE` | | | URL da imagem de perfil do usuário. |
| `status` | `user_status` | `FALSE` | `active` | | Status do perfil (ativo, inativo, convidado). |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do perfil. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do perfil. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `id` REFERENCES `auth.users(id)` ON DELETE CASCADE (Esta é uma chave primária e estrangeira, garantindo 1:1 com `auth.users`)

**Índices:** Nenhuns adicionais, pois o `id` já é PK.

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `profiles` 1:1 `auth.users` (Um perfil para cada usuário de autenticação).
*   `profiles` 1:N `tenant_users` (Um perfil pode estar associado a várias oficinas).

**Políticas RLS:**
*   **`SELECT`:** Usuários podem visualizar seu próprio perfil. Membros de um tenant podem visualizar perfis de outros membros do mesmo tenant.
*   **`INSERT`:** Usuários podem criar seu próprio perfil (geralmente após o registro no Supabase Auth).
*   **`UPDATE`:** Usuários podem atualizar seu próprio perfil. `owner` ou `admin` podem atualizar o status de outros perfis na sua oficina.
*   **`DELETE`:** Usuários podem excluir seu próprio perfil (se permitido). `owner` ou `admin` podem inativar perfis de outros usuários na sua oficina (soft delete).

```sql
-- SQL CREATE TABLE for profiles
CREATE TABLE profiles (
  id uuid PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name text,
  phone text,
  avatar_url text,
  status user_status NOT NULL DEFAULT 'active',
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for profiles
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY profiles_select_policy
ON profiles FOR SELECT
TO authenticated
USING (id = auth.uid() OR EXISTS (SELECT 1 FROM tenant_users tu WHERE tu.user_id = profiles.id AND tu.status = 'active' AND is_active_tenant_member(tu.tenant_id)));

CREATE POLICY profiles_insert_policy
ON profiles FOR INSERT
TO authenticated
WITH CHECK (id = auth.uid());

CREATE POLICY profiles_update_policy
ON profiles FOR UPDATE
TO authenticated
USING (id = auth.uid() OR EXISTS (SELECT 1 FROM tenant_users tu WHERE tu.user_id = profiles.id AND tu.status = 'active' AND has_tenant_role(tu.tenant_id, ARRAY['owner', 'admin']::tenant_role[])))
WITH CHECK (id = auth.uid() OR EXISTS (SELECT 1 FROM tenant_users tu WHERE tu.user_id = profiles.id AND tu.status = 'active' AND has_tenant_role(tu.tenant_id, ARRAY['owner', 'admin']::tenant_role[])));

CREATE POLICY profiles_delete_policy
ON profiles FOR DELETE
TO authenticated
USING (id = auth.uid()); -- A exclusão de perfis de outros usuários deve ser feita via soft delete ou inativação em tenant_users

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_profiles
BEFORE UPDATE ON profiles
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.4 `customers`

**Objetivo:** Armazenar informações dos clientes de cada oficina. Cada cliente pertence a um `tenant_id`.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do cliente. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `type` | `customer_type` | `FALSE` | `individual` | | Tipo de cliente (individual ou company). |
| `name` | `text` | `FALSE` | | | Nome completo ou razão social do cliente. |
| `document` | `text` | `TRUE` | | | CPF ou CNPJ do cliente. |
| `secondary_document` | `text` | `TRUE` | | | RG ou Inscrição Estadual. |
| `email` | `text` | `TRUE` | | | E-mail de contato do cliente. |
| `phone` | `text` | `TRUE` | | | Telefone principal do cliente. |
| `whatsapp` | `text` | `TRUE` | | | Número de WhatsApp do cliente. |
| `birth_date` | `date` | `TRUE` | | | Data de nascimento (para pessoa física). |
| `notes` | `text` | `TRUE` | | | Observações adicionais sobre o cliente. |
| `status` | `record_status` | `FALSE` | `active` | | Status do registro (ativo ou inativo). |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |
| `deleted_at` | `timestamptz` | `TRUE` | | | Data e hora da exclusão lógica (soft delete). |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_customers_tenant_id_name` ON `customers` (`tenant_id`, `name`)
*   `idx_customers_tenant_id_document` ON `customers` (`tenant_id`, `document`) WHERE `document` IS NOT NULL
*   `idx_customers_tenant_id_phone` ON `customers` (`tenant_id`, `phone`) WHERE `phone` IS NOT NULL

**Constraints:**
*   `UNIQUE (tenant_id, document)` WHERE `document` IS NOT NULL (Garante que o documento seja único por tenant, se preenchido).

**Relacionamentos:**
*   `customers` N:1 `tenants` (Muitos clientes pertencem a uma oficina).
*   `customers` 1:N `customer_addresses` (Um cliente pode ter muitos endereços).
*   `customers` 1:N `vehicles` (Um cliente pode ter muitos veículos).
*   `customers` 1:N `work_orders` (Um cliente pode ter muitas ordens de serviço).
*   `customers` 1:N `accounts_receivable` (Um cliente pode ter muitas contas a receber).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos do tenant podem visualizar clientes da sua oficina que não foram logicamente excluídos.
*   **`INSERT`:** `owner`, `admin` ou `employee` podem criar novos clientes para sua oficina.
*   **`UPDATE`:** `owner`, `admin` ou `employee` podem atualizar clientes da sua oficina que não foram logicamente excluídos.
*   **`DELETE`:** `owner` ou `admin` podem realizar o soft delete de clientes da sua oficina.

```sql
-- SQL CREATE TABLE for customers
CREATE TABLE customers (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  type customer_type NOT NULL DEFAULT 'individual',
  name text NOT NULL,
  document text,
  secondary_document text,
  email text,
  phone text,
  whatsapp text,
  birth_date date,
  notes text,
  status record_status NOT NULL DEFAULT 'active',
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz,
  CONSTRAINT unique_document_per_tenant UNIQUE (tenant_id, document) WHERE (document IS NOT NULL)
);

-- RLS for customers
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;

CREATE POLICY customers_select_policy
ON customers FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL);

CREATE POLICY customers_insert_policy
ON customers FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY customers_update_policy
ON customers FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY customers_delete_policy
ON customers FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_customers
BEFORE UPDATE ON customers
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.5 `customer_addresses`

**Objetivo:** Armazenar os endereços dos clientes. Um cliente pode ter múltiplos endereços.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do endereço. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `customer_id` | `uuid` | `FALSE` | | FK | Referência ao cliente. |
| `zip_code` | `text` | `TRUE` | | | CEP do endereço. |
| `street` | `text` | `TRUE` | | | Nome da rua. |
| `number` | `text` | `TRUE` | | | Número do imóvel. |
| `complement` | `text` | `TRUE` | | | Complemento do endereço. |
| `district` | `text` | `TRUE` | | | Bairro. |
| `city` | `text` | `TRUE` | | | Cidade. |
| `state` | `char(2)` | `TRUE` | | | Estado (UF). |
| `country` | `text` | `FALSE` | `Brasil` | | País. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `customer_id` REFERENCES `customers(id)` ON DELETE CASCADE

**Índices:**
*   `idx_customer_addresses_tenant_id_customer_id` ON `customer_addresses` (`tenant_id`, `customer_id`)

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `customer_addresses` N:1 `customers` (Muitos endereços pertencem a um cliente).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos do tenant podem visualizar endereços de clientes da sua oficina.
*   **`INSERT`:** `owner`, `admin` ou `employee` podem adicionar endereços para clientes da sua oficina.
*   **`UPDATE`:** `owner`, `admin` ou `employee` podem atualizar endereços de clientes da sua oficina.
*   **`DELETE`:** `owner`, `admin` ou `employee` podem remover endereços de clientes da sua oficina.

```sql
-- SQL CREATE TABLE for customer_addresses
CREATE TABLE customer_addresses (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  customer_id uuid NOT NULL REFERENCES customers(id) ON DELETE CASCADE,
  zip_code text,
  street text,
  number text,
  complement text,
  district text,
  city text,
  state char(2),
  country text NOT NULL DEFAULT 'Brasil',
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for customer_addresses
ALTER TABLE customer_addresses ENABLE ROW LEVEL SECURITY;

CREATE POLICY customer_addresses_select_policy
ON customer_addresses FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id));

CREATE POLICY customer_addresses_insert_policy
ON customer_addresses FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY customer_addresses_update_policy
ON customer_addresses FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY customer_addresses_delete_policy
ON customer_addresses FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_customer_addresses
BEFORE UPDATE ON customer_addresses
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.6 `vehicles`

**Objetivo:** Armazenar informações dos veículos dos clientes de cada oficina. Cada veículo pertence a um cliente e, consequentemente, a um `tenant_id`.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do veículo. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `customer_id` | `uuid` | `FALSE` | | FK | Referência ao cliente proprietário do veículo. |
| `plate` | `text` | `FALSE` | | | Placa do veículo. |
| `brand` | `text` | `TRUE` | | | Marca do veículo (ex: Toyota, Fiat). |
| `model` | `text` | `TRUE` | | | Modelo do veículo (ex: Corolla, Uno). |
| `manufacture_year` | `integer` | `TRUE` | | | Ano de fabricação do veículo. |
| `model_year` | `integer` | `TRUE` | | | Ano do modelo do veículo. |
| `color` | `text` | `TRUE` | | | Cor predominante do veículo. |
| `renavam` | `text` | `TRUE` | | | Número do RENAVAM. |
| `chassis` | `text` | `TRUE` | | | Número do chassi. |
| `current_mileage` | `integer` | `TRUE` | `0` | | Quilometragem atual do veículo. |
| `fuel_type` | `fuel_type` | `TRUE` | | | Tipo de combustível. |
| `notes` | `text` | `TRUE` | | | Observações adicionais sobre o veículo. |
| `status` | `record_status` | `FALSE` | `active` | | Status do registro (ativo ou inativo). |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |
| `deleted_at` | `timestamptz` | `TRUE` | | | Data e hora da exclusão lógica (soft delete). |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `customer_id` REFERENCES `customers(id)` ON DELETE CASCADE
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_vehicles_tenant_id_plate` ON `vehicles` (`tenant_id`, `plate`) WHERE `deleted_at` IS NULL
*   `idx_vehicles_tenant_id_customer_id` ON `vehicles` (`tenant_id`, `customer_id`)

**Constraints:**
*   `UNIQUE (tenant_id, plate)` WHERE `deleted_at` IS NULL (Garante que a placa seja única por tenant para veículos ativos).

**Relacionamentos:**
*   `vehicles` N:1 `customers` (Muitos veículos pertencem a um cliente).
*   `vehicles` 1:N `work_orders` (Um veículo pode ter muitas ordens de serviço).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos do tenant podem visualizar veículos da sua oficina que não foram logicamente excluídos.
*   **`INSERT`:** `owner`, `admin` ou `employee` podem criar novos veículos para clientes da sua oficina.
*   **`UPDATE`:** `owner`, `admin` ou `employee` podem atualizar veículos da sua oficina que não foram logicamente excluídos.
*   **`DELETE`:** `owner` ou `admin` podem realizar o soft delete de veículos da sua oficina.

```sql
-- SQL CREATE TABLE for vehicles
CREATE TABLE vehicles (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  customer_id uuid NOT NULL REFERENCES customers(id) ON DELETE CASCADE,
  plate text NOT NULL,
  brand text,
  model text,
  manufacture_year integer,
  model_year integer,
  color text,
  renavam text,
  chassis text,
  current_mileage integer DEFAULT 0,
  fuel_type fuel_type,
  notes text,
  status record_status NOT NULL DEFAULT 'active',
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz,
  CONSTRAINT unique_plate_per_tenant UNIQUE (tenant_id, plate) WHERE (deleted_at IS NULL)
);

-- RLS for vehicles
ALTER TABLE vehicles ENABLE ROW LEVEL SECURITY;

CREATE POLICY vehicles_select_policy
ON vehicles FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL);

CREATE POLICY vehicles_insert_policy
ON vehicles FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY vehicles_update_policy
ON vehicles FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY vehicles_delete_policy
ON vehicles FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_vehicles
BEFORE UPDATE ON vehicles
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.7 `services`

**Objetivo:** Armazenar o catálogo de serviços oferecidos por cada oficina. Cada serviço pertence a um `tenant_id`.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do serviço. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `name` | `text` | `FALSE` | | | Nome do serviço. |
| `internal_code` | `text` | `TRUE` | | | Código interno do serviço (opcional). |
| `category` | `text` | `TRUE` | | | Categoria do serviço (ex: Manutenção, Elétrica). |
| `description` | `text` | `TRUE` | | | Descrição detalhada do serviço. |
| `default_price` | `numeric(12,2)` | `FALSE` | `0.00` | | Preço padrão sugerido para o serviço. |
| `estimated_minutes` | `integer` | `TRUE` | | | Tempo estimado para execução do serviço em minutos. |
| `is_active` | `boolean` | `FALSE` | `TRUE` | | Indica se o serviço está ativo e disponível para uso. |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |
| `deleted_at` | `timestamptz` | `TRUE` | | | Data e hora da exclusão lógica (soft delete). |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_services_tenant_id_name` ON `services` (`tenant_id`, `name`) WHERE `deleted_at` IS NULL
*   `idx_services_tenant_id_category` ON `services` (`tenant_id`, `category`) WHERE `deleted_at` IS NULL

**Constraints:**
*   `UNIQUE (tenant_id, name)` WHERE `deleted_at` IS NULL (Garante que o nome do serviço seja único por tenant para serviços ativos).

**Relacionamentos:**
*   `services` N:1 `tenants` (Muitos serviços pertencem a uma oficina).
*   `services` 1:N `work_order_services` (Um serviço pode estar em muitas ordens de serviço).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos do tenant podem visualizar serviços da sua oficina que não foram logicamente excluídos.
*   **`INSERT`:** `owner` ou `admin` podem criar novos serviços para sua oficina.
*   **`UPDATE`:** `owner` ou `admin` podem atualizar serviços da sua oficina que não foram logicamente excluídos.
*   **`DELETE`:** `owner` ou `admin` podem realizar o soft delete de serviços da sua oficina.

```sql
-- SQL CREATE TABLE for services
CREATE TABLE services (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  name text NOT NULL,
  internal_code text,
  category text,
  description text,
  default_price numeric(12,2) NOT NULL DEFAULT 0.00,
  estimated_minutes integer,
  is_active boolean NOT NULL DEFAULT TRUE,
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz,
  CONSTRAINT unique_service_name_per_tenant UNIQUE (tenant_id, name) WHERE (deleted_at IS NULL)
);

-- RLS for services
ALTER TABLE services ENABLE ROW LEVEL SECURITY;

CREATE POLICY services_select_policy
ON services FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL);

CREATE POLICY services_insert_policy
ON services FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY services_update_policy
ON services FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY services_delete_policy
ON services FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_services
BEFORE UPDATE ON services
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.8 `products`

**Objetivo:** Armazenar o catálogo de produtos e peças utilizados por cada oficina. Cada produto pertence a um `tenant_id`.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do produto. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `name` | `text` | `FALSE` | | | Nome do produto ou peça. |
| `sku` | `text` | `TRUE` | | | SKU ou código de barras do produto. |
| `category` | `text` | `TRUE` | | | Categoria do produto (ex: Óleo, Filtros, Pneus). |
| `description` | `text` | `TRUE` | | | Descrição detalhada do produto. |
| `unit` | `text` | `FALSE` | `un` | | Unidade de medida (ex: un, litro, kg, par). |
| `cost_price` | `numeric(12,2)` | `TRUE` | `0.00` | | Custo de aquisição do produto. |
| `sale_price` | `numeric(12,2)` | `FALSE` | `0.00` | | Preço de venda padrão do produto. |
| `stock_quantity` | `numeric(12,3)` | `TRUE` | `0.000` | | Quantidade em estoque (permite casas decimais para líquidos, etc.). |
| `min_stock_quantity` | `numeric(12,3)` | `TRUE` | `0.000` | | Quantidade mínima em estoque para alerta. |
| `is_active` | `boolean` | `FALSE` | `TRUE` | | Indica se o produto está ativo e disponível para uso. |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |
| `deleted_at` | `timestamptz` | `TRUE` | | | Data e hora da exclusão lógica (soft delete). |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_products_tenant_id_name` ON `products` (`tenant_id`, `name`) WHERE `deleted_at` IS NULL
*   `idx_products_tenant_id_sku` ON `products` (`tenant_id`, `sku`) WHERE `sku` IS NOT NULL AND `deleted_at` IS NULL
*   `idx_products_tenant_id_category` ON `products` (`tenant_id`, `category`) WHERE `deleted_at` IS NULL

**Constraints:**
*   `UNIQUE (tenant_id, name)` WHERE `deleted_at` IS NULL (Garante que o nome do produto seja único por tenant para produtos ativos).
*   `UNIQUE (tenant_id, sku)` WHERE `sku` IS NOT NULL AND `deleted_at` IS NULL (Garante que o SKU seja único por tenant para produtos ativos, se preenchido).

**Relacionamentos:**
*   `products` N:1 `tenants` (Muitos produtos pertencem a uma oficina).
*   `products` 1:N `work_order_products` (Um produto pode estar em muitas ordens de serviço).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos do tenant podem visualizar produtos da sua oficina que não foram logicamente excluídos.
*   **`INSERT`:** `owner` ou `admin` podem criar novos produtos para sua oficina.
*   **`UPDATE`:** `owner` ou `admin` podem atualizar produtos da sua oficina que não foram logicamente excluídos.
*   **`DELETE`:** `owner` ou `admin` podem realizar o soft delete de produtos da sua oficina.

```sql
-- SQL CREATE TABLE for products
CREATE TABLE products (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  name text NOT NULL,
  sku text,
  category text,
  description text,
  unit text NOT NULL DEFAULT 'un',
  cost_price numeric(12,2) DEFAULT 0.00,
  sale_price numeric(12,2) NOT NULL DEFAULT 0.00,
  stock_quantity numeric(12,3) DEFAULT 0.000,
  min_stock_quantity numeric(12,3) DEFAULT 0.000,
  is_active boolean NOT NULL DEFAULT TRUE,
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz,
  CONSTRAINT unique_product_name_per_tenant UNIQUE (tenant_id, name) WHERE (deleted_at IS NULL),
  CONSTRAINT unique_product_sku_per_tenant UNIQUE (tenant_id, sku) WHERE (sku IS NOT NULL AND deleted_at IS NULL)
);

-- RLS for products
ALTER TABLE products ENABLE ROW LEVEL SECURITY;

CREATE POLICY products_select_policy
ON products FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL);

CREATE POLICY products_insert_policy
ON products FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY products_update_policy
ON products FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY products_delete_policy
ON products FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_products
BEFORE UPDATE ON products
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.9 `work_orders`

**Objetivo:** Armazenar as informações principais das ordens de serviço. Esta é a tabela central para o fluxo operacional da oficina.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da ordem de serviço. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `number` | `bigint` | `FALSE` | | | Número sequencial da OS, único por tenant. |
| `customer_id` | `uuid` | `FALSE` | | FK | Referência ao cliente da OS. |
| `vehicle_id` | `uuid` | `FALSE` | | FK | Referência ao veículo da OS. |
| `status` | `work_order_status` | `FALSE` | `draft` | | Status atual da ordem de serviço. |
| `customer_report` | `text` | `TRUE` | | | Relato do cliente sobre o problema. |
| `diagnosis` | `text` | `TRUE` | | | Diagnóstico técnico realizado pela oficina. |
| `internal_notes` | `text` | `TRUE` | | | Observações internas da oficina. |
| `entry_mileage` | `integer` | `TRUE` | | | Quilometragem do veículo na entrada da oficina. |
| `expected_delivery_at` | `timestamptz` | `TRUE` | | | Previsão de data e hora de entrega do veículo. |
| `approved_at` | `timestamptz` | `TRUE` | | | Data e hora da aprovação da OS pelo cliente. |
| `completed_at` | `timestamptz` | `TRUE` | | | Data e hora da conclusão dos serviços. |
| `delivered_at` | `timestamptz` | `TRUE` | | | Data e hora da entrega do veículo ao cliente. |
| `assigned_to` | `uuid` | `TRUE` | | FK | Usuário responsável técnico pela OS. |
| `services_total` | `numeric(12,2)` | `FALSE` | `0.00` | | Soma total dos valores dos serviços. |
| `products_total` | `numeric(12,2)` | `FALSE` | `0.00` | | Soma total dos valores dos produtos. |
| `discount_total` | `numeric(12,2)` | `FALSE` | `0.00` | | Total de descontos aplicados na OS. |
| `additional_total` | `numeric(12,2)` | `FALSE` | `0.00` | | Total de acréscimos aplicados na OS. |
| `grand_total` | `numeric(12,2)` | `FALSE` | `0.00` | | Valor total final da OS. |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |
| `deleted_at` | `timestamptz` | `TRUE` | | | Data e hora da exclusão lógica (soft delete). |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `customer_id` REFERENCES `customers(id)` ON DELETE RESTRICT
*   `vehicle_id` REFERENCES `vehicles(id)` ON DELETE RESTRICT
*   `assigned_to` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_work_orders_tenant_id_number` ON `work_orders` (`tenant_id`, `number`) WHERE `deleted_at` IS NULL
*   `idx_work_orders_tenant_id_customer_id` ON `work_orders` (`tenant_id`, `customer_id`)
*   `idx_work_orders_tenant_id_vehicle_id` ON `work_orders` (`tenant_id`, `vehicle_id`)
*   `idx_work_orders_tenant_id_status` ON `work_orders` (`tenant_id`, `status`)

**Constraints:**
*   `UNIQUE (tenant_id, number)` WHERE `deleted_at` IS NULL (Garante que o número da OS seja único por tenant para OS ativas).

**Relacionamentos:**
*   `work_orders` N:1 `tenants` (Muitas OS pertencem a uma oficina).
*   `work_orders` N:1 `customers` (Muitas OS são para um cliente).
*   `work_orders` N:1 `vehicles` (Muitas OS são para um veículo).
*   `work_orders` 1:N `work_order_services` (Uma OS tem muitos serviços).
*   `work_orders` 1:N `work_order_products` (Uma OS tem muitos produtos).
*   `work_orders` 1:N `work_order_history` (Uma OS tem um histórico de status).
*   `work_orders` 1:N `accounts_receivable` (Uma OS pode gerar muitas contas a receber).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos do tenant podem visualizar ordens de serviço da sua oficina que não foram logicamente excluídas.
*   **`INSERT`:** `owner`, `admin` ou `employee` podem criar novas ordens de serviço para sua oficina.
*   **`UPDATE`:** `owner`, `admin` ou `employee` podem atualizar ordens de serviço da sua oficina que não foram logicamente excluídas, respeitando as regras de transição de status.
*   **`DELETE`:** `owner` ou `admin` podem realizar o soft delete de ordens de serviço da sua oficina.

```sql
-- SQL CREATE TABLE for work_orders
CREATE TABLE work_orders (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  number bigint NOT NULL,
  customer_id uuid NOT NULL REFERENCES customers(id) ON DELETE RESTRICT,
  vehicle_id uuid NOT NULL REFERENCES vehicles(id) ON DELETE RESTRICT,
  status work_order_status NOT NULL DEFAULT 'draft',
  customer_report text,
  diagnosis text,
  internal_notes text,
  entry_mileage integer,
  expected_delivery_at timestamptz,
  approved_at timestamptz,
  completed_at timestamptz,
  delivered_at timestamptz,
  assigned_to uuid REFERENCES profiles(id) ON DELETE SET NULL,
  services_total numeric(12,2) NOT NULL DEFAULT 0.00,
  products_total numeric(12,2) NOT NULL DEFAULT 0.00,
  discount_total numeric(12,2) NOT NULL DEFAULT 0.00,
  additional_total numeric(12,2) NOT NULL DEFAULT 0.00,
  grand_total numeric(12,2) NOT NULL DEFAULT 0.00,
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz,
  CONSTRAINT unique_work_order_number_per_tenant UNIQUE (tenant_id, number) WHERE (deleted_at IS NULL)
);

-- RLS for work_orders
ALTER TABLE work_orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY work_orders_select_policy
ON work_orders FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL);

CREATE POLICY work_orders_insert_policy
ON work_orders FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY work_orders_update_policy
ON work_orders FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY work_orders_delete_policy
ON work_orders FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_work_orders
BEFORE UPDATE ON work_orders
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

-- Trigger para gerar número sequencial da OS (implementação detalhada na seção de Triggers)
CREATE TRIGGER generate_work_order_number_trigger
BEFORE INSERT ON work_orders
FOR EACH ROW EXECUTE FUNCTION generate_work_order_number();


### 3.10 `work_order_services`

**Objetivo:** Armazenar os serviços associados a cada ordem de serviço, com seus detalhes (preço, quantidade, desconto) no momento da inclusão para preservar o histórico.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do item de serviço na OS. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `work_order_id` | `uuid` | `FALSE` | | FK | Referência à ordem de serviço. |
| `service_id` | `uuid` | `TRUE` | | FK | Referência ao serviço do catálogo (opcional, se for um serviço avulso). |
| `description` | `text` | `FALSE` | | | Descrição do serviço (congelada no momento da inclusão). |
| `quantity` | `numeric(12,3)` | `FALSE` | `1.000` | | Quantidade do serviço. |
| `unit_price` | `numeric(12,2)` | `FALSE` | `0.00` | | Preço unitário do serviço (congelado). |
| `discount` | `numeric(12,2)` | `FALSE` | `0.00` | | Desconto aplicado a este item de serviço. |
| `total` | `numeric(12,2)` | `FALSE` | `0.00` | | Total do item de serviço (quantidade * preço - desconto). |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `work_order_id` REFERENCES `work_orders(id)` ON DELETE CASCADE
*   `service_id` REFERENCES `services(id)` ON DELETE SET NULL
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_work_order_services_tenant_id_work_order_id` ON `work_order_services` (`tenant_id`, `work_order_id`)

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `work_order_services` N:1 `work_orders` (Muitos itens de serviço pertencem a uma OS).
*   `work_order_services` N:1 `services` (Muitos itens de serviço se referem a um serviço do catálogo).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos do tenant podem visualizar serviços de ordens de serviço da sua oficina.
*   **`INSERT`:** `owner`, `admin` ou `employee` podem adicionar serviços a ordens de serviço da sua oficina.
*   **`UPDATE`:** `owner`, `admin` ou `employee` podem atualizar serviços de ordens de serviço da sua oficina.
*   **`DELETE`:** `owner`, `admin` ou `employee` podem remover serviços de ordens de serviço da sua oficina.

```sql
-- SQL CREATE TABLE for work_order_services
CREATE TABLE work_order_services (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  work_order_id uuid NOT NULL REFERENCES work_orders(id) ON DELETE CASCADE,
  service_id uuid REFERENCES services(id) ON DELETE SET NULL,
  description text NOT NULL,
  quantity numeric(12,3) NOT NULL DEFAULT 1.000,
  unit_price numeric(12,2) NOT NULL DEFAULT 0.00,
  discount numeric(12,2) NOT NULL DEFAULT 0.00,
  total numeric(12,2) NOT NULL DEFAULT 0.00,
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for work_order_services
ALTER TABLE work_order_services ENABLE ROW LEVEL SECURITY;

CREATE POLICY work_order_services_select_policy
ON work_order_services FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id));

CREATE POLICY work_order_services_insert_policy
ON work_order_services FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY work_order_services_update_policy
ON work_order_services FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY work_order_services_delete_policy
ON work_order_services FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_work_order_services
BEFORE UPDATE ON work_order_services
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.11 `work_order_products`

**Objetivo:** Armazenar os produtos/peças associados a cada ordem de serviço, com seus detalhes (preço, quantidade, custo, desconto) no momento da inclusão para preservar o histórico.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do item de produto na OS. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `work_order_id` | `uuid` | `FALSE` | | FK | Referência à ordem de serviço. |
| `product_id` | `uuid` | `TRUE` | | FK | Referência ao produto do catálogo (opcional, se for um produto avulso). |
| `description` | `text` | `FALSE` | | | Descrição do produto (congelada no momento da inclusão). |
| `quantity` | `numeric(12,3)` | `FALSE` | `1.000` | | Quantidade do produto. |
| `unit_price` | `numeric(12,2)` | `FALSE` | `0.00` | | Preço unitário do produto (congelado). |
| `cost_price` | `numeric(12,2)` | `TRUE` | `0.00` | | Custo unitário do produto (congelado). |
| `discount` | `numeric(12,2)` | `FALSE` | `0.00` | | Desconto aplicado a este item de produto. |
| `total` | `numeric(12,2)` | `FALSE` | `0.00` | | Total do item de produto (quantidade * preço - desconto). |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `work_order_id` REFERENCES `work_orders(id)` ON DELETE CASCADE
*   `product_id` REFERENCES `products(id)` ON DELETE SET NULL
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_work_order_products_tenant_id_work_order_id` ON `work_order_products` (`tenant_id`, `work_order_id`)

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `work_order_products` N:1 `work_orders` (Muitos itens de produto pertencem a uma OS).
*   `work_order_products` N:1 `products` (Muitos itens de produto se referem a um produto do catálogo).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos do tenant podem visualizar produtos de ordens de serviço da sua oficina.
*   **`INSERT`:** `owner`, `admin` ou `employee` podem adicionar produtos a ordens de serviço da sua oficina.
*   **`UPDATE`:** `owner`, `admin` ou `employee` podem atualizar produtos de ordens de serviço da sua oficina.
*   **`DELETE`:** `owner`, `admin` ou `employee` podem remover produtos de ordens de serviço da sua oficina.

```sql
-- SQL CREATE TABLE for work_order_products
CREATE TABLE work_order_products (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  work_order_id uuid NOT NULL REFERENCES work_orders(id) ON DELETE CASCADE,
  product_id uuid REFERENCES products(id) ON DELETE SET NULL,
  description text NOT NULL,
  quantity numeric(12,3) NOT NULL DEFAULT 1.000,
  unit_price numeric(12,2) NOT NULL DEFAULT 0.00,
  cost_price numeric(12,2) DEFAULT 0.00,
  discount numeric(12,2) NOT NULL DEFAULT 0.00,
  total numeric(12,2) NOT NULL DEFAULT 0.00,
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for work_order_products
ALTER TABLE work_order_products ENABLE ROW LEVEL SECURITY;

CREATE POLICY work_order_products_select_policy
ON work_order_products FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id));

CREATE POLICY work_order_products_insert_policy
ON work_order_products FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY work_order_products_update_policy
ON work_order_products FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

CREATE POLICY work_order_products_delete_policy
ON work_order_products FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_work_order_products
BEFORE UPDATE ON work_order_products
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.12 `work_order_history`

**Objetivo:** Registrar o histórico de mudanças de status de cada ordem de serviço, incluindo quem fez a alteração e a justificativa.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do registro de histórico. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `work_order_id` | `uuid` | `FALSE` | | FK | Referência à ordem de serviço. |
| `from_status` | `work_order_status` | `TRUE` | | | Status anterior da OS. |
| `to_status` | `work_order_status` | `FALSE` | | | Novo status da OS. |
| `changed_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que realizou a mudança de status. |
| `notes` | `text` | `TRUE` | | | Justificativa ou observação sobre a mudança. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da mudança de status. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `work_order_id` REFERENCES `work_orders(id)` ON DELETE CASCADE
*   `changed_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_work_order_history_tenant_id_work_order_id` ON `work_order_history` (`tenant_id`, `work_order_id`)

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `work_order_history` N:1 `work_orders` (Muitos registros de histórico pertencem a uma OS).

**Políticas RLS:**
*   **`SELECT`:** Membros ativos do tenant podem visualizar o histórico de status de ordens de serviço da sua oficina.
*   **`INSERT`:** `owner`, `admin` ou `employee` podem registrar mudanças de status (geralmente via trigger ou função).
*   **`UPDATE`:** Não permitido (histórico deve ser imutável).
*   **`DELETE`:** Não permitido (histórico deve ser imutável).

```sql
-- SQL CREATE TABLE for work_order_history
CREATE TABLE work_order_history (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  work_order_id uuid NOT NULL REFERENCES work_orders(id) ON DELETE CASCADE,
  from_status work_order_status,
  to_status work_order_status NOT NULL,
  changed_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  notes text,
  created_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for work_order_history
ALTER TABLE work_order_history ENABLE ROW LEVEL SECURITY;

CREATE POLICY work_order_history_select_policy
ON work_order_history FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id));

CREATE POLICY work_order_history_insert_policy
ON work_order_history FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[]));

-- UPDATE e DELETE não são permitidos para histórico


### 3.13 `accounts_receivable`

**Objetivo:** Gerenciar as contas a receber de cada oficina, geralmente geradas a partir de ordens de serviço finalizadas.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da conta a receber. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `customer_id` | `uuid` | `FALSE` | | FK | Referência ao cliente devedor. |
| `work_order_id` | `uuid` | `TRUE` | | FK | Referência à OS que gerou a conta (opcional). |
| `description` | `text` | `FALSE` | | | Descrição da conta a receber. |
| `amount` | `numeric(12,2)` | `FALSE` | `0.00` | | Valor total a receber. |
| `due_date` | `date` | `FALSE` | | | Data de vencimento da conta. |
| `paid_at` | `timestamptz` | `TRUE` | | | Data e hora do pagamento da conta. |
| `status` | `financial_status` | `FALSE` | `pending` | | Status da conta (pending, paid, overdue, cancelled). |
| `payment_method` | `payment_method` | `TRUE` | | | Método de pagamento utilizado (se pago). |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |
| `deleted_at` | `timestamptz` | `TRUE` | | | Data e hora da exclusão lógica (soft delete). |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `customer_id` REFERENCES `customers(id)` ON DELETE RESTRICT
*   `work_order_id` REFERENCES `work_orders(id)` ON DELETE SET NULL
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_accounts_receivable_tenant_id_due_date` ON `accounts_receivable` (`tenant_id`, `due_date`) WHERE `deleted_at` IS NULL
*   `idx_accounts_receivable_tenant_id_customer_id` ON `accounts_receivable` (`tenant_id`, `customer_id`) WHERE `deleted_at` IS NULL
*   `idx_accounts_receivable_tenant_id_status` ON `accounts_receivable` (`tenant_id`, `status`) WHERE `deleted_at` IS NULL

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `accounts_receivable` N:1 `tenants` (Muitas contas a receber pertencem a uma oficina).
*   `accounts_receivable` N:1 `customers` (Muitas contas a receber são de um cliente).
*   `accounts_receivable` N:1 `work_orders` (Muitas contas a receber podem ser geradas por uma OS).

**Políticas RLS:**
*   **`SELECT`:** `owner` ou `admin` podem visualizar contas a receber da sua oficina que não foram logicamente excluídas.
*   **`INSERT`:** `owner` ou `admin` podem criar novas contas a receber para sua oficina.
*   **`UPDATE`:** `owner` ou `admin` podem atualizar contas a receber da sua oficina que não foram logicamente excluídas.
*   **`DELETE`:** `owner` ou `admin` podem realizar o soft delete de contas a receber da sua oficina.

```sql
-- SQL CREATE TABLE for accounts_receivable
CREATE TABLE accounts_receivable (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  customer_id uuid NOT NULL REFERENCES customers(id) ON DELETE RESTRICT,
  work_order_id uuid REFERENCES work_orders(id) ON DELETE SET NULL,
  description text NOT NULL,
  amount numeric(12,2) NOT NULL DEFAULT 0.00,
  due_date date NOT NULL,
  paid_at timestamptz,
  status financial_status NOT NULL DEFAULT 'pending',
  payment_method payment_method,
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz
);

-- RLS for accounts_receivable
ALTER TABLE accounts_receivable ENABLE ROW LEVEL SECURITY;

CREATE POLICY accounts_receivable_select_policy
ON accounts_receivable FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY accounts_receivable_insert_policy
ON accounts_receivable FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY accounts_receivable_update_policy
ON accounts_receivable FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY accounts_receivable_delete_policy
ON accounts_receivable FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_accounts_receivable
BEFORE UPDATE ON accounts_receivable
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.14 `accounts_payable`

**Objetivo:** Gerenciar as contas a pagar de cada oficina, registrando despesas com fornecedores, aluguel, etc.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da conta a pagar. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `supplier_name` | `text` | `TRUE` | | | Nome do fornecedor ou credor. |
| `category` | `text` | `TRUE` | | | Categoria da despesa (ex: Aluguel, Peças, Salários). |
| `description` | `text` | `FALSE` | | | Descrição da conta a pagar. |
| `amount` | `numeric(12,2)` | `FALSE` | `0.00` | | Valor total a pagar. |
| `due_date` | `date` | `FALSE` | | | Data de vencimento da conta. |
| `paid_at` | `timestamptz` | `TRUE` | | | Data e hora do pagamento da conta. |
| `status` | `financial_status` | `FALSE` | `pending` | | Status da conta (pending, paid, overdue, cancelled). |
| `payment_method` | `payment_method` | `TRUE` | | | Método de pagamento utilizado (se pago). |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |
| `deleted_at` | `timestamptz` | `TRUE` | | | Data e hora da exclusão lógica (soft delete). |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_accounts_payable_tenant_id_due_date` ON `accounts_payable` (`tenant_id`, `due_date`) WHERE `deleted_at` IS NULL
*   `idx_accounts_payable_tenant_id_category` ON `accounts_payable` (`tenant_id`, `category`) WHERE `deleted_at` IS NULL
*   `idx_accounts_payable_tenant_id_status` ON `accounts_payable` (`tenant_id`, `status`) WHERE `deleted_at` IS NULL

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `accounts_payable` N:1 `tenants` (Muitas contas a pagar pertencem a uma oficina).

**Políticas RLS:**
*   **`SELECT`:** `owner` ou `admin` podem visualizar contas a pagar da sua oficina que não foram logicamente excluídas.
*   **`INSERT`:** `owner` ou `admin` podem criar novas contas a pagar para sua oficina.
*   **`UPDATE`:** `owner` ou `admin` podem atualizar contas a pagar da sua oficina que não foram logicamente excluídas.
*   **`DELETE`:** `owner` ou `admin` podem realizar o soft delete de contas a pagar da sua oficina.

```sql
-- SQL CREATE TABLE for accounts_payable
CREATE TABLE accounts_payable (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  supplier_name text,
  category text,
  description text NOT NULL,
  amount numeric(12,2) NOT NULL DEFAULT 0.00,
  due_date date NOT NULL,
  paid_at timestamptz,
  status financial_status NOT NULL DEFAULT 'pending',
  payment_method payment_method,
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz
);

-- RLS for accounts_payable
ALTER TABLE accounts_payable ENABLE ROW LEVEL SECURITY;

CREATE POLICY accounts_payable_select_policy
ON accounts_payable FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY accounts_payable_insert_policy
ON accounts_payable FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY accounts_payable_update_policy
ON accounts_payable FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]))
WITH CHECK (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY accounts_payable_delete_policy
ON accounts_payable FOR DELETE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_accounts_payable
BEFORE UPDATE ON accounts_payable
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.15 `financial_transactions`

**Objetivo:** Registrar todas as transações financeiras (receitas e despesas) que ocorrem na oficina, para fins de fluxo de caixa e auditoria.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da transação financeira. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `type` | `transaction_type` | `FALSE` | | | Tipo da transação (income ou expense). |
| `source` | `text` | `TRUE` | | | Origem da transação (ex: OS, manual, ajuste, assinatura). |
| `source_id` | `uuid` | `TRUE` | | | ID da entidade de origem (ex: work_order_id, subscription_id). |
| `description` | `text` | `FALSE` | | | Descrição da transação. |
| `amount` | `numeric(12,2)` | `FALSE` | `0.00` | | Valor da transação. |
| `transaction_date` | `date` | `FALSE` | `now()` | | Data em que a transação ocorreu (data de caixa). |
| `payment_method` | `payment_method` | `TRUE` | | | Método de pagamento/recebimento. |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_financial_transactions_tenant_id_date` ON `financial_transactions` (`tenant_id`, `transaction_date`)
*   `idx_financial_transactions_tenant_id_type` ON `financial_transactions` (`tenant_id`, `type`)

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `financial_transactions` N:1 `tenants` (Muitas transações pertencem a uma oficina).

**Políticas RLS:**
*   **`SELECT`:** `owner` ou `admin` podem visualizar transações financeiras da sua oficina.
*   **`INSERT`:** `owner` ou `admin` podem criar novas transações financeiras para sua oficina.
*   **`UPDATE`:** Não permitido (transações financeiras devem ser imutáveis, apenas estornadas ou ajustadas com novas transações).
*   **`DELETE`:** Não permitido (transações financeiras devem ser imutáveis).

```sql
-- SQL CREATE TABLE for financial_transactions
CREATE TABLE financial_transactions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  type transaction_type NOT NULL,
  source text,
  source_id uuid,
  description text NOT NULL,
  amount numeric(12,2) NOT NULL DEFAULT 0.00,
  transaction_date date NOT NULL DEFAULT now(),
  payment_method payment_method,
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for financial_transactions
ALTER TABLE financial_transactions ENABLE ROW LEVEL SECURITY;

CREATE POLICY financial_transactions_select_policy
ON financial_transactions FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY financial_transactions_insert_policy
ON financial_transactions FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

-- UPDATE e DELETE não são permitidos para transações financeiras


### 3.16 `subscriptions`

**Objetivo:** Armazenar informações sobre a assinatura de cada oficina com o OficinaPro, incluindo o plano, status e IDs da Stripe.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da assinatura. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `plan_id` | `uuid` | `TRUE` | | FK | Referência ao plano de assinatura interno (se houver). |
| `stripe_customer_id` | `text` | `FALSE` | | | ID do cliente na Stripe. |
| `stripe_subscription_id` | `text` | `FALSE` | | | ID da assinatura na Stripe. |
| `status` | `subscription_status` | `FALSE` | `trialing` | | Status da assinatura (trialing, active, canceled, etc.). |
| `trial_ends_at` | `timestamptz` | `TRUE` | | | Data e hora de término do período de trial. |
| `current_period_start` | `timestamptz` | `TRUE` | | | Início do período de cobrança atual. |
| `current_period_end` | `timestamptz` | `TRUE` | | | Fim do período de cobrança atual. |
| `cancel_at_period_end` | `boolean` | `FALSE` | `FALSE` | | Indica se o cancelamento está agendado para o fim do período. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `plan_id` REFERENCES `subscription_plans(id)` ON DELETE SET NULL

**Índices:**
*   `idx_subscriptions_tenant_id` ON `subscriptions` (`tenant_id`)
*   `idx_subscriptions_stripe_customer_id` ON `subscriptions` (`stripe_customer_id`)
*   `idx_subscriptions_stripe_subscription_id` ON `subscriptions` (`stripe_subscription_id`)

**Constraints:**
*   `UNIQUE (tenant_id)` (Uma oficina só pode ter uma assinatura ativa).
*   `UNIQUE (stripe_subscription_id)`

**Relacionamentos:**
*   `subscriptions` N:1 `tenants` (Muitas assinaturas pertencem a uma oficina).
*   `subscriptions` N:1 `subscription_plans` (Muitas assinaturas se referem a um plano).

**Políticas RLS:**
*   **`SELECT`:** Apenas `owner` ou `admin` do tenant podem visualizar a assinatura da sua oficina.
*   **`INSERT`:** Apenas `owner` podem criar uma nova assinatura para sua oficina.
*   **`UPDATE`:** Apenas `owner` podem atualizar a assinatura da sua oficina (geralmente via webhooks da Stripe).
*   **`DELETE`:** Não permitido (assinaturas são canceladas, não excluídas).

```sql
-- SQL CREATE TABLE for subscriptions
CREATE TABLE subscriptions (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE UNIQUE,
  plan_id uuid REFERENCES subscription_plans(id) ON DELETE SET NULL,
  stripe_customer_id text NOT NULL,
  stripe_subscription_id text NOT NULL UNIQUE,
  status subscription_status NOT NULL DEFAULT 'trialing',
  trial_ends_at timestamptz,
  current_period_start timestamptz,
  current_period_end timestamptz,
  cancel_at_period_end boolean NOT NULL DEFAULT FALSE,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for subscriptions
ALTER TABLE subscriptions ENABLE ROW LEVEL SECURITY;

CREATE POLICY subscriptions_select_policy
ON subscriptions FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY subscriptions_insert_policy
ON subscriptions FOR INSERT
TO authenticated
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner']::tenant_role[]));

CREATE POLICY subscriptions_update_policy
ON subscriptions FOR UPDATE
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner']::tenant_role[]))
WITH CHECK (has_tenant_role(tenant_id, ARRAY['owner']::tenant_role[]));

-- DELETE não é permitido para assinaturas

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_subscriptions
BEFORE UPDATE ON subscriptions
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.17 `subscription_plans`

**Objetivo:** Armazenar os planos de assinatura disponíveis no sistema. Esta tabela é global (não multi-tenant) e define os planos que as oficinas podem contratar.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do plano. |
| `name` | `text` | `FALSE` | | | Nome do plano (ex: Básico, Pro, Premium). |
| `description` | `text` | `TRUE` | | | Descrição detalhada do plano. |
| `billing_interval` | `billing_interval` | `FALSE` | | | Intervalo de cobrança (monthly, yearly). |
| `price` | `numeric(12,2)` | `FALSE` | `0.00` | | Preço do plano. |
| `stripe_price_id` | `text` | `FALSE` | | | ID do preço correspondente na Stripe. |
| `features` | `jsonb` | `TRUE` | `[]` | | Lista de recursos/limites incluídos no plano. |
| `is_active` | `boolean` | `FALSE` | `TRUE` | | Indica se o plano está ativo e disponível para contratação. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:** Nenhuma.

**Índices:**
*   `idx_subscription_plans_stripe_price_id` ON `subscription_plans` (`stripe_price_id`)

**Constraints:**
*   `UNIQUE (stripe_price_id)`

**Relacionamentos:**
*   `subscription_plans` 1:N `subscriptions` (Um plano pode ter muitas assinaturas).

**Políticas RLS:**
*   **`SELECT`:** Todos os usuários autenticados podem visualizar os planos disponíveis.
*   **`INSERT`:** Apenas `service_role` ou `admin` (via backend) podem criar novos planos.
*   **`UPDATE`:** Apenas `service_role` ou `admin` (via backend) podem atualizar planos.
*   **`DELETE`:** Não permitido (planos são inativados, não excluídos).

```sql
-- SQL CREATE TABLE for subscription_plans
CREATE TABLE subscription_plans (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  description text,
  billing_interval billing_interval NOT NULL,
  price numeric(12,2) NOT NULL DEFAULT 0.00,
  stripe_price_id text NOT NULL UNIQUE,
  features jsonb DEFAULT '[]'::jsonb,
  is_active boolean NOT NULL DEFAULT TRUE,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for subscription_plans (Global, visível para todos autenticados)
ALTER TABLE subscription_plans ENABLE ROW LEVEL SECURITY;

CREATE POLICY subscription_plans_select_policy
ON subscription_plans FOR SELECT
TO authenticated
USING (TRUE);

CREATE POLICY subscription_plans_insert_policy
ON subscription_plans FOR INSERT
TO service_role
WITH CHECK (TRUE); -- Apenas service_role pode inserir

CREATE POLICY subscription_plans_update_policy
ON subscription_plans FOR UPDATE
TO service_role
USING (TRUE)
WITH CHECK (TRUE); -- Apenas service_role pode atualizar

-- DELETE não é permitido para planos

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_subscription_plans
BEFORE UPDATE ON subscription_plans
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.18 `invoices`

**Objetivo:** Armazenar registros de faturas geradas, principalmente as relacionadas a assinaturas da Stripe. Pode ser usado para faturas de OS também.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da fatura. |
| `tenant_id` | `uuid` | `FALSE` | | FK | Referência à oficina proprietária. |
| `subscription_id` | `uuid` | `TRUE` | | FK | Referência à assinatura (se for fatura de assinatura). |
| `work_order_id` | `uuid` | `TRUE` | | FK | Referência à OS (se for fatura de OS). |
| `stripe_invoice_id` | `text` | `TRUE` | | | ID da fatura na Stripe. |
| `invoice_number` | `text` | `TRUE` | | | Número da fatura (interno ou externo). |
| `amount` | `numeric(12,2)` | `FALSE` | `0.00` | | Valor total da fatura. |
| `currency` | `char(3)` | `FALSE` | `BRL` | | Moeda da fatura (ex: BRL, USD). |
| `due_date` | `date` | `TRUE` | | | Data de vencimento da fatura. |
| `paid_at` | `timestamptz` | `TRUE` | | | Data e hora do pagamento da fatura. |
| `status` | `financial_status` | `FALSE` | `pending` | | Status da fatura (pending, paid, overdue, cancelled). |
| `invoice_pdf_url` | `text` | `TRUE` | | | URL do PDF da fatura (se gerado). |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `subscription_id` REFERENCES `subscriptions(id)` ON DELETE SET NULL
*   `work_order_id` REFERENCES `work_orders(id)` ON DELETE SET NULL

**Índices:**
*   `idx_invoices_tenant_id_due_date` ON `invoices` (`tenant_id`, `due_date`)
*   `idx_invoices_stripe_invoice_id` ON `invoices` (`stripe_invoice_id`)

**Constraints:**
*   `UNIQUE (stripe_invoice_id)` WHERE `stripe_invoice_id` IS NOT NULL

**Relacionamentos:**
*   `invoices` N:1 `tenants` (Muitas faturas pertencem a uma oficina).
*   `invoices` N:1 `subscriptions` (Muitas faturas podem ser de uma assinatura).
*   `invoices` N:1 `work_orders` (Muitas faturas podem ser de uma OS).

**Políticas RLS:**
*   **`SELECT`:** Apenas `owner` ou `admin` do tenant podem visualizar faturas da sua oficina.
*   **`INSERT`:** Apenas `service_role` ou `admin` (via backend/webhooks) podem criar faturas.
*   **`UPDATE`:** Apenas `service_role` ou `admin` (via backend/webhooks) podem atualizar faturas.
*   **`DELETE`:** Não permitido (faturas são canceladas, não excluídas).

```sql
-- SQL CREATE TABLE for invoices
CREATE TABLE invoices (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  subscription_id uuid REFERENCES subscriptions(id) ON DELETE SET NULL,
  work_order_id uuid REFERENCES work_orders(id) ON DELETE SET NULL,
  stripe_invoice_id text UNIQUE,
  invoice_number text,
  amount numeric(12,2) NOT NULL DEFAULT 0.00,
  currency char(3) NOT NULL DEFAULT 'BRL',
  due_date date,
  paid_at timestamptz,
  status financial_status NOT NULL DEFAULT 'pending',
  invoice_pdf_url text,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for invoices
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;

CREATE POLICY invoices_select_policy
ON invoices FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[]));

CREATE POLICY invoices_insert_policy
ON invoices FOR INSERT
TO service_role
WITH CHECK (TRUE); -- Apenas service_role pode inserir

CREATE POLICY invoices_update_policy
ON invoices FOR UPDATE
TO service_role
USING (TRUE)
WITH CHECK (TRUE); -- Apenas service_role pode atualizar

-- DELETE não é permitido para faturas

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_invoices
BEFORE UPDATE ON invoices
FOR EACH ROW EXECUTE FUNCTION set_updated_at();


### 3.19 `audit_logs`

**Objetivo:** Registrar ações importantes realizadas no sistema para fins de auditoria, segurança e rastreabilidade.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único do log de auditoria. |
| `tenant_id` | `uuid` | `TRUE` | | FK | Referência à oficina (se a ação for contextual a um tenant). |
| `user_id` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que realizou a ação. |
| `action` | `text` | `FALSE` | | | Descrição da ação (ex: 'customer_created', 'work_order_status_updated'). |
| `entity` | `text` | `TRUE` | | | Nome da entidade afetada (ex: 'customers', 'work_orders'). |
| `entity_id` | `uuid` | `TRUE` | | | ID da entidade afetada. |
| `old_value` | `jsonb` | `TRUE` | | | Estado anterior do registro (para updates). |
| `new_value` | `jsonb` | `TRUE` | | | Novo estado do registro (para inserts/updates). |
| `metadata` | `jsonb` | `TRUE` | `{} ` | | Metadados adicionais da ação. |
| `ip_address` | `inet` | `TRUE` | | | Endereço IP de onde a ação foi originada. |
| `user_agent` | `text` | `TRUE` | | | User agent do navegador/cliente. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da ação. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE SET NULL
*   `user_id` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_audit_logs_tenant_id_created_at` ON `audit_logs` (`tenant_id`, `created_at`)
*   `idx_audit_logs_user_id` ON `audit_logs` (`user_id`)
*   `idx_audit_logs_entity_id` ON `audit_logs` (`entity_id`)

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `audit_logs` N:1 `tenants` (Muitos logs podem estar relacionados a uma oficina).
*   `audit_logs` N:1 `profiles` (Muitos logs são gerados por um usuário).

**Políticas RLS:**
*   **`SELECT`:** Apenas `owner` ou `admin` do tenant podem visualizar logs de auditoria da sua oficina. Logs globais (sem `tenant_id`) podem ser acessados apenas por `service_role`.
*   **`INSERT`:** Apenas `service_role` ou funções/triggers podem inserir logs de auditoria.
*   **`UPDATE`:** Não permitido (logs de auditoria são imutáveis).
*   **`DELETE`:** Não permitido (logs de auditoria são imutáveis).

```sql
-- SQL CREATE TABLE for audit_logs
CREATE TABLE audit_logs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid REFERENCES tenants(id) ON DELETE SET NULL,
  user_id uuid REFERENCES profiles(id) ON DELETE SET NULL,
  action text NOT NULL,
  entity text,
  entity_id uuid,
  old_value jsonb,
  new_value jsonb,
  metadata jsonb DEFAULT '{}'::jsonb,
  ip_address inet,
  user_agent text,
  created_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for audit_logs
ALTER TABLE audit_logs ENABLE ROW LEVEL SECURITY;

CREATE POLICY audit_logs_select_policy
ON audit_logs FOR SELECT
TO authenticated
USING (tenant_id IS NULL OR (is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[])));

CREATE POLICY audit_logs_insert_policy
ON audit_logs FOR INSERT
TO service_role
WITH CHECK (TRUE); -- Apenas service_role pode inserir

-- UPDATE e DELETE não são permitidos para logs de auditoria


### 3.20 `notifications`

**Objetivo:** Armazenar notificações para os usuários, que podem ser específicas de um tenant ou globais.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da notificação. |
| `tenant_id` | `uuid` | `TRUE` | | FK | Referência à oficina (se a notificação for específica de um tenant). |
| `user_id` | `uuid` | `FALSE` | | FK | Usuário destinatário da notificação. |
| `type` | `notification_type` | `FALSE` | `info` | | Tipo da notificação (system, alert, info, warning). |
| `title` | `text` | `FALSE` | | | Título da notificação. |
| `message` | `text` | `FALSE` | | | Conteúdo da mensagem da notificação. |
| `link` | `text` | `TRUE` | | | URL para onde a notificação direciona. |
| `status` | `notification_status` | `FALSE` | `unread` | | Status da notificação (unread, read, archived). |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação da notificação. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `user_id` REFERENCES `profiles(id)` ON DELETE CASCADE

**Índices:**
*   `idx_notifications_user_id_status` ON `notifications` (`user_id`, `status`)
*   `idx_notifications_tenant_id_user_id` ON `notifications` (`tenant_id`, `user_id`)

**Constraints:** Nenhuma adicional.

**Relacionamentos:**
*   `notifications` N:1 `tenants` (Muitas notificações podem ser para uma oficina).
*   `notifications` N:1 `profiles` (Muitas notificações são para um usuário).

**Políticas RLS:**
*   **`SELECT`:** Usuários podem visualizar suas próprias notificações. `owner` ou `admin` podem visualizar notificações de outros usuários do seu tenant.
*   **`INSERT`:** Apenas `service_role` ou funções/triggers podem criar notificações.
*   **`UPDATE`:** Usuários podem marcar suas próprias notificações como lidas/arquivadas. `owner` ou `admin` podem atualizar notificações de outros usuários do seu tenant.
*   **`DELETE`:** Não permitido (notificações são arquivadas, não excluídas).

```sql
-- SQL CREATE TABLE for notifications
CREATE TABLE notifications (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid REFERENCES tenants(id) ON DELETE CASCADE,
  user_id uuid NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  type notification_type NOT NULL DEFAULT 'info',
  title text NOT NULL,
  message text NOT NULL,
  link text,
  status notification_status NOT NULL DEFAULT 'unread',
  created_at timestamptz NOT NULL DEFAULT now()
);

-- RLS for notifications
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;

CREATE POLICY notifications_select_policy
ON notifications FOR SELECT
TO authenticated
USING (user_id = auth.uid() OR (tenant_id IS NOT NULL AND is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[])));

CREATE POLICY notifications_insert_policy
ON notifications FOR INSERT
TO service_role
WITH CHECK (TRUE); -- Apenas service_role pode inserir

CREATE POLICY notifications_update_policy
ON notifications FOR UPDATE
TO authenticated
USING (user_id = auth.uid() OR (tenant_id IS NOT NULL AND is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[])))
WITH CHECK (user_id = auth.uid() OR (tenant_id IS NOT NULL AND is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[])));

-- DELETE não é permitido para notificações


### 3.21 `settings`

**Objetivo:** Armazenar configurações específicas de cada oficina ou configurações globais do sistema.

| Campo | Tipo de Dado | Nullable | Valor Padrão | Chave | Descrição |
|---|---|---|---|---|---|
| `id` | `uuid` | `FALSE` | `gen_random_uuid()` | PK | Identificador único da configuração. |
| `tenant_id` | `uuid` | `TRUE` | | FK | Referência à oficina (se a configuração for específica de um tenant). |
| `key` | `text` | `FALSE` | | | Chave da configuração (ex: 'work_order_prefix', 'currency_symbol'). |
| `value` | `text` | `TRUE` | | | Valor da configuração. |
| `type` | `setting_type` | `FALSE` | `general` | | Tipo da configuração (general, financial, operational). |
| `is_public` | `boolean` | `FALSE` | `FALSE` | | Indica se a configuração pode ser lida por qualquer usuário autenticado (mesmo sem ser do tenant). |
| `created_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Usuário que criou o registro. |
| `updated_by` | `uuid` | `TRUE` | `auth.uid()` | FK | Último usuário que atualizou o registro. |
| `created_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora de criação do registro. |
| `updated_at` | `timestamptz` | `FALSE` | `now()` | | Data e hora da última atualização do registro. |

**Chaves Primárias:**
*   `id`

**Chaves Estrangeiras:**
*   `tenant_id` REFERENCES `tenants(id)` ON DELETE CASCADE
*   `created_by` REFERENCES `profiles(id)` ON DELETE SET NULL
*   `updated_by` REFERENCES `profiles(id)` ON DELETE SET NULL

**Índices:**
*   `idx_settings_tenant_id_key` ON `settings` (`tenant_id`, `key`) WHERE `tenant_id` IS NOT NULL
*   `idx_settings_key` ON `settings` (`key`) WHERE `tenant_id` IS NULL

**Constraints:**
*   `UNIQUE (tenant_id, key)` (Garante que uma chave de configuração seja única por tenant ou global).

**Relacionamentos:**
*   `settings` N:1 `tenants` (Muitas configurações podem ser para uma oficina).

**Políticas RLS:**
*   **`SELECT`:** Usuários podem visualizar configurações públicas. `owner` ou `admin` podem visualizar todas as configurações do seu tenant.
*   **`INSERT`:** Apenas `owner` ou `admin` podem criar configurações para seu tenant. `service_role` pode criar configurações globais.
*   **`UPDATE`:** Apenas `owner` ou `admin` podem atualizar configurações do seu tenant. `service_role` pode atualizar configurações globais.
*   **`DELETE`:** Apenas `owner` ou `admin` podem excluir configurações do seu tenant. `service_role` pode excluir configurações globais.

```sql
-- SQL CREATE TABLE for settings
CREATE TABLE settings (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id uuid REFERENCES tenants(id) ON DELETE CASCADE,
  key text NOT NULL,
  value text,
  type setting_type NOT NULL DEFAULT 'general',
  is_public boolean NOT NULL DEFAULT FALSE,
  created_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  updated_by uuid REFERENCES profiles(id) ON DELETE SET NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  updated_at timestamptz NOT NULL DEFAULT now(),
  CONSTRAINT unique_setting_key_per_tenant UNIQUE (tenant_id, key)
);

-- RLS for settings
ALTER TABLE settings ENABLE ROW LEVEL SECURITY;

CREATE POLICY settings_select_policy
ON settings FOR SELECT
TO authenticated
USING (is_public = TRUE OR (tenant_id IS NOT NULL AND is_active_tenant_member(tenant_id) AND has_tenant_role(tenant_id, ARRAY['owner', 'admin', 'employee']::tenant_role[])));

CREATE POLICY settings_insert_policy
ON settings FOR INSERT
TO authenticated
WITH CHECK (tenant_id IS NULL AND auth.role() = 'service_role' OR (tenant_id IS NOT NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[])));

CREATE POLICY settings_update_policy
ON settings FOR UPDATE
TO authenticated
USING (tenant_id IS NULL AND auth.role() = 'service_role' OR (tenant_id IS NOT NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[])))
WITH CHECK (tenant_id IS NULL AND auth.role() = 'service_role' OR (tenant_id IS NOT NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[])));

CREATE POLICY settings_delete_policy
ON settings FOR DELETE
TO authenticated
USING (tenant_id IS NULL AND auth.role() = 'service_role' OR (tenant_id IS NOT NULL AND has_tenant_role(tenant_id, ARRAY['owner', 'admin']::tenant_role[])));

-- Trigger for updated_at
CREATE TRIGGER set_updated_at_settings
BEFORE UPDATE ON settings
FOR EACH ROW EXECUTE FUNCTION set_updated_at();



## 4. Funções SQL

Para suportar as políticas RLS e a lógica de negócio, as seguintes funções SQL serão criadas no esquema `public` (ou `auth` para funções relacionadas à autenticação, se apropriado).

### 4.1 `is_active_tenant_member(p_tenant_id uuid)`

**Objetivo:** Verifica se o usuário autenticado (`auth.uid()`) é um membro ativo do tenant especificado.

```sql
CREATE OR REPLACE FUNCTION is_active_tenant_member(p_tenant_id uuid)
RETURNS boolean
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  RETURN EXISTS (
    SELECT 1
    FROM public.tenant_users
    WHERE tenant_id = p_tenant_id
      AND user_id = auth.uid()
      AND status = 'active'
  );
END;
$$
;
```

### 4.2 `has_tenant_role(p_tenant_id uuid, p_roles tenant_role[])`

**Objetivo:** Verifica se o usuário autenticado (`auth.uid()`) é um membro ativo do tenant especificado e possui um dos perfis (`p_roles`) fornecidos.

```sql
CREATE OR REPLACE FUNCTION has_tenant_role(p_tenant_id uuid, p_roles tenant_role[])
RETURNS boolean
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  RETURN EXISTS (
    SELECT 1
    FROM public.tenant_users
    WHERE tenant_id = p_tenant_id
      AND user_id = auth.uid()
      AND status = 'active'
      AND role = ANY(p_roles)
  );
END;
$$
;
```

### 4.3 `generate_work_order_number()`

**Objetivo:** Gera um número sequencial único para a Ordem de Serviço dentro do contexto de um `tenant_id`. Esta função será usada como um trigger `BEFORE INSERT` na tabela `work_orders`.

```sql
CREATE OR REPLACE FUNCTION generate_work_order_number()
RETURNS trigger
LANGUAGE plpgsql
AS $$
DECLARE
  next_number bigint;
BEGIN
  -- Garante que o tenant_id está presente
  IF NEW.tenant_id IS NULL THEN
    RAISE EXCEPTION 'tenant_id cannot be NULL for work_orders';
  END IF;

  -- Bloqueia a tabela para garantir atomicidade na geração do número
  LOCK TABLE work_orders IN EXCLUSIVE MODE;

  -- Encontra o maior número de OS para o tenant atual e incrementa
  SELECT COALESCE(MAX(number), 0) + 1
  INTO next_number
  FROM work_orders
  WHERE tenant_id = NEW.tenant_id;

  NEW.number := next_number;

  RETURN NEW;
END;
$$
;
```

### 4.4 `create_audit_log()`

**Objetivo:** Função genérica para criar um registro de auditoria na tabela `audit_logs`. Pode ser chamada por triggers ou diretamente pelo backend para registrar ações importantes.

```sql
CREATE OR REPLACE FUNCTION create_audit_log(
  p_tenant_id uuid,
  p_user_id uuid,
  p_action text,
  p_entity text,
  p_entity_id uuid,
  p_old_value jsonb DEFAULT NULL,
  p_new_value jsonb DEFAULT NULL,
  p_metadata jsonb DEFAULT '{}'::jsonb
)
RETURNS void
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  INSERT INTO public.audit_logs (
    tenant_id,
    user_id,
    action,
    entity,
    entity_id,
    old_value,
    new_value,
    metadata,
    ip_address,
    user_agent
  )
  VALUES (
    p_tenant_id,
    p_user_id,
    p_action,
    p_entity,
    p_entity_id,
    p_old_value,
    p_new_value,
    p_metadata,
    -- auth.request_ip_address() e auth.request_user_agent() são funções do Supabase para obter IP e User-Agent
    auth.request_ip_address(),
    auth.request_user_agent()
  );
END;
$$
;
```

## 5. Triggers

Os triggers são utilizados para automatizar a execução de funções em resposta a eventos específicos (INSERT, UPDATE, DELETE) em uma tabela. Eles são cruciais para manter a integridade e a auditoria do sistema.

### 5.1 `set_updated_at()`

**Objetivo:** Uma função genérica para atualizar automaticamente a coluna `updated_at` de uma tabela para o timestamp atual (`now()`) sempre que uma linha é modificada. Esta função é referenciada por triggers em quase todas as tabelas.

```sql
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$
;
```

**Aplicação:** Conforme visto nas definições das tabelas, um trigger `BEFORE UPDATE` é criado para cada tabela que possui a coluna `updated_at` e precisa ser atualizada automaticamente.

Exemplo:
```sql
CREATE TRIGGER set_updated_at_tenants
BEFORE UPDATE ON tenants
FOR EACH ROW EXECUTE FUNCTION set_updated_at();
```

### 5.2 Auditoria (via `create_audit_log`)

**Objetivo:** Registrar automaticamente ações de `INSERT`, `UPDATE` e `DELETE` em tabelas críticas na tabela `audit_logs`.

Para cada tabela que necessita de auditoria detalhada (ex: `tenants`, `customers`, `work_orders`, `products`, `services`), triggers `AFTER INSERT`, `AFTER UPDATE` e `AFTER DELETE` serão criados para chamar a função `create_audit_log`.

Exemplo para `customers`:

```sql
CREATE OR REPLACE FUNCTION audit_customers_changes()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  IF (TG_OP = 'INSERT') THEN
    PERFORM create_audit_log(
      NEW.tenant_id,
      NEW.created_by,
      'customer_created',
      'customers',
      NEW.id,
      NULL,
      to_jsonb(NEW)
    );
    RETURN NEW;
  ELSIF (TG_OP = 'UPDATE') THEN
    PERFORM create_audit_log(
      NEW.tenant_id,
      NEW.updated_by,
      'customer_updated',
      'customers',
      NEW.id,
      to_jsonb(OLD),
      to_jsonb(NEW)
    );
    RETURN NEW;
  ELSIF (TG_OP = 'DELETE') THEN
    PERFORM create_audit_log(
      OLD.tenant_id,
      auth.uid(), -- O usuário que deletou
      'customer_deleted',
      'customers',
      OLD.id,
      to_jsonb(OLD),
      NULL
    );
    RETURN OLD;
  END IF;
  RETURN NULL;
END;
$$
;

CREATE TRIGGER audit_customers_trigger
AFTER INSERT OR UPDATE OR DELETE ON customers
FOR EACH ROW EXECUTE FUNCTION audit_customers_changes();
```

### 5.3 Geração de número da OS por tenant

**Objetivo:** Garantir que cada Ordem de Serviço tenha um número sequencial único dentro do seu respectivo tenant.

**Aplicação:** O trigger `generate_work_order_number_trigger` já foi definido na tabela `work_orders` e chama a função `generate_work_order_number()` antes de cada `INSERT`.

Exemplo:
```sql
CREATE TRIGGER generate_work_order_number_trigger
BEFORE INSERT ON work_orders
FOR EACH ROW EXECUTE FUNCTION generate_work_order_number();
```

### 5.4 Criação automática do owner

**Objetivo:** Após a criação de um novo `tenant`, um usuário (`auth.uid()`) deve ser automaticamente associado a este tenant com o perfil de `owner`.

```sql
CREATE OR REPLACE FUNCTION create_tenant_owner()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  INSERT INTO public.tenant_users (
    tenant_id,
    user_id,
    role,
    status
  )
  VALUES (
    NEW.id,
    auth.uid(),
    'owner',
    'active'
  );
  RETURN NEW;
END;
$$
;

CREATE TRIGGER create_tenant_owner_trigger
AFTER INSERT ON tenants
FOR EACH ROW EXECUTE FUNCTION create_tenant_owner();
```

### 5.5 Sincronização de pagamentos (Exemplo)

**Objetivo:** Atualizar o status de `accounts_receivable` ou `accounts_payable` quando uma `financial_transaction` correspondente é registrada. Isso pode ser mais complexo e envolver webhooks da Stripe, mas um trigger simples pode ser usado para atualizações internas.

Exemplo (simplificado para `accounts_receivable`):

```sql
CREATE OR REPLACE FUNCTION sync_payment_status()
RETURNS TRIGGER
LANGUAGE plpgsql
SECURITY DEFINER
AS $$
BEGIN
  IF NEW.type = 'income' AND NEW.source = 'accounts_receivable' AND NEW.source_id IS NOT NULL THEN
    UPDATE public.accounts_receivable
    SET status = 'paid',
        paid_at = NEW.created_at,
        payment_method = NEW.payment_method
    WHERE id = NEW.source_id
      AND tenant_id = NEW.tenant_id
      AND status = 'pending';
  END IF;
  RETURN NEW;
END;
$$
;

CREATE TRIGGER sync_payment_status_trigger
AFTER INSERT ON financial_transactions
FOR EACH ROW EXECUTE FUNCTION sync_payment_status();
```

## 6. Policies RLS

As políticas de Row Level Security (RLS) são a pedra angular do isolamento de dados multi-tenant no OficinaPro. Elas foram detalhadas para cada tabela na seção 3. Aqui, resumimos os princípios gerais e a importância.

**Princípios Gerais:**

*   **Habilitação Universal:** O RLS é habilitado em todas as tabelas que contêm dados específicos de tenants (`ALTER TABLE <table_name> ENABLE ROW LEVEL SECURITY;`).
*   **`auth.uid()`:** A função `auth.uid()` do Supabase é utilizada para identificar o usuário atualmente autenticado.
*   **`is_active_tenant_member(tenant_id)`:** Esta função customizada é a base para a maioria das políticas `SELECT`, garantindo que o usuário só veja dados do tenant ao qual pertence e é um membro ativo.
*   **`has_tenant_role(tenant_id, ARRAY[roles])`:** Esta função customizada é usada para controlar permissões de `INSERT`, `UPDATE` e `DELETE` com base no perfil do usuário dentro do tenant.
*   **`WITH CHECK` para `INSERT`/`UPDATE`:** Garante que o usuário tem permissão para criar ou modificar o registro *antes* da operação, e não apenas para ver o registro *depois*.
*   **`USING` para `SELECT`/`DELETE`:** Define quais registros o usuário pode ver ou deletar.
*   **`service_role`:** Para operações administrativas ou de sistema (ex: criação de planos de assinatura, inserção de logs de auditoria, sincronização Stripe), políticas específicas para o `service_role` do Supabase são definidas, permitindo acesso irrestrito a essas operações controladas pelo backend.
*   **Soft Delete:** As políticas `SELECT` para tabelas com soft delete incluem `AND deleted_at IS NULL` para excluir registros logicamente deletados das consultas padrão.

**Exemplo de Política RLS (SELECT para `customers`):**

```sql
CREATE POLICY customers_select_policy
ON customers FOR SELECT
TO authenticated
USING (is_active_tenant_member(tenant_id) AND deleted_at IS NULL);
```

Esta política permite que qualquer usuário autenticado (`TO authenticated`) selecione registros da tabela `customers` *apenas se* ele for um membro ativo do `tenant_id` associado ao cliente *e* o cliente não tiver sido logicamente excluído. Isso demonstra a combinação de isolamento multi-tenant, controle de acesso baseado em função e tratamento de soft delete diretamente no nível do banco de dados.

## 7. Índices

Além dos índices já especificados nas definições de cada tabela, é crucial criar índices compostos adicionais para otimizar o desempenho das consultas, especialmente aquelas que envolvem `tenant_id` e outras colunas frequentemente usadas em cláusulas `WHERE` ou `ORDER BY`.

**Princípios para Criação de Índices:**

*   **`tenant_id` como primeira coluna:** Para tabelas multi-tenant, o `tenant_id` deve ser a primeira coluna em índices compostos. Isso permite que o PostgreSQL filtre rapidamente os dados de um tenant específico antes de aplicar outros critérios.
*   **Colunas de busca frequente:** Incluir colunas usadas em filtros (`WHERE`), junções (`JOIN`) e ordenação (`ORDER BY`).
*   **Evitar excesso de índices:** Muitos índices podem degradar o desempenho de `INSERT`, `UPDATE` e `DELETE`. É importante balancear a necessidade de leitura rápida com o custo de escrita.
*   **Considerar `deleted_at`:** Para tabelas com soft delete, incluir `deleted_at` no índice (geralmente com `WHERE deleted_at IS NULL`) pode otimizar consultas que filtram por registros ativos.

**Exemplos de Índices Compostos Adicionais:**

```sql
-- Índices para performance geral

-- customers
CREATE INDEX idx_customers_tenant_id_created_at ON customers (tenant_id, created_at DESC) WHERE deleted_at IS NULL;

-- vehicles
CREATE INDEX idx_vehicles_tenant_id_model ON vehicles (tenant_id, model) WHERE deleted_at IS NULL;

-- services
CREATE INDEX idx_services_tenant_id_default_price ON services (tenant_id, default_price) WHERE deleted_at IS NULL;

-- products
CREATE INDEX idx_products_tenant_id_sale_price ON products (tenant_id, sale_price) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_tenant_id_stock_quantity ON products (tenant_id, stock_quantity) WHERE deleted_at IS NULL;

-- work_orders
CREATE INDEX idx_work_orders_tenant_id_created_at ON work_orders (tenant_id, created_at DESC) WHERE deleted_at IS NULL;
CREATE INDEX idx_work_orders_tenant_id_status_created_at ON work_orders (tenant_id, status, created_at DESC) WHERE deleted_at IS NULL;

-- accounts_receivable
CREATE INDEX idx_accounts_receivable_tenant_id_status_due_date ON accounts_receivable (tenant_id, status, due_date) WHERE deleted_at IS NULL;

-- accounts_payable
CREATE INDEX idx_accounts_payable_tenant_id_status_due_date ON accounts_payable (tenant_id, status, due_date) WHERE deleted_at IS NULL;

-- financial_transactions
CREATE INDEX idx_financial_transactions_tenant_id_type_date ON financial_transactions (tenant_id, type, transaction_date DESC);

-- audit_logs
CREATE INDEX idx_audit_logs_tenant_id_action_created_at ON audit_logs (tenant_id, action, created_at DESC);

-- notifications
CREATE INDEX idx_notifications_user_id_status_created_at ON notifications (user_id, status, created_at DESC);

-- settings
CREATE INDEX idx_settings_tenant_id_type ON settings (tenant_id, type) WHERE tenant_id IS NOT NULL;
```

## 8. Migrations

As migrations são scripts SQL versionados que permitem gerenciar as alterações no esquema do banco de dados de forma controlada e replicável. No contexto do Supabase, as migrations são essenciais para evoluir o banco de dados ao longo do tempo. A estrutura proposta visa organizar as alterações em arquivos lógicos.

**Estrutura de Diretórios:**

As migrations serão armazenadas em um diretório específico (ex: `supabase/migrations/`) e executadas em ordem cronológica.

**Arquivos de Migration Propostos:**

*   **`001_initial_schema.sql`:**
    *   Criação de todas as extensões necessárias (ex: `pgcrypto`).
    *   Criação de todos os `ENUMs`.
    *   Criação de todas as tabelas (`CREATE TABLE`).
    *   Definição de chaves primárias, chaves estrangeiras e constraints de unicidade.
    *   Criação da função `set_updated_at()`.
    *   Criação dos triggers `set_updated_at` para todas as tabelas relevantes.

*   **`002_rls.sql`:**
    *   Habilitação do Row Level Security (`ALTER TABLE ... ENABLE ROW LEVEL SECURITY;`) para todas as tabelas multi-tenant.
    *   Criação das funções auxiliares para RLS: `is_active_tenant_member()` e `has_tenant_role()`.
    *   Criação de todas as políticas RLS (`CREATE POLICY`) para `SELECT`, `INSERT`, `UPDATE` e `DELETE` em todas as tabelas multi-tenant.

*   **`003_functions.sql`:**
    *   Criação de funções SQL de lógica de negócio, como `generate_work_order_number()` e `create_audit_log()`.
    *   Criação de funções auxiliares para triggers de auditoria (ex: `audit_customers_changes()`).

*   **`004_indexes.sql`:**
    *   Criação de todos os índices compostos adicionais para otimização de performance (`CREATE INDEX`).

*   **`005_triggers.sql`:**
    *   Criação de triggers de lógica de negócio e auditoria que chamam as funções definidas em `003_functions.sql`.
    *   Exemplos: `create_tenant_owner_trigger`, `audit_customers_trigger`, `generate_work_order_number_trigger`, `sync_payment_status_trigger`.

## 9. Seeds

Seeds são scripts que populam o banco de dados com dados iniciais, essenciais para o funcionamento do sistema ou para fins de desenvolvimento e teste. No OficinaPro, os seeds serão usados para configurar o ambiente inicial de um novo tenant ou para dados globais.

**Seed Inicial Proposto (`seed.sql` ou via código de onboarding):**

*   **Criação de um `Owner` e `Tenant`:**
    *   Após o registro de um novo usuário via Supabase Auth, o processo de onboarding deve criar um novo `tenant` e associar o usuário como `owner` via `tenant_users`.
    *   Isso pode ser feito por uma função de banco de dados ou pelo backend (Next.js Server Actions) que chama as APIs do Supabase.

*   **Configurações Padrão:**
    *   Inserção de configurações padrão na tabela `settings` para o novo tenant (ex: `currency_symbol`, `work_order_prefix`, `default_tax_rate`).
    *   Para configurações globais (sem `tenant_id`), estas podem ser inseridas uma única vez no ambiente de produção.

*   **Planos de Assinatura (`subscription_plans`):**
    *   Inserção dos planos de assinatura disponíveis no sistema na tabela `subscription_plans`.
    *   Estes são dados globais e devem ser inseridos uma única vez, geralmente como parte da implantação inicial ou via um script de administração.

Exemplo de seed para `subscription_plans`:

```sql
INSERT INTO subscription_plans (id, name, description, billing_interval, price, stripe_price_id, features, is_active)
VALUES
  (
    gen_random_uuid(),
    'Plano Básico',
    'Ideal para pequenas oficinas, inclui gestão de clientes e OS.',
    'monthly',
    99.90,
    'price_123monthlybasic',
    '{"max_users": 1, "max_vehicles": 100, "reports": false}',
    TRUE
  ),
  (
    gen_random_uuid(),
    'Plano Pro',
    'Para oficinas em crescimento, com controle financeiro e múltiplos usuários.',
    'yearly',
    999.00,
    'price_456yearlypro',
    '{"max_users": 5, "max_vehicles": 1000, "reports": true, "financial_control": true}',
    TRUE
  )
ON CONFLICT (stripe_price_id) DO UPDATE SET
  name = EXCLUDED.name,
  description = EXCLUDED.description,
  billing_interval = EXCLUDED.billing_interval,
  price = EXCLUDED.price,
  features = EXCLUDED.features,
  is_active = EXCLUDED.is_active;
```

## 10. Stripe

A integração com a Stripe é fundamental para o modelo de negócios SaaS do OficinaPro, gerenciando assinaturas, pagamentos e webhooks. O banco de dados armazena referências cruciais para sincronizar o estado da assinatura com a plataforma de pagamentos.

### 10.1 `stripe_customer_id`

*   **Localização:** Tabela `subscriptions`.
*   **Objetivo:** Armazena o ID único do cliente na Stripe. Este ID é gerado quando um novo cliente (oficina) é criado na Stripe para gerenciar suas informações de faturamento e pagamentos.
*   **Uso:** Essencial para realizar operações de cobrança, gerenciar métodos de pagamento e recuperar o histórico de faturamento de um cliente via API da Stripe.

### 10.2 `stripe_subscription_id`

*   **Localização:** Tabela `subscriptions`.
*   **Objetivo:** Armazena o ID único da assinatura ativa do cliente na Stripe. Cada assinatura corresponde a um plano e um intervalo de cobrança.
*   **Uso:** Utilizado para gerenciar o ciclo de vida da assinatura (cancelamento, upgrade, downgrade, pausa) e para sincronizar o status da assinatura entre a Stripe e o OficinaPro.

### 10.3 Webhooks

*   **Objetivo:** A Stripe envia eventos (webhooks) para o backend do OficinaPro (Next.js Server Actions) sempre que ocorre uma mudança de estado relevante (ex: `customer.subscription.updated`, `invoice.payment_succeeded`, `invoice.payment_failed`).
*   **Processamento:** O backend deve ter um endpoint seguro para receber e processar esses webhooks. Ao receber um evento, o backend:
    1.  Verifica a assinatura do webhook para garantir a autenticidade.
    2.  Atualiza o status da assinatura na tabela `subscriptions` com base no evento recebido.
    3.  Cria ou atualiza registros na tabela `invoices` e `financial_transactions` conforme necessário.
    4.  Pode acionar notificações para o usuário (ex: 
notificação de pagamento bem-sucedido ou falha).

### 10.4 Sincronização de Status

*   **Objetivo:** Manter o status da assinatura do tenant no banco de dados do OficinaPro (`subscriptions.status` e `tenants.status`) sempre sincronizado com o estado real da assinatura na Stripe.
*   **Mecanismo:** Principalmente via webhooks da Stripe. Quando um evento como `customer.subscription.updated` ou `customer.subscription.deleted` é recebido, o backend deve:
    1.  Consultar a API da Stripe para obter os detalhes mais recentes da assinatura.
    2.  Atualizar a linha correspondente na tabela `subscriptions` (e, consequentemente, o `status` na tabela `tenants`).
    3.  Lidar com transições de estado (ex: de `trialing` para `active`, de `active` para `past_due`, de `past_due` para `canceled`).
*   **Resiliência:** Implementar um mecanismo de retry para webhooks e um processo de reconciliação diário/semanal para verificar e corrigir quaisquer dessincronizações entre o Supabase e a Stripe.

## Conclusão

Este documento detalha a arquitetura do banco de dados do OficinaPro, um sistema SaaS multiempresa, com foco em PostgreSQL e Supabase. A implementação rigorosa das políticas de Row Level Security (RLS), a estrutura de tabelas com `tenant_id` em todas as entidades operacionais, e a utilização de funções e triggers garantem o isolamento de dados, a segurança e a integridade do sistema. A integração com a Stripe é projetada para gerenciar eficientemente as assinaturas e pagamentos, mantendo a sincronização através de webhooks. Este guia serve como base sólida para o desenvolvimento e a evolução do banco de dados do OficinaPro, permitindo que outras IAs e desenvolvedores construam sobre esta fundação robusta.

## 8. Migrations

As migrations são scripts SQL versionados que permitem gerenciar as alterações no esquema do banco de dados de forma controlada e replicável. No contexto do Supabase, as migrations são essenciais para evoluir o banco de dados ao longo do tempo. A estrutura proposta visa organizar as alterações em arquivos lógicos.

**Estrutura de Diretórios:**

As migrations serão armazenadas em um diretório específico (ex: `supabase/migrations/`) e executadas em ordem cronológica.

**Arquivos de Migration Propostos:**

*   **`001_initial_schema.sql`:**
    *   Criação de todas as extensões necessárias (ex: `pgcrypto`).
    *   Criação de todos os `ENUMs`.
    *   Criação de todas as tabelas (`CREATE TABLE`).
    *   Definição de chaves primárias, chaves estrangeiras e constraints de unicidade.
    *   Criação da função `set_updated_at()`.
    *   Criação dos triggers `set_updated_at` para todas as tabelas relevantes.

*   **`002_rls.sql`:**
    *   Habilitação do Row Level Security (`ALTER TABLE ... ENABLE ROW LEVEL SECURITY;`) para todas as tabelas multi-tenant.
    *   Criação das funções auxiliares para RLS: `is_active_tenant_member()` e `has_tenant_role()`.
    *   Criação de todas as políticas RLS (`CREATE POLICY`) para `SELECT`, `INSERT`, `UPDATE` e `DELETE` em todas as tabelas multi-tenant.

*   **`003_functions.sql`:**
    *   Criação de funções SQL de lógica de negócio, como `generate_work_order_number()` e `create_audit_log()`.
    *   Criação de funções auxiliares para triggers de auditoria (ex: `audit_customers_changes()`).

*   **`004_indexes.sql`:**
    *   Criação de todos os índices compostos adicionais para otimização de performance (`CREATE INDEX`).

*   **`005_triggers.sql`:**
    *   Criação de triggers de lógica de negócio e auditoria que chamam as funções definidas em `003_functions.sql`.
    *   Exemplos: `create_tenant_owner_trigger`, `audit_customers_trigger`, `generate_work_order_number_trigger`, `sync_payment_status_trigger`.

## 9. Seeds

Seeds são scripts que populam o banco de dados com dados iniciais, essenciais para o funcionamento do sistema ou para fins de desenvolvimento e teste. No OficinaPro, os seeds serão usados para configurar o ambiente inicial de um novo tenant ou para dados globais.

**Seed Inicial Proposto (`seed.sql` ou via código de onboarding):**

*   **Criação de um `Owner` e `Tenant`:**
    *   Após o registro de um novo usuário via Supabase Auth, o processo de onboarding deve criar um novo `tenant` e associar o usuário como `owner` via `tenant_users`.
    *   Isso pode ser feito por uma função de banco de dados ou pelo backend (Next.js Server Actions) que chama as APIs do Supabase.

*   **Configurações Padrão:**
    *   Inserção de configurações padrão na tabela `settings` para o novo tenant (ex: `currency_symbol`, `work_order_prefix`, `default_tax_rate`).
    *   Para configurações globais (sem `tenant_id`), estas podem ser inseridas uma única vez no ambiente de produção.

*   **Planos de Assinatura (`subscription_plans`):**
    *   Inserção dos planos de assinatura disponíveis no sistema na tabela `subscription_plans`.
    *   Estes são dados globais e devem ser inseridos uma única vez, geralmente como parte da implantação inicial ou via um script de administração.

Exemplo de seed para `subscription_plans`:

```sql
INSERT INTO subscription_plans (id, name, description, billing_interval, price, stripe_price_id, features, is_active)
VALUES
  (
    gen_random_uuid(),
    'Plano Básico',
    'Ideal para pequenas oficinas, inclui gestão de clientes e OS.',
    'monthly',
    99.90,
    'price_123monthlybasic',
    '{"max_users": 1, "max_vehicles": 100, "reports": false}',
    TRUE
  ),
  (
    gen_random_uuid(),
    'Plano Pro',
    'Para oficinas em crescimento, com controle financeiro e múltiplos usuários.',
    'yearly',
    999.00,
    'price_456yearlypro',
    '{"max_users": 5, "max_vehicles": 1000, "reports": true, "financial_control": true}',
    TRUE
  )
ON CONFLICT (stripe_price_id) DO UPDATE SET
  name = EXCLUDED.name,
  description = EXCLUDED.description,
  billing_interval = EXCLUDED.billing_interval,
  price = EXCLUDED.price,
  features = EXCLUDED.features,
  is_active = EXCLUDED.is_active;
```

## 10. Stripe

A integração com a Stripe é fundamental para o modelo de negócios SaaS do OficinaPro, gerenciando assinaturas, pagamentos e webhooks. O banco de dados armazena referências cruciais para sincronizar o estado da assinatura com a plataforma de pagamentos.

### 10.1 `stripe_customer_id`

*   **Localização:** Tabela `subscriptions`.
*   **Objetivo:** Armazena o ID único do cliente na Stripe. Este ID é gerado quando um novo cliente (oficina) é criado na Stripe para gerenciar suas informações de faturamento e pagamentos.
*   **Uso:** Essencial para realizar operações de cobrança, gerenciar métodos de pagamento e recuperar o histórico de faturamento de um cliente via API da Stripe.

### 10.2 `stripe_subscription_id`

*   **Localização:** Tabela `subscriptions`.
*   **Objetivo:** Armazena o ID único da assinatura ativa do cliente na Stripe. Cada assinatura corresponde a um plano e um intervalo de cobrança.
*   **Uso:** Utilizado para gerenciar o ciclo de vida da assinatura (cancelamento, upgrade, downgrade, pausa) e para sincronizar o status da assinatura entre a Stripe e o OficinaPro.

### 10.3 Webhooks

*   **Objetivo:** A Stripe envia eventos (webhooks) para o backend do OficinaPro (Next.js Server Actions) sempre que ocorre uma mudança de estado relevante (ex: `customer.subscription.updated`, `invoice.payment_succeeded`, `invoice.payment_failed`).
*   **Processamento:** O backend deve ter um endpoint seguro para receber e processar esses webhooks. Ao receber um evento, o backend:
    1.  Verifica a assinatura do webhook para garantir a autenticidade.
    2.  Atualiza o status da assinatura na tabela `subscriptions` com base no evento recebido.
    3.  Cria ou atualiza registros na tabela `invoices` e `financial_transactions` conforme necessário.
    4.  Pode acionar notificações para o usuário (ex: notificação de pagamento bem-sucedido ou falha).

### 10.4 Sincronização de Status

*   **Objetivo:** Manter o status da assinatura do tenant no banco de dados do OficinaPro (`subscriptions.status` e `tenants.status`) sempre sincronizado com o estado real da assinatura na Stripe.
*   **Mecanismo:** Principalmente via webhooks da Stripe. Quando um evento como `customer.subscription.updated` ou `customer.subscription.deleted` é recebido, o backend deve:
    1.  Consultar a API da Stripe para obter os detalhes mais recentes da assinatura.
    2.  Atualizar a linha correspondente na tabela `subscriptions` (e, consequentemente, o `status` na tabela `tenants`).
    3.  Lidar com transições de estado (ex: de `trialing` para `active`, de `active` para `past_due`, de `past_due` para `canceled`).
*   **Resiliência:** Implementar um mecanismo de retry para webhooks e um processo de reconciliação diário/semanal para verificar e corrigir quaisquer dessincronizações entre o Supabase e a Stripe.

## Conclusão

Este documento detalha a arquitetura do banco de dados do OficinaPro, um sistema SaaS multiempresa, com foco em PostgreSQL e Supabase. A implementação rigorosa das políticas de Row Level Security (RLS), a estrutura de tabelas com `tenant_id` em todas as entidades operacionais, e a utilização de funções e triggers garantem o isolamento de dados, a segurança e a integridade do sistema. A integração com a Stripe é projetada para gerenciar eficientemente as assinaturas e pagamentos, mantendo a sincronização através de webhooks. Este guia serve como base sólida para o desenvolvimento e a evolução do banco de dados do OficinaPro, permitindo que outras IAs e desenvolvedores construam sobre esta fundação robusta.
