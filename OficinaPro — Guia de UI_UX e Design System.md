# OficinaPro — Guia de UI/UX e Design System

**Autor:** Manus AI  
**Data:** 09 de junho de 2026  
**Versão:** 1.0  
**Finalidade:** Este documento detalha a identidade visual, os princípios de design, os componentes de UI/UX e os wireframes das telas principais do sistema OficinaPro. Ele serve como uma especificação completa para o desenvolvimento da interface do usuário, garantindo consistência e uma experiência premium, moderna e limpa, inspirada em plataformas como Stripe, Linear, Vercel, Notion e HubSpot.

## 1. Identidade Visual

A identidade visual do OficinaPro é construída a partir da logo fornecida, que apresenta um texto em tom de cinza escuro e um elemento de destaque em azul vibrante. O objetivo é transmitir profissionalismo, modernidade e eficiência, adequados a um SaaS de gestão para oficinas mecânicas.

### 1.1 Cores

As cores do sistema serão derivadas da logo, complementadas por uma paleta neutra e cores semânticas para feedback (sucesso, alerta, erro). A inspiração em interfaces como Stripe e Vercel sugere um uso estratégico de cores para guiar o usuário e destacar informações importantes, mantendo a sobriedade geral.

#### 1.1.1 Cores Principais

| Nome | Hexadecimal | RGB | Uso |
|---|---|---|---|
| **Primária Azul** | `#3366ff` | `rgb(51, 102, 255)` | Elementos interativos principais (botões primários, links, ícones de destaque, indicadores de progresso, acentos gráficos). |
| **Primária Escura** | `#201e1e` | `rgb(32, 30, 30)` | Texto principal, fundos de elementos escuros (tema escuro), elementos de navegação. |

#### 1.1.2 Cores Secundárias

As cores secundárias serão variações do azul primário e tons de cinza para criar profundidade e hierarquia visual, além de um tom de verde para representar sucesso e um tom de vermelho para erros.

| Nome | Hexadecimal | RGB | Uso |
|---|---|---|---|
| **Azul Claro** | `#6699ff` | `rgb(102, 153, 255)` | Estados de hover/active para elementos primários, backgrounds sutis. |
| **Azul Escuro** | `#0033cc` | `rgb(0, 51, 204)` | Texto em destaque sobre fundos claros, elementos de foco. |

#### 1.1.3 Paleta Neutra

A paleta neutra é essencial para a legibilidade e para criar um ambiente visual limpo, inspirada na estética minimalista de Notion e Linear.

| Nome | Hexadecimal | RGB | Uso |
|---|---|---|---|
| **Preto (Fundo)** | `#000000` | `rgb(0, 0, 0)` | Fundo principal do tema escuro. |
| **Cinza Muito Escuro** | `#1a1a1a` | `rgb(26, 26, 26)` | Elementos de interface no tema escuro, texto secundário. |
| **Cinza Escuro** | `#333333` | `rgb(51, 51, 51)` | Texto padrão, bordas de elementos, ícones. |
| **Cinza Médio** | `#666666` | `rgb(102, 102, 102)` | Texto auxiliar, placeholders, divisores. |
| **Cinza Claro** | `#cccccc` | `rgb(204, 204, 204)` | Bordas de inputs, backgrounds de elementos desabilitados. |
| **Cinza Muito Claro** | `#f0f0f0` | `rgb(240, 240, 240)` | Backgrounds de seções, estados de hover/active sutis no tema claro. |
| **Branco** | `#ffffff` | `rgb(255, 255, 255)` | Fundo principal do tema claro, texto sobre fundos escuros. |

#### 1.1.4 Cores Semânticas (Feedback)

| Nome | Hexadecimal | RGB | Uso |
|---|---|---|---|
| **Sucesso** | `#28a745` | `rgb(40, 167, 69)` | Mensagens de sucesso, ícones de confirmação. |
| **Alerta** | `#ffc107` | `rgb(255, 193, 7)` | Mensagens de aviso, ícones de atenção. |
| **Erro** | `#dc3545` | `rgb(220, 53, 69)` | Mensagens de erro, validações negativas, ícones de falha. |

### 1.2 Temas

O sistema oferecerá suporte a tema claro e tema escuro, com a possibilidade de o usuário alternar entre eles. O tema escuro será baseado no preto da logo e nos tons de cinza escuro, enquanto o tema claro utilizará o branco e tons de cinza claro como base.

| Elemento | Tema Claro | Tema Escuro |
|---|---|---|
| Fundo principal | `#ffffff` (Branco) | `#000000` (Preto) |
| Texto principal | `#333333` (Cinza Escuro) | `#f0f0f0` (Cinza Muito Claro) |
| Texto secundário | `#666666` (Cinza Médio) | `#cccccc` (Cinza Claro) |
| Botão primário | Fundo: `#3366ff`, Texto: `#ffffff` | Fundo: `#3366ff`, Texto: `#ffffff` |
| Bordas | `#cccccc` (Cinza Claro) | `#333333` (Cinza Escuro) |

### 1.3 Tipografia

A fonte **Inter** será utilizada em todo o sistema, conhecida por sua legibilidade em telas e sua estética moderna e neutra, alinhada com as referências de design. Serão definidas escalas de tamanho e peso para títulos, subtítulos, texto padrão, tabelas e botões.

