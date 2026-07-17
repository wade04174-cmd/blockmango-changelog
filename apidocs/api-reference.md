# API Reference

Complete documentation of all BlockmanGO REST API endpoints.

**Base URL**: `https://gw.sandboxol.com`

---

## Authentication

> **Every request** to `gw.sandboxol.com` requires signed headers. Requests without valid signatures return `401 Unauthorized`.

### Required Headers

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `userid` | string | Numeric user ID | `45890736` |
| `access-token` | string | JWT session token from login | `eyJhbGciOi...` |
| `userdeviceid` | string | Device fingerprint hash | `E0-8F-4C-8D-E2-1C` |
| `x-apikey` | string | API key (randomly selected from 3 static pairs) | `6aDtpIdzQdgGwrpP6HzuPA` |
| `x-nonce` | string | Random 32-character hex string | `a1b2c3d4e5f67890abcdef1234567890` |
| `x-time` | string | Current Unix timestamp | `1700000000` |
| `x-sign` | string | MD5 signature (see algorithm below) | `md5hash...` |
| `x-urlpath` | string | Request path being called | `/user/api/v1/user/info` |

### Login-Specific Headers

The login endpoint (`POST /user/api/v4/account/login`) uses different headers:

| Header | Type | Description | Example |
|--------|------|-------------|---------|
| `bmg-device-id` | string | Device ID | `E0-8F-4C-8D-E2-1C` |
| `bmg-sign` | string | Device signature | `xEGRkZHZJbCH+27N+LBJxktf57OrioeTvwcAUumrFhQ=` |
| `os` | string | Always `android` | `android` |
| `apptype` | string | Client identifier | `WZRD-TOOL` |
| `x-apikey` | string | API key | |
| `x-nonce` | string | Random 32-char hex | |
| `x-time` | string | Unix timestamp | |
| `x-sign` | string | MD5 signature | |
| `x-urlpath` | string | Always `/user/api/v4/account/login` | |
| `content-type` | string | Always `application/json; charset=UTF-8` | |

### Game Server Headers

Requests to game servers (dispatched after auth) use these additional headers:

| Header | Type | Description |
|--------|------|-------------|
| `x-shahe-uid` | string | User ID |
| `x-shahe-token` | string | Game session token |
| `userId` | string | User ID |
| `access-token` | string | Main access token |

### API Key Pairs

There are **3 static API key pairs** hardcoded in the APK. The client randomly selects one per request. These keys have never rotated across any observed version.

| # | API Key | Secret Key | First Seen |
|---|---------|------------|------------|
| 1 | `6aDtpIdzQdgGwrpP6HzuPA` | `9EuDKGtoWAOWoQH1cRng-d5ihNN60hkGLaRiaZTk-6s` | v2.95.3 (2024) |
| 2 | `h0jCHbhVd9Fpkx-FGkxeRw` | `lOTB7DNdMMpdyUO-psJ5b2ivYGmU5RAy6j6bkpoMYcs` | v2.95.3 (2024) |
| 3 | `dM9XM3sxjfVI6AC77GS9rw` | `6aNQVhd8pP-Gg7_xM2PTEp92G-77tzHGnPKrwslxmAg` | v2.95.3 (2024) |

### Signature Algorithm

#### Step 1: Build the signing input string

```
sign_input = apikey + path + nonce + timestamp + sorted_params + body + secret_key
```

Where:
- `apikey` — The selected API key (e.g., `6aDtpIdzQdgGwrpP6HzuPA`)
- `path` — The request path (e.g., `/user/api/v1/user/info`)
- `nonce` — Random 32-character hex string
- `timestamp` — Current Unix timestamp as string
- `sorted_params` — URL query parameters sorted by key, joined with `&` (e.g., `key1=val1&key2=val2`). Empty string if no params.
- `body` — JSON request body as compact string (empty for GET requests)
- `secret_key` — The corresponding secret key for the selected API key

#### Step 2: MD5 hash

```python
first_hash = hashlib.md5(sign_input.encode()).hexdigest()
```

#### Step 3: Device binding (optional)

If a `userdeviceid` / device ID is present:

```python
sign = hashlib.md5((first_hash + user_device_id).encode()).hexdigest()
```

If no device ID, `sign = first_hash`.

### Full Signing Implementation

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

def generate_sign(path, body="", params=None, device_id=None):
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

### RSA Password Encryption

Account passwords are encrypted with RSA before transmission:

| Property | Value |
|----------|-------|
| Algorithm | RSA/ECB/PKCS1Padding |
| Key Size | 1024-bit (below modern 2048-bit minimum) |
| Chunk Size | 117 bytes |
| Encoding | Base64 |

RSA Public Key:
```
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCLzlsA+3wXCAph80r/xs1bWhVrsJSOQmSBTA0GaBpVIzXqFBaibDmYA3WJDM9rcQ7KpYSyrJ02iFlsN43RnizrHfS+xPtdwuxBQ2Clow5cYPZucqQYL9HIlbBLoighH2eGQqGlVadL7r384iKTz9mmckSUa8hhJzS+WwUAqVO3DwIDAQAB
```

### JWT Game Tokens

Game sessions use RS256-signed JWT tokens obtained via `GET /game/api/v3/game/auth`.

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

### Default Headers

The client also sends these static headers with every request:

| Header | Value | Added |
|--------|-------|-------|
| `packagename` | `blockymods` | v2.95.3 |
| `packagenamefull` | `com.sandboxol.blockymods` | v2.95.3 |
| `appversion` | `5671` (latest) | Updated per version |
| `appversionname` | `3.21.1` (latest) | Updated per version |
| `androidversion` | `36` | v3.21.1 |
| `channel` | `sandbox` | v2.95.3 |
| `region` | `sandbox` | v2.95.3 |
| `env` | `prd` | v2.95.3 |
| `clienttype` | `client` | v2.95.3 |
| `userlanguage` | `en_US` | v2.95.3 |
| `user-agent` | `okhttp/4.12.0` | v3.20.3 |

