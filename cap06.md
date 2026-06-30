# Resumo — Capítulo 6: Serviços de Nomes

**Livro:** *Distributed Systems*, Maarten van Steen e Andrew S. Tanenbaum, 4ª edição.  
**Escopo da prova:** Cap. 6.1 e Cap. 6.3 até 6.3.3.

**Entra na prova:**

- 6.1 — Names, identifiers, and addresses
- 6.3 — Structured naming
- 6.3.1 — Name spaces
- 6.3.2 — Name resolution
- 6.3.3 — The implementation of a name space

**Fora do escopo:**

- 6.2 — Flat naming
- 6.3.4 — DNS
- 6.3.5 — NFS
- 6.4 — Attribute-based naming
- 6.5 — Named-data networking

---

# Visão geral

Serviços de nomes resolvem um problema básico:

```text
Dado um nome, como encontrar a entidade ou o endereço necessário para acessá-la?
```

Em sistemas distribuídos, isso é mais difícil porque:

- entidades podem estar em máquinas diferentes;
- entidades podem mudar de endereço;
- entidades podem ter vários pontos de acesso;
- o sistema de nomes pode precisar ser distribuído;
- o sistema precisa escalar para muitos nomes e muitas consultas;
- falhas em servidores de nomes podem tornar recursos inacessíveis.

Exemplo intuitivo:

```text
Nome: www.exemplo.com
Endereço resolvido: 192.0.2.10:443
Entidade acessada: serviço Web
```

O usuário quer usar o nome.  
O sistema precisa descobrir onde acessar a entidade.

---

# 6.1 — Names, identifiers, and addresses

## Entidade

Uma **entidade** é qualquer coisa que possa ser nomeada em um sistema distribuído.

Exemplos:

- host;
- impressora;
- disco;
- arquivo;
- processo;
- usuário;
- mailbox;
- página Web;
- janela gráfica;
- mensagem;
- conexão de rede;
- serviço.

A entidade pode oferecer operações.

Exemplo:

```text
Impressora:
- imprimir documento
- consultar status
- cancelar job
```

```text
Conexão de rede:
- enviar dados
- receber dados
- configurar QoS
- consultar estado
```

---

## Nome

Um **nome** é uma cadeia de bits ou caracteres usada para referenciar uma entidade.

Exemplos:

```text
/home/rian/arquivo.txt
www.ufg.br
processo-123
printer-lab-2
```

O nome pode ser:

- legível por humanos;
- legível apenas por máquinas;
- estruturado;
- plano;
- único;
- não único;
- independente de localização;
- dependente de localização.

---

## Ponto de acesso

Para operar sobre uma entidade, é necessário acessá-la.

Um **ponto de acesso** é o local ou mecanismo pelo qual uma entidade pode ser acessada.

Exemplo:

```text
Servidor Web rodando em uma máquina específica.
```

O ponto de acesso pode ser descrito por:

```text
IP + porta
```

Exemplo:

```text
200.137.192.10:443
```

---

## Endereço

Um **endereço** é o nome de um ponto de acesso.

Formalmente:

```text
Endereço = nome de um ponto de acesso
```

Exemplo:

```text
192.168.0.10:8080
```

Esse endereço não nomeia necessariamente a entidade abstrata “serviço”.  
Ele nomeia um ponto específico pelo qual a entidade pode ser acessada.

---

## Entidade com múltiplos pontos de acesso

Uma entidade pode ter mais de um ponto de acesso.

Exemplo:

```text
Um serviço Web replicado em vários servidores.
```

Pode haver vários endereços:

```text
Servidor A: 10.0.0.1:443
Servidor B: 10.0.0.2:443
Servidor C: 10.0.0.3:443
```

Todos podem servir a mesma entidade lógica:

```text
www.exemplo.com
```

Por isso, usar diretamente um endereço como nome da entidade é ruim.

---

## Por que não usar endereço como nome?

Porque endereço muda.

Exemplos:

- servidor migra para outro host;
- processo reinicia em outra porta;
- laptop muda de rede e recebe outro IP;
- usuário troca de provedor e muda e-mail;
- serviço passa a usar balanceamento de carga;
- recurso é replicado.

