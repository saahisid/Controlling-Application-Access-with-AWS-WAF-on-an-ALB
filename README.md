# Controlling-Application-Access-with-AWS-WAF-on-an-ALB

Plan of Action
The plan of action describes the step-by-step approach to implement the AWS practice of hosting two
applications with controlled access using ALB and WAF.
1. Provision EC2 Instance
○ Launch an Ubuntu or Amazon Linux EC2 instance.
○ Connect via SSH using the key pair.
○ Install Apache2 web server.
2. Deploy Applications
○ Create two web applications: Website1 and Website2.
○ Configure Virtual Hosts to serve:
■ /Website1 → Website1 application
■ /Website2 → Website2 application
○ Add basic content to each application for testing.
3. Configure Load Balancer (ALB)
○ Create an Application Load Balancer in the same VPC.
○ Set up a Target Group and register the EC2 instance.
○ Configure Listener rules to forward traffic to the Target Group.
○ Test ALB routing to ensure traffic reaches the EC2 applications.
4. Set Up AWS WAF
○ Create a Web ACL and attach it to the ALB.
○ Create IP Sets for User 1 and User 2.
○ Create WAF rules to enforce access restrictions:
■ User 1 → Block /Website1, Allow /Website2
■ User 2 → Block /Website2, Allow /Website1
○ Verify rule priority so that block rules are applied before allow rules.
5. Configure Security Groups & Networking
○ Allow only necessary ports: HTTP (80), HTTPS (443), SSH (22).
○ Ensure proper VPC and subnet routing for ALB → EC2 communication.
6. Testing & Validation
○ Access both applications from User 1 and User 2 IP addresses.
○ Confirm that the correct 403 Forbidden messages appear for blocked sites and
allowed sites load normally.

Implementation Details
EC2 Setup and Application Deployment
First, you need to create an instance. To do this, click on Launch Instance on the right-hand side.

Create a normal Ubuntu instance with a basic configuration. Give your instance a name under the “Name and tags” section. In the “Application and OS Images (Amazon Machine Image)” section, select the Ubuntu tab under Quick Start, and choose an Ubuntu Server 24.04 LTS AMI. 




You can either select the default VPC or create a custom VPC based on your requirements. In the security group settings, make sure to add rules for both SSH (port 22) and HTTP (port 80) to allow the necessary access.





In the Advanced section, make sure to add the script shown below. It will automatically install Apache and create two application directories — website1 and website2 — each with a sample HTML page.
#!/bin/bash 
# Update system 
sudo apt update -y
sudo apt upgrade -y
# Install Apache
sudo apt install apache2 -y
# Enable Apache at boot
sudo systemctl enable apache2
sudo systemctl start apache2
# Create application directories
sudo mkdir -p /var/www/html/website1
sudo mkdir -p /var/www/html/website2
# Create sample HTML for website1
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
# Create sample HTML for website2
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
# Restart Apache to apply changes
sudo systemctl restart apache2


