# Car Insurance Ontology (Finnish Traffic Insurance Law)
# Based on Liikennevakuutuslaki (Traffic Insurance Act)

## Overview
This ontology represents the concepts, relationships, and rules from Finnish car insurance legislation (Liikennevakuutuslaki).

---

# CLASSES / ENTITIES

## 1. Vehicle (Ajoneuvo)
Represents any motor vehicle subject to insurance requirements.

### Subclasses:
- **MotorVehicle** (Moottoriajoneuvo)
  - PassengerCar (Henkilöauto)
  - Motorcycle (Moottoripyörä)
  - Truck (Kuorma-auto)
  - Bus (Linja-auto)
- **Trailer** (Perävaunu)
- **TraktTractor** (ori)

### Attributes:
- vehicleId: String (unique identifier)
- registrationNumber: String
- vehicleType: VehicleType
- firstRegistrationDate: Date
- owner: Person | LegalEntity

---

## 2. Insurance (Vakuutus)
The insurance contract itself.

### Subclasses:
- **MandatoryTrafficInsurance** (Liikennevakuutus)
  - Statutory motor liability insurance
  - Required by law for all vehicles
  
- **ComprehensiveInsurance** (Kaskovakuutus)
  - **FullComprehensive** (täyskasko)
  - **PartialComprehensive** (osittaiskasko)
  - **FireAndTheft** (palovarkauskasko)

### Attributes:
- policyNumber: String
- insuranceCompany: InsuranceCompany
- policyHolder: Person | LegalEntity
- startDate: Date
- endDate: Date
- premium: Decimal
- deductible: Decimal (omavastuu)
- coverage: CoverageType

---

## 3. Person (Henkilö)
Any natural person involved in insurance.

### Subclasses:
- **PolicyHolder** (Vakuutuksenottaja)
  - The person who owns the insurance contract
  
- **Insured** (Vakuutettu)
  - Person covered by the insurance
  
- **Driver** (Kuljettaja)
  - Person driving the vehicle at time of incident
  
- **InjuredParty** (Loukkaantunut)
  - Person who suffered injury
  
- **DamagedParty** (Vahinkoa kärsinyt)
  - Person who suffered property damage

### Attributes:
- personalId: String (henkilötunnus)
- firstName: String
- lastName: String
- address: Address
- drivingLicense: DrivingLicense

---

## 4. LegalEntity (Oikeushenkilö)
Corporations and organizations.

### Subclasses:
- **InsuranceCompany** (Vakuutusyhtiö)
  - Licensed insurance provider
  
- **PolicyHoldingCompany** (Vakuutuksenottajayritys)
  - Company holding fleet insurance
  
- **RepairShop** (Korjaamo)
  - Authorized repair facility

### Attributes:
- businessId: String (Y-tunnus)
- name: String
- address: Address

---

## 5. InsuranceEvent (Vakuutustapahtuma)
An event that triggers insurance coverage.

### Subclasses:
- **TrafficAccident** (Liikennevakuutus)
  - Collision with another vehicle
  - Single-vehicle accident
  - Accident involving pedestrian
  
- **FireIncident** (Tulipalo)
  - Vehicle fire
  
- **TheftEvent** (Varkaus)
  - Vehicle theft
  - Theft of parts
  
- **NaturalDamage** (Luonnonvaurio)
  - Storm damage
  - Flood damage
  - Hail damage

### Attributes:
- eventId: String
- eventDate: DateTime
- location: Coordinates
- description: String
- involvedVehicles: List<Vehicle>
- involvedPersons: List<Person>

---

## 6. Damage (Vahinko)
Damage resulting from an insurance event.

### Subclasses:
- **PropertyDamage** (Omaisuusvahinko)
  - Vehicle damage
  - Property damage to third parties
  
- **PersonalInjury** (Henkilövahinko)
  - Physical injury
  - Psychological injury
  - Death

### Attributes:
- damageId: String
- damageType: DamageType
- severity: SeverityLevel
- estimatedCost: Decimal
- repairCost: Decimal
- medicalCost: Decimal

---

## 7. Compensation (Korvaus)
Payment or benefit provided by insurance.

### Subclasses:
- **PropertyDamageCompensation** (Omaisuuskorvaus)
- **MedicalExpenseCompensation** (Sairaanhoitokorvaus)
- **DisabilityCompensation** (Työkyvyttömyyskorvaus)
- **DeathCompensation** (Kuolemakorvaus)
- **PainAndSuffering** (Kivus ja haitta)
- **LostWagesCompensation** (Ansionmenetyskorvaus)

### Attributes:
- compensationId: String
- claimId: String
- amount: Decimal
- currency: String (EUR)
- paymentDate: Date
- recipient: Person

---

## 8. Claim (Korvausvaatimus)
A formal request for compensation.