Se o nome da entidade for o próprio endereço, qualquer mudança de endereço quebra referências.

Exemplo ruim:

```text
Cliente salva o endereço 10.0.0.7:9000 como nome do serviço.
```

Se o serviço migrar para:

```text
10.0.0.12:9000
```

o cliente perde a referência.

Solução melhor:

```text
Nome lógico: pagamento.servicos.local
Endereço atual: 10.0.0.12:9000
```

O nome lógico é resolvido para o endereço atual.

---

# Nome independente de localização

Um nome é **independente de localização** quando não codifica o local físico ou o ponto de acesso da entidade.

Exemplo:

```text
www.exemplo.com
```

Esse nome não diz diretamente:

- qual servidor executa o serviço;
- qual IP será usado;
- em qual cidade está o servidor;
- se há uma ou várias réplicas.

Vantagem:

```text
A entidade pode mudar de endereço sem mudar de nome.
```

Isso é essencial para mobilidade, replicação e transparência de localização.

---

# Identificador

Um **identificador** é um tipo especial de nome usado para identificar unicamente uma entidade.

Um identificador verdadeiro deve obedecer a três propriedades:

1. Um identificador se refere a no máximo uma entidade.
2. Cada entidade é referida por no máximo um identificador.
3. Um identificador sempre se refere à mesma entidade, isto é, não é reutilizado.

Em forma curta:

```text
identificador = nome único, estável e não reutilizado
```

---

## Exemplo de identificador

Bom identificador:

```text
UUID: 550e8400-e29b-41d4-a716-446655440000
```

Ele é criado para apontar para uma entidade específica.

Mau identificador:

```text
Número de telefone
```

Por quê?

Porque pode ser reutilizado por outra pessoa ou empresa.

Outro mau identificador:

```text
Endereço IP
```

Por quê?

Porque pode ser reassociado a outro host ou mudar com DHCP.

---

## Nome comum vs identificador vs endereço

| Conceito | Função | Exemplo | Problema principal |
|---|---|---|---|
| Nome comum | Referenciar entidade | `www.ufg.br` | Pode não ser único em todos os contextos |
| Identificador | Identificar entidade sem ambiguidade | UUID | Pode ser ilegível para humanos |
| Endereço | Acessar ponto de acesso | `200.137.x.x:443` | Pode mudar ou ser reutilizado |

---

## Teste conceitual

Se dois processos possuem o mesmo identificador verdadeiro, então eles se referem à mesma entidade.

```text
id(A) == id(B)  =>  mesma entidade
```

Mas isso não vale necessariamente para nomes comuns.

Exemplo:

```text
"João Silva"
```

Pode referir várias pessoas.

Também não vale necessariamente para endereços reutilizáveis.

Exemplo:

```text
IP usado hoje por máquina A pode ser usado amanhã por máquina B.
```

---

# Tema central do capítulo

O problema central de serviços de nomes é:

```text
Como resolver nomes e identificadores para endereços?
```

Existem duas abordagens gerais:

## 1. Tabela de pares nome-endereço

Mantém-se uma tabela, possivelmente distribuída:

```text
nome -> endereço
```

Exemplo conceitual:

```text
printer-lab-2 -> 10.0.0.50:631
```

## 2. Roteamento da requisição de resolução

Em vez de consultar uma tabela única, a requisição vai sendo encaminhada pelo sistema até chegar a um servidor que sabe resolver o nome.

Exemplo conceitual:

```text
raiz -> domínio -> subdomínio -> entidade
```

A prova provavelmente cobra mais a ideia geral do que os detalhes de DHT ou DNS, porque 6.2 e 6.3.4 estão fora do escopo.

---

# 6.3 — Structured naming

## Ideia

Nomes estruturados são compostos por partes menores, normalmente legíveis por humanos.

Exemplos:

```text
/home/rian/docs/prova.md
www.inf.ufg.br
org/empresa/departamento/usuario
```

Eles são mais convenientes que nomes planos, porque expressam organização hierárquica.

---

