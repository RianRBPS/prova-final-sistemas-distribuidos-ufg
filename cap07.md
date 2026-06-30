# Resumo — Capítulo 7: Consistência e Replicação

**Livro:** *Distributed Systems*, Maarten van Steen e Andrew S. Tanenbaum, 4ª edição.  
**Escopo da prova:** Cap. 7.1, 7.2, 7.3 até 7.3.4, 7.4 e 7.5 até 7.5.2.

**Entra na prova:**

- 7.1 — Introduction
- 7.1.1 — Reasons for replication
- 7.1.2 — Replication as scaling technique
- 7.2 — Data-centric consistency models
- 7.2.1 — Consistent ordering of operations
- 7.2.2 — Eventual consistency
- 7.2.3 — Continuous consistency
- 7.3 — Client-centric consistency models
- 7.3.1 — Monotonic reads
- 7.3.2 — Monotonic writes
- 7.3.3 — Read your writes
- 7.3.4 — Writes follow reads
- 7.4 — Replica management
- 7.4.1 — Finding the best server location
- 7.4.2 — Content replication and placement
- 7.4.3 — Content distribution
- 7.4.4 — Managing replicated objects
- 7.5 — Consistency protocols
- 7.5.1 — Sequential consistency: Primary-based protocols
- 7.5.2 — Sequential consistency: Replicated-write protocols

**Fora do escopo:**

- 7.3.5 — ZooKeeper
- 7.5.3 — Cache-coherence protocols
- 7.5.4 — Implementing continuous consistency
- 7.5.5 — Implementing client-centric consistency
- 7.6 — Caching and replication in the Web
- 7.7 — Summary

---

# Visão geral do capítulo

Replicação significa manter múltiplas cópias de dados ou serviços.

Ela é usada principalmente para:

1. aumentar confiabilidade;
2. melhorar desempenho;
3. escalar o sistema;
4. aproximar dados dos clientes;
5. tolerar falhas.

O problema central:

```text
Quanto mais réplicas, mais difícil manter todas consistentes.
```

A pergunta do capítulo é:

```text
Quando uma réplica é atualizada, o que os clientes podem esperar das outras réplicas?
```

Essa expectativa é formalizada por um **modelo de consistência**.

---

# Termos básicos

## Réplica

Uma cópia de um dado, objeto ou serviço.

Exemplo:

```text
Arquivo X copiado em três servidores.
```

Cada cópia é uma réplica.

## Replicação

Técnica de manter múltiplas réplicas.

## Consistência

Propriedade que define quais valores podem ser observados quando dados replicados são lidos e escritos.

## Modelo de consistência

Contrato entre o armazenamento distribuído e os processos que usam esse armazenamento.

Ele diz:

```text
Quais resultados de leitura são válidos depois de operações de escrita?
```

## Protocolo de consistência

Implementação concreta de um modelo de consistência.

Exemplo:

```text
Modelo: consistência sequencial.
Protocolo: primary-backup ou quorum.
```

---

# 7.1 — Introdução

A replicação parece uma solução óbvia:

```text
Se uma cópia é boa, várias cópias parecem melhores.
```

Mas várias cópias criam um problema:

```text
Se uma cópia muda, as outras ficam antigas.
```

Exemplo:

```text
Replica A: saldo = 100
Replica B: saldo = 100

Cliente deposita 50 na réplica A.

Replica A: saldo = 150
Replica B: saldo = 100
```

Agora o sistema está inconsistente.

A pergunta passa a ser:

```text
B deve ser atualizada imediatamente?
Pode ser atualizada depois?
Clientes podem ler B enquanto ela está antiga?
```

A resposta depende do modelo de consistência.

---

# 7.1.1 — Razões para replicação

## 1. Confiabilidade

Se uma réplica falha, outra pode continuar atendendo.

Exemplo:

```text
Servidor A caiu.
Cliente passa a usar servidor B.
```

Também ajuda contra corrupção de dados.

Exemplo com três cópias:

```text
Replica A retorna 10
Replica B retorna 10
Replica C retorna 99
```

O sistema pode assumir que `10` é o valor correto por maioria.

---

## 2. Desempenho

Replicação melhora desempenho quando muitos processos acessam o mesmo dado.

Sem replicação:

```text
Todos os clientes acessam um único servidor.
```

Problema:

```text
gargalo no servidor
alta latência
sobrecarga de rede
```

Com replicação:

```text
Clientes são distribuídos entre várias réplicas.
```

Resultado:

```text
menor carga por servidor
menor latência
maior vazão
```

---

## 3. Escalabilidade geográfica

Clientes distantes sofrem com latência.

Solução:

```text
colocar réplicas próximas dos clientes
```

Exemplo:

```text
Cliente no Brasil acessa réplica na América do Sul.
Cliente na Europa acessa réplica na Europa.
```

Isso reduz tempo de acesso.

Mas gera custo:

```text
As réplicas precisam ser atualizadas entre si.
```

---

## Preço da replicação

A replicação aumenta confiabilidade e desempenho, mas cria custo de consistência.

Exemplo de cache Web:

```text
Usuário acessa página P.
Navegador guarda cópia local.
Servidor atualiza P.
Navegador pode continuar mostrando versão antiga.
```

Pergunta:

```text
Isso é aceitável?
```

Depende da aplicação.

Para notícia ou catálogo, talvez sim por alguns segundos ou minutos.

Para saldo bancário, provavelmente não.

---

# 7.1.2 — Replicação como técnica de escalabilidade

Replicação escala bem principalmente quando há muitas leituras e poucas escritas.

Exemplo favorável:

```text
10.000 leituras por segundo
10 escritas por segundo
```

Nesse caso, réplicas ajudam muito.

Exemplo desfavorável:

```text
10.000 leituras por segundo
9.000 escritas por segundo
```

