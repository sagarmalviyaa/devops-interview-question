# Linux & Shell Scripting — Scenario-Based Questions

---

## S1: A server is running very slowly. How would you diagnose the problem?

**Answer:**

**Step-by-step approach:**

1. **Check system load:**
   ```bash
   uptime
   # Shows load averages for 1, 5, and 15 minutes
   # If load > number of CPU cores, the system is overloaded
   ```

2. **Check CPU and memory usage:**
   ```bash
   top
   # Look for processes using high %CPU or %MEM
   # Press 'P' to sort by CPU, 'M' to sort by memory
   ```

3. **Check available memory:**
   ```bash
   free -h
   # If "available" is very low, you're running out of RAM
   ```

4. **Check disk space:**
   ```bash
   df -h
   # If any filesystem is at 100%, that's likely the problem
   ```

5. **Check disk I/O:**
   ```bash
   iostat -x 1 5
   # If %util is near 100%, the disk is a bottleneck
   ```

6. **Check for too many processes:**
   ```bash
   ps aux | wc -l
   # Hundreds of zombie or stuck processes can slow things down
   ```

7. **Check recent logs:**
   ```bash
   dmesg | tail -50              # Kernel messages
   journalctl -p err --since "1 hour ago"  # Recent errors
   ```

8. **Check network:**
   ```bash
   ss -s                         # Connection summary
   netstat -an | grep ESTABLISHED | wc -l  # Active connections
   ```

**Common causes:** Runaway process eating CPU, out of memory (OOM), full disk, too many open connections, disk I/O bottleneck.

---

## S2: A disk is 100% full. How do you find what's using the space and fix it?

**Answer:**

1. **Confirm the problem:**
   ```bash
   df -h
   # Find which filesystem is full
   ```

2. **Find the largest directories:**
   ```bash
   du -sh /* 2>/dev/null | sort -rh | head -10
   # Then drill deeper into the largest directory
   du -sh /var/* | sort -rh | head -10
   ```

3. **Common culprits:**
   - `/var/log` — Log files that grew too large
   - `/tmp` — Temporary files not cleaned up
   - `/var/lib/docker` — Docker images and containers
   - Old backups or core dumps

4. **Quick fixes:**
   ```bash
   # Clear old logs
   find /var/log -name "*.log" -mtime +30 -delete
   
   # Truncate a large log file (keeps the file but empties it)
   > /var/log/large-app.log
   
   # Clean Docker resources
   docker system prune -af
   
   # Remove old packages
   apt autoremove    # Debian/Ubuntu
   yum autoremove    # CentOS/RHEL
   ```

5. **Prevent it from happening again:**
   - Set up log rotation (`logrotate`)
   - Add disk space monitoring with alerts
   - Set up automated cleanup cron jobs

---

## S3: You need to find all files modified in the last 24 hours that contain the word "ERROR". How?

**Answer:**

```bash
# Method 1: Using find + grep
find /var/log -type f -mtime -1 -exec grep -l "ERROR" {} \;

# Method 2: Faster with xargs
find /var/log -type f -mtime -1 | xargs grep -l "ERROR" 2>/dev/null

# Method 3: Also show the matching lines
find /var/log -type f -mtime -1 -exec grep -Hn "ERROR" {} \;
# -H shows filename, -n shows line number

# Method 4: If you also want to count errors per file
find /var/log -type f -mtime -1 -exec grep -c "ERROR" {} \; | grep -v ":0$"
```

**Explanation of flags:**
- `-type f` — Only regular files (not directories)
- `-mtime -1` — Modified in the last 1 day
- `-l` — Only show filenames (not the matching lines)
- `-H` — Show filename with each match
- `-n` — Show line numbers

---

## S4: Write a script that checks if a list of servers is reachable and sends an alert if any are down.

**Answer:**

```bash
#!/bin/bash
# health-check.sh — Check server availability

SERVERS=("web1.example.com" "web2.example.com" "db1.example.com" "cache1.example.com")
ALERT_EMAIL="ops-team@example.com"
LOG_FILE="/var/log/health-check.log"
DOWN_SERVERS=()

echo "$(date): Starting health check" >> "$LOG_FILE"

for server in "${SERVERS[@]}"; do
    if ping -c 2 -W 3 "$server" > /dev/null 2>&1; then
        echo "$(date): $server is UP" >> "$LOG_FILE"
    else
        echo "$(date): $server is DOWN!" >> "$LOG_FILE"
        DOWN_SERVERS+=("$server")
    fi
done

# Send alert if any servers are down
if [ ${#DOWN_SERVERS[@]} -gt 0 ]; then
    MESSAGE="ALERT: The following servers are unreachable:\n"
    for s in "${DOWN_SERVERS[@]}"; do
        MESSAGE+="  - $s\n"
    done
    MESSAGE+="\nTime: $(date)\nPlease investigate immediately."
    
    echo -e "$MESSAGE" | mail -s "SERVER DOWN ALERT" "$ALERT_EMAIL"
    echo "$(date): Alert sent for ${#DOWN_SERVERS[@]} down server(s)" >> "$LOG_FILE"
fi

echo "$(date): Health check complete" >> "$LOG_FILE"
```

