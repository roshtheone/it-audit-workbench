# it-audit-workbench

A reproducible IAM control-testing workbench that turns raw identity evidence into traceable, reviewer-ready audit findings.



## Why I built this

Identity and access management is one of the places where cybersecurity, governance, and audit meet. Companies need to know more than whether a policy exists — they need evidence that terminated users are removed, privileged accounts are protected, access is periodically reviewed, conflicting permissions are detected, and every conclusion can be traced back to source evidence.

I built this project to model that workflow end to end.

The workbench simulates an IAM audit for a fictional financial-services organization and automates the repeatable, data-driven portion of IT General Controls testing. It generates a controlled identity environment, evaluates access risks, records evidence, derives finding severity from explainable rules, and produces both management-facing and reviewer-facing outputs.

The goal is not to replace auditor judgment. The goal is to demonstrate how security and audit teams can automate high-volume control testing while preserving evidence quality, repeatability, and accountability.

# What the system does

The pipeline creates a synthetic organization with 252 identities, privileged accounts, entitlement assignments, HR termination data, MFA configuration, and access-review evidence. It then executes eight IAM / ITGC control tests.

Control

Security / business risk

AUD-001 — Terminated users with active accounts

Former employees may retain unauthorized access after separation.

AUD-002 — Dormant accounts

Unused identities expand attack surface and can become takeover targets.

AUD-003 — Privileged accounts without MFA

A compromised password can become a privileged compromise.

AUD-004 — Privileged access outside IT

Excessive standing privilege violates least-privilege principles.

AUD-005 — Shared / generic accounts

Shared credentials weaken individual accountability and non-repudiation.

AUD-006 — Segregation-of-duties conflicts

A single user may be able to initiate and approve sensitive transactions.

AUD-007 — Stale privileged access reviews

Inappropriate administrative access can remain undetected.

AUD-008 — Missing access-review evidence

A control may exist operationally but still fail audit because it cannot be evidenced.

Each failed control becomes a structured finding using:

Criteria → Condition → Cause → Risk → Recommendation → Evidence

Why this matters to a cybersecurity team

This project is audit-focused, but the underlying problems are security engineering problems too.

Identity lifecycle risk — validates whether terminated users are actually de-provisioned.

Privileged access management — identifies administrative accounts without adequate safeguards.

Strong authentication — tests MFA coverage for high-impact identities.

Least privilege — flags administrative roles that do not align with business function.

Access governance — evaluates review freshness and retained approval evidence.

Segregation of duties — detects toxic entitlement combinations that can enable fraud or abuse.

Attack-surface reduction — surfaces dormant and shared accounts that should be disabled, removed, or controlled.

Evidence integrity — hashes evidence so a reviewer can detect whether the records behind a finding have changed.

Repeatability — deterministic data and rule-based logic produce consistent results across runs.

In an enterprise environment, the same pattern could consume data from platforms such as Microsoft Entra ID, Okta, Workday, AWS IAM, Active Directory, PAM tools, and a GRC platform instead of synthetic JSON files.

Architecture

                 ┌─────────────────────┐
               ##   │     HR Feed         │
                 │ Terminations / JML  │
                 └──────────┬──────────┘
                            │
┌────────────────────┐      │      ┌────────────────────┐
│ Identity Directory │──────┼──────│ SaaS Entitlements │
│ users / MFA / roles│      │      │ roles / approvals │
└──────────┬─────────┘      │      └─────────┬──────────┘
           └────────────────┼────────────────┘
                            ▼
                 ┌─────────────────────┐
                 │ Normalized Evidence │
                 └──────────┬──────────┘
                            ▼
                 ┌─────────────────────┐
                 │ Control Test Engine │
                 │     8 controls      │
                 └──────────┬──────────┘
                            ▼
             ┌─────────────────────────────┐
             │ Exception + Severity Logic  │
             │ rule-based / explainable    │
             └──────────────┬──────────────┘
                            ▼
             ┌─────────────────────────────┐
             │ Finding + Evidence Objects  │
             │ SHA-256 evidence linkage    │
             └───────┬─────────────┬───────┘
                     │             │
             ┌───────▼──────┐ ┌────▼─────────────┐
             │ Audit Trail  │ │ Reports / PDFs   │
             └──────────────┘ └────┬─────────────┘
                                   ▼
                           Interactive Dashboard

