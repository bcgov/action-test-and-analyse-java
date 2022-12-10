<!-- Badges -->
[![Issues](https://img.shields.io/github/issues/bcgov-nr/action-test-and-analyse-java)](/../../issues)
[![Pull Requests](https://img.shields.io/github/issues-pr/bcgov-nr/action-test-and-analyse-java)](/../../pulls)
[![MIT License](https://img.shields.io/github/license/bcgov-nr/action-test-and-analyse-java.svg)](/LICENSE)
[![Lifecycle](https://img.shields.io/badge/Lifecycle-Experimental-339999)](https://github.com/bcgov/repomountie/blob/master/doc/lifecycle-badges.md)

<!-- Reference-Style link -->
[SonarCloud]: https://sonarcloud.io
[Issues]: https://docs.github.com/en/issues/tracking-your-work-with-issues/creating-an-issue
[Pull Requests]: https://docs.github.com/en/desktop/contributing-and-collaborating-using-github-desktop/working-with-your-remote-repository-on-github-or-github-enterprise/creating-an-issue-or-pull-request

# Unit Test (Java), Coverage and Analysis with SonarCloud

This action runs unit tests and optionally runs analysis, including coverage, using [SonarCloud](https://sonarcloud.io).  SonarCloud can be configured to comment on pull requests or stop failing workflows.

This Action supports Java.  Another for [JavaScript/TypeScript](https://github.com/bcgov-nr/action-test-and-analyse) is available.


# Usage

```yaml
- uses: bcgov-nr/action-test-and-analyse-java@v.0.1.0
  with:
    ### Required

    # Commands to run unit tests
    # Please configure your app to generate coverage (coverage/lcov.info)
    commands: |
      ./mvnw test

    # Project/app directory
    dir: backend

    ### Typical / recommended

    # Java package manager cache, defaults to maven
    java-cache: maven

    # Java distribution, defaults to temurin
    java-distribution: temurin

    # Java version, defaults to 17 (LTS)
    java-version: "17"

    # Sonar arguments
    # https://docs.sonarcloud.io/advanced-setup/analysis-parameters/
    sonar_args: |
        -Dsonar.exclusions=**/coverage/**
        -Dsonar.organization=bcgov-sonarcloud
        -Dsonar.projectKey=bcgov_${{ github.repository }}

    # Sonar project token
    # Available from sonarcloud.io or your organization administrator
    # BCGov i.e. https://github.com/BCDevOps/devops-requests/issues/new/choose
    # Provide an unpopulated token for pre-setup, section will be skipped
    sonar_project_token:
      description: ${{ secrets.SONAR_TOKEN }}

    ### Usually a bad idea / not recommended

    # Repository to clone and process
    # Useful for consuming outside reposities, defaults to the current one
    repository: <organization>/<repository>
```

# Example, Single Directory with SonarCloud Analysis

Run unit tests and provide results to SonarCloud.  This is a full workflow that runs on pull requests, merge to main and workflow_dispatch.  Use a GitHub Action and Dependabot secret for ${{ secrets.SONAR_TOKEN }}.

Create or modify a GitHub workflow, like below.  E.g. `./github/workflows/unit-tests.yml`

Note: SonarCloud allows pre-setup.  Configure without a token to skip steps until one is provided.

```yaml
name: Unit Tests and Analysis

on:
  pull_request:
  push:
    branches:
      - main
    paths-ignore:
      - ".github/**"
      - "**.md"
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  tests:
    name: Run Unit Tests and Analyse
    runs-on: ubuntu-22.04
    steps:
      - uses: bcgov-nr/action-test-and-analyse-java@v.0.1.0
        with:
          commands: |
            ./mvnw test
          dir: frontend
          java-cache: maven
          java-distribution: temurin
          java-version: "17"
          sonar_args: |
            -Dsonar.exclusions=**/coverage/**
            -Dsonar.organization=bcgov-nr
            -Dsonar.projectKey=bcgov-nr_action-test-and-analyse-java
          sonar_project_token: ${{ secrets.SONAR_TOKEN }}
```

# Example, Single Directory, Only Running Unit Tests (No SonarCloud)

Run unit tests, but not SonarCloud.

```yaml
jobs:
  tests:
    name: Run Unit Tests and Analyse
    runs-on: ubuntu-22.04
    steps:
      - uses: bcgov-nr/action-test-and-analyse-java@v.0.1.0
        with:
          commands: |
            ./mvnw test
          dir: frontend
          java-cache: maven
          java-distribution: temurin
          java-version: "17"
```

# Example, Matrix / Multiple Directories with Sonar Cloud

Unit test and analyze projects in multiple directories in parallel using matrices.  Please note how matrices required that secrets use [matrix.variable] syntax.

```yaml
jobs:
  tests:
    name: Unit Tests
    runs-on: ubuntu-22.04
    strategy:
      matrix:
        dir: [backend, frontend]
        include:
          - dir: backend
            token: SONAR_TOKEN_BACKEND
          - dir: frontend
            token: SONAR_TOKEN_FRONTEND
    steps:
      - uses: actions/checkout@v3
      - uses: bcgov-nr/action-test-and-analyse-java@v.0.1.0
        with:
          commands: |
            ./mvnw test
          dir: ${{ matrix.dir }}
          java-cache: maven
          java-distribution: temurin
          java-version: "17"
          sonar_args: |
            -Dsonar.exclusions=**/coverage/**
            -Dsonar.organization=bcgov-nr
            -Dsonar.projectKey=bcgov-nr_action-test-and-analyse-java_${{ matrix.dir }}
          sonar_project_token: ${{ secrets[matrix.token] }}
```

# Sonar Project Token

SonarCloud project tokens are free, available from [SonarCloud] or your organization's aministrators.

For BC Government projects, please create an [issue for our platform team](https://github.com/BCDevOps/devops-requests/issues/new/choose).

After sign up, a token should be available from your project on the [SonarCloud] site.  Multirepo projects (e.g. backend, frontend) will have multiple projects.  Click `Administration > Analysis Method > GitHub Actions (tutorial)` to find yours.

E.g. https://sonarcloud.io/project/configuration?id={\<PROJECT\>}&analysisMode=GitHubActions

# Feedback

Please contribute your ideas!  [Issues] and [pull requests] are appreciated.

<!-- # Acknowledgements

This Action is provided courtesty of the Forestry Suite of Applications, part of the Government of British Columbia. -->
