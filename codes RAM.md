# Gravity Equation
### [Gravity Dataset Download](https://drive.google.com/drive/folders/1YvhiWfYivtZyk6N1RPxgLMa4gekKO2KP) 
```stata
************************************************************
// install packages
ssc install estout, replace

// define your path
global path "G:\My Drive\Class\International Economics\360\data\gravity\hands on gravity stata tutorial"  //swith "\" to "/" for mac users

// load gravity data 
use "$path\gravity.dta", clear

/* 
  quietly ds
  local varlist `r(varlist)'
  
  foreach new in newlist varname varlabel {
  quietly generate str `new' = ""
  }
  
  local k 1
  foreach var of varlist `varlist' {
  local varlabel : variable label `var'
  quietly replace varname = "`var'" in `k'
  quietly replace varlabel = "`varlabel'" in `k'
  local ++k
  }
  
  quietly export excel varname varlabel using Varnamesnlabels.xlsx, ///
  sheet("Names & Labels") firstrow(variables)
*/


encode iso_o, gen(iso_O)
encode iso_d, gen(iso_D)
gen lnX=ln(flow)
gen lndist=ln(distw)
gen lnGDP_o=ln(gdp_o)
gen lnGDP_d=ln(gdp_d)

label var iso_o "origin country ISO"
label var iso_d "destination country ISO"
label var lnGDP_o "ln Origin GDP"
label var lnGDP_d "ln Destination GDP"
label var lndist "ln Distance"
label var contig "1 for contiguity"
label var gsp "GSP Dummy"
label var rta "RTA"
label var gatt_d "GATT/WTO Member"
label var comlang_off "1 for common language"

/* Regression Full*/
eststo clear
eststo: qui reghdfe lnX lnGDP_o lnGDP_d rta lndist gatt_d contig comlang_off gsp,a(year)
eststo: qui reghdfe lnX lnGDP_o lnGDP_d rta lndist gatt_d contig comlang_off gsp,a(year iso_o iso_d)
eststo: qui reghdfe lnX lnGDP_o lnGDP_d rta,a(year iso_O#iso_D)
esttab,se 
```
### Visualization
```stata
* Scatter plot with linear fit line
  twoway ///
    (scatter flow gdp_o if iso_d=="USA" & year==2006 & (inlist(iso_o,"AUT", "BEL", "BGR", "DNK", "EST", "FIN", "FRA", "DEU") |inlist(iso_o,"CAN", "MEX") ), mlabel(iso_o) msize(medium)) ///
    (lfit flow gdp_o if iso_d=="USA" & year==2006 & inlist(iso_o,"AUT", "BEL", "BGR", "DNK", "EST", "FIN", "FRA", "DEU")), ///
    title("EU Exports to US vs GDP") ///
    subtitle("Higher GDP correlates with higher export flows") ///
    xtitle("GDP (billions USD)") ///
    ytitle("Exports to US (billions USD)") ///
    legend(order(1 "EU Countries" 2 "Linear fit")) ///
    note("Data source: Your source here") ///
    graphregion(color(white)) ///
    scheme(s1mono)
```
---
### The China Shock
In this module, we will visualize the spatial distribution of the "China Shock." We are using the Autor, Dorn, & Hanson (ADH) methodology to create the choropleth map.
We will use the Stata command `maptile` to generate these visuals.
Before we can map the data, we must align our economic unit of analysis with geographic boundaries.
* Commuting Zones (CZs): The economic data works at the "Commuting Zone" level (local labor markets).
* Counties: The map boundaries are at the "County" level.
We perform a merge using a "crosswalk" file (`cw_cty_czone.dta`) to assign every US county to its corresponding Commuting Zone.
```stata
// 1. Load ADH Data and Merge with Crosswalk
use "${data}/workfile_china_long.dta", replace
merge 1:m czone using "${data}/cw_cty_czone.dta", assert(2 3) keep(3) nogen

// 2. Prepare FIPS codes for mapping
// 'countyfips' is a helper to standardize ID variables
countyfips, fips(cty_fips)
cap drop _merge
cap drop county
gen county = cty_fips

// 3. Plot Import Penetration
maptile d_tradeusch_pw, geo(county1990) fcolor(Reds) twopt(title(""))
```
