# A simplified quantum wave simulator (1D & 2D, split-operator method)

This solves the time-dependent Schrodinger equation

$$
i\hbar \frac{\partial \psi}{\partial t} = -\frac{\hbar^2}{2m} \nabla^2 \psi + V(x,t)\psi
$$

for a complex-valued wavefunction ψ, using the **split-operator (Strang
splitting) Fourier method**. Units are natural (ħ = m = 1) unless stated
otherwise.

## Why split-operator 

A naive explicit scheme — forward Euler in time, central differences in
space — applied directly to the Schrodinger equation is **unconditionally
unstable**: its update matrix is not unitary, so $\int |\psi|^2 \, \mathrm{d}x$ grows
exponentially no matter how small `dt` is (see `demo_stability.py` — the
naive scheme blows past 10⁸ in norm by t ≈ 0.6, while nothing else about
the run has changed). The fix used here — the split-operator method —
approximates the propagator as

$$
\exp\left( -\frac{iH\,dt}{\hbar} \right) \approx \exp\left( -\frac{iV\,dt}{2\hbar} \right) \cdot \exp\left( -\frac{iT\,dt}{\hbar} \right) \cdot \exp\left( -\frac{iV\,dt}{2\hbar} \right) + \mathcal{O}(dt^3)
$$

Each factor is applied *exactly*: the potential factor is a pointwise phase
in real space, and the kinetic factor $T = \frac{p^2}{2m}$ is a pointwise phase in
Fourier space (computed via FFT). Both factors are individually unitary, so
the composed step conserves $\int |\psi|^2$ to machine precision at *any* step size
— this is verified in every demo below and directly contrasted with the
unstable naive scheme in `demo_stability.py`.

The grid is periodic (as FFTs require), so **absorbing boundary layers**
(polynomial complex absorbing potentials, CAP) are used at the domain edges
to soak up outgoing probability flux instead of letting it wrap around.

## Project layout

```
core.py                 - Grid1D/Grid2D, SplitOperator1D/2D solvers,
                           potentials, CAP boundaries, probability current
demo_tunneling.py        - 1D: wave packet tunneling through a rectangular
                            barrier; compares simulated transmission to the
                            analytic textbook formula
demo_double_slit.py       - 2D: single-slit diffraction & double-slit
                            interference; compares fringe spacing to
                            λL/d
demo_scattering.py        - 2D: wave packet scattering off a smooth
                            Gaussian potential bump; shows the probability
                            current field splitting around the obstacle
demo_harmonic.py           - Harmonic oscillator: (A) eigenvalues from
                            direct diagonalization vs analytic (n+1/2)ħω;
                            (B) coherent-state dynamics — ⟨x⟩(t) tracks the
                            classical trajectory, energy stays constant
demo_stability.py          - Naive explicit FD vs Crank-Nicolson (implicit,
                            unitary) vs split-operator: demonstrates the
                            instability the exercise warns about, and why
                            the two unitary schemes don't suffer from it
output/                    - all generated PNGs and GIFs
```

Run any demo standalone: `python3 demo_tunneling.py`, etc. All of them
print numeric verification (normalization, comparison to analytic results)
to stdout in addition to saving figures.

## What each requirement maps to

| Requirement | Where |
|---|---|
| Complex-valued wavefunction | `core.py`: all `psi` arrays are complex, propagated with genuinely complex phase factors |
| Tunneling | `demo_tunneling.py` |
| Diffraction | `demo_double_slit.py` (`single_slit_diffraction_*`) |
| Double-slit interference | `demo_double_slit.py` (`double_slit_interference_*`) |
| Wave-packet scattering | `demo_scattering.py` |
| Display \|ψ\|² | all demos (heatmaps / line plots) |
| Display phase | `demo_tunneling.py` (`tunneling_summary.png`, middle panel) |
| Display probability current | `demo_tunneling.py` (bottom panel) and `demo_scattering.py` (`scattering_current.png`, quiver plot) |
| Verify normalization | every demo tracks and plots `∫\|ψ\|² dx` (or `dx dy`) over time |
| **Difficult extension** — absorbing boundary layers | `cap_1d`/`cap_2d` in `core.py`, used in every propagating demo |
| **Difficult extension** — time-dependent potentials | `SplitOperator1D/2D` accept `V(x,t)` as a callable and evaluate it at the half-steps of the Strang split; not exercised in a dedicated demo here but supported directly (pass a function instead of an array) |
| **Difficult extension** — split-operator Fourier method | this *is* the core method used throughout |
| **Difficult extension** — compare finite differences vs spectral | `demo_stability.py` |
| **Difficult extension** — harmonic oscillator eigenvalues vs analytical | `demo_harmonic.py` Part A |

Not implemented (flagged rather than faked): continuous measurement /
wavefunction collapse. That requires a stochastic unraveling of the master
equation (quantum trajectories) layered on top of this unitary propagator —
a substantial addition in its own right rather than a natural extension of
the split-operator solver, so I left it out rather than bolt on something
half-right.

## Selected results

- **Tunneling**: for E₀ = 4.5 below a barrier of height V₀ = 6 and width
  1.5, the single-energy plane-wave formula predicts T ≈ 0.017; the
  simulated wave packet (which has a spread of energies, so its
  higher-k tail tunnels more easily) transmits P ≈ 0.031 — the right
  ballpark and the right direction of deviation.
- **Double-slit**: fringe spacing on the "screen" matches the elementary
  prediction  $\Delta y = \frac{\lambda L}{d}$ closely (see `double_slit_interference_summary.png`).
- **Harmonic oscillator**: numerically diagonalized eigenvalues match
  $\left(n + \frac{1}{2}\right)\hbar\omega$ to better than 1.3×10⁻⁴ over the lowest 8 levels; a
  displaced Gaussian's ⟨x⟩(t) exactly tracks the classical
  $x_0 \cos(\omega t)$ trajectory (coherent state, no spreading) with energy
  conserved.
- **Stability**: the naive explicit finite-difference scheme blows up
  (norm > 10⁸) within about half a natural time unit; Crank-Nicolson and
  the split-operator method both conserve norm to ~10⁻¹³ over the same run.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Summary of the Tunneling simulation :**
<img width="1260" height="1260" alt="Image" src="https://github.com/user-attachments/assets/161efa55-9245-4f2d-af44-9999985af1f1" />
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**Tunneling animation :**
<img width="800" height="450" alt="Image" src="https://github.com/user-attachments/assets/3d8d49b0-1cc3-470d-8951-c7026e60a13b" />
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
