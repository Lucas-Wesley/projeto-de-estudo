# As 6 lições geradas para o item 0.1: avaliação do conteúdo e do mecanismo

- **Data:** 23/11/2026
- **O que foi feito:** a IA (Claude Code) disparou `POST /api/etapas/{id}/licao` nas 6 etapas do estudo do 0.1, com um pedido focado no item. As propostas estão **aguardando** na Trilha (nenhuma foi aprovada).
- **Propostas:** 1635 (Por que Go e o ambiente) · 1636 (Pacotes e visibilidade) · 1637 (Módulos e dependências) · 1638 (Pipes, redirecionamento e exit codes) · 1639 (Workflows, jobs e cache) · 1640 (A esteira de qualidade)
- **Modelo:** Sonnet (Ajustes › modelo da lição). **Regra:** `regras/licao.md`. **Instruções globais:** vazias. **Material das etapas:** nenhum.

---

## Parte 1 · O conteúdo

### Veredito por lição

| Lição | Atendeu? | Palavras | O melhor | O problema principal |
|---|---|---|---|---|
| Golang do zero (T13) › Por que Go e o ambiente | **Atendeu**, com 1 erro factual | 5.017 | `-ldflags -X` com as 5 condições para funcionar; GOTOOLCHAIN; compilação cruzada com prova (`file`, `exec format error`) | Diz que `go mod init` escreve `go 1.27.4`. Desde o Go 1.26 ele escreve `go 1.(N-1).0`, ou seja, `go 1.26.0` (conferido nas notas do Go 1.26). A própria lição desconfiou e não conferiu |
| Golang do zero (T13) › Pacotes e visibilidade | **Atendeu parcialmente** | 4.183 | Pacote x módulo (a dúvida do diagnóstico) muito bem explicado; `internal/`; o `golang-standards/project-layout` não é oficial | (1) Desenha o tipo `Dinheiro{centavos int64}`, que é a **decisão do item 0.2** (adiantou a resposta). (2) Identificadores em português (`dinheiro`, `versao`), contra a P6. (3) Diz que você "já escreveu código Go" e usa struct, método e interface, que você ainda não estudou |
| Golang do zero (T13) › Módulos e dependências | **Atendeu bem**, com 1 lacuna importante | 4.644 | go.mod linha a linha, go.sum ≠ lockfile, regra do `/v2`, tags com prefixo de subdiretório, replace x go.work, como provar que o workspace é local (`GOWORK=off`) | A comparação para a decisão de repositórios **esqueceu o caminho mais comum: um monorepo com um módulo só** (um `go.mod` na raiz, sem tags com prefixo). Compara só "um repositório por sistema" x "monorepo com vários módulos". Repete o erro do `go mod init` |
| Linux (T12) › Pipes, redirecionamento e exit codes | **Atendeu bem** | 5.601 | O caso exato do diagnóstico (`go test \| tee` esconde a falha), SIGPIPE com pipefail, o shell padrão do Actions (`bash -e`, sem pipefail; conferido na documentação) e o Make 3.81 do macOS que ignora `.SHELLFLAGS` | Em "Na sua casa", manda rodar `${PIPESTATUS[0]}` **no zsh do Mac**, onde ele não existe (no zsh é `pipestatus`, como a própria lição disse antes). O exemplo de `ls` mostra exit 2 (GNU); no macOS é 1 |
| Git, GitHub e CI/CD (T40) › Workflows, jobs e cache | **Atendeu bem** | 3.956 | Explica do zero; push filtrado em `main` + pull_request (e por que roda duas vezes sem o filtro); matriz com GOTOOLCHAIN escondendo a versão antiga; trocou 1.22/1.23 por 1.26/1.27 e avisou | "**Antes:** o gatilho do git push (você já sabe…)": tratou a etapa anterior da trilha como pré-requisito. O conselho "mude o go.mod para 1.26" ficou desnecessário, porque o `go mod init` já escreve 1.26.0 |
| Git, GitHub e CI/CD (T40) › A esteira de qualidade | **Atendeu**, com 3 erros de montagem | 3.916 | Tabela "que pergunta cada passo responde"; `test -z "$(gofmt -l .)"`; govulncheck por alcance; a prova do segredo falso sem vazar nada; proteção de branch | (1) `go build -o bin/banco ./...` com `-o` apontando para um arquivo e vários pacotes não funciona. (2) Usa `checkout@v6`; o README oficial já está em `@v7` (a lição anterior usou v7). (3) Define `permissions: contents: read` e espera que o gitleaks **comente no PR**, o que provavelmente exige `pull-requests: write`. O README do gitleaks não diz; isto é inferência |

