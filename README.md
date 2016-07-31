# ansible-ec2-mxnet
This repository is an deployment example.
We choose **ansible** to be the deployment tool.
We want to deploy a **mxnet-GPU version on aws ec2**.

# Preparing AWS
We use aws ec2 for this example, start by creating EC2 Seurity Group for your application.
And create a port(ex: TCP/22) for SSH.

Then, we'll get a keypair.pem file for SSH to your instance. 
remember to keep keypair.pem in location ~/.ssh/

# Install ansible
- ansible installation , http://docs.ansible.com/ansible/intro_installation.html <br>

# Ansible Playbook structure
```
deploy.yml
inventory
group_vars/
  all.yml
roles/
  launch/tasks/
  dependency/tasks/
  cuda/tasks/
  mxnet/tasks
```
# Setting instance environment
- groups/all.yml
Custom your ec2 environment, including keypair, security group, instance type, image, region... etc.
And remember, add your own aws crediential access/secret key.
```yaml
# ec2 variable
ec2_keypair: "your-key-pair"    # need to modify
ec2_security_group: "your-security-group"  # need to modify
ec2_instance_type: "your-instance-type" # ex: g2.2xlarge
ec2_image: "your-image"  # ex: ami-9abea4fb
ec2_region: "your-region" # ex: us-west-2
ec2_count: "1"
ec2_tag_Name: "your-ec2-instance-name" # need to modify
ec2_tag_Type: "your-tag-type" # need to modify
ec2_tag_Environment: "dev"
ec2_volume_size: 200 # 200 is represnet 200G size
 
# aws crediential
aws_access_key: "your-access-key"
aws_secret_key: "your-secret-key"
```

# Usage
- Custom variable need to be modified, the variables are in the following file
```
- group_vars/all.yml
- roles/cuda/tasks/main.yml
  line: https://{{your-path}}/cuda/cudnn-7.5-linux-x64-v5.1-rc.tgz
```
- After modified, run
```
ansible-playbook -i inventory deploy.yml
```

# deploy.yml overall

```yaml
- name: Provision an EC2 Instance
  hosts: localhost
  connection: local
  gather_facts: False
  tags: provisioning
  roles:
    - role: launch
      name: ami_build
  
- name: Deploy the application
  hosts: ami_build
  vars:
     ansible_ssh_user: "ubuntu"
     ansible_ssh_private_key_file: "~/.ssh/{{ec2_keypair}}.pem"
     ansible_ssh_port: 22
  roles:
    - dependency
    - cuda
    - mxnet

- name : Build AMI
  hosts: localhost
  connection: local
  gather_facts: True
  tasks:
    - name: Create AMI
      ec2_ami:
        instance_id: "{{ item.id }}"
        region: "{{ ec2_region }}"
        wait: no
        name: "{{ Your AMI name }}" # need th modify
        state: present
      register: ami
      with_items: "{{ ec2.instances }}"

```
We use three plays in the playbook. 
All the three plays share the same variable from group_vars/.

- First play
launch ec2 instance

- Second play
recevied the ec2 instance IP, and start to install dependency, cuda, mxnet

- Final play
save the instance images

# Why use deployment tool?
Deployment tool can record what we do, step by step.
We can clearly know what kind of packages are installed.
It is more convenient to maintain and modify when we transfer our ansible playbook to next deployer. 

# Reference
- [atplanet/ansible-auto-scaling-tutorial](https://github.com/atplanet/ansible-auto-scaling-tutorial)</br> 
- [mxnet offical guide](https://github.com/dmlc/mxnet/blob/master/docs/how_to/build.md)</br>
- [Deep learning for hackers with MXnet (1) GPU installation and MNIST](https://no2147483647.wordpress.com/2015/12/07/deep-learning-for-hackers-with-mxnet-1/)</br>

