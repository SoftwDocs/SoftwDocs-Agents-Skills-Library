---
name: hipaa-compliance-kit
description: Complete HIPAA compliance implementation for healthcare applications. Covers PHI encryption, audit logging, access controls, BAA templates, and regulatory requirements handling.
tags: [hipaa, healthcare, compliance, security, audit, phi]
version: 1.0.0
author: SoftwDocs
---

# HIPAA Compliance Kit

## Overview

A comprehensive skill for building HIPAA-compliant healthcare applications. Covers Protected Health Information (PHI) handling, encryption standards, audit logging, access controls, Business Associate Agreements (BAA), and regulatory compliance requirements.

## When to Use This Skill

- Building healthcare applications handling patient data
- Developing telemedicine platforms
- Creating electronic health record (EHR) systems
- Building medical device software
- Developing pharmacy management systems
- Implementing healthcare analytics platforms

## HIPAA Requirements Overview

### Privacy Rule
- Protects all individually identifiable health information (PHI)
- Gives patients rights over their health information
- Sets limits on uses and disclosures of PHI

### Security Rule
- Administrative safeguards (policies, procedures, training)
- Physical safeguards (facility access, device security)
- Technical safeguards (access control, encryption, audit controls)

### Breach Notification Rule
- Requires notification of breaches of unsecured PHI
- Timeline: 60 days for affected individuals
- Notification to HHS for breaches affecting 500+ individuals

## Core Implementation Patterns

### Pattern 1: PHI Data Classification

TypeScript schemas for PHI classification:

```typescript
import { z } from 'zod';

// PHI data types requiring special handling
const PHIDataTypes = z.enum([
  'name',
  'address',
  'date_of_birth',
  'social_security_number',
  'medical_record_number',
  'health_plan_beneficiary_number',
  'account_number',
  'certificate/license_number',
  'vehicle_identifier',
  'device_identifier',
  'web_url',
  'ip_address',
  'biometric_identifier',
  'photograph',
  'any_other_unique_identifying_number',
]);

// Patient data schema with PHI markers
const PatientSchema = z.object({
  id: z.string().uuid(),
  
  // Direct identifiers (PHI)
  firstName: z.string().describe('PHI: Direct identifier'),
  lastName: z.string().describe('PHI: Direct identifier'),
  dateOfBirth: z.date().describe('PHI: Direct identifier'),
  socialSecurityNumber: z.string().optional().describe('PHI: Direct identifier'),
  
  // Health information (PHI)
  medicalRecordNumber: z.string().describe('PHI: Health information'),
  diagnoses: z.array(z.string()).describe('PHI: Health information'),
  medications: z.array(z.string()).describe('PHI: Health information'),
  allergies: z.array(z.string()).describe('PHI: Health information'),
  
  // Contact information (PHI)
  email: z.string().email().describe('PHI: Contact information'),
  phone: z.string().describe('PHI: Contact information'),
  address: z.object({
    street: z.string().describe('PHI: Address'),
    city: z.string().describe('PHI: Address'),
    state: z.string().describe('PHI: Address'),
    zip: z.string().describe('PHI: Address'),
  }),
  
  // Metadata
  createdAt: z.date(),
  updatedAt: z.date(),
  lastAccessedAt: z.date(),
});

// PHI access log
const PHIAccessLogSchema = z.object({
  id: z.string().uuid(),
  patientId: z.string().uuid(),
  userId: z.string().uuid(),
  action: z.enum(['view', 'create', 'update', 'delete', 'export']),
  fieldsAccessed: z.array(PHIDataTypes),
  purpose: z.string(),
  timestamp: z.date(),
  ipAddress: z.string().describe('PHI: IP address'),
  userAgent: z.string(),
});

type Patient = z.infer<typeof PatientSchema>;
type PHIAccessLog = z.infer<typeof PHIAccessLogSchema>;
```

### Pattern 2: Encryption at Rest and in Transit

Comprehensive encryption implementation:

```typescript
import crypto from 'crypto';
import { encrypt as aesEncrypt, decrypt as aesDecrypt } from 'crypto-js/aes';
import UTF8 from 'crypto-js/enc-utf8';

// Encryption configuration
const ENCRYPTION_CONFIG = {
  algorithm: 'aes-256-gcm',
  keyLength: 32, // 256 bits
  ivLength: 16, // 128 bits
  tagLength: 16, // 128 bits
  saltLength: 32,
  iterations: 100000,
};

// Generate encryption key from password
export function deriveKey(password: string, salt: Buffer): Buffer {
  return crypto.pbkdf2Sync(
    password,
    salt,
    ENCRYPTION_CONFIG.iterations,
    ENCRYPTION_CONFIG.keyLength,
    'sha256'
  );
}

// Encrypt PHI data
export function encryptPHI(data: string, encryptionKey: string): {
  encrypted: string;
  iv: string;
  tag: string;
} {
  const iv = crypto.randomBytes(ENCRYPTION_CONFIG.ivLength);
  const key = Buffer.from(encryptionKey, 'hex');
  
  const cipher = crypto.createCipheriv(
    ENCRYPTION_CONFIG.algorithm,
    key,
    iv
  );
  
  let encrypted = cipher.update(data, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const tag = cipher.getAuthTag();
  
  return {
    encrypted,
    iv: iv.toString('hex'),
    tag: tag.toString('hex'),
  };
}

// Decrypt PHI data
export function decryptPHI(
  encrypted: string,
  iv: string,
  tag: string,
  encryptionKey: string
): string {
  const key = Buffer.from(encryptionKey, 'hex');
  const decipher = crypto.createDecipheriv(
    ENCRYPTION_CONFIG.algorithm,
    key,
    Buffer.from(iv, 'hex')
  );
  
  decipher.setAuthTag(Buffer.from(tag, 'hex'));
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}

// Field-level encryption for database
export class PHIEncryptor {
  constructor(private encryptionKey: string) {}
  
  encryptField(value: string): string {
    const { encrypted, iv, tag } = encryptPHI(value, this.encryptionKey);
    return JSON.stringify({ encrypted, iv, tag });
  }
  
  decryptField(encryptedValue: string): string {
    const { encrypted, iv, tag } = JSON.parse(encryptedValue);
    return decryptPHI(encrypted, iv, tag, this.encryptionKey);
  }
  
  encryptPatient(patient: Patient): Patient {
    const encryptor = new PHIEncryptor(this.encryptionKey);
    
    return {
      ...patient,
      firstName: encryptor.encryptField(patient.firstName),
      lastName: encryptor.encryptField(patient.lastName),
      socialSecurityNumber: patient.socialSecurityNumber
        ? encryptor.encryptField(patient.socialSecurityNumber)
        : undefined,
      email: encryptor.encryptField(patient.email),
      phone: encryptor.encryptField(patient.phone),
      address: {
        ...patient.address,
        street: encryptor.encryptField(patient.address.street),
        city: encryptor.encryptField(patient.address.city),
        state: encryptor.encryptField(patient.address.state),
        zip: encryptor.encryptField(patient.address.zip),
      },
    };
  }
}
```

### Pattern 3: Audit Logging System

Comprehensive audit trail for PHI access:

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export class AuditLogger {
  async logPHIAccess(params: {
    patientId: string;
    userId: string;
    action: 'view' | 'create' | 'update' | 'delete' | 'export';
    fieldsAccessed: string[];
    purpose: string;
    ipAddress: string;
    userAgent: string;
  }) {
    await prisma.pHIAccessLog.create({
      data: {
        ...params,
        timestamp: new Date(),
      },
    });
  }
  
  async logAuthentication(params: {
    userId: string;
    success: boolean;
    ipAddress: string;
    userAgent: string;
    failureReason?: string;
  }) {
    await prisma.authenticationLog.create({
      data: {
        ...params,
        timestamp: new Date(),
      },
    });
  }
  
  async logDataExport(params: {
    userId: string;
    patientIds: string[];
    format: 'pdf' | 'csv' | 'json';
    purpose: string;
    ipAddress: string;
  }) {
    await prisma.dataExportLog.create({
      data: {
        ...params,
        timestamp: new Date(),
      },
    });
  }
  
  async logSecurityIncident(params: {
    incidentType: string;
    severity: 'low' | 'medium' | 'high' | 'critical';
    description: string;
    affectedPatients?: string[];
    userId?: string;
    ipAddress?: string;
  }) {
    await prisma.securityIncidentLog.create({
      data: {
        ...params,
        timestamp: new Date(),
        status: 'open',
      },
    });
  }
}

