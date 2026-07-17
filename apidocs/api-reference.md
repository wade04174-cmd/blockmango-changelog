# API Reference

Complete documentation of all BlockmanGO REST API endpoints.

**Base URL**: `https://gw.sandboxol.com`

---

## Game Endpoints

### Get Game Info

```
GET /game/api/v3/games/{gameId}
```

| Param | Type | Description |
|-------|------|-------------|
| `gameId` | string | Game identifier (e.g., `g1008`) |
| `appVersion` | int | App version code (default: 5663) |

**Returns**: `GameMetadata` — game title, description, banners, online count, engine version

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

---

### Get Game Update Tips

```
GET /game/api/v1/games/update/tip/info/app/{gameId}
```

| Param | Type | Description |
|-------|------|-------------|
| `engineVersion` | int | Engine version |
| `isNew` | int | 0 or 1 |

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

---

### Get Games Requiring Update

```
GET /game/api/v1/games/update/list/{userId}
```

| Param | Type | Description |
|-------|------|-------------|
| `oldEngineVersion` | int | Current engine version |
| `newEngineVersion` | int | Target engine version |

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

---

### Get Party-Enabled Games

```
GET /game/api/v1/games/all/open/party
```

| Param | Type | Description |
|-------|------|-------------|
| `appVersion` | int | App version |

**Returns**: List of `PartyGame` objects

---

### Get Mining Token Balance

```
GET /game/api/v1/backpack/mining-token/balance/with-transform
```

**Returns**: `MiningTokenBalance` — token amounts, item info, transform data

---

### Get User Highlights

```
GET /game/api/v1/user/highlight/data
```

| Param | Type | Description |
|-------|------|-------------|
| `targetId` | int | User ID |

**Returns**: `UserHighlights` — game preferences

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

---

### Get Creative Rooms

```
GET /game/api/v1/gameroom/creative/list
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

---

### Get Recently Played Rooms

```
GET /game/api/v1/gameroom/recent/played
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Always 0 |

---

### Get Recently Created Rooms

```
GET /game/api/v1/gameroom/recent/created
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Always 0 |

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

---

### Get Ping Servers

```
GET /game/api/v1/ping/server/list
```

**Returns**: List of `PingServer` — region, clan, URL

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

---

### Dispatch Game Server

```
POST {game_server}/v1/dispatch
```

**Returns**: `GameDispatch` — server assignment, CDN info

---

## User Endpoints

### Get User Info

```
GET /user/api/v1/user/info
```

**Returns**: `UserProfile` — userId, nickName, picUrl, diamonds, golds, vipLevel, region

---

### Get User Details

```
GET /user/api/v3/user/details/info
```

**Returns**: `UserProfile` — extended user data

---

### Update User Info

```
PUT /user/api/v1/user/info
```

Body: JSON with fields to update (nickName, picUrl, sex, etc.)

---

### Get User Career

```
GET /user/api/v1/users/bg-careers
```

| Param | Type | Description |
|-------|------|-------------|
| `userId` | int | Target user ID |

**Returns**: `UserCareer` — killCount, victoryRate, totalPlayedCount, totalTime, etc.

---

### Get Security Settings

```
GET /user/api/v2/users/verify/user/security/settings
```

| Param | Type | Description |
|-------|------|-------------|
| `userId` | int | User ID |

**Returns**: `SecuritySettings` — email, bindEmail, secretQuestionList

---

### Get User Shop Info

```
GET /user/api/v1/user/shop/info
```

**Returns**: `UserShopInfo` — diamonds, golds, clothVoucher, gdiamonds

---

### Get Avatar Frames

```
GET /user/api/v1/user/avatar/frame
```

**Returns**: List of `AvatarFrame` objects

---

### Get VIP Info

```
GET /user/api/v1/vip/users/{userId}
```

**Returns**: `UserVip` — level, exp, maxExp, vipIcon

---

### Get VIP Privileges

```
GET /user/api/v1/vip/privilege/all
```

**Returns**: List of privilege definitions

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

---

### Set Language

```
GET /user/api/v1/user/language
```

| Param | Type | Description |
|-------|------|-------------|
| `userId` | int | User ID |
| `language` | string | Language code (e.g., `en`) |

---

### Get Device Token

```
GET /user/api/v1/users/device/token
```

**Returns**: RongCloud IM token string

---

### Get AB Test Assignments

```
GET /user/api/v1/abtest/getUserTest
```

**Returns**: AB test assignment data

---

### Trigger Client Event

```
POST /user/api/v1/client/trigger/event
```

| Param | Type | Description |
|-------|------|-------------|
| `triggerType` | int | Event type (0, 1, etc.) |

---

### Get Chat Migration Data

```
GET /user/api/v1/chat/migrate/data
```

**Returns**: RongCloud key rotation data

---

### Get Personal Space Effects

```
GET /user/api/v1/personal/space/effects
```

**Returns**: List of available effects

---

### Get Account Auth Token (v5)

```
GET /user/api/v5/account/auth-token
```

| Param | Type | Description |
|-------|------|-------------|
| `q` | string | Base64 encoded query |

**Returns**: `AuthTokenResponse` — userId, accessToken, clientIp, country, region

---

## Account Registration

### Check Account Available

```
POST /user/api/v1/account/invalid/check
```

| Param | Type | Description |
|-------|------|-------------|
| `account` | string | Desired username |
| `type` | int | Always 1 |

**Returns**: `bool` — true if available

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

---

## Friend Endpoints

### Get Friends

```
GET /friend/api/v3/friends
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page (max 500) |
| `showBigParty` | int | Always 1 |

