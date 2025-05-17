# Pod

## Conceitos Iniciais

O **Pod** é a menor unidade na estrutura do Kubernetes. O **Pod** é um host lógico com recursos de rede e armazenamento. Aqui cabe algumas resalvas sobre os recursos de Rede e Armazenamento.

### Rede

Cada Pod tem seu próprio namespace de rede, e os containers dentro do Pod compartilham esse namespace. O Pod também tem seu endereço de IP único. Todos os containers dentro desse Pod compartilham esse IP.

> **Importante**: Todos os containers dentro do Pod compartilham o endereço IP do Pod.

Mas aí surge o questionamento. Se um container A precisa acessar algum recurso do container B dentro do Pod, como ocorre a comunicação?

```
Como todos os containers estão no mesmo namespace de rede, eles se comunicam como se estivessem no mesmo host:

# Container A dentro do Pod pode acessar o Container B assim:
curl http://localhost:8080  # se o B expôs essa porta

```

---

Outra dúvida que pode surgir é, como é feita a comunicação entre os Pods? Como cada Pod tem seu IP e se comunica com outros Pods via esse IP.



### Armazenamento

Os Pods podem compartilhar volumes se forem montados em comum (como um volume persistente), mas não compartilham automaticamente o armazenamento com outros nodes ou Pods.

---

É importante notar que cada **Pod** tem seu próprio conjunto de namespaces e cgroups para acessar os recursos da máquina.

> **Nota**: Vale lembrar que assim como no Docker, se não definirmos limites de cgroups um Pod pode acabar monopolizando os recursos e afetando o desempenho de outros Pods.

## Ciclo de Vida

O clico de vida de um Pod envolve alguns estados. São eles

- `Pending`
- `Running`
- `Succeeded`
- `Failed`
- `Unknown`

### Pending

O Pod foi aceito pela API do Kubernetes, mas ainda não foi agendado ou iniciado completamente.

Isso pode significar

- O scheduler ainda não decidiu onde o Pod vai rodar.

- Está aguardando download da imagem do container.

- Recursos (CPU/memória) ainda não estão disponíveis.

### Running

O Pod foi agendado para um node e os containers foram criados com sucesso.

### Succeeded

Todos os containers terminaram com código de saída 0 (sem erro). Tipicamente, ocorre em Pods de execução única (como Jobs ou scripts curtos).

### Failed

Um ou mais containers terminaram com erro (código diferente de 0).

Também pode acontecer se

- Um container não pôde ser iniciado.

- Excedeu limites de recursos.

- Foi finalizado de forma anormal.

### Unknown

O estado do Pod não pode ser determinado, geralmente por problemas de comunicação com o Node onde ele está (Node offline, falha no kubelet, etc.).

## O que é o CrashLoopBackoff?

O **crashLoopBakoof** é um status de erro que aparece quando o container dentro do Pod está falhando repetidamente ao iniciar.

---

**Fluxo**

O container inicia, crasha, ou seja, falha e o Kubernetes tenta reiniciá-lo. Após algumas falhas, o Kubernetes entra no modo **back-off**, ou seja, ele espera mais tempo antes de tentar novamente.

> **Nota**: Esse tempo cresce exponencialmente.

Os motivos pelos quais um container fica com esse status podem ser os mais diversos. É interessante sempre fazer uma investigação, analisar logs, iniciar o container com um processo de sleep rodando como principal para acessar o container e fazer uma depuração in loco.

## YAML de um Pod

Esta é a estrutura de um arquivo **.yaml** de um Pod

```bash
apiVersion: v1
kind: Pod
metadata:
  name: $NomeDoPod
spec:
  containers:
    - name: $NomeDoContainer
      image: $ImagemDoContainer
```

## Init Containers e Sidecars

### Init Containers

**Init Containers** são containers especiais que executam sequencialmente antes dos containers principais. Cada init container deve finalizar com sucesso `(exit 0)` para o próximo começar.

Tá, mas e se o init container falhar? Se isso ocorrer, o Kubernetes reinicia o Pod e reexecuta os Init Containers.

Mas por qual motivo usar **Init Containers**? Aqui estão alguns motivos

- Preparar diretórios e volumes, por exemplo, criar diretórios em um volume antes da aplicação começar a executar.
- Esperar dependências externas, por exemplo, aguardar um banco de dados ou um serviço externo estar pronto.
- Validar ou baixar configurações.

---

**Exemplo**:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: $NomeDoPod
spec:
  initContainers:
    - name: init-banco
      image: busybox
      command: ['sh', '-c', 'until nc -z db 5432; do echo aguardando; sleep 2; done']
  containers:
    - name: app
      image: $NomeDaImagemDaAPI
      ports:
        - containerPort: 8080
```

> Esse Init Container só libera a aplicação quando o banco (db) estiver ouvindo na porta 5432.

### Sidecar

O **Sidecar** é um container que roda em paralelo ao container principal dentro do Pod. Ele compartilha os recursos do Pod, como rede, volume e o clico de vida com os demais containers.

Os casos de uso mais comuns para um Sidecar são

- `logs`, ou seja, um container que coleta logs de um volume compartilhado com o container principal e os envia para outro local.

- `monitoramento`, ou seja, um sidecar com agente de monitoramento como Prometheus Exporter.

---

**Exemplo**

```bash
apiVersion: v1
kind: Pod
metadata:
  name: $NomeDoPodSidecar
spec:
  containers:
    - name: app
      image: $NomeDaimagemAPI
      volumeMounts:
        - name: logs
          mountPath: /var/log/app
    - name: log-forwarder
      image: fluentbit
      volumeMounts:
        - name: logs
          mountPath: /fluentbit/logs
  volumes:
    - name: logs
      emptyDir: {}
```

> Aqui, o container de logs acessa o mesmo volume que a aplicação e envia os arquivos para fora.

Há diversas possibilidades de utilizar um Sidecar, então estude caba bom rsrs.

## Conclusão

Como sempre, é importante consultar a documentação do [Kubenetes](https://kubernetes.io/docs/concepts/workloads/pods/) sobre os Pods.