# Exerc-cios-Kubernets
Práticas para aprender kubernets 
Lista de Exercícios de Kubernetes:
 
Como criar um Deployment com 3 réplicas usando uma imagem do NGINX e verificar os Pods criados?

1. Criar o Deployment
Crie um arquivo YAML para o Deployment. O Kubernetes usará esse arquivo para criar e gerenciar o Deployment com 3 réplicas.

Exemplo de arquivo nginx-deployment.yaml:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3  # Definindo 3 réplicas
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest  # Usando a imagem oficial do NGINX
        ports:
        - containerPort: 80
```
Explicação do YAML:

apiVersion: apps/v1: Especifica que estamos criando um Deployment.
kind: Deployment: Define que é um objeto do tipo Deployment.
replicas: 3: Define que o Kubernetes deve criar 3 réplicas do contêiner.
image: nginx:latest: Usa a imagem oficial do NGINX.
containerPort: 80: Expõe a porta 80 no contêiner para acesso HTTP.
2. Aplicar o Deployment no Cluster
Agora que você tem o arquivo YAML, aplique-o no seu cluster Minikube.

Salve o conteúdo acima em um arquivo chamado nginx-deployment.yaml.
Aplique o Deployment:
```
kubectl apply -f nginx-deployment.yaml
```
Esse comando criará o Deployment e 3 réplicas do NGINX no cluster.

3. Verificar os Pods Criados
Após aplicar o Deployment, você pode verificar os Pods criados para garantir que tudo está funcionando corretamente.

Listar todos os Pods:
```
kubectl get pods
```
Esse comando deve retornar uma lista com 3 Pods do NGINX, semelhantes a isso:

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5d6f6c5b8d-k6p5x   1/1     Running   0          10s
nginx-deployment-5d6f6c5b8d-n7wpf   1/1     Running   0          10s
nginx-deployment-5d6f6c5b8d-wf9d8   1/1     Running   0          10s
```
READY: Mostra quantos contêineres dentro do Pod estão prontos (1/1 significa que o contêiner está pronto).
STATUS: O estado atual do Pod (ex.: Running, Pending, CrashLoopBackOff, etc.).
RESTARTS: O número de reinicializações do Pod.
AGE: O tempo de vida do Pod desde que foi criado.
Descrever os Pods (Opcional): Para ver mais detalhes sobre um Pod específico, como logs e status, você pode usar:
```
kubectl describe pod <nome-do-pod>
```
4. Verificar o Deployment
Além dos Pods, você pode verificar o Deployment em si:

```
kubectl get deployments
```
Esse comando mostrará detalhes sobre o Deployment, como o número de réplicas que você definiu.

5. Escalar o Deployment (Opcional)
Se você quiser alterar o número de réplicas, pode escalar o Deployment a qualquer momento. Por exemplo, para aumentar ou diminuir as réplicas para 5:

```
kubectl scale deployment nginx-deployment --replicas=5
```
Esse comando ajustará o número de Pods conforme a necessidade.

6. Limpar os Recursos (Opcional)
Depois de testar ou se não precisar mais do Deployment, você pode excluí-lo:

```
kubectl delete -f nginx-deployment.yaml
```
Esse comando irá excluir o Deployment e todos os Pods associados.

Como adicionar um ConfigMap para armazenar configurações e utilizá-lo em um Deployment para definir variáveis de ambiente?

1. Criar o ConfigMap
Um ConfigMap armazena informações que podem ser usadas em Pods ou Deployments. Para criá-lo, você pode definir um arquivo YAML ou usar a linha de comando do kubectl.

Exemplo de arquivo YAML para o ConfigMap (configmap.yaml):
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: "db.example.com"
  DB_PORT: "5432"
  DB_USER: "admin"
  DB_PASSWORD: "password123"
```
metadata.name: Nome do ConfigMap (app-config).
data: As variáveis de configuração (em formato chave-valor) que você quer armazenar.
Criando o ConfigMap com a CLI:
Se preferir, você pode criar o ConfigMap diretamente com o kubectl:

```
kubectl create configmap app-config \
  --from-literal=DB_HOST=db.example.com \
  --from-literal=DB_PORT=5432 \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=password123
```
2. Criar ou Modificar o Deployment para Usar o ConfigMap
Agora, você pode modificar ou criar um Deployment para usar o ConfigMap e injetar as variáveis de ambiente nos contêineres.

Exemplo de deployment usando o ConfigMap (nginx-deployment.yaml):
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        env:
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config  # Nome do ConfigMap
              key: DB_HOST      # Chave para a variável de ambiente
        - name: DB_PORT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_PORT
        - name: DB_USER
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_USER
        - name: DB_PASSWORD
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DB_PASSWORD
        ports:
        - containerPort: 80
```
Explicação do YAML:

env: Definimos as variáveis de ambiente que o contêiner deve usar.
valueFrom.configMapKeyRef: Faz a referência a um valor armazenado no ConfigMap.
name: O nome do ConfigMap (app-config).
key: A chave dentro do ConfigMap que corresponde ao valor da variável de ambiente.
3. Aplicar o ConfigMap e o Deployment
Após definir o ConfigMap e o Deployment, você pode aplicá-los no cluster Minikube.

Aplicar o ConfigMap:

```
kubectl apply -f configmap.yaml
```
Aplicar o Deployment:

```
kubectl apply -f nginx-deployment.yaml
```
4. Verificar o Deployment e os Pods
Depois de aplicar os arquivos, verifique se o Deployment e os Pods foram criados corretamente.

Verificar os Pods:

```
kubectl get pods
```
Verificar as variáveis de ambiente no Pod: Você pode executar o comando abaixo para verificar se as variáveis de ambiente foram injetadas corretamente dentro do contêiner:

```
kubectl exec -it <nome-do-pod> -- printenv
```
Esse comando retornará uma lista de variáveis de ambiente, incluindo as que vêm do ConfigMap (DB_HOST, DB_PORT, etc.).

5. Atualizar o ConfigMap (Se necessário)
Se você precisar atualizar as configurações armazenadas no ConfigMap, pode fazer isso e atualizar os Pods. Quando o ConfigMap é alterado, os Pods que o utilizam não são automaticamente reiniciados.

Para atualizar o ConfigMap, edite o arquivo YAML ou use o kubectl para modificar as variáveis.

Após atualizar o ConfigMap, reinicie o Deployment para que as mudanças sejam aplicadas:

```
kubectl rollout restart deployment/nginx-deployment
```
6. Excluir o ConfigMap e o Deployment (Se necessário)
Se você não precisar mais do ConfigMap ou do Deployment, pode excluí-los com:

Excluir o Deployment:

```
kubectl delete -f nginx-deployment.yaml
```
Excluir o ConfigMap:

```
kubectl delete configmap app-config
```

Como criar um serviço do tipo ClusterIP para expor uma aplicação e verificar sua conectividade dentro do cluster? 

1. Criar um Serviço do tipo ClusterIP
Primeiro, vamos criar um serviço do tipo ClusterIP que expõe a aplicação internamente no cluster.

Exemplo de um Serviço ClusterIP para uma aplicação NGINX:
Suponhamos que você já tenha um Deployment rodando uma aplicação, como o NGINX. Agora, vamos criar um serviço para expô-lo internamente.

Crie o arquivo YAML para o Serviço (nginx-service.yaml):
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80 
 clusterIP: None    
