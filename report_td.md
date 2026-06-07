# Prerequis

Création du cluster pour le TD
Environement: Windows

```
k3d cluster create td-k8s --servers 1 --agents 2 --port "8080:80@loadbalancer" --k3s-arg "--disable=traefik@server:0"
kubectl get nodes
```

Ouput:
```
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> k3d cluster create td-k8s --servers 1 --agents 2 --port "8080:80@loadbalancer" --k3s-arg "--disable=traefik@server:0"
INFO[0000] portmapping '8080:80' targets the loadbalancer: defaulting to [servers:*:proxy agents:*:proxy] 
INFO[0000] Prep: Network                                
INFO[0000] Re-using existing network 'k3d-td-k8s' (53b3bec0318f041c9cdfaafd4adcc8e5926e09d8427738b49445be783d87b427) 
INFO[0000] Created image volume k3d-td-k8s-images       
INFO[0000] Starting new tools node...                   
INFO[0000] Starting node 'k3d-td-k8s-tools'             
INFO[0001] Creating node 'k3d-td-k8s-server-0'          
INFO[0001] Creating node 'k3d-td-k8s-agent-0'           
INFO[0001] Creating node 'k3d-td-k8s-agent-1'           
INFO[0001] Creating LoadBalancer 'k3d-td-k8s-serverlb'  
INFO[0002] Using the k3d-tools node to gather environment information 
INFO[0002] Starting new tools node...                   
INFO[0003] Starting node 'k3d-td-k8s-tools'             
INFO[0004] Starting cluster 'td-k8s'                    
INFO[0004] Starting servers...                          
INFO[0004] Starting node 'k3d-td-k8s-server-0'          
INFO[0010] Starting agents...                           
INFO[0011] Starting node 'k3d-td-k8s-agent-0'           
INFO[0011] Starting node 'k3d-td-k8s-agent-1'           
INFO[0016] Starting helpers...                          
INFO[0016] Starting node 'k3d-td-k8s-serverlb'          
INFO[0023] Injecting records for hostAliases (incl. host.k3d.internal) and for 5 network members into CoreDNS configmap... 
INFO[0026] Cluster 'td-k8s' created successfully!       
INFO[0026] You can now use it like this:                
    kubectl cluster-info
```

```shell
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> kubectl get nodes
NAME                  STATUS   ROLES                  AGE     VERSION
k3d-td-k8s-agent-0    Ready    <none>                 5m31s   v1.31.5+k3s1
k3d-td-k8s-agent-1    Ready    <none>                 5m31s   v1.31.5+k3s1
k3d-td-k8s-server-0   Ready    control-plane,master   5m37s   v1.31.5+k3s1
```

```shell
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> docker ps --filter "name=k3d-td-k8s"
CONTAINER ID   IMAGE                            COMMAND                  CREATED         STATUS         PORTS                                           NAMES
e9a9d839d4dc   ghcr.io/k3d-io/k3d-tools:5.8.3   "/app/k3d-tools noop"    6 minutes ago   Up 6 minutes                                                   k3d-td-k8s-tools
114f601d12b5   ghcr.io/k3d-io/k3d-proxy:5.8.3   "/bin/sh -c nginx-pr…"   6 minutes ago   Up 6 minutes   0.0.0.0:8080->80/tcp, 0.0.0.0:61604->6443/tcp   k3d-td-k8s-serverlb
e51495f9975d   rancher/k3s:v1.31.5-k3s1         "/bin/k3d-entrypoint…"   6 minutes ago   Up 6 minutes                                                   k3d-td-k8s-agent-1
398658e1e663   rancher/k3s:v1.31.5-k3s1         "/bin/k3d-entrypoint…"   6 minutes ago   Up 6 minutes                                                   k3d-td-k8s-agent-0
58cfdbe76996   rancher/k3s:v1.31.5-k3s1         "/bin/k3d-entrypoint…"   6 minutes ago   Up 6 minutes                                                   k3d-td-k8s-server-0
```

```
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> docker pull stephanparichon/epsi-k8s-bff:1.0
>> docker pull stephanparichon/epsi-k8s-front:1.0
>> docker pull stephanparichon/epsi-k8s-front:2.0
1.0: Pulling from stephanparichon/epsi-k8s-bff
cf998ddc23af: Pull complete 
5f49c09562dc: Pull complete 
4feea04c1543: Pull complete 
d2e3624464ff: Pull complete 
fff4e2c1b189: Pull complete 
b2cbbfe903b0: Pull complete 
cb73c963d1b3: Pull complete 
Digest: sha256:5b8b532e6cbe984899d5b9bbd55061e8efa22f7360c046909010f3d1bfeaded9
Status: Downloaded newer image for stephanparichon/epsi-k8s-bff:1.0
docker.io/stephanparichon/epsi-k8s-bff:1.0
1.0: Pulling from stephanparichon/epsi-k8s-front
6f2e79bec71f: Pull complete 
81bd8ed7ec67: Pull complete 
34a64644b756: Pull complete 
b464cfdf2a63: Pull complete 
f9ce9734d584: Pull complete 
d7e507024086: Pull complete 
197eb75867ef: Pull complete 
39c2ddfd6010: Pull complete 
61ca4f733c80: Pull complete 
Digest: sha256:7647bd204e4c77484e7a24c46e604bcad7afac1f95ed6a92271bc404687fdae2
Status: Downloaded newer image for stephanparichon/epsi-k8s-front:1.0
docker.io/stephanparichon/epsi-k8s-front:1.0
2.0: Pulling from stephanparichon/epsi-k8s-front
7b116614dbf0: Pull complete 
Digest: sha256:98dab0df3a62fbd081675be48613c6fb30750adf2228402e3c0620892e9d53cc
Status: Downloaded newer image for stephanparichon/epsi-k8s-front:2.0
docker.io/stephanparichon/epsi-k8s-front:2.0
```

```
k3d image import `
  stephanparichon/epsi-k8s-bff:1.0 `
  stephanparichon/epsi-k8s-front:1.0 `
  stephanparichon/epsi-k8s-front:2.0 `
  --cluster td-k8s
```

```
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> kubectl run smoke-nginx --image=nginx:1.27-alpine --port=80 --labels="app=smoke"
pod/smoke-nginx created
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> kubectl expose pod smoke-nginx --type=LoadBalancer --port=80
service/smoke-nginx exposed
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> kubectl get pod,svc
NAME              READY   STATUS    RESTARTS   AGE
pod/smoke-nginx   1/1     Running   0          13s

NAME                  TYPE           CLUSTER-IP     EXTERNAL-IP                        PORT(S)        AGE
service/kubernetes    ClusterIP      10.43.0.1      <none>                             443/TCP        28m
service/smoke-nginx   LoadBalancer   10.43.73.166   172.18.0.3,172.18.0.4,172.18.0.5   80:30686/TCP   7s
```

```
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> kubectl delete pod smoke-nginx
pod "smoke-nginx" deleted
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> kubectl delete svc smoke-nginx
service "smoke-nginx" deleted
PS C:\Users\yutin\OneDrive\Bureau\Workplace\td-todoliste> 
```

Prérequis OK.

