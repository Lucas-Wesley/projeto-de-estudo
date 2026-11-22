# Crítica do Projeto de estudo — SWE-2 Max

Revisão pedida em `revisoes/PEDIDO-DE-CRITICA.md`. Escopo lido: `ROADMAP.md` (v3),
`METODOLOGIA.md` (v1), `DECISOES.md` (P1–P8), `CLAUDE.md`, `ESTADO.md`, `REPERTORIO.md`,
`PROMPT.md`, `itens/_MODELO.md`, `README.md`, as trilhas 22, 36, 37, 40 e 44 pela API,
`GET /api/projeto`, a lista completa de trilhas e `regras/criar-trilha.md`.

Tom pedido: sem elogio. Então deixo claro de saída — o plano tem decisões genuinamente boas
(domínio com partidas dobradas, K8s depois do orquestrador próprio, Rust por reescrita,
observabilidade cedo, "estudar porque precisa implementar"). O que segue é o que está errado,
o que falta, o que sobra e o que melhora o processo.

---

## 0. Minha leitura inicial (só ROADMAP + METODOLOGIA)

O que anotei antes de abrir o resto:

1. **É um currículo de 2+ anos vestido de "projeto".** O documento evita dizer isso.
2. **Cerimônia uniforme.** O mesmo ciclo de 8 fases para "criar `go.mod`" e para "implementar Raft".
3. **A0–A9 é elegante até A4 e cada vez mais ficcional depois.**
4. **Livro por capítulo solto é a parte mais frágil estruturalmente.**
5. **Não há marcos visíveis nem métricas — só uma linha de chegada a anos de distância.**

Depois de ler DECISOES, CLAUDE.md e a API: dois pontos melhoraram (a plataforma existe de
verdade e lê o caderno; os gatilhos P1/P4/P7 mostram que alguém previu os problemas) e um
piorou muito: **as trilhas T36–T44, que fecham "todas as lacunas", estão vazias** — 200+
etapas sem material e sem lição. O "não há mais lacunas" da seção 8 é otimista: não há lacuna
de *roteiro*; há uma lacuna enorme de *conteúdo*.

---

## 1. Escopo e realismo

**É coisa demais, e o plano sabe disso — o nome original no git era "Projeto infinito".**

Contas com dados reais:

- **60 itens** de roadmap (56 nas fases 0–9 + R1–R4).
- As trilhas do projeto (T21–T44) somam **657,6h de etapas cadastradas** pela própria API;
  todas as trilhas juntas, **~1.154h**. A T8 (Redes) sozinha: 98 etapas, 143h.
- Os 3 livros: 3.880 min (~65h) de leitura guiada cadastrada — e isso não conta reler,
  anotar e a checagem.
- As etapas dos livros estimam 60–110 min por capítulo. O cap. 8 do Kleppmann (transações,
  isolamento, 2PL, SSI) em 100 minutos é leitura corrida, não estudo.

Estimativa honesta por item: estudo (3–12h) + construção (3–15h, com picos de 30–60h nos
itens 5.5, 5.8, 6.1, 6.2, 7.4, 8.2, 9.1) + overhead do ciclo e do caderno (1–2h).
**Total: 1.000–1.500h. A 10h/semana: 2 a 3 anos. A 7h/semana: 3 a 4,5 anos.**

O problema não é a duração em si — um plano de estudo de 3 anos pode ser legítimo. O problema
é que ele é **apresentado como um único projeto com uma única linha de chegada** (§9b: item
9.2 + 3 livros). Nada entre hoje e daqui a 2 anos conta como "fim". Isso é a receita
documentada para abandono silencioso na Fase 5.

**O que cortaria ou adiaria sem perder o essencial:**