```
pot: 80       # Porta que será exposta no ClusterIP
targetPort: 80 # Porta onde o NGINX está escutando no contêiner
clusterIP: None # O tipo ClusterIP por padrão é 'None' quando não definido
metadata.name: Nome do serviço, nginx-service neste caso.
spec.selector: Seleciona os Pods que terão o tráfego encaminhado, no caso, todos os Pods com o label app: nginx.
ports: Define as portas no serviço. A porta 80 será exposta, e o tráfego será encaminhado para a porta 80 no Pod.
clusterIP: None: Especifica que o serviço será do tipo ClusterIP, que é o valor padrão quando o campo não é definido. Isso faz o serviço ser acessível apenas dentro do cluster.
2. Criar o Serviço no Cluster
Agora, vamos aplicar o arquivo YAML para criar o serviço no Kubernetes.

Salve o conteúdo acima em um arquivo chamado nginx-service.yaml.
Aplique o serviço no seu cluster:
```
kubectl apply -f nginx-service.yaml
```
3. Verificar o Serviço Criado
Para verificar se o serviço foi criado corretamente, use o comando kubectl get services ou kubectl get svc.

```
kubectl get svc
```
Esse comando retornará uma lista de serviços no cluster, e o seu serviço nginx-service deverá aparecer. O tipo será ClusterIP, e a coluna CLUSTER-IP mostrará o endereço IP do serviço dentro do cluster.

Exemplo de saída:

```
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx-service    ClusterIP   10.100.200.150   <none>        80/TCP         10s
CLUSTER-IP: O IP interno do serviço dentro do cluster (neste caso, 10.100.200.150).
PORT(S): A porta exposta pelo serviço (neste caso, 80/TCP).
```
4. Verificar a Conectividade Dentro do Cluster
Agora que o serviço foi criado, vamos verificar se ele está acessível a partir de outros Pods dentro do cluster.

4.1. Criar um Pod temporário para testar a conectividade
Você pode usar um Pod temporário, como o busybox, para testar a conectividade ao serviço nginx-service.

Crie um Pod temporário executando o seguinte comando:

```
kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
```
Esse comando criará um Pod temporário baseado na imagem busybox e lhe dará acesso a um terminal interativo.

4.2. Testar a Conectividade Usando wget ou curl
Dentro do Pod busybox, você pode usar comandos como wget ou curl para testar a conectividade com o serviço.

Para usar o wget (caso a imagem busybox tenha esse comando):

```
wget -qO- nginx-service:80
```
Ou, caso tenha o curl disponível:

```
curl nginx-service:80
```
Esses comandos vão tentar acessar o serviço nginx-service pela porta 80. Se o serviço estiver funcionando corretamente, você deverá receber a resposta do NGINX, indicando que a conectividade foi bem-sucedida.

4.3. Saída Esperada:
Se tudo estiver correto, a resposta do curl ou wget será a página inicial padrão do NGINX.

5. Excluir o Pod Temporário (Se necessário)
Depois de testar a conectividade, você pode excluir o Pod temporário com:

```
kubectl delete pod busybox
```

Como configurar um Pod com um volume persistente utilizando o armazenamento local do Minikube?

1. Habilitar o Armazenamento Local no Minikube
Antes de começar, o Minikube precisa estar configurado para usar armazenamento local. Em algumas versões do Minikube, o armazenamento local não está habilitado por padrão. No entanto, a partir das versões mais recentes, você pode usar volumes locais diretamente. Para garantir que você tenha o suporte adequado, execute o seguinte comando para verificar se o Minikube está configurado corretamente:

```
minikube start --mount-string=/tmp:/tmp --mount
```
O comando acima monta um diretório da máquina local (/tmp) no Minikube, permitindo que você use essa pasta como armazenamento persistente. Substitua /tmp com o diretório desejado.

2. Criar o PersistentVolume (PV)
O PersistentVolume (PV) define o armazenamento físico (no caso, o diretório local no Minikube) que será utilizado pelos Pods. Vamos criar um PV que aponta para o diretório montado localmente.

Exemplo de arquivo YAML para o PV (local-pv.yaml):

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 5Gi  # Defina a quantidade de armazenamento desejada
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce  # Define que o volume será montado como leitura e gravação
  persistentVolumeReclaimPolicy: Retain  # Política de retenção do PV
  storageClassName: manual
  local:
    path: /tmp/data  # Caminho do diretório local no Minikube
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - minikube  # Certifique-se de que o volume será montado no nó "minikube"
```
capacity.storage: O tamanho do volume. Você pode ajustar de acordo com suas necessidades.
local.path: O caminho para o diretório local do Minikube onde o armazenamento será feito.
nodeAffinity: Garante que o volume seja montado no nó correto (no caso, minikube).
persistentVolumeReclaimPolicy: Define o que acontece com o volume quando ele é liberado. A política Retain mantém o volume, o que é útil para casos em que você deseja verificar ou manter os dados.
3. Criar o PersistentVolumeClaim (PVC)
Agora, crie um PersistentVolumeClaim (PVC) que solicita o volume criado. O PVC é usado pelo Pod para acessar o volume.

Exemplo de arquivo YAML para o PVC (local-pvc.yaml):
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi  # Tamanho do volume solicitado, deve ser igual ou menor que o do PV
  storageClassName: manual
```
resources.requests.storage: O PVC solicita um volume com o mesmo tamanho ou menor que o PV (no caso, 5Gi).
storageClassName: O storageClassName deve corresponder ao storageClassName definido no PV (manual).
4. Criar o Pod com o PVC
Agora, crie um Pod que usará o PVC para acessar o volume persistente. O Pod pode ser qualquer tipo de aplicação, e neste exemplo, estamos criando um Pod básico com um contêiner NGINX que usa o volume.

Exemplo de arquivo YAML para o Pod (nginx-pod.yaml):
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: nginx-storage
      mountPath: /usr/share/nginx/html  # O diretório dentro do contêiner onde o volume será montado
  volumes:
  - name: nginx-storage
    persistentVolumeClaim:
      claimName: local-pvc  # O PVC que o Pod irá usar
```
volumeMounts.mountPath: O diretório dentro do contêiner onde o volume será montado.
persistentVolumeClaim.claimName: O nome do PVC criado anteriormente (local-pvc).
5. Aplicar os Arquivos YAML
Agora, aplique os arquivos YAML para criar o PV, PVC e Pod.

Criar o PersistentVolume:

```
kubectl apply -f local-pv.yaml
```
Criar o PersistentVolumeClaim:

```
kubectl apply -f local-pvc.yaml
```
Criar o Pod:

```
kubectl apply -f nginx-pod.yaml
```
6. Verificar a Criação e o Status dos Recursos
Verificar o PersistentVolume:

```
kubectl get pv
```
Verificar o PersistentVolumeClaim:

```
kubectl get pvc
```
Verificar o Pod:

```
kubectl get pod nginx-pod
```
7. Verificar a Conectividade e os Dados Persistentes
Se o Pod foi criado com sucesso, você pode verificar se o volume foi montado corretamente dentro do contêiner. Para isso, use o comando kubectl exec para acessar o Pod:

```
kubectl exec -it nginx-pod -- sh
```
Dentro do contêiner, você pode verificar se o diretório foi montado corretamente:

```
ls /usr/share/nginx/html
```
Se o volume foi montado corretamente, você verá o conteúdo do diretório /tmp/data (ou outro diretório que você tenha configurado).

8. Limpar os Recursos (Opcional)
Após a verificação, você pode excluir os recursos criados:

Excluir o Pod:

```
kubectl delete pod nginx-pod
```
Excluir o PVC:

```
kubectl delete pvc local-pvc
```
Excluir o PV:

```
kubectl delete pv local-pv
```

Como habilitar o addon do ingress no Minikube, criar um recurso Ingress e acessar uma aplicação pelo navegador?

1. Habilitar o Ingress no Minikube
Minikube vem com um addon do Ingress que você pode habilitar para criar regras de roteamento de tráfego HTTP/S no seu cluster. O Ingress permite que você defina URLs para acessar serviços internos com base em regras de URL.

Para habilitar o addon do Ingress, basta usar o seguinte comando:

```
minikube addons enable ingress
```
Esse comando habilita o NGINX Ingress Controller, que é um controlador de entrada de tráfego HTTP que o Minikube usa para rotear as requisições para os serviços corretos.

2. Criar uma Aplicação de Exemplo
Antes de criar o recurso Ingress, você precisa de uma aplicação rodando no Kubernetes. Vamos usar um exemplo simples com o NGINX.

Criar um Deployment e um Serviço para a Aplicação:
Crie um arquivo YAML para o Deployment e Serviço do NGINX (nginx-deployment-service.yaml):
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
Aplique o arquivo para criar o Deployment e o Serviço:
```
kubectl apply -f nginx-deployment-service.yaml
```
Verifique se o Deployment e o Serviço foram criados corretamente:

```
kubectl get deployments
kubectl get svc
```
Você deve ver o NGINX rodando com um serviço exposto na porta 80.

3. Criar o Recurso Ingress
Agora que você tem o serviço rodando, vamos criar um recurso Ingress que expõe a aplicação NGINX via HTTP.

