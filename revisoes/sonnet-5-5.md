# Revisão crítica: Projeto de estudo (Banco, Nuvem e Escola)

**Modelo:** Claude Sonnet 5.5 · **Data:** 22/11/2026 · **Modo:** só leitura (nada do projeto foi alterado; o único arquivo escrito é este).

**Como li:** (1) só ROADMAP e METODOLOGIA, anotando a opinião antes de ler o resto; (2) DECISOES, CLAUDE.md, ESTADO, REPERTORIO, PROMPT e `itens/_MODELO.md`; (3) a API da Trilha: `/api/trilhas` (todas), as trilhas 22, 36 e 37 em detalhe, as 44 em varredura (minutos, lições, materiais) e `/api/projeto`. Cruzei as referências "Tn › Módulo › Etapa" do ROADMAP com os minutos reais de cada etapa para estimar a carga.
**Não li** as outras revisões que já estão em `revisoes/` (para não contaminar a minha). Os números de **construção** abaixo são estimativa minha (declarada como tal); os de **estudo** vêm da API.

---

## 0. Veredito em 10 linhas

1. O plano é **inteligente e mal calibrado**: a ideia (Banco → Nuvem → Escola, evoluindo por pressão) é boa; o tamanho não cabe em 1–2 h/dia. Minha conta dá **3 a 7 anos** (mais provável 4–5), não "um projeto com fim".
2. O maior risco **não é técnico, é comportamental**: em ~1,5 h de 21/11 saíram roadmap v2→v3, 24 trilhas e ~1.200 etapas. Hoje, **0 itens feitos e ~0,5 h de estudo registradas em 1.856 etapas** (API). Planejar virou o trabalho. Isso precisa parar antes de qualquer outra melhoria.
3. A metodologia **ensina pouco do que promete** e **gasta muito**: cinco fases de conversa (estudo→plano) antes do primeiro código, o caderno é **escrito pela IA** (não por você), e **não há recuperação ativa nem repetição do que *você* construiu**. Falta Feynman, kata, "prever antes de rodar".
4. A IA tem **autoridade demais e atrito de menos**: ela escolhe a fonte, grada, decide "o melhor caminho", revisa o código e escreve o caderno. Um modelo só, sem verdade externa. E o CLAUDE.md manda **oferecer adiantar o que se repete**, que é o contrário de praticar.
5. O estudo está **superdimensionado em fontes** (6 fontes para "transações") e **subdimensionado em tempo** (toda etapa vale 60 ou 90 min; Rust do zero em ~40 h).
6. Para "bancos a fundo" há **lacuna real**: nenhum item constrói um motor de armazenamento (B-tree/LSM + WAL + teste de queda). Raft (o melhor conteúdo de sistemas distribuídos do plano) só aparece na Fase 6, depois de ~2 anos de Nuvem.
7. A Fase 5 (Nuvem) reimplementa Vault + MinIO + IAM + cert-manager + escalonador + K8s-lite. Cada peça é um projeto de meses. O Kubernetes de verdade, que o mercado exige de quem é de plataforma, vem **depois** de tudo.
8. A Escola (Fases 7–8) gasta ~1/4 do esforço em **mídia** (HLS, SFU, RTMP), que está fora dos seus objetivos declarados. O que ela tem de valioso (7.3 PIX/webhook, 7.4 sandbox do juiz) cabe em 2 itens.
9. Os livros: a regra "os 3 livros inteiros ou o projeto não termina" (§9b) **força capítulos em itens que não os pedem**, o contrário do princípio "estudar porque precisa".
10. O fim é **tudo-ou-nada**. Sem marcos demonstráveis cedo, sem trilho principal/extra e sem válvulas de escape, o desfecho mais provável é abandono em 60%.

---

## 1. Visão de fora (anotada antes de ler o resto)

Só com ROADMAP + METODOLOGIA, as minhas primeiras reações, que depois confirmei ou corrigi:

- **Confirmei:** o escopo é de uma equipe de 3–5 pessoas em 2 anos, não de uma pessoa em 1–2 h/dia. Sessenta itens, cada um com estudo de várias fontes + construção + ADR + caderno.
- **Confirmei:** o plano é organizado por **camadas técnicas** (Banco, depois Nuvem, depois Banco-2, depois Nuvem-2…), não por **fatias entregáveis**. O primeiro "uau" (PIX entre dois bancos) está no item 3.2.
- **Confirmei:** o princípio "decidir antes de implementar" (METODOLOGIA §3.5→§3.7) **contradiz** itens do próprio ROADMAP que implementam e só então decidem (1.3, 4.2, 5.8). Detalhe em §5.
- **Corrigi:** achei que faltaria "DevOps de verdade". Não falta tema (T41, T40, 5.5–5.8, 9.1); falta **ordem** (Terraform e K8s tarde demais) e **uso do homelab real** como produção.
- **Corrigi:** achei que a lista de lacunas (§8) estaria completa. Não está: as próprias referências do ROADMAP têm 3 "sem par" (`/api/projeto`, `referencias.sem_par`) e dois ◐ abertos (linhas 319 e 543), e a §8 diz "nenhuma lacuna aberta".
- **Novo ao ler as trilhas:** o ✔ "você tem e cobre bem" (ROADMAP §5, linha 189) **não significa conteúdo**. Em T9–T15, T36–T44 e T21–T23 a API traz `tem_licao=false` e `materiais=[]` em todas as etapas. É um roteiro, não uma fonte.

---

## 2. Escopo e realismo

### 2.1 A conta (estudo: da API; construção: minha estimativa)

**Estudo.** Somando os minutos reais de todas as etapas/módulos citados em "Estudar antes" + livros + cursos, **sem contar duas vezes**: **≈ 1.120 h**. Isso é a lista de *candidatos* (a IA deve escolher o mínimo, METODOLOGIA §3.2), então o consumo real será menor; mas várias fontes são **alternativas para o mesmo assunto** (T34 109 h, T25 68 h, T26 53 h, T24 33 h, T29 33 h só nos citados) e a escolha não está feita. Assumo ~60% de aproveitamento: **~650 h**. Corrigindo a subestimativa das etapas (§2.2), **650–1.100 h**.

| Item (ROADMAP §6) | Horas de fonte listadas (API) |
|---|---|
| 0.1 O repositório e o ambiente | **12,9 h** (para criar um repo com Makefile) |
| 0.3 Conta e razão | 17,5 h |
| 1.3 Postgres | 19,3 h |
| 1.6 Rodar como serviço (Dockerfile + health) | **23,7 h** |
| 3.1 Domínio do PIX | 21,7 h |
| 5.5 Vários nós | 21,5 h |
| 5.8 Kubernetes | 31,7 h |
| Fase 0 inteira (0.1–0.4) | 42,7 h antes de escrever o primeiro Go "de verdade" |

