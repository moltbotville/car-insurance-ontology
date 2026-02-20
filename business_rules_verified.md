# Car Insurance Business Rules - Verified
# Based on Finnish Traffic Insurance Act (Liikennevakuutuslaki) 460/2016
# Current as of: 16.1.2026/47 (amendments included)

> **LEGAL DISCLAIMER**: These rules are derived directly from the official Finnish law text.
> Some euro amounts may be subject to adjustment by government decree.

---

## 1. MANDATORY INSURANCE RULES

### Rule 1.1: Mandatory Insurance Requirement
```json
{
  "rule_id": "INS-001",
  "name": "MandatoryTrafficInsuranceRequired",
  "description": "All vehicles with permanent location in Finland must have mandatory traffic insurance",
  "condition": "vehicle.permanentLocation = 'FI' AND vehicle.requiresInsurance = true",
  "action": "REQUIRE insurance.type = 'TrafficInsurance'",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 5 §"
}
```

### Rule 1.2: Owner and Holder Jointly Liable
```json
{
  "rule_id": "INS-002",
  "name": "OwnerAndHolderJointlyLiable",
  "description": "Vehicle owner and holder are jointly responsible for insurance",
  "condition": "vehicle.owner AND vehicle.holder",
  "action": "BOTH liable for insurance obligation",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 6 §"
}
```

### Rule 1.3: Insurance Obligation Begins at Ownership Transfer
```json
{
  "rule_id": "INS-003",
  "name": "InsuranceObligationStartsAtOwnership",
  "description": "Insurance obligation begins from the day of ownership or possession transfer",
  "condition": "vehicle.ownershipTransferred = true",
  "action": "insurance.mustBeTaken WITHIN 0 days",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 6 §"
}
```

### Rule 1.4: EEA Vehicle Import - Choice of Jurisdiction
```json
{
  "rule_id": "INS-004",
  "name": "EEAVehicleInsuranceChoice",
  "description": "When importing from another EEA country, buyer can choose where to insure",
  "condition": "vehicle.importedFromEEA = true",
  "action": "vehicle.insuranceCountry = CHOICE(vehicle.registrationCountry, 'FI')",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 6a §"
}
```

### Rule 1.5: Border Traffic Insurance Required
```json
{
  "rule_id": "INS-005",
  "name": "BorderTrafficInsuranceRequired",
  "description": "Vehicles from third countries require border traffic insurance for temporary use in Finland",
  "condition": "vehicle.permanentLocation = 'ThirdCountry' AND vehicle.temporaryUseInFinland = true",
  "action": "REQUIRE insurance.type = 'BorderTrafficInsurance'",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 7 §"
}
```

### Rule 1.6: Exceptions from Insurance Obligation
```json
{
  "rule_id": "INS-006",
  "name": "InsuranceExceptions",
  "description": "Certain vehicles are exempt from mandatory insurance",
  "condition": "vehicle.type IN ['AgriculturalMachine', 'Trailer', 'Wheelchair', 'GovernmentVehicle']",
  "action": "insurance.optional = true",
  "severity": "MEDIUM",
  "legal_reference": "Liikennevakuutuslaki 8 §",
  "notes": "Includes: tractors ≤15km/h, agricultural machines, unregistered trailers, wheelchairs, government vehicles"
}
```

---

## 2. PREMIUM RULES

### Rule 2.1: Premium Calculation Principles
```json
{
  "rule_id": "PRE-001",
  "name": "PremiumMustBeReasonable",
  "description": "Premiums must be in reasonable proportion to expected costs",
  "condition": "true",
  "action": "premium.mustBeProportionalToExpectedCosts",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 20 §"
}
```

### Rule 2.2: Claims History Impact on Premium
```json
{
  "rule_id": "PRE-002",
  "name": "ClaimsHistoryAffectsPremium",
  "description": "Premium calculation must include claims history for cars, vans, motorcycles, buses, trucks",
  "condition": "vehicle.type IN ['Car', 'Van', 'RV', 'Bus', 'Truck', 'Motorcycle']",
  "action": "premium.adjustByClaimsHistory",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 20 §"
}
```