### Response Format

All endpoints return JSON with this envelope:

```json
{
  "code": 1,
  "message": "SUCCESS",
  "data": {}
}
```

- `code: 1` = success
- `code: 6` = token expired (requires re-auth)
- Other codes indicate specific errors

### Error Codes

| Code | Meaning |
|------|---------|
| 1 | Success |
| 6 | Token expired |
| 2002 | Nickname too long |
| 2003 | Nickname already exists |
| 2004 | Forbidden characters in nickname |
| 3010 | Friend requests full |
| 3011 | Friend requests disabled |
| 3015 | Family not accepting recruits |
| 5006 | Insufficient gold cubes |
| 2014 | Already purchased |
| 7006 | Not in a clan |

---

## Game Endpoints

### Get Game Info

```
GET /game/api/v3/games/{gameId}
```

| Param | Type | Description |
|-------|------|-------------|
| `gameId` | string | Game identifier (e.g., `g1008`) |
| `appVersion` | int | App version code (default: 5671) |

**Returns**: `GameMetadata` — game title, description, banners, online count, engine version

> Added: v2.95.3

---

### Get Game Auth Token

```
GET /game/api/v3/game/auth
```

| Param | Type | Description |
|-------|------|-------------|
| `typeId` | string | Game ID (e.g., `g1015`) |
| `targetId` | int | User ID |
| `gameVersion` | int | Engine version (10114, 20084, 40040) |

**Returns**: `GameAuthResponse` — JWT token, signature, server URL, region, engine type

> Added: v2.95.3

---

### Get Party Auth

```
GET /game/api/v3/party/auth
```

| Param | Type | Description |
|-------|------|-------------|
| `userId` | int | User ID |
| `isNewEngine` | bool | New engine flag |
| `typeId` | string | Type ID (empty for default) |
| `engine4Version` | int | Engine 4 version (40040) |

**Returns**: Party auth data with JWT, party service IPs

> Added: v2.95.3

---

### Get Game Update Tips

```
GET /game/api/v1/games/update/tip/info/app/{gameId}
```

| Param | Type | Description |
|-------|------|-------------|
| `engineVersion` | int | Engine version |
| `isNew` | int | 0 or 1 |

> Added: v2.95.3

---

### Get Game Catalog

```
GET /game/api/v2/game/revision/all/list/by/condition/combine
```

| Param | Type | Description |
|-------|------|-------------|
| `filterTypeId` | int | Game type filter (0 = none) |
| `isFilter` | int | Enable filtering |
| `os` | string | `android` |
| `pageNo` | int | Page number (0-indexed) |
| `pageSize` | int | Items per page |

**Returns**: List of game sections with games

> Added: v2.95.3

---

### Get Games Requiring Update

```
GET /game/api/v1/games/update/list/{userId}
```

| Param | Type | Description |
|-------|------|-------------|
| `oldEngineVersion` | int | Current engine version |
| `newEngineVersion` | int | Target engine version |

> Added: v2.95.3

---

### Update Engine Version

```
PUT /game/api/v1/games/engine
```

| Param | Type | Description |
|-------|------|-------------|
| `engineVersion` | int | Current version |
| `newEngineVersion` | int | New version |
| `thirdEngineVersion` | int | Third-party version |

> Added: v2.95.3

---

### Check Engine Resource Update

```
GET /game/api/v2/games/app-engine/check-update
```

| Param | Type | Description |
|-------|------|-------------|
| `engineVersion` | int | Engine version |
| `commonResVersion` | int | Common resource version |
| `gameResVersion` | int | Game resource version |
| `gameType` | string | Game ID |
| `engineType` | string | `v2` |

> Added: v2.95.3

---

### Get Party-Enabled Games

```
GET /game/api/v1/games/all/open/party
```

| Param | Type | Description |
|-------|------|-------------|
| `appVersion` | int | App version |

**Returns**: List of `PartyGame` objects

> Added: v2.95.3

---

### Get Mining Token Balance

```
GET /game/api/v1/backpack/mining-token/balance/with-transform
```

**Returns**: `MiningTokenBalance` — token amounts, item info, transform data

> Added: v3.21.1 (mining system)

---

### Get User Highlights

```
GET /game/api/v1/user/highlight/data
```

| Param | Type | Description |
|-------|------|-------------|
| `targetId` | int | User ID |

**Returns**: `UserHighlights` — game preferences

> Added: v2.95.3

---

### Claim Game Reward (Turntable)

```
PUT /game/api/v1/game/g{gameId}/turntable
```

| Param | Type | Description |
|-------|------|-------------|
| `gameId` | string | Game ID (e.g., `g1018`) — note: `g` prefix is part of the path |

**Returns**: int — reward amount

> Added: v2.95.3. Used by WZRD-TOOL for ad-free reward claiming.

---

## Game Room Endpoints

### List Game Rooms

```
GET /game/api/v4/gameroom/list
```

| Param | Type | Description |
|-------|------|-------------|
| `mode` | string | Room filter: `""` (all), `PVP`, `RUN`, `CREATE`, `SOCIAL`, `GAME`, `MINING` |
| `evList` | int[] | Engine versions (repeated param: `evList=10114&evList=20084&evList=40040`) |
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

**Returns**: List of `GameRoom` objects

**Room fields**: `roomId`, `name`, `userId`, `nickName`, `mode`, `gameType`, `gameName`, `engineType`, `curUsers`, `maxUsers`, `gameAddr`, `regionId`, `mapAddress`, `needPassword`, `miningLevel`, `totalPoolValue`, `remainingPoolValue`, `miningRewardBonus`, `miningStatus`

