# ansible-aws
Deployment on AWS using Ansible.

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

## Launch and Setup Instances

1. Run the playbook:\
`ansible-playbook setup_ec2.yml`

2. To create new instances, use the ec2_create tag:\
`ansible-playbook setup_ec2.yml --tags create_ec2`

3. Terminate all instances:\
`ansible-playbook setup_ec2.yml --tags clear_ec2`

4. Test Dynamic Inventory:\
`ansible-playbook setup_ec2.yml --tags create_ec2,test_inventory`

5. Create Instances and install Docker:
`ansible-playbook setup_ec2.yml --tags create_ec2,install_docker`

5. Login to instances:\
`ssh -i ~/.ssh/my_aws ubuntu@(address)`


