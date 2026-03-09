# 📊 CITARION vs Cornix: Детальный Анализ Соответствия

> **Дата анализа:** 09.03.2026  
> **Источник:** Cornix Knowledge Base (237 файлов)  
> **Статус:** 🔴 Критический аудит функциональности

---

## 🎯 Резюме

| Категория | Реализовано | Требуется | Соответствие |
|-----------|-------------|-----------|--------------|
| **Биржи** | 11 | 10 | ✅ 110% |
| **Сигналы (парсинг)** | Частично | Полно | ⚠️ 60% |
| **Настройки бота** | UI готово | Логика | ⚠️ 50% |
| **DCA** | UI готово | Движок | 🔴 30% |
| **Grid** | UI готово | Движок | 🔴 30% |
| **Trailing Stop** | UI готово | 5 типов | 🔴 20% |
| **Take-Profit стратегии** | UI готово | 9 стратегий | 🔴 25% |
| **Telegram интеграция** | Webhook | Полно | ⚠️ 40% |
| **TradingView** | Endpoint | Полно | ⚠️ 50% |
| **Real Trading** | ❌ Нет | Критично | 🔴 0% |

---

## 📋 1. Поддерживаемые Биржи

### Cornix официально поддерживает (10 бирж):

| Тип | Биржи |
|-----|-------|
| **Spot** | Binance, ByBit, OKX, Bitget, KuCoin, BingX, Coinbase, Huobi |
| **Futures** | Binance, ByBit, OKX, Bitget, KuCoin, BingX, HyperLiquid, BloFin |
| **Inverse** | Binance, ByBit, OKX, Bitget, BitMEX, BloFin |

### ❌ НЕ поддерживаются Cornix:
- `okx.eu`, `okx.us`
- `bybit.eu`
- `binance.us`

### CITARION реализовано (11 бирж):

| Биржа | Spot | Futures | Inverse | Статус |
|-------|------|---------|---------|--------|
| Binance | ✅ | ✅ | ✅ | ✅ Полностью |
| Bybit | ✅ | ✅ | ✅ | ✅ Полностью |
| OKX | ✅ | ✅ | ✅ | ⚠️ Нет фильтрации okx.eu/us |
| Bitget | ✅ | ✅ | ✅ | ✅ Полностью |
| KuCoin | ✅ | ✅ | - | ✅ Полностью |
| BingX | ✅ | ✅ | - | ✅ Полностью |
| Coinbase | ✅ | - | - | ✅ Полностью |
| Huobi | ✅ | - | - | ✅ Полностью |
| HyperLiquid | - | ✅ | - | ✅ Полностью |
| BitMEX | - | - | ✅ | ✅ Полностью |
| BloFin | - | ✅ | ✅ | ✅ Полностью |

### 🎯 Вывод по биржам:
✅ **CITARION поддерживает даже больше бирж, чем Cornix** (11 vs 10)

⚠️ **Рекомендация:** Добавить валидацию для исключения неподдерживаемых доменов (okx.eu, bybit.eu, binance.us)

---

## 📋 2. Формат Сигналов (Signal Posting Format)

### Cornix требует:

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

### Требования Cornix:
- ✅ До 10 Take-Profit уровней
- ✅ До 10 Entry уровней
- ✅ Stop Loss — всегда один уровень
- ✅ Все цены указываются в **ценах**, не в процентах
- ✅ Trailing Configuration должен быть в точном формате
- ✅ Сигналы должны быть в **plain text** формате

### CITARION реализовано:

| Функция | Статус | Файл |
|---------|--------|------|
| Парсинг сигналов | ⚠️ Частично | `src/lib/signal-parser.ts` |
| Поддержка хештегов | ✅ Да | `#BTCUSDT`, `#LONG` |
| Multiple TP | ✅ UI готово | `src/components/bot/bot-config-form.tsx` |
| Multiple Entries | ✅ UI готово | `src/components/bot/bot-config-form.tsx` |
| Trailing формат | 🔴 Не реализовано | - |
| Валидация формата | 🔴 Нет | - |

