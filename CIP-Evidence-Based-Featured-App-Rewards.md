# CIP: Evidence-Based Featured App Rewards

```
CIP: <unassigned>
Title: Evidence-Based Featured App Rewards
Author:
 Joris Delanoue <joris@fairmint.co>
Discussions-To: https://github.com/jorisparis/canton-featured-apps/issues
Status: Draft
Type: Tokenomics
Created: 2026-09-23
License: CC0-1.0
```

**Disclosure.** The author also serves as General Director of the Canton Foundation. Fairmint is an SEC-registered transfer agent with a FINRA broker-dealer application under review. This proposal is submitted as a working document from an ecosystem participant, not as an official position of the Canton Foundation.

## Abstract

Canton is moving toward programmatic measurement of application activity and toward Featured App status tied to conditions that must remain satisfied over time. This proposal extends that direction to the evidence available about Featured Apps.

It introduces a programmable framework for adjusting Featured App reward weight based on verifiable claims relevant to an application's activities. Governance defines the rules and the evidence types recognized; applications satisfy those rules through network-native state or verifiable external attestations; the network computes the resulting evidence state.

Canton should remain open to applications, while the economic weight assigned to Featured Apps should reflect verifiable evidence of accountability. The framework does not restrict network access, application deployment, transaction execution, or validator interoperability. It applies to Featured App economic incentives.

The precise integration point with the existing traffic-based reward mechanism is a design question for the working group, not a settled part of this proposal. Concrete reward weights are deliberately not specified and should follow a non-economic pilot before any production activation.

**Terminology note.** This proposal uses "evidence" rather than "trust" to describe what the framework consumes and produces. Registration, SOC 2, audits, and similar inputs are evidence of specific facts or controls. They do not constitute a certification that an application is trustworthy overall, and this framework is not intended as an endorsement or safety certification.

## Specification

### 1. Scope

This CIP applies to Featured App reward weighting. It does not restrict network access, application deployment, transaction execution, or validator interoperability. Non-featured applications remain full network participants.

### 2. Application Activities

Applicability begins with what the Featured App actually does. Rather than assigning an application to a single category, the framework identifies the activities performed by that application.

Illustrative activities include analytics, developer infrastructure, identity infrastructure, asset issuance, securities transfer, transfer agency, brokerage, custody, settlement, wallets, and other regulated financial activity.

An application may perform more than one activity. Each activity maps to the evidence requirements relevant to that activity:

```text
ANALYTICS
    Regulatory Registration: NOT_APPLICABLE
    AML Program: NOT_APPLICABLE
    Security Baseline: APPLICABLE

BROKERAGE (in a jurisdiction requiring it)
    Regulatory Registration: REQUIRED
    AML Program: REQUIRED
    Security Baseline: REQUIRED
```

Regulatory requirements depend on the actual activity and the jurisdiction in which it is performed. The rule set should express applicability conditionally rather than treating a label like "brokerage" as universally implying a fixed compliance profile.

Where an application performs multiple activities, the applicable requirements are combined. The most restrictive applicable requirement prevails. This avoids allowing an application to reduce its evidence requirements simply by declaring a less demanding activity.

The initial activity taxonomy and applicability rules are to be established through governance and versioned over time.

### 3. Evidence Claims

The framework evaluates discrete claims rather than assigning broad subjective judgments. A claim represents a fact about an application, its operator, or its operating environment.

Examples: legal entity verified, regulatory registration active, security audit current, incident response process attested, Featured App staking requirement satisfied.

An illustrative claim model:

```text
EvidenceClaim {
    subject
    claimType
    value
    issuer
    issuedAt
    validUntil
    evidenceReference
    status
}
```

Possible states include `VERIFIED`, `NOT_VERIFIED`, `NOT_APPLICABLE`, `EXPIRED`, `REVOKED`, and `UNDER_REVIEW`. The exact data model is a design question for implementation.

### 4. Sources of Evidence

Evidence inputs come from three places.

Some facts are network-native and can be established directly from Canton, including active Featured App status, staking, production history, and qualifying activity.

Others exist outside Canton and must be represented through verifiable attestations from recognized issuers, including regulatory registrations, SOC 2 examinations, security audits, and insurance coverage. An attestation should identify the subject, the claim, the issuer, the date of issuance, an expiration date where relevant, and its current validity. The underlying confidential evidence does not need to be published onchain; the framework needs a verifiable representation of the relevant fact.

A small remainder require governance resolution: disputed activity classification, attestor recognition, appeals, emergency action, or claim types not yet covered by the rule set. Governance-resolved cases are exceptions, not the default scoring path.

### 5. Eligibility Gates

Not every claim is a weighted score. Some requirements are fundamental to a particular activity.

The framework distinguishes between eligibility gates, which must be satisfied to reach a particular evidence tier, and weighted signals, which affect the economic weight once eligibility is satisfied.

If an application's activity requires regulatory registration in the jurisdiction where the activity is performed, valid regulatory status is a hard requirement for the highest evidence tier. A high score for security, transparency, or network activity does not compensate for the absence of a legally required registration:

```text
if required_regulatory_status != VERIFIED:
    highest_evidence_tier_eligible = false
```

### 6. Evidence Dimensions

Claims are grouped into dimensions that remain useful for human interpretation:

| Dimension                 | Example Inputs                                                                          |
| ------------------------- | --------------------------------------------------------------------------------------- |
| **Legal Accountability**  | Legal entity verified, jurisdiction identified, accountable operator identified         |
| **Compliance**            | Regulatory status, KYC/KYB controls, sanctions controls, AML program where applicable   |
| **Security**              | Security audits, SOC 2, ISO/IEC 27001, incident response, unresolved material incidents |
| **Transparency**          | Public documentation, governance information, required disclosures                      |
| **Operational Readiness** | Production status, operational history, SLAs, business continuity                       |
| **Ecosystem Reputation**  | Institutional references, partner attestations, relevant ecosystem attestations         |
| **Network Activity**      | Production history, qualifying Canton activity, other protocol-visible contribution     |

These dimensions organize the inputs. They do not reintroduce discretionary scoring.

### 7. Evidence Computation

The evidence function takes applicable valid claims as inputs and produces a deterministic evidence state:

```text
EvidenceState = f(
    AppActivities,
    ApplicableClaims,
    EligibilityGates,
    WeightedSignals,
    RuleSetVersion
)
```

The rule set defines applicable claims by activity, mandatory claims, accepted states, weights, thresholds, expiration treatment, and resulting economic weights. The rule set is public and versioned. Given the same application activities, claims, and rule-set version, independent implementations produce the same result.

### 8. Evidence Tiers

Human-readable tiers sit above the underlying programmable state:

| Tier                            | Description                                    |
| ------------------------------- | ---------------------------------------------- |
| **Experimental**                | Minimal verified evidence                      |
| **Verified Builder**            | Identifiable operator with baseline controls   |
| **Trusted Infrastructure**      | Mature production infrastructure               |
| **Institutional Evidence**      | Highest applicable requirements satisfied      |

Tiers are labels for the underlying state, not manually assigned badges. They describe the evidence available for the purpose of reward weighting; they are not safety certifications or endorsements.

Concrete reward weights per tier are deliberately not specified in this draft. Whether weighting is discrete or continuous, the range of weights, and whether higher weights would dilute other applications' rewards are questions the pilot is intended to answer. Any economic activation should follow published pilot results.

### 9. State Changes

Evidence state changes when its inputs change. If a security-audit attestation expires without renewal, the claim moves to `EXPIRED`, the evidence state is recalculated, and the evidence weight may change accordingly. The same applies to revoked regulatory status, expired certifications, or changed staking status.

