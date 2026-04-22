---
name: pharmacy-pos-logic
description: Specialized business logic for pharmacy/medical store management systems. Prescription handling, drug interactions, inventory expiry tracking, insurance claims, and regulatory compliance.
tags: [pharmacy, pos, medical, healthcare, prescription, inventory, compliance]
version: 2.0.0
author: SoftwDocs
---

# Pharmacy POS Logic

## Overview

A comprehensive skill for building pharmacy and medical store management systems. Covers prescription validation, drug interaction checking, expiry tracking, insurance claims processing, inventory management, and healthcare regulatory compliance.

## Domain Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    PHARMACY POS ENTITIES                        │
├─────────────────────────────────────────────────────────────────┤
│  Patient <-> Prescription <-> Dispensed Medicine                │
│     │             │                  │                          │
│     ▼             ▼                  ▼                          │
│  Insurance    Drug/Medicine    Inventory Stock                │
│  Provider     Database           Management                     │
├─────────────────────────────────────────────────────────────────┤
│  Prescriber   Drug Interaction   Insurance Claim                │
│  (Doctor)     Checker            Processing                     │
└─────────────────────────────────────────────────────────────────┘
```

## Core TypeScript Schemas

```typescript
// Prescription validation schema
const PrescriptionSchema = z.object({
  patientId: z.string().uuid(),
  prescriberId: z.string().uuid(),
  medications: z.array(z.object({
    ndc: z.string().regex(/^\d{11}$/), // 11-digit NDC
    quantity: z.number().positive().max(9999),
    daysSupply: z.number().int().positive().max(365),
    refills: z.number().int().min(0).max(11),
  })).min(1).max(15),
  issueDate: z.date().max(new Date()),
  expiryDate: z.date(),
});

// Drug interaction types
interface DrugInteraction {
  interactingDrugNdc: string;
  severity: 'contraindicated' | 'major' | 'moderate' | 'minor';
  mechanism: string;
  recommendation: string;
}

// Inventory with expiry tracking
interface InventoryItem {
  medicineId: string;
  batchNumber: string;
  expiryDate: Date;
  quantityInStock: number;
  reorderLevel: number;
  isExpired: boolean;
  daysUntilExpiry: number;
}

// Insurance claim structure
interface InsuranceClaim {
  claimNumber: string;
  memberId: string;
  bin: string; // Bank Identification Number
  status: 'pending' | 'approved' | 'rejected';
  rejectionReason?: string;
  approvedAmount?: number;
  copayAmount?: number;
}
```

## 1. Prescription Validation

```typescript
// lib/pharmacy/prescription-validator.ts
export class PrescriptionValidator {
  async validate(prescription: Prescription): Promise<ValidationResult> {
    const errors: ValidationError[] = [];

    // 1. Check expiration
    if (prescription.expiryDate < new Date()) {
      errors.push({
        code: 'PRESCRIPTION_EXPIRED',
        message: 'Prescription has expired',
        severity: 'error',
      });
    }

    // 2. Validate prescriber license
    const prescriber = await this.getPrescriber(prescription.prescriberId);
    if (!prescriber?.licenseActive) {
      errors.push({
        code: 'INACTIVE_PRESCRIBER',
        message: 'Prescriber license is inactive',
        severity: 'error',
      });
    }

    // 3. Controlled substance checks
    for (const med of prescription.medications) {
      const drug = await this.getDrugByNdc(med.ndc);
      
      if (drug?.controlledSubstance?.deaSchedule === 2 && med.refills > 0) {
        errors.push({
          code: 'CII_NO_REFILLS',
          message: 'Schedule II substances cannot have refills',
          severity: 'error',
        });
      }
    }

    return { isValid: errors.length === 0, errors, warnings: [] };
  }
}
```

## 2. Drug Interaction Checking

```typescript
// lib/pharmacy/drug-interactions.ts
export class DrugInteractionChecker {
  async checkInteractions(
    newDrugNdc: string,
    currentMedications: string[],
    patientAllergies: string[]
  ): Promise<InteractionResult> {
    const interactions: DrugInteraction[] = [];

    // Check drug-drug interactions
    for (const currentNdc of currentMedications) {
      const interaction = await this.getInteraction(newDrugNdc, currentNdc);
      if (interaction) interactions.push(interaction);
    }

    // Check allergy interactions
    const newDrug = await this.getDrugByNdc(newDrugNdc);
    for (const allergy of patientAllergies) {
      if (this.isAllergenInDrug(allergy, newDrug)) {
        interactions.push({
          severity: 'contraindicated',
          message: `Patient allergic to ${allergy} - found in ${newDrug.genericName}`,
        });
      }
    }

    return {
      hasContraindication: interactions.some(i => i.severity === 'contraindicated'),
      hasMajorInteraction: interactions.some(i => i.severity === 'major'),
      interactions,
    };
  }
}
```

## 3. Inventory Management with Expiry

```typescript
// lib/pharmacy/inventory.ts
export class PharmacyInventory {
  // Get all items expiring within days
  async getExpiringItems(days: number = 30): Promise<InventoryItem[]> {
    const cutoffDate = new Date();
    cutoffDate.setDate(cutoffDate.getDate() + days);

    return await prisma.inventory.findMany({
      where: {
        expiryDate: { lte: cutoffDate },
        quantityInStock: { gt: 0 },
      },
      orderBy: { expiryDate: 'asc' },
    });
  }

