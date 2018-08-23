# Setup K8 cluster using EC2 Instance using KOPS (gossip Cluster - Will not be using Route53)

--- Assuming you have following binaries installed in your machine
- awscli (pip install awscli)
- kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- kops (https://github.com/kubernetes/kops)

--- Create a IAM user and grant Administrator Privilieges, make sure you create programatic access also
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
