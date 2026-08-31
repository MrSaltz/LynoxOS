# 0005 — Damage tracking do compositor: blit parcial, não expansão de região

> 🇺🇸 [Read in English](../../en/decisions/0005-damage-tracking.md)

**Status**: Decidido (milestone Advanced Window System, estabilização da
sessão de desktop viva). **Área**: [sistema de janelas](../architecture/window-system.md).

## Contexto

O compositor originalmente redesenhava toda janela visível a cada frame,
independente de algo ter realmente mudado — correto, mas desperdiçador
assim que o sistema precisou sustentar taxas de quadro interativas.
Introduzir damage tracking (registrar quais regiões da tela realmente
mudaram desde o último frame, e só redesenhar essas) devia corrigir isso,
mas a primeira tentativa de implementação introduziu um bug real e
visível de correção pro usuário: elementos de UI (uma taskbar, texto de
status) desapareciam intermitentemente dependendo da posição do cursor.

## Alternativas consideradas

- **(a) Teste de interseção por janela, redesenha a janela inteira se
  ela intersecta a região suja de algum jeito.** A abordagem inicial. Ela
  quebrou a composição por ordem-z: redesenhar uma janela de fundo em
  tela cheia sem também redesenhar o que legitimamente está por cima dela
  NESSA MESMA REGIÃO deixava o fundo pintar silenciosamente por cima de
  elementos de primeiro plano sempre que os dois compartilhavam qualquer
  área suja — o que, pra uma janela em tela cheia, é essencialmente
  sempre (o cursor se movendo em qualquer lugar da tela conta como dano
  que uma janela em tela cheia "intersecta").
- **(b) Expandir a região de dano rastreada pra cobrir os limites
  inteiros de uma janela assim que qualquer parte dela registrar como
  tocada, iterando de trás pra frente.** Isso corrigiu o bug de elementos
  desaparecendo (correção restaurada), mas reintroduziu exatamente o
  problema de performance que damage tracking deveria resolver: como a
  janela de fundo é tela cheia, expandir dano pros limites inteiros dela
  efetivamente envenena a região rastreada de volta pra tela inteira em
  quase todo frame.
- **(c) Pra cada janela, calcular só a sub-região exata de sobreposição
  entre aquela janela e a área suja, e redesenhar SÓ essa sub-região
  recortada** — sem nenhuma etapa de expansão. A ordem de desenho de
  trás pra frente naturalmente repinta o resultado final correto dentro
  daquela área recortada, já que qualquer coisa sentada por cima do fundo
  ganha sua própria vez de desenhar por cima dos mesmos pixels.

## Decisão

**(c)** — recorta o redesenho de cada janela pra sua interseção exata
com a região suja, sem etapa de expansão nenhuma. Verificado
interativamente: elementos pararam de desaparecer, e a taxa de quadros se
recuperou da regressão introduzida por (b).

## Consequências

- Essa é a versão rodando atualmente: uma taxa de quadros real e medida
  na faixa de ~15–20 FPS num framebuffer emulado e sem aceleração — uma
  limitação conhecida e aceita de renderização por software nesse
  estágio, não uma regressão esperando ser corrigida revisitando essa
  decisão específica (ver
  [desktop.md](../architecture/desktop.md#renderização-cadência-de-frame-e-um-contador-de-fps-ao-vivo)).
- A lição geral generalizou pra além desse bug específico: sempre que uma
  correção é verificada só pra correção, vale a pena re-checar
  explicitamente que a propriedade de performance original que a mudança
  devia preservar de fato ainda se sustenta — (b) PARECIA uma correção
  natural e incremental sobre (a), e já era uma regressão da meta de
  performance antes de alguém medir diretamente.