### Rule 2.3: Premium Refund on Early Termination
```json
{
  "rule_id": "PRE-003",
  "name": "PremiumRefundOnTermination",
  "description": "If insurance ends early, premium refund for unused period",
  "condition": "insurance.terminatedEarly = true",
  "action": "refund premium for unused coverage period",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 23 §",
  "notes": "Minimum refund threshold: €8 (adjustable by decree)"
}
```

### Rule 2.4: Increased Premium for Vehicle Used During Traffic Exemption
```json
{
  "rule_id": "PRE-004",
  "name": "IncreasedPremiumForUnreportedUse",
  "description": "If vehicle used in traffic while registered as temporarily removed, premium up to 3x",
  "condition": "vehicle.registeredAsRemovedFromTraffic AND vehicle.usedInTraffic = true",
  "action": "premium.multiplier = UP_TO 3.0",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 22 §"
}
```

### Rule 2.5: Default Interest on Unpaid Premium
```json
{
  "rule_id": "PRE-005",
  "name": "DefaultInterestOnUnpaidPremium",
  "description": "Interest on unpaid premium according to Interest Act",
  "condition": "premium.unpaid = true",
  "action": "charge interest at Interest Act rate",
  "severity": "MEDIUM",
  "legal_reference": "Liikennevakuutuslaki 24 §"
}
```

### Rule 2.6: Premium Collectible by Distraint
```json
{
  "rule_id": "PRE-006",
  "name": "PremiumDistrainable",
  "description": "Premium with interest is directly distrainable",
  "condition": "premium.unpaid AND premium.withInterest",
  "action": "distrainable = true",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 25 §"
}
```

### Rule 2.7: Premium-Equivalent Payment for Uninsured
```json
{
  "rule_id": "PRE-007",
  "name": "PremiumEquivalentPayment",
  "description": "Uninsured vehicle owner must pay premium-equivalent for uncovered period",
  "condition": "vehicle.insured = false AND vehicle.requiredInsurance = true",
  "action": "pay premiumEquivalent for period without insurance",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 27 §",
  "notes": "Maximum 5 years back + current year"
}
```

### Rule 2.8: Penalty Fee for Uninsured
```json
{
  "rule_id": "PRE-008",
  "name": "PenaltyFeeForNoInsurance",
  "description": "Penalty fee up to 3x premium for failure to insure",
  "condition": "insurance.obligationBreached = true",
  "action": "penaltyFee = UP_TO premiumEquivalent * 3",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 28 §"
}
```

---

## 3. COVERAGE RULES

### Rule 3.1: No-Fault Compensation
```json
{
  "rule_id": "COV-001",
  "name": "NoFaultCompensation",
  "description": "Traffic accidents compensated regardless of tort liability",
  "condition": "event.type = 'TrafficAccident'",
  "action": "compensate = true",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 31 §"
}
```

### Rule 3.2: Validity Throughout EEA
```json
{
  "rule_id": "COV-002",
  "name": "EEACoverage",
  "description": "Single premium covers all EEA countries",
  "condition": "premium.paid = true",
  "action": "coverage.validInAllEEA = true",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 13 §"
}
```

### Rule 3.3: Property Damage Compensation
```json
{
  "rule_id": "COV-003",
  "name": "PropertyDamageCompensation",
  "description": "Property damage compensated according to Tort Liability Act chapter 5 §5",
  "condition": "event.type = 'PropertyDamage'",
  "action": "compensate = repairCost OR fairMarketValue",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 37 §"
}
```

### Rule 3.4: Property Damage Maximum
```json
{
  "rule_id": "COV-004",
  "name": "PropertyDamageMaximum",
  "description": "Maximum property damage compensation €5,000,000 per insurance",
  "condition": "event.type = 'PropertyDamage'",
  "action": "maxCompensation = 5000000 EUR",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 38 §"
}
```

