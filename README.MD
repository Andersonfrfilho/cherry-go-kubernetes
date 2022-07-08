1 - criar cluster com o kind:
```bash 
kind create cluster
```
2 - informações sobre o cluster pelo o kubectl
```bash
kubectl cluster-info --context kind-kind
```

2.1 - listar configurações de clusters
```bash
kubectl config get-clusters
```
2.2 - mudar contexto de cluster
```bash
kubectl config use-context nome-do-cluster
```

3 - create a cluster with config and name
```bash
kind create cluster --config=k8s/kind.yaml --name=fullcycle
```
1 - mount image:
```docker build -t andersonfrfilho/hello-nodejs .```
2 - test image
```docker run --rm -p 3000:3000 andersonfrfilho/hello-nodejs```

3 - criar pod
```sh
kubectl apply -f k8s/pod.yaml
```

4 - Olhar os logs
kubectl logs nomeDoPod

5 - Deletar deployment
kubectl delete deploy nodeDeployment

5.1 - Deletar um statefulset
kubectl delete StatefulSet postgres

## para acessar os pods vc temq eu ter um mecanismo de rede que são os services

7 - caso tenha 1problema no get de pods executar:
```sh
kubectl config set-context --current --namespace=default
```

8 - Executar comando dentro dos pods:
```kubectl exec -it nome-do-pod -- bash```

9 - verificar service criado (postgres-h) nome_service:
```ping postgres-h```

10 - verificar service por pod (postgres-0.postgres-h) nome do nome_pod.nome_service
```ping postgres-0.postgres-h```
obs:
- ```pod.service.namespace.service.cluster.local```
- ```postgres-3.postgres-h.default.svc.cluster.local```

## Replicaset
- ficam monitorando quando a aplicação cair e replicam os pods
11 - traz todas replicasets
```kubectl get replicasets```

## Deployments:
- modificam as replicas conforme as versões mudam
12 - rollout mostrar as revision
```kubectl rollout history deployment nodejs-server-deployment```
13 - retornar para versão anterior
```kubectl rollout undo deployment nodejs-server-deployment```
14 - retornar para versão especifica
```kubectl rollout undo deployment nodejs-server-deployment --to-revision={{pegue a revision com o comando de history}}```
15 - detalhes do objeto de deployment
```kubectl describe deployment nodejs-server-deployment```

## services
- services são responsaveis pelo a comunicação dentro do kubernetes
13 - redirecionamento de porta no kubernetes
- ```kubectl port-forward service/nodejs-server-service 3000:3000```
14 - acessar o service por um proxy do kubernetes para informações da api do kubernets
- ```kubectl proxy --port=3000```
* ClusterIp -> ip interno do servidor
* NodePort -> expor pra fora uma porta
* LoadBalancer -> gera um ip para vc acessar seu app de fora

15 - descrição de pods:
```kubectl describe pod nomeDoPod```

16 - kubectl --generator
17 - instalar metricas no cluster
  * 
ver informações do cluster hpa
```watch -n1 kubectl get hpa```
ver informações de top pod
```kubectl top pod nodejs-server-78676c7f4f-qmp7k```
17 - HPA 
baixe o servico:
wget link do repositório
https://github.com/kubernetes-sigs/metrics-server

1 - vai baixa o seguinte arquivo:
2 - components.yaml
3 - renomeie para metrics-server
4 - para executar local adicione no seguinte arquivo na parte que original:
```yaml
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```
adicione a ultima linha:
```yaml
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
```
teste de carga: utilizando o fortio
//-it crie a image
//--rm remove o container quando terminar
//--image= nome da imagem 
//-- execute o comando dentro da imagem
//load-> lib do fortio
//-qps (queries per seconds) numeros de queries por segundo
//-t tempo de execução do teste
//-c números de conexões

```sh
kubectl run -it fortio --rm --image=fortio/fortio -- load -qps 1800 -t 120s -c 70 "http://nodejs-server-service:3000/healthz"
```

```sh
echo -n "value" | base64
```

```sh
kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=andersonfrfilho --docker-password=afrf102030 --docker-email=andersonfrfilho@gmail.com
```

```yaml
  template:
    metadata:
      labels:
        app: cherry-go-backend 
    spec:
      containers:
        - name: cherry-go-backend-api-container
          image: "andersonfrfilho/cherry-go_api_backend:1.0.0"
          resources:
            requests: 
              cpu: "0.3"
              memory: 20Mi 
            limits: 
              cpu: 500m
              memory: 25Mi 
          startupProbe: 
            httpGet:
              path: '/healthcheck'
              port: 3333 
            periodSeconds: 3 
            failureThreshold: 30 
          readinessProbe: 
            httpGet:
              path: '/healthcheck'
              port: 3333 
            periodSeconds: 3 
            failureThreshold: 1 
          livenessProbe:
            httpGet: 
              path: '/healthcheck'
              port: 3333 
            periodSeconds: 5 
            failureThreshold: 1 
            timeoutSeconds: 1 
            successThreshold: 1 
          env:
            # - name: POSTGRES_PASSWORD
            #   value: "102030"      
          # envFrom:
          #   - configMapRef:
          #       name: cherry-go-env
          #   - secretRef:
          #       name: cherry-go-secret-env
      imagePullSecrets:
      - name: regcred
```
connect database:
```sh
kubectl port-forward service/postgres-h 5432:5432
```

```sh
kubectl delete pvc --all 
```

kubectl port-forward service/cherry-go-backend-service 3333:80
## criando cluster cherry-go

## service do kafka

1º - crie um namespace
kubectl create namespace cherry-go-kafka
2º - crie o kafka cluster
helm install cherry-go-kafka --set externalAccess.enabled=false,serviceAccount.create=true,rbac.create=true,deleteTopicEnable=true,allowPlaintextListener=true,autoCreateTopicsEnable=true bitnami/kafka -n cherry-go-kafka

