---
sort: 1
---
# IAM - Identity & Access Management

## Testing IAM policies with the IAM policy simulator
Documentation : [Link](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html)\
Simulator Tool : [Link](https://policysim.aws.amazon.com/home/index.jsp?#)
	

## IAM Policy Documentation

### Allow Full Access to View/Edit Billing Page:

For allowing an IAM Entity to have access to billing page (Read/Write)\
Apart from this, Remember to activate IAM User and role access to console:
- My account -> IAM User and Role Access to Billing Information -> Activate IAM Access 

Service Type: Billing (aws-portal)

JSON:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "aws-portal:*",
            "Resource": "*"
        }
    ]
}
```

### Allow EC2 machine to Access ENI resources

To allow ec2 machine for releasing,associating and describing ENIs.\
Use Case: During Auto-Scaling, scripts can be run for the new ec2 host to take over an elastic IP. This policy will provide the permissions for it

Service Type: ec2

JSON:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ec2:ReleaseAddress",
                "ec2:DisassociateAddress",
                "ec2:DescribeNetworkInterfaces",
                "ec2:AssociateAddress",
                "ec2:AllocateAddress"
            ],
            "Resource": "*"
        }
    ]
}
```

### Allow EC2 machine to decode authorization message 

To allow ec2 machine to decode authorization messages using aws cli

Service Type: ec2

JSON
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "sts:DecodeAuthorizationMessage",
            "Resource": "*"
        }
    ]
}
```