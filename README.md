# VRE-PARFE-CO2e-Passport  -  Extension Set Spec

**Adopted title:** Vre Parfe CO2 Passport
**Public IWA extension set name:** `VRE-PARFE-CO2e-Passport`
**Current local package/schema ID:** `VRE-PARFE-ContinuousCO2e`
**Status:** Local submission package prepared for manual file selection into the IWA pull request. Cross-referenced against live on-chain data and the current `MISSIONCONTROL_LIVE.js` source. No production file has been modified to produce this document.

## Submission decisions

- Use `VRE-PARFE-CO2e-Passport` as the public IWA-facing extension set name.
- Keep `VRE-PARFE-ContinuousCO2e` as the current local package and schema ID until the next naming pass.
- Use ASCII-safe `Vre Parfe` text in submitted files unless IWA explicitly requests UTF-8 accented branding.
- Treat this as a local package review. Files will be manually selected for GitHub upload.
- The package passed the local hygiene checks after cleanup: JSON parse, embedded `dataExample` JSON parse, non-ASCII scan, and trailing-whitespace scan.
- Next stage: reduce the canonical five-envelope structure into abbreviated forms through an explicit mapping, without changing the dependency model or reintroducing CO2-level telemetry fields.

---

## 1. Purpose

VRE-PARFE-ContinuousCO2e is the sovereign CO2e passport extension set for Vre Parfe-issued CO2 tonne-lots, where CO2 issuance is derived from issued REC evidence, governed CO2 methodology scope, emission-factor scope, allocation reconciliation, HCS replayability, and HTS lifecycle state.

**Dependency chain (binding):**

```
Telemetry -> REC issuance -> REC allocation -> CO2 methodology -> CO2 passport / HTS unit
```

CO2 does not originate from raw telemetry directly, and this Extension Set must not be built or described as if it does. Raw readings remain available for audit through the REC evidence chain, reached by traversing `parentRecEvidence`. CO2 applies its own governed methodology and GEF/emission-factor scope on top of already-issued REC evidence -- it is a **CO2 Passport from governed REC evidence**, not a parallel telemetry consumer.

Do not model this extension around legacy VVB-led verification either. Continuous verification automation, HCS replay, governed methodology scope, governed emission-factor scope, parent REC evidence, and HTS serialised unit issuance remain the operating model; a VVB is referenced only where required for external assurance, sovereign approval, Article 6/compliance use, or methodology recognition  -  never as the primary source of truth for routine digital issuance.

## 2. Where this sits relative to what already exists

