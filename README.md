# BIND-Lab
 DNS management + automation


deploying master and slave servers, creating and managing zones, adding DNS records and different DNS configurations







## Inventory list:


**contorl node** -----> *"win11"* - (192.168.1.100) - Ubuntu 24.04 (WSL2) - runs Ansible and manages remote nodes

**master dns server** -----> *"dns-master"* - (192.168.1.11) - Ubuntu Server - primary authoritative server

**slave dns server** -----> *"dns-slave"* - (192.168.1.22) - Ubuntu Server - secondary server for redundancy

<img width="672" height="260" alt="hosts" src="https://github.com/user-attachments/assets/02617400-3176-4212-9708-bd80798c0a8a" />





## Initial Deploy:


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






<details> 
 
<summary> Master </summary>



#### Create Zone


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


now my play for this part, 

`deploy_sina_zone.yaml`

```
- name: Deploy sina zone on master
  hosts: dns-master
  become: yes
  tasks:
    - name: Copy named.conf.local
      copy:
        src: ../files/named.conf.local
        dest: /etc/bind/named.conf.local
        owner: root
        group: bind
        mode: '0644'
      notify: restart bind

    - name: Copy db.sina zone file
      copy:
        src: ../files/db.sina
        dest: /etc/bind/db.sina
        owner: root
        group: bind
        mode: '0644'
      notify: restart bind

  handlers:
    - name: restart bind
      systemd:
        name: bind9
        state: restarted
```




<img width="1647" height="450" alt="deploy" src="https://github.com/user-attachments/assets/307d4b34-adec-47f6-b37a-9a033ee9dc8a" />


now *dns-master* is ready.

</details> 





<details> 
 
<summary> Slave </summary>


`named.conf.local.slave` (to send to the slave mashin),

```
zone "sina" {
    type slave;
    file "/var/lib/bind/db.sina";
    masters { 192.168.1.11; };
};
```

and, play for setting the conf file to start the syncing
`deploy_slave.yaml`

```
- name: Deploy BIND on slave
  hosts: dns-slave
  become: yes

  tasks:
    - name: Install BIND9
      apt:
        name: bind9
        state: present
        update_cache: yes

    - name: Copy named.conf.local for slave
      copy:
        src: ../../files/slave/named.conf.local.slave
        dest: /etc/bind/named.conf.local
        owner: root
        group: bind
        mode: '0644'
      notify: restart bind

    - name: Ensure BIND is running
      systemd:
        name: bind9
        state: started
        enabled: yes

  handlers:
    - name: restart bind
      systemd:
        name: bind9
        state: restarted
```


now a test:

<img width="952" height="552" alt="dig slave" src="https://github.com/user-attachments/assets/47595b54-ed70-45a1-9bcf-49a37742189f" />


and a test for ensuring the servers synchronization after adding a new record on master (`test.sina`):

<img width="1637" height="831" alt="logcheck" src="https://github.com/user-attachments/assets/e62f43b6-61ad-4af6-8bce-da660f36a714" />



*dns-master* is ready.


</details> 

-------------------------------------------------------------------



## Tasks and Scenarios:


trying to automate some task.


#### Add Record
( Insert a selected DNS record )


#### Modify Record
( Delete or update a record )


#### Configure Cache-Only Mode
( Set up a node as a caching resolver )














