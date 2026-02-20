# Car Insurance Business Rules
# Based on Finnish Traffic Insurance Act (Liikennevakuutuslaki)

## 1. MANDATORY INSURANCE RULES

### Rule 1.1: Mandatory Insurance Requirement
```json
{
  "rule_id": "INS-001",
  "name": "MandatoryTrafficInsuranceRequired",
  "description": "All vehicles registered in Finland must have mandatory traffic insurance",
  "condition": "vehicle.registrationRequired = true AND vehicle.registeredIn = 'FI'",
  "action": "REQUIRE insurance.type = MandatoryTrafficInsurance",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 2§"
}
```

### Rule 1.2: New Vehicle Insurance Deadline
```json
{
  "rule_id": "INS-002",
  "name": "NewVehicleInsuranceDeadline",
  "description": "Insurance must be obtained before vehicle registration",
  "condition": "vehicle.registrationPending = true",
  "action": "REQUIRE insurance.active = true BEFORE vehicle.registrationDate",
  "deadline": "vehicle.registrationDate",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 3§"
}
```

---

## 2. PREMIUM CALCULATION RULES

### Rule 2.1: Base Premium by Vehicle Type
```json
{
  "rule_id": "PRE-001",
  "name": "VehicleTypeBasePremium",
  "description": "Base premium determined by vehicle type",
  "condition": "vehicle.type = vehicleType",
  "action": "SET premium.base = vehicleType.rateTable",
  "factors": {
    "PASSENGER_CAR": 300,
    "MOTORCYCLE": 200,
    "TRUCK": 500,
    "BUS": 600,
    "TRACTOR": 150
  },
  "legal_reference": "Liikennevakuutuslaki 8§"
}
```

### Rule 2.2: Driver Age Premium Adjustment
```json
{
  "rule_id": "PRE-002",
  "name": "YoungDriverPremiumSurcharge",
  "description": "Additional premium for young drivers under 25",
  "condition": "driver.age < 25 AND insurance.type = Comprehensive",
  "action": "APPLY premium.multiplier = 1.5",
  "severity": "HIGH"
}
```

### Rule 2.3: Driving Experience Discount
```json
{
  "rule_id": "PRE-003",
  "name": "DrivingExperienceDiscount",
  "description": "Discount for experienced drivers",
  "condition": "driver.yearsOfExperience >= 10 AND claimHistory.clean = true",
  "action": "APPLY premium.discount = 0.8",
  "severity": "MEDIUM"
}
```

### Rule 2.4: No Claims Bonus
```json
{
  "rule_id": "PRE-004",
  "name": "NoClaimsBonus",
  "description": "Progressive discount for claim-free years",
  "condition": "policy.claimFreeYears >= n",
  "action": "APPLY premium.discount = noClaimsTable[n]",
  "table": {
    "1": 0.90,
    "2": 0.85,
    "3": 0.80,
    "4": 0.75,
    "5": 0.70
  },
  "legal_reference": "Liikennevakuutuslaki 9§"
}
```

---

## 3. COVERAGE RULES

### Rule 3.1: Mandatory Coverage - Third Party Liability
```json
{
  "rule_id": "COV-001",
  "name": "ThirdPartyLiabilityCoverage",
  "description": "Mandatory coverage for damages to third parties",
  "condition": "insurance.type = MandatoryTrafficInsurance",
  "coverage": {
    "property_damage": {
      "maximum": 5000000,
      "currency": "EUR"
    },
    "personal_injury": {
      "maximum": 10000000,
      "currency": "EUR"
    }
  },
  "legal_reference": "Liikennevakuutuslaki 12§"
}
```

### Rule 3.2: Comprehensive Coverage Option
```json
{
  "rule_id": "COV-002",
  "name": "ComprehensiveCoverageIncludes",
  "description": "Full comprehensive includes all damage to own vehicle",
  "condition": "insurance.coverage = FullComprehensive",
  "coverage": {
    "own_vehicle_damage": true,
    "fire_damage": true,
    "theft": true,
    "glass_damage": true,
    "natural_damage": true,
    "vandalism": true
  },
  "exclusions": [
    "intent",
    "driving_without_license",
    "driving_under_influence"
  ],
  "legal_reference": "Liikennevakuutuslaki 15§"
}
```

