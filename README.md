# Pluging PHYEX in numerical weather prediction model

Dependence and compilation
--------------------------
PHYEX has a dependence with [fiat](https://github.com/ecmwf-ifs/fiat).

Since the different parameterizations have been developed together, it is advisable to compile the whole PHYEX rather than trying to isolate the parameterizations from each other. It will still be possible, in the model, to call only some of them.

PHYEX is delivered with a compilation system that could allow the creation of an autonomous library that can be called from the model. However, to ensure full compatibility with the compilation options of the host model, it is preferable to insert the PHYEX sources directly into the compilation system of the model.

Calling order
-------------
Several orders of call of the parameterizations are certainly possible, here is presented the one used in the AROME model:

  - saturation adjustment
  - radiation (not included in PHYEX)
  - surface (not included in PHYEX)
  - shallow convection
  - turbulence
  - microphysics (except saturation adjustment done at the beginning)

The saturation adjustment is called first. It modifies the state variables to provide adjusted variables to the other processes (temperature, vapor / cloud condensates mixing-ratio and cloud fraction). Here is the list of the dependencies between the parameterizations:

|Variable name        |Variable content                                                   |Produced by                      |Used by                        |
|---------------------|-------------------------------------------------------------------|---------------------------------|-------------------------------|
|s'r' / sigmas_s**2   |Normalized correlation between over-saturation and condensate      |saturation adjustement           |turbulence                     |
|RC_MF / RI_MF / CF_MF|Shallow convection cloud (liquid and ice mixing-ratio and fraction)|shallow convection (CLOUD='DIRE')|saturation adjustement         |
|sigma_s_MF           |Shallow convection contribution to the over-saturation s.t.d       |shallow convection (CLOUD='STAT')|saturation adjustement         |
|sigmas_s_turb        |Turbulence contribution to the over-saturation s.t.d               |turbulence                       |saturation adjustment & microphysics    |
|SFTH / SFRV          |Surface fluxes of theta and water vapor mixing ratio               |surface                          |turbulence and shallow convect.|
|SFU / SFV            |Surface momentum fluxes                                            |surface                          |turbulence                     |
|FLXTHV_MF            |Shallow convection contribution to the theta_v vertical flux       |shallow convection               |turbulence                     |
|MF_UP                |Shallow convection mass flux                                       |shallow convection               |turbulence                     |
|RC / RI              |Cloud liquid and ice mixing-ratio                                  |saturation adjustement           |radiation, surface, shallow convection, turbulence, microphysics|
|CF                   |Cloud fraction                                                     |saturation adjustement           |radiation, microphysics        |

There is no magic order and it is necessary to save some outputs of the physics to be used during the next time step. With the AROME order, depending on the shallow convection cloud scheme used, 2 or 4 variables must be saved:

- sigmas_s_turb
- outputs of the shallow convection cloud scheme (sigma_s_MF if CLOUD='STAT', RC_MF / RI_MF / CF_MF otherwise)
