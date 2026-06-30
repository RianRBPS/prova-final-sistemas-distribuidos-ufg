# Resumo — Capítulo 8: Tolerância a Falhas

**Livro:** *Distributed Systems*, Maarten van Steen e Andrew S. Tanenbaum, 4ª edição.  
**Escopo da prova:** Cap. 8.1, Cap. 8.2 apenas 8.2.1, 8.2.2, 8.2.3 e 8.2.8, e Cap. 8.3.

**Entra na prova:**

- 8.1 — Introduction to fault tolerance
- 8.1.1 — Basic concepts
- 8.1.2 — Failure models
- 8.1.3 — Failure masking by redundancy
- 8.2.1 — Resilience by process groups
- 8.2.2 — Failure masking and replication
- 8.2.3 — Consensus in faulty systems with crash failures
- 8.2.8 — Failure detection
- 8.3 — Reliable client-server communication
- 8.3.1 — Point-to-point communication
- 8.3.2 — RPC semantics in the presence of failures

**Fora do escopo:**

- 8.2.4 — Paxos
- 8.2.5 — Consensus in faulty systems with arbitrary failures
- 8.2.6 — Consensus in blockchain systems
- 8.2.7 — Some limitations on realizing fault tolerance
- 8.4 — Reliable group communication
- 8.5 — Distributed commit
- 8.6 — Recovery

---

# Visão geral do capítulo

Tolerância a falhas trata de manter o sistema funcionando corretamente mesmo quando partes dele falham.

Em sistemas distribuídos, falhas são normais porque há muitos componentes independentes:

- processos;
- máquinas;
- discos;
- redes;
- mensagens;
- serviços;
- links;
- clocks;
- datacenters.

A dificuldade central:

```text
Em sistemas distribuídos, falhas podem ser parciais.
```

Isso significa:

```text
Uma parte do sistema falha, mas o restante continua executando.
```

Exemplo:

```text
Cliente está funcionando.
Servidor está funcionando.
Rede entre eles está falhando.
```

Do ponto de vista do cliente, é difícil saber se:

- o servidor caiu;
- a rede atrasou;
- a resposta se perdeu;
- o servidor está lento;
- a requisição nunca chegou;
- a requisição chegou, foi executada, mas a resposta se perdeu.

Esse é o núcleo do capítulo.

---

# 8.1 — Introdução à tolerância a falhas

Um sistema tolerante a falhas é projetado para continuar prestando serviço mesmo diante de falhas em alguns componentes.

A ideia não é impedir todas as falhas.

A ideia é:

```text
falhas podem ocorrer, mas o sistema deve mascará-las, recuperá-las ou reduzir seus efeitos.
```

---

# 8.1.1 — Conceitos básicos

## Dependabilidade

Dependabilidade é a capacidade de um sistema ser confiável para uso.

Ela envolve várias propriedades.

---

## Availability — disponibilidade

Disponibilidade mede se o sistema está pronto para ser usado em determinado momento.

Pergunta:

```text
O sistema está operacional agora?
```

Exemplo:

```text
Um site disponível 99,9% do tempo.
```

Alta disponibilidade significa pouco tempo fora do ar.

---

## Reliability — confiabilidade

Confiabilidade mede se o sistema continua funcionando corretamente durante um intervalo de tempo.

Pergunta:

```text
O sistema vai continuar funcionando corretamente pelos próximos X minutos/horas/dias?
```

Exemplo:

```text
Um sistema de controle de voo precisa funcionar sem falhas durante todo o voo.
```

Diferença essencial:

```text
Disponibilidade olha para um instante.
Confiabilidade olha para um intervalo.
```

---

## Safety — segurança operacional

Safety significa que, mesmo se o sistema falhar, ele não deve causar dano grave.

Pergunta:

```text
Se falhar, o sistema falha de forma segura?
```

Exemplo:

```text
Sistema ferroviário falha e faz os trens pararem.
```

Isso é melhor do que falhar permitindo colisão.

---

## Maintainability — manutenibilidade

Manutenibilidade mede a facilidade e rapidez com que o sistema pode ser reparado.

Pergunta:

```text
Quando falhar, quão rápido ele pode ser consertado?
```

Exemplo:

```text
Sistema com reinicialização automática, logs claros e substituição rápida de componentes.
```

---

# Fault, error e failure

Esses três termos são fundamentais.

## Fault — falta/defeito

Uma **fault** é a causa de um erro.

Exemplos:

- bug no código;
- disco defeituoso;
- bit flip em memória;
- link de rede quebrado;
- configuração errada;
- fonte de energia falhando;
- operador humano comete erro.

Fault é a causa.

---

## Error — erro

Um **error** é uma parte do estado do sistema que está incorreta.

Exemplo:

```text
Variável deveria ser x = 10, mas está x = 99.
```

O erro é um estado interno incorreto.

---

## Failure — falha

Uma **failure** ocorre quando o sistema deixa de entregar o serviço especificado.

Exemplo:

```text
Cliente pede saldo.
Sistema retorna saldo errado.
```

Ou:

```text
Cliente pede resposta.
Sistema não responde.
```

Failure é o comportamento observável incorreto.

---

## Cadeia causal

A relação é:

```text
fault -> error -> failure
```

