# Linux Logs Cheat Sheet — SOC Analyst Reference

## 1. Authentication Logs
| Distro | Path |
|---|---|
| Debian/Ubuntu | `/var/log/auth.log` |
| RHEL/CentOS/Fedora | `/var/log/secure` |

**Tracks:** SSH logins/logouts, `sudo` usage, `su` switches, PAM events, failed/successful password attempts, new user/group creation.

**Red flags:**
- Repeated `Failed password` from same/varied IPs → brute force
- `Accepted password` for root or service accounts from unfamiliar IPs
- `sudo: ... COMMAND=` entries for unexpected privileged commands
- New accounts added right before/after odd login activity

**Quick commands:**
```bash
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
grep "sudo" /var/log/auth.log
journalctl -u sshd --since "1 hour ago"
```

---

## 2. System-Wide Logs
| Distro | Path |
|---|---|
| Debian/Ubuntu | `/var/log/syslog` |
| RHEL/CentOS | `/var/log/messages` |
| Kernel events | `/var/log/kern.log` or `dmesg` |

**Tracks:** kernel messages, daemon start/stop, hardware events, general OS activity.

**Red flags:**
- Unexpected kernel module loads (possible rootkit)
- Daemons crashing/restarting repeatedly
- USB device insertions on servers that shouldn't have physical access
- Out-of-memory killer events tied to suspicious processes

**Quick commands:**
```bash
tail -f /var/log/syslog
dmesg | grep -i "usb\|error\|denied"
journalctl -k   # kernel ring buffer via journald
```

---

## 3. Startup / Boot Logs
| Distro | Path |
|---|---|
| Most distros | `/var/log/boot.log` |
| systemd | `journalctl -b` |

**Tracks:** services started at boot, boot-time success/failure.

**Red flags:**
- Unknown service enabled to start at boot (persistence mechanism)
- Critical security service (firewall, auditd, EDR agent) failing to start

**Quick commands:**
```bash
journalctl -b -p err     # boot errors only
systemctl list-unit-files --state=enabled
```

---

## 4. Audit Logs (auditd)
| Path |
|---|
| `/var/log/audit/audit.log` |

**Tracks:** system calls, file access, command execution, SELinux denials — configured via audit rules (`auditctl`, `/etc/audit/rules.d/`).

**Why it matters:** Deepest visibility layer — used for forensics, compliance (PCI-DSS, HIPAA), tracing exactly who touched what.

**Red flags:**
- `type=AVC denied` (SELinux denial) on sensitive files
- Unexpected `execve` of tools like `nc`, `curl`, `wget`, `python -c`
- Access to `/etc/shadow`, `/etc/passwd`, SSH keys outside normal admin activity

**Quick commands:**
```bash
ausearch -m avc -ts recent
aureport --summary
ausearch -f /etc/shadow
```

---

## 5. Cron / Scheduled Job Logs
| Distro | Path |
|---|---|
| Debian/Ubuntu | `/var/log/cron.log` |
| RHEL/CentOS | `/var/log/cron` |
| Also check | `journalctl -u cron` or `-u crond` |

**Tracks:** execution of cron jobs, including system and per-user crontabs.

**Red flags:**
- New/unfamiliar cron entries, especially ones downloading/executing remote scripts
- Jobs running as root that weren't there during baseline
- Jobs in `/etc/cron.d/`, `/var/spool/cron/`, or systemd timers that mirror known persistence techniques

**Quick commands:**
```bash
crontab -l -u <user>
cat /etc/cron.d/* /etc/crontab
grep CRON /var/log/syslog
systemctl list-timers --all
```

---

## 6. User Session Logs (utmp / wtmp / btmp)
| Log | Purpose | View with |
|---|---|---|
| `/var/run/utmp` | Currently logged-in users | `who`, `w` |
| `/var/log/wtmp` | Historical login/logout records | `last` |
| `/var/log/btmp` | Failed login attempts | `lastb` |
| `/var/log/lastlog` | Last login time per user | `lastlog` |

**Red flags:**
- Logins at unusual hours or from unusual source IPs/ttys
- `lastb` showing high-volume failed logins (brute force)
- Gaps in `wtmp` (possible log tampering/anti-forensics)

**Quick commands:**
```bash
last -a | head -20
lastb | head -20
lastlog | grep -v "Never"
```

---

