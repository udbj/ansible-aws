# ansible-aws
Deployment on AWS using Ansible.

# Launch new instances
1. Create SSH key using:
  ssh-keygen -t rsa -b 4096 -f ~/.ssh/my_aws

2. Create vault pass file:
  openssl rand -base64 2048 > vault.pass

3. Create Ansible vault file:
  ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass

4. Run the playbook:
  ansible-playbook playbook.yml --vault-password-file vault.pass

5. To create new instances, use the ec2_create tag:
  ansible-playbook playbook.yml --ask-vault-pass --tags create_ec2


