# Windows 11 RDP via GitHub Actions

> Free Windows 11 RDP server on GitHub Actions with Bore tunnel — no credit card, no VPN, no ngrok needed.

## How It Works

This project spins up a **free Windows RDP server** using GitHub Actions runners. A GitHub Actions workflow:

1. **Cleans storage** — deletes ~70GB of pre-installed bloatware (Visual Studio, Android SDK, .NET, databases, browsers, etc.)
2. **Enables RDP** — configures Remote Desktop with aggressive performance optimizations
3. **Creates user** — `mastertech` with admin privileges
4. **Sets wallpaper** — lightweight Windows 11 wallpaper
5. **Starts Bore tunnel** — exposes port 3389 (RDP) via `bore.pub` to get a public address
6. **Keeps alive** — maintains tunnel for up to 6 hours (GitHub Actions timeout)

## Quick Start

### 1. Fork this repository

Click the **Fork** button on GitHub.

### 2. Trigger the workflow

Go to **Actions** tab → **WindowsRDP** → **Run workflow** → Click **Run workflow**

### 3. Get your credentials

After ~3-5 minutes, check the workflow run logs or **Step Summary** for:

```
ADDRESS  : bore.pub:XXXXX
USERNAME : mastertech
PASSWORD : Mast3rT3ch@2026!
```

### 4. Connect via RDP

- **Windows**: Open "Remote Desktop Connection" → Enter `bore.pub:XXXXX`
- **Mac**: Install "Microsoft Remote Desktop" from App Store
- **Android**: Install "Microsoft RDP" from Play Store
- **iOS**: Install "Microsoft RDP" from App Store

> **Tip**: In RDP client settings, set connection speed to **Low-speed broadband (256 Kbps - 2 Mbps)** and set colors to **High Color (16-bit)** for best performance.

## Architecture

```
Your Device ──RDP──> bore.pub:PORT ──TCP──> GitHub Actions Runner (Windows)
                                              ├── RDP enabled on port 3389
                                              ├── User: mastertech
                                              └── Bore tunnel forwarding
```

## What Gets Cleaned Up (~70GB freed)

| Program | Space Saved |
|---------|-------------|
| Microsoft Visual Studio | ~31.6 GB |
| Android SDK | ~14.0 GB |
| .NET SDK | ~11.6 GB |
| GitHub hostedtoolcache | ~4.8 GB |
| GHCUP (Haskell) | ~4.2 GB |
| R + rtools | ~3.3 GB |
| Azure Cosmos DB Emulator | ~2.5 GB |
| LLVM | ~2.4 GB |
| MongoDB | ~2.0 GB |
| Windows SDK | ~2.4 GB |
| PostgreSQL | ~1.0 GB |
| Strawberry Perl | ~1.0 GB |
| OpenSSL | ~0.8 GB |
| Unity Hub | ~0.6 GB |
| MySQL Server | ~0.6 GB |
| AWS CLI + SAM | ~0.5 GB |
| Firefox | ~0.3 GB |
| CMake, ImageMagick, IIS, WiX | ~0.7 GB |
| Temp files, Pagefile, Hibernation | ~5+ GB |

## RDP Optimizations Applied

### Visual Performance
- Color depth reduced to **16-bit** (less bandwidth)
- All visual effects **disabled** (animations, shadows, transparency)
- Wallpaper set to lightweight image (no heavy backgrounds)
- Font smoothing disabled
- Desktop composition disabled

### Network Performance
- **TCP only** — UDP disabled for stability
- Low latency TCP settings (`TcpAckFrequency=1`, `TCPNoDelay=1`)
- RDP compression enabled
- DNS cache flushed

### System Optimization
- **Windows Defender real-time protection OFF**
- **20+ heavy services disabled** (SysMain, DiagTrack, Windows Update, BITS, Xbox, etc.)
- High Performance power plan
- Hibernation + Pagefile disabled
- Game Bar/DVR disabled
- OneDrive, Edge, Teams killed

## Files

| File | Description |
|------|-------------|
| `.github/workflows/rdp.yml` | Main workflow — cleanup, RDP setup, bore tunnel |

## Problems We Faced & How We Fixed Them

### Problem 1: ngrok TCP Requires Credit Card

**What happened**: We initially used ngrok to tunnel RDP (port 3389). But ngrok requires a **credit/debit card** to use TCP endpoints — even on the free tier. They don't charge, but without a card you get `ERR_NGROK_108` error.

**Research**: We searched extensively for alternatives:
- ngrok — needs card for TCP
- Tailscale — needs VPN app on client, no public IP
- Cloudflare Tunnel — needs domain + account for named tunnels
- Bore — free, open source, no account, no card ✅

**Fix**: Switched from ngrok to **Bore** (`github.com/ekzhang/bore`). It's a simple TCP tunnel — no signup, no card, no config. Just `bore local 3389 --to bore.pub` and you get a public address.

---

### Problem 2: Password Hidden in GitHub Actions Logs

**What happened**: We generated a random password and stored it in `$env:GITHUB_ENV`. GitHub Actions automatically **masks** any value passed through `GITHUB_ENV` — so the password showed as `***` in logs. User couldn't see what password to use for RDP login.

**Fix**: 
- Used a **fixed password** `Mast3rT3ch@2026!` instead of random generation
- Wrote credentials to **Step Summary** (always visible on Actions page)
- Also saved credentials to a **text file on the desktop** inside the VM
- Added **Base64 encoded** password in case plain text gets masked

---

### Problem 3: Screen Flickering & Slow RDP

**What happened**: After connecting via RDP, the screen was **flickering pixels**, felt very **slow and laggy**, and consumed too much data. The tunnel was bottlenecking the connection.

