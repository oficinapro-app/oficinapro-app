# OficinaPro — Biblioteca de Componentes

**Autor:** Manus AI  
**Data:** 09 de junho de 2026  
**Versão:** 1.0  
**Finalidade:** Este documento serve como a referência oficial para a biblioteca de componentes reutilizáveis do frontend do OficinaPro. Ele detalha a estrutura, propósito, propriedades, variantes, estados, comportamentos, exemplos de uso, responsividade e acessibilidade de cada componente, utilizando Next.js 15, Tailwind CSS e shadcn/ui como base tecnológica.

## 1. Padrões de Componentes

### 1.1 Nomenclatura e Organização

*   **Convenção de Nomes:** Componentes seguirão a convenção PascalCase (ex: `Sidebar`, `DataTable`).
*   **Estrutura de Pastas:** Componentes serão organizados em uma estrutura lógica dentro da pasta `components/`:
    *   `components/ui/`: Componentes base do shadcn/ui (diretamente importados ou estendidos).
    *   `components/layout/`: Componentes de layout (Sidebar, Header).
    *   `components/dashboard/`: Componentes específicos do dashboard (KPI Card, Chart Card).
    *   `components/forms/`: Componentes de formulário (Input, Select, DatePicker).
    *   `components/feedback/`: Componentes de feedback (Toast, EmptyState).
    *   `components/shared/`: Componentes genéricos e reutilizáveis em várias partes do sistema (DataTable, SearchBar).
    *   `components/domain/`: Componentes específicos de domínio (WorkOrderCard, ServiceItemRow).

### 1.2 Reutilização e Composição

*   **Composição:** Priorizar a composição de componentes menores e mais simples para construir componentes mais complexos.
*   **Props:** Utilizar props claras e bem definidas para configurar o comportamento e a aparência dos componentes.
*   **Slots:** Em casos de uso avançados, utilizar o padrão de slots para injetar conteúdo dinâmico.

### 1.3 Acessibilidade (A11y)

Todos os componentes serão desenvolvidos com foco em acessibilidade, seguindo as diretrizes WCAG e utilizando os recursos de acessibilidade do shadcn/ui (baseado em Radix UI).

*   **Semântica HTML:** Utilizar elementos HTML semânticos apropriados.
*   **Atributos ARIA:** Aplicar atributos ARIA quando necessário para melhorar a experiência de usuários de tecnologias assistivas.
*   **Navegação por Teclado:** Garantir que todos os elementos interativos sejam navegáveis e operáveis via teclado.
*   **Contraste de Cores:** Manter um contraste de cores adequado para garantir legibilidade.

### 1.4 Responsividade

Os componentes serão projetados para serem responsivos por padrão, adaptando-se a diferentes tamanhos de tela (desktop, tablet, mobile) utilizando Tailwind CSS.

*   **Breakpoints:** Utilizar os breakpoints padrão do Tailwind CSS (`sm`, `md`, `lg`, `xl`, `2xl`).
*   **Flexbox/Grid:** Preferir Flexbox e Grid para layouts flexíveis.

## 2. Componentes de Layout

### 2.1 `Sidebar`

*   **Objetivo:** Componente de navegação lateral principal do sistema, recolhível.
*   **Props:**
    *   `isOpen`: boolean (controla se a sidebar está aberta ou recolhida).
    *   `onToggle`: () => void (função para alternar o estado da sidebar).
    *   `items`: array de objetos (links de navegação, com `label`, `icon`, `href`).
    *   `activeItem`: string (ID ou rota do item ativo).
*   **Variantes:** Aberta, Recolhida.
*   **Estados:** Hover nos itens, Item ativo.
*   **Comportamentos:**
    *   Ao clicar no ícone de toggle, a sidebar se expande/recolhe.
    *   Em telas menores, a sidebar pode se comportar como um drawer (deslizando da lateral).
