# Processos

> 🇺🇸 [Read in English](../../en/architecture/processes.md)

Cobre a metade de processos da fase Memory/Processes/Userspace (M18–M23) e
a metade de carregamento de executável da fase Runtime/Executables
(M24–M26): transformar um espaço de endereço isolado (ver
[memory.md](memory.md)) em algo que consegue de fato rodar um programa.
Pra camada de runtime EM CIMA de um processo rodando (syscalls, drivers,
filesystem, rede), ver [userspace.md](userspace.md).

## Ciclo de vida de processo

Um processo no Lynox possui: seu próprio espaço de endereço (ver
[memory.md#isolamento-de-processo-via-cr3](memory.md#isolamento-de-processo-via-cr3)),
uma ou mais threads (ver [threads.md](threads.md)) agendadas dentro
daquele espaço de endereço, uma tabela de capabilities que ele detém (ver
[capabilities.md](capabilities.md)), e descritores de arquivo abertos pro
filesystem. Criar, rodar e destruir um processo toca os três subsistemas
centrais do kernel de uma vez, por isso essa fase vem depois de memória e
concorrência estarem sólidas.

## O ELF loader

Em vez de aceitar código de máquina escrito à mão como único jeito de
colocar um programa em userspace, o Lynox implementa um ELF loader de
verdade: parseia o header ELF e os program headers, mapeia cada segmento
`PT_LOAD` no espaço de endereço do novo processo com as permissões certas
(somente-leitura pra `.rodata`, executável pra `.text`, gravável pra
`.data`/`.bss`), e realiza as relocations que o binário exige. Validação
de ABI rejeita qualquer coisa que não bata com o que o kernel espera
(arquitetura errada, classe errada, tipos de segmento não suportados) em
vez de tentar rodar algo malformado na esperança de dar certo.

Isso importa além de correção: significa que programas de userspace pro
Lynox são COMPILADOS NORMALMENTE — com uma toolchain padrão de Rust ou C
mirando a ABI certa — em vez de exigir um formato especial montado à mão.
Qualquer coisa que produza um binário ELF64 conforme é um programa de
userspace válido pro Lynox, o que é o que permite que o runtime de
userspace (ver [userspace.md](userspace.md)) exista como código compilado
normal em vez de uma pilha de bytes hexadecimais embutidos no kernel.

## Onde isso aparece depois

- As aplicações do desktop (File Manager, Settings, Terminal, System UI —
  ver [desktop.md](desktop.md)) são elas mesmas processos rodando por
  esse mesmo ciclo de vida e loader, não código de kernel especial.
- Toda syscall que um processo faz é checada contra o modelo de
  capabilities (ver [capabilities.md](capabilities.md)) antes de poder
  tocar um recurso.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (intervalo M18–M26).
