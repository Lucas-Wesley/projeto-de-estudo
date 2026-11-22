# Metodologia do Projeto de estudo

> Como o Lucas estuda e constrói o ecossistema Banco, Nuvem e Escola com a IA.
> Definida pelo Lucas em 21/11/2026. **Vale para todo item do [ROADMAP](ROADMAP.md)**: código, terminal, configuração, desenho, qualquer coisa.
> **Versão 2** (22/11/2026), depois da revisão de três IAs (`revisoes/`): três tipos de item, abertura de sessão com recuperação, o Lucas escreve a decisão e o que aprendeu, a escada de ajuda, a faixa de leitura dos livros e a regra dos tópicos (P10).

---

## 1. A ideia

O projeto é grande, então ele é resolvido **item por item, em pedaços pequenos**. Não há temporadas nem marcos: **o progresso é cada item construído e conferido funcionando**. Cada item segue o ciclo, no tamanho que o tipo dele pede (seção 3):

```
problema → estudo → checagem → caminhos → decisão → plano → execução → fechamento
```

O Lucas **estuda porque precisa decidir** e **decide antes de implementar** (ou, nos experimentos, decide depois de medir). A IA não entrega a resposta pronta: ela mostra o problema, indica o que estudar e prova por quê, confere se ele aprendeu, discute os caminhos, pergunta como ele faria e confere cada passo que ele executa.

Há dois lados, e eles conversam **por documentos**:

| Lado | Onde | Papel |
|---|---|---|
| **Plataforma** | Trilha › Projeto de estudo (seção 6) | Mostra o roadmap, em que item estamos, o que estudar (com link para a etapa) e **gera o prompt** que abre a conversa com a IA |
| **IA** | Claude Code, na IDE, nesta pasta | Conduz o ciclo do item, olha o código de verdade, confere a execução e **registra tudo nos arquivos** |

A IA **não** fica dentro da plataforma: no Claude Code ela já enxerga o código, o terminal e os documentos, e o prompt gerado só diz em que ponto estamos.

---

## 2. Os documentos (a memória do projeto)

Tudo o que importa fica em arquivo, para que qualquer sessão nova da IA (ou o Lucas daqui a um mês) saiba **o que queremos fazer, o caminho, onde paramos e qual é o próximo passo**.

| Arquivo | O que guarda | Quem atualiza |
|---|---|---|
| [CLAUDE.md](CLAUDE.md) | As instruções da IA: o que ler, como conduzir, o que registrar | Só muda com a metodologia |
| [METODOLOGIA.md](METODOLOGIA.md) | Este documento | Lucas e IA, quando a metodologia mudar |
| [ROADMAP.md](ROADMAP.md) | Os itens, em ordem, com o tipo, os tópicos, o critério de pronto e as fontes primárias; o fio da arquitetura; a faixa de leitura | IA propõe, Lucas aprova |
| [ESTADO.md](ESTADO.md) | **Onde paramos**: o item atual, a fase, o próximo passo, o próximo capítulo da faixa e as dúvidas abertas | IA, **ao fim de toda sessão** |
| [REPERTORIO.md](REPERTORIO.md) | O que o Lucas **já fez**, com que autonomia e quando foi a última vez | IA registra, Lucas confirma |
| [ERROS.md](ERROS.md) | "Eu achava X, era Y": os erros da checagem e da execução. Alimenta as perguntas de recuperação | IA registra na hora |
| `itens/<id>-<nome>.md` | O caderno do item (a partir de [itens/_MODELO.md](itens/_MODELO.md)) | IA, durante o item; as partes marcadas são do Lucas |
| [DECISOES.md](DECISOES.md) | Uma linha por decisão, com o porquê e o gatilho (o detalhe fica no item) | IA, na fase de decisão |

Regra de ouro: **ao terminar uma fase, a IA escreve no caderno do item; ao terminar a sessão, atualiza o ESTADO.md.** A conversa pode acabar a qualquer hora; os arquivos não.

**Precedência:** se dois documentos divergirem, vale METODOLOGIA > ROADMAP > ESTADO > caderno do item, e a IA avisa a divergência.

---

## 3. O ciclo de um item

