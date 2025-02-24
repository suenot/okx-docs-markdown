# Rate Limits

OKX implements rate limits to protect our API infrastructure and ensure fair usage. Understanding and respecting these limits is crucial for maintaining stable API access.

## Rate Limit Rules

Rate limits are applied based on:
1. API Key
2. IP Address
3. User ID
4. Endpoint specific limits

## REST API Rate Limits

### Public Endpoints

| Endpoint Type | Rate Limit | Time Window |
|---------------|------------|-------------|
| Market Data | 20 requests | Per 2 seconds |
| Trading Data | 10 requests | Per 2 seconds |
| Index Data | 20 requests | Per 2 seconds |

### Private Endpoints

| Endpoint Type | Rate Limit | Time Window |
|---------------|------------|-------------|
| Order Placement | 60 requests | Per 2 seconds |
| Order Cancellation | 60 requests | Per 2 seconds |
| Order Query | 20 requests | Per 2 seconds |
| Account Info | 10 requests | Per 2 seconds |

### Special Endpoints

Some endpoints have specific rate limits:

| Endpoint | Rate Limit | Time Window |
|----------|------------|-------------|
| Place Multiple Orders | 300 orders | Per 2 seconds |
| Cancel Multiple Orders | 300 orders | Per 2 seconds |
| Batch Transfer | 10 requests | Per 2 seconds |

## WebSocket Rate Limits

### Connection Limits
- Maximum 5 concurrent WebSocket connections per IP
- Maximum 3 authenticated connections per account

### Subscription Limits
- Maximum 240 subscriptions per connection
- Maximum 480 subscriptions per account

### Message Limits
- Maximum 240 messages per second per connection
- Maximum 480 messages per second per account

## Rate Limit Headers

Response headers include rate limit information:

```
OK-RateLimit-Remaining: Number of requests remaining
OK-RateLimit-Reset: Time until limit resets (UTC timestamp)
OK-RateLimit-Limit: Total requests allowed
```

## Rate Limit Error Response

When rate limit is exceeded:
```json
{
    "code": "50005",
    "msg": "Request frequency too high",
    "data": []
}
```

## Best Practices

### Handling Rate Limits

1. Monitor rate limit headers
```python
def check_rate_limit(response):
    remaining = response.headers.get('OK-RateLimit-Remaining')
    reset_time = response.headers.get('OK-RateLimit-Reset')
    if int(remaining) < 10:
        wait_time = int(reset_time) - int(time.time())
        time.sleep(wait_time)
```

2. Implement backoff strategy
```python
def exponential_backoff(retry_count):
    wait_time = min(300, (2 ** retry_count))  # Max 300 seconds
    time.sleep(wait_time)
```

### Optimization Tips

1. Use WebSocket for real-time data
2. Batch requests when possible
3. Cache frequently accessed data
4. Implement request queuing
5. Use connection pooling

## Monitoring and Alerts

1. Track rate limit usage
2. Set up alerts for approaching limits
3. Monitor request latency
4. Log rate limit errors
5. Implement circuit breakers

## VIP User Limits

Higher rate limits are available for VIP users:

| VIP Level | Rate Multiplier |
|-----------|----------------|
| VIP 1 | 2x |
| VIP 2 | 3x |
| VIP 3 | 4x |
| VIP 4 | 5x |
| VIP 5 | 6x |

Contact OKX support for VIP level upgrades.
