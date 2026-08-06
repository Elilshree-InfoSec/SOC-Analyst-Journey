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
