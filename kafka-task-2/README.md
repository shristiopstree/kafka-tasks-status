# Open the given below link

https://www.confluent.io/installer/machines and follow the steps given below

#

# Ansible installation
1. Environment
2. Machines
3. Security
4. Dependencies
5. Installation

## Confluent Platform Deployment

This repository contains the resources and instructions for deploying Confluent Platform across six remote machines.

## Prerequisites

### Machine Requirements
Ensure each machine meets the following specifications:
- **Hardware**:
  - At least 2 vCPUs
  - 8 GB of RAM
  - 8 GB of storage
- **Operating System**:
  - RHEL/CentOS 7.x or 8.x
  - Debian 9
  - Ubuntu 16.04 LTS or 18.04 LTS
- **Network**:
  - Machines must be able to connect to each other via specific TCP ports.
  - Machines must be able to reach the Confluent software package repository at `https://packages.confluent.io`.

### Access Requirements
- **SSH Access**:
  - A compatible SSH private key stored on your workstation.
  - The private key must allow access to all six machines as a Linux user with superuser (sudo) privileges.
- **Hostnames**:
  - Ensure you can SSH into each machine using its hostname or IP address from your workstation.

## Configuration

Provide the following information to configure the installation:

1. **Linux Username**: 
   Enter the username that has sudo privileges on all machines.

2. **SSH Private Key Path**:
   Enter the file path to the SSH private key on your workstation.

3. **Hostnames or IP Addresses**:
   List the hostnames or IPs of the six machines:
   - Machine 1
   - Machine 2
   - Machine 3
   - Machine 4
   - Machine 5
   - Machine 6

#
# Confluent Platform Setup Instructions

## Overview
This document details the configuration and firewall rules required to deploy Confluent Platform services across the specified EC2 instances in AWS.

## Services Deployment Plan

- **ZooKeeper** (Single-node cluster): 
  - **Node:** `ec2-3-145-167-35.us-east-2.compute.amazonaws.com`

- **Kafka** (Three-node cluster):
  - **Nodes:** 
    - `ec2-18-191-124-234.us-east-2.compute.amazonaws.com`
    - `ec2-3-142-251-120.us-east-2.compute.amazonaws.com`
    - `ec2-18-188-50-26.us-east-2.compute.amazonaws.com`

- **Schema Registry, ksqlDB, Kafka Connect:**
  - **Node:** `ec2-18-227-46-65.us-east-2.compute.amazonaws.com`

- **Control Center:**
  - **Node:** `ec2-3-16-203-126.us-east-2.compute.amazonaws.com`

## Firewall and Network Security Group Rules

### Egress Access for All Machines
```plaintext
Allow
port: any
protocol: any
to ip/ipBlock: 0.0.0.0/0
```

### SSH Access for All Machines
```plaintext
Allow
port: 22
protocol: TCP
from ip/ipBlock: your workstation
```

### Node-Specific Rules

#### `ec2-3-145-167-35.us-east-2.compute.amazonaws.com` (ZooKeeper)
```plaintext
Allow
port: 2181
protocol: TCP
from ip/ipBlock: ec2-18-191-124-234.us-east-2.compute.amazonaws.com, ec2-3-142-251-120.us-east-2.compute.amazonaws.com, ec2-18-188-50-26.us-east-2.compute.amazonaws.com
```

#### Kafka Nodes (`ec2-18-191-124-234.us-east-2.compute.amazonaws.com`, `ec2-3-142-251-120.us-east-2.compute.amazonaws.com`, `ec2-18-188-50-26.us-east-2.compute.amazonaws.com`)

- **Inter Broker Communication**
```plaintext
Allow
port: 9091
protocol: TCP
from ip/ipBlock: ec2-18-191-124-234.us-east-2.compute.amazonaws.com, ec2-3-142-251-120.us-east-2.compute.amazonaws.com, ec2-18-188-50-26.us-east-2.compute.amazonaws.com
```

