# kubernetes-workshop

## github repository

create a github repository (may be public or private, doesn't matter much)

copy folder examples from this repo to your repo and create `flux` folder in your repo

## minikube

install minikube [github.com/kubernetes/minikube](https://github.com/kubernetes/minikube)

Create minikube cluster:

```bash
minikube start --kubernetes-version='v1.15.4' --cpus=4 --memory='4000mb'
```

Verify it's working:

```bash
$ kubectl version --short
Client Version: v1.15.3
Server Version: v1.15.4
```

## helm

<!-- not run, just explain -->
<!-- helm init -o yaml > examples/resources/tiller.yaml -->

```bash
kubectl apply -f examples/resources/tiller.yaml
```

```bash
$ helm version
Client: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
```

Copy `examples/resources/tiller.yaml` to `flux/resources/tiller.yaml` and push to the repository.

## flux

```bash
helm repo add fluxcd https://charts.fluxcd.io
```

```bash
kubectl apply -f https://raw.githubusercontent.com/fluxcd/flux/helm-0.10.1/deploy-helm/flux-helm-release-crd.yaml
```

```bash
helm upgrade --install flux --namespace kube-system -f examples/flux-initial-values.yaml --version 0.14.1 fluxcd/flux
```

```bash
kubectl -n kube-system logs deployment/flux | grep identity.pub | cut -d '"' -f2
```

Add public key from output to your repository, so Flux could access it.

`Settings -> deploy keys -> add deploy key -> paste key -> Allow write access`

```bash
kubectl -n kube-system logs deployment/flux -f
```

## create slack workspace

1. Create slack workspace
2. create slack app
3. enable incomming webhooks
4. create two webhooks. one for fluxcloud for notifications, seconds for alertmanager. Examples:
    - `#flux-notifications`
    - `#prometheus-alerts`
5. Test that notification goes in manually.

```bash
curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' https://hooks.slack.com/services/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
```

## fluxcloud

1. add `examples/resources/fluxcloud.yaml` to `flux/` folder
2. edit the file, use first webhook as `SLACK_URL` value.
3. commit and push

```bash
$ kubectl -n kube-system logs deployment/fluxcloud -f
[{#flux-notifications *}]
Using Slack exporter
```

## flux v2

1. put `examples/helmreleases/flux.yaml` under `flux/` folder as well.
2. check the difference between `examples/helmreleases/flux.yaml` and initial flux vaules that you have deployed (`examples/flux-initial-values.yaml`)
3. git commit
4. wait for deploy

Verify that flux connected to fluxcloud successfully.

```bash
$ kubectl -n kube-system logs deployment/fluxcloud -f
[{#flux-notifications *}]
Using Slack exporter
Request for:/v11/daemon
client connected!
```

## test notification

put `examples/namespaces/namespace.yaml` under flux and look in slack channel for notification to appear.

## prometheus

Prometheus operator is used in this example.
https://github.com/helm/charts/tree/master/stable/prometheus-operator
https://github.com/coreos/prometheus-operator

Create CRD:

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/alertmanager.crd.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/prometheus.crd.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/prometheusrule.crd.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/servicemonitor.crd.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/podmonitor.crd.yaml
```

1. put `examples/helmreleases/prometheus-operator.yaml` under flux.
2. edit the file, use second webhook as alertmanager config for `slack_api_url:`
3. push to github

```bash
kubectl port-forward -n kube-system svc/prometheus-operator-prometheus 9090:9090
kubectl port-forward -n kube-system svc/prometheus-operator-grafana 8080:80
kubectl port-forward -n kube-system svc/prometheus-operator-alertmanager 9093:9093
```

Note: credentials to grafana are `admin/prom-operator`

## add blackbox exporter

https://github.com/helm/charts/tree/master/stable/prometheus-blackbox-exporter
https://github.com/prometheus/blackbox_exporter

put `examples/helmreleases/blackbox-exporter.yaml` under flux as usual.

verify the functionality:

```bash
kubectl port-forward -n kube-system svc/prometheus-blackbox-exporter 9115:9115
curl "http://localhost:9115/probe?target=https://google.com&module=http_2xx"
```

1. add prom rule under flux
2. add 2 service monitors under flux.
3. see the results in Prometheus, and alerts in slack.

## faq

### run flux sync manually

```bash
fluxctl --k8s-fwd-ns kube-system sync
```

### push to repo

```bash
git add flux/resources/filename
git commit -m "add filename"
git push
```

## enchancements to do (homework)

- put crds under flux as well
- add sealed secrets for managing secrets ([github.com/bitnami-labs/sealed-secrets](https://github.com/bitnami-labs/sealed-secrets))
