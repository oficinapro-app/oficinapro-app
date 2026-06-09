# OficinaPro — Especificação Técnica do Frontend

**Autor:** Manus AI  
**Data:** 09 de junho de 2026  
**Versão:** 1.0  
**Finalidade:** Este documento detalha a especificação técnica do frontend para o sistema OficinaPro, um SaaS multiempresa para oficinas mecânicas. Ele serve como guia oficial para o desenvolvimento da interface do usuário, garantindo consistência, funcionalidade e uma experiência de usuário premium, inspirada em plataformas como Stripe, Linear, Vercel e Notion.

## 1. Estrutura do Projeto

A estrutura do projeto seguirá as melhores práticas para aplicações Next.js 15 com TypeScript, Tailwind CSS e shadcn/ui, promovendo modularidade, escalabilidade e fácil manutenção.

```
OficinaPro/
├── app/                      # Rotas e páginas da aplicação
│   ├── (auth)/               # Rotas de autenticação (login, cadastro, etc.)
│   │   ├── login/page.tsx
│   │   ├── register/page.tsx
│   │   └── ...
│   ├── (app)/                # Rotas principais da aplicação (protegidas)
│   │   ├── dashboard/page.tsx
│   │   ├── customers/        # Módulo de clientes
│   │   │   ├── page.tsx
│   │   │   ├── [id]/page.tsx
│   │   │   └── ...
│   │   ├── vehicles/
│   │   ├── work-orders/
│   │   ├── financial/
│   │   ├── settings/
│   │   └── ...
│   ├── api/                  # API Routes (para Server Actions/Route Handlers)
│   │   ├── auth/
│   │   ├── webhooks/
│   │   └── ...
│   ├── layout.tsx            # Layouts globais e aninhados
│   ├── globals.css           # Estilos globais
│   └── ...
├── components/               # Componentes React reutilizáveis
│   ├── ui/                   # Componentes shadcn/ui (gerados via CLI)
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   └── ...
│   ├── common/               # Componentes genéricos da aplicação
│   │   ├── Header.tsx
│   │   ├── Sidebar.tsx
│   │   ├── UserMenu.tsx
│   │   ├── DataTable.tsx
│   │   └── ...
│   ├── modules/              # Componentes específicos de módulos (ex: CustomerForm)
│   │   ├── customers/
│   │   ├── work-orders/
│   │   └── ...
│   └── providers/            # Context Providers (ex: AuthProvider, ThemeProvider)
├── hooks/                    # Custom React Hooks
│   ├── useAuth.ts
│   ├── useTenant.ts
│   ├── useFormValidation.ts
│   └── ...
├── lib/                      # Funções utilitárias e de configuração
│   ├── utils.ts              # Funções utilitárias gerais (ex: cn do shadcn/ui)
│   ├── supabase.ts           # Cliente Supabase
│   ├── stripe.ts             # Cliente Stripe
│   ├── constants.ts          # Constantes da aplicação
│   └── ...
├── services/                 # Funções para interação com APIs/Supabase
│   ├── auth.ts
│   ├── customers.ts
│   ├── workOrders.ts
│   └── ...
├── types/                    # Definições de tipos TypeScript
│   ├── next-auth.d.ts
│   ├── database.ts           # Tipos gerados do Supabase (supabase gen types ts)
│   ├── ui.ts
│   └── ...
├── validations/              # Schemas de validação (ex: Zod)
│   ├── auth.ts
│   ├── customer.ts
│   ├── workOrder.ts
│   └── ...
├── public/                   # Arquivos estáticos (imagens, fontes)
├── styles/                   # Estilos Tailwind CSS customizados
├── .env                      # Variáveis de ambiente
├── tsconfig.json             # Configuração TypeScript
├── tailwind.config.ts        # Configuração Tailwind CSS
├── next.config.js            # Configuração Next.js
└── package.json              # Dependências e scripts
```

## 2. Componentes Reutilizáveis

Os componentes serão construídos utilizando `shadcn/ui` como base, estendendo-os e criando novos componentes compostos para atender às necessidades específicas do OficinaPro. O foco é na reusabilidade, acessibilidade e conformidade com o design system.

### 2.1 `DataTable`

**Objetivo:** Exibir dados tabulares de forma paginada, filtrável e ordenável. Essencial para listagens de Clientes, Veículos, Ordens de Serviço, Produtos, etc.

*   **Base:** `shadcn/ui` Table.
*   **Funcionalidades:**
    *   Paginação (com controle de número de itens por página).
    *   Ordenação por coluna.
    *   Filtro global por texto.
    *   Filtros por coluna (dropdown, date range, etc.).
    *   Seleção de linhas (checkbox).
    *   Ações em massa (ex: excluir selecionados).
    *   Estados de loading (esqueleto).
    *   Empty state (mensagem quando não há dados).
*   **Propriedades:** `data`, `columns`, `onRowClick`, `onSelectionChange`, `isLoading`, `emptyStateContent`.

### 2.2 `SearchBar`

**Objetivo:** Componente de entrada de texto para realizar buscas em listagens ou em todo o sistema.

*   **Base:** `shadcn/ui` Input.
*   **Funcionalidades:**
    *   Ícone de lupa.
    *   Clear button.
    *   Debounce para evitar requisições excessivas.
*   **Propriedades:** `placeholder`, `onSearch`, `value`.

### 2.3 `FormField`

**Objetivo:** Componente wrapper para campos de formulário, padronizando rótulos, mensagens de erro e validações.

*   **Base:** `shadcn/ui` Form.
*   **Funcionalidades:**
    *   Exibição de `label`.
    *   Exibição de `hint` text.
    *   Exibição de mensagens de erro de validação.
    *   Integração com bibliotecas de validação (ex: Zod, React Hook Form).
*   **Propriedades:** `label`, `name`, `control`, `render` (para o componente de input), `errorMessage`.

### 2.4 `StatusBadge`

**Objetivo:** Exibir o status de uma entidade (OS, Cliente, Assinatura) de forma visualmente distinta.

*   **Base:** `shadcn/ui` Badge.
*   **Funcionalidades:**
    *   Cores e estilos variados para diferentes status (ex: `success` para 'Ativo', `warning` para 'Pendente', `destructive` para 'Cancelado').
    *   Texto customizável.
*   **Propriedades:** `status` (enum), `text`.

### 2.5 `ConfirmDialog`

**Objetivo:** Modal de confirmação para ações destrutivas ou importantes.

*   **Base:** `shadcn/ui` AlertDialog.
*   **Funcionalidades:**
    *   Título e descrição customizáveis.
    *   Botões de 
confirmação e cancelamento.
    *   Callback para ação de confirmação.
*   **Propriedades:** `title`, `description`, `onConfirm`, `open`, `onOpenChange`.

### 2.6 `EmptyState`

**Objetivo:** Exibir uma mensagem amigável e uma ilustração quando não há dados para exibir em uma lista ou seção.

*   **Base:** Componente customizado.
*   **Funcionalidades:**
    *   Título e descrição.
    *   Ícone ou ilustração relevante.
    *   Botão de ação (opcional, ex: "Adicionar Cliente").
*   **Propriedades:** `title`, `description`, `icon`, `actionButton`.

### 2.7 `KPICard`

**Objetivo:** Exibir Key Performance Indicators (KPIs) de forma clara e concisa no Dashboard.

*   **Base:** `shadcn/ui` Card.
*   **Funcionalidades:**
    *   Título do KPI.
    *   Valor principal.
    *   Variação percentual ou numérica (opcional).
    *   Ícone ou indicador de tendência.
*   **Propriedades:** `title`, `value`, `change`, `trend`, `icon`.

### 2.8 `MetricCard`

**Objetivo:** Similar ao `KPICard`, mas para métricas mais detalhadas ou com gráficos em miniatura.

*   **Base:** `shadcn/ui` Card.
*   **Funcionalidades:**
    *   Título da métrica.
    *   Valor principal.
    *   Subtítulo ou período.
    *   Mini-gráfico (sparkline) para visualização de tendência.
*   **Propriedades:** `title`, `value`, `subtitle`, `chartData`.

### 2.9 `Charts`

**Objetivo:** Componentes para exibir diferentes tipos de gráficos (barras, linhas, pizza) para visualização de dados financeiros e operacionais.

*   **Base:** Biblioteca de gráficos (ex: Recharts, Chart.js com React-Chartjs-2).
*   **Funcionalidades:**
    *   Gráficos de Linha (faturamento mensal, despesas).
    *   Gráficos de Barra (serviços mais populares, produtos mais vendidos).
    *   Gráficos de Pizza/Rosca (distribuição de despesas por categoria).
    *   Tooltips interativos.
    *   Responsividade.
*   **Propriedades:** `type`, `data`, `options`.

### 2.10 `Sidebar`

**Objetivo:** Navegação principal da aplicação, recolhível e responsiva.

*   **Base:** Componente customizado.
*   **Funcionalidades:**
    *   Links de navegação para os módulos principais.
    *   Ícones associados a cada link.
    *   Estado recolhido/expandido.
    *   Indicador de item ativo.
    *   Responsividade (ocultar em mobile, aparecer como drawer).
*   **Propriedades:** `isOpen`, `onToggle`.

### 2.11 `Header`

**Objetivo:** Topbar da aplicação, contendo logo, título da página, menu do usuário e notificações.

*   **Base:** Componente customizado.
*   **Funcionalidades:**
    *   Logo do OficinaPro.
    *   Título da página atual (Breadcrumbs).
    *   Botão para recolher/expandir Sidebar.
    *   Menu do usuário (com avatar, nome, opções de perfil/logout).
    *   Ícone de notificações (com badge de novas notificações).
    *   Botão de tema (claro/escuro).
*   **Propriedades:** `pageTitle`, `user`, `notifications`.

### 2.12 `UserMenu`

**Objetivo:** Menu dropdown para o usuário autenticado, acessível pelo Header.

*   **Base:** `shadcn/ui` DropdownMenu.
*   **Funcionalidades:**
    *   Exibir nome e e-mail do usuário.
    *   Opções de navegação (Meu Perfil, Configurações da Empresa, Gerenciar Usuários).
    *   Opção de Logout.
*   **Propriedades:** `user`.

## 3. Páginas do Sistema

Esta seção descreve cada página da aplicação, incluindo seu objetivo, rota, perfis de acesso, componentes, campos, botões, validações, responsividade e integrações.

### 3.1 Autenticação

As páginas de autenticação são o ponto de entrada do sistema, projetadas para serem seguras, intuitivas e com uma experiência de usuário limpa, inspirada em Stripe/Vercel.

#### 3.1.1 Login

**Objetivo:** Permitir que usuários existentes acessem o sistema.

**Rota:** `/login`

**Perfis com acesso:** Todos os usuários (não autenticados).

**Componentes utilizados:**
*   `FormField` (para e-mail e senha)
*   `Button` (para login e links)
*   `Input` (para e-mail e senha)
*   `Link` (para recuperação de senha e cadastro)
*   `Card` (para agrupar o formulário)

