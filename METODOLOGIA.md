# Metodologia do Projeto de estudo

> Como o Lucas estuda e constrói o ecossistema Banco, Nuvem e Escola com a IA.
> Definida pelo Lucas em 21/11/2026. **Vale para todo item do [ROADMAP](ROADMAP.md)**: código, terminal, configuração, desenho, qualquer coisa.
> Versão 1.

---

## 1. A ideia

O projeto é grande, então ele é resolvido **item por item, em pedaços pequenos**. Cada item segue o mesmo ciclo:

```
problema → estudo → checagem → caminhos → decisão → plano → execução → fechamento
```

O Lucas **estuda porque precisa decidir** e **decide antes de implementar**. A IA não entrega a resposta pronta: ela mostra o problema, indica o que estudar e prova por quê, confere se ele aprendeu, discute os caminhos, pergunta como ele faria e confere cada passo que ele executa.

Há dois lados, e eles conversam **por documentos**:

| Lado | Onde | Papel |
|---|---|---|
| **Plataforma** | Trilha › Projeto de estudo (a construir, ver a seção 6) | Mostra o roadmap, em que item estamos, o que estudar (com link para a etapa) e **gera o prompt** que abre a conversa com a IA |
| **IA** | Claude Code, na IDE, nesta pasta | Conduz o ciclo do item, olha o código de verdade, confere a execução e **registra tudo nos arquivos** |

A IA **não** fica dentro da plataforma: no Claude Code ela já enxerga o código, o terminal e os documentos, e o prompt gerado só diz em que ponto estamos.

---

## 2. Os documentos (a memória do projeto)

Tudo o que importa fica em arquivo, para que qualquer sessão nova da IA (ou o Lucas daqui a um mês) saiba **o que queremos fazer, o caminho, onde paramos e qual é o próximo passo**.

| Arquivo | O que guarda | Quem atualiza |
|---|---|---|
| [CLAUDE.md](CLAUDE.md) | As instruções da IA: o que ler, como conduzir, o que registrar | Só muda com a metodologia |
| [METODOLOGIA.md](METODOLOGIA.md) | Este documento | Lucas e IA, quando a metodologia mudar |
| [ROADMAP.md](ROADMAP.md) | Os itens, em ordem, com o que estudar e o critério de pronto; o fio da arquitetura | IA propõe, Lucas aprova |
| [ESTADO.md](ESTADO.md) | **Onde paramos**: o item atual, a fase dele e o próximo passo concreto | IA, ao fim de toda fase |
| [REPERTORIO.md](REPERTORIO.md) | O que o Lucas **já fez** e com que autonomia; é o que permite à IA adiantar o que se repete | IA registra, Lucas confirma |
| `itens/<id>-<nome>.md` | O caderno do item: tudo o que aconteceu em cada fase (a partir de [itens/_MODELO.md](itens/_MODELO.md)) | IA, durante o item |
| `DECISOES.md` | Uma linha por decisão tomada (o detalhe fica no item) | IA, na fase de decisão |

Regra de ouro: **ao terminar uma fase, a IA escreve no caderno do item e atualiza o ESTADO.md antes de seguir.** A conversa pode acabar a qualquer hora; os arquivos não.

---

## 3. O ciclo de um item

### 3.1 Problema
A IA apresenta o item: **o que** vamos construir, **por que agora** (o que no projeto pede isso) e **onde ele encaixa no todo** (o que vem antes, o que depende dele). Curto e concreto.

### 3.2 Estudo
A IA indica o que estudar, sempre no formato `Tn › Módulo › Etapa` (pode ser uma etapa, um capítulo inteiro, dois capítulos, um livro; o tamanho é o que o item pede, sem regra fixa de pouco ou muito).

Para cada fonte, a IA **prova a necessidade**: "você precisa saber **A, B e C** porque, na fase de caminhos, vai ter de decidir **X**, e sem A não dá para avaliar a opção 1". O foco é sempre **a decisão ou a ação** que aquele estudo destrava.

- A IA escolhe entre **todas** as trilhas, não só as listadas no roadmap, e prefere a melhor fonte (pode combinar curso + livro).
- Antes de indicar, ela **consulta a Trilha** (`GET https://trilha.internal/api/trilhas/{id}`) para ver o que já está concluído.
- Ela pergunta se o estudo já foi feito. Se o Lucas disser **"já sei isso"**, o item pula direto para a checagem.

### 3.3 Checagem
Quando o Lucas diz que estudou, a IA **confirma que ele aprendeu**: perguntas de cenário ("o que acontece se…?", "o que você acha disto?"), não de decoreba. Para cada resposta, a IA diz **certo, errado ou incompleto** e corrige. Se faltar base, volta ao estudo com a fonte exata.

### 3.4 Caminhos
A IA apresenta **2 a 4 caminhos** possíveis para o item, cada um com **vantagens e desvantagens**. Vantagem e desvantagem **não são opinião**: são consequências técnicas, e a IA diz quando o Lucas avaliou certo ou errado.

Depois vem a **argumentação**: o Lucas defende um caminho, a IA questiona, aponta o que ele não considerou e diz qual é o **melhor caminho para este cenário** e por quê.

### 3.5 Decisão
Os dois chegam a um consenso e a IA registra:
- o caminho escolhido e **por quê**;
- os caminhos descartados e por quê;
- **o gatilho de evolução**: o que, no futuro, nos faria mudar esta decisão (ver a seção 5).

### 3.6 Plano
A IA **não** dá o passo a passo. Ela pergunta: **"como você implementaria isso? O que vem primeiro?"** O Lucas responde na teoria, sem precisar fazer. Se estiver certo, segue; se não, a IA corrige a linha de raciocínio; se ele não souber, tudo bem, a IA explica e seguem.

