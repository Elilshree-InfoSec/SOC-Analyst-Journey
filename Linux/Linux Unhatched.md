# Cisco - Linux Unhatched (SOC Analyst Reference)

## 1. What Is Linux?
- An **operating system** (like Windows/macOS) that lets other software (apps) run on hardware.
- Runs almost everywhere: desktops/laptops, servers, Android phones, cloud platforms (AWS, Google Cloud), Chromebooks, network gear (Cisco).
- Over half of the internet's web pages are served from Linux-based servers.
- Popular distributions: **Kali** (security/pentesting), **Ubuntu** (development), **Mint** (general use), **Red Hat Enterprise Linux (RHEL)** (enterprise servers).
- Why it matters for SOC/cybersecurity: Linux is used for penetration testing, vulnerability assessment, log analysis, and runs most network/server infrastructure you'll be monitoring.

## 2. GUI vs CLI
- **GUI** (Graphical User Interface): icons/windows — user-friendly but less precise.
- **CLI** (Command Line Interface): text-based commands — faster, more powerful, essential for system administration, troubleshooting, and seeing exactly what a system is doing.

## 3. Basic Command Syntax
```
command [options...] [arguments...]
```
- **Command**: the program name to run (e.g. `ls`).
- **Options**: modify command behavior (usually prefixed with `-`).
- **Arguments**: the target/item the command acts on (e.g. a filename).
- Commands are **case-sensitive** (`ls` works, `LS` fails).
- Multiple options can be combined: `ls -l -r` = `ls -rl` = `ls -lr`.

## 4. Navigating the Filesystem

### pwd — Print Working Directory
```
pwd
```
Shows your current location in the filesystem (e.g. `/home/sysadmin`).

### cd — Change Directory
```
cd [path]
```
- **Absolute path**: starts from root `/` (e.g. `cd /home/sysadmin`). Always begins with `/`.
- **Relative path**: starts from your current location (e.g. `cd Documents`, `cd School/Art`). Does *not* begin with `/`.

### Shortcuts
| Symbol | Meaning |
|---|---|
| `..` | Parent directory (one level up) |
| `.`  | Current directory |
| `~`  | Home directory of current user |

Example:
```
cd ..     # go up one level
cd ~      # jump to home directory
```

## 5. Listing Files — `ls`
```
ls [options] [path]
```
| Option | Effect |
|---|---|
| *(none)* | List files in current directory |
| `-l` | Long listing (permissions, owner, size, date) |
| `-t` | Sort by timestamp (newest first) |
| `-S` | Sort by file size (largest first) |
| `-r` | Reverse the sort order |

### Reading `ls -l` output
```
-rw-r--r-- 1 sysadmin sysadmin 647 Dec 20 2017 hello.sh
```
| Field | Meaning |
|---|---|
| 1st char | File type: `-`=file, `d`=directory, `l`=symlink, `s`=socket, `p`=pipe, `b`/`c`=device |
| Next 9 chars | Permissions (owner/group/other, 3 chars each) |
| Number | Hard link count |
| Name x2 | User owner, then Group owner |
| Number | File size (bytes) |
| Date | Last modified timestamp |
| Name | Filename |

## 6. Permissions

Three permission sets apply, in order of precedence: **owner → group → other**. Only the first matching set applies to you.

| Set | Applies to |
|---|---|
| Owner (1st triplet) | The user who owns the file |
| Group (2nd triplet) | Members of the group that owns the file |
| Other (3rd triplet) | Everyone else |

| Permission | Effect on File | Effect on Directory |
|---|---|---|
| `r` (read) | View/copy contents | List directory contents (non-detailed without `x`) |
| `w` (write) | Modify/overwrite contents | Add/remove files (needs `x` too) |
| `x` (execute) | Run as a program (scripts also need `r`) | Enter/`cd` into the directory |

⚠️ Owner permissions always take priority — even if group/other have more access, the owner is locked to their own permission set.

### chmod — Change Permissions (symbolic method)
```
chmod [set][action][permission] file
```
| Set | Action | Permission |
|---|---|---|
| `u`=user, `g`=group, `o`=other, `a`=all | `+`=add, `-`=remove, `=`=set exactly | `r`, `w`, `x` |

Example — give owner execute permission:
```
chmod u+x hello.sh
```
Only root or the file's owner can change permissions.