- **8.2 (SFU com Pion) e 8.3 (RTMP ao vivo)** — construir um SFU é projeto de meses mesmo para
  quem é sênior em mídia; o retorno para backend/plataforma é desproporcional. WebRTC 1:1 com
  Pion (ou LiveKit self-hosted operado por você) ensina 80% por 30% do custo. Marcar 8.2/8.3
  como *stretch*, não como caminho.
- **5.4 (IAM com motor de políticas)** — um avaliador estilo AWS IAM é divertido mas de baixo
  valor marginal; ACLs simples bastam para o cenário.
- **Módulo "C++ essencial" da T36 (R3)** — aprender C++ para comparar modelos de memória é o
  pedaço mais cortável do Rust; o laboratório sobrevive com leitura de referência.
- **7.7 e 7.8** — mantenha como opcionais.
- **Duplicidade de fontes** — vários itens listam trilha escrita + curso gravado cobrindo o
  mesmo assunto (ex.: 0.1 lista T13 fundamentos *e* T24 "01 e 02", que é introdução de curso;
  4.4 lista T32 + T21 + T23 + T9 + T15 + T18). A METODOLOGIA manda a IA escolher o mínimo, mas
  a lista bruta convida a super-estudar. Uma poda nas listas "Estudar antes" reduziria ruído.
- **Fase 6 inteira** é a mais profunda e a mais dispensável para empregabilidade — e a mais
  valiosa como diferencial de aprendizado. Não corte; *rotule*. O erro é tratar tudo como
  igualmente obrigatório.

**Correção estrutural:** dividir o projeto em **edições com fim próprio** (ver mudança 1).

---

## 2. Ordem e progressão

A macrosequência está certa: domínio puro → HTTP → hexagonal → Postgres → distribuído →
plataforma → peças de dados → produto → AWS. Três acertos que valem registrar porque fogem do
senso comum: **observabilidade no item 1.7** (quase todo mundo joga para o fim), **K8s depois
de construir o próprio orquestrador** (5.8 ensina exatamente o que o K8s resolve) e
**antifraude extraída de propósito** (4.4 é o melhor exercício de microsserviços do plano).

Problemas, em ordem de gravidade:

1. **A primeira semana é só teoria.** §9 manda ler T23 caps. 1–2 + T21 caps. 1, 2 e 21 + T40 +
   T13 fundamentos *antes* do 0.1 — que é `go mod init`, um Makefile e um `main.go`. Seis
   capítulos e um módulo de git para criar um repositório. O padrão se repete: item 0.2
   (200 linhas de código) exige IEEE 754, value objects, testes de tabela e fuzzing. A razão
   estudo:código está invertida no começo — justamente quando a motivação mais precisa de
   código rodando. Falta um "escreva código no dia 1" e a permissão para o estudo acontecer
   *em paralelo* com a execução em itens mecânicos.
2. **Ordem dos capítulos viola dependência interna dos livros** — ver seção 6.
3. **CI não é item de roadmap.** §2 promete "CI no GitHub Actions rodando vet, lint, test
   -race e build" e marca como lacuna coberta pela T40. Mas a esteira do Banco é *projeto de
   trilha*, não item: pode nunca chegar ao repositório. Um repo público de portfólio sem CI
   por meses contradiz o objetivo. Vira item (0.5 ou dentro do 0.1) com govulncheck,
   dependabot e gitleaks de brinde.
4. **Rust com meses de intervalo (P4).** R1 depois da Fase 1, R2 depois da 3 — serão 2–4
   meses entre um Rust e outro. Linguagem se mantém com contato frequente, não com quatro
   mergulhos espaçados. O próprio gatilho de P4 admite ("se ficar esquecido, encurtar") —
   melhor resolver de design: kata semanal de 30 min em Rust, ou comprimir R1–R4 em blocos.
