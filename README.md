# Self-Managed Kubernetes Cluster deployement with Terraform and Ansible
This repositrory contains the files to deploy a Kubernetes cluster on AWS. The infrastructure is deployed with Terraform wheras the cluster configuration is done with Ansible.

🧠 The core idea

* Terraform is responsible for creating the infrastructure
* Ansible is responsible for configuring it
* The “contract” between them is inventory + variables

## Version v1:
This version conprises 1 control node + 1 worker node. Both nodes run as stand alone nodes (i.e. not in an ASG).

We use Terraform to generate Ansible inventory file dynamically. To do so Terraform uses a template file that it polulates at runtime. Once the infrastructure is created you can check connectivity wiith "ansible -m ping all".

<pre>
v1
├── ansible
│   ├── ansible.cfg
│   ├── inventory
│   │   └── local-hosts.init
│   ├── playbooks
│   │   ├── k8s-cluster-setup.yaml
│   │   ├── k8s-common.yaml
│   │   ├── k8s-control.yaml
│   │   └── k8s-workers.yaml
│   └── templates
│       ├── kubeadm-config.yaml
│       └── kubeadm-config.yaml.j2
└── terraform
    ├── inventory.tpl
    ├── main.tf
    └── variable.tf


</pre>


To deploy v1, navigate to the terraform directory and run

If not done already initialize your terraform workspace.

    $ terraform init

then preview the changes Terraform will make with 

    $ terraform plan

then makes the changes defined by your plan to create all resources with

    $ terraform apply

Terraform will create the inventory file that ansible uses to access the node. To check that the nodes are reachable, navigate to the ansible directory and run

    $ ansible -m ping all

To setup the K8s cluster, navigate to the ansible directory and run

        $ ansible-playbook playbooks/k8s-cluster-setup.yaml   


## version v2:

With this version we create 1 control node with 2 worker nodes created in an ASG. In this version we enable inventory plugins in ansible to dynamically retrieve the worker nodes IP addresses. 

```
v2
├── ansible
│   ├── ansible.cfg
│   ├── aws_ec2.yaml
│   ├── inventory
│   │   └── local-hosts.init
│   ├── playbooks
│   │   ├── k8s-cluster-setup.yaml
│   │   ├── k8s-common.yaml
│   │   ├── k8s-control.yaml
│   │   └── k8s-workers.yaml
│   └── templates
│       ├── kubeadm-config.yaml
│       └── kubeadm-config.yaml.j2
└── terraform
    ├── inventory.tpl
    ├── main.tf
    ├── terraform.tfstate
    ├── terraform.tfstate.backup
    └── variable.tf
```

✅ Step 1: Make sure the AWS collection is installed

In this version you need amazom.aws collection for ansible. The EC2 inventory plugin lives in the amazon.aws collection.

To check if it's install run the command

    $ ansible-galaxy collection list | grep amazon.aws

if nothing shows up → install it:

    $ ansible-galaxy collection install amazon.aws

Also required (on your local machine) is the aws python sdk:

    $ pip install boto3 botocore

Verify boto3 is availavle

    python3 - <<EOF
    import boto3
    print("boto3 OK")
    EOF

✅ Step 2: Enable inventory plugins in ansible.cfg (THIS IS THE BIG ONE)

Ansible does not auto-enable dynamic inventory plugins unless configured.

Create or update ansible/ansible.cfg:

    [defaults]
    inventory = ./ansible
    host_key_checking = False
    interpreter_python = auto_silent

    [inventory]
    enable_plugins = aws_ec2, yaml, ini

📌 Important

enable_plugins is mandatory
aws_ec2 must be explicitly listed

✅ Step 3: Verify your inventory file path & name

Your inventory must end in .yml or .yaml and live in the inventory directory.


To deploy v2, navigate to the terraform directory and run

    $ terraform apply

To setup the K8s cluster, navigate to the ansible directory and run

        $ ansible-playbook playbooks/k8s-cluster-setup.yaml

        
