# Sistema de Janelas

> 🇺🇸 [Read in English](../../en/architecture/window-system.md)

Cobre a parte de janelas e composição da fase Graphics/Desktop (M28–M32) e
a fase Advanced Window System (M34): tudo que transforma uma coleção de
surfaces de desenho (ver [graphics.md](graphics.md)) em janelas
independentemente móveis, redimensionáveis e fecháveis na tela.

## Janelas como surfaces próprias

Cada janela é dona da sua própria surface de desenho offscreen (ver
[graphics.md](graphics.md#surfaces-uma-abstração-dois-destinos)) — uma
aplicação desenha na surface da sua própria janela sem nenhuma consciência
de onde aquela janela está na tela, se está visível no momento, ou quais
outras janelas existem. Essa separação (uma aplicação só desenha no seu
próprio buffer) é o que torna o trabalho do compositor bem definido: ele é
puramente responsável por organizar conteúdo de janela já renderizado num
frame final, nunca por nada que uma aplicação desenha.

## O compositor e a ordem-z

O compositor é responsável por pegar toda janela atualmente visível (cada
uma com sua própria posição, tamanho e surface de desenho) e produzir o
único frame final que é apresentado pro framebuffer real. Janelas são
mantidas numa ordem-z estrita (de trás pra frente) que também decide em
qual janela um clique cai (uma janela por cima ganha), e camadas especiais
existem por cima da ordem-z normal: janelas modais bloqueiam interação com
qualquer coisa atrás delas até serem dispensadas, e janelas always-on-top
ficam acima de janelas normais mesmo depois de perderem o foco, com modais
ainda tendo precedência sobre always-on-top quando os dois estão presentes.

## Damage tracking

Recompor toda janela visível no frame final a cada frame, independente de
algo ter realmente mudado, desperdiça trabalho que um desktop estático não
precisa pagar. Damage tracking registra quais REGIÕES da tela foram
realmente tocadas desde o último frame, e o compositor só redesenha
janelas dentro dessas regiões — e só a PORÇÃO sobreposta específica de
cada janela, não a janela inteira, o que importa mais pra uma janela em
tela cheia como o fundo do desktop: redesenhar ingenuamente "a janela
inteira" toda vez que ela meramente SOBREPÕE um pouco de dano (o que uma
janela em tela cheia sempre faz) apagaria silenciosamente o benefício de
performance que o damage tracking foi construído pra trazer. Acertar isso
exigiu duas iterações — a primeira correção (expandir a região de dano
pra cobrir uma janela inteira assim que ela registrasse como tocada)
resolveu um bug visual mas reintroduziu o problema de performance
descartado, exatamente por esse motivo; a versão que funciona só redesenha
a sub-região realmente sobreposta de cada janela, sem nenhuma expansão de
região.

## Chrome e ciclo de vida de janela

Além da abstração básica de janela, o sistema de janelas implementa o
conjunto completo de interações esperadas de uma janela de desktop de
verdade: quadros arrastáveis e redimensionáveis com barras de título
reais, minimizar/maximizar (com a posição/tamanho original lembrados pra
restaurar), encaixe nas bordas pra metade ou tela cheia, popups/menus que
se fecham sozinhos num clique fora sem também disparar o que está por
baixo, e um ciclo de fechamento interceptável — uma janela pode ser PEDIDA
pra fechar sem ser forçada, deixando uma aplicação decidir se confirma ou
veta o pedido. Barras de título, menus e popups são construídos a partir
do mesmo toolkit de widgets descrito em [ui.md](ui.md), não código de
desenho avulso específico pro chrome de janela. Cada um desses mecanismos
foi construído e verificado independentemente, como unidade testável,
antes de ser conectado à sessão de desktop viva (ver
[desktop.md](desktop.md)).

## Onde isso aparece depois

- A sessão de desktop (ver [desktop.md](desktop.md)) é o que de fato move
  esse compositor e sistema de janelas continuamente, a partir de
  interrupções reais de mouse/teclado em vez de entrada sintética de
  teste.
- Ícones e outros assets carregados (ver
  [graphics.md](graphics.md#o-sistema-de-imagemasset-m35-em-andamento))
  devem eventualmente aparecer no chrome de janela (barras de título, a
  taskbar) quando M35 chegar nesse ponto.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (intervalo M28–M34).