# 6.3.1 — Name spaces

Um **espaço de nomes** é uma estrutura que organiza nomes.

No livro, um espaço de nomes pode ser representado como um grafo direcionado e rotulado.

Esse grafo tem dois tipos principais de nós:

1. nós folha;
2. nós diretório.

---

## Nó folha

Um **nó folha** representa uma entidade nomeada.

Ele não possui arestas de saída.

Pode armazenar:

- endereço da entidade;
- estado da entidade;
- informação para acessar a entidade;
- metadados.

Exemplo em sistema de arquivos:

```text
arquivo.txt
```

Um arquivo é normalmente folha.

---

## Nó diretório

Um **nó diretório** possui arestas de saída.

Cada aresta tem um rótulo.

O rótulo é uma parte do nome.

Exemplo:

```text
/
└── home
    └── rian
        └── prova.md
```

Aqui:

```text
/       = nó diretório raiz
home    = aresta/rótulo para outro diretório
rian    = aresta/rótulo para outro diretório
prova.md = nó folha
```

---

## Caminho

Um nome estruturado normalmente é um caminho no grafo de nomes.

Exemplo:

```text
/home/rian/prova.md
```

Esse caminho significa:

```text
comece na raiz
procure "home"
dentro de "home", procure "rian"
dentro de "rian", procure "prova.md"
```

---

## Nome absoluto

Um **nome absoluto** começa em um ponto fixo conhecido, normalmente a raiz.

Exemplo Unix:

```text
/home/rian/prova.md
```

A resolução começa na raiz `/`.

---

## Nome relativo

Um **nome relativo** é interpretado a partir de um contexto atual.

Exemplo:

```text
docs/prova.md
```

Esse nome depende do diretório atual.

Se o diretório atual for:

```text
/home/rian
```

então:

```text
docs/prova.md
```

resolve para:

```text
/home/rian/docs/prova.md
```

---

## Diretório como tabela

Um diretório pode ser visto como uma tabela:

```text
rótulo -> identificador do próximo nó
```

Exemplo:

```text
Diretório /home/rian

docs     -> nó 103
prova.md -> nó 104
fotos    -> nó 105
```

A resolução consulta essas tabelas uma por uma.

---

## Grafo, árvore e compartilhamento

Um espaço de nomes pode parecer uma árvore, mas pode ser um grafo.

Isso acontece quando mais de um caminho leva ao mesmo nó.

Exemplo:

```text
/keys
/home/rian/keys
```

Ambos podem apontar para o mesmo nó.

Isso permite aliases, links e compartilhamento.

---

# Aliases

Um **alias** é outro nome para a mesma entidade.

Exemplo:

```text
/keys
/home/rian/keys
```

Ambos se referem ao mesmo objeto.

Há duas formas principais de implementar aliases.

---

## Hard link

Um hard link ocorre quando dois caminhos absolutos levam diretamente ao mesmo nó.

Exemplo:

```text
/keys           -> nó n5
/home/rian/keys -> nó n5
```

Os dois caminhos são nomes reais para o mesmo nó.

Característica:

```text
Não há um arquivo especial apontando para outro nome.
Os dois nomes apontam para o mesmo nó.
```

---

## Symbolic link

Um symbolic link é um nó que armazena outro caminho.

Exemplo:

```text
/home/rian/keys -> contém o caminho "/keys"
```

Ao resolver:

```text
/home/rian/keys
```

o sistema encontra um nó cujo conteúdo é outro nome:

```text
/keys
```

Então a resolução continua usando esse novo caminho.

Característica:

```text
O link simbólico aponta para um nome, não diretamente para o nó.
```

Se o destino muda ou deixa de existir, o link pode quebrar.

---

# Montagem de espaços de nomes

A resolução de nomes pode atravessar mais de um espaço de nomes.

Isso acontece por **mounting**.

Exemplo em sistema de arquivos:

```text
Diretório local: /mnt/remoto
Sistema remoto montado ali
```

Depois da montagem, o usuário acessa:

```text
/mnt/remoto/arquivo.txt
```

como se o arquivo estivesse dentro do espaço local.

