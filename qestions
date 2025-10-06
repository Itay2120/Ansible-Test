#  RHCE Practice – Real Exam Style Questions

## This page contains questions and answers from RHCE practice tests.  
You have 5 VMs: 1 control node and 4 managed nodes.  
SSH and `/etc/hosts` are pre-configured. User `george` has sudo privileges.

---

## Question 1 – Install and Configure Ansible

**Question:**  
- Install Ansible on the control node.  
- Create inventory with groups: `dev`, `test`, `prod` and a children group `public`.  
- Create `ansible.cfg` and include only: `inventory`, `collections_path`, `roles_path`.

**Answer:**

```bash
sudo dnf install -y ansible
cd /home/george/something
vim inventory


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


ansible-config init --disabled > ansible.cfg


ansible.cfg:
inventory = ./inventory
collections_paths = /home/george/.ansible/collections:/usr/share/ansible/collections
roles_path = /home/george/plays/roles:/usr/share/ansible/roles

