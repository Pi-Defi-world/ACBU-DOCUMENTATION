# Security Audit Report Template

**Project:** ACBU (African Currency Basket Unit)  
**Audit Target:** [Repository / Component / Release]  
**Audit Period:** [Start Date] - [End Date]  
**Report Date:** [Date]  
**Auditor(s):** [Name / Team / Organization]  
**Status:** Draft / Final

---

## 1. Executive Summary

Provide a concise overview of the audit, including the systems reviewed, the overall security posture, critical risks, and recommended next steps.

| Summary Item | Details |
| :--- | :--- |
| Audit scope | [Smart contracts, backend API, frontend, infrastructure, operations] |
| Overall risk rating | Critical / High / Medium / Low |
| Critical findings | [Number] |
| High findings | [Number] |
| Medium findings | [Number] |
| Low findings | [Number] |
| Informational findings | [Number] |

### Key Conclusions

- [Conclusion 1]
- [Conclusion 2]
- [Conclusion 3]

---

## 2. Scope

Describe what was included in the audit and identify any exclusions.

### 2.1 In-Scope Assets

| Area | Asset | Version / Commit | Notes |
| :--- | :--- | :--- | :--- |
| Smart contracts | [Contract name] | [Commit hash / tag] | [Notes] |
| Backend | [Service name] | [Commit hash / tag] | [Notes] |
| Frontend | [Application name] | [Commit hash / tag] | [Notes] |
| Infrastructure | [Environment] | [Configuration version] | [Notes] |
| Operations | [Process / Runbook] | [Version] | [Notes] |

### 2.2 Out-of-Scope Assets

- [Asset or system excluded from review]
- [Reason for exclusion]

### 2.3 Assumptions and Constraints

- [Assumption or constraint]
- [Time, access, dependency, or environment limitation]

---

## 3. Methodology

Describe how the audit was performed and which standards, tools, and techniques were used.

### 3.1 Review Activities

- Architecture and threat model review
- Manual source code review
- Dependency and configuration review
- Authentication and authorization review
- Input validation and business logic review
- Smart contract access control and economic risk review
- Logging, monitoring, and incident response review

### 3.2 Testing Approach

| Test Area | Method | Evidence |
| :--- | :--- | :--- |
| Static analysis | [Tool / Manual review] | [Output / Link] |
| Dependency scanning | [Tool] | [Output / Link] |
| Unit and integration tests | [Command / Suite] | [Output / Link] |
| Manual verification | [Procedure] | [Notes] |
| Smart contract tests | [Framework / Command] | [Output / Link] |

### 3.3 Risk Rating Criteria

| Severity | Description |
| :--- | :--- |
| Critical | Direct loss of funds, complete system compromise, or irreversible protocol failure is likely or easily exploitable. |
| High | Significant financial, operational, compliance, or user-impacting risk with realistic exploitation conditions. |
| Medium | Material weakness requiring specific conditions, limited access, or chained exploitation. |
| Low | Limited security impact, defense-in-depth issue, or weakness with low exploitability. |
| Informational | Observation, hardening recommendation, or documentation improvement with no direct security impact. |

---

## 4. Findings Summary

| ID | Title | Severity | Status | Owner |
| :--- | :--- | :--- | :--- | :--- |
| ACBU-SEC-001 | [Finding title] | Critical / High / Medium / Low / Informational | Open / Fixed / Accepted / Mitigated | [Owner] |
| ACBU-SEC-002 | [Finding title] | Critical / High / Medium / Low / Informational | Open / Fixed / Accepted / Mitigated | [Owner] |

### Severity Distribution

| Severity | Count |
| :--- | :--- |
| Critical | [Number] |
| High | [Number] |
| Medium | [Number] |
| Low | [Number] |
| Informational | [Number] |

---

## 5. Detailed Findings

Use one subsection per finding. Keep evidence specific and include file paths, line numbers, transaction hashes, logs, screenshots, or reproduction steps where applicable.

### ACBU-SEC-001: [Finding Title]

**Severity:** Critical / High / Medium / Low / Informational  
**Status:** Open / Fixed / Accepted / Mitigated  
**Affected Area:** [Smart Contracts / Backend / Frontend / Infrastructure / Operations]  
**Affected Asset:** [File, contract, endpoint, service, or process]  
**Owner:** [Responsible person or team]

#### Description

[Describe the issue clearly and objectively.]

#### Impact

[Explain the realistic business, technical, financial, user, or compliance impact.]

#### Evidence

```text
[Relevant code path, command output, request, response, transaction, or log excerpt]
```

#### Reproduction Steps

1. [Step 1]
2. [Step 2]
3. [Step 3]

#### Recommendation

[Describe the recommended fix or mitigation.]

#### Remediation Notes

[Document the implemented fix, pull request, commit hash, or accepted risk decision.]

#### Retest Result

**Retested By:** [Name]  
**Retest Date:** [Date]  
**Result:** Passed / Failed / Partially Fixed / Not Retested

---

## 6. Remediation Plan

| Finding ID | Action Required | Owner | Target Date | Status |
| :--- | :--- | :--- | :--- | :--- |
| ACBU-SEC-001 | [Remediation task] | [Owner] | [Date] | Open / In Progress / Complete |
| ACBU-SEC-002 | [Remediation task] | [Owner] | [Date] | Open / In Progress / Complete |

---

## 7. Retest Summary

| Finding ID | Original Severity | Retest Status | Evidence | Notes |
| :--- | :--- | :--- | :--- | :--- |
| ACBU-SEC-001 | [Severity] | Passed / Failed / Partially Fixed | [Link / reference] | [Notes] |
| ACBU-SEC-002 | [Severity] | Passed / Failed / Partially Fixed | [Link / reference] | [Notes] |

---

## 8. Residual Risk

Document any accepted risks, deferred fixes, compensating controls, and follow-up requirements.

| Risk | Decision | Approver | Review Date |
| :--- | :--- | :--- | :--- |
| [Residual risk] | Accepted / Deferred / Mitigated | [Approver] | [Date] |

---

## 9. Appendices

### 9.1 Tools and Versions

| Tool | Version | Purpose |
| :--- | :--- | :--- |
| [Tool name] | [Version] | [Purpose] |

### 9.2 References

- [Architecture document]
- [Security guide]
- [Threat model]
- [Test report]
- [Relevant pull request or commit]

### 9.3 Evidence Inventory

| Evidence ID | Description | Location |
| :--- | :--- | :--- |
| EVID-001 | [Evidence description] | [File path / link] |

---

## 10. Approval

| Role | Name | Signature / Approval | Date |
| :--- | :--- | :--- | :--- |
| Auditor | [Name] | [Approval] | [Date] |
| Engineering Owner | [Name] | [Approval] | [Date] |
| Security Owner | [Name] | [Approval] | [Date] |
