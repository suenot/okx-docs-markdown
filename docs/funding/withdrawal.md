# Withdrawal

## Overview

The withdrawal endpoints allow you to:
- Withdraw funds from your account
- Cancel pending withdrawals
- View withdrawal history
- Get withdrawal payment methods
- Check maximum withdrawal amounts

## Endpoints

### Withdrawal
```http
POST /api/v5/asset/withdrawal
```

Rate Limit: 6 requests per second
Rate limit rule: User ID
Permission: Withdraw

Withdraw funds from your OKX account.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| ccy | String | Yes | Currency, e.g., BTC |
| amt | String | Yes | Withdrawal amount. Withdrawal fee is not included |
| dest | String | Yes | Withdrawal destination (3: internal, 4: on-chain) |
| toAddr | String | Yes | Withdrawal address. For internal transfers, this should be the recipient's email or phone number |
| fee | String | Yes | Transaction fee |
| chain | String | No | Chain name, e.g., USDT-ERC20. Default is the main chain |
| areaCode | String | No | Area code for the phone number. Required when toAddr is a phone number |
| clientId | String | No | Client-supplied ID |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "amt": "0.1",
        "wdId": "67485",
        "ccy": "BTC",
        "clientId": "",
        "chain": "BTC-Bitcoin"
    }]
}
```

### Cancel Withdrawal
```http
POST /api/v5/asset/cancel-withdrawal
```

Rate Limit: 6 requests per second
Rate limit rule: User ID
Permission: Trade

Cancel an ongoing withdrawal.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| wdId | String | Yes | Withdrawal ID |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "wdId": "67485"
    }]
}
```

### Get Withdrawal History
```http
GET /api/v5/asset/withdrawal-history
```

Rate Limit: 6 requests per second
Rate limit rule: User ID
Permission: Read

Get the withdrawal history.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| ccy | String | No | Currency, e.g., BTC |
| wdId | String | No | Withdrawal ID |
| clientId | String | No | Client-supplied ID |
| txId | String | No | Hash record of the withdrawal |
| type | String | No | Withdrawal type (3: internal, 4: on-chain) |
| state | String | No | Status (-3: canceling, -2: canceled, -1: failed, 0: pending, 1: sending, 2: sent, 3: awaiting email verification, 4: awaiting manual verification, 5: awaiting identity verification) |
| after | String | No | Pagination of data to return records earlier than the requested wdId |
| before | String | No | Pagination of data to return records newer than the requested wdId |
| limit | String | No | Number of results per request, maximum 100, default 100 |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "ccy": "BTC",
        "chain": "BTC-Bitcoin",
        "amt": "0.1",
        "ts": "1597026383085",
        "from": "",
        "to": "bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh",
        "txId": "e5ea63a68ab5b8ea676486f36bb46c58639ad6a31cf27f40d1f074109960f6f5",
        "state": "2",
        "wdId": "67485",
        "clientId": ""
    }]
}
```

### Get Withdrawal Payment Methods
```http
GET /api/v5/asset/withdrawal-payment-methods
```

Rate Limit: 3 requests per second
Rate limit rule: User ID
Permission: Read

Get available withdrawal payment methods.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| ccy | String | Yes | Currency |

## Best Practices

1. Withdrawal Security
   - Always verify withdrawal addresses
   - Use whitelisted addresses when possible
   - Enable additional security features
   - Monitor withdrawal limits

2. Transaction Management
   - Keep track of withdrawal IDs
   - Monitor withdrawal status
   - Handle different withdrawal states
   - Implement proper error handling

3. Fee Management
   - Check fee before withdrawal
   - Consider different chain fees
   - Monitor network congestion

## Example Implementation

```python
import requests
import time
import hmac
import base64

