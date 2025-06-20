# GHA Workflows

This repository hosts a set of custom reusable github actions workflows.

## package-test-uv.yml

Test a Plone package. Test environment is bootstrapped using uv and buildout.

### Inputs

| Name                  | Type     | Required | Default              | Description                                                                 |
|-----------------------|----------|----------|----------------------|-----------------------------------------------------------------------------|
| buildout_command      | string   | Yes      | .venv/bin/buildout   | Command to run buildout                                                     |
| buildout_config_file  | string   | Yes      | buildout.cfg         | Buildout configuration file to use                                          |
| buildout_options      | string   | No       | (empty)              | Additional options to pass to buildout                                      |
| continue_on_error     | boolean  | No       | false                | Continue on error                                                           |
| matrix_experimental   | boolean  | No       | false                | Enable experimental matrix                                                  |
| plone_version         | string   | No       | 6.1                  | Plone version to use                                                        |
| python_version        | string   | Yes       | 3.13                 | Python version to use                                                       |
| requirements_file     | string   | Yes      | requirements.txt     | Requirements file to use for dependency installation                        |
| runner_label          | string   | Yes      | (none)               | GitHub Actions runner label to use                                          |
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