**Construção** (estimativa minha, por fase, com a IA escrevendo os "periféricos"): F0 30–60 h · F1 100–180 · F2 150–300 · F3 150–300 · F4 100–200 · F5 250–500 · F6 150–350 · F7 120–250 · F8 100–250 · F9 50–100 → **1.200–2.500 h**. Referências de calibração: um Raft que passe testes sérios custa 40–80 h; um SFU multiparte funcional, 40 h+; um PaaS com borda+agente+construtor, 100 h+.

**Total ≈ 1.850–3.600 h.** Com 1,5 h/dia e ~330 dias úteis/ano (≈ 500 h/ano): **3,7 a 7 anos**. Com 2 h/dia: 2,8 a 5,5. Com 1 h/dia: 5,6 a 11. *Confiança: média no estudo, baixa na construção; mas mesmo o extremo otimista passa de 2,5 anos.*

### 2.2 As estimativas de minutos são uniformes e, portanto, falsas

- Toda etapa de roteiro vale **60 min** (ou **90 min** se for "Projeto"): T36 (35 etapas = 39,5 h), T37 (34 etapas = 38,5 h). Ver `minutos` na API.
- T37 M3 "MVCC e VACUUM" = 60 min; "Níveis de isolamento e anomalias" (reproduzir lost update, write skew, phantom) = 60 min. Kleppmann cap. 8 sozinho vale 120 min na T22.
- T36 M5 "Projeto: lista ligada dupla segura" (Rc/Weak/RefCell, comparada com Box) = **90 min**. É um exercício notoriamente de várias horas.
- T36 inteira ("Rust do zero" até `unsafe`, FFI, mmap, Tokio) em **~40 h**. Para chegar a escrever um segmento de log com `mmap`/`fsync` sem se perder, 150–250 h é o ordinário.
- T37 M8 "Projeto: mini KV com WAL" = **90 min**. Um KV com WAL **e teste de recuperação após queda** leva dias.

**Consequência:** o painel "Hoje"/previsão da Trilha vai mentir sistematicamente (T18 já prevê `2026-12-04`, mas é a única trilha com meta; ver §10). Um plano que erra o tempo por 2–4× produz a sensação de "estou sempre atrasado".

### 2.3 O que cortar ou adiar sem perder o essencial

O essencial dos seus objetivos (Go, Rust, arquitetura, system design, bancos a fundo, plataforma) está em **Fases 0–4 + partes de 5 + 6.1/6.2 + 7.3/7.4**. Cortaria ou adiaria (justificativa nas seções seguintes):

| Cortar/adiar | Onde | Por quê |
|---|---|---|
| Fase 8 inteira (chat, SFU, RTMP) e 7.6 (vídeo/HLS), 7.7 (trie), 7.8 (notificações) | ROADMAP §6 | Mídia/produto, fora dos objetivos. 7.6/8.2/8.3 sozinhos puxam T43 inteira (15 h) + T8 Borda + WebRTC no navegador |
| 5.3 mini-S3 e 5.4 IAM próprio | §6 Fase 5 | Cada um é um projeto grande para um conceito que se aprende **usando** MinIO/OPA e lendo o modelo de política da AWS |
| 4.2 event sourcing "experimental" | §6 | Útil, mas é exercício opcional; mover para "extra" |
| C++ (T36 M6 inteiro) | T36, ROADMAP R3 | Você não pediu C++. O "laboratório de 9 programas em 3 linguagens" é um desvio. Basta ler o texto comparativo e fazer o lado Go/Rust |
| T4 (Cloud Practitioner), T5 (Gramática), T17 (Inglês) **ativas** | Trilha | ROADMAP linha 4 diz que 5 e 17 "ficam de fora"; T4 é certificação de entrada, sem valor para sênior. Pausar |
| 7.5 (problemas do juiz = T11) | §6 | Engenhoso, mas puxa 43 h de algoritmos; extra |
| Duplicatas de fonte | §7.1 | T18, T15, T30, T8 "System design", T22/T21 cobrem o mesmo; escolher **uma** principal |

---

## 3. Ordem e progressão

### 3.1 A sequência das fases faz sentido? Em geral sim, com cinco ressalvas

1. **3.3 (Saga) vem antes de 3.4 (Outbox) e 3.5 (Resiliência), mas o critério de pronto da 3.3 só é cumprível com os dois.** (ROADMAP §6, 3.3 linhas 422–431 vs 3.4 e 3.5). "Testes para cada falha (o SPI cai antes, durante ou depois)" exige estado durável da saga, idempotência na chamada ao SPI e retry com backoff. Ou a 3.3 entrega uma saga conscientemente quebrada (e isso deve estar escrito), ou 3.4/3.5 sobem para antes.
2. **A extração da antifraude (4.4) vem *antes* do teste de carga (4.5).** O ADR da 4.4 "justifica a extração" por escala e isolamento de falha (§4, A4), mas a medição que sustentaria isso só vem na 4.5. Inverta (4.5 → 4.4) ou admita que é exercício de extração sem evidência.
3. **Raft (6.2) e a replicação (T22 cap. 6) estão tarde demais** para o objetivo "sistemas distribuídos a fundo". As Fases 3–4 já pedem conceitos de consenso, réplica e relógios (3.3, 3.6, 3.7, 4.6), mas o cap. 6 do Kleppmann só é lido no 4.6 e Raft só na Fase 6 (~ano 3).
4. **Kubernetes (5.8) é o último item de infra.** Para quem quer ser de plataforma, K8s é o piso do mercado. A lógica "construo o meu e depois comparo" é defensável, mas atrasa 2–3 anos a habilidade mais negociável e só funciona se você **chegar** lá. Uma versão fina antes (k3s rodando o Banco, depois da 2.7) faria a comparação da 5.8 ser real.
5. **Terraform/IaC real aparece só na 9.1 (última fase).** O homelab já é um ambiente real onde `terraform` (provider do Proxmox) e um pipeline CI/CD valeriam desde a Fase 1–2.

### 3.2 A "evolução por pressão" (A0–A9) é crível ou forçada?

**Parte crível, parte encenada.** Classificação honesta (ROADMAP §4):