Crie um arquivo YAML para o Ingress (nginx-ingress.yaml):
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: nginx.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```
host: nginx.local: Define que a URL nginx.local será usada para acessar o serviço.
path: /: Define que todas as requisições para o caminho / serão roteadas para o serviço nginx-service.
backend.service.name: O nome do serviço que o Ingress irá rotear o tráfego.
backend.service.port.number: A porta onde o serviço está exposto (porta 80 no nosso caso).
Aplique o arquivo YAML para criar o recurso Ingress:
```
kubectl apply -f nginx-ingress.yaml
```
Verifique se o recurso Ingress foi criado corretamente:

```
kubectl get ingress
```
O Ingress deve agora estar configurado para rotear as requisições para o serviço NGINX.

4. Acessar a Aplicação pelo Navegador
Agora que o Ingress está configurado, você pode acessar a aplicação usando o host definido no recurso Ingress (nginx.local).

Modificar o Arquivo hosts
Como estamos usando Minikube, o endereço nginx.local não será resolvido automaticamente para o IP do Minikube. Você precisa adicionar uma entrada no seu arquivo hosts para que o nome do host nginx.local seja resolvido para o IP do Minikube.

Obtenha o IP do Minikube:
```
minikube ip
```
Esse comando retornará o IP do Minikube, algo como 192.168.99.100.

Modifique o arquivo hosts no seu computador:
No Linux/macOS: Edite o arquivo /etc/hosts.
No Windows: Edite o arquivo C:\Windows\System32\drivers\etc\hosts.
Adicione a seguinte linha ao final do arquivo:

```
<minikube-ip> nginx.local
```
Substitua <minikube-ip> pelo IP retornado pelo comando minikube ip. Exemplo:

```
192.168.99.100 nginx.local
```
Salve o arquivo.

5. Testar a Aplicação no Navegador
Agora, abra o seu navegador e vá para o endereço http://nginx.local. Se tudo estiver configurado corretamente, você verá a página inicial do NGINX.

6. Verificar os Logs do Ingress Controller (Opcional)
Se você encontrar problemas, pode verificar os logs do Ingress Controller para mais informações.

Encontre o Pod do Ingress Controller:
```
kubectl get pods -n kube-system
```
Verifique os logs do Ingress Controller (NGINX):
```
kubectl logs <nginx-ingress-pod> -n kube-system
```
Substitua <nginx-ingress-pod> pelo nome real do Pod do Ingress Controller.

Como criar um Pod com limites e solicitações de recursos (CPU e memória) definidos?

1. O que são Solicitações e Limites de Recursos?
Solicitações de Recursos (Requests): São os recursos que o Kubernetes garante para o contêiner. O Kubernetes tentará sempre garantir que esses recursos sejam disponibilizados ao contêiner.
Limites de Recursos (Limits): São os recursos máximos que o contêiner pode consumir. Se o contêiner tentar consumir mais que o limite, o Kubernetes pode restringir a utilização de CPU ou memória, e em casos de violação de limite de memória, o contêiner pode ser finalizado.
2. Exemplo de Definição de Pod com Limites e Solicitações
Aqui está um exemplo de como você pode criar um Pod com contêineres que possuem limites e solicitações para CPU e memória:

Arquivo YAML do Pod (pod-resource-limits.yaml):
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    resources:
      requests:
        memory: "64Mi"   # Solicitação de memória (64 MiB)
        cpu: "250m"      # Solicitação de CPU (250 millicores)
      limits:
        memory: "128Mi"  # Limite de memória (128 MiB)
        cpu: "500m"      # Limite de CPU (500 millicores)
```
Explicação do YAML:
resources.requests.memory: Solicita 64 MiB de memória.
resources.requests.cpu: Solicita 250 millicores (0,25 cores de CPU).
resources.limits.memory: Define o limite de memória para 128 MiB. Se o contêiner tentar usar mais memória do que o limite, o Kubernetes poderá matar o contêiner.
resources.limits.cpu: Define o limite de CPU para 500 millicores (0,5 cores de CPU). Se o contêiner tentar usar mais CPU do que o limite, ele será restringido.
3. Aplicar o YAML e Criar o Pod
Após criar o arquivo YAML, aplique-o para criar o Pod no Kubernetes:
```
kubectl apply -f pod-resource-limits.yaml
```
4. Verificar o Pod Criado e os Recursos
Você pode verificar se os limites e as solicitações de recursos foram aplicados corretamente usando o seguinte comando:
```
kubectl describe pod nginx-pod
```
Esse comando mostrará detalhes sobre o Pod, incluindo as solicitações e limites de recursos para os contêineres.

5. Exemplo de Uso de Deployment com Limites e Solicitações
Se você estiver usando um Deployment em vez de um Pod simples, a definição de recursos será semelhante. Aqui está um exemplo de um Deployment com limites e solicitações de recursos:

Arquivo YAML do Deployment (nginx-deployment-resource-limits.yaml):
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        resources:
          requests:
            memory: "64Mi"  # Solicitação de memória
            cpu: "250m"     # Solicitação de CPU
          limits:
            memory: "128Mi" # Limite de memória
            cpu: "500m"     # Limite de CPU
