
# AWS VPC Peering Project

**Connect two VPCs. Prove private connectivity.**

A hands-on AWS networking project completed as part of the EWB Cloud Labs AWS Internship, Batch 1. The project connects two Virtual Private Clouds (VPCs) through VPC peering, allowing private communication between application servers. A public bastion host in VPC A provides administrative SSH access to the private servers.

## Project Information

| Field | Details |
|---|---|
| Student | Kandikatla Sailaja |
| Batch | 1 |
| Region | Mumbai (`ap-south-1`) |
| Project | EWB Cloud Labs — AWS VPC Peering |
| Completed | October 2026 |
| Full report | `Kandikatla_Sailaja_EWB_VPC_Peering.pdf` |

## Architecture

<img width="775" height="659" alt="0" src="https://github.com/user-attachments/assets/78b9cdb2-a0bf-40f0-a1e6-c25c8c2d63d2" />

- **Administration:** Laptop → Bastion → App-A or App-B using SSH ProxyJump.
- **Application traffic:** App-A ↔ App-B using private IP addresses through the VPC peering connection.
- VPC B has no internet gateway. Both application servers are designed to remain private, without public IPv4 addresses.

## Address Plan

| Resource | Name tag | CIDR / Private IP |
|---|---|---|
| VPC A | `ewb-vpc-a` | `10.10.0.0/16` |
| Public subnet A | `ewb-a-public` | `10.10.1.0/24` |
| Private subnet A | `ewb-a-private` | `10.10.2.0/24` |
| VPC B | `ewb-vpc-b` | `10.20.0.0/16` |
| Private subnet B | `ewb-b-private` | `10.20.1.0/24` |
| Bastion | `ewb-bastion` | `10.10.1.10` + public IPv4 |
| App-A | `ewb-app-a` | `10.10.2.10` |
| App-B | `ewb-app-b` | `10.20.1.10` |

The application servers use private IP addresses. The bastion is the public entry point for SSH administration.

## AWS Resource Inventory

Resource IDs recorded during the project:

| Resource | AWS resource ID |
|---|---|
| VPC A | `vpc-0eb3a7706098e7110` |
| VPC B | `vpc-0a9f2293842e117a2` |
| Public subnet A | `subnet-0769cf8f819b2a8d2` |
| Private subnet A | `subnet-052c6d9411135d9a7` |
| Private subnet B | `subnet-0d9a9a92b1c481eed` |
| Internet gateway | `igw-06deee1691533f122` |
| VPC peering connection | `pcx-08200e6ae1aa24cbe` |
| Public route table | `rtb-0bb846f4723b0f01c` |
| Private route table A | `rtb-05a1c48bb1d6d331b` |
| Private route table B | `rtb-0215f99ca98b45ec8` |
| Bastion security group | `sg-06cacf3b93f151f2d` |
| App-A security group | `sg-06aa848aef879dfa9` |
| App-B security group | `sg-0889ff22e69c5d375` |

These are historical resource IDs. The lab resources were deleted during cleanup and are no longer active.

## What This Project Demonstrates

- Designing non-overlapping VPC CIDR ranges.
- Creating public and private subnets.
- Associating custom route tables with the appropriate subnets.
- Establishing VPC peering between two VPCs.
- Configuring forward and return routes.
- Restricting inbound traffic using security groups.
- Administering private EC2 instances through a bastion host.
- Testing connectivity using private IP addresses and ICMP.
- Understanding the effect of security group rules on network traffic.
- Cleaning up AWS resources after completing a lab.

## Routing Configuration

The following describes the intended routing design used for the project.

| Route table | Destination | Target | Purpose |
|---|---|---|---|
| Public route table A | `10.10.0.0/16` | Local | Traffic within VPC A |
| Public route table A | `0.0.0.0/0` | Internet gateway | Bastion internet access |
| Public route table A | `10.20.0.0/16` | VPC peering | Bastion access to VPC B |
| Private route table A | `10.10.0.0/16` | Local | Traffic within VPC A |
| Private route table A | `10.20.0.0/16` | VPC peering | App-A to App-B |
| Private route table B | `10.20.0.0/16` | Local | Traffic within VPC B |
| Private route table B | `10.10.0.0/16` | VPC peering | Return traffic to VPC A |

The local routes are automatically created by AWS. The peering routes provide the forward and return paths between the VPCs.

## Security Groups

