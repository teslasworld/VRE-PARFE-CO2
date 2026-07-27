# VRE-PARFE-CO2e-Passport — Carbon Token Extension Set for IWA TTF

**Status**: ✅ **READY FOR IWA SUBMISSION**  
**Public Name**: `VRE-PARFE-CO2e-Passport`  
**Package ID**: `VRE-PARFE-ContinuousCO2e` (v1.0.0)  
**Standard**: GBBC/IWA dMRV 3.0  
**Issuing Authority**: Three T's (Mauritius) Limited (C19166743)  
**Current Deployment**: Hedera **Testnet**  
**Mainnet Migration**: 2026-11-17 @ 00:00 hrs MUT  
**Live Proof**: 2026-07-27 (Testnet) ✅  

---

## What This Is

VRE-PARFE-CO2e-Passport is a **production-ready extension set** for the GBBC/IWA Token Taxonomy Framework that enables **carbon token issuance from renewable energy evidence**.

### The Innovation

Instead of conflating evidence layers (a common mistake), this extension properly composes with VRE-PARFE-HydroRE:

```
┌─────────────────────────────────────────────────┐
│ Telemetry Layer (VRE-PARFE-HydroRE)            │
│ └─ Raw sensor readings + Ed25519 signature     │
│ └─ Physics gate (capacity limit validation)    │
│ └─ Aggregates to 1,000 kWh REC                 │
│ └─ Issues REC token (0.0.8585273)              │
└─────────────────────────────────────────────────┘
                        ↓ parentRecEvidence
┌─────────────────────────────────────────────────┐
│ CO2 Derivation Layer (VRE-PARFE-CO2e) ← NEW   │
│ └─ References parent REC (doesn't duplicate)  │
│ └─ Applies CO2 methodology scope (governed)    │
│ └─ Applies Grid Emission Factor (governed)     │
│ └─ Performs allocation reconciliation gate      │
│ └─ Issues CO2 tonne-lot (0.0.8585274)         │
│ └─ Lifecycle tracking (MINTED→TRANSFERRED→...) │
└─────────────────────────────────────────────────┘
```

**Key architectural principle**: CO2 does NOT re-consume raw telemetry. It verifies REC evidence, which in turn traces to telemetry if needed.

---

## Live Verification (Fresh Proof — 2026-07-27 Testnet)

### Complete Evidence Chain on Hedera Testnet

✅ **Governance** (Topic 0.0.8479702 testnet, Seq 86)  
- `MINTING_POLICY_GRANT` active for both REC and CO2 minting
- Both asset types enabled: `["REC", "CO2_TONNE_LOT"]`

✅ **Telemetry** (Topic 0.0.8585272 testnet, Seq 492363)  
- Single sensor reading: 81.78 kWh
- Ed25519 signature: `Pun060wg+...` (verifiable independently)
- Hash: `76fabf49e7fec97e1c1b59b8b9880a4273eaed6f42d6a7fba394e69366177cf7`

✅ **REC Aggregation** (Seq 492359–492362)  
- 154+ sensor readings accumulated
- Reaches 10,000 kWh threshold
- Minted on HTS: Token `0.0.8585273`, Serial `36799`

✅ **REC Passport** (Seq 492358)  
- Regulatory metadata: ISO 14064, EU-CBAM, US-GAAP, CORSIA  
- **Explicitly aligned**: `"iwa_gbbc_dmrv": "TTF dMRV 3.0"`  
- GEF: 0.9908 kgCO2e/kWh

✅ **CO2 Passport** (Seq 492364)  
- Derives from two parent RECs
- REC 1: 1,268 tCO2e (10,000 kWh @ 0.1268 kgCO2e/kWh)
- REC 2: 8,732 tCO2e (10,000 kWh @ 0.8732 kgCO2e/kWh)
- Governance anchor: Topic 0.0.8479702, Seq 84

✅ **CO2 Confirmation** (Seq 492365)  
- HTS token minted: `0.0.8585274`  
- NFT serial: `34113`  
- Quantity: 10,000 tCO2e  
- Status: `CO2_TONNE_LOT_MINTED`  

✅ **Allocation Reconciliation** (Seq 492366)  
- **Verified**: 1,268,000,000 + 8,732,000,000 = 10,000,000,000 mgCO2e ✓
- Both parent RECs valid and non-revoked
- Final status: `ALLOCATED_CO2`

### Verify Independently (Testnet)

**Browser** (Hashscan Testnet):
```
https://hashscan.io/testnet/topic/0.0.8585272?s=492366
```

**API** (Hedera Mirror Node Testnet):
```bash
curl -X GET "https://testnet.mirrornode.hedera.com/api/v1/topics/0.0.8585272/messages/492366"
```

**Details**: See `hcs-verification/HCS_PROOF_VERIFICATION.md`

---

## Deployment Timeline

