# Octue reusable workflows
Octue's reusable GitHub Actions workflows for:
- Building, deploying, and shelling into dockerised Django servers and Octue services on Google Cloud Run
- Code and release quality control

## Deployment
All of these reusable workflows assume the relevant Google Cloud infrastructure has already been set up.

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

### Deploying a Cloud Run Octue service
This workflow builds and deploys a revision of a dockerised Octue service to Google Cloud Run, storing the image in 
Google Cloud Artifact Registry. A Google Pub/Sub push subscription is created for the revision and, if given, it's 
registered with a service registry. The Dockerfile can be located locally or at a URL.

**Example usage**

Add the following job to a workflow:

```shell
...

jobs:
  ...
  
  deploy:
    if: "!contains(github.event.head_commit.message, 'skipci')"
    uses: octue/workflows/.github/workflows/deploy-cloud-run-service.yml@main
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
      cloud_run_flags: '--ingress=all --allow-unauthenticated --service-account=my-service-service-account@my-project.iam.gserviceaccount.com --max-instances=10 --memory=2048Mi'
      service_registry_endpoint: https://example.com/integrations/octue/services

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

  ...      
```

## Code and release quality control

### Checking release semantic version

### Automatically updating a pull request description with release notes