Exemplo:

```text
Bug no código -> variável corrompida -> resposta errada ao cliente
```

Nem toda fault vira failure.

Exemplo:

```text
Bug existe, mas nunca é executado.
```

Nem todo error vira failure.

Exemplo:

```text
Estado interno errado é detectado e corrigido antes de afetar o cliente.
```

Tolerância a falhas tenta impedir que faults virem failures visíveis.

---

# Tipos de faults

## Transient fault

Falha temporária que ocorre uma vez e desaparece.

Exemplo:

```text
Interferência momentânea causa perda de uma mensagem.
```

## Intermittent fault

Falha que aparece, desaparece e reaparece.

Exemplo:

```text
Conector de rede com mau contato.
```

É difícil de diagnosticar porque o sistema parece funcionar às vezes.

## Permanent fault

Falha persistente que continua até reparo.

Exemplo:

```text
Disco queimado.
```

---

# Técnicas gerais para dependabilidade

## Fault prevention

Evitar que faults sejam introduzidas.

Exemplos:

- bom projeto;
- testes;
- revisão de código;
- hardware confiável;
- controle de configuração.

## Fault tolerance

Continuar funcionando mesmo com faults.

Exemplos:

- replicação;
- redundância;
- retry;
- votação;
- failover.

## Fault removal

Remover faults existentes.

Exemplos:

- debugging;
- manutenção;
- substituição de componente defeituoso.

## Fault forecasting

Estimar ocorrência e impacto de faults.

Exemplos:

- análise estatística;
- monitoramento;
- previsão de MTBF;
- modelagem de risco.

---

# 8.1.2 — Modelos de falha

Um modelo de falha descreve como um componente pode falhar.

Isso é necessário porque protocolos diferentes assumem falhas diferentes.

Exemplo:

```text
Um protocolo que tolera crash pode não tolerar comportamento bizantino.
```

---

# Failure models principais

## 1. Crash failure

Um processo executa normalmente até parar completamente.

Depois que para, não faz mais nada.

Exemplo:

```text
Servidor desliga.
Processo termina.
Máquina reinicia.
```

Características:

- antes de cair, comportamento é correto;
- depois de cair, não envia mensagens;
- não envia respostas erradas;
- não age maliciosamente.

Modelo relativamente simples.

---

## 2. Omission failure

O processo ou canal deixa de fazer algo que deveria fazer.

Exemplo:

```text
mensagem não é enviada
mensagem não é recebida
requisição é ignorada
resposta se perde
```

Pode ocorrer no envio ou recebimento.

### Send omission

Processo deveria enviar uma mensagem, mas não envia.

### Receive omission

Processo deveria receber uma mensagem, mas não recebe.

### Channel omission

Canal perde uma mensagem.

---

## 3. Timing failure

O resultado está correto, mas chega fora do intervalo de tempo esperado.

Exemplo:

```text
Servidor responde corretamente, mas tarde demais.
```

Importante em sistemas de tempo real.

Exemplo:

```text
Airbag responde certo, mas depois da colisão.
```

Resposta correta atrasada ainda é falha.

---

## 4. Response failure

O processo responde, mas a resposta está errada.

Há dois casos principais.

### Value failure

O valor retornado está incorreto.

Exemplo:

```text
saldo correto = 100
servidor retorna 900
```

### State transition failure

O servidor executa uma transição de estado incorreta.

Exemplo:

```text
Ao receber depósito, deveria somar.
Mas subtrai.
```

---

## 5. Arbitrary failure / Byzantine failure

O processo pode se comportar de qualquer forma.

Exemplos:

- enviar respostas diferentes para clientes diferentes;
- mentir;
- corromper mensagens;
- agir maliciosamente;
- violar protocolo;
- parecer correto por um tempo e depois agir errado.

É o modelo mais geral e mais difícil.

A prova não entra nos protocolos bizantinos, mas o conceito pode aparecer na tabela de modelos.

---

## Hierarquia aproximada

Do mais simples ao mais difícil:

```text
crash < omission < timing < response < arbitrary/Byzantine
```

Quanto mais geral o modelo de falha, mais caro e complexo é tolerá-lo.

---

# Fail-silent e fail-stop

## Fail-silent

Um componente é fail-silent quando, ao falhar, simplesmente para de produzir saída.

Exemplo:

```text
Servidor caiu e não responde mais.
```

Ele não retorna respostas erradas.

## Fail-stop

Fail-stop é um modelo mais forte.

Além de parar, a falha é detectável por outros processos.

Exemplo:

```text
Servidor caiu e todos conseguem detectar claramente essa queda.
```

Em sistemas distribuídos reais, fail-stop é difícil porque timeout não prova falha.

Pode ser apenas atraso de rede.

---

# 8.1.3 — Mascaramento de falhas por redundância

A técnica principal para mascarar falhas é redundância.

Redundância significa adicionar recursos extras para tolerar falhas.

---

# Tipos de redundância

## 1. Redundância de informação

Adiciona bits extras para detectar ou corrigir erros.

Exemplos:

- checksum;
- paridade;
- códigos de correção de erro;
- hash;
- RAID com paridade.

Exemplo:

```text
Mensagem + checksum
```

