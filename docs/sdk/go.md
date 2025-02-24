# Go SDK

## Overview

OKX предоставляет Go SDK для работы с V5 API. SDK поддерживает все REST API эндпоинты и WebSocket соединения, обеспечивая удобный способ взаимодействия с биржей на языке Go.

## Installation

```bash
go get github.com/iaping/go-okx
```

## Features

- Полная поддержка REST API V5
- WebSocket API для real-time данных
- Поддержка публичных и приватных каналов
- Автоматическая обработка аутентификации
- Поддержка тестовой сети
- Типизированные структуры для всех запросов и ответов

## REST API Examples

### Basic Setup

```go
package main

import (
    "log"
    
    "github.com/iaping/go-okx/common"
    "github.com/iaping/go-okx/rest"
)

func main() {
    // Создание клиента для live trading
    auth := common.NewAuth(
        "YOUR-API-KEY",
        "YOUR-SECRET-KEY",
        "YOUR-PASSPHRASE",
        false, // false для live trading, true для testnet
    )
    
    client := rest.New("", auth, nil)
}
```

### Account Operations

```go
package main

import (
    "log"
    
    "github.com/iaping/go-okx/rest"
    "github.com/iaping/go-okx/rest/api/account"
)

func main() {
    auth := common.NewAuth("YOUR-API-KEY", "YOUR-SECRET-KEY", "YOUR-PASSPHRASE", false)
    client := rest.New("", auth, nil)
    
    // Получение баланса
    balanceParam := &account.GetBalanceParam{}
    balanceReq, balanceResp := account.NewGetBalance(balanceParam)
    if err := client.Do(balanceReq, balanceResp); err != nil {
        log.Fatal(err)
    }
    log.Printf("Balance: %+v\n", balanceResp)
    
    // Получение позиций
    positionsParam := &account.GetPositionsParam{
        InstType: "SPOT",
    }
    positionsReq, positionsResp := account.NewGetPositions(positionsParam)
    if err := client.Do(positionsReq, positionsResp); err != nil {
        log.Fatal(err)
    }
    log.Printf("Positions: %+v\n", positionsResp)
}
```

### Trading Operations

```go
package main

import (
    "log"
    
    "github.com/iaping/go-okx/rest"
    "github.com/iaping/go-okx/rest/api/trade"
)

func main() {
    auth := common.NewAuth("YOUR-API-KEY", "YOUR-SECRET-KEY", "YOUR-PASSPHRASE", false)
    client := rest.New("", auth, nil)
    
    // Размещение ордера
    orderParam := &trade.PlaceOrderParam{
        InstId:  "BTC-USDT",
        TdMode:  "cash",
        Side:    "buy",
        OrdType: "limit",
        Px:      "20000",
        Sz:      "0.01",
    }
    orderReq, orderResp := trade.NewPlaceOrder(orderParam)
    if err := client.Do(orderReq, orderResp); err != nil {
        log.Fatal(err)
    }
    log.Printf("Order placed: %+v\n", orderResp)
    
    // Отмена ордера
    cancelParam := &trade.CancelOrderParam{
        InstId: "BTC-USDT",
        OrdId:  orderResp.OrdId,
    }
    cancelReq, cancelResp := trade.NewCancelOrder(cancelParam)
    if err := client.Do(cancelReq, cancelResp); err != nil {
        log.Fatal(err)
    }
    log.Printf("Order cancelled: %+v\n", cancelResp)
}
```

### Market Data

```go
package main

import (
    "log"
    
    "github.com/iaping/go-okx/rest"
    "github.com/iaping/go-okx/rest/api/market"
)

func main() {
    client := rest.New("", nil, nil) // Для публичных эндпоинтов аутентификация не нужна
    
    // Получение тикера
    tickerParam := &market.GetTickerParam{
        InstId: "BTC-USDT",
    }
    tickerReq, tickerResp := market.NewGetTicker(tickerParam)
    if err := client.Do(tickerReq, tickerResp); err != nil {
        log.Fatal(err)
    }
    log.Printf("Ticker: %+v\n", tickerResp)
    
    // Получение книги ордеров
    bookParam := &market.GetOrderBookParam{
        InstId: "BTC-USDT",
        Sz:     "20",
    }
    bookReq, bookResp := market.NewGetOrderBook(bookParam)
    if err := client.Do(bookReq, bookResp); err != nil {
        log.Fatal(err)
    }
    log.Printf("OrderBook: %+v\n", bookResp)
}
```