---

## Mount point e mounting point

### Mount point

É o nó no espaço de nomes local onde outro espaço será anexado.

Exemplo:

```text
/mnt/remoto
```

### Mounting point

É o nó de entrada no espaço de nomes estrangeiro, normalmente sua raiz.

Exemplo:

```text
raiz do sistema remoto
```

---

## Informações necessárias para montar um espaço remoto

Em sistema distribuído, para montar um espaço de nomes estrangeiro, geralmente é necessário saber:

1. o nome do protocolo de acesso;
2. o nome do servidor;
3. o nome do ponto de montagem no espaço estrangeiro.

Exemplo conceitual:

```text
protocolo://servidor/caminho
```

Cada parte também precisa ser resolvida.

```text
protocolo -> implementação do protocolo
servidor  -> endereço do servidor
caminho   -> nó dentro do espaço remoto
```

---

# 6.3.2 — Name resolution

## Definição

**Resolução de nomes** é o processo de procurar o objeto ou informação associada a um nome.

Dado um caminho:

```text
N:[label1, label2, ..., labeln]
```

a resolução funciona assim:

1. Começa no nó `N`.
2. Procura `label1` na tabela do diretório de `N`.
3. Obtém o identificador do próximo nó.
4. Nesse próximo nó, procura `label2`.
5. Repete até `labeln`.
6. Retorna o conteúdo do nó final.

---

## Exemplo

Nome:

```text
/home/rian/prova.md
```

Resolução:

```text
raiz "/"        procura "home"
nó /home       procura "rian"
nó /home/rian  procura "prova.md"
nó final       retorna conteúdo/endereço/metadados
```

---

## Closure mechanism

A resolução precisa saber onde começar.

Esse ponto inicial é definido por um **closure mechanism**.

Definição curta:

```text
Closure mechanism = mecanismo que determina o nó inicial da resolução.
```

Exemplo:

```text
/home/rian/prova.md
```

O sistema sabe que nomes começando com `/` começam na raiz do sistema de arquivos.

Outro exemplo:

```text
HOME
```

O sistema sabe que deve resolver `HOME` consultando a tabela de variáveis de ambiente do usuário atual.

---

## Por que closure mechanism importa?

Porque a raiz também é um nome/entidade.

Para resolver qualquer nome, o sistema já precisa ter algum ponto inicial conhecido.

Sem isso, ocorre regressão infinita:

```text
Para resolver nome A, preciso saber onde começa.
Mas o ponto inicial também precisaria ser resolvido.
```

O closure mechanism quebra essa regressão.

---

# Resolução iterativa

Na resolução iterativa, o cliente ou resolvedor consulta um servidor por vez.

Cada servidor responde:

```text
Não sei a resposta final, mas consulte este próximo servidor.
```

Modelo:

```text
cliente -> servidor raiz
cliente -> servidor intermediário
cliente -> servidor final
cliente recebe resposta final
```

O cliente controla o processo.

---

## Vantagens da resolução iterativa

- menor carga em cada servidor;
- servidores intermediários não precisam resolver tudo;
- bom para camadas globais;
- mais simples para servidores de alto nível.

---

## Desvantagens da resolução iterativa

- mais trabalho para o cliente/resolvedor;
- mais mensagens entre cliente e diferentes servidores;
- o cliente precisa lidar com a sequência de consultas.

---

# Resolução recursiva

Na resolução recursiva, o cliente consulta um servidor e espera a resposta final.

O servidor consultado fica responsável por resolver o restante, consultando outros servidores se necessário.

Modelo:

```text
cliente -> servidor raiz
             servidor raiz -> servidor intermediário
                                  servidor intermediário -> servidor final
                                  servidor intermediário <- resposta
             servidor raiz <- resposta
cliente <- resposta final
```

O cliente faz uma consulta.  
Os servidores fazem o restante.

---

## Vantagens da resolução recursiva

- simplifica o cliente;
- pode reduzir custos de comunicação em alguns cenários;
- permite caching mais efetivo nos servidores;
- servidores aprendem endereços de servidores de níveis inferiores.

---

