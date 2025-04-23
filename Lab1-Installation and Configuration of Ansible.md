## Installation and Configuration of Ansible 

### Task 1: Installing on a VM 
Here we have taken RHEL but the same can be done on Ubutu as well. The package manager changes.

`***NOTE: Our labs are designed to work RHEL machines.***`

* Launch a RHEL 9 instance in us-east-1. 
* Choose t2.micro. 
* In security group, allow SSH (22), HTTP (80), and HTTPS (443) for all incoming traffic. 
* Add Tag Name: control-node

Once the EC2 is up & running, SSH into one of it and set the hostname as 'Control-Node'. 
```
sudo hostnamectl set-hostname control-node
```
```
bash
```

Update the package repository with latest available versions
```
sudo yum check-update
```

Install latest version of Python. 
```
sudo yum install python3-pip -y 
```
```
python3 --version
```
```
sudo pip3 install --upgrade pip
```
Install awscli, boto, boto3 and ansible. Boto/Boto3 are AWS SDK which will be needed while accessing AWS APIs
```
sudo pip3 install awscli boto boto3
```
```
sudo pip3 install ansible
```
```
pip show ansible
```

Authorize aws credentials
```
aws configure
```
#### Enter the Credentials as below. Example:
| **Access Key ID**    | **Secret Access Key** |
| -----------------    | --------------------- |
| 77AKIAWJXSSHRD27T6SC | 777H4Vh0U5oenKfmJ/+FEnGAmZvQLX7zTT |

```
sudo yum install wget -y
```

Create the Script file to launch 2 more instances.
```
vi ansible_script.yaml
```
```
---
- hosts: localhost
  connection: local

  tasks:
    - name: Execute curl command to get token
      shell: "curl -X PUT 'http://169.254.169.254/latest/api/token' -H 'X-aws-ec2-metadata-token-ttl-seconds: 21600'"
      register: TOKEN

    - name: Get region of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/placement/region/"
      register: region

    - name: Get AMI ID of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/ami-id"
      register: ami_id

    - name: Get keypair of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/public-keys/| cut -c 3-100 "
      register: kp

    - name: Get Instance Type of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/instance-type"
      register: instance_type


    - name: Get subnet id of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs/$(curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs)/subnet-id"
      register: subnet

    - name: Get security group of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs/$(curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs)/security-group-ids/"
      register: secgrp


    - name: Generate SSH keypair
      openssh_keypair:
        force: yes
        path: /home/ec2-user/.ssh/id_rsa

    - name: Get the public key
      shell: cat /home/ec2-user/.ssh/id_rsa.pub
      register: pubkey

    - name: Create EC2 instance
      ec2_instance:
        key_name: "{{ kp.stdout }}"
        security_group: "{{ secgrp.stdout }}"
        instance_type: "{{ instance_type.stdout }}"
        image_id: "{{ ami_id.stdout }}"         # "ami-0931978297f275f71"
        wait: true
        region: "{{ region.stdout }}"
        tags:
          Name: "{{ item }}"
        vpc_subnet_id: "{{ subnet.stdout }}"
        network:
         assign_public_ip: yes
        user_data: |
           #!/bin/bash
           echo "{{ pubkey.stdout }}" >> /home/ec2-user/.ssh/authorized_keys
      register: ec2var
      loop:
          - managed-node1
          - managed-nodes2

    - name: Make ansible directory
      file:
        path: /etc/ansible
        state: directory
      become: yes

    - debug:
        msg: "{{ ec2var.results[0].instances[0].private_ip_address }}"

    - debug:
        msg: "{{ ec2var.results[1].instances[0].private_ip_address }}"

```

Execute the script
```
ansible-playbook ansible_script.yaml
```

Once you get the ip addresses, do the following:

```
sudo vi /etc/ansible/hosts
```

Add the prive IP addresses, by pressing "INSERT" 
```
managed-node1   ansible_ssh_host=<private-ip-managed-node1>  ansible_ssh_user=ec2-user

managed-node2   ansible_ssh_host=<private-ip-managed-node2>  ansible_ssh_user=ec2-user
 
```
![image](https://github.com/user-attachments/assets/a646936e-2b79-4da1-857f-87d6599e4ad2)


Save the file using "ESCAPE + :wq!"

list all managed node ip addresses.
```
ansible all --list-hosts
```
![image](https://github.com/user-attachments/assets/bf61d4ab-1556-4728-ad39-7a1bb7fa7ddc)


### Task 2:  SSH into each of them and set the hostnames.
```
ssh ec2-user@< Replace Node 1 IP >
```
```
sudo hostnamectl set-hostname managed-node1
```
```
exit
```
```
ssh ec2-user@< Replace Node 2 IP >
```
```
sudo hostnamectl set-hostname managed-node2
```
```
exit
```

Use ping module to check if the managed nodes are able to interpret the ansible modules
```
ansible all -m ping
```
![image](https://github.com/user-attachments/assets/e7e246ad-4b51-4e8e-b4c5-8c4aba695c82)
