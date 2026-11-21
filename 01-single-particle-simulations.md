---
title: "Single Particle Simulations with `ddsim`"
teaching: 30
exercises: 20
---

::::::::::::::::::::::::::::::::::::::::::::: questions

- How can I simulate single particles for detector studies?

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: objectives

- Know where to find the available options for `ddsim`.
- Understand the differences between steering files and command line options.
- Know what the key output file collections are.

:::::::::::::::::::::::::::::::::::::::::::::

In this first episode we will go through the running of single particle events with `ddsim`, using the built-in event generator of `ddsim`. This is the quickest way so run some straightforward tests of the geometry and produce output hits in the detectors for further analysis.

::::::::::::::::::::::::::::::::::::::::::::: callout

You may also see reference to using `npsim` in some tutorials. We will discuss the difference between `ddsim` and `npsim`, along with cases where one should be used over the other, in the next episode of this tutorial. As a brief overview, `npsim` is `ddsim` under the hood which is itself, predominantly Geant4 under the hood. We will start with `ddsim` simply to demonstrate usage.

:::::::::::::::::::::::::::::::::::::::::::::

## Passing options to `ddsim`

The program `ddsim` is part of the DD4hep installation when it is compiled with Geant4 support. In the EIC standard environment `eic-shell` it is available and used for many simulations in the suite of continuous integration and benchmarking checks for the geometry. Simply entering `ddsim --help` will show the large wealth of options that can be passed to `ddsim` on the command line.

It is also possible to use a python steering file to control the simulations. This is often a more convenient approach for sharing settings with others or archiving them for later reference or to iteratively add functionality. However, it should be noted that this can also lead to divergence in 'default' running conditions when these are not propagated to other environments.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise: explore the `ddsim` options

- Consult the available options of `ddsim` by passing it the `--help` flag.
- Find the appropriate option to specify the output file when using `ddsim`.
- Use the `--dumpSteeringFile` flag to print out a default steering file, and redirect it into a file `steering.py`.

::::::::::::::: solution

The output file is set with `--outputFile`. A default steering file can be written with:

```bash
ddsim --dumpSteeringFile > steering.py
```

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

## Available options in `ddsim`

There are several option blocks that can be used with `ddsim`. Those are best looked at in the `steering.py` file, where additional documentation is added.

- `SIM.action.*` or `--action.*` options can be used to tune sensitive detector actions,
- `SIM.field.*` or `--field.*` options affect the magnetic field steppers,
- `SIM.filter.*` or `--filter.*` options can add filters to sensitive detectors,
- `SIM.gun.*` or `--gun.*` options can set single particle gun settings,
- `SIM.physics.*` or `--physics.*` options allow setting the physics list,
- `SIM.random.*` or `--random.*` options can be used to fix the random seed.

Some options, such as `SIM.physics.setupUserPhysics`, take as argument a python function, so they can only be used inside the python steering files. We will come back to this later.

When using the steering file approach, it is often useful to remove all options which you will not change (this allows you to take advantage of updates to the `ddsim` command itself without being stuck on old default settings). In this case, you would simply start from a steering file that only contains:

```python
from DDSim.DD4hepSimulation import DD4hepSimulation
from g4units import mm, GeV, MeV
SIM = DD4hepSimulation()
```

and which can be passed to `ddsim` with the `--steeringFile` flag:

```bash
ddsim --steeringFile steering.py
```

This steering file merely sets up the simulation object that can then be configured with settings that deviate from the default.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise: a minimal steering file

- Compose a 'minimal' steering file that only contains the required header line.
- Attempt to run this steering file and note what `ddsim` claims is missing (we will specify this on the command line next).

::::::::::::::: solution

A minimal steering file contains only the three lines shown above (the `DD4hepSimulation` import and
`SIM = DD4hepSimulation()`). Running it with `ddsim --steeringFile steering.py` reports that no
geometry compact file was given, and that the number of events and the source of the events (an
input file or the particle gun) are not specified.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

