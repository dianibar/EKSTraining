
<h3 align="center">EKS Training</h3>

<!-- GETTING STARTED -->

## Push and Pull and Image to ECR

### Create the repository
1. Open the Amazon ECR console at https://console.aws.amazon.com/ecr/.
2. Choose Get Started.
3. For Visibility settings, choose Private.
4. For Repository name, specify a name for the repository hello_app.
5. For Tag immutability, choose the tag mutability setting for the repository.

### push the image
1. Create a Folder hello_app
2. Inside the folder create a python script hello_world.py with the following content
```
print("Hello World")
```    
3. In the Same Folder create a dockerfile with the following content
```
# Use an official Python runtime as a parent image
FROM python:3.8

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app

COPY ./hello_world.py /app

# Set the entry point to run the scripts
ENTRYPOINT ["python", "hello_world.py"]
```
4. Build the image 
docker build -t my-hello-world .
5. Select the repository and click the button view push commands
* Authenticate with the repository
   
 ```
 aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin <account_number>.dkr.ecr.<region>.amazonaws.com
 ```  
   * Tag the image
```
docker tag my-hello-world <account_number>.dkr.ecr.<region>.amazonaws.com/hello_app:latest

```

   * Push the image
```
docker tag my-hello-world <account_number>.dkr.ecr.<region>.amazonaws.com/hello_app:latest

```

## Creating and setting up the Cloud9 environment

1. Create the EC3 instance profile role:

* Open the IAM console.
* Choose Roles.
* Choose Create role
* Choose AWS Service and then choose EC2 from the list.
* Choose Next: Permissions and search and check the IAM policy AdministratorAccess
* Check the policy AWSCloud9SSMInstanceProfile


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


## Create an EKS cluster 

1. Clone this project and move to the Terraform folder to create the cluster

```
git clone https://github.com/dianibar/EKSTraining.git

cd EKSTraining/terraform/
```

2. Install Terraform using the following commands:

```
wget https://releases.hashicorp.com/terraform/1.7.4/terraform_1.7.4_linux_amd64.zip

unzip terraform_1.7.4_linux_amd64.zip

sudo mv terraform /usr/local/bin
```

3. Create the EKS cluster using terraform

```
terraform init

terraform apply
```

4. Install kubectl

```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.29.0/2024-01-04/bin/linux/amd64/kubectl

chmod +x ./kubectl

kubectl version --client
```

5. Update kubeconfig

```
aws eks update-kubeconfig --region ap-southeast-2 --name my-eks
```

## Deploy an application in an EKS cluster

We will create a application following the steps provided in this link [Deploy a sample application](https://docs.aws.amazon.com/eks/latest/userguide/sample-deployment.html) 

1. create a namespace

```
kubectl create namespace eks-sample-app
```

2. Create a deployment
*     Create a file **eks-sample-deployment.yaml** with the following content
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eks-sample-linux-deployment
  namespace: eks-sample-app
  labels:
    app: eks-sample-linux-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: eks-sample-linux-app
  template:
    metadata:
      labels:
        app: eks-sample-linux-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values:
                - amd64
                - arm64
      containers:
      - name: nginx
        image: public.ecr.aws/nginx/nginx:1.23
        ports:
        - name: http
          containerPort: 80
        imagePullPolicy: IfNotPresent
      nodeSelector:
        kubernetes.io/os: linux
```

3. Deploy the application

```
kubectl apply -f eks-sample-deployment.yaml
```

4. Exec in the created container and check nginx running

```
kubectl exec -n eks-sample-app -it eks-sample-linux-deployment-xxx.. -- sh 
curl localhost
```

5. Create a service to access the application from outside the cluster for that create a file called eks-sample-service.yaml with the following 

```
apiVersion: v1
kind: Service
metadata:
  name: eks-sample-linux-service
  namespace: eks-sample-app
  labels:
    app: eks-sample-linux-app
spec:
  selector:
    app: eks-sample-linux-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

6. Deploy the service

```
kubectl apply -f eks-sample-service.yaml
```

7. Check all the resources in the namespace

```
kubectl get all -n eks-sample-app
```

8. Create a file eks-sample-service.yaml with the following content

```
apiVersion: v1
kind: Service
metadata:
  name: eks-sample-linux-service
  namespace: eks-sample-app
  labels:
    app: eks-sample-linux-app
spec:
  selector:
    app: eks-sample-linux-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
