

![Aerobulk Logo](https://brodeau.github.io/images/projects/aerobulk_logo_s.svg)

Online documentation: https://brodeau.github.io/aerobulk/

**AeroBulk** is a Fortran package/library which gathers state-of-the-art
algorithms, turbulent closures, parameterizations, and thermodynamics (empirical) functions used to compute
turbulent fluxes at the air-sea interface.
These turbulent fluxes are wind stress, evaporation (latent heat flux) and
sensible heat flux. These fluxes are estimated by means of so-called **aerodynamic bulk formulae** from the
sea surface temperature, and atmospheric surface parameters: wind speed, air temperature and specific
humidity.

![Aerobulk Logo](https://brodeau.github.io/images/projects/bulk.svg)

In AeroBulk, 4 algorithms are available to compute
the drag, sensible heat and moisture transfer coefficients (C<sub>D</sub>,
C<sub>H</sub> and C<sub>E</sub>) used in the bulk formulaes:

*   COARE v3.0 ([Fairall _et al._ 2003](http://dx.doi.org/10.1175/1520-0442(2003)016<0571:BPOASF>2.0.CO;2))
*   COARE v3.5 ([Edson _et al._ 2013](http://dx.doi.org/10.1175/jpo-d-12-0173.1))
*   ECMWF ([IFS (Cy40) documentation](https://software.ecmwf.int/wiki/display/IFS/CY40R1+Official+IFS+Documentation))
*   NCAR (Large & Yeager 2004, [2009](http://dx.doi.org/10.1007/s00382-008-0441-3))

In the COARE and ECMWF algorithms, a cool-skin/warm layer scheme is included 
and can be activated if the input sea-surface temperature is the bulk SST
(usually measured a few tenths of meters below the surface). Activation
of these cool-skin/warm layer schemes requires the surface downwelling shortwave
and longwave radiative flux components to be provided. The NCAR algorithm is to
be used only with the bulk SST.

Beside bulk algorithms AeroBulk also provides a variety of functions to accurately estimate relevant atmospheric variable such as density of air, different expressions of the humidity of air, viscosity of air, specific humidity at saturation, Monin-Obukhov length, wind gustiness, etc...  

The focus in AeroBulk is readability, efficiency and portability towards either
modern GCMs (Fortran 90, set of modules and a library).






# **> Computing turbulent fluxes with AeroBulk**

AeroBulk can also directly compute the 3 turbulent fluxes with the routine aerobulk\_model()
of module **mod\_aerobulk** (mod\_aerobulk.f90):

       PROGRAM TEST_FLUX
           USE mod_aerobulk
           ...
           CALL AEROBULK_MODEL( calgo, zt, zu, sst, t_zt, q_zt, U_zu, V_zu, SLP, &
           &                    Qe, Qh, Tau_x, Tau_y                             &
           &                   [, Niter=N, rad_sw=Rsw, rad_lw=Rlw] )
           ...
       END PROGRAM TEST_FLUX

INPUT ARGUMENTS:

*   calgo: (String) algorithm to use (coare/coare35/ncar/ecmwf)
*   zt : (Sc,real) height for temperature and spec. hum. of air [m]
*   zu : (Sc,real) height for wind speed (generally 10m) [m]
*   sst : (2D,real) SST [K]
*   t_zt : (2D,real) potential air temperature at zt [K]
*   q_zt : (2D,real) specific humidity of air at zt [kg/kg]
*   U_zu : (2D,real) zonal scalar wind speed at 10m [m/s]
*   V_zu : (2D,real) meridional scalar wind speed at 10m [m/s]
*   SLP : (2D,real) sea-level pressure [Pa]

[ OPTIONAL INPUT ARGUMENT: ]

*   Niter: (Sc,int) number of iterations (default is 4)
*   rad_sw: (2D,real) downw. shortwave rad. at surface (>0) [W/m^2]
*   rad_lw: (2D,real)downw. longwave rad. at surface (>0) [W/m^2]

(The presence of rad_sw and rad_sw triggers the use of the Cool-Skin Warm-Layer parameterization with COARE* and ECMWF algorithms)

OUTPUT ARGUMENTS:

*   Qe : (2D,real) latent heat flux [W/m^2]
*   Qh : (2D,real) sensible heat flux [W/m^2]
*   Tau_x : (2D,real) zonal wind stress [N/m^2]
*   Tau_y : (2D,real) meridional wind stress [N/m^2]

Example of a call, using COARE 3.0 algorithm with cool-skin warm-layer parameterization and 10 iterations:

           CALL AEROBULK_MODEL( 'coare', 2., 10., sst, t_zt, q_zt, U_zu, V_zu, SLP, &
           &                    Qe, Qh, Tau_x, Tau_y,                               &
           &                    Niter=10, rad_sw=Rsw, rad_lw=Rlw )






# **> Computing transfer coefficients with AeroBulk**

In AeroBulk, 3 different routines are available to compute the bulk transfer
(_a.k.a_ exchange) coefficients C<sub>D</sub>, C<sub>H</sub> and
C<sub>E</sub>. Beside computing the transfer coefficients, these routines adjust
air temperature and humidity from height _z<sub>t</sub>_ to the reference height
(wind) _z<sub>u</sub>_. They also return the bulk wind speed, which is the
scalar wind speed at height _z<sub>u</sub>_ with the potential inclusion of a
gustiness contribution (in calm and unstable conditions).


**> TURB_COARE, transfer coefficients with COARE 3.0/3.5**

Use _turb_coare() of module **mod\_blk\_coare** (mod\_blk\_coare.f90).  
Example of a call:

              PROGRAM TEST_COEFF
                  USE mod_const
                  USE mod_blk_coare
                  ...
                  jpi = Ni ! x-shape of the 2D domain
                  jpj = Nj ! y-shape of the 2D domain
                  ...
                  CALL TURB_COARE( cver, zt, zu, T_s, t_zt, q_s, q_zt, U_zu,  &
                  &                Cd, Ch, Ce, t_zu, q_zu, U_blk              &
                  &                [ , rad_sw=Rsw, rad_lw=Rlw, slp=P ]        &
                  &                [ , xz0=z0, xu_star=u_s, xL=L ] )
                  ...
              END PROGRAM TEST_COEFF

**INPUT ARGUMENTS:**

*   cver: (String) version of the COARE algorithm ['3.0' or '3.5']
*   zt: (Sc,real) height for air temperature and humidity [m]
*   zu: (Sc,real) height for wind speed (generally 10m) [m]
*   t_zt: (2D,real) potential air temperature at zt [K]
*   q_zt: (2D,real) air spec. humidity of at zt [kg/kg]
*   U_zu: (2D,real) scalar wind speed at zu [m/s]

**INPUT and OUTPUT ARGUMENTS:**

*   T_s: (2D,real) surface temperature [K]
    *   input: bulk SST
    *   output: skin temperature or SST (unchanged)
*   q_s: (2D,real) surface satur. spec. humidity at T_s [kg/kg]
    *   input: saturation at bulk SST (not needed if skin p. used)
    *   output: saturation at skin temp. or at SST (unchanged)

**[ OPTIONAL INPUT ARGUMENTS: ]**

*   rad_sw: (2D,real) downw. shortw. rad. at surface (>0) [W/m^2]
*   rad_lw: (2D,real) downw. longw. rad. at surface (>0) [W/m^2]
*   slp: (2D,real) sea-level pressure [Pa]

(The presence of these 3 optional input parameters triggers the use of the
Cool-Skin Warm-Layer parameterization)

**OUTPUT ARGUMENTS:**

*   Cd: (2D,real) drag coefficient
*   Ch: (2D,real) sensible heat transfer coefficient
*   Ce: (2D,real) moisture transfer (evaporation) coefficient
*   t_zu: (2D,real) air pot. temperature adjusted at zu [K]
*   q_zu: (2D,real) air spec. humidity adjusted at zu [kg/kg]
*   Ublk: (2D,real) bulk wind speed at 10m [m/s]

**[ OPTIONAL OUTPUT ARGUMENTS: ]**

*   z0: (2D,real) roughness length of the sea surface [m]
*   u_s: (2D,real) friction velocity [m/s]
*   L: (2D,real) Monin-Obukhov length [m]

**> Some Examples**

Using COARE 3.0 without the cool-skin warm-layer parameterization, with air temperature and humidity provided at 2m and wind at 10m:  

              PROGRAM TEST_COEFF
                  USE mod_const
                  USE mod_blk_coare
                  ...
                  jpi = Ni ! x-shape of the 2D domain
                  jpj = Nj ! y-shape of the 2D domain
                  ...
                  CALL TURB_COARE( '3.0', 2., 10., Ts, t2, qs, q2, U10, &
                  &                Cd, Ch, Ce, t10, q10, U_blk )
                  ...
              END PROGRAM TEST_COEFF

In this case, Ts and qs, the surface temperature and saturation specific humidity won't be modified. The relevant value of qs must be provided as input.  

Now the same but using the cool-skin warm-layer parameterization:

              PROGRAM TEST_COEFF
                  USE mod_const
                  USE mod_blk_coare
                  ...
                  jpi = Ni ! x-shape of the 2D domain
                  jpj = Nj ! y-shape of the 2D domain
                  ...
                  CALL TURB_COARE( '3.0', 2., 10., Ts, t2, qs, q2, U10, &
                  &                Cd, Ch, Ce, t10, q10, U_blk,         & 
                  &                rad_sw=Rsw, rad_lw=Rlw, slp=MSL )
                  ...
              END PROGRAM TEST_COEFF

Here, Ts is the bulk SST as input and will become the skin temperature as
output! qs is irrelevant as input and is the saturation specific humidity at
temperature Ts as output!




# **> Computing atmospheric state variables with AeroBulk**

A selection of useful functions to estimate some atmospheric state
variables in the marine boundary layer are available in the module
**mod\_thermo** (mod\_thermo.f90).  
Example for computing SSQ of Eq.(1) out of the SST and the SLP:

              PROGRAM TEST_THERMO
                  USE mod_const
                  USE mod_thermo
                  ...

                  SSQ(:,:) = 0.98*q_sat(SST, SLP)
                  ...
              END PROGRAM TEST_THERMO








**> Acknowledging AeroBulk**

_To acknowledge/reference AeroBulk in your scientific work, please cite:_  
Brodeau, L., B. Barnier, S. Gulev, and C. Woods, 2016: Climatologically significant effects of some approximations in the bulk parameterizations of turbulent air-sea fluxes. _J. Phys. Oceanogr._, **47 (1)**, 5–28, 10.1175/JPO-D-16-0169.1. [ [DOI](http://dx.doi.org/10.1175/JPO-D-16-0169.1) ]  
