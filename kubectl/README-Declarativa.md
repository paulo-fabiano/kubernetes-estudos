# Kubeclt - Forma Declarariva

## Conceitos Iniciais

A abordagem declarativa é a abordagem recomendada para gerenciar ambientes de produção em um cluster Kubernetes. Com essa abordagem nos usamos os famosos **manifestos** que podem ser escritos no padrão **.yaml** ou **.json**.

> **Nota**: Atualmente o formato `.yaml` é o mais utilizado.

## Comandos

O principal comando que nós iremos utilizar na abordagem declarativa é o **apply -f**.

```bash
kubectl apply -f $NomeDoArquivo
```

> **Nota**: A variável $NomeDoArquivo é o nome do arquivo que irá criar o respectivo objeto do Kubernetes. Seja ele um Pod, um Deployment, um Service e etc.

## Estrutura do Arquivo .YAML

A estrutura de um arquivo **.yaml** é basicamente a seguinte

```
apiVersion:
kind:
metadata:
spec:
```

Todos os arquivos **.yaml** seguem essa estrutura básica. Cada objeto tem suas particularidades então é sempre interessante consultar a documentação do [Kubernetes](http://kubernetes.io/) quando for criar um objeto.

---

**Exemplo**: Criando um Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

Para criarmos esse Pod asamos o comando **apply -f**

```bash
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```

Este exemplo foi tirado da documentação do Kubernetes: [simple-pod.yaml](https://kubernetes.io/docs/concepts/workloads/pods/).

---

## Componentes do YAML

### apiVersion

Indica qual a versão da API do Kubernetes estamos usando para o recurso.

### kind

O **kind** informa o tipo de recurso que estamos criando. Há diversos tipos de recursos como Pods, Services, Crons e etc.

### metadata

O **metadata** contém informações sobre o recurso, como

- `name`: nome do recurso (obrigatório)
- `labels`: etiquetas para identificação (opcional, porém altamente recomendado)
- `namespace`: em qual namespace o recurso está (opcional)


### spec

O **spec** define a especificação do recurso.

- Para um Pod: os containers, imagens, portas...

- Para um Deployment: quantas réplicas, o template do pod...

- Para um Service: tipo de serviço, portas, selector...

---

É interessante sempre consultar a documentação do [Kubernetes](kubernetes.io) quando formos criar um recurso.

**Exemplo Completo**

```
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
  labels:
    app: minha-app
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

## Conclusão

Após todas essas explicações finalizamos o módulo sobre interação **declaratica** com Cluster Kubernetes.