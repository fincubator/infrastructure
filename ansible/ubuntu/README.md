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
python -V
python3 -V


ansible-playbook common.yml --extra-vars "target=all" -i inventory_dev


