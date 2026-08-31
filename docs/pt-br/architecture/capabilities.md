# Capabilities

> 🇺🇸 [Read in English](../../en/architecture/capabilities.md)

Cobre a metade de modelo de segurança da fase Memory/Processes/Userspace
(M18–M23): como o Lynox decide se um processo tem permissão de fazer algo.

## Por que não uid/gid estilo Unix

O modelo tradicional Unix checa QUEM VOCÊ É (um id de usuário/grupo)
contra QUEM É DONO DO RECURSO — uma identidade global e ambiente que toda
syscall carrega implicitamente. Esse modelo tem uma fraqueza conhecida:
qualquer código rodando como um determinado usuário herda TODO o acesso
daquele usuário, precisando ou não da maior parte dele. Um bug ou uma
dependência maliciosa numa parte do código de um processo pode alcançar
qualquer coisa que o usuário do processo tenha permissão de tocar.

## Syscalls baseadas em capabilities

O Lynox, em vez disso, exige que um processo detenha uma capability
explícita e infalsificável pra um recurso específico antes que uma
syscall tocando aquele recurso tenha permissão de ter sucesso — não existe
nenhuma saída de emergência ambiente do tipo "sou root, então posso tocar
qualquer coisa". Uma capability é concedida deliberadamente (ex.: quando
um arquivo é aberto, ou quando um recurso é explicitamente entregue a um
processo filho), e um processo que nunca recebeu uma capability pra algo
não consegue alcançar aquilo, independente de qual contexto de usuário
ele nominalmente esteja rodando.

Essa é a mesma família de ideia referenciada na decisão fundacional de
modelo de kernel do projeto (ver
[overview.md](overview.md#modelo-de-kernel-híbrido-com-disciplina-de-microkernel-nas-fronteiras))
como ponto de referência pra design de IPC e controle de acesso em
microkernels modernos orientados a capabilities — adotado pelo mesmo
motivo: compõe melhor com isolamento forte entre processos do que um
modelo de identidade ambiente, e torna "esse pedaço específico de código
só recebeu o acesso que de fato foi entregue a ele" uma propriedade que o
kernel consegue aplicar, não só uma convenção.

## Onde isso aparece depois

- A criação de processo (ver [processes.md](processes.md)) é onde
  capabilities são concedidas a um processo novo pela primeira vez.
- O [filesystem](filesystem.md), a pilha de rede e todo outro subsistema
  dono de recurso checam uma capability antes de honrar um pedido, em vez
  de confiar na identidade de quem chamou.
- Endpoints de [IPC](ipc.md) são eles mesmos recursos checados por
  capability, não um canal ambiente que qualquer processo alcança.

Ver [`milestones/pt-br/milestones.md`](../../../milestones/pt-br/milestones.md) pro lugar dessa fase no checklist de milestones (intervalo M18–M23).
