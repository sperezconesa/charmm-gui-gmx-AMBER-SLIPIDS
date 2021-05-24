# Parametrizing CHARMM-GUI files for GROMACS with AMBER14SB+SLIPIDS16

If you use [ GROMACS ](http://www.gromacs.org/) and [CHARMM-GUI](https://charmm-gui.org/)
you might have been in the situation
of not being able to use AMBER force fields without having to do some manual reparametrization.
This repo is here to help you in case you want to combine [ AMBER14SB ](https://pubs.acs.org/doi/abs/10.1021/acs.jctc.5b00255) and [SLIPIDS16](https://pubs.acs.org/doi/abs/10.1021/acs.jpcb.6b05422).
These files and scripts are made so you can easily prepare your system in CHARMM-GUI
and then parametrize your system with AMBER14SB and SLIPIDS16 force fields
and run your molecular dynamics simulation with GROMACS. Please note that
not all lipids are supported by SLIPIDS16.

These scripts and files are not guaranteed to be bug-free (nothing is)
and you are responsible to check things. In any case, I hope this helps
you! Please contact sergio.perez.conesa at scilifelab.se or make an [issue](https://github.com/sperezconesa/charmm-gui-gmx-AMBER-SLIPIDS/issues) if
something doesn't work or you want to contribute.

You can find an example parametrization of KcsA starting from a `step5_input.pdb`
and `topol.top` obtained from CHARMM-GUI. The steps taken to process the files
are numbered in files.

## "Installing" the force field files

If you want `gmx pdb2gmx` to find the force field files even if they are not in
the current path you must use the following:
```bash
export GMXLIB=/insert/path/to/charmm-gui-gmx-AMBER-SLIPIDS/
```

## Amber/Slipids parametrization

1. Obtain the normal charmm-gui structure files for gromacs using the CHARMM36 force field.
Note that the lipids you choose should be supported by SLipids.
2. Change all the `HN` atomnames of the protein to  `H` in the `step5_input.pdb`.
3. Capped termini (ACE and NME) in charmm-gui are included inside the
residue but in the AMBER .rtp they are a different residue called ACE and NME
with sligthly different atom names. You'll have to split the terminal residues
into two (i. e. CYS+NME)
4. The CY atom of ACE should be changed to C.
5. If some of the charged residues are neutral you should change their resnames
e. g. GLU -\> GLH if you have protanated glutamates.
6. Add the approprient pbc to the pdb (can be found in `step5_input.gro`).
E. g. `CRYST1   81.029   81.029  102.123  90.00  90.00  90.00 P 1           1`
7. At this point your file should look like `example/outputs/step5_input_modified.pdb`
8. From the modified `step5_input.pdb` make a pdb file with just the protein.
9. Run `pdb2gmx` only on the protein pdb to get a topol.top file. Make sure `gmx` can find
the `amber14sb_slipids16.ff` directory.
10. Modify the topol.top file to add the non-protein molecules (water, lipids, ions etc. ) in the `[ molecules ]` section.
You can base this in the original `topol.top` from charmm-gui.
11. Add the missing .itp files of the lipid molecules that have been included:
```text
#include "./amber14sb_slipids16.ff/POPC.itp"
#include "./amber14sb_slipids16.ff/POPG.itp"
#include "./amber14sb_slipids16.ff/CHL1.itp"
```
12. Your `topol.top` file should look like the one on `example/outputs/topol.top`
13. Manually add restrains to `topol_Protein_chain_A.itp`.
This can be done by adding the same restrains that are generated by charmm-gui for the charmm-FF.
Checkout `example/outputs/topol_Protein_chain_A.itp` as an example.
14. Generate mdp file that is appropriet for AMBER-FF which is different than the ones produced by charmm-gui. You can use `example.mdp` but most changes only involve PME and VdW settings.
15. Run grompp on the full system pdb. If there are any warnings atomnames, there might be the need for some atom reordering in the rtp files. This shouldn't happen
with normal aminoacids or lipids with the itps that have been made. I like to use the `-pp topol_pp.top` option to generate a standalone top file that I can move around. Nevertheless this top file must be remade if you change the restrains.

## Make itp for lipids present in lipids.rtp but not in this repository.

If in this directory there is no itp file for a particular lipid but it is a part of lipids.rtp do the following:
1. Make a pdb file containing only one lipid as obtained from charmm-gui (e. g. POPC.pdb)
2. Run `gmx pdb2gmx -f POPC.pdb`
3. Edit the obtained topol.top and convert it to a `POPC.itp` itp file.
4. Move POPC.itp to amber14sb_parmbsc1.ff
5. There might be atomname problems when running grompp.
The only way to fix this is reorder atoms in lipids.rtp and remake the lipid's itp file.
6. Restrains to lipids can be added to the itp file using the same restrains that are generated by charmm-gui for the CHARMM-FF.

## Other missing stuff

There are some discrepancies between how CHARMM-GUI and the `aminoacids.rtp` names
some aminoacid (termini, protonation states etc.) and some atomnames. I have prepared
these files to accomodate my needs, but if your problem needs modifications let me know
how I can help. If you were able to add new things, please share them with me so I
can incorporate them to the repo!