  // FIFO dispensing - oldest stock first
  async dispenseFIFO(
    medicineId: string,
    quantity: number
  ): Promise<DispenseResult> {
    const batches = await prisma.inventory.findMany({
      where: {
        medicineId,
        quantityInStock: { gt: 0 },
        expiryDate: { gt: new Date() },
      },
      orderBy: { expiryDate: 'asc' },
    });

    const dispensed: Array<{ batchId: string; quantity: number }> = [];
    let remaining = quantity;

    for (const batch of batches) {
      if (remaining <= 0) break;
      
      const take = Math.min(remaining, batch.quantityInStock);
      dispensed.push({ batchId: batch.id, quantity: take });
      remaining -= take;
    }

    if (remaining > 0) {
      return { success: false, error: 'Insufficient stock' };
    }

    return { success: true, dispensed };
  }

  // Reorder points
  async checkReorderNeeds(): Promise<ReorderAlert[]> {
    const lowStock = await prisma.inventory.groupBy({
      by: ['medicineId'],
      having: {
        quantityInStock: { sum: { lte: prisma.inventory.fields.reorderLevel } },
      },
    });

    return lowStock.map(item => ({
      medicineId: item.medicineId,
      currentStock: item._sum.quantityInStock,
      reorderLevel: item.reorderLevel,
      suggestedOrderQuantity: item.reorderQuantity,
    }));
  }
}
```

## 4. Insurance Claims Processing

```typescript
// lib/pharmacy/insurance.ts
export class InsuranceProcessor {
  async submitClaim(
    prescription: Prescription,
    medication: DispensedMedication
  ): Promise<InsuranceClaim> {
    const patient = await this.getPatient(prescription.patientId);
    
    if (!patient.insurance) {
      return { status: 'rejected', rejectionReason: 'No insurance on file' };
    }

    const claim = {
      bin: patient.insurance.bin,
      pcn: patient.insurance.pcn,
      memberId: patient.insurance.memberId,
      groupNumber: patient.insurance.groupNumber,
      ndc: medication.ndc,
      quantity: medication.quantityDispensed,
      daysSupply: medication.daysSupply,
      prescriberNpi: prescription.prescriberInfo.npi,
      pharmacyNpi: process.env.PHARMACY_NPI,
    };

    // Submit to insurance processor (NCPDP D.0 format)
    const response = await this.submitToProcessor(claim);

    return {
      claimNumber: response.claimNumber,
      status: response.status,
      approvedAmount: response.approvedAmount,
      copayAmount: response.patientPay,
      rejectionReason: response.rejectionMessage,
    };
  }