Nesse caso, cada escrita precisa ser propagada para muitas réplicas. O custo pode destruir o ganho da replicação.

---

## Read-write ratio

Relação entre leituras e escritas.

```text
read-write ratio alto  = muitas leituras, poucas escritas
read-write ratio baixo = muitas escritas em relação às leituras
```

Replicação é mais vantajosa quando o read-write ratio é alto.

---

## Escalabilidade vs consistência

Quanto mais forte a consistência exigida, mais difícil escalar.

Exemplo:

```text
Toda escrita precisa chegar a todas as réplicas antes de responder ao cliente.
```

Isso dá consistência forte, mas aumenta latência.

Alternativa:

```text
Responder ao cliente imediatamente e propagar depois.
```

Isso melhora desempenho, mas permite inconsistência temporária.

Essa é a tensão central do capítulo:

```text
consistência forte  <->  desempenho/escalabilidade
```

---

# 7.2 — Modelos de consistência centrados em dados

Modelos centrados em dados descrevem o comportamento esperado do sistema de armazenamento compartilhado quando vários processos leem e escrevem dados.

Foco:

```text
O que todos os processos podem observar sobre o conjunto de dados replicados?
```

Eles não olham apenas para um cliente individual.

Eles tentam definir regras globais para leituras e escritas concorrentes.

---

# 7.2.1 — Ordenação consistente de operações

A principal questão é ordenar operações de leitura e escrita.

Operações típicas:

```text
W(x)a = write em x com valor a
R(x)a = read de x retornando a
```

Exemplo:

```text
W1(x)a
```

Processo `P1` escreve valor `a` em `x`.

```text
R2(x)a
```

Processo `P2` lê `x` e obtém `a`.

---

# Consistência estrita

## Definição conceitual

Uma leitura em `x` sempre retorna o valor da escrita mais recente em `x`.

Formalmente, se uma escrita em `x` ocorreu no tempo absoluto `t`, qualquer leitura posterior deve retornar esse valor ou valor mais recente.

## Problema

Consistência estrita exige relógio global perfeito.

Em sistemas distribuídos reais:

```text
não há relógio global perfeito
mensagens atrasam
eventos concorrentes são difíceis de ordenar pelo tempo real
```

Por isso, consistência estrita é mais um ideal teórico do que um modelo prático.

---

# Linearizabilidade

Linearizabilidade é mais prática que consistência estrita, mas ainda forte.

Ideia:

```text
Cada operação parece ocorrer instantaneamente em algum ponto entre seu início e seu fim.
```

Se operação A termina antes de operação B começar, todos devem observar A antes de B.

Exemplo:

```text
W(x)=1 termina às 10:00:01
R(x) começa às 10:00:02
```

A leitura deve ver `1` ou algo posterior.

## Características

- respeita a ordem real de operações não sobrepostas;
- faz o objeto parecer único e não replicado;
- é forte e intuitiva;
- é cara em sistemas distribuídos grandes.

---

# Consistência sequencial

Consistência sequencial é mais fraca que linearizabilidade.

## Definição clássica

O resultado de qualquer execução deve ser o mesmo que se todas as operações fossem executadas em alguma ordem sequencial, e as operações de cada processo individual aparecem nessa ordem conforme foram executadas por esse processo.

Em forma curta:

```text
Todos veem uma única ordem global das operações.
Essa ordem preserva a ordem local de cada processo.
Mas não precisa respeitar o tempo real.
```

## O que ela garante

Se `P1` executa:

```text
W1(x)a
W1(y)a
```

então a ordem global deve preservar:

```text
W1(x)a antes de W1(y)a
```

Se `P2` executa:

```text
W2(y)b
W2(x)b
```

então a ordem global deve preservar:

```text
W2(y)b antes de W2(x)b
```

Mas a intercalação entre operações de `P1` e `P2` pode variar.

---

## Diferença entre linearizabilidade e consistência sequencial

| Critério | Linearizabilidade | Consistência sequencial |
|---|---|---|
| Respeita tempo real? | Sim | Não necessariamente |
| Exige ordem global? | Sim | Sim |
| Preserva ordem local dos processos? | Sim | Sim |
| Mais forte? | Sim | Não |
| Mais cara? | Geralmente sim | Geralmente menos |

Resumo:

```text
Linearizabilidade = ordem global + ordem local + tempo real.
Sequencial = ordem global + ordem local, sem exigir tempo real.
```

---

# Consistência causal

Consistência causal usa a relação de causalidade entre operações.

Ideia:

```text
Operações causalmente relacionadas devem ser vistas por todos na mesma ordem.
Operações concorrentes podem ser vistas em ordens diferentes.
```

## Quando há causalidade?

Há causalidade quando uma operação pode ter influenciado outra.

Exemplo:

```text
P1 escreve x = 1.
P2 lê x = 1.
Depois P2 escreve y = 2.
```

A escrita em `y` depende causalmente da escrita em `x`, porque `P2` viu `x = 1` antes de escrever `y`.

Então todos devem observar:

```text
W(x)=1 antes de W(y)=2
```

## Operações concorrentes

Se duas escritas não têm relação causal, podem ser vistas em ordens diferentes.

Exemplo:

```text
P1 escreve x = a
P2 escreve x = b
```

Se não houve comunicação ou dependência entre elas, são concorrentes.

Uma réplica pode ver:

```text
a depois b
```

Outra pode ver:

```text
b depois a
```

Isso ainda pode ser causalmente consistente.

---

# FIFO / PRAM consistency

FIFO consistency, também chamada de PRAM consistency em algumas discussões, garante que as escritas de cada processo sejam vistas pelos demais na ordem em que foram emitidas por aquele processo.

Exemplo:

```text
P1: W(x)=1 depois W(x)=2
```

Todos devem ver a escrita `1` antes da escrita `2`.

