# Linux Logs — SOC Analyst

---

## 1. Auth Log — logins, sudo, SSH

| | |
|---|---|
| **Path (Debian/Ubuntu)** | `/var/log/auth.log` |
| **Path (RHEL/CentOS)** | `/var/log/secure` |
| **What it tells you** | Who logged in, who used sudo, failed/successful password attempts |
| **#1 red flag** | Repeated `Failed password` entries → brute force attempt |
| **Command** | `grep "Failed password" /var/log/auth.log` |

---

## 2. Syslog — general system activity

| | |
|---|---|
| **Path (Debian/Ubuntu)** | `/var/log/syslog` |
| **Path (RHEL/CentOS)** | `/var/log/messages` |
| **What it tells you** | Everything else — daemon start/stop, general OS events |
| **#1 red flag** | A service/daemon crashing or restarting repeatedly |
| **Command** | `tail -f /var/log/syslog` |

---

## 3. Cron Logs — scheduled jobs

| | |
|---|---|
| **Path (Debian/Ubuntu)** | `/var/log/cron.log` |
| **Path (RHEL/CentOS)** | `/var/log/cron` |
| **What it tells you** | What scheduled jobs ran, and when |
| **#1 red flag** | A cron job you don't recognize — attackers use cron for persistence |
| **Command** | `cat /etc/cron.d/* /etc/crontab` |

---

## 4. Login History — wtmp / btmp

| | |
|---|---|
| **Path** | `/var/log/wtmp` (successful), `/var/log/btmp` (failed) |
| **View with** | `last` (wtmp), `lastb` (btmp) |
| **What it tells you** | Who logged in/out, from where, and who failed to |
| **#1 red flag** | `lastb` showing a wall of failed logins → brute force |
| **Command** | `last -a \| head -20` |

---

## 5. Bash History — what commands were run

| | |
|---|---|
| **Path** | `~/.bash_history`, root: `/root/.bash_history` |
| **What it tells you** | The exact commands a user (or attacker) typed |
| **#1 red flag** | `curl ... \| bash` (download+execute), or `history -c` (attacker wiping tracks) |
| **Command** | `cat ~/.bash_history` |

---

## Your Tier 1 Triage Chain

When a Linux alert comes in, check in this order:

```
1. auth.log/secure   → did someone log in or use sudo unusually?
2. last / lastb       → confirm it — who, from where, how many failed?
3. bash_history        → if confirmed bad, what did they do?
4. cron                → did they set up persistence?
5. syslog               → fallback if none of the above explains it
```

Learn this chain until it's automatic. That sequence *is* Tier 1 Linux triage.
