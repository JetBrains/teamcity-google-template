# TeamCity Google Cloud Deployment Manager template
[![JetBrains incubator project](http://jb.gg/badges/incubator.svg)](https://confluence.jetbrains.com/display/ALL/JetBrains+on+GitHub)

Allows creating a TeamCity deployment in Google Cloud by using the [gcloud tool](https://cloud.google.com/sdk/gcloud/) locally or in the [Google Cloud console](https://console.cloud.google.com/).

The template allows deploying a TeamCity [server](https://hub.docker.com/r/jetbrains/teamcity-server/) and [agent](https://hub.docker.com/r/jetbrains/teamcity-agent/) in Google Cloud Platform. It creates a MySQL database, a Google Compute Engine (GCE) instance with CoreOS and starts TeamCity in a Docker container.

## Pre-requisites

### Service account

Please use the [IAM console](https://console.cloud.google.com/iam-admin/iam/project) to ensure that the [Deployment Manager service account](https://cloud.google.com/deployment-manager/docs/access-control#access_control_for_deployment_manager) `[project_number]@cloudservices.gserviceaccount.com` has the `Project Owner` role.

To do it, use the following command:

```sh
> gcloud projects add-iam-policy-binding $(gcloud config get-value project) --member serviceAccount:$(gcloud projects describe $(gcloud config get-value project) --format="value(projectNumber)")@cloudservices.gserviceaccount.com --role roles/owner
```

### Google Cloud APIs

Ensure that you have enabled the following Google Cloud APIs in your project:
* [Cloud Deployment Manager V2 API](https://console.cloud.google.com/apis/api/deploymentmanager.googleapis.com/overview)
* [Google Cloud Resource Manager API](https://console.cloud.google.com/apis/api/cloudresourcemanager.googleapis.com/overview)
* [Google Identity and Access Management (IAM) API](https://console.cloud.google.com/apis/api/iam.googleapis.com/overview)
* [Cloud SQL Administration API](https://console.developers.google.com/apis/api/sqladmin.googleapis.com/overview)

To do it, use the following command:

```sh
> gcloud services enable deploymentmanager.googleapis.com sqladmin.googleapis.com iam.googleapis.com cloudresourcemanager.googleapis.com runtimeconfig.googleapis.com
```

## Deployment

Deploy TeamCity as a template by specifying the following properties:

```sh
> gcloud deployment-manager deployments create teamcity --template https://raw.githubusercontent.com/JetBrains/teamcity-google-template/master/teamcity.jinja --properties zone:<zone>
```

**Note**: Deployment will take several minutes, on completion you will be able to navigate to the `teamcityUrl` output value to see the TeamCity web UI.

### Properties

It is possible to specify the following properties for your deployment:

* `zone` - the [zone](https://cloud.google.com/compute/docs/regions-zones/) in which this deployment will run.
* `version` - the [TeamCity version](https://www.jetbrains.com/teamcity/download/) to be deployed.
* `installationSize` - the size of the installation: small/medium/large.
* `serviceAccount` - the e-mail of the service account specified for the TeamCity GCE instance.
* `createStorageBucket` - allows creating a storage bucket to store build artifacts.
* `network` - the network name in the [same region](https://cloud.google.com/compute/docs/regions-zones/) which will be used by the TeamCity GCE instance.
* `subnetwork` - the subnetwork name in the same region which will be used by the TeamCity GCE instance.

#### Installation Size

The list of pre-configured installation types:

| Installation Size | Typical Usage             | vCPU | RAM  | VM Data Disk | Database         |
| ----------------- | ------------------------- | ---- | ---- | ------------ | ---------------- |
| Small             | 3 users, 100 builds/day   | 1    | 3 GB | 32 GB HDD    | db-n1-standard-1 |
| Medium            | 5 users, 300 builds/day   | 2    | 4 GB | 64 GB SSD    | db-n1-standard-1 |
| Large             | 20 users, 1000 builds/day | 4    | 8 GB | 128 GB SSD   | db-n1-standard-2 |

**Note**: See pricing for [Google Compute Engine](https://cloud.google.com/compute/pricing#custommachinetypepricing) and [MySQL database](https://cloud.google.com/sql/docs/mysql/pricing).

### Further Steps

**Note**: TeamCity server exposes an HTTP endpoint, so  make sure to enable the HTTPS endpoint for the GCE instance for production usage.

## TeamCity Update

To change the TeamCity version, start the deployment script with the required version number and then execute the [Reset](https://cloud.google.com/compute/docs/instances/restarting-an-instance) action on the TeamCity virtual machine:
 
```sh
> gcloud deployment-manager deployments update teamcity --template https://raw.githubusercontent.com/JetBrains/teamcity-google-template/master/teamcity.jinja --properties zone:<zone>,version:2017.2.2
```

**Note**: The `zone` parameter cannot be changed during the while deployment update.

## Under the Hood

During deployment, the template allocates the following resource:
* Service account with `Project Viewer`, `Cloud SQL Client`, `Compute Instance Admin`, `Storage Object Admin` and `Service Account Token Creator` roles.
* Network, firewall rules, and static IP address.
* MySQL database and user.
* GCE instance with a data disk powered by CoreOS and the assigned service account.

### GCE Instance

After deployment you will be able to connect to the GCE instance via SSH. In CoreOS TeamCity works as the following systemd service:

* `teamcity-server.service` - launches TeamCity server.
* `teamcity-agent.service` - launches TeamCity agent.

### Installed Plugins

The template installs the following Google Cloud Platform integrations in TeamCity:

* [Google Cloud Agents](https://plugins.jetbrains.com/plugin/9704-google-cloud-agents) - allows scaling the pool of TeamCity build agents by leveraging GCE.
* [Google Artifacts Storage](https://plugins.jetbrains.com/plugin/9634-google-artifact-storage) - allows storing build artifacts in Google Storage Blobs.

## Common Problems

### "Subnetwork should be specified for custom subnetmode network" error while deployment

It happens when subnetwork was not specified or does not exist in the specified zone.

### Could not connect to the TeamCity server with custom network

Ensure that you have configured firewall rules to access TeamCity server on HTTP/HTTPS port.