Mas escritas de processos diferentes podem ser vistas em ordens diferentes.

## Comparação

```text
Sequencial: todos concordam em uma ordem global.
Causal: todos concordam na ordem de operações causalmente relacionadas.
FIFO/PRAM: todos concordam apenas na ordem das escritas de cada processo.
```

---

# Modelos com sincronização

Alguns modelos mais fracos assumem que o programador usa operações de sincronização, como locks.

Ideia:

```text
O sistema só precisa tornar os dados consistentes em pontos de sincronização.
```

Exemplo:

```text
acquire(lock)
ler/escrever dados
release(lock)
```

Entre `acquire` e `release`, o processo pode operar localmente.

A consistência precisa ser garantida nos momentos de sincronização.

---

## Weak consistency

Regras conceituais:

1. Operações sobre variáveis de sincronização são sequencialmente consistentes.
2. Antes de acessar uma variável de sincronização, todas as escritas anteriores devem estar concluídas.
3. Antes de acessar dados compartilhados, sincronizações anteriores devem estar concluídas.

Interpretação simples:

```text
Dados comuns só precisam estar consistentes depois de sincronização.
```

---

## Release consistency

Refina a weak consistency separando sincronização em duas operações:

```text
acquire = entrar na região protegida
release = sair da região protegida
```

No `acquire`, o processo obtém atualizações necessárias.

No `release`, propaga suas alterações.

---

## Entry consistency

Associa dados específicos a objetos de sincronização específicos.

Exemplo:

```text
lock L protege variável x
```

Ao fazer `acquire(L)`, o sistema só precisa atualizar os dados associados a `L`, não todos os dados compartilhados.

Vantagem:

```text
menos tráfego e maior desempenho
```

---

# Consistência vs coerência

## Consistência

Refere-se ao comportamento de um conjunto de dados.

Exemplo:

```text
x e y juntos devem obedecer a uma ordem consistente de operações.
```

## Coerência

Refere-se normalmente a um único item de dado replicado.

Exemplo:

```text
Todas as cópias de x devem eventualmente concordar na ordem das escritas em x.
```

Resumo:

```text
Consistência = conjunto de dados.
Coerência = uma variável ou item replicado.
```

---

# 7.2.2 — Consistência eventual

Consistência eventual é um modelo fraco, mas muito usado.

## Definição conceitual

Se nenhuma nova atualização ocorrer, todas as réplicas acabarão convergindo para o mesmo valor.

Em forma curta:

```text
Se o sistema parar de receber escritas, as réplicas eventualmente ficam iguais.
```

---

## Quando faz sentido?

Faz sentido quando:

- a aplicação tolera leituras temporariamente antigas;
- há muitas leituras e poucas escritas;
- usuários tendem a acessar a mesma réplica;
- baixa latência é mais importante que consistência imediata;
- operação offline ou desconectada é necessária.

Exemplos:

- DNS;
- caches;
- feeds sociais;
- perfis de usuário;
- catálogos;
- sistemas mobile;
- replicação geográfica com baixa exigência de consistência imediata.

---

## Problema

Durante o período de convergência, clientes podem observar valores diferentes.

Exemplo:

```text
Replica A: x = 10
Replica B: x = 20
Replica C: x = 10
```

Isso pode ser aceitável em algumas aplicações e inaceitável em outras.

---

## Conflitos

Se duas réplicas recebem escritas concorrentes, pode haver conflito.

Exemplo:

```text
Replica A: usuário altera endereço para Goiânia.
Replica B: usuário altera endereço para Brasília.
```

Depois, o sistema precisa decidir:

```text
qual valor vence?
combinar valores?
pedir resolução manual?
usar timestamp?
usar regra de aplicação?
```

---

# 7.2.3 — Consistência contínua

Consistência contínua não exige que todas as réplicas estejam sempre iguais.

Ela define limites aceitáveis de divergência.

Em vez de perguntar:

```text
As réplicas estão iguais?
```

pergunta:

```text
Quão diferentes elas podem estar?
```

O livro destaca três dimensões principais:

1. desvio numérico;
2. desvio de staleness;
3. desvio de ordenação.

---

## 1. Desvio numérico

Limita a diferença no valor dos dados entre réplicas.

Exemplo:

```text
Preço de ação em réplica A = 100
Preço de ação em réplica B = 103
```

Se o limite permitido é 5, isso é aceitável.

Se o limite permitido é 2, não é aceitável.

Uso típico:

```text
dados financeiros aproximados
contadores
estoques
métricas agregadas
```

---

## 2. Desvio de staleness

Limita o quão velha uma réplica pode estar.

Exemplo:

```text
Réplica pode estar no máximo 30 segundos desatualizada.
```

Se a última atualização foi há 10 segundos, tudo bem.

Se foi há 2 minutos, precisa atualizar.

Uso típico:

```text
caches Web
notícias
feeds
informações de catálogo
```

---

## 3. Desvio de ordenação

Limita quantas escritas pendentes ou tentativas podem existir sem sincronização.

Exemplo:

```text
Uma réplica pode ter no máximo 5 operações pendentes antes de sincronizar.
```

Isso controla o grau de divergência na ordem de operações.

---

## Conit

O livro usa a ideia de **conit**, uma unidade de consistência.

Um conit define sobre qual parte dos dados a consistência será medida.

Exemplo:

```text
um registro
um conjunto de registros
uma tabela
um arquivo
um bloco
```

Quanto menor o conit:

```text
mais controle fino
maior overhead
```

Quanto maior o conit:

```text
menos overhead
menos precisão
```

---

# 7.3 — Modelos de consistência centrados no cliente

Modelos centrados no cliente olham para a experiência de um único cliente ao acessar diferentes réplicas ao longo do tempo.

Cenário típico:

```text
Cliente acessa réplica A hoje.
Depois acessa réplica B.
Depois acessa réplica C.
```

