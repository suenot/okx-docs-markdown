# Private WebSocket API

## Overview

The private WebSocket API provides real-time updates for:
- Account Updates
- Position Updates
- Order Updates
- Balance Updates
- Algorithmic Order Updates

## Connection

### WebSocket Endpoint
```
wss://ws.okx.com:8443/ws/v5/private
```

### Authentication
Private channels require authentication. Login message format:
```json
{
    "op": "login",
    "args": [{
        "apiKey": "YOUR_API_KEY",
        "passphrase": "YOUR_PASSPHRASE",
        "timestamp": "1538054050",
        "sign": "YOUR_SIGNATURE"
    }]
}
```

## Channels

### Account Channel
```json
{
    "op": "subscribe",
    "args": [{
        "channel": "account",
        "ccy": "BTC"
    }]
}
```

#### Push Data Example
```json
{
    "arg": {
        "channel": "account",
        "ccy": "BTC"
    },
    "data": [{
        "uTime": "1597026383085",
        "totalEq": "41624.92", 
        "isoEq": "3624.92",
        "adjEq": "41624.92",
        "ordFroz": "0",
        "imr": "4162.49",
        "mmr": "3329.99",
        "notionalUsd": "",
        "mgnRatio": "41.624924",
        "details": [{
            "ccy": "BTC",
            "eq": "1.23",
            "cashBal": "1.23",
            "uTime": "1597026383085",
            "isoEq": "0",
            "availEq": "1.23",
            "disEq": "1.23",
            "availBal": "1.23",
            "frozenBal": "0",
            "ordFrozen": "0",
            "liab": "0",
            "upl": "0",
            "uplLiab": "0",
            "crossLiab": "0",
            "isoLiab": "0",
            "mgnRatio": "",
            "interest": "0",
            "twap": "0",
            "maxLoan": "",
            "eqUsd": "41624.92",
            "notionalLever": "0"
        }]
    }]
}
```

### Positions Channel
```json
{
    "op": "subscribe",
    "args": [{
        "channel": "positions",
        "instType": "FUTURES"
    }]
}
```

#### Push Data Example
```json
{
    "arg": {
        "channel": "positions",
        "instType": "FUTURES"
    },
    "data": [{
        "adl": "1",
        "availPos": "1",
        "avgPx": "2566.31",
        "cTime": "1597026383085",
        "ccy": "USDT",
        "deltaBS": "",
        "deltaPA": "",
        "gammaBS": "",
        "gammaPA": "",
        "imr": "",
        "instId": "BTC-USDT-SWAP",
        "instType": "SWAP",
        "interest": "0",
        "last": "2566.22",
        "lever": "10",
        "liab": "",
        "liabCcy": "",
        "liqPx": "2261.2533675",
        "markPx": "2566.22",
        "margin": "0.0003896645377994",
        "mgnMode": "isolated",
        "mgnRatio": "170.99370509",
        "mmr": "0.0000779329075599",
        "notionalUsd": "2566.22",
        "optVal": "",
        "pos": "1",
        "posCcy": "",
        "posId": "1234567",
        "posSide": "long",
        "thetaBS": "",
        "thetaPA": "",
        "tradeId": "123",
        "uTime": "1597026383085",
        "upl": "0.00009786",
        "uplRatio": "0.0047",
        "vegaBS": "",
        "vegaPA": ""
    }]
}
```

### Orders Channel
```json
{
    "op": "subscribe",
    "args": [{
        "channel": "orders",
        "instType": "SPOT"
    }]
}
```

#### Push Data Example
```json
{
    "arg": {
        "channel": "orders",
        "instType": "SPOT"
    },
    "data": [{
        "instType": "SPOT",
        "instId": "BTC-USDT",
        "ccy": "",
        "ordId": "312269865356374016",
        "clOrdId": "b15",
        "tag": "",
        "px": "8200",
        "sz": "0.001",
        "pnl": "0",
        "ordType": "limit",
        "side": "buy",
        "posSide": "long",
        "tdMode": "cash",
        "accFillSz": "0",
        "fillPx": "0",
        "tradeId": "0",
        "fillSz": "0",
        "fillTime": "0",
        "state": "live",
        "avgPx": "0",
        "lever": "0",
        "tpTriggerPx": "",
        "tpOrdPx": "",
        "slTriggerPx": "",
        "slOrdPx": "",
        "feeCcy": "",
        "fee": "",
        "rebateCcy": "",
        "rebate": "",
        "tgtCcy": "",
        "category": "",
        "uTime": "1597026383085",
        "cTime": "1597026383085"
    }]
}
```

## Best Practices

1. Connection Management
   - Implement heartbeat mechanism
   - Handle reconnection with exponential backoff
   - Monitor connection state

2. Data Processing
   - Process messages asynchronously
   - Maintain local state
   - Handle message ordering

3. Error Handling
   - Handle connection errors
   - Process error messages
   - Implement logging

## Example Implementation

```python
import websockets
import json
import asyncio
import time
import hmac
import base64
import datetime

class OKXWebSocketClient:
    def __init__(self, api_key, secret_key, passphrase):
        self.api_key = api_key
        self.secret_key = secret_key
        self.passphrase = passphrase
        self.ws = None
        self.connected = False
        
    def get_timestamp(self):
        now = datetime.datetime.utcnow()
        t = now.isoformat("T", "milliseconds")
        return t + "Z"
        
    def sign(self, timestamp, method, request_path, body):
        message = timestamp + method + request_path + body
        mac = hmac.new(
            bytes(self.secret_key, encoding='utf8'),
            bytes(message, encoding='utf-8'),
            digestmod='sha256'
        )
        d = mac.digest()
        return base64.b64encode(d).decode()
        
    async def connect(self):
        try:
            self.ws = await websockets.connect('wss://ws.okx.com:8443/ws/v5/private')
            timestamp = self.get_timestamp()
            sign = self.sign(timestamp, 'GET', '/users/self/verify', '')
            
            login_data = {
                "op": "login",
                "args": [{
                    "apiKey": self.api_key,
                    "passphrase": self.passphrase,
                    "timestamp": timestamp,
                    "sign": sign
                }]
            }
            
            await self.ws.send(json.dumps(login_data))
            response = await self.ws.recv()
            self.connected = True
            print(f"Connected: {response}")
            
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
            
    async def receive_messages(self):
        while True:
            try:
                message = await self.ws.recv()
                data = json.loads(message)
                await self.process_message(data)
                
            except Exception as e:
                print(f"Error receiving message: {str(e)}")
                if not self.connected:
                    await self.connect()
                    
    async def process_message(self, message):
        # Implement your message processing logic here
        print(f"Received message: {json.dumps(message, indent=2)}")
        
    async def heartbeat(self):
        while True:
            if self.connected:
                try:
                    await self.ws.send('ping')
                    await asyncio.sleep(15)
                except Exception as e:
                    print(f"Heartbeat failed: {str(e)}")
                    self.connected = False
                    
    async def start(self):
        await self.connect()
        channels = [
            {"channel": "account"},
            {"channel": "positions", "instType": "FUTURES"},
            {"channel": "orders", "instType": "SPOT"}
        ]
        await self.subscribe(channels)
        
        # Start heartbeat and message receiving tasks
        await asyncio.gather(
            self.heartbeat(),
            self.receive_messages()
        )

# Usage Example
async def main():
    client = OKXWebSocketClient(
        api_key='YOUR_API_KEY',
        secret_key='YOUR_SECRET_KEY',
        passphrase='YOUR_PASSPHRASE'
    )
    await client.start()

if __name__ == "__main__":
    asyncio.run(main())
```
