# Crítica do Projeto de estudo (Claude Opus 5.5)

> Revisão pedida em `revisoes/PEDIDO-DE-CRITICA.md`, feita em 22/11/2026, só leitura.
> Li, nesta ordem: ROADMAP e METODOLOGIA (e anotei a visão de fora antes de seguir), DECISOES, CLAUDE.md, ESTADO, REPERTORIO, PROMPT, `itens/_MODELO.md`, as trilhas 22, 36, 37, 38, 41 e 42 pela API (mais a lista de todas e o `/api/projeto`), a regra `criar-trilha.md` e um trecho do código do app Trilha.

---

## 0. O veredito em cinco linhas

1. **O plano é bom no desenho e inviável no tamanho.** Pelas minhas contas, são de 4 a 7 anos a 1–2 h por dia. Sem fins intermediários, a chance de abandono é alta.
2. **O maior risco não é técnico: é o plano virar o projeto.** Já existem 39 trilhas, ~1.100 h de estudo catalogadas, uma aba na plataforma e 10 commits de planejamento, e 8 etapas concluídas no total.
3. **A metodologia ensina a decidir, mas quase não ensina a lembrar nem a escrever.** A IA registra tudo, a checagem é só imediata, e a revisão espaçada some quando a trilha é pausada (P7).
4. **A "evolução por pressão" é, na maior parte, um cronograma com roupa de pressão.** Diga isso com honestidade e deixe "não mudar" ser uma resposta possível.
5. **Livros por capítulo solto funcionam para o Khononov, mal para o Kleppmann e adiam o fim do projeto para daqui a anos.**

---

## 1. Visão de fora (antes de ler os porquês)

Anotei isto só com o ROADMAP e a METODOLOGIA. Mantive como estava; no fim da seção digo o que mudou depois de ler o resto.

- **O que está muito bom.** A Fase 0 (Go puro modelando dinheiro, ROADMAP §6, 0.1–0.4) é das melhores aberturas de estudo de Go que já vi: invariantes, rateio, fuzzing e o detector de corrida antes de qualquer HTTP. Os critérios de "Pronto quando" são objetivos e testáveis quase sempre (1.4 "50 vezes em paralelo e só um débito", 3.6 "a conciliação fecha no zero"). O domínio bancário força os problemas certos. A regra de separar serviços só por motivo concreto (ROADMAP §2) é madura.
- **Escopo.** São 60 itens, e vários são projetos de meses por si só: o KV com Raft (6.2), o broker (6.1), o mini S3 compatível com a CLI da AWS (5.3), o IAM (5.4), o escalonador com overlay (5.5), o SFU de WebRTC (8.2) e a live RTMP→HLS (8.3).
- **Ordem.** A Fase 2 (a Nuvem) chega cedo demais e sem pressão real: um compose no homelab funciona bem para um Banco com um usuário. Kubernetes só aparece no 5.8, tarde para quem quer plataforma. Tracing só no 5.6, quando o sistema ficou distribuído no 3.2.
- **Reinvenção demais.** Proxy, CA/ACME, S3, IAM, cofre, broker, KV, LRU, busca e SFU. Cada "faça o seu" ensina, mas dez deles diluem o que interessa.
- **A pressão A0–A9 (ROADMAP §4)** é uma sequência planejada. Sem carga real, não há gatilho real.
- **Inconsistências.** O CI é "uma lacuna, ver a seção 8" em ROADMAP §2, aparece como "coberta" em ROADMAP §8 linha 1, e nenhum item o implementa.
- **Banco de dados a fundo** está fraco para o objetivo declarado: faltam itens de EXPLAIN e planos, de migrations sem downtime, de pool de conexões e de particionamento.
- **Metodologia.** Oito fases para todo item viram ritual nos itens pequenos. O estudo vem sempre antes de tentar. A checagem é só imediata. A IA escreve todos os documentos, então o Lucas quase não escreve. A IA "diz qual é o melhor caminho", o que na prática é a IA decidindo.

**O que mudou depois de ler o resto:** o P1 já prevê "um modo curto se o ciclo ficar pesado", e o P4 já admite que o Rust pode ser esquecido. Os dois riscos foram vistos e empurrados para depois; eu os resolveria agora. O que piorou: as trilhas do projeto não têm nenhuma lição pronta (seção 4), e pausar a trilha no fechamento derruba a revisão espaçada (seção 5).

---

## 2. Escopo e realismo

### Quanto tempo leva

Somei os minutos declarados na Trilha para cada etapa e módulo citados no "Estudar antes" do ROADMAP §6, sem contar duas vezes, a partir do `/api/projeto`:

| Bloco | Estudo declarado |
|---|---|
| Fase 0 | 36 h |
| Fase 1 | 64 h |
| Fase 2 | 41 h |
| Fase 3 | 55 h |
| Fase 4 | 35 h |
| Fase 5 | 64 h |
| Fase 6 | 35 h |
| Rust R1–R4 | 35 h |
| Fase 7 | 41 h |
| Fase 8 | 17 h |
| Fase 9 | 14 h |
| **Total** | **~437 h** |

