# Examples

Real-world code samples for common BlockmanGO API operations.

---

## Basic Setup

```python
from blockmango import BlockmanGOClient
from blockmango.exceptions import APIError, NetworkError

client = BlockmanGOClient(
    user_id=45890736,
    access_token="your_token_here"
)
```

---

## User Profile

```python
# Get basic user info
user = client.get_user_info()
print(f"User: {user.nickName} (ID: {user.userId})")
print(f"VIP Level: {user.vipLevel}")
print(f"Region: {user.region}")

# Get detailed info
details = client.get_user_details()

# Get career stats
career = client.get_user_career()
print(f"Kills: {career.killCount}")
print(f"Win Rate: {career.victoryRate:.1%}")
print(f"Games Played: {career.totalPlayedCount}")
print(f"Total Play Time: {career.totalTime / 3600:.1f} hours")

# Update profile
client.update_user_info(nickName="New Name")

# Set language
client.set_language("en")
```

---

## Friends

```python
# Get all friends
friends = client.get_friends(page_size=100)
print(f"You have {len(friends)} friends")

# Show online friends
for friend in friends:
    status = "Online" if friend.status == 30 else "Offline"
    print(f"  {friend.nickName} — {status}")

# Get specific friend status
friend = client.get_friend_status(user_id=12345)
print(f"Friend status: {'Online' if friend.status == 30 else 'Offline'}")

# Send friend request
client.send_friend_request(friend_id=12345, message="Let's play!")

# Approve pending request
client.approve_friend_request(friend_id=12345)

# Set alias
client.set_friend_alias(friend_id=12345, alias="Best Buddy")

# Like a user
client.add_popularity(friend_id=12345)
pop = client.get_popularity(user_id=12345)
print(f"Popularity: {pop.popularity}")
```

---

## Game Rooms

```python
from blockmango.models import GameMode

# List all rooms
rooms = client.get_game_rooms(mode=GameMode.ALL)
print(f"Total rooms: {len(rooms)}")

# List PVP rooms
pvp_rooms = client.get_game_rooms(mode=GameMode.PVP, page_size=5)
for room in pvp_rooms:
    print(f"{room.name} — {room.curUsers}/{room.maxUsers} players")
    print(f"  Game: {room.gameName} ({room.gameType})")
    print(f"  Server: {room.gameAddr}")

# Recently played
recent = client.get_recently_played_rooms()

# UGC feed
ugc = client.get_random_ugc_feed(page_size=10)
```

---

## Games

```python
# Get game info
bedwars = client.get_game_info("g1008")
print(f"{bedwars.gameTitle} — {bedwars.onlineNumber} online")
print(f"Likes: {bedwars.praiseNumber}")

# Browse game catalog
catalog = client.get_game_catalog(page=0, page_size=20)
for section in catalog:
    if isinstance(section, dict) and "response" in section:
        games = section["response"].get("pageInfo", {}).get("data", [])
        for game in games:
            title = game.get("gameTitle", "Unknown")
            online = game.get("onlineNumber", 0)
            print(f"  {title} — {online} online")

# Get party-enabled games
party_games = client.get_party_enabled_games()
for game in party_games:
    print(f"  {game.gameName} (Party enabled)")

# Get mining token balance
mining = client.get_mining_token_balance()
print(f"Mining tokens: {mining.tokenAmount}")
```

---

## Game Auth & Server Connection

```python
# Get JWT token for game session
auth = client.get_game_auth("g1008", game_version=10114)
print(f"Game server: {auth.dispUrl}")
print(f"Region: {auth.region}")
print(f"Engine: {auth.engineType}")

# Get party auth
party = client.get_party_auth()
print(f"Party service: {party}")

# Get ping servers
servers = client.get_ping_servers()
for server in servers:
    print(f"  Region {server.region}: {server.url}")
```

---

## Decorations

```python
# Get currently equipped decorations
equipped = client.get_equipped_decorations(user_id=45890736)
print(f"Equipped items: {len(equipped)}")
for item in equipped:
    print(f"  {item.name} (ID: {item.id})")

# Browse shop by category
hats = client.browse_shop_decorations("fashi")
print(f"Available hats: {len(hats)}")
for hat in hats:
    print(f"  {hat.name} — {hat.price} diamonds")

# Get recommended decorations
hot_items = client.get_recommended_decorations(decoration_type="hot")
new_items = client.get_recommended_decorations(decoration_type="new")

# Get suit bundles
suits = client.get_suit_decorations()
for suit in suits:
    print(f"  {suit.name} ({len(suit.shopDecorationInfos)} items)")

# Equip a decoration
client.equip_decoration(decoration_id=1234)

# Unequip a decoration
client.unequip_decoration(decoration_id=1234)

# Buy a decoration
result = client.buy_decoration(
    decoration_id=1234,
    diamond=100,
    cloth_voucher=50,
    pay_type="clothVoucher"
)

# Get user decorations by category
hair = client.get_user_decorations_by_category(category="toufa")
print(f"Your hair items: {len(hair)}")

# Get expiring decorations
expiring = client.get_expiring_decorations()
for item in expiring:
    print(f"  Expiring: {item}")
```

