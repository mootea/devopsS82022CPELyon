# Devops S8 2022 CPE Lyon
*Ayant été absent pour cause de Covid-19 (retour le 03/02/2022), je n'ai pas pu avoir accès au serveur pour Ansible donc je n'ai pas pu avancer ce TP. J'ai seulement pu me renseigner pour connaitre au minimum le fonctionnement*

# TP 3

# Intro

## Inventories

setup.yml
```
all: 
  vars:
    ansible_user: auremoote
    ansible_ssh_private_key_file: /Users/auremoo/Git/0. CPE/devopsS82022CPELyon/ansible/private_key
    children:
    prod:
    hosts: 127.0.0.1
```
Ping
```
ansible all -i inventories/setup.yml -m ping
```
->
```
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
```
Donc
```
ansible localhost -i inventories/setup.yml -m ping
```
->
```
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

## Facts

```
ansible localhost -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
```
->
```
localhost | SUCCESS => {
    "ansible_facts": {
        "ansible_distribution": "MacOSX",
        "ansible_distribution_major_version": "12",
        "ansible_distribution_release": "21.3.0",
        "ansible_distribution_version": "12.2"
    },
    "changed": false
}
```
Puis
```
ansible localhost -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become
````
-> *je ne sais pas comment expliquer cette erreur*
```
localhost | FAILED! => {
    "changed": false,
    "msg": [
        "Could not detect which major revision of yum is in use, which is required to determine module backend.",
        "You should manually specify use_backend to tell the module whether to use the yum (yum3) or dnf (yum4) backend})"
    ]
}
```

# Playbooks

## First playbook

playbook.yml
```
- hosts: all
  gather_facts: false
  become: yes
  tasks:
    - name: Test connection
      ping:
```

## Advanced playbook