### chown — Change Ownership
```
chown [new_owner] file
```
- Changing the **user owner** requires `sudo`/root — even the file's own owner can't give it away.
- Changing **group owner** can be done by root or the file owner.
```
sudo chown root hello.sh
```

## 7. Administrative Access

### su — Switch User
```
su -        # opens a new login shell as root (recommended: use -, -l, or --login)
exit        # return to previous user
```

### sudo — Run a Single Command as Another User (default: root)
```
sudo command
```
- Doesn't open a new shell — just elevates one command.
- Safer than `su` because it's explicit and temporary; password is cached for ~5 minutes between uses.
- Use `-u` to specify a different user besides root.

**su vs sudo**: `su` switches your entire session to root; `sudo` runs just one command with root privileges.

## 8. Viewing File Contents

| Command | Use |
|---|---|
| `cat file` | Show entire file at once (best for small files) |
| `head file` | Show first 10 lines (default) |
| `tail file` | Show last 10 lines (default) |
| `head -n 5 file` | Show first 5 lines |
| `tail -n 5 file` | Show last 5 lines |
| `more` / `less` | Pager commands — scroll through large files (covered in Linux Essentials) |

`head`/`tail` are especially useful for large, frequently-updated files like **system/security logs** — a key SOC analyst use case.

## 9. Copying Files

### cp — Copy Files
```
cp source destination
```
```
cp /etc/passwd .        # copy into current directory (. = current dir)
```
Requirements: execute permission on source's directory, read permission on the source file, and write+execute permission on the destination directory.

### dd — Bit-Level Copy Utility
```
dd if=<input> of=<output> bs=<block size> count=<number of blocks>
```
| Arg | Meaning |
|---|---|
| `if` | Input file/source |
| `of` | Output file/destination |
| `bs` | Block size (K/M/G/T suffixes) |
| `count` | Number of blocks to copy |

Uses: cloning/wiping entire disks or partitions, raw copies to USB/CD, backing up the MBR, creating fixed-size files (e.g. swap files).
```
dd if=/dev/zero of=/tmp/swapex bs=1M count=50   # create a 50MB zero-filled file
dd if=/dev/sda of=/dev/sdb                       # clone one disk to another
```
⚠️ `dd` is powerful and destructive if misused — double-check `if=`/`of=` before running.

## 10. Moving & Renaming Files — `mv`
```
mv source destination
```
- Moves one or more files to a destination directory (last argument = destination).
- Moving a file **within the same directory** effectively **renames** it:
```
mv animals.txt zoo.txt
```
- Requires write + execute permission on **both** the source and destination directories.

## 11. Removing Files — `rm`
```
rm [options] file
```
- Deletes files **permanently** — there is no "recycle bin" in the CLI.
- By default, `rm` will not delete a directory — use `-r` (or `-R`) for recursive deletion:
```
rm -r Work     # deletes the directory and everything inside it
```
- Requires write + execute permission on the containing directory.
- ⚠️ Be extremely careful with `rm -r`, especially combined with wildcards or run as root/`sudo` — a key source of accidental/malicious data loss that SOC analysts investigate.

## 12. Filtering Input — `grep` & Regular Expressions

### grep Basics
```
grep [options] pattern [file]
```
Searches input (or a file) for lines matching a pattern and prints matching lines.
```
grep sysadmin passwd     # find lines containing "sysadmin"
```
- If no file is given, `grep` reads from **STDIN** (keyboard input) — press `Ctrl+D` to end input.
- Wrap patterns in single quotes (`'pattern'`) so the shell doesn't misinterpret special characters.

### Basic Regular Expression Characters
| Character | Meaning |
|---|---|
| `.` | Matches any one single character |
| `[ ]` | Matches any one character listed/ranged inside (e.g. `[0-9]`) |
| `[^ ]` | Matches any one character **NOT** listed inside the brackets |
| `*` | Matches zero or more of the *preceding* character/pattern |
| `^pattern` | Anchors match to the **beginning** of the line (only if `^` is the first character) |
| `pattern$` | Anchors match to the **end** of the line (only if `$` is the last character) |