---

## Clans

```python
# Get my clan
try:
    clan = client.get_my_clan()
    print(f"Clan: {clan.name} (Level {clan.level})")
    print(f"Members: {clan.currentCount}/{clan.maxCount}")
except APIError:
    print("Not in a clan")

# Get recommended clans
recommended = client.get_recommended_clans()
for clan in recommended:
    print(f"  {clan.name} — {clan.currentCount}/{clan.maxCount} members")

# Create a clan
result = client.create_clan(
    name="My Clan",
    details="A fun clan for everyone",
    tags=["pvp", "friendly"]
)

# Join a clan
client.join_clan(clan_id=12345, message="I'd like to join!")

# Leave a clan
client.leave_clan(clan_id=12345)

# Search clans
results = client.search_clans(clan_name="test")
for clan in results:
    print(f"  {clan}")

# Get clan members
members = client.get_clan_members()
for member in members:
    print(f"  {member}")

# Invite to clan
client.invite_to_clan(friend_ids=[12345, 67890], message="Join us!")

# Get clan tasks
tasks = client.get_clan_tasks()

# Accept a task
client.accept_clan_task(task_id=1, is_team_task=True)

# Claim task reward
client.claim_clan_task(task_id=1, is_team_task=True)
```

---

## Currency & Payments

```python
# Get wealth
wealth = client.get_wealth()
print(f"Diamonds: {wealth.diamonds}")
print(f"Gold: {wealth.golds}")
print(f"G-Diamonds: {wealth.gDiamonds}")

# Get shop info
shop = client.get_user_shop_info()
print(f"Cloth Vouchers: {shop.clothVoucher}")

# Check payment red points
payments = client.get_payment_red_points()
if payments.monthCard:
    print("Month card available!")

# Get growth fund
fund = client.get_growth_fund_info("g1015")
print(f"Growth fund: {fund}")
```

---

## Mail & Messaging

```python
# Check new mail
has_new = client.check_new_mail()
if has_new:
    mails = client.get_mail_list()
    for mail in mails:
        print(f"  {mail}")

# Mark mail as read
client.mark_mail_read(mail_id=12345)

# Get group chats
groups = client.get_group_chat_list(page_size=10)
for group in groups:
    print(f"  {group.groupName} — {len(group.groupMembers)} members")

# Create a group
group = client.create_group_chat(
    group_name="My Group",
    members=[12345, 67890]
)

# Get unread notifications
unread = client.get_unread_notifications()
print(f"Unread: {unread}")
```

---

## Activities & Events

```python
# Get activity home page
activities = client.get_activity_home_page()
print(f"Activities: {activities}")

# Get current theme
theme = client.get_theme_info()
if isinstance(theme, dict) and "name" in theme:
    print(f"Current theme: {theme['name']}")

# Get ad banners
banners = client.get_ad_banner(language="en_US")
for banner in banners:
    print(f"  Banner: {banner}")

# Get holiday score
holiday = client.get_holiday_score_info()
print(f"Holiday event: {holiday}")

# Get newbie status
newbie = client.get_newbie_activity_status()
print(f"Newbie tasks: {newbie}")
```

---

## Configuration

```python
# Check for app updates
update = client.check_app_version()
print(f"Update info: {update}")

# Get game room tabs
tabs = client.get_game_room_tab_config()
print(f"Room tabs: {tabs}")

# Get server time
timestamp = client.get_server_time()
print(f"Server time: {timestamp}")

# Get resource config
config = client.get_game_resource_config()
print(f"Resource config: {config}")
```

---

## Analytics

```python
# Send analytics event
client.send_analytics_event([
    {
        "event": "custom_event",
        "properties": {
            "key": "value"
        },
        "timestamp": 1700000000
    }
])
```

---

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
    if e.data:
        print(f"Additional data: {e.data}")
except AuthenticationError:
    print("Auth failed — check access token")
except RateLimitError:
    print("Rate limited — try again later")
except TokenExpiredError:
    print("Token expired — re-authenticate")
except NetworkError as e:
    print(f"Network error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
finally:
    client.close()
```

---

## Full Example

```python
from blockmango import BlockmanGOClient
from blockmango.models import GameMode
from blockmango.exceptions import APIError

def main():
    with BlockmanGOClient(
        user_id=45890736,
        access_token="your_token"
    ) as client:
        # User info
        user = client.get_user_details()
        print(f"Welcome, {user.nickName}!")

        # Friends
        friends = client.get_friends()
        online = [f for f in friends if f.status == 30]
        print(f"{len(online)}/{len(friends)} friends online")

        # Game rooms
        rooms = client.get_game_rooms(mode=GameMode.PVP)
        print(f"{len(rooms)} PVP rooms available")

        # Wealth
        wealth = client.get_wealth()
        print(f"Diamonds: {wealth.diamonds}, Gold: {wealth.golds}")

        # Decorations
        equipped = client.get_equipped_decorations()
        print(f"Equipped: {len(equipped)} items")

if __name__ == "__main__":
    main()
```