| Transição | Pressão | Veredito |
|---|---|---|
| A0→A1 (hexagonal) | "A dor das camadas acopladas aparece em 1.1" | **Encenada**: você leu T21 cap. 10 e vai ler cap. 3 e T23 cap. 8 **antes** de sentir a dor. Funciona como descoberta guiada, mas o "gatilho" em DECISOES.md vira ficção retrospectiva |
| A2 (monólito modular), A3 (dois bancos → saga) | Orgânica: de fato o domínio pede | **Crível** |
| A4 (extrair antifraude) | "Perfil de recurso diferente" | **Forçada**: não há carga que justifique (4.5 vem depois) |
| A5 (Nuvem própria com agentes) | "Apps precisam de banco, fila, segredos e vários nós" | **Encenada**: nada obriga a construir um PaaS; um compose resolve. É um exercício de construir, e tudo bem, mas chame de exercício |
| A7 (broker/KV próprios) | "A Nuvem oferece fila e config gerenciadas" | **Encenada** (mesmo motivo) |
| A8, A9 | — | Produto e fechamento |

Isso não é defeito se declarado: separe **"pressão real"** de **"exercício deliberado"** em §4 e em DECISOES.md (coluna "tipo do gatilho"). Hoje a metodologia (§5) vende tudo como pressão, e é tentador a IA "fingir" um gatilho para justificar o item.

As *fitness functions* da §4 misturam **testes automatizáveis** (depguard na 1.2, "módulo só fala com outro pela API pública", "apagou na mão, a Nuvem recria") com **critérios de pronto** (a "Well-Architected" na A9 não é uma fitness function). Só as primeiras protegem a arquitetura no tempo; liste-as num lugar só e rode-as no CI.

---

## 4. Lacunas e excessos

### 4.1 O que falta para um engenheiro backend/plataforma sênior

| Lacuna | Evidência | Sugestão |
|---|---|---|
| **Motor de armazenamento (B-tree/LSM, buffer pool, WAL, recuperação)**: o seu objetivo "bancos a fundo" | ROADMAP não tem item; T37 M8 são **2 etapas + 1 projeto de 90 min** | Novo item entre 4.6 e 5.x: "storage engine" (LSM *ou* B+tree) com WAL e **teste de queda** (kill -9 + verificação). Fontes externas: CMU 15-445 (Pavlo), *Database Internals* (Petrov), *PostgreSQL Internals* (Rogov, gratuito). |
| **Verdade externa para consenso** | 6.2 usa critério próprio ("derruba o líder") | Usar o **MIT 6.5840 (labs de Raft)**: suíte de testes adversarial e independente da IA. Alternativa: teste de simulação determinística + um modelo TLA+ curto do Raft e da saga |
| **Segurança de plataforma**: threat modeling, supply chain (SBOM, assinatura de imagem, scan de dependências), gestão de segredos além do cofre | só `gitleaks` (§2) e T39 genérica | 1 item "threat model do Banco" na Fase 1/3 e um passo de supply chain no CI da 2.5 |
| **Resposta a incidente e pós-mortem** | 3.6 e 4.6 pedem runbook, mas nenhum pós-mortem | Todo item de caos/failover fecha com um pós-mortem escrito (formato curto, sem culpa). É a habilidade sênior mais visível |
| **Ler código de produção alheio** | ausente | Antes de 6.1/6.2: ler partes do etcd/raft, NATS, o `walreceiver` do Postgres. Comparar com o seu |
| **Feedback humano externo** | só a IA revisa (METODOLOGIA §4) | Pelo menos 1 revisão humana por fase (colega, comunidade, PR numa lib aberta) |
| **Contratos de API e evolução de esquema** | 3.2 cita "T42 › Contratos"; sem item | Pequeno item: versionamento de API + compatibilidade de eventos (T22 cap. 5) com teste de contrato |
| **Design doc cedo** | só na 9.2 | Um design doc curto por fase, não só no fim |

### 4.2 O que sobra

- **Mídia** (7.6, 8.2, 8.3, T43): ROADMAP §6; fora dos objetivos que você declarou. É a parte mais cara em esforço por conceito de backend aprendido.
- **T44** como trilha de 14 h: razoável *se* limitada a 3–4 etapas (CORS/cookies/SSE/WebSocket). Fica como está.
- **Cursos pagos duplicados** (T24, T25, T26, T27, T28, T29, T30, T31, T32, T33, T34, T35) listados em quase todo item do §7.1. Para a IA, listar tudo como candidato **aumenta** a decisão que ela tem de fazer e o seu ruído.
- **C++** (ver §2.3).
- **Português/Inglês/AWS CLF**: ativos na Trilha contra a própria regra (§2.3).

---

## 5. Metodologia: ensina ou vira burocracia?

### 5.1 O que funciona (guardar)

Intercalar assuntos; começar pelo domínio e pelo erro (0.4 "primeiro do jeito errado"); "gatilho de evolução" por decisão; "pular é permitido"; REPERTORIO com níveis. São bons mecanismos; o problema está ao redor.

### 5.2 Onde o ciclo trava

1. **Cinco fases de conversa antes do código.** Problema → estudo → checagem → caminhos → decisão → plano: até aqui **você não escreveu nada** (METODOLOGIA §3.1–3.6). Com 1–2 h/dia, um item médio gasta **dias** só nisso. A experiência tende a ser "estou em reunião com a IA", não "estou construindo".
2. **Contradição interna: "decidir antes de implementar" vs. os itens que implementam e depois decidem.** METODOLOGIA §3.5 põe a decisão antes do plano e da execução. Mas ROADMAP 1.3 (linha 272: "duas estratégias comparadas... Os dois resultados vão para um ADR"), 4.2 ("branch experimental... um ADR compara"), 5.8 ("um ADR compara") exigem o contrário: construir os dois e **então** decidir. E está certo: *você só consegue avaliar trade-offs de lock, event sourcing ou K8s depois de sentir a dor.* A metodologia deveria ter dois modos explícitos: **"decidir→construir"** (decisões caras de reverter: esquema, saga, fronteiras) e **"spike→decidir"** (decisões que só se enxergam construindo). O gatilho P1 ("criar um modo curto") já admite o problema, mas está só como promessa.
3. **"Caminhos" vira estratégia de espantalho quando há uma resposta óbvia** (1.1 API HTTP, 0.2 dinheiro em centavos, 1.5 Argon2). A IA tem de inventar 2–4 caminhos "com vantagens e desvantagens técnicas" (§3.4) mesmo quando o espaço de decisão é trivial. Classificar por item: *decisão real* vs *técnica a aprender*.
4. **O caderno é escrito pela IA** (METODOLOGIA §3.8, item 1: "A IA completa o caderno... o que aprendemos"; §2 coluna "Quem atualiza"). Escrever o que aprendeu **é** o exercício (técnica de Feynman, efeito de geração). Quando a IA escreve "O que aprendi", você terceirizou justamente a parte que fixa o conhecimento, e produz uma ilusão de domínio ("está tudo documentado").
5. **A checagem é feita por quem ensinou.** §3.3: a mesma IA explica, pergunta e corrige. Não há verdade externa. Para conteúdo que a IA gera na hora (todas as lições de T9–T15 e T36–T44), os erros do professor e do avaliador são correlacionados.
6. **A conferência na execução (§3.7) é boa**, mas só vale para código. Para decisões, checagem e ADRs não há equivalente (um teste, um experimento, uma contagem).

