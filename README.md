# ansible-aws
Deployment on AWS using Ansible.

## Initial Setup
1. Generate Access and Secret keys:\
`(Username tab) > My Security Credentials > Access Keys > Create New Access Key`

2. Create Ansible config file:\
```
sudo mkdir /etc/ansible
sudo touch /etc/ansible/ansible.cfg
```

3. Setup Ansible vars folder:
```
|-/
| |-opt
| |   |-ansible
| |   |       |-inventory
| |   |       |         |-group_vars
| |   |       |         |          |-all
| |   |       |         |          |
```
Under the inventory folder:
  * Create Ansible vault and vault pass files:
  ```
  openssl rand -base64 2048 > vault.pass
  
  ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass
  ```
  
  * Create `aws_ec2.yml` script file for dynamic inventory:
  ```
  ---
  plugin: aws_ec2
  aws_access_key: ABCD
  aws_secret_key: 1234

  regions:
    - ap-southeast-2

  keyed_groups:
    - key: tags
      prefix: tag

  compose:
    ansible_user: "{{ansible_user}}"
    ansible_ssh_private_key_file: "{{key_file}}"
  ```
  
4. Generate SSH key:\
`ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_aws`

5. In the Ansible config file, add the following:
```
[defaults]
inventory = /opt/ansible/inventory/aws_ec2.yml
vault_password_file = /opt/ansible/inventory/vault.pass
private_key_file = /Users/Dave/.ssh/my_aws
host_key_checking = False

[inventory]
enable_plugins = aws_ec2
```

## Launch new instances

4. Run the playbook:\
`ansible-playbook launch_instance.yml --vault-password-file vault.pass`

5. To create new instances, use the ec2_create tag:\
`ansible-playbook launch_instance.yml --vault-password-file vault.pass --tags create_ec2`

6. Terminate all instances:\
`ansible-playbook launch_instance.yml --vault-password-file vault.pass --tags clear_ec2`

7. Login to instances:\
`ssh -i ~/.ssh/my_aws ubuntu@(address)`

