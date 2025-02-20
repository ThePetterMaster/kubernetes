# Kubernetes 
## Arquitetura
Um cluster é um conjunto de máquinas, que pode ser do tipo node ou master.

![](/arquiteturak8.png)

![](/arquiteturak82.png)

Kubelet - Garantir que os contêineres estejam em execução em cada nó do cluster.

Kube-proxy - Gerencia o tráfego entre os serviços e pods de um cluster.

Etcd é um sistema de armazenamento de chave-valor distribuído, de código aberto, que é usado para configurar, descobrir serviços e coordenar sistemas distribuídos.
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

ou pela label

````
curl http://svc-pod-2
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
      targetPort: 80
      nodePort: 30000 
  selector:
    app: primeiro-pod
````
Obs: targetPort se não foi definido, será o mesmo do port.
Obs: nodePort é a porta que será acessada fora do cluster.

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

Acessando externamente no windows: localhost:3000

Acessando externamente no linux:
````
kubectl get nodes -o wide
````
Ip em  INTERNAL-IP

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

![](/aula3_video1_imagem1.webp)


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
## Storage Classes 

Persistence volumes são criados automaticamente, pod se comunica diretamente com o pvc.

![](/StoregeClasses.png)

````
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard
  fstype: ext4
````

## O que é Statefull set?

Semelhante ao storage classes, porém cada pod tem um identificador de modo que o pod possa ser substituído e ser considerado o mesmo pod.

![](/statefulset.png)

````
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: sistema-noticias-statefulset
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: sistema-noticias
      name: sistema-noticias
    spec:
      containers:
        - name: sistema-noticias-container
          image: aluracursos/sistema-noticias:1
          ports:
            - containerPort: 80
          envFrom:
            - configMapRef:
                name: sistema-configmap
          volumeMounts:
            - name: imagens
              mountPath: /var/www/html/uploads
            - name: sessao
              mountPath: /tmp
      volumes:
        - name: imagens
          persistentVolumeClaim:
            claimName: imagens-pvc
        - name: sessao
          persistentVolumeClaim:
            claimName: sessao-pvc
  selector:
    matchLabels:
      app: sistema-noticias
  serviceName: svc-sistema-noticias
````
## O que são Liveness Probs e Readiness Probs ?

Liveness Probs: restarta o pod depois de estar com problemas, no exemplo depois de 3 requisições http com status erro.

Readiness Probs: faz o pod não receber requisições depois de estar com problemas, no exemplo depois de 5 requisições http com status erro.

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
          livenessProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 10
            failureThreshold: 3
            initialDelaySeconds: 20
          readinessProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 10
            failureThreshold: 5
            initialDelaySeconds: 3
  replicas: 3
  selector:
    matchLabels:
      app: portal-noticias
````

## O que é Horizontal pod auto scaler?

Aumento/diminuição da quantidade de pods com base da demanda.

Configurando a quantidade de 10 milicores de consumo médio por pod no deployment:

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
          livenessProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 10
            failureThreshold: 3
            initialDelaySeconds: 20
          readinessProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 10
            failureThreshold: 5
            initialDelaySeconds: 3
          resources:
            requests:
              cpu: 10m
  replicas: 3
  selector:
    matchLabels:
      app: portal-noticias
````

Definindo hpa:

- Mínimo de 1 réplica.
- Máximo de 10 réplicas.
- Útilização de 50% da cpu definido no deplyment(10 milicores)
  
````
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: portal-noticias-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: portal-noticias-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
````

## Servidor de estresse: 

https://cdn3.gnarususercontent.com.br/1916-kubernetes/stress.zip

stress.sh

## Servidor de métricas linux

````
minikube addons enable metrics-server
````

## Servidor de métricas no windows

components.yaml
````
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:aggregated-metrics-reader
  labels:
    rbac.authorization.k8s.io/aggregate-to-view: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
rules:
- apiGroups: ["metrics.k8s.io"]
  resources: ["pods", "nodes"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: apiregistration.k8s.io/v1beta1
kind: APIService
metadata:
  name: v1beta1.metrics.k8s.io
spec:
  service:
    name: metrics-server
    namespace: kube-system
  group: metrics.k8s.io
  version: v1beta1
  insecureSkipTLSVerify: true
  groupPriorityMinimum: 100
  versionPriority: 100
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: metrics-server
  namespace: kube-system
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
      # mount in tmp so we can safely use from-scratch images and/or read-only containers
      - name: tmp-dir
        emptyDir: {}
      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server/metrics-server:v0.3.7
        imagePullPolicy: IfNotPresent
        args:
          - --cert-dir=/tmp
          - --secure-port=4443
          - --kubelet-insecure-tls
        ports:
        - name: main-port
          containerPort: 4443
          protocol: TCP
        securityContext:
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
      nodeSelector:
        kubernetes.io/os: linux
        kubernetes.io/arch: "amd64"
---
apiVersion: v1
kind: Service
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    kubernetes.io/name: "Metrics-server"
    kubernetes.io/cluster-service: "true"
spec:
  selector:
    k8s-app: metrics-server
  ports:
  - port: 443
    protocol: TCP
    targetPort: main-port
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  - nodes/stats
  - namespaces
  - configmaps
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
````
## Ingress

![](/Ingress_basico.png)

### Conexão com service

![](/ingress_service.png)

### Múltiplos paths

![](/multiplos_paths.png)

### Múltiplos sub domínios

![](/Multiplos_subdominios.png)

### TLS

![](/ingress_tls.png)


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

