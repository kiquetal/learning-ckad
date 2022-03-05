#### Health probing

    Kubernetes provids a concept called health probing

- Readiness Probe
  - Even after an application has been started up, it may still need to execute
    configuration procedures- for example, connecting to a database and preparing data.
- Liveness probe
  - Once the application is running, we'll want to make sure that it still works as expected
    without issues
- Startup probe
  - Legacy applications in particular can take a long time to start up, we're talking minutes
    sometimes. This probe can be instantiated to waitfor a predefined amount of time before 
    a liveness probe is allowed to start probing.
  


#### ReadinesProbe
| Method | Option | Description|
| ------ | ------ | ---------- |
|Custom method        | exec.commnand        | Execute a command inside of the container            |
| HTTP GET            | httpGet              | Sends an HTTP GET to an endpoint |
| TCP socket connection| tcpSocket           | Tries to open a tcp socket connection to a port|


Every probe offers a set of attributes that can further configure the runtime behavior, as shown,

|Attribute | Default Value | Descrition |
| -------- | ------------- | --------- |
| initialDelaySeconds | 0 | Delay in seconds until first checks is executed| 
| periodSeconds | 10 | Interval for executing a check |
| timeoutSeconds| 1 | Maximum number of seconds until check operation times out | 
| successThreshold | 1 | Number of successful check attempts until p[robe is considered successful after a failure|
| failuteThreshold | 3 | Number of failures for check attempts before probe is marked failed and takes action|



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-pod
spec:
  containers:
  - image: bmuschko/nodejs-hello-world:1.0.0
    name: hello-world
    ports:
    - name: nodejs-port
      containerPort: 3000
    readinessProbe:
      httpGet:
        path: /
        port: nodejs-port
      initialDelaySeconds: 2
      periodSeconds: 8
```

    $ kubectl create -f readiness-probe.yaml
    pod/readiness-pod created
    $ kubectl get pod readiness-pod
    NAME                READY   STATUS    RESTARTS   AGE
    pod/readiness-pod   0/1     Running   0          6s
    $ kubectl get pod readiness-pod
    NAME                READY   STATUS    RESTARTS   AGE
    pod/readiness-pod   1/1     Running   0          68s
    $ kubectl describe pod readiness-pod
    ...
    Containers:
    hello-world:
    ...
    Readiness:      http-get http://:nodejs-port/ delay=2s timeout=1s \
                    period=8s #success=1 #failure=3
    ...

#### LivenessProbe
    
    A liveness probe chekcs if the application is still working as expected down the road.
    For the purpose of demostrating a liveness probe, we'll use a custom command.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-pod
spec:
  containers:
  - image: busybox
    name: app
    args:
    - /bin/sh
    - -c
    - 'while true; do touch /tmp/heartbeat.txt; sleep 5; done;'
    livenessProbe:
      exec:
        command:
        - test `find /tmp/heartbeat.txt -mmin -1`
      initialDelaySeconds: 5
      periodSeconds: 30
```

    $ kubectl create -f liveness-probe.yaml
    pod/liveness-pod created
    $ kubectl get pod liveness-pod
    NAME               READY   STATUS    RESTARTS   AGE
    pod/liveness-pod   1/1     Running   0          22s
    $ kubectl describe pod liveness-pod
    ...
    Containers:
    app:
    ...
    Restart Count:  0
    Liveness:       exec [test `find /tmp/heartbeat.txt -mmin -1`] delay=5s \
    timeout=1s period=30s #success=1 #failure=3
    ...


##### Pod Error causes

   | Status    | Root Cause     | Potential fix |
   | --------- | -------------- | ------------- |
   | ImagePullBackOff | Image could not be pulled from registry | Check correct image name, check that image name exists in registry, verify network access from node to registry |
   | CrashLoopBackOff | Application or command run in container crashes | Check command executed in container, ensure that image can properly execute|
   | CreateContainerConfigError | ConfigMap or Secret refere3nce by container cannot be found | Check correct name of the configuration object verify the exitence of the configuration object in the namespace|

##### Commands to inspect

    kubectl describe pod secret-pod
    kubectl get pods failing-pod
    kubectl logs failing-pod
    kubectl exec failing-pod -it -- /bin/sh
    kubectl get endpoints myservice
    kubectl get pods --show-labels
    kubectl describe service myservice
    kubectl get service myapp -o yaml | grep targetPort:
        targetPort: 80
    kubectl get pods myapp-68bf896d89-qfhlv -o yaml | grep containerPort:
      - containerPort: 80

