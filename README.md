# *ESSE REPO VAI VIRAR PRIVADO DEPOIS*

# Escopo de Produto — CRM Interno (KCRM)

**Status:** Rascunho v0.1 · **Autor:** Felipe Ortiz · **Data:** 12/08/2026 **Objetivo do documento:** alinhar visão, escopo do MVP e decisões arquiteturais antes do início do desenvolvimento.

---

## 1. Visão

Construir um CRM próprio para uso interno, mas **arquitetado desde o dia 1 para se tornar um produto comercial multi-cliente**. O foco é resolver as dores internas que não são atendidas pelos CRMs testados internamente como: Visão financeira que separe receita recorrente (MRR) de pagamentos únicos e comissionamento por vendedor.

> **Princípio norteador:** construir _interno-primeiro, comercial-pronto_. Toda decisão de dados, permissão e infraestrutura assume múltiplos clientes desde o início, mesmo enquanto houver apenas um.

## 2. Problema

Depois de um tempo de experiência com os principais CRMs do mercado concluímos que alguns detalhes únicos a Krolik não estavam sendo preenchidos, alem de alguns detalhes mais aparentes.

O CRM atual (**Agendor**) é inadequado para o ciclo de vendas complexo da empresa. Alternativas avaliadas também não resolvem:

| Ferramenta | Situação   | Motivo                                                                                            |
| ---------- | ---------- | ------------------------------------------------------------------------------------------------- |
| Agendor    | Em uso     | Ver limitações abaixo                                                                             |
| PipeRun    | Descartada | Funcional, porém cara (~R$130/usuário)                                                            |
| Pipedrive  | Descartada | Muito caro                                                                                        |
| RD Station | Descartada | Muito caro                                                                                        |
| Chat Label | Referência | Separa produtos por ciclo de cobrança, mas **não** oferece relatórios que segmentem MRR × pontual |

**Limitações centrais que motivam o projeto:**

1. **Relatórios financeiros** — MRR e pagamentos únicos são consolidados, impossibilitando acompanhar vendas recorrentes com precisão e calcular comissões.
2. **Lacuna do mercado** — mesmo quem separa produtos por ciclo (Chat Label) falha no mesmo ponto: **não há camada de relatório** que segmente as categorias. Essa é a oportunidade que diferencia o produto.

## 3. Usuários e personas

| Persona              | Uso principal              | Necessidade-chave                                                                                                         |
| -------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **SDR**              | Prospecção e qualificação  | Cadastro e associação inicial no funil simples e rápido (nome, telefone); repassar oportunidade ao Closer sem retrabalho. |
| **Closer**           | Negociação e fechamento    | Funil detalhado, geração de proposta, registro de produtos vendidos                                                       |
| **Gestor de vendas** | Acompanhamento e comissões | Relatórios MRR × pontual, cálculo de comissão, previsibilidade                                                            |
| **Marketing**        | Inteligência de mercado    | Volume de produtos vendidos (ex.: "1.000 ramais")                                                                         |
| **Admin**            | Configuração               | Gestão de usuários, funis, templates, permissões                                                                          |

## 4. Escopo do MVP (versão utilizável — meta ~2 meses)

O objetivo do MVP é **substituir o Agendor no dia a dia**, não entregar tudo. As duas features essenciais (propostas + financeiro) são obrigatórias, mas um CRM só é "utilizável" com o básico de pipeline funcionando.

### 4.1 Fundação (pré-requisito, não é feature visível)

- Cadastro detalhado de contatos e empresas.
- Estrutura de oportunidades (deal) com valor, produtos, estágio, responsável.
- Modelo de dados **multi-tenant** (ver §6).
- Autenticação e papéis básicos (SDR, Closer, Gestor, Admin).
- **Importação inicial do Agendor** (necessidade interna imediata; vira feature de onboarding comercial depois).

### 4.2 Geração de propostas

- Upload ou criação de **template personalizado** (PDF/Word) de proposta por organização, **preservando design e branding**.
- Geração automática de uma **página final** anexada a proposta, com dados do CRM (produtos, valores, endereço, etc.).

### 4.3 Relatórios financeiros (MRR × pontual)

