# Spread Trading

## Overview

Spread trading на OKX позволяет трейдерам одновременно торговать разницей цен между связанными инструментами. Эта функция особенно полезна для арбитражных стратегий и управления рисками.

## Features

1. Order Management
   - Place spread orders
   - Cancel orders
   - Modify existing orders
   - View order status

2. Market Data
   - Real-time spreads
   - Order book data
   - Trading history
   - Candlestick data

3. Trade History
   - Active orders
   - Historical orders
   - Trade details
   - Position tracking

## API Endpoints

### Order Management

#### Place Order
```http
POST /api/v5/spread/order
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Trade
- Description: Place a new spread trading order

#### Cancel Order
```http
POST /api/v5/spread/cancel-order
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Trade
- Description: Cancel an existing spread order

#### Cancel All Orders
```http
POST /api/v5/spread/cancel-all-orders
```
- Rate Limit: 10 requests per 2 seconds
- Permission: Trade
- Description: Cancel all active spread orders

#### Amend Order
```http
POST /api/v5/spread/amend-order
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Trade
- Description: Modify an existing spread order

### Order Information

#### Get Order Details
```http
GET /api/v5/spread/order
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Read
- Description: Get details of a specific spread order

#### Get Active Orders
```http
GET /api/v5/spread/orders-pending
```
- Rate Limit: 10 requests per 2 seconds
- Permission: Read
- Description: Get list of active spread orders

#### Get Order History (21 Days)
```http
GET /api/v5/spread/orders-history
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Read
- Description: Get spread order history for last 21 days

#### Get Order History (3 Months)
```http
GET /api/v5/spread/orders-history-archive
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Read
- Description: Get spread order history for last 3 months

### Market Data

#### Get Public Spreads
```http
GET /api/v5/spread/spreads
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Read
- Description: Get public spread information

#### Get Order Book
```http
GET /api/v5/spread/order-book
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Read
- Description: Get spread order book data

#### Get Ticker
```http
GET /api/v5/spread/ticker
```
- Rate Limit: 20 requests per 2 seconds
- Permission: Read
- Description: Get spread ticker information

#### Get Candlesticks
```http
GET /api/v5/spread/candles
```
- Rate Limit: 40 requests per 2 seconds
- Permission: Read
- Description: Get spread candlestick data

## Implementation Example

