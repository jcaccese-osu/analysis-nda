Definition of convenience variables
Most of these are simple re-definitions of existing columns with simplier names, other columns are re-scored versions of nda18 columns.

Start by reading in the merged data from disk.

```R
script.dir <- "~/Desktop/ABCD/analysis-nda18/notebooks/general/"
setwd(script.dir)
nda18 = readRDS("nda2.0.1_orig.Rds")
#The site_name is anonymized and stored per event in case participants move from one site to another during the study.
nda18$abcd_site = nda18$site_id_l #site_id_l is in longitudianl tracking instrument
### Subjectid
nda18$subjectid = nda18$src_subject_id
### Age (in month)
nda18$age = nda18$interview_age
##NDA used to using "gender" as "sex at birth"; rename it as "sex" and reset empty as NA;
nda18$sex=nda18$gender
nda18$sex[which(nda18$sex=="")]=NA
nda18$sex=factor( nda18$sex, levels= c("F","M"))
#rm gender
nda18=nda18[,-which(colnames(nda18)=="gender")]
### Female. 
nda18$female = factor(as.numeric(nda18$sex == "F"), levels = 0:1, labels = c("no", "yes") ) 
### Household income
household.income = nda18$demo_comb_income_p
household.income[nda18$demo_comb_income_p == "1"] = 1 # "[<50K]"
household.income[nda18$demo_comb_income_p == "2"] = 1 # "[<50K]"
household.income[nda18$demo_comb_income_p == "3"] = 1 # "[<50K]"
household.income[nda18$demo_comb_income_p == "4"] = 1 # "[<50K]"
household.income[nda18$demo_comb_income_p == "5"] = 1 # "[<50K]"
household.income[nda18$demo_comb_income_p == "6"] = 1 # "[<50K]"
household.income[nda18$demo_comb_income_p == "7"] = 2 # "[>=50K & <100K]"
household.income[nda18$demo_comb_income_p == "8"] = 2 # "[>=50K & <100K]"
household.income[nda18$demo_comb_income_p == "9"] = 3 # "[>=100K]"
household.income[nda18$demo_comb_income_p == "10"] = 3 # "[>=100K]"
household.income[nda18$demo_comb_income_p == "777"] = NA
household.income[nda18$demo_comb_income_p == "999"] = NA
household.income[household.income %in% c(NA, "999", "777")] = NA
nda18$household.income = factor( household.income, levels= 1:3, labels = c("[<50K]", "[>=50K & <100K]", "[>=100K]") )
#highest education: 5 different levels. These levels correspond to the numbers published by the American Community Survey (ACS). 
high.educ1 = nda18$demo_prnt_ed_p
high.educ2 = nda18$demo_prtnr_ed_p
high.educ1[which(high.educ1 == "999")] = NA
high.educ2[which(high.educ2 == "999")] = NA
high.educ1[which(high.educ1 == "777")] = NA
high.educ2[which(high.educ2 == "777")] = NA
high.educ = pmax(as.numeric(as.character(high.educ1)), as.numeric(as.character(high.educ2)), na.rm=T)
idx <- which(high.educ %in% 0:12, arr.ind = TRUE)
high.educ[idx] = 1 # "< HS Diploma"
idx <- which(high.educ %in% 13:14, arr.ind = TRUE)
high.educ[idx] = 2 # "HS Diploma/GED"
idx <- which(high.educ %in% 15:17, arr.ind = TRUE)
high.educ[idx] = 3 # "Some College"
idx <- which(high.educ == 18, arr.ind = TRUE)
high.educ[idx] = 4 # "Bachelor"
idx <- which(high.educ %in% 19:21, arr.ind = TRUE)
high.educ[idx] = 5 # "Post Graduate Degree"
high.educ[which(high.educ == "999")]=NA
high.educ[which(high.educ == "777")]=NA
nda18$high.educ = factor( high.educ, levels= 1:5, labels = c("< HS Diploma","HS Diploma/GED","Some College","Bachelor","Post Graduate Degree") )
### Marrital status
married = rep(NA, length(nda18$demo_prnt_marital_p))
married[nda18$demo_prnt_marital_p == 1] = 1
married[nda18$demo_prnt_marital_p %in% 2:6] = 0
nda18$married = factor( married, levels= 0:1, labels = c("no", "yes") )
#Add another variable that also includes couples that just live together. 
married.livingtogether = rep(NA, length(nda18$demo_prnt_marital_p))
married.livingtogether[nda18$demo_prnt_marital_p %in% c(1,6)] = 1
married.livingtogether[nda18$demo_prnt_marital_p %in% 2:5] = 0
nda18$married.or.livingtogether = factor( married.livingtogether, levels= 0:1, labels = c("no", "yes") )
### Body-Mass index, remove outlier >36 or < 11 based on the recommendation from Rebecca Umbach, PhD.
nda18$anthro_bmi_calc = as.numeric(as.character(nda18$anthro_weight_calc)) / as.numeric(as.character(nda18$anthro_height_calc))^2 * 703
nda18$anthro_bmi_calc[which(nda18$anthro_bmi_calc>36 | nda18$anthro_bmi_calc < 11)]=NA; 
#These variables can be changed overtime,but need baseline values filled in follow up visits
bl.vars=c("married.or.livingtogether","married","high.educ","household.income") 
bl.demo=nda18[which(nda18$eventname=="baseline_year_1_arm_1"),c("subjectid",bl.vars)]
colnames(bl.demo)[-1]=paste0(bl.vars,".bl") #rename these variables to baseline variables
nda18=merge(nda18,bl.demo,by=c("subjectid"))
dim(nda18)
#Save the new data frame again.
saveRDS(nda18, "nda2.0.1_demo.Rds")
```

Next: categorical_extension