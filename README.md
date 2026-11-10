# Spike test lab (simulator)

A single-page browser tool for exploring how spike-response tests behave. You build a hypothetical neuron, give it
a stimulus-locked response (or none), and run a set of statistical tests on its simulated spike trains. You can
also measure how often each test calls a non-responding cell significant. Every cell is generated in the browser;
the page contains and loads no recorded data.

## Features

- Build a simulated cell: baseline rate, interval distribution (exponential, gamma, log-normal, inverse Gaussian,
  inverse gamma, Gaussian, power law, refractory), number of trials, response direction, size, latency, jitter,
  duration and shape, slow drift, trial-to-trial gain and rate changes.
- Nine presets and a "New random cell" button.
- Raster, PSTH with SEM band, interval histogram and spike-count histogram with fitted laws.
- 14 tests on the same cell: paired Wilcoxon, T4 phase signed-rank, T4-pre, T4 or T4-pre, T4 + T4-pre (one BH
  family), BM (relative effect), Brunner-Munzel, signed log rate ratio, Cramer-von Mises timing (one- and
  two-sample), ZETA, ISI-resampling null, per-trial count tests, mean over trials.
- Per-test pictures (how the test sees the cell, and its null distribution) and assumption checks with flags.
- Window-length sweep: each test re-run with the response window ending at 0.25, 0.5, 1, 2, 3 s and a window
  derived from the simulated response.
- False-alarm check: simulates many null cells (optionally with bursty spikes, gain, drift, few trials) and shows
  the share called significant with 95% Clopper-Pearson intervals.
- Presenter mode (larger text and charts).

## Requirements

- A modern browser with Web Workers and canvas. Tested only with headless Chrome; other browsers were not tried.
- To serve the page locally: [uv](https://docs.astral.sh/uv/) (it provides Python's `http.server`), or any
  other static file server.
- Internet access is only used for the Google Fonts stylesheet (IBM Plex, Source Serif). The page does not need
  the network for anything else.

## Setup

No build step and no packages. Clone or copy the folder.

## Run

From this folder in Windows PowerShell:

```powershell
uv run --no-project python -m http.server 8803 --directory app
```

Then open <http://localhost:8803/index.html>. The command runs in the foreground until you stop it.

The page needs no data files. Serving it over http is the only way it was tested.

## How to use

1. **Build a cell** tab: pick a preset or move the sliders under "Cell settings". The raster, PSTH, summary
   tiles and distribution plots update.
2. **Run a test** tab: choose a method. The page shows the verdict, p value, observed statistic, the two
   pictures, the assumption checks, and a table of all methods on this cell. Test settings (windows, alpha, number
   of shuffles, shift null and so on) are under "Test settings". Press "Run window sweep" for the window-length
   comparison.
3. **False alarms** tab: choose problem toggles, the number of simulated cells and shuffles per test, then
   press "Run false-alarm check".

Settings are remembered in the browser's local storage. "New random cell" draws a new seed.

## How it works

- **Simulation**: a continuous spike train is generated with a baseline rate and the chosen interval law; events
  are placed with random gaps; on responding trials the rate is multiplied by a response profile (excitation or
  suppression, with latency, jitter, duration and shape). Optional slow rate fluctuation, trial gain, per-second
  rate changes and drift are applied on top. Trials are cut from -3 to +3 s around each event.
- **Tests**: each test compares a baseline window (default -2 to -0.25 s) with a response window (default 0-2 s),
  or tests spike timing inside the window. p values come from a shift null (spike trains rotated in time and
  re-cut at the events), from signed-rank or normal approximations, or from resampled inter-spike intervals,
  depending on the test and the setting. Multiple-testing correction uses Benjamini-Hochberg.
- **Methods the page names**: ZETA (with a Gumbel fit of the peak deviation, "ZETA paper" in the page), Brunner-Munzel
  test (Welch-Satterthwaite degrees of freedom), Cramer-von Mises statistics, scipy 1.16 signed-rank behaviour
  (Pratt zero handling, exact or normal approximation), Spearman rank correlation with a t approximation,
  Clopper-Pearson intervals, Benjamini-Hochberg.
- Heavy computations (window sweep, false-alarm check) run in Web Workers built from the page's own source.

## Project layout

```n app/index.html   the page (HTML, CSS, test engine, UI; no other files)
 LICENSE          GNU GPL v3
```n

## Limitations

- Simulated cells only. p values and false-alarm rates describe the simulator you set up, not any
  measured data.
- The tests are simplified teaching versions; shuffle counts are small (a few hundred), so the smallest possible
  p is about 1 / (shuffles + 1).
- False-alarm results from a few dozen cells are noisy; use 200 or more cells for a stable estimate.
- Fonts are loaded from Google Fonts. With those two hosts blocked the page still ran all three tabs (checked on
  localhost in headless Chrome); a fully offline machine was not tried.

## License

GNU General Public License v3.0, see `LICENSE`
