# TeamCity Google Cloud Deployment Manager template
[![JetBrains incubator project](http://jb.gg/badges/incubator.svg)](https://confluence.jetbrains.com/display/ALL/JetBrains+on+GitHub)

Allows to create TeamCity deployment in Google Cloud by using [gcloud tool](https://cloud.google.com/sdk/gcloud/) locally or in the [Google Cloud console](https://console.cloud.google.com/).

The template allows deploying a TeamCity [server](https://hub.docker.com/r/jetbrains/teamcity-server/) and [agent](https://hub.docker.com/r/jetbrains/teamcity-agent/) in Google Cloud Platform. It creates a MySQL database, a Google Compute Engine (GCE) instance with CoreOS and starts TeamCity in a docker container.

## Pre-requisites

### Service account

Please ensure in the [IAM console](https://console.cloud.google.com/iam-admin/iam/project) that [Deployment Manager service account](https://cloud.google.com/deployment-manager/docs/access-control#access_control_for_deployment_manager) `[project_number]@cloudservices.gserviceaccount.com` has `Project Owner` role.

You could also do that via following command:

```sh
> gcloud projects add-iam-policy-binding $(gcloud config get-value project) --member serviceAccount:$(gcloud projects describe $(gcloud config get-value project) --format="value(projectNumber)")@cloudservices.gserviceaccount.com --role roles/owner
```

### Google Cloud APIs

Ensure that you have enabled following Google Cloud APIs in your project:
* [Cloud Deployment Manager V2 API](https://console.cloud.google.com/apis/api/deploymentmanager.googleapis.com/overview)
* [Google Cloud Resource Manager API](https://console.cloud.google.com/apis/api/cloudresourcemanager.googleapis.com/overview)
* [Google Identity and Access Management (IAM) API](https://console.cloud.google.com/apis/api/iam.googleapis.com/overview)
* [Cloud SQL Administration API](https://console.developers.google.com/apis/api/sqladmin.googleapis.com/overview)

You could do it via the following command:

```sh
> gcloud services enable deploymentmanager.googleapis.com sqladmin.googleapis.com iam.googleapis.com cloudresourcemanager.googleapis.com runtimeconfig.googleapis.com
```

## Deployment

Deploy TeamCity as a template by specifying properties:

```sh
> gcloud deployment-manager deployments create teamcity --template https://raw.githubusercontent.com/dtretyakov/teamcity-google-template/master/teamcity.jinja --properties zone:us-central1-a,version:2017.2.2
```

**Note**: Deployment will take several minutes, on completion you could navigate to the `teamcityUrl` output value to see TeamCity UI.

### Properties

You could specify the following properties for deployment:

* `zone` - [zone](https://cloud.google.com/compute/docs/regions-zones/) in which this deployment will run.
* `version` - [TeamCity version](https://www.jetbrains.com/teamcity/download/) to deploy.
* `installationSize` - the size of installation: small/medium/large.
* `serviceAccount` - the e-mail of service account specified for TeamCity GCE instance.

#### Installation Size

List of pre-configured installation types:

| Installation Size | Typical Usage             | vCPU | RAM  | VM Data Disk | Database         |
| ----------------- | ------------------------- | ---- | ---- | ------------ | ---------------- |
| Small             | 3 users, 100 builds/day   | 1    | 3 GB | 32 GB HDD    | db-n1-standard-1 |
| Medium            | 5 users, 300 builds/day   | 2    | 4 GB | 64 GB SSD    | db-n1-standard-1 |
| Large             | 20 users, 1000 builds/day | 4    | 8 GB | 128 GB SSD   | db-n1-standard-2 |

**Note**: Pricing for [Google Compute Engine](https://cloud.google.com/compute/pricing#custommachinetypepricing) and [MySQL database](https://cloud.google.com/sql/docs/mysql/pricing).

### Further Steps

**Note**: TeamCity server exposes HTTP endpoint, so please make sure to enable HTTPS endpoint for GCE instance for production usage.

## TeamCity Update

To change the TeamCity version you could start deployment script with a required version number and then execute [Reset](https://cloud.google.com/compute/docs/instances/restarting-an-instance) action on the teamcity virtual machine:
 
```sh
> gcloud deployment-manager deployments update teamcity --template https://raw.githubusercontent.com/dtretyakov/teamcity-google-template/master/teamcity.jinja --properties zone:us-central1-a,version:2017.2.2
```

**Note**: you could not change the `zone` parameter while update.

## Under the Hood

The template allocates while deployment following resource:
* Service account with `Project Viewer`, `Cloud SQL Client`, `Compute Instance Admin`, `Storage Object Admin` and `Service Account Token Creator` roles.
* Network, firewall rules and static IP address.
* MySQL database and user.
* GCE instance with data disk powered by CoreOS and assigned service account.

### GCE Instance

After deployment you will be able to connect to the GCE instance via SSH. In CoreOS TeamCity works as the following systemd service:

* `teamcity-server.service` - launches TeamCity server.
* `teamcity-agent.service` - launches TeamCity agent.

### Installed Plugins

The template installs the following Google Cloud Platform integrations in TeamCity:

* [Google Cloud Agents](https://plugins.jetbrains.com/plugin/9704-google-cloud-agents) - allows to scale the pool of TeamCity build agents by leveraging GCE.
* [Google Artifacts Storage](https://plugins.jetbrains.com/plugin/9634-google-artifact-storage) - allows to store build artifacts in Google Storage Blobs.