*   **Exemplos de uso:**
    ```tsx
    <Sidebar
      isOpen={isSidebarOpen}
      onToggle={() => setIsSidebarOpen(!isSidebarOpen)}
      items={navigationItems}
      activeItem={currentRoute}
    />
    ```
*   **Responsividade:** Em mobile, pode ser um drawer que abre com um botão de menu (hambúrguer) na `Header`.
*   **Acessibilidade:** Navegação por teclado, atributos `aria-expanded` e `aria-controls`.

### 2.2 `Header`

*   **Objetivo:** Barra superior do sistema, contendo logo, título, menu de usuário e notificações.
*   **Props:**
    *   `title`: string (título da página atual).
    *   `onSidebarToggle`: () => void (função para abrir/fechar a sidebar em mobile).
    *   `user`: objeto (informações do usuário logado).
    *   `notifications`: array de objetos (notificações).
*   **Variantes:** Padrão.
*   **Estados:** Notificações não lidas, menu de usuário aberto.
*   **Comportamentos:**
    *   Exibe o nome do usuário e sua foto de perfil.
    *   Ícone de sino para notificações, com badge para não lidas.
    *   Menu dropdown para ações do usuário (perfil, configurações, logout).
*   **Exemplos de uso:**
    ```tsx
    <Header
      title="Dashboard"
      onSidebarToggle={handleSidebarToggle}
      user={currentUser}
      notifications={userNotifications}
    />
    ```
*   **Responsividade:** O botão de toggle da sidebar aparece em telas menores. O menu de usuário pode se adaptar para ocupar menos espaço.
*   **Acessibilidade:** Navegação por teclado, rótulos ARIA para botões e menus.

### 2.3 `UserMenu`

*   **Objetivo:** Menu dropdown para ações relacionadas ao usuário (perfil, configurações, logout).
*   **Props:**
    *   `user`: objeto (informações do usuário, ex: `name`, `email`, `avatarUrl`).
    *   `onLogout`: () => void (função para logout).
    *   `onProfileClick`: () => void (função para navegar para o perfil).
    *   `onSettingsClick`: () => void (função para navegar para configurações).
*   **Variantes:** Padrão.
*   **Estados:** Aberto, Fechado.
*   **Comportamentos:**
    *   Abre ao clicar no avatar/nome do usuário.
    *   Fecha ao clicar fora ou selecionar um item.
*   **Exemplos de uso:**
    ```tsx
    <UserMenu
      user={currentUser}
      onLogout={handleLogout}
      onProfileClick={() => router.push('/profile')}
    />
    ```
*   **Responsividade:** Adapta-se ao espaço disponível, pode ser um drawer em mobile.
*   **Acessibilidade:** Navegação por teclado, gerenciamento de foco, atributos ARIA para menu e itens.

### 2.4 `Breadcrumb`

*   **Objetivo:** Componente de navegação que indica a localização atual do usuário na hierarquia do sistema.
*   **Props:**
    *   `items`: array de objetos (com `label`, `href`, `isCurrent`).
*   **Variantes:** Padrão.
*   **Estados:** Item atual (não clicável).
*   **Comportamentos:**
    *   Links para páginas anteriores na hierarquia.
    *   O último item representa a página atual e não é um link.
*   **Exemplos de uso:**
    ```tsx
    <Breadcrumb
      items={[
        { label: 'Dashboard', href: '/dashboard' },
        { label: 'Clientes', href: '/customers' },
        { label: 'Detalhes do Cliente', isCurrent: true }
      ]}
    />
    ```
*   **Responsividade:** Pode truncar itens ou usar um dropdown para itens intermediários em telas menores.
*   **Acessibilidade:** Navegação por teclado, `aria-current="page"` para o item atual.


## 3. Componentes de Dashboard

### 3.1 `KPI Card`

