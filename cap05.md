# Resumo — Capítulo 5: Coordenação

**Livro:** *Distributed Systems*, Maarten van Steen e Andrew S. Tanenbaum, 4ª edição.  
**Escopo da prova:** Cap. 5.1, 5.2, 5.3 até 5.3.5, e 5.4 exceto 5.4.3, 5.4.4 e 5.4.6.

**Entra na prova:**

- 5.1 — Clock synchronization
- 5.2 — Logical clocks
- 5.3 — Mutual exclusion, até 5.3.5
- 5.4 — Election algorithms
  - 5.4.1 — Bully algorithm
  - 5.4.2 — Ring algorithm
  - 5.4.5 — Elections in large-scale systems

**Fora do escopo:**

- 5.3.6 — ZooKeeper
- 5.4.3 — Leader election in ZooKeeper
- 5.4.4 — Leader election in Raft
- 5.4.6 — Elections in wireless environments
- 5.5 em diante

---

# Visão geral do capítulo

Coordenação trata de como processos em sistemas distribuídos organizam suas ações.

Em um sistema centralizado, há memória compartilhada, relógio único e controle mais simples.

Em um sistema distribuído, não há:

- relógio global confiável;
- memória compartilhada global;
- conhecimento instantâneo do estado dos outros processos;
- garantia de que mensagens chegarão sem atraso;
- forma perfeita de distinguir processo lento de processo falho.

Por isso, problemas simples em sistemas locais ficam difíceis em sistemas distribuídos.

O capítulo trabalha quatro problemas principais:

1. Sincronizar relógios físicos.
2. Ordenar eventos usando relógios lógicos.
3. Garantir exclusão mútua distribuída.
4. Eleger coordenadores/líderes.

---

# 5.1 — Sincronização de relógios

## Problema central

Cada máquina tem seu próprio relógio físico.

Mesmo que dois computadores sejam configurados com o mesmo horário, seus relógios podem se afastar com o tempo.

Esse afastamento acontece porque os cristais de quartzo não oscilam exatamente na mesma frequência.

Esse erro acumulado é chamado de **clock drift**.

A diferença entre os horários mostrados por dois relógios é chamada de **clock skew**.

---

## Exemplo clássico: `make`

O programa `make` decide se deve recompilar um arquivo comparando timestamps.

Exemplo local:

```text
input.c  = 2151
input.o  = 2150
```

Como `input.c` é mais novo que `input.o`, o arquivo precisa ser recompilado.

Em um sistema distribuído, isso pode falhar:

```text
Máquina A altera output.c, mas seu relógio está atrasado.
Máquina B vê output.o como mais novo.
make conclui incorretamente que não precisa recompilar.
```

Moral: timestamps distribuídos podem causar erros se os relógios não estiverem sincronizados.

---

# 5.1.1 — Relógios físicos

Computadores usam temporizadores baseados em cristal de quartzo.

O temporizador gera interrupções periódicas chamadas **clock ticks**.

A cada tick, o sistema operacional atualiza seu relógio de software.

O relógio físico não é perfeito.

Ele pode ser:

- mais lento que o tempo real;
- mais rápido que o tempo real;
- aproximadamente correto, mas ainda sujeito a desvio.

---

## Conceitos importantes

### Clock skew

Diferença entre dois relógios.

```text
Clock A = 10:00:00
Clock B = 10:00:05
Clock skew = 5 segundos
```

### Clock drift

Tendência de um relógio se afastar do tempo correto ao longo do tempo.

### Clock drift rate

Taxa de desvio por unidade de tempo.

Exemplo:

```text
Um relógio pode desviar 1 microssegundo por segundo.
```

Com o tempo, pequenos desvios se acumulam.

---

## UTC, TAI e leap seconds

**TAI** é o tempo atômico internacional, baseado em relógios atômicos.

**UTC** é o tempo civil coordenado usado como referência prática.

Como a rotação da Terra não é perfeitamente regular, o UTC pode receber **leap seconds** para permanecer alinhado com o tempo solar.

Para a prova, o essencial é:

```text
UTC = referência externa de tempo real.
Relógios de computadores precisam ser sincronizados com UTC ou entre si.
```

---

# 5.1.2 — Algoritmos de sincronização de relógios

Há dois objetivos diferentes:

## Sincronização externa

Relógios dos computadores devem ficar próximos de uma referência externa, como UTC.

Formalmente:

```text
|Cp(t) - t| <= α
```

Onde:

- `Cp(t)` é o relógio do processo/máquina `p`;
- `t` é o tempo real;
- `α` é o limite máximo de erro.