### 3.0 Abertura de toda sessão
1. **Três perguntas de recuperação**, sem consulta: de itens de 2 a 6 semanas atrás e do [ERROS.md](ERROS.md). A IA diz certo, errado ou incompleto, e o erro vai para o ERROS.md.
2. **O objetivo da sessão:** o que fica pronto hoje.
3. **A faixa de leitura:** qual é o próximo capítulo (ROADMAP §7.3) e se o anterior foi lido.

São 5 minutos. Se o Lucas pedir para pular, pula.

### Os três tipos de item
Cada item do roadmap diz o seu **tipo**, e o ciclo roda no tamanho certo:

| Tipo | Quando | O ciclo |
|---|---|---|
| **Prática** | Técnica conhecida, sem decisão de verdade (criar o repositório, um Dockerfile, um CRUD a mais) | problema → estudo mínimo → plano → execução → fechamento |
| **Decisão** | Há caminhos de verdade e a escolha custa caro para desfazer (esquema, saga, fronteiras) | o ciclo inteiro |
| **Experimento** | A decisão só aparece construindo (lock pessimista x otimista, event sourcing, B+tree x LSM, Kubernetes x a sua Nuvem) | problema → estudo mínimo → **protótipo dos caminhos, medido** → decisão com os números → plano → execução → fechamento |

A IA pode propor mudar o tipo de um item ("este parecia prática, mas tem uma decisão de verdade") e o Lucas decide.

### 3.1 Problema
A IA apresenta o item: **o que** vamos construir, **por que agora** (o que no projeto pede isso) e **onde ele encaixa no todo** (o que vem antes, o que depende dele). Curto e concreto.

### 3.2 Estudo
1. **Diagnóstico primeiro.** Antes de indicar qualquer fonte, a IA descobre o que o Lucas já sabe: 2 ou 3 perguntas, ou uma tarefa de 10 minutos ("escreva o rateio de R$ 10,00 por 3"). O que ele já faz não entra no estudo.
2. **Tentativa ingênua** (itens de decisão e experimento): 30 a 45 minutos tentando do jeito que ele acha, **antes** de estudar. O estudo depois rende mais, porque ele sabe o que procurar.
3. **O que estudar.** A IA indica os tópicos, sempre no formato `Tn › Módulo › Etapa`, e **prova a necessidade**: "você precisa saber **A, B e C** porque, na fase de caminhos, vai ter de decidir **X**". O foco é sempre a decisão ou a ação que aquele estudo destrava.
   - A IA escolhe entre **todas** as trilhas, não só as listadas no roadmap.
   - Antes de indicar, ela **consulta a Trilha** (`GET https://trilha.internal/api/trilhas/{id}`) para ver o que já está concluído.
   - **O material** (curso, vídeo, documentação) é escolhido **na hora, pelo Lucas** (P10). A IA sugere a fonte primária do item ("Consultar" no roadmap) como preferência.
4. **Teto.** O estudo antes de codar fica em cerca de **um terço** do tempo do item. O resto é sob demanda, durante a execução.

A lista escolhida vai para a tabela "2. Estudo" do caderno do item, que a aba da Trilha mostra como "O que estudar, nesta ordem".

### 3.3 Checagem
Quando o Lucas diz que estudou (ou que **já sabe**), a IA **confirma que ele aprendeu**: perguntas de cenário ("o que acontece se…?"), não de decoreba. Para cada resposta, a IA diz **certo, errado ou incompleto**, corrige e **cita a fonte** (o capítulo, a documentação, um experimento). Se faltar base, volta ao estudo com a fonte exata. Os erros vão para o ERROS.md.

### 3.4 Caminhos
A IA apresenta **2 a 4 caminhos** possíveis, cada um com **vantagens e desvantagens**. Elas são consequências técnicas, com a fonte; quando for **julgamento** (e muita decisão de arquitetura é), a IA diz que é julgamento.

Nos itens que constroem uma peça que existe pronta, **"usar a peça pronta" é sempre um dos caminhos**.

Depois vem a **argumentação**, nesta ordem:
1. **O Lucas escreve a decisão dele** e o porquê no caderno, com um pré-mortem curto ("como isso pode dar errado?").
2. Só então a IA questiona, aponta o que ele não considerou e diz qual é o **melhor caminho para este cenário**, e por quê.
3. Se os dois divergirem, o caderno registra a divergência e, no fechamento, quem estava certo.

