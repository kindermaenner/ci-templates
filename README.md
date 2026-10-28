# ci-templates

This repository contains reusable CI templates, composite actions, and pipelines for our GitHub projects.

It is designed with a **Zero-Artifact Architecture**: workflows execute steps on a single runner or use composite actions to avoid unnecessary `upload-artifact` usage, saving GitHub Actions storage budget and accelerating CI pipelines.

---

## Standard Pipelines (Reusable Workflows)

Standard pipelines run all steps (Lint, Test, SonarQube, Badges, Deploy) in **a single job on one runner**, eliminating intermediate artifact upload/download.

### 1. WordPress Plugin Pipeline
Runs PHPCS, PHP-CS-Fixer, Pest tests (with code coverage), SonarQube CLI scan, and badge trigger.

```yaml
# In your wordpress plugin repo: .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  ci:
    uses: kindermaenner/ci-templates/.github/workflows/wordpress-plugin-pipeline.yml@main
    with:
      php-version: "8.2"
      sonar-properties: "-Dsonar.projectKey=my-plugin"
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      BADGE_APP_ID: ${{ secrets.BADGE_APP_ID }}
      BADGE_APP_PRIVATE_KEY: ${{ secrets.BADGE_APP_PRIVATE_KEY }}
```

### WordPress Plugin Release
The release workflow is called from a plugin repository's tag-triggered workflow after its CI checks pass. It requires a `vMAJOR.MINOR.PATCH` tag, installs production Composer dependencies, packages the plugin (including `vendor/`), and uploads the versioned ZIP, checksum, a replaceable ZIP in `latest/`, and a stable update manifest to the configured FTP location. The manifest points to the versioned ZIP URL.

```yaml
on:
  release:
    types: [published]

jobs:
  release:
    uses: kindermaenner/ci-templates/.github/workflows/wordpress-plugin-release.yml@main
    with:
      plugin-file: MyPlugin.php
      plugin-slug: my-plugin
      ftp-target-path: example.com/plugin-updates/my-plugin
      public-url: https://example.com/plugin-updates/my-plugin
    secrets:
      FTP_HOST: ${{ secrets.FTP_HOST }}
      FTP_USER: ${{ secrets.FTP_USER }}
      FTP_PASSWORD: ${{ secrets.FTP_PASSWORD }}
```

The FTP destination must map to the supplied public HTTPS URL. A published GitHub Release matching the pushed tag must exist and contain release notes; this includes releases marked as pre-releases on GitHub. The workflow copies those notes to `release_notes` and includes the notes, publication date, release URL, pre-release status, security-fix marker, and compare link for all published version tags in `releases` in `manifest.json`. A release is marked as a security update when its notes include a `Security:` heading. The manifest also includes the plugin name and required PHP version from the plugin header, and the configured PHP version as `tested_php`. Keep `php-version` aligned with the version actually tested by the calling CI workflow. The plugin updater can use this data to show a cumulative linked changelog, release date, pre-release/security labels, and PHP compatibility in WordPress's update details. WordPress compatibility is not reported because this pipeline does not currently test against WordPress versions.

### 2. Android App Pipeline
Runs Gradle Build, Lint, Kover coverage, optional instrumented tests, SonarQube scan via Gradle, optional FTP upload, and badge trigger.

```yaml
# In your android app repo: .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  ci:
    uses: kindermaenner/ci-templates/.github/workflows/android-pipeline.yml@main
    with:
      keystore-file: "release.keystore"
      build-type: "Release"
      run-sonar: true
      run-ftp-upload: false
    secrets:
      RELEASE_KEYSTORE_BASE64: ${{ secrets.RELEASE_KEYSTORE_BASE64 }}
      RELEASE_KEYSTORE_PASSWORD: ${{ secrets.RELEASE_KEYSTORE_PASSWORD }}
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      BADGE_APP_ID: ${{ secrets.BADGE_APP_ID }}
      BADGE_APP_PRIVATE_KEY: ${{ secrets.BADGE_APP_PRIVATE_KEY }}
```

---

## Composite Actions (`.github/actions/`)

For custom pipelines, you can include individual composite actions directly as steps within your job:

### SonarQube CLI Scan (`.github/actions/sonar-cli`)
Executes SonarScanner CLI 8.0.1 directly on the runner without uploading/downloading coverage artifacts. The default Linux x64 package includes its own Java 21 runtime. You can override the scanner version with the `scanner-version` input.

```yaml
- name: Run SonarQube Scan
  uses: kindermaenner/ci-templates/.github/actions/sonar-cli@main
  with:
    sonar-properties: "-Dsonar.projectKey=my-project"
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
```

### SonarQube Gradle Scan (`.github/actions/sonar-gradle`)
Executes Gradle Sonar task directly on the runner.

```yaml
- name: Run SonarQube Gradle Scan
  uses: kindermaenner/ci-templates/.github/actions/sonar-gradle@main
  with:
    sonar-token: ${{ secrets.SONAR_TOKEN }}
    sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
```

### FTP Upload (`.github/actions/ftp-upload`)
Uploads build outputs (e.g., APKs or zip packages) directly via `curl --ftp-ssl` without creating GitHub artifacts.

```yaml
- name: Upload APKs to FTP
  uses: kindermaenner/ci-templates/.github/actions/ftp-upload@main
  with:
    source-path: "app/build/outputs/apk/release"
    target-path: "apps/my-app"
    ftp-host: ${{ secrets.FTP_HOST }}
    ftp-user: ${{ secrets.FTP_USER }}
    ftp-password: ${{ secrets.FTP_PASSWORD }}
```

### Sonar Badge Trigger (`.github/actions/badge-trigger`)
Triggers the centralized badge repository update workflow via GitHub App Token.

```yaml
- name: Trigger Badge Update
  uses: kindermaenner/ci-templates/.github/actions/badge-trigger@main
  with:
    badge-app-id: ${{ secrets.BADGE_APP_ID }}
    badge-app-private-key: ${{ secrets.BADGE_APP_PRIVATE_KEY }}
```

---

## Housekeeping & Cleanup

* `common-github-cleanup.yml`: Scheduled daily job to clean up workflow runs older than 30 days, clear old caches, and delete orphaned artifacts older than 1 day.

---

## License

This repository is licensed under the MIT License.
