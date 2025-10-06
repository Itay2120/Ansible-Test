# RHCE Practice Exam — Task List

## Exam Environment Overview

### Scenario Description

In this RHCE practice environment, you are provided with **five virtual machines**:

- **control** — The Ansible control node  
- **node1**, **node2**, **node3**, **node4** — Managed hosts

A user account named **george** is pre-configured on all systems.  
This user has **sudo privileges** without requiring a password.

SSH key-based authentication has already been configured between all nodes.  
You can verify host mappings in `/etc/hosts`.

---

### Important Notes

- All tasks in this exam must be performed **from the control node**.
- Use **Ansible** to manage the other four nodes.
- Avoid using `root` directly unless explicitly required.
- All playbooks should be created in `/home/george/ansible/` unless otherwise specified.

---

### Environment Validation

You can verify SSH access and host configuration with the following commands:

```bash
# Verify host entries
cat /etc/hosts

# Check connectivity to all nodes
ansible all -m ping -u george

# Verify sudo privileges
ansible all -m command -a "whoami" -u george -b
```

---

## Task 1: Install and Configure Ansible

### Requirements

1. Install Ansible on the **control** node.
2. Create an inventory file at `/home/george/ansible/inventory` with the following structure:
   - A group named **dev** containing **node1**
   - A group named **test** containing **node2** and **node3**
   - A group named **prod** containing **node4**
   - A group named **public** that includes both **dev** and **prod** as children
3. Create an Ansible configuration file at `/home/george/ansible/ansible.cfg` with the following settings:
   - Inventory path pointing to your inventory file
   - Collections path: `/home/george/.ansible/collections:/usr/share/ansible/collections`
   - Roles path: `/home/george/ansible/roles:/usr/share/ansible/roles`

### Solution

```bash
# Install Ansible
sudo dnf install -y ansible-core

# Create working directory
mkdir -p /home/george/ansible
cd /home/george/ansible

# Create inventory file
cat > inventory << 'EOF'
[dev]
node1

[test]
node2
node3

[prod]
node4

[public:children]
dev
prod
EOF

# Generate base configuration
ansible-config init --disabled > ansible.cfg

# Edit ansible.cfg to include only required settings
vim ansible.cfg
```

**ansible.cfg** content:

```ini
[defaults]
inventory = /home/george/ansible/inventory
collections_paths = /home/george/.ansible/collections:/usr/share/ansible/collections
roles_path = /home/george/ansible/roles:/usr/share/ansible/roles
remote_user = george

[privilege_escalation]
become = true
become_method = sudo
become_user = root
```

**Verify:**

```bash
ansible all -m ping
ansible dev --list-hosts
ansible public --list-hosts
```

---

## Task 2: Install Ansible Collections

### Requirements

You have been provided with a URL to download Ansible collection archives.

1. Create a directory at `/home/george/ansible/mycollection`
2. Download the following three collection files from `http://server.example.com/collections/`:
   - `collection1.tar.gz`
   - `collection2.tar.gz`
   - `collection3.tar.gz`
3. Install these collections using a requirements file
4. Collections should be installed to `/home/george/.ansible/collections`

### Solution

```bash
# Create collection directory
mkdir -p /home/george/ansible/mycollection
cd /home/george/ansible/mycollection

# Download collection files
wget http://server.example.com/collections/collection1.tar.gz
wget http://server.example.com/collections/collection2.tar.gz
wget http://server.example.com/collections/collection3.tar.gz

# Create requirements file
cat > requirements.yml << 'EOF'
---
collections:
  - name: /home/george/ansible/mycollection/collection1.tar.gz
  - name: /home/george/ansible/mycollection/collection2.tar.gz
  - name: /home/george/ansible/mycollection/collection3.tar.gz
EOF

# Install collections
ansible-galaxy collection install -r requirements.yml -p /home/george/.ansible/collections
```

**Verify:**

```bash
ansible-galaxy collection list
```

---

## Task 3: Configure Local Repositories

### Requirements

