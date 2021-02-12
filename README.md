# BRIL_BIBGenerator
>UNDER CONSTRUCTION
### Introduction
This guide gives instructions on how to set up and run Beam Induced Background (BIB, alternatively MIB for Machine Induced Background) simulations in CMSSW with a two step method (BIB particle generation + simulation).
For an official introduction and manual for CMSSW have a look at the offline workbook: 
https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBook

## Setup
Login to _lxplus_. <br>
```sh
ssh -Y username@lxplus.cern.ch
```
If you don't have any CMSSW release already, navigate to an empty directory with and then type <br>
```sh
cmsrel CMSSW_11_2_0_pre6
````
which creates a local copy of CMSSW version 11.2.pre6. To find out about a list of currently available CMSSW releases, type
```sh
scram list -a
```
or visit the official [CMSSW github](https://github.com/cms-sw/cmssw) page, where you can also browse the simulation source code. Now you can navigate to the source directory within the created CMSSW release <br>
```sh
cd CMSSW_11_2_0_pre6/src
```
and activate a CMSSW working environment, paths and compiler (while being in the /src directory) by typing <br>
```sh
cmsenv
```
This command has to be issued only once but every time you open up a new terminal an start to wotk with CMSSW. In the _/src_ directory you can now clone the current repository that will be used for BIB generation: <br>
```sh
git clone https://github.com/pkicsiny/BRIL_BIBGenerator.git
```
CMSSW can only find and work with the code if it is located in the /src directory. Therefore the subdirectory BRIL_BIBGenerator/GeneratorInterface should be symlinked from the _/src_ directory as shown below: <br>
```sh
ln -s BRIL_BIBGenerator/GeneratorInterface GeneratorInterface
```
In addition, you will need to get another repository that contains two config files which will be used to run the generation and simulation step respectively, located at `BIBGeneration/python/`. <br>
The cofig file BH_generation.py launches the generation step by invoking code from 
```sh
BRIL_BIBGenerator/GeneratorInterface/BeamHaloGenerator/python/MIB_generator_cff.py
``` which in turn invokes CMSSW through `BRIL_BIBGenerator/GeneratorInterface/BeamHaloGenerator/src/BeamHaloProducer.cc`. This config file can be found here:
```sh
https://github.com/pkicsiny/BRIL_ITsim/BIBGenerator/python/BH_generation.py
```
and is used for the generation step. The FLUKA simulated beam induced background files can be found at https://bbgen.web.cern.ch/HL-LHC/
that you can download and place on lxplus to a directory. It is recommended to use /eos file system because of the large size of these input files. <br>
Running the generator step is done by <br>
```sh
cmsRun BH_generator.py
```
In this config file there are some user parameters that you can set specific to the usecase: <br>
nEvents: number of events to take from the FLUKA input files. <br>
nThreads: number of parallel computing threads to use. <br>
jobId: relevant when running the generation step on lxbatch, where the simulation is split into smaller chunks each having a unique job ID. <br>
outputDirectory: absolute path of the diractory to where the output file is created. <br>
inputPath: absolute path specifying the location of the FLUKA input files. <br>
The line
```sh
options.inputFiles= [inputPath + "/" + f for f in os.listdir(inputPath) if f[:3] == "run"]
```
automatically parses the file names in the _inputPath_ directory and selects files whose name begins with "run". This parsing can also be adapted or removed depending on the specific usecase. The output file name can also be changed under the _#specify output name_ comment. Running the generator step will produce a root file in the output directory. It contains the same particles as in the FLUKA dumnp input, but in a different, CMSSW friendly format, called HEPMC. <br>
Although the CMS geometry plays no role in the generation step, CMSSW always expects the input geometry to be specified. Currently the most up to date Phase 2 full CMS geometry is used, just as a 'placeholder': <br>
```sh
process.load('Configuration.Geometry.GeometryExtended2026D63_cff')
```

The second step consists of the transport of paticles and the simulation of particle-matter interactions in the CMSSW geometry model. This step can be launched by executing:
`cmsRun BH_SimTrigRec.py` <br>
