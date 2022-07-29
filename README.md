# Knative on EKS

---


```
EKS - 1.22
CNI - v1.10.1-eksbuild.1
aws-load-balancer-controller - v2.4.1 (helm chart - 1.4.1)

---

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"23", GitVersion:"v1.23.0", GitCommit:"ab69524f795c42094a6630298ff53f3c3ebab7f4", GitTreeState:"clean", BuildDate:"2021-12-07T18:16:20Z", GoVersion:"go1.17.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"22+", GitVersion:"v1.22.10-eks-84b4fe6", GitCommit:"cc6a1b4915a99f49f5510ef0667f94b9ca832a8a", GitTreeState:"clean", BuildDate:"2022-06-09T18:24:04Z", GoVersion:"go1.16.15", Compiler:"gc", Platform:"linux/amd64"}

$ istioctl version
no running Istio pods in "istio-system"
1.14.2

$ kn version
Version:      v20220713-local-bfdc0a21
Build Date:   2022-07-13 09:04:48
Git Revision: bfdc0a21
Supported APIs:
* Serving
  - serving.knative.dev/v1 (knative-serving v0.33.0)
* Eventing
  - sources.knative.dev/v1 (knative-eventing v0.33.0)
  - eventing.knative.dev/v1 (knative-eventing v0.33.0)
```

## Install the Knative Serving component

```
$ kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.6.0/serving-crds.yaml
customresourcedefinition.apiextensions.k8s.io/certificates.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/configurations.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/clusterdomainclaims.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/domainmappings.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/ingresses.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/metrics.autoscaling.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/podautoscalers.autoscaling.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/revisions.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/routes.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/serverlessservices.networking.internal.knative.dev created
customresourcedefinition.apiextensions.k8s.io/services.serving.knative.dev created
customresourcedefinition.apiextensions.k8s.io/images.caching.internal.knative.dev created


$ kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.6.0/serving-core.yaml
namespace/knative-serving created
clusterrole.rbac.authorization.k8s.io/knative-serving-aggregated-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/knative-serving-addressable-resolver created
clusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-admin created
clusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-edit created
clusterrole.rbac.authorization.k8s.io/knative-serving-namespaced-view created
clusterrole.rbac.authorization.k8s.io/knative-serving-core created
clusterrole.rbac.authorization.k8s.io/knative-serving-podspecable-binding created
serviceaccount/controller created
clusterrole.rbac.authorization.k8s.io/knative-serving-admin created
clusterrolebinding.rbac.authorization.k8s.io/knative-serving-controller-admin created
clusterrolebinding.rbac.authorization.k8s.io/knative-serving-controller-addressable-resolver created
customresourcedefinition.apiextensions.k8s.io/images.caching.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/certificates.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/configurations.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/clusterdomainclaims.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/domainmappings.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/ingresses.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/metrics.autoscaling.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/podautoscalers.autoscaling.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/revisions.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/routes.serving.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/serverlessservices.networking.internal.knative.dev unchanged
customresourcedefinition.apiextensions.k8s.io/services.serving.knative.dev unchanged
secret/serving-certs-ctrl-ca created
secret/knative-serving-certs created
image.caching.internal.knative.dev/queue-proxy created
configmap/config-autoscaler created
configmap/config-defaults created
configmap/config-deployment created
configmap/config-domain created
configmap/config-features created
configmap/config-gc created
configmap/config-leader-election created
configmap/config-logging created
configmap/config-network created
configmap/config-observability created
configmap/config-tracing created
horizontalpodautoscaler.autoscaling/activator created
poddisruptionbudget.policy/activator-pdb created
deployment.apps/activator created
service/activator-service created
deployment.apps/autoscaler created
service/autoscaler created
deployment.apps/controller created
service/controller created
deployment.apps/domain-mapping created
deployment.apps/domainmapping-webhook created
service/domainmapping-webhook created
horizontalpodautoscaler.autoscaling/webhook created
poddisruptionbudget.policy/webhook-pdb created
deployment.apps/webhook created
service/webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/config.webhook.serving.knative.dev created
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.serving.knative.dev created
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.domainmapping.serving.knative.dev created
secret/domainmapping-webhook-certs created
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.webhook.domainmapping.serving.knative.dev created
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.webhook.serving.knative.dev created
secret/webhook-certs created

```