// Middleware for automatic audit logging
export function auditPHIAccess(auditLogger: AuditLogger) {
  return async (req: any, res: any, next: any) => {
    const originalSend = res.send;
    
    res.send = function (data: any) {
      if (req.patientId && req.userId) {
        auditLogger.logPHIAccess({
          patientId: req.patientId,
          userId: req.userId,
          action: req.method.toLowerCase(),
          fieldsAccessed: req.accessedFields || [],
          purpose: req.purpose || 'Unknown',
          ipAddress: req.ip,
          userAgent: req.headers['user-agent'],
        });
      }
      originalSend.call(this, data);
    };
    
    next();
  };
}
```

### Pattern 4: Access Control System

Role-based access control (RBAC) for PHI:

```typescript
import { z } from 'zod';

// User roles with PHI access levels
const UserRoleSchema = z.enum([
  'patient',
  'provider',
  'nurse',
  'admin',
  'auditor',
  'billing',
  'researcher',
]);

// PHI access permissions
const PHIPermissionSchema = z.object({
  canViewOwnPHI: z.boolean(),
  canViewAllPHI: z.boolean(),
  canCreatePHI: z.boolean(),
  canUpdatePHI: z.boolean(),
  canDeletePHI: z.boolean(),
  canExportPHI: z.boolean(),
  canViewAuditLogs: z.boolean(),
  canManageUsers: z.boolean(),
});

// Role-based permission mapping
const ROLE_PERMISSIONS: Record<z.infer<typeof UserRoleSchema>, z.infer<typeof PHIPermissionSchema>> = {
  patient: {
    canViewOwnPHI: true,
    canViewAllPHI: false,
    canCreatePHI: false,
    canUpdatePHI: false,
    canDeletePHI: false,
    canExportPHI: true,
    canViewAuditLogs: false,
    canManageUsers: false,
  },
  provider: {
    canViewOwnPHI: false,
    canViewAllPHI: true,
    canCreatePHI: true,
    canUpdatePHI: true,
    canDeletePHI: false,
    canExportPHI: true,
    canViewAuditLogs: false,
    canManageUsers: false,
  },
  nurse: {
    canViewOwnPHI: false,
    canViewAllPHI: true,
    canCreatePHI: true,
    canUpdatePHI: true,
    canDeletePHI: false,
    canExportPHI: false,
    canViewAuditLogs: false,
    canManageUsers: false,
  },
  admin: {
    canViewOwnPHI: false,
    canViewAllPHI: true,
    canCreatePHI: true,
    canUpdatePHI: true,
    canDeletePHI: true,
    canExportPHI: true,
    canViewAuditLogs: true,
    canManageUsers: true,
  },
  auditor: {
    canViewOwnPHI: false,
    canViewAllPHI: true,
    canCreatePHI: false,
    canUpdatePHI: false,
    canDeletePHI: false,
    canExportPHI: true,
    canViewAuditLogs: true,
    canManageUsers: false,
  },
  billing: {
    canViewOwnPHI: false,
    canViewAllPHI: true,
    canCreatePHI: false,
    canUpdatePHI: false,
    canDeletePHI: false,
    canExportPHI: true,
    canViewAuditLogs: false,
    canManageUsers: false,
  },
  researcher: {
    canViewOwnPHI: false,
    canViewAllPHI: true,
    canCreatePHI: false,
    canUpdatePHI: false,
    canDeletePHI: false,
    canExportPHI: true,
    canViewAuditLogs: false,
    canManageUsers: false,
  },
};

export class AccessControl {
  constructor(private userRole: z.infer<typeof UserRoleSchema>) {}
  
  hasPermission(permission: keyof z.infer<typeof PHIPermissionSchema>): boolean {
    return ROLE_PERMISSIONS[this.userRole][permission];
  }
  
  canAccessPatient(patientId: string, requestingUserId: string): boolean {
    // Patients can only access their own data
    if (this.userRole === 'patient') {
      return patientId === requestingUserId;
    }
    
    // Other roles need appropriate permissions
    return this.hasPermission('canViewAllPHI');
  }
  
  canExportPHI(): boolean {
    return this.hasPermission('canExportPHI');
  }
  
