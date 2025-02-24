# Sub-Account System

## Overview

The OKX sub-account system allows master accounts to create and manage multiple sub-accounts. This feature is particularly useful for institutional traders, fund managers, and professional traders who need to manage multiple trading strategies or client accounts separately.

## Features

1. Account Management
   - Create sub-accounts
   - View sub-account list
   - Reset sub-account API keys
   - Set transfer permissions

2. Balance Management
   - View trading balances
   - View funding balances
   - Check maximum withdrawals
   - Monitor transfer history

3. Transfer Management
   - Transfer between sub-accounts
   - Manage transfer permissions
   - Track transfer history

## API Endpoints

### Account Management

#### Get Sub-Account List
```http
GET /api/v5/users/subaccount/list
```
- Rate Limit: 2 requests per 2 seconds
- Permission: Read
- Description: Retrieves a list of all sub-accounts under the master account

#### Reset Sub-Account API Key
```http
POST /api/v5/users/subaccount/modify-apikey
```
- Rate Limit: 1 request per second
- Permission: Trade
- Description: Resets or modifies API key settings for a sub-account

### Balance Management

#### Get Trading Balance
```http
GET /api/v5/account/subaccount/balances
```
- Rate Limit: 6 requests per 2 seconds
- Permission: Read
- Description: Retrieves the trading account balance of a sub-account

#### Get Funding Balance
```http
GET /api/v5/asset/subaccount/balances
```
- Rate Limit: 6 requests per 2 seconds
- Permission: Read
- Description: Retrieves the funding account balance of a sub-account

#### Get Maximum Withdrawals
```http
GET /api/v5/account/subaccount/max-withdrawal
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Read
- Description: Gets the maximum withdrawal amount for a sub-account

### Transfer Management

#### Transfer Between Sub-Accounts
```http
POST /api/v5/asset/subaccount/transfer
```
- Rate Limit: 1 request per second
- Permission: Trade
- Description: Transfers assets between sub-accounts

#### Get Transfer History
```http
GET /api/v5/asset/subaccount/bills
```
- Rate Limit: 6 requests per second
- Permission: Read
- Description: Retrieves the transfer history between sub-accounts

#### Get Managed Sub-Account Transfer History
```http
GET /api/v5/asset/subaccount/managed-subaccount-bills
```
- Rate Limit: 6 requests per second
- Permission: Read
- Description: Retrieves the transfer history for managed sub-accounts

#### Set Transfer Permission
```http
POST /api/v5/users/subaccount/set-transfer-out
```
- Rate Limit: 1 request per second
- Permission: Trade
- Description: Sets the transfer permission for a sub-account

## Implementation Example

```python
import okx.Account as Account
import okx.SubAccount as SubAccount

# Initialize clients
accountAPI = Account.AccountAPI(api_key, secret_key, passphrase)
subAccountAPI = SubAccount.SubAccountAPI(api_key, secret_key, passphrase)

def manage_sub_accounts():
    """Example of sub-account management"""
    
    # Get sub-account list
    result = subAccountAPI.get_subaccount_list()
    print(f"Sub-accounts: {result}")
    
    # Get sub-account trading balance
    result = accountAPI.get_subaccount_balance(subAcct="sub-account-name")
    print(f"Trading balance: {result}")
    
    # Transfer between sub-accounts
    result = subAccountAPI.subaccount_transfer(
        ccy="USDT",
        amt="100",
        from_="6",  # 6: Funding account
        to="18",    # 18: Trading account
        fromSubAccount="source-subaccount",
        toSubAccount="target-subaccount"
    )
    print(f"Transfer result: {result}")
    
    # Get transfer history
    result = subAccountAPI.get_subaccount_bills()
    print(f"Transfer history: {result}")

def set_sub_account_permissions():
    """Example of setting sub-account permissions"""
    
    # Set transfer permission
    result = subAccountAPI.set_transfer_out_permission(
        subAcct="sub-account-name",
        canTransOut=False  # Disable transfers out
    )
    print(f"Permission set result: {result}")
    
    # Reset API key
    result = subAccountAPI.reset_subaccount_apikey(
        subAcct="sub-account-name",
        apiKey="old-api-key",
        ip="192.168.1.1",
        perm="read_only"
    )
    print(f"API key reset result: {result}")
```

## Best Practices

1. Security
   - Regularly rotate API keys
   - Implement IP restrictions
   - Monitor account activity
   - Set appropriate permissions

2. Balance Management
   - Regular balance reconciliation
   - Monitor transfer limits
   - Track withdrawal limits
   - Set alerts for large transfers

3. Account Structure
   - Logical naming convention
   - Clear account hierarchy
   - Documented account purposes
   - Regular account review

4. Risk Management
   - Set transfer limits
   - Monitor trading activity
   - Regular permission review
   - Audit trail maintenance

## Error Handling

Common errors to handle:

1. Insufficient Balance
```json
{
    "code": "50001",
    "msg": "Insufficient balance"
}
```

2. Invalid Permission
```json
{
    "code": "50002",
    "msg": "No permission to transfer out"
}
```

3. Account Not Found
```json
{
    "code": "50003",
    "msg": "Sub-account does not exist"
}
```

4. Rate Limit Exceeded
```json
{
    "code": "50004",
    "msg": "Request too frequent"
}
```

## Security Considerations

1. API Key Management
   - Generate separate API keys for each sub-account
   - Implement key rotation policy
   - Use IP whitelisting
   - Set appropriate permissions

2. Transfer Security
   - Implement approval workflows
   - Set transfer limits
   - Monitor unusual activity
   - Regular audits

3. Access Control
   - Role-based access
   - Two-factor authentication
   - Activity logging
   - Regular permission review

## Rate Limits

| Endpoint Category | Rate Limit | Rule |
|------------------|------------|------|
| Account List | 2 req/2s | User ID |
| Balance Query | 6 req/2s | User ID |
| Transfers | 1 req/s | User ID |
| History Query | 6 req/s | User ID |
| Max Withdrawal | 20 req/2s | User ID |

## Support and Resources

1. Documentation
   - API documentation
   - SDK examples
   - Best practices
   - Error codes

2. Technical Support
   - 24/7 support
   - Integration assistance
   - Issue resolution
   - System updates

3. Account Management
   - Account setup help
   - Permission configuration
   - Balance management
   - Transfer assistance