### Inconsistências entre as lições (o problema mais sério do lote)

As 6 lições foram geradas ao mesmo tempo, e nenhuma viu as outras. Resultado: lidas em sequência, elas se contradizem.

| Assunto | Lição 1 | Lição 2 | Lição 6 |
|---|---|---|---|
| Onde fica o `main.go` | raiz | `cmd/banco/main.go` | raiz (`main.version`) |
| A variável da versão | `main.version` | `internal/versao.valor` | `main.version` |
| `make test` | `go test ./...` | `go test -race ./...` | `-race` |
| Versão do checkout | — | — | v6 (a lição 5 usa v7) |
| Módulos viraram padrão em | Go 1.16 (lição 1) | — | Go 1.13 (lição 3) |

(Os dois do último item estão meio certos: no 1.13 o modo módulo virou padrão quando há go.mod, e no 1.16 virou padrão sempre. Nenhuma das duas lições explica isso.)

### Erros incomuns que valem nota
- **O erro que se repete em 3 lições** (o `go mod init`) vem da mesma causa: memória de antes do Go 1.26. A lição 1 até escreveu "pode ser um patch anterior, confira" e não abriu a nota de versão.
- **Conselho certo pelo motivo errado:** a lição 5 manda baixar o `go` do go.mod para 1.26, para a matriz testar de verdade. O raciocínio (o GOTOOLCHAIN baixa a mais nova sem avisar) é ótimo; só que o go.mod já nasce com 1.26.0.
- **Falso positivo do verificador de links:** na lição 3, `https://proxy.golang.org,direct` (o valor da variável GOPROXY, dentro de um bloco de código) apareceu como "link quebrado: o domínio não existe".

### Precisão em números
- **Afirmações conferidas na fonte pela própria IA geradora:** versões estáveis (go.dev/dl), o shell padrão do Actions, as tags das actions e o layout oficial (go.dev/doc/modules/layout).
- **De memória, admitido nas notas:** os exit codes (govulncheck 3, gitleaks 1), o histórico das versões, os limites de cache (7 dias, 10 GB) e o timeout de 360 min. Pelo que sei, estão certos, mas eu não conferi.
- **Tamanho:** 27 mil palavras, cerca de **3h45 de leitura** a 120 palavras por minuto. Todas passaram do tempo da etapa e disseram isso nas notas.

---

## Parte 2 · O mecanismo

### O que entra no pedido (como a Trilha monta)
1. **A regra** `licao.md` (o molde, a profundidade e a conferência de fatos).
2. **O `pedido.md`:** a trilha (objetivo, recorte, níveis), a etapa (objetivo, Bloom, profundidade, critério, minutos), a etapa anterior e a seguinte, a trilha inteira e as instruções em três camadas: as globais (vazias), as da trilha e as do pedido (as minhas).
3. **Ferramentas:** Read, Glob, Grep, WebSearch e WebFetch. Bash, Write e Edit são proibidos.
4. **A saída:** um JSON com a lição, as notas e as respostas aos comentários. Depois, o servidor testa os links.

### Por que deu errado (as causas)