## Running a first single-particle simulation

When we used the minimal steering file, `ddsim` pointed out that we did not specify the 'geometry compact file', nor the number of events and source of those events. In this sections we'll specify the geometry and tell `ddsim` to use a particle gun.

The compact file is the entry point of our geometry, for which we must load the geometry environment first

```bash
source /opt/detector/epic-main/bin/thisepic.sh
ddsim --steeringFile steering.py --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml
```

Next, we will specify that we want `ddsim` to use the DD4hep particle gun, with `--enableGun` (or `-G`), and that we want 10 events, with `--numberOfEvents 10` (or `-N 10`):

```bash
ddsim --steeringFile steering.py --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml -G -N 10
```

::::::::::::::::::::::::::::::::::::::::::::: callout

If you are running `eic-shell` directly on cvmfs, this may take a little while the first time. All large Geant4 data files are accessed for the first time and need to be retrieved.

:::::::::::::::::::::::::::::::::::::::::::::

When we run `ddsim`, by default it prints out some information for each generated event:

```text
GenerationInit   INFO  +++ Initializing event 1. Within run:0 event 1.
Gun              INFO  Particle [0] mu-          Mom:10.000 GeV vertex:( 0.000  0.000  0.000)[mm] direction:( 0.000  0.000  1.000)
Gun              INFO  Shoot [0] 10.000 GeV mu- pos:(0.000 0.000 0.000)[mm] dir:( 0.000  0.000  1.000)
Gun              INFO  +-> Interaction [0] 10.000 GeV mu- pos:(0.000 0.000 0.000)[mm]
Gun              INFO  +++   +-> ID:  0 mu-          status:00000002 PDG:    13 Vtx:(+0.00e+00,+0.00e+00,+0.00e+00)[mm] time: +0.00e+00 [ns] #Dau:  0 #Par:0
PrimaryHandler   INFO  +++++ G4PrimaryVertex at (+0.00e+00,+0.00e+00,+0.00e+00) [mm] +0.00e+00 [ns]
ParticleHandler  INFO  +++ Event 0 Begin event action. Access event related information.
```

You will notice that the particle gun has reverted to a default particle in a default direction: a 10 GeV muon in the positive z direction. That is, of course, not going to result in many hits in our ePIC detector... We will now take a closer look at some of the particle gun options.

```text
  --gun.energy GUN.ENERGY
  --gun.particle GUN.PARTICLE
  --gun.multiplicity GUN.MULTIPLICITY
  --gun.phiMin GUN.PHIMIN
                        Minimal azimuthal angle for random distribution
  --gun.phiMax GUN.PHIMAX
  --gun.thetaMin GUN.THETAMIN
  --gun.thetaMax GUN.THETAMAX
  --gun.momentumMin GUN.MOMENTUMMIN
                        Minimal momentum when using distribution (default = 0.0)
  --gun.momentumMax GUN.MOMENTUMMAX
  --gun.direction GUN.DIRECTION
                         direction of the particle gun, 3 vector
  --gun.distribution {uniform,cos(theta),eta,pseudorapidity,ffbar}
                        choose the distribution of the random direction for theta

                            Options for random distributions:

                            'uniform' is the default distribution, flat in theta
                            'cos(theta)' is flat in cos(theta)
                            'eta', or 'pseudorapidity' is flat in pseudorapity
                            'ffbar' is distributed according to 1+cos^2(theta)

                            Setting a distribution will set isotrop = True

  --gun.isotrop GUN.ISOTROP
                         isotropic distribution for the particle gun

                            use the options phiMin, phiMax, thetaMin, and thetaMax to limit the range of randomly distributed directions
                            if one of these options is not None the random distribution will be set to True and cannot be turned off!

  --gun.position GUN.POSITION
                         position of the particle gun, 3 vector
```

