# BIND-Lab
### DNS management


deploying master and slave servers, creating and managing zones, adding DNS records and different DNS configurations




## inventory list:


**contorl node** -----> *"win11"* - (192.168.1.100) - Ubuntu 24.04 (WSL2) - runs Ansible and manages remote nodes

**master dns server** -----> *"dns-master"* - (192.168.1.11) - Ubuntu Server - primary authoritative server

**slave dns server** -----> *"dns-slave"* - (192.168.1.22) - Ubuntu Server - secondary server for redundancy





## Deploy:

first a connectivity check with machines:



