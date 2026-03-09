# 🚀 Критические Компоненты — Реализация

> **Дата:** 2026-03-09  
> **Статус:** ✅ Реализовано  
> **Спринт:** 1 из 3 (Критично)

---

## 📋 Обзор Реализации

В рамках **Sprint 1 (Критично)** реализованы следующие компоненты:

| Компонент | Файл | Статус | Описание |
|-----------|------|--------|----------|
| **Binance Client** | `src/lib/exchange/binance-client.ts` | ✅ | API клиент Binance Futures |
| **Signal Parser** | `src/lib/signal-parser.ts` | ✅ | Парсер сигналов Cornix |
| **Amount Calculator** | `src/lib/auto-trading/amount-per-trade.ts` | ✅ | 4 типа расчёта размера позиции |
| **Trailing Stop** | `src/lib/auto-trading/trailing-stop.ts` | ✅ | 5 типов Trailing Stop |
| **TP Strategy** | `src/lib/auto-trading/tp-strategy.ts` | ✅ | 9 стратегий Take-Profit |
| **Trade Open API** | `src/app/api/trade/open/route.ts` | ✅ | Endpoint для открытия сделок |


---

## 🏦 2. Binance Futures API Client

**Файл:** `src/lib/exchange/binance-client.ts`

### Реализованные методы

| Метод | Описание | Статус |
|-------|----------|--------|
| `placeOrder()` | Размещение ордера | ✅ |
| `getPosition()` | Получение позиции | ✅ |
| `closePosition()` | Закрытие позиции | ✅ |
| `getBalance()` | Получение баланса | ✅ |
| `setLeverage()` | Установка плеча | ✅ |
| `setMarginMode()` | ISOLATED/CROSSED | ✅ |
| `cancelOrder()` | Отмена ордера | ✅ |
| `cancelAllOrders()` | Отмена всех ордеров | ✅ |
| `getTickerPrice()` | Получение цены | ✅ |

### Пример использования

```typescript
import { BinanceClient } from '@/lib/exchange/binance-client';

const client = new BinanceClient({
  apiKey: 'your-api-key',
  apiSecret: 'your-api-secret',
  testnet: true, // Testnet для разработки
});

// Размещение ордера
const order = await client.placeOrder({
  symbol: 'BTC/USDT',
  side: 'LONG',
  type: 'MARKET',
  quantity: 0.001,
  leverage: 10,
});

// Получение позиции
const position = await client.getPosition('BTC/USDT');

// Закрытие позиции
await client.closePosition('BTC/USDT');
```

### Безопасность

- ✅ HMAC SHA256 подпись запросов
- ✅ recvWindow защита от replay attacks
- ✅ Testnet режим для разработки
- ✅ Обработка ошибок API

---

## 📡 3. Cornix Signal Parser

**Файл:** `src/lib/signal-parser.ts`

### Поддерживаемый формат

```
⚡⚡ #BTC/USDT ⚡⚡
Exchanges: Binance Futures
Signal Type: Regular (Long)
Leverage: Isolated (5X)

Entry Zone:
38766.9 - 38766.9

Take-Profit Targets:
1) 38766.9
2) 38766.9

Stop Targets:
1) 38766.9

Trailing Configuration:
Entry: Percentage (0.5%)
Take-Profit: Percentage (0.5%)
Stop: Moving Target - Trigger: Target (1)
```

### Распарсиваемые поля

| Поле | Формат | Статус |
|------|--------|--------|
| Symbol | `#BTC/USDT`, `#BTCUSDT` | ✅ |
| Exchange | `Binance Futures`, `Bybit Spot` | ✅ |
| Signal Type | `LONG`, `SHORT` | ✅ |
| Leverage | `Isolated (5X)`, `Cross (10X)` | ✅ |
| Entries | До 10 уровней | ✅ |
| Take-Profits | До 10 уровней | ✅ |
| Stop Loss | 1 уровень | ✅ |
| Trailing | 5 типов | ✅ |

### Пример использования

```typescript
import { cornixSignalParser } from '@/lib/signal-parser';

const signal = cornixSignalParser.parse(`
⚡⚡ #BTC/USDT ⚡⚡
Signal Type: Long
Leverage: Isolated (10X)
Entry Zone: 95000 - 95500
Take-Profit Targets:
1) 97000
2) 99000
Stop Targets:
1) 94000
`);

if (signal) {
  console.log('Символ:', signal.symbol);
  console.log('Тип:', signal.signalType);
  console.log('Плечо:', signal.leverage);
  console.log('TP:', signal.takeProfits);
  console.log('SL:', signal.stopLoss);
}
```

---

## 💰 4. Amount Per Trade Calculator

**Файл:** `src/lib/auto-trading/amount-per-trade.ts`

### 4 типа расчёта

