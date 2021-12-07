
# Logzio cloudwatch metrics

##  Overview
The Helm tool is used to manage packages of pre-configured Kubernetes resources that use charts.
Use this Helm chart to ship AWS cloudwatch metrics to Logz.io via the OpenTelemetry collector and cloudwatch exporter.

#### Standard configuration

##### Deploy the Helm chart
First add `logzio-helm` repo
```shell
helm repo add logzio-helm https://logzio.github.io/logzio-helm
helm repo update
````

###### Configure the chart values

| Helm value | Description |
|---|---|
| config.otel.logzio_region (Required)| Your Logz.io region code. For example if your region is US, then your region code is `us`. You can find your region code here: https://docs.logz.io/user-guide/accounts/account-region.html#regions-and-urls. |
| config.otel.token (Required)| Token for shipping metrics to your Logz.io account. Find it under Settings > Manage accounts. [_How do I look up my Metrics account token?_](/user-guide/accounts/finding-your-metrics-account-token/) |
| config.otel.scrape_interval | The time interval (in seconds) during which the Cloudwatch exporter retrieves metrics from Cloudwatch, and the Opentelemtry collector scrapes and sends the metrics to Logz.io. Default = `300`.   **Note:** This value must be a multiple of 60.|
| config.otel.p8s_logzio_name | The value of the `p8s_logzio_name` external label. This variable identifies which Prometheus environment the metrics arriving at Logz.io came from. Default = `logzio-cloudwatch-metrics`.  |
| config.otel.custom_listener | Set a custom URL to ship metrics to (for example, http://localhost:9200). This overrides the `LOGZIO_REGION` Environment variable. |
| config.otel.scrape_timeout | The time to wait before throttling a scrape request to cloudwatch exporter, Default = `120`|
| config.otel.remote_timeout | the time to wait before throttling remote write post request to logz.io, Default = `120`|
| config.otel.log_level | Opentelemetry log level, Default = `debug` |
| config.otel.logzio_log_level | `builder.py` Python script log level. Default = `info` |
| config.otel.AWS_ACCESS_KEY_ID | Your IAM user's access key ID. |
| config.otel.AWS_SECRET_ACCESS_KEY | Your IAM user's secret key. |
| config.cloudwatch.region (Required) | Your region's slug. You can find this in the AWS Console region menu (in the top menu, to the right).  **Note:** This is the region that you will collect metrics from. |
| config.cloudwatch.aws_namespaces (Required) | Comma-separated list of namespaces of the metrics you want to collect. You can find a complete list of namespaces at [_AWS Services That Publish CloudWatch Metrics_](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html).   **Note:** This Environment variable is required unless you define the `CUSTOM_CONFIG` Environment variable |
| config.cloudwatch.custom_config | Mount your cloudwatch exporter configuration file `-v pathToConfig/config.yml:config_files/cloudwatch.yml` and set to `true` if you want to use custom configuration for cloudwatch exporter, Default = `false` |
| config.cloudwatch.role_arn | Your IAM role to assume. |
| config.cloudwatch.set_timestamp | Boolean for whether to set the Prometheus metric timestamp as the original Cloudwatch timestamp. Default = `false` |
| config.cloudwatch.period_seconds | period to request the metric for. Only the most recent data point is used. Default = `300` |
| config.cloudwatch.range_seconds | how far back to request data for. Useful for cases such as Billing metrics that are only set every few hours. Default = `300` |
| config.cloudwatch.delay_seconds | The newest data to request. Used to avoid collecting data that has not fully converged. Default = `300` |

##### Customize the metrics collected by the Helm chart

You can use a custom cloudwatch exporter configuration by adding inline yaml to the `custom_cloudwatch_config` section. For example:
```yaml
custom_cloudwatch_config: |-
  region: eu-west-1
  period_seconds: 240
  metrics:
  - aws_namespace: AWS/ELB
    aws_metric_name: HealthyHostCount
    aws_dimensions: [AvailabilityZone, LoadBalancerName]
    aws_statistics: [Average]
  - aws_namespace: AWS/ELB
    aws_metric_name: UnHealthyHostCount
    aws_dimensions: [AvailabilityZone, LoadBalancerName]
    aws_statistics: [Average]
```
To learn more about cloudwatch exporter configuration you can refer to the [documantation](https://github.com/prometheus/cloudwatch_exporter#configuration)

###### Run the Helm deployment code 
To deploy the Helm chart, enter the relevant parameters for the placeholders and run the code.

```
helm install  \
--set config.otel.token=<<TOKEN>> \
--set config.otel.logzio_region=<<LOGZIO_REGION>> \
--set config.otel.AWS_ACCESS_KEY_ID=<<AWS-ACCESS-KEY>> \
--set config.otel.AWS_SECRET_ACCESS_KEY=<<AWS-SECRET-KEY>> \
--set config.cloudwatch.region=<<AWS-REGION>> \
--set "config.cloudwatch.aws_namespaces={<<AWS-NAMESPACES>>}" \
logzio-cloudwatch-metrics logzio-helm/cloudwatch-metrics
```

##### Check Logz.io for your metrics

Give your metrics some time to get from your system to ours, then open [Logz.io](https://app.logz.io/).


####  Customizing Helm chart parameters


##### Configure customization options

You can use the following options to update the Helm chart parameters: 

* Specify parameters using the `--set key=value[,key=value]` argument to `helm install`

* Edit the `values.yaml`

* Overide default values with your own `my_values.yaml` and apply it in the `helm install` command. 

###### Example:

```
helm install logzio-ccloudwatch-metrics logzio-helm/cloudwatch-metrics -f my_values.yaml 
```

#### Uninstalling the Chart

The uninstall command is used to remove all the Kubernetes components associated with the chart and to delete the release.  

To uninstall the `cloudwatch-metrics` deployment, use the following command:

```shell
helm uninstall cloudwatch-metrics
```


## Change log

* 1.0.0 - Initial realese
