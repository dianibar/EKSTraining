
<h3 align="center">EKS Training</h3>


<!-- GETTING STARTED -->
## Creating and setting up the Cloud9 environment

1. Create the IAM policy for the EC2 instance profile role:

The following instructions are provided in this [link](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/deploy-an-amazon-eks-cluster-from-aws-cloud9-using-an-ec2-instance-profile.html#deploy-an-amazon-eks-cluster-from-aws-cloud9-using-an-ec2-instance-profile-epics) 
* Open the IAM console.
* Choose Policies. 
* Create policy. 
* Choose the JSON tab and paste the following content

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "iam:GetRole",
                "iam:PassRole",
                "logs:CreateLogStream",
                "ec2:DeleteTags",
                "cloudformation:CreateStack",
                "ec2:CreateTags",
                "logs:DescribeLogStreams",
                "cloudformation:DeleteStack",
                "cloudformation:CreateChangeSet",
                "cloudformation:DescribeStacks"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:subnet/*",
                "arn:aws:ec2:*:*:vpc/*",
                "arn:aws:cloudformation:*:*:stack/*/*",
                "arn:aws:iam::*:role/*",
                "arn:aws:logs:*:*:log-group:/aws/eks/*:*"
            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "iam:AWSServiceName": "elasticloadbalancing.amazonaws.com"
                }
            }
        },
        {
            "Sid": "VisualEditor2",
            "Effect": "Allow",
            "Action": "iam:CreateServiceLinkedRole",
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "iam:AWSServiceName": "eks.amazonaws.com"
                }
            }
        },
        {
            "Sid": "VisualEditor3",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:ModifyListener",
                "elasticloadbalancing:DetachLoadBalancerFromSubnets",
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:DescribeInstances",
                "elasticloadbalancing:RegisterTargets",
                "ec2:CreateKeyPair",
                "ec2:DescribeVolumesModifications",
                "elasticloadbalancing:DeleteLoadBalancer",
                "elasticloadbalancing:DescribeLoadBalancers",
                "ec2:DeleteVolume",
                "ec2:CreateNetworkInterfacePermission",
                "autoscaling:DescribeAutoScalingGroups",
                "ec2:CreateRoute",
                "iam:ListAttachedRolePolicies",
                "ec2:DescribeVolumes",
                "elasticloadbalancing:DescribeLoadBalancerPolicies",
                "autoscaling:UpdateAutoScalingGroup",
                "ec2:DescribeKeyPairs",
                "elasticloadbalancing:ModifyTargetGroupAttributes",
                "elasticloadbalancing:DeregisterInstancesFromLoadBalancer",
                "elasticloadbalancing:RegisterInstancesWithLoadBalancer",
                "ec2:DescribeRouteTables",
                "ec2:DetachVolume",
                "ec2:ModifyVolume",
                "elasticloadbalancing:SetLoadBalancerPoliciesForBackendServer",
                "ec2:CreateTags",
                "elasticloadbalancing:CreateTargetGroup",
                "ec2:ModifyNetworkInterfaceAttribute",
                "ec2:DeleteNetworkInterface",
                "elasticloadbalancing:DeregisterTargets",
                "logs:CreateLogGroup",
                "ec2:CreateVolume",
                "elasticloadbalancing:DescribeLoadBalancerAttributes",
                "ec2:CreateNetworkInterface",
                "ec2:RevokeSecurityGroupIngress",
                "elasticloadbalancing:DescribeTargetGroupAttributes",
                "elasticloadbalancing:AddTags",
                "elasticloadbalancing:DeleteLoadBalancerListeners",
                "ec2:DescribeSubnets",
                "elasticloadbalancing:ModifyLoadBalancerAttributes",
                "cloudformation:ValidateTemplate",
                "ec2:AttachVolume",
                "eks:UpdateClusterVersion",
                "elasticloadbalancing:ConfigureHealthCheck",
                "ec2:DescribeDhcpOptions",
                "elasticloadbalancing:CreateListener",
                "elasticloadbalancing:DescribeListeners",
                "ec2:DescribeNetworkInterfaces",
                "ec2:CreateSecurityGroup",
                "elasticloadbalancing:ApplySecurityGroupsToLoadBalancer",
                "kms:DescribeKey",
                "ec2:ModifyInstanceAttribute",
                "elasticloadbalancing:CreateLoadBalancerPolicy",
                "elasticloadbalancing:CreateLoadBalancer",
                "elasticloadbalancing:AttachLoadBalancerToSubnets",
                "ec2:DeleteRoute",
                "elasticloadbalancing:DeleteTargetGroup",
                "elasticloadbalancing:CreateLoadBalancerListeners",
                "ec2:DescribeSecurityGroups",
                "elasticloadbalancing:SetLoadBalancerPoliciesOfListener",
                "ec2:DescribeVpcs",
                "ec2:DeleteSecurityGroup",
                "elasticloadbalancing:DescribeTargetHealth",
                "elasticloadbalancing:DescribeTargetGroups",
                "route53:AssociateVPCWithHostedZone",
                "elasticloadbalancing:ModifyTargetGroup",
                "elasticloadbalancing:DeleteListener"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor4",
            "Effect": "Allow",
            "Action": [
                "ssm:GetParameters",
                "logs:PutLogEvents",
                "ssm:GetParameter"
            ],
            "Resource": [
                "arn:aws:logs:*:*:log-group:/aws/eks/*:*:*",
                "arn:aws:ssm:*:*:parameter/*"
            ]
        },
        {
            "Sid": "VisualEditor5",
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": [
                "arn:aws:eks:*:*:cluster/*",
                "arn:aws:eks:*:*:nodegroup/*/*/*"
            ]
        }
    ]
}
```

* Choose Review policy.
* Enter a Name for the policy like eks-instance-profile-for-cloud9
* Select Create Policy

2. Create the EC3 instance profile role:

* Open the IAM console.
* Choose Roles.
* Choose Create role
* Choose AWS Service and then choose EC2 from the list.
* Choose Next: Permissions and search for the IAM policy that you created earlier.
* In the Review section, enter a name for the role like role-eks-instance-profile-for-cloud9
* Then choose create role.

3. Create IAM policy for Amazon EKS RBAC role

* Open the IAM console.
* Choose Policies. 
* Create policy. 
* Choose the JSON tab and paste the following content

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:ListClusters",
                "eks:DescribeCluster"
            ],
            "Resource": "*"
        }
    ]
}
```
* Choose Review policy.
* Enter a Name for the policy like policy-for-eks-rbac
* Select Create Policy