Isso ainda deixa de fora boa parte do §7.1 (cursos como T24, T29 e T32) e subestima o real. A trilha 36 promete Rust do zero até unsafe, FFI e Tokio em ~40 h, e o "Projeto: mini KV com WAL" da T37 tem 90 minutos. Multiplique por 1,5 a 2.

A construção pesa mais que o estudo. Minha estimativa para alguém que aprende Go no caminho:

| Tipo de item | Exemplos | Horas por item |
|---|---|---|
| Pequeno | 0.1, 0.2, 6.3 | 3–15 |
| Médio | 1.1, 1.4, 1.5, 3.7, 7.7 | 15–30 |
| Grande | 2.3, 2.4, 3.2, 3.3, 5.3, 5.4, 7.4 | 30–60 |
| Enorme | 5.5, 5.8, 6.1, 6.2, 8.2 | 60–150 |

O total fica entre **1.500 e 2.500 h de construção**, mais **650–900 h de estudo**, mais **15–25% do ciclo** (checagem, caminhos, registro, abrir e fechar sessão). Isso dá **~2.400 a 4.000 h**.

A 1,5 h por dia, 6 dias por semana e 46 semanas por ano, são ~415 h por ano: **de 6 a 9 anos**. A 2 h por dia, todos os dias, são ~650 h por ano: **de 4 a 6 anos**. Mesmo cortando a minha estimativa pela metade, passa de 3 anos.

**Conclusão: é coisa demais para um projeto só, com um fim só.** O fim declarado (ROADMAP §9b: o 9.2 fechado e os três livros lidos) está a anos de distância, e não há nenhum fim intermediário.

### O que eu cortaria ou adiaria sem perder o essencial

| Item | O que fazer | Por quê |
|---|---|---|
| **Fase 8 inteira** (8.1 chat, 8.2 SFU, 8.3 live) | Cortar ou deixar como bônus | Mídia em tempo real é um nicho. Para um engenheiro backend ou de plataforma, rende pouco pelo esforço. O 8.1 (WebSocket com pub/sub entre instâncias) é o único que vale, e cabe como um item pequeno da Fase 7 |
| **7.6 vídeo** | Reduzir a upload → fila → ffmpeg → HLS no object storage, sem CDN própria | A lição de backend é o processamento assíncrono, não a mídia |
| **7.5 e 7.7** | Virar katas, não itens | São exercícios de algoritmos que não pedem decisão de arquitetura |
| **5.3 mini S3** | Usar MinIO e estudar a URL assinada | Ser compatível com `aws s3 cp` exige implementar a assinatura SigV4, que é um buraco de semanas com pouco aprendizado de arquitetura |
| **5.2 cofre** | Usar OpenBao ou Vault e implementar só o envelope encryption numa função | O conceito cabe em 50 linhas; o produto, não |
| **2.4 CA e ACME próprios** | Usar step-ca ou a CA interna do Caddy e aprender PKI configurando, mais o mTLS do 3.2 | Escrever um ACME ensina o protocolo ACME, não arquitetura |
| **5.4 IAM** | Manter pequeno (avaliar políticas com OPA ou Cedar, ou um avaliador mínimo) | O núcleo interessante é a avaliação de política, não o cadastro |
| **5.5 escalonador próprio** | Trocar pelo escalonador do Kubernetes e um operator (ver a seção 3) | É mais vendável e ensina o mesmo loop de reconciliação |
| **R3, a parte de C++** | Cortar | C++ não está nos seus objetivos. A comparação Go × Rust basta |
| **Escola (Fase 7)** | Manter só 7.1, 7.3 e 7.4 | O valor está no segundo domínio, no webhook entre empresas e no sandbox. O resto repete o que o Banco já ensinou |

Com esses cortes, o projeto cai para algo como 1.500–2.000 h. Continua grande, e por isso a mudança 1 da seção 8 divide o projeto em temporadas.

---

## 3. Ordem e progressão

### O que está bem
- **A Fase 0 antes de tudo** e o "primeiro do jeito errado" do 0.4 estão certos.
- **O 1.3 comparando `FOR UPDATE` com lock otimista e medindo** é exatamente decidir por experimento.
- **O 4.2 (event sourcing como experimento com ADR)** está certo porque "não adotar" é uma resposta possível.

### O que está fora de lugar

1. **O 1.2 (hexagonal) vem antes da dor.** ROADMAP §4, linha A1: "a dor das camadas acopladas aparece em 1.1", mas a dor real é trocar a memória pelo Postgres, que só acontece no 1.3. Do jeito que está, você refatora para hexagonal por instrução. **Inverta: 1.1 → 1.3 (Postgres direto no handler) → 1.2 (extrair as portas porque ficou ruim de testar).** Assim a pressão é real e o ADR do 1.2 tem um "antes" para comparar.
2. **A Fase 2 (Nuvem) antes da Fase 3 (PIX).** O motivo escrito ("o Banco precisa de uma casa melhor que um compose feito à mão", ROADMAP §6, Fase 2) não se sustenta: um compose com Caddy atende bem. Enquanto isso, a Fase 3 é "a mais rica para System Design e DDD" (palavras do próprio ROADMAP) e é o que mais casa com os seus objetivos de system design e dados. **Faça 0 → 1 → 3 → 4.5 e 4.6 → Nuvem.**
3. **Kubernetes no 5.8 é tarde demais para quem quer plataforma.** Hoje ele vem depois de ~25 itens. Proposta:
   - Nuvem v0.1 enxuta: 2.1 minicontainer, 2.2 agente e 2.5 git push, num nó só.
   - Logo depois, o Kubernetes (5.8).
   - A Nuvem v0.2 vira **um operator no Kubernetes**: uma CRD `App` e controladores que provisionam o banco, os segredos e o domínio.

   É o que times de plataforma fazem de verdade (uma plataforma interna em cima do Kubernetes). Você aprende o loop de reconciliação (5.7) no código de um controlador, que é o projeto da própria T41 › Orquestração por dentro.