### Attributes:
- claimId: String
- claimDate: Date
- claimant: Person
- insurancePolicy: Insurance
- insuranceEvent: InsuranceEvent
- damage: Damage
- status: ClaimStatus
- processedDate: Date
- decisionDate: Date

### ClaimStatus:
- submitted (jätetty)
- under_review (käsittelyssä)
- additional_info_required (lisätietoja pyydetty)
- approved (hyväksytty)
- rejected (hylätty)
- partially_approved (osittain hyväksytty)
- paid (maksettu)

---

# RELATIONSHIPS

## Vehicle Relationships
- vehicle owner -> owns -> Vehicle
- vehicle registered_at -> registration authority
- vehicle covered_by -> Insurance

## Insurance Relationships
- insurance provided_by -> InsuranceCompany
- insurance held_by -> PolicyHolder
- insurance covers -> Vehicle
- insurance covers -> Person

## Event Relationships
- event caused_by -> Vehicle
- event involved -> Person
- event resulted_in -> Damage
- event triggered_by -> Driver

## Compensation Relationships
- compensation paid_by -> InsuranceCompany
- compensation paid_to -> Person
- compensation for_damage -> Damage
- compensation requested_via -> Claim

---

# RULES / INFERENCE

## Mandatory Insurance Rule
```
IF vehicle.registrationRequired = true
THEN vehicle.mustHaveInsurance = MandatoryTrafficInsurance
```

## Coverage Determination Rule
```
IF damage.type = PropertyDamage 
   AND damage.severity = TotalLoss
THEN compensation.maximum = Vehicle.marketValue
```

## Deductible Rule
```
IF driver.age < 25 
   AND insurance.type = Comprehensive
THEN insurance.deductible = standardDeductible * 1.5
```

## Claim Processing Rule
```
IF claim.status = under_review 
   AND daysSinceClaim > 30
THEN claim.reminder = true
```

---

# ATTRIBUTES ENUMERATIONS

## VehicleType
- PASSENGER_CAR
- MOTORCYCLE
- TRUCK
- BUS
- TRACTOR
- TRAILER
- OTHER

## CoverageType
- THIRD_PARTY_LIABILITY
- COMPREHENSIVE
- FIRE
- THEFT
- GLASS
- NATURAL_DAMAGE
- LEGAL_EXPENSE

## DamageType
- COLLISION
- ROLLOVER
- FIRE
- THEFT
- VANDALISM
- NATURAL_DISASTER
- ANIMAL_HIT

## SeverityLevel
- MINOR
- MODERATE
- MAJOR
- TOTAL_LOSS

---

# REGULATORY CONCEPTS

## Key Legislation References
- Liikennevakuutuslaki (279/2011) - Traffic Insurance Act
- Laki liikennevakuutuksen korvausasian käsittelystä - Claims Processing Act
- Vakuutussopimuslaki - Insurance Contract Act

## Regulatory Bodies
- Finanssivalvonta (FIVA) - Financial Supervisory Authority
- Liikennevakuutuskeskus - Finnish Motor Insurance Centre

---

# EXAMPLE INSTANCES

## Example 1: Basic Traffic Insurance
```
Instance:Policy001
  type: MandatoryTrafficInsurance
  policyNumber: "TV-123456"
  insuranceCompany: IfVakuutus
  policyHolder: JohnDoe
  vehicle: ABC123
  premium: 300.00 EUR
  deductible: 150.00 EUR
```

## Example 2: Claim Process
```
Instance:Claim001
  claimId: "K-2024-001"
  claimDate: 2024-01-15
  claimant: JohnDoe
  insuranceEvent: Accident001
  damage: VehicleDamage001
  status: approved
  compensation: Compensation001
```

---

# TAXONOMY SUMMARY

```
Insurance
├── MandatoryTrafficInsurance (Liikennevakuutus)
│   └── ThirdPartyLiability
├── VoluntaryInsurance (Vapaaehtoinen vakuutus)
│   ├── Comprehensive (Kasko)
│   ├── Fire (Palovakuutus)
│   ├── Theft (Varkausvakuutus)
│   └── LegalProtection (Oikeusturvavakuutus)

Vehicle
├── MotorVehicle
│   ├── PassengerCar
│   ├── Motorcycle
│   ├── Truck
│   └── Bus
├── Trailer
└── AgriculturalVehicle

Damage
├── PropertyDamage
│   ├── VehicleDamage
│   └── ThirdPartyPropertyDamage
└── PersonalInjury
    ├── PhysicalInjury
    └── Death
```

---

# NOTES

1. This ontology is based on Finnish traffic insurance law (Liikennevakuutuslaki)
2. Mandatory liability insurance is governed by law and covers damages to third parties
3. Comprehensive insurance is voluntary but includes various coverage options
4. The deductible (omavastuu) varies by insurance type and driver history
5. Claims must be filed within specific timeframes prescribed by law
