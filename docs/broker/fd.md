# Fully Disclosed (FD) Broker

## Overview

The Fully Disclosed (FD) broker program offers two types of integration:
1. API Broker
2. OAuth Broker

This documentation covers both integration types and their features.

## API Broker

API brokers can integrate with OKX using API endpoints to provide trading services to their users.

### Features
- Direct API integration with OKX
- Users create API keys on OKX and provide them to the broker
- Trading through broker UI while using OKX as the exchange
- Custom commission rates
- High security with IP whitelisting

### Implementation

```python
import requests
import time
import hmac
import base64

class OKXFDBrokerClient:
    def __init__(self, api_key, secret_key, passphrase, is_sandbox=False):
        """
        Initialize OKX FD Broker Client
        
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
            
    def get_rebate_info(self, user_id=None):
        """
        Get user's broker rebate information
        
        Args:
            user_id (str, optional): User ID
            
        Returns:
            dict: Rebate information
        """
        params = {}
        if user_id:
            params["userId"] = user_id
            
        return self._request('GET', '/api/v5/broker/fd/rebate-info', params=params)
        
    def generate_rebate_details(self, begin_date, end_date=None, broker_type="0"):
        """
        Generate rebate details download link
        
        Args:
            begin_date (str): Begin date in YYYYMMDD format
            end_date (str, optional): End date in YYYYMMDD format
            broker_type (str): Broker type (0: API broker, 1: OAuth broker)
            
        Returns:
            dict: Download link information
        """
        data = {
            "begin": begin_date,
            "type": broker_type
        }
        
        if end_date:
            data["end"] = end_date
            
        return self._request('POST', '/api/v5/broker/fd/rebate-details', data=data)

# Usage Example
def main():
    client = OKXFDBrokerClient(
        api_key='YOUR_API_KEY',
        secret_key='YOUR_SECRET_KEY',
        passphrase='YOUR_PASSPHRASE'
    )
    
    # Get rebate information
    rebate_info = client.get_rebate_info()
    print(f"Rebate info: {rebate_info}")
    
    # Generate rebate details
    details = client.generate_rebate_details(
        begin_date='20240101',
        end_date='20240131',
        broker_type='0'
    )
    print(f"Rebate details: {details}")

if __name__ == "__main__":
    main()
```

## OAuth Broker

OAuth brokers can provide a more streamlined user experience through OAuth authentication.

### Features
- OAuth login for enhanced security
- Automatic API key creation
- Simplified user onboarding
- High commission rates
- Flexible commission management

### OAuth Implementation Steps

1. Register as an OAuth Broker
   - Contact OKX to become an OAuth broker
   - Receive OAuth credentials
   - Set up redirect URLs

2. Implement OAuth Flow
   ```python
   def get_oauth_url():
       """Generate OAuth URL for user authentication"""
       params = {
           "client_id": "YOUR_CLIENT_ID",
           "response_type": "code",
           "redirect_uri": "YOUR_REDIRECT_URI",
           "scope": "trade_read,trade_write",
           "state": "random_state_string"
       }
       return "https://www.okx.com/v3/oauth/authorize?" + urlencode(params)

   def handle_oauth_callback(code):
       """Handle OAuth callback and get access token"""
       data = {
           "client_id": "YOUR_CLIENT_ID",
           "client_secret": "YOUR_CLIENT_SECRET",
           "code": code,
           "grant_type": "authorization_code",
           "redirect_uri": "YOUR_REDIRECT_URI"
       }
       response = requests.post("https://www.okx.com/v3/oauth/token", json=data)
       return response.json()
   ```

3. Use Access Token
   ```python
   def make_api_request(access_token, endpoint):
       """Make API request using OAuth access token"""
       headers = {
           "Authorization": f"Bearer {access_token}",
           "Content-Type": "application/json"
       }
       response = requests.get(f"https://www.okx.com/api/v5/{endpoint}", headers=headers)
       return response.json()
   ```

## Commission Structure

### Commission Rates
- Up to 50% commission rates
- Commission from both new and existing users
- VIP user commission benefits
- Additional affiliate program rebates

### Commission Types
1. Trading Fee Commission
   - Based on trading volume
   - Tiered commission rates
   - Volume-based bonuses

2. Affiliate Commission
   - User referral rewards
   - Long-term revenue sharing
   - Multi-level commission structure

## Best Practices

1. Security
   - Implement IP whitelisting
   - Use HTTPS for all connections
   - Secure API key storage
   - Regular security audits

2. User Management
   - Clear user onboarding
   - Transparent fee structure
   - Real-time balance updates
   - Efficient support system

3. Commission Management
   - Regular commission tracking
   - Automated reporting
   - Clear revenue attribution
   - Timely payouts

## Error Handling

Common errors to handle:
1. Authentication failures
2. Rate limit exceeded
3. Invalid parameters
4. Network issues
5. Expired tokens
6. Commission calculation errors

## Security Considerations

1. API Security
   - Secure key storage
   - Request signing
   - IP restrictions
   - Access control

2. OAuth Security
   - State parameter validation
   - HTTPS only
   - Token management
   - Scope restrictions

3. User Data Protection
   - Data encryption
   - Privacy compliance
   - Audit logging
   - Access monitoring

## Advantages

1. High Commission
   - Up to 50% commission rates
   - Commission from VIP users
   - Combined affiliate rebates
   - Volume-based bonuses

2. Flexible Management
   - Custom commission rates
   - Dashboard management
   - Real-time reporting
   - Revenue analytics

3. Enhanced Security
   - FAST API implementation
   - IP whitelisting
   - OAuth integration
   - Secure trading

## Integration Steps

1. Initial Setup
   - Register as broker
   - Receive credentials
   - Set up environment
   - Configure security

2. API Integration
   - Implement endpoints
   - Set up authentication
   - Test connections
   - Monitor performance

3. User Management
   - Create onboarding flow
   - Set up user tracking
   - Implement reporting
   - Configure support

4. Commission System
   - Set up tracking
   - Configure rates
   - Implement reporting
   - Test calculations

## Support and Resources

1. Documentation
   - API documentation
   - Integration guides
   - Best practices
   - Code examples

2. Technical Support
   - Developer support
   - Integration assistance
   - Issue resolution
   - System updates

3. Business Support
   - Account management
   - Commission optimization
   - Marketing resources
   - Growth strategies