O receptor recalcula o checksum e verifica se a mensagem foi corrompida.

---

## 2. Redundância de tempo

Executa novamente uma operação.

Exemplos:

- retry de mensagem;
- retransmissão;
- repetir cálculo;
- refazer requisição após timeout.

Exemplo:

```text
Cliente envia requisição.
Não recebe resposta.
Cliente retransmite.
```

Boa para falhas transientes.

Problema:

```text
Repetir operação não idempotente pode causar efeito duplicado.
```

Exemplo:

```text
debitar R$100 duas vezes.
```

---

## 3. Redundância física

Usa componentes extras.

Exemplos:

- múltiplos servidores;
- discos replicados;
- fontes de energia redundantes;
- datacenters diferentes;
- múltiplas rotas de rede;
- processos replicados.

Exemplo:

```text
Três servidores executam o mesmo serviço.
Se um cai, os outros continuam.
```

---

# Mascaramento por votação

Com réplicas, o sistema pode usar votação.

Exemplo:

```text
Replica A retorna 42
Replica B retorna 42
Replica C retorna 99
```

A maioria retorna `42`.

Logo, o sistema escolhe `42`.

Para tolerar `k` falhas por valor incorreto, normalmente precisa de:

```text
2k + 1 réplicas
```

Exemplo:

```text
k = 1
2k + 1 = 3 réplicas
```

Com 3 réplicas, uma pode estar errada e a maioria ainda vence.

---

# 8.2.1 — Resiliência por grupos de processos

A ideia é organizar processos em grupos.

Um grupo de processos pode agir como se fosse um único serviço lógico.

Exemplo:

```text
Cliente envia requisição para o grupo "servidor de arquivos".
Algum membro do grupo responde.
```

O cliente não precisa saber exatamente qual réplica está ativa.

---

# Process group

Um process group é um conjunto de processos que cooperam.

O grupo pode ser usado para:

- replicar serviço;
- tolerar falhas;
- distribuir carga;
- mascarar falha de membros;
- implementar comunicação coletiva.

---

# Grupo estático vs dinâmico

## Grupo estático

A composição do grupo é fixa.

Exemplo:

```text
Servidores S1, S2, S3 sempre formam o grupo.
```

Vantagem:

```text
simples
```

Desvantagem:

```text
não lida bem com entrada/saída dinâmica
```

## Grupo dinâmico

Processos podem entrar e sair.

Exemplo:

```text
Novas réplicas são adicionadas.
Réplicas falhas são removidas.
```

Requer gerenciamento de membros.

---

# Group membership

Group membership é o problema de manter a lista de membros ativos do grupo.

Perguntas:

```text
Quem pertence ao grupo agora?
Quem falhou?
Quem entrou?
Quem saiu?
Todos concordam sobre a composição atual?
```

Isso é difícil porque detectar falhas é difícil.

---

# Grupos planos e hierárquicos

## Grupo plano

Todos os processos são equivalentes.

Não há coordenador fixo.

Vantagens:

- sem ponto único de falha;
- decisões podem ser distribuídas.

Desvantagens:

- coordenação mais complexa;
- mais mensagens;
- consenso mais difícil.

---

## Grupo hierárquico

Há um coordenador.

Exemplo:

```text
Coordenador recebe requisições e distribui aos workers.
```

Vantagens:

- simples;
- decisões centralizadas;
- menos mensagens.

Desvantagens:

- coordenador pode virar gargalo;
- coordenador pode ser ponto único de falha;
- exige eleição ou substituição se coordenador cair.

---

# 8.2.2 — Mascaramento de falhas e replicação

A replicação de processos permite mascarar falhas.

Há duas abordagens principais:

1. primary-based protocols;
2. replicated-write / active replication.

No Cap. 8, o foco é tolerância a falhas, mas a ideia se conecta ao Cap. 7.

---

# Primary-backup para tolerância a falhas

## Ideia

Um processo primário recebe requisições.

Backups mantêm cópias do estado.

Se o primário falha, um backup assume.

Modelo:

```text
cliente -> primário -> backups
```

---

## Funcionamento básico

1. Cliente envia requisição ao primário.
2. Primário executa ou ordena a operação.
3. Primário envia atualização de estado aos backups.
4. Backups confirmam.
5. Primário responde ao cliente.
6. Se primário falha, backup é promovido.

---

## Vantagens

- simples;
- boa semântica para cliente;
- backups permitem recuperação;
- só um processo ordena operações.

---

## Desvantagens

- primário é gargalo;
- falha do primário exige detecção;
- failover pode causar interrupção;
- se primário respondeu ao cliente antes de atualizar backups, pode haver perda de operação.

---

# Replicação ativa

## Ideia

Todas as réplicas executam as mesmas operações.

Modelo:

```text
cliente -> multicast para todas as réplicas
```

Cada réplica processa a requisição.

---

## Requisitos

Para todas terminarem com o mesmo estado, é necessário:

```text
mesmas operações
mesma ordem
execução determinística
```

Se qualquer uma dessas condições falhar, as réplicas podem divergir.

---

## Vantagens

- não há primário único;
- boa tolerância a falha de réplicas;
- pode responder com votação ou primeira resposta correta;
- útil para mascarar crash failures.

---

