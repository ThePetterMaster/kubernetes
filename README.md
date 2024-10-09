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

### Cluster IP

Define ip fixo para comunicações internas

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

### Tipos de services
![](/servicetipos.png)
## O que é Config Map?
![](/configmap.png)
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