| Phase | Network | Date | Status |
|-------|---------|------|--------|
| **Phase 1-5** | Testnet | 2026-07-27 | ✅ Live & Verified |
| **Mainnet Migration** | Testnet → Mainnet | 2026-11-17 00:00 MUT | ⏳ Scheduled |
| **Phase 6+** | Mainnet | Post-2026-11-17 | 📋 Planned |

---

## The Five-Envelope Model

| Envelope | Purpose | Coverage |
|----------|---------|----------|
| **standardEnvelope** | Extension set + methodology identity (VP-CO2E-RE-GRID-001, V1) | ✅ Complete |
| **externalMethodology** | CDM/Gold Standard reference (AMS-I.D) — explicit, governed, not defaulted | ✅ Complete |
| **continuousVerification** | CO2's own gates: REC validity, methodology scope, GEF scope, allocation reconciliation, audit replay | ✅ Complete |
| **auditAssurance** | Auditor role + HCS replay coordinates (does NOT include telemetry topic) | ✅ Complete |
| **issuedUnit** | CO2 tonne-lot HTS token, serial, lifecycle, retirement | ✅ Complete |

**Source of Truth**: All five envelopes are part of a canonical 21-key passport object on HCS.

---

## The 21-Key Passport Structure

```json
{
  "type": "CO2_REGULATORY_PASSPORT",
  "assetType": "CO2_TONNE_LOT",
  "co2ClaimId": "PC-MU-CO2-...",
  "lotId": "CO2-TONNE-...",
  "scopeKey": "...",
  "claim": { "methodologyId": "VP-CO2E-RE-GRID-001", ... },
  "accounting": { "boundary": "...", ... },
  "parentRecEvidence": [ { "claimId": "PC-MU-REC-...", "tokenId": "0.0.8585273", ... } ],
  "gefScope": { "gefValueKgPerKwh": 0.9908, ... },
  "methodScope": { "hcsTopicId": "0.0.8479702", "hcsSequence": "84", ... },
  "standardEnvelope": { "methodologyId": "VP-CO2E-RE-GRID-001", "extensionSetId": "VRE-PARFE-ContinuousCO2e", ... },
  "externalMethodology": { "methodologyId": "AMS-I.D", "validationRefCodes": ["ARTICLE_6_4_PRINCIPLES"], ... },
  "continuousVerification": { "recEvidenceValidityStatus": "PASSED", "allocationReconciliationStatus": "PASSED", ... },
  "auditAssurance": { "controlEvidence": { "governanceTopicId": "0.0.8479702", "mintEventsTopicId": "0.0.8585272", ... } },
  "issuedUnit": { "quantityMgCO2e": 10000000000, "tokenId": "0.0.8585274", "serial": "34113", ... },
  "allocations": [ { "parentRecClaimId": "PC-MU-REC-...", "consumedMgCO2e": 1268000000, ... } ],
  "netCo2eMg": 10000000000,
  "tonneLotMgCO2e": 10000000000,
  "status": "PASSPORT_SUBMITTED",
  "ts": "2026-07-27T11:09:23.890Z",
  "meta": { "issuer": "Three T's (Mauritius) Limited", "authorityRef": "0.0.8411690", ... }
}
```

**Abbreviation**: See `AbbreviationProfile.json` for Phase 5 (v1.0.0) compact aliases.

---

## Directory Structure

```
.
├── README.md (this file)
└── Vre Parfe GBBC-IWA Extension/
    ├── VRE-PARFE-ContinuousCO2e-SPEC.md          Main specification
    ├── PASSPORT_PRODUCTION_READINESS_ADDENDUM.md Phase 6/7 roadmap
    ├── MISSIONCONTROL_CO2_PASSPORT_ADOPTION_INSTRUCTIONS.md
    ├── IWA_SUBMISSION_NOTES.md                   ← Start here for IWA review
    ├── ABBREVIATION_AND_RECONSTRUCTION_TASK.md
    │
    ├── VRE-PARFE-ContinuousCO2e/
    │   ├── GettingStarted.md
    │   ├── DeploymentPackage/
    │   │   ├── VariableTemplates.json           69 entries (all 21 top-level keys)
    │   │   ├── FormulaTemplates.json            3 gate equations
    │   │   ├── EntityExtensionTemplates.json
    │   │   ├── ExtensionSet.json
    │   │   ├── MessagePairs.json
    │   │   ├── AbbreviationProfile.json         118+ aliases, Phase 5
    │   │   └── protos/
    │   │       └── vreParfeContinuousCO2e.proto  Complete, no truncation
    │   │
    │   ├── InstancePackage/
    │   │   ├── AimFixedVariables.json           Fixed parameters (GEF, capacity)
    │   │   └── ClaimSources.json
    │   │
    │   └── examples/
    │       ├── canonical_passport_sample.json    Full 21-key example
    │       └── abbreviated_passport_sample.json  Abbreviated v1 example
    │
    ├── hcs-verification/                        ← Fresh proof (2026-07-27 testnet)
    │   ├── HCS_PROOF_VERIFICATION.md            Complete evidence chain
    │   ├── allocation_reconciliation_proof.json  Live reconciliation (1,268 + 8,732 = 10,000)
    │   └── README.md                            How to independently verify
    │
    └── tools/
        └── record_reconstruction_tool.js        Canonical ↔ abbreviated conversion
```