**Returns**: List of `FriendUser` objects

---

### Get Friend Status

```
GET /friend/api/v3/friends/{userId}
```

| Param | Type | Description |
|-------|------|-------------|
| `showBigParty` | int | Always 1 |

**Returns**: `FriendUser`

---

### Get All Friends Status

```
GET /friend/api/v2/friends/status
```

| Param | Type | Description |
|-------|------|-------------|
| `showBigParty` | int | Always 1 |

**Returns**: List of `FriendUser`

---

### Get Friend Request Count

```
GET /friend/api/v1/friends/apply/num
```

**Returns**: int

---

### Get Friend Notifications

```
GET /friend/api/v1/friends/notice-list
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

---

### Send Friend Request

```
POST /friend/api/v1/friends
```

Body:
```json
{
  "channel": 1,
  "friendId": 12345,
  "gameId": "",
  "msg": "Let's play!",
  "type": 1
}
```

---

### Get Popularity

```
GET /friend/api/v1/popularity/{userId}
```

**Returns**: `Popularity` — popularity count, like count

---

### Delete Friend

```
DELETE /friend/api/v1/friends
```

| Param | Type | Description |
|-------|------|-------------|
| `friendId` | int | Friend user ID |

---

### Add Popularity (Like)

```
POST /friend/api/v1/popularity
```

| Param | Type | Description |
|-------|------|-------------|
| `friendId` | int | User ID to like |

---

### Set Friend Alias

```
POST /friend/api/v1/friends/{friendId}/alias
```

| Param | Type | Description |
|-------|------|-------------|
| `alias` | string | Nickname alias |

---

### Approve Friend Request

```
PUT /friend/api/v1/friends/{friendId}/agreement
```

---

### Reject Friend Request

```
PUT /friend/api/v1/friends/{friendId}/rejection
```

---

### Blacklist Friend

```
DELETE /friend/api/v1/friends/black
```

| Param | Type | Description |
|-------|------|-------------|
| `friendId` | int | User ID to blacklist |

---

### Approve All Friend Requests

```
POST /friend/api/v1/friends/requests/approve-all
```

---

### Reject All Friend Requests

```
POST /friend/api/v1/friends/requests/reject-all
```

---

### Get Friend Tags

```
GET /friend/api/v1/tag/only
```

---

### Get Family Recruit Status

```
GET /friend/api/v1/family/recruit/status
```

---

### Get Homeland Entrance

```
GET /friend/api/v1/homeland/suspended/entrance
```

---

## Clan Endpoints

### Get My Clan

```
GET /clan/api/v3/clan/tribe
```

**Returns**: `ClanInfo` — clanId, name, headPic, tags, level, chiefId, currentCount, maxCount

---

### Get Clan Info

```
GET /clan/api/v2/clan/tribe
```

| Param | Type | Description |
|-------|------|-------------|
| `clanId` | int | Clan ID |

---

### Get Recommended Clans

```
GET /clan/api/v1/clan/tribe/recommendation
```

**Returns**: List of `ClanInfo`

---

### Get My Clan ID

```
GET /clan/api/v1/clan/tribe/id
```

**Returns**: int (0 = no clan)

---

### Get Clan Red Points

```
GET /clan/api/v1/red-point
```

---

### Get Clan Tasks

```
GET /clan/api/v3/clan/tasks
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | int | Task type (0 = all) |