**Schedule it with cron:**
```bash
# Run every 5 minutes
*/5 * * * * /scripts/health-check.sh
```

---

## S5: A process is consuming 100% CPU. How do you identify and handle it?

**Answer:**

1. **Identify the process:**
   ```bash
   top -bn1 | head -15
   # Or sort by CPU
   ps aux --sort=-%cpu | head -10
   ```

2. **Get more details about the process:**
   ```bash
   # If PID is 12345
   ps -p 12345 -o pid,ppid,user,cmd,%cpu,%mem
   
   # Check what files it has open
   lsof -p 12345
   
   # Check what it's doing (system calls)
   strace -p 12345 -c    # Summary of system calls
   ```

3. **Decide what to do:**
   - **If it's a known application bug:** Restart the service
     ```bash
     systemctl restart my-app
     ```
   - **If it's a runaway process:** Kill it gracefully first
     ```bash
     kill 12345          # SIGTERM — ask nicely
     sleep 5
     kill -9 12345       # SIGKILL — force if still running
     ```
   - **If it's a legitimate workload:** Consider if you need more CPU resources

4. **Investigate the root cause:**
   ```bash
   # Check application logs
   journalctl -u my-app --since "1 hour ago"
   
   # Check if it happens regularly
   sar -u 1 10    # CPU usage over time
   ```

5. **Prevent recurrence:**
   - Set up CPU limits with `cgroups` or container resource limits
   - Add monitoring alerts for high CPU usage
   - Fix the underlying bug in the application

---

## S6: You need to replace a configuration value across hundreds of files in a directory. How?

**Answer:**

```bash
# Method 1: find + sed (most common)
find /etc/myapp/ -name "*.conf" -exec sed -i 's/old_value/new_value/g' {} \;

# Method 2: Faster with xargs
find /etc/myapp/ -name "*.conf" | xargs sed -i 's/old_value/new_value/g'

# Method 3: Preview changes first (dry run — don't modify files)
find /etc/myapp/ -name "*.conf" -exec grep -l "old_value" {} \;
# This shows which files would be affected

# Method 4: With backup (creates .bak files)
find /etc/myapp/ -name "*.conf" -exec sed -i.bak 's/old_value/new_value/g' {} \;

# Method 5: Replace with special characters (use different delimiter)
# If the value contains '/', use '|' as delimiter
sed -i 's|/old/path|/new/path|g' file.conf
```

**Safety tips:**
- Always preview which files will be affected before running
- Create backups with `sed -i.bak`
- Test on one file first
- Use version control (Git) so you can revert if something goes wrong

---

## S7: You need to write a script that monitors a log file in real time and sends an alert when a specific pattern appears.

**Answer:**

```bash
#!/bin/bash
# log-monitor.sh — Watch a log file for specific patterns

LOG_FILE="/var/log/application.log"
ALERT_PATTERNS=("OutOfMemoryError" "Connection refused" "FATAL" "disk full")
WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

send_alert() {
    local message="$1"
    # Send to Slack
    curl -s -X POST -H 'Content-type: application/json' \
        --data "{\"text\": \"🚨 ALERT: $message\"}" \
        "$WEBHOOK_URL"
    
    echo "$(date): Alert sent — $message" >> /var/log/log-monitor.log
}

echo "Monitoring $LOG_FILE for critical patterns..."

# tail -F follows the file even if it's rotated
tail -F "$LOG_FILE" | while read -r line; do
    for pattern in "${ALERT_PATTERNS[@]}"; do
        if echo "$line" | grep -qi "$pattern"; then
            send_alert "Pattern '$pattern' found: $line"
            break  # Don't send multiple alerts for the same line
        fi
    done
done
```

**Run it as a background service:**
```bash
# Create a systemd service
cat > /etc/systemd/system/log-monitor.service << EOF
[Unit]
Description=Log Monitor Service
After=network.target

[Service]
ExecStart=/scripts/log-monitor.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
EOF

systemctl enable log-monitor
systemctl start log-monitor
```

---

## S8: A junior developer accidentally deleted important files. How do you try to recover them?

**Answer:**

**Immediate steps:**
1. **Stop writing to the disk** — The data might still be on disk but marked as "free space." New writes could overwrite it.

2. **Check if the file is still open by a process:**
   ```bash
   # If a process still has the file open, you can recover it
   lsof | grep "deleted"
   
   # If found (e.g., PID 5678, file descriptor 3):
   cp /proc/5678/fd/3 /recovered/myfile.txt
   ```

