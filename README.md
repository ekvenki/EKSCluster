# EKSCluster
EKS Cluster using Kops command

Docker installation in EC2:
===========================
1. Update the installed packages and package cache:
sudo yum update -y

2. Install the most recent Docker Community Edition package.
Amazon Linux 2:
sudo amazon-linux-extras install docker

Amazon Linux.
sudo yum install docker

3. Start the Docker service.
sudo service docker start

4. Add the ec2-user to the docker group to execute Docker commands without using sudo.
sudo usermod -a -G docker ec2-user

5. Log out and log back in again to pick up the new docker group permissions. You can accomplish this by closing your current SSH terminal window and reconnecting to your instance in a new one. Your new SSH session will have the appropriate docker group permissions.

6. Verify that the ec2-user can run Docker commands without sudo.
docker info

# Create a Docker image with a simple web application
1. touch Dockerfile
2. Edit Dockerfile (update as in Dockerfile in repo)
3. To build Docker image from Dockerfile
docker build -t hello-world .

4. Run docker images to verify image was created successfully
docker images --filter reference=hello-world

5. Run the newly built image:
docker run -t -i -p 80:80 hello-world

# To tag the image and push it into Amazon ECR
1. Create an Amazon ECR repository to store hello-world image.
aws ecr create-repository --repository-name hello-repository --region region

2. Tag hello-world image
docker tag hello-world aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository

3. Run aws ecr get-login-password command.
aws ecr get-login-password | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com

4. Push the image to Amazon ECR with repositoryUri
docker push aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository

Cleanup:
aws ecr delete-repository --repository-name hello-repository --region region --force

# Kubectl installation in Amazon Linux 2:
1. Install Kubectl: (Kubernetes 1.18)
curl -o kubectl https://amazon-eks.s3.cn-north-1.amazonaws.com.cn/1.18.9/2020-11-02/bin/linux/amd64/kubectl

2. Apply execute permissions to binary:
chmod +x ./kubectl

3. copy binary to a folder in PATH.If already installed a version of kubectl, creating a $HOME/bin/kubectl and ensuring that $HOME/bin comes first in your $PATH.
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin

4. Add the $HOME/bin path to your shell initialization file so that it is configured when you open a shell.
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc

5. Verify kubectl version
kubectl version --short --client