> Added: v2.95.3

---

### Get Creative Rooms

```
GET /game/api/v1/gameroom/creative/list
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

> Added: v2.95.3

---

### Get Recently Played Rooms

```
GET /game/api/v1/gameroom/recent/played
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Always 0 |

> Added: v2.95.3

---

### Get Recently Created Rooms

```
GET /game/api/v1/gameroom/recent/created
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Always 0 |

> Added: v2.95.3

---

### Get Random UGC Feed

```
GET /game/api/v1/gameroom/work/random-feed
```

| Param | Type | Description |
|-------|------|-------------|
| `pageSize` | int | Items per page |
| `cursor` | int | Pagination cursor |
| `refresh` | bool | Force refresh |

> Added: v2.95.3

---

### Get Ping Servers

```
GET /game/api/v1/ping/server/list
```

**Returns**: List of `PingServer` — region, clan, URL

> Added: v2.95.3

---

### Get Game Resource

```
GET {game_server}/v1/game-res
```

| Param | Type | Description |
|-------|------|-------------|
| `gameType` | string | Game ID |
| `engineVersion` | int | Engine version |
| `resVersion` | int | Resource version |

**Returns**: `GameResource` — download URL, MD5, size

> Added: v2.95.3

---

### Get Game Map

```
GET {game_server}/v1/game-map
```

| Param | Type | Description |
|-------|------|-------------|
| `mapName` | string | Map name |
| `gameType` | string | Game ID |
| `engineVersion` | int | Engine version |

> Added: v2.95.3

---

### Dispatch Game Server

```
POST {game_server}/v1/dispatch
```

| Param | Type | Description |
|-------|------|-------------|
| Body | JSON | `{"userId": int, "rid": int, "ever": int, "name": string}` |

**Returns**: `GameDispatch` — server assignment, CDN info, `gaddr` (IP:port)

> Added: v2.95.3

---

### Follow Player (Spy)

```
POST {game_server}/v1/follow
```

| Param | Type | Description |
|-------|------|-------------|
| Body | JSON | `{"userId": int, "targetId": int, "rid": int, "ever": int, "name": string}` |

**Returns**: Game and server info for the target player

> Added: v2.95.3. Used by WZRD-TOOL's `/w spy` command to track player locations.

---

## User Endpoints

### Get User Info

```
GET /user/api/v1/user/info
```

**Returns**: `UserProfile` — userId, nickName, picUrl, diamonds, golds, vipLevel, region

> Added: v2.95.3

---

### Get User Details (v3)

```
GET /user/api/v3/user/details/info
```

**Returns**: `UserProfile` — extended user data including registration time

> Added: v3.20.3

---

### Update User Info

```
PUT /user/api/v1/user/info
```

Body: JSON with fields to update (nickName, picUrl, sex, birthday, details, etc.)

> Added: v2.95.3

---

### Change Nickname (v3)

```
PUT /user/api/v3/user/nickName
```

| Param | Type | Description |
|-------|------|-------------|
| `newName` | string | New nickname (max 20 chars via tool, 16 in-game) |
| `oldName` | string | Current nickname |

> Added: v3.20.3. Separate endpoint from general user info update.

---

### Get User Career

```
GET /user/api/v1/users/bg-careers
```

| Param | Type | Description |
|-------|------|-------------|
| `userId` | int | Target user ID |

**Returns**: `UserCareer` — killCount, victoryRate, totalPlayedCount, totalTime, etc.

> Added: v2.95.3

---

### Get Security Settings

```
GET /user/api/v2/users/verify/user/security/settings
```

| Param | Type | Description |
|-------|------|-------------|
| `userId` | int | User ID |

**Returns**: `SecuritySettings` — email, bindEmail, secretQuestionList

> Added: v2.95.3

---

### Get User Shop Info

```
GET /user/api/v1/user/shop/info
```

**Returns**: `UserShopInfo` — diamonds, golds, clothVoucher, gdiamonds

> Added: v2.95.3

---

### Get Avatar Frames

```
GET /user/api/v1/user/avatar/frame
```

**Returns**: List of `AvatarFrame` objects

> Added: v2.95.3

---

### Get VIP Info

```
GET /user/api/v1/vip/users/{userId}
```

**Returns**: `UserVip` — level, exp, maxExp, vipIcon

> Added: v2.95.3

---

### Get VIP Privileges

```
GET /user/api/v1/vip/privilege/all
```

**Returns**: List of privilege definitions

> Added: v2.95.3

---

### Upload File

```
POST /user/api/v1/file
```

| Param | Type | Description |
|-------|------|-------------|
| `fileName` | string | File name |
| `fileType` | string | File type (jpg, png) |

Body: Multipart file upload

**Returns**: URL string

> Added: v2.95.3

---

### Set Language

```
GET /user/api/v1/user/language
```

| Param | Type | Description |
|-------|------|-------------|
| `userId` | int | User ID |
| `language` | string | Language code (e.g., `en`) |

> Added: v2.95.3

---

### Get Device Token

```
GET /user/api/v1/users/device/token
```

**Returns**: RongCloud IM token string

> Added: v2.95.3

---

### Get AB Test Assignments

```
GET /user/api/v1/abtest/getUserTest
```

**Returns**: AB test assignment data

> Added: v2.95.3

---

### Trigger Client Event

```
POST /user/api/v1/client/trigger/event
```

| Param | Type | Description |
|-------|------|-------------|
| `triggerType` | int | Event type (0, 1, etc.) |

> Added: v2.95.3

---

### Get Chat Migration Data

```
GET /user/api/v1/chat/migrate/data
```

**Returns**: RongCloud key rotation data

> Added: v3.21.1

---

### Get Personal Space Effects

```
GET /user/api/v1/personal/space/effects
```

**Returns**: List of available effects

> Added: v3.21.1

---

### Get Account Auth Token (v5)

```
GET /user/api/v5/account/auth-token
```

| Param | Type | Description |
|-------|------|-------------|
| `q` | string | Base64 encoded query |

**Returns**: `AuthTokenResponse` — userId, accessToken, clientIp, country, region

> Added: v3.21.1

---

## Account Endpoints

### Login (v4)

```
POST /user/api/v4/account/login
```

Body:
```json
{
  "account": "username_or_userid",
  "password": "RSA_ENCRYPTED_PASSWORD",
  "tsvAccount": "",
  "tsvPlatform": "",
  "tsvToken": ""
}
```

| Header | Description |
|--------|-------------|
| `bmg-device-id` | Device fingerprint |
| `bmg-sign` | Device signature |
| `apptype` | Client identifier (e.g., `WZRD-TOOL`) |

**Returns**: `{ "code": 1, "data": { "accessToken": "eyJ...", "userId": 12345 } }`

**Notes**:
- Password must be RSA-encrypted (1024-bit PKCS1v1.5, Base64 encoded)
- If `accessToken` is `null`, 2FA is enabled
- Token-based login: use JWT as `password` (must start with `eyJ`), provide numeric user ID as `account`, and include `bmg-device-id`

> Added: v2.95.3

---

### Check Account Available

```
POST /user/api/v1/account/invalid/check
```

| Param | Type | Description |
|-------|------|-------------|
| `account` | string | Desired username |
| `type` | int | Always 1 |

**Returns**: `bool` — true if available

> Added: v2.95.3

---

### Register Account (v4)

```
POST /user/api/v4/account/register
```

| Param | Type | Description |
|-------|------|-------------|
| `account` | string | Username |

Body:
```json
{
  "account": "username",
  "code": "",
  "confirmPassword": "RSA_ENCRYPTED_PASSWORD",
  "email": "",
  "password": "RSA_ENCRYPTED_PASSWORD",
  "userId": 0
}
```

**Returns**: Dict with `userId`, `accessToken`, `account`, `nickName`, `password`

> Added: v2.95.3

---

## Friend Endpoints

### Get Friends (v3)

```
GET /friend/api/v3/friends
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page (max 500) |
| `showBigParty` | int | Always 1 |

