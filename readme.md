# OKX API V5 Documentation

## Getting Started
- [Introduction](./docs/introduction.md)
- [Authentication](./docs/authentication.md)
- [Error Codes](./docs/error-codes.md)
- [Rate Limits](./docs/rate-limits.md)

## API Access
- REST API Base URL: `https://www.okx.com`
- WebSocket Public Channel: `wss://ws.okx.com:8443/ws/v5/public`
- WebSocket Private Channel: `wss://ws.okx.com:8443/ws/v5/private`

## SDK Examples
- [Python SDK](./docs/sdk/python.md)
- [Node.js SDK](./docs/sdk/nodejs.md)
- [Java SDK](./docs/sdk/java.md)
- [Go SDK](./docs/sdk/go.md)
- [PHP SDK](./docs/sdk/php.md)

## Trading Account
### REST API
- [Account Management](./docs/trading/account.md)
  - Get Account Configuration
  - Get Account Balance
  - Get Positions
  - Get Account and Position Risk
  - Get Bills History
- [Trading](./docs/trading/trade.md)
  - Place Order
  - Place Multiple Orders
  - Cancel Order
  - Cancel Multiple Orders
  - Amend Order
  - Get Order Details
  - Get Order List
  - Get Order History
- [Market Data](./docs/trading/market.md)
  - Get Tickers
  - Get Order Book
  - Get Candlesticks
  - Get Trades
  - Get Index Tickers
  - Get Mark Price

### WebSocket API
- [Private Channel](./docs/trading/ws-private.md)
  - Account Channel
  - Positions Channel
  - Orders Channel
  - Balance and Position Channel
- [Public Channel](./docs/trading/ws-public.md)
  - Instruments Channel
  - Tickers Channel
  - Order Books Channel
  - Trades Channel

## Funding Account
- [Deposit](./docs/funding/deposit.md)
  - Get Deposit Address
  - Get Deposit History
- [Withdrawal](./docs/funding/withdrawal.md)
  - Withdraw
  - Cancel Withdrawal
  - Get Withdrawal History
- [Transfer](./docs/funding/transfer.md)
  - Transfer Between Accounts
  - Get Transfer History

## Broker Program
- [DMA Broker](./docs/broker/dma.md)
  - Get Sub-account List
  - Create Sub-account
  - Get Sub-account Trading Balance
  - Sub-account Transfer
- [Fully Disclosed Broker](./docs/broker/fd.md)
  - Create API Key
  - Reset API Key
  - Delete API Key
  - Get Rebate Information

## Additional Features
- [Sub-Account System](./docs/features/subaccount.md)
  - Sub-account Management
  - Sub-account Transfer
  - Sub-account Trading
- [Block Trading](./docs/features/block-trading.md)
  - Create RFQ
  - Create Quote
  - Execute Quote
- [Spread Trading](./docs/features/spread-trading.md)
  - Get Spread Products
  - Place Spread Orders

## Best Practices
- Use WebSocket for real-time market data
- Implement proper error handling
- Follow rate limits guidelines
- Maintain WebSocket connection
- Use appropriate order types
- Implement risk management

## Additional Resources
- [Official Website](https://www.okx.com)
- [API Status](https://www.okx.com/status)
- [Support Center](https://www.okx.com/support-center)
- [API Support](https://www.okx.com/support-center/api-support)