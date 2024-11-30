Terraform Commands
-----------------
terraform init
terraform plan
terraform apply
terraform destory
terraform fmt
terraform validate
terraform output
terraform providers

variables
----------

variable "string_type" {
 description = "This is a variable of type string"
 type        = string
 default     = "Default string value for this variable"
}

variable "string_heredoc_type" {
 description = "This is a variable of type string"
 type        = string
 default     = <<EOF
hello, this is Sumeet.
Do visit my website!
EOF
}

variable "number_type" {
 description = "This is a variable of type number"
 type        = number
 default     = 42
}

variable "boolean_type" {
 description = "This is a variable of type bool"
 type        = bool
 default     = true
}

variable "list_type" {
 description = "This is a variable of type list"
 type        = list(string)
 default     = ["string1", "string2", "string3"]
}

variable "map_type" {
 description = "This is a variable of type map"
 type        = map(string)
 default     = {
   key1 = "value1"
   key2 = "value2"
 }
}



blocks
------------------

Provider
-------

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 1.0"
    }
  }
}

provider "aws" {
  region = "eu-east-1"
}


>=1.0 - Greater than or equal to the version
<=1.0 - Less than or equal to the version
~>2.0 - Any version in the 2.X range
>=2.10, <=2.30 - Any version between 2.10 and 2.30


resource
--------
resource "aws_eip" "web_eip" {
instance = aws_instance.web.id
  tags     = var.tags
}
 
output
-----------
output "sns_topic_arn" {
  value       = aws_sns_topic.alarm_topic.arn
  description = "ARN of the SNS topic"
}


aws resources
-------------
aws_vpc
aws_subnet
aws_internet_gateway
aws_route_table
aws_route_table_association
aws_security_group
aws_key_pair
aws_instance
aws_eip
aws_ebs_volume
aws_volume_attachment
aws_sns_topic
aws_sns_topic_subscription
aws_cloudwatch_metric_alarm
aws_s3_bucket
aws_s3_object