class OKXWithdrawalClient:
    def __init__(self, api_key, secret_key, passphrase, is_sandbox=False):
        """
        Initialize OKX Withdrawal Client
        
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
        
    def _request(self, method, path, params=None, data=None):
        """Make request to OKX API"""
        timestamp = str(int(time.time()))
        
        headers = {
            "OK-ACCESS-KEY": self.api_key,
            "OK-ACCESS-SIGN": self._generate_signature(timestamp, method, path, data or ''),
            "OK-ACCESS-TIMESTAMP": timestamp,
            "OK-ACCESS-PASSPHRASE": self.passphrase,
            "Content-Type": "application/json"
        }
        
        url = self.base_url + path
        try:
            response = requests.request(method, url, headers=headers, 
                                     params=params, json=data if data else None)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"API request failed: {str(e)}")
            return None
            
    def withdraw(self, currency, amount, dest, to_addr, fee, chain=None, 
                area_code=None, client_id=None):
        """
        Withdraw funds
        
        Args:
            currency (str): Currency code
            amount (str): Withdrawal amount
            dest (str): Destination (3: internal, 4: on-chain)
            to_addr (str): Withdrawal address
            fee (str): Transaction fee
            chain (str, optional): Chain name
            area_code (str, optional): Area code for phone number
            client_id (str, optional): Client-supplied ID
            
        Returns:
            dict: Withdrawal response
        """
        data = {
            "ccy": currency,
            "amt": amount,
            "dest": dest,
            "toAddr": to_addr,
            "fee": fee
        }
        
        if chain:
            data["chain"] = chain
        if area_code:
            data["areaCode"] = area_code
        if client_id:
            data["clientId"] = client_id
            
        return self._request('POST', '/api/v5/asset/withdrawal', data=data)
        
    def cancel_withdrawal(self, withdrawal_id):
        """
        Cancel withdrawal
        
        Args:
            withdrawal_id (str): Withdrawal ID
            
        Returns:
            dict: Cancellation response
        """
        data = {"wdId": withdrawal_id}
        return self._request('POST', '/api/v5/asset/cancel-withdrawal', data=data)
        
    def get_withdrawal_history(self, currency=None, withdrawal_id=None, 
                             client_id=None, txid=None, state=None, limit=100):
        """
        Get withdrawal history
        
        Args:
            currency (str, optional): Currency code
            withdrawal_id (str, optional): Withdrawal ID
            client_id (str, optional): Client-supplied ID
            txid (str, optional): Transaction hash
            state (str, optional): Withdrawal state
            limit (int, optional): Number of records to return
            
        Returns:
            dict: Withdrawal history
        """
        params = {}
        if currency:
            params["ccy"] = currency
        if withdrawal_id:
            params["wdId"] = withdrawal_id
        if client_id:
            params["clientId"] = client_id
        if txid:
            params["txId"] = txid
        if state:
            params["state"] = state
        if limit:
            params["limit"] = str(limit)
            
        return self._request('GET', '/api/v5/asset/withdrawal-history', params=params)
        
    def get_withdrawal_methods(self, currency):
        """
        Get withdrawal payment methods
        
        Args:
            currency (str): Currency code
            
        Returns:
            dict: Available payment methods
        """
        params = {"ccy": currency}
        return self._request('GET', '/api/v5/asset/withdrawal-payment-methods', params=params)

# Usage Example
def main():
    client = OKXWithdrawalClient(
        api_key='YOUR_API_KEY',
        secret_key='YOUR_SECRET_KEY',
        passphrase='YOUR_PASSPHRASE'
    )
    
    # Get withdrawal methods for BTC
    methods = client.get_withdrawal_methods('BTC')
    print(f"Withdrawal methods: {methods}")
    
    # Withdraw BTC
    withdrawal = client.withdraw(
        currency='BTC',
        amount='0.1',
        dest='4',  # on-chain withdrawal
        to_addr='bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh',
        fee='0.0005',
        chain='BTC-Bitcoin'
    )
    print(f"Withdrawal response: {withdrawal}")
    
    # Get withdrawal history
    history = client.get_withdrawal_history(currency='BTC', limit=5)
    print(f"Recent withdrawals: {history}")

if __name__ == "__main__":
    main()
```

## Error Handling

Common errors to handle:
1. Insufficient balance
2. Invalid withdrawal address
3. Withdrawal amount below minimum
4. Withdrawal amount above maximum
5. Invalid chain selection
6. Rate limit exceeded
7. Network congestion

## Security Considerations

1. API Key Security
   - Use API keys with minimum necessary permissions
   - Regularly rotate API keys
   - Store API keys securely

2. Address Validation
   - Implement address format validation
   - Use address whitelisting
   - Double-check addresses before withdrawal

3. Amount Validation
   - Check against account balance
   - Verify against withdrawal limits
   - Consider fee requirements
