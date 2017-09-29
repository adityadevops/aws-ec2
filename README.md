# aws-ec2

Using Ansible to provision EC2 instance.

# Setting up Ansible
Using Boto for Ansible - AWS interactions

```bash
add-apt-repository ppa:ansible/ansible
apt-get install ansible
```
# Sample boto

```ini
[Credentials]
aws_access_key_id = <your_access_key_here>
aws_secret_access_key = <your_secret_key_here>
```
## Step 1: Setup EC2 instance

``` yaml
- name: Launch base server
  ec2:
    assign_public_ip: yes
    group_id: "{{ sg_group_id }}"
    image: "{{ ami_id }}"
    instance_type: "{{ ec2_instance_type }}"
    instance_profile_name: "{{ profile_name }}"
    count_tag: 
      Group: "{{ tag_ec2_group }}"
    exact_count: "{{ instance_count }}"
    key_name: "{{ name_key }}"
    region: "{{ ec2_region }}"
    vpc_subnet_id: "{{ subnet_id }}"
    wait: yes
    assign_public_ip: yes
    instance_tags: {
      "Name": "tomcat",
    }
  register: ec2
 ```
 ## Step 2: Deploying Static page
 
 ``` yaml
 - name: Deploy war file
  template:
    src: index.html
    dest: /var/www/html/
    mode: '0777'
  register: war_deployed
 
- name: Restart tomcat
  service:
    name: httpd
    state: restarted
  when: war_deployed.changed
```
## Step 3: Add host to the selenium group

``` yaml
- name: Get the ipaddress of first instance
  set_fact:
     ip_address_inst: "{{ ec2.instances[0].public_ip }}"

- name: Add host to the selenium group
  add_host:
      groups: tomcat
      name: "{{ ip_address_inst }}"
      ansible_user: ec2-user
      ansible_port: 22
      ansible_ssh_private_key_file: "/tmp/new.pem"
      ansible_become_user: root
      ansible_become: true
 ```
  