### Rule 3.5: Personal Injury Compensation
```json
{
  "rule_id": "COV-005",
  "name": "PersonalInjuryCompensation",
  "description": "Personal injury compensated according to Tort Liability Act chapters 5 and 7",
  "condition": "event.type = 'PersonalInjury'",
  "action": "compensate according to TortLiabilityAct",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 34 §"
}
```

### Rule 3.6: Continuous Compensation Index Adjustment
```json
{
  "rule_id": "COV-006",
  "name": "IndexAdjustment",
  "description": "Continuous compensations adjusted annually by employee pension index",
  "condition": "compensation.type = 'Continuous'",
  "action": "adjustByIndex ANNUALLY",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 35 §"
}
```

---

## 4. DEDUCTIBLE & EXCLUSION RULES

### Rule 4.1: Self-Caused Accident Reduction
```json
{
  "rule_id": "EXC-001",
  "name": "SelfCausedAccidentReduction",
  "description": "If victim caused accident through gross negligence, compensation may be reduced or denied",
  "condition": "victim.grossNegligence = true",
  "action": "compensate REDUCE OR DENY",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 47 §"
}
```

### Rule 4.2: Alcohol/Drug Influence
```json
{
  "rule_id": "EXC-002",
  "name": "AlcoholReduction",
  "description": "If driver has BAC ≥1.2‰ or impaired ability, compensation reduced based on contribution",
  "condition": "driver.BAC >= 1.2 OR driver.impaired = true",
  "action": "reduce compensation BY driverContribution",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 48 §"
}
```

### Rule 4.3: Alcohol (Lower Threshold)
```json
{
  "rule_id": "EXC-003",
  "name": "AlcoholReductionLower",
  "description": "If driver BAC 0.5-1.19‰, compensation reduced proportionally",
  "condition": "driver.BAC >= 0.5 AND driver.BAC < 1.2",
  "action": "reduce compensation proportionally",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 48 §"
}
```

### Rule 4.4: Illegal Vehicle Use
```json
{
  "rule_id": "EXC-004",
  "name": "IllegalUserCompensation",
  "description": "Person in illegally taken vehicle compensated only for special reason if knew vehicle was illegal",
  "condition": "person.inIllegalVehicle AND person.knewIllegal = true",
  "action": "compensate ONLY for specialReason",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 49 §"
}
```

### Rule 4.5: Damage to Own Vehicle Excluded
```json
{
  "rule_id": "EXC-005",
  "name": "OwnVehicleExcluded",
  "description": "Damage to the insured vehicle itself not covered",
  "condition": "damage.target = insuredVehicle",
  "action": "compensate = false",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 40 §"
}
```

### Rule 4.6: Damage During Work Excluded
```json
{
  "rule_id": "EXC-006",
  "name": "WorkExclusion",
  "description": "Damage during loading/unloading to persons or property involved in work excluded",
  "condition": "event.occurredDuring = 'LoadingOrUnloading' AND damage.target = 'WorkParticipant'",
  "action": "compensate = false",
  "severity": "MEDIUM",
  "legal_reference": "Liikennevakuutuslaki 42 §"
}
```

### Rule 4.7: Insanity/Emergency Exception
```json
{
  "rule_id": "EXC-007",
  "name": "InsanityException",
  "description": "No reduction/refusal if victim under 12, insane, or acting in emergency",
  "condition": "victim.age < 12 OR victim.insane OR victim.emergency = true",
  "action": "compensate FULL",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 50 §"
}
```

---

## 5. CLAIMS PROCEDURE RULES

### Rule 5.1: Direct Claim Right
```json
{
  "rule_id": "CLM-001",
  "name": "DirectClaimRight",
  "description": "Victim can claim directly from insurance company",
  "condition": "victim.claiming = true",
  "action": "claim可直接向保险公司提出",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 60 §"
}
```

### Rule 5.2: Claims Time Limit
```json
{
  "rule_id": "CLM-002",
  "name": "ClaimsTimeLimit",
  "description": "Claim must be filed within 3 years of knowing about accident, max 10 years",
  "condition": "claim.filed = true",
  "action": "deadline = 3 years (knowledge) OR 10 years (max)",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 61 §"
}
```

