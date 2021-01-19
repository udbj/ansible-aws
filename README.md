# ansible-aws
A single Ansible script to create EC2 instances and set up a dockerised couchDB cluster on them. 

## Initial Setup
1. Generate Access and Secret keys:\
`(Username tab) > My Security Credentials > Access Keys > Create New Access Key`

2. Setup Ansible inventory folder:\
`mkdir -p /opt/ansible/inventory`

3. Under the `inventory` folder:
  * Create Ansible vault and vault pass files:
  ```
  openssl rand -base64 2048 > vault.pass
  
  ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass
  ```
  
  * Create `aws_ec2.yml` script file for dynamic inventory:
  ```
  ---
  plugin: aws_ec2
  aws_access_key: ACCESSKEY
  aws_secret_key: SECRETKEY

  regions:
    - ap-southeast-2

  keyed_groups:
    - key: tags
      prefix: tag

  ```
  
  * Final directory structure:
  ```
  |-/
  | |-opt
  | |   |-ansible
  | |   |       |-inventory
  | |   |       |         |-aws_ec2.yml
  | |   |       |         |-vault.pass
  | |   |       |         |
```
  
4. Generate SSH key:\
`ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_aws`

5. Create Ansible config file:
```
sudo mkdir /etc/ansible
sudo touch /etc/ansible/ansible.cfg
```

6. In the Ansible config file, add the following:
```
[defaults]
inventory = /opt/ansible/inventory/aws_ec2.yml
vault_password_file = /opt/ansible/inventory/vault.pass
private_key_file = /Users/Dave/.ssh/my_aws
host_key_checking = False
remote_user = ubuntu

[inventory]
enable_plugins = aws_ec2
```
7. Use `couchdb-hash-pwd` to generate pre-hashed, pre-salted password for every CouchDB node.

8. Generate cluster secret and UUID using `uuidgen`.

9. Create variables file for the script:
```
---
ec2_access_key: ACCESSKEY
ec2_secret_key: SECRETKEY
couchdb_name: COUCHDB
couchdb_hashed_pass: -pbkdf2-preHashedPassword
couchdb_pass: COUCHDBPASS
erl_cookie: ERLCOOKIE
couchdb_secret: COUCHDB-SECRET-UUID
cluster_id: COUCHDB-CLUSTER-UUID
```

## Launch and Setup Instances

1. To set up everything, run:\
`ansible-playbook setup_ec2.yml --tags setup_full`

2. To do everything step-by-step:

* To create new instances:\
`ansible-playbook setup_ec2.yml --tags create_ec2`

* Test Dynamic Inventory:\
`ansible-playbook setup_ec2.yml --tags test_inventory`

* Setup Docker on created instances:\
`ansible-playbook setup_ec2.yml --tags setup_docker`

* Setup CouchDB on Docker:\
`ansible-playbook setup_ec2.yml --tags setup_couchdb`

3. Login to instances by specifying the `ssh` key:\
`ssh -i ~/.ssh/my_aws ubuntu@(address)`

4. To check which nodes are connected to cluster:\
`curl http://USERNAME:PASSWORD@(address):5984/_membership`

5. Terminate all instances:\
`ansible-playbook setup_ec2.yml --tags clear_ec2`

## Notes
* The script allows traffic from all addresses. Configure the security rules as per your needs.