## Desvantagens

- precisa de multicast ordenado;
- operações não determinísticas são problemáticas;
- mais custo de processamento;
- pode haver múltiplas respostas ao cliente;
- coordenação é mais complexa.

---

# Quantidade de réplicas

## Para falhas por crash

Para tolerar `k` falhas por crash, são necessárias pelo menos:

```text
k + 1 réplicas
```

Exemplo:

```text
Tolerar 1 crash -> 2 réplicas
Tolerar 2 crashes -> 3 réplicas
```

Por quê?

Se `k` réplicas caírem, ainda sobra pelo menos uma funcionando.

---

## Para falhas bizantinas ou valor incorreto

Para mascarar `k` falhas arbitrárias por votação simples, costuma-se precisar de:

```text
2k + 1 réplicas
```

Exemplo:

```text
Tolerar 1 resposta errada -> 3 réplicas
```

Observação:

Protocolos bizantinos completos podem exigir mais réplicas, mas isso está fora do escopo detalhado da prova.

---

# 8.2.3 — Consenso em sistemas com crash failures

Consenso é o problema de fazer processos concordarem em um valor.

Exemplo:

```text
Qual servidor será o novo líder?
Qual operação será confirmada?
Qual valor será escrito?
```

---

# Propriedades básicas do consenso

Um algoritmo de consenso normalmente deve satisfazer três propriedades.

## Agreement

Todos os processos corretos decidem o mesmo valor.

```text
Não pode haver dois processos corretos decidindo valores diferentes.
```

## Validity

Se um processo decide um valor, esse valor deve ter sido proposto por algum processo.

```text
O algoritmo não inventa valor arbitrário.
```

Em algumas versões, validity é mais forte:

```text
Se todos propõem v, então todos decidem v.
```

## Termination

Todo processo correto eventualmente decide algum valor.

```text
O algoritmo não pode esperar para sempre.
```

---

# Crash failures

Neste escopo, o consenso considera falhas por crash.

Ou seja:

```text
processos podem parar
mas não mentem
não enviam mensagens corrompidas
não agem maliciosamente
```

---

# Consenso é necessário para quê?

Exemplos:

- eleger novo líder;
- concordar sobre ordem de operações;
- confirmar atualização replicada;
- decidir membros de um grupo;
- decidir se uma transação deve continuar;
- manter réplicas consistentes.

---

# Problema fundamental

Em sistema distribuído assíncrono, é impossível distinguir perfeitamente:

```text
processo lento
```

de:

```text
processo morto
```

Timeouts são suspeitas, não provas.

Essa dificuldade afeta consenso, eleição e detecção de falhas.

---

# Consenso com crash em modelo síncrono

Em um modelo síncrono, há limites conhecidos para:

- tempo de processamento;
- atraso de mensagens;
- drift de relógios.

Com esses limites, é mais fácil detectar falhas e avançar em rodadas.

Exemplo conceitual:

```text
Rodada 1: processos enviam valores.
Rodada 2: repassam valores conhecidos.
Depois de rodadas suficientes, decidem.
```

Se o sistema tolera até `f` crash failures, algoritmos por rodada podem exigir até:

```text
f + 1 rodadas
```

A intuição:

```text
cada rodada ajuda a revelar se algum processo caiu antes de disseminar informação.
```

---

# Consenso com crash em modelo assíncrono

Em modelo assíncrono, não há limite conhecido para atraso de mensagens.

Logo:

```text
timeout não prova falha
```

Um processo pode estar:

- lento;
- isolado temporariamente;
- com mensagens atrasadas;
- realmente morto.

Essa incerteza torna consenso muito difícil.

O detalhe formal de impossibilidade e Paxos está fora do escopo, mas a intuição é importante:

```text
sem hipóteses extras, consenso confiável em sistema assíncrono com falhas é problemático.
```

---

# 8.2.8 — Detecção de falhas

Detecção de falhas é o mecanismo pelo qual processos suspeitam que outros falharam.

É usada em:

- eleição de líder;
- failover;
- remoção de membros do grupo;
- reconfiguração;
- retransmissão;
- recuperação.

---

# Detector de falhas

Um failure detector é um componente que informa suspeitas de falha.

Exemplo:

```text
suspect(P7)
```

Significa:

```text
O detector suspeita que P7 falhou.
```

Não significa certeza absoluta.

---

# Heartbeats

Mecanismo comum:

```text
processo envia mensagens periódicas "estou vivo"
```

Se outro processo deixa de receber heartbeats por certo tempo, suspeita de falha.

Exemplo:

```text
P1 espera heartbeat de P2 a cada 5 segundos.
Se passam 15 segundos sem heartbeat, P1 suspeita de P2.
```

---

# Timeouts

Timeout é o tempo máximo que um processo espera por resposta antes de suspeitar de falha.

Problema:

```text
timeout curto -> falsas suspeitas
timeout longo -> detecção lenta
```

Trade-off:

| Timeout | Vantagem | Desvantagem |
|---|---|---|
| Curto | detecta rápido | muitas falsas suspeitas |
| Longo | menos falsas suspeitas | recuperação lenta |

---

# Falsa suspeita

Ocorre quando o detector suspeita de um processo que não falhou.

Exemplo:

```text
P2 está vivo, mas a rede atrasou seus heartbeats.
P1 suspeita que P2 caiu.
```

Falsas suspeitas podem causar:

- eleição desnecessária;
- troca indevida de líder;
- reconfiguração errada;
- duplicidade de primário;
- perda temporária de disponibilidade.

---

# Detecção centralizada

Um processo monitora os demais.

Exemplo:

```text
coordenador recebe heartbeats de todos
```

Vantagem:

```text
simples
```

Desvantagem:

```text
monitor central é ponto único de falha e gargalo
```

---

# Detecção distribuída

Processos monitoram uns aos outros.

Exemplo:

```text
cada processo monitora alguns vizinhos
```

Vantagem:

```text
não depende de um único monitor
```

Desvantagem:

```text
mais complexa
pode haver suspeitas inconsistentes
```

---

# Detecção hierárquica

Organiza processos em grupos.

Monitores locais observam membros do grupo.

Monitores superiores observam os monitores locais.

Vantagem:

```text
escalabilidade
```

Desvantagem:

```text
mais estrutura e complexidade
```

---

# Propriedades de detectores de falha

Duas propriedades clássicas:

## Completeness

Processos falhos são eventualmente suspeitos.

Em forma curta:

```text
se P falhou, alguém eventualmente suspeita de P
```

## Accuracy

Processos corretos não são suspeitos incorretamente.

Em forma curta:

```text
se P está correto, ninguém deveria suspeitar de P
```

---

## Impossibilidade prática

Em sistemas assíncronos, não é possível garantir simultaneamente detecção perfeita com:

```text
completeness total
accuracy total
```

Motivo:

```text
não há limite conhecido para atraso
```

Logo, não há como saber se um processo está morto ou apenas lento.

---

# 8.3 — Comunicação cliente-servidor confiável

Mesmo se cliente e servidor usam RPC, falhas podem ocorrer.

Problema:

```text
Como o cliente sabe se uma operação remota foi executada?
```

Essa pergunta é mais difícil do que parece.

---

# 8.3.1 — Comunicação ponto a ponto

## Comunicação confiável sobre canais não confiáveis

Redes podem:

- perder mensagens;
- duplicar mensagens;
- reordenar mensagens;
- corromper mensagens;
- atrasar mensagens.

Mecanismos comuns para melhorar confiabilidade:

- acknowledgments;
- retransmissão;
- números de sequência;
- checksums;
- timeouts;
- controle de duplicatas.

---

# Acknowledgment — ACK

O receptor confirma recebimento.

Exemplo:

```text
Cliente envia mensagem M.
Servidor recebe M.
Servidor envia ACK(M).
```

Se o cliente não recebe ACK, pode retransmitir.

---

# Retransmissão

Se não chega confirmação dentro do timeout, o emissor envia novamente.

Exemplo:

```text
send(M)
espera ACK
timeout
send(M) novamente
```

Problema:

```text
A mensagem original pode ter chegado.
Apenas o ACK se perdeu.
```

Nesse caso, a retransmissão cria duplicata.

---

# Números de sequência

Servem para detectar duplicatas e ordenar mensagens.

Exemplo:

```text
M1 seq=10
M2 seq=11
```

Se o receptor recebe duas vezes:

```text
seq=10
```

ele sabe que a segunda é duplicata.

---

# Idempotência

Uma operação é idempotente se executá-la mais de uma vez tem o mesmo efeito que executá-la uma vez.

Exemplo idempotente:

```text
set x = 10
```

Executar uma ou várias vezes resulta em:

```text
x = 10
```

Exemplo não idempotente:

```text
x = x + 1
```

Executar duas vezes altera o resultado.

Outro exemplo não idempotente:

```text
debitar R$100
```

Executar duas vezes causa dano.

Isso é central para RPC com retransmissão.

---

# 8.3.2 — Semântica de RPC na presença de falhas

RPC tenta parecer chamada local.

Mas falhas tornam isso difícil.

Cenários de falha:

1. cliente não consegue localizar servidor;
2. requisição do cliente para o servidor se perde;
3. servidor cai depois de receber requisição;
4. resposta do servidor se perde;
5. cliente cai depois de enviar requisição.

---

# 1. Cliente não consegue localizar servidor

Problema:

```text
Cliente quer chamar serviço, mas não encontra o servidor.
```

Causas:

- servidor fora do ar;
- servidor mudou de endereço;
- serviço de nomes falhou;
- rede particionada;
- cliente consultou nome errado.

Possíveis respostas:

- lançar exceção;
- retornar erro;
- tentar outro servidor;
- tentar novamente depois;
- usar cache de localização.

---

# 2. Requisição se perde

Cliente envia requisição, mas ela não chega ao servidor.

Solução comum:

```text
timeout + retransmissão
```

Cliente espera resposta.

Se não chega, envia novamente.

Se a requisição original realmente se perdeu, isso resolve.

Se a requisição chegou e a resposta se perdeu, pode gerar duplicação.

---

# 3. Servidor cai depois de receber requisição

Este é um dos casos mais difíceis.

Possibilidades:

```text
Servidor caiu antes de executar.
Servidor executou parcialmente.
Servidor executou completamente, mas caiu antes de responder.
```

