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
6. [Route 53 Private Zones](#private-zones)
7. [Network Access Contrl List (ACL)](#network-acl)
8. [VPC Peering](#vpc-peering)
9. [VPC Endpoints](#vpc-endpoints)
10. [Bastion Hosts](#bastion-hosts)

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

## Route 53 Private Zones <a name="private-zones"></a>

With VPC DNS Resolution, we are able to create private zones:

1. Set enableDnsSupport to true
   - Helps decide if DNS resolution is supported for VPC.
   - Default true during VPC creation.
   - If true, all instances in VPC queries the AWS DNS server at 169.254.169.253
2. Set enableDnsHostname to true
   - If true, assigns public hostname to EC2 instance if it has a public IP.
   - False by default for newly created VPC. True by default for Default VPC.
3. Create a private zone in Route 53.

For more information on creating Route 53 zones, refer to Route 53 section of this blog (coming soon).

## Network Access Contrl List (ACL) <a name="network-acl"></a>

**\[Exam tip]: Understand the difference between Network ACL and Security Groups.**

Network ACL are like subneet level firewall:

- Each subnet will only have one ACL, and new subnets are assigned Default ACL.
- Newly created ACLs will deny everything.
- ACLs are a great way of blocking a specific IP.
- Rules are defined as follows:
  - Rules have a number (1 - 32766), in which a lower number is a higher precedence.
  - E.g if #100 is ALLOW \<IP> and #200 is DENY \<IP>, \<IP> is allowed because #100 takes precedence.
  - Last rule is an asterisk (\*) that denies a request in case of no rule match.
  - AWS recommends adding rules by increments of 100.

### Incoming Requests

Incoming requests will be evaluated by both ACL and Security Group during ingress. During egress, Security Group is stateful so it will not evaluate if inbound traffic was allowed. ACL will still evaluate egress traffic because it is stateless.

<p align=center>
  <img src="assets/Network ACL.jpg">
</p>

### Outgoing Requests

For outgoing requests, both ACL and SG will evaluate the outbound traffic, but incoming traffic will only be evaluated by SG. This is just like above.

<p align=center>
  <img src="assets/Network ACL2.jpg">
</p>

## VPC Peering <a name="vpc-peering"></a>

VPC peering is used to connect 2 VPCs privately using AWS network, making them behave as if they were in the same network. VPCs must have non-overlapping CIDR.

**\[Exam tips]:**

1. VPC Peering is **not transitive**. VPC A peered VPC B and VPC C peered VPC B does not mean VPC A and C are connected.
2. We must update route tables in each VPC's subnets to ensure instances can communicate. Destination to the other VPC's CIDR and target the VPC peering.
3. VPC peering can work inter-region, cross account.
4. We can reference a security group of a peered VPC.

## VPC Endpoints <a name="vpc-endpoints"></a>

VPC Endpoints are used for accessing AWS services (S3, DynamoDB, etc) without routing through the public internet.

There are 2 kinds of VPC Endpoints:

1. **Interface:** provisions an ENI (private IP address) as an entry point (must attach security group) for most AWS services.
2. **Gateway:** provisions a target and must be used in a route table (for S3 and DynamoDB)

## Bastion Hosts <a name="bastion-hosts"></a>

Bastion Hosts are used to SSH into instances in the private subnet. Bastion Hosts are in the public subnet.

**\[Exam tip]:** Ensure that Bastion Hosts have tight security group controls, allowing only certain IPs to SSH to it.
