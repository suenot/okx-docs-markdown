# Market Data

## Overview

Market data endpoints provide real-time and historical market information:
- Tickers
- Order Book
- Recent Trades
- Candlestick Data
- Index Prices
- Mark Prices
- Open Interest

## Endpoints

### Get Tickers
```http
GET /api/v5/market/tickers
```

Returns the latest price snapshot, best bid/ask price, and trading volume in the last 24 hours.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instType | String | Yes | Instrument type: SPOT, MARGIN, SWAP, FUTURES, OPTION |
| instFamily | String | No | Instrument family |
| uly | String | No | Underlying |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "instType": "SPOT",
        "instId": "BTC-USDT",
        "last": "43000.1",
        "lastSz": "0.1",
        "askPx": "43000.2",
        "askSz": "0.1",
        "bidPx": "43000.0",
        "bidSz": "0.1",
        "open24h": "42000.1",
        "high24h": "43500.1",
        "low24h": "41500.1",
        "volCcy24h": "1000.1",
        "vol24h": "1000.1",
        "ts": "1597026383085",
        "sodUtc0": "40000.1",
        "sodUtc8": "41000.1"
    }]
}
```

### Get Order Book
```http
GET /api/v5/market/books
```

Returns the order book data.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instId | String | Yes | Instrument ID |
| sz | String | No | Order book depth per side. Maximum 400, default 1 |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "asks": [
            ["43000.1", "0.1", "0", "1"],
            ["43000.2", "0.2", "0", "1"]
        ],
        "bids": [
            ["42999.9", "0.1", "0", "1"],
            ["42999.8", "0.2", "0", "1"]
        ],
        "ts": "1597026383085"
    }]
}
```

### Get Candlesticks
```http
GET /api/v5/market/candles
```

Returns candlestick data.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instId | String | Yes | Instrument ID |
| bar | String | No | Bar size: 1m, 3m, 5m, 15m, 30m, 1H, 2H, 4H, 6H, 12H, 1D, 1W, 1M, 3M, 6M, 1Y |
| after | String | No | Pagination of data to return records earlier than the requested ts |
| before | String | No | Pagination of data to return records newer than the requested ts |
| limit | String | No | Number of results per request, maximum 100, default 100 |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [
        ["1597026383085", "8533.02", "8553.74", "8527.17", "8548.26", "45.59", "389354.2"],
        ["1597026383085", "8533.02", "8553.74", "8527.17", "8548.26", "45.59", "389354.2"]
    ]
}
```

### Get Trades
```http
GET /api/v5/market/trades
```

Returns the recent trades data.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instId | String | Yes | Instrument ID |
| limit | String | No | Number of results per request, maximum 100, default 100 |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "instId": "BTC-USDT",
        "tradeId": "123",
        "px": "43000.1",
        "sz": "0.1",
        "side": "buy",
        "ts": "1597026383085"
    }]
}
```

### Get Index Tickers
```http
GET /api/v5/market/index-tickers
```

Returns the index tickers.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| quoteCcy | String | No | Quote currency |
| instId | String | No | Index instrument ID |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "instId": "BTC-USDT",
        "idxPx": "43000.1",
        "high24h": "43500.1",
        "low24h": "41500.1",
        "open24h": "42000.1",
        "sodUtc0": "40000.1",
        "sodUtc8": "41000.1",
        "ts": "1597026383085"
    }]
}
```

## Best Practices

1. Data Handling
   - Cache frequently accessed data
   - Implement websocket for real-time updates
   - Handle rate limits appropriately

2. Order Book Management
   - Maintain local order book copy
   - Use snapshots and deltas
   - Implement order book checksum verification

3. Performance Optimization
   - Use appropriate data structures
   - Implement efficient updates
   - Handle large datasets

## Example Implementation

```python
def get_order_book(api_key, inst_id, depth=20):
    """
    Get order book data
    
    Args:
        inst_id (str): Instrument ID
        depth (int): Order book depth (default: 20)
    
    Returns:
        dict: Order book data
    """
    
    url = f"https://www.okx.com/api/v5/market/books?instId={inst_id}&sz={depth}"
    
    headers = {
        "OK-ACCESS-KEY": api_key,
        "Content-Type": "application/json"
    }
    
    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            data = response.json()
            if data["code"] == "0":
                return {
                    "asks": [[float(price), float(size)] for price, size, _, _ in data["data"][0]["asks"]],
                    "bids": [[float(price), float(size)] for price, size, _, _ in data["data"][0]["bids"]],
                    "timestamp": int(data["data"][0]["ts"])
                }
        return None
    except Exception as e:
        print(f"Error fetching order book: {str(e)}")
        return None

def get_candlesticks(api_key, inst_id, bar="1m", limit=100):
    """
    Get candlestick data
    
    Args:
        inst_id (str): Instrument ID
        bar (str): Candlestick interval
        limit (int): Number of candlesticks
    
    Returns:
        list: Candlestick data
    """
    
    url = f"https://www.okx.com/api/v5/market/candles?instId={inst_id}&bar={bar}&limit={limit}"
    
    headers = {
        "OK-ACCESS-KEY": api_key,
        "Content-Type": "application/json"
    }
    
    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            data = response.json()
            if data["code"] == "0":
                return [{
                    "timestamp": int(candle[0]),
                    "open": float(candle[1]),
                    "high": float(candle[2]),
                    "low": float(candle[3]),
                    "close": float(candle[4]),
                    "volume": float(candle[5]),
                    "volume_ccy": float(candle[6])
                } for candle in data["data"]]
        return None
    except Exception as e:
        print(f"Error fetching candlesticks: {str(e)}")
        return None
```