### 5.3 O que falta comparado com técnicas conhecidas

| Técnica | Estado no projeto | Acrescentar |
|---|---|---|
| **Recuperação ativa** | Só existe no estudo (revisão espaçada da Trilha). Nada traz de volta *o que você construiu* | Abrir **cada sessão** com 3 perguntas da sessão anterior, feitas pela IA **sem consulta**; ao fechar cada item, 3 cartões "refaça do zero" |
| **Revisão espaçada do código** | Ausente | Agenda de *katas*: rateio (0.2), transferência com lock ordenado (0.4), idempotência (1.4), LRU (4.3), rate limiter (3.7), **reescritos sem olhar** a cada 3–4 semanas. 20–30 min |
| **Feynman** | O caderno é da IA | O *fechamento* (§3.8) é **você** quem escreve: "explique como a um júnior" em 10 linhas + 1 diagrama; a IA só aponta furos |
| **Prática deliberada** | Há feedback imediato, mas **não há graduação de dificuldade** nem repetição na borda do seu nível | Por tema, 3 níveis (guiado → sozinho → com falha injetada) e só avançar quando o nível anterior passar sem ajuda |
| **Pré-teste** | Ausente (a checagem vem *depois*) | Antes do estudo, 3 perguntas "o que você espera que aconteça se…" (aprende-se mais quando se erra antes) |
| **Exemplos trabalhados/andaimes** | §3.7 diz "pede a ajuda que quiser" | **Escada de ajuda** formal: dica → conceito → esqueleto → solução, e quando houver solução, **redigitar no dia seguinte** |
| **Projeto guiado com testes externos** | Ausente | MIT 6.5840, CodeCrafters, "build your own X": o teste independente é a melhor checagem possível |
| **Intercalação** | Presente | Manter |
| **Registro de erros (misconceptions)** | Ausente | `ERROS.md`: "achei X, era Y". Alimenta as perguntas da recuperação |

### 5.4 O ciclo "vira burocracia"?

Para **itens de construção com assunto novo** (1.3, 3.3, 6.2), o ciclo é proporcional. Para **itens de aprender uma técnica** (0.2, 1.1, 1.4, 2.6, 7.2), a cerimônia é desproporcional. Meça: se o item tem ≥ 4 fases de conversa para um problema com 1 solução razoável, use o **modo curto**: problema (3 linhas) → *spike* seu → conferência da IA → fechamento (você escreve). Defina o modo curto **agora**, não como gatilho futuro.

---

## 6. A IA como mentora

### 6.1 Riscos concretos (com a evidência nos documentos)

1. **Dependência por desenho.** METODOLOGIA §3.7: "Quanto a IA escreve é decidido na hora, pelo Lucas". CLAUDE.md: "Não escreva o núcleo... sem ele pedir". Em ambos o único freio é você pedir, sem custo nenhum. Cansado, travado, às 23 h, o caminho de menor esforço é pedir. Falta atrito: escada de ajuda (§5.3), cronômetro ("trava há 25 min?"), e **registrar no REPERTORIO quando a IA escreveu** (hoje o nível "com ajuda" não diferencia dica de solução).
2. **CLAUDE.md incentiva o contrário de praticar.** Linha: "Antes de cada passo, olhe o REPERTORIO: se ele já fez algo quase igual sozinho, **ofereça** adiantar". METODOLOGIA §4 repete. A segunda repetição é onde a fluência se forma (prática deliberada). Trocar por: "se já fez, **peça para refazer sem olhar e cronometrado**; só adiante se você demonstrar domínio nessa reprise".
3. **Delegação do que mais importa para o seu objetivo.** METODOLOGIA §4: "Periférico (boilerplate, scripts, CI, configuração repetida, painéis) → A IA escreve e o Lucas revisa". Para quem quer **DevOps/plataforma**, Dockerfile, CI, compose, Terraform, scripts de deploy **são o conteúdo**, não periferia. Delegue Vue, painéis, CSS; **nunca** Dockerfile/CI/IaC/manifestos nos itens 1.6, 2.x, 5.x, 9.1.
4. **A IA decide.** METODOLOGIA §3.4 e CLAUDE.md: "diz qual é o melhor caminho para este cenário". Mesmo que você argumente primeiro, a recomendação final ancora. Adicione: antes da recomendação, **você escreve a decisão com suas palavras e um pré-mortem** ("como isso falha?"). Registre os desacordos e quem estava certo; é uma métrica de calibração (§10).
5. **Monocultura de um modelo.** O mesmo modelo cria o plano, gera as lições, escolhe as fontes, avalia a checagem, revisa o código e escreve o caderno. Erros viram ruído correlacionado. Contramedidas: testes externos (§5.3), documentação oficial como árbitro ("cite a seção"), uma segunda IA (como esta revisão) em cada ADR importante, e um humano por fase.
6. **Conteúdo gerado não verificado como fonte de estudo.** T9–T15 e T36–T44 têm 0 lições e 0 materiais; as lições serão geradas. Em temas sujeitos a versão (Postgres, Go, Rust, Kubernetes) e a nuance (Raft, isolamento), isso é risco de aprender um erro com confiança. Exigir `materiais` com **fonte primária** antes de uma etapa ficar disponível.
7. **Autoria.** CLAUDE.md: "Commits só com a autoria do Lucas, sem... menção à IA". Em repositórios *públicos* que funcionam como portfólio, e onde a IA escreve periféricos e testes, isso mistura "feito por você" com "feito com a IA". É decisão sua, mas o REPERTORIO (também público) já registra a autonomia; considere uma seção "Como a IA foi usada" no README de cada repo.

### 6.2 Os documentos bastam como memória?

- **ESTADO.md** (14 linhas): bom para "onde estou", **insuficiente** para (a) mais de um item ativo (a trilha R de Rust corre *em paralelo* às fases, mas ESTADO guarda "item atual" no singular), (b) dúvidas abertas, (c) o que fiz errado, (d) como retomar depois de 3 semanas de pausa.
- **REPERTORIO.md**: níveis sem **data**. Um "sozinho" de março não vale em novembro. Adicionar `última vez`, link para o commit e a tática de reprise (§5.3).
- **Caderno do item** (`itens/_MODELO.md`): oito seções preenchidas pela IA; é muito para manter e é a IA que o mantém. Reduza para 4 (problema/decisão/o que errei/o que aprendi **escrito por você**).
- **Falta:** `ERROS.md`, uma pilha de interrupções ("estou no 1.3, pausei por X, voltei por Y") e uma data de "revalidar antes de retomar".

