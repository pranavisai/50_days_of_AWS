## Create Key Pair in AWS 
1. command : aws ec2 create-key-pair  --key-name datacenter-kp   --key-type rsa  --region us-east-1   --query 'KeyMaterial'   --output text > datacenter-kp.pem
2. To check all pairs in the region: aws ec2 describe-key-pairs --region us-east-1 --query 'KeyPairs[*].KeyName'  --output table
3. To check the sepcific key: aws ec2 describe-key-pairs   --key-names datacenter-kp  --region us-east-1

## Create a security group and add the inbound rule
1. Verify the VPC ID: aws ec2 describe-vpcs \
  --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" \
  --output text \
  --region us-east-1
2. Create the security group with the VPC ID: aws ec2 create-security-group \
  --group-name devops-sg \
  --description "Security group for Nautilus App Servers" \
  --vpc-id <VPC_ID> \
  --region us-east-1
3. Add HTTP Inbound rule (port 80): aws ec2 authorize-security-group-ingress \
  --group-name devops-sg \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0 \
  --region us-east-1
4. Add SSH Inbound Rule (port 22): aws ec2 authorize-security-group-ingress \
  --group-name devops-sg \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0 \
  --region us-east-1
5. Verify the security group creation: aws ec2 describe-security-groups \
  --group-names devops-sg \
  --region us-east-1

## Create a subnet
1. Find the default VPC ID in the region: aws ec2 describe-vpcs \
  --filters "Name=isDefault,Values=true" \
  --query "Vpcs[0].VpcId" \
  --output text \
  --region us-east-1
2. Create a subnet with a CIDR block to not overlap with the others: aws ec2 create-subnet \
  --vpc-id <VPC_ID> \
  --cidr-block 172.31.100.0/24 \
  --availability-zone us-east-1a \
  --region us-east-1
3. Naming the subnet using the subnet-id: aws ec2 create-tags \
  --resources <SUBNET_ID> \
  --tags Key=Name,Value=xfusion-subnet \
  --region us-east-1
4. Verifying the subnet: aws ec2 describe-subnets \
  --filters "Name=tag:Name,Values=xfusion-subnet" \
  --region us-east-1

## S3 Versioning
1. S3 Versioning is used to preserve, retrieve, and restore every version of an object in a bucket. It protects against accidental deletion, unintended overwrites, and provides a simple recovery mechanism for data changes.
2. Enable S3 Versioning -> aws s3api put-bucket-versioning \
  --bucket nautilus-s3-11491 \
  --versioning-configuration Status=Enabled \
  --region us-east-1
3. Verify -> aws s3api get-bucket-versioning \
  --bucket nautilus-s3-11491 \
  --region us-east-1

## Creating EBS Volume
1. Creating the EBS Volume -> aws ec2 create-volume \
  --size 2 \ (The size is in GB)
  --volume-type gp3 \
  --availability-zone us-east-1a \
  --region us-east-1
2. Add tag name as required -> aws ec2 create-tags \
  --resources <VOLUME_ID> \
  --tags Key=Name,Value=nautilus-volume \
  --region us-east-1
3. Verify using Query -> aws ec2 describe-volumes \
  --filters "Name=tag:Name,Values=nautilus-volume" \
  --query "Volumes[*].[VolumeId,Size,VolumeType,AvailabilityZone]" \
  --output table \
  --region us-east-1

## Create and launch EC2 instance
1. Create the RSA key pair first -> aws ec2 create-key-pair \
  --key-name xfusion-kp \
  --key-type rsa \
  --query 'KeyMaterial' \
  --output text \
  --region us-east-1 > xfusion-kp.pem
2. Secure the private key -> chmod 400 xfusion-kp.pem
3. Find the AMI ID (as given in this example) -> aws ssm get-parameters \
  --names /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 \
  --query "Parameters[0].Value" \
  --output text \
  --region us-east-1
