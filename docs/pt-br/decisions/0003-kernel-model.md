# 0003 — Modelo de kernel: híbrido, não monolito ou microkernel puro

> 🇺🇸 [Read in English](../../en/decisions/0003-kernel-model.md)

**Status**: Decidido (fundacional, antes de M0). **Área**: [núcleo do kernel](../architecture/kernel.md).

## Contexto

A prioridade declarada do projeto é segurança — especificamente,
isolamento forte entre drivers/serviços e o resto do sistema, com a
capacidade de mover um componente de espaço de kernel pra espaço de
usuário depois sem uma reescrita completa. A decisão de modelo de kernel
precisava ser tomada antes de qualquer código ser escrito, já que ela
molda a toolchain, o design de IPC e todo subsistema construído em cima
dela.

## Alternativas consideradas

| Modelo | Exemplos de referência | Vantagem | Desvantagem |
|---|---|---|---|
| Monolítico puro | Linux | Rápido de colocar pra funcionar, sem overhead de IPC | Isolamento fraco por padrão; retrofit de segurança depois é doloroso; um bug de driver pode derrubar o kernel inteiro |
| Microkernel puro | seL4, MINIX 3 | Isolamento máximo, superfície de ataque mínima no kernel, formalmente verificável no caso do seL4 | Custo de IPC alto se mal projetado; curva de construção muito mais íngreme; construir um do zero sozinho é um projeto de vários anos só pro núcleo |
| Híbrido | Windows NT, XNU (macOS/iOS) | Um kernel pequeno mais serviços em espaço de usuário onde compensa, mantendo a liberdade de manter caminhos quentes no kernel | Menos "puro" em teoria; exige disciplina pra evitar deslizar de volta pra um monolito por conveniência |
| Microkernel moderno pragmático | Zircon (Fuchsia) | IPC desenhado como primitiva de primeira classe (handles, channels) desde o dia um, baseado em capabilities desde o início | Ainda um projeto grande; exige tratar IPC como central, não um acréscimo |

## Decisão

**Híbrido, com disciplina de microkernel aplicada nas fronteiras.** Um
núcleo pequeno cuida de bring-up de CPU, memória, o scheduler,
interrupções, IPC e as primitivas de segurança baseadas em capabilities.
Drivers e serviços de sistema são arquitetados como se fossem
componentes separados, falantes de IPC, independente de onde eles
atualmente executam — alguns drivers simples hoje rodam em espaço de
kernel por simplicidade de bring-up, mas atrás da mesma interface que
uma versão em espaço de usuário do mesmo driver usaria.

## Consequências

- Isso evita o "imposto de microkernel" de desenhar IPC três vezes (uma
  vez ingenuamente, depois duas vezes mais pra corrigir), porque IPC é
  desenhado como uma primitiva real desde o ponto em que drivers
  primeiro precisam dele (ver [ipc.md](../architecture/ipc.md)), não
  encaixado depois do fato.
- Também evita a armadilha comum onde o "podemos isolar isso depois" de
  um monolito nunca de fato acontece, porque nada foi desenhado com
  aquela fronteira em mente — a disciplina de interface é aplicada
  AGORA, enquanto ainda é barato aplicar.
- O trade-off explícito aceito: algum código em espaço de kernel hoje não
  está literalmente isolado do jeito que um design totalmente
  microkernel aplicaria. Isso é tratado como uma simplificação de
  bring-up deliberada e temporária, não a arquitetura final — as costuras
  necessárias pra mover ele pra espaço de usuário depois já estão no
  lugar.
