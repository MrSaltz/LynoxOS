# Desktop

> 🇺🇸 [Read in English](../../en/architecture/desktop.md)

Cobre entrada de mouse/ponteiro (M31) e a sessão de desktop viva
(M34.15–M34.16): a camada que transforma o sistema de janelas (ver
[window-system.md](window-system.md)) e o [toolkit de UI](ui.md) de um
conjunto de mecanismos independentemente testáveis em algo que uma pessoa
de fato usa, continuamente, movido por hardware real.

## De testes sintéticos a uma sessão viva

Pela maior parte da história do projeto, a pilha gráfica foi provada
correta através de testes automatizados: uma função de teste alimenta um
pacote de mouse sintético ou um evento de tecla diretamente no código
sendo testado e checa o resultado. Esse é o jeito certo de verificar cada
mecanismo isoladamente, mas é uma coisa fundamentalmente diferente de um
kernel que continua rodando, reagindo continuamente a o que um mouse e
teclado reais de fato mandam pra ele. M34.15 é o milestone que deu esse
salto: uma thread do scheduler (ver [threads.md](threads.md)) que nunca
retorna, consumindo entrada real de mouse e teclado movida por interrupção
e renderizando o resultado continuamente, em vez de um kernel que roda uma
suíte de testes e fica ocioso.

Esse salto revelou bugs reais que testes sintéticos estruturalmente não
conseguiriam ter encontrado — porque eles só existem quando entrada chega
na velocidade real de hardware, intercalada com interrupções reais,
sustentada ao longo do tempo, em vez de numa única chamada de função
controlada. Exemplos: um deadlock que só ocorre ao fechar uma aplicação de
dentro do mesmo lock que ela ainda precisa liberar; framing de pacote de
mouse que dessincroniza silenciosamente por causa de um único byte de
"confirmação" inesperado que vaza bem quando uma linha de interrupção é
ligada (não observável a menos que algo esteja de fato escutando naquela
IRQ quando isso acontece); e o cursor nunca sendo desenhado na tela, apesar
de todo o resto do tratamento de mouse funcionar, simplesmente porque nada
nos testes automatizados jamais precisou OLHAR pra tela. Verificar esse
milestone exigiu abandonar regressão automatizada headless em favor de
teste manual interativo — e cruzar comportamento entre dois hypervisors
independentes (QEMU e VirtualBox), especificamente pra separar um bug real
de kernel de um artefato de um ambiente de virtualização específico.

## Roteamento de entrada

Entrada de mouse vem de um driver PS/2 de verdade: pacotes são
decodificados, posição do cursor é rastreada, e cliques são roteados
através da ordem-z do sistema de janelas (ver
[window-system.md](window-system.md#o-compositor-e-a-ordem-z)) pra
determinar em qual janela — e, a partir daí, qual widget dentro daquela
janela (ver [ui.md](ui.md#modelo-de-interação)) — um clique ou evento de
tecla de fato cai. A sessão de desktop é dona desse roteamento; o
comportamento de foco/duplo-clique/pointer-capture em nível de widget em
si vive no toolkit de UI.

## Renderização: cadência de frame e um contador de FPS ao vivo

O loop de renderização era originalmente reativo — recompor a tela só
quando algo reportava atividade. Esse modelo produziu bugs visuais de
correção sob uso interativo sustentado (elementos somem brevemente
dependendo da posição do mouse) assim que damage tracking foi introduzido
(ver [window-system.md](window-system.md#damage-tracking)), e foi
substituído por um loop de cadência fixa: entrada ainda é processada toda
iteração, mas renderização é pautada num intervalo fixo, desacoplando
"quão rápido reagimos à entrada" de "quão rápido redesenhamos". Um
contador de FPS real, amostrado ao vivo, reporta a taxa de quadros
realmente alcançada — renderização por software num framebuffer emulado e
sem aceleração hoje sustenta aproximadamente 15–20 FPS, o que é aceito
como uma limitação conhecida e documentada desse estágio em vez de algo
pra continuar perseguindo; uma melhoria real aqui depende de trabalho de
renderização acelerada por hardware ou em nível de driver planejado pra
um milestone futuro.

## Aplicações e a taskbar

Cinco aplicações embutidas rodam em cima do desktop hoje — um File
Manager, um app de Settings, um System UI (mostrando estatísticas ao vivo,
incluindo o contador de FPS acima), um Terminal, e a própria Taskbar —
todas construídas sobre o mesmo [toolkit de UI](ui.md) e o mesmo
[sistema de janelas](window-system.md) que toda outra janela usa; nenhuma
delas é tratada como caso especial dentro do kernel. As entradas da
taskbar são clicáveis: clicar numa restaura/foca a janela correspondente,
reaproveitando a mesma lógica de layout que um painel de UI genérico usa
internamente, em vez de um layout feito à mão só pra taskbar.

## Onde isso aparece depois

- Ícones carregados (ver
  [graphics.md](graphics.md#o-sistema-de-imagemasset-m35-em-andamento))
  devem eventualmente substituir visuais provisórios na taskbar e nas
  barras de título de janela.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (intervalo M31, M33, M34).
