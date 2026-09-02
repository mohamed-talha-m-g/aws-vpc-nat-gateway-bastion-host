# AWS VPC Architecture: NAT Gateway & Bastion Host Setup

A hands-on AWS networking project that builds a custom VPC with public and private subnets, uses a **NAT Gateway** to give the private subnet outbound internet access, and demonstrates the **bastion host pattern** — SSH-ing into a private EC2 instance through a public-facing "jump" instance.

---

## 📐 Architecture

```
                         Internet
                             │
                    ┌────────┴────────┐
                    │ Internet Gateway│
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │   Custom VPC     │  12.0.0.0/24
                    │                  │
        ┌───────────┴───────┐  ┌───────┴────────────┐
        │  Public Subnet     │  │  Private Subnet     │
        │  12.0.1.0/24       │  │  12.0.2.0/24         │
        │                    │  │                      │
        │  ┌──────────────┐  │  │  ┌────────────────┐  │
        │  │ Public EC2    │──┼──┼─▶│ Private EC2     │  │
        │  │ (Bastion Host)│  │  │  │ (No Public IP)  │  │
        │  │ Public IP: ✅ │  │  │  └────────────────┘  │
        │  └──────────────┘  │  │           ▲          │
        │         │          │  │           │          │
        │  ┌──────▼───────┐  │  │  Private Route Table │
        │  │  NAT Gateway  │◀─┼──┼───────────┘          │
        │  └──────────────┘  │  │  (0.0.0.0/0 → NAT GW)│
        └────────────────────┘  └──────────────────────┘
```

| Component | CIDR / Detail |
|---|---|
| VPC | `12.0.0.0/24` (256 IPs) |
| Public Subnet | `12.0.1.0/24` |
| Private Subnet | `12.0.2.0/24` |
| Public Route Table | `0.0.0.0/0` → Internet Gateway |
| Private Route Table | `0.0.0.0/0` → NAT Gateway |
| Public EC2 | Public IP enabled — acts as bastion/jump host |
| Private EC2 | No public IP — reachable only from the bastion via private IP |

---

## 🎯 What This Project Demonstrates

- Designing a custom VPC and subnetting it into public/private tiers
- Configuring route tables and associating them with the right subnets
- Attaching and routing traffic through an Internet Gateway
- Setting up a NAT Gateway so a private subnet can reach the internet (updates, patches) **without** being reachable from it
- Security-conscious instance design: the private EC2 has no public IP and is isolated from direct internet exposure
- The bastion/jump-host pattern for administering private resources over SSH
- SSH key pair (`.pem`) management and permissions (`chmod 400`)

---

## 🛠️ Steps Performed

1. **Created the VPC** — `12.0.0.0/24`, with one public subnet (`12.0.1.0/24`) and one private subnet (`12.0.2.0/24`).
2. **Created and attached an Internet Gateway (IGW)** to the VPC.
3. **Created two route tables** — one for the public subnet, one for the private subnet.
4. **Configured the public route table** — added a route to `0.0.0.0/0` via the IGW, and associated it with the public subnet.
5. **Configured the private route table** and associated it with the private subnet (no direct internet route yet).
6. **Created a NAT Gateway** inside the public subnet, allocated an Elastic IP to it.
7. **Updated the private route table** — added a route to `0.0.0.0/0` via the NAT Gateway, giving the private subnet outbound-only internet access.
8. **Launched two EC2 instances** in the custom VPC:
   - Public EC2 — public IP enabled, placed in the public subnet (bastion host).
   - Private EC2 — **no public IP**, placed in the private subnet.
9. **Connected to the public EC2 instance**, copied the `.pem` key onto it via `vi`, and set permissions with `chmod 400 key.pem`.
10. **SSH'd from the public EC2 into the private EC2** using the private instance's private IP — verifying the private instance is reachable only through the bastion.

## 📸 Screenshots

| Step | Screenshot |
|---|---|
| VPC & Subnets created | `![VPC Setup](screenshots/01-vpc-subnets.png)` |
| Internet Gateway attached | `![IGW](screenshots/02-igw.png)` |
| Route tables configured | `![Route Tables](screenshots/03-route-tables.png)` |
| NAT Gateway created | `![NAT Gateway](screenshots/04-nat-gateway.png)` |
| EC2 instances running | `![EC2 Instances](screenshots/05-ec2-instances.png)` |
| Internet Gateway attach to vpc | `![IGW Setup](screenshots/06-igw-attach-to-vpc)` |
| Route tables associate to subnets | `![Route table associat to subnet](screenshots/07-rt-associate-to-private-subnet)` |
| Subnet association | `![Subnets](screenshots/08-subnet-associate)` |


## 🧰 Tools & Services Used

`AWS VPC` · `EC2` · `Internet Gateway` · `NAT Gateway` · `Route Tables` · `Security Groups` · `SSH` · `Elastic IP`

---

## 🔑 Key Takeaways

- A NAT Gateway lets private resources initiate outbound connections (e.g., for updates) while remaining unreachable from the public internet — a core pattern for securing backend infrastructure.
- The bastion host pattern keeps SSH exposure to a single, tightly controlled entry point instead of exposing every instance.
- Route table associations, not subnet naming, are what actually determine whether a subnet is "public" or "private" in AWS.

---

## ⚠️ Security Note

This is a learning/lab project. In a production setup you'd also want to:
- Restrict the bastion's security group to SSH from a known IP range only, not `0.0.0.0/0`
- Use a Session Manager (SSM) connection instead of a bastion host where possible
- Never commit `.pem` key files to version control (see `.gitignore` in this repo)

---

## 📄 License

This project is shared for educational purposes.
