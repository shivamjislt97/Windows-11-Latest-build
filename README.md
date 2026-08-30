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

## Known Issues & Limitations

### 1. Bore Server Unreliability
**Problem**: `bore.pub` is a community-run server with no SLA. It can go down anytime.

**Solution**: Self-host your own bore server on a free VPS (Oracle Cloud Always Free), or use `cloudflared` as an alternative tunnel.

### 2. 6-Hour Timeout
**Problem**: GitHub Actions jobs have a maximum timeout of 6 hours. The RDP session dies after that.

**Solution**: Re-trigger the workflow when it expires. Could implement a self-retrigger using a cron workflow.

### 3. No Persistent IP/Address
**Problem**: Bore assigns a random port each time. You get a new `bore.pub:XXXXX` address every run.

**Solution**: Use Cloudflare Named Tunnel with your own domain for a persistent address.

### 4. Bandwidth Limitations
**Problem**: Bore tunnel adds latency. RDP over tunnel is slower than direct connection.

**Solution**: This is inherent to tunneling. The 16-bit color depth and disabled visual effects help reduce bandwidth usage.

### 5. Runner Disk Space
**Problem**: GitHub Actions runners have limited disk space (~14GB free after OS). Heavy programs eat into this.

**Solution**: The cleanup step deletes ~70GB of pre-installed software to free space.

### 6. Password Visible in Logs
**Problem**: GitHub Actions masks values stored in `$env:GITHUB_ENV`, hiding the password in logs.

**Solution**: Fixed password `Mast3rT3ch@2026!` is used instead of random generation. Credentials are also written to Step Summary and a desktop file.

## Improvements Roadmap

### Phase 1: Reliability
- [ ] Add fallback tunnel (try bore → cloudflared → localtunnel)
- [ ] Auto-restart workflow on failure
- [ ] Add health check endpoint
- [ ] Persist credentials to a gist or external storage

### Phase 2: Performance
- [ ] Test and compare bore vs cloudflared vs frp for latency
- [ ] Add RDP session recording
- [ ] Implement bandwidth throttling
- [ ] Add GPU acceleration (if available on runner)

### Phase 3: Features
- [ ] Multi-user support (multiple RDP users)
- [ ] Browser-based RDP (via Apache Guacamole)
- [ ] File transfer support
- [ ] Clipboard sharing optimization
- [ ] Sound forwarding

### Phase 4: Self-Hosted Option
- [ ] Deploy bore server on Oracle Cloud Always Free
- [ ] Add WireGuard tunnel as alternative
- [ ] Docker-based deployment option
- [ ] Terraform/Ansible for one-click setup

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