---

### Get Clan Members

```
GET /clan/api/v1/clan/tribe/member
```

---

### Get Clan Bulletin

```
GET /clan/api/v1/clan/tribe/bulletin
```

---

### Get Clan Currency

```
GET /clan/api/v1/clan/tribe/currency
```

---

### Create Clan

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

---

### Leave Clan

```
DELETE /clan/api/v1/clan/tribe/member
```

| Param | Type | Description |
|-------|------|-------------|
| `clanId` | int | Clan ID |

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

---

### Agree Clan Member

```
PUT /clan/api/v1/clan/tribe/member/agreement
```

| Param | Type | Description |
|-------|------|-------------|
| `otherId` | int | Applicant user ID |

---

### Reject Clan Member

```
PUT /clan/api/v1/clan/tribe/member/rejection
```

| Param | Type | Description |
|-------|------|-------------|
| `otherId` | int | Applicant user ID |

---

### Mute Clan Member

```
POST /clan/api/v1/clan/tribe/mute/member
```

| Param | Type | Description |
|-------|------|-------------|
| `memberId` | int | Member ID |
| `minute` | int | Duration in minutes |

---

### Unmute Clan Member

```
DELETE /clan/api/v1/clan/tribe/mute/member
```

| Param | Type | Description |
|-------|------|-------------|
| `memberId` | int | Member ID |

---

### Mute All Clan

```
PUT /clan/api/v1/clan/tribe/mute
```

| Param | Type | Description |
|-------|------|-------------|
| `muteStatus` | int | 1 = mute, 0 = unmute |

---

### Remove Clan Members

```
DELETE /clan/api/v1/clan/tribe/member/remove/batch
```

Body: Array of member IDs

---

### Edit Clan

```
PUT /clan/api/v1/clan/tribe
```

Body: JSON with fields to update

---

### Edit Clan Elders

```
PUT /clan/api/v1/clan/tribe/members
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | string | `add` or `remove` |
| `otherIds` | string | Comma-separated user IDs |

---

### Set Clan Authentication

```
PUT /clan/api/v1/clan/free/verification
```

| Param | Type | Description |
|-------|------|-------------|
| `freeVerify` | int | 1 = enabled, 0 = disabled |

---

### Buy Clan Decoration

```
PUT /clan/api/v1/clan/decorations/purchase
```

| Param | Type | Description |
|-------|------|-------------|
| `decorationId` | int | Decoration ID |

---

### Accept Clan Task

```
PUT /clan/api/v1/clan/tasks/accept
```

| Param | Type | Description |
|-------|------|-------------|
| `id` | int | Task ID |
| `type` | int | 0 = team, 1 = personal |

---

### Claim Clan Task

```
PUT /clan/api/v1/clan/tasks
```

| Param | Type | Description |
|-------|------|-------------|
| `id` | int | Task ID |
| `type` | int | 0 = team, 1 = personal |

---

### Get Personal Clan Tasks

```
GET /clan/api/v3/clan/personal/tasks
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | int | Always 1 |

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

