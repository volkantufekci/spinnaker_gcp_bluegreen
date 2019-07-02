# spinnaker_gcp_bluegreen

* `gcloud config set compute/zone us-central1-f` 

### Create a GKE Cluster
`gcloud container clusters create alyans2 --machine-type=n1-standard-2` 

### Create service account on Cloud IAM
`gcloud iam service-accounts create  spinnaker-account --display-name spinnaker-account` 

### Export some variables in order to use in commands later
```
export SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:spinnaker-account" \
    --format='value(email)')
export PROJECT=$(gcloud info --format='value(config.project)')
```

### Spinnaker needs to keep its configuration on a storage; assign storage.admin role to the service account
```
gcloud projects add-iam-policy-binding $PROJECT \
  --role roles/storage.admin --member serviceAccount:$SA_EMAIL
```

### Download service account's key; it will be passed to halyard
`gcloud iam service-accounts keys create spinnaker-sa.json --iam-account $SA_EMAIL`

## Deploying Spinnaker via Helm
### Install Helm on MacOSX
`brew install kubernetes-helm`

### Grant tiller
* `kubectl create clusterrolebinding user-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)`
* `kubectl create serviceaccount tiller --namespace kube-system`
* `kubectl create clusterrolebinding tiller-admin-binding --clusterrole=cluster-admin --serviceaccount=kube-system:tiller`

### Grant Spinnaker
`kubectl create clusterrolebinding --clusterrole=cluster-admin --serviceaccount=default:default spinnaker-admin`

### Initialize Helm and Install Tiller
* `./helm init --service-account=tiller`
* `./helm update`

### Create a bucket for Spinnaker to store its configuration
`export PROJECT=$(gcloud info --format='value(config.project)')`
`export BUCKET=$PROJECT-spinnaker-config`
`gsutil mb -c regional -l us-central1 gs://$BUCKET`

### Create Spinnaker Config
https://github.com/volkantufekci/spinnaker_gcp_bluegreen/blob/master/spinnaker-config.yaml

### Deploy Spinnaker
`helm install -n cd stable/spinnaker -f spinnaker-config.yaml --timeout 600  --version 1.13.0 --wait`

### Connect to Spinnaker via port-forward
```
export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" \
    -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &
```
