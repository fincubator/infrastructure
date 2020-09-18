# Ansible playbooks for Ubuntu 18.04 Bionic
Deploy gateway and booker services.


## Install
### Linux (Ubuntu 18.04)

##### Requirements on management server
* Ansible

##### Requirements on service server
* Git
* Docker
* Docker Compose

Check that python3 installed:
```bash
python -V
Python 3.6.9
python3 -V
Python 3.6.9
```

Install common software on all hosts:
```bash
ansible-playbook common.yml --extra-vars "target=all" -i inventory_dev
```

Install postgresql on database servers:
```bash
ansible-playbook db.yml -i inventory_dev
```




