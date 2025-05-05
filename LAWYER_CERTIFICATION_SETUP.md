# Lawyer Certification System Setup

This document outlines the implementation of the Lawyer Certification System and how to properly set it up.

## Components Added

1. **Backend Components**:
   - `LawyerCertification` model for storing certification data
   - File upload middleware using multer
   - Migration script for creating the certification table
   - Enhanced auth registration endpoint to handle file uploads
   - Admin-specific endpoints for reviewing and managing certifications

2. **Frontend Components**:
   - Enhanced registration form for file uploads
   - Registration pending page for lawyers
   - File type selection during upload
   - Integration with backend API for certificate upload

## Setup Instructions

### Database Migration

To set up the LawyerCertification table in your database:

1. Ensure your database is running and properly configured in `.env` file
2. Run the migration script:
   ```
   node src/scripts/runMigrations.js
   ```

Alternatively, if you prefer to use Sequelize's sync method:

1. Add the new model to your models index file
2. Run server with sync enabled

### File Storage

The system is configured to store certification files in the `uploads/certifications` directory. Make sure this directory exists and has proper permissions:

```
mkdir -p uploads/certifications
chmod 755 uploads/certifications
```

## Testing

To test the lawyer certification system:

1. Register as a lawyer through the application
2. Upload certification documents during registration
3. Log in as an admin to view and approve certifications
4. Verify that lawyers are properly marked as verified after approval

## API Endpoints

### User Registration with Certification
```
POST /api/auth/register
Content-Type: multipart/form-data

Fields:
- email: string
- password: string
- firstName: string
- lastName: string
- role: "lawyer"
- certificationFiles: file[]
- certificationTypes: string[] (license, degree, barAssociation, identity, other)
```

### Admin Certification Management
```
GET /api/admin/certifications - Get all certifications
GET /api/admin/certifications/:id - Get specific certification
POST /api/admin/certifications/:id/review - Approve or reject certification
```

### User Certification Management
```
GET /api/certifications - Get user's certifications
POST /api/certifications - Upload additional certification
DELETE /api/certifications/:id - Delete a certification
```

## Implementation Notes

1. Certifications are stored with a status of "pending", "approved", or "rejected"
2. Lawyers are not verified until an admin approves their certification
3. The system maintains a detailed history of certification reviews
4. Files are uniquely named using UUIDs to prevent conflicts