| Elemento | Família | Peso | Tamanho (px) | Line-height (rem) | Uso |
|---|---|---|---|---|---|
| **Título 1 (H1)** | Inter | Bold (700) | 36 | 2.5 | Títulos de página principal, dashboards. |
| **Título 2 (H2)** | Inter | SemiBold (600) | 28 | 2 | Títulos de seção, cabeçalhos de módulos. |
| **Título 3 (H3)** | Inter | SemiBold (600) | 22 | 1.75 | Subtítulos importantes, cards. |
| **Subtítulo** | Inter | Medium (500) | 18 | 1.5 | Descrições curtas, cabeçalhos de tabelas. |
| **Texto Padrão** | Inter | Regular (400) | 16 | 1.5 | Corpo de texto, parágrafos, labels. |
| **Texto Pequeno** | Inter | Regular (400) | 14 | 1.25 | Textos auxiliares, legendas, tooltips. |
| **Tabelas (Cabeçalho)** | Inter | Medium (500) | 14 | 1.25 | Cabeçalhos de colunas. |
| **Tabelas (Célula)** | Inter | Regular (400) | 14 | 1.25 | Conteúdo das células. |
| **Botões (Primário)** | Inter | SemiBold (600) | 16 | 1.5 | Botões de ação principal. |
| **Botões (Secundário)** | Inter | Medium (500) | 14 | 1.25 | Botões de ação secundária. |

## 2. Layout Geral

O layout do OficinaPro será responsivo e adaptável a diferentes tamanhos de tela, com uma estrutura de navegação clara e consistente. A inspiração em Linear e Vercel sugere um layout com sidebar lateral e topbar, otimizando o espaço para o conteúdo principal.

### 2.1 Sidebar Lateral Recolhível

A sidebar lateral abrigará a navegação principal do sistema. Ela será recolhível para otimizar o espaço em telas menores e permitir que o usuário se concentre no conteúdo. No estado recolhido, apenas ícones serão visíveis, expandindo para mostrar os rótulos ao passar o mouse ou clicar.

| Elemento | Descrição | Comportamento |
|---|---|---|
| **Logo do OficinaPro** | No topo da sidebar. | Sempre visível, link para o dashboard. |
| **Itens de Navegação** | Ícones e rótulos para módulos (Dashboard, Clientes, Veículos, OS, Financeiro, etc.). | Estado ativo destacado com a cor Primária Azul. |
| **Alternador de Tema** | Ícone para alternar entre tema claro e escuro. | Posicionado na parte inferior da sidebar. |
| **Botão de Recolher/Expandir** | Ícone para controlar o estado da sidebar. | Recolhe a sidebar para mostrar apenas ícones. |

### 2.2 Topbar

A topbar, ou barra superior, conterá elementos de contexto e ações rápidas, como o nome da oficina ativa, breadcrumbs, notificações e o menu do usuário.

| Elemento | Descrição | Comportamento |
|---|---|---|
| **Nome da Oficina Ativa** | Exibe o nome da oficina que o usuário está gerenciando. | Clicável para alternar entre tenants (se o usuário tiver múltiplos). |
| **Breadcrumbs** | Indica a localização atual do usuário na hierarquia do sistema. | Navegação auxiliar. |
| **Notificações** | Ícone de sino com contador para novas notificações. | Abre um drawer ou dropdown com a lista de notificações. |
| **Menu do Usuário** | Avatar do usuário com dropdown para perfil, configurações e logout. | Acesso rápido a ações relacionadas ao usuário. |

### 2.3 Cards

Cards serão utilizados para agrupar informações relacionadas, como KPIs no dashboard, detalhes de clientes ou veículos. Eles terão bordas sutis e sombras leves para criar profundidade.

| Elemento | Descrição | Estilo |
|---|---|---|
| **Título do Card** | H3 ou Subtítulo. | Inter SemiBold, cor Primária Escura/Cinza Muito Claro. |
| **Conteúdo** | Texto padrão, listas, gráficos pequenos. | Inter Regular, cores da paleta neutra. |
| **Ações** | Botões ou links no rodapé do card. | Botões secundários ou links. |

### 2.4 KPIs (Key Performance Indicators)

KPIs serão apresentados em cards dedicados no dashboard, com números grandes e claros, acompanhados de um rótulo e, opcionalmente, um indicador de variação.

| Elemento | Descrição | Estilo |
|---|---|---|
| **Valor Principal** | Número grande e negrito. | Inter Bold, tamanho H1/H2, cor Primária Escura/Cinza Muito Claro. |
| **Rótulo** | Descrição do KPI. | Inter Regular, Texto Pequeno, Cinza Médio/Cinza Claro. |
| **Variação (opcional)** | Indicador de crescimento/queda. | Texto Pequeno, cor Sucesso/Erro. |

### 2.5 Dashboard

O dashboard será a tela inicial após o login, oferecendo uma visão geral do negócio. Será composto por uma grade de cards de KPIs, gráficos e atalhos rápidos, com layout responsivo.

| Seção | Conteúdo | Prioridade |
|---|---|---|
| **Visão Geral** | Cards de Faturamento, OS Abertas, Contas a Receber, Contas a Pagar. | Alta |
| **Gráficos** | Gráfico de linha de Receitas vs Despesas, Gráfico de barras de OS por Status. | Média |
| **Atalhos Rápidos** | Botões para "Nova OS", "Novo Cliente", "Novo Veículo". | Alta |
| **Atividades Recentes** | Lista de últimas OS, pagamentos, etc. | Média |

### 2.6 Breadcrumbs

Os breadcrumbs serão utilizados para indicar a navegação e permitir que o usuário retorne facilmente a níveis anteriores. Serão discretos, com texto em Cinza Médio e links em Primária Azul.

| Elemento | Descrição | Estilo |
|---|---|---|
| **Item de Navegação** | Nome da página ou seção. | Inter Regular, Texto Pequeno, Cinza Médio. |
| **Link Ativo** | Página atual. | Inter SemiBold, Texto Pequeno, Primária Escura/Cinza Muito Claro. |
| **Separador** | Ícone de seta ou barra. | Cinza Médio. |