## 7. Package Management Logs
| Distro | Path |
|---|---|
| Debian/Ubuntu (dpkg) | `/var/log/dpkg.log` |
| Debian/Ubuntu (apt) | `/var/log/apt/history.log` |
| RHEL/CentOS (yum/dnf) | `/var/log/yum.log` or `dnf.log` |

**Tracks:** software installed, removed, or upgraded.

**Red flags:**
- Installation of tools like netcat, nmap, hydra, or compilers on production servers
- Package installs that don't align with a change ticket
- Downgrades of security-relevant packages

**Quick commands:**
```bash
grep " install " /var/log/dpkg.log
zcat /var/log/apt/history.log*.gz | grep Install
```

---

## 8. Bash Command History
| Path |
|---|
| `~/.bash_history` (per user) |
| Root: `/root/.bash_history` |

**Why it's critical:** Direct record of interactive commands run by a user or attacker post-compromise. Often the single most valuable artifact in an incident.

**Red flags:**
- Recon commands (`whoami`, `id`, `uname -a`, `cat /etc/passwd`)
- Download/execute chains (`curl ... | bash`, `wget` + `chmod +x`)
- History cleared (`history -c`) or `HISTFILE` unset — a major tampering indicator
- Timestamps missing (attackers often disable `HISTTIMEFORMAT`)

**Quick commands:**
```bash
cat ~/.bash_history
export HISTTIMEFORMAT="%F %T "   # enable timestamps going forward
find / -name ".bash_history" 2>/dev/null
```

---

## Bonus: Other Logs Worth Knowing
| Log type | Path | Notes |
|---|---|---|
| systemd journal | `journalctl` (binary, `/var/log/journal/`) | Central query tool for most of the above on modern distros |
| Mail logs | `/var/log/mail.log` or `/var/log/maillog` | Phishing/spam relay abuse |
| Web server (Apache) | `/var/log/apache2/access.log`, `error.log` | Web attacks, scanning, exploitation attempts |
| Web server (Nginx) | `/var/log/nginx/access.log`, `error.log` | Same as above |
| Firewall (UFW) | `/var/log/ufw.log` | Blocked/allowed connection attempts |
| iptables | via `syslog`/`kern.log` if logging rules configured | Network-level filtering events |
| Docker | `journalctl -u docker` or `/var/lib/docker/containers/*/*.log` | Container-level activity |

---

## Master List — Every Log File a SOC Analyst Should Know

### Core system & auth
| Log | Path | Purpose |
|---|---|---|
| Auth log | `/var/log/auth.log` (Debian/Ubuntu) | Logins, sudo, SSH, PAM |
| Secure log | `/var/log/secure` (RHEL/CentOS) | Same as above, RH-family |
| Syslog | `/var/log/syslog` | General system activity |
| Messages | `/var/log/messages` | RH-family equivalent of syslog |
| Kernel log | `/var/log/kern.log` | Kernel-level events, driver issues |
| dmesg | `dmesg` (buffer, not a flat file) | Kernel ring buffer since boot |
| Boot log | `/var/log/boot.log` | Service start/fail at boot |
| Systemd journal | `/var/log/journal/` (query via `journalctl`) | Unified binary log, modern distros |
| Fail login counter | `/var/log/faillog` or `/var/log/tallylog` | PAM failed-login counters (pam_tally2/pam_faillock) |
| Last login | `/var/log/lastlog` | Per-user last login timestamp |

### Session & identity
| Log | Path | Purpose |
|---|---|---|
| utmp | `/var/run/utmp` | Currently active sessions |
| wtmp | `/var/log/wtmp` | Historical login/logout |
| btmp | `/var/log/btmp` | Failed login attempts |
| Bash history | `~/.bash_history` | Interactive command history |
| Zsh history | `~/.zsh_history` | Same, if zsh is the shell |
| Sudoers log | Captured in `auth.log`/`secure` | Privilege escalation actions |

### Audit & compliance
| Log | Path | Purpose |
|---|---|---|
| auditd | `/var/log/audit/audit.log` | Syscalls, file access, exec tracking |
| SELinux denials | `/var/log/audit/audit.log` (AVC) or `/var/log/setroubleshoot/setroubleshootd.log` | Policy violations |
| AppArmor | `/var/log/syslog` (tagged apparmor) or `journalctl` | MAC policy denials on Debian/Ubuntu |

