# Introduction
In this page I will write the questions that I saw in the RHCE exam.
You will open the test and see 5 VMs — 1 control node and 4 managed nodes controlled with Ansible.
In the test you will use a user that is already created for you — in my test it was george, who already has sudo privileges.
The SSH connection is already configured before the test, but you can verify it in /etc/hosts.

# Question 1
Install Ansible on the control node.
Create an inventory with 3 main groups: dev, test, and prod, and a child group named public.
Create ansible.cfg including only inventory, collections_paths, and roles_path.

# Answer:
sudo dnf install -y ansible
cd /home/george/something
vim inventory

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

ansible-config init --disabled > ansible.cfg

Then edit and include only:
inventory = ./inventory
collections_paths = /home/george/.ansible/collections:/usr/share/ansible/collections
roles_path = /home/george/plays/roles:/usr/share/ansible/roles


# Question 2
Install three files into a directory named mycollection and add them to the Ansible collection.

# Answer:
mkdir -p mycollection
cd mycollection
wget http://this/is/url/file1.yml
wget http://this/is/url/file2.yml
wget http://this/is/url/file3.yml

To install from a requirements file:
ansible-galaxy collection install -r requirements.yml -p /home/george/mycollection


# Question 3
Create a playbook called something.yml that adds two repositories (BaseOS and AppStream) to all hosts

# Answer:
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
        description: RHEL 9 BaseOS Local Repo
        baseurl: file:///reposerver/BaseOS
        enabled: true
        gpgcheck: true
        gpgkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release

    - name: Configure AppStream repository
      yum_repository:
        name: AppStream
        description: RHEL 9 AppStream Local Repo
        baseurl: file:///reposerver/AppStream
        enabled: true
        gpgcheck: true
        gpgkey: file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release


# Question 4
Create a playbook that installs packages as follows:

- Install httpd and php on dev and test groups.

- Install "Development Tools" on prod.

- Ensure all packages on prod are up to date

# Answer:
---
- name: Install httpd and php on dev and test
  hosts: dev,test
  become: true
  tasks:
    - name: Install packages
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

    - name: Ensure all packages are up to date
      dnf:
        name: "*"
        state: latest


# Question 5
Create a playbook that creates a file on each group with different content:

- dev → “welcome to dev servers”

- test → “welcome to test servers”

- prod → “welcome to prod servers”

# Answer:
---
- name: Create group-specific welcome files
  hosts: all
  become: true
  tasks:
    - name: Create dev file
      copy:
        dest: /etc/welcome.txt
        content: "welcome to dev servers"
      when: "'dev' in group_names"

    - name: Create test file
      copy:
        dest: /etc/welcome.txt
        content: "welcome to test servers"
      when: "'test' in group_names"

    - name: Create prod file
      copy:
        dest: /etc/welcome.txt
        content: "welcome to prod servers"
      when: "'prod' in group_names"


# Question 6
Create a role named webserver that installs and enables the httpd service.

# Answer:
ansible-galaxy init webserver

Edit roles/webserver/tasks/main.yml:

---
- name: Install httpd
  dnf:
    name: httpd
    state: present

- name: Enable and start httpd
  service:
    name: httpd
    state: started
    enabled: true

Use the role in a playbook:
---
- hosts: dev
  become: true
  roles:
    - webserver


# Question 7
Create a Jinja2 template file /templates/index.j2 with hostname and date, and deploy it to /var/www/html/index.html.

#Answer
templates/index.j2:
Welcome to {{ ansible_hostname }}
Deployed on {{ ansible_date_time.date }}

Playbook:
---
- hosts: all
  become: true
  tasks:
    - name: Deploy index.html from template
      template:
        src: templates/index.j2
        dest: /var/www/html/index.html

# Question 8
Create a group called admin and a user deploy who belongs to that group, with password redhat.

# Answer:
---
- hosts: all
  become: true
  tasks:
    - name: Create admin group
      group:
        name: admin
        state: present

    - name: Create deploy user
      user:
        name: deploy
        groups: admin
        password: "{{ 'redhat' | password_hash('sha512') }}"
 