4. Find the default security group ID -> aws ec2 describe-security-groups \
  --filters Name=group-name,Values=default \
  --query "SecurityGroups[0].GroupId" \
  --output text \
  --region us-east-1
5. Launch the EC2 instance -> aws ec2 run-instances \
  --image-id <AMI_ID> \
  --instance-type t2.micro \
  --key-name xfusion-kp \
  --security-group-ids <SECURITY_GROUP_ID> \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=xfusion-ec2}]' \
  --region us-east-1
6. Verify -> aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" \
  --query "Reservations[].Instances[].{ID:InstanceId,State:State.Name,Type:InstanceType,Key:KeyName}" \
  --output table \
  --region us-east-1

## Changing the EC2 instance type 
1. Find the instance ID with name -> aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=nautilus-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text \
  --region us-east-1
2. Stop the instance -> aws ec2 stop-instances \
  --instance-ids <INSTANCE_ID> \
  --region us-east-1
3. Wait for the instance to stop -> aws ec2 wait instance-stopped \
  --instance-ids <INSTANCE_ID> \
  --region us-east-1
4. Change the instance type -> aws ec2 modify-instance-attribute \
  --instance-id <INSTANCE_ID> \
  --instance-type "{\"Value\":\"t2.nano\"}" \
  --region us-east-1
5. Start the instance -> aws ec2 start-instances \
  --instance-ids <INSTANCE_ID> \
  --region us-east-1
6. Wait until it's running -> aws ec2 wait instance-running \
  --instance-ids <INSTANCE_ID> \
  --region us-east-1
7. Verify -> aws ec2 describe-instances \
  --instance-ids <INSTANCE_ID> \
  --query "Reservations[].Instances[].{State:State.Name,Type:InstanceType}" \
  --output table \
  --region us-east-1

## Enable the stop protection for an instance
1. This is done to prevent accidental stopping of an instance. 
2. Example to get instance ID -> aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=datacenter-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text \
  --region us-east-1
3. Enable stop protection -> aws ec2 modify-instance-attribute \
  --instance-id <instance-id> \
  --disable-api-stop "{\"Value\":true}" \
  --region us-east-1
4. Verify the stop protection -> aws ec2 describe-instance-attribute \
  --instance-id <instance-id> \
  --attribute disableApiStop \
  --region us-east-1

## Enable the termination protection for an instance
1. This is done to prevent terminating (deleting) the instance.
2. Example to get instance ID -> aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=devops-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text \
  --region us-east-1
3. Enable termination protection -> aws ec2 modify-instance-attribute \
  --instance-id i-0fb44451a1bbf9b3a \
  --disable-api-termination "{\"Value\":true}" \
  --region us-east-1
4. Verify the termination protection -> aws ec2 describe-instance-attribute \
  --instance-id i-0fb44451a1bbf9b3a \
  --attribute disableApiTermination \
  --region us-east-1

## Attach EC2 instance to elastic-ip
1. Getting the EC2 instance ID -> aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=<ec2-instance-name>" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text \
  --region us-east-1
2. Get the Elastic IP allocation ID -> aws ec2 describe-addresses \
  --filters "Name=tag:Name,Values=<elastic-ip-name>" \
  --query "Addresses[].AllocationId" \
  --output text \
  --region us-east-1
3. Associate -> aws ec2 associate-address \
  --instance-id <ec2-instance-id> \
  --allocation-id <elastic-ip-allocation-id> \
  --region us-east-1
4. Verify -> aws ec2 describe-addresses \
  --allocation-ids <allocation-id> \
  --query "Addresses[].{PublicIP:PublicIp,Instance:InstanceId}" \
  --output table \
  --region us-east-1

## Attach the network interface to the EC2 instance
1. Instance ID -> aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" \
  --query "Reservations[].Instances[].InstanceId" \
  --output text \
  --region us-east-1
2. Network Interface ID (ENI ID) -> aws ec2 describe-network-interfaces \
  --filters "Name=tag:Name,Values=xfusion-eni" \
  --query "NetworkInterfaces[].NetworkInterfaceId" \
  --output text \
  --region us-east-1
