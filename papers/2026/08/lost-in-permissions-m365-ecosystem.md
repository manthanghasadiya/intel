# Lost in Permissions: Exploring the Microsoft 365 App Ecosystem

**Paper:** [arXiv:2608.02336](https://arxiv.org/abs/2608.02336)
**Authors:** Vincenzo Longo, Alberto Verna, Nikhil Jha, Marco Mellia
**Institution:** Politecnico di Torino, Italy
**Published:** August 3, 2026
**Subjects:** Cryptography and Security (cs.CR)
**Size:** 3,543 KB

---

## Problem Statement (Plain English)

Microsoft 365 tenants integrate thousands of third-party applications via OAuth, giving those apps access to emails, files, calendars, chats, and user directories. Despite the security implications of these permission grants, nobody has systematically studied the M365 app ecosystem: what apps exist, what permissions they request, whether those permissions match what the apps claim to do, and whether users can even find out what an app actually needs before installing it.

## Methodology (Technical)

**1. Data Collection:**
- Combined public M365 marketplace APIs with automated tenant-side deployment
- Crawled over 8,000 third-party M365 applications
- Extracted descriptions, declared categories, and OAuth permission scopes

**2. Transparency Assessment:**
- Found only 1,069 of 8,000+ apps exposed both descriptions and permission sets
- Identified significant inconsistencies between different official distribution channels (Azure portal, M365 admin center, marketplace)

**3. Anomaly Detection Pipeline:**
- **Neural Topic Modelling:** Clustered apps by declared functionality (calendar apps, CRM integrations, document tools, etc.)
- **Unsupervised Anomaly Detection per Topic:** Within each functional cluster, identified apps whose permission profiles deviated significantly from peer apps
- **LLM-Assisted Analysis:** Used LLMs to analyze the most anomalous cases — why does a survey tool need directory-wide read/write?
- **Blind Manual Inspection:** Verified LLM findings against human judgment to confirm correlation between anomalous permissions and actual risk

**4. Permission Risk Scoring:**
- Developed a pipeline that identifies which specific permissions most contribute to making an app anomalous within its functional cluster
- Output: actionable insights for tenant administrators (which apps are over-privileged, which permissions are the problem)

## Key Results

- **8,000+ apps crawled**, but only **1,069 (13.4%) exposed both descriptions and permissions** — massive transparency gap
- **Many apps request overly broad tenant-wide scopes** (directory-wide read/write access) violating least-privilege principles
- **Confirmed correlation between anomalous permission profiles and actual permission risk** — the anomaly detection approach works
- **Systemic opacity**: permission disclosure is inconsistent across distribution channels, app descriptions, and the marketplace
- **Structural immaturity:** the M365 ecosystem lacks enforcement mechanisms for permission-permission-to-function alignment, meaning developers can request anything regardless of what their app does

## What's Novel

1. **First systematic measurement of M365 third-party app ecosystem** — no prior academic work has crawled the M365 marketplace at this scale with a security/privacy lens.
2. **Topic-aware anomaly detection for permission profiling** — rather than evaluating apps in isolation (too many false positives), the approach compares apps *within their functional cluster*. A calendar app needing mailbox.read is normal; a note-taking app needing directory.readwrite.all is anomalous.
3. **LLM-assisted risk analysis** — combining unsupervised anomaly detection with LLM reasoning over anomaly explanations creates human-readable risk assessments for tenant admins.
4. **Privacy-preserving methodology** — the crawl uses public marketplace APIs and automated tenant-side deployment without needing access to actual tenant data.

## My Connection (to Manny's Work)

For red team operations targeting M365 tenants: this paper reveals that OAuth app poisoning and over-privileged app exploitation are systematic, not isolated, risks. The anomaly detection methodology could be inverted for offensive use — find apps that have anomalous permission profiles and already exist in a target tenant, then craft attacks that exploit their over-broad access. The 86.6% transparency gap (apps not exposing permissions publicly) means defenders can't audit what's installed — but attackers who can enumerate installed apps via the Graph API can profile their capabilities. The tenant-wide directory read/write access pattern in many M365 apps is a single-compromise-to-domain-admin escalation path if the app's service principal is hijacked.

## What I Learned (Plain English)

Microsoft 365's app ecosystem is a mess from a security perspective. Most apps don't even publicly disclose what permissions they need, many ask for way more access than their function requires (a survey tool with full directory write access), and there's no enforcement that permissions should match declared functionality. The researchers built a system that groups apps by what they claim to do and then flags ones that have suspiciously more access than their peers — and it works. The practical takeaway for any M365 tenant: audit your installed third-party apps, especially their OAuth scopes, because the platform doesn't do it for you.