## Desvantagens da resolução recursiva

- aumenta a carga dos servidores;
- servidores precisam manter mais estado;
- servidores de alto nível podem virar gargalo;
- geralmente é inadequada para a camada global em larga escala.

---

# Iterativa vs recursiva

| Critério | Iterativa | Recursiva |
|---|---|---|
| Quem controla a resolução | Cliente/resolvedor | Servidor consultado |
| Carga no servidor | Menor | Maior |
| Carga no cliente | Maior | Menor |
| Caching nos servidores intermediários | Menos efetivo | Mais efetivo |
| Uso típico | Camadas globais | Camadas administrativas ou locais |
| Risco | Mais consultas no cliente | Sobrecarga no servidor |

---

# Caching

Caching armazena resultados de resoluções anteriores.

Exemplo:

```text
www.exemplo.com -> 192.0.2.10
```

Se o mesmo nome for consultado novamente, o resultado pode vir do cache.

Vantagem:

```text
menos latência
menos carga nos servidores de nomes
melhor escalabilidade
```

Problema:

```text
o cache pode ficar desatualizado
```

Esse problema é mais grave em partes do espaço de nomes que mudam frequentemente.

---

# 6.3.3 — Implementação de um espaço de nomes

## Serviço de nomes

Um **serviço de nomes** é implementado por **servidores de nomes**.

Em sistemas pequenos, pode haver um único servidor de nomes.

Em sistemas grandes, o espaço de nomes precisa ser distribuído entre vários servidores.

Motivo:

```text
escala, desempenho, disponibilidade e distribuição geográfica.
```

---

## Distribuição do espaço de nomes

Espaços de nomes grandes são normalmente hierárquicos.

Para implementá-los de forma escalável, o livro divide o espaço em três camadas lógicas:

1. camada global;
2. camada administrativa;
3. camada gerencial.

---

# Camada global

A camada global contém os nós de mais alto nível.

Exemplos:

```text
raiz
nós próximos da raiz
grandes organizações
grupos de organizações
domínios de topo
```

Características:

- poucos nós;
- escala geográfica mundial;
- mudanças raras;
- alta estabilidade;
- altíssima necessidade de disponibilidade;
- pode tolerar resposta em segundos;
- usa muitas réplicas;
- caching é muito efetivo;
- atualizações podem propagar de forma preguiçosa.

Ponto crítico:

```text
Se um servidor da camada global falha, uma grande parte do espaço de nomes pode ficar inalcançável.
```

Por isso, servidores globais precisam de alta disponibilidade e replicação.

---

# Camada administrativa

A camada administrativa contém nós gerenciados por uma organização ou unidade administrativa.

Exemplos:

```text
departamentos
hosts de uma organização
usuários de uma organização
serviços internos
```

Características:

- muitos nós;
- escopo organizacional;
- mudanças mais frequentes que na camada global;
- ainda relativamente estável;
- disponibilidade importante para a organização;
- resposta esperada em milissegundos;
- usa poucas réplicas ou nenhuma;
- caching ainda é útil;
- atualizações devem ser mais rápidas que na camada global.

Exemplo:

```text
Criar uma conta de usuário não pode levar horas para aparecer.
```

---

# Camada gerencial

A camada gerencial contém nós que mudam frequentemente.

Exemplos:

```text
hosts locais
arquivos compartilhados
bibliotecas
binários
diretórios de usuários
arquivos de usuários
```

Características:

- número vasto de nós;
- escopo local/departamental;
- mudanças frequentes;
- mantida por administradores e usuários finais;
- disponibilidade menos exigente;
- desempenho muito importante;
- respostas devem ser imediatas;
- normalmente sem réplicas;
- caching é menos efetivo, porque os dados mudam muito.

---

## Tabela comparativa das camadas