### Scheduled tasks & persistence
| Log | Path | Purpose |
|---|---|---|
| Cron | `/var/log/cron.log` (Debian) / `/var/log/cron` (RHEL) | Scheduled job execution |
| Systemd timers | `journalctl -u <timer>` | Modern replacement/companion to cron |
| At jobs | `/var/log/syslog` (tagged atd) | One-off scheduled jobs |

### Package management
| Log | Path | Purpose |
|---|---|---|
| dpkg | `/var/log/dpkg.log` | Debian/Ubuntu package installs |
| apt history | `/var/log/apt/history.log` | Higher-level apt actions |
| yum | `/var/log/yum.log` | RHEL/CentOS older package manager |
| dnf | `/var/log/dnf.log` | RHEL/Fedora newer package manager |
| pacman | `/var/log/pacman.log` | Arch-based systems |

### Network & perimeter
| Log | Path | Purpose |
|---|---|---|
| UFW firewall | `/var/log/ufw.log` | Allowed/blocked connections (Ubuntu) |
| iptables | via `syslog`/`kern.log` if `--log` rules set | Raw netfilter logging |
| firewalld | `journalctl -u firewalld` | RHEL-family firewall manager |
| NetworkManager | `/var/log/NetworkManager` or `journalctl -u NetworkManager` | Interface/connection changes |
| wpa_supplicant | `/var/log/wpa_supplicant.log` | Wireless auth events |
| DNS (BIND/named) | `/var/log/named.log` or `/var/log/messages` | DNS query/resolution activity |
| NTP | `/var/log/ntp.log` or `journalctl -u ntpd` | Time sync issues (relevant for log integrity/replay) |
| fail2ban | `/var/log/fail2ban.log` | Ban/unban actions from intrusion prevention |

### Application & service layer
| Log | Path | Purpose |
|---|---|---|
| Apache access | `/var/log/apache2/access.log` or `/var/log/httpd/access_log` | Web requests |
| Apache error | `/var/log/apache2/error.log` or `.../error_log` | Web server errors |
| Nginx access/error | `/var/log/nginx/access.log`, `error.log` | Same, Nginx |
| Mail | `/var/log/mail.log` or `/var/log/maillog` | SMTP relay, spam/phishing abuse |
| MySQL/MariaDB | `/var/log/mysql/error.log`, `mysql-slow.log` | DB errors, slow/suspicious queries |
| PostgreSQL | `/var/log/postgresql/` | DB activity |
| Samba | `/var/log/samba/` | SMB file share access |
| vsftpd | `/var/log/vsftpd.log` | FTP access |
| PHP-FPM | `/var/log/php-fpm/` | PHP application errors (webshell indicators) |
| Docker | `journalctl -u docker` or `/var/lib/docker/containers/*/*.log` | Container runtime activity |
| CUPS | `/var/log/cups/` | Print system (rarely relevant, but exists) |

### Security tooling
| Log | Path | Purpose |
|---|---|---|
| ClamAV | `/var/log/clamav/` | AV scan results |
| OSSEC/Wazuh agent | `/var/ossec/logs/` | HIDS alerts |
| rsyslog | `/var/log/rsyslog.log` (or its own status log) | Log forwarding pipeline health — check if logs stop arriving |

### Install / provisioning
| Log | Path | Purpose |
|---|---|---|
| Anaconda (RHEL installer) | `/var/log/anaconda/` | OS install-time events, rarely relevant post-deployment |
| cloud-init | `/var/log/cloud-init.log` | Cloud VM provisioning (AWS/Azure/GCP instances) — check for injected user-data scripts |

---

## Analyst Workflow Tips
1. **Baseline first.** Know what "normal" boot services, cron jobs, and login patterns look like before you can spot anomalies.
2. **Centralize.** Ship logs to a SIEM (Splunk, Elastic, Wazuh, etc.) — local logs can be deleted or tampered with by an attacker with root.
3. **Correlate across sources.** A single failed SSH login is noise; that + a new cron job + a cleared `.bash_history` is a story.
4. **Watch for gaps, not just events.** Missing log entries, truncated files, or reset timestamps are themselves indicators of tampering.
5. **Protect log integrity.** Use `auditd` immutable mode, remote syslog forwarding, and file integrity monitoring on `/var/log/` itself.
