# Decisões

> 🇺🇸 [Read in English](../../en/decisions/README.md)

Registros de Decisão de Arquitetura do Lynox — decisões técnicas
específicas, pontuais no tempo, escritas como: contexto, as alternativas
reais consideradas, a decisão e suas consequências. São um subconjunto
curado das decisões tomadas ao longo do projeto (ver
[`docs/pt-br/architecture/`](../architecture/overview.md) pra como cada
subsistema funciona) — escolhidas porque ilustram um trade-off ou um
raciocínio útil por si só, fora do milestone específico de onde veio.

- [0001 — Linguagem do kernel: Rust](0001-rust.md)
- [0002 — Bootloader: Limine](0002-limine.md)
- [0003 — Modelo de kernel: híbrido, não monolito ou microkernel puro](0003-kernel-model.md)
- [0004 — Imutabilidade de Image: convenção em vez de divisão em nível de tipo](0004-image-immutability.md)
- [0005 — Damage tracking do compositor: blit parcial, não expansão de região](0005-damage-tracking.md)

Ver [`docs/pt-br/design/coding-principles.md`](../design/coding-principles.md)
pras regras gerais sob as quais essas decisões foram tomadas.
