##### Pod Design

    Labels are an essential tool for querying, filtering and sorting
    Kubernetes objects. Annotations only represent descriptive metada for kubernetes
    objects but have no ability to be used for queries.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: labeled-pod
  labels:
    env: dev
    tier: backend
spec:
  containers:
  - image: nginx
    name: nginx
```
    kubectl run labeled-pod --image=nginx \
    --restart=Never --labels= tier=beckend, env=dev


kubectl describe pod labeled-pod | grep -C 2 Labels:
kubectl get pod labeled-pod -o yaml | grep -C 1 labels:

kubectl get pods -l 'team in (shiny, legacy)' --show-labels
kubectl get pods -l env=prod --show-labels

#### Annotations

    Annotations are declared similarly to labels, but they serve a differente purpose.
    They represent key-value pairs for providing descriptive metadata. They cannot be used
    for querying.

    Inspecting annotations
    
    kubectl describe pod annotated-pod | grep -C 2 Annotations:

    kubectl get pod annotated-pod -o yaml | grep -C 3 annotations:
    
##### Modifying annotations

    kubectl annotate pod annotated-pod oncall='800-555-1212'

#### Deployments

The selector `spec.selector.matchLabels`matches on the key-value pair
app=my-deploy with the label defined under the `template`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deploy
  labels:
    app: my-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-deploy
  template:
    metadata:
      labels:
        app: my-deploy
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
```

#### Rendering deployments details
kubectl get deployments,pods,replicasets    
kubectl describe deployment.apps/my-deploy
kubectl describe replicaset.apps/my-deploy-8448c488b5
kubectl describe pod/my-deploy-8448c488b5-mzx5g

#### Rolling out a new revision
kubectl set image deployment my-deploy nginx=nginx:1.19.2
kubectl rollout history deployment my-deploy    
kubectl rollout history deployments my-deploy --revision=2
#### Revert revision
kubectl rollout undo deployment my-deploy --to-revision=1

#### Manually scaling a deployment

kubectl scale deployment my-deploy --replicas=5
kubectl describe deployment.apps/my-deploy

Horizontal Pod Autoscaler (HPA)
Scales the number of Pod replicas based on CPU and memory thresholds.

Vertical Pod Autoscaler (VPA)
Scales the CPU and memory allocation for existing Pods based on historic metrics.

kubectl autoscale deployment my-deploy --cpu-percent=70 --min=2 --max=8
