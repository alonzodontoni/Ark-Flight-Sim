# Ark: a 6DOF rocket flight simulator

A six-degree-of-freedom flight simulator for finned, passively stable rockets, written in Python, with
Monte Carlo landing dispersion and a browser replay. It flies a rocket from the pad to the ground: launch
rail, powered flight, coast, parachute descent, in a real atmosphere with wind.

The starting idea came from [yst-2803/rocket-sim](https://github.com/yst-2803/rocket-sim), a 1D vertical
model of an Estes Alpha III. This project keeps that repo's habit of checking everything, and replaces the
physics: 6DOF rigid-body dynamics with quaternions, a standard or recorded atmosphere, Barrowman stability,
a drag build-up from geometry, variable mass and inertia, wind and gusts, recovery, and uncertainty analysis.

Open `web/index.html` in a browser for the 3D replay. It is one self-contained file holding a searchable library
of 14 rockets (13 of them really flew, with measured apogees). Pick one and it flies at once, in the browser, using
a JavaScript port of the same equations (`web/sim.js`) that the test suite checks against the Python core. You can
change the wind, rail tilt, drag, motor impulse or mass and watch it re-fly, race two rockets, and run 200
Monte Carlo variations live to see where it would land.

## Results

Apogee above ground for three rockets. Two of them really flew, with altimeter data.

| Rocket | This simulator | Measured | RocketPy | OpenRocket | Error |
| --- | --- | --- | --- | --- | --- |
| Estes Alpha III, C6-7 | 376.8 m | none | not run | 385.5 m | -2.3% against OpenRocket |
| EPFL Bella Lui (2020), K828FJ | 461.5 m | 459.0 m | 460.5 m | not run | **+0.5% against the flight** |
| Notre Dame Rocket Team (2020), L1395 | 1287.0 m | 1320.3 m | 1293.4 m | not run | **-2.5% against the flight** |

Sources: the OpenRocket figure is from the inspiration repo's README; the RocketPy and measured figures are
from RocketPy's published examples ([Bella Lui](https://docs.rocketpy.org/en/latest/examples/bella_lui_flight_sim.html),
[NDRT 2020](https://docs.rocketpy.org/en/latest/examples/ndrt_2020_flight_sim.html)), and the measured
apogees were re-read from the flight data files in `data/flights`. Neither OpenRocket nor RocketPy was run
for this project; their numbers are quoted.

What these comparisons do and do not show:

- The two team rockets use the teams' own drag coefficients, masses and the recorded weather, the same
  inputs RocketPy used. So the agreement tests the dynamics, atmosphere, rail and integration, not the
  drag prediction.
- With this project's own drag build-up in place of the teams' values, Bella Lui gives 462.7 m (+0.8%) and
  Notre Dame gives 1424 m (**+7.9%**). The build-up predicts C_d = 0.31 for the Notre Dame rocket where the
  team used 0.44; it knows nothing about that rocket's protuberances. For the Alpha III the build-up gives
  C_d = 0.60 against OpenRocket's 0.56, and 367.5 m.
- Descent is less accurate than ascent. The simulated Notre Dame rocket falls faster under its drogue than
  the real one did (the tumbling airframe's drag is not modelled), so it lands 397 m from the pad where the
  team recorded about 690 m of drift.

### The library: thirteen real flights

Eleven more university rockets were added from the inputs their teams published through RocketPy's documentation
(`scripts/ingest_library.py`). Each has a measured apogee and RocketPy's own published simulation of it, so every
rocket gives two comparisons: against another simulator fed the same inputs, and against reality.

| Rocket | Team | Measured | RocketPy (published) | This simulator | vs RocketPy | vs measured |
| --- | --- | --- | --- | --- | --- | --- |
| NDRT launch vehicle (2020) | Notre Dame Rocket Team | 1,320 m | 1,293 m | 1,287 m | -0.5% | -2.5% |
| Bella Lui (2020) | EPFL Rocket Team | 459 m | 461 m | 461 m | +0.2% | +0.5% |
| Valetudo (2019) | Projeto Jupiter | 860 m | 825 m | 813 m | -1.5% | -5.5% |
| Andromeda (2022) | Aristotle Space and Aeronautics Team | 3,415 m | 3,615 m | 3,615 m | +0.0% | +5.9% |
| Astra (2022) | Faraday Rocketry UPV | 3,250 m | 3,043 m | 3,042 m | -0.0% | -6.4% |
| Erebus-11 (2022) | BME Suborbitals | 3,020 m | 2,751 m | 2,751 m | -0.0% | -8.9% |
| Cavour (2023) | PoliTo Rocket Team | 2,789 m | 2,819 m | 2,823 m | +0.1% | +1.2% |
| Genesis (2023) | Faraday Rocketry UPV | 2,916 m | 3,076 m | 3,047 m | -0.9% | +4.5% |
| Halcyon (2023) | Aerospace Team Graz | 3,450 m | 3,213 m | 3,222 m | +0.3% | -6.6% |
| Juno III (2023) | Projeto Jupiter | 3,213 m | 3,026 m | 3,081 m | +1.8% | -4.1% |
| Lince (2023) | STAR UC3M | 3,587 m | 3,284 m | 3,285 m | +0.0% | -8.4% |
| Prometheus (2023) | Western Engineering Rocketry Team | 3,898 m | 4,190 m | 4,182 m | -0.2% | +7.3% |
| Defiance Mk. IV (2024) | University of Toronto Aerospace Team | 9,308 m | 9,238 m | 9,219 m | -0.2% | -1.0% |

- **Against RocketPy, same inputs: 0.45% mean difference**, worst 1.8%. Two independently written simulators
  agreeing this closely across 0.5 to 9 km, solid and hybrid motors, is the strongest evidence here that the
  equations of motion, atmosphere and integration are right.
- **Against the real flights: 4.8% mean error** (RocketPy's published results: 4.9%). The two simulators miss
  reality by the same amount in the same direction on every rocket, so that error comes from the published inputs
  (drag curves, masses, thrust curves, weather), not from either simulator.
- Two entries are hybrids (Halcyon, Defiance) with the propellant lumped at a fixed position, and two exceed
  Mach 1 (Defiance 1.36, Andromeda 1.01), where this model's normal-force treatment is not validated. Their
  apogees still agree with RocketPy because drag comes from the teams' own curves.

![Bella Lui](docs/validation_bella_lui.png)
![Notre Dame](docs/validation_ndrt_2020.png)

## Verification

112 automated tests (`pytest`), each against an answer known from outside the code (five of them need Node, to
run the browser port, and are skipped without it). The main ones:

| Check | Expected | Result |
| --- | --- | --- |
| Standard atmosphere at 0, 11 and 20 km | 1976 standard tables | matches to 4 significant figures |
| Three motor files: total impulse and class | 8.82, 2071, 4895 N s; C, K, L | within 0.5% |
| Barrowman worked example | CP at 443.8 mm, C_Na 8.827 | matches |
| Vacuum ballistic flight | range 1019.72 m, apex 254.93 m, 14.421 s | matches, energy constant |
| Terminal velocity and vertical coast with drag | 51.93 m/s; 213.02 m in 5.782 s (closed form) | matches |
| RK4 order on a smooth problem | 4 | 3.7 to 3.9, approaching 4 |
| Torque-free spinning body | precession period 0.6347 s, angular momentum constant | matches |
| Pitch oscillation in a fixed airstream | 15.78 rad/s, damping ratio 0.0837 | 15.777 rad/s, 0.0837 |
| Regression against the inspiration repo (constant density, C_d 0.56) | 373.4 m, 116.2 m/s, 7.94 s | 373.5 m, 116.1 m/s, 7.94 s |
| Wind from the west, then the east | mirror-image flights, nose into the wind | matches |
| Roll angle at rail exit | no effect on a symmetric rocket | no effect |
| CP ahead of CG | tumbles | tumbles, run completes |
| Monte Carlo with 1 and 2 workers, same seed | identical | identical |
| 95% ellipse on 20,000 synthetic points | holds 95% | within 0.7 points of 95% |
| Browser port (`web/sim.js`) against the Python core, three flights | same apogee | within 0.2% (0.001% as built) |
| Eleven library rockets against RocketPy's published apogee | same inputs, same answer | all within 2.5% (mean 0.45%) |

## Findings

- **A real atmosphere explains only part of the inspiration repo's gap.** Its author put the 3.1% shortfall
  against OpenRocket down to constant air density. Switching from constant sea-level density to the standard
  atmosphere raises the Alpha III's apogee by 3.3 m (0.9%), about 28% of that gap. For the Notre Dame flight
  to 1.3 km the same switch is worth 2.0%.
- **Fin edge shape dominates small-rocket drag.** In the build-up, squaring off the Alpha III's fin edges
  takes C_d from 0.60 to 0.96 and costs 23% of apogee (OpenRocket: 0.56 to 1.08).
- **Weathercocking.** A vertical-rail Bella Lui in a 10 m/s wind leaves the rail at 29 degrees angle of
  attack, turns into the wind, reaches apogee 162 m upwind, and loses 6.8% of its still-air apogee.
- **Variable-mass rotation terms barely matter here.** Jet damping and the inertia-rate term change apogee
  by less than 0.2 m for all three rockets.
- **Transverse inertia matters more than expected at rail exit.** RocketPy's listed inertia for the Notre
  Dame rocket (73.3 kg m^2) looks too large for its mass and length; this project uses a rod estimate
  (18.2). The larger value resists the turn into the wind at rail exit and gives 1301 m instead of 1287 m.
  The true value is unknown, so this is a 1% uncertainty in the validation, not a correction.
- **What drives the outcome (500-run Monte Carlo, Notre Dame).** Apogee is 1276 +/- 69 m, dominated by total
  impulse (rank correlation 0.80), then drag and mass. The 95% landing ellipse has semi-axes of 465 m by
  406 m. One at a time, thrust misalignment of 0.2 degrees moves the landing point about 150 m and wind
  direction (15 degrees) about 105 m. Note that a "2 sigma" ellipse in two dimensions holds 86.5% of points,
  not 95%; the 95% ellipse is 2.45 sigma.

![Monte Carlo](docs/montecarlo_ndrt_2020.png)
![Sensitivity](docs/tornado_ndrt_2020.png)

More plots are in `docs/`: drag build-up against Mach, static margin through the burn, weathercocking, the
same Monte Carlo and sensitivity study for Bella Lui, and RK4 convergence on the full flight (which does not
show clean 4th order, because the thrust curve is only piecewise linear).

## Limits

- Flat, non-rotating Earth; rigid body; no roll moments (no fin cant, no roll damping).
- Barrowman assumptions: small angles, subsonic, body lift neglected. The normal force uses sin(alpha) so it
  stays bounded when a rocket tumbles, which is a convenience, not a validated model.
- The drag build-up is subsonic-first. Transonic and supersonic branches exist but nothing here tests them;
  no reference rocket exceeds Mach 0.53.
- On the rail the wind is ignored and rail friction is zero.
- Parachutes are point drag with a linear inflation ramp. Airframe drag during descent is not modelled.
- Gusts are a first-order Gauss-Markov process in time, simpler than the Dryden model.
- The Alpha III geometry and component masses are approximate; only its total masses are matched.
- The two team rockets use lumped mass properties with estimated transverse inertia (see Findings).
- Uncertainty spreads in the configs are assumptions, not measurements.
- The browser port flies the recorded wind or one constant wind, with no gusts and no thrust misalignment, and
  takes drag from a constant or a table. Its live Monte Carlo therefore gives a narrower landing spread than the
  Python study. The Python core is the reference.
- The library is the set of rockets whose inputs are published; it is built offline by `scripts/ingest_library.py`.
  The page cannot search the internet for new rockets. Adding one means adding its parameters to that script.
- Library rockets use the teams' published mass properties and drag curves as given, including inertia values that
  look doubtful, and hybrid propellant is lumped at a fixed position.
- The CI workflow is included but has not been run on GitHub.

## Use

```
pip install -e ".[dev]"
pytest                                    # 112 tests, about 30 s
ark info data/rockets/ndrt_2020.yaml
ark run  data/rockets/ndrt_2020.yaml -o flight.json
ark mc   data/rockets/ndrt_2020.yaml -n 500 -o mc.csv
python scripts/validate.py                # headline table, overlay plots, example trajectories
python scripts/analysis.py 500            # findings, sensitivity, Monte Carlo (about 7 minutes on 2 cores)
python scripts/ingest_library.py <RocketPy checkout>   # regenerate data/library from the published inputs
python scripts/build_web.py               # rebuild web/index.html and docs/library.json
```

### Viewing and sharing the replay page

- On your own computer: open `web/index.html` in a browser. No server or install is needed; everything is
  inside that one file (it fetches its fonts from Google Fonts and falls back to system fonts offline).
- On the web with GitHub Pages: push this repository to GitHub, then in the repository's Settings, under
  Pages, choose "Deploy from a branch", the `main` branch and the `/ (root)` folder. The page is then at
  `https://<your-username>.github.io/<repository-name>/web/`.
- After changing a rocket or the model, run `validate.py`, `analysis.py` and `build_web.py` (above) to rebuild
  `web/index.html`, then commit it.

A rocket is a YAML file: components, motor file, launch site, wind, solver settings, uncertainties. See
`data/rockets/`. Unknown keys and missing fields are errors.

## Layout

```
src/ark/
  environment.py   standard atmosphere, sounding, gravity, wind and gusts
  motor.py         RASP .eng parser, thrust / impulse / propellant mass
  geometry.py      nose profiles, shells of revolution, fin planform
  massprops.py     mass, CG and inertia against time
  aero.py          Barrowman normal force and CP; drag build-up; lookup and constant backends
  quaternion.py    quaternion algebra and rail attitude
  dynamics.py      rail (1D), free flight (6DOF) and descent (3DOF) equations of motion
  integrate.py     RK4 with event location; SciPy wrapper
  flight.py        phase sequencing, events, summary
  config.py        load, validate, build
  output.py        trajectory JSON for the web page
  webmodel.py      exports a rocket as plain data for the browser port
  montecarlo.py    sampling, parallel runs, dispersion ellipse, sensitivity
  cli.py
data/              motors, rocket configs, recorded weather, flight data, and library/ (sources in data/README.md)
tests/             112 tests
scripts/           validate.py, analysis.py, ingest_library.py, build_web.py
web/               template.html, sim.js (browser port), index.html (built, self-contained), example trajectories
docs/              model.md (equations and conventions), plots, results as JSON and CSV
```

## References and credit

- J. S. Barrowman, *The Practical Calculation of the Aerodynamic Characteristics of Slender Finned Vehicles*, 1967.
- S. Niskanen, *OpenRocket technical documentation* (drag build-up formulas, chapter 3).
- S. Box, C. M. Bishop, H. Hunt, *Stochastic Six-Degree-of-Freedom Flight Simulator for Passively Controlled
  High-Power Rockets*, J. Aerospace Engineering 24(1), 2011.
- *U.S. Standard Atmosphere, 1976*, NASA-TM-X-74335.
- [yst-2803/rocket-sim](https://github.com/yst-2803/rocket-sim): the inspiration. No code from it is used.
- [RocketPy](https://github.com/RocketPy-Team/RocketPy): source of the teams' published inputs, weather and flight
  data, and of the comparison figures. No RocketPy code is used or run; its results are quoted as published.