```
Aplicar o Deployment:
```
kubectl apply -f nginx-deployment-resource-limits.yaml
```
Verificar o Deployment e os Pods Criados:
```
kubectl get deployments
kubectl get pods
```
Descrever o Deployment para verificar os recursos:
```
kubectl describe deployment nginx-deployment
```
6. Monitoramento dos Recursos Usados
Você pode verificar o uso real dos recursos pelos Pods utilizando o kubectl top:
```
kubectl top pod nginx-pod
```
Esse comando retornará a quantidade de CPU e memória que o Pod está consumindo atualmente.

Como verificar o uso de recursos do cluster utilizando o addon metrics-server?

1. Habilitar o Metrics Server no Minikube
O Metrics Server precisa estar habilitado no cluster para coletar e expor as métricas de recursos. Se você estiver usando o Minikube, pode habilitar o Metrics Server da seguinte forma:

Habilitar o Metrics Server no Minikube:
```
minikube addons enable metrics-server
```
Esse comando habilita o Metrics Server, que começará a coletar as métricas de CPU e memória.

2. Verificar a instalação do Metrics Server
Após habilitar o Metrics Server, você pode verificar se ele está funcionando corretamente. O Metrics Server normalmente roda como um Pod dentro do namespace kube-system.

Verifique os Pods do Metrics Server:

```
kubectl get pods -n kube-system
```
Procure por um Pod chamado metrics-server na lista. Se ele estiver funcionando corretamente, você verá o estado "Running".

Exemplo:

```
NAME                          READY   STATUS    RESTARTS   AGE
metrics-server-xxxxxxxxxx-yyyyy   1/1     Running   0          5m
```
3. Verificar o Uso de Recursos com o kubectl top
Com o Metrics Server em funcionamento, você pode usar o comando kubectl top para verificar o uso de recursos dos Pods, Nodes, Deployments e Namespaces.

Verificar o Uso de Recursos de um Node:
```
kubectl top nodes
```
Esse comando exibirá as métricas de CPU e memória usadas pelos nodes no cluster. O comando mostra as métricas de cada nó, incluindo a quantidade de CPU e memória que está sendo usada, e a quantidade total de recursos disponíveis.

Exemplo de saída:

```
NAME        CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
minikube    500m          25%   1024Mi          50%
```
Verificar o Uso de Recursos de um Pod:
```
kubectl top pod
```
Esse comando mostra o uso de CPU e memória de todos os Pods no cluster. Para um Pod específico, adicione o nome do Pod ao comando:

```
kubectl top pod <nome-do-pod>
```
Exemplo de saída para um Pod:

```
NAME             CPU(cores)   MEMORY(bytes)
nginx-pod        50m          64Mi
```
Verificar o Uso de Recursos de um Deployment:
Para verificar o uso de recursos de um Deployment específico, use o seguinte comando:

```
kubectl top deployment <nome-do-deployment>
```
Exemplo:

```
kubectl top deployment nginx-deployment
```
Verificar o Uso de Recursos de um Namespace:
Você também pode verificar o uso de recursos por namespace:

```
kubectl top namespace <nome-do-namespace>
```
4. Solucionando Problemas com o Metrics Server
Se você não conseguir ver as métricas usando o comando kubectl top, pode ser que o Metrics Server não tenha sido configurado corretamente ou esteja com problemas. Algumas possíveis causas incluem:

Metrics Server não está rodando: Verifique se o Metrics Server está rodando corretamente usando kubectl get pods -n kube-system.
Problemas de configuração: Em alguns casos, a configuração de rede ou permissões pode estar impedindo que o Metrics Server colete as métricas corretamente. Verifique os logs do Metrics Server para mais informações.
Para verificar os logs do Metrics Server, use o comando:

```
kubectl logs -n kube-system <metrics-server-pod-name>
```
Esse comando pode fornecer informações úteis sobre o que pode estar dando errado.

Como simular a indisponibilidade de um nó do Minikube e observar o comportamento do cluster?

. Verificar o Estado do Cluster
Antes de simular a falha de um nó, verifique o estado atual do seu cluster e seus recursos. Use o comando abaixo para verificar o status de seus Pods e nós:

```
kubectl get nodes
kubectl get pods -A
```
Esse comando mostrará se você tem algum Pod em execução e o estado do nó, que no Minikube será geralmente o minikube com o status Ready.

2. Simular a Indisponibilidade do Nó
Como o Minikube é um cluster de nó único, você pode simular a falha de um nó de duas maneiras:

Opção 1: Interromper o Minikube
Uma forma simples de "simular" a falha de um nó é parar o Minikube. Isso efetivamente desconectará o nó do cluster, causando a falha de todos os Pods em execução.

Interrompa o Minikube:
```
minikube stop
```
Esse comando fará com que o Minikube pare o nó virtual, tornando-o "indisponível".

Verifique o Status Após a Falha:
Após interromper o Minikube, verifique o estado do cluster:

```
kubectl get nodes
kubectl get pods -A
```
O nó deve aparecer como NotReady ou Terminating, e os Pods poderão ser reiniciados ou removidos se estiverem configurados para isso. Caso você tenha Deployments ou StatefulSets, o Kubernetes tentará agendar os Pods em outros nós (se existirem).

Opção 2: Remover o Nó Manualmente
Outra maneira de simular a falha de um nó é remover o nó manualmente usando comandos do Kubernetes, o que força o Kubernetes a reagendar os Pods em outros nós (no caso do Minikube, no mesmo nó, mas simula a falha).

Excluir o nó do cluster (simulação de falha):
```
kubectl delete node minikube
```
Esse comando força a remoção do nó do cluster. O Kubernetes então tentará reagendar os Pods em outro nó, mas como estamos em um cluster de nó único, isso pode resultar na reinicialização de Pods ou falha nas réplicas.

Verifique o Status do Cluster Após a Falha:
Após excluir o nó, use os comandos abaixo para verificar como o Kubernetes reagiu:

```
kubectl get nodes
kubectl get pods -A
```
Você pode ver que os Pods podem ser recriados ou movidos dependendo da configuração.

3. Observar o Comportamento do Cluster
Ao simular a falha de um nó, observe o comportamento do cluster:

Reagendamento de Pods: O Kubernetes tentará reagendar os Pods em outros nós, caso existam. Como o Minikube tem apenas um nó, isso pode resultar em falha de Pods caso não haja mais recursos disponíveis.

Verificação de Logs: Verifique os logs do controlador de replicação ou do Deployment para ver como ele está reagindo à falha do nó.

```
kubectl logs -f <pod-name>
```
Se você tiver Deployments configurados, o Kubernetes tentará garantir que o número desejado de réplicas de Pods seja mantido. Ou seja, se um Pod falhar, o Deployment tentará criar outro Pod, mas se não houver nós disponíveis para agendar o Pod, o Pod não será reiniciado.

Exemplo de Verificação do Reagendamento de Pods:
Descrever um Deployment:
```
kubectl describe deployment <deployment-name>
```
Esse comando mostrará como o Deployment está reagindo à falha do nó e se ele está tentando recriar Pods.

Verificar os Pods em Execução:
```
kubectl get pods
```
Observe se os Pods foram removidos e recriados ou se estão em status de CrashLoopBackOff.

4. Simular a Recuperação do Nó
Depois de simular a falha do nó, você pode tentar recuperar o Minikube ou reiniciar o nó virtual para observar como o Kubernetes se comporta ao retornar ao estado normal.

Recuperar o Minikube:
Se você usou o comando minikube stop, pode reiniciar o Minikube com:

```
minikube start
```
Esse comando vai iniciar o nó novamente, e o Kubernetes tentará reconectar e agendar os Pods no nó.

Reiniciar o Nó no Minikube:
Se você excluiu manualmente o nó com kubectl delete node minikube, você pode adicionar o nó de volta ao cluster executando o seguinte:

```
minikube start
```
Esse comando restaurará o nó e o Kubernetes começará a reagendar os Pods no nó, se necessário.

5. Monitoramento durante a Falha de Nó
Se você tiver configurado o Horizontal Pod Autoscaler (HPA) ou PodDisruptionBudgets (PDB), esses recursos podem ajudar a controlar como o cluster reage à falha de um nó e ao reagendamento de Pods. Esses recursos são mais eficazes em clusters com múltiplos nós.

HPA: Ajusta automaticamente o número de réplicas do Pod com base na utilização de recursos (como CPU ou memória).

PDB: Controla o número mínimo de Pods que devem estar disponíveis durante eventos de manutenção ou falha.

Como configurar uma aplicação para fazer uso de secrets armazenados no cluster?

Criar um Secret no Kubernetes
Primeiro, você precisa criar o Secret no cluster. Você pode criar o Secret de várias maneiras, como usando o comando kubectl ou um arquivo YAML.

Criando um Secret com o comando kubectl
Por exemplo, vamos criar um Secret para armazenar uma senha e um nome de usuário para um banco de dados:

```
kubectl create secret generic my-db-secret \
  --from-literal=db-user=myuser \
  --from-literal=db-password=mypassword