1. **Live on-chain data**  -  HCS topic `0.0.8585272`, sequences 374200-374208, and governance topic `0.0.8479702`, sequences 76-78, pasted directly from Hashscan by the project owner on 2026-07-06. Ground truth for what Mission Control emits after five-envelope adoption and explicit external-methodology governance.
2. **`MISSIONCONTROL_LIVE.js`**  -  `buildExternalMethodologySnapshot`, `buildCo2OffsetLotConfirmationPayload`, `buildCo2RegulatoryPassportPayload`, and the CO2 method-scope readiness gate. Verified to emit the five-envelope model and to source `externalMethodology` from governed method-scope evidence, including the existing `parentRecEvidence` array  -  which correctly models the REC-first dependency and needed no structural change.
3. **`VRE-PARFE-HydroRE`** (merged PR #16)  -  the REC/telemetry evidence layer this set composes with along exactly one seam: `parentRecEvidence`.

## 3. The five-envelope passport model (revised)

| Envelope | Field | Live status | Owner |
|---|---|---|---|
| standardEnvelope | `standardFamily`, `standardVersion`, `extensionSetId`, `extensionSetVersion`, `qualityStandardRole` | Absent | CO2 |
| standardEnvelope | `methodologySystem`, `methodologyAuthority`, `methodologyId`, `methodologyVersion` | Present, scattered across `claim`/`methodScope` | CO2 |
| externalMethodology | `authority`, `methodologyId`, `methodologyName`, `validationRefCodes` | Present; must be explicit in governed `CO2_METHOD_SCOPE_GRANT` and replayed via `methodScopeHcsSequence` | CO2 |
| externalMethodology | `documentVersion`, `documentRef`, `documentHash`, `purpose`, `dependencyStatus` | Absent | CO2 |
| continuousVerification | `methodologyGovernanceStatus`, `gefScopeGovernanceStatus` | Absent | **CO2** (its own governance gates) |
| continuousVerification | `recEvidenceValidityStatus` | Absent | **CO2** (checks parent RECs resolve validly; does not re-check telemetry) |
| continuousVerification | `allocationReconciliationStatus` | Implicit invariant only (verified: 675.2M + 324.8M = 1,000M mgCO2e) | **CO2** |
| continuousVerification | ~~sensor/gateway/signature/physics/fraud/sequence-integrity gates~~ | N/A to this extension set | **REC/telemetry layer (VRE-PARFE-HydroRE)**  -  removed from this schema; reached only by traversal |
| auditAssurance | `controlEvidence.governanceTopicId`, `.methodScopeHcsSequence`, `.mintEventsTopicId`, `.passportHcsSequence`, `.confirmationHcsSequence` | Recoverable from live fields, not named as such | CO2 |
| auditAssurance | ~~`controlEvidence.telemetryTopicId`~~ | **Removed**  -  see Section 4 | Not a CO2-level field |
| auditAssurance | `assuranceModel`, `auditorRole`, `vvbDependency`, `vvbUseWhereRequired` | Absent | CO2 |
| issuedUnit | `quantityMgCO2e`/`quantityKgCO2e`/`tokenId`/`serial`/`htsTxId` | Present | CO2 |
| issuedUnit | `unit`, `lifecycleStatus`, `retirementStatus` | Absent/ad hoc | CO2 |

### 3.1 Verified allocation math (unchanged)

```
allocations[0].consumedMgCO2e = 675,200,000
allocations[1].consumedMgCO2e = 324,800,000
sum                            = 1,000,000,000  =  totalMgCO2e  OK
totalMgCO2e / 1,000,000         = 1000.000000    =  totalKgCO2e OK
```

Now a named gate: `allocationReconciliationStatus` (FormulaTemplates.json Equation 2).

### 3.2 External methodology explicitness (binding)

The external UNFCCC/CDM-style methodology citation is distinct from the sovereign Vre Parfe methodology identity.

- `standardEnvelope.methodologyId` / `claim.methodologyId` identify Vre Parfe's own sovereign methodology, e.g. `VP-CO2E-RE-GRID-001`.
- `externalMethodology.methodologyId` identifies the reference-only external methodology citation, e.g. `AMS-I.D`.

This extension set must not silently invent the external citation. A valid CO2 method scope must declare `externalMethodology.methodologyId` and `externalMethodology.validationRefCodes` in governed HCS evidence. Mission Control must treat a missing external methodology declaration as an incomplete method scope, not as permission to default to `ACM0002`, `AMS-I.D`, or any validation tool.

Verified live correction:

- Governance topic `0.0.8479702`, sequences 76-77: `CO2_METHOD_SCOPE_GRANT` explicitly declares `externalMethodology.methodologyId = "AMS-I.D"` and `validationRefCodes = ["ARTICLE_6_4_PRINCIPLES"]`.
- Governance topic `0.0.8479702`, sequence 78: governance audit event records `CO2_METHOD_SCOPE_GRANTED`.
- Mint-events topic `0.0.8585272`, sequences 374200-374204: `CO2_REGULATORY_PASSPORT` carries `externalMethodology.methodologyId = "AMS-I.D"` and `methodScope.hcsSequence = "77"`.
- Mint-events topic `0.0.8585272`, sequences 374205-374208: `CO2_OFFSET_LOT_CONFIRMATION` carries the same `AMS-I.D` external methodology and minted HTS serial `31371`.

## 4. Correction: `telemetryTopicId` is not a CO2-level dependency

The first draft of this spec listed a missing `telemetryTopicId` as the "highest-leverage single change," on the reasoning that a CO2 passport holder should be able to jump straight to raw telemetry. That framing implied CO2 originates from telemetry directly, which is incorrect and has been reversed.

**Corrected model:**

- CO2's one and only hop down is `parentRecEvidence` (already present, unchanged, verified live: `claimId`, `tokenId`, `serial`, `finalHcsTopicId`, `finalHcsSequence` per parent REC).
- Resolving a `parentRecEvidence` entry means fetching that REC's own passport record. Whether *that* record references its originating signed-telemetry topic (the CO2e-layer analogue of HydroRE topic `0.0.8480236`) is a property of the REC passport format  -  out of scope for `VRE-PARFE-ContinuousCO2e`.
- `ControlEvidence.telemetry_topic_id` has been removed from the proto (field number `3` marked `reserved`, not reused) and from every JSON template. Nothing in this Extension Set caches a telemetry coordinate.
- `recEvidenceValidityStatus` (new gate) is what CO2 actually asserts: every referenced parent REC resolves to a valid, minted, non-revoked token. It does not assert anything about the telemetry underneath that REC.
- `auditReplayStatus` values changed from `REPLAYABLE_FROM_HCS` to `REPLAYABLE_VIA_REC`, naming the one-hop guarantee CO2 actually provides.

If full three-hop replay (CO2 -> REC -> telemetry) needs to be expressed as a single machine-checkable status, that belongs in a future `VRE-PARFE-RECPassport` extension set's own control evidence, composed with this one via `parentRecEvidence` -- not folded into CO2's schema.

## 5. Hand-off instructions

- **Do not edit `MISSIONCONTROL_LIVE.js` directly**  -  that file is live and out of scope for this pass. Propose integration as a diff/patch against the named functions for the owner or their engineer to review and apply.
- New fields are additive: `claim`, `accounting`, `parentRecEvidence`, `gefScope`, `methodScope`, `allocations`, `co2Token`, `co2Passport` all continue to exist unchanged. `standardEnvelope`, the revised `continuousVerification`, `auditAssurance`, and the extended `issuedUnit`/`externalMethodology` fields are new siblings, not a replacement schema.
- `validateSchema(passport)` (`submitCo2RegulatoryPassport` ~line 2784, `submitCo2OffsetLotConfirmation`) is the natural enforcement point once the new envelopes are added.
- `recEvidenceValidityStatus`, `methodologyGovernanceStatus`, `gefScopeGovernanceStatus`, and `allocationReconciliationStatus` should be computed from checks Mission Control already performs (or can perform against `methodScope`/`gefScope`/`parentRecEvidence`) at CO2 build time  -  not asserted as static "PASSED" literals.
- Do not add a telemetry topic reference to the CO2 payload. If deeper replay tooling is wanted later, build it as a traversal helper that follows `parentRecEvidence` into the REC's own passport, rather than as a new CO2-level field.

## 6. Known defects found and not carried forward

Two files in the merged `VRE-PARFE-HydroRE` package (confirmed via `git clone` of the actual repository, not just the PR diff view) are structurally invalid  -  see `HydroRE-fixes/` for corrected replacements:

- `DeploymentPackage/VariableTemplates.json`  -  truncated mid-object; fails `json.load`.
- `DeploymentPackage/protos/vreParfeExtensionSet.proto`  -  truncated inside `message PhysicsGateOptions`; never defines `message LedgerAttestationOptions`.

## 7. Next stage: abbreviated forms

The canonical five-envelope model remains the source of truth for this submission:

```text
standardEnvelope
externalMethodology
continuousVerification
auditAssurance
issuedUnit
```

The next stage is to create abbreviated forms for IWA review or runtime compactness. That pass must be a mapped representation of the canonical model, not a semantic rewrite.

The first abbreviation pass is now defined as a scoped transport/view profile. Root aliases apply globally. Inner aliases apply only inside their owning envelope so that, for example, `issuedUnit.tokenId` can be abbreviated as `unit.tok` without renaming `parentRecEvidence[].tokenId`.

Root aliases:

| Canonical | Abbreviation |
|---|---|
| `standardEnvelope` | `std` |
| `externalMethodology` | `extMethod` |
| `continuousVerification` | `cv` |
| `auditAssurance` | `audit` |
| `issuedUnit` | `unit` |
| `parentRecEvidence` | `parentRec` |
| `allocations` | `alloc` |

Primary scoped aliases:

| Scope | Canonical | Abbreviation |
|---|---|---|
| `standardEnvelope` | `standardFamily`, `standardVersion`, `extensionSetId`, `extensionSetVersion` | `sf`, `sv`, `esId`, `esVer` |
| `standardEnvelope` | `methodologySystem`, `methodologyAuthority`, `methodologyId`, `methodologyVersion`, `qualityStandardRole` | `methSys`, `methAuth`, `methId`, `methVer`, `qsRole` |
| `externalMethodology` | `authority`, `methodologyId`, `methodologyName`, `validationRefCodes`, `dependencyStatus` | `extAuth`, `extId`, `extName`, `valRefs`, `depStatus` |
| `externalMethodology` | `documentVersion`, `documentRef`, `documentHash`, `purpose` | `docVer`, `docRef`, `docHash`, `purp` |
| `continuousVerification` | `sourceOfTruth`, `verificationEngine` | `srcTruth`, `verEng` |
| `continuousVerification` | `recEvidenceValidityStatus`, `methodologyGovernanceStatus`, `gefScopeGovernanceStatus`, `allocationReconciliationStatus`, `auditReplayStatus` | `recValid`, `methodGov`, `gefGov`, `allocRecon`, `replay` |
| `auditAssurance` | `assuranceModel`, `auditorRole`, `vvbDependency`, `vvbUseWhereRequired`, `controlEvidence` | `assrModel`, `audRole`, `vvbDep`, `vvbUse`, `ctrl` |
| `controlEvidence` | `schemaVersion`, `governanceTopicId`, `mintEventsTopicId`, `methodScopeHcsSequence`, `passportHcsSequence`, `confirmationHcsSequence` | `schemaVer`, `govTopic`, `mintTopic`, `methodSeq`, `passportSeq`, `confirmSeq` |
| `issuedUnit` | `assetType`, `claimType`, `quantityMgCO2e`, `quantityKgCO2e`, `unit`, `tokenId`, `serial`, `htsTxId`, `lifecycleStatus`, `retirementStatus` | `asset`, `claim`, `qtyMg`, `qtyKg`, `u`, `tok`, `ser`, `htsTx`, `life`, `retire` |
| `allocations` | `parentRecClaimId`, `consumedMgCO2e` | `recClaim`, `consMg` |

Rules for the abbreviation pass:

- Keep the canonical terms in the spec until IWA accepts abbreviated aliases.
- Do not merge REC/telemetry-layer controls into the CO2 layer.
- Do not add `telemetryTopicId` or any direct telemetry coordinate to this extension set.
- Keep `externalMethodology` distinct from the sovereign Vre Parfe methodology identity.
- Preserve `REPLAYABLE_VIA_REC` as the replay guarantee.
- Keep the canonical spec and proto definitions as the source of truth until IWA accepts abbreviated aliases as schema-facing names.

### 7.1 Record reconstruction tool

The local helper `tools/record_reconstruction_tool.js` supports the abbreviation pass by converting mapped keys in either direction:

```text
tools\record_reconstruction_tool.cmd --to canonical input.json output.canonical.json
tools\record_reconstruction_tool.cmd --to abbreviated input.json output.abbrev.json
tools\record_reconstruction_tool.cmd --map
```

The tool preserves unknown fields and performs no evidence defaulting. It warns on CO2-level telemetry coordinates because those must remain outside this extension set. The sample records in `examples/canonical_passport_sample.json` and `examples/abbreviated_passport_sample.json` round-trip exactly through the tool.

### 7.2 Deployment package implementation

The abbreviation profile is implemented as a registry-layer module, not as a replacement for the canonical envelope messages:

| Artifact | Implementation |
|---|---|
| Machine-readable alias map | `DeploymentPackage/AbbreviationProfile.json` |
| Module | `REGISTRY-RECONSTRUCTION-MODULE` |
| Entity extension template | `Abbreviated Passport Transport Profile Options` |
| Proto types | `AbbreviatedPassportProfileOptions`, `AliasMapping`, `PassportReconstructionRequest`, `PassportReconstructionResponse` |
| Message pair | `Registry Passport Reconstruction Message Pair` |

This makes the abbreviated form available for display, review, and registry quasi-verification while preserving the canonical five-envelope model as the normative source.
