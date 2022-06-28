# CCS-RTM-GBDT-BO
Carbon Capture Storage (CCS) relevant Reactive Transport Modelling (RTM) of microfractures in basaltic rock; emulated using Gradient Boosted Decision Trees (GBDT) and subsequently optimised using Bayesian Optimisation (BO) framework.

# Overview

In 2022, an MRes project at the University of Cambridge on the subject of optimisation of CCS was initiated; this focussed upon the mineralisation of carbon that takes place when CO2 saturated fluid is pumped into fractured basaltic rock. The aim of this was to potentially find out what the best aqueous species compositions would be for injected carbonated fluids that are pumped down into such fractured basaltic fields of rock. The objectives therefore, were to firstly generate an RTM which mimicked the real-life geochemistry experienced in such basalts, secondly to obtain a dataset that might be representative of the behaviour of such a geochemical environment as you alter the composition of fluids it is exposed to, thirdly to train a lightweight model to replicate the behaviours of the underlying RTM allowing faster querying of the parameter space with respect to carbonates generated as a function of fluid composition, and finally to use a bayesian optimisation framework to carry out a structured analysis and optimisation of the geochemical system modelled to generate the highest amounts of carbonate possible by manipulating the fluid composition.

This README is intended to give an overview of the project and its development, as well as provide further information on the code lodged in this repository. The GitHub repository itself has the purpose of providing open and transparent access to all the methods applied during the course of this project. Unfortunately, the datasets generated during the course of this project are not available online due to their shear size; but if required email

# Timeline: Method Development

