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
6. [Egress Only Internet Gateway](#egress-igw)
7. [Route 53 Private Zones](#private-zones)
8. [Network Access Contrl List (ACL)](#network-acl)
9. [VPC Peering](#vpc-peering)s
10. [VPC Endpoints](#vpc-endpoints)
11. [Bastion Hosts](#bastion-hosts)
12. [Flow Logs](#flow-logs)
13. [Site to Site VPN](#site-to-site-vpn)
14. [Direct Connect](#direct-connect)
15. [Transit Gateways](#transit-gateways)

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

**Note:** Max CIDR size in AWS is /16, so we cannot assign more than that.

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

## Egress Only Internet Gateway <a name="egress-igw"></a>

Egress only Internet Gateways are like NAT Gateways, but are used for instances with IPv6 addresses. Since all IPv6 are public addresses, all IPv6 instances are internet accessible. The solution is to use a Egress only Internet Gateway between IPv6 instances and the internet.

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
  <img src="assets/network-acl.jpg">
</p>

### Outgoing Requests

For outgoing requests, both ACL and SG will evaluate the outbound traffic, but incoming traffic will only be evaluated by SG. This is just like above.

<p align=center>
  <img src="assets/network-acl2.jpg">
</p>

**\[Hands-on tip]:** Don't forget to set outbound rules for SG on EC2 instances. The next section in VPC peering requires SG outbound to be set properly.

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

## Flow Logs <a name="flow-logs"></a>

There are 3 kinds of flow logs that capture information about IP traffic going into our interfaces:

1. VPC flow logs
2. Subnet flow logs
3. ENI flow logs

Data can go to S3 / CloudWatch. Flow logs can capture network information from AWS managed interfaces too, such as ELB, RDS, ElastiCache, Redshift, Workspaces.

Below are the default fields of flow logs:

| Fields       | Description                                                                                                                                                                                                                                                                                                                                                     |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| version      | The VPC Flow Logs version. If you use the default format, the version is 2. If you use a custom format, the version is the highest version among the specified fields. For example, if you only specify fields from version 2, the version is 2. If you specify a mixture of fields from versions 2, 3, and 4, the version is 4.                                |
| account-id   | The AWS account ID of the owner of the source network interface for which traffic is recorded. If the network interface is created by an AWS service, for example when creating a VPC endpoint or Network Load Balancer, the record may display unknown for this field.                                                                                         |
| interface-id | The ID of the network interface for which the traffic is recorded.                                                                                                                                                                                                                                                                                              |
| srcaddr      | The source address for incoming traffic, or the IPv4 or IPv6 address of the network interface for outgoing traffic on the network interface. The IPv4 address of the network interface is always its private IPv4 address. See also pkt-srcaddr.                                                                                                                |
| dstaddr      | The destination address for outgoing traffic, or the IPv4 or IPv6 address of the network interface for incoming traffic on the network interface. The IPv4 address of the network interface is always its private IPv4 address. See also pkt-dstaddr.                                                                                                           |
| srcport      | The source port of the traffic.                                                                                                                                                                                                                                                                                                                                 |
| dstport      | The destination port of the traffic.                                                                                                                                                                                                                                                                                                                            |
| protocol     | The IANA protocol number of the traffic. For more information, see Assigned Internet Protocol Numbers                                                                                                                                                                                                                                                           |
| packets      | The number of packets transferred during the flow.                                                                                                                                                                                                                                                                                                              |
| bytes        | The number of bytes transferred during the flow.                                                                                                                                                                                                                                                                                                                |
| start        | The time, in Unix seconds, when the first packet of the flow was received within the aggregation interval. This might be up to 60 seconds after the packet was transmitted or received on the network interface.                                                                                                                                                |
| end          | The time, in Unix seconds, when the last packet of the flow was received within the aggregation interval. This might be up to 60 seconds after the packet was transmitted or received on the network interface.                                                                                                                                                 |
| action       | The action that is associated with the traffic: ACCEPT: The recorded traffic was permitted by the security groups and network ACLs. REJECT: The recorded traffic was not permitted by the security groups or network ACLs.                                                                                                                                      |
| log-status   | The logging status of the flow log: OK: Data is logging normally to the chosen destinations. NODATA: There was no network traffic to or from the network interface during the aggregation interval. SKIPDATA: Some flow log records were skipped during the aggregation interval. This may be because of an internal capacity constraint, or an internal error. |

**\[Exam tip]:** Action field's ACCEPT or REJECT can be due to SG / Network ACL

## Site to Site VPN <a name="site-to-site-vpn"></a>

To set up a Site to Site VPN connection from on-premise datacentre to AWS, we need the following to be setup:

1. VPN gateway (Attached to VPC)
2. Site to Site VPN connection
3. Customer gateway on premise

Customer gateway is a software application or physical device on customer side of the VPN connection. We need a static, routable IP address for the customer gateway device. If gateway is behind a NAT, NAT-T must be configured and use public IP address of the NAT.

## Direct Connect <a name="direct-connect"></a>

Provides a dedicated private connection from customer network to VPC:

- Dedicated connection must be setup between datacentre and AWS Direct Connect locations.
- Need to setup a Virtual Private Gateway to VPC

Direct connect can access public resources (S3) and private (EC2) on same connection.

Use Cases:

- Increase bandwidth throughput - working with large data sets at lower costs.
- More consistent network experience - applications using real-time data feeds.
- For hybrid environments.

Direct connect gateway can connect to multiple VPCs, but it is not a VPC peering point.

Connection types:

1. Dedicated connections: 1Gbps and 10 Gbps capacity.

   - Physical ethernet port dedicated to customer.

2. Hosted connections: 50Mbps, 500Mbps, to 10Gbps

   - Capacity can be added or removed on demand.
   - 1, 2, 5, 10 Gbps available at select AWS Direct Connect Partners.

There is a minimum lead time of 1 month to establish the connection.

**Note:** Data in transit is not encrypted. The solution is to provide a VPN connection on top of Direct Connect.

## Transit Gateway <a name="transit-gateway></a>

Transit Gateway is a peering service between thousands of VPC and Direct Connect and VPN connections to provide a hub-and-spoke connection between them. This is a regional resource, but can work cross-region and cross-account (using Resource Access Manager).

This simplifies the network topology and provides IP multicast.
