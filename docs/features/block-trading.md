# Block Trading

## Overview

Block trading on OKX allows institutional clients and high-net-worth individuals to execute large orders outside the regular order book. This feature helps minimize market impact and provides better price execution for large trades.

## Features

1. RFQ (Request for Quote) System
   - Create and manage RFQs
   - Receive competitive quotes
   - Execute trades at agreed prices
   - Cancel or modify requests

2. Quote Management
   - Create and send quotes
   - Manage quote lifecycle
   - Track quote status
   - Cancel quotes

3. Market Making Program (MMP)
   - Configure MMP settings
   - Monitor MMP status
   - Reset MMP when needed
   - Manage quote products

4. Trade Information
   - View block tickers
   - Access trade history
   - Monitor market activity
   - Track counterparties

## API Endpoints

### RFQ Management

#### Create RFQ
```http
POST /api/v5/rfq/create-rfq
```
- Rate Limit: 5 requests per 2 seconds; 150 requests per 12 hours
- Permission: Trade
- Description: Create a new Request for Quote

#### Cancel RFQ
```http
POST /api/v5/rfq/cancel-rfq
```
- Rate Limit: 5 requests per 2 seconds
- Permission: Trade
- Description: Cancel an existing RFQ

#### Cancel Multiple RFQs
```http
POST /api/v5/rfq/cancel-batch-rfqs
```
- Rate Limit: 2 requests per 2 seconds
- Permission: Trade
- Description: Cancel multiple RFQs at once

#### Cancel All RFQs
```http
POST /api/v5/rfq/cancel-all-rfqs
```
- Rate Limit: 2 requests per 2 seconds
- Permission: Trade
- Description: Cancel all active RFQs

### Quote Management

#### Create Quote
```http
POST /api/v5/rfq/create-quote
```
- Rate Limit: 50 requests per 2 seconds
- Permission: Trade
- Description: Create a new quote in response to an RFQ

#### Execute Quote
```http
POST /api/v5/rfq/execute-quote
```
- Rate Limit: 2 requests per 3 seconds
- Permission: Trade
- Description: Execute a received quote

#### Cancel Quote
```http
POST /api/v5/rfq/cancel-quote
```
- Rate Limit: 50 requests per 2 seconds
- Permission: Trade
- Description: Cancel an existing quote

#### Cancel Multiple Quotes
```http
POST /api/v5/rfq/cancel-batch-quotes
```
- Rate Limit: 2 requests per 2 seconds
- Permission: Trade
- Description: Cancel multiple quotes at once

### Market Making Program (MMP)

#### Set MMP Configuration
```http
POST /api/v5/rfq/mmp-config
```
- Rate Limit: 1 request per 10 seconds
- Permission: Trade
- Description: Configure MMP settings

#### Reset MMP Status
```http
POST /api/v5/rfq/reset-mmp
```
- Rate Limit: 5 requests per 2 seconds
- Permission: Trade
- Description: Reset MMP status when triggered

#### Get MMP Configuration
```http
GET /api/v5/rfq/get-mmp-config
```
- Rate Limit: 5 requests per 2 seconds
- Permission: Read
- Description: Retrieve current MMP settings

### Market Data

#### Get Block Tickers
```http
GET /api/v5/market/block-tickers
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Read
- Description: Get block trading market data

#### Get Block Trades
```http
GET /api/v5/market/block-trades
```
- Rate Limit: 5 requests per 2 seconds
- Permission: Read
- Description: Get block trade execution history

## Implementation Example

```python
import okx.BlockTrading as BlockTrading
import json

