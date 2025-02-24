# Transfer

## Overview

The transfer endpoints allow you to:
- Transfer funds between different accounts (Trading, Funding, etc.)
- Check transfer status
- View transfer history
- Manage sub-account transfers

## Endpoints

### Funds Transfer
```http
POST /api/v5/asset/transfer
```

Rate Limit: 2 requests per second
Rate limit rule: User ID + Currency
Permission: Trade

Transfer funds between your OKX accounts.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| ccy | String | Yes | Currency, e.g., BTC |
| amt | String | Yes | Transfer amount |
| from | String | Yes | Transfer from (6: Funding account, 18: Trading account) |
| to | String | Yes | Transfer to (6: Funding account, 18: Trading account) |
| subAcct | String | No | Sub-account name |
| type | String | No | 0: transfer within account, 1: master account to sub-account, 2: sub-account to master account |
| loanTrans | Boolean | No | Whether or not borrowed coins can be transferred out under Multi-currency margin and Portfolio margin |
| clientId | String | No | Client-supplied ID |
| omitPosRisk | String | No | Ignore position risk. Default is false |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "transId": "754147",
        "ccy": "USDT",
        "clientId": "",
        "from": "6",
        "amt": "0.1",
        "to": "18"
    }]
}
```

### Get Transfer State
```http
GET /api/v5/asset/transfer-state
```

Rate Limit: 10 requests per second
Rate limit rule: User ID
Permission: Read

Get the state of a transfer.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| transId | String | Yes | Transfer ID |
| clientId | String | No | Client-supplied ID |
| type | String | No | 0: transfer within account, 1: master account to sub-account, 2: sub-account to master account |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "transId": "754147",
        "clientId": "",
        "state": "success"
    }]
}
```

### Get Transfer History
```http
GET /api/v5/asset/transfer-history
```

Rate Limit: 6 requests per second
Rate limit rule: User ID
Permission: Read

Get the transfer history.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| ccy | String | No | Currency, e.g., BTC |
| type | String | No | 0: transfer within account, 1: master account to sub-account, 2: sub-account to master account |
| from | String | No | Transfer from (6: Funding account, 18: Trading account) |
| to | String | No | Transfer to (6: Funding account, 18: Trading account) |
| subAcct | String | No | Sub-account name |
| after | String | No | Pagination of data to return records earlier than the requested transId |
| before | String | No | Pagination of data to return records newer than the requested transId |
| limit | String | No | Number of results per request, maximum 100, default 100 |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "transId": "754147",
        "ccy": "USDT",
        "clientId": "",
        "from": "6",
        "amt": "0.1",
        "to": "18",
        "state": "success",
        "ts": "1597026383085"
    }]
}
```

## Best Practices

1. Transfer Management
   - Verify account balances before transfer
   - Keep track of transfer IDs
   - Monitor transfer status
   - Handle different transfer states

2. Error Prevention
   - Check transfer limits
   - Validate currency availability
   - Verify account permissions
   - Consider trading positions

3. Rate Limiting
   - Implement rate limit tracking
   - Handle rate limit errors
   - Use exponential backoff

## Example Implementation

```python
import requests
import time
import hmac
import base64

class OKXTransferClient:
    def __init__(self, api_key, secret_key, passphrase, is_sandbox=False):
        """
        Initialize OKX Transfer Client
        
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
            
    def transfer(self, currency, amount, from_account, to_account, 
                sub_account=None, transfer_type="0", client_id=None):
        """
        Transfer funds between accounts
        
        Args:
            currency (str): Currency code
            amount (str): Transfer amount
            from_account (str): Source account (6: Funding, 18: Trading)
            to_account (str): Destination account (6: Funding, 18: Trading)
            sub_account (str, optional): Sub-account name
            transfer_type (str, optional): Transfer type
            client_id (str, optional): Client-supplied ID
            
        Returns:
            dict: Transfer response
        """
        data = {
            "ccy": currency,
            "amt": amount,
            "from": from_account,
            "to": to_account,
            "type": transfer_type
        }
        
        if sub_account:
            data["subAcct"] = sub_account
        if client_id:
            data["clientId"] = client_id
            
        return self._request('POST', '/api/v5/asset/transfer', data=data)
        
    def get_transfer_state(self, transfer_id, client_id=None, transfer_type="0"):
        """
        Get transfer state
        
        Args:
            transfer_id (str): Transfer ID
            client_id (str, optional): Client-supplied ID
            transfer_type (str, optional): Transfer type
            
        Returns:
            dict: Transfer state
        """
        params = {
            "transId": transfer_id,
            "type": transfer_type
        }
        
        if client_id:
            params["clientId"] = client_id
            
        return self._request('GET', '/api/v5/asset/transfer-state', params=params)
        
    def get_transfer_history(self, currency=None, transfer_type=None, 
                           from_account=None, to_account=None, 
                           sub_account=None, limit=100):
        """
        Get transfer history
        
        Args:
            currency (str, optional): Currency code
            transfer_type (str, optional): Transfer type
            from_account (str, optional): Source account
            to_account (str, optional): Destination account
            sub_account (str, optional): Sub-account name
            limit (int, optional): Number of records to return
            
        Returns:
            dict: Transfer history
        """
        params = {}
        if currency:
            params["ccy"] = currency
        if transfer_type:
            params["type"] = transfer_type
        if from_account:
            params["from"] = from_account
        if to_account:
            params["to"] = to_account
        if sub_account:
            params["subAcct"] = sub_account
        if limit:
            params["limit"] = str(limit)
            
        return self._request('GET', '/api/v5/asset/transfer-history', params=params)

# Usage Example
def main():
    client = OKXTransferClient(
        api_key='YOUR_API_KEY',
        secret_key='YOUR_SECRET_KEY',
        passphrase='YOUR_PASSPHRASE'
    )
    
    # Transfer USDT from Funding to Trading account
    transfer = client.transfer(
        currency='USDT',
        amount='100',
        from_account='6',  # Funding account
        to_account='18',   # Trading account
    )
    print(f"Transfer response: {transfer}")
    
    if transfer and transfer['code'] == '0':
        # Check transfer state
        transfer_id = transfer['data'][0]['transId']
        state = client.get_transfer_state(transfer_id)
        print(f"Transfer state: {state}")
    
    # Get transfer history
    history = client.get_transfer_history(currency='USDT', limit=5)
    print(f"Recent transfers: {history}")

if __name__ == "__main__":
    main()
```

## Error Handling

Common errors to handle:
1. Insufficient balance
2. Invalid account type
3. Invalid currency
4. Transfer amount below minimum
5. Transfer amount above maximum
6. Rate limit exceeded
7. Invalid sub-account
8. Account restrictions

## Security Considerations

1. API Key Security
   - Use API keys with minimum necessary permissions
   - Regularly rotate API keys
   - Store API keys securely

2. Transfer Validation
   - Implement amount validation
   - Verify account types
   - Check transfer limits
   - Monitor unusual patterns

3. Account Security
   - Enable additional security features
   - Monitor account activity
   - Implement IP whitelisting
   - Use strong passwords

## Account Types

1. Funding Account (6)
   - Main account for deposits and withdrawals
   - Store funds not used for trading
   - Manage external transfers

2. Trading Account (18)
   - Account for spot and margin trading
   - Manage trading positions
   - Access to trading features

3. Sub-accounts
   - Separate accounts under main account
   - Individual balance management
   - Transfer restrictions configurable
   - Useful for team trading