---

### Transfer Clan Chief

```
PUT /clan/api/v1/clan/tribe/member
```

| Param | Type | Description |
|-------|------|-------------|
| `otherId` | int | New chief ID |
| `type` | int | Always 3 |

---

### Dissolve Clan

```
DELETE /clan/api/v1/clan/tribe
```

| Param | Type | Description |
|-------|------|-------------|
| `clanId` | int | Clan ID |

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

---

### Get User Decorations (v2)

```
GET /decoration/api/v1/decorations-v2/{userId}/using
```

---

### Equip Decoration

```
PUT /decoration/api/v1/decorations/using/new
```

| Param | Type | Description |
|-------|------|-------------|
| `ids` | int | Decoration ID |

---

### Unequip Decoration

```
DELETE /decoration/api/v1/decorations/using/new
```

| Param | Type | Description |
|-------|------|-------------|
| `ids` | int | Decoration ID |

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

---

### Get User Suits

```
GET /decoration/api/v1/new/decorations/users/{userId}/suit
```

| Param | Type | Description |
|-------|------|-------------|
| `os` | string | `android` |
| `engineVersion` | int | Engine version |

---

### Check Decoration Resources

```
GET /decoration/api/v1/new/decorations/check/resource
```

| Param | Type | Description |
|-------|------|-------------|
| `resVersion` | int | Resource version |
| `engineVersion` | int | Engine version |

---

### Get Decoration Recommendations

```
GET /decoration/api/v1/recommend/get
```

| Param | Type | Description |
|-------|------|-------------|
| `triggerType` | int | Trigger type |

---

### Get Decoration Red Points

```
GET /decoration/api/v1/redpoints
```

**Returns**: `DecorationRedPoints` — summary, newPublish

---

### Click Decoration Red Points

```
POST /decoration/api/v1/redpoints/click
```

| Param | Type | Description |
|-------|------|-------------|
| `type` | string | `summary` |

---

### Get Decoration Model Number

```
GET /decoration/api/v1/user/decoration/model/number
```

---

### Get Expiring Decorations

```
GET /decoration/api/v1/decorations-v2/users/{userId}/expire
```

---

### Get Decoration Collections

```
GET /decoration/api/v1/collection/user/list
```

---

### Get Current Decoration Price

```
POST /decoration/api/v1/decoration/current/price
```

---

## Payment/Shop Endpoints

### Get Wealth

```
GET /pay/api/v1/wealth/user
```

**Returns**: `Wealth` — diamonds, golds, gDiamonds, money

---

### Get Payment Red Points

```
GET /pay/api/v1/red-point
```

**Returns**: `PaymentRedPoints` — monthCard, directPurchase

---

### Get GCube Pack Info

```
GET /activity/api/v1/limit/gcube/pack/info
```

---

### Get Growth Fund Info

```
GET /pay/api/v1/growth/fund/info
```

| Param | Type | Description |
|-------|------|-------------|
| `gameId` | string | Game ID |

---

### Get Xsolla Billing

```
GET /pay/api/v1/xsolla/billing/get
```

---

### Is Xsolla Enabled

```
GET /pay/api/v1/xsolla/enable
```

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

---

## Mail/Message Endpoints

### Check New Mail

```
GET /mailbox/api/v1/mail/new
```

---

### Get Mail List

```
GET /mailbox/api/v1/mail
```

---

### Mark Mail Read

```
PUT /mailbox/api/v1/mail
```

| Param | Type | Description |
|-------|------|-------------|
| `status` | int | 2 = read |
| `ids` | int | Mail ID |

---

### Get Group Chat List

```
GET /msg/api/v1/msg/group/chat/list
```

| Param | Type | Description |
|-------|------|-------------|
| `pageNo` | int | Page number |
| `pageSize` | int | Items per page |

---

### Get Group Chat Info

```
GET /msg/api/v1/msg/group/chat/info
```

| Param | Type | Description |
|-------|------|-------------|
| `groupId` | string | Group ID |

