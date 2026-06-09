# OficinaPro — Estratégia de Testes

**Autor:** Manus AI  
**Data:** 09 de junho de 2026  
**Versão:** 1.0  
**Finalidade:** Este documento define a estratégia abrangente de testes para o sistema OficinaPro, um SaaS multiempresa para oficinas mecânicas. Ele detalha os tipos de testes, ferramentas, fluxos críticos e critérios de qualidade para garantir a robustez, segurança e performance do sistema.

## 1. Estratégia Geral de Testes

A estratégia de testes do OficinaPro segue uma abordagem em pirâmide, priorizando testes unitários rápidos e focados, complementados por testes de integração e testes end-to-end (E2E) para cobrir os fluxos mais críticos.

*   **Testes Unitários:** Foco na lógica de negócio isolada, funções e componentes individuais.
*   **Testes de Integração:** Verificação da comunicação entre módulos, serviços e integrações externas (Supabase, Stripe).
*   **Testes E2E:** Simulação de cenários de usuário completos através da interface do usuário.
*   **Testes de Segurança:** Validação do controle de acesso, isolamento de dados e proteção contra vulnerabilidades.
*   **Testes de Performance:** Avaliação da responsividade e escalabilidade do sistema.

**Ferramentas:**
*   **Vitest:** Para testes unitários e de integração (JavaScript/TypeScript).
*   **Playwright:** Para testes End-to-End (E2E).

## 2. Testes Unitários

Os testes unitários são a base da pirâmide de testes, garantindo que pequenas unidades de código funcionem conforme o esperado de forma isolada. Serão escritos utilizando **Vitest**.

### 2.1 Regras de Negócio

**Objetivo:** Validar a implementação correta das regras de negócio definidas.

**Pré-requisitos:** Funções ou métodos que encapsulam a lógica de negócio.

**Passos:**
1.  Criar cenários de teste para cada regra de negócio.
2.  Mockar dependências externas (banco de dados, APIs).
3.  Invocar a função/método com entradas válidas e inválidas.
4.  Verificar o resultado.

**Resultado esperado:** A função/método retorna o valor correto ou lança a exceção esperada.

**Critérios de aceite:**
*   Cobertura de todas as regras de negócio críticas.
*   Testes claros e concisos.

**Exemplos:**

#### `AuthService.signInWithPassword`
*   **Objetivo:** Testar o processo de login.
*   **Cenários:**
    *   Login com credenciais válidas.
    *   Login com e-mail inválido.
    *   Login com senha incorreta.
    *   Login com usuário inativo.
    *   Login com tenant inativo.

#### `WorkOrderService.updateWorkOrderStatus`
*   **Objetivo:** Validar as transições de status da Ordem de Serviço.
*   **Cenários:**
    *   Transição válida (ex: `draft` para `open`).
    *   Transição inválida (ex: `completed` para `draft`).
    *   Tentativa de alterar status por usuário sem permissão.

### 2.2 Cálculos Financeiros

**Objetivo:** Garantir a precisão de todos os cálculos financeiros.

**Pré-requisitos:** Funções de cálculo de impostos, descontos, totais de OS, etc.

**Passos:**
1.  Definir entradas com valores conhecidos.
2.  Executar a função de cálculo.
3.  Comparar o resultado com o valor esperado.

**Resultado esperado:** O cálculo retorna o valor exato.

**Critérios de aceite:**
*   Precisão em diferentes cenários (com/sem desconto, impostos).
*   Tratamento de valores de ponto flutuante.

**Exemplos:**

#### `calculateWorkOrderTotal(services, products)`
*   **Objetivo:** Calcular o valor total de uma OS.
*   **Cenários:**
    *   OS com apenas serviços.
    *   OS com apenas produtos.
    *   OS com serviços e produtos, com descontos.
    *   OS com valores zero.

### 2.3 Cálculos de OS

**Objetivo:** Validar cálculos específicos de Ordens de Serviço, como quilometragem percorrida, tempo de serviço, etc.

**Pré-requisitos:** Funções de cálculo relacionadas à OS.

**Passos:**
1.  Fornecer dados de entrada.
2.  Executar a função.
3.  Verificar o resultado.

**Resultado esperado:** O cálculo da OS é preciso.

**Critérios de aceite:**
*   Precisão dos cálculos.

**Exemplos:**