---

## Standards & Regulatory Alignment

✅ **ISO 14064-1:2018** — GHG Quantification & Reporting  
✅ **EU-CBAM** — Carbon Border Adjustment Mechanism  
✅ **US-GAAP-S2** — Financial Accounting & Sustainability  
✅ **CORSIA** — Carbon Offsetting & Reduction Scheme  
✅ **IWA GBBC dMRV 3.0** — Token Taxonomy Framework ← **This extension set**  

---

## Key Design Decisions

### 1. No Telemetry Topic at CO2 Level
**Why**: CO2 derives from REC evidence, not telemetry. Proper auditor workflow is CO2 → REC → telemetry (via REC's passport).  
**Result**: `auditReplayStatus = "REPLAYABLE_VIA_REC"` (not direct telemetry)

### 2. External Methodology Is Governed
**Why**: Prevent silent substitution of incompatible methodologies (e.g., CDM → Gold Standard).  
**Result**: Runtime rejects missing/defaulted `externalMethodology.methodologyId`

### 3. Two Duplicate Fields Retained
**Why**: Schema backward compatibility (SCHEMAS_V2.3.js requires both).  
**Result**: Phase 6 can consolidate post-launch without breaking existing validators

---

## Phase Roadmap

### Phase 5 (Current) ✅ COMPLETE
**Key-Level Abbreviation** (v1.0.0)
- 15 root-level aliases
- 118+ scoped aliases
- Lossless reconstruction
- Zero evidence defaulting
- Tested on Testnet (2026-07-27)

### Phase 6 (Post-Mainnet Migration) ⏳ Proposed
**Content-Level Optimization**
- Omit null fields
- Deduplicate methodology
- Est. 40–50% payload reduction
- Target: Post-2026-11-17

### Phase 7 (Future) ⏳ Proposed
**Methodology Governance Reference**
- Static fields as HCS record
- Pointer-based reference
- Further compression

---

## For IWA Reviewers

**Start Here**:
1. Read `IWA_SUBMISSION_NOTES.md` (this directory)
2. Review `Vre Parfe GBBC-IWA Extension/VRE-PARFE-ContinuousCO2e-SPEC.md` (detailed spec)
3. Verify `hcs-verification/HCS_PROOF_VERIFICATION.md` (live testnet proof)

**Technical Details**:
- `DeploymentPackage/VariableTemplates.json` — All 69 variable definitions
- `DeploymentPackage/protos/vreParfeContinuousCO2e.proto` — Proto definitions (complete, no truncation)
- `examples/canonical_passport_sample.json` — 21-key sample
- `examples/abbreviated_passport_sample.json` — Phase 5 abbreviation sample

**Verification**:
- `hcs-verification/allocation_reconciliation_proof.json` — Live reconciliation math
- Hashscan (Testnet): https://hashscan.io/testnet/topic/0.0.8585272?s=492366
- API (Testnet): https://testnet.mirrornode.hedera.com/api/v1/topics/0.0.8585272/messages/492366
- **Note**: Testnet entity IDs; new entity IDs will be assigned on mainnet migration (2026-11-17)

---

## Recommended Next Steps

✅ **Accept this submission** — Production-ready, testnet-verified, standards-aligned  
✅ **Link from VRE-PARFE-HydroRE** — Point readers to this as downstream consumer  
✅ **Use as reference** — Template for future token extensions (H2O, carbon removal)  
✅ **Phase 6 roadmap** — Propose content-level optimization post-mainnet migration  

---

## Contact

**Issuing Authority**: Three T's (Mauritius) Limited  
**Registration No**: C19166743  
**GitHub**: https://github.com/teslasworld/VRE-PARFE-CO2  
**Hedera Entity**: 0.0.8411690 (Testnet; new entity ID on Mainnet)  

**Deployment Timeline**:
- **Testnet**: 2026-07-27 (current, proven)
- **Mainnet**: 2026-11-17 @ 00:00 hrs MUT (scheduled migration)

---

**Generated**: 2026-07-27  
**Current Network**: Hedera Testnet  
**Proof Valid Until**: Mainnet migration (2026-11-17) + indefinite on mainnet HCS  
**Status**: ✅ READY FOR IWA SUBMISSION (Testnet Proof — Mainnet scheduled 2026-11-17)
