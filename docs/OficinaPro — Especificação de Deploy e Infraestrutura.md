# OficinaPro — Especificação de Deploy e Infraestrutura

**Autor:** Manus AI  
**Data:** 09 de junho de 2026  
**Versão:** 1.0  
**Finalidade:** Este documento detalha a estratégia de deploy e a infraestrutura do sistema OficinaPro, um SaaS multiempresa para oficinas mecânicas. Ele abrange o gerenciamento de código-fonte, ambientes de desenvolvimento e produção, segurança, backup e observabilidade, utilizando GitHub, Vercel, Supabase e Stripe.

## 1. Gerenciamento de Código e Git Flow

O projeto OficinaPro utilizará o **Git Flow** como modelo de ramificação (branching model) para gerenciar o ciclo de vida do desenvolvimento, garantindo um fluxo de trabalho organizado e colaborativo no **GitHub**.

### 1.1 Branches

*   **`main`:**
    *   Representa o código em produção. Apenas deploys estáveis e testados são mesclados nesta branch.
    *   Protegida: Nenhuma alteração direta é permitida. Todas as mesclagens devem vir de `develop` via Pull Request (PR) aprovado.
    *   Deploys automáticos para o ambiente de **Production** na Vercel.

*   **`develop`:**
    *   Representa o código em estágio de desenvolvimento ativo, contendo todas as funcionalidades concluídas e testadas das branches de `feature`.
    *   Base para novas `feature` branches.
    *   Protegida: Nenhuma alteração direta é permitida. Todas as mesclagens devem vir de `feature/*` via PR aprovado.
    *   Deploys automáticos para o ambiente de **Preview** na Vercel (ou um ambiente de staging dedicado, se necessário).

*   **`feature/*`:**
    *   Branches criadas a partir de `develop` para o desenvolvimento de novas funcionalidades, melhorias ou correções de bugs.
    *   Nomenclatura: `feature/nome-da-funcionalidade` ou `bugfix/descricao-do-bug`.
    *   Cada `feature` branch deve ser pequena e focada em uma única tarefa.
    *   Mescladas de volta para `develop` via Pull Request.

### 1.2 Git Flow e Deploy Automático

O fluxo de trabalho será o seguinte:

1.  **Desenvolvimento de Funcionalidades:** Desenvolvedores criam branches `feature/*` a partir de `develop`.
2.  **Pull Requests (PRs):** Ao concluir uma funcionalidade, um PR é aberto de `feature/*` para `develop`.
3.  **Code Reviews:** Todos os PRs devem passar por um code review por pelo menos um outro desenvolvedor. O code review garante a qualidade do código, aderência aos padrões e identificação de potenciais bugs.
4.  **Testes Automatizados:** O GitHub Actions (ou similar) será configurado para rodar testes unitários, de integração e E2E automaticamente em cada PR e em cada push para `develop` e `main`.
5.  **Deploy de Preview (Vercel):** Cada PR aberto para `develop` (ou `main`) acionará um **Preview Deployment** na Vercel. Isso permite que a funcionalidade seja testada em um ambiente isolado com uma URL única antes da mesclagem.
6.  **Mesclagem para `develop`:** Após aprovação do code review e sucesso dos testes, o PR é mesclado para `develop`.
7.  **Deploy de Staging/Preview (Vercel):** A mesclagem para `develop` acionará um deploy automático para o ambiente de **Preview** da Vercel, que servirá como ambiente de staging para testes mais abrangentes.
8.  **Mesclagem para `main`:** Quando um conjunto de funcionalidades em `develop` estiver pronto para ir para produção, um PR é aberto de `develop` para `main`.
9.  **Deploy de Produção (Vercel):** A mesclagem para `main` acionará um deploy automático para o ambiente de **Production** na Vercel, tornando as novas funcionalidades disponíveis para os usuários finais.

## 2. Variáveis de Ambiente

