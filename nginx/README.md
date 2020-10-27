# NGINX ingress controller deployment via a helm chart on Platform9 Managed Kubernetes
 
Here we are going to deploy the well known NGINX ingress controller on top of the Platform9 Managed Kubernetes freedom tier via NGINX helm chart. This deployment is going to require a PMK/FT 4.4+ cluster (Kubernetes 1.17 or above) with metallb loadbalancer on baremetal servers/VMs running on Ubuntu 18.04 or Centos 7.6.

# Prerequisites:
Helmv3 preinstalled and kubernetes context preconfigured on the VM.


Deployment
Add the Helm repo for NGINX ingress controller
```bash
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
"ingress-nginx" has been added to your repositories
```

Update the repo cache
```bash
$ helm repo update and list the latest version of the nginx helm chart
...Successfully got an update from the "ingress-nginx" chart repository
Update Complete. ⎈Happy Helming!⎈

$ helm search repo ingress-nginx/ingress-nginx
NAME                        CHART VERSION APP VERSION DESCRIPTION
ingress-nginx/ingress-nginx 3.7.1         0.40.2      Ingress controller for Kubernetes using NGINX a...
```

Install the latest helm chart for NGINX on the cluster.

```bash
$ helm show chart ingress-nginx/ingress-nginx
apiVersion: v1
appVersion: 0.40.2
description: Ingress controller for Kubernetes using NGINX as a reverse proxy and
  load balancer
home: https://github.com/kubernetes/ingress-nginx
icon: https://upload.wikimedia.org/wikipedia/commons/thumb/c/c5/Nginx_logo.svg/500px-Nginx_logo.svg.png
keywords:
- ingress
- nginx
kubeVersion: '>=1.16.0-0'
maintainers:
- name: ChiefAlexander
name: ingress-nginx
sources:
- https://github.com/kubernetes/ingress-nginx
version: 3.7.1
```

Install the chart
```bash
$ kubectl create ns ingress-nginx ; helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx
namespace/ingress-nginx created
NAME: ingress-nginx
LAST DEPLOYED: Thu Oct 22 14:10:07 2020
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller'

An example Ingress that makes use of the controller:

  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```

Validate the ingress controller pods are running. Notice that NGINX ingress has got the load balancer IP from metallb.
```bash
$ kubectl get ns ingress-nginx
NAME            STATUS   AGE
ingress-nginx   Active   80s

$ kubectl get all -n ingress-nginx
NAME                                            READY   STATUS    RESTARTS   AGE
pod/ingress-nginx-controller-74cd44464c-wnvp8   1/1     Running   0          87s

NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.21.191.58   10.128.231.217   80:30265/TCP,443:31087/TCP   87s
service/ingress-nginx-controller-admission   ClusterIP      10.21.73.114   <none>           443/TCP                      87s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           87s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-74cd44464c   1         1         1       87s
```
Clone the cicd repo to test the ingress controller with a nodejs based pf9reactapp.

```bash
$ git clone https://github.com/KoolKubernetes/cicd.git
```

Deploy the app with ingress resource to leverage NGINX ingress controller.
```bash
$ kubectl apply -f cicd/jenkins/webapp01/k8s/ingress-nginx-app.yaml
namespace/p9-react-app created
deployment.apps/p9-react-app created
service/p9-react-app created
ingress.networking.k8s.io/pf9app-routing created
```

Validate the app is running in p9-react-app namespace with an ingress resource.

```bash
$ kubectl get all -n p9-react-app
NAME                                READY   STATUS    RESTARTS   AGE
pod/p9-react-app-67d4df6bb6-5f944   1/1     Running   0          17s
pod/p9-react-app-67d4df6bb6-t8p45   1/1     Running   0          17s


NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/p9-react-app   ClusterIP   10.21.39.141   <none>        80/TCP    18s


NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/p9-react-app   2/2     2            2           18s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/p9-react-app-67d4df6bb6   2         2         2       18s

$ kubectl get ingress -n p9-react-app
NAME             HOSTS                    ADDRESS         PORTS   AGE
pf9app-routing   pf9app.platform9.horse   10.128.229.21   80      40m
```

The ingress resource gets created as shown below.
```bash
$ kubectl describe ingress -n p9-react-app
Name:             pf9app-routing
Namespace:        p9-react-app
Address:          10.128.229.21
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                    Path  Backends
  ----                    ----  --------
  pf9app.platform9.horse
                          /   p9-react-app:80 (10.20.20.22:80,10.20.96.22:80)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"networking.k8s.io/v1beta1","kind":"Ingress","metadata":{"annotations":{},"name":"pf9app-routing","namespace":"p9-react-app"},"spec":{"rules":[{"host":"pf9app.platform9.horse","http":{"paths":[{"backend":{"serviceName":"p9-react-app","servicePort":80},"path":"/"}]}}]}}

Events:
  Type    Reason  Age                  From                      Message
  ----    ------  ----                 ----                      -------
  Normal  CREATE  45m                  nginx-ingress-controller  Ingress p9-react-app/pf9app-routing

```

The Service p9-react-app within Kubernetes namespace p9-react-app is now exposed outside via hostname pf9app.platform9.horse. A DNS entry for this hostname with the ingress controller loadbalancer IP address will get you to the app in the browser. Modify the Manifest to set the hostname of your choice before deploying the app manifest on the cluster.


![add-cred-dhub](https://github.com/KoolKubernetes/ingress/blob/master/nginx/images/app-ingress.png)


Ingress controller should be running on a node which has enough resources. It can be further configured to add authentication, TLS termination so on and so forth. A wild card DNS hostname is used for resolving all the host urls for different microservices that are exposed via ingress controller.



# Reference

https://kubernetes.github.io/ingress-nginx/

https://kubernetes.github.io/ingress-nginx/deploy/#using-helm
