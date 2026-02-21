Ingress Controller
OIDC provider

trust and authentication provider

REGION_CODE=us-east-1 CLUSTER_NAME=roboshop ACC_ID=816817860311

eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve

Download IAM Policy

curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.13.4/docs/install/iam_policy.json

Create IAM Policy

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

Create IAM Role and K8 ServiceAccount.

Mapping between IAM ROle and K8 Service account

eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve

This is for secret reader

eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=roboshop \
--name=roboshop-mysql-secret-reader \
--attach-policy-arn=arn:aws:iam::816817860311:policy/RoboshopMYSQLSECRETReader \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve

after exec you can enter this
aws secretsmanager get-secret-value --secret-id roboshop/mysql/password

if you want to create service account through yaml you can use this command
kubectl get sa roboshop-mysql-secret-reader -o yaml

if you want to see only password
aws secretsmanager get-secret-value \
  --secret-id roboshop/mysql/password \
  --query 'SecretString' \
  --output text


Install Load balancer controller drivers

helm repo add eks https://aws.github.io/eks-charts

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller

