#### Simple Configuration

- ConfigMap
- Secret
- Volume
- SecurityContenxt
- Resource Boundaries
- ResourceQuota
- Service Account


#### Difference between ConfigMap and Secret

   Secrets and configMap are almost the same, but Secret stores
   their values in base64,but does not **encrypy** it.



 kubectl create configmap db-config --from-literal=db=staging
 kubectl create configmap db-config --from-env-file=config.env
 kubectl create configmap db-config --from-file=config.txt


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backend-config
data:
  database_url: jdbc:postgresql://localhost/test
  user: fred
```
Consuming configMap

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    envFrom:
    - configMapRef:
        name: backend-config
```  
See content of pod

kubectl exec configured-pod -- env

Mounting a ConfigMap as Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: backend-config
```