O sistema pode ser eventualmente consistente globalmente, mas ainda precisa evitar comportamentos absurdos para esse cliente.

Exemplo ruim:

```text
Usuário lê um e-mail em São Paulo.
Depois abre o e-mail em Goiânia.
O e-mail desaparece porque a réplica local está atrasada.
```

Modelos client-centric evitam esse tipo de regressão.

---

## Notação de versões

`xi` representa uma versão do dado `x`.

Exemplo:

```text
x1 = versão antiga
x2 = versão mais nova
```

Se `x2` foi produzida a partir de `x1`, então:

```text
x2 segue de x1
```

Em forma conceitual:

```text
x1 -> x2
```

Se duas versões foram geradas independentemente, são concorrentes.

---

# 7.3.1 — Monotonic reads

## Definição

Se um processo lê uma versão de `x`, qualquer leitura posterior de `x` por esse mesmo processo deve retornar a mesma versão ou uma versão mais recente.

Em forma curta:

```text
Depois que o cliente viu uma versão, ele não pode ver uma versão mais antiga.
```

---

## Exemplo

Usuário abre mailbox em São Paulo.

```text
Vê mensagens: A, B, C
```

Depois abre mailbox em Goiânia.

Com monotonic reads, ele deve ver:

```text
A, B, C
```

ou algo mais recente:

```text
A, B, C, D
```

Não pode ver apenas:

```text
A, B
```

---

## Violação

```text
Primeira leitura: versão x5
Segunda leitura: versão x3
```

Isso viola monotonic reads.

---

## Intuição para prova

Monotonic reads impede que o usuário “volte no tempo” em leituras.

---

# 7.3.2 — Monotonic writes

## Definição

Uma operação de escrita por um processo deve ser concluída antes de qualquer escrita posterior desse mesmo processo.

Em forma curta:

```text
As escritas de um mesmo cliente devem ser aplicadas na ordem em que ele as fez.
```

---

## Exemplo

Usuário atualiza um documento:

```text
W1: título = "Versão 1"
W2: título = "Versão 2"
```

O sistema não pode aplicar `W2` antes de `W1`.

---

## Violação

Se uma réplica recebe:

```text
W2 antes de W1
```

pode acabar com estado incorreto.

---

## Intuição para prova

Monotonic writes preserva a ordem das escritas de um mesmo cliente.

---

# 7.3.3 — Read your writes

## Definição

Depois que um processo escreve um valor, leituras posteriores desse mesmo processo devem retornar esse valor ou uma versão mais recente.

Em forma curta:

```text
O cliente deve conseguir ver suas próprias escritas.
```

---

## Exemplo

Usuário altera a própria senha.

Logo depois tenta fazer login.

Com read-your-writes, o login deve reconhecer a nova senha.

Outro exemplo:

Usuário publica um comentário.

Depois atualiza a página.

Ele deve ver o próprio comentário.

---

## Violação

```text
Usuário muda foto de perfil.
Atualiza a página.
Foto antiga aparece.
```

Isso viola read-your-writes.

---

## Intuição para prova

Read-your-writes impede que o sistema ignore alterações feitas pelo próprio usuário.

---

# 7.3.4 — Writes follow reads

## Definição

Uma escrita feita por um processo depois de uma leitura deve ser realizada sobre uma versão igual ou mais recente que a versão lida.

Em forma curta:

```text
Se o cliente leu algo e depois escreveu, sua escrita deve respeitar aquilo que ele leu.
```

---

## Exemplo

Usuário lê:

```text
post = "A"
```

Depois comenta:

```text
"Concordo com A"
```

A escrita do comentário depende da leitura do post.

O sistema não deve aplicar o comentário em uma versão anterior em que o post ainda não existia.

---

## Outro exemplo

Cliente lê saldo:

```text
saldo = 100
```

Depois faz operação baseada nesse saldo.

A escrita posterior deve seguir a leitura feita, não uma versão mais antiga.

---

## Intuição para prova

Writes follow reads preserva causalidade entre leitura anterior e escrita posterior do mesmo cliente.

---

# Comparação dos modelos client-centric

| Modelo | Garante | Exemplo |
|---|---|---|
| Monotonic reads | Leituras não voltam para versões antigas | E-mail visto antes continua visível |
| Monotonic writes | Escritas do cliente são aplicadas na ordem | Editar documento v1 antes de v2 |
| Read your writes | Cliente vê suas próprias atualizações | Ver próprio comentário após postar |
| Writes follow reads | Escrita posterior respeita leitura anterior | Comentário depende do post lido |

---

# 7.4 — Gerenciamento de réplicas

Replica management trata de decisões práticas:

```text
Onde colocar réplicas?
Que conteúdo replicar?
Como distribuir atualizações?
Quem inicia propagação?
```

A parte de consistência define o contrato.

A parte de gerenciamento define a organização operacional das réplicas.

---

# 7.4.1 — Encontrando a melhor localização para servidores

A pergunta é:

```text
Onde colocar K réplicas entre N possíveis locais?
```

Esse problema é computacionalmente difícil e normalmente resolvido por heurísticas.

Critérios possíveis:

- latência entre cliente e servidor;
- largura de banda;
- distância lógica;
- número de saltos;
- custo de hospedagem;
- custo de atualização;
- disponibilidade;
- energia;
- requisitos de QoS.

---

## QoS-aware placement

Escolhe localizações otimizando qualidade de serviço.

Exemplo:

```text
minimizar latência média dos clientes
```

Ou:

```text
garantir que todo cliente tenha latência abaixo de 100 ms
```

---

## Consistency-aware placement

Considera o custo de manter réplicas consistentes.

Exemplo:

```text
Uma réplica próxima dos clientes reduz latência de leitura,
mas aumenta custo de propagação de atualizações.
```

