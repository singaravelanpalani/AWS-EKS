# EKS Storage with EBS - Elastic Block Store

## Step-01: Introduction
- Create IAM Policy for EBS
- Associate IAM Policy to Worker Node IAM Role
- Install EBS CSI Driver

## Step-02:  Create IAM policy JSON File

```
# Create the Policy JSON file
cat <<EOF > Amazon_EBS_CSI_Driver.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:AttachVolume",
        "ec2:CreateSnapshot",
        "ec2:CreateTags",
        "ec2:CreateVolume",
        "ec2:DeleteSnapshot",
        "ec2:DeleteTags",
        "ec2:DeleteVolume",
        "ec2:DescribeInstances",
        "ec2:DescribeSnapshots",
        "ec2:DescribeTags",
        "ec2:DescribeVolumes",
        "ec2:DetachVolume"
      ],
      "Resource": "*"
    }
  ]
}
EOF

```

## Step-03: Get the IAM role Worker Nodes using and Associate this Inline policy to that role
```
# Get Worker node IAM Role ARN
kubectl -n kube-system describe configmap aws-auth

# from output check rolearn
arn:aws:iam::054760367290:role/eksctl-etechbrain-dev-nodegroup-e-NodeInstanceRole-QQ2C0GV41KNH

# Add the IAM Policy to Above Role
aws iam put-role-policy --role-name eksctl-etechbrain-dev-nodegroup-e-NodeInstanceRole-QQ2C0GV41KNH \
                        --policy-name Amazon_EBS_CSI_Driver \
                        --policy-document "file://Amazon_EBS_CSI_Driver.json"
```


## Step-04: Deploy Amazon EBS CSI Driver  
- Verify kubectl version, it should be 1.14 or later
```
kubectl version --client --short
```
- Deploy Amazon EBS CSI Driver
```
# Deploy EBS CSI Driver
kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"

# Verify ebs-csi pods running
kubectl get pods -n kube-system
```
