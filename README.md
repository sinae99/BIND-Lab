# BIND-Lab
 DNS management


deploying master and slave servers, creating and managing zones, adding DNS records and different DNS configurations




## Inventory list:


**contorl node** -----> *"win11"* - (192.168.1.100) - Ubuntu 24.04 (WSL2) - runs Ansible and manages remote nodes

**master dns server** -----> *"dns-master"* - (192.168.1.11) - Ubuntu Server - primary authoritative server

**slave dns server** -----> *"dns-slave"* - (192.168.1.22) - Ubuntu Server - secondary server for redundancy

<img width="672" height="260" alt="hosts" src="https://github.com/user-attachments/assets/02617400-3176-4212-9708-bd80798c0a8a" />





## Deploy:

`ansible.cfg` :

```
[defaults]
inventory = ./hosts.ini
remote_user = s
private_key_file = ~/.ssh/id_ed25519
host_key_checking = False
ansible_python_interpreter=/usr/bin/python3
```


first a connectivity check with machines after sharing the pub keys:


<img width="788" height="265" alt="ping" src="https://github.com/user-attachments/assets/5fcc5218-abd5-4f8b-84e8-ae963f871819" />








-------------------------------------------------------------------



## Tasks and Scenarios:


#### Create Zone
( Add a new zone definition and database file )

`install_bind_master.yaml` 


```
- name: Install BIND on master
  hosts: dns-master
  become: yes
  tasks:
    - name: update and install BIND
      apt:
        name: bind9
        state: present
        update_cache: yes

    - name: ensure BIND service is running
      systemd:
        name: bind9
        enabled: yes
        state: started
```

<img width="1645" height="470" alt="install_bind_master" src="https://github.com/user-attachments/assets/e6b551a5-7bd3-490a-9530-3796075702c3" />




#### Create DB
( Add a new database file )

i created files that i need in this step:


`db.sina`:


<img width="655" height="508" alt="db sina" src="https://github.com/user-attachments/assets/d38948f7-5e4b-4b4c-822b-1515073ed455" />


`named.conf.local`:


```
zone "sina" {
    type master;
    file "/etc/bind/db.sina";
    allow-transfer { 192.168.1.12; };
};
```


#### Add Record
( Insert a selected DNS record )


#### Modify Record
( Delete or update a record )


#### Configure Cache-Only Mode
( Set up a node as a caching resolver )














