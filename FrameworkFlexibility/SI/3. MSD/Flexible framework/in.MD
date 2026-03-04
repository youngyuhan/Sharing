#################################### set box ###########################################################
units           real
atom_style      full
boundary        p p p

########################### set initial config and FF ###################################################
bond_style      harmonic
angle_style     hybrid fourier cosine/periodic
dihedral_style  harmonic
improper_style  fourier
pair_style      hybrid/overlay lj/cut 12.5 coul/long 12.5
pair_modify     tail yes mix arithmetic
kspace_style	ewald 1.0E-06
#kspace_modify	gewald 0.5

dielectric      1.0
special_bonds   lj/coul 0.0 0.0 1.0
box tilt        large
read_data       LUXFOA_O2.data

pair_coeff	1	1	lj/cut	0.06	3.118146
pair_coeff	1	2	lj/cut	0.064342832	3.1894175
pair_coeff	2	2	lj/cut	0.069	3.260689
pair_coeff	1	3	lj/cut	0.079372539	3.2744985
pair_coeff	2	3	lj/cut	0.085117566	3.34577
pair_coeff	3	3	lj/cut	0.105	3.430851
pair_coeff	1	4	lj/cut	0.02792848	2.8780485
pair_coeff	2	4	lj/cut	0.029949958	2.94932
pair_coeff	3	4	lj/cut	0.036945906	3.034401
pair_coeff	4	4	lj/cut	0.013	2.637951
pair_coeff	1	5	lj/cut	0.05138093	2.84464
pair_coeff	2	5	lj/cut	0.055099909	2.9159115
pair_coeff	3	5	lj/cut	0.067970582	3.0009925
pair_coeff	4	5	lj/cut	0.023916521	2.6045425
pair_coeff	5	5	lj/cut	0.044	2.571134
pair_coeff	1	6	lj/cut	0.076446059	3.069073
pair_coeff	2	6	lj/cut	0.081979266	3.1403445
pair_coeff	3	6	lj/cut	0.101128631	3.2254255
pair_coeff	4	6	lj/cut	0.035583704	2.8289755
pair_coeff	5	6	lj/cut	0.065464494	2.795567
pair_coeff	6	6	lj/cut	0.0974	3.02
pair_coeff	1	7	lj/cut	0	1.559073
pair_coeff	2	7	lj/cut	0	1.6303445
pair_coeff	3	7	lj/cut	0	1.7154255
pair_coeff	4	7	lj/cut	0	1.3189755
pair_coeff	5	7	lj/cut	0	1.285567
pair_coeff	6	7	lj/cut	0	1.51
pair_coeff	7	7	lj/cut	0	0
					
pair_coeff	1*6	6	coul/long		
pair_coeff	1*7	7	coul/long		

neighbor        2.0 bin
neigh_modify    every 1 delay 0 check yes

group           frame type 1:5
group           adsorbate type 6:7

############################ Record thermodynamic data #################################################
compute         COM_ads adsorbate com
variable        xad equal c_COM_ads[1]
variable        yad equal c_COM_ads[2]
variable        zad equal c_COM_ads[3]
variable        t equal 308.15
variable        s equal myseed #random seed

thermo          1000 
log             thermo.dat
thermo_style    custom step etotal pe ke enthalpy vol press temp v_xad v_yad v_zad 
compute         t_temp all temp
thermo_modify   temp t_temp format float %4.10g

################################# pre-run setting #####################################################
timestep        1
run_style       verlet
fix             TETHER1 all momentum 1 linear 1 1 1 angular
velocity        all create $t $s rot yes mom yes dist gaussian

######################################  NVT #############################################################
print           "------> Equilibrium Run"
fix             FLEX_FRAME frame nvt temp $t $t 100 
fix             RIGID_MOLECULE adsorbate rigid/nvt molecule temp $t $t 100 
fix             RECENTER frame recenter INIT INIT INIT
run             10000000
                                       
reset_timestep  0

print           "------> MSD Run"
# mean square displacment
compute         1 adsorbate msd
variable        msdx equal "c_1[1]"
variable        msdy equal "c_1[2]"
variable        msdz equal "c_1[3]"
variable        msd  equal "c_1[4]"
variable        mdstep equal "step"

fix             2 adsorbate print 1000 "${mdstep}  ${msdx}  ${msdy}  ${msdz}  ${msd}" screen no append msd_traj.ads

dump            Trajectory all xyz 100000 snapshots.xyz
dump_modify     Trajectory element O N C Mn H Na K

run             40000000

unfix           FLEX_FRAME
unfix           RIGID_MOLECULE
