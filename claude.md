# DEFI$BUILD — Contexto para IA (claude.md)

## O que é este projeto

`DEFI$BUILD` é um dashboard web estático (single HTML file) voltado para traders DeFi e investidores em Bitcoin brasileiros. Funciona 100% no browser, sem backend, usando APIs públicas gratuitas.

---

## Stack e Decisões Técnicas

- **Linguagem:** HTML5 + CSS3 + JavaScript ES6+ vanilla
- **Arquivo:** `index.html` único (CSS e JS inline — portabilidade máxima)
- **Sem frameworks:** sem React, Vue, Angular — apenas JS puro
- **Gráficos:** Chart.js via CDN (`https://cdn.jsdelivr.net/npm/chart.js`)
- **Fontes:** Google Fonts — `JetBrains Mono` + `IBM Plex Sans`
- **Ícones:** Lucide via CDN ou SVG inline

---

## APIs e Endpoints

### CoinGecko (sem chave para free tier)
```
// Preço BTC
GET https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd,brl&include_market_cap=true

// Top 50 por market cap
GET https://api.coingecko.com/api/v3/coins/markets?vs_currency=usd&order=market_cap_desc&per_page=50&page=1&sparkline=false
```

### AwesomeAPI (sem chave)
```
// Cotação atual USD/BRL
GET https://economia.awesomeapi.com.br/json/last/USD-BRL

// Histórico 30 dias USD/BRL
GET https://economia.awesomeapi.com.br/json/daily/USD-BRL/30
```

### ExchangeRate-API (sem chave para open endpoint)
```
// Todas as taxas relativas ao USD
GET https://open.er-api.com/v6/latest/USD
```

---

## Estrutura de Módulos (IDs no HTML)

| Tab | ID do container | Função principal |
|---|---|---|
| Converter BTC | `#tab-converter` | SAT ↔ BTC ↔ BRL ↔ USD |
| Dólar | `#tab-dolar` | USD/BRL ao vivo + histórico 30d |
| Moedas | `#tab-moedas` | Conversor mundial 30+ moedas |
| Top 50 | `#tab-crypto` | Tabela market cap ao vivo |
| DCA | `#tab-dca` | Simulador Dollar Cost Averaging |
| Dic. BTC | `#tab-dic-btc` | Glossário 100 termos Bitcoin |
| Dic. DeFi | `#tab-dic-defi` | Glossário 100 termos DeFi |

---

## Padrões de Código

### Cache local (evitar rate limit)
```javascript
const CACHE_TTL = 5 * 60 * 1000; // 5 minutos
async function fetchWithCache(url, key) {
  const cached = localStorage.getItem(key);
  if (cached) {
    const { data, ts } = JSON.parse(cached);
    if (Date.now() - ts < CACHE_TTL) return data;
  }
  const res = await fetch(url);
  const data = await res.json();
  localStorage.setItem(key, JSON.stringify({ data, ts: Date.now() }));
  return data;
}
```

### Formatação de valores
```javascript
const fmtBRL = v => v.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
const fmtUSD = v => v.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
const fmtBTC = v => v.toFixed(8);
const fmtPct = v => `${v >= 0 ? '+' : ''}${v.toFixed(2)}%`;
```

### Estado da aplicação
```javascript
const state = {
  btcUsd: 0,
  btcBrl: 0,
  btcMarketCap: 0,
  usdBrl: 0,
  usdBrlHistory: [], // array de { date, close, open, high, low, pct }
  topCryptos: [],    // array CoinGecko markets
  currencies: {},    // objeto ExchangeRate
  activeTab: 'converter'
};
```

---

## Design Tokens (CSS Variables)

