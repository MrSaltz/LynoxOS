# Toolkit de UI

> 🇺🇸 [Read in English](../../en/architecture/ui.md)

Cobre a fase UI Interaction (M33) e o toolkit de widgets próprio construído
junto com o [sistema de janelas](window-system.md): a camada que
aplicações de fato usam pra montar sua interface, em vez de chamar
primitivas de gráficos (ver [graphics.md](graphics.md)) diretamente.

## Um toolkit de widgets próprio

Em vez de cada aplicação desenhar seus próprios botões e labels à mão, o
Lynox tem um toolkit de widgets pequeno e reusável: um `Theme`
compartilhado (cores, espaçamento) contra o qual todo widget desenha, um
comportamento base `Widget` que todo controle implementa, e um conjunto de
widgets concretos — `Label`, `Panel` (um container que organiza widgets
heterogêneos), `Button`, `TitleBar` e `Menu`. Cada um desses desenha
através da mesma [abstração `Surface`](graphics.md#surfaces-uma-abstração-dois-destinos)
que tudo mais na engine gráfica usa — um widget não faz ideia se está
sendo desenhado na surface viva de uma janela ou num buffer offscreen pra
um teste.

## Modelo de interação

Um widget sendo desenhado corretamente e um widget RESPONDENDO a entrada
corretamente são problemas diferentes, tratados como sua própria etapa:

- **Foco de teclado** — exatamente um widget por vez pode ter o foco, e
  só o widget focado recebe eventos de teclado; visualmente, um widget
  focado é distinguível de um que só tem o mouse por cima.
- **Duplo-clique** — reconhecido por uma checagem em nível de widget de
  janela-de-tempo-mais-distância (um segundo clique dentro de um tempo
  curto e um raio pequeno de pixels conta como duplo-clique), avaliado
  como uma função pura, testável independentemente, em vez de algo
  embutido no loop de tratamento de entrada.
- **Pointer capture** — assim que um botão é pressionado, ele continua no
  estado visual e lógico de "pressionado" até o botão do mouse ser solto,
  mesmo que o cursor saia brevemente da área do widget antes — batendo com
  o comportamento de toolkits de UI reais, e evitando uma classe de bug
  onde um arrasto que sai dos limites de um botão é lido erroneamente
  como um soltar.
- **Entrada unificada** — entrada de mouse e teclado são canalizadas por
  um único ponto de entrada por widget, em vez de cada widget implementar
  seu próprio caminho separado de tratamento de mouse e de teclado.

## Onde isso aparece depois

- O chrome de janela (barras de título, menus, popups — ver
  [window-system.md](window-system.md#chrome-e-ciclo-de-vida-de-janela))
  é construído a partir desses mesmos widgets, não código de desenho
  avulso separado.
- As aplicações embutidas do desktop e a taskbar (ver
  [desktop.md](desktop.md)) são os consumidores reais e essenciais do
  toolkit — cada uma delas é construída a partir de `Panel`/`Label`/
  `Button` em vez de código de desenho feito à mão.
- Ver [`docs/pt-br/design/ui-guidelines.md`](../design/ui-guidelines.md)
  pros princípios visuais e de interação que o toolkit segue.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (M33).
