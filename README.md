# Inverse-Designed Bus–Ring Coupler for a Ring Resonator (Meep)

Adjoint-based inverse design of the coupling region of a silicon-photonic add-drop ring resonator with [Meep](https://meep.readthedocs.io). Runs in Google Colab.

## Workflow
1. **Target → κ²:** a coupled-ring model with the measured loop loss converts a target loaded Q into the required cross-coupling power κ².
2. **Coupler optimization:** a four-port 6 × 4 µm region (bus + ring waveguide) is optimized so the cross power matches κ² over 1.50–1.60 µm, with minimal reflection and backward coupling.
3. **Assembly:** two copies (the second rotated by 180°) form a racetrack ring; full-ring FDTD, a lossy-coupler model fit, and GDS export.

## Method
- Objective: relative error of cross and through power plus reflection penalty, averaged over 5 wavelengths
- Parametrization: mirror symmetry about the port mid-line, solid port stubs, conic filter (r = 0.15 µm), tanh projection with β = 4 → 64
- Start design: two waveguides approaching to a 150 nm gap (uniform gray start converged to "no coupling")
- Optimizer: Adam on the adjoint gradient; final design binarized (β = 256) and re-simulated at higher resolution
- Optional `REFINE` step: continue from the saved design with extra weight on the cross-power error

## Parameters
| Parameter | Value |
|---|---|
| Platform | 220 nm Si on SiO2, 2D effective index, TE-like |
| Waveguide width | 450 nm |
| Port separation (bus to ring) | 1.2 µm |
| Design region | 6 × 4 µm |
| Loop radius | 5 µm, ring length ≈ 43 µm |
| Target | loaded Q = 1500 with assumed loop loss 0.78 dB → κ² = 0.092 |

## Results (2D effective-index FDTD; some values read from plots)
| Quantity | Value |
|---|---|
| Coupler excess loss | 0.14 dB (band mean) |
| Reflection / backward coupling | below -29 dB / below -46 dB |
| Phase between through and cross | 88.7° |
| κ² (target 0.092) | mean 0.074, range 0.059–0.091 over 1.50–1.60 µm |
| Ring resonances | 1.5377 µm and 1.5549 µm |
| FSR | 17.1 nm measured (17.3 nm fit, 17.4 nm expected) |
| Loaded Q | ≈ 2000 (2030 / 2080; sampling 0.1 nm) |
| Drop peak | -5.2 dB / -4.2 dB |
| Through extinction | 9.9 dB |
| Fitted κ² of the full ring | 0.072 (isolated coupler: 0.074) |

The fitted round-trip loss reached the lower bound of the model (≈ 0 dB), so it is not resolved by this data. A separate half-loop test gave 0.25 dB per semicircle.

## Limitations
- κ² is about 19% below the target and not flat in wavelength; Q is therefore higher than the 1500 target.
- 2D effective-index model: validate with 3D FDTD before fabrication.
- Q accuracy is limited by 0.1 nm spectral sampling (about ±10%).

## Lessons
- Start from a physically sensible design; a uniform gray start got stuck in the "straight bus" optimum.
- Use `DECAY_TOL = 1e-8` for rings and check `max(T+D+R) ≤ 1`; a loose tolerance produced power > 1 and fake ripples.

## Usage
1. Open `InverseDesign_Ring_Step1.ipynb` in Colab and run the install cells.
2. Set `QUICK_TEST = True` first; then run fully.
3. Edit `Q_TARGET` and `A_RT_DB` for your target and loop loss.

Outputs: `ringcoupler_design_rho.npy`, `invring_results.npz`, `invdes_ring.gds`.

## Requirements
`pymeep`, `autograd`, `numpy`, `scipy`, `matplotlib`, `gdstk`.
