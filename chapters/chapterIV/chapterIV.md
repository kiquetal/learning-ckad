#### Notes

    Why you would run several container within a pod?

    For a init container or
    to provide helper functionality, containers runninger helpers are called sidecars

#####  Init Containers
    
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: business-app
spec:
  initContainers:
  - name: configurer
    image: busybox:1.32.0
    command: ['sh', '-c', 'echo Configuring application... && \
              mkdir -p /usr/shared/app && echo -e "{\"dbConfig\": \
              {\"host\":\"localhost\",\"port\":5432,\"dbName\":\"customers\"}}" \
              > /usr/shared/app/config.json']
    volumeMounts:
    - name: configdir
      mountPath: "/usr/shared/app"
  containers:
  - image: bmuschko/nodejs-read-config:1.0.0
    name: web
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: configdir
      mountPath: "/usr/shared/app"
  volumes:
  - name: configdir
    emptyDir: {}
```

#### The sidecar pattern

    Usually used for monitoring task or watcher capabilities. The sidecars
    are not part of the main traffic or API of the primary application. They usually
    operate asynchronouly and are not involved in the public API.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webserver
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - name: logs-vol
      mountPath: /var/log/nginx
  - name: sidecar
    image: busybox
    command: ["sh","-c","while true; do if [ \"$(cat /var/log/nginx/error.log \
              | grep 'error')\" != \"\" ]; then echo 'Error discovered!'; fi; \
              sleep 10; done"]
    volumeMounts:
    - name: logs-vol
      mountPath: /var/log/nginx
  volumes:
  - name: logs-vol
    emptyDir: {}
```

#### The adapter pattern

    A sidecar container executes transformation logic that turns the log entries
    into the format needed by the external system without having to change application
    logic.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - 'while true; do echo "$(date) | $(du -sh ~)" >> /var/logs/diskspace.txt; \
       sleep 5; done;'
    image: busybox
    name: app
    volumeMounts:
      - name: config-volume
        mountPath: /var/logs
  - image: busybox
    name: transformer
    args:
    - /bin/sh
    - -c
    - 'sleep 20; while true; do while read LINE; do echo "$LINE" | cut -f2 -d"|" \
       >> $(date +%Y-%m-%d-%H-%M-%S)-transformed.txt; done < \
       /var/logs/diskspace.txt; sleep 20; done;'
    volumeMounts:
    - name: config-volume
      mountPath: /var/logs
  volumes:
  - name: config-volume
    emptyDir: {}
````

    $ kubectl logs business-app -c configurer

    $ kubectl create -f adapter.yaml
    pod/adapter created
    $ kubectl get pods adapter
    NAME      READY   STATUS    RESTARTS   AGE
    adapter   2/2     Running   0          10s
    $ kubectl exec adapter --container=transformer -it -- /bin/sh
    / # cat /var/logs/diskspace.txt
    Sun Jul 19 20:28:07 UTC 2020 | 4.0K	/root
    Sun Jul 19 20:28:12 UTC 2020 | 4.0K	/root
    / # ls -l
    total 40
    -rw-r--r--  1  root  root  60 Jul 19 20:28 2020-07-19-20-28-28-transformed.txt
    ...
    / # cat 2020-07-19-20-28-28-transformed.txt
    4.0K	/root
    4.0K	/root

#### Ambassador pattern

    The ambassador pattern provides a proxy for communicating with externalservices
    The overarchin goal is to hide and/or abstract the complexity of interacting with other
    parts of the system. 
    Responsabilities like retry logic, security concern like providing authentication or
    authorization.

    Any calls made from the bussiness application need to be funneled through the ambassador
    container.


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rate-limiter
spec:
  containers:
  - name: business-app
    image: bmuschko/nodejs-business-app:1.0.0
    ports:
    - containerPort: 8080
  - name: ambassador
    image: bmuschko/nodejs-ambassador:1.0.0
    ports:
    - containerPort: 8081
```
    Every container running inside of the same pod, can communicate via localhost by default.


    $ kubectl create -f ambassador.yaml
    pod/rate-limiter created

    $ kubectl get pods rate-limiter
    NAME           READY   STATUS    RESTARTS   AGE
    rate-limiter   2/2     Running   0          5s

    $ kubectl exec rate-limiter -it -c business-app -- /bin/sh

    # curl localhost:8080/test
    {"args":{"test":"123"},"headers":{"x-forwarded-proto":"https", \
    "x-forwarded-port":"443","host":"postman-echo.com", \
    "x-amzn-trace-id":"Root=1-5f177dba-e736991e882d12fcffd23f34"}, \
    "url":"https://postman-echo.com/get?test=123"}
    ...

    # curl localhost:8080/test
    Too many requests have been made from this IP, please try again after an hour
    