## 3. Componentes (shadcn/ui)

Todos os componentes serão implementados utilizando shadcn/ui, que oferece uma base sólida de componentes acessíveis e estilizados com Tailwind CSS. A customização seguirá a identidade visual definida, garantindo consistência e um visual premium.

### 3.1 Botões

| Tipo | Aparência | Comportamento | Estado |
|---|---|---|---|
| **Primário** | Fundo: Primária Azul, Texto: Branco. | Clique: feedback visual (sombra, leve escala). | Hover: Azul Claro. Disabled: Opacidade reduzida. |
| **Secundário** | Fundo: Cinza Muito Claro/Cinza Muito Escuro (tema escuro), Texto: Primária Escura/Cinza Muito Claro. | Clique: feedback visual. | Hover: Cinza Claro/Cinza Escuro. Disabled: Opacidade reduzida. |
| **Outline** | Fundo: Transparente, Borda: Primária Azul, Texto: Primária Azul. | Clique: feedback visual. | Hover: Fundo Azul Claro. Disabled: Opacidade reduzida. |
| **Ghost** | Fundo: Transparente, Texto: Primária Escura/Cinza Muito Claro. | Clique: feedback visual. | Hover: Cinza Muito Claro/Cinza Muito Escuro. Disabled: Opacidade reduzida. |
| **Link** | Fundo: Transparente, Texto: Primária Azul, Sublinhado. | Clique: feedback visual. | Hover: Sublinhado mais forte. Disabled: Opacidade reduzida. |

### 3.2 Inputs (Text, Number, Email, Password)

| Aparência | Comportamento | Estado |
|---|---|---|---|
| **Padrão** | Borda: Cinza Claro, Fundo: Branco/Cinza Muito Escuro, Texto: Primária Escura/Cinza Muito Claro. | Foco: Borda Primária Azul, sombra sutil. | Erro: Borda Erro, mensagem de erro abaixo. Disabled: Fundo Cinza Muito Claro/Cinza Escuro, texto Cinza Médio/Cinza Claro. |
| **Label** | Acima do input, Inter SemiBold, Texto Pequeno. | Sempre visível. | |
| **Placeholder** | Texto em Cinza Médio/Cinza Claro. | Desaparece ao digitar. | |

### 3.3 Selects (Dropdowns)

| Aparência | Comportamento | Estado |
|---|---|---|---|
| **Padrão** | Similar ao input, com ícone de seta para baixo. | Clique: Abre lista de opções. | Foco: Borda Primária Azul. Erro: Borda Erro. Disabled: Opacidade reduzida. |
| **Opções** | Lista suspensa com fundo Branco/Cinza Muito Escuro, texto Primária Escura/Cinza Muito Claro. | Hover: Fundo Cinza Muito Claro/Cinza Escuro. Selecionado: Ícone de check, texto Primária Azul. |

### 3.4 Tabelas

As tabelas serão limpas e legíveis, com linhas divisórias sutis e destaque para a linha selecionada.

| Elemento | Aparência | Comportamento |
|---|---|---|
| **Cabeçalho** | Fundo: Cinza Muito Claro/Cinza Muito Escuro, Texto: Subtítulo, Primária Escura/Cinza Muito Claro. | Colunas clicáveis para ordenação (ícone de seta). |
| **Linhas** | Fundo: Branco/Preto, Texto: Texto Padrão. | Hover: Fundo Cinza Muito Claro/Cinza Muito Escuro. Selecionado: Fundo Azul Claro com borda esquerda Primária Azul. |
| **Paginação** | Botões de navegação (anterior, próximo, números de página). | Padrão shadcn/ui. |

### 3.5 Modais

Modais serão usados para interações que exigem foco total do usuário, como confirmações ou formulários complexos. Serão centralizados, com overlay escuro e fundo branco/cinza muito escuro.

| Elemento | Aparência | Comportamento |
|---|---|---|
| **Overlay** | Fundo preto com 50% de opacidade. | Clicar fora fecha o modal (se não for crítico). |
| **Conteúdo** | Fundo Branco/Cinza Muito Escuro, bordas arredondadas, sombra. | Título (H2/H3), corpo de texto, botões de ação (Primário/Secundário). |

### 3.6 Drawers

Drawers serão utilizados para formulários secundários ou painéis de detalhes que não exigem interrupção total do fluxo. Eles deslizarão da lateral (direita ou esquerda).

| Elemento | Aparência | Comportamento |
|---|---|---|---|
| **Overlay** | Fundo preto com 30% de opacidade. | Clicar fora fecha o drawer. |
| **Conteúdo** | Fundo Branco/Cinza Muito Escuro, desliza da lateral, largura configurável. | Título (H2/H3), formulário, botões de ação. |

### 3.7 Tabs

Tabs serão usadas para organizar conteúdo em seções dentro de uma mesma tela, como nas abas de uma Ordem de Serviço.

| Aparência | Comportamento | Estado |
|---|---|---|---|
| **Padrão** | Texto Cinza Médio/Cinza Claro. | Clique: Ativa a aba. |
| **Ativa** | Texto Primária Escura/Cinza Muito Claro, borda inferior Primária Azul. | Destacada. |

### 3.8 Badges

Badges serão usados para exibir status ou categorias de forma concisa, como o status de uma OS ou de um cliente.

