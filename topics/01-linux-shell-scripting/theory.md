# Linux & Shell Scripting — Theory Questions

---

## Q1: What is Linux, and why is it so important in DevOps?

**Answer:**
Linux is a free, open-source operating system (the software that manages your computer's hardware and lets you run programs). It's based on Unix and was created by Linus Torvalds in 1991.

It's important in DevOps because:
- **Most servers run Linux** — over 90% of cloud servers and almost all containers use Linux.
- **It's free and open-source** — no licensing costs, and you can customize it.
- **It's stable and secure** — Linux servers can run for years without restarting.
- **Automation-friendly** — everything can be done from the command line, which makes scripting easy.
- **All major DevOps tools** (Docker, Kubernetes, Ansible, etc.) are built for Linux first.

---

## Q2: What is the difference between a Linux distribution and the Linux kernel?

**Answer:**
- **Linux kernel** — The core part of the operating system. It manages hardware (CPU, memory, disk) and provides basic services to programs. Think of it as the engine of a car.
- **Linux distribution (distro)** — A complete operating system built around the kernel. It includes the kernel plus additional software like package managers, desktop environments, and utilities. Think of it as the whole car (engine + body + seats + wheels).

Popular distros:
- **Ubuntu / Debian** — User-friendly, great for beginners and servers
- **CentOS / Rocky Linux / AlmaLinux** — Enterprise-focused, stable
- **Amazon Linux** — Optimized for AWS
- **Alpine** — Extremely small, popular for Docker containers

---

## Q3: What is the Linux file system hierarchy? Name the key directories.

**Answer:**
Linux organizes files in a tree structure starting from `/` (root). Key directories:

| Directory | Purpose |
|-----------|---------|
| `/` | Root — the top of the file system tree |
| `/home` | User home directories (e.g., `/home/john`) |
| `/root` | Home directory for the root (admin) user |
| `/etc` | Configuration files for the system and applications |
| `/var` | Variable data — logs (`/var/log`), temporary files |
| `/tmp` | Temporary files (deleted on reboot) |
| `/opt` | Optional/third-party software |
| `/usr` | User programs and utilities |
| `/bin` | Essential command binaries (ls, cp, etc.) |
| `/sbin` | System binaries (for admin tasks) |
| `/dev` | Device files (representing hardware) |
| `/proc` | Virtual filesystem with process and system info |
| `/mnt` and `/media` | Mount points for external drives |

---

## Q4: What are file permissions in Linux? How do you read and set them?

**Answer:**
Every file and directory in Linux has three types of permissions for three groups of users:

**Permission types:**
- **r (read)** — Can view the file's contents
- **w (write)** — Can modify the file
- **x (execute)** — Can run the file as a program

**User groups:**
- **Owner (u)** — The user who created the file
- **Group (g)** — Users in the file's group
- **Others (o)** — Everyone else

**Reading permissions:**
```bash
ls -l myfile.txt
# Output: -rwxr-xr-- 1 john devops 1024 Mar 1 10:00 myfile.txt
#          ^^^         Owner permissions: read + write + execute
#             ^^^      Group permissions: read + execute
#                ^^^   Others permissions: read only
```

**Setting permissions with numbers (octal):**
- r = 4, w = 2, x = 1
- `chmod 755 myfile.txt` → Owner: rwx (7), Group: r-x (5), Others: r-x (5)
- `chmod 644 myfile.txt` → Owner: rw- (6), Group: r-- (4), Others: r-- (4)

**Setting permissions with letters:**
```bash
chmod u+x script.sh    # Add execute permission for owner
chmod g-w file.txt      # Remove write permission for group
chmod o=r file.txt      # Set others to read-only
```

---

## Q5: What is the difference between a process and a thread?

**Answer:**
- **Process** — An independent running program. It has its own memory space, its own resources, and runs separately from other processes. Example: When you open a web browser, that's a process.
- **Thread** — A smaller unit of execution *inside* a process. Threads share the same memory space and resources of their parent process. Example: In a web browser, one thread loads a page while another plays a video.

**Key differences:**

| Feature | Process | Thread |
|---------|---------|--------|
| Memory | Separate memory space | Shared memory space |
| Creation | Slower (more resources needed) | Faster (lightweight) |
| Communication | Harder (needs IPC mechanisms) | Easier (shared memory) |
| Crash impact | One process crash doesn't affect others | One thread crash can crash the whole process |

**Useful commands:**
```bash
ps aux          # List all running processes
top             # Real-time process monitoring
htop            # Better version of top (if installed)
kill PID        # Send termination signal to a process
kill -9 PID     # Force kill a process
```

---

## Q6: What is a shell? What's the difference between Bash and sh?

**Answer:**
A **shell** is a program that takes your typed commands and tells the operating system what to do. It's the interface between you and the Linux kernel.

- **sh (Bourne Shell)** — The original Unix shell. Simple and available on almost every system.
- **Bash (Bourne Again Shell)** — An improved version of sh. It adds features like command history, tab completion, arrays, and better scripting capabilities. It's the default shell on most Linux systems.
- **Other shells:** Zsh (popular on macOS), Fish (user-friendly), Dash (fast, minimal).

**How to check your current shell:**
```bash
echo $SHELL       # Shows default shell
echo $0           # Shows current shell
```

---

## Q7: What are environment variables? How do you set them?

**Answer:**
Environment variables are named values stored in the shell's memory that programs can read. They configure how programs behave without changing code.

**Common environment variables:**
```bash
echo $HOME        # Your home directory (/home/username)
echo $PATH        # Directories where the system looks for commands
echo $USER        # Current username
echo $SHELL       # Current shell
echo $PWD         # Current working directory
```

**Setting environment variables:**
```bash
# Temporary (only for current session)
export MY_VAR="hello"

# Permanent (add to ~/.bashrc or ~/.bash_profile)
echo 'export MY_VAR="hello"' >> ~/.bashrc
source ~/.bashrc    # Reload the file to apply changes

# For a single command only
MY_VAR="hello" ./my_script.sh
```

**Why they matter in DevOps:**
- Store configuration (database URLs, API keys)
- CI/CD pipelines use them heavily for secrets and settings
- Docker containers use environment variables for configuration

---

## Q8: What is the difference between `>` and `>>` in Linux?

**Answer:**
Both are **output redirection operators** — they send command output to a file instead of the screen.

- **`>`** — **Overwrites** the file. If the file exists, its contents are replaced.
- **`>>`** — **Appends** to the file. New content is added at the end.

```bash
echo "first line" > file.txt     # file.txt now contains: "first line"
echo "second line" > file.txt    # file.txt now contains: "second line" (first line is gone!)

echo "first line" > file.txt     # file.txt contains: "first line"
echo "second line" >> file.txt   # file.txt contains: "first line" AND "second line"
```

**Other useful redirections:**
```bash
command 2> error.log        # Redirect errors only
command > output.log 2>&1   # Redirect both output and errors
command < input.txt         # Use file as input
command | another_command   # Pipe: send output of one command as input to another
```

---

## Q9: What is a pipe (`|`) in Linux?

**Answer:**
A pipe takes the **output of one command** and sends it as **input to another command**. It lets you chain commands together.

```bash
# Without pipe — two separate steps
ls -la > filelist.txt
grep ".log" filelist.txt

# With pipe — one step
ls -la | grep ".log"

# More examples
cat /var/log/syslog | grep "error" | wc -l     # Count error lines in log
ps aux | grep nginx                              # Find nginx processes
history | tail -20                               # Show last 20 commands
```

**Think of it like an assembly line:** each command does one job and passes the result to the next.

---

## Q10: What is `grep` and how do you use it?

**Answer:**
`grep` (Global Regular Expression Print) searches for text patterns in files or command output. It's one of the most-used commands in DevOps.

```bash
# Basic usage
grep "error" logfile.txt              # Find lines containing "error"
grep -i "error" logfile.txt           # Case-insensitive search
grep -r "TODO" /project/              # Search recursively in all files
grep -n "error" logfile.txt           # Show line numbers
grep -c "error" logfile.txt           # Count matching lines
grep -v "debug" logfile.txt           # Show lines that DON'T match (invert)
grep -l "password" /etc/*             # List only filenames that match

# With regular expressions
grep "^Start" logfile.txt             # Lines starting with "Start"
grep "end$" logfile.txt               # Lines ending with "end"
grep -E "error|warning" logfile.txt   # Lines with "error" OR "warning"
```

---

## Q11: What is `sed` and `awk`? When would you use each?

**Answer:**

**`sed` (Stream Editor)** — Used to find and replace text in files or streams.
```bash
# Replace first occurrence on each line
sed 's/old/new/' file.txt

# Replace ALL occurrences on each line
sed 's/old/new/g' file.txt

# Edit file in place (modify the actual file)
sed -i 's/old/new/g' file.txt

# Delete lines matching a pattern
sed '/pattern/d' file.txt

# Replace on a specific line (line 3)
sed '3s/old/new/' file.txt
```

**`awk`** — Used to process and extract data from structured text (like columns).
```bash
# Print the second column
awk '{print $2}' file.txt

# Print lines where column 3 is greater than 100
awk '$3 > 100' file.txt

# Use a custom separator (comma)
awk -F',' '{print $1, $3}' data.csv

# Sum values in column 2
awk '{sum += $2} END {print sum}' file.txt
```

**When to use which:**
- **sed** → When you need to find and replace text
- **awk** → When you need to extract or process columns of data

---

## Q12: What is a cron job? How do you schedule one?

**Answer:**
A **cron job** is a scheduled task that runs automatically at specified times. The `cron` daemon (background service) checks every minute if any jobs need to run.

**Cron syntax:**
```
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └── Day of week (0-7, 0 and 7 = Sunday)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

**Examples:**
```bash
# Edit your cron jobs
crontab -e

# Run a backup every day at 2:30 AM
30 2 * * * /scripts/backup.sh

# Run every 5 minutes
*/5 * * * * /scripts/health-check.sh

# Run every Monday at 9 AM
0 9 * * 1 /scripts/weekly-report.sh

# Run on the 1st of every month at midnight
0 0 1 * * /scripts/monthly-cleanup.sh

# List current cron jobs
crontab -l
```

**DevOps use cases:** Log rotation, backups, certificate renewal, cleanup scripts, health checks.

---

## Q13: What is the difference between soft links and hard links?

**Answer:**

- **Hard link** — A direct pointer to the file's data on disk. The file and the hard link are equal — deleting one doesn't affect the other. They share the same data.
- **Soft link (symbolic link / symlink)** — A shortcut that points to the file's *name* (path). If the original file is deleted, the symlink breaks.

```bash
# Create a hard link
ln original.txt hardlink.txt

# Create a soft link
ln -s original.txt softlink.txt

# Check links
ls -li    # The 'i' shows inode numbers — hard links share the same inode
```

| Feature | Hard Link | Soft Link |
|---------|-----------|-----------|
| Points to | File's data (inode) | File's path (name) |
| Breaks if original deleted? | No | Yes |
| Works across filesystems? | No | Yes |
| Works for directories? | No | Yes |

---

## Q14: What is a Bash script? What are the key elements?

**Answer:**
A Bash script is a text file containing a series of commands that Bash executes in order. It automates repetitive tasks.

**Key elements:**
```bash
#!/bin/bash
# ^ Shebang line — tells the system to use Bash to run this script

# Variables
NAME="DevOps"
echo "Hello, $NAME"

# Command-line arguments
echo "First argument: $1"
echo "All arguments: $@"
echo "Number of arguments: $#"

# Conditional (if/else)
if [ -f "/etc/hosts" ]; then
    echo "File exists"
else
    echo "File not found"
fi

# Loops
for server in web1 web2 web3; do
    echo "Checking $server"
    ping -c 1 "$server"
done

# While loop
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done

# Functions
check_disk() {
    usage=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
    if [ "$usage" -gt 80 ]; then
        echo "WARNING: Disk usage is ${usage}%"
    fi
}
check_disk

# Exit codes
# 0 = success, non-zero = failure
exit 0
```

**Making a script executable:**
```bash
chmod +x myscript.sh
./myscript.sh
```

---

## Q15: What is the `systemctl` command? What is systemd?

**Answer:**
**systemd** is the system and service manager for most modern Linux distributions. It starts services (like web servers, databases) when the system boots and manages them while running.

**`systemctl`** is the command to interact with systemd:

```bash
# Start/stop/restart a service
systemctl start nginx
systemctl stop nginx
systemctl restart nginx

# Enable a service to start on boot
systemctl enable nginx

# Disable a service from starting on boot
systemctl disable nginx

# Check service status
systemctl status nginx

# List all running services
systemctl list-units --type=service --state=running

# View service logs
journalctl -u nginx -f    # Follow logs in real time
```

**DevOps relevance:** You'll manage services like Docker, Jenkins, Prometheus, and web servers using systemctl.

---

## Q16: What is the difference between `apt` and `yum`?

**Answer:**
Both are **package managers** — tools that install, update, and remove software. The difference is which Linux distro uses which:

| Feature | `apt` (Advanced Package Tool) | `yum` / `dnf` |
|---------|------|------|
| Used by | Debian, Ubuntu | CentOS, RHEL, Fedora, Amazon Linux |
| Package format | `.deb` | `.rpm` |
| Install | `apt install nginx` | `yum install nginx` |
| Update all | `apt update && apt upgrade` | `yum update` |
| Remove | `apt remove nginx` | `yum remove nginx` |
| Search | `apt search nginx` | `yum search nginx` |

**Note:** `dnf` is the modern replacement for `yum` on newer RHEL/Fedora systems. Commands are very similar.

---

## Q17: What is SSH and how does key-based authentication work?

**Answer:**
**SSH (Secure Shell)** is a protocol for securely connecting to remote machines over a network. All communication is encrypted.

**Password-based authentication:**
```bash
ssh username@server-ip    # You'll be prompted for a password
```

**Key-based authentication (more secure, no password needed):**

1. **Generate a key pair** on your local machine:
   ```bash
   ssh-keygen -t ed25519 -C "your-email@example.com"
   # Creates two files:
   # ~/.ssh/id_ed25519       (private key — NEVER share this)
   # ~/.ssh/id_ed25519.pub   (public key — safe to share)
   ```

2. **Copy the public key** to the remote server:
   ```bash
   ssh-copy-id username@server-ip
   ```

3. **Connect without a password:**
   ```bash
   ssh username@server-ip    # No password prompt!
   ```

**How it works:** The server has your public key. When you connect, it sends a challenge that only your private key can solve. If the answer is correct, you're in. No password travels over the network.

---

## Q18: What are `stdin`, `stdout`, and `stderr`?

**Answer:**
These are the three standard data streams in Linux:

| Stream | Number | Description | Example |
|--------|--------|-------------|---------|
| **stdin** (standard input) | 0 | Data going INTO a program | Keyboard input, piped data |
| **stdout** (standard output) | 1 | Normal output FROM a program | Command results |
| **stderr** (standard error) | 2 | Error messages FROM a program | Error messages |

```bash
# Redirect stdout to a file
ls > output.txt

# Redirect stderr to a file
ls /nonexistent 2> errors.txt

# Redirect both stdout and stderr to the same file
ls /home /nonexistent > all.txt 2>&1

# Redirect both (modern syntax)
ls /home /nonexistent &> all.txt

# Discard output completely
command > /dev/null 2>&1
```

---

## Q19: What is the `find` command and how is it different from `locate`?

**Answer:**

**`find`** — Searches the file system in real time. Slower but always up-to-date.
```bash
find /var/log -name "*.log"                    # Find all .log files
find / -type f -size +100M                     # Files larger than 100MB
find /home -user john                          # Files owned by john
find /tmp -mtime +7 -delete                    # Delete files older than 7 days
find . -name "*.sh" -exec chmod +x {} \;       # Make all .sh files executable
```

**`locate`** — Searches a pre-built database. Very fast but might be outdated.
```bash
locate nginx.conf           # Instantly find the file
sudo updatedb               # Update the database manually
```

| Feature | `find` | `locate` |
|---------|--------|----------|
| Speed | Slower (scans filesystem) | Very fast (uses database) |
| Accuracy | Always current | May be outdated |
| Flexibility | Many filters (size, date, owner) | Name-based only |

---

## Q20: What is `xargs` and when would you use it?

**Answer:**
`xargs` takes input (usually from a pipe) and converts it into arguments for another command. It's useful when you need to pass a list of items to a command.

```bash
# Delete all .tmp files found by find
find /tmp -name "*.tmp" | xargs rm

# Run a command on each line of a file
cat servers.txt | xargs -I {} ssh {} "uptime"

# Parallel execution (run 4 at a time)
cat urls.txt | xargs -P 4 -I {} curl -s {}

# Safe handling of filenames with spaces
find . -name "*.log" -print0 | xargs -0 rm
```

**Without xargs:** `find . -name "*.log" -exec rm {} \;` (slower — runs `rm` once per file)
**With xargs:** `find . -name "*.log" | xargs rm` (faster — runs `rm` with multiple files at once)

---

## Q21: What is the `/proc` filesystem?

**Answer:**
`/proc` is a **virtual filesystem** — it doesn't exist on disk. Instead, the kernel creates it in memory to provide information about running processes and system status.

```bash
cat /proc/cpuinfo          # CPU information
cat /proc/meminfo          # Memory information
cat /proc/version          # Kernel version
cat /proc/uptime           # System uptime in seconds
cat /proc/loadavg          # System load averages
ls /proc/1234/             # Information about process with PID 1234
cat /proc/1234/status      # Status of that process
cat /proc/1234/cmdline     # Command that started the process
```

**DevOps use:** Monitoring scripts often read from `/proc` to check system health.

---

## Q22: What are `top`, `htop`, `vmstat`, `iostat`, and `netstat` used for?

**Answer:**
These are system monitoring commands:

| Command | What It Shows | Key Info |
|---------|--------------|----------|
| `top` | Real-time process list | CPU%, memory%, running processes |
| `htop` | Better version of `top` | Color-coded, easier to read, mouse support |
| `vmstat` | Virtual memory statistics | Memory, swap, CPU, I/O activity |
| `iostat` | Disk I/O statistics | Read/write speeds, disk utilization |
| `netstat` / `ss` | Network connections | Open ports, active connections |
| `df -h` | Disk space usage | Free/used space per filesystem |
| `du -sh *` | Directory sizes | How much space each folder uses |
| `free -h` | Memory usage | Total, used, free, cached RAM |

```bash
top -bn1 | head -20        # Snapshot of top (non-interactive)
vmstat 1 5                  # Report every 1 second, 5 times
iostat -x 1                 # Extended disk stats every second
ss -tulnp                   # Show listening ports with process names
```

---

## Q23: What is `tar` and how do you use it for backups?

**Answer:**
`tar` (Tape Archive) bundles multiple files into a single archive file, optionally compressed.

```bash
# Create a compressed archive
tar -czvf backup.tar.gz /path/to/directory
# c = create, z = gzip compress, v = verbose, f = filename

# Extract an archive
tar -xzvf backup.tar.gz
# x = extract

# Extract to a specific directory
tar -xzvf backup.tar.gz -C /destination/

# List contents without extracting
tar -tzvf backup.tar.gz

# Create with bzip2 compression (smaller but slower)
tar -cjvf backup.tar.bz2 /path/to/directory
```

**Backup script example:**
```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
tar -czvf "/backups/app-${DATE}.tar.gz" /var/www/app/
find /backups -name "*.tar.gz" -mtime +30 -delete  # Remove backups older than 30 days
```

---

## Q24: What is the difference between `su` and `sudo`?

**Answer:**

- **`su` (Switch User)** — Switches to another user account entirely. You need that user's password.
  ```bash
  su -            # Switch to root (need root password)
  su - john       # Switch to john (need john's password)
  ```

- **`sudo` (Superuser Do)** — Runs a single command as root (or another user) using YOUR password. You must be in the sudoers file.
  ```bash
  sudo apt update           # Run one command as root
  sudo -u john command      # Run as john
  ```

**Why `sudo` is preferred:**
- You don't need to share the root password
- Every `sudo` command is logged (audit trail)
- You can limit which commands each user can run
- It follows the principle of least privilege

**Sudoers file:** `/etc/sudoers` (edit with `visudo` only)
```bash
# Allow john to run any command
john ALL=(ALL:ALL) ALL

# Allow john to restart nginx only
john ALL=(ALL) /usr/bin/systemctl restart nginx
```

---

## Q25: What are signals in Linux? Name the most common ones.

**Answer:**
Signals are notifications sent to processes to tell them to do something (like stop or restart).

| Signal | Number | Meaning | Use |
|--------|--------|---------|-----|
| `SIGHUP` | 1 | Hangup | Reload configuration |
| `SIGINT` | 2 | Interrupt | Ctrl+C — politely ask to stop |
| `SIGKILL` | 9 | Kill | Force stop — cannot be caught or ignored |
| `SIGTERM` | 15 | Terminate | Default kill signal — ask to stop gracefully |
| `SIGSTOP` | 19 | Stop | Pause the process |
| `SIGCONT` | 18 | Continue | Resume a paused process |

```bash
kill PID            # Sends SIGTERM (graceful stop)
kill -9 PID         # Sends SIGKILL (force stop)
kill -HUP PID       # Sends SIGHUP (reload config)
killall nginx       # Kill all processes named nginx
pkill -f "python"   # Kill processes matching a pattern
```

**Best practice:** Always try `SIGTERM` first. Only use `SIGKILL` as a last resort, because `SIGKILL` doesn't let the process clean up (close files, release resources).