# Error Codes

## Response Format

All API responses follow this format:

```json
{
    "code": "0",
    "msg": "",
    "data": []
}
```

- `code`: Error code, "0" means success
- `msg`: Error message
- `data`: Response data

## Common Error Codes

### System Level (0-99)

| Code | Message | Description |
|------|---------|-------------|
| 0 | Success | The request was successful |
| 50000 | System error | Internal system error |
| 50001 | System is under maintenance | System maintenance, try again later |
| 50002 | Service unavailable | Service temporarily unavailable |
| 50004 | IP is prohibited | IP address is restricted |
| 50005 | Request frequency too high | Rate limit exceeded |

### Authentication (100-199)

| Code | Message | Description |
|------|---------|-------------|
| 50111 | Invalid API Key | The API key is invalid |
| 50112 | Invalid signature | Request signature verification failed |
| 50113 | Invalid timestamp | Timestamp is outside acceptable range |
| 50114 | Invalid passphrase | API key passphrase is incorrect |
| 50115 | IP not allowed | IP address not in whitelist |

### Request Format (200-299)

| Code | Message | Description |
|------|---------|-------------|
| 51000 | Parameter error | Invalid parameter provided |
| 51001 | Invalid symbol | Trading pair does not exist |
| 51002 | Invalid currency | Currency does not exist |
| 51003 | Invalid amount | Amount is invalid |
| 51004 | Invalid type | Order type is invalid |

### Trading (300-399)

| Code | Message | Description |
|------|---------|-------------|
| 51100 | Balance insufficient | Insufficient account balance |
| 51101 | Order does not exist | The specified order cannot be found |
| 51102 | Order already canceled | Order was already canceled |
| 51103 | Order is completed | Order is already completed |
| 51104 | Order amount too small | Order amount below minimum |

### Account (400-499)

| Code | Message | Description |
|------|---------|-------------|
| 51200 | Account suspended | Account is suspended |
| 51201 | Account not exist | Account does not exist |
| 51202 | Account in liquidation | Account is being liquidated |
| 51203 | Account in transfer | Account transfer in progress |
| 51204 | Account restricted | Account has trading restrictions |

### WebSocket (500-599)

| Code | Message | Description |
|------|---------|-------------|
| 51300 | Invalid WS request | WebSocket request format invalid |
| 51301 | Invalid WS channel | Channel does not exist |
| 51302 | WS not connected | WebSocket connection lost |
| 51303 | WS auth failed | WebSocket authentication failed |
| 51304 | WS subscription failed | Failed to subscribe to channel |

## Error Handling Best Practices

1. Always check the response code
2. Implement retry logic for system errors
3. Handle rate limits appropriately
4. Log errors for debugging
5. Provide meaningful error messages to users

## Common Error Scenarios

### Rate Limiting
```json
{
    "code": "50005",
    "msg": "Request frequency too high",
    "data": []
}
```
Solution: Implement rate limiting on your side

### Invalid Signature
```json
{
    "code": "50112",
    "msg": "Invalid signature",
    "data": []
}
```
Solution: Check signature generation process

### Insufficient Balance
```json
{
    "code": "51100",
    "msg": "Balance insufficient",
    "data": []
}
```
Solution: Ensure sufficient funds before trading

## HTTP Status Codes

| Status Code | Description |
|-------------|-------------|
| 200 | Success |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 429 | Too Many Requests |
| 500 | Internal Server Error |
| 503 | Service Unavailable |
