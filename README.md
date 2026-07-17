# BlockmanGO Changelog Tracker

Reverse-engineered APK changelog tracking for [BlockmanGO](https://play.google.com/store/apps/details?id=com.sandboxol.blockymods) (com.sandboxol.blockymods). Tracks every update by scanning 22 categories of data and generating structured diffs.

> **Latest tracked: v3.21.1** (build 5671) — APK doubled to 200 MB, NetEase anti-tamper wrapper added.

---

## What This Does

```
APK/XAPK → Scan (22 categories) → Compare → Diff Report → Archive
```

Drop in any BlockmanGO APK or XAPK file and get a complete changelog of what changed — new activities, permissions, strings, classes, URLs, IPs, file counts, and size differences.

## Quick Start

```bash
pip install androguard

# Track a new version (auto-compares against previous)
python track_versions.py "Blockman+Go_3.21.1.xapk"

# List all tracked versions
python track_versions.py --list

# Create initial baseline
python track_versions.py --create-baseline "Blockman+Go_2.95.3.xapk"
```

## Tracked Versions

| Version | Build | Size | Activities | Methods | Strings | URLs |
|---------|-------|------|------------|---------|---------|------|
| v3.21.1 | 5671 | 200.7 MB | 235 | 611,470 | 436,176 | 311 |
| v3.20.3 | 5663 | 124.5 MB | 235 | 452,328 | 302,733 | 265 |
| v2.95.3 | 5123 | 94.5 MB | 167 | — | — | — |

---

## Version Diff Reports

### v2.95.3 → v3.21.1

| Metric | Change |
|--------|--------|
| Permissions | +4 / -2 |
| Activities | +68 |
| Services | +1 |
| Strings | +220,200 / -77,112 |
| Files | +3,018 / -959 |
| URLs | +107 |
| IPs | +19 |
| Size | 94.5 MB → 200.7 MB (+106.2 MB) |

#### Manifest Changes

- **Application wrapper**: `com.sandboxol.blockymods.App` → `com.netease.nis.wrapper.MyApplication` (NetEase anti-tamper)
- **Min SDK**: 21 → 23
- **Target SDK**: 34 → 35
- **Build**: 5123 → 5671

#### New Permissions

- `FOREGROUND_SERVICE_MEDIA_PROJECTION` — screen recording/casting
- `WRITE_INTERNAL_STORAGE` — expanded storage access
- `com.huawei.appmarket.service.commondata.permission.GET_COMMON_DATA` — Huawei AppGallery integration
- `com.samsung.android.mapsagent.permission.READ_APP_INFO` — Samsung device integration

#### Removed Permissions

- `READ_MEDIA_IMAGES`
- `READ_MEDIA_VIDEO`

#### New Activities (+68)

Major additions:
- **Mining system**: `MiningActivity`, `MinerActivity`, `MiningStoreActivity`, `GameMiningStoreActivity`
- **Personal Space**: `PersonalSpaceActivity`, `PersonalDressActivity`, `PersonalEffectsActivity`
- **Payment/Top-up**: `GameOfficialTopUpActivity`, `OfficialTopUpActivity`, `WebViewPaymentActivity`
- **Events**: `WorldCup2026HomeActivity`, `Christmas25Activity`, `Halloween25Activity`, `Ann8CelebrationActivity`
- **Game Room**: `GameRoomDetailActivity`, `GameCreateEditModeActivityActivity`, `ArchiveManagerActivity`
- **Social**: `PassCardActivity`, `BlindBoxActivity`, `ComebackActivity`, `HolidayScoreActivity`
- **Ad SDKs**: Chartboost, InMobi, TradPlus, Bigo, MBridge, Unity IronSource
- **RongCloud messaging**: 20+ new friend/group management activities

#### New Services

- `VideoRepositoryDownloadService` (Chartboost video caching)

#### New Receivers

- `ProfileInstallReceiver` (AndroidX baseline profile)
- `NetWorkChangeReceiver` (MBridge network state)

#### New Providers

- `LevelPlayActivityLifecycleProvider` (IronSource)
- `MBComponentLifecycleProvider` (MBridge)
- `PicassoProvider` (image loading)
- `AppLaunchMonitorInstaller` (Tencent monitoring)

#### Notable New URLs

- `https://points.blockmango.com` — new points/rewards system
- `https://www.blockmango.com` — domain migration from `.net` to `.com`
- `https://discord.gg/Y4ZfuxGdh9` — official Discord server
- `https://android.bugly.tencent.com` — Tencent crash reporting
- Various Chartboost, InMobi, TradPlus ad SDK endpoints
- `https://ca.turing.captcha.qcloud.com` — Tencent CAPTCHA

---

## What Gets Scanned

Every APK is analyzed across 22 categories:

| # | Category | What's Extracted |
|---|----------|-----------------|
| 1 | Manifest | Package, version, SDK targets, 30+ attributes |
| 2 | Permissions | Declared, requested, dangerous, custom |
| 3 | Activities | All activities with intent filters |
| 4 | Services | All services |
| 5 | Receivers | All broadcast receivers |
| 6 | Providers | All content providers |
| 7 | Intent Filters | By action/scheme/category, deeplinks |
| 8 | Meta-data | All manifest meta-data |
| 9 | Strings | XML resources, DEX bytecode, native .so, assets |
| 10 | Resources | Full file inventory, categorized |
| 11 | Resource IDs | resources.arsc analysis |
| 12 | Smali Classes | Package hierarchy, class counts |
| 13 | Classes | Full class hierarchy (androguard) |
| 14 | Methods | Total / internal / external counts |
| 15 | Fields | Total / internal / external counts |
| 16 | Native Libs | .so files by architecture |
| 17 | Assets | JSON parsing, text file analysis |
| 18 | DEX Files | Per-DEX string/type/class/method counts |
| 19 | Certificates | Signing info |
| 20 | File Listing | Complete tree with sizes |
| 21 | URLs | All discovered endpoints |
| 22 | IPs | All IP addresses |

Special extractions from strings:
- **URLs** (http/https endpoints)
- **IP addresses** (server infrastructure)
- **Package references** (com.*, net.*, org.*)

---

## Output Structure

Each version produces a snapshot in `versions/`:

```
versions/v3_21_1_build5671/
├── full_scan.json          # Complete scan (all 22 categories)
├── manifest.json
├── permissions.json
├── activities.json
├── services.json
├── receivers.json
├── providers.json
├── intent_filters.json
├── meta_data.json
├── strings.json            # Extracted strings (URLs, IPs, etc.)
├── resources.json
├── resource_ids.json
├── smali.json
├── classes.json
├── methods.json
├── fields.json
├── native_libs.json
├── assets.json
├── dex_files.json
├── certificates.json
├── file_listing.json
└── sizes.json
```

Diff reports go in `reports/`:

```
reports/
├── v2.95.3_to_v3.21.1.md      # Human-readable
└── v2.95.3_to_v3.21.1.json    # Machine-readable
```

Archived snapshots go in `archives/`:

```
archives/
└── blockmango_v3.21.1_build5671.zip
```

---

## API Reference

Full documentation of the BlockmanGO REST API (100+ endpoints).

### Hosts

| Host | Purpose |
|------|---------|
| `gw.sandboxol.com` | Main API gateway |
| `di.sandboxol.com` | Analytics/tracking |
| `staticgs.sandboxol.com` | CDN static assets |
| `static.sandboxol.com` | CDN static assets |

### Authentication

All requests require signed headers:

| Header | Description |
|--------|-------------|
| `userid` | Numeric user ID |
| `access-token` | Session token |
| `userdeviceid` | Device fingerprint |
| `x-apikey` | API key |
| `x-nonce` | Random 32-char hex |
| `x-time` | Unix timestamp |
| `x-sign` | MD5 signature |
| `x-urlpath` | Request path |

### Response Format

```json
{
  "code": 1,
  "message": "SUCCESS",
  "data": {}
}
```

### API Modules

| Module | Endpoints | Description |
|--------|-----------|-------------|
| Game | `/game/api/v3/*` | Auth, catalog, rooms, resources |
| User | `/user/api/v1/*` | Profile, career, VIP, upload |
| Friend | `/friend/api/v3/*` | List, requests, popularity |
| Clan | `/clan/api/v3/*` | Info, create, join, tasks |
| Decoration | `/decoration/api/v1/*` | Equip, browse, buy |
| Payment | `/pay/api/v1/*` | Wealth, billing, funds |
| Mail | `/mailbox/api/v1/*` | Mail, group chat |
| Activity | `/activity/api/v1/*` | Events, themes, ads |
| Config | `/config/files/*` | Version check, resources |

### Game IDs

| ID | Game | Engine |
|----|------|--------|
| g1008 | Bed Wars | v1 (10114) |
| g1048 | Sky Block | v1 (10114) |
| g1018 | Egg War | v1 (10114) |
| g1025 | Hero Tycoon 2 | v1 |
| g2052 | Free City RP | v2 (20084) |
| g2066 | Social Hall | v2 (20084) |

68 games total. See [Game Reference](apidocs/games.md) for the full list.

### Security Findings

- No certificate pinning
- MD5-based request signing (cryptographically broken)
- Static API keys that never rotate
- User IDs exposed as plain integers
- Game server IPs leaked in auth responses
- Analytics sent over HTTP (not HTTPS)

See [Security Notes](apidocs/security.md) for details.

---

## Requirements

```bash
pip install androguard
```

---

## Full Documentation

| Document | Description |
|----------|-------------|
| [API Getting Started](apidocs/getting-started.md) | Installation, setup, quick start |
| [Authentication](apidocs/authentication.md) | API keys, signing, tokens |
| [API Reference](apidocs/api-reference.md) | All 100+ endpoints |
| [Data Models](apidocs/models.md) | Request/response structures |
| [Game Reference](apidocs/games.md) | 68 game IDs, server infrastructure |
| [Security Notes](apidocs/security.md) | Documented vulnerabilities |
| [Examples](apidocs/examples.md) | Code samples |
| [Changelog README](changelog/README.md) | Tracker tool documentation |

---

## How It Works

1. **Extract** — Unpack APK/XAPK, parse manifest, extract DEX/assets/resources
2. **Scan** — Analyze 22 categories using androguard (bytecode analysis)
3. **Compare** — Set operations (added/removed/changed) across all categories
4. **Report** — Generate markdown + JSON diff reports
5. **Archive** — Zip each version snapshot for archival

---

## Disclaimer

This project is for educational and research purposes only. It is not affiliated with BlockmanGO, SandboxOL, or NetEase. Use at your own risk. Do not use this tool to violate the game's Terms of Service.

---

## License

MIT