Se os dados mudam muito, colocar muitas réplicas pode ser ruim.

---

## Ideia prática

Não existe “melhor local” universal.

A escolha depende de:

```text
padrão de acesso
frequência de atualização
geografia dos clientes
custo de rede
nível de consistência exigido
```

---

# 7.4.2 — Replicação e posicionamento de conteúdo

Há três tipos principais de réplicas.

---

## 1. Réplicas permanentes

São réplicas estáveis e planejadas.

Exemplo:

```text
servidores principais de um banco de dados distribuído
data centers regionais
servidores de DNS
```

Características:

- existem por longo prazo;
- são administradas explicitamente;
- fazem parte da arquitetura do sistema;
- normalmente participam dos protocolos de consistência.

---

## 2. Réplicas iniciadas pelo servidor

São criadas dinamicamente pelo sistema para melhorar desempenho.

Exemplo:

```text
conteúdo popular é copiado para um servidor próximo dos clientes.
```

O servidor percebe aumento de demanda e cria réplica.

Características:

- criadas sob demanda;
- controladas pelo sistema;
- podem ser removidas quando a demanda cai;
- melhoram latência e reduzem carga.

---

## 3. Réplicas iniciadas pelo cliente

São caches mantidos pelos clientes.

Exemplo:

```text
cache do navegador
cache local de arquivo
cache de aplicação
```

Características:

- ficam próximas do cliente;
- reduzem latência;
- reduzem carga do servidor;
- podem ficar desatualizadas;
- normalmente exigem validação ou expiração.

---

## Comparação

| Tipo | Quem cria | Duração | Exemplo | Problema principal |
|---|---|---|---|---|
| Permanente | Administrador/sistema | Longa | data center | custo de consistência |
| Iniciada pelo servidor | Servidor/sistema | Variável | CDN interna | decidir quando criar/remover |
| Iniciada pelo cliente | Cliente | Curta/média | cache Web | staleness |

---

# 7.4.3 — Distribuição de conteúdo

Há três perguntas principais:

1. O que propagar?
2. Para onde propagar?
3. Quem inicia a propagação?

---

# O que propagar?

## 1. Invalidations

Em vez de enviar o novo valor, envia-se apenas uma notificação de que o dado está inválido.

Exemplo:

```text
x está desatualizado
```

A réplica descarta ou marca sua cópia como inválida.

Quando precisar do dado, busca versão atual.

### Quando é bom?

Quando há muitas escritas e poucas leituras.

Exemplo:

```text
x muda várias vezes antes de alguém ler.
```

Enviar todas as versões seria desperdício.

---

## 2. Transferir dados modificados

Envia-se o novo valor do dado.

Exemplo:

```text
x = 42
```

### Quando é bom?

Quando há muitas leituras depois das escritas.

Se a réplica provavelmente será lida, faz sentido já enviar o valor atualizado.

---

## 3. Propagar operações

Em vez de enviar o novo estado, envia-se a operação que deve ser executada.

Exemplo:

```text
incrementar contador em 1
```

ou:

```text
adicionar item X ao carrinho
```

Isso é comum em **active replication**.

### Vantagem

Pode economizar banda se a operação é pequena e o estado é grande.

### Exigência

Todas as réplicas devem executar operações:

```text
na mesma ordem
de forma determinística
```

Se a ordem divergir, o estado final pode divergir.

---

# Para onde propagar?

Não é obrigatório atualizar todas as réplicas imediatamente.

Opções:

```text
todas as réplicas
apenas réplicas permanentes
apenas réplicas próximas
apenas réplicas que têm clientes interessados
apenas quando alguém pedir
```

A escolha depende do protocolo de distribuição.

---

# Quem inicia a propagação?

## Push-based protocols

Também chamados de server-based protocols.

O servidor envia atualizações para réplicas sem que elas peçam.

Exemplo:

```text
Servidor atualiza x e envia x novo para todas as réplicas.
```

### Quando usar?

Quando:

- consistência forte é desejada;
- read-write ratio é alto;
- as réplicas são compartilhadas por muitos clientes;
- é provável que a atualização seja útil.

### Vantagem

Dados atualizados ficam disponíveis imediatamente.

### Desvantagem

Pode desperdiçar banda se ninguém ler a atualização.

---

## Pull-based protocols

Também chamados de client-based protocols.

A réplica ou cliente pede atualizações quando precisa.

Exemplo:

```text
Cache pergunta ao servidor: minha cópia ainda é válida?
```

### Quando usar?

Quando:

- caches são pouco usados;
- clientes são muitos e dispersos;
- dados mudam frequentemente;
- consistência pode ser relaxada.

### Vantagem

Evita enviar atualizações inúteis.

### Desvantagem

Primeira leitura pode ficar mais lenta.

---

## Push vs pull

| Critério | Push | Pull |
|---|---|---|
| Iniciador | Servidor | Cliente/réplica |
| Melhor para | Muitas leituras | Muitas escritas ou acessos raros |
| Consistência | Mais forte/imediata | Mais fraca/sob demanda |
| Custo | Pode enviar dados inúteis | Pode atrasar leitura |
| Exemplo | Réplicas permanentes | Cache Web |

---

## Leases

Lease é uma permissão temporária dada a uma réplica ou cliente.

Enquanto o lease está válido, o servidor se compromete a avisar sobre atualizações ou permitir certas operações.

Exemplo:

```text
Cliente recebe lease de 30 segundos para cachear x.
Durante 30 segundos, pode usar x com confiança definida pelo protocolo.
```

Leases combinam ideias de push e pull.

Se o lease expira, o cliente precisa renovar ou validar.

---

# 7.4.4 — Gerenciando objetos replicados

Quando objetos são replicados, chamadas entre objetos replicados podem causar multiplicação de mensagens.

Exemplo:

