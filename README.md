# 🎼 Dashboard — Masterclass Alma Romântica

🔗 **[Acessar dashboard ao vivo](https://suporteb16-collab.github.io/dashboard-ar0526/)**

Dashboard de performance de vendas vs. investimento em Meta Ads para o produto **Masterclass Alma Romântica** (Escola do Ouvido / Confraria Musical Hélicon).

---

## 📁 Estrutura

```
dashboard-alma-romantica.html   ← Arquivo principal (standalone)
README.md                       ← Este arquivo
```

---

## 📊 Métricas e funcionalidades

### Cards de visão geral
| Métrica | Lógica |
|---|---|
| Total de Vendas | Contagem distinta de `order_ref` onde `order_status = paid` **e** `Product_product_name` contém "Alma Romântica" |
| Total de Faturamento | Soma da coluna `Faturamento` (KIWIFY) para `order_status = paid` e produto Alma Romântica |
| Total de Investimento | Soma de `Spend (Cost, Amount Spent)` (META ADS) no período |
| ROAS | `Faturamento pago / Investimento` — apenas onde `utm_medium = quente ou frio` |
| CAC | `Investimento / Vendas pagas` — apenas onde `utm_medium = quente ou frio` |

### Gráficos de evolução diária
- **Vendas por dia** — barras, contagem distinta de `order_ref`
- **Faturamento por dia** — linha
- **Investimento por dia** — linha (Meta Ads)
- **CAC por dia** — linha com sobreposição de investimento

### Tabelas cruzadas
1. **Por utm_medium / Adset Name** — Investimento, Vendas, Faturamento, ROAS, CAC
2. **Por utm_content / Ad Name** — Impressões, CTR, Investimento, Vendas, Faturamento, ROAS, CAC
3. **Por lote de preço**:
   - Confrades: `Faturamento` entre R$ 80 e R$ 100
   - Alunos / ex-alunos: `Faturamento` entre R$ 101 e R$ 180
   - Público geral: `Faturamento` ≥ R$ 185

### Outros gráficos (rosca)
- **Origem de tráfego** — Pago (quente/frio) vs. Orgânico
- **Transações por status** — paid / refunded / waiting_payment / refused
- **Aprovação de cartão** — apenas transações com `payment_method = credit_card`

### Filtro de período
Controle de data de início e fim aplicado sobre ambas as bases (KIWIFY e META ADS).

---

## ⚙️ Arquitetura — Cloudflare Worker

Os dados são buscados via **Cloudflare Worker** (proxy seguro), que autentica na Google Sheets API com uma Service Account. A planilha pode permanecer **privada**.

**Worker:** `https://raspy-bush-2c66.henrscard.workers.dev`

**Secrets configurados no Worker:**
| Secret | Valor |
|---|---|
| `SHEET_ID` | ID da planilha Google |
| `GOOGLE_SA_KEY` | JSON completo da Service Account |

**Service Account com acesso à planilha (Visualizador):**
`cromador-dashboard@analise-de-dados-mallet.iam.gserviceaccount.com`

A planilha **não precisa ser pública**. O Worker autentica via JWT gerado a partir da chave da Service Account.

---

## 🚀 Deploy

### Cloudflare Pages

1. Faça o push do repositório para o GitHub (veja abaixo)
2. Acesse [pages.cloudflare.com](https://pages.cloudflare.com)
3. Clique em **Create a project → Connect to Git**
4. Selecione o repositório
5. Configurações de build:
   - **Framework preset:** None
   - **Build command:** *(vazio)*
   - **Build output directory:** `/` (raiz)
6. Clique em **Save and Deploy**

O arquivo `dashboard-alma-romantica.html` será servido automaticamente. Acesse via URL fornecida pela Cloudflare (ex: `alma-romantica.pages.dev`).

Para usar como `index.html`, renomeie o arquivo ou crie um redirect.

---

### GitHub

```bash
# Inicializar repositório
git init
git add .
git commit -m "feat: dashboard Alma Romântica"

# Criar repositório no GitHub (via CLI ou interface)
gh repo create alma-romantica-dashboard --public --source=. --push

# Ou via remote manual:
git remote add origin https://github.com/SEU_USUARIO/alma-romantica-dashboard.git
git branch -M main
git push -u origin main
```

Para atualizar após mudanças:
```bash
git add .
git commit -m "update: ajuste no dashboard"
git push
```

---

## 🗂️ Estrutura das abas esperadas na planilha

### Aba `KIWIFY`
Colunas relevantes utilizadas:
- `order_ref` — identificador único do pedido
- `order_status` — filtro principal (`paid`)
- `Faturamento` — valor em BRL (formato `R$ 185,64`)
- `Data de Criação` — data da transação
- `TrackingParameters_utm_medium` — origem: `quente`, `frio`, ou orgânico
- `TrackingParameters_utm_content` — identificador do anúncio
- `payment_method` — `credit_card`, `pix`, etc.
- `card_type`, `card_rejection_reason`

### Aba `META ADS`
Colunas relevantes:
- `Date` — data do gasto
- `Adset Name` — conjunto de anúncios (cruzado com `utm_medium`)
- `Ad Name` — anúncio (cruzado com `utm_content`)
- `Spend (Cost, Amount Spent)` — valor gasto em BRL
- `Impressions` — impressões
- `Clicks` — cliques (usado para calcular CTR)

---

## 🛠️ Personalização

Para trocar a planilha, edite a linha no `<script>` do HTML:
```js
const SHEET_ID = '1bPxq4nmFicmJihL1uKtnQMcgXSD0hYIZwgL14xPSL-s';
```

Para ajustar os intervalos de lote:
```js
function getLote(fat) {
  if (fat >= 80 && fat <= 100) return 'Confrades';
  if (fat >= 101 && fat <= 180) return 'Alunos / ex-alunos';
  if (fat >= 185) return 'Público geral';
  return 'Outro';
}
```

---

## 📝 Observações

- O dashboard lê dados **em tempo real** da planilha sempre que é aberto ou o filtro é aplicado
- Não há backend — tudo roda no browser do cliente
- O cruzamento KIWIFY × META ADS por `utm_medium`/`utm_content` depende de correspondência exata de nomes entre as colunas
- Linhas com `order_ref` duplicado são deduplicadas (conta apenas a mais recente com `status = paid`)
- Registros sem `Faturamento` ou com `#VALUE!` são ignorados

---

*Desenvolvido para: Escola do Ouvido / Confraria Musical Hélicon — maio de 2026*