### Rule 3.3: Fire and Theft Coverage
```json
{
  "rule_id": "COV-003",
  "name": "FireAndTheftCoverage",
  "description": "Fire and theft coverage for vehicles",
  "condition": "insurance.coverage = FireAndTheft",
  "coverage": {
    "fire_damage": true,
    "theft": true,
    "theft_of_parts": true
  },
  "exclusions": [
    "vehicle_left_unlocked",
    "keys_left_in_vehicle"
  ],
  "legal_reference": "Liikennevakuutuslaki 16§"
}
```

---

## 4. DEDUCTIBLE RULES

### Rule 4.1: Standard Deductible
```json
{
  "rule_id": "DED-001",
  "name": "StandardDeductibleAmount",
  "description": "Base deductible amount by insurance type",
  "condition": "insurance.type = insuranceType",
  "action": "SET deductible.amount = deductibleTable[insuranceType]",
  "table": {
    "MandatoryTrafficInsurance": 0,
    "PartialComprehensive": 200,
    "FullComprehensive": 150
  },
  "legal_reference": "Liikennevakuutuslaki 18§"
}
```

### Rule 4.2: Young Driver Deductible Increase
```json
{
  "rule_id": "DED-002",
  "name": "YoungDriverDeductibleIncrease",
  "description": "Increased deductible for drivers under 25",
  "condition": "driver.age < 25 AND insurance.type = Comprehensive",
  "action": "SET deductible.amount = deductible.amount * 1.5",
  "minimum": 300,
  "severity": "HIGH"
}
```

### Rule 4.3: drunken Driving Deductible
```json
{
  "rule_id": "DED-003",
  "name": "DrunkenDrivingDeductible",
  "description": "Maximum deductible when driving under influence",
  "condition": "event.driver.intoxicated = true",
  "action": "SET deductible.amount = 2000",
  "note": "Insurance still covers third party, but own damage has high deductible",
  "legal_reference": "Liikennevakuutuslaki 19§"
}
```

---

## 5. CLAIM PROCESSING RULES

### Rule 5.1: Claim Filing Deadline
```json
{
  "rule_id": "CLM-001",
  "name": "ClaimFilingDeadline",
  "description": "Claim must be filed within 30 days of incident",
  "condition": "claim.date - event.date > 30 days",
  "action": "IF exceeded THEN require justification ELSE accept",
  "grace_period_days": 30,
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 32§"
}
```

### Rule 5.2: Claim Investigation Period
```json
{
  "rule_id": "CLM-002",
  "name": "ClaimInvestigationPeriod",
  "description": "Insurance company must process claim within 30 days",
  "condition": "claim.status = under_review",
  "action": "SET deadline = claim.receivedDate + 30 days",
  "action_extension": "IF complex THEN notify claimant AND extend to 60 days",
  "severity": "MEDIUM",
  "legal_reference": "Liikennevakuutuslaki 33§"
}
```

### Rule 5.3: Required Claim Documentation
```json
{
  "rule_id": "CLM-003",
  "name": "RequiredClaimDocuments",
  "description": "Minimum documentation for claim processing",
  "condition": "claim.type = claimType",
  "required_documents": [
    "police_report (if involving third party)",
    "damage_photos",
    "repair_estimate",
    "vehicle_registration",
    "driver_license",
    "insurance_policy_number"
  ],
  "action": "IF documents incomplete THEN request additional_info",
  "severity": "HIGH"
}
```

---

## 6. COMPENSATION RULES