## Sincronização interna

Relógios dos computadores devem ficar próximos uns dos outros, mesmo que não estejam corretos em relação ao UTC.

Formalmente:

```text
|Cp(t) - Cq(t)| <= π
```

Onde:

- `Cp(t)` é o relógio da máquina `p`;
- `Cq(t)` é o relógio da máquina `q`;
- `π` é a precisão máxima aceitável entre relógios.

Ponto importante:

```text
Relógios precisos entre si não são necessariamente corretos em relação ao UTC.
```

---

## Precisão vs acurácia

### Precisão

Relógios concordam entre si.

Exemplo:

```text
Todos os servidores estão 5 minutos atrasados, mas todos mostram o mesmo horário.
```

Eles são precisos entre si, mas não acurados.

### Acurácia

Relógios concordam com o tempo real/UTC.

Exemplo:

```text
Servidor mostra horário quase igual ao UTC.
```

---

## Relação entre drift e ressincronização

Se dois relógios podem se afastar em direções opostas, a diferença entre eles pode crescer até:

```text
2ρ · Δt
```

Onde:

- `ρ` é a taxa máxima de drift;
- `Δt` é o tempo desde a última sincronização.

Para manter precisão máxima `π`, é necessário ressincronizar pelo menos a cada:

```text
π / (2ρ)
```

Interpretação:

```text
Quanto maior o drift, mais frequentemente é necessário sincronizar.
Quanto menor a precisão tolerada, mais frequentemente é necessário sincronizar.
```

---

# Network Time Protocol — NTP

NTP sincroniza relógios por troca de mensagens com servidores de tempo.

Modelo simplificado:

```text
A envia requisição para B no tempo T1.
B recebe no tempo T2.
B envia resposta no tempo T3.
A recebe resposta no tempo T4.
```

A usa esses quatro tempos para estimar:

- atraso de rede;
- diferença entre seu relógio e o relógio de B.

## Offset

O offset estima quanto o relógio de A difere do relógio de B.

Fórmula conceitual:

```text
θ = ((T2 - T1) + (T3 - T4)) / 2
```

Se `θ > 0`, A está atrasado em relação a B.  
Se `θ < 0`, A está adiantado em relação a B.

## Delay

O delay estima o atraso de comunicação.

```text
δ = ((T4 - T1) - (T3 - T2)) / 2
```

NTP coleta várias medições e prefere a medição com menor atraso, porque menor atraso tende a significar menor incerteza.

---

## Ajuste de relógio: não voltar o tempo

Um sistema não deve simplesmente mover o relógio para trás.

Isso pode quebrar aplicações baseadas em timestamps.

Exemplo:

```text
Arquivo criado depois pode parecer ter sido criado antes.
```

Solução: ajustar gradualmente.

Se o relógio está adiantado:

```text
Em vez de subtrair tempo, o sistema passa a contar mais devagar temporariamente.
```

Se o relógio está atrasado:

```text
O sistema conta mais rápido temporariamente.
```

---

## Estratos do NTP

NTP organiza servidores em níveis chamados **strata**.

```text
Stratum 0 = relógio de referência, como relógio atômico ou GPS.
Stratum 1 = servidor conectado diretamente ao relógio de referência.
Stratum 2 = servidor sincronizado com stratum 1.
Stratum 3 = servidor sincronizado com stratum 2.
```

Quanto menor o stratum, mais próximo da fonte de tempo.

---

# Reference Broadcast Synchronization — RBS

RBS é um protocolo de sincronização para redes sem fio/sensores.

Diferença principal em relação ao NTP:

```text
NTP sincroniza emissor e receptor.
RBS sincroniza receptores entre si.
```

Funcionamento:

1. Um nó envia uma mensagem de referência por broadcast.
2. Os receptores registram o instante local em que receberam a mensagem.
3. Os receptores trocam entre si esses tempos de recebimento.
4. Comparam os tempos para estimar seus offsets relativos.

Vantagem:

```text
Remove parte da incerteza causada pelo tempo de envio e acesso ao meio.
```

Isso é útil em redes sem fio, onde o tempo para acessar o canal pode variar bastante.

---

# 5.2 — Relógios lógicos

Relógios físicos tentam medir tempo real.

Relógios lógicos tentam ordenar eventos.

Em muitos problemas distribuídos, não importa saber se algo ocorreu às 10:00:01.

Importa saber:

```text
Evento A aconteceu antes do evento B?
Evento A pode ter causado o evento B?
Eventos A e B são concorrentes?
```

---

# 5.2.1 — Relógios lógicos de Lamport