```
Esse comando cria um Secret chamado my-db-secret, com as chaves db-user e db-password, que armazenam respectivamente myuser e mypassword.

Criando um Secret usando um arquivo YAML
Você também pode criar o Secret definindo-o em um arquivo YAML:

```
apiVersion: v1
kind: Secret
metadata:
  name: my-db-secret
type: Opaque
data:
  db-user: bXl1c2Vy  # 'myuser' codificado em base64
  db-password: bXlwYXNzd29yZA==  # 'mypassword' codificado em base64
```
Para criar o Secret a partir do arquivo YAML, use o comando:

```
kubectl apply -f secret.yaml
```
2: Usar o Secret em um Deployment
Após criar o Secret, você pode usá-lo em um Deployment de diferentes maneiras. A forma mais comum de acessar os Secrets é via variáveis de ambiente ou volumes.

Opção 1: Usar o Secret como variáveis de ambiente
Aqui está um exemplo de como usar o Secret como variáveis de ambiente em um contêiner dentro de um Deployment.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: nginx
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: my-db-secret
              key: db-user
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-db-secret
              key: db-password
```
Neste exemplo, os valores de DB_USER e DB_PASSWORD serão extraídos do Secret my-db-secret e injetados como variáveis de ambiente no contêiner. O Kubernetes irá buscar as chaves db-user e db-password no Secret e passá-las como variáveis de ambiente para o contêiner.

Opção 2: Usar o Secret como volume
Você também pode montar o Secret como um volume em um Pod. Cada chave do Secret será exposta como um arquivo no volume, onde o nome do arquivo será a chave e o conteúdo do arquivo será o valor.

Aqui está um exemplo de como montar o Secret como um volume:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: nginx
        volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: secret-volume
        secret:
          secretName: my-db-secret
