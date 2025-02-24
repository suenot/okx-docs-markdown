# Trading

## Overview

The trading endpoints allow you to:
- Place new orders
- Cancel existing orders
- Amend orders
- Query order status
- Get order history
- Place and manage batch orders

## Order Types

- Market Order
- Limit Order
- Post-only
- Fill-or-kill
- Immediate-or-cancel

## Endpoints

### Place Order
```http
POST /api/v5/trade/order
```

Places a new order.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instId | String | Yes | Instrument ID |
| tdMode | String | Yes | Trade mode: cash, cross, isolated |
| side | String | Yes | buy, sell |
| ordType | String | Yes | Order type: market, limit, post_only, fok, ioc |
| sz | String | Yes | Size of the order |
| px | String | No | Price of the order, required for non-market orders |
| posSide | String | No | Position side: long, short, net |
| reduceOnly | Boolean | No | Whether to close position only |
| tgtCcy | String | No | Unit of the size: base_ccy, quote_ccy |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "clOrdId": "b15",
        "ordId": "312269865356374016",
        "tag": "",
        "sCode": "0",
        "sMsg": ""
    }]
}
```

### Place Multiple Orders
```http
POST /api/v5/trade/batch-orders
```

Places multiple orders in a single request (up to 20 orders).

#### Request Parameters
Array of order objects, each containing the same parameters as single order placement.

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "clOrdId": "b15",
        "ordId": "312269865356374016",
        "tag": "",
        "sCode": "0",
        "sMsg": ""
    }]
}
```

### Cancel Order
```http
POST /api/v5/trade/cancel-order
```

Cancels an existing order.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instId | String | Yes | Instrument ID |
| ordId | String | No | Order ID |
| clOrdId | String | No | Client order ID |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "clOrdId": "b15",
        "ordId": "312269865356374016",
        "sCode": "0",
        "sMsg": ""
    }]
}
```

### Amend Order
```http
POST /api/v5/trade/amend-order
```

Modifies an existing order.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instId | String | Yes | Instrument ID |
| ordId | String | No | Order ID |
| clOrdId | String | No | Client order ID |
| reqId | String | No | Request ID |
| newSz | String | No | New size |
| newPx | String | No | New price |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "clOrdId": "b15",
        "ordId": "312269865356374016",
        "reqId": "",
        "sCode": "0",
        "sMsg": ""
    }]
}
```

### Get Order Details
```http
GET /api/v5/trade/order
```

Gets details of a specific order.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instId | String | Yes | Instrument ID |
| ordId | String | No | Order ID |
| clOrdId | String | No | Client order ID |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "instType": "SPOT",
        "instId": "BTC-USDT",
        "ccy": "",
        "ordId": "312269865356374016",
        "clOrdId": "b15",
        "tag": "",
        "px": "8200",
        "sz": "0.001",
        "pnl": "0",
        "ordType": "limit",
        "side": "buy",
        "posSide": "long",
        "tdMode": "cash",
        "accFillSz": "0",
        "fillPx": "0",
        "tradeId": "0",
        "fillSz": "0",
        "fillTime": "0",
        "state": "live",
        "avgPx": "0",
        "lever": "0",
        "tpTriggerPx": "",
        "tpOrdPx": "",
        "slTriggerPx": "",
        "slOrdPx": "",
        "feeCcy": "",
        "fee": "",
        "rebateCcy": "",
        "rebate": "",
        "tgtCcy": "",
        "category": "",
        "uTime": "1597026383085",
        "cTime": "1597026383085"
    }]
}
```

## Best Practices

1. Order Management
   - Use client order IDs for tracking
   - Implement order state monitoring
   - Handle partial fills appropriately

2. Risk Management
   - Set appropriate order sizes
   - Use stop loss orders
   - Monitor account balance

3. Performance
   - Batch orders when possible
   - Handle rate limits
   - Use WebSocket for order updates

## Error Handling

Common errors to handle:
- Insufficient funds
- Invalid order parameters
- Rate limit exceeded
- Market closed
- Invalid instrument ID

## Example Implementation

```python
def place_order(api_key, secret_key, passphrase, inst_id, td_mode, side, ord_type, size, price=None):
    """
    Place a new order
    
    Args:
        inst_id (str): Instrument ID
        td_mode (str): Trade mode (cash, cross, isolated)
        side (str): Order side (buy, sell)
        ord_type (str): Order type (market, limit, post_only, fok, ioc)
        size (str): Order size
        price (str, optional): Order price, required for limit orders
    
    Returns:
        dict: Order response
    """
    
    # API endpoint
    url = "https://www.okx.com/api/v5/trade/order"
    
    # Prepare request body
    body = {
        "instId": inst_id,
        "tdMode": td_mode,
        "side": side,
        "ordType": ord_type,
        "sz": size
    }
    
    if price and ord_type != "market":
        body["px"] = price
    
    # Generate timestamp
    timestamp = str(int(time.time()))
    
    # Create signature
    signature = create_signature(timestamp, "POST", "/api/v5/trade/order", body)
    
    # Headers
    headers = {
        "OK-ACCESS-KEY": api_key,
        "OK-ACCESS-SIGN": signature,
        "OK-ACCESS-TIMESTAMP": timestamp,
        "OK-ACCESS-PASSPHRASE": passphrase,
        "Content-Type": "application/json"
    }
    
    try:
        response = requests.post(url, headers=headers, json=body)
        if response.status_code == 200:
            return response.json()
        else:
            print(f"Error: {response.status_code}")
            return None
    except Exception as e:
        print(f"Exception: {str(e)}")
        return None
```
