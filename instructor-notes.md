---
title: "Instructor Notes"
---

This tutorial follows the [environment setup](https://eic.github.io/tutorial-setting-up-environment/)
and [geometry development with DD4hep](https://eic.github.io/tutorial-geometry-development-using-dd4hep/)
tutorials and assumes learners already have a working `eic-shell`.

## Before the session

- Ask learners to complete the [Setup](../learners/setup.md) page in advance and to have `eic-shell`
  installed and working. On systems without `/cvmfs` the container download is large and should not
  be done live.
- The physics-event episode reads large HepMC3 input files from the EIC XRootD server. Network
  access to `root://dtn-eic.jlab.org` from the training systems should be confirmed beforehand, and
  learners should be aware that centrally produced files are many GB in size.

## Timing

Each episode is roughly 30 minutes of teaching and 20 minutes of exercises. The first `ddsim` run
against cvmfs can be slow the first time, as Geant4 data files are fetched; budget extra time.
