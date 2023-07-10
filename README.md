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


Create a ServiceAccount and ClusterRoleBinding yaml files as it's indicated in:

https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md


dashboard_ServiceAccount.yml:
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

```


dashboard_ClusterRoleBinding.yml:
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

```


```
> kubectl apply -f dashboard_ServiceAccount.yml 
serviceaccount/admin-user created

> kubectl apply -f dashboard_ClusterRoleBinding.yml 
clusterrolebinding.rbac.authorization.k8s.io/admin-user created

```

Obtain a token bearer:

```
kubectl -n kubernetes-dashboard create token admin-user

 
eyJhbGciOiJSUzI1NiIsImtpZCI6Im1qWmdYY3J5YWFmdU9oa2MtSjRMSi1uMWVHMGVHWWdpV0FWZnN1MHVQY3MifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjg4OTc1Mjc4LCJpYXQiOjE2ODg5NzE2NzgsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJhZG1pbi11c2VyIiwidWlkIjoiMTAyMzc4M2YtY2MyMi00ZGJkLTkzMWMtZWI2MmU5Y2Y3N2I2In19LCJuYmYiOjE2ODg5NzE2NzgsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDphZG1pbi11c2VyIn0.JK6EOt3td_NM1TN3mk3dFHlD2Rhnzf7VHgVnsYEu9g_40FAwbzEallcFifhr4SHhkb0Z-iNP7rC-2rNpoldwUm2252kpBglkvmUbNc0OeG1q3QpJCO1OcrqpeDRzXew8Lv91yxjaQLSuDbFewzbHUepWCk05DMvs68q8bWdnNkQ5kiRSShz2pThXT8Ugd-kZuy43y1JarsYG1aCvdmVYanL2Cta7idG5aKNBYpMygZnijpBes6onNX9vFb3LOYrXV8Gl9oncJIC0eVpKj3dtFP0c0-aEkWHCPO8wHY8oGdU6Pu_yoHvSZVQhU-HPdYAHfxlreXy7dZwZQr56SAz5Mw

```


Now copy the token in the dashboard in order to authenticate.

But this was done in order to authenticat as admin user into the Dashboard.
For the project we must create a regular user named Sandry. 

First we create a namespace called "myproject":


```
> kubectl create namespace myproject
namespace/myproject created

```

Create the ServiceAccount:

```
> kubectl create serviceaccount sandry --namespace myproject
serviceaccount/sandry created

```

Create a ClusterRoleBinding with admin rights for Sandry:

```
> kubectl create clusterrolebinding crb_sandry --serviceaccount=myproject:sandry --clusterrole=cluster-admin
clusterrolebinding.rbac.authorization.k8s.io/crb_sandry created
```

Obtain the token bearer for Sandry user in order to login into the dashboard:


```
kubectl -n kubernetes-dashboard create token sandry -n myproject

eyJhbGciOiJSUzI1NiIsImtpZCI6Im1qWmdYY3J5YWFmdU9oa2MtSjRMSi1uMWVHMGVHWWdpV0FWZnN1MHVQY3MifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjg4OTc4NTkyLCJpYXQiOjE2ODg5NzQ5OTIsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJteXByb2plY3QiLCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoic2FuZHJ5IiwidWlkIjoiN2I5MmMyMmMtNzMwNi00ZGM5LTlkNmItY2FjMjJhMzU4OTMzIn19LCJuYmYiOjE2ODg5NzQ5OTIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpteXByb2plY3Q6c2FuZHJ5In0.JDSuxNFQ2YFZHvIEGpE4wWV_aljNPPE0JQEXKXjckhzM4v3ztMFS6RdZvgKK-G-16Xwa_yRJdSzhUh3-_dVMrK52flCGq5gHRPycDPwX3kMO-wC6Ua7Q0zLRWfPUDT6Iy7SfeENahkxwjOg_jbPYU3dk20GyxXeVzoWjLpMqbfsVzsZriiw7hWNZ5Z2tsvYrQC_I70Ie5bhpQmGvMKqa9W2skTwMsfTi9tp3KI7-TVcX5-qBDrCtF99BhbJY_5SuLDa7iGnrbefkwAJbAYY8jkob49fNVrZbkoPMvrPb4pqzhvqb1DmbEBvb-EYx84NdvXy_6UYNQEcP92kFgKS-pg
```


# NFS server in worker-node-3

https://fabianlee.org/2022/01/11/kubernetes-readwritemany-rwx-nfs-mount-using-static-volume/

Login into worker-node-3 and install NFS server packages:


```
sudo apt update

sudo apt install nfs-common nfs-kernel-server -y

```

Create a directory to export via NFS:

```
sudo mkdir -p /data/nfs3
sudo chown nobody:nogroup /data/nfs3
sudo chmod g+rwxs /data/nfs3 
```

Configure NFS exports:

/etc/exports:
```
/data/nfs3  192.168.0.0/16(rw,sync,no_subtree_check,no_root_squash)
```

```
> sudo exportfs -av
exporting 192.168.0.0/16:/data/nfs3

> sudo systemctl restart nfs-kernel-server

> sudo systemctl status nfs-kernel-server

● nfs-server.service - NFS server and services
     Loaded: loaded (/lib/systemd/system/nfs-server.service; enabled; vendor preset: enabled)
     Active: active (exited) since Mon 2023-07-10 07:56:59 UTC; 107ms ago
    Process: 34144 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 34145 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
   Main PID: 34145 (code=exited, status=0/SUCCESS)
        CPU: 18ms

Jul 10 07:56:59 worker-node-3 systemd[1]: Starting NFS server and services...
Jul 10 07:56:59 worker-node-3 systemd[1]: Finished NFS server and services.

```


Now, we must ensure all the cluster node have the NFS client installed. So we must 
do in all the nodes:


```
sudo apt update
sudo apt install nfs-common -y
```

And verify in every node that the NFS export is available. 
In my lab 192.168.56.105 is the worker-node-3's IP:

```
> /sbin/showmount -e 192.168.56.105
Export list for 192.168.56.105:
/data/nfs3 192.168.0.0/16
```

https://kubebyexample.com/learning-paths/application-development-kubernetes/lesson-4-customize-deployments-application-2

Enter into pod:

kubectl exec -it -n myproject  mariadb-deployment-59d9468bb8-t4bg4  -- bash


# mariadb deployment with Kubernetes Dashboard

From the kubernetes dashboard we will create the 
following resources:

mariadb-configmap.yaml
mariadb-secret.yaml
mariadb-deployment-pvc.yaml


After that, mariadb should be running. We can test with:


```

> kubectl get all -n myproject -o wide
NAME                                      READY   STATUS    RESTARTS   AGE   IP           NODE            NOMINATED NODE   READINESS GATES
pod/mariadb-deployment-575cf699f8-khthw   1/1     Running   0          45m   10.244.3.8   worker-node-3   <none>           <none>

NAME                               TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/mariadb-internal-service   NodePort   10.103.45.95   <none>        3306:32096/TCP   45m   app=mariadb

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES    SELECTOR
deployment.apps/mariadb-deployment   1/1     1            1           45m   mariadb      mariadb   app=mariadb

NAME                                            DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES    SELECTOR
replicaset.apps/mariadb-deployment-575cf699f8   1         1         1       45m   mariadb      mariadb   app=mariadb,pod-template-hash=575cf699f8



> mysql -uroot -psecret -h 192.168.56.103 -P 32096 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 11.0.2-MariaDB-1:11.0.2+maria~ubu2204 mariadb.org binary distribution

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

root@192.168.56.103 [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
5 rows in set (0.02 sec)
```