| Critério | Global | Administrativa | Gerencial |
|---|---|---|---|
| Escala geográfica | Mundial | Organização | Departamento/local |
| Número de nós | Poucos | Muitos | Vastos |
| Estabilidade | Muito alta | Alta/média | Baixa |
| Responsividade | Segundos | Milissegundos | Imediata |
| Propagação de updates | Preguiçosa | Rápida/imediata | Imediata |
| Réplicas | Muitas | Nenhuma ou poucas | Normalmente nenhuma |
| Cache no cliente | Sim | Sim | Às vezes |
| Exemplo | raiz/domínio alto | departamento/hosts | arquivos de usuários |

---

# Zonas

Um espaço de nomes pode ser dividido em partes não sobrepostas chamadas **zonas**.

Uma zona é uma parte do espaço de nomes implementada por um servidor de nomes separado.

Exemplo conceitual:

```text
raiz
├── org
├── com
└── br
    └── ufg
        └── inf
```

Possíveis zonas:

```text
zona da raiz
zona de br
zona de ufg.br
zona de inf.ufg.br
```

Cada zona pode ser administrada por uma entidade diferente.

---

## Por que zonas são importantes?

Zonas permitem delegação.

Exemplo:

```text
A raiz não precisa conhecer todos os hosts da UFG.
Ela só precisa saber quem resolve "br" ou "ufg.br".
```

Cada parte do espaço de nomes fica sob responsabilidade de servidores específicos.

Isso melhora:

- escalabilidade;
- administração;
- autonomia;
- distribuição da carga;
- disponibilidade.

---

# Replicação em servidores de nomes

Servidores de nomes podem ser replicados para aumentar disponibilidade.

Isso é especialmente importante na camada global.

Vantagens:

- tolerância a falhas;
- maior capacidade de atendimento;
- menor latência se réplicas estiverem geograficamente distribuídas.

Problema:

```text
réplicas precisam se manter consistentes
```

Na camada global, isso é mais fácil porque atualizações são raras.

Na camada gerencial, replicação é mais difícil porque mudanças são frequentes.

---

# Caching por camada

## Global

Caching é muito útil.

Motivo:

```text
nomes mudam pouco
```

## Administrativa

Caching também é útil, mas updates precisam aparecer mais rápido.

## Gerencial

Caching é menos útil.

Motivo:

```text
dados mudam com frequência
```

---

# O que provavelmente cai na prova

## 1. Diferença entre nome, identificador e endereço

Resposta curta:

```text
Nome referencia entidade.
Identificador referencia entidade de forma única, estável e não reutilizada.
Endereço referencia ponto de acesso.
```

---

## 2. Por que endereço não é bom identificador?

Porque endereço pode mudar ou ser reassociado a outra entidade.

Exemplo:

```text
IP muda com DHCP.
Porta muda quando processo reinicia.
Telefone pode ser reaproveitado por outra pessoa.
```

---

## 3. O que é nome independente de localização?

É um nome que não codifica o ponto de acesso da entidade.

Vantagem:

```text
a entidade pode migrar ou ser replicada sem mudar de nome.
```

---

## 4. O que é espaço de nomes?

É uma estrutura que organiza nomes.

No modelo do livro:

```text
grafo direcionado rotulado com nós folha e nós diretório.
```

---

## 5. O que é um nó folha?

É um nó que representa uma entidade e não possui arestas de saída.

Pode armazenar endereço, estado ou metadados da entidade.

---

## 6. O que é um nó diretório?

É um nó com arestas de saída rotuladas.

Funciona como uma tabela:

```text
rótulo -> identificador de nó
```

---

## 7. Como funciona a resolução de nomes?

A resolução percorre o espaço de nomes passo a passo.

Exemplo:

```text
/home/rian/prova.md
```

Resolve:

```text
/ -> home -> rian -> prova.md
```

---

## 8. O que é closure mechanism?

É o mecanismo que determina onde a resolução começa.

Exemplo:

```text
Nomes começando com "/" começam na raiz.
Variável HOME é resolvida na tabela de ambiente do usuário.
```

---

## 9. Diferença entre hard link e symbolic link

Hard link:

```text
dois nomes apontam diretamente para o mesmo nó
```

Symbolic link:

```text
um nó armazena outro caminho, e a resolução continua nesse caminho
```

---

## 10. O que é mounting?

É anexar um espaço de nomes estrangeiro a um ponto em outro espaço de nomes.

