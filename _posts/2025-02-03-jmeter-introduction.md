---
title: "Aprenda a Criar Testes de Performance no JMeter Passo a Passo"
categories:
  - Java
  - Ferramentas
  - JMeter
  - Testes
  - Performance
tags:
  - java
  - ferramentas
  - testes
  - performance
toc: true
toc_label: "Conteúdo"
toc_icon: "file"
---

Neste _post_ mostro como criar um teste de performance básico utilizando JMeter.

[Apache JMeter]() (ou apenas _JMeter_) é uma ferramenta para testes de carga desenvolvida e mantida pela [Apache Foundation](). 

Usando JMeter é possível simular cenários onde seu sistema é requisitado por diversos usuários ao mesmo tempo e assim ter uma ideia de como seu site/API se comporta com uma determinada quantidade de usuários. 

### Instalando JMeter no Windows

O JMeter não possui um .exe, mas para executar o aplicativo é bem simples. 

Acesse a [página de downloads](https://jmeter.apache.org/download_jmeter.cgi#binaries) e baixe o arquivo .zip

![baixando jmeter do site](/assets/images/2025-02-03-img/download.png){: .align-center}

Descompacte o arquivo, acesse a pasta `bin/` e dê duplo clique no arquivo `jmeter.bat`

![executando arquivo .bat do jmeter](/assets/images/2025-02-03-img/arquivo-bat.png){: .align-center}

### O Cenário

Imagine que você precisou construir um microsserviço com apenas um endpoint `/login` que receberá uma _request_ toda vez que um usuário fizer login no sistema que está sendo desenvolvido. 

Como um desenvolvedor precavido você questionou quantos usuários farão login na plataforma diariamente, qual o horário de maior demanda, quantos usuários a plataforma tem cadastrados ou qual a expectativa inicial da empresa (todas essas informações vão te ajudar a decidir a quantidade de memória do seu servidor, quantidade de máquinas no cluster, tamanho do banco de dados, enfim, informações que te ajudem a construir a aplicação de maneira escalável e com o menor custo possível).

Seu gerente então respondeu: 

- 20.000 usuários farão login por dia
- No horário de maior demanda, entre 09:00 e 09:05 é esperado 15.000 logins, ou seja, uma média de 50 logins por segundo
 

Vamos criar nossa aplicação agora considerando pelo menos o dobro disso: 
`
- 40.000 usuários por dia
- No horário de maior demanda 30.000 logins, ou seja, uma média de 100 logins por segundo
` 

### Criando um Plano de Testes

O `Test Plan` (Plano de Teste) é a raiz do nosso teste, dentro dele teremos que configurar as informações do endpoint que vamos testar, as condições de testes (quantos usuários, qual a frequência das requisições, etc) e informações de saída do nosso teste (histórico de chamadas, gráficos, etc). 

Vou nomear o meu teste de `teste_performance_login` (não esqueça de salvar seu plano de teste `CTRL` + `S`)

![modificando nome do teste](/assets/images/2025-02-03-img/test-name.png){: .align-center}

Com o botão direito no _test plan_ clique em `Add` > `Threads (Users)` > `Thread Group`

![adicionando thread group ao teste](/assets/images/2025-02-03-img/thread-group.png){: .align-center}

Vamos nomear o grupo de usuários como `Login User Group` , preencher com as seguintes informações e salvar:  

- `Number of Threads (users)` = 100
- `Infinite` = Marcar
- `Same user on each iteration` = Desmarcar
- `Specify Thread Lifetime` = Marcar
- `Duration (seconds)` = 300

![preenchendo valores no thread-group](/assets/images/2025-02-03-img/thread-group-filled.png){: .align-center}

Marcamos **Loop Count** como `Infinite`, pois a propriedade **Duration (seconds)** controlará o tempo de execução do teste.
Em **Number of Threads** colocamos 100, o que equivale a 100 usuários disponíveis para requisitar nossa aplicação. 

Para garantir que apenas 100 requisições por segundo sejam feitas, vamos adicionar um `Constant Throughput Timer`.

Clique com o botão direito em `Http Request` > `Add` > `Timer` > `Constant Throughput Timer`

![adicionando configurações de latencia](/assets/images/2025-02-03-img/add-latency.png){: .align-center}

Agora, clique com o botão direito em `Login User Group` > `Add` > `Sampler` > `HTTP Request` e preencha com os valores

- `Target throughput (in samples per minute)` = 6000
- `Calculate Throughput based on` = all active threads

Se queremos 100 requisições por segundo, devemos multiplicar por 60 para converter para amostras por minuto: 100 x 60 = 6000, daí o motivo de `Target throughput` = 6000

![adicionando request](/assets/images/2025-02-03-img/add-request.png){: .align-center}

e vamos configurar nossa _request_ com os valores abaixo:

- `Server Name or IP` = localhost
- `Port Number` = 8080
- `HTTP Request` = POST
- `Path` = /login


Ainda configurando nossa request, em _Body Data_ adicione o JSON abaixo: 

```json
{
    "user": "${__RandomString(10,ABCDEFGHIJKLMNOPQRSTUVWXYZ,random)}",
}
``` 

![adicionando dados na request](/assets/images/2025-02-03-img/add-request-data.png){: .align-center}

Agora vamos adicionar os HTTP Headers. Clique com o botão direito sobre `HTTP Request` > `Add` > `Config Element` > `HTTP Header Manager`

![adicionando header](/assets/images/2025-02-03-img/add-header.png){: .align-center}

Vamos nomear o _header_ como `HTTP Header Manager - /login` , clicar em `Add`  e preencher: 

- `Name` = Content-Type
- `Value` = application/json

![adicionando dados no header](/assets/images/2025-02-03-img/add-header-data.png){: .align-center}

Repare que aqui podemos colocar outros _headers_ como **Authentication**, **Accept-Encoding**, enfim qualquer _header_ que ajude nos nossos testes. 

Estamos quase finalizando a configuração dos nossos testes, agora vamos adicionar os **Listeners**, que vão nos ajudar a visualizar a resposta de cada uma das nossas requisições. 

Novamente com o botão direito em `HTTP Request` > `Add`  > `Listener` > `View Results in Table`, depois salve o arquivo (sempre lembre de salvar seu arquivo pra não perder nenhuma configuração)

![adicionando tabela para visualização de resultado](/assets/images/2025-02-03-img/view-table.png){: .align-center}

### Executando nosso Plano de Testes


Para rodar nosso plano vamos precisar agora da nossa API rodando localmente na porta 8080 e com um _endpoint_ `/login` ativo. Você pode baixar a API [neste link](https://github.com/victor-VN/jmeter-article-test-login) 
e rodar dando um _play_ no IntelliJ ou outra IDE de sua preferência


Agora que temos nossa API rodando localmente

![rodando api](/assets/images/2025-02-03-img/api-running.png){: .align-center}

basta dar um _play_ no nosso plano de testes e verificar a saída clicando em `View Results in Table`

![executando testes](/assets/images/2025-02-03-img/executing.png){: .align-center}

Você também pode verificar os logs da aplicação enquanto o teste está em execução.

### Algumas Considerações

Neste _post_ eu explico como configurar um teste básico usando JMeter, mas existem muitas outras formas de se configurar um teste no JMeter e algumas _features_ muito legais que podem te ajudar no dia a dia como:

- Integração com Cloud (Azure, AWS, Google)
- Integração com pipelines (CI/CD)
- _Assertions_ em _response codes_, isso é, dependendo da resposta da sua API na execução do teste você pode executar uma ação
- Modularidade: você pode configurar testes pequenos e combinar com outros testes (ex: configurar um _token_ de aplicação e utilizar em outros testes)

Em _posts_ futuros pretendo trazer algumas destas configurações, mas por enquanto vamos ficar apenas com o teste básico. 

Uma outra opção interessante é combinar os testes com o [visualVM](https://visualvm.github.io/), uma aplicação que permite acompanhar em tempo real como a JVM da sua aplicação Java está se comportando enquanto as _requests_ são feitas. 

### Conclusão

Fizemos aqui a configuração e executamos um teste simples usando Apache JMeter, espero que tenha te ajudado a conhecer uma nova ferramenta e deixo aqui uma dica: **utilize o JMeter para evidenciar seu trabalho**. Desenvolveu uma nova API? Crie testes com o JMeter, exceda as expectativas de usuários e apresente aos clientes e à sua equipe relatórios mostrando que sua aplicação suporta muitas requisições, mostre que teve o cuidado de se preocupar com a performance da aplicação que você desenvolveu. 
