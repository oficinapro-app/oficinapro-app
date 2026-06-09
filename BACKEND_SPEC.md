# OficinaPro — Especificação Técnica do Backend

**Autor:** Manus AI  
**Data:** 09 de junho de 2026  
**Versão:** 1.0  
**Finalidade:** Este documento detalha a especificação técnica do backend para o sistema OficinaPro, um SaaS multiempresa para oficinas mecânicas. Ele serve como guia oficial para o desenvolvimento da lógica de negócio, garantindo segurança, escalabilidade, performance e a correta integração com serviços externos como Supabase e Stripe.

## 1. Arquitetura do Backend

A arquitetura do backend do OficinaPro é projetada para ser robusta, escalável e segura, suportando um modelo SaaS multiempresa. Utiliza-se uma abordagem de arquitetura em camadas, onde cada camada tem responsabilidades bem definidas, promovendo a separação de preocupações e facilitando a manutenção e evolução do sistema.

### 1.1 Multi-tenant

O OficinaPro é um sistema multi-tenant, o que significa que uma única instância da aplicação serve a múltiplos clientes (oficinas), com cada cliente sendo um "tenant".

*   **Identificação do Tenant:** Cada requisição ao backend deve ser associada a um `tenant_id` válido. Este `tenant_id` é extraído do contexto da sessão do usuário autenticado.
*   **Contexto do Tenant:** Todas as operações de dados e lógica de negócio são executadas dentro do contexto do `tenant_id` do usuário logado, garantindo que os dados de um tenant nunca sejam acessados por outro.

### 1.2 Isolamento por Tenant

O isolamento de dados entre tenants é uma prioridade máxima para garantir a segurança e a privacidade. As principais estratégias para alcançar este isolamento são:

*   **`tenant_id` em todas as tabelas operacionais:** Cada tabela que armazena dados específicos de um tenant (clientes, veículos, ordens de serviço, produtos, etc.) possui uma coluna `tenant_id` (UUID) como chave estrangeira para a tabela `tenants`.
*   **Row Level Security (RLS):** O PostgreSQL, através do Supabase, é configurado com políticas de RLS para garantir que os usuários só possam acessar as linhas de dados que pertencem ao seu `tenant_id` associado. Isso é aplicado diretamente no nível do banco de dados, fornecendo uma camada de segurança fundamental.
*   **Validação em Nível de Aplicação:** Além do RLS, o backend implementa validações adicionais para garantir que todas as operações de escrita e leitura respeitem o `tenant_id` do usuário autenticado. Isso serve como uma camada de defesa extra.

### 1.3 Row Level Security (RLS)

O RLS é a pedra angular do isolamento de dados no OficinaPro. Ele garante que, mesmo que uma query maliciosa ou um erro de aplicação tente acessar dados de outro tenant, o banco de dados impedirá o acesso.

*   **Implementação:** Políticas de RLS serão criadas para todas as tabelas multi-tenant (`customers`, `vehicles`, `work_orders`, `products`, `services`, `accounts_receivable`, `accounts_payable`, `financial_transactions`, `subscriptions`, `invoices`, `audit_logs`, `notifications`, `settings`).
*   **Contexto de Usuário:** O `tenant_id` do usuário autenticado é obtido através do JWT do Supabase Auth e disponibilizado para as políticas de RLS via `auth.uid()` e `auth.jwt()`, que podem ser usadas para consultar o `tenant_id` associado ao `user_id` na tabela `tenant_users`.
*   **Exemplo de Política:** Uma política de `SELECT` para a tabela `customers` permitiria acesso apenas se `customers.tenant_id = (SELECT tenant_id FROM tenant_users WHERE user_id = auth.uid())`.

### 1.4 Server Actions e Route Handlers (Next.js 15)

O backend do OficinaPro será construído utilizando as capacidades do Next.js 15, especificamente Server Actions e Route Handlers, para operações de API e lógica de negócio.

*   **Server Actions:** Serão utilizados para a maioria das operações de CRUD e lógica de negócio que precisam ser executadas no servidor, mas são invocadas diretamente do frontend. Eles oferecem uma forma segura e performática de interagir com o banco de dados e serviços externos, sem a necessidade de criar endpoints de API REST explícitos para cada operação.
*   **Route Handlers:** Serão utilizados para endpoints de API mais tradicionais, como webhooks (ex: Stripe webhooks) ou APIs públicas que não se encaixam no modelo de Server Actions.

### 1.5 Estrutura de Camadas

A arquitetura do backend segue um padrão de camadas para organizar a lógica de negócio e as interações com o banco de dados e serviços externos.

*   **`app/` (Server Actions/Route Handlers):** Esta camada contém os pontos de entrada do backend, onde as requisições do frontend (via Server Actions) ou de serviços externos (via Route Handlers) são recebidas. Eles orquestram a execução dos casos de uso.
*   **`use-cases/` (ou `services/`):** Esta camada contém a lógica de negócio central da aplicação. Cada caso de uso representa uma funcionalidade específica do sistema (ex: `CreateCustomer`, `UpdateWorkOrderStatus`). Eles coordenam as operações entre os `repositories` e `services` externos.
*   **`repositories/`:** Esta camada é responsável pela interação direta com o banco de dados (Supabase/PostgreSQL). Cada repositório encapsula a lógica de acesso a dados para uma entidade específica (ex: `CustomerRepository`, `WorkOrderRepository`). Eles abstraem os detalhes do banco de dados do restante da aplicação.
*   **`services/` (externos):** Esta camada contém a lógica para interagir com serviços externos (ex: `StripeService`, `NotificationService`). Eles encapsulam a complexidade das APIs de terceiros.
*   **`validations/`:** Contém os schemas de validação (ex: Zod) para garantir a integridade dos dados de entrada em todas as camadas.
*   **`types/`:** Definições de tipos TypeScript para todas as entidades e estruturas de dados do sistema.
*   **`lib/`:** Funções utilitárias e de configuração global (ex: cliente Supabase, cliente Stripe).

```
OficinaPro/
├── app/                      # Server Actions e Route Handlers
│   ├── api/                  # Route Handlers (ex: webhooks)
│   │   ├── stripe/webhook.ts
│   │   └── ...
│   ├── (app)/                # Server Actions (invocados do frontend)
│   │   ├── customers/actions.ts
│   │   ├── work-orders/actions.ts
│   │   └── ...
├── lib/                      # Funções utilitárias e de configuração
│   ├── supabase.ts           # Cliente Supabase
│   ├── stripe.ts             # Cliente Stripe
│   ├── auth.ts               # Funções auxiliares de autenticação
│   ├── constants.ts          # Constantes globais
│   └── ...
├── use-cases/                # Lógica de negócio (orquestra repositories e services)
│   ├── auth/                 # Casos de uso de autenticação
│   │   ├── login.ts
│   │   ├── register.ts
│   │   └── ...
│   ├── customers/            # Casos de uso de clientes
│   │   ├── createCustomer.ts
│   │   ├── updateCustomer.ts
│   │   └── ...
│   ├── work-orders/
│   ├── financial/
│   ├── subscriptions/
│   └── ...
├── repositories/             # Interação com o banco de dados (Supabase/PostgreSQL)
│   ├── customerRepository.ts
│   ├── workOrderRepository.ts
│   ├── tenantRepository.ts
│   └── ...
├── services/                 # Interação com serviços externos
│   ├── stripeService.ts
│   ├── notificationService.ts
│   ├── auditService.ts
│   └── ...
├── types/                    # Definições de tipos TypeScript
│   ├── database.ts           # Tipos gerados do Supabase
│   ├── entities.ts           # Tipos de entidades de domínio
│   ├── auth.ts
│   └── ...
├── validations/              # Schemas de validação (ex: Zod)
│   ├── auth.ts
│   ├── customer.ts
│   ├── workOrder.ts
│   └── ...
└── ...
```

### 1.6 Validações

As validações são aplicadas em múltiplas camadas para garantir a integridade e segurança dos dados:

*   **Validação de Schema (Zod):** Todos os dados de entrada (payloads de Server Actions, Route Handlers) são validados usando schemas Zod. Isso garante que os dados estejam no formato e tipo esperados antes de qualquer processamento.
*   **Validação de Regras de Negócio:** Realizada na camada de `use-cases`, onde a lógica de negócio verifica se os dados de entrada fazem sentido dentro do contexto do sistema (ex: um usuário não pode ter uma `role` inválida).
*   **Validação de Banco de Dados (Constraints):** O próprio PostgreSQL impõe constraints (NOT NULL, UNIQUE, CHECK, Foreign Keys) para garantir a integridade referencial e a validade dos dados no nível mais baixo.

### 1.7 Auditoria

Um sistema de auditoria robusto é essencial para rastrear alterações importantes e garantir a conformidade.

*   **`audit_logs` Tabela:** Todas as operações de CRUD em entidades críticas (clientes, veículos, ordens de serviço, produtos, serviços, usuários, configurações) são registradas na tabela `audit_logs`.
*   **Detalhes do Log:** Cada log inclui: `user_id`, `tenant_id`, `action` (CREATE, UPDATE, DELETE), `entity_type`, `entity_id`, `old_data` (JSONB), `new_data` (JSONB), `ip_address`, `user_agent`, `created_at`.
*   **Implementação:** Triggers de banco de dados (para operações de baixo nível) e/ou lógica na camada de `use-cases` (para operações de alto nível) são usados para gerar os logs de auditoria.

### 1.8 Logs

Uma estratégia de logging abrangente é crucial para monitoramento, depuração e análise de problemas.

*   **Logs de Erro:** Todos os erros inesperados e exceções são registrados com detalhes (stack trace, contexto da requisição, `tenant_id`, `user_id`).
*   **Logs de Sistema:** Eventos importantes do sistema (início/fim de processos, integrações com serviços externos, etc.) são registrados para monitoramento operacional.
*   **Logs Financeiros:** Transações financeiras (pagamentos, cobranças, reembolsos) são registradas com detalhes para fins de reconciliação e auditoria.
*   **Ferramentas:** Utilização de uma biblioteca de logging (ex: Winston, Pino) e integração com um serviço de agregação de logs (ex: Vercel Analytics, Datadog, Sentry) para centralizar e analisar os logs.

## 2. Casos de Uso

Esta seção detalha os principais casos de uso do sistema, descrevendo o fluxo, regras de negócio, validações e resultados esperados para cada funcionalidade. Cada caso de uso será implementado como uma função ou classe na camada `use-cases/`.

### 2.1 Autenticação

#### 2.1.1 Login

**Objetivo:** Autenticar um usuário existente no sistema.

**Entrada:** `email` (string), `password` (string).

**Regras de Negócio:**
*   O e-mail e a senha devem corresponder a um usuário registrado.
*   O usuário deve estar ativo.
*   O tenant associado ao usuário deve estar ativo e com uma assinatura válida (ou em trial).

**Validações:**
*   `email`: Formato de e-mail válido, obrigatório.
*   `password`: Obrigatório.

**Permissões:** Usuários não autenticados.

**Fluxo:**
1.  Recebe `email` e `password`.
2.  Chama `AuthService.signInWithPassword(email, password)`.
3.  Em caso de sucesso, o Supabase retorna um JWT.
4.  Decodifica o JWT para obter `user_id` e `tenant_id` (via RLS context).
5.  Verifica o status do `tenant` e da `subscription` (via `SubscriptionService`).
6.  Retorna sucesso e informações do usuário/tenant.

**Erros Possíveis:**
*   `401 Unauthorized`: Credenciais inválidas.
*   `403 Forbidden`: Usuário inativo, tenant inativo ou assinatura inválida.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** JWT de autenticação e informações básicas do usuário/tenant.

**Critérios de Aceite:**
*   Usuário com credenciais válidas consegue fazer login.
*   Usuário com credenciais inválidas recebe erro de autenticação.
*   Usuário com tenant inativo ou assinatura expirada é impedido de logar.

#### 2.1.2 Cadastro

**Objetivo:** Registrar um novo usuário e criar um novo tenant (oficina) no sistema.

**Entrada:** `fullName` (string), `email` (string), `password` (string), `companyName` (string).

**Regras de Negócio:**
*   O e-mail não deve estar em uso.
*   A senha deve atender aos requisitos de segurança (mínimo 6 caracteres).
*   Um novo `tenant` é criado, e o usuário é associado como `owner`.
*   Um período de trial é iniciado automaticamente para o novo tenant.

**Validações:**
*   `fullName`: Obrigatório.
*   `email`: Formato de e-mail válido, obrigatório, único.
*   `password`: Obrigatório, mínimo 6 caracteres.
*   `companyName`: Obrigatório.

**Permissões:** Usuários não autenticados.

**Fluxo:**
1.  Recebe dados de cadastro.
2.  Chama `AuthService.signUp(email, password)` para criar o usuário no Supabase Auth.
3.  Em caso de sucesso, cria um novo `tenant` via `TenantRepository.createTenant(companyName)`.
4.  Associa o `user_id` recém-criado ao `tenant_id` como `owner` via `TenantRepository.addTenantUser(tenantId, userId, 'owner')`.
5.  Inicia um período de trial para o novo tenant via `SubscriptionService.startTrial(tenantId)`.
6.  Retorna sucesso e informações do usuário/tenant.

**Erros Possíveis:**
*   `409 Conflict`: E-mail já registrado.
*   `422 Unprocessable Entity`: Dados de entrada inválidos.
*   `500 Internal Server Error`: Erro inesperado na criação do tenant ou trial.

**Resultado Esperado:** Novo usuário e tenant criados, trial iniciado.

**Critérios de Aceite:**
*   Novo usuário e oficina são criados com sucesso.
*   E-mail duplicado resulta em erro.
*   Trial é ativado automaticamente.
*   Usuário é definido como `owner` do novo tenant.

#### 2.1.3 Recuperação de Senha

**Objetivo:** Enviar um link de redefinição de senha para o e-mail do usuário.

**Entrada:** `email` (string).

**Regras de Negócio:**
*   O e-mail deve corresponder a um usuário registrado.

**Validações:**
*   `email`: Formato de e-mail válido, obrigatório.

**Permissões:** Usuários não autenticados.

**Fluxo:**
1.  Recebe `email`.
2.  Chama `AuthService.sendPasswordResetEmail(email)`.
3.  Retorna sucesso (mesmo que o e-mail não exista, para evitar enumeração de usuários).

**Erros Possíveis:**
*   `422 Unprocessable Entity`: E-mail inválido.
*   `500 Internal Server Error`: Erro no serviço de e-mail.

**Resultado Esperado:** E-mail de redefinição enviado.

**Critérios de Aceite:**
*   Link de redefinição é enviado para e-mail válido.
*   Mensagem de sucesso é exibida ao usuário, sem revelar se o e-mail existe.

#### 2.1.4 Alteração de Senha

**Objetivo:** Redefinir a senha de um usuário após a validação do token de recuperação.

**Entrada:** `newPassword` (string), `confirmPassword` (string).

**Regras de Negócio:**
*   `newPassword` e `confirmPassword` devem ser idênticas.
*   A nova senha deve atender aos requisitos de segurança.
*   O usuário deve estar no contexto de redefinição de senha (via token).

**Validações:**
*   `newPassword`: Obrigatório, mínimo 6 caracteres.
*   `confirmPassword`: Obrigatório, deve ser igual a `newPassword`.

**Permissões:** Usuários com token de redefinição de senha válido.

**Fluxo:**
1.  Recebe `newPassword` e `confirmPassword`.
2.  Valida se as senhas coincidem e atendem aos requisitos.
3.  Chama `AuthService.updateUserPassword(newPassword)` (o Supabase Auth gerencia o contexto do token).
4.  Retorna sucesso.

**Erros Possíveis:**
*   `422 Unprocessable Entity`: Senhas não coincidem ou são inválidas.
*   `401 Unauthorized`: Token de redefinição inválido ou expirado.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Senha do usuário atualizada.