5. **A0–A9 "por pressão": crível até A4, ficção honesta depois.** Até A4 os gatilhos são
   reais (HTTP, outro contexto, outra organização, perfil de recurso). Em A5–A7 o "gatilho" é
   decisão de produto/currículo: nenhuma pressão obriga a trocar o NATS pelo seu broker nem a
   criar um cofre com envelope encryption — é vontade de aprender. A ficção funciona
   (motiva a arquitetura), mas seria mais honesto e mais útil chamar de *motivo de estudo* a
   partir de A5 — inclusive para o caderno registrar a verdade. Detalhe: a "fitness function"
   de A3 ("conciliação fecha no zero") é teste de aceitação, não fitness function contínua no
   sentido do Richards & Ford; o linter de imports do A1 é o exemplo correto.
6. **A fronteira SPI é simulada.** Você escreve os dois lados, então o "outra empresa" (o
   motivo mais forte da lista de §2) nunca impõe nada de fora. Mitigação barata: congelar a
   API do SPI numa versão e proibir mudar os dois lados no mesmo item — a evolução de
   contrato vira problema real de versionamento (casa com T22 cap. 5).
7. **Dependência de cronograma alheio:** vários itens citam módulos ⏳ da T18 que ainda não
   existem (M18, M21, M22…). Nunca bloquear item por isso; sempre haver fonte B.

---

## 3. Lacunas e excessos

**O que falta para um engenheiro backend/plataforma sênior:**

1. **Operar no tempo — a maior lacuna.** Tudo é construir → testar → fechar. Nenhum item faz o
   sistema *viver*: alerta que dispara às 22h, disco que enche, certificado que expira,
   upgrade de major do Postgres, postmortem. 4.6 ensaia failover uma vez e acabou. Um item
   recorrente **"Semana de operação"** (fim das fases 4, 5 e 8: alertas + runbook + incidente
   injetado + postmortem escrito) ensina o que nenhum item de construção ensina.
2. **Revisão por humanos.** A IA te corrige o tempo todo; ninguém mais te corrige nunca. Um PR
   em projeto open-source Go, ou um review externo do design doc do 9.2, calibra o que a
   câmara de eco não calibra.
3. **Migração de dados real.** Expand-migrate-contract no livro-razão com carga rodando
   existe como *etapa* da T37 ("migração com carga rodando") mas não como item do Banco. É o
   problema clássico que separa quem já operou de quem só construiu.
4. **"Apagar uma pessoa" (LGPD/PII).** Chave PIX é CPF — dado pessoal. Como deletar alguém de
   um livro-razão imutável? Crypto-shredding/pseudonimização é um item barato, real e que nenhum
   tutorial ensina.
5. **Supply chain mínimo** — coberto acima (govulncheck, dependabot, SBOM, scan de imagem).
6. **Alertas e error budget** — 1.7 escreve um SLO e nunca mais se fala nele. SLO sem alerta é
   decoração.
7. **E-mail de verdade** — 7.8 promete e-mail; self-host de e-mail (SPF/DKIM/reputação) é um
   poço de dor que o item não precifica. Ou assume e aprende, ou cai fora e fica só com ntfy.

**O que sobra:** já dito — 8.2/8.3 como caminho (ok como stretch), 5.4 completo, o C++ do R3,
e parte da fase 8. E uma sobra de processo: listas "Estudar antes" de 6–9 fontes por item.

**Uso subaproveitado:** você tem apps reais em produção (a própria Trilha, Biblioteca,
Acervo, Streaming). O plano nunca os usa como superfície de estudo. Dois usos óbvios: (a)
analisar/refatorar a Trilha com o DDD aprendido — código legado seu vale mais que greenfield;
(b) **a Nuvem só vira plataforma quando hospeda um tenant real**: migrar um app seu para ela
(no fim da "edição 1") dá stakes que o Banco fictício nunca dará.

---

## 4. Metodologia: ensina ou vira burocracia?

**Ensina — para itens de design. Vira teatro — para itens mecânicos.**

