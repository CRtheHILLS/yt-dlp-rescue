<div align="center">

# 🚑 yt-dlp-rescue

### *YouTube broke your downloads. We fixed it.*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![yt-dlp](https://img.shields.io/badge/yt--dlp-2024.12+-red.svg)](https://github.com/yt-dlp/yt-dlp)
[![YouTube](https://img.shields.io/badge/YouTube-SABR%20Fix-FF0000?logo=youtube)](https://github.com/yt-dlp/yt-dlp/issues)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/crkim/yt-dlp-rescue/pulls)

<br>

> **Since late 2024, YouTube forced SABR streaming.**
> No matter what quality you pick, yt-dlp only downloads **360p**.
> This repo is the battle-tested fix.

<br>

[Quick Fix](#-quick-fix) · [Why It Happens](#-why-it-happens) · [All Commands](#-all-commands) · [Config File](#-permanent-config) · [Troubleshooting](#-troubleshooting)

</div>

---

<br>

## 🎬 Before vs After

<table>
<tr>
<td width="50%">

### ❌ Before (Broken)
```
$ yt-dlp -f "bestvideo+bestaudio" URL

[youtube] Extracting URL...
[download] 100% of 9.2MiB

# Requested 1080p → Got 360p 😤
# Requested 720p  → Got 360p 😤
# Requested 4K    → Got 360p 😤
```

**Every quality = same 9MB file (360p)**

</td>
<td width="50%">

### ✅ After (Fixed)
```
$ yt-dlp --extractor-args \
  "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" \
  -S "res:1080" URL

[download] 100% of 34.1MiB  ← Real 1080p! 🎉
```

**Each quality = correct file size**

</td>
</tr>
</table>

<br>

## 📊 Proof It Works

> Tested with the same video, same URL, different quality settings:

```
┌──────────┬───────────┬────────────────────────────────────────┐
│ Quality  │ File Size │ Visual                                 │
├──────────┼───────────┼────────────────────────────────────────┤
│ 360p     │    9 MB   │ ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ 480p     │   15 MB   │ ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ 720p     │   21 MB   │ ██████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ 1080p    │   34 MB   │ █████████░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ 4K       │  244 MB   │ ████████████████████████████████████░░ │
└──────────┴───────────┴────────────────────────────────────────┘
  Before fix: ALL qualities → 9 MB (360p) 💀
  After fix:  Each quality → correct size ✅
```

<br>

---

<br>

## ⚡ Quick Fix

Just add **two flags** to your yt-dlp command:

```bash
yt-dlp \
  --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \  # ← Flag 1: Use working clients
  -f "bestvideo*+bestaudio/best" \
  -S "res:1080" \                                                          # ← Flag 2: Sort by resolution
  --merge-output-format mp4 \
  "https://youtube.com/watch?v=VIDEO_ID"
```

| Flag | What it does |
|:-----|:-------------|
| `player_client=web,android_vr,tv_downgraded` | Bypasses SABR — gets full format list (144p~4K) |
| `-S "res:1080"` | Picks best format matching your target resolution |
| `-f "bestvideo*+bestaudio/best"` | The `*` adds progressive fallback for reliability |

> 💡 **That's literally it.** The rest of this README explains *why* it works and gives you convenience shortcuts.

<br>

---

<br>

## 🔬 Why It Happens

<details>
<summary><b>📖 Click to expand the full technical explanation</b></summary>

<br>

### The Root Cause: YouTube's SABR Migration

Starting in late 2024, YouTube began forcing **SABR (Server ABR)** on the `web` client. This means:

```
                    ┌─────────────────────────────────────────┐
                    │         YouTube's SABR Change           │
                    ├─────────────────────────────────────────┤
                    │                                         │
  Before (2024):    │  web client → 20+ DASH formats          │
                    │               (144p, 240p, ... 4K)      │
                    │                                         │
  After (2025+):    │  web client → 1 progressive format      │
                    │               (360p only!)               │
                    │                                         │
                    └─────────────────────────────────────────┘
```

### Why format IDs (`-f 137+140`) break

```
  ┌──────────────────┐          ┌──────────────────┐
  │   --dump-json    │          │  Actual Download  │
  │   says: 137      │  ═══╳═▶ │  137 not found!   │
  │   available      │          │  (SABR replaced)  │
  └──────────────────┘          └──────────────────┘
        IDs CHANGE between requests.
        YouTube A/B tests swap formats.
        SABR may list but block downloads.
```

**Solution:** Use `-S` (sort) instead — yt-dlp picks the best match **at download time**.

### Which clients still work?

| Client | DASH Formats | Quality Range | Token Needed |
|:-------|:-------------|:--------------|:-------------|
| `web` *(default)* | ❌ SABR only | 360p | PO Token |
| `android_vr` | ✅ Full list | 144p ~ 4K | None |
| `tv_downgraded` | ✅ Full list | 144p ~ 4K | None |
| `web_embedded` | ✅ Full list | 144p ~ 4K | None |

By combining multiple clients, yt-dlp gets the full format catalog.

</details>

<br>

---

<br>

## 📋 All Commands

### 🎥 Video Download

```bash
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#  Just change the number after "res:" !
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# 📺 4K (best available)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" --merge-output-format mp4 "URL"

# 🖥️ 1080p (Full HD)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" -S "res:1080" --merge-output-format mp4 "URL"

# 💻 720p (HD)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" -S "res:720" --merge-output-format mp4 "URL"

# 📱 480p (SD)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" -S "res:480" --merge-output-format mp4 "URL"

# 📱 360p (Low)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" -S "res:360" --merge-output-format mp4 "URL"
```

### 🎵 Audio Only

```bash
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -x --audio-format mp3 --audio-quality 192 "URL"
```

### 🔍 List Available Formats

```bash
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -F "URL"
```

<br>

---

<br>

## 💾 Permanent Config

> *Tired of typing long commands?* Save this once, use forever.

Create `~/.config/yt-dlp/config`:

```ini
# ─── yt-dlp-rescue config ───────────────────
# Fix YouTube SABR quality selection
--extractor-args youtube:player_client=web,android_vr,tv_downgraded

# Best video + audio with progressive fallback
-f bestvideo*+bestaudio/best

# Output format
--merge-output-format mp4

# Reliability
--retries 3
--fragment-retries 3
--socket-timeout 30
# ─────────────────────────────────────────────
```

Now just run:
```bash
yt-dlp -S "res:1080" "URL"   # That's it! 🎉
```

<br>

---

<br>

## 🩹 Troubleshooting

<details>
<summary><b>😤 Still getting 360p</b></summary>

```bash
# 1. Clear stale cache
yt-dlp --rm-cache-dir

# 2. Update yt-dlp
pip install -U yt-dlp

# 3. Try different client order
yt-dlp --extractor-args "youtube:player_client=tv_downgraded,android_vr,web" \
  -f "bestvideo*+bestaudio/best" -S "res:1080" "URL"
```
</details>

<details>
<summary><b>🤖 "Sign in to confirm you're not a bot"</b></summary>

```bash
# Use cookies from your browser
yt-dlp --cookies-from-browser chrome "URL"

# Or export cookies manually
yt-dlp --cookies cookies.txt "URL"
```
</details>

<details>
<summary><b>🔧 ffmpeg not found</b></summary>

```bash
# macOS
brew install ffmpeg

# Ubuntu / Debian
sudo apt install ffmpeg

# Windows
winget install ffmpeg
```

> Without ffmpeg, you're limited to progressive formats (combined video+audio files). Install ffmpeg for the full quality range.
</details>

<details>
<summary><b>💡 What does the <code>*</code> in <code>bestvideo*</code> mean?</b></summary>

```
-f "bestvideo*+bestaudio/best"
             ▲
             └── This asterisk = safety net!

  bestvideo   →  video-only streams
  bestvideo*  →  video-only + progressive (video+audio) streams

If DASH merge fails, the * lets yt-dlp fall back to
a single-file format instead of failing completely.
```
</details>

<br>

---

<br>

<div align="center">

## 🤝 Contributing

YouTube changes things constantly.
Found a better fix? Something stopped working?

**[Open an Issue](../../issues) · [Send a PR](../../pulls)**

<br>

---

<br>

### If this rescued your downloads, leave a ⭐

*It helps others find this fix too.*

<br>

Made with ☕ and frustration by [@crkim](https://github.com/crkim)

</div>
