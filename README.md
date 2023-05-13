# k8s-deploymnets

The goal is to deploy the below application stack in kubernetes 
* webserver (nginx)
* postgres
* redis


The following criteria must be met
* The webserver must be able to connect to redis and postgres service
* Database data is persistent
* Inbound communication from the internet to the web application needs to be available via HTTPS only
* web server and redis containers should be configured with autoscaling based on CPU and memory usage
* Make sure that the redis container only deploys on nodes that have the label of: cache, using affinities
* Ensure an example of liveliness, readiness, and startup probes are documented


### Software pre-requirements

The below requirements are listed for MacOS. Ideally it should also work for other distributions 
* `brew install kubectl `
* `brew install helm`
* `brew install helmfile`
* `helm plugin install https://github.com/databus23/helm-diff `
*  Kubernetes cluster (Docker-Desktop for MacOS)

### Pre-configure cluster 

####Mandatory 
* Install kubernetes metrics server for built-in horizontal auto pod scaling 

  `kubectl apply -f misc/metrics.yaml`

* Add `cache` labels on k8s worker nodes. So that the affinity rules are applied for redis deployments 

  `kubectl label nodes docker-desktop in-memory-database=cache`

####Optional

* Configure storage class if you have any, if not you can skip this part. By default, pods are configured to use `hostpath` for volumes if the storage class is not configured.

  `kubectl apply -f misc/storageClass.yaml misc/persistentVolume.yaml`

* Configure vault secret if you have any, if not you can skip this part. By default, the secrets are read from plain text if vault is not configured.

  `kubectl apply -f misc/storageClass.yaml`


### Deployment

* Apply/Update the values accordingly in `values.yaml` globally to override the values in all the charts `values.yaml` files 

    ```
    # If set to `pvc` is to `true`, then pvc logic will be applied else volumes are mounted hostpath (default setting)
    pvc:
      enabled: false
      storageClassName: my-local-storage
    # If set to `vault` is to `true`, then secrets will be rendered from vault else from plain text
    vault:
      enabled : false```

* `helmfile apply --namespace service`

* `helmfile destroy --namespace service`




### Outputs/Results

* Please refer to outputs directory for results (screenshots)
* The webserver can reach postgres and redis as they are in same namespace
* The webserver is reachable via https only
* Autoscaling parameter based on CPU and memory is defined in every `values.yaml` of charts 
  For example: 
  ```
  autoscaling:
    enabled: true
    minReplicas: 1
    maxReplicas: 20
    targetCPUUtilizationPercentage: 80
    targetMemoryUtilizationPercentage: 80```
* Volumes are configured for persistent storage for databases 
* Liveliness, readiness, and startup probes configured in all the `deployment.yaml`
* Added label cache so that redis can use the affinity rule