**Critérios de Aceite:**
*   Senha é alterada com sucesso.
*   Erros são retornados para senhas inválidas ou não coincidentes.

#### 2.1.5 Logout

**Objetivo:** Encerrar a sessão do usuário.

**Entrada:** Nenhuma.

**Regras de Negócio:** Nenhuma.

**Validações:** Nenhuma.

**Permissões:** Usuários autenticados.

**Fluxo:**
1.  Chama `AuthService.signOut()`.
2.  Invalida o JWT do usuário.
3.  Retorna sucesso.

**Erros Possíveis:**
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Sessão do usuário encerrada.

**Critérios de Aceite:**
*   Usuário é deslogado com sucesso.

### 2.2 Empresas (Tenants)

#### 2.2.1 Criar Tenant (via cadastro)

**Objetivo:** Criar um novo registro de oficina (tenant) no sistema.

**Entrada:** `companyName` (string), `ownerUserId` (UUID).

**Regras de Negócio:**
*   `companyName` deve ser único.
*   O `ownerUserId` deve ser um usuário válido e não associado a outro tenant como owner.

**Validações:**
*   `companyName`: Obrigatório, único.
*   `ownerUserId`: Obrigatório, UUID válido.

**Permissões:** Interno (invocado pelo caso de uso de cadastro).

**Fluxo:**
1.  Recebe `companyName` e `ownerUserId`.
2.  Chama `TenantRepository.createTenant(companyName)`.
3.  Associa o `ownerUserId` ao novo `tenant_id` com a role `owner`.
4.  Retorna o `tenant_id` criado.

**Erros Possíveis:**
*   `409 Conflict`: Nome da empresa já existe.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Novo tenant criado.

**Critérios de Aceite:**
*   Tenant é criado com sucesso e associado ao owner.
*   Nome de empresa duplicado resulta em erro.

#### 2.2.2 Atualizar Informações da Oficina

**Objetivo:** Atualizar os dados cadastrais e configurações de uma oficina.

**Entrada:** `tenantId` (UUID), `data` (objeto com campos a serem atualizados: `companyName`, `cnpj`, `stateRegistration`, `email`, `phone`, `whatsapp`, `logoUrl`, `address`, `financialSettings`, `operationalSettings`).

**Regras de Negócio:**
*   Apenas o `owner` ou `admin` do tenant pode atualizar as informações.
*   `companyName` deve ser único (exceto para o próprio tenant).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `companyName`: Opcional, único (se fornecido).
*   Outros campos: Formatos válidos.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões do usuário logado (`hasTenantRole(tenantId, ['owner', 'admin'])`).
3.  Chama `TenantRepository.updateTenant(tenantId, data)`.
4.  Registra auditoria.
5.  Retorna sucesso.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Tenant não encontrado.
*   `409 Conflict`: Nome da empresa já existe.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Informações do tenant atualizadas.

**Critérios de Aceite:**
*   Informações da oficina são atualizadas com sucesso.
*   Usuários sem permissão são impedidos de atualizar.
*   Auditoria é registrada.

#### 2.2.3 Configurações (Gerais e Específicas)

**Objetivo:** Gerenciar configurações gerais do tenant e preferências específicas.

**Entrada:** `tenantId` (UUID), `key` (string), `value` (any).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem alterar configurações do tenant.
*   Algumas configurações podem ser somente leitura ou ter validações específicas.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `key`: Obrigatório.
*   `value`: Validado conforme o tipo de configuração.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `key` e `value`.
2.  Verifica permissões.
3.  Chama `SettingsRepository.updateSetting(tenantId, key, value)`.
4.  Registra auditoria.
5.  Retorna sucesso.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Configuração não encontrada.
*   `422 Unprocessable Entity`: Valor inválido.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Configuração atualizada.

**Critérios de Aceite:**
*   Configurações são atualizadas com sucesso.
*   Permissões são respeitadas.
*   Auditoria é registrada.

### 2.3 Usuários

#### 2.3.1 Convidar Usuário

**Objetivo:** Convidar um novo usuário para o tenant, atribuindo-lhe uma role.

**Entrada:** `tenantId` (UUID), `email` (string), `role` (`tenant_role` enum).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem convidar usuários.
*   O e-mail não deve estar em uso no Supabase Auth.
*   O número de usuários ativos não pode exceder o limite do plano de assinatura.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `email`: Formato de e-mail válido, obrigatório.
*   `role`: Obrigatório, deve ser um valor válido do enum `tenant_role`.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `email` e `role`.
2.  Verifica permissões e limite de usuários do plano de assinatura.
3.  Chama `AuthService.inviteUserByEmail(email)`.
4.  Em caso de sucesso, cria um `tenant_user` com `user_id` do usuário convidado, `tenant_id` e `role`.
5.  Registra auditoria.
6.  Retorna sucesso.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão ou limite de usuários excedido.
*   `409 Conflict`: E-mail já registrado.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Convite enviado, `tenant_user` criado.

**Critérios de Aceite:**
*   Convite é enviado e `tenant_user` é criado.
*   Limite de usuários do plano é respeitado.
*   Usuários sem permissão são impedidos de convidar.
*   Auditoria é registrada.

#### 2.3.2 Alterar Permissões do Usuário

**Objetivo:** Mudar a role de um usuário dentro do tenant.

**Entrada:** `tenantId` (UUID), `targetUserId` (UUID), `newRole` (`tenant_role` enum).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem alterar permissões.
*   Um `owner` não pode alterar a própria role para algo diferente de `owner`.
*   Um `admin` não pode alterar a role de um `owner`.
*   A role `owner` não pode ser removida se for a única.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `targetUserId`: Obrigatório, UUID válido.
*   `newRole`: Obrigatório, valor válido do enum `tenant_role`.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `targetUserId` e `newRole`.
2.  Verifica permissões e regras de negócio.
3.  Chama `TenantRepository.updateTenantUserRole(tenantId, targetUserId, newRole)`.
4.  Registra auditoria.
5.  Retorna sucesso.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão ou violação de regras de negócio.
*   `404 Not Found`: Usuário não encontrado no tenant.
*   `422 Unprocessable Entity`: Role inválida.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Role do usuário atualizada.

**Critérios de Aceite:**
*   Role do usuário é alterada com sucesso, respeitando as regras.
*   Usuários sem permissão são impedidos de alterar roles.
*   Auditoria é registrada.

#### 2.3.3 Remover Usuário

**Objetivo:** Remover um usuário de um tenant.

**Entrada:** `tenantId` (UUID), `targetUserId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem remover usuários.
*   Um `owner` não pode remover a si mesmo se for o único `owner`.
*   Um `admin` não pode remover um `owner`.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `targetUserId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `targetUserId`.
2.  Verifica permissões e regras de negócio.
3.  Chama `TenantRepository.removeTenantUser(tenantId, targetUserId)` (soft delete ou exclusão).
4.  Registra auditoria.
5.  Retorna sucesso.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão ou violação de regras de negócio.
*   `404 Not Found`: Usuário não encontrado no tenant.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Usuário removido do tenant.

**Critérios de Aceite:**
*   Usuário é removido com sucesso, respeitando as regras.
*   Usuários sem permissão são impedidos de remover.
*   Auditoria é registrada.

### 2.4 Clientes

#### 2.4.1 Criar Cliente

**Objetivo:** Cadastrar um novo cliente para o tenant.

**Entrada:** `tenantId` (UUID), `data` (objeto com `name`, `document`, `email`, `phone`, `address`, etc.).

**Regras de Negócio:**
*   O `document` (CPF/CNPJ) deve ser único por tenant para clientes ativos.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `name`: Obrigatório.
*   `document`: Obrigatório, formato válido, único.
*   `email`: Formato válido (se fornecido).

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `CustomerRepository.createCustomer(tenantId, data)`.
5.  Registra auditoria.
6.  Retorna o cliente criado.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `409 Conflict`: Documento já existe.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Novo cliente cadastrado.

**Critérios de Aceite:**
*   Cliente é criado com sucesso.
*   Documento duplicado resulta em erro.
*   Auditoria é registrada.

#### 2.4.2 Editar Cliente

**Objetivo:** Atualizar os dados de um cliente existente.

**Entrada:** `tenantId` (UUID), `customerId` (UUID), `data` (objeto com campos a serem atualizados).

**Regras de Negócio:**
*   Apenas `owner`, `admin` ou `employee` podem editar clientes.
*   O `document` (CPF/CNPJ) deve ser único por tenant para clientes ativos (exceto para o próprio cliente).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `customerId`: Obrigatório, UUID válido.
*   `document`: Opcional, formato válido, único (se fornecido).

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `customerId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `CustomerRepository.updateCustomer(tenantId, customerId, data)`.
5.  Registra auditoria.
6.  Retorna o cliente atualizado.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Cliente não encontrado.
*   `409 Conflict`: Documento já existe para outro cliente.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Cliente atualizado.

**Critérios de Aceite:**
*   Cliente é atualizado com sucesso.
*   Auditoria é registrada.

#### 2.4.3 Arquivar Cliente (Soft Delete)

**Objetivo:** Marcar um cliente como inativo sem removê-lo permanentemente do banco de dados.

**Entrada:** `tenantId` (UUID), `customerId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem arquivar clientes.
*   Um cliente com ordens de serviço abertas não pode ser arquivado.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `customerId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `customerId`.
2.  Verifica permissões.
3.  Verifica se o cliente tem OS abertas.
4.  Chama `CustomerRepository.softDeleteCustomer(tenantId, customerId)` (define `deleted_at` e `status='inactive'`).
5.  Registra auditoria.
6.  Retorna sucesso.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Cliente não encontrado.
*   `409 Conflict`: Cliente com OS abertas.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Cliente marcado como inativo.

**Critérios de Aceite:**
*   Cliente é arquivado com sucesso.
*   Cliente com OS abertas não pode ser arquivado.
*   Auditoria é registrada.

#### 2.4.4 Buscar Clientes

**Objetivo:** Recuperar uma lista de clientes com opções de busca e paginação.

**Entrada:** `tenantId` (UUID), `filters` (objeto com `searchQuery`, `status`, `page`, `pageSize`, `sortBy`, `sortOrder`).

**Regras de Negócio:**
*   Apenas usuários do tenant podem buscar clientes.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `page`, `pageSize`: Numéricos, positivos.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `filters`.
2.  Verifica permissões.
3.  Chama `CustomerRepository.findCustomers(tenantId, filters)`.
4.  Retorna lista de clientes e total para paginação.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista paginada de clientes.

**Critérios de Aceite:**
*   Busca e filtro funcionam corretamente.
*   Paginação retorna os dados esperados.
*   Permissões são respeitadas.

### 2.5 Veículos

#### 2.5.1 Criar Veículo

**Objetivo:** Cadastrar um novo veículo e associá-lo a um cliente.

**Entrada:** `tenantId` (UUID), `data` (objeto com `customerId`, `plate`, `brand`, `model`, `year`, etc.).

**Regras de Negócio:**
*   A `plate` (placa) deve ser única por tenant para veículos ativos.
*   O `customerId` deve ser um cliente existente no tenant.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `customerId`: Obrigatório, UUID válido.
*   `plate`: Obrigatório, formato válido, único.
*   `brand`, `model`, `year`: Obrigatórios.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `VehicleRepository.createVehicle(tenantId, data)`.
5.  Registra auditoria.
6.  Retorna o veículo criado.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Cliente não encontrado.
*   `409 Conflict`: Placa já existe.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Novo veículo cadastrado.

**Critérios de Aceite:**
*   Veículo é criado com sucesso e associado ao cliente.
*   Placa duplicada resulta em erro.
*   Auditoria é registrada.

#### 2.5.2 Atualizar Veículo

**Objetivo:** Atualizar os dados de um veículo existente.

**Entrada:** `tenantId` (UUID), `vehicleId` (UUID), `data` (objeto com campos a serem atualizados).

**Regras de Negócio:**
*   Apenas `owner`, `admin` ou `employee` podem editar veículos.
*   A `plate` (placa) deve ser única por tenant para veículos ativos (exceto para o próprio veículo).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `vehicleId`: Obrigatório, UUID válido.
*   `plate`: Opcional, formato válido, único (se fornecido).

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `vehicleId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `VehicleRepository.updateVehicle(tenantId, vehicleId, data)`.
5.  Registra auditoria.
6.  Retorna o veículo atualizado.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Veículo não encontrado.
*   `409 Conflict`: Placa já existe para outro veículo.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Veículo atualizado.

**Critérios de Aceite:**
*   Veículo é atualizado com sucesso.
*   Auditoria é registrada.

#### 2.5.3 Histórico de Veículo

**Objetivo:** Recuperar o histórico de ordens de serviço de um veículo.

**Entrada:** `tenantId` (UUID), `vehicleId` (UUID).

**Regras de Negócio:**
*   Apenas usuários do tenant podem ver o histórico de veículos.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `vehicleId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `vehicleId`.
2.  Verifica permissões.
3.  Chama `WorkOrderRepository.findWorkOrdersByVehicle(tenantId, vehicleId)`.
4.  Retorna lista de ordens de serviço.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Veículo não encontrado.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista de ordens de serviço do veículo.

**Critérios de Aceite:**
*   Histórico de OS do veículo é retornado corretamente.
*   Permissões são respeitadas.

### 2.6 Serviços (Catálogo)

#### 2.6.1 Criar Serviço

**Objetivo:** Cadastrar um novo serviço no catálogo da oficina.

**Entrada:** `tenantId` (UUID), `data` (objeto com `name`, `description`, `defaultPrice`, `estimatedTime`, etc.).

**Regras de Negócio:**
*   O `name` do serviço deve ser único por tenant para serviços ativos.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `name`: Obrigatório, único.
*   `defaultPrice`: Numérico, positivo.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `ServiceCatalogRepository.createService(tenantId, data)`.
5.  Registra auditoria.
6.  Retorna o serviço criado.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `409 Conflict`: Nome do serviço já existe.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Novo serviço cadastrado.

**Critérios de Aceite:**
*   Serviço é criado com sucesso.
*   Nome de serviço duplicado resulta em erro.
*   Auditoria é registrada.

#### 2.6.2 Editar Serviço

**Objetivo:** Atualizar os dados de um serviço existente no catálogo.

**Entrada:** `tenantId` (UUID), `serviceId` (UUID), `data` (objeto com campos a serem atualizados).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem editar serviços.
*   O `name` do serviço deve ser único por tenant para serviços ativos (exceto para o próprio serviço).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `serviceId`: Obrigatório, UUID válido.
*   `name`: Opcional, único (se fornecido).

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `serviceId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `ServiceCatalogRepository.updateService(tenantId, serviceId, data)`.
5.  Registra auditoria.
6.  Retorna o serviço atualizado.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Serviço não encontrado.
*   `409 Conflict`: Nome do serviço já existe para outro serviço.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Serviço atualizado.

**Critérios de Aceite:**
*   Serviço é atualizado com sucesso.
*   Auditoria é registrada.

#### 2.6.3 Arquivar Serviço (Soft Delete)

**Objetivo:** Marcar um serviço como inativo no catálogo.

**Entrada:** `tenantId` (UUID), `serviceId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem arquivar serviços.
*   Um serviço não pode ser arquivado se estiver em uma OS aberta.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `serviceId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `serviceId`.
2.  Verifica permissões.
3.  Verifica se o serviço está em alguma OS aberta.
4.  Chama `ServiceCatalogRepository.softDeleteService(tenantId, serviceId)`.
5.  Registra auditoria.
6.  Retorna sucesso.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Serviço não encontrado.
*   `409 Conflict`: Serviço em uso em OS aberta.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Serviço marcado como inativo.

**Critérios de Aceite:**
*   Serviço é arquivado com sucesso.
*   Serviço em uso não pode ser arquivado.
*   Auditoria é registrada.

#### 2.6.4 Buscar Serviços

**Objetivo:** Recuperar uma lista de serviços do catálogo com opções de busca e paginação.