- **Kafka AdminClient API, Custom Applications**
```plaintext
Allow
port: 9092
protocol: TCP
from ip/ipBlock: 0.0.0.0/0
```
*Alternatively:*
```plaintext
Allow
port: 9092
protocol: TCP
from ip/ipBlock: ec2-18-227-46-65.us-east-2.compute.amazonaws.com, ec2-3-16-203-126.us-east-2.compute.amazonaws.com, and your workstation
```

- **MDS & Embedded Kafka Rest** *(Optional)*
```plaintext
Allow
port: 8090
protocol: TCP
to ip/ipBlock: 0.0.0.0/0
```
*Alternatively:*
```plaintext
Allow
port: 8090
protocol: TCP
from ip/ipBlock: ec2-3-16-203-126.us-east-2.compute.amazonaws.com and your workstation
```

- **Standalone REST Proxy** *(Optional)*
```plaintext
Allow
port: 8082
protocol: TCP
to ip/ipBlock: 0.0.0.0/0
```
*Alternatively:*
```plaintext
Allow
port: 8082
protocol: TCP
from ip/ipBlock: ec2-3-16-203-126.us-east-2.compute.amazonaws.com and your workstation
```

#### `ec2-18-227-46-65.us-east-2.compute.amazonaws.com` (Schema Registry, ksqlDB, Kafka Connect)

- **Kafka Connect**
```plaintext
Allow
port: 8083
protocol: TCP
from ip/ipBlock: 0.0.0.0/0
```
*Alternatively:*
```plaintext
Allow
port: 8083
protocol: TCP
from ip/ipBlock: ec2-3-16-203-126.us-east-2.compute.amazonaws.com and your workstation
```

- **KSQL**
```plaintext
Allow
port: 8088
protocol: TCP
from ip/ipBlock: 0.0.0.0/0
```
*Alternatively:*
```plaintext
Allow
port: 8088
protocol: TCP
from ip/ipBlock: ec2-3-16-203-126.us-east-2.compute.amazonaws.com and your workstation
```

- **Schema Registry**
```plaintext
Allow
port: 8081
protocol: TCP
from ip/ipBlock: 0.0.0.0/0
```
*Alternatively:*
```plaintext
Allow
port: 8081
protocol: TCP
from ip/ipBlock: ec2-3-16-203-126.us-east-2.compute.amazonaws.com and your workstation
```

#### `ec2-3-16-203-126.us-east-2.compute.amazonaws.com` (Control Center)

- **Confluent Control Center (Browser Access)**
```plaintext
Allow
port: 9021
protocol: TCP
from ip/ipBlock: 0.0.0.0/0
```
*Alternatively:*
```plaintext
Allow
port: 9021
protocol: TCP
from ip/ipBlock: your workstation
```

- **Metadata Service and Embedded Kafka REST** *(Optional)*
```plaintext
Allow
port: 8090
protocol: TCP
to ip/ipBlock: 0.0.0.0/0
```
*Alternatively:*
```plaintext
Allow
port: 8090
protocol: TCP
from ip/ipBlock: ec2-3-16-203-126.us-east-2.compute.amazonaws.com and your workstation
```

## Notes
- Adjust the firewall rules for more restrictive ingress based on your security requirements.
- Ensure the rules are applied to the correct security groups in AWS.
- Replace `your workstation` with the actual IP or CIDR block of your local machine for secure access.

#

# Confluent Platform Installation using Ansible

This guide provides the steps to install the Confluent Platform on your machines using Ansible.

## Prerequisites

Before you begin, ensure that the following are in place:

1. **Ansible** is installed on your local machine.
2. **SSH access** to the target machines (with private key).
3. **Ubuntu user** has sudo privileges on the target machines.
4. The target machines should be accessible via SSH.

#
## Steps

### 1. Install the Confluent Platform Ansible Playbook

