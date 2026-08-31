# Filesystem

> 🇺🇸 [Read in English](../../en/architecture/filesystem.md)

Cobre a metade de storage da fase Hardware/Drivers/Networking (M12–M17):
transformar um controlador de storage cru num filesystem de verdade que
todo outro subsistema consegue ler e escrever.

## AHCI: um driver de storage de verdade

O driver de storage fala com um controlador AHCI real — transferências
DMA, command lists, FIS (Frame Information Structures) e PRDT (Physical
Region Descriptor Tables) — em vez de uma interface de bloco
simplificada/emulada. Descobrir o controlador em primeiro lugar passa por
enumeração PCI (ver [userspace.md](userspace.md#arquitetura-de-driver)),
não um endereço de hardware fixo e assumido.

## O Virtual Filesystem (VFS)

Em cima do driver de bloco cru fica uma camada de Virtual Filesystem com
sua própria implementação de filesystem persistente e um cache LRU
write-through. Toda camada superior — os descritores de arquivo abertos de
um processo, o File Manager do desktop (ver [desktop.md](desktop.md)) —
fala com uma interface de filesystem única e consistente independente do
que realmente está por trás de um determinado caminho, do mesmo jeito que
o runtime de userspace dá aos programas uma superfície de syscall estável
independente de qual driver acaba tratando um determinado pedido (ver
[userspace.md](userspace.md#o-runtime-de-userspace)).

## Onde isso aparece depois

- O pipeline de imagem/asset (ver
  [graphics.md](graphics.md#o-sistema-de-imagemasset-m35-em-andamento))
  carrega seus bytes crus por essa mesma camada de VFS, não um caminho de
  I/O ad-hoc separado — uma imagem carregada e um arquivo de texto aberto
  pelo File Manager passam pelo mesmo mecanismo de leitura exato.
- Todo acesso a arquivo checado por capability (ver
  [capabilities.md](capabilities.md)) é aplicado nessa camada.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (intervalo M12–M17).