| Тип | Формула | Статус |
|-----|---------|--------|
| **PERCENTAGE** | `(Total × %) / Entry` | ✅ |
| **RISK_PERCENTAGE** | `(Portfolio × Risk%) / ((Entry-SL)/Entry)` | ✅ |
| **FIXED_BTC** | `BTC Amount → Quote Asset` | ✅ |
| **FIXED_USD** | `USD Amount → Quote Asset` | ✅ |

### Пример использования

```typescript
import { amountPerTradeCalculator, AmountPerTradeConfig } from '@/lib/auto-trading/amount-per-trade';

// Конфигурация
const config: AmountPerTradeConfig = {
  type: 'RISK_PERCENTAGE',
  value: 2, // Риск 2% от портфеля
};

// Баланс
const balance = {
  available: 8000,
  inUse: 2000,
  total: 10000,
};

// Параметры сделки
const params = {
  entryPrice: 95000,
  stopLoss: 94000,
  symbol: 'BTC/USDT',
  quoteAsset: 'USDT',
};

// Расчёт
const result = amountPerTradeCalculator.calculate(config, balance, params);

console.log('Количество:', result.quantity); // 0.2 BTC
console.log('Сумма:', result.amount); // $19,000
console.log('Риск:', result.riskAmount); // $200 (2% от $10,000)
```

### Risk Percentage формула

```
Position Size = (Portfolio × Risk%) / ((Entry - SL) / Entry)

Пример:
- Portfolio: $10,000
- Risk: 2% = $200
- Entry: $95,000
- SL: $94,000
- Potential Loss: (95000-94000)/95000 = 1.05%

Position Size = $200 / 0.0105 = $19,047
Quantity = $19,047 / $95,000 = 0.2 BTC
```

---

## 🎯 5. Trailing Stop-Loss Manager

**Файл:** `src/lib/auto-trading/trailing-stop.ts`

### 5 типов Trailing Stop

| Тип | Описание | Статус |
|-----|----------|--------|
| **BREAKEVEN** | SL → Entry Price | ✅ |
| **MOVING_TARGET** | SL → Previous TP | ✅ |
| **MOVING_2_TARGET** | SL → 2 TPs Back | ✅ |
| **PERCENT_BELOW_TRIGGER** | SL = Trigger - X% | ✅ |
| **PERCENT_BELOW_HIGHEST** | SL = Max - X% | ✅ |

### Пример использования

```typescript
import { trailingStopManager, TrailingStopConfig } from '@/lib/auto-trading/trailing-stop';

// Конфигурация
const config: TrailingStopConfig = {
  type: 'MOVING_TARGET',
  trigger: {
    type: 'TARGET',
    value: 1, // После достижения TP1
  },
};

// Позиция
const position = {
  id: 'trade-123',
  symbol: 'BTC/USDT',
  side: 'LONG',
  entryPrice: 95000,
  currentPrice: 97000,
  stopLoss: 94000,
  takeProfits: [97000, 99000, 101000],
  filledEntries: [95000],
  quantity: 0.2,
  leverage: 10,
};

// Обновление SL
const result = await trailingStopManager.updateStopLoss(
  position,
  config,
  97000 // Текущая цена
);

if (result.updated) {
  console.log('Новый SL:', result.newStopLoss); // 95000 (Breakeven)
  console.log('Причина:', result.reason);
}
```

### WebSocket мониторинг

```typescript
import { trailingStopMonitor } from '@/lib/auto-trading/trailing-stop';

// Мониторинг позиции
await trailingStopMonitor.monitor(
  position,
  config,
  (symbol, callback) => {
    // Подписка на WebSocket биржи
    binanceWS.subscribe(symbol, callback);
  }
);

// Событие обновления SL
trailingStopMonitor.on('stopLossUpdated', (data) => {
  console.log(`SL обновлён: ${data.newStopLoss}`);
});
```

---

## 📊 6. Take-Profit Ratio Strategy

**Файл:** `src/lib/auto-trading/tp-strategy.ts`

### 9 стратегий распределения

| Стратегия | 4 TP Пример | Статус |
|-----------|-------------|--------|
| **EVENLY_DIVIDED** | 25% / 25% / 25% / 25% | ✅ |
| **ONE_TARGET** | 100% / 0% / 0% / 0% | ✅ |
| **TWO_TARGETS** | 50% / 50% / 0% / 0% | ✅ |
| **THREE_TARGETS** | 33.3% / 33.3% / 33.3% / 0% | ✅ |
| **FIFTY_ON_FIRST** | 50% / 16.7% / 16.7% / 16.7% | ✅ |
| **DECREASING_EXPONENTIAL** | 53.3% / 26.7% / 13.3% / 6.7% | ✅ |
| **INCREASING_EXPONENTIAL** | 6.7% / 13.3% / 26.7% / 53.3% | ✅ |
| **SKIP_FIRST** | 0% / 33.3% / 33.3% / 33.3% | ✅ |
| **CUSTOM** | Пользовательские % | ✅ |