3. Check if instance is running -> aws ec2 describe-instance-status \
  --instance-ids <INSTANCE_ID> \
  --query "InstanceStatuses[].{Instance:InstanceStatus.Status,System:SystemStatus.Status}" \
  --output table \
  --region us-east-1
4. Device index 0 is the Primary network interface (created automatically when the instance launches). The first additional network interface should be connected to device index 1.
   Attach the network interface to the instance -> aws ec2 attach-network-interface \
  --network-interface-id <ENI_ID> \
  --instance-id <INSTANCE_ID> \
  --device-index 1 \
  --region us-east-1
7. Verify -> aws ec2 describe-network-interfaces \
  --network-interface-ids <ENI_ID> \
  --query "NetworkInterfaces[].{Status:Status,AttachmentStatus:Attachment.Status,InstanceId:Attachment.InstanceId}" \
  --output table \
  --region us-east-1

## Attach volume to EC2 instance
1. Get the EC2 Instance ID.
2. Get Volume ID -> aws ec2 describe-volumes \
  --filters "Name=tag:Name,Values=<volume-name>" \
  --query "Volumes[].VolumeId" \
  --output text \
  --region us-east-1
3. Attach -> aws ec2 attach-volume \
  --volume-id <VOLUME_ID> \
  --instance-id <INSTANCE_ID> \
  --device <device-name> \
  --region us-east-1
4. Verify -> aws ec2 describe-volumes \
  --volume-ids <VOLUME_ID> \
  --query "Volumes[].Attachments[].{State:State,Device:Device,Instance:InstanceId}" \
  --output table \
  --region us-east-1

## Create AMI (Amazon Machine Image) for existing EC2 instance
1. Get Instance ID
2. Create AMI -> aws ec2 create-image \
  --instance-id <INSTANCE_ID> \
  --name "<ami-name>" \
  --region us-east-1
   We will get the image ID from this.
3. Use the wait command. This command blocks until the AMI state becomes available. -> aws ec2 wait image-available \
  --image-ids <AMI_ID> \
  --region us-east-1
4. Verify if in available state -> aws ec2 describe-images \
  --image-ids <AMI_ID> \
  --query "Images[].{Name:Name,State:State}" \
  --output table \
  --region us-east-1

## Delete EC2 instance
1. Get the instance ID with the instance name.
2. Terminate the instance -> aws ec2 terminate-instances \
  --instance-ids <INSTANCE_ID> \
  --region us-east-1
3. Wait until it's terminated -> aws ec2 wait instance-terminated \
  --instance-ids <INSTANCE_ID> \
  --region us-east-1
4. Verify the output, should be "terminated" -> aws ec2 describe-instances \
  --instance-ids <INSTANCE_ID> \
  --query "Reservations[].Instances[].State.Name" \
  --output text \
  --region us-east-1

## Create Snapshot of volume
1. Get Volume ID -> aws ec2 describe-volumes \
  --filters "Name=tag:Name,Values=xfusion-vol" \
  --query "Volumes[].VolumeId" \
  --output text \
  --region us-east-1
2. Create a snapshot with a name and description. We get a snapshot ID in return -> aws ec2 create-snapshot \
  --volume-id <VOLUME_ID> \
  --description "<description>" \
  --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=<snapshot-name>}]' \
  --region us-east-1
3. Verify the snapshot -> aws ec2 describe-snapshots \
  --snapshot-ids <SNAPSHOT_ID> \
  --query "Snapshots[].{Name:Tags[?Key=='Name']|[0].Value,Description:Description,State:State}" \
  --output table \
  --region us-east-1

## Create IAM User
1. IAM is a global service. We do not specify the region for it.
2. An IAM User is an individual identity in AWS created for a person or application. It has its own username, password, access keys, and permissions.
3. Create user -> aws iam create-user \
  --user-name <user-name>
4. Verify -> aws iam get-user \
  --user-name <user-name>