  // Handle rejection codes
  getRejectionAction(code: string): RejectionAction {
    const actions: Record<string, RejectionAction> = {
      '75': { action: 'prior_authorization', message: 'Prior authorization required' },
      '76': { action: 'plan_limitation', message: 'Plan limitation exceeded' },
      '79': { action: 'refill_too_soon', message: 'Refill too soon' },
      '88': { action: 'drug_not_covered', message: 'Drug not covered - suggest alternative' },
    };

    return actions[code] || { action: 'review', message: 'Review required' };
  }
}
```

## 5. POS Transaction Flow

```typescript
// lib/pharmacy/pos.ts
export class PharmacyPOS {
  async processPrescription(
    prescriptionId: string
  ): Promise<POSTransaction> {
    const transaction = await prisma.$transaction(async (tx) => {
      // 1. Validate prescription
      const prescription = await tx.prescription.findUnique({
        where: { id: prescriptionId },
        include: { medications: true, patient: true },
      });

      const validation = await this.validator.validate(prescription);
      if (!validation.isValid) {
        throw new ValidationError(validation.errors);
      }

      // 2. Check drug interactions
      const currentMeds = await this.getCurrentMedications(prescription.patientId);
      for (const med of prescription.medications) {
        const interactions = await this.interactionChecker.checkInteractions(
          med.ndc,
          currentMeds.map(m => m.ndc),
          prescription.patient.allergies
        );

        if (interactions.hasContraindication) {
          throw new DrugInteractionError(interactions);
        }
      }

      // 3. Dispense from inventory (FIFO)
      const dispensed: DispensedMedication[] = [];
      for (const med of prescription.medications) {
        const result = await this.inventory.dispenseFIFO(med.ndc, med.quantity);
        if (!result.success) throw new OutOfStockError(med.drugName);

        // Update inventory
        for (const { batchId, quantity } of result.dispensed) {
          await tx.inventory.update({
            where: { id: batchId },
            data: { quantityInStock: { decrement: quantity } },
          });
        }

        dispensed.push({
          medicationId: med.id,
          quantityDispensed: med.quantity,
          batches: result.dispensed,
        });
      }

      // 4. Process insurance
      const claims: InsuranceClaim[] = [];
      for (const med of dispensed) {
        const claim = await this.insurance.submitClaim(prescription, med);
        claims.push(claim);
      }

      // 5. Calculate pricing
      const pricing = this.calculatePricing(dispensed, claims);

      // 6. Create transaction record
      const transaction = await tx.pOSTransaction.create({
        data: {
          prescriptionId,
          patientId: prescription.patientId,
          dispensedMedications: dispensed,
          insuranceClaims: claims,
          ...pricing,
          status: 'completed',
        },
      });

      // 7. Update prescription status
      await tx.prescription.update({
        where: { id: prescriptionId },
        data: {
          status: 'filled',
          refillsRemaining: { decrement: 1 },
        },
      });

      return transaction;
    });

    return transaction;
  }

  private calculatePricing(
    dispensed: DispensedMedication[],
    claims: InsuranceClaim[]
  ): PricingBreakdown {
    const subtotal = dispensed.reduce((sum, med) => sum + med.price.totalPrice, 0);
    const insuranceCovered = claims.reduce((sum, claim) => 
      sum + (claim.approvedAmount || 0), 0);
    const tax = subtotal * 0.08; // 8% tax
    
    return {
      subtotal,
      insuranceCovered,
      tax,
      total: subtotal - insuranceCovered + tax,
      patientPay: subtotal - insuranceCovered + tax,
    };
  }
}
```

## 6. Compliance & Reporting

```typescript
// lib/pharmacy/compliance.ts
export class PharmacyCompliance {
  // Controlled substance reporting (DEA)
  async generateCsReport(schedule: number, startDate: Date, endDate: Date) {
    const dispensed = await prisma.dispensedMedication.findMany({
      where: {
        createdAt: { gte: startDate, lte: endDate },
        medication: {
          controlledSubstance: { deaSchedule: schedule },
        },
      },
      include: {
        prescription: { include: { prescriberInfo: true } },
        patient: true,
      },
    });

    return dispensed.map(d => ({
      deaNumber: d.prescription.prescriberInfo.deaNumber,
      patientId: d.patient.id,
      patientAddress: d.patient.address,
      drugNdc: d.medicationId,
      quantity: d.quantityDispensed,
      dateFilled: d.dispensedAt,
      prescriptionNumber: d.prescription.prescriptionNumber,
    }));
  }

