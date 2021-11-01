# **k8s -  Self Healing**

O kubernetes possibilita escalabilidade e resiliência por meio do `Replicaset`, no entanto, para que estes funcionem corretamente é necessário ensiar ao Kubernetes como identificar se um container já concluiu seu carregamento inicial, se está saudável e também se está pronto para receber requisições. Tudo isso é possível por meio dos recursos `Startup Probe`, `Liveness Probe` e `Readiness Probe`.


## **Startup Probe** 
Seu uso é recomendado para aplicações cujo o carregamento pode ser demorado, como por exemplo o Sql Server. Seu objetivo, quando configurado, é evitar que o `liveness probe` seja acionado antes mesmo da aplicação estar completamente pronta. Caso contrário, o `linveness` receberia como resposta que o container não está saudável fazendo com o que este seja reiniciado imediatamente e isso colocaria o container em um loop infinito de reinicializações.

Sua verificação pode ser realizada no endpoint de `health check`.
```yaml
    spec:
        containers:
            ...
            livenessProbe:
                httpGet:
                    path: /health
                    port: 80
                    scheme: HTTP                
                failureThreshold: 30
                periodSeconds: 10
```


## **Liveness Probe** 
Utilizado pelo kubernetes para identificar se o container de um determinado pod está saudável. Esta verificação pode ser feita por meio do **HttpGet** para um endpoint de health check ou através do **TcpSocket** escutando uma determinada porta ou ainda com o **Exec** utilzando algum commando shell que especifique se o pod está saudável ou não.

```yaml
    spec:
        containers:
            ...
            livenessProbe:
                httpGet:
                    path: /health
                    port: 80
                    scheme: HTTP
                timeoutSeconds: 1
                failureThreshold: 3
                periodSeconds: 5
```

 

## **Readiness Probe**
Seu uso é indicado para aplicações que apesar de já estarem saudáveis ainda precisam de algum processamento mais demorado para então começar a receber requisições, por exemplo, machine learning.

Não pode utilizar o mesmo endpoint de heatlh check, pois tem propósito diferente. É altamente recomendado um endpoint que retorne se a aplicação está ou não pronta para receber requisições.

```yaml
    spec:
        containers:
            ...
            livenessProbe:
                httpGet:
                    path: /read
                    port: 80
                    scheme: HTTP
                timeoutSeconds: 1
                failureThreshold: 3
                periodSeconds: 5
```

Em um cenário que haja apenas um pod e o container não está pronto, todos os endpoints da api deixarão de responder, pois o bind entre o service o pod é interrompido enquanto este não pode receber requisições. Isso pode ser verificado com o seguinte commando:

```bash
$ kubectl get endpoints
NAME                           ENDPOINTS           AGE
kubernetes                     100.65.1.125:443    21m
mongodb-service                10.244.1.57:27017   14m
pedelogo-catalog-api-service                       14m
```
Observe que o pod da api não possui um endpoint não está pronto, como podemos ver abaixo
```bash
$ kubectl get pods
NAME                                               READY   STATUS    RESTARTS   AGE 
mongodb-deployment-795b9c646f-b882n                1/1     Running   0          19m 
pedelogo-catalog-api-deployment-5bdf8d458c-9b4lp   0/1     Running   0          7m1s
```
> Este cenário foi apontado somente para exemplificar o uso do Readiness Probe. Em aplicações de produção é altamente recomendável que haje redundância de containers a fim de evitar indisponibilidade da aplicação.



# Referências
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
