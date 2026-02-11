# Gravity Equation
### [Dataset Download HERE](https://drive.google.com/drive/folders/1YvhiWfYivtZyk6N1RPxgLMa4gekKO2KP) 
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
cd "C:\Users\yang.liang\Downloads" // change this to your own download
use "workfile_china_long.dta", replace
// 2. Install packages
// (you must first install the packages and the maptile files, see [instruction]:(https://michaelstepner.com/maptile/)
ssc install maptile, replace
ssc install spmap, replace
maptile_install using "http://files.michaelstepner.com/geo_cz1990.zip", replace
// 3. Plot Import Penetration
rename czone cz
maptile d_tradeusch_pw, geo(cz1990) fcolor(Reds) twopt(title(""))
```
---
### Empirical Testing of the Ricardian Model
The simple Ricardian model delivers three core predictions:
1. Comparative Advantage: Countries trade more when their productivity differences are larger. Greater productivity differences → stronger comparative advantage → more trade.
2. Gains from Trade

| Theory Concept | Data Proxy                               |
| -------------- | ---------------------------------------- |
| Productivity   | GDP per capita (`gdpcap_o`, `gdpcap_d`)  |
| Trade          | Bilateral trade flow (`flow`)            |
| Trade Costs    | Distance (`distw`), contiguity, language |
| Income         | GDP per capita                           |
| Openness       | Exports / GDP                            |

**Hypothesis Testing 1**
```stata
// load the same gravity dataset//

* Log trade
gen ln_trade = ln(flow)

* Log GDP per capita
gen ln_gdppc_o = ln(gdpcap_o)
gen ln_gdppc_d = ln(gdpcap_d)

* Productivity difference
gen diff_prod = abs(ln_gdppc_o - ln_gdppc_d)

* Log distance
gen ln_dist = ln(distw)

// for the USA
twoway (scatter ln_trade diff_prod if year==2005 & iso_o=="USA", msize(vsmall)) ///
       (lfit ln_trade diff_prod if year==2005 & iso_o=="USA"), ///
       title("Trade and Productivity Differences") ///
       ytitle("Log Bilateral Trade") ///
       xtitle("Absolute Log GDP per Capita Difference") ///
       graphregion(color(white)) ///
       scheme(s1mono)

// for CHINA
twoway (scatter ln_trade diff_prod if year==2005 & iso_o=="CHN", msize(vsmall)) ///
       (lfit ln_trade diff_prod if year==2005 & iso_o=="CHN"), ///
       title("Trade and Productivity Differences") ///
       ytitle("Log Bilateral Trade") ///
       xtitle("Absolute Log GDP per Capita Difference") ///
       graphregion(color(white)) ///
       scheme(s1mono)
```

**Hypothesis Testing 2**
```stata
collapse (sum) exports=flow ///
         (mean) gdp_o gdpcap_o pop_o, ///
         by(year iso_o)

rename iso_o iso
rename gdp_o gdp
rename gdpcap_o gdpcap
rename pop_o pop

gen openness = exports / gdp * 100
gen ln_open = ln(openness)
gen ln_gdppc = ln(gdpcap)

twoway (scatter ln_gdppc ln_open, msize(small)) ///
       (lfit ln_gdppc ln_open), ///
       title("Income and Trade Openness") ///
       ytitle("Log GDP per Capita") ///
       xtitle("Log Openness")
```
