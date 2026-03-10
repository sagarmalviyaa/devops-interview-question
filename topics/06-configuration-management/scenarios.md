# Configuration Management — Scenario-Based Questions

---

## S1: You have 200 servers that need a security patch applied within 24 hours. How do you do it?

**Answer:**

**Using Ansible:**

1. **Prepare the inventory** (list of servers):
   ```ini
   # inventory.ini
   [webservers]
   web[01:50].example.com
   
   [appservers]
   app[01:100].example.com
   
   [dbservers]
   db[01:50].example.com
   ```

2. **Write the playbook:**
   ```yaml
   # security-patch.yml
   - name: Apply security patch
     hosts: all
     become: yes
     serial: 20        # Patch 20 servers at a time (rolling update)
     max_fail_percentage: 10  # Stop if more than 10% fail
     
     tasks:
       - name: Update the vulnerable package
         apt:
           name: openssl
           state: latest
           update_cache: yes
         when: ansible_os_family == "Debian"
       
       - name: Update the vulnerable package (RHEL)
         yum:
           name: openssl
           state: latest
         when: ansible_os_family == "RedHat"
       
       - name: Restart affected services
         service:
           name: nginx
           state: restarted
         when: "'webservers' in group_names"
       
       - name: Verify patch applied
         command: openssl version
         register: ssl_version
       
       - name: Report version
         debug:
           msg: "{{ inventory_hostname }}: {{ ssl_version.stdout }}"
   ```

3. **Run with rolling deployment:**
   ```bash
   # Dry run first
   ansible-playbook -i inventory.ini security-patch.yml --check
   
   # Apply for real
   ansible-playbook -i inventory.ini security-patch.yml
   ```

4. **Verify:**
   ```bash
   ansible all -i inventory.ini -m command -a "openssl version"
   ```

**Key decisions:**
- `serial: 20` — Don't patch all 200 at once; do 20 at a time
- `max_fail_percentage: 10` — Stop if too many failures (something is wrong)
- Test on a small group first, then roll out to all

---

## S2: A developer says "it works on my machine but not on the server." How does configuration management help?

**Answer:**

**The problem:** The developer's machine and the server have different configurations — different package versions, different settings, different environment variables.

**How configuration management solves this:**

1. **Define the environment in code:**
   ```yaml
   # server-setup.yml
   - name: Configure application server
     hosts: appservers
     tasks:
       - name: Install exact Node.js version
         apt:
           name: nodejs=20.11.0-1nodesource1
           state: present
       
       - name: Install exact npm packages
         npm:
           path: /var/www/app
           state: present
       
       - name: Set environment variables
         template:
           src: env.j2
           dest: /var/www/app/.env
       
       - name: Configure system limits
         sysctl:
           name: net.core.somaxconn
           value: '1024'
   ```

2. **Use the same playbook for all environments:**
   ```bash
   ansible-playbook server-setup.yml -e "env=development"
   ansible-playbook server-setup.yml -e "env=staging"
   ansible-playbook server-setup.yml -e "env=production"
   ```

3. **Even better — use containers:**
   - Package the application with its exact dependencies in a Docker image
   - The same image runs everywhere (dev, staging, production)
   - "Works on my machine" becomes "works everywhere"

---

## S3: You discover that someone manually changed the configuration on a production server, causing it to differ from the other servers. How do you handle this?

**Answer:**

1. **Detect the drift:**
   ```bash
   # Run Ansible in check mode to see differences
   ansible-playbook server-config.yml --check --diff
   # This shows what would change without actually changing anything
   ```

2. **Investigate why:**
   - Check server logs: `last` (who logged in), `history` (what commands were run)
   - Ask the team if anyone made manual changes
   - Document the finding

3. **Fix the drift:**
   ```bash
   # Re-apply the configuration to bring it back to desired state
   ansible-playbook server-config.yml --limit problem-server
   ```

4. **Prevent it from happening again:**
   - **Remove direct SSH access** for most team members
   - **Use read-only access** — only CI/CD can make changes
   - **Set up drift detection** on a schedule:
     ```bash
     # Cron job that runs Ansible in check mode and alerts on drift
     0 */6 * * * ansible-playbook server-config.yml --check 2>&1 | grep -i "changed" && send-alert
     ```
   - **Implement change management** — all changes go through code review and CI/CD
   - **Consider immutable infrastructure** — replace servers instead of modifying them