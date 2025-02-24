# Java SDK

## Overview

OKX предоставляет официальный Java SDK для работы с V5 API. SDK поддерживает все REST API эндпоинты и WebSocket соединения, а также включает поддержку различных систем сборки (Maven, Gradle, sbt).

## Installation

### Maven

```xml
<dependency>
    <groupId>xyz.felh</groupId>
    <artifactId>okx-v5-java</artifactId>
    <version>0.5.2024072701</version>
</dependency>
```

### Gradle

```groovy
implementation group: 'xyz.felh', name: 'okx-v5-java', version: '0.5.2024072701'
```

### sbt

```scala
libraryDependencies += "xyz.felh" % "okx-v5-java" % "0.5.2024072701"
```

## Features

- Полная поддержка REST API V5
- WebSocket API для real-time данных
- Поддержка всех типов торговли (спот, фьючерсы, свопы)
- Поддержка алгоритмической торговли
- Асинхронные операции
- Автоматическая обработка ошибок
- Поддержка тестовой сети

## REST API Examples

### Basic Setup

```java
import xyz.felh.okx.client.*;
import xyz.felh.okx.model.*;

// Создание клиента для live trading
OkxV5RestClient client = new OkxV5RestClient.Builder()
    .apiKey("YOUR-API-KEY")
    .secretKey("YOUR-SECRET-KEY")
    .passphrase("YOUR-PASSPHRASE")
    .build();

// Создание клиента для demo trading
OkxV5RestClient testnetClient = new OkxV5RestClient.Builder()
    .apiKey("YOUR-API-KEY")
    .secretKey("YOUR-SECRET-KEY")
    .passphrase("YOUR-PASSPHRASE")
    .sandbox(true)
    .build();
```

### Account Operations

```java
// Получение баланса
AccountBalance balance = client.getAccountApi().getBalance();

// Получение позиций
List<Position> positions = client.getAccountApi().getPositions();

// Получение истории транзакций
List<Bill> bills = client.getAccountApi().getBills(BillsRequest.builder()
    .instType("SPOT")
    .build());

// Настройка кредитного плеча
SetLeverageResult leverage = client.getAccountApi().setLeverage(SetLeverageRequest.builder()
    .instId("BTC-USDT")
    .lever("10")
    .mgnMode("isolated")
    .build());
```

### Trading Operations

```java
// Размещение ордера
PlaceOrderResult order = client.getTradeApi().placeOrder(PlaceOrderRequest.builder()
    .instId("BTC-USDT")
    .tdMode("cash")
    .side("buy")
    .ordType("limit")
    .px("20000")
    .sz("0.01")
    .build());

// Отмена ордера
CancelOrderResult cancelled = client.getTradeApi().cancelOrder(CancelOrderRequest.builder()
    .instId("BTC-USDT")
    .ordId(order.getOrdId())
    .build());

// Получение открытых ордеров
List<Order> orders = client.getTradeApi().getOrderList(OrderListRequest.builder()
    .instType("SPOT")
    .instId("BTC-USDT")
    .build());

// Размещение нескольких ордеров
List<PlaceOrderRequest> requests = Arrays.asList(
    PlaceOrderRequest.builder()
        .instId("BTC-USDT")
        .tdMode("cash")
        .side("buy")
        .ordType("limit")
        .px("19000")
        .sz("0.01")
        .build(),
    PlaceOrderRequest.builder()
        .instId("BTC-USDT")
        .tdMode("cash")
        .side("buy")
        .ordType("limit")
        .px("18000")
        .sz("0.01")
        .build()
);
List<PlaceOrderResult> batchOrders = client.getTradeApi().placeMultipleOrders(requests);
```

### Market Data

```java
// Получение тикера
Ticker ticker = client.getMarketApi().getTicker(TickerRequest.builder()
    .instId("BTC-USDT")
    .build());

// Получение книги ордеров
OrderBook orderbook = client.getMarketApi().getOrderBook(OrderBookRequest.builder()
    .instId("BTC-USDT")
    .sz("20")
    .build());

// Получение исторических свечей
List<Candlestick> candles = client.getMarketApi().getCandlesticks(CandlesticksRequest.builder()
    .instId("BTC-USDT")
    .bar("1D")
    .build());
```

