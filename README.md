# PVM: Programmable Protocol Emulator for Tiny Tapeout IHP

**An open-source reprogrammable serial-protocol ASIC for the Jane Street design competition, targeting Tiny Tapeout on the IHP SG13CMOS5L 130 nm open PDK.**

A host loads a program image over SPI. A 32-word microcode engine then drives or samples the chip's pins, a small
configurable logic fabric handles selected logic and timing tasks, and a passive monitor records rule verdicts and
measured intervals. The same RTL runs a 115.2 kbaud UART transmitter, an SPI Mode 0 controller and an I2C write with ACK
and clock stretching. A held-out DMX512 TX demo (two packets from a 29-word image) was added after the RTL freeze to test
that the architecture generalizes.

> **Status (Nov 2026).** RTL is frozen at the `rtl-freeze` tag and passes the registered simulation, mutation and
> gate-level replay checks listed below. `HARDWARE_VALIDATED=NO`, `FORMAL_PROOF=NO`, `SILICON=NO`. The design has not been
> fabricated. The competition submission is planned for Dec 2026.

## Design

| Block | Role |
|---|---|
| Host interface | SPI image loader, frozen through numbered interface revisions |
| Engine | 32-word program memory, PIO-style instruction set with FIFO, shift, WAIT, jump and timeout paths |
| Fabric | Four configurable LUT4s in a feed-forward network, a counter, a holding register for Manchester output, and an open-drain arbitration guard that can release SDA and SCL after arbitration loss |
| Monitor | Passive; evaluates programmable timing rules and records verdicts and measured intervals |

Four build variants share the same 32-word architecture. Full description: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md),
[docs/ISA.md](docs/ISA.md).

## Physical results

Open-source flow: LibreLane (Yosys, OpenROAD) with the Tiny Tapeout GitHub Actions flow on the IHP open PDK.

| Metric | Result |
|---|---|
| Tile footprint | 8x4 tiles |
| Clock | 50 MHz |
| Utilization | 30.97% |
| Setup slack, slow corner | +3.180 ns |
| Worst hold slack | +0.108 ns |
| Tiny Tapeout precheck | Passed on the release build |

Details and the routed demo replay package: [results/v1.0_routed.md](results/v1.0_routed.md),
[results/fit_study.md](results/fit_study.md).

## Verification

All results come from recorded, reproducible jobs. Claim levels are labeled per stage in [STATUS.md](STATUS.md).

- **Stage checks.** 71 jobs accepted under Icarus Verilog on Windows, each with mutation testing of its targeted check.
- **Second simulator.** Verilator lint for the four variants and timing-enabled replay of 26 distinct positive benches at
  two seeds each.
- **Instruction-set model.** A spec-derived ISS matched 13,848 baseline demo cycles and 7,680 additional directed cycles in
  lockstep with the RTL.
- **I2C arbitration.** A 360-case arbitration study (four jobs, including two mutants) after a fix to arbitration-loss
  handling.
- **Timing repair.** The 50 MHz timing repair was accepted only after four cycle-by-cycle trace matches against the
  previous RTL and a killed one-cycle output-delay mutant.
- **Gate-level replay.** The three protocol demos replay functionally on the routed netlist with pinned IHP cell models.
- **Not done.** Hardware validation, formal proof and silicon. The DemoBoard SPI adapter test is pending.

See [docs/VERIFICATION.md](docs/VERIFICATION.md).

## Quick start

```sh
python3 -m tools.pvm_image tools/examples/uart_tx.pvm /tmp/pvm-uart.json \
  --config tools/examples/uart_tx.json --tx 0x35
python3 tb_rtl/pvm_run.py run --select all     # needs iverilog and vvp
```

Toolchain guide: [docs/TOOLCHAIN.md](docs/TOOLCHAIN.md).

## Contributions

PVM is a two-person project. The git history in this repository is the record of authorship.

- **Arnav Bhandari:** repository owner; most of the RTL, tooling, assembler, ISS and CI work.
- **Kalash Bheda:** RTL review and approval (including interface decision D-012 and the frozen RTL), the independent
  verification package and blocker records, and project status tracking.

## Tools and AI use

The design, tests and evidence were built with AI coding assistance. All generated work was checked through the recorded
jobs above. See [docs/AI_USE.md](docs/AI_USE.md) for the full disclosure.

## License

Apache License 2.0. See [LICENSE](LICENSE).
