# Resumo — Capítulo 4: Comunicação

**Escopo da prova:** Cap. 4.2 e 4.3.  
**Fora do escopo:** MPI e multicast do Cap. 4.4.

---

## 4.2 — RPC: Remote Procedure Call

RPC é uma tentativa de esconder a comunicação remota atrás de uma chamada de procedimento normal.

```text
resultado = procedimento_remoto(argumentos)
```

Parece uma chamada local, mas internamente ocorre troca de mensagens entre cliente e servidor.

### Funcionamento básico

1. O cliente chama um **client stub**.
2. O client stub empacota os parâmetros em uma mensagem.
3. A mensagem é enviada pela rede.
4. O **server stub** recebe a mensagem.
5. O server stub desempacota os parâmetros.
6. O servidor executa o procedimento real.
7. O resultado volta pelo caminho inverso.

O objetivo é obter **transparência de acesso**: o programador chama uma função, e os stubs escondem a comunicação.

---

## Problemas fundamentais do RPC

Uma chamada remota nunca é exatamente igual a uma chamada local.

### Diferenças importantes

**Espaços de endereçamento diferentes**

Um ponteiro da máquina A não aponta para nada útil na máquina B.

**Representação de dados**

Inteiros, floats, strings e estruturas podem ter formatos diferentes em máquinas distintas.

**Falhas**

Podem ocorrer falhas no cliente, no servidor, na rede, na requisição ou na resposta.

**Latência**

Uma chamada local usa memória/processador.  
Uma chamada remota envolve rede, serialização, espera e resposta.

---

## Marshaling e unmarshaling

**Marshaling** é o processo de transformar parâmetros em uma representação transmissível pela rede.

**Unmarshaling** é o processo inverso: reconstruir os dados no destino.

Ponto essencial:

- Passagem por valor funciona bem em RPC.
- Passagem por referência é problemática, porque referências locais não fazem sentido em outra máquina.

---

## RPC-based application support

Normalmente usa-se uma **IDL**, ou **Interface Definition Language**.

A IDL descreve:

- quais procedimentos remotos existem;
- quais parâmetros recebem;
- quais resultados retornam.

A partir da IDL, ferramentas podem gerar automaticamente:

- client stub;
- server stub;
- código de empacotamento/desempacotamento;
- interfaces para cliente e servidor.

Isso reduz erro manual e padroniza a comunicação.

---

## Variações de RPC

### RPC síncrono

O cliente chama o servidor e fica bloqueado até receber resposta.

Uso típico: quando o resultado é necessário imediatamente.

### RPC assíncrono

O cliente envia a requisição e não espera o processamento completo.

Uso típico: quando não há resultado importante ou quando o cliente pode continuar trabalhando.

### Deferred synchronous RPC

O cliente envia a requisição, continua executando e busca ou recebe o resultado depois.

Uso típico: quando existe resposta, mas não é necessário bloquear imediatamente.

### Multicast RPC

O cliente envia uma chamada para vários servidores.

Uso típico:

- paralelismo;
- tolerância a falhas;
- replicação;
- receber a primeira resposta válida;
- esperar maioria concordar.

---

# 4.3 — Comunicação orientada a mensagens

Aqui o modelo muda.

RPC tenta esconder mensagens.  
Comunicação orientada a mensagens assume explicitamente que processos trocam mensagens.

Modelo básico:

```text
send(destino, mensagem)
receive(origem, mensagem)
```

A seção existe porque RPC nem sempre serve. Às vezes:

- o receptor não está executando no momento da requisição;
- o cliente não quer bloquear;
- a aplicação precisa de comunicação assíncrona;
- o sistema precisa desacoplar produtor e consumidor.

---

## Comunicação transiente vs persistente

### Comunicação transiente

A mensagem existe enquanto os processos ou o sistema de comunicação estão ativos.

Se o receptor não estiver disponível, a mensagem pode falhar ou ser perdida.

### Comunicação persistente

