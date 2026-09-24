# Release Instructions

This project releases through the GitHub Actions workflow in `.github/workflows/release.yml`.
You do not need to run Maven release commands manually. Create the correct release branch and push it; the workflow prepares and performs the release.

## Before You Start

Make sure:

- You have permission to push branches and tags.
- The build is green before releasing.
- The root `pom.xml` version is in `MAJOR.MINOR.PATCH-SNAPSHOT` format without leading zeroes.
  - Example: `1.0.0-SNAPSHOT`
- The required GitHub secrets are configured:
  - `CI_DEPLOY_USERNAME`, `CI_DEPLOY_PASSWORD`
  - `CI_GPG_PRIVATE_KEY`, `CI_GPG_PASSPHRASE`
  - `GH_TOKEN`
- The release workflow has `contents: write` permission so Maven can push release commits and tags.

Check the current project version:

```bash
./mvnw help:evaluate -Dexpression=project.version -q -DforceStdout
```

The release version is the current Maven version without `-SNAPSHOT`.
The release branch type controls the next development version only.

Example:

```text
Current version: 1.0.0-SNAPSHOT
Release version: 1.0.0
Release tag:     v1.0.0
```

## How to Release

Start from the latest release-ready branch, usually `main`:

```bash
git checkout main
git pull origin main
```

Then create one of the supported release branches below and push it. The workflow is manual-only: after pushing the branch, run `.github/workflows/release.yml` from GitHub Actions and select the release branch as the workflow branch. If the selected branch does not start with `release/`, the release job is skipped.

## Major Release Example

Use this when releasing breaking changes or a new major version line.

If the current version is `1.0.0-SNAPSHOT`:

- The workflow releases `1.0.0`.
- The workflow prepares the next development version as `2.0.0-SNAPSHOT`.

Create and push the branch, then run the release workflow manually using this branch:

```bash
git checkout -b release/major
git push origin release/major
```

## Minor Release Example

Use this when releasing normal new features.

If the current version is `1.0.0-SNAPSHOT`:

- The workflow releases `1.0.0`.
- The workflow prepares the next development version as `1.1.0-SNAPSHOT`.

Create and push the branch, then run the release workflow manually using this branch:

```bash
git checkout -b release/minor
git push origin release/minor
```

## Patch Release Example

Use this when releasing a bug fix for the current minor version.

If the current version is `1.0.0-SNAPSHOT`:

- The workflow releases `1.0.0`.
- The workflow prepares the next development version as `1.0.1-SNAPSHOT`.

Create and push the branch, then run the release workflow manually using this branch:

```bash
git checkout -b release/patch
git push origin release/patch
```

## What the Workflow Does

After the workflow is manually started on a `release/` branch, `.github/workflows/release.yml` will:

- Check out the release branch with full Git history.
- Set up Temurin JDK 21 and Maven Central/GPG credentials.
- Resolve Maven dependencies and plugins using `settings.xml`.
- Compute:
  - `RELEASE_VERSION` from the current Maven version without `-SNAPSHOT`.
  - `TAG` as `v<release-version>`.
  - `DEVELOPMENT_VERSION` from the release branch type.
- Validate that the current Maven version is in `MAJOR.MINOR.PATCH-SNAPSHOT` format without leading zeroes.
- Run `mvn -B release:prepare -P release` with the computed release version, next development version, and tag.
- Run `mvn -B release:perform -P release -DinteractiveMode=false`.

The `release` Maven profile signs release artifacts and attaches source and Javadoc JARs. The workflow is Maven-focused and publishes through the configured Maven release and central publishing setup.

## After the Release

Verify:

- The GitHub Actions workflow completed successfully.
- Log in to Sonatype to verify artifacts and click the publish button to publish to Maven Central. Note: Ask FINOS admin for Sonatype credentials.
- The Git tag exists, for example `v1.0.0`.
- The release commit and next development version commit were pushed.
- The root `pom.xml` was moved to the expected next `-SNAPSHOT` version.

## Notes

- Use `release/major`, `release/minor`, or `release/patch` for manual releases.
- The workflow also recognizes child branches such as `release/major/<name>`, `release/minor/<name>`, and `release/patch/<name>`.
- The workflow can be started manually from other branches, but the release job will be skipped unless the selected branch starts with `release/`.
- `hotfix/*` branches are not supported by the current workflow.
- For exact workflow steps, see `.github/workflows/release.yml`.
