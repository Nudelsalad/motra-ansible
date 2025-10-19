# Prerequisites
- Ansible installed on your control machine
- SSH access to target machines
- install galaxy roles from requirements.yml

```bash
ansible-galaxy install -r requirements.yml
```

run the playbook

```bash
ansible-playbook site.yml
```