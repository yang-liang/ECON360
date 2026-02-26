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
---


### Empirical Illustration of Export Composition (Heckscher–Ohlin Model)
**Objective**
This exercise uses 1995 HS92 trade data to examine whether countries export relatively more **labor-intensive** or **capital-intensive** goods.
The goal is to provide a simple empirical illustration of the **Heckscher–Ohlin (H-O) model**, which predicts that countries export goods that intensively use their relatively abundant factors.
In this example, we focus on the **United States (1995)** and visualize whether its exports are more capital-intensive or labor-intensive.
**Data Sources**
- **HS92 Trade Data (1995)**  
  Bilateral export values by exporter, importer, and 6-digit HS product code.
- **CEPII Country Codes**  
  Country identifiers and ISO2/ISO3 codes.
### 1. Merge Country Codes with Trade Data
- Import CEPII country codes.
- Merge with HS92 trade data using exporter country code.
- Drop unmatched country observations.
- Rename variables for clarity (e.g., `exporter`, `importer`, `export_value`, `hs92`).
This ensures that each export observation is linked to its corresponding country information.
### 2. Aggregate Trade Data
Trade data are collapsed to obtain total export value by:
- `exporter`
- `exporter_iso3`
- `hs92` (6-digit product code)
This removes importer-level variation and focuses on total exports by country and product.
### 3. Classify Products by Factor Intensity
Products are grouped into labor- or capital-intensive industries based on HS chapter codes:
#### Labor-Intensive Industries
- HS 50–63 → Textiles & Apparel  
- HS 64–67 → Footwear  
#### Capital-Intensive Industries
- HS 72–83 → Metals  
- HS 84–85 → Machinery  
- HS 87–89 → Transport Equipment  
The HS chapter is extracted from the 6-digit HS92 code.
### 4. Construct Export Shares
For each exporter:
1. Sum export values by factor-intensity type.
2. Compute total exports.
3. Calculate export shares:

\[
\text{Export Share}_{i,t} = 
\frac{\text{Exports of type}}{\text{Total exports}}
\]
This generates two shares per country:
- Labor-intensive export share
- Capital-intensive export share
### Interpretation
If the United States displays a larger share of capital-intensive exports, this is consistent with:
- The U.S. being relatively capital-abundant.
- The H-O model prediction that capital-abundant countries export capital-intensive goods.
This exercise demonstrates how trade data alone can provide a simple empirical illustration of comparative advantage.
---
```stata
set more off
clear all

global path "G:\My Drive\Class\International Economics\360\data\gravity\hands on gravity stata tutorial"

import delimited "${path}\country_codes_V202102.csv", clear

* merge with trade flows based on exporter's country #code
rename country_code i // rename country_name to i (exporter) 
merge 1:m i using "${path}\HS92_1995.dta" // for year 1995
* check merge! 
assert _merge==1 | _merge==3 // 1: only in master file (country codes), 2: only in using (trade flows), 3: matched in both

drop if _merge==1   // drop country codes with no trade
drop _merge

* this means that we have successfully merged the trade flows with country codes, and we can proceed with our analysis.

* data cleaning and rename varialbes for easier interpretation
rename i exporter
rename j importer

rename country_name_full exporter_name
rename country_name_abbreviation exporter_abbrev
rename iso_2digit_alpha exporter_iso2
rename iso_3digit_alpha exporter_iso3

rename v export_value
rename q quantity
rename k hs92
describe

* collapse the data to get total export value by exporter, exporter iso3 code, and hs92 code
collapse (sum) export_value, by(exporter exporter_iso3 hs92)

/* 
  Labor-intensive:
  HS 50–63 → Textiles & Apparel
  HS 64–67 → Footwear
  Capital-intensive:
  HS 84–85 → Machinery
  HS 87–89 → Transport equipment
  HS 72–83 → Metals 
*/

gen hs_chapter = floor(hs92/10000)

gen labor_intensive = inrange(hs_chapter,50,67)
gen capital_intensive = inrange(hs_chapter,72,89)

* collapse the data to get total export value by exporter, labor_intensive, and capital_intensive
keep if labor_intensive==1 | capital_intensive==1
gen type = labor_intensive*1 + capital_intensive*2 // 1 for labor-intensive, 2 for capital-intensive
collapse (sum) export_value, by(exporter type)

* calculate the share of labor-intensive and capital-intensive exports for each exporter
bysort exporter: egen total_exports = total(export_value)
gen export_share = export_value / total_exports

label define typelab 1 "Labor-Intensive" 2 "Capital-Intensive"
label values type typelab

graph bar export_share if exporter==842, ///
    over(type) ///
    title("US Export Composition (1995)") ///
    subtitle("1= Labor-Intensive vs 2= Capital-Intensive") ///
    ytitle("Share of Total Exports") ///
    blabel(bar, format(%4.2f))
```
---
