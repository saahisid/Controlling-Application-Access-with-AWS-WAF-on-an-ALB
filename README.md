# Controlling Application Access with AWS WAF on an ALB

This project demonstrates how to host **two web applications** on a single EC2 instance behind an **Application Load Balancer (ALB)** and control access using **AWS WAF**.

---

## 📌 Plan of Action
The following steps describe how to implement this practice setup:

### 1. Provision EC2 Instance
- Launch an **Ubuntu** or **Amazon Linux** EC2 instance.
- Connect via **SSH** using your key pair.
- Install **Apache2 web server**.

### 2. Deploy Applications
- Create two web applications: **Website1** and **Website2**.
- Configure Virtual Hosts to serve:
  - `/website1 → Website1 application`
  - `/website2 → Website2 application`
- Add sample HTML content to each for testing.

### 3. Configure Load Balancer (ALB)
- Create an **Application Load Balancer** in the same VPC.
- Create a **Target Group** and register the EC2 instance.
- Configure **Listener Rules** to forward traffic to the target group.
- Test ALB routing to ensure requests reach the EC2 instance.

### 4. Set Up AWS WAF
- Create a **Web ACL** and attach it to the ALB.
- Create **IP Sets** for User1 and User2.
- Define rules:
  - **User1 → Block /website1, Allow /website2**
  - **User2 → Block /website2, Allow /website1**
- Verify that **block rules take priority** over allow rules.

### 5. Configure Security Groups & Networking
- Allow only:
  - **HTTP (80)**
  - **HTTPS (443)**
  - **SSH (22)**
- Ensure proper **VPC + subnet routing** for ALB → EC2 communication.

### 6. Testing & Validation
- Access `/website1` and `/website2` from **User1 and User2 IPs**.
- Validate:
  - **Blocked requests → 403 Forbidden**
  - **Allowed requests → Website loads correctly**

---

## ⚙️ Implementation Details

### EC2 Setup & Application Deployment
1. Launch a **Ubuntu Server 24.04 LTS** instance.
2. Configure security group to allow **SSH (22)** and **HTTP (80)**.
3. In the **Advanced Section**, add the following script as **user data**:

### Deploy Applications on EC2

Run the following script on your EC2 instance to install Apache, create two applications (`/website1` and `/website2`), and set up HTML files:

```bash
#!/bin/bash 
# Update system 
sudo apt update -y
sudo apt upgrade -y

# Install Apache
sudo apt install apache2 -y

# Enable Apache
sudo systemctl enable apache2
sudo systemctl start apache2

# Create application directories
sudo mkdir -p /var/www/html/website1
sudo mkdir -p /var/www/html/website2

# Website1 HTML
cat <<EOF | sudo tee /var/www/html/website1/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Website1 Application</title>
</head>
<body>
    <h1>Welcome to Website1 Application</h1>
    <p>Hostname: $(hostname)</p>
    <p>Server IP: $(hostname -I | awk '{print $1}')</p>
</body>
</html>
EOF

# Website2 HTML
cat <<EOF | sudo tee /var/www/html/website2/index.html
<!DOCTYPE html>
<html>
<head>
    <title>Website2 Application</title>
</head>
<body>
    <h1>Welcome to Website2 Application</h1>
    <p>Hostname: $(hostname)</p>
    <p>Server IP: $(hostname -I | awk '{print $1}')</p>
</body>
</html>
EOF

# Restart Apache
sudo systemctl restart apache2
---

✅ This will install Apache, create both applications, and start the web server.

### Verify Applications
Now, open your browser and test:

- `http://<Public-IP>/website1`  
- `http://<Public-IP>/website2`






