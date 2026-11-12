# Course Planning: Google Cloud DevOps Release and Rollback Patterns

---

## Learner Profile

Cloud delivery engineers working in Google Cloud who are responsible for shipping application changes safely, validating that deployments succeed, and recovering quickly when they do not. This includes engineers who currently deploy to Cloud Run, GKE, Cloud Functions, or Compute Engine but have not yet formalized a release strategy or practiced systematic rollback across those targets.

---

## Learner Prerequisites

- Hands-on experience deploying to at least one Google Cloud target (Cloud Run, GKE, Cloud Functions, or Compute Engine).
- Familiarity with CI/CD concepts: what a build pipeline does, what an artifact is, and what "promoting to production" means.
- Comfort reading and writing YAML configuration files.
- Basic command-line experience with `gcloud` or `kubectl`.
- Google Cloud fundamentals (IAM, projects, regions) are assumed but not tested.

---

## Purpose

Delivery engineers on Google Cloud frequently ship directly to production without a defined release strategy, discover failures reactively through user reports or alert storms, and roll back manually using ad-hoc commands with no documented process. The result is longer outages, repeated mistakes, and high stress during incidents.

This course gives engineers a repeatable system: a way to choose the right release pattern for the deployment target and change type, a Cloud Build and Cloud Deploy pipeline that enforces staged promotion with gates and approvals, and a clear decision process for rolling back, retrying, or forward-fixing when a release fails. When engineers finish this course they will ship with less risk and recover with less chaos.

---

## Author Notes

- Will need a GitHub repo with working sample code: `cloudbuild.yaml`, `clouddeploy.yaml`, `skaffold.yaml`, and Kubernetes manifests for a simple demo application targeting Cloud Run and GKE.
- Lab environments will need a GCP project with Cloud Build, Cloud Deploy, Artifact Registry, Cloud Run, and GKE APIs enabled.
- The broken-release lab (Module 3) requires a pre-built container image that intentionally returns HTTP 500 on `/health` to trigger the rollback scenario.
- Diagram assets (`course-diagram.svg`) and the full clip-level outline (`course-outline.md`) are in this repository.
- AI-assisted workflow examples (log summarization, release evidence review) may be incorporated lightly in Module 3 clip 4 — confirm with curriculum before recording.

---

## Short Description

*(250 character limit)*

Safe releases fail less, and failed releases recover faster. This course teaches you to choose and apply Google Cloud release strategies using Cloud Build, Cloud Deploy, and platform-native rollback across Cloud Run, GKE, Cloud Functions, and Compute Engine.

---

## Long Description

Shipping to production without a release strategy means every deployment is a gamble — and every failure becomes an all-hands scramble. In this course, **Google Cloud DevOps Release and Rollback Patterns**, you'll gain the ability to choose, build, and execute safe release strategies on Google Cloud and recover cleanly when things go wrong.

First, you'll explore the five core release patterns — canary, blue/green, traffic-shifting, staged rollout, and replacement — and learn how to match each one to the right Google Cloud deployment target based on change risk and rollback cost.

Next, you'll discover how to build a controlled delivery pipeline using Cloud Build, Cloud Deploy, and Skaffold, with staged promotion across dev, staging, and production, manual approvals, automated validation gates, and clear rollback points at every stage.

Finally, you'll learn how to respond when a release fails: reading validation signals from Cloud Monitoring and Cloud Logging, choosing between rollback, retry, and forward-fix using release evidence, executing the rollback for each deployment target, and documenting the recovery handoff so the next person on-call has what they need.

When you're finished with this course, you'll have the skills and practical patterns needed to ship on Google Cloud with confidence — and to recover quickly and systematically when a release doesn't go as planned.
