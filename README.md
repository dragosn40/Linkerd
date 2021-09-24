# Linkerd


Setting up our Kubernetes Cluster
Before installing Linkerd we need an underlying Kubernetes Cluster. If you don't have one, you can create on with the following steps:

export ORG=dragosn
export PRODUCT=notepad
export ENV=stg
export PROJECT_PREFIX=2  
export PROJECT_NAME=$ORG-$PRODUCT-$ENV
export PROJECT_ID=$ORG-$PRODUCT-$ENV-$PROJECT_PREFIX

VPC
gcloud compute networks subnets create gke-priv-cluster-subnet \
--network $ORG-$PRODUCT-$ENV-vpc \
--range 10.128.0.0/26 \
--region us-central1 --enable-flow-logs \
--enable-private-ip-google-access \
--secondary-range services=10.10.0.0/21,pods=172.10.0.0/18

Cluster
gcloud container clusters create  $ORG-$PRODUCT-$ENV-cluster \
    --region us-central1 \
    --num-nodes 1 \
    --machine-type "e2-medium" \
    --cluster-version "1.21.3-gke.2001" \
    --release-channel "rapid" \
    --enable-network-policy \
    --network $ORG-$PRODUCT-$ENV-vpc \
    --subnetwork gke-priv-cluster-subnet \
    --cluster-secondary-range-name pods \
    --services-secondary-range-name services \
    --enable-ip-alias \
    --enable-private-nodes \
    --master-ipv4-cidr 172.16.0.0/28
    
  Instaling the control plane   
    gcloud compute firewall-rules create gke-to-linkerd-control-plane \
  --network "$NETWORK" \
  --allow "tcp:8443,tcp:8089" \
  --source-ranges "$MASTER_IPV4_CIDR" \
  --target-tags "$NETWORK_TARGET_TAG" \
  --priority 1000 \
  --description "Allow traffic on ports 8443, 8089 for linkerd control-plane components"
