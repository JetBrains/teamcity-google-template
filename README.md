# TeamCity template for Google Cloud Deployment Manager

Allows to create TeamCity deployment in Google Cloud by using [gcloud tool](https://cloud.google.com/sdk/gcloud/) locally or in the [Google Cloud console](https://console.cloud.google.com/).

## Pre-requisites

### Google Cloud APIs

Ensure that you have enabled following Google Cloud APIs in your project:
* [Cloud Deployment Manager V2 API](https://console.cloud.google.com/apis/api/deploymentmanager.googleapis.com/overview)
* [Cloud SQL Administration API](https://console.developers.google.com/apis/api/sqladmin.googleapis.com/overview)

### Service account

If you don't have [Compute Engine default service account](https://cloud.google.com/compute/docs/access/service-accounts#compute_engine_default_service_account) in your project you need to [create a service account](https://cloud.google.com/compute/docs/access/service-accounts#newserviceaccounts) for TeamCity and assign the [following roles](https://cloud.google.com/iam/docs/understanding-roles):
* `Cloud SQL Client` - to access TeamCity database.
* `Compute Instance Admin (v1)` - to use use [Google cloud build agents](https://plugins.jetbrains.com/plugin/9704-google-cloud-agents).
* `Project Viewer` / `Storage Object Admin` - to store [TeamCity build artifacts in Google Storage](https://plugins.jetbrains.com/plugin/9634-google-artifact-storage).

## Deployment

Clone this repository locally:
```
> git clone https://github.com/dtretyakov/teamcity-google-template.git
> cd teamcity-google-template
```

### Properties

You could specify the following properties for deployment:

* `zone` - [zone](https://cloud.google.com/compute/docs/regions-zones/) in which this deployment will run.
* `version` - TeamCity version to deploy.
* `installationSize` - the size of installation: small/medium/large
* `serviceAccount` - the service account e-mail for TeamCity virtual machine.

#### Installation Size

List of pre-configured installation types:

| Installation Size | Typical Usage             | Machine Type   | VM Data Disk | Database         |
| ----------------- | ------------------------- | -------------- | ------------ | ---------------- |
| Small             | 3 users, 100 builds/day   | n1-standard-1  | 32 GB HDD    | db-n1-standard-1 |
| Medium            | 5 users, 300 builds/day   | n1-highcpu-2   | 64 GB SSD    | db-n1-standard-1 |
| Large             | 20 users, 1000 builds/day | n1-highcpu-4   | 128 GB SSD   | db-n1-standard-2 |

**Note**: Pricing for [Google Compute Engine](https://cloud.google.com/compute/pricing) and [MySQL database](https://cloud.google.com/sql/docs/mysql/pricing).

### Template

Deploy TeamCity as a template by specifying properties:
```
> gcloud deployment-manager deployments create test --template teamcity.jinja --properties \
  zone:us-central1-a,version:2017.2.2,installationSize:medium,serviceAccount:account@my-project-123iam.gserviceaccount.com
```

### Configuration

Deploy TeamCity after editing `teamcity.yaml` config file via following command:
```
> gcloud deployment-manager deployments create test --config teamcity.yaml
```

**Note**: Deployment will take several minutes, on completion you could navigate to the `teamcityUrl` output value to see TeamCity UI.