Create a playbook named `/home/george/ansible/repository.yml` that configures local YUM/DNF repositories on all managed hosts.

The playbook should:
1. Import the GPG key from `/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release`
2. Configure a repository named **BaseOS**:
   - Description: "RHEL 9 BaseOS Local Repo"
   - Base URL: `file:///reposerver/BaseOS`
   - GPG check: enabled
3. Configure a repository named **AppStream**:
   - Description: "RHEL 9 AppStream Local Repo"
   - Base URL: `file:///reposerver/AppStream`
   - GPG check: enabled

### Solution

```yaml
---
- name: Configure local repositories with GPG check
  hosts: all
  become: true
  tasks:
    - name: Import GPG key
      ansible.builtin.rpm_key:
        state: present
        key: /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

    - name: Configure BaseOS repository
      ansible.builtin.yum_repository:
        name: BaseOS
        description: RHEL 9 BaseOS Local Repo
        baseurl: file:///reposerver/BaseOS
        enabled: true
        gpgcheck: true
        gpgkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

    - name: Configure AppStream repository
      ansible.builtin.yum_repository:
        name: AppStream
        description: RHEL 9 AppStream Local Repo
        baseurl: file:///reposerver/AppStream
        enabled: true
        gpgcheck: true
        gpgkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

**Execute:**

```bash
ansible-playbook repository.yml
```

**Verify:**

```bash
ansible all -m shell -a "dnf repolist"
```

---

## Task 4: Install Software Packages

### Requirements

Create a playbook named `/home/george/ansible/packages.yml` that installs software packages as follows:

1. On **dev** and **test** hosts:
   - Install `httpd` package
   - Install `php` package

2. On **prod** hosts:
   - Install the **Development Tools** package group
   - Ensure all packages are updated to the latest version

### Solution

```yaml
---
- name: Install httpd and php on dev and test
  hosts: dev,test
  become: true
  tasks:
    - name: Install httpd and php packages
      ansible.builtin.dnf:
        name:
          - httpd
          - php
        state: present

- name: Install Development Tools and update prod
  hosts: prod
  become: true
  tasks:
    - name: Install Development Tools group
      ansible.builtin.dnf:
        name: "@Development Tools"
        state: present

    - name: Ensure all packages are up to date
      ansible.builtin.dnf:
        name: "*"
        state: latest
```

**Execute:**

```bash
ansible-playbook packages.yml
```

**Verify:**

```bash
ansible dev,test -m shell -a "rpm -q httpd php"
ansible prod -m shell -a "dnf group list --installed"
```

---

## Task 5: Create Group-Specific Files

### Requirements

Create a playbook named `/home/george/ansible/welcome.yml` that creates a file at `/etc/welcome.txt` on each managed host.

The file content should be different based on the host's group membership:
- Hosts in **dev** group: `"welcome to dev servers"`
- Hosts in **test** group: `"welcome to test servers"`
- Hosts in **prod** group: `"welcome to prod servers"`

### Solution

```yaml
---
- name: Create group-specific welcome files
  hosts: all
  become: true
  tasks:
    - name: Create welcome file for dev servers
      ansible.builtin.copy:
        dest: /etc/welcome.txt
        content: "welcome to dev servers\n"
        mode: '0644'
      when: "'dev' in group_names"

    - name: Create welcome file for test servers
      ansible.builtin.copy:
        dest: /etc/welcome.txt
        content: "welcome to test servers\n"
        mode: '0644'
      when: "'test' in group_names"

    - name: Create welcome file for prod servers
      ansible.builtin.copy:
        dest: /etc/welcome.txt
        content: "welcome to prod servers\n"
        mode: '0644'
      when: "'prod' in group_names"
