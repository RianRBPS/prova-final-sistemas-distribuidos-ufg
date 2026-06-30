# Resumo — Capítulo 9: Segurança

**Livro:** *Distributed Systems*, Maarten van Steen e Andrew S. Tanenbaum, 4ª edição.  
**Escopo da prova:** Cap. 9.1, Cap. 9.2 até 9.2.3, e Cap. 9.3 até o início da p. 581.

**Entra na prova:**

- 9.1 — Introduction to security
- 9.1.1 — Security threats, policies, and mechanisms
- 9.1.2 — Design issues
- 9.2 — Cryptography
- 9.2.1 — Basics
- 9.2.2 — Symmetric and asymmetric cryptosystems
- 9.2.3 — Hash functions
- 9.3 — Authentication
- 9.3.1 — Introduction to authentication
- 9.3.2 — Authentication protocols, até antes de Kerberos

**Fora do escopo:**

- 9.2.4 — Key management
- Kerberos
- TLS/SSL
- Trust
- Authorization
- Monitoring
- Firewalls
- Intrusion detection
- Delegation
- Attribute-based access control

---

# Visão geral do capítulo

Segurança em sistemas distribuídos trata de proteger recursos, dados, serviços e comunicação contra uso indevido.

Em sistemas distribuídos, segurança é mais difícil porque:

- há muitos processos;
- há comunicação por rede;
- há múltiplas máquinas;
- há múltiplos administradores;
- partes do sistema podem estar em organizações diferentes;
- mensagens podem ser interceptadas, modificadas ou inseridas;
- usuários e serviços precisam se autenticar remotamente;
- falhas em uma parte podem comprometer o sistema inteiro.

Objetivo prático:

```text
garantir que apenas entidades autorizadas acessem recursos,
e que a comunicação preserve confidencialidade, integridade e autenticidade.
```

---

# Conceitos fundamentais

## Confidentiality — confidencialidade

Garante que informação só seja revelada a entidades autorizadas.

Exemplo:

```text
Apenas Alice e Bob conseguem ler uma mensagem criptografada.
```

Ataque típico:

```text
eavesdropping
interceptação
leitura não autorizada
```

---

## Integrity — integridade

Garante que dados e mensagens não sejam alterados sem autorização.

Exemplo:

```text
Mensagem enviada: transferir R$100
Mensagem recebida: transferir R$100
```

Violação:

```text
Atacante altera para transferir R$1000.
```

---

## Availability — disponibilidade

Garante que recursos e serviços estejam acessíveis quando necessários.

Exemplo:

```text
Servidor continua respondendo a usuários legítimos.
```

Ataque típico:

```text
DoS
DDoS
sobrecarga intencional
```

---

## Authentication — autenticação

Verifica a identidade alegada por uma entidade.

Pergunta:

```text
Você é realmente quem diz ser?
```

Exemplo:

```text
Alice prova para Bob que é Alice.
```

---

## Authorization — autorização

Verifica se uma entidade autenticada tem permissão para executar uma ação.

Pergunta:

```text
Você tem permissão para fazer isso?
```

Exemplo:

```text
Alice está autenticada, mas pode ler este arquivo?
```

Observação:

```text
Autenticação vem antes de autorização.
```

---

# 9.1 — Introdução à segurança

Segurança não é um módulo isolado.

Ela atravessa:

- comunicação;
- processos;
- nomes;
- replicação;
- tolerância a falhas;
- autenticação;
- autorização;
- auditoria;
- armazenamento;
- aplicações.

Um sistema distribuído inseguro não é confiável.

---

# 9.1.1 — Ameaças, políticas e mecanismos

## Ativo

Um ativo é algo que precisa ser protegido.

Exemplos:

- arquivo;
- banco de dados;
- senha;
- chave criptográfica;
- processo;
- serviço;
- mensagem;
- registro médico;
- saldo bancário;
- identidade de usuário.

---

## Ameaça

Uma ameaça é uma possível violação de segurança.

Exemplos:

- interceptar mensagens;
- modificar dados;
- apagar arquivos;
- roubar chave privada;
- fingir ser outro usuário;
- executar operação sem permissão;
- derrubar um serviço.

---

## Ataque

Um ataque é uma tentativa concreta de explorar uma ameaça.

Exemplo:

```text
Threat: alguém pode descobrir senhas.
Attack: enviar página falsa de login e coletar credenciais.
```

---

## Política de segurança

Uma política de segurança define o que é permitido e o que é proibido.

Ela especifica:

```text
quem pode acessar o quê
em quais condições
com quais operações
```

Exemplo:

```text
Apenas médicos podem ler prontuários de pacientes sob seus cuidados.
```

Outro exemplo:

```text
Usuários comuns podem ler seus próprios arquivos,
mas não podem ler arquivos de outros usuários.
```

A política diz o objetivo.

---

## Mecanismo de segurança

Um mecanismo é a implementação usada para aplicar a política.

Exemplos:

- criptografia;
- autenticação;
- autorização;
- listas de controle de acesso;
- assinaturas digitais;
- logs;
- auditoria;
- firewalls;
- protocolos seguros.

A política diz:

```text
o que deve acontecer
```

O mecanismo diz:

```text
como isso será imposto
```

---

# Mecanismos principais