### 3.5 Decisão
A decisão fica registrada:
- no **ADR** do repositório, **escrito pelo Lucas** (em inglês, P6); a IA critica e aponta os furos;
- em uma linha no [DECISOES.md](DECISOES.md): o caminho, **por quê**, os descartados e **o gatilho de evolução** (o que nos faria mudar; seção 5).

### 3.6 Plano
A IA **não** dá o passo a passo. Ela pergunta: **"como você implementaria isso? O que vem primeiro?"** O Lucas responde na teoria. Se estiver certo, segue; se não, a IA corrige a linha de raciocínio; se ele não souber, a IA explica e seguem.

O resultado é a lista de passos **construída junto**, cada um com o que fazer, o critério de pronto e quem faz (seção 4).

### 3.7 Execução
O Lucas faz (código, comando, configuração), de preferência **em blocos** do plano, e mostra o resultado como um pull request: a IA revisa o diff, roda os testes, olha a saída e o estado do serviço, e diz o que está certo, o que está errado e por quê.

Se o Lucas travar, a ajuda segue uma **escada**, um degrau por vez:
1. uma dica;
2. uma pergunta que guia;
3. o esqueleto (assinaturas, pseudocódigo);
4. o código.

O degrau usado vai para o caderno. Quando a IA escreve um trecho, **o Lucas explica o trecho de volta** antes de seguir: revisar não é o mesmo que entender.

### 3.8 Fechamento
O item fecha quando o critério de pronto do roadmap é cumprido.
1. **O Lucas escreve o "O que aprendi"**: 10 linhas, como se explicasse a um júnior. A IA aponta os furos.
2. A IA completa o caderno (o que foi feito, os degraus de ajuda, o que ficou para depois) e anota **3 perguntas do item** para as recuperações futuras.
3. Atualiza o [REPERTORIO.md](REPERTORIO.md) (com a data) e o [ERROS.md](ERROS.md).
4. **Confere os gatilhos abertos** no DECISOES.md: algum disparou com o que acabamos de fazer?
5. Marca o item como feito no roadmap (✅) e aponta o próximo no [ESTADO.md](ESTADO.md).
6. Lembra o Lucas de marcar na Trilha as etapas estudadas e os capítulos lidos na faixa.

### 3.9 Fechamento de uma fase
No fim de cada fase do roadmap, um **design doc curto** (uma página: o que existe, as decisões, o que dói) no repositório, escrito pelo Lucas. É o ensaio do 9.2.

---

## 4. Quem faz o quê

| Situação | Quem faz |
|---|---|
| O conceito que o item ensina (o núcleo) | **Lucas** escreve; a IA confere |
| Dockerfile, CI, IaC, manifestos, scripts de deploy | **Lucas**, na primeira vez: para quem quer plataforma, isso é conteúdo, não periferia |
| Algo que o REPERTORIO mostra no nível **domino** (o Lucas fez mais de uma vez sem ajuda) | A IA **propõe** adiantar ("você já fez isto no 1.1 e no 2.6; posso adiantar?"). Abaixo de "domino", a repetição é a prática: o Lucas refaz |
| Algo novo para o Lucas | A IA avisa que é novo e o Lucas faz |
| Interfaces: painéis, telas, CSS (P5) | A IA escreve e o Lucas revisa |
| Testes de aceitação | O Lucas faz **o primeiro**; depois a IA escreve e o Lucas revisa |
| Revisão de código | A IA, aqui no Claude Code, a cada bloco da execução |

A IA **sempre avisa** antes de adiantar algo e **nunca** adianta o núcleo de um item novo sem o Lucas pedir.

**Katas:** uma sessão por semana **sem IA**, de 20 a 30 minutos: refazer de memória uma peça já feita (o rateio, a transferência com locks ordenados, o rate limiter, o LRU) ou um desafio dos Gossip Glomers. Quando passa sem ajuda, o REPERTORIO sobe de nível.

---

## 5. A progressão: tudo evolui, inclusive a arquitetura

O estudo é **coerente e cumulativo**: cada item se apoia nos anteriores, sem conceitos soltos fora de hora.