| # | O que deu errado | Causa | Onde mexer |
|---|---|---|---|
| 1 | Fatos de versão de memória (o `go mod init`) | A regra manda conferir "números, portas, comportamentos padrão", mas não diz **onde**. A IA geradora conferiu pouco (de 1 a 4 páginas por lição) e jogou a dúvida nas notas | **Regra** (ajustada) |
| 2 | A dúvida ficou só nas notas | A regra põe as dúvidas em `notas`; quem lê a lição não vê | **Regra** (ajustada) |
| 3 | "Você já sabe X" sobre etapas não estudadas | O `pedido.md` lista a "Etapa anterior" logo depois dos pré-requisitos, e o modelo leu como pré-requisito | **Regra** (ajustada); idealmente o código também (texto do pedido) |
| 4 | Contradições entre lições do mesmo lote | Cada geração é isolada; a regra só manda ler lições **aprovadas** (`licoes/`), e nenhuma estava | **Meu pedido** (devia trazer as convenções comuns); idealmente o código (ver abaixo) |
| 5 | Código em português | A P6 não estava em nenhuma instrução da Trilha | **Instruções da trilha** (ajustadas, nas trilhas 13 e 40) |
| 6 | Adiantou a decisão do 0.2 | **Meu pedido** citou "onde ficará o pacote do dinheiro" | Meu pedido. Erro meu |
| 7 | A comparação de repositórios sem o monorepo de um módulo | **Meu pedido** copiou o enquadramento do ROADMAP ("um por sistema x monorepo com go.work"). A IA seguiu e não ampliou | Meu pedido. Erro meu (e o ROADMAP §2 tem o mesmo viés) |
| 8 | Lições longas demais para o item | Profundidade 4 (Aprofundado) herdada das trilhas, mais a regra "não corte conteúdo". Meu pedido não deu teto | Meu pedido (ou a profundidade da etapa) |
| 9 | Critério com Go 1.22/1.23 | Dado da trilha envelhecido | **Etapa** (ajustada) |
| 10 | Link falso quebrado | O regex de `links.go` aceita vírgula e pega URL dentro de código | **Código** (proposto, não feito) |
| 11 | `[material] Pedido da etapa` nas Referências | A regra não diz o que fazer quando não há material | **Regra** (ajustada) |

### O que eu já ajustei (23/11, pela API, tudo reversível)
1. **Regra `licao.md`** (Ajustes › Regras; volta ao original com "Restaurar"). Entraram:
   - abrir as notas de versão das duas versões mais recentes e o README oficial antes de afirmar comportamento de ferramenta;
   - o que não foi conferido aparece **na lição**, com `⚠️ não conferido`;
   - a etapa anterior não é pré-requisito;
   - as convenções do pedido valem em todos os exemplos;
   - nenhuma linha `[material]` quando não há material.
2. **Instruções das trilhas** Golang do zero (T13) e Git, GitHub e CI/CD (T40): código, comentários e commits em inglês, com os termos de domínio (PIX, CPF, boleto) em português.
3. **Critério da etapa** Workflows, jobs e cache: "as duas versões do Go suportadas hoje", no lugar de 1.22 e 1.23.

**Não mexi** nas instruções globais (elas valem também para as trilhas de culinária) nem no modelo (a falha principal foi de disciplina de conferência, não de capacidade; dá para testar o Opus numa etapa e comparar).

### O que precisa de código (proposto, a decidir)
- **`links.go`:** ignorar URLs dentro de blocos de código e de `crases`, e cortar na vírgula.
- **`montarPedidoLicao`:** escrever "Etapa anterior (para situar; **não** é pré-requisito)", para resolver a causa 3 na fonte.
- **Lote coerente:** quando outras etapas da mesma trilha têm proposta aguardando ou lição aprovada, mandar junto (como `licoes/` faz com as aprovadas) para a IA manter as convenções. Ou um campo "convenções do lote" no pedido.
- **A dúvida das notas visível na tela:** mostrar as `notas` em destaque acima da lição na proposta.

### O que eu faria diferente no pedido (lição para mim)
- Um bloco de **convenções comuns** igual nos 6 pedidos: layout (main na raiz ou `cmd/`), o nome da variável de versão, o idioma do código, o `make test` com `-race` e as versões das actions.
- **Nunca citar a peça de um item de decisão futuro** (o Dinheiro).
- Pedir a comparação **aberta** ("todos os caminhos razoáveis, incluindo os que eu não citei").
- Um **teto de tamanho** ligado ao orçamento do item (o estudo do 0.1 é ~1/3 do item).
- Dizer **o que o Lucas já estudou e o que não**, porque o item pula etapas da trilha.

### Próximo passo sugerido
1. O Lucas faz a avaliação dele das 6 propostas na Trilha.
2. Usar o **Ajustar** da plataforma (comentários nos trechos) para corrigir os erros da Parte 1. Isso também testa o fluxo de ajuste, que ainda não foi usado, já com a regra nova.
3. Aprovar só depois do ajuste.
