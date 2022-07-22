## Developer Story

---

Developer workflow

GitHub: PR -> Review -> Approve -> Merge to `main`
When sprint 1 ends, Lead Dev for th sprint cuts `release/1` branch from `main

---

### Feature request

- Sprint 1 begins and dev scope is to implement a new container registry offering into the application. The registry will host certain container images that are required to be built in a more compliant/secure environment. The requirements:
  - hardcode value for registry at build-time and make it immutable
  - provide token-based authentication mechanism to the application and its' downstream consumers
  - maintain existing functionality of registries already configured
  -  implement additional logging to allow users to validate that their image was pulled from the correct registry
  -  implement a new field in the backend that reads metadata from a container image and displays the OCI labels for the container (allows an easy UX to validate source of the container they are running)

New Registry:
secure-registry.company.com

New Registry Environments:
qa.secure-registry.company.com
prod.secure-registry.company.com

dev will fork the `main` product repo branch and create a branch named `sprint-1/feature/registry`. They will work inside of this branch exclusively. When the feature is ready for review, they will create a PR against the upstream `main` branch from their `sprint-1/feature/registry`. The PR will be reviewed and once it is approved, it will be merged into the `main` upstream branch.

Once Sprint 1 has completed (4 weeks) and all PRs are merged, QA will begin their validation and integration testing.

Scenario: While validating in preparation for cutting the `release/1` branch, QA files a bug with the developer due to the new registry feature in the application not properly logging that a container was pulled from the secure registry.

The developer will perform the necessary code changes and then open a PR against the `main` branch and once it has been reviewed and approved, it will be merged and QA will perform testing specific to the areas surrounding the registry feature. 

Following the release flow model, the `release/1` branch will then be deployed to various environments in stages with the ultimate goal of deploying to production.

QA validates that the dev's fix addresses the issue and now the QA Manager will create a `release/1` branch from `main`. When this is done, a tag `v0.0.1-qa` is created based on automation using `${VERSION}-${TARGET_ENV}`. container images and binaries are built in CI and images are deployed to either qa.secure-registry.company.com or qa.registry.company.com and binaries are published directly to page for the Github tag `v0.0.1-qa`

`release/1` branch is first deployed to QA environment to be run through the full gambit of automated testing as well as manual verification of new features.
What this looks like: QA verifies that certain container images are pulled from `qa.secure-registry.company.com` instead of `qa.registry.company.com`. They also validate that the application is logging when images are pulled from the secure registry instead of the standard registry. Lastly, they ensure that the UX/UI properly displays the labels from container images originating from both registries.

after QA signs off, QA manager and the UAT team manager approves creating a new tag `v0.0.1-uat` from the `release/1` branch that will be deployed to UAT environment upon successful CI run. container images and binaries are built in CI and images are deployed to either uat.secure-registry.company.com or uat.registry.company.com and binaries are published directly to page for the Github tag `v0.0.1-uat`

after a successful validation from the UAT group, QA, UAT, and Dev managers create a new release with the tag `v0.0.1` from the `release/1` branch. This tag will be deployed to the `production` environment. The developers feature is now live in production.

---

Automation creates QA artifacts

GitHub Actions kicks off CI due to a new `release/*` branch being created:
`v0.0.1-qa` is tagged and service accounts are created

v0.0.1-uat-gh-sa -> GitHub service account uses token based authentication
v0.0.1-uat-acr-sa -> Azure Container Registry service account uses token-based authentication

triggering image + binary builds, integration + validation tests, and then publishing of artifacts if CI completes successfully
All binaries and tarballs are uploaded to the `v0.0.1-qa` tag
containers images are published to ACR for either `qa.secure-registry.company.com` or `qa.registry.company.com`

---

QA Manager creates tag `v0.0.1-uat` which triggers:
service accounts are created
v0.0.1-uat-gh-sa -> GitHub service account uses token based authentication
v0.0.1-uat-acr-sa -> Azure Container Registry service account uses token-based authentication

image + binary builds, standard integration + validation tests, and then publishing of artifacts if CI completes successfully
All binaries and tarballs are uploaded to the `v0.0.1-uat` tag
containers images are published to ACR for either `uat.secure-registry.company.com` or `uat.registry.company.com`
All QA Tokens are invalidated and the service accounts disabled (`v0.0.1-uat-gh-sa` and `v0.0.1-uat-acr-sa`)

---

UAT Team manager creates a new release with tag `v0.0.1`
service accounts are created
v0.0.1-prod-gh-sa -> GitHub service account uses token based authentication
v0.0.1-prod-acr-sa -> Azure Container Registry service account uses token-based authentication

All QA service accounts are verified as being disabled 
All UAT Tokens are invalidated and the service accounts disabled (`v0.0.1-uat-gh-sa` and `v0.0.1-uat-acr-sa`)

image + binary builds, standard integration + validation tests, and then publishing of artifacts if CI completes successfully
All binaries and tarballs are uploaded to the `v0.0.1` release
containers images are published to ACR for either `prod.secure-registry.company.com` or `prod.registry.company.com`
All QA Tokens are invalidated and the service accounts disabled (`v0.0.1-prod-gh-sa` and `v0.0.1-prod-acr-sa`)

once CI has finished and all artifacts have been validated (incl. smoke testing), v0.0.1 is rolled out to production.

once `v0.0.1` has a successful deployment to production:

All Production Tokens are invalidated and the service accounts disabled (`v0.0.1-prod-gh-sa` and `v0.0.1-prod-acr-sa`)

A custodian-style automation runs and ensures that no resources remain from the QA and UAT processes of the release. This reduces the risk of unnecessary cloud-spend because of forgotten cloud resources (VMs, Storage Blobs, Virtual Disks, etc)

Cleanup:

24 hours after v0.0.1 is successfully deployed to production, all service accounts tied to the version are deleted barring manual intervention. 

---

Why is manual intervention required as part of the design?

To get initial buy-in on the new CI/CD system, the relevant people need to be considered and involved from the get-go.

Giving a manager the "power" to approve/deny a release empowers both the manager and their team and enforces the idea that every team matters. Ideally, it should never get to a point where a manager is the only person stating that a release _should not_ happen because that implies that there were massive process and policy failures prior.

Ultimately, it is easy to lose sight of the people involved with the development, testing, and deployment cycles until a large enough problem develops with the people involved that it makes the tech become a secondary concern.

In the future, it is possible that manual intervention will only be required for pushing to production but until _every_ team is comfortable with the new process, the new CI/CD pipeline, and 100% buy-in has been achieved, it should not be a priority.

To play devil's advocate - it is important to have qualified and available backups to anyone in charge of every step of a release. A process is broken if a manager going on vacation means that a release is impossible until they return.

Change management has to be both valuable to the risk assessment and executive teams and act as a non-blocker to the operations and developer teams who are working within the CM process. CM is only effective when it is a two-way street and both parties work together. 

---

