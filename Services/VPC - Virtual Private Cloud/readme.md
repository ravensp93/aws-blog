---
sort: 2
---

# VPC - Virtual Private Cloud

## Contents

1. [CIDR - Classless Inter-Domain Routing](#cidr)
2. [Default VPC](#default-vpc)
3. [Subnets](#subnets)
4. [Internet Gateways and Route Tables](#gw-route-tables)
5. [Network Address Translation (NAT)](#nat)

## CIDR - Classless Inter-Domain Routing <a name="cidr"></a>

CIDR helps to define IP address range. A CIDR has 2 components:

1. Base IP (e.g 192.168.44.13) - represents an IP contained in the range.
2. Subnet Mask (e.g /26) - defines how many bits can change in the IP.

Subnet masks works in the following way:

- /32 allows for 1 (i.e 2^0) IP
- /31 allows for 2 (i.e 2^1) IP
- /30 allows for 4 (i.e 2^2) IP
- ...
- /24 allows for 256 (i.e 2^8) IP
- /16 allows for 65,536 (i.e 2^16) IP
- /0 allows for all 2^32 IP

More commonly used:

- /32 - no IP number can change (i.e 192.168.188.234)
- /24 - last IP number can change (i.e 192.168.188.x)
- /16 - last 2 IP number can change (i.e 192.168.x.x)
- /8 - last 3 IP number can change (i.e 192.x.x.x)
- /0 - all IP numbers can change (i.e x.x.x.x)

For quick calculation, go to https://www.ipaddressguide.com/cidr.

Private IP can only be allowed the following ranges as established by Internet Assigned Numbers Authority (IANA):

- 10.0.0.0/8 - for big networks with 2^24 IP
- 172.16.0.0 - 172.31.255.255 (172.16.0.0/12) - default AWS private addressing
- 192.168.0.0/16 - for home networks

## Default VPC <a name="default-vpc"></a>

Configurations of the Default VPC includes:

- Internet connectivity.
- Public IP for all instances.
- Public and private DNS name for all instances.

## Subnets <a name="subnets"></a>

Subnets are tied to Availability Zones. Subnets can be either public or private. Usually public subnets are smaller than private subnets, because all applications reside in private subnets while application load balancers resides in public subnets.

**Note**: AWS **reserves 5 IP addresses** in each subnet:

- 10.0.0.0: Network address
- 10.0.0.1: VPC Router
- 10.0.0.2: Mapping to AWS DNS
- 10.0.0.3: Reserved for future use
- 10.0.0.255: Network broadcast address, reserved as AWS does not allow broadcast in VPC

## Internet Gateways and Route Tables <a name="igw-route-tables"></a>

Internet Gateways help VPC instances connect with the internet, but require route tables to be edited. **Each VPC can only have one internet gateway attached.**

To connect resources in a VPC subnet to internet:

1. Create an internet gateway.
2. Create a route table and attach to VPC.
3. Associate route table with the VPC subnets.
4. Add a route with Destination: 0.0.0.0/0 and target:\<created internet gateway> in route table.

## Network Address Translation (NAT) <a name="nat"></a>

While Internet Gateways allow instances in the public subnet to connect to the internet, they also cause our instances to be accessible from the internet.

**For instances in the private subnet, we do not want them to be accessible from the internet, but we want them to access the internet** (to download software etc). This is what NAT is for.

### Using NAT Instance

The old way of NAT for private subnet is using NAT EC2 Instances. To do so, we must:

1. Launch an Amazon Linux AMI with NAT pre-configured in public subnet.
2. Set security groups inbound rules with allow HTTP/HTTPS from private subnets and allow SSH from our own desktop.
3. Set security groups outbound rules to allow HTTP/HTTPS to internet.
4. Turn off source/destination check on NAT Instance.
5. In VPC console, create a route with target as NAT Instance and Destination as 0.0.0.0, making any public destination traffic route to NAT Instance's ENI.

### Using NAT Gateway

This is the proper way to set up NAT is the route to a NAT Gateway in the public subnet.

NAT Gateway is resilient in a single AZ. Multiple NAT Gateways must be created one in each AZ for multi-AZ availability.

1. Create a NAT Gateway in the public subnet via the VPC console.
2. Attach an Elastic IP to the NAT Gateway.
3. Add a route in the route table of the private subnet for destination 0.0.0.0 to target the NAT Gateway.