As variáveis de ambiente são essenciais para configurar o aplicativo em diferentes ambientes (local, preview, production) e para proteger informações sensíveis. Elas serão gerenciadas no `.env.local` (para desenvolvimento local) e diretamente na configuração da Vercel e Supabase para os ambientes de deploy.

### 2.1 Variáveis Essenciais

*   `NEXT_PUBLIC_SUPABASE_URL`: URL pública da instância do Supabase. Utilizada no frontend e backend.
*   `NEXT_PUBLIC_SUPABASE_ANON_KEY`: Chave `anon` pública do Supabase. Utilizada no frontend para acesso anônimo e autenticação.
*   `SUPABASE_SERVICE_ROLE_KEY`: Chave `service_role` do Supabase. **Extremamente sensível**, utilizada apenas no backend (Server Actions, Route Handlers) para operações que exigem privilégios elevados, ignorando RLS. **NUNCA expor no frontend.**
*   `STRIPE_SECRET_KEY`: Chave secreta da API da Stripe. Utilizada no backend para interagir com a API da Stripe (criação de clientes, assinaturas, etc.).
*   `STRIPE_WEBHOOK_SECRET`: Chave secreta para verificar a autenticidade dos webhooks da Stripe. Utilizada no Route Handler de webhook.
*   `NEXT_PUBLIC_APP_URL`: URL base da aplicação (ex: `https://oficinapro.com.br`). Utilizada para redirecionamentos e construção de URLs absolutas (ex: para webhooks).

### 2.2 Gerenciamento de Variáveis

*   **Local:** Definidas no arquivo `.env.local` (não versionado no Git).
*   **Vercel (Preview/Production):** Configuradas diretamente no dashboard da Vercel para cada ambiente. Variáveis sensíveis devem ser marcadas como "Secret".
*   **Supabase:** As chaves do Supabase são geradas automaticamente. A `SUPABASE_SERVICE_ROLE_KEY` deve ser tratada como um segredo de ambiente na Vercel.

## 3. Ambientes

O OficinaPro terá três ambientes principais para desenvolvimento, teste e produção.

### 3.1 Local

*   **Propósito:** Desenvolvimento individual por desenvolvedores.
*   **Configuração:** Utiliza variáveis de ambiente do `.env.local`. Pode usar uma instância local do Supabase (via Docker) ou uma instância de desenvolvimento remota.
*   **Características:** Rápido feedback, fácil depuração.

### 3.2 Preview (Staging/Desenvolvimento)

*   **Propósito:** Testes de integração, validação de funcionalidades, demonstrações internas.
*   **Configuração:** Deploys automáticos de branches `develop` e `feature/*` na Vercel. Cada PR terá uma URL de preview única.
*   **Características:** Ambiente isolado, dados de teste, acesso restrito a equipes de desenvolvimento e QA.

### 3.3 Production

*   **Propósito:** Ambiente para usuários finais.
*   **Configuração:** Deploy automático da branch `main` na Vercel.
*   **Características:** Alta disponibilidade, segurança robusta, dados reais, monitoramento contínuo.

## 4. Vercel

A Vercel será a plataforma de deploy para o frontend (Next.js) e backend (Server Actions, Route Handlers).

### 4.1 Preview Deployments

*   Cada Pull Request no GitHub gera um deploy de preview com uma URL única (ex: `oficinapro-pr-123.vercel.app`).
*   Permite testar funcionalidades isoladamente antes da mesclagem.
*   Integração com GitHub para feedback direto nos PRs.

### 4.2 Production Deployments

*   Deploys automáticos da branch `main` para o domínio principal (ex: `oficinapro.com.br`).
*   Otimizações de performance (CDN, Edge Functions).
*   Rollbacks fáceis para versões anteriores em caso de problemas.

## 5. Segurança

A segurança é primordial para um SaaS multiempresa. As seguintes medidas serão implementadas:

### 5.1 Secrets