### 6.3 O CLAUDE.md está bom?

**O que está bom:** ordem de leitura clara; "confira de verdade"; "pergunte como ele faria antes de mostrar o seu"; um bloco de perguntas por vez; sem Co-Authored-By.
**O que falta ou está errado:**
- Nenhuma instrução de **abertura de sessão** com recuperação ativa.
- Nenhuma regra de **honestidade epistêmica**: "se não tiver certeza, diga; cite a doc oficial; verifique versão antes de afirmar". Para Postgres/Rust/K8s isso é essencial.
- Nenhuma regra de **tempo**: "este passo passou de N min sem progresso: escada de ajuda".
- "Ofereça adiantar" (ver §6.1, item 2).
- "Recomende sempre o melhor caminho" sem exigir antes a sua decisão escrita (§6.1, item 4).
- Lê 5 arquivos + 1 `curl` antes de *qualquer* resposta. O ROADMAP tem 71 KB (~1.000 linhas); a API `/api/projeto` já serve, por item, o `markdown` e o `prompt` prontos. Use isso para encurtar a leitura (e para parar de duplicar o item entre ROADMAP §6, §7.1 e §7.2).
- Aponta `codigo/<nome>/` como destino dos repositórios, mas a pasta **não existe** (só consta no `.gitignore`); o pedido de revisão dizia "ainda vazio". Nada cria a pasta nem diz como (submódulo? clone independente?). Trivial, mas é o primeiro obstáculo do 0.1.

---

## 7. Os livros: capítulos soltos, na hora em que o item pede, funciona?

**Funciona para referência; não funciona para livros progressivos, e a regra do §9b piora.**

1. **Khononov (T23) é progressivo.** Cada capítulo constrói o vocabulário do seguinte. A ordem planejada (§7.2): 1–2 (0.3), **5–6 antes** (0.2), 8 (1.2), 3–4 e 12 (3.1), 9 (3.3), 15 (3.4), 7 (4.2), 11 e 14 (4.4), 10, 13, 18 (7.1), 16–17 (9.2). Você lerá o cap. 5 (lógica simples x modelo de domínio) **antes** do 1 e do 2 e os capítulos 10 e 13 (heurísticas, DDD no mundo real) **~3 anos depois** de 5–6. O livro inteiro é ~17 h (T23: 1.040 min). Leia-o em ordem **antes do 0.3**, em 2–3 semanas. Custo baixo, ganho alto.
2. **Kleppmann (T22) é modular**, e ler por capítulo é razoável. Mas a ordem planejada separa pares que se explicam: cap. 8 (transações, 1.3) e 9/10 (distribuídos/consenso, 3.3) são lidos **antes** do 6 (replicação, 4.6) e do 7 (sharding, 6.3), e o cap. 10 pressupõe o 9. Leia por **partes**: Parte I (1–5) na Fase 1; cap. 6–7 antes de 3.3; 8–10 em sequência; 11–13 mais tarde.
3. **Richards e Ford (T21)** é um catálogo; por capítulo funciona. Mas vários capítulos são **forçados** para cumprir a regra:
   - cap. 24 e 25 (equipes, negociação/liderança) em 7.1 e 9.2 ("defender a arquitetura é negociar");
   - cap. 16 (baseado em espaço) em "8.x"; cap. 13 (microkernel) em "2.x";
   - T22 cap. 14 (ética) e T23 cap. 16–17 (malha de dados) em 9.2.
   Isso contradiz o princípio do ROADMAP ("estudar porque precisa", linha 3). Foram encaixados porque §9b diz "o projeto só termina com os três livros lidos" e "um capítulo não sai".
4. **Ler capítulo "no momento certo" cria um custo oculto:** reler o resumo do livro cada vez que o item pede o capítulo seguinte (ver o "revisto em…" em §7.2). Sem anotações suas (Feynman), o fio do livro se perde.

**Proposta:** (a) separar a leitura dos livros do fio dos itens: **clube de leitura semanal**, 1 capítulo/semana, **em ordem**, ao lado do projeto, com um resumo seu de 10 linhas; (b) no ROADMAP, a §7.2 vira **ponteiro** ("o item X *usa* o cap. N") e não **obrigação de leitura**; (c) tirar "3 livros inteiros" da condição de fim (§9b).

---

## 8. As decisões P1–P8, contestadas

| # | Decisão | Contestação |
|---|---|---|
| **P1** Metodologia v1 (ciclo de 8 fases) | Gatilho: "se o ciclo ficar pesado, criar modo curto" | **O gatilho é a desculpa para não definir o modo curto hoje.** Já se sabe que é pesado para ≥ metade dos itens (§5.4). Definir agora, com critério de quando usar cada modo |
| **P2** Monólito modular; extrai só por um dos 5 motivos (ROADMAP §2) | Bom princípio | **O motivo 5, "outra linguagem" (§2, linha 73), é o mais fraco e é anti-padrão** (T21 cap. 18 e 19 desaconselham). Está ali para justificar o Rust no `fila` (6.4). Rust num serviço separado por *perfil de recurso e isolamento* é defensável; "outra linguagem" sozinha, não |
| **P3** Caderno em repo próprio; a Trilha só lê | Separar ferramenta e conteúdo é certo | Mas cria **duas fontes de verdade** que *você* reconcilia manualmente: progresso do estudo na Trilha (você marca) vs. estado do item no repo (a IA marca). METODOLOGIA §3.8, item 4 ("lembra o Lucas de marcar as etapas") transforma isso em tarefa recorrente. Automatizar (commit do ESTADO → a aba lê) ou aceitar que um dos dois é só leitura |
| **P4** Rust em paralelo (R1–R4), sempre reescrevendo algo de Go | A ideia é ótima | **A cadência é longa demais.** R1 "depois da Fase 1", R2 "depois da Fase 3", R3 "depois da Fase 4", R4 "depois da Fase 5" (ROADMAP, tabela R). Entre R1 e R2 passam as Fases 2–3 (14 itens, centenas de horas, ~8–10 meses a 1,5 h/dia). Ownership/lifetimes se perdem. O próprio gatilho de P4 admite: "se o Rust ficar esquecido". **Proposta:** 1 sessão fixa de Rust por semana (30–45 min), portando **pequenas peças que você acaba de fazer em Go** (rate limiter, LRU, `resiliencia`, o ID E2E) |
| **P5** Frontend só no que o backend exige | Correto e coerente com o objetivo | Ressalva: 8.2 (SFU/WebRTC) exige um cliente de navegador funcional (getUserMedia, `RTCPeerConnection`); "a IA escreve e você revisa" aqui não é periférico, é metade do item. Mais um motivo para cortar a Fase 8 |
| **P6** Inglês no código, português no caderno | Bom para portfólio | Custo: ADRs em inglês, escritos por você (e não pela IA, §5.2) **reduzem** a qualidade do Feynman (explicar em língua secundária). Sugiro: **escrever o ADR em português** (caderno) e publicar a versão em inglês só quando o ADR for consolidado. E Inglês (T17) está "de fora de propósito" no ROADMAP, mas é pré-requisito prático desta decisão |
| **P7** Trilhas do item entram no Hoje; no fechamento voltam a pausar | Lógica sensata | **Na prática hoje está desligada:** 13 trilhas estão `ativa` (T4, T5, T8–T15, T17, T18, T20 etc.) e só a T18 tem meta de estudo (30 min, seg–sex; `/api/hoje`). As outras têm `dias:[]`, `minutos:0`: **não há orçamento semanal de estudo para o projeto**. Sem orçamento, o Hoje não programa nada |
| **P8** Dev no Mac com Docker; homelab quando o item exige Linux ou vários nós | Faz sentido | Não trata **(a)** a diferença de arquitetura: Mac M4 é arm64 e o Proxmox é, provavelmente, x86 → imagens `buildx` multiarch, `GOARCH`, FFI do Rust, `--platform` em `testcontainers`, uma fonte constante de atrito (itens 1.6, 2.x, 5.x). Nem **(b)** o ciclo de desenvolvimento: "editar no Mac, rodar numa VM do Proxmox" é lento; uma VM Linux **local** (OrbStack/Lima, arm64) serve para namespaces/cgroups (2.1) e para o `minicontainer` sem sair do Mac. Nem **(c)** um **orçamento de recursos**: 5.5 (3 VMs + WireGuard + VLAN) e 5.8 (3 nós K8s) com Postgres primário+réplica, Prometheus, Grafana e dois Bancos exigem um quadro de RAM/CPU do Proxmox que o roadmap não tem |