Lamport introduz a relação **happens-before**, escrita como:

```text
a -> b
```

Lê-se:

```text
evento a aconteceu antes do evento b
```

---

## Regras do happens-before

### Regra 1 — Ordem local

Se `a` e `b` são eventos no mesmo processo e `a` ocorre antes de `b`, então:

```text
a -> b
```

### Regra 2 — Envio e recebimento de mensagem

Se `a` é o envio de uma mensagem e `b` é o recebimento dessa mesma mensagem, então:

```text
a -> b
```

Uma mensagem não pode ser recebida antes de ser enviada.

### Regra 3 — Transitividade

Se:

```text
a -> b
b -> c
```

então:

```text
a -> c
```

---

## Eventos concorrentes

Se dois eventos ocorrem em processos diferentes e não há cadeia de mensagens ligando um ao outro, eles são concorrentes.

```text
Não é possível afirmar que a -> b.
Não é possível afirmar que b -> a.
```

Isso não significa que ocorreram exatamente ao mesmo tempo.

Significa apenas que o sistema distribuído não tem informação causal suficiente para ordenar os dois.

---

## Propriedade desejada

Queremos atribuir um valor lógico `C(e)` a cada evento `e`.

A propriedade fundamental é:

```text
Se a -> b, então C(a) < C(b)
```

Atenção:

```text
C(a) < C(b) não implica necessariamente a -> b.
```

Ou seja:

- Lamport preserva causalidade conhecida;
- mas não detecta concorrência com precisão.

---

## Algoritmo de Lamport

Cada processo `Pi` mantém um contador local `Ci`.

### Regra 1

Antes de cada evento local, o processo incrementa seu contador:

```text
Ci <- Ci + 1
```

### Regra 2

Ao enviar uma mensagem `m`, o processo inclui o timestamp:

```text
ts(m) = Ci
```

### Regra 3

Ao receber uma mensagem `m`, o processo ajusta seu contador:

```text
Cj <- max(Cj, ts(m))
Cj <- Cj + 1
```

Depois entrega a mensagem à aplicação.

---

## Empate com identificador do processo

Dois eventos podem receber o mesmo timestamp lógico.

Para obter ordenação total, usa-se o identificador do processo como desempate.

```text
<tempo, id_do_processo>
```

Exemplo:

```text
<40, P1> < <40, P2> se P1 tem id menor que P2
```

---

# Multicast totalmente ordenado com Lamport

Problema:

Em sistemas replicados, todos os servidores precisam aplicar operações na mesma ordem.

Exemplo:

```text
Conta inicial = 1000

Operação A: depósito de 100
Operação B: juros de 1%
```

Se uma réplica executa:

```text
A depois B = 1111
```

e outra executa:

```text
B depois A = 1110
```

as réplicas ficam inconsistentes.

O valor final depende da ordem.

---

## Solução: multicast totalmente ordenado

Objetivo:

```text
Todas as mensagens devem ser entregues a todos os processos na mesma ordem.
```

Com Lamport:

1. Cada mensagem recebe timestamp lógico.
2. Cada processo mantém uma fila local ordenada por timestamp.
3. Ao receber mensagem, o processo coloca na fila.
4. O processo envia acknowledgments.
5. Uma mensagem só é entregue à aplicação quando:
   - está no início da fila;
   - foi reconhecida pelos demais processos.

Resultado:

```text
Todos os processos entregam as mensagens na mesma ordem.
```

Uso típico:

```text
replicação de máquina de estados
state machine replication
```

---

# 5.2.2 — Relógios vetoriais

Relógios de Lamport dizem:

```text
Se a -> b, então C(a) < C(b)
```

Mas não permitem concluir o inverso.

Relógios vetoriais melhoram isso.

Eles permitem detectar causalidade e concorrência com mais precisão.

---

## Ideia principal

Cada processo mantém um vetor com uma posição para cada processo do sistema.

Exemplo com 3 processos:

```text
VC = [tempo_P1, tempo_P2, tempo_P3]
```

Em `Pi`, a posição `VC[i]` representa quantos eventos ocorreram em `Pi`.

As outras posições representam o que `Pi` sabe sobre os eventos dos outros processos.

---

## Algoritmo de relógio vetorial

Cada processo `Pi` mantém um vetor `VCi`.

### Regra 1 — Evento local

Antes de executar um evento:

```text
VCi[i] <- VCi[i] + 1
```

### Regra 2 — Envio de mensagem

Ao enviar mensagem `m`, o processo anexa seu vetor:

```text
ts(m) = VCi
```

### Regra 3 — Recebimento de mensagem