```text
Objeto B tem 3 réplicas.
Objeto C tem 3 réplicas.
Cada réplica de B chama cada réplica de C.
```

Sem controle, uma única invocação lógica pode virar várias invocações físicas duplicadas.

---

## Problema

Se cada réplica de B encaminha a mesma requisição para C, C pode executar a mesma operação várias vezes.

Isso quebra a semântica esperada.

Exemplo:

```text
Cliente pede: debitar R$10.
Três réplicas encaminham.
Sistema debita R$30.
```

---

## Solução com coordenador

Uma abordagem:

1. As réplicas de B recebem a mesma invocação.
2. Cada réplica atribui o mesmo identificador único à invocação.
3. Apenas um coordenador das réplicas de B encaminha a requisição para C.
4. As outras réplicas seguram suas cópias.
5. Em C, apenas um coordenador retorna a resposta.
6. As demais réplicas seguram respostas duplicadas.

Resultado:

```text
A invocação lógica ocorre uma vez, apesar da replicação.
```

---

## Ideia para prova

Replicação de objetos exige evitar:

```text
requisições duplicadas
respostas duplicadas
efeitos colaterais duplicados
```

---

# 7.5 — Protocolos de consistência

Um modelo de consistência diz o que deve acontecer.

Um protocolo de consistência diz como implementar.

O escopo da prova cobre protocolos para consistência sequencial:

1. primary-based protocols;
2. replicated-write protocols.

---

# 7.5.1 — Consistência sequencial: protocolos primary-based

A ideia central:

```text
Existe uma réplica primária responsável por ordenar escritas.
```

Leituras podem ocorrer localmente, mas escritas passam pela primária.

Isso simplifica a ordenação global.

---

# Remote-write protocols / primary-backup

## Funcionamento

1. Cliente quer escrever em `x`.
2. A requisição é enviada para a réplica primária de `x`.
3. A primária executa a escrita localmente.
4. A primária encaminha a atualização para os backups.
5. Backups aplicam a atualização.
6. Backups enviam ACK para a primária.
7. Quando os ACKs necessários chegam, a primária responde ao cliente.

---

## Propriedade

A primária impõe uma ordem única sobre todas as escritas.

Logo, todos veem as escritas na mesma ordem.

Isso implementa consistência sequencial de forma direta.

---

## Variante bloqueante

A primária só responde ao cliente depois que os backups confirmam.

Vantagem:

```text
maior segurança e melhor tolerância a falhas
```

Desvantagem:

```text
maior latência
```

---

## Variante não bloqueante

A primária responde ao cliente depois de aplicar localmente, antes de todos os backups confirmarem.

Vantagem:

```text
menor latência
```

Desvantagem:

```text
se a primária falha antes de propagar, a escrita pode ser perdida
```

---

## Vantagens do primary-backup

- simples de entender;
- ordenação centralizada das escritas;
- bom para consistência sequencial;
- leituras podem ser locais;
- fácil raciocinar sobre estado.

---

## Desvantagens do primary-backup

- primária pode virar gargalo;
- falha da primária exige eleição ou failover;
- escritas remotas podem ser lentas para clientes longe da primária;
- bloqueio aumenta latência;
- não bloqueio reduz segurança.

---

# Local-write protocols

Nesse modelo, a cópia primária pode migrar para o processo que deseja escrever.

## Funcionamento

1. Processo quer atualizar `x`.
2. Localiza a primária atual de `x`.
3. A primária é movida para perto do processo ou para sua réplica local.
4. O processo realiza escritas localmente.
5. Depois, atualizações são propagadas para os backups.

---

## Vantagem

Bom quando o mesmo processo faz várias escritas sucessivas.

Exemplo:

```text
Usuário edita documento offline.
Faz várias alterações localmente.
Depois reconecta e propaga as mudanças.
```

---

## Uso em computadores móveis

Antes de desconectar:

```text
dispositivo recebe a cópia primária dos dados que pretende alterar
```

Durante desconexão:

```text
faz atualizações localmente
```

Ao reconectar:

```text
propaga mudanças para as outras réplicas
```

---

## Desvantagens

- localizar e mover a primária custa tempo;
- conflitos podem ocorrer se outro processo também quiser escrever;
- propagação posterior pode gerar inconsistência temporária;
- protocolo é mais complexo que primary-backup fixo.

---

# 7.5.2 — Consistência sequencial: replicated-write protocols

Em replicated-write protocols, escritas podem ocorrer em múltiplas réplicas, não apenas em uma primária.

Duas abordagens importantes:

1. active replication;
2. quorum-based protocols.

---

# Active replication

## Ideia

Cada réplica possui um processo capaz de executar a operação.

A operação de escrita é enviada para todas as réplicas.

Exemplo:

```text
operação: depositar 100
```

Todas as réplicas executam:

```text
saldo = saldo + 100
```

---

## Requisito crucial

Todas as réplicas devem executar:

```text
as mesmas operações
na mesma ordem
de forma determinística
```

Se a ordem for diferente, os estados podem divergir.

Exemplo:

```text
Operação A: saldo = saldo + 100
Operação B: saldo = saldo * 1.01
```

Ordem A depois B:

```text
(1000 + 100) * 1.01 = 1111
```

Ordem B depois A:

```text
1000 * 1.01 + 100 = 1110
```

Resultado diferente.

---

## Necessidade de multicast totalmente ordenado

Para active replication funcionar, é necessário que todas as réplicas recebam operações na mesma ordem.

Solução típica:

```text
multicast totalmente ordenado
```

Uma forma prática:

```text
sequenciador central atribui número de sequência a cada operação
```

Depois, todas as réplicas executam na ordem dos números.

---

## Problema do sequenciador

O sequenciador pode virar:

- gargalo de desempenho;
- ponto único de falha;
- limite de escalabilidade.

---

## Múltiplos sequenciadores

