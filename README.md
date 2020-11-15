# EKSCluster
EKS Cluster using Kops command

Prerequiste:
1. Docker installation
2. Kubectl Installation
3. aws configure

I. Docker installation in EC2:
===============================
1. Update the installed packages and package cache: <br/>
sudo yum update -y

2. Install the most recent Docker Community Edition package.<br/>
Amazon Linux 2: <br/>
sudo amazon-linux-extras install docker<br/>
Amazon Linux.<br/>
sudo yum install docker

3. Start the Docker service.<br/>
sudo service docker start

4. Add the ec2-user to the docker group to execute Docker commands without using sudo.<br/>
sudo usermod -a -G docker ec2-user

5. Log out and log back in again to pick up the new docker group permissions. or accomplish this by closing current SSH terminal window and reconnecting to instance in a new one. SSH session will be reflected with appropriate docker group permissions.

6. Verify that the ec2-user can run Docker commands without sudo.<br/>
docker info

# Create a Docker image with a simple web application
1. touch Dockerfile 
2. Edit Dockerfile (update as in Dockerfile in repo)
3. To build Docker image from Dockerfile. <br/>
docker build -t hello-world .
4. Run docker images to verify image was created successfully<br/>
docker images --filter reference=hello-world
5. Run the newly built image:<br/>
docker run -t -i -p 80:80 hello-world

# To tag the image and push it into Amazon ECR
1. Create an Amazon ECR repository to store hello-world image.<br/>
aws ecr create-repository --repository-name hello-repository --region region

2. Tag hello-world image<br/>
docker tag hello-world aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository

3. Run aws ecr get-login-password command.<br/>
aws ecr get-login-password | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com

4. Push the image to Amazon ECR with repositoryUri<br/>
docker push aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository

Cleanup:
aws ecr delete-repository --repository-name hello-repository --region region --force

II. Kubectl installation in Amazon Linux 2:
============================================
1. Install Kubectl: (Kubernetes 1.18)<br/>
curl -o kubectl https://amazon-eks.s3.cn-north-1.amazonaws.com.cn/1.18.9/2020-11-02/bin/linux/amd64/kubectl

2. Apply execute permissions to binary:<br/>
chmod +x ./kubectl

3. copy binary to a folder in PATH.If already installed a version of kubectl, creating a $HOME/bin/kubectl and ensuring that $HOME/bin comes first in your $PATH.<br/>
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

4. Add the $HOME/bin path to your shell initialization file so that it is configured when you open a shell.<br/>
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc

5. Verify kubectl version<br/>
kubectl version --short --client

# To create a Kubernetes cluster on AWS (EKS) using Kubernetes Operations(kops):
==============================================================================
1. Install kops IN LINUX:<br/>
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

2. Change permission:<br/>
chmod +x kops-linux-amd64

3. Move kops to bin folder<br/>
sudo mv kops-linux-amd64 /usr/local/bin/kops

4. Create a S3 bucket with versioning (To persist its state)
a. aws s3api create-bucket --bucket imesh-kops-state-store --region us-east-1
b. aws s3api put-bucket-versioning --bucket raman-kops-state-store --versioning-configuration Status=Enabled

5. Create a IAM user with following permissions:<br/>
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
AmazonVPCFullAccess

6. Edit ~/.bash_profile: <br/>
export KOPS_CLUSTER_NAME=raman.k8s.local
export KOPS_STATE_STORE=s3://raman-kops-state-store
export AWS_ACCESS_KEY=AKIA3GNTJITY5I23LR6P
export AWS_SECRET_KEY=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

7. Create a cluster: <br/>
kops create cluster \
--node-count=2 \
--node-size=t2.medium \
--zones=us-east-1a \
--name=${KOPS_CLUSTER_NAME}

8. Review the Kubernetes cluster definition by executing the below command:<br/>
kops edit cluster --name ${KOPS_CLUSTER_NAME}

9. Kubernetes cluster on AWS by executing kops update command:<br/>
kops update cluster --name ${KOPS_CLUSTER_NAME} --yes

10. command to check its status and wait until the cluster becomes ready:<br/>
kops validate cluster

11. To deploy the Kubernetes dashboard to access the cluster via its web based user interface: <br/>
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.4/aio/deploy/recommended.yaml

12. to find the admin user’s password: <br/>
kops get secrets kube --type secret -oplaintext

13. to find the Kubernetes master hostname: <br/>
kubectl cluster-info

14. Access the Kubernetes dashboard using the following URL: <br/>
https://<kubernetes-master-hostname>/ui

Note: the secret name used here is different from the previous one:
kops get secrets admin --type secret -oplaintext


kops delete cluster --name=${KOPS_CLUSTER_NAME}
