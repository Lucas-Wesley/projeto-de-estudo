# Roadmap: Banco, Nuvem e Escola

> Estudar porque precisa implementar, e implementar o que acabou de estudar.
> Criado em 21/11/2026. Português (5) e Inglês (17) ficam de fora de propósito.
> **Versão 3** (21/11/2026): os três livros com o sumário conferido e lidos por capítulo, no momento certo (seção 7.2); as trilhas que faltavam (T36 a T44), que fecham todas as lacunas; Rust do zero numa trilha paralela (R1 a R4); Kubernetes (5.8).
> Cada passo aqui é um **item** da metodologia: problema → estudo → checagem → caminhos → decisão → plano → execução → fechamento. Onde estamos: [ESTADO.md](ESTADO.md).

---

## 1. A ideia em uma página

Três "empresas" fictícias, cada uma com o seu repositório público no GitHub, que conversam entre si como empresas de verdade:

```
                 ┌────────────────────────────────────────────────┐
                 │                 NUVEM (infra)                  │
                 │  deploy por git push · HTTPS · banco gerenciado │
                 │  fila · object storage · segredos · métricas   │
                 └───────▲───────────────▲───────────────▲────────┘
                         │ roda em       │ roda em       │ roda em
                 ┌───────┴──────┐  ┌─────┴──────┐  ┌─────┴───────┐
                 │    BANCO     │◄─┤    SPI     ├─►│  BANCO 2    │
                 │ contas, PIX, │  │ "Banco     │  │ (outra      │
                 │ razão, extrato│  │  Central") │  │  instância) │
                 └───────▲──────┘  └────────────┘  └─────────────┘
                         │ cobra via PIX, webhook assinado
                 ┌───────┴──────────────────────────────────────┐
                 │  ESCOLA (produto)                             │
                 │  cursos, matrícula, juiz de código, vídeo,    │
                 │  aula ao vivo                                 │
                 └───────────────────────────────────────────────┘
```

- **Banco**: o domínio mais rico para DDD. Dinheiro não pode sumir, então ele força consistência, concorrência, idempotência, sagas e conciliação. **É por onde começamos**, porque dá para fazer só com Go, sem nenhuma infraestrutura.
- **Nuvem**: aparece quando o Banco precisa de uma casa. Cada coisa que o Banco ou a Escola pedem (um banco de dados, uma fila, um storage) vira um requisito da Nuvem.
- **Escola**: o produto que junta tudo. Ela é cliente do Banco (cobra as matrículas por PIX) e roda na Nuvem. Traz o juiz de código (que reaproveita o contêiner que você fez para a Nuvem), vídeo, CDN e tempo real.
- **SPI**: um simulador do Banco Central (o DICT, que diz de quem é cada chave PIX, e a liquidação). Ele permite rodar **dois bancos** trocando PIX, e é aí que aparecem os problemas de sistema distribuído de verdade.

### Dá para juntar os três? Dá, e fica melhor do que separado

Sem português e inglês, a Escola perderia parte do motivo de existir. O Banco sozinho fica fraco em infra e em algoritmos, e a Nuvem sozinha fica fraca em DDD. Juntos eles se completam:

| Trilha | Banco | Nuvem | Escola |
|---|:-:|:-:|:-:|
| T1 Home lab: rede | ○ | ● | ○ |
| T4 AWS Cloud Practitioner | ◐ | ● | ◐ |
| T8 Redes | ● | ● | ● |
| T9 DDD | ● | ◐ | ● |
| T10 Base em tecnologia | ● | ● | ◐ |
| T11 Algoritmos | ◐ | ● | ● |
| T12 Linux | ◐ | ● | ● |
| T13 Golang | ● | ● | ● |
| T14 AWS para engenharia | ◐ | ● | ◐ |
| T15 System Design | ● | ● | ● |
| T18 System Design T4 | ● | ● | ● |

● muito usada · ◐ usada em parte · ○ pouco

O DDD fica bem mais apurado, como você imaginou. São **três domínios diferentes** (financeiro, infraestrutura e educação) e **integrações entre organizações**, com anticorruption layer, contratos, webhooks e sagas entre empresas. Esse é o tipo de integração que uma trilha de DDD sozinha nunca te faz viver.

---

## 2. Juntos ou separados? (repositórios e microsserviços)

### A regra

Separe em outro serviço ou repositório **só quando houver um motivo concreto**. Os motivos são estes:

1. **Fronteira entre organizações ou de confiança.** É outra "empresa" ou outro nível de segurança.
2. **Roda em outra máquina por natureza.** Um agente em cada nó, um proxy na borda.
3. **Perfil de recurso ou escala diferente.** Transcodificação de vídeo come CPU, o juiz de código precisa de isolamento.
4. **Isolamento de falha ou segurança.** Se aquilo cair ou for invadido, não pode levar o resto junto.
5. **Outra linguagem.** Um componente em Rust, por exemplo.

Mantenha **junto** quando duas coisas precisam da **mesma transação**. Exemplo: o saldo e o lançamento nunca podem ser separados. Mantenha junto também quando elas sempre mudam juntas.

Todo o resto começa como **monólito modular** (T9 › Monólito modular), com módulos bem separados por dentro. Quando um módulo tiver um dos motivos acima, ele é **extraído** com um ADR explicando por quê. Fazer essa extração de propósito é o melhor exercício de microsserviços que existe. Ele conversa direto com o livro que você está lendo.

### Os repositórios

| Repositório | Formato | Por quê |
|---|---|---|
| `banco` | Monólito modular: contas, razão, PIX, extrato, antifraude | O núcleo transacional precisa ficar junto. A **antifraude** é extraída depois, de propósito, como exercício (motivos 3 e 4) |
| `spi` | Serviço próprio | É outra organização, o "Banco Central" (motivo 1) |
| `nuvem` | **Monorepo com vários binários**: `nuvemd` (API e plano de controle), `agente` (um por nó), `borda` (proxy de entrada), `construtor` (build) e a CLI `nuvem` | Aqui são microsserviços de verdade, porque cada um roda num lugar diferente (motivos 2 e 3) |
| `escola` | Monólito modular + `juiz` + `midia` como serviços à parte | O juiz roda código de estranhos (motivo 4). A mídia come CPU e escala de outro jeito (motivo 3) |
| `resiliencia` | Biblioteca Go | Retry com backoff, circuit breaker, rate limiter e bulkhead, usados pelos três |
| `minicontainer` | Projeto de estudo que vira peça | Namespaces e cgroups em Go. Depois vira o sandbox do juiz |
| `fila` | Broker de mensagens próprio (log, partições, consumer groups) | A Nuvem passa a oferecer como "fila gerenciada" e o Banco migra para ela |
| `kv` | KV distribuído com Raft | Vira a configuração e descoberta de serviço da Nuvem (como o etcd) |

### Repositórios públicos: o mínimo

- Licença MIT, README com o "porquê" de cada projeto e uma pasta `docs/adr/` com as decisões.
- Nenhum segredo no repositório. Variáveis de ambiente e um verificador de segredos (gitleaks) no CI.
- CI no GitHub Actions rodando `go vet`, lint, `go test -race` e build. Isso é uma lacuna, ver a seção 8.
- Commits só com a sua autoria.

---

## 3. As trilhas

