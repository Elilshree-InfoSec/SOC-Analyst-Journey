# Linux Command Hacksheet

Command: ls -l
Explanation: Lists files with details such as permissions, owners, and sizes.
Cybersecurity use case: Reveals insecure file permissions. Example: if /etc/shadow is
world-readable, attackers can steal password hashes.

Command: cd
Explanation: Changes current directory.
Cybersecurity use case: Jump directly to critical folders like /var/log during incident response to
save time.

Command: pwd
Explanation: Prints the current working directory.
Cybersecurity use case: Prevents dangerous mistakes such as deleting files in /home/prod when
you thought you were in /tmp.

Command: mkdir -p
Explanation: Creates directories, including parent folders if needed.
Cybersecurity use case: Helps organize forensic evidence. Example: mkdir -p
/analysis/malware_samples for chain-of-custody.

Command: cp
Explanation: Copies files.
Cybersecurity use case: Always back up configurations. Example: cp sshd_config
sshd_config.bak to avoid lockout.

Command: mv
Explanation: Moves or renames files.
Cybersecurity use case: Rotate logs. Example: mv access.log access.old.log to separate old
evidence from new logs.

Command: rm -i
Explanation: Removes files with confirmation prompt.
Cybersecurity use case: Avoid catastrophic errors. A wrong rm -rf can wipe servers. -i adds
safety.

Command: chmod
Explanation: Changes file permissions.
Cybersecurity use case: Secures sensitive files. Example: chmod 600 id_rsa so only you can
read SSH keys.

Command: chown
Explanation: Changes file ownership.
Cybersecurity use case: Prevents privilege abuse. Example: chown www-data:www-data
/var/www/html ensures web files are not root-owned.

Command: cat
Explanation: Displays file contents.
Cybersecurity use case: Quickly inspect files like /etc/passwd to check for unauthorized accounts.

Command: less
Explanation: Scrolls through large files.
Cybersecurity use case: Analyze big log files like /var/log/auth.log to investigate incidents.

Command: grep
Explanation: Searches for text patterns in files.
Cybersecurity use case: Find attacks fast. Example: grep 'Failed password' /var/log/auth.log
shows brute-force attempts.

Command: awk
Explanation: Extracts and processes specific columns in text.
Cybersecurity use case: Filter useful info. Example: awk '{print $1}' access.log lists IPs for threat
hunting.

Command: sed
Explanation: Edits files with search/replace.
Cybersecurity use case: Securely apply fixes. Example: sed -i 's/PermitRootLogin
yes/PermitRootLogin no/' sshd_config disables root login.

Command: echo
Explanation: Outputs or appends text.
Cybersecurity use case: Quick fixes. Example: echo 'nameserver 8.8.8.8' >> /etc/resolv.conf
restores DNS resolution.

Command: top / htop
Explanation: Shows running processes and resource usage.
Cybersecurity use case: Spot abnormal processes like crypto miners or hidden malware.

Command: ps aux | grep
Explanation: Lists all processes and filters by keyword.
Cybersecurity use case: Check for rogue processes such as unauthorized SSH sessions.

Command: kill -9
Explanation: Terminates a process using its ID.
Cybersecurity use case: Stop malicious processes immediately to reduce damage.

Command: uptime
Explanation: Displays system running time.
Cybersecurity use case: Unexpected reboots may signal attacker activity after privilege
escalation.

Command: ping -c 4
Explanation: Checks connectivity to a host.
Cybersecurity use case: Differentiate between network outage and DNS tampering.

Command: curl -I
Explanation: Fetches only HTTP headers.
Cybersecurity use case: Verify missing security headers that attackers exploit.

Command: wget
Explanation: Downloads files from the internet.
Cybersecurity use case: Download forensic tools or known indicators of compromise.

Command: ss -tulnp
Explanation: Lists open ports and associated processes.
Cybersecurity use case: Find hidden backdoors. Example: port 1337 open without reason.

Command: ip a
Explanation: Shows network interfaces and IPs.
Cybersecurity use case: Detect rogue interfaces created by malware.

Command: traceroute
Explanation: Shows path packets take to a host.
Cybersecurity use case: Detect suspicious detours of network traffic.

Command: dig
Explanation: Performs DNS lookups.
Cybersecurity use case: Check SPF/DKIM/DMARC to prevent email spoofing attacks.

Command: whoami
Explanation: Prints the current username.
Cybersecurity use case: Avoid mistakes when logged in as root without realizing it.

Command: id
Explanation: Shows user ID, groups, and roles.
Cybersecurity use case: Confirms whether account has elevated privileges.

Command: systemctl status
Explanation: Shows the status of a service.
Cybersecurity use case: Check if services like SSH are disabled by malware.

Command: man
Explanation: Displays manual for a command.
Cybersecurity use case: Vital reference during incidents when you cannot Google.

Command: apt-get update && apt-get upgrade
Explanation: Updates packages and installs patches.
Cybersecurity use case: Patch management prevents exploits and ransomware attacks.

Command: history
Explanation: Shows previously executed commands.
Cybersecurity use case: Track attacker activity or investigator steps.

Command: clear
Explanation: Clears the terminal screen.
Cybersecurity use case: Keeps terminal organized during incident handling.

Command: which
Explanation: Shows path of a program.
Cybersecurity use case: Detect tampered binaries. Example: ssh replaced in /tmp.

Command: sudo
Explanation: Runs commands with elevated privileges.
Cybersecurity use case: Most critical for defenders. Securely escalate privileges while leaving
logs for auditing.

