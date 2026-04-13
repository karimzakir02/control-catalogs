# Control Catalog Style Guide

This guide defines the writing conventions for entries in CCC service-specific control catalogs. These controls build on the [core catalog](../core-catalog/) and address risks unique to a particular service type.

---

## General Conventions

### Voice and Tone

- Write in a professional, technical tone suitable for a security-engineering audience.
- Use present tense throughout.
- Be precise and specific — avoid vague language like "properly", "appropriately", or "as needed".
- Avoid marketing language or subjective qualifiers ("best-in-class", "robust", "comprehensive").

### Title Case

All titles use Title Case. Capitalize every word except:

- Articles: a, an, the
- Short prepositions (four letters or fewer): in, of, for, to, at, by, on, via, with
- Conjunctions: and, or, but, nor

Always capitalize the first and last word regardless of part of speech. Examples: "Prevent Requests to Buckets or Objects with Untrusted KMS Keys", "Alert on Key-version Changes".

### Formatting

- Use the YAML block literal (`|`) for any multi-line text field (`objective`, `text`).
- End all sentences with a period.
- Do not end titles with a period or any trailing punctuation.
- Use one blank line between paragraphs within a field, not between sentences.
- Keep entries to the minimum words needed to convey the requirement clearly.

### RFC 2119 Keywords

Assessment requirement `text` fields use [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119) keywords to express obligation levels:

| Keyword | Meaning | When to Use |
|---|---|---|
| **MUST** | Absolute requirement | The condition is mandatory for compliance |
| **MUST NOT** | Absolute prohibition | The condition must never occur |
| **SHOULD** | Recommended | The condition is expected unless a justified reason exists |
| **MAY** | Optional | The condition is permitted but not required |

Always write these keywords in UPPERCASE. Use **MUST** by default — only reach for SHOULD or MAY when the requirement genuinely permits exceptions.

RFC 2119 keywords apply only to assessment requirement `text` fields. Do not use them in titles, objectives, or recommendations.

### Logical Connectors

- Use uppercase **AND** when multiple conditions must all be met in a single requirement.
- Use lowercase "or" for alternatives in prose.
- Use semicolons to separate related but distinct clauses within a single requirement.

---

## Control Titles