*   **Objetivo:** Exibir um Key Performance Indicator (KPI) com um valor principal, um rótulo e, opcionalmente, uma variação percentual ou ícone.
*   **Props:**
    *   `label`: string (rótulo do KPI, ex: "Faturamento Total").
    *   `value`: string (valor principal, ex: "R$ 15.000,00").
    *   `icon`: ReactNode (ícone opcional).
    *   `change`: string (opcional, ex: "+5%" ou "-2%").
    *   `changeType`: 'positive' | 'negative' | 'neutral' (opcional, para estilizar a variação).
*   **Variantes:** Padrão, com ícone, com variação positiva, com variação negativa.
*   **Estados:** Padrão.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <KPICard
      label="Faturamento Total"
      value="R$ 15.000,00"
      icon={<DollarSignIcon />}
      change="+10%"
      changeType="positive"
    />
    ```
*   **Responsividade:** Adapta-se ao tamanho do container, pode ter fonte menor em telas pequenas.
*   **Acessibilidade:** Rótulos claros, contraste de cores.

### 3.2 `Metric Card`

*   **Objetivo:** Exibir uma métrica mais detalhada, com título, valor, descrição e, opcionalmente, um gráfico em miniatura.
*   **Props:**
    *   `title`: string (título da métrica).
    *   `value`: string (valor principal).
    *   `description`: string (descrição adicional).
    *   `chartData`: array de números (opcional, para um sparkline).
*   **Variantes:** Padrão, com gráfico.
*   **Estados:** Padrão.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <MetricCard
      title="OS Abertas"
      value="42"
      description="2 novas hoje"
      chartData={[10, 12, 15, 13, 18, 20, 22]}
    />
    ```
*   **Responsividade:** Adapta-se ao tamanho do container.
*   **Acessibilidade:** Rótulos claros, texto alternativo para gráficos.

### 3.3 `Chart Card`

*   **Objetivo:** Container para exibir diferentes tipos de gráficos (barras, linhas, pizza) com um título e, opcionalmente, um seletor de período.
*   **Props:**
    *   `title`: string (título do gráfico).
    *   `children`: ReactNode (o componente do gráfico real, ex: `<BarChart />`).
    *   `onPeriodChange`: (period: string) => void (opcional, para seletor de período).
