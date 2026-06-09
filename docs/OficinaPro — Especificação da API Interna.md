# OficinaPro — Especificação da API Interna

**Autor:** Manus AI  
**Data:** 09 de junho de 2026  
**Versão:** 1.0  
**Finalidade:** Este documento detalha a especificação da API interna do sistema OficinaPro, um SaaS multiempresa para oficinas mecânicas. Ele serve como guia oficial para a comunicação entre o frontend e o backend, abrangendo Server Actions, Route Handlers, webhooks da Stripe, padrões de resposta, tratamento de erros e requisitos de segurança.

## 1. Padrões Gerais da API

### 1.1 Autenticação e Autorização

O sistema utiliza o Supabase Auth para autenticação e autorização, baseado em JSON Web Tokens (JWT).

*   **Autenticação:** O frontend obtém um JWT do Supabase Auth após o login do usuário. Este token é enviado em todas as requisições subsequentes (Server Actions e Route Handlers) no cabeçalho `Authorization: Bearer <token>`.
*   **Autorização:** A autorização é baseada nas `roles` do usuário (`owner`, `admin`, `employee`) e no `tenant_id` extraído do JWT. As políticas de Row Level Security (RLS) no PostgreSQL e as verificações de permissão na camada de `use-cases` do backend garantem que o usuário só possa acessar e manipular dados aos quais tem direito.

### 1.2 Multi-tenancy

Todas as operações da API são contextuais ao `tenant_id` do usuário autenticado. O `tenant_id` é extraído do JWT e utilizado para filtrar dados e aplicar regras de negócio, garantindo o isolamento completo entre os tenants.

### 1.3 Padrões de Resposta

As respostas da API seguirão um padrão consistente para facilitar o consumo pelo frontend.

*   **Sucesso (2xx):**
    *   **`200 OK`:** Para operações de leitura (GET) e atualizações bem-sucedidas (PUT/PATCH) que retornam dados.
    *   **`201 Created`:** Para criação de novos recursos (POST).
    *   **`204 No Content`:** Para operações bem-sucedidas que não retornam dados (ex: DELETE).
    *   **Corpo da Resposta:** Em caso de sucesso com dados, o corpo da resposta será um objeto JSON contendo o recurso solicitado ou uma lista de recursos.
    ```json
    // Exemplo de 200 OK para um único recurso
    {
      "data": { /* objeto do recurso */ },
      "message": "Recurso obtido com sucesso."
    }

    // Exemplo de 200 OK para uma lista de recursos com paginação
    {
      "data": [ /* array de recursos */ ],
      "pagination": {
        "total": 100,
        "page": 1,
        "pageSize": 10,
        "totalPages": 10
      },
      "message": "Lista de recursos obtida com sucesso."
    }
    ```

*   **Erro (4xx, 5xx):**
    *   **Corpo da Resposta:** Em caso de erro, o corpo da resposta será um objeto JSON contendo detalhes do erro.
    ```json
    {
      "error": {
        "code": "INVALID_INPUT",
        "message": "Os dados fornecidos são inválidos.",
        "details": [
          { "field": "email", "message": "Formato de e-mail inválido." },
          { "field": "password", "message": "A senha deve ter no mínimo 6 caracteres." }
        ]
      }
    }
    ```

### 1.4 Tratamento de Erros

O backend implementará um tratamento de erros consistente, mapeando exceções internas para códigos de status HTTP e mensagens de erro padronizadas.

*   **`400 Bad Request`:** Erros de validação de entrada (ex: formato inválido, campos ausentes).
*   **`401 Unauthorized`:** Falha na autenticação (token ausente, inválido ou expirado).
*   **`403 Forbidden`:** Usuário autenticado, mas sem permissão para realizar a ação (autorização).
*   **`404 Not Found`:** Recurso solicitado não encontrado.
*   **`409 Conflict`:** Conflito de recursos (ex: e-mail já registrado, placa duplicada, transição de status inválida).
*   **`422 Unprocessable Entity`:** Erros de regras de negócio que impedem a conclusão da operação.
*   **`500 Internal Server Error`:** Erros inesperados no servidor.

### 1.5 Rate Limiting

Para proteger a API contra abusos e garantir a disponibilidade, o rate limiting será aplicado em endpoints sensíveis (ex: login, registro, recuperação de senha) e globalmente para todas as requisições.

*   **Implementação:** Pode ser configurado no nível do Vercel ou através de uma biblioteca no backend.
*   **Resposta:** Requisições que excederem o limite receberão um `429 Too Many Requests`.

### 1.6 Idempotência

Operações que modificam o estado do sistema (POST, PUT, PATCH, DELETE) devem ser idempotentes, especialmente aquelas que podem ser retentadas pelo cliente ou por webhooks.

*   **Implementação:** Utilização de um cabeçalho `Idempotency-Key` (UUID) nas requisições do cliente. O backend armazenará o resultado da primeira requisição com aquela chave e retornará o mesmo resultado para requisições subsequentes com a mesma chave dentro de um período de tempo.
*   **Webhooks Stripe:** Os webhooks da Stripe já incluem um cabeçalho `Stripe-Signature` que pode ser usado para verificar a autenticidade e um `id` de evento que pode ser usado para idempotência.

## 2. Server Actions

Os Server Actions são a principal forma de interação do frontend com o backend no Next.js 15. Eles permitem invocar funções do servidor diretamente do cliente, com segurança e tipagem.

### 2.1 Autenticação

#### `signIn(formData: FormData)`

*   **Objetivo:** Autenticar um usuário com e-mail e senha.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente, é uma função exportada.
*   **Perfis autorizados:** Nenhum (usuário não autenticado).
*   **Request Body:** `FormData` contendo:
    *   `email`: string (obrigatório)
    *   `password`: string (obrigatório)
