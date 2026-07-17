# Data Models

All API responses are mapped to typed Python dataclasses.

---

## Enums

### GameMode

Room type filters:

| Value | Description |
|-------|-------------|
| `ALL` | All rooms (empty string) |
| `PVP` | Player vs Player |
| `RUN` | Run/escape mode |
| `CREATE` | Creative/build mode |
| `SOCIAL` | Social/hangout mode |
| `GAME` | Minigames |
| `MINING` | Mining mode |
| `RECENT_PLAY` | Recently played |
| `RECENT_CREATE` | Recently created |

### Sex

| Value | Description |
|-------|-------------|
| `0` | Unspecified |
| `1` | Male |
| `2` | Female |

### EngineType

| Value | Version ID | Games |
|-------|------------|-------|
| `V1` | 10114 | Bed Wars, Sky Block, TNT Tag |
| `V2` | 20084 | Free City RP, Social Hall |
| `V3` | 40040 | Party games |

---

## Core Models

### APIResponse

Standard API response envelope.

| Field | Type | Description |
|-------|------|-------------|
| `code` | int | 1 = success, other = error |
| `message` | str | Response message |
| `data` | Any | Response payload |
| `other` | Any | Additional data |

Property: `success` → `bool` (True if code == 1)

---

### UserProfile

User profile data.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `userId` | int | — | User ID |
| `sex` | int | 0 | Gender (0/1/2) |
| `nickName` | str | "" | Display name |
| `picUrl` | str | "" | Avatar URL |
| `picType` | int | 0 | Avatar type |
| `decorationPicUrl` | str | "" | Decorated avatar URL |
| `telephone` | str | "" | Phone number |
| `email` | str | "" | Email address |
| `birthday` | str | "" | Birthday (YYYY-MM-DD) |
| `details` | str | "" | Bio/details |
| `diamonds` | int | 0 | Diamond balance |
| `golds` | int | 0 | Gold balance |
| `vipLevel` | int | 0 | VIP level |
| `userLevel` | int | 0 | User level |
| `region` | str | "" | Region code |
| `createTime` | int | 0 | Account creation timestamp |

---

### FriendUser

Friend list item.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `userId` | int | — | Friend's user ID |
| `nickName` | str | "" | Display name |
| `picUrl` | str | "" | Avatar URL |
| `decorationPicUrl` | str | "" | Decorated avatar |
| `alias` | str | None | Custom nickname |
| `status` | int | 0 | 10 = offline, 30 = online |
| `gameId` | str | None | Current game ID |
| `partyInfo` | dict | None | Party info |
| `vip` | int | 0 | VIP level |
| `sex` | int | 0 | Gender |
| `details` | str | "" | Bio |
| `clanId` | int | 0 | Clan ID |
| `clanName` | str | None | Clan name |
| `role` | int | 0 | Role in clan |

---

### SecuritySettings

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `userId` | int | — | User ID |
| `email` | str | "" | Email |
| `bindEmail` | bool | False | Email bound |
| `secretQuestionList` | list | [] | Security questions |
| `ids` | list | [] | ID list |
| `country` | str | "" | Country |
| `region` | str | "" | Region |

---

### UserCareer

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `userId` | int | 0 | User ID |
| `completeRate` | float | 0 | Completion rate |
| `victoryRate` | float | 0 | Win rate |
| `killCount` | int | 0 | Total kills |
| `averageTime` | float | 0 | Avg session time |
| `totalTime` | float | 0 | Total play time |
| `timeCount` | int | 0 | Session count |
| `totalPlayedCount` | int | 0 | Games played |
| `friendNum` | int | 0 | Friend count |
| `familyNum` | int | 0 | Clan member count |
| `decorationCount` | int | 0 | Owned decorations |
| `suitCount` | int | 0 | Owned suits |

---

### UserShopInfo

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `userId` | int | — | User ID |
| `diamonds` | int | 0 | Diamonds |
| `golds` | int | 0 | Gold |
| `clothVoucher` | int | 0 | Cloth vouchers |
| `gdiamonds` | int | 0 | G-Diamonds |

---

### UserVip

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `userId` | int | — | User ID |
| `lv` | int | 0 | VIP level |
| `exp` | int | 0 | Current XP |
| `maxExp` | int | 100 | Max XP for level |
| `vipIcon` | str | "" | VIP icon URL |

---

### Popularity

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `popularity` | int | 0 | Popularity score |
| `like` | int | 0 | Like count |
| `hasNewLikes` | bool | None | New likes since last check |

---

### Wealth

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `userId` | int | — | User ID |
| `diamonds` | int | 0 | Diamonds |
| `golds` | int | 0 | Gold |
| `gDiamonds` | int | 0 | G-Diamonds |
| `gDiamondsProfit` | int | 0 | G-Diamond profit |
| `money` | float | 0.0 | Real money balance |
| `ngDiamonds` | float | 0.0 | NG Diamonds |
| `sameUser` | bool | False | Same user flag |
| `firstPunch` | bool | False | First punch flag |

---

## Shop Models