**Returns**: List of `FriendUser` objects

> Added: v3.20.3

---

### Get Friend Status (v2)

```
GET /friend/api/v2/friends/status
```

| Param | Type | Description |
|-------|------|-------------|
| `showBigParty` | int | Always 1 |

**Returns**: List of `FriendUser` with `currentTime` for precise offline tracking

> Added: v2.95.3. Used by WZRD-TOOL to show exact logout timestamps.

---

### Get Friend Status (v3)

```
GET /friend/api/v3/friends/{userId}
```

| Param | Type | Description |
|-------|------|-------------|
| `showBigParty` | int | Always 1 |

**Returns**: `FriendUser`

> Added: v3.20.3

---

### Get Friend's Gaming Status

```
GET /friend/api/v1/friends/{userId}/gaming
```

| Param | Type | Description |
|-------|------|-------------|
| `showBigParty` | int | Always 1 |
| `engine4Version` | int | Engine 4 version (e.g., 40040) |

**Returns**: Gaming info with `gameId`, `gamingInfo.regionId`

> Added: v2.95.3. Used by WZRD-TOOL's new spy method to detect which game a player is in.

---

### Get Friend Request Count

```
GET /friend/api/v1/friends/apply/num
```

**Returns**: int

> Added: v2.95.3

---

### Get Friend Notifications

```
GET /friend/api/v1/friends/notice-list
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

> Added: v2.95.3

---

### Send Friend Request

```
POST /friend/api/v1/friends
```

Body:
```json
{
  "channel": 7,
  "friendId": 12345,
  "gameId": "",
  "msg": "wzrd add",
  "type": 1
}
```

| Field | Type | Description |
|-------|------|-------------|
| `channel` | int | Channel (7 = standard) |
| `friendId` | int | Target user ID |
| `gameId` | string | Game context (optional, e.g., `g1008`) |
| `msg` | string | Message |
| `type` | int | Request type |

> Added: v2.95.3

---

### Get Popularity

```
GET /friend/api/v1/popularity/{userId}
```

**Returns**: `Popularity` — popularity count, like count

> Added: v2.95.3

---

### Add Popularity (Like)

```
POST /friend/api/v1/popularity
```

| Param | Type | Description |
|-------|------|-------------|
| `friendId` | int | User ID to like |

> Added: v2.95.3

---

### Delete Friend (Blacklist)

```
DELETE /friend/api/v1/friends/black
```

| Param | Type | Description |
|-------|------|-------------|
| `friendId` | int | Friend user ID |

> Added: v2.95.3

---

### Set Friend Alias

```
POST /friend/api/v1/friends/{friendId}/alias
```

| Param | Type | Description |
|-------|------|-------------|
| `alias` | string | Nickname alias |

> Added: v2.95.3

---

### Approve Friend Request

```
PUT /friend/api/v1/friends/{friendId}/agreement
```

> Added: v2.95.3

---

### Reject Friend Request

```
PUT /friend/api/v1/friends/{friendId}/rejection
```

> Added: v2.95.3

---

### Approve All Friend Requests

```
POST /friend/api/v1/friends/requests/approve-all
```

> Added: v3.20.3

---

### Reject All Friend Requests

```
POST /friend/api/v1/friends/requests/reject-all
```

> Added: v3.20.3

---

### Get Friend Tags

```
GET /friend/api/v1/tag/only
```

> Added: v2.95.3

---

### Get Family Recruit Status

```
GET /friend/api/v1/family/recruit/status
```

> Added: v2.95.3

---

### Get Family Recruit List

```
GET /friend/api/v1/family/recruit
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | string | Filter type (`0` = all) |

