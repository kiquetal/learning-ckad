#### Kubernetes primitives

    Are the basic building blocks anchored in the k8s architecture for creating
    and operating an application on the platform. 

#### Analogy with oriented-object programming

    A kubernetes primitive is the equivalent of a class.
    THe instance of a class in object-oriented programming is the
    equivalent of a class. The instance of a class in object-oriented programming
    is an object,managing its owen state and having the ability
    to communicate with other parts of the system. Whenever you create
    a K8s objerct, you produce such an instance.

![](./images/k8sobject.png)

- Api version:

      Defines the structure of a primitive and uses it to validate
      correctness.

- Kind: 
   
      What type of object are we going to create?

- Metadata:
    
      Metadata describes higher-level information about the object
      its name, what namepsace it lives on, or whateverit defines
      labels and annotations

- Spec:

       The specification(spec for short) declares the desired states,
        how should this object look after it has been created?

- Status:

        The status describes the actual state of an object.
        The k8s cntrollers and their reconciliation loops
        constantly try to transition a k8s objects from the desired
        state into the actual state.


#### Imperative vs declarative approach

  Declarative approach creates the object with a yaml file.
  
  Hybrid approach
  To create a manifest file you should run it with
  --dry-run=client
  
    kubectl run frontend --image=nginx --restart=Never --port=80
    -o yaml --dry-run= client > pod.yaml

  Delete a object
  
    kubectl delete pod frontend
    kubectl delete -f pod.yaml

  Editing a live object
  
     kubectl edit pod frontend

  Replacing
  
     kubectl replace -f pod.yaml

  Updating a live object

   The `craate` command instantiates a new object.
   Trying to execute the create command for an existing object will produce
   an error.
   
    kubectl apply -f pod.yaml

  
#### Lifecycle PODs

|Option| Desscription|
|------|-------------|
| `PENDING` | The pod has been accepted by the Kubernetes system, but one or more of the container images has not  been created.
| `Running` | At least one container is still running, or is in the process of starting or restarting.
| `Succeeded` | All containers in the Pod terminated successfully|
| `Failed` | Containers in the POd terminated, at least one failed with an error.|
| `Unknown`  | The state of Pod could not be obtained.|

#### Pod details

  kubectl describe pods hazelcast
  kubectl logs hazelcast

  kubectl describe pods hazelcast | grep Image:

#### Log in the container

  kubectl exec -it hazelcast -- /bin/sh
  
#### Sending commands to pod

  ```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - command: ["/bin/sh"]
    args: ["-c", "while true; do date; sleep 10; done"]
    image: busybox
    name: mypod
  restartPolicy: Never

  ```
 Alternative to could create via the imperative approach.

  ```bash
kubectl run mypod --image=busybox -o yaml --dry-run=client --restart=Never \
  > pod.yaml -- /bin/sh -c "while true; do date; sleep 10; done"
  ```

#### Namespace

  Usefull to isolate applications
  
    kubectl create namespace code-red
  
    kubectl run pod --image=nginx --restart=Never -n code-red 
    kubectl get pods -n code-red
