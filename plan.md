# DEFI$BUILD — Plano do Projeto

## Visão Geral

**Nome:** DEFI$BUILD  
**Tipo:** Dashboard Web Single-Page (HTML/CSS/JS puro — sem framework, sem backend)  
**Objetivo:** Painel completo de referência para traders e investidores DeFi brasileiros — conversão, cotações, simulação DCA, preços de crypto e glossários técnicos.  
**Público:** Trader DeFi, Analista de Criptoativos, investidor em BTC — nível intermediário a avançado.

---

## Estrutura de Módulos

### 1. SAT / BTC Converter
- Conversão bidirecional: Satoshis ↔ BTC ↔ BRL ↔ USD
- Cotação BTC ao vivo via CoinGecko API (free)
- Indicadores: BTC/BRL, BTC/USD, Market Cap

### 2. Cotação Dólar (USD/BRL)
- Cotação atual em tempo real (via AwesomeAPI ou ExchangeRate-API free)
- Tabela de dados históricos: últimos 30 dias (abertura, fechamento, máxima, mínima, variação %)
- Mini gráfico de linha (30 dias)

### 3. Conversor de Moedas Mundial
- Suporte a 30+ moedas (USD, BRL, EUR, GBP, JPY, ARS, etc.)
- Taxas ao vivo via ExchangeRate-API (free tier)
- Interface: campo origem → campo destino, swap instantâneo

### 4. Top 50 Criptomoedas por Market Cap
- Dados: rank, nome, símbolo, preço USD, variação 24h, market cap, volume
- Atualização via CoinGecko API
- Ordenação por colunas, busca por nome/símbolo
- Badge colorido: verde (positivo) / vermelho (negativo)

### 5. Simulador DCA — Bitcoin
- Inputs: valor mensal (BRL ou USD), data de início, frequência
- Cálculo: total investido, BTC acumulado, valor atual, ROI %
- Comparação com: S&P 500, Ouro, Ibovespa, Poupança (dados históricos estáticos)
- Gráfico de evolução patrimonial

### 6. Dicionário Bitcoin
- 100 termos: Bitcoin, mineração, Lightning Network, macroeconomia
- Busca instantânea por termo
- Categorias: Técnico, Econômico, Lightning, Mineração
- Exibição: card expansível com definição + exemplo de uso

### 7. Dicionário DeFi
- 100 termos DeFi: protocolo, AMM, yield farming, liquidez, etc.
- Mesma estrutura do Dicionário Bitcoin
- Categorias: Protocolo, Tokens, Liquidez, Risco, Governança

---

## Arquitetura Técnica

```
defi-build/
├── index.html          # Arquivo único (SPA com tabs)
├── plan.md             # Este arquivo
├── claude.md           # Contexto para IA
└── assets/
    ├── style.css       # Opcional se extraído
    └── app.js          # Opcional se extraído
```

> **Decisão:** Arquivo único `index.html` com CSS e JS inline — máxima portabilidade, zero dependências locais.

### APIs Utilizadas (todas free, sem autenticação obrigatória)

| Serviço | Endpoint | Dados |
|---|---|---|
| CoinGecko | `/api/v3/simple/price` | BTC/USD, BTC/BRL |
| CoinGecko | `/api/v3/coins/markets` | Top 50 por market cap |
| AwesomeAPI | `/json/last/USD-BRL` | USD/BRL atual |
| AwesomeAPI | `/json/daily/USD-BRL/30` | Histórico 30 dias |
| ExchangeRate-API | `/v6/latest/USD` | Todas as moedas |

### Dados Estáticos (sem API)
- Histórico S&P 500, Ouro, Ibovespa para DCA: array JS com retornos anuais históricos
- Dicionários Bitcoin e DeFi: objetos JS hardcoded no HTML

---

## Design System

**Tema:** Dark mode obrigatório — referência nos prints  
**Paleta:**
- Background: `#0a0a0a` / `#111111`
- Surface: `#1a1a1a` / `#1e1e1e`
- Accent BTC: `#f7931a` (laranja Bitcoin)
- Accent DeFi: `#00d4aa` (verde/teal)
- Texto primário: `#ffffff`
- Texto secundário: `#888888`
- Positivo: `#22c55e`
- Negativo: `#ef4444`

**Tipografia:**
- Display: `Space Mono` ou `JetBrains Mono` (dados financeiros)
- Body: `IBM Plex Sans`

**Componentes:**
- Cards com `border: 1px solid #2a2a2a` e `border-radius: 12px`
- Tabs de navegação no topo (sticky)
- Inputs com fundo `#111`, borda sutil, foco laranja

---

## Navegação (Tabs)

```
[₿ Converter] [$ Dólar] [🌍 Moedas] [📊 Top 50] [📈 DCA] [📖 Dic. BTC] [📖 Dic. DeFi]
```

---

## Cronograma de Desenvolvimento

| Fase | Entregável | Estimativa |
|---|---|---|
| 1 | Estrutura HTML + Design System + Navegação | 1h |
| 2 | Módulo BTC Converter + API CoinGecko | 1h |
| 3 | Módulo USD/BRL + histórico + gráfico | 1h |
| 4 | Conversor de moedas + Top 50 cripto | 1.5h |
| 5 | Simulador DCA completo com gráfico | 2h |
| 6 | Dicionários Bitcoin + DeFi (200 termos) | 1.5h |
| 7 | Polish, responsividade, error handling | 1h |
| **Total** | | **~9h** |

---

## Limitações Conhecidas

- CoinGecko free tier: 10-30 req/min — adicionar cache local (5 min)
- Dados históricos do DCA para ativos tradicionais são estáticos (não real-time)
- Sem backend = sem autenticação, sem carteiras salvas
- ExchangeRate-API free: 1.500 req/mês — suficiente para uso pessoal

---

## Métricas de Sucesso

- [ ] Conversão BTC/SAT/BRL/USD funcional com preço ao vivo
- [ ] Tabela USD/BRL últimos 30 dias com gráfico
- [ ] Conversor mundial com 30+ moedas
- [ ] Top 50 crypto atualizando ao abrir a aba
- [ ] DCA calculando ROI corretamente e mostrando gráfico
- [ ] 200 termos nos dicionários, busca < 100ms
- [ ] Carrega em < 3s em conexão 4G