Para escalar, pode-se usar múltiplos sequenciadores.

Ideia:

```text
clientes são divididos em grupos
cada grupo usa um sequenciador
sequenciadores ordenam atualizações entre si usando timestamps de Lamport
```

Trade-off:

```text
poucos sequenciadores = simples, mas gargalo
muitos sequenciadores = mais paralelismo, mas mais coordenação
```

---

# Quorum-based protocols

## Ideia

Para ler ou escrever um dado replicado, o cliente precisa obter permissão de um conjunto de réplicas.

Esse conjunto é chamado de quorum.

Com `N` réplicas:

```text
NR = tamanho do quorum de leitura
NW = tamanho do quorum de escrita
```

Para garantir consistência, normalmente exigimos:

```text
NR + NW > N
NW > N/2
```

---

## Por que essas condições?

### `NR + NW > N`

Garante que todo quorum de leitura intersecta todo quorum de escrita.

Assim, uma leitura tem contato com pelo menos uma réplica que viu a escrita mais recente.

### `NW > N/2`

Garante que dois quorums de escrita sempre se intersectam.

Assim, duas escritas concorrentes não podem ocorrer em conjuntos totalmente separados.

---

## Exemplo com N = 5

Escolha:

```text
NR = 3
NW = 3
```

Verificação:

```text
NR + NW = 6 > 5
NW = 3 > 2.5
```

Funciona.

Para escrever, cliente precisa de 3 réplicas.

Para ler, cliente consulta 3 réplicas e escolhe a versão mais recente.

---

## Version numbers

Cada escrita cria uma nova versão.

Exemplo:

```text
versão 8 -> versão 9
```

Ao ler, cliente consulta várias réplicas e escolhe a maior versão válida.

---

## Trade-offs de quorum

Se `NR` é pequeno e `NW` é grande:

```text
leituras rápidas
escritas caras
```

Bom para sistemas com muitas leituras.

Se `NR` é grande e `NW` é pequeno:

```text
leituras caras
escritas rápidas
```

Menos comum, mas útil quando escritas precisam ser rápidas.

---

## Vantagens de quorum

- não depende de uma única primária;
- tolera falhas de algumas réplicas;
- permite configurar custo de leitura vs escrita;
- evita ponto único de falha óbvio.

---

## Desvantagens de quorum

- mais complexo;
- exige versionamento;
- exige coordenação entre réplicas;
- pode bloquear se não houver réplicas suficientes disponíveis;
- latência pode ser alta por exigir contato com múltiplos servidores.

---

# Comparação: primary-based vs replicated-write

| Critério | Primary-based | Replicated-write |
|---|---|---|
| Onde ocorre escrita | Primária | Várias réplicas |
| Ordenação | Primária ordena | Precisa de multicast ordenado ou quorum |
| Simplicidade | Maior | Menor |
| Gargalo | Primária | Coordenação distribuída |
| Falha crítica | Falha da primária | Falta de quorum ou divergência |
| Bom para | Sistemas simples e fortes | Alta disponibilidade e replicação ativa |

---

# Comparação de modelos de consistência

| Modelo | Ideia | Força | Custo |
|---|---|---:|---:|
| Estrita | Leitura vê escrita mais recente em tempo absoluto | Muito forte | Impraticável |
| Linearizável | Operação parece instantânea entre início e fim | Muito forte | Alto |
| Sequencial | Todos veem uma ordem global que preserva ordem local | Forte | Alto/médio |
| Causal | Todos preservam ordem causal | Médio | Médio |
| FIFO/PRAM | Preserva ordem das escritas de cada processo | Fraco/médio | Menor |
| Eventual | Réplicas convergem se não houver novas escritas | Fraco | Baixo |
| Contínua | Divergência é limitada por métricas | Ajustável | Ajustável |

---

# Comparação: data-centric vs client-centric

## Data-centric

Pergunta:

```text
O que todos os processos podem observar sobre o conjunto de dados compartilhados?
```

Exemplo:

```text
Todos veem as escritas na mesma ordem?
```

## Client-centric

Pergunta:

```text
O que um cliente individual pode observar ao mudar de réplica?
```

Exemplo:

```text
Esse cliente pode ver uma versão mais antiga do que já viu antes?
```

---

# Exemplos rápidos para prova

## Exemplo 1 — Monotonic reads

```text
Cliente lê x = 10 na réplica A.
Depois lê x = 8 na réplica B.
```

Violação:

```text
monotonic reads
```

---

## Exemplo 2 — Read your writes

```text
Cliente escreve x = 20.
Depois lê x = 10.
```

Violação:

```text
read your writes
```

---

## Exemplo 3 — Monotonic writes

```text
Cliente faz W1: x = 1.
Depois faz W2: x = 2.
Réplica aplica W2 antes de W1.
```

Violação:

```text
monotonic writes
```

---

## Exemplo 4 — Writes follow reads

```text
Cliente lê post P.
Depois comenta em P.
Comentário é aplicado em réplica que ainda não tem P.
```

Violação:

```text
writes follow reads
```

---

## Exemplo 5 — Eventual consistency

```text
Replica A tem x = 1.
Replica B tem x = 2.
Não há novas escritas.
Depois de algum tempo, A e B convergem para x = 2.
```

Isso é compatível com consistência eventual.

---

## Exemplo 6 — Quorum

```text
N = 7
NR = 3
NW = 5
```

Verificação:

```text
NR + NW = 8 > 7
NW = 5 > 3.5
```

Configuração válida.

---

# Fórmulas e condições importantes

## Quorum

```text
NR + NW > N
NW > N/2
```

## Maioria simples

```text
maioria = floor(N/2) + 1
```

## Exemplo

```text
N = 5
maioria = 3
```

---

# Perguntas típicas de prova

## 1. Por que replicar dados?

