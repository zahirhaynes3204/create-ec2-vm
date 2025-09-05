[
](https://www.loom.com/share/b9f12da7aae645c6962a3eb318400071?sid=cd323ab7-c874-4a1e-adac-09c769fbce41)
# Create a Virtual Machine on AWS (EC2) — Step‑by‑Step

Spin up a Windows or Linux virtual machine (VM) on AWS using EC2. This guide is beginner‑friendly and designed for your portfolio/GitHub.

> **What you’ll do:** pick an OS image (AMI), choose an instance type, set up networking & security, create a key pair, launch, connect, and clean up.

---

## Prerequisites

- **AWS account** with billing enabled
- **IAM user** with permission to use EC2 (e.g., `AmazonEC2FullAccess` for practice)
- **Region** selected (top‑right in AWS console)
- **Local tools (optional):**
  - For Linux/macOS: built‑in **SSH**
  - For Windows: **PuTTY** (SSH) and **Remote Desktop Connection (RDP)**

> ⚠️ **Costs**: t2.micro/t3.micro are free‑tier eligible in many regions, but **always stop/terminate** when done.

---

## 1) Launch an Instance (Console)

1. Sign in to **AWS Console** → search **EC2** → **Instances** → **Launch instances**.
2. **Name** your VM (e.g., `portfolio-ec2-vm`).
3. **Application & OS Image (AMI):**
   - For Windows: *Microsoft Windows Server* (e.g., 2022 Base)
   - For Linux: *Amazon Linux 2023* or *Ubuntu 22.04 LTS*
4. **Instance type:** `t2.micro` or `t3.micro` (free‑tier eligible).
5. **Key pair (login):** **Create new key pair** → type `RSA` → **Download** the `.pem` file and keep it safe.
6. **Network settings:**
   - **VPC/Subnet:** leave default (for first VM)
   - **Auto‑assign public IP:** **On**
   - **Security Group:** create a new group
     - For **Linux**: allow **SSH (TCP 22)** from **My IP**
     - For **Windows**: allow **RDP (TCP 3389)** from **My IP**
7. **Storage:** 8–30 GiB gp3 is fine for testing.
8. Click **Launch instance**. Wait until **Instance state = Running** and **Status checks = 2/2 checks passed**.

---

## 2) Connect to the VM

### A) Connect to a Linux EC2 (SSH)
1. From **EC2 → Instances**, select your instance → **Connect** → **SSH client** tab.
2. Open your terminal in the folder with your **.pem** key.
3. Fix key permissions (first time only):
   ```bash
   chmod 400 your-key.pem
   ```
4. SSH in (replace values shown in console):
   ```bash
   ssh -i your-key.pem ec2-user@<Public-IP-or-DNS>
   ```
   - Amazon Linux user: `ec2-user`
   - Ubuntu user: `ubuntu`

### B) Connect to a Windows EC2 (RDP)
1. In **EC2 → Instances**, select your instance → **Connect** → **RDP client** tab.
2. Click **Get password** → upload your **.pem** → **Decrypt password**.
3. Download the **RDP file** or open **Remote Desktop Connection**:
   - Computer: `<Public-IP-or-DNS>`
   - Username: `Administrator`
   - Password: (the decrypted value)

> If the connection fails, confirm the **security group** allows the port (22 for SSH, 3389 for RDP) from your current IP, and that your **instance has a public IP**.

---

## 3) Post‑Launch Setup (Optional but Recommended)

- **Update packages/OS**
  - Linux (Amazon Linux/Ubuntu):
    ```bash
    sudo yum update -y    # Amazon Linux
    # or
    sudo apt update && sudo apt upgrade -y   # Ubuntu
    ```
  - Windows: run **Windows Update**
- **Install tools**
  - Linux example:
    ```bash
    sudo dnf install -y git
    # or on Ubuntu
    sudo apt install -y git
    ```
  - Windows: install **Chocolatey** / **winget** to add tools quickly
- **Create a non‑root user** (Linux):
  ```bash
  sudo adduser devuser
  sudo usermod -aG wheel devuser    # Amazon Linux (sudo group)
  sudo su - devuser
  ```
- **Harden security**: restrict inbound rules to **My IP**, disable password SSH, rotate keys, enable SSM (AWS Systems Manager).

---

## 4) Stop / Start / Terminate

- **Stop**: VM is off (you keep the EBS disk). **Billed for storage**, not compute.
- **Start**: boots the same VM again (new public IP unless you use Elastic IP).
- **Terminate**: permanently deletes the VM (and EBS if “Delete on termination” is enabled).

**How to do it:** EC2 → Instances → select → **Instance state** → choose **Stop**, **Start**, or **Terminate**.

---

## 5) Troubleshooting

- **Status checks stuck**: wait a few minutes; if failing, try **Reboot** or check system logs.
- **Timeouts**: security group or local firewall blocking 22/3389; confirm **My IP** is correct.
- **No public IP**: enable in the instance’s **Networking** → assign **Elastic IP** if needed.
- **Key mismatch**: you must use the **same .pem** you created at launch; if lost, create a new instance.

---

## (Bonus) One‑Command Launch via AWS CLI

> Requires the **AWS CLI** configured with a profile and SSH key already created.

```bash
# Example for Amazon Linux 2023 in us-east-1
AMI_ID=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=al2023-ami-*-x86_64" "Name=state,Values=available" \
  --query 'Images | sort_by(@,&CreationDate)[-1].ImageId' --output text)

aws ec2 run-instances \
  --image-id $AMI_ID \
  --instance-type t3.micro \
  --key-name your-key-name \
  --security-group-ids sg-xxxxxxxx \
  --subnet-id subnet-xxxxxxxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=portfolio-ec2-vm}]'
```

---

## Repo Ideas

- Include this guide as `README.md`
- Add a `/scripts` folder with init scripts (e.g., install Docker, Git, Python)
- Add an **architecture diagram** png (optional)
- Add a **CLEANUP.md** with stop/terminate steps to avoid surprise charges

---

## Credits

Created by **Zahir Haynes** for learning/cloud portfolio. Feel free to fork and adapt.
