# Kubernetes 
## Arquitetura
Um cluster é um conjunto de máquinas, que pode ser do tipo node ou master.

![](/arquiteturak8.png)

![](/arquiteturak82.png)
## O que é um pod?
Menor abstração do k8, encapsular um container.
![](/pod2.png)
Criando um pod:

`kubectl run nginx-pod --image=nginx:latest`

Listar todos os pods:

`kubectl get pods`

Listar e acompanhar todos os pods em tempo real suas alterações:

`kubectl get pods --watch`

Informações de um pod:

`kubectl describe pod nginx-pod`

Editar um pod:

`kubectl edit pod nginx-pod`

Criando pod de maneira declarativa no yaml:

```
apiVersion: v1
kind: Pod
metadata:
  name: primeiro-pod-declarativo
spec:
  containers:
    - name: container-pod-1
      image: nginx:latest
```

Executando o yml:

`kubectl apply -f .\primeiro-pod.yaml`


## O que é um service?
![](/service2.png)
### Tipos de services
![](/servicetipos.png)
#### Cluster IP

Define ip fixo para comunicações internas do cluster.

pod-2.yamal
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
    app: segundo-pod
spec:
  containers:
    - name: container-pod-2
      image: nginx:latest
      ports:
        - containerPort: 80
```

svc-pod-2.yamal
```
apiVersion: v1
kind: Service
metadata:
  name: svc-pod-2
spec:
  type: ClusterIP
  selector:
    app: segundo-pod
  ports:
    - port: 9000
      targetPort: 80
```

Listar services:
````
kubectl get service
````

````
kubectl get svc
````

Testar a comunicação:

````
kubeclt exec -it pod-1 --bash
````


````
curl ip-do-service
````

### Node port
Se comunica fora do cluster.


svc-pod-1.yamal
````
apiVersion: v1
kind: Service
metadata:
  name: svc-pod-1
spec:
  type: NodePort
  ports:
    - port: 80
      #targetPort: 80
      nodePort: 30000
  selector:
    app: primeiro-pod
````

pod-1.yamal
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    app: primeiro-pod
spec:
  containers:
    - name: container-pod-1
      image: nginx:latest
      ports:
        - containerPort: 80
```

### Load balancer
````
apiVersion: v1
kind: Service
metadata:
  name: svc-pod-loadbalancer-1
spec:
  type: LoadBalancer
  ports:
    - port: 80
      nodePort: 30000
  selector:
    app: primeiro-pod
````
## O que é Config Map?

Configuração de variáveis de ambientes em um yaml separado.

![](/configmap.png)

````
kubectl get configmap
````

````
kubectl  describe configmap NOME-CONFIGMAP
````

````
apiVersion: v1
kind: Pod
metadata:
  name: db-noticias
  labels:
    app: db-noticias
spec:
  containers:
    - name: db-noticias-container
      image: aluracursos/mysql-db:1
      ports:
        - containerPort: 3306
      envFrom:
        - configMapRef:
            name: db-configmap
````

````
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-configmap
data:
  MYSQL_ROOT_PASSWORD: q1w2e3r4
  MYSQL_DATABASE: empresa
  MYSQL_PASSWORD: q1w2e3r4
````
## O que são volumes?
![](/volumes.png)
## O que é deployment?
![](/deployment.png)
## O que é statefull?
![](/statefull.png)
# Kubernetes arquitetura
## Node processes
![](/nodeprocesses.png)
## Master nodes
Escalonar pod
Monitorar
Reestartar pod
Novo node
## Processos do master node
### Api server
![](/apiserver.png)
### Scheduler
![](/scheduler.png)
![](/scheduler2.png)
### Controller manager
![](/controllermanager.png)
### ETCD
![](/etcd.png)