### Rule 6.1: Property Damage Compensation Limit
```json
{
  "rule_id": "CMP-001",
  "name": "PropertyDamageCompensationLimit",
  "description": "Maximum compensation for property damage",
  "condition": "damage.type = PropertyDamage",
  "action": "SET compensation.maximum = MIN(damage.actualCost, coverage.limit)",
  "limits": {
    "mandatory_insurance": 5000000,
    "comprehensive": 50000
  },
  "legal_reference": "Liikennevakuutuslaki 40§"
}
```

### Rule 6.2: Total Loss Compensation
```json
{
  "rule_id": "CMP-002",
  "name": "TotalLossCompensation",
  "description": "Compensation when vehicle is total loss",
  "condition": "damage.severity = TotalLoss AND repairCost > vehicle.marketValue * 0.8",
  "action": "SET compensation.amount = vehicle.marketValue - deductible",
  "note": "Market value at time of accident, not original purchase price",
  "legal_reference": "Liikennevakuutuslaki 42§"
}
```

### Rule 6.3: Personal Injury Compensation
```json
{
  "rule_id": "CMP-003",
  "name": "PersonalInjuryCompensation",
  "description": "Comprehensive compensation for personal injuries",
  "condition": "damage.type = PersonalInjury",
  "compensation_types": [
    "medical_expenses",
    "lost_wages",
    "rehabilitation",
    "pain_and_suffering",
    "permanent_disability"
  ],
  "action": "CALCULATE compensation = SUM(all_types)",
  "legal_reference": "Liikennevakuutuslaki 45-50§"
}
```

### Rule 6.4: Depreciation on Compensation
```json
{
  "rule_id": "CMP-004",
  "name": "VehicleDepreciation",
  "description": "Apply depreciation to older vehicles",
  "condition": "vehicle.age > 5 years AND damage.type = PropertyDamage",
  "action": "APPLY depreciation = vehicle.age * 0.10",
  "maximum_depreciation": 0.70,
  "note": "Maximum 70% depreciation applied"
}
```

---

## 7. EXCLUSION RULES

### Rule 7.1: Driving Without License
```json
{
  "rule_id": "EXC-001",
  "name": "ExclusionNoDrivingLicense",
  "description": "No coverage if driver has no valid license",
  "condition": "driver.license.valid = false",
  "action": "DENY compensation FOR own_vehicle",
  "note": "Third party coverage still applies",
  "legal_reference": "Liikennevakuutuslaki 55§"
}
```

### Rule 7.2: Driving Under Influence
```json
{
  "rule_id": "EXC-002",
  "name": "ExclusionDrivingUnderInfluence",
  "description": "Limited coverage when driving intoxicated",
  "condition": "driver.intoxicated = true OR driver.drugs = true",
  "action": "REDUCE compensation = 50% FOR own_vehicle",
  "note": "Third party still covered in full",
  "legal_reference": "Liikennevakuutuslaki 56§"
}
```

### Rule 7.3: Intentional Damage
```json
{
  "rule_id": "EXC-003",
  "name": "ExclusionIntentionalDamage",
  "description": "No coverage for intentional damage",
  "condition": "event.intentional = true",
  "action": "DENY compensation COMPLETELY",
  "legal_reference": "Liikennevakuutuslaki 57§"
}
```

### Rule 7.4: Racing Excluded
```json
{
  "rule_id": "EXC-004",
  "name": "ExclusionRacing",
  "description": "No coverage during racing or speed competition",
  "condition": "vehicle.purpose = racing OR location = racetrack",
  "action": "DENY compensation",
  "legal_reference": "Liikennevakuutuslaki 58§"
}
```

### Rule 7.5: War and Natural Disaster Exclusions
```json
{
  "rule_id": "EXC-005",
  "name": "ExclusionWarDamage",
  "description": "No coverage for war or act of war damages",
  "condition": "event.cause = war OR event.cause = act_of_war",
  "action": "DENY compensation",
  "legal_reference": "Liikennevakuutuslaki 59§"
}
```

---

## 8. INSURANCE COMPANY OBLIGATIONS