4. **O tracing no 5.6 chega tarde.** O sistema fica distribuído no 3.2 (gRPC entre bancos e SPI). OpenTelemetry tem que entrar no 3.2 ou no 3.3, porque a saga sem trace é depurada no escuro.
5. **O teste de carga no 4.5 chega tarde.** Sem uma linha de base, nenhum gatilho é medido. Ponha um k6 mínimo no 1.7, junto do SLO. Daí em diante, o gatilho pode ser "o SLO quebrou sob X req/s".
6. **O CI não está em nenhum item.** Ponha no 0.1, porque `go test -race`, o lint e o depguard do 1.2 são as fitness functions da METODOLOGIA §5.4 e só protegem algo se rodarem sozinhos.
7. **O blue/green do 2.7 ignora a migration do banco.** Esse é o ponto difícil de um deploy sem downtime: mudar o esquema em expandir e contrair com duas versões rodando. A T37 já tem a etapa (› Operação › Migrations sem downtime e "Projeto: migração com carga rodando"), mas nenhum item do ROADMAP a usa, exceto o 5.1 de passagem.

### A pressão A0–A9 é crível?

Em parte. Passei estágio por estágio:

| Estágio | A pressão é real? |
|---|---|
| A0 → A1 | Quase. Fica real se o 1.3 vier antes do 1.2 (item 1 acima) |
| A2 monólito modular | Fraca. O Banco já nasce com módulos; é mais nomear do que evoluir |
| **A3 distribuído por organização** | **Real.** Dois bancos não compartilham transação |
| A4 extrair a antifraude | Inventada. Num homelab não há escala. A fitness function "antifraude fora do ar não impede o PIX" se resolve **dentro** do monólito, com chamada assíncrona e um bulkhead |
| A5 plataforma | Inventada. É objetivo de estudo, não pressão |
| A6 Kubernetes | O próprio ROADMAP admite que não há pressão |
| A7 broker e KV próprios | Exercício puro |
| A8 Escola | Real como segundo domínio |
| A9 | É fechamento, não evolução |

Exercício de propósito não é problema. O problema é **chamar de pressão** e ter a decisão já tomada no roadmap. O 4.4 diz "um serviço `antifraude` **extraído**" no "Implementar". Se o resultado está escrito antes da fase de caminhos, a fase de caminhos é teatro e treina viés de confirmação, o oposto do que um arquiteto precisa.

**Proposta:**
- Marque cada estágio como **pressão medida** (o gatilho é um teste ou um SLO que falha) ou **exercício deliberado**.
- Nos exercícios, escreva o "Implementar" como pergunta: "a antifraude deve sair do monólito?". O ADR pode concluir que não; nesse caso, você faz a extração num branch para aprender e registra por que ela não ficou.

---

## 4. Lacunas e excessos

### O que falta para um engenheiro backend e de plataforma sênior

| Lacuna | Onde entraria |
|---|---|
| **Log de auditoria imutável.** Num banco, quem fez o quê e quando é requisito, não detalhe | 1.5 ou 3.x |
| **Design de API e evolução de contrato:** versionamento, paginação, erros no formato RFC 9457, OpenAPI e compatibilidade para trás | 1.1 e 3.2 |
| **Migrations sem downtime** (expandir e contrair) | 2.7 |
| **Segurança como prática:** modelagem de ameaças (STRIDE) no 3.2 e no 7.4, OWASP API Top 10 no 1.5, cadeia de suprimentos (SBOM, assinatura de imagem com cosign, varredura de dependências) no 2.5 | itens citados |
| **OAuth2 e OIDC.** A "API de cobrança para lojistas" (7.3) é o caso clássico de client credentials, e o roadmap só tem HMAC | 7.3 |
| **Backpressure e descarte de carga:** controle de admissão, deadline propagado, fila limitada. O 3.5 cobre retry e breaker, mas não o lado de quem recebe | 3.5 |
| **Incidente, plantão e postmortem como prática.** A T41 › SRE tem, mas nenhum item usa. Um **game day** depois do 3.6 é barato e ensina muito | depois do 3.6 |
| **Kafka de verdade.** O roadmap vai do NATS ou RabbitMQ (3.4) direto para o broker próprio (6.1). O mercado usa Kafka. Usar o Redpanda ou o Kafka no 3.4 também torna o 6.1 uma comparação real ("o meu log contra o deles") | 3.4 |
| **Ler código de produção.** Para cada "faça o seu", leia o núcleo do original: `etcd/raft` antes do 6.2, o log do Kafka antes do 6.1, `net/http/httputil` antes do 2.3. Engenheiro sênior lê mais código do que escreve | 6.1, 6.2, 2.3 |
| **Prática de sistemas distribuídos em pequeno.** Os desafios *Gossip Glomers* (Fly.io com o Maelstrom, em Go) são katas de broadcast, contadores, log de Kafka e transações, testados com partição de rede. Encaixam antes do 6.1 e do 6.2 e custam dias, não meses | antes do 6.1 e do 6.2 |
| **System design no papel.** Construir cobre poucos sistemas, e fundo. A amplitude vem de desenhar muitos: um design de 45 minutos por semana (encurtador de URL, feed, rate limiter global, pagamento) com a T15 e a T18 | faixa semanal |
| **Design doc por fase**, não só no 9.2 | fim de cada fase |