After launching the instance, open your browser and enter the public IP followed by /website1 or /website2 (e.g., http://<Public-IP>/website2). If everything is set up correctly, you’ll see a welcome message confirming your application is running.

Set up a Target Group
After that, you need to create a target group to register your instance and route traffic through a load balancer.



After clicking on "Create target group", follow these steps:
Choose "Instance" as the target type.
Enter a name for your target group.
Select the VPC – if you have created a custom VPC, select that; otherwise, choose the default VPC.
Leave all other settings as the default.
Click "Next" to proceed.






After clicking "Next" on the target group setup page, you'll reach the "Register targets" section. Here, select the instance you want to include as a target, then click "Include as pending below" to add it. Once the instance appears in the pending list, click "Create target group", and then click "Continue" to proceed.

After creating the target group, you will see a confirmation message indicating that the target group was created successfully.







Load Balancer Configuration
Click on "Create Load Balancer" to begin setting up your load balancer.

Select “Application Load Balancer” to proceed with creating your load balancer.






After clicking on “Create ALB", you need to enter your ALB name, select the scheme, and choose the load balancer IP address type.

In the Network Mapping section, select the same VPC you used when creating the target group. Then, based on your preference, choose the availability zones and make sure to select at least one subnet in each zone for high availability.


You need to create a new security group for the Load Balancer. To do this, click on "Create a new security group" and configure the necessary inbound rules, such as allowing HTTP (port 80) to ensure web traffic can reach the load balancer.

While creating a new security group for the ALB, add inbound rules for both HTTP (port 80) and SSH (port 22) to allow web and remote access. After adding the rules, click on "Create security group" to proceed.

In the Listener configuration section, add the target group you created earlier. Leave all other settings as default, then proceed to the next step.

You will see a confirmation message indicating that the Load Balancer has been successfully created.


WAF Configuration and Rules

After successfully creating the Application Load Balancer (ALB), you need to configure AWS WAF (Web Application Firewall). In AWS WAF, start by creating an IP Set that includes the IP addresses you want to allow—for example, the IPs used by User One and User Two. This IP Set will help you control and restrict access based on IP addresses.












After clicking "Create IP Set", provide a name for the IP Set and select the same region where you created your Load Balancer and EC2 instance. Then, enter the IP addresses you want to allow (e.g., for User One and User Two). Make sure to add /32 at the end of each IP address (e.g., 192.0.2.10/32) to ensure that only those specific IPs can access the application, preventing access from any other users.


In the same way, you can create an IP Set for User 2. Click on "Create IP Set" and enter the IP address of User 2, followed by /32 (e.g., 110.224.121.145/32), to allow access only from that specific IP address.


After successfully creating IP sets for both User 1 and User 2, the next step is to create a Web ACL (Web Access Control List) in AWS WAF. 



After opening Web ACL in AWS WAF, click on "Create Web ACL" to begin setting up the access control for your application. This will allow you to define rules that control access to your application through the ALB based on the IP sets you created.



Select "Regional resources", then enter a name for your Web ACL. Next, click "Add AWS resources" to associate the Web ACL with your Application Load Balancer (ALB).

In the AWS resources type section, select "Application Load Balancer (ALB)", then choose the specific ALB you want to associate with the Web ACL. After selecting it, click on "Add" to attach the resource.



After adding the AWS resource, click Next. In the Rules section, click on Add rules, then choose Add my own rules and rule groups to create custom rules for your Web ACL.






In the Add my own rules and rule groups section, select Rule builder and give your rule a meaningful name to identify it easily.



In the If a request statement, select “Matches all the statements (AND)” to ensure that all conditions within the rule must be met for the rule to apply.


 Statement 1 
Inspect → URI path
This means AWS WAF will look at the URI path of the request.
Example: If the request is https://example.com/website1/home, the URI path is /website1/home.


Match type → Starts with string
WAF will check if the URI path starts with a specific string.


String to match → /website1/
This tells WAF to look for requests where the path starts with /website1/.
Example:
  /website1/home → Match
 /app/website1/ → Not a match






 Statement 2 
Inspect → Originates from an IP address in
This means WAF will check the source IP address of the request.


IP set → user1
Here, user1 is an IP set you created earlier (a collection of IP addresses or ranges).
Example: 192.168.1.10, 203.0.113.0/24, etc.


IP address to use as the originating address
Two options:
✅ Source IP address → WAF checks the actual IP address of the client making the request.
❌ IP address in header → WAF would check IPs inside headers like X-Forwarded-For (useful if traffic comes through proxies/CDNs, but risky since headers can be spoofed).
In your case, Source IP address is selected, which is safer.


Putting this together with your Statement 1 (URI Path rule)

Statement 1 checked if the request path starts with /website1/.
Statement 2 checks if the request comes from an IP in user1.

The statement will only take effect if all conditions are true—meaning traffic will be allowed only when both condition in the rule is met.



In the Action section, select Allow so that only the users matching your rule will be permitted to access the URL, ensuring restricted access exclusively to those users.



After clicking Add rule for User 1, repeat the same process for User 2, but configure the rule to allow access only to website1 and Keep the default action set to Block, which means that whenever an unknown user tries to access the application, their request will be automatically blocked.


Click Next and set the priority of your rules so that they are evalWebsite1ed first before any other rules.



Click Next twice to proceed to the Review page, then click Create Web ACL to finalize and activate your settings.






After successfully creating the Web ACL, go to your Load Balancer, copy its DNS URL, and paste it into your browser. Then, add /website1 at the end of the URL to access the first website (e.g., http://your-load-balancer-dns/website1).



Testing & Validation
User 1 Access Scenario
The website is successfully blocked for User 1, meaning that User 1 no longer has access as per the rules you configured in the Web ACL.


User 1 is successfully able to access Website 2, confirming that the access rules are working correctly for that user.




User 2 Access Scenario

User 2 is also successfully able to access Website 1, showing that the access permissions are correctly applied.



User 2 is also successfully blocked from accessing Website 2, confirming that the access restrictions are functioning properly.


