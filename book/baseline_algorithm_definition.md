# Baseline Algorithm Definition

In the following sections, the Level-2 sea-ice concentration algorithm is further described. It consists in those steps:

The CIMR Level-2 {term}`SIC` algorithm has 3 main steps ({numref}`fig_sic_concept`):
1. L1B remapping
2. Computation of intermediate SICs
3. Pansharpening for Level-2 SICs.

```{figure} ./static_imgs/SIC_concept_diagram.png
--- 
name: fig_sic_concept
---
Concept of the CIMR Level-2 {term}`SIC` processing moving from input L1B TB (left) to Level-2 SICs (right). Ellipses and colours represent
different microwave channels and their spatial resolution.
```

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

As introduced above, the CIMR Level-2 SIC processing implements a series of steps that are run on three different combinations of microwave channels. Here
we present the steps on a generic channel combination. We refer to this generic description later in the chapter when describing the actual implementation.

#### A generic formulation for a SIC algorithm and its uncertainties

At any microwave channel $i$, and neglecting the contribution of the atmosphere to the signal, the TB recorded by CIMR over polar ocean areas is a linear combination of the
typical signature of open water $<T_{i}^{W}>$ (the open water tie-point), and of the typical signature of consolidated ice (100\% SIC) $<T_{i}^{I}>$ (the sea ice tie-point).
The weighting between the two signatures is the {term}`SIC` (noted $C$). Considering $n$ microwave channels leads to:

$$
\left \{
\begin{array}{c}
T_{1} = (1 - C) \times <T_{1}^{W}> + C \times <T_{1}^{I}> \\
... \\
T_{n} = (1 - C) \times <T_{n}^{W}> + C \times <T_{n}^{I}>
\end{array}
\right.
$$ (eq_mixTb)

where $<T>$ stands for the average $T$.

By re-arranging the terms in Eq. {eq}`eq_mixTb`, and taking the dot product of both sides of the equations by a unit vector \vec{v}, we obtain:

$$
C(\vec{T}) = \frac{\vec{v} \cdot (\vec{T} - <\vec{T}^W>)}{\vec{v} \cdot (<\vec{T}^I> - <\vec{T}^W>)}
$$ (eq_mixC2)

where $\vec{T}=(T_1,...,T_i,...,T_n)$ the vector of $T_B$ measured in $n$ channels, and $\vec{v}=(v_1,...,v_i,...,v_n)$ a unit vector in $\mathbb{R}^n$ ($\|\vec{v}\|=1$). In general, $\vec{T}$ can include all
microwave channels of a microwave imager ($n=10$ for CIMR counting only the V- and H-pol channels). For this Level-2 SIC algorithm however, we consider $n=2$ (`KA` configuration) and $n=3$ (`CKA` and `KUKA` configurations).

Eq. {eq}`eq_mixC2` defines a general form for our SIC algorithms. The SIC is the ratio of the distance from the measured TB to the water tie-point, to the distance between the ice and the water tie-points, both distances being measured along an $n$-dimensional direction $\vec{v}$. By construction, these algorithms all achieve zero bias on average at the water and consolidated sea-ice cases, since $C(<\vec{T}^W>)=0$ and $C(<\vec{T}^I>)=1$.

Vector $\vec{v}$ holds the coefficients of the algorithm, that give relative weights to the $n$ channels. Once the tie-point are known, the crux of designing algorithms is thus to find $\vec{v}$ that yields the best SIC accuracy.

Uncertainty propagation can be applied to express the uncertainty in $C(\vec{T})$ as a function of the uncertainties in the free parameters in equation {eq}`eq_mixC2`: 

$$
\Sigma_C = \frac{1}{(\vec{v} \cdot (<\vec{T}^I>-<\vec{T}^W>))^2} [ \vec{v}\Sigma_T\vec{v}^T + (1-C)^{2}\times \vec{v}\Sigma_W\vec{v}^T + C^{2}\times \vec{v}\Sigma_I\vec{v}^T ]
$$ (eq_varFull)

The three terms in Eq. {eq}`eq_varFull` correspond respectively to the uncertainty contribution due to the radiometer instrument noise (the Ne$\Delta$T), the geo-physical
uncertainty of the Water emissivities (including surface effects from wind roughening, temperature, etc...) and those of the Ice emissivities (including a number of contributions
such as ice type, snow depth, layering, etc...)

Eq. {eq}`eq_varFull` expresses the total uncertainty (variance) of SIC as a combination of the uncertainty (variance) contributions from the Water and Ice signatures (\emph{aka} tie-points uncertainty)
and the instrument noise, normalized by the dynamic range between Ice and Water mean signatures, measured along the direction of $\vec{v}$. If the dynamic range (in the denominator) is small with
respect to the uncertainties (in the numerator), then the total uncertainty will be large. This is the main reason why algorithm using C-band (large dynamic range between water and ice) have lower
retrieval uncertainty than those only using Ku- or Ka-band (smaller dynamic range).

An alternative expression for eq. {eq}`eq_varFull` is given in eq. {eq}`eq_varSimp`:

$$
\Sigma_C = \Sigma_{Ne \Delta T} + (1-C)^{2} \times \Sigma_{CW} + C^{2} \times \Sigma_{CI}
$$ (eq_varSimp)

with

$$
\Sigma_{Ne \Delta T} = \frac{\vec{v}\Sigma_T\vec{v}^T}{(\vec{v} \cdot (<\vec{T}^I>-<\vec{T}^W>))^2}
$$ (eq:varT)

$$
\Sigma_{CW} = \frac{\vec{v}\Sigma_W\vec{v}^T}{(\vec{v} \cdot (<\vec{T}^I>-<\vec{T}^W>))^2}
$$ (eq:varW)

$$
\Sigma_{CI} = \frac{\vec{v}\Sigma_I\vec{v}^T}{(\vec{v} \cdot (<\vec{T}^I>-<\vec{T}^W>))^2}
$$ (eq:varI)

Scalars $\Sigma_{CW}$ and $\Sigma_{CI}$ are the uncertainty (variance) achieved at the low-end (open water) and high-end (consolidated ice) of the sea-ice concentration range.
They are the two scalars we want to minimize when tuning our SIC algorithms. Given the variance-covariance matrices of the radiometric-induced noise, and those induced by the water
and the consolidated ice signatures, the algorithm coefficients embedded in $\vec{v}$ control if the algorithm is more or less accurate at either end of the SIC range. 

#### Dynamic tuning of the generic SIC algorithm

The algorithm coefficients embedded in $\vec{v}$ control the retrieval accuracy (eq. {eq}`eq_varSimp`) of the generic SIC algorithm (eq. {eq}`eq_mixC2`). An important step
for defining a SIC algorithm is thus to tune $\vec{v}$ to achieve best accuracy. Consider a set of TB samples extracted over known 0% SIC conditions, and another set extracted
over known 100% SIC conditions. We describe below the procedure to tune a generic SIC algorithm against these two TB samples. The strategies to select these TB samples will be
presented at a later stage.

##### Computation of the tie-points and tie-points variability

From the set of TBs for known 0% SIC conditions, we compute the mean TB signature $<\vec{T}^W>$ (the water tie-point) and the variance-covariance matrix of the variability around the tie-point: $\Sigma_W$. 

From the set of TBs for known 100% SIC conditions, we compute the mean TB signature $<\vec{T}^I>$ (the ice tie-point) and the variance-covariance matrix of the variability around the tie-point: $\Sigma_I$. 

##### Defining $\vec{u}$ as the vector sustaining the ice line

We compute the unit vector $\vec{u}$ that sustains the direction of largest variance in $\vec{v}\Sigma_I$. $\vec{u}$ is obtained by Principal Component Analysis (PCA) of the set of TBs for
known 100% SIC conditions. $\vec{u}$ defines the "consolidated ice line", a concept used in many heritage SIC algorithms including the Bootstrap algorithms {cite:p}`comiso:1986:sic` and
the Bristol algorithm {cite:p}`smith:1996:bristol`. The ice line extends from Multiyear ice (MYI) to Firstyear ice (FYI).

##### Optimizing $\vec{v}$ perpendicular to the ice line

Vector $\vec{v}$ as entering eq. {eq}`eq_mixC2` is chosen to be perpendicular to vector $\vec{u}$.

$$
\vec{u} \cdot \vec{v} = 0
$$ (eq_vdotu)

This ensures that any TB points on the ice line gets a SIC of 100% across all sea-ice types (along the ice line).

For a 2-dimensional algorithm ($n=2$ as for the `KA` configuration), the constraint in eq. {eq}`eq_vdotu` results in a
unique vector $\vec{v}$. Indeed, let $\vec{u}=(u_1,u_2)$ then $\vec{v}=(-u2,u1)$. A 2-dimensional algorithm like `KA` is
fully defined from the two sets of TBs over known ocean and ice conditions, since those sets define the tie-points
$<\vec{T}^W>$ and $<\vec{T}^I>$, as well as $\vec{u}$ and automatically $\vec{v}$.

For 3-dimensional algorithms ($n=3$ as for the `CKA` and the `KUKA` configurations), the constraint in eq. {eq}`eq_vdotu`
does not define a unique vector $\vec{v}$. One degree of freedom is left for defining $\vec{v}$: the rotation angle $\theta_v$ around
the ice-line defined by $\vec{u}$. An infinity of $\vec{v}$ - and thus an infinity of SIC algorithms as defined by eq. {eq}`eq_mixC2` - 
pass the constraint in eq. {eq}`eq_vdotu`. To fully define a 3-dimensional algorithm, we find the rotation angle $\theta_v$ that
minimizes the retrieval uncertainty of the algorithm for either the set of known 0% SIC conditions, or the set of known 100% SIC
conditions. In general, the $\theta_v$ that minimizes the retrieval uncertainty over open water conditions does not minimize the
retrieval uncertainty over consolidated ice conditions (and vice-versa). We thus define two $\theta_v$ values, corresponding to
two $\vec{v}$ vectors and thus two SIC algorithms. The concept of a 3-dimensional algorithm and the optimization of the rotation
angle $\theta_v$ are illustrated on {numref}`fig_3d_rotation`. The two SIC algorithms, called "BestIce" and "BestOW", will be combined
into a hybrid SIC algorithm as described in {ref}`sec_hybrid_sics`.

```{figure} ./static_imgs/SIC_3d_and_rotation.png
--- 
name: fig_3d_rotation
---
Three-dimensional diagram of open-water (H) and closed-ice (ice line between D and A) brightness temperatures in a `KUKA` configuration (Ku-V, Ka-V, Ka-H space) (black dots). The original figure is from {cite:t}`smith:1996:bristol`.
The direction U (violet, sustained by unit vector $\vec{u}$) is shown, and vectors $\vec{v}_{Bristol}$ (blue), $\vec{v}_{BestIce}$ (red), and $\vec{v}_{BestOW}$ (green) are added, as well as an illustration of the optimization of the direction
of $\theta_v$ around the ice-line. (b) Evolution of the SIC algorithm accuracy for open-water (blue) and closed-ice (red) training samples as a function of the rotation angle $\theta_v$ in the range $[-90^{\circ};+90^{\circ}]$.
Square symbols indicate the $\theta_v$ and accuracy for the Bootstrap Frequency Model (BFM) algorithm and the Bristol (BRI) algorithms. Disk symbols locate the optimized algorithm. Figure reproduced from {cite:t}`lavergne:2019:sicv2`.
```

The optimization of $\theta_v$ values uses a Quaternion rotation notation in 3 dimensions. A brute-force approach is chosen where
all $\theta_v$ values in the range $[-90^{\circ};+90^{\circ}]$ are tested with a step of $1^{\circ}$.

#### Implementing a consolidated sea-ice curve



(sec_hybrid_sics)=
#### An hybrid SIC algorithms


#### Pan-sharpening of SIC fields

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