# Technologies used

Core engineering

Python 3 — control-testing engine, data generation, reporting, PDF generation, and orchestration.

JSON — portable evidence store for identities, HR records, entitlements, findings, and audit-trail events.

Bash — one-command pipeline through run.sh.

HTML / CSS / JavaScript — self-contained interactive reviewer dashboard with no frontend framework dependency.

# Python libraries

Faker — deterministic synthetic employee and identity generation.

ReportLab — generation of individual audit workpapers and the combined case-file PDF pack.

pypdf — PDF validation / handling within the workpaper pipeline.

hashlib / SHA-256 — evidence integrity and traceability using Python's standard library.

Security / audit concepts modeled

Identity and Access Management (IAM)

## IT General Controls (ITGC)

Joiner / Mover / Leaver (JML) lifecycle

Privileged Access Management concepts

Multi-Factor Authentication (MFA)

Least privilege

Segregation of Duties (SoD)

Access recertification

Evidence retention and integrity

NIST SP 800-53 control concepts

SOX ITGC access-control concepts

COSO-aligned control thinking

Framework references in this portfolio are used to model audit/control concepts. This project is not an official compliance certification or proprietary audit methodology.

## Evidence and finding model

A useful security control test should be explainable after it runs. Each finding therefore carries more than an exception count.

Finding
├── finding_id
├── test_id
├── criteria
├── condition
├── cause
├── risk
├── recommendation
├── population
├── exceptions
├── sample_size
├── evidence[]
├── evidence_hash
├── management_response
└── remediation_status

The evidence hash is calculated from the retained evidence records. If the underlying evidence changes, the resulting hash changes as well. This creates a simple integrity check between the finding and the records that supported it.

## Engineering decisions

1. Deterministic synthetic data

The generator is seeded so the same source data produces the same exceptions and evidence hashes. This makes the project suitable for regression testing and demonstration without exposing real employee data.

2. Pure-function control tests

Each control test is implemented independently instead of burying all rules in one monolithic script. This makes tests easier to reason about, validate, and extend.

3. Explainable severity rules

Severity is derived from defined rules instead of assigning a rating manually after the test runs. Privileged-account exceptions receive greater impact weighting, while certain inherently risky conditions have minimum severity floors.

4. Full-population testing where practical

Where the synthetic dataset allows it, the engine tests the entire population rather than relying only on samples. Sampling is demonstrated separately for evidence-retention testing.

5. Security boundary is explicit

Automation handles repeatable evidence analysis. It does not claim to replace:

stakeholder walkthroughs,

control-design assessment,

contextual exception validation,

compensating-control evaluation,

management discussion,

or professional audit judgment.

That separation is intentional.

Outputs

Running the project generates several different reviewer artifacts.

Interactive audit dashboard

dashboard/index.html

A self-contained browser experience for reviewing engagement metrics and drilling into individual findings, evidence, risk, recommendations, and case files.

Finding dataset

reports/findings.json

Machine-readable structured findings generated by the control engine.

Audit trail

reports/audit_trail.json

Records each control execution so both passes and failures are represented in the audit record.

Management audit report

reports/AUDIT_REPORT.md

Executive summary, severity roll-up, and detailed findings.

PDF audit workpapers

dashboard/case-files/

Contains one workpaper per finding, a combined case-file pack, and an audit workpaper field guide.

Run locally

Requirements

Python 3

pip

Install dependencies:

pip install -r requirements.txt

Run the full pipeline:

./run.sh

The pipeline executes:

1. Generate synthetic evidence
2. Run all eight control tests
3. Build findings and audit trail
4. Generate management report
5. Rebuild interactive dashboard
6. Generate PDF workpapers

Then open:

dashboard/index.html

Run stages manually

python3 data/generate.py
python3 engine/tests.py
python3 engine/report.py
python3 engine/dashboard.py
python3 engine/case_files.py
python3 engine/test_engine.py

Testing strategy

The project includes automated checks for the control engine rather than treating generated findings as automatically correct.

Testing covers:

expected detection of intentionally injected control failures,

deterministic output across identical runs,

negative / clean scenarios to guard against false positives,

finding-model consistency.

Run the engine test suite with:

python3 engine/test_engine.py

Repository structure

audit-workbench/
├── dashboard/
│   ├── index.html
│   └── case-files/
│       ├── AUD-001-case-file.pdf
│       ├── ...
│       ├── AUD-008-case-file.pdf
│       ├── all-audit-case-files.pdf
│       └── audit-workpaper-field-guide.pdf
├── data/
│   ├── generate.py
│   ├── identities.json
│   ├── hr_feed.json
│   └── entitlements.json
├── engine/
│   ├── tests.py
│   ├── report.py
│   ├── dashboard.py
│   ├── case_files.py
│   └── test_engine.py
├── reports/
│   ├── findings.json
│   ├── audit_trail.json
│   └── AUDIT_REPORT.md
├── docs/
│   ├── DESIGN.md
│   ├── CONTROL_TESTS.md
│   ├── ISSUES.md
│   └── PORTFOLIO_COPY.md
├── requirements.txt
├── run.sh
├── LICENSE
└── README.md

Enterprise evolution

If I were evolving this from a portfolio workbench into a production security/audit platform, the next steps would be:

Live identity connectors — Entra ID, Okta, Active Directory, Workday, AWS IAM.

Central evidence store — Postgres plus encrypted object storage instead of local JSON.

RBAC — separate auditor, reviewer, control-owner, and administrator permissions.

Secrets management — managed credentials for source-system connectors.

Continuous control monitoring — scheduled testing and drift detection instead of point-in-time execution.

Finding workflow — management responses, ownership, due dates, retesting, and closure approvals.

Ticketing integration — ServiceNow / Jira remediation workflows.

Immutable retention — enterprise evidence-retention controls and WORM-capable storage.

Policy-as-code — configurable rules so control logic can be reviewed and versioned independently of application code.

Observability — execution metrics, failed connector alerts, test health, and evidence freshness monitoring.

What this project demonstrates

From a cybersecurity / security engineering perspective, this project demonstrates my ability to think across more than one layer of a control problem:

translate a security requirement into a testable condition,

normalize multiple evidence sources,

automate IAM risk detection,

design explainable rule logic,

preserve evidence integrity,

distinguish technical exceptions from business risk,

generate artifacts for both technical reviewers and management,

test the automation itself,

document architectural tradeoffs,

and communicate where automation should stop and human judgment begins.

The most important lesson behind the project is simple: a control is only as useful as the evidence that proves it is operating.

Security and privacy

All employee and company data in this repository is synthetic.

No real credentials, tokens, secrets, or personal employee records are required.

The engine reads source evidence and generates derivative findings; it does not require write access to a real identity platform.

Evidence integrity is modeled using SHA-256 hashing.

If this design were connected to production identity systems, additional safeguards would be required, including encryption at rest and in transit, connector least privilege, secrets management, RBAC, retention controls, monitoring, and formal data-handling requirements.

Documentation

docs/DESIGN.md — architecture, methodology, technical decisions, risks, and roadmap.

docs/CONTROL_TESTS.md — objective, inputs, logic, exceptions, severity, and evidence expectations for every control.

reports/AUDIT_REPORT.md — generated management-facing audit report.

dashboard/case-files/all-audit-case-files.pdf — combined reviewer workpaper pack.

Disclaimer

This is a portfolio case study built with synthetic data. Northstar Financial Services is fictional. The project is designed to demonstrate cybersecurity, IAM, automation, and IT-audit concepts and does not represent an audit opinion, compliance certification, or the proprietary methodology/templates of any audit firm.
