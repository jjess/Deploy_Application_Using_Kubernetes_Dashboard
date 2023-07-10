# Kubernetes Dashboard Install

```
> kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Edit the service to use NodePort instead of ClusterIP:

```
> kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard


apiVersion: v1
kind: Service
metadata:
  annotations:

  ...

  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}

```

Then delete the pod in order to be recreated with NodePort service type.
```
kubectl delete pod kubernetes-dashboard-78c79f97b4-xuc2k -n kubernetes-dashboard
```


```
> kubectl get all -n kubernetes-dashboard -o wide

NAME                                             READY   STATUS    RESTARTS        AGE     IP            NODE            NOMINATED NODE   READINESS GATES
pod/dashboard-metrics-scraper-5cb4f4bb9c-bvchn   1/1     Running   1 (2d18h ago)   2d18h   10.244.2.85   worker-node     <none>           <none>
pod/kubernetes-dashboard-6967859bff-fzdj5        1/1     Running   1 (2d18h ago)   2d18h   10.244.1.8    worker-node-2   <none>           <none>

NAME                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE     SELECTOR
service/dashboard-metrics-scraper   ClusterIP   10.104.114.198   <none>        8000/TCP        2d18h   k8s-app=dashboard-metrics-scraper
service/kubernetes-dashboard        NodePort    10.109.127.192   <none>        443:32582/TCP   2d18h   k8s-app=kubernetes-dashboard

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS                  IMAGES                                SELECTOR
deployment.apps/dashboard-metrics-scraper   1/1     1            1           2d18h   dashboard-metrics-scraper   kubernetesui/metrics-scraper:v1.0.8   k8s-app=dashboard-metrics-scraper
deployment.apps/kubernetes-dashboard        1/1     1            1           2d18h   kubernetes-dashboard        kubernetesui/dashboard:v2.7.0         k8s-app=kubernetes-dashboard

NAME                                                   DESIRED   CURRENT   READY   AGE     CONTAINERS                  IMAGES                                SELECTOR
replicaset.apps/dashboard-metrics-scraper-5cb4f4bb9c   1         1         1       2d18h   dashboard-metrics-scraper   kubernetesui/metrics-scraper:v1.0.8   k8s-app=dashboard-metrics-scraper,pod-template-hash=5cb4f4bb9c
replicaset.apps/kubernetes-dashboard-6967859bff        1         1         1       2d18h   kubernetes-dashboard        kubernetesui/dashboard:v2.7.0         k8s-app=kubernetes-dashboard,pod-template-hash=6967859bff



> kubectl get svc --all-namespaces 

NAMESPACE              NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default                kubernetes                  ClusterIP   10.96.0.1        <none>        443/TCP                  4d21h
jenkins                jenkins-svc                 NodePort    10.108.35.178    <none>        8080:30000/TCP           66d
kube-system            kube-dns                    ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   67d
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.104.114.198   <none>        8000/TCP                 2d18h
kubernetes-dashboard   kubernetes-dashboard        NodePort    10.109.127.192   <none>        443:32582/TCP            2d18h
```



Now the dashboard is available at:

https://192.168.56.102:32582