3. **Check the trash/recycle bin:**
   ```bash
   ls ~/.local/share/Trash/files/    # Desktop environments
   ```

4. **Check version control:**
   ```bash
   git log --all --full-history -- path/to/deleted/file
   git checkout HEAD~1 -- path/to/deleted/file
   ```

5. **Check backups:**
   ```bash
   # Check if you have recent backups
   ls /backups/
   # Restore from backup
   tar -xzvf /backups/latest.tar.gz path/to/file
   ```

6. **Use recovery tools (last resort):**
   ```bash
   # For ext4 filesystems
   sudo extundelete /dev/sda1 --restore-file path/to/file
   
   # For general recovery
   sudo testdisk /dev/sda
   ```

**Prevention:**
- Use version control (Git) for all important files
- Set up automated backups
- Use `rm -i` (interactive mode) or `trash-cli` instead of `rm`
- Implement proper access controls

---

## S9: You need to set up passwordless SSH access from a CI/CD server to 50 deployment servers. How?

**Answer:**

```bash
#!/bin/bash
# setup-ssh-keys.sh — Deploy SSH keys to multiple servers

# Step 1: Generate a key pair on the CI/CD server (if not already done)
if [ ! -f ~/.ssh/id_ed25519 ]; then
    ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -N ""
    echo "Key pair generated."
fi

# Step 2: Read server list and deploy the public key
SERVERS_FILE="servers.txt"  # One server per line: user@hostname

while IFS= read -r server; do
    echo "Deploying key to $server..."
    
    # Using ssh-copy-id (requires password once)
    ssh-copy-id -i ~/.ssh/id_ed25519.pub "$server" 2>/dev/null
    
    if [ $? -eq 0 ]; then
        echo "  ✅ Success: $server"
    else
        echo "  ❌ Failed: $server"
    fi
done < "$SERVERS_FILE"

# Step 3: Test connections
echo ""
echo "Testing connections..."
while IFS= read -r server; do
    ssh -o BatchMode=yes -o ConnectTimeout=5 "$server" "echo 'OK'" 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "  ✅ $server — Connected"
    else
        echo "  ❌ $server — Failed"
    fi
done < "$SERVERS_FILE"
```

**For large-scale deployment, use Ansible instead:**
```yaml
# deploy-ssh-keys.yml
- hosts: all
  tasks:
    - name: Deploy CI/CD server's public key
      authorized_key:
        user: deploy
        key: "{{ lookup('file', '/var/lib/jenkins/.ssh/id_ed25519.pub') }}"
        state: present
```

**Security best practices:**
- Use a dedicated service account (not root)
- Limit the key's permissions in `authorized_keys` if possible
- Rotate keys periodically
- Use SSH certificates for even better security at scale

---

## S10: Write a script that creates a daily backup of a PostgreSQL database, compresses it, and keeps only the last 7 days of backups.

**Answer:**

```bash
#!/bin/bash
# pg-backup.sh — Daily PostgreSQL backup with rotation

# Configuration
DB_NAME="myapp_production"
DB_USER="backup_user"
DB_HOST="localhost"
BACKUP_DIR="/backups/postgres"
RETENTION_DAYS=7
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${DATE}.sql.gz"
LOG_FILE="/var/log/pg-backup.log"

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') — $1" >> "$LOG_FILE"
}

log "Starting backup of $DB_NAME"

# Create the backup (compressed)
pg_dump -h "$DB_HOST" -U "$DB_USER" "$DB_NAME" | gzip > "$BACKUP_FILE"

# Check if backup was successful
if [ $? -eq 0 ] && [ -s "$BACKUP_FILE" ]; then
    SIZE=$(du -h "$BACKUP_FILE" | awk '{print $1}')
    log "Backup successful: $BACKUP_FILE ($SIZE)"
else
    log "ERROR: Backup failed!"
    rm -f "$BACKUP_FILE"  # Remove empty/corrupt file
    
    # Send alert
    echo "PostgreSQL backup failed for $DB_NAME at $(date)" | \
        mail -s "BACKUP FAILURE: $DB_NAME" ops@example.com
    exit 1
fi

# Remove backups older than retention period
DELETED=$(find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +${RETENTION_DAYS} -delete -print | wc -l)
log "Cleaned up $DELETED old backup(s)"

# Verify we still have recent backups
BACKUP_COUNT=$(find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" | wc -l)
log "Total backups remaining: $BACKUP_COUNT"

log "Backup process complete"
```

**Cron schedule:**
```bash
# Run daily at 2 AM
0 2 * * * /scripts/pg-backup.sh
```

**Restore from backup:**
```bash
gunzip -c /backups/postgres/myapp_production_20260309.sql.gz | psql -h localhost -U admin myapp_production
```