  canViewAuditLogs(): boolean {
    return this.hasPermission('canViewAuditLogs');
  }
}

// Middleware for access control
export function requirePHIPermission(permission: keyof z.infer<typeof PHIPermissionSchema>) {
  return (req: any, res: any, next: any) => {
    const accessControl = new AccessControl(req.user.role);
    
    if (!accessControl.hasPermission(permission)) {
      return res.status(403).json({
        error: 'Insufficient permissions',
        required: permission,
      });
    }
    
    next();
  };
}
```

### Pattern 5: Minimum Necessary Standard

Implementing the minimum necessary principle:

```typescript
export class MinimumNecessaryFilter {
  filterPatientData(
    patient: Patient,
    requestingRole: z.infer<typeof UserRoleSchema>,
    purpose: string
  ): Partial<Patient> {
    switch (requestingRole) {
      case 'billing':
        // Billing only needs name, insurance info, procedures
        return {
          id: patient.id,
          firstName: patient.firstName,
          lastName: patient.lastName,
          // Add insurance and procedure fields
        };
        
      case 'researcher':
        // Researcher gets de-identified data
        return {
          id: patient.id,
          // No direct identifiers
          diagnoses: patient.diagnoses,
          medications: patient.medications,
          // Remove all direct identifiers
        };
        
      case 'auditor':
        // Auditor gets full data with audit trail
        return patient;
        
      default:
        // Provider/nurse gets relevant clinical data
        return {
          id: patient.id,
          firstName: patient.firstName,
          lastName: patient.lastName,
          dateOfBirth: patient.dateOfBirth,
          diagnoses: patient.diagnoses,
          medications: patient.medications,
          allergies: patient.allergies,
        };
    }
  }
  
  redactPHI(text: string, fieldsToRedact: string[]): string {
    let redacted = text;
    
    fieldsToRedact.forEach((field) => {
      const regex = new RegExp(`${field}:\\s*[^\\n]+`, 'gi');
      redacted = redacted.replace(regex, `${field}: [REDACTED]`);
    });
    
    return redacted;
  }
}
```

## Database Schema with HIPAA Compliance

```prisma
model Patient {
  id              String   @id @default(uuid())
  firstName       String   @map("first_name")
  lastName        String   @map("last_name")
  dateOfBirth     DateTime @map("date_of_birth")
  socialSecurityNumber String? @map("ssn") @db.Text
  email           String   @db.Text
  phone           String   @db.Text
  street          String   @map("address_street") @db.Text
  city            String   @map("address_city") @db.Text
  state           String   @map("address_state") @db.Text
  zip             String   @map("address_zip") @db.Text
  
  // Encrypted fields (stored as JSON)
  encryptedFields Json?
  
  createdAt       DateTime @default(now()) @map("created_at")
  updatedAt       DateTime @updatedAt @map("updated_at")
  lastAccessedAt  DateTime @map("last_accessed_at")
  
  // Audit trail
  accessLogs      PHIAccessLog[]
  
  @@map("patients")
}

model PHIAccessLog {
  id              String   @id @default(uuid())
  patientId       String   @map("patient_id")
  userId          String   @map("user_id")
  action          String
  fieldsAccessed  Json     @map("fields_accessed")
  purpose         String
  timestamp       DateTime @default(now())
  ipAddress       String   @map("ip_address") @db.Text
  userAgent       String   @map("user_agent") @db.Text
  
  patient         Patient  @relation(fields: [patientId], references: [id])
  
  @@map("phi_access_logs")
  @@index([patientId])
  @@index([userId])
  @@index([timestamp])
}

model AuthenticationLog {
  id              String   @id @default(uuid())
  userId          String   @map("user_id")
  success         Boolean
  ipAddress       String   @map("ip_address") @db.Text
  userAgent       String   @map("user_agent") @db.Text
  failureReason   String?  @map("failure_reason")
  timestamp       DateTime @default(now())
  
  @@map("authentication_logs")
  @@index([userId])
  @@index([timestamp])
}

