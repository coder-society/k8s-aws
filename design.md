# Kubernetes Cluster Design
This document describes the various aspects of the design of our Kubernetes clusters.

## Creating
We use the [Kops](https://github.com/kubernetes/kops) tool to create our Kubernetes clusters,
via a wrapper script.

### Internet Gateway
Each cluster operates in private subnets (one per AZ, typically three), but can connect to the
Internet through
[NAT gateways](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-gateway.html)
(one per AZ). Each NAT gateway has an elastic IP assigned to it.

## Logging
We use the more or less standard EFK ([Elasticsearch](https://www.elastic.co/products/elasticsearch)/
[Fluentd](http://www.fluentd.org/)/[Kibana](https://www.elastic.co/products/kibana)) stack as our
logging soluton. Elasticsearch provides a searchable database of log entries, which are collected
from our cluster by Fluentd. Kibana provides a graphical interface to Elasticsearch.

Unless you have a public endpoint for Kibana, access it via the kubectl proxy:

1. Start `kubectl proxy 8001` in a terminal.
2. Visit http://localhost:8001/api/v1/proxy/namespaces/kube-system/services/kibana-logging in your
browser.

## Monitoring
We use [Prometheus](https://prometheus.io) as our monitoring system, as this is open source and
the currently most popular monitoring system for Kubernetes. It is furthermore extremely powerful
and extensible. We integrate Prometheus into our clusters via the [CoreOS Prometheus
Operator](https://github.com/coreos/prometheus-operator) for Kubernetes as the latter installs
an automated "operator" (a sort of bot) inside the cluster that extends the Kubernetes API
to allow creating/configuring Prometheus clusters. The operator will also manage our Prometheus
clusters for us automatically, saving human effort basically.

Moreover, we use the
[kube-prometheus](https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus)
"package" to install a whole monitoring stack in addition to the Prometheus Operator itself. It
provides:

* A Prometheus cluster setup
* A configuration of the Prometheus cluster that covers all Kubernetes core components and exporters
* A default set of alerting rules on the cluster components' health
* A Grafana instance configured with dashboards on cluster metrics
* A three node highly available AlertManager cluster

### Alerts
In addition to Prometheus itself, the Prometheus Operator installs
[Alertmanager](https://prometheus.io/docs/alerting/alertmanager/), which is a piece of companion
software that'll act on alerts from Prometheus and ensure that notifications are sent the way
we want, such as by email.

We control Alertmanager's behaviour, such as how it should notify by email, via its configuration
file. We don't edit the configuration file directly, but via a template in
assets/alertmanager-config.yaml. The real configuration gets generated from this template by
the kops wrapper script, substituting any variables.

#### Debugging
In order to debug your alerts setup, you can trigger an alert with Alertmanager by hand
(providing that the local port 9093 is forwarded to Alertmanager):
`curl -d '[{"labels": {"Alertname": "Test Alert"}}]' http://localhost:9093/api/v1/alerts`. This
is typically useful in order to see if you receive notifications for alerts as you expect.
