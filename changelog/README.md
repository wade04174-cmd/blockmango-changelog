# BlockmanGO APK Changelog Tracker

Exhaustive APK scanning, versioning, and diff system for BlockmanGO (com.sandboxol.blockymods).

---

## How It Works

1. Drop in an APK or XAPK file
2. Scanner extracts **everything** — 22 categories of data into structured JSON
3. Compares against the previous version automatically
4. Outputs markdown + JSON diff reports
5. Zips each version snapshot for archival

---

## Quick Start

```bash
# Install dependency
pip install androguard

# First run — creates baseline + zip archive
python track_versions.py "C:\path\to\Blockman+Go_2.95.3_APKPure.xapk"

# Future versions — auto-compares against previous
python track_versions.py "C:\path\to\Blockman+Go_3.21.0_APKPure.xapk"

# List all tracked versions
python track_versions.py --list
```

That's it. The scanner handles APK and XAPK files natively.

---

## What Gets Scanned

Every APK is exhaustively analyzed across 22 categories:

| # | Category | What's Extracted |
|---|----------|-----------------|
| 1 | Metadata | Filename, size, MD5/SHA1/SHA256 hashes, source type |
| 2 | Manifest | Package, version, SDK targets, 30+ manifest attributes |
| 3 | Permissions | Declared, requested, dangerous, custom — full categorization |
| 4 | Activities | All activities, main launcher, intent filters |
| 5 | Services | All services, intent filters |
| 6 | Receivers | All broadcast receivers, intent filters |
| 7 | Providers | All content providers |
| 8 | Intent Filters | Grouped by action/scheme/category, deeplink detection |
| 9 | Meta-data | All meta-data entries from manifest |
| 10 | Strings | 4 sources: XML resources, DEX bytecode, native .so, asset files |
| 11 | Resources | Full file inventory, categorized by type and directory |
| 12 | Resource IDs | resources.arsc analysis, string resource mappings |
| 13 | Smali Classes | Every class with package, method/field counts, source file, type flags |
| 14 | Classes | Full class hierarchy from androguard |
| 15 | Methods | Total / internal / external method counts |
| 16 | Fields | Total / internal / external field counts |
| 17 | Native Libs | .so files grouped by architecture, size, compression status |
| 18 | Assets | JSON parsing, text file analysis, file type breakdown |
| 19 | DEX Files | Per-DEX: string/type/class/method counts, MD5 hashes |
| 20 | Certificates | Signing cert files, signature hashes |
| 21 | File Listing | Complete file tree with sizes and compression ratios |
| 22 | Sizes | Disk usage breakdown by top-level directory |

Special extractions from strings:
- **URLs** (http/https endpoints)
- **IP addresses** (server infrastructure)
- **Package references** (com.*, net.*, org.*)

---

## Output Structure

```
changelog/
├── scanner.py              # Scanning engine
├── scanner_ext.py          # Scanning engine (part 2)
├── diff_engine.py          # Comparison + report generation
├── compare_apk.py          # CLI: scan + compare
├── track_versions.py       # CLI: scan + compare + save + zip
│
├── versions/               # Per-version scan snapshots
│   ├── v2_95_3_buildXXX/
│   │   ├── full_scan.json  # Complete scan (all 22 categories)
│   │   ├── manifest.json   # Individual sections
│   │   ├── permissions.json
│   │   ├── activities.json
│   │   ├── strings.json
│   │   ├── smali.json
│   │   └── ... (one per category)
│   └── v3_20_3_build5663/
│       └── ...
│
├── archives/               # Zipped snapshots
│   ├── blockmango_v2.95.3_buildXXX.zip
│   └── blockmango_v3.20.3_build5663.zip
│
└── reports/                # Diff reports
    ├── v2.95.3_to_v3.20.3.md     # Human-readable
    └── v2.95.3_to_v3.20.3.json   # Machine-readable
```

---

## Version Snapshots

Each version produces a `full_scan.json` with this structure:

```json
{
  "_meta": {
    "scan_date": "2026-07-17T12:00:00",
    "apk_size_mb": 94.5,
    "md5": "...",
    "sha256": "...",
    "source_type": "xapk"
  },
  "manifest": {
    "package": "com.sandboxol.blockymods",
    "version_name": "2.95.3",
    "version_code": 395,
    "min_sdk": 21,
    "target_sdk": 33
  },
  "permissions": {
    "dangerous": ["android.permission.CAMERA", ...],
    "normal": [...],
    "custom": [...]
  },
  "activities": { "list": [...], "count": 5, "main_launcher": [...] },
  "services": { "list": [...], "count": 3 },
  "receivers": { "list": [...], "count": 7 },
  "strings": {
    "urls": ["https://gw.sandboxol.com/...", ...],
    "ip_addresses": ["43.166.142.211", ...],
    "counts": { "total_unique": 45000 }
  },
  "smali": {
    "class_count": 8500,
    "packages": { "com/sandboxol/blockymods": 1200, ... },
    "classes": [{ "name": "...", "method_count": 15, ... }]
  },
  "methods": { "count": 95000, "internal": 72000, "external": 23000 },
  "native_libs": { "abis": ["arm64-v8a", "armeabi-v7a"], ... },
  "assets": { "json_files": {...}, "count": 350 },
  "_summary": { ... }
}
```

---

## Diff Reports

When comparing two versions, reports show every change:

### Markdown (human-readable)
```markdown
# BlockmanGO Changelog: v2.95.3 → v3.20.3

## Summary
| Metric | Change |
|--------|--------|
| Permissions | +3 / -0 |
| Activities | +2 / -1 |
| Classes | +1200 / -340 |
| Strings | +4500 / -890 |

## Permissions Added
+ `android.permission.POST_NOTIFICATIONS` ⚠️ DANGEROUS
+ `android.permission.READ_MEDIA_IMAGES` ⚠️ DANGEROUS
+ `android.permission.READ_MEDIA_VIDEO` ⚠️ DANGEROUS

## Classes Added (+1200)
+ `com/sandboxol/blockymods/features/NewFeatureManager`
+ `com/sandboxol/blockymods/ui/NewActivity`
...

## URLs Added
+ `https://gw.sandboxol.com/game/api/v5/new/endpoint`
```

### JSON (machine-readable)
Same data structured for programmatic analysis.

---

## CLI Commands

```bash
# Scan only (no save, no compare)
python compare_apk.py app.apk

# Scan + save snapshot
python compare_apk.py app.apk --save

# Scan + compare against latest saved version
python compare_apk.py app.apk --compare-latest

# Scan + save + compare
python compare_apk.py app.apk --save --compare-latest

# Full pipeline: scan + compare + save + zip
python track_versions.py app.apk

# Create baseline (first version)
python track_versions.py --create-baseline app.apk

# List all tracked versions
python track_versions.py --list

# Compare against specific old version
python compare_apk.py new.apk --old versions/v2_95_3_buildXXX

# Output diff as JSON
python compare_apk.py new.apk --compare-latest --json

# Skip zip creation
python track_versions.py app.apk --no-zip
```

---

## Python API

```python
from scanner import APKScanner
from diff_engine import compare_scans, generate_markdown_report

# Scan any APK or XAPK
scanner = APKScanner("path/to/app.apk")  # or .xapk
result = scanner.scan_full()

# Access any category
print(result["manifest"]["version_name"])
print(result["strings"]["urls"])
print(result["smali"]["class_count"])

# Save snapshot
scanner.save("versions/v1.0.0")

# Compare two scans
diff = compare_scans(old_scan, new_scan)
report = generate_markdown_report(diff)
```

---

## Supported Formats

| Format | Support |
|--------|---------|
| `.apk` | Direct scan |
| `.xapk` | Auto-extracts main APK, scans it, preserves XAPK metadata |

---

## Requirements

```bash
pip install androguard
```

Optional for APK download tracking:
```bash
pip install requests
```