A mensagem é armazenada até poder ser entregue ou processada.

O emissor e o receptor não precisam estar ativos ao mesmo tempo.

Esta é uma distinção central do Cap. 4.3.

---

## 4.3.1 — Sockets

Um socket é uma extremidade de comunicação.

Um processo cria um socket para enviar ou receber dados pela rede.

### TCP socket

- orientado à conexão;
- fluxo de bytes;
- ordem preservada;
- confiável no nível de transporte.

### UDP socket

- sem conexão;
- trabalha com datagramas;
- mais simples e rápido;
- não oferece garantias fortes de entrega, ordem ou ausência de duplicação.

Sockets são baixo nível. Eles não sabem o que é “requisição”, “resposta”, “pedido”, “evento” ou “transação”.

A aplicação precisa definir o protocolo.

Exemplo:

```text
cliente envia:  "GET item42"
servidor envia: "valor = 10"
```

O socket transporta bytes.  
Quem entende a semântica é a aplicação.

---

## 4.3.2 — Mensagens transientes avançadas

Aqui entra middleware de mensagens, especialmente padrões como os do ZeroMQ.

ZeroMQ ainda é comunicação transiente, mas fornece abstrações mais úteis que sockets puros.

### Request-reply

Cliente envia requisição.  
Servidor responde.

Parece RPC, mas ainda é baseado em mensagens explícitas.

### Publish-subscribe

Produtores publicam mensagens em tópicos.  
Consumidores assinam tópicos.

O produtor não precisa conhecer os consumidores.

Exemplo:

```text
sensores publicam temperatura
serviços interessados assinam o tópico "temperatura"
```

### Push-pull / pipeline

Produtor empurra tarefas.  
Workers recebem e processam.

Uso típico: distribuição de trabalho.

Exemplo:

```text
produtor envia imagens
workers redimensionam as imagens em paralelo
```

### Observação de prova

MPI aparece nessa região do capítulo, mas está fora do escopo informado pelo professor.

---

# 4.3.3 — Comunicação persistente por filas

Esse é o núcleo mais importante do Cap. 4.3.

Modelo:

```text
produtor -> fila -> consumidor
```

A aplicação não entrega diretamente ao consumidor.  
Ela entrega a mensagem a uma fila.

A fila é mantida por um **queue manager**.

O queue manager pode:

- armazenar mensagens;
- encaminhar mensagens;
- entregar mensagens;
- rotear mensagens;
- controlar filas.

---

## Vantagem central: desacoplamento temporal

O emissor pode enviar mesmo se o receptor estiver desligado.

O receptor pode processar depois.

Isso é útil em sistemas distribuídos porque falhas e indisponibilidade parcial são comuns.

Exemplo:

Em vez de um sistema de pedidos chamar diretamente:

```text
emailService.sendConfirmation()
```

ele coloca uma mensagem na fila:

```text
OrderCreated
```

Depois, o serviço de email consome essa mensagem e envia a confirmação.

Se o serviço de email estiver fora do ar, a mensagem continua na fila.

---

## Operações típicas de fila

### PUT

Insere uma mensagem em uma fila.

### GET

Remove ou lê uma mensagem da fila.  
Pode bloquear até existir mensagem.

### POLL

Verifica se há mensagem sem bloquear.

### NOTIFY

Registra um tratador para ser chamado quando chegar mensagem.

---

## Pontos importantes sobre filas

Podem cair em prova:

- diferença entre fila local e remota;
- roteamento entre queue managers;
- nome lógico da fila versus endereço físico;
- armazenamento persistente;
- entrega assíncrona;
- possibilidade de duplicação;
- confirmação de processamento;
- diferença entre enviar mensagem e garantir que ela foi processada.

---

# Message brokers

Um message broker é um intermediário mais sofisticado que apenas uma fila.

Ele pode:

- traduzir formatos de mensagem;
- rotear mensagens;
- aplicar regras;
- integrar aplicações legadas;
- converter protocolos;
- filtrar mensagens por conteúdo ou tópico.