*   Todas as chaves de API, credenciais de banco de dados e outras informações sensíveis serão armazenadas como variáveis de ambiente secretas na Vercel e no Supabase.
*   **NUNCA** hardcode secrets no código-fonte.
*   Acesso restrito a secrets apenas por processos de deploy e funções de backend.

### 5.2 HTTPS

*   Todas as comunicações entre o cliente e o servidor serão criptografadas via HTTPS por padrão (Vercel e Supabase fornecem isso automaticamente).
*   Forçar HTTPS para todas as rotas.

### 5.3 Cookies

*   Cookies de sessão serão marcados como `HttpOnly`, `Secure` e `SameSite=Lax` para prevenir ataques XSS e CSRF.
*   Utilizar as funcionalidades de segurança de cookies do Next.js e Supabase Auth.

### 5.4 Webhooks

*   Todos os webhooks (especialmente da Stripe) serão verificados usando assinaturas secretas para garantir que as requisições vêm de uma fonte legítima.
*   Implementar idempotência no processamento de webhooks para evitar duplicação de eventos.

## 6. Backup

Uma estratégia robusta de backup é essencial para recuperação de desastres e continuidade do negócio.

### 6.1 Banco de Dados (Supabase/PostgreSQL)

*   **Backups Automáticos:** O Supabase oferece backups diários automáticos do PostgreSQL. Verificar e configurar a retenção de backups conforme a política de recuperação de desastres.
*   **Point-in-Time Recovery (PITR):** Habilitar PITR no Supabase para recuperação de dados em qualquer ponto no tempo.
*   **Exportação Manual:** Possibilidade de exportar dumps do banco de dados periodicamente para armazenamento externo (ex: S3).

### 6.2 Storage (Supabase Storage)

*   O Supabase Storage armazena arquivos em buckets S3. O S3 oferece alta durabilidade e redundância.
*   Configurar versionamento de objetos nos buckets para permitir a recuperação de versões anteriores de arquivos.
*   Considerar replicação entre regiões para maior resiliência.

### 6.3 Logs

*   Logs de aplicação e sistema serão centralizados e armazenados por um período definido para auditoria e depuração.
*   A Vercel integra-se com provedores de log para centralização.

## 7. Observabilidade

A observabilidade é fundamental para entender o comportamento do sistema em produção, identificar problemas rapidamente e garantir a performance.

### 7.1 Logs

*   **Centralização:** Todos os logs (aplicação, banco de dados, webhooks) serão centralizados em uma plataforma de agregação de logs (ex: Vercel Logs, Datadog, Logtail).
*   **Níveis de Log:** Utilizar níveis de log apropriados (DEBUG, INFO, WARN, ERROR, FATAL).
*   **Contexto:** Incluir contexto relevante nos logs (tenant_id, user_id, request_id) para facilitar a depuração em ambientes multiempresa.

### 7.2 Erros

*   **Monitoramento de Erros:** Utilizar uma ferramenta de monitoramento de erros (ex: Sentry, Rollbar) para capturar e alertar sobre exceções e erros em tempo real no frontend e backend.
*   **Alertas:** Configurar alertas para erros críticos que afetam a experiência do usuário ou a integridade dos dados.

### 7.3 Monitoramento

*   **Performance da Aplicação (APM):** Monitorar a performance do frontend e backend (tempos de resposta, latência, uso de recursos) usando ferramentas APM (ex: Vercel Analytics, Datadog, New Relic).
*   **Métricas do Banco de Dados:** Monitorar métricas do PostgreSQL no Supabase (uso de CPU, conexões, queries lentas).
*   **Uptime:** Monitorar a disponibilidade da aplicação e dos serviços externos (Supabase, Stripe).

## Conclusão

Este documento estabelece uma base sólida para a infraestrutura e as práticas de DevOps do OficinaPro. Ao seguir estas diretrizes, garantiremos um processo de desenvolvimento e deploy eficiente, seguro e confiável, resultando em um sistema estável e de alta performance para as oficinas mecânicas.