## WebSocket Examples

### Public WebSocket

```go
package main

import (
    "log"
    
    "github.com/iaping/go-okx/ws/public"
)

func main() {
    // Обработчик для тикеров
    handler := func(c public.EventTickers) {
        log.Printf("Received ticker: %+v\n", c)
    }
    
    // Обработчик ошибок
    errorHandler := func(err error) {
        log.Printf("WebSocket error: %v\n", err)
    }
    
    // Подписка на тикеры
    if err := public.SubscribeTickers(
        "BTC-USDT",
        handler,
        errorHandler,
        false, // false для live trading, true для testnet
    ); err != nil {
        log.Fatal(err)
    }
    
    select {} // Держим соединение открытым
}
```

### Private WebSocket

```go
package main

import (
    "log"
    
    "github.com/iaping/go-okx/common"
    "github.com/iaping/go-okx/ws"
    "github.com/iaping/go-okx/ws/private"
)

func main() {
    auth := common.NewAuth("YOUR-API-KEY", "YOUR-SECRET-KEY", "YOUR-PASSPHRASE", false)
    
    // Параметры подписки
    args := &ws.Args{
        InstType: "SPOT",
    }
    
    // Обработчик для ордеров
    handler := func(c private.EventOrders) {
        log.Printf("Received order update: %+v\n", c)
    }
    
    // Обработчик ошибок
    errorHandler := func(err error) {
        log.Printf("WebSocket error: %v\n", err)
    }
    
    // Подписка на обновления ордеров
    if err := private.SubscribeOrders(args, auth, handler, errorHandler); err != nil {
        log.Fatal(err)
    }
    
    select {} // Держим соединение открытым
}
```

## Error Handling

```go
package main

import (
    "log"
    
    "github.com/iaping/go-okx/rest"
    "github.com/iaping/go-okx/rest/api/trade"
)

func main() {
    client := rest.New("", auth, nil)
    
    orderParam := &trade.PlaceOrderParam{
        InstId:  "BTC-USDT",
        TdMode:  "cash",
        Side:    "buy",
        OrdType: "limit",
        Px:      "20000",
        Sz:      "0.01",
    }
    
    orderReq, orderResp := trade.NewPlaceOrder(orderParam)
    if err := client.Do(orderReq, orderResp); err != nil {
        switch e := err.(type) {
        case *rest.Error:
            // Обработка API ошибок
            log.Printf("API Error: code=%s, message=%s\n", e.Code, e.Message)
        default:
            // Обработка других ошибок
            log.Printf("Error: %v\n", err)
        }
        return
    }
    
    log.Printf("Order placed successfully: %+v\n", orderResp)
}
```

## Best Practices

1. **Управление соединениями**
   - Переиспользуйте REST клиенты
   - Правильно закрывайте WebSocket соединения
   - Обрабатывайте разрывы соединения

2. **Обработка ошибок**
   - Всегда проверяйте ошибки
   - Реализуйте retry механизм
   - Логируйте все ошибки

3. **Производительность**
   - Используйте goroutines для параллельных запросов
   - Кэшируйте часто используемые данные
   - Следите за лимитами запросов

4. **Безопасность**
   - Храните API ключи в переменных окружения
   - Используйте IP whitelist
   - Регулярно обновляйте SDK

## Resources

- [GitHub Repository](https://github.com/iaping/go-okx)
- [API Documentation](https://www.okx.com/docs-v5/en/)
- [Go Packages](https://pkg.go.dev/github.com/iaping/go-okx)
- [Examples](https://github.com/iaping/go-okx/tree/master/examples)