  // Generate audit trail
  async getAuditTrail(
    entityType: 'patient' | 'prescription' | 'inventory',
    entityId: string,
    startDate?: Date
  ): Promise<AuditEntry[]> {
    return await prisma.auditLog.findMany({
      where: {
        entityType,
        entityId,
        timestamp: startDate ? { gte: startDate } : undefined,
      },
      orderBy: { timestamp: 'desc' },
      include: { user: { select: { name: true } } },
    });
  }

  // Check for prescription fraud patterns
  async detectFraudPatterns(): Promise<FraudAlert[]> {
    // Multiple pharmacies
    const multiPharmacy = await prisma.$queryRaw`
      SELECT patient_id, COUNT(DISTINCT pharmacy_id) as pharmacy_count
      FROM prescriptions
      WHERE created_at > NOW() - INTERVAL '30 days'
      GROUP BY patient_id
      HAVING COUNT(DISTINCT pharmacy_id) > 3
    `;

    // Early refills
    const earlyRefills = await prisma.$queryRaw`
      SELECT patient_id, medication_id, 
             AVG(days_supply - days_since_last_fill) as avg_early_days
      FROM prescription_fills
      WHERE created_at > NOW() - INTERVAL '90 days'
      GROUP BY patient_id, medication_id
      HAVING AVG(days_supply - days_since_last_fill) < -5
    `;

    return [
      ...multiPharmacy.map(r => ({
        type: 'multi_pharmacy',
        patientId: r.patient_id,
        severity: 'high',
      })),
      ...earlyRefills.map(r => ({
        type: 'early_refill_pattern',
        patientId: r.patient_id,
        medicationId: r.medication_id,
        severity: 'medium',
      })),
    ];
  }
}
```

## 7. Database Schema (Prisma)

```prisma
// schema.prisma
model Patient {
  id                String   @id @default(uuid())
  mrn               String   @unique
  firstName         String
  lastName          String
  dateOfBirth       DateTime
  gender            String
  phone             String
  email             String?
  allergies         String[]
  medicalConditions String[]
  
  prescriptions     Prescription[]
  insurance         InsuranceInfo?
  
  createdAt         DateTime @default(now())
  updatedAt         DateTime @updatedAt
}

model Prescription {
  id                String      @id @default(uuid())
  prescriptionNumber String    @unique
  patientId         String
  prescriberId      String
  
  patient           Patient     @relation(fields: [patientId], references: [id])
  medications       PrescribedMedication[]
  
  issueDate         DateTime
  expiryDate        DateTime
  refillsAuthorized Int
  refillsRemaining  Int
  status            String      // pending, filled, partial, refused
  
  createdAt         DateTime    @default(now())
}

model Medicine {
  id                String   @id @default(uuid())
  ndc               String   @unique @db.VarChar(11)
  brandName         String
  genericName       String
  strength          String
  form              String
  schedule          String   // OTC, C-V, C-IV, C-III, C-II
  
  inventory         Inventory[]
  interactions      DrugInteraction[]
  
  @@index([genericName])
  @@index([schedule])
}

model Inventory {
  id              String   @id @default(uuid())
  medicineId      String
  batchNumber     String
  lotNumber       String
  expiryDate      DateTime
  quantityInStock Int
  reorderLevel    Int
  unitCost        Decimal
  sellingPrice    Decimal
  
  medicine        Medicine @relation(fields: [medicineId], references: [id])
  
  @@index([medicineId])
  @@index([expiryDate])
}

