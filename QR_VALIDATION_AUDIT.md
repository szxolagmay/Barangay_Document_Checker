# QR Code Validation & Audit Logging System

## Overview
The Barangay Document Checker now includes comprehensive QR code validation with detailed audit logging for security monitoring and fraud prevention.

## QR Validation Process

### 1. Frontend Processing (DocuCheck.tsx)
- **Image Upload**: Users upload QR code images through the web interface
- **QR Decoding**: Uses jsQR library to extract hash from QR code image
- **Validation Request**: Sends extracted hash to backend API for verification
- **Result Display**: Shows validation results with detailed feedback messages

### 2. Backend Validation (server.js)
- **Hash Verification**: Searches across all document tables (indigency, clearance, business_permit)
- **Database Lookup**: Parallel queries to find matching hash codes
- **Response Generation**: Returns detailed document information if valid
- **Audit Logging**: Automatically logs all verification attempts

## Audit Logging Coverage

### 1. User Authentication Monitoring
```javascript
// Successful Login
logAuditEntry('Login', null, null, null, userId, userName, userRole, 'Success');

// Failed Login
logAuditEntry('Login', null, null, null, 0, 'Unknown User', 'Unknown', 'Failed', 'Invalid Credentials');
```

### 2. Document Issuance Tracking
```javascript
// Successful Document Creation
logAuditEntry('Document Issuance', documentId, documentType, 'System', userId, userName, userRole, 'Success');

// Failed Document Creation
logAuditEntry('Document Issuance', null, documentType, 'System', userId, userName, userRole, 'Failed', 'Database Error');
```

### 3. QR Verification Monitoring
```javascript
// Successful QR Verification
logAuditEntry('QR Verification', documentId, documentType, 'QR Upload', 0, 'Web User', 'Public User', 'Success');

// Failed QR Verification - Hash Not Found
logAuditEntry('QR Verification', null, null, 'QR Upload', 0, 'Web User', 'Public User', 'Failed', 'Hash not found in database');

// Failed QR Verification - No Hash Provided
logAuditEntry('QR Verification', null, null, 'QR Upload', 0, 'Web User', 'Public User', 'Failed', 'No hash provided');

// Failed QR Verification - Database Error
logAuditEntry('QR Verification', null, null, 'QR Upload', 0, 'Web User', 'Public User', 'Failed', 'Database error during validation');
```

## Error Scenarios Covered

### Frontend Error Handling
1. **No Image Uploaded**: Clear message prompting user to upload QR code
2. **QR Decode Failure**: Handles cases where no valid QR code is found in image
3. **Network Errors**: Graceful fallback to mock validation when backend is unavailable
4. **Invalid QR Format**: Proper error messages for malformed QR codes

### Backend Error Logging
1. **Missing Hash Parameter**: Logs and responds with appropriate error
2. **Database Connection Issues**: Logs database errors and provides fallback response
3. **Hash Not Found**: Logs unsuccessful verification attempts
4. **Invalid Document References**: Handles cases where document IDs don't match

## API Endpoints

### QR Validation
```
POST /api/validate-qr
Body: { "hash": "document_hash_code" }
Response: { "isValid": boolean, "message": string, "documentType": string, "document": object }
```

### Audit Log Retrieval
```
GET /api/audit-logs
Response: Array of log entries with timestamps, actions, and details
```

### Test QR Logging (Development)
```
POST /api/test-qr-logging
Body: { "scenario": "missing_hash|invalid_hash|valid_clearance|database_error" }
Response: { "message": "Logged: [scenario description]" }
```

### Audit Summary
```
GET /api/audit-summary
Response: { "summary": [{ "action_type": string, "status": string, "count": number }] }
```

## Database Schema

### log_entries Table
```sql
CREATE TABLE log_entries (
  id INT PRIMARY KEY AUTO_INCREMENT,
  timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
  action_type VARCHAR(50),
  document_id INT,
  document_type VARCHAR(100),
  checker_method VARCHAR(50),
  user_id INT,
  user_name VARCHAR(100),
  user_role VARCHAR(50),
  status VARCHAR(20),
  failure_reason TEXT
);
```

## Security Features

### 1. Fraud Detection
- All QR verification attempts are logged with timestamps
- Failed verifications include specific failure reasons
- Pattern analysis can identify suspicious verification attempts

### 2. Access Monitoring
- All user logins (successful and failed) are tracked
- Document issuance is monitored for unauthorized access
- User roles and permissions are logged for audit trails

### 3. Data Integrity
- Hash verification ensures document authenticity
- Comprehensive error logging helps identify system issues
- Backup validation modes ensure system reliability

## Usage Examples

### Testing QR Validation Logging
1. Start the server: `node server.js`
2. Upload a QR code image in DocuCheck
3. Check audit logs in the admin panel
4. View detailed failure reasons for invalid QR codes

### Monitoring Security Events
1. Access `/audit` page in admin panel
2. Filter by action type (Login, Document Issuance, QR Verification)
3. Review failure reasons for security analysis
4. Export logs for compliance reporting

## Benefits

1. **Comprehensive Monitoring**: All user activities are tracked automatically
2. **Fraud Prevention**: Invalid QR codes and suspicious patterns are logged
3. **Security Auditing**: Complete audit trail for compliance requirements
4. **Error Debugging**: Detailed error logging helps identify system issues
5. **User Experience**: Clear feedback messages guide users through validation process