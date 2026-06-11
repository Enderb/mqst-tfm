# QICK and tProcessor patterns

## Two API generations

| | tprocv1 (legacy) | tprocv2 (current) |
|---|---|---|
| Base class | `AveragerProgram`, `RAveragerProgram` | `AsmV2` (from `qick.asm_v2`) |
| Sweep | `RAveragerProgram.update()` increments registers | `QickParam` + `NDAveragerProgram` |
| Used in thesis | Appendix A code listings | `ssro_raw_waveform.ipynb` |

The notebooks use **tprocv2** (`AsmV2`). The thesis appendix code listings use **tprocv1** patterns for pedagogical clarity; real experiments should prefer tprocv2.

## tprocv2 program skeleton

```python
from qick.asm_v2 import AsmV2

class MyProgram(AsmV2):
    def __init__(self, soccfg, cfg):
        self.cfg = cfg
        super().__init__(soccfg, reps=cfg["shots"])

    def initialize(self):
        cfg = self.cfg
        self.declare_gen(ch=cfg["res_ch"], nqz=1)
        self.declare_readout(
            ch=cfg["adc_ch"],
            length=cfg["raw_length"],
            freq=cfg["res_freq"],
            gen_ch=cfg["res_ch"],
        )
        self.set_pulse_registers(
            ch=cfg["res_ch"], style="const",
            freq=cfg["res_freq"], phase=0,
            gain=cfg["res_gain"],
            length=self.us2cycles(cfg["res_length"], gen_ch=cfg["res_ch"]),
        )
        self.synci(200)   # allow registers to settle

    def body(self):
        cfg = self.cfg
        self.trigger(adcs=[cfg["adc_ch"]], adc_trig_offset=cfg["adc_trig_offset"])
        self.pulse(ch=cfg["res_ch"])
        self.wait_all()
        self.sync_all(self.us2cycles(cfg["relax_delay"]))
```

## Key tprocv1 instructions

| Instruction | Purpose |
|---|---|
| `self.measure(pulse_ch, adcs, adc_trig_offset, wait, syncdelay)` | Fire pulse + arm ADC in one call |
| `self.synci(cycles)` | Advance tProcessor time only (leaves ADC timeline alone) |
| `self.sync_all(cycles)` | Advance ALL timelines to the furthest point + extra delay |
| `self.wait_all()` | Stall until all ADC windows close |
| `self.condj(page, reg_a, op, reg_b, label)` | Conditional branch |
| `self.regwi(page, reg, value)` | Write immediate to register |
| `self.mathi(page, out, in1, op, in2)` | Register arithmetic |

**Critical timing rule:** inside a `body()` that has an open ADC window, use `synci` to advance time between pulses. `sync_all` would jump past the end of the ADC window, scheduling subsequent pulses after the window has already closed.

## Config dict conventions (tprocv2)

```python
cfg = {
    "qubit_ch"   : 0,       # DAC driving qubit (tprocv2 channel index)
    "res_ch"     : 1,       # DAC driving resonator
    "adc_ch"     : 0,       # ADC channel
    "qubit_freq" : 4500.0,  # MHz
    "res_freq"   : 6505.1,  # MHz
    "res_gain"   : 0.3,     # 0–1 DAC full-scale fraction
    "raw_length" : 1024,    # ADC buffer samples
    "relax_delay": 500.0,   # µs between shots
    "shots"      : 2000,    # reps passed to AsmV2
}
```

## Phase calibration (tprocv1)

The random DDS phase offset at power-on follows `φ(f) = φ₀ − 360·τ·f` (τ ≈ 251 µs). Calibrate once per session with a loopback sweep, then set `cfg["res_phase"] = soccfg.deg2reg(phi_res)`.

## Resonator spectroscopy sweep

Use a Python outer loop (not `RAveragerProgram`) so that both the DAC NCO and the ADC NCO are updated together:

```python
for f in freqs:
    cfg["pulse_freq"] = f
    prog = AveragerProgram(soccfg, cfg)
    iq = prog.acquire(soc, load_pulses=True, progress=False)
    amps.append(np.abs(iq[0][0][0] + 1j*iq[0][0][1]))
```

`RAveragerProgram.update()` can only change DAC-side registers; the ADC NCO requires an AXI bus write from the ARM core.

## Conditional active reset (tprocv1 idiom)

```python
def body(self):
    # 1. Readout
    self.trigger(adcs=[cfg["ro_ch"]], adc_trig_offset=cfg["adc_trig_offset"])
    self.pulse(ch=cfg["res_ch"])
    self.wait_all()
    # 2. Threshold decision written to register by FPGA
    # 3. Conditional π-pulse
    self.condj(0, self.r_decision, '<', 1, 'SKIP_PI')
    self.pulse(ch=cfg["qubit_ch"])   # π-pulse
    self.label('SKIP_PI')
    self.synci(self.us2cycles(cfg["relax_delay"]))
```

Labels are resolved at compile time by the Python assembler; no runtime symbol table exists.