**Entrada:** `tenantId` (UUID), `filters` (objeto com `searchQuery`, `status`, `page`, `pageSize`, `sortBy`, `sortOrder`).

**Regras de Negócio:**
*   Apenas usuários do tenant podem buscar serviços.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `page`, `pageSize`: Numéricos, positivos.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `filters`.
2.  Verifica permissões.
3.  Chama `ServiceCatalogRepository.findServices(tenantId, filters)`.
4.  Retorna lista de serviços e total para paginação.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista paginada de serviços.

**Critérios de Aceite:**
*   Busca e filtro funcionam corretamente.
*   Paginação retorna os dados esperados.
*   Permissões são respeitadas.

### 2.7 Produtos (Catálogo)

#### 2.7.1 Criar Produto

**Objetivo:** Cadastrar um novo produto/peça no catálogo da oficina.

**Entrada:** `tenantId` (UUID), `data` (objeto com `name`, `sku`, `description`, `costPrice`, `salePrice`, `stockQuantity`, `minStockQuantity`, etc.).

**Regras de Negócio:**
*   O `name` e `sku` do produto devem ser únicos por tenant para produtos ativos.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `name`: Obrigatório, único.
*   `sku`: Opcional, único (se fornecido).
*   `costPrice`, `salePrice`, `stockQuantity`, `minStockQuantity`: Numéricos, positivos.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `ProductCatalogRepository.createProduct(tenantId, data)`.
5.  Registra auditoria.
6.  Retorna o produto criado.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `409 Conflict`: Nome ou SKU do produto já existe.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Novo produto cadastrado.

**Critérios de Aceite:**
*   Produto é criado com sucesso.
*   Nome/SKU duplicado resulta em erro.
*   Auditoria é registrada.

#### 2.7.2 Editar Produto

**Objetivo:** Atualizar os dados de um produto existente no catálogo.

**Entrada:** `tenantId` (UUID), `productId` (UUID), `data` (objeto com campos a serem atualizados).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem editar produtos.
*   O `name` e `sku` do produto devem ser únicos por tenant para produtos ativos (exceto para o próprio produto).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `productId`: Obrigatório, UUID válido.
*   `name`, `sku`: Opcional, único (se fornecido).

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `productId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `ProductCatalogRepository.updateProduct(tenantId, productId, data)`.
5.  Registra auditoria.
6.  Retorna o produto atualizado.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Produto não encontrado.
*   `409 Conflict`: Nome ou SKU do produto já existe para outro produto.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Produto atualizado.

**Critérios de Aceite:**
*   Produto é atualizado com sucesso.
*   Auditoria é registrada.

#### 2.7.3 Arquivar Produto (Soft Delete)

**Objetivo:** Marcar um produto como inativo no catálogo.

**Entrada:** `tenantId` (UUID), `productId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem arquivar produtos.
*   Um produto não pode ser arquivado se estiver em uma OS aberta.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `productId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `productId`.
2.  Verifica permissões.
3.  Verifica se o produto está em alguma OS aberta.
4.  Chama `ProductCatalogRepository.softDeleteProduct(tenantId, productId)`.
5.  Registra auditoria.
6.  Retorna sucesso.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Produto não encontrado.
*   `409 Conflict`: Produto em uso em OS aberta.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Produto marcado como inativo.

**Critérios de Aceite:**
*   Produto é arquivado com sucesso.
*   Produto em uso não pode ser arquivado.
*   Auditoria é registrada.

#### 2.7.4 Buscar Produtos

**Objetivo:** Recuperar uma lista de produtos do catálogo com opções de busca e paginação.

**Entrada:** `tenantId` (UUID), `filters` (objeto com `searchQuery`, `status`, `page`, `pageSize`, `sortBy`, `sortOrder`).

**Regras de Negócio:**
*   Apenas usuários do tenant podem buscar produtos.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `page`, `pageSize`: Numéricos, positivos.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `filters`.
2.  Verifica permissões.
3.  Chama `ProductCatalogRepository.findProducts(tenantId, filters)`.
4.  Retorna lista de produtos e total para paginação.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista paginada de produtos.

**Critérios de Aceite:**
*   Busca e filtro funcionam corretamente.
*   Paginação retorna os dados esperados.
*   Permissões são respeitadas.

### 2.8 Ordens de Serviço

#### 2.8.1 Abrir OS

**Objetivo:** Criar uma nova Ordem de Serviço para um cliente e veículo específicos.

**Entrada:** `tenantId` (UUID), `data` (objeto com `customerId`, `vehicleId`, `clientReport`, `entryMileage`, `expectedDeliveryDate`, `responsibleEmployeeId`, `internalNotes`).

**Regras de Negócio:**
*   `customerId` e `vehicleId` devem existir e pertencer ao `tenantId`.
*   `vehicleId` deve estar associado ao `customerId`.
*   Um número de OS único é gerado automaticamente para o tenant.
*   O status inicial da OS é `draft` ou `open`.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `customerId`: Obrigatório, UUID válido.
*   `vehicleId`: Obrigatório, UUID válido.
*   `clientReport`: Obrigatório.
*   `entryMileage`: Numérico, positivo.
*   `expectedDeliveryDate`: Data futura ou presente.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data` e a associação cliente-veículo.
4.  Gera um número de OS único (`generate_work_order_number()`).
5.  Chama `WorkOrderRepository.createWorkOrder(tenantId, data)`.
6.  Registra auditoria (`create_audit_log()`).
7.  Retorna a OS criada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Cliente ou veículo não encontrado.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Nova Ordem de Serviço criada com status inicial.

**Critérios de Aceite:**
*   OS é criada com sucesso, número único gerado.
*   Cliente e veículo são corretamente associados.
*   Auditoria é registrada.

#### 2.8.2 Alterar Status da OS

**Objetivo:** Mudar o status de uma Ordem de Serviço, seguindo um fluxo predefinido.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID), `newStatus` (`work_order_status` enum), `notes` (string, opcional).

**Regras de Negócio:**
*   Apenas `owner`, `admin` ou `employee` podem alterar o status (com restrições).
*   A transição de status deve seguir o fluxo definido (ex: `draft` -> `open` -> `waiting_approval` -> `approved` -> `in_progress` -> `completed` -> `delivered`).
*   Não é possível mudar para um status anterior (exceto `reopened`).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.
*   `newStatus`: Obrigatório, valor válido do enum `work_order_status`.

**Permissões:** `owner`, `admin`, `employee` do tenant (com restrições de status).

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`, `newStatus` e `notes`.
2.  Verifica permissões e o fluxo de transição de status.
3.  Chama `WorkOrderRepository.updateWorkOrderStatus(tenantId, workOrderId, newStatus)`.
4.  Registra histórico da OS (`work_order_history`) e auditoria.
5.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão ou transição de status inválida.
*   `404 Not Found`: OS não encontrada.
*   `422 Unprocessable Entity`: Status inválido.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Status da OS atualizado, histórico registrado.

**Critérios de Aceite:**
*   Status da OS é alterado com sucesso, respeitando o fluxo.
*   Histórico da OS é registrado.
*   Auditoria é registrada.

#### 2.8.3 Aprovar OS

**Objetivo:** Marcar uma Ordem de Serviço como aprovada pelo cliente.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID).

**Regras de Negócio:**
*   A OS deve estar no status `waiting_approval`.
*   Apenas `owner` ou `admin` podem aprovar.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`.
2.  Verifica permissões e status da OS.
3.  Chama `WorkOrderService.approveWorkOrder(tenantId, workOrderId)` (que altera o status para `approved`).
4.  Registra histórico e auditoria.
5.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS não encontrada.
*   `409 Conflict`: OS não está em `waiting_approval`.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** OS com status `approved`.

**Critérios de Aceite:**
*   OS é aprovada com sucesso.
*   Auditoria é registrada.

#### 2.8.4 Adicionar Serviços à OS

**Objetivo:** Adicionar um ou mais serviços a uma Ordem de Serviço.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID), `services` (array de objetos com `serviceId` (opcional), `description`, `quantity`, `unitPrice`, `discount`).

**Regras de Negócio:**
*   A OS não pode estar em status `completed`, `delivered` ou `canceled`.
*   `serviceId` (se fornecido) deve ser um serviço existente no catálogo do tenant.
*   `quantity` e `unitPrice` devem ser positivos.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.
*   `services`: Array não vazio.
*   Cada item de serviço: `description` obrigatório, `quantity` e `unitPrice` numéricos e positivos.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`, `services`.
2.  Verifica permissões e status da OS.
3.  Para cada serviço:
    *   Valida os dados.
    *   Chama `WorkOrderRepository.addWorkOrderService(tenantId, workOrderId, serviceData)`.
4.  Atualiza o valor total da OS.
5.  Registra auditoria.
6.  Retorna a OS atualizada com os serviços.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS ou serviço do catálogo não encontrado.
*   `409 Conflict`: OS em status finalizado.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Serviços adicionados à OS, total da OS atualizado.

**Critérios de Aceite:**
*   Serviços são adicionados com sucesso.
*   Total da OS é recalculado.
*   Auditoria é registrada.

#### 2.8.5 Adicionar Produtos à OS

**Objetivo:** Adicionar um ou mais produtos/peças a uma Ordem de Serviço.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID), `products` (array de objetos com `productId` (opcional), `description`, `quantity`, `unitPrice`, `discount`).

**Regras de Negócio:**
*   A OS não pode estar em status `completed`, `delivered` ou `canceled`.
*   `productId` (se fornecido) deve ser um produto existente no catálogo do tenant.
*   `quantity` e `unitPrice` devem ser positivos.
*   A `quantity` não pode exceder o `stockQuantity` disponível (se `productId` for do catálogo).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.
*   `products`: Array não vazio.
*   Cada item de produto: `description` obrigatório, `quantity` e `unitPrice` numéricos e positivos.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`, `products`.
2.  Verifica permissões e status da OS.
3.  Para cada produto:
    *   Valida os dados e o estoque.
    *   Chama `WorkOrderRepository.addWorkOrderProduct(tenantId, workOrderId, productData)`.
    *   Atualiza o `stockQuantity` do produto no catálogo (se for do catálogo).
4.  Atualiza o valor total da OS.
5.  Registra auditoria.
6.  Retorna a OS atualizada com os produtos.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS ou produto do catálogo não encontrado.
*   `409 Conflict`: OS em status finalizado, estoque insuficiente.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Produtos adicionados à OS, total da OS atualizado, estoque atualizado.

**Critérios de Aceite:**
*   Produtos são adicionados com sucesso.
*   Total da OS é recalculado.
*   Estoque é atualizado corretamente.
*   Auditoria é registrada.

#### 2.8.6 Finalizar OS

**Objetivo:** Marcar uma Ordem de Serviço como concluída e gerar contas a receber.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID).

**Regras de Negócio:**
*   A OS deve estar no status `in_progress` ou `approved`.
*   Apenas `owner` ou `admin` podem finalizar.
*   Gera um registro em `accounts_receivable` para o valor total da OS.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`.
2.  Verifica permissões e status da OS.
3.  Chama `WorkOrderService.finalizeWorkOrder(tenantId, workOrderId)` (que altera o status para `completed`).
4.  Cria um registro em `accounts_receivable` com o valor total da OS.
5.  Registra histórico e auditoria.
6.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS não encontrada.
*   `409 Conflict`: OS não está em status válido para finalização.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** OS com status `completed`, conta a receber gerada.

**Critérios de Aceite:**
*   OS é finalizada com sucesso.
*   Conta a receber é gerada corretamente.
*   Auditoria é registrada.

#### 2.8.7 Reabrir OS

**Objetivo:** Mudar o status de uma OS `completed` ou `delivered` de volta para `in_progress`.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID).

**Regras de Negócio:**
*   A OS deve estar no status `completed` ou `delivered`.
*   Apenas `owner` ou `admin` podem reabrir.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`.
2.  Verifica permissões e status da OS.
3.  Chama `WorkOrderService.reopenWorkOrder(tenantId, workOrderId)` (que altera o status para `in_progress`).
4.  Registra histórico e auditoria.
5.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS não encontrada.
*   `409 Conflict`: OS não está em status válido para reabertura.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** OS com status `in_progress`.

**Critérios de Aceite:**
*   OS é reaberta com sucesso.
*   Auditoria é registrada.

#### 2.8.8 Cancelar OS

**Objetivo:** Marcar uma Ordem de Serviço como cancelada.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID).

