#!/bin/bash
# Install Apache
yum update -y
yum install -y httpd aws-cli

# Enable and start Apache
systemctl enable httpd
systemctl start httpd

# Get instance metadata
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
PUBLIC_IP=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
SUBNET_ID=$(aws ec2 describe-instances --instance-ids "$INSTANCE_ID" --query "Reservations[0].Instances[0].SubnetId" --output text --region $(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | grep region | awk -F\" '{print $4}'))

# Create the index.html page
cat <<EOF > /var/www/html/index.html
<!DOCTYPE html>
<html>
  <head>
    <title>EC2 Info</title>
    <style>
      body { font-family: Arial; background-color: #f4f4f4; text-align: center; padding-top: 50px; }
      h1 { color: #2e6da4; }
      p { font-size: 20px; color: #333; }
    </style>
  </head>
  <body>
    <h1>✅ I'm up!</h1>
    <p><strong>Instance ID:</strong> $INSTANCE_ID</p>
    <p><strong>Public IP:</strong> $PUBLIC_IP</p>
    <p><strong>Subnet ID:</strong> $SUBNET_ID</p>
  </body>
</html>
EOF