Where the result follows directly from the rule set, promotion or demotion does not require a new discretionary governance decision. Governance intervention remains available for disputed facts, appeals, or emergencies.

### 10. Integration with Featured App Rewards

The output of the evidence function is intended to influence the reward weight applied to Featured App activity:

```text
Qualifying App Activity
        |
        v
Traffic-Based Measurement
        |
        v
Featured App Evidence Weight
        |
        v
Reward Calculation
```

Traffic calculation determines the amount of qualifying activity. The evidence framework determines the economic weight applied to it. The technical mechanism connecting the two, including whether it flows through an existing reward parameter or a new one, is a design question for working-group review and is not settled by this proposal.

### 11. Governance Responsibilities

Governance primarily maintains the framework rather than evaluating individual applications. Its responsibilities include:

* defining application activities,
* defining applicability rules,
* approving claim schemas,
* recognizing accepted attestors,
* establishing eligibility gates and thresholds,
* versioning the rule set,
* handling disputes, appeals, and emergency intervention.

Changes to the rule set follow the appropriate Canton governance process. Where possible, individual application outcomes follow automatically from the approved rules and verified state.

## Motivation

Canton is increasingly becoming infrastructure for institutional financial activity. As that activity grows, the consequences of economically endorsing an application grow with it. A security failure, sanctions violation, regulatory problem, or material operational failure at a Featured App can create reputational and economic damage far beyond the application itself.

Canton should not solve this by creating a certification committee that manually decides which applications are trustworthy. That would introduce subjectivity, governance overhead, and barriers to new entrants.

Recent Canton tokenomics proposals point toward a different model: define objective rules and let observable state determine economic outcomes. The same principle should apply to evidence of accountability.

The underlying hypothesis worth testing is that higher evidence of accountability creates network value that raw traffic measurement does not capture. That hypothesis needs to be tested against gaming, costs for smaller builders, and possible reward concentration. The pilot proposed in the Path Forward section is intended to produce evidence on those questions before any economic activation.

## Rationale

### Relationship to Existing Featured App Tokenomics

This proposal complements the existing Featured App reward architecture rather than replacing it.

Traffic-based App Rewards provide a mechanism for measuring qualifying application activity from network activity. Featured App staking provides an example of Featured App eligibility being tied to an objective condition that must remain satisfied over time.

This proposal addresses a separate question: once qualifying Featured App activity has been measured, should all Featured Apps receive the same economic weight regardless of the evidence available about what they do and how they operate? The proposed answer is no.

### Design Principles

Six principles inform the design:

**1. Open Participation.** The framework does not determine who may build or transact on Canton. An application may operate on Canton without receiving the highest level of Featured App economic incentives. Evidence weighting is an incentive mechanism, not an access-control mechanism.

**2. Programmability by Default.** Evidence is represented and evaluated programmatically wherever the underlying facts can be expressed through network state or verifiable attestations. The same valid inputs should produce the same result.

**3. Activity-Based Applicability.** Applications are evaluated only against requirements relevant to what they actually do. An analytics provider is not penalized for lacking AML controls. An application conducting regulated financial activity cannot compensate for missing regulatory status through strong scores elsewhere.

**4. Existing Assurance Should Be Reused.** The framework recognizes existing sources of assurance rather than recreating them. Regulatory registration, SOC 2 examinations, ISO/IEC 27001 certification, and external security audits serve as inputs where relevant.

**5. Continuous State.** Evidence is not a permanent badge. Inputs may expire, be revoked, or change. When they do, the resulting evidence state is recalculated.

**6. Minimal Governance.** Governance defines and maintains the rules. It does not ordinarily score applications manually when the relevant facts can be represented programmatically.

### Why Evidence, Not Trust or Assurance

Registration, SOC 2, audits, and similar inputs are evidence of specific facts or controls. They do not prove trustworthiness in the general sense. Framing the output as evidence rather than trust or assurance keeps the framework honest about what it measures, avoids the certification connotations of "assurance," and reduces the risk that a tier label is read as an endorsement.

