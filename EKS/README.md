### Install the `AWS CLI` : <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html">Docs</a>
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
apt install unzip
unzip awscliv2.zip
sudo ./aws/install
aws --version
```
<img width="708" alt="Screenshot 2022-12-22 at 12 07 08 PM" src="https://user-images.githubusercontent.com/103893307/209091728-43e0232f-fc53-4e32-9b32-e134494fd1b6.png">

### Installing `kubectl` : <a href="https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html">Docs</a>
```
curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.24.7/2022-10-31/bin/linux/amd64/kubectl
```
```
chmod +x ./kubectl
```
```
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
```
```
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
```
```
kubectl version --short --client
```
<img width="1025" alt="Screenshot 2022-12-22 at 1 50 03 PM" src="https://user-images.githubusercontent.com/103893307/209091884-f6031734-bf69-473f-892f-2a75df9ea57b.png">

### Installing `eksctl` : <a href="https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html#installing-eksctl">Docs</a>
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```
```
sudo mv /tmp/eksctl /usr/local/bin
```
```
eksctl version
```
<img width="390" alt="Screenshot 2022-12-22 at 1 52 02 PM" src="https://user-images.githubusercontent.com/103893307/209092048-e91c348d-7a97-47b6-80fe-75a3173ec738.png">

### Configure AWS Command Line using Security Credentials
- Go to AWS Management Console --> Services --> IAM
- Select the IAM User: <your_iam_user_name>
- **Important Note:** Use only IAM user to generate **Security Credentials**. Never ever use Root User. (Highly not recommended)
- Click on **Security credentials** tab
- Click on **Create access key**
- Copy Access ID and Secret access key
- Go to command line and provide the required details

```
aws configure
AWS Access Key ID [None]: XXXXXXXXXXXXXXXXXX  (Replace your creds when prompted)
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXX  (Replace your creds when prompted)
Default region name [None]: us-east-1
Default output format [None]: json
```
- Test if AWS CLI is working after configuring the above
```
aws ec2 describe-vpcs
```
<img width="779" alt="Screenshot 2022-12-22 at 12 44 15 PM" src="https://user-images.githubusercontent.com/103893307/209093391-412dc3bc-f6f4-40b3-a43e-bd78f926d218.png">



### Create EKS Cluster using eksctl

- It will take 15 to 20 minutes to create the Cluster Control Plane 
#### Create Cluster
```
eksctl create cluster --name=eksdemo \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup 
```
<img width="1440" alt="Screenshot 2022-12-23 at 6 29 24 AM" src="https://user-images.githubusercontent.com/103893307/209253022-941ec415-7e68-4f07-9fb7-ef2eead90278.png">


#### Get List of clusters
```
eksctl get cluster --region us-east-1                 
```
<img width="1110" alt="Screenshot 2022-12-23 at 6 30 00 AM" src="https://user-images.githubusercontent.com/103893307/209253091-87c5fa4b-cca4-40a4-8f46-d1a7d24bdd66.png">


### Create & Associate IAM OIDC Provider for our EKS Cluster
- To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create &  associate OIDC identity provider.
- To do so using `eksctl` we can use the  below command. 
- Use latest eksctl version (as on today the latest version is `0.123.0`)              
#### Template
```
eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluter-name> \
    --approve
```
#### Replace with region & cluster name
```
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster eksdemo \
    --approve
```
<img width="1195" alt="Screenshot 2022-12-23 at 6 30 19 AM" src="https://user-images.githubusercontent.com/103893307/209253121-03db232e-8c6a-4200-993e-14df84a91b92.png">

### Create <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-services-ec2-keypairs.html">EC2 Keypair</a>

