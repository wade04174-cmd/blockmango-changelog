# BlockmanGO API Documentation

Public documentation for the BlockmanGO (BlockyMods) REST API, reverse-engineered from mitmproxy traffic captures (5,000+ flows analyzed).

> **Disclaimer**: This documentation is for educational and research purposes only. This project is not affiliated with BlockmanGO or SandboxOL. Use at your own risk.

---

## Table of Contents

- [Getting Started](getting-started.md) — Installation, setup, and quick start
- [Authentication](authentication.md) — API keys, request signing, tokens
- [API Reference](api-reference.md) — Complete endpoint documentation (100+ endpoints)
- [Data Models](models.md) — All request/response data structures
- [Game Reference](games.md) — Game IDs, engine versions, server infrastructure
- [Security Notes](security.md) — Documented vulnerabilities and security findings
- [Examples](examples.md) — Code samples for common operations

---

## Overview

BlockmanGO (package: `com.sandboxol.blockymods`) is a sandbox multiplayer game platform. The API is a REST-based service hosted on `gw.sandboxol.com` with:

- **100+ endpoints** across 12+ API domains
- **MD5-based request signing** (`x-sign` header)
- **JWT tokens** for game server authentication (RS256)
- **60+ games** including Bed Wars, Sky Block, Egg War, and user-generated content

## Base URL

```
https://gw.sandboxol.com
```

## Response Format

All API responses follow a standard envelope:

```json
{
  "code": 1,
  "message": "SUCCESS",
  "data": {},
  "other": null
}
```

`code: 1` indicates success. Any other code is an error.

## Quick Example

```python
from blockmango import BlockmanGOClient

client = BlockmanGOClient(
    user_id=45890736,
    access_token="your_token_here"
)

# Get user profile
user = client.get_user_info()
print(f"Welcome, {user.nickName}!")

# List game rooms
rooms = client.get_game_rooms(mode="PVP")
print(f"Found {len(rooms)} PVP rooms")

# Get currency
wealth = client.get_wealth()
print(f"Diamonds: {wealth.diamonds}")

client.close()
```

## API Domains

| Domain | Purpose | Protocol |
|--------|---------|----------|
| `gw.sandboxol.com` | Main API Gateway | HTTPS |
| `di.sandboxol.com` | Analytics/Event Tracking | HTTP |
| `staticgs.sandboxol.com` | CDN Static Assets | HTTPS |
| `static.sandboxol.com` | CDN Static Assets | HTTPS |
| Game Servers | Direct game communication | HTTP |

## Games Supported

| Game | ID | Engine |
|------|----|--------|
| Bed Wars | g1008 | v1 (10114) |
| Sky Block | g1048 | v1 (10114) |
| Egg War | g1018 | v1 (10114) |
| Hero Tycoon 2 | g1025 | v1 |
| Free City RP | g2052 | v2 (20084) |
| Social Hall | g2066 | v2 (20084) |
| Treasure Hunter | g1015 | v1 (10114) |
| Murder Mystery | g1009 | v1 |
| TNT Tag | g1026 | v1 |
| Build Battle | g1023 | v1 |

Plus 50+ more games. See [Game Reference](games.md) for the full list.
