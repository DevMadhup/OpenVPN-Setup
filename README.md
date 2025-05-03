
# Secure AWS Private Server Access Using OpenVPN

---

## Use Case Overview

You want to host a **private EC2 instance** (e.g., a backend app, database, or admin tool) that is **not accessible to the public**. To access it securely, you'll use an **OpenVPN server** on AWS to create a private, encrypted tunnel into your AWS environment.

---

## Architecture Diagram (Text Representation)

```
[Your Laptop] 
    |
    | OpenVPN (encrypted tunnel)
    |
[AWS OpenVPN EC2 Instance (public IP)]
    |
    | (internal AWS network)
    |
[Private EC2 Server (no public IP)]
```

---

## Step-by-Step Setup

### STEP 1: Create the OpenVPN EC2 Server

1. **Launch an EC2 instance**
   - **AMI**: Ubuntu Server 22.04
   - **Type**: t2.micro
   - **Key Pair**: Create or use an existing one (e.g., `vpn-key.pem`)
   - **Security Group**: Allow:
     - UDP **1194** (OpenVPN)
     - TCP **22** (SSH)

2. **SSH into the instance**
```bash
ssh -i vpn-key.pem ubuntu@<openvpn-ec2-public-ip>
```

3. **Install OpenVPN with an installer script**
```bash
wget https://git.io/vpn -O openvpn-install.sh
chmod +x openvpn-install.sh
sudo ./openvpn-install.sh
```

4. **Follow the prompts**:
   - Public IP: accept default
   - Protocol: UDP
   - Port: 1194
   - Client name: `client1`

5. **Transfer the client configuration to your laptop**:
```bash
scp -i vpn-key.pem ubuntu@<openvpn-ec2-public-ip>:client1.ovpn .
```

---

### STEP 2: Create the Private EC2 Instance

1. **Launch a second EC2 instance**
   - **No Public IP**
   - Place it in the **same VPC** and **subnet** as the OpenVPN server
   - Use the **same key pair**
   - **Security Group**:
     - Allow **SSH (port 22)** from the **OpenVPN server's private IP** only

2. **Note the private IP** of this EC2 instance (e.g., `172.31.24.22`)

---

### STEP 3: Connect to VPN from Your Laptop

1. Install **OpenVPN Connect** (GUI) or **openvpn CLI** on your laptop
2. Import `client1.ovpn`
3. Click **Connect**

---

### STEP 4: SSH into the Private EC2 Instance

Once connected to VPN:
```bash
ssh -i vpn-key.pem ubuntu@<private-ec2-private-ip>
```

This works because your laptop is now inside the AWS network via VPN.

---

## Security Best Practices

- Store `vpn-key.pem` securely
- Disable root login and password authentication on all EC2 instances
- Allow SSH only from trusted IPs or the VPN server
- Rotate VPN keys periodically

---

## Result

You now have:
- A **secure VPN tunnel** to your AWS environment
- A **private EC2 server** accessible only after connecting via VPN
- An infrastructure that's safe from external scanning or attack