- Create a new EC2 Keypair with name as `eks-demo`
- This keypair we will use it when creating the EKS NodeGroup.
- This will help us to login to the EKS Worker Nodes using Terminal.
```
aws ec2 create-key-pair --key-name eks-demo --query 'KeyMaterial' --output text > eks-demo.pem
```
```
chmod 400 eks-demo.pem
```
```
aws ec2 describe-key-pairs --key-name eks-demo
```
### Create Node Group with additional Add-Ons in Public Subnets
- These add-ons will create the respective IAM policies for us automatically within our Node Group role.
# Create Public Node Group   
```
eksctl create nodegroup --cluster=eksdemo \
                        --region=us-east-1 \
                        --name=eksdemo-ng-public1 \
                        --node-type=t3.medium \
                        --nodes=2 \
                        --nodes-min=2 \
                        --nodes-max=4 \
                        --node-volume-size=20 \
                        --ssh-access \
                        --ssh-public-key=eks-demo \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access 
```
<img width="1440" alt="Screenshot 2022-12-23 at 6 52 24 AM" src="https://user-images.githubusercontent.com/103893307/209253162-1121fac4-6125-42fe-ac7f-70dcb3cbb8ea.png">


### Verify Cluster & Nodes

#### Verify Cluster, NodeGroup in EKS Management Console
- Go to Services -> Elastic Kubernetes Service -> eksdemo

### List Worker Nodes
```
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=<clusterName> --region us-east-1

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide
```
<img width="1433" alt="Screenshot 2022-12-23 at 6 53 24 AM" src="https://user-images.githubusercontent.com/103893307/209253474-fb6c4a69-d2ac-46f1-a12d-843b40399911.png">

### Verify Worker Node IAM Role and list of Policies
- Go to Services -> EC2 -> Worker Nodes
- Click on **IAM Role associated to EC2 Worker Nodes**

### Verify Security Group Associated to Worker Nodes
- Go to Services -> EC2 -> Worker Nodes
- Click on **Security Group** associated to EC2 Instance which contains `remote` in the name.

### Verify CloudFormation Stacks
- Verify Control Plane Stack & Events
- Verify NodeGroup Stack & Events

### Login to Worker Node using Key eks-demo
- Login to worker node
```
# For Linux
ssh -i eks-demo.pem ec2-user@<Public-IP-of-Worker-Node>

# For Windows 
Use putty
```
<img width="1440" alt="Screenshot 2022-12-22 at 2 53 44 PM" src="https://user-images.githubusercontent.com/103893307/209112997-d9e01ed7-f9c0-4411-bc7f-c1497e2a0bd6.png">


### Testing 
```
kubectl run nginx --image nginx
kubectl expose po nginx --port 80 --name nginx-svc --type NodePort
```

# Delete EKS Cluster & Node Groups

## Delete Node Group
- We can delete a nodegroup separately using below `eksctl delete nodegroup`
```
# List EKS Clusters
eksctl get clusters

# Capture Node Group name
eksctl get nodegroup --cluster=<clusterName>
eksctl get nodegroup --cluster=eksdemo

# Delete Node Group
eksctl delete nodegroup --cluster=<clusterName> --name=<nodegroupName>
eksctl delete nodegroup --cluster=eksdemo --name=eksdemo-ng-public1
```

## Delete Cluster  
- We can delete cluster using `eksctl delete cluster`
```
# Delete Cluster
eksctl delete cluster <clusterName>
eksctl delete cluster eksdemo
```

### Note : Rollback any Security Group Changes
- When we create a EKS cluster using `eksctl` it creates the worker node security group with only port 22 access.
- When we progress through the course, we will be creating many **NodePort Services** to access and test our applications via browser. 
- During this process, we need to add an additional rule to this automatically created security group, allowing access to our applications we have deployed. 
- So the point we need to understand here is when we are deleting the cluster using `eksctl`, its core components should be in same state which means roll back the change we have done to security group before deleting the cluster.
- In this way, cluster will get deleted without any issues, else we might have issues and we need to refer cloudformation events and manually delete few things. In short, we need to go to many places for deletions. 
