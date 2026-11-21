---
title: "Running physics simulations with `ddsim`"
teaching: 30
exercises: 20
---

::::::::::::::::::::::::::::::::::::::::::::: questions

- How can I simulate events from physics event generators?

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: objectives

- `ddsim` can simulate HepMC3 events in DD4hep geometries.
- `npsim` can be used as an alternative for simulations with optical photons.

:::::::::::::::::::::::::::::::::::::::::::::

We now move on to running simulations on HepMC3 event input files from (in this case) Pythia8.

## Using centrally produced input files

The large input files for simulation campaigns are stored on xrootd, but the `eic-shell` environment allows us to access them. We can connect to the server and take a look at the file:

```bash
xrdfs root://dtn-eic.jlab.org
ls /volatile/eic/EPIC/Tutorials/pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hepmc
exit
```

This input file is large (GBs). For this tutorial we only need the first few thousand lines. We can do this for the first 20000 lines using the following command:

```bash
xrdfs root://dtn-eic.jlab.org cat /volatile/eic/EPIC/Tutorials/pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hepmc | head -n 20000 > pythia8NCDIS_10x100.hepmc
```

We can now specify this HepMC3 input file as input to `ddsim`:

```bash
ddsim --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml --numberOfEvents 10 --inputFiles pythia8NCDIS_10x100.hepmc --outputFile pythia8NCDIS_10x100.edm4hep.root
```

Instead of downloading files, we can also request events on-demand from the publicly accessible EIC XRootD server, but in this case we must use the `hepmc3.tree.root` input file extension:

```bash
ddsim --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml --numberOfEvents 10 --inputFiles root://dtn-eic.jlab.org//work/eic2/EPIC/Tutorials/pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hepmc3.tree.root --outputFile pythia8NCDIS_10x100.edm4hep.root
```

## Creating your own input files

Rather than relying on the centrally produced events, we can also create events ourselves. This gives us flexibility to run on and off certain event generator effects.

### Head-on versus rotated collision frames: the "afterburner"

In this exercise, we will use Pythia8 to generate DIS neutral current interactions, but we could use other event generators as well. However, we have to pay attention to the reference frames in which interactions are generated. Most event generators are set up to generate events in the head-on collision frame of reference. This is not the reference frame in which beams collide at the EIC: the beams have a crossing angle of -0.025 mrad. In addition, there are beam energy smearing effects that cause the beam energies to deviated from the 'exact' values indicated in the settings: a 10 GeV electron beam contains in reality electrons with energies distributed around 10 GeV. To correct for crossing angle and beam energy smearing, we can modify the event generator or we can apply an "afterburner" which rotates and boosts events from the head-on fram into the correct frame. The afterburner is beyond the scope of this episode, but can be found at [eic/afterburner](https://github.com/eic/afterburner).

### Using Pythia8 with crossing angle and beam energy corrections

Rather than relying on the afterburner, we have modified Pythia8 to include the required corrections directly upon event generation. The steering code and input files can be found at [eic/eicSimuBeamEffects](https://github.com/eic/eicSimuBeamEffects), so we start with using git to obtain this code.

```bash
git clone https://github.com/eic/eicSimuBeamEffects
```

We can compile the code inside the `eic-shell` environment (which includes the Pythia8 event generator libraries that are used by this simulation):

```bash
cd eicSimuBeamEffects/Pythia8
make
```

After compilation, we can use the executable `runBeamShapeHepMC.exe` to generate events, but we need to provide some arguments:

```bash
./runBeamShapeHepMC.exe
Wrong number of arguments
program.exe steer configuration hadronE leptonE xangle out.hist.root out.hepmc
```

The various steering files in `steerFiles` contain various beam conitions. Here we will use the 10 GeV electron on 100 GeV proton conditions in the high beam divergence setting (`hiDiv`), or the steering file `dis_eicBeam_hiDiv_10x100`. The `hiDiv` setting requires the `configuration` flag value `1` (as explained in the `README.md` file).

```bash
./runBeamShapeHepMC.exe steerFiles/dis_eicBeam_hiDiv_10x100 1 100 10 -0.025 \
  pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hist.root \
  pythia8NCDIS_10x100_minQ2=1_beamEffects_xAngle=-0.025_hiDiv.hepmc
```

We can now run the output files through the `ddsim` simulation as before.

## NPSim as an alternative to `ddsim`

Since some of the options that we pass to `ddsim` can only be provided through a steering file (such as python functions), or are otherwise cumbersome to provide on the command line, we provide `npsim` as a layer on top of `ddsim` that has these options pre-configured. This is as if you would take your steering file options and contribute them back to a central location for others to use them.

`npsim` can be easily interpreted (since it has sections that look exactly like the steering file). We can look at its python source code, located at `/opt/local/bin/npsim.py` in the `eic-shell` environment.

Currently `npsim` has the following additional options:

- Cerenkov and optical photon physics are added through a python setup function,
- an optical photon filter is created and added to the DRICH,
- a modified tracking detector action which absorbs optical photon,
- the minimal energy deposition in the tracker is set to zero.

You can run `npsim` exactly as you would run `ddsim`.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise: compare `npsim` with `ddsim`

- Rerun the previous Pythia8 simulation with `npsim`, and notice any difference in running time. Because of the addition of optical photon physics, the simulation will run more slowly.
- Open the output file and verify that more hits (from optical photons with PDG code -22) are stored in the hits branches for the relevant RICH detector.

::::::::::::::: solution

Running `npsim` with the same `--compactFile`, `--inputFiles`, and `--outputFile` arguments as the
earlier `ddsim` command takes noticeably longer because optical photon physics is now simulated.
Inspecting the DRICH `*Hits` branch in the output shows many additional hits, including optical
photons (PDG code -22), that were absent from the `ddsim` output.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: callout

For most full physics analyses, the use of `npsim` is strongly preferred. The performance of some detector systems, including Cherenkov detectors as mentioned above, will be very different if `ddsim` is used. Similarly, trackers and calorimeters may appear to perform very differently. For some specialist use cases, `ddsim` may be preferred. If you are unsure whether such cases apply, use `npsim` when running your simulations.

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: keypoints

- `ddsim` or `npsim` are both able to simulate physics events.
- **`npsim` should be used in almost all regular use cases for ePIC simulations**.

:::::::::::::::::::::::::::::::::::::::::::::
