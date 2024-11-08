# Creating Amazon EC2 Instances
#### Lab overview
AWS provides multiple ways to launch Amazon Elastic Compute Cloud (Amazon EC2) instance. 
In this lab, you use the AWS Management Console to launch an EC2 instance and then use it as a bastion host to launch another EC2 instance, which will be a web server. You use EC2 Instance Connect to securely connect to the bastion host and use the AWS Command Line Interface (AWS CLI) to launch a web server instance.
The following diagram illustrates the final architecture that you will build:

![צילום מסך 2024-11-08 141023](https://github.com/user-attachments/assets/ba858e00-d5bf-4209-99ad-4877357d4714)


#### Objectives
After completing this lab, you should be able to do the following:
* Launch an EC2 instance by using the AWS Management Console.
* Connect to the EC2 instance by using EC2 Instance Connect.
* Launch an EC2 instance by using the AWS CLI.

## Task 3: Launching an EC2 Instance Using the AWS CLI

This task involves launching an EC2 instance using the AWS CLI, which allows for automated provisioning and configuration of AWS resources.
The steps include:

1. *Retrieve the AMI*: Use the AWS Systems Manager Parameter Store to get the ID of the latest Amazon Linux 2 AMI.
Command:
AZ=`curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone`
export AWS_DEFAULT_REGION=${AZ::-1}
AMI=$(aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 --query 'Parameters[0].[Value]' --output text)
echo $AMI

2. *Retrieve the Subnet*: Get the subnet ID for the public subnet using the describe-subnets command.
Command:
SUBNET=$(aws ec2 describe-subnets --filters 'Name=tag:Name,Values=Public Subnet' --query Subnets[].SubnetId --output text)
echo $SUBNET

3. *Retrieve the Security Group*: Get the security group ID for the web security group that allows HTTP traffic.
Command:
SG=$(aws ec2 describe-security-groups --filters Name=group-name,Values=WebSecurityGroup --query SecurityGroups[].GroupId --output text)
echo $SG

4. *Download the User Data Script*: Download a script that installs and configures the Apache web server and web application.
Command:
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-3-23732/171-lab-JAWS-create-ec2/s3/UserData.txt
cat UserData.txt

5. *Launch the Instance*: Use the run-instances command to launch the instance with the specified AMI, subnet, security group, and user data script.
Command:
INSTANCE=$(
  aws ec2 run-instances \
  --image-id $AMI \
  --subnet-id $SUBNET \
  --security-group-ids $SG \
  --user-data file:///home/ec2-user/UserData.txt \
  --instance-type t3.micro \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=Web Server}]' \
  --query 'Instances[*].InstanceId' \
  --output text \
)
echo $INSTANCE

6. *Wait for the Instance to be Ready*: Monitor the instance's status using the describe-instances command.
Command:
aws ec2 describe-instances --instance-ids $INSTANCE --query 'Reservations[].Instances[].State.Name' --output text

7. *Test the Web Server*: Retrieve the public DNS name of the instance and test the web server by accessing it through a browser.
Command:
aws ec2 describe-instances --instance-ids $INSTANCE --query Reservations[].Instances[].PublicDnsName --output text


The AWS CLI allows for repeatable and reliable automation, making it ideal for consistent deployments.