Para aumentar confiabilidade, melhorar desempenho e escalar o sistema geograficamente ou em número de clientes.

---

## 2. Qual é o custo da replicação?

Manter cópias consistentes. Quando uma réplica é atualizada, as outras precisam ser atualizadas ou o sistema deve aceitar inconsistência temporária.

---

## 3. Quando replicação melhora desempenho?

Quando há muitas leituras em relação a escritas e quando réplicas podem ser colocadas próximas dos clientes.

---

## 4. Quando replicação pode piorar desempenho?

Quando há muitas escritas, porque cada atualização precisa ser propagada, ordenada ou reconciliada.

---

## 5. O que é consistência sequencial?

É a propriedade de que o resultado equivale a alguma execução sequencial das operações, preservando a ordem das operações de cada processo.

---

## 6. Qual a diferença entre consistência sequencial e linearizabilidade?

Linearizabilidade respeita tempo real. Consistência sequencial não precisa respeitar tempo real, apenas ordem local dos processos.

---

## 7. O que é consistência causal?

Operações causalmente relacionadas devem ser vistas por todos na mesma ordem. Operações concorrentes podem ser vistas em ordens diferentes.

---

## 8. O que é consistência eventual?

Se não houver novas atualizações, todas as réplicas eventualmente convergem para o mesmo estado.

---

## 9. O que é consistência contínua?

É um modelo que permite divergência controlada entre réplicas, usando limites de desvio numérico, staleness e ordenação.

---

## 10. O que são monotonic reads?

Depois que um cliente lê uma versão, ele nunca deve ler uma versão mais antiga.

---

## 11. O que são monotonic writes?

Escritas de um mesmo cliente devem ser aplicadas na ordem em que foram feitas.

---

## 12. O que é read your writes?

Depois de escrever, o cliente deve conseguir ler sua própria escrita ou versão posterior.

---

## 13. O que é writes follow reads?

Uma escrita feita depois de uma leitura deve ser aplicada sobre uma versão que inclui aquilo que foi lido.

---

## 14. Quais são os tipos de réplicas?

Réplicas permanentes, réplicas iniciadas pelo servidor e réplicas iniciadas pelo cliente.

---

## 15. O que pode ser propagado entre réplicas?

Invalidations, dados modificados ou operações.

---

## 16. Diferença entre push e pull?

Push: servidor envia atualizações sem solicitação.

Pull: cliente/réplica solicita atualizações quando precisa.

---

## 17. Como funciona primary-backup?

Todas as escritas vão para a primária. A primária ordena e propaga para backups. Leituras podem ocorrer localmente.

---

## 18. Qual o problema do primary-backup?

A primária pode ser gargalo e sua falha exige recuperação ou eleição.

---

## 19. Como funciona active replication?

A operação é enviada para todas as réplicas, que executam a mesma operação na mesma ordem.

---

## 20. Por que active replication precisa de multicast totalmente ordenado?

Porque réplicas devem executar operações na mesma ordem para terminar no mesmo estado.

---

## 21. Como funciona quorum?

Leituras e escritas precisam obter respostas/permissões de subconjuntos de réplicas. Os quorums devem se intersectar para garantir que leituras encontrem escritas recentes e escritas concorrentes não sejam independentes.

---

# Respostas curtas para decorar

## Replicação

```text
Manter múltiplas cópias de dados ou serviços para aumentar confiabilidade, desempenho ou escalabilidade.
```

## Modelo de consistência

```text
Contrato que define quais resultados de leitura são válidos diante de escritas concorrentes ou replicadas.
```

## Protocolo de consistência

```text
Mecanismo concreto que implementa um modelo de consistência.
```

## Consistência sequencial

```text
Todos observam uma ordem sequencial global que preserva a ordem local de cada processo.
```

## Linearizabilidade

```text
Cada operação parece ocorrer instantaneamente entre seu início e fim, respeitando tempo real.
```

## Consistência causal

```text
Operações causalmente relacionadas são vistas por todos na mesma ordem.
```

## Consistência eventual

```text
Sem novas escritas, todas as réplicas eventualmente convergem.
```

## Consistência contínua

```text
Réplicas podem divergir dentro de limites definidos.
```

## Monotonic reads

```text
Cliente não lê versões mais antigas do que já leu.
```

## Monotonic writes

```text
Escritas do cliente são aplicadas na ordem emitida.
```

## Read your writes

```text
Cliente vê suas próprias escritas.
```

## Writes follow reads

```text
Escrita posterior respeita a versão lida anteriormente.
```

## Primary-backup

```text
Uma primária ordena escritas e propaga para backups.
```

## Active replication

```text
Todas as réplicas executam as mesmas operações na mesma ordem.
```

## Quorum

```text
Leituras e escritas usam subconjuntos sobrepostos de réplicas.
```

---

# Prioridade para estudar

## Prioridade alta

- Razões para replicação.
- Trade-off replicação vs consistência.
- Consistência sequencial.
- Linearizabilidade.
- Consistência causal.
- Eventual consistency.
- Continuous consistency: numerical, staleness e ordering deviation.
- Monotonic reads.
- Monotonic writes.
- Read your writes.
- Writes follow reads.
- Tipos de réplicas.
- Push vs pull.
- Invalidations vs transferência de dados vs propagação de operações.
- Primary-backup.
- Active replication.
- Quorum e fórmulas `NR + NW > N`, `NW > N/2`.

## Prioridade média

- Weak, release e entry consistency.
- Conits.
- Server placement.
- Réplicas iniciadas pelo servidor.
- Leases.
- Gerenciamento de objetos replicados.
- Sequencer e multicast totalmente ordenado.

## Prioridade baixa

- Detalhes matemáticos de posicionamento de servidores.
- Taxonomias completas de replica placement.
- Exemplos específicos fora do escopo, como Web/CDN em 7.6.
