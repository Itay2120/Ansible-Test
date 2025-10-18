# RHCE Exam Practice Questions 

## Overview

This repository contains **17 real exam-style questions** from the **Red Hat Certified Engineer (RHCE)** exam, complete with detailed solutions and Ansible playbook examples.

These questions were documented after taking the actual RHCE exam and have been reorganized into a professional, easy-to-follow format to help others prepare for the certification.

---

##  What's Inside?

This repository includes:

- **17 Complete Exam Tasks** - Real questions from the RHCE exam
- **Detailed Solutions** - Step-by-step Ansible playbooks with explanations
- **Syntax-Highlighted Code** - Easy-to-copy YAML and Bash examples
- **Verification Commands** - How to validate each solution
- **Best Practices** - Tips and common pitfalls to avoid
- **Quick Reference Tables** - Essential Ansible modules and commands

---

## Exam Topics Covered

### Core Ansible Skills
- Installation and configuration (ansible.cfg, inventory)
- Collections and roles management
- Playbook creation and execution

### System Configuration
- YUM/DNF repository management
- Package installation and updates
- SELinux configuration
- User and group management

### Web Services
- Apache (httpd) deployment
- Firewall configuration
- Jinja2 templates
- Symbolic links for web content

### Advanced Topics
- LVM storage management (partitions, VG, LV)
- Ansible Vault (encryption, password management)
- System facts and conditional tasks
- Cron job automation

---

## How to Use This Repository

### For Exam Preparation:

1. **Read through each task** to understand the requirements
2. **Try solving it yourself** before looking at the solution
3. **Compare your solution** with the provided playbook
4. **Test in your lab environment** using the verification commands
5. **Take notes** on areas where you struggled

### For Quick Reference:

- Jump to specific topics using the task numbers
- Use the **Common Modules Reference Table** for quick syntax lookup
- Check the **Useful Commands** section at the bottom

---

## Exam Environment Structure

The RHCE exam provides:
- **1 Control Node** - Where you run Ansible commands
- **4 Managed Hosts** (node1, node2, node3, node4)
- **Pre-configured user** with sudo privileges (e.g., "george")
- **SSH key authentication** already set up between nodes

All tasks must be performed from the control node using Ansible.

---

## Task List

| # | Topic | Key Concepts |
|---|-------|--------------|
| 1 | Install and Configure Ansible | ansible.cfg, inventory, groups |
| 2 | Install Ansible Collections | ansible-galaxy, collections_paths |
| 3 | Configure YUM Repositories | yum_repository, rpm_key |
| 4 | Install Software Packages | dnf module, package groups |
| 5 | Create Group-Specific Files | conditional tasks, group_names |
| 6 | Configure SELinux | selinux module, lineinfile |
| 7 | Install Existing Roles | role structure, role dependencies |
| 8 | Create Custom Role | ansible-galaxy init, role tasks |
| 9 | Deploy Web Content | templates, firewall, services |
| 10 | Create Symbolic Links | file module, link state |
| 11 | Customize System Info File | ansible facts, get_url, lineinfile |
| 12 | Create LVM Storage | parted, lvg, lvol, filesystem, mount |
| 13 | Create Ansible Vault | ansible-vault create, encryption |
| 14 | Create Users from Encrypted File | vault, user module, loops, conditionals |
| 15 | Change Vault Password | ansible-vault rekey |
| 16 | Create Cron Jobs | cron module, scheduled tasks |

---

## Prerequisites

Before using these materials, ensure you have:

- **RHEL 9** or compatible system (Rocky Linux, AlmaLinux)
- **Ansible Core** installed (version 2.14+)
- Basic Linux system administration knowledge
- Basic YAML syntax understanding
- Access to a lab environment with multiple VMs

---

## Tips for Success

### Before the Exam:
- Practice all 16 tasks multiple times
- Get comfortable with `ansible-doc` command
- Memorize common module names and parameters
- Practice typing playbooks quickly and accurately
- Learn to use `--syntax-check` and `--check` flags

### During the Exam:
- Read each question carefully and note the requirements
- Always verify your inventory and ansible.cfg first
- Test connectivity with `ansible all -m ping`
- Use syntax-check before running playbooks
- Verify each task after execution
- Manage your time - don't spend too long on one task

### Common Pitfalls to Avoid:
- Forgetting `become: true` when root privileges needed
- Wrong indentation in YAML files
- Incorrect inventory group names
- Not setting services to start at boot (`enabled: true`)
- Forgetting to open firewall ports
- Wrong path for collections or roles in ansible.cfg

---

## Quick Reference

### Essential Ansible Commands

```bash
# Test connectivity
ansible all -m ping

# Run playbook with syntax check
ansible-playbook playbook.yml --syntax-check

# Dry run (check mode)
ansible-playbook playbook.yml --check

# Run with vault password
ansible-playbook playbook.yml --vault-password-file=secret

# List hosts in inventory
ansible all --list-hosts

# View ansible configuration
ansible-config dump

# Get module documentation
ansible-doc <module_name>
```

### Most Common Modules

```yaml
# Package management
dnf:
  name: httpd
  state: present

# Service management
service:
  name: httpd
  state: started
  enabled: true

# File operations
copy:
  dest: /path/to/file
  content: "text content"

# Template deployment
template:
  src: template.j2
  dest: /path/to/file

# User management
user:
  name: username
  state: present
  groups: groupname
```

---

## Contributing

If you find any errors or have suggestions for improvements:
1. Open an issue describing the problem
2. Submit a pull request with your proposed changes
3. Share additional exam questions you encountered

---

## Disclaimer

This repository is for **educational purposes only**. The questions are based on personal exam experience and may not reflect the exact current exam content. Red Hat regularly updates their exams, so always refer to the official Red Hat documentation and exam objectives.

---

## Additional Resources

- [Red Hat RHCE Official Page](https://www.redhat.com/en/services/certification/rhce)
- [Ansible Official Documentation](https://docs.ansible.com/)
- [Ansible Galaxy](https://galaxy.ansible.com/)
- [RHEL 9 Documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9)

---

## License

This project is provided as-is for educational purposes. Feel free to use, modify, and share.

---

## Good Luck!

Remember: **Practice makes perfect**. Work through these tasks multiple times until you can complete them confidently without referring to the solutions.

**You've got this! **

---

*Last Updated: October 2025*
