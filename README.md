# frezchem
_Legacy repository of the `FREZCHEM` code developed by [Giles Marion](https://www.dri.edu/directory/giles-marion) (Desert Research Institute) et al. This and previous versions used to be freely accessible at frezchem.dri.edu but that page is no longer accessible._

_Efforts are underway by Jonathan Toner and David Catling at the University of Washington to revisit the thermodynamic model behind `FREZCHEM` and move it to the versatile `PHREEQC` code. Here is an [example publication](https://www.researchgate.net/profile/Jonathan_Toner/publication/271601980_Modeling_salt_precipitation_from_brines_on_Mars_Evaporation_versus_freezing_origin_for_soil_salts/links/5aa2eb92a6fdccd544b75832/Modeling-salt-precipitation-from-brines-on-Mars-Evaporation-versus-freezing-origin-for-soil-salts.pdf) from their work._

_Below are some of the release notes for `FREZCHEM` version 17. See [full v17 release notes](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf)._

## Release notes (v17)
Attached is a "beta" version of the `FREZCHEM` model that includes:
* chloride, bromide, perchlorate, nitrate, sulfate, and bicarbonate-carbonate salts, 
* strong acid chemistry, 
* ferrous and ferric iron chemistry,
* aluminum and silicon chemistries,
* ammonia and ammonium chemistries,
* methane, ethane, and propane chemistries, and 
* gas hydrate chemistry. 

This version includes both temperature and pressure dependencies. This repository includes a FORTRAN source code file, four input files, and a PDF file including these instructions as well as a list of chemical species in the model (Table 1), instructions for model inputs (Tables 2-5, 8, and 10), and four examples of model outputs (Tables 6-7, 9 and 11).

### General advice
This model is very much a work in progress. I _[Giles Marion]_ may be adding new chemistries to the model in the next few years. I have not spent much time debugging the model or making it user-friendly. In addition, there are convergence problems, at times, with the model. My _[Giles Marion's]_ version of the model was created with Absoft's ProFortran for Mac. Porting this code to another FORTRAN compiler is always problematic. _(Note from Marc: I use the freely available [`GFortran compiler`](http://hpc.sourceforge.net))._ Once you have a `FREZCHEM` model working, verify correctness from an example from published data (see examples in the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf)). If you have additional problems, [email me](https://www.dri.edu/directory/giles-marion) _(Giles Marion)_. Indicate the `FREZCHEM` version you are using (e.g., `FREZCHEM17`) and your model input.

The model is an equilibrium chemical thermodynamic model, meaning it will always select the most stable minerals. There are a few minerals (e.g., aragonite and vaterite) that are always metastable with respect to other minerals (e.g., calcite). To explicitly include a metastable mineral in your calculations necessitates removing the stable mineral from the mineral database. This is most simply done by assigning the stable mineral an arbitrary high _Ksp_ through `SOLIDPHASE.txt`. The # of the _Ksp_ for a specific mineral in the FORTRAN program is the same as the solid phase # in Table 1 of the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf) (e.g., `K52` is the solubility product for calcite). If you are using the model to calculate pH, then you should make sure that the initial solution is charge-balanced. Otherwise, the model will force a charge balance by changing the bicarbonate-carbonate or acid concentrations, which could lead to a serious error in calculated pH if the solution is badly charge-balanced. If necessary, force a charge-balance in the initial solution by changing a major constituent that minimizes the effect on pH (e.g., Na or Cl). If you input Fe, Al, Si, or alkalinity, then you will have four options on how to deal with pH; adding NH3-NH4 adds a fifth option (see Table 2 of the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf)).

Compared to earlier versions, this Version 17 of the `FREZCHEM` model contains new parameterizations dealing with ethane, propane, methane, and nitrogen species. Also, Versions 15-17 lowered the temperature level to 173 K (see Tables 8-11 of the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf)).

## Model inputs
A fundamental change was made in `FREZCHEM 13` on how to input data into the model. Earlier versions required inputs via the computer screen. Versions 13-17 require inputs via data files; this approach simplifies and speeds up model inputting. There are four input files that must be built to run FREZCHEM. In the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf): 
* __Table 2__ describes the main model inputs (material in the `Input.txt` file of Table 3)
* __Table 3__ presents the main `Input.txt` file. Inputs are all placed to the left of the `,`. Do not remove the `,`. That comma separates model input from descriptive words.
* __Table 4__ describes in more detail how to handle gases in `Input.txt`
* __Table 5__ lumps togeher three minor input files: `SOLIDPHASE.txt`, `SOLIDMASS.txt`, and `NUANCES.txt`.
  * `SOLIDPHASE.txt` (Table 5A) allows the user to remove all solid phases from equilibrium calculations or some specific minerals. That option allows for a pure solution phase calculation without any minerals precipitating. In the molar to molal conversion example in Table 7 of the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf)), all solid phases had to be removed. Note the assigned exceptionally high equilibrium constants in Table 7, which is what keeps the solid phases from precipitating. In a previous Phoenix site calculation ([Marion et al., 2010](https://doi.org/10.1016/j.icarus.2009.12.003)), we removed magnesite (MgCO3) and dolomite [CaMg(CO3)2] from model calculations (Table 5A), which led to calcite (CaCO3) and hydromagnesite [3Mg(CO3)2•MgSO4•3H2O] precipitating. Generally, model calculations start with aqueous/gas phases, without initial solid phases; but if you want a particular solid phase to control the solution phase chemical composition, then you can specify this solid phase in `SOLIDPHASE.txt`.
  * `SOLIDPHASE.txt` (Table 5B) is where the molar amounts of the phases in `SOLIDPHASE.txt` are specified. These are arbitrary amounts that must not completely dissolve in 1.0 kg H2O. For example, we assumed that Earth seawater would have been saturated with dolomite during Snowball Earth ([Marion and Kargel, 2008](https://doi.org/10.1007/978-3-540-75679-8)). Changing the first line of SOLIDMASS.txt from No(0) to YES(1) and specifying dolomite would allow saturation with dolomite.
  * `NUANCES.txt` (Table 5C) allows for temperature, water content, or pressure changes to be adjusted during a specific run. For example, if you want to know the eutectic temperature of a salt assemblage, and you know that this will occur slightly below 259 K, you could change the _∆T_ term from 5 K between 298 and 263 K (as assigned by `Input.txt`) to 1 K between 263 and 259 K, and 0.1 K below 259 K. This scenario would allow for a more accurate estimate of the eutectic temperature than using either a 5 K or 1 K term for the _∆T_ decrement. With respect to `NUANCES.txt`, always retain two steps for temperature, water content, and pressure changes, even if you need to duplicate two steps (e.g., 263.15 0.1, 263.15 0.1).

### `Input.txt`
* `Title:` Any alphanumeric character up to 50 characters.
* `Freeze(1) or Evaporation(2) or Pressure (3) Pathway:` Enter `1`, `2`, or `3` depending on whether you
want to simulate a temperature change (`1`), an evaporation (`2`), or a pressure change (`3`). For evaluating a single point, enter `1`.
* `Equilibrium(1) or Fractional(2) Crystallization:` In equilibrium crystallization (`1`), precipitated solids are allowed to re-equilibrate with the solution phase as environmental conditions change. In fractional crystallization (`2`), precipitated solids are removed and not allowed to re-equilibrate with the solution phase as environmental conditions change.
* `Seawater Salinity:` If you want seawater salinity to govern the calculations, enter `1` for Yes, and `0` for No.
* `Practical Salinity:` If Yes in the above line, enter `SP`. If No, enter `0.0`.
* `Calcite Supersaturation in Seawater:` If you want this to be considered, enter `1` for Yes, or `0` for No.

#### Cations
`Sodium (m/kg):` Enter sodium molality (moles/kg(water)). Otherwise, enter `0.0`. Same for `Potassium`, `Calcium`, `Magnesium`, `Strontium`, `Ferrous Iron`, `Ferric Iron`, `Aluminum`, `Silicon`, and `Ammonium`.

#### Acidity and pH
If `Iron`, `Aluminum`, `Silicon`, `Alkalinity`, or `Ammonia-Ammonium` molalities are nonzero, then choose an
acidity option:
* `Acidity ignored (Option 1)`, enter `1`.
* `Acidity fixed by pH (Option 2)`, enter `2`.
* `Acidity fixed by H+ ion concentration (Option 3)`, enter `3`. 
* `Acidity fixed by alkalinity (Option 4)`, enter `4`.
* `Acidity fixed by NH3(aq) and NH4(aq) (Option 5)`, enter `5`.
* `Initial pH:` If an acidity opttion was chosen, then for Option `1`, enter `0`; Option `2`, enter the pH; Options `3` and `4`, enter an approximate pH; Option `5`, enter `10`. If no acidity option was chosen, specify the pH. (e.g., `7.0`).

#### Anions
* `Chloride (m/kg):` Enter chloride molality (moles/kg(water)). Otherwise, enter `0.0`. Same for `Bromide`, `Perchlorate`, `Sulfate`, and `Nitrate`.
* `Carbonate Alkalinity:` Enter as equivalents/kg(water). If alkalinity = 0.0, then you must enter `0.0`. The latter will cause the model to skip all bicarbonate-carbonate with pH chemistries in the model.
* `Sulfite Alkalinity:` Enter as equivalents/kg(water). If alkalinity = 0.0, then you must enter `0.0`.
* `Sulfide Acidity:` Enter as equivalents/kg(water). If acidity = 0.0, then you must enter `0.0`.
* `Acidity:` Enter as equivalents/kg(water). This is the total hydrogen concentration, if known initially. Generally this is only known for strong acid solutions. For example, for a 1 molal H2SO4 solution, enter `2.00`. Otherwise, enter `0.0`. The equations used to calculate pH for the alkalinity and acidity cases are incompatible. So, a specification of either carbonate alkalinity or acidity requires that the other variable be assigned a value of `0.0`. This will channel the calculations to the proper algorithm.
  * `HCl(bars):` If the HCl atmospheric concentration is known, then specify here. Otherwise, enter `0.0`. If you specify `0.0`, then the model will calculate HCl(bars). Note that if you specify HCl(bars) or the other acids below, then these properties override the total acidity specification (see above). That is, the solution is equilibrated with the atmospheric concentration. You can specify atmospheric concentrations for some acids (e.g., HCl and HNO3) and leave other acid partial pressure unspecified (e.g., H2SO4 = `0.0`).
  * `HNO3(bars):` If the HNO3 atmospheric concentration is known, then specify here. Otherwise, enter `0.0`.
  * `H2SO4(bars):` If the H2SO4 atmospheric concentration is known, then specify here. Otherwise, enter `0.0`.
* `Boron (m/kg):` Enter boron molality (moles/kg(water)). Otherwise, enter `0.0`.
* `Fluoride (m/kg):` Enter fluoride molality (moles/kg(water)). Otherwise, enter `0.0`.

#### Gases
* `Initial Total Pressure (bars):` Enter the initial total pressure of the system.
* `Initial CO2(bars):` If alkalinity > `0.0` or CO2 hydrates are simulated, then specify the initial concentration of CO2(g) in bars.
* `Mole Fraction of CO2:` Enter the mole fraction of CO2(g) for the system (mole fraction = PCO2/total pressure). For pure CO2, enter `1.0`. If `0.0`, then CO2(g) is fixed and independent of total pressure.
`O2(bars):` If the atmospheric concentration of oxygen is known, then specify here. Otherwise, enter `0.0`. If you are interested in ferrous iron chemistry, then you may want to assign O2 a value of `0.0`. Otherwise, it is likely that the insolubility of ferric minerals in the presence of O2 will cause all the iron to precipitate as a ferric mineral (see discussions in [Marion et al., 2003a](https://doi.org/10.1016/S0016-7037(03)00372-7)).
* `Initial CH4(bars):` If CH4 is simulated, then specify the initial concentration of CH4(g) in bars.
* `Mole Fraction of CH4:` Enter the mole fraction of CH4(g) for the system (mole fraction = PCH4/total pressure). For pure CH4, enter `1.0`. If `0.0`, then CH4(g) is fixed and independent of total pressure.
* `Initial NH3(bars):` If NH3(g) are inputs, then specify the initial concentration of NH3(g) in bars.
* `Initial NH3(aq):` Enter NH3(aq) molality (moles/kg(water)). Do not enter positive values (> 0) for both `NH3(bars)` and `NH3(aq)`.
* `Initial N2(bars):` If the atmospheric concentration of nitrogen is known, then specify here. Otherwise, enter `0.0`.
* `Mole Fraction of N2:` Enter the mole fraction of N2(g) for the system (mole fraction = PN2/total pressure). For pure N2, enter `1.0`. If `0.0`, then N2(g) is fixed and independent of total pressure.

#### Gas hydrates
`Mixed CH4-CO2 Gas Hydrate?:` If both CH4(g) and CO2(g) are specified as inputs, then you can use this data to estimate the stability of a mixed CH4-CO2 gas hydrate (`1`) or treat the two gases as independent gas hydrates (`0`). Same for 
oher combinations of N2, CH4, CO2, C2H6, and C3H8.

#### Other inputs
* `Molar to Molal Conversion?:` If you want to convert molar data into molal concentrations, then enter `1` for Yes or `0` for No.
* `Salinity/liter:` If "Yes" above, then you must enter the total aqueous salinity (g salt/liter), which can be calculated from molar data [g salt/liter = ∑(moles/liter) x (g salt/mole)]. In the case depicted in Table 7 of the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf), the `SL` value is 316.57 g salt/liter (5.417 x 58.44).
* `Initial Temperature(K):` Enter the temperature Kelvin for start of simulation (e.g., `273.15`).
* For Temperature Change Pathway(1):
  * `Final Temperature(K):` Enter final temperature of simulation (e.g., `263.15`).
  * `Temperature Decrement(K):` The temperature interval between simulations (e.g. `1`). For the above temperature designations, the model would calculate equilibrium starting at 273.15 K and ending at 263.15 K at 1 K intervals. If you want to change the decrement in a run (e.g., to reduce the step size near an equilibrium), see File `NUANCES.txt.`
* For Evaporation Pathway(2):
  * `Initial Water (g):` Normally enter `1000`. The standard weight basis of the model is 1000 g water plus associated salts. In you enter `100` instead, the initial ion concentrations specified above will be multiplied by 10 (i.e., 1000/100). This feature of the model is useful in precisely locating where minerals start to precipitate during the evaporation process without having to calculate every small change between 1000 g and 1 g.
  * `Final Water (g):` Enter the final amount of water that you want to remain in the system (e.g., `100`).
  * `Water Decrement (g):` Enter the water decrement for simulations (e.g., 50 g). Specifying initial = `1000`, final = `100`, and decrement = `50` would result in calculations at 1000 g, 950 g, ... 100 g. To change the decrement in a run (e.g., to reduce the step size near an equilibrium), use `NUANCES.txt.`
* For Pressure Pathway(3):
  * `Final Pressure(bars):` Enter the final pressure of the simulation [e.g., `101.01325` bars (100 atm)].
  * `Pressure Increment(bars):` Enter the pressure increment. For example, if initial pressure is 1.01 bars, final pressure is 101.01 bars, and pressure increment is 1.0 bars, then the simulation would calculate at 1.01, 2.01, 3.01, ... 101.01325 bars. To change the increment in a run, use `NUANCES.txt.`

## Model outputs

See Table 6 of the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf).

* `Temp(K)`: temperature.
* `Ion.Str.` is the ionic strength of the equilibrium solution (see Table 6 of the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf)).
* `RHO` is the density of the solution.
* `Phi` is the osmotic coefficient of the equilibrium solution.
* `H2O(g)` is the amount of water remaining as liquid.
* `Ice` is the amount of water that formed ice between 273.15 K. The mass basis for calculation in the model is 1.0 kg of water (except for evaporation); therefore, the water in liquid water + ice + hydrated salts should always sum to 1.0 kg.
* `Press.(bars)`: pressure.

### Solution section
* `Initial Conc.` are the input concentrations at 273 K (Table 3).
* `Final Conc.` are the equilibrium concentrations at 273 K. 
* `Act. coef.` (activity coefficient) and `Activity` are self-explanatory.
* `Moles` are the # of moles in the solution phase.
* `Mass Balance`: for the major constituents, this column should generally agree with the input column (`Initial Conc.`). __This is the best check on the internal consistency of the calculations.__ 

### Solids section
* `Moles`: final number of moles present.
* `Equil. Constant`: equilibrium constant.
* `Accum Moles`: net number of moles of that solid that have precipitated.

## Example simulations using the new features of recent versions
__Table 6__ of the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf) is a case where we examined sodium/chloride/ethane/propane ([Marion et al., 2015](https://doi.org/10.1016/j.icarus.2015.04.035)) using the `Input.txt` file in Table 3 of the [PDF file](https://github.com/MarcNeveu/frezchem/blob/master/Notes_FRIENDS_OF_FREZCHEM17.pdf). The high concentrations of ethane and propane cases causes a solid phase of these gases forming together on a mixed gas hydrate. See the bottom of Table 6 with both ethane and propane forming together.

__Table 7__ is a case where we _[Marion et al.]_ converted molar into molal concentrations. In the upper table there are three columns labeled `rho`, `SA`, and `CF` that represent model calculated estimates of density (kg(soln.)/liter or g/cm3), absolute salinity [g salt/kg(soln.)], and the conversion factor [liters/kg(H2O)]. The iterations quickly converted molar concentrations (`5.4170` moles/liter under `Initial Conc.`) into molal concentrations [`6.1458` moles/kg(H2O) under `Final Conc.`]. In addition to inputs of molar concentrations, this algorithm also requires salinity on a liter basis (`SL`) (see Table 3). In this case, the `SL` value is `316.57` g/liter [= 5.417 x 58.44 (molecular weight of NaCl)]. So even with no prior knowledge about solution density (the model arbitrarily assigns initial density = 1.00 g/cm3), we were able to quickly calculate density and convert molar to molal concentrations. In turn, molal concentrations could be directly imported into `FREZCHEM` to explore geochemical processes. Note that all the potential solid phases were assigned high solubility products to prevent their precipitation (Table 7). FORTRAN model inputs to accomplish this negation of solid phases are in the `SOLIDPHASE.txt` file in Table 5 and must be implemented by the user. See [Marion (2007)](https://doi.org/10.1016/j.cageo.2006.10.009) for a fuller discussion of the techniques used in this algorithm.

__Table 8__ is an input file for a Titan simulation from 273 K to 173 K that is dominated with NH3(aq), NH3(g), NH4(aq), and CH4(g).

__Table 9__ is the output of this simulation after the initial temperature at 273 K dropped to 173 K. At 173 K, ice had formed and methane hydrate, NH4Cl, and (NH4)2SO4 had precipitated. NH3(aq), which started at 10.0 molal, has risen to 36.27 molal that is approaching the eutectic where NH3•2H2O would precipitate. In this case, the pH started at 10.0 (Table 8) and rose to 18.03 (Table 9). The latter may not be accurate.

__Table 10__ is an input file for a Titan simulation from 273 to 173 K that is dominated with NH3(aq), NH3(g), NH4(aq), N2(bars), and CH4(bars).

__Table 11__ is the output of this simulation after the initial temperature at 273 K dropped to 173 K. At 173 K, ice had formed, NH4Cl and (NH4)2SO4 had precipitated, and N2•6H2O-CH4•6H2O had formed. NH3(aq), which started at 10.0 molal, has risen to 30.34 molal. In this case, the pH started at 10.0 (Table 10) and rose to 17.89. These Tables 10-11 are very similar to Tables 8-9, except that a mixed gas hydrate formed in this case and methane hydrate formed in the previous case.

## Publications
The validation of this model is discussed in 17 publications: 
1. Spencer et al. (1990) The prediction of mineral solubilities in natural waters: A chemical equilibrium model for the Na-K-Ca-Mg-Cl-SO4-H2O system. _Geochim. Cosmochim. Acta_ __54__:575-590. https://doi.org/10.1016/0016-7037(90)90354-N
2. Marion and Farren (1999) Mineral solubilities in the Na-K-Mg-Ca-Cl-SO4-H2O system: A re-evaluation of the sulfate chemistry in the Spencer-Møller-Weare model. _Geochim. Cosmochim. Acta_ __63__:1305-1318. https://doi.org/10.1016/S0016-7037(99)00102-7
3. Marion (2001) Carbonate mineral solubility at low temperatures in the Na-K-Mg-Ca-H-Cl-SO4-OH-HCO3-CO3-CO2-H2O system. _Geochim. Cosmochim. Acta_ __65__:1883-1896. https://doi.org/10.1016/S0016-7037(00)00588-3
4. Marion (2002) A molal-based model for strong acid chemistry at low temperatures (<200 to 298 K). _Geochim. Cosmochim. Acta_ __66__:2499-2516. https://doi.org/10.1016/S0016-7037(02)00857-8
5. Marion et al. (2003) Modeling aqueous ferrous iron chemistry at low temperatures with application to Mars. _Geochim. Cosmochim. Acta_ __67__:4251-4266. https://doi.org/10.1016/S0016-7037(03)00372-7
6. Marion et al. (2005) Effects of pressure on aqueous chemical equilibria at subzero temperatures with applications to Europa. _Geochim. Cosmochim. Acta_ __69__:259-274. https://doi.org/10.1016/j.gca.2004.06.024
7. Marion et al. (2006) Modeling gas hydrate equilibria in electrolyte solutions. _Calphad_ __30__:248-259. https://doi.org/10.1016/j.calphad.2006.04.002
8. Marion (2007) Adapting molar data (without density) for molal models. _Computers & Geosciences_ __33__:829-834. https://doi.org/10.1016/j.cageo.2006.10.009
9. Marion and Kargel (2008) _Cold Aqueous Planetary Geochemistry with FREZCHEM: From Modeling to the Search for Life at the Limits._ Springer. https://doi.org/10.1007/978-3-540-75679-8
10. Marion et al. (2008) Modeling ferrous-ferric iron chemistry with application to Martian surface geochemistry. _Geochim. Cosmochim. Acta_ __72__:242-266. https://doi.org/10.1016/j.gca.2007.10.012
11. Marion et al. (2009) Br/Cl partitioning in chloride minerals in the Burns formation on Mars. _Icarus_ __200__:436-445. https://doi.org/10.1016/j.icarus.2008.12.004
12. Marion et al. (2009) Modeling aluminum-silicon chemistries and application to Australian playa lakes as analogues for Mars. _Geochim. Cosmochim. Acta_ __73__:3493-3511. https://doi.org/10.1016/j.gca.2009.03.013
13. Marion et al. (2011) Modeling hot spring chemistries with applications to Martian silica formation. _Icarus_ __212__:629-642. https://doi.org/10.1016/j.icarus.2011.01.035
14. Marion et al. (2012) Modeling ammonia-ammonium aqueous chemistries in the Solar System’s icy bodies. _Icarus_ __220__:932-946. https://doi.org/10.1016/j.icarus.2012.06.016
15. Marion et al. (2013) Sulfite-sulfide-sulfate-carbonate equilibria with applications to Mars. _Icarus_ __225__:342-351. https://doi.org/10.1016/j.icarus.2013.02.035
16. Marion et al (2014) Modeling nitrogen-gas, -liquid, -solid chemistries at low temperatures (173-298 K) with applications to Titan. _Icarus_ __236__:1-8. https://doi.org/10.1016/j.icarus.2014.03.025
17. Marion et al (2015) Modeling nitrogen and methane with ethane and propane gas hydrates at low temperatures (173-290 K) with applications to Titan. _Icarus_ __257__:355-361. https://doi.org/10.1016/j.icarus.2015.04.035
