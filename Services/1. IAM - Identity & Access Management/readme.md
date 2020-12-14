---
sort: 1
---
# IAM - Identity & Access Management

## IAM Policy Documentation

### Allow Full Access to View/Edit Billing Page:
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

### Allow EC2 Instances to access ENI resources
Service Type: EC2

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