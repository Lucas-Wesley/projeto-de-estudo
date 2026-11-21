# Decisões do projeto

Uma linha por decisão. O detalhe (caminhos, argumentação) fica no caderno do item.

| # | Item | Decisão | Gatilho de evolução | Data |
|---|---|---|---|---|
| P1 | — | A metodologia (METODOLOGIA.md v1): ciclo de 8 fases, documentos como memória, plataforma mostra e gera o prompt, a IA conduz no Claude Code | Se o ciclo ficar pesado para itens pequenos, criar um modo curto | 21/11/2026 |
| P2 | — | O projeto começa como monólito modular; extrai só com um dos 5 motivos do ROADMAP §2 | O próprio fio da arquitetura (ROADMAP §4) | 21/11/2026 |
| P3 | — | **Onde mora cada coisa.** O caderno (metodologia, roadmap, estado, itens, repertório) é um repositório próprio, `projeto-de-estudo`, separado do app Trilha. O código de cada sistema fica no seu repositório (`codigo/banco`, `codigo/nuvem`…), com README e `docs/adr/` das decisões técnicas dele. A aba "Projeto de estudo" da Trilha só **lê** o caderno | Se o caderno precisar de escrita pela plataforma (ex.: marcar fase sem a IA), rever | 21/11/2026 |
| P4 | — | Rust entra como trilha paralela (R1 a R4), sempre reescrevendo algo já feito em Go, antes da peça real (6.4) | Se o Rust ficar esquecido entre os itens, encurtar o intervalo | 21/11/2026 |
| P5 | — | **Frontend só no que o backend exige** (requisições, CORS e cookies, SSE, WebSocket e WebRTC no navegador: T44). Framework próprio, React/Vue por dentro, CSS e design ficam fora; as interfaces são periféricas (a IA escreve, o Lucas revisa) | Se um item depender de entender algo do frontend além disso, ele entra como etapa pontual | 21/11/2026 |
| P6 | — | **Idioma do código: misto.** Domínio em português (Conta, Razao, Lancamento, Transferir — a linguagem ubíqua), infraestrutura e termos técnicos em inglês (handler, repository, middleware, config). Commits e documentação em português | Se o projeto ganhar colaboradores de fora do Brasil, rever | 21/11/2026 |
| P7 | — | **Trilhas no Hoje:** ao abrir um item, a IA ativa as trilhas que ele pede (as etapas entram no Hoje com revisão espaçada); quando o item fecha e nenhum item aberto usa mais a trilha, ela volta a pausar | Se o Hoje ficar pesado, ativar só a trilha principal do item | 21/11/2026 |
