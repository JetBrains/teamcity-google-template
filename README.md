# TeamCity template for Google Cloud Deployment Manager

Allows to create TeamCity deployment in Google Cloud by using [gcloud tool](https://cloud.google.com/sdk/gcloud/) locally or in the [Google Cloud console](https://console.cloud.google.com/).

For that copy the repository:
```
> git clone https://github.com/dtretyakov/teamcity-google-template.git
> cd teamcity-google-template
```

And deploy it by specifying template properties:
```
> gcloud deployment-manager deployments create test --template teamcity.jinja --properties zone:us-central1-a,version:2017.2.2
```

Or after editing `teamcity.yaml` config file via following command:
```
> gcloud deployment-manager deployments create test --config teamcity.yaml
```

Deployment will take several minutes, on completion you could navigate to the `teamcityUrl` output value to see TeamCity UI.

## Required Google Cloud APIs

* Enable [Cloud SQL Administration API](https://console.developers.google.com/apis/api/sqladmin.googleapis.com/overview)
* Enable [Identity and Access Management (IAM) API](https://console.developers.google.com/apis/api/iam.googleapis.com/overview)