**Regras de Negócio:**
*   A OS não pode estar em status `completed` ou `delivered`.
*   Apenas `owner` ou `admin` podem cancelar.
*   Se houver produtos alocados, o estoque deve ser revertido.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`.
2.  Verifica permissões e status da OS.
3.  Chama `WorkOrderService.cancelWorkOrder(tenantId, workOrderId)` (que altera o status para `canceled`).
4.  Reverte o estoque de produtos alocados.
5.  Registra histórico e auditoria.
6.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS não encontrada.
*   `409 Conflict`: OS em status finalizado.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** OS com status `canceled`, estoque revertido.

**Critérios de Aceite:**
*   OS é cancelada com sucesso.
*   Estoque de produtos é revertido.
*   Auditoria é registrada.

### 2.9 Financeiro

#### 2.9.1 Criar Receita (Manual)

**Objetivo:** Registrar uma nova receita manual no sistema.

**Entrada:** `tenantId` (UUID), `data` (objeto com `description`, `amount`, `transactionDate`, `paymentMethod`, `customerId` (opcional)).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem criar receitas.
*   `amount` deve ser positivo.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `description`: Obrigatório.
*   `amount`: Numérico, positivo.
*   `transactionDate`: Data válida.
*   `paymentMethod`: Valor válido do enum `payment_method`.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `FinancialRepository.createFinancialTransaction(tenantId, data, type=\'income\')`.
5.  Registra auditoria.
6.  Retorna a transação criada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Nova transação de receita registrada.

**Critérios de Aceite:**
*   Receita manual é criada com sucesso.
*   Auditoria é registrada.

#### 2.9.2 Criar Despesa (Manual)

**Objetivo:** Registrar uma nova despesa manual no sistema.

**Entrada:** `tenantId` (UUID), `data` (objeto com `description`, `amount`, `transactionDate`, `paymentMethod`, `supplier` (opcional), `category` (opcional)).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem criar despesas.
*   `amount` deve ser positivo.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `description`: Obrigatório.
*   `amount`: Numérico, positivo.
*   `transactionDate`: Data válida.
*   `paymentMethod`: Valor válido do enum `payment_method`.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `FinancialRepository.createFinancialTransaction(tenantId, data, type=\'expense\')`.
5.  Registra auditoria.
6.  Retorna a transação criada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Nova transação de despesa registrada.

**Critérios de Aceite:**
*   Despesa manual é criada com sucesso.
*   Auditoria é registrada.

#### 2.9.3 Contas a Receber

**Objetivo:** Gerenciar e listar todas as contas a receber do tenant.

**Entrada:** `tenantId` (UUID), `filters` (objeto com `status`, `dueDateRange`, `customerId`, `page`, `pageSize`).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem acessar.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `page`, `pageSize`: Numéricos, positivos.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `filters`.
2.  Verifica permissões.
3.  Chama `FinancialRepository.findAccountsReceivable(tenantId, filters)`.
4.  Retorna lista de contas a receber e total para paginação.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista paginada de contas a receber.

**Critérios de Aceite:**
*   Listagem funciona com filtros e paginação.
*   Permissões são respeitadas.

#### 2.9.4 Contas a Pagar

**Objetivo:** Gerenciar e listar todas as contas a pagar do tenant.

**Entrada:** `tenantId` (UUID), `filters` (objeto com `status`, `dueDateRange`, `supplier`, `page`, `pageSize`).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem acessar.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `page`, `pageSize`: Numéricos, positivos.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `filters`.
2.  Verifica permissões.
3.  Chama `FinancialRepository.findAccountsPayable(tenantId, filters)`.
4.  Retorna lista de contas a pagar e total para paginação.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista paginada de contas a pagar.

**Critérios de Aceite:**
*   Listagem funciona com filtros e paginação.
*   Permissões são respeitadas.

#### 2.9.5 Fluxo de Caixa

**Objetivo:** Visualizar o histórico de transações financeiras (receitas e despesas).

**Entrada:** `tenantId` (UUID), `filters` (objeto com `dateRange`, `type`, `paymentMethod`, `page`, `pageSize`).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem acessar.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `page`, `pageSize`: Numéricos, positivos.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `filters`.
2.  Verifica permissões.
3.  Chama `FinancialRepository.findFinancialTransactions(tenantId, filters)`.
4.  Retorna lista de transações e total para paginação.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista paginada de transações financeiras.

**Critérios de Aceite:**
*   Listagem funciona com filtros e paginação.
*   Permissões são respeitadas.

### 2.10 Assinaturas

#### 2.10.1 Iniciar Trial

**Objetivo:** Ativar o período de teste gratuito para um novo tenant.

**Entrada:** `tenantId` (UUID).

**Regras de Negócio:**
*   Apenas tenants recém-criados ou sem assinatura ativa podem iniciar trial.
*   O trial tem uma duração fixa (ex: 14 dias).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.

**Permissões:** Interno (invocado pelo caso de uso de cadastro).

**Fluxo:**
1.  Recebe `tenantId`.
2.  Verifica se o tenant pode iniciar trial.
3.  Chama `SubscriptionRepository.createTrialSubscription(tenantId)`.
4.  Atualiza o status do tenant para `trialing`.
5.  Registra auditoria.
6.  Retorna a assinatura de trial.

**Erros Possíveis:**
*   `409 Conflict`: Tenant já possui assinatura ou trial.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Assinatura de trial criada.

**Critérios de Aceite:**
*   Trial é iniciado com sucesso para novos tenants.
*   Auditoria é registrada.

#### 2.10.2 Upgrade de Plano

**Objetivo:** Mudar a assinatura do tenant para um plano superior.

**Entrada:** `tenantId` (UUID), `newPlanId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` do tenant pode fazer upgrade.
*   O `newPlanId` deve ser um plano ativo e de valor superior ao atual.
*   Integração com Stripe para gerenciar a mudança de plano e prorrogação.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `newPlanId`: Obrigatório, UUID válido, plano ativo.

**Permissões:** `owner` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `newPlanId`.
2.  Verifica permissões e validade do novo plano.
3.  Chama `StripeService.updateSubscriptionPlan(stripeSubscriptionId, newStripePriceId)`.
4.  Em caso de sucesso na Stripe, atualiza `subscriptions` no banco de dados.
5.  Registra auditoria.
6.  Retorna a assinatura atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Plano ou assinatura não encontrada.
*   `422 Unprocessable Entity`: Plano inválido.
*   `500 Internal Server Error`: Erro na integração com Stripe.

**Resultado Esperado:** Assinatura atualizada para o novo plano.

**Critérios de Aceite:**
*   Upgrade de plano é processado com sucesso na Stripe e no sistema.
*   Auditoria é registrada.

#### 2.10.3 Downgrade de Plano

**Objetivo:** Mudar a assinatura do tenant para um plano inferior.

**Entrada:** `tenantId` (UUID), `newPlanId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` do tenant pode fazer downgrade.
*   O `newPlanId` deve ser um plano ativo e de valor inferior ao atual.
*   Integração com Stripe para gerenciar a mudança de plano (geralmente com prorrogação ou crédito).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `newPlanId`: Obrigatório, UUID válido, plano ativo.

**Permissões:** `owner` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `newPlanId`.
2.  Verifica permissões e validade do novo plano.
3.  Chama `StripeService.updateSubscriptionPlan(stripeSubscriptionId, newStripePriceId)`.
4.  Em caso de sucesso na Stripe, atualiza `subscriptions` no banco de dados.
5.  Registra auditoria.
6.  Retorna a assinatura atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Plano ou assinatura não encontrada.
*   `422 Unprocessable Entity`: Plano inválido.
*   `500 Internal Server Error`: Erro na integração com Stripe.

**Resultado Esperado:** Assinatura atualizada para o novo plano.

**Critérios de Aceite:**
*   Downgrade de plano é processado com sucesso na Stripe e no sistema.
*   Auditoria é registrada.

#### 2.10.4 Cancelamento de Assinatura

**Objetivo:** Cancelar a assinatura ativa de um tenant.

**Entrada:** `tenantId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` do tenant pode cancelar.
*   A assinatura é cancelada na Stripe e marcada como `canceled` no sistema.
*   O acesso ao sistema é mantido até o final do período de faturamento atual.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.

**Permissões:** `owner` do tenant.

**Fluxo:**
1.  Recebe `tenantId`.
2.  Verifica permissões.
3.  Chama `StripeService.cancelSubscription(stripeSubscriptionId)`.
4.  Em caso de sucesso na Stripe, atualiza `subscriptions` no banco de dados para `canceled` e define `cancel_at_period_end`.
5.  Registra auditoria.
6.  Retorna a assinatura atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Assinatura não encontrada.
*   `500 Internal Server Error`: Erro na integração com Stripe.

**Resultado Esperado:** Assinatura cancelada, acesso mantido até o fim do período.

**Critérios de Aceite:**
*   Assinatura é cancelada com sucesso na Stripe e no sistema.
*   Acesso é mantido até o final do período de faturamento.
*   Auditoria é registrada.

## 3. Serviços

Esta seção detalha os serviços que encapsulam a lógica de negócio e as interações com o banco de dados e serviços externos. Cada serviço será implementado como uma classe ou módulo.

### 3.1 `AuthService`

**Responsabilidades:** Gerenciar a autenticação de usuários via Supabase Auth.

**Métodos:**
*   `signInWithPassword(email, password)`: Autentica o usuário.
*   `signUp(email, password)`: Registra um novo usuário.
*   `sendPasswordResetEmail(email)`: Envia e-mail de recuperação de senha.
*   `updateUserPassword(newPassword)`: Atualiza a senha do usuário.
*   `signOut()`: Desloga o usuário.
*   `inviteUserByEmail(email)`: Envia convite para novo usuário.

**Dependências:** Supabase Auth client.

**Regras:** Interage diretamente com o Supabase Auth para todas as operações de autenticação.

### 3.2 `TenantService`

**Responsabilidades:** Gerenciar a criação, atualização e recuperação de informações dos tenants e seus usuários.

**Métodos:**
*   `createTenant(companyName, ownerUserId)`: Cria um novo tenant e associa o owner.
*   `updateTenant(tenantId, data)`: Atualiza informações do tenant.
*   `getTenantById(tenantId)`: Recupera informações de um tenant.
*   `addTenantUser(tenantId, userId, role)`: Adiciona um usuário a um tenant.
*   `updateTenantUserRole(tenantId, userId, newRole)`: Atualiza a role de um usuário no tenant.
*   `removeTenantUser(tenantId, userId)`: Remove um usuário do tenant.

**Dependências:** `TenantRepository`, `UserRepository`.

**Regras:** Aplica regras de negócio relacionadas a tenants e usuários (ex: unicidade de nome, limites de usuários, transições de role).

### 3.3 `CustomerService`

**Responsabilidades:** Gerenciar o CRUD de clientes.

**Métodos:**
*   `createCustomer(tenantId, data)`: Cria um novo cliente.
*   `updateCustomer(tenantId, customerId, data)`: Atualiza um cliente.
*   `softDeleteCustomer(tenantId, customerId)`: Arquiva um cliente.
*   `findCustomers(tenantId, filters)`: Busca e filtra clientes.
*   `getCustomerById(tenantId, customerId)`: Recupera um cliente específico.

**Dependências:** `CustomerRepository`.

**Regras:** Validações de dados do cliente, unicidade de documento, verifica OS abertas antes de arquivar.

### 3.4 `VehicleService`

**Responsabilidades:** Gerenciar o CRUD de veículos e seu histórico.

**Métodos:**
*   `createVehicle(tenantId, data)`: Cria um novo veículo.
*   `updateVehicle(tenantId, vehicleId, data)`: Atualiza um veículo.
*   `softDeleteVehicle(tenantId, vehicleId)`: Arquiva um veículo.
*   `findVehicles(tenantId, filters)`: Busca e filtra veículos.
*   `getVehicleById(tenantId, vehicleId)`: Recupera um veículo específico.
*   `getVehicleWorkOrderHistory(tenantId, vehicleId)`: Recupera histórico de OS do veículo.

**Dependências:** `VehicleRepository`, `WorkOrderRepository`.

**Regras:** Validações de dados do veículo, unicidade de placa, verifica OS abertas antes de arquivar.

### 3.5 `ServiceCatalogService`

**Responsabilidades:** Gerenciar o CRUD do catálogo de serviços da oficina.

**Métodos:**
*   `createService(tenantId, data)`: Cria um novo serviço.
*   `updateService(tenantId, serviceId, data)`: Atualiza um serviço.
*   `softDeleteService(tenantId, serviceId)`: Arquiva um serviço.
*   `findServices(tenantId, filters)`: Busca e filtra serviços.
*   `getServiceById(tenantId, serviceId)`: Recupera um serviço específico.

**Dependências:** `ServiceCatalogRepository`.

**Regras:** Validações de dados do serviço, unicidade de nome, verifica uso em OS abertas antes de arquivar.

### 3.6 `ProductCatalogService`

**Responsabilidades:** Gerenciar o CRUD do catálogo de produtos/peças da oficina.

**Métodos:**
*   `createProduct(tenantId, data)`: Cria um novo produto.
*   `updateProduct(tenantId, productId, data)`: Atualiza um produto.
*   `softDeleteProduct(tenantId, productId)`: Arquiva um produto.
*   `findProducts(tenantId, filters)`: Busca e filtra produtos.
*   `getProductById(tenantId, productId)`: Recupera um produto específico.

**Dependências:** `ProductCatalogRepository`.

**Regras:** Validações de dados do produto, unicidade de nome/SKU, verifica uso em OS abertas antes de arquivar, gerencia estoque.

### 3.7 `WorkOrderService`

**Responsabilidades:** Gerenciar o ciclo de vida completo das Ordens de Serviço.

**Métodos:**
*   `createWorkOrder(tenantId, data)`: Abre uma nova OS.
*   `updateWorkOrder(tenantId, workOrderId, data)`: Atualiza dados gerais da OS.
*   `updateWorkOrderStatus(tenantId, workOrderId, newStatus)`: Altera o status da OS.
*   `approveWorkOrder(tenantId, workOrderId)`: Aprova uma OS.
*   `addWorkOrderService(tenantId, workOrderId, serviceData)`: Adiciona serviço à OS.
*   `addWorkOrderProduct(tenantId, workOrderId, productData)`: Adiciona produto à OS.
*   `finalizeWorkOrder(tenantId, workOrderId)`: Finaliza a OS e gera conta a receber.
*   `reopenWorkOrder(tenantId, workOrderId)`: Reabre uma OS.
*   `cancelWorkOrder(tenantId, workOrderId)`: Cancela uma OS.
*   `getWorkOrderById(tenantId, workOrderId)`: Recupera uma OS completa.
*   `findWorkOrders(tenantId, filters)`: Busca e filtra OS.

**Dependências:** `WorkOrderRepository`, `CustomerRepository`, `VehicleRepository`, `ServiceCatalogRepository`, `ProductCatalogRepository`, `FinancialService`.

**Regras:** Validações de fluxo de status, estoque de produtos, cálculos de totais da OS, geração de contas a receber.

### 3.8 `FinancialService`

**Responsabilidades:** Gerenciar transações financeiras, contas a receber e a pagar.

**Métodos:**
*   `createIncome(tenantId, data)`: Registra uma receita manual.
*   `createExpense(tenantId, data)`: Registra uma despesa manual.
*   `registerPayment(tenantId, receivableId, amount, paymentMethod)`: Registra pagamento de conta a receber.
*   `findAccountsReceivable(tenantId, filters)`: Busca contas a receber.
*   `findAccountsPayable(tenantId, filters)`: Busca contas a pagar.
*   `findFinancialTransactions(tenantId, filters)`: Busca transações de fluxo de caixa.

**Dependências:** `FinancialRepository`, `AccountsReceivableRepository`, `AccountsPayableRepository`.

**Regras:** Validações de valores, datas, métodos de pagamento. Atualização de status de contas.

### 3.9 `SubscriptionService`

**Responsabilidades:** Gerenciar o ciclo de vida das assinaturas do tenant, incluindo trial, upgrade, downgrade e cancelamento.

**Métodos:**
*   `startTrial(tenantId)`: Inicia o período de trial.
*   `upgradeSubscription(tenantId, newPlanId)`: Faz upgrade de plano.
*   `downgradeSubscription(tenantId, newPlanId)`: Faz downgrade de plano.
*   `cancelSubscription(tenantId)`: Cancela a assinatura.
*   `getSubscriptionDetails(tenantId)`: Recupera detalhes da assinatura atual.
*   `handleStripeWebhook(event)`: Processa eventos de webhook da Stripe.

**Dependências:** `SubscriptionRepository`, `StripeService`, `TenantRepository`.

**Regras:** Validações de planos, limites de usuários, sincronização com Stripe, regras de prorrogação/crédito.

### 3.10 `NotificationService`

**Responsabilidades:** Gerenciar a criação e entrega de notificações internas e por e-mail.

**Métodos:**
*   `createNotification(tenantId, userId, type, title, message)`: Cria uma notificação interna.
*   `sendEmailNotification(to, subject, body)`: Envia notificação por e-mail.
*   `markNotificationAsRead(tenantId, notificationId)`: Marca notificação como lida.

**Dependências:** `NotificationRepository`, serviço de e-mail (ex: Resend, Nodemailer).

**Regras:** Define tipos de notificação, prioridades, e regras de entrega.

### 3.11 `AuditService`

**Responsabilidades:** Registrar e gerenciar logs de auditoria.

**Métodos:**
*   `createAuditLog(tenantId, userId, action, entityType, entityId, oldData, newData, ipAddress, userAgent)`: Cria um registro de auditoria.
*   `findAuditLogs(tenantId, filters)`: Busca e filtra logs de auditoria.

**Dependências:** `AuditLogRepository`.

**Regras:** Define quais ações são auditáveis e o formato dos logs.

### 3.12 `SettingsService`

**Responsabilidades:** Gerenciar as configurações do tenant.

**Métodos:**
*   `getSetting(tenantId, key)`: Recupera uma configuração.
*   `updateSetting(tenantId, key, value)`: Atualiza uma configuração.

**Dependências:** `SettingsRepository`.

**Regras:** Validações de configurações, permissões de acesso.

## 4. Integração com Supabase

### 4.1 Queries

Todas as interações com o banco de dados PostgreSQL serão realizadas através do cliente Supabase, utilizando o construtor de queries fluente ou funções SQL diretas (RPCs) para operações mais complexas.

*   **Cliente Supabase:** `createClient` do `@supabase/supabase-js` será usado para interagir com o banco de dados.
*   **Tipagem:** Utilização de tipos gerados automaticamente pelo Supabase CLI (`supabase gen types typescript --schema public > types/supabase.ts`) para garantir segurança de tipo nas queries.
*   **`tenant_id`:** Todas as queries em tabelas multi-tenant incluirão uma cláusula `WHERE tenant_id = current_tenant_id` (implicitamente via RLS ou explicitamente em casos específicos).
*   **Transações:** Operações que envolvem múltiplas escritas ou atualizações serão encapsuladas em transações para garantir atomicidade.

### 4.2 Policies RLS

As políticas de Row Level Security (RLS) são configuradas diretamente no PostgreSQL via Supabase para impor o isolamento de dados por tenant e o controle de acesso baseado em role.

*   **Habilitação:** RLS será habilitado para todas as tabelas multi-tenant.
*   **Funções de Contexto:** Funções como `auth.uid()` e `auth.jwt()` serão usadas nas políticas para obter o `user_id` do usuário autenticado e, a partir dele, o `tenant_id` associado (via `tenant_users`).
*   **Políticas por Operação:** Políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` serão definidas para cada tabela, garantindo que o usuário só possa operar em dados do seu `tenant_id` e de acordo com sua `role`.
*   **Exemplo de Política (SELECT para `customers`):**
    ```sql
    CREATE POLICY "Tenants can view their own customers" ON customers
    FOR SELECT USING (tenant_id = (SELECT tenant_id FROM tenant_users WHERE user_id = auth.uid()));
    ```
*   **Exemplo de Política (INSERT para `customers`):**
    ```sql
    CREATE POLICY "Tenants can create customers" ON customers
    FOR INSERT WITH CHECK (tenant_id = (SELECT tenant_id FROM tenant_users WHERE user_id = auth.uid()));
    ```

### 4.3 Uploads (Supabase Storage)

O Supabase Storage será utilizado para armazenar arquivos como logos de oficinas, fotos de veículos e anexos de ordens de serviço.

*   **Buckets:** Serão criados buckets específicos (ex: `logos`, `vehicle-photos`, `work-order-attachments`).
*   **Políticas de Acesso:** Políticas de segurança serão configuradas para cada bucket para controlar quem pode fazer upload, download e visualizar arquivos, geralmente baseadas no `tenant_id` e `user_id`.
*   **Geração de URLs:** URLs públicas ou assinadas serão geradas para acesso aos arquivos, conforme a necessidade.

### 4.4 Transactions

Para garantir a integridade dos dados em operações que envolvem múltiplas modificações, as transações serão utilizadas.

*   **Uso:** Em casos de uso como `finalizeWorkOrder` (que atualiza o status da OS e cria uma conta a receber) ou `addWorkOrderProduct` (que adiciona um produto à OS e atualiza o estoque).
*   **Implementação:** O cliente Supabase permite o uso de transações de banco de dados para agrupar operações.

### 4.5 Soft Delete

Em vez de excluir permanentemente registros, o sistema utilizará o conceito de soft delete para a maioria das entidades operacionais (clientes, veículos, serviços, produtos, usuários de tenant).

*   **Coluna `deleted_at`:** Adição de uma coluna `deleted_at` (timestamp com timezone, nullable) em tabelas relevantes.
*   **Lógica de Negócio:** Ao invés de `DELETE`, as operações de "exclusão" irão atualizar a coluna `deleted_at` com o timestamp atual e, opcionalmente, mudar o `status` para `inactive`.
*   **Queries:** Todas as queries de leitura incluirão `WHERE deleted_at IS NULL` para filtrar registros ativos por padrão.
*   **RLS:** As políticas de RLS também considerarão a coluna `deleted_at`.

## 5. Integração com Stripe

A integração com a Stripe é crucial para o gerenciamento de assinaturas e pagamentos.

### 5.1 Checkout

Para a criação de novas assinaturas ou upgrades, o sistema utilizará o Stripe Checkout.

*   **Sessões de Checkout:** O backend criará sessões de checkout na Stripe, especificando o plano (price ID), o cliente (customer ID) e URLs de sucesso/cancelamento.
*   **Redirecionamento:** O frontend redirecionará o usuário para a URL da sessão de checkout da Stripe.

### 5.2 Customer Portal

O Stripe Customer Portal será utilizado para permitir que os usuários gerenciem suas informações de faturamento, métodos de pagamento e histórico de faturas diretamente na Stripe.

*   **Geração de Link:** O backend gerará um link para o Customer Portal para o `stripe_customer_id` do tenant.
*   **Redirecionamento:** O frontend redirecionará o usuário para este link.

### 5.3 Trial

O período de trial é gerenciado tanto no sistema quanto na Stripe.

*   **Criação de Assinatura com Trial:** Ao iniciar o trial, uma assinatura é criada na Stripe com `trial_period_days`.
*   **Sincronização:** Webhooks da Stripe (ex: `customer.subscription.trial_will_end`, `customer.subscription.updated`) são usados para sincronizar o status do trial no sistema.

### 5.4 Assinaturas (Upgrade/Downgrade)

*   **API da Stripe:** O `StripeService` utilizará a API da Stripe para atualizar assinaturas existentes, alterando o `price_id` do plano.
*   **Prorrogação:** A Stripe gerencia automaticamente a prorrogação e os créditos/débitos resultantes de upgrades/downgrades.
*   **Webhooks:** Eventos `customer.subscription.updated` são essenciais para manter o sistema sincronizado com as mudanças de plano.

### 5.5 Cancelamentos

*   **API da Stripe:** O `StripeService` chamará a API da Stripe para cancelar assinaturas.
*   **`cancel_at_period_end`:** Assinaturas são geralmente canceladas no final do período de faturamento atual para evitar cobranças inesperadas.
*   **Webhooks:** Eventos `customer.subscription.deleted` ou `customer.subscription.updated` (com `status=\'canceled\'`) são usados para refletir o cancelamento no sistema.

### 5.6 Webhooks

Os webhooks da Stripe são fundamentais para manter o estado do sistema sincronizado com as transações e eventos de assinatura na Stripe.

*   **Endpoint:** Um Route Handler (`/api/stripe/webhook`) será configurado para receber os eventos da Stripe.
*   **Segurança:** O endpoint verificará a assinatura do webhook para garantir que a requisição é legítima da Stripe.
*   **Idempotência:** O processamento de webhooks será idempotente para lidar com entregas duplicadas. Cada evento da Stripe tem um ID único que pode ser usado para evitar o reprocessamento.
*   **Eventos Chave:**
    *   `checkout.session.completed`: Acionado após um checkout bem-sucedido. Usado para criar ou atualizar a assinatura no sistema e associar o `stripe_customer_id` e `stripe_subscription_id` ao tenant.
    *   `customer.subscription.created`: Confirma a criação de uma nova assinatura. Usado para atualizar o status da assinatura no sistema.
    *   `customer.subscription.updated`: Acionado em qualquer mudança na assinatura (status, plano, trial). Usado para sincronizar o `subscription_status`, `current_period_end`, `stripe_price_id` e outros detalhes.
    *   `customer.subscription.deleted`: Acionado quando uma assinatura é cancelada ou expira. Usado para marcar a assinatura como `canceled` e, potencialmente, desativar o tenant se não houver outra assinatura ativa.
    *   `invoice.payment_failed`: Acionado quando um pagamento de fatura falha. Usado para notificar o usuário, atualizar o `subscription_status` para `past_due` e, se necessário, iniciar o processo de dunning.

*   **Sincronização de Status:** O `SubscriptionService` processará esses webhooks para atualizar o status da assinatura do tenant na tabela `subscriptions` e, consequentemente, o `status` do tenant na tabela `tenants`.

## 6. Auditoria

O sistema de auditoria registra todas as ações significativas para garantir rastreabilidade, conformidade e segurança.

### 6.1 Registrar

Cada registro de auditoria na tabela `audit_logs` conterá:

*   **`user_id`:** Quem executou a ação.
*   **`tenant_id`:** A qual tenant a ação pertence.
*   **`action`:** Tipo de ação (CREATE, UPDATE, DELETE, LOGIN, LOGOUT, etc.).
*   **`entity_type`:** Tabela ou entidade afetada (ex: `customers`, `work_orders`, `settings`).
*   **`entity_id`:** ID do registro afetado.
*   **`old_data` (JSONB):** Estado do registro antes da alteração (para UPDATE/DELETE).
*   **`new_data` (JSONB):** Estado do registro após a alteração (para CREATE/UPDATE).
*   **`ip_address`:** Endereço IP de origem da requisição.
*   **`user_agent`:** User agent do cliente.
*   **`created_at`:** Quando a ação foi executada.

### 6.2 Implementação

*   **Triggers de Banco de Dados:** Para operações de CRUD de baixo nível, triggers `AFTER INSERT`, `AFTER UPDATE`, `AFTER DELETE` podem ser usados para popular a tabela `audit_logs` automaticamente.
*   **Lógica de Serviço/Caso de Uso:** Para ações mais complexas ou que envolvem múltiplos passos, a lógica de registro de auditoria será incorporada nos `use-cases` ou `services` (ex: `AuditService.createAuditLog()`).

## 7. Logs

Uma estratégia de logging eficaz é vital para a saúde e monitoramento do sistema.

### 7.1 Estratégia

*   **Níveis de Log:** Utilização de níveis de log padrão (DEBUG, INFO, WARN, ERROR, FATAL).
*   **Formato:** Logs em formato JSON para facilitar a ingestão e análise por ferramentas de agregação.
*   **Contexto:** Cada log incluirá contexto relevante como `tenant_id`, `user_id`, `requestId`, `module`, `function`, `errorStack`.
*   **Ferramentas:** Biblioteca de logging (ex: Pino) integrada com um serviço de monitoramento (ex: Vercel Log Drains, Sentry).

### 7.2 Tipos de Logs

*   **Logs de Erro:** Capturam todas as exceções não tratadas e erros esperados, com stack traces completos.
*   **Logs de Sistema:** Registram eventos operacionais importantes (ex: inicialização de serviços, chamadas de API externas, processamento de webhooks).
*   **Logs Financeiros:** Detalhes de transações financeiras, status de pagamento, eventos de faturamento da Stripe.

## 8. Uploads

O gerenciamento de uploads de arquivos será feito via Supabase Storage.

### 8.1 Documentação

*   **Logo da Oficina:** Armazenada em um bucket `logos`. A URL será salva na tabela `tenants`.
*   **Fotos dos Veículos:** Armazenadas em um bucket `vehicle-photos`. URLs salvas na tabela `vehicles`.
*   **Anexos da OS:** Documentos, imagens relacionadas a uma OS. Armazenados em um bucket `work-order-attachments`. URLs salvas na tabela `work_order_attachments` (nova tabela).

### 8.2 Processo

1.  Frontend envia o arquivo para um Server Action.
2.  Server Action utiliza o cliente Supabase Storage para fazer o upload do arquivo para o bucket apropriado.
3.  A URL pública (ou assinada, se necessário) do arquivo é salva no banco de dados.
4.  Políticas de segurança do Storage garantem que apenas usuários autorizados (do tenant correto) possam acessar/gerenciar seus arquivos.

## 9. Notificações

O sistema de notificações informará os usuários sobre eventos importantes.

### 9.1 Tipos

*   **Notificações Internas:** Exibidas dentro da interface do usuário (ex: no sino de notificações, na página `/notifications`).
*   **E-mail:** Enviadas para o endereço de e-mail do usuário (ex: confirmação de cadastro, alerta de pagamento falho, resumo semanal).
*   **Futuro WhatsApp:** Considerado para futuras integrações (ex: lembretes de OS, status de veículo).

### 9.2 Implementação

*   **`NotificationService`:** Orquestra a criação e envio de notificações.
*   **Templates:** Utilização de templates para e-mails e mensagens para garantir consistência.
*   **Fila de Mensagens:** Para notificações de alto volume ou assíncronas, pode-se considerar uma fila de mensagens (ex: Redis, Supabase Edge Functions com Deno Deploy).

## 10. Segurança

A segurança é um pilar fundamental do OficinaPro, abordada em múltiplas camadas.

### 10.1 Controle de Acesso

*   **Autenticação:** Gerenciada pelo Supabase Auth (JWT).
*   **Autorização:** Baseada em roles (`tenant_role` enum) e permissões granulares.
*   **RLS:** Imposto no nível do banco de dados para isolamento de dados por tenant.

### 10.2 Permissões por Perfil

*   **`owner`:** Acesso total a todas as funcionalidades e configurações do tenant.
*   **`admin`:** Acesso quase total, mas com restrições em operações críticas (ex: exclusão de owner, cancelamento de assinatura).
*   **`employee`:** Acesso restrito a funcionalidades operacionais (CRUD de clientes, veículos, OS), sem acesso a configurações financeiras ou de tenant.
*   **Implementação:** Verificações de permissão na camada de `use-cases` e nas políticas de RLS.

### 10.3 Proteção Contra Acesso Entre Tenants

*   **RLS:** Principal mecanismo de proteção.
*   **Validação de `tenant_id`:** Todas as queries e operações de escrita/leitura no backend devem incluir o `tenant_id` do usuário autenticado.
*   **Testes:** Testes de integração e segurança para garantir que nenhum dado de um tenant possa ser acessado por outro.

### 10.4 Rate Limiting

*   **Objetivo:** Proteger o backend contra ataques de força bruta e uso excessivo de recursos.
*   **Implementação:** Pode ser configurado no nível do Vercel, ou implementado em Route Handlers específicos para rotas sensíveis (ex: login, registro).

### 10.5 Sanitização e Validação de Entrada

*   **Validação:** Todos os dados de entrada são validados usando Zod para garantir o formato e tipo corretos.
*   **Sanitização:** Dados de entrada (especialmente strings) são sanitizados para prevenir ataques como XSS e SQL Injection (o uso de prepared statements pelo Supabase client já ajuda contra SQL Injection).

### 10.6 Proteção Contra Duplicidade

*   **Constraints de Banco de Dados:** `UNIQUE` constraints em colunas como `email` (para usuários), `document` (para clientes), `plate` (para veículos), `name`/`sku` (para produtos/serviços) combinadas com `tenant_id`.
*   **Lógica de Negócio:** Verificações adicionais na camada de `use-cases` antes de criar novos registros.

## 11. Performance

A performance é otimizada em várias frentes para garantir uma experiência de usuário rápida e responsiva.

### 11.1 Paginação

*   **Queries Paginadas:** Todas as listagens de dados (clientes, veículos, OS, produtos, serviços, logs) utilizarão paginação (`LIMIT` e `OFFSET` ou `cursor-based pagination`).
*   **Contagem Total:** Queries separadas para obter a contagem total de registros para a paginação.

### 11.2 Cache

*   **Cache de Dados:** Dados frequentemente acessados e que não mudam com frequência (ex: configurações de tenant, planos de assinatura) podem ser cacheados em memória ou em um serviço de cache (ex: Redis).
*   **Cache de Queries:** O Supabase e o PostgreSQL possuem mecanismos de cache internos que podem ser aproveitados.

### 11.3 Índices

*   **Índices de Banco de Dados:** Criação de índices apropriados em colunas frequentemente usadas em cláusulas `WHERE`, `ORDER BY` e `JOIN`.
*   **Índices Compostos:** Para tabelas multi-tenant, índices compostos começando com `tenant_id` são cruciais para otimizar queries.

### 11.4 Lazy Loading

*   **Relacionamentos:** Carregamento de dados relacionados (ex: serviços e produtos de uma OS) apenas quando necessário, para evitar o carregamento excessivo de dados.
*   **Componentes Frontend:** Utilização de lazy loading no frontend para componentes e rotas.

## 12. Estratégia de Erros

Uma estratégia de tratamento de erros consistente é fundamental para a robustez do backend e para fornecer feedback útil ao frontend.

### 12.1 Padrões de Erro

*   **Erros Customizados:** Definição de classes de erro customizadas para erros de negócio específicos (ex: `TenantNotFound`, `PermissionDenied`, `InvalidStatusTransition`).
*   **Códigos HTTP:** Mapeamento de erros para códigos de status HTTP apropriados.

### 12.2 Códigos de Status HTTP

*   **`400 Bad Request`:** Requisição malformada, validação de entrada falhou.
*   **`401 Unauthorized`:** Falha na autenticação (credenciais inválidas, token ausente/expirado).
*   **`403 Forbidden`:** Usuário autenticado, mas sem permissão para executar a ação (autorização falhou).
*   **`404 Not Found`:** Recurso solicitado não encontrado.
*   **`409 Conflict`:** Conflito de recursos (ex: e-mail já registrado, placa duplicada).
*   **`422 Unprocessable Entity`:** Validação de regras de negócio falhou (ex: transição de status inválida).
*   **`500 Internal Server Error`:** Erro inesperado no servidor.

### 12.3 Tratamento de Erros Global

*   Um middleware ou interceptor global para capturar e formatar erros, garantindo que o frontend receba respostas de erro consistentes.

## 13. Critérios de Aceite

Para cada módulo e caso de uso, os critérios de aceite definidos nas seções anteriores servem como base para validar a implementação. Além disso, os seguintes critérios gerais se aplicam:

*   **Funcionalidade:** Todas as funcionalidades descritas operam conforme o esperado.
*   **Segurança:** O isolamento de dados por tenant é garantido. As permissões de usuário são respeitadas. O sistema é resistente a vulnerabilidades comuns.
*   **Performance:** As operações críticas respondem dentro dos limites de tempo aceitáveis.
*   **Confiabilidade:** O sistema lida com erros de forma graciosa e recupera-se de falhas.
*   **Manutenibilidade:** O código é limpo, modular e fácil de entender e modificar.
*   **Escalabilidade:** A arquitetura suporta o crescimento do número de tenants e usuários.

## Conclusão

Este documento fornece uma especificação técnica abrangente para o backend do OficinaPro, cobrindo a arquitetura multi-tenant, a estrutura de camadas, os casos de uso detalhados, as integrações com Supabase e Stripe, e as estratégias de segurança, auditoria e performance. A aderência a esta especificação garantirá que o desenvolvimento do backend seja robusto, seguro e escalável, fornecendo uma base sólida para o funcionamento do sistema. Com esta base, as ferramentas de IA (Cursor, Claude, ChatGPT) e os desenvolvedores terão todas as informações necessárias para construir a lógica de negócio do OficinaPro com precisão e qualidade.

### 2.8 Ordens de Serviço

#### 2.8.1 Abrir OS

**Objetivo:** Criar uma nova Ordem de Serviço para um cliente e veículo específicos.

**Entrada:** `tenantId` (UUID), `data` (objeto com `customerId`, `vehicleId`, `clientReport`, `entryMileage`, `expectedDeliveryDate`, `responsibleEmployeeId`, `internalNotes`).

**Regras de Negócio:**
*   `customerId` e `vehicleId` devem existir e pertencer ao `tenantId`.
*   `vehicleId` deve estar associado ao `customerId`.
*   Um número de OS único é gerado automaticamente para o tenant.
*   O status inicial da OS é `draft` ou `open`.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `customerId`: Obrigatório, UUID válido.
*   `vehicleId`: Obrigatório, UUID válido.
*   `clientReport`: Obrigatório.
*   `entryMileage`: Numérico, positivo.
*   `expectedDeliveryDate`: Data futura ou presente.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data` e a associação cliente-veículo.
4.  Gera um número de OS único (`generate_work_order_number()`).
5.  Chama `WorkOrderRepository.createWorkOrder(tenantId, data)`.
6.  Registra auditoria (`create_audit_log()`).
7.  Retorna a OS criada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Cliente ou veículo não encontrado.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Nova Ordem de Serviço criada com status inicial.

**Critérios de Aceite:**
*   OS é criada com sucesso, número único gerado.
*   Cliente e veículo são corretamente associados.
*   Auditoria é registrada.

#### 2.8.2 Alterar Status da OS

**Objetivo:** Mudar o status de uma Ordem de Serviço, seguindo um fluxo predefinido.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID), `newStatus` (`work_order_status` enum), `notes` (string, opcional).

**Regras de Negócio:**
*   Apenas `owner`, `admin` ou `employee` podem alterar o status (com restrições).
*   A transição de status deve seguir o fluxo definido (ex: `draft` -> `open` -> `waiting_approval` -> `approved` -> `in_progress` -> `completed` -> `delivered`).
*   Não é possível mudar para um status anterior (exceto `reopened`).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.
*   `newStatus`: Obrigatório, valor válido do enum `work_order_status`.

**Permissões:** `owner`, `admin`, `employee` do tenant (com restrições de status).

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`, `newStatus` e `notes`.
2.  Verifica permissões e o fluxo de transição de status.
3.  Chama `WorkOrderRepository.updateWorkOrderStatus(tenantId, workOrderId, newStatus)`.
4.  Registra histórico da OS (`work_order_history`) e auditoria.
5.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão ou transição de status inválida.
*   `404 Not Found`: OS não encontrada.
*   `422 Unprocessable Entity`: Status inválido.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Status da OS atualizado, histórico registrado.

**Critérios de Aceite:**
*   Status da OS é alterado com sucesso, respeitando o fluxo.
*   Histórico da OS é registrado.
*   Auditoria é registrada.

#### 2.8.3 Aprovar OS

**Objetivo:** Marcar uma Ordem de Serviço como aprovada pelo cliente.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID).

