# ha_postgres_cluster

# PostgreSQL High-Availability Cluster wih Patroni, etcd, HaProxy, PgBouncer & Ansible :elephant: 


`ha_postgres_cluster` automates the deployment and management of highly available PostgreSQL clusters in production environments. This solution is tailored for use on dedicated physical servers, virtual machines, and within both on-premises and cloud-based infrastructures.

---

### HA CLUSTER ARCHITECTURE

<img width="1010" alt="image" src="https://github.com/user-attachments/assets/39f55240-4063-41bd-a273-006697a0f73e">


#### 1. PostgreSQL High-Availability with Load Balancing

This solution enables load distribution for read operations and also allows for scaling out the cluster with read-only replicas. 

##### Components:

- [**Patroni**](https://github.com/zalando/patroni) is a template for you to create your own customized, high-availability solution using Python and - for maximum accessibility - a distributed configuration store like ZooKeeper, etcd, Consul or Kubernetes. Used for automate the management of PostgreSQL instances and auto failover.

- [**etcd**](https://github.com/etcd-io/etcd) is a distributed reliable key-value store for the most critical data of a distributed system. etcd is written in Go and uses the [Raft](https://raft.github.io/) consensus algorithm to manage a highly-available replicated log. It is used by Patroni to store information about the status of the cluster and PostgreSQL configuration parameters.

- [**PgBouncer**](https://pgbouncer.github.io/features.html) is a connection pooler for PostgreSQL.

##### Components of HAProxy load balancing:

- [**HAProxy**](http://www.haproxy.org/) is a free, very fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications. 

- [**Keepalived**](https://github.com/acassen/keepalived)  (optional) provides a virtual high-available IP address (VIP) and single entry point for databases access.
Implementing VRRP (Virtual Router Redundancy Protocol) for Linux. In our configuration keepalived checks the status of the HAProxy service and in case of a failure delegates the VIP to another server in the cluster.

> [!NOTE]
> Your application must have support sending read requests to a custom port 5001, and write requests to port 5000.

List of ports when using HAProxy:
- port 5000 (read / write) master
- port 5001 (read only) all replicas
- port 5002 (read only) synchronous replica only
- port 5003 (read only) asynchronous replicas only


References: https://github.com/vitabaks/postgresql_cluster/blob/master/README.md


### Installation Steps:

<details><summary>Click here to expand</summary><p>

0. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) on one control node (which could easily be a laptop)

```
sudo apt update && sudo apt install -y python3-pip sshpass git
pip3 install ansible
```

1. Download or clone this repository

```
git clone https://github.com/mhmt1420/ha_postgres_cluster.git
```

2. Go to the directory


3. Install depedencies on the Ansible control node

```
 # Update system packages
sudo apt update && sudo apt upgrade -y

# Install Python3 and Pip
sudo apt install -y python3 python3-pip

# Install Ansible
sudo apt install -y ansible

# Install required Python libraries
pip3 install psycopg2-binary netaddr jinja2 pyyaml requests

```

4. Edit the inventory file based on your environment!

Specify (non-public) IP addresses and connection settings such as (`ansible_user`) etc. for your environment

5. Edit the variable file main.yml

6. Try to connect to hosts from ansible server

7. Run playbooks in-order as below:

```
ansible-playbook -i inventory.yaml main.yml -kK
```

```
ansible-playbook -i inventory-etcd.yml configure-etcd.yml -kK
```

```
ansible-playbook -i inventory-haproxy.yml haproxy-conf.yml -kK
```

```
ansible-playbook -i inventory-keepalived.yml keepalived-conf.yml -kK
```

```
ansible-playbook -i inventory-haproxy.yml keepalived-conf.yml -kK
```

```
ansible-playbook -i inventory.yaml pgbouncer-conf.yml -kK
```




> **Note**  
> If there are missing `sudo` passwords in Ansible, try using the options `-kK`. It will prompt for passwords.  
> - `-k` (`--ask-pass`): Ask for connection password.  
> - `-K` (`--ask-become-pass`): Ask for privilege escalation password.  
> Additionally, the `sshpass` program must be installed.


