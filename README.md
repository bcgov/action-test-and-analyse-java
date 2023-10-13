**BREAKING CHANGES in v2.0:**
* **java_version is now required (previously defaulted to 17)**
* **sonar_project_token has been renamed sonar_token**

<!-- Badges -->
[![Issues](https://img.shields.io/github/issues/bcgov-nr/action-test-and-analyse-java)](/../../issues)
[![Pull Requests](https://img.shields.io/github/issues-pr/bcgov-nr/action-test-and-analyse-java)](/../../pulls)
[![MIT License](https://img.shields.io/github/license/bcgov-nr/action-test-and-analyse-java.svg)](/LICENSE)
[![Lifecycle](https://img.shields.io/badge/Lifecycle-Experimental-339999)](https://github.com/bcgov/repomountie/blob/master/doc/lifecycle-badges.md)

<!-- Reference-Style link -->
[SonarCloud]: https://sonarcloud.io
[Issues]: https://docs.github.com/en/issues/tracking-your-work-with-issues/creating-an-issue
[Pull Requests]: https://docs.github.com/en/desktop/contributing-and-collaborating-using-github-desktop/working-with-your-remote-repository-on-github-or-github-enterprise/creating-an-issue-or-pull-request

# Test and Analyze with Triggers and SonarCloud

This action runs tests, dependent on triggers, optionally sending results and coverage to [SonarCloud](https://sonarcloud.io).  Test and SonarCloud can be configured to comment on pull requests or stop failing workflows.

Conditional triggers are used to determine whether tests need to be run.  If triggers are matched, then the appropriate code has changed and should be tested.  Tests always run if no triggers are provided.  Untriggered runs do little other than report a success.

Only Java is supported by this action.  Please see our [JavaScript action](https://github.com/bcgov-nr/action-test-and-analyse) or upcoming Python action as required.

# Usage

```yaml
- uses: bcgov-nr/action-test-and-analyse-java@main
  with:
    ### Required

    # Commands to run unit tests
    # Please configure your app to generate coverage (coverage/lcov.info)
    commands: |
      mvn -B verify -P all-tests checkstyle:checkstyle -Dcheckstyle.skip=false

    # Project/app directory
    dir: backend

    # Java version, previously defaulted to 17 (LTS)
    java-version: "17"

    ### Typical / recommended

    # Java package manager cache, defaults to maven
    java-cache: maven

    # Java distribution, defaults to temurin
    java-distribution: temurin

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
    sonar_token:
      description: ${{ secrets.SONAR_TOKEN }}

    # Bash array to diff for build triggering
    # Optional, defaults to nothing, which forces a build
    triggers: ('frontend/')

    ### Usually a bad idea / not recommended

    # Overrides the default branch to diff against
    # Defaults to the default branch, usually `main`
    diff_branch: ${{ github.event.repository.default_branch }}

    # Repository to clone and process
    # Useful for consuming other repos, like in testing
    # Defaults to the current one
    repository: ${{ github.repository }}

    # Branch to clone and process
    # Useful for consuming non-default branches, like in testing
    # Defants to empty, cloning the default branch
    branch: ""

    # Bash array of events for limiting triggers, otherwise trigger automatically
    # E.g. ("pull_request" "push" "workflow_dispatch")
    # Defaults to only using triggers with pull requests
    triggers_event: "('pull_request')"
```

# Example, Single Directory with SonarCloud Analysis

Run tests and provide results to SonarCloud.  This is a full workflow that runs on pull requests, merge to main and workflow_dispatch.  Use a GitHub Action secret to provide ${{ secrets.SONAR_TOKEN }}.

The specified triggers will be used to decide whether this job runs tests and analysis or just exists successfully.

Create or modify a GitHub workflow, like below.  E.g. `./github/workflows/tests.yml`

Note: Provde an unpopulated SONAR_TOKEN until one is provisioned.  SonarCloud will only run once populated, allowing for pre-setup.

```yaml
name: Analysis

on:
  pull_request:
  push:
    branches: [main]
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  tests:
    name: Unit Tests
    runs-on: ubuntu-22.04
    steps:
      - uses: bcgov-nr/action-test-and-analyse-java@main
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
          sonar_token: ${{ secrets.SONAR_TOKEN }}
          triggers: ('frontend/' 'charts/frontend')
```

# Example, Only Running Tests (No SonarCloud), No Triggers

No triggers are provided so tests will always run.  SonarCloud is skipped.

```yaml
jobs:
  tests:
    name: Unit Tests
    runs-on: ubuntu-22.04
    steps:
      - uses: bcgov-nr/action-test-and-analyse-java@main
        with:
          commands: |
            ./mvnw test
          dir: frontend
          java-cache: maven
          java-distribution: temurin
          java-version: "17"
```

# Example, Matrix / Multiple Directories with Sonar Cloud and Triggers

Test projects in multiple directories in parallel.  This time `repository` and `branch` are provided.  Please note how secrets must be passed in to composite Actions using the secrets[matrix.variable] syntax.

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
            triggers: ('frontend/' 'charts/frontend')
          - dir: frontend
            token: SONAR_TOKEN_FRONTEND
            triggers: ('backend/' 'charts/backend')
    steps:
      - uses: actions/checkout@v4
      - uses: bcgov-nr/action-test-and-analyse-java@main
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
          sonar_token: ${{ secrets[matrix.token] }}
          triggers: ${{ matrix.triggers }}
          repository: bcgov/quickstart-openshift
          branch: main
```

# Sonar Project Token

SonarCloud project tokens are free, available from [SonarCloud] or your organization's aministrators.

For BC Government projects, please create an [issue for our platform team](https://github.com/BCDevOps/devops-requests/issues/new/choose).

After sign up, a token should be available from your project on the [SonarCloud] site.  Multirepo projects (e.g. backend, frontend) will have multiple projects.  Click `Administration > Analysis Method > GitHub Actions (tutorial)` to find yours.

E.g. https://sonarcloud.io/project/configuration?id={<PROJECT>}&analysisMode=GitHubActions

# Triggers and Triggers_Event

Triggers are used to limit test running to only appropriate files are changed.  This is generally not desirable outside of pull requests, so `triggers_event` defaults to `("pull_request")`.  Override this behaviour by specifying a bash array using any of the many, many [event types GitHub provides](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#push), e.g. `("branch_protection_rule" "workflow_dispatch" "push")`.

# Feedback

Please contribute your ideas!  [Issues] and [pull requests] are appreciated.

<!-- # Acknowledgements

This Action is provided courtesty of the Forestry Suite of Applications, part of the Government of British Columbia. -->