*   **Variantes:** Padrão, com seletor de período.
*   **Estados:** Padrão.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <ChartCard title="Faturamento Mensal" onPeriodChange={handlePeriodChange}>
      <LineChart data={monthlyRevenueData} />
    </ChartCard>
    ```
*   **Responsividade:** O gráfico interno deve ser responsivo. O card se adapta ao container.
*   **Acessibilidade:** Título claro, texto alternativo para o gráfico, navegação por teclado para o seletor de período.

## 4. Componentes de Tabelas

### 4.1 `DataTable`

*   **Objetivo:** Exibir dados tabulares com funcionalidades de ordenação, seleção e, opcionalmente, filtros e paginação.
*   **Props:**
    *   `columns`: array de objetos (definição das colunas, ex: `accessorKey`, `header`, `cell`).
    *   `data`: array de objetos (dados a serem exibidos).
    *   `onRowClick`: (row: TData) => void (opcional, para ação ao clicar na linha).
    *   `isLoading`: boolean (opcional, para exibir skeleton).
    *   `pagination`: objeto (opcional, com `total`, `page`, `pageSize`, `onPageChange`, `onPageSizeChange`).
    *   `onSortChange`: (sortBy: string, sortOrder: 'asc' | 'desc') => void (opcional).
    *   `onSelectionChange`: (selectedRows: string[]) => void (opcional).
*   **Variantes:** Padrão, com seleção, com paginação, com ordenação.
*   **Estados:** Carregando, Vazio, Erro, Linhas selecionadas, Linha em hover.
*   **Comportamentos:**
    *   Ordenação ao clicar no cabeçalho da coluna.
    *   Seleção de linhas via checkbox.
    *   Paginação.
*   **Exemplos de uso:**
    ```tsx
    <DataTable
      columns={customerColumns}
      data={customers}
      onRowClick={handleCustomerClick}
      isLoading={loadingCustomers}
      pagination={customerPagination}
    />
    ```
*   **Responsividade:** Colunas podem ser ocultadas em telas menores, ou a tabela pode se tornar "scrollable" horizontalmente.
*   **Acessibilidade:** Navegação por teclado, cabeçalhos de coluna com `scope="col"`, `aria-sort` para ordenação.

### 4.2 `Pagination`

*   **Objetivo:** Componente de controle de paginação para tabelas e listas.
*   **Props:**
    *   `totalItems`: number (número total de itens).
    *   `currentPage`: number (página atual).
    *   `itemsPerPage`: number (itens por página).
    *   `onPageChange`: (page: number) => void (função para mudar de página).
    *   `onItemsPerPageChange`: (size: number) => void (opcional, para mudar itens por página).
*   **Variantes:** Padrão.
*   **Estados:** Botões de página desabilitados (primeira/última página).
*   **Comportamentos:**
    *   Navegação para página anterior/próxima.
    *   Seleção de número de itens por página.
*   **Exemplos de uso:**
    ```tsx
    <Pagination
      totalItems={100}
      currentPage={1}
      itemsPerPage={10}
      onPageChange={setPage}
      onItemsPerPageChange={setPageSize}
    />
    ```
*   **Responsividade:** Pode simplificar a exibição dos números de página em telas menores.
*   **Acessibilidade:** Navegação por teclado, rótulos ARIA para botões de navegação.

### 4.3 `SearchBar`

*   **Objetivo:** Campo de entrada para pesquisa de texto com um botão de busca ou auto-submit.
*   **Props:**
    *   `placeholder`: string (texto de placeholder).
    *   `onSearch`: (query: string) => void (função chamada ao submeter a busca).
    *   `defaultValue`: string (opcional, valor inicial).
    *   `debounceTime`: number (opcional, tempo em ms para debounce, padrão 300).
*   **Variantes:** Padrão.
*   **Estados:** Vazio, digitando, com resultado.
*   **Comportamentos:**
    *   Submete a busca ao pressionar Enter ou clicar no botão.
    *   Pode ter debounce para evitar requisições excessivas.
*   **Exemplos de uso:**
    ```tsx
    <SearchBar placeholder="Buscar clientes..." onSearch={handleSearch} />
    ```
*   **Responsividade:** Adapta-se à largura do container.
*   **Acessibilidade:** Rótulo associado, placeholder descritivo.

### 4.4 `Filters`

*   **Objetivo:** Componente para aplicar múltiplos filtros a uma lista de dados.
*   **Props:**
    *   `filters`: array de objetos (definição dos filtros, ex: `key`, `label`, `type`, `options`).
    *   `onApplyFilters`: (filters: Record<string, any>) => void (função chamada ao aplicar filtros).
    *   `defaultValues`: Record<string, any> (opcional, valores iniciais dos filtros).
*   **Variantes:** Padrão.
*   **Estados:** Filtros aplicados, filtros pendentes.
*   **Comportamentos:**
    *   Pode abrir um modal ou drawer para seleção de filtros.
    *   Botão para aplicar e limpar filtros.
*   **Exemplos de uso:**
    ```tsx
    <Filters
      filters={[
        { key: 'status', label: 'Status', type: 'select', options: [{ value: 'active', label: 'Ativo' }] },
        { key: 'dateRange', label: 'Período', type: 'dateRange' }
      ]}
      onApplyFilters={handleApplyFilters}
    />
    ```
*   **Responsividade:** Em mobile, pode se transformar em um botão que abre um drawer de filtros.
*   **Acessibilidade:** Navegação por teclado, rótulos claros para cada filtro, feedback visual de filtros aplicados.

## 5. Componentes de Formulários

### 5.1 `FormField`

*   **Objetivo:** Wrapper para campos de formulário, fornecendo rótulo, descrição, mensagem de erro e layout consistente.
*   **Props:**
    *   `label`: string (rótulo do campo).
    *   `htmlFor`: string (ID do input associado).
    *   `description`: string (opcional, texto de ajuda).
    *   `errorMessage`: string (opcional, mensagem de erro).
    *   `children`: ReactNode (o componente de input real).
*   **Variantes:** Padrão, com erro, com descrição.
*   **Estados:** Padrão.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <FormField label="Nome Completo" htmlFor="fullName" errorMessage={errors.fullName}>
      <Input id="fullName" {...register("fullName")} />
    </FormField>
    ```
