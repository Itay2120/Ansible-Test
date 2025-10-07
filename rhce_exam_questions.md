# RHCE Practice Exam — Complete Task List

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

- All tasks in this exam must be performed **from the control node**
- Use **Ansible** to manage the other four nodes
- The user **george** already has sudo privileges configured
- SSH connections are pre-configured between all nodes
- Work from `/home/george/ansible/` directory unless otherwise specified

---

## Task 1: Install and Configure Ansible

### Requirements

1. Install Ansible on the **control** node
2. Create an inventory file with the following structure:
   - A group named **dev** containing **node1**
   - A group named **test** containing **node2** and **node3**
   - A group named **prod** containing **node3** and **node4**
   - A group named **public** that includes both **dev** and **prod**
3. Create an `ansible.cfg` file with **only** these configurations:
   - `inventory path` = ./inventory
   - `collections_paths` = /home/george/mycollection
   - `roles_path` = /home/george/roles

### Solution

```bash
# Install Ansible
dnf install -y ansible

# Create working directory
mkdir -p /home/george/ansible
cd /home/george/ansible
vim inventory
```

**inventory file:**

```ini
[dev]
node1

[test]
node2
node3

[prod]
node3
node4

[public:children]
dev
prod
```

**Create ansible.cfg:**

```bash
# Generate base configuration with all options disabled
ansible-config init --disabled > ansible.cfg

# Edit to enable only required settings
vim ansible.cfg
```

**ansible.cfg (uncomment and set only these lines):**

```ini
[defaults]
inventory = ./inventory
collections_paths = /home/george/ansible/mycollection
roles_path = /home/george/plays/roles:/usr/share/ansible/roles
```

**Verify:**

```bash
ansible all -m ping
ansible all --list-hosts
```

---

## Task 2: Install Ansible Collections

### Requirements

1. Create a directory named `mycollection`
2. Download 3 collection files from a provided URL
3. Install these files to the Ansible collection path

### Solution

```bash
# Create collection directory
mkdir -p mycollection
cd mycollection

# Download collection files (replace with actual URLs from exam)
wget http://server.example.com/collections/file1.tar.gz
wget http://server.example.com/collections/file2.tar.gz
wget http://server.example.com/collections/file3.tar.gz

# Install collections using requirements file
ansible-galaxy collection install -r requirements.yml -p /home/george/ansible/mycollection
```

**Alternative: Install directly from files**

```bash
ansible-galaxy collection install file1.tar.gz -p /home/george/ansible/mycollection
ansible-galaxy collection install file2.tar.gz -p /home/george/ansible/mycollection
ansible-galaxy collection install file3.tar.gz -p /home/george/ansible/mycollection
```

**Verify:**

```bash
ansible-galaxy collection list
```

---

## Task 3: Configure YUM/DNF Repositories

### Requirements

Create a playbook that configures local repositories on **all hosts**:
- Repository 1: **BaseOS** at `file:///reposerver/BaseOS`
- Repository 2: **AppStream** at `file:///reposerver/AppStream`
- Both repositories should have GPG check enabled

### Solution

**Create `repository.yml`:**

```yaml
---
- name: Configure local repositories with GPG check
  hosts: all
  become: true

  tasks:
    - name: Import GPG key
      rpm_key:
        state: present
        key: /etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

    - name: Configure BaseOS repository
      yum_repository:
        name: BaseOS
        description: 'RHEL 9 BaseOS Local Repo'
        baseurl: file:///reposerver/BaseOS
        enabled: true
        gpgcheck: true
        gpgkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

    - name: Configure AppStream repository
      yum_repository:
        name: AppStream
        description: 'RHEL 9 AppStream Local Repo'
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

Create a playbook that installs packages with the following requirements:

1. On **dev** and **test** groups:
   - Install `httpd` package
   - Install `php` package

2. On **prod** group:
   - Install **Development Tools** package group
   - Ensure all packages are updated to the latest version

### Solution

**Create `packages.yml`:**

```yaml
---
- name: Install packages on dev and test
  hosts: dev,test
  become: true

  tasks:
    - name: Install httpd and php packages
      dnf:
        name:
          - httpd
          - php
        state: present

