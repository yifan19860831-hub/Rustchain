# RustChain Mining Optimization Guide

> **Maximize your RTC rewards with proven optimization strategies**

This comprehensive guide covers hardware, system, and network optimizations for RustChain mining. Following these recommendations can increase your effective earnings by **2-3×** compared to default configurations.

---

## Table of Contents

1. [Hardware Optimization](#1-hardware-optimization)
2. [System Optimization](#2-system-optimization)
3. [Network Optimization](#3-network-optimization)
4. [Monitoring & Troubleshooting](#4-monitoring--troubleshooting)
5. [Quick Reference](#5-quick-reference)

---

## 1. Hardware Optimization

### 1.1 Understanding the Antiquity Multiplier

RustChain uses **Proof-of-Antiquity** — older hardware earns higher rewards:

| Hardware | Era | Multiplier | Relative Earnings |
|----------|-----|------------|-------------------|
| PowerPC G4 | 1999-2005 | **2.5×** | 0.30 RTC/epoch |
| PowerPC G5 | 2003-2006 | **2.0×** | 0.24 RTC/epoch |
| PowerPC G3 | 1997-2003 | **1.8×** | 0.21 RTC/epoch |
| IBM POWER8 | 2014 | **1.5×** | 0.18 RTC/epoch |
| Pentium 4 | 2000-2008 | **1.5×** | 0.18 RTC/epoch |
| Core 2 Duo | 2006-2011 | **1.3×** | 0.16 RTC/epoch |
| Apple Silicon | 2020+ | **1.2×** | 0.14 RTC/epoch |
| Modern x86_64 | Current | **1.0×** | 0.12 RTC/epoch |

> ⚠️ **Note:** Multipliers decay at 15%/year to prevent permanent advantage.

### 1.2 Hardware Selection Strategy

#### Optimal Mining Hardware (Ranked)

1. **PowerPC G4 Mac Mini / iBook** (Best ROI)
   - Purchase: $50-150 on eBay
   - Power: 15-25W
   - Multiplier: 2.5×
   - **Break-even: ~2-3 months**

2. **PowerPC G5 PowerMac**
   - Purchase: $80-200
   - Power: 80-150W (higher power cost)
   - Multiplier: 2.0×

3. **Pentium 4 Era Machines**
   - Purchase: $20-50
   - Power: 60-100W
   - Multiplier: 1.5×

4. **Core 2 Duo Laptops**
   - Purchase: $30-80
   - Power: 30-50W
   - Multiplier: 1.3×

#### Hardware Optimization Tips

```bash
# Verify your hardware multiplier
curl -sk "https://rustchain.org/api/miners" | grep -A5 "your_wallet_name"

# Check hardware fingerprint recognition
python3 ~/.rustchain/fingerprint_checks.py --verbose
```

### 1.3 Multi-Machine Strategy

**Key Rule:** Each unique hardware device = 1 vote per epoch

Running multiple miners on **different** vintage machines:

| Setup | Total Multiplier | Daily RTC (est.) |
|-------|-----------------|------------------|
| 1× G4 Mac | 2.5× | 0.72 RTC |
| 3× G4 Macs | 7.5× | 2.16 RTC |
| 1× G4 + 1× G5 + 1× P4 | 6.0× | 1.73 RTC |

> 🎯 **Strategy:** Diversify across multiple vintage machines rather than upgrading to fewer newer machines.

### 1.4 Hardware Maintenance

**Prevent multiplier loss from hardware failure:**

1. **Clean dust from vintage systems** monthly (thermal throttling reduces attestation success)
2. **Replace dried thermal paste** on CPUs >10 years old
3. **Check capacitor health** — bulging capacitors cause instability
4. **Use UPS** for power protection (unexpected shutdowns = missed epochs)

```bash
# Monitor thermal throttling (Linux)
watch -n 5 'cat /sys/class/thermal/thermal_zone*/temp'

# Check for attestation failures due to thermal issues
journalctl --user -u rustchain-miner | grep -i "thermal\|throttl"
```

---

## 2. System Optimization

### 2.1 Installation Best Practices

#### Fresh Install (Recommended)

```bash
# Clean install with custom wallet
curl -sSL https://raw.githubusercontent.com/Scottcjn/Rustchain/main/install-miner.sh | bash -s -- --wallet my-optimized-miner

# Verify installation
~/.rustchain/start.sh &
sleep 30
curl -sk "https://rustchain.org/wallet/balance?miner_id=my-optimized-miner"
```

#### Virtual Environment Isolation

```bash
# Ensure you're using an isolated venv
ls -la ~/.rustchain/venv/bin/python

# Upgrade dependencies safely
source ~/.rustchain/venv/bin/activate
pip install --upgrade requests
```

### 2.2 Service Configuration

#### Linux (systemd) Optimization

Create optimized service file:

```bash
# ~/.config/systemd/user/rustchain-miner.service
[Unit]
Description=RustChain Miner (Optimized)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/home/$USER/.rustchain/venv/bin/python /home/$USER/.rustchain/rustchain_miner.py --wallet YOUR_WALLET
Restart=always
RestartSec=10

# Resource limits (prevent runaway memory)
MemoryMax=512M
MemoryHigh=384M

# I/O priority (don't starve other processes)
IOSchedulingClass=idle
CPUSchedulingPolicy=idle

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=rustchain-miner

[Install]
WantedBy=default.target
```

```bash
# Apply optimized configuration
systemctl --user daemon-reload
systemctl --user restart rustchain-miner
systemctl --user enable rustchain-miner
```

#### macOS (launchd) Optimization

```xml
<!-- ~/Library/LaunchAgents/com.rustchain.miner.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.rustchain.miner</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/YOUR_USER/.rustchain/venv/bin/python</string>
        <string>-u</string>
        <string>/Users/YOUR_USER/.rustchain/rustchain_miner.py</string>
        <string>--wallet</string>
        <string>YOUR_WALLET</string>
    </array>
    <key>WorkingDirectory</key>
    <string>/Users/YOUR_USER/.rustchain</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>LowPriorityIO</key>
    <true/>
    <key>Nice</key>
    <integer>10</integer>
</dict>
</plist>
```

```bash
# Apply configuration
launchctl unload ~/Library/LaunchAgents/com.rustchain.miner.plist 2>/dev/null
launchctl load ~/Library/LaunchAgents/com.rustchain.miner.plist
```

### 2.3 Resource Management

#### CPU Priority

Run miner with low priority to avoid impacting system performance:

```bash
# Linux: nice + ionice
nice -n 19 ionice -c 3 ~/.rustchain/venv/bin/python \
  ~/.rustchain/rustchain_miner.py --wallet YOUR_WALLET

# macOS: renice
sudo renice 10 -p $(pgrep -f rustchain_miner.py)
```

#### Memory Optimization

```bash
# Monitor miner memory usage
watch -n 5 'ps -o pid,rss,command -p $(pgrep -f rustchain_miner.py)'

# If memory exceeds 512MB, restart service
systemctl --user restart rustchain-miner
```

### 2.4 Log Management

```bash
# Create log rotation config
cat > /etc/logrotate.d/rustchain-miner << 'EOF'
~/.rustchain/miner.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 $USER $USER
}
EOF

# View recent errors only
journalctl --user -u rustchain-miner -p err --since "24h ago"

# Real-time monitoring (filtered)
journalctl --user -u rustchain-miner -f | grep -E "attestation|reward|epoch"
```

---

## 3. Network Optimization

### 3.1 Node Selection

RustChain operates multiple nodes. Connect to the lowest-latency node:

| Node | IP | Location | Role |
|------|-----|----------|------|
| Node 1 | 50.28.86.131 | US East | Primary + Explorer |
| Node 2 | 50.28.86.153 | US East | Ergo Anchor |
| Node 3 | 76.8.228.245 | Community | External |

#### Test Node Latency

```bash
# Test latency to each node
for node in 50.28.86.131 50.28.86.153 76.8.228.245; do
  echo -n "$node: "
  curl -sk -o /dev/null -w "%{time_total}s\n" --connect-timeout 5 \
    "https://$node/health"
done
```

### 3.2 Connection Stability

#### Network Requirements

- **Minimum bandwidth:** 100 Kbps (upload + download)
- **Maximum latency:** 500ms (higher = missed epochs)
- **Packet loss tolerance:** <1%
- **Uptime requirement:** 95%+ for optimal rewards

#### Connection Monitoring

```bash
# Create monitoring script
cat > ~/.rustchain/check_connectivity.sh << 'EOF'
#!/bin/bash
NODE="https://rustchain.org"
LOG="$HOME/.rustchain/connectivity.log"

echo "$(date '+%Y-%m-%d %H:%M:%S')" >> $LOG

# Test connectivity
if curl -sk --connect-timeout 10 "$NODE/health" > /dev/null 2>&1; then
  echo "  Status: ONLINE" >> $LOG
  curl -sk --connect-timeout 10 "$NODE/epoch" >> $LOG 2>&1
else
  echo "  Status: OFFLINE" >> $LOG
  # Attempt restart
  systemctl --user restart rustchain-miner
  echo "  Action: Restarted miner service" >> $LOG
fi
EOF

chmod +x ~/.rustchain/check_connectivity.sh

# Add to crontab (every 5 minutes)
(crontab -l 2>/dev/null; echo "*/5 * * * * $HOME/.rustchain/check_connectivity.sh") | crontab -
```

### 3.3 Firewall Configuration

#### Linux (iptables/ufw)

```bash
# Allow outbound HTTPS (required for mining)
sudo ufw allow out 443/tcp comment "RustChain HTTPS"

# Or with iptables
sudo iptables -A OUTPUT -p tcp --dport 443 -m comment --comment "RustChain" -j ACCEPT
```

#### macOS (pf)

```bash
# Check firewall status
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate

# Allow Python (required for miner)
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --add /Users/YOUR_USER/.rustchain/venv/bin/python
sudo /usr/libexec/ApplicationFirewall/socketfilterfw --unblockapp /Users/YOUR_USER/.rustchain/venv/bin/python
```

### 3.4 Attestation Optimization

The attestation process is critical for earning rewards. Optimize success rate:

#### Pre-Attestation Checklist

```bash
# 1. Verify network connectivity
curl -sk https://rustchain.org/health

# 2. Check current epoch
curl -sk https://rustchain.org/epoch

# 3. Verify wallet exists
curl -sk "https://rustchain.org/wallet/balance?miner_id=YOUR_WALLET"

# 4. Test attestation challenge
curl -sk -X POST https://rustchain.org/attest/challenge \
  -H "Content-Type: application/json" \
  -d '{}'
```

#### Attestation Failure Recovery

```bash
# Create auto-recovery script
cat > ~/.rustchain/recover_attestation.sh << 'EOF'
#!/bin/bash
WALLET="YOUR_WALLET"
NODE="https://rustchain.org"

# Check if attestation is failing
LAST_ATTEST=$(journalctl --user -u rustchain-miner --since "10m ago" \
  | grep -c "attestation failed")

if [ "$LAST_ATTEST" -gt 3 ]; then
  echo "Multiple attestation failures detected. Attempting recovery..."
  
  # Stop miner
  systemctl --user stop rustchain-miner
  
  # Clear stale state
  rm -f ~/.rustchain/*.state ~/.rustchain/*.lock
  
  # Restart
  systemctl --user start rustchain-miner
  
  # Verify recovery
  sleep 30
  curl -sk "$NODE/wallet/balance?miner_id=$WALLET"
fi
EOF

chmod +x ~/.rustchain/recover_attestation.sh
```

### 3.5 Bandwidth Optimization

For limited bandwidth connections:

```bash
# Modify miner to reduce request frequency (advanced)
# Edit ~/.rustchain/rustchain_miner.py

# Find and adjust:
# HEARTBEAT_INTERVAL = 60  # Change to 120 for half the requests
# ATTESTATION_TIMEOUT = 30  # Reduce from 60 if network is fast
```

---

## 4. Monitoring & Troubleshooting

### 4.1 Essential Monitoring Commands

```bash
# Check miner status
systemctl --user status rustchain-miner

# View live logs (filtered for important events)
journalctl --user -u rustchain-miner -f | grep -E "epoch|reward|attest|error"

# Check wallet balance
curl -sk "https://rustchain.org/wallet/balance?miner_id=YOUR_WALLET"

# List all active miners (find your position)
curl -sk https://rustchain.org/api/miners | jq '.[] | select(.wallet | contains("YOUR_WALLET"))'

# Check node health
curl -sk https://rustchain.org/health
```

### 4.2 Common Issues & Solutions

#### Issue: "Could not reach network"

```bash
# Diagnose
curl -I https://rustchain.org

# Solutions:
# 1. Check internet connection
ping -c 4 8.8.8.8

# 2. Check DNS
nslookup rustchain.org

# 3. Try alternative DNS
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf

# 4. Check firewall
sudo ufw status
```

#### Issue: "Attestation failed"

```bash
# Check hardware fingerprint
python3 ~/.rustchain/fingerprint_checks.py --verbose

# Verify system time (must be accurate)
timedatectl status  # Linux
date                # macOS

# Restart with fresh state
systemctl --user stop rustchain-miner
rm -f ~/.rustchain/*.state
systemctl --user start rustchain-miner
```

#### Issue: "Miner exits immediately"

```bash
# Check logs
journalctl --user -u rustchain-miner --since "5m ago"

# Verify wallet exists
curl -sk "https://rustchain.org/wallet/balance?miner_id=YOUR_WALLET"

# Reinstall if needed
curl -sSL https://raw.githubusercontent.com/Scottcjn/Rustchain/main/install-miner.sh | bash -s -- --uninstall
curl -sSL https://raw.githubusercontent.com/Scottcjn/Rustchain/main/install-miner.sh | bash -s -- --wallet YOUR_WALLET
```

#### Issue: "Low rewards / No rewards"

```bash
# 1. Verify you're in the active miners list
curl -sk https://rustchain.org/api/miners | jq '.[] | select(.wallet | contains("YOUR_WALLET"))'

# 2. Check epoch participation
curl -sk https://rustchain.org/epoch

# 3. Verify hardware multiplier
curl -sk "https://rustchain.org/wallet/balance?miner_id=YOUR_WALLET" | jq '.multiplier'

# 4. Check attestation success rate
journalctl --user -u rustchain-miner | grep -c "attestation success"
journalctl --user -u rustchain-miner | grep -c "attestation failed"
```

### 4.3 Performance Dashboard

Create a monitoring dashboard script:

```bash
cat > ~/.rustchain/dashboard.sh << 'EOF'
#!/bin/bash
WALLET="YOUR_WALLET"
NODE="https://rustchain.org"

echo "╔════════════════════════════════════════════════════════╗"
echo "║           RustChain Mining Dashboard                  ║"
echo "╚════════════════════════════════════════════════════════╝"
echo

echo "📊 Wallet: $WALLET"
echo "💰 Balance: $(curl -sk "$NODE/wallet/balance?miner_id=$WALLET" 2>/dev/null | jq -r '.balance // "N/A"') RTC"
echo "🔢 Multiplier: $(curl -sk "$NODE/wallet/balance?miner_id=$WALLET" 2>/dev/null | jq -r '.multiplier // "N/A"')×"
echo

echo "🌐 Node Status: $(curl -sk --connect-timeout 5 "$NODE/health" 2>/dev/null && echo '✅ ONLINE' || echo '❌ OFFLINE')"
echo "📅 Current Epoch: $(curl -sk "$NODE/epoch" 2>/dev/null | jq -r '.current // "N/A"')"
echo

echo "🖥️  Miner Service: $(systemctl --user is-active rustchain-miner 2>/dev/null || echo 'N/A')"
echo "📈 Uptime: $(systemctl --user show rustchain-miner --property=ActiveEnterTimestamp 2>/dev/null)"
echo

echo "📊 Attestation Stats (last 24h):"
echo "   Success: $(journalctl --user -u rustchain-miner --since "24h ago" | grep -c "attestation success")"
echo "   Failed:  $(journalctl --user -u rustchain-miner --since "24h ago" | grep -c "attestation failed")"
echo

echo "💻 System Resources:"
echo "   CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}' 2>/dev/null || echo 'N/A')%"
echo "   Memory: $(ps -o rss= -p $(pgrep -f rustchain_miner.py) 2>/dev/null | awk '{printf "%.1f MB", $1/1024}' || echo 'N/A')"
echo
EOF

chmod +x ~/.rustchain/dashboard.sh

# Run dashboard
~/.rustchain/dashboard.sh
```

---

## 5. Quick Reference

### 5.1 Installation Commands

```bash
# Install
curl -sSL https://raw.githubusercontent.com/Scottcjn/Rustchain/main/install-miner.sh | bash

# Install with custom wallet
curl -sSL https://raw.githubusercontent.com/Scottcjn/Rustchain/main/install-miner.sh | bash -s -- --wallet my-wallet

# Uninstall
curl -sSL https://raw.githubusercontent.com/Scottcjn/Rustchain/main/install-miner.sh | bash -s -- --uninstall

# Dry run (preview)
curl -sSL https://raw.githubusercontent.com/Scottcjn/Rustchain/main/install-miner.sh | bash -s -- --dry-run
```

### 5.2 Service Management

#### Linux

```bash
# Status
systemctl --user status rustchain-miner

# Start/Stop/Restart
systemctl --user start rustchain-miner
systemctl --user stop rustchain-miner
systemctl --user restart rustchain-miner

# Enable/Disable auto-start
systemctl --user enable rustchain-miner
systemctl --user disable rustchain-miner

# View logs
journalctl --user -u rustchain-miner -f
```

#### macOS

```bash
# Status
launchctl list | grep rustchain

# Start/Stop
launchctl start com.rustchain.miner
launchctl stop com.rustchain.miner

# Load/Unload
launchctl load ~/Library/LaunchAgents/com.rustchain.miner.plist
launchctl unload ~/Library/LaunchAgents/com.rustchain.miner.plist

# View logs
tail -f ~/.rustchain/miner.log
```

### 5.3 API Endpoints

```bash
# Health check
curl -sk https://rustchain.org/health

# Current epoch
curl -sk https://rustchain.org/epoch

# Wallet balance
curl -sk "https://rustchain.org/wallet/balance?miner_id=WALLET_NAME"

# Active miners
curl -sk https://rustchain.org/api/miners

# Attestation challenge
curl -sk -X POST https://rustchain.org/attest/challenge -H "Content-Type: application/json" -d '{}'
```

### 5.4 Optimization Checklist

- [ ] Using vintage hardware (PowerPC G4/G5 recommended)
- [ ] Service configured with low CPU priority
- [ ] Memory limits set (512MB max)
- [ ] Log rotation configured
- [ ] Network connectivity monitored
- [ ] Firewall allows HTTPS outbound
- [ ] System time synchronized
- [ ] UPS connected (for desktop miners)
- [ ] Dashboard script installed
- [ ] Auto-recovery scripts configured

---

## Contributing

Found an optimization technique not listed here? Submit a PR to add it!

**Related Bounties:**
- [#1646](https://github.com/Scottcjn/rustchain-bounties/issues/1646) - Mining Optimization Guide (this document)

---

## License

This guide is part of the RustChain ecosystem and follows the same license as the main repository.

---

*Last updated: March 2026*
*Version: 1.0.0*