A arquitetura **existe desde o dia 1**, consciente ou não. Por isso ela é tratada como um **fio contínuo**:

1. Cada fase do roadmap começa com uma arquitetura que **dá conta até certo ponto**.
2. Toda decisão registra o seu **gatilho de evolução** ("quando acontecer X, esta escolha deixa de servir").
3. Quando o gatilho dispara, **a pressão vira um item**.
4. O roadmap diz, para cada estágio, se a pressão é **real** (um teste ou uma medida que falha) ou um **exercício deliberado** (fazemos para aprender). O ADR usa essas palavras: não se inventa pressão.
5. **Fitness functions** (testes que verificam a arquitetura, como "o domínio não importa infra") rodam no CI e protegem o que já foi decidido.

A sequência planejada está no [ROADMAP](ROADMAP.md), seção "O fio da arquitetura".

---

## 6. A plataforma: Trilha › Projeto de estudo

Uma opção no menu da Trilha (https://trilha.internal/#/projeto).

**O que ela mostra**
- As fases e os itens do roadmap, com o estado de cada um (a fazer · em andamento, com a fase do ciclo · feito).
- No item: o tipo, o que implementar, o critério de pronto, as fontes primárias, o estudo (**"O que estudar, nesta ordem"** quando o caderno já tem a lista; antes disso, os candidatos agrupados por trilha) e o caderno do item.
- O ESTADO atual em destaque e o progresso dos três livros.

**O botão principal: "Conversar com a IA"**
Gera o texto do prompt (modelo em [PROMPT.md](PROMPT.md)) já preenchido. O Lucas copia, cola no Claude Code e a conversa segue a metodologia.

**De onde vêm os dados**
- **Os documentos deste caderno são a fonte da verdade.** A plataforma lê, não escreve.
- **Onde mora cada coisa** (P3): o caderno é o repositório `projeto-de-estudo`, separado do app Trilha. O código fica em `codigo/<repositório>`, cada um com o seu README e `docs/adr/`. O caderno guarda **o processo de aprender**; o repositório do código guarda **o que foi decidido e construído**. O caderno do item aponta os commits e os ADRs.
- O progresso do **estudo** é o da própria Trilha (etapas concluídas). A IA consulta pela API.

---

## 7. Regras gerais

- **Nada começa sem estar descrito.** Antes de mexer em código, o item precisa ter o problema e o plano no caderno.
- **Pular é permitido.** O Lucas pode dizer "já sei" (vai para a checagem) ou "pula este item" (a IA registra o motivo).
- **Fonte ou julgamento.** A IA diz se está certo, errado ou depende, e **de quê** depende, citando a fonte. Quando não houver fonte, diz que é julgamento. Se não souber, diz que não sabe.
- **Explicação detalhada** sempre que for preciso para o Lucas conseguir opinar.
- **A IA conduz, o Lucas manda.** Se ele pedir para mudar o rumo, a IA muda e registra.

---

## 8. Material novo (um vídeo, uma tecnologia, um curso)

O Lucas pode mandar qualquer coisa que achou interessante. A IA encaixa assim:

1. **Já existe um item que trata disso?** O material vira fonte extra daquele item (seção 7.1 do roadmap).
2. **É algo novo que cabe no ecossistema?** Vira um item novo, na fase em que ele faz sentido, com o mesmo formato dos outros.
3. **Não cabe no projeto?** Vai para a trilha certa na Trilha, fora do roadmap, e a IA diz por que não entrou.

Em qualquer caso: **os capítulos dos três livros nunca saem.** O projeto tem fim, e o fim inclui os três livros lidos (pela faixa, ROADMAP §7.3).

---

## 9. Tópicos e material (P10)

- **Todo tópico que o projeto pede está numa trilha ou num dos três livros.** É o mais importante: o tópico bem definido, no lugar certo.
- Quando falta um tópico, ele entra como **etapa na trilha certa**; se nenhuma trilha serve, nasce uma **trilha nova**. O item que o usa passa a citá-lo.
- **O material vem depois**, na hora do estudo, e quem escolhe é o Lucas: os cursos que ele já tem, vídeos, documentação, um curso novo. A preferência é a **fonte primária** que o item indica em "Consultar".