O resultado é a lista de passos **construída junto**, cada um com:
- o que fazer;
- o critério de pronto (como saber que o passo está certo);
- quem faz: **Lucas** ou **IA** (ver a seção 4).

### 3.7 Execução
Um passo de cada vez. O Lucas faz (código, comando, configuração) e avisa. A IA **confere de verdade**: lê o código, roda os testes, olha a saída do terminal, o estado do serviço. Diz o que está certo, o que está errado e por quê. Só então vai para o próximo passo.

Se o Lucas travar, ele pede a ajuda que quiser: uma dica, a explicação do conceito, o passo detalhado linha a linha ou até que a IA escreva aquele trecho. **Quanto a IA escreve é decidido na hora, pelo Lucas.**

### 3.8 Fechamento
O item fecha quando o critério de pronto do roadmap é cumprido. A IA:
1. completa o caderno do item (o que foi feito, o que aprendemos, o que ficou para depois);
2. atualiza o [REPERTORIO.md](REPERTORIO.md) com o que o Lucas fez e com que autonomia;
3. marca o item como feito no roadmap e aponta o próximo no [ESTADO.md](ESTADO.md);
4. lembra o Lucas de marcar as etapas estudadas como concluídas na Trilha (se ainda não marcou).

---

## 4. Quem faz o quê

| Situação | Quem faz |
|---|---|
| O conceito que o item ensina (o núcleo) | **Lucas** escreve; a IA confere |
| Algo que o REPERTORIO mostra que o Lucas **já fez sozinho** e é quase igual (ex.: a segunda API de listagem depois de ele ter feito a primeira) | A IA **propõe** fazer ("você já fez isto no item 1.1; posso adiantar?"). O Lucas aceita ou recusa |
| Algo novo para o Lucas | A IA avisa que é novo e o Lucas faz |
| Periférico (boilerplate, scripts, CI, configuração repetida, painéis) | A IA escreve e o Lucas revisa |
| Testes de aceitação | O Lucas faz **o primeiro** (para aprender o conceito); depois a IA escreve e o Lucas revisa |
| Revisão de código | A IA, aqui no Claude Code, a cada passo da execução |

A IA **sempre avisa** antes de adiantar algo e **nunca** adianta o núcleo de um item novo sem o Lucas pedir.

---

## 5. A progressão: tudo evolui, inclusive a arquitetura

O estudo é **coerente e cumulativo**: cada item se apoia nos anteriores, sem conceitos soltos fora de hora.

A arquitetura **existe desde o dia 1**, consciente ou não. Por isso ela é tratada como um **fio contínuo**, não como um capítulo:

1. Cada fase do roadmap começa com uma arquitetura que **dá conta até certo ponto**.
2. Toda decisão registra o seu **gatilho de evolução** ("quando acontecer X, esta escolha deixa de servir").
3. Quando o gatilho dispara, **a pressão vira um item**: estudar, discutir caminhos e evoluir a arquitetura. Isso é feito de propósito, como exercício.
4. **Fitness functions** (testes que verificam a arquitetura, como "o domínio não importa infra") protegem o que já foi decidido.

A sequência planejada está no [ROADMAP](ROADMAP.md), seção "O fio da arquitetura": de um pacote Go puro até a plataforma distribuída com Kubernetes.

---

## 6. A plataforma: Trilha › Projeto de estudo (a construir)

Uma nova opção no menu da Trilha: **Projeto de estudo**.

**O que ela mostra**
- As fases e os itens do roadmap, com o estado de cada um (a fazer · em andamento, com a fase do ciclo · feito).
- No item: o problema, o que estudar com **link direto para cada etapa** da Trilha (e se ela já está concluída), o critério de pronto e o caderno do item.
- O ESTADO atual em destaque: "você está no item 1.3, na fase de caminhos".

**O botão principal: "Conversar com a IA"**
Gera o texto do prompt (modelo em [PROMPT.md](PROMPT.md)) já preenchido com o item, a fase e os arquivos. O Lucas copia, cola no Claude Code e a conversa segue a metodologia.

**De onde vêm os dados**
- **Os documentos deste caderno são a fonte da verdade** sobre o projeto. A plataforma lê, não escreve.
- **Onde mora cada coisa** (decisão P3): o caderno é o repositório `projeto-de-estudo`, separado do app Trilha (o app é a ferramenta; o caderno é o conteúdo, que muda todo dia). O código fica nos repositórios de cada sistema (`codigo/banco`, `codigo/nuvem`…), cada um com o seu README e `docs/adr/`. O caderno guarda **o processo de aprender** (estudo, checagem, caminhos, repertório); o repositório do código guarda **o que foi decidido e construído**. O caderno do item aponta os commits.
- O progresso do **estudo** é o da própria Trilha (etapas concluídas). A IA consulta pela API.

Assim, cada lado registra o seu passo e o outro lê: o Lucas marca o estudo na Trilha, a IA registra o resto nos documentos.

---

## 7. Regras gerais

- **Nada começa sem estar descrito.** Antes de mexer em código, o item precisa ter o problema e o plano no caderno.
- **Pular é permitido.** O Lucas pode dizer "já sei" (vai para a checagem) ou "pula este item" (a IA registra o motivo).
- **Tudo tem resposta certa para o cenário.** A IA não fica em cima do muro: diz se está certo, errado ou depende, e **de quê** depende.
- **Explicação detalhada** sempre que for preciso para o Lucas conseguir opinar.
- **A IA conduz, o Lucas manda.** Se ele pedir para mudar o rumo, a IA muda e registra.
