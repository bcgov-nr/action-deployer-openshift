<!-- Badges -->
[![Issues](https://img.shields.io/github/issues/bcgov-nr/action-deployer-openshift)](/../../issues)
[![Pull Requests](https://img.shields.io/github/issues-pr/bcgov-nr/action-deployer-openshift)](/../../pulls)
[![Apache 2.0 License](https://img.shields.io/github/license/bcgov-nr/action-deployer-openshift.svg)](/LICENSE)
[![Lifecycle](https://img.shields.io/badge/Lifecycle-Experimental-339999)](https://github.com/bcgov/repomountie/blob/master/doc/lifecycle-badges.md)

<!-- Reference-Style link -->
[Issues]: https://docs.github.com/en/issues/tracking-your-work-with-issues/creating-an-issue
[Pull Requests]: https://docs.github.com/en/desktop/contributing-and-collaborating-using-github-desktop/working-with-your-remote-repository-on-github-or-github-enterprise/creating-an-issue-or-pull-request

# OpenShift Deployer with Route Verification or Penetration Testing

GitHub Action. Deploy to OpenShift using templates. Runs route verification or penetration tests.  Most of the heavy lifting here is done in template configuration.

Testing has only been done with public containers on ghcr.io (GitHub Container Registry) so far.

# Usage

```yaml
- uses: bcgov-nr/action-deployer-openshift@main
  with:
    ### Required

    # OpenShift template file
    file: frontend/openshift.deploy.yml

    # OpenShift project/namespace
    oc_namespace: abc123-dev

    # OpenShift server
    oc_server: https://api.silver.devops.gov.bc.ca:6443
    
    # OpenShift token
    # Usually available as a secret in your project/namespace
    oc_token: ${{ secrets.OC_TOKEN }}
    
    # Overwrite objects using `oc apply` or only create with `oc create`
    # Expected errors from `oc create` are handled with `set +o pipefail`
    overwrite: true


    ### Typical / recommended

    # Template parameters/variables to pass
    parameters: -p PR_NO=${{ github.event.number }}

    # Run a ZAProxy penetration test against any routes? [true/false]
    penetration_test: false

    # Allow ZAProxy alerts to fail the workflow? [true/false]
    penetration_test_fail: false


    ### Usually a bad idea / not recommended

    # Repository to clone and process
    # Useful for consuming other repos, defaults to the current one
    repository: ${{ github.repository }}
```

# Example, Single Template

Deploy a single template.  Multiple GitHub secrets are used.

```yaml
deploys:
  name: Deploys
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v3
    - name: Deploys
      uses: bcgov-nr/action-deployer-openshift.yml@main
      with:
        file: frontend/openshift.deploy.yml
        oc_namespace: ${{ secrets.OC_NAMESPACE }}
        oc_server: ${{ secrets.OC_SERVER }}
        oc_token: ${{ secrets.OC_TOKEN }}
        overwrite: yes
        parameters:
          -p MIN_REPLICAS=1 -p MAX_REPLICAS=2
          -p PR_NUMBER=${{ github.event.number }}
```

# Example, Matrix / Multiple Templates

Deploy multiple templates in parallel.  This time penetration tests are enabled.  Runs on pull requests (PRs).

```yaml
deploys:
name: Deploys
runs-on: ubuntu-latest
  strategy:
    matrix:
    name: [backend, database, frontend, init]
    include:
      - name: backend
        file: backend/openshift.deploy.yml
        overwrite: true
        parameters: -p MIN_REPLICAS=1 -p MAX_REPLICAS=2
      - name: database
        overwrite: false
        file: database/openshift.deploy.yml
      - name: frontend
        overwrite: true
        file: frontend/openshift.deploy.yml
        parameters: -p MIN_REPLICAS=1 -p MAX_REPLICAS=2
      - name: init
        overwrite: false
        file: common/openshift.init.yml
steps:
  - uses: actions/checkout@v3
  - name: Deploys
    uses: bcgov-nr/action-deployer-openshift.yml@main
    with:
      file: ${{ matrix.file }}
      oc_namespace: ${{ secrets.OC_NAMESPACE }}
      oc_server: ${{ secrets.OC_SERVER }}
      oc_token: ${{ secrets.OC_TOKEN }}
      overwrite: ${{ matrix.overwrite }}
      parameters:
        -p COMMON_TEMPLATE_VAR=whatever-${{ github.event.number }}
        ${{ matrix.parameters }}
      penetration_test: true
```

# Route Verification vs Penetration Testing

Deployment templates are parsed for a route.  If found, those routes are verified with a curl command for status code 200 (success).  This ensures that applications are accessible from outside their OpenShift namespace/project.

Provide `penetration_test: true` to instead run a penetration test using [OWASP ZAP (Zed Attack Proxy)](https://github.com/zaproxy/action-full-scan) against that route. `penetration_test_fail: false` can be used to fail pipelines where problems are found.

# Troubleshooting

## Dependabot Pull Requests Failing

Pull requests created by Dependabot require their own secrets.  See `GitHub Repo > Settings > Secrets > Dependabot`.

# Feedback

Please contribute your ideas!  [Issues] and [pull requests] are appreciated.

Idea: Can anyone test with Kubernetes, which OpenShift is based on?

# Acknowledgements

This Action is provided courtesty of the Forestry Suite of Applications, part of the Government of British Columbia.