---

## 9. As trilhas (22, 36, 37 em detalhe; varredura nas outras)

### T22 (Kleppmann): bem dividida, mas com problemas de uso
- 14 etapas, 1 por capítulo, 40–120 min; **coerente** com o livro. Bom.
- Sem lição, sem material: `instrucoes` manda "não invente conteúdo do livro" e o "guia de leitura" será gerado. **Perigoso para um livro que você tem fisicamente**: o guia gerado de um livro de 800 páginas vai conter erros plausíveis. Melhor: **seus** resumos + a IA só gera as **perguntas**.
- `nivel_partida` e `nivel_alvo` **vazios** (conferi na T22 e na T20; não varri as demais). Sem linha de base, o "já sei" (METODOLOGIA §3.2) fica na honra.

### T36 (Rust)
- Estrutura **muito boa** em módulos (M1 Ownership; M2 tipos próprios e erros; M5 smart pointers; M7 concorrência), com projeto ao fim de cada módulo que **reescreve algo já feito em Go**.
- **Problemas:** (1) tempo irreal (§2.2); (2) M6 inteiro é C++ (ver §2.3); (3) "Projeto: laboratório de memória... do item **6.4**" (M6) vs. ROADMAP R3 (esse laboratório é do R3, o 6.4 é o segmento de log): nomenclatura trocada; (4) `unsafe`, FFI e `mmap`+`fsync` em M8 como "2 etapas de 60 min" são o território onde Rust morde. Para o 6.4 isto precisa de 3× mais; (5) não há nenhum material nem fonte nomeada (o *Rust Book*, *Rustlings*, *Rust for Rustaceans* são evidentes).

### T37 (PostgreSQL)
- Cobertura temática boa (M0–M8), **mas é usuário-de-Postgres, não bancos a fundo**: só M2 ("Como o Postgres guarda os dados") e M8 ("por dentro", 2 etapas) tocam internals.
- **Sobreposições com o roadmap**: M8 "mini KV com WAL" é quase o 6.1; M7 pede um projeto "extrato em ClickHouse via CDC" que **não existe como item** no ROADMAP (a escola/Banco usa só Postgres). M6 (sharding/Citus) e M7 (Mongo/Cassandra/ClickHouse) são tópicos de *leitura*, não de construção: manter como leitura de 1 sessão.
- M3 "Transações e concorrência" (MVCC, isolamento, locks) é a melhor parte; falta amarrar explicitamente ao teste da 0.4 (soma global) como **critério de aprendizagem**.

### Varredura das 44 (o que as três não mostram)
- **Zero conteúdo** em T9–T15 (a base "✔ você tem e cobre bem") e em T36–T44: `tem_licao=false`, `materiais=[]`. Só T4 (30/30), T5 (40/48), T8 (38 materiais), T17, T18 e os cursos T19–T35 têm conteúdo ou material.
- **Progresso:** 2 etapas em T18 e 6 em T20; todo o resto em 0.
- **Só T18 tem meta de estudo.**
- `/api/projeto`: 770 referências casadas, 12 parciais, 3 "sem par": *"Prometheus, Grafana e OpenTelemetry na prática"*, *"PostgreSQL a fundo — a mesma lacuna da 1.3"* e *"o que você mesmo construiu em 5.5 a 5.7"*. As duas primeiras são lacunas **abertas** que a §8 do ROADMAP nega.
- Fragmentação do assunto "sistemas distribuídos / system design": **T8 (System design I/II), T15, T18, T30, T21/T22/T32** cobrem o mesmo núcleo. Escolher uma trilha-mãe (sugiro T15 + T22) e rebaixar as demais a "consulta".

---

## 10. Motivação e fim

**Por que o abandono é provável (pré-mortem):**
1. **Mês 1:** a distância entre "planejei tudo" e "codei nada". Hoje: ESTADO diz "projeto ainda não começou". O ritual de planejar rende dopamina; o trabalho duro (0.1–0.4) não rende nada comparável.
2. **Meses 3–6:** a Fase 1 é "CRUD de banco com Postgres", entediante *se você já faz isso*, e o ROADMAP não pergunta o que você já sabe (REPERTORIO vazio; `nivel_partida` vazio).
3. **Meses 8–14:** a Fase 2 exige Linux de verdade (P8), uma VM, mais atrito, e é a primeira em que o estudo (23 h para um Dockerfile) parece desproporcional.
4. **Ano 2–3:** a Fase 5 é um mundo de 8 projetos grandes; o fim parece *sempre distante*.
5. **O fim é tudo-ou-nada** (§9b: 9.2 fechado **e** três livros 100%). Sem escada de entregas, ficar em 60% é "falhei".

