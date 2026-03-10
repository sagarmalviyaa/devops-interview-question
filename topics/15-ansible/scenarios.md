# Ansible — Scenario-Based Questions

---

## S1: You need to deploy an application to 500 servers with zero downtime. How do you use Ansible?

**Answer:**

```yaml
---
- name: Deploy application with zero downtime
  hosts: appservers
  become: yes
  serial: 50                    # Deploy to 50 servers at a time
  max_fail_percentage: 5        # Stop if more than 5% fail
  
  pre_tasks:
    - name: Remove from load balancer
      uri:
        url: "http://lb.example.com/api/deregister/{{ inventory_hostname }}"
        method: POST
    
    - name: Wait for connections to drain
      pause:
        seconds: 30

  tasks:
    - name: Stop application
      service:
        name: myapp
        state: stopped

    - name: Deploy new version
      copy:
        src: "releases/myapp-{{ app_version }}.tar.gz"
        dest: /opt/myapp/
      
    - name: Extract application
      unarchive:
        src: "/opt/myapp/myapp-{{ app_version }}.tar.gz"
        dest: /opt/myapp/
        remote_src: yes

    - name: Start application
      service:
        name: myapp
        state: started

    - name: Wait for health check
      uri:
        url: "http://localhost:8080/health"
        status_code: 200
      register: health
      until: health.status == 200
      retries: 10
      delay: 5

  post_tasks:
    - name: Register back to load balancer
      uri:
        url: "http://lb.example.com/api/register/{{ inventory_hostname }}"
        method: POST
```

**Key strategies:**
- `serial: 50` — Rolling deployment, 50 at a time
- Remove from LB → Deploy → Health check → Add back to LB
- `max_fail_percentage` — Stop if too many failures

---

## S2: You need to configure different settings for different environments using the same playbook. How?

**Answer:**

**Use group variables:**

```
inventory/
├── production/
│   ├── hosts
│   └── group_vars/
│       └── all.yml
├── staging/
│   ├── hosts
│   └── group_vars/
│       └── all.yml
```

```yaml
# inventory/production/group_vars/all.yml
environment: production
db_host: prod-db.example.com
app_replicas: 5
debug_mode: false
```

```yaml
# inventory/staging/group_vars/all.yml
environment: staging
db_host: staging-db.example.com
app_replicas: 2
debug_mode: true
```

```yaml
# playbook.yml (same for all environments)
- hosts: all
  tasks:
    - name: Deploy config
      template:
        src: app.conf.j2
        dest: /etc/myapp/app.conf
```

```bash
# Run for specific environment
ansible-playbook -i inventory/staging playbook.yml
ansible-playbook -i inventory/production playbook.yml
```

---

## S3: An Ansible playbook is taking too long to run across 200 servers. How do you speed it up?

**Answer:**

1. **Increase parallelism:**
   ```ini
   # ansible.cfg
   [defaults]
   forks = 50    # Default is 5, increase to 50
   ```

2. **Disable fact gathering** if not needed:
   ```yaml
   - hosts: all
     gather_facts: no    # Saves 2-5 seconds per host
   ```

3. **Use `async` for long-running tasks:**
   ```yaml
   - name: Run long task
     command: /scripts/long-task.sh
     async: 300      # Max 300 seconds
     poll: 0         # Don't wait (fire and forget)
   ```

4. **Use `free` strategy** (don't wait for all hosts to finish each task):
   ```yaml
   - hosts: all
     strategy: free
   ```

5. **Enable pipelining:**
   ```ini
   [ssh_connection]
   pipelining = True    # Reduces SSH operations
   ```

6. **Use `mitogen`** strategy plugin (significantly faster SSH):
   ```ini
   [defaults]
   strategy_plugins = /path/to/mitogen/ansible_mitogen/plugins/strategy
   strategy = mitogen_linear
   ```

7. **Cache facts** to avoid re-gathering:
   ```ini
   [defaults]
   fact_caching = jsonfile
   fact_caching_connection = /tmp/ansible_facts
   fact_caching_timeout = 3600
   ```