| Extra Header | Value |
|--------------|-------|
| `userlanguage` | `ru_RU` |

**Returns**: List of recruitment ads with `ownerId`, `nickName`, `status`

> Added: v2.95.3. Used by WZRD-TOOL's mass family spam feature.

---

### Apply to Family

```
POST /friend/api/v1/family/apply
```

Body:
```json
{
  "age": 999999999,
  "msg": "wzrd",
  "ownerId": 12345,
  "sex": 1,
  "type": 1
}
```

> Added: v2.95.3. Allows arbitrary age values up to 2147483647 (int32 max).

---

### Publish Family Recruitment

```
POST /friend/api/v1/family/recruit
```

Body:
```json
{
  "age": 200,
  "memberName": "wzrd",
  "memberType": 2,
  "msg": "wzrd",
  "ownerName": "wzrd",
  "ownerType": 2
}
```

| Extra Header | Value |
|--------------|-------|
| `appversion` | `1488` |
| `userlanguage` | `ru_RU` |

> Added: v2.95.3

---

### Delete Family Recruitment

```
DELETE /friend/api/v1/family/recruit
```

Body: Same as publish (identifies which ad to delete)

> Added: v2.95.3

---

### Get Homeland Entrance

```
GET /friend/api/v1/homeland/suspended/entrance
```

> Added: v2.95.3

---

## Clan Endpoints

### Get My Clan (v3)

```
GET /clan/api/v3/clan/tribe
```

**Returns**: `ClanInfo` — clanId, name, headPic, tags, level, chiefId, currentCount, maxCount

> Added: v3.20.3

---

### Get My Clan Base (v1)

```
GET /clan/api/v1/clan/tribe/base
```

**Returns**: `ClanInfo` — clanId, name, level, currentCount, maxCount, experience, region, details, headPic, tags

> Added: v2.95.3. Used by WZRD-TOOL for clan operations.

---

### Get Clan Info (v2)

```
GET /clan/api/v2/clan/tribe
```

| Param | Type | Description |
|-------|------|-------------|
| `clanId` | int | Clan ID |

> Added: v3.20.3

---

### Get Recommended Clans

```
GET /clan/api/v1/clan/tribe/recommendation
```

**Returns**: List of `ClanInfo`

> Added: v2.95.3

---

### Get My Clan ID

```
GET /clan/api/v1/clan/tribe/id
```

**Returns**: int (0 = no clan)

> Added: v2.95.3

---

### Get Clan Red Points

```
GET /clan/api/v1/red-point
```

> Added: v2.95.3

---

### Get Clan Tasks

```
GET /clan/api/v3/clan/tasks
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | int | Task type (0 = all) |

> Added: v3.20.3

---

### Get Clan Members

```
GET /clan/api/v1/clan/tribe/member
```

> Added: v2.95.3

---

### Get Clan Bulletin

```
GET /clan/api/v1/clan/tribe/bulletin
```

> Added: v2.95.3

---

### Get Clan Currency

```
GET /clan/api/v1/clan/tribe/currency
```

> Added: v2.95.3

---

### Create Clan (v3)

```
POST /clan/api/v3/clan/tribe
```

Body:
```json
{
  "clanId": 0,
  "currency": 2,
  "details": "Clan description",
  "headPic": "",
  "name": "My Clan",
  "tags": ["tag1", "tag2"]
}
```

> Added: v3.20.3

---

### Edit Clan (v1)

```
PUT /clan/api/v1/clan/tribe
```

Body:
```json
{
  "clanId": 12345,
  "currency": 0,
  "details": "New description",
  "headPic": "https://...",
  "name": "Clan Name",
  "tags": ["tag1", "tag2", "tag3"]
}
```

**Notes**: The `tags` field accepts a text string, allowing more tags than the official client permits. Tag count is not validated server-side.

> Added: v2.95.3. Used by WZRD-TOOL's extended tag feature.

---

### Join Clan

```
POST /clan/api/v1/clan/tribe/member
```

Body:
```json
{
  "clanId": 12345,
  "msg": "Please let me join!"
}
```

> Added: v2.95.3

---

### Leave Clan

```
DELETE /clan/api/v1/clan/tribe/member
```

| Param | Type | Description |
|-------|------|-------------|
| `clanId` | int | Clan ID |

> Added: v2.95.3

---

### Search Clans

```
GET /clan/api/v1/clan/tribe/blurry/info
```

| Param | Type | Description |
|-------|------|-------------|
| `clanName` | string | Search query |
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

> Added: v2.95.3

---

### Invite to Clan

```
POST /clan/api/v1/clan/tribe/member/invite
```

Body:
```json
{
  "friendIds": [12345, 67890],
  "msg": "Join my clan!"
}
```

> Added: v2.95.3

---

### Agree Clan Member

```
PUT /clan/api/v1/clan/tribe/member/agreement
```

| Param | Type | Description |
|-------|------|-------------|
| `otherId` | int | Applicant user ID |

> Added: v2.95.3

---

### Reject Clan Member

```
PUT /clan/api/v1/clan/tribe/member/rejection
```

| Param | Type | Description |
|-------|------|-------------|
| `otherId` | int | Applicant user ID |

> Added: v2.95.3

---

### Mute Clan Member

```
POST /clan/api/v1/clan/tribe/mute/member
```

| Param | Type | Description |
|-------|------|-------------|
| `memberId` | int | Member ID |
| `minute` | int | Duration in minutes |

> Added: v2.95.3

---

### Unmute Clan Member

```
DELETE /clan/api/v1/clan/tribe/mute/member
```

| Param | Type | Description |
|-------|------|-------------|
| `memberId` | int | Member ID |

> Added: v2.95.3

---

### Mute All Clan

```
PUT /clan/api/v1/clan/tribe/mute
```

| Param | Type | Description |
|-------|------|-------------|
| `muteStatus` | int | 1 = mute, 0 = unmute |

> Added: v2.95.3

---

### Remove Clan Members (Batch)

```
DELETE /clan/api/v1/clan/tribe/member/remove/batch
```

Body: Array of member IDs

> Added: v2.95.3

---

### Edit Clan Elders

```
PUT /clan/api/v1/clan/tribe/members
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | string | `add` or `remove` |
| `otherIds` | string | Comma-separated user IDs |

