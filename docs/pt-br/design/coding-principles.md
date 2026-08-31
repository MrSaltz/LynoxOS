# Princípios de Código & Metodologia

> 🇺🇸 [Read in English](../../en/design/coding-principles.md)

Esta página descreve COMO o Lynox é construído, como processo — as regras
que se aplicam a todo milestone, independente de qual subsistema ele toca.
Ver [`docs/pt-br/architecture/`](../architecture/overview.md) pro que já
foi construído, [`docs/pt-br/design/ui-guidelines.md`](ui-guidelines.md) e
[`docs/pt-br/design/graphics-guidelines.md`](graphics-guidelines.md) pros
princípios de design específicos de cada subsistema, e
[`docs/pt-br/decisions/`](../decisions/) pras decisões específicas
tomadas sob essas regras.

## A disciplina por submilestone

Todo submilestone no roadmap segue a mesma sequência:

1. **Objetivo** — que problema isso resolve, dito concretamente.
2. **Alternativas reais** — as opções de verdade consideradas, não um
   espantalho ao lado da abordagem escolhida. Se só existe uma opção
   razoável, isso é dito explicitamente em vez de preenchido com
   alternativas falsas.
3. **Decisão justificada** — qual opção foi escolhida, e por quê, em
   termos do trade-off de verdade (não "é mais limpo" sem dizer mais
   limpo COMO, ou a que custo).
4. **Arquitetura** — a forma da solução antes de escrevê-la.
5. **Implementação.**
6. **Verificação** — ver abaixo; essa etapa é tratada como
   não-negociável.
7. **Documentação** — a decisão, as alternativas rejeitadas e o que foi
   de fato verificado ficam escritos, não só o código resultante.

## Verificação significa rodar, não compilar

Um build bem-sucedido prova que o código é sintaticamente e tipicamente
válido — não prova nada sobre se a feature de fato funciona. Todo
milestone é verificado realmente inicializando o sistema (tipicamente
headless no QEMU, checado contra um marcador esperado de sucesso/falha na
saída serial) e, pra qualquer coisa tocando código voltado a hardware
real, repetido várias vezes pra capturar falhas intermitentes que uma
única execução perderia. Milestones que tocam comportamento interativo
(entrada de mouse/teclado, a sessão de desktop viva) foram verificados à
mão, interativamente, especificamente porque algumas classes de bug só
aparecem sob uso real, sustentado e movido por interrupção — nenhuma
quantidade de teste automatizado sintético teria encontrado eles (ver
[`docs/pt-br/architecture/desktop.md`](../architecture/desktop.md) pra
um exemplo concreto).

## Sem complexidade especulativa

Uma regra recorrente, referenciada ao longo do changelog como "nenhuma
feature é adicionada porque seria legal": toda abstração nova precisa de
um consumidor concreto ou uma necessidade arquitetural real NO MOMENTO EM
QUE É CONSTRUÍDA, não uma hipotética futura. Quando uma escolha de design
só compensaria pra um consumidor que ainda não existe, a opção mais
simples é escolhida agora, e a mais geral é adiada até que algo real
realmente precise dela. Isso aparece constantemente também em decisões
menores — ex.: escolher o algoritmo correto mais simples (escala de
imagem nearest-neighbor, desenho de linha por Bresenham) em vez de um
mais sofisticado, porque nada demonstrou que a qualidade da versão mais
simples é de fato um problema ainda — ver
[`docs/pt-br/design/graphics-guidelines.md`](graphics-guidelines.md) pra
como isso se manifesta especificamente na engine gráfica.

## Mudanças pequenas e isoladas

Todo submilestone é escopado pra ser independentemente compreensível e
independentemente verificável. Quando uma tentativa de implementação
revela que o design "mais correto" exigiria tocar dezenas de sites de
chamada não relacionados pela base de código inteira, por um benefício
que nada precisa hoje, isso é tratado como um sinal pra recuar e escolher
a opção menor e mais contida em vez disso — ver
[`docs/pt-br/decisions/`](../decisions/) pra um exemplo específico desse
trade-off acontecendo de verdade.

## Documentação honesta de falha

Bugs encontrados durante verificação — inclusive os que acabaram sendo
regressões auto-infligidas introduzidas mais cedo na mesma sessão — são
registrados no changelog e nos documentos de arquitetura como achados
reais, com o que causou eles e como foram corrigidos, em vez de
silenciosamente corrigidos e deixados sem documentação. O changelog é
escrito pra ser lido como um relato honesto do que de fato aconteceu, não
uma lista de features polida depois do fato.

## Como isso parece de fora

O resultado prático dessas regras é um projeto onde todo milestone em
[`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md)
corresponde a uma unidade de trabalho real, verificada e documentada, não
um apanhado de destaques curado.
