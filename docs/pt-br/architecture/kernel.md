# Núcleo do Kernel

> 🇺🇸 [Read in English](../../en/architecture/kernel.md)

Cobre a fase Foundation (M0–M7): a parte do sistema que existe antes de
haver um scheduler, um filesystem, uma pilha de rede ou uma tela pra
desenhar. Pro que é construído diretamente em cima disso, ver
[threads.md](threads.md) (scheduling e concorrência) e [ipc.md](ipc.md)
(comunicação entre processos).

## Sequência de bring-up

Depois que o Limine passa o controle (ver
[overview.md](overview.md#fluxo-de-boot)), o kernel é responsável por tudo
que um ambiente bare-metal não dá de graça:

1. **GDT/IDT** — o kernel monta sua própria Global Descriptor Table e
   Interrupt Descriptor Table em vez de confiar no que o bootloader deixou,
   já que precisa de controle total sobre semântica de segmento/privilégio
   e vetores de exceção/interrupção daqui pra frente.
2. **Console serial** — o canal de saída mais cedo possível (COM1), usado
   antes de existir um console de framebuffer.
3. **Interrupções** — o PIC (e depois tratamento adjacente ao APIC) é
   programado pra que interrupções de hardware (timer, teclado, mouse,
   disco, rede) consigam de fato chegar no kernel em vez de disparar
   comportamento indefinido.
4. **Memória física** — um frame allocator (baseado em bitmap: um bit por
   frame físico, rastreando livre/usado) transforma o memory map cru que o
   Limine forneceu em algo que o resto do kernel consegue pedir páginas.
5. **Heap** — um heap allocator do kernel fica em cima do frame allocator
   pra que a crate `alloc` do Rust (`Vec`, `Box`, `Rc`, etc.) funcione
   dentro do kernel, o que é o que torna escrever código de kernel em Rust
   idiomático viável de verdade num ambiente `no_std`.

## Onde isso aparece depois

- O frame allocator sustenta a [memória virtual](memory.md).
- A interrupção de timer programada aqui é o que move o scheduler
  preemptivo (ver [threads.md](threads.md)).
- A infraestrutura de interrupção montada aqui é por onde todo driver, a
  [camada de userspace](userspace.md) e a
  [sessão de desktop viva](desktop.md) recebem eventos de hardware.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (M0–M7).