> Added: v2.95.3

---

### Set Clan Authentication

```
PUT /clan/api/v1/clan/free/verification
```

| Param | Type | Description |
|-------|------|-------------|
| `freeVerify` | int | 1 = enabled, 0 = disabled |

> Added: v2.95.3

---

### Buy Clan Decoration

```
PUT /clan/api/v1/clan/decorations/purchase
```

| Param | Type | Description |
|-------|------|-------------|
| `decorationId` | int | Decoration ID |

> Added: v2.95.3

---

### Accept Clan Task

```
PUT /clan/api/v1/clan/tasks/accept
```

| Param | Type | Description |
|-------|------|-------------|
| `id` | int | Task ID |
| `type` | int | 0 = team, 1 = personal |

> Added: v2.95.3

---

### Claim Clan Task

```
PUT /clan/api/v1/clan/tasks
```

| Param | Type | Description |
|-------|------|-------------|
| `id` | int | Task ID |
| `type` | int | 0 = team, 1 = personal |

> Added: v2.95.3

---

### Get Personal Clan Tasks (v3)

```
GET /clan/api/v3/clan/personal/tasks
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | int | Always 1 |

> Added: v3.20.3

---

### Update Clan Notice

```
POST /clan/api/v1/clan/tribe/bulletin
```

Body:
```json
{
  "content": "New clan notice text"
}
```

> Added: v2.95.3

---

### Transfer Clan Chief

```
PUT /clan/api/v1/clan/tribe/member
```

| Param | Type | Description |
|-------|------|-------------|
| `otherId` | int | New chief ID |
| `type` | int | Always 3 |

> Added: v2.95.3

---

### Dissolve Clan

```
DELETE /clan/api/v1/clan/tribe
```

| Param | Type | Description |
|-------|------|-------------|
| `clanId` | int | Clan ID |

> Added: v2.95.3

---

### Get Clan Rank

```
GET /clan/api/v1/clan/rank
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | string | `"all"` |
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

| Extra Header | Value |
|--------------|-------|
| `language` | `wzrd` |

**Returns**: `{ "data": { "pageInfo": { "totalSize": int } } }`

> Added: v2.95.3. Used by WZRD-TOOL to get total clan count.

---

## Decoration Endpoints

### Get Equipped Decorations

```
GET /decoration/api/v1/decorations/using
```

| Param | Type | Description |
|-------|------|-------------|
| `otherId` | int | User ID (0 = self) |

**Returns**: List of `EquippedDecoration`

> Added: v2.95.3

---

### Get User Decorations (v2)

```
GET /decoration/api/v1/decorations-v2/{userId}/using
```

> Added: v3.20.3

---

### Equip Decoration

```
PUT /decoration/api/v1/decorations/using/new
```

| Param | Type | Description |
|-------|------|-------------|
| `ids` | int | Decoration ID |

> Added: v2.95.3

---

### Unequip Decoration

```
DELETE /decoration/api/v1/decorations/using/new
```

| Param | Type | Description |
|-------|------|-------------|
| `ids` | int | Decoration ID |

> Added: v2.95.3

---

### Get User Decorations by Category

```
GET /decoration/api/v1/new/decorations/users/{userId}/classify/{category}
```

Categories: `all`, `beijing`, `beishi`, `dongzuo`, `fashi`, `fuse`, `huangguan`, `huanraowu`, `jianshi`, `kuzi`, `liantiyi`, `mianshi`, `shangyi`, `toufa`, `weishi`, `xie`, `zhengmian`

| Param | Type | Description |
|-------|------|-------------|
| `engineVersion` | int | Engine version |
| `os` | string | `android` |
| `showVip` | int | 1 |

> Added: v2.95.3

---

### Get User Suits

```
GET /decoration/api/v1/new/decorations/users/{userId}/suit
```

| Param | Type | Description |
|-------|------|-------------|
| `os` | string | `android` |
| `engineVersion` | int | Engine version |

> Added: v2.95.3

---

### Check Decoration Resources

```
GET /decoration/api/v1/new/decorations/check/resource
```

| Param | Type | Description |
|-------|------|-------------|
| `resVersion` | int | Resource version |
| `engineVersion` | int | Engine version |

> Added: v2.95.3

---

### Get Decoration Recommendations

```
GET /decoration/api/v1/recommend/get
```

| Param | Type | Description |
|-------|------|-------------|
| `triggerType` | int | Trigger type |

> Added: v2.95.3

---

### Get Decoration Red Points

```
GET /decoration/api/v1/redpoints
```

**Returns**: `DecorationRedPoints` — summary, newPublish

> Added: v2.95.3

---

### Click Decoration Red Points

