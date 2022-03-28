# Deploying-KOps-on-AWS

### KOps

> *Kubernetes Operations or KOps is an open source project used to set up Kubernetes clusters easily and swiftly. **KOps** allows deployment of highly available Kubernetes   clusters on clouds that can be AWS, GCP, Azure etc*

### AWS Cluster Resources

 A KOPS-created Kubernetes cluster leverages the following AWS resource types:
-   IAM Policy and Roles
-   VPC, Subnets, Route Tables and Internet Gateways
-   Security Groups
-   Route53 A Records
-   Elastic Load Balancers (ELBs), Auto Scaling Groups (ASG)
-   Elastic Block Store (EBS) volumes and S3 

### Cluster Creation: required AWS IAM Roles

-   AmazonEC2FullAccess
-   AmazonRoute53FullAccess
-   AmazonS3FullAccess
-   IAMFullAccess
-   AmazonVPCFullAccess

### EC2 Instance Types

There are three types of EC2 instances that get created with an Kubernetes cluster on AWS:

1.  Bastion nodes
2.  Master nodes
3.  Worker nodes

Bastion nodes provide an external facing point of entry into a network containing private network instances. This host can provide a single point of fortification or audit and can be started and stopped to enable or disable inbound SSH communication from the Internet, some call bastion as the “jump server”.

### EC2 Policies

Bastion nodes have the most basic policy, which merely allows them to describe regions for the account.

**Master nodes have the following policies:**

-   full access to EC2
-   full access to Elastic Load Balancing
-   describe, terminate, and capacity change operations against Auto Scaling Groups
-   read and fetch operations for ECR
-   describe for EC2
-   read, notification, and limited update operations for Route 53
-   read and write operations for S3 (limited to the KOPS state store)

Worker nodes have a subset of the master policies:

-   read and fetch operations for ECR
-   describe for EC2
-   read, notification, and limited update operations for Route 53
-   read and write operations for S3 (limited to the KOPS state store)


- [ ] Create AWS IAM Roles and Policy
 - Select type of trusted entity  **AWS service**
 - Choose a use case **EC2**

 ![s1](https://user-images.githubusercontent.com/65442845/117052135-fe393a00-ad34-11eb-853b-7dd18ca9bb68.PNG)

 - Next attach permission by searching in the search bar
 >  #AmazonEC2FullAccess
 >  #AmazonRoute53FullAccess
 >  #AmazonS3FullAccess
 >  #IAMFullAccess
 >  #AmazonVPCFullAccess
 
- Next Add Tag
 
 ![image](https://user-images.githubusercontent.com/65442845/117054949-4017af80-ad38-11eb-8f32-90b8ce50f859.png)

- Review and create Role Name
 ![image](https://user-images.githubusercontent.com/65442845/117055410-bf0ce800-ad38-11eb-8e4f-34313b826bfe.png)

- [ ] Create Route 53 Private/Public Hosted Zone

 - Create Hosted Zone put your domain name, Description, choose type of hosted type like i took private domain, choose your region and VPI ID
 
  ![image](https://user-images.githubusercontent.com/65442845/117056081-891c3380-ad39-11eb-868e-027e7fe78001.png)
  ![image](https://user-images.githubusercontent.com/65442845/117056592-1e1f2c80-ad3a-11eb-977d-ea3c43ddd172.png)
 - Click Create hosted zone

- [ ] Create S3
 - create bucket with Name and Region with noval name with default policy
 
  ![image](https://user-images.githubusercontent.com/65442845/117057393-04321980-ad3b-11eb-8e91-2a981d6e2717.png)
 
 - scroll down and create bucket

- [ ] Launch Bastion Host

 - Launch EC2 instance with the help of any ami type and attach IAM role and security group port 22 for ssh rest use default configuration
 - here i used RHEL - 8 
 
  ![image](https://user-images.githubusercontent.com/65442845/117057558-36437b80-ad3b-11eb-8ef2-b21c0a167ac9.png)
  
 - Don't forget to attach your roles in IAM Role
  ![image](https://user-images.githubusercontent.com/65442845/117058482-4c057080-ad3c-11eb-843a-5f9ce71d8892.png)
 
## Now login to your bastion host and create cluster with following commands
 
 ### Install kops

     curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    
     chmod +x kops-linux-amd64
    
     sudo mv kops-linux-amd64 /usr/local/bin/kops
   
 ### Install kubelet

     curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    
     chmod +x ./kubectl
    
     sudo mv ./kubectl /usr/local/bin/kubectl
     
 ### Add ENV variables to Bashrc file
  `vi ~/.bashrc`
    
       export KOPS_CLUSTER_NAME=sudhanshu.com
       export KOPS_STATE_STORE=s3://sudhanshu.com
    
  `source ~/.bashrc`
 ### Create ssh key pair
  `ssh-keygen`
 
 ### Now it's time to create cluster
     kops create cluster --state=${KOPS_STATE_STORE} --node-count=2  --master-size=t2.medium  --node-size=t2.micro --zones=us-east-1a --name=${KOPS_CLUSTER_NAME} --dns private --master-count 1 --networking flannel --image=ami-096fda3c22c1c990a

     
 ### Update the Cluster
     kops update cluster --name <cluster-name> --yes --admin

 ### Validate Cluster
     kops validate cluster 
                OR
     kops validate cluster --wait 10m

 ### Check Availability of Nodes
     kubectl get nodes     
 ### To connect to the master
     ssh admin@api.<master-name same as ROUTE53-name>
## Destroy the kubernetes cluster
     kops delete cluster  --yes  

  
  
  




