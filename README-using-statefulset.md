# Kubernetes Redis Cluster

### Create NFS storages


### Create Persistent Volumes

```
kubectl create -f persistentvolume
```

### Create Persistent Volumes Claims

```
kubectl create -f persistentvolumeclaims
```

### Create Redis Cluster Configuration

```
kubectl create configmap redis-conf --from-file=redis.conf
```

### Create Redis Services

```
kubectl create -f statefulset-services/redis-headless.yaml
```

### Create Redis Statefulset

```
kubectl create -f statefulset/redis-statefulset.yaml
```

### Connect Nodes

```
kubectl run -i --tty ubuntu --image=ubuntu \
  --restart=Never /bin/bash
```

```
apt-get update
apt-get install -y vim wget python2.7 python-pip redis-tools dnsutils
```

*Note:* `redis-trib` doesn't support hostnames (see [this issue](https://github.com/antirez/redis/issues/2565)), so we use `dig` to resolve our cluster IPs.

```
pip install redis-trib
```

```
redis-trib.py create \
  `dig +short redis-app-0.redis-service.default.svc.cluster.local`:6379 \
  `dig +short redis-app-1.redis-service.default.svc.cluster.local`:6379 \
  `dig +short redis-app-2.redis-service.default.svc.cluster.local`:6379

redis-trib.py replicate \
  --master-addr `dig +short redis-app-0.redis-service.default.svc.cluster.local`:6379 \
  --slave-addr `dig +short redis-app-3.redis-service.default.svc.cluster.local`:6379
redis-trib.py replicate \
  --master-addr `dig +short redis-app-1.redis-service.default.svc.cluster.local`:6379 \
  --slave-addr `dig +short redis-app-4.redis-service.default.svc.cluster.local`:6379
redis-trib.py replicate \
  --master-addr `dig +short redis-app-2.redis-service.default.svc.cluster.local`:6379 \
  --slave-addr `dig +short redis-app-5.redis-service.default.svc.cluster.local`:6379
```

### Accessing redis cli

Connect to any redis pod
```
kubectl exec -it <podName> -- /bin/sh
```
Access cli
```
/usr/local/bin/redis-cli -c -p 6379
```
To check cluster nodes
```
/usr/local/bin/redis-cli -p 6379 cluster nodes
```

# Access redis cluster externally.
Make sure your nginx ingress controller has started with args `tcp-services-configmap`.
If not ,modify it,add this arg.
```
--tcp-services-configmap=$(POD_NAMESPACE)/nginx-tcp-ingress-configmap
```

Create TCP ingress ConfigMap

```
$ kubectl apply -f ingress/redis-cluster-configmap.yaml
```

Now you can access your redis cluster  with the IP nginx ingress running  and port 9379.

### Contribs

```
Originally contributed by Kelsey Hightower (https://github.com/kelseyhightower/kubernetes-redis-cluster)
```

### References
https://github.com/sanderploegsma/redis-cluster/blob/master/redis-cluster.yml

http://blog.opskumu.com/k8s-ingress.html
