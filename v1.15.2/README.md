# -------------- HCP Consul Installation Guide ------------------- #

1. Create EKS cluster

eksctl create cluster \
--name cluster-1 \
--region eu-central-1 \
--nodegroup-name ng \
--node-type t3.medium \
--nodes 3 \
--nodes-min 3 \
--nodes-max 4 \
--vpc-cidr 10.0.0.0/16 \
--vpc-nat-mode Disable

2. Install CSI plugin

https://catalog.workshops.aws/eks-immersionday/en-US/pesistentstorage-ebs/ebs-csi-driver

export CLUSTER="cluster-1"
export ACCOUNT_ID="775964700675"

a. Create OpenID Connect (OIDC) Provider

eksctl utils associate-iam-oidc-provider \
  --region=eu-central-1 \
  --cluster=$CLUSTER \
  --approve


b. Configure IAM Role for Service Account
    You can associate an IAM role with a Kubernetes service account. This service account can then provide AWS permissions to the containers in any pod that uses that service account

    aws iam create-policy \
  --policy-name AmazonEBSCSIDriverPolicy \
  --policy-document file://AmazonEBSCSIDriverPolicy.json

      (AmazonEBSCSIDriverPolicy: https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/example-iam-policy.json)


    # Create k8s service account and create IAM role and attached it with the policy
    eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster $CLUSTER \
    --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AmazonEBSCSIDriverPolicy \
    --approve \
    --role-only \
    --role-name AmazonEKS_EBS_CSI_DriverRole

    eksctl get iamserviceaccount --cluster=${CLUSTER}
    k get sa -n kube-system | grep ebs-csi


c. Add EBS CSI add-on

eksctl create addon --name aws-ebs-csi-driver --cluster $CLUSTER \
    --service-account-role-arn arn:aws:iam::${ACCOUNT_ID}:role/AmazonEKS_EBS_CSI_DriverRole --force

k get pods -n kube-system | grep ebs-csi
eksctl get addon --name aws-ebs-csi-driver --cluster $CLUSTER


d. Define Storage Class
Dynamic Volume Provisioning  allows storage volumes to be created on-demand

k create -f mysql-storageclass.yaml


3. Install ALB controller

a.
export CLUSTER="test-cluster-1"
export ACCOUNT_ID="269453396250"

eksctl utils associate-iam-oidc-provider \
  --region=eu-central-1 \
  --cluster=$CLUSTER \
  --approve

b.
aws iam create-policy \
      --policy-name AWSLoadBalancerControllerIAMPolicy \
      --policy-document file://AWSLoadBalancerControllerIAMPolicy.json

eksctl create iamserviceaccount \
--cluster=${CLUSTER} \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region=eu-central-1 \
--approve

k get sa -n kube-system | grep aws-load-balancer

c. 
helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=${CLUSTER} --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller

k get pods -n kube-system | grep aws-load-balancer


4. Install Consul Helm charts

k create namespace consul

k create secret generic "consul-bootstrap-token" --from-literal="token=dd6c27b6-6e06-8762-11d8-da86dba7a96c" --namespace consul

k create secret generic consul-ca-cert --from-file="caCert=ca.pem" --namespace consul <-- Not Required 

k apply --kustomize="github.com/hashicorp/consul-api-gateway/config/crd?ref=v0.5.3"

helm repo add hashicorp https://helm.releases.hashicorp.com && helm repo update

helm install consul hashicorp/consul --version 1.1.1 -f values.yaml --create-namespace --namespace consul
or
helm upgrade consul hashicorp/consul -n consul -f values.yaml --namespace consul --install 

kubectl apply -f api-gateway.yaml && \
    k wait --for=condition=ready gateway/api-gateway --timeout=90s && \
    k apply -f httproute.yaml

5. Deploy demo app
k apply -f apps/hashicups/




# -----------------------------------------------------------------------

k run -it network-multitool --image=praqma/network-multitool -- /bin/bash

HCP-cluster token : d019e214-7df7-024a-282a-286c9f4ce029