O ciclo problema → estudo → checagem → caminhos → decisão → plano → execução → fechamento é
excelente quando há decisão real (3.3 saga, 4.4 extração, 6.2 Raft). É pantomima quando não há
("2 a 4 caminhos com vantagens e desvantagens" para um Dockerfile multi-stage gera opções
fabricadas e ensina a cumprir ritual, não a decidir). P1 já prevê um "modo curto" — *como
gatilho reativo*. Esperar doer para criar o modo curto é o erro: defina-o agora e classifique
cada item S/M/L no próprio roadmap.

Onde trava:

- **Estudo serializado antes da execução.** A regra do ciclo proíbe começar a codar antes de
  checar o estudo. Com 1–2h/dia, um item cuja fase de estudo leva 10h empurra qualquer código
  para a segunda semana. Para itens mecânicos, estudo e execução deviam correr em paralelo.
- **Checagem é só do item corrente.** Cenário-pergunta é bom (recuperação ativa + transfer),
  mas nada re-sonda o item de 3 meses atrás. A plataforma já tem simulado `intercalado` —
  ninguém o usa no projeto.
- **Sem regra de WIP/tamanho.** Nada diz "item > ~2 semanas → divide". Itens como 5.5 ou 8.2
  virariam vários itens, cada um com "pronto quando" próprio.
- **"Quanto a IA escreve é decidido na hora"** (METODOLOGIA §3.7) é uma catraca que só gira
  num sentido: sob cansaço, a tendência é pedir mais à IA. O REPERTORIO mede o resultado
  depois, mas não há trava *durante*.
- **Revisar ≠ aprender.** Quando a IA escreve periférico e você "revisa", nada verifica que
  você entendeu. Faltou a checagem incluir *você explicar o trecho que a IA escreveu*.

Contra as técnicas clássicas:

| Técnica | No plano? | Veredito |
|---|---|---|
| Prática deliberada | Sim (itens miram habilidades específicas) | Bom |
| Recuperação ativa | Parcial (checagem por item, não cumulativa) | Adicionar simulado intercalado mensal |
| Revisão espaçada | Só na Trilha, por etapa | Estender ao projeto: cada fechamento exporta 3–5 cartões ao Flashcards (a integração já existe) |
| Feynman | Fraco ("O que aprendi" no fechamento é vago) | Teach-back obrigatório: 10 linhas explicando o conceito a quem não sabe |
| Katas / re-prática | Ausente | Reimplementar um núcleo a frio todo mês (Dinheiro, outbox, rate limiter) |
| Projeto guiado | Sim, é a espinha dorsal | Bom |
| Feedback humano | Ausente | 1 PR externo ou review pago por edição |

E um detalhe pequeno com efeito grande: `itens/_MODELO.md` não registra **tempo gasto** nem
**dúvidas abertas**. Sem tempo, não há métrica; sem dúvidas, o ponto fraco morre na sessão.

---

## 5. A IA como mentora

**Riscos, do mais provável ao mais grave:**

