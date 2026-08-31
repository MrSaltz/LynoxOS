# IPC

> 🇺🇸 [Read in English](../../en/architecture/ipc.md)

Cobre a primitiva de comunicação entre processos introduzida na fase
Concurrency (M11): como processos separados e isolados (ver
[processes.md](processes.md)) conversam entre si, diferente de como
threads DENTRO de um processo se sincronizam (ver [threads.md](threads.md)).

## Por que o IPC precisava ser desenhado cedo, não encaixado depois

O modelo de kernel do Lynox é híbrido, com disciplina de microkernel
aplicada nas fronteiras (ver
[0003 — Modelo de kernel](../decisions/0003-kernel-model.md)): um núcleo
pequeno cuida de CPU, memória, scheduling e primitivas de segurança,
enquanto drivers e serviços são arquitetados como se fossem componentes
separados se comunicando por IPC — mesmo os que hoje rodam em espaço de
kernel por simplicidade de bring-up. Se "mover esse driver pra userspace
depois" vai ser uma migração fácil ou uma reescrita completa é decidido
quase inteiramente por o IPC ter sido desenhado como uma primitiva real
desde o início. Encaixar IPC depois que um sistema já cresceu em torno de
chamadas de função diretas e memória compartilhada é uma das formas mais
comuns de um projeto de kernel híbrido silenciosamente voltar a ser um
monolito na prática, independente da arquitetura declarada.

## Referência de design

O design de IPC do Lynox usa o modelo de handles/channels usado por
microkernels modernos orientados a capabilities como ponto de partida pra
avaliação (Zircon/Fuchsia é a referência específica) — não pra copiar por
completo, mas porque esse design já resolveu o problema específico que o
Lynox precisa resolver: comunicação eficiente entre processos isolados,
respeitando capabilities, desenhada como primitiva de primeira classe em
vez de um acréscimo. Toda operação de IPC é checada contra o mesmo modelo
de capabilities que o resto da superfície de syscall usa (ver
[capabilities.md](capabilities.md)) — um processo só consegue se comunicar
por um channel pro qual ele realmente tem uma capability.

## Onde isso aparece depois

- Drivers e serviços de sistema que são arquiteturalmente separados do
  núcleo do kernel (ver [userspace.md](userspace.md)) dependem dessa
  camada pra falar de volta com o kernel e entre si.
- O modelo de capabilities (ver [capabilities.md](capabilities.md)) é o
  que torna um endpoint de IPC um recurso controlado em vez de um canal
  ambiente que qualquer coisa alcança.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (M11).