#### `calculateMileageDifference(entryMileage, exitMileage)`
*   **Objetivo:** Calcular a diferença de quilometragem.
*   **Cenários:**
    *   Valores válidos.
    *   `exitMileage` menor que `entryMileage` (deve lançar erro).

### 2.4 Validações

**Objetivo:** Assegurar que todas as validações de entrada de dados funcionem corretamente.

**Pré-requisitos:** Schemas de validação (ex: Zod) ou funções de validação.

**Passos:**
1.  Fornecer dados que devem passar na validação.
2.  Fornecer dados que devem falhar na validação.
3.  Verificar se a validação se comporta como esperado (sucesso ou erro).

**Resultado esperado:** A validação aceita dados válidos e rejeita dados inválidos.

**Critérios de aceite:**
*   Cobertura de todos os campos validados.
*   Mensagens de erro claras e específicas.

**Exemplos:**

#### `customerSchema.parse(data)`
*   **Objetivo:** Validar dados de entrada para criação/atualização de cliente.
*   **Cenários:**
    *   Dados de cliente válidos.
    *   E-mail com formato inválido.
    *   CPF/CNPJ com formato inválido.
    *   Campos obrigatórios ausentes.

## 3. Testes de Integração

Os testes de integração verificam a interação entre diferentes módulos do sistema e com serviços externos. Serão escritos utilizando **Vitest**.

### 3.1 Supabase (Database e Auth)

**Objetivo:** Validar a correta interação com o banco de dados PostgreSQL via Supabase e o sistema de autenticação.

**Pré-requisitos:** Uma instância de teste do Supabase (local ou remota) configurada.

**Passos:**
1.  Realizar operações de CRUD através dos `repositories` ou `services`.
2.  Verificar se os dados são persistidos/recuperados corretamente no banco.
3.  Testar fluxos de autenticação (login, registro, recuperação de senha) com o Supabase Auth.

**Resultado esperado:** As operações de banco de dados e autenticação funcionam conforme o esperado.

**Critérios de aceite:**
*   Dados são salvos e lidos corretamente.
*   Usuários são autenticados e seus tokens são válidos.
*   Erros de banco de dados são tratados adequadamente.

**Exemplos:**

#### `CustomerService.createCustomer`
*   **Objetivo:** Testar a criação de um cliente no banco de dados.
*   **Cenários:**
    *   Criar cliente com dados válidos.
    *   Tentar criar cliente com documento duplicado (deve falhar).
    *   Verificar se o `tenant_id` é corretamente associado.

#### `AuthService.signUp`
*   **Objetivo:** Testar o registro de um novo usuário e tenant.
*   **Cenários:**
    *   Registrar novo usuário e verificar se o tenant é criado e o usuário é associado como `owner`.
    *   Verificar se o trial é iniciado.

### 3.2 Stripe

**Objetivo:** Validar a integração com a API da Stripe para gerenciamento de assinaturas e pagamentos.

**Pré-requisitos:** Credenciais de teste da Stripe configuradas.

**Passos:**
1.  Simular chamadas à API da Stripe através do `StripeService`.
2.  Verificar se as assinaturas são criadas, atualizadas ou canceladas corretamente na Stripe.
3.  Testar o processamento de webhooks da Stripe com eventos simulados.

**Resultado esperado:** As operações da Stripe são executadas com sucesso e o estado do sistema é sincronizado.

**Critérios de aceite:**
*   Assinaturas são gerenciadas corretamente (upgrade, downgrade, cancelamento).
*   Webhooks são processados de forma idempotente e atualizam o banco de dados.
*   Erros da API da Stripe são tratados.

**Exemplos:**

#### `SubscriptionService.upgradeSubscription`
*   **Objetivo:** Testar o upgrade de um plano de assinatura.
*   **Cenários:**
    *   Upgrade de plano bem-sucedido e verificação na Stripe.
    *   Falha no upgrade devido a plano inválido.

#### `StripeWebhookHandler`
*   **Objetivo:** Testar o processamento de eventos de webhook.
*   **Cenários:**
    *   Receber evento `checkout.session.completed` e verificar criação de assinatura no sistema.
    *   Receber evento `customer.subscription.updated` e verificar atualização de status.

### 3.3 Row Level Security (RLS)

**Objetivo:** Garantir que as políticas de RLS no Supabase/PostgreSQL estejam corretamente configuradas para isolamento de dados por tenant e controle de acesso.

