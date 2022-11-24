1. Create prim.json file -> this file contains structural information about the primitive cell (we usually use exp cell) and initialize project:
```bash
casm init
```
Data structure: 
```
Basis:
	Coordinate -> coordiante for each site
	occupant_dof -> [Na,Va]  "Va" for vacancy
	…
	Coordinate_mode -> cartesian or fractional
Description
Lattice_vectors
Title
```

```json
	{
	  "basis" : [
	    {
	        "coordinate" : [ 0.500000, 0.500000, 0.500000],
	        "occupant_dof" : ["Na","Va"]
	        },
	        {
	        "coordinate" : [ 0.000000, 0.000000, 0.000000],
	        "occupant_dof" : ["Na","Va"]
	        },
	        {
	        "coordinate" : [ 0.889670, 0.610330, 0.250000],
	        "occupant_dof" : ["Na","Va"]
	        },
	        {
	        "coordinate" : [ 0.610330, 0.250000, 0.889670],
	        "occupant_dof" : ["Na","Va"]
	        },
	        {
	        "coordinate" : [ 0.250000, 0.889670, 0.610330],
	        "occupant_dof" : ["Na","Va"]
	        },
	        {
	        "coordinate" : [ 0.389670, 0.750000, 0.110330],
	        "occupant_dof" : ["Na","Va"]
	        },
	        {
	        "coordinate" : [ 0.750000, 0.110330, 0.389670],
	        "occupant_dof" : ["Na","Va"]
	        },
	        {
	        "coordinate" : [ 0.110330, 0.389670, 0.750000],
	        "occupant_dof" : ["Na","Va"]
	        },
	        {
	        "coordinate" : [ 0.352810, 0.352810, 0.352810],
	        "occupant_dof" : ["Zr"]
	        }
	  ],
	  "coordinate_mode" : "Fractional",
	  "description" : "Si-based NASICON ",
	  "lattice_vectors" : [
	    [4.593150 ,2.651856  , 7.393667],
	    [-4.593150, 2.651856 , 7.393667],
	    [-0.000000, -5.303713, 7.393667]
	  ],
	  "title" : "NASICON_prim"
	}
```

2. Create composition axes
This is to define the composition that used for phase diagram
2 coupled axes are used for 2D cases, see "useful emails" for reason why we need this 2 coupled axes
.casm/composition_axes.json
```json
{
	  "current_axes" : "coupled",
	  "custom_axes" : {
	    "coupled" : {
	      "a" : [
	        [ 2.000000000000 ],
	        [ 6.000000000000 ],
	        [ 4.000000000000 ],
	        [ 0.000000000000 ],
	        [ 6.000000000000 ],
	        [ 24.000000000000 ]
	      ],
	      "b" : [
	        [ 2.000000000000 ],
	        [ 6.000000000000 ],
	        [ 4.000000000000 ],
	        [ 0.000000000000 ],
	        [ -6.000000000000 ],
	        [ 24.000000000000 ]
	      ],
	      "components" : [ "Na", "Va", "Zr", "Si", "P", "O" ],
	      "independent_compositions" : 2,
	      "origin" : [
	        [ 8.000000000000 ],
	        [ 0.000000000000 ],
	        [ 4.000000000000 ],
	        [ 6.000000000000 ],
	        [ 0.000000000000 ],
	        [ 24.000000000000 ]
	      ]
	    }
	  }
	}
```

Composition = Origin + (End-Origin)x      (End = a or b here)
Then compute composition axes
```bash
casm composition -c
```
3. Import calculated DFT results
Use "vasp.relax.report" to generate properties.calc.json in each directories
Generate a file list containing all the path to POSCAR "reports_path_primitive.txt"
Import results into .casm/config_list.json <- if you want to update new results, make sure you exclude old paths in "reports_path_primitive.txt" otherwise it will have duplications in database
```bash
casm import --batch reports_path.txt --ideal --data --min-energy
```

4. Choose chemical reference (per species = per atom)
```
	'[
	        {"Na": 8.0, "Zr": 4.0, "Si": 6.0, "P": 0.0, "O": 24.0, "energy_per_species": -7.39616323214285714285},
	        {"Na": 2.0, "Zr": 4.0, "Si": 0.0, "P": 6.0, "O": 24.0, "energy_per_species": -7.93335068250000000000},
	        {"Na": 8.0, "Zr": 0.0, "Si": 6.0, "P": 0.0, "O": 24.0, "energy_per_species":  0.00000000000000000000}
	]'
```

Pass the piece above directly to the command!!
```bash
casm ref --set '[{"Na": 8.0, "Zr": 4.0, "Si": 6.0, "P": 0.0, "O": 24.0, "energy_per_species": -7.39616323214285714285}, {"Na": 2.0, "Zr": 4.0, "Si": 0.0, "P": 6.0, "O": 24.0, "energy_per_species": -7.93335068250000000000}, {"Na": 1.0, "energy_per_species": -1.308547}, {"Zr": 1.0, "energy_per_species": -8.547687}]'
```

The chemical reference can be updated later
```bash
casm update
```
Here I used the lowest energy structure should be used
```bash
casm ref --set '[{"Na": 8.0, "Zr": 4.0, "Si": 6.0, "P": 0.0, "O": 24.0, "energy_per_species": -11.732566428571428}, {"Na": 2.0, "Zr": 4.0, "Si": 0.0, "P": 6.0, "O": 24.0, "energy_per_species": -12.525085}, {"Na": 1.0, "energy_per_species": -4.2040927}, {"Zr": 1.0, "energy_per_species": -30.6929575}]'
```

