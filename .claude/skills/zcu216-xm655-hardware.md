# ZCU216 + XM655 hardware conventions

## Board overview

| Property | Value |
|---|---|
| FPGA | Xilinx Zynq UltraScale+ RFSoC |
| DAC channels | 16 × 14-bit, up to 9.85 GSPS |
| ADC channels | 16 × 12-bit, up to 2.5 GSPS |
| Processor | 4× ARM Cortex-A53 @ 1.5 GHz |
| OS | PYNQ Linux (boots from SD card) |
| FPGA fabric clock (QICK) | 384 MHz → 1 cycle ≈ 2.6 ns |

## Channel mapping (default QICK bitstream)

| QICK name | Physical port | HD header | Usage in this thesis |
|---|---|---|---|
| Generator 6 | DAC 2_231 | JHC3 | Readout / control drive |
| Readout 0 | ADC 0_226 | JHC5 | IQ acquisition |

The tprocv2 notebooks use a different numbering (ch=0, ch=1) that maps to the first available generators in the loaded bitstream — check `soccfg.get_cfg()` to confirm the mapping for a given bitstream.

## XM655 analogue front-end

- Provides BALUN (balanced-to-unbalanced) transformers converting differential HD-header signals to single-ended SMA.
- BALUN frequency range: **10 MHz – 1 GHz** only. For resonators above 1 GHz, use pigtail cables directly on the HD header pins + discrete BALUNs.
- Frequency bands above ~3 GHz: limited BALUN-equipped SMA ports; check XM655 schematic for which SMA connectors cover which channel.

## Loopback path (calibration)

```
DAC 2_231 (JHC3) → HD2-SMA pigtail → BALUN → SMA cable → BALUN → HD2-SMA pigtail → ADC 0_226 (JHC5)
```

Round-trip cable delay: τ ≈ 251 µs (dominates over on-chip delay).

## Session startup checklist

1. Power on; wait ~60 s for PYNQ to boot.
2. SSH into the board. If not already running, start the Pyro4 server:
   ```python
   import Pyro4, qick
   soc = qick.QickSoc()
   daemon = Pyro4.Daemon()
   uri = daemon.register(soc, "qick.soc")
   Pyro4.locateNS().register("qick.soc", uri)
   daemon.requestLoop()
   ```
3. From laptop: `soc = Pyro4.Proxy("PYRONAME:qick.soc")`
4. Connect loopback and run phase calibration (must re-run after every reboot).
5. Use `acquire_decimated()` to find `adc_trig_offset` and verify signal shape.
6. Switch to `acquire()` (accumulated) for all data-taking.
7. At end of session: `soc.reset_gens()` to halt all running generators.

## Phase calibration

The DDS oscillators boot with a random mutual phase offset each power cycle. Compensate with a loopback frequency sweep:

```python
# Fit φ(f) = φ₀ − 360·τ·f  using scipy.optimize.least_squares
# Handle 0°/360° wrap before fitting
phi_res = phi0 - 360 * tau * f_res   # phase at readout frequency
cfg["res_phase"] = soccfg.deg2reg(phi_res)
```

Calibration is valid for the entire session; must be re-run after reboot or bitstream reload.

## NQZ (Nyquist zone) selection

- `nqz=1`: fundamental Nyquist zone (0 to f_clk/2). Use for signals below ~2.5 GHz on ADC, ~5 GHz on DAC.
- `nqz=2`: second Nyquist zone (f_clk/2 to f_clk). DAC image in this zone; ADC aliases back. Required for high-frequency tones without upconversion.
- Set in `declare_gen(ch=..., nqz=...)` and `declare_readout(ch=..., ...)`.

## ADC buffer rate

After the 8× internal decimation fabric, the effective ADC sample rate seen by the raw buffer is ~512 MSPS. Rule of thumb: 1 µs ≈ 512 buffer samples. Set `raw_length` in samples, not microseconds.
