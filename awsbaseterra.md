
**`variables.tf`**
```hcl
variable "region" {
  default = "us-east-1"
}
 
variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  default     = "ami-0c02fb55956c7d316" # Replace with your preferred AMI ID
}
 
variable "instance_type" {
  description = "Type of the EC2 instance"
  default     = "t2.micro"
}
 
variable "key_name" {
  description = "Key pair name for SSH access"
  default     = "my-key-pair"
}
 
variable "tags" {
  description = "Tags for the resources"
  default     = {
    Environment = "Dev"
    Project     = "Terraform"
  }
}
```
 
---
 
**`main.tf`**
```hcl
provider "aws" {
  region = var.region
}
 
# Create VPC
resource "aws_vpc" "main" {
cidr_block = "10.0.0.0/16"
  enable_dns_hostnames = true
  tags                 = var.tags
}
 
# Create Public Subnet
resource "aws_subnet" "public" {
vpc_id = aws_vpc.main.id
cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true
  tags                    = var.tags
}
 
# Create Internet Gateway
resource "aws_internet_gateway" "igw" {
vpc_id = aws_vpc.main.id
  tags   = var.tags
}
 
# Create Route Table
resource "aws_route_table" "public" {
vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
gateway_id = aws_internet_gateway.igw.id
  }
  tags = var.tags
}
 
# Associate Route Table with Subnet
resource "aws_route_table_association" "public" {
subnet_id = aws_subnet.public.id
route_table_id = aws_route_table.public.id
}
 
# Create Security Group
resource "aws_security_group" "web_sg" {
vpc_id = aws_vpc.main.id
 
  # Allow inbound SSH access
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # Open to all; replace with specific IP for better security
  }
 
  # Allow inbound HTTP access
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
 
  # Allow all outbound traffic
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
 
  tags = var.tags
}
 
# Create Key Pair
resource "aws_key_pair" "key" {
  key_name   = var.key_name
public_key = file("~/.ssh/id_rsa.pub") # Replace with your public key file path
}
 
# Create EC2 Instance
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
subnet_id = aws_subnet.public.id
  key_name      = aws_key_pair.key.key_name
security_groups = [aws_security_group.web_sg.name]
 
  tags = var.tags
}
 
# Attach Elastic IP to EC2
resource "aws_eip" "web_eip" {
instance = aws_instance.web.id
  tags     = var.tags
}
 
# Create EBS Volume
resource "aws_ebs_volume" "web_volume" {
  availability_zone = aws_instance.web.availability_zone
  size              = 10 # Volume size in GiB
  tags              = var.tags
}
 
# Attach EBS Volume to EC2
resource "aws_volume_attachment" "web_volume_attachment" {
  device_name = "/dev/xvdf"
volume_id = aws_ebs_volume.web_volume.id
instance_id = aws_instance.web.id
}
 
# Create SNS Topic
resource "aws_sns_topic" "alarm_topic" {
  name = "ec2-cpu-alarm-topic"
}
 
# Subscribe Email to SNS Topic
resource "aws_sns_topic_subscription" "email" {
  topic_arn = aws_sns_topic.alarm_topic.arn
  protocol  = "email"
  endpoint  = "your-email@example.com" # Replace with your email
}
 
# Create CloudWatch Alarm
resource "aws_cloudwatch_metric_alarm" "cpu_alarm" {
  alarm_name          = "high-cpu-utilization"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 300
  statistic           = "Average"
  threshold           = 70
  alarm_actions       = [aws_sns_topic.alarm_topic.arn]
  dimensions = {
InstanceId = aws_instance.web.id
  }
}
```
 
---
 
**`outputs.tf`**
```hcl
output "instance_public_ip" {
  value       = aws_eip.web_eip.public_ip
  description = "Public IP of the EC2 instance"
}
 
output "sns_topic_arn" {
  value       = aws_sns_topic.alarm_topic.arn
  description = "ARN of the SNS topic"
}
 
output "security_group_id" {
value = aws_security_group.web_sg.id
  description = "ID of the Security Group"
}
```
 
--------------------------------
provider "aws" {
    region = "us-west-2" 
    access_key = "ExAmpLEKey21fjwljsfjkrf"
    secret_key = "EXamPLeSECreTKEySDAHhhhsJJn23Ksda"
}

terraform destroy -target aws_instance.my_ec2

>=1.0 - Greater than or equal to the version
<=1.0 - Less than or equal to the version
~>2.0 - Any version in the 2.X range
>=2.10, <=2.30 - Any version between 2.10 and 2.30

terraform apply -var-file="prod.tfvars"
export TF_VAR_variablename="value"

eg. export TF_VAR_instancetype="t2.nano"

