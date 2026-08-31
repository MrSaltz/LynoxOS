# 0004 — Imutabilidade de Image: convenção em vez de divisão em nível de tipo

> 🇺🇸 [Read in English](../../en/decisions/0004-image-immutability.md)

**Status**: Decidido (milestone Image / Asset System). **Área**: [gráficos](../architecture/graphics.md).

## Contexto

A abstração central da engine gráfica é uma `Surface` — qualquer coisa
que consegue reportar seu tamanho e ler/escrever pixels individuais —
implementada tanto pelo framebuffer real quanto por um buffer de desenho
em memória. Introduzir um asset de imagem carregado, efetivamente
imutável, levantou a questão de se essa imutabilidade deveria ser uma
garantia real, aplicada pelo sistema de tipos, ou uma convenção
documentada apoiada por um no-op em tempo de execução.

## Alternativas consideradas

- **(a) Reaproveitar o tipo de buffer de desenho mutável existente
  diretamente** como "o" tipo de imagem, sem tipo novo nenhum. Opção mais
  simples, mas confunde dois conceitos genuinamente diferentes (a surface
  mutável de uma janela, redesenhada todo frame, vs. um asset carregado e
  reusado).
- **(b) Dividir a trait `Surface` existente numa supertrait
  somente-leitura** (tamanho + leitura de pixel) e uma trait completa de
  leitura/escrita construída em cima dela, com o novo tipo de imagem
  implementando só a somente-leitura — tornando "você não pode escrever
  nisso" uma garantia de tempo de compilação, não uma convenção.
- **(c) Um novo tipo de imagem implementando a trait `Surface` completa
  existente**, com seu método de escrita de pixel sendo um no-op
  deliberado e documentado.

## O que aconteceu

A alternativa (b) foi tentada primeiro — parecia a escolha
arquiteturalmente "correta", já que tornaria imutabilidade uma garantia
real de sistema de tipos em vez de uma promessa documentada. Construir
ela revelou o custo de verdade: dezenas de sites de chamada por toda a
base de código que só importam a trait de leitura/escrita (porque essa
era a única que existia antes) precisariam cada um adicionalmente
importar a nova trait somente-leitura também, só pra continuar resolvendo
métodos que já chamavam — a resolução de método do Rust exige que a
trait específica que fornece um método esteja no escopo, não só qualquer
supertrait dela. Nenhum desses sites de chamada estava fazendo nada
errado; eles só precisariam da mudança pra continuar compilando.

## Decisão

**(c)** — o novo tipo de imagem implementa a trait `Surface` completa
existente, e seu método de escrita de pixel é um no-op documentado. Isso
significou que a operação genérica existente de "copiar uma região de
uma surface pra outra" (já usada por toda a engine gráfica) funcionou com
o novo tipo imediatamente, com zero mudanças na assinatura dela e zero
mudanças em qualquer um dos dezenas de sites de chamada não relacionados
que só precisavam de acesso de leitura em primeiro lugar.

## Consequências

- A garantia de imutabilidade é aplicada por documentação e cobertura de
  teste (um teste que deliberadamente chama a escrita no-op e confirma
  que nada mudou) em vez de pelo sistema de tipos.
- Isso foi julgado um trade-off aceitável especificamente porque nenhum
  consumidor real jamais tinha tentado escrever numa imagem carregada por
  engano — a garantia mais forte estaria resolvendo um problema que ainda
  não existia de verdade, a um custo real pra todo site de chamada
  existente.
- Se um bug real envolvendo escritas acidentais numa imagem carregada
  algum dia aparecer, a alternativa (b) — ou algum outro tipo de
  aplicação em nível de tipo — é a próxima coisa natural a revisitar,
  agora apoiada num caso concreto de por que vale o efeito cascata.
