# Kubectl - Forma Imperátiva 

## Conceitos Iniciais

O comando **kubectl** é uma ferramenta de linha de comando para interagirmos e gerenciarmos nosso cluster Kubernetes. Com ele nós podemos criar, atualizar, deletar recursos. Além disso podemos inspecionar o estado do cluster e executar comandos.

## Comandos

Os principais comandos do **kubectl** são: 

- `kubectl get`
- `kubectl describe`
- `kubectl create`
- `kubectl delete`

Com esses comandos podemos criar objetos de forma **imperátiva**, ou seja, passando o comando com parâmetros do **kubectl**. Há diversos comandos, então sempre consulte a documentação oficial do [Kubernetes](http://kubernetes.io/). Abaixo irei falar sobre alguns comandos.

### kubectl get

O comando **get** busca os objetos do nosso cluster, sejam eles Pods, Nodes, Namespaces, Services, ReplicaSets e etc.

```bash
kubectl get $OBJETO
```

> **Nota**: O valor da variável $OBJETO é o objeto que queremos buscar.

Mas aí vem a dúvida. Vamos pensar no cenário onde estamos buscando Pods. Se eu usar pura e simplimente o comando 

```bash
kubectl get pods
```

a saída do comando serão todos os Pods?

Bom a resposta é NÃO! Pois quando não especificamos o namespace o comando traz somente os Pods do namespace **default**. Vamos supor que nós temos um namespace chamado `homologacao`, caso nós precisemos buscar os Pods deste namespace usamos o comando 

```bash
kubectl get pods -n homologacao
```

---

Mas aí surge outra pergunta? Como ver todos os Pods de todos os namespaces? A resposta é bastante simples. Para vermos todos os Pods do nosso Cluster usamos o parâmetro `-A`.

```bash
kubectl get pods -A
```

Com esse parâmetro veremos todos os Pods presentes em nosso Cluster.

---

Surge outra dúvida, como ver informações detalhadas dos Pods? Para isso usamos o comando 

```bash
kubectl get pods -n homologacao -o wide
```

O parâmetro `-o wide` nos traz informações como IP do Pod, o Node que ele está rodando, a imagem utilizada e etc.

---

Agora pense no caso de queremos filtrar os Pods por uma ou mais labels. Para isso usamos o comando

```bash
kubectl get pods -l app=nginx
```

> **Nota**: Aqui o filtro seria app=nginx, com isso obteremos todos os Pods que possuem essa label.

### kubectl describe

O comando **describe** descreve o objeto do Kubernetes. Ele nos fornece um detalhamento do recurso que queremos analisar.

```bash
kubectl describe $OBJETO
```

> **Nota**: O valor da variável $OBJETO é o objeto que queremos buscar.

---

O **describe** nos mostra informações como:

- Nome
- Namespace
- Labels
- Annotations
- Eventos
- Restart Count
- Últimos erros
- Status detalhados

### kubectl delete

O comando **delete** apaga um determinado objeto ou recurso do Kubernetes. É interessante notar que podemos tanto deletar passando o nome do recurso ou o arquivo `.yaml`.

```bash
kubectl delete pod $NomeDoPod
```

> **Nota**: A variável $NomeDoPod recebe o nome do Pod que iremos deletar.

---

```bash
kubectl delete deployment $NomeDoDeployment -n $NomeDoNamespace
```

> **Nota**: Aqui segue o mesmo princípio. As variáveis são os nomes dos recursos.

--- 

Para deletar outros objetos ou recursos o princípio é o mesmo. 

```bash
kubectl delete $TipoDoRecurso $NomedoRecurso -n $NomeDoNamespace
```

### kubectl create

Com o comando **create** podemos criar todo e qualquer tipo de recusos,seja Pods, Deployments, Services e etc.

```bash
kubectl create deployment nginx --image=nginx:latest
```

> **Nota**: Nesse caso estou criando um Deployment do nginx.

Poderia ainda criar em um namespace espacífico passando o parâmetro `- n $NomeDoNamespace`.

### kubectl scale

Pense agora que nós iremos escalar o Deployment que acabamos de criar, para fazer isso usamos o comando

```bash
kubectl scale deployment nginx --replicas=5
```

> **Nota**: Nessa situação estamos escalando nosso nginx para cinco réplicas no nosso Deployment.

---

Podemos ainda configurar um escalonamento com base em métricas de usdo CPU e/ou memória.

```bash
kubectl autoscale deployment $NomeDoDeployment --min=2 --max=5 --cpu-percent=80
```

Nesse caso, quando o uso de CPU chegar a 80%, os Pods poderão ser escalados até 5 Pods.

### kubectl run 

O comando **run** cria um Pod, isso mesmo, somente um Pod.

> **Nota**: Dependendo da versão ele pode criar um Deployment também.

Com isso, podemos criar Pods rapidamenta para testar uma aplicação, validar uma ideia, ou fazer testes rápidos.

```bash
kubectl run $NomeDoPod --image=$NomeDaImagem
```

Supondo que vamos criar um Pod do **nginx**, podemos usar o seguinte comando

```bash
kubectl run nginx --image=nginx
```

### kubectl expose

O comando **expose** cria um Service para expor um recurso.

```bash
kubectl expose $TipoDoRecurso $NomeDoRecurso --port=$Porta --type=$TipoDeService
```
> **Nota**: O parâmetro `--port=$Port` define a porta do Service.

--- 

**Exemplo**

```bash
kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort
```

Nesse exemplo nós estamos criando um Service para acessar o Deployent, bom o Pod do Deployment rsrs. Nesse caso

- --port é a porta do Service.
- --target-port é a porta do Pod

O Kubernetes escolhe automaticamente uma porta no range de 30000 a 32767. Com isso podemos acessa o nosso Pod com o IP do Node.

No caso para fazer um teste em um Cluster com Minikube você pode ver o IP do Node com o comando

```bash
kubectl get nodes -o wide
```

Procura pela informação que traz o IP do Node.

Agora façamos um **curl** e vejamos a mágica

```bash
curl http://<IPDoNode>:$PortaGeradaPeloK8s
```

```
Saída


<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

### kubectl exec 

O comando **exec** executa comandos dentro de um Pod.

```bash
kubectl exec -it $NomeDoPod --/bin/bash
```

> **Nota**: Com esse comando abrimos um terminal iterativo dentro do Pod

Similarmente ao `docker exec`, podemos executar vários comandos nos nossos Pods.

### kubectl cp

O comando **cp** copia arquivos de um local para um Pod.

```bash
kubectl cp ./$NomeDoArquivo $NomeDoPod:$DiratorioDeDestino
```

### kubectl logs

O comando **logs** nos permite vizualizar os logs de um Pod.

```bash
kubectl logs $NomeDoPod
```

--- 

Caso o Pod tenha múltiplos containers passamos o parâmetro `-c $NomeDoContainer` para especificar o container.

```bash
kubectl logs $NomeDoPod -c $NomeDoContainer
```

---

Um parâmetro interessante também é o `--previos`, com ele podemos ver os logs do container antes da falha ocorrer.

```bash
kubectl logs $NomeDoPod --previous
```

## Conclusão

Após todas essas explicações finalizamos o módulo sobre interação **imperativa** com Cluster Kubernetes.
