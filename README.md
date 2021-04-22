# Naming Standards for Labels for Prometheus and Loki 

Guide for naming labels on Prometheus and Loki.

> NOTE: It is important to remember that as working together with both Prometheus and Loki, you apply the same or similar naming conventions on both to keep a logical standard and ease of use and understanding.

## Prometheus

Prometheus stores all data as time series (streams of timestamped values that belong to the same metric).

Each time series is uniquely identified by the **metric name** and an optional key-value pair called **label**.

The **metric name** specified the general feature of a system that is being measures.

Labels enable Prometheus's dimensional data model: any given combination of labels for the same metric name identifies a particular dimensional instantiation of that metric.

### Notation

```
<metric name>{<label name>=<label value>, ...}
```

For example, an API call using the metric `api_http_requests_total` might look something like this:

```
api_http_requests_total{method="POST", handler="/messages"}
```

## **Prometheus Best practices**

* **Avoid using labels to store dimensions with high cardinality (many different label values), like user IDs, emails, etc.**

* **Don't put the label names in the metric names, to avoid redundancy and confusion.**
* **Do not overuse labels - keep the cardinality of metric below 10.**
* **The majority of your metrics should have no labels.**
* **When unsure of using labels, you can start with no labels and add more labels over time as concrete use cases arise.**

## Loki

Labels in Loki have an important task, they define a stream.

This means that, every label key value defines the stream, when one label value changes, this creates a new stream.

Loki simplifies this aspect, by not handling metric names, just labels, and also use streams instead of series.

### Examples

This would be a basic use for STATIC labeling in Loki.

``` yaml
scrape_configs:
 - job_name: system
   pipeline_stages:
   static_configs:
   - targets:
     - localhost
     labels:
      job: syslog
      env: dev
      __path__: /var/log/syslog
 - job_name: system
   pipeline_stages:
   static_configs:
   - targets:
     - localhost
     labels:
      job: apache
      env: dev
      __path__: /var/log/apache.log

```

This assigns labels `job=syslog` and `job=apache`, creating several streams that you can query as follows:

``` 
{job="syslog"} - show logs where job label is syslog
{job="apache"} - show logs where job label is apache
{job=~"apache|syslog"} - show logs where job is apache OR syslog
{env="dev"} - show logs with env=dev, returning both log streams
```

You can also define labels dynamically as the following example shows:

``` yaml
- job_name: system
   pipeline_stages:
      - regex:
        expression: "^(?P<ip>\\S+) (?P<identd>\\S+) (?P<user>\\S+) \\[(?P<timestamp>[\\w:/]+\\s[+\\-]\\d{4})\\] \"(?P<action>\\S+)\\s?(?P<path>\\S+)?\\s?(?P<protocol>\\S+)?\" (?P<status_code>\\d{3}|-) (?P<size>\\d+|-)\\s?\"?(?P<referer>[^\"]*)\"?\\s?\"?(?P<useragent>[^\"]*)?\"?$"
    - labels:
        action:
        status_code:
   static_configs:
   - targets:
      - localhost
     labels:
      job: apache
      env: dev
      __path__: /var/log/apache.log
```

You can make use of the regex parsing a log line like the following, to give values to the labels

```
11.11.11.11 - frank [25/Jan/2000:14:00:01 -0500] "GET /1986.js HTTP/1.1" 200 932 "-" "Mozilla/5.0 (Windows; U; Windows NT 5.1; de; rv:1.9.1.7) Gecko/20091221 Firefox/3.5.7 GTB6"
```

Creating streams for each of your API calls.
Using labels responsibly in Loki will make it the most efficient and cost-effective.

> A responsible management of labels includes parallelization, to avoid high cardinality that could kill Loki with the generation of way too many streams. **Try to keep your streams and stream churn to a minimum.**

## **Loki Best practices**

* **Static labels are good**
* **Use dynamic labels sparingly**
* **Label values must always be bounded** - when using dynamic labels, don't have them unbounded or with infinite values
* **Logs must be in increasing time order per stream**
* **Use `chunk_target_size`** for more processing efficiency
* **Use filter expressions over dynamic labels**

<br>

## References

Prometheus:

[Naming practices](https://prometheus.io/docs/practices/naming/)

[Instrumentation practices](https://prometheus.io/docs/practices/instrumentation/)


Loki: 

[Getting started with labels](https://grafana.com/docs/loki/latest/getting-started/labels/)

[Loki best practices](https://grafana.com/docs/loki/latest/best-practices/)