Verify: The following components are now available on knative-serving namespace. 
```
$ kubectl get all -n knative-serving
NAME                                         READY   STATUS    RESTARTS   AGE
pod/activator-5f9df88fbb-bdfjw               1/1     Running   0          18m
pod/autoscaler-c7d7bd5f-w6mhm                1/1     Running   0          18m
pod/controller-5f64fb5c49-lg86b              1/1     Running   0          18m
pod/domain-mapping-7499d44b7f-wzvlg          1/1     Running   0          18m
pod/domainmapping-webhook-695848595b-dcphs   1/1     Running   0          18m
pod/webhook-79d8bd799d-ddr9l                 1/1     Running   0          18m

NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                   AGE
service/activator-service            ClusterIP   172.20.91.212    <none>        9090/TCP,8008/TCP,80/TCP,81/TCP,443/TCP   18m
service/autoscaler                   ClusterIP   172.20.247.254   <none>        9090/TCP,8008/TCP,8080/TCP                18m
service/autoscaler-bucket-00-of-01   ClusterIP   172.20.67.199    <none>        8080/TCP                                  17m
service/controller                   ClusterIP   172.20.61.172    <none>        9090/TCP,8008/TCP                         18m
service/domainmapping-webhook        ClusterIP   172.20.170.123   <none>        9090/TCP,8008/TCP,443/TCP                 18m
service/webhook                      ClusterIP   172.20.96.104    <none>        9090/TCP,8008/TCP,443/TCP                 18m

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/activator               1/1     1            1           18m
deployment.apps/autoscaler              1/1     1            1           18m
deployment.apps/controller              1/1     1            1           18m
deployment.apps/domain-mapping          1/1     1            1           18m
deployment.apps/domainmapping-webhook   1/1     1            1           18m
deployment.apps/webhook                 1/1     1            1           18m

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/activator-5f9df88fbb               1         1         1       18m
replicaset.apps/autoscaler-c7d7bd5f                1         1         1       18m
replicaset.apps/controller-5f64fb5c49              1         1         1       18m
replicaset.apps/domain-mapping-7499d44b7f          1         1         1       18m
replicaset.apps/domainmapping-webhook-695848595b   1         1         1       18m
replicaset.apps/webhook-79d8bd799d                 1         1         1       18m

NAME                                            REFERENCE              TARGETS          MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/activator   Deployment/activator   <unknown>/100%   1         20        1          18m
horizontalpodautoscaler.autoscaling/webhook     Deployment/webhook     <unknown>/100%   1         5         1          18m

```

## Install istio

```
$ istioctl install -y
✔ Istio core installed                                                                                                                                                                                          
✔ Istiod installed                                                                                                                                                                                              
✔ Ingress gateways installed                                                                                                                                                                                    
✔ Installation complete                                                                                                                                                                                         
Making this installation the default for injection and validation.
Thank you for installing Istio 1.14.  Please take a few minutes to tell us about your install/upgrade experience!  https://forms.gle/yEtCbt45FZ3VoDT5A

$ kubectl label namespace knative-serving istio-injection=enabled
namespace/knative-serving labeled

$ touch PERMISSIVE.yaml
$ cat <<EOF >>PERMISSIVE.yaml
apiVersion: "security.istio.io/v1beta1"
kind: "PeerAuthentication"
metadata:
  name: "default"
  namespace: "knative-serving"
spec:
  mtls:
    mode: PERMISSIVE
EOF
$ kubectl apply -f PERMISSIVE.yaml
peerauthentication.security.istio.io/default created

$ kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.6.0/net-istio.yaml
clusterrole.rbac.authorization.k8s.io/knative-serving-istio created
gateway.networking.istio.io/knative-ingress-gateway created
gateway.networking.istio.io/knative-local-gateway created
service/knative-local-gateway created
configmap/config-istio created
peerauthentication.security.istio.io/webhook created
peerauthentication.security.istio.io/domainmapping-webhook created
peerauthentication.security.istio.io/net-istio-webhook created
deployment.apps/net-istio-controller created
deployment.apps/net-istio-webhook created
secret/net-istio-webhook-certs created
service/net-istio-webhook created
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.istio.networking.internal.knative.dev created
validatingwebhookconfiguration.admissionregistration.k8s.io/config.webhook.istio.networking.internal.knative.dev created
```