O livro destaca quatro mecanismos importantes.

## 1. Encryption — criptografia

Transforma plaintext em ciphertext.

Objetivo:

```text
proteger confidencialidade
```

Também pode ajudar em integridade e autenticação, dependendo do protocolo.

---

## 2. Authentication — autenticação

Verifica identidade.

Exemplo:

```text
senha
chave privada
certificado
biometria
```

---

## 3. Authorization — autorização

Controla permissões.

Exemplo:

```text
Alice pode ler arquivo X, mas não pode modificá-lo.
```

---

## 4. Auditing — auditoria

Registra ações para posterior análise.

Exemplo:

```text
Quem acessou qual arquivo?
Quando?
De qual IP?
Qual operação executou?
```

Auditoria não impede diretamente o ataque, mas ajuda a:

- detectar violações;
- reconstruir eventos;
- responsabilizar usuários;
- melhorar mecanismos de defesa.

---

# Ameaças em comunicação

Considere:

```text
Sender S -> Receiver R
```

Um intruso pode agir de três formas principais.

## 1. Interceptação

O atacante lê a mensagem.

```text
S -> mensagem -> R
      atacante copia
```

Afeta:

```text
confidencialidade
```

---

## 2. Modificação

O atacante altera a mensagem.

```text
S envia: pagar 100
R recebe: pagar 1000
```

Afeta:

```text
integridade
```

---

## 3. Inserção

O atacante injeta mensagens falsas.

```text
Atacante envia mensagem fingindo ser S.
```

Afeta:

```text
autenticidade
integridade
```

---

# 9.1.2 — Questões de projeto

## Princípios de segurança

O livro destaca princípios clássicos de projeto de sistemas seguros.

---

# Fail-safe defaults

Padrões devem ser seguros por padrão.

Regra prática:

```text
Se não há permissão explícita, negar acesso.
```

Exemplo ruim:

```text
usuário admin/admin como senha padrão
```

Exemplo melhor:

```text
cada dispositivo vem com senha única forte
```

Interpretação:

```text
Acesso deve ser baseado em permissões, não em exclusões.
```

---

# Open design

A segurança não deve depender de segredo no design.

Regra prática:

```text
O mecanismo pode ser público.
A segurança deve depender das chaves, não de esconder o algoritmo.
```

Isso combate:

```text
security by obscurity
```

Exemplo:

```text
Algoritmo criptográfico público, chave secreta.
```

---

# Separation of privilege

Operações críticas não devem depender de uma única condição ou entidade.

Regra prática:

```text
Exigir múltiplas permissões independentes para ações sensíveis.
```

Exemplo:

```text
Duas pessoas precisam autorizar uma operação crítica.
```

Outro exemplo:

```text
Arquivo muito sensível criptografado com duas chaves diferentes.
```

---

# Least privilege

Cada entidade deve receber apenas os privilégios necessários.

Regra prática:

```text
Dar o mínimo acesso suficiente para executar a tarefa.
```

Exemplo:

```text
Serviço de backup pode ler arquivos,
mas não pode alterar permissões de usuários.
```

Vantagem:

```text
Se o serviço for comprometido, o dano é limitado.
```

---

# Least common mechanism

Evitar mecanismos compartilhados desnecessários entre componentes.

Regra prática:

```text
Quanto menos componentes compartilham um mecanismo crítico, menor o risco de interferência e vazamento.
```

Exemplo:

```text
Evitar que aplicações diferentes compartilhem o mesmo arquivo temporário, chave ou canal sem necessidade.
```

---

# Onde implementar segurança?

Há várias camadas possíveis.

## 1. Nível de rede

Exemplo:

```text
VPN
IPsec
```

Vantagem:

```text
protege tráfego entre redes ou hosts
```

Limitação:

```text
pode não proteger contra ataques dentro dos endpoints
```

---

## 2. Nível de sistema operacional

Exemplos:

```text
permissões de arquivos
usuários e grupos
SSH
controle de processos
```

Vantagem:

```text
reutiliza mecanismos do sistema operacional
```

---

## 3. Middleware

Exemplos:

```text
secure RPC
servidores Web seguros
bancos de dados com controle de acesso
```

Vantagem:

```text
aplicações usam serviços de segurança já disponíveis
```

---

## 4. Aplicação / end-to-end

A própria aplicação implementa a segurança.

Exemplos:

```text
Signal
WhatsApp
Telegram
aplicação criptografa antes de enviar para a nuvem
```

Vantagem:

```text
proteção fim-a-fim
não depende tanto da infraestrutura intermediária
```

Desvantagem:

```text
a aplicação precisa implementar e manter os mecanismos corretamente
```

---

# Trusted Computing Base — TCB

A **TCB** é o conjunto de todos os componentes que precisam ser confiáveis para que a política de segurança seja aplicada.

Inclui:

- hardware;
- firmware;
- sistema operacional;
- middleware;
- bibliotecas;
- aplicações privilegiadas;
- administradores;
- scripts;
- bancos de dados usados para controle de acesso.

Definição curta:

```text
TCB = tudo que precisa estar correto e confiável para a segurança funcionar.
```

Regra prática:

```text
Quanto menor a TCB, melhor.
```

Por quê?

```text
Menos componentes para auditar.
Menos componentes que podem ser comprometidos.
Menor superfície de ataque.
```