**Bancos de dados a fundo é um objetivo declarado seu e está sub-representado no ROADMAP.** A T37 é boa (EXPLAIN, bloat, pool, particionamento, PITR, mini KV com WAL), mas os itens só puxam parte dela. Proposta: um item "4.x O extrato com 10 milhões de lançamentos" que junta particionamento por mês, índices, EXPLAIN ANALYZE e a comparação com o modelo CQRS do 4.1.

Repare também num conflito pequeno que vale como decisão: a T37 › Tipos e restrições manda usar "numeric para dinheiro", e o 0.2 usa `int64` em centavos. É um bom "caminhos" para o 1.3; só não deixe a lição gerada decidir por você.

### O que sobra
- **Já listei os cortes na seção 2.** Somo dois pontos.
- **Repositórios demais.** O ROADMAP §2 cria oito ou mais (`banco`, `spi`, `nuvem`, `escola`, `resiliencia`, `minicontainer`, `fila`, `kv`, além de `rust-estudos`), cada um com CI, README, licença e ADRs. A regra do próprio §2 ("separe só com motivo concreto") vale para repositórios também, e o motivo 1 (outra organização) é ficção. Um monorepo com `go.work` e pastas por "empresa" mantém a narrativa e corta o custo fixo. Separe quando publicar algo que outros vão importar, como a `resiliencia`.
- **Material demais.** São 39 trilhas, das quais 13 estão ativas, e cursos com 300 (T24) e 247 (T25) etapas. O ROADMAP §5 diz que a IA escolhe a melhor fonte, mas o catálogo enorme tem um custo: cada item vira uma pesquisa de fonte.

### O "✔ você tem e cobre bem" não é verdade hoje

ROADMAP §5 define ✔ como "você tem e cobre bem". Consultei a API:

| Trilhas | Etapas | Etapas com lição | Etapas com material |
|---|---|---|---|
| 9, 11, 12, 13, 15 | 204 | 0 | 0 |
| 22, 36, 37, 38, 41, 42 | 146 | 0 | 0 |
| 8 | 98 | 0 | 38 |

O que existe é **um título, um objetivo e um critério**. O conteúdo depende de lições geradas sob demanda. Isso tem dois custos:
- **Qualidade.** Para assuntos de fundo, lição gerada sem fonte (a regra `criar-trilha.md`, lição 14, deixa o C9 falhar de propósito) é a fonte menos confiável que você poderia escolher. Para Rust, *The Rust Programming Language* (gratuito) com o Rustlings é melhor e mais barato que 35 lições geradas. Para Postgres, a documentação oficial e *The Internals of PostgreSQL*. Para Raft, o paper.
- **Tempo e tokens.** Gerar e revisar centenas de lições é trabalho que não é estudo.

Troque a legenda para algo honesto ("roteiro pronto, conteúdo a gerar") e prefira fonte primária nos itens de fundo.

Pequenos achados nas trilhas:
- As instruções das trilhas 21 a 35 apontam para `projetos/ROADMAP.md` e usam o nome antigo "Projeto infinito"; o caminho atual é `projeto-de-estudo/ROADMAP.md`.
- 101 das 361 referências do ROADMAP apontam para **módulos inteiros** (ex.: "T25 › 10 Docker"). Por isso o 1.6 (Dockerfile, slog e encerramento gracioso) soma ~24 h de estudo declarado, e o 5.8 ~19 h.

---

## 5. Metodologia

### O ciclo ensina ou vira burocracia?

**Ensina a decidir, e isso é raro e valioso.** "Estudar porque precisa decidir" (METODOLOGIA §1) e "o gatilho de evolução" (§3.5) são as melhores ideias do projeto.

**Vira burocracia em três lugares:**

1. **Itens sem decisão real.** O 0.1 (criar o repositório) passaria por "caminhos 2 a 4" e "argumentação". O CLAUDE.md manda "não pule fase sem o Lucas pedir", então a cerimônia é o padrão. O P1 previu o "modo curto" como gatilho; crie agora. Proposta de dois tipos de item, marcados no próprio ROADMAP:
   - **Item de prática:** problema → construir → conferir → fechar.
   - **Item de decisão:** o ciclo inteiro.

   Pela minha leitura, uns 15 dos 60 itens são de decisão.
