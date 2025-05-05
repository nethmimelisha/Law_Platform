# Legal Services Platform

A comprehensive platform connecting clients with legal professionals, offering secure messaging, appointment scheduling, and case management.

## Project Structure

This project consists of two main components:

- **law-platform-client**: React frontend application
- **law-platform-server**: Node.js backend API with MySQL/AWS RDS database

## Features

- User authentication and role-based access control
- Lawyer directory with search, filtering, and ratings
- Secure messaging system between clients and lawyers
- Appointment scheduling and management
- Case tracking and documentation
- Admin dashboard for platform oversight
- Review and rating system for lawyers

## Backend Setup

### Prerequisites

- Node.js (v14 or higher)
- MySQL database (local or AWS RDS)
- AWS account (for production deployment)

### Installation

1. Navigate to the server directory:
   ```bash
   cd law-platform-server
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Set up environment variables:
   - Copy `.env.example` to `.env`
   - Update the variables with your specific configuration

4. Initialize the database:
   ```bash
   npm run db:sync
   ```

5. Start the development server:
   ```bash
   npm run dev
   ```

### Environment Variables

The following environment variables need to be set in the `.env` file:

```
# Server Configuration
NODE_ENV=development
PORT=5000
SYNC_DB=true

# JWT Configuration
JWT_SECRET=your_jwt_secret_key
JWT_EXPIRES_IN=30d

# Database Configuration
DB_HOST=localhost
DB_USERNAME=root
DB_PASSWORD=your_password
DB_DATABASE=law_platform
DB_PORT=3306
DB_DIALECT=mysql

# AWS Configuration (Production)
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
S3_BUCKET=your_bucket_name
```

## Frontend Setup

### Prerequisites

- Node.js (v14 or higher)
- npm or yarn

### Installation

1. Navigate to the client directory:
   ```bash
   cd law-platform-client
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Create `.env` file for API URL:
   ```
   REACT_APP_API_URL=http://localhost:5000/api
   ```

4. Start the development server:
   ```bash
   npm start
   ```

## AWS RDS Setup (Production)

1. Create a MySQL RDS instance in the AWS console
2. Configure VPC and security groups to allow connections
3. Update the `.env` file with RDS endpoint and credentials
4. Set `SYNC_DB=true` temporarily to initialize tables
5. After first successful connection, set `SYNC_DB=false`

## API Documentation

The API endpoints are organized into the following resources:

- `/api/auth` - Authentication and user profile management
- `/api/users` - User administration (admin only)
- `/api/lawyers` - Lawyer directory and profiles
- `/api/messages` - Communication system
- `/api/appointments` - Consultation scheduling
- `/api/reviews` - Ratings and testimonials

For detailed API documentation, see [API.md](API.md)

## Future Enhancements

- Integration of a chatbot using RAG (Retrieval-Augmented Generation)
- Document management system with OCR
- Integrated billing and payment system
- Mobile application

## License

This project is proprietary and confidential.
