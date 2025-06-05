>Ansible uses **connection modules** to communicate with managed systems, and when working with **AWS**, it relies on modules built with the **`boto` or `boto3` libraries** (Python SDKs for AWS). These modules allow Ansible to manage AWS resources such as EC2 instances, S3 buckets, IAM users, and more. To use them, you must have `boto` (for older modules) or `boto3` (for most modern ones) installed and properly configured with AWS credentials, either via environment variables, config files, or IAM roles.

AWS concepts:
- **EC2 (Elastic Compute Cloud)** – Virtual servers to run applications in the cloud.
- **S3 (Simple Storage Service)** – Scalable object storage for files, backups, and static content.
- **RDS (Relational Database Service)** – Managed relational databases like MySQL, PostgreSQL, and Aurora.
- **VPC (Virtual Private Cloud)** – Isolated network environment where you launch AWS resources.
- **IAM (Identity and Access Management)** – Controls who can access what in your AWS account.
- **Lambda** – Run code without managing servers (serverless functions).
- **CloudFormation** – Define infrastructure as code using YAML or JSON templates.
- **CloudWatch** – Monitor AWS resources and applications with logs, metrics, and alerts.
- **Route 53** – Scalable DNS and domain registration service.
- **ELB (Elastic Load Balancer)** – Distributes traffic across multiple EC2 instances or services.
- **EBS (Elastic Block Store)** – Persistent block storage volumes for EC2 instances.
- **SQS (Simple Queue Service)** – Managed message queuing for decoupling services.
- **SNS (Simple Notification Service)** – Pub/sub messaging for notifications and alerts.
- **ECS (Elastic Container Service)** – Container orchestration service for running Docker containers.
- **CloudTrail** – Tracks user activity and API calls for auditing and security.

First we need to install `boto` and `boto3`:
`pip3 install boto boto3` 

Create key-pair and access key on AWS and configure in .ssh in order to be able to connect to AWS instances.

To use Ansible’s AWS modules like `ec2_group` (which rely on **`boto` or `boto3`**), you typically need to export your AWS credentials as **environment variables** so Ansible can authenticate with AWS.

```bash
export AWS_ACCESS_KEY_ID="your-access-key-id"
export AWS_SECRET_ACCESS_KEY="your-secret-access-key"
export AWS_DEFAULT_REGION="us-east-1"  # or another region
```

```yaml
---
- hosts: localhost
  connection: local
  gather_facts: false
 
  vars:
    desired_instances: 20

  tasks:
    - name: Create a security group in AWS for SSH access and HTTP
      ec2_group:
         name: ansible
         description: Ansible Security Group
         region: us-east-1
         rules:
            - proto: tcp
              from_port: 80
              to_port: 80
              cidr_ip: 0.0.0.0/0
            - proto: tcp
              from_port: 22
              to_port: 22
              cidr_ip: 0.0.0.0/0

    # Capture all instances that already have a tag of Ansible
    - name: Gather information about existing instances
      ec2_instance_info:
        filters:
          "tag:Name": Ansible
          instance-state-name: running
        region: us-east-1
      register: ec2_info

    # Compare the number of instances to launch vs those available, avoid negative numbers
    - name: Calculate the number of instances to launch
      set_fact:
        # Ensure it is never less than zero
        instances_to_launch: "{{ [0, desired_instances - (ec2_info.instances | length)] | max }}"

    # Even with zero instances requested, this needs to be executed 
    # to populate information about existing instances
    - name: Provision a set of instances and/or capture existing information
      ec2_instance:
        key_name: ansible
        name: Ansible
        security_group: ansible
        instance_type: t2.micro
        image_id: ami-0fe630eb857a6ec83
        region: us-east-1
        wait: true
        count: "{{ instances_to_launch }}"
      register: ec2

    - name: Refresh inventory to ensure new instances exist in inventory
      meta: refresh_inventory

- hosts: tag_Name_Ansible
  gather_facts: false
    
  tasks:
    - name: Wait for SSH to be online
      wait_for_connection:
        delay: 10
        timeout: 320
      vars:
        ansible_ssh_private_key_file: /home/ansible/.ssh/ansible.pem
        ansible_user: ec2-user
      retries: 20

- hosts: tag_Name_Ansible
    
  # Roles: list of roles to be imported into the play
  roles:
    - { role: webapp, target_dir: /usr/share/nginx/html }

- hosts: tag_Name_Ansible

  tasks:
    - debug:
        msg: "Check http://{{ ansible_host }}"

    - pause:
        prompt: "Verify service availability and continue to terminate"

    - name: Terminate EC2 instances
      ec2_instance:
        state: absent
        instance_ids: "{{ instance_id }}"
        region: "{{ placement.region }}"
        wait: true
      delegate_to: localhost

- hosts: localhost
  connection: local
  gather_facts: false

  tasks:
  - name: Remove ansible security group
    ec2_group:
      name: ansible
      region: us-east-1
      state: absent
    register: result
    until: result is success
    retries: 20
    delay: 10
...
```

A **dynamic inventory** automatically pulls the list of hosts from a cloud provider like **AWS**, instead of manually defining them in a static `inventory` file. This is essential when your infrastructure is dynamic — e.g., EC2 instances being created or terminated frequently.

```ini
[defaults]
inventory = ./inventory
enable_plugins = aws_ec2
host_key_checking = False
forks = 20
ansible_managed = Managed by Ansible - file:{file} - host:{host} - uid:{uid}
```
(enables `aws_ec2` inventory plugin)

`inventory/aws_ec2.yaml`
```yaml
plugin: amazon.aws.aws_ec2
cache: false
keyed_groups:
  - prefix: tag
    key: tags
```

**List dynamic inventories:**
`ansible-inventory -i inventory/aws_ec2.yaml --graph`

