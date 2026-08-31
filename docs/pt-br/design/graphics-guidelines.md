# Diretrizes de Gráficos

> 🇺🇸 [Read in English](../../en/design/graphics-guidelines.md)

Princípios de design específicos da [engine gráfica](../architecture/graphics.md)
e do [sistema de janelas](../architecture/window-system.md), em cima das
regras gerais em [`coding-principles.md`](coding-principles.md).

## Uma abstração, todo destino

Código de desenho é escrito uma vez contra a
[abstração `Surface`](../architecture/graphics.md#surfaces-uma-abstração-dois-destinos)
e nunca contra um destino concreto específico (o framebuffer real vs. um
buffer offscreen). Isso é o que torna double buffering, composição e
testes possíveis sem duplicar um único algoritmo de desenho — uma rotina
de preencher retângulo escrita uma vez funciona identicamente se a saída
dela algum dia chega numa tela ou fica só num buffer em memória de um
teste.

## Correção primeiro, trabalho de performance é medido, não adivinhado

Novas features de gráficos/composição são construídas pra correção
primeiro; uma regressão de performance só é aceita como corrigida depois
que é de fato MEDIDA como corrigida (um contador de FPS ao vivo — ver
[`docs/pt-br/architecture/desktop.md`](../architecture/desktop.md#renderização-cadência-de-frame-e-um-contador-de-fps-ao-vivo)
— existe especificamente pra isso não ser adivinhação). O histórico de
damage tracking do compositor deste projeto (ver
[0005 — Damage tracking](../decisions/0005-damage-tracking.md)) é o
exemplo cautelar concreto: uma correção que parecia uma melhoria natural
e incremental sobre um bug conhecido acabou silenciosamente regredindo a
própria propriedade de performance que deveria preservar, e isso só foi
capturado porque performance foi remedida, não assumida.

## O algoritmo correto mais simples, melhorado só quando algo real exigir

Escala de imagem nearest-neighbor, desenho de linha por Bresenham,
blitting pixel a pixel — em todo caso, o algoritmo mais simples que
produz um resultado CORRETO é escolhido em vez de um mais sofisticado,
até que uma necessidade real e demonstrada de mais qualidade ou
velocidade apareça. Essa é a instância específica de gráficos da regra
geral de "sem complexidade especulativa" (ver
[`coding-principles.md`](coding-principles.md#sem-complexidade-especulativa)):
um filtro de escala mais suave ou um caminho de cópia acelerado por
hardware é trabalho real e útil QUANDO ALGO PRECISA DELE — construir
antes disso só significa manter mais complexidade por um benefício que
ninguém pediu ainda.

## Dano, não força bruta

Assim que uma cena consegue mudar incrementalmente (janelas se movendo,
um cursor animando), redesenhar a tela inteira a cada frame independente
do que mudou é tratado como a coisa a evitar, não o padrão a que
recorrer. Damage tracking (ver
[`docs/pt-br/architecture/window-system.md`](../architecture/window-system.md#damage-tracking))
existe porque um desktop estático não deveria custar o mesmo pra
renderizar que um onde tudo está se movendo ao mesmo tempo — mas ver a
nota acima: uma otimização de damage tracking não é considerada "pronta"
até que seu benefício de performance seja de fato confirmado, não só sua
correção.