---

# Honest-but-curious server

Um servidor **honest-but-curious** segue o protocolo, mas tenta aprender o máximo possível observando os dados.

Exemplo:

```text
Servidor de nuvem armazena arquivos corretamente,
mas tenta inferir conteúdo ou padrões de acesso.
```

Esse modelo é comum em privacidade e criptografia aplicada.

---

# Privacidade

Privacidade e confidencialidade são relacionadas, mas não são idênticas.

## Confidencialidade

```text
Informação não é revelada sem autorização.
```

## Privacidade

```text
Controle apropriado do fluxo de informação pessoal.
```

Exemplo:

```text
Mesmo se um app só compartilha dados com parceiros autorizados,
ainda pode violar privacidade se o fluxo for inadequado ou impossível de revogar.
```

Ideia importante:

```text
Privacidade não é apenas esconder dados.
É controlar quem recebe, quando recebe, por qual motivo e por quanto tempo.
```

---

# Segurança e GDPR

O livro discute que sistemas distribuídos modernos precisam considerar requisitos como:

- coletar dados apenas para propósitos explícitos;
- não armazenar indefinidamente;
- permitir acesso do usuário aos próprios dados;
- permitir apagamento;
- restringir uso para fins contestados;
- auditar operações;
- controlar acesso;
- criptografar dados.

Mensagem importante:

```text
Adicionar privacidade depois do sistema pronto pode ser muito difícil.
Privacidade deve ser considerada no projeto.
```

---

# 9.2 — Criptografia

Criptografia é o mecanismo central para proteger comunicação e dados.

Ela fornece base para:

- confidencialidade;
- integridade;
- autenticação;
- assinaturas digitais;
- canais seguros;
- armazenamento seguro;
- protocolos de autenticação.

---

# 9.2.1 — Básico

## Plaintext

Mensagem original legível.

```text
P = plaintext
```

Exemplo:

```text
transferir R$100 para Bob
```

---

## Ciphertext

Mensagem criptografada.

```text
C = ciphertext
```

Exemplo:

```text
8fA91xQ...
```

---

## Encryption

Transforma plaintext em ciphertext.

```text
C = EK(P)
```

Onde:

- `E` = encryption;
- `K` = chave;
- `P` = plaintext;
- `C` = ciphertext.

---

## Decryption

Transforma ciphertext de volta em plaintext.

```text
P = DK(C)
```

Onde:

- `D` = decryption;
- `K` = chave.

---

# Três ataques contra comunicação

## 1. Eavesdropping / interceptação

Atacante lê a mensagem durante transmissão.

Criptografia ajuda porque o atacante vê apenas ciphertext.

---

## 2. Modificação

Atacante altera a mensagem.

Criptografia e mecanismos de integridade ajudam a detectar alteração.

---

## 3. Inserção

Atacante injeta mensagens falsas.

Autenticação e integridade ajudam a detectar que a mensagem não veio da entidade legítima.

---

# 9.2.2 — Criptossistemas simétricos e assimétricos

## Criptografia simétrica

A mesma chave é usada para criptografar e descriptografar.

```text
P = DK(EK(P))
DK = EK
```

Também chamada:

```text
secret-key cryptography
shared-key cryptography
```

Notação:

```text
KA,B = chave secreta compartilhada entre A e B
```

Exemplo:

```text
Alice e Bob compartilham KA,B.
Alice envia C = KA,B(m).
Bob usa KA,B para recuperar m.
```

---

## Vantagens da criptografia simétrica

- rápida;
- eficiente;
- boa para grandes volumes de dados;
- adequada para session keys.

---

## Desvantagens da criptografia simétrica

- exige chave compartilhada prévia;
- distribuição da chave é difícil;
- se há muitos pares de usuários, o número de chaves cresce muito.

Para `N` entidades, chaves par-a-par podem exigir:

```text
N(N - 1) / 2 chaves
```

---

# Criptografia assimétrica

Usa par de chaves:

```text
PKA = chave pública de Alice
SKA = chave privada/secreta de Alice
```

Uma chave é pública.  
A outra deve permanecer secreta.

Também chamada:

```text
public-key cryptography
```

---

## Uso 1 — Confidencialidade

Se Alice quer mandar mensagem secreta para Bob, usa a chave pública de Bob.

```text
C = PKB(m)
```

Apenas Bob, com sua chave privada, pode descriptografar:

```text
m = SKB(C)
```

Interpretação:

```text
Criptografa com a chave pública do destinatário.
```

---

## Uso 2 — Autenticação / assinatura

Se Alice quer provar que a mensagem veio dela, usa sua chave privada.

```text
sig = SKA(m)
```

Bob verifica com a chave pública de Alice:

```text
m = PKA(sig)
```

Interpretação:

```text
Assina com a chave privada do remetente.
Verifica com a chave pública do remetente.
```

---

## Cuidado essencial

Criptografia assimétrica depende de saber se a chave pública realmente pertence à entidade alegada.

Exemplo:

```text
Alice acha que está usando PKB.
Mas atacante entregou PKC fingindo ser PKB.
```

Nesse caso, Alice pode estar criptografando para o atacante.

Por isso, distribuição e validação de chaves públicas são críticas.