### Why Eligibility Gates

Purely weighted scoring allows an application to compensate for missing capabilities with strengths elsewhere. That is inappropriate for capabilities that are legally required for a given activity. Eligibility gates make certain evidence types non-substitutable for their applicable tier. This also makes the framework easier to reason about and harder to game.

### Path Forward

The proposal is deliberately incomplete on parameters that require calibration and testing. Detailed choices about the initial activity taxonomy, claim taxonomy, weight calibration, attestor recognition process, and privacy handling remain for the working group.

Suggested steps:

1. Open the proposal for technical and tokenomics review via the CIP process.
2. Form a small working group spanning tokenomics, protocol engineering, Featured App operators, security, and regulated financial infrastructure.
3. Define a minimal v1 activity taxonomy, claim taxonomy, and applicability model.
4. Determine the technical integration point with the existing reward mechanism.
5. Build a reference implementation of the evidence function.
6. Run a non-economic pilot across a small set of Featured Apps, evaluating the evidence hypothesis, gaming resistance, costs for smaller builders, and reward concentration effects.
7. Publish pilot results before proposing production reward activation.

## Backwards Compatibility

This proposal is additive. It does not modify how qualifying application activity is measured, how Featured App status is granted, or how Featured App staking operates. Its output is intended to be combined with existing reward calculations rather than replace them.

Applications that would qualify for the lowest evidence tier under this framework are equivalent, for reward-calculation purposes, to their treatment prior to the framework's activation, subject to the specific integration mechanism the working group adopts.

No changes to existing Daml models, Canton protocol behavior, or Featured App marker semantics are required by this proposal at the level of specification described here. Any protocol-level changes required by a specific integration mechanism are out of scope for this draft and would be addressed in the working group's technical design.

## Reference Implementation

A reference implementation of the evidence function is proposed as part of the Path Forward, not delivered with this draft. Tokenomics CIPs do not require a reference implementation at draft stage.

## Copyright

This CIP is licensed under CC0-1.0 (Creative Commons CC0 1.0 Universal). To the extent possible under law, the author has waived all copyright and related rights to this work.

## Changelog

* 2026-09-23: Initial draft.

---

## Annex A: Isolation Principles for Implementation

The framework should be designed so that unrelated facts do not produce unintended consequences. Three principles apply.

**Applicability isolation.** A requirement affects an application only when relevant to one of that application's activities.

**Claim isolation.** The effect of a claim becoming invalid, expired, or revoked is defined by the rule set, not by producing arbitrary consequences elsewhere in the evidence state.

**Application isolation.** Changes to one Featured App's evidence state do not affect another Featured App's evidence state, except where explicitly required by the reward mechanism itself.

These principles make evidence weighting easier to implement, audit, and predict.

## Annex B: Illustrative Example

Consider a Featured App providing settlement infrastructure. Its declared activities are:

```text
SETTLEMENT
DEVELOPER_INFRASTRUCTURE
```

Those activities make the following claims applicable:

```text
LegalEntityVerified           VERIFIED
SecurityAudit                 VERIFIED
SOC2                          VERIFIED
IncidentResponse              VERIFIED
ProductionDeployment          VERIFIED
OperationalHistory            VERIFIED
FeaturedAppRequirement        VERIFIED
```

With the applicable requirements satisfied and remaining signals meeting the required thresholds, the application reaches an evidence tier appropriate to its activities. The corresponding evidence weight is applied to its Featured App reward calculation according to the rule set.

If the SOC 2 attestation is subsequently not renewed:

```text
SOC2 = EXPIRED
```

the evidence state is recalculated. Depending on the rule set, the change may lower the evidence tier or its weight. The outcome follows from the state change rather than requiring governance to rescore the application manually. That is the property the framework is designed to produce.
