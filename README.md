# Services

## Cluster initialization

### Helm

```console
#Create tiller service account
$ kubectl apply -f helm/sa.yaml

#Initialize tiller on cluster
$ helm init --service-account helm
```

### Cert-manager

First install cert-manager (check [this](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html#installing-with-helm))

```console
# Install the CustomResourceDefinition resources separately
$kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/deploy/manifests/00-crds.yaml

# Create the namespace for cert-manager
$kubectl create namespace cert-manager

# Label the cert-manager namespace to disable resource validation
$kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true

# Add the Jetstack Helm repository
$helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
$helm repo update

# Install the cert-manager Helm chart
$helm install \
  --name cert-manager \
  --namespace cert-manager \
  --version v0.8.1 \
  jetstack/cert-manager

```

Then deploy ClusterIssuer Resource : 

```console
# Deploy clouddns key as a secret
$ kubectl create secret generic clouddns --namespace=cert-manager --from-file=certificates/clouddns.key.json

# Deploy ClusterIssuer
$ kubectl apply -f certificates/letsencrypt-clusterissuer.yaml
```



## Services deployment

### Sentry

Sentry is deployed using official helm chart (check [this](https://github.com/helm/charts/tree/master/stable/sentry#sentry)) with customized docker image and values.yaml

#### Prerequisites

##### Certificate

Sentry certificate is managed using cert-manager with following command :

```console
$ kubectl apply -f certificates/sentry.mathrix.fr-certificate
```
##### Bucket access

Sentry uses bucket as data storage. Therefore, it requires service account on GCP that has R-W access to this bucket.

```console
$ kubectl create secret generic gcs-json --from-file=sentry.mathrix.fr/credentials.json
```

#### Deployement

Sentry deployment may take a (very) long time. Adding --timeout=600 (default : 300) may solve timeout errors.

```console
$ helm install --name sentry --wait -f values.yaml stable/sentry
```

#### Initial configuration

Current values.yaml create a default admin account with user `admin@sentry.local`. Get password with following command :

```console
$ kubectl get secret --namespace default sentry -o jsonpath="{.data.user-password}" | base64 --decode
```