**O que eu colocaria:**
- **Marcos demonstráveis a cada 6–8 semanas** (não por fase técnica):
  - **M1 (semana ~4):** `banco` público no GitHub com extrato por HTTP e testes em CI (0.1–1.2).
  - **M2 (~semana 12):** Banco no Postgres, soma zero sob 1.000 goroutines, idempotente (1.3–1.4), com um GIF no README.
  - **M3 (~semana 24):** PIX entre dois bancos com saga e caos (3.2–3.6), relatório de conciliação.
  - **M4:** "Quanto aguenta" + failover medido com RTO/RPO (4.5–4.6).
  - **M5:** push → no ar em < 2 min (2.5).
  - Cada marco = **demo em vídeo de 2 min + post curto** (o repositório já é público).
- **Trilho principal e trilho extra.** Defina uma **linha de chegada 1 ("sênior de backend/plataforma")**: Fases 0–4 + 5.5/5.7 (versões enxutas) + 6.2 + 7.3/7.4 + R1–R3. O que fica fora (Escola de mídia, 5.3/5.4, 4.2, C++, T43/T44 aprofundados) vira "linha 2, opcional". **Defina o corte antes**, não quando estiver atrasado.
- **Orçamento real:** meta semanal (ex.: 5 sessões × 90 min) **na Trilha**, hoje só a T18 tem. Um item aberto por vez (WIP = 1) + a trilha de Rust (WIP = 2).
- **Métricas que importam** (e que a plataforma pode mostrar em `/api/projeto`): horas/semana, itens fechados/mês, taxa de acerto da recuperação ativa (§5.3), katas refeitas, ADRs escritos, tempo "item aberto → fechado" e **tempo até o primeiro commit do item** (se passa de 3 dias, o ciclo está pesado).
- **Regra de pausa:** permitir "pausado" no ESTADO com `RETOMAR.md` (você já usa isso na trilha de Português). Pausar não é falhar.
- **Usar o homelab real como produção:** as apps que você já mantém têm usuários de verdade (você). Aplique SLO, backup restaurado e alerta nelas **já na Fase 1** (item 1.7 e 4.6). Ensina mais do que um Banco fictício, e dá motivação por ter algo real.

---

## 11. As 10 mudanças (em ordem de impacto)

### 1. Congelar o planejamento e começar o 0.1 em 48 h, com orçamento semanal
**Por quê:** 0 itens e ~0,5 h de estudo registradas contra 1.856 etapas (API); roadmap v3 criado em ~1,5 h; o risco nº 1 é nunca sair da fase "preparação" (ESTADO.md). **Como:** (a) "congelamento": nenhuma versão nova do ROADMAP até o fim da Fase 1, salvo mudança disparada por atrito real; (b) meta semanal (ex.: 5 × 90 min) cadastrada na Trilha para o item corrente; (c) WIP = 1 (+ Rust).
**Onde:** ESTADO.md (novo "orçamento semanal" e "regra de congelamento"), METODOLOGIA §7 (regras gerais), Trilha (`meta` da trilha do item).

### 2. Dividir o roadmap em linha 1 (core) e linha 2 (extra), com marcos de 6–8 semanas
**Por quê:** a estimativa dá 3–7 anos para tudo (§2.1); o fim tudo-ou-nada (§9b) é receita de abandono (§10). **Como:** marcar cada item `★ core` ou `☆ extra` no §6; reescrever §9b: o projeto "termina" na linha 1; o resto é bônus. Marcos M1–M5 com demo.
**Onde:** ROADMAP §6 (marcas), §9 (marcos no lugar de "primeiras semanas"), §9b (fim), ESTADO.md.

### 3. Dois modos de ciclo, com teto de estudo e spike-first
**Por quê:** a contradição "decidir antes de implementar" vs. 1.3/4.2/5.8 (§5.2); cinco fases sem código; 6 fontes para o mesmo assunto; 12,9 h de fonte para criar um repositório. **Como:** modo **curto** (técnica conhecida/sem decisão real: problema → *spike* seu → conferência → fechamento escrito por você) e modo **completo** (decisão cara de reverter); em ambos, **teto de 3–4 h** de estudo antes do primeiro código e **uma fonte principal + uma de consulta** por item (a IA escolhe; o resto some do §6). Itens "spike→decidir": 1.3, 4.2, 5.8, 6.4.
**Onde:** METODOLOGIA §3 (modos e teto), §3.2, §3.5; DECISOES.md P1 (o "modo curto" deixa de ser gatilho); ROADMAP §6/§7.1 (cortar para o mínimo, marcando "alternativas").

### 4. Recuperação ativa, Feynman e katas
**Por quê:** a metodologia ensina o que você *ouve*, não o que você *retém* (§5.3). O caderno é da IA (§5.2, item 4). **Como:** abertura de sessão com 3 perguntas sem consulta; fechamento escrito **por você** (10 linhas + diagrama); katas agendadas (20–30 min, sem olhar) das peças-chave; `ERROS.md`; pré-teste antes do estudo.
**Onde:** METODOLOGIA §3.3 (pré-teste, recuperação), §3.8 (você escreve), nova §9 (katas); CLAUDE.md "Como conduzir"; `itens/_MODELO.md` (reduzir a 4 seções; "O que aprendi" marcado "escrito por Lucas"); REPERTORIO (coluna `última vez`).

### 5. Reduzir a delegação à IA e dar atrito ao pedido de código
**Por quê:** o único freio hoje é você não pedir (§6.1). O CLAUDE.md manda **oferecer** adiantar o repetido, e a METODOLOGIA delega CI/scripts/config, que são o conteúdo do seu objetivo. **Como:** escada de ajuda formal (dica → conceito → esqueleto → solução, com **reescrita no dia seguinte**); cronômetro (25 min); registrar no REPERTORIO "IA escreveu"; trocar "ofereça adiantar" por "peça para refazer sem olhar"; periféricos delegáveis = só interface/painéis/CSS, **nunca** Dockerfile/CI/IaC/manifestos.
**Onde:** METODOLOGIA §3.7 e §4; CLAUDE.md ("Como conduzir", "Antes de cada passo"); REPERTORIO (nível `IA escreveu`).

### 6. Diagnóstico inicial e compressão das Fases 0–1
**Por quê:** você já mantém apps Go em Docker no homelab; o plano gasta 42,7 h de fontes antes da 0.4 e 23,7 h só para um Dockerfile na 1.6 (§2.1); `nivel_partida` e REPERTORIO estão vazios. **Como:** 1 semana de **diagnóstico** (as 5 coisas que você faz hoje sem ajuda: rateio com int64, mutex com ordem de locks, API `net/http`, Dockerfile multi-stage, `slog`+`/health`); preencher "o que já sabia" no REPERTORIO; itens 0.1/0.4/1.1/1.6/1.7 viram **modo curto** ou *kata* de checagem; só estudar o que o diagnóstico mostrar.
**Onde:** METODOLOGIA §3.2 (diagnóstico antes do estudo), REPERTORIO ("O que sabia antes"), ROADMAP §6 itens 0.1–1.7, T-trilhas (`nivel_partida`).

