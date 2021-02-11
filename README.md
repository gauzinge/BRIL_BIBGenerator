# BRIL_BIBGenerator
>>IN PROGRESS<<
### Introduction
 This guide gives instructions on setting up and running Beam Induced Background (BIB, alternatively MIB for Machine Induced Background) simulations with a two step method (BIB particle generation + simulation).
Link to CMSSW offline workbook: 
https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBook

## Setup
Login to _lxplus_. <br>
`ssh -Y username@lxplus.cern.ch` <br>
Navigate to a directory where you want CMSSW to be located, then type <br>
`cmsrel CMSSW_11_2_0_pre6`
To find out about a list of currently available CMSSW releases:
`scram list -a`
Navigate to the source directory. <br>
`cd CMSSW_10_2_18/src`
To activate CMSSW working environment, paths and compiler (while being in the /src directory): <br>
`cmsenv`
In the /src directory you can clone the current repository that will be used for BIB generation: <br>
`git clone https://github.com/pkicsiny/BRIL_BIBGenerator.git` <br>
By default, CMSSW can only find and work with the code if it is located in the /src directory. Therefore the subdirectory BRIL_BIBGenerator/GeneratorInterface should be symlinked from the /src directory as shown below: <br>
`ln -s BRIL_BIBGenerator/GeneratorInterface GeneratorInterface` <br>

In addition, you will need to get the coclone another repository that contains the main config files, located at `BIBGeneration/python/`. <br>

The cofig file BH_generation.py launches the generation step by invoking code from `BRIL_BIBGenerator/GeneratorInterface/BeamHaloGenerator/python/MIB_generator_cff.py` which in turn invokes CMSSW through `BRIL_BIBGenerator/GeneratorInterface/BeamHaloGenerator/src/BeamHaloProducer.cc`. <br>
The main config file can be found here:
`https://github.com/pkicsiny/BRIL_ITsim/BIBGenerator/python` <br>
This directory contains 2 config files: 
`BH_generator.py` used for the generation step<br>
`BH_SimTrigRec.py` used for the simulation step <br>
The FLUKA simulated beam induced background files can be found at https://bbgen.web.cern.ch/HL-LHC/
that you can download and place on lxplus to a directory. It is recommended to use /eos file system because of the large size of these input files. <br>
Running the generator step is done by <br>
`cmsRun BH_generator.py` <br>
In this config file the following user parameters can be set: <br>
nEvents: number of events to take from the FLUKA input files.
nThreads: number of parallel computing threads to use.
jobId: relevant when running the generation step on lxbatch, where the simulation is split into smaller chunks each having a unique job ID.
outputDirectory: absolute path of the diractory to where the output file is created.
inputPath: absolute path specifying the location of the FLUKA input files.
Currently the line
`options.inputFiles= [inputPath + "/" + f for f in os.listdir(inputPath) if f[:3] == "run"]`
automatically parses the file names in the directory and selects whose file name begins with "run". This can be adapted to the specific usecase.
The output file name can also be changed under the `#specify output name` comment. <br>
This will produce a root file in the output folder. It contains the same particles as in the FLUKA dumnp input, but in a different, CMSSW friendly format, called HEPMC. <br>
Although the CMS geometry plays no role in the generation step, CMSSW always expects the input geometry to be specified. Currently the most up to date Phase 2 full CMS geometry is used, just as a 'placeholder': <br>
`process.load('Configuration.Geometry.GeometryExtended2026D63_cff')` <br>

The second step consists of the transport of paticles and the simulation of particle-matter interactions in the CMSSW geometry model. This step can be launched by executing:
`cmsRun BH_SimTrigRec.py` <br>
