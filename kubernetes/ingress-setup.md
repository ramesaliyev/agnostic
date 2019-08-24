[Home](../README.md)

# Ingress Setup

## Default Backend Preparation

First deploy a `deployment` and `service` for your future `default-backend`;

Make sure you're on the `default` namespace. (See [cheatsheet](../cheatsheets/k8s.md).)

First create yaml files;

    mkdir -p ~/k8s-default-backend
    touch ~/k8s-default-backend/deployment.yaml
    touch ~/k8s-default-backend/service.yaml

Insert into files;

`deployment.yaml`

    kind: Deployment
    apiVersion: extensions/v1beta1
    metadata:
      name: default-backend
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: default_backend
      template:
        metadata:
          labels:
            app: default_backend
        spec:
          containers:
          - name: default-backend-http
            image: ramesaliyev/default-backend:1.0.0
            imagePullPolicy: IfNotPresent
            ports:
            - name: http
              containerPort: 8080

`service.yaml`

    kind: Service
    apiVersion: v1
    metadata:
      name: default-backend-service
    spec:
      selector:
        app: default_backend
      ports:
      - port: 80
        targetPort: http

Now apply both of them;

    kubectl apply -f ~/k8s-default-backend/deployment.yaml
    kubectl apply -f ~/k8s-default-backend/service.yaml

## Setup the Nginx Ingress

Download two `ingress-nginx` config yaml; `mandatory.yaml` and `service-nodeport.yaml`

    mkdir -p $HOME/k8s-ingress-setup
    wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml -O $HOME/k8s-ingress-setup/mandatory.yaml
    wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml -O $HOME/k8s-ingress-setup/service-nodeport.yaml

Open `mandatory.yaml`;

    vi $HOME/k8s-ingress-setup/mandatory.yaml

Find `nginx-ingress-controller` args and add `default-backend-service` argument. For now set it to `default/default-backend-service`

    . . .
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.1
          args:
            - /nginx-ingress-controller
            - --default-backend-service=default/default-backend-service
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
    . . .

Save and exit.

Open `service-nodeport.yaml`;

    vi $HOME/k8s-ingress-setup/service-nodeport.yaml

Add `externalIPs` section under `spec`, specify your `master-node` public IP.

    spec:
      externalIPs:
        - 116.xxx.xxx.xxx
      type: NodePort

Save and exit.

Now apply both of configs.

    kubectl apply -f $HOME/k8s-ingress-setup/mandatory.yaml
    kubectl apply -f $HOME/k8s-ingress-setup/service-nodeport.yaml

$>

    namespace/ingress-nginx created
    configmap/nginx-configuration created
    configmap/tcp-services created
    configmap/udp-services created
    serviceaccount/nginx-ingress-serviceaccount created
    clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
    role.rbac.authorization.k8s.io/nginx-ingress-role created
    rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
    clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
    deployment.apps/nginx-ingress-controller created

$>

    service/ingress-nginx created

Copy these yaml files to your local machine if you want;

    scp ra@ra-vm2:~/k8s-ingress-setup/mandatory.yaml ~/tmp
    scp ra@ra-vm2:~/k8s-ingress-setup/service-nodeport.yaml ~/tmp

Now if you try to reach your machine via IP or a hostname that it doesnt recognize, default backend will respond.

Try to navigate `ravm2.com` in your browser.

## Applying Ingress Rules

CONTINUE FROM HERE.