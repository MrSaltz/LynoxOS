# Diretrizes de UI

> 🇺🇸 [Read in English](../../en/design/ui-guidelines.md)

Princípios de design específicos do [toolkit de UI](../architecture/ui.md)
e do [sistema de janelas](../architecture/window-system.md), em cima das
regras gerais em [`coding-principles.md`](coding-principles.md).

## Um tema compartilhado, sem estilo por widget

Todo widget desenha contra um único `Theme` compartilhado (cores,
espaçamento) em vez de fixar suas próprias constantes visuais. Isso não é
tanto uma preferência visual quanto uma garantia de consistência: um
widget novo não consegue acidentalmente parecer inconsistente com os já
existentes, porque ele nunca teve a opção de definir suas próprias cores
em primeiro lugar.

## Estado é visível, não só lógico

Estados de interação que importam funcionalmente (focado vs. não-focado,
com hover vs. pressionado, uma janela focada vs. uma que não está) sempre
recebem um tratamento visual distinto, não só rastreados internamente. Um
usuário nunca deveria precisar adivinhar qual janela vai receber a
próxima tecla, ou se um botão registrou um clique — o estado em que o
sistema realmente está e o estado em que ele visualmente parece estar são
mantidos em sincronia por construção.

## Mecanismos são construídos e provados independentemente antes de serem conectados

Todo mecanismo de gerenciamento de janela (arrastar, redimensionar,
minimizar/maximizar, encaixe nas bordas, bloqueio modal, dispensa de
popup) foi construído e verificado como sua própria unidade
independentemente testável antes de ser conectado à sessão de desktop
viva (ver [`docs/pt-br/architecture/desktop.md`](../architecture/desktop.md)).
Isso espelha a regra geral de "mudanças pequenas e isoladas" em
[`coding-principles.md`](coding-principles.md#mudanças-pequenas-e-isoladas),
aplicada especificamente a comportamento interativo: um bug encontrado
depois é muito mais fácil de localizar quando cada mecanismo já tem sua
própria cobertura de regressão de antes de ele ser combinado com os
outros.

## Sem saídas de emergência globais escondidas

Janelas modais bloqueiam interação com tudo atrás delas até serem
dispensadas — não existe porta dos fundos pra um evento alcançar uma
janela que ele não deveria conseguir receber entrada no momento. A mesma
disciplina que governa a
[segurança baseada em capabilities](../architecture/capabilities.md) (sem
acesso ambiente, só concessões explícitas) aparece aqui como uma regra de
UI: a capacidade de uma janela de receber entrada é explícita e aplicada,
não uma convenção que outro código deve respeitar.