2. **Estudo pesado antes de tocar no código.** O 1.6 tem ~24 h de estudo declarado antes de um Dockerfile. A 1,5 h por dia, são três semanas lendo antes de codar, e é aí que a motivação morre. Ponha um teto de estudo antes de começar (algo como 25–35% do tempo do item) e deixe o resto "sob demanda" na execução.
3. **A execução passo a passo com a IA conferindo cada passo** (§3.7) transforma cada sessão numa conversa e aumenta o custo de abrir e fechar. Faça como num time: você executa um bloco do plano sozinho, abre um "PR" (o diff) e a IA revisa como revisora de código. É mais realista e treina autonomia.

### Comparação com técnicas conhecidas

| Técnica | Como está | O que acrescentar |
|---|---|---|
| **Prática deliberada** (no limite da habilidade, com feedback e **repetição**) | Tem feedback (a IA confere). Quase não tem repetição | **A METODOLOGIA §4 remove a repetição de propósito:** se você fez "quase igual sozinho", a IA oferece adiantar. Mas é na segunda e na terceira vez sozinho que a habilidade fixa. Adiante só no nível "domino" do REPERTORIO, não no "sozinho" |
| **Recuperação ativa** | A checagem (§3.3) vem logo depois do estudo, quando tudo está fresco. Isso mede reconhecimento de curto prazo | **Abra toda sessão com 3 perguntas de itens de 2 a 6 semanas atrás, sem consulta.** Leva 5 minutos e é a intervenção de maior efeito comprovado da lista |
| **Revisão espaçada** | Existe na Trilha, mas **o P7 a desliga**. O código do app (`home-lab-trilha-estudo/backend/dominio.go:148`) só mostra revisões de trilhas com `estado = 'ativa'`. Pausar a trilha no fechamento do item, como manda o P7, some com as revisões exatamente quando elas mais importam. Além disso, a revisão usa as perguntas dos exercícios, e sem lição não há exercício | Não pause trilha com revisão pendente (ou mude a consulta para incluir revisões de trilhas pausadas). No fechamento, gere de 3 a 5 perguntas **do item** (das decisões e dos erros da checagem) e mande para a revisão espaçada ou para o Flashcards |
| **Técnica de Feynman** | Ausente. A IA escreve o caderno, o DECISOES e o REPERTORIO (CLAUDE.md, "O que registrar") | **Você escreve o ADR e o "O que aprendi" do fechamento,** de 150 a 300 palavras, sem IA, como se explicasse a um júnior. A IA só critica. Escrever é onde a lacuna aparece |
| **Projeto guiado** | É o forte do plano | Mantenha |
| **Katas** | Ausentes | **Uma sessão por semana sem IA:** um Gossip Glomers, um exercício do Exercism (Go e depois Rust) ou refazer de memória o rate limiter ou o LRU. Dá fluência e mede a independência |
| **Falha produtiva** (tentar antes de estudar) | Só no 0.4 | Nos itens de decisão, 30 a 45 minutos de "tentativa ingênua" antes do estudo. O estudo depois rende mais porque você sabe o que procurar |
| **Intercalação** | O roadmap mistura trilhas, o que é bom | Mantenha |

### Onde o ciclo trava, na prática
- **"Já sei isso" pula para a checagem** (§3.2), mas a checagem é a própria IA perguntando. Sem um critério objetivo, vira autoavaliação. Use o critério da etapa da trilha (que é verificável) como prova.
- **"Tudo tem resposta certa para o cenário" e "vantagens e desvantagens não são opinião"** (METODOLOGIA §3.4 e §7; CLAUDE.md, "Como conduzir"). Muita decisão de arquitetura é julgamento sob incerteza. Essa regra dá autoridade demais à IA, que erra com confiança. Troque por: "a IA diz certo ou errado **e cita a fonte** (capítulo, documentação ou um experimento); quando for julgamento, diz que é julgamento".

---

## 6. A IA como mentora

### Os riscos, do maior para o menor

1. **A IA decide por você.** O fluxo da METODOLOGIA §3.4 é: a IA apresenta os caminhos, você argumenta, e a IA "diz qual é o melhor". Com uma IA articulada, você vai concordar quase sempre, e o caderno vai registrar "consenso". **Correção:** escreva a sua decisão e o porquê **no caderno, antes** de a IA dar a recomendação dela. Depois compare. A taxa de divergência e quem estava certo viram métrica.
2. **Dependência na execução.** A METODOLOGIA §3.7 diz "quanto a IA escreve é decidido na hora, pelo Lucas". Às 23 h, cansado, a resposta vai ser "escreve você". **Correção:**
   - uma escada de ajuda fixa: dica → pergunta guiada → pseudocódigo → código;
   - cada degrau usado vai para o caderno;
   - "% do núcleo escrito por mim" vira métrica no REPERTORIO;
   - mais as katas sem IA da seção 5.
3. **Você não escreve.** Coberto acima (Feynman). Se a IA redige o caderno, o ADR e o repertório, os documentos ficam bons e você aprende menos.
4. **Perder o fio entre sessões.**
   - **O ESTADO** é atualizado "ao fim de toda fase" (METODOLOGIA §2). Mas a execução de um item grande dura semanas, e uma sessão que termina no meio da fase deixa o ESTADO velho. Atualize **ao fim de toda sessão**.
   - **O caderno do item** vai crescer até virar transcrição. Ponha um limite (uma ou duas telas por fase) e um "Onde paramos" sempre no topo, que o `_MODELO.md` já tem.
   - **Os gatilhos registrados** no DECISOES nunca são conferidos por ninguém. O CLAUDE.md devia mandar a IA, a cada fechamento, ler os gatilhos abertos e dizer se algum disparou.