```
Neste exemplo, o Secret my-db-secret será montado no diretório /etc/secrets do contêiner. Dentro deste diretório, o contêiner encontrará dois arquivos:

/etc/secrets/db-user com o valor de myuser
/etc/secrets/db-password com o valor de mypassword
O contêiner pode então acessar esses arquivos diretamente para obter as informações sensíveis.

3: Verificar a Configuração
Após configurar o Deployment, você pode verificar se o Secret foi corretamente injetado no contêiner.

Verificar os Pods em execução:
```
kubectl get pods
```
Verificar as variáveis de ambiente do contêiner:
Se você configurou o Secret como variáveis de ambiente, pode verificar as variáveis de ambiente do contêiner:

```
kubectl exec -it <pod-name> -- env
```
Verificar o volume montado:
Se você montou o Secret como volume, pode verificar se os arquivos estão presentes dentro do diretório do contêiner:

```
kubectl exec -it <pod-name> -- ls /etc/secrets
```
4: Melhorando a Segurança (Opções Avançadas)
Ao usar Secrets em Kubernetes, há algumas boas práticas para garantir que suas informações sensíveis sejam tratadas de maneira segura:

Encrypt Secrets: O Kubernetes pode ser configurado para criptografar Secrets armazenados no etcd (o banco de dados distribuído do Kubernetes). Isso ajuda a proteger os Secrets em repouso.
RBAC (Role-Based Access Control): Use RBAC para limitar quem pode acessar e visualizar os Secrets no cluster. Apenas usuários ou serviços que realmente necessitam dos Secrets devem ter permissões para acessá-los.
Restrição de leitura de Secrets: Configure permissões adequadas para garantir que apenas os Pods ou namespaces específicos possam acessar determinados Secrets.

Como criar um Job no Kubernetes para executar uma tarefa pontual no cluster do Minikube?

1: Criar um Job no Kubernetes
Primeiro, você cria um arquivo YAML que define o Job. Um Job pode ser configurado para rodar um contêiner e garantir que a execução seja completada com sucesso. Aqui está um exemplo de um Job simples que executa um contêiner do nginx para realizar uma tarefa pontual.

Exemplo de arquivo YAML para o Job:
```
apiVersion: batch/v1
kind: Job
metadata:
  name: my-task-job
spec:
  template:
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        command: ["echo", "Hello, Kubernetes!"]
      restartPolicy: Never
  backoffLimit: 4
```
Explicação do YAML:
apiVersion: A versão da API que estamos usando para o recurso, no caso é batch/v1, que é a versão para Jobs no Kubernetes.
kind: O tipo do recurso, que neste caso é um Job.
metadata: Definições de metadados para o Job, como o name (nome do Job).
spec: A especificação do Job.
template: Define o Pod que será executado pelo Job. Aqui você define um contêiner com a imagem do nginx, mas você pode usar qualquer imagem necessária para sua tarefa.
command: O comando que será executado pelo contêiner, que neste caso é apenas echo "Hello, Kubernetes!".
restartPolicy: Never: Garante que o Pod não será reiniciado após a execução, o que é típico para Jobs.
backoffLimit: 4: O número de tentativas que o Kubernetes fará para executar o Job em caso de falha. Após atingir esse limite, o Job será marcado como falho.
 2: Aplicar o Job no Cluster
Agora que você tem o arquivo YAML, pode aplicar o Job no cluster do Minikube.

```
kubectl apply -f job.yaml
```
Esse comando criará o Job no seu cluster. O Kubernetes irá garantir que o comando especificado no contêiner seja executado até a conclusão.

 3: Verificar o Status do Job
Após o Job ser criado, você pode verificar o status dele para ver se ele foi executado corretamente.

Verificar os Jobs em execução:
```
kubectl get jobs
```
Verificar os Pods criados pelo Job:
O Kubernetes cria automaticamente Pods para executar o Job. Você pode verificar os Pods criados:

```
kubectl get pods
```
Você verá um Pod com um nome gerado automaticamente associado ao Job.

Verificar os Logs do Pod:
Para ver a saída do Job, você pode verificar os logs do Pod que foi criado para o Job:

```
kubectl logs <pod-name>
```
Substitua <pod-name> pelo nome do Pod que você obteve com o comando kubectl get pods.

Verificar o status do Job:
O Job terá um status de succeeded ou failed. Você pode verificar se o Job foi concluído com sucesso com:

```
kubectl describe job my-task-job
```
Esse comando fornecerá mais detalhes sobre o Job, incluindo o número de execuções bem-sucedidas e falhadas.

4: Excluir o Job (opcional)
Após a execução do Job, você pode querer removê-lo do cluster. Para isso, você pode deletar o Job e os Pods criados automaticamente.

```
kubectl delete job my-task-job
```
Esse comando removerá o Job e seus Pods associados.

