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
## O que é Replica set?
![](/replicaset2.png)
Aumentar a quantidade de um mesmo pod.

Serve para dar dsiponibilidade ou balanceamento de carga.

````
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: portal-noticias-deployment
spec:
  template:
    metadata:
      name: portal-noticias
      labels:
        app: portal-noticias
    spec:
      containers:
        - name: portal-noticias-container
          image: aluracursos/portal-noticias:1
          ports:
            - containerPort: 80
          envFrom:
            - configMapRef:
                name: portal-configmap
  replicas: 3
  selector:
    matchLabels:
      app: portal-noticias
````

Irá aparecer 3 pod:
````
kubectl get pods
````

Ao deletar algum pod do replica set, será criado outro.

Consultar replica sets:

````
kubectl get rs
````
ou
````
kubectl get replicasets
````


## O que é Deployment?
![](/deployment2.png)

Uma camada a mais de um replica set. Permite o controle das versões.
````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: portal-noticias-deployment
spec:
  template:
    metadata:
      name: portal-noticias
      labels:
        app: portal-noticias
    spec:
      containers:
        - name: portal-noticias-container
          image: aluracursos/portal-noticias:1
          ports:
            - containerPort: 80
          envFrom:
            - configMapRef:
                name: portal-configmap
  replicas: 3
  selector:
    matchLabels:
      app: portal-noticias
````
Ao criar um deployment necessariamente cria um replica set.

Consultar deployments:

````
kubectl get deployments
````

Consultar o histórico das mudanças:
````
kubectl rollout history deployment nome-do-deployment
````

Alterar o nome da mudança:

````
kubectl annotate deployment nome-do-deployment kubernetes.io/change-cause="Uma mensagem criada"
````

Mudar para uma revision específica( no exemplo a segunda):
````
kubectl rollout undo deployment nome-do-deployment --to-revision=2
```` 

## O que são volumes?
Maneira de persistir os dados.

![](/volume2.png)

````
apiVersion: v1
kind: Pod
metadata:
  name: um-pod-qualquer
spec:
  containers:
    - name: nginx-container
      image: nginx:latest
      volumeMounts:
        - mountPath: /pasta-de-arquivos
          name: volume-pod
  volumes:
    - name: volume-pod
      hostPath:
        path: /C/Users/Daniel/Desktop/uma-pasta-no-host
        type: Directory
````
Verificar o arquivo no pod:
````
kubectl exec -it pod-volume --container nome-do-container -- bash
````

````
cd ./pasta-de-arquivos ls
````
## Persistence volumes e persistence volumes claims
Dados persistidos em cloud.
````
apiVersion: v1
kind: PersistVolume
metadata:
    name: pv-1
spec:
    capacity:
        storage: 10Gi
    accessModes:
        - ReadWriteOnce
    gcePersistentDisk:
        pdName: pv-disk
    storageClassName: standard
````

````
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pvc-1
spec:
    accessModes:
        - ReadWriteOnde
    resources:
        requests:
            storage: 10Gi
    storageClassName: standard
````

````
apiVersion: v1
kind: Pod
metadata:
    name: pod-pv
spec:
    containers:
        - name: nginx-container
            image: nginx-latest
            volumeMounts:
                - mountPath: /volume-dentro-do-container
                name: primeiro-pv
    volumes:
        - name: primeiro-pv
            hostPath:
                persistentVolumeClaim:
                    claimName: pvc-1
````
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

