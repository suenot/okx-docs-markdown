# Public WebSocket API

## Overview

The public WebSocket API provides real-time market data:
- Tickers
- Order Book
- Trades
- Candlesticks
- Index Tickers
- Mark Price
- Open Interest
- Funding Rate

## Connection

### WebSocket Endpoint
```
wss://ws.okx.com:8443/ws/v5/public
```

### Connection Process
1. Connect to WebSocket endpoint
2. Subscribe to desired channels
3. Handle incoming messages
4. Implement heartbeat mechanism

## Channels

### Tickers Channel
```json
{
    "op": "subscribe",
    "args": [{
        "channel": "tickers",
        "instId": "BTC-USDT"
    }]
}
```

#### Push Data Example
```json
{
    "arg": {
        "channel": "tickers",
        "instId": "BTC-USDT"
    },
    "data": [{
        "instType": "SPOT",
        "instId": "BTC-USDT",
        "last": "43000.1",
        "lastSz": "0.1",
        "askPx": "43000.2",
        "askSz": "0.1",
        "bidPx": "43000.0",
        "bidSz": "0.1",
        "open24h": "42000.1",
        "high24h": "43500.1",
        "low24h": "41500.1",
        "volCcy24h": "1000.1",
        "vol24h": "1000.1",
        "ts": "1597026383085",
        "sodUtc0": "40000.1",
        "sodUtc8": "41000.1"
    }]
}
```

### Order Book Channel
```json
{
    "op": "subscribe",
    "args": [{
        "channel": "books",
        "instId": "BTC-USDT"
    }]
}
```

#### Push Data Example
```json
{
    "arg": {
        "channel": "books",
        "instId": "BTC-USDT"
    },
    "data": [{
        "asks": [
            ["43000.1", "0.1", "0", "1"],
            ["43000.2", "0.2", "0", "1"]
        ],
        "bids": [
            ["42999.9", "0.1", "0", "1"],
            ["42999.8", "0.2", "0", "1"]
        ],
        "ts": "1597026383085",
        "checksum": 1234567
    }]
}
```

### Trades Channel
```json
{
    "op": "subscribe",
    "args": [{
        "channel": "trades",
        "instId": "BTC-USDT"
    }]
}
```

#### Push Data Example
```json
{
    "arg": {
        "channel": "trades",
        "instId": "BTC-USDT"
    },
    "data": [{
        "instId": "BTC-USDT",
        "tradeId": "123",
        "px": "43000.1",
        "sz": "0.1",
        "side": "buy",
        "ts": "1597026383085"
    }]
}
```

### Candlesticks Channel
```json
{
    "op": "subscribe",
    "args": [{
        "channel": "candle1m",
        "instId": "BTC-USDT"
    }]
}
```

#### Push Data Example
```json
{
    "arg": {
        "channel": "candle1m",
        "instId": "BTC-USDT"
    },
    "data": [
        ["1597026383085", "8533.02", "8553.74", "8527.17", "8548.26", "45.59", "389354.2"]
    ]
}
```

## Best Practices

1. Connection Management
   - Implement heartbeat mechanism (ping every 30 seconds)
   - Handle reconnection with exponential backoff
   - Monitor connection state

2. Order Book Management
   - Maintain local order book
   - Verify checksum
   - Handle order book updates efficiently

3. Data Processing
   - Process messages asynchronously
   - Implement message queuing
   - Handle message ordering

## Example Implementation

