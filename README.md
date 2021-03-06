# Kubernetes Logging Demo

An example of deploying a local Kubernetes cluster using Minikube,
capturing the cluster's logs, and sending them to an external
system. 

The code gives you the ability to create logs either using a 
random logging pod or by means of an API that can be built
using this repo as well.

You will have the options of sending the logs to either Elasticsearch
or Kafka or both. For both of these services, I recommend using
[Aiven's platform](https://console.aiven.io/signup?utm_source=github&utm_medium=organic&utm_campaign=k8s-logging&utm_content=signup)
as it easy to get started with and security is
a first class citizen. You can signup for a free trial
which should give you plenty of credits to get started.

## Dependencies

### Install Docker
 * https://docs.docker.com/get-docker/

### Install Minikube and create k8s cluster
 * https://minikube.sigs.k8s.io/docs/start/

### Install Helm
 * https://helm.sh/docs/intro/install/

### Install Kubectl
 * https://kubernetes.io/docs/tasks/tools/

### FluentD Helm Chart Reference
 * https://github.com/bitnami/charts/tree/master/bitnami/fluentd/#installing-the-chart

## Elasticsearch
Create an [Aiven Elasticsearch](https://aiven.io/elasticsearch) 
service. Take note of the host, port, username and password. 
These will be added to `k8s-logging-demo/chart/values.yaml` 
or they can be specified via CLI.

## Kafka (Optional)
Create an [Aiven Kafka](https://aiven.io/kafka)
service. Take note of the host, port, and SASL connection
details. There will be added to `k8s-logging-demo/chart/values.yaml` 
or they can be specified via CLI.

## API (Optional)
Build the api locally
```bash
cd k8s-logging-demo
docker build -t local/log-demo .
```
Set the value of `api.enabled` to `true` in `chart/values.yaml`

## Build and Deploy
Create `logging` namespace
```bash
kubectl create ns logging
```

Add and Update Helm Dependencies
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm dependency update chart
```

Point your shell to minikube's docker-daemon, run:
```bash
eval $(minikube -p minikube docker-env)
``` 

Install the Helm Chart
```bash
helm install -n logging log-demo chart \
--set elasticsearch.hosts=<ES Host> \
--set elasticsearch.pw=<ES Password>
```

## Random Logging (Optional)
```bash
kubectl create deployment -n logging --image=chentex/random-logger:latest logger
```
To remove:
```bash
kubectl delete -n logging deployment/logger
```

## Viewing Logs
Open Kibana and check discovery. You will see kube system 
logs in the dashboard

Port forward the API
```bash
kubectl port-forward -n logging svc/api-service 8080:8080
```

Make a curl request
```bash
curl -X POST localhost:8080/echo -d'{"key": "HELLO THERE"}'
```

You should be able to see a logged warning message from the
api that has the payload in it.


## Optional Configurations
You can enable Kafka output as well. Add the following to the 
helm command
```
kafka_broker=<host>:<port>
kafka_user=user
kafka_pw=password
kafka_ca_cert='
<Certificate Contents>
'

--set kafka.brokers=$kafka_broker \
--set kafka.user=$kafka_user \
--set kafka.pw=$kafka_pw \
--set kafka.caCert=$kafka_ca_cert \
--set kafka.enabled=true
```

To turn either of kafka or elasticsearch output off
just set their enabled flag to `false`