Ao receber mensagem `m` com vetor `ts(m)`, o processo atualiza cada posição:

```text
VCj[k] <- max(VCj[k], ts(m)[k])
```

Depois registra o recebimento como evento local:

```text
VCj[j] <- VCj[j] + 1
```

---

## Comparação de vetores

Dizemos que:

```text
V(a) < V(b)
```

se:

```text
Para todo k: V(a)[k] <= V(b)[k]
E existe pelo menos um k tal que V(a)[k] < V(b)[k]
```

Interpretação:

```text
Se V(a) < V(b), então a pode ter causado b.
```

Se nem `V(a) < V(b)` nem `V(b) < V(a)`, os eventos são concorrentes.

---

## Exemplo conceitual

```text
Evento a: [2, 1, 0]
Evento b: [4, 3, 0]
```

Como todas as posições de `a` são menores ou iguais às de `b`, e pelo menos uma é menor:

```text
a < b
```

Logo, `a` pode preceder causalmente `b`.

Agora:

```text
Evento x: [4, 1, 0]
Evento y: [2, 3, 0]
```

Comparando:

```text
x[1] > y[1]
x[2] < y[2]
```

Nenhum vetor é menor que o outro.

Logo:

```text
x e y são concorrentes.
```

---

## Lamport vs vetor

### Lamport

Vantagens:

- simples;
- barato;
- bom para ordenação total artificial;
- útil em multicast totalmente ordenado.

Limitação:

```text
C(a) < C(b) não prova que a causou b.
```

### Vetorial

Vantagens:

- detecta causalidade;
- detecta concorrência;
- útil para consistência causal.

Limitação:

```text
Vetor cresce com o número de processos.
```

Em sistemas grandes, o custo de carregar vetores pode ser alto.

---

# 5.3 — Exclusão mútua

Exclusão mútua garante que no máximo um processo por vez acesse um recurso crítico.

Exemplo de recurso crítico:

- arquivo compartilhado;
- banco de dados;
- impressora;
- seção crítica de código;
- estrutura replicada que exige atualização exclusiva.

Em sistemas locais, usa-se lock, mutex ou semáforo.

Em sistemas distribuídos, isso é mais difícil porque:

- não há memória compartilhada;
- não há relógio global;
- mensagens atrasam;
- processos podem falhar;
- um processo pode não saber se outro está lento ou morto.

---

# 5.3.1 — Visão geral

Há duas famílias principais de algoritmos.

## 1. Algoritmos baseados em token

Existe um único token no sistema.

```text
Quem possui o token pode acessar o recurso.
Quem não possui o token deve esperar.
```

Vantagens:

- evita deadlock com relativa facilidade;
- pode evitar starvation;
- simples de entender.

Problema principal:

```text
Se o token for perdido, o sistema precisa criar outro.
Mas criar outro token sem criar duplicata é difícil.
```

Se houver dois tokens, a exclusão mútua é violada.

---

## 2. Algoritmos baseados em permissão

Um processo precisa pedir permissão antes de entrar na região crítica.

Essa permissão pode vir de:

- um coordenador central;
- todos os outros processos;
- uma maioria;
- coordenadores de réplicas.

---

# 5.3.2 — Algoritmo centralizado

## Funcionamento

Um processo é escolhido como coordenador.

Quando um processo quer acessar o recurso:

1. Envia `REQUEST` ao coordenador.
2. Se o recurso está livre, o coordenador responde `GRANT`.
3. O processo entra na região crítica.
4. Ao terminar, envia `RELEASE`.
5. O coordenador libera o próximo processo da fila.

---

## Mensagens

Por uso do recurso:

```text
REQUEST
GRANT
RELEASE
```

Total:

```text
3 mensagens
```

---

## Vantagens

- simples;
- fácil de implementar;
- justo se usar fila FIFO;
- sem starvation se o coordenador funcionar corretamente;
- eficiente em número de mensagens.

---

## Desvantagens

- coordenador é ponto único de falha;
- coordenador pode virar gargalo;
- se o coordenador não responde, o cliente pode não saber se:
  - o coordenador morreu;
  - o pedido foi negado;
  - a mensagem atrasou.

Apesar disso, é muito usado na prática porque é simples e previsível.

---

# 5.3.3 — Algoritmo distribuído de Ricart-Agrawala

Este algoritmo usa relógios lógicos de Lamport.

Ideia:

```text
Todos os processos participam da decisão.
O processo com menor timestamp ganha prioridade.
```

---

## Funcionamento

Quando `Pi` quer entrar na região crítica:

1. Incrementa seu relógio lógico.
2. Envia `REQUEST` para todos os outros processos.
3. A mensagem contém:
   - nome do recurso;
   - id do processo;
   - timestamp lógico.
4. `Pi` espera receber `OK` de todos.
5. Quando todos respondem `OK`, `Pi` entra na região crítica.
6. Ao sair, `Pi` envia `OK` para os processos cujos pedidos estavam pendentes.

---

## Como um processo responde a REQUEST

Ao receber `REQUEST` de outro processo, há três casos.

### Caso 1

O receptor não está na região crítica e não quer entrar.

Resposta:

```text
envia OK imediatamente
```

### Caso 2

O receptor já está na região crítica.

Resposta:

```text
não responde agora
coloca o pedido na fila
```

### Caso 3

O receptor também quer entrar na região crítica.

Compara timestamps.

```text
Menor timestamp vence.
```

Se o pedido recebido tem timestamp menor:

```text
envia OK
```

Se o próprio pedido tem timestamp menor:

```text
adia resposta
coloca pedido recebido na fila
```

Se houver empate, usa-se o identificador do processo.

---

## Propriedades

Garante exclusão mútua porque todos concordam na ordem dos pedidos.

Não há deadlock se mensagens são confiáveis.

Não há starvation porque timestamps impõem ordem.

---

## Custo

Com `N` processos:

```text
N - 1 REQUESTs
N - 1 OKs
```

Total:

```text
2(N - 1) mensagens
```

---

## Problemas

Grande ponto fraco:

```text
Se qualquer processo falha, ele pode deixar de responder OK.
```

Então o sistema pode bloquear.

Outros problemas:

- exige conhecer todos os processos do grupo;
- mudanças de grupo são difíceis;
- mensagens precisam ser confiáveis;
- todos participam de todas as decisões;
- ruim para grupos grandes ou dinâmicos.

---

# 5.3.4 — Algoritmo de anel com token

## Ideia

Os processos são organizados em um anel lógico.

Existe um único token circulando.

```text
P0 -> P1 -> P2 -> ... -> PN-1 -> P0
```

Quem recebe o token pode entrar na região crítica.

Se não quiser entrar, apenas repassa o token.

---

## Funcionamento

1. O token começa em algum processo.
2. Cada processo conhece seu sucessor no anel.
3. Ao receber o token:
   - se quer o recurso, entra na região crítica;
   - ao terminar, passa o token adiante;
   - se não quer o recurso, passa o token imediatamente.
4. O processo não pode usar o mesmo token duas vezes seguidas sem repassá-lo.

---

## Vantagens

- exclusão mútua é garantida porque só há um token;
- sem starvation, pois o token circula em ordem;
- simples conceitualmente;
- cada processo tem chance de acessar o recurso.

---

## Desvantagens

- se o token for perdido, precisa ser regenerado;
- detectar token perdido é difícil;
- se um processo cair, o anel precisa ser reconfigurado;
- todos precisam manter informação sobre a configuração do anel;
- se ninguém quer usar o recurso, o token continua circulando inutilmente.

---

# 5.3.5 — Algoritmo descentralizado por votação

## Ideia

O recurso é replicado `N` vezes.

Cada réplica tem seu próprio coordenador.

Para acessar o recurso, o processo precisa obter votos de uma maioria:

```text
m > N/2
```

Isso garante que duas maiorias sempre se sobrepõem.

Logo, dois processos não deveriam conseguir maiorias independentes ao mesmo tempo.

---

## Funcionamento

1. Processo quer acessar recurso.
2. Envia pedido aos `N` coordenadores.
3. Cada coordenador concede ou nega voto.
4. Se o processo obtém maioria `m > N/2`, entra na região crítica.
5. Ao sair, libera os votos.
6. Se não obtém maioria, espera um tempo aleatório e tenta novamente.

---

## Vantagens

- não há único coordenador central;
- tolera algumas falhas;
- distribui a decisão;
- usa ideia de quorum/maioria.

---

## Desvantagens

- mais complexo;
- pode haver contenção alta;
- se muitos processos competem, vários podem obter minorias e ninguém obtém maioria;
- coordenadores podem falhar e esquecer votos concedidos;
- correção passa a ser probabilística em alguns cenários.

---

# Comparação dos algoritmos de exclusão mútua