4. Create an IAM role for the Amazon EKS RBAC

* Open the IAM console.
* Choose Roles.
* Choose Create role
* Choose AWS Service and then choose EC2 from the list.
* Choose Next: Permissions and search for the IAM policy that you created earlier.
* In the Review section, enter a name for the role like role-eks-admin-for-rbac.
* Then choose create role.

5. Create a Cloud9 environment 

* Follow the instructions to create the Cloud9 environment, available in this [link](https://docs.aws.amazon.com/cloud9/latest/user-guide/create-environment-main.html)
* Remove the temporary IAM credentials for AWS Cloud9. 
   * choose Settings in the gear icon. 
   * Under Preferences, choose AWS settings and then choose Credentials. 
   * Turn off AWS managed temporary credentials and close the tab.
* Attach the EC2 instance profile to the underlying EC2 instance. 
   * Open the Amazon EC2 console and choose the EC2 instance that matches your environment in AWS Cloud9. 
   * If you used the name that we recommended, the EC2 instance is called aws-cloud9-eks-management-env.
   * Choose the EC2 instance, choose Actions, and then choose Instance settings. 
   * Choose Attach/replace IAM role. Search for role-eks-instance-profile-for-cloud9 or the name of the IAM role that you created earlier, and then choose Apply.

* Install terraform running the following commands:
```
wget https://releases.hashicorp.com/terraform/1.7.5/terraform_1.7.5_linux_amd64.zip
unzip terraform_1.7.5_linux_amd64.zip
sudo mv terraform /usr/local/bin
```

## Getting familiar with docker and Dockerfile

For this part of the lab we are going to use this [AWS lab](https://catalog.us-east-1.prod.workshops.aws/workshops/ed1a8610-c721-43be-b8e7-0f300f74684e/en-US/contdock/dockerbasics)


### Create an EKS cluster 

1. Clone this project and move to the Terraform folder to create the cluster

```
git clone https://github.com/dianibar/EKSTraining.git

cd EKSTrainig/terraform


```
### Create a Docker image using an image base that supports ARM and push it to ECR

1. In the folder graviton-poc/ruby-on-rails there is a helloworld rubby application. In the folder, there is also a Docker file using ruby:latest as the base image. Checking in [DockerHub](https://hub.docker.com/_/ruby) we can see that this image supports ARM architecture.

2. ECR supports multi-architecture container images. To be able to create an image that is built for different architectures you can use an emulator. For this follow the instructions provided in this [AWS blog](https://aws.amazon.com/blogs/compute/how-to-quickly-setup-an-experimental-environment-to-run-containers-on-x86-and-aws-graviton2-based-amazon-ec2-instances-effort-to-port-a-container-based-application-from-x86-to-graviton2/) we can follow these steps:
   
   * Install buildx

   ```
   curl --silent -L https://github.com/docker/buildx/releases/download/v0.13.1/buildx-v0.13.1.linux-amd64 -o buildx-v0.13.1.linux-amd64

   chmod a+x buildx-v0.13.1.linux-amd64

   mkdir -p ~/.docker/cli-plugins
   
   mv buildx-v0.13.1.linux-amd64 ~/.docker/cli-plugins/docker-buildx
   
   docker buildx
   ```
   * Enter the following command to configure Buildx binary for different architectures

   ```
   docker run --privileged --rm tonistiigi/binfmt --install all
   ``` 
   * Check to see a list of build environment.
   ```
   docker buildx ls
   ```
   * Create a new builder named mybuild and switch to it to use it as default.
3. Create a multi-arch image for x*6 and Arm64 and push them to Amazon ECR
   * Set the environment variables
   ```
   AWS_ACCOUNT_ID=aws-account-id
   AWS_REGION=us-west-2
   ```
   * Authenticate your Docker client to your Amazon ECR registry

   ```
   login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
   ```
   * Create your multi-archi image and push it to ECR
   ```
   docker buildx build --platform linux/amd64,linux/arm64 --tag ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/myrepo:latest --push .
   ```
### Run the application on EKS

1. The folder graviton-poc/ruby-on-rails/kubernetes has a file with a service an a deployment. The deployment is configured to spread to multiple nodes. The eks cluster has been configured to have one node using t3.micro and another node using t4g.micro so it will show that the app runs in x86 and ARM.

```
cd graviton-poc/ruby-on-rails/kubernetes
kubectl apply 
```