### Rule 8.1: Minimum Settlement Period
```json
{
  "rule_id": "OBL-001",
  "name": "MinimumSettlementPeriod",
  "description": "Company must make offer within 30 days",
  "condition": "claim.all_documents_received = true",
  "action": "SET deadline = receivedDate + 30 days",
  "action_extension": "IF disputed THEN pay_minimum_within_30_days",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 35§"
}
```

### Rule 8.2: Right of Subrogation
```json
{
  "rule_id": "OBL-002",
  "name": "SubrogationRight",
  "description": "Insurance can recover from responsible third party",
  "condition": "compensation.paid = true AND third_party.responsible = true",
  "action": "EXERCISE right_of_subrogation AGAINST third_party",
  "note": "Does not affect claimant's compensation",
  "legal_reference": "Liikennevakuutuslaki 75§"
}
```

---

## 9. POLICY renewal RULES

### Rule 9.1: Policy Renewal Notice
```json
{
  "rule_id": "REN-001",
  "name": "PolicyRenewalNotice",
  "description": "Insurer must notify renewal 30 days before expiry",
  "condition": "policy.endDate - currentDate <= 30 days",
  "action": "SEND renewal_notice TO policyHolder",
  "notice_period_days": 30,
  "severity": "MEDIUM",
  "legal_reference": "Vakuutussopimuslaki 12§"
}
```

### Rule 9.2: Grace Period After Expiry
```json
{
  "rule_id": "REN-002",
  "name": "GracePeriodAfterExpiry",
  "description": "Short grace period for late renewal",
  "condition": "policy.expired = true AND daysSinceExpiry <= 14",
  "action": "ALLOW renewal WITHOUT new_medical_examination",
  "grace_period_days": 14,
  "legal_reference": "Vakuutussopimuslaki 13§"
}
```

---

## 10. DISPUTE RESOLUTION RULES

### Rule 10.1: Appeal Process
```json
{
  "rule_id": "DIS-001",
  "name": "ClaimAppealProcess",
  "description": "Right to appeal denied claims",
  "condition": "claim.status = rejected",
  "action": "INFORM claimant OF appeal_rights AND deadline",
  "appeal_deadline_days": 30,
  "escalation": "InsuranceCourt (Vakuutusoikeus)",
  "legal_reference": "Liikennevakuutuslaki 80§"
}
```

---

## RULE SUMMARY TABLE

| Category | Rule ID | Name | Severity |
|----------|---------|------|----------|
| Mandatory | INS-001 | Mandatory Insurance Required | CRITICAL |
| Mandatory | INS-002 | New Vehicle Insurance Deadline | CRITICAL |
| Premium | PRE-001 | Vehicle Type Base Premium | HIGH |
| Premium | PRE-002 | Young Driver Surcharge | HIGH |
| Premium | PRE-003 | Experience Discount | MEDIUM |
| Premium | PRE-004 | No Claims Bonus | MEDIUM |
| Coverage | COV-001 | Third Party Liability | CRITICAL |
| Coverage | COV-002 | Comprehensive Coverage | HIGH |
| Coverage | COV-003 | Fire and Theft | MEDIUM |
| Deductible | DED-001 | Standard Deductible | HIGH |
| Deductible | DED-002 | Young Driver Increase | HIGH |
| Deductible | DED-003 | Drunken Driving | CRITICAL |
| Claims | CLM-001 | Filing Deadline | HIGH |
| Claims | CLM-002 | Investigation Period | MEDIUM |
| Claims | CLM-003 | Required Documents | HIGH |
| Compensation | CMP-001 | Property Damage Limit | HIGH |
| Compensation | CMP-002 | Total Loss | HIGH |
| Compensation | CMP-003 | Personal Injury | HIGH |
| Compensation | CMP-004 | Depreciation | MEDIUM |
| Exclusions | EXC-001 | No License | CRITICAL |
| Exclusions | EXC-002 | Under Influence | CRITICAL |
| Exclusions | EXC-003 | Intentional | CRITICAL |
| Exclusions | EXC-004 | Racing | HIGH |
| Exclusions | EXC-005 | War Damage | HIGH |