Exemplo:

```text
/mnt/remoto
```

passa a dar acesso a um sistema de arquivos remoto.

---

## 11. O que é necessário para montar um espaço remoto?

Geralmente:

1. protocolo de acesso;
2. servidor;
3. ponto de montagem no espaço estrangeiro.

---

## 12. Diferença entre resolução iterativa e recursiva

Iterativa:

```text
cliente consulta cada servidor sucessivamente
```

Recursiva:

```text
cliente consulta um servidor, e esse servidor resolve o restante
```

---

## 13. Por que resolução recursiva pode melhorar caching?

Porque servidores intermediários aprendem resultados de resoluções para níveis inferiores e podem reutilizá-los depois.

---

## 14. Por que servidores globais preferem resolução iterativa?

Porque resolução recursiva aumenta a carga dos servidores.

Servidores globais já são críticos para grande parte do espaço de nomes.

---

## 15. Quais são as três camadas de implementação de um espaço de nomes?

```text
global
administrativa
gerencial
```

---

## 16. Compare as três camadas

Global:

```text
poucos nós, alta estabilidade, muitas réplicas, caching forte
```

Administrativa:

```text
organização, mudanças moderadas, resposta em milissegundos
```

Gerencial:

```text
muitos nós, mudanças frequentes, resposta imediata, pouco cache
```

---

## 17. O que é uma zona?

Uma zona é uma parte não sobreposta do espaço de nomes implementada por um servidor de nomes separado.

Serve para distribuir e delegar a administração do espaço de nomes.

---

# Respostas curtas para decorar

## Nome

```text
Cadeia de bits ou caracteres usada para referir uma entidade.
```

## Entidade

```text
Qualquer recurso ou objeto nomeável no sistema distribuído.
```

## Ponto de acesso

```text
Local ou mecanismo pelo qual uma entidade pode ser acessada.
```

## Endereço

```text
Nome de um ponto de acesso.
```

## Identificador

```text
Nome único, não ambíguo e não reutilizado para uma entidade.
```

## Nome independente de localização

```text
Nome que não depende do endereço atual da entidade.
```

## Espaço de nomes

```text
Grafo direcionado rotulado usado para organizar nomes.
```

## Nó folha

```text
Nó que representa uma entidade e não possui arestas de saída.
```

## Nó diretório

```text
Nó que mapeia rótulos para outros nós.
```

## Resolução de nomes

```text
Processo de percorrer o espaço de nomes para encontrar o nó associado a um nome.
```

## Closure mechanism

```text
Mecanismo que define onde a resolução começa.
```

## Hard link

```text
Múltiplos caminhos apontam diretamente para o mesmo nó.
```

## Symbolic link

```text
Um nó armazena um caminho para outro nome.
```

## Mounting

```text
Integração de um espaço de nomes estrangeiro em um ponto de outro espaço de nomes.
```

## Resolução iterativa

```text
O cliente consulta os servidores um por um.
```

## Resolução recursiva

```text
O servidor consultado resolve o restante do caminho.
```

## Zona

```text
Parte do espaço de nomes gerenciada por um servidor de nomes específico.
```

---

# Prioridade para estudar

## Prioridade alta

- Nome vs identificador vs endereço.
- Ponto de acesso.
- Por que endereço não deve ser usado como identificador.
- Nome independente de localização.
- Espaço de nomes como grafo.
- Nó folha e nó diretório.
- Caminho absoluto e relativo.
- Resolução de nomes passo a passo.
- Closure mechanism.
- Hard link vs symbolic link.
- Mounting.
- Resolução iterativa vs recursiva.
- Camadas global, administrativa e gerencial.
- Zonas.

## Prioridade média

- Caching de nomes.
- Replicação de servidores de nomes.
- Diferenças de requisitos entre as camadas.
- Vantagens e desvantagens de recursão.

## Prioridade baixa ou fora do escopo

- DHTs.
- Broadcasting e forwarding pointers.
- DNS em detalhe.
- NFS em detalhe.
- LDAP.
- Attribute-based naming.
- Named-data networking.
