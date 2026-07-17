# Getting Started

## Installation

### From source

```bash
git clone <repo-url>
cd blockmango
pip install -e .
```

### Dependencies

- Python 3.8+
- `httpx>=0.24.0`
- `cryptography` (for RSA password encryption)

```bash
pip install httpx cryptography
```

## Prerequisites

You need two things to use the API:

1. **User ID** — Your numeric user ID (e.g., `45890736`)
2. **Access Token** — A long-lived session token from login

These can be obtained by:
- Capturing traffic with mitmproxy while logging in
- Using the debug scripts in the repository

## Quick Start

```python
from blockmango import BlockmanGOClient

# Initialize the client
client = BlockmanGOClient(
    user_id=45890736,
    access_token="your_access_token_here"
)

# Get your user info
user = client.get_user_info()
print(f"Logged in as: {user.nickName}")
print(f"User ID: {user.userId}")
print(f"VIP Level: {user.vipLevel}")

# Always close when done
client.close()
```

## Context Manager

The client supports context managers for automatic cleanup:

```python
with BlockmanGOClient(user_id=123, access_token="token") as client:
    user = client.get_user_info()
    print(user.nickName)
# Connection automatically closed
```

## Device ID

An optional `user_device_id` parameter can be provided. If not given, one is auto-generated from your user ID:

```python
client = BlockmanGOClient(
    user_id=45890736,
    access_token="token",
    user_device_id="custom_device_hash"
)
```

The device ID is used in the `x-sign` header for request signing.

## Default Headers

The client automatically sends these headers with every request:

| Header | Value |
|--------|-------|
| `packagename` | `blockymods` |
| `packagenamefull` | `com.sandboxol.blockymods` |
| `appversion` | `5663` |
| `appversionname` | `3.20.3` |
| `androidversion` | `36` |
| `channel` | `sandbox` |
| `region` | `sandbox` |
| `env` | `prd` |
| `clienttype` | `client` |
| `userlanguage` | `en_US` |

## Error Handling

```python
from blockmango.exceptions import (
    APIError,
    AuthenticationError,
    RateLimitError,
    NetworkError,
    TokenExpiredError
)

try:
    user = client.get_user_info()
except APIError as e:
    print(f"API Error [{e.code}]: {e.message}")
except AuthenticationError as e:
    print("Authentication failed — check your tokens")
except RateLimitError:
    print("Rate limited — slow down")
except TokenExpiredError:
    print("Token expired — re-authenticate")
except NetworkError as e:
    print(f"Network error: {e}")
```

## Next Steps

- [Authentication](authentication.md) — Learn about request signing
- [API Reference](api-reference.md) — Browse all endpoints
- [Examples](examples.md) — Real-world code samples
