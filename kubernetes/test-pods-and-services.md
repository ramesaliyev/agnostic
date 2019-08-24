[Home](../README.md)

# Test PODs and Services

First be sure you're in the kubernetes `namespace` you want. In this example we are in the `kesim` namespace. Check [cheatsheet](../cheatsheets/k8s.md) to create and switch between namespaces.

Create temporary folder under home directory to save yaml files.

    mkdir -p ~/k8s-test-setup

Go to directory

    cd ~/k8s-test-setup

## First test deployment

Create `deployment.yaml` with following content;

    kind: Deployment
    apiVersion: extensions/v1beta1
    metadata:
      name: service-test
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: service_test_pod
      template:
        metadata:
          labels:
            app: service_test_pod
        spec:
          containers:
          - name: simple-http
            image: python:2.7
            imagePullPolicy: IfNotPresent
            command: ["/bin/bash"]
            args: ["-c", "echo \"<p>Hello from $(hostname)</p>\" > index.html; python -m SimpleHTTPServer 8080"]
            ports:
            - name: http
              containerPort: 8080

This deployment will create our PODs with 2 replicas.

Apply the deployment;

    kubectl apply -f deployment.yaml

Check if pods are created

    kubectl get pods

$>

    NAME                            READY   STATUS              RESTARTS   AGE
    service-test-85b6644b4d-95ds7   0/1     ContainerCreating   0          7s
    service-test-85b6644b4d-ckd5q   0/1     ContainerCreating   0          7s

Status is `ContainerCreating` because kubernetes is fetching the image for the first time. After some time check the pods again.

    kubectl get pods

$>

    NAME                            READY   STATUS    RESTARTS   AGE
    service-test-85b6644b4d-95ds7   1/1     Running   0          101s
    service-test-85b6644b4d-ckd5q   1/1     Running   0          101s

As you can see PODs are running now.

You can also check pods with `-o wide` flag.

    kubectl get pods -o wide

$>

    NAME                            READY   STATUS    RESTARTS   AGE     IP               NODE
    service-test-85b6644b4d-95ds7   1/1     Running   0          2m31s   10.202.217.4     ra-vm2-master-node
    service-test-85b6644b4d-ckd5q   1/1     Running   0          2m31s   10.202.208.193   ra-vm3-slave-node1

Observe that one of our POD is on `ra-vm2-master-node` and the other is on `ra-vm3-slave-node1` VM. And we can see PODs private IPs.

We can also use query to see what their pod network addresses are:

    kubectl get pods --selector=app=service_test_pod -o jsonpath='{.items[*].status.podIP}'

$>

    10.202.217.4 10.202.208.193

We can demonstrate that the pod network is operating by creating a simple client pod to make a request, and then viewing the output.

Create `client.yaml` with following content;

    apiVersion: v1
    kind: Pod
    metadata:
      name: service-test-client1
    spec:
      restartPolicy: Never
      containers:
      - name: test-client1
        image: alpine
        command: ["/bin/sh"]
        args: ["-c", "echo 'GET / HTTP/1.1\r\n\r\n' | nc 10.202.217.4 8080"]

Apply the POD;

    kubectl apply -f client.yaml

After this pod is created the command will run to completion, the pod will enter the “completed” state;

    kubectl get po

$>

    NAME                            READY   STATUS      RESTARTS   AGE
    service-test-85b6644b4d-95ds7   1/1     Running     0          7m9s
    service-test-85b6644b4d-ckd5q   1/1     Running     0          7m9s
    service-test-client1            0/1     Completed   0          27s

 The output can then be retrieved with;

    kubectl logs service-test-client1

$>

    HTTP/1.0 200 OK
    Server: SimpleHTTP/0.6 Python/2.7.16
    Date: Sun, 25 Aug 2019 16:36:40 GMT
    Content-type: text/html
    Content-Length: 48
    Last-Modified: Sun, 25 Aug 2019 16:30:33 GMT

    <p>Hello from service-test-85b6644b4d-95ds7</p>

You can also get into one container and curl to see response;

    kubectl exec -it service-test-85b6644b4d-95ds7 -- /bin/bash

And inside the container do;

    root@service-test-85b6644b4d-95ds7:/# curl localhost:8080
    <p>Hello from service-test-85b6644b4d-95ds7</p>

    root@service-test-85b6644b4d-95ds7:/# curl 10.202.217.4:8080
    <p>Hello from service-test-85b6644b4d-95ds7</p>

    root@service-test-85b6644b4d-95ds7:/# curl 10.202.208.193:8080
    <p>Hello from service-test-85b6644b4d-ckd5q</p>

