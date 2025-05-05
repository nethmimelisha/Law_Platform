# AWS RDS Setup Guide for Legal Services Platform

This guide provides step-by-step instructions for setting up an AWS RDS MySQL instance for your Legal Services Platform.

## Prerequisites

- AWS account with appropriate permissions
- AWS CLI installed and configured (optional, but helpful)
- Basic knowledge of AWS services and networking

## Steps to Create and Configure RDS Instance

### 1. Create a MySQL RDS Instance

1. Log in to the AWS Management Console
2. Navigate to the RDS dashboard
3. Click "Create database"
4. Choose the following options:
   - **Engine type**: MySQL
   - **Version**: 8.0 or later
   - **Templates**: Production or Dev/Test based on your needs
   - **DB instance identifier**: `law-platform-db` (or your preferred name)
   - **Credentials**: Set a secure master username and password
   - **DB instance class**: Select based on your expected workload 
     - For development: `db.t3.small` is sufficient
     - For production: Consider `db.t3.medium` or higher
   - **Storage**: 
     - Type: General Purpose SSD (gp2)
     - Allocated: 20 GB minimum (increase as needed)
     - Enable storage autoscaling with a maximum of 100 GB
   - **Availability & durability**: 
     - Dev/Test: Single DB instance
     - Production: Multi-AZ deployment
   - **VPC, subnet, and security**: 
     - Use default VPC or create a new one
     - Create a new DB subnet group or use existing
     - Public access: No (for security)
   - **VPC security group**: Create new or use existing
   - **Database authentication**: Password authentication
   - **Monitoring**: Enable Enhanced monitoring (recommended for production)
   - **Backup**: 
     - Enable automated backups
     - Retention period: 7 days (or as needed)
     - Backup window: Select No preference or choose specific time
   - **Maintenance**: 
     - Enable auto minor version upgrade
     - Maintenance window: No preference or choose specific time

5. Click "Create database"

### 2. Configure Security Group

1. Navigate to the EC2 dashboard > Security Groups
2. Find the security group associated with your RDS instance
3. Edit inbound rules:
   - Add a rule: MySQL/Aurora (3306), Source: Your application server IP
   - For development with local machine: Add your IP address
   - Never open to 0.0.0.0/0 (all traffic)

### 3. Create Database and User

1. Connect to your RDS instance using a MySQL client:
   ```bash
   mysql -h your-rds-endpoint.rds.amazonaws.com -u admin -p
   ```

2. Create the database:
   ```sql
   CREATE DATABASE law_platform CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

3. Create an application user (instead of using the master user):
   ```sql
   CREATE USER 'law_app_user'@'%' IDENTIFIED BY 'your-secure-password';
   GRANT ALL PRIVILEGES ON law_platform.* TO 'law_app_user'@'%';
   FLUSH PRIVILEGES;
   ```

### 4. Update Environment Variables

Update your `.env` file with the RDS connection details:

```
# Database Configuration
DB_HOST=your-rds-endpoint.rds.amazonaws.com
DB_USERNAME=law_app_user
DB_PASSWORD=your-secure-password
DB_DATABASE=law_platform
DB_PORT=3306
DB_DIALECT=mysql
```

### 5. Initialize Database Tables

Run the initialization script to create your database tables:

```bash
cd law-platform-server
node src/scripts/initDb.js
```

Or set `SYNC_DB=true` in your `.env` file temporarily and start the server once.

### 6. Security Best Practices

1. **Encryption**:
   - Enable encryption at rest for your RDS instance
   - Use SSL/TLS for connections:
     ```
     DB_SSL=true
     ```

2. **Access Control**:
   - Restrict security group access to necessary IPs only
   - Use IAM database authentication for production if possible
   - Regularly rotate database passwords

3. **Monitoring**:
   - Set up CloudWatch alarms for:
     - High CPU utilization
     - Storage space
     - Database connections
     - Free memory

4. **Backup Strategy**:
   - Automated backups with appropriate retention
   - Consider point-in-time recovery for production databases
   - Test restoration procedures periodically

5. **Cost Management**:
   - Use appropriate instance type for your workload
   - Consider stopping dev/test instances when not in use
   - Enable storage autoscaling with reasonable limits

## Troubleshooting Connection Issues

If you have trouble connecting to your RDS instance:

1. Verify security group inbound rules allow traffic from your IP
2. Check that your database user has correct permissions
3. Ensure you're using the correct endpoint, username, and password
4. For VPC issues, check that subnet configurations allow outbound internet access
5. If using an EC2 instance to connect, ensure both instances are in the same VPC or have proper routes

## Migrating to Production

When moving from development to production:

1. Use a Multi-AZ deployment for high availability
2. Implement a read replica for read-heavy workloads
3. Set up automated snapshots with appropriate retention
4. Consider using AWS Secrets Manager for credential management
5. Implement a more robust monitoring solution with CloudWatch
6. Set up alarms for critical metrics
