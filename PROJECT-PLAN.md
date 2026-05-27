# KEES-Server2: Secure Password Manager Backend

## Project Overview
KEES-Server2 is a secure backend server for storing and managing passwords across multiple platforms including bank accounts, social media, credit/debit cards, cloud service providers (GitHub, Google, AWS, etc.), and any other online services.

---

## 1. Core Features & Requirements

### 1.1 User Management
- **User Authentication**
  - User registration with email validation
  - Secure login with JWT or session-based authentication
  - Password reset functionality with email verification
  - Two-factor authentication (2FA) support using TOTP/OTP
  - Account lockout after failed login attempts

### 1.2 Password Storage
- **Categories/Vaults**
  - Bank Accounts
  - Social Media Accounts
  - Credit/Debit Cards
  - Cloud Services (GitHub, Google, AWS, etc.)
  - Email Accounts
  - Custom Categories

- **Password Entry Fields**
  - Service name/type
  - Username/Email
  - Password (encrypted)
  - URL/Endpoint
  - PIN/Security Code (encrypted)
  - Card Details (encrypted):
    - Card Number
    - CVV
    - Expiry Date
    - Cardholder Name
  - Additional Notes (encrypted)
  - Tags for organization
  - Last Updated timestamp
  - Creation timestamp

### 1.3 Security Features
- **Encryption**
  - End-to-end encryption using AES-256-GCM
  - Master password derivation using PBKDF2
  - Secure key storage and management
  - Salted password hashing (bcrypt) for user passwords

- **Access Control**
  - User-specific data isolation
  - Audit logs for sensitive operations
  - Session management with token expiration
  - IP-based access logging (optional)

### 1.4 Data Management
- **CRUD Operations**
  - Create new password entries
  - Read/Retrieve password entries
  - Update existing entries
  - Delete entries with soft delete option (recoverable)
  - Search and filter across stored passwords

- **Backup & Recovery**
  - Encrypted backup functionality
  - Data export in encrypted format
  - Bulk import from CSV/JSON (with encryption)
  - Recovery mechanism for deleted entries

---

## 2. Technical Architecture

### 2.1 Technology Stack
```
Backend Framework:     Express.js (Node.js)
Database:             PostgreSQL (or MongoDB as alternative)
Encryption:           crypto (Node.js built-in), bcryptjs
Authentication:       JWT + Refresh Tokens
Email Service:        Nodemailer / SendGrid
Environment Config:   dotenv
API Documentation:    Swagger/OpenAPI
Testing:              Jest, Supertest
Logging:              Winston, Morgan
Rate Limiting:        express-rate-limit
```

### 2.2 Database Schema (PostgreSQL)
```
Tables:
├── users
│   ├── id (UUID, PK)
│   ├── email (UNIQUE, indexed)
│   ├── password_hash (bcrypt)
│   ├── master_key_salt
│   ├── encryption_key_hash
│   ├── two_fa_enabled
│   ├── two_fa_secret
│   ├── is_active
│   ├── created_at
│   ├── updated_at
│   └── last_login
│
├── passwords
│   ├── id (UUID, PK)
│   ├── user_id (FK -> users)
│   ├── service_name
│   ├── category
│   ├── username_email (encrypted)
│   ├── password (encrypted)
│   ├── url (encrypted, optional)
│   ├── notes (encrypted, optional)
│   ├── tags (JSON array)
│   ├── is_deleted (soft delete)
│   ├── created_at
│   ├── updated_at
│   └── deleted_at (nullable)
│
├── card_details
│   ├── id (UUID, PK)
│   ├── user_id (FK -> users)
│   ├── card_type (Visa, Mastercard, Amex, etc.)
│   ├── cardholder_name (encrypted)
│   ├── card_number (encrypted)
│   ├── cvv (encrypted)
│   ├── expiry_date (encrypted)
│   ├── bank_name (encrypted)
│   ├── notes (encrypted, optional)
│   ├── is_deleted (soft delete)
│   ├── created_at
│   ├── updated_at
│   └── deleted_at (nullable)
│
├── audit_logs
│   ├── id (UUID, PK)
│   ├── user_id (FK -> users)
│   ├── action (create, read, update, delete, export, import)
│   ├── resource_type (password, card, user)
│   ├── resource_id
│   ├── ip_address
│   ├── user_agent
│   ├── timestamp
│   └── metadata (JSON)
│
├── sessions
│   ├── id (UUID, PK)
│   ├── user_id (FK -> users)
│   ├── token_hash
│   ├── refresh_token_hash
│   ├── expires_at
│   ├── ip_address
│   ├── created_at
│   └── revoked_at (nullable, for logout)
│
└── two_fa_attempts
    ├── id (UUID, PK)
    ├── user_id (FK -> users)
    ├── attempt_count
    ├── last_attempt_at
    └── locked_until (nullable)
```