Examples:
```
grep '^root' /etc/passwd     # lines starting with "root"
grep 'r$' file.txt           # lines ending in "r"
grep 'r..f' file.txt         # r, any 2 chars, then f (e.g. "reef", "roof")
grep '[0-9]' file.txt        # lines containing any digit
grep '[^0-9]' file.txt       # lines containing any non-digit character
grep 're*d' file.txt         # "r", zero+ "e"s, then "d" (matches "rd", "red", "reed"...)
```
⚠️ `[^0-9]` matches lines that **contain** a non-digit — it does NOT mean "lines without any digits."

### Extended Regex (require `egrep` or `grep -E`)
| Character | Meaning |
|---|---|
| `+` | One or more of the previous pattern |
| `?` | Preceding pattern is optional |
| `{ }` | Min/max/exact number of matches |
| `\|` | Alternation (logical OR) |
| `( )` | Grouping |

**SOC relevance**: `grep` + regex is one of your most-used tools for searching logs (auth.log, syslog, etc.) for IPs, usernames, error codes, or attack signatures.

## 13. Viewing Processes — `ps`
```
ps [options]
```
- By default, shows processes running in the **current terminal**.
- `ps -e` — show **every** process on the system.
- `ps -ef` — every process, with full details (options/arguments used to start it).

| Column | Meaning |
|---|---|
| `PID` | Process ID (unique identifier, used to control the process) |
| `TTY` | Terminal the process is running in |
| `TIME` | CPU time used |
| `CMD` | Command that started the process |

- Processes run with the privileges of the user who started them.
- Regular users generally can't control other users' processes; **root** can control any process.
- **SOC relevance**: `ps` is a core tool for spotting suspicious/unexpected running processes during incident investigation.

## 14. Updating User Passwords — `passwd`
```
passwd [options] [user]
```
- Regular users can only change their **own** password; **root** can change anyone's.
```
passwd              # change your own password
sudo passwd sysadmin # root changing another user's password
```
- Check password status with `-S`:
```
passwd -S sysadmin
```
| Field | Meaning |
|---|---|
| Username | The account name |
| Status | `P`=usable password, `L`=locked, `NP`=no password |
| Change Date | Date password was last changed |
| Minimum | Days before password can be changed again |
| Maximum | Days until password expires |
| Warn | Days before expiry the user is warned |
| Inactive | Days after expiry the account stays active |

## 15. I/O Redirection
Three file descriptors ("tracks") every command uses:
| Descriptor | Abbreviation | Meaning |
|---|---|---|
| Standard Input | `STDIN` | Input a command receives (usually keyboard) |
| Standard Output | `STDOUT` | Normal output of a command |
| Standard Error | `STDERR` | Error messages from a failed/incorrect command |

### Redirecting STDOUT to a File
```
command > file      # overwrite file with command's output
command >> file     # append command's output to file
```
Examples:
```
cat food.txt > newfile1.txt        # copies content into a new file (overwrites)
echo "Hello" > newfile1.txt        # replaces file contents with "Hello"
echo "More text" >> newfile1.txt   # appends "More text" without erasing existing content
```
- `echo` simply prints text to the terminal — combined with `>`/`>>` it's a quick way to write/append notes to files.
- Requires **write permission** on the target file to redirect into it.
- **SOC relevance**: redirection is how you'll save command output (e.g. `grep` results from logs) into report files for documentation.

## 16. Text Editor — `vi` / `vim`
`vi` (or its modern version `vim`) is the universal Linux text editor — available on virtually every distro, works in both CLI and GUI, and its core commands have stayed stable for decades.
```
vi filename
```

### Three Modes
| Mode | Purpose |
|---|---|
| Command mode | Default mode; move around, delete/copy/paste text, enter other modes. Press `Esc` to return here. |
| Insert mode | Type/add text into the document |
| Ex mode | Save, quit, and other file-level commands (enter with `:`) |

### Movement (Command Mode)
`[count] motion` — e.g. `5h` = move left 5 characters, `3w` = move 3 words forward.

| Key | Moves |
|---|---|
| `h` `j` `k` `l` | left / down / up / right |
| `w` / `b` | one word forward / back |
| `^` / `$` | beginning / end of line |
| `5G` | go to line 5 |
| `gg` or `1G` | first line |
| `G` | last line |

### Actions (Command Mode)
| Action | Vi Key | Meaning |
|---|---|---|
| Cut | `d` | delete (e.g. `dd`=delete line, `dw`=delete word) |
| Copy | `y` | yank (e.g. `yy`=yank line) |
| Paste | `p` / `P` | put after / before cursor |
| Change | `c` | delete + switch to insert mode (e.g. `cw`=change word) |

