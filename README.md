# USDC/USDT Market-Making Trading Bot

Tento projekt implementuje market-making strategii pro pár **USDC/USDT** na Binance. Bot používá **Fill or Kill (FOK)** limitní příkazy k nákupu a prodeji aktiv na základě aktuálního spreadu mezi **bid** a **ask** cenou. Projekt využívá **CCXT (CryptoCurrency eXchange Trading Library)** a **asynchronní API volání** pro efektivní a rychlé obchodování.

---

## 📌 Požadavky

Před spuštěním tohoto bota je nutné mít:
- **Python 3.7+** nainstalovaný na systému
- **CCXT knihovnu** pro asynchronní obchodování
- **Binance API klíče** (v sandbox módu nebo live)

### 📦 Instalace závislostí

Před spuštěním nainstaluj potřebné knihovny:

```bash
pip install ccxt asyncio
```

---

## ⚙️ Konfigurace

V souboru **main.py** je nutné upravit následující sekce:

### 🔑 API Klíče

Zadej své **Binance API** klíče v části:

```python
api_key = "TVŮJ_API_KLÍČ"
api_secret = "TVŮJ_API_SECRET"
```

> **Upozornění**: Nikdy nesdílej své API klíče veřejně!

### 📡 Sandbox mód

Bot může běžet v **testovacím sandbox módu** nebo v live režimu:

```python
exchange.set_sandbox_mode(True)  # True = testovací režim, False = live obchodování
```

---

## 📜 Funkce

### 🔄 Automatické získávání tržních dat

Bot pravidelně načítá **order book** a získává nejlepší **bid** a **ask** ceny pro obchodování.

```python
async def get_best_bid_ask():
    order_book = await safe_api_call(exchange.fetch_order_book, symbol)
    if not order_book or not order_book['bids'] or not order_book['asks']:
        return None, None
    
    best_bid = order_book['bids'][0][0]
    best_ask = order_book['asks'][0][0]
    return best_bid, best_ask
```

### 🛒 Market-Making strategie

Bot provádí **limitní nákupní příkazy** na nejlepší **bid** ceně a **limitní prodejní příkazy** na nejlepší **ask** ceně s využitím strategie **Fill or Kill (FOK)**.

- Pokud je **spread dostatečně velký**, bot se pokusí provést obchod.
- Každý obchod má **dynamickou velikost** (5 % dostupného USDT, minimálně 1 USDC).
- Bot **nespekuluje** na cenu – snaží se pouze využít spread mezi bid/ask.

```python
buy_order = await safe_api_call(exchange.create_limit_buy_order, symbol, trade_size, best_bid, {'timeInForce': 'FOK'})
```

```python
sell_order = await safe_api_call(exchange.create_limit_sell_order, symbol, position["amount"], best_ask, {'timeInForce': 'FOK'})
```

### 🔄 Ochrana proti chybám

- **Safe API call** mechanismus s **automatickými retry pokusy** v případě chyb.
- **Rate limit ochrana** při příliš častých API voláních.
- **Logovací systém** zaznamenávající chyby a obchodní aktivity.

```python
async def safe_api_call(api_func, *args, **kwargs):
    retries = 3
    for attempt in range(retries):
        try:
            return await api_func(*args, **kwargs)
        except (ccxt.NetworkError, ccxt.ExchangeError) as e:
            logger.warning(f"API Chyba ({attempt + 1}/{retries}): {e}")
        await asyncio.sleep(2 ** attempt)
    return None
```

### 📝 Logování

Bot zaznamenává veškeré obchodní aktivity do **souboru** i **konzole**.

```python
logger = logging.getLogger("trader")
logger.setLevel(logging.DEBUG)
file_handler = RotatingFileHandler("trade_log.txt", maxBytes=10*1024*1024, backupCount=5)
```

---

## 🚀 Spuštění

Po úpravě konfigurace spustíš bota příkazem:

```bash
python main.py
```

> **Ukončení bota**: Stiskni **CTRL + C**.

---

## 📊 Statistiky

Po spuštění bot:
- Zobrazuje **aktuální zisk** v **USDT**.
- Počítá **celkový počet obchodů**.

```python
logger.info(f"Celkový zisk: {total_profit:.2f} USDT | Počet obchodů: {executed_trades}")
```

---

## 🔥 Možnosti rozšíření

- **Vylepšení market-making strategie** (dynamické rozpoznávání spreadu, hloubková analýza order booku)
- **Podpora více párů** (např. USDT/BUSD, BTC/USDT)
- **AI-driven predikce** a lepší alokace kapitálu

---

## ⚠️ Upozornění

Tento bot je určen pouze k **testovacím a vzdělávacím účelům**. Live obchodování je na vlastní **riziko**!

---

## 📌 Licence

Tento projekt je pod licencí **MIT** – můžeš ho používat, upravovat a sdílet volně.