O cliente não sabe qual caso ocorreu.

Exemplo:

```text
Cliente pede: transferir R$100.
Servidor cai.
```

Pergunta:

```text
A transferência aconteceu?
```

O cliente não sabe.

---

# 4. Resposta se perde

Servidor executa a operação e envia resposta.

A resposta se perde.

Cliente observa apenas:

```text
não recebi resposta
```

Ele não sabe se:

- requisição se perdeu;
- servidor caiu;
- servidor executou mas resposta se perdeu;
- rede atrasou.

Se o cliente retransmite, o servidor pode executar de novo.

---

# 5. Cliente cai depois de enviar requisição

O servidor pode estar executando uma operação para um cliente que já caiu.

Esse cálculo é chamado de **orphan computation**.

---

# Orphans

Um orphan é uma computação que continua executando no servidor mesmo depois que o cliente morreu ou não precisa mais do resultado.

Problemas causados:

- desperdício de CPU;
- locks podem ficar presos;
- transações podem continuar indevidamente;
- resposta pode ser enviada a cliente reiniciado;
- efeitos colaterais podem ocorrer sem controle.

---

## Estratégias para lidar com orphans

### Extermination

Quando o cliente reinicia, ele usa um log para localizar e matar computações órfãs.

Problema:

```text
custo de manter log
difícil localizar todos os orphans
```

### Reincarnation

O tempo é dividido em épocas.

Quando cliente reinicia, começa nova época.

Computações de épocas antigas são descartadas.

Exemplo:

```text
Cliente C época 7.
Depois reinicia como época 8.
Mensagens da época 7 são ignoradas.
```

### Gentle reincarnation

Quando cliente reinicia, tenta verificar se há computações antigas antes de matá-las.

É menos agressivo que reincarnation puro.

### Expiration

Cada RPC recebe tempo máximo de vida.

Se passar do limite, é descartado.

Problema:

```text
escolher timeout correto é difícil
```

---

# Semânticas de RPC

Na presença de falhas, sistemas RPC tentam oferecer alguma semântica sobre quantas vezes a operação foi executada.

---

# Maybe semantics

## Ideia

A chamada pode ter sido executada zero ou uma vez.

O cliente não sabe.

Exemplo:

```text
Cliente envia RPC.
Não recebe resposta.
Sistema não tenta garantir nada.
```

Resultado:

```text
talvez executou
talvez não executou
```

Vantagem:

```text
simples
```

Desvantagem:

```text
sem garantia útil
```

---

# At-least-once semantics

## Ideia

O sistema retransmite até obter resposta.

A operação será executada uma ou mais vezes.

Exemplo:

```text
Cliente envia.
Timeout.
Cliente retransmite.
Servidor pode executar duas vezes.
```

Boa para operações idempotentes.

Exemplo:

```text
set temperatura_alvo = 22
```

Perigosa para operações não idempotentes.

Exemplo:

```text
debitar R$100
```

Se executar duas vezes:

```text
debita R$200
```

---

# At-most-once semantics

## Ideia

A operação é executada no máximo uma vez.

Para isso, o servidor precisa detectar duplicatas.

Mecanismos:

- identificador único de requisição;
- número de sequência;
- cache de respostas;
- registro de requisições já processadas.

Funcionamento:

1. Cliente envia requisição com ID único.
2. Servidor verifica se já processou esse ID.
3. Se não processou, executa e guarda resposta.
4. Se receber duplicata, não executa de novo.
5. Reenvia a resposta guardada.

---

## Vantagem

Evita execução duplicada.

Boa para operações não idempotentes.

---

## Limitação

At-most-once não garante que a operação será executada.

Se o servidor cai antes de executar, pode ser zero vezes.

Então:

```text
at-most-once = zero ou uma vez, nunca mais de uma
```

Não é exatamente “exatamente uma vez”.

---

# Exactly-once semantics

## Ideia ideal

A operação é executada exatamente uma vez.

Esse é o comportamento que programadores geralmente desejam.

## Problema

Em sistemas distribuídos com falhas, exactly-once é extremamente difícil ou impossível de garantir de forma geral.

Motivo:

```text
não é sempre possível saber se a operação foi executada antes da falha
```

Em prática, sistemas simulam algo próximo usando:

- transações;
- logs persistentes;
- deduplicação;
- idempotência;
- confirmação persistente;
- protocolos de commit.

Mas exactly-once real é uma abstração forte.

---

# Comparação das semânticas de RPC

| Semântica | Execuções possíveis | Como funciona | Risco |
|---|---|---|---|
| Maybe | 0 ou 1 | sem retry forte | cliente não sabe |
| At-least-once | 1 ou mais | retry até resposta | duplicação |
| At-most-once | 0 ou 1 | deduplicação | pode não executar |
| Exactly-once | exatamente 1 | ideal/abstração forte | difícil de garantir |

---

# Quando usar cada uma?

## Maybe

Quando a operação não é crítica.

Exemplo:

```text
log estatístico não essencial
```

## At-least-once

Quando a operação é idempotente.

Exemplo:

```text
consultar status
set valor = X
```

## At-most-once

Quando duplicação é perigosa.

Exemplo:

```text
pagamento
débito
criação de pedido
reserva de passagem
```