model DispensedMedication {
  id                String   @id @default(uuid())
  prescriptionId    String
  medicationId      String
  patientId         String
  quantityDispensed Int
  daysSupply        Int
  refillsRemaining  Int
  pharmacistId      String
  dispensedAt       DateTime @default(now())
  counselingProvided Boolean
}
```

## 8. UI Components for Pharmacy POS

```tsx
// components/pharmacy/prescription-verification.tsx
export function PrescriptionVerification({ prescription }: Props) {
  const { data: interactions } = useDrugInteractions(
    prescription.medications.map(m => m.ndc),
    prescription.patient.allergies
  );

  return (
    <div className="space-y-4">
      {/* Patient Info Card */}
      <Card>
        <CardHeader>
          <div className="flex items-center gap-4">
            <Avatar>
              <AvatarFallback>
                {prescription.patient.firstName[0]}{prescription.patient.lastName[0]}
              </AvatarFallback>
            </Avatar>
            <div>
              <h3>{prescription.patient.firstName} {prescription.patient.lastName}</h3>
              <p className="text-sm text-muted-foreground">
                DOB: {formatDate(prescription.patient.dateOfBirth)} | 
                MRN: {prescription.patient.mrn}
              </p>
            </div>
          </div>
        </CardHeader>
        <CardContent>
          {prescription.patient.allergies.length > 0 && (
            <Alert variant="destructive">
              <AlertTriangle className="h-4 w-4" />
              <AlertTitle>Allergies</AlertTitle>
              <AlertDescription>
                {prescription.patient.allergies.join(', ')}
              </AlertDescription>
            </Alert>
          )}
        </CardContent>
      </Card>

      {/* Drug Interactions */}
      {interactions?.map(interaction => (
        <Alert 
          key={interaction.id}
          variant={interaction.severity === 'contraindicated' ? 'destructive' : 'warning'}
        >
          <AlertCircle className="h-4 w-4" />
          <AlertTitle>{interaction.severity.toUpperCase()} Interaction</AlertTitle>
          <AlertDescription>{interaction.recommendation}</AlertDescription>
        </Alert>
      ))}

      {/* Medications List */}
      {prescription.medications.map(med => (
        <Card key={med.id}>
          <CardHeader>
            <div className="flex justify-between items-start">
              <div>
                <CardTitle>{med.drugName}</CardTitle>
                <p className="text-sm text-muted-foreground">{med.genericName}</p>
              </div>
              <Badge variant={med.substitutionAllowed ? 'default' : 'secondary'}>
                {med.substitutionAllowed ? 'Generic OK' : 'DAW'}
              </Badge>
            </div>
          </CardHeader>
          <CardContent>
            <div className="grid grid-cols-3 gap-4 text-sm">
              <div>
                <p className="text-muted-foreground">Quantity</p>
                <p className="font-medium">{med.quantity} {med.quantityUnit}</p>
              </div>
              <div>
                <p className="text-muted-foreground">Days Supply</p>
                <p className="font-medium">{med.daysSupply} days</p>
              </div>
              <div>
                <p className="text-muted-foreground">Refills</p>
                <p className="font-medium">{prescription.refillsRemaining} remaining</p>
              </div>
            </div>
            <div className="mt-4">
              <p className="text-muted-foreground">Directions (SIG)</p>
              <p className="font-medium">{med.sig}</p>
            </div>
          </CardContent>
        </Card>
      ))}
    </div>
  );
}

// Inventory expiry warning component
export function ExpiryAlerts() {
  const { data: expiring } = useExpiringInventory(30);

  if (!expiring?.length) return null;

  return (
    <Alert variant="warning">
      <AlertCircle className="h-4 w-4" />
      <AlertTitle>Expiring Inventory</AlertTitle>
      <AlertDescription>
        {expiring.length} items expiring within 30 days
      </AlertDescription>
      <Button variant="outline" size="sm" asChild className="mt-2">
        <Link href="/inventory/expiring">View Details</Link>
      </Button>
    </Alert>
  );
}
```

## Regulatory Compliance Checklist

- [ ] HIPAA compliance for patient data
- [ ] DEA controlled substance tracking
- [ ] State pharmacy board regulations
- [ ] FDA adverse event reporting
- [ ] Prescription monitoring program (PMP) integration
- [ ] Audit trail for all transactions
- [ ] Data retention policies
- [ ] Secure data encryption at rest and in transit

## Integration Points

1. **NCPDP D.0** - Insurance claims
2. **Surescripts** - E-prescribing
3. **State PMP** - Prescription monitoring
4. **First Databank (FDB)** - Drug database
5. **Medi-Span** - Drug pricing
6. **NPI Registry** - Provider verification