---

### Create Group Chat

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

---

### Get Unread Notifications

```
GET /msg/api/v1/user-notification/unread
```

---

### Get Group Join Requests

```
GET /msg/api/v1/msg/group/chat/request/list
```

---

### Get Group Creation Cost

```
GET /msg/api/v1/group/chat/price
```

---

### Edit Group Chat

```
PUT /msg/api/v1/msg/group/chat/modify
```

Body: JSON with fields to update (groupName, groupNotice, inviteStatus, noticePic)

---

### Set Group Member Admin

```
PUT /msg/api/v1/msg/group/chat/member
```

Body: `{ groupId, inviterId, memberIds, operationType }`

---

### Mute Group Member

```
POST /msg/api/v1/msg/group/chat/forbidden/member
```

Body: `{ groupId, memberId, minutes }`

---

### Invite to Group

```
POST /msg/api/v1/msg/group/chat/invite
```

Body: `{ groupId, memberIds }`

---

### Kick from Group

```
PUT /msg/api/v1/msg/group/chat/kickOut
```

Body: `{ groupId, groupName, inviterId, memberIds }`

---

### Approve Group Request

```
PUT /msg/api/v1/msg/group/chat/agreement
```

Body: `{ groupId, operateId, type, userId }`

---

### Quit Group

```
PUT /msg/api/v1/msg/group/chat/quit
```

| Param | Type | Description |
|-------|------|-------------|
| `groupId` | string | Group ID |
| `groupName` | string | Group name |

---

### Apply to Group

```
POST /msg/api/v1/msg/group/chat/apply
```

| Param | Type | Description |
|-------|------|-------------|
| `groupId` | string | Group ID |
| `msg` | string | Join message |

---

### Transfer Group

```
PUT /msg/api/v1/msg/group/chat/transfer
```

Body: `{ groupId, inviterId, userId }`

---

## Activity Endpoints

### Get Activity Home Page

```
GET /activity/api/v1/activity/home/page
```

---

### Get Activity List Groups

```
GET /activity/api/v2/homepage/activity-list/group
```

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

---

### Get Theme Info

```
GET /activity/api/v2/theme
```

---

### Get Newbie Activity Status

```
GET /activity/api/v2/newbie/activity/status
```

---

### Get Holiday Score Info

```
GET /activity/api/v1/holiday-score/info
```

| Param | Type | Description |
|-------|------|-------------|
| `activityId` | string | Activity ID |

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

---

### Get Clap Face Images

```
GET /activity/api/v3/clap/face/image/get
```

| Param | Type | Description |
|-------|------|-------------|
| `ignorePositionType` | int | 1 |

---

### Get Background Image

```
GET /activity/api/v1/background-image
```

---

### Get Game Detail Activity

```
GET /activity/api/v1/homepage/game-detail/icon-activity
```

| Param | Type | Description |
|-------|------|-------------|
| `gameId` | string | Game ID |

---

## Config Endpoints

### Get Config

```
GET /config/files/{configName}
```

Known config names: `blockmods-config-v1`, `blockymods-check-version`, `game_resource_config`, `sharing-game-resource-config`, `deeplink-remote-config`, `game-detail-to-editor`, `app_sticker_config`

---

### Check App Version

```
GET /config/files/blockymods-check-version
```

---

### Get Game Resource Config

```
GET /config/files/game_resource_config
```

---

### Get ML Config

```
GET /config/ml/files/{configName}
```

Known ML configs: `common_lang_img_url_config`, `game_room_def_title_config`, `game_room_tab_config`, `im_fraud_rule_config`, `new_player_guide_for_rbx_config`, `tribe_introduction_banner`

---

### Get Game Room Tab Config

```
GET /config/ml/files/game_room_tab_config
```

---

## Other Endpoints

### Get Server Time

```
GET /server-time
```

---

### Get Comment Window Flag

```
GET /process/api/v1/comment/window/flag
```

| Param | Type | Description |
|-------|------|-------------|
| `userId` | int | User ID |