---

## 3. API Endpoints

### 3.1 Authentication Endpoints
```
POST   /api/auth/register              - Register new user
POST   /api/auth/login                 - Login user
POST   /api/auth/logout                - Logout user
POST   /api/auth/refresh               - Refresh JWT token
POST   /api/auth/verify-email          - Verify email address
POST   /api/auth/forgot-password       - Initiate password reset
POST   /api/auth/reset-password        - Complete password reset
POST   /api/auth/2fa/enable            - Enable 2FA
POST   /api/auth/2fa/disable           - Disable 2FA
POST   /api/auth/2fa/verify            - Verify 2FA code during login
GET    /api/auth/verify-2fa-setup      - Get 2FA setup details
```

### 3.2 Password Management Endpoints
```
GET    /api/passwords                  - Get all passwords (with filtering)
GET    /api/passwords/:id              - Get specific password
POST   /api/passwords                  - Create new password entry
PUT    /api/passwords/:id              - Update password entry
DELETE /api/passwords/:id              - Delete password entry
POST   /api/passwords/search           - Search passwords
POST   /api/passwords/:id/copy         - Log password copy action
POST   /api/passwords/bulk-import      - Import passwords from file
GET    /api/passwords/export           - Export passwords as encrypted file
POST   /api/passwords/:id/restore      - Restore deleted password
```

### 3.3 Card Management Endpoints
```
GET    /api/cards                      - Get all stored cards
GET    /api/cards/:id                  - Get specific card
POST   /api/cards                      - Add new card
PUT    /api/cards/:id                  - Update card details
DELETE /api/cards/:id                  - Delete card
POST   /api/cards/:id/restore          - Restore deleted card
POST   /api/cards/bulk-import          - Import cards
GET    /api/cards/export               - Export cards
```

### 3.4 User Profile Endpoints
```
GET    /api/user/profile               - Get user profile
PUT    /api/user/profile               - Update profile
POST   /api/user/change-password       - Change master password
DELETE /api/user/account               - Delete account (with data purge)
GET    /api/user/audit-logs            - View audit logs
GET    /api/user/sessions              - View active sessions
DELETE /api/user/sessions/:id          - Revoke specific session
POST   /api/user/sessions/revoke-all   - Revoke all sessions
```

### 3.5 Admin Endpoints (Optional)
```
GET    /api/admin/users                - List users
GET    /api/admin/audit-logs           - View system-wide audit logs
DELETE /api/admin/users/:id            - Delete user (admin action)
```

---

## 4. Security Implementation Strategy

### 4.1 Encryption Flow
```
Master Password Entry
    ↓
Derive Master Key (PBKDF2)
    ↓
Generate User-Specific Encryption Key (AES-256)
    ↓
Encrypt Sensitive Data (AES-256-GCM)
    ↓
Store Encrypted Data in Database
```

### 4.2 Authentication Flow
```
User Login with Email & Password
    ↓
Verify against bcrypt hash
    ↓
Check 2FA if enabled
    ↓
Issue JWT + Refresh Token (signed with secret)
    ↓
Store session with token hash
    ↓
Return tokens to client
```

### 4.3 Security Best Practices
- **Never store passwords in plain text**
- **Use HTTPS only (TLS 1.2+)**
- **Implement rate limiting** on authentication endpoints
- **Add CSRF protection** for state-changing operations
- **Implement CORS** with whitelisted origins
- **Use secure HTTP headers** (Helmet.js)
- **Implement request validation** (Joi/Yup)
- **Add API versioning** for backward compatibility
- **Implement request signing** for sensitive operations
- **Log all sensitive operations** to audit trail
- **Implement account lockout** after failed attempts
- **Add IP whitelisting** (optional feature)

---

## 5. Development Phases

### Phase 1: Project Setup & Infrastructure (Week 1)
- [ ] Set up database (PostgreSQL with migrations)
- [ ] Configure environment variables
- [ ] Set up basic Express app structure
- [ ] Implement middleware (error handling, logging, CORS, rate limiting)
- [ ] Create database models and migrations
- [ ] Set up testing infrastructure

### Phase 2: Authentication System (Week 2)
- [ ] Implement user registration with email validation
- [ ] Implement secure login with JWT
- [ ] Implement password hashing (bcrypt)
- [ ] Implement refresh token mechanism
- [ ] Implement 2FA (TOTP)
- [ ] Implement password reset flow
- [ ] Write tests for auth endpoints

### Phase 3: Encryption & Key Management (Week 3)
- [ ] Implement master key derivation (PBKDF2)
- [ ] Implement AES-256-GCM encryption/decryption utilities
- [ ] Implement secure key storage
- [ ] Create encryption middleware
- [ ] Test encryption with various data types