Command: tail -f
Explanation: Follows a file and prints new lines as they're written.
Cybersecurity use case: Watch a log update live, e.g. tail -f /var/log/auth.log during an active brute-force attempt.

Command: find
Explanation: Searches the filesystem by name, time, size, or other attributes.
Cybersecurity use case: find /var/www -mtime -1 surfaces files modified in the last 24 hours — often how you spot a freshly dropped webshell.

Command: stat
Explanation: Shows full timestamps for a file — created, modified, accessed.
Cybersecurity use case: Confirms exactly when a suspicious file appeared or was last touched, for building an incident timeline.

Command: file
Explanation: Identifies what a file actually is, independent of its extension.
Cybersecurity use case: Catches disguised payloads, like an executable renamed invoice.pdf.

Command: strings
Explanation: Extracts readable text from a binary file.
Cybersecurity use case: Pull hidden URLs, IPs, or hardcoded credentials out of a suspicious binary without executing it.

Command: sha256sum / md5sum
Explanation: Generates a cryptographic hash of a file.
Cybersecurity use case: Compare a binary's hash against threat intel or a known-good baseline to detect tampering.

Command: netstat -tulpn
Explanation: Lists listening ports and active connections with owning processes.
Cybersecurity use case: Older but still-common alternative to ss for spotting unexpected listeners.

Command: lsof -i
Explanation: Lists open network connections and the process holding each one.
Cybersecurity use case: lsof -i :80 or lsof -i | grep ESTABLISHED pinpoints exactly what's talking to a suspicious IP.

Command: tcpdump
Explanation: Captures raw network traffic at the packet level.
Cybersecurity use case: tcpdump -i eth0 -w capture.pcap records live traffic to/from an attacker's IP for later analysis in Wireshark.

Command: pstree / ps auxf
Explanation: Displays running processes as a parent-child tree.
Cybersecurity use case: Reveals a webserver process spawning a shell — a strong sign of successful remote code execution.

Command: hostname
Explanation: Prints the machine's configured name.
Cybersecurity use case: Quick sanity check that you've landed on the correct host during a multi-server incident.

Command: uname -a
Explanation: Shows kernel and OS version.
Cybersecurity use case: Confirms whether the box is running a kernel version vulnerable to a known local privilege escalation exploit.

Command: last
Explanation: Shows a history of user logins, including source IP.
Cybersecurity use case: A login from an unfamiliar country at 3 a.m. is a classic sign of a compromised account.

Command: w
Explanation: Shows who is currently logged in and what they're running.
Cybersecurity use case: Real-time check for an active attacker session, not just historical ones.

Command: crontab -l
Explanation: Lists scheduled jobs for the current user.
Cybersecurity use case: Attackers frequently use cron to re-download payloads or maintain persistence after a reboot.

Command: lsblk
Explanation: Lists block devices attached to the system.
Cybersecurity use case: Flags an unauthorized USB drive or unexpected mounted volume used for data exfiltration.

Command: journalctl -u <service>
Explanation: Shows systemd logs for a specific service.
Cybersecurity use case: journalctl -u ssh gives service-level log detail on modern distros that don't rely solely on flat files in /var/log.

Command: sort / uniq -c
Explanation: Sorts lines and counts duplicate occurrences.
Cybersecurity use case: Chained after grep/awk to rank the most frequent offending IPs in a log.

Command: cut -d: -f1
Explanation: Splits each line on a delimiter and extracts one field.
Cybersecurity use case: cut -d: -f1 /etc/passwd quickly lists all usernames to spot an account that shouldn't exist.

Command: wc -l
Explanation: Counts lines of input.
Cybersecurity use case: Get an exact count of failed login attempts rather than eyeballing a scrolling log.

Command: iptables -A INPUT -s <IP> -j DROP
Explanation: Adds a kernel-level firewall rule dropping traffic from a source.
Cybersecurity use case: Immediately cuts off an attacker's C2 connection during containment.

Command: ufw deny from <IP>
Explanation: Blocks an IP using the Uncomplicated Firewall front-end.
Cybersecurity use case: Faster containment step on systems already managed with UFW.

Command: ip route add blackhole <IP>
Explanation: Silently discards all traffic to/from an address.
Cybersecurity use case: Cuts communication without sending any response, avoiding tipping off automated attacker tooling.

Command: pkill -u <username> / killall <name>
Explanation: Kills all processes owned by a user, or all processes sharing a name.
Cybersecurity use case: Shuts down every instance of a malware process at once rather than hunting individual PIDs.

Command: kill -STOP <PID>
Explanation: Freezes a process without terminating it.
Cybersecurity use case: Halts malicious activity while preserving the process in memory for forensic analysis.

Command: fuser -k <port>/tcp
Explanation: Kills whatever process is bound to a specific port.
Cybersecurity use case: Fast way to shut down a rogue listener (e.g. a reverse shell on port 4444) without knowing its PID.

Command: passwd -l <username>
Explanation: Locks a user's password.
Cybersecurity use case: Blocks password-based login for a compromised account during containment (note: SSH keys may still work).

Command: usermod -L <username> / usermod -s /usr/sbin/nologin <username>
Explanation: Locks an account or strips its login shell.
Cybersecurity use case: Boots an attacker out of active sessions and prevents new ones.

Command: crontab -r -u <username>
Explanation: Deletes all scheduled jobs for a user.
Cybersecurity use case: Strips out any persistence mechanism the attacker planted via cron.

Command: chattr +i <file>
Explanation: Sets the immutable attribute on a file.
Cybersecurity use case: Protects preserved evidence so it can't be altered or deleted — even by root — until the attribute is removed.