- Use **Title Case** (see [Title Case](#title-case) above).
- Begin with an **imperative verb** describing the protective action.
- Keep to 3-12 words.

Common opening verbs and when to use them:

| Verb | Use When |
|---|---|
| Prevent | Blocking an undesired action or state |
| Ensure | Guaranteeing a desired condition exists |
| Enforce | Making a policy or constraint mandatory |
| Restrict | Limiting scope of access or operations |
| Validate | Checking input, configuration, or state |
| Alert | Generating a notification on a condition |
| Limit | Capping a quantity or rate |
| Monitor | Observing and reporting on activity |
| Automate | Removing manual steps from a process |
| Scrub / Sanitize | Removing sensitive data from output |

**Good:** "Prevent Requests to Buckets or Objects with Untrusted KMS Keys"
**Good:** "Enforce Automatic Rotation"
**Good:** "Alert on Key-version Changes"
**Bad:** "Untrusted KMS Keys are Blocked" (passive — use imperative)
**Bad:** "Key Rotation" (too vague, not actionable)

---

## Objective

- Write 1-3 complete sentences.
- Open with an imperative verb: "Ensure that...", "Prevent...", "Restrict...", "Generate...", "Monitor...".
- Explain the protection goal, the mechanism, and the benefit.
- Reference the specific service domain where relevant (e.g., "object storage buckets", "load balancer listeners", "KMS keys").

**Example:**

```yaml
objective: |
  Prevent any requests to object storage buckets or objects using
  untrusted KMS keys to protect against unauthorized data encryption
  or sensitive data decryption.
```

### Objective vs. Core

If a control's objective overlaps significantly with a core control, import the core control instead of duplicating it. Service-specific controls should address risks that only apply to this service type.

---

## Group

Every control must have a `group` field. The group describes the security or operational domain the control belongs to — not which service it is part of.

Assign the group by asking: **"What breaks if this control is missing?"**

- Unauthorized access → `Access`
- Data exposed in transit or at rest → `Encryption`
- Loss of visibility into what happened → `Observability`
- Data lost, corrupted, or leaked → `CCC.Core.Data`
- Resources misconfigured or exhausted → `CCC.Core.Resource`
- Network traffic misrouted or exposed → `CCC.Core.Networking`

When a control spans two domains, pick the one that best describes its _primary purpose_. For example, a control about encrypting data at rest using CMEK belongs in `Encryption`, not `CCC.Core.Data`.

See the [group assignment guide](../CLAUDE.md#group-assignment-guide) for the full list of available groups with descriptions.

---

## Applicability

Each assessment requirement must include an `applicability` list specifying which Traffic Light Protocol (TLP) levels the requirement applies to:

| Level | Meaning |
|---|---|
| `tlp-clear` | Public, unrestricted information |
| `tlp-green` | Community-wide sharing |
| `tlp-amber` | Limited sharing within the organization |
| `tlp-red` | Restricted to named recipients only |

List levels from least restrictive to most restrictive. Most requirements apply to `tlp-amber` and `tlp-red`. Requirements that apply universally should list all four levels.

---

## Threat Mappings

Each control should include a `threat-mappings` section linking it to the threats it mitigates:

- Group entries under the correct parent `reference-id` (e.g., `CCC.ObjStor.Threats`, `CCC.Core.Threats`).
- If mapped threats come from multiple service prefixes, split them into separate groups.

---

## Assessment Requirement `text`

- Write as a single testable statement.
- Use the conditional pattern: **"When [trigger condition], the service MUST [requirement]."**
- The subject is typically "the service", but may be "the system", "an alert", or "the load balancer" when clarity demands it.
- Include specific, measurable thresholds where applicable (e.g., "within five minutes", "80 percent of capacity", "2000 requests").
- Each AR should test exactly one condition — split compound requirements into separate ARs.

**Example:**

```yaml
text: |
  When a request is made to read a bucket, the service MUST prevent
  any request using KMS keys not listed as trusted by the organization.
```

**Example with a measurable threshold:**

```yaml
text: |
  When a single client sends more than 2000 requests within a
  one-minute window, the load balancer MUST throttle or reject
  excess requests.
```

---

## Recommendation

- Write as brief, actionable implementation guidance.
- Use imperative verbs: "Track", "Configure", "Review", "Implement", "Define".
- Include specific examples, tool references, or timeframes where helpful.
- Leave as an empty string (`""`) when the AR text is self-explanatory.
- **Exception to the provider-name rule:** Recommendations are the one place where cloud-provider-specific product names are acceptable, because they serve as concrete implementation hints. Parenthetical lists covering multiple providers are preferred so no single provider is favored.

**Example:**

```yaml
recommendation: |
  Use native event services (e.g., CloudWatch Events, Azure Monitor,
  Cloud Audit Logs) to detect key-version lifecycle changes.
```

---

## Imports

When referencing core controls in the `imports` section:

- The `remarks` field should match or closely paraphrase the core control's title.
- Keep remarks concise — they are a label, not a description.

**Example:**

```yaml
imports:
  - reference-id: CCC.Core.Controls
    entries:
      - reference-id: CCC.Core.CN01
        remarks: Prevent Unencrypted Requests
```

---

## Service-Specific Language

Unlike the core catalog, service-specific controls should name the service domain directly. Use the canonical terminology for the service type:

| Service | Use terms like |
|---|---|
| Object Storage | buckets, objects, retention policies |
| Key Management | keys, key versions, key rings, rotation |
| Load Balancer | listeners, backends, health checks, targets |
| Virtual Machines | instances, images, boot disks, snapshots |
| IAM | principals, roles, policies, permissions |
| GenAI | models, prompts, inference, fine-tuning |

Avoid cloud-provider-specific product names (S3, GCS, ALB, CloudFront). Use generic service-type terms instead. Note that "KMS" is acceptable as a generic abbreviation for key management service.
