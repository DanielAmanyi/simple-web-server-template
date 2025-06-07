#!/bin/bash
# Switch to root user
sudo su

# Fetch IMDSv2 Token
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Retrieve instance metadata using IMDSv2
PRIVATE_IP=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/local-ipv4)
HOSTNAME=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/hostname)

# Update and install Apache (httpd)
yum update -y
yum install -y httpd.x86_64

# Start and enable Apache
systemctl start httpd.service
systemctl enable httpd.service

# Enable Server-Side Includes (SSI) in Apache
sed -i 's/#AddOutputFilter INCLUDES .shtml/AddOutputFilter INCLUDES .html/' /etc/httpd/conf/httpd.conf
sed -i 's/Options Indexes FollowSymLinks/Options Indexes FollowSymLinks Includes/' /etc/httpd/conf/httpd.conf

# Remove default Apache test page
rm -f /var/www/html/index.html /var/www/html/index.shtml

# Create the HTML file with instance details
cat << EOF > /var/www/html/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>High Availability in Cloud Computing - Amanyi Daniel Training</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            min-height: 100vh;
            background: linear-gradient(to bottom, #2c3e50, #3498db);
            color: white;
            display: flex;
            flex-direction: column;
        }
        header {
            text-align: center;
            padding: 2rem;
        }
        .amanyi-banner {
            background-color: #f39c12;
            color: #2c3e50;
            text-align: center;
            padding: 0.5rem;
            font-weight: bold;
        }
        main {
            flex-grow: 1;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 2rem;
        }
        .hostname, .private-ip {
            background-color: white;
            color: black;
            padding: 1rem 2rem;
            border-radius: 8px;
            font-size: 1.5rem;
            margin-bottom: 1rem;
        }
        .refresh-instruction {
            background-color: #e74c3c;
            color: white;
            padding: 0.5rem 1rem;
            border-radius: 4px;
            margin-bottom: 2rem;
            text-align: center;
        }
        .info {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-around;
            width: 100%;
            max-width: 1200px;
        }
        .info-section {
            flex-basis: 45%;
            margin-bottom: 2rem;
        }
        footer {
            text-align: center;
            padding: 1rem;
            background-color: rgba(0, 0, 0, 0.5);
        }
    </style>
</head>
<body>
    <div class="amanyi-banner">
        Training Provided by Amanyi Daniel
    </div>
    <header>
        <h1>High Availability in Cloud Computing</h1>
    </header>
    <main>
        <div class="hostname">
            Hostname: <!--#exec cmd="hostname -f" -->
        </div>
        <div class="private-ip">
            Private IP: $PRIVATE_IP
        </div>
        <div class="refresh-instruction">
            <strong>Refresh the page several times to see the Private IP change, demonstrating High Availability in action!</strong>
        </div>
        <div class="info">
            <div class="info-section">
                <h2>What is High Availability?</h2>
                <p>High Availability (HA) in cloud computing refers to the ability of a system to remain operational and accessible even in the face of component failures. It ensures that services remain available to users with minimal downtime, typically aiming for 99.9% uptime or higher.</p>
            </div>
            <div class="info-section">
                <h2>Benefits of High Availability</h2>
                <ul>
                    <li>Minimized downtime and service interruptions</li>
                    <li>Improved user experience and satisfaction</li>
                    <li>Increased business continuity</li>
                    <li>Enhanced disaster recovery capabilities</li>
                    <li>Better resource utilization</li>
                </ul>
            </div>
            <div class="info-section">
                <h2>Implementation Strategies</h2>
                <ul>
                    <li>Redundancy: Duplicate critical components</li>
                    <li>Load Balancing: Distribute traffic across multiple servers</li>
                    <li>Failover Systems: Automatic switching to backup systems</li>
                    <li>Data Replication: Maintain multiple copies of data across locations</li>
                    <li>Geographic Distribution: Deploy across multiple regions or availability zones</li>
                </ul>
            </div>
            <div class="info-section">
                <h2>Use Cases</h2>
                <ul>
                    <li>E-commerce platforms ensuring 24/7 availability</li>
                    <li>Financial services requiring constant uptime</li>
                    <li>Healthcare systems with critical patient data</li>
                    <li>Global content delivery networks</li>
                    <li>SaaS applications with service level agreements (SLAs)</li>
                </ul>
            </div>
            <div class="info-section">
                <h2>Scaling Strategies</h2>
                <ul>
                    <li>Vertical Scaling: Increasing the power of existing servers</li>
                    <li>Horizontal Scaling: Adding more servers to distribute load</li>
                    <li>Auto-scaling: Dynamically adjusting resources based on demand</li>
                    <li>Database Sharding: Partitioning data across multiple databases</li>
                    <li>Microservices Architecture: Breaking down applications into smaller, scalable services</li>
                </ul>
            </div>
            <div class="info-section">
                <h2>Monitoring and Management</h2>
                <p>Implementing high availability requires continuous monitoring and management. Cloud providers offer tools for real-time monitoring, automated alerts, and performance analytics to ensure systems remain highly available and quickly recover from any issues.</p>
            </div>
        </div>
    </main>
    <footer>
        <p>&copy; 2025 High Availability in Cloud Computing Demo | Amanyi Daniel Training</p>
    </footer>
</body>
</html>
EOF

# Set correct file permissions
chmod 644 /var/www/html/index.html
chown apache:apache /var/www/html/index.html

# Restart Apache to apply changes
systemctl restart httpd.service
