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
