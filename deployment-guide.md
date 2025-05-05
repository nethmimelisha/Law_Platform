# Law Platform Deployment Guide

This document provides instructions for deploying the Law Platform service to production using AWS services.

## Prerequisites

- AWS Account with access to RDS, EC2, S3, and IAM
- Node.js (v14+) and npm installed on deployment machine
- Git installed on deployment machine

## AWS RDS Database Setup

1. **Create an RDS Instance**

   - Log in to AWS Console
   - Navigate to RDS service
   - Click "Create database"
   - Select MySQL (or your preferred database engine)
   - Choose "Production" template
   - Set up the following:
     - DB instance identifier: `law-platform-db`
     - Master username: `admin` (or your preferred username)
     - Master password: (create a secure password)
   - Configure instance:
     - DB instance class: db.t3.small (or appropriate for your expected traffic)
     - Storage: 20GB gp2 (increase as needed)
   - Configure connectivity:
     - VPC: Use your default VPC or create a new one
     - Subnet group: Create new DB subnet group
     - Public access: No (for security)
     - VPC security group: Create new (or use existing)
   - Additional configuration:
     - Initial database name: `law_platform`
     - Enable automated backups
     - Enable encryption
   - Click "Create database"

2. **Configure Security Group for RDS**

   - Navigate to the created RDS instance
   - Click on the VPC security group
   - Add inbound rule:
     - Type: MySQL/Aurora
     - Protocol: TCP
     - Port Range: 3306
     - Source: Your EC2 security group (or specific IP if deploying elsewhere)

## Server Deployment

### Option 1: Deploy to EC2

1. **Create EC2 Instance**

   - Navigate to EC2 service
   - Launch a new instance
   - Select Amazon Linux 2 or Ubuntu
   - Choose instance type (t2.micro for small-scale testing, t2.small or larger for production)
   - Configure security group to allow HTTP (80), HTTPS (443), and SSH (22)
   - Launch instance with a key pair

2. **Connect to EC2 Instance**

   ```bash
   ssh -i your-key.pem ec2-user@your-ec2-instance-ip
   ```

3. **Install Required Software**

   ```bash
   # Update system
   sudo yum update -y  # For Amazon Linux
   # OR
   sudo apt update && sudo apt upgrade -y  # For Ubuntu

   # Install Node.js
   curl -sL https://rpm.nodesource.com/setup_16.x | sudo bash -  # For Amazon Linux
   sudo yum install -y nodejs  # For Amazon Linux
   # OR
   curl -sL https://deb.nodesource.com/setup_16.x | sudo -E bash -  # For Ubuntu
   sudo apt install -y nodejs  # For Ubuntu

   # Install Git
   sudo yum install git -y  # For Amazon Linux
   # OR
   sudo apt install git -y  # For Ubuntu

   # Install PM2 for process management
   sudo npm install -g pm2
   ```

4. **Clone and Set Up the Application**

   ```bash
   git clone https://your-repo-url/law-platform-service.git
   cd law-platform-service/law-platform-server

   # Install dependencies
   npm install --production

   # Copy .env.production to .env
   cp .env.production .env

   # Edit the .env file with your AWS RDS details
   nano .env
   ```

5. **Set Up the Database**

   ```bash
   # Run migrations
   npx sequelize-cli db:migrate

   # Seed the database
   node src/scripts/seed.js
   ```

6. **Start the Server**

   ```bash
   # Start the application with PM2
   pm2 start src/server.js --name "law-platform-api"
   
   # Ensure PM2 starts on system boot
   pm2 startup
   pm2 save
   ```

