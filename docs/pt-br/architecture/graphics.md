# Gráficos

> 🇺🇸 [Read in English](../../en/architecture/graphics.md)

Cobre a fundação de renderização da fase Graphics/Desktop (M27) e o Image/
Asset System em andamento (M35): tudo abaixo do gerenciador de janelas
(ver [window-system.md](window-system.md)) responsável por transformar
dados de pixel em algo na tela.

## Ponto de partida: o framebuffer

O framebuffer GOP (Graphics Output Protocol) do UEFI, entregue ao kernel
pelo Limine no boot (ver [overview.md](overview.md#fluxo-de-boot)), é o
único destino real de pixels no hardware de verdade. Tudo na engine
gráfica é desenhado pra que esse framebuffer seja só UM destino possível
entre vários — o mesmo código de desenho que mira a tela física também
mira um buffer em memória, o que é o que torna double buffering e um
compositor possíveis (ver abaixo).

## Surfaces: uma abstração, dois destinos

Uma `Surface` é qualquer coisa que consegue reportar seu próprio tamanho e
responder "qual a cor do pixel em (x, y)" / "defina o pixel em (x, y) pra
essa cor". O framebuffer físico implementa isso; um buffer de pixel
offscreen, puramente em memória, também implementa. Toda primitiva de
desenho (retângulos preenchidos/contornados, linhas, texto, copiar uma
região retangular de uma surface pra outra) é escrita uma vez, de forma
genérica, contra essa abstração — ela não faz ideia se está desenhando na
tela de verdade ou num buffer que ainda não foi mostrado a ninguém.

Essa operação genérica de "copiar uma região de uma surface pra outra" (um
*blit*) é o único mecanismo em que toda peça de nível mais alto da engine
gráfica se apoia: apresentar um frame offscreen terminado pro framebuffer
real é um blit; o compositor desenhando o conteúdo de uma janela no frame
final é um blit; escalar e desenhar uma imagem carregada (abaixo)
reaproveita a mesma operação exata, sem nenhuma mudança na assinatura.

## Por que essa divisão importa: double buffering e damage tracking

Como uma `Surface` não se importa com o que ela representa, o sistema
consegue desenhar um frame inteiro num buffer offscreen primeiro e só
copiar (blit) o resultado terminado pro framebuffer real uma vez — evitando
o tearing/flicker visível de desenhar incrementalmente direto na tela. Em
cima disso, o compositor rastreia quais REGIÕES da tela realmente mudaram
entre frames (ver [window-system.md](window-system.md#damage-tracking)),
pra que um desktop estático não pague o custo de redesenhar tudo que não
mudou.

## O sistema de Imagem/Asset (M35, em andamento)

Tudo acima trata uma `Surface` como algo redesenhado todo frame — o modelo
certo pro conteúdo de uma janela, errado pra um asset carregado como um
ícone, que é decodificado uma vez e depois reusado sem mudança, possivelmente
em vários lugares ao mesmo tempo (o mesmo ícone numa entrada da taskbar e
na barra de título de uma janela, por exemplo).

M35 introduz `Image`: um asset carregado, efetivamente imutável, distinto
da surface de desenho mutável de uma janela, mas implementando
deliberadamente a mesma abstração `Surface` — pra que toda primitiva de
desenho existente (blitting, em particular) já funcione com ela, sem
nenhum código novo pra essa parte. Uma alternativa de design inicial teria
dividido `Surface` numa trait separada, somente-leitura, especificamente
pra fazer a imutabilidade de uma imagem ser aplicada pelo sistema de
tipos em vez de por convenção; foi rejeitada assim que ficou claro que o
efeito cascata (todo site de chamada existente na base de código que só
importa a trait mutável precisaria importar a nova também, só pra
continuar resolvendo métodos que já usava) custava muito mais que o
benefício, já que nada jamais tinha tentado escrever numa imagem por
engano. Esse tipo de trade-off — um design teoricamente mais limpo versus
um que não exige tocar dezenas de sites de chamada não relacionados —
aparece com frequência suficiente nesse projeto que tem seu próprio
registro: ver
[0004 — Imutabilidade de Image](../decisions/0004-image-immutability.md).

A partir de M35.1–M35.6: o tipo `Image` existe, conversão de formato de
pixel (RGB/BGR, com e sem byte de alpha) entre bytes crus e a
representação interna de cor da engine está implementada, bytes crus
podem ser carregados do [filesystem](filesystem.md) e decodificados numa
`Image` usável, e escala nearest-neighbor produz uma cópia redimensionada
de qualquer surface. Composição de transparência/alpha, um pipeline de
ícone/asset e um cache de decodificação são os próximos passos planejados
— ver [`roadmap/pt-br/roadmap.md`](../../../roadmap/pt-br/roadmap.md) e
[`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md)
pra lista completa.

## Onde isso aparece depois

- O compositor do sistema de janelas (ver
  [window-system.md](window-system.md)) é o principal consumidor de
  surfaces, blitting e damage tracking.
- O [toolkit de UI](ui.md) e as aplicações do desktop (ver
  [desktop.md](desktop.md)) desenham através dessa mesma camada de
  primitivas.
