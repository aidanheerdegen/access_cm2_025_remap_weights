# Remapping weights for ACCESS-CM2-025 config

The ACCESS CM2 model coupled to a 0.25 degree ocean model
is a new configuration being developed by the CSIRO, with assistance
from the ARC Centre of Excellence for Climate Extremes.

One required element is new remapping weights files for the OASIS
coupler to exchange fields between the Atmosphere and Ice components.

This repository contains grid files in SCRIPT format, from which the 
remapping weights are generated. 

The MOM Ocean model grid is the same as the Ice model grid, through 
which fields are exchanged between Ocean and Atmosphere. 

There are three UM atmospheric grids, tracer (`um1t`), u (`um1u`) and 
v (`um1v`), all of which were generated from this grid file:
```
/g/data/access/access-cm2/input_O1/cpl_n96/oasis3_grids_N96_06032014.nc
```

The repo contains a script, `make_weights` which creates the remapping
weights files using the [ESMF_RegridWeightGen](http://earthsystemmodeling.org/docs/release/ESMF_8_0_1/ESMF_refdoc/node3.html#SECTION03020000000000000000) tool.

A local version of the ESMF_RegridWeightGen tool was compiled with 
bug fixes for tripole grids from Russ Fiedler. 

```
/scratch/v45/raf599/esmf/apps/appsO/Linux.intel.64.openmpi.default/ESMF_RegridWeightGen
```

This was copied and used in this script.

The remapping weights required was based on the existing ACCESS-CM2 
namcouple file:
```
/g/data/access/access-cm2/input_O1/cpl_n96/namcouple_112fields_360_bil_GA7.1
```
It lists a total of 112 fields being exchanged
```
# This is the total number of fields being exchanged. 
### 38 fields  atm -> ice
### 19 fields  ice -> ocn
###  9 fields  ocn -> ice
### 46 fields  ice -> atm
```
We are only concerned with the atm/ice and ice/atm fields, 84 in total. Of those
there are 80 fields using `um1t`, 2 using `um1u` and 2 using `um1v`.

Breaking it down further, for `um1t` there are 44 fields `ocn/cice` -> `um1t`, all
of which are 1st order conservative. There are 36 fields in the opposite direction,
`um1t` -> `cice`, 34 are first order conservative, and 2 bilinear.

Each of the u and v grids have a field exchanged with the ocean/ice grid, bilinear 
from atmosphere to ocean/ice, and conservative from ocean/ice to atmosphere.

This is summarised in the following table, where the source grid is in the left 
most column, and the destination grids are in the right hand columns.

| source   | cice/ocn    | um1t | um1u | um1v |
| -------- | ----------- | ---- | ---- | ---- |
| cice/ocn |    x        | cons | cons | cons | 
| um1t     | bilin / cons|  x   |  x   |  x   |
| um1u     | bilin       |  x   |  x   |  x   |
| um1v     | bilin       |  x   |  x   |  x   |

Second order conservative remapping has been used, as this produces smoother fields
with better derivatives. 

In addition to bilinear remapping weights files, the patch recovery weighting scheme
has also been generated, as it typically performs better than bilinear, so that both
schemes could be evaluated. 