The security design restricts SSH and ICMP traffic to specific sources.

| Security group | Protocol | Intended source |
|---|---|---|
| Bastion | SSH / TCP 22 | My public IPv4 address `/32` |
| App-A | SSH / TCP 22 | `10.10.1.10/32` |
| App-A | ICMP IPv4 | `10.20.1.10/32` |
| App-B | SSH / TCP 22 | `10.10.1.10/32` |
| App-B | ICMP IPv4 | `10.10.2.10/32` |

The application servers accept SSH from the bastion's private IP. ICMP rules permit the intended ping tests between the application servers.

Security group rules should be verified against the saved configuration screenshots before treating this table as a complete historical record.

## Private SSH Access

SSH ProxyJump allows the local machine to connect to a private server through the bastion. The private key remains on the local machine.

Example SSH configuration:

```sshconfig
Host ewb-bastion
    HostName <BASTION_PUBLIC_IP>
    User ec2-user
    IdentityFile ~/.ssh/ewb-lab.pem
    IdentitiesOnly yes

Host ewb-app-a
    HostName 10.10.2.10
    User ec2-user
    IdentityFile ~/.ssh/ewb-lab.pem
    IdentitiesOnly yes
    ProxyJump ewb-bastion

Host ewb-app-b
    HostName 10.20.1.10
    User ec2-user
    IdentityFile ~/.ssh/ewb-lab.pem
    IdentitiesOnly yes
    ProxyJump ewb-bastion
```

## Connectivity Tests

| Test | Purpose | Recorded outcome |
|---|---|---|
| Forward ping | App-A → App-B | Successful |
| Reverse ping | App-B → App-A | Successful |
| Private SSH | Laptop → App-B through bastion | Successful |
| Isolation | Confirm private application servers have no public IPs | Document using saved evidence |
| Controlled failure | Remove the destination ICMP rule, retest, then restore it | Include the recorded evidence if available |

Example commands executed on the relevant private hosts:

```bash
ping -c 4 10.20.1.10
ping -c 4 10.10.2.10
hostname
hostname -I
```

These tests demonstrate the role of routing and security group rules in private connectivity. Screenshots should contain the actual command output rather than expected results alone.

## Forward and Return Path

**Forward path — App-A to App-B**

The private route table in VPC A matches the destination `10.20.0.0/16` and forwards traffic through the VPC peering connection. The destination security group must allow the relevant inbound traffic from App-A.

**Return path — App-B to App-A**

The private route table in VPC B matches `10.10.0.0/16` and sends replies through the same peering connection. Security groups are stateful, so responses to permitted connections are automatically allowed.

**Bastion administration**

The bastion establishes a separate SSH connection to the destination private server. App-B therefore sees the bastion's private IP as the SSH source, not the laptop's public IP.

## Key Concepts Learned

- VPC peering connects two VPCs but does not automatically create routes.
- Both sides need the appropriate routes for bidirectional connectivity.
- VPC CIDR ranges must not overlap.
- Peering is non-transitive.
- VPC peering does not provide VPC B with internet access through VPC A's internet gateway.
- Security groups control which traffic is allowed to reach instances.
- A bastion host provides a controlled administrative entry point to private instances.
- Private IP connectivity does not require application servers to have public IP addresses.

## Cleanup

After saving the project evidence, I deleted the AWS resources created for the lab, including the EC2 instances and associated lab networking resources.

Cleanup included checking for remaining EBS volumes, peering connections, routes, security groups, subnets, custom route tables and the internet gateway.

The report and repository preserve the learning record after the AWS resources have been removed.

## Repository Structure

```text
aws-vpc-peering-project/
└── screenshots/
|    ├── 01-Image 1
|    ├── 02-Image 2
|    ├── 03-Image 3
|    ├── 04-Image 4
|    ├──    .
|    ├──    .
|    ├──    .
|    └── 11-Image 17
├── .gitignore
├── Architecture.png
├── Kandikatla_Sailaja_EWB_VPC_Peering.pdf
└── README.md
```

## Security Notes

- Do not commit `.pem` private keys, passwords, access tokens or other credentials.
- Do not expose SSH or ICMP to `0.0.0.0/0` unnecessarily.
- Keep application servers private.
- Remove unused AWS resources after completing the lab to reduce ongoing costs.

## Author

**Kandikatla Sailaja**  
EWB Cloud Labs — AWS Internship, Batch 1