```
POST /decoration/api/v1/redpoints/click
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | string | `summary` |

> Added: v2.95.3

---

### Get Decoration Model Number

```
GET /decoration/api/v1/user/decoration/model/number
```

> Added: v2.95.3

---

### Get Expiring Decorations

```
GET /decoration/api/v1/decorations-v2/users/{userId}/expire
```

> Added: v3.20.3

---

### Get Decoration Collections

```
GET /decoration/api/v1/collection/user/list
```

> Added: v2.95.3

---

### Get Current Decoration Price

```
POST /decoration/api/v1/decoration/current/price
```

> Added: v2.95.3

---

## Payment/Shop Endpoints

### Get Wealth

```
GET /pay/api/v1/wealth/user
```

**Returns**: `Wealth` — diamonds, golds, gDiamonds, money

> Added: v2.95.3

---

### Get Payment Red Points

```
GET /pay/api/v1/red-point
```

**Returns**: `PaymentRedPoints` — monthCard, directPurchase

> Added: v2.95.3

---

### Get GCube Pack Info

```
GET /activity/api/v1/limit/gcube/pack/info
```

> Added: v3.20.3

---

### Get Growth Fund Info

```
GET /pay/api/v1/growth/fund/info
```

| Param | Type | Description |
|-------|------|-------------|
| `gameId` | string | Game ID |

> Added: v2.95.3

---

### Get Xsolla Billing

```
GET /pay/api/v1/xsolla/billing/get
```

> Added: v2.95.3

---

### Is Xsolla Enabled

```
GET /pay/api/v1/xsolla/enable
```

> Added: v2.95.3

---

### Browse Shop Decorations

```
GET /shop/api/v1/new/shop/decorations/classify/{category}
```

| Param | Type | Description |
|-------|------|-------------|
| `engineVersion` | int | Engine version |
| `os` | string | `android` |
| `showVip` | int | 1 |

**Returns**: List of `ShopDecoration`

> Added: v2.95.3

---

### Get Recommended Decorations

```
GET /shop/api/v1/new/shop/recommend-decorations
```

| Param | Type | Description |
|-------|------|-------------|
| `engineVersion` | int | Engine version |
| `type` | string | `hot` or `new` |
| `showVip` | int | 1 |
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

> Added: v2.95.3

---

### Get Suit Decorations

```
GET /shop/api/v1/new/shop/suit/decorations
```

| Param | Type | Description |
|-------|------|-------------|
| `os` | string | `android` |
| `engineVersion` | int | Engine version |

**Returns**: List of `ShopSuit`

> Added: v2.95.3

---

### Buy Decoration

```
POST /shop/api/v1/new/shop/decorations/buy
```

| Param | Type | Description |
|-------|------|-------------|
| `diamond` | int | Diamonds to spend |
| `gold` | int | Gold to spend |
| `clothVoucher` | int | Cloth vouchers to spend |
| `payType` | string | Payment type |

> Added: v2.95.3

---

### Get Game Props Shop (v2)

```
GET /shop/api/v2/shop/game/props/new
```

| Param | Type | Description |
|-------|------|-------------|
| `gameId` | string | Game ID (e.g., `g1008`) |
| `engineVersion` | int | Engine version |

| Extra Header | Value |
|--------------|-------|
| `language` | `ru` |

**Returns**: List of game-specific shop items with `id`, `name`, `price`, `currency`, `status`, `expireDate`

> Added: v2.95.3. Used by WZRD-TOOL to access the old/hidden shop.

---

### Buy Game Props (v2)

```
PUT /shop/api/v2/shop/game/props/new
```

| Param | Type | Description |
|-------|------|-------------|
| `gameId` | string | Game ID |
| `propsId` | string | Item ID |

| Extra Header | Value |
|--------------|-------|
| `language` | `ru` |

> Added: v2.95.3

---

## Backpack Endpoints

### Get Backpack Contents (v2)

```
GET /backpack/api/v2/backpack/{userId}
```

| Param | Type | Description |
|-------|------|-------------|
| `pageSize` | int | Items per page (use 9999 for all) |

**Returns**: `{ "data": { "pageInfo": { "data": [{ "itemId": "10068", "amount": int }] } } }`

**Known Item IDs**:
| Item ID | Description |
|---------|-------------|
| `10068` | Event tickets |

> Added: v2.95.3. Used by WZRD-TOOL's `/w tickets` command.

---

## Mail/Message Endpoints

### Check New Mail

```
GET /mailbox/api/v1/mail/new
```

> Added: v2.95.3

---

### Get Mail List

```
GET /mailbox/api/v1/mail
```

> Added: v2.95.3

---

### Mark Mail Read

```
PUT /mailbox/api/v1/mail
```

| Param | Type | Description |
|-------|------|-------------|
| `status` | int | 2 = read |
| `ids` | int | Mail ID |

> Added: v2.95.3

---

### Get Group Chat List

```
GET /msg/api/v1/msg/group/chat/list
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

> Added: v2.95.3

---

### Get Group Chat Info

```
GET /msg/api/v1/msg/group/chat/info
```

| Param | Type | Description |
|-------|------|-------------|
| `groupId` | string | Group ID |

> Added: v2.95.3

---

### Create Group Chat (v2)

```
POST /msg/api/v2/msg/group/chat
```

Body:
```json
{
  "groupName": "My Group",
  "members": [12345, 67890]
}
```

> Added: v3.20.3

---

### Get Unread Notifications

```
GET /msg/api/v1/user-notification/unread
```

> Added: v2.95.3

---

### Get Group Join Requests

```
GET /msg/api/v1/msg/group/chat/request/list
```

> Added: v2.95.3

---

### Get Group Creation Cost

```
GET /msg/api/v1/group/chat/price
```

> Added: v2.95.3

---

### Edit Group Chat

```
PUT /msg/api/v1/msg/group/chat/modify
```

Body: JSON with fields to update (groupName, groupNotice, inviteStatus, noticePic)

> Added: v2.95.3

---

### Set Group Member Admin

```
PUT /msg/api/v1/msg/group/chat/member
```

Body: `{ groupId, inviterId, memberIds, operationType }`

> Added: v2.95.3

---

### Mute Group Member

```
POST /msg/api/v1/msg/group/chat/forbidden/member
```

Body: `{ groupId, memberId, minutes }`

> Added: v2.95.3

---

### Invite to Group

```
POST /msg/api/v1/msg/group/chat/invite
```

Body: `{ groupId, memberIds }`

