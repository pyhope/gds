# Gibbs dividing surface analysis

## Code Dependencies
```
numpy
matplotlib
lmfit
pytim
MDAnalysis
```
## Important Note: After installing `pytim`, find the path to its source code and replace the file `gaussian_kde_pbc.py` with `./similarity/gaussian_kde_pbc.py` in this repository.

## Code Usage
1. Extract one frame from the MD trajectory for the subsequent calculation. e.g.:
```sh
python extract_frame.py -i npt.dump -t 830000
```
Usage:
```
extract_frame.py [-h] [-i INPUT_FILENAME] [-o OUTPUT_FILENAME] [-t TIMESTEP]

optional arguments:
  -h, --help            show this help message and exit
  --input_filename INPUT_FILENAME, -i INPUT_FILENAME
                        Input filename (default: npt.dump)
  --output_filename OUTPUT_FILENAME, -o OUTPUT_FILENAME
                        Output filename (default: selected.dump)
  --timestep TIMESTEP, -t TIMESTEP
                        Timestep of the target frame. The last frame will be used if not provided.
```

2. Convert the format from LAMMPS dump file to extended XYZ
```sh
python similarity/dump2xyz.py Mg O Fe He
```
Usage:
```
dump2xyz.py [-h] [--dump_file DUMP_FILE] [--output_file OUTPUT_FILE] elements [elements ...]

Convert LAMMPS dump to extended XYZ with specified elements.

positional arguments:
  elements              List of element symbols in the order of their types in LAMMPS dump

optional arguments:
  -h, --help            show this help message and exit
  --dump_file DUMP_FILE
                        Path to the LAMMPS dump file (default: selected.dump)
  --output_file OUTPUT_FILE
                        Path for the output XYZ file (default: merge.xyz)
```

3. Run the Gibbs dividing surface analysis.
```sh
python similarity/stat.py -nw 2 -m mass
```
The theory can be found in Section 2.4 of Jie's recent preprint (https://doi.org/10.22541/essoar.171412549.91013860/v1). The only parameter that we need to adjust is `-nw`, which determines the scale of the assumed interface region. For example, if `nw` is 2, the metal phase is defined by $a_i > 2w$ and the oxide phase is defined by $a_i < 2w$, where $a_i$ is the proximity of ith atom to the interface, and $w$ is the thickness of the interface fitted by the model.

usage: 
```
stat.py [-h] [--begin BEGIN] [--end END] [--project_axis PROJECT_AXIS] [--step STEP] [--file FILE] [--show] [--num_interface_w NUM_INTERFACE_W] [--mode MODE]

optional arguments:
  -h, --help            show this help message and exit
  --begin BEGIN, -b BEGIN
                        begining index, default: 0
  --end END, -e END     end index, default is end of file
  --project_axis PROJECT_AXIS, -p PROJECT_AXIS
                        default 2(z), 0,1,2 => x,y,z
  --step STEP, -s STEP  step
  --file FILE, -f FILE  path to xyz file to analyze, default is merge.xyz in the cwd
  --show, -sh           Default: show results
  --num_interface_w NUM_INTERFACE_W, -nw NUM_INTERFACE_W
                        Default: must set!
  --mode MODE, -m MODE  Default: mass mode, k or mass, must set!
```
A text file named `sum_proximity_0_0.txt` will be generated. It records the counts of atoms in each phase as well as the interface. The example is
```
#  nw = 2 
#  solid liquid interface 
#  id O           Mg           Fe           W          lw      chi
0 12 85 159    16 87 153    3571 0 525    11 0 5        5.8344    0.1540
```
Here the three numbers below each element represent the counts of atoms in phase 1, phase 2, and interface, respectively. Note that the order `solid liquid interface ` shown in the second line may not be correct. You can check the numbers below `Fe` to identify the correct order. `lw` equals $nw \times w$, and the 3rd number for each element shows the number of atoms in an interface region with thickness `lw`. High `chi` means better fitting during the analysis (better Gibbs dividing surface).