Todas na Trilha (https://trilha.internal). A IA consulta o progresso por `GET /api/trilhas/{id}`.
As **pausadas** são fontes que o projeto puxa quando um item pede: não entram no Hoje até você ativar.

| Id | Trilha | Tipo | Estado |
|---|---|---|---|
| T1 | Home lab: rede | escrita pela IA | rascunho |
| T4 | AWS Cloud Practitioner (CLF-C02) | escrita pela IA | ativa |
| T8 | Redes de computadores para engenharia de software | escrita pela IA | ativa |
| T9 | DDD (Domain-Driven Design) | escrita pela IA | ativa |
| T10 | Conhecimento de base em tecnologia | escrita pela IA | ativa |
| T11 | Algoritmos e estruturas de dados | escrita pela IA | ativa |
| T12 | Linux para desenvolvimento e servidores | escrita pela IA | ativa |
| T13 | Golang do zero ao backend em produção | escrita pela IA | ativa |
| T14 | AWS para engenharia de software | escrita pela IA | ativa |
| T15 | System Design | escrita pela IA | ativa |
| T18 | System Design T4 (LINUXtips) | curso | ativa |
| T19 | Full Cycle 4.0 (catálogo) | catálogo de cursos | pausada |
| T20 | Tech Leads Club | curso | ativa |
| **T21** | **Livro: Fundamentos da Arquitetura de Software** (Richards e Ford, 2ª ed.) | livro | pausada |
| **T22** | **Livro: Projetando Aplicações com Uso Intensivo de Dados** (Kleppmann e Riccomini, 2ª ed.) | livro | pausada |
| **T23** | **Livro: Aprenda Domain-Driven Design** (Khononov) | livro | pausada |
| T24 | Go Expert (Full Cycle) | curso | pausada |
| T25 | Formação DevOps Pro | curso | pausada |
| T26 | Algoritmos e estruturas de dados (Elemar Jr) | curso | pausada |
| T27 | Padrões de projeto (Elemar Jr) | curso | pausada |
| T28 | DDD do jeito certo (Elemar Jr) | curso | pausada |
| T29 | Arquitetura de software (MBA Full Cycle) | curso | pausada |
| T30 | System Design (MBA Full Cycle) | curso | pausada |
| T31 | DDD (MBA Full Cycle) | curso | pausada |
| T32 | Microsserviços, eventos e Kafka (MBA Full Cycle) | curso | pausada |
| T33 | Banco de dados (MBA Full Cycle) | curso | pausada |
| T34 | Infraestrutura: containers, cloud, IaC e edge (MBA Full Cycle) | curso | pausada |
| T35 | Observabilidade, DevOps e SRE (MBA Full Cycle) | curso | pausada |
| **T36** | **Rust do zero: memória sem coletor** | roteiro | pausada |
| **T37** | **Bancos de dados a fundo (PostgreSQL)** | roteiro | pausada |
| **T38** | **Domínio bancário e PIX** | roteiro | pausada |
| **T39** | **Autenticação, autorização e segurança de aplicações** | roteiro | pausada |
| **T40** | **Git, GitHub e CI/CD** | roteiro | pausada |
| **T41** | **Plataforma, SRE e virtualização** (a parte de DevOps que faltava) | roteiro | pausada |
| **T42** | **Testes de software** | roteiro | pausada |
| **T43** | **Mídia e streaming** | roteiro | pausada |
| **T44** | **Frontend por dentro** | roteiro | pausada |

As de **roteiro** (T36 a T44) seguem o padrão das trilhas 8 a 15: cada etapa diz o que aprender e como praticar, cada módulo fecha com um projeto, e o conteúdo entra depois (lição gerada ou "Encaixar material"), da melhor fonte. Vários projetos de módulo **são** itens deste roadmap (o Dinheiro em Rust, o esquema do Banco, o gatilho do git push, o sandbox do juiz).

Ficaram de fora, a seu pedido: Reputação e Marketing Pessoal (Elemar Jr); Marketing Pessoal, Liderança e Trabalho em Equipe, Empreendedorismo e Prompt Engineering (MBA); e os encontros ao vivo de carreira e mentoria. Os bônus FC TechWeek IA #1 e #2 também ficaram de fora, junto com o Prompt Engineering.

### Os livros no projeto
Os livros são a **teoria de fundo** e acompanham o projeto inteiro, capítulo a capítulo, no momento em que o item precisa:

- **T23 Khononov (DDD)** abre: o domínio vem antes da tecnologia (Fase 0).
- **T21 Richards e Ford (arquitetura)** acompanha o fio da arquitetura (seção 4) do começo ao fim.
- **T22 Kleppmann (dados)** entra quando o dado fica sério: o Postgres (Fase 1) e, depois, tudo que é distribuído (Fases 3, 4 e 6).

**Nenhum livro é lido de uma vez.** Cada capítulo é lido quando um item precisa dele, e os três foram escritos para isso: cada capítulo se sustenta sozinho. O plano de leitura, capítulo por capítulo, está na seção 7.2.

---

## 4. O fio da arquitetura

A arquitetura existe desde o dia 1. Ela **evolui por pressão**: cada estágio aguenta até certo ponto, e quando o gatilho dispara, a evolução vira um item (METODOLOGIA §5).

| Estágio | Quando | A arquitetura | O gatilho que força o próximo | Fitness function | Livros |
|---|---|---|---|---|---|
| **A0 · Domínio puro** | Fase 0 | Um módulo Go, só domínio e testes. Sem HTTP, sem banco. Decisão consciente: "começar simples" (ADR 1) | Precisamos de HTTP e de persistência | Testes de propriedade (a soma global é zero) | T23 cap. 1, 2, 5 e 6 · T21 cap. 1, 2 e 21 |
| **A1 · Camadas → hexagonal** | 1.1 a 1.3 | HTTP e Postgres entram como adaptadores; o domínio não sabe deles | Trocar memória por Postgres sem tocar no domínio (a dor das camadas acopladas aparece em 1.1) | Linter de imports: domínio não importa infra (1.2) | T21 cap. 3, 10 · T23 cap. 8 · T29 › Hexagonal e Clean |
| **A2 · Monólito modular** | 3.1 | Contas, Razão e PIX viram módulos com fronteiras e um context map | O PIX traz outro contexto com outra linguagem | Módulo só fala com outro pela API pública dele | T21 cap. 8, 11 · T23 cap. 3, 4, 12 |
| **A3 · Distribuído por fronteira de organização** | 3.2 a 3.7 | O SPI é outra "empresa": rede, sagas, outbox, idempotência | Dois bancos precisam trocar dinheiro sem transação comum | Teste de caos + conciliação fechando no zero | T21 cap. 9 (falácias) · T22 cap. 9 e 10 · T23 cap. 9 |
| **A4 · Primeira extração** | 4.1 a 4.4 | CQRS no extrato; a antifraude sai do monólito por motivo (escala e isolamento de falha) | Perfil de recurso diferente; uma falha não pode travar o PIX | Antifraude fora do ar não impede PIX | T21 cap. 7 (quanta), 14, 18, 19 · T23 cap. 11, 14 · T22 cap. 12 |
| **A5 · Plataforma** | Fase 5 | A Nuvem: agentes por nó, plano de controle, loop de reconciliação (microsserviços por natureza) | Apps precisam de banco, fila, segredos e vários nós | Apagou na mão, a Nuvem recria | T21 cap. 15 · T22 cap. 6 |
| **A6 · Kubernetes** | 5.8 | A mesma plataforma, agora no orquestrador da indústria, para comparar com a sua | Saber o que o K8s resolve que a sua Nuvem não resolve (e o custo disso) | Derrubar um nó do cluster sem perder o Banco | T25 · T34 |
| **A7 · Peças de dados próprias** | Fase 6 | Broker e KV seus, com consenso; uma peça em Rust | A Nuvem oferece fila e configuração "gerenciadas" | Mata o líder, o cluster segue sem duas verdades | T22 cap. 4, 6, 7, 10, 12 |
| **A8 · Produto em cima da plataforma** | Fases 7 e 8 | A Escola: monólito modular + serviços por motivo (juiz, mídia), orientada a eventos | Um segundo domínio reaproveitando tudo | Contratos entre Escola e Banco testados | T21 cap. 12, 15, 16 · T23 cap. 15 |
| **A9 · Arquitetura defendida** | Fase 9 | O ecossistema na AWS e o design doc revisado | — | Well-Architected | T21 cap. 22, 23, 26, 27 · T22 cap. 13 · T29 › Solution Architecture Document |

Toda decisão de arquitetura registra o gatilho em [DECISOES.md](DECISOES.md). Quando um gatilho dispara antes do planejado, o item de evolução é puxado para a frente.

---

## 5. Como ler os passos

Cada passo tem o mesmo formato:

- **Implementar**: o que vai existir no fim.
- **Estudar antes**: as etapas das suas trilhas, no formato `Tn › Módulo › Etapa`.
  - ✔ você tem e cobre bem
  - ◐ você tem, mas é raso para este uso e precisa aprofundar
  - ⏳ módulo da T18 que ainda não tem etapas (você está colando as aulas)
  - ✖ **lacuna**: nenhuma trilha cobre (na v3 não há mais nenhuma; ver a seção 8)
- **Pronto quando**: o critério objetivo para dar o passo como feito.
- **Repositório**: onde entra.

A ordem pula de trilha em trilha de propósito. Ela segue a ordem em que você vai **precisar** de cada coisa.

**"Estudar antes" é a lista de candidatos, não a ordem final.** Na fase de estudo do item, a IA olha todas as trilhas (inclusive as que entraram depois deste roadmap), escolhe a melhor fonte e prova por que ela é necessária (METODOLOGIA §3.2).

---

## 6. O roadmap

### Fase 0: o primeiro centavo (Go puro, sem infra)
*Objetivo: aprender Go modelando dinheiro. Zero banco de dados, zero rede.*

#### 0.1 O repositório e o ambiente
- **Implementar:** o repositório `banco` no GitHub com `go.mod`, `Makefile` (test, lint, run), um `main.go` que imprime a versão e um teste passando.
- **Estudar antes:**
  - ✔ T13 › Fundamentos › Por que Go e o ambiente
  - ✔ T12 › Terminal e shell › O shell; Pipes, redirecionamento e exit codes
  - ✔ T13 › Pacotes, módulos e testes › Módulos e dependências
  - ✔ T40 › Git no dia a dia; Colaboração no GitHub
- **Pronto quando:** `make test` passa e o repositório público tem README e licença.
- **Repositório:** `banco`

#### 0.2 Dinheiro sem float
- **Implementar:** o value object `Dinheiro` em centavos (`int64`), com soma, subtração, comparação, **rateio** (R$ 10,00 dividido por 3 sem sumir centavo) e detecção de overflow.
- **Estudar antes:**
  - ✔ T10 › Números e bits › Inteiros: tamanho, sinal e overflow
  - ✔ T10 › Números e bits › Ponto flutuante (IEEE 754). É aqui que você vê por que dinheiro nunca é float.
  - ✔ T13 › Fundamentos › Tipos, variáveis e controle de fluxo; Funções
  - ✔ T9 › Tático I › Value objects (pulando à frente na T9, de propósito)
  - ✔ T13 › Pacotes, módulos e testes › Testes de tabela e subtestes
- **Pronto quando:** os testes de tabela cobrem soma, rateio e overflow, e um teste de fuzzing garante que o rateio sempre soma o total (T13 › Benchmarks, fuzzing e mocks).
- **Repositório:** `banco`

#### 0.3 Conta, razão e partidas dobradas (em memória)
- **Implementar:** a entidade `Conta` e o `Razão` (o livro de lançamentos). Toda movimentação gera **dois lançamentos**, um débito e um crédito, e o saldo é derivado dos lançamentos. As invariantes: uma conta comum nunca fica negativa, e a soma de tudo no sistema é sempre zero.
- **Estudar antes:**
  - ✔ T9 › Por que DDD › Complexidade essencial e acidental; Modelo de domínio e linguagem ubíqua
  - ✔ T9 › Tático I › Entidades e identidade; Agregados e invariantes
  - ✔ T13 › Tipos compostos › Structs e embutimento; Maps; Arrays e slices por dentro
  - ✔ T13 › Métodos, interfaces e erros › Métodos e receptores; Erros idiomáticos
  - ✔ T38 › Contabilidade para quem programa; Contas e dinheiro
- **Pronto quando:** existe um glossário (a linguagem ubíqua) no README e um teste de propriedade mostra que, depois de N operações aleatórias, a soma global é zero.
- **Repositório:** `banco`

#### 0.4 Mil transferências ao mesmo tempo
- **Implementar:** transferências concorrentes sem perder centavo. Primeiro do jeito errado, para ver o detector de corrida apontar o problema. Depois com mutex por conta, **ordenando os locks** para não dar deadlock.
- **Estudar antes:**
  - ✔ T13 › Concorrência › Goroutines; Channels e select; sync, atomic e o detector de corrida
  - ✔ T13 › Padrões de concorrência › Vazamentos e deadlocks
  - ✔ T9 › Tático I › Agregados na prática: concorrência e tamanho
- **Pronto quando:** `go test -race` passa com 1.000 goroutines transferindo entre 10 contas e a soma global continua zero.
- **Repositório:** `banco`

---

### Fase 1: Banco v0.1 (API e banco de dados)
*Objetivo: o Banco vira um serviço de verdade, que você usa por HTTP.*

#### 1.1 A API HTTP
- **Implementar:** os endpoints abrir conta, depositar, transferir, consultar saldo e extrato, com `net/http` puro (sem framework), JSON e erros padronizados.
- **Estudar antes:**
  - ✔ T18 › M1 › Protocolos TCP e UDP; HTTP/1.*, HTTP/2 e HTTP/3 (é o módulo em que você está agora)
  - ✔ T8 › HTTP e APIs › HTTP/1.1: requisição, resposta e cabeçalhos
  - ✔ T13 › Biblioteca padrão e APIs HTTP › Servidor HTTP com net/http; encoding/json e time; Validação, erros e configuração
  - ✔ T10 › Tempo e datas › Unix time, UTC e fusos; ISO 8601 (data do lançamento sempre em UTC)
- **Pronto quando:** um script `curl` (ou um arquivo `.http`) faz o fluxo inteiro e testes com `httptest` cobrem cada endpoint.
- **Repositório:** `banco`

#### 1.2 Arquitetura hexagonal
- **Implementar:** reorganizar em `dominio/` (sem imports de infra), `aplicacao/` (casos de uso), `adaptadores/http` e `adaptadores/memoria`. O repositório vira uma interface.
- **Estudar antes:**
  - ✔ T9 › Tático II › Serviços de domínio e de aplicação; Repositórios e factories
  - ✔ T9 › Arquitetura com DDD › Arquitetura hexagonal e clean
  - ✔ T13 › Métodos, interfaces e erros › Interfaces; T13 › Pacotes e visibilidade
- **Pronto quando:** um teste do caso de uso "transferir" roda sem HTTP e sem banco de dados, e um linter de imports (depguard) impede o domínio de importar infra.
- **Repositório:** `banco`

#### 1.3 Postgres, transações e locks
- **Implementar:** o adaptador Postgres, com migrations, a transferência numa única transação e duas estratégias comparadas: `SELECT … FOR UPDATE` e lock otimista por versão. Os dois resultados vão para um ADR.
- **Estudar antes:**
  - ✔ T13 › Dados e comunicação › database/sql e migrations; sqlc, ORMs e trade-offs
  - ✔ T18 › M3 › Modelo ACID
  - ✔ T15 › Dados em escala › SQL x NoSQL; Índices e consultas
  - ✔ T11 › Árvores de busca › B-trees e índices de banco
  - ⏳ T18 › M4 Databases, modelos de dados e indexação
  - ✔ T37 › Modelagem relacional; Transações e concorrência (MVCC, isolamento, anomalias, locks). **O ponto mais crítico do Banco.**
- **Pronto quando:** o teste da Fase 0.4 roda contra o Postgres real (com testcontainers) e a soma global continua zero.
- **Repositório:** `banco`

#### 1.4 Idempotência
- **Implementar:** o cabeçalho `Idempotency-Key` em toda operação que move dinheiro. Se a mesma requisição chegar duas vezes, ela devolve a mesma resposta e não duplica a transferência.
- **Estudar antes:**
  - ✔ T15 › Confiabilidade › Timeouts, retries, backoff e idempotência
  - ✔ T8 › Programação de rede › Timeouts, retries e pools de conexão no cliente
  - ⏳ T18 › M21 Patterns de resiliência
- **Pronto quando:** um teste dispara a mesma transferência 50 vezes em paralelo e só um débito acontece.
- **Repositório:** `banco`

#### 1.5 Autenticação
- **Implementar:** login do cliente com senha (Argon2), sessão ou token, e cada cliente só enxerga as próprias contas.
- **Estudar antes:**
  - ✔ T10 › Criptografia aplicada › Hash e HMAC; Senhas: bcrypt, scrypt e Argon2
  - ✔ T8 › HTTP e APIs › Segurança no navegador: CORS, cookies e cabeçalhos
  - ✔ T39 › Fundamentos; Sessões e tokens
- **Pronto quando:** existem testes do tipo "um cliente tenta ler a conta de outro" que retornam 404 (e não 403).
- **Repositório:** `banco`

#### 1.6 Rodar como serviço no homelab
- **Implementar:** build estático, um Dockerfile multi-stage, logs estruturados (`slog`), `/health` e `/ready`, e encerramento gracioso (termina as transferências em andamento antes de sair).
- **Estudar antes:**
  - ✔ T12 › Processos e serviços › Processos e sinais; systemd; journald e logs
  - ✔ T13 › Concorrência › context
  - ✔ T13 › Ferramentas, runtime e produção › Build e deploy
  - ◐ T8 › Redes de contêineres › Redes do Docker e do Compose (cobre a rede, não as imagens)
  - ✔ T25 › 10 Docker · T34 › 012 Docker & Kubernetes (Dockerfile até a imagem de produção)
- **Pronto quando:** o Banco roda no Proxmox via compose, e um `kill -TERM` no meio de uma carga não perde nenhuma transferência.
- **Repositório:** `banco`

#### 1.7 Ver o que acontece
- **Implementar:** métricas Prometheus (transferências por segundo, latência p50/p99, erros) e um painel no Grafana.
- **Estudar antes:**
  - ✔ T13 › Ferramentas, runtime e produção › Observabilidade
  - ✔ T15 › Fundamentos › Disponibilidade, SLI, SLO e SLA
  - ✔ T15 › Operação, segurança e design docs › Observabilidade
  - ⏳ T18 › M28 Observabilidade e monitoramento
  - ◐ **Prometheus, Grafana e OpenTelemetry na prática** (as trilhas dão o conceito, não a ferramenta)
- **Pronto quando:** um SLO está escrito no README ("99% das transferências em menos de 300 ms") e o painel mostra se ele está sendo cumprido.
- **Repositório:** `banco`

---

### Fase 2: Nuvem v0.1 ("dou um push e sobe")
*Motivo: o Banco precisa de uma casa melhor que um compose feito à mão.*

#### 2.1 O que é um contêiner, de verdade
- **Implementar:** um `minicontainer` em Go que roda um processo com namespaces (PID, mount, UTS, net), cgroups (limite de CPU e memória) e um rootfs próprio.
- **Estudar antes:**
  - ✔ T12 › Por dentro do Linux › Kernel, chamadas de sistema e strace; Namespaces e cgroups; Memória: virtual, cache, swap e OOM; Projeto: mini contêiner
  - ✔ T8 › Redes de contêineres › Namespaces, veth e bridges no Linux; Projeto: a rede do Docker feita à mão
  - ✔ T10 › Sistemas de arquivos › Blocos, inodes e metadados
- **Pronto quando:** `minicontainer run --mem 64m alpine sh` funciona, e um processo que estoura a memória morre por OOM sem derrubar o host.
- **Repositório:** `minicontainer`
- *A Nuvem v0.1 usa o Docker/containerd por baixo, por pragmatismo. O minicontainer serve para você entender o que acontece ali dentro, e volta como sandbox na Fase 7.*

#### 2.2 O agente do nó
- **Implementar:** o `agente`, um daemon que recebe "rode a imagem X com estas variáveis", conversa com a API do Docker e reporta o estado (rodando, caiu, reiniciando).
- **Estudar antes:**
  - ✔ T13 › Concorrência › context; T13 › Padrões de concorrência
  - ✔ T13 › Biblioteca padrão e APIs HTTP › Cliente HTTP e resiliência
  - ✔ T12 › Processos e serviços › systemd: unidades e serviços
- **Pronto quando:** se você mata o contêiner na mão, o agente percebe e sobe outro.
- **Repositório:** `nuvem`

#### 2.3 A borda: proxy reverso próprio
- **Implementar:** o `borda`, um proxy reverso em Go que roteia `app.nuvem.internal` para o contêiner certo e se atualiza sozinho quando um app sobe ou cai. Mais um DNS curinga `*.nuvem.internal` no AdGuard.
- **Estudar antes:**
  - ✔ T1 › Fundamentos › DNS: de um nome para um IP
  - ✔ T18 › M1 › DNS
  - ✔ T8 › DNS a fundo › Registros DNS; Projeto: o DNS próprio do laboratório
  - ✔ T8 › Borda › Proxy direto, proxy reverso e API gateway; Balanceamento de carga: L4 × L7 e algoritmos
  - ✔ T15 › Blocos de construção › Balanceamento de carga
  - ⏳ T18 › M7 Load balancers e proxies reversos
- **Pronto quando:** com dois apps no ar, cada um responde no seu nome e o proxy balanceia entre duas réplicas.
- **Repositório:** `nuvem`

#### 2.4 HTTPS automático
- **Implementar:** uma CA própria da Nuvem e emissão automática de certificado por app (ACME interno, ou a CA da própria Nuvem assinando), com renovação.
- **Estudar antes:**
  - ✔ T10 › Criptografia aplicada › Criptografia simétrica e assimétrica; Certificados e PKI
  - ✔ T18 › M1 › SSL e TLS
  - ✔ T8 › Segurança de rede › Criptografia para quem desenvolve; TLS e certificados; PKI própria, ACME e mTLS
- **Pronto quando:** um app novo abre com cadeado no Mac e no Samsung (com a CA instalada) sem nenhum passo manual.
- **Repositório:** `nuvem`

#### 2.5 Do git push ao app no ar
- **Implementar:** o `construtor`. Um `git push nuvem main` dispara um hook que faz o build da imagem pelo Dockerfile do repositório, depois o deploy e a troca da versão.
- **Estudar antes:**
  - ✔ T12 › Shell script › Scripts robustos; Projeto: script de deploy
  - ✔ T8 › Infra como código › CI/CD e rede: deploy sem downtime
  - ✔ T40 › Git por dentro (o projeto do módulo é este gatilho)
  - ✔ T25 › 10 Docker (build e cache de camadas)
- **Pronto quando:** você dá push numa mudança do Banco e ela está no ar em menos de 2 minutos.
- **Repositório:** `nuvem`

#### 2.6 API, CLI e logs ao vivo
- **Implementar:** o `nuvemd` (API: apps, deploys, variáveis) e a CLI `nuvem` (`apps`, `deploy`, `logs -f`, `env set`). Os logs chegam em tempo real por SSE.
- **Estudar antes:**
  - ✔ T13 › Fundamentos › Projeto: CLI de conversão
  - ✔ T8 › HTTP e APIs › Tempo real: WebSockets, SSE e long polling
  - ✔ T15 › Comunicação entre serviços › REST, gRPC e GraphQL
  - ✔ T13 › Ferramentas, runtime e produção › Ferramentas: vet, lint e formatação
- **Pronto quando:** você faz todo o ciclo de vida de um app só pela CLI.
- **Repositório:** `nuvem`

#### 2.7 O Banco se muda para a Nuvem
- **Implementar:** o Banco roda na Nuvem com deploy blue/green (sobe a versão nova, testa e troca o tráfego).
- **Estudar antes:**
  - ✔ T8 › Infra como código › Projeto: pipeline que publica sem derrubar
  - ⏳ T18 › M22 Modelos de deployment: blue/green, canary, feature toggles
- **Pronto quando:** um teste de carga rodando durante um deploy termina com zero erros.
- **Repositórios:** `nuvem` e `banco`

---

### Fase 3: Banco v0.2 (PIX entre dois bancos)
*Aqui o sistema vira distribuído de verdade. Esta é a fase mais rica para System Design e DDD.*

#### 3.1 Descobrir o domínio do PIX
- **Implementar:** um Event Storming do PIX (documento e imagem no repositório) e o context map: Contas, Razão, PIX, DICT e SPI.
- **Estudar antes:**
  - ✔ T9 › Descobrir o domínio › Event Storming: big picture; Process e design level; Projeto: workshop de descoberta
  - ✔ T9 › Design estratégico › Bounded context; Context mapping; Projeto: context map
  - ✔ T9 › Por que DDD › Subdomínios: core, suporte e genérico
  - ✔ T38 › O sistema de pagamentos; PIX
- **Pronto quando:** o context map diz para cada relação qual padrão ela usa (ACL, cliente/fornecedor etc.).
- **Repositório:** `banco` (pasta `docs/`)

#### 3.2 O SPI e o segundo banco
- **Implementar:** o serviço `spi`, com o DICT (chave para banco e conta) e a liquidação (debita um banco, credita o outro). **Duas instâncias do Banco** com ISPBs diferentes, conversando por **gRPC com mTLS**.
- **Estudar antes:**
  - ✔ T13 › Dados e comunicação › gRPC
  - ✔ T10 › Formatos de dados e compressão › Formatos binários: Protobuf, MessagePack, Avro
  - ✔ T8 › HTTP e APIs › Estilos de API: REST, gRPC e GraphQL
  - ✔ T8 › Segurança de rede › PKI própria, ACME e mTLS; Projeto: serviços com confiança zero
  - ⏳ T18 › M12 Comunicação síncrona
- **Pronto quando:** um PIX do Banco A para uma chave do Banco B cai na conta certa, e um cliente sem certificado é recusado.
- **Repositório:** `spi`, `banco`

#### 3.3 O PIX como saga
- **Implementar:** a transferência entre bancos como uma **saga**: reserva o valor na origem, envia ao SPI, o destino credita e a origem confirma. Cada passo tem a sua compensação em caso de falha.
- **Estudar antes:**
  - ✔ T9 › Integração entre contextos › Sagas e processos de longa duração
  - ✔ T15 › Consistência distribuída › Transações distribuídas: 2PC e sagas
  - ✔ T18 › M3 › Modelo BASE; Teorema CAP; PACELC
  - ✔ T15 › Dados em escala › CAP, PACELC e modelos de consistência
  - ⏳ T18 › M19 Saga pattern
- **Pronto quando:** um diagrama de estados da saga está no ADR e há testes para cada falha possível (o SPI cai antes, durante ou depois).
- **Repositórios:** `banco`, `spi`

#### 3.4 Eventos confiáveis (outbox)
- **Implementar:** o outbox, em que o evento ("PIX enviado") é gravado na **mesma transação** do lançamento e um publicador o entrega na fila depois. Por enquanto a fila é o NATS ou o RabbitMQ; na Fase 6 vira a sua.
- **Estudar antes:**
  - ✔ T9 › Tático II › Eventos de domínio
  - ✔ T9 › Integração entre contextos › Eventos de integração e contratos; Outbox e entrega confiável
  - ✔ T15 › Consistência distribuída › Outbox, CDC e consistência eventual
  - ✔ T15 › Comunicação entre serviços › Mensageria: filas x pub/sub x streaming
  - ✔ T13 › Dados e comunicação › Eventos e mensageria; Projeto: serviço com eventos
  - ✔ T8 › System design I › Comunicação síncrona × assíncrona; Mensageria e streaming
  - ⏳ T18 › M13 Comunicação assíncrona
- **Pronto quando:** você derruba a fila por 5 minutos, nenhum evento se perde, e os consumidores tratam eventos duplicados (consumidor idempotente).
- **Repositório:** `banco`

#### 3.5 Resiliência (a biblioteca `resiliencia`)
- **Implementar:** a biblioteca pública com timeout, retry com backoff exponencial e jitter, circuit breaker e bulkhead. O Banco passa a usá-la para chamar o SPI.
- **Estudar antes:**
  - ✔ T15 › Confiabilidade › Falhas em sistemas distribuídos; Circuit breaker, bulkhead e degradação
  - ✔ T8 › System design II › Resiliência: falhas parciais e como conviver com elas
  - ✔ T13 › Biblioteca padrão e APIs HTTP › Cliente HTTP e resiliência
  - ✔ T13 › Métodos, interfaces e erros › Generics
  - ⏳ T18 › M21 Patterns de resiliência; M25 Bulkhead patterns
- **Pronto quando:** a biblioteca tem testes determinísticos (com relógio falso) e um README que explica cada padrão.
- **Repositório:** `resiliencia`

#### 3.6 Caos e conciliação
- **Implementar:** um Toxiproxy entre os bancos e o SPI (latência, perda de pacote, corte). Um job de **conciliação** confere, no fim do dia, que o que o SPI liquidou bate com os razões dos dois bancos.
- **Estudar antes:**
  - ✔ T8 › Transporte › Projeto: TCP e UDP em rede ruim
  - ✔ T8 › Observabilidade e troubleshooting › Engenharia do caos na rede
  - ✔ T12 › Processos e serviços › Agendamento: cron e timers
  - ✔ T38 › Operação financeira
- **Pronto quando:** depois de uma hora de caos, a conciliação fecha no zero ou aponta exatamente a diferença.
- **Repositórios:** `banco`, `spi`

#### 3.7 Rate limit e IDs
- **Implementar:** limite de PIX por cliente e por chave (token bucket ou sliding window) e um ID de ponta a ponta (E2E) ordenável no tempo, no estilo ULID ou Snowflake.
- **Estudar antes:**
  - ✔ T15 › Confiabilidade › Rate limiting; Projeto: design de um rate limiter
  - ✔ T11 › Estruturas lineares › Two pointers, sliding window e prefix sum; Pilhas, filas e deques
  - ✔ T15 › Consistência distribuída › IDs únicos e relógios
  - ✔ T10 › Tempo e datas › Relógios: NTP e monotônico
  - ✔ T10 › Texto e codificações › Base64, hex e codificação de URL
- **Pronto quando:** o rate limiter entra na biblioteca `resiliencia` e os IDs gerados em dois nós nunca colidem (teste com 1 milhão de IDs).
- **Repositórios:** `banco`, `resiliencia`

---

### Fase 4: Banco v0.3 (leitura, antifraude e escala)

#### 4.1 Extrato com CQRS
- **Implementar:** um modelo de leitura do extrato, separado e atualizado por eventos, com paginação por cursor.
- **Estudar antes:**
  - ✔ T9 › Arquitetura com DDD › CQRS
  - ✔ T15 › Dados em escala › Índices e consultas
  - ⏳ T18 › M18 CQRS
- **Pronto quando:** um extrato de 100 mil lançamentos abre em menos de 50 ms, e a defasagem entre escrita e leitura aparece como métrica.
- **Repositório:** `banco`

#### 4.2 Experimento: razão com event sourcing
- **Implementar:** um branch ou módulo experimental em que o razão é **só eventos** e o saldo é uma projeção, com snapshots. Um ADR compara com a Fase 0.3.
- **Estudar antes:**
  - ✔ T9 › Arquitetura com DDD › Event sourcing
  - ⏳ T18 › M20 Event sourcing pattern
- **Pronto quando:** o ADR diz qual dos dois modelos fica e por quê.
- **Repositório:** `banco`

#### 4.3 Cache (incluindo um LRU seu)
- **Implementar:** cache-aside na consulta de chave do DICT, com invalidação por evento. Primeiro com o **seu** LRU em memória, depois com Redis ou Valkey.
- **Estudar antes:**
  - ✔ T11 › Hashing › Tabelas hash; Projeto: cache LRU
  - ✔ T11 › Estruturas lineares › Listas ligadas
  - ✔ T15 › Blocos de construção › Cache
  - ✔ T14 › Armazenamento e bancos › Cache: ElastiCache (como referência)
  - ⏳ T18 › M5 Estratégias de caching
- **Pronto quando:** o painel mostra a taxa de acerto e um teste prova que, depois de trocar o dono de uma chave, ninguém recebe o banco antigo.
- **Repositórios:** `spi`, `banco`

#### 4.4 Antifraude (e a primeira extração para microsserviço)
- **Implementar:** um serviço `antifraude` **extraído** do monólito. Ele usa regras simples, um grafo de transferências para detectar ciclos e "contas laranja", e um bloom filter de chaves bloqueadas. A conversa com o Banco é assíncrona, por eventos.
- **Estudar antes:**
  - ✔ T11 › Grafos › Representação e travessias; Ordenação topológica e ciclos
  - ✔ T11 › Strings, probabilísticas e complexidade › Estruturas probabilísticas
  - ✔ T15 › Comunicação entre serviços › Microsserviços x monólito modular
  - ✔ T9 › Design estratégico › Fronteiras, times e Conway
  - ✔ T9 › DDD no mundo real › Refatorar para um insight mais profundo
  - ⏳ T18 › M6 Microsserviços, monolitos e domínios; M15 Scale cube
  - ✔ T32 › 002 a 004 · T21 cap. 18 e 19 · T23 cap. 14
- **Pronto quando:** um ADR justifica a extração e a antifraude fora do ar **não** impede um PIX. Ela degrada (o PIX passa e é revisado depois) em vez de travar.
- **Repositório:** `banco` (monorepo ganha um segundo binário)

#### 4.5 Quanto o Banco aguenta?
- **Implementar:** testes de carga com k6, estimativa de capacidade, perfil com pprof e uma correção real de gargalo.
- **Estudar antes:**
  - ✔ T15 › Fundamentos › Estimativas de ordem de grandeza
  - ✔ T8 › Desempenho e testes de carga › Testes de carga e benchmarks; Projeto: achar e eliminar o gargalo
  - ✔ T13 › Ferramentas, runtime e produção › pprof e trace; Runtime: scheduler, GC e escape analysis
  - ✔ T12 › Desempenho e diagnóstico › Método USE e ferramentas; perf e flame graphs
  - ✔ T11 › Análise de algoritmos › Big-O; Projeto: laboratório de benchmarks
  - ⏳ T18 › M14 Performance; M23 Capacity planning; M24 Testes de carga e estresse
- **Pronto quando:** existe um relatório "aguenta X PIX/s com p99 de Y ms, o gargalo é Z", com o antes e depois da correção.
- **Repositório:** `banco`

#### 4.6 Réplica e failover
- **Implementar:** o Postgres com uma réplica de leitura (o extrato lê dela) e um failover ensaiado. Backup com restauração testada.
- **Estudar antes:**
  - ✔ T15 › Dados em escala › Replicação
  - ✔ T12 › Discos e sistemas de arquivos › Backup e restauração
  - ✔ T18 › M2 › Storage, RAID (o porquê da durabilidade)
  - ✔ T10 › Sistemas de arquivos › SSD, HDD, RAID e durabilidade
  - ⏳ T18 › M17 Replicação de dados; M27 SPOF e disaster recovery
  - ◐ PostgreSQL a fundo (replicação física e lógica) — a mesma lacuna da 1.3
- **Pronto quando:** você derruba o primário e mede o RTO e o RPO de verdade, e uma restauração a partir do backup funciona.
- **Repositório:** `banco` (e um runbook em `docs/`)

---

### Fase 5: Nuvem v0.2 (serviços gerenciados)
*Cada passo aqui nasce de algo que o Banco pediu nas fases 3 e 4.*

#### 5.1 Banco de dados gerenciado
- **Implementar:** `nuvem db create` sobe um Postgres com volume próprio, backup diário automático, credenciais injetadas no app e réplica opcional.
- **Estudar antes:**
  - ✔ T14 › Armazenamento e bancos › RDS e Aurora (o modelo de referência)
  - ✔ T4 › Armazenamento e bancos de dados › Bancos de dados
  - ✔ T12 › Discos e sistemas de arquivos › Discos, partições e montagem; LVM; Projeto: disco de dados
  - ✔ T18 › M2 › Block Storage; DAS; NAS
- **Pronto quando:** o Banco passa a usar o banco gerenciado e o backup é restaurado num teste automático por semana.
- **Repositório:** `nuvem`

#### 5.2 Segredos gerenciados
- **Implementar:** um cofre com envelope encryption (uma chave mestra criptografa as chaves de dados), com rotação e auditoria de quem leu o quê.
- **Estudar antes:**
  - ✔ T10 › Criptografia aplicada › Criptografia simétrica e assimétrica
  - ✔ T4 › Segurança e identidade › Criptografia e proteção de dados: KMS, CloudHSM…; Identidade federada… guarda de segredos
  - ✔ T14 › Segurança, observabilidade e custo › Segredos e criptografia
- **Pronto quando:** nenhum app tem segredo em variável de ambiente em texto puro no disco, e a rotação da chave não derruba nada.
- **Repositório:** `nuvem`

#### 5.3 Object storage (um mini S3)
- **Implementar:** uma API compatível com o básico do S3 (PUT, GET, LIST, URLs assinadas) com armazenamento por hash do conteúdo. Ela vai guardar os backups do Banco e, depois, os vídeos da Escola.
- **Estudar antes:**
  - ✔ T18 › M2 › Object Storage; File Storage
  - ✔ T10 › Sistemas de arquivos › Bloco, arquivo e objeto
  - ✔ T14 › Armazenamento e bancos › S3 a fundo; T4 › Amazon S3
  - ✔ T10 › Criptografia aplicada › Hash e HMAC (é como funciona a URL assinada)
- **Pronto quando:** o `aws s3 cp` (a CLI oficial, apontada para a sua Nuvem) consegue enviar e baixar arquivos.
- **Repositório:** `nuvem`

#### 5.4 IAM da Nuvem
- **Implementar:** usuários, tokens de API e políticas ("o app X pode ler o bucket Y"), com avaliação de política no estilo da AWS.
- **Estudar antes:**
  - ✔ T4 › Segurança e identidade › Responsabilidade compartilhada; IAM: usuário raiz, usuários, grupos, funções e políticas
  - ✔ T14 › Conta, acesso e ferramentas › IAM a fundo
  - ✔ T39 › Autorização (motores de política e o IAM em miniatura)
- **Pronto quando:** o teste "app sem permissão tenta ler o bucket" falha com uma mensagem clara de qual política negou.
- **Repositório:** `nuvem`

#### 5.5 Vários nós e o escalonador
- **Implementar:** a Nuvem em 3 VMs do Proxmox. Um escalonador escolhe o nó (bin packing por CPU e memória), health checks reagendam os apps quando um nó cai, e uma rede overlay com WireGuard liga os nós numa VLAN de laboratório separada.
- **Estudar antes:**
  - ✔ T11 › Técnicas de projeto de algoritmos › Algoritmos gulosos; T11 › Heaps e filas de prioridade
  - ✔ T15 › Blocos de construção › Escala vertical, horizontal e serviços stateless
  - ✔ T8 › Segurança de rede › VPN e túneis: WireGuard, IPsec e SSH
  - ✔ T8 › Enlace e rede local › VLANs, Wi-Fi e segmentação da rede
  - ✔ T1 › Redes separadas e firewall (toda a seção)
  - ✔ T4 › Computação › Elasticidade e alta disponibilidade
  - ✔ T41 › Virtualização; Contêineres e microVMs por dentro
  - ✔ T41 › Orquestração por dentro
- **Pronto quando:** você desliga uma VM e todos os apps voltam nas outras em menos de 1 minuto.
- **Repositório:** `nuvem`

#### 5.6 Autoscaling e observabilidade como serviço
- **Implementar:** réplicas que sobem e descem por métrica (CPU ou requisições), e logs e métricas de cada app disponíveis pela CLI e no painel, com tracing distribuído entre o Banco, o SPI e a antifraude.
- **Estudar antes:**
  - ✔ T14 › Computação › Auto Scaling; T4 › Elasticidade
  - ✔ T8 › Observabilidade e troubleshooting › Tracing distribuído e logs de acesso; Métricas de rede, SLIs e SLOs
  - ✔ T14 › Segurança, observabilidade e custo › CloudWatch e X-Ray (referência)
- **Pronto quando:** sob carga o Banco sobe de 2 para 5 réplicas e volta, e um PIX lento aparece num trace com o culpado.
- **Repositório:** `nuvem`

#### 5.7 Tudo declarado (IaC da sua Nuvem)
- **Implementar:** um `nuvem.yaml` por app (réplicas, banco, segredos, domínio) e um loop de reconciliação que aplica as diferenças, no estilo GitOps.
- **Estudar antes:**
  - ✔ T14 › Infraestrutura como código › CloudFormation; Terraform na AWS
  - ✔ T8 › Infra como código › Rede como código com Terraform; GitOps
  - ✔ T41 › Orquestração por dentro; Infraestrutura como código
- **Pronto quando:** você apaga um app na mão, e a Nuvem recria a partir do yaml.
- **Repositório:** `nuvem`

#### 5.8 Kubernetes de verdade (e a comparação com a sua Nuvem)
- **Implementar:** um cluster Kubernetes de 3 nós nas VMs do Proxmox (k3s ou kubeadm), o Banco nele com Helm, Ingress, cert-manager e Prometheus, e um deploy canário. Um ADR compara o que o K8s faz e a sua Nuvem não (e o contrário), com o custo de operar cada um.
- **Estudar antes:**
  - ✔ T25 › capítulos 7 a 9 (Kubernetes local, como serviço e com kubeadm), 12 a 29 (deploy, services, configuração, ciclo do pod, volumes, distribuição dos pods, segurança, Ingress, Helm, estratégias de deploy)
  - ✔ T34 › 012 Docker & Kubernetes (Introdução ao Kubernetes até Helm)
  - ✔ T25 › 31 Traefik; 32 Cert-Manager e Let's Encrypt; 30 Prometheus e Grafana
  - ✔ T19 › Kubernetes, Helm, Pipelines/GitOps/ArgoCD (catálogo, se quiser aprofundar)
  - ✔ o que você mesmo construiu em 5.5 a 5.7 (o loop de reconciliação é o mesmo conceito)
- **Pronto quando:** você derruba um nó e o Banco continua respondendo, um canário com erro volta sozinho, e o ADR está escrito.
- **Repositório:** `nuvem` (pasta `k8s/`)

---

### Fase 6: peças de baixo nível (repositórios próprios)
*O ponto em que você troca uma peça pronta por uma sua e a Nuvem passa a oferecê-la. É também o lugar natural para o Rust.*

#### 6.1 O seu broker de mensagens
- **Implementar:** o `fila`, um log append-only em disco com partições, offsets, consumer groups e retenção. A Nuvem oferece como "fila gerenciada" e o Banco troca o NATS por ele.
- **Estudar antes:**
  - ✔ T11 › Estruturas lineares (filas, arrays dinâmicos)
  - ✔ T10 › Formatos de dados e compressão › Formatos binários; Compressão
  - ✔ T10 › Números e bits › Endianness e alinhamento
  - ✔ T12 › Desempenho e diagnóstico › Disco e I/O
  - ✔ T13 › Biblioteca padrão › io, bufio, os e arquivos
  - ✔ T14 › Mensageria e integração › SQS; SNS e fan-out; Kinesis e streaming (referência)
  - ⏳ T18 › M13 Comunicação assíncrona; M16 Sharding e particionamento
  - ✔ T37 › Por dentro de um banco · T22 cap. 4 e 12
- **Pronto quando:** você mata o broker durante a escrita e, ao voltar, nenhuma mensagem confirmada se perdeu.
- **Repositório:** `fila`

#### 6.2 KV distribuído com Raft
- **Implementar:** o `kv`, com Raft (eleição, replicação do log, snapshots). A Nuvem passa a guardar nele a configuração e o registro de serviços.
- **Estudar antes:**
  - ✔ T15 › Consistência distribuída › Consenso: Raft
  - ✔ T8 › System design II › Partições de rede, consistência e CAP; Projeto: um key-value distribuído que aguenta falhas
  - ✔ T18 › M3 inteiro (CAP, PACELC, as críticas modernas)
  - ⏳ T18 › Extra: Read the papers (o paper do Raft)
  - ✔ T22 cap. 9 e 10
- **Pronto quando:** com 3 nós, você derruba o líder e o cluster continua aceitando escritas, e um teste de partição de rede não produz duas verdades.
- **Repositório:** `kv`

#### 6.3 Consistent hashing
- **Implementar:** um anel de consistent hashing com nós virtuais, usado para distribuir as partições do `fila` e o cache da borda.
- **Estudar antes:**
  - ✔ T8 › System design II › Escala: sharding, consistent hashing e descoberta de serviço
  - ✔ T15 › Dados em escala › Particionamento e sharding
  - ⏳ T18 › M16 Sharding e particionamento
- **Pronto quando:** ao adicionar um nó, só cerca de 1/N das chaves mudam de lugar (medido).
- **Repositórios:** `fila`, `nuvem`

#### 6.4 Rust: a peça de verdade
- **Por quê aqui:** o segmento de log do `fila` é código de baixo nível, quente e cheio de buffers. É o lugar em que o modelo de memória do Rust faz diferença de verdade.
- **Antes:** a trilha paralela de Rust (R1 a R4, logo abaixo) precisa estar feita. Você chega aqui já tendo escrito Rust quatro vezes.
- **Implementar:** o segmento de log do `fila` reescrito em Rust (append, leitura por offset, índice esparso), chamado pelo broker em Go (por processo separado ou FFI, decisão do item).
- **Estudar antes:**
  - ✔ T36 › Baixo nível (unsafe, FFI, I/O em disco: buffers, mmap e fsync)
  - ✔ T22 cap. 4 (log estruturado, LSM) e T37 › Por dentro de um banco
- **Pronto quando:** a peça em Rust passa nos mesmos testes da versão Go, com o benchmark comparado e um ADR sobre a fronteira Go–Rust.
- **Repositório:** `fila`

---

### Trilha paralela: Rust do zero (R1 a R4)
*Você nunca viu Rust. Então ele entra aos poucos, em paralelo às fases, sempre reescrevendo algo que você já fez em Go: o problema é conhecido e só a linguagem é nova. A T36 é a base de todos.*

| Item | Quando | Estudar | Implementar | Pronto quando |
|---|---|---|---|---|
| **R1** · Primeiro Rust | Depois da Fase 1 | T36 › Primeiros passos; Ownership; Tipos próprios e erros | O `Dinheiro` (0.2) em Rust | Os mesmos casos de teste do Go passam em Rust |
| **R2** · Rust organizado | Depois da Fase 3 | T36 › Abstração; Módulos, testes e ferramentas | Um leitor do extrato do Banco (CSV) em Rust, com biblioteca e binário | clippy sem avisos e testes de documentação rodando |
| **R3** · A memória comparada | Depois da Fase 4 | T36 › Smart pointers; **Rust x C++ x Go** | O **laboratório de memória**: use-after-free, data race e invalidação de iterador em C++, Go e Rust | O texto do laboratório explica os três modelos (RAII manual, coletor, ownership) com os 9 programas |
| **R4** · Concorrência e baixo nível | Depois da Fase 5 | T36 › Concorrência sem medo; Baixo nível | Um KV em TCP com Tokio e o protótipo do segmento de log | Pronto para o 6.4 |

Repositório: `rust-estudos` (R1 a R3) e `fila` (R4 em diante).

---

### Fase 7: Escola v0.1 (cliente do Banco, rodando na Nuvem)

#### 7.1 O domínio da Escola
- **Implementar:** o Event Storming e o context map da Escola: Catálogo, Matrícula, Aprendizado, Avaliação (juiz), Cobrança (ACL para o Banco), Mídia e Aulas ao vivo. A decisão de o que é monólito e o que é serviço vai num ADR.
- **Estudar antes:**
  - ✔ T9 › Descobrir o domínio; T9 › Design estratégico (agora você reaplica com outro domínio)
  - ✔ T9 › Arquitetura com DDD › Monólito modular
  - ✔ T15 › Comunicação entre serviços › Microsserviços x monólito modular
- **Pronto quando:** o context map mostra a Escola como **cliente** do Banco (cliente/fornecedor + ACL).
- **Repositório:** `escola`

#### 7.2 Catálogo, matrícula e progresso
- **Implementar:** o núcleo do monólito modular, com cursos, módulos, aulas, matrícula e progresso.
- **Estudar antes:**
  - ✔ T9 › Tático I e II (agregados, eventos e repositórios, agora num domínio novo)
  - ✔ T13 › Biblioteca padrão e APIs HTTP › Projeto: API REST completa
- **Pronto quando:** existe um curso com aulas e um aluno matriculado avançando.
- **Repositório:** `escola`

#### 7.3 Pagar a matrícula com PIX do seu Banco
- **Implementar:** a Escola cria uma cobrança no Banco (QR PIX) e o Banco avisa por **webhook assinado com HMAC**. A matrícula só é liberada com o pagamento confirmado, e há estorno se o aluno desistir dentro do prazo.
- **Estudar antes:**
  - ✔ T9 › Integração entre contextos › Anticorruption layer na prática; Eventos de integração e contratos; Projeto: dois contextos integrados
  - ✔ T10 › Criptografia aplicada › Hash e HMAC
  - ✔ T15 › Consistência distribuída › Projeto: design de pagamentos
- **Pronto quando:** um webhook duplicado, fora de ordem ou com a assinatura errada não libera nem cancela nada indevidamente.
- **Repositórios:** `escola`, `banco` (o Banco ganha a API de cobrança para lojistas)

#### 7.4 O juiz de código
- **Implementar:** o serviço `juiz`. O aluno envia código Go, a submissão entra numa fila, os workers rodam o código no **sandbox do `minicontainer`** (sem rede, com limite de CPU, memória e tempo, e um sistema de arquivos só de leitura) e devolvem o resultado dos testes.
- **Estudar antes:**
  - ✔ T12 › Por dentro do Linux › Namespaces e cgroups; Memória: OOM (revisão, agora com o olhar de segurança)
  - ✔ T12 › Pacotes, usuários e segurança › Firewall e endurecimento
  - ✔ T8 › Segurança de rede › Ataques e defesas de rede
  - ✔ T13 › Padrões de concorrência (worker pool); T13 › Concorrência › Projeto: produtor e consumidor
  - ✔ T15 › Blocos de construção › Filas e processamento assíncrono
  - ✔ T39 › Sandbox: rodar código de estranhos · T41 › Contêineres e microVMs por dentro
- **Pronto quando:** um `for {}`, um fork bomb, a leitura de `/etc/passwd` do host e uma tentativa de abrir conexão de rede falham, e nenhum deles afeta as outras submissões.
- **Repositório:** `escola` (binário `juiz` no monorepo)

#### 7.5 Os problemas do juiz são a trilha de Algoritmos
- **Implementar:** cada projeto da T11 vira um problema no juiz, com testes escondidos. Você estuda Algoritmos **resolvendo dentro do sistema que você construiu**.
- **Estudar antes:** a própria T11, na ordem dela, em paralelo com as outras fases.
- **Pronto quando:** os projetos da T11 (benchmarks, LRU, ordenação, mapa ordenado, autocomplete, rotas, diff) estão no juiz.
- **Repositório:** `escola`

#### 7.6 Vídeo das aulas
- **Implementar:** upload para o object storage da Nuvem (5.3), transcodificação com ffmpeg numa fila (o serviço `midia`), HLS em várias qualidades e cache na borda.
- **Estudar antes:**
  - ✔ T15 › Estudos de caso › Notificações e upload de vídeo
  - ✔ T8 › Borda › CDN e cache na borda; T15 › CDN e borda
  - ✔ T8 › HTTP e APIs › Cache HTTP e conteúdo condicional
  - ✔ T43 › Fundamentos de mídia; Vídeo sob demanda
- **Pronto quando:** uma aula de 1 hora toca no Samsung trocando de qualidade conforme a rede, e a segunda visualização vem do cache da borda.
- **Repositório:** `escola` (binário `midia`)

#### 7.7 Busca de cursos com autocomplete
- **Implementar:** um índice invertido simples e autocomplete com trie, sem Elasticsearch.
- **Estudar antes:**
  - ✔ T11 › Heaps e tries › Tries; Projeto: autocomplete
  - ✔ T11 › Strings, probabilísticas e complexidade › Busca em strings
  - ✔ T15 › Estudos de caso › Busca e autocomplete
  - ✔ T10 › Texto e codificações › Unicode e UTF-8; Normalização e comparação de texto (para "programação" achar "programacao")
- **Pronto quando:** a busca acha o curso mesmo sem acento e sugere enquanto você digita em menos de 20 ms.
- **Repositório:** `escola`

#### 7.8 Notificações
- **Implementar:** e-mail e push (ntfy) por fan-out, nos eventos "pagamento confirmado", "nova aula" e "submissão corrigida".
- **Estudar antes:**
  - ✔ T14 › Mensageria e integração › SNS e fan-out; Projeto: processamento assíncrono
  - ✔ T4 › Outros serviços › Integração de aplicações: SQS, SNS, EventBridge…
- **Pronto quando:** uma falha no envio de e-mail não trava nada, e a notificação é reenviada depois.
- **Repositório:** `escola`

---

### Fase 8: Escola v0.2 (ao vivo)

#### 8.1 O chat da aula
- **Implementar:** um chat por WebSocket que escala em várias instâncias, com pub/sub entre elas (na sua `fila` ou no Redis).
- **Estudar antes:**
  - ✔ T8 › Programação de rede › I/O bloqueante, não bloqueante e event loop; Projeto: servidor de chat e cliente resiliente
  - ✔ T15 › Comunicação entre serviços › Tempo real: WebSocket, SSE e long polling; Projeto: design de um chat
- **Pronto quando:** com 3 instâncias atrás da borda, mil clientes simulados conversam sem perder mensagens quando uma instância cai.
- **Repositório:** `escola`

#### 8.2 A aula ao vivo em vídeo (o clone do Meet)
- **Implementar:** uma sala de vídeo com WebRTC e um SFU em Go (com a biblioteca Pion), um servidor TURN e a sinalização por WebSocket.
- **Estudar antes:**
  - ✔ T8 › HTTP e APIs › WebRTC: conexão ponto a ponto, ICE e arquiteturas multiparte
  - ✔ T8 › Transporte › UDP e quando usá-lo; QUIC e o transporte moderno
  - ✔ T8 › IP › NAT, PAT e por que a internet funciona com IPv4 (é a razão do TURN)
  - ✔ T43 › Tempo real com WebRTC
- **Pronto quando:** 4 pessoas (Mac, Samsung e duas abas) estão numa sala, inclusive uma de fora de casa pelo Tailscale.
- **Repositório:** `escola`

#### 8.3 A live para muitos (o YouTube)
- **Implementar:** ingestão por RTMP (OBS → Escola), transcodificação ao vivo para HLS e distribuição pela borda com cache.
- **Estudar antes:**
  - ✔ T8 › Borda › CDN e cache na borda
  - ✔ T15 › Blocos de construção › CDN e borda
  - ✔ T43 › Ao vivo
- **Pronto quando:** um teste de carga com 500 espectadores simulados não aumenta o uso de CPU da origem.
- **Repositório:** `escola`

---

### Fase 9: e na AWS de verdade?
*Para fechar o ciclo e comparar com o que você construiu.*

#### 9.1 O Banco na AWS com IaC
- **Implementar:** o Banco rodando na AWS com Terraform (VPC, ECS Fargate, RDS, SQS, ALB) e CI/CD. Tudo destruído no fim do dia para não gerar custo.
- **Estudar antes:**
  - ✔ T14 inteira, principalmente Rede na AWS; Contêineres e serverless; Armazenamento e bancos; Mensageria; Infraestrutura como código
  - ✔ T8 › Redes na nuvem (toda a seção)
  - ✔ T4 › Cobrança, preços e suporte (para não levar susto na fatura)
- **Pronto quando:** `terraform apply` sobe tudo, `terraform destroy` tira tudo, e o custo do teste fica anotado.
- **Repositório:** `banco` (pasta `infra/aws`)

#### 9.2 Arquitetura defendida
- **Implementar:** um design doc do ecossistema inteiro revisado pelos seis pilares do Well-Architected, comparando a sua Nuvem com a AWS.
- **Estudar antes:**
  - ✔ T15 › Operação, segurança e design docs › Design docs e ADRs; Projeto: design doc revisado
  - ✔ T14 › Segurança, observabilidade e custo › Well-Architected; Projeto final: arquitetura defendida
  - ✔ T4 › Conceitos de nuvem › Well-Architected Framework
  - ⏳ T18 › M26 Cell-based pattern (como seria o Banco em células)
- **Pronto quando:** o documento existe e está publicado no README do ecossistema.

---

## 7. A ordem de estudo

### 7.1 Item a item

Somam-se ao "Estudar antes" de cada item. Livro sempre por **capítulo**, nunca inteiro. Na fase de estudo, a IA escolhe o mínimo que destrava a decisão do item e justifica (METODOLOGIA §3.2).

| Item | Livros (capítulos) | Trilhas e cursos |
|---|---|---|
| **0.1** Repositório | T21 cap. 1, 2 e 21 (o que é arquitetura; o ADR desde o primeiro dia) | T40 › Git no dia a dia; Colaboração no GitHub · T24 › 01 e 02 |
| **0.2** Dinheiro | T23 cap. 5 e 6 (lógica simples x modelo de domínio; objetos de valor) | T42 › Fundamentos · T24 › 03 Fundação; 08 Testing · T31 e T28 (objetos de valor) |
| **0.3** Razão | T23 cap. 1 e 2 (subdomínios, linguagem ubíqua) | **T38 › Contabilidade para quem programa; Contas e dinheiro** · T42 › TDD · T31 › Prática: Agregados |
| **0.4** Concorrência | — | T24 › 10 Multithreading · T42 › Propriedades e fuzzing |
| **1.1** API | T21 cap. 10 (camadas) | T24 › 04 e 09 APIs · T42 › Integração (testes de API) |
| **1.2** Hexagonal | T21 cap. 3 (modularidade) · T23 cap. 8 (portas e adaptadores) | T29 › 010 Hexagonal e Clean · T24 › 20 Clean Architecture · T27 · T29 › 008 SOLID |
| **1.3** Postgres | **T22 cap. 3, 4 e 8** (modelos, armazenamento, **transações e isolamento**) | **T37 › Modelagem; SQL além do CRUD; Índices e planos; Transações e concorrência** · T33 · T24 › 06, 17 SQLC, 18 UOW · T42 › Testcontainers |
| **1.4** Idempotência | T22 cap. 9 (redes não confiáveis) | T32 › Bônus · Comunicação e resiliência |
| **1.5** Autenticação | — | **T39 › Fundamentos; Sessões e tokens** |
| **1.6** Serviço | — | T24 › 05 Context; 21 Deploy · T25 › 10 Docker · T34 › 012 |
| **1.7** Observabilidade | T22 cap. 2 (percentis) · T21 cap. 4, 5 e 6 (características e funções de aptidão) | **T41 › SRE; Observabilidade na prática** · T35 › 019 · T25 › 30 e 34 |
| **R1** Primeiro Rust | — | **T36 › módulos 1 a 3** |
| **2.1 a 2.6** Nuvem v0.1 | T22 cap. 1 (nuvem x auto-hospedagem) · T21 cap. 13 (microkernel: a CLI e o agente) | **T40 › Git por dentro** (2.5) · T41 › Contêineres por dentro (2.1) · T25 › 10 e 33 · T24 › 16 Cobra CLI (2.6) |
| **2.7** Blue/green | — | T40 › Entrega contínua · T25 › 29 Estratégias de deploy |
| **3.1** Domínio do PIX | **T23 cap. 3, 4 e 12** · T21 cap. 8, 11 e 23 (componentes, monólito modular, diagramas C4) | **T38 › O sistema de pagamentos; PIX** · T20 › Event Storming · T31 › Core Domain · T29 › Encontros › MDL e C4 |
| **3.2** SPI e gRPC | T22 cap. 5 (codificação e evolução) · T21 cap. 9 (as falácias) | T24 › 14 gRPC · T39 › Segurança de APIs (mTLS) · T42 › Contratos |
| **3.3** Saga | **T22 cap. 8, 9 e 10** · T23 cap. 9 · T21 cap. 20 (orquestração x coreografia) | T32 › 008 Saga; Case Mercado Bitcoin |
| **3.4** Outbox | T23 cap. 9 e 15 · T22 cap. 12 (CDC) | T32 › 005 e 006; 016 Kafka · T24 › 11 · T37 › Replicação lógica e CDC |
| **3.5** Resiliência | — | T32 › Bônus · Comunicação e resiliência · T27 |
| **3.6** Caos e conciliação | T22 cap. 9 e 11 · T21 cap. 22 (risco) | **T38 › Operação financeira** · T41 › Confiabilidade (caos) |
| **3.7** Rate limit e IDs | T22 cap. 10 (geradores de IDs e relógios lógicos) | — |
| **R2** Rust organizado | — | **T36 › módulos 4 e 5** |
| **4.1** CQRS | T23 cap. 8 · T21 cap. 20 · T22 cap. 3 e 12 | T32 › 007 CQRS |
| **4.2** Event sourcing | **T23 cap. 7** · T22 cap. 3 e 12 | — |
| **4.3** Cache | — | T26 · T37 › Redis |
| **4.4** Extração | **T21 cap. 7, 14, 17, 18 e 19** · **T23 cap. 11 e 14** | T32 › 002 a 004 · T29 › Encontros › SOS monólito; Provocações · T30 › Bônus |
| **4.5** Carga | T22 cap. 2 | T26 › Big-O · T42 › Desempenho · T41 › Capacidade |
| **4.6** Réplica | **T22 cap. 6** | **T37 › Durabilidade e replicação** · T41 › Backup e DR |
| **R3** A memória comparada | — | **T36 › módulos 6 e 7** (inclui o C++) |
| **5.1 a 5.7** Nuvem v0.2 | T21 cap. 15 · T22 cap. 6 | **T41 › Virtualização; Orquestração por dentro; IaC** · **T39 › Autorização** (5.4) · T37 › Operação (5.1) · T34 › 018 e 022 · T24 › 15 Upload S3 |
| **5.8** Kubernetes | — | T25 · T34 · T40 › GitOps |
| **R4** Concorrência e baixo nível | — | **T36 › módulos 8 e 9** |
| **6.1** Broker | **T22 cap. 4 e 12** | T37 › Por dentro de um banco · T32 › 016 Kafka |
| **6.2** Raft | **T22 cap. 10** | — |
| **6.3** Consistent hashing | **T22 cap. 7** | T37 › Escala · T30 |
| **6.4** Rust | — | T36 › Baixo nível |
| **7.1** Domínio da Escola | T23 cap. 10, 13 e 18 · T21 cap. 19 e 24 | T31 · T28 |
| **7.3** Pagar com PIX | — | T38 › PIX (QR dinâmico, devolução) · T39 › Segurança de APIs |
| **7.4 e 7.5** Juiz | T21 cap. 12 (pipeline) | **T39 › Sandbox** · T26 e T11 (os problemas) |
| **7.6 e 8.x** Vídeo e ao vivo | T21 cap. 15 e 16 | **T43** inteira · T30 › Lab Netflix · T34 › 021 Edge |
| **Painel da Nuvem e da Escola** | — | **T44** (quando chegar a hora da interface) |
| **9.1** AWS | — | T34 › 018 e 022 · T25 › 35 · T40 › CI/CD (OIDC) |
| **9.2** Arquitetura defendida | **T21 cap. 22, 23, 25, 26 e 27** · T22 cap. 13 e 14 · T23 cap. 16 e 17 | T29 › 005 Solution Architecture Document · T30 › Design Docs |

### 7.2 Os livros, capítulo por capítulo

A ordem em que cada capítulo aparece no projeto. É assim que os livros são lidos: aos pedaços, quando o item pede.

**Os 58 capítulos estão encaixados** (27 + 14 + 18 — nenhum fica de fora). Quando o projeto terminar, os três livros estarão lidos inteiros. Alguns capítulos aparecem em mais de um item: a primeira vez é a leitura, as outras são revisão.

**T23 · Aprenda Domain-Driven Design (Khononov)**: abre o projeto e volta a cada domínio novo.

| Cap. | Assunto | Lido no item |
|---|---|---|
| 1 e 2 | Domínios, subdomínios e linguagem ubíqua | 0.3 |
| 5 e 6 | Lógica simples x modelo de domínio | 0.2 |
| 8 | Padrões de arquitetura (portas e adaptadores, CQRS) | 1.2 (e revisto em 4.1) |
| 3 e 4 | Contextos delimitados e integração entre eles | 3.1 |
| 12 | EventStorming | 3.1 |
| 9 | Padrões de comunicação (outbox, saga) | 3.3 e 3.4 |
| 15 | Arquitetura orientada a eventos | 3.4 |
| 7 | Modelando a dimensão do tempo (event sourcing) | 4.2 |
| 11 e 14 | Evoluir o design; microsserviços | 4.4 |
| 10, 13 e 18 | Heurísticas, DDD no mundo real, estudo de caso | 7.1 |
| 16 e 17 | Malha de dados; palavras finais | 9.2 |

**T21 · Fundamentos da Arquitetura de Software (Richards e Ford)**: acompanha o fio da arquitetura (seção 4).

| Cap. | Assunto | Lido no item |
|---|---|---|
| 1, 2 e 21 | O que é arquitetura, pensar em trade-offs, ADRs | 0.1 |
| 10 | Estilo em camadas | 1.1 |
| 3 | Modularidade (coesão, acoplamento) | 1.2 |
| 4, 5 e 6 | Características arquiteturais e funções de aptidão | 1.7 |
| 13 | Microkernel | 2.x |
| 8, 11 e 23 | Componentes, monólito modular, diagramas | 3.1 |
| 9 | Monolítico x distribuído e as falácias | 3.2 |
| 20 | Padrões arquiteturais (orquestração x coreografia, CQRS) | 3.3 e 4.1 |
| 22 | Risco arquitetural | 3.6 |
| 7, 14, 17, 18 e 19 | Quantum, baseado em serviços, SOA, microsserviços, escolher o estilo | 4.4 |
| 15 | Orientado a eventos | 5.x (e revisto em 7.6) |
| 12 | Pipeline | 7.4 |
| 16 | Baseado em espaço | 8.x |
| 26 e 27 | Interseções e as leis revisitadas | 9.2 |
| 24 | Tornando as equipes eficazes (limites, listas de verificação) | 7.1 (a Escola tem vários contextos, como se fossem times) |
| 25 | Negociação e liderança | 9.2 (defender a arquitetura é negociar) |

**T22 · Projetando Aplicações com Uso Intensivo de Dados (Kleppmann e Riccomini, 2ª ed.)**: entra quando o dado fica sério.

| Cap. | Assunto | Lido no item |
|---|---|---|
| 2 | Requisitos não funcionais (latência, percentis) | 1.7 |
| 3, 4 e 8 | Modelos de dados, armazenamento, **transações** | 1.3 |
| 9 | Problemas com sistemas distribuídos | 1.4 (redes) e 3.3 |
| 1 | Trade-offs, nuvem x auto-hospedagem | 2.x |
| 5 | Codificação e evolução | 3.2 |
| 10 | Consistência e consenso | 3.3, 3.7 e 6.2 |
| 12 | Processamento de fluxos (CDC, event sourcing) | 3.4, 4.1 e 6.1 |
| 11 | Processamento em lote | 3.6 |
| 6 | Replicação | 4.6 |
| 7 | Sharding | 6.3 |
| 13 e 14 | A filosofia do streaming; fazer a coisa certa | 9.2 |

---

## 8. Lacunas

**Situação v3: nenhuma lacuna aberta.** As que existiam viraram as trilhas T36 a T44 (roteiro pronto, conteúdo a encaixar).

| # | Lacuna | Coberta por |
|---|---|---|
| 1 | Git e GitHub (uso e por dentro) | T40 · T25 › 3 |
| 2 | Domínio bancário e PIX | T38 |
| 3 | PostgreSQL a fundo | T37 · T22 cap. 8 · T33 |
| 4 | Autenticação e autorização | T39 |
| 5 | Docker e contêineres a fundo | T25 › 10 · T34 › 012 · T41 › Contêineres por dentro |
| 6 | Microsserviços | T32 · T21 cap. 18 · T23 cap. 14 |
| 7 | Testes de software | T42 |
| 8 | Observabilidade na prática | T41 · T35 · T25 › 30 e 34 |
| 9 | Virtualização e orquestração | T41 · T25 · T34 |
| 10 | Bancos e brokers por dentro | T37 › Por dentro de um banco · T22 cap. 4 e 12 |
| 11 | Sistemas distribuídos a fundo | T22 cap. 6 a 10 |
| 12 | Mídia e streaming | T43 |
| 13 | Frontend por dentro | T44 |
| 14 | Rust (+ C++ para comparar) | T36 (trilha paralela R1 a R4) |
| 15 | IaC | T41 · T25 › 35 · T34 › 022 |
| 16 | DevOps e SRE | T41 · T40 · T35 |

**O que ainda é raso:**
- **T36 a T44** nasceram só com o roteiro: o conteúdo de cada etapa entra quando o item chegar nela (lição gerada ou material encaixado).
- **T18:** só os módulos 0 a 3 têm etapas. Os ⏳ ganham profundidade conforme você cola as aulas.
- **T1 Home lab: rede:** ainda é rascunho (usada na 2.3 e na 5.5).
- Os cursos da T19 (Full Cycle 4.0) e da T20 (Tech Leads) podem substituir o conteúdo das trilhas de roteiro quando forem baixados.

---

## 9. Ordem sugerida das primeiras semanas

1. **T23 cap. 1 e 2** + **T21 cap. 1, 2 e 21** + **T40 › Git no dia a dia** + **T13 › Fundamentos** → **0.1**
2. **T10 › Números e bits** + **T23 cap. 5 e 6** + **T42 › Fundamentos** → **0.2** (Dinheiro)
3. **T38 › Contabilidade para quem programa** + **T9 › Tático I** + **T13 › Tipos compostos e Métodos** → **0.3** (Razão)
4. **T13 › Concorrência** + **T24 › 10 Multithreading** → **0.4**
5. **T18 › M1** + **T13 › Biblioteca padrão e APIs HTTP** + **T21 cap. 10** → **1.1** (API)
6. **T21 cap. 3** + **T23 cap. 8** + **T9 › Arquitetura hexagonal** → **1.2** (a primeira evolução de arquitetura: A0 → A1)
7. **T22 cap. 3, 4 e 8** + **T37 › Transações e concorrência** + **T18 › M3** → **1.3**

A T18 segue em paralelo, um módulo por semana.

---

## 9b. O fim do projeto

O projeto tem fim. Ele termina quando **as duas condições** forem verdade:

1. **O item 9.2 está fechado**: o ecossistema (Banco, SPI, Nuvem e Escola) rodando, e a arquitetura defendida num design doc.
2. **Os três livros estão lidos inteiros**: as trilhas T21, T22 e T23 com 100% das etapas concluídas na Trilha.

Para isso não escapar no caminho:
- Todo fechamento de item confere se os capítulos de livro daquele item foram marcados na Trilha (METODOLOGIA §3.8).
- **Material novo nunca empurra um capítulo para fora.** Um vídeo, uma tecnologia ou um curso que você mandar entra como fonte extra de um item, ou vira um item novo, mas os capítulos continuam onde estão (METODOLOGIA §8).
- Se o roadmap mudar e um item sair, os capítulos dele vão para o item mais próximo. A tabela 7.2 sempre cobre os 58 capítulos.

---

## 10. Como este roadmap evolui

- **Quando entra uma trilha nova:** a IA relê as etapas dela, atualiza a seção 7 e as lacunas (seção 8) e propõe itens que só ela torna possíveis.
- **Quando a T18 ganha etapas num módulo:** os ⏳ daquele módulo viram ✔ com as etapas exatas.
- **Quando um item fecha:** ele é marcado aqui (✅ no título), o caderno fica em `itens/` e o próximo vai para o [ESTADO.md](ESTADO.md).
- **Quando um gatilho de evolução dispara antes da hora:** o item de arquitetura correspondente sobe na fila (seção 4).
- A **versão** deste documento sobe a cada revisão (v1 → v2 → …).

---

## 11. Metodologia

Definida pelo Lucas em 21/11/2026: **[METODOLOGIA.md](METODOLOGIA.md)**. A IA segue as instruções de [CLAUDE.md](CLAUDE.md).

Em uma linha: **para cada item, a IA apresenta o problema, indica e justifica o estudo, confere o aprendizado, discute os caminhos, registra a decisão, pergunta como o Lucas faria, confere cada passo da execução e registra tudo nos documentos.**
