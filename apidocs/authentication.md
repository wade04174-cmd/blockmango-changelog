# Authentication

## Overview

The BlockmanGO API uses a custom request signing scheme. Every request to `gw.sandboxol.com` requires signed headers.

## Required Headers

| Header | Description | Example |
|--------|-------------|---------|
| `userid` | Numeric user ID | `45890736` |
| `access-token` | Session token | `eyJhbGciOi...` |
| `userdeviceid` | Device fingerprint hash | `a1b2c3d4e5f6g7h8` |
| `x-apikey` | Static API key | `6aDtpIdzQdgGwrpP6HzuPA` |
| `x-nonce` | Random 32-char hex string | `a1b2c3d4e5f6...` |
| `x-time` | Unix timestamp | `1700000000` |
| `x-sign` | MD5 signature | `md5hash...` |
| `x-urlpath` | Request path | `/user/api/v1/user/info` |

## API Key Pairs

There are 3 known static key pairs:

| API Key | Secret Key |
|---------|------------|
| `6aDtpIdzQdgGwrpP6HzuPA` | `9EuDKGtoWAOWoQH1cRng-d5ihNN60hkGLaRiaZTk-6s` |
| `h0jCHbhVd9Fpkx-FGkxeRw` | `lOTB7DNdMMpdyUO-psJ5b2ivYGmU5RAy6j6bkpoMYcs` |
| `dM9XM3sxjfVI6AC77GS9rw` | `6aNQVhd8pP-Gg7_xM2PTEp92G-77tzHGnPKrwslxmAg` |

The client randomly selects one pair per request.

## Signature Algorithm

The `x-sign` header is generated as follows:

### Step 1: Build the sign input string

```
sign_input = apikey + path + nonce + timestamp + sorted_params + body + secret_key
```

Where:
- `apikey` — The selected API key
- `path` — The request path (e.g., `/user/api/v1/user/info`)
- `nonce` — Random 32-character hex string
- `timestamp` — Current Unix timestamp as string
- `sorted_params` — URL query parameters sorted by key, joined with `&` (e.g., `key1=val1&key2=val2`)
- `body` — JSON request body as compact string (empty for GET requests)
- `secret_key` — The corresponding secret key

### Step 2: MD5 hash

```python
first_hash = md5(sign_input.encode()).hexdigest()
```

### Step 3: Device binding (optional)

If a device ID is present:

```python
sign = md5((first_hash + user_device_id).encode()).hexdigest()
```

Otherwise, `sign = first_hash`.

## Python Implementation

```python
import hashlib
import random
import time
import json

KEY_PAIRS = [
    ("6aDtpIdzQdgGwrpP6HzuPA", "9EuDKGtoWAOWoQH1cRng-d5ihNN60hkGLaRiaZTk-6s"),
    ("h0jCHbhVd9Fpkx-FGkxeRw", "lOTB7DNdMMpdyUO-psJ5b2ivYGmU5RAy6j6bkpoMYcs"),
    ("dM9XM3sxjfVI6AC77GS9rw", "6aNQVhd8pP-Gg7_xM2PTEp92G-77tzHGnPKrwslxmAg"),
]

def generate_sign(path, method="GET", body="", params=None, device_id=None):
    ak, sk = random.choice(KEY_PAIRS)
    nonce = ''.join(random.choices('0123456789abcdef', k=32))
    timestamp = str(int(time.time()))

    params = params or {}
    sorted_params = '&'.join(f"{k}={params[k]}" for k in sorted(params))
    sign_input = f"{ak}{path}{nonce}{timestamp}{sorted_params}{body}{sk}"

    first = hashlib.md5(sign_input.encode()).hexdigest()

    if device_id:
        sign = hashlib.md5((first + device_id).encode()).hexdigest()
    else:
        sign = first

    return {
        "x-apikey": ak,
        "x-nonce": nonce,
        "x-time": timestamp,
        "x-sign": sign,
        "x-urlpath": path,
    }
```

## JWT Game Tokens

Game sessions use RS256-signed JWT tokens obtained via:

```
GET /game/api/v3/game/auth?typeId={gameId}&targetId={userId}&gameVersion={engineVer}
```

JWT payload structure:
```json
{
  "uid": 45890736,
  "gtype": "g1015",
  "launchTraceId": "...",
  "iss": "45890736",
  "exp": 1700003600
}
```

## Account Registration

Account passwords are encrypted with RSA before transmission:

- **Algorithm**: RSA/ECB/PKCS1Padding
- **Key Size**: 1024-bit
- **Chunk Size**: 117 bytes
- **Encoding**: Base64

The RSA public key:
```
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCLzlsA+3wXCAph80r/xs1bWhVrsJSOQmSBTA0GaBpVIzXqFBaibDmYA3WJDM9rcQ7KpYSyrJ02iFlsN43RnizrHfS+xPtdwuxBQ2Clow5cYPZucqQYL9HIlbBLoighH2eGQqGlVadL7r384iKTz9mmckSUa8hhJzS+WwUAqVO3DwIDAQAB
```

## Authentication Without Signing

Some endpoints (e.g., tourist login) do not require `x-sign` headers. These are typically unauthenticated endpoints.

## Security Notes

- API keys are static and never rotate
- MD5 is cryptographically broken
- User IDs are plain integers in headers
- No certificate pinning on the API
- Analytics are sent over HTTP (not HTTPS)
