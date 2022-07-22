## CI/CD Triggers

PR Create/Update
Tag Create/Update
Branch Create/Update
Release Create/Update
Nightlies via cronjob (ex. smoke tests)
Manual Actions (ex. manual workflows in GH Actions)

## Secret Management

Where are secrets stored?

[GitHub Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
Github and Azure Service accounts are created whenever a new tag is cut specific to the target environment and will only have permissions to push containers, manifests, and binaries to tags matching `${VERSION}-${TARGET_ENV}-${SERVICE}-sa`. tokens are injected into the `publish` stage of the build pipeline via sops



## Compliance

- build logs uploaded to read-only s3 bucket
- change management required for pushes to production
- PRs require two approvers to be merged
- PRs require unit testing and documentation to be added if a feature is added or modified
  - GitHub automation that requires any changes to files matching the following wildcards `*.go, *.sh, *Dockerfile*, *Makefile*` 
  - Two methods of overriding the testing/docs requirement using a bot that watches Github comments on the repo. These actions only work for a select group of managers that are responsible for the teams working on the product. This ensures accountability and awareness of code being added/updated without accompanying tests/docs.
    - `/approve-without-tests`
    - `/approve-without-docs`
- main branch protection - no direct pushes to main allowed
-  

## RBAC 
- use existing GitHub authentication 
- using high-level GH groups to control who has access to what repositories and who can run certain automated actions

## Process Flow for new Release

Trigger: `release/1` branch is created

v0.0.1-qa-gh-sa -> GitHub service account uses token based authentication
v0.0.1-qa-acr-sa -> Azure Container Registry service account uses token-based authentication

when `v0.0.1-uat` tag is created:
All QA Tokens are invalidated and the service accounts disabled (`v0.0.1-qa-gh-sa` and `v0.0.1-qa-acr-sa`)

v0.0.1-uat-gh-sa -> GitHub service account uses token based authentication
v0.0.1-uat-acr-sa -> Azure Container Registry service account uses token-based authentication

when `v0.0.1` tag is created:
All QA service accounts are verified as being disabled 
All UAT Tokens are invalidated and the service accounts disabled (`v0.0.1-uat-gh-sa` and `v0.0.1-uat-acr-sa`)

v0.0.1-prod-gh-sa -> GitHub service account uses token based authentication
v0.0.1-prod-acr-sa -> Azure Container Registry service account uses token-based authentication

when `v0.0.1` has a successful deployment to production:

All Production Tokens are invalidated and the service accounts disabled (`v0.0.1-prod-gh-sa` and `v0.0.1-prod-acr-sa`)

Cleanup:

24 hours after v0.0.1 is successfully deployed to production, all service accounts tied to the version are deleted barring manual intervention


Infrastructure: Azure VMs, GitHub Runners (GitHub Actions), GitHub, AKS, Azure Container Registry (ACR)


QA/UAT ACR: Private but allows external users access using token-based authentication (`ex. docker login`)
Prod ACR: Public but requires token-based authentication for any user trying to pull the image (`ex. docker login`)

---


## SBOM

https://anchore.com/sbom/creating-sbom-attestations-using-syft-and-sigstore/

Anchore syft pulls image from container registry -> generate sbom and attestation (spdx or cyclonedx) -> sigstore cosign attaches attestation to container image and pushes to container registry -> user can view/access sbom and attestation for all published container images

---

## Canary vs. Blue/Green

Approach: Hybrid blue/green approach that also borrows from the canary deployment strategy.

Two environments:

staging, comprised of qa and uat
production

Staging is sized at roughly 1/3 of production. Mainly, this reduction in compute and overall resources is accomplished at the worker level as production may need 100 nodes in a cluster to properly handle load in a HA way whereas QA only requires a small percentage of nodes to test proper HA. Load testing can be accomplished using spot instances and automated runbooks that provision, deploy, load and scale test, and then destroy.

Releases are rolled out to staging first. QA gets the first deployment and provides for extended testing that may capture issues that will not be seen in CI when it is running against a PR prior to merging. This also provides developers an active environment to debug any issues with. Once QA signs off, UAT is deployed next and should require minimal involvement from developers. Once the UAT and QA teams sign off, production deployment is scheduled and a rolling update approach is taken. This is done by leveraging upgrade strategies in kubernetes.

Both environments use Rancher as the orchestration platform managing the downstream AKS clusters

---

## Example Application

Application: internal Go and Gin RESTful web service API that serves as the backend for an internal application

Backing infrastructure: Kubernetes (Azure AKS)
Multi-Region production AKS clusters that have horizontal pod autoscaling (HPA) enabled that leverages both standard and custom metrics built as part of a collaboration between operations and developers.
Deployment strategy: 10% max unavailable pods in any given downstream cluster.


---

## Metrics

Production metrics are built on top of the HPA metrics and provide both automated and aggregated feedback. 

Load tracking over first 72 hours post-production release (cpu, memory, disk io (avg read+write, iowait, inodes, latency), http requests served, bandwidth consumption, load average, etc)
Report comparing latest release perf metrics against last 2 minor releases and latest minor from last major release
automatically generating reports of code changes when >5% overall environment load increase or >10% for a single component/app post-release and creating a GH issue and assigning to team based on CODEOWNERS file
feedback incorporated into resource limits/requests for apps (via PR by developers)
