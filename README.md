# Nx Cloud & K8s Example

A lot of organizations deploy Nx Private Cloud to Kubernetes. This repo is an example of how to deploy Nx Cloud to Google Cloud (GKE). Some configuration may look different for you depending on where and how you deploy things.

## Steps

1. Create a GKE cluster
2. Create secrets by running `kubectl apply -f secrets.yml`
3. Deploy MongoDB Kubernetes Operator. [See here](https://github.com/mongodb/mongodb-kubernetes-operator)
4. Create a MongoDB replica set by running `kubectl apply -f mongodb.yml`
5. Create a persistent volume claim by running `kubectl apply -f pvc.yml`
6. Create an Nx Cloud pod by running `kubectl apply -f cloud.yml`
7. Create an external load balancer or an Ingress instance by running either `kubectl apply -f load-balancer.yml` or `kubectl apply -f ingress`

See the information below about each step.

## Secrets

Secrets aren't encrypted, so we recommend you use an appropriate KMS plugin. This repo is simply an example.

## MongoDB

If you are using a hosted MongoDB installation (e.g., Mongo Atlas or CosmosSB, or you are running one yourself), you can skip steps 3 and 4. It's also possible to run Nx Cloud with the MongoDB database embedded (see cloud-embedded-db.yml), but we recommend against it.

## PVC

If you use AWS or Azure, you can configure Nx Cloud to store cached artifacts on S3 or Azure Blob. See [here](https://nx.app/docs/get-started-with-private-cloud-community#using-external-file-storage) for more information. If you do that, you don't need a PVC (see cloud-external-file-storage.yml).

## Nx Cloud Pod

First, if the container embeds MongoDB, we cannot run more than 1 replica and the rollout strategy has to be configured as follows:

```
maxUnavailable: 1
maxSurge: 0
```

The Deployment controller will bring the old replica down before starting the new one. The Nx Cloud client (that is running your tasks on your CI) will wait for the new container to boot, so no downtime will occur.

Second, NX_CLOUD_APP_URL is the domain of the external IP of the LoadBalancer. You use it when running: `NX_CLOUD_API=http://some-ip-or-domain nx connect-to-nx-cloud`.

Third, if you use AWS or Azure to store cached artifacts, and you don't run with the MongoDB embedded, Nx Cloud doesn't need a persistent volume.

Fourth, if you are using an embedded file server, the PersistentVolumeClaim's accessMode affects how many replicas you can run. If it's set to ReadWriteMany, you can run many replicas. If it's set to ReadWriteOne, you can run many replicas.

## Load Balancer and Ingress

There are many ways to expose Nx Cloud to the outside world. This repo contains two examples:
* External load balancer (see load-balancer.yml)
* Ingress (see ingress.yml). You will have to configure a static IP using the gcloud cli.