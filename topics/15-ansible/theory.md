# Ansible — Theory Questions

---

## Q1: What is Ansible?

**Answer:**
**Ansible** is an open-source automation tool for configuration management, application deployment, and task automation. It's agentless — it connects to machines via SSH and doesn't require any software installed on target machines.

**Key features:**
- **Agentless** — No agent needed on managed machines (uses SSH)
- **YAML-based** — Easy to read and write playbooks
- **Idempotent** — Running the same playbook twice produces the same result
- **Large module library** — 3,000+ built-in modules
- **Push-based** — You initiate the configuration from a control node

---

## Q2: What is an Ansible Playbook?

**Answer:**
A **playbook** is a YAML file that defines a set of tasks to execute on target machines.

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes              # Run as root (sudo)
  vars:
    http_port: 80
    app_version: "2.1.0"

  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy Nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart Nginx

    - name: Ensure Nginx is running
      service:
        name: nginx
        state: started
        enabled: yes

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

**Key components:**
- **hosts** — Which machines to target
- **become** — Run with elevated privileges
- **vars** — Variables for the playbook
- **tasks** — List of actions to perform
- **handlers** — Tasks that run only when notified (triggered by changes)

---

## Q3: What is an Ansible Inventory?

**Answer:**
An **inventory** defines the target machines Ansible manages.

**Static inventory (INI format):**
```ini
[webservers]
web1.example.com
web2.example.com ansible_user=deploy

[dbservers]
db1.example.com ansible_port=2222
db2.example.com

[production:children]
webservers
dbservers

[production:vars]
ansible_user=admin
environment=production
```

**Dynamic inventory:** Automatically discovers hosts from cloud providers (AWS, Azure, GCP).
```bash
# AWS EC2 dynamic inventory
ansible-inventory -i aws_ec2.yml --list
```

---

## Q4: What are Ansible Roles?

**Answer:**
**Roles** are a way to organize playbooks into reusable, shareable components. They follow a standard directory structure.

```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml        # Main task list
    ├── handlers/
    │   └── main.yml        # Handlers
    ├── templates/
    │   └── nginx.conf.j2   # Jinja2 templates
    ├── files/
    │   └── index.html      # Static files
    ├── vars/
    │   └── main.yml        # Role variables
    ├── defaults/
    │   └── main.yml        # Default variables (lowest priority)
    └── meta/
        └── main.yml        # Role metadata and dependencies
```

**Using roles:**
```yaml
- hosts: webservers
  roles:
    - nginx
    - { role: app, app_version: "2.1.0" }
    - certbot
```

**Ansible Galaxy** — Public repository of community roles:
```bash
ansible-galaxy install geerlingguy.docker
ansible-galaxy install geerlingguy.nginx
```

---

## Q5: What is Ansible Vault?

**Answer:**
**Ansible Vault** encrypts sensitive data (passwords, API keys) so you can safely store them in version control.

```bash
# Create an encrypted file
ansible-vault create secrets.yml

# Encrypt an existing file
ansible-vault encrypt secrets.yml

# Edit an encrypted file
ansible-vault edit secrets.yml

# Decrypt a file
ansible-vault decrypt secrets.yml

# Run a playbook with vault password
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

**Encrypting individual variables:**
```yaml
# In your vars file
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  6238653136...encrypted_data...
```

---

## Q6: What are Ansible modules? Name some common ones.

**Answer:**
**Modules** are units of work that Ansible executes. Each module handles a specific task.

| Module | Purpose | Example |
|--------|---------|---------|
| `apt` / `yum` | Package management | Install/remove software |
| `service` / `systemd` | Service management | Start/stop/restart services |
| `copy` | Copy files to remote | Deploy config files |
| `template` | Copy with variable substitution | Dynamic config files |
| `file` | Manage files/directories | Create dirs, set permissions |
| `user` | Manage users | Create/delete users |
| `git` | Git operations | Clone repositories |
| `docker_container` | Docker management | Run containers |
| `command` / `shell` | Run commands | Execute arbitrary commands |
| `lineinfile` | Edit lines in files | Add/modify config lines |
| `cron` | Manage cron jobs | Schedule tasks |
| `uri` | HTTP requests | API calls, health checks |

---

## Q7: What is the difference between `command`, `shell`, and `raw` modules?

**Answer:**

| Module | Description | Use When |
|--------|-------------|----------|
| `command` | Executes a command directly (no shell features) | Simple commands, most secure |
| `shell` | Executes through a shell (supports pipes, redirects) | Need shell features (`|`, `>`, `&&`) |
| `raw` | Sends raw commands over SSH (no Python needed) | Bootstrapping, installing Python |

```yaml
# command — no shell features
- command: ls /tmp

# shell — supports pipes and redirects
- shell: cat /var/log/syslog | grep error > /tmp/errors.txt

# raw — for machines without Python
- raw: apt-get install -y python3
```

**Best practice:** Use specific modules (like `apt`, `copy`, `service`) instead of `command`/`shell` whenever possible — they're idempotent.

---

## Q8: What are Ansible facts?

**Answer:**
**Facts** are system information automatically gathered from managed machines. Ansible collects them at the start of each playbook run.

```yaml
# Use facts in playbooks
- name: Show OS info
  debug:
    msg: "OS: {{ ansible_distribution }} {{ ansible_distribution_version }}"

- name: Install package based on OS
  apt:
    name: nginx
  when: ansible_os_family == "Debian"

- name: Install package based on OS
  yum:
    name: nginx
  when: ansible_os_family == "RedHat"
```

**Common facts:**
- `ansible_hostname` — Machine hostname
- `ansible_os_family` — OS family (Debian, RedHat)
- `ansible_distribution` — Specific distro (Ubuntu, CentOS)
- `ansible_memtotal_mb` — Total RAM
- `ansible_processor_vcpus` — Number of CPUs
- `ansible_default_ipv4.address` — IP address

```bash
# View all facts for a host
ansible hostname -m setup
```

---

## Q9: What is idempotency in Ansible?

**Answer:**
**Idempotency** means running the same playbook multiple times produces the same result. The first run makes changes; subsequent runs detect no changes are needed and do nothing.

**Idempotent (good):**
```yaml
- name: Ensure Nginx is installed
  apt:
    name: nginx
    state: present    # If already installed, does nothing
```

**Not idempotent (avoid):**
```yaml
- name: Add line to config
  shell: echo "new_setting=true" >> /etc/myapp.conf
  # This appends the line EVERY time it runs!
```

**Fix:**
```yaml
- name: Add line to config
  lineinfile:
    path: /etc/myapp.conf
    line: "new_setting=true"
    state: present    # Only adds if not already there
```

---

## Q10: What are Ansible handlers?

**Answer:**
**Handlers** are tasks that only run when notified by another task. They're typically used to restart services after configuration changes.

```yaml
tasks:
  - name: Update Nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Restart Nginx          # Only notifies if the file changed

  - name: Update SSL certificate
    copy:
      src: cert.pem
      dest: /etc/nginx/cert.pem
    notify: Restart Nginx          # Same handler, only runs once

handlers:
  - name: Restart Nginx
    service:
      name: nginx
      state: restarted
```

**Key behavior:**
- Handlers run only once, even if notified multiple times
- They run at the end of the play (after all tasks)
- They run in the order they're defined (not the order they're notified)