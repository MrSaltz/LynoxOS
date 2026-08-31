# Threads & Concorrência

> 🇺🇸 [Read in English](../../en/architecture/threads.md)

Cobre a fase Concurrency (M8–M11): transformar um kernel que só roda uma
coisa por vez (ver [kernel.md](kernel.md)) num que roda várias coisas com
segurança, ao mesmo tempo, de forma preemptiva.

## O scheduler preemptivo

O Lynox usa um scheduler preemptivo — threads não precisam ceder o
processador voluntariamente; uma interrupção de timer (o PIT, programado
cedo no boot, ver [kernel.md](kernel.md#sequência-de-bring-up)) força uma
troca de contexto numa cadência regular. Threads são a unidade básica de
execução do kernel, e o próprio desktop é, no fim das contas, só mais uma
thread agendada (ver [desktop.md](desktop.md)) que nunca retorna.

Construir um scheduler preemptivo corretamente, sozinho, já revela bugs
reais de concorrência cedo — por isso essa fase vem logo depois de
Foundation, antes de qualquer coisa que dependa de threads não pisarem
umas nas outras (drivers, filesystem, pilha de rede, compositor) ser
construída.

## Primitivas de concorrência

Em cima do scheduler, o Lynox implementa o kit padrão que um kernel
precisa pra deixar múltiplas threads (e depois, handlers de interrupção)
tocarem estado compartilhado com segurança:

- **Bloqueio / wakeup** — uma thread pode se suspender esperando uma
  condição e ser acordada por outra thread ou por um handler de
  interrupção, em vez de ficar girando ocupada (busy-spinning).
- **Mutex** — exclusão mútua pra estruturas de dados do kernel.
- **Condvar** — variáveis de condição em cima de bloqueio/wakeup, pra
  "espera até essa coisa específica mudar" em vez de ficar checando um
  lock repetidamente.
- **Semáforo** — sincronização por contagem, usada onde o modelo binário
  de um mutex não encaixa (ex.: um pool limitado de recursos).
- **RwLock** — lock de leitor/escritor onde muitos leitores concorrentes
  são comuns e um escritor precisa de acesso exclusivo.

Um tema recorrente no projeto inteiro (ver
[`docs/pt-br/design/coding-principles.md`](../design/coding-principles.md))
é que essas primitivas são de fato USADAS, não só implementadas — milestones
posteriores (a fila de comandos do driver AHCI, a pilha TCP/IP, o
compositor, a sessão de desktop) exercitam elas sob carga real, concorrente
e movida por interrupção, que é onde bugs sutis (deadlocks, wakeups
perdidos, inversão de prioridade) realmente aparecem. A sessão de desktop
viva (ver [desktop.md](desktop.md)) achou um deadlock real desse jeito —
fechar uma aplicação de dentro de um lock que ela ainda precisava liberar
— que nenhum teste sintético single-threaded jamais tinha exercitado.

## Onde isso aparece depois

- Todo driver, a [camada de userspace](userspace.md), o
  [compositor](window-system.md) e a
  [sessão de desktop viva](desktop.md) rodam como threads agendadas e
  dependem dessas primitivas pra tocar estado compartilhado com
  segurança.
- [IPC](ipc.md) é uma preocupação relacionada mas distinta: essas
  primitivas sincronizam threads DENTRO de um mesmo espaço de endereço,
  enquanto IPC é sobre comunicação ENTRE processos separados e isolados.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (M8–M11).