### ShopDecoration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | int | — | Decoration ID |
| `typeId` | int | 0 | Category type |
| `camera` | str | "" | Camera position |
| `name` | str | None | Item name |
| `iconUrl` | str | "" | Icon URL |
| `sex` | int | 0 | Gender restriction |
| `tag` | list | [] | Tags |
| `resourceId` | str | "" | Resource ID |
| `details` | str | None | Description |
| `expire` | int | 0 | Expiry timestamp |
| `price` | int | 0 | Diamond price |
| `discountPrice` | int | None | Discounted price |
| `clothVoucherPrice` | int | 0 | Voucher price |
| `discountClothVoucherPrice` | int | None | Discounted voucher price |
| `currency` | int | 1 | Currency type |
| `quantity` | int | 0 | Quantity |
| `isNew` | int | 0 | New item flag |
| `hasPurchase` | int | 0 | Purchased flag |

---

### ShopSuit

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `suitId` | int | — | Suit ID |
| `iconUrl` | str | "" | Icon URL |
| `currency` | int | 1 | Currency type |
| `name` | str | "" | Suit name |
| `details` | str | "" | Description |
| `isNew` | int | 0 | New flag |
| `sex` | int | 0 | Gender restriction |
| `isRecommend` | int | 0 | Recommended flag |
| `quantity` | int | 0 | Quantity |
| `hasPurchase` | int | 0 | Purchased flag |
| `limitedTimes` | list | [] | Limited time info |
| `discountPrices` | Any | None | Discount prices |
| `shopDecorationInfos` | list | [] | Included decorations |

---

### EquippedDecoration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | int | — | Decoration ID |
| `typeId` | int | 0 | Category type |
| `camera` | str | "" | Camera position |
| `name` | str | "" | Item name |
| `iconUrl` | str | "" | Icon URL |
| `status` | int | 0 | Status |
| `sex` | int | 0 | Gender restriction |
| `resourceId` | str | "" | Resource ID |
| `details` | str | "" | Description |
| `expire` | int | 0 | Expiry timestamp |
| `type` | int | 0 | Type |
| `clanLevel` | int | 0 | Required clan level |
| `vip` | int | 0 | Required VIP |
| `occupyPosition` | list | [] | Body positions |
| `buyTime` | Any | None | Purchase time |

---

### DecorationInfo

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | int | — | Decoration ID |
| `typeId` | int | 0 | Category type |
| `camera` | str | "" | Camera position |
| `name` | str | "" | Item name |
| `iconUrl` | str | "" | Icon URL |
| `sex` | int | 0 | Gender restriction |
| `tag` | list | [] | Tags |
| `resourceId` | str | "" | Resource ID |
| `details` | str | None | Description |
| `expire` | int | 0 | Expiry timestamp |
| `type` | int | 0 | Type |
| `status` | int | 0 | Ownership status |
| `price` | int | 0 | Diamond price |
| `clothVoucherPrice` | int | 0 | Voucher price |
| `currency` | int | 1 | Currency type |
| `occupyPosition` | list | [] | Body positions |

---

## Game Models

### GameRoom

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `roomId` | str | — | Room ID |
| `name` | str | "" | Room name |
| `userId` | int | 0 | Owner user ID |
| `nickName` | str | "" | Owner name |
| `picUrl` | str | "" | Owner avatar |
| `language` | str | "" | Room language |
| `mode` | str | "" | Game mode |
| `gameType` | str | "" | Game ID |
| `gameName` | str | "" | Game name |
| `gameIconUrl` | str | "" | Game icon |
| `engineType` | int | 0 | Engine version |
| `curUsers` | int | 0 | Current players |
| `maxUsers` | int | 0 | Max players |
| `gameAddr` | str | "" | Server IP:port |
| `regionId` | int | 0 | Region ID |
| `mapAddress` | str | "" | Map download URL |
| `needPassword` | bool | False | Password required |
| `miningLevel` | int | 0 | Mining level |
| `totalPoolValue` | int | 0 | Total mining pool |
| `remainingPoolValue` | int | 0 | Remaining pool |
| `miningRewardBonus` | float | 0.0 | Mining bonus |
| `newDeviceRebateRatio` | float | 0.0 | New device bonus |
| `miningStatus` | str | "" | Mining status |

---

### GameServer

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `region` | int | — | Region ID |
| `clan` | str | "defaultClan" | Clan identifier |
| `url` | str | "" | Server URL |

---

### GameDispatch

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `gid` | str | "" | Game ID |
| `gaddr` | str | "" | Game address |
| `name` | str | "" | Server name |
| `mid` | str | "" | Map ID |
| `mname` | str | "" | Map name |
| `downurl` | str | "" | Download URL |
| `region` | int | 0 | Region ID |
| `resVersion` | int | 0 | Resource version |
| `requestIds` | dict | {} | Request IDs |
| `gameType` | str | "" | Game type |
| `cdns` | list | [] | CDN mirrors |

---

### GameAuthResponse

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `token` | str | "" | JWT token |
| `signature` | str | "" | Token signature |
| `timestamp` | int | 0 | Token timestamp |
| `region` | int | 0 | Server region |
| `dispUrl` | str | "" | Game server URL |
| `engineType` | str | "" | Engine type |
| `country` | str | "" | Country code |
| `launchTraceId` | str | "" | Trace ID |
| `staging` | bool | False | Staging flag |