| Tipo | Aparência | Uso |
|---|---|---|
| **Padrão** | Fundo Cinza Muito Claro/Cinza Muito Escuro, Texto Cinza Escuro/Cinza Claro. | Status genéricos. |
| **Sucesso** | Fundo Verde Sucesso (claro), Texto Verde Sucesso (escuro). | Status 
de sucesso (Ex: OS Finalizada, Cliente Ativo). |
| **Alerta** | Fundo Amarelo Alerta (claro), Texto Amarelo Alerta (escuro). | Status de aviso (Ex: OS Aguardando Aprovação, Conta Vencendo). |
| **Erro** | Fundo Vermelho Erro (claro), Texto Vermelho Erro (escuro). | Status de erro (Ex: OS Cancelada, Assinatura Inativa). |
| **Primário** | Fundo Primária Azul (claro), Texto Primária Azul (escuro). | Destaque para informações importantes. |

### 3.9 Toasts

Toasts serão mensagens de feedback não intrusivas que aparecem temporariamente no canto da tela para informar o usuário sobre o resultado de uma ação (sucesso, erro, informação).

| Tipo | Aparência | Comportamento |
|---|---|---|---|
| **Sucesso** | Fundo Verde Sucesso, Ícone de check, Texto Branco. | Desaparece automaticamente após 3-5 segundos. |
| **Erro** | Fundo Vermelho Erro, Ícone de X, Texto Branco. | Desaparece automaticamente ou pode ser fechado manualmente. |
| **Informação** | Fundo Cinza Escuro, Ícone de informação, Texto Branco. | Desaparece automaticamente. |

### 3.10 Dropdowns

Dropdowns serão usados para menus de contexto, seleção de opções ou ações secundárias, como o menu do usuário na topbar.

| Elemento | Aparência | Comportamento |
|---|---|---|---|
| **Gatilho** | Botão ou ícone. | Clique abre/fecha o menu. |
| **Menu** | Fundo Branco/Cinza Muito Escuro, bordas arredondadas, sombra. | Itens de menu com hover sutil. |
| **Itens** | Texto Padrão, Primária Escura/Cinza Muito Claro. | Hover: Fundo Cinza Muito Claro/Cinza Muito Escuro. |

### 3.11 Date Picker

O Date Picker será um componente para seleção de datas, integrado a inputs de formulário.

| Elemento | Aparência | Comportamento |
|---|---|---|---|
| **Input** | Padrão de input de texto, com ícone de calendário. | Clique no input ou ícone abre o calendário. |
| **Calendário** | Modal ou popover com navegação por mês/ano. | Seleção de dia destaca com Primária Azul. |

### 3.12 Loading States

Estados de carregamento serão visuais e informativos, indicando que o sistema está processando uma requisição.

| Tipo | Aparência | Uso |
|---|---|---|---|
| **Spinner** | Ícone giratório (Primária Azul). | Carregamento de dados em componentes específicos. |
| **Barra de Progresso** | Barra horizontal (Primária Azul). | Carregamento de página ou envio de formulário. |
| **Overlay de Carregamento** | Fundo semitransparente com spinner centralizado. | Carregamento de tela inteira ou modal. |

### 3.13 Skeletons

Skeletons serão usados para melhorar a percepção de performance durante o carregamento de conteúdo, exibindo um layout temporário que simula a estrutura da informação que está sendo carregada.

| Elemento | Aparência | Uso |
|---|---|---|
| **Linhas de Texto** | Retângulos cinzas de diferentes larguras. | Simular carregamento de texto em cards, tabelas. |
| **Imagens** | Retângulos cinzas com bordas arredondadas. | Simular carregamento de avatares, logos. |
| **Cards/Tabelas** | Estrutura completa do componente em tons de cinza. | Carregamento inicial de listas e dashboards. |

### 3.14 Empty States

Empty states serão telas ou componentes que informam ao usuário quando não há dados para exibir, oferecendo orientações claras sobre como preencher ou iniciar o uso.

| Elemento | Aparência | Uso |
|---|---|---|
| **Ícone Ilustrativo** | Ícone grande e relevante (Cinza Médio). | Em telas de lista vazias (Ex: Nenhum cliente cadastrado). |
| **Título** | H3, Cinza Escuro/Cinza Muito Claro. | Mensagem clara sobre a ausência de dados. |
| **Descrição** | Texto Padrão, Cinza Médio/Cinza Claro. | Explicação e orientação. |
| **Botão de Ação** | Botão Primário. | Atalho para criar o primeiro item (Ex: Cadastrar primeiro cliente). |

## 4. Telas do Sistema

As telas do sistema serão projetadas com foco na usabilidade, clareza e eficiência, seguindo os princípios de design definidos. Serão apresentados wireframes conceituais e descrições da experiência do usuário para cada tela principal.

### 4.1 Login

**Objetivo:** Permitir que o usuário acesse o sistema com suas credenciais.

**Experiência do Usuário:** Uma tela limpa e minimalista, com o logo do OficinaPro centralizado. Campos de e-mail e senha bem definidos, botão de login em destaque e links para recuperação de senha e cadastro. A validação deve ser em tempo real ou após o envio do formulário, com mensagens claras de erro.

**Componentes:**
*   **Logo:** OficinaPro (texto Cinza Escuro, barra Azul Primária).
*   **Título:** 
H2 "Bem-vindo de volta!".
*   **Inputs:** E-mail e Senha (com ícone de olho para mostrar/esconder senha).
*   **Botões:** Botão Primário "Entrar", Botão Ghost "Esqueci minha senha", Botão Ghost "Criar conta".
*   **Validações:** Mensagens de erro claras para credenciais inválidas ou campos vazios.

### 4.2 Cadastro

**Objetivo:** Permitir que novos usuários criem uma conta e iniciem o processo de cadastro da oficina.

**Experiência do Usuário:** Similar à tela de login, mas com campos adicionais para nome e confirmação de senha. Deve incluir um checkbox para aceite dos termos de uso e política de privacidade. Após o cadastro, o usuário deve ser informado sobre a necessidade de confirmação por e-mail.