> Added: v2.95.3

---

### Kick from Group

```
PUT /msg/api/v1/msg/group/chat/kickOut
```

Body: `{ groupId, groupName, inviterId, memberIds }`

> Added: v2.95.3

---

### Approve Group Request

```
PUT /msg/api/v1/msg/group/chat/agreement
```

Body: `{ groupId, operateId, type, userId }`

> Added: v2.95.3

---

### Quit Group

```
PUT /msg/api/v1/msg/group/chat/quit
```

| Param | Type | Description |
|-------|------|-------------|
| `groupId` | string | Group ID |
| `groupName` | string | Group name |

> Added: v2.95.3

---

### Apply to Group

```
POST /msg/api/v1/msg/group/chat/apply
```

| Param | Type | Description |
|-------|------|-------------|
| `groupId` | string | Group ID |
| `msg` | string | Join message |

> Added: v2.95.3

---

### Transfer Group

```
PUT /msg/api/v1/msg/group/chat/transfer
```

Body: `{ groupId, inviterId, userId }`

> Added: v2.95.3

---

## Activity Endpoints

### Get Activity Home Page

```
GET /activity/api/v1/activity/home/page
```

> Added: v2.95.3

---

### Get Activity List Groups (v2)

```
GET /activity/api/v2/homepage/activity-list/group
```

> Added: v3.20.3

---

### Get Ad Banner

```
GET /activity/api/v1/homepage/ad-banner
```

| Param | Type | Description |
|-------|------|-------------|
| `language` | string | Language code |
| `userId` | int | User ID |
| `region` | string | Region |
| `tabCode` | string | Tab code |

> Added: v2.95.3

---

### Get Theme Info (v2)

```
GET /activity/api/v2/theme
```

> Added: v3.20.3

---

### Get Newbie Activity Status (v2)

```
GET /activity/api/v2/newbie/activity/status
```

> Added: v3.20.3

---

### Get Holiday Score Info

```
GET /activity/api/v1/holiday-score/info
```

| Param | Type | Description |
|-------|------|-------------|
| `activityId` | string | Activity ID |

> Added: v3.21.1

---

### Get Holiday Score Ranking

```
GET /activity/api/v1/holiday-score/rank/period
```

| Param | Type | Description |
|-------|------|-------------|
| `activityId` | string | Activity ID |
| `periodKey` | string | Period key |
| `page` | int | Page number |
| `pageSize` | int | Items per page |

> Added: v3.21.1

---

### Get Clap Face Images (v3)

```
GET /activity/api/v3/clap/face/image/get
```

| Param | Type | Description |
|-------|------|-------------|
| `ignorePositionType` | int | 1 |

> Added: v3.21.1

---

### Get Background Image

```
GET /activity/api/v1/background-image
```

> Added: v2.95.3

---

### Get Game Detail Activity

```
GET /activity/api/v1/homepage/game-detail/icon-activity
```

| Param | Type | Description |
|-------|------|-------------|
| `gameId` | string | Game ID |

> Added: v2.95.3

---

## Config Endpoints

### Get Config

```
GET /config/files/{configName}
```

Known config names: `blockmods-config-v1`, `blockymods-check-version`, `game_resource_config`, `sharing-game-resource-config`, `deeplink-remote-config`, `game-detail-to-editor`, `app_sticker_config`

> Added: v2.95.3

---

### Check App Version (Official)

```
GET /config/files/blockymods-official-check-version
```

**Returns**: `{ "data": { "apkUrl": "https://...", "engineN": "version" } }`

**Notes**: The `apkUrl` field contains engine version numbers in the URL pattern `engine{N}-{version}`. This is used to dynamically determine engine versions at runtime.

> Added: v2.95.3. Used by WZRD-TOOL to fetch engine versions and APK URL.

---

### Get Game Resource Config

```
GET /config/files/game_resource_config
```

> Added: v2.95.3

---

### Get ML Config

```
GET /config/ml/files/{configName}
```

Known ML configs: `common_lang_img_url_config`, `game_room_def_title_config`, `game_room_tab_config`, `im_fraud_rule_config`, `new_player_guide_for_rbx_config`, `tribe_introduction_banner`

> Added: v2.95.3

---

### Get Game Room Tab Config

```
GET /config/ml/files/game_room_tab_config
```

> Added: v2.95.3

---

## Other Endpoints

### Get Server Time

```
GET /server-time
```

> Added: v2.95.3

---

### Get Comment Window Flag

```
GET /process/api/v1/comment/window/flag
```

| Param | Type | Description |
|-------|------|-------------|
| `userId` | int | User ID |

> Added: v2.95.3

---

## Version History

| Version | Build | Date | API Changes |
|---------|-------|------|-------------|
| v2.95.3 | 5123 | 2024-06 | Initial API documentation. Core endpoints for user, friend, clan, game, shop, decoration, mail, activity, config. |
| v3.20.3 | 5663 | 2025-09 | Added v3 endpoints (user details, friend status, clan info, group chat). Added user-agent `okhttp/4.12.0`. |
| v3.21.1 | 5671 | 2025-12 | NetEase anti-tamper wrapper. Added mining, personal space, holiday score, clap face, auth-token v5 endpoints. Added `appversion`/`appversionname` headers. New ad SDKs (Chartboost, InMobi, TradPlus, Bigo, MBridge, IronSource). |

---

## Security Notes

- **No certificate pinning** — MITM interception possible
- **MD5 request signing** — cryptographically broken
- **Static API keys** — never rotate across versions
- **Plain integer user IDs** — enumerable/sequential
- **Game server IPs leaked** in auth responses
- **No rate limiting** observed on any endpoint
- **1024-bit RSA** for password encryption (below 2048-bit minimum)
- **Analytics sent over HTTP** (not HTTPS) to `http://di.sandboxol.com`
