# 🐧 Linux Fundamentals Part 1

## 🌍 Where is Linux Used?

* 🌐 Web servers
* 📱 Android phones
* 🚗 Car systems
* 🏪 Point of Sale (POS)
* 🏭 Industrial systems
* ☁️ Cloud infrastructure
* 🔐 Cybersecurity (SOC, Pentesting, DFIR)

**Why Linux?**

* Lightweight
* Secure
* Stable
* Open Source
* Highly customizable

---

# 💻 Why Learn Linux for SOC?

* Analyze logs
* Investigate incidents
* Run security tools
* Manage Linux servers
* Automate tasks with scripts

---

# 🖥️ Terminal (CLI)

* Primary way to interact with Linux.
* Commands = Instructions given to the OS.
* Output = Result returned by the command.

---

# 📚 Essential Commands

| Command  | Purpose                |
| -------- | ---------------------- |
| `whoami` | Show current user      |
| `pwd`    | Show current directory |
| `ls`     | List files/folders     |
| `cd`     | Change directory       |
| `cat`    | Display file contents  |
| `echo`   | Print text             |

---

# 🔍 Searching

## `find`

Search for files by name.

```bash
find -name passwords.txt
```

## `grep`

Search for text inside files.

```bash
grep "THM" access.log
```

**SOC Use:** Search logs for:

* Failed logins
* IP addresses
* Usernames
* Errors
* Indicators of Compromise (IOCs)

---

# 📄 Output Redirection

| Operator | Purpose        |
| -------- | -------------- |
| `>`      | Overwrite file |
| `>>`     | Append to file |

Example:

```bash
echo "TryHackMe" > notes.txt
echo "Linux" >> notes.txt
```

---

# ⚙️ Command Operators

| Operator | Purpose                                    |
| -------- | ------------------------------------------ |
| `&`      | Run command in background                  |
| `&&`     | Run next command only if previous succeeds |

Example:

```bash
mkdir logs && cd logs
```

---

# 🔐 SOC Analyst Key Takeaways

* Linux is the dominant OS for servers and cybersecurity.
* Learn terminal navigation before advanced topics.
* `grep` is one of the most important commands for log analysis.
* `find` helps locate files quickly.
* Redirection (`>` and `>>`) is useful for saving investigation results.
* These basic commands are used daily by SOC Analysts, System Administrators, and Penetration Testers.

---

#💭 Personal Reflection

This room helped me understand the basics of Linux and why it is essential in cybersecurity. I gained confidence using the terminal and learned core commands that will support my future SOC analyst tasks, such as navigating the system and searching through logs.

---

- Room Completed: ✅
- Difficulty: Easy
- Time Taken: 15 minutes