```python
import okx.SpreadTrading as SpreadTrading
import json
from typing import Dict, List, Optional

class OKXSpreadTrading:
    def __init__(self, api_key: str, secret_key: str, passphrase: str, is_sandbox: bool = False):
        """
        Initialize Spread Trading client
        
        Args:
            api_key: API key
            secret_key: Secret key
            passphrase: API passphrase
            is_sandbox: Whether to use sandbox environment
        """
        self.client = SpreadTrading.SpreadTradingAPI(
            api_key, secret_key, passphrase, is_sandbox
        )
        
    def place_order(self, 
                    sprd: str,
                    side: str,
                    sz: str,
                    px: Optional[str] = None,
                    order_type: str = "limit") -> Dict:
        """
        Place a spread trading order
        
        Args:
            sprd: Spread trading instrument
            side: Order side (buy or sell)
            sz: Order size
            px: Order price (required for limit orders)
            order_type: Order type (limit or market)
            
        Returns:
            Dict: Order placement result
        """
        try:
            result = self.client.place_order(
                sprd=sprd,
                side=side,
                ordType=order_type,
                sz=sz,
                px=px
            )
            return result
        except Exception as e:
            print(f"Error placing order: {str(e)}")
            return None
            
    def cancel_order(self, ordId: str, sprd: str) -> Dict:
        """
        Cancel a spread order
        
        Args:
            ordId: Order ID to cancel
            sprd: Spread trading instrument
            
        Returns:
            Dict: Cancellation result
        """
        try:
            result = self.client.cancel_order(
                ordId=ordId,
                sprd=sprd
            )
            return result
        except Exception as e:
            print(f"Error cancelling order: {str(e)}")
            return None
            
    def amend_order(self, 
                    ordId: str,
                    sprd: str,
                    new_sz: Optional[str] = None,
                    new_px: Optional[str] = None) -> Dict:
        """
        Modify an existing spread order
        
        Args:
            ordId: Order ID to modify
            sprd: Spread trading instrument
            new_sz: New order size
            new_px: New order price
            
        Returns:
            Dict: Order modification result
        """
        try:
            result = self.client.amend_order(
                ordId=ordId,
                sprd=sprd,
                newSz=new_sz,
                newPx=new_px
            )
            return result
        except Exception as e:
            print(f"Error amending order: {str(e)}")
            return None
            
    def get_order_book(self, sprd: str, sz: Optional[str] = None) -> Dict:
        """
        Get spread order book data
        
        Args:
            sprd: Spread trading instrument
            sz: Order book depth
            
        Returns:
            Dict: Order book data
        """
        try:
            result = self.client.get_order_book(
                sprd=sprd,
                sz=sz
            )
            return result
        except Exception as e:
            print(f"Error getting order book: {str(e)}")
            return None

# Usage Example
def main():
    spread_trading = OKXSpreadTrading(
        api_key='YOUR_API_KEY',
        secret_key='YOUR_SECRET_KEY',
        passphrase='YOUR_PASSPHRASE'
    )
    
    # Place a limit order
    order = spread_trading.place_order(
        sprd="BTC-USD_BTC-USDT",
        side="buy",
        sz="1",
        px="100",
        order_type="limit"
    )
    
    if order and order.get('code') == '0':
        order_id = order['data']['ordId']
        
        # Get order book data
        book = spread_trading.get_order_book(
            sprd="BTC-USD_BTC-USDT",
            sz="20"
        )
        print(f"Order book: {book}")
        
        # Modify order
        modified = spread_trading.amend_order(
            ordId=order_id,
            sprd="BTC-USD_BTC-USDT",
            new_px="101"
        )
        print(f"Order modification result: {modified}")
        
        # Cancel order
        cancelled = spread_trading.cancel_order(
            ordId=order_id,
            sprd="BTC-USD_BTC-USDT"
        )
        print(f"Order cancellation result: {cancelled}")

if __name__ == "__main__":
    main()
```

## Best Practices

1. Order Management
   - Validate order parameters
   - Monitor order status
   - Implement error handling
   - Track order history

2. Risk Management
   - Set position limits
   - Monitor exposure
   - Track spread changes
   - Implement stop-loss

3. Market Data
   - Cache market data
   - Monitor data quality
   - Handle disconnections
   - Track latency

4. Performance
   - Optimize API usage
   - Respect rate limits
   - Batch operations
   - Monitor system health

## Error Handling

Common errors to handle:

1. Invalid Order
```json
{
    "code": "50001",
    "msg": "Invalid order parameters"
}
```

2. Insufficient Balance
```json
{
    "code": "50002",
    "msg": "Insufficient trading balance"
}
```

3. Rate Limit Exceeded
```json
{
    "code": "50003",
    "msg": "Too many requests"
}
```

4. Market Closed
```json
{
    "code": "50004",
    "msg": "Market is closed"
}
```

## Rate Limits

| Endpoint Category | Rate Limit | Rule |
|------------------|------------|------|
| Order Placement | 20 req/2s | User ID |
| Order Cancellation | 20 req/2s | User ID |
| Order Modification | 20 req/2s | User ID |
| Market Data | 20-40 req/2s | IP |
| Order History | 20 req/2s | User ID |
| Active Orders | 10 req/2s | User ID |

## Security Considerations

1. API Security
   - Secure key storage
   - Request signing
   - IP restrictions
   - Access monitoring

2. Order Validation
   - Price validation
   - Size validation
   - Balance checks
   - Position limits

3. Risk Controls
   - Maximum order size
   - Price deviation checks
   - Exposure limits
   - Loss limits

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

3. Trading Support
   - Strategy consultation
   - Risk management
   - Market analysis
   - Performance optimization
