# Authentication

## API Key Creation

To access private endpoints, you need to create an API key through the OKX website:

1. Log in to your OKX account
2. Navigate to "API Management"
3. Click "Create API Key"
4. Select permissions:
   - Read: Market data and account information
   - Trade: Place and manage orders
   - Withdraw: Transfer funds and manage withdrawals
5. Configure IP whitelist (recommended)
6. Save your API credentials:
   - API Key
   - Secret Key
   - Passphrase

## Authentication Components

### Required Headers

All authenticated endpoints require the following HTTP headers:

```
OK-ACCESS-KEY: Your API key
OK-ACCESS-SIGN: Signature
OK-ACCESS-TIMESTAMP: UTC timestamp
OK-ACCESS-PASSPHRASE: Your API passphrase
```

### Timestamp

- Must be in ISO format: YYYY-MM-DDThh:mm:ss.sssZ
- Server accepts requests within ±30 seconds of server time
- Use `/api/v5/public/time` to synchronize time

### Request Signing

1. Create prehash string:
```
timestamp + method + requestPath + body
```

2. Sign using HMAC SHA256:
```python
# Python example
import hmac
import base64
from datetime import datetime

def sign(timestamp, method, request_path, body, secret_key):
    message = timestamp + method + request_path + body
    mac = hmac.new(
        bytes(secret_key, encoding='utf8'),
        bytes(message, encoding='utf-8'),
        digestmod='sha256'
    )
    d = mac.digest()
    return base64.b64encode(d).decode()
```

## Example Request

```python
import requests
import time

# API credentials
api_key = "YOUR-API-KEY"
secret_key = "YOUR-SECRET-KEY"
passphrase = "YOUR-PASSPHRASE"

# Request details
timestamp = datetime.utcnow().isoformat()[:-3] + 'Z'
method = 'GET'
request_path = '/api/v5/account/balance'
body = ''

# Generate signature
signature = sign(timestamp, method, request_path, body, secret_key)

# Headers
headers = {
    'OK-ACCESS-KEY': api_key,
    'OK-ACCESS-SIGN': signature,
    'OK-ACCESS-TIMESTAMP': timestamp,
    'OK-ACCESS-PASSPHRASE': passphrase
}

# Make request
url = 'https://www.okx.com' + request_path
response = requests.get(url, headers=headers)
print(response.json())
```

## Demo Trading

For testing, use demo trading API endpoints:
- REST base URL: `https://www.okx.com`
- WebSocket public channel: `wss://ws.okx.com:8443/ws/v5/public`
- WebSocket private channel: `wss://ws.okx.com:8443/ws/v5/private`

## Security Best Practices

1. API Key Protection
   - Never share your API credentials
   - Store credentials securely
   - Use environment variables
   - Regularly rotate API keys

2. IP Whitelist
   - Enable IP whitelist
   - Only allow trusted IPs
   - Update list when necessary

3. Permission Management
   - Use minimum required permissions
   - Create separate keys for different purposes
   - Regularly audit API key usage

4. Request Validation
   - Verify server SSL certificate
   - Check response signatures
   - Validate timestamps

5. Error Handling
   - Implement proper error handling
   - Monitor for unauthorized access
   - Log authentication failures