## Exactly-once

Desejável quando a operação deve ocorrer uma única vez, mas normalmente exige mecanismos adicionais.

Exemplo:

```text
transação bancária
```

---

# Problema clássico: requisição duplicada

Cenário:

```text
Cliente envia: debitar R$100
Servidor executa.
Resposta se perde.
Cliente retransmite.
Servidor executa de novo.
```

Resultado incorreto:

```text
R$200 debitados.
```

Solução:

```text
usar ID único de requisição
servidor guarda requisições já executadas
se duplicata chegar, reenvia resposta anterior sem executar de novo
```

---

# Problema clássico: resposta duplicada

Cenário:

```text
Cliente retransmite requisição.
Servidor detecta duplicata.
Servidor reenvia mesma resposta.
Cliente pode receber duas respostas.
```

Solução:

```text
cliente também usa número de sequência e ignora respostas antigas ou duplicadas.
```

---

# Problema clássico: servidor crasha

Cenário:

```text
Cliente envia operação.
Servidor executa parcialmente.
Servidor cai.
Servidor reinicia.
Cliente retransmite.
```

Para manter semântica forte, o servidor precisa saber:

```text
operação foi executada?
operação foi confirmada?
resposta foi enviada?
estado foi persistido?
```

Isso exige log persistente ou transação.

---

# Conceitos que conectam Cap. 8 com capítulos anteriores

## Com Cap. 4

RPC e mensagens precisam lidar com falhas.

```text
Timeout + retransmissão + ACK + deduplicação
```

são mecanismos de comunicação confiável.

## Com Cap. 5

Eleição e detectores de falha são usados para substituir coordenadores.

```text
coordenador caiu -> detectar -> eleger novo
```

## Com Cap. 7

Replicação mascara falhas, mas exige consistência.

```text
mais réplicas -> mais disponibilidade
mais consistência -> mais coordenação
```

---

# Exemplos rápidos para prova

## Exemplo 1 — Fault, error, failure

```text
Bug no código de cálculo de saldo.
Variável saldo fica negativa indevidamente.
Cliente recebe saldo errado.
```

Classificação:

```text
fault = bug
error = saldo interno incorreto
failure = resposta errada ao cliente
```

---

## Exemplo 2 — Crash failure

```text
Servidor para e nunca mais responde.
```

Modelo:

```text
crash failure
```

---

## Exemplo 3 — Omission failure

```text
Servidor deveria enviar resposta, mas a mensagem não é enviada.
```

Modelo:

```text
omission failure
```

---

## Exemplo 4 — Timing failure

```text
Servidor retorna resposta correta depois do prazo máximo.
```

Modelo:

```text
timing failure
```

---

## Exemplo 5 — Response failure

```text
Servidor responde, mas com valor errado.
```

Modelo:

```text
response failure
```

---

## Exemplo 6 — Byzantine failure

```text
Servidor envia "sim" para um cliente e "não" para outro na mesma operação.
```

Modelo:

```text
arbitrary/Byzantine failure
```

---

## Exemplo 7 — At-least-once perigoso

```text
Operação: debitar R$100.
Cliente retransmite após timeout.
Servidor executa duas vezes.
```

Problema:

```text
operação não idempotente
```

---

## Exemplo 8 — At-most-once

```text
Cliente envia requisição ID=50.
Servidor executa e guarda resposta.
Cliente retransmite ID=50.
Servidor reenvia resposta sem executar de novo.
```

Semântica:

```text
at-most-once
```

---

# Fórmulas e números importantes

## Réplicas para tolerar crash failures

```text
k + 1 réplicas toleram k falhas por crash
```

Exemplo:

```text
3 réplicas toleram 2 crashes
```

desde que uma permaneça ativa.

---

## Réplicas para mascarar respostas erradas por maioria simples

```text
2k + 1 réplicas toleram k respostas erradas
```

Exemplo:

```text
3 réplicas toleram 1 resposta errada
5 réplicas toleram 2 respostas erradas
```

---

# Perguntas típicas de prova

## 1. Qual a diferença entre fault, error e failure?

Fault é a causa.  
Error é estado interno incorreto.  
Failure é comportamento observável fora da especificação.

```text
fault -> error -> failure
```

---

## 2. Qual a diferença entre disponibilidade e confiabilidade?

Disponibilidade mede se o sistema está operacional em um instante.

Confiabilidade mede se o sistema continua correto durante um intervalo.

---

## 3. O que é falha parcial?

É quando parte do sistema falha, mas o restante continua executando.

Isso é típico de sistemas distribuídos.

---

## 4. O que é crash failure?

Processo executa corretamente até parar completamente.

Depois de parar, não envia mais mensagens.

---

## 5. O que é omission failure?

Omissão ocorre quando uma mensagem, envio, recebimento ou resposta esperada não acontece.

---

## 6. O que é timing failure?

O resultado está correto, mas ocorre fora do prazo esperado.

---

## 7. O que é response failure?

O processo responde, mas com valor errado ou transição de estado incorreta.

---

## 8. O que é Byzantine failure?

Falha arbitrária: o processo pode se comportar de qualquer forma, inclusive enviando respostas diferentes ou maliciosas.

---