*   **Responsividade:** Adapta-se à largura do container.
*   **Acessibilidade:** Associa rótulo ao input via `htmlFor`.

### 5.2 `Input`

*   **Objetivo:** Campo de entrada de texto padrão.
*   **Props:** Todas as props de um `<input>` HTML padrão, mais:
    *   `variant`: `default` | `ghost` (opcional, para estilos diferentes).
    *   `isInvalid`: boolean (opcional, para indicar estado de erro).
*   **Variantes:** Padrão, ghost.
*   **Estados:** Normal, Foco, Desabilitado, Erro.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <Input placeholder="Digite seu nome" isInvalid={!!errors.name} />
    ```
*   **Responsividade:** Adapta-se à largura do container.
*   **Acessibilidade:** Rótulo associado (via `FormField`), placeholder.

### 5.3 `Select`

*   **Objetivo:** Componente de seleção de opções.
*   **Props:**
    *   `options`: array de objetos (com `value`, `label`).
    *   `placeholder`: string (texto de placeholder).
    *   `onValueChange`: (value: string) => void.
    *   `defaultValue`: string (opcional).
    *   `isInvalid`: boolean (opcional).
*   **Variantes:** Padrão.
*   **Estados:** Normal, Foco, Desabilitado, Erro, Aberto.
*   **Comportamentos:** Abre um dropdown com opções ao clicar.
*   **Exemplos de uso:**
    ```tsx
    <Select
      options={[{ value: 'active', label: 'Ativo' }, { value: 'inactive', label: 'Inativo' } ]}
      placeholder="Selecione um status"
    />
    ```
*   **Responsividade:** O dropdown pode se adaptar à tela.
*   **Acessibilidade:** Navegação por teclado, rótulos ARIA.

### 5.4 `DatePicker`

*   **Objetivo:** Componente para seleção de datas.
*   **Props:**
    *   `value`: Date (opcional, data selecionada).
    *   `onChange`: (date: Date | undefined) => void.
    *   `placeholder`: string (texto de placeholder).
    *   `isInvalid`: boolean (opcional).
*   **Variantes:** Padrão.
*   **Estados:** Normal, Foco, Desabilitado, Erro, Calendário aberto.
*   **Comportamentos:** Abre um calendário ao clicar no campo.
*   **Exemplos de uso:**
    ```tsx
    <DatePicker value={selectedDate} onChange={setSelectedDate} placeholder="Selecione uma data" />
    ```
*   **Responsividade:** O calendário pode ser um modal em telas menores.
*   **Acessibilidade:** Navegação por teclado no calendário, rótulos ARIA.

### 5.5 `MoneyInput`

*   **Objetivo:** Campo de entrada formatado para valores monetários.
*   **Props:**
    *   `value`: number (opcional).
    *   `onChange`: (value: number | undefined) => void.
    *   `placeholder`: string (opcional).
    *   `isInvalid`: boolean (opcional).
    *   `currency`: string (opcional, padrão 'BRL').
*   **Variantes:** Padrão.
*   **Estados:** Normal, Foco, Desabilitado, Erro.
*   **Comportamentos:** Formata o valor digitado como moeda.
*   **Exemplos de uso:**
    ```tsx
    <MoneyInput value={amount} onChange={setAmount} placeholder="0,00" />
    ```
*   **Responsividade:** Adapta-se à largura do container.
*   **Acessibilidade:** Rótulo associado.

### 5.6 `PhoneInput`

*   **Objetivo:** Campo de entrada formatado para números de telefone.
*   **Props:**
    *   `value`: string (opcional).
    *   `onChange`: (value: string) => void.
    *   `placeholder`: string (opcional).
    *   `isInvalid`: boolean (opcional).
*   **Variantes:** Padrão.
*   **Estados:** Normal, Foco, Desabilitado, Erro.
*   **Comportamentos:** Aplica máscara de telefone ao digitar.
*   **Exemplos de uso:**
    ```tsx
    <PhoneInput value={phoneNumber} onChange={setPhoneNumber} placeholder="(XX) XXXXX-XXXX" />
    ```
*   **Responsividade:** Adapta-se à largura do container.
*   **Acessibilidade:** Rótulo associado.

## 6. Componentes de Feedback

### 6.1 `Toast`

*   **Objetivo:** Exibir mensagens de feedback temporárias e não intrusivas.
*   **Props:**
    *   `variant`: `success` | `error` | `warning` | `info`.
    *   `title`: string (título da mensagem).
    *   `description`: string (opcional, detalhes da mensagem).
    *   `duration`: number (opcional, tempo em ms para fechar automaticamente, padrão 5000).
*   **Variantes:** Sucesso, Erro, Aviso, Informação.
*   **Estados:** Visível, Oculto.
*   **Comportamentos:** Aparece no canto da tela, desaparece após um tempo ou ao ser fechado manualmente.
*   **Exemplos de uso:**
    ```tsx
    toast({
      title: "Sucesso!",
      description: "Operação realizada com êxito.",
      variant: "success",
    });
    ```
*   **Responsividade:** Posição e tamanho podem se ajustar em telas menores.
*   **Acessibilidade:** Anunciado por leitores de tela (aria-live region).

### 6.2 `EmptyState`

*   **Objetivo:** Exibir uma mensagem e/ou ilustração quando não há dados para mostrar.
*   **Props:**
    *   `title`: string (título da mensagem).
    *   `description`: string (descrição detalhada).
    *   `icon`: ReactNode (ícone ou ilustração opcional).
    *   `actionButton`: ReactNode (botão de ação opcional, ex: "Criar Novo").
*   **Variantes:** Padrão, com ícone, com botão de ação.
*   **Estados:** Padrão.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <EmptyState
      title="Nenhum cliente encontrado"
      description="Comece cadastrando seu primeiro cliente para gerenciar sua oficina."
      icon={<UsersIcon />}
      actionButton={<Button>Adicionar Cliente</Button>}
    />
    ```