**Regras de Negócio:**
*   A OS deve estar no status `waiting_approval`.
*   Apenas `owner` ou `admin` podem aprovar.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`.
2.  Verifica permissões e status da OS.
3.  Chama `WorkOrderService.approveWorkOrder(tenantId, workOrderId)` (que altera o status para `approved`).
4.  Registra histórico e auditoria.
5.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS não encontrada.
*   `409 Conflict`: OS não está em `waiting_approval`.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** OS com status `approved`.

**Critérios de Aceite:**
*   OS é aprovada com sucesso.
*   Auditoria é registrada.

#### 2.8.4 Adicionar Serviços à OS

**Objetivo:** Adicionar um ou mais serviços a uma Ordem de Serviço.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID), `services` (array de objetos com `serviceId` (opcional), `description`, `quantity`, `unitPrice`, `discount`).

**Regras de Negócio:**
*   A OS não pode estar em status `completed`, `delivered` ou `canceled`.
*   `serviceId` (se fornecido) deve ser um serviço existente no catálogo do tenant.
*   `quantity` e `unitPrice` devem ser positivos.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.
*   `services`: Array não vazio.
*   Cada item de serviço: `description` obrigatório, `quantity` e `unitPrice` numéricos e positivos.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`, `services`.
2.  Verifica permissões e status da OS.
3.  Para cada serviço:
    *   Valida os dados.
    *   Chama `WorkOrderRepository.addWorkOrderService(tenantId, workOrderId, serviceData)`.
