# Course Outline: Google Cloud DevOps Release and Rollback Patterns

**Author:** Ian Fogelman  
**Length:** 90 minutes  
**Level:** Intermediate  
**Skill Path:** CI/CD Architecture for Cloud Platforms  
**Job Roles:** Cloud Solutions Architect · Cloud Engineer · DevOps Engineer  
**Tags:** Google Cloud Platform · Cloud Build · Cloud Deploy · Continuous Delivery

---

## Learner Profile

A cloud delivery engineer working in Google Cloud who needs to choose and apply safe release strategies, validate deployments, and recover from failed releases across common Google Cloud deployment targets. The learner understands how to deploy to at least one Google Cloud target but has not yet formalized a release strategy or practiced systematic rollback.

**Prerequisites assumed:**
- Working knowledge of at least one Google Cloud deployment target (Cloud Run, GKE, Cloud Functions, or Compute Engine)
- Familiarity with CI concepts (builds, artifacts, pipelines)
- Basic understanding of YAML and command-line tooling

**Out of scope:**
- IaC authoring (Terraform, Config Connector)
- Kubernetes administration fundamentals
- Observability foundations
- Multi-account governance or enterprise platform design

---

## Terminal Objectives

| # | Terminal Objective |
|---|---|
| T1 | Apply Google Cloud release strategies to cloud workloads. |
| T2 | Plan Google Cloud rollback and recovery paths for failed releases. |

---

## Module 1 — Release Patterns & Targets

**Phase:** KNOW  
**Covers:** Terminal 1 (part)  
**Estimated length:** ~28 minutes

### Overview

Before writing a single pipeline step, a delivery engineer needs to match the right release pattern to the right deployment target. This module maps the five core release approaches available in Google Cloud to the deployment targets that support them and explains the production risk trade-offs of each choice.

### Enabling Objectives

- Match Google Cloud deployment targets to release patterns that reduce production risk.
- Compare canary, blue/green, traffic-shifting, staged, and replacement release approaches in Google Cloud.

---

### Clip 1 — The Five Release Patterns on Google Cloud (~5 min)

Introduces the five patterns covered in the course — canary, blue/green, traffic-shifting, staged rollout, and replacement — and frames them around a single question: how much of your production traffic are you willing to risk on an unproven version at each step? Each pattern is described in terms of rollout speed, blast radius if it fails, and the rollback cost.

| Pattern | Traffic split | Blast radius | Rollback cost |
|---|---|---|---|
| Canary | Small % initially | Low | Low — shift traffic back |
| Blue/Green | Full switch at cutover | Medium | Low — route back to blue |
| Traffic-shifting | Incremental % steps | Graduated | Low — shift back |
| Staged rollout | Region or instance subset | Medium | Medium — redeploy previous |
| Replacement | Full swap, old gone | High | High — redeploy from artifact |

---

### Clip 2 — Google Cloud Deployment Targets (~6 min)

Covers the four deployment targets in scope — Cloud Run, GKE, Cloud Functions, and Compute Engine — describing how each handles versioning, traffic routing, and rollback at the platform level. Establishes the vocabulary (revisions, deployments, instance groups, function versions) used throughout the course.

**Cloud Run:** Revisions are immutable. Traffic is split by revision using percentage-based routing. Rollback = shift 100% traffic to a prior revision.

**GKE:** Deployments manage replica sets. Rolling updates replace pods gradually. Rollback via `kubectl rollout undo` or by applying a previous manifest.

**Cloud Functions:** Each deploy creates a new version. Traffic can be split between versions (Gen 2). Rollback = redirect traffic to a prior version.

**Compute Engine:** Managed Instance Groups (MIGs) support rolling actions and canary updates using instance templates. Rollback = apply the previous instance template.

---

### Clip 3 — Matching Patterns to Targets (~6 min)

