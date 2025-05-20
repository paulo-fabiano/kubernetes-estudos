# Arquitetura do K8S

## Visão Geral

Sobre a Arquiterura do Kubernetes temos dois conceitos muito importantes: **Control Plane ou Master** e **Data Plane ou Worker**.

                        +-----------------------------+
                        |     CONTROL PLANE (Master)  |
                        +-----------------------------+
                        |  kube-apiserver             |
                        |  kube-scheduler             |
                        |  kube-controller-manager    |
                        |  etcd                       |
                        |  cloud-controller-mgr       |
                        +-----------------------------+
                
                         ⇅ Comunicação via API Server
                
                        +-----------------------------+
                        |     DATA PLANE (Worker)     |
                        +-----------------------------+
                        |  kubelet                    |
                        |  kube-proxy                 |
                        |  container runtime          |
                        |  (Pods)                     |
                        +-----------------------------+


### kube-apiserver

O **kube-apiserver** expõe uma API para que seja possível haver intereação com o **Nó Master**

### kube-scheduler

O **kube-scheduler** decide em qual Node os Pods serão executados com base em recursos de CPU, memória, rede e etc.

> **Nota**: Quando criar um novo Pod e não especificamos o Node que ele irá rodar, o scheduler faz essa escolha.

### kube-controller-manager

O **kube-controller-manager** faz o controle do Cluster para verificar se ele está no estado desejado. O **kube-controller-manager** compara o estado real com o estado desejado e age quando é necessário fazer correções.

### etcd

O **etcd** é o banco de dados distribuído que armazena o estado atual do Cluster. Tudo que é de configurações críticas, definição de Pods, Nodes e etc estão armazenados no **etcd**. É interessante fazer o backup dele com frequência e separá-lo em um Nó a parte.

### cloud-controller-mgr

O **cloud-controller-manager** faz a integração do Cluster com os provedores de Cloud, no caso ele é utilizado por provedores como AWS, GCP, AZURE.

> **Nota**:  Em ambientes on-premises ou locais, esse componente geralmente não é necessário ou é substituído por controladores customizados.

> **Importante**: Quando estamos utilizando serviços de Kubernetes na nuvem nós não temos controle ao Control Plane, só aos Nodes, pois quem faz toda a parte de administração deste Control Plane é o provedor do serviço.

### kubelet

O **kubelet** é o agente de cada Node, ele se conecta ao **kube-apiserver** para obter intruções, informar status de health check, liveness probe e etc. 

O **kubelet** também interage com o runtime, seja ele o **containerd**, **Docker** ou outros para crias os containers.

### kubeproxy

O **kubeproxy** é um proxy de rede que roda em cada Nó do nosso cluster, implementado a lógica do Service, gerenciando o tráfego de rede entre os Pods.

## Conclusão

Para mais detalhes da arquitetura do Kubernetes consulta a seção [Arquitetura do Cluster](https://kubernetes.io/docs/concepts/architecture/).