1. **Câmara de eco.** A IA recomenda o caminho, avalia a sua defesa do caminho e depois revisa
   o código do caminho — às vezes código que ela mesma escreveu ("a IA escreve e o Lucas
   revisa" não impede ela de revisar depois na execução). Regra ausente: *quem escreveu não é
   quem certifica*; para trechos da IA, o caderno devia registrar você explicando o código.
2. **Ground truth duplo-cego.** Nas trilhas de roteiro, a lição é gerada por uma IA e a
   checagem conduzida por outra IA — nenhuma ancora em fonte primária. Um erro convincente na
   lição vira "conhecimento conferido" na checagem. O CLAUDE.md não tem a regra óbvia:
   *"nas afirmações centrais, cite a fonte primária; se não souber, diga que não sabe."*
3. **Dependência e erosão gradual.** O ciclo é defensável contra "a IA decide por você" — ela
   pergunta antes de plano e execução. O ponto fraco é o §3.7 (decide na hora) combinado com
   fadiga sua e disponibilidade infinita dela.
4. **Memória entre sessões: suficiente para o projeto, insuficiente para você.** ESTADO +
   caderno + DECISOES + REPERTORIO preservam *o que foi feito e decidido*. Não preservam *onde
   você sofre* — dúvidas abertas, conceitos que precisaram de 3 explicações, o que a IA já
   teve que corrigir duas vezes. A Trilha tem "pontos fracos por conceito" no domínio; o
   caderno do projeto não captura o equivalente. Adicione "dúvidas abertas" ao modelo de item
   e uma linha de "atenção" no ESTADO.
5. **Drift entre DECISOES e ADRs.** A mesma decisão vive em `DECISOES.md` (caderno) e
   `docs/adr/` (repo). Sem referência cruzada, divergem em meses.
6. **`CLAUDE.md` está bom, com 4 furos:** falta a regra de citação de fonte primária; falta o
   modo curto; falta cadência de revisão cumulativa (só checa o item corrente); e não define
   precedência quando os documentos divergirem entre si (na prática METODOLOGIA > ROADMAP >
   ESTADO — deixe explícito). Menor: o nome e o formato prendem a mentoria a um fornecedor
   específico; um dia você vai querer comparar mentores — os documentos já permitem.

---

## 6. Livros por capítulo solto

**Funciona como referência, falha como ordem — e a seção 7.2 mistura os dois.**

A afirmação "cada capítulo se sustenta sozinho" (ROADMAP §3, "Os livros no projeto") é metade
verdade: os capítulos *podem* ser consultados isoladamente, mas os três livros têm ordem
cumulativa interna que o plano viola nos pontos errados:

- **T22 (Kleppmann):** a Parte II é escrita em cadeia — replicação (cap. 6) e particionamento
  (cap. 7) sustentam os capítulos de consistência e consenso (9, 10). O plano lê cap. 8 no
  1.3, cap. 9 no 1.4, **cap. 10 no 3.3/3.7 — meses antes de cap. 6 (item 4.6) e cap. 7
  (item 6.3)**. Você vai ler consenso e linearizabilidade sem ter lido líder/seguidor e
  quorum onde o autor os define.
- **T21 (Richards & Ford):** os capítulos de estilo avaliam cada estilo com estrelas nas
  *características* definidas nos caps. 4–6. O plano lê cap. 10 (camadas) no 1.1 e só chega
  aos caps. 4–6 no 1.7 — a tabela de avaliação do estilo chega sem vocabulário.
- **T23 (Khononov):** cap. 8 (padrões arquiteturais) presume contextos delimitados dos caps.
  3–4; o plano lê 8 no 1.2 e 3–4 no 3.1. Mais leve, mas o mesmo desenho.

Não estou defendendo livro lido na ordem a qualquer custo — a leitura JIT é a decisão certa
*para uso*. O problema é tratar leitura de referência e leitura de estudo como a mesma coisa.
**Correção:** dentro de cada parte dos livros, respeitar a ordem do autor (6→7→…→10 na Parte
II do Kleppmann custa só reordenar itens de leitura, não itens de construção); e quando a
ordem tiver que ser violada, marcar a etapa como *referência* (skim para decidir agora) com
revisita marcada — a própria tabela 7.2 já sabe fazer isso ("a primeira vez é a leitura, as
outras são revisão"), só não aplica onde dói.

Segundo ponto, mais banal e mais real: **as trilhas de roteiro estão vazias**. T36, T37 e
T40 lidas pela API: `materiais: 0`, `tem_licao: false` em todas as etapas. §8 diz "o conteúdo
entra depois, da melhor fonte" — ou seja, quem escreve ~200 lições de Rust/Postgres/auth/
mídia? Ou a IA gera (qualidade não verificada — ver seção 5, risco 2) ou você cura material
(horas não orçadas). A correção barata é **amarrar a fonte primária de cada trilha de roteiro
agora**: TRPL/Rust by Example para a T36, docs do PostgreSQL + *Use The Index, Luke* +
*PostgreSQL Internals* (Rogov) para a T37, OWASP/ASVS para a T39, RFCs e o paper do Raft onde
couber. Uma tarde de curadoria agora em vez de improviso por item.

---

## 7. Motivação e fim

**Como está:** o fim é binário e está a 2+ anos (§9b). Os marcos intermediários são técnicos
e invisíveis — "um teste passa", "um ADR existe". Nenhuma métrica de progresso em lugar algum
(o ESTADO não mostra %; a aba Projeto mostra itens, mas ninguém celebra itens).

**O que eu colocaria:**

1. **Edições com nome e fim** (mudança 1): cada uma fecha com artefato público — demo em
   GIF/vídeo no README do repo, post curto, tag `v1.0` no `banco`. A cada ~2–3 meses existe
   algo *visível para terceiros*, não só para você.
2. **Um ponto de parada honroso.** Se a vida apertar, fechar a edição 1 ("o Banco completo
   rodando na minha Nuvem, com um app real meu hospedado nela") é um fim com prestígio, não
   um abandono. Planos bons dão saída digna; planos frágeis transformam pausa em fracasso.
3. **Métricas mínimas no ESTADO:** itens feitos/total, horas da semana, % do núcleo escrito
   por você (o REPERTORIO já coleta a matéria-prima), % dos 3 livros. Uma linha por mês.
4. **Revisão do próprio plano:** uma retro mensal de 15 min — "o que está travando, o que o
   ciclo virou burocracia" — registrada no METODOLOGIA. Hoje a metodologia só muda quando
   alguém lembra de mudá-la; o gatilho deveria ser o calendário.
5. **Stakes reais:** hospedar um app seu de verdade na Nuvem (não só os fictícios) e colocar
   um curso real na Escola. Sistema com usuário real ensina operação; sistema sem usuário
   ensina só construção.

---

## 8. As 10 mudanças (ordem de impacto)

| # | Mudança | Por quê | Onde |
|---|---|---|---|
| 1 | **Fatiar em edições com fim próprio e demo pública.** Edição 1 = Fases 0–4 + 5.1–5.3 + um app real seu migrado para a Nuvem ("Banco completo, plataforma básica, tenant real"); Edição 2 = resto da 5 + 6; Edição 3 = 7–8; Edição 4 = 9. Cada edição fecha com artefato público. | Mata os dois maiores riscos de uma vez: abandono sem marco (§7 aqui) e escopo sem escala (§1). Dá ponto de parada honroso a cada ~6–9 meses. | ROADMAP §6 e §9b; METODOLOGIA §3.8 |
| 2 | **Dois modos de ciclo, definidos agora.** Curto (problema → estudo mínimo → plano → execução → fechamento) para itens mecânicos; completo para itens de decisão. Classificar S/M/L já no roadmap. | P1 prevê o modo curto só *se doer*. Sem ele, o ciclo de 8 fases vira teatro em metade dos 60 itens e desgasta a parte que mais ensina. | DECISOES P1; METODOLOGIA §3; ROADMAP §5 |
| 3 | **Amarrar fonte primária nas trilhas de roteiro + regra de citação no CLAUDE.md.** TRPL → T36; docs PG + Rogov + Use The Index Luke → T37; OWASP → T39; paper do Raft onde couber. E: "nas afirmações centrais, cite a fonte; se não souber, diga." | Hoje o conteúdo de ~200 etapas não existe (API: `materiais:0`, `tem_licao:false` em T36/T37/T40) e o examinador não ancoreia em nada externo. Dupla IA sem ground truth é como erro vira "conhecimento conferido". | ROADMAP §8; CLAUDE.md; `criar-trilha.md` C9 |
| 4 | **Item novo "Semana de operação" no fim das edições 1–3 + uma revisão por humano por edição.** Alertas de verdade, incidente injetado, postmortem, runbook; e um PR open-source ou review externo do design. | É a maior lacuna de senioridade: o plano ensina a construir tudo e a operar nada. E é a única calibração fora da câmara de eco IA–Lucas. | ROADMAP §6 (novo item); METODOLOGIA §4 |
| 5 | **Descope explícito: 8.2 → WebRTC 1:1 (SFU completo vira stretch), 8.3 opcional, 5.4 → ACLs.** Marcar itens como essencial/desejável (MoSCoW por fase). | Remove ~150–250h do caminho crítico sem tocar no que diferencia o projeto. | ROADMAP §6 fases 5 e 8 |
| 6 | **Fechamento exporta conhecimento: 3–5 cartões ao Flashcards + teach-back de 10 linhas + simulado intercalado mensal.** E verificar se pausar trilha (P7) suspende as revisões dela no Hoje — se sim, não pausar trilhas já estudadas. | Hoje a revisão espaçada para na fronteira da Trilha e a checagem morre no item. A infraestrutura para tudo isso já existe no app. | METODOLOGIA §3.8 e §4; DECISOES P7; CLAUDE.md |
| 7 | **Kata mensal + checagem do que a IA escreveu.** Reimplementar um núcleo a frio, cronometrado; e quando a IA escrever periférico, a execução inclui você explicando o trecho. | Revisar código da IA ≠ aprender. Sem re-prática, "com ajuda" nunca vira "domino" (o REPERTORIO só sobe se houver segunda vez — que hoje ninguém agenda). | METODOLOGIA §3.7 e §4 |
| 8 | **Métricas no ESTADO + orçamento por item.** Linha mensal: itens feitos, horas/semana, % livros, % escrito por você. Regra: item > ~2 semanas de agenda → dividir antes de começar. | Sem medida, drift de escopo é invisível até ser tarde. O modelo de item nem campo de tempo tem. | ESTADO.md; itens/_MODELO.md; ROADMAP §10 |
| 9 | **CI vira item (0.5 ou dentro do 0.1) com supply-chain mínimo:** GitHub Actions (vet, lint, test -race, build) + govulncheck + dependabot + gitleaks. | §2 promete CI e §8 o terceiriza para uma trilha; nenhum item o implementa no `banco`. É barato, é portfólio e é o tipo de coisa que se paga caro por adiar. | ROADMAP §2 e §6 item 0.1 |
| 10 | **Corrigir dependências internas dos livros na tabela 7.2:** T22 ler 6→7 antes de 9–10; T21 ler 4–6 antes dos estilos; T23 ler 3–4 antes do 8. Onde a ordem tiver que quebrar, marcar a etapa como *referência* com revisita marcada. | A premissa "capítulo se sustenta sozinho" é falsa nas Partes cumulativas; ler consenso antes de replicação gera a ilusão de entendimento que a checagem não pega (ela testa o item, não o livro). | ROADMAP §7.2 e §3 |

**Menções rápidas (fora do top-10):** campo "dúvidas abertas" + "tempo" em `itens/_MODELO.md`;
referência cruzada DECISOES ↔ ADRs; item "apagar uma pessoa" (LGPD vs razão imutável);
decidir e-mail vs só ntfy no 7.8; precedência explícita entre documentos no CLAUDE.md;
congelar a API do SPI para tornar a fronteira "outra empresa" mais real; nunca bloquear item
por módulo ⏳ da T18.

---

## Veredito de uma linha

O conteúdo do roadmap é melhor que 95% dos planos de estudo por projeto que existem; o que o
ameaça não é técnico, é **aritmética e processo**: 1.000+ horas sem marco intermediário,
cerimônia única para todo tamanho de item, conteúdo de trilha que ainda não existe e um fim
a anos de distância. As mudanças 1, 2 e 3 resolvem 80% do risco.