| Algoritmo | Mensagens por entrada/saída | Atraso antes de entrar | Ponto forte | Ponto fraco |
|---|---:|---:|---|---|
| Centralizado | 3 | 2 tempos de mensagem | Simples e eficiente | Coordenador é ponto único de falha |
| Distribuído Ricart-Agrawala | 2(N - 1) | 2(N - 1) | Sem coordenador central | Qualquer processo pode bloquear o sistema |
| Anel com token | 1 até infinito | 0 até N - 1 | Sem starvation | Token pode ser perdido |
| Descentralizado por votação | Depende de N e tentativas | Depende de N e tentativas | Sem ponto único óbvio | Complexo e pode ter baixa utilização |

Mensagem prática:

```text
Algoritmos distribuídos não são automaticamente melhores que centralizados.
Frequentemente são mais frágeis diante de falhas.
```

---

# 5.4 — Algoritmos de eleição

Muitos sistemas precisam de um processo especial:

- coordenador;
- líder;
- primário;
- sequenciador;
- gerente de recurso.

Se esse processo falha, os demais precisam escolher um novo.

Esse é o problema de eleição.

---

## Hipóteses comuns

O livro assume que:

- cada processo tem identificador único;
- processos conhecem os identificadores dos outros;
- processos não sabem com certeza quem está ativo ou falho;
- o objetivo é que todos concordem no novo coordenador;
- geralmente vence o processo com maior identificador.

---

# 5.4.1 — Bully algorithm

## Ideia

O processo com maior identificador entre os processos ativos deve virar coordenador.

Chama-se bully porque o processo de maior id “manda” nos menores quando volta ou vence.

---

## Funcionamento

Quando `Pk` percebe que o coordenador não responde:

1. `Pk` envia `ELECTION` para todos os processos com id maior.
2. Se ninguém responde, `Pk` vence.
3. `Pk` envia `COORDINATOR` para todos.
4. Se algum processo de id maior responde `OK`, esse processo maior assume a eleição.
5. O processo maior repete o procedimento.
6. No final, o maior processo ativo anuncia que é o coordenador.

---

## Exemplo

Suponha processos:

```text
P0 P1 P2 P3 P4 P5 P6 P7
```

`P7` era coordenador, mas caiu.

`P4` percebe a falha.

`P4` envia `ELECTION` para:

```text
P5 P6 P7
```

`P5` e `P6` respondem `OK`.

`P4` para sua eleição.

`P5` e `P6` iniciam eleições.

`P6` envia `OK` para `P5`.

`P6` percebe que `P7` caiu e vence.

`P6` envia:

```text
COORDINATOR
```

para os demais.

---

## Quando processo maior volta

Se um processo de id alto estava fora e retorna, ele inicia eleição.

Se for o maior ativo, vence e substitui o coordenador atual.

---

## Vantagens

- simples;
- garante que o maior id ativo vence;
- funciona bem em grupos pequenos com conhecimento completo dos membros.

---

## Desvantagens

- muitas mensagens;
- pressupõe que processos conhecem os outros;
- detecção de falha precisa funcionar razoavelmente;
- processo de id alto que volta pode causar troca de líder;
- ruim para sistemas muito grandes ou muito dinâmicos.

---

# 5.4.2 — Algoritmo de eleição em anel

## Ideia

Os processos são organizados em um anel lógico.

Cada processo conhece seu sucessor.

A eleição circula pelo anel coletando identificadores dos processos ativos.

---

## Funcionamento

Quando um processo detecta falha do coordenador:

1. Cria mensagem `ELECTION` com seu próprio id.
2. Envia ao sucessor no anel.
3. Se o sucessor falhou, pula para o próximo.
4. Cada processo ativo adiciona seu id à mensagem.
5. Quando a mensagem retorna ao iniciador, ele escolhe o maior id da lista.
6. A mensagem vira `COORDINATOR`.
7. A mensagem circula novamente informando:
   - quem é o novo coordenador;
   - quais processos ativos estão no novo anel.
8. Ao completar a volta, a mensagem é removida.

---

## Eleições simultâneas

Se dois processos detectam a falha ao mesmo tempo, podem circular duas mensagens `ELECTION`.

Isso não quebra o algoritmo.

Ambas devem chegar à mesma conclusão, desde que vejam o mesmo conjunto de processos ativos.

O custo é apenas mais mensagens.

---

## Vantagens

- estrutura simples;
- não exige enviar mensagem para todos diretamente;
- funciona bem se o anel lógico é mantido corretamente.

---

## Desvantagens

- precisa manter o anel;
- falhas exigem pular processos inativos;
- pode ser lento, pois a mensagem precisa circular;
- ruim se a topologia muda muito;
- também depende de detecção de falhas.

---

# Bully vs anel

