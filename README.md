# KicadDesignBlocks

Reusable KiCad circuit blocks: a schematic and its PCB layout, one self-contained
KiCad project per folder. Each project can be opened or shared on its own, for
example for an external review.

| Block | What it is | Status |
|---|---|---|
| [`TPS563201/`](TPS563201/) | Buck 5 V → 3.3 V / 1 A, TI TPS563201, 4 layers. Generated from the [MultiSensors](https://github.com/ludodefgh/MultiSensors) board | layout done, not fabricated |
| [`TPS5430Buck/`](TPS5430Buck/) | Power input + buck based on the TI TPS5430: screw-terminal input, polyfuses (schematic note: "5/12/24V Power In") | earlier stand-alone project, history imported from the former `TPS5430Buck` repo |

## Conventions

- One folder per block, named after the main part, holding its own `.kicad_pro`.
- Symbols that are not in the standard KiCad libraries go in a project-local library
  (`sym/`, declared in the folder's `sym-lib-table`), so the project opens anywhere.
- `docs/` holds what a reviewer needs without KiCad: schematic PDF, layout image, and
  the sources the values came from.
- If a block is generated from another project, its README says so and names the
  script. Change it at the source and re-export.
