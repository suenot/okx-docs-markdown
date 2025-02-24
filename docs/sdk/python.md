# Python SDK

## Overview

OKX предоставляет официальный Python SDK для работы с V5 API. SDK поддерживает все REST API эндпоинты, WebSocket соединения и тестовую сеть.

## Installation

Требования:
- Python >= 3.9
- websockets >= 6.0 (для WebSocket API)

Установка через pip:

```bash
pip install python-okx
```

## Features

- Полная поддержка всех REST API эндпоинтов
- Реализация приватных и публичных WebSocket соединений
- Поддержка тестовой сети (Testnet)
- Автоматическая обработка WebSocket соединений с реконнектом
- Мультиплексированные WebSocket соединения

## Authentication

```python
import okx.Account as Account
import okx.MarketData as MarketData
import okx.PublicData as PublicData
import okx.Trade as Trade
import okx.SubAccount as SubAccount
import okx.Funding as Funding
import okx.SpreadTrading as SpreadTrading
import okx.BlockTrading as BlockTrading

# Конфигурация для live trading
api_key = "YOUR-API-KEY"
secret_key = "YOUR-SECRET-KEY"
passphrase = "YOUR-PASSPHRASE"
flag = "0"  # 0: live trading, 1: demo trading

# Инициализация клиентов
accountAPI = Account.AccountAPI(api_key, secret_key, passphrase, False, flag)
marketDataAPI = MarketData.MarketDataAPI(api_key, secret_key, passphrase, False, flag)
publicDataAPI = PublicData.PublicDataAPI(api_key, secret_key, passphrase, False, flag)
tradeAPI = Trade.TradeAPI(api_key, secret_key, passphrase, False, flag)
```

## REST API Examples

### Account Operations

```python
# Получение баланса аккаунта
result = accountAPI.get_account_balance()
print(result)

# Получение позиций
result = accountAPI.get_positions()
print(result)

# Получение истории транзакций
result = accountAPI.get_account_bills()
print(result)
```

### Trading Operations

```python
# Размещение ордера
result = tradeAPI.place_order(
    instId="BTC-USDT",
    tdMode="cash",
    side="buy",
    ordType="limit",
    px="20000",
    sz="0.01"
)
print(result)

# Отмена ордера
result = tradeAPI.cancel_order(
    instId="BTC-USDT",
    ordId="12345"
)
print(result)

# Получение открытых ордеров
result = tradeAPI.get_order_list(
    instType="SPOT",
    instId="BTC-USDT"
)
print(result)
```

### Market Data

```python
# Получение тикера
result = marketDataAPI.get_ticker(
    instId="BTC-USDT"
)
print(result)

# Получение книги ордеров
result = marketDataAPI.get_orderbook(
    instId="BTC-USDT",
    sz="20"
)
print(result)

# Получение исторических свечей
result = marketDataAPI.get_candlesticks(
    instId="BTC-USDT",
    bar="1D"
)
print(result)
```

## WebSocket Examples

### Public WebSocket

```python
import asyncio
from okx.websocket import PublicWebsocket

async def public_ws_example():
    # Инициализация WebSocket клиента
    ws = PublicWebsocket()
    
    # Определение callback функции
    async def on_message(message):
        print(f"Received message: {message}")
    
    # Установка callback
    ws.on_message = on_message
    
    # Подключение к WebSocket
    await ws.connect()
    
    # Подписка на каналы
    await ws.subscribe([
        {
            "channel": "tickers",
            "instId": "BTC-USDT"
        },
        {
            "channel": "candle1m",
            "instId": "BTC-USDT"
        }
    ])
    
    # Держим соединение активным
    while True:
        await asyncio.sleep(1)

# Запуск WebSocket клиента
asyncio.run(public_ws_example())
```

### Private WebSocket

```python
import asyncio
from okx.websocket import PrivateWebsocket

async def private_ws_example():
    # Инициализация WebSocket клиента
    ws = PrivateWebsocket(
        api_key="YOUR-API-KEY",
        api_secret_key="YOUR-SECRET-KEY",
        passphrase="YOUR-PASSPHRASE"
    )
    
    # Определение callback функции
    async def on_message(message):
        print(f"Received message: {message}")
    
    # Установка callback
    ws.on_message = on_message
    
    # Подключение к WebSocket
    await ws.connect()
    
    # Подписка на каналы
    await ws.subscribe([
        {
            "channel": "account",
            "instType": "SPOT"
        },
        {
            "channel": "orders",
            "instType": "SPOT"
        }
    ])
    
    # Держим соединение активным
    while True:
        await asyncio.sleep(1)

# Запуск WebSocket клиента
asyncio.run(private_ws_example())
```

## Error Handling

SDK возвращает ответы в следующем формате:

```python
{
    'code': '0',          # Код ответа. '0' означает успех
    'msg': '',           # Сообщение об ошибке
    'data': [...]        # Данные ответа
}
```

Пример обработки ошибок:

```python
def place_order_safely(trade_api, **params):
    try:
        result = trade_api.place_order(**params)
        if result['code'] == '0':
            print("Order placed successfully")
            return result['data']
        else:
            print(f"Error placing order: {result['msg']}")
            return None
    except Exception as e:
        print(f"Exception occurred: {str(e)}")
        return None
```

## Best Practices

1. **Управление соединением**
   - Всегда закрывайте WebSocket соединения когда они больше не нужны
   - Используйте механизм переподключения для WebSocket
   - Обрабатывайте таймауты и ошибки сети

2. **Безопасность**
   - Храните API ключи в безопасном месте
   - Используйте переменные окружения для API ключей
   - Регулярно обновляйте API ключи

3. **Производительность**
   - Используйте WebSocket для получения real-time данных
   - Кэшируйте часто используемые данные
   - Соблюдайте ограничения по частоте запросов

4. **Отладка**
   - Включите подробное логирование для отладки
   - Сохраняйте важные события и ошибки
   - Мониторьте производительность системы

## Resources

- [Официальная документация API V5](https://www.okx.com/docs-v5/en/)
- [GitHub репозиторий SDK](https://github.com/okxapi/python-okx)
- [PyPI пакет](https://pypi.org/project/python-okx/)
- [Telegram канал OKX API](https://t.me/OKXAPI)

## Tutorials

- [Спот трейдинг с Jupyter Notebook](https://www.okx.com/help/how-can-i-do-spot-trading-with-the-jupyter-notebook)
- [Деривативы с Jupyter Notebook](https://www.okx.com/help/how-can-i-do-derivatives-trading-with-the-jupyter-notebook)

## Support

Если у вас возникли проблемы с WebSocket API, обратитесь к следующим ресурсам:

- [Документация asyncio](https://docs.python.org/3/library/asyncio-dev.html)
- [Документация websockets](https://websockets.readthedocs.io/en/stable/intro.html)
- [GitHub websockets](https://github.com/aaugustin/websockets)