| Critério | Bully | Anel |
|---|---|---|
| Quem vence | Maior id ativo | Maior id ativo |
| Estrutura | Conhecimento dos processos maiores | Sucessor no anel |
| Mensagens | Pode ser alto | Circula pelo anel |
| Retorno de processo maior | Pode tomar liderança | Depende da nova eleição |
| Melhor caso | Grupos pequenos | Grupos com anel estável |
| Ponto fraco | Muitas mensagens e instabilidade com retorno de nós altos | Manutenção do anel |

---

# 5.4.5 — Eleições em sistemas de larga escala

Algoritmos como bully e anel assumem grupos relativamente pequenos.

Em sistemas grandes, especialmente blockchains permissionless, pode haver muitos candidatos.

Nesses casos, a eleição de líder pode ser feita por mecanismos econômicos/probabilísticos.

---

# Proof of Work

## Ideia

Candidatos competem para resolver um puzzle computacional.

Quem resolve primeiro vira líder.

Em blockchain, o líder normalmente ganha o direito de adicionar o próximo bloco.

---

## Funcionamento conceitual

1. O validador monta um bloco de transações.
2. Calcula um hash do bloco.
3. Procura um valor extra chamado **nonce**.
4. O objetivo é que o hash do bloco com o nonce tenha certo número de zeros iniciais.

Exemplo simplificado:

```text
hash(bloco + nonce) = 000000...xyz
```

Quanto mais zeros exigidos, mais difícil é encontrar o nonce.

---

## Por que é difícil

Uma função hash criptográfica tem propriedades importantes:

- é fácil calcular o hash de um dado;
- é extremamente difícil encontrar entrada que produza um hash desejado;
- pequena mudança na entrada altera radicalmente o hash.

Assim, encontrar o nonce exige tentativa e erro.

---

## Dificuldade

Se o sistema exige mais zeros iniciais, a chance de um nonce aleatório funcionar diminui.

```text
1 zero  -> chance aproximada de 1/2
2 zeros -> chance aproximada de 1/4
3 zeros -> chance aproximada de 1/8
```

Com muitos zeros, o custo computacional fica enorme.

---

## Trade-off

### Corridas longas

Vantagem:

```text
Menor chance de dois líderes vencerem quase ao mesmo tempo.
```

Desvantagem:

```text
Baixa taxa de transações.
```

### Corridas curtas

Vantagem:

```text
Maior taxa de transações.
```

Desvantagem:

```text
Maior chance de líderes concorrentes e versões divergentes da blockchain.
```

---

## Crítica principal

Proof of Work desperdiça muita energia e poder computacional.

O trabalho serve para tornar a liderança cara, não para resolver um problema útil da aplicação.

---

# Proof of Stake

## Ideia

Em vez de gastar computação, a chance de ser eleito depende da quantidade de tokens possuídos ou colocados em stake.

Quanto mais tokens um participante possui, maior a probabilidade de ser escolhido como líder.

---

## Funcionamento simplificado

1. Existem `N` tokens no sistema.
2. Um procedimento público gera um número aleatório `k` entre 1 e `N`.
3. O dono do token `k` é escolhido como líder.

Exemplo:

```text
Participante A possui 40% dos tokens.
Logo, tem aproximadamente 40% de chance de ser escolhido.
```

---

## Vantagens

- gasta menos energia;
- evita corrida computacional inútil;
- escala melhor que Proof of Work em alguns cenários.

---

## Problemas

No Proof of Work, há custo antes de virar líder.

No Proof of Stake, esse custo antecipado é menor.

Isso pode aumentar a vulnerabilidade a ataques ou comportamentos desonestos se o protocolo não impuser penalidades adequadas.

---

# Seleção de superpeers

Em algumas redes peer-to-peer, não se quer eleger apenas um líder.

É necessário selecionar vários **superpeers**.

Superpeers são nós com papel especial, por exemplo:

- indexar conteúdo;
- rotear mensagens;
- coordenar grupos locais;
- reduzir carga dos nós comuns.

---

## Requisitos para superpeers

Uma boa seleção deve garantir:

1. nós comuns acessam superpeers com baixa latência;
2. superpeers ficam bem distribuídos na rede;
3. há proporção adequada entre superpeers e nós comuns;
4. nenhum superpeer fica sobrecarregado.

---

## Em DHTs

Em sistemas baseados em DHT, pode-se reservar parte do espaço de identificadores para superpeers.

Exemplo:

```text
Usar os primeiros k bits do identificador para definir regiões de superpeers.
```

A ideia é distribuir superpeers de forma relativamente uniforme no overlay.

---

## Em espaço geométrico

Outra ideia é espalhar tokens em um espaço geométrico.

Cada token representa um futuro superpeer.

