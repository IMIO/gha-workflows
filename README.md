# GHA Workflows

This repository hosts a set of custom reusable github actions workflows.

## helm-release.yml

Release a Helm chart to a GitHub Pages Helm repository and send notifications via Mattermost.

### Inputs

| Name              | Type    | Required | Default    | Description                                              |
|-------------------|---------|----------|-------------|----------------------------------------------------------|
| helm_version      | string  | No       | 3.18.4      | Helm version to use                                      |
| helm_dependencies | string  | No       | —           | Helm dependencies                                        |
| index_dir         | string  | No       | .           | Index directory                                          |
| charts_dir        | string  | No       | .           | Charts directory                                         |
| target_dir        | string  | No       | test        | Target directory to release                              |
| app_id            | string  | Yes      | —           | GitHub App ID                                            |
| private_key       | string  | Yes      | —           | GitHub App private key                                   |
| runner_label      | string  | No       | ubuntu-latest | GitHub Actions runner label to use                    |

**Secrets**:

| Name                    | Required | Description                                 |
|-------------------------|----------|---------------------------------------------|
| mattermost_webhook_url  | No       | Webhook URL for Mattermost notifications    |

### Example of usage

```yaml
jobs:
  release-helm-chart:
    uses: IMIO/gha-workflows/.github/workflows/helm-release.yml@main
    with:
      helm_version: '3.18.4'
      index_dir: '.'
      charts_dir: '.'
      target_dir: 'test'
      app_id: ${{ secrets.APP_ID }}
      private_key: ${{ secrets.PRIVATE_KEY }}
    secrets:
      mattermost_webhook_url: ${{ secrets.MATTERMOST_WEBHOOK_URL }}
```

## helm-test.yml

Lint and test a Helm chart and send notifications via Mattermost.

### Inputs

| Name           | Type    | Required | Default    | Description                                              |
|----------------|---------|----------|-------------|----------------------------------------------------------|
| python_version | string  | No       | 3.10        | Python version to use                                    |
| helm_version   | string  | No       | v3.18.4     | Helm version to use                                      |
| helm_release   | string  | No       | test         | Helm release name                                        |
| helm_namespace | string  | No       | test         | Helm namespace name                                      |
| runner_label   | string  | No       | ubuntu-latest | GitHub Actions runner label to use                    |

**Secrets**:

| Name                    | Required | Description                                 |
|-------------------------|----------|---------------------------------------------|
| mattermost_webhook_url  | No       | Webhook URL for Mattermost notifications    |

### Example of usage

```yaml
jobs:
  helm-test:
    uses: IMIO/gha-workflows/.github/workflows/helm-test.yml@main
    with:
      python_version: '3.10'
      helm_version: 'v3.18.4'
      helm_release: 'test'
      helm_namespace: 'test'
    secrets:
      mattermost_webhook_url: ${{ secrets.MATTERMOST_WEBHOOK_URL }}
```

## package-test-uv.yml

Test a Plone package. Test environment is bootstrapped using uv and buildout.

### Inputs

| Name                  | Type     | Required | Default              | Description                                                                 |
|-----------------------|----------|----------|----------------------|-----------------------------------------------------------------------------|
| buildout_command      | string   | No      | .venv/bin/buildout   | Command to run buildout                                                     |
| buildout_config_file  | string   | No       | buildout.cfg         | Buildout configuration file to use                                          |
| buildout_options      | string   | No       | (empty)              | Additional options to pass to buildout                                      |
| continue_on_error     | boolean  | No       | false                | Continue on error                                                           |
| matrix_experimental   | boolean  | No       | false                | Enable experimental matrix                                                  |
| plone_version         | string   | No       | 6.1                  | Plone version to use                                                        |
| python_version        | string   | No       | 3.13                 | Python version to use                                                       |
| requirements_file     | string   | No       | requirements.txt     | Requirements file to use for dependency installation                        |
| runner_label          | string   | No       | (none)               | GitHub Actions runner label to use                                          |
| soffice               | boolean  | No       | false                 | Launch soffice (LibreOffice in service mode)                                |
| test_command          | string   | No       | bin/test             | Command to run tests                                                        |

**Secrets**:

| Name                    | Required | Description                                                                 |
|-------------------------|----------|-----------------------------------------------------------------------------|
| mattermost_webhook_url  | No       | Mattermost webhook URL for notifications (optional)                         |


### Example of usage

#### One python version, one plone version

```yaml
test:
    uses: IMIO/gha-workflows/.github/workflows/package-test-uv.yml@main
```

#### Multiple versions (github actions matrixes)

Thanks to github actions matrixes, you can launch tests on multiple plone and python versions.

In the below example, we run tests on 2 python versions and 2 plone versions (4 tests in total). You need a specific requirements and buildout config file for each plone version.

```yaml
test:
    uses: IMIO/gha-workflows/.github/workflows/package-test-uv.yml@main
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
    uses: IMIO/gha-workflows/.github/workflows/promote-staging-to-production.yml@main
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

