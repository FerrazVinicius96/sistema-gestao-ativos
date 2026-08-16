# Sistema de Gestão de Ativos (SGA)

Sistema full-stack para gestão do ciclo de vida de ativos de TI de uma rede pública municipal — do recebimento da solicitação ao descarte do equipamento — usado por administradores, gestores e operadores de campo.

## Contexto

O SGA nasceu para substituir o controle de patrimônio de TI feito em planilhas por uma rede de ensino municipal: milhares de ativos (notebooks, desktops, tablets, periféricos) distribuídos entre unidades administrativas e escolares, com movimentações, empréstimos, substituições e baixas que precisavam de rastreabilidade.

Este repositório é um fork sanitizado de portfólio, derivado de um projeto real desenvolvido durante um estágio em tecnologia. Dados de produção (pessoas, senhas, IPs internos, identificação do órgão) foram removidos do código e do histórico — o que resta é a arquitetura e a lógica de negócio.

## Minha contribuição

### Módulo de Solicitações de TI — mudança no início do fluxo

**Antes:** o sistema só existia a partir do momento em que um ativo já estava sendo movimentado. Não havia registro formal de *pedido* — uma escola ou setor solicitava um equipamento por e-mail, SEI ou ordem direta, e essa solicitação não tinha rastro no sistema até alguém criar a movimentação manualmente. Não existia aprovação, fila de espera por falta de estoque, nem controle de quem já foi avisado (DIT) sobre uma aprovação.

**O que implementei:** um módulo de solicitações (`requests`) que passou a ser a porta de entrada do fluxo, acoplado às movimentações existentes por uma FK opcional (`request_id` nullable em `asset_movements`) — migração desenhada para não impactar nenhum registro histórico. O módulo cobre:

- Máquina de estados unificada (requisitado → visita técnica solicitada/realizada → aguardando aprovação → aprovado → indisponível em estoque → em execução → concluído, com reprovado/cancelado como saídas), reaproveitada para os três tipos de solicitação (acréscimo, substituição, empréstimo)
- Visita técnica: agendamento, múltiplas visitas sequenciais com histórico completo, e decisão por item individual (quantidade constatada, aprovação parcial)
- Acompanhamento de ciência da equipe de infraestrutura (DIT) sobre aprovações, com KPI de pendências
- Fila de "indisponível em estoque" agrupada por tipo de equipamento, com tempo de espera
- Painel de rotas de visita técnica agrupado por RPA (região político-administrativa)

**Impacto:** processos que antes só existiam como e-mail/ofício/chamado (e portanto invisíveis para métricas e auditoria) passaram a ter protocolo, status e histórico dentro do sistema — cobrindo acréscimo, substituição por avaria e empréstimo de equipamento, do pedido inicial até a entrega confirmada.

### Módulo de Feedback

FAB (botão de ação flutuante) para captura de reclamações, elogios, observações e dúvidas dos usuários finais, com tela de gestão dedicada para triagem.

### Evolução do módulo de tablets

Trabalhei sobre o módulo pré-existente de distribuição de tablets a estudantes: importação de histórico de entregas retroativas, exibição de contagem de pendências (alunos com necessidade de tecnologia assistiva — Livox), número de série no lote de entrega, e metadados de RPA no recibo.

### Trabalhar dentro de um monólito

Boa parte deste trabalho foi construído sobre uma base de código monolítica (backend e frontend com mais de 12 mil linhas cada, em `server.js` e `App.tsx`). A decisão de acoplar o módulo de solicitações via FK opcional, em vez de reescrever o fluxo de movimentações existente, foi deliberada: manter compatibilidade total com dados e comportamento já em produção enquanto uma funcionalidade nova e crítica era adicionada.

## Stack técnica

- **Backend:** Node.js, Express
- **Frontend:** React, TypeScript
- **Banco de dados:** PostgreSQL
- **Autenticação:** JWT

## Arquitetura

- 17 tabelas no banco de dados
- 143 rotas de API
- 18 views/telas no frontend
- 5 perfis de acesso distintos
- Geração de documentos (recibos, termos de responsabilidade) em PDF via `pdfmake`

## Decisões técnicas relevantes

- **Migração sem impacto:** o vínculo entre solicitações e movimentações foi implementado como coluna opcional, permitindo que o módulo novo convivesse com todo o histórico de movimentações já existente sem migração de dados retroativa.
- **Máquina de estados única, reaproveitada:** em vez de um fluxo por tipo de solicitação (acréscimo/substituição/empréstimo), optei por um único conjunto de status compartilhado, com transições condicionadas ao tipo — reduz duplicação de lógica de validação no backend e de UI no frontend.
- **Decisão granular por item:** tanto a visita técnica quanto a aprovação permitem decisão por equipamento individual dentro da mesma solicitação (quantidade constatada, aprovação parcial), em vez de aprovar/reprovar a solicitação inteira — reflete como a aprovação realmente acontece na prática (nem todo item de uma solicitação multi-equipamento é aprovado igual).

## Como rodar localmente

Pré-requisitos: Node.js, PostgreSQL.

```bash
# Backend
cd backend
cp .env.example .env   # preencha DATABASE_URL e JWT_SECRET
npm install
npm start

# Frontend
cd frontend
cp .env.example .env   # aponte REACT_APP_API_URL para o backend local
npm install
npm start
```

O schema do banco é criado pelas migrations em `backend/src/migrations/`.

## Screenshots

Cadastro de Solicitação:

<img width="3430" height="1275" alt="Captura de tela 2026-08-15 222002" src="https://github.com/user-attachments/assets/63dd5e49-b65b-4a9f-838c-66fda84d418b" />

Detalhes da Solicitação associados à máquina de estados: 

<img width="661" height="634" alt="Captura de tela 2026-08-15 222201" src="https://github.com/user-attachments/assets/79c8dda5-cd12-40db-8dda-6348af391b5b" />

## Nota de origem

Este repositório é um fork sanitizado de um sistema desenvolvido durante um estágio em tecnologia, com dados institucionais e de produção removidos para servir como peça de portfólio.