Detalhes de certificados e gerenciamento de chaves estão fora do escopo da prova.

---

# Simétrica vs assimétrica

| Critério | Simétrica | Assimétrica |
|---|---|---|
| Chaves | Uma chave compartilhada | Par público/privado |
| Desempenho | Mais rápida | Mais lenta |
| Distribuição de chave | Difícil | Mais flexível |
| Confidencialidade | Usa chave compartilhada | Usa chave pública do destinatário |
| Assinatura | Não natural | Usa chave privada do remetente |
| Uso típico | Dados de sessão | Autenticação, troca de chave, assinatura |

---

# Homomorphic encryption

Criptografia homomórfica permite executar certas operações sobre ciphertexts.

Ideia:

```text
EK(x) ⋆ EK(y) = EK(x ⋆ y)
```

Ou seja:

```text
operação no dado criptografado gera resultado criptografado correspondente à operação no dado original
```

Exemplo:

```text
Servidor soma valores criptografados sem ver os valores originais.
```

---

## Full homomorphic encryption — FHE

Permite computações gerais, como adição e multiplicação suficientes para expressar operações complexas.

Problema:

```text
muito caro computacionalmente
```

---

## Partial homomorphic encryption — PHE

Permite apenas certas operações, como só adição ou só multiplicação.

Vantagem:

```text
mais eficiente
```

Desvantagem:

```text
serve para aplicações mais específicas
```

---

# 9.2.3 — Hash functions

Uma função hash recebe entrada de tamanho arbitrário e produz saída de tamanho fixo.

```text
h = H(m)
```

Onde:

- `m` = mensagem;
- `H` = função hash;
- `h` = hash / digest.

Exemplo:

```text
H("senha123") = valor fixo de N bits
```

---

## Características

Uma função hash criptográfica deve ter propriedades fortes.

---

# 1. One-way function

Dado:

```text
h = H(m)
```

deve ser computacionalmente inviável descobrir `m` a partir de `h`.

Em forma curta:

```text
fácil calcular H(m)
difícil inverter H
```

---

# 2. Weak collision resistance

Dado um `m` específico, deve ser inviável encontrar outro `m'` diferente com o mesmo hash.

```text
m' != m
H(m') = H(m)
```

---

# 3. Strong collision resistance

Deve ser inviável encontrar qualquer par `m` e `m'` diferentes com o mesmo hash.

```text
m != m'
H(m) = H(m')
```

---

# Uso em armazenamento de senhas

Em vez de armazenar a senha diretamente:

```text
senha = pw
```

armazenar:

```text
H(pw)
```

Quando Alice tenta login com `pw'`, o sistema verifica:

```text
H(pw') == H(pw)
```

Observação prática:

```text
Em sistemas reais, usa-se salt e funções próprias para senha.
```

Mas o conceito do capítulo é:

```text
armazenar hash é melhor que armazenar senha em claro.
```

---

# Message digest

Um message digest é o hash de uma mensagem.

```text
digest = H(m)
```

Se a mensagem mudar, o digest muda.

Uso:

```text
detectar alteração de mensagem
assinar mensagens de forma eficiente
```

---

# Assinatura digital

Assinatura digital resolve dois problemas:

1. Integridade: detectar alteração da mensagem.
2. Não repúdio: impedir que o remetente negue depois ter assinado.

Exemplo:

```text
Alice confirma por mensagem que comprará um item por R$500.
Bob não deve conseguir alterar para R$5000.
Alice não deve conseguir negar que enviou a confirmação.
```

---

## Assinar a mensagem inteira

Alice poderia assinar criptografando a mensagem com sua chave privada:

```text
sig = SKA(m)
```

Bob verifica com a chave pública de Alice:

```text
m = PKA(sig)
```

Problema:

```text
caro para mensagens grandes
```

---

## Assinar o hash da mensagem

Forma mais eficiente:

```text
sig = SKA(H(m))
```

Alice envia:

```text
[m, sig]
```

Bob calcula:

```text
h' = H(m)
```

e verifica:

```text
h' == PKA(sig)
```

Se os valores coincidem:

```text
a mensagem não foi alterada
a assinatura corresponde à chave privada de Alice
```

---

## Confidencialidade + assinatura

Assinatura sozinha não esconde o conteúdo.

Se Alice envia:

```text
[m, sig]
```

todos podem ler `m`.

Se também quer confidencialidade, deve criptografar para Bob.

Exemplo conceitual:

```text
C = PKB([m, sig])
```

Bob descriptografa com sua chave privada e verifica assinatura com a chave pública de Alice.

---

# Notação importante

```text
KA,B     = chave secreta compartilhada por A e B
PKA      = chave pública de A
SKA      = chave privada/secreta de A
EK(P)    = criptografia do plaintext P com chave EK
DK(C)    = descriptografia do ciphertext C com chave DK
H(m)     = hash de m
[m]A     = mensagem m assinada por A
```

---

# 9.3 — Autenticação

Autenticação verifica a identidade alegada de uma entidade.

Pode autenticar:

- pessoa;
- processo;
- servidor;
- cliente;
- dispositivo;
- componente de software.

Pergunta central:

```text
Como Alice prova para Bob que ela é Alice?
```

---

# 9.3.1 — Introdução à autenticação

## Fatores de autenticação