### 7. Trazer "bancos a fundo" e consenso para antes, com testes externos
**Por quê:** é o seu objetivo declarado e hoje é o que mais falta/atrasa (§4.1): sem item de storage engine; Raft na Fase 6; critérios de pronto feitos pela própria IA. **Como:** (a) novo item após 4.6: **"storage engine" (LSM ou B+tree) com WAL e teste de queda** (substitui o "mini KV 90 min" da T37 M8); (b) mover 6.2 (Raft) para logo depois da Fase 3, usando **MIT 6.5840** como suíte de testes; (c) ler código (etcd/raft, NATS) antes e depois; (d) um modelo TLA+ curto da saga e do Raft; (e) Kleppmann cap. 6–7 antes da 3.3.
**Onde:** ROADMAP §6 (novo item; reordenar 6.2), §4 (A7 vira A5'), §7.1/§7.2; T37 (M8), T22.

### 8. Encolher a Nuvem (Fase 5) e antecipar o Kubernetes fino
**Por quê:** 5.2/5.3/5.4 reimplementam Vault/MinIO/IAM (projetos de meses para conceitos que se aprendem usando); K8s é a habilidade mais negociável e está no fim (§3.1, §4.1). **Como:** 5.2/5.3/5.4 → **usar** Vault/MinIO/OPA e escrever um ADR + um **protótipo mínimo** (envelope encryption de 1 dia, avaliador de política de 1 dia); k3s com o Banco logo após a 2.7 (Helm + Ingress); manter 5.5/5.7 (a sua Nuvem) e reservar a 5.8 para a **comparação**; Terraform do Proxmox desde a Fase 2; consertar a ordem 3.3 ↔ 3.4/3.5 e 4.4 ↔ 4.5.
**Onde:** ROADMAP §6 Fase 2 e Fase 5, §4 (A5/A6), §6 itens 3.3–3.5, 4.4–4.5.

### 9. Rust em cadência fixa (semanal), sem C++
**Por quê:** a cadência de P4 deixa meses entre R1–R4 (§8, P4); C++ não é objetivo seu; os tempos são 2–4× curtos (§2.2). **Como:** 1 sessão de Rust por semana, portando **pequenas peças que acabou de escrever em Go**; R1 logo depois da 0.4 (o `Dinheiro` fresco), não depois da Fase 1; trocar o laboratório C++/Go/Rust por **texto comparativo + 2 bugs em Go e Rust**; reestimar T36 (≥ 100 h); fontes nomeadas (Rust Book, Rustlings).
**Onde:** ROADMAP tabela R1–R4 e item 6.4; DECISOES.md P4; T36 (M6, minutos e `materiais`).

### 10. Livros fora do fio dos itens (leitura em ordem, em ritmo semanal)
**Por quê:** capítulos fora de ordem em Khononov e Kleppmann, e capítulos forçados só para cumprir o §9b (§7). **Como:** clube de leitura de 1 capítulo/semana, em ordem; Khononov **antes do 0.3**; a §7.2 vira "ponteiros", e os capítulos sem item natural (T21 cap. 24, 25; T22 cap. 14; T23 cap. 16, 17) ficam fora do critério de fim; resumo seu de 10 linhas por capítulo.
**Onde:** ROADMAP §3 ("Os livros no projeto"), §7.2, §9b; METODOLOGIA §3.8 item 4, §8 (último parágrafo).

---

## 12. Defeitos factuais e pequenas incoerências (não entram nas 10)

| # | Onde | Problema |
|---|---|---|
| a | ROADMAP §8 (linha 943) × §6 itens 1.7 (linha 319) e 4.6 (linha 543) | §8 diz "nenhuma lacuna aberta", mas há ◐ explícitos para "Prometheus/Grafana/OpenTelemetry na prática" e "PostgreSQL a fundo" (este último, aliás, já virou T37; o ◐ do 4.6 está obsoleto) |
| b | ROADMAP §5 (linha 189) | ✔ "você tem e cobre bem" quando as trilhas citadas (T9–T15, T36–T44) não têm lições nem materiais |
| c | T36 M6 projeto × ROADMAP R3 | "laboratório de memória do item 6.4" vs. R3 |
| d | ROADMAP linha 4 × Trilha | "Português (5) e Inglês (17) ficam de fora" e ambos `ativa` na Trilha; T4 também |
| e | ESTADO.md "Trilhas 21–44 pausadas" × `/api/trilhas` | 13 trilhas T1–T20 estão ativas, sem meta |
| f | ROADMAP §4 × §7.1 × §7.2 | O mesmo mapeamento livro→item aparece em **três** tabelas; qualquer edição exige três mudanças. A API já parseia o item; manter **uma** |
| g | ROADMAP §6 item 5.8 | "T25 › capítulos 7 a 9... 12 a 29" são ~25 capítulos de um curso de 247 etapas para uma decisão de comparação; é a fonte que mais pesa (31,7 h) |
| h | `.gitignore` × CLAUDE.md × pedido de revisão | `codigo/` não existe; o pedido dizia "ainda vazio" |
| i | T37 M7 | O projeto ClickHouse+CDC não corresponde a nenhum item do ROADMAP |

---

## 13. O que eu **não** mexeria

- Começar pelo Banco em Go puro (Fase 0) e deixar HTTP/Postgres para depois.
- O formato de cada item no ROADMAP: **Implementar / Estudar antes / Pronto quando / Repositório**. Critérios de pronto concretos (soma global zero sob `-race`, kill -9 sem perder transferência, p99 medido).
- O princípio "módulo só vira serviço com motivo + ADR" (P2), menos o motivo 5.
- "Intercalar" assuntos entre trilhas (T9, T13, T37…).
- REPERTORIO como ideia, só com a data e o "IA escreveu".
- Repositório público + ADRs.

---

## 14. Limites desta revisão

- Estimativas de **construção** (§2.1) são minhas, sem dado real de velocidade; refaça-as depois do M1 com o seu ritmo medido.
- Não verifiquei o conteúdo dos três livros nem seus sumários (o ROADMAP diz "conferido pelas fotos"); só a coerência interna do plano.
- Não testei como a aba "Projeto de estudo" ou o Hoje se comportam na interface; usei a API.
- Não li as outras revisões em `revisoes/` por escolha.
