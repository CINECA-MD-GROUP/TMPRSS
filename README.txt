
Procedure
---------

0. Minimisation
gmx grompp -f step6.0_minimization.mdp -c step5_input.gro -p topol.top -o min.tpr -maxwarn 1

1. Equilibration NVT
gmx grompp -v -f eq1.mdp -c step6.0_minimization.gro -n index.ndx -p topol.top -maxwarn 1

2. Equilibration NPT
gmx grompp -v -f eq2.mdp -c eq1.gro -t eq1.cpt -n index.ndx -p topol.top -maxwarn 1

3. 1 microsecond production (NPT)
gmx grompp -f prod.mdp -c eq2.gro -t eq2.cpt -p topol.top -n index.ndx -maxwarn 1


For replica production run edit prod.mdp and uncomment:

continuation = no
gen_vel=yes
gen_temp=303.15
gen_seed = -1

and re-rerun grommp, without -t option.



Notes
-----

1. The step*.mdp files from Charmm-gui gave LINCS problems (due to restraints) so abandoned them
using instead procedure from: http://www.mdtutorials.com/gmx/membrane_protein/06_equil.html
*however* I retained the cutoffs and other params from the CHARMGUI:

rcoulomb    = 0.9       ; short-range electrostatic cutoff (in nm)
rvdw        = 0.9       ; short-range van der Waals cutoff (in nm)
tau_p       = 5.0                   ; time constant, in ps
ref_p       = 1.0   1.0             ; reference pressure, x-y, z (in bar)
compressibility = 4.5e-5    4.5e-5
tau_t       = 1.0 1.0
ref_t       = 303.15  303.15     

2. vdwtype=PME
This was in the charmm files and is recommended by the gmx developers but:
 - cannot be used with GPU direct on M100
 - grompp gives a note suggesting not compatible with C6 parameters we used

therefore set
   vdwtype=Cut-off

3. tc-groups= SOLU_MEMB SOLV
Unlike charmm, but agreeing with the tutorial , I used two: 
 membrane+protein solvent 
makes more sense given the protein is in the membrane and gives fewer T fluctuations.

4. temperature coupling
-charmm uses berendsen(eq) then nose-hoover (prod)
-tutorial uses V-rescale (NVT) then Nose-hoover (NPT and production)

For equilibration used V-rescale (NVT) then Nose-Hoover (NPT)
Nose-hoover not supported on GPU so for production used V-rescale

5. constraints =all-bonds
both charmm and tutorial use all-bonds.
However, all-bonds not supported on GPU so we have used:

constraints = h-bonds

for production. For equilibration = all-bonds

6. For performance:

nstcalcenergy = 25000 (instead of 100) ; mdp option
-nstlist 400 (mdrun option)
-bonded gpu (mdrun option) - makes a big difference