---

### GameResource

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `durl` | str | "" | Download URL |
| `resVersion` | int | 0 | Resource version |
| `size` | int | 0 | File size |
| `md5` | str | "" | File MD5 |
| `cdns` | list | [] | CDN mirrors |

---

### GameMetadata

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `gameId` | str | — | Game ID |
| `gameTitle` | str | "" | Game title |
| `bannerPic` | list | [] | Banner images |
| `gameDetail` | str | "" | Description |
| `praiseNumber` | float | 0 | Like count |
| `gameCoverPic` | str | "" | Cover image |
| `isPublish` | int | 0 | Published flag |
| `visitorEnter` | int | 0 | Visitor entry |
| `version` | int | 0 | Game version |
| `engineVersion` | int | 0 | Engine version |
| `onlineNumber` | int | 0 | Online players |
| `gameTypes` | list | [] | Game type tags |
| `isNewEngine` | int | 0 | New engine flag |
| `isUgcGame` | int | 0 | UGC flag |

---

### PartyGame

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `gameId` | str | — | Game ID |
| `gameName` | str | "" | Game name |
| `isNewEngine` | int | 0 | New engine flag |
| `onlineNumber` | int | 0 | Online players |
| `picUrl` | str | "" | Game icon |
| `isUgcGame` | int | 0 | UGC flag |
| `engineVersion` | int | 0 | Engine version |
| `isGameMode` | int | 0 | Game mode flag |
| `gameDetail` | str | "" | Description |

---

## Clan/Group Models

### ClanInfo

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `clanId` | int | — | Clan ID |
| `name` | str | "" | Clan name |
| `headPic` | str | "" | Clan icon |
| `tags` | list | [] | Tags |
| `details` | str | "" | Description |
| `level` | int | 0 | Clan level |
| `chiefId` | int | 0 | Leader user ID |
| `chiefNickName` | str | "" | Leader name |
| `currentCount` | int | 0 | Current members |
| `maxCount` | int | 0 | Max members |
| `freeVerify` | int | 0 | Auto-join enabled |

---

### GroupChat

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `ownerId` | int | 0 | Owner user ID |
| `groupId` | str | "" | Group ID |
| `groupPic` | str | None | Group icon |
| `groupName` | str | "" | Group name |
| `groupNotice` | str | None | Group notice |
| `noticePic` | list | [] | Notice images |
| `officialGroup` | int | 0 | Official flag |
| `releaseTime` | str | "" | Release time |
| `ownerRegion` | str | "" | Owner region |
| `forbiddenWordsStatus` | int | 0 | Word filter |
| `inviteStatus` | int | 0 | Invite settings |
| `groupMembers` | list | [] | Member list |

---

## Other Models

### AvatarFrame

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | str | — | Frame ID |
| `resourceId` | str | "" | Resource ID |
| `picUrl` | str | "" | Frame image |
| `status` | int | 0 | Status |
| `priority` | int | 0 | Display priority |
| `name` | str | "" | Frame name |
| `desc` | str | "" | Description |

---

### PaymentRedPoints

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `monthCard` | bool | False | Month card notification |
| `directPurchase` | bool | False | Direct purchase notification |

---

### DecorationRedPoints

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `summary` | int | 0 | Summary red dot count |
| `newPublish` | int | 0 | New items count |

---

### AuthTokenResponse

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `userId` | int | — | User ID |
| `accessToken` | str | "" | Auth token |
| `clientIp` | str | "" | Client IP |
| `httpDN` | str | None | HTTP domain name |
| `httpsDN` | str | None | HTTPS domain name |
| `country` | str | "" | Country |
| `region` | str | "" | Region |
| `cuboOpenId` | str | None | Cubo open ID |
| `isNewUser` | bool | False | New user flag |

---

### MiningTokenBalance

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `tokenAmount` | int | 0 | Main token amount |
| `subTokenAmount` | int | 0 | Sub token amount |
| `itemInfoMap` | dict | {} | Item info map |
| `transformInfoList` | list | [] | Transform info |
| `isTransform` | bool | False | Transform available |

---

### UGCWork

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `id` | int | 0 | Work ID |
| `archiveId` | int | 0 | Archive ID |
| `mapId` | str | "" | Map ID |
| `userId` | int | 0 | Creator ID |
| `userNickname` | str | "" | Creator name |
| `userAvatar` | str | "" | Creator avatar |
| `workName` | str | "" | Work name |
| `workIcon` | str | "" | Work icon |

---

### PingServer

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `region` | int | — | Region ID |
| `clan` | str | "defaultClan" | Clan identifier |
| `url` | str | "" | Server URL |

---

### UserHighlights

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `userId` | int | 0 | User ID |
| `gamePreferences` | list | [] | Game preference data |

---

### CDNMirror

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `cdnId` | str | "" | CDN ID |
| `cdnName` | str | "" | CDN name |
| `cdnUrl` | str | "" | CDN URL |
| `url` | str | "" | Mirror URL |
