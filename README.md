# Secure Multi-Tier Network Architecture Using Amazon VPC (`Dee-vpc`)

A perimeter-secured, multi-subnet virtual network infrastructure built on **Amazon Virtual Private Cloud (VPC)**. This project demonstrates network isolation across public and private subnets inside **`Dee-vpc`** (`vpc-083257800fe1edc0d`, CIDR `10.0.0.0/16`), bidirectional public internet routing via **`Dee-IGW`** (`igw-02a69284227856f26`), controlled private outbound access via a **NAT Gateway**, and layered traffic filtering using **Security Groups** and **Route Tables**.

---

## Architecture Overview

### How It Works
1. **Custom VPC (`Dee-vpc`):** A dedicated **Amazon VPC** (`vpc-083257800fe1edc0d`) with IPv4 CIDR **`10.0.0.0/16`** is partitioned into isolated **Public Subnets** (for internet-facing resources) and **Private Subnets** (for internal backend workloads).
2. **Public Internet Routing (`Dee-IGW`):** The Internet Gateway **`Dee-IGW`** (`igw-02a69284227856f26`) is attached to `Dee-vpc` to enable bidirectional internet connectivity for public-facing components via a custom **Public Route Table** (`0.0.0.0/0 -> igw-02a69284227856f26`).
3. **Controlled Private Outbound Access (NAT Gateway):** A **NAT Gateway** deployed inside a public subnet allows instances in private subnets to download patches and updates from the internet via a **Private Route Table** (`0.0.0.0/0 -> nat-id`) while blocking unsolicited inbound internet connections.
4. **Defense-in-Depth Security:** Stateful **Security Groups** and Network ACLs (`acl-0a518a87755bb8d8d`) enforce least-privilege port rules across tiers.

---

## Repository Structure

```text
Amazon-VPC/
├── images/
│   ├── Screenshot 2026-10-19 235145.png
│   ├── Screenshot 2026-10-19 235213.png
│   ├── Screenshot 2026-10-19 235306.png
│   ├── Screenshot 2026-10-19 235816.png
│   ├── Screenshot 2026-10-20 000340.png
│   └── Screenshot 2026-10-20 000925.png
└── README.md
```

---

## AWS Services & Resources Used

| Component | Resource Name / ID | Purpose |
| :--- | :--- | :--- |
| **Amazon VPC** | `Dee-vpc` (`vpc-083257800fe1edc0d`) | Isolated virtual network environment configured with IPv4 CIDR `10.0.0.0/16`. |
| **Internet Gateway** | `Dee-IGW` (`igw-02a69284227856f26`) | Attached to `Dee-vpc` to provide internet connectivity for public subnet resources. |
| **Public & Private Subnets** | Multi-tier subnets inside `10.0.0.0/16` | Separates internet-accessible resources from internal backend workloads. |
| **NAT Gateway** | Public-subnet NAT Gateway | Enables outbound-only internet access for instances residing in private subnets. |
| **Route Tables** | Main (`rtb-0efcfb41ee747dea4`) & Custom Route Tables | Directs network traffic between subnets, `Dee-IGW`, and the NAT Gateway. |
| **Security Groups & NACL** | Tiered SGs & `acl-0a518a87755bb8d8d` | Enforces stateful instance-level and stateless subnet-level traffic filtering. |

---

## Setup & Deployment Walkthrough

### 1. Create the Custom VPC (`Dee-vpc`)
1. Open the **Amazon VPC Console** and create **`Dee-vpc`** (`vpc-083257800fe1edc0d`) with IPv4 CIDR **`10.0.0.0/16`**.
2. Verify the default Main Route Table (`rtb-0efcfb41ee747dea4`) and Main Network ACL (`acl-0a518a87755bb8d8d`) are provisioned.

- **Screenshot:** [View Screenshot 2026-10-19 235145.png](https://github.com/deeh-tech/Amazon-VPC/blob/main/images/Screenshot%202026-10-19%20235145.png)

---

### 2. Configure Public & Private Subnets
1. Partition **`10.0.0.0/16`** into dedicated **Public Subnets** and **Private Subnets** across Availability Zones.
2. Enable auto-assign public IPv4 addresses on the public subnets.

- **Screenshot:** [View Screenshot 2026-10-19 235213.png](https://github.com/deeh-tech/Amazon-VPC/blob/main/images/Screenshot%202026-10-19%20235213.png)

---

### 3. Create & Attach the Internet Gateway (`Dee-IGW`)
1. Navigate to **VPC** > **Internet gateways** and create **`Dee-IGW`** (`igw-02a69284227856f26`).
2. Attach **`Dee-IGW`** to **`Dee-vpc`** (`vpc-083257800fe1edc0d`) and confirm its state changes to **Attached**.

- **Screenshot:** [View Screenshot 2026-10-19 235306.png](https://github.com/deeh-tech/Amazon-VPC/blob/main/images/Screenshot%202026-10-19%20235306.png)

---

### 4. Configure Route Tables & NAT Gateway
1. Create a **Public Route Table**, associate it with your public subnets, and add a default route (`0.0.0.0/0`) targeting **`Dee-IGW`** (`igw-02a69284227856f26`).
2. Provision a **NAT Gateway** with an Elastic IP inside a public subnet and configure a **Private Route Table** routing `0.0.0.0/0` to the NAT Gateway.

- **Screenshot:** [View Screenshot 2026-10-19 235816.png](https://github.com/deeh-tech/Amazon-VPC/blob/main/images/Screenshot%202026-10-19%20235816.png)

---

## Testing & Verification

### 5. Security Groups & Bastion Host Connectivity
1. Configure least-privilege **Security Groups** restricting inbound SSH (Port 22) to trusted sources and internal private-tier access strictly to the Bastion Host security group.
2. Verify SSH access into the public Bastion Host and jump connectivity into the private subnet instance.

- **Screenshot:** [View Screenshot 2026-10-20 000340.png](https://github.com/deeh-tech/Amazon-VPC/blob/main/images/Screenshot%202026-10-20%20000340.png)

---

### 6. Verify Outbound Internet Access via NAT Gateway
1. Confirm that instances in the private subnet can reach external endpoints outbound through the NAT Gateway while remaining isolated from unsolicited inbound internet traffic.

- **Screenshot:** [View Screenshot 2026-10-20 000925.png](https://github.com/deeh-tech/Amazon-VPC/blob/main/images/Screenshot%202026-10-20%20000925.png)

---

## Key Findings & Takeaways

- **Network Segmentation (`10.0.0.0/16`):** Separating public and private subnets inside `Dee-vpc` isolates backend compute workloads from direct public internet exposure.
- **Controlled Internet Ingress & Egress:** Pairing `Dee-IGW` (`igw-02a69284227856f26`) on public route tables with a NAT Gateway on private route tables ensures public services remain reachable while private instances can securely pull OS updates outbound.
- **Layered Perimeter Defense:** Combining stateful Security Groups with subnet-level Network ACLs (`acl-0a518a87755bb8d8d`) enforces defense-in-depth across every tier.