*   **Responsividade:** Layout pode se ajustar para mobile.
*   **Acessibilidade:** Texto claro e descritivo.

### 6.3 `Skeleton`

*   **Objetivo:** Exibir um placeholder visual enquanto o conteúdo real está sendo carregado.
*   **Props:**
    *   `width`: string | number (largura do skeleton).
    *   `height`: string | number (altura do skeleton).
    *   `className`: string (classes adicionais do Tailwind).
*   **Variantes:** Retangular, circular.
*   **Estados:** Padrão.
*   **Comportamentos:** Animação de carregamento.
*   **Exemplos de uso:**
    ```tsx
    <Skeleton className="h-4 w-[250px]" />
    <Skeleton className="h-12 w-12 rounded-full" />
    ```
*   **Responsividade:** Largura e altura podem ser definidas em unidades relativas.
*   **Acessibilidade:** Indica que o conteúdo está carregando.

### 6.4 `LoadingSpinner`

*   **Objetivo:** Exibir um indicador visual de que uma operação está em andamento.
*   **Props:**
    *   `size`: `small` | `medium` | `large` (opcional, padrão `medium`).
    *   `className`: string (classes adicionais do Tailwind).
*   **Variantes:** Diferentes tamanhos.
*   **Estados:** Girando.
*   **Comportamentos:** Animação de rotação contínua.
*   **Exemplos de uso:**
    ```tsx
    <LoadingSpinner size="medium" />
    ```