7. **Configure Nginx (Optional for SSL and reverse proxy)**

   ```bash
   sudo yum install nginx -y  # For Amazon Linux
   # OR
   sudo apt install nginx -y  # For Ubuntu

   # Configure Nginx
   sudo nano /etc/nginx/conf.d/law-platform.conf
   ```

   Add the following configuration:

   ```nginx
   server {
       listen 80;
       server_name yourdomain.com www.yourdomain.com;

       location / {
           proxy_pass http://localhost:5000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

   ```bash
   # Test Nginx configuration
   sudo nginx -t

   # Restart Nginx
   sudo systemctl restart nginx

   # Allow Nginx through firewall
   sudo firewall-cmd --permanent --add-service=http
   sudo firewall-cmd --permanent --add-service=https
   sudo firewall-cmd --reload
   ```

8. **Set Up SSL with Let's Encrypt (Optional)**

   ```bash
   sudo yum install certbot python3-certbot-nginx -y  # For Amazon Linux
   # OR
   sudo apt install certbot python3-certbot-nginx -y  # For Ubuntu

   sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```

### Option 2: Deploy Using Docker (Recommended)

1. **Create a Dockerfile in the server directory**

   The Dockerfile is already included in the repository. If you need to modify it:

   ```bash
   nano Dockerfile
   ```

2. **Build and Push Docker Image**

   ```bash
   # Build the image
   docker build -t law-platform-api .

   # Tag the image for your repository
   docker tag law-platform-api your-repo/law-platform-api:latest

   # Push to repository (if using Docker Hub or ECR)
   docker login
   docker push your-repo/law-platform-api:latest
   ```

3. **Run the Docker Container**

   ```bash
   # Run the container
   docker run -d \
     --name law-platform-api \
     -p 5000:5000 \
     --env-file .env \
     your-repo/law-platform-api:latest
   ```

### Option 3: Deploy to AWS Elastic Beanstalk

1. **Install AWS CLI and EB CLI**

   ```bash
   pip install awscli
   pip install awsebcli
   ```

2. **Initialize Elastic Beanstalk Application**

   ```bash
   cd law-platform-server
   eb init

   # Follow the prompts to select region and create the application
   ```

3. **Create Environment and Deploy**

   ```bash
   eb create law-platform-production

   # Once created, you can deploy updates with
   eb deploy
   ```

## Client Deployment

1. **Update API Endpoint in Client**

   ```bash
   cd ../law-platform-client
   
   # Update the API base URL in .env file
   echo "REACT_APP_API_BASE_URL=https://your-api-domain.com/api" > .env.production
   ```

2. **Build the Client**

   ```bash
   npm install
   npm run build
   ```

3. **Deploy to S3 (Optional)**

   ```bash
   # Install AWS CLI
   pip install awscli
   
   # Configure AWS CLI
   aws configure
   
   # Create S3 bucket
   aws s3 mb s3://your-law-platform-bucket
   
   # Configure for static website hosting
   aws s3 website s3://your-law-platform-bucket --index-document index.html --error-document index.html
   
   # Set bucket policy for public read
   aws s3api put-bucket-policy --bucket your-law-platform-bucket --policy '{
     "Version": "2012-10-17",
     "Statement": [
       {
         "Sid": "PublicReadGetObject",
         "Effect": "Allow",
         "Principal": "*",
         "Action": "s3:GetObject",
         "Resource": "arn:aws:s3:::your-law-platform-bucket/*"
       }
     ]
   }'
   
   # Upload build to S3
   aws s3 sync build/ s3://your-law-platform-bucket
   
   # Create CloudFront distribution for HTTPS and better performance (optional)
   # This step is best done through the AWS Console
   ```

## Monitoring and Maintenance

### Setting Up Monitoring

1. **Set Up CloudWatch for RDS and EC2**

   - Navigate to CloudWatch in AWS Console
   - Create alarms for:
     - RDS CPU Utilization (> 80% for 5 minutes)
     - RDS Free Storage Space (< 10% for 5 minutes)
     - EC2 CPU Utilization (> 80% for 5 minutes)
     - EC2 Status Checks Failure

2. **Set Up Application Logging**

   ```bash
   # On EC2
   pm2 install pm2-logrotate
   pm2 set pm2-logrotate:max_size 10M
   pm2 set pm2-logrotate:retain 10
   ```

### Database Backups

1. **Configure Automated RDS Backups**

   - Navigate to your RDS instance in AWS Console
   - Select "Modify"
   - Under "Backup":
     - Set backup retention period (7-30 days recommended)
     - Set backup window to a low-traffic time

2. **Set Up Regular Database Dumps**

   ```bash
   # Create a backup script
   nano ~/backup_db.sh
   ```

   Add the following content:

   ```bash
   #!/bin/bash
   TIMESTAMP=$(date +"%Y%m%d-%H%M%S")
   BACKUP_DIR="/path/to/backups"
   
   # Create backup directory if it doesn't exist
   mkdir -p $BACKUP_DIR
   
   # Use mysqldump to create a backup
   mysqldump -h your-aws-rds-endpoint.rds.amazonaws.com -u admin -p'your-password' law_platform > $BACKUP_DIR/law_platform_$TIMESTAMP.sql
   
   # Compress the backup
   gzip $BACKUP_DIR/law_platform_$TIMESTAMP.sql
   
   # Delete backups older than 7 days
   find $BACKUP_DIR -name "law_platform_*.sql.gz" -mtime +7 -delete
   ```

   Make the script executable and schedule it:

   ```bash
   chmod +x ~/backup_db.sh
   
   # Add to crontab to run daily
   (crontab -l 2>/dev/null; echo "0 2 * * * /home/ec2-user/backup_db.sh") | crontab -
   ```

## Security Best Practices

1. **Keep Software Updated**

   ```bash
   # Regular updates
   sudo yum update -y  # For Amazon Linux
   # OR
   sudo apt update && sudo apt upgrade -y  # For Ubuntu
   
   # Update Node.js dependencies
   cd /path/to/law-platform-server
   npm audit fix
   ```

2. **Secure Your RDS Instance**

   - Use SSL/TLS for database connections
   - Use strong passwords
   - Limit network access to your application servers only
   - Rotate credentials regularly

3. **Implement Application Security**

   - Keep your JWT_SECRET secure and rotate it periodically
   - Implement rate limiting
   - Use HTTPS for all communications
   - Set secure HTTP headers
   - Sanitize all user inputs

## Troubleshooting

### Common Issues

1. **Connection to RDS fails**
   - Check security group rules
   - Verify credentials in .env file
   - Ensure the RDS instance is running

2. **Application crashes after deployment**
   - Check server logs: `pm2 logs`
   - Verify all environment variables are set correctly
   - Check disk space: `df -h`

3. **Slow queries or application performance**
   - Monitor RDS with CloudWatch
   - Check slow query logs
   - Consider scaling up your database instance

## Support

For additional support or questions, please contact the development team at support@yourdomain.com or open an issue in the project repository.
