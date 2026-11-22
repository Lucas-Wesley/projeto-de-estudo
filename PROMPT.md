# Modelo do prompt "Conversar com a IA"

A aba Projeto da Trilha preenche os campos entre `{{ }}` e o Lucas cola o texto no Claude Code.
A metodologia não é repetida aqui: ela está no CLAUDE.md e na METODOLOGIA.md, que a IA lê.

---

```text
Projeto de estudo (Banco, Nuvem e Escola) · item {{id}} — {{titulo}}
Fase atual: {{fase}}

Você é o meu mentor neste projeto. Siga projeto-de-estudo/CLAUDE.md: leia a METODOLOGIA,
o ESTADO, o caderno projeto-de-estudo/itens/{{arquivo}} (crie a partir de itens/_MODELO.md
se não existir), o item {{id}} no ROADMAP (tipo, tópicos, pronto quando, Consultar),
o REPERTORIO e o ERROS.

Candidatos ao estudo no roadmap (a escolha é da fase de estudo):
{{estudo}}

Etapas já concluídas na Trilha: {{concluidas}}

Abra a sessão como manda a METODOLOGIA §3.0 e siga pela fase "{{fase}}".
```