*   **Responsividade:** Tamanho pode ser ajustado.
*   **Acessibilidade:** Atributos ARIA para indicar estado de carregamento.

### 6.5 `ConfirmDialog`

*   **Objetivo:** Solicitar confirmação do usuário antes de executar uma ação destrutiva ou irreversível.
*   **Props:**
    *   `title`: string (título do diálogo).
    *   `description`: string (descrição da ação).
    *   `onConfirm`: () => void (função a ser executada na confirmação).
    *   `onCancel`: () => void (função a ser executada no cancelamento).
    *   `open`: boolean (controla a visibilidade do diálogo).
    *   `setOpen`: (open: boolean) => void.
    *   `confirmText`: string (opcional, texto do botão de confirmação, padrão "Confirmar").
    *   `cancelText`: string (opcional, texto do botão de cancelamento, padrão "Cancelar").
*   **Variantes:** Padrão.
*   **Estados:** Aberto, Fechado.
*   **Comportamentos:**
    *   Bloqueia a interação com o restante da página.
    *   Fecha ao confirmar, cancelar ou clicar fora.
*   **Exemplos de uso:**
    ```tsx
    <ConfirmDialog
      open={isConfirmOpen}
      setOpen={setIsConfirmOpen}
      title="Confirmar Exclusão"
      description="Tem certeza que deseja excluir este item? Esta ação não pode ser desfeita."
      onConfirm={handleDeleteConfirm}
    />
    ```
*   **Responsividade:** Ocupa a tela inteira em mobile.
*   **Acessibilidade:** Gerenciamento de foco, atributos ARIA para diálogo modal.

## 7. Componentes Específicos de Ordens de Serviço

### 7.1 `StatusBadge`

*   **Objetivo:** Exibir o status de uma Ordem de Serviço (ou outro item) de forma visualmente distinta.
*   **Props:**
    *   `status`: string (o status a ser exibido, ex: `open`, `in_progress`, `completed`).
*   **Variantes:** Cores e textos diferentes para cada status (ex: `open` = azul, `in_progress` = amarelo, `completed` = verde).
*   **Estados:** Padrão.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <StatusBadge status="in_progress" />
    ```
*   **Responsividade:** Adapta-se ao tamanho do texto.
*   **Acessibilidade:** Texto descritivo do status.

### 7.2 `WorkOrderCard`

*   **Objetivo:** Exibir um resumo de uma Ordem de Serviço em formato de card, ideal para listagens.
*   **Props:**
    *   `workOrder`: objeto (dados da OS, incluindo `number`, `customerName`, `vehiclePlate`, `status`, `totalAmount`).
    *   `onClick`: (workOrderId: UUID) => void (ação ao clicar no card).
*   **Variantes:** Padrão.
*   **Estados:** Hover.
*   **Comportamentos:** Clicável para ver detalhes da OS.
*   **Exemplos de uso:**
    ```tsx
    <WorkOrderCard workOrder={osData} onClick={handleViewWorkOrder} />
    ```
*   **Responsividade:** Layout flexível para se adaptar a diferentes larguras de tela.
*   **Acessibilidade:** Clicável, rótulos claros para as informações.

### 7.3 `Timeline`

*   **Objetivo:** Exibir uma sequência cronológica de eventos (ex: histórico de status da OS).
*   **Props:**
    *   `events`: array de objetos (com `date`, `title`, `description`, `icon` (opcional)).
*   **Variantes:** Padrão.
*   **Estados:** N/A.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <Timeline events={workOrderHistoryEvents} />
    ```