**Pré-requisitos:** Instância de teste do Supabase com RLS habilitado e políticas configuradas.

**Passos:**
1.  Autenticar como um usuário de um `tenant_id` específico.
2.  Tentar acessar dados de outro `tenant_id`.
3.  Tentar realizar operações (INSERT, UPDATE, DELETE) sem as permissões adequadas para a `role` do usuário.

**Resultado esperado:** Acesso a dados de outros tenants é negado. Operações sem permissão são negadas.

**Critérios de aceite:**
*   Nenhum dado de um tenant é visível para outro tenant.
*   Usuários só podem realizar ações permitidas por sua `role`.
*   Tentativas de acesso não autorizado resultam em erros `403 Forbidden`.

**Exemplos:**

#### `CustomerRepository.findCustomers`
*   **Objetivo:** Testar o isolamento de clientes por tenant.
*   **Cenários:**
    *   Usuário do Tenant A busca clientes e só vê clientes do Tenant A.
    *   Usuário do Tenant A tenta buscar cliente do Tenant B (deve retornar vazio ou erro).

### 3.4 Uploads (Supabase Storage)

**Objetivo:** Validar o upload e acesso a arquivos via Supabase Storage.

**Pré-requisitos:** Buckets do Supabase Storage configurados com políticas de segurança.

**Passos:**
1.  Realizar upload de um arquivo (ex: logo da oficina) através do Server Action.
2.  Verificar se o arquivo é armazenado no bucket correto.
3.  Tentar acessar um arquivo de outro tenant.

**Resultado esperado:** Arquivos são armazenados e acessíveis apenas por usuários autorizados do tenant correto.

**Critérios de aceite:**
*   Uploads são bem-sucedidos.
*   Acesso a arquivos é restrito por tenant.
*   URLs de arquivos são geradas corretamente.

**Exemplos:**

#### `updateTenantLogo(file)`
*   **Objetivo:** Testar o upload da logo da oficina.
*   **Cenários:**
    *   Upload de imagem e verificação de armazenamento no bucket `logos`.
    *   Tentativa de upload por usuário sem permissão.

## 4. Testes End-to-End (E2E)

Os testes E2E simulam a interação de um usuário real com a aplicação, cobrindo fluxos de ponta a ponta. Serão escritos utilizando **Playwright**.

### 4.1 Fluxos Críticos do Usuário

**Objetivo:** Validar a funcionalidade completa da aplicação, desde a interface do usuário até o backend e o banco de dados, para os cenários mais importantes.

**Pré-requisitos:** Aplicação frontend e backend rodando, banco de dados populado com dados de teste.

**Passos:**
1.  Navegar para as páginas da aplicação.
2.  Interagir com elementos da UI (cliques, preenchimento de formulários).
3.  Verificar o estado da UI e os dados exibidos.
4.  Verificar o estado do banco de dados após operações de escrita.

**Resultado esperado:** Os fluxos de usuário funcionam corretamente, a UI responde como esperado e os dados são persistidos.

**Critérios de aceite:**
*   Todos os fluxos críticos podem ser concluídos com sucesso.
*   A interface do usuário é responsiva e interativa.
*   Mensagens de erro são exibidas corretamente.

**Exemplos:**

#### 4.1.1 Login
*   **Objetivo:** Testar o processo de login de um usuário existente.
*   **Pré-requisitos:** Usuário de teste cadastrado.
*   **Passos:**
    1.  Navegar para a página de login.
    2.  Preencher e-mail e senha válidos.
    3.  Clicar no botão "Entrar".
    4.  Verificar redirecionamento para o Dashboard.
*   **Resultado esperado:** Usuário logado e redirecionado para o Dashboard.

#### 4.1.2 Cadastro
*   **Objetivo:** Testar o processo de cadastro de um novo usuário e criação de oficina.
*   **Pré-requisitos:** Nenhum.
*   **Passos:**
    1.  Navegar para a página de cadastro.
    2.  Preencher todos os campos (nome, e-mail, senha, nome da oficina).
    3.  Clicar no botão "Cadastrar".
    4.  Verificar redirecionamento para a página de confirmação de e-mail ou Dashboard (se auto-confirmado).
*   **Resultado esperado:** Novo usuário e tenant criados, e-mail de confirmação enviado (se aplicável).