Works through the pattern-to-target matrix, explaining which patterns are native to each target and which require additional tooling or workarounds. Introduces the decision criteria: change risk (how risky is this change?), validation time needed (how long does it take to know if it's good?), and rollback urgency (how fast do you need to recover?).

**Pattern-to-target fit:**

| | Cloud Run | GKE | Cloud Functions | Compute Engine |
|---|---|---|---|---|
| Canary | ✓ native | ✓ via Deployment/Istio | ✓ Gen 2 traffic split | ✓ MIG canary |
| Blue/Green | ✓ via traffic tags | ✓ via service switch | ✓ via alias | ✓ via two MIGs |
| Traffic-shifting | ✓ native | ✓ Istio/Gateway | ✓ Gen 2 | ✓ via gradual rollout |
| Staged rollout | ✓ per region | ✓ rolling update | — | ✓ MIG rolling action |
| Replacement | ✓ deploy new revision | ✓ rolling replace | ✓ deploy new version | ✓ recreate MIG |

---

### Clip 4 — Choosing the Right Pattern (Decision Demo) (~6 min)

Walks through three realistic delivery scenarios and applies the decision criteria to arrive at a recommended pattern:

1. A stateless API on Cloud Run receiving steady traffic — favors traffic-shifting with a small initial canary slice.
2. A batch job on Compute Engine with a 2-hour warm-up window — favors blue/green with a cutover gate.
3. A GKE microservice with a flaky upstream dependency — favors staged rollout with per-region health validation.

Demonstrates how to read the scenario signals (change type, traffic sensitivity, validation window) rather than defaulting to one pattern for everything.

---

### Clip 5 — Lab Introduction: Mapping Patterns to a Scenario (~5 min)

Introduces the hands-on lab. Learners receive a fictional delivery scenario and a set of system constraints and must select an appropriate release pattern for each of two deployment targets, justify the choice, and identify where rollback points would be inserted. No infrastructure is provisioned in this lab — it is a decision and documentation exercise using the matrix from Clip 3.

**Lab: Release Pattern Decision Exercise**
- Scenario A: Cloud Run service, latency-sensitive, canary preferred
- Scenario B: GKE workload, schema migration in progress, staged rollout required
- Deliverable: completed decision worksheet with pattern choice, rationale, and rollback point

---

## Module 2 — Pipeline Control & Execution

**Phase:** EXECUTE  
**Covers:** Terminal 1 (part)  
**Estimated length:** ~32 minutes

### Overview

Knowing the right pattern is not enough — you need a pipeline that enforces it. This module builds a complete Cloud Deploy delivery pipeline from scratch, adds Cloud Build as the artifact producer, configures approvals and gates between stages, and maps the staged promotion flow so learners can see exactly where validation and rollback points live.

### Enabling Objectives

- Use Cloud Build, Cloud Deploy, approvals, and delivery pipelines to control release movement.
- Map staged promotion, validation, and rollback points into a Google Cloud release flow.

---

### Clip 1 — Cloud Build as the Artifact Producer (~5 min)

Explains Cloud Build's role upstream of Cloud Deploy: compiling code, running tests, building a container image, pushing to Artifact Registry, and signing the artifact. Covers the `cloudbuild.yaml` structure — steps, substitutions, and artifact output — and explains how artifact trust (a signed, tagged image in Artifact Registry) is what Cloud Deploy acts on, not raw source code.

Key concepts: build triggers, substitution variables (`$SHORT_SHA`, `$BRANCH_NAME`), Artifact Registry repositories, binary authorization basics.

---

### Clip 2 — Cloud Deploy Delivery Pipelines (~7 min)

Introduces Cloud Deploy's core objects: delivery pipeline, targets, releases, and rollouts. Walks through a three-target pipeline (`dev → staging → prod`) defined in `clouddeploy.yaml`, showing how each target maps to a Cloud Run service or GKE cluster. Explains the promotion model: a release is immutable; promoting it moves the same artifact through targets without rebuilding.

Demonstrates creating a release and observing the automatic rollout to `dev`.

```yaml
# clouddeploy.yaml (simplified)
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: my-app-pipeline
serialPipeline:
  stages:
    - targetId: dev
    - targetId: staging
      profiles: [staging]
    - targetId: prod
      profiles: [prod]
```

---

### Clip 3 — Approvals, Gates, and Manual Promotion (~6 min)

Shows how to require a human approval before a rollout proceeds to a sensitive target. Configures `requireApproval: true` on the `staging` and `prod` targets. Demonstrates the approval workflow in the Google Cloud Console and via `gcloud deploy releases promote`. Explains when automated gates (based on Skaffold verify or custom scripts) supplement or replace manual approvals.

Introduces the concept of a rollback point: every approved promotion is a point to which Cloud Deploy can roll back without re-running the pipeline.

---

### Clip 4 — Skaffold and Deployment Manifests (~5 min)

Explains how Skaffold acts as the deployment engine within Cloud Deploy. Covers `skaffold.yaml` profiles that control which manifests are applied per target (dev vs prod replica counts, resource limits, feature flags). Shows how to use `skaffold verify` to run post-deploy validation tests automatically, turning a simple pipeline step into a gate.

---

### Clip 5 — Mapping the Full Release Flow (~4 min)

Ties the previous clips together into a single end-to-end diagram: commit → Cloud Build → Artifact Registry → Cloud Deploy release → dev rollout → automated verify → promotion → staging approval → staging rollout → prod approval → prod rollout. Marks the rollback points (after each approved rollout) and the validation checkpoints (Skaffold verify steps, approval gates). This diagram becomes the reference architecture for Module 3.

---

### Clip 6 — Lab Introduction: Building a Delivery Pipeline (~5 min)

Introduces the hands-on lab. Learners create a Cloud Deploy delivery pipeline targeting a Cloud Run service with three stages (dev, staging, prod), trigger a build in Cloud Build, promote the release through the pipeline, and configure a manual approval on the prod target.

**Lab: Cloud Deploy Pipeline with Approvals**
- Create Cloud Deploy pipeline and targets (`clouddeploy.yaml`)
- Trigger a Cloud Build run, push image to Artifact Registry
- Create a Cloud Deploy release and observe `dev` rollout
- Promote to `staging`, then configure and exercise prod approval
- Verify: prod rollout completes with the correct image SHA

---

## Module 3 — Validation & Recovery

**Phase:** RECOVER  
**Covers:** Terminal 2  
**Estimated length:** ~30 minutes

### Overview

A release pipeline that can only go forward is incomplete. This module focuses on what happens when a rollout goes wrong: how to read the signals that indicate failure, how to choose between rollback, retry, and forward-fix for each deployment target, how to actually execute the rollback, and how to document the incident so the next handoff is clean.

### Enabling Objectives

- Define validation signals that indicate release success or failure.
- Select rollback options for Cloud Run, Cloud Functions, GKE, Compute Engine, and managed workloads.
- Use release evidence to choose rollback, retry, or forward-fix actions.
- Document the recovery handoff needed after a failed Google Cloud release.

---

### Clip 1 — Validation Signals: Knowing When a Release Has Failed (~6 min)

Covers the three categories of release validation signals and where to find them in Google Cloud:

**Traffic signals (Cloud Monitoring):** HTTP error rate (4xx/5xx), latency p50/p95/p99, throughput drop. These are the fastest indicators and can be automated.

**Health signals (target-native):** Cloud Run instance health checks, GKE readiness/liveness probes, Compute Engine health check pass/fail on the MIG. These block traffic from reaching unhealthy instances.

**Functional signals (logs and traces):** Application error patterns in Cloud Logging, trace error spans in Cloud Trace, custom metric thresholds. Slower but more precise.

Explains the difference between a signal that means *stop immediately* (spike in 5xx errors) versus one that means *watch and decide* (slight latency increase within SLO). Introduces the concept of a rollback threshold: a pre-agreed metric value that triggers a rollback decision without debate.

---

### Clip 2 — Rollback by Target (~7 min)

Walks through the rollback mechanism for each of the four deployment targets, showing the exact command or console action for each.

**Cloud Run:**
```bash
# Shift 100% traffic back to previous revision
gcloud run services update-traffic SERVICE \
  --to-revisions=PREV_REVISION=100 \
  --region=REGION
```

**GKE (Deployment):**
```bash
# Roll back to the previous replica set
kubectl rollout undo deployment/my-app -n my-namespace

# Roll back to a specific revision
kubectl rollout undo deployment/my-app --to-revision=3 -n my-namespace
```

**Cloud Functions (Gen 2):**
```bash
# Shift traffic back to a known-good version
gcloud functions deploy FUNCTION_NAME \
  --serve-all-traffic-latest-revision=false
# Then update traffic split via console or API
```

**Compute Engine (MIG):**
```bash
# Roll back a MIG to a previous instance template
gcloud compute instance-groups managed rolling-action start-update MIG_NAME \
  --version=template=PREV_TEMPLATE \
  --region=REGION
```

Also covers Cloud Deploy's built-in rollback:
```bash
gcloud deploy rollbacks create --delivery-pipeline=PIPELINE \
  --release=RELEASE --rollout=ROLLOUT --target-id=TARGET
```

---

### Clip 3 — Rollback, Retry, or Forward-Fix (~5 min)

Introduces the three-way decision framework using release evidence:

| Signal | Likely cause | Recommended action |
|---|---|---|
| Immediate spike in 5xx errors, all instances | Bad artifact, config error | **Rollback** — shift traffic immediately |
| Errors on a subset of instances, others healthy | Flaky deployment, infrastructure noise | **Retry** — redeploy to affected instances |
| Latency increase, no errors, within SLO | Performance regression, not critical | **Forward-fix** — patch and promote a new release |
| Errors tied to data migration, schema mismatch | Deployment order issue | **Forward-fix** — apply the missing migration, re-verify |
| Errors in one region only | Regional infrastructure issue | **Retry** — rerun in that region, or failover |

Emphasizes that the decision must be made against pre-agreed thresholds defined before deployment, not in the heat of an incident.

---

### Clip 4 — Reading Release Evidence (~4 min)

Shows where to gather the evidence needed to make the rollback/retry/forward-fix decision in Google Cloud:

- **Cloud Monitoring:** alert policies on error rate and latency tied to a release label; SLO burn rate alerts
- **Cloud Logging:** structured log queries filtered by revision or deployment label to isolate errors introduced by the new version
- **Cloud Deploy audit log:** records every rollout action, approval, and promotion — the definitive record of what deployed when
- **Artifact Registry:** confirms the exact image SHA running in each target, enabling comparison between good and bad versions

Demonstrates a 60-second investigation flow: check Monitoring for the signal, confirm in Logging which requests are failing, check Cloud Deploy to identify the rollout timestamp, compare to the previous known-good release.

---

### Clip 5 — Documenting the Recovery Handoff (~3 min)

Covers the minimum documentation needed after a failed release and recovery, using a simple structured template:

```
## Release Recovery Record

**Date / Time:**
**Service / Pipeline:**
**Release that failed:**  (image SHA, Cloud Deploy release ID)
**Target(s) affected:**
**Signal that triggered recovery:**  (metric name, threshold, observed value)
**Action taken:**  (rollback / retry / forward-fix)
**Rolled back to:**  (previous release ID or revision)
**Time to recovery (TTR):**
**Root cause (initial):**
**Follow-up required:**  (ticket ID or none)
**Handoff to:**  (name or on-call rotation)
```

Explains that this record lives in the incident ticket or runbook, not just in someone's head — it is the input to the post-incident review and to improving the release pipeline's gates and thresholds.

---

### Clip 6 — Lab Introduction: Validate, Roll Back, and Document (~5 min)

Introduces the hands-on lab. Learners deploy a deliberately broken release to a Cloud Run service via Cloud Deploy, observe the failure signal in Cloud Monitoring, execute the rollback using `gcloud run services update-traffic`, confirm the service is healthy, and complete a recovery record using the template from Clip 5.

**Lab: Validating a Failing Release and Rolling Back**
- Deploy a known-good Cloud Run revision via Cloud Deploy
- Deploy a broken revision (intentionally returns 500 on /health)
- Observe Cloud Monitoring error rate alert fire
- Execute traffic rollback to the prior revision
- Verify service health returns to baseline
- Complete the recovery record template

---

## Appendix: Key Services Reference

| Service | Role in this course |
|---|---|
| Cloud Build | Builds artifacts, runs tests, pushes to Artifact Registry |
| Artifact Registry | Stores and signs container images |
| Cloud Deploy | Manages delivery pipelines, promotions, approvals, rollbacks |
| Skaffold | Deployment engine and post-deploy verify runner |
| Cloud Run | Serverless target; revision-based traffic splitting |
| GKE | Kubernetes target; rolling updates and rollout undo |
| Cloud Functions | Serverless functions; version-based traffic split (Gen 2) |
| Compute Engine MIG | VM target; rolling actions and instance template rollback |
| Cloud Monitoring | Error rate, latency, and SLO signals for release validation |
| Cloud Logging | Structured log evidence for release investigation |

---

## Appendix: Release Pattern Decision Cheat Sheet

```
Is your change high risk or hard to validate quickly?
  YES → Use canary or traffic-shifting (small blast radius first)
  NO  → staged rollout or replacement may be acceptable

Does your target support native traffic splitting?
  Cloud Run / Cloud Functions Gen 2 → YES (use it)
  GKE → YES with Istio or Gateway API
  Compute Engine → YES via MIG canary

Do you need a clean cutover with instant rollback?
  → Blue/Green

Do you need gradual exposure with automated validation gates?
  → Traffic-shifting or canary with Skaffold verify

Is rollback cost a primary constraint (e.g., stateful workload)?
  → Avoid replacement; prefer canary or blue/green
```