**Componentes:**
*   **Logo:** OficinaPro.
*   **Título:** H2 "Crie sua conta".
*   **Inputs:** Nome completo, E-mail, Senha, Confirmar Senha.
*   **Checkbox:** "Concordo com os Termos de Uso e Política de Privacidade".
*   **Botões:** Botão Primário "Cadastrar", Botão Ghost "Já tenho conta".
*   **Validações:** Senha forte, senhas coincidentes, e-mail válido, aceite de termos.

### 4.3 Dashboard

**Objetivo:** Fornecer uma visão geral e rápida do desempenho da oficina, com acesso a indicadores chave e atalhos para ações comuns.

**Experiência do Usuário:** Um layout organizado em cards, com KPIs em destaque no topo, seguidos por gráficos de tendência e listas de atividades recentes. Atalhos rápidos para as ações mais frequentes devem estar visíveis. A responsividade é crucial para garantir a usabilidade em diferentes dispositivos.

**Componentes:**
*   **Topbar:** Nome da Oficina, Breadcrumbs (Dashboard), Notificações, Menu do Usuário.
*   **Sidebar:** Navegação principal.
*   **Cards de KPIs:** Faturamento, OS Abertas, Contas a Receber, Contas a Pagar (com valores e indicadores de variação).
*   **Gráficos:** Gráfico de linha (Receitas vs Despesas por período), Gráfico de barras (OS por Status).
*   **Atalhos Rápidos:** Botões Primários/Secundários para "Nova Ordem de Serviço", "Novo Cliente", "Novo Veículo".
*   **Tabela de Atividades Recentes:** Últimas OS, últimos pagamentos.
*   **Empty States:** Mensagem e botão para iniciar o uso caso não haja dados.

### 4.4 Clientes

**Objetivo:** Gerenciar o cadastro de clientes da oficina, incluindo suas informações pessoais, de contato e histórico.

**Experiência do Usuário:** Uma tela de listagem com barra de busca e filtros para facilitar a localização de clientes. Cada cliente na lista deve ter opções rápidas para visualizar detalhes, editar ou inativar. O formulário de cadastro/edição deve ser claro e organizado.

**Componentes:**
*   **Topbar e Sidebar.**
*   **Cabeçalho da Página:** Título H1 "Clientes", Botão Primário "Novo Cliente".
*   **Barra de Busca:** Input de texto com ícone de lupa.
*   **Filtros:** Dropdowns para "Status" (Ativo, Inativo) e outros critérios.
*   **Tabela de Clientes:** Colunas para Nome, Documento, Telefone, WhatsApp, Status. Ações (Editar, Visualizar, Inativar/Ativar) em cada linha.
*   **Paginação.**
*   **Modal/Drawer de Cadastro/Edição:** Formulário com Inputs para Nome/Razão Social, Tipo de Cliente, CPF/CNPJ, E-mail, Telefone, WhatsApp, Data de Nascimento, Endereço (campos separados), Observações. Botões "Salvar" e "Cancelar".
*   **Empty State:** Ícone ilustrativo, mensagem "Nenhum cliente cadastrado ainda.", Botão Primário "Cadastrar primeiro cliente".

### 4.5 Veículos

**Objetivo:** Gerenciar o cadastro de veículos, vinculando-os aos clientes e registrando suas características.

**Experiência do Usuário:** Listagem de veículos com busca por placa, marca, modelo ou cliente. Opções para editar, visualizar detalhes ou abrir uma nova Ordem de Serviço diretamente do veículo. O formulário de cadastro/edição deve ser detalhado, mas intuitivo.

**Componentes:**
*   **Topbar e Sidebar.**
*   **Cabeçalho da Página:** Título H1 "Veículos", Botão Primário "Novo Veículo".
*   **Barra de Busca:** Input de texto.
*   **Filtros:** Dropdowns para "Marca", "Modelo", "Status".
*   **Tabela de Veículos:** Colunas para Placa, Cliente, Marca, Modelo, Ano, KM, Status. Ações (Editar, Visualizar, Abrir OS).
*   **Paginação.**
*   **Modal/Drawer de Cadastro/Edição:** Formulário com Inputs para Cliente (Select com busca), Placa, Marca, Modelo, Ano Fabricação, Ano Modelo, Cor, Renavam, Chassi, Quilometragem Atual, Tipo de Combustível, Observações. Botões "Salvar" e "Cancelar".
*   **Empty State:** Ícone ilustrativo, mensagem "Nenhum veículo cadastrado ainda.", Botão Primário "Cadastrar primeiro veículo".

### 4.6 Ordem de Serviço

**Objetivo:** Gerenciar o ciclo de vida completo de uma Ordem de Serviço, desde a abertura até a finalização.

**Experiência do Usuário:** Uma tela de listagem com filtros avançados por status, cliente, veículo e datas. A tela de detalhes da OS deve ser organizada em abas para facilitar a navegação entre as diferentes seções (dados gerais, diagnóstico, serviços, produtos, financeiro, histórico). Botões de ação contextualizados com o status atual da OS.

