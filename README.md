# GHA Workflows

This repository hosts a set of custom reusable github actions workflows.

## Available workflows

- [package-test-uv.yml](#package-test-uvyml)
- [promote-staging-to-production.yml](#promote-staging-to-productionyml)
- [package-test-legacy.yml](#package-test-legacyyml)
- [package-test-coverage.yml](#package-test-coverageyml)
- [package-full-test.yml](#package-full-testyml)


## package-test-uv.yml

Test a Plone package. Test environment is bootstrapped using [uv](https://github.com/astral-sh/uv) and [buildout](https://github.com/buildout/buildout).

> **Note:** Supported Python versions are listed on [uv documentation](https://docs.astral.sh/uv/reference/policies/platforms/#python-support). If you need to use a deprecated Python 3 version, you can pin an older uv version using the `uv_version` input. For Python 2 support, use the `package-test-legacy.yml` workflow instead.


### Inputs

| Name                  | Type     | Required | Default              | Description                                                                 |
|-----------------------|----------|----------|----------------------|-----------------------------------------------------------------------------|
| buildout_command      | string   | No      | .venv/bin/buildout   | Command to run buildout                                                     |
| buildout_config_file  | string   | No      | buildout.cfg         | Buildout configuration file to use                                          |
| buildout_options      | string   | No       | (empty)              | Additional options to pass to buildout                                      |
| continue_on_error     | boolean  | No       | true                | Continue on error                                                           |
| python_version        | string   | No       | 3.13                 | Python version to use                                                       |
| requirements_file     | string   | No      | requirements.txt     | Requirements file to use for dependency installation                        |
| runner_label          | string   | No      | ubuntu-latest        | GitHub Actions runner label to use                                          |
| soffice               | boolean  | No       | false                | Launch soffice (LibreOffice in service mode)                                |
| system_dependencies   | string   | No       | (empty)              | System dependencies to install before running tests                         |
| test_command          | string   | No      | bin/test             | Command to run tests                                                        |
| uv_version            | string   | No       | 0.7.20               | Version of uv to use                                                        |

**Secrets**:

| Name                    | Required | Description                                                                 |
|-------------------------|----------|-----------------------------------------------------------------------------|
| mattermost_webhook_url  | No       | Mattermost webhook URL for notifications (optional)                         |


### Example of usage

#### Simple use-case (one version)

> **Tip:** If your repository follows the default values for all workflow inputs, you only need this single line to run the tests.

```yaml
test:
    uses: IMIO/gha-workflows/.github/workflows/package-test-uv.yml@v1
```

#### Multiple versions (github actions matrixes)

Thanks to github actions matrixes, you can launch tests on multiple plone and python versions.

In the below example, we run tests on 2 python versions and 2 plone versions (4 tests in total). You need a specific requirements and buildout config file for each plone version.

```yaml
test:
    uses: IMIO/gha-workflows/.github/workflows/package-test-uv.yml@v1
    strategy:
      matrix:
        python_version: ['3.10', '3.13']
        plone_version: ['6.0', '6.1']
    with:
      buildout_config_file: gha_${{ matrix.plone_version }}.cfg
      python_version: ${{ matrix.python_version }}
      requirements_file: requirements_${{ matrix.plone_version }}.txt
```

## promote-staging-to-production.yml

This workflow promotes a Docker image from staging to production and deploys it using Rundeck.

It tags the specified Docker image in the registry and notifies via Mattermost.

It also runs a Rundeck job to deploy the image to the specified node.

### Inputs

| Name                  | Type     | Required | Default      | Description                                              |
|-----------------------|----------|----------|--------------|----------------------------------------------------------|
| github_environment    | string   | No       | production   | GitHub environment to use for the job                    |
| image_name            | string   | Yes      | —            | Name of the Docker image                                 |
| image_tag_staging     | string   | Yes      | —            | Tag of the Docker image in staging                       |
| image_tag_production  | string   | Yes      | —            | Tag of the Docker image in production                    |
| rundeck_job_id        | string   | Yes      | —            | ID of the Rundeck job to run for deployment              |
| quick_release         | boolean  | No       | false        | Whether this is a quick release                          |
| runner_label          | string   | No       | gha-runners  | Label for the GitHub runner to use                       |
| schedule_time         | string   | No       | 03:00        | Time to schedule the deployment tomorrow (e.g., "03:00") |
| service_name          | string   | No       | —            | Name of the service being deployed                       |

> [!NOTE]
> If your RunDeck job needs to specify nodes (it's the case for a job where nodes are not checked by default), you can specify nodes by [setting a GitHub environment variable called NODE_NAME](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/managing-environments-for-deployment#environment-variables). Multiple nodes can be specified, separated by commas (e.g., node1.lan,node2.lan).

**Secrets**:

| Name                    | Required | Description                                 |
|-------------------------|----------|---------------------------------------------|
| mattermost_webhook_url  | No       | Webhook URL for Mattermost notifications    |
| registry_url            | Yes      | URL of the registry                         |
| registry_username       | Yes      | Username for the registry                   |
| registry_password       | Yes      | Password for the registry                   |
| rundeck_url             | Yes      | URL of the Rundeck server                   |
| rundeck_token           | Yes      | Token for the Rundeck server                |



### Example of usage

```yaml
jobs:
  promote-staging-to-production:
    uses: IMIO/gha-workflows/.github/workflows/promote-staging-to-production.yml@v1
    with:
      github_environment: production
      image_name: myapp
      image_tag_staging: staging
      image_tag_production: latest
      rundeck_job_id: 5b7c2640-1234-4b52-abcd-32745b326cd1
      runner_label: ubuntu-latest
      schedule_time: '02:00'
      service_name: myappservice
      quick_release: true
    secrets:
      registry_url: ${{ secrets.HARBOR_URL }}
      registry_username: ${{ secrets.HARBOR_USERNAME }}
      registry_password: ${{ secrets.HARBOR_PASSWORD }}
      rundeck_url: ${{ secrets.RUNDECK_URL }}
      rundeck_token: ${{ secrets.RUNDECK_TOKEN }}
```

## package-test-legacy.yml

Test a Plone package using legacy buildout and Python versions.

### Inputs

| Name                  | Type     | Required | Default              | Description                                                                 |
|-----------------------|----------|----------|----------------------|-----------------------------------------------------------------------------|
| buildout_command      | string   | No       | bin/buildout         | Command to run buildout                                                     |
| buildout_config_file  | string   | No       | buildout.cfg         | Buildout configuration file to use                                          |
| continue_on_error     | boolean  | No       | false                | Continue on error                                                           |
| matrix_experimental   | boolean  | No       | false                | Enable experimental matrix                                                  |
| plone_version         | string   | No       | 4.3                  | Plone version to use                                                        |
| python_version        | string   | No       | 2.7                  | Python version to use                                                       |
| requirements_file     | string   | No       | requirements.txt     | Requirements file to use for dependency installation                        |
| runner_label          | string   | No       | ubuntu-latest        | GitHub Actions runner label to use                                          |
| soffice               | boolean  | No       | false                | Launch soffice (LibreOffice in service mode)                                |
| test_command          | string   | No       | bin/test             | Command to run tests                                                        |

**Secrets**:

| Name                    | Required | Description                                                                 |
|-------------------------|----------|-----------------------------------------------------------------------------|
| mattermost_webhook_url  | No       | Mattermost webhook URL for notifications (optional)                         |

### Example of usage

```yaml
test:
    uses: IMIO/gha-workflows/.github/workflows/package-test-legacy.yml@v1
    with:
      buildout_config_file: buildout.cfg
      python_version: 2.7
      requirements_file: requirements.txt
```

## package-test-coverage.yml

Test a Plone package and generate a coverage report. Test environment is bootstrapped using [uv](https://github.com/astral-sh/uv) and [buildout](https://github.com/buildout/buildout). Optionally uploads the coverage report to Coveralls.

### Inputs

| Name                  | Type     | Required | Default                                                      | Description                                                      |
|-----------------------|----------|----------|--------------------------------------------------------------|------------------------------------------------------------------|
| buildout_command      | string   | No       | .venv/bin/buildout                                           | Command to run buildout                                           |
| buildout_config_file  | string   | No       | buildout.cfg                                                 | Buildout configuration file to use                                |
| buildout_options      | string   | No       | (empty)                                                      | Additional options to pass to buildout                            |
| continue_on_error     | boolean  | No       | false                                                        | Continue on error                                                 |
| plone_version         | string   | No       | 6.1                                                          | Plone version to use                                              |
| requirements_file     | string   | No       | requirements.txt                                              | Requirements file to use for dependency installation              |
| runner_label          | string   | No       | ubuntu-latest                                                 | GitHub Actions runner label to use                                |
| soffice               | boolean  | No       | false                                                        | Launch soffice (LibreOffice in service mode)                      |
| test_command          | string   | No       | uvx coverage run bin/test -t !robot >> $GITHUB_STEP_SUMMARY   | Command to run tests with coverage                                |
| upload_to_coveralls   | boolean  | No       | false                                                        | Upload coverage report to Coveralls                               |

**Secrets**:

| Name                    | Required | Description                                                                 |
|-------------------------|----------|-----------------------------------------------------------------------------|
| mattermost_webhook_url  | No       | Mattermost webhook URL for notifications (optional)                         |

### Example of usage

```yaml
test:
    uses: IMIO/gha-workflows/.github/workflows/package-test-coverage.yml@v1
    with:
      upload_to_coveralls: true
```

## package-full-test.yml

Full test pipeline for a Plone package, combining three parallel jobs in a single reusable workflow:

1. **code-analysis** — runs a code analysis check (black by default) via `IMIO/gha/code-analysis-notify@v6`
2. **test** — runs [package-test-uv.yml](#package-test-uvyml) on each Python version of the `python_versions` matrix
3. **coverage** — runs [package-test-coverage.yml](#package-test-coverageyml)

> **Note:** Since the Python matrix lives inside this workflow, pass the versions as a JSON array string via the `python_versions` input (e.g. `'["3.12", "3.13"]'`) instead of a `strategy.matrix` on the calling job.

### Inputs

| Name                    | Type     | Required | Default                                                          | Description                                                      |
|-------------------------|----------|----------|------------------------------------------------------------------|------------------------------------------------------------------|
| base_dir                | string   | No       | src                                                              | Base directory for the code analysis check                       |
| buildout_command        | string   | No       | .venv/bin/buildout                                               | Command to run buildout                                          |
| buildout_config_file    | string   | No       | buildout.cfg                                                     | Buildout configuration file to use                               |
| buildout_options        | string   | No       | (empty)                                                          | Additional options to pass to buildout                           |
| check                   | string   | No       | black                                                            | Code analysis check to run                                       |
| check_path              | string   | Yes      | —                                                                | Package path(s) to run the code analysis check on                |
| continue_on_error       | boolean  | No       | true                                                             | Continue on error (test job)                                     |
| coverage_python_version | string   | No       | 3.13                                                             | Python version for the coverage job                              |
| coverage_test_command   | string   | No       | .venv/bin/coverage run bin/test -t !robot >> $GITHUB_STEP_SUMMARY | Test command for the coverage job                                |
| python_versions         | string   | No       | ["3.13"]                                                         | Python versions for the test matrix, as a JSON array             |
| requirements_file       | string   | No       | requirements.txt                                                 | Requirements file to use for dependency installation             |
| runner_label            | string   | No       | ubuntu-latest                                                    | GitHub Actions runner label to use                               |
| soffice                 | boolean  | No       | false                                                            | Launch soffice (LibreOffice in service mode)                     |
| system_dependencies     | string   | No       | (empty)                                                          | System dependencies to install before running tests              |
| test_command            | string   | No       | bin/test                                                         | Command to run tests                                             |
| upload_to_coveralls     | boolean  | No       | false                                                            | Upload coverage report to Coveralls                              |
| uv_version              | string   | No       | 0.7.20                                                           | Version of uv to use                                             |

**Secrets**:

| Name                    | Required | Description                                                                 |
|-------------------------|----------|-----------------------------------------------------------------------------|
| mattermost_webhook_url  | No       | Mattermost webhook URL for notifications (optional)                         |

### Example of usage

```yaml
name: Tests

on:
  push:
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  full-test:
    uses: IMIO/gha-workflows/.github/workflows/package-full-test.yml@v1
    secrets:
      mattermost_webhook_url: ${{ secrets.SMARTWEB_MATTERMOST_WEBHOOK_URL }}
    with:
      check_path: |
        imio/events/core
      buildout_config_file: test_plone6.cfg
      python_versions: '["3.12", "3.13"]'
      runner_label: gha-runners-smartweb
      test_command: TZ=UTC bin/test
      upload_to_coveralls: true
```