While many of the options have straighforward names, others may be more confusing. The `gun.distribution` option is particularly relevant when we want to distribute single particle events over a range of angles. Depending on your needs, you may prefer one over the other, but in this tutorial we will simply use `cos(theta)` to throw uniformly on the unit sphere in the forward direction:

```bash
ddsim --steeringFile steering.py --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml -G -N 10 --gun.thetaMin "3*deg" --gun.thetaMax "45*deg" --gun.distribution "cos(theta)" --gun.momentumMin "1*GeV" --gun.momentumMax "10*GeV" --gun.particle "pi+"
```

::::::::::::::::::::::::::::::::::::::::::::: callout

Avoid the use of the `gun.energy` option, since it is inherently more ambiguous than `gun.momentum` for massive particles (in the context of EIC).

:::::::::::::::::::::::::::::::::::::::::::::

Note how we pass arguments with units in this example. The double quotes are in many cases necessary on the command line to avoid having the `*` be expanded by your shell. In the python steering file they can be ommitted and one can simply write, e.g., `SIM.gun.thetaMin = 3*deg`.

When running this simulation above, note the output on the command line

```text
GenerationInit   INFO  +++ Initializing event 1. Within run:0 event 1.
Gun              INFO  Particle [0] pi+          Mom:4.924 GeV vertex:( 0.000  0.000  0.000)[mm] direction:( 0.017  0.403  0.915)
Gun              INFO  Shoot [0] 10.000 GeV pi+ pos:(0.000 0.000 0.000)[mm] dir:( 0.000  0.000  1.000)
Gun              INFO  +-> Interaction [0] 10.000 GeV pi+ pos:(0.000 0.000 0.000)[mm]
Gun              INFO  +++   +-> ID:  0 pi+          status:00000002 PDG:   211 Vtx:(+0.00e+00,+0.00e+00,+0.00e+00)[mm] time: +0.00e+00 [ns] #Dau:  0 #Par:0
PrimaryHandler   INFO  +++++ G4PrimaryVertex at (+0.00e+00,+0.00e+00,+0.00e+00) [mm] +0.00e+00 [ns]
ParticleHandler  INFO  +++ Event 0 Begin event action. Access event related information.
```

For historical reasons (which are being addressed), the relevant line is the one with `Particle [0]`.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise: shoot into the negative endcap

- Simulate 10 events in the negative endcap, using an angle distribution that is uniform in `cos(theta)`, and with an electron multiplicity of 2.
- Add all gun options (but not the number of events) to the minimal steering file you constructed earlier and rename the steering file to `ee_1GeV_10GeV_EndcapN.py` and run the simulation again.
- Verify that the momentum ranges and angular ranges are correct in the output.

::::::::::::::: solution

The negative endcap corresponds to large polar angles, so set `--gun.thetaMin "135*deg"` and
`--gun.thetaMax "177*deg"` (or use a negative-`z` direction), with `--gun.particle "e-"`,
`--gun.multiplicity 2`, and `--gun.distribution "cos(theta)"`. Moving these into the steering file as
`SIM.gun.*` assignments and saving it as `ee_1GeV_10GeV_EndcapN.py` reproduces the same run. The
`Particle [0]` lines in the output should show electrons with momenta between 1 and 10 GeV pointing
into the negative endcap.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

## Output files

Until now we have not bothered to check the output files (in case you were wondering and went exploring, you may have noticed that output went into `dummyOutput.slcio`). In this section we'll explore how to write output files.

The command line option to use to specify the output file is the `--outputFile` option (or `SIM.outputFile` in the steering file). Depending on the extension of the output file, a specific output file format is chosen. The default output is in the slcio format, but we have standardized on the EDM4hep data model inside ROOT files. To choose this output file format, use a file extension `.edm4hep.root`. We could, for example, run the following command:

