# Probes

O **kubelet** usa as Probes para decidir quando irá reiniciar um container dentro de um Pod. Temos três tipos de Probes: livenessProbe, readinessProbe e startupProbe.

> **Nota**: Um padrão comum é usar as Probes com um endpoint HTTP.

## livenessProbe

O Liveness Probe determina quando um container será restartado. Ele faz isso checando com uma periodicidade se o container ainda está "vivo".

## readinessProbe

O Readiness Probe determina quando um container está pronto para receber tráfego. De forma parecida com o **livenessProbe** o readinessProbe verifica se o container ainda pode receber solicitações.

## startupProbe

Às vezes, precisamos lidar com aplicativos que exigem tempo de inicialização adicional na primeira inicialização. Nesses casos, pode ser complicado configurar parâmetros de sondagem de atividade sem comprometer a resposta. A solução é configurar o startupProbe para cobrir o pior caso de tempo de inicialização.

---

## Formas De Uso

Há três formas de usar as Probes, ou seja, livenessProbe, readinessProbe e startupProbe. Essas formas são com **httpGet**, **tcpSocket**, **exec**.

### httpGet

Nesse modo, o **httpGet** faz uma requisição GET para um endpoint dentro do container. Seguindo essa forma devemos configurar um endpoint especifico dentro da nossa aplicação que nos retorne quando ela estiver normal.

**Exemplo** 

```text
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

### tcpSocket

Verifica se a porta TCP do contêiner está aceitando conexões. Útil para aplicações que não têm um endpoint HTTP específico, mas escutam em uma porta.

```text
livenessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 5
  periodSeconds: 10
```

### exec 

Executa um comando dentro do contêiner. Se o comando retornar código 0, está ok. Se for diferente, o contêiner será reiniciado.

```text
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 10
```

# Conclusão

Cheche essa seção na documentação para tirar duvídas: [Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/).