Run the following command to download the Confluent Platform (CP) Ansible Playbook from GitHub:

```bash
ansible-galaxy collection install git+https://github.com/confluentinc/cp-ansible.git
```

### 2. Create the `ansible.cfg` Configuration File

In your current directory, create a file called `ansible.cfg` and paste the following content:

```ini
[defaults]
hash_behaviour=merge
```

### 3. Create the `hosts.yml` Inventory File

In your current directory, create a file called `hosts.yml` and paste the following content. This configuration defines the target hosts for different components of the Confluent Platform:

```yaml
all:
  vars:
    ansible_connection: ssh
    ansible_user: ubuntu
    ansible_become: true
    ansible_ssh_private_key_file: /tmp/certs/ssh_priv.pem
    ansible_ssh_common_args: -o StrictHostKeyChecking=no
zookeeper:
  hosts:
    ec2-3-145-167-35.us-east-2.compute.amazonaws.com:
kafka_broker:
  hosts:
    ec2-18-191-124-234.us-east-2.compute.amazonaws.com:
    ec2-3-142-251-120.us-east-2.compute.amazonaws.com:
    ec2-18-188-50-26.us-east-2.compute.amazonaws.com:
schema_registry:
  hosts:
    ec2-18-227-46-65.us-east-2.compute.amazonaws.com:
ksql:
  hosts:
    ec2-18-227-46-65.us-east-2.compute.amazonaws.com:
kafka_connect:
  hosts:
    ec2-18-227-46-65.us-east-2.compute.amazonaws.com:
      vars:
        kafka_connect_confluent_hub_plugins:
          - confluentinc/kafka-connect-datagen:0.4.0
control_center:
  hosts:
    ec2-3-16-203-126.us-east-2.compute.amazonaws.com:
```

### 4. Run the Ansible Playbook

Once the `ansible.cfg` and `hosts.yml` files are in place, run the following Ansible command to start the Confluent Platform installation:

```bash
ansible-playbook -i hosts.yml confluent.platform.all
```

This command will install all components of the Confluent Platform on the specified hosts.

### 5. Monitor the Playbook Execution

The playbook will take about 15 minutes to complete. You can monitor the progress in your terminal. If the playbook runs successfully, the final output will show `failed=0` for all the machines in the inventory, indicating that there were no errors during the installation process.

Example of a successful playbook execution output:

```
PLAY RECAP
******************************************************************************************************************************************************
ec2-3-145-167-35.us-east-2.compute.amazonaws.com      : ok=30  changed=15  unreachable=0  failed=0  skipped=27  rescued=0  ignored=0
ec2-18-191-124-234.us-east-2.compute.amazonaws.com    : ok=38  changed=25  unreachable=0  failed=0  skipped=30  rescued=0  ignored=0
ec2-3-142-251-120.us-east-2.compute.amazonaws.com     : ok=38  changed=10  unreachable=0  failed=0  skipped=29  rescued=0  ignored=0
ec2-18-188-50-26.us-east-2.compute.amazonaws.com      : ok=38  changed=12  unreachable=0  failed=0  skipped=29  rescued=0  ignored=0
ec2-18-227-46-65.us-east-2.compute.amazonaws.com      : ok=81  changed=57  unreachable=0  failed=0  skipped=88  rescued=0  ignored=0
ec2-3-16-203-126.us-east-2.compute.amazonaws.com      : ok=81  changed=57  unreachable=0  failed=0  skipped=88  rescued=0  ignored=0
```

### Troubleshooting

- If you encounter any issues, check the output logs for specific error messages and address any connectivity or permission issues.
- Ensure that all target machines are accessible via SSH and that the private key path in the `hosts.yml` file is correct.

## Conclusion

By following these steps, the Confluent Platform will be installed on your specified EC2 instances. If successful, you should be able to access all the components like Zookeeper, Kafka, Schema Registry, KSQL, Kafka Connect, and Control Center.

Feel free to reach out for additional help!