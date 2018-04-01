# k8s-aws
This is a kops wrapper script for creating Kubernetes clusters on AWS and maintaining them.
Certain addons are also installed into newly created clusters:

* Heapster
* Kubernetes Dashboard
* Elasticsearch/Fluentd/Kibana (EFK) logging stack
* [Prometheus Operator](https://github.com/coreos/prometheus-operator) monitoring stack

The cluster is additionally set up with
[RBAC authorization](https://kubernetes.io/docs/admin/authorization/rbac/) for heightened security.

## Requirements
We use Python 3. To install dependencies, issue the following command at the root of this project:
`pip3 install -r requirements.txt`.

## Secrets
Write your secrets in secrets.yaml, see secrets.yaml.sample.

## Usage
The ./kops script supports a few subcommands and a number of arguments:

### Create a Cluster
The create subcommand allows you to create a Kubernetes cluster within your AWS account. You must have AWS CLI configured and S3 bucket created.

See [Setting AWS cluster](https://github.com/kubernetes/kops/blob/master/docs/aws.md) for more information.

```
./kops --aws_profile <preconfigured-aws-profile> create <cluster-name> <cluster-category> <email-for-notifications> <smtp-domain> <base-domain-for-cluster> <s3-bucket-name>
```

Example:

```
./kops --aws_profile example create my-cluster staging contact@example.com email-smtp.eu-west-1.amazonaws.com:587 example.com my-cluster-bucket
```

## Logging
### Kibana
In order to access Kibana, start `kubectl --kubeconfig $KUBECONFIG proxy 8001` and open
http://localhost:8001/api/v1/proxy/namespaces/kube-system/services/kibana-logging in your browser.
The username/password is elastic/changeme.

## Monitoring
We use Prometheus, via CoreOS'
[Prometheus Operator](https://github.com/coreos/prometheus-operator), as our monitoring solution.
In addition to Prometheus Operator itself, we install the whole
[kube-prometheus](https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus)
stack that among other things ensures that the Kubernetes cluster itself and the host
infrastructure are monitored.

### Upgrading to a New Version
When upgrading to a newer version of the Prometheus Operator stack, we need to first upgrade
Prometheus Operator itself (by applying its manifests). Once this is ready, we can apply the
manifests for the rest of the stack. Prometheus Operator will take care of upgrading corresponding
components in the cluster.

### Visualization
We use Grafana for visualizing monitoring information, is pre-configured with useful dashboards.

In order to access Prometheus Operator, start `kubectl --kubeconfig $KUBECONFIG proxy 8001` and open.

For Grafana UI:
http://localhost:8001/api/v1/namespaces/monitoring/services/grafana/proxy in your browser.

For AlertManager UI:
http://localhost:8001/api/v1/namespaces/monitoring/services/alertmanager-main:web/proxy in your browser.

For Prometheus UI:
http://localhost:8001/api/v1/namespaces/monitoring/services/prometheus-operated:web/proxy in your browser.
