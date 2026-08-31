# Userspace

> 🇺🇸 [Read in English](../../en/architecture/userspace.md)

Cobre a metade de drivers/rede da fase Hardware/Drivers/Networking
(M12–M17) e a metade de runtime da fase Runtime/Executables (M24–M26): a
camada entre um processo rodando (ver [processes.md](processes.md)) e o
hardware/serviços reais dos quais ele depende. Storage e o filesystem têm
sua própria página — ver [filesystem.md](filesystem.md).

## Arquitetura de driver

Drivers no Lynox são escritos seguindo a mesma disciplina descrita no
modelo de kernel (ver
[0003 — Modelo de kernel](../decisions/0003-kernel-model.md)): mesmo os
que hoje rodam em espaço de kernel por simplicidade de bring-up são
construídos atrás de uma interface que uma versão em userspace do mesmo
driver poderia usar sem mudança nenhuma. Enumeração PCI é o que descobre
hardware real em primeiro lugar (o controlador de storage AHCI e o
controlador de rede e1000, em particular), em vez de assumir endereços de
hardware fixos.

## Rede: uma pilha TCP/IP de verdade

A pilha de rede é construída em camadas sobre o driver e1000: Ethernet,
ARP, IPv4, UDP, TCP, uma API de sockets, DHCP, DNS e IPv6 — cada uma
verificada tanto via loopback quanto via hardware emulado. Assim como com
storage (ver [filesystem.md](filesystem.md)), camadas superiores falam com
uma interface de sockets estável sem precisar saber as especificidades da
placa de rede por baixo.

## O runtime de userspace

"Runtime" aqui significa a camada entre syscalls cruas e um programa que
parece código normal: cola de startup/entry, uma camada de wrapper de
syscall, e superfície suficiente no formato de standard library pra que
programas de userspace não precisem montar números de syscall e
convenções de chamada à mão. Existem tanto um runtime do lado do kernel
(pra código de teste/utilitário em espaço de kernel) quanto um runtime de
userspace, compartilhando a mesma ABI de syscall por baixo — e toda
syscall que essa camada faz é checada contra o modelo de capabilities (ver
[capabilities.md](capabilities.md)) antes de poder tocar um recurso.

## Onde isso aparece depois

- O [ELF loader](processes.md#o-elf-loader) é o que transforma um binário
  compilado num processo rodando usando essa mesma camada de runtime.
- As aplicações do desktop (ver [desktop.md](desktop.md)) são processos de
  userspace comuns construídos sobre esse runtime, não código de kernel
  especial.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (intervalo M12–M17, M24–M26).