5. **Dependência da infraestrutura.** O progresso do estudo mora na Trilha, no homelab, que teve problemas de disco e de rede este mês. Deixe a metodologia funcionar sem a API: o ESTADO, que está no git, é a fonte mínima.

### O CLAUDE.md

**Bom:** curto, "confira de verdade", "não escreva o núcleo", commits sem coautoria, nada de segredo.

**O que mudar:**
- **Falta um protocolo de sessão.** Abrir com 3 perguntas de recuperação, combinar o objetivo da sessão (o que fica pronto hoje), fechar com um resumo de 3 linhas e o próximo passo no ESTADO.
- **"Recomende sempre o melhor caminho, mas deixe o Lucas argumentar primeiro"** precisa virar "deixe o Lucas **registrar a decisão** primeiro".
- **"Na checagem e nos caminhos, diga certo, errado ou incompleto"** precisa de "cite a fonte; se for julgamento, diga".
- **"Antes de cada passo, olhe o REPERTORIO e ofereça adiantar"** devia exigir o nível "domino".
- **O P7 dá à IA o poder de ativar e pausar trilhas na Trilha.** Além do problema da revisão espaçada, ativar a trilha **inteira** joga no Hoje etapas que o item não usa (a T37 tem 34 etapas, e o 1.3 usa umas 16). Ative só as etapas do item, ou não deixe a IA mexer.
- **Falta a regra para material novo não inflar o roadmap.** A METODOLOGIA §8 manda transformar o que é novo e cabe em "item novo". Sem limite, o roadmap só cresce. Ponha um estacionamento ("depois") e a regra de "um entra, um sai" dentro da temporada.
- **O PROMPT.md repete a metodologia em 8 linhas.** São duas cópias que vão divergir. O Claude Code já carrega o CLAUDE.md da pasta, então o prompt só precisa do item, da fase e do que mudou desde a última sessão.

### O DECISOES.md

O DECISOES.md não segue a própria regra. A METODOLOGIA §3.5 exige "o caminho escolhido e **por quê**" e "os descartados e por quê". A tabela do DECISOES tem só "Decisão" e "Gatilho": o P2, o P4, o P5 e o P7 não dizem por quê nem o que foi descartado.

Contestando uma por uma:

| Decisão | Minha posição |
|---|---|
| **P1** ciclo de 8 fases para tudo | **Discordo do "para tudo".** Crie os dois tipos de item agora (seção 5), não quando "ficar pesado" |
| **P2** monólito modular | **Concordo.** Mas aplique a mesma regra aos repositórios (seção 4) |
| **P3** caderno separado do código | **Concordo.** Um efeito colateral: o caderno é público, e nele ficam os seus erros na checagem. Se isso inibir a honestidade das respostas, deixe a seção 3 dos cadernos fora do repositório público |
| **P4** Rust em paralelo, reescrevendo o que já fez em Go | **A ideia de reescrever o que já conhece é ótima. O espaçamento não é.** De R1 ("depois da Fase 1") a R4 ("depois da Fase 5") passam, pelas minhas contas, de 2 a 3 anos. Rust esquecido entre um e outro é quase certo, e o próprio gatilho do P4 admite. Duas saídas melhores: uma sessão fixa de Rust por semana, ou um **bloco concentrado** de 6 a 8 semanas quando o Go estiver fluente (depois da Fase 1 ou da 3), seguido de katas mensais. Para uma linguagem difícil, eu prefiro o bloco |
| **P5** frontend só no que o backend exige | **Concordo.** Vou além: sem painéis próprios. Grafana e a CLI bastam. Interface só onde o backend exige, como a página do WebRTC |
| **P6** inglês no código | **Concordo,** pelo portfólio. Os termos do PIX em português fazem sentido |
| **P7** trilhas do item no Hoje | **Discordo como está.** Mata a revisão espaçada e enche o Hoje (ver acima). E hoje há **13 trilhas ativas, 11 delas sem nenhuma etapa concluída** (4, 8 a 15 e as de português e inglês). Pause já tudo o que não for do item atual |
| **P8** desenvolver no Mac, Linux no Proxmox | **Concordo em parte.** Para os itens que pedem Linux num nó só (2.1 e 7.4), uma VM Linux local (Lima, OrbStack ou UTM, arm64) dá um ciclo bem mais rápido que editar no Mac e rodar no Proxmox. O Proxmox fica para vários nós (5.5 e 5.8). Atenção: o Mac é arm64 e o M920q é x86. As imagens do 1.6 e do 2.5 precisam ser multi-arch, e isso deveria estar no critério de pronto. Confira também se a RAM do M920q comporta 3 VMs de Nuvem mais 3 de Kubernetes mais as réplicas do Postgres ao mesmo tempo |

---

## 7. Livros

**Capítulo solto quando o item pede funciona bem para o Khononov, razoavelmente para o Richards e Ford, e mal para o Kleppmann.**