### 🎯 Вывод по сигналам:
⚠️ **Парсинг работает только для базовых сигналов**

🔴 **Критично:** Trailing Configuration не парсится согласно формату Cornix

### 📝 Требуется реализовать:

```typescript
// src/lib/signal-parser.ts
interface CornixSignal {
  symbol: string;           // #BTC/USDT
  exchange: string;         // Binance Futures
  signalType: 'LONG' | 'SHORT';
  leverage: {
    type: 'Isolated' | 'Cross';
    value: number;          // 5X
  };
  entries: {
    min: number;
    max: number;
  }[];
  takeProfits: number[];    // До 10 уровней
  stopLoss: number;         // Один уровень
  trailing?: {
    entry?: { type: string; value: number };
    takeProfit?: { type: string; value: number };
    stop: {
      type: 'Moving Target' | 'Moving 2-Target' | ...;
      trigger: number;
    };
  };
}
```

---

## 📋 3. Настройки Сигналов Бота (Signals Bot Advanced Settings)

### Cornix Advanced Settings (5 вкладок):

| Вкладка | Параметры | CITARION |
|---------|-----------|----------|
| **General** | Symbols filter, Direction, Leverage | ✅ UI готово |
| **Entries** | Personal/Channel, Entry strategy | ⚠️ UI есть, логики нет |
| **Take-Profits** | Personal/Channel, TP ratios | ⚠️ UI есть, логики нет |
| **Stop** | Stop-Loss, Trailing (5 типов) | 🔴 UI частично |
| **Advanced** | Override, Filters, Notifications | ⚠️ UI готово |

### 🔴 Критические несоответствия:

#### 3.1 Amount Per Trade (4 типа)

Cornix поддерживает:
1. **Percentage** — % от всего портфеля (available + in use)
2. **Risk Percentage** — % риска от входа до SL
3. **Fixed BTC Amount** — Фиксированная сумма в BTC
4. **Fixed USD Amount** — Фиксированная сумма в USD

**CITARION:** Только Percentage (UI есть, расчёт Risk % не реализован)

#### 3.2 Leverage Settings

Cornix поддерживает:
- **Global Settings** — для всех символов
- **Custom Symbols** — индивидуально на символ
- **Margin Type:** Cross / Isolated
- **Multiplier:** "Up to X" / "Exactly X"

**CITARION:** Только глобальное плечо (нет индивидуальных настроек на символ)

#### 3.3 Take-Profit Ratios Strategy (9 стратегий)

Cornix поддерживает:
1. Evenly Divided (по умолчанию)
2. One Target (100% на первый TP)
3. Two Targets (50%/50%)
4. Three Targets (33.33% каждый)
5. Fifty On First Target (50% + остальные поровну)
6. Decreasing Exponential (53.3% → 26.7% → 13.3% → 6.7%)
7. Increasing Exponential (6.7% → 13.3% → 26.7% → 53.3%)
8. Skip First (0% первый, остальные поровну)
9. Custom Strategy (своя стратегия)

**CITARION:** Только UI без логики распределения

### 📝 Требуется реализовать:

```typescript
// src/lib/auto-trading/tp-strategy.ts
enum TakeProfitStrategy {
  EVENLY_DIVIDED = 'EVENLY_DIVIDED',
  ONE_TARGET = 'ONE_TARGET',
  TWO_TARGETS = 'TWO_TARGETS',
  THREE_TARGETS = 'THREE_TARGETS',
  FIFTY_ON_FIRST = 'FIFTY_ON_FIRST',
  DECREASING_EXPONENTIAL = 'DECREASING_EXPONENTIAL',
  INCREASING_EXPONENTIAL = 'INCREASING_EXPONENTIAL',
  SKIP_FIRST = 'SKIP_FIRST',
  CUSTOM = 'CUSTOM'
}

function calculateTPRatios(
  strategy: TakeProfitStrategy,
  tpCount: number,
  customRatios?: number[]
): number[] {
  // Реализация 9 стратегий
}
```

---

## 📋 4. Trailing Stop-Loss (5 типов)