From the outset it was noted that one of the key geochemical environments in which carbon mineralisation takes place are dead-end microfractures, where diffusion is the dominant transport mechanism for movement of aqueous species. It was therefore decided that a basaltic rock fracture would be modelled. To model this, a reactive transport modelling software by the name of [Crunchtope](https://bitbucket.org/crunchflow/crunchtope-dev/wiki/Home) was used to simulate the behaviour of geochemical conditions in time and space.

Initially, evolution of the chemistry in such a rock was confined to a **basaltic cube model (bcm)** which didn't involve any transport processes and was effectively carried out in a single block of the 3D space thatCrunchTope can work within; this made it a 0D model.

This model progressed from a simple model with the correct volume fractions of the major mineral groups (bcm_1 sub-series), to a model detailing all the primary minerals themselves and their respective rates (bcm_2 sub-series), to a variety of models containing unserpentinised/serpentinised basalts (bcm_3 sub-series) (A great deal of referencing to the user manual was required to get as far as I did find it [here](https://netl.doe.gov/sites/default/files/netl-file/CrunchFlow-Manual.pdf)). The wider bcm test series culminated in the bcm_4 sub-series which incorporated primary/secondary species, all the necessary minerals and their thermodynamic/kinetic rates, a porosity within the cube to emulate a fractured space through which fluid could percolate, and the specific surface areas expected of the individual minerals. Many of the parameterisations cams from two key papers, one by [Xiong et al. 2017](https://www.sciencedirect.com/science/article/pii/S1750583617305595?via%3Dihub) and the other by [Liu et al. 2019](https://www.sciencedirect.com/science/article/pii/S1750583618309009?via%3Dihub). And study of these papers later led to the development of the bfm.

The **basaltic flow model (bfm)** model series, despite the slightly misleading titling, that might be more suggestive of advection as opposed to diffusion, was aimed at modelling a basaltic fracture in 1D as opposed to the simple 0D model developed previously.

This model series progressed by initially focussing on the implementation of diffusion across a number of discretised blocks (bfm_1 sub-series), subsequently  focus was placed on calibrating the model to behave like laboratory experiments carried out Xiong et al. 2017 (bfm_3 sub-series). This calibration ultimately failed to be achieved for serpentinised basalts, but some success was obtained in work on an unserpentinised flood basalt- this spawned the next sub-series (bfm_4) which studied the flood basalt in further detail and also looked at a second set of temperatures more representative of the lab experiment; database values had to be edited here to be in-line with the T and P worked with. Once the databases had been set up to handle 100oC temperatures and 100bar pressures the bfm_4 sub-series generated the final model of the fracture bfm_4-3 which, with some tweaking of SSAs used, replicated trends in experimental data. At this point a couple of unsuccessful side projects are ended (e.g. the attempt to introduce magnetite in 5-0), and the final model sub-series engaged. bfm_5 was an amalgamation of runs carried out on Omphalos, and despite their varied names are effectively the same underlying .in file. After a number of trial and error runs (bfm5-1 to 5-4) the final bfm_5-5 is generated and ran 32,000 times with its fluid composition altered in a representative manner across these tests to generate a datasets that might be representative of the underlting behaviours of the RTM. This task was carried out by 300CPU's and facilitated by the use of a python-based wrapper named [Omphalos](https://github.com/a-fotherby/Omphalos), which can generate multiple thousands of .in files and distribute them to multiple CPU's for runs to be carried out in an efficient manner; it then handily gathers data together, collates it, and generates a python readable .pkl file. It was with this data that a surrogate model could be trained to ape the underlying behaviour of the RTM, and therewith pave the way to a faster optimisation route.

The sizeable dataset generated by Omphalos was used to train a number of [XGBoost](https://xgboost.readthedocs.io/en/stable/) GBDT models to predict target values (carbonates generated within the fracture after a set period of time) from predictors (fluid species composition). Four surrogate models were generated, each subsequent model having been trained for an order of magnitude greater 'time' (n.b. epochs).

Each of the surrogate models were then treated as black-box functions that needed evaluating and a global optimum finding; a bayesian optimisation framework was used for this purpose. In this case, a pre-built package developed on GitHub entitled ['BayesianOptimization'](https://github.com/fmfn/BayesianOptimization) was implemented in the local python environment to carry out this optimisation procedure.

# Notes
File types that may be encountered when carrying out future work on this project:
* **.in** files parameterise runs carried out by CrunchTope.
* **.out** files are output files generated by CrunchTope after a run.
* **.tec** files are another type of output file generated by CrunchTope, can be read like a csv with a little googling.
* **.dbs** files specify all the kinetic and thermodynamic properties of minerals, species, gases etc for your CrunchTope run; .in files must reference .dbs files when they are read by CrunchTope.
* **.ant** files are catchily entitled 'PestControl' and are placed in folder next to RTM runs for user-friendliness purposes. When one of these files exists in a directory and has the correct .in file name written into it in plain text you can fire up CrunchTope and run the referenced .in file from the command line if you are in the same directory. Note you will still need to have set up a CrunchTope alias in your shell.
* **.ipynb** files are jupyter notebook files for mixed workflow between python and markdown. These files visualise almost all of the work done within this project.
* **.yaml** files have been used for the purpose of instructing the Omphalos CrunchTope wrapper of how to carry out large-scale distributed RTM run batches.
* **.pkl** files have been used for the purpose of containing large datasets of RTM runs and their outputs in a python friendly manner; due to their multi-GB size when they come out the other end of an Omphalos workflow, they have not been submitted to this GitHub page for fear of the dreaded size limit message... If you are planning on repeating the experiments carried out herein and therefore need the underlying dataset (i.e. .pkl files) write to my university email listed at the bottom of this README and I will try my utmost to get the relevant data to you.
* **.txt** files are plain text notes on what the individual models were geared towards at the time of development; things have been busy so please accept my apologies in advance for scarcity of note taking at times in these!
* **.json** files have been used for the purpose of storing information during bayesian optimisation processes, they contain target and predictor values. Another storage fuction for this file type has been the stowing of XGB models themselves for later booting up and querying.
* **.yml** a yaml file that has the purpose of providing an automatic way of setting up a conda environment for use with all the packages required in this project.

Systems that might be required during work on this subject area:
* [Visual Studio Code](https://code.visualstudio.com/) -  Not necessarily required, but works well alongside the syntax highlighters I developed for working effectively with the [.in](https://github.com/ThomasDodd97/CF-IF-LS) and [.out](https://github.com/ThomasDodd97/CF-OF-LS) and [.dbs](https://github.com/ThomasDodd97/CF-DF-LS) files that CrunchTope requires the manual editing of.
* [Jupyter Notebooks](https://jupyter.org/) - Programming environment that allows supports python, julia, R, and markdown simultaneously.
* [CrunchTope/CrunchFlow](https://bitbucket.org/crunchflow/crunchtope-dev/wiki/Home) - RTM software for development of geochemical environments
* [PETSc](https://petsc.org/release/) - Portable Extensible Toolkit for Scientific Computation; partial differential equation solving utility that is required for CrunchTope to function correctly.
* [Omphalos](https://github.com/a-fotherby/Omphalos) - A python based wrapper for CrunchTope facilitating the generation of thousands of .in files and subsequent distributed running of the associated RTM runs.
* [Weights and Biases](https://wandb.ai/site) - A webapp that can interlink with jupyter notebooks to keep track of machine learning training sessions.
* [XGBoost](https://xgboost.readthedocs.io/en/stable/) - A library that can be used from a python based environment to carryout GBDT upon datasets to train models of data, can be deployed from a jupyter notebook environment.
* [BayesianOptimization](https://github.com/fmfn/BayesianOptimization) - A github based project that implements bayesian optimisation in a python based environment, can be sued within jupyter notebook to optimise black-box functions such as those generated by XGBoost!
* [Conda](https://docs.conda.io/en/latest/) - package manager for keeping track of multiple development environments within jupyter notebooks etc.

Websites of notable utility during the course of this project:
* Pennsylvania State University Website - A damned useful [course](https://www.e-education.psu.edu/png550/node/829) giving the basics on how to use CrunchTope to run RTM simulations.
* Thermoddem Website - A useful [resource](https://thermoddem.brgm.fr/) for lookup of all minerals and their associated thermochemical and mineralogical properties.
* SUPCRTBL Website - An [webapp](https://models.earth.indiana.edu/supcrtbl.php) capable of generating thermodynamic property tables for most mineral dissolution reaction you might have in mind.

# Contributors
Project Lead - Thomas Højlund-Dodd - MRes/PhD Student, The University of Cambridge
* tjh98@cam.ac.uk
* https://github.com/ThomasDodd97

Project Supervisor - Alexandra Turchyn - Professor of Biogeochemistry, The University of Cambridge
* atur07@esc.cam.ac.uk

Project Co-Supervisor - Angus Fotherby - PhD Student, The University of CambridgeUniversity
* af606@cam.ac.uk
* https://github.com/a-fotherby