*   **Responsividade:** Adapta-se verticalmente.
*   **Acessibilidade:** Ordem lógica dos eventos, texto claro.

### 7.4 `ServiceItemRow`

*   **Objetivo:** Exibir um item de serviço dentro de uma lista (ex: na aba de serviços da OS).
*   **Props:**
    *   `service`: objeto (dados do serviço, incluindo `description`, `quantity`, `unitPrice`, `total`).
    *   `onEdit`: (serviceId: UUID) => void (opcional, para edição).
    *   `onRemove`: (serviceId: UUID) => void (opcional, para remoção).
*   **Variantes:** Padrão, com botões de ação.
*   **Estados:** Hover.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <ServiceItemRow service={osService} onEdit={handleEditService} onRemove={handleRemoveService} />
    ```
*   **Responsividade:** Colunas podem ser empilhadas em telas menores.
*   **Acessibilidade:** Clicável, rótulos claros para as informações e botões.

### 7.5 `ProductItemRow`

*   **Objetivo:** Exibir um item de produto/peça dentro de uma lista (ex: na aba de produtos da OS).
*   **Props:**
    *   `product`: objeto (dados do produto, incluindo `description`, `quantity`, `unitPrice`, `total`).
    *   `onEdit`: (productId: UUID) => void (opcional, para edição).
    *   `onRemove`: (productId: UUID) => void (opcional, para remoção).
*   **Variantes:** Padrão, com botões de ação.
*   **Estados:** Hover.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <ProductItemRow product={osProduct} onEdit={handleEditProduct} onRemove={handleRemoveProduct} />
    ```
*   **Responsividade:** Colunas podem ser empilhadas em telas menores.
*   **Acessibilidade:** Clicável, rótulos claros para as informações e botões.

## 8. Componentes Específicos de Financeiro

### 8.1 `TransactionCard`

*   **Objetivo:** Exibir um resumo de uma transação financeira (receita ou despesa).
*   **Props:**
    *   `transaction`: objeto (dados da transação, incluindo `description`, `amount`, `type`, `date`).
    *   `onClick`: (transactionId: UUID) => void (opcional, para ver detalhes).
*   **Variantes:** Receita (verde), Despesa (vermelho).
*   **Estados:** Hover.
*   **Comportamentos:** Clicável para ver detalhes da transação.
*   **Exemplos de uso:**
    ```tsx
    <TransactionCard transaction={financialTransaction} onClick={handleViewTransaction} />
    ```
*   **Responsividade:** Layout flexível.
*   **Acessibilidade:** Clicável, rótulos claros.

### 8.2 `BalanceCard`

*   **Objetivo:** Exibir o saldo atual ou um resumo financeiro (receitas, despesas, saldo).
*   **Props:**
    *   `label`: string (rótulo, ex: "Saldo Atual").
    *   `value`: string (valor, ex: "R$ 10.000,00").
    *   `type`: `positive` | `negative` | `neutral` (opcional, para estilizar o valor).
*   **Variantes:** Padrão, positivo, negativo.
*   **Estados:** N/A.
*   **Comportamentos:** N/A.
*   **Exemplos de uso:**
    ```tsx
    <BalanceCard label="Saldo Atual" value="R$ 10.000,00" type="positive" />
    ```
*   **Responsividade:** Adapta-se ao tamanho do container.
*   **Acessibilidade:** Rótulos claros, contraste de cores.

## Conclusão

Este documento detalha a biblioteca de componentes do OficinaPro, fornecendo uma base sólida para o desenvolvimento frontend. A utilização consistente destes componentes, seguindo os padrões de design e acessibilidade, garantirá uma interface de usuário coesa, intuitiva e de alta qualidade. Com esta especificação, as ferramentas de IA e os desenvolvedores terão todas as informações necessárias para construir a interface do OficinaPro com precisão e eficiência.