*   **Response:**
    ```json
    {
      "data": {
        "user": { /* objeto do usuário Supabase */ },
        "tenant": { /* objeto do tenant associado */ }
      },
      "message": "Login realizado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: E-mail ou senha ausentes/inválidos.
    *   `401 Unauthorized`: Credenciais inválidas.
    *   `403 Forbidden`: Usuário inativo, tenant inativo ou assinatura inválida.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Formato de e-mail, presença de senha.
*   **Permissões:** Nenhuma (acesso público).
*   **Regras de negócio:** Verificar status do usuário, tenant e assinatura.
*   **Integração com Supabase:** `AuthService.signInWithPassword`.
*   **Auditoria:** Registrar evento de login (sucesso/falha).

#### `signUp(formData: FormData)`

*   **Objetivo:** Registrar um novo usuário e criar um novo tenant.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** Nenhum.
*   **Request Body:** `FormData` contendo:
    *   `fullName`: string (obrigatório)
    *   `email`: string (obrigatório)
    *   `password`: string (obrigatório)
    *   `companyName`: string (obrigatório)
*   **Response:**
    ```json
    {
      "data": {
        "user": { /* objeto do usuário Supabase */ },
        "tenant": { /* objeto do tenant criado */ }
      },
      "message": "Cadastro realizado com sucesso. Verifique seu e-mail para confirmar."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `409 Conflict`: E-mail já registrado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Formato de e-mail, força da senha, unicidade do e-mail e nome da empresa.
*   **Permissões:** Nenhuma.
*   **Regras de negócio:** Criação de tenant, associação de owner, início de trial.
*   **Integração com Supabase:** `AuthService.signUp`, `TenantService.createTenant`, `SubscriptionService.startTrial`.
*   **Auditoria:** Registrar evento de cadastro.

#### `sendPasswordResetEmail(formData: FormData)`

*   **Objetivo:** Enviar link de recuperação de senha.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** Nenhum.
*   **Request Body:** `FormData` contendo:
    *   `email`: string (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Se o e-mail estiver registrado, um link de recuperação foi enviado."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: E-mail ausente/inválido.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Formato de e-mail.
*   **Permissões:** Nenhuma.
*   **Regras de negócio:** Não revelar se o e-mail existe.
*   **Integração com Supabase:** `AuthService.sendPasswordResetEmail`.
*   **Auditoria:** Registrar tentativa de recuperação de senha.

#### `updatePassword(formData: FormData)`

*   **Objetivo:** Alterar a senha do usuário.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** Usuário com token de redefinição válido.
*   **Request Body:** `FormData` contendo:
    *   `newPassword`: string (obrigatório)
    *   `confirmPassword`: string (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Senha alterada com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Senhas ausentes/inválidas ou não coincidem.
    *   `401 Unauthorized`: Token de redefinição inválido/expirado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Força da senha, confirmação de senha.
*   **Permissões:** Token de redefinição.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `AuthService.updateUserPassword`.
*   **Auditoria:** Registrar alteração de senha.

#### `signOut()`

*   **Objetivo:** Deslogar o usuário.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** Usuário autenticado.
*   **Request Body:** Nenhum.
*   **Response:**
    ```json
    {
      "message": "Logout realizado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Nenhuma.
*   **Permissões:** Usuário autenticado.
*   **Regras de negócio:** Nenhuma.
*   **Integração com Supabase:** `AuthService.signOut`.
*   **Auditoria:** Registrar evento de logout.

### 2.2 Empresas (Tenants)

#### `updateTenant(formData: FormData)`

*   **Objetivo:** Atualizar informações da oficina (tenant).
*   **Método HTTP:** PUT/PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `companyName`: string (opcional)
    *   `cnpj`: string (opcional)
    *   `stateRegistration`: string (opcional)
    *   `email`: string (opcional)
    *   `phone`: string (opcional)
    *   `whatsapp`: string (opcional)
    *   `logoFile`: File (opcional, para upload)
    *   `address`: JSON string (opcional)
    *   `financialSettings`: JSON string (opcional)
    *   `operationalSettings`: JSON string (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto do tenant atualizado */ },
      "message": "Informações da oficina atualizadas com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Tenant não encontrado.
    *   `409 Conflict`: Nome da empresa já existe.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Formato dos campos, unicidade do nome da empresa.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `TenantService.updateTenant`, `Supabase Storage` para logo.
*   **Auditoria:** Registrar atualização do tenant.

### 2.3 Usuários

#### `inviteUser(formData: FormData)`

*   **Objetivo:** Convidar um novo usuário para o tenant.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `email`: string (obrigatório)
    *   `role`: string (`tenant_role` enum, obrigatório)
*   **Response:**
    ```json
    {
      "message": "Convite enviado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão ou limite de usuários excedido.
    *   `409 Conflict`: E-mail já registrado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Formato de e-mail, `role` válida.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Limite de usuários por plano de assinatura.
*   **Integração com Supabase:** `AuthService.inviteUserByEmail`, `TenantService.addTenantUser`.
*   **Auditoria:** Registrar convite de usuário.

#### `updateUserRole(formData: FormData)`

*   **Objetivo:** Alterar a role de um usuário no tenant.
*   **Método HTTP:** PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `userId`: UUID (obrigatório)
    *   `newRole`: string (`tenant_role` enum, obrigatório)
*   **Response:**
    ```json
    {
      "message": "Permissão do usuário atualizada com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão ou violação de regras de negócio.
    *   `404 Not Found`: Usuário não encontrado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `userId` válido, `newRole` válida.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Restrições para alterar a própria role ou a role de um `owner`.
*   **Integração com Supabase:** `TenantService.updateTenantUserRole`.
*   **Auditoria:** Registrar alteração de permissão.

#### `removeUser(formData: FormData)`

*   **Objetivo:** Remover um usuário de um tenant.
*   **Método HTTP:** DELETE (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `userId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Usuário removido com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `userId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão ou violação de regras de negócio.
    *   `404 Not Found`: Usuário não encontrado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `userId` válido.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Restrições para remover o único `owner` ou um `owner` por um `admin`.
*   **Integração com Supabase:** `TenantService.removeTenantUser`.
*   **Auditoria:** Registrar remoção de usuário.

### 2.4 Clientes

#### `createCustomer(formData: FormData)`

*   **Objetivo:** Cadastrar um novo cliente.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `name`: string (obrigatório)
    *   `document`: string (obrigatório, CPF/CNPJ)
    *   `email`: string (opcional)
    *   `phone`: string (opcional)
    *   `address`: JSON string (opcional)
    *   `notes`: string (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto do cliente criado */ },
      "message": "Cliente cadastrado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `409 Conflict`: Documento já existe.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `name`, `document` (formato e unicidade).
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Unicidade do documento por tenant.
*   **Integração com Supabase:** `CustomerService.createCustomer`.
*   **Auditoria:** Registrar criação de cliente.

#### `updateCustomer(formData: FormData)`

*   **Objetivo:** Atualizar dados de um cliente existente.
*   **Método HTTP:** PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `customerId`: UUID (obrigatório)
    *   `name`: string (opcional)
    *   `document`: string (opcional)
    *   `email`: string (opcional)
    *   `phone`: string (opcional)
    *   `address`: JSON string (opcional)
    *   `notes`: string (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto do cliente atualizado */ },
      "message": "Cliente atualizado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Cliente não encontrado.
    *   `409 Conflict`: Documento já existe para outro cliente.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `customerId` válido, formato dos campos, unicidade do documento.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Unicidade do documento por tenant.
*   **Integração com Supabase:** `CustomerService.updateCustomer`.
*   **Auditoria:** Registrar atualização de cliente.

#### `archiveCustomer(formData: FormData)`

*   **Objetivo:** Arquivar (soft delete) um cliente.
*   **Método HTTP:** DELETE (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `customerId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Cliente arquivado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `customerId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Cliente não encontrado.
    *   `409 Conflict`: Cliente possui OS abertas.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `customerId` válido.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Não permitir arquivar cliente com OS abertas.
*   **Integração com Supabase:** `CustomerService.softDeleteCustomer`.
*   **Auditoria:** Registrar arquivamento de cliente.

#### `getCustomers(queryParams: URLSearchParams)`

*   **Objetivo:** Listar e filtrar clientes.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `searchQuery`: string (opcional, busca por nome, documento, e-mail)
    *   `status`: string (`active`, `inactive`, `all`, opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
    *   `sortBy`: string (opcional, ex: `name`, `createdAt`)
    *   `sortOrder`: string (`asc`, `desc`, opcional, padrão `asc`)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de cliente */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação e ordenação.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `CustomerService.findCustomers`.
*   **Auditoria:** Nenhuma (leitura).

#### `getCustomerById(customerId: UUID)`

*   **Objetivo:** Obter detalhes de um cliente específico.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `customerId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "data": { /* objeto do cliente */ }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `customerId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Cliente não encontrado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `customerId` válido.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `CustomerService.getCustomerById`.
*   **Auditoria:** Nenhuma (leitura).

### 2.5 Veículos

#### `createVehicle(formData: FormData)`

*   **Objetivo:** Cadastrar um novo veículo.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `customerId`: UUID (obrigatório)
    *   `plate`: string (obrigatório)
    *   `brand`: string (obrigatório)
    *   `model`: string (obrigatório)
    *   `year`: number (obrigatório)
    *   `color`: string (opcional)
    *   `chassis`: string (opcional)
    *   `engine`: string (opcional)
    *   `fuelType`: string (`fuel_type` enum, opcional)
    *   `notes`: string (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto do veículo criado */ },
      "message": "Veículo cadastrado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Cliente não encontrado.
    *   `409 Conflict`: Placa já existe.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `customerId`, `plate`, `brand`, `model`, `year` (formato e unicidade da placa).
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Unicidade da placa por tenant, associação com cliente existente.
*   **Integração com Supabase:** `VehicleService.createVehicle`.
*   **Auditoria:** Registrar criação de veículo.

#### `updateVehicle(formData: FormData)`

*   **Objetivo:** Atualizar dados de um veículo existente.
*   **Método HTTP:** PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `vehicleId`: UUID (obrigatório)
    *   `customerId`: UUID (opcional)
    *   `plate`: string (opcional)
    *   `brand`: string (opcional)
    *   `model`: string (opcional)
    *   `year`: number (opcional)
    *   `color`: string (opcional)
    *   `chassis`: string (opcional)
    *   `engine`: string (opcional)
    *   `fuelType`: string (opcional)
    *   `notes`: string (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto do veículo atualizado */ },
      "message": "Veículo atualizado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Veículo ou cliente não encontrado.
    *   `409 Conflict`: Placa já existe para outro veículo.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `vehicleId` válido, formato dos campos, unicidade da placa.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Unicidade da placa por tenant, associação com cliente existente.
*   **Integração com Supabase:** `VehicleService.updateVehicle`.
*   **Auditoria:** Registrar atualização de veículo.

#### `getVehicleHistory(vehicleId: UUID)`

*   **Objetivo:** Obter o histórico de ordens de serviço de um veículo.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `vehicleId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de ordem de serviço */ ]
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `vehicleId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Veículo não encontrado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `vehicleId` válido.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `VehicleService.getVehicleWorkOrderHistory`.
*   **Auditoria:** Nenhuma (leitura).

#### `getVehicles(queryParams: URLSearchParams)`

*   **Objetivo:** Listar e filtrar veículos.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `searchQuery`: string (opcional, busca por placa, marca, modelo)
    *   `customerId`: UUID (opcional, filtra por cliente)
    *   `status`: string (`active`, `inactive`, `all`, opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
    *   `sortBy`: string (opcional, ex: `plate`, `createdAt`)
    *   `sortOrder`: string (`asc`, `desc`, opcional, padrão `asc`)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de veículo */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação e ordenação.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `VehicleService.findVehicles`.
*   **Auditoria:** Nenhuma (leitura).

### 2.6 Serviços (Catálogo)

#### `createService(formData: FormData)`

*   **Objetivo:** Cadastrar um novo serviço no catálogo da oficina.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `name`: string (obrigatório)
    *   `description`: string (opcional)
    *   `defaultPrice`: number (obrigatório)
    *   `estimatedTime`: number (opcional, em minutos)
*   **Response:**
    ```json
    {
      "data": { /* objeto do serviço criado */ },
      "message": "Serviço cadastrado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `409 Conflict`: Nome do serviço já existe.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `name` (unicidade), `defaultPrice` (numérico, positivo).
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Unicidade do nome do serviço por tenant.
*   **Integração com Supabase:** `ServiceCatalogService.createService`.
*   **Auditoria:** Registrar criação de serviço.

#### `updateService(formData: FormData)`

*   **Objetivo:** Atualizar dados de um serviço existente.
*   **Método HTTP:** PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `serviceId`: UUID (obrigatório)
    *   `name`: string (opcional)
    *   `description`: string (opcional)
    *   `defaultPrice`: number (opcional)
    *   `estimatedTime`: number (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto do serviço atualizado */ },
      "message": "Serviço atualizado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Serviço não encontrado.
    *   `409 Conflict`: Nome do serviço já existe para outro serviço.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `serviceId` válido, `name` (unicidade), `defaultPrice` (numérico, positivo).
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Unicidade do nome do serviço por tenant.
*   **Integração com Supabase:** `ServiceCatalogService.updateService`.
*   **Auditoria:** Registrar atualização de serviço.

#### `archiveService(formData: FormData)`

*   **Objetivo:** Arquivar (soft delete) um serviço.
*   **Método HTTP:** DELETE (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `serviceId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Serviço arquivado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `serviceId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Serviço não encontrado.
    *   `409 Conflict`: Serviço em uso em OS aberta.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `serviceId` válido.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Não permitir arquivar serviço em uso em OS aberta.
*   **Integração com Supabase:** `ServiceCatalogService.softDeleteService`.
*   **Auditoria:** Registrar arquivamento de serviço.

#### `getServices(queryParams: URLSearchParams)`

*   **Objetivo:** Listar e filtrar serviços do catálogo.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `searchQuery`: string (opcional, busca por nome, descrição)
    *   `status`: string (`active`, `inactive`, `all`, opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
    *   `sortBy`: string (opcional, ex: `name`, `defaultPrice`)
    *   `sortOrder`: string (`asc`, `desc`, opcional, padrão `asc`)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de serviço */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação e ordenação.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `ServiceCatalogService.findServices`.
*   **Auditoria:** Nenhuma (leitura).

#### `getServiceById(serviceId: UUID)`

*   **Objetivo:** Obter detalhes de um serviço específico.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `serviceId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "data": { /* objeto do serviço */ }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `serviceId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Serviço não encontrado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `serviceId` válido.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `ServiceCatalogService.getServiceById`.
*   **Auditoria:** Nenhuma (leitura).

### 2.7 Produtos (Catálogo)

#### `createProduct(formData: FormData)`

*   **Objetivo:** Cadastrar um novo produto/peça no catálogo da oficina.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `name`: string (obrigatório)
    *   `sku`: string (opcional)
    *   `description`: string (opcional)
    *   `costPrice`: number (obrigatório)
    *   `salePrice`: number (obrigatório)
    *   `stockQuantity`: number (obrigatório)
    *   `minStockQuantity`: number (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto do produto criado */ },
      "message": "Produto cadastrado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `409 Conflict`: Nome ou SKU do produto já existe.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `name` (unicidade), `sku` (unicidade), preços e quantidades (numéricos, positivos).
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Unicidade do nome/SKU do produto por tenant.
*   **Integração com Supabase:** `ProductCatalogService.createProduct`.
*   **Auditoria:** Registrar criação de produto.

#### `updateProduct(formData: FormData)`

*   **Objetivo:** Atualizar dados de um produto existente.
*   **Método HTTP:** PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `productId`: UUID (obrigatório)
    *   `name`: string (opcional)
    *   `sku`: string (opcional)
    *   `description`: string (opcional)
    *   `costPrice`: number (opcional)
    *   `salePrice`: number (opcional)
    *   `stockQuantity`: number (opcional)
    *   `minStockQuantity`: number (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto do produto atualizado */ },
      "message": "Produto atualizado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Produto não encontrado.
    *   `409 Conflict`: Nome ou SKU do produto já existe para outro produto.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `productId` válido, `name` (unicidade), `sku` (unicidade), preços e quantidades (numéricos, positivos).
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Unicidade do nome/SKU do produto por tenant.
*   **Integração com Supabase:** `ProductCatalogService.updateProduct`.
*   **Auditoria:** Registrar atualização de produto.

#### `archiveProduct(formData: FormData)`

*   **Objetivo:** Arquivar (soft delete) um produto.
*   **Método HTTP:** DELETE (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `productId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Produto arquivado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `productId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Produto não encontrado.
    *   `409 Conflict`: Produto em uso em OS aberta.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `productId` válido.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Não permitir arquivar produto em uso em OS aberta.
*   **Integração com Supabase:** `ProductCatalogService.softDeleteProduct`.
*   **Auditoria:** Registrar arquivamento de produto.

#### `getProducts(queryParams: URLSearchParams)`

*   **Objetivo:** Listar e filtrar produtos do catálogo.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `searchQuery`: string (opcional, busca por nome, SKU, descrição)
    *   `status`: string (`active`, `inactive`, `all`, opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
    *   `sortBy`: string (opcional, ex: `name`, `salePrice`, `stockQuantity`)
    *   `sortOrder`: string (`asc`, `desc`, opcional, padrão `asc`)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de produto */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação e ordenação.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `ProductCatalogService.findProducts`.
*   **Auditoria:** Nenhuma (leitura).

#### `getProductById(productId: UUID)`

*   **Objetivo:** Obter detalhes de um produto específico.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `productId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "data": { /* objeto do produto */ }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `productId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Produto não encontrado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `productId` válido.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `ProductCatalogService.getProductById`.
*   **Auditoria:** Nenhuma (leitura).

### 2.8 Ordens de Serviço

#### `createWorkOrder(formData: FormData)`

*   **Objetivo:** Abrir uma nova Ordem de Serviço.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `customerId`: UUID (obrigatório)
    *   `vehicleId`: UUID (obrigatório)
    *   `clientReport`: string (obrigatório)
    *   `entryMileage`: number (obrigatório)
    *   `expectedDeliveryDate`: string (data, opcional)
    *   `responsibleEmployeeId`: UUID (opcional)
    *   `internalNotes`: string (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto da OS criada */ },
      "message": "Ordem de Serviço aberta com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Cliente ou veículo não encontrado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `customerId`, `vehicleId`, `clientReport`, `entryMileage` (formato, existência).
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Geração de número de OS, associação cliente-veículo.
*   **Integração com Supabase:** `WorkOrderService.createWorkOrder`.
*   **Auditoria:** Registrar criação de OS.

#### `updateWorkOrder(formData: FormData)`

*   **Objetivo:** Atualizar dados gerais de uma Ordem de Serviço.
*   **Método HTTP:** PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `workOrderId`: UUID (obrigatório)
    *   `clientReport`: string (opcional)
    *   `diagnosis`: string (opcional)
    *   `entryMileage`: number (opcional)
    *   `exitMileage`: number (opcional)
    *   `expectedDeliveryDate`: string (data, opcional)
    *   `deliveryDate`: string (data, opcional)
    *   `responsibleEmployeeId`: UUID (opcional)
    *   `internalNotes`: string (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto da OS atualizada */ },
      "message": "Ordem de Serviço atualizada com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: OS não encontrada.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `workOrderId` válido, formato dos campos.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `WorkOrderService.updateWorkOrder`.
*   **Auditoria:** Registrar atualização de OS.

#### `updateWorkOrderStatus(formData: FormData)`

*   **Objetivo:** Alterar o status de uma Ordem de Serviço.
*   **Método HTTP:** PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `workOrderId`: UUID (obrigatório)
    *   `newStatus`: string (`work_order_status` enum, obrigatório)
    *   `notes`: string (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto da OS atualizada */ },
      "message": "Status da Ordem de Serviço atualizado para {newStatus}."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão ou transição de status inválida.
    *   `404 Not Found`: OS não encontrada.
    *   `422 Unprocessable Entity`: Status inválido.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `workOrderId` válido, `newStatus` válido.
*   **Permissões:** `owner`, `admin`, `employee` do tenant (com restrições de status).
*   **Regras de negócio:** Fluxo de transição de status da OS.
*   **Integração com Supabase:** `WorkOrderService.updateWorkOrderStatus`.
*   **Auditoria:** Registrar alteração de status da OS.

#### `approveWorkOrder(formData: FormData)`

*   **Objetivo:** Aprovar uma Ordem de Serviço.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `workOrderId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Ordem de Serviço aprovada com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `workOrderId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: OS não encontrada.
    *   `409 Conflict`: OS não está em status de espera por aprovação.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `workOrderId` válido.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** OS deve estar em `waiting_approval`.
*   **Integração com Supabase:** `WorkOrderService.approveWorkOrder`.
*   **Auditoria:** Registrar aprovação de OS.

#### `addWorkOrderService(formData: FormData)`

*   **Objetivo:** Adicionar um serviço a uma Ordem de Serviço.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `workOrderId`: UUID (obrigatório)
    *   `serviceId`: UUID (opcional, se for do catálogo)
    *   `description`: string (obrigatório)
    *   `quantity`: number (obrigatório)
    *   `unitPrice`: number (obrigatório)
    *   `discount`: number (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto da OS atualizada com serviços */ },
      "message": "Serviço adicionado à Ordem de Serviço."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: OS ou serviço do catálogo não encontrado.
    *   `409 Conflict`: OS em status finalizado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `workOrderId` válido, `description`, `quantity`, `unitPrice` (formato, valores positivos).
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** OS não pode estar finalizada, recalcular total da OS.
*   **Integração com Supabase:** `WorkOrderService.addWorkOrderService`.
*   **Auditoria:** Registrar adição de serviço à OS.

#### `addWorkOrderProduct(formData: FormData)`

*   **Objetivo:** Adicionar um produto a uma Ordem de Serviço.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `workOrderId`: UUID (obrigatório)
    *   `productId`: UUID (opcional, se for do catálogo)
    *   `description`: string (obrigatório)
    *   `quantity`: number (obrigatório)
    *   `unitPrice`: number (obrigatório)
    *   `discount`: number (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto da OS atualizada com produtos */ },
      "message": "Produto adicionado à Ordem de Serviço."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: OS ou produto do catálogo não encontrado.
    *   `409 Conflict`: OS em status finalizado, estoque insuficiente.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `workOrderId` válido, `description`, `quantity`, `unitPrice` (formato, valores positivos).
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** OS não pode estar finalizada, recalcular total da OS, verificar e atualizar estoque.
*   **Integração com Supabase:** `WorkOrderService.addWorkOrderProduct`.
*   **Auditoria:** Registrar adição de produto à OS.

#### `finalizeWorkOrder(formData: FormData)`

*   **Objetivo:** Finalizar uma Ordem de Serviço.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `workOrderId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Ordem de Serviço finalizada com sucesso. Conta a receber gerada."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `workOrderId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: OS não encontrada.
    *   `409 Conflict`: OS não está em status válido para finalização.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `workOrderId` válido.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** OS deve estar em `in_progress` ou `approved`, gerar conta a receber.
*   **Integração com Supabase:** `WorkOrderService.finalizeWorkOrder`.
*   **Auditoria:** Registrar finalização de OS.

#### `reopenWorkOrder(formData: FormData)`

*   **Objetivo:** Reabrir uma Ordem de Serviço finalizada.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `workOrderId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Ordem de Serviço reaberta com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `workOrderId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: OS não encontrada.
    *   `409 Conflict`: OS não está em status válido para reabertura.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `workOrderId` válido.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** OS deve estar em `completed` ou `delivered`.
*   **Integração com Supabase:** `WorkOrderService.reopenWorkOrder`.
*   **Auditoria:** Registrar reabertura de OS.

#### `cancelWorkOrder(formData: FormData)`

*   **Objetivo:** Cancelar uma Ordem de Serviço.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `workOrderId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Ordem de Serviço cancelada com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `workOrderId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: OS não encontrada.
    *   `409 Conflict`: OS em status finalizado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `workOrderId` válido.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** OS não pode estar finalizada, reverter estoque de produtos.
*   **Integração com Supabase:** `WorkOrderService.cancelWorkOrder`.
*   **Auditoria:** Registrar cancelamento de OS.

#### `getWorkOrders(queryParams: URLSearchParams)`

*   **Objetivo:** Listar e filtrar Ordens de Serviço.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `searchQuery`: string (opcional, busca por número da OS, cliente, veículo)
    *   `status`: string (`work_order_status` enum, opcional)
    *   `customerId`: UUID (opcional)
    *   `vehicleId`: UUID (opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
    *   `sortBy`: string (opcional, ex: `createdAt`, `expectedDeliveryDate`)
    *   `sortOrder`: string (`asc`, `desc`, opcional, padrão `asc`)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de OS */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação e ordenação.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `WorkOrderService.findWorkOrders`.
*   **Auditoria:** Nenhuma (leitura).

#### `getWorkOrderById(workOrderId: UUID)`

*   **Objetivo:** Obter detalhes completos de uma Ordem de Serviço.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin`, `employee` do tenant.
*   **Query Parameters:**
    *   `workOrderId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "data": { /* objeto completo da OS com serviços, produtos, histórico */ }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `workOrderId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: OS não encontrada.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `workOrderId` válido.
*   **Permissões:** `owner`, `admin`, `employee` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `WorkOrderService.getWorkOrderById`.
*   **Auditoria:** Nenhuma (leitura).

### 2.9 Financeiro

#### `createIncome(formData: FormData)`

*   **Objetivo:** Registrar uma nova receita manual.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `description`: string (obrigatório)
    *   `amount`: number (obrigatório)
    *   `transactionDate`: string (data, obrigatório)
    *   `paymentMethod`: string (`payment_method` enum, obrigatório)
    *   `customerId`: UUID (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto da transação de receita criada */ },
      "message": "Receita registrada com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `description`, `amount` (numérico, positivo), `transactionDate`, `paymentMethod`.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `FinancialService.createIncome`.
*   **Auditoria:** Registrar criação de receita.

#### `createExpense(formData: FormData)`

*   **Objetivo:** Registrar uma nova despesa manual.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `description`: string (obrigatório)
    *   `amount`: number (obrigatório)
    *   `transactionDate`: string (data, obrigatório)
    *   `paymentMethod`: string (`payment_method` enum, obrigatório)
    *   `supplier`: string (opcional)
    *   `category`: string (opcional)
*   **Response:**
    ```json
    {
      "data": { /* objeto da transação de despesa criada */ },
      "message": "Despesa registrada com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `description`, `amount` (numérico, positivo), `transactionDate`, `paymentMethod`.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `FinancialService.createExpense`.
*   **Auditoria:** Registrar criação de despesa.

#### `getAccountsReceivable(queryParams: URLSearchParams)`

*   **Objetivo:** Listar e filtrar contas a receber.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Query Parameters:**
    *   `status`: string (`financial_status` enum, opcional)
    *   `dueDateStart`: string (data, opcional)
    *   `dueDateEnd`: string (data, opcional)
    *   `customerId`: UUID (opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
    *   `sortBy`: string (opcional, ex: `dueDate`, `amount`)
    *   `sortOrder`: string (`asc`, `desc`, opcional, padrão `asc`)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de conta a receber */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação e ordenação, formatos de data.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `FinancialService.findAccountsReceivable`.
*   **Auditoria:** Nenhuma (leitura).

#### `getAccountsPayable(queryParams: URLSearchParams)`

*   **Objetivo:** Listar e filtrar contas a pagar.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Query Parameters:**
    *   `status`: string (`financial_status` enum, opcional)
    *   `dueDateStart`: string (data, opcional)
    *   `dueDateEnd`: string (data, opcional)
    *   `supplier`: string (opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
    *   `sortBy`: string (opcional, ex: `dueDate`, `amount`)
    *   `sortOrder`: string (`asc`, `desc`, opcional, padrão `asc`)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de conta a pagar */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação e ordenação, formatos de data.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `FinancialService.findAccountsPayable`.
*   **Auditoria:** Nenhuma (leitura).

#### `getFinancialTransactions(queryParams: URLSearchParams)`

*   **Objetivo:** Listar e filtrar transações financeiras (fluxo de caixa).
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Query Parameters:**
    *   `type`: string (`transaction_type` enum, opcional)
    *   `paymentMethod`: string (`payment_method` enum, opcional)
    *   `dateStart`: string (data, opcional)
    *   `dateEnd`: string (data, opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
    *   `sortBy`: string (opcional, ex: `transactionDate`, `amount`)
    *   `sortOrder`: string (`asc`, `desc`, opcional, padrão `asc`)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de transação financeira */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação e ordenação, formatos de data.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `FinancialService.findFinancialTransactions`.
*   **Auditoria:** Nenhuma (leitura).

### 2.10 Assinaturas

#### `startTrial()`

*   **Objetivo:** Iniciar o período de trial para o tenant.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner` (apenas para tenants sem assinatura ativa).
*   **Request Body:** Nenhum.
*   **Response:**
    ```json
    {
      "message": "Período de trial iniciado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `409 Conflict`: Tenant já possui assinatura ou trial ativo.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Nenhuma.
*   **Permissões:** `owner` do tenant.
*   **Regras de negócio:** Apenas um trial por tenant, tenant não pode ter assinatura ativa.
*   **Integração com Supabase:** `SubscriptionService.startTrial`.
*   **Auditoria:** Registrar início de trial.

#### `upgradeSubscription(formData: FormData)`

*   **Objetivo:** Fazer upgrade do plano de assinatura do tenant.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `newPlanId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Upgrade de plano solicitado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `newPlanId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Plano não encontrado.
    *   `409 Conflict`: Plano inválido para upgrade.
    *   `500 Internal Server Error`: Erro na integração com Stripe.
*   **Validações:** `newPlanId` válido.
*   **Permissões:** `owner` do tenant.
*   **Regras de negócio:** `newPlanId` deve ser um plano superior ao atual.
*   **Integração com Stripe:** `SubscriptionService.upgradeSubscription`.
*   **Auditoria:** Registrar solicitação de upgrade.

#### `downgradeSubscription(formData: FormData)`

*   **Objetivo:** Fazer downgrade do plano de assinatura do tenant.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `newPlanId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Downgrade de plano solicitado com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `newPlanId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Plano não encontrado.
    *   `409 Conflict`: Plano inválido para downgrade.
    *   `500 Internal Server Error`: Erro na integração com Stripe.
*   **Validações:** `newPlanId` válido.
*   **Permissões:** `owner` do tenant.
*   **Regras de negócio:** `newPlanId` deve ser um plano inferior ao atual.
*   **Integração com Stripe:** `SubscriptionService.downgradeSubscription`.
*   **Auditoria:** Registrar solicitação de downgrade.

#### `cancelSubscription()`

*   **Objetivo:** Cancelar a assinatura ativa do tenant.
*   **Método HTTP:** POST (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner` do tenant.
*   **Request Body:** Nenhum.
*   **Response:**
    ```json
    {
      "message": "Assinatura cancelada com sucesso. O acesso será mantido até o final do período atual."
    }
    ```
*   **Códigos de erro:**
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Assinatura não encontrada.
    *   `500 Internal Server Error`: Erro na integração com Stripe.
*   **Validações:** Nenhuma.
*   **Permissões:** `owner` do tenant.
*   **Regras de negócio:** Assinatura é cancelada no final do período de faturamento.
*   **Integração com Stripe:** `SubscriptionService.cancelSubscription`.
*   **Auditoria:** Registrar cancelamento de assinatura.

#### `getSubscriptionDetails()`

*   **Objetivo:** Obter detalhes da assinatura atual do tenant.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Query Parameters:** Nenhum.
*   **Response:**
    ```json
    {
      "data": { /* objeto de detalhes da assinatura */ }
    }
    ```
*   **Códigos de erro:**
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `404 Not Found`: Assinatura não encontrada.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Nenhuma.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `SubscriptionService.getSubscriptionDetails`.
*   **Auditoria:** Nenhuma (leitura).

### 2.11 Configurações

#### `getSettings()`

*   **Objetivo:** Obter todas as configurações do tenant.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Query Parameters:** Nenhum.
*   **Response:**
    ```json
    {
      "data": { /* objeto com todas as configurações */ }
    }
    ```
*   **Códigos de erro:**
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Nenhuma.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `SettingsService.getSettings`.
*   **Auditoria:** Nenhuma (leitura).

#### `updateSetting(formData: FormData)`

*   **Objetivo:** Atualizar uma configuração específica do tenant.
*   **Método HTTP:** PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Request Body:** `FormData` contendo:
    *   `key`: string (obrigatório, nome da configuração)
    *   `value`: any (obrigatório, valor da configuração)
*   **Response:**
    ```json
    {
      "message": "Configuração atualizada com sucesso."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Dados ausentes/inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `key` e `value` válidos.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Validação do tipo de dado para cada `key` de configuração.
*   **Integração com Supabase:** `SettingsService.updateSetting`.
*   **Auditoria:** Registrar atualização de configuração.

### 2.12 Notificações

#### `getNotifications(queryParams: URLSearchParams)`

*   **Objetivo:** Listar notificações do usuário logado.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** Usuário autenticado.
*   **Query Parameters:**
    *   `status`: string (`read`, `unread`, `all`, opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de notificação */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação.
*   **Permissões:** Usuário autenticado.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `NotificationService.getNotifications`.
*   **Auditoria:** Nenhuma (leitura).

#### `markNotificationAsRead(formData: FormData)`

*   **Objetivo:** Marcar uma notificação como lida.
*   **Método HTTP:** PATCH (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** Usuário autenticado.
*   **Request Body:** `FormData` contendo:
    *   `notificationId`: UUID (obrigatório)
*   **Response:**
    ```json
    {
      "message": "Notificação marcada como lida."
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: `notificationId` ausente/inválido.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão (tentando marcar notificação de outro usuário).
    *   `404 Not Found`: Notificação não encontrada.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** `notificationId` válido.
*   **Permissões:** Usuário autenticado (apenas suas próprias notificações).
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `NotificationService.markNotificationAsRead`.
*   **Auditoria:** Nenhuma.

### 2.13 Auditoria

#### `getAuditLogs(queryParams: URLSearchParams)`

*   **Objetivo:** Listar logs de auditoria do tenant.
*   **Método HTTP:** GET (invocado via Server Action).
*   **Rota:** Não aplicável diretamente.
*   **Perfis autorizados:** `owner`, `admin` do tenant.
*   **Query Parameters:**
    *   `userId`: UUID (opcional, filtra por usuário)
    *   `action`: string (opcional, filtra por tipo de ação)
    *   `entityType`: string (opcional, filtra por tipo de entidade)
    *   `startDate`: string (data, opcional)
    *   `endDate`: string (data, opcional)
    *   `page`: number (opcional, padrão 1)
    *   `pageSize`: number (opcional, padrão 10)
*   **Response:**
    ```json
    {
      "data": [ /* array de objetos de log de auditoria */ ],
      "pagination": {
        "total": 50,
        "page": 1,
        "pageSize": 10,
        "totalPages": 5
      }
    }
    ```
*   **Códigos de erro:**
    *   `400 Bad Request`: Parâmetros de query inválidos.
    *   `401 Unauthorized`: Usuário não autenticado.
    *   `403 Forbidden`: Usuário sem permissão.
    *   `500 Internal Server Error`: Erro inesperado.
*   **Validações:** Parâmetros de paginação e filtros.
*   **Permissões:** `owner`, `admin` do tenant.
*   **Regras de negócio:** Nenhuma adicional.
*   **Integração com Supabase:** `AuditService.findAuditLogs`.
*   **Auditoria:** Nenhuma (leitura).

## 3. Route Handlers

Route Handlers são utilizados para endpoints que não se encaixam no modelo de Server Actions, como webhooks.

### 3.1 Webhooks da Stripe

#### `POST /api/stripe/webhook`

*   **Objetivo:** Receber e processar eventos de webhook da Stripe para sincronizar o estado das assinaturas.
*   **Método HTTP:** POST.
*   **Rota:** `/api/stripe/webhook`.
*   **Perfis autorizados:** Nenhum (autenticação via `Stripe-Signature`).
*   **Request Body:** Objeto de evento da Stripe.
*   **Response:**
    *   `200 OK`: Evento processado com sucesso.
    *   `400 Bad Request`: Assinatura do webhook inválida, corpo da requisição inválido.
    *   `500 Internal Server Error`: Erro no processamento do evento.
*   **Códigos de erro:**
    *   `400 Bad Request`: Assinatura inválida, corpo inválido.
    *   `500 Internal Server Error`: Erro no processamento.
*   **Validações:** Verificação da assinatura do webhook da Stripe.
*   **Permissões:** Nenhuma (acesso público, mas protegido por assinatura).
*   **Regras de negócio:** Processamento idempotente dos eventos, atualização do status da assinatura no banco de dados.
*   **Integração com Stripe:** `StripeService.handleStripeWebhook`.
*   **Auditoria:** Registrar eventos de webhook processados.

### 3.1.1 Eventos Chave Processados

*   `checkout.session.completed`
    *   **Objetivo:** Criar ou atualizar a assinatura do tenant após um checkout bem-sucedido.
    *   **Ações:** Extrair `customer_id` e `subscription_id` da Stripe, associar ao `tenant_id` no banco de dados, atualizar `subscription_status`.
*   `customer.subscription.created`
    *   **Objetivo:** Confirmar a criação de uma nova assinatura.
    *   **Ações:** Atualizar o status da assinatura no banco de dados para `active`.
*   `customer.subscription.updated`
    *   **Objetivo:** Sincronizar mudanças no plano, status ou período de trial da assinatura.
    *   **Ações:** Atualizar `subscription_status`, `current_period_end`, `stripe_price_id` e outros detalhes da assinatura no banco de dados.
*   `customer.subscription.deleted`
    *   **Objetivo:** Marcar a assinatura como cancelada ou expirada.
    *   **Ações:** Atualizar `subscription_status` para `canceled` ou `inactive` no banco de dados.
*   `invoice.payment_failed`
    *   **Objetivo:** Lidar com falhas de pagamento.
    *   **Ações:** Atualizar `subscription_status` para `past_due`, notificar o usuário, iniciar processo de dunning.

## Conclusão

Este documento detalha a especificação da API interna do OficinaPro, fornecendo um guia completo para o desenvolvimento do backend e a integração com o frontend. A adesão a estes padrões garantirá uma API robusta, segura, escalável e fácil de manter, permitindo que as ferramentas de IA e os desenvolvedores construam o sistema com alta qualidade e consistência.