**Componentes:**
*   **Topbar e Sidebar.**
*   **Cabeçalho da Página:** Título H1 "Ordens de Serviço", Botão Primário "Nova OS".
*   **Barra de Busca:** Input de texto (por número da OS, cliente, placa).
*   **Filtros:** Dropdowns para "Status" (Draft, Aberta, Diagnóstico, etc.), "Cliente", "Veículo", "Período".
*   **Tabela de Ordens de Serviço:** Colunas para Número, Cliente, Veículo (Placa), Status, Data Abertura, Previsão Entrega, Valor Total. Ações (Visualizar, Editar, Mudar Status).
*   **Paginação.**
*   **Tela de Detalhes da OS (com Tabs):**
    *   **Aba "Geral":** Inputs para Cliente (Select), Veículo (Select), Status (Badge), Relato do Cliente (Textarea), Quilometragem Entrada, Previsão de Entrega (Date Picker), Responsável Técnico (Select).
    *   **Aba "Diagnóstico":** Textarea para Diagnóstico Técnico, Observações Internas.
    *   **Aba "Serviços":** Tabela de `work_order_services` com Inputs para Serviço (Select com busca), Quantidade, Preço Unitário, Desconto. Botão "Adicionar Serviço".
    *   **Aba "Produtos":** Tabela de `work_order_products` com Inputs para Produto (Select com busca), Quantidade, Preço Unitário, Desconto. Botão "Adicionar Produto".
    *   **Aba "Financeiro":** Resumo de Totais (Serviços, Produtos, Descontos, Acréscimos, Total Geral). Tabela de Contas a Receber vinculadas. Botão "Registrar Pagamento".
    *   **Aba "Histórico":** Tabela de `work_order_status_history` com Data, Usuário, Status Anterior, Novo Status, Justificativa.
*   **Botões de Ação na OS:** Contextuais ao status (Ex: "Salvar Rascunho", "Enviar para Diagnóstico", "Aprovar OS", "Finalizar OS", "Cancelar OS", "Reabrir OS").
*   **Empty State:** Ícone ilustrativo, mensagem "Nenhuma Ordem de Serviço criada ainda.", Botão Primário "Criar primeira OS".

### 4.7 Financeiro

**Objetivo:** Gerenciar as finanças da oficina, incluindo receitas, despesas, contas a receber e a pagar.

**Experiência do Usuário:** Uma interface clara com abas para diferentes seções financeiras (Visão Geral, Receitas, Despesas, Contas a Receber, Contas a Pagar, Fluxo de Caixa). Cada seção deve apresentar listagens com filtros e opções para registrar novas transações ou pagamentos.

**Componentes:**
*   **Topbar e Sidebar.**
*   **Cabeçalho da Página:** Título H1 "Financeiro".
*   **Tabs:** "Visão Geral", "Receitas", "Despesas", "Contas a Receber", "Contas a Pagar", "Fluxo de Caixa".
*   **Visão Geral (Tab):** Cards de KPIs (Faturamento, Contas a Receber Pendentes, Contas a Pagar Pendentes), Gráfico de linha (Fluxo de Caixa).
*   **Receitas/Despesas (Tabs):** Tabela de `financial_transactions` com filtros por período, categoria, tipo. Botão Primário "Registrar Receita/Despesa". Modal/Drawer para formulário de registro.
*   **Contas a Receber/Pagar (Tabs):** Tabela de `accounts_receivable`/`accounts_payable` com filtros por status, vencimento, cliente/fornecedor. Ações (Registrar Pagamento, Editar, Cancelar). Modal/Drawer para registro de pagamento.
*   **Fluxo de Caixa (Tab):** Tabela ou gráfico de barras/linhas mostrando o saldo diário/mensal. Filtros por período.
*   **Empty States:** Mensagens e botões para iniciar o registro financeiro.

### 4.8 Produtos

**Objetivo:** Gerenciar o catálogo de produtos e peças da oficina.

**Experiência do Usuário:** Listagem de produtos com busca e filtros. Opções para adicionar, editar, inativar produtos. O formulário deve permitir registrar nome, SKU, categoria, descrição, unidade, custo, preço de venda e informações de estoque.

**Componentes:**
*   **Topbar e Sidebar.**
*   **Cabeçalho da Página:** Título H1 "Produtos", Botão Primário "Novo Produto".
*   **Barra de Busca:** Input de texto.
*   **Filtros:** Dropdowns para "Categoria", "Status" (Ativo, Inativo).
*   **Tabela de Produtos:** Colunas para Nome, SKU, Categoria, Unidade, Preço de Venda, Estoque Atual, Status. Ações (Editar, Ativar/Inativar).
*   **Paginação.**
*   **Modal/Drawer de Cadastro/Edição:** Formulário com Inputs para Nome, SKU, Categoria, Descrição, Unidade (Select), Custo, Preço de Venda, Estoque Atual, Estoque Mínimo. Botões "Salvar" e "Cancelar".
*   **Empty State:** Ícone ilustrativo, mensagem "Nenhum produto cadastrado ainda.", Botão Primário "Cadastrar primeiro produto".

### 4.9 Serviços

**Objetivo:** Gerenciar o catálogo de serviços oferecidos pela oficina.

**Experiência do Usuário:** Listagem de serviços com busca e filtros. Opções para adicionar, editar, inativar serviços. O formulário deve permitir registrar nome, código interno, categoria, descrição, valor padrão e tempo estimado.

**Componentes:**
*   **Topbar e Sidebar.**
*   **Cabeçalho da Página:** Título H1 "Serviços", Botão Primário "Novo Serviço".
*   **Barra de Busca:** Input de texto.
*   **Filtros:** Dropdowns para "Categoria", "Status" (Ativo, Inativo).
*   **Tabela de Serviços:** Colunas para Nome, Código Interno, Categoria, Valor Padrão, Tempo Estimado, Status. Ações (Editar, Ativar/Inativar).
*   **Paginação.**
*   **Modal/Drawer de Cadastro/Edição:** Formulário com Inputs para Nome, Código Interno, Categoria, Descrição, Valor Padrão, Tempo Estimado (em minutos). Botões "Salvar" e "Cancelar".
*   **Empty State:** Ícone ilustrativo, mensagem "Nenhum serviço cadastrado ainda.", Botão Primário "Cadastrar primeiro serviço".

### 4.10 Assinaturas

**Objetivo:** Permitir que o proprietário da oficina gerencie o plano de assinatura do OficinaPro.