Há três categorias clássicas.

---

## 1. Algo que você sabe

Exemplos:

- senha;
- PIN;
- frase secreta.

Problema:

```text
pode ser fraco, reutilizado, vazado ou roubado por phishing.
```

---

## 2. Algo que você possui

Exemplos:

- celular;
- smart card;
- token físico;
- chave de segurança;
- aplicativo autenticador.

---

## 3. Algo que você é

Exemplos:

- impressão digital;
- reconhecimento facial;
- íris;
- voz;
- biometria comportamental.

---

# Single-factor authentication

Usa apenas um fator.

Exemplo:

```text
senha
```

Problema:

```text
se a senha vaza, o atacante entra.
```

---

# Multi-factor authentication — MFA

Usa mais de um fator.

Exemplo:

```text
senha + código no celular
senha + biometria
senha + chave física
```

Vantagem:

```text
comprometimento de um fator não basta para autenticar.
```

---

# Continuous authentication

Autenticação tradicional ocorre uma vez, no início da sessão.

Problema:

```text
Alice autentica.
Depois Bob usa o computador dela.
O serviço ainda acha que está falando com Alice.
```

Continuous authentication tenta verificar a identidade durante a sessão.

Exemplos:

- presença de dispositivo pessoal próximo;
- biometria contínua;
- padrão de digitação;
- localização conhecida via sinais Wi-Fi;
- segunda autenticação se o contexto muda.

---

# Autenticação e integridade

Autenticação e integridade devem andar juntas.

## Autenticação sem integridade

Bob sabe que Alice enviou uma mensagem, mas não sabe se ela foi alterada.

Problema:

```text
Mensagem pode ter sido modificada no caminho.
```

## Integridade sem autenticação

Bob sabe que a mensagem não foi alterada, mas não sabe quem a enviou.

Problema:

```text
Mensagem íntegra pode ter vindo de um atacante.
```

Conclusão:

```text
Autenticação precisa de integridade.
Integridade precisa de autenticação para ser útil.
```

---

# Session key

Após autenticação, as partes normalmente usam uma **session key**.

Definição:

```text
chave secreta temporária usada durante uma sessão de comunicação
```

Quando a sessão termina:

```text
a session key é descartada
```

Uso:

- criptografar mensagens;
- garantir integridade;
- reduzir uso de chaves de longa duração;
- limitar dano se uma chave de sessão for comprometida.

---

# 9.3.2 — Protocolos de autenticação

## Challenge-response

Challenge-response prova conhecimento de uma chave sem enviar a chave pela rede.

Ideia:

```text
Bob envia desafio aleatório.
Alice responde aplicando a chave secreta ao desafio.
```

Se a resposta estiver correta, Bob conclui:

```text
Alice conhece a chave secreta.
```

---

# Autenticação com chave secreta compartilhada

Suponha que Alice e Bob compartilham:

```text
KA,B
```

Bob quer autenticar Alice.

## Protocolo simples

1. Alice envia sua identidade para Bob.

```text
A -> B: A
```

2. Bob envia um desafio aleatório `RB`.

```text
B -> A: RB
```

3. Alice responde cifrando o desafio com a chave compartilhada.

```text
A -> B: KA,B(RB)
```

4. Bob verifica se a resposta corresponde ao desafio.

Se sim:

```text
Bob autentica Alice.
```

---

## Por que usar desafio aleatório?

Para evitar replay attack.

Sem desafio, um atacante poderia gravar uma resposta antiga e reutilizá-la.

Com desafio novo:

```text
resposta antiga não serve para desafio novo
```

---

# Mutual authentication com chave secreta

No protocolo anterior, Bob autentica Alice.

Mas Alice ainda não autenticou Bob.

Para autenticação mútua:

1. Bob desafia Alice com `RB`.
2. Alice responde `KA,B(RB)`.
3. Alice desafia Bob com `RA`.
4. Bob responde `KA,B(RA)`.

Resultado:

```text
Bob sabe que fala com Alice.
Alice sabe que fala com Bob.
```

---

# Reflection attack

O livro mostra que tentar reduzir o protocolo para menos mensagens pode quebrar a segurança.

Ataque:

```text
Chuck quer se passar por Alice diante de Bob.
Bob envia desafio.
Chuck abre uma segunda sessão com Bob e faz Bob cifrar o próprio desafio.
Chuck usa a resposta obtida para completar a primeira sessão.
```

Problema:

```text
O protocolo usa desafios de forma simétrica e permite refletir o desafio de volta.
```

---

## Defesa contra reflection attack

Usar papéis e desafios distinguíveis.

Exemplos:

```text
desafios diferentes para iniciador e respondedor
Alice usa números ímpares
Bob usa números pares
incluir identidade e contexto na mensagem assinada/cifrada
não reutilizar o mesmo desafio em execuções diferentes
```

Mensagem para prova:

```text
Protocolos de segurança não podem ser "otimizados" ingenuamente.
Reduzir mensagens pode quebrar autenticação.
```

---

# Nonce

Um nonce é um número aleatório usado uma única vez.

Função principal:

```text
associar uma resposta a uma execução específica do protocolo
```

Ajuda contra:

- replay attack;
- respostas antigas;
- mensagens fora de contexto.

Exemplo:

```text
Alice envia RA.
Resposta deve conter RA.
Se contém outro valor, não é resposta à sessão atual.
```

---

# Key Distribution Center — KDC

Problema da chave compartilhada par-a-par:

```text
N entidades exigem N(N - 1) / 2 chaves.
```

Para muitos hosts, isso não escala.

Solução:

```text
KDC
```

Cada entidade compartilha uma chave apenas com o KDC.

Para `N` entidades:

```text
N chaves com o KDC
```

Em vez de:

```text
N(N - 1) / 2 chaves entre pares
```

---

## Ideia do KDC

Alice quer falar com Bob.

1. Alice pede ao KDC uma chave para falar com Bob.
2. KDC gera uma session key `KA,B`.
3. KDC envia `KA,B` para Alice, cifrada com a chave que Alice compartilha com o KDC.
4. KDC também prepara informação para Bob, cifrada com a chave que Bob compartilha com o KDC.
5. Alice usa essa informação para estabelecer canal com Bob.

---

# Ticket

Um ticket é uma mensagem gerada pelo KDC para ser apresentada ao servidor.

Exemplo conceitual:

```text
ticket = KB,KDC(KA,B, A)
```

Apenas Bob consegue abrir o ticket, porque está cifrado com a chave que Bob compartilha com o KDC.

Alice não precisa conhecer a chave de Bob com o KDC.

Ela apenas repassa o ticket.

---

# Needham-Schroeder com KDC

O protocolo Needham-Schroeder usa KDC, session key, ticket e nonces.

Objetivo:

```text
Alice e Bob obtêm uma chave de sessão KA,B com ajuda do KDC.
```

Elementos principais:

- Alice;
- Bob;
- KDC;
- nonce de Alice;
- chave de sessão;
- ticket para Bob.

---

## Por que incluir nonce?

Para evitar replay de respostas antigas.

Exemplo de ataque sem nonce:

```text
Chuck grava uma resposta antiga do KDC.
Depois reenvia para Alice.
Alice pode acreditar que é uma resposta nova.
```

Com nonce:

```text
resposta antiga contém nonce antigo
Alice detecta que não corresponde à sessão atual.
```

---

## Por que incluir a identidade de Bob na resposta?

Para evitar troca maliciosa de destinatário.

Ataque possível sem identidade:

```text
Alice pede chave para falar com Bob.
Chuck altera a mensagem para pedir chave para falar com Chuck.
KDC responde.
Alice pode acreditar que a chave é para Bob.
```

Defesa:

```text
KDC inclui a identidade B na resposta.
Alice verifica se a resposta corresponde ao Bob que ela pediu.
```

---

## Reuso malicioso de session key antiga

Se um atacante obtém uma chave de sessão antiga, pode tentar reutilizá-la.

Defesas:

- nonces;
- desafios;
- timestamps;
- limitar tempo de vida de chaves;
- gerar nova session key por sessão;
- verificar frescor da chave.

---

# Autenticação com criptografia de chave pública

Agora não há KDC simétrico.

Hipótese:

```text
Alice conhece a chave pública verdadeira de Bob.
Bob conhece a chave pública verdadeira de Alice.
```

Cuidado:

```text
Se a chave pública for falsa, o protocolo falha.
```

---

## Protocolo conceitual

Alice quer autenticar Bob e estabelecer canal seguro.

1. Alice envia desafio `RA` cifrado com a chave pública de Bob.

```text
A -> B: PKB(RA)
```

Somente Bob consegue decifrar, pois só Bob tem `SKB`.

2. Bob decifra `RA`, gera seu desafio `RB` e uma session key `KA,B`.

Bob envia para Alice, cifrado com a chave pública de Alice:

```text
B -> A: PKA(RA, RB, KA,B)
```

3. Alice decifra com sua chave privada `SKA`.

Ela verifica `RA`.

Se `RA` está correto:

```text
Alice autentica Bob.
```

4. Alice responde ao desafio de Bob usando a session key.

```text
A -> B: KA,B(RB)
```

Se Bob verifica `RB`, ele autentica Alice.

Resultado:

```text
autenticação mútua
session key compartilhada
canal seguro inicializado
```

---

# Por que session keys são necessárias?

Mesmo quando chaves públicas/privadas ou chaves compartilhadas longas existem, usa-se uma session key temporária.

Motivos:

## 1. Reduzir uso de chaves de longa duração

Quanto mais uma chave é usada, maior o material disponível para criptoanálise.

Interpretação simples:

```text
chaves sofrem "desgaste" pelo uso
```

Logo:

```text
usar chaves longas apenas para autenticar e criar chaves temporárias
```

---

## 2. Melhor desempenho

Criptografia simétrica com session key costuma ser mais barata que criptografia assimétrica.

Uso comum:

```text
assimétrica para autenticar/trocar chave
simétrica para dados da sessão
```

---

## 3. Limitar dano

Se uma session key vaza, apenas aquela sessão é comprometida.

As chaves de longo prazo continuam protegidas.

---

## 4. Reduzir risco de replay de sessão

Cada sessão usa chave nova.

Reutilizar uma sessão inteira antiga fica mais difícil.

Para replay de mensagens individuais, ainda podem ser necessários:

- timestamps;
- sequence numbers;
- nonces;
- identificadores de sessão.