Nothing in this example shows which node the client pod was created on, but regardless of where it ran in the cluster it would be able to reach the server pod and get a response back thanks to the pod network. However if the server pod were to die and be restarted, or be rescheduled to a different node, its IP would almost certainly change and the client would break. We avoid this by creating a service.

## First test service

A service is a type of kubernetes resource that causes a proxy to be configured to forward requests to a set of pods. The set of pods that will receive traffic is determined by the selector, which matches labels assigned to the pods when they were created. Once the service is created we can see that it has been assigned an IP address and will accept requests on port 80.

Create `service.yaml` with following content;

    kind: Service
    apiVersion: v1
    metadata:
      name: service-test
    spec:
      selector:
        app: service_test_pod
      ports:
      - port: 80
        targetPort: http

Apply the Service;

    kubectl apply -f service.yaml

Get service

    kubectl get service service-test

$>

    NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    service-test   ClusterIP   10.101.142.13   <none>        80/TCP    17s

You can request this IP from an POD and it will load balance requests between pods;

Exec into one pod;

    kubectl exec -it service-test-85b6644b4d-95ds7 -- /bin/bash

Make a curl to service IP.

    root@service-test-85b6644b4d-95ds7:/# curl 10.101.142.13
    <p>Hello from service-test-85b6644b4d-95ds7</p>

    root@service-test-85b6644b4d-95ds7:/# curl 10.101.142.13
    <p>Hello from service-test-85b6644b4d-95ds7</p>

    root@service-test-85b6644b4d-95ds7:/# curl 10.101.142.13
    <p>Hello from service-test-85b6644b4d-ckd5q</p>

Requests can be sent to the service IP directly but it would be better to use a hostname that resolves to the IP address. Fortunately kubernetes provides an internal cluster DNS that resolves the service name.

Try to make curl to `service-test` within the same pod;

    root@service-test-85b6644b4d-95ds7:/# curl service-test
    <p>Hello from service-test-85b6644b4d-ckd5q</p>

    root@service-test-85b6644b4d-95ds7:/# curl service-test
    <p>Hello from service-test-85b6644b4d-95ds7</p>

    root@service-test-85b6644b4d-95ds7:/# curl service-test
    <p>Hello from service-test-85b6644b4d-ckd5q</p>

And with a slight change to the `client.yaml` we can make use of it. Change IP with the `service-test`;

    // change this

    args: ["-c", "echo 'GET / HTTP/1.1\r\n\r\n' | nc 10.202.217.4 8080"]

    // with this

    args: ["-c", "echo 'GET / HTTP/1.1\r\n\r\n' | nc service-test 80"]

Remove and re-apply the client pod.

    kubectl delete po service-test-client1
    kubectl apply -f client.yaml

See the logs;

    HTTP/1.0 200 OK
    Server: SimpleHTTP/0.6 Python/2.7.16
    Date: Sun, 25 Aug 2019 18:27:49 GMT
    Content-type: text/html
    Content-Length: 48
    Last-Modified: Sun, 25 Aug 2019 16:30:30 GMT

    <p>Hello from service-test-85b6644b4d-ckd5q</p>

Now our service and worker pods are working!

## First test NodePort

You can create a `NodePort Service` to make pods available to public;

Create `nodeport.yaml` with following content;

    kind: Service
    apiVersion: v1
    metadata:
      name: service-test-nodeport
    spec:
      type: NodePort
      selector:
        app: service_test_pod
      ports:
      - port: 80
        targetPort: http

Apply `nodeport` service.

    kubectl apply -f nodeport.yaml

This will create a `NodePort Service` and now ours pods reachable at the IP address of the node as well as at the assigned cluster IP on the services network.

First see service;

    kubectl get svc

$>

    NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    service-test            ClusterIP   10.101.142.13   <none>        80/TCP         64m
    service-test-nodeport   NodePort    10.104.71.31    <none>        80:30764/TCP   27s

Observe IP and PORT of our `NodePort`.

Now you can do a couple of things;

    // Reach through IP address of the node.
    curl 10.0.0.3:30764
    curl 10.0.0.4:30764

    // On nodes; reach through localhost
    curl localhost:30764

    // On pods; reach by using node port name
    curl service-test-nodeport

    // Reach from public by public IP of nodes;
    ravm2.com:30764
    ravm3.com:30764

Use NodePorts for debugging purposes.

Next step is [Ingress setup](./ingress-setup.md).