**Experiência do Usuário:** Uma tela dedicada ao proprietário, exibindo o plano atual, status da assinatura, período vigente e opções para upgrade, cancelamento ou acesso ao portal de cliente da Stripe. Deve ser clara sobre os benefícios do plano e as consequências de um cancelamento.

**Componentes:**
*   **Topbar e Sidebar.**
*   **Cabeçalho da Página:** Título H1 "Minha Assinatura".
*   **Card de Plano Atual:** Exibe Nome do Plano, Intervalo (Mensal/Anual), Preço, Recursos do Plano. Status (Badge: Ativo, Trial, Vencido, Cancelado).
*   **Informações de Período:** "Próxima cobrança em: [data]", "Trial termina em: [data]".
*   **Botões de Ação:** Botão Primário "Alterar Plano" (leva para seleção de planos), Botão Secundário "Gerenciar Cobrança (Stripe)" (abre portal Stripe), Botão Outline "Cancelar Assinatura" (com modal de confirmação).
*   **Mensagens:** Informações sobre o trial, inadimplência ou cancelamento agendado.

### 4.11 Configurações

**Objetivo:** Permitir que o usuário gerencie seu perfil, dados da oficina, usuários e preferências do sistema.

**Experiência do Usuário:** Uma tela com abas ou um menu lateral secundário para navegar entre as diferentes seções de configuração (Perfil, Empresa, Usuários, Preferências). Cada seção deve ter formulários claros e botões de salvar/cancelar.

**Componentes:**
*   **Topbar e Sidebar.**
*   **Cabeçalho da Página:** Título H1 "Configurações".
*   **Tabs/Menu Lateral Secundário:** "Meu Perfil", "Minha Oficina", "Usuários", "Preferências".
*   **Meu Perfil (Tab):** Formulário com Inputs para Nome, Telefone, Avatar (upload), Senha (com campos para Nova Senha, Confirmar Nova Senha). Botão "Salvar Alterações".
*   **Minha Oficina (Tab):** Formulário com Inputs para Nome Fantasia, Razão Social, CNPJ/CPF, Inscrição Estadual, E-mail, Telefone, WhatsApp, Logo (upload), Endereço. Botão "Salvar Alterações".
*   **Usuários (Tab):** Tabela de `tenant_users` com Nome, E-mail, Perfil, Status. Ações (Editar Perfil, Ativar/Inativar, Remover). Botão Primário "Convidar Usuário" (com modal para e-mail e perfil).
*   **Preferências (Tab):** Formulário com Inputs para Numeração de OS (automática/manual, prefixo), Moeda Padrão, Padrão de Vencimento, Mensagens Padrão (para OS, etc.). Botão "Salvar Preferências".

## 5. Responsividade

O sistema será totalmente responsivo, adaptando-se a diferentes tamanhos de tela para garantir uma experiência de usuário consistente e funcional em desktop, tablet e celular.

### 5.1 Desktop (>= 1024px)

*   **Layout:** Sidebar lateral expandida, topbar completa, conteúdo principal em grid de 2 ou 3 colunas para dashboards e listagens.
*   **Navegação:** Sidebar sempre visível, com opção de recolher.
*   **Tabelas:** Colunas completas, ações visíveis.
*   **Formulários:** Layout em 2 colunas ou mais, aproveitando o espaço.

### 5.2 Tablet (768px - 1023px)

*   **Layout:** Sidebar lateral recolhida por padrão (apenas ícones), topbar adaptada, conteúdo principal em grid de 1 ou 2 colunas.
*   **Navegação:** Sidebar recolhida, expande ao clicar. Menu hambúrguer na topbar para acesso rápido.
*   **Tabelas:** Colunas importantes visíveis, detalhes adicionais podem ser expandidos ou acessados em tela de detalhes.
*   **Formulários:** Layout em 1 ou 2 colunas.

### 5.3 Celular (< 768px)

*   **Layout:** Sidebar completamente oculta, acessível via menu hambúrguer na topbar. Conteúdo principal em layout de coluna única.
*   **Navegação:** Menu hambúrguer na topbar para abrir a sidebar como um drawer.
*   **Tabelas:** Versão simplificada (cards para cada item da lista) ou com rolagem horizontal. Ações em dropdowns ou botões flutuantes.
*   **Formulários:** Layout de coluna única, campos grandes e fáceis de tocar.

## 6. Design System para Next.js 15 + Tailwind + shadcn/ui

O Design System do OficinaPro será construído sobre a base do shadcn/ui, que é um conjunto de componentes reutilizáveis e acessíveis, estilizados com Tailwind CSS. A customização será feita para refletir a identidade visual definida neste documento, garantindo que o código seja consistente e fácil de manter.

### 6.1 Estrutura de Arquivos (Exemplo)

```text
src/
  app/
  components/
    ui/ # Componentes base do shadcn/ui (customizados)
      button.tsx
      input.tsx
      table.tsx
      dialog.tsx # Modal
      drawer.tsx
      tabs.tsx
      badge.tsx
      toast.tsx
      dropdown-menu.tsx
      calendar.tsx # Date Picker
      # ... outros componentes shadcn/ui
    custom/ # Componentes específicos do OficinaPro
      sidebar.tsx
      topbar.tsx
      kpi-card.tsx
      empty-state.tsx
      loading-spinner.tsx
      # ... outros componentes compostos
  lib/
    utils.ts # Funções utilitárias, como `cn` para classes Tailwind
  styles/
    globals.css # Tailwind base, customizações globais, fontes
    theme.ts # Definições de cores para tema claro/escuro
  hooks/
  context/
  # ... outras pastas do Next.js
```

