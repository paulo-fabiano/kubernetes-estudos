# Como o Kubernetes realmente cria um container

Quando criamos um Pod no Kubernetes, não é o Control Plane que "cria" o container.  
Quem executa esse trabalho é o **Node** onde o Pod foi agendado.

## Fluxo de Criação

- O **kube-apiserver** recebe o manifesto do Pod (YAML) via `kubectl apply`.
- O **kube-scheduler** escolhe em qual Node o Pod vai rodar.
- O Pod entra no estado `Pending`.

> **Nota**: Para mais informações sobre Pods consulte a seção sobre [Pods](./../pod/README.md) neste repositório.

## Agora entra o Node em ação

### `kubelet` (que roda no Node)

- O **kubelet** detecta que há um novo Pod agendado para ele.
- Vai até o **kube-apiserver** do Kubernetes buscar os detalhes do Pod.
- Valida o que precisa ser criado (container, volume, etc).

### `kubelet` chama o Container Runtime via **CRI**

- O `kubelet` **não executa containers diretamente**.
- Ele usa a **CRI (Container Runtime Interface)** para se comunicar com o runtime.

#### CRI

O **CRI (Container Runtime Interface)** é uma interface padronizada criada pelo Kubernetes para permitir que o **kubelet** se comunique com diferentes runtimes de container.

```text
Mas por qual motivo o CRI existe?

Antes do CRI, o kubelet precisava se integrar diretamente com o Docker, o que limitava o Kubernetes a um único runtime.

Com o CRI, o Kubernetes pôde:

- Ser independente do Docker

- Suportar múltiplos runtimes (ex: containerd, cri-o, gVisor)

- Ter uma abstração limpa e modular
```

```text
Como o CRI funciona

O kubelet não executa containers, ele envia comandos como “crie esse container” ou “delete esse Pod” via gRPC para o runtime, usando a CRI. 

O runtime (ex: containerd) executa as instruções, faz o download da imagem, cria os namespaces, monta o sistema de arquivos e executa o container via runc.
```

### O **Container Runtime** (como `containerd` ou `cri-o`)

- Recebe os comandos via CRI.
- Puxa a imagem do registry (ex: `nginx:latest`).
- Usa uma ferramenta como `runc` para **criar e isolar o container** usando:
  - Namespaces
  - cgroups
  - rootfs

### O container está rodando

- O `kubelet` agora monitora:
  - Se o container está saudável
  - Se precisa ser reiniciado
  - Se as probes (`liveness`, `readiness`) estão OK

---

## Visão simplificada do fluxo:

```text
                    API Server → Scheduler → Node escolhido
                                         ↓
                                   +-------------+
                                   |   Kubelet   |
                                   +-------------+
                                         ↓ CRI
                                   +-------------+
                                   | Runtime     | ← containerd, cri-o
                                   +-------------+
                                         ↓
                                   +-------------+
                                   | Container   | ← runc + namespaces
                                   +-------------+
