# Kubernetes_pipeline

Current stage of the repository is to migrate to using Helm Charts instead of individual manifests.

The /charts directory contains charts for each pipeline step.

Each Chart has the necessary manifests in the /templates directory and when installing the chart, the templates take values from the values.yaml file.

## Creating a chart
There is a chart inside the /charts directory called step_starter. This chart is a template chart, which means it can be used as a base chart to create new charts that will have some common resources.

Run `helm create --starter /absolute/path/to/step_starter chart_name` to create a new chart.

## Installing a chart
First create a release `helm package path/to/chart`
Run `helm upgrade --install release-name path/to/chart.gz`
You are encouraged to create a values.override.yaml file that you can fill with custom values, overriding the defaults of the chart's values.yaml.
To install the chart with custom values run `helm upgrade --install -f path/to/values.override.yaml release-name path/to/chart.gz`

## Best practices
- Do not use local packages. Instead, add the remote [repo](https://alercebroker.github.io/kubernetes_pipeline/) with all the charts available and install from there.
  
  Once Helm has been set up correctly, add the repo as follows:

      helm repo add <alias> https://alercebroker.github.io/kubernetes_pipeline

  If you had already added this repo earlier, run `helm repo update` to retrieve
  the latest versions of the packages.  You can then run `helm search repo <alias>` to see the charts.

  To install the <chart-name> chart:

      helm install <release-name> <alias>/<chart-name>

  To uninstall the chart:

      helm delete <release-name>

## About the pipeline

### Pipeline topology (simplified)

![Image](https://user-images.githubusercontent.com/20263599/228923692-27d46532-955f-4f8c-9cfe-94489300fb59.png)

### Steps explanation

 - [Sorting Hat](https://github.com/alercebroker/sorting_hat_step): Assigns an ALeRCE ID (aid) to the alerts received by it. This step performs a crossmatch to check if the alerted object must be assigned to a previously created `aid` or need to create a new `aid` that will contain this alert.
 - [Previous Candidates (prv_candidates)](https://github.com/alercebroker/prv_candidates_step): Decodes the `extra_fields` (only in ZTF alerts for now) and obtains the previous candidates in the alert. Returns an AVRO package with 3 fields: the `aid`, a list of *detections* and a list of *non_detections*. 
 - [Stamp Classifier](https://github.com/alercebroker/stamp_classifier_step): Uses the `stamp_classifier` algorithm to classify images into different astronomical objects. 
 - [Magstats](https://github.com/alercebroker/magstats_step): Calculates some important statistics of the object.
 - [Lightcurve](https://github.com/alercebroker/lightcurve-step): Retrieves the full lightcurve of a given object.
 - [Xmatch](https://github.com/alercebroker/xmatch_step): Performs a crossmatch using the ALLWISE catalog.
 - [Features](https://github.com/alercebroker/feature_step): Obtain the features of a given lightcurve, using FeatureExtractors.
 - [LC Classifier](https://github.com/alercebroker/lc_classification_step): Perform a inference given a set of features. This might not get a classification if the object lacks of certain features.

Other steps that aren't part of the alert processing pipeline

 - [S3](https://github.com/alercebroker/s3_step): Upload the AVRO files in a AWS S3 Bucket. 
 - [Watchlist](https://github.com/alercebroker/watchlist_step): Receives alerts and perform crossmatch to check if there are objects on a watchlist created by an user. If there is, then the user is notified.
 - [Scribe](https://github.com/alercebroker/alerce-scribe): CQRS Step which allows other steps to perform asyncronous write database operations.

### Glossary

 - Alert:
 - Survey: ...
 - Object: ...
 - Detection: ...
 - Non Detection: ...

More details in the PIPELINE.md file