### 6.2 Configuração do Tailwind CSS (`tailwind.config.ts`)

O arquivo de configuração do Tailwind será estendido para incluir as cores personalizadas, tipografia e outras variáveis de design.

```javascript
/** @type {import(\'tailwindcss\').Config} */
module.exports = {
  darkMode: ["class"], // Suporte a tema escuro
  content: [
    "./pages/**/*.{ts,tsx}",
    "./components/**/*.{ts,tsx}",
    "./app/**/*.{ts,tsx}",
    "./src/**/*.{ts,tsx}",
  ],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: {
        "2xl": "1400px",
      },
    },
    extend: {
      colors: {
        // Cores Primárias
        primary: {
          DEFAULT: "#3366ff", // Primária Azul
          foreground: "#ffffff",
          light: "#6699ff", // Azul Claro
          dark: "#0033cc", // Azul Escuro
        },
        // Paleta Neutra
        background: "hsl(var(--background))", // Variável CSS para tema
        foreground: "hsl(var(--foreground))", // Variável CSS para tema
        card: "hsl(var(--card))",
        "card-foreground": "hsl(var(--card-foreground))",
        popover: "hsl(var(--popover))",
        "popover-foreground": "hsl(var(--popover-foreground))",
        muted: "hsl(var(--muted))",
        "muted-foreground": "hsl(var(--muted-foreground))",
        accent: "hsl(var(--accent))",
        "accent-foreground": "hsl(var(--accent-foreground))",
        destructive: {
          DEFAULT: "#dc3545", // Erro
          foreground: "#ffffff",
        },
        success: {
          DEFAULT: "#28a745", // Sucesso
          foreground: "#ffffff",
        },
        warning: {
          DEFAULT: "#ffc107", // Alerta
          foreground: "#000000",
        },
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
      keyframes: {
        "accordion-down": {
          from: { height: 0 },
          to: { height: "var(--radix-accordion-content-height)" },
        },
        "accordion-up": {
          from: { height: "var(--radix-accordion-content-height)" },
          to: { height: 0 },
        },
      },
      animation: {
        "accordion-down": "accordion-down 0.2s ease-out",
        "accordion-up": "accordion-up 0.2s ease-out",
      },
      fontFamily: {
        sans: ["Inter", "sans-serif"], // Configura a fonte Inter
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
};
```

### 6.3 Variáveis CSS para Temas (`globals.css`)

As variáveis CSS serão usadas para implementar o tema claro e escuro, permitindo que o Tailwind CSS utilize as cores corretas com base na classe `dark` no `html`.

```css
@layer base {
  :root {
    --background: 0 0% 100%; /* Branco */
    --foreground: 222.2 84% 4.9%; /* Cinza Escuro */
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 221 83% 60%; /* Azul Primário */
    --primary-foreground: 210 20% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%; /* Erro */
    --destructive-foreground: 210 20% 98%;
    --success: 120 73.7% 45.1%; /* Sucesso */
    --success-foreground: 210 20% 98%;
    --warning: 45 100% 72.5%; /* Alerta */
    --warning-foreground: 222.2 47.4% 11.2%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%; /* Preto */
    --foreground: 210 20% 98%; /* Cinza Muito Claro */
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 20% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 20% 98%;
    --primary: 221 83% 60%; /* Azul Primário */
    --primary-foreground: 210 20% 98%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 20% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 20% 98%;
    --destructive: 0 62.8% 30.6%; /* Erro */
    --destructive-foreground: 210 20% 98%;
    --success: 120 73.7% 45.1%; /* Sucesso */
    --success-foreground: 210 20% 98%;
    --warning: 45 100% 72.5%; /* Alerta */
    --warning-foreground: 222.2 84% 4.9%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
    font-family: "Inter", sans-serif;
  }
}
```

## 7. Critérios de Aceite do Design System

O Design System será considerado completo e pronto para uso quando atender aos seguintes critérios:

*   **Consistência:** Todos os componentes e telas seguem as diretrizes de cores, tipografia e espaçamento definidos.
*   **Responsividade:** O layout se adapta perfeitamente a desktop, tablet e celular, sem quebras visuais ou perda de funcionalidade.
*   **Acessibilidade:** Componentes são acessíveis (foco, contraste, semântica) e compatíveis com leitores de tela.
*   **Reutilização:** Componentes são modulares e podem ser facilmente reutilizados em diferentes partes do sistema.
*   **Documentação:** O Design System é bem documentado, facilitando o entendimento e a colaboração da equipe de desenvolvimento.
*   **Performance:** A implementação não introduz gargalos de performance na renderização da interface.

## 8. Referências Visuais

As seguintes plataformas serviram de inspiração visual para o Design System do OficinaPro:

*   **Stripe:** [https://stripe.com/](https://stripe.com/) - Minimalismo, tipografia clara, uso estratégico de cores.
*   **Linear:** [https://linear.app/](https://linear.app/) - Layout limpo, navegação eficiente, tema escuro bem executado.
*   **Vercel:** [https://vercel.com/](https://vercel.com/) - Design moderno, responsividade, uso de componentes modulares.
*   **Notion:** [https://www.notion.so/](https://www.notion.so/) - Foco na legibilidade, organização de conteúdo, flexibilidade.
*   **HubSpot:** [https://www.hubspot.com/](https://www.hubspot.com/) - Dashboards informativos, componentes de formulário robustos.

---

**Conclusão:** Este Guia de UI/UX e Design System fornece uma base sólida para a construção da interface do OficinaPro. Ao seguir estas diretrizes e utilizar os componentes shadcn/ui customizados, a equipe de desenvolvimento poderá criar um produto com uma experiência de usuário premium, moderna e intuitiva, alinhada com as expectativas de um SaaS de alta qualidade.
