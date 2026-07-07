# 🎼 Dashboard Masterclass — Maestro Thiago Santos

Dashboard de performance para as Masterclass do Maestro Thiago Santos, desenvolvido pela [Agência B16](https://agenciab16.com.br).

---

## 🌐 URL

```
https://suporteb16-collab.github.io/dashboard-masterclass-maestro/
```

---

## 🏗️ Arquitetura

```
Google Sheets (privada)
      ↓
Cloudflare Worker (raspy-bush-2c66.henrscard.workers.dev)
      ↓
Dashboard HTML (GitHub Pages)
```

---

## 📋 Planilha

**ID:** `1bPxq4nmFicmJihL1uKtnQMcgXSD0hYIZwgL14xPSL-s`

### Abas utilizadas

| Aba | Conteúdo |
|---|---|
| `KIWIFY` | Dados de vendas (todos os produtos) |
| `META ADS` | Dados de campanhas Meta Ads |
| `TAG_CAMPANHA` | Relação tag × produto × período |

### Estrutura da aba `TAG_CAMPANHA`

| Coluna | Descrição | Exemplo |
|---|---|---|
| `tag_campanha` | Identificador único da campanha | `AR0526` |
| `tag_metaads_masterclass` | Tag no Campaign Name (Meta) | `[AR]` |
| `tag_metaads_produto_principal` | Tag do produto principal (Meta) | `[CH]` |
| `produto_masterclass` | Nome exato do produto na Kiwify | `Masterclass - Alma Romântica` |
| `produto_principal` | Nome exato do produto principal | `Confraria Musical Hélicon` |
| `data_inicio` | Data de início da campanha | `2026-05-12` |
| `data_fim` | Data de fim da campanha | `2026-06-01` |

### Campanhas configuradas

| Tag | Masterclass | Produto Principal | Período |
|---|---|---|---|
| AR0526 | Masterclass - Alma Romântica | — | 12/05 a 01/06/2026 |
| ANEA0726 | Masterclass - A Noite e o Êxtase da Alma | Confraria Musical Hélicon | 01/07 a 15/07/2026 |

---

## ⚙️ Cloudflare Worker

**Worker:** `raspy-bush-2c66.henrscard.workers.dev`

### Atualização necessária

Adicionar `TAG_CAMPANHA` à lista `ALLOWED_SHEETS` no código do Worker (arquivo `worker.js` neste repositório).

### Secrets configurados

| Secret | Descrição |
|---|---|
| `SHEET_ID` | ID da planilha Google Sheets |
| `GOOGLE_SA_KEY` | JSON completo da Service Account GCP |

---

## 📊 Estrutura do Dashboard

### 1. Big Numbers
- **Ingressos Vendidos** — pedidos pagos únicos (product = masterclass)
- **Faturamento Masterclass** — soma de `Faturamento` (paid)
- **Investimento** — soma de `Spend` do Meta Ads no período
- **ROAS** — faturamento / investimento
- **CAC** — investimento / ingressos vendidos

### 2. Funil de Conversão
Todos os dados são do Meta Ads, exceto a última etapa (Kiwify):

```
Investimento (R$)
    ↓ impressões
Impressões → CPM
    ↓ CTR
Cliques → CPC
    ↓ % dos cliques
Página do Evento (LPV) → CPV
    ↓ % da página
Checkout Iniciado → CP Checkout
    ↓ % dos checkouts
Ingressos Vendidos → CAC
```

### 3. Evolução Diária
- Ingressos por dia (barras)
- Faturamento por dia (linha)
- Investimento por dia (linha)

### 4. Por Origem de Tráfego
- Tabela `utm_medium` × Adset com Investimento, Ingressos, Faturamento, ROAS, CAC
- Tabela `utm_content` × Ad Name com Impressões, CTR, Investimento, Ingressos, Faturamento, ROAS, CAC

### 5. Produto Principal Associado
Aparece apenas quando uma masterclass com produto principal associado está selecionada:
- Vendas do produto principal
- Faturamento, ticket médio, taxa de conversão (vendas produto / ingressos mc)
- ROAS combinado (fat. mc + fat. produto) / investimento

### 6. Outras Análises
- Origem de tráfego (Pago vs Orgânico)
- Transações por status (paid / refunded / waiting / refused)
- Método de pagamento

### 7. Vendas por Faixa de Preço (Lote)
- Confrades: R$ 80–100
- Alunos / ex-alunos: R$ 101–180
- Público geral: R$ 185+

---

## 🔽 Filtros disponíveis

| Filtro | Descrição |
|---|---|
| Masterclass | Filtra por campanha (lido dinamicamente da aba `TAG_CAMPANHA`) |
| Produto Principal | Filtra por produto associado |
| Data (de/até) | Filtra período de análise |

---

## 🚀 Deploy

1. Suba `index.html`, `worker.js` e `README.md` no repositório
2. Atualize o Worker no Cloudflare com o novo `worker.js` (adiciona aba `TAG_CAMPANHA`)
3. Ative o GitHub Pages: **Settings → Pages → branch `main` / root**

---

## 📦 Adicionar nova Masterclass

1. Adicione uma linha na aba `TAG_CAMPANHA` da planilha
2. O filtro do dashboard aparece automaticamente — nenhuma alteração no código necessária

---

**Agência B16** — [henrscard@gmail.com](mailto:henrscard@gmail.com)
