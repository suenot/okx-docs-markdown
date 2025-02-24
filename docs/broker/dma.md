# DMA Broker

## Overview

Direct Market Access (DMA) broker functionality allows brokers to:
- Manage sub-accounts
- Set and view fee rates
- Create and manage API keys for sub-accounts
- Monitor sub-account activities

## Endpoints

### Get Sub-account List
```http
GET /api/v5/broker/nd/subaccount-info
```

Rate Limit: 1 request per second
Rate limit rule: User ID
Permission: Read

Retrieve a list of sub-accounts and their information.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| subAcct | String | No | Sub-account name |
| page | String | No | Page number |
| limit | String | No | Number of results per request, maximum 100, default 100 |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "subAcct": "sub-account-1",
        "label": "test",
        "acctLv": "1",
        "uid": "123456789",
        "ts": "1597026383085"
    }]
}
```

### Get Sub-account Fee Rates
```http
GET /api/v5/broker/nd/subaccount-fee-rate
```

Rate Limit: 1 request per second
Rate limit rule: User ID
Permission: Read

Get the fee rates of sub-accounts.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| subAcct | String | Yes | Sub-account name |
| instType | String | No | Instrument type (SPOT, MARGIN, SWAP, FUTURES, OPTION) |
| instId | String | No | Instrument ID, e.g., BTC-USDT |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "subAcct": "sub-account-1",
        "instType": "SPOT",
        "taker": "0.0015",
        "maker": "0.001",
        "ts": "1597026383085"
    }]
}
```

### Create API Key for Sub-account
```http
POST /api/v5/broker/nd/subaccount-apikey
```

Rate Limit: 40 requests per second
Rate limit rule: User ID
Permission: Trade

Create an API key for a sub-account.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| subAcct | String | Yes | Sub-account name |
| label | String | No | API key label |
| passphrase | String | Yes | API key passphrase |
| ip | String | No | IP addresses allowed to access the API key, separate with comma |
| perm | String | Yes | Permissions (read_only, trade, withdraw) |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "subAcct": "sub-account-1",
        "label": "api-key-1",
        "apiKey": "0123456789ABCDEF",
        "secretKey": "0123456789ABCDEF0123456789ABCDEF",
        "passphrase": "passphrase123",
        "perm": "read_only,trade",
        "ip": "1.2.3.4",
        "ts": "1597026383085"
    }]
}
```

### Query Sub-account API Key
```http
GET /api/v5/broker/nd/subaccount-apikey
```

Rate Limit: 1 request per second
Rate limit rule: User ID
Permission: Read

Query the API keys of a sub-account.

#### Request Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| subAcct | String | Yes | Sub-account name |
| apiKey | String | No | API key |

#### Response Example
```json
{
    "code": "0",
    "msg": "",
    "data": [{
        "subAcct": "sub-account-1",
        "label": "api-key-1",
        "apiKey": "0123456789ABCDEF",
        "perm": "read_only,trade",
        "ip": "1.2.3.4",
        "ts": "1597026383085"
    }]
}
```

## Best Practices

1. Sub-account Management
   - Use meaningful sub-account names
   - Keep track of sub-account IDs
   - Monitor sub-account activities
   - Implement proper access controls

2. API Key Security
   - Use strong passphrases
   - Implement IP restrictions
   - Grant minimum necessary permissions
   - Rotate API keys regularly

3. Fee Management
   - Monitor fee rates regularly
   - Adjust rates based on volume
   - Track fee revenue
   - Consider competitive rates

## Example Implementation

```python
import requests
import time
import hmac
import base64

