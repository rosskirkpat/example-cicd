## Developer Story

New Registry:
secure-registry.company.com

New Registry Environments:
qa.secure-registry.company.com
uat.secure-registry.company.com
prod.secure-registry.company.com

- Sprint 1 begins and dev scope is to implement a new container registry offering into the application. The registry will host certain container images that are required to be built in a more compliant/secure environment. The requirements:
  - hardcode value for registry at build-time and make it immutable
  - provide token-based authentication mechanism to the application and its' downstream consumers
  - maintain existing functionality of registries already configured
  -  implement additional logging to allow users to validate that their image was pulled from the correct registry
  -  implement a new field in the backend that reads metadata from a container image and displays the OCI labels for the container (allows an easy UX to validate source of the container they are running)

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
