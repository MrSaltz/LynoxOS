# 0002 — Bootloader: Limine

> 🇺🇸 [Read in English](../../en/decisions/0002-limine.md)

**Status**: Decidido (fundacional, antes de M0). **Área**: [visão geral](../architecture/overview.md#fluxo-de-boot).

## Contexto

Um kernel UEFI x86-64 precisa de algo que saia dos Boot Services do
firmware, monte um ambiente inicial (um memory map, um framebuffer, um
ponteiro ACPI, paginação básica) e salte pro entry point do kernel já em
long mode. Escrever esse estágio do zero, só pra provar que o próprio
kernel funciona, significaria resolver um problema majoritariamente já
resolvido antes de chegar a escrever qualquer código de kernel de
verdade.

## Alternativas consideradas

- **Escrever um bootloader UEFI próprio.** Controle total, mas uma
  quantidade grande de trabalho (tratamento de protocolo UEFI,
  negociação GOP, obtenção de memory map, descoberta de ACPI) gasto
  antes do kernel propriamente dito fazer qualquer coisa — o lugar
  errado pra gastar orçamento de risco cedo nesse projeto.
- **`bootimage`/boot BIOS feito à mão.** Arrastaria preocupações de BIOS
  legado que o projeto não tem interesse em suportar (ver
  [`roadmap`](../../../roadmap/pt-br/roadmap.md) — UEFI-only é escopo
  explícito).
- **Limine.** Um bootloader UEFI+BIOS maduro e mantido ativamente, usado
  por muitos projetos de OS hobby e alguns maiores. Ele entrega ao
  kernel um memory map, um framebuffer GOP e um ponteiro ACPI RSDP num
  protocolo bem definido, e vem com binários pré-compilados que não
  precisam ser compilados como parte do build deste projeto.

## Decisão

**Limine**, usando seus binários UEFI pré-compilados (vendorizados, sem
modificação), com uma ISO UEFI-only (sem entrada de boot BIOS —
consistente com o escopo declarado do projeto).

## Consequências

- O primeiro milestone verificável do kernel (M1 — handoff do
  bootloader) foi alcançado rapidamente, porque as partes difíceis do
  boot UEFI já estavam resolvidas por um projeto externo maduro.
- O projeto depende do protocolo e da compatibilidade binária do Limine
  daqui pra frente; isso é tratado como uma dependência aceitável e
  revisitável — se controle de cadeia de confiança do Secure Boot algum
  dia se tornar um requisito real (ver a fase de segurança/confiabilidade
  em [`roadmap`](../../../roadmap/pt-br/roadmap.md)), essa decisão pode
  precisar ser revisitada, já que exigiria que o projeto controlasse mais
  da própria cadeia de boot do que um binário de bootloader pré-compilado
  permite.