```css
:root {
  --bg-primary: #0a0a0a;
  --bg-surface: #141414;
  --bg-card: #1a1a1a;
  --bg-input: #111111;
  --border: #2a2a2a;
  --border-hover: #3a3a3a;
  
  --text-primary: #f5f5f5;
  --text-secondary: #888888;
  --text-muted: #555555;
  
  --accent-btc: #f7931a;
  --accent-defi: #00d4aa;
  --accent-blue: #3b82f6;
  
  --positive: #22c55e;
  --negative: #ef4444;
  --neutral: #888888;
  
  --font-mono: 'JetBrains Mono', monospace;
  --font-sans: 'IBM Plex Sans', sans-serif;
  
  --radius-sm: 8px;
  --radius-md: 12px;
  --radius-lg: 16px;
  
  --shadow: 0 4px 24px rgba(0,0,0,0.4);
}
```

---

## Glossário Bitcoin — Categorias e Amostra

**Categorias:** `bitcoin` | `mineracao` | `lightning` | `macro` | `tecnico`

Amostra de estrutura:
```javascript
const DICT_BTC = [
  {
    term: "UTXO",
    category: "tecnico",
    definition: "Unspent Transaction Output. Saldo de BTC que ainda não foi gasto. O modelo contábil do Bitcoin é baseado em UTXOs, diferente de saldos de conta.",
    example: "Quando você recebe 0.01 BTC, cria um UTXO de 0.01 BTC no seu endereço."
  },
  // ... 99 outros termos
];
```

---

## Glossário DeFi — Categorias e Amostra

**Categorias:** `protocolo` | `tokens` | `liquidez` | `risco` | `governanca`

```javascript
const DICT_DEFI = [
  {
    term: "AMM",
    category: "protocolo",
    definition: "Automated Market Maker. Protocolo que usa fórmulas matemáticas (ex: x*y=k) para precificar ativos em pools de liquidez, sem necessidade de order book.",
    example: "Uniswap V2 usa AMM com a fórmula de produto constante para todas as trocas."
  },
  // ... 99 outros termos
];
```

---

## Simulador DCA — Lógica de Cálculo

```javascript
// Dados históricos BTC/USD mensais (necessário array com preço no 1º dia de cada mês)
// Fonte: dados estáticos de 2015-2026

function simulateDCA(monthlyBRL, startYear, startMonth) {
  let totalInvested = 0;
  let totalBTC = 0;
  
  // Para cada mês de startDate até hoje:
  // 1. Pegar preço BTC/BRL naquele mês
  // 2. totalBTC += monthlyBRL / priceBTCBRL
  // 3. totalInvested += monthlyBRL
  
  const currentValueBRL = totalBTC * state.btcBrl;
  const roi = ((currentValueBRL - totalInvested) / totalInvested) * 100;
  
  return { totalInvested, totalBTC, currentValueBRL, roi };
}
```

---

## Notas para Desenvolvimento

1. **Rate limiting CoinGecko:** Se retornar 429, mostrar dados em cache com aviso "Dados em cache — atualizado há X min"
2. **AwesomeAPI histórico:** Retorna array ordenado do mais recente para o mais antigo — inverter antes de plotar
3. **DCA histórico:** Usar array JS estático com preços mensais BTC/BRL desde Jan/2015 — não depender de API
4. **Responsividade:** Layout de 1 coluna em mobile, 2-3 colunas em desktop
5. **Tab ativa:** Preservar no `location.hash` para que o link funcione diretamente em qualquer módulo
6. **Loading states:** Skeleton loaders nos cards enquanto API carrega — evitar tela em branco

---

## Checklist de QA antes de entregar

- [ ] Converter: digitar SAT atualiza BTC, BRL e USD em tempo real
- [ ] Dólar: tabela mostra 30 linhas com variação colorida
- [ ] Moedas: swap de moeda origem/destino funciona
- [ ] Top 50: variação 24h verde/vermelho correta
- [ ] DCA: ROI % bate com cálculo manual
- [ ] Busca nos dicionários: retorna resultados em < 100ms
- [ ] Sem erros de console em Chrome/Firefox
- [ ] Abre offline após primeira carga (cache localStorage)
