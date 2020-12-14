---
sort: 1
---
# AWS CLI - Command Line Interface

Interact with AWS Proprietary Services (S3, Dynamo DB, etc..) using:\
1) CLI on Local Computer
setup: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html\
2) CLI on EC2 Machines\
3) AWS SDK on Local Computer\
4) AWS SDK on EC2 Machines\
5) AWS Instance Metadata on EC2

## AWS CLI Configuration (CLI on local computer)

Configure CLI to access your AWS account through the web.

<p align=center>
  <img src="blob/aws-cli-pic1.PNG">
</p>

To Acquire Access Key ID & Secret Access key:\ 
IAM > Users > Select User > Security Credentials > Create Access Key

To configure AWS CLI to connect to your account, in your CLI:\
1) Execute "aws configure"\
2) Specify Access Key ID / Secret Access Key / Default Region\
3) Verify configuration with ls ~/.aws, you should see "config" and "credentials"

## AWS CLI Commands Repository 