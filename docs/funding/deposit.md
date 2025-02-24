# Deposit

## Overview

The deposit endpoints allow you to:
- Get deposit addresses for cryptocurrencies
- View deposit history
- Get currency information
- Set up notifications

## Endpoints

### Get Deposit Address
```http
GET /api/v5/asset/deposit-address
```

Rate Limit: 6 requests per second
Rate limit rule: User ID
Permission: Read

Retrieves the deposit addresses of currencies, including previously-used addresses.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| ccy | String | Yes | Currency, e.g., BTC |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "addr": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "tag": "",
        "memo": "",
        "pmtId": "",
        "ccy": "BTC",
        "chain": "BTC-Bitcoin",
        "to": "6",
        "selected": true
    }]
}
```

### Get Deposit History
```http
GET /api/v5/asset/deposit-history
```

Rate Limit: 6 requests per second
Rate limit rule: User ID
Permission: Read

Returns the deposit history.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| ccy | String | No | Currency, e.g., BTC |
| txId | String | No | Hash record of the deposit |
| fromWdId | String | No | Start withdrawal ID |
| limit | String | No | Number of results per request. Maximum is 100, default is 100 |
| state | String | No | State (0: waiting for confirmation, 1: deposit credited, 2: deposit successful) |
| after | String | No | Pagination of data to return records earlier than the requested ts |
| before | String | No | Pagination of data to return records newer than the requested ts |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "ccy": "BTC",
        "chain": "BTC-Bitcoin",
        "amt": "0.01",
        "from": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "to": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "txId": "f24fb891b3c8123c2ba833834f9c37594d6c0899",
        "ts": "1597026383085",
        "state": "2",
        "depId": "12344321"
    }]
}
```

### Get Currencies
```http
GET /api/v5/asset/currencies
```

Rate Limit: 6 requests per second
Rate limit rule: User ID
Permission: Read

Retrieve a list of all currencies available which are related to the current account's KYC entity.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| ccy | String | No | Single currency or multiple currencies separated with comma, e.g. `BTC` or `BTC,ETH` |

## Best Practices

1. Address Management
   - Always verify addresses before use
   - Store addresses securely
   - Use whitelisting when possible

2. Transaction Monitoring
   - Monitor deposit status regularly
   - Implement notifications for status changes
   - Track confirmation counts

3. Security
   - Validate deposit amounts
   - Check for suspicious activity
   - Implement address whitelisting

## Example Implementation

```python
import requests
import time
import hmac
import base64

class OKXDepositClient:
    def __init__(self, api_key, secret_key, passphrase, is_sandbox=False):
        """
        Initialize OKX Deposit Client
        
        Args:
            api_key (str): Your API key
            secret_key (str): Your secret key
            passphrase (str): Your API passphrase
            is_sandbox (bool): Whether to use sandbox environment
        """
        self.api_key = api_key
        self.secret_key = secret_key
        self.passphrase = passphrase
        self.base_url = "https://www.okx.com" if not is_sandbox else "https://www.okx.com/api/v5/sandbox"
        
    def _generate_signature(self, timestamp, method, request_path, body=''):
        """Generate signature for request"""
        message = timestamp + method + request_path + body
        mac = hmac.new(
            bytes(self.secret_key, encoding='utf8'),
            bytes(message, encoding='utf-8'),
            digestmod='sha256'
        )
        d = mac.digest()
        return base64.b64encode(d).decode()
        
    def _request(self, method, path, params=None):
        """Make request to OKX API"""
        timestamp = str(int(time.time()))
        
        headers = {
            "OK-ACCESS-KEY": self.api_key,
            "OK-ACCESS-SIGN": self._generate_signature(timestamp, method, path),
            "OK-ACCESS-TIMESTAMP": timestamp,
            "OK-ACCESS-PASSPHRASE": self.passphrase
        }
        
        url = self.base_url + path
        try:
            response = requests.request(method, url, headers=headers, params=params)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"API request failed: {str(e)}")
            return None
        
    def get_deposit_address(self, currency):
        """
        Get deposit address for a specific currency
        
        Args:
            currency (str): Currency code (e.g., 'BTC')
            
        Returns:
            dict: Deposit address information
        """
        params = {'ccy': currency}
        return self._request('GET', '/api/v5/asset/deposit-address', params)
        
    def get_deposit_history(self, currency=None, txid=None, state=None, limit=100):
        """
        Get deposit history
        
        Args:
            currency (str, optional): Currency code
            txid (str, optional): Transaction ID
            state (str, optional): Deposit state
            limit (int, optional): Number of records to return
            
        Returns:
            dict: Deposit history
        """
        params = {}
        if currency:
            params['ccy'] = currency
        if txid:
            params['txId'] = txid
        if state:
            params['state'] = state
        if limit:
            params['limit'] = str(limit)
            
        return self._request('GET', '/api/v5/asset/deposit-history', params)
        
    def get_currencies(self, currencies=None):
        """
        Get currency information
        
        Args:
            currencies (str, optional): Single currency or multiple currencies
                                    separated by comma (e.g., 'BTC' or 'BTC,ETH')
            
        Returns:
            dict: Currency information
        """
        params = {}
        if currencies:
            params['ccy'] = currencies
            
        return self._request('GET', '/api/v5/asset/currencies', params)

# Usage Example
def main():
    client = OKXDepositClient(
        api_key='YOUR_API_KEY',
        secret_key='YOUR_SECRET_KEY',
        passphrase='YOUR_PASSPHRASE'
    )
    
    # Get deposit address for BTC
    btc_address = client.get_deposit_address('BTC')
    print(f"BTC deposit address: {btc_address}")
    
    # Get deposit history for BTC
    history = client.get_deposit_history(currency='BTC', limit=5)
    print(f"Recent deposits: {history}")
    
    # Get currency information for BTC and ETH
    currencies = client.get_currencies('BTC,ETH')
    print(f"Currency info: {currencies}")

if __name__ == "__main__":
    main()
```

## Error Handling

Common errors to handle:
1. Invalid currency
2. Network congestion
3. Maintenance mode
4. Rate limit exceeded (6 requests per second)
5. Invalid address format

## Network Support

Supported networks should be checked using the `/api/v5/asset/currencies` endpoint, as they may change over time. Common examples include:
- BTC: Bitcoin, Lightning Network
- ETH: ERC20
- USDT: ERC20, TRC20, Omni
- XRP: Ripple Network
- SOL: Solana Network