**Campos:**
*   **E-mail:** `type=
'email'`, `placeholder=

`E-mail`
*   **Senha:** `type='password'`, `placeholder='Senha'`
*   **Lembrar-me:** `type='checkbox'` (opcional)

**Botões:**
*   **Entrar:** Botão primário para submeter o formulário.
*   **Esqueceu a senha?** Link para a página de recuperação de senha.
*   **Criar conta:** Link para a página de cadastro.

**Estados de loading:**
*   Botão "Entrar" desabilitado e com spinner durante a submissão.

**Empty states:** Não aplicável.

**Mensagens de erro:**
*   "E-mail ou senha inválidos."
*   "Por favor, preencha todos os campos."
*   "Ocorreu um erro inesperado. Tente novamente."

**Validações:**
*   E-mail: Formato válido e campo obrigatório.
*   Senha: Campo obrigatório.

**Responsividade:** Layout centralizado e responsivo para mobile, tablet e desktop.

**Fluxo do usuário:**
1.  Usuário acessa `/login`.
2.  Preenche e-mail e senha.
3.  Clica em "Entrar".
4.  Em caso de sucesso, é redirecionado para o Dashboard (`/dashboard`).
5.  Em caso de falha, exibe mensagem de erro.

**Permissões:** Acesso público.

**Integrações com Supabase:**
*   Utiliza `supabase.auth.signInWithPassword()` para autenticação.

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Usuário consegue fazer login com credenciais válidas.
*   Mensagens de erro claras para credenciais inválidas ou campos vazios.
*   Links para recuperação de senha e cadastro funcionam.
*   Feedback visual de loading durante a submissão.

#### 3.1.2 Cadastro

**Objetivo:** Permitir que novos usuários criem uma conta e registrem sua oficina.

**Rota:** `/register`

**Perfis com acesso:** Todos os usuários (não autenticados).

**Componentes utilizados:**
*   `FormField` (para e-mail, senha, nome completo, nome da oficina)
*   `Button` (para cadastro e login)
*   `Input` (para campos de texto e senha)
*   `Link` (para login)
*   `Card` (para agrupar o formulário)

**Campos:**
*   **Nome Completo:** `type='text'`, `placeholder='Seu nome completo'`
*   **E-mail:** `type='email'`, `placeholder='Seu e-mail'`
*   **Senha:** `type='password'`, `placeholder='Crie uma senha'`
*   **Confirmar Senha:** `type='password'`, `placeholder='Confirme sua senha'`
*   **Nome da Oficina:** `type='text'`, `placeholder='Nome da sua oficina'`

**Botões:**
*   **Criar Conta:** Botão primário para submeter o formulário.
*   **Já tem uma conta? Entrar:** Link para a página de login.

**Estados de loading:**
*   Botão "Criar Conta" desabilitado e com spinner durante a submissão.

**Empty states:** Não aplicável.

**Mensagens de erro:**
*   "E-mail já cadastrado."
*   "As senhas não coincidem."
*   "A senha deve ter no mínimo 6 caracteres."
*   "Por favor, preencha todos os campos."
*   "Ocorreu um erro inesperado. Tente novamente."

**Validações:**
*   Nome Completo: Obrigatório.
*   E-mail: Formato válido e campo obrigatório.
*   Senha: Mínimo de 6 caracteres, obrigatório.
*   Confirmar Senha: Deve ser igual à senha.
*   Nome da Oficina: Obrigatório.

**Responsividade:** Layout centralizado e responsivo para mobile, tablet e desktop.

**Fluxo do usuário:**
1.  Usuário acessa `/register`.
2.  Preenche os dados de cadastro.
3.  Clica em "Criar Conta".
4.  Em caso de sucesso, é redirecionado para uma página de confirmação de e-mail ou diretamente para o Dashboard (dependendo da configuração de confirmação de e-mail do Supabase).
5.  Em caso de falha, exibe mensagem de erro.

**Permissões:** Acesso público.

**Integrações com Supabase:**
*   Utiliza `supabase.auth.signUp()` para criar o usuário.
*   Cria um novo `tenant` e um `tenant_user` com `role='owner'` e `status='active'` para o usuário recém-criado (via trigger de banco de dados ou Server Action).

**Integrações com Stripe:** Nenhuma (a assinatura é iniciada após o trial, em outra etapa).

**Critérios de aceite:**
*   Usuário consegue criar uma nova conta e registrar uma oficina.
*   Mensagens de erro claras para dados inválidos ou já existentes.
*   Links para login funcionam.
*   Feedback visual de loading durante a submissão.
*   Novo tenant e owner são criados no banco de dados.

#### 3.1.3 Recuperação de Senha

**Objetivo:** Permitir que usuários redefinam suas senhas caso as tenham esquecido.

**Rota:** `/forgot-password`

**Perfis com acesso:** Todos os usuários (não autenticados).

**Componentes utilizados:**
*   `FormField` (para e-mail)
*   `Button` (para enviar e-mail)
*   `Input` (para e-mail)
*   `Link` (para login)
*   `Card` (para agrupar o formulário)

**Campos:**
*   **E-mail:** `type='email'`, `placeholder='Seu e-mail cadastrado'`

**Botões:**
*   **Enviar link de recuperação:** Botão primário para submeter o formulário.
*   **Voltar para o login:** Link para a página de login.

**Estados de loading:**
*   Botão "Enviar link de recuperação" desabilitado e com spinner durante a submissão.

**Empty states:** Não aplicável.

**Mensagens de erro:**
*   "Por favor, insira um e-mail válido."
*   "Ocorreu um erro ao enviar o e-mail. Tente novamente."

**Validações:**
*   E-mail: Formato válido e campo obrigatório.

**Responsividade:** Layout centralizado e responsivo para mobile, tablet e desktop.

**Fluxo do usuário:**
1.  Usuário acessa `/forgot-password`.
2.  Insere seu e-mail cadastrado.
3.  Clica em "Enviar link de recuperação".
4.  Em caso de sucesso, exibe uma mensagem informando que um e-mail foi enviado e redireciona para a página de login.
5.  Em caso de falha, exibe mensagem de erro.

**Permissões:** Acesso público.

**Integrações com Supabase:**
*   Utiliza `supabase.auth.resetPasswordForEmail()` para enviar o e-mail de recuperação.

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Usuário consegue solicitar um link de recuperação de senha.
*   Mensagem de sucesso clara após o envio do e-mail.
*   Mensagens de erro para e-mail inválido ou falha no envio.
*   Feedback visual de loading durante a submissão.

#### 3.1.4 Alteração de Senha

**Objetivo:** Permitir que usuários redefinam suas senhas após clicarem no link de recuperação enviado por e-mail.

**Rota:** `/update-password` (ou similar, dependendo da configuração do Supabase)

**Perfis com acesso:** Usuários com token de recuperação de senha válido (geralmente autenticados temporariamente pelo Supabase).

**Componentes utilizados:**
*   `FormField` (para nova senha e confirmação)
*   `Button` (para alterar senha)
*   `Input` (para senhas)
*   `Card` (para agrupar o formulário)

**Campos:**
*   **Nova Senha:** `type='password'`, `placeholder='Nova senha'`
*   **Confirmar Nova Senha:** `type='password'`, `placeholder='Confirme a nova senha'`

**Botões:**
*   **Alterar Senha:** Botão primário para submeter o formulário.

**Estados de loading:**
*   Botão "Alterar Senha" desabilitado e com spinner durante a submissão.

**Empty states:** Não aplicável.

**Mensagens de erro:**
*   "As senhas não coincidem."
*   "A senha deve ter no mínimo 6 caracteres."
*   "Ocorreu um erro ao alterar a senha. Tente novamente."

**Validações:**
*   Nova Senha: Mínimo de 6 caracteres, obrigatório.
*   Confirmar Nova Senha: Deve ser igual à nova senha.

**Responsividade:** Layout centralizado e responsivo para mobile, tablet e desktop.

**Fluxo do usuário:**
1.  Usuário clica no link de recuperação de senha recebido por e-mail.
2.  É redirecionado para `/update-password` (ou similar).
3.  Define e confirma a nova senha.
4.  Clica em "Alterar Senha".
5.  Em caso de sucesso, exibe uma mensagem de sucesso e redireciona para a página de login.
6.  Em caso de falha, exibe mensagem de erro.

**Permissões:** Acesso restrito a usuários com token de recuperação válido.

**Integrações com Supabase:**
*   Utiliza `supabase.auth.updateUser()` para atualizar a senha do usuário.

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Usuário consegue redefinir a senha com sucesso.
*   Mensagens de erro claras para senhas inválidas ou que não coincidem.
*   Feedback visual de loading durante a submissão.

### 3.2 Dashboard

**Objetivo:** Fornecer uma visão geral e rápida do desempenho da oficina, com KPIs, gráficos e atalhos para as ações mais comuns. Deve ser um dashboard premium, limpo e intuitivo.

**Rota:** `/dashboard`

**Perfis com acesso:** `owner`, `admin`, `employee` (com dados filtrados por permissão).

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `KPICard`
*   `MetricCard`
*   `Charts` (para gráficos de linha, barra, rosca)
*   `Card` (para agrupar seções)
*   `Button` (para atalhos)
*   `EmptyState` (para dados não disponíveis)
*   `Skeleton` (para estados de loading)

**Campos:**
*   **KPIs:** Faturamento Total, OS Abertas, Contas a Receber (vencidas/a vencer), Contas a Pagar (vencidas/a vencer), Lucratividade.
*   **Gráficos:** Faturamento Mensal (linha), Despesas por Categoria (rosca), Serviços Mais Realizados (barra), Produtos Mais Vendidos (barra).
*   **Atalhos Rápidos:** Botões para "Nova OS", "Novo Cliente", "Novo Veículo", "Registrar Pagamento".

**Botões:**
*   Botões de atalho rápido.
*   Botões de filtro de período para KPIs e gráficos (ex: Últimos 7 dias, Mês atual, Ano atual).

**Estados de loading:**
*   `Skeleton` para cada `KPICard`, `MetricCard` e `Charts` enquanto os dados são carregados.

**Empty states:**
*   "Nenhum dado de faturamento disponível para o período selecionado." (para gráficos)
*   "Nenhuma OS aberta no momento." (para KPI de OS Abertas)

**Mensagens de erro:**
*   "Não foi possível carregar os dados do dashboard. Tente novamente."

**Validações:** Não aplicável diretamente na exibição, mas os dados vêm de validações de backend.

**Responsividade:**
*   **Desktop:** Layout de 2 ou 3 colunas, com sidebar expandida.
*   **Tablet:** Layout de 1 ou 2 colunas, sidebar recolhida ou como drawer.
*   **Celular:** Layout de 1 coluna, sidebar como drawer, cards empilhados.

**Fluxo do usuário:**
1.  Usuário faz login ou acessa `/dashboard`.
2.  Visualiza um resumo financeiro e operacional da oficina.
3.  Pode clicar em atalhos para iniciar novas ações.
4.  Pode filtrar o período dos dados exibidos.

**Permissões:**
*   `owner`, `admin`: Acesso total aos dados do dashboard.
*   `employee`: Acesso limitado a KPIs operacionais (OS abertas, etc.), sem acesso a dados financeiros sensíveis.

**Integrações com Supabase:**
*   Busca dados agregados de `work_orders`, `accounts_receivable`, `accounts_payable`, `financial_transactions` (via Server Actions ou RPCs).

**Integrações com Stripe:** Nenhuma direta, mas os dados financeiros podem ser influenciados por transações Stripe.

**Critérios de aceite:**
*   Dashboard carrega rapidamente e exibe dados relevantes.
*   KPIs e gráficos são claros e fáceis de entender.
*   Atalhos rápidos funcionam e direcionam para as páginas corretas.
*   Responsividade adequada em diferentes tamanhos de tela.
*   Permissões de acesso aos dados são respeitadas.

### 3.3 Clientes

**Objetivo:** Gerenciar o cadastro de clientes da oficina, incluindo suas informações pessoais, veículos associados e histórico de serviços.

**Rota:** `/customers` (listagem), `/customers/new` (cadastro), `/customers/[id]` (detalhes/edição)

**Perfis com acesso:** `owner`, `admin`, `employee`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem)
*   `SearchBar`
*   `Button` (para adicionar, editar, excluir)
*   `FormField` (para formulários de cadastro/edição)
*   `Input`, `Select`, `DatePicker` (para campos de formulário)
*   `Modal` ou `Drawer` (para formulários de cadastro/edição)
*   `ConfirmDialog` (para exclusão)
*   `EmptyState`
*   `Skeleton`
*   `Tabs` (para detalhes do cliente: Dados, Veículos, Histórico)

**Campos (Listagem):**
*   Nome do Cliente
*   Documento (CPF/CNPJ)
*   Telefone
*   E-mail
*   Status

**Campos (Cadastro/Edição):**
*   **Tipo de Cliente:** `Select` (Pessoa Física, Pessoa Jurídica)
*   **Nome/Razão Social:** `Input`
*   **CPF/CNPJ:** `Input` (máscara)
*   **RG/Inscrição Estadual:** `Input`
*   **E-mail:** `Input`
*   **Telefone:** `Input` (máscara)
*   **WhatsApp:** `Input` (máscara)
*   **Data de Nascimento:** `DatePicker`
*   **CEP:** `Input` (busca automática de endereço)
*   **Endereço:** `Input` (Rua, Número, Complemento, Bairro, Cidade, Estado)
*   **Observações:** `Textarea`
*   **Status:** `Select` (Ativo, Inativo)

**Botões:**
*   **Novo Cliente:** Na página de listagem.
*   **Salvar / Cancelar:** No formulário de cadastro/edição.
*   **Editar / Excluir:** Na linha da tabela ou na página de detalhes.
*   **Adicionar Endereço / Adicionar Veículo:** Na página de detalhes do cliente.

**Estados de loading:**
*   `Skeleton` na `DataTable` durante o carregamento da lista.
*   Formulários desabilitados e com spinner durante a submissão.

**Empty states:**
*   "Nenhum cliente cadastrado ainda. Clique em 'Novo Cliente' para começar."
*   "Nenhum veículo associado a este cliente."

**Mensagens de erro:**
*   "Nome do cliente é obrigatório."
*   "CPF/CNPJ inválido."
*   "E-mail já cadastrado para outro cliente."
*   "Não foi possível salvar o cliente. Tente novamente."

**Validações:**
*   Todos os campos obrigatórios.
*   Formato de e-mail, telefone, documento.
*   Unicidade de documento por tenant.

**Responsividade:**
*   Listagem: Tabela com colunas ocultáveis em telas menores, ou layout de cards.
*   Formulários: Adaptáveis a telas menores, com campos empilhados.

**Fluxo do usuário:**
1.  Usuário acessa `/customers`.
2.  Visualiza a lista de clientes, pode buscar e filtrar.
3.  Clica em "Novo Cliente" para abrir um formulário (modal/drawer).
4.  Preenche os dados e salva.
5.  Clica em um cliente para ver detalhes, editar ou adicionar veículos/endereços.

**Permissões:**
*   `owner`, `admin`: CRUD completo de clientes.
*   `employee`: Visualizar, criar, editar clientes. Exclusão (soft delete) pode ser restrita a `owner`/`admin`.

**Integrações com Supabase:**
*   `customers` (CRUD)
*   `customer_addresses` (CRUD)
*   `vehicles` (listagem e associação)
*   `work_orders` (listagem de histórico)

**Integrações com Stripe:** Nenhuma direta.

**Critérios de aceite:**
*   Listagem de clientes funciona com busca, filtro e paginação.
*   Cadastro e edição de clientes funcionam com validações.
*   Exclusão de cliente (soft delete) funciona com confirmação.
*   Página de detalhes do cliente exibe informações completas e relacionadas.
*   Permissões de acesso são respeitadas.

### 3.4 Veículos

**Objetivo:** Gerenciar o cadastro de veículos, associá-los a clientes e visualizar seu histórico de ordens de serviço.

**Rota:** `/vehicles` (listagem), `/vehicles/new` (cadastro), `/vehicles/[id]` (detalhes/edição)

**Perfis com acesso:** `owner`, `admin`, `employee`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem)
*   `SearchBar`
*   `Button` (para adicionar, editar, excluir, abrir OS)
*   `FormField` (para formulários de cadastro/edição)
*   `Input`, `Select` (para campos de formulário)
*   `Modal` ou `Drawer` (para formulários de cadastro/edição)
*   `ConfirmDialog` (para exclusão)
*   `EmptyState`
*   `Skeleton`
*   `Tabs` (para detalhes do veículo: Dados, Histórico de OS)

**Campos (Listagem):**
*   Placa
*   Marca
*   Modelo
*   Ano
*   Cliente Proprietário

**Campos (Cadastro/Edição):**
*   **Cliente:** `Select` (busca de clientes existentes)
*   **Placa:** `Input`
*   **Marca:** `Input`
*   **Modelo:** `Input`
*   **Ano Fabricação:** `Input` (numérico)
*   **Ano Modelo:** `Input` (numérico)
*   **Cor:** `Input`
*   **RENAVAM:** `Input`
*   **Chassi:** `Input`
*   **Quilometragem Atual:** `Input` (numérico)
*   **Tipo de Combustível:** `Select` (Gasolina, Etanol, Flex, Diesel, Elétrico, Híbrido, Outro)
*   **Observações:** `Textarea`
*   **Status:** `Select` (Ativo, Inativo)

**Botões:**
*   **Novo Veículo:** Na página de listagem.
*   **Salvar / Cancelar:** No formulário de cadastro/edição.
*   **Editar / Excluir:** Na linha da tabela ou na página de detalhes.
*   **Abrir Nova OS:** Na página de detalhes do veículo.

**Estados de loading:**
*   `Skeleton` na `DataTable` durante o carregamento da lista.
*   Formulários desabilitados e com spinner durante a submissão.

**Empty states:**
*   "Nenhum veículo cadastrado ainda. Clique em 'Novo Veículo' para começar."
*   "Nenhuma Ordem de Serviço para este veículo."

**Mensagens de erro:**
*   "Placa é obrigatória."
*   "Placa já cadastrada para outro veículo ativo neste tenant."
*   "Não foi possível salvar o veículo. Tente novamente."

**Validações:**
*   Todos os campos obrigatórios.
*   Formato de placa, anos.
*   Unicidade de placa por tenant para veículos ativos.

**Responsividade:**
*   Listagem: Tabela com colunas ocultáveis em telas menores, ou layout de cards.
*   Formulários: Adaptáveis a telas menores, com campos empilhados.

**Fluxo do usuário:**
1.  Usuário acessa `/vehicles`.
2.  Visualiza a lista de veículos, pode buscar e filtrar.
3.  Clica em "Novo Veículo" para abrir um formulário (modal/drawer).
4.  Preenche os dados e salva.
5.  Clica em um veículo para ver detalhes, editar ou abrir uma nova OS.

**Permissões:**
*   `owner`, `admin`: CRUD completo de veículos.
*   `employee`: Visualizar, criar, editar veículos. Exclusão (soft delete) pode ser restrita a `owner`/`admin`.

**Integrações com Supabase:**
*   `vehicles` (CRUD)
*   `customers` (seleção de cliente)
*   `work_orders` (listagem de histórico e criação de nova OS)

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Listagem de veículos funciona com busca, filtro e paginação.
*   Cadastro e edição de veículos funcionam com validações.
*   Exclusão de veículo (soft delete) funciona com confirmação.
*   Página de detalhes do veículo exibe informações completas e relacionadas.
*   Botão "Abrir Nova OS" funciona e pré-preenche os dados do cliente/veículo.
*   Permissões de acesso são respeitadas.

### 3.5 Serviços

**Objetivo:** Gerenciar o catálogo de serviços oferecidos pela oficina.

**Rota:** `/services` (listagem), `/services/new` (cadastro), `/services/[id]` (detalhes/edição)

**Perfis com acesso:** `owner`, `admin`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem)
*   `SearchBar`
*   `Button` (para adicionar, editar, excluir)
*   `FormField` (para formulários de cadastro/edição)
*   `Input`, `Textarea`, `Checkbox` (para campos de formulário)
*   `Modal` ou `Drawer` (para formulários de cadastro/edição)
*   `ConfirmDialog` (para exclusão)
*   `EmptyState`
*   `Skeleton`

**Campos (Listagem):**
*   Nome do Serviço
*   Categoria
*   Preço Padrão
*   Ativo

**Campos (Cadastro/Edição):**
*   **Nome do Serviço:** `Input`
*   **Código Interno:** `Input` (opcional)
*   **Categoria:** `Input` ou `Select` (com sugestões)
*   **Descrição:** `Textarea`
*   **Preço Padrão:** `Input` (numérico, formato monetário)
*   **Tempo Estimado (minutos):** `Input` (numérico)
*   **Ativo:** `Checkbox`

**Botões:**
*   **Novo Serviço:** Na página de listagem.
*   **Salvar / Cancelar:** No formulário de cadastro/edição.
*   **Editar / Excluir:** Na linha da tabela.

**Estados de loading:**
*   `Skeleton` na `DataTable` durante o carregamento da lista.
*   Formulários desabilitados e com spinner durante a submissão.

**Empty states:**
*   "Nenhum serviço cadastrado ainda. Clique em 'Novo Serviço' para começar."

**Mensagens de erro:**
*   "Nome do serviço é obrigatório."
*   "Nome do serviço já cadastrado neste tenant."
*   "Preço padrão deve ser um valor numérico válido."
*   "Não foi possível salvar o serviço. Tente novamente."

**Validações:**
*   Todos os campos obrigatórios.
*   Formato numérico para preço e tempo.
*   Unicidade de nome de serviço por tenant para serviços ativos.

**Responsividade:**
*   Listagem: Tabela com colunas ocultáveis em telas menores.
*   Formulários: Adaptáveis a telas menores.

**Fluxo do usuário:**
1.  Usuário acessa `/services`.
2.  Visualiza a lista de serviços, pode buscar e filtrar.
3.  Clica em "Novo Serviço" para abrir um formulário (modal/drawer).
4.  Preenche os dados e salva.
5.  Clica em "Editar" ou "Excluir" na linha de um serviço.

**Permissões:**
*   `owner`, `admin`: CRUD completo de serviços.
*   `employee`: Apenas visualizar serviços (não pode criar, editar ou excluir).

**Integrações com Supabase:**
*   `services` (CRUD)

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Listagem de serviços funciona com busca, filtro e paginação.
*   Cadastro e edição de serviços funcionam com validações.
*   Exclusão de serviço (soft delete) funciona com confirmação.
*   Permissões de acesso são respeitadas.

### 3.6 Produtos

**Objetivo:** Gerenciar o catálogo de produtos e peças utilizados pela oficina, incluindo controle de estoque.

**Rota:** `/products` (listagem), `/products/new` (cadastro), `/products/[id]` (detalhes/edição)

**Perfis com acesso:** `owner`, `admin`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem)
*   `SearchBar`
*   `Button` (para adicionar, editar, excluir)
*   `FormField` (para formulários de cadastro/edição)
*   `Input`, `Textarea`, `Select`, `Checkbox` (para campos de formulário)
*   `Modal` ou `Drawer` (para formulários de cadastro/edição)
*   `ConfirmDialog` (para exclusão)
*   `EmptyState`
*   `Skeleton`

**Campos (Listagem):**
*   Nome do Produto
*   SKU
*   Categoria
*   Preço de Venda
*   Estoque Atual
*   Ativo

**Campos (Cadastro/Edição):**
*   **Nome do Produto:** `Input`
*   **SKU:** `Input` (opcional)
*   **Categoria:** `Input` ou `Select` (com sugestões)
*   **Descrição:** `Textarea`
*   **Unidade:** `Select` (un, litro, kg, par, etc.)
*   **Preço de Custo:** `Input` (numérico, formato monetário)
*   **Preço de Venda:** `Input` (numérico, formato monetário)
*   **Quantidade em Estoque:** `Input` (numérico, permite decimais)
*   **Estoque Mínimo:** `Input` (numérico, permite decimais)
*   **Ativo:** `Checkbox`

**Botões:**
*   **Novo Produto:** Na página de listagem.
*   **Salvar / Cancelar:** No formulário de cadastro/edição.
*   **Editar / Excluir:** Na linha da tabela.

**Estados de loading:**
*   `Skeleton` na `DataTable` durante o carregamento da lista.
*   Formulários desabilitados e com spinner durante a submissão.

**Empty states:**
*   "Nenhum produto cadastrado ainda. Clique em 'Novo Produto' para começar."

**Mensagens de erro:**
*   "Nome do produto é obrigatório."
*   "Nome do produto já cadastrado neste tenant."
*   "SKU já cadastrado neste tenant."
*   "Preço de venda deve ser um valor numérico válido."
*   "Não foi possível salvar o produto. Tente novamente."

**Validações:**
*   Todos os campos obrigatórios.
*   Formato numérico para preços e quantidades.
*   Unicidade de nome e SKU por tenant para produtos ativos.

**Responsividade:**
*   Listagem: Tabela com colunas ocultáveis em telas menores.
*   Formulários: Adaptáveis a telas menores.

**Fluxo do usuário:**
1.  Usuário acessa `/products`.
2.  Visualiza a lista de produtos, pode buscar e filtrar.
3.  Clica em "Novo Produto" para abrir um formulário (modal/drawer).
4.  Preenche os dados e salva.
5.  Clica em "Editar" ou "Excluir" na linha de um produto.

**Permissões:**
*   `owner`, `admin`: CRUD completo de produtos.
*   `employee`: Apenas visualizar produtos (não pode criar, editar ou excluir).

**Integrações com Supabase:**
*   `products` (CRUD)

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Listagem de produtos funciona com busca, filtro e paginação.
*   Cadastro e edição de produtos funcionam com validações.
*   Exclusão de produto (soft delete) funciona com confirmação.
*   Controle de estoque (quantidade atual e mínima) é exibido corretamente.
*   Permissões de acesso são respeitadas.

### 3.7 Ordens de Serviço

**Objetivo:** Gerenciar o ciclo de vida completo das ordens de serviço, desde a abertura até a finalização e entrega do veículo.

**Rota:** `/work-orders` (listagem), `/work-orders/new` (cadastro), `/work-orders/[id]` (detalhes/edição)

**Perfis com acesso:** `owner`, `admin`, `employee`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem)
*   `SearchBar`
*   `Button` (para adicionar, editar, excluir, mudar status)
*   `FormField` (para formulários)
*   `Input`, `Select`, `Textarea`, `DatePicker` (para campos de formulário)
*   `Modal` ou `Drawer` (para formulários)
*   `ConfirmDialog` (para exclusão, aprovação, finalização)
*   `EmptyState`
*   `Skeleton`
*   `Tabs` (para abas de detalhes da OS)
*   `StatusBadge`

**Campos (Listagem):**
*   Número da OS
*   Cliente
*   Veículo (Placa/Modelo)
*   Status
*   Data de Abertura
*   Valor Total

**Campos (Cadastro/Edição - Aba Dados Gerais):**
*   **Cliente:** `Select` (busca de clientes existentes)
*   **Veículo:** `Select` (busca de veículos do cliente selecionado)
*   **Relato do Cliente:** `Textarea`
*   **Quilometragem de Entrada:** `Input` (numérico)
*   **Previsão de Entrega:** `DatePicker`
*   **Técnico Responsável:** `Select` (usuários do tenant com role `employee`)
*   **Observações Internas:** `Textarea`

**Campos (Aba Diagnóstico):**
*   **Diagnóstico Técnico:** `Textarea`

**Campos (Aba Serviços):**
*   Listagem de `work_order_services` (DataTable interna)
*   Botão "Adicionar Serviço" (abre modal/drawer para selecionar serviço do catálogo ou adicionar avulso)
*   Campos para cada item de serviço: Descrição, Quantidade, Preço Unitário, Desconto, Total.

**Campos (Aba Produtos):**
*   Listagem de `work_order_products` (DataTable interna)
*   Botão "Adicionar Produto" (abre modal/drawer para selecionar produto do catálogo ou adicionar avulso)
*   Campos para cada item de produto: Descrição, Quantidade, Preço Unitário, Desconto, Total.

**Campos (Aba Financeiro):**
*   Resumo de totais (Serviços, Produtos, Desconto, Acréscimos, Total Geral).
*   Botão "Registrar Pagamento" (abre modal para registrar `financial_transaction` e atualizar `accounts_receivable`).
*   Listagem de pagamentos/contas a receber associados.

**Campos (Aba Histórico):**
*   Listagem de `work_order_history` (DataTable interna)
*   Exibe: Status Anterior, Novo Status, Usuário, Data/Hora, Observações.

**Botões:**
*   **Nova OS:** Na página de listagem.
*   **Salvar Rascunho / Salvar e Abrir:** No formulário de cadastro/edição.
*   **Mudar Status:** Botão principal na página de detalhes da OS (dropdown com opções: Diagnóstico, Aguardando Aprovação, Aprovada, Em Andamento, Concluída, Entregue, Cancelada, Reabrir).
*   **Aprovar OS:** Botão específico quando status é `waiting_approval`.
*   **Finalizar OS:** Botão específico quando status é `in_progress`.
*   **Registrar Pagamento:** Na aba Financeiro.
*   **Editar / Excluir:** Na página de detalhes (exclusão via soft delete).

**Estados de loading:**
*   `Skeleton` na `DataTable` e nas abas de detalhes.
*   Botões desabilitados e com spinner durante ações de mudança de status ou salvamento.

**Empty states:**
*   "Nenhuma Ordem de Serviço encontrada."
*   "Nenhum serviço adicionado a esta OS."
*   "Nenhum produto adicionado a esta OS."
*   "Nenhum histórico de status para esta OS."

**Mensagens de erro:**
*   "Cliente e Veículo são obrigatórios para abrir uma OS."
*   "Não é possível mudar para este status. Verifique o fluxo."
*   "Não foi possível salvar a Ordem de Serviço. Tente novamente."

**Validações:**
*   Campos obrigatórios.
*   Validação de fluxo de status (ex: não pode ir de `draft` para `completed` diretamente).
*   Validação de estoque ao adicionar produtos.

**Responsividade:**
*   Listagem: Tabela com colunas ocultáveis.
*   Página de detalhes: Abas adaptáveis, layout de 1 coluna em mobile.

**Fluxo do usuário:**
1.  Usuário acessa `/work-orders` e vê a lista.
2.  Clica em "Nova OS", seleciona cliente/veículo, preenche relato e salva como rascunho ou abre.
3.  Na página de detalhes, navega pelas abas (Dados, Diagnóstico, Serviços, Produtos, Financeiro, Histórico).
4.  Adiciona serviços e produtos, registra diagnóstico.
5.  Muda o status da OS conforme o andamento (ex: para `waiting_approval`).
6.  Cliente aprova, status muda para `approved`.
7.  Serviços são executados, status muda para `in_progress`.
8.  OS é finalizada, status muda para `completed`.
9.  Pagamento é registrado, veículo é entregue, status muda para `delivered`.

**Permissões:**
*   `owner`, `admin`: CRUD completo de OS, mudança de status, aprovação, finalização.
*   `employee`: Criar, visualizar, editar OS (exceto campos financeiros sensíveis), mudar status (com restrições).

**Integrações com Supabase:**
*   `work_orders` (CRUD)
*   `work_order_services` (CRUD)
*   `work_order_products` (CRUD)
*   `work_order_history` (INSERT via trigger)
*   `customers`, `vehicles`, `services`, `products`, `profiles` (para seleção e exibição de dados relacionados)
*   `accounts_receivable`, `financial_transactions` (para aba Financeiro)

**Integrações com Stripe:** Nenhuma direta.

**Critérios de aceite:**
*   Listagem de OS funciona com busca, filtro e paginação.
*   Criação e edição de OS funcionam com todas as abas e validações.
*   Fluxo de mudança de status é intuitivo e respeita as regras de negócio.
*   Cálculo de totais de serviços, produtos, descontos e total geral é preciso.
*   Histórico de status é registrado automaticamente.
*   Permissões de acesso são respeitadas.

### 3.8 Financeiro

**Objetivo:** Gerenciar as finanças da oficina, incluindo receitas, despesas, contas a receber e a pagar, e fluxo de caixa.

**Rota:** `/financial` (visão geral), `/financial/receivables` (contas a receber), `/financial/payables` (contas a pagar), `/financial/transactions` (fluxo de caixa)

**Perfis com acesso:** `owner`, `admin`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `KPICard`
*   `MetricCard`
*   `Charts` (para gráficos de linha, barra, rosca)
*   `Card` (para agrupar seções)
*   `DataTable` (para listagens de contas e transações)
*   `SearchBar`, `Select`, `DatePicker` (para filtros)
*   `Button` (para adicionar, editar, registrar pagamento/recebimento)
*   `Modal` ou `Drawer` (para formulários)
*   `ConfirmDialog` (para exclusão/cancelamento)
*   `EmptyState`
*   `Skeleton`
*   `Tabs` (para navegação entre sub-seções financeiras)

**Campos (Visão Geral):**
*   KPIs: Receita Total, Despesa Total, Lucro Líquido, Contas a Receber (vencidas/a vencer), Contas a Pagar (vencidas/a vencer).
*   Gráficos: Fluxo de Caixa Mensal (linha), Despesas por Categoria (rosca), Receitas por Origem (barra).

**Campos (Contas a Receber/Pagar - Listagem):**
*   Descrição
*   Cliente/Fornecedor
*   Valor
*   Data de Vencimento
*   Status
*   Data de Pagamento

**Campos (Contas a Receber - Cadastro/Edição):**
*   **Cliente:** `Select`
*   **Descrição:** `Input`
*   **Valor:** `Input` (monetário)
*   **Data de Vencimento:** `DatePicker`
*   **Status:** `Select` (Pendente, Pago, Vencido, Cancelado)
*   **Método de Pagamento:** `Select` (se status for Pago)

**Campos (Contas a Pagar - Cadastro/Edição):**
*   **Fornecedor:** `Input`
*   **Categoria:** `Input` ou `Select`
*   **Descrição:** `Input`
*   **Valor:** `Input` (monetário)
*   **Data de Vencimento:** `DatePicker`
*   **Status:** `Select` (Pendente, Pago, Vencido, Cancelado)
*   **Método de Pagamento:** `Select` (se status for Pago)

**Campos (Fluxo de Caixa - Listagem):**
*   Tipo (Receita/Despesa)
*   Descrição
*   Valor
*   Data da Transação
*   Método de Pagamento
*   Origem (OS, Manual, Assinatura)

**Botões:**
*   **Adicionar Receita / Adicionar Despesa:** Na visão geral ou nas respectivas listagens.
*   **Registrar Pagamento / Registrar Recebimento:** Na listagem de contas.
*   **Editar / Excluir / Cancelar:** Na linha da tabela.
*   Botões de filtro de período.

**Estados de loading:**
*   `Skeleton` para KPIs, gráficos e DataTables.
*   Formulários desabilitados e com spinner.

**Empty states:**
*   "Nenhuma conta a receber encontrada."
*   "Nenhuma transação financeira registrada."

**Mensagens de erro:**
*   "Valor é obrigatório e deve ser numérico."
*   "Data de vencimento é obrigatória."
*   "Não foi possível salvar a conta. Tente novamente."

**Validações:**
*   Campos obrigatórios.
*   Formato numérico para valores.
*   Datas válidas.

**Responsividade:**
*   Layout de cards e tabelas adaptáveis.
*   Gráficos responsivos.

**Fluxo do usuário:**
1.  Usuário acessa `/financial` e vê o resumo.
2.  Navega para Contas a Receber, Contas a Pagar ou Fluxo de Caixa.
3.  Adiciona novas contas/transações, registra pagamentos, edita ou cancela.

**Permissões:**
*   `owner`, `admin`: Acesso total a todas as funcionalidades financeiras.
*   `employee`: Nenhuma permissão para esta seção.

**Integrações com Supabase:**
*   `accounts_receivable` (CRUD)
*   `accounts_payable` (CRUD)
*   `financial_transactions` (CRUD)
*   `work_orders`, `customers` (para relacionamentos)

**Integrações com Stripe:**
*   Sincronização de pagamentos de assinaturas (via webhooks, que podem gerar `financial_transactions` e `invoices`).

**Critérios de aceite:**
*   Visão geral financeira clara e atualizada.
*   Listagens de contas e transações funcionam com filtros.
*   Cadastro, edição e registro de pagamentos/recebimentos funcionam.
*   Cálculos financeiros são precisos.
*   Permissões de acesso são respeitadas.

### 3.9 Assinaturas

**Objetivo:** Permitir que o `owner` da oficina gerencie sua assinatura com o OficinaPro, incluindo visualização do plano atual, upgrade, downgrade e cancelamento.

**Rota:** `/subscriptions`

**Perfis com acesso:** `owner`, `admin` (apenas visualização).

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `Card` (para exibir detalhes do plano atual e opções)
*   `Button` (para upgrade, downgrade, cancelar, ir para portal Stripe)
*   `Modal` ou `Drawer` (para confirmação de ações)
*   `EmptyState`
*   `Skeleton`
*   `StatusBadge`

**Campos:**
*   **Plano Atual:** Nome do plano, descrição, preço, intervalo de cobrança.
*   **Status da Assinatura:** `StatusBadge` (Ativa, Trial, Vencida, Cancelada).
*   **Próxima Cobrança:** Data e valor.
*   **Recursos do Plano:** Lista de funcionalidades/limites.

**Botões:**
*   **Mudar Plano:** Abre uma lista de planos disponíveis para upgrade/downgrade.
*   **Cancelar Assinatura:** Abre um `ConfirmDialog`.
*   **Gerenciar Pagamento (Portal Stripe):** Redireciona para o portal de clientes da Stripe.

**Estados de loading:**
*   `Skeleton` para os detalhes da assinatura.
*   Botões desabilitados e com spinner durante ações de mudança/cancelamento.

**Empty states:**
*   "Nenhuma assinatura ativa encontrada. Inicie seu período de trial!"

**Mensagens de erro:**
*   "Não foi possível carregar os detalhes da assinatura. Tente novamente."
*   "Ocorreu um erro ao tentar mudar o plano. Tente novamente."
*   "Não foi possível cancelar a assinatura. Entre em contato com o suporte."

**Validações:**
*   Ações de upgrade/downgrade/cancelamento devem ser validadas pelo backend e Stripe.

**Responsividade:** Layout de cards adaptável.

**Fluxo do usuário:**
1.  `owner` acessa `/subscriptions`.
2.  Visualiza o plano atual e seu status.
3.  Pode optar por mudar de plano, cancelar ou gerenciar pagamentos via Stripe.

**Permissões:**
*   `owner`: Acesso total (visualizar, mudar plano, cancelar).
*   `admin`: Apenas visualizar o status da assinatura.
*   `employee`: Nenhuma permissão para esta seção.

**Integrações com Supabase:**
*   `subscriptions` (leitura e atualização via backend/webhooks)
*   `subscription_plans` (leitura)
*   `tenants` (atualização do status do tenant)

**Integrações com Stripe:**
*   API da Stripe para listar planos, criar/atualizar/cancelar assinaturas.
*   Redirecionamento para o Stripe Customer Portal.

**Critérios de aceite:**
*   Exibição clara do plano atual e status da assinatura.
*   Funcionalidades de mudança de plano e cancelamento funcionam corretamente.
*   Redirecionamento para o portal Stripe funciona.
*   Mensagens de feedback claras para todas as ações.
*   Permissões de acesso são respeitadas.

### 3.10 Configurações

**Objetivo:** Permitir que `owner` e `admin` configurem aspectos da oficina, gerenciem usuários e definam preferências pessoais.

**Rota:** `/settings` (visão geral), `/settings/profile` (perfil), `/settings/company` (empresa), `/settings/users` (usuários), `/settings/preferences` (preferências)

**Perfis com acesso:** `owner`, `admin`, `employee` (apenas perfil).

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `Tabs` (para navegação entre sub-seções de configurações)
*   `FormField`
*   `Input`, `Select`, `Textarea`, `Checkbox`, `FileUpload` (para logo)
*   `Button` (para salvar, convidar usuário, remover usuário)
*   `DataTable` (para listagem de usuários)
*   `ConfirmDialog` (para remover usuário)
*   `EmptyState`
*   `Skeleton`

**Campos (Perfil - `/settings/profile`):**
*   **Nome Completo:** `Input`
*   **E-mail:** `Input` (somente leitura, via Supabase Auth)
*   **Telefone:** `Input`
*   **URL do Avatar:** `FileUpload` ou `Input`
*   **Senha:** Botão "Alterar Senha" (redireciona para `/forgot-password` ou modal)

**Campos (Empresa - `/settings/company`):**
*   **Razão Social:** `Input`
*   **Nome Fantasia:** `Input`
*   **CNPJ/CPF:** `Input`
*   **Inscrição Estadual:** `Input`
*   **E-mail:** `Input`
*   **Telefone:** `Input`
*   **WhatsApp:** `Input`
*   **Logo:** `FileUpload` (para upload no Supabase Storage)
*   **Endereço:** Campos de `Input` para CEP, Rua, Número, Complemento, Bairro, Cidade, Estado, País.
*   **Configurações Financeiras:** Moeda padrão, Taxa de imposto padrão.
*   **Configurações Operacionais:** Prefixo de OS, Próximo número de OS (somente leitura).

**Campos (Usuários - `/settings/users`):**
*   Listagem de `tenant_users` (DataTable)
*   Botão "Convidar Usuário" (abre modal com campo de e-mail e seleção de `role`)
*   Campos para cada usuário: Nome, E-mail, Perfil (`role`), Status (`membership_status`), Ações (Editar Perfil, Remover).

**Campos (Preferências - `/settings/preferences`):**
*   **Tema:** `Select` (Sistema, Claro, Escuro)
*   **Idioma:** `Select` (Português, Inglês)
*   **Notificações por E-mail:** `Checkbox`
*   **Notificações no Aplicativo:** `Checkbox`

**Botões:**
*   **Salvar Alterações:** Em cada sub-seção de configurações.
*   **Convidar Usuário:** Na sub-seção de Usuários.
*   **Editar Perfil / Remover:** Na linha da tabela de usuários.
*   **Alterar Senha:** Na sub-seção de Perfil.

**Estados de loading:**
*   `Skeleton` para formulários e DataTables.
*   Botões desabilitados e com spinner durante submissão.

**Empty states:**
*   "Nenhum usuário convidado ainda."

**Mensagens de erro:**
*   "E-mail inválido para convite."
*   "Não foi possível salvar as configurações. Tente novamente."
*   "Não foi possível convidar o usuário. Tente novamente."

**Validações:**
*   Campos obrigatórios e formatos válidos.
*   Validação de e-mail para convite.

**Responsividade:**
*   Abas adaptáveis, formulários empilhados em mobile.
*   Tabela de usuários com colunas ocultáveis.

**Fluxo do usuário:**
1.  Usuário acessa `/settings`.
2.  Navega entre as abas para configurar perfil, empresa, usuários ou preferências.
3.  `owner`/`admin` convida novos usuários ou gerencia os existentes.

**Permissões:**
*   `owner`: Acesso total a todas as configurações.
*   `admin`: Acesso a configurações da empresa e usuários (exceto exclusão de owner), mas não pode alterar plano de assinatura.
*   `employee`: Apenas acesso à sub-seseção de Perfil (`/settings/profile`).

**Integrações com Supabase:**
*   `profiles` (CRUD para perfil)
*   `tenants` (CRUD para configurações da empresa)
*   `tenant_users` (CRUD para usuários do tenant)
*   `settings` (CRUD para preferências e outras configurações)
*   Supabase Storage (para upload de logo)
*   Supabase Auth (para convite de usuários)

**Integrações com Stripe:** Nenhuma direta.

**Critérios de aceite:**
*   Todas as sub-seções de configurações funcionam.
*   Salvar alterações funciona com validações e feedback.
*   Convite e gerenciamento de usuários funcionam.
*   Upload de logo funciona.
*   Permissões de acesso são respeitadas.

### 3.11 Notificações

**Objetivo:** Exibir uma lista de notificações para o usuário, permitindo que ele as marque como lidas ou as arquive.

**Rota:** `/notifications`

**Perfis com acesso:** `owner`, `admin`, `employee`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem de notificações)
*   `Button` (para marcar como lida, arquivar, excluir)
*   `StatusBadge` (para status da notificação)
*   `EmptyState`
*   `Skeleton`

**Campos (Listagem):**
*   Tipo (`notification_type`)
*   Título
*   Mensagem (truncada)
*   Data
*   Status (`notification_status`)

**Botões:**
*   **Marcar como Lida:** Na linha da notificação.
*   **Arquivar:** Na linha da notificação.
*   **Marcar todas como lidas:** Botão global.

**Estados de loading:**
*   `Skeleton` na `DataTable`.

**Empty states:**
*   "Nenhuma notificação para exibir."

**Mensagens de erro:**
*   "Não foi possível carregar as notificações. Tente novamente."

**Validações:** Nenhuma direta.

**Responsividade:** Tabela com colunas ocultáveis.

**Fluxo do usuário:**
1.  Usuário acessa `/notifications`.
2.  Visualiza a lista de notificações.
3.  Pode interagir com as notificações (marcar como lida, arquivar).

**Permissões:**
*   Todos os perfis podem visualizar e gerenciar suas próprias notificações.
*   `owner`, `admin` podem visualizar e gerenciar notificações de outros usuários do tenant (para fins de suporte/auditoria).

**Integrações com Supabase:**
*   `notifications` (leitura e atualização de status).

**Integrações com Stripe:** Nenhuma direta.

**Critérios de aceite:**
*   Listagem de notificações funciona.
*   Marcar como lida e arquivar funcionam.
*   Permissões de acesso são respeitadas.

### 3.12 Auditoria

**Objetivo:** Exibir um log de auditoria das ações importantes realizadas no sistema, para rastreabilidade e segurança.

**Rota:** `/audit-logs`

**Perfis com acesso:** `owner`, `admin`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem de logs)
*   `SearchBar`, `Select`, `DatePicker` (para filtros)
*   `EmptyState`
*   `Skeleton`

**Campos (Listagem):**
*   Data/Hora
*   Usuário
*   Ação
*   Entidade
*   ID da Entidade
*   IP

**Botões:**
*   Botões de filtro por data, usuário, ação, entidade.

**Estados de loading:**
*   `Skeleton` na `DataTable`.

**Empty states:**
*   "Nenhum log de auditoria encontrado para os critérios selecionados."

**Mensagens de erro:**
*   "Não foi possível carregar os logs de auditoria. Tente novamente."

**Validações:** Nenhuma direta.

**Responsividade:** Tabela com colunas ocultáveis.

**Fluxo do usuário:**
1.  `owner`/`admin` acessa `/audit-logs`.
2.  Visualiza a lista de logs, pode filtrar para encontrar ações específicas.

**Permissões:**
*   `owner`, `admin`: Acesso total aos logs de auditoria do seu tenant.
*   `employee`: Nenhuma permissão para esta seção.

**Integrações com Supabase:**
*   `audit_logs` (leitura).

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Listagem de logs funciona com filtros e paginação.
*   Informações do log são claras e úteis.
*   Permissões de acesso são respeitadas.

### 3.13 Página 404 (Não Encontrado)

**Objetivo:** Informar ao usuário que a página solicitada não foi encontrada.

**Rota:** Qualquer rota não existente.

**Perfis com acesso:** Todos.

**Componentes utilizados:**
*   Layout básico da aplicação (Header/Sidebar, se aplicável).
*   `Card` ou `div` centralizado.
*   `Button` (para voltar ao Dashboard).

**Campos:**
*   Título: "404 - Página Não Encontrada"
*   Mensagem: "A página que você está procurando não existe ou foi movida."

**Botões:**
*   **Voltar para o Dashboard:** Redireciona para `/dashboard`.

**Estados de loading:** Não aplicável.

**Empty states:** Não aplicável.

**Mensagens de erro:** Não aplicável.

**Validações:** Não aplicável.

**Responsividade:** Layout centralizado e responsivo.

**Fluxo do usuário:**
1.  Usuário tenta acessar uma URL inválida.
2.  É redirecionado para a página 404.
3.  Pode clicar para voltar ao dashboard.

**Permissões:** Acesso público.

**Integrações com Supabase:** Nenhuma.

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Página 404 é exibida para rotas inválidas.
*   Botão "Voltar para o Dashboard" funciona.

### 3.14 Página 500 (Erro Interno do Servidor)

**Objetivo:** Informar ao usuário que ocorreu um erro inesperado no servidor.

**Rota:** Erros internos do servidor.

**Perfis com acesso:** Todos.

**Componentes utilizados:**
*   Layout básico da aplicação.
*   `Card` ou `div` centralizado.
*   `Button` (para voltar ao Dashboard).

**Campos:**
*   Título: "500 - Erro Interno do Servidor"
*   Mensagem: "Ocorreu um erro inesperado. Nossa equipe já foi notificada. Por favor, tente novamente mais tarde ou entre em contato com o suporte."

**Botões:**
*   **Voltar para o Dashboard:** Redireciona para `/dashboard`.

**Estados de loading:** Não aplicável.

**Empty states:** Não aplicável.

**Mensagens de erro:** Não aplicável.

**Validações:** Não aplicável.

**Responsividade:** Layout centralizado e responsivo.

**Fluxo do usuário:**
1.  Ocorre um erro fatal no servidor.
2.  Usuário é redirecionado para a página 500.
3.  Pode clicar para voltar ao dashboard.

**Permissões:** Acesso público.

**Integrações com Supabase:** Nenhuma direta, mas erros podem ser logados no backend.

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Página 500 é exibida para erros internos.
*   Botão "Voltar para o Dashboard" funciona.

## 4. Estados Globais e Feedback ao Usuário

Para garantir uma experiência de usuário fluida e informativa, o sistema implementará estados globais de feedback e carregamento.

### 4.1 Loading

**Objetivo:** Indicar que uma operação está em andamento e o usuário deve aguardar.

*   **Implementação:** Spinners em botões, indicadores de progresso em formulários, `Skeleton` loaders em listas e dashboards.
*   **Contexto:** Usado em submissões de formulários, carregamento de dados em listas, gráficos e dashboards.

### 4.2 Skeletons

**Objetivo:** Fornecer um feedback visual de que o conteúdo está sendo carregado, preenchendo o layout com placeholders antes que os dados reais cheguem.

*   **Implementação:** Componentes `Skeleton` do `shadcn/ui` ou customizados.
*   **Contexto:** Listagens (`DataTable`), `KPICard`, `MetricCard`, `Charts`, e outras seções com carregamento assíncrono de dados.

### 4.3 Empty States

**Objetivo:** Informar ao usuário quando não há dados para exibir em uma determinada seção, oferecendo uma ação clara para preencher essa lacuna.

*   **Implementação:** Componente `EmptyState` customizado com ícone, título, descrição e botão de ação (ex: "Adicionar Cliente").
*   **Contexto:** Listagens vazias (Clientes, Veículos, OS, Produtos, Serviços, Notificações, Logs de Auditoria).

### 4.4 Toasts

**Objetivo:** Exibir mensagens de feedback rápidas e não intrusivas para o usuário, informando sobre o sucesso ou falha de uma operação.

*   **Implementação:** Componente `Toast` do `shadcn/ui`.
*   **Tipos:** Sucesso, Erro, Aviso, Informação.
*   **Contexto:** Após salvar um formulário, excluir um item, convidar um usuário, etc.

### 4.5 Modais

**Objetivo:** Exibir conteúdo ou formulários em uma janela sobreposta à página principal, exigindo interação do usuário.

*   **Implementação:** Componente `Dialog` do `shadcn/ui`.
*   **Contexto:** Formulários de cadastro/edição (clientes, veículos, serviços, produtos), confirmações (`ConfirmDialog`), seleção de itens.

### 4.6 Drawers

**Objetivo:** Similar aos modais, mas deslizam da lateral da tela, ideal para formulários mais complexos ou detalhes que precisam de mais espaço.

*   **Implementação:** Componente `Drawer` (se disponível no `shadcn/ui` ou customizado).
*   **Contexto:** Alternativa aos modais para formulários de cadastro/edição, filtros avançados.

### 4.7 Confirmações

**Objetivo:** Solicitar confirmação do usuário antes de executar ações destrutivas ou irreversíveis.

*   **Implementação:** Componente `ConfirmDialog` (baseado em `AlertDialog` do `shadcn/ui`).
*   **Contexto:** Exclusão de registros (clientes, veículos, produtos, serviços), cancelamento de assinatura, finalização de OS.

## Conclusão

Este documento fornece uma especificação técnica abrangente para o frontend do OficinaPro, cobrindo a estrutura do projeto, componentes reutilizáveis, e o detalhamento de cada página do sistema. A aderência a esta especificação garantirá que o desenvolvimento do frontend seja consistente, eficiente e resulte em uma experiência de usuário premium, alinhada com as melhores práticas de UI/UX e as referências de design estabelecidas. Com esta base, as ferramentas de IA (Cursor, Claude, ChatGPT) e os desenvolvedores terão todas as informações necessárias para construir a interface do OficinaPro com precisão e qualidade.

### 3.7 Ordens de Serviço

**Objetivo:** Gerenciar o ciclo de vida completo das ordens de serviço, desde a abertura até a finalização e entrega do veículo.

**Rota:** `/work-orders` (listagem), `/work-orders/new` (cadastro), `/work-orders/[id]` (detalhes/edição)

**Perfis com acesso:** `owner`, `admin`, `employee`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem)
*   `SearchBar`
*   `Button` (para adicionar, editar, excluir, mudar status)
*   `FormField` (para formulários)
*   `Input`, `Select`, `Textarea`, `DatePicker` (para campos de formulário)
*   `Modal` ou `Drawer` (para formulários)
*   `ConfirmDialog` (para exclusão, aprovação, finalização)
*   `EmptyState`
*   `Skeleton`
*   `Tabs` (para abas de detalhes da OS)
*   `StatusBadge`

**Campos (Listagem):**
*   Número da OS
*   Cliente
*   Veículo (Placa/Modelo)
*   Status
*   Data de Abertura
*   Valor Total

**Campos (Cadastro/Edição - Aba Dados Gerais):**
*   **Cliente:** `Select` (busca de clientes existentes)
*   **Veículo:** `Select` (busca de veículos do cliente selecionado)
*   **Relato do Cliente:** `Textarea`
*   **Quilometragem de Entrada:** `Input` (numérico)
*   **Previsão de Entrega:** `DatePicker`
*   **Técnico Responsável:** `Select` (usuários do tenant com role `employee`)
*   **Observações Internas:** `Textarea`

**Campos (Aba Diagnóstico):**
*   **Diagnóstico Técnico:** `Textarea`

**Campos (Aba Serviços):**
*   Listagem de `work_order_services` (DataTable interna)
*   Botão "Adicionar Serviço" (abre modal/drawer para selecionar serviço do catálogo ou adicionar avulso)
*   Campos para cada item de serviço: Descrição, Quantidade, Preço Unitário, Desconto, Total.

**Campos (Aba Produtos):**
*   Listagem de `work_order_products` (DataTable interna)
*   Botão "Adicionar Produto" (abre modal/drawer para selecionar produto do catálogo ou adicionar avulso)
*   Campos para cada item de produto: Descrição, Quantidade, Preço Unitário, Desconto, Total.

**Campos (Aba Financeiro):**
*   Resumo de totais (Serviços, Produtos, Desconto, Acréscimos, Total Geral).
*   Botão "Registrar Pagamento" (abre modal para registrar `financial_transaction` e atualizar `accounts_receivable`).
*   Listagem de pagamentos/contas a receber associados.

**Campos (Aba Histórico):**
*   Listagem de `work_order_history` (DataTable interna)
*   Exibe: Status Anterior, Novo Status, Usuário, Data/Hora, Observações.

**Botões:**
*   **Nova OS:** Na página de listagem.
*   **Salvar Rascunho / Salvar e Abrir:** No formulário de cadastro/edição.
*   **Mudar Status:** Botão principal na página de detalhes da OS (dropdown com opções: Diagnóstico, Aguardando Aprovação, Aprovada, Em Andamento, Concluída, Entregue, Cancelada, Reabrir).
*   **Aprovar OS:** Botão específico quando status é `waiting_approval`.
*   **Finalizar OS:** Botão específico quando status é `in_progress`.
*   **Registrar Pagamento:** Na aba Financeiro.
*   **Editar / Excluir:** Na página de detalhes (exclusão via soft delete).

**Estados de loading:**
*   `Skeleton` na `DataTable` e nas abas de detalhes.
*   Botões desabilitados e com spinner durante ações de mudança de status ou salvamento.

**Empty states:**
*   "Nenhuma Ordem de Serviço encontrada."
*   "Nenhum serviço adicionado a esta OS."
*   "Nenhum produto adicionado a esta OS."
*   "Nenhum histórico de status para esta OS."

**Mensagens de erro:**
*   "Cliente e Veículo são obrigatórios para abrir uma OS."
*   "Não é possível mudar para este status. Verifique o fluxo."
*   "Não foi possível salvar a Ordem de Serviço. Tente novamente."

**Validações:**
*   Campos obrigatórios.
*   Validação de fluxo de status (ex: não pode ir de `draft` para `completed` diretamente).
*   Validação de estoque ao adicionar produtos.

**Responsividade:**
*   Listagem: Tabela com colunas ocultáveis.
*   Página de detalhes: Abas adaptáveis, layout de 1 coluna em mobile.

**Fluxo do usuário:**
1.  Usuário acessa `/work-orders` e vê a lista.
2.  Clica em "Nova OS", seleciona cliente/veículo, preenche relato e salva como rascunho ou abre.
3.  Na página de detalhes, navega pelas abas (Dados, Diagnóstico, Serviços, Produtos, Financeiro, Histórico).
4.  Adiciona serviços e produtos, registra diagnóstico.
5.  Muda o status da OS conforme o andamento (ex: para `waiting_approval`).
6.  Cliente aprova, status muda para `approved`.
7.  Serviços são executados, status muda para `in_progress`.
8.  OS é finalizada, status muda para `completed`.
9.  Pagamento é registrado, veículo é entregue, status muda para `delivered`.

**Permissões:**
*   `owner`, `admin`: CRUD completo de OS, mudança de status, aprovação, finalização.
*   `employee`: Criar, visualizar, editar OS (exceto campos financeiros sensíveis), mudar status (com restrições).

**Integrações com Supabase:**
*   `work_orders` (CRUD)
*   `work_order_services` (CRUD)
*   `work_order_products` (CRUD)
*   `work_order_history` (INSERT via trigger)
*   `customers`, `vehicles`, `services`, `products`, `profiles` (para seleção e exibição de dados relacionados)
*   `accounts_receivable`, `financial_transactions` (para aba Financeiro)

**Integrações com Stripe:** Nenhuma direta.

**Critérios de aceite:**
*   Listagem de OS funciona com busca, filtro e paginação.
*   Criação e edição de OS funcionam com todas as abas e validações.
*   Fluxo de mudança de status é intuitivo e respeita as regras de negócio.
*   Cálculo de totais de serviços, produtos, descontos e total geral é preciso.
*   Histórico de status é registrado automaticamente.
*   Permissões de acesso são respeitadas.

### 3.8 Financeiro

**Objetivo:** Gerenciar as finanças da oficina, incluindo receitas, despesas, contas a receber e a pagar, e fluxo de caixa.

**Rota:** `/financial` (visão geral), `/financial/receivables` (contas a receber), `/financial/payables` (contas a pagar), `/financial/transactions` (fluxo de caixa)

**Perfis com acesso:** `owner`, `admin`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `KPICard`
*   `MetricCard`
*   `Charts` (para gráficos de linha, barra, rosca)
*   `Card` (para agrupar seções)
*   `DataTable` (para listagens de contas e transações)
*   `SearchBar`, `Select`, `DatePicker` (para filtros)
*   `Button` (para adicionar, editar, registrar pagamento/recebimento)
*   `Modal` ou `Drawer` (para formulários)
*   `ConfirmDialog` (para exclusão/cancelamento)
*   `EmptyState`
*   `Skeleton`
*   `Tabs` (para navegação entre sub-seções financeiras)

**Campos (Visão Geral):**
*   KPIs: Receita Total, Despesa Total, Lucro Líquido, Contas a Receber (vencidas/a vencer), Contas a Pagar (vencidas/a vencer).
*   Gráficos: Fluxo de Caixa Mensal (linha), Despesas por Categoria (rosca), Receitas por Origem (barra).

**Campos (Contas a Receber/Pagar - Listagem):**
*   Descrição
*   Cliente/Fornecedor
*   Valor
*   Data de Vencimento
*   Status
*   Data de Pagamento

**Campos (Contas a Receber - Cadastro/Edição):**
*   **Cliente:** `Select`
*   **Descrição:** `Input`
*   **Valor:** `Input` (monetário)
*   **Data de Vencimento:** `DatePicker`
*   **Status:** `Select` (Pendente, Pago, Vencido, Cancelado)
*   **Método de Pagamento:** `Select` (se status for Pago)

**Campos (Contas a Pagar - Cadastro/Edição):**
*   **Fornecedor:** `Input`
*   **Categoria:** `Input` ou `Select`
*   **Descrição:** `Input`
*   **Valor:** `Input` (monetário)
*   **Data de Vencimento:** `DatePicker`
*   **Status:** `Select` (Pendente, Pago, Vencido, Cancelado)
*   **Método de Pagamento:** `Select` (se status for Pago)

**Campos (Fluxo de Caixa - Listagem):**
*   Tipo (Receita/Despesa)
*   Descrição
*   Valor
*   Data da Transação
*   Método de Pagamento
*   Origem (OS, Manual, Assinatura)

**Botões:**
*   **Adicionar Receita / Adicionar Despesa:** Na visão geral ou nas respectivas listagens.
*   **Registrar Pagamento / Registrar Recebimento:** Na listagem de contas.
*   **Editar / Excluir / Cancelar:** Na linha da tabela.
*   Botões de filtro de período.

**Estados de loading:**
*   `Skeleton` para KPIs, gráficos e DataTables.
*   Formulários desabilitados e com spinner.

**Empty states:**
*   "Nenhuma conta a receber encontrada."
*   "Nenhuma transação financeira registrada."

**Mensagens de erro:**
*   "Valor é obrigatório e deve ser numérico."
*   "Data de vencimento é obrigatória."
*   "Não foi possível salvar a conta. Tente novamente."

**Validações:**
*   Campos obrigatórios.
*   Formato numérico para valores.
*   Datas válidas.

**Responsividade:**
*   Layout de cards e tabelas adaptáveis.
*   Gráficos responsivos.

**Fluxo do usuário:**
1.  Usuário acessa `/financial` e vê o resumo.
2.  Navega para Contas a Receber, Contas a Pagar ou Fluxo de Caixa.
3.  Adiciona novas contas/transações, registra pagamentos, edita ou cancela.

**Permissões:**
*   `owner`, `admin`: Acesso total a todas as funcionalidades financeiras.
*   `employee`: Nenhuma permissão para esta seção.

**Integrações com Supabase:**
*   `accounts_receivable` (CRUD)
*   `accounts_payable` (CRUD)
*   `financial_transactions` (CRUD)
*   `work_orders`, `customers` (para relacionamentos)

**Integrações com Stripe:**
*   Sincronização de pagamentos de assinaturas (via webhooks, que podem gerar `financial_transactions` e `invoices`).

**Critérios de aceite:**
*   Visão geral financeira clara e atualizada.
*   Listagens de contas e transações funcionam com filtros.
*   Cadastro, edição e registro de pagamentos/recebimentos funcionam.
*   Cálculos financeiros são precisos.
*   Permissões de acesso são respeitadas.

### 3.9 Assinaturas

**Objetivo:** Permitir que o `owner` da oficina gerencie sua assinatura com o OficinaPro, incluindo visualização do plano atual, upgrade, downgrade e cancelamento.

**Rota:** `/subscriptions`

**Perfis com acesso:** `owner`, `admin` (apenas visualização).

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `Card` (para exibir detalhes do plano atual e opções)
*   `Button` (para upgrade, downgrade, cancelar, ir para portal Stripe)
*   `Modal` ou `Drawer` (para confirmação de ações)
*   `EmptyState`
*   `Skeleton`
*   `StatusBadge`

**Campos:**
*   **Plano Atual:** Nome do plano, descrição, preço, intervalo de cobrança.
*   **Status da Assinatura:** `StatusBadge` (Ativa, Trial, Vencida, Cancelada).
*   **Próxima Cobrança:** Data e valor.
*   **Recursos do Plano:** Lista de funcionalidades/limites.

**Botões:**
*   **Mudar Plano:** Abre uma lista de planos disponíveis para upgrade/downgrade.
*   **Cancelar Assinatura:** Abre um `ConfirmDialog`.
*   **Gerenciar Pagamento (Portal Stripe):** Redireciona para o portal de clientes da Stripe.

**Estados de loading:**
*   `Skeleton` para os detalhes da assinatura.
*   Botões desabilitados e com spinner durante ações de mudança/cancelamento.

**Empty states:**
*   "Nenhuma assinatura ativa encontrada. Inicie seu período de trial!"

**Mensagens de erro:**
*   "Não foi possível carregar os detalhes da assinatura. Tente novamente."
*   "Ocorreu um erro ao tentar mudar o plano. Tente novamente."
*   "Não foi possível cancelar a assinatura. Entre em contato com o suporte."

**Validações:**
*   Ações de upgrade/downgrade/cancelamento devem ser validadas pelo backend e Stripe.

**Responsividade:** Layout de cards adaptável.

**Fluxo do usuário:**
1.  `owner` acessa `/subscriptions`.
2.  Visualiza o plano atual e seu status.
3.  Pode optar por mudar de plano, cancelar ou gerenciar pagamentos via Stripe.

**Permissões:**
*   `owner`: Acesso total (visualizar, mudar plano, cancelar).
*   `admin`: Apenas visualizar o status da assinatura.
*   `employee`: Nenhuma permissão para esta seção.

**Integrações com Supabase:**
*   `subscriptions` (leitura e atualização via backend/webhooks)
*   `subscription_plans` (leitura)
*   `tenants` (atualização do status do tenant)

**Integrações com Stripe:**
*   API da Stripe para listar planos, criar/atualizar/cancelar assinaturas.
*   Redirecionamento para o Stripe Customer Portal.

**Critérios de aceite:**
*   Exibição clara do plano atual e status da assinatura.
*   Funcionalidades de mudança de plano e cancelamento funcionam corretamente.
*   Redirecionamento para o portal Stripe funciona.
*   Mensagens de feedback claras para todas as ações.
*   Permissões de acesso são respeitadas.
### 3.10 Configurações

**Objetivo:** Permitir que `owner` e `admin` configurem aspectos da oficina, gerenciem usuários e definam preferências pessoais.

**Rota:** `/settings` (visão geral), `/settings/profile` (perfil), `/settings/company` (empresa), `/settings/users` (usuários), `/settings/preferences` (preferências)

**Perfis com acesso:** `owner`, `admin`, `employee` (apenas perfil).

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `Tabs` (para navegação entre sub-seções de configurações)
*   `FormField`
*   `Input`, `Select`, `Textarea`, `Checkbox`, `FileUpload` (para logo)
*   `Button` (para salvar, convidar usuário, remover usuário)
*   `DataTable` (para listagem de usuários)
*   `ConfirmDialog` (para remover usuário)
*   `EmptyState`
*   `Skeleton`

**Campos (Perfil - `/settings/profile`):**
*   **Nome Completo:** `Input`
*   **E-mail:** `Input` (somente leitura, via Supabase Auth)
*   **Telefone:** `Input`
*   **URL do Avatar:** `FileUpload` ou `Input`
*   **Senha:** Botão "Alterar Senha" (redireciona para `/forgot-password` ou modal)

**Campos (Empresa - `/settings/company`):**
*   **Razão Social:** `Input`
*   **Nome Fantasia:** `Input`
*   **CNPJ/CPF:** `Input`
*   **Inscrição Estadual:** `Input`
*   **E-mail:** `Input`
*   **Telefone:** `Input`
*   **WhatsApp:** `Input`
*   **Logo:** `FileUpload` (para upload no Supabase Storage)
*   **Endereço:** Campos de `Input` para CEP, Rua, Número, Complemento, Bairro, Cidade, Estado, País.
*   **Configurações Financeiras:** Moeda padrão, Taxa de imposto padrão.
*   **Configurações Operacionais:** Prefixo de OS, Próximo número de OS (somente leitura).

**Campos (Usuários - `/settings/users`):**
*   Listagem de `tenant_users` (DataTable)
*   Botão "Convidar Usuário" (abre modal com campo de e-mail e seleção de `role`)
*   Campos para cada usuário: Nome, E-mail, Perfil (`role`), Status (`membership_status`), Ações (Editar Perfil, Remover).

**Campos (Preferências - `/settings/preferences`):**
*   **Tema:** `Select` (Sistema, Claro, Escuro)
*   **Idioma:** `Select` (Português, Inglês)
*   **Notificações por E-mail:** `Checkbox`
*   **Notificações no Aplicativo:** `Checkbox`

**Botões:**
*   **Salvar Alterações:** Em cada sub-seção de configurações.
*   **Convidar Usuário:** Na sub-seção de Usuários.
*   **Editar Perfil / Remover:** Na linha da tabela de usuários.
*   **Alterar Senha:** Na sub-seção de Perfil.

**Estados de loading:**
*   `Skeleton` para formulários e DataTables.
*   Botões desabilitados e com spinner durante submissão.

**Empty states:**
*   "Nenhum usuário convidado ainda."

**Mensagens de erro:**
*   "E-mail inválido para convite."
*   "Não foi possível salvar as configurações. Tente novamente."
*   "Não foi possível convidar o usuário. Tente novamente."

**Validações:**
*   Campos obrigatórios e formatos válidos.
*   Validação de e-mail para convite.

**Responsividade:**
*   Abas adaptáveis, formulários empilhados em mobile.
*   Tabela de usuários com colunas ocultáveis.

**Fluxo do usuário:**
1.  Usuário acessa `/settings`.
2.  Navega entre as abas para configurar perfil, empresa, usuários ou preferências.
3.  `owner`/`admin` convida novos usuários ou gerencia os existentes.

**Permissões:**
*   `owner`: Acesso total a todas as configurações.
*   `admin`: Acesso a configurações da empresa e usuários (exceto exclusão de owner), mas não pode alterar plano de assinatura.
*   `employee`: Apenas acesso à sub-seseção de Perfil (`/settings/profile`).

**Integrações com Supabase:**
*   `profiles` (CRUD para perfil)
*   `tenants` (CRUD para configurações da empresa)
*   `tenant_users` (CRUD para usuários do tenant)
*   `settings` (CRUD para preferências e outras configurações)
*   Supabase Storage (para upload de logo)
*   Supabase Auth (para convite de usuários)

**Integrações com Stripe:** Nenhuma direta.

**Critérios de aceite:**
*   Todas as sub-seções de configurações funcionam.
*   Salvar alterações funciona com validações e feedback.
*   Convite e gerenciamento de usuários funcionam.
*   Upload de logo funciona.
*   Permissões de acesso são respeitadas.

### 3.11 Notificações

**Objetivo:** Exibir uma lista de notificações para o usuário, permitindo que ele as marque como lidas ou as arquive.

**Rota:** `/notifications`

**Perfis com acesso:** `owner`, `admin`, `employee`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem de notificações)
*   `Button` (para marcar como lida, arquivar, excluir)
*   `StatusBadge` (para status da notificação)
*   `EmptyState`
*   `Skeleton`

**Campos (Listagem):**
*   Tipo (`notification_type`)
*   Título
*   Mensagem (truncada)
*   Data
*   Status (`notification_status`)

**Botões:**
*   **Marcar como Lida:** Na linha da notificação.
*   **Arquivar:** Na linha da notificação.
*   **Marcar todas como lidas:** Botão global.

**Estados de loading:**
*   `Skeleton` na `DataTable`.

**Empty states:**
*   "Nenhuma notificação para exibir."

**Mensagens de erro:**
*   "Não foi possível carregar as notificações. Tente novamente."

**Validações:** Nenhuma direta.

**Responsividade:** Tabela com colunas ocultáveis.

**Fluxo do usuário:**
1.  Usuário acessa `/notifications`.
2.  Visualiza a lista de notificações.
3.  Pode interagir com as notificações (marcar como lida, arquivar).

**Permissões:**
*   Todos os perfis podem visualizar e gerenciar suas próprias notificações.
*   `owner`, `admin` podem visualizar e gerenciar notificações de outros usuários do tenant (para fins de suporte/auditoria).

**Integrações com Supabase:**
*   `notifications` (leitura e atualização de status).

**Integrações com Stripe:** Nenhuma direta.

**Critérios de aceite:**
*   Listagem de notificações funciona.
*   Marcar como lida e arquivar funcionam.
*   Permissões de acesso são respeitadas.

### 3.12 Auditoria

**Objetivo:** Exibir um log de auditoria das ações importantes realizadas no sistema, para rastreabilidade e segurança.

**Rota:** `/audit-logs`

**Perfis com acesso:** `owner`, `admin`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem de logs)
*   `SearchBar`, `Select`, `DatePicker` (para filtros)
*   `EmptyState`
*   `Skeleton`

**Campos (Listagem):**
*   Data/Hora
*   Usuário
*   Ação
*   Entidade
*   ID da Entidade
*   IP

**Botões:**
*   Botões de filtro por data, usuário, ação, entidade.

**Estados de loading:**
*   `Skeleton` na `DataTable`.

**Empty states:**
*   "Nenhum log de auditoria encontrado para os critérios selecionados."

**Mensagens de erro:**
*   "Não foi possível carregar os logs de auditoria. Tente novamente."

**Validações:** Nenhuma direta.

**Responsividade:** Tabela com colunas ocultáveis.

**Fluxo do usuário:**
1.  `owner`/`admin` acessa `/audit-logs`.
2.  Visualiza a lista de logs, pode filtrar para encontrar ações específicas.

**Permissões:**
*   `owner`, `admin`: Acesso total aos logs de auditoria do seu tenant.
*   `employee`: Nenhuma permissão para esta seção.

**Integrações com Supabase:**
*   `audit_logs` (leitura).

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Listagem de logs funciona com filtros e paginação.
*   Informações do log são claras e úteis.
*   Permissões de acesso são respeitadas.

### 3.13 Página 404 (Não Encontrado)

**Objetivo:** Informar ao usuário que a página solicitada não foi encontrada.

**Rota:** Qualquer rota não existente.

**Perfis com acesso:** Todos.

**Componentes utilizados:**
*   Layout básico da aplicação (Header/Sidebar, se aplicável).
*   `Card` ou `div` centralizado.
*   `Button` (para voltar ao Dashboard).

**Campos:**
*   Título: "404 - Página Não Encontrada"
*   Mensagem: "A página que você está procurando não existe ou foi movida."

**Botões:**
*   **Voltar para o Dashboard:** Redireciona para `/dashboard`.

**Estados de loading:** Não aplicável.

**Empty states:** Não aplicável.

**Mensagens de erro:** Não aplicável.

**Validações:** Não aplicável.

**Responsividade:** Layout centralizado e responsivo.

**Fluxo do usuário:**
1.  Usuário tenta acessar uma URL inválida.
2.  É redirecionado para a página 404.
3.  Pode clicar para voltar ao dashboard.

**Permissões:** Acesso público.

**Integrações com Supabase:** Nenhuma.

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Página 404 é exibida para rotas inválidas.
*   Botão "Voltar para o Dashboard" funciona.

### 3.14 Página 500 (Erro Interno do Servidor)

**Objetivo:** Informar ao usuário que ocorreu um erro inesperado no servidor.

**Rota:** Erros internos do servidor.

**Perfis com acesso:** Todos.

**Componentes utilizados:**
*   Layout básico da aplicação.
*   `Card` ou `div` centralizado.
*   `Button` (para voltar ao Dashboard).

**Campos:**
*   Título: "500 - Erro Interno do Servidor"
*   Mensagem: "Ocorreu um erro inesperado. Nossa equipe já foi notificada. Por favor, tente novamente mais tarde ou entre em contato com o suporte."

**Botões:**
*   **Voltar para o Dashboard:** Redireciona para `/dashboard`.

**Estados de loading:** Não aplicável.

**Empty states:** Não aplicável.

**Mensagens de erro:** Não aplicável.

**Validações:** Não aplicável.

**Responsividade:** Layout centralizado e responsivo.

**Fluxo do usuário:**
1.  Ocorre um erro fatal no servidor.
2.  Usuário é redirecionado para a página 500.
3.  Pode clicar para voltar ao dashboard.

**Permissões:** Acesso público.

**Integrações com Supabase:** Nenhuma direta, mas erros podem ser logados no backend.

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Página 500 é exibida para erros internos.
*   Botão "Voltar para o Dashboard" funciona.

## 4. Estados Globais e Feedback ao Usuário

Para garantir uma experiência de usuário fluida e informativa, o sistema implementará estados globais de feedback e carregamento.

### 4.1 Loading

**Objetivo:** Indicar que uma operação está em andamento e o usuário deve aguardar.

*   **Implementação:** Spinners em botões, indicadores de progresso em formulários, `Skeleton` loaders em listas e dashboards.
*   **Contexto:** Usado em submissões de formulários, carregamento de dados em listas, gráficos e dashboards.

### 4.2 Skeletons

**Objetivo:** Fornecer um feedback visual de que o conteúdo está sendo carregado, preenchendo o layout com placeholders antes que os dados reais cheguem.

*   **Implementação:** Componentes `Skeleton` do `shadcn/ui` ou customizados.
*   **Contexto:** Listagens (`DataTable`), `KPICard`, `MetricCard`, `Charts`, e outras seções com carregamento assíncrono de dados.

### 4.3 Empty States

**Objetivo:** Informar ao usuário quando não há dados para exibir em uma determinada seção, oferecendo uma ação clara para preencher essa lacuna.

*   **Implementação:** Componente `EmptyState` customizado com ícone, título, descrição e botão de ação (ex: "Adicionar Cliente").
*   **Contexto:** Listagens vazias (Clientes, Veículos, OS, Produtos, Serviços, Notificações, Logs de Auditoria).

### 4.4 Toasts

**Objetivo:** Exibir mensagens de feedback rápidas e não intrusivas para o usuário, informando sobre o sucesso ou falha de uma operação.

*   **Implementação:** Componente `Toast` do `shadcn/ui`.
*   **Tipos:** Sucesso, Erro, Aviso, Informação.
*   **Contexto:** Após salvar um formulário, excluir um item, convidar um usuário, etc.

### 4.5 Modais

**Objetivo:** Exibir conteúdo ou formulários em uma janela sobreposta à página principal, exigindo interação do usuário.

*   **Implementação:** Componente `Dialog` do `shadcn/ui`.
*   **Contexto:** Formulários de cadastro/edição (clientes, veículos, serviços, produtos), confirmações (`ConfirmDialog`), seleção de itens.

### 4.6 Drawers

**Objetivo:** Similar aos modais, mas deslizam da lateral da tela, ideal para formulários mais complexos ou detalhes que precisam de mais espaço.

*   **Implementação:** Componente `Drawer` (se disponível no `shadcn/ui` ou customizado).
*   **Contexto:** Alternativa aos modais para formulários de cadastro/edição, filtros avançados.

### 4.7 Confirmações

**Objetivo:** Solicitar confirmação do usuário antes de executar ações destrutivas ou irreversíveis.

*   **Implementação:** Componente `ConfirmDialog` (baseado em `AlertDialog` do `shadcn/ui`).
*   **Contexto:** Exclusão de registros (clientes, veículos, produtos, serviços), cancelamento de assinatura, finalização de OS.

## Conclusão

Este documento fornece uma especificação técnica abrangente para o frontend do OficinaPro, cobrindo a estrutura do projeto, componentes reutilizáveis, e o detalhamento de cada página do sistema. A aderência a esta especificação garantirá que o desenvolvimento do frontend seja consistente, eficiente e resulte em uma experiência de usuário premium, alinhada com as melhores práticas de UI/UX e as referências de design estabelecidas. Com esta base, as ferramentas de IA (Cursor, Claude, ChatGPT) e os desenvolvedores terão todas as informações necessárias para construir a interface do OficinaPro com precisão e qualidade.

### 3.10 Configurações

**Objetivo:** Permitir que `owner` e `admin` configurem aspectos da oficina, gerenciem usuários e definam preferências pessoais.

**Rota:** `/settings` (visão geral), `/settings/profile` (perfil), `/settings/company` (empresa), `/settings/users` (usuários), `/settings/preferences` (preferências)

**Perfis com acesso:** `owner`, `admin`, `employee` (apenas perfil).

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `Tabs` (para navegação entre sub-seções de configurações)
*   `FormField`
*   `Input`, `Select`, `Textarea`, `Checkbox`, `FileUpload` (para logo)
*   `Button` (para salvar, convidar usuário, remover usuário)
*   `DataTable` (para listagem de usuários)
*   `ConfirmDialog` (para remover usuário)
*   `EmptyState`
*   `Skeleton`

**Campos (Perfil - `/settings/profile`):**
*   **Nome Completo:** `Input`
*   **E-mail:** `Input` (somente leitura, via Supabase Auth)
*   **Telefone:** `Input`
*   **URL do Avatar:** `FileUpload` ou `Input`
*   **Senha:** Botão "Alterar Senha" (redireciona para `/forgot-password` ou modal)

**Campos (Empresa - `/settings/company`):**
*   **Razão Social:** `Input`
*   **Nome Fantasia:** `Input`
*   **CNPJ/CPF:** `Input`
*   **Inscrição Estadual:** `Input`
*   **E-mail:** `Input`
*   **Telefone:** `Input`
*   **WhatsApp:** `Input`
*   **Logo:** `FileUpload` (para upload no Supabase Storage)
*   **Endereço:** Campos de `Input` para CEP, Rua, Número, Complemento, Bairro, Cidade, Estado, País.
*   **Configurações Financeiras:** Moeda padrão, Taxa de imposto padrão.
*   **Configurações Operacionais:** Prefixo de OS, Próximo número de OS (somente leitura).

**Campos (Usuários - `/settings/users`):**
*   Listagem de `tenant_users` (DataTable)
*   Botão "Convidar Usuário" (abre modal com campo de e-mail e seleção de `role`)
*   Campos para cada usuário: Nome, E-mail, Perfil (`role`), Status (`membership_status`), Ações (Editar Perfil, Remover).

**Campos (Preferências - `/settings/preferences`):**
*   **Tema:** `Select` (Sistema, Claro, Escuro)
*   **Idioma:** `Select` (Português, Inglês)
*   **Notificações por E-mail:** `Checkbox`
*   **Notificações no Aplicativo:** `Checkbox`

**Botões:**
*   **Salvar Alterações:** Em cada sub-seção de configurações.
*   **Convidar Usuário:** Na sub-seção de Usuários.
*   **Editar Perfil / Remover:** Na linha da tabela de usuários.
*   **Alterar Senha:** Na sub-seção de Perfil.

**Estados de loading:**
*   `Skeleton` para formulários e DataTables.
*   Botões desabilitados e com spinner durante submissão.

**Empty states:**
*   "Nenhum usuário convidado ainda."

**Mensagens de erro:**
*   "E-mail inválido para convite."
*   "Não foi possível salvar as configurações. Tente novamente."
*   "Não foi possível convidar o usuário. Tente novamente."

**Validações:**
*   Campos obrigatórios e formatos válidos.
*   Validação de e-mail para convite.

**Responsividade:**
*   Abas adaptáveis, formulários empilhados em mobile.
*   Tabela de usuários com colunas ocultáveis.

**Fluxo do usuário:**
1.  Usuário acessa `/settings`.
2.  Navega entre as abas para configurar perfil, empresa, usuários ou preferências.
3.  `owner`/`admin` convida novos usuários ou gerencia os existentes.

**Permissões:**
*   `owner`: Acesso total a todas as configurações.
*   `admin`: Acesso a configurações da empresa e usuários (exceto exclusão de owner), mas não pode alterar plano de assinatura.
*   `employee`: Apenas acesso à sub-seseção de Perfil (`/settings/profile`).

**Integrações com Supabase:**
*   `profiles` (CRUD para perfil)
*   `tenants` (CRUD para configurações da empresa)
*   `tenant_users` (CRUD para usuários do tenant)
*   `settings` (CRUD para preferências e outras configurações)
*   Supabase Storage (para upload de logo)
*   Supabase Auth (para convite de usuários)

**Integrações com Stripe:** Nenhuma direta.

**Critérios de aceite:**
*   Todas as sub-seções de configurações funcionam.
*   Salvar alterações funciona com validações e feedback.
*   Convite e gerenciamento de usuários funcionam.
*   Upload de logo funciona.
*   Permissões de acesso são respeitadas.

### 3.11 Notificações

**Objetivo:** Exibir uma lista de notificações para o usuário, permitindo que ele as marque como lidas ou as arquive.

**Rota:** `/notifications`

**Perfis com acesso:** `owner`, `admin`, `employee`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem de notificações)
*   `Button` (para marcar como lida, arquivar, excluir)
*   `StatusBadge` (para status da notificação)
*   `EmptyState`
*   `Skeleton`

**Campos (Listagem):**
*   Tipo (`notification_type`)
*   Título
*   Mensagem (truncada)
*   Data
*   Status (`notification_status`)

**Botões:**
*   **Marcar como Lida:** Na linha da notificação.
*   **Arquivar:** Na linha da notificação.
*   **Marcar todas como lidas:** Botão global.

**Estados de loading:**
*   `Skeleton` na `DataTable`.

**Empty states:**
*   "Nenhuma notificação para exibir."

**Mensagens de erro:**
*   "Não foi possível carregar as notificações. Tente novamente."

**Validações:** Nenhuma direta.

**Responsividade:** Tabela com colunas ocultáveis.

**Fluxo do usuário:**
1.  Usuário acessa `/notifications`.
2.  Visualiza a lista de notificações.
3.  Pode interagir com as notificações (marcar como lida, arquivar).

**Permissões:**
*   Todos os perfis podem visualizar e gerenciar suas próprias notificações.
*   `owner`, `admin` podem visualizar e gerenciar notificações de outros usuários do tenant (para fins de suporte/auditoria).

**Integrações com Supabase:**
*   `notifications` (leitura e atualização de status).

**Integrações com Stripe:** Nenhuma direta.

**Critérios de aceite:**
*   Listagem de notificações funciona.
*   Marcar como lida e arquivar funcionam.
*   Permissões de acesso são respeitadas.

### 3.12 Auditoria

**Objetivo:** Exibir um log de auditoria das ações importantes realizadas no sistema, para rastreabilidade e segurança.

**Rota:** `/audit-logs`

**Perfis com acesso:** `owner`, `admin`.

**Componentes utilizados:**
*   `Header`
*   `Sidebar`
*   `DataTable` (para listagem de logs)
*   `SearchBar`, `Select`, `DatePicker` (para filtros)
*   `EmptyState`
*   `Skeleton`

**Campos (Listagem):**
*   Data/Hora
*   Usuário
*   Ação
*   Entidade
*   ID da Entidade
*   IP

**Botões:**
*   Botões de filtro por data, usuário, ação, entidade.

**Estados de loading:**
*   `Skeleton` na `DataTable`.

**Empty states:**
*   "Nenhum log de auditoria encontrado para os critérios selecionados."

**Mensagens de erro:**
*   "Não foi possível carregar os logs de auditoria. Tente novamente."

**Validações:** Nenhuma direta.

**Responsividade:** Tabela com colunas ocultáveis.

**Fluxo do usuário:**
1.  `owner`/`admin` acessa `/audit-logs`.
2.  Visualiza a lista de logs, pode filtrar para encontrar ações específicas.

**Permissões:**
*   `owner`, `admin`: Acesso total aos logs de auditoria do seu tenant.
*   `employee`: Nenhuma permissão para esta seção.

**Integrações com Supabase:**
*   `audit_logs` (leitura).

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Listagem de logs funciona com filtros e paginação.
*   Informações do log são claras e úteis.
*   Permissões de acesso são respeitadas.

### 3.13 Página 404 (Não Encontrado)

**Objetivo:** Informar ao usuário que a página solicitada não foi encontrada.

**Rota:** Qualquer rota não existente.

**Perfis com acesso:** Todos.

**Componentes utilizados:**
*   Layout básico da aplicação (Header/Sidebar, se aplicável).
*   `Card` ou `div` centralizado.
*   `Button` (para voltar ao Dashboard).

**Campos:**
*   Título: "404 - Página Não Encontrada"
*   Mensagem: "A página que você está procurando não existe ou foi movida."

**Botões:**
*   **Voltar para o Dashboard:** Redireciona para `/dashboard`.

**Estados de loading:** Não aplicável.

**Empty states:** Não aplicável.

**Mensagens de erro:** Não aplicável.

**Validações:** Não aplicável.

**Responsividade:** Layout centralizado e responsivo.

**Fluxo do usuário:**
1.  Usuário tenta acessar uma URL inválida.
2.  É redirecionado para a página 404.
3.  Pode clicar para voltar ao dashboard.

**Permissões:** Acesso público.

**Integrações com Supabase:** Nenhuma.

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Página 404 é exibida para rotas inválidas.
*   Botão "Voltar para o Dashboard" funciona.

### 3.14 Página 500 (Erro Interno do Servidor)

**Objetivo:** Informar ao usuário que ocorreu um erro inesperado no servidor.

**Rota:** Erros internos do servidor.

**Perfis com acesso:** Todos.

**Componentes utilizados:**
*   Layout básico da aplicação.
*   `Card` ou `div` centralizado.
*   `Button` (para voltar ao Dashboard).

**Campos:**
*   Título: "500 - Erro Interno do Servidor"
*   Mensagem: "Ocorreu um erro inesperado. Nossa equipe já foi notificada. Por favor, tente novamente mais tarde ou entre em contato com o suporte."

**Botões:**
*   **Voltar para o Dashboard:** Redireciona para `/dashboard`.

**Estados de loading:** Não aplicável.

**Empty states:** Não aplicável.

**Mensagens de erro:** Não aplicável.

**Validações:** Não aplicável.

**Responsividade:** Layout centralizado e responsivo.

**Fluxo do usuário:**
1.  Ocorre um erro fatal no servidor.
2.  Usuário é redirecionado para a página 500.
3.  Pode clicar para voltar ao dashboard.

**Permissões:** Acesso público.

**Integrações com Supabase:** Nenhuma direta, mas erros podem ser logados no backend.

**Integrações com Stripe:** Nenhuma.

**Critérios de aceite:**
*   Página 500 é exibida para erros internos.
*   Botão "Voltar para o Dashboard" funciona.

## 4. Estados Globais e Feedback ao Usuário

Para garantir uma experiência de usuário fluida e informativa, o sistema implementará estados globais de feedback e carregamento.

### 4.1 Loading

**Objetivo:** Indicar que uma operação está em andamento e o usuário deve aguardar.

*   **Implementação:** Spinners em botões, indicadores de progresso em formulários, `Skeleton` loaders em listas e dashboards.
*   **Contexto:** Usado em submissões de formulários, carregamento de dados em listas, gráficos e dashboards.

### 4.2 Skeletons

**Objetivo:** Fornecer um feedback visual de que o conteúdo está sendo carregado, preenchendo o layout com placeholders antes que os dados reais cheguem.

*   **Implementação:** Componentes `Skeleton` do `shadcn/ui` ou customizados.
*   **Contexto:** Listagens (`DataTable`), `KPICard`, `MetricCard`, `Charts`, e outras seções com carregamento assíncrono de dados.

### 4.3 Empty States

**Objetivo:** Informar ao usuário quando não há dados para exibir em uma determinada seção, oferecendo uma ação clara para preencher essa lacuna.

*   **Implementação:** Componente `EmptyState` customizado com ícone, título, descrição e botão de ação (ex: "Adicionar Cliente").
*   **Contexto:** Listagens vazias (Clientes, Veículos, OS, Produtos, Serviços, Notificações, Logs de Auditoria).

### 4.4 Toasts

**Objetivo:** Exibir mensagens de feedback rápidas e não intrusivas para o usuário, informando sobre o sucesso ou falha de uma operação.

*   **Implementação:** Componente `Toast` do `shadcn/ui`.
*   **Tipos:** Sucesso, Erro, Aviso, Informação.
*   **Contexto:** Após salvar um formulário, excluir um item, convidar um usuário, etc.

### 4.5 Modais

**Objetivo:** Exibir conteúdo ou formulários em uma janela sobreposta à página principal, exigindo interação do usuário.

*   **Implementação:** Componente `Dialog` do `shadcn/ui`.
*   **Contexto:** Formulários de cadastro/edição (clientes, veículos, serviços, produtos), confirmações (`ConfirmDialog`), seleção de itens.

### 4.6 Drawers

**Objetivo:** Similar aos modais, mas deslizam da lateral da tela, ideal para formulários mais complexos ou detalhes que precisam de mais espaço.

*   **Implementação:** Componente `Drawer` (se disponível no `shadcn/ui` ou customizado).
*   **Contexto:** Alternativa aos modais para formulários de cadastro/edição, filtros avançados.

### 4.7 Confirmações

**Objetivo:** Solicitar confirmação do usuário antes de executar ações destrutivas ou irreversíveis.

*   **Implementação:** Componente `ConfirmDialog` (baseado em `AlertDialog` do `shadcn/ui`).
*   **Contexto:** Exclusão de registros (clientes, veículos, produtos, serviços), cancelamento de assinatura, finalização de OS.

## Conclusão

Este documento fornece uma especificação técnica abrangente para o frontend do OficinaPro, cobrindo a estrutura do projeto, componentes reutilizáveis, e o detalhamento de cada página do sistema. A aderência a esta especificação garantirá que o desenvolvimento do frontend seja consistente, eficiente e resulte em uma experiência de usuário premium, alinhada com as melhores práticas de UI/UX e as referências de design estabelecidas. Com esta base, as ferramentas de IA (Cursor, Claude, ChatGPT) e os desenvolvedores terão todas as informações necessárias para construir a interface do OficinaPro com precisão e qualidade.