#### 4.1.3 Criar Cliente
*   **Objetivo:** Testar o cadastro de um novo cliente.
*   **Pré-requisitos:** Usuário logado com permissão de `admin` ou `owner`.
*   **Passos:**
    1.  Navegar para a página de Clientes.
    2.  Clicar no botão "Novo Cliente".
    3.  Preencher formulário com dados válidos.
    4.  Clicar no botão "Salvar".
    5.  Verificar se o cliente aparece na lista e se os dados estão corretos.
*   **Resultado esperado:** Cliente cadastrado e visível na lista.

#### 4.1.4 Criar Veículo
*   **Objetivo:** Testar o cadastro de um novo veículo para um cliente existente.
*   **Pré-requisitos:** Usuário logado com permissão de `admin` ou `owner`, cliente existente.
*   **Passos:**
    1.  Navegar para a página de detalhes do Cliente.
    2.  Clicar no botão "Novo Veículo".
    3.  Preencher formulário com dados válidos (placa, marca, modelo).
    4.  Clicar no botão "Salvar".
    5.  Verificar se o veículo aparece na lista de veículos do cliente.
*   **Resultado esperado:** Veículo cadastrado e associado ao cliente.

#### 4.1.5 Abrir Ordem de Serviço
*   **Objetivo:** Testar a abertura de uma nova Ordem de Serviço.
*   **Pré-requisitos:** Usuário logado com permissão de `admin` ou `owner`, cliente e veículo existentes.
*   **Passos:**
    1.  Navegar para a página de Ordens de Serviço.
    2.  Clicar no botão "Nova OS".
    3.  Selecionar cliente e veículo.
    4.  Preencher dados da OS (relato do cliente, quilometragem).
    5.  Clicar no botão "Abrir OS".
    6.  Verificar se a OS é criada e redireciona para a página de detalhes da OS.
*   **Resultado esperado:** OS criada com status inicial e número gerado.

#### 4.1.6 Aprovar Ordem de Serviço
*   **Objetivo:** Testar a aprovação de uma Ordem de Serviço.
*   **Pré-requisitos:** Usuário logado com permissão de `admin` ou `owner`, OS em status `waiting_approval`.
*   **Passos:**
    1.  Navegar para a página de detalhes da OS.
    2.  Clicar no botão "Aprovar OS".
    3.  Confirmar a ação.
    4.  Verificar se o status da OS muda para `approved`.
*   **Resultado esperado:** OS aprovada.

#### 4.1.7 Finalizar Ordem de Serviço
*   **Objetivo:** Testar a finalização de uma Ordem de Serviço.
*   **Pré-requisitos:** Usuário logado com permissão de `admin` ou `owner`, OS em status `approved` ou `in_progress` com serviços/produtos adicionados.
*   **Passos:**
    1.  Navegar para a página de detalhes da OS.
    2.  Clicar no botão "Finalizar OS".
    3.  Confirmar a ação.
    4.  Verificar se o status da OS muda para `completed` e se uma conta a receber é gerada no Financeiro.
*   **Resultado esperado:** OS finalizada e conta a receber criada.

#### 4.1.8 Registrar Pagamento
*   **Objetivo:** Testar o registro de um pagamento para uma conta a receber.
*   **Pré-requisitos:** Usuário logado com permissão de `admin` ou `owner`, conta a receber pendente.
*   **Passos:**
    1.  Navegar para a página de Contas a Receber.
    2.  Selecionar uma conta a receber.
    3.  Clicar no botão "Registrar Pagamento".
    4.  Preencher valor e método de pagamento.
    5.  Clicar em "Confirmar Pagamento".
    6.  Verificar se o status da conta a receber muda para `paid` ou `partially_paid`.
*   **Resultado esperado:** Pagamento registrado e status da conta atualizado.

#### 4.1.9 Assinar Plano
*   **Objetivo:** Testar o fluxo de assinatura de um plano via Stripe Checkout.
*   **Pré-requisitos:** Usuário logado com permissão de `owner`, tenant sem assinatura ativa.
*   **Passos:**
    1.  Navegar para a página de Assinaturas.
    2.  Selecionar um plano e clicar em "Assinar".
    3.  Ser redirecionado para o Stripe Checkout.
    4.  Completar o pagamento no Stripe (usando dados de teste).
    5.  Ser redirecionado de volta para a aplicação.
    6.  Verificar se o status da assinatura é `active`.