### Grid Trading

```java
// Размещение grid trading ордера
PlaceGridAlgoOrderResult gridOrder = client.getGridApi().placeOrder(PlaceGridAlgoOrderRequest.builder()
    .instId("BTC-USDT")
    .algoOrdType("grid")
    .maxPx("25000")
    .minPx("15000")
    .gridNum("10")
    .quoteSz("1000")
    .build());

// Получение списка grid ордеров
List<GridAlgoOrder> gridOrders = client.getGridApi().getOrders(GridAlgoOrdersRequest.builder()
    .algoOrdType("grid")
    .build());

// Остановка grid trading
StopGridAlgoOrderResult stopped = client.getGridApi().stopOrder(StopGridAlgoOrderRequest.builder()
    .algoId(gridOrder.getAlgoId())
    .instId("BTC-USDT")
    .build());
```

## WebSocket Examples

### Public WebSocket

```java
import xyz.felh.okx.websocket.*;

// Создание WebSocket клиента
OkxV5WebSocketClient wsClient = new OkxV5WebSocketClient();

// Подписка на тикер
wsClient.subscribeTicker("BTC-USDT", new WebSocketCallback() {
    @Override
    public void onMessage(String message) {
        System.out.println("Received ticker update: " + message);
    }

    @Override
    public void onFailure(Throwable t) {
        System.err.println("WebSocket error: " + t.getMessage());
    }
});

// Подписка на книгу ордеров
wsClient.subscribeOrderBook("BTC-USDT", "20", new WebSocketCallback() {
    @Override
    public void onMessage(String message) {
        System.out.println("Received orderbook update: " + message);
    }
});

// Закрытие соединения
wsClient.close();
```

### Private WebSocket

```java
// Создание WebSocket клиента с аутентификацией
OkxV5WebSocketClient wsClient = new OkxV5WebSocketClient.Builder()
    .apiKey("YOUR-API-KEY")
    .secretKey("YOUR-SECRET-KEY")
    .passphrase("YOUR-PASSPHRASE")
    .build();

// Подписка на обновления аккаунта
wsClient.subscribeAccount(new WebSocketCallback() {
    @Override
    public void onMessage(String message) {
        System.out.println("Received account update: " + message);
    }
});

// Подписка на обновления ордеров
wsClient.subscribeOrders("SPOT", new WebSocketCallback() {
    @Override
    public void onMessage(String message) {
        System.out.println("Received order update: " + message);
    }
});
```

## Error Handling

```java
try {
    PlaceOrderResult order = client.getTradeApi().placeOrder(PlaceOrderRequest.builder()
        .instId("BTC-USDT")
        .tdMode("cash")
        .side("buy")
        .ordType("limit")
        .px("20000")
        .sz("0.01")
        .build());
} catch (OkxApiException e) {
    // Обработка API ошибок
    System.err.println("Error code: " + e.getErrorCode());
    System.err.println("Error message: " + e.getMessage());
} catch (Exception e) {
    // Обработка других ошибок
    System.err.println("Unexpected error: " + e.getMessage());
}
```

## Best Practices

1. **Управление ресурсами**
   - Используйте try-with-resources для WebSocket клиентов
   - Закрывайте неиспользуемые соединения
   - Переиспользуйте REST клиенты

2. **Обработка ошибок**
   - Всегда обрабатывайте OkxApiException
   - Реализуйте retry механизм для временных ошибок
   - Логируйте все ошибки

3. **Производительность**
   - Используйте batch операции где возможно
   - Кэшируйте часто используемые данные
   - Следите за лимитами запросов

4. **Безопасность**
   - Храните API ключи в защищенном месте
   - Используйте переменные окружения
   - Регулярно обновляйте SDK

## Resources

- [Maven Repository](https://mvnrepository.com/artifact/xyz.felh/okx-v5-java)
- [GitHub Repository](https://github.com/forestwanglin/okx-v5-java)
- [API Documentation](https://www.okx.com/docs-v5/en/)
- [JavaDoc](https://javadoc.io/doc/xyz.felh/okx-v5-java)
