---
title: 'Reference'
---

## Glossary

EIC
:   Electron-Ion Collider

ddsim
:   The DD4hep command-line driver for Geant4 detector simulations, including a built-in particle gun.

npsim
:   A thin layer on top of `ddsim` with ePIC-specific options (such as Cerenkov and optical photon
    physics) pre-configured. Recommended for almost all regular ePIC simulations.

eic-shell
:   The EIC standard software environment, a singularity/apptainer or docker container with a
    curated selection of software components used for EIC simulations and analysis.

EDM4hep
:   The common event data model used to store ePIC simulation output in ROOT files
    (`.edm4hep.root`).

HepMC3
:   The event-record format used to exchange generator-level events (e.g. from Pythia8) as input to
    simulation.