```python
import websockets
import json
import asyncio
import time
from collections import defaultdict

class OKXPublicWebSocketClient:
    def __init__(self):
        self.ws = None
        self.connected = False
        self.order_books = defaultdict(dict)
        
    async def connect(self):
        try:
            self.ws = await websockets.connect('wss://ws.okx.com:8443/ws/v5/public')
            self.connected = True
            print("Connected to OKX WebSocket")
            
        except Exception as e:
            print(f"Connection failed: {str(e)}")
            self.connected = False
            
    async def subscribe(self, channels):
        if not self.connected:
            await self.connect()
            
        try:
            subscribe_data = {
                "op": "subscribe",
                "args": channels
            }
            await self.ws.send(json.dumps(subscribe_data))
            response = await self.ws.recv()
            print(f"Subscription response: {response}")
            
        except Exception as e:
            print(f"Subscription failed: {str(e)}")
            
    def update_order_book(self, inst_id, data):
        """Update local order book"""
        book = self.order_books[inst_id]
        
        # Initialize or update asks and bids
        if 'asks' in data:
            book['asks'] = {float(price): float(size) for price, size, *_ in data['asks']}
        if 'bids' in data:
            book['bids'] = {float(price): float(size) for price, size, *_ in data['bids']}
            
        # Verify checksum if provided
        if 'checksum' in data:
            if not self.verify_checksum(inst_id, data['checksum']):
                print(f"Checksum verification failed for {inst_id}")
                return False
        return True
        
    def verify_checksum(self, inst_id, checksum):
        """Verify order book checksum"""
        book = self.order_books[inst_id]
        # Implement checksum verification logic
        return True
        
    async def process_message(self, message):
        """Process incoming WebSocket message"""
        try:
            data = json.loads(message)
            
            # Handle different channel types
            if 'arg' in data:
                channel = data['arg']['channel']
                inst_id = data['arg']['instId']
                
                if channel == 'books':
                    self.update_order_book(inst_id, data['data'][0])
                elif channel == 'trades':
                    await self.process_trades(data['data'])
                elif channel == 'tickers':
                    await self.process_tickers(data['data'])
                elif channel.startswith('candle'):
                    await self.process_candles(data['data'])
                    
        except Exception as e:
            print(f"Error processing message: {str(e)}")
            
    async def process_trades(self, trades):
        """Process trade data"""
        for trade in trades:
            print(f"Trade: {json.dumps(trade, indent=2)}")
            
    async def process_tickers(self, tickers):
        """Process ticker data"""
        for ticker in tickers:
            print(f"Ticker: {json.dumps(ticker, indent=2)}")
            
    async def process_candles(self, candles):
        """Process candlestick data"""
        for candle in candles:
            print(f"Candle: {json.dumps(candle, indent=2)}")
            
    async def heartbeat(self):
        """Send heartbeat to keep connection alive"""
        while True:
            if self.connected:
                try:
                    await self.ws.send('ping')
                    await asyncio.sleep(15)
                except Exception as e:
                    print(f"Heartbeat failed: {str(e)}")
                    self.connected = False
                    
    async def receive_messages(self):
        """Receive and process messages"""
        while True:
            try:
                message = await self.ws.recv()
                if message == 'pong':
                    continue
                await self.process_message(message)
                
            except Exception as e:
                print(f"Error receiving message: {str(e)}")
                if not self.connected:
                    await self.connect()
                    
    async def start(self):
        """Start WebSocket client"""
        await self.connect()
        
        # Subscribe to channels
        channels = [
            {"channel": "tickers", "instId": "BTC-USDT"},
            {"channel": "books", "instId": "BTC-USDT"},
            {"channel": "trades", "instId": "BTC-USDT"},
            {"channel": "candle1m", "instId": "BTC-USDT"}
        ]
        await self.subscribe(channels)
        
        # Start heartbeat and message receiving tasks
        await asyncio.gather(
            self.heartbeat(),
            self.receive_messages()
        )

# Usage Example
async def main():
    client = OKXPublicWebSocketClient()
    await client.start()

if __name__ == "__main__":
    asyncio.run(main())
```

## Error Handling

Common errors to handle:
1. Connection failures
2. Subscription errors
3. Message parsing errors
4. Checksum verification failures
5. Rate limiting

## Rate Limits

- WebSocket connections per IP: 50
- Message rate per connection: 240 messages per second
- Subscription limit per connection: 240 pairs