```bash
ddsim --steeringFile ee_1GeV_10GeV_EndcapN.py --compactFile $DETECTOR_PATH/$DETECTOR_CONFIG.xml --numberOfEvents 10 --outputFile ee_1GeV_10GeV_EndcapN_1e1.edm4hep.root
```

::::::::::::::::::::::::::::::::::::::::::::: callout

The extension `.edm4hep.root` of the output file is important since `ddsim` infers the output file type from the extension.

:::::::::::::::::::::::::::::::::::::::::::::

Let's take a look at the output file in ROOT (you can do this inside or outside the container, depending on your system and facility with opening ROOT browsers).

```bash
root -l ee_1GeV_10GeV_EndcapN_1e1.edm4hep.root
root [1] .ls
TFile**         ee_1GeV_10GeV_EndcapN_1e1.edm4hep.root  data file
 TFile*         ee_1GeV_10GeV_EndcapN_1e1.edm4hep.root  data file
  KEY: TTree    events;1        Events tree
  KEY: TTree    metadata;1      Metadata tree
  KEY: TTree    run_metadata;1  Run metadata tree
  KEY: TTree    evt_metadata;1  Event metadata tree
  KEY: TTree    col_metadata;1  Collection metadata tree
```

All EDM4hep files will have the same structure. We focus here on the `events` tree (if you see `EVENT`, in capital letters, you will need to ensure that you are indeed using the `.edm4hep.root` extension).

::::::::::::::::::::::::::::::::::::::::::::: callout

In a future version of `ddsim` the output file format will change slightly, and the various `metatadata` trees will not be there anymore.

:::::::::::::::::::::::::::::::::::::::::::::

### `MCParticles`

Let's first focus on the `MCParticles` branch. This contains 'truth' information and exact Geant4 step output for selected tracks. It is filled independent of sensitive detectors defined in the geometry. It is most useful to analyze the initial state of the simulation (i.e. the final state of the event generator). In our case, the number of generated particles from the particle gun is always 2, which is what we expect to recover by looking at the `@MCParticles.size()` in the `events` tree:

```c++
root [2] events->Draw("@MCParticles.size()")
```

Wait a minute. What happened here? The `MCParticles` branch contains more than just the generated events. To include only the generated particles that were 'thrown' into the detector, we can select those with `generatorStatus` equal to 1.

```c++
root [3] events->Draw("MCParticles.PDG","MCParticles.generatorStatus==1")
```

Another approach which is sometimes useful to reduce the number of 'other' particles that are written is to pass the option `--part.minimalKineticEnergy "1*TeV"`. This restricts the particles that are written to those that have an energy higher than 1 TeV. Initial state event generator particles are exempt from this requirement and are always written.

### `*Hits`

The second set of branches that are important, and that are used as input to the event reconstruction framework, are the `*Hits` branches. These branches include the hits in sensitive detectors: either tracker hits or calorimeter hits. You will notice that these two different types of hits contain slightly different information.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise: inspect the output file

- Open the file `ee_1GeV_10GeV_EndcapN_1e1.edm4hep.root` which you just created in a local ROOT installation or online at [the JSROOT web viewer](https://root.cern/js/latest).
- Plot the z component of the momentum for generated particles only and verify that this is indeed negative for particles going towards the negative endcap.
- Verify that the total number of entries is consistent with the multiplicity and number of events you have simulated.
- Plot the deposited energy of the hits in the endcap Ecal and compare the number of hits in the positive and negative endcaps.

::::::::::::::: solution

Plotting `MCParticles.momentum.z` with the selection `MCParticles.generatorStatus==1` shows negative
values, confirming the particles head into the negative endcap. With a multiplicity of 2 over 10
events you expect 20 generated entries. The endcap Ecal `*Hits` branches (e.g.
`EcalEndcapNHits.energy`) should show energy deposits concentrated in the negative endcap and few or
none in the positive endcap.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: keypoints

- `ddsim` can be used with a particle gun to generate single particle simulations.

:::::::::::::::::::::::::::::::::::::::::::::
