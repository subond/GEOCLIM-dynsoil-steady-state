# GEOCLIM-dynsoil-steady-state

GEOCLIM - DynSoil - Steady-State computes geographically-distributed chemical weathering rates (along with associated parameters) at steady-state, according to the given climatology (temperature and runoff) for several CO<sub>2</sub> levels (2 at least!), the topographic slope and the lithology fraction in each grid cell. The climatology is typically taken from the output of a general circulation climate model. The total weathering flux is assumed to equal the CO<sub>2</sub> degassing by the solid Earth.

The DynSoil component of the model was developed by Pierre Maffre during his PhD research with Yves Godderis and is detailed in his thesis: Interactions entre tectonique, érosion, altération des roches silicatées et climat à l'échelle des temps géologiques: rôle des chaînes de montagnes. Océan, Atmosphère. Université Toulouse III Paul Sabatier, 2018. Français. https://tel.archives-ouvertes.fr/tel-02059359
The overall framework of integrating a silicate weathering model with varying climatology to model ancient pCO<sub>2</sub> levels is that of the GEOCLIM suite of models developed by Yves Godderis: https://geoclimmodel.wordpress.com

The model has two modes:
	1) Forward mode: impose a CO<sub>2</sub> level to the model, it gets the corresponding climate field and computes weathering
	2) Backward mode: impose a volcanic degassing to the model, it finds by inversion the CO<sub>2</sub> level for which the total CO<sub>2</sub> consumption equals the degassing. The precision level can be set in the main program "gdss_mainprog.f90".

The continental climate (i.e. temperature and runoff) is interpolated linearly between the different CO<sub>2</sub> levels. At this time, the algorithm used for the interpolation does not allow extrapolation beyond the lowest and highest level.
The climate interpolation is done by the subroutines of the module "climate_module.f90". DynSoil integration is done by subroutines of the module "dynsoil_steady_state_module.f90". The inversion (in backward mode) is done in the main program.

In addition, the model can be run either in single run mode, for which it computes weathering one time for one given set of parameters, or in multiple runs mode, for which an netCDF unlimited dimension is created, and the model is run as many times as the number of a given set of parameters.

## Repository architecture
The input/output information is stated in the file "IO_INTERFACE.txt", as well as the flags for model mode (Forward, Backward, Single run, Multirun). This file is in the root directory.
It includes the names of input files to use of continent area, lithology, temperature, runoff, slope. The model checks the consistency of this input dataset (size, coordinates, units) and DynSoil is run accordingly using these geographic-climatic-lithologic settings.
These input files should be in netCDF format. The names of the dimensions for the input variables in these netCDF files also need to be stated in "IO_INTERFACE.txt" in the INPUT CONDITION section.

The executable files are meant to be run in the directory "executables/". For this reason, the program looks for the relative path "../IO_INTERFACE.txt", as specified line 11 of the main program "gdss_mainprog.f90". Hence, the code will crash if it is not run directory one level after the root directory. All other paths can be changed in the file "IO_INTERFACE.txt" without recompiling the code.

The physical parameters of DynSoil and the forcings (CO<sub>2</sub> or degassing) are read in two separated text files whose names are stated in "IO_INTERFACE.txt". See the documentation in "IO_INTERFACE.txt" for the format of these files.

Conventionally, the parameter file is in the folder "parameters/", the forcings file is in the folder "forcings/" and the outputs are stored in the folder "outputs/". Templates can be found in those folders. Note that the reading format of the parameter file is not the same in the single run or the multirun mode.

"IO_INTERFACE.txt" is read by the subroutine 'make_input_output' in the file "io_module.f90", it creates the netCDF output file (conventionnaly in the "output/" directory) plus a fortran scratch text file with the ID of the variables to record (unit=IOUT), and potentially two others scratch files (unit=IFORC and unit=IPARAM) recording the forcings and the parameters to be used in the main program.

## Compilance checks
The program conducts a certain number of compliance checks in the subroutine "make_input_output" (file "io_module.f90"), that includes checking the forcing variables units (area, temperature, runoff, slope...), the consistency of the geographic axes of all forcing variables and the consistency of their continental mask. For this last check only, to program is not automatically aborted in case of failure, but the user is asked for continuing to run it (0 to abort, 1 to continue). This argument can be enter interactively, or the user can enter it as an argument of the executable:
./executable 1   => run the program whatever the case
./executable 0   => abort in case of continental mask inconsistency

## Compilation tips:
This program uses syntax allowed in fortran 2003 or later (subroutines with allocatable dummy arguments, and the use of the subroutine "get_command_argument"). Make sure your compiler support this syntax (with gfortran: "-std=f2003").

## Parameter exploration:
This repository also contains a version of the model where the user can vary the parameters for a unique given CO<sub>2</sub> level. It can be found in the folder "preprocessing/parameter_exploration/". See the README in this folder for more information.