- Cada produto/item carrega um **tipo de cobrança**: recorrente (mensal/anual) ou pontual(Encaixar reajuste anual em cima de impostos).
- Relatório que **segmenta receita recorrente de pagamentos únicos**.
- **Normalização de MRR:** definir regra para planos anuais (recomendo `valor_anual / 12` para o MRR, mantendo o valor cheio no contrato).
- Base para cálculo de comissão (a lógica de comissão em si pode começar manual/exportável e ser automatizada na Fase 2).

### 4.4 Gestão de oportunidades

- **Clonagem independente** de oportunidade, **incluindo anexos**, para transferência de funil (SDR → Closer).
    - _Nota de produto:_ clonar (em vez de mover) preserva o registro original do SDR para atribuição/métricas. Recomendo **vincular clone ↔ original** para não duplicar receita nos relatórios e ainda creditar ambos os vendedores.
- **Funis separados:** Cadastro funil simplificado do SDR (nome, telefone) e funil detalhado do Closer.

### 4.5 Fora do escopo do MVP

- Relatório de volume de produtos para marketing (Fase 2).
- Cálculo automatizado de comissões (Fase 2 — no MVP, exportar dados basta).
- Dashboards analíticos avançados.
- Substituição de variáveis dentro do template de proposta.
- Assinatura eletrônica de propostas.
- Recursos comerciais (white-label, billing, self-serve) — Fase 3.

## 5. Métricas de sucesso

- **Adoção interna:** 100% do time de vendas migrado do Agendor.
- **Confiabilidade financeira:** MRR e receita pontual reconciliam com o financeiro (meta: divergência < 1%).
- **Handoff SDR → Closer:** sem retrabalho de recadastro (0 dados reinseridos manualmente).

## 6. Decisões arquiteturais — interno → comercial

> Esta seção é a mais importante para a meta de escalabilidade. Cada item abaixo é barato de acertar agora e caro de corrigir depois.

1. **Multi-tenancy desde o dia 1.** Isolamento de dados por organização (tenant), mesmo com um único cliente hoje. Migrar de single-tenant para multi-tenant depois costuma ser reescrita.
2. **RBAC (papéis e permissões)** desde o início: SDR, Closer, Gestor, Admin — e prever papéis customizáveis por organização no futuro.
3. **LGPD.** O CRM guarda dados pessoais (PII). Prever consentimento, exportação, exclusão e log de auditoria já na modelagem.
4. **Importação/migração como capability, não script.** O import do Agendor é necessidade interna imediata evitando retrabalho para cadastro dos leads.

## 7. Riscos e questões em aberto

**Questões para fechar antes do desenvolvimento:**

- Qual o **volume esperado** de organizações/usuários no cenário comercial? (dimensiona multi-tenancy e infra)
- O CRM **calcula comissão** internamente ou **integra com o financeiro**? Quem é a fonte da verdade?
- **Integrações necessárias** no nosso cenário: telefonia/VoIp, WhatsApp, e-mail? Alguma é bloqueante para nosso MVP?
- Regra de **normalização de MRR** para planos anuais.
- **Assinatura de proposta** (ex.: Clicksign/DocuSign) entra em que fase?

**Riscos:**

- **Prazo de ~2 meses é agressivo.** Se surgir pressão, o corte deve preservar as duas features essenciais + pipeline básico; o resto desce de fase.
- **Dupla contagem de receita** na clonagem SDR→Closer se o vínculo clone↔original não for tratado.

## 8. Faseamento sugerido

| Fase                        | Entrega                                                                                  | Foco                  |
| --------------------------- | ---------------------------------------------------------------------------------------- | --------------------- |
| **Fase 0 — Fundação**       | Multi-tenancy, auth/RBAC, modelo de dados, import Agendor                                | Base comercial-pronta |
| **Fase 1 — MVP (~2 meses)** | Pipeline, funis SDR+Closer, clonagem, geração de proposta, relatório MRR × pontual       | Substituir o Agendor  |
| **Fase 2 — Inteligência**   | Contabilidade de comissões automatizadas, relatório de produtos p/ marketing, dashboards | Precisão e insights   |
| **Fase 3 — Comercial**      | White-label, billing/assinaturas, onboarding self-serve, planos                          | Go-to-market          |