### Phase 4: Password Management API (Week 4)
- [ ] Implement CRUD operations for passwords
- [ ] Implement search and filtering
- [ ] Implement soft delete with recovery
- [ ] Implement tagging system
- [ ] Implement bulk import/export
- [ ] Write comprehensive tests

### Phase 5: Card Management API (Week 5)
- [ ] Implement card storage with PCI-DSS considerations
- [ ] Implement card CRUD operations
- [ ] Implement card validation (Luhn algorithm)
- [ ] Implement masked card display
- [ ] Write tests

### Phase 6: Audit & Monitoring (Week 6)
- [ ] Implement audit logging system
- [ ] Implement session management
- [ ] Add user profile endpoints
- [ ] Implement access logs
- [ ] Create admin dashboard endpoints

### Phase 7: Testing & Documentation (Week 7)
- [ ] Write integration tests
- [ ] Write security tests
- [ ] Create API documentation (Swagger)
- [ ] Create deployment guide
- [ ] Perform security audit

### Phase 8: Deployment & Optimization (Week 8)
- [ ] Set up production environment
- [ ] Implement monitoring and alerting
- [ ] Optimize database queries
- [ ] Set up backup strategy
- [ ] Deploy to production

---

## 6. Error Handling & Response Format

### 6.1 Standard Response Format
```json
{
  "success": true/false,
  "message": "Descriptive message",
  "data": {},
  "error": {
    "code": "ERROR_CODE",
    "details": {}
  },
  "timestamp": "2026-05-27T16:59:20Z"
}
```

### 6.2 HTTP Status Codes
- `200` - OK
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `409` - Conflict
- `429` - Too Many Requests
- `500` - Internal Server Error

---

## 7. Environment Variables Required
```
NODE_ENV=development/production
PORT=3000
DATABASE_URL=postgresql://user:password@localhost:5432/kees_db
JWT_SECRET=<random-secret-key>
JWT_REFRESH_SECRET=<random-secret-key>
JWT_EXPIRY=24h
EMAIL_SERVICE=gmail/sendgrid
EMAIL_FROM=noreply@kees.com
MASTER_KEY_SALT_LENGTH=32
ENCRYPTION_ALGORITHM=aes-256-gcm
PBKDF2_ITERATIONS=100000
RATE_LIMIT_WINDOW=15
RATE_LIMIT_MAX_REQUESTS=100
```

---

## 8. Dependencies to Add
```json
{
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.10.0",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.1.0",
    "dotenv": "^16.3.1",
    "cors": "^2.8.5",
    "helmet": "^7.0.0",
    "express-rate-limit": "^7.0.0",
    "joi": "^17.10.0",
    "nodemailer": "^6.9.6",
    "speakeasy": "^2.0.0",
    "qrcode": "^1.5.3",
    "winston": "^3.11.0",
    "morgan": "^1.10.0",
    "uuid": "^9.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.7.0",
    "supertest": "^6.3.3",
    "db-migrate": "^0.11.13"
  }
}
```

---

## 9. Success Metrics
- [ ] All API endpoints functional and tested
- [ ] 90%+ test coverage
- [ ] Zero security vulnerabilities (via npm audit)
- [ ] Response time < 200ms for 95th percentile
- [ ] Full audit trail for all operations
- [ ] Comprehensive API documentation
- [ ] Encrypted data at rest and in transit
- [ ] 2FA implementation working correctly
- [ ] Backup and recovery processes functional

---

## 10. Known Constraints & Considerations
- **PCI-DSS Compliance**: Card storage requires adherence to PCI-DSS standards
- **Data Residency**: Consider GDPR compliance if serving EU users
- **Rate Limiting**: Implement to prevent brute force attacks
- **Key Rotation**: Plan for periodic encryption key rotation
- **Data Deletion**: Implement secure wiping of deleted data
- **Backup Strategy**: Encrypted backups with access controls

---

## 11. Future Enhancements
- [ ] Password strength analyzer and generator
- [ ] Breach detection (check against known breaches)
- [ ] Multi-device sync
- [ ] Browser extensions (Chrome, Firefox, Safari)
- [ ] Mobile app integration
- [ ] Team/Organization support
- [ ] Sharing encrypted passwords securely
- [ ] Biometric authentication support
- [ ] Hardware security key support
- [ ] Machine learning for anomaly detection

---

## 12. Deployment Checklist
- [ ] Environment variables configured
- [ ] Database migrations applied
- [ ] SSL/TLS certificates installed
- [ ] Rate limiting enabled
- [ ] Security headers configured
- [ ] Logging and monitoring enabled
- [ ] Backup system activated
- [ ] CDN configured (if applicable)
- [ ] DNS configured
- [ ] Email service configured
- [ ] Error tracking (Sentry) integrated
- [ ] Performance monitoring enabled

---

**Last Updated**: 2026-05-27
**Status**: In Development
**Version**: 1.0.0-draft