### Rule 5.3: Investigation Must Start Within 7 Days
```json
{
  "rule_id": "CLM-003",
  "name": "InvestigationStartDeadline",
  "description": "Insurance company must start investigation within 7 business days",
  "condition": "claim.received = true",
  "action": "investigation.start WITHIN 7 business days",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 62 §"
}
```

### Rule 5.4: Compensation Payment Deadline
```json
{
  "rule_id": "CLM-004",
  "name": "CompensationDeadline",
  "description": "Compensation must be paid within 1 month of receiving required documents",
  "condition": "documents.complete = true",
  "action": "pay within 1 month",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 62 §"
}
```

### Rule 5.5: Delay Interest
```json
{
  "rule_id": "CLM-005",
  "name": "DelayInterest",
  "description": "Delayed compensation interest at Interest Act rate + increase",
  "condition": "compensation.paidLate = true",
  "action": "pay delayInterestAtRate PLUS increase",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 67 §"
}
```

### Rule 5.6: Mandatory Board Recommendation
```json
{
  "rule_id": "CLM-006",
  "name": "BoardRecommendationRequired",
  "description": "Must request Traffic and Patient Injury Board recommendation for certain complex cases",
  "condition": "claim.type IN ['PermanentLossOfEarnings', 'Death', 'SevereDisability', 'IncreasedCompensation']",
  "action": "request boardRecommendation REQUIRED",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 66 §"
}
```

### Rule 5.7: Claim Limitation Period
```json
{
  "rule_id": "CLM-007",
  "name": "CourtClaimLimitation",
  "description": "Court action must be filed within 3 years of decision",
  "condition": "claim.disputed = true",
  "action": "sue within 3 years of decision",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 79 §"
}
```

---

## 6. TRAFFIC INSURANCE CENTRE RULES

### Rule 6.1: Centre Covers Uninsured Vehicle Damage
```json
{
  "rule_id": "CEN-001",
  "name": "CentreCoversUninsured",
  "description": "Traffic Insurance Centre compensates damage caused by uninsured vehicle",
  "condition": "vehicle.insured = false AND vehicle.shouldBeInsured = true",
  "action": "centre.compensates",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 46 §"
}
```

### Rule 6.2: Centre Covers Unknown Vehicle Damage
```json
{
  "rule_id": "CEN-002",
  "name": "CentreCoversUnknownVehicle",
  "description": "Centre compensates personal injury from unknown vehicle",
  "condition": "vehicle.unknown = true AND event.type = 'PersonalInjury'",
  "action": "centre.compensatesPersonalInjury",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 44 §"
}
```

### Rule 6.3: Centre Covers Exempt Vehicle Damage
```json
{
  "rule_id": "CEN-003",
  "name": "CentreCoversExemptVehicle",
  "description": "Centre compensates damage from exempt vehicles (agricultural, etc.)",
  "condition": "vehicle.exempt = true AND vehicle.permanentLocation = 'FI'",
  "action": "centre.compensates",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 43 §"
}
```

---

## 7. INSURANCE COMPANY OBLIGATIONS

### Rule 7.1: Cannot Refuse Insurance
```json
{
  "rule_id": "ICO-001",
  "name": "CannotRefuseInsurance",
  "description": "Insurance company cannot refuse to provide insurance for eligible vehicle",
  "condition": "company.licensed = true AND vehicle.eligible = true",
  "action": "MUST provide insurance",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 17 §"
}
```

### Rule 7.2: Must Issue Green Card
```json
{
  "rule_id": "ICO-002",
  "name": "MustIssueGreenCard",
  "description": "Company must issue green card (international insurance certificate) on request",
  "condition": "customer.requestedGreenCard = true",
  "action": "issue greenCard",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 17 §"
}
```