*   **Resultado esperado:** Assinatura ativada e sincronizada com a Stripe.

## 5. Testes de Segurança

Os testes de segurança são cruciais para um SaaS multiempresa, garantindo a proteção dos dados e o controle de acesso.

### 5.1 Testes de Permissões

**Objetivo:** Validar que cada perfil de usuário (owner, admin, employee) tem acesso apenas às funcionalidades e dados permitidos.

**Pré-requisitos:** Usuários de teste com diferentes perfis e tenants.

**Passos:**
1.  Logar com um usuário de um perfil específico (ex: `employee`).
2.  Tentar acessar funcionalidades restritas a outros perfis (ex: configurações de assinatura, convidar usuários).
3.  Tentar realizar operações de escrita/leitura em dados que não pertencem ao seu perfil.

**Resultado esperado:** Acesso negado para funcionalidades e dados não autorizados, com mensagens de erro apropriadas (`403 Forbidden`).

**Critérios de aceite:**
*   Nenhum usuário pode realizar ações para as quais não tem permissão.
*   As mensagens de erro para acesso negado são claras.

### 5.2 Testes Multiempresa (Multi-tenant)

**Objetivo:** Garantir o isolamento completo dos dados entre diferentes tenants.

**Pré-requisitos:** Múltiplos tenants de teste com dados distintos.

**Passos:**
1.  Logar com um usuário do Tenant A.
2.  Tentar acessar dados (clientes, veículos, OS, etc.) que pertencem ao Tenant B.
3.  Tentar realizar operações de escrita/leitura que afetem dados do Tenant B.

**Resultado esperado:** Acesso e modificação de dados de outros tenants são estritamente negados.

**Critérios de aceite:**
*   Nenhum dado de um tenant é visível ou modificável por outro tenant.
*   As políticas de RLS estão funcionando corretamente.

### 5.3 Testes de Vulnerabilidade

**Objetivo:** Identificar e mitigar vulnerabilidades de segurança comuns.

**Pré-requisitos:** Ferramentas de análise de segurança (ex: OWASP ZAP, Snyk).

**Passos:**
1.  Realizar varreduras de segurança automatizadas.
2.  Testar manualmente cenários de injeção de SQL, XSS, CSRF, etc.
3.  Verificar a sanitização e validação de entrada de dados.

**Resultado esperado:** Nenhuma vulnerabilidade crítica encontrada. O sistema é resistente a ataques comuns.

**Critérios de aceite:**
*   Relatório de segurança limpo ou com vulnerabilidades de baixo risco tratadas.
*   Todas as entradas de usuário são validadas e sanitizadas.

## 6. Testes de Performance

**Objetivo:** Avaliar a velocidade, responsividade e estabilidade do sistema sob diferentes cargas de trabalho.

**Pré-requisitos:** Ferramentas de teste de carga (ex: k6, JMeter).

**Passos:**
1.  Simular um número crescente de usuários simultâneos.
2.  Monitorar o tempo de resposta da API e do frontend.
3.  Monitorar o uso de recursos do servidor (CPU, memória, banco de dados).

**Resultado esperado:** O sistema mantém tempos de resposta aceitáveis e estabilidade sob carga esperada.

**Critérios de aceite:**
*   Tempo de resposta da API < 200ms para 90% das requisições.
*   Tempo de carregamento da página < 2s.
*   O sistema escala horizontalmente conforme a demanda.

## 7. Critérios de Qualidade

Além dos testes específicos, os seguintes critérios gerais de qualidade serão aplicados:

*   **Confiabilidade:** O sistema opera sem falhas ou interrupções inesperadas.
*   **Usabilidade:** A interface do usuário é intuitiva e fácil de usar.
*   **Manutenibilidade:** O código é limpo, bem documentado e fácil de modificar.
*   **Escalabilidade:** A arquitetura suporta o crescimento futuro de usuários e dados.
*   **Consistência:** O comportamento do sistema é consistente em diferentes cenários e plataformas.

## Conclusão

Este documento estabelece uma estratégia de testes robusta para o OficinaPro, cobrindo desde testes unitários de baixo nível até testes E2E de alto nível, além de aspectos cruciais de segurança e performance. A implementação rigorosa desta estratégia garantirá a entrega de um produto de software de alta qualidade, confiável e seguro, que atenda às expectativas dos usuários e aos requisitos de negócio.