### Entering Insert Mode
| Key | Effect |
|---|---|
| `i` / `I` | insert before cursor / at start of line |
| `a` / `A` | insert after cursor / at end of line |
| `o` / `O` | open new line below / above cursor |

### Searching
```
/pattern      # search forward, Enter to search
?pattern      # search backward
n / N         # repeat search forward / backward
```

### Ex Mode Commands (press `:` from command mode)
| Command | Effect |
|---|---|
| `:w` | Save (write) file |
| `:w filename` | Save a copy as filename |
| `:q` | Quit (fails if unsaved changes) |
| `:q!` | Quit without saving (force) |
| `:wq` or `ZZ` | Save and quit |

## 17. Network Configuration

### ifconfig — View/Configure Network Interfaces
```
ifconfig
```
Shows interfaces (e.g. `eth0`), their IPv4 address, and status (`UP` = active). `iwconfig` is the equivalent for wireless interfaces. `lo` is the **loopback** device, used when the system sends network data to itself.

### ping — Test Connectivity
```
ping [-c count] host_or_ip
```
Sends ICMP packets to test if a remote host is reachable.
```
ping -c 4 192.168.1.2      # send exactly 4 pings
```
- Success = replies received; failure = "Destination Host Unreachable."
- Some hosts/networks are configured to **not** respond to pings as a security measure, so a failed ping doesn't always mean the host is down.
- Works with IPs or hostnames (e.g. `ping yahoo.com` also confirms DNS resolution is working).
- **SOC relevance**: `ifconfig`/`ip a` and `ping` are baseline tools for verifying network reachability during triage.

## 18. Package Management
Debian/Ubuntu systems use `dpkg` (low-level) and `apt-get` (front-end, easier to use). Most package commands require `sudo`.

| Command | Purpose |
|---|---|
| `sudo apt-get update` | Refresh the local list of available packages (run before installing) |
| `apt-cache search keyword` | Search for a package by keyword |
| `sudo apt-get install package` | Install a package (or update it if already installed) |
| `sudo apt-get upgrade` | Upgrade all installed packages |
| `sudo apt-get remove package` | Remove a package, keep its config files |
| `sudo apt-get purge package` | Remove a package **and** its config files completely |

**SOC relevance**: keeping packages updated (`update`/`upgrade`) is basic patch-management hygiene; knowing what's installed helps identify unauthorized/malicious software.

## 19. Shutting Down — `shutdown`
```
shutdown [options] time [message]
```
Requires root access. Safely brings the system down, warning logged-in users and blocking new logins in the final 5 minutes.

| Time format | Meaning |
|---|---|
| `now` | Shut down immediately |
| `hh:mm` | Shut down at a specific time |
| `+minutes` | Shut down after a delay (e.g. `+1`) |

```
sudo shutdown now
sudo shutdown +5 "Maintenance in 5 minutes"
```
Use the `date` command to check current system time/timezone (often UTC) before scheduling a shutdown.

---

## Quick-Reference Cheat Sheet
| Task | Command |
|---|---|
| Where am I? | `pwd` |
| List files | `ls -l` |
| Change directory | `cd path` |
| Go home | `cd ~` |
| Go up one level | `cd ..` |
| View a file | `cat file` |
| View first/last lines | `head file` / `tail file` |
| Copy a file | `cp source dest` |
| Change permissions | `chmod u+x file` |
| Change ownership | `sudo chown user file` |
| Become root | `su -` |
| Run one command as root | `sudo command` |
| Move/rename a file | `mv source dest` |
| Delete a file | `rm file` |
| Delete a directory | `rm -r directory` |
| Search text for a pattern | `grep 'pattern' file` |
| List running processes | `ps -ef` |
| Change a password | `passwd [user]` |
| Redirect output (overwrite) | `command > file` |
| Redirect output (append) | `command >> file` |
| Edit a file | `vi file` |
| Save & quit vi | `:wq` or `ZZ` |
| Quit vi without saving | `:q!` |
| Check network interfaces | `ifconfig` |
| Test connectivity | `ping -c 4 host` |
| Update package list | `sudo apt-get update` |
| Install a package | `sudo apt-get install pkg` |
| Shut down now | `sudo shutdown now` |
