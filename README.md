# Google Kubernetes / RedHat OpenShift + OpenContrail SDN

Quickly bringup a Kubernetes cluster with OpenContrail SDN on different clouds (AWS/Azure/GCP/OpenStack)

## AWS CloudFormation

AWS [CloudFormation](https://aws.amazon.com/cloudformation/) provides a common language for you to describe and provision all the infrastructure resources in your cloud environment. CloudFormation allows you to use a simple text file to model and provision, in an automated and secure manner, all the resources needed for your applications across all regions and accounts. This file serves as the single source of truth for your cloud environment.

* Google Kubernetes with OpenContrail SDN

     <a href="https://console.aws.amazon.com/cloudformation/home?region=us-west-1#/stacks/new?stackName=contrail-k8s&amp;templateURL=https://s3-us-west-1.amazonaws.com/contrail-k8-501/k8s-contrail-stack-501.yaml" target="_blank"><img alt="Launch Stack" src="https://cdn.rawgit.com/buildkite/cloudformation-launch-stack-button-svg/master/launch-stack.svg"></a>


* RedHat OpenShift with OpenContrail SDN

     <a href="https://console.aws.amazon.com/cloudformation/home?region=us-west-1#/stacks/new?stackName=contrail-k8s&amp;templateURL=https://s3-us-west-1.amazonaws.com/contrail-k8-501/openshift-contrail-stack-5.yaml" target="_blank"><img alt="Launch Stack" src="https://cdn.rawgit.com/buildkite/cloudformation-launch-stack-button-svg/master/launch-stack.svg"></a>


## HashiCorp Terraform

HashiCorp [Terraform](https://www.terraform.io/) enables you to safely and predictably create, change, and improve infrastructure. It is an open source tool that codifies APIs into declarative configuration files that can be shared amongst team members, treated as code, edited, reviewed, and versioned.

* Kubernetes with OpenContrail SDN

  [Download](https://www.terraform.io/downloads.html) Terraform
  
          (host)# ssh-keygen -b 2048 -t rsa -f .ssh/terraform
          (host)# git clone https://github.com/savithruml/cloud-ops
          (host)# cd cloud-ops/aws/terraform/k8s
          (host)# vi terraform.tfvars
      
               ...
                    # AWS access key
                    aws_access_key            = "<access-key>"

                    # AWS secret key
                    aws_secret_key            = "<secret-key>"
               ...
  
          (host)# terraform apply
      
## Red Hat Ansible

Red Hat® [Ansible](https://www.ansible.com/) makes it easy to scale automation, manage complex deployments and speed productivity. Extend the power of Ansible with workflows to streamline jobs and simple tools to share solutions with your team.

* Kubernetes with OpenContrail SDN

    1) Bring your ansible node. This node can either sit indide the AWS environment or outside
    
            (ansible-node)# apt-get update -y && apt-get install python python-pip -y
            (ansible-node)# pip install -U ansible boto boto3
      
            (ansible-node)# cd /root
            (ansible-node)# git clone https://github.com/savithruml/ansible-labs
            
    2) Populate /root/ansible-labs/aws/playbooks/group_vars/all file with AWS creds & cluster info
      
            (ansible-node)# cat /root/ansible-labs/aws/playbooks/group_vars/all
            
                  aws_access_key: <key-here> 
                  aws_secret_key: <secret-key-here>
                  key_name: <key>
                  aws_region: <region>
                  vpc_id: <vpc>
                  vpc_subnet_id: <subnet>
                  ami_id: <image>
                  instance_type: <flavor>
                  count: 2
                  ec2_tag: contrail-k8s
                 
            (ansible-node)# cd /root/ansible-labs/aws       
            (ansible-node)# ansible-playbook -i inventory/hosts playbooks/deploy-vms.yml
            
        This should bring up 2 instances in AWS with root password set to "c0ntrail123"
    
    4) Prepare nodes for deployment

    5) Run these commands on all nodes. This will enable root access with password
    
            (all-nodes)# sudo su
            (all-nodes)# sed -i -e 's/#PermitRootLogin yes/PermitRootLogin yes/g' -e 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config 
            (all-nodes)# service sshd restart
    
    6) Logout & login as root user
    
            (ansible-node)# ssh-keygen –t rsa
            (ansible-node)# ssh-copy-id root@<k8s-master>
            (ansible-node)# ssh-copy-id root@<k8s-node>
             
    7) Populate /root/ansible-labs/k8s/hosts with k8s-master & k8s-node info
    
            (ansible-node)# cat /root/ansible-labs/k8s/hosts
       
               [masters]
               10.10.10.1

               [nodes]
               10.10.10.2
        
    8) Run the play
 
            (ansible-node)# cd /root/ansible-labs/k8s
            (ansible-node)# ansible-playbook -i hosts site.yml
            
## OpenStack Heat

OpenStack [Heat](https://wiki.openstack.org/wiki/Heat) is a service in OpenStack to orchestrate composite cloud applications using a declarative template format through an OpenStack-native REST API.

* Kubernetes with OpenContrail SDN

            (openstack-controller)# git clone https://github.com/savithruml/cloud-ops
            (openstack-controller)# cd cloud-ops/openstack/nested-k8s/provisioning/heat
            (openstack-controller)# vi /root/nested-mode-contrail-networking/provisioning/heat/deploy-nested.env
                           
                  ...
                     parameters:

                     # FQDN of the public network
                     public_network_fqdn: "default-domain:admin:public"

                     # Name of the master instance
                     master_instance_name: "nested-master"

                     # IP address of the master instance
                     master_ip: "<master-instance-IPv4-address>"
                  ...
            
            (openstack-controller)# heat stack-create nested -f deploy-nested.yaml -e deploy-nested.env


#test test 