- **Kleppmann (T22) é cumulativo.** A replicação (cap. 6) é a base do particionamento (7), das transações distribuídas e do consenso (10). O ROADMAP §7.2 manda ler o **cap. 10 no 3.3**, o **cap. 6 no 4.6** e o **cap. 7 no 6.3**: consenso antes de replicação. O cap. 9 é dividido entre o 1.4 e o 3.3. E o 1.3 empilha os capítulos 3, 4 e 8 (~150 páginas densas) num item só.
- **Khononov (T23):** a ordem 5–6 (no 0.2) antes de 1–2 (no 0.3) é só inverter. O resto funciona.
- **Richards e Ford (T21):** os caps. 4 a 6 (características arquiteturais) só no 1.7. São justamente o vocabulário para dizer **o que** pressiona a arquitetura, então deveriam vir no 0.1, junto do 1, 2 e 21.
- **O fio de cada livro se perde.** Os últimos capítulos ficam todos no 9.2 (T21 22–27, T22 13–14, T23 16–17), ou seja, daqui a anos. E o fim do projeto depende disso (ROADMAP §9b).

**Proposta:** leia os três **em ordem, numa faixa fixa e curta** (por exemplo, 3 sessões de 30 minutos por semana). No ritmo de um capítulo por semana, os três saem em cerca de um ano. Os itens passam a citar o capítulo como **releitura aplicada**, que é revisão espaçada de graça: a primeira passada dá o mapa, a segunda (no item) dá a profundidade. E o fim do projeto deixa de depender de chegar ao 9.2.

---

## 8. Motivação e fim

**O que mais me preocupa,** dito com respeito e sem rodeio: o histórico do git mostra que o roadmap chegou à v3 e ganhou uma metodologia num dia só (21/11), e a plataforma já tem a aba do projeto. Ao mesmo tempo, a API mostra **8 etapas concluídas em 39 trilhas**, e as trilhas 8 a 15, ativas, estão em 0%. Construir a ferramenta e o plano é gostoso e parece progresso. O risco número 1 é este, não o Raft.

**Regra que eu adotaria: nenhuma mudança no plano ou na plataforma até o 1.3 estar fechado,** exceto as desta revisão que você aceitar.

**Para não abandonar:**
- **Temporadas com fim próprio** (mudança 1). Cada uma termina num resultado que você mostraria a alguém, e parar no fim de qualquer temporada é um projeto concluído, não abandonado.
- **Entregas visíveis por fase:** uma tag de versão (`banco v0.1`), uma demo gravada de 3 minutos e um texto curto e público ("como fiz um razão de partidas dobradas em Go que nunca perde um centavo"). O texto é também o Feynman.
- **Métricas de progresso, poucas e honestas, revistas toda semana:**

  | Métrica | Por quê |
  |---|---|
  | Horas por semana (registradas, não estimadas) | Sem isso, nenhuma previsão vale |
  | Itens fechados por mês | É a velocidade real |
  | % do núcleo escrito por você | É a métrica de dependência |
  | Acertos nas 3 perguntas de abertura | É a retenção |
  | Dias desde o último commit em `codigo/` | É o alarme de "virou planejamento" |

- **Uma retrospectiva por mês, de 30 minutos:** o que travou, o que cortar, e o roadmap da temporada **encolhe** se a velocidade pedir.
- **Uma sessão mínima que conta:** nos dias ruins, 20 minutos (uma kata ou um capítulo) mantêm a sequência. Sequência quebrada é o começo do abandono.

---

## 9. As 10 mudanças, em ordem de impacto

### 1. Dividir em temporadas de 6 a 10 meses, cada uma com fim próprio, e cortar a Fase 8
**Por quê:** de 4 a 9 anos com um fim só não se sustenta, e um fim a cada temporada dá motivo para continuar.

**Proposta:**
- **T1:** Fases 0 e 1, mais o CI → "Banco v0.1 no homelab com SLO".
- **T2:** Fase 3, mais o 4.5 e o 4.6 → "PIX entre dois bancos sob caos, conciliando".
- **T3:** Nuvem enxuta, Kubernetes e um operator → "Banco no Kubernetes com GitOps".
- **T4:** `fila` e `kv`, mais o bloco de Rust.
- **T5 (opcional):** Escola reduzida (7.1, 7.3, 7.4) e AWS.

**Onde:** ROADMAP §6 (fases), §9b (um fim por temporada), §4.

### 2. Escrever é com você: ADR, "O que aprendi" e a decisão antes da IA
**Por quê:** é a mudança que mais aumenta o aprendizado por hora e a que mais reduz o risco de a IA decidir.

**Onde:** METODOLOGIA §3.4, §3.5 e §3.8; CLAUDE.md, "Como conduzir" e "O que registrar"; `itens/_MODELO.md` (seção 5 com "Minha decisão antes da recomendação"; seção 8 escrita pelo Lucas).

### 3. Recuperação tardia e conserto da revisão espaçada
**Por quê:** hoje nada traz de volta o que foi aprendido há um mês, e o P7 desliga as revisões ao pausar a trilha (`dominio.go:148` filtra `estado = 'ativa'`).

**O que fazer:**
- 3 perguntas de itens antigos no começo de toda sessão;
- as perguntas do item vão para a revisão espaçada no fechamento;
- não pausar trilha com revisão pendente.