```

**Execute:**

```bash
ansible-playbook welcome.yml
```

**Verify:**

```bash
ansible all -m shell -a "cat /etc/welcome.txt"
```

---

## Task 6: Create and Use an Ansible Role

### Requirements

1. Create a role named **webserver** in `/home/george/ansible/roles/`
2. The role should:
   - Install the `httpd` package
   - Start and enable the `httpd` service
3. Create a playbook named `/home/george/ansible/webserver.yml` that applies this role to the **dev** group

### Solution

```bash
# Create role structure
cd /home/george/ansible/roles
ansible-galaxy init webserver
```

**Edit `/home/george/ansible/roles/webserver/tasks/main.yml`:**

```yaml
---
# tasks file for webserver
- name: Install httpd package
  ansible.builtin.dnf:
    name: httpd
    state: present

- name: Start and enable httpd service
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true

- name: Allow HTTP through firewall
  ansible.posix.firewalld:
    service: http
    permanent: true
    state: enabled
    immediate: true
```

**Create `/home/george/ansible/webserver.yml`:**

```yaml
---
- name: Deploy webserver role
  hosts: dev
  become: true
  roles:
    - webserver
```

**Execute:**

```bash
ansible-playbook webserver.yml
```

**Verify:**

```bash
ansible dev -m shell -a "systemctl status httpd"
```

---

## Task 7: Deploy Web Content Using Templates

### Requirements

1. Create a Jinja2 template file at `/home/george/ansible/templates/index.j2`
2. The template should contain:
   - A welcome message with the hostname
   - The deployment date
3. Create a playbook named `/home/george/ansible/webcontent.yml` that:
   - Deploys the template to `/var/www/html/index.html` on all managed hosts

### Solution

**Create template directory and file:**

```bash
mkdir -p /home/george/ansible/templates
```

**Create `/home/george/ansible/templates/index.j2`:**

```jinja2
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome to {{ ansible_hostname }}</h1>
    <p>This server was deployed on {{ ansible_date_time.date }}</p>
    <p>Current time: {{ ansible_date_time.time }}</p>
</body>
</html>
```

**Create `/home/george/ansible/webcontent.yml`:**

```yaml
---
- name: Deploy web content from template
  hosts: all
  become: true
  tasks:
    - name: Deploy index.html from template
      ansible.builtin.template:
        src: templates/index.j2
        dest: /var/www/html/index.html
        owner: apache
        group: apache
        mode: '0644'
```

**Execute:**

```bash
ansible-playbook webcontent.yml
```

**Verify:**

```bash
ansible all -m shell -a "cat /var/www/html/index.html"
```

---

## Task 8: User and Group Management

### Requirements

Create a playbook named `/home/george/ansible/users.yml` that:

1. Creates a group named **admin** on all managed hosts
2. Creates a user named **deploy** with the following properties:
   - Primary group: **admin**
   - Password: `redhat` (properly hashed)
   - Shell: `/bin/bash`

### Solution

**Create `/home/george/ansible/users.yml`:**

```yaml
---
- name: Create admin group and deploy user
  hosts: all
  become: true
  tasks:
    - name: Create admin group
      ansible.builtin.group:
        name: admin
        state: present
        gid: 2000

    - name: Create deploy user
      ansible.builtin.user:
        name: deploy
        group: admin
        password: "{{ 'redhat' | password_hash('sha512') }}"
        shell: /bin/bash
        home: /home/deploy
        createhome: true
        state: present
```

**Execute:**

```bash
ansible-playbook users.yml
```

**Verify:**

```bash
ansible all -m shell -a "id deploy"
ansible all -m shell -a "getent group admin"
```

---

## Additional Tips

### Testing Playbooks

Always use the `--syntax-check` and `--check` flags before running playbooks:

```bash
ansible-playbook playbook.yml --syntax-check
ansible-playbook playbook.yml --check
ansible-playbook playbook.yml
```

### Common Modules Reference

- **ansible.builtin.copy** — Copy files to remote hosts
- **ansible.builtin.template** — Deploy Jinja2 templates
- **ansible.builtin.dnf/yum** — Manage packages
- **ansible.builtin.service** — Manage services
- **ansible.builtin.user** — Manage users
- **ansible.builtin.group** — Manage groups
- **ansible.builtin.file** — Manage files and directories
- **ansible.builtin.lineinfile** — Modify specific lines in files

### Good Luck! 🎯