message
```
NAME: cherry-go-kafka
LAST DEPLOYED: Thu Jun  2 09:58:07 2022
NAMESPACE: cherry-go-kafka
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: kafka
CHART VERSION: 17.1.0
APP VERSION: 3.2.0

** Please be patient while the chart is being deployed **

Kafka can be accessed by consumers via port 9092 on the following DNS name from within your cluster:

    cherry-go-kafka.cherry-go-kafka.svc.cluster.local

Each Kafka broker can be accessed by producers via port 9092 on the following DNS name(s) from within your cluster:

    cherry-go-kafka-0.cherry-go-kafka-headless.cherry-go-kafka.svc.cluster.local:9092

To create a pod that you can use as a Kafka client run the following commands:

    kubectl run cherry-go-kafka-client --restart='Never' --image docker.io/bitnami/kafka:3.2.0-debian-10-r4 --namespace cherry-go-kafka --command -- sleep infinity
    kubectl exec --tty -i cherry-go-kafka-client --namespace cherry-go-kafka -- bash

    PRODUCER:
        kafka-console-producer.sh \

            --broker-list cherry-go-kafka-0.cherry-go-kafka-headless.cherry-go-kafka.svc.cluster.local:9092 \
            --topic test

    CONSUMER:
        kafka-console-consumer.sh \

            --bootstrap-server cherry-go-kafka.cherry-go-kafka.svc.cluster.local:9092 \
            --topic test \
            --from-beginning
```
consumer
groffe-broker-kafka.groffe-testes-kafka.svc.cluster.local
producer
groffe-broker-kafka-0.groffe-broker-kafka-headless.groffe-testes-kafka.svc.cluster.local:9092


kubectl get pods -n cherry-go-kafka
kubectl get services -n cherry-go-kafka

delete namespace
kubectl delete all --all -n cherry-go-kafka
kubectl delete ns cherry-go-kafka

## criar postgres Database
1.5 - create database

1.5.1 - kubectl apply -f .\postgres\persistentVolume.yaml
1.5.2 - kubectl apply -f .\postgres\persistentVolumeClaim.yaml
1.5.3 - kubectl apply -f .\postgres\deployment.yaml
1.5.4 - kubectl apply -f .\postgres\service.yaml

## criar mongo Database
1.5 - create database

1.5.1 - kubectl apply -f .\mongo\persistentVolume.yaml
1.5.2 - kubectl apply -f .\mongo\persistentVolumeClaim.yaml
1.5.3 - kubectl apply -f .\mongo\deployment.yaml
1.5.4 - kubectl apply -f .\mongo\service.yaml

## criar redis service
2 - create redis service

helm install cherry-go-redis --set persistence.storageClass=nfs-client,redis.replicas.persistence.storageClass=nfs-client bitnami/redis --set volumePermissions.enabled=true

the message

```
NAME: cherry-go-redis
LAST DEPLOYED: Thu Jun  2 10:08:39 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: redis
CHART VERSION: 16.10.1
APP VERSION: 6.2.7

** Please be patient while the chart is being deployed **

Redis&trade; can be accessed on the following DNS names from within your cluster:

    cherry-go-redis-master.default.svc.cluster.local for read/write operations (port 6379)
    cherry-go-redis-replicas.default.svc.cluster.local for read-only operations (port 6379)



To get your password run:

    export REDIS_PASSWORD=$(kubectl get secret --namespace default cherry-go-redis -o jsonpath="{.data.redis-password}" | base64 -d)

To connect to your Redis&trade; server:

1. Run a Redis&trade; pod that you can use as a client:

   kubectl run --namespace default redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:6.2.7-debian-10-r23 --command -- sleep infinity

   Use the following command to attach to the pod:

   kubectl exec --tty -i redis-client \
   --namespace default -- bash

2. Connect using the Redis&trade; CLI:
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h cherry-go-redis-master
   REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h cherry-go-redis-replicas

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace default svc/cherry-go-redis-master 6379:6379 &
    REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p 6379
```    
in windows get password run command
kubectl get secret --namespace default cherry-go-redis -o jsonpath="{.data.redis-password}"
convert base64 to string

## criar api

kubectl apply -f ./api;

## criar backoffice 

kubectl apply -f ./backoffice;

## criar front

kubectl apply -f ./client

instalando ingress:
digital ocean pegar o provedor neste site -> https://kubernetes.github.io/ingress-nginx/deploy/#digital-ocean
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.1/deploy/static/provider/do/deploy.yaml

kubectl get all -n ingress-nginx

Ajustar dominio

kubectl apply -f ./ingress

#######################################################

kubectl create namespace ingress-nginx

helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace

kubectl create namespace ingress-nginx-cherry-go-backoffice
helm install my-release nginx-stable/nginx-ingress
helm install ingress-nginx-cherry-go-backoffice nginx-stable/nginx-ingress --namespace ingress-nginx --create-namespace

kubectl get service ingress-nginx-controller --namespace=ingress-nginx

install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.0/cert-manager.yaml

aguardar o load balancer ser liberado para adicionar no seu dns -> kubectl get service ingress-nginx-controller --namespace=ingress-nginx

kubectl apply -f .\api\cluster-issuer.yaml

kubectl apply -f .\api\ingress.yaml

## criar o worker

cd worker;kubectl apply -f .\configmap-env.yaml;kubectl apply -f .\secret.yaml;kubectl apply -f .\deployment.yaml;kubectl apply -f .\hpa.yaml;kubectl apply -f .\service.yaml;cd ..;

# configurar na digital ocean

doctl kubernetes cluster kubeconfig save 3aba708d-0c1e-495f-ae33-247bf56f31fa