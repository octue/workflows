# Octue reusable workflows
Octue's reusable GitHub Actions workflows for:
- Building, deploying, and shelling into dockerised Django servers and Octue services on Google Cloud Run
- Code and release quality control

## Deployment
All of these reusable workflows assume the relevant Google Cloud infrastructure has already been set up.

### Deploying a Kubernetes/Kueue Octue Twined service revision
This workflow builds and deploys a revision of a dockerised Octue Twined service revision to Kubernetes/Kueue, storing 
the image in Google Cloud Artifact Registry. If a service registry is specified, the service revision is registered with
it. Unless another dockerfile is provided locally or from a URL, the default data service `Dockerfile` is used (based 
on python3.11).

**Example usage**

Add the following job to a workflow:

```shell
...

jobs:
  ...
  
  deploy:
    if: "!contains(github.event.head_commit.message, 'skipci')"
    uses: octue/workflows/.github/workflows/build-twined-service.yml@main
    permissions:
      id-token: write
      contents: read
    with:
      gcp_project_name: my-project
      gcp_project_number: 1234
      gcp_region: europe-west3
      gcp_resource_affix: my-resource-affix
      gcp_service_name: my-service-name
      local_dockerfile: path/to/Dockerfile
      service_registry_endpoint: https://example.com/integrations/octue/services

  ...
```

### Deploying a Cloud Run django server
This workflow builds and deploys a dockerised django server to Google Cloud Run, storing the image in Google Cloud 
Artifact Registry.

**Example usage**

Add the following job to a workflow:

```shell
...

jobs:
  ...
  
  deploy:
    if: "!contains(github.event.head_commit.message, 'skipci')"
    uses: octue/workflows/.github/workflows/deploy-cloud-run-django-server.yml@main
    permissions:
      id-token: write
      contents: read
    with:
      gcp_project_name: my-project
      gcp_project_number: 1234
      gcp_region: europe-west3
      gcp_resource_affix: my-resource-affix
      gcp_environment: main
      server_name: server
      dockerfile: path/to/Dockerfile  # Defaults to Dockerfile in repository root.
      cloud_run_base_url: some-prefix.something.run.app
      cloud_run_flags: "--ingress=all --allow-unauthenticated --service-account=my-server-service-account@my-project.iam.gserviceaccount.com --max-instances=10 --memory=2048Mi"
      django_settings_module: server.settings
      build_args: |
        "SOME_VARIABLE=foo"
        "ANOTHER_VARIABLE=bar"
    secrets:
      build_secrets: |
        "SOME_SECRET=${{ secrets.SOME_GITHUB_SECRET }}"
  
  ...
```


### Shelling into a Cloud Run django server
This workflow spins up a container from the provided image, populates its environment, connects the database to it, and
sets up a `tmate` session to allow the caller to ssh into it. We recommend using this workflow with a 
`workflow_dispatch` trigger.

**Example usage**

Add the following to a workflow:

```shell
on:
  workflow_dispatch:
    inputs:
      image:
        description: "The URI of the image to run as a container"
        type: string

jobs:
  ...
  
  shell:
    uses: octue/workflows/.github/workflows/shell-django-server.yml@main
    permissions:
      contents: read
      id-token: write
    with:
      image: ${{ github.event.inputs.image }}
      gcp_project_name: my-project
      gcp_project_number: 1234
      gcp_region: europe-west3
      gcp_resource_affix: my-resource-affix
      server_name: server
      settings_module: server.settings
      database_name: primary

  ...      
```

## Code and release quality control

### Checking release semantic version
This workflow checks if the semantic version in the repository's `setup.py`, `pyproject.toml`, or `package.json` file
is the version expected according to the most recent version tag and the 
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) since then. We recommend using this workflow on
release branches.

**Example usage**

Add the following to a workflow:

```shell
on:
  pull_request:
    branches:
      - main
      
jobs:
  ...
  
  check-semantic-version:
    uses: octue/workflows/.github/workflows/check-semantic-version.yml@main
    with:
      path: pyproject.toml
      breaking_change_indicated_by: minor
  
  ...
```      

### Automatically updating a pull request description with release notes
This workflow takes the pull request's commit messages in the form of 
[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) and uses them to generate the basis for a 
description of the pull request that can also be used as release notes. This workflow must be used with a `pull_request`
trigger.

**Example usage**

Add the following to a workflow:

```shell

on: [pull_request]

jobs:
  ...
  
  generate-pull-request-description:
    uses: octue/workflows/.github/workflows/generate-pull-request-description.yml@main
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
    permissions:
      contents: read
      pull-requests: write

  ...     
```