5. Create basis function (it's better to use chebychev basis function)
basis_sets/bset.default/bspecs.json
Occupation can be changed to other properties like spin etc..
Orbit_branch_specs: set the size of cluster for generating basis function, usually decrease with the increment of order
```json
	{
	    "basis_functions" : {
	      "site_basis_functions" : "occupation"
	    },
	    "orbit_branch_specs" : {
	      "2" : {"max_length" : 10.0000},
	      "3" : {"max_length" : 6.00000},
	      "4" : {"max_length" : 5.00000}
	    }
	}
```

Then compile to get basis function -> it might take 30 mins!
```bash
casm bset -u
```
6. Prepare fitting ECI
Create a folder e.g. fit_1
Select candidates for fitting and save to "train"
```bash
casm select --set is_calculated -o train
```
	Create casm-learn input file fit.json using lasso algorithm
	Candidate list file: "filename" (train here) should exist in this folder
```json
	{
	  "estimator": {
	    "method": "Lasso",
	    "kwargs": {
	      "alpha": 0.0001,
	      "max_iter": 1000000.0
	    }
	  },
	  "feature_selection": {
	    "method": "SelectFromModel",
	    "kwargs": null
	  },
	  "problem_specs": {
	    "data": {
	      "y": "formation_energy",
	      "X": "corr",
	      "kwargs": null,
	      "type": "selection",
	      "filename": "train"
	    },
	    "cv": {
	      "penalty": 0.0,
	      "method": "LeaveOneOut"
	    }
	  },
	  "n_halloffame": 25
	}
```

7. Fit ECI
```bash
casm-learn -s fit.json
```
Problem specs file will be generated "fit_specs.pkl" storing the training data, weights, and cross-validation train/test sets and "fit_halloffame.pkl" storing the selected candidates
Then adjust fit.json and repeat fitting until the fitting is satisfied (use least feature to reproduce most results). See "casm-learn --settings-format"
```bash
casm-learn --settings-format
```
Generation: eci.json and use it for monte carlo
```bash
casm-learn -s fit.json  --select 0
```
8. Plot convex hull
Query energies from database:
```bash
casm query -k  'comp(a)' 'formation_energy'    'clex(formation_energy)' 'hull_dist(ALL,atom_frac)'  'clex_hull_dist(ALL,atom_frac)' -c train  -o data.dat
```
Query hull from database
```bash
casm query  -k  'comp(a)' 'formation_energy' 'clex(formation_energy)'  'on_hull(ALL,comp)' 'on_clex_hull(ALL,comp)' 'comp_n(Na)' -c train   -o hull.dat
```
9. You'll see that the difference between cluster expansion (clex) convex hull is far away from DFT convex hull. To fix this , firstly, fix the correlation (cluster expansion coefficient, see useful emails Point term) and fit the weight. Then use:
```bash
filename=$1
./clean.sh
rm ${filename%.*}_*
casm-learn -s $filename
casm-learn -s $filename --checkhull
casm-learn -s $filename --select 0
casm-learn -s $filename --hall --indiv 0 --format json > ${filename%.*}-eci.json
#casm query -k  'comp(a)' 'formation_energy'    'clex(formation_energy)' 'hull_dist(ALL,atom_frac)'  'clex_hull_dist(ALL,atom_frac)' -c ALL  -o data.dat
#casm query  -k  'comp(a)' 'formation_energy' 'clex(formation_energy)'  'on_hull(ALL,comp)' 'on_clex_hull(ALL,comp)' 'comp_n(Na)' -c ALL  -o hull.dat
casm query -k  'comp(a)' 'formation_energy'    'clex(formation_energy)' 'hull_dist(ALL,atom_frac)'  'clex_hull_dist(ALL,atom_frac)' -c train   -o data.dat
casm query  -k  'comp(a)' 'formation_energy' 'clex(formation_energy)'  'on_hull(ALL,comp)' 'on_clex_hull(ALL,comp)' 'comp_n(Na)' -c train   -o hull.dat
python plot_convex_refactor.py
mv Convex_hull.pdf ${filename%.*}.pdf
mv hull.dat ${filename%.*}_hull.dat
mv data.dat ${filename%.*}_fit.dat
echo ${filename%.*}
open ${filename%.*}.pdf
```

10. To do fitting. Tuning the weight of train file until the error (CV) become small. In addition, the ECI should follow the general trend: pair is dominant then triplet, quadruplet etc.
First of all, use following command to query corr to train_weight.dat
```bash
casm query -k "formation_energy corr" -c train -o casm_learn_input
```
Next, add a column called "weight" and put all the point term
Then, using following fit.json to do fitting
```json
    {
     "estimator": {
     "method": "Lasso",
     "kwargs": {
     "alpha": 0.0001,
     "max_iter": 1000000.0
     }
     },
     "feature_selection": {
     "method": "SelectFromModel",
     "kwargs": null
     },
     "problem_specs": {
     "data": {
     "y": "formation_energy",
     "X": "corr",
     "kwargs": null,
     "type": "selection",
     "filename": "train"
     },
     "cv": {
     "penalty": 0.0,
     "method": "LeaveOneOut"
        },
        "weight":{
            "method":"wCustom"
        }
     },
     "n_halloffame": 25
}
```

