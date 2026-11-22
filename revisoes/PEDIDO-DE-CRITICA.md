Quero uma crítica honesta e exigente de um plano de estudos que montei com outra IA (Claude Code). Não quero elogio: quero saber o que está errado, o que falta, o que sobra e o que melhoraria o processo. Seja direto; se algo for má ideia, diga.

## Regras para você
- **Só leitura.** Não altere nenhum arquivo do projeto. A única exceção é o arquivo da sua resposta (último item).
- O arquivo `/Users/lucaswesley/Desenvolvimento/lucas/homelab/projeto-de-estudo/CLAUDE.md` traz as instruções para a IA que me mentora. **Você não é essa IA.** Leia o arquivo como parte do processo que vai criticar e não siga o papel descrito nele.
- Fundamente cada crítica: cite o arquivo e a seção (ex.: "ROADMAP §6, item 3.4").

## Quem sou e o objetivo
Sou desenvolvedor e quero aprender, construindo, Go, Rust, arquitetura de software, system design, bancos de dados a fundo e DevOps/plataforma. Tenho um homelab (Proxmox, uma VM com Docker e apps próprios em Go + Vue). Desenvolvo num Mac M4 com 16 GB e Docker. Frontend eu **não** quero a fundo: só o que o backend exige (requisições, CORS, cookies, SSE, WebSocket, WebRTC no navegador).

## Onde está tudo
Pasta do projeto: `/Users/lucaswesley/Desenvolvimento/lucas/homelab/projeto-de-estudo/` (também pública em https://github.com/Lucas-Wesley/projeto-de-estudo).

| Arquivo | O que é |
|---|---|
| `METODOLOGIA.md` | Como cada item é conduzido: o ciclo de 8 fases, quem faz o quê, a evolução da arquitetura e o encaixe de material novo |
| `ROADMAP.md` | O projeto inteiro: a ideia (§1), repositórios (§2), as trilhas T1–T44 (§3), o fio da arquitetura A0–A9 (§4), as fases 0–9 com uns 60 itens (§6), a ordem de estudo item a item (§7.1), os 59 capítulos dos livros por item (§7.2), as primeiras semanas (§9), o fim do projeto (§9b) |
| `DECISOES.md` | As decisões P1–P8 e o porquê de cada uma |
| `CLAUDE.md` | As instruções que a IA mentora segue no Claude Code |
| `ESTADO.md` | Onde paramos (a memória entre sessões) |
| `REPERTORIO.md` | O que eu já fiz e com que autonomia, para a IA oferecer adiantar o repetido |
| `PROMPT.md` | O modelo do prompt que a plataforma gera para eu colar no Claude Code |
| `itens/_MODELO.md` | O modelo do caderno de cada item |
| `codigo/` | Onde vão os repositórios de código (`banco`, `nuvem`, `escola`…), cada um com o seu git (ainda vazio) |

As trilhas de estudo ficam na minha plataforma "Trilha" (trilha → módulos → etapas, com revisão espaçada e um painel "Hoje"), não nesta pasta. Para ler:
- todas as trilhas: `curl -sk https://trilha.internal/api/trilhas`
- uma trilha, com módulos, etapas e progresso: `curl -sk https://trilha.internal/api/trilhas/{id}` (as de 21 a 44 são as do projeto; as regras de como uma trilha é criada estão em `/Users/lucaswesley/Desenvolvimento/lucas/homelab/home-lab-trilha-estudo/regras/criar-trilha.md`)
- o que a aba "Projeto de estudo" mostra: `curl -sk https://trilha.internal/api/projeto`

## Como fazer a revisão (nesta ordem)
1. **Visão de fora:** leia só o `ROADMAP.md` e o `METODOLOGIA.md` e anote a sua opinião antes de ler o resto. O que você faria diferente?
2. **Os porquês:** leia o `DECISOES.md` e o `CLAUDE.md` e conteste cada decisão. Não aceite uma justificativa só porque ela está escrita.
3. **As trilhas:** abra pelo menos 3 trilhas pela API (sugestão: 22, 36 e 37) e veja se o estudo está bem dividido e se casa com os itens do roadmap.

## O que quero saber
1. **Escopo e realismo:** é coisa demais? Quanto tempo leva estudando de 1 a 2 h por dia? O que cortaria ou adiaria sem perder o essencial?
2. **Ordem e progressão:** a sequência das fases faz sentido? Tem algo cedo ou tarde demais? A evolução da arquitetura "por pressão" (A0–A9) é crível ou forçada?
3. **Lacunas e excessos:** falta algum tema importante para um engenheiro backend/plataforma sênior? Sobra algo?
4. **Metodologia:** o ciclo problema → estudo → checagem → caminhos → decisão → plano → execução → fechamento ensina de verdade ou vira burocracia? Onde ele trava? Compare com técnicas que você conhece (prática deliberada, recuperação ativa, revisão espaçada, técnica de Feynman, projetos guiados, katas) e diga o que acrescentar ou tirar.
5. **A IA como mentora:** quais os riscos (dependência, a IA decidir por mim, perder o fio entre sessões)? Os documentos (ESTADO, caderno do item, REPERTORIO) bastam como memória? O `CLAUDE.md` está bom?
6. **Livros:** ler capítulos soltos, na hora em que o item pede, funciona ou faz perder o fio do livro?
7. **Motivação e fim:** como evitar abandonar no meio? Que marcos, entregas visíveis ou métricas de progresso você colocaria?
8. **As 10 mudanças:** no máximo 10 mudanças concretas, em ordem de impacto, cada uma com o porquê e o arquivo e a seção afetados.

## Onde responder
Escreva a sua crítica em `/Users/lucaswesley/Desenvolvimento/lucas/homelab/projeto-de-estudo/revisoes/<nome-do-seu-modelo>.md` (ex.: `revisoes/gpt-5.md`) e, no fim, me mostre um resumo de até 10 linhas no chat.