Exemplo:

Sistema A envia:

```text
cliente_id
```

Sistema B espera:

```text
customerId
```

O broker pode transformar a mensagem para compatibilizar os sistemas.

Mensagem importante:

O broker reduz acoplamento entre aplicações, mas adiciona complexidade e pode virar ponto crítico da arquitetura.

---

# 4.3.4 — AMQP

AMQP é um protocolo padronizado para sistemas de filas e mensagens.

Ele surgiu para evitar que cada fornecedor tivesse um sistema proprietário incompatível.

## Conceitos principais

### Application

Programa que produz ou consome mensagens.

### Queue manager

Gerencia filas e comunicação.

### Queue

Armazena mensagens.

### Producer

Nó que envia mensagens.

### Consumer

Nó que recebe mensagens.

### Connection

Conexão estável entre aplicação e queue manager.

### Channel

Canal lógico unidirecional dentro de uma conexão.

### Session

Agrupamento lógico de canais para comunicação bidirecional.

### Link

Caminho lógico por onde uma mensagem é transferida.

---

## Controle de fluxo

AMQP pode usar controle de fluxo por créditos.

O receptor informa quantas mensagens o emissor pode enviar.

Isso evita sobrecarregar o receptor.

---

## Confiabilidade em AMQP

Uma mensagem pode ficar em estado **unsettled** enquanto não há confirmação final.

Quando a transferência é concluída, ela vira **settled**.

Interpretação:

```text
unsettled = ainda não posso esquecer essa mensagem
settled   = transferência concluída
```

Isso permite confiabilidade fim-a-fim: o sistema mantém estado da mensagem até saber que ela foi tratada.

---

# RabbitMQ e exchange

No exemplo prático de RabbitMQ, o produtor normalmente não envia direto para uma fila.

Ele publica em um **exchange**.

O exchange roteia a mensagem para uma ou mais filas com base em regras, como **binding keys**.

Diferença essencial:

```text
exchange = roteia
queue    = armazena
```

Modelo:

```text
producer -> exchange -> queue -> consumer
```

---

# Comparações essenciais para prova

## RPC vs mensagens

RPC esconde a comunicação.  
Mensagens tornam a comunicação explícita.

RPC tende a ser síncrono.  
Mensagens podem ser assíncronas.

RPC acopla cliente e servidor no tempo.  
Filas desacoplam.

RPC parece chamada de função.  
Mensagens parecem envio de dados ou eventos.

---

## Socket vs middleware de mensagens

Socket é baixo nível.  
Middleware fornece padrões como request-reply, publish-subscribe e push-pull.

Socket transporta bytes.  
Middleware dá semântica de comunicação.

---

## Comunicação transiente vs persistente

Transiente exige disponibilidade no momento da comunicação.

Persistente armazena a mensagem até entrega ou processamento.

---

## Fila vs broker

Fila armazena mensagens.

Broker roteia, transforma e integra mensagens.

---

## Exchange vs queue

Exchange decide para onde a mensagem vai.

Queue guarda a mensagem até o consumidor processar.

---

# Prioridade para prova

## Alta prioridade

- RPC básico;
- stubs;
- marshaling e unmarshaling;
- problemas de passagem de parâmetros em RPC;
- RPC síncrono, assíncrono e deferred synchronous;
- diferença entre RPC e mensagens;
- sockets TCP/UDP em nível conceitual;
- comunicação transiente versus persistente;
- filas;
- queue managers;
- desacoplamento temporal;
- AMQP: producer, consumer, queue, queue manager, connection, channel, link, settled/unsettled.

## Média prioridade

- ZeroMQ;
- request-reply;
- publish-subscribe;
- push-pull;
- message brokers;
- transformação de mensagens;
- RabbitMQ;
- exchange;
- queue;
- binding key.

## Baixa prioridade ou fora do escopo

- MPI;
- multicast do Cap. 4.4;
- detalhes de código Python.
