# NEB basics
An algorithm to find the transition state between the initial and final images
- Reference: https://theory.cm.utexas.edu/henkelman/research/saddle/neb/
- Key papers: https://theory.cm.utexas.edu/henkelman/pubs/henkelman00_9978.pdf (https://aip.scitation.org/doi/10.1063/1.1323224)
- Climbing image NEB (CI-NEB) vs normal NEB calculations: CI-NEB can be activated using `LCLIMB = True` in `INCAR`. However, according to our experience, this algorithm is not as stable as regular NEB. You can first do a regular NEB (`LCLIMB = False` ). Then make a separate folder and copy all images such as 00, 01, ... to this folder, copy all `CONTCAR` to `POSCAR` within each image, and change `LCLIMB = True` and recalculate.
# Procedure
1. Create structures for  initial and final state -> by hand or using Pymatgen 
2. Put the migrating specie (ion) to the top of the coordinate list in `POSCAR/CONTCAR`, also make sure that the coordinates of the initial and final images are matching with each other in sequence 
3. Fully relax both initial and final images
4. Take the relaxed initial and final images, then interpolate the trajectory (`interpola_3.0.py`). VASP input settings are written manually in `interpola_3.0.py`, you should have a look and change settings accordingly. This will generate 00, 01, 02 ... N_image-1. `00` and `N_image-1` are the initial and final images, respectively, and they are not involved in NEB calculations.
5. Before doing NEB calculations, check the trajectory using `mergepath.py` , which will generate  a structure file (`NEBPath.vasp`).
6. Then run NEB using the NEB version of `VASP` (ref: https://theory.cm.utexas.edu/vtsttools/), ask anybody in the group for this binary. 
7. After NEB calculations, use `analyze_python3.py` to get the energy profile (NEB plot: `barrier.pdf` and raw data: `data.dat`, `data.csv`). The trajectory can be visualized using `mergepath.py`

# Some Useful Scripts
1. `interpola_3.0.py` from Piero: this can generate interpolated images (CONTCARs) after initial and final structures are relaxed. It should be run in your NEB folder where 00, 01, 02... exist.
 ```bash
 python interpola_3.0.py -i 4 -e 1
 ```
 `-i` means the number of images to be interpolated, `-e` specifies the number of electrons to be added to the structure (see Tips #2)
3. `mergepath.py` from Piero: it can generate a `CIF` file to visualize your migration path. It should be run in your NEB folder where 00, 01, 02... exist.
```bash
 python mergepath.py -e Na
```
`-e` can be used to specifying the migrating species to be visualized in a structure file (`NEBPath.vasp`). This code can be used to check if your migration path makes sense before NEB calculations, as well as generating the final trajectory after NEB calculations.
1. `analyze_python3.py` from Piero: this will read `OSZICAR` and `OUTCAR` to get an energy profile along the migration path. It should be noted that energies of images are normalized to make sure the summation of the energies of initial and final images are zero.
```bash
python analyze_python3.py
```

# Tips
1. You might need supercell if your unit cell is too small (lattice parameters are less than 8-10 Å) to get rid of image interactions
2. When creating initial and final images, you might need to create vacancies. Then in order to make the cell charge balanced, you need to add additional electrons by specifying the total electron using `NELECT` in `INCAR`
3. Usually initial and final images are the nearest neighboring migrating species. If the distances are large (~ 5 Å), it's better to do a bond valance calculation (SoftBV: https://www.dmse.nus.edu.sg/asn/softBV-GUI.html or ref: https://pubs.acs.org/doi/10.1021/acs.chemmater.0c03893) to estimate the diffusion path.
4. NEB can only be run on large machines such as NSCC because the resource consumption of each image is equivalent to one relaxation, so the total resource consumption will be (N_image-2) * Resource_single_relax (initial and final are excluded)
5. `interpola_3.0.py`, `analyze_python3.py` and `mergepath.py` have built-in documentations. You can use the following command to call out help:
```bash
python interpola_3.0.py -h
```
6. NEB calculations might be hard to converge than regular relaxations, make sure your initial migration path physically make sense.