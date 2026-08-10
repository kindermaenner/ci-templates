# ci-templates

This repository contains reusable CI templates for our GitHub projects.

The workflows provide standardized build, linting, testing, security checks, SonarQube integration, and badge updates across multiple languages and stacks (Android/Kotlin, PHP, Python).
They can be included in other repositories using GitHub’s workflow_call mechanism to avoid duplication and maintain consistent CI behavior.

# included areas
- Android: ktlint, detekt, build, tests, SonarQube, badge updates
- PHP: phpcs, php-cs-fixer, PHPUnit, dependency security checks
- Python: flake8, black, pytest, pip-audit
- Common: badge trigger, SonarQube trigger, Docker-related checks

# usage

To use a template in another repository:
```yaml
jobs:
  lint:
    uses: kindermaenner/ci-templates/.github/workflows/android-lint.yml@main
```

# goal

Provide a central, maintainable set of CI building blocks that can be reused across all projects, reducing duplication and ensuring consistent quality standards.

# license

This repository is licensed under the MIT License.