### Cornix поддерживает 5 типов Trailing Stop:

| Тип | Описание | Trigger | CITARION |
|-----|----------|---------|----------|
| **Breakeven** | Перемещает SL в цену входа | Target или % | 🔴 Нет |
| **Moving Target** | SL перемещается к предыдущему TP | Target (по умолчанию #1) | 🔴 Нет |
| **Moving 2-Target** | SL перемещается на 2 TP назад | Target (по умолчанию #2) | 🔴 Нет |
| **Percent Below Trigger** | SL на X% ниже триггера | Target или % | 🔴 Нет |
| **Percent Below Highest** | SL на X% ниже максимума | Target или % | 🔴 Нет |

### CITARION реализовано:
- ✅ UI для Trailing (вкл/выкл, процент)
- 🔴 **Ни один из 5 типов не реализован**
- 🔴 Нет мониторинга цены в реальном времени
- 🔴 Нет WebSocket подключения к бирже

### 📝 Требуется реализовать:

```typescript
// src/lib/auto-trading/trailing-stop.ts
enum TrailingStopType {
  BREAKEVEN = 'BREAKEVEN',
  MOVING_TARGET = 'MOVING_TARGET',
  MOVING_2_TARGET = 'MOVING_2_TARGET',
  PERCENT_BELOW_TRIGGER = 'PERCENT_BELOW_TRIGGER',
  PERCENT_BELOW_HIGHEST = 'PERCENT_BELOW_HIGHEST'
}

interface TrailingStopConfig {
  type: TrailingStopType;
  trigger: {
    type: 'TARGET' | 'PERCENT';
    value: number; // номер TP или процент
  };
  trailingPercent?: number; // для Percent Below типов
}

class TrailingStopManager {
  private highestPrice: number = 0;
  
  async updateStopLoss(
    positionId: string,
    currentPrice: number,
    config: TrailingStopConfig,
    takeProfits: number[],
    entryPrice: number
  ): Promise<void> {
    // Логика для каждого из 5 типов
  }
}
```

---

## 📋 5. DCA (Dollar Cost Averaging)

### Cornix DCA параметры:

| Параметр | Описание | CITARION |
|----------|----------|----------|
| **Safety Orders** | Дополнительные ордера при падении | 🔴 Нет |
| **Safety Order Step** | Шаг между ордерами (%) | 🔴 Нет |
| **Safety Order Scale** | Масштабирование размера ордеров | 🔴 Нет |
| **Max Safety Orders** | Максимальное количество | 🔴 Нет |
| **TP per Level** | Тейк-профит на каждый уровень | 🔴 Нет |

### CITARION реализовано:
- ✅ UI для DCA в `bot-config-form.tsx`
- ✅ Поля: DCA Enabled, DCA Levels, DCA Scale
- 🔴 **Движок DCA не реализован**
- 🔴 Нет мониторинга для открытия safety ордеров
- 🔴 Нет расчёта усреднённой цены входа

### 📝 Требуется реализовать:

```typescript
// src/lib/dca-bot/dca-bot-engine.ts
interface DCAConfig {
  enabled: boolean;
  maxSafetyOrders: number;      // 3-10
  safetyOrderStepPercent: number; // 2%
  safetyOrderScale: number;     // 1.5 (каждый следующий в 1.5 раза больше)
  takeProfitPerLevel: boolean;  // TP на каждый уровень
}

interface DCAPosition {
  baseOrder: Order;
  safetyOrders: Order[];
  averageEntryPrice: number;
  totalInvested: number;
}

class DCAEngine {
  async openSafetyOrder(
    positionId: string,
    currentPrice: number,
    config: DCAConfig
  ): Promise<Order> {
    // Расчёт размера ордера с учётом scale
    // Обновление averageEntryPrice
  }
}
```

---

## 📋 6. Grid Trading

### Cornix Grid параметры:

| Параметр | Описание | CITARION |
|----------|----------|----------|
| **Grid Range** | Верхняя/нижняя граница | 🔴 Нет |
| **Grid Levels** | Количество уровней | 🔴 Нет |
| **Grid Type** | Arithmetic / Geometric | 🔴 Нет |
| **Profit per Grid** | Прибыль на уровень (%) | 🔴 Нет |
| **Trailing Grid** | Трейлинг сетки | 🔴 Нет |

### CITARION реализовано:
- ✅ UI для Grid Bot в `grid-bot-manager.tsx`
- ✅ Кнопки: Pause, Resume
- 🔴 **Движок Grid не реализован**
- 🔴 Нет расчёта уровней сетки
- 🔴 Нет автоматического размещения ордеров

### 📝 Требуется реализовать:

```typescript
// src/lib/grid-bot/grid-bot-engine.ts
interface GridConfig {
  lowerPrice: number;
  upperPrice: number;
  gridLevels: number;
  gridType: 'ARITHMETIC' | 'GEOMETRIC';
  profitPerGridPercent: number;
  trailingEnabled: boolean;
}

interface GridLevel {
  price: number;
  orderType: 'BUY' | 'SELL';
  orderId?: string;
  filled?: boolean;
}

class GridEngine {
  private levels: GridLevel[] = [];
  
  async initializeGrid(config: GridConfig): Promise<void> {
    // Расчёт уровней сетки
    // Размещение начальных ордеров
  }
  
  async onOrderFilled(orderId: string): Promise<void> {
    // Размещение противоположного ордера
    // Обновление PnL
  }
}
```

---

## 📋 7. Telegram Интеграция

### Cornix Telegram функционал:

| Функция | Описание | CITARION |
|---------|----------|----------|
| **Bot Login** | Вход через Telegram | 🔴 Нет |
| **Signal Parsing** | Парсинг из каналов | ⚠️ Webhook есть |
| **Admin Commands** | /start, /stop, /status | 🔴 Нет |
| **Notifications** | Уведомления о сделках | 🔴 Нет |
| **User Management** | Управление пользователями | 🔴 Нет |

### CITARION реализовано:
- ✅ Webhook endpoint: `/api/telegram/webhook`
- ✅ Settings panel: `telegram-settings.tsx`
- ✅ Telegraf библиотека в dependencies
- 🔴 **Webhook не обрабатывает сигналы**
- 🔴 Нет команд бота
- 🔴 Нет авторизации через Telegram

### 📝 Требуется реализовать:

```typescript
// src/lib/telegram-bot-v2.ts
import { Telegraf, Context } from 'telegraf';

class TelegramBot {
  private bot: Telegraf<Context>;
  
  async start(): Promise<void> {
    this.bot.start((ctx) => ctx.reply('Welcome to CITARION!'));
    this.bot.command('status', this.handleStatus.bind(this));
    this.bot.command('settings', this.handleSettings.bind(this));
    this.bot.on('message', this.handleSignal.bind(this));
  }
  
  private async handleSignal(ctx: Context): Promise<void> {
    const message = ctx.message?.text;
    if (message) {
      const signal = await parseSignal(message);
      // Исполнение сигнала
    }
  }
}
```

---

## 📋 8. TradingView Webhooks

### Cornix TradingView Bot:

| Функция | Описание | CITARION |
|---------|----------|----------|
| **Webhook URL** | Уникальный URL на бота | ⚠️ Endpoint есть |
| **JSON Parsing** | Парсинг JSON от TV | 🔴 Нет |
| **Alert Mapping** | Привязка алертов к боту | 🔴 Нет |
| **Message Format** | Специальный формат | 🔴 Нет |

### CITARION реализовано:
- ✅ Endpoint: `/api/webhook/tradingview`
- ✅ `tradingview-parser.ts` (базовый парсер)
- 🔴 **Нет документации по формату JSON**
- 🔴 Нет привязки к конкретным ботам

### 📝 Требуется реализовать:

```typescript
// src/lib/tradingview-parser.ts
interface TradingViewAlert {
  bot_id: string;
  action: 'OPEN' | 'CLOSE' | 'UPDATE';
  symbol: string;
  side: 'LONG' | 'SHORT';
  entryPrice?: number;
  takeProfit?: number[];
  stopLoss?: number;
  leverage?: number;
}

async function parseTradingViewWebhook(
  payload: TradingViewAlert
): Promise<Signal> {
  // Валидация bot_id
  // Парсинг параметров
  // Исполнение через соответствующий бот
}
```

---

## 📋 9. Real Trading (Самое Критичное!)

### Cornix:
- ✅ Полная интеграция с API бирж
- ✅ Размещение реальных ордеров
- ✅ Синхронизация позиций
- ✅ Обработка ошибок API

### CITARION:
- 🔴 **ТОЛЬКО PAPER TRADING**
- 🔴 Нет реальных API ключей в работе
- 🔴 Нет размещения ордеров на биржах
- ✅ UI для подключения бирж есть
- ✅ API endpoints есть (`/api/exchange/connect`)

### 📝 Критично реализовать:

```typescript
// src/lib/exchange/binance-client.ts
class BinanceClient {
  private apiKey: string;
  private apiSecret: string;
  
  async placeOrder(params: OrderParams): Promise<Order> {
    // Подпись запроса (HMAC SHA256)
    // Отправка на Binance API
    // Обработка ошибок
  }
  
  async getPosition(symbol: string): Promise<Position> {
    // Получение позиции с биржи
  }
  
  async closePosition(positionId: string): Promise<void> {
    // Закрытие позиции на бирже
  }
}
```

---

## 📋 10. Ошибки и Уведомления

### Cornix имеет 44 типа ошибок:

| Категория | Примеры | CITARION |
|-----------|---------|----------|
| **Funds** | Not Enough Funds, Max Allowed Size | 🔴 Нет |
| **Exchange** | Rejected by Exchange, Canceled on Exchange | 🔴 Нет |
| **KYC** | KYC is Required | 🔴 Нет |
| **Limits** | Risk Limit Exceeded, Orders Exceeded | 🔴 Нет |
| **Liquidation** | Position Liquidated | 🔴 Нет |

### CITARION:
- ✅ Базовая обработка ошибок в API
- 🔴 Нет классификации ошибок по типу Cornix
- 🔴 Нет уведомлений пользователю

---

## 🎯 Итоговая Таблица Соответствия

| Компонент | Реализовано | Требуется | Приоритет |
|-----------|-------------|-----------|-----------|
| **Биржи (API)** | 11/10 | Валидация доменов | 🔴 Критично |
| **Real Trading** | 0% | Полная интеграция | 🔴 Критично |
| **Signal Parser** | 60% | Trailing, валидация | 🔴 Критично |
| **Amount Per Trade** | 25% (1/4 типа) | Risk %, Fixed BTC/USD | 🟡 Важно |
| **Leverage Settings** | 30% | Custom Symbols | 🟡 Важно |
| **TP Strategies** | 10% (UI) | 9 стратегий | 🟡 Важно |
| **Trailing Stop** | 0% (UI есть) | 5 типов + мониторинг | 🟡 Важно |
| **DCA Engine** | 0% (UI есть) | Safety orders | 🟡 Важно |
| **Grid Engine** | 0% (UI есть) | Уровни, ордера | 🟡 Важно |
| **Telegram Bot** | 20% (webhook) | Commands, login | 🟢 Желательно |
| **TradingView** | 50% (endpoint) | JSON format, mapping | 🟢 Желательно |
| **Error Handling** | 30% | 44 типа ошибок | 🟢 Желательно |

---

## 🚀 План Доработки (Roadmap)

### 🔴 Sprint 1: Критично (2-4 недели)

1. **Real Trading Integration**
   - [ ] Binance API клиент (placeOrder, getPosition, closePosition)
   - [ ] Bybit API клиент
   - [ ] Шифрование API ключей (AES-256)
   - [ ] Обработка ошибок API

2. **Signal Parser Enhancement**
   - [ ] Парсинг Trailing Configuration
   - [ ] Валидация формата Cornix
   - [ ] Поддержка 10 TP и 10 Entry

3. **Amount Per Trade**
   - [ ] Risk Percentage расчёт
   - [ ] Fixed BTC Amount
   - [ ] Fixed USD Amount

### 🟡 Sprint 2: Важно (4-6 недель)

4. **Trailing Stop Engine**
   - [ ] 5 типов trailing stop
   - [ ] WebSocket мониторинг цены
   - [ ] Автоматическое обновление SL

5. **TP Strategies**
   - [ ] 9 стратегий распределения
   - [ ] Custom strategy editor

6. **DCA Engine**
   - [ ] Safety orders логика
   - [ ] Усреднение цены входа
   - [ ] TP per level

7. **Grid Engine**
   - [ ] Расчёт уровней сетки
   - [ ] Автоматические ордера
   - [ ] Trailing grid

### 🟢 Sprint 3: Желательно (6-8 недель)

8. **Telegram Bot**
   - [ ] Команды (/start, /stop, /status)
   - [ ] Уведомления о сделках
   - [ ] Авторизация через Telegram

9. **TradingView Integration**
   - [ ] JSON format документация
   - [ ] Mapping алертов к ботам

10. **Error Handling**
    - [ ] 44 типа ошибок Cornix
    - [ ] Уведомления пользователю

---

## 📊 Оценка Готовности

```
Общая готовность CITARION к production: 35%

├─ Frontend (UI/UX)          ████████████████░░░░ 80%
├─ Backend (API)             ████████████░░░░░░░░ 60%
├─ Database (Prisma)         ████████████████░░░░ 85%
├─ Exchange Integration      ████░░░░░░░░░░░░░░░░ 20%
├─ Signal Parsing            ████████░░░░░░░░░░░░ 40%
├─ Trading Engines           ████░░░░░░░░░░░░░░░░ 20%
├─ Risk Management           ████████████░░░░░░░░ 60%
├─ Telegram/Discord          ████░░░░░░░░░░░░░░░░ 20%
└─ Testing/Monitoring        ██████░░░░░░░░░░░░░░ 30%
```

---

## 💡 Рекомендации

### Немедленно (эта неделя):

1. **Исправить MCP Filesystem путь** (всё ещё `/Users` вместо `C:\Users\Trading\CITARION`)
2. **Аудит `.env`** на наличие секретов перед коммитом
3. **Добавить `.mcpignore`** для ускорения работы MCP

### Краткосрочно (1 месяц):

4. **Реализовать Binance API** для реальной торговли
5. **Доработать парсер сигналов** под формат Cornix
6. **Добавить Risk %** расчёт для Amount Per Trade

### Долгосрочно (3 месяца):

7. **Полная реализация 5 типов Trailing Stop**
8. **DCA и Grid движки**
9. **Telegram бот с командами**

---

## 📞 Контакты

**CITARION Audit Report** © 2026  
**Аудит проведён:** Qwen 3.5 Plus (Основной разработчик)  
**На основе:** Cornix Knowledge Base (237 файлов)

---

## 📄 Источники

- [Cornix Supported Exchanges](https://help.cornix.io/en/articles/5814836-supported-exchanges)
- [Signal Posting Format](https://help.cornix.io/en/articles/11659507-signal-posting-format)
- [What Is the Cornix Signals Bot?](https://help.cornix.io/en/articles/8800202-what-is-the-cornix-signals-bot)
- [Signals Bot Advanced Settings](https://help.cornix.io/en/articles/8800218-signals-bot-advanced-settings-overview)
- [Amount Per Trade](https://help.cornix.io/en/articles/5814867-amount-per-trade)
- [Leverage](https://help.cornix.io/en/articles/5814858-leverage)
- [Trailing Stop-loss](https://help.cornix.io/en/articles/5814874-trailing-stop-loss)
- [Take-Profit Ratios Strategy](https://help.cornix.io/en/articles/5814868-take-profit-ratios-strategy)
- [DCA Strategy](https://help.cornix.io/en/articles/6062657-dca-dollar-cost-averaging-strategy)
- [Grid Trading](https://help.cornix.io/en/articles/8253570-what-is-grid-trading)
