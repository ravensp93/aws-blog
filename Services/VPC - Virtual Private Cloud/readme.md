---
sort: 2
---

# VPC - Virtual Private Cloud

## Contents

1. [CIDR - Classless Inter-Domain Routing](#cidr)
2. [Default VPC](#default-vpc)

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

## Subnets

Subnets are tied to Availability Zones. Subnets can be either public or private. Usually public subnets are smaller than private subnets, because all applications reside in private subnets while application load balancers resides in public subnets.

**Note**: AWS reserves 5 IP addresses in each subnet:

- 10.0.0.0: Network address
- 10.0.0.1: VPC Router
- 10.0.0.2: Mapping to AWS DNS
- 10.0.0.3: Reserved for future use
- 10.0.0.255: Network broadcast address, reserved as AWS does not allow broadcast in VPC
