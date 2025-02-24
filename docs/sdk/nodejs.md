# Node.js SDK

## Overview

OKX предоставляет мощный и современный Node.js SDK для работы с V5 API. SDK написан на TypeScript и предоставляет полную типизацию для всех API запросов и ответов.

## Features

- Полная интеграция со всеми OKX V5 API эндпоинтами
- TypeScript поддержка с типами для большинства API запросов и ответов
- Более 100 end-to-end тестов
- Надежная WebSocket интеграция:
  - Настраиваемые heartbeat для определения проблем с соединением
  - Автоматическое переподключение и переподписка
  - Автоматическая аутентификация и обработка heartbeat
- Поддержка браузера через webpack bundle

## Installation

Требования:
- Node.js >= 12
- npm или yarn

```bash
npm install okx-api
# или
yarn add okx-api
```

## REST API Examples

### Basic Setup

```typescript
import { RestClient } from 'okx-api';

// Для live trading
const client = new RestClient({
  apiKey: 'YOUR-API-KEY',
  apiSecret: 'YOUR-API-SECRET',
  apiPass: 'YOUR-API-PASSPHRASE'
});

// Для demo trading
const testnetClient = new RestClient({
  apiKey: 'YOUR-API-KEY',
  apiSecret: 'YOUR-API-SECRET',
  apiPass: 'YOUR-API-PASSPHRASE',
  testnet: true
});
```

### Market Data

```typescript
// Получение тикера
const ticker = await client.getTickerInfo({
  instId: 'BTC-USDT'
});

// Получение книги ордеров
const orderbook = await client.getOrderBook({
  instId: 'BTC-USDT',
  sz: '20'
});

// Получение исторических свечей
const candles = await client.getCandles({
  instId: 'BTC-USDT',
  bar: '1D'
});
```

### Trading

```typescript
// Размещение ордера
const order = await client.placeOrder({
  instId: 'BTC-USDT',
  tdMode: 'cash',
  side: 'buy',
  ordType: 'limit',
  px: '20000',
  sz: '0.01'
});

// Отмена ордера
const cancelled = await client.cancelOrder({
  instId: 'BTC-USDT',
  ordId: order.ordId
});

// Получение открытых ордеров
const orders = await client.getOrderList({
  instType: 'SPOT',
  instId: 'BTC-USDT'
});
```

### Account Operations

```typescript
// Получение баланса
const balance = await client.getBalance();

// Получение позиций
const positions = await client.getPositions();

// Получение истории торгов
const trades = await client.getTradeHistory({
  instType: 'SPOT'
});
```

## WebSocket Examples

### Public WebSocket

```typescript
import { WebsocketClient } from 'okx-api';

const wsClient = new WebsocketClient({
  market: 'spot'
});

// Подписка на тикер
wsClient.subscribe([{
  channel: 'tickers',
  instId: 'BTC-USDT'
}]);

// Подписка на книгу ордеров
wsClient.subscribe([{
  channel: 'books',
  instId: 'BTC-USDT'
}]);

// Обработка событий
wsClient.on('update', (data) => {
  console.log('Received update:', data);
});

wsClient.on('error', (error) => {
  console.error('Error:', error);
});

// Отписка
wsClient.unsubscribe([{
  channel: 'tickers',
  instId: 'BTC-USDT'
}]);
```

### Private WebSocket

```typescript
import { WebsocketClient } from 'okx-api';

const wsClient = new WebsocketClient({
  apiKey: 'YOUR-API-KEY',
  apiSecret: 'YOUR-API-SECRET',
  apiPass: 'YOUR-API-PASSPHRASE'
});

// Подписка на обновления аккаунта
wsClient.subscribe([{
  channel: 'account',
  instType: 'SPOT'
}]);

// Подписка на обновления ордеров
wsClient.subscribe([{
  channel: 'orders',
  instType: 'SPOT'
}]);

// Обработка событий
wsClient.on('update', (data) => {
  console.log('Received update:', data);
});

// Закрытие соединения
wsClient.close();
```

## Error Handling

SDK автоматически обрабатывает ответы API:

```typescript
try {
  const result = await client.placeOrder({
    instId: 'BTC-USDT',
    tdMode: 'cash',
    side: 'buy',
    ordType: 'limit',
    px: '20000',
    sz: '0.01'
  });
  console.log('Order placed:', result);
} catch (error) {
  if (error.response) {
    // API вернул ошибку
    console.error('API Error:', error.response.data);
  } else {
    // Сетевая или другая ошибка
    console.error('Error:', error.message);
  }
}
```

## Best Practices

1. **Управление соединением**
   - Используйте WebSocket для real-time данных
   - Обрабатывайте разрывы соединения
   - Следите за heartbeat

2. **Производительность**
   - Переиспользуйте WebSocket соединения
   - Группируйте подписки
   - Кэшируйте часто используемые данные

3. **Безопасность**
   - Храните API ключи в переменных окружения
   - Используйте IP whitelist
   - Регулярно обновляйте ключи

4. **Отладка**
   - Включите логирование для WebSocket
   - Отслеживайте ошибки и latency
   - Используйте TypeScript для лучшей поддержки IDE

## TypeScript Support

SDK написан на TypeScript и предоставляет полную типизацию:

```typescript
import { 
  RestClient, 
  WebsocketClient,
  PlaceOrderRequest,
  OrderResponse,
  WebsocketEvent
} from 'okx-api';

// Все типы доступны для импорта
const order: PlaceOrderRequest = {
  instId: 'BTC-USDT',
  tdMode: 'cash',
  side: 'buy',
  ordType: 'limit',
  px: '20000',
  sz: '0.01'
};
```

## Resources

- [NPM Package](https://www.npmjs.com/package/okx-api)
- [GitHub Repository](https://github.com/tiagosiebler/okx-api)
- [API Documentation](https://www.okx.com/docs-v5/en/)
- [TypeScript Documentation](https://tsdocs.dev/docs/okx-api)
- [Examples Repository](https://github.com/tiagosiebler/awesome-crypto-examples)

## Support

- [GitHub Issues](https://github.com/tiagosiebler/okx-api/issues)
- [Telegram Community](https://t.me/nodetraders)
- [Twitter Updates](https://twitter.com/QuantSDKs)