- name: Install Development Tools and update prod
  hosts: prod
  become: true

  tasks:
    - name: Install Development Tools group
      dnf:
        name: "@Development Tools"
        state: present

    - name: Make sure all packages are up to date
      dnf:
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
ansible prod -m shell -a "dnf group list --installed | grep Development"
```

---

## Task 5: Create Group-Specific Files

### Requirements

Create a playbook that creates a file on each group with different content:
- **dev** hosts: file contains `"welcome to dev servers"`
- **test** hosts: file contains `"welcome to test servers"`
- **prod** hosts: file contains `"welcome to prod servers"`

### Solution

**Create `welcome.yml`:**

```yaml
---
- name: Create text file for dev group
  hosts: dev
  become: true

  tasks:
    - name: Create welcome file for dev
      copy:
        dest: /etc/welcome.txt
        content: "welcome to dev servers"

- name: Create text file for test group
  hosts: test
  become: true

  tasks:
    - name: Create welcome file for test
      copy:
        dest: /etc/welcome.txt
        content: "welcome to test servers"

- name: Create text file for prod group
  hosts: prod
  become: true

  tasks:
    - name: Create welcome file for prod
      copy:
        dest: /etc/welcome.txt
        content: "welcome to prod servers"
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

## Task 6: Configure SELinux

### Requirements

Create a playbook that sets SELinux to **enforcing** mode on all managed hosts.

### Solution

**Option 1: Using selinux module (Recommended)**

```yaml
---
- name: Configure SELinux to enforcing
  hosts: all
  become: true

  tasks:
    - name: Set SELinux to enforcing mode
      selinux:
        policy: targeted
        state: enforcing
```

**Option 2: Using lineinfile module**

```yaml
---
- name: Configure SELinux to enforcing
  hosts: all
  become: true

  tasks:
    - name: Set SELinux to enforcing in config file
      lineinfile:
        path: /etc/selinux/config
        regexp: '^SELINUX='
        line: 'SELINUX=enforcing'

    - name: Set SELinux to enforcing immediately
      command: setenforce 1
      when: ansible_selinux.status == "enabled"
```

**Execute:**

```bash
ansible-playbook selinux.yml
```

**Verify:**

```bash
ansible all -m shell -a "getenforce"
ansible all -m shell -a "cat /etc/selinux/config | grep ^SELINUX="
```

---

## Task 7: Install and Use Existing Roles

### Requirements

1. Download 2 role files from a provided URL to the roles directory
2. Rename the first role to a specified name
3. Create a playbook that executes both roles

### Solution

```bash
# Create roles directory
mkdir -p roles
cd roles

# Download role files (URLs provided in exam)
wget http://server.example.com/roles/role1.tar.gz
wget http://server.example.com/roles/role2.tar.gz

# Extract and rename first role
tar -xzf role1.tar.gz
mv role1 role55

# Extract second role
tar -xzf role2.tar.gz
```

**Create `use_roles.yml`:**

```yaml
---
- name: Execute downloaded roles
  hosts: all
  become: true

  roles:
    - role55
    - role2
```

**Execute:**

```bash
ansible-playbook use_roles.yml
```

---

## Task 8: Create Custom Role

### Requirements

1. Create a role named **apache** that installs the `httpd` package
2. Create a playbook that applies this role to specified groups

### Solution

```bash
# Create role structure
ansible-galaxy init apache
```

**Edit `roles/apache/tasks/main.yml`:**

```yaml
---
# tasks file for apache
- name: Install httpd package
  dnf:
    name: httpd
    state: present

- name: Start and enable httpd service
  service:
    name: httpd
    state: started
    enabled: true
```

**Create `apache_deploy.yml`:**

```yaml
---
- name: Use the apache role
  hosts: all
  become: true

  roles:
    - apache
```

**Execute:**

```bash
ansible-playbook apache_deploy.yml
```

**Verify:**

```bash
ansible all -m shell -a "systemctl status httpd"
```

---

## Task 9: Deploy Web Content with Templates

