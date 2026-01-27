# Gravity Equation
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

local year_subsample "if year>=1984 & year <= 2004"
eststo: qui reghdfe lnX lnGDP_o lnGDP_d rta lndist gatt_d contig comlang_off gsp `year_subsample',a(year)
eststo: qui reghdfe lnX lnGDP_o lnGDP_d rta lndist gatt_d contig comlang_off gsp `year_subsample',a(year iso_o iso_d)
eststo: qui reghdfe lnX lnGDP_o lnGDP_d rta `year_subsample',a(year iso_O#iso_D)
esttab,se

local year_subsample "if year>=1989 & year <= 1999"
eststo: qui reghdfe lnX lnGDP_o lnGDP_d rta lndist gatt_d contig comlang_off gsp `year_subsample',a(year)
eststo: qui reghdfe lnX lnGDP_o lnGDP_d rta lndist gatt_d contig comlang_off gsp `year_subsample',a(year iso_o iso_d)
eststo: qui reghdfe lnX lnGDP_o lnGDP_d rta `year_subsample',a(year iso_O#iso_D)
esttab,se

scatter lnX lnGDP_o if iso_d=="USA",ms(oh) graphregion(color(white)) scheme(s1mono)
```
---
