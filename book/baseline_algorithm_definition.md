# Baseline Algorithm Definition



The CIMR Level-2 {term}`SIC` algorithm has 3 main steps:
1. L1B remapping
2. Computation of intermediate SICs
3. Pansharpening for Level-2 SICs.

The first two steps above are applied three times, for three combinations of CIMR TB channels `CKA`, `KUKA`, and `KA`. `CKA` and `KUKA` combinations
have both been tested in the EUMETSAT OSI SAF and ESA CCI Sea Ice projects {cite:p}`lavergne:2019:sicv2`. The `KU` combination is the basis for the
'Polarization Mode' of the Bootstrap SIC algorithm {cite:p}`comiso:1986:sic,ivanova:2015:sicci1`.

{numref}`combis` introduces the three channel combinations used to prepare intermediate SICs in the Level-2 algorithm.
The channel combinations are applied at different spatial resolutions, dictated by the lowest frequency of the set. They result
in different accuracies (algorithms using C-band have better accuracy than those using only Ku and Ka channels). Note that the '<= X km'
in column 'Resolution' corresponds to the spatial resolution after the L1B remapping step and is thus at least the native resolution of
the coarsest channel, but can also be better in case Backus-Gilbert techniques are implemented to take advantage of the overlap of FoVs
(most relevant at C-band and away from the swath center at Ku-band).

```{table} The three combinations of microwave channels used for the intermediate SICs.
:name: combis
| Id          | Channels  |  Resolution |   SIC Accuracy |
| ----------- | --------  | ------ | ------ |
| `CKA`       | (C-Vpol, Ka-Vpol, Ka-Hpol) |  C (<= 15 km)  |  Excellent |
| `KUKA`       | (Ku-Vpol, Ka-Vpol, Ka-Hpol) |  Ku (<= 5 km)  |  Good |
| `KA`       | (Ka-Vpol, Ka-Hpol) |  Ka (4-5 km)  |  Poor |
```

The third step deploys a pan-sharpening methodology to combine pairs of intermediate SICs into the final Level-2 SICs. Pan-sharpening (aka image fusion, resolution enhancement, etc...) is
a robust and straightforward method to improve the spatial resolution of a 'base' image by means of a 'sharpener' image. Pan-sharpening has been used
in many fields of Earth Observation, mostly with higher-resolution optical imagery {cite:p}`dadrasjavan:2021:pansharpen`. In the context of passive microwave SIC it has
been used in the NORSEX algorithm {cite:p}`kloster:1996:eocompendium` and more recently by {cite:t}`kilic:2020:sic` and in the context of the
ESA CCI+ Sea Ice project.

In our case, the 'base' image can be the high-accuracy / coarse-resolution intermediate SIC from `CKA` and the 'sharpener' image can be the
low-accuracy / fine-resolution intermediate SIC from `KA`. But other pan-sharpening combinations are possible. All pan-sharpened SICs
should be made available in the Level-2 product (with one of them given prominent visibility as an entry point).

```{table} Level-2 SICs, most of them obtained from pan-sharpening of intermediate SICs.
:name: pansharp
| Id          | Base image  |  Sharpener |   Resolution | SIC Accuracy |
| ----------- | --------    | ------     | ------ | --- |
| `CKA`       | SIC from `CKA` | n/a | C (<= 15 km)  |  Excellent |
| `CKA@KU`    | SIC from `CKA` | SIC from `KUKA` | Ku (<= 5 km)  |  Excellent |
| `CKA@KA`    | SIC from `CKA` | SIC from `KA` | Ka (4-5 km)  |  Excellent |
| `KUKA@KA`   | SIC from `KUKA` | SIC from `KA` | Ka (4-5 km)  |  Good |
```

{numref}`pansharp` lists candidate Level-2 SICs from CIMR, most of them obtained from pan-sharpening of intermediate SICs (`CKA` is directly
from the intermediate SICs, without pan-sharpening). It is expected that one of these candidate Level-2 SICs will emerge as the best for most
users and be flagged as the main L2 SIC product (e.g. `CKA@KA`).

### Retrieval Method

Subsection Text


### Forward Model

Subsection Text


### CIMR Level-1b re-sampling approach

Subsection Text


### Algorithm Assumptions and Simplifications

Subsection Text

### Level-2 end to end algorithm functional flow diagram

Subsection Text

### Functional description of each Algorithm step

Subsection Text

##### Mathematical description

SubSubsection Text
##### Input data

SubSubsection Text

##### Output data

SubSubsection Text

##### Auxiliary data

SubSubsection Text

##### Ancillary data

SubSubsection Text

##### Validation process

SubSubsection Text