**Research**: Found that RDP defaults to 32-bit color depth and enables all visual effects — way too much data for a tunneled connection.

**Fix**: Aggressive optimization:
- **Color depth 32-bit → 16-bit** (massive bandwidth reduction)
- **UDP disabled** — TCP only for stable connection
- **All visual effects OFF** — animations, shadows, transparency, font smoothing
- **Wallpaper set to lightweight image** instead of heavy default
- **Windows Defender real-time OFF** — biggest CPU hog
- **20+ services disabled** — SysMain, DiagTrack, Windows Update, BITS, Xbox services
- **Game Bar/DVR disabled** — unnecessary GPU usage
- **OneDrive, Edge, Teams killed** — memory hogs

---

### Problem 4: Disk Space Full on Runner

**What happened**: GitHub Actions Windows runner comes with ~80GB total, but after OS and pre-installed software (Visual Studio, Android SDK, .NET, databases, browsers), only ~14GB was free. Not enough for RDP to work smoothly.

**Fix**: Added a **cleanup step** as the first action in the workflow. It deletes:
- Visual Studio (~31GB)
- Android SDK (~14GB)
- .NET SDK (~11GB)
- hostedtoolcache (~5GB)
- Haskell, R, MongoDB, LLVM, PostgreSQL, MySQL, AWS CLI, Firefox, Unity, CMake
- Temp files, hibernation file, pagefile
- **Total: ~70GB+ freed**

---

### Problem 5: Tailscale Requires VPN App

**What happened**: The original workflow used Tailscale to expose RDP. Problem — Tailscale requires installing a **VPN client app** on every device that wants to connect. Users had to:
1. Install Tailscale app
2. Create account / login
3. Join the same network
4. Then connect RDP

This was too many steps and not user-friendly.

**Fix**: Switched to Bore tunnel. Now users just need any standard RDP client — no app install, no account, no VPN. Just enter the `bore.pub:PORT` address and connect.

---

### Problem 6: No Persistent Address

**What happened**: Every time the workflow runs, Bore assigns a **new random port**. So `bore.pub:12345` becomes `bore.pub:54321` next time. Users can't bookmark or save the address.

**Current status**: This is still a limitation. Bore's free public server doesn't support fixed ports.

**Potential fix**: Self-host your own Bore server on Oracle Cloud Always Free VPS — then you can configure a fixed port.

---

## Improvements We Made

### Improvement 1: Tunnel Migration (ngrok → Bore)
- **Before**: ngrok required credit card, 1GB/month limit, 5000 TCP connections/month
- **After**: Bore — no card, no limits, open source, MIT license
- **Impact**: Anyone can now use this without financial barriers

### Improvement 2: Fixed Credential Visibility
- **Before**: Password masked as `***` in logs, user couldn't connect
- **After**: Fixed password visible in Step Summary, logs, and desktop file
- **Impact**: User can always see their login credentials

### Improvement 3: Aggressive RDP Optimization
- **Before**: Default 32-bit color, all visual effects on, slow and flickery
- **After**: 16-bit color, all effects off, UDP disabled, Defender off
- **Impact**: Much smoother RDP experience over tunnel

### Improvement 4: 70GB+ Storage Cleanup
- **Before**: ~14GB free, not enough for comfortable use
- **After**: ~84GB free after cleanup
- **Impact**: User gets a usable Windows desktop with plenty of space

### Improvement 5: Wallpaper Customization
- **Before**: Solid color background (boring)
- **After**: Lightweight Windows 11 wallpaper from WallpaperHub
- **Impact**: Better visual experience without performance hit

### Improvement 6: Service Optimization
- **Before**: 20+ unnecessary services running (SysMain, DiagTrack, Xbox, etc.)
- **After**: All heavy services disabled, high performance power plan
- **Impact**: More CPU/RAM available for RDP and user tasks

## Alternatives Comparison

| Feature | This Project | ngrok | Tailscale | Oracle Cloud Free |
|---------|-------------|-------|-----------|-------------------|
| **Free** | Yes | Yes (limited) | Yes | Yes |
| **Credit Card** | No | Yes (for TCP) | No | Yes (verification) |
| **VPN Required** | No | No | Yes | No |
| **Public IP** | Via tunnel | Via tunnel | No (private) | Yes (real) |
| **Traffic Limit** | Unlimited | 1 GB/month | 100GB/mo | 10 TB/month |
| **Duration** | 6 hours | Limited | Unlimited | Unlimited |
| **Setup Time** | ~3 min | ~2 min | ~5 min | ~30 min |

## FAQ

**Q: Is this really free?**
A: Yes. GitHub Actions provides free minutes for public repositories. Bore tunnel is free and open source.

**Q: Can I use this for gaming?**
A: Not recommended. RDP is not optimized for gaming. The latency from tunneling makes it impractical.

**Q: Is my data secure?**
A: RDP traffic is encrypted. However, bore.pub server can theoretically intercept traffic. For sensitive use, self-host the bore server.

**Q: How do I get a persistent RDP address?**
A: Use Cloudflare Tunnel with your own domain ($10/year). Or self-host bore server on Oracle Cloud Always Free.

**Q: The connection is still slow. What can I do?**
A: In your RDP client:
1. Set color depth to 16-bit or lower
2. Disable wallpaper, font smoothing, desktop composition
3. Set connection speed to 256 Kbps
4. Disable clipboard sharing if not needed

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test by triggering the workflow
5. Submit a pull request

## License

MIT License - use freely, modify as needed.

## Credits

- [Bore](https://github.com/ekzhang/bore) - Open source TCP tunnel
- [GitHub Actions](https://github.com/features/actions) - Free CI/CD runners
- [WallpaperHub](https://wallpaperhub.app/) - Windows 11 wallpaper