### Requirements

Create a playbook that:
1. Installs `httpd` and `firewalld` packages
2. Ensures both services start after reboot
3. Opens port 80 in the firewall
4. Deploys a Jinja2 template with different content per group

### Solution

**Create `templates/index.html.j2`:**

```jinja2
<html>
  <body>
    <h1>{{ page_content }}</h1>
  </body>
</html>
```

**Create group variables:**

```bash
mkdir -p group_vars
echo "page_content: 'This is a WEB server'" > group_vars/dev.yml
echo "page_content: 'This is a DATABASE server'" > group_vars/prod.yml
```

**Create `webserver.yml`:**

```yaml
---
- name: Install httpd, firewall, and create template
  hosts: all
  become: true

  tasks:
    - name: Install httpd and firewalld
      dnf:
        name:
          - httpd
          - firewalld
        state: present

    - name: Open port 80 in firewall
      firewalld:
        port: 80/tcp
        permanent: true
        state: enabled
        immediate: true

    - name: Enable and start services
      service:
        name: "{{ item }}"
        state: started
        enabled: true
      loop:
        - httpd
        - firewalld

    - name: Deploy the web page template
      template:
        src: templates/index.html.j2
        dest: /var/www/html/index.html
        mode: '0644'
```

**Execute:**

```bash
ansible-playbook webserver.yml
```

**Verify:**

```bash
ansible all -m shell -a "curl http://localhost"
ansible all -m shell -a "firewall-cmd --list-ports"
```

---

## Task 10: Create Symbolic Link for Web Content

### Requirements

1. Create a directory /etc/mywebdir with an `index.html` file inside
2. Create a symbolic link from this directory to `/var/www/html/`
3. Verify the content is accessible via HTTP

### Solution

**Create `weblink.yml`:**

```yaml
---
- name: Create linked directory for web content
  hosts: all
  become: true

  tasks:
    - name: Create custom directory
      file:
        path: /etc/mywebdir
        state: directory
        mode: '0755'

    - name: Create index.html
      copy:
        dest: /etc/mywebdir/index.html
        content: Welcome to {{ ansible_hostname }}

    - name: Create symbolic link
      file:
        src: /etc/mywebdir
        dest: /var/www/html/mywebdir
        state: link
```

**Execute:**

```bash
ansible-playbook weblink.yml
```

**Verify:**

```bash
ansible all -m shell -a "curl http://localhost/mywebdir/"
ansible all -m shell -a "ls -l /var/www/html/mywebdir"
```

---

## Task 11: Customize System Information File

### Requirements

1. Download a file from a provided URL
2. Modify the file to include system-specific information:
   - Hostname
   - BIOS version
   - RAM in MB
   - vda disk size
   - vdb disk size
3.  NONE if somthing not present 

### Solution

**First, gather facts to find variable names:**

```bash
ansible localhost -m setup > facts.txt
vim facts.txt
# Search for: ansible_hostname, ansible_bios_version, ansible_memtotal_mb, ansible_devices
```

**Create `system_info.yml`:**

```yaml
---
- name: Create file and customize with system info
  hosts: all
  become: true

  tasks:
    - name: Download the template file
      get_url:
        url: http://server.example.com/system_info.txt
        dest: /tmp/system_info.txt
        mode: '0644'

    - name: Set hostname
      lineinfile:
        path: /tmp/system_info.txt
        regexp: '^HOST_NAME='
        line: 'HOST_NAME={{ ansible_hostname }}'

    - name: Set BIOS version
      lineinfile:
        path: /tmp/system_info.txt
        regexp: '^BIOS_VERSION='
        line: 'BIOS_VERSION={{ ansible_bios_version }}'

    - name: Set RAM size
      lineinfile:
        path: /tmp/system_info.txt
        regexp: '^MEMORY='
        line: 'MEMORY={{ ansible_memtotal_mb }}'

    - name: Set vda size
      lineinfile:
        path: /tmp/system_info.txt
        regexp: '^VDA_SIZE='
        line: 'VDA_SIZE={{ ansible_devices.vda.size | default("NONE") }}'

    - name: Set vdb size
      lineinfile:
        path: /tmp/system_info.txt
        regexp: '^VDB_SIZE='
        line: 'VDB_SIZE={{ ansible_devices.vdb.size | default("NONE") }}'
```

