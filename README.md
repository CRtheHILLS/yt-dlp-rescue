<div align="center">

<img src="https://img.shields.io/badge/yt--dlp-rescue-FF0000?style=for-the-badge&logo=youtube&logoColor=white" alt="yt-dlp-rescue" />

# yt-dlp-rescue

### *YouTube broke your downloads. We fixed it. Permanently.*

<br>

[![Version](https://img.shields.io/badge/version-2.0.0-blue?style=flat-square)](https://github.com/bravomylife-lab/yt-dlp-rescue/releases)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![yt-dlp](https://img.shields.io/badge/yt--dlp-2025.01+-red.svg?style=flat-square)](https://github.com/yt-dlp/yt-dlp)
[![YouTube](https://img.shields.io/badge/YouTube-SABR%20Fix-FF0000?style=flat-square&logo=youtube)](https://github.com/yt-dlp/yt-dlp/issues)
[![Bot Bypass](https://img.shields.io/badge/Bot%20Detection-Bypassed-brightgreen?style=flat-square&logo=shield)](#-bot-detection-bypass)
[![Cloud Ready](https://img.shields.io/badge/Cloud-Ready-0078D4?style=flat-square&logo=microsoftazure)](#%EF%B8%8F-cloudserver-deployment)
[![Node.js](https://img.shields.io/badge/Node.js-Required-339933?style=flat-square&logo=nodedotjs&logoColor=white)](#-bot-detection-bypass)
[![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Docker](https://img.shields.io/badge/Docker-Supported-2496ED?style=flat-square&logo=docker&logoColor=white)](#%EF%B8%8F-cloudserver-deployment)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](../../pulls)
[![Stars](https://img.shields.io/github/stars/bravomylife-lab/yt-dlp-rescue?style=flat-square&color=gold)](../../stargazers)

<br>

```
  _   _          _ _
 | | | |        | | |
 | |_| |_ _____ | | |_ ____     ____ ___  ___  _____ _ _  ___
 |_   _) ___ | | | |  _ \   / ___) _ \/ __|/ ___ | | |/ _ \
   | |_| ____|| | | | |_) ) | |  |  __/\__ \ (___| |_|  __/
    \__)_____) \_)_)  __/  |_|   \___||___/\____)____/\___|
                    |_|     v2.0 - Battle Tested
```

<br>

> **YouTube broke downloads in two ways:**
>
> **Problem 1** &#x1F4C9; **SABR streaming** (2024~) — forces 360p regardless of quality setting
>
> **Problem 2** &#x1F6AB; **Bot detection** (2025~) — blocks datacenter/cloud IPs entirely
>
> **This repo fixes both. Battle-tested in production serving real users.**

<br>

[&#x26A1; Quick Fix](#-quick-fix) &nbsp;&bull;&nbsp; [&#x1F916; Bot Bypass](#-bot-detection-bypass) &nbsp;&bull;&nbsp; [&#x2601;&#xFE0F; Cloud Deploy](#%EF%B8%8F-cloudserver-deployment) &nbsp;&bull;&nbsp; [&#x1F4CB; Commands](#-all-commands) &nbsp;&bull;&nbsp; [&#x1F4BE; Config](#-permanent-config) &nbsp;&bull;&nbsp; [&#x1F6E0;&#xFE0F; Troubleshoot](#%EF%B8%8F-troubleshooting)

</div>

---

<br>

## &#x1F3AC; Before vs After

<table>
<tr>
<td width="50%">

### &#x274C; Before (Broken)
```
$ yt-dlp -f "bestvideo+bestaudio" URL

[youtube] Extracting URL...
[download] 100% of 9.2MiB

# Requested 1080p -> Got 360p
# Requested 720p  -> Got 360p
# Requested 4K    -> Got 360p
```

**Every quality = same 9MB file (360p)** &#x1F480;

</td>
<td width="50%">

### &#x2705; After (Fixed)
```
$ yt-dlp --extractor-args \
  "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" \
  -S "res:1080" URL

[download] 100% of 34.1MiB  <- Real 1080p!
```

**Each quality = correct file size** &#x1F389;

</td>
</tr>
</table>

<br>

## &#x1F4CA; Proof It Works

> Tested with the same video, same URL, different quality settings:

```
  Quality    File Size    Visualization
  -------    ---------    ----------------------------------------
  360p          9 MB      ##
  480p         15 MB      ####
  720p         21 MB      ######
  1080p        34 MB      #########
  4K          244 MB      ##################################

  Before fix: ALL qualities -> 9 MB (360p)  [BROKEN]
  After fix:  Each quality  -> correct size [FIXED]
```

<br>

---

<br>

## &#x26A1; Quick Fix

> **Copy-paste this. It just works.**

```bash
yt-dlp \
  --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" \
  -S "res:1080" \
  --merge-output-format mp4 \
  "https://youtube.com/watch?v=VIDEO_ID"
```

| Flag | What it does |
|:-----|:-------------|
| `player_client=web,android_vr,tv_downgraded` | Bypasses SABR — gets full format list (144p~4K) |
| `-S "res:1080"` | Picks best format matching your target resolution |
| `-f "bestvideo*+bestaudio/best"` | The `*` adds progressive fallback for reliability |

> &#x1F4A1; **That's literally it.** The rest of this README covers bot detection, cloud deployment, and advanced usage.

<br>

---

<br>

## &#x1F52C; Why It Happens

<details>
<summary><b>&#x1F4D6; Click to expand the full technical explanation</b></summary>

<br>

### The Root Cause: YouTube's SABR Migration

Starting in late 2024, YouTube forced **SABR (Server ABR)** on the `web` client:

```
  +---------------------------------------------+
  |         YouTube's SABR Change               |
  +---------------------------------------------+
  |                                             |
  |  Before (2024):                             |
  |    web client -> 20+ DASH formats           |
  |                  (144p, 240p, ... 4K)       |
  |                                             |
  |  After (2025+):                             |
  |    web client -> 1 progressive format       |
  |                  (360p only!)               |
  |                                             |
  +---------------------------------------------+
```

### Why format IDs (`-f 137+140`) break

```
  +------------------+          +------------------+
  |   --dump-json    |          |  Actual Download  |
  |   says: 137      |  ==X==> |  137 not found!   |
  |   available      |          |  (SABR replaced)  |
  +------------------+          +------------------+

  - IDs CHANGE between requests
  - YouTube A/B tests swap formats
  - SABR may list but block downloads
```

**Solution:** Use `-S` (sort) instead — yt-dlp picks the best match **at download time**.

### Which clients work? (Updated March 2026)

| Client | Status | Quality Range | Token Needed | Notes |
|:-------|:-------|:--------------|:-------------|:------|
| `web` *(default)* | &#x274C; SABR only | 360p | PO Token | Broken for quality selection |
| `android_vr` | &#x2705; Full DASH | 144p ~ 4K | None | May intermittently return 360p only |
| `tv_downgraded` | &#x2705; Full DASH | 144p ~ 4K | None | **Most reliable fallback** |
| `web_creator` | &#x2705; Full DASH | 144p ~ 4K | None | YouTube Studio client |
| `mweb` | &#x2705; Full DASH | 144p ~ 4K | None | Mobile web client |
| `web_embedded` | &#x2705; Full DASH | 144p ~ 4K | None | Embedded player client |

By combining multiple clients, yt-dlp gets the full format catalog from whichever responds.

</details>

<br>

---

<br>

## &#x1F916; Bot Detection Bypass

> &#x1F6A8; **New in 2025-2026:** YouTube aggressively blocks datacenter IPs and suspicious requests with *"Sign in to confirm you're not a bot"* errors.

<br>

### &#x1F504; Strategy 1: Client Rotation (Free, No Setup)

If one client gets blocked, rotate to another:

```bash
# Primary strategy
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" URL

# Blocked? Try these alternatives in order:
yt-dlp --extractor-args "youtube:player_client=tv_downgraded,web,android_vr" URL
yt-dlp --extractor-args "youtube:player_client=android_vr,tv_downgraded,web" URL
yt-dlp --extractor-args "youtube:player_client=web_creator,tv_downgraded,android_vr" URL
yt-dlp --extractor-args "youtube:player_client=mweb,tv_downgraded,web" URL
```

### &#x1F510; Strategy 2: PO Token Auto-Generation (THE KEY FIX)

> &#x1F3AF; **This is the #1 fix that actually solved our production issues.** Without proper PO Token setup, datacenter servers get blocked even with client rotation.

PO Tokens (Proof-of-Origin) prove to YouTube that you're not a bot. There are two modes:

**Mode A: Plugin Only (for personal use)**

```bash
# Install the PO Token provider plugin
pip install bgutil-ytdlp-pot-provider

# Use with yt-dlp (Node.js required)
yt-dlp --js-runtimes node \
  --remote-components ejs:github \
  --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  URL
```

**Mode B: Dedicated PO Token Server (for cloud/production - RECOMMENDED)**

```bash
# 1. Clone and build the PO Token server
git clone https://github.com/Brainicism/bgutil-ytdlp-pot-provider.git
cd bgutil-ytdlp-pot-provider/server
npm ci && npx tsc

# 2. Start the server (runs on port 4416)
node build/main.js &

# 3. CRITICAL: Set this env var so yt-dlp connects to the server
export YT_DLP_POT_PROVIDER_URL="http://127.0.0.1:4416"

# 4. Now yt-dlp automatically uses the PO Token server
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" URL
```

> &#x1F6A8; **Common mistake:** Running the PO Token server WITHOUT setting `YT_DLP_POT_PROVIDER_URL`. The server runs fine but yt-dlp doesn't know it exists! This single env var was the fix that solved our production bot-blocking issues.

### &#x1F9F9; Strategy 3: Clear Cache Regularly

YouTube's challenge solver responses go stale:

```bash
# Clear cache when downloads start failing
yt-dlp --rm-cache-dir

# Then retry
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" URL
```

### &#x1F36A; Strategy 4: Browser Cookies (Last Resort)

```bash
# Auto-extract from browser (easiest)
yt-dlp --cookies-from-browser chrome URL
yt-dlp --cookies-from-browser firefox URL

# Or export manually with a browser extension
yt-dlp --cookies cookies.txt URL
```

> &#x26A0;&#xFE0F; Cookies expire and need periodic re-export. The strategies above are preferred for automation.

<br>

---

<br>

## &#x2601;&#xFE0F; Cloud/Server Deployment

> Running yt-dlp on **Railway, Heroku, AWS, GCP, or any cloud server**? YouTube blocks datacenter IPs at the network level. Here's how to fix it permanently.

### &#x26A0;&#xFE0F; The Problem

```
  Your Server (Datacenter IP)
         |
         v
  +-------------------+
  |     YouTube       |
  |                   |
  |  IP Reputation DB |-----> BLOCKED
  |  "Datacenter IP   |
  |   detected"       |
  +-------------------+
```

### &#x2705; The Solution: Cloudflare WARP Proxy

Route yt-dlp through Cloudflare's network. YouTube doesn't block Cloudflare IPs.

**Using wireproxy (userspace WireGuard, no root/NET_ADMIN needed):**

```bash
# 1. Install wgcf (WARP credential generator)
curl -fsSL -o /usr/local/bin/wgcf \
  "https://github.com/ViRb3/wgcf/releases/download/v2.2.22/wgcf_2.2.22_linux_amd64"
chmod +x /usr/local/bin/wgcf

# 2. Register free WARP account + generate WireGuard config
wgcf register --accept-tos
wgcf generate

# 3. Download wireproxy
# https://github.com/pufferffish/wireproxy/releases

# 4. Add SOCKS5 section to config
cat >> wgcf-profile.conf << 'EOF'

[Socks5]
BindAddress = 127.0.0.1:1080
EOF

# 5. Start proxy + use with yt-dlp
wireproxy -c wgcf-profile.conf &
yt-dlp --proxy socks5://127.0.0.1:1080 URL
```

**Using Docker (one-liner):**

```bash
# Pre-built WARP proxy images
docker run -d -p 1080:1080 ghcr.io/kingcc/warproxy:latest
# or
docker run -d -p 9091:9091 ghcr.io/mon-ius/docker-warp-socks:v5

# Then use with yt-dlp
yt-dlp --proxy socks5://127.0.0.1:1080 URL
```

<br>

### &#x1F3D7;&#xFE0F; Production Architecture

> This is the exact architecture running in production, serving real users worldwide.

```
  +-------------------------------------------------------+
  |                 Docker Container                       |
  |                                                        |
  |   +------------------+    +--------------------+       |
  |   | bgutil PO Token  |--->| HTTP :4416         |       |
  |   | Server (Node.js) |    | Auto-generates     |       |
  |   |                  |    | YouTube auth tokens |       |
  |   +------------------+    +--------------------+       |
  |         |                          |                   |
  |         |  YT_DLP_POT_PROVIDER_URL |                   |
  |         |  = http://127.0.0.1:4416 |                   |
  |         v                          v                   |
  |   +------------------+    +--------------------+       |
  |   | yt-dlp           |--->| 5 client strategies|       |
  |   | + PO Token plugin|    | + free proxy pool  |       |
  |   +------------------+    +--------------------+       |
  |         |                                              |
  |   +------------------+    +--------------------+       |
  |   | Flask / Gunicorn |--->| HTTP :8080         |       |
  |   |                  |    | Web Interface       |       |
  |   +------------------+    +--------------------+       |
  |                                                        |
  +-------------------------------------------------------+
```

> &#x1F4A1; **The critical connection:** `YT_DLP_POT_PROVIDER_URL` env var links yt-dlp to the PO Token server. Without it, the server runs but yt-dlp ignores it.

**5-Layer Defense System:**

| Layer | Defense | Description |
|:-----:|:--------|:------------|
| 1 | **PO Token Server** | bgutil generates auth tokens + `YT_DLP_POT_PROVIDER_URL` connects to yt-dlp **(THE KEY FIX)** |
| 2 | **Client Rotation** | 5 player_client strategies, auto-fallback on failure |
| 3 | **Free Proxy Pool** | Auto-fetches proxies from ProxyScrape, rotates on failure |
| 4 | **Cache Management** | Auto-clears stale cache every 15 minutes |
| 5 | **Retry + Backoff** | Up to 15 attempts (5 direct + 10 proxy) with random delays |

<br>

---

<br>

## &#x1F4CB; All Commands

### &#x1F3A5; Video Download

```bash
# Just change the number after "res:" !

# 4K (best available)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" --merge-output-format mp4 "URL"

# 1080p (Full HD)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" -S "res:1080" --merge-output-format mp4 "URL"

# 720p (HD)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" -S "res:720" --merge-output-format mp4 "URL"

# 480p (SD)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" -S "res:480" --merge-output-format mp4 "URL"

# 360p (Low)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -f "bestvideo*+bestaudio/best" -S "res:360" --merge-output-format mp4 "URL"
```

### &#x1F3B5; Audio Only

```bash
# MP3 (192kbps - standard)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -x --audio-format mp3 --audio-quality 192 "URL"

# MP3 (320kbps - highest quality)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -x --audio-format mp3 --audio-quality 320 "URL"

# FLAC (lossless)
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  -x --audio-format flac "URL"
```

### &#x1F50D; List Available Formats

```bash
yt-dlp --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" -F "URL"
```

### &#x1F6E1;&#xFE0F; Full Command with All Protections

```bash
yt-dlp \
  --extractor-args "youtube:player_client=web,android_vr,tv_downgraded" \
  --js-runtimes node \
  --remote-components ejs:github \
  --rm-cache-dir \
  -f "bestvideo*+bestaudio/best" \
  -S "res:1080" \
  --merge-output-format mp4 \
  --retries 3 \
  --fragment-retries 3 \
  "URL"
```

<br>

---

<br>

## &#x1F4BE; Permanent Config

> *Tired of typing long commands?* Save this once, use forever.

Create `~/.config/yt-dlp/config`:

```ini
# yt-dlp-rescue config v2.0 (March 2026)
# https://github.com/bravomylife-lab/yt-dlp-rescue

# Fix YouTube SABR quality selection
--extractor-args youtube:player_client=web,android_vr,tv_downgraded

# Best video + audio with progressive fallback
-f bestvideo*+bestaudio/best

# Output format
--merge-output-format mp4

# Bot detection bypass: PO Token auto-generation
# Requires: pip install bgutil-ytdlp-pot-provider
# Requires: Node.js installed
# --js-runtimes node
# --remote-components ejs:github

# Reliability settings
--retries 3
--fragment-retries 3
--socket-timeout 30
```

Now just run:
```bash
yt-dlp -S "res:1080" "URL"   # That's it!
```

<br>

---

<br>

## &#x1F6E0;&#xFE0F; Troubleshooting

<details>
<summary><b>&#x1F624; Still getting 360p</b></summary>

```bash
# 1. Update yt-dlp to latest
pip install -U yt-dlp

# 2. Clear stale cache
yt-dlp --rm-cache-dir

# 3. Try different client order
yt-dlp --extractor-args "youtube:player_client=tv_downgraded,web_creator,mweb" \
  -f "bestvideo*+bestaudio/best" -S "res:1080" "URL"

# 4. Check what formats are actually available
yt-dlp --extractor-args "youtube:player_client=tv_downgraded" -F "URL"
```
</details>

<details>
<summary><b>&#x1F916; "Sign in to confirm you're not a bot"</b></summary>

This means YouTube flagged your IP. Try in order:

```bash
# 1. Clear cache + different client
yt-dlp --rm-cache-dir
yt-dlp --extractor-args "youtube:player_client=tv_downgraded,web,android_vr" "URL"

# 2. Enable PO Token generation (install once)
pip install bgutil-ytdlp-pot-provider
yt-dlp --js-runtimes node --remote-components ejs:github "URL"

# 3. Use WARP proxy (if on cloud/datacenter)
yt-dlp --proxy socks5://127.0.0.1:1080 "URL"

# 4. Last resort: browser cookies
yt-dlp --cookies-from-browser chrome "URL"
```
</details>

<details>
<summary><b>&#x1F6AB; "HTTP Error 403: Forbidden"</b></summary>

```bash
# Your IP is blocked. Rotate user agent + client:
yt-dlp --user-agent "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36" \
  --extractor-args "youtube:player_client=tv_downgraded,web_creator" "URL"

# If on cloud server, use WARP proxy (see Cloud Deployment section)
```
</details>

<details>
<summary><b>&#x1F527; ffmpeg not found</b></summary>

```bash
# macOS
brew install ffmpeg

# Ubuntu / Debian
sudo apt install ffmpeg

# Windows
winget install ffmpeg

# Verify
ffmpeg -version
```

> Without ffmpeg, you're limited to progressive formats. Install ffmpeg for the full quality range.
</details>

<details>
<summary><b>&#x1F4A1; What does the <code>*</code> in <code>bestvideo*</code> mean?</b></summary>

```
-f "bestvideo*+bestaudio/best"
             ^
             +-- This asterisk = safety net!

  bestvideo   ->  video-only streams
  bestvideo*  ->  video-only + progressive (video+audio) streams

If DASH merge fails, the * lets yt-dlp fall back to
a single-file format instead of failing completely.
```
</details>

<details>
<summary><b>&#x1F4BB; Works locally, fails on my server</b></summary>

Your server has a datacenter IP which YouTube blocks at the network level. This cannot be fixed with cookies or PO Tokens alone.

&#x27A1;&#xFE0F; See the [Cloud/Server Deployment](#%EF%B8%8F-cloudserver-deployment) section for the WARP proxy solution.
</details>

<br>

---

<br>

## &#x1F4C3; Quick Reference Card

```
+------------------------------------------------------------------+
|                    yt-dlp-rescue v2.0                             |
+------------------------------------------------------------------+
|                                                                  |
|  QUALITY FIX (SABR bypass):                                      |
|    --extractor-args                                              |
|      "youtube:player_client=web,android_vr,tv_downgraded"        |
|    -f "bestvideo*+bestaudio/best"                                |
|    -S "res:1080"                                                 |
|                                                                  |
|  BOT DETECTION FIX:                                              |
|    pip install bgutil-ytdlp-pot-provider                         |
|    --js-runtimes node --remote-components ejs:github             |
|                                                                  |
|  CLOUD/SERVER FIX:                                               |
|    --proxy socks5://127.0.0.1:1080  (via WARP wireproxy)         |
|                                                                  |
|  WHEN THINGS BREAK:                                              |
|    yt-dlp --rm-cache-dir && pip install -U yt-dlp                |
|                                                                  |
+------------------------------------------------------------------+
```

<br>

---

<br>

## &#x1F4C5; Changelog

| Version | Date | Changes |
|:--------|:-----|:--------|
| **v2.1.0** | 2026-03-13 | PO Token server mode (`YT_DLP_POT_PROVIDER_URL`), free proxy auto-rotation, production-tested fix |
| **v2.0.0** | 2026-03-13 | Bot detection bypass, cloud deployment guide, WARP proxy, 5-layer defense, client rotation |
| **v1.0.0** | 2026-03-01 | Initial release: SABR quality fix, format sort, multi-client strategy |

<br>

---

<br>

<div align="center">

## &#x1F91D; Contributing

YouTube changes things constantly.
Found a better fix? Something stopped working?

**[Open an Issue](../../issues) &nbsp;/&nbsp; [Send a PR](../../pulls)**

<br>

### &#x2B50; If this rescued your downloads, leave a star

*It helps others find this fix too.*

<br>

---

<br>

<sub>

**Keywords:** yt-dlp fix, youtube 360p fix, youtube sabr fix, yt-dlp quality fix, youtube download broken, yt-dlp bot detection, youtube sign in to confirm, yt-dlp cloud server, yt-dlp datacenter blocked, yt-dlp proxy, yt-dlp warp, youtube downloader fix 2025, youtube downloader fix 2026, yt-dlp player_client, yt-dlp format sort, yt-dlp rescue, youtube quality stuck 360p

</sub>

<br>

Made with determination by [@bravomylife-lab](https://github.com/bravomylife-lab)

</div>