---

# Comparações essenciais

## Confidencialidade vs integridade vs autenticação

| Conceito | Pergunta | Mecanismo típico |
|---|---|---|
| Confidencialidade | Quem pode ler? | Criptografia |
| Integridade | A mensagem foi alterada? | Hash, MAC, assinatura |
| Autenticação | Quem enviou/quem é? | Senha, chave, certificado, challenge-response |

---

## Autenticação vs autorização

| Conceito | Pergunta | Exemplo |
|---|---|---|
| Autenticação | Quem é você? | Alice prova que é Alice |
| Autorização | O que você pode fazer? | Alice pode ler, mas não editar |

---

## Assinatura digital vs criptografia para sigilo

| Objetivo | Chave usada |
|---|---|
| Enviar mensagem secreta para Bob | chave pública de Bob |
| Provar que Alice assinou | chave privada de Alice |
| Verificar assinatura de Alice | chave pública de Alice |

---

## Hash vs criptografia

| Critério | Hash | Criptografia |
|---|---|---|
| Saída | tamanho fixo | tamanho relacionado à entrada |
| Reversível? | Não | Sim, com chave correta |
| Uso | integridade, digest, senha | confidencialidade |
| Exemplo | `H(m)` | `EK(P)` |

---

## Chave simétrica vs chave pública

| Critério | Simétrica | Pública/assimétrica |
|---|---|---|
| Chaves | uma compartilhada | par pública/privada |
| Velocidade | rápida | mais lenta |
| Problema principal | distribuição da chave | autenticar chave pública |
| Uso típico | dados de sessão | autenticação/troca de chave |

---

# Exemplos rápidos para prova

## Exemplo 1 — Interceptação

```text
Chuck lê uma mensagem entre Alice e Bob.
```

Ataque:

```text
eavesdropping
```

Propriedade ameaçada:

```text
confidencialidade
```

---

## Exemplo 2 — Modificação

```text
Alice envia "pagar 100".
Chuck altera para "pagar 1000".
```

Propriedade ameaçada:

```text
integridade
```

---

## Exemplo 3 — Inserção

```text
Chuck envia mensagem falsa fingindo ser Alice.
```

Propriedade ameaçada:

```text
autenticidade
```

---

## Exemplo 4 — Hash de senha

Sistema armazena:

```text
H(pw)
```

Quando usuário digita `pw'`, verifica:

```text
H(pw') == H(pw)
```

Objetivo:

```text
não armazenar senha em claro
```

---

## Exemplo 5 — Assinatura digital

Alice envia:

```text
[m, SKA(H(m))]
```

Bob verifica:

```text
H(m) == PKA(SKA(H(m)))
```

Objetivo:

```text
integridade e não repúdio
```

---

## Exemplo 6 — Challenge-response

Bob envia:

```text
RB
```

Alice responde:

```text
KA,B(RB)
```

Bob conclui:

```text
Alice conhece KA,B
```

---

## Exemplo 7 — Replay attack

Chuck grava uma resposta antiga de Alice e tenta reutilizar depois.

Defesa:

```text
nonce
challenge novo
timestamp
sequence number
session key nova
```

---

## Exemplo 8 — Reflection attack

Chuck usa Bob para cifrar o desafio que o próprio Bob exigiu.

Defesa:

```text
distinguir papéis
usar desafios diferentes
incluir contexto e identidade nas mensagens
```

---

# Fórmulas e notações importantes

## Criptografia

```text
C = EK(P)
P = DK(C)
```

---

## Criptografia simétrica

```text
P = DK(EK(P))
DK = EK
```

---

## Chaves par-a-par

```text
N(N - 1) / 2
```

---

## Hash

```text
h = H(m)
```

---

## Assinatura com digest

```text
sig = SKA(H(m))
```

Verificação:

```text
H(m) == PKA(sig)
```

---

## Confidencialidade com chave pública

```text
C = PKB(m)
m = SKB(C)
```

---

## Autenticação com chave privada

```text
sig = SKA(H(m))
verificar com PKA
```

---

# Perguntas típicas de prova

## 1. Qual a diferença entre política e mecanismo de segurança?

Política define o que é permitido ou proibido.

Mecanismo implementa a política.

---

## 2. Quais são os mecanismos básicos de segurança?

Criptografia, autenticação, autorização e auditoria.

---

## 3. Qual a diferença entre autenticação e autorização?

Autenticação verifica identidade.

Autorização verifica permissões.

---

## 4. O que é fail-safe default?

É projetar o sistema para negar acesso por padrão, exceto quando há permissão explícita.

---

## 5. O que é open design?

É não depender de segredo no projeto do mecanismo. A segurança deve depender das chaves e da correção do mecanismo, não de esconder o algoritmo.

---

## 6. O que é least privilege?

Cada entidade deve ter apenas os privilégios necessários para sua tarefa.

---

## 7. O que é TCB?

É o conjunto de componentes que precisam ser confiáveis para que a política de segurança seja aplicada.

---

## 8. Qual a diferença entre plaintext e ciphertext?

Plaintext é a mensagem original.

Ciphertext é a mensagem criptografada.

---

## 9. Qual a diferença entre criptografia simétrica e assimétrica?

Na simétrica, a mesma chave é usada para criptografar e descriptografar.

