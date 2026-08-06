# 🐧 Linux Command Cheatsheet (SOC Analyst Notes)

# 📂 File & Directory Management

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `ls -l` | List files with permissions | Check insecure file permissions |
| `cd` | Change directory | Navigate to important folders (`/var/log`) |
| `pwd` | Show current directory | Prevent mistakes before modifying files |
| `mkdir -p` | Create directories | Organize forensic evidence |
| `cp` | Copy files | Backup configuration files |
| `mv` | Move/Rename files | Rotate or archive logs |
| `rm -i` | Remove with confirmation | Avoid accidental deletion |

---

# 🔒 File Permissions

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `chmod` | Change permissions | Secure sensitive files (SSH keys) |
| `chown` | Change ownership | Correct ownership to prevent abuse |

---

# 📄 Viewing Files

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `cat` | Display file | Quickly inspect configuration files |
| `less` | Read large files | Investigate large log files |
| `tail -f` | Monitor file live | Watch logs during active attacks |
| `stat` | File timestamps | Build incident timeline |
| `file` | Detect actual file type | Find disguised malware |
| `strings` | Extract readable text | Find URLs, IPs, credentials in malware |

---

# 🔍 Searching & Text Processing

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `grep` | Search text | Find failed logins, IOCs |
| `awk` | Extract columns | Pull IP addresses from logs |
| `sed` | Replace/edit text | Modify configurations safely |
| `cut` | Extract fields | List usernames from `/etc/passwd` |
| `sort` + `uniq -c` | Count duplicates | Identify top attacking IPs |
| `wc -l` | Count lines | Count failed login attempts |

---

# ⚙️ Process Management

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `top` / `htop` | Monitor processes | Detect malware or crypto miners |
| `ps aux \| grep` | Find processes | Search suspicious processes |
| `pstree` / `ps auxf` | Process tree | Detect spawned shells (RCE) |
| `kill -9` | Force kill process | Stop malicious process |
| `kill -STOP` | Freeze process | Preserve malware for forensics |
| `pkill` / `killall` | Kill multiple processes | Stop all malware instances |
| `fuser -k` | Kill process using a port | Remove rogue listener |

---

# 🌐 Networking

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `ping -c 4` | Test connectivity | Verify network vs DNS issue |
| `curl -I` | View HTTP headers | Check security headers |
| `wget` | Download files | Download forensic tools |
| `ss -tulnp` | View listening ports | Detect backdoors |
| `netstat -tulpn` | List connections | Alternative to `ss` |
| `lsof -i` | Show network connections | Identify suspicious connections |
| `tcpdump` | Capture packets | Collect PCAP evidence |
| `ip a` | Show IP addresses | Detect rogue interfaces |
| `traceroute` | Trace network path | Investigate suspicious routing |
| `dig` | DNS lookup | Verify DNS records |

---

# 👤 User & Privileges

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `whoami` | Current user | Confirm privileges |
| `id` | User & groups | Check elevated access |
| `sudo` | Run as administrator | Execute privileged commands |
| `passwd -l` | Lock account | Disable compromised account |
| `usermod -L` | Lock user | Prevent attacker login |

---

# 🖥️ System Information

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `uptime` | System uptime | Detect unexpected reboot |
| `hostname` | Machine name | Verify correct host |
| `uname -a` | Kernel info | Check vulnerable kernel |
| `systemctl status` | Service status | Verify SSH/web services |
| `journalctl -u <service>` | Service logs | Investigate service issues |
| `history` | Command history | Review attacker actions |
| `man` | Command manual | Offline documentation |
| `which` | Program location | Detect replaced binaries |
| `clear` | Clear terminal | Improve readability |

---

# 🔍 User Activity

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `last` | Login history | Detect suspicious logins |
| `w` | Active users | Check for attacker sessions |

---

# 💾 Storage & Persistence

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `lsblk` | Show disks | Detect unauthorized USB drives |
| `crontab -l` | View cron jobs | Find persistence |
| `crontab -r` | Remove cron jobs | Remove attacker persistence |

---

# 🔐 Hashing

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `sha256sum` / `md5sum` | Generate file hash | Verify file integrity |

---

# 🛡️ Firewall & Containment

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `iptables -A INPUT -s <IP> -j DROP` | Block IP | Immediate containment |
| `ufw deny from <IP>` | Block IP (UFW) | Quick firewall rule |
| `ip route add blackhole <IP>` | Blackhole traffic | Silent containment |

---

# 📌 Useful Miscellaneous

| Command | Purpose | SOC Use Case |
|----------|----------|--------------|
| `find` | Search filesystem | Find recently modified files |
| `echo` | Output text | Quick configuration edits |
| `apt-get update && apt-get upgrade` | Update system | Patch vulnerabilities |
| `chattr +i` | Make file immutable | Preserve forensic evidence |