Verify istio and knative installation 
 ```                                                                                     
$ kubectl get all -n istio-system
NAME                                       READY   STATUS    RESTARTS   AGE
pod/istio-ingressgateway-f65885448-r7777   1/1     Running   0          25m
pod/istiod-5cdcff89c6-vfb7j                1/1     Running   0          25m

NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP                                                                PORT(S)                                      AGE
service/istio-ingressgateway    LoadBalancer   172.20.151.15    a2064bddcf6ed4d0ba63cd2c905ccacc-2103144973.ap-south-1.elb.amazonaws.com   15021:32560/TCP,80:32082/TCP,443:30066/TCP   25m
service/istiod                  ClusterIP      172.20.202.252   <none>                                                                     15010/TCP,15012/TCP,443/TCP,15014/TCP        25m
service/knative-local-gateway   ClusterIP      172.20.148.4     <none>                                                                     80/TCP                                       9m47s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   1/1     1            1           25m
deployment.apps/istiod                 1/1     1            1           25m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-ingressgateway-f65885448   1         1         1       25m
replicaset.apps/istiod-5cdcff89c6                1         1         1       25m

NAME                                                       REFERENCE                         TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   <unknown>/80%   1         5         1          25m
horizontalpodautoscaler.autoscaling/istiod                                                                                             53/UDP,53/TCP                                137m

$ kubectl get all -n knative-serving
NAME                                         READY   STATUS    RESTARTS   AGE
pod/activator-5f9df88fbb-bdfjw               1/1     Running   0          51m
pod/autoscaler-c7d7bd5f-w6mhm                1/1     Running   0          51m
pod/controller-5f64fb5c49-lg86b              1/1     Running   0          51m
pod/domain-mapping-7499d44b7f-wzvlg          1/1     Running   0          51m
pod/domainmapping-webhook-695848595b-dcphs   1/1     Running   0          51m
pod/net-istio-controller-c67797bd5-hnpcr     1/1     Running   0          10m
pod/net-istio-webhook-588f859479-mkt4r       2/2     Running   0          10m
pod/webhook-79d8bd799d-ddr9l                 1/1     Running   0          51m

NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                   AGE
service/activator-service            ClusterIP   172.20.91.212    <none>        9090/TCP,8008/TCP,80/TCP,81/TCP,443/TCP   51m
service/autoscaler                   ClusterIP   172.20.247.254   <none>        9090/TCP,8008/TCP,8080/TCP                51m
service/autoscaler-bucket-00-of-01   ClusterIP   172.20.67.199    <none>        8080/TCP                                  51m
service/controller                   ClusterIP   172.20.61.172    <none>        9090/TCP,8008/TCP                         51m
service/domainmapping-webhook        ClusterIP   172.20.170.123   <none>        9090/TCP,8008/TCP,443/TCP                 51m
service/net-istio-webhook            ClusterIP   172.20.130.249   <none>        9090/TCP,8008/TCP,443/TCP                 10m
service/webhook                      ClusterIP   172.20.96.104    <none>        9090/TCP,8008/TCP,443/TCP                 51m

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/activator               1/1     1            1           51m
deployment.apps/autoscaler              1/1     1            1           51m
deployment.apps/controller              1/1     1            1           51m
deployment.apps/domain-mapping          1/1     1            1           51m
deployment.apps/domainmapping-webhook   1/1     1            1           51m
deployment.apps/net-istio-controller    1/1     1            1           10m
deployment.apps/net-istio-webhook       1/1     1            1           10m
deployment.apps/webhook                 1/1     1            1           51m

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/activator-5f9df88fbb               1         1         1       51m
replicaset.apps/autoscaler-c7d7bd5f                1         1         1       51m
replicaset.apps/controller-5f64fb5c49              1         1         1       51m
replicaset.apps/domain-mapping-7499d44b7f          1         1         1       51m
replicaset.apps/domainmapping-webhook-695848595b   1         1         1       51m
replicaset.apps/net-istio-controller-c67797bd5     1         1         1       10m
replicaset.apps/net-istio-webhook-588f859479       1         1         1       10m
replicaset.apps/webhook-79d8bd799d                 1         1         1       51m

NAME                                            REFERENCE              TARGETS          MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/activator   Deployment/activator   <unknown>/100%   1         20        1          51m
horizontalpodautoscaler.autoscaling/webhook     Deployment/webhook     <unknown>/100%   1         5         1          51m

```

### Domain Config
```
$ kubectl patch configmap/config-domain \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"kn.timam.io":""}}'
configmap/config-domain patched
```