# kubernetes-workshop

## github repository

create a github repository (may be public or private, doesn't matter much)
copy folder examples from this repo to your repo and create flux folder in your repo

## minikube

install minikube
https://github.com/kubernetes/minikube

run
minikube start --kubernetes-version='v1.15.4' --cpus=4 --memory='4000mb'

$ kubectl version --short
Client Version: v1.15.3
Server Version: v1.15.4

## helm

not run, just explain
helm init -o yaml > examples/resources/tiller.yaml

kubectl apply -f examples/resources/tiller.yaml

$ helm version
Client: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}

cp examples/resources/tiller.yaml flux/resources/tiller.yaml
git add flux/resources/tiller.yaml
git commit -m "add tiller"
git push

# flux

helm repo add fluxcd https://charts.fluxcd.io

kubectl apply -f https://raw.githubusercontent.com/fluxcd/flux/helm-0.10.1/deploy-helm/flux-helm-release-crd.yaml

helm upgrade --install flux --namespace kube-system -f examples/flux-initial-values.yaml --version 0.14.1 fluxcd/flux

kubectl -n kube-system logs deployment/flux | grep identity.pub | cut -d '"' -f2

settings -> deploy keys -> add deploy key -> paste key -> Allow write access.

kubectl -n kube-system logs deployment/flux -f

## create slack workspace

create slack app
enable incomming webhooks
create a webhook
test it 
curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' https://hooks.slack.com/services/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

two webhooks
one for fluxcloud for notifications
seconds for alertmanager
#flux-notifications
#prometheus-alerts

## fluxcloud

add it to flux/ folder and commit
edit configuration, use first webhook

$ kubectl -n kube-system logs deployment/fluxcloud -f
$ kubectl -n kube-system logs deployment/fluxcloud -f
[{#flux-notifications *}]
Using Slack exporter

## flux v2

put flux under itself?
git commit
wait for deploy

kubectl -n kube-system logs deployment/fluxcloud -f
$ kubectl -n kube-system logs deployment/fluxcloud -f
[{#flux-notifications *}]
Using Slack exporter
Request for:/v11/daemon
client connected!
flux cloud is connected

## test notifications

namespaces/namespace.yaml to git

notification appeared.

# prometheus


kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/alertmanager.crd.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/prometheus.crd.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/prometheusrule.crd.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/servicemonitor.crd.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/v0.32.0/example/prometheus-operator-crd/podmonitor.crd.yaml

put prom-operator to helmreleases
edit, use second webhook

kubectl port-forward -n kube-system svc/prom-operator-prometheus 9090:9090

kubectl port-forward -n kube-system svc/prom-operator-grafana 8080:80

admin/prom-operator










faq

fluxctl --k8s-fwd-ns kube-system sync


enchancements:

put crds under flux as well