Os tokens exercem “forças de repulsão” entre si.

Com o tempo, eles se espalham pela rede.

Quando um token fica tempo suficiente em um nó, esse nó se promove a superpeer.

---

# Fórmulas importantes

## Precisão entre relógios

```text
|Cp(t) - Cq(t)| <= π
```

## Acurácia em relação ao tempo real

```text
|Cp(t) - t| <= α
```

## Intervalo máximo de ressincronização

```text
π / (2ρ)
```

## Offset no NTP

```text
θ = ((T2 - T1) + (T3 - T4)) / 2
```

## Delay no NTP

```text
δ = ((T4 - T1) - (T3 - T2)) / 2
```

## Lamport

```text
Se a -> b, então C(a) < C(b)
```

## Vetores

```text
V(a) < V(b)
```

se:

```text
Para todo k: V(a)[k] <= V(b)[k]
E existe k tal que V(a)[k] < V(b)[k]
```

## Maioria em votação

```text
m > N/2
```

---

# Perguntas típicas de prova

## 1. Por que sincronizar relógios é difícil em sistemas distribuídos?

Porque cada máquina tem seu próprio relógio físico, e esses relógios sofrem drift. Como não há relógio global compartilhado, eventos podem receber timestamps inconsistentes.

---

## 2. Qual a diferença entre precisão e acurácia?

Precisão mede o quanto os relógios concordam entre si.

Acurácia mede o quanto um relógio concorda com uma referência externa, como UTC.

---

## 3. Por que um relógio não deve ser ajustado para trás?

Porque isso pode quebrar aplicações que assumem timestamps crescentes. Arquivos, logs e eventos poderiam parecer ocorrer em ordem inversa.

---

## 4. Qual a diferença entre relógio físico e relógio lógico?

Relógio físico tenta medir tempo real.

Relógio lógico tenta ordenar eventos.

---

## 5. O que significa `a -> b`?

Significa que o evento `a` aconteceu antes do evento `b` segundo a relação causal happens-before.

---

## 6. Relógios de Lamport detectam concorrência?

Não com precisão.

Eles garantem:

```text
Se a -> b, então C(a) < C(b)
```

Mas o inverso não é garantido.

---

## 7. Para que servem relógios vetoriais?

Para capturar causalidade com mais precisão e detectar eventos concorrentes.

---

## 8. Qual a diferença entre token-based e permission-based mutual exclusion?

Token-based: quem tem o token entra na região crítica.

Permission-based: processo precisa receber permissão de outros processos.

---

## 9. Por que o algoritmo centralizado é usado apesar do ponto único de falha?

Porque é simples, eficiente, fácil de entender e pode ser tornado mais tolerante a falhas com replicação ou técnicas adicionais.

---

## 10. Qual o custo do algoritmo distribuído de Ricart-Agrawala?

```text
2(N - 1) mensagens
```

para cada entrada na região crítica.

---

## 11. Qual o principal problema do anel com token?

Perda do token.

Detectar e recriar o token sem criar duplicatas é difícil.

---

## 12. Por que votação usa maioria?

Porque duas maiorias sempre se intersectam.

Isso impede que dois processos obtenham permissões totalmente independentes ao mesmo tempo.

---

## 13. O que o bully algorithm garante?

Garante que o processo ativo com maior identificador se torna coordenador.

---

## 14. Como funciona a eleição em anel?

Uma mensagem circula pelo anel coletando ids dos processos ativos. Ao retornar ao iniciador, escolhe-se o maior id e uma mensagem de coordenador informa todos.

---

## 15. Qual o trade-off do Proof of Work?

Mais dificuldade reduz colisões de líderes, mas reduz throughput e aumenta gasto computacional.

Menos dificuldade aumenta throughput, mas aumenta risco de forks/líderes simultâneos.

---

# Prioridade para estudar

## Prioridade alta

- Diferença entre clock skew e clock drift.
- Precisão vs acurácia.
- NTP: T1, T2, T3, T4, offset e delay.
- Lamport: happens-before, algoritmo e limitação.
- Relógios vetoriais: atualização, comparação e concorrência.
- Exclusão mútua centralizada.
- Ricart-Agrawala.
- Token ring.
- Bully algorithm.
- Ring election algorithm.

## Prioridade média

- RBS.
- Descentralized mutual exclusion por votação.
- Comparação de custos dos algoritmos de exclusão mútua.
- Proof of Work e Proof of Stake.

## Prioridade baixa

- UTC/TAI/leap seconds em detalhe.
- Fórmulas probabilísticas da votação descentralizada.
- Seleção de superpeers.
