# Kubernetes(k8s) for Developers

As title explains, this repo helps to understand k8s from Developers Perspective.

Pre-Requiste to understand k8s
  - Software Development lifecycle
  - Container Native Development/Container Run time(CNCF Implementation)
  - Basic knowlege about yaml/yml

Docker Desktop would be ideal for Experimenting with k8s futures.

`kubectl` is the command to communicate with cluster apiServer. using this command we can create various resources in k8s cluster. But yaml/yml file is the preferred way to create resource/define states in k8s.

*command to verify k8s cluster status*

`kubectl cluster-info` and 
`kubectl get all` should give you the status of your cluster.

## Pods
  Pods are minimal working unit of k8s
  - `kubectl create -f pods/nginx-pod.yml` will create my-nginx pod
  - `kubectl get pods` will list out the pod READY status and RUNNING status

  As of now my-nginx pod can be accessed with in cluster. Try to get the ip of the node in which my-nginx pod is running.
  - `kubectl get pods -o wide` will list out the ip of the node.
  - `curl <node IP:80>` will list out nginx welcome page.

  To access the pod outside the cluster we should do port-forward. So that it can be accessed everywhere.
  - `kubectl port-forward pod/nginx 80:80` this will enable to access the pod outside the k8s cluster.

  - Below command can be used to validate any reousrce yaml file.
  `kubectl create -f pods/nginx-pod.yml --dry-run=client --validate=true`

### Pods Healthcheck
  - Readiness Probe: when should a container start receiving traffic?
  - Liveness Probe: when should a container restart due to Health issue?

    As explained in the liveness probe, k8s keeps monitoring its Health Status. To break this, I logged into container `kubectl exec -it my-nginx sh` and removed the index.html from /usr/share/nginx/html folder. I exited out from container

    Now I describe the pod by giving `kubectl describe pod/my-nginx` command. At the bottom, I got `Warning Unhealthy Message stating that Liveness probe failed: HTTP probe failed with statuscode: 404` and `Normal type message that it will be restarted`.

## Deployments
  Deployments are more abstract way of deploying pods and replica sets. In General, we create deployment yaml file with desired no of pods in replica sets attribute.
  - `kubectl create -f deployments/nginx-deployment.yml --save-config` will create deployment resources.
  - `kubectl get all` will list out all resources including pods, replicasets and deployments.
  - `kubectl describe <pods>|<replicasets>|<deployments> nginx` should list out the requested resources and its state details.
  - `kubectl get deployments --show-labels` will list out deployments along with label details[Though I have given my selector label app=nginx, its not showing. But describe deployments does show Labels details. This needs to be looked into]
  - `kubectl get deployments -l app=nginx` 
  - Deployments are highly useful for zero-downtime deployment. By default it uses *Rolling Update* strategy to bring down the older version and spin up the new version of application.

  There are two new optional attributes[initailDelaySeconds and minReadySeconds] can be specified before k8s brings down the older version of pod
    - initialDelaySeconds: Number of seconds after the container has started before liveness/readiness probes are initiated
    - minReadySeconds: Number of seconds for which a newly created container should accept traffic/Ready for Rolling Update process.

  Difference between initialDelaySeconds(ReadinessProbe) and minReadySeconds(RollingUpdate) property:
    *Lets say container in the pod has started `t` seconds. Readiness proble will be initiated at `t+initialDelaySeconds` seconds. Assume pod become ready at t1 seconds( `t1 > t+initialDelaySeconds`). So this pod will be available after `t1+minReadySeconds` seconds to receive traffic OR considered ready for rolling update process.          
