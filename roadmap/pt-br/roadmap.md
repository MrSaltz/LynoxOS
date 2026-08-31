# Lynox — Roadmap

> 🇺🇸 [Read in English](../en/roadmap.md) · [← Voltar pro README](../../docs/pt-br/README.md)

Direção de alto nível do projeto. Pro checklist completo, veja
[`milestones/pt-br/milestones.md`](../../milestones/pt-br/milestones.md);
pros documentos de arquitetura e decisões de engenharia por trás dessas
fases, veja os docs linkados a partir do
[README principal](../../docs/pt-br/README.md).

## Pra onde o projeto está indo

Lynox é um sistema operacional UEFI x86-64 construído do zero: kernel,
drivers, gerenciamento de memória, pilha de rede e um desktop gráfico
próprios — não é uma distro Linux, não é uma modificação do Windows, e não
é construído em torno de um compilador ou linguagem já existente. Segue
uma metodologia rígida, verificada por teste: toda funcionalidade é
projetada, implementada e provada no QEMU antes de ser marcada como
concluída.

## Fases

1. **Foundation** — toolchain, boot, inicialização de CPU, console,
   interrupções, memória física, heap, scheduler/threads. ✅ Concluído
2. **Concurrency** — primitivas de sincronização, block/wakeup, IPC,
   concorrência avançada (condvars, semáforos, rwlocks). ✅ Concluído
3. **Hardware / Drivers** — arquitetura de drivers, enumeração PCI,
   storage, VFS com filesystem persistente, gerenciamento de dispositivos,
   pilha TCP/IP completa. ✅ Concluído
4. **Memory / Processes / Userspace** — memória virtual, processos,
   syscalls com capabilities, userspace, ciclo de vida de processo,
   segurança/capabilities. ✅ Concluído
5. **Runtime / Executables** — runtime de kernel e de usuário, ELF loader
   compatível com toolchains normais (LLVM/Clang, GCC, Rust). ✅ Concluído
6. **Graphics / Desktop** — engine gráfica 2D, sistema de janelas,
   compositor, desktop funcional com aplicações reais, entrada de mouse,
   decoração de janela. ✅ Concluído
7. **Ecosystem** *(fase atual)* — interação de UI profunda, sistema de
   janelas avançado, SDK, modelo de aplicação, ferramentas de
   desenvolvedor. 🚧 Em andamento — ver abaixo.
8. **SMP / Advanced Hardware** — suporte multi-core, controladores de
   interrupção avançados, recursos PCIe. ⬜ Planejado
9. **Security / Reliability** — hardening, sandboxing, tratamento de
   falhas, engenharia de release. ⬜ Planejado
10. **Platform** — áudio, compatibilidade com hardware físico, toolchain
    de produção, plataforma de aplicações, serviços de rede, aceleração
    gráfica, expansão de storage, instalação/recovery, compatibilidade
    POSIX, serviços de sistema. ⬜ Planejado
11. **Final Release** — Lynox 1.0. ⬜ Planejado

## Foco atual: fase Ecosystem

- **UI Interaction** — foco de teclado, duplo-clique, pointer capture,
  entrada unificada. ✅ Concluído
- **Advanced Window System** — barra de título real, redimensionar,
  arrastar, minimizar/maximizar, janelas modais, popups/menus, aplicações
  com múltiplas janelas, ciclo de fechamento interceptável, encaixe nas
  bordas, restrições de tamanho, handle de resize, decorações reativas ao
  foco, renderização consistente de always-on-top/modal, e uma sessão de
  desktop viva, realmente interativa e persistente, movida por
  interrupções reais de mouse/teclado. ✅ Concluído
- **Image/Asset System** — um tipo de imagem carregada e imutável,
  distinto da superfície mutável de uma janela. 🚧 Em andamento
- **Rendering Pipeline, UI Composition, SDK, Application Model, Package
  Manager, Developer Tools, Compatibility Layer** — planejados em
  seguida, nessa ordem. ⬜

## O que este projeto NÃO é

- Não é uma distribuição Linux personalizada nem uma modificação de um
  kernel já existente.
- Não está construindo uma linguagem de programação ou compilador
  próprios — Lynox usa toolchains existentes (LLVM/Clang, GCC, Rust).
- Não é uma game engine, e nunca vai hospedar uma como parte do roadmap
  do OS.

## Princípios norteadores

- Acesso a hardware sempre passa por uma camada de abstração real (driver
  → abstração genérica → kernel), nunca fixado numa plataforma de
  virtualização específica.
- Nenhuma funcionalidade é adicionada "porque seria legal" — todo
  milestone novo precisa de um consumidor concreto ou necessidade
  arquitetural.
- Milestones concluídos e o histórico deles nunca são reabertos ou
  reescritos sem uma necessidade técnica comprovada.