## 9. Quais são os tipos de redundância?

Informação, tempo e física.

---

## 10. Como redundância de tempo funciona?

Reexecutando ou retransmitindo operações.

Boa para falhas transientes, mas perigosa para operações não idempotentes.

---

## 11. Como redundância física funciona?

Usando componentes extras, como múltiplos servidores ou discos.

---

## 12. O que é um grupo de processos?

Conjunto de processos que cooperam e podem agir como um único serviço lógico.

---

## 13. Diferença entre grupo plano e hierárquico?

Grupo plano não tem coordenador fixo.

Grupo hierárquico tem coordenador.

Plano evita ponto único de falha, mas é mais complexo.

Hierárquico é simples, mas o coordenador pode falhar.

---

## 14. Como primary-backup mascara falhas?

Backups mantêm cópia do estado. Se o primário falha, um backup assume.

---

## 15. Como active replication mascara falhas?

Todas as réplicas executam as mesmas operações. Se uma falha, outras continuam.

---

## 16. Quais propriedades o consenso deve satisfazer?

Agreement, validity e termination.

---

## 17. Por que consenso é difícil em sistemas assíncronos?

Porque não há limite conhecido para atraso de mensagens, então não é possível distinguir perfeitamente processo lento de processo falho.

---

## 18. O que é um failure detector?

Componente que informa suspeitas de que processos falharam.

---

## 19. Qual o trade-off de timeout em detecção de falhas?

Timeout curto detecta rápido, mas gera falsas suspeitas.

Timeout longo reduz falsas suspeitas, mas demora para detectar falhas reais.

---

## 20. O que é heartbeat?

Mensagem periódica enviada por um processo para indicar que está vivo.

---

## 21. O que é at-least-once RPC?

A operação é executada uma ou mais vezes.

Boa para operações idempotentes.

---

## 22. O que é at-most-once RPC?

A operação é executada no máximo uma vez.

Usa deduplicação com identificadores de requisição.

---

## 23. Por que exactly-once é difícil?

Porque após falhas não é sempre possível saber se a operação foi executada, se a resposta se perdeu ou se o servidor caiu antes de concluir.

---

## 24. O que é orphan computation?

Computação no servidor que continua depois que o cliente que a iniciou caiu ou desistiu.

---

# Respostas curtas para decorar

## Dependabilidade

```text
Capacidade de um sistema prestar serviço confiável.
```

## Availability

```text
Probabilidade de o sistema estar operacional em um instante.
```

## Reliability

```text
Probabilidade de o sistema funcionar corretamente por um intervalo.
```

## Safety

```text
Mesmo falhando, o sistema não causa dano grave.
```

## Maintainability

```text
Facilidade e rapidez de reparar o sistema.
```

## Fault

```text
Causa de um erro.
```

## Error

```text
Estado interno incorreto.
```

## Failure

```text
Desvio observável do serviço especificado.
```

## Crash failure

```text
Processo para e não faz mais nada.
```

## Omission failure

```text
Algo esperado não é enviado, recebido ou executado.
```

## Timing failure

```text
Resposta correta ocorre fora do prazo.
```

## Response failure

```text
Resposta errada ou transição de estado errada.
```

## Byzantine failure

```text
Comportamento arbitrário ou malicioso.
```

## Redundância

```text
Uso de recursos extras para mascarar falhas.
```

## Heartbeat

```text
Mensagem periódica para indicar que processo está vivo.
```

## Failure detector

```text
Mecanismo que suspeita falhas de processos.
```

## Consensus

```text
Problema de fazer processos corretos concordarem em um valor.
```

## Agreement

```text
Processos corretos decidem o mesmo valor.
```

## Validity

```text
Valor decidido foi proposto ou respeita a condição de validade.
```

## Termination

```text
Processos corretos eventualmente decidem.
```

## At-least-once

```text
RPC executa uma ou mais vezes.
```

## At-most-once

```text
RPC executa zero ou uma vez.
```

## Maybe

```text
RPC pode ou não ter executado.
```

## Orphan

```text
Computação remota que continua após a queda/desistência do cliente.
```

---

# Prioridade para estudar

## Prioridade alta

- Fault, error e failure.
- Availability vs reliability vs safety vs maintainability.
- Crash, omission, timing, response e Byzantine failures.
- Redundância de informação, tempo e física.
- Process groups.
- Grupo plano vs hierárquico.
- Primary-backup.
- Active replication.
- Réplicas necessárias para crash failures.
- Agreement, validity e termination.
- Failure detectors.
- Heartbeats e timeouts.
- Falsas suspeitas.
- ACK, retransmissão e números de sequência.
- Idempotência.
- Maybe, at-least-once, at-most-once e exactly-once.
- Orphan computations.

## Prioridade média

- Fail-silent vs fail-stop.
- Fault prevention, tolerance, removal e forecasting.
- Membership de grupos.
- Detecção centralizada, distribuída e hierárquica.
- Estratégias para orphans: extermination, reincarnation, gentle reincarnation e expiration.

## Prioridade baixa ou fora do escopo

- Paxos.
- Consenso bizantino.
- Blockchain consensus.
- Reliable group communication.
- Atomic multicast.
- Distributed commit.
- Checkpointing e recovery.
