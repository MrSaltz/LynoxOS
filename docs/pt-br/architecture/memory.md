# Memória

> 🇺🇸 [Read in English](../../en/architecture/memory.md)

Cobre a metade de memória da fase Memory/Processes/Userspace (M18–M23):
transformar o frame allocator físico (ver
[kernel.md](kernel.md#sequência-de-bring-up)) em memória virtual completa
com isolamento real entre processos.

## Por que memória virtual, e por que agora

Tudo antes dessa fase roda num único espaço de endereço físico plano —
correto pra um kernel single-threaded provando drivers e primitivas de
concorrência, mas incompatível com a prioridade declarada do projeto de
isolamento forte (ver
[overview.md](overview.md#modelo-de-kernel-híbrido-com-disciplina-de-microkernel-nas-fronteiras)).
Isolamento real de processo exige que cada processo acredite que é dono do
espaço de endereço inteiro, com o hardware de paginação da CPU garantindo
essa crença.

## Paginação

O Lynox implementa a paginação de 4 níveis do x86-64 (PML4 → PDPT → PD →
PT) diretamente, em vez de depender de uma crate de abstração de
paginação, pra que o kernel tenha controle total sobre o layout das
tabelas de página, bits de permissão, e como mapeamentos de espaço de
kernel e de usuário coexistem nas mesmas tabelas (mapeamentos de kernel
estão presentes no espaço de endereço de todo processo no nível de
privilégio certo, então uma syscall ou interrupção não precisa trocar de
espaço de endereço pra alcançar código de kernel).

## Copy-on-write

Em vez de copiar profundamente o espaço de endereço inteiro de um processo
a cada operação estilo `fork`, páginas são compartilhadas em modo
somente-leitura entre pai e filho até que um deles escreva — nesse
momento, um page fault dispara uma cópia de verdade, e só daquela página.
Essa é a técnica padrão pra tornar a criação de processo barata sem abrir
mão de isolamento: o próprio mecanismo de page fault da CPU é o que aplica
"pode compartilhar até divergir".

## Isolamento de processo via CR3

Cada processo ganha sua própria tabela de página de nível mais alto
(PML4), e uma troca de contexto entre processos com espaços de endereço
diferentes recarrega o registrador CR3 pra apontar pras tabelas de página
do novo processo. Isso é o que torna o isolamento de processo real, em vez
de uma convenção que dois processos concordam em respeitar: um processo
literalmente não consegue construir um ponteiro que alcance a memória
privada de outro processo, porque o hardware não vai traduzir ele.

## Onde isso aparece depois

- A criação de processo e o ELF loader (ver [processes.md](processes.md))
  dependem de conseguir montar um espaço de endereço novo por processo de
  forma barata.
- O modelo de capabilities (ver [capabilities.md](capabilities.md)) só faz
  sentido especificamente porque o isolamento de espaço de endereço é
  real — uma capability controla acesso a um RECURSO, e o isolamento de
  processo é o que garante que um processo não consegue contornar esse
  controle alcançando a memória de outro processo diretamente.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (intervalo M18–M20).
