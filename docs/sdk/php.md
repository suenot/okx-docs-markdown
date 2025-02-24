# PHP SDK

## Overview

OKX предоставляет PHP SDK для работы с V5 API. SDK поддерживает все REST API эндпоинты и WebSocket соединения, обеспечивая удобный способ взаимодействия с биржей на языке PHP.

## Requirements

- PHP >= 7.2
- Composer
- OpenSSL Extension
- cURL Extension

## Installation

```bash
composer require lin/okex
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

```php
<?php
use Lin\Okex\OkexV5;

// Создание клиента для live trading
$okex = new OkexV5($apiKey, $secretKey, $passphrase);

// Создание клиента для testnet
$okex = new OkexV5($apiKey, $secretKey, $passphrase, true);
```

### Account Operations

```php
<?php
use Lin\Okex\OkexV5;

$okex = new OkexV5($apiKey, $secretKey, $passphrase);

try {
    // Получение баланса
    $balance = $okex->account()->getBalance();
    print_r($balance);
    
    // Получение позиций
    $positions = $okex->account()->getPositions();
    print_r($positions);
    
    // Получение истории транзакций
    $bills = $okex->account()->getBills();
    print_r($bills);
    
    // Настройка кредитного плеча
    $leverage = $okex->account()->postSetLeverage([
        'instId' => 'BTC-USDT',
        'lever' => '5',
        'mgnMode' => 'cross',
    ]);
    print_r($leverage);
    
} catch (Exception $e) {
    print_r(json_decode($e->getMessage(), true));
}
```

### Trading Operations

```php
<?php
use Lin\Okex\OkexV5;

$okex = new OkexV5($apiKey, $secretKey, $passphrase);

try {
    // Размещение ордера
    $order = $okex->trade()->postOrder([
        'instId' => 'BTC-USDT',
        'tdMode' => 'cash',
        'clOrdId' => 'custom_order_id',
        'side' => 'buy',
        'ordType' => 'limit',
        'sz' => '0.01',
        'px' => '20000',
    ]);
    print_r($order);
    
    // Отмена ордера
    $cancel = $okex->trade()->postCancelOrder([
        'instId' => 'BTC-USDT',
        'ordId' => $order['ordId'],
    ]);
    print_r($cancel);
    
    // Изменение ордера
    $amend = $okex->trade()->postAmendOrder([
        'instId' => 'BTC-USDT',
        'ordId' => $order['ordId'],
        'newSz' => '0.02',
        'newPx' => '21000',
    ]);
    print_r($amend);
    
    // Получение информации об ордере
    $orderInfo = $okex->trade()->getOrder([
        'instId' => 'BTC-USDT',
        'ordId' => $order['ordId'],
    ]);
    print_r($orderInfo);
    
} catch (Exception $e) {
    print_r(json_decode($e->getMessage(), true));
}
```

### Market Data

```php
<?php
use Lin\Okex\OkexV5;

$okex = new OkexV5(); // Для публичных эндпоинтов аутентификация не нужна

try {
    // Получение тикера
    $ticker = $okex->market()->getTicker([
        'instId' => 'BTC-USDT',
    ]);
    print_r($ticker);
    
    // Получение тикеров для всех спот пар
    $tickers = $okex->market()->getTickers([
        'instType' => 'SPOT',
    ]);
    print_r($tickers);
    
    // Получение книги ордеров
    $orderbook = $okex->market()->getBooks([
        'instId' => 'BTC-USDT',
        'sz' => '20',
    ]);
    print_r($orderbook);
    
    // Получение исторических свечей
    $candles = $okex->market()->getCandles([
        'instId' => 'BTC-USDT',
        'bar' => '1D',
    ]);
    print_r($candles);
    
} catch (Exception $e) {
    print_r(json_decode($e->getMessage(), true));
}
```

## WebSocket Examples

### Server Setup

```php
<?php
use Lin\Okex\OkexWebSocketV5;

$okex = new OkexWebSocketV5();

// Конфигурация WebSocket сервера
$okex->config([
    // Включить логирование
    'log' => true,
    
    // Адрес и порт демона (по умолчанию 0.0.0.0:2207)
    'global' => '127.0.0.1:2207',
    
    // Время сердцебиения (по умолчанию 20 секунд)
    'ping_time' => 20,
    
    // Время мониторинга подписок (по умолчанию 2 секунды)
    'listen_time' => 2,
    
    // Время обновления данных (по умолчанию 0.1 секунды)
    'data_time' => 0.1,
    
    // Размер очереди сообщений (по умолчанию 100)
    'queue_count' => 100,
]);

// Запуск сервера
$okex->start();
```

### Client Setup and Subscriptions

```php
<?php
use Lin\Okex\OkexWebSocketV5;

$okex = new OkexWebSocketV5();

// Конфигурация клиента
$okex->config([
    'log' => true,
]);

// Подписка на публичные каналы
$okex->subscribe([
    ["channel" => "tickers", "instId" => "BTC-USDT"],
    ["channel" => "books", "instId" => "BTC-USDT"],
    ["channel" => "candle5m", "instId" => "BTC-USDT"],
]);

// Подписка на приватные каналы
$okex->keysecret([
    'key' => 'your-api-key',
    'secret' => 'your-secret-key',
    'passphrase' => 'your-passphrase',
]);

$okex->subscribe([
    // Публичные каналы
    ["channel" => "tickers", "instId" => "BTC-USDT"],
    ["channel" => "books", "instId" => "BTC-USDT"],
    
    // Приватные каналы
    ["channel" => "account", "ccy" => "BTC"],
    ["channel" => "positions", "instType" => "FUTURES"],
    ["channel" => "orders", "instType" => "SPOT"],
]);

// Получение данных
while(true) {
    $data = $okex->getSubscribe();
    print_r($data);
    sleep(1);
}
```

## Error Handling

```php
<?php
use Lin\Okex\OkexV5;

$okex = new OkexV5($apiKey, $secretKey, $passphrase);

try {
    $result = $okex->trade()->postOrder([
        'instId' => 'BTC-USDT',
        'tdMode' => 'cash',
        'side' => 'buy',
        'ordType' => 'limit',
        'sz' => '0.01',
        'px' => '20000',
    ]);
    print_r($result);
} catch (Exception $e) {
    $error = json_decode($e->getMessage(), true);
    
    // Обработка API ошибок
    if (isset($error['code'])) {
        echo "API Error Code: " . $error['code'] . "\n";
        echo "API Error Message: " . $error['msg'] . "\n";
    } else {
        // Обработка других ошибок
        echo "Error: " . $e->getMessage() . "\n";
    }
}
```

## Best Practices

1. **Управление соединениями**
   - Переиспользуйте REST клиенты
   - Правильно закрывайте WebSocket соединения
   - Обрабатывайте разрывы соединения

2. **Обработка ошибок**
   - Всегда используйте try-catch блоки
   - Реализуйте retry механизм
   - Логируйте все ошибки

3. **Производительность**
   - Используйте асинхронные операции где возможно
   - Кэшируйте часто используемые данные
   - Следите за лимитами запросов

4. **Безопасность**
   - Храните API ключи в .env файле
   - Используйте IP whitelist
   - Регулярно обновляйте SDK

## Resources

- [GitHub Repository](https://github.com/zhouaini528/okex-php)
- [Composer Package](https://packagist.org/packages/lin/okex)
- [API Documentation](https://www.okx.com/docs-v5/en/)
- [Examples](https://github.com/zhouaini528/okex-php/tree/master/tests/okex_v5)