**Execute:**

```bash
ansible-playbook system_info.yml
```

**Verify:**

```bash
ansible all -m shell -a "cat /tmp/system_info.txt"
```

---

## Task 12: Create LVM Storage

### Requirements

Create a playbook that:
1. Creates a new partition on `/dev/vdb`
2. Creates a volume group named `vg_database`
3. Creates a logical volume named `lv_mysql` with 512MB size
4. Formats the LV with ext4 filesystem
5. Mounts it permanently

### Solution

**Create `storage.yml`:**

```yaml
---
- name: Configure LVM storage
  hosts: all
  become: true

  tasks:
    - name: Create new partition
      parted:
        device: /dev/vdb
        number: 1
        state: present
        part_start: 1MiB
        part_end: 800MiB

    - name: Create volume group
      lvg:
        vg: vg_database
        pvs: /dev/vdb1

    - name: Create logical volume
      lvol:
        vg: vg_database
        lv: lv_mysql
        size: 512

    - name: Create ext4 filesystem
      filesystem:
        fstype: ext4
        dev: /dev/vg_database/lv_mysql

    - name: Create mount point directory
      file:
        path: /mnt/mysql_data
        state: directory
        mode: '0755'

    - name: Mount the filesystem permanently
      mount:
        path: /mnt/mysql_data
        src: /dev/vg_database/lv_mysql
        fstype: ext4
        state: mounted
```

**Execute:**

```bash
ansible-playbook storage.yml
```

**Verify:**

```bash
ansible all -m shell -a "lsblk"
ansible all -m shell -a "df -h /mnt/mysql_data"
ansible all -m shell -a "cat /etc/fstab | grep mysql"
```

---

## Task 13: Create Ansible Vault

### Requirements

1. Create a file named `secret` containing the password `password`
2. Create an encrypted vault file called **vault.yml** using the password from `secret`
3. The vault should contain a variable `user_password` set to `devops`

### Solution

```bash
# Create password file
echo 'password' > secret

# Create encrypted vault file
ansible-vault create vault.yml --vault-password-file=secret
```

**Inside the vault editor, add:**

```yaml
---
user_password: devops
```

**Alternative: Create vault in one command**

```bash
echo -e "---\nuser_password: devops" | ansible-vault encrypt --vault-password-file=secret --output=vault.yml
```

**Verify:**

```bash
# View encrypted file
cat vault.yml

# Decrypt and view
ansible-vault view vault.yml --vault-password-file=secret
```

---

## Task 14: Create Users from Encrypted File

### Requirements

1. Download a file containing user information (name and job)
2. Encrypt this file with ansible-vault
3. Create a playbook that:
   - Uses the encrypted user list
   - Uses the password from the vault created earlier
   - Creates users based on their job attribute
   - Sets their password using the vault password

### Solution

```bash
# Download users file
wget http://server.example.com/users.yml

# Encrypt the users file
ansible-vault encrypt users.yml --vault-password-file=secret
```

**Example users.yml content (before encryption):**

```yaml
---
users:
  - name: user1
    job: developer
  - name: user2
    job: database
  - name: user3
    job: developer
```

**Create `create_users.yml`:**

```yaml
---
- name: Add users from encrypted file
  hosts: all
  become: true
  vars_files:
    - vault.yml
    - users.yml

  tasks:
    - name: Create developer users
      user:
        name: "{{ item.name }}"
        password: "{{ user_password | password_hash('sha512') }}"
        groups: developers
        state: present
      loop: "{{ users }}"
      when: item.job == "developer"

    - name: Create database users
      user:
        name: "{{ item.name }}"
        password: "{{ user_password | password_hash('sha512') }}"
        groups: database
        state: present
      loop: "{{ users }}"
      when: item.job == "database"
```

**Execute:**

```bash
ansible-playbook create_users.yml --vault-password-file=secret
```

**Verify:**