### Пример использования

```typescript
import { takeProfitRatioCalculator, TakeProfitConfig } from '@/lib/auto-trading/tp-strategy';

// Конфигурация
const config: TakeProfitConfig = {
  strategy: 'FIFTY_ON_FIRST',
};

// TP уровни
const takeProfits = [97000, 99000, 101000, 103000];

// Общее количество
const totalQuantity = 0.2; // BTC

// Расчёт
const ratios = takeProfitRatioCalculator.calculate(
  config,
  takeProfits,
  totalQuantity
);

ratios.forEach(tp => {
  console.log(`TP${tp.tpIndex + 1}: ${tp.ratio}% = ${tp.quantity} BTC @ $${tp.tpPrice}`);
});

// Вывод:
// TP1: 50% = 0.1 BTC @ $97000
// TP2: 16.7% = 0.0334 BTC @ $99000
// TP3: 16.7% = 0.0334 BTC @ $101000
// TP4: 16.7% = 0.0334 BTC @ $103000
```

---

## 🛒 7. Trade Open API Endpoint

**Файл:** `src/app/api/trade/open/route.ts`

### Request

```typescript
POST /api/trade/open

{
  "userId": "user-123",
  "signalText": "⚡⚡ #BTC/USDT ⚡⚡\nLong\n...",
  
  // Или структурированно:
  "symbol": "BTC/USDT",
  "side": "LONG",
  "exchange": "Binance Futures",
  
  // Параметры
  "amountType": "RISK_PERCENTAGE",
  "amountValue": 2,
  "leverage": 10,
  "marginType": "ISOLATED",
  
  // Опционально
  "botConfigId": "config-456"
}
```

### Response

```typescript
{
  "success": true,
  "tradeId": "trade-789",
  "orderId": 12345678,
  "symbol": "BTC/USDT",
  "side": "LONG",
  "quantity": 0.2,
  "entryPrice": 95000,
  "takeProfits": [97000, 99000, 101000],
  "stopLoss": 94000
}
```

### Flow

```
1. Валидация запроса
       ↓
2. Парсинг сигнала (CornixParser)
       ↓
3. Получение настроек бота (Prisma)
       ↓
4. Расчёт баланса (PortfolioBalance)
       ↓
5. Расчёт размера позиции (AmountCalculator)
       ↓
6. Расчёт TP ratios (TPStrategy)
       ↓
7. Инициализация биржи (BinanceClient)
       ↓
8. Размещение ордера (placeOrder)
       ↓
9. Сохранение в БД (Prisma Trade)
       ↓
10. Настройка Trailing Stop
       ↓
11. Логирование (SystemLog)
```

---

## 📝 Следующие шаги

### Sprint 2 (Важно) — 4-6 недель

- [ ] **Bybit API Client** — вторая биржа
- [ ] **Bitget API Client** — вторая биржа
- [ ] **Bingx API Client** — вторая биржа
- [ ] **OKX API Client** — третья биржа
- [ ] **DCA Engine** — безопасность ордера
- [ ] **Grid Engine** — уровни сетки
- [ ] **Telegram Bot** — команды, уведомления
- [ ] **TradingView Webhook** — JSON формат

### Sprint 3 (Желательно) — 6-8 недель

- [ ] **Error Handling** — 44 типа ошибок Cornix
- [ ] **Backtesting** — исторические данные
- [ ] **Analytics** — PnL статистика
- [ ] **Multi-user** — NextAuth интеграция
- [ ] **Mobile App** — React Native

---

## 🧪 Тестирование

### Unit Tests

```bash
# Запуск тестов
bun run test

# Тесты для компонентов
bun run test src/lib/signal-parser.test.ts
bun run test src/lib/auto-trading/amount-per-trade.test.ts
bun run test src/lib/auto-trading/trailing-stop.test.ts
bun run test src/lib/auto-trading/tp-strategy.test.ts
```

### Integration Tests

```bash
# Тесты API endpoints
bun run test src/app/api/trade/open/route.test.ts

# Тесты Binance Client (testnet)
bun run test src/lib/exchange/binance-client.test.ts
```

### Manual Testing

2. **Signal Parser:** Протестировать 10+ сигналов
3. **Amount Calculator:** Все 4 типа расчёта
4. **Trailing Stop:** Все 5 типов
5. **TP Strategy:** Все 9 стратегий
6. **Trade Open:** Real trade на testnet

---

## 📞 Контакты

**Реализовал:** Qwen 3.5 Plus (Основной разработчик)  
**Дата:** 2026-03-09  
**Спринт:** 1 из 3 (Критично)  
**Статус:** ✅ Готово к тестированию

---

## 📄 Лицензия

MIT License — CITARION Project