class OKXBlockTrading:
    def __init__(self, api_key, secret_key, passphrase, is_sandbox=False):
        """Initialize Block Trading client"""
        self.client = BlockTrading.BlockTradingAPI(
            api_key, secret_key, passphrase, is_sandbox
        )
        
    def create_rfq(self, counterparties, legs):
        """
        Create a new RFQ
        
        Args:
            counterparties (list): List of counterparty IDs
            legs (list): List of trading legs with instrument details
            
        Returns:
            dict: RFQ creation result
        """
        try:
            result = self.client.create_rfq(
                counterparties=counterparties,
                legs=legs,
                anonymous=False
            )
            return result
        except Exception as e:
            print(f"Error creating RFQ: {str(e)}")
            return None
            
    def create_quote(self, rfq_id, legs):
        """
        Create quote in response to RFQ
        
        Args:
            rfq_id (str): RFQ identifier
            legs (list): Quote legs with prices
            
        Returns:
            dict: Quote creation result
        """
        try:
            result = self.client.create_quote(
                rfqId=rfq_id,
                legs=legs,
                quoteType="firm"
            )
            return result
        except Exception as e:
            print(f"Error creating quote: {str(e)}")
            return None
            
    def execute_quote(self, rfq_id, quote_id):
        """
        Execute a received quote
        
        Args:
            rfq_id (str): RFQ identifier
            quote_id (str): Quote identifier
            
        Returns:
            dict: Quote execution result
        """
        try:
            result = self.client.execute_quote(
                rfqId=rfq_id,
                quoteId=quote_id
            )
            return result
        except Exception as e:
            print(f"Error executing quote: {str(e)}")
            return None
            
    def configure_mmp(self, timeout_ms=3000, quote_threshold=3):
        """
        Configure Market Making Program settings
        
        Args:
            timeout_ms (int): Quote timeout in milliseconds
            quote_threshold (int): Maximum quotes allowed
            
        Returns:
            dict: MMP configuration result
        """
        try:
            result = self.client.mmp_config(
                timeoutMs=timeout_ms,
                quoteThreshold=quote_threshold
            )
            return result
        except Exception as e:
            print(f"Error configuring MMP: {str(e)}")
            return None

# Usage Example
def main():
    block_trading = OKXBlockTrading(
        api_key='YOUR_API_KEY',
        secret_key='YOUR_SECRET_KEY',
        passphrase='YOUR_PASSPHRASE'
    )
    
    # Create RFQ
    legs = [{
        "sz": "1",
        "side": "buy",
        "instId": "BTC-USDT"
    }]
    
    rfq = block_trading.create_rfq(
        counterparties=["CP1", "CP2"],
        legs=legs
    )
    
    if rfq and rfq.get('code') == '0':
        rfq_id = rfq['data']['rfqId']
        
        # Create quote
        quote = block_trading.create_quote(
            rfq_id=rfq_id,
            legs=[{
                "px": "35000",
                "sz": "1",
                "instId": "BTC-USDT",
                "side": "sell"
            }]
        )
        
        if quote and quote.get('code') == '0':
            quote_id = quote['data']['quoteId']
            
            # Execute quote
            execution = block_trading.execute_quote(rfq_id, quote_id)
            print(f"Trade execution result: {execution}")

if __name__ == "__main__":
    main()
```

## Best Practices

1. RFQ Management
   - Set appropriate timeouts
   - Monitor quote responses
   - Validate counterparties
   - Track execution status

2. Quote Management
   - Implement quote validation
   - Monitor quote lifecycle
   - Handle quote rejections
   - Track quote performance

3. Risk Management
   - Set position limits
   - Monitor exposure
   - Implement price checks
   - Track counterparty risk

4. MMP Configuration
   - Set appropriate timeouts
   - Monitor quote quality
   - Track success rates
   - Adjust parameters as needed

## Error Handling

Common errors to handle:

1. Invalid RFQ
```json
{
    "code": "50001",
    "msg": "RFQ validation failed"
}
```

2. Quote Expired
```json
{
    "code": "50002",
    "msg": "Quote has expired"
}
```

3. MMP Triggered
```json
{
    "code": "50003",
    "msg": "MMP protection triggered"
}
```

4. Invalid Price
```json
{
    "code": "50004",
    "msg": "Price outside allowed range"
}
```

## Rate Limits

| Endpoint Category | Rate Limit | Rule |
|------------------|------------|------|
| Create RFQ | 5 req/2s | User ID |
| Create Quote | 50 req/2s | User ID |
| Execute Quote | 2 req/3s | User ID |
| Cancel Operations | 2 req/2s | User ID |
| Market Data | 20 req/2s | IP |
| MMP Config | 1 req/10s | User ID |

## Security Considerations

1. API Security
   - Secure key storage
   - Request signing
   - IP restrictions
   - Access monitoring

2. Trade Validation
   - Price validation
   - Size validation
   - Counterparty validation
   - Risk checks

3. MMP Protection
   - Quote monitoring
   - Automatic triggers
   - Reset procedures
   - Parameter tuning

## Support and Resources

1. Documentation
   - API documentation
   - Integration guides
   - Best practices
   - Error codes

2. Technical Support
   - 24/7 support
   - Integration assistance
   - Issue resolution
   - System updates

3. Market Making Support
   - MMP configuration
   - Performance optimization
   - Risk management
   - Market analysis