Na assimétrica, há um par de chaves pública e privada.

---

## 10. Como Alice envia uma mensagem confidencial para Bob usando chave pública?

Alice criptografa com a chave pública de Bob.

```text
C = PKB(m)
```

Bob descriptografa com sua chave privada.

---

## 11. Como Alice assina uma mensagem?

Alice assina o hash da mensagem com sua chave privada.

```text
sig = SKA(H(m))
```

Bob verifica com a chave pública de Alice.

---

## 12. O que uma função hash criptográfica deve garantir?

Ser one-way e resistente a colisões fracas e fortes.

---

## 13. Por que assinar o hash em vez da mensagem inteira?

Porque o hash tem tamanho fixo e é mais barato assinar.

---

## 14. O que é nonce?

Número aleatório usado uma única vez para ligar mensagens a uma execução específica do protocolo e evitar replay.

---

## 15. O que é challenge-response?

Protocolo em que uma parte envia um desafio aleatório, e a outra prova conhecer uma chave ao responder corretamente ao desafio.

---

## 16. Por que autenticação e integridade devem andar juntas?

Porque saber quem enviou uma mensagem não adianta se a mensagem pode ter sido alterada.

E saber que uma mensagem está íntegra não adianta se não se sabe quem enviou.

---

## 17. Qual o problema de chaves compartilhadas par-a-par?

Para `N` entidades, são necessárias `N(N - 1) / 2` chaves.

Isso não escala.

---

## 18. Como o KDC resolve o problema de escalabilidade de chaves?

Cada entidade compartilha uma chave apenas com o KDC.

O KDC gera chaves de sessão para pares que querem se comunicar.

---

## 19. O que é ticket?

Mensagem gerada pelo KDC, geralmente cifrada para o servidor, que permite ao cliente provar que recebeu uma chave autorizada.

---

## 20. Por que usar session keys?

Para reduzir uso de chaves de longa duração, melhorar desempenho, limitar dano em caso de vazamento e reduzir risco de replay de sessões.

---

# Respostas curtas para decorar

## Security policy

```text
Especifica o que é permitido e proibido.
```

## Security mechanism

```text
Implementa uma política de segurança.
```

## Confidentiality

```text
Somente entidades autorizadas podem ler a informação.
```

## Integrity

```text
Dados não são modificados sem autorização.
```

## Authentication

```text
Verificação da identidade alegada.
```

## Authorization

```text
Verificação de permissão para uma ação.
```

## Auditing

```text
Registro e análise de ações no sistema.
```

## Fail-safe defaults

```text
Negar por padrão.
```

## Open design

```text
Não depender de segredo no algoritmo.
```

## Least privilege

```text
Conceder apenas os privilégios necessários.
```

## TCB

```text
Componentes que precisam ser confiáveis para a segurança funcionar.
```

## Plaintext

```text
Mensagem original.
```

## Ciphertext

```text
Mensagem criptografada.
```

## Symmetric cryptography

```text
Mesma chave para criptografar e descriptografar.
```

## Asymmetric cryptography

```text
Par de chaves pública e privada.
```

## Hash

```text
Função que mapeia mensagem arbitrária para saída fixa.
```

## Message digest

```text
Hash de uma mensagem.
```

## Digital signature

```text
Assinatura feita com chave privada, normalmente sobre o hash da mensagem.
```

## Nonce

```text
Valor aleatório usado uma única vez.
```

## Challenge-response

```text
Prova de conhecimento de chave sem enviar a chave.
```

## KDC

```text
Terceiro confiável que distribui chaves de sessão.
```

## Session key

```text
Chave temporária usada durante uma sessão.
```

---

# Prioridade para estudar

## Prioridade alta

- Confidencialidade, integridade, disponibilidade.
- Autenticação vs autorização.
- Política vs mecanismo de segurança.
- Encryption, authentication, authorization e auditing.
- Fail-safe defaults.
- Open design.
- Separation of privilege.
- Least privilege.
- Least common mechanism.
- TCB.
- Plaintext, ciphertext, encryption e decryption.
- Ataques: interceptação, modificação e inserção.
- Criptografia simétrica vs assimétrica.
- Uso de chave pública para confidencialidade.
- Uso de chave privada para assinatura.
- Hash functions.
- One-way, weak collision resistance e strong collision resistance.
- Message digest.
- Assinatura digital com `sig = SKA(H(m))`.
- Autenticação por fatores.
- MFA.
- Challenge-response.
- Nonce.
- Replay attack.
- Reflection attack.
- KDC.
- Ticket.
- Needham-Schroeder em alto nível.
- Session keys.

## Prioridade média

- Privacy vs confidentiality.
- GDPR como preocupação de projeto.
- Onde implementar segurança: rede, SO, middleware ou aplicação.
- Honest-but-curious server.
- Homomorphic encryption.
- Continuous authentication.
- Diferença entre autenticação unilateral e mútua.
- Autenticação com chave pública.

## Prioridade baixa ou fora do escopo

- Key management em detalhe.
- Certificados e autoridade certificadora em detalhe.
- SSH em detalhe.
- Diffie-Hellman em detalhe.
- Kerberos.
- TLS/SSL.
- Trust.
- Authorization em profundidade.
- Firewalls.
- Intrusion detection.