4.  Atualiza o valor total da OS.
5.  Registra auditoria.
6.  Retorna a OS atualizada com os serviços.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS ou serviço do catálogo não encontrado.
*   `409 Conflict`: OS em status finalizado.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Serviços adicionados à OS, total da OS atualizado.

**Critérios de Aceite:**
*   Serviços são adicionados com sucesso.
*   Total da OS é recalculado.
*   Auditoria é registrada.

#### 2.8.5 Adicionar Produtos à OS

**Objetivo:** Adicionar um ou mais produtos/peças a uma Ordem de Serviço.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID), `products` (array de objetos com `productId` (opcional), `description`, `quantity`, `unitPrice`, `discount`).

**Regras de Negócio:**
*   A OS não pode estar em status `completed`, `delivered` ou `canceled`.
*   `productId` (se fornecido) deve ser um produto existente no catálogo do tenant.
*   `quantity` e `unitPrice` devem ser positivos.
*   A `quantity` não pode exceder o `stockQuantity` disponível (se `productId` for do catálogo).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.
*   `products`: Array não vazio.
*   Cada item de produto: `description` obrigatório, `quantity` e `unitPrice` numéricos e positivos.

**Permissões:** `owner`, `admin`, `employee` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`, `products`.
2.  Verifica permissões e status da OS.
3.  Para cada produto:
    *   Valida os dados.
    *   Chama `WorkOrderRepository.addWorkOrderProduct(tenantId, workOrderId, productData)`.
    *   Atualiza o `stockQuantity` do produto no catálogo (se for do catálogo).
4.  Atualiza o valor total da OS.
5.  Registra auditoria.
6.  Retorna a OS atualizada com os produtos.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS ou produto do catálogo não encontrado.
*   `409 Conflict`: OS em status finalizado, estoque insuficiente.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Produtos adicionados à OS, total da OS atualizado, estoque atualizado.

**Critérios de Aceite:**
*   Produtos são adicionados com sucesso.
*   Total da OS é recalculado.
*   Estoque é atualizado corretamente.
*   Auditoria é registrada.

#### 2.8.6 Finalizar OS

**Objetivo:** Marcar uma Ordem de Serviço como concluída e gerar contas a receber.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID).

**Regras de Negócio:**
*   A OS deve estar no status `in_progress` ou `approved`.
*   Apenas `owner` ou `admin` podem finalizar.
*   Gera um registro em `accounts_receivable` para o valor total da OS.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`.
2.  Verifica permissões e status da OS.
3.  Chama `WorkOrderService.finalizeWorkOrder(tenantId, workOrderId)` (que altera o status para `completed`).
4.  Cria um registro em `accounts_receivable` com o valor total da OS.
5.  Registra histórico e auditoria.
6.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS não encontrada.
*   `409 Conflict`: OS não está em status válido para finalização.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** OS com status `completed`, conta a receber gerada.

**Critérios de Aceite:**
*   OS é finalizada com sucesso.
*   Conta a receber é gerada corretamente.
*   Auditoria é registrada.

#### 2.8.7 Reabrir OS

**Objetivo:** Mudar o status de uma OS `completed` ou `delivered` de volta para `in_progress`.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID).