class OKXDMABrokerClient:
    def __init__(self, api_key, secret_key, passphrase, is_sandbox=False):
        """
        Initialize OKX DMA Broker Client
        
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
            
    def get_subaccount_list(self, subaccount=None, page=None, limit=100):
        """
        Get sub-account list
        
        Args:
            subaccount (str, optional): Sub-account name
            page (str, optional): Page number
            limit (int, optional): Number of results per request
            
        Returns:
            dict: Sub-account list
        """
        params = {}
        if subaccount:
            params["subAcct"] = subaccount
        if page:
            params["page"] = page
        if limit:
            params["limit"] = str(limit)
            
        return self._request('GET', '/api/v5/broker/nd/subaccount-info', params=params)
        
    def get_subaccount_fee_rates(self, subaccount, inst_type=None, inst_id=None):
        """
        Get sub-account fee rates
        
        Args:
            subaccount (str): Sub-account name
            inst_type (str, optional): Instrument type
            inst_id (str, optional): Instrument ID
            
        Returns:
            dict: Fee rates
        """
        params = {"subAcct": subaccount}
        if inst_type:
            params["instType"] = inst_type
        if inst_id:
            params["instId"] = inst_id
            
        return self._request('GET', '/api/v5/broker/nd/subaccount-fee-rate', params=params)
        
    def create_subaccount_apikey(self, subaccount, passphrase, permissions,
                                label=None, ip=None):
        """
        Create API key for sub-account
        
        Args:
            subaccount (str): Sub-account name
            passphrase (str): API key passphrase
            permissions (str): Permissions (read_only, trade, withdraw)
            label (str, optional): API key label
            ip (str, optional): Allowed IP addresses
            
        Returns:
            dict: API key details
        """
        data = {
            "subAcct": subaccount,
            "passphrase": passphrase,
            "perm": permissions
        }
        
        if label:
            data["label"] = label
        if ip:
            data["ip"] = ip
            
        return self._request('POST', '/api/v5/broker/nd/subaccount-apikey', data=data)
        
    def query_subaccount_apikey(self, subaccount, api_key=None):
        """
        Query sub-account API keys
        
        Args:
            subaccount (str): Sub-account name
            api_key (str, optional): API key
            
        Returns:
            dict: API key information
        """
        params = {"subAcct": subaccount}
        if api_key:
            params["apiKey"] = api_key
            
        return self._request('GET', '/api/v5/broker/nd/subaccount-apikey', params=params)

# Usage Example
def main():
    client = OKXDMABrokerClient(
        api_key='YOUR_API_KEY',
        secret_key='YOUR_SECRET_KEY',
        passphrase='YOUR_PASSPHRASE'
    )
    
    # Get sub-account list
    subaccounts = client.get_subaccount_list(limit=5)
    print(f"Sub-accounts: {subaccounts}")
    
    # Get fee rates for a sub-account
    if subaccounts and subaccounts['code'] == '0' and subaccounts['data']:
        subaccount = subaccounts['data'][0]['subAcct']
        fee_rates = client.get_subaccount_fee_rates(
            subaccount=subaccount,
            inst_type='SPOT'
        )
        print(f"Fee rates: {fee_rates}")
    
    # Create API key for sub-account
    new_key = client.create_subaccount_apikey(
        subaccount='sub-account-1',
        passphrase='strong-passphrase',
        permissions='read_only,trade',
        label='trading-bot',
        ip='1.2.3.4'
    )
    print(f"New API key: {new_key}")

if __name__ == "__main__":
    main()
```

## Error Handling

Common errors to handle:
1. Invalid sub-account
2. Permission denied
3. Rate limit exceeded
4. Invalid API key parameters
5. Network issues
6. Invalid fee rate parameters
7. Duplicate API key label

## Security Considerations

1. API Key Management
   - Secure storage of API keys
   - Regular key rotation
   - IP restrictions
   - Minimum necessary permissions

2. Sub-account Security
   - Strong passwords
   - Activity monitoring
   - Access controls
   - Regular audits

3. Network Security
   - SSL/TLS encryption
   - IP whitelisting
   - Request signing
   - Timestamp validation

## DMA Broker Features

1. Sub-account Management
   - Create and manage sub-accounts
   - Monitor sub-account activities
   - Set trading permissions
   - Configure risk limits

2. Fee Management
   - Set custom fee rates
   - Volume-based tiers
   - Fee reports
   - Revenue sharing

3. API Access
   - Create API keys
   - Set permissions
   - Monitor API usage
   - Manage IP restrictions

4. Risk Management
   - Position limits
   - Order size limits
   - Trading pair restrictions
   - Leverage controls
