# Setup K8 cluster using EC2 Instance using KOPS (gossip Cluster - Will not be using Route53)

Assuming you have following binaries installed in your machine
- awscli (pip install awscli)
- kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- kops (https://github.com/kubernetes/kops)

Create a IAM user and grant Administrator Privilieges, make sure you create programatic access also
- Download the creds file this will be used later

Run aws configure and insert Access & Secret Keys

Create a S3 bucket to store the state of cluster (I am using EU Region)
```
aws s3api create-bucket --bucket sn-dev-kops-state-store --region eu-west-1 --create-bucket-configuration LocationConstraint=eu-west-1
```
Export the Cluster name and State
```
export KOPS_CLUSTER_NAME=sn-dev.k8s.local
export KOPS_STATE_STORE=s3://sn-dev-kops-state-store
```
Set Variables for Master Node
```
Master_Count="1"
Master_Volume_Size="50"
Master_Instance_Type="t2.medium"
Master_Zones="eu-west-1a"
```
Set Variables for Nodes
```
Node_Count="1"
Node_Size="t2.medium"
Node_Volume="20"
Node_Zones="eu-west-1a,eu-west-1b,eu-west-1c"
```
Set Kubernetes Version
```
Kubernetes_version="1.9.9"
```
Set SSH Access for login to all nodes, you can generate one using : ssh-keygen -t rsa
```
SSH_Public_Key=<path to pub key file>
```
Create a Cluster with kops, using private topology once cluster is created will create bastion for accessing the nodes
```
kops create cluster \
--zones=$Node_Zones \
--name=${KOPS_CLUSTER_NAME} \
--node-count=$Node_Count \
--node-size=$Node_Size \
--node-volume-size=$Node_Volume \
--master-size=$Master_Instance_Type \
--master-count=$Master_Count \
--master-volume-size=$Master_Volume_Size \
--master-zones=$Master_Zones \
--ssh-public-key=$SSH_Public_Key \
--kubernetes-version=$Kubernetes_version \
--authorization=alwaysAllow \
--cloud=aws \
--associate-public-ip=false \
--topology=private \
--networking=flannel
```
Deploy the Cluster
```
kops update cluster --name ${KOPS_CLUSTER_NAME} --yes
```
Let create an bastion instance group for remote managment
```
kops create instancegroup bastions --role Bastion --subnet utility-us-east-1a --name ${KOPS_CLUSTER_NAME}
```

Lets update the Cluster so it will create all above pools
```
kops update cluster --name ${KOPS_CLUSTER_NAME} --yes
```
## Take a break for tea/coffee and once your back verify your cluster would be up and running
```
kops validate cluster"
```

##### Once you have created a S3 bucket then rest all stuffs you can put in a file and run as shell script

##### One Cluster is created and you get the result of kops validate cluster, then you can login to your bastion host run the following command to find the ELB DNS entry for login
```
aws elb describe-load-balancers
... output 
"DNSName": "bastion-sn-dev-k8s-loc-8q1c0n-560521935.eu-west-1.elb.amazonaws.com",
...output 
```