**Regras de Negócio:**
*   A OS deve estar no status `completed` ou `delivered`.
*   Apenas `owner` ou `admin` podem reabrir.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`.
2.  Verifica permissões e status da OS.
3.  Chama `WorkOrderService.reopenWorkOrder(tenantId, workOrderId)` (que altera o status para `in_progress`).
4.  Registra histórico e auditoria.
5.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS não encontrada.
*   `409 Conflict`: OS não está em status válido para reabertura.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** OS com status `in_progress`.

**Critérios de Aceite:**
*   OS é reaberta com sucesso.
*   Auditoria é registrada.

#### 2.8.8 Cancelar OS

**Objetivo:** Marcar uma Ordem de Serviço como cancelada.

**Entrada:** `tenantId` (UUID), `workOrderId` (UUID).

**Regras de Negócio:**
*   A OS não pode estar em status `completed` ou `delivered`.
*   Apenas `owner` ou `admin` podem cancelar.
*   Se houver produtos alocados, o estoque deve ser revertido.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `workOrderId`: Obrigatório, UUID válido.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `workOrderId`.
2.  Verifica permissões e status da OS.
3.  Chama `WorkOrderService.cancelWorkOrder(tenantId, workOrderId)` (que altera o status para `canceled`).
4.  Reverte o estoque de produtos alocados.
5.  Registra histórico e auditoria.
6.  Retorna a OS atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: OS não encontrada.
*   `409 Conflict`: OS em status finalizado.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** OS com status `canceled`, estoque revertido.

**Critérios de Aceite:**
*   OS é cancelada com sucesso.
*   Estoque de produtos é revertido.
*   Auditoria é registrada.

### 2.9 Financeiro

#### 2.9.1 Criar Receita (Manual)

**Objetivo:** Registrar uma nova receita manual no sistema.

**Entrada:** `tenantId` (UUID), `data` (objeto com `description`, `amount`, `transactionDate`, `paymentMethod`, `customerId` (opcional)).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem criar receitas.
*   `amount` deve ser positivo.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `description`: Obrigatório.
*   `amount`: Numérico, positivo.
*   `transactionDate`: Data válida.
*   `paymentMethod`: Valor válido do enum `payment_method`.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `FinancialRepository.createFinancialTransaction(tenantId, data, type=\'income\')`.
5.  Registra auditoria.
6.  Retorna a transação criada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Nova transação de receita registrada.

**Critérios de Aceite:**
*   Receita manual é criada com sucesso.
*   Auditoria é registrada.

#### 2.9.2 Criar Despesa (Manual)

**Objetivo:** Registrar uma nova despesa manual no sistema.

**Entrada:** `tenantId` (UUID), `data` (objeto com `description`, `amount`, `transactionDate`, `paymentMethod`, `supplier` (opcional), `category` (opcional)).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem criar despesas.
*   `amount` deve ser positivo.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `description`: Obrigatório.
*   `amount`: Numérico, positivo.
*   `transactionDate`: Data válida.
*   `paymentMethod`: Valor válido do enum `payment_method`.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `data`.
2.  Verifica permissões.
3.  Valida `data`.
4.  Chama `FinancialRepository.createFinancialTransaction(tenantId, data, type=\'expense\')`.
5.  Registra auditoria.
6.  Retorna a transação criada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `422 Unprocessable Entity`: Dados inválidos.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Nova transação de despesa registrada.

**Critérios de Aceite:**
*   Despesa manual é criada com sucesso.
*   Auditoria é registrada.

#### 2.9.3 Contas a Receber

**Objetivo:** Gerenciar e listar todas as contas a receber do tenant.

**Entrada:** `tenantId` (UUID), `filters` (objeto com `status`, `dueDateRange`, `customerId`, `page`, `pageSize`).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem acessar.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `page`, `pageSize`: Numéricos, positivos.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `filters`.
2.  Verifica permissões.
3.  Chama `FinancialRepository.findAccountsReceivable(tenantId, filters)`.
4.  Retorna lista de contas a receber e total para paginação.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista paginada de contas a receber.

**Critérios de Aceite:**
*   Listagem funciona com filtros e paginação.
*   Permissões são respeitadas.

#### 2.9.4 Contas a Pagar

**Objetivo:** Gerenciar e listar todas as contas a pagar do tenant.

**Entrada:** `tenantId` (UUID), `filters` (objeto com `status`, `dueDateRange`, `supplier`, `page`, `pageSize`).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem acessar.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `page`, `pageSize`: Numéricos, positivos.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `filters`.
2.  Verifica permissões.
3.  Chama `FinancialRepository.findAccountsPayable(tenantId, filters)`.
4.  Retorna lista de contas a pagar e total para paginação.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista paginada de contas a pagar.

**Critérios de Aceite:**
*   Listagem funciona com filtros e paginação.
*   Permissões são respeitadas.

#### 2.9.5 Fluxo de Caixa

**Objetivo:** Visualizar o histórico de transações financeiras (receitas e despesas).

**Entrada:** `tenantId` (UUID), `filters` (objeto com `dateRange`, `type`, `paymentMethod`, `page`, `pageSize`).

**Regras de Negócio:**
*   Apenas `owner` ou `admin` podem acessar.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `page`, `pageSize`: Numéricos, positivos.

**Permissões:** `owner`, `admin` do tenant.

**Fluxo:**
1.  Recebe `tenantId` e `filters`.
2.  Verifica permissões.
3.  Chama `FinancialRepository.findFinancialTransactions(tenantId, filters)`.
4.  Retorna lista de transações e total para paginação.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Lista paginada de transações financeiras.

**Critérios de Aceite:**
*   Listagem funciona com filtros e paginação.
*   Permissões são respeitadas.

### 2.10 Assinaturas

#### 2.10.1 Iniciar Trial

**Objetivo:** Ativar o período de teste gratuito para um novo tenant.

**Entrada:** `tenantId` (UUID).

**Regras de Negócio:**
*   Apenas tenants recém-criados ou sem assinatura ativa podem iniciar trial.
*   O trial tem uma duração fixa (ex: 14 dias).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.

**Permissões:** Interno (invocado pelo caso de uso de cadastro).

**Fluxo:**
1.  Recebe `tenantId`.
2.  Verifica se o tenant pode iniciar trial.
3.  Chama `SubscriptionRepository.createTrialSubscription(tenantId)`.
4.  Atualiza o status do tenant para `trialing`.
5.  Registra auditoria.
6.  Retorna a assinatura de trial.

**Erros Possíveis:**
*   `409 Conflict`: Tenant já possui assinatura ou trial.
*   `500 Internal Server Error`: Erro inesperado.

**Resultado Esperado:** Assinatura de trial criada.

**Critérios de Aceite:**
*   Trial é iniciado com sucesso para novos tenants.
*   Auditoria é registrada.

#### 2.10.2 Upgrade de Plano

**Objetivo:** Mudar a assinatura do tenant para um plano superior.

**Entrada:** `tenantId` (UUID), `newPlanId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` do tenant pode fazer upgrade.
*   O `newPlanId` deve ser um plano ativo e de valor superior ao atual.
*   Integração com Stripe para gerenciar a mudança de plano e prorrogação.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `newPlanId`: Obrigatório, UUID válido, plano ativo.

**Permissões:** `owner` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `newPlanId`.
2.  Verifica permissões e validade do novo plano.
3.  Chama `StripeService.updateSubscriptionPlan(stripeSubscriptionId, newStripePriceId)`.
4.  Em caso de sucesso na Stripe, atualiza `subscriptions` no banco de dados.
5.  Registra auditoria.
6.  Retorna a assinatura atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Plano ou assinatura não encontrada.
*   `422 Unprocessable Entity`: Plano inválido.
*   `500 Internal Server Error`: Erro na integração com Stripe.

**Resultado Esperado:** Assinatura atualizada para o novo plano.

**Critérios de Aceite:**
*   Upgrade de plano é processado com sucesso na Stripe e no sistema.
*   Auditoria é registrada.

#### 2.10.3 Downgrade de Plano

**Objetivo:** Mudar a assinatura do tenant para um plano inferior.

**Entrada:** `tenantId` (UUID), `newPlanId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` do tenant pode fazer downgrade.
*   O `newPlanId` deve ser um plano ativo e de valor inferior ao atual.
*   Integração com Stripe para gerenciar a mudança de plano (geralmente com prorrogação ou crédito).

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.
*   `newPlanId`: Obrigatório, UUID válido, plano ativo.

**Permissões:** `owner` do tenant.

**Fluxo:**
1.  Recebe `tenantId`, `newPlanId`.
2.  Verifica permissões e validade do novo plano.
3.  Chama `StripeService.updateSubscriptionPlan(stripeSubscriptionId, newStripePriceId)`.
4.  Em caso de sucesso na Stripe, atualiza `subscriptions` no banco de dados.
5.  Registra auditoria.
6.  Retorna a assinatura atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Plano ou assinatura não encontrada.
*   `422 Unprocessable Entity`: Plano inválido.
*   `500 Internal Server Error`: Erro na integração com Stripe.

**Resultado Esperado:** Assinatura atualizada para o novo plano.

**Critérios de Aceite:**
*   Downgrade de plano é processado com sucesso na Stripe e no sistema.
*   Auditoria é registrada.

#### 2.10.4 Cancelamento de Assinatura

**Objetivo:** Cancelar a assinatura ativa de um tenant.

**Entrada:** `tenantId` (UUID).

**Regras de Negócio:**
*   Apenas `owner` do tenant pode cancelar.
*   A assinatura é cancelada na Stripe e marcada como `canceled` no sistema.
*   O acesso ao sistema é mantido até o final do período de faturamento atual.

**Validações:**
*   `tenantId`: Obrigatório, UUID válido.

**Permissões:** `owner` do tenant.

**Fluxo:**
1.  Recebe `tenantId`.
2.  Verifica permissões.
3.  Chama `StripeService.cancelSubscription(stripeSubscriptionId)`.
4.  Em caso de sucesso na Stripe, atualiza `subscriptions` no banco de dados para `canceled` e define `cancel_at_period_end`.
5.  Registra auditoria.
6.  Retorna a assinatura atualizada.

**Erros Possíveis:**
*   `403 Forbidden`: Usuário sem permissão.
*   `404 Not Found`: Assinatura não encontrada.
*   `500 Internal Server Error`: Erro na integração com Stripe.

**Resultado Esperado:** Assinatura cancelada, acesso mantido até o fim do período.

**Critérios de Aceite:**
*   Assinatura é cancelada com sucesso na Stripe e no sistema.
*   Acesso é mantido até o final do período de faturamento.
*   Auditoria é registrada.

## 3. Serviços

Esta seção detalha os serviços que encapsulam a lógica de negócio e as interações com o banco de dados e serviços externos. Cada serviço será implementado como uma classe ou módulo.

### 3.1 `AuthService`

**Responsabilidades:** Gerenciar a autenticação de usuários via Supabase Auth.

**Métodos:**
*   `signInWithPassword(email, password)`: Autentica o usuário.
*   `signUp(email, password)`: Registra um novo usuário.
*   `sendPasswordResetEmail(email)`: Envia e-mail de recuperação de senha.
*   `updateUserPassword(newPassword)`: Atualiza a senha do usuário.
*   `signOut()`: Desloga o usuário.
*   `inviteUserByEmail(email)`: Envia convite para novo usuário.

**Dependências:** Supabase Auth client.

**Regras:** Interage diretamente com o Supabase Auth para todas as operações de autenticação.

### 3.2 `TenantService`

**Responsabilidades:** Gerenciar a criação, atualização e recuperação de informações dos tenants e seus usuários.

**Métodos:**
*   `createTenant(companyName, ownerUserId)`: Cria um novo tenant e associa o owner.
*   `updateTenant(tenantId, data)`: Atualiza informações do tenant.
*   `getTenantById(tenantId)`: Recupera informações de um tenant.
*   `addTenantUser(tenantId, userId, role)`: Adiciona um usuário a um tenant.
*   `updateTenantUserRole(tenantId, userId, newRole)`: Atualiza a role de um usuário no tenant.
*   `removeTenantUser(tenantId, userId)`: Remove um usuário do tenant.

**Dependências:** `TenantRepository`, `UserRepository`.

**Regras:** Aplica regras de negócio relacionadas a tenants e usuários (ex: unicidade de nome, limites de usuários, transições de role).

### 3.3 `CustomerService`

**Responsabilidades:** Gerenciar o CRUD de clientes.

**Métodos:**
*   `createCustomer(tenantId, data)`: Cria um novo cliente.
*   `updateCustomer(tenantId, customerId, data)`: Atualiza um cliente.
*   `softDeleteCustomer(tenantId, customerId)`: Arquiva um cliente.
*   `findCustomers(tenantId, filters)`: Busca e filtra clientes.
*   `getCustomerById(tenantId, customerId)`: Recupera um cliente específico.

**Dependências:** `CustomerRepository`.

**Regras:** Validações de dados do cliente, unicidade de documento, verifica OS abertas antes de arquivar.

### 3.4 `VehicleService`

**Responsabilidades:** Gerenciar o CRUD de veículos e seu histórico.

**Métodos:**
*   `createVehicle(tenantId, data)`: Cria um novo veículo.
*   `updateVehicle(tenantId, vehicleId, data)`: Atualiza um veículo.
*   `softDeleteVehicle(tenantId, vehicleId)`: Arquiva um veículo.
*   `findVehicles(tenantId, filters)`: Busca e filtra veículos.
*   `getVehicleById(tenantId, vehicleId)`: Recupera um veículo específico.
*   `getVehicleWorkOrderHistory(tenantId, vehicleId)`: Recupera histórico de OS do veículo.

**Dependências:** `VehicleRepository`, `WorkOrderRepository`.

**Regras:** Validações de dados do veículo, unicidade de placa, verifica OS abertas antes de arquivar.

### 3.5 `ServiceCatalogService`

**Responsabilidades:** Gerenciar o CRUD do catálogo de serviços da oficina.

**Métodos:**
*   `createService(tenantId, data)`: Cria um novo serviço.
*   `updateService(tenantId, serviceId, data)`: Atualiza um serviço.
*   `softDeleteService(tenantId, serviceId)`: Arquiva um serviço.
*   `findServices(tenantId, filters)`: Busca e filtra serviços.
*   `getServiceById(tenantId, serviceId)`: Recupera um serviço específico.

**Dependências:** `ServiceCatalogRepository`.

**Regras:** Validações de dados do serviço, unicidade de nome, verifica uso em OS abertas antes de arquivar.

### 3.6 `ProductCatalogService`

**Responsabilidades:** Gerenciar o CRUD do catálogo de produtos/peças da oficina.

**Métodos:**
*   `createProduct(tenantId, data)`: Cria um novo produto.
*   `updateProduct(tenantId, productId, data)`: Atualiza um produto.
*   `softDeleteProduct(tenantId, productId)`: Arquiva um produto.
*   `findProducts(tenantId, filters)`: Busca e filtra produtos.
*   `getProductById(tenantId, productId)`: Recupera um produto específico.

**Dependências:** `ProductCatalogRepository`.

**Regras:** Validações de dados do produto, unicidade de nome/SKU, verifica uso em OS abertas antes de arquivar, gerencia estoque.

### 3.7 `WorkOrderService`

**Responsabilidades:** Gerenciar o ciclo de vida completo das Ordens de Serviço.

**Métodos:**
*   `createWorkOrder(tenantId, data)`: Abre uma nova OS.
*   `updateWorkOrder(tenantId, workOrderId, data)`: Atualiza dados gerais da OS.
*   `updateWorkOrderStatus(tenantId, workOrderId, newStatus)`: Altera o status da OS.
*   `approveWorkOrder(tenantId, workOrderId)`: Aprova uma OS.
*   `addWorkOrderService(tenantId, workOrderId, serviceData)`: Adiciona serviço à OS.
*   `addWorkOrderProduct(tenantId, workOrderId, productData)`: Adiciona produto à OS.
*   `finalizeWorkOrder(tenantId, workOrderId)`: Finaliza a OS e gera conta a receber.
*   `reopenWorkOrder(tenantId, workOrderId)`: Reabre uma OS.
*   `cancelWorkOrder(tenantId, workOrderId)`: Cancela uma OS.
*   `getWorkOrderById(tenantId, workOrderId)`: Recupera uma OS completa.
*   `findWorkOrders(tenantId, filters)`: Busca e filtra OS.

**Dependências:** `WorkOrderRepository`, `CustomerRepository`, `VehicleRepository`, `ServiceCatalogRepository`, `ProductCatalogRepository`, `FinancialService`.

**Regras:** Validações de fluxo de status, estoque de produtos, cálculos de totais da OS, geração de contas a receber.

### 3.8 `FinancialService`

**Responsabilidades:** Gerenciar transações financeiras, contas a receber e a pagar.

**Métodos:**
*   `createIncome(tenantId, data)`: Registra uma receita manual.
*   `createExpense(tenantId, data)`: Registra uma despesa manual.
*   `registerPayment(tenantId, receivableId, amount, paymentMethod)`: Registra pagamento de conta a receber.
*   `findAccountsReceivable(tenantId, filters)`: Busca contas a receber.
*   `findAccountsPayable(tenantId, filters)`: Busca contas a pagar.
*   `findFinancialTransactions(tenantId, filters)`: Busca transações de fluxo de caixa.

**Dependências:** `FinancialRepository`, `AccountsReceivableRepository`, `AccountsPayableRepository`.

**Regras:** Validações de valores, datas, métodos de pagamento. Atualização de status de contas.

### 3.9 `SubscriptionService`

**Responsabilidades:** Gerenciar o ciclo de vida das assinaturas do tenant, incluindo trial, upgrade, downgrade e cancelamento.

**Métodos:**
*   `startTrial(tenantId)`: Inicia o período de trial.
*   `upgradeSubscription(tenantId, newPlanId)`: Faz upgrade de plano.
*   `downgradeSubscription(tenantId, newPlanId)`: Faz downgrade de plano.
*   `cancelSubscription(tenantId)`: Cancela a assinatura.
*   `getSubscriptionDetails(tenantId)`: Recupera detalhes da assinatura atual.
*   `handleStripeWebhook(event)`: Processa eventos de webhook da Stripe.

**Dependências:** `SubscriptionRepository`, `StripeService`, `TenantRepository`.

**Regras:** Validações de planos, limites de usuários, sincronização com Stripe, regras de prorrogação/crédito.

### 3.10 `NotificationService`

**Responsabilidades:** Gerenciar a criação e entrega de notificações internas e por e-mail.

**Métodos:**
*   `createNotification(tenantId, userId, type, title, message)`: Cria uma notificação interna.
*   `sendEmailNotification(to, subject, body)`: Envia notificação por e-mail.
*   `markNotificationAsRead(tenantId, notificationId)`: Marca notificação como lida.

**Dependências:** `NotificationRepository`, serviço de e-mail (ex: Resend, Nodemailer).

**Regras:** Define tipos de notificação, prioridades, e regras de entrega.

### 3.11 `AuditService`

**Responsabilidades:** Registrar e gerenciar logs de auditoria.

**Métodos:**
*   `createAuditLog(tenantId, userId, action, entityType, entityId, oldData, newData, ipAddress, userAgent)`: Cria um registro de auditoria.
*   `findAuditLogs(tenantId, filters)`: Busca e filtra logs de auditoria.

**Dependências:** `AuditLogRepository`.

**Regras:** Define quais ações são auditáveis e o formato dos logs.

### 3.12 `SettingsService`

**Responsabilidades:** Gerenciar as configurações do tenant.

**Métodos:**
*   `getSetting(tenantId, key)`: Recupera uma configuração.
*   `updateSetting(tenantId, key, value)`: Atualiza uma configuração.

**Dependências:** `SettingsRepository`.

**Regras:** Validações de configurações, permissões de acesso.

## 4. Integração com Supabase

### 4.1 Queries

Todas as interações com o banco de dados PostgreSQL serão realizadas através do cliente Supabase, utilizando o construtor de queries fluente ou funções SQL diretas (RPCs) para operações mais complexas.

*   **Cliente Supabase:** `createClient` do `@supabase/supabase-js` será usado para interagir com o banco de dados.
*   **Tipagem:** Utilização de tipos gerados automaticamente pelo Supabase CLI (`supabase gen types typescript --schema public > types/supabase.ts`) para garantir segurança de tipo nas queries.
*   **`tenant_id`:** Todas as queries em tabelas multi-tenant incluirão uma cláusula `WHERE tenant_id = current_tenant_id` (implicitamente via RLS ou explicitamente em casos específicos).
*   **Transações:** Operações que envolvem múltiplas escritas ou atualizações serão encapsuladas em transações para garantir atomicidade.

### 4.2 Policies RLS

As políticas de Row Level Security (RLS) são configuradas diretamente no PostgreSQL via Supabase para impor o isolamento de dados por tenant e o controle de acesso baseado em role.

*   **Habilitação:** RLS será habilitado para todas as tabelas multi-tenant.
*   **Funções de Contexto:** Funções como `auth.uid()` e `auth.jwt()` serão usadas nas políticas para obter o `user_id` do usuário autenticado e, a partir dele, o `tenant_id` associado (via `tenant_users`).
*   **Políticas por Operação:** Políticas `SELECT`, `INSERT`, `UPDATE`, `DELETE` serão definidas para cada tabela, garantindo que o usuário só possa operar em dados do seu `tenant_id` e de acordo com sua `role`.
*   **Exemplo de Política (SELECT para `customers`):**
    ```sql
    CREATE POLICY "Tenants can view their own customers" ON customers
    FOR SELECT USING (tenant_id = (SELECT tenant_id FROM tenant_users WHERE user_id = auth.uid()));
    ```
*   **Exemplo de Política (INSERT para `customers`):**
    ```sql
    CREATE POLICY "Tenants can create customers" ON customers
    FOR INSERT WITH CHECK (tenant_id = (SELECT tenant_id FROM tenant_users WHERE user_id = auth.uid()));
    ```

### 4.3 Uploads (Supabase Storage)

O Supabase Storage será utilizado para armazenar arquivos como logos de oficinas, fotos de veículos e anexos de ordens de serviço.

*   **Buckets:** Serão criados buckets específicos (ex: `logos`, `vehicle-photos`, `work-order-attachments`).
*   **Políticas de Acesso:** Políticas de segurança serão configuradas para cada bucket para controlar quem pode fazer upload, download e visualizar arquivos, geralmente baseadas no `tenant_id` e `user_id`.
*   **Geração de URLs:** URLs públicas ou assinadas serão geradas para acesso aos arquivos, conforme a necessidade.

### 4.4 Transactions

Para garantir a integridade dos dados em operações que envolvem múltiplas modificações, as transações serão utilizadas.

*   **Uso:** Em casos de uso como `finalizeWorkOrder` (que atualiza o status da OS e cria uma conta a receber) ou `addWorkOrderProduct` (que adiciona um produto à OS e atualiza o estoque).
*   **Implementação:** O cliente Supabase permite o uso de transações de banco de dados para agrupar operações.

### 4.5 Soft Delete

Em vez de excluir permanentemente registros, o sistema utilizará o conceito de soft delete para a maioria das entidades operacionais (clientes, veículos, serviços, produtos, usuários de tenant).

*   **Coluna `deleted_at`:** Adição de uma coluna `deleted_at` (timestamp com timezone, nullable) em tabelas relevantes.
*   **Lógica de Negócio:** Ao invés de `DELETE`, as operações de "exclusão" irão atualizar a coluna `deleted_at` com o timestamp atual e, opcionalmente, mudar o `status` para `inactive`.
*   **Queries:** Todas as queries de leitura incluirão `WHERE deleted_at IS NULL` para filtrar registros ativos por padrão.
*   **RLS:** As políticas de RLS também considerarão a coluna `deleted_at`.

## 5. Integração com Stripe

A integração com a Stripe é crucial para o gerenciamento de assinaturas e pagamentos.

### 5.1 Checkout

Para a criação de novas assinaturas ou upgrades, o sistema utilizará o Stripe Checkout.

*   **Sessões de Checkout:** O backend criará sessões de checkout na Stripe, especificando o plano (price ID), o cliente (customer ID) e URLs de sucesso/cancelamento.
*   **Redirecionamento:** O frontend redirecionará o usuário para a URL da sessão de checkout da Stripe.

### 5.2 Customer Portal

O Stripe Customer Portal será utilizado para permitir que os usuários gerenciem suas informações de faturamento, métodos de pagamento e histórico de faturas diretamente na Stripe.

*   **Geração de Link:** O backend gerará um link para o Customer Portal para o `stripe_customer_id` do tenant.
*   **Redirecionamento:** O frontend redirecionará o usuário para este link.

### 5.3 Trial

O período de trial é gerenciado tanto no sistema quanto na Stripe.

*   **Criação de Assinatura com Trial:** Ao iniciar o trial, uma assinatura é criada na Stripe com `trial_period_days`.
*   **Sincronização:** Webhooks da Stripe (ex: `customer.subscription.trial_will_end`, `customer.subscription.updated`) são usados para sincronizar o status do trial no sistema.

### 5.4 Assinaturas (Upgrade/Downgrade)

*   **API da Stripe:** O `StripeService` utilizará a API da Stripe para atualizar assinaturas existentes, alterando o `price_id` do plano.
*   **Prorrogação:** A Stripe gerencia automaticamente a prorrogação e os créditos/débitos resultantes de upgrades/downgrades.
*   **Webhooks:** Eventos `customer.subscription.updated` são essenciais para manter o sistema sincronizado com as mudanças de plano.

### 5.5 Cancelamentos

*   **API da Stripe:** O `StripeService` chamará a API da Stripe para cancelar assinaturas.
*   **`cancel_at_period_end`:** Assinaturas são geralmente canceladas no final do período de faturamento atual para evitar cobranças inesperadas.
*   **Webhooks:** Eventos `customer.subscription.deleted` ou `customer.subscription.updated` (com `status=\'canceled\'`) são usados para refletir o cancelamento no sistema.

### 5.6 Webhooks

Os webhooks da Stripe são fundamentais para manter o estado do sistema sincronizado com as transações e eventos de assinatura na Stripe.

*   **Endpoint:** Um Route Handler (`/api/stripe/webhook`) será configurado para receber os eventos da Stripe.
*   **Segurança:** O endpoint verificará a assinatura do webhook para garantir que a requisição é legítima da Stripe.
*   **Idempotência:** O processamento de webhooks será idempotente para lidar com entregas duplicadas. Cada evento da Stripe tem um ID único que pode ser usado para evitar o reprocessamento.
*   **Eventos Chave:**
    *   `checkout.session.completed`: Acionado após um checkout bem-sucedido. Usado para criar ou atualizar a assinatura no sistema e associar o `stripe_customer_id` e `stripe_subscription_id` ao tenant.
    *   `customer.subscription.created`: Confirma a criação de uma nova assinatura. Usado para atualizar o status da assinatura no sistema.
    *   `customer.subscription.updated`: Acionado em qualquer mudança na assinatura (status, plano, trial). Usado para sincronizar o `subscription_status`, `current_period_end`, `stripe_price_id` e outros detalhes.
    *   `customer.subscription.deleted`: Acionado quando uma assinatura é cancelada ou expira. Usado para marcar a assinatura como `canceled` e, potencialmente, desativar o tenant se não houver outra assinatura ativa.
    *   `invoice.payment_failed`: Acionado quando um pagamento de fatura falha. Usado para notificar o usuário, atualizar o `subscription_status` para `past_due` e, se necessário, iniciar o processo de dunning.

*   **Sincronização de Status:** O `SubscriptionService` processará esses webhooks para atualizar o status da assinatura do tenant na tabela `subscriptions` e, consequentemente, o `status` do tenant na tabela `tenants`.

## 6. Auditoria

O sistema de auditoria registra todas as ações significativas para garantir rastreabilidade, conformidade e segurança.

### 6.1 Registrar

Cada registro de auditoria na tabela `audit_logs` conterá:

*   **`user_id`:** Quem executou a ação.
*   **`tenant_id`:** A qual tenant a ação pertence.
*   **`action`:** Tipo de ação (CREATE, UPDATE, DELETE, LOGIN, LOGOUT, etc.).
*   **`entity_type`:** Tabela ou entidade afetada (ex: `customers`, `work_orders`, `settings`).
*   **`entity_id`:** ID do registro afetado.
*   **`old_data` (JSONB):** Estado do registro antes da alteração (para UPDATE/DELETE).
*   **`new_data` (JSONB):** Estado do registro após a alteração (para CREATE/UPDATE).
*   **`ip_address`:** Endereço IP de origem da requisição.
*   **`user_agent`:** User agent do cliente.
*   **`created_at`:** Quando a ação foi executada.

### 6.2 Implementação

*   **Triggers de Banco de Dados:** Para operações de CRUD de baixo nível, triggers `AFTER INSERT`, `AFTER UPDATE`, `AFTER DELETE` podem ser usados para popular a tabela `audit_logs` automaticamente.
*   **Lógica de Serviço/Caso de Uso:** Para ações mais complexas ou que envolvem múltiplos passos, a lógica de registro de auditoria será incorporada nos `use-cases` ou `services` (ex: `AuditService.createAuditLog()`).

## 7. Logs

Uma estratégia de logging eficaz é vital para a saúde e monitoramento do sistema.

### 7.1 Estratégia

*   **Níveis de Log:** Utilização de níveis de log padrão (DEBUG, INFO, WARN, ERROR, FATAL).
*   **Formato:** Logs em formato JSON para facilitar a ingestão e análise por ferramentas de agregação.
*   **Contexto:** Cada log incluirá contexto relevante como `tenant_id`, `user_id`, `requestId`, `module`, `function`, `errorStack`.
*   **Ferramentas:** Biblioteca de logging (ex: Pino) integrada com um serviço de monitoramento (ex: Vercel Log Drains, Sentry).

### 7.2 Tipos de Logs

*   **Logs de Erro:** Capturam todas as exceções não tratadas e erros esperados, com stack traces completos.
*   **Logs de Sistema:** Registram eventos operacionais importantes (ex: inicialização de serviços, chamadas de API externas, processamento de webhooks).
*   **Logs Financeiros:** Detalhes de transações financeiras, status de pagamento, eventos de faturamento da Stripe.

## 8. Uploads

O gerenciamento de uploads de arquivos será feito via Supabase Storage.

### 8.1 Documentação

*   **Logo da Oficina:** Armazenada em um bucket `logos`. A URL será salva na tabela `tenants`.
*   **Fotos dos Veículos:** Armazenadas em um bucket `vehicle-photos`. URLs salvas na tabela `vehicles`.
*   **Anexos da OS:** Documentos, imagens relacionadas a uma OS. Armazenados em um bucket `work-order-attachments`. URLs salvas na tabela `work_order_attachments` (nova tabela).

### 8.2 Processo

1.  Frontend envia o arquivo para um Server Action.
2.  Server Action utiliza o cliente Supabase Storage para fazer o upload do arquivo para o bucket apropriado.
3.  A URL pública (ou assinada, se necessário) do arquivo é salva no banco de dados.
4.  Políticas de segurança do Storage garantem que apenas usuários autorizados (do tenant correto) possam acessar/gerenciar seus arquivos.

## 9. Notificações

O sistema de notificações informará os usuários sobre eventos importantes.

### 9.1 Tipos

*   **Notificações Internas:** Exibidas dentro da interface do usuário (ex: no sino de notificações, na página `/notifications`).
*   **E-mail:** Enviadas para o endereço de e-mail do usuário (ex: confirmação de cadastro, alerta de pagamento falho, resumo semanal).
*   **Futuro WhatsApp:** Considerado para futuras integrações (ex: lembretes de OS, status de veículo).

### 9.2 Implementação

*   **`NotificationService`:** Orquestra a criação e envio de notificações.
*   **Templates:** Utilização de templates para e-mails e mensagens para garantir consistência.
*   **Fila de Mensagens:** Para notificações de alto volume ou assíncronas, pode-se considerar uma fila de mensagens (ex: Redis, Supabase Edge Functions com Deno Deploy).

## 10. Segurança

A segurança é um pilar fundamental do OficinaPro, abordada em múltiplas camadas.

### 10.1 Controle de Acesso

*   **Autenticação:** Gerenciada pelo Supabase Auth (JWT).
*   **Autorização:** Baseada em roles (`tenant_role` enum) e permissões granulares.
*   **RLS:** Imposto no nível do banco de dados para isolamento de dados por tenant.

### 10.2 Permissões por Perfil

*   **`owner`:** Acesso total a todas as funcionalidades e configurações do tenant.
*   **`admin`:** Acesso quase total, mas com restrições em operações críticas (ex: exclusão de owner, cancelamento de assinatura).
*   **`employee`:** Acesso restrito a funcionalidades operacionais (CRUD de clientes, veículos, OS), sem acesso a configurações financeiras ou de tenant.
*   **Implementação:** Verificações de permissão na camada de `use-cases` e nas políticas de RLS.

### 10.3 Proteção Contra Acesso Entre Tenants

*   **RLS:** Principal mecanismo de proteção.
*   **Validação de `tenant_id`:** Todas as queries e operações de escrita/leitura no backend devem incluir o `tenant_id` do usuário autenticado.
*   **Testes:** Testes de integração e segurança para garantir que nenhum dado de um tenant possa ser acessado por outro.

### 10.4 Rate Limiting

*   **Objetivo:** Proteger o backend contra ataques de força bruta e uso excessivo de recursos.
*   **Implementação:** Pode ser configurado no nível do Vercel, ou implementado em Route Handlers específicos para rotas sensíveis (ex: login, registro).

### 10.5 Sanitização e Validação de Entrada

*   **Validação:** Todos os dados de entrada são validados usando Zod para garantir o formato e tipo corretos.
*   **Sanitização:** Dados de entrada (especialmente strings) são sanitizados para prevenir ataques como XSS e SQL Injection (o uso de prepared statements pelo Supabase client já ajuda contra SQL Injection).

### 10.6 Proteção Contra Duplicidade

*   **Constraints de Banco de Dados:** `UNIQUE` constraints em colunas como `email` (para usuários), `document` (para clientes), `plate` (para veículos), `name`/`sku` (para produtos/serviços) combinadas com `tenant_id`.
*   **Lógica de Negócio:** Verificações adicionais na camada de `use-cases` antes de criar novos registros.

## 11. Performance

A performance é otimizada em várias frentes para garantir uma experiência de usuário rápida e responsiva.

### 11.1 Paginação

*   **Queries Paginadas:** Todas as listagens de dados (clientes, veículos, OS, produtos, serviços, logs) utilizarão paginação (`LIMIT` e `OFFSET` ou `cursor-based pagination`).
*   **Contagem Total:** Queries separadas para obter a contagem total de registros para a paginação.

### 11.2 Cache

*   **Cache de Dados:** Dados frequentemente acessados e que não mudam com frequência (ex: configurações de tenant, planos de assinatura) podem ser cacheados em memória ou em um serviço de cache (ex: Redis).
*   **Cache de Queries:** O Supabase e o PostgreSQL possuem mecanismos de cache internos que podem ser aproveitados.

### 11.3 Índices

*   **Índices de Banco de Dados:** Criação de índices apropriados em colunas frequentemente usadas em cláusulas `WHERE`, `ORDER BY` e `JOIN`.
*   **Índices Compostos:** Para tabelas multi-tenant, índices compostos começando com `tenant_id` são cruciais para otimizar queries.

### 11.4 Lazy Loading

*   **Relacionamentos:** Carregamento de dados relacionados (ex: serviços e produtos de uma OS) apenas quando necessário, para evitar o carregamento excessivo de dados.
*   **Componentes Frontend:** Utilização de lazy loading no frontend para componentes e rotas.

## 12. Estratégia de Erros

Uma estratégia de tratamento de erros consistente é fundamental para a robustez do backend e para fornecer feedback útil ao frontend.

### 12.1 Padrões de Erro

*   **Erros Customizados:** Definição de classes de erro customizadas para erros de negócio específicos (ex: `TenantNotFound`, `PermissionDenied`, `InvalidStatusTransition`).
*   **Códigos HTTP:** Mapeamento de erros para códigos de status HTTP apropriados.

### 12.2 Códigos de Status HTTP

*   **`400 Bad Request`:** Requisição malformada, validação de entrada falhou.
*   **`401 Unauthorized`:** Falha na autenticação (credenciais inválidas, token ausente/expirado).
*   **`403 Forbidden`:** Usuário autenticado, mas sem permissão para executar a ação (autorização falhou).
*   **`404 Not Found`:** Recurso solicitado não encontrado.
*   **`409 Conflict`:** Conflito de recursos (ex: e-mail já registrado, placa duplicada).
*   **`422 Unprocessable Entity`:** Validação de regras de negócio falhou (ex: transição de status inválida).
*   **`500 Internal Server Error`:** Erro inesperado no servidor.

### 12.3 Tratamento de Erros Global

*   Um middleware ou interceptor global para capturar e formatar erros, garantindo que o frontend receba respostas de erro consistentes.

## 13. Critérios de Aceite

Para cada módulo e caso de uso, os critérios de aceite definidos nas seções anteriores servem como base para validar a implementação. Além disso, os seguintes critérios gerais se aplicam:

*   **Funcionalidade:** Todas as funcionalidades descritas operam conforme o esperado.
*   **Segurança:** O isolamento de dados por tenant é garantido. As permissões de usuário são respeitadas. O sistema é resistente a vulnerabilidades comuns.
*   **Performance:** As operações críticas respondem dentro dos limites de tempo aceitáveis.
*   **Confiabilidade:** O sistema lida com erros de forma graciosa e recupera-se de falhas.
*   **Manutenibilidade:** O código é limpo, modular e fácil de entender e modificar.
*   **Escalabilidade:** A arquitetura suporta o crescimento do número de tenants e usuários.

## Conclusão

Este documento fornece uma especificação técnica abrangente para o backend do OficinaPro, cobrindo a arquitetura multi-tenant, a estrutura de camadas, os casos de uso detalhados, as integrações com Supabase e Stripe, e as estratégias de segurança, auditoria e performance. A aderência a esta especificação garantirá que o desenvolvimento do backend seja robusto, seguro e escalável, fornecendo uma base sólida para o funcionamento do sistema. Com esta base, as ferramentas de IA (Cursor, Claude, ChatGPT) e os desenvolvedores terão todas as informações necessárias para construir a lógica de negócio do OficinaPro com precisão e qualidade.
