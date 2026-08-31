# Visão Geral da Arquitetura

> 🇺🇸 [Read in English](../../en/architecture/overview.md)

Esta página é o mapa. Ela explica as escolhas fundacionais que moldam todo
o resto do sistema, e linka pra um documento dedicado de cada área
principal. Nenhuma dessas páginas percorre código-fonte privado linha a
linha — elas explicam a FORMA de cada subsistema e o raciocínio por trás
dela, no nível que um engenheiro de sistemas gostaria de ter antes de
decidir se o design se sustenta.

- [Núcleo do kernel](kernel.md) — boot, inicialização de CPU, interrupções,
  memória física.
- [Userspace](userspace.md) — drivers, pilha TCP/IP, runtime de userspace.
- [Memória](memory.md) — memória virtual, paginação, copy-on-write,
  isolamento de processos.
- [Processos](processes.md) — ciclo de vida de processo, ELF loader.
- [Threads](threads.md) — scheduler preemptivo, primitivas de concorrência.
- [Capabilities](capabilities.md) — o modelo de syscalls baseado em
  capabilities e por que não é uid/gid estilo Unix.
- [IPC](ipc.md) — comunicação entre processos isolados.
- [Filesystem](filesystem.md) — o driver AHCI e o Virtual Filesystem.
- [Gráficos](graphics.md) — a engine gráfica 2D, superfícies, o sistema de
  imagem/asset.
- [Sistema de janelas](window-system.md) — janelas, chrome, o compositor.
- [Toolkit de UI](ui.md) — o toolkit de widgets e o modelo de interação.
- [Desktop](desktop.md) — a sessão de desktop viva, roteamento de entrada,
  as aplicações embutidas.

## Modelo de kernel: híbrido, com disciplina de microkernel nas fronteiras

Existem três opções reais pra quanto vive em espaço de kernel: um kernel
puramente monolítico (Linux), um microkernel puro (seL4, MINIX 3), ou um
híbrido (Windows NT, XNU). Um monolito puro é o jeito mais rápido de ter
algo funcionando, mas retrofit de isolamento depois é doloroso — um bug num
driver pode derrubar o kernel inteiro. Um microkernel puro dá o isolamento
mais forte, mas construir um do zero, sozinho, é um projeto de vários anos
só pra acertar o núcleo, e uma camada de IPC mal projetada pode dominar a
performance.

O Lynox segue o caminho híbrido deliberadamente: um núcleo pequeno cuida de
CPU, memória, scheduler, interrupções e as primitivas de segurança
(capabilities). Drivers e serviços de sistema são arquitetados como se
fossem componentes separados falando por IPC desde o primeiro dia — mesmo
os que hoje rodam em espaço de kernel por simplicidade de bring-up usam a
mesma interface que uma versão em userspace usaria. Esse é o detalhe que
importa: o objetivo não é "monolítico agora, microkernel depois" (essa
migração praticamente nunca acontece na prática, porque nada foi desenhado
pra isso) — é "híbrido agora, com as costuras já cortadas nos lugares
certos".

## Linguagem: Rust

A linguagem do kernel é a decisão mais cara de reverter no projeto inteiro
— ela determina a toolchain, o que é fácil de escrever, e a forma de toda
classe de bug possível. O Lynox escolheu Rust em vez de C (o padrão
histórico pra OS dev) porque a prioridade #1 do projeto é segurança, e
memory safety por construção elimina a classe de bug mais comum em kernels
C (buffer overflow, use-after-free, data race) sem depender de disciplina
humana perfeita sustentada por anos. O ecossistema Rust bare-metal já
estava maduro o suficiente na época dessa decisão — crates reais pra
acesso a CPU/registradores, um ecossistema de bootloader mantido, e um
precedente real em uso de produção (existe um SO completo, Redox) — que a
curva de aprendizado extra (lutar com o borrow checker em código
`unsafe`/`no_std` enquanto também se aprende desenvolvimento de OS) foi
julgada válida.

Assembly (NASM) é usado só pro mínimo inevitável — o stub de entrada do
boot, troca de contexto, stubs de interrupção. C é permitido só como
exceção pontual, se algum dia for necessário vendorizar uma biblioteca de
baixo nível sem equivalente em Rust; isso não foi necessário até agora.

## Toolchain

- **Host de build**: WSL2 (Ubuntu) — ferramentas de OS dev (`nasm`,
  `qemu`, `ovmf`, `gdb`) são cidadãs de primeira classe no Linux; evita
  gambiarras de path/permissão do Windows sem nenhum benefício.
- **Toolchain Rust**: `rustup` nightly (necessário pras features
  `no_std`/`build-std`), target customizado `x86_64-unknown-none` — um
  kernel bare-metal não pode usar o `std` pré-compilado do host.
- **Linker**: `lld` (vem com Rust/LLVM) mais um linker script próprio, pra
  controle total do layout de memória do kernel.
- **Bootloader**: [Limine](https://github.com/limine-bootloader/limine) —
  um bootloader UEFI/BIOS maduro e mantido ativamente. Ele entrega ao
  kernel um memory map, um framebuffer GOP e um ponteiro ACPI RSDP, o que
  evita reinventar o estágio de boot só pra provar que o kernel roda.
- **Firmware de VM**: OVMF (TianoCore) — o firmware UEFI open-source
  padrão pro QEMU.
- **Emulador**: QEMU, pra iteração rápida, snapshot e anexar o GDB sem
  precisar de hardware físico a cada ciclo.

## Fluxo de boot

```
Liga o PC
  → Firmware UEFI (OVMF na VM / UEFI real em hardware físico)
  → Limine (carrega kernel.elf, monta o memory map, framebuffer GOP, ACPI RSDP)
  → Limine sai dos UEFI Boot Services e salta pro entry point do kernel
     (já em long mode, paginação básica montada pelo protocolo Limine)
  → Kernel: monta seu próprio GDT/IDT, inicializa o console serial
  → Kernel inicializa subsistemas em ordem de dependência (memória →
     scheduler → drivers → filesystem → rede → gráficos → desktop)
  → A sessão de desktop viva começa e nunca retorna
```

A saída serial (COM1) foi a primeira coisa que o kernel conseguiu fazer,
antes de existir um console de framebuffer — é o jeito mais simples
possível de provar que o kernel está vivo, sem depender de fonte bitmap
nem driver de vídeo. Essa ordem (provar que liga, depois provar que
desenha texto) é representativa de como o projeto inteiro é sequenciado: a
afirmação mais arriscada e fundamental é provada primeiro, sempre.

## Formato de executável

O kernel é um binário ELF64 comum — a saída nativa do target Rust
`x86_64-unknown-none`, bem suportado pelo Limine e por toda ferramenta de
debug padrão (`gdb`, `objdump`, `readelf`). Não havia motivo pra inventar
um formato próprio nesse estágio.

## Como isso mapeia pro roadmap

Cada uma das páginas linkadas acima corresponde a um ou mais intervalos de
milestone em
[`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md)
— ver esse arquivo pra lista completa e granular. Ver
[`docs/pt-br/design/coding-principles.md`](../design/coding-principles.md)
pra metodologia que todo milestone segue.
