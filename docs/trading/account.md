# Account Management

## Overview

Account management endpoints allow you to:
- View account configurations
- Check balances
- Monitor positions
- Review trading history
- Manage account settings

## Endpoints

### Get Account Configuration
```http
GET /api/v5/account/config
```

Retrieves the current account configuration including leverage and margin mode.

#### Request Parameters
None

#### Response Example
```json
{
    "code": "0",
    "data": [{
        "acctLv": "2",
        "autoLoan": false,
        "ctIsoMode": "automatic",
        "greeksType": "PA",
        "level": "1",
        "levelTmp": "1",
        "mgnIsoMode": "automatic",
        "posMode": "long_short_mode",
        "uid": "754263"
    }],
    "msg": ""
}
```

### Get Account Balance
```http
GET /api/v5/account/balance
```

Returns the balance details for all assets in the account.

#### Request Parameters
None

#### Response Example
```json
{
    "code": "0",
    "data": [{
        "adjEq": "10679688.0460531643092577",
        "details": [{
            "availBal": "10679688.0460531643092577",
            "availEq": "10679688.0460531643092577",
            "cashBal": "10679688.0460531643092577",
            "ccy": "USDT",
            "crossLiab": "0",
            "disEq": "10679688.0460531643092577",
            "eq": "10679688.0460531643092577",
            "eqUsd": "10679688.0460531643092577",
            "frozenBal": "0",
            "interest": "0",
            "isoEq": "0",
            "isoLiab": "0",
            "isoUpl": "0",
            "liab": "0",
            "maxLoan": "10000",
            "mgnRatio": "",
            "notionalLever": "0",
            "ordFrozen": "0",
            "stgyEq": "0",
            "twap": "0",
            "uTime": "1597026383085",
            "upl": "0",
            "uplLiab": "0"
        }],
        "imr": "0",
        "isoEq": "0",
        "mgnRatio": "",
        "mmr": "0",
        "notionalUsd": "",
        "ordFroz": "0",
        "totalEq": "10679688.0460531643092577",
        "uTime": "1597026383085"
    }]
}
```

### Get Positions
```http
GET /api/v5/account/positions
```

Returns information about current positions.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instType | String | No | Instrument type: MARGIN, SWAP, FUTURES, OPTION |
| instId | String | No | Instrument ID |
| posId | String | No | Position ID |

#### Response Example
```json
{
    "code": "0",
    "data": [{
        "adl": "1",
        "availPos": "1",
        "avgPx": "2566.31",
        "cTime": "1597026383085",
        "ccy": "USDT",
        "deltaBS": "",
        "deltaPA": "",
        "gammaBS": "",
        "gammaPA": "",
        "imr": "",
        "instId": "BTC-USDT-SWAP",
        "instType": "SWAP",
        "interest": "0",
        "last": "2566.22",
        "lever": "10",
        "liab": "",
        "liabCcy": "",
        "liqPx": "2261.2533675",
        "markPx": "2566.22",
        "margin": "0.0003896645377994",
        "mgnMode": "isolated",
        "mgnRatio": "170.99370509",
        "mmr": "0.0000779329075599",
        "notionalUsd": "2566.22",
        "optVal": "",
        "pos": "1",
        "posCcy": "",
        "posId": "1234567",
        "posSide": "long",
        "thetaBS": "",
        "thetaPA": "",
        "tradeId": "123",
        "uTime": "1597026383085",
        "upl": "0.00009786",
        "uplRatio": "0.0047",
        "vegaBS": "",
        "vegaPA": ""
    }]
}
```

### Get Account and Position Risk
```http
GET /api/v5/account/account-position-risk
```

Returns account and position risk data.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instType | String | No | Instrument type |

#### Response Example
```json
{
    "code": "0",
    "data": [{
        "adjEq": "10679688.0460531643092577",
        "balData": [{
            "ccy": "USDT",
            "disEq": "10679688.0460531643092577",
            "eq": "10679688.0460531643092577"
        }],
        "posData": [{
            "baseBal": "1",
            "ccy": "BTC",
            "instId": "BTC-USDT",
            "instType": "SPOT",
            "mgnMode": "cross",
            "notionalCcy": "0.1",
            "notionalUsd": "2000",
            "pos": "1",
            "posCcy": "BTC",
            "posId": "1234567",
            "posSide": "long"
        }],
        "ts": "1597026383085"
    }]
}
```

### Get Bills History
```http
GET /api/v5/account/bills
```

Returns the bills history of the account.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| instType | String | No | Instrument type |
| ccy | String | No | Currency |
| mgnMode | String | No | Margin mode |
| ctType | String | No | Contract type |
| type | String | No | Bill type |
| subType | String | No | Bill sub-type |
| after | String | No | Pagination of data to return records earlier than the requested bill ID |
| before | String | No | Pagination of data to return records newer than the requested bill ID |
| limit | String | No | Number of results per request, maximum 100, default 100 |

#### Response Example
```json
{
    "code": "0",
    "data": [{
        "bal": "0.0000819307998198",
        "balChg": "0.0000819307998198",
        "billId": "12345",
        "ccy": "BTC",
        "execType": "",
        "fee": "0",
        "from": "",
        "instId": "",
        "instType": "",
        "mgnMode": "",
        "notes": "",
        "ordId": "",
        "pnl": "",
        "posBal": "",
        "posBalChg": "",
        "subType": "2",
        "sz": "0.0000819307998198",
        "to": "",
        "ts": "1597026383085",
        "type": "1"
    }]
}
```

## Best Practices

1. Account Monitoring
   - Regularly check account balance
   - Monitor position risks
   - Review bills history for unusual activity

2. Risk Management
   - Set appropriate leverage levels
   - Monitor margin ratios
   - Set up position alerts

3. Performance Optimization
   - Cache account configuration
   - Use WebSocket for real-time updates
   - Implement rate limit handling

## Error Handling

Common errors to handle:
- Insufficient balance
- Invalid position ID
- Rate limit exceeded
- Invalid margin mode
- System maintenance

## Example Implementation

```python
import requests
import time

def get_account_balance(api_key, secret_key, passphrase):
    # API endpoint
    url = "https://www.okx.com/api/v5/account/balance"
    
    # Generate timestamp
    timestamp = str(int(time.time()))
    
    # Create signature (implementation required)
    signature = create_signature(timestamp, "GET", "/api/v5/account/balance", "")
    
    # Headers
    headers = {
        "OK-ACCESS-KEY": api_key,
        "OK-ACCESS-SIGN": signature,
        "OK-ACCESS-TIMESTAMP": timestamp,
        "OK-ACCESS-PASSPHRASE": passphrase
    }
    
    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            return response.json()
        else:
            print(f"Error: {response.status_code}")
            return None
    except Exception as e:
        print(f"Exception: {str(e)}")
        return None
```
