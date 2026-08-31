# Lynox

> 🇺🇸 [Read in English](../../README.md)

**Sistema operacional para PC x86-64 (UEFI) construído do zero em Rust** —
kernel, drivers, memória virtual, processos/threads, syscalls com modelo de
capabilities, filesystem persistente próprio, pilha de rede TCP/IP, ELF
loader, engine gráfica 2D, sistema de janelas, compositor e um desktop com
aplicações próprias.

O kernel e a arquitetura do sistema operacional Lynox são desenvolvidos do
zero em Rust. O projeto não deriva do Linux, do Windows nem de nenhum outro
sistema operacional já existente. Dependências de terceiros e padrões
abertos são usados onde estiver explicitamente documentado (ver
[Componentes de terceiros](#componentes-de-terceiros)).

> **Este repositório é o registro público de engenharia do Lynox — não o
> código-fonte do kernel.** O kernel, os drivers e o código de userspace são
> desenvolvidos num repositório privado enquanto o projeto ainda está em
> desenvolvimento ativo e inicial. O que está aqui é real: o roadmap, a
> metodologia, e um conjunto de documentos de arquitetura que explicam
> COMO e POR QUE cada subsistema funciona daquele jeito, sem percorrer a
> implementação privada linha a linha. Ver
> [Público vs. privado](#público-vs-privado) abaixo.

## Status

**M0-M34 completos, M35 (Image / Asset System) em andamento (M35.1-M35.6
prontos)** — ~265 commits. Toolchain/boot, concorrência, drivers/PCI/
storage/VFS, rede TCP/IP, memória virtual, processos, syscalls/capabilities,
runtime de kernel e de usuário, ELF loader e um desktop gráfico completo já
estão prontos. **M34 (Advanced Window System)** está totalmente fechado —
barra de título real, redimensionar, arrastar, minimizar/maximizar, janelas
modais, popups/menus, aplicações com múltiplas janelas, ciclo de fechamento
interceptável, encaixe nas bordas, restrições de tamanho, handle de resize,
decoração reativa ao foco, ordem visual/de clique consistente pra
always-on-top e modal, uma taskbar clicável, e uma **sessão de desktop viva e
persistente** movida por interrupções reais de mouse/teclado, com contador
de FPS em tempo real e um compositor com damage tracking. **M35 (Image /
Asset System)** está em andamento: um tipo de imagem carregada e imutável,
conversão de formato de pixel, leitura de bytes crus do filesystem,
decodificação pra uma imagem usável e escala nearest-neighbor já estão
prontos.

- Roadmap e direção: [`roadmap/pt-br/roadmap.md`](../../roadmap/pt-br/roadmap.md)
- Checklist de milestones: [`milestones/pt-br/milestones.md`](../../milestones/pt-br/milestones.md)
- Documentos de arquitetura (como e por quê): [`architecture/overview.md`](architecture/overview.md)
- Princípios de design e metodologia: [`design/coding-principles.md`](design/coding-principles.md)
- Decisões de engenharia notáveis: [`decisions/`](decisions/)

## Destaques técnicos

- **Kernel próprio em Rust** (`no_std`, bare-metal), sem depender de
  libc/OS hospedeiro.
- **Memória virtual completa**: paginação própria, copy-on-write,
  isolamento real entre processos via troca de CR3.
- **Scheduler preemptivo** com threads, bloqueio/wakeup,
  mutex/condvar/semáforo/rwlock.
- **Syscalls com modelo de capabilities** (não é uid/gid estilo Unix) —
  cada processo só acessa um recurso pra que tenha uma capability
  explícita.
- **VFS com filesystem persistente próprio** sobre um driver AHCI real
  (DMA, command list/FIS/PRDT), com cache LRU write-through.
- **Pilha de rede TCP/IP própria** sobre um driver e1000 real:
  Ethernet/ARP/IPv4/UDP/TCP/Sockets/DHCP/DNS/IPv6, verificada em loopback
  e via hardware emulado.
- **ELF loader real**: parsing, mapeamento de segmentos, relocations,
  validação de ABI — programas de userspace compilados normalmente, não
  bytes de assembly escritos à mão.
- **Engine gráfica 2D própria** sobre framebuffer UEFI (GOP): superfícies,
  primitivas, fontes, compositor com double buffering e damage tracking.
- **Desktop funcional**: 5 aplicações (File Manager, Settings, System UI,
  Terminal, Taskbar) sobre uma camada de UI própria, com driver de mouse
  PS/2 real roteando cliques pra janela certa e uma taskbar clicável.
- **Sistema de janelas completo**: redimensionar, arrastar, minimizar/
  maximizar, modal e always-on-top, com encaixe nas bordas, restrições de
  tamanho, handle de resize e decoração reativa ao foco.
- **Sessão de desktop viva e persistente**: uma thread do scheduler que
  nunca retorna, consumindo interrupções reais de mouse/teclado e
  renderizando o resultado continuamente com um compositor com damage
  tracking e um contador de FPS em tempo real — verificada
  interativamente em dois hypervisors independentes (QEMU e VirtualBox).

## Metodologia

Todo submilestone segue a mesma disciplina: objetivo → alternativas reais
consideradas → decisão justificada → arquitetura → implementação →
verificação → documentação. Verificação significa realmente RODAR o
resultado (um boot headless no QEMU, checado por um marcador de sucesso/
falha na saída serial, repetido várias vezes pra confirmar estabilidade),
nunca só uma compilação bem-sucedida. Bugs encontrados no caminho ficam
registrados honestamente nos documentos de arquitetura — inclusive os que
eram regressões auto-infligidas — em vez de corrigidos silenciosamente.
Ver
[`design/coding-principles.md`](design/coding-principles.md) pro conjunto
completo de regras norteadoras (sem abstração prematura, sem complexidade
especulativa, exige um consumidor real antes de qualquer feature nova).

## Público vs. privado

O Lynox é construído em aberto no sentido que mais importa pra uma
audiência de engenharia: o roadmap, o raciocínio por trás de cada decisão
de design, o histórico completo de commits e o que cada um mudou, e (à
medida que demos ficarem disponíveis) o sistema rodando de verdade estão
todos aqui. O que fica privado por enquanto é o código-fonte literal do
kernel/drivers/userspace, porque o projeto ainda está em desenvolvimento
ativo e inicial, e a intenção é manter o registro de engenharia aberto sem
abrir revisão de código sobre um alvo em movimento antes de estar pronto.

Isso não é uma promessa de que o código NUNCA será publicado, e não é uma
tentativa de esconder que partes privadas existem — os documentos de
arquitetura descrevem todos os subsistemas, inclusive os que não têm
código-fonte aqui. É uma escolha deliberada, do estágio atual, sobre o que
está pronto pra ser revisado como código versus o que está pronto pra ser
lido como uma história de engenharia.

## Componentes de terceiros

- **Limine** — bootloader UEFI/BIOS (binário, vendorizado, sem modificação).
- **OVMF (TianoCore)** — firmware UEFI usado em desenvolvimento/teste no QEMU.
- Um número pequeno de crates Rust pra primitivas de baixo nível (ex.:
  bindings de CPU/registradores, uma fonte bitmap pro console). O kernel,
  drivers, filesystem, pilha de rede, engine gráfica, sistema de janelas e
  desktop são código próprio.

## Documentação

- **Arquitetura** — como e por que cada subsistema funciona daquele jeito,
  ver [`architecture/overview.md`](architecture/overview.md) pro mapa.
  Cobre: [kernel](architecture/kernel.md), [userspace](architecture/userspace.md),
  [memória](architecture/memory.md), [processos](architecture/processes.md),
  [threads](architecture/threads.md), [capabilities](architecture/capabilities.md),
  [IPC](architecture/ipc.md), [filesystem](architecture/filesystem.md),
  [gráficos](architecture/graphics.md), [sistema de janelas](architecture/window-system.md),
  [toolkit de UI](architecture/ui.md) e [desktop](architecture/desktop.md).
- **Design** — [`design/coding-principles.md`](design/coding-principles.md)
  (metodologia geral), [`design/ui-guidelines.md`](design/ui-guidelines.md)
  e [`design/graphics-guidelines.md`](design/graphics-guidelines.md)
  (princípios específicos de UI e gráficos).
- **Decisões** — [`decisions/`](decisions/), registros de decisões
  arquiteturais específicas com alternativas consideradas e consequências.

## Roadmap

- [`roadmap/pt-br/roadmap.md`](../../roadmap/pt-br/roadmap.md) — direção, fases e o que está fora de escopo.
- [`milestones/pt-br/milestones.md`](../../milestones/pt-br/milestones.md) — uma linha por milestone principal (M0-M65), o que está feito e o que falta.

Estrutura de fases (ver [`milestones/pt-br/milestones.md`](../../milestones/pt-br/milestones.md) para a lista completa):
Foundation (M0-M7) → Concurrency (M8-M11) → Hardware/Drivers/Networking
(M12-M17) → Memory/Processes/Userspace (M18-M23) → Runtime/Executables
(M24-M26) → Graphics/Desktop (M27-M32) → **Ecosystem (M33-M42, em
andamento — M34 fechado, M35 em andamento)** → SMP/Advanced Hardware
(M43-M47) → Security/Reliability (M48-M54) → Platform (M55-M64) → Final
Release (M65).

## Screenshots / demos

[`screenshots/`](../../screenshots/) e [`demos/`](../../demos/) vão guardar
capturas à medida que os milestones produzirem algo visualmente
interessante pra mostrar (a sessão de desktop viva, o sistema de janelas,
aplicações reais) — organizados por boot, desktop, aplicações e
desenvolvimento. Vazios por enquanto.

## Acompanhando o projeto / feedback

Issues estão abertas pra perguntas, feedback e discussão sobre a
arquitetura ou o roadmap. Contribuições de código pro kernel não estão
sendo aceitas enquanto o código-fonte continuar privado — ver
[`.github/CONTRIBUTING.md`](../../.github/CONTRIBUTING.md) pra postura
atual e que tipo de Issue é bem-vinda, e
[`.github/SECURITY.md`](../../.github/SECURITY.md) pra como reportar uma
preocupação de segurança.

## Licença

Ver [`LICENSE`](../../LICENSE). Ela cobre o conteúdo deste repositório
(documentação, documentos de arquitetura, roadmap) — não concede nenhum
direito sobre o código-fonte privado do kernel/SO Lynox, que não está
incluído aqui.

"Lynox" é usado aqui só como o nome deste projeto. Não é reivindicado
como marca registrada, e nenhuma afiliação com, ou endosso de, qualquer
outro produto, empresa ou marca de nome parecido é pretendida ou
implicada.