model SecurityIncidentLog {
  id              String   @id @default(uuid())
  incidentType    String   @map("incident_type")
  severity        String
  description     String   @db.Text
  affectedPatients Json?   @map("affected_patients")
  userId          String?  @map("user_id")
  ipAddress       String?  @map("ip_address") @db.Text
  status          String   @default("open")
  timestamp       DateTime @default(now())
  resolvedAt      DateTime? @map("resolved_at")
  
  @@map("security_incident_logs")
  @@index([timestamp])
  @@index([status])
}
```

## Business Associate Agreement (BAA) Template

```markdown
# Business Associate Agreement (BAA)

## 1. Definitions

**Business Associate (BA):** [Company Name]
**Covered Entity (CE):** [Healthcare Organization]

## 2. Permitted Uses and Disclosures

The BA may use and disclose PHI only for:
- Providing services to the CE
- Required by law
- As authorized by the patient

## 3. Safeguards

The BA agrees to implement:
- Administrative safeguards
- Physical safeguards
- Technical safeguards
- As required by HIPAA Security Rule

## 4. Reporting

The BA will report:
- Any security incident within 24 hours
- Any breach of unsecured PHI within 60 days
- Any government investigation involving PHI

## 5. Termination

This agreement terminates if:
- The BA breaches any provision
- The CE terminates the business relationship
- Either party provides 30 days notice

## 6. Return or Destruction

Upon termination, the BA will:
- Return all PHI to the CE
- Destroy all PHI if requested
- Provide certification of destruction

## 7. Compliance

The BA agrees to:
- Comply with all HIPAA requirements
- Allow CE to audit compliance
- Train employees on HIPAA requirements
```

## Security Incident Response Plan

```typescript
export class SecurityIncidentResponse {
  async detectIncident(params: {
    incidentType: string;
    severity: 'low' | 'medium' | 'high' | 'critical';
    description: string;
  }) {
    // Log incident
    await auditLogger.logSecurityIncident(params);
    
    // Notify security team
    await this.notifySecurityTeam(params);
    
    // If critical, initiate immediate response
    if (params.severity === 'critical') {
      await this.initiateCriticalResponse(params);
    }
  }
  
  async notifySecurityTeam(incident: any) {
    // Send alerts via multiple channels
    await this.sendEmailAlert(incident);
    await this.sendSlackAlert(incident);
    await this.sendPagerDutyAlert(incident);
  }
  
  async initiateCriticalResponse(incident: any) {
    // Lock affected accounts
    await this.lockAffectedAccounts(incident);
    
    // Preserve evidence
    await this.preserveEvidence(incident);
    
    // Notify HHS if breach affects 500+ individuals
    await this.assessBreachNotification(incident);
  }
  
  async assessBreachNotification(incident: any) {
    const affectedCount = await this.countAffectedPatients(incident);
    
    if (affectedCount >= 500) {
      // Notify HHS within 60 days
      await this.notifyHHS(incident);
      
      // Notify media
      await this.notifyMedia(incident);
    }
    
    // Notify affected individuals
    await this.notifyAffectedIndividuals(incident);
  }
}
```

## Best Practices

### ✅ Do

- Encrypt all PHI at rest and in transit
- Log all PHI access with timestamps
- Implement role-based access control
- Use minimum necessary principle
- Conduct regular security audits
- Train all employees on HIPAA requirements
- Have a BAA with all third-party vendors
- Implement a breach response plan

### ❌ Don't

- Store PHI in unencrypted form
- Share PHI without proper authorization
- Ignore audit logs
- Use default passwords
- Allow unnecessary PHI access
- Forget to revoke access for terminated employees
- Store PHI on personal devices
- Skip security training

## Compliance Checklist

- [ ] All PHI encrypted at rest (AES-256)
- [ ] All PHI encrypted in transit (TLS 1.3)
- [ ] Comprehensive audit logging implemented
- [ ] Role-based access control in place
- [ ] Minimum necessary principle applied
- [ ] Business Associate Agreements signed
- [ ] Security incident response plan created
- [ ] Regular security audits scheduled
- [ ] Employee HIPAA training completed
- [ ] Breach notification procedures documented
- [ ] Data retention policy established
- [ ] Disaster recovery plan implemented

## Resources

- [HIPAA Privacy Rule](https://www.hhs.gov/hipaa/for-professionals/privacy/)
- [HIPAA Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/)
- [Breach Notification Rule](https://www.hhs.gov/hipaa/for-professionals/breach-notification/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
