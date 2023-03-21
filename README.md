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