**Onde:** METODOLOGIA §3.3 e §3.8; DECISOES P7; CLAUDE.md, "Trilhas no Hoje".

### 4. Dois tipos de item e um teto de estudo antes de codar
**Por quê:** é o que impede o ciclo de virar burocracia e as três semanas de leitura antes do primeiro Dockerfile (1.6).

**O que fazer:**
- marcar no ROADMAP cada item como "prática" ou "decisão";
- estudo prévio limitado a ~1/3 do item;
- 30 a 45 minutos de tentativa ingênua antes do estudo nos itens de decisão.

**Onde:** METODOLOGIA §3 e §3.2; DECISOES P1; ROADMAP §5 e §6.

### 5. Reordenar
**O que fazer:**
- o 1.3 antes do 1.2;
- a Fase 3 antes da Fase 2;
- o Kubernetes logo depois da Nuvem v0.1, e a Nuvem v0.2 como operator;
- OpenTelemetry no 3.2;
- k6 de base no 1.7;
- CI no 0.1;
- migrations sem downtime no 2.7.

**Por quê:** cada troca transforma um passo prescrito em pressão real, ou põe a ferramenta onde o problema aparece.

**Onde:** ROADMAP §4 (A1, A5, A6) e §6 (0.1, 1.2, 1.3, 1.7, 2.7, 3.2, 5.5–5.8).

### 6. Trocar as reinvenções periféricas por peças prontas e aprofundar as três que importam
**O que fazer:**
- step-ca no lugar da CA própria, OpenBao no lugar do cofre, MinIO no lugar do S3, OPA ou Cedar no IAM;
- manter o `minicontainer`, a `fila` e o `kv`, cada um com a leitura do código original e os Gossip Glomers antes;
- Kafka ou Redpanda no 3.4.

**Por quê:** concentra o esforço no que ensina arquitetura e dados, e aproxima do que o mercado usa.

**Onde:** ROADMAP §2 (tabela dos repositórios), §6 (2.4, 3.4, 5.2–5.4, 6.1, 6.2).

### 7. Livros em ordem, numa faixa fixa, com os itens como releitura
**Por quê:** conserta a ordem quebrada do Kleppmann, mantém o fio de cada livro e tira o fim do projeto da dependência do 9.2.

**Onde:** ROADMAP §3 ("Os livros no projeto"), §7.2 e §9b; METODOLOGIA §3.8 e §8.

### 8. Anti-dependência explícita
**O que fazer:**
- a escada de ajuda registrada no caderno;
- adiantar só no nível "domino";
- uma sessão por semana sem IA;
- a execução em blocos com revisão tipo PR;
- o ESTADO ao fim de **toda sessão**.

**Por quê:** o objetivo é você conseguir fazer sem a IA, e hoje nada mede isso.

**Onde:** METODOLOGIA §2, §3.7 e §4; CLAUDE.md; REPERTORIO (coluna "% do núcleo meu").

### 9. Fonte primária nos itens de fundo e uma legenda honesta
**O que fazer:**
- o Rust Book e o Rustlings no lugar das lições geradas da T36;
- a documentação do Postgres e *The Internals of PostgreSQL* na T37;
- o paper do Raft no 6.2;
- trocar o ✔ por "roteiro pronto, conteúdo a gerar" enquanto não houver lição;
- corrigir as instruções das trilhas 21 a 35 (`projetos/ROADMAP.md` e "Projeto infinito").

**Por quê:** hoje há 0 lições nas trilhas do projeto, e conteúdo gerado é a fonte menos confiável para assunto de fundo.

**Onde:** ROADMAP §5 (legenda) e §8 ("O que ainda é raso"); trilhas 21–37 na Trilha.

### 10. Ritmo, foco e um congelamento do plano
**O que fazer:**
- pausar já as trilhas ativas que não são do item atual (hoje são 13);
- registrar as horas;
- uma retrospectiva por mês que pode encolher a temporada;
- uma entrega pública por fase;
- "um entra, um sai" para material novo;
- **não mexer no plano nem na plataforma até fechar o 1.3**.

**Por quê:** o risco número 1 é o planejamento continuar ocupando o lugar da construção.

**Onde:** METODOLOGIA §8; ROADMAP §10; ESTADO (horas e próxima retrospectiva); DECISOES (registrar a regra do congelamento).

---

### Pequenos ajustes que não entraram nas 10
- **CI contraditório:** ROADMAP §2 diz "lacuna, ver a seção 8"; a §8 diz coberta; nenhum item implementa.
- **DECISOES sem o porquê:** acrescente as colunas "Por quê" e "Descartados", como a METODOLOGIA §3.5 exige.
- **2.4 no Samsung:** "com a CA instalada" funciona no Chrome. Apps nativos do Android ignoram CA de usuário por padrão, o que não afeta este projeto, mas vale saber se um dia testar por app.
- **`numeric` × `int64`:** registre como caminhos do 1.3 (seção 4).
- **R2 (leitor de CSV do extrato):** é bom como exercício e fraco como motivação. Se mantiver o Rust em bloco, troque por algo que o Banco use, como um validador do arquivo de conciliação do 3.6.