```bash
ansible all -m shell -a "getent passwd | grep user"
```

---

## Task 15: Change Vault Password

### Requirements

Change the password of an existing vault file from the old password to a new password.

### Solution

```bash
# Method 1: Interactive password change
ansible-vault rekey vault.yml

# Method 2: Using password file
ansible-vault rekey vault.yml --vault-password-file=secret --new-vault-password-file=new_secret

# Method 3: Ask for both passwords
ansible-vault rekey vault.yml --ask-vault-pass
```

**When prompted:**
1. Enter old password: `password`
2. Enter new password: `newpassword`
3. Confirm new password: `newpassword`

**Verify:**

```bash
# Try with old password (should fail)
ansible-vault view vault.yml --vault-password-file=secret

# Try with new password (should work)
ansible-vault view vault.yml --vault-password-file=new_secret
```

---

## Task 16: Create Cron Job

### Requirements

Create a cron job that:
- Runs as user **natasha**
- Executes every **2 minutes**
- Performs a specific task (e.g., append output to a file)

### Solution

**Create `cron.yml`:**

```yaml
---
- name: Create cron job
  hosts: all
  become: true

  tasks:
    - name: Ensure user natasha exists
      user:
        name: natasha
        state: present

    - name: Create cron job for natasha
      cron:
        name: "Check system status"
        user: natasha
        minute: "*/2"
        job: "date >> /tmp/cron_output.txt"
```

**Execute:**

```bash
ansible-playbook cron.yml
```

**Verify:**

```bash
# Check cron jobs for natasha
ansible all -m shell -a "crontab -u natasha -l"

# Wait 2 minutes and check output
ansible all -m shell -a "cat /tmp/cron_output.txt"
```

---

## Exam Tips and Best Practices

### Before Starting

```bash
# Always verify your environment first
ansible all -m ping
ansible all --list-hosts
cat ansible.cfg
cat inventory
```

### During the Exam

1. **Always use syntax check before running:**
   ```bash
   ansible-playbook playbook.yml --syntax-check
   ```

2. **Use check mode for dry runs:**
   ```bash
   ansible-playbook playbook.yml --check
   ```

3. **Verify after each task:**
   ```bash
   ansible all -m shell -a "command_to_verify"
   ```

### Common Ansible Modules

| Module | Purpose | Example |
|--------|---------|---------|
| `copy` | Copy files | `copy: dest=/path content="text"` |
| `template` | Deploy Jinja2 templates | `template: src=file.j2 dest=/path` |
| `dnf` | Manage packages | `dnf: name=httpd state=present` |
| `service` | Manage services | `service: name=httpd state=started` |
| `user` | Manage users | `user: name=john state=present` |
| `group` | Manage groups | `group: name=admins state=present` |
| `file` | Manage files/dirs | `file: path=/dir state=directory` |
| `lineinfile` | Modify file lines | `lineinfile: path=/file regexp='^X' line='X=value'` |
| `get_url` | Download files | `get_url: url=http://... dest=/path` |
| `parted` | Partition disks | `parted: device=/dev/vdb number=1` |
| `lvg` | Manage VGs | `lvg: vg=myvg pvs=/dev/vdb1` |
| `lvol` | Manage LVs | `lvol: vg=myvg lv=mylv size=512m` |

### Useful Commands

```bash
# Check Ansible version
ansible --version

# List all hosts in a group
ansible group_name --list-hosts

# Test connection
ansible all -m ping

# Gather facts about a host
ansible hostname -m setup

# Run ad-hoc command
ansible all -m shell -a "uptime"

# Check playbook syntax
ansible-playbook playbook.yml --syntax-check

# Dry run (check mode)
ansible-playbook playbook.yml --check

# Run with vault password
ansible-playbook playbook.yml --vault-password-file=secret

# List available roles
ansible-galaxy list

# View encrypted vault
ansible-vault view vault.yml
```

---

## Good Luck! 

Remember:
- Read each question carefully
- Test your playbooks before final execution
- Verify results after each task
- Use proper YAML formatting
- Always use `become: true` when needed
- Keep vault passwords secure