### Rule 7.3: Claims History Certificate
```json
{
  "rule_id": "ICO-003",
  "name": "ClaimsHistoryCertificate",
  "description": "Must provide claims history certificate within 15 days of request",
  "condition": "customer.requestedCertificate = true",
  "action": "issue certificate WITHIN 15 days",
  "severity": "MEDIUM",
  "legal_reference": "Liikennevakuutuslaki 19 §"
}
```

### Rule 7.4: Document Retention
```json
{
  "rule_id": "ICO-004",
  "name": "DocumentRetention",
  "description": "Must retain personal injury documents 100 years, other documents 10 years",
  "condition": "true",
  "action": "retain documents 100 years (injury), 10 years (other)",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 85 §"
}
```

---

## 8. VEHICLE USAGE RULES

### Rule 8.1: Usage Prohibition for Uninsured
```json
{
  "rule_id": "VEH-001",
  "name": "UsageProhibition",
  "description": "Uninsured vehicle usage in traffic is prohibited",
  "condition": "vehicle.insured = false AND vehicle.usedInTraffic = true",
  "action": "usage = PROHIBITED",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 30 §"
}
```

### Rule 8.2: Continuity After Ownership Change
```json
{
  "rule_id": "VEH-002",
  "name": "ContinuityAfterSale",
  "description": "Insurance covers 7 days after ownership transfer if new owner doesn't insure",
  "condition": "vehicle.sold = true AND newOwner.notInsured = true",
  "action": "coverage extends 7 days",
  "severity": "MEDIUM",
  "legal_reference": "Liikennevakuutuslaki 18 §"
}
```

---

## 9. INTERNATIONAL RULES

### Rule 9.1: EEA Citizen Choice of Law
```json
{
  "rule_id": "INT-001",
  "name": "EEACitizenChoiceOfLaw",
  "description": "Finnish resident victim can choose Finnish law for accident in another EEA country",
  "condition": "victim.resident = 'FI' AND accident.country IN EEA AND accident.country != 'FI'",
  "action": "victim.canChoose FinnishLaw",
  "severity": "MEDIUM",
  "legal_reference": "Liikennevakuutuslaki 13 §"
}
```

### Rule 9.2: Foreign Vehicle - Finnish Accident
```json
{
  "rule_id": "INT-002",
  "name": "ForeignVehicleFinnishAccident",
  "description": "Foreign vehicle accident in Finland - Finnish minimum coverage applies",
  "condition": "vehicle.registeredOutsideFI AND accident.country = 'FI'",
  "action": "apply FINNISH minimum standards",
  "severity": "HIGH",
  "legal_reference": "Liikennevakuutuslaki 13 §"
}
```

---

## 10. INSOLVENCY RULES

### Rule 10.1: Centre Covers in Company Insolvency
```json
{
  "rule_id": "INS-001",
  "name": "CentreCoversInsolvency",
  "description": "Traffic Insurance Centre compensates if insurance company becomes insolvent",
  "condition": "company.insolvent = true AND victim.resident = 'FI'",
  "action": "centre.compensates",
  "severity": "CRITICAL",
  "legal_reference": "Liikennevakuutuslaki 91a §"
}
```

---

## Key Definitions (from §2)

- **Ajoneuvo (Vehicle)**: Motor vehicle moving mechanically on land (not rails), speed >25km/h or weight >25kg, plus rented e-scooters and trailers
- **Vakuutusyhtiö (Insurance Company)**: Company licensed for insurance under directive 2009/103/EY
- **Vakuutuksenottaja (Policyholder)**: Person who made insurance contract
- **Vakuutettu (Insured)**: Person for whose benefit insurance is valid
- **Liikennevahinko (Traffic Accident)**: Personal injury or property damage from vehicle use in traffic
- **ETA-valtio (EEA State)**: Country belonging to European Economic Area
- **Kolmas maa (Third Country)**: Country outside EEA

---

## Amendments Included

- 279/2011 (original)
- 333/2018
- 983/2018
- 951/2019
- 960/2019
- 244/2020
- 92/2021
- 566/2022
- 794/2022
- 718/2023
- 218/2024
- 598/2024
- 955/2024
- 47/2026 